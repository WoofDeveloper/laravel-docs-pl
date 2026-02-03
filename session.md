# Sesja HTTP

- [Wprowadzenie](#introduction)
    - [Konfiguracja](#configuration)
    - [Wymagania wstępne sterowników](#driver-prerequisites)
- [Interakcja z sesją](#interacting-with-the-session)
    - [Pobieranie danych](#retrieving-data)
    - [Przechowywanie danych](#storing-data)
    - [Dane flash](#flash-data)
    - [Usuwanie danych](#deleting-data)
    - [Regenerowanie ID sesji](#regenerating-the-session-id)
- [Pamięć podręczna sesji](#session-cache)
- [Blokowanie sesji](#session-blocking)
- [Dodawanie niestandardowych sterowników sesji](#adding-custom-session-drivers)
    - [Implementacja sterownika](#implementing-the-driver)
    - [Rejestracja sterownika](#registering-the-driver)

<a name="introduction"></a>
## Wprowadzenie

Ponieważ aplikacje oparte na HTTP są bezstanowe, sesje zapewniają sposób przechowywania informacji o użytkowniku w wielu żądaniach. Te informacje o użytkowniku są zazwyczaj umieszczane w trwałym magazynie / backendzie, który może być dostępny z kolejnych żądań.

Laravel jest dostarczany z różnymi backendami sesji, do których dostęp odbywa się za pomocą wyrazistego, ujednoliconego API. Obsługa popularnych backendów takich jak [Memcached](https://memcached.org), [Redis](https://redis.io) i bazy danych jest zawarta.

<a name="configuration"></a>
### Konfiguracja

Plik konfiguracyjny sesji Twojej aplikacji jest przechowywany w `config/session.php`. Upewnij się, że przejrzałeś opcje dostępne w tym pliku. Domyślnie Laravel jest skonfigurowany do używania sterownika sesji `database`.

Opcja konfiguracyjna `driver` sesji definiuje, gdzie dane sesji będą przechowywane dla każdego żądania. Laravel zawiera różne sterowniki:

<div class="content-list" markdown="1">

- `file` - sesje są przechowywane w `storage/framework/sessions`.
- `cookie` - sesje są przechowywane w bezpiecznych, zaszyfrowanych ciasteczkach.
- `database` - sesje są przechowywane w relacyjnej bazie danych.
- `memcached` / `redis` - sesje są przechowywane w jednym z tych szybkich magazynów opartych na pamięci podręcznej.
- `dynamodb` - sesje są przechowywane w AWS DynamoDB.
- `array` - sesje są przechowywane w tablicy PHP i nie będą utrwalane.

</div>

> [!NOTE]
> Sterownik array jest używany głównie podczas [testowania](/docs/{{version}}/testing) i zapobiega utrwalaniu danych przechowywanych w sesji.

<a name="driver-prerequisites"></a>
### Wymagania wstępne sterowników

<a name="database"></a>
#### Baza danych

Podczas używania sterownika sesji `database`, musisz upewnić się, że masz tabelę bazy danych do przechowywania danych sesji. Zazwyczaj jest to zawarte w domyślnej [migracji bazy danych](/docs/{{version}}/migrations) Laravel `0001_01_01_000000_create_users_table.php`; jednak, jeśli z jakiegoś powodu nie masz tabeli `sessions`, możesz użyć polecenia Artisan `make:session-table`, aby wygenerować tę migrację:

```shell
php artisan make:session-table

php artisan migrate
```

<a name="redis"></a>
#### Redis

Przed używaniem sesji Redis z Laravel, musisz zainstalować rozszerzenie PHP PhpRedis poprzez PECL lub zainstalować pakiet `predis/predis` (~1.0) poprzez Composer. Aby uzyskać więcej informacji o konfiguracji Redis, zapoznaj się z [dokumentacją Redis](/docs/{{version}}/redis#configuration) Laravel.

> [!NOTE]
> Zmienna środowiskowa `SESSION_CONNECTION` lub opcja `connection` w pliku konfiguracyjnym `session.php` może być użyta do określenia, które połączenie Redis jest używane do przechowywania sesji.

<a name="interacting-with-the-session"></a>
## Interakcja z sesją

<a name="retrieving-data"></a>
### Pobieranie danych

Istnieją dwa podstawowe sposoby pracy z danymi sesji w Laravel: globalny helper `session` oraz poprzez instancję `Request`. Najpierw przyjrzyjmy się dostępowi do sesji poprzez instancję `Request`, która może być type-hinted na zamknięciu trasy lub metodzie kontrolera. Pamiętaj, że zależności metody kontrolera są automatycznie wstrzykiwane przez [kontener usług](/docs/{{version}}/container) Laravel:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     */
    public function show(Request $request, string $id): View
    {
        $value = $request->session()->get('key');

        // ...

        $user = $this->users->find($id);

        return view('user.profile', ['user' => $user]);
    }
}
```

Podczas pobierania elementu z sesji, możesz również przekazać domyślną wartość jako drugi argument do metody `get`. Ta domyślna wartość zostanie zwrócona, jeśli określony klucz nie istnieje w sesji. Jeśli przekażesz zamknięcie jako domyślną wartość do metody `get` i żądany klucz nie istnieje, zamknięcie zostanie wykonane, a jego wynik zwrócony:

```php
$value = $request->session()->get('key', 'default');

$value = $request->session()->get('key', function () {
    return 'default';
});
```

<a name="the-global-session-helper"></a>
#### Globalny helper sesji

Możesz również użyć globalnej funkcji PHP `session` do pobierania i przechowywania danych w sesji. Gdy helper `session` jest wywoływany z pojedynczym argumentem typu string, zwróci wartość tego klucza sesji. Gdy helper jest wywoływany z tablicą par klucz / wartość, te wartości będą przechowywane w sesji:

```php
Route::get('/home', function () {
    // Retrieve a piece of data from the session...
    $value = session('key');

    // Specifying a default value...
    $value = session('key', 'default');

    // Store a piece of data in the session...
    session(['key' => 'value']);
});
```

> [!NOTE]
> Istnieje niewielka praktyczna różnica między używaniem sesji poprzez instancję żądania HTTP a używaniem globalnego helpera `session`. Obie metody są [testowalne](/docs/{{version}}/testing) poprzez metodę `assertSessionHas`, która jest dostępna we wszystkich przypadkach testowych.

<a name="retrieving-all-session-data"></a>
#### Pobieranie wszystkich danych sesji

Jeśli chcesz pobrać wszystkie dane w sesji, możesz użyć metody `all`:

```php
$data = $request->session()->all();
```

<a name="retrieving-a-portion-of-the-session-data"></a>
#### Pobieranie części danych sesji

Metody `only` i `except` mogą być użyte do pobierania podzbioru danych sesji:

```php
$data = $request->session()->only(['username', 'email']);

$data = $request->session()->except(['username', 'email']);
```

<a name="determining-if-an-item-exists-in-the-session"></a>
#### Sprawdzanie, czy element istnieje w sesji

Aby sprawdzić, czy element jest obecny w sesji, możesz użyć metody `has`. Metoda `has` zwraca `true`, jeśli element jest obecny i nie jest `null`:

```php
if ($request->session()->has('users')) {
    // ...
}
```

Aby sprawdzić, czy element jest obecny w sesji, nawet jeśli jego wartość to `null`, możesz użyć metody `exists`:

```php
if ($request->session()->exists('users')) {
    // ...
}
```

Aby sprawdzić, czy element nie jest obecny w sesji, możesz użyć metody `missing`. Metoda `missing` zwraca `true`, jeśli element nie jest obecny:

```php
if ($request->session()->missing('users')) {
    // ...
}
```

<a name="storing-data"></a>
### Przechowywanie danych

Aby przechowywać dane w sesji, zazwyczaj użyjesz metody `put` instancji żądania lub globalnego helpera `session`:

```php
// Via a request instance...
$request->session()->put('key', 'value');

// Via the global "session" helper...
session(['key' => 'value']);
```

<a name="pushing-to-array-session-values"></a>
#### Dodawanie do tablicowych wartości sesji

Metoda `push` może być użyta do dodania nowej wartości do wartości sesji, która jest tablicą. Na przykład, jeśli klucz `user.teams` zawiera tablicę nazw zespołów, możesz dodać nową wartość do tablicy w następujący sposób:

```php
$request->session()->push('user.teams', 'developers');
```

<a name="retrieving-deleting-an-item"></a>
#### Pobieranie i usuwanie elementu

Metoda `pull` pobierze i usunie element z sesji w jednej instrukcji:

```php
$value = $request->session()->pull('key', 'default');
```

<a name="incrementing-and-decrementing-session-values"></a>
#### Inkrementacja i dekrementacja wartości sesji

Jeśli dane sesji zawierają liczbę całkowitą, którą chcesz zwiększyć lub zmniejszyć, możesz użyć metod `increment` i `decrement`:

```php
$request->session()->increment('count');

$request->session()->increment('count', $incrementBy = 2);

$request->session()->decrement('count');

$request->session()->decrement('count', $decrementBy = 2);
```

<a name="flash-data"></a>
### Dane flash

Niekiedy możesz chcieć przechowywać elementy w sesji tylko dla następnego żądania. Możesz to zrobić używając metody `flash`. Dane przechowywane w sesji przy użyciu tej metody będą dostępne natychmiast i podczas kolejnego żądania HTTP. Po kolejnym żądaniu HTTP dane flash zostaną usunięte. Dane flash są szczególnie przydatne dla krótkotrwałych komunikatów statusu:

```php
$request->session()->flash('status', 'Task was successful!');
```

Jeśli musisz zachować dane flash dla kilku żądań, możesz użyć metody `reflash`, która zachowa wszystkie dane flash dla dodatkowego żądania. Jeśli musisz zachować tylko określone dane flash, możesz użyć metody `keep`:

```php
$request->session()->reflash();

$request->session()->keep(['username', 'email']);
```

Aby zachować dane flash tylko dla bieżącego żądania, możesz użyć metody `now`:

```php
$request->session()->now('status', 'Task was successful!');
```

<a name="deleting-data"></a>
### Usuwanie danych

Metoda `forget` usunie fragment danych z sesji. Jeśli chcesz usunąć wszystkie dane z sesji, możesz użyć metody `flush`:

```php
// Forget a single key...
$request->session()->forget('name');

// Forget multiple keys...
$request->session()->forget(['name', 'status']);

$request->session()->flush();
```

<a name="regenerating-the-session-id"></a>
### Regenerowanie ID sesji

Regenerowanie ID sesji jest często wykonywane w celu zapobiegania wykorzystywaniu przez złośliwych użytkowników ataku [session fixation](https://owasp.org/www-community/attacks/Session_fixation) na Twoją aplikację.

Laravel automatycznie regeneruje ID sesji podczas uwierzytelniania, jeśli używasz jednego z [zestawów startowych aplikacji](/docs/{{version}}/starter-kits) Laravel lub [Laravel Fortify](/docs/{{version}}/fortify); jednak, jeśli musisz ręcznie zregenerować ID sesji, możesz użyć metody `regenerate`:

```php
$request->session()->regenerate();
```

Jeśli musisz zregenerować ID sesji i usunąć wszystkie dane z sesji w jednej instrukcji, możesz użyć metody `invalidate`:

```php
$request->session()->invalidate();
```

<a name="session-cache"></a>
## Pamięć podręczna sesji

Pamięć podręczna sesji Laravel zapewnia wygodny sposób buforowania danych, które są ograniczone do indywidualnej sesji użytkownika. W przeciwieństwie do globalnej pamięci podręcznej aplikacji, dane pamięci podręcznej sesji są automatycznie izolowane na sesję i są czyszczone, gdy sesja wygasa lub zostaje zniszczona. Pamięć podręczna sesji obsługuje wszystkie znane [metody pamięci podręcznej Laravel](/docs/{{version}}/cache), takie jak `get`, `put`, `remember`, `forget` i więcej, ale ograniczone do bieżącej sesji.

Pamięć podręczna sesji jest idealna do przechowywania tymczasowych, specyficznych dla użytkownika danych, które chcesz zachować w wielu żądaniach w ramach tej samej sesji, ale nie musisz przechowywać na stałe. Obejmuje to takie rzeczy jak dane formularza, tymczasowe obliczenia, odpowiedzi API lub wszelkie inne efemeryczne dane, które powinny być powiązane z sesją określonego użytkownika.

Możesz uzyskać dostęp do pamięci podręcznej sesji poprzez metodę `cache` na sesji:

```php
$discount = $request->session()->cache()->get('discount');

$request->session()->cache()->put(
    'discount', 10, now()->plus(minutes: 5)
);
```

Aby uzyskać więcej informacji na temat metod pamięci podręcznej Laravel, zapoznaj się z [dokumentacją pamięci podręcznej](/docs/{{version}}/cache).

<a name="session-blocking"></a>
## Blokowanie sesji

> [!WARNING]
> Aby korzystać z blokowania sesji, Twoja aplikacja musi używać sterownika pamięci podręcznej, który obsługuje [blokady atomowe](/docs/{{version}}/cache#atomic-locks). Obecnie te sterowniki pamięci podręcznej obejmują sterowniki `memcached`, `dynamodb`, `redis`, `mongodb` (zawarty w oficjalnym pakiecie `mongodb/laravel-mongodb`), `database`, `file` i `array`. Dodatkowo nie możesz używać sterownika sesji `cookie`.

Domyślnie Laravel pozwala na równoczesne wykonywanie żądań używających tej samej sesji. Tak więc, na przykład, jeśli użyjesz biblioteki HTTP JavaScript do wykonania dwóch żądań HTTP do Twojej aplikacji, oba będą wykonywane w tym samym czasie. Dla wielu aplikacji nie stanowi to problemu; jednak utrata danych sesji może wystąpić w niewielkim podzbiorze aplikacji, które wykonują równoczesne żądania do dwóch różnych punktów końcowych aplikacji, które oba zapisują dane do sesji.

Aby temu zapobiec, Laravel zapewnia funkcjonalność, która pozwala ograniczyć równoczesne żądania dla danej sesji. Aby rozpocząć, możesz po prostu połączyć łańcuchowo metodę `block` z definicją trasy. W tym przykładzie przychodzące żądanie do punktu końcowego `/profile` uzyska blokadę sesji. Podczas gdy ta blokada jest utrzymywana, wszelkie przychodzące żądania do punktów końcowych `/profile` lub `/order`, które współdzielą ten sam identyfikator sesji, będą czekać na zakończenie wykonania pierwszego żądania przed kontynuowaniem swojego wykonania:

```php
Route::post('/profile', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);

Route::post('/order', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);
```

Metoda `block` akceptuje dwa opcjonalne argumenty. Pierwszy argument akceptowany przez metodę `block` to maksymalna liczba sekund, przez jaką blokada sesji powinna być utrzymywana przed jej zwolnieniem. Oczywiście, jeśli żądanie zakończy się przed tym czasem, blokada zostanie zwolniona wcześniej.

Drugi argument akceptowany przez metodę `block` to liczba sekund, przez jaką żądanie powinno czekać podczas próby uzyskania blokady sesji. Wyjątek `Illuminate\Contracts\Cache\LockTimeoutException` zostanie zgłoszony, jeśli żądanie nie będzie w stanie uzyskać blokady sesji w określonej liczbie sekund.

Jeśli żaden z tych argumentów nie zostanie przekazany, blokada zostanie uzyskana maksymalnie na 10 sekund, a żądania będą czekać maksymalnie 10 sekund podczas próby uzyskania blokady:

```php
Route::post('/profile', function () {
    // ...
})->block();
```

<a name="adding-custom-session-drivers"></a>
## Dodawanie niestandardowych sterowników sesji

<a name="implementing-the-driver"></a>
### Implementacja sterownika

Jeśli żaden z istniejących sterowników sesji nie pasuje do potrzeb Twojej aplikacji, Laravel umożliwia napisanie własnego handlera sesji. Twój niestandardowy sterownik sesji powinien implementować wbudowany w PHP interfejs `SessionHandlerInterface`. Ten interfejs zawiera tylko kilka prostych metod. Szkieletowa implementacja MongoDB wygląda następująco:

```php
<?php

namespace App\Extensions;

class MongoSessionHandler implements \SessionHandlerInterface
{
    public function open($savePath, $sessionName) {}
    public function close() {}
    public function read($sessionId) {}
    public function write($sessionId, $data) {}
    public function destroy($sessionId) {}
    public function gc($lifetime) {}
}
```

Ponieważ Laravel nie zawiera domyślnego katalogu do przechowywania Twoich rozszerzeń, możesz umieścić je w dowolnym miejscu. W tym przykładzie utworzyliśmy katalog `Extensions` do przechowywania `MongoSessionHandler`.

Ponieważ cel tych metod nie jest łatwo zrozumiały, oto przegląd celu każdej metody:

<div class="content-list" markdown="1">

- Metoda `open` zazwyczaj byłaby używana w systemach przechowywania sesji opartych na plikach. Ponieważ Laravel jest dostarczany ze sterownikiem sesji `file`, rzadko będziesz potrzebować umieścić cokolwiek w tej metodzie. Możesz po prostu pozostawić tę metodę pustą.
- Metoda `close`, podobnie jak metoda `open`, również zazwyczaj może być pominięta. Dla większości sterowników nie jest potrzebna.
- Metoda `read` powinna zwrócić wersję string danych sesji związanych z danym `$sessionId`. Nie ma potrzeby wykonywania jakiejkolwiek serializacji lub innego kodowania podczas pobierania lub przechowywania danych sesji w Twoim sterowniku, ponieważ Laravel wykona serializację za Ciebie.
- Metoda `write` powinna zapisać dany string `$data` związany z `$sessionId` do jakiegoś trwałego systemu przechowywania, takiego jak MongoDB lub inny system przechowywania Twojego wyboru. Ponownie, nie powinieneś wykonywać żadnej serializacji - Laravel już to dla Ciebie obsłużył.
- Metoda `destroy` powinna usunąć dane związane z `$sessionId` z trwałego przechowywania.
- Metoda `gc` powinna zniszczyć wszystkie dane sesji, które są starsze niż dany `$lifetime`, który jest znacznikiem czasu UNIX. Dla systemów z samouwolnieniem, takich jak Memcached i Redis, ta metoda może pozostać pusta.

</div>

<a name="registering-the-driver"></a>
### Rejestracja sterownika

Po zaimplementowaniu sterownika, jesteś gotowy do zarejestrowania go w Laravel. Aby dodać dodatkowe sterowniki do backendu sesji Laravel, możesz użyć metody `extend` dostarczonej przez [fasadę](/docs/{{version}}/facades) `Session`. Powinieneś wywołać metodę `extend` z metody `boot` [dostawcy usług](/docs/{{version}}/providers). Możesz to zrobić z istniejącego `App\Providers\AppServiceProvider` lub utworzyć zupełnie nowego dostawcę:

```php
<?php

namespace App\Providers;

use App\Extensions\MongoSessionHandler;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Session;
use Illuminate\Support\ServiceProvider;

class SessionServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Session::extend('mongo', function (Application $app) {
            // Return an implementation of SessionHandlerInterface...
            return new MongoSessionHandler;
        });
    }
}
```

Po zarejestrowaniu sterownika sesji, możesz określić sterownik `mongo` jako sterownik sesji Twojej aplikacji, używając zmiennej środowiskowej `SESSION_DRIVER` lub w pliku konfiguracyjnym aplikacji `config/session.php`.
