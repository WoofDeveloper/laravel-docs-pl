# Middleware

- [Wprowadzenie](#introduction)
- [Definiowanie Middleware](#defining-middleware)
- [Rejestrowanie Middleware](#registering-middleware)
    - [Globalne Middleware](#global-middleware)
    - [Przypisywanie Middleware do Tras](#assigning-middleware-to-routes)
    - [Grupy Middleware](#middleware-groups)
    - [Aliasy Middleware](#middleware-aliases)
    - [Sortowanie Middleware](#sorting-middleware)
- [Parametry Middleware](#middleware-parameters)
- [Terminalny Middleware](#terminable-middleware)

<a name="introduction"></a>
## Wprowadzenie

Middleware zapewniają wygodny mechanizm do inspekcji i filtrowania żądań HTTP wchodzących do aplikacji. Na przykład Laravel zawiera middleware, który weryfikuje, czy użytkownik aplikacji jest uwierzytelniony. Jeśli użytkownik nie jest uwierzytelniony, middleware przekieruje użytkownika do ekranu logowania aplikacji. Jednak jeśli użytkownik jest uwierzytelniony, middleware pozwoli żądaniu przejść dalej do aplikacji.

Dodatkowe middleware mogą być napisane w celu wykonywania różnych zadań poza uwierzytelnianiem. Na przykład middleware logowania może logować wszystkie przychodzące żądania do aplikacji. Różnorodne middleware są zawarte w Laravel, w tym middleware do uwierzytelniania i ochrony CSRF; jednak wszystkie middleware zdefiniowane przez użytkownika są zazwyczaj zlokalizowane w katalogu `app/Http/Middleware` aplikacji.

<a name="defining-middleware"></a>
## Definiowanie Middleware

Aby utworzyć nowy middleware, użyj polecenia Artisan `make:middleware`:

```shell
php artisan make:middleware EnsureTokenIsValid
```

To polecenie umieści nową klasę `EnsureTokenIsValid` w katalogu `app/Http/Middleware` aplikacji. W tym middleware pozwolimy na dostęp do trasy tylko wtedy, gdy dostarczony parametr `token` będzie pasował do określonej wartości. W przeciwnym razie przekierujemy użytkowników z powrotem do URI `/home`:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureTokenIsValid
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->input('token') !== 'my-secret-token') {
            return redirect('/home');
        }

        return $next($request);
    }
}
```

Jak widać, jeśli dany `token` nie pasuje do naszego sekretnego tokenu, middleware zwróci przekierowanie HTTP do klienta; w przeciwnym razie żądanie zostanie przekazane dalej do aplikacji. Aby przekazać żądanie głębiej do aplikacji (pozwalając middleware "przejść"), należy wywołać callback `$next` z `$request`.

Najlepiej wyobrazić sobie middleware jako serię "warstw", przez które żądania HTTP muszą przejść, zanim trafią do aplikacji. Każda warstwa może zbadać żądanie, a nawet całkowicie je odrzucić.

> [!NOTE]
> Wszystkie middleware są rozwiązywane przez [kontener usług](/docs/{{version}}/container), więc możesz określić typ dowolnych zależności, których potrzebujesz w konstruktorze middleware.

<a name="middleware-and-responses"></a>
#### Middleware i Odpowiedzi

Oczywiście middleware może wykonywać zadania przed lub po przekazaniu żądania głębiej do aplikacji. Na przykład poniższy middleware wykona pewne zadanie **przed** obsłużeniem żądania przez aplikację:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class BeforeMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        // Perform action

        return $next($request);
    }
}
```

Jednakże ten middleware wykona swoje zadanie **po** obsłużeniu żądania przez aplikację:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class AfterMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        $response = $next($request);

        // Perform action

        return $response;
    }
}
```

<a name="registering-middleware"></a>
## Rejestrowanie Middleware

<a name="global-middleware"></a>
### Globalne Middleware

Jeśli chcesz, aby middleware działał podczas każdego żądania HTTP do aplikacji, możesz dodać go do globalnego stosu middleware w pliku `bootstrap/app.php` aplikacji:

```php
use App\Http\Middleware\EnsureTokenIsValid;

->withMiddleware(function (Middleware $middleware): void {
     $middleware->append(EnsureTokenIsValid::class);
})
```

Obiekt `$middleware` dostarczony do zamknięcia `withMiddleware` jest instancją `Illuminate\Foundation\Configuration\Middleware` i jest odpowiedzialny za zarządzanie middleware przypisanym do tras aplikacji. Metoda `append` dodaje middleware na końcu listy globalnych middleware. Jeśli chcesz dodać middleware na początku listy, powinieneś użyć metody `prepend`.

<a name="manually-managing-laravels-default-global-middleware"></a>
#### Ręczne Zarządzanie Domyślnymi Globalnymi Middleware Laravel

Jeśli chcesz ręcznie zarządzać globalnym stosem middleware Laravel, możesz dostarczyć domyślny stos globalnych middleware Laravel do metody `use`. Następnie możesz dostosować domyślny stos middleware według potrzeb:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->use([
        \Illuminate\Foundation\Http\Middleware\InvokeDeferredCallbacks::class,
        // \Illuminate\Http\Middleware\TrustHosts::class,
        \Illuminate\Http\Middleware\TrustProxies::class,
        \Illuminate\Http\Middleware\HandleCors::class,
        \Illuminate\Foundation\Http\Middleware\PreventRequestsDuringMaintenance::class,
        \Illuminate\Http\Middleware\ValidatePostSize::class,
        \Illuminate\Foundation\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
    ]);
})
```

<a name="assigning-middleware-to-routes"></a>
### Przypisywanie Middleware do Tras

Jeśli chcesz przypisać middleware do określonych tras, możesz wywołać metodę `middleware` podczas definiowania trasy:

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::get('/profile', function () {
    // ...
})->middleware(EnsureTokenIsValid::class);
```

Możesz przypisać wiele middleware do trasy, przekazując tablicę nazw middleware do metody `middleware`:

```php
Route::get('/', function () {
    // ...
})->middleware([First::class, Second::class]);
```

<a name="excluding-middleware"></a>
#### Wykluczanie Middleware

Podczas przypisywania middleware do grupy tras, czasami może być konieczne zapobieżenie zastosowaniu middleware do pojedynczej trasy w grupie. Możesz to osiągnąć za pomocą metody `withoutMiddleware`:

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::middleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/', function () {
        // ...
    });

    Route::get('/profile', function () {
        // ...
    })->withoutMiddleware([EnsureTokenIsValid::class]);
});
```

Możesz również wykluczyć dany zestaw middleware z całej [grupy](/docs/{{version}}/routing#route-groups) definicji tras:

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/profile', function () {
        // ...
    });
});
```

Metoda `withoutMiddleware` może usuwać tylko middleware tras i nie ma zastosowania do [globalnych middleware](#global-middleware).

<a name="middleware-groups"></a>
### Grupy Middleware

Czasami możesz chcieć zgrupować kilka middleware pod jedną nazwą, aby łatwiej było je przypisać do tras. Możesz to osiągnąć za pomocą metody `appendToGroup` w pliku `bootstrap/app.php` aplikacji:

```php
use App\Http\Middleware\First;
use App\Http\Middleware\Second;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->appendToGroup('group-name', [
        First::class,
        Second::class,
    ]);

    $middleware->prependToGroup('group-name', [
        First::class,
        Second::class,
    ]);
})
```

Grupy middleware mogą być przypisane do tras i akcji kontrolera przy użyciu tej samej składni co pojedyncze middleware:

```php
Route::get('/', function () {
    // ...
})->middleware('group-name');

