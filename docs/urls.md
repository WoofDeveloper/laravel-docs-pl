# Generowanie URL

- [Wprowadzenie](#introduction)
- [Podstawy](#the-basics)
    - [Generowanie URL-i](#generating-urls)
    - [Dostęp do Bieżącego URL](#accessing-the-current-url)
- [URL-e dla Nazwanych Tras](#urls-for-named-routes)
    - [Podpisane URL-e](#signed-urls)
- [URL-e dla Akcji Kontrolerów](#urls-for-controller-actions)
- [Fluent URI Objects](#fluent-uri-objects)
- [Wartości Domyślne](#default-values)

<a name="introduction"></a>
## Wprowadzenie

Laravel zapewnia kilka helperów, które pomagają w generowaniu URL-i dla twojej aplikacji. Te helpery są szczególnie przydatne podczas tworzenia linków w szablonach i odpowiedziach API lub podczas generowania odpowiedzi przekierowujących do innej części aplikacji.

<a name="the-basics"></a>
## Podstawy

<a name="generating-urls"></a>
### Generowanie URL-i

Helper `url` może być użyty do generowania dowolnych URL-i dla twojej aplikacji. Wygenerowany URL automatycznie użyje schematu (HTTP lub HTTPS) i hosta z bieżącego żądania obsługiwanego przez aplikację:

```php
$post = App\Models\Post::find(1);

echo url("/posts/{$post->id}");

// http://example.com/posts/1
```

Aby wygenerować URL z parametrami query string, możesz użyć metody `query`:

```php
echo url()->query('/posts', ['search' => 'Laravel']);

// https://example.com/posts?search=Laravel

echo url()->query('/posts?sort=latest', ['search' => 'Laravel']);

// http://example.com/posts?sort=latest&search=Laravel
```

Dostarczenie parametrów query string, które już istnieją w ścieżce, nadpisze ich istniejącą wartość:

```php
echo url()->query('/posts?sort=latest', ['sort' => 'oldest']);

// http://example.com/posts?sort=oldest
```

Tablice wartości mogą również być przekazywane jako parametry zapytania. Te wartości zostaną poprawnie przekształcone i zakodowane w wygenerowanym URL:

```php
echo $url = url()->query('/posts', ['columns' => ['title', 'body']]);

// http://example.com/posts?columns%5B0%5D=title&columns%5B1%5D=body

echo urldecode($url);

// http://example.com/posts?columns[0]=title&columns[1]=body
```

<a name="accessing-the-current-url"></a>
### Dostęp do Bieżącego URL

Jeśli nie podano ścieżki do helpera `url`, zwracana jest instancja `Illuminate\Routing\UrlGenerator`, umożliwiająca dostęp do informacji o bieżącym URL:

```php
// Pobierz bieżący URL bez query string...
echo url()->current();

// Pobierz bieżący URL wraz z query string...
echo url()->full();
```

Każda z tych metod może być również dostępna poprzez [fasadę](/docs/{{version}}/facades) `URL`:

```php
use Illuminate\Support\Facades\URL;

echo URL::current();
```

<a name="accessing-the-previous-url"></a>
#### Dostęp do Poprzedniego URL

Czasami przydatne jest znanie poprzedniego URL, z którego użytkownik przyszedł. Możesz uzyskać dostęp do poprzedniego URL za pomocą metod `previous` i `previousPath` helpera `url`:

```php
// Pobierz pełny URL dla poprzedniego żądania...
echo url()->previous();

// Pobierz ścieżkę dla poprzedniego żądania...
echo url()->previousPath();
```

Lub poprzez [sesję](/docs/{{version}}/session), możesz uzyskać dostęp do poprzedniego URL jako instancję [fluent URI](#fluent-uri-objects):

```php
use Illuminate\Http\Request;

Route::post('/users', function (Request $request) {
    $previousUri = $request->session()->previousUri();

    // ...
});
```

Możliwe jest również pobranie nazwy trasy dla poprzednio odwiedzonego URL poprzez sesję:

```php
$previousRoute = $request->session()->previousRoute();
```

<a name="urls-for-named-routes"></a>
## URL-e dla Nazwanych Tras

Helper `route` może być użyty do generowania URL-i dla [nazwanych tras](/docs/{{version}}/routing#named-routes). Nazwane trasy pozwalają generować URL-e bez wiązania się z rzeczywistym URL zdefiniowanym w trasie. W związku z tym, jeśli URL trasy się zmieni, nie trzeba wprowadzać zmian w wywołaniach funkcji `route`. Na przykład, wyobraź sobie, że twoja aplikacja zawiera trasę zdefiniowaną w następujący sposób:

```php
Route::get('/post/{post}', function (Post $post) {
    // ...
})->name('post.show');
```

Aby wygenerować URL do tej trasy, możesz użyć helpera `route` w następujący sposób:

```php
echo route('post.show', ['post' => 1]);

// http://example.com/post/1
```

Oczywiście, helper `route` może być również użyty do generowania URL-i dla tras z wieloma parametrami:

```php
Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
    // ...
})->name('comment.show');

echo route('comment.show', ['post' => 1, 'comment' => 3]);

// http://example.com/post/1/comment/3
```

Wszelkie dodatkowe elementy tablicy, które nie odpowiadają parametrom definicji trasy, zostaną dodane do query string URL:

```php
echo route('post.show', ['post' => 1, 'search' => 'rocket']);

// http://example.com/post/1?search=rocket
```

<a name="eloquent-models"></a>
#### Modele Eloquent

Często będziesz generować URL-e używając klucza trasy (zazwyczaj klucza głównego) [modeli Eloquent](/docs/{{version}}/eloquent). Z tego powodu możesz przekazać modele Eloquent jako wartości parametrów. Helper `route` automatycznie wyciągnie klucz trasy modelu:

```php
echo route('post.show', ['post' => $post]);
```

<a name="signed-urls"></a>
### Podpisane URL-e

Laravel pozwala łatwo tworzyć "podpisane" URL-e do nazwanych tras. Te URL-e mają dołączony hash "podpisu" do query string, który pozwala Laravel zweryfikować, że URL nie został zmodyfikowany od momentu utworzenia. Podpisane URL-e są szczególnie przydatne dla tras, które są publicznie dostępne, ale wymagają warstwy ochrony przed manipulacją URL.

Na przykład, możesz użyć podpisanych URL-i do implementacji publicznego linku "rezygnacja z subskrypcji", który jest wysyłany e-mailem do klientów. Aby utworzyć podpisany URL do nazwanej trasy, użyj metody `signedRoute` fasady `URL`:

```php
use Illuminate\Support\Facades\URL;

return URL::signedRoute('unsubscribe', ['user' => 1]);
```

Możesz wykluczyć domenę z hasha podpisanego URL, podając argument `absolute` do metody `signedRoute`:

```php
return URL::signedRoute('unsubscribe', ['user' => 1], absolute: false);
```

Jeśli chcesz wygenerować tymczasowy podpisany URL trasy, który wygasa po określonym czasie, możesz użyć metody `temporarySignedRoute`. Kiedy Laravel waliduje tymczasowy podpisany URL trasy, sprawdzi, czy znacznik czasu wygaśnięcia zakodowany w podpisanym URL nie upłynął:

```php
use Illuminate\Support\Facades\URL;

return URL::temporarySignedRoute(
    'unsubscribe', now()->plus(minutes: 30), ['user' => 1]
);
```

<a name="validating-signed-route-requests"></a>
#### Walidacja Podpisanych Żądań Trasy

Aby zweryfikować, że przychodzące żądanie ma ważny podpis, powinieś wywołać metodę `hasValidSignature` na przychodzącej instancji `Illuminate\Http\Request`:

```php
use Illuminate\Http\Request;

Route::get('/unsubscribe/{user}', function (Request $request) {
    if (! $request->hasValidSignature()) {
        abort(401);
    }

    // ...
})->name('unsubscribe');
```

Czasami możesz potrzebować, aby frontend twojej aplikacji mógł dodawać dane do podpisanego URL, na przykład podczas stronicowania po stronie klienta. Dlatego możesz określić parametry zapytania, które powinny być ignorowane podczas walidacji podpisanego URL, używając metody `hasValidSignatureWhileIgnoring`. Pamiętaj, że ignorowanie parametrów pozwala każdemu na modyfikację tych parametrów w żądaniu:

```php
if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
    abort(401);
}
```

Zamiast walidować podpisane URL-e używając instancji przychodzącego żądania, możesz przypisać [middleware](/docs/{{version}}/middleware) `signed` (`Illuminate\Routing\Middleware\ValidateSignature`) do trasy. Jeśli przychodzące żądanie nie ma ważnego podpisu, middleware automatycznie zwróci odpowiedź HTTP `403`:

```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed');
```

Jeśli twoje podpisane URL-e nie zawierają domeny w hashu URL, powinieneś przekazać argument `relative` do middleware:

```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed:relative');
```

<a name="responding-to-invalid-signed-routes"></a>
#### Odpowiadanie na Nieprawidłowe Podpisane Trasy

Kiedy ktoś odwiedza podpisany URL, który wygasł, otrzyma ogólną stronę błędu dla kodu statusu HTTP `403`. Możesz jednak dostosować to zachowanie, definiując własne zamknięcie "render" dla wyjątku `InvalidSignatureException` w pliku `bootstrap/app.php` twojej aplikacji:

```php
use Illuminate\Routing\Exceptions\InvalidSignatureException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->render(function (InvalidSignatureException $e) {
        return response()->view('errors.link-expired', status: 403);
    });
})
```

<a name="urls-for-controller-actions"></a>
## URL-e dla Akcji Kontrolerów

Funkcja `action` generuje URL dla podanej akcji kontrolera:

```php
use App\Http\Controllers\HomeController;

$url = action([HomeController::class, 'index']);
```

Jeśli metoda kontrolera akceptuje parametry trasy, możesz przekazać tablicę asocjacyjną parametrów trasy jako drugi argument funkcji:

```php
$url = action([UserController::class, 'profile'], ['id' => 1]);
```

<a name="fluent-uri-objects"></a>
## Fluent URI Objects

Klasa `Uri` w Laravel zapewnia wygodny i płynny interfejs do tworzenia i manipulowania URI poprzez obiekty. Ta klasa owija funkcjonalność dostarczane przez pakiet League URI i bezproblemowo integruje się z systemem routingu Laravel.

Możesz łatwo utworzyć instancję `Uri` używając metod statycznych:

```php
use App\Http\Controllers\UserController;
use App\Http\Controllers\InvokableController;
use Illuminate\Support\Uri;

// Wygeneruj instancję URI z podanego ciągu znaków...
$uri = Uri::of('https://example.com/path');

// Wygeneruj instancje URI do ścieżek, nazwanych tras lub akcji kontrolerów...
$uri = Uri::to('/dashboard');
$uri = Uri::route('users.show', ['user' => 1]);
$uri = Uri::signedRoute('users.show', ['user' => 1]);
$uri = Uri::temporarySignedRoute('user.index', now()->plus(minutes: 5));
$uri = Uri::action([UserController::class, 'index']);
$uri = Uri::action(InvokableController::class);

// Wygeneruj instancję URI z bieżącego URL żądania...
$uri = $request->uri();

// Wygeneruj instancję URI z URL poprzedniego żądania...
$uri = $request->session()->previousUri();
```

Po uzyskaniu instancji URI możesz ją płynnie modyfikować:

```php
$uri = Uri::of('https://example.com')
    ->withScheme('http')
    ->withHost('test.com')
    ->withPort(8000)
    ->withPath('/users')
    ->withQuery(['page' => 2])
    ->withFragment('section-1');
```

Więcej informacji na temat pracy z fluent URI objects znajdziesz w [dokumentacji URI](/docs/{{version}}/helpers#uri).

<a name="default-values"></a>
## Wartości Domyślne

Dla niektórych aplikacji możesz chcieć określić wartości domyślne dla całego żądania dla określonych parametrów URL. Na przykład wyobraź sobie, że wiele twoich tras definiuje parametr `{locale}`:

```php
Route::get('/{locale}/posts', function () {
    // ...
})->name('post.index');
```

Niepodręczne jest ciągłe przekazywanie `locale` za każdym razem, gdy wywołujesz helper `route`. Możesz więc użyć metody `URL::defaults` do zdefiniowania wartości domyślnej dla tego parametru, która będzie zawsze stosowana podczas bieżącego żądania. Możesz wywołać tę metodę z [middleware trasy](/docs/{{version}}/middleware#assigning-middleware-to-routes), aby mieć dostęp do bieżącego żądania:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\URL;
use Symfony\Component\HttpFoundation\Response;

class SetDefaultLocaleForUrls
{
    /**
     * Obsłuż przychodzące żądanie.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        URL::defaults(['locale' => $request->user()->locale]);

        return $next($request);
    }
}
```

Po ustawieniu wartości domyślnej dla parametru `locale` nie musisz już przekazywać jego wartości podczas generowania URL-i za pomocą helpera `route`.

<a name="url-defaults-middleware-priority"></a>
#### Wartości Domyślne URL i Priorytet Middleware

Ustawianie wartości domyślnych URL może kolidować z obsługą niejawnych wiązań modeli Laravel. Dlatego powinieś [nadać priorytet swoim middleware](/docs/{{version}}/middleware#sorting-middleware), które ustawiają wartości domyślne URL, aby były wykonywane przed własnym middleware `SubstituteBindings` Laravel. Możesz to osiągnąć, używając metody middleware `priority` w pliku `bootstrap/app.php` twojej aplikacji:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->prependToPriorityList(
        before: \Illuminate\Routing\Middleware\SubstituteBindings::class,
        prepend: \App\Http\Middleware\SetDefaultLocaleForUrls::class,
    );
})
```
