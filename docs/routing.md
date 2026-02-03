# Routing

- [Podstawy Routingu](#basic-routing)
    - [Domyślne Pliki Routów](#the-default-route-files)
    - [Route Przekierowań](#redirect-routes)
    - [Route Widoków](#view-routes)
    - [Wyświetlanie Listy Routów](#listing-your-routes)
    - [Dostosowanie Routingu](#routing-customization)
- [Parametry Route](#route-parameters)
    - [Parametry Wymagane](#required-parameters)
    - [Parametry Opcjonalne](#parameters-optional-parameters)
    - [Ograniczenia Wyrażeń Regularnych](#parameters-regular-expression-constraints)
- [Nazwane Route](#named-routes)
- [Grupy Route](#route-groups)
    - [Middleware](#route-group-middleware)
    - [Kontrolery](#route-group-controllers)
    - [Routing Subdomen](#route-group-subdomain-routing)
    - [Prefiksy Route](#route-group-prefixes)
    - [Prefiksy Nazw Route](#route-group-name-prefixes)
- [Wiązanie Modelu z Route](#route-model-binding)
    - [Wiązanie Niejawne](#implicit-binding)
    - [Niejawne Wiązanie Enum](#implicit-enum-binding)
    - [Wiązanie Jawne](#explicit-binding)
- [Route Zastępcze (Fallback)](#fallback-routes)
- [Ograniczanie Częstotliwości (Rate Limiting)](#rate-limiting)
    - [Definiowanie Rate Limiterów](#defining-rate-limiters)
    - [Przypisywanie Rate Limiterów do Route](#attaching-rate-limiters-to-routes)
- [Podszywanie się pod Metody Formularzy](#form-method-spoofing)
- [Dostęp do Bieżącego Route](#accessing-the-current-route)
- [Cross-Origin Resource Sharing (CORS)](#cors)
- [Cachowanie Route](#route-caching)

<a name="basic-routing"></a>
## Podstawy Routingu

Najprostsze route Laravel przyjmują URI i zamknięcie, zapewniając bardzo prostą i wyrazistą metodę definiowania tras i zachowań bez skomplikowanych plików konfiguracyjnych routingu:

```php
use Illuminate\Support\Facades\Route;

Route::get('/greeting', function () {
    return 'Hello World';
});
```

<a name="the-default-route-files"></a>
### Domyślne Pliki Routów

Wszystkie route Laravel są definiowane w plikach routów, które znajdują się w katalogu `routes`. Te pliki są automatycznie ładowane przez Laravel przy użyciu konfiguracji określonej w pliku `bootstrap/app.php` Twojej aplikacji. Plik `routes/web.php` definiuje route dla interfejsu webowego. Te route mają przypisaną [grupę middleware](/docs/{{version}}/middleware#laravels-default-middleware-groups) `web`, która zapewnia funkcje takie jak stan sesji i ochrona CSRF.

W większości aplikacji zaczniesz od definiowania routów w pliku `routes/web.php`. Route zdefiniowane w `routes/web.php` mogą być dostępne poprzez wpisanie URL zdefiniowanego route w przeglądarce. Na przykład, możesz uzyskać dostęp do następującego route, przechodząc do `http://example.com/user` w przeglądarce:

```php
use App\Http\Controllers\UserController;

Route::get('/user', [UserController::class, 'index']);
```

<a name="api-routes"></a>
#### Route API

Jeśli Twoja aplikacja będzie również oferować bezstanowe API, możesz włączyć routing API używając polecenia Artisan `install:api`:

```shell
php artisan install:api
```

Polecenie `install:api` instaluje [Laravel Sanctum](/docs/{{version}}/sanctum), który zapewnia solidną, ale prostą ochronę uwierzytelniania tokenów API, która może być używana do uwierzytelniania konsumentów API stron trzecich, SPA lub aplikacji mobilnych. Dodatkowo polecenie `install:api` tworzy plik `routes/api.php`:

```php
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

Route w `routes/api.php` są bezstanowe i mają przypisaną [grupę middleware](/docs/{{version}}/middleware#laravels-default-middleware-groups) `api`. Dodatkowo, prefiks URI `/api` jest automatycznie stosowany do tych route, więc nie musisz ręcznie stosować go do każdego route w pliku. Możesz zmienić prefiks, modyfikując plik `bootstrap/app.php` swojej aplikacji:

```php
->withRouting(
    api: __DIR__.'/../routes/api.php',
    apiPrefix: 'api/admin',
    // ...
)
```

<a name="available-router-methods"></a>
#### Dostępne Metody Routera

Router pozwala rejestrować route, które odpowiadają na dowolny czasownik HTTP:

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

Czasami może być konieczne zarejestrowanie route, który odpowiada na wiele czasowników HTTP. Możesz to zrobić używając metody `match`. Lub możesz nawet zarejestrować route, który odpowiada na wszystkie czasowniki HTTP używając metody `any`:

```php
Route::match(['get', 'post'], '/', function () {
    // ...
});

Route::any('/', function () {
    // ...
});
```

> [!NOTE]
> Podczas definiowania wielu route, które współdzielą ten sam URI, route używające metod `get`, `post`, `put`, `patch`, `delete` i `options` powinny być zdefiniowane przed route używającymi metod `any`, `match` i `redirect`. Zapewnia to, że przychodzące żądanie jest dopasowane do prawidłowego route.

<a name="dependency-injection"></a>
#### Wstrzykiwanie Zależności

Możesz określić typ-wskazówkę dla dowolnych zależności wymaganych przez Twój route w sygnaturze callback Twojego route. Zadeklarowane zależności będą automatycznie rozwiązane i wstrzyknięte do callback przez [kontener usług](/docs/{{version}}/container) Laravel. Na przykład, możesz określić typ-wskazówkę klasy `Illuminate\Http\Request`, aby bieżące żądanie HTTP zostało automatycznie wstrzyknięte do callback Twojego route:

```php
use Illuminate\Http\Request;

Route::get('/users', function (Request $request) {
    // ...
});
```

<a name="csrf-protection"></a>
#### Ochrona CSRF

Pamiętaj, że wszystkie formularze HTML wskazujące na route `POST`, `PUT`, `PATCH` lub `DELETE` zdefiniowane w pliku route `web` powinny zawierać pole tokena CSRF. W przeciwnym razie żądanie zostanie odrzucone. Więcej informacji o ochronie CSRF możesz przeczytać w [dokumentacji CSRF](/docs/{{version}}/csrf):

```blade
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

<a name="redirect-routes"></a>
### Route Przekierowań

Jeśli definiujesz route, który przekierowuje do innego URI, możesz użyć metody `Route::redirect`. Ta metoda zapewnia wygodny skrót, więc nie musisz definiować pełnego route ani kontrolera do wykonywania prostego przekierowania:

```php
Route::redirect('/here', '/there');
```

Domyślnie `Route::redirect` zwraca kod statusu `302`. Możesz dostosować kod statusu używając opcjonalnego trzeciego parametru:

```php
Route::redirect('/here', '/there', 301);
```

Lub możesz użyć metody `Route::permanentRedirect`, aby zwrócić kod statusu `301`:

```php
Route::permanentRedirect('/here', '/there');
```

> [!WARNING]
> Podczas używania parametrów route w route przekierowań, następujące parametry są zarezerwowane przez Laravel i nie mogą być używane: `destination` i `status`.

<a name="view-routes"></a>
### Route Widoków

Jeśli Twój route musi tylko zwrócić [widok](/docs/{{version}}/views), możesz użyć metody `Route::view`. Podobnie jak metoda `redirect`, ta metoda zapewnia prosty skrót, więc nie musisz definiować pełnego route ani kontrolera. Metoda `view` przyjmuje URI jako pierwszy argument i nazwę widoku jako drugi argument. Dodatkowo możesz przekazać tablicę danych do widoku jako opcjonalny trzeci argument:

```php
Route::view('/welcome', 'welcome');

Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

> [!WARNING]
> Podczas używania parametrów route w route widoków, następujące parametry są zarezerwowane przez Laravel i nie mogą być używane: `view`, `data`, `status` i `headers`.

<a name="listing-your-routes"></a>
### Wyświetlanie Listy Twoich Route

Polecenie Artisan `route:list` może łatwo zapewnić przegląd wszystkich route zdefiniowanych przez Twoją aplikację:

```shell
php artisan route:list
```

Domyślnie middleware route przypisane do każdego route nie będą wyświetlane w wyjściu `route:list`; jednak możesz polecić Laravel, aby wyświetlał middleware route i nazwy grup middleware, dodając opcję `-v` do polecenia:

```shell
php artisan route:list -v

# Rozwijanie grup middleware...
php artisan route:list -vv
```

Możesz również polecić Laravel, aby pokazywał tylko route zaczynające się od danego URI:

```shell
php artisan route:list --path=api
```

Ponadto możesz polecić Laravel, aby ukrył wszystkie route zdefiniowane przez pakiety stron trzecich, dodając opcję `--except-vendor` podczas wykonywania polecenia `route:list`:

```shell
php artisan route:list --except-vendor
```

Podobnie możesz również polecić Laravel, aby pokazywał tylko route zdefiniowane przez pakiety stron trzecich, dodając opcję `--only-vendor` podczas wykonywania polecenia `route:list`:

```shell
php artisan route:list --only-vendor
```

<a name="routing-customization"></a>
### Dostosowanie Routingu

Domyślnie route Twojej aplikacji są konfigurowane i ładowane przez plik `bootstrap/app.php`:

```php
<?php

use Illuminate\Foundation\Application;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )->create();
```

However, sometimes you may want to define an entirely new file to contain a subset of your application's routes. To accomplish this, you may provide a `then` closure to the `withRouting` method. Within this closure, you may register any additional routes that are necessary for your application:

```php
use Illuminate\Support\Facades\Route;

->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
    then: function () {
        Route::middleware('api')
            ->prefix('webhooks')
            ->name('webhooks.')
            ->group(base_path('routes/webhooks.php'));
    },
)
```

Or, you may even take complete control over route registration by providing a `using` closure to the `withRouting` method. When this argument is passed, no HTTP routes will be registered by the framework and you are responsible for manually registering all routes:

```php
use Illuminate\Support\Facades\Route;

->withRouting(
    commands: __DIR__.'/../routes/console.php',
    using: function () {
        Route::middleware('api')
            ->prefix('api')
            ->group(base_path('routes/api.php'));

        Route::middleware('web')
            ->group(base_path('routes/web.php'));
    },
)
```

<a name="route-parameters"></a>
## Parametry Route

<a name="required-parameters"></a>
### Parametry Wymagane

Czasami będziesz musiał przechwycić segmenty URI w swoim route. Na przykład, może być konieczne przechwycenie ID użytkownika z URL. Możesz to zrobić, definiując parametry route:

```php
Route::get('/user/{id}', function (string $id) {
    return 'User '.$id;
});
```

Możesz zdefiniować tyle parametrów route, ile wymaga Twój route:

```php
Route::get('/posts/{post}/comments/{comment}', function (string $postId, string $commentId) {
    // ...
});
```

Parametry route są zawsze zamknięte w nawiasach klamrowych `{}` i powinny składać się ze znaków alfabetycznych. Podkreślenia (`_`) są również akceptowane w nazwach parametrów route. Parametry route są wstrzykiwane do callbacków / kontrolerów route na podstawie ich kolejności - nazwy argumentów callback / kontrolera route nie mają znaczenia.

<a name="parameters-and-dependency-injection"></a>
#### Parameters and Dependency Injection

If your route has dependencies that you would like the Laravel service container to automatically inject into your route's callback, you should list your route parameters after your dependencies:

```php
use Illuminate\Http\Request;

Route::get('/user/{id}', function (Request $request, string $id) {
    return 'User '.$id;
});
```

<a name="parameters-optional-parameters"></a>
### Parametry Opcjonalne

Od czasu do czasu możesz potrzebować określić parametr route, który może nie zawsze być obecny w URI. Możesz to zrobić, umieszczając znak `?` po nazwie parametru. Upewnij się, że nadasz odpowiedniej zmiennej route wartość domyślną:

```php
Route::get('/user/{name?}', function (?string $name = null) {
    return $name;
});

Route::get('/user/{name?}', function (?string $name = 'John') {
    return $name;
});
```

<a name="parameters-regular-expression-constraints"></a>
### Ograniczenia Wyrażeń Regularnych

Możesz ograniczyć format parametrów route używając metody `where` na instancji route. Metoda `where` przyjmuje nazwę parametru i wyrażenie regularne określające, jak parametr powinien być ograniczony:

```php
Route::get('/user/{name}', function (string $name) {
    // ...
})->where('name', '[A-Za-z]+');

Route::get('/user/{id}', function (string $id) {
    // ...
})->where('id', '[0-9]+');

Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

Dla wygody, niektóre często używane wzorce wyrażeń regularnych mają metody pomocnicze, które pozwalają szybko dodać ograniczenia wzorca do Twoich route:

```php
Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->whereNumber('id')->whereAlpha('name');

Route::get('/user/{name}', function (string $name) {
    // ...
})->whereAlphaNumeric('name');

Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUuid('id');

Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUlid('id');

Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', ['movie', 'song', 'painting']);

Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', CategoryEnum::cases());
```

Jeśli przychodzące żądanie nie pasuje do ograniczeń wzorca route, zostanie zwrócona odpowiedź HTTP 404.

<a name="parameters-global-constraints"></a>
#### Ograniczenia Globalne

Jeśli chcesz, aby parametr route był zawsze ograniczony przez dane wyrażenie regularne, możesz użyć metody `pattern`. Powinieneś zdefiniować te wzorce w metodzie `boot` klasy `App\Providers\AppServiceProvider` Twojej aplikacji:

```php
use Illuminate\Support\Facades\Route;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Route::pattern('id', '[0-9]+');
}
```

Po zdefiniowaniu wzorca jest on automatycznie stosowany do wszystkich route używających tej nazwy parametru:

```php
Route::get('/user/{id}', function (string $id) {
    // Only executed if {id} is numeric...
});
```

<a name="parameters-encoded-forward-slashes"></a>
#### Zakodowane Ukośniki

Komponent routingu Laravel pozwala na obecność wszystkich znaków oprócz `/` w wartościach parametrów route. Musisz jawnie zezwolić, aby `/` był częścią Twojego symbolu wieloznacznego, używając wyrażenia regularnego warunku `where`:

```php
Route::get('/search/{search}', function (string $search) {
    return $search;
})->where('search', '.*');
```

> [!WARNING]
> Zakodowane ukośniki są obsługiwane tylko w ostatnim segmencie route.

<a name="named-routes"></a>
## Nazwane Route

Nazwane route pozwalają na wygodne generowanie URLów lub przekierowań dla określonych route. Możesz określić nazwę dla route, łącząc metodę `name` z definicją route:

```php
Route::get('/user/profile', function () {
    // ...
})->name('profile');
```

Możesz również określić nazwy route dla akcji kontrolera:

```php
Route::get(
    '/user/profile',
    [UserProfileController::class, 'show']
)->name('profile');
```

> [!WARNING]
> Nazwy route powinny zawsze być unikalne.

<a name="generating-urls-to-named-routes"></a>
#### Generowanie URLów do Nazwanych Route

Po przypisaniu nazwy do danego route, możesz użyć nazwy route podczas generowania URLów lub przekierowań za pomocą funkcji pomocniczych `route` i `redirect` Laravel:

```php
// Generowanie URLów...
$url = route('profile');

// Generowanie przekierowań...
return redirect()->route('profile');

return to_route('profile');
```

Jeśli nazwany route definiuje parametry, możesz przekazać parametry jako drugi argument do funkcji `route`. Podane parametry zostaną automatycznie wstawione do wygenerowanego URL we właściwych pozycjach:

```php
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');

$url = route('profile', ['id' => 1]);
```

Jeśli przekazujesz dodatkowe parametry w tablicy, te pary klucz / wartość zostaną automatycznie dodane do ciągu zapytania wygenerowanego URL:

```php
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');

$url = route('profile', ['id' => 1, 'photos' => 'yes']);

// http://example.com/user/1/profile?photos=yes
```

> [!NOTE]
> Czasami możesz chcieć określić wartości domyślne dla parametrów URL obowiązujących na poziomie całego żądania, takie jak obecne locale. Aby to osiągnąć, możesz użyć [metody URL::defaults](/docs/{{version}}/urls#default-values).

<a name="inspecting-the-current-route"></a>
#### Inspecting the Current Route

If you would like to determine if the current request was routed to a given named route, you may use the `named` method on a Route instance. For example, you may check the current route name from a route middleware:

```php
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

/**
 * Handle an incoming request.
 *
 * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
 */
public function handle(Request $request, Closure $next): Response
{
    if ($request->route()->named('profile')) {
        // ...
    }

    return $next($request);
}
```

<a name="route-groups"></a>
## Grupy Route

Grupy route pozwalają współdzielić atrybuty route, takie jak middleware, w dużej liczbie route bez konieczności definiowania tych atrybuów dla każdego pojedynczego route.

Zagnieżdżone grupy próbują inteligentnie "połączyć" atrybuty ze swoją grupą nadrzędną. Middleware i warunki `where` są łączone, podczas gdy nazwy i prefiksy są dodawane. Ograniczniki przestrzeni nazw i ukośniki w prefiksach URI są automatycznie dodawane tam, gdzie jest to właściwe.

<a name="route-group-middleware"></a>
### Middleware

Aby przypisać [middleware](/docs/{{version}}/middleware) do wszystkich route w grupie, możesz użyć metody `middleware` przed zdefiniowaniem grupy. Middleware są wykonywane w kolejności, w jakiej są wymienione w tablicy:

```php
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // Uses first & second middleware...
    });

    Route::get('/user/profile', function () {
        // Uses first & second middleware...
    });
});
```

<a name="route-group-controllers"></a>
### Kontrolery

Jeśli grupa route wykorzystuje ten sam [kontroler](/docs/{{version}}/controllers), możesz użyć metody `controller`, aby zdefiniować wspólny kontroler dla wszystkich route w grupie. Następnie, podczas definiowania route, musisz tylko podać metodę kontrolera, którą wywołują:

```php
use App\Http\Controllers\OrderController;

Route::controller(OrderController::class)->group(function () {
    Route::get('/orders/{id}', 'show');
    Route::post('/orders', 'store');
});
```

<a name="route-group-subdomain-routing"></a>
### Routing Subdomen

Grupy route mogą również być używane do obsługi routingu subdomen. Subdomeny mogą mieć przypisane parametry route tak jak URI route, pozwalając przechwytywac część subdomeny do użycia w route lub kontrolerze. Subdomene można określić, wywołując metodę `domain` przed zdefiniowaniem grupy:

```php
Route::domain('{account}.example.com')->group(function () {
    Route::get('/user/{id}', function (string $account, string $id) {
        // ...
    });
});
```

> [!WARNING]
> Aby zapewnić, że Twoje route subdomen są osiągalne, powinieneś zarejestrować route subdomen przed zarejestrowaniem route domeny głównej. Zapobiegnie to nadpisaniu route subdomen przez route domeny głównej, które mają tę samą ścieżkę URI.

<a name="route-group-prefixes"></a>
### Prefiksy Route

Metody `prefix` można użyć do prefiksowania każdego route w grupie danym URI. Na przykład, możesz chcieć poprzedzić wszystkie URI route w grupie prefiksem `admin`:

```php
Route::prefix('admin')->group(function () {
    Route::get('/users', function () {
        // Matches The "/admin/users" URL
    });
});
```

<a name="route-group-name-prefixes"></a>
### Prefiksy Nazw Route

Metody `name` można użyć do prefiksowania każdej nazwy route w grupie danym ciągiem znaków. Na przykład, możesz chcieć poprzedzić nazwy wszystkich route w grupie prefiksem `admin`. Podany ciąg jest prefiksowany do nazwy route dokładnie tak, jak jest określony, więc upewnimy się, że dodamy końcową kropkę `.` w prefiksie:

```php
Route::name('admin.')->group(function () {
    Route::get('/users', function () {
        // Route assigned name "admin.users"...
    })->name('users');
});
```

<a name="route-model-binding"></a>
## Wiązanie Modelu z Route

Podczas wstrzykiwania ID modelu do route lub akcji kontrolera, często będziesz zapytywać bazę danych, aby pobrać model odpowiadający temu ID. Wiązanie modelu z route Laravel zapewnia wygodny sposób automatycznego wstrzykiwania instancji modelu bezpośrednio do Twoich route. Na przykład, zamiast wstrzykiwać ID użytkownika, możesz wstrzyknąć całą instancję modelu `User` pasującą do danego ID.

<a name="implicit-binding"></a>
### Wiązanie Niejawne

Laravel automatycznie rozwiązuje modele Eloquent zdefiniowane w route lub akcjach kontrolera, których nazwy zmiennych z type-hint pasują do nazwy segmentu route. Na przykład:

```php
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    return $user->email;
});
```

Ponieważ zmienna `$user` ma type-hint jako model Eloquent `App\Models\User` i nazwa zmiennej pasuje do segmentu URI `{user}`, Laravel automatycznie wstrzyknie instancję modelu, która ma ID pasujące do odpowiedniej wartości z URI żądania. Jeśli pasująca instancja modelu nie zostanie znaleziona w bazie danych, automatycznie zostanie wygenerowana odpowiedź HTTP 404.

Oczywieście, wiązanie niejawne jest również możliwe podczas używania metod kontrolera. Ponownie zwróć uwagę, że segment URI `{user}` pasuje do zmiennej `$user` w kontrolerze, która zawiera type-hint `App\Models\User`:

```php
use App\Http\Controllers\UserController;
use App\Models\User;

// Route definition...
Route::get('/users/{user}', [UserController::class, 'show']);

// Controller method definition...
public function show(User $user)
{
    return view('user.profile', ['user' => $user]);
}
```

<a name="implicit-soft-deleted-models"></a>
#### Modele Miękko Usunięte

Zwykle wiązanie niejawne modelu nie będzie pobierać modeli, które zostały [miękko usunięte](/docs/{{version}}/eloquent#soft-deleting). Jednak możesz polecić wiązaniu niejawnemu pobieranie tych modeli, łącząc metodę `withTrashed` do definicji Twojego route:

```php
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    return $user->email;
})->withTrashed();
```

<a name="customizing-the-default-key-name"></a>
#### Dostosowanie Klucza

Czasami możesz chcieć rozwiązywać modele Eloquent używając kolumny innej niż `id`. Aby to zrobić, możesz określić kolumnę w definicji parametru route:

```php
use App\Models\Post;

Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});
```

Jeśli chcesz, aby wiązanie modelu zawsze używało kolumny bazy danych innej niż `id` podczas pobierania danej klasy modelu, możesz nadpisać metodę `getRouteKeyName` w modelu Eloquent:

```php
/**
 * Get the route key for the model.
 */
public function getRouteKeyName(): string
{
    return 'slug';
}
```

<a name="implicit-model-binding-scoping"></a>
#### Custom Keys and Scoping

When implicitly binding multiple Eloquent models in a single route definition, you may wish to scope the second Eloquent model such that it must be a child of the previous Eloquent model. For example, consider this route definition that retrieves a blog post by slug for a specific user:

```php
use App\Models\Post;
use App\Models\User;

Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
});
```

When using a custom keyed implicit binding as a nested route parameter, Laravel will automatically scope the query to retrieve the nested model by its parent using conventions to guess the relationship name on the parent. In this case, it will be assumed that the `User` model has a relationship named `posts` (the plural form of the route parameter name) which can be used to retrieve the `Post` model.

If you wish, you may instruct Laravel to scope "child" bindings even when a custom key is not provided. To do so, you may invoke the `scopeBindings` method when defining your route:

```php
use App\Models\Post;
use App\Models\User;

Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();
```

Or, you may instruct an entire group of route definitions to use scoped bindings:

```php
Route::scopeBindings()->group(function () {
    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    });
});
```

Similarly, you may explicitly instruct Laravel to not scope bindings by invoking the `withoutScopedBindings` method:

```php
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
})->withoutScopedBindings();
```

<a name="customizing-missing-model-behavior"></a>
#### Customizing Missing Model Behavior

Typically, a 404 HTTP response will be generated if an implicitly bound model is not found. However, you may customize this behavior by calling the `missing` method when defining your route. The `missing` method accepts a closure that will be invoked if an implicitly bound model cannot be found:

```php
use App\Http\Controllers\LocationsController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;

Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
    ->name('locations.view')
    ->missing(function (Request $request) {
        return Redirect::route('locations.index');
    });
```

<a name="implicit-enum-binding"></a>
### Implicit Enum Binding

PHP 8.1 introduced support for [Enums](https://www.php.net/manual/en/language.enumerations.backed.php). To complement this feature, Laravel allows you to type-hint a [string-backed Enum](https://www.php.net/manual/en/language.enumerations.backed.php) on your route definition and Laravel will only invoke the route if that route segment corresponds to a valid Enum value. Otherwise, a 404 HTTP response will be returned automatically. For example, given the following Enum:

```php
<?php

namespace App\Enums;

enum Category: string
{
    case Fruits = 'fruits';
    case People = 'people';
}
```

You may define a route that will only be invoked if the `{category}` route segment is `fruits` or `people`. Otherwise, Laravel will return a 404 HTTP response:

```php
use App\Enums\Category;
use Illuminate\Support\Facades\Route;

Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
```

<a name="explicit-binding"></a>
### Explicit Binding

You are not required to use Laravel's implicit, convention based model resolution in order to use model binding. You can also explicitly define how route parameters correspond to models. To register an explicit binding, use the router's `model` method to specify the class for a given parameter. You should define your explicit model bindings at the beginning of the `boot` method of your `AppServiceProvider` class:

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Route::model('user', User::class);
}
```

Next, define a route that contains a `{user}` parameter:

```php
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    // ...
});
```

Since we have bound all `{user}` parameters to the `App\Models\User` model, an instance of that class will be injected into the route. So, for example, a request to `users/1` will inject the `User` instance from the database which has an ID of `1`.

If a matching model instance is not found in the database, a 404 HTTP response will be automatically generated.

<a name="customizing-the-resolution-logic"></a>
#### Customizing the Resolution Logic

If you wish to define your own model binding resolution logic, you may use the `Route::bind` method. The closure you pass to the `bind` method will receive the value of the URI segment and should return the instance of the class that should be injected into the route. Again, this customization should take place in the `boot` method of your application's `AppServiceProvider`:

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Route::bind('user', function (string $value) {
        return User::where('name', $value)->firstOrFail();
    });
}
```

Alternatively, you may override the `resolveRouteBinding` method on your Eloquent model. This method will receive the value of the URI segment and should return the instance of the class that should be injected into the route:

```php
/**
 * Retrieve the model for a bound value.
 *
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveRouteBinding($value, $field = null)
{
    return $this->where('name', $value)->firstOrFail();
}
```

If a route is utilizing [implicit binding scoping](#implicit-model-binding-scoping), the `resolveChildRouteBinding` method will be used to resolve the child binding of the parent model:

```php
/**
 * Retrieve the child model for a bound value.
 *
 * @param  string  $childType
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveChildRouteBinding($childType, $value, $field)
{
    return parent::resolveChildRouteBinding($childType, $value, $field);
}
```

<a name="fallback-routes"></a>
## Route Zastępcze (Fallback)

Używając metody `Route::fallback`, możesz zdefiniować route, który zostanie wykonany, gdy żaden inny route nie pasuje do przychodzącego żądania. Zwykle nieobsłużone żądania automatycznie renderują stronę "404" za pośrednictwem obsługi wyjątków Twojej aplikacji. Jednak ponieważ zazwyczaj definiujesz route `fallback` w pliku `routes/web.php`, wszystkie middleware w grupie middleware `web` będą miały zastosowanie do tego route. Możesz dodać dodatkowe middleware do tego route według potrzeb:

```php
Route::fallback(function () {
    // ...
});
```

<a name="rate-limiting"></a>
## Ograniczanie Częstotliwości (Rate Limiting)

<a name="defining-rate-limiters"></a>
### Definiowanie Rate Limiterów

Laravel zawiera potężne i konfigurowalne usługi ograniczania częstotliwości, których możesz użyć do ograniczenia ilości ruchu dla danego route lub grupy route. Aby zacząć, powinieneś zdefiniować konfiguracje rate limiterów, które spełniają potrzeby Twojej aplikacji.

Rate limitery mogą być zdefiniowane w metodzie `boot` klasy `App\Providers\AppServiceProvider` Twojej aplikacji:

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    });
}
```

Rate limiters are defined using the `RateLimiter` facade's `for` method. The `for` method accepts a rate limiter name and a closure that returns the limit configuration that should apply to routes that are assigned to the rate limiter. Limit configuration are instances of the `Illuminate\Cache\RateLimiting\Limit` class. This class contains helpful "builder" methods so that you can quickly define your limit. The rate limiter name may be any string you wish:

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000);
    });
}
```

If the incoming request exceeds the specified rate limit, a response with a 429 HTTP status code will automatically be returned by Laravel. If you would like to define your own response that should be returned by a rate limit, you may use the `response` method:

```php
RateLimiter::for('global', function (Request $request) {
    return Limit::perMinute(1000)->response(function (Request $request, array $headers) {
        return response('Custom response...', 429, $headers);
    });
});
```

Since rate limiter callbacks receive the incoming HTTP request instance, you may build the appropriate rate limit dynamically based on the incoming request or authenticated user:

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
        ? Limit::none()
        : Limit::perHour(10);
});
```

<a name="segmenting-rate-limits"></a>
#### Segmenting Rate Limits

Sometimes you may wish to segment rate limits by some arbitrary value. For example, you may wish to allow users to access a given route 100 times per minute per IP address. To accomplish this, you may use the `by` method when building your rate limit:

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
        ? Limit::none()
        : Limit::perMinute(100)->by($request->ip());
});
```

To illustrate this feature using another example, we can limit access to the route to 100 times per minute per authenticated user ID or 10 times per minute per IP address for guests:

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()
        ? Limit::perMinute(100)->by($request->user()->id)
        : Limit::perMinute(10)->by($request->ip());
});
```

<a name="multiple-rate-limits"></a>
#### Multiple Rate Limits

If needed, you may return an array of rate limits for a given rate limiter configuration. Each rate limit will be evaluated for the route based on the order they are placed within the array:

```php
RateLimiter::for('login', function (Request $request) {
    return [
        Limit::perMinute(500),
        Limit::perMinute(3)->by($request->input('email')),
    ];
});
```

If you're assigning multiple rate limits segmented by identical `by` values, you should ensure that each `by` value is unique. The easiest way to achieve this is to prefix the values given to the `by` method:

```php
RateLimiter::for('uploads', function (Request $request) {
    return [
        Limit::perMinute(10)->by('minute:'.$request->user()->id),
        Limit::perDay(1000)->by('day:'.$request->user()->id),
    ];
});
```

<a name="response-base-rate-limiting"></a>
#### Response-Based Rate Limiting

In addition to rate limiting incoming requests, Laravel allows you to rate limit based on the response using the `after` method. This is useful when you only want to count certain responses toward the rate limit, such as validation errors, 404 responses, or other specific HTTP status codes.

The `after` method accepts a closure that receives the response and should return `true` if the response should be counted toward the rate limit, or `false` if it should be ignored. This is particularly useful for preventing enumeration attacks by limiting consecutive 404 responses, or allowing users to retry requests that fail validation without exhausting their rate limit on an endpoint that should only throttle successful operations:

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
use Symfony\Component\HttpFoundation\Response;

RateLimiter::for('resource-not-found', function (Request $request) {
    return Limit::perMinute(10)
        ->by($request->user()?->id ?: $request->ip())
        ->after(function (Response $response) {
            // Only count 404 responses toward the rate limit to prevent enumeration...
            return $response->status() === 404;
        });
});
```

<a name="attaching-rate-limiters-to-routes"></a>
### Attaching Rate Limiters to Routes

Rate limiters may be attached to routes or route groups using the `throttle` [middleware](/docs/{{version}}/middleware). The throttle middleware accepts the name of the rate limiter you wish to assign to the route:

```php
Route::middleware(['throttle:uploads'])->group(function () {
    Route::post('/audio', function () {
        // ...
    });

    Route::post('/video', function () {
        // ...
    });
});
```

<a name="throttling-with-redis"></a>
#### Throttling With Redis

By default, the `throttle` middleware is mapped to the `Illuminate\Routing\Middleware\ThrottleRequests` class. However, if you are using Redis as your application's cache driver, you may wish to instruct Laravel to use Redis to manage rate limiting. To do so, you should use the `throttleWithRedis` method in your application's `bootstrap/app.php` file. This method maps the `throttle` middleware to the `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` middleware class:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->throttleWithRedis();
    // ...
})
```

<a name="form-method-spoofing"></a>
## Podszywanie się pod Metody Formularzy

Formularze HTML nie obsługują akcji `PUT`, `PATCH` ani `DELETE`. Więc podczas definiowania route `PUT`, `PATCH` lub `DELETE`, które są wywoływane z formularza HTML, będziesz musiał dodać ukryte pole `_method` do formularza. Wartość wysłana z polem `_method` będzie używana jako metoda żądania HTTP:

```blade
<form action="/example" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```

Dla wygody, możesz użyć [dyrektywy Blade](/docs/{{version}}/blade) `@method`, aby wygenerować pole wejściowe `_method`:

```blade
<form action="/example" method="POST">
    @method('PUT')
    @csrf
</form>
```

<a name="accessing-the-current-route"></a>
## Dostęp do Bieżącego Route

Możesz użyć metod `current`, `currentRouteName` i `currentRouteAction` na fasadzie `Route`, aby uzyskać dostęp do informacji o route obsługującym przychodzące żądanie:

```php
use Illuminate\Support\Facades\Route;

$route = Route::current(); // Illuminate\Routing\Route
$name = Route::currentRouteName(); // string
$action = Route::currentRouteAction(); // string
```

Możesz sięgnąć do dokumentacji API dla [klasy bazowej fasady Route](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/Router.html) oraz [instancji Route](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/Route.html), aby przejrzeć wszystkie metody dostępne w klasach routera i route.

<a name="cors"></a>
## Cross-Origin Resource Sharing (CORS)

Laravel może automatycznie odpowiadać na żądania HTTP `OPTIONS` CORS z wartościami, które skonfigurujesz. Żądania `OPTIONS` będą automatycznie obsługiwane przez [middleware](/docs/{{version}}/middleware) `HandleCors`, który jest automatycznie dołączony do globalnego stosu middleware Twojej aplikacji.

Czasami może być konieczne dostosowanie wartości konfiguracji CORS dla Twojej aplikacji. Możesz to zrobić, publikując plik konfiguracyjny `cors` za pomocą polecenia Artisan `config:publish`:

```shell
php artisan config:publish cors
```

To polecenie umieści plik konfiguracyjny `cors.php` w katalogu `config` Twojej aplikacji.

> [!NOTE]
> Więcej informacji o CORS i nagłówkach CORS możesz znaleźć w [dokumentacji sieci web MDN dotyczącej CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers).

<a name="route-caching"></a>
## Cachowanie Route

Podczas wdrażania aplikacji do produkcji, powinieneś skorzystać z cache route Laravel. Użycie cache route drastycznie zmniejszy czas potrzebny na zarejestrowanie wszystkich route Twojej aplikacji. Aby wygenerować cache route, wykonaj polecenie Artisan `route:cache`:

```shell
php artisan route:cache
```

Po uruchomieniu tego polecenia, plik cache route będzie ładowany przy każdym żądaniu. Pamiętaj, że jeśli dodasz nowe route, będziesz musiał wygenerować świeży cache route. Z tego powodu powinieneś uruchamiać polecenie `route:cache` tylko podczas wdrażania projektu.

Możesz użyć polecenia `route:clear`, aby wyczyścić cache route:

```shell
php artisan route:clear
```