Route::middleware(['group-name'])->group(function () {
    // ...
});
```

<a name="laravels-default-middleware-groups"></a>
#### Domyślne Grupy Middleware Laravel

Laravel zawiera predefiniowane grupy middleware `web` i `api`, które zawierają popularne middleware, które możesz chcieć zastosować do swoich tras web i API. Pamiętaj, że Laravel automatycznie stosuje te grupy middleware do odpowiadających plików `routes/web.php` i `routes/api.php`:

<div class="overflow-auto">

| The `web` Middleware Group                                |
| --------------------------------------------------------- |
| `Illuminate\Cookie\Middleware\EncryptCookies`             |
| `Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse` |
| `Illuminate\Session\Middleware\StartSession`              |
| `Illuminate\View\Middleware\ShareErrorsFromSession`       |
| `Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` |
| `Illuminate\Routing\Middleware\SubstituteBindings`        |

</div>

<div class="overflow-auto">

| The `api` Middleware Group                         |
| -------------------------------------------------- |
| `Illuminate\Routing\Middleware\SubstituteBindings` |

</div>

Jeśli chcesz dodać lub poprzedzić middleware do tych grup, możesz użyć metod `web` i `api` w pliku `bootstrap/app.php` aplikacji. Metody `web` i `api` są wygodnymi alternatywami dla metody `appendToGroup`:

```php
use App\Http\Middleware\EnsureTokenIsValid;
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->web(append: [
        EnsureUserIsSubscribed::class,
    ]);

    $middleware->api(prepend: [
        EnsureTokenIsValid::class,
    ]);
})
```

Możesz nawet zamienić jeden z domyślnych wpisów grupy middleware Laravel na własny middleware:

```php
use App\Http\Middleware\StartCustomSession;
use Illuminate\Session\Middleware\StartSession;

