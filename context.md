# Kontekst

- [Wprowadzenie](#introduction)
    - [Jak to działa](#how-it-works)
- [Przechwytywanie kontekstu](#capturing-context)
    - [Stosy](#stacks)
- [Pobieranie kontekstu](#retrieving-context)
    - [Określanie istnienia elementu](#determining-item-existence)
- [Usuwanie kontekstu](#removing-context)
- [Ukryty kontekst](#hidden-context)
- [Zdarzenia](#events)
    - [Odwadnianie](#dehydrating)
    - [Nawadnianie](#hydrated)

<a name="introduction"></a>
## Wprowadzenie

Możliwości "kontekstu" Laravel umożliwiają przechwytywanie, pobieranie i udostępnianie informacji w ramach żądań, zadań i poleceń wykonywanych w Twojej aplikacji. Te przechwycone informacje są również zawarte w logach zapisywanych przez aplikację, dając głębszy wgląd w historię wykonywania kodu, która wystąpiła przed zapisaniem wpisu do logu i umożliwiając śledzenie przepływów wykonywania w systemie rozproszonym.

<a name="how-it-works"></a>
### Jak to działa

Najlepszym sposobem na zrozumienie możliwości kontekstu Laravel jest zobaczenie go w akcji przy użyciu wbudowanych funkcji logowania. Aby rozpocząć, możesz [dodać informacje do kontekstu](#capturing-context) używając fasady `Context`. W tym przykładzie użyjemy [middleware](/docs/{{version}}/middleware) do dodania adresu URL żądania i unikalnego identyfikatora śledzenia do kontekstu przy każdym przychodzącym żądaniu:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;

class AddContext
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        Context::add('url', $request->url());
        Context::add('trace_id', Str::uuid()->toString());

        return $next($request);
    }
}
```

Informacje dodane do kontekstu są automatycznie dołączane jako metadane do wszystkich [wpisów logu](/docs/{{version}}/logging) zapisywanych podczas żądania. Dołączanie kontekstu jako metadanych pozwala odróżnić informacje przekazywane do pojedynczych wpisów logu od informacji udostępnianych przez `Context`. Na przykład, wyobraź sobie, że zapisujemy następujący wpis logu:

```php
Log::info('User authenticated.', ['auth_id' => Auth::id()]);
```

Zapisany log będzie zawierał `auth_id` przekazany do wpisu logu, ale będzie również zawierał `url` i `trace_id` z kontekstu jako metadane:

```text
User authenticated. {"auth_id":27} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```

Informacje dodane do kontekstu są również udostępniane zadaniom wysyłanym do kolejki. Na przykład, wyobraź sobie, że wysyłamy zadanie `ProcessPodcast` do kolejki po dodaniu niektórych informacji do kontekstu:

```php
// In our middleware...
Context::add('url', $request->url());
Context::add('trace_id', Str::uuid()->toString());

// In our controller...
ProcessPodcast::dispatch($podcast);
```

Kiedy zadanie jest wysyłane, wszelkie informacje aktualnie przechowywane w kontekście są przechwytywane i udostępniane zadaniu. Przechwycone informacje są następnie nawadniane z powrotem do bieżącego kontekstu podczas wykonywania zadania. Więc, jeśli metoda handle naszego zadania miałaby zapisać do logu:

```php
class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    // ...

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        Log::info('Processing podcast.', [
            'podcast_id' => $this->podcast->id,
        ]);

        // ...
    }
}
```

Wynikowy wpis logu będzie zawierał informacje, które zostały dodane do kontekstu podczas żądania, które pierwotnie wysłało zadanie:

```text
Processing podcast. {"podcast_id":95} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```

Chociaż skupiliśmy się na wbudowanych funkcjach związanych z logowaniem w kontekście Laravel, poniższa dokumentacja zilustruje, jak kontekst pozwala udostępniać informacje między granicami żądania HTTP / zadania w kolejce, a nawet jak dodawać [ukryte dane kontekstu](#hidden-context), które nie są zapisywane z wpisami logu.

<a name="capturing-context"></a>
## Przechwytywanie kontekstu

Możesz przechowywać informacje w bieżącym kontekście używając metody `add` fasady `Context`:

```php
use Illuminate\Support\Facades\Context;

Context::add('key', 'value');
```

Aby dodać wiele elementów naraz, możesz przekazać tablicę asocjacyjną do metody `add`:

```php
Context::add([
    'first_key' => 'value',
    'second_key' => 'value',
]);
```

Metoda `add` nadpisze każdą istniejącą wartość, która ma ten sam klucz. Jeśli chcesz dodać informacje do kontekstu tylko wtedy, gdy klucz jeszcze nie istnieje, możesz użyć metody `addIf`:

```php
Context::add('key', 'first');

Context::get('key');
// "first"

Context::addIf('key', 'second');

Context::get('key');
// "first"
```

Kontekst zapewnia również wygodne metody do zwiększania lub zmniejszania danego klucza. Obie te metody akceptują co najmniej jeden argument: klucz do śledzenia. Drugi argument może być podany, aby określić wartość, o którą klucz powinien być zwiększony lub zmniejszony:

```php
Context::increment('records_added');
Context::increment('records_added', 5);

Context::decrement('records_added');
Context::decrement('records_added', 5);
```

<a name="conditional-context"></a>
#### Warunkowy kontekst

Metoda `when` może być użyta do dodania danych do kontekstu na podstawie danego warunku. Pierwsze zamknięcie dostarczone do metody `when` zostanie wywołane, jeśli dany warunek zostanie oceniony jako `true`, podczas gdy drugie zamknięcie zostanie wywołane, jeśli warunek zostanie oceniony jako `false`:

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Context;

Context::when(
    Auth::user()->isAdmin(),
    fn ($context) => $context->add('permissions', Auth::user()->permissions),
    fn ($context) => $context->add('permissions', []),
);
```

<a name="scoped-context"></a>
#### Kontekst z zakresem

Metoda `scope` zapewnia sposób na tymczasową modyfikację kontekstu podczas wykonywania danego wywołania zwrotnego i przywrócenie kontekstu do jego pierwotnego stanu, gdy wywołanie zwrotne zakończy wykonywanie. Dodatkowo, możesz przekazać dodatkowe dane, które powinny być scalone z kontekstem (jako drugi i trzeci argument) podczas wykonywania zamknięcia.

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Facades\Log;

Context::add('trace_id', 'abc-999');
Context::addHidden('user_id', 123);

Context::scope(
    function () {
        Context::add('action', 'adding_friend');

        $userId = Context::getHidden('user_id');

        Log::debug("Adding user [{$userId}] to friends list.");
        // Adding user [987] to friends list.  {"trace_id":"abc-999","user_name":"taylor_otwell","action":"adding_friend"}
    },
    data: ['user_name' => 'taylor_otwell'],
    hidden: ['user_id' => 987],
);

Context::all();
// [
//     'trace_id' => 'abc-999',
// ]

Context::allHidden();
// [
//     'user_id' => 123,
// ]
```

> [!WARNING]
> Jeśli obiekt w kontekście zostanie zmodyfikowany wewnątrz zamknięcia z zakresem, ta mutacja zostanie odzwierciedlona poza zakresem.

<a name="stacks"></a>
### Stosy

Kontekst oferuje możliwość tworzenia "stosów", które są listami danych przechowywanych w kolejności, w jakiej zostały dodane. Możesz dodać informacje do stosu, wywołując metodę `push`:

```php
use Illuminate\Support\Facades\Context;

Context::push('breadcrumbs', 'first_value');

Context::push('breadcrumbs', 'second_value', 'third_value');

Context::get('breadcrumbs');
// [
//     'first_value',
//     'second_value',
//     'third_value',
// ]
```

Stosy mogą być przydatne do przechwytywania historycznych informacji o żądaniu, takich jak zdarzenia zachodzące w całej aplikacji. Na przykład, możesz utworzyć słuchacza zdarzeń, który będzie dodawał do stosu za każdym razem, gdy wykonywane jest zapytanie, przechwytując SQL zapytania i czas trwania jako krotkę:

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Facades\DB;

// In AppServiceProvider.php...
DB::listen(function ($event) {
    Context::push('queries', [$event->time, $event->sql]);
});
```

Możesz określić, czy wartość znajduje się w stosie, używając metod `stackContains` i `hiddenStackContains`:

```php
if (Context::stackContains('breadcrumbs', 'first_value')) {
    //
}

if (Context::hiddenStackContains('secrets', 'first_value')) {
    //
}
```

Metody `stackContains` i `hiddenStackContains` akceptują również zamknięcie jako drugi argument, umożliwiając większą kontrolę nad operacją porównywania wartości:

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Str;

return Context::stackContains('breadcrumbs', function ($value) {
    return Str::startsWith($value, 'query_');
});
```

<a name="retrieving-context"></a>
## Pobieranie kontekstu

Możesz pobrać informacje z kontekstu używając metody `get` fasady `Context`:

```php
use Illuminate\Support\Facades\Context;

$value = Context::get('key');
```

Metody `only` i `except` mogą być użyte do pobrania podzbioru informacji z kontekstu:

```php
$data = Context::only(['first_key', 'second_key']);

$data = Context::except(['first_key']);
```

Metoda `pull` może być użyta do pobrania informacji z kontekstu i natychmiastowego usunięcia ich z kontekstu:

```php
$value = Context::pull('key');
```

Jeśli dane kontekstu są przechowywane w [stosie](#stacks), możesz usunąć elementy ze stosu używając metody `pop`:

```php
Context::push('breadcrumbs', 'first_value', 'second_value');

Context::pop('breadcrumbs');
// second_value

Context::get('breadcrumbs');
// ['first_value']
```

Metody `remember` i `rememberHidden` mogą być użyte do pobrania informacji z kontekstu, jednocześnie ustawiając wartość kontekstu na wartość zwróconą przez dane zamknięcie, jeśli żądane informacje nie istnieją:

```php
$permissions = Context::remember(
    'user-permissions',
    fn () => $user->permissions,
);
```

Jeśli chcesz pobrać wszystkie informacje przechowywane w kontekście, możesz wywołać metodę `all`:

```php
$data = Context::all();
```

<a name="determining-item-existence"></a>
### Określanie istnienia elementu

Możesz użyć metod `has` i `missing`, aby określić, czy kontekst ma jakąkolwiek wartość przechowywaną dla danego klucza:

```php
use Illuminate\Support\Facades\Context;

if (Context::has('key')) {
    // ...
}

if (Context::missing('key')) {
    // ...
}
```

Metoda `has` zwróci `true` niezależnie od przechowywanej wartości. Tak więc, na przykład, klucz z wartością `null` będzie uważany za obecny:

```php
Context::add('key', null);

Context::has('key');
// true
```

<a name="removing-context"></a>
## Usuwanie kontekstu

Metoda `forget` może być użyta do usunięcia klucza i jego wartości z bieżącego kontekstu:

```php
use Illuminate\Support\Facades\Context;

Context::add(['first_key' => 1, 'second_key' => 2]);

Context::forget('first_key');

Context::all();

// ['second_key' => 2]
```

Możesz zapomnieć kilka kluczy naraz, przekazując tablicę do metody `forget`:

```php
Context::forget(['first_key', 'second_key']);
```

<a name="hidden-context"></a>
## Ukryty kontekst

Kontekst oferuje możliwość przechowywania "ukrytych" danych. Te ukryte informacje nie są dołączane do logów i nie są dostępne za pomocą metod pobierania danych udokumentowanych powyżej. Kontekst zapewnia inny zestaw metod do interakcji z ukrytymi informacjami kontekstu:

```php
use Illuminate\Support\Facades\Context;

Context::addHidden('key', 'value');

Context::getHidden('key');
// 'value'

Context::get('key');
// null
```

Metody "ukryte" odzwierciedlają funkcjonalność metod nieukrytych udokumentowanych powyżej:

```php
Context::addHidden(/* ... */);
Context::addHiddenIf(/* ... */);
Context::pushHidden(/* ... */);
Context::getHidden(/* ... */);
Context::pullHidden(/* ... */);
Context::popHidden(/* ... */);
Context::onlyHidden(/* ... */);
Context::exceptHidden(/* ... */);
Context::allHidden(/* ... */);
Context::hasHidden(/* ... */);
Context::missingHidden(/* ... */);
Context::forgetHidden(/* ... */);
```

<a name="events"></a>
## Zdarzenia

Kontekst wysyła dwa zdarzenia, które pozwalają na podpięcie się do procesu nawadniania i odwadniania kontekstu.

Aby zilustrować, jak te zdarzenia mogą być używane, wyobraź sobie, że w middleware twojej aplikacji ustawiasz wartość konfiguracji `app.locale` na podstawie nagłówka `Accept-Language` przychodzącego żądania HTTP. Zdarzenia kontekstu pozwalają na przechwycenie tej wartości podczas żądania i przywrócenie jej w kolejce, zapewniając, że powiadomienia wysyłane w kolejce mają poprawną wartość `app.locale`. Możemy użyć zdarzeń kontekstu i [ukrytych](#hidden-context) danych, aby to osiągnąć, co zilustruje poniższa dokumentacja.

<a name="dehydrating"></a>
### Odwadnianie

Kiedy zadanie jest wysyłane do kolejki, dane w kontekście są "odwadniane" i przechwytywane wraz z ładunkiem zadania. Metoda `Context::dehydrating` pozwala zarejestrować zamknięcie, które zostanie wywołane podczas procesu odwadniania. W ramach tego zamknięcia możesz wprowadzić zmiany w danych, które będą udostępniane zadaniu w kolejce.

Typowo powinieneś zarejestrować wywołania zwrotne `dehydrating` w metodzie `boot` klasy `AppServiceProvider` twojej aplikacji:

```php
use Illuminate\Log\Context\Repository;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Context;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Context::dehydrating(function (Repository $context) {
        $context->addHidden('locale', Config::get('app.locale'));
    });
}
```

> [!NOTE]
> Nie powinieneś używać fasady `Context` w wywołaniu zwrotnym `dehydrating`, ponieważ to zmieni kontekst bieżącego procesu. Upewnij się, że wprowadzasz zmiany tylko w repozytorium przekazanym do wywołania zwrotnego.

<a name="hydrated"></a>
### Nawadnianie

Kiedy zadanie w kolejce zaczyna wykonywać się w kolejce, każdy kontekst, który został udostępniony zadaniu, zostanie "nawodniony" z powrotem do bieżącego kontekstu. Metoda `Context::hydrated` pozwala zarejestrować zamknięcie, które zostanie wywołane podczas procesu nawadniania.

Typowo powinieneś zarejestrować wywołania zwrotne `hydrated` w metodzie `boot` klasy `AppServiceProvider` twojej aplikacji:

```php
use Illuminate\Log\Context\Repository;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Context;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Context::hydrated(function (Repository $context) {
        if ($context->hasHidden('locale')) {
            Config::set('app.locale', $context->getHidden('locale'));
        }
    });
}
```

> [!NOTE]
> Nie powinieneś używać fasady `Context` w wywołaniu zwrotnym `hydrated`, a zamiast tego upewnij się, że wprowadzasz zmiany tylko w repozytorium przekazanym do wywołania zwrotnego.