$middleware->web(replace: [
    StartSession::class => StartCustomSession::class,
]);
```

Możesz również całkowicie usunąć middleware:

```php
$middleware->web(remove: [
    StartSession::class,
]);
```

<a name="manually-managing-laravels-default-middleware-groups"></a>
#### Ręczne Zarządzanie Domyślnymi Grupami Middleware Laravel

Jeśli chcesz ręcznie zarządzać wszystkimi middleware w domyślnych grupach middleware `web` i `api` Laravel, możesz całkowicie przedefiniować grupy. Poniższy przykład zdefiniuje grupy middleware `web` i `api` z ich domyślnymi middleware, pozwalając na ich dostosowanie według potrzeb:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->group('web', [
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        // \Illuminate\Session\Middleware\AuthenticateSession::class,
    ]);

    $middleware->group('api', [
        // \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        // 'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ]);
})
```

> [!NOTE]
> Domyślnie grupy middleware `web` i `api` są automatycznie stosowane do odpowiadających plików `routes/web.php` i `routes/api.php` aplikacji przez plik `bootstrap/app.php`.

<a name="middleware-aliases"></a>
### Aliasy Middleware

Możesz przypisać aliasy do middleware w pliku `bootstrap/app.php` aplikacji. Aliasy middleware pozwalają zdefiniować krótki alias dla danej klasy middleware, co może być szczególnie przydatne dla middleware z długimi nazwami klas:

```php
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'subscribed' => EnsureUserIsSubscribed::class
    ]);
})
```

Po zdefiniowaniu aliasu middleware w pliku `bootstrap/app.php` aplikacji, możesz użyć aliasu podczas przypisywania middleware do tras:

```php
Route::get('/profile', function () {
    // ...
})->middleware('subscribed');
```

Dla wygody, niektóre wbudowane middleware Laravel są domyślnie aliasowane. Na przykład middleware `auth` jest aliasem dla middleware `Illuminate\Auth\Middleware\Authenticate`. Poniżej znajduje się lista domyślnych aliasów middleware:

<div class="overflow-auto">

| Alias              | Middleware                                                                                                    |
| ------------------ | ------------------------------------------------------------------------------------------------------------- |
| `auth`             | `Illuminate\Auth\Middleware\Authenticate`                                                                     |
| `auth.basic`       | `Illuminate\Auth\Middleware\AuthenticateWithBasicAuth`                                                        |
| `auth.session`     | `Illuminate\Session\Middleware\AuthenticateSession`                                                           |
| `cache.headers`    | `Illuminate\Http\Middleware\SetCacheHeaders`                                                                  |
| `can`              | `Illuminate\Auth\Middleware\Authorize`                                                                        |
| `guest`            | `Illuminate\Auth\Middleware\RedirectIfAuthenticated`                                                          |
| `password.confirm` | `Illuminate\Auth\Middleware\RequirePassword`                                                                  |
| `precognitive`     | `Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests`                                            |
| `signed`           | `Illuminate\Routing\Middleware\ValidateSignature`                                                             |
| `subscribed`       | `\Spark\Http\Middleware\VerifyBillableIsSubscribed`                                                           |
| `throttle`         | `Illuminate\Routing\Middleware\ThrottleRequests` or `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` |
| `verified`         | `Illuminate\Auth\Middleware\EnsureEmailIsVerified`                                                            |

</div>

<a name="sorting-middleware"></a>
### Sortowanie Middleware

Rzadko może być potrzebne, aby middleware wykonywał się w określonej kolejności, ale nie masz kontroli nad ich kolejnością, gdy są przypisywane do trasy. W takich sytuacjach możesz określić priorytet middleware za pomocą metody `priority` w pliku `bootstrap/app.php` aplikacji:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->priority([
        \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ]);
})
```

<a name="middleware-parameters"></a>
## Parametry Middleware

Middleware może również otrzymywać dodatkowe parametry. Na przykład, jeśli aplikacja musi zweryfikować, że uwierzytelniony użytkownik posiada daną "rolę" przed wykonaniem określonej akcji, możesz utworzyć middleware `EnsureUserHasRole`, które otrzymuje nazwę roli jako dodatkowy argument.

Dodatkowe parametry middleware zostaną przekazane do middleware po argumencie `$next`:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserHasRole
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next, string $role): Response
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }

        return $next($request);
    }
}
```

Parametry middleware mogą być określone podczas definiowania trasy poprzez oddzielenie nazwy middleware i parametrów znakiem `:`:

```php
use App\Http\Middleware\EnsureUserHasRole;

Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware(EnsureUserHasRole::class.':editor');
```

Wiele parametrów może być rozdzielonych przecinkami:

```php
Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware(EnsureUserHasRole::class.':editor,publisher');
```

<a name="terminable-middleware"></a>
## Terminalny Middleware

Czasami middleware może potrzebować wykonać pewną pracę po wysłaniu odpowiedzi HTTP do przeglądarki. Jeśli zdefiniujesz metodę `terminate` w swoim middleware, a twój serwer webowy używa [FastCGI](https://www.php.net/manual/en/install.fpm.php), metoda `terminate` zostanie automatycznie wywołana po wysłaniu odpowiedzi do przeglądarki:

```php
<?php

namespace Illuminate\Session\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class TerminatingMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        return $next($request);
    }

    /**
     * Handle tasks after the response has been sent to the browser.
     */
    public function terminate(Request $request, Response $response): void
    {
        // ...
    }
}
```

Metoda `terminate` powinna otrzymać zarówno żądanie, jak i odpowiedź. Po zdefiniowaniu terminalnego middleware, powinieneś dodać go do listy tras lub globalnych middleware w pliku `bootstrap/app.php` aplikacji.

Podczas wywoływania metody `terminate` w middleware, Laravel rozwiąże nową instancję middleware z [kontenera usług](/docs/{{version}}/container). Jeśli chcesz użyć tej samej instancji middleware, gdy wywoływane są metody `handle` i `terminate`, zarejestruj middleware w kontenerze za pomocą metody `singleton` kontenera. Zazwyczaj powinno to być wykonane w metodzie `register` twojego `AppServiceProvider`:

```php
use App\Http\Middleware\TerminatingMiddleware;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(TerminatingMiddleware::class);
}
```
