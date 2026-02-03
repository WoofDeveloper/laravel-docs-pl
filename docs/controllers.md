# Kontrolery

- [Wprowadzenie](#introduction)
- [Pisanie kontrolerów](#writing-controllers)
    - [Podstawowe kontrolery](#basic-controllers)
    - [Kontrolery jednoakcyjne](#single-action-controllers)
- [Middleware kontrolera](#controller-middleware)
- [Kontrolery zasobów](#resource-controllers)
    - [Częściowe trasy zasobów](#restful-partial-resource-routes)
    - [Zagnieżdżone zasoby](#restful-nested-resources)
    - [Nazywanie tras zasobów](#restful-naming-resource-routes)
    - [Nazywanie parametrów tras zasobów](#restful-naming-resource-route-parameters)
    - [Zakres tras zasobów](#restful-scoping-resource-routes)
    - [Lokalizowanie URI zasobów](#restful-localizing-resource-uris)
    - [Uzupełnianie kontrolerów zasobów](#restful-supplementing-resource-controllers)
    - [Kontrolery zasobów singleton](#singleton-resource-controllers)
    - [Middleware i kontrolery zasobów](#middleware-and-resource-controllers)
- [Wstrzykiwanie zależności i kontrolery](#dependency-injection-and-controllers)

<a name="introduction"></a>
## Wprowadzenie

Zamiast definiować całą logikę obsługi żądań jako domknięcia w plikach tras, możesz zorganizować to zachowanie przy użyciu klas "kontrolerów". Kontrolery mogą grupować powiązaną logikę obsługi żądań w jednej klasie. Na przykład klasa `UserController` może obsługiwać wszystkie przychodzące żądania związane z użytkownikami, w tym wyświetlanie, tworzenie, aktualizowanie i usuwanie użytkowników. Domyślnie kontrolery są przechowywane w katalogu `app/Http/Controllers`.

<a name="writing-controllers"></a>
## Pisanie kontrolerów

<a name="basic-controllers"></a>
### Podstawowe kontrolery

Aby szybko wygenerować nowy kontroler, możesz uruchomić polecenie Artisan `make:controller`. Domyślnie wszystkie kontrolery dla Twojej aplikacji są przechowywane w katalogu `app/Http/Controllers`:

```shell
php artisan make:controller UserController
```

Spójrzmy na przykład podstawowego kontrolera. Kontroler może mieć dowolną liczbę publicznych metod, które będą odpowiadać na przychodzące żądania HTTP:

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Pokaż profil dla danego użytkownika.
     */
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

Po napisaniu klasy kontrolera i metody możesz zdefiniować trasę do metody kontrolera w następujący sposób:

```php
use App\Http\Controllers\UserController;

Route::get('/user/{id}', [UserController::class, 'show']);
```

Gdy przychodzące żądanie pasuje do określonego URI trasy, zostanie wywołana metoda `show` w klasie `App\Http\Controllers\UserController`, a parametry trasy zostaną przekazane do metody.

> [!NOTE]
> Kontrolery **nie muszą** rozszerzać klasy bazowej. Jednak czasami wygodne jest rozszerzenie klasy bazowego kontrolera, która zawiera metody, które powinny być współdzielone przez wszystkie Twoje kontrolery.

<a name="single-action-controllers"></a>
### Kontrolery jednoakcyjne

Jeśli akcja kontrolera jest szczególnie złożona, możesz uznać za wygodne poświęcenie całej klasy kontrolera tej pojedynczej akcji. Aby to osiągnąć, możesz zdefiniować pojedynczą metodę `__invoke` w kontrolerze:

```php
<?php

namespace App\Http\Controllers;

class ProvisionServer extends Controller
{
    /**
     * Zainicjuj nowy serwer webowy.
     */
    public function __invoke()
    {
        // ...
    }
}
```

Podczas rejestrowania tras dla kontrolerów jednoakcyjnych nie musisz określać metody kontrolera. Zamiast tego możesz po prostu przekazać nazwę kontrolera do routera:

```php
use App\Http\Controllers\ProvisionServer;

Route::post('/server', ProvisionServer::class);
```

Możesz wygenerować kontroler wywoływalny, używając opcji `--invokable` polecenia Artisan `make:controller`:

```shell
php artisan make:controller ProvisionServer --invokable
```

> [!NOTE]
> Szablony kontrolerów mogą być dostosowane przy użyciu [publikowania szablonów](/docs/{{version}}/artisan#stub-customization).

<a name="controller-middleware"></a>
## Middleware kontrolera

[Middleware](/docs/{{version}}/middleware) może być przypisany do tras kontrolera w plikach tras:

```php
Route::get('/profile', [UserController::class, 'show'])->middleware('auth');
```

Możesz również uznać za wygodne określenie middleware w klasie kontrolera. Aby to zrobić, Twój kontroler powinien implementować interfejs `HasMiddleware`, który określa, że kontroler powinien mieć statyczną metodę `middleware`. Z tej metody możesz zwrócić tablicę middleware, które powinny być zastosowane do akcji kontrolera:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Routing\Controllers\HasMiddleware;
use Illuminate\Routing\Controllers\Middleware;

class UserController implements HasMiddleware
{
    /**
     * Pobierz middleware, które powinny być przypisane do kontrolera.
     */
    public static function middleware(): array
    {
        return [
            'auth',
            new Middleware('log', only: ['index']),
            new Middleware('subscribed', except: ['store']),
        ];
    }

    // ...
}
```

Możesz również zdefiniować middleware kontrolera jako domknięcia, co zapewnia wygodny sposób definiowania wbudowanego middleware bez pisania całej klasy middleware:

```php
use Closure;
use Illuminate\Http\Request;

/**
 * Pobierz middleware, które powinny być przypisane do kontrolera.
 */
public static function middleware(): array
{
    return [
        function (Request $request, Closure $next) {
            return $next($request);
        },
    ];
}
```

<a name="resource-controllers"></a>
## Kontrolery zasobów

Jeśli myślisz o każdym modelu Eloquent w swojej aplikacji jako o "zasobie", typowe jest wykonywanie tych samych zestawów akcji względem każdego zasobu w aplikacji. Na przykład wyobraź sobie, że Twoja aplikacja zawiera model `Photo` i model `Movie`. Jest prawdopodobne, że użytkownicy mogą tworzyć, odczytywać, aktualizować lub usuwać te zasoby.

Z powodu tego powszechnego przypadku użycia, routing zasobów Laravel przypisuje typowe trasy tworzenia, odczytu, aktualizacji i usuwania ("CRUD") do kontrolera za pomocą jednej linii kodu. Aby rozpocząć, możemy użyć opcji `--resource` polecenia Artisan `make:controller`, aby szybko utworzyć kontroler do obsługi tych akcji:

```shell
php artisan make:controller PhotoController --resource
```

To polecenie wygeneruje kontroler w `app/Http/Controllers/PhotoController.php`. Kontroler będzie zawierał metodę dla każdej z dostępnych operacji zasobów. Następnie możesz zarejestrować trasę zasobu, która wskazuje na kontroler:

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class);
```

Ta pojedyncza deklaracja trasy tworzy wiele tras do obsługi różnych akcji na zasobie. Wygenerowany kontroler będzie już miał metody przygotowane dla każdej z tych akcji. Pamiętaj, że zawsze możesz uzyskać szybki przegląd tras swojej aplikacji, uruchamiając polecenie Artisan `route:list`.

Możesz nawet zarejestrować wiele kontrolerów zasobów jednocześnie, przekazując tablicę do metody `resources`:

```php
Route::resources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

Metoda `softDeletableResources` rejestruje wiele kontrolerów zasobów, które wszystkie używają metody `withTrashed`:

```php
Route::softDeletableResources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

<a name="actions-handled-by-resource-controllers"></a>
#### Akcje obsługiwane przez kontrolery zasobów

<div class="overflow-auto">

| Czasownik | URI                    | Akcja   | Nazwa trasy    |
| --------- | ---------------------- | ------- | -------------- |
| GET       | `/photos`              | index   | photos.index   |
| GET       | `/photos/create`       | create  | photos.create  |
| POST      | `/photos`              | store   | photos.store   |
| GET       | `/photos/{photo}`      | show    | photos.show    |
| GET       | `/photos/{photo}/edit` | edit    | photos.edit    |
| PUT/PATCH | `/photos/{photo}`      | update  | photos.update  |
| DELETE    | `/photos/{photo}`      | destroy | photos.destroy |

</div>

<a name="customizing-missing-model-behavior"></a>
#### Dostosowywanie zachowania brakującego modelu

Zazwyczaj generowana jest odpowiedź HTTP 404, jeśli niejawnie związany model zasobu nie zostanie znaleziony. Możesz jednak dostosować to zachowanie, wywołując metodę `missing` podczas definiowania trasy zasobu. Metoda `missing` akceptuje domknięcie, które zostanie wywołane, jeśli niejawnie związany model nie może zostać znaleziony dla żadnej z tras zasobu:

```php
use App\Http\Controllers\PhotoController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;

Route::resource('photos', PhotoController::class)
    ->missing(function (Request $request) {
        return Redirect::route('photos.index');
    });
```

<a name="soft-deleted-models"></a>
#### Modele usunięte nietrwale

Zazwyczaj niejawne wiązanie modelu nie pobiera modeli, które zostały [usunięte nietrwale](/docs/{{version}}/eloquent#soft-deleting), a zamiast tego zwróci odpowiedź HTTP 404. Możesz jednak poinstruować framework, aby zezwalał na modele usunięte nietrwale, wywołując metodę `withTrashed` podczas definiowania trasy zasobu:

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->withTrashed();
```

Wywołanie `withTrashed` bez argumentów zezwoli na modele usunięte nietrwale dla tras zasobów `show`, `edit` i `update`. Możesz określić podzbiór tych tras, przekazując tablicę do metody `withTrashed`:

```php
Route::resource('photos', PhotoController::class)->withTrashed(['show']);
```

<a name="specifying-the-resource-model"></a>
#### Określanie modelu zasobu

Jeśli używasz [wiązania modelu trasy](/docs/{{version}}/routing#route-model-binding) i chciałbyś, aby metody kontrolera zasobu wskazywały na instancję modelu, możesz użyć opcji `--model` podczas generowania kontrolera:

```shell
php artisan make:controller PhotoController --model=Photo --resource
```

<a name="generating-form-requests"></a>
#### Generowanie żądań formularzy

Możesz podać opcję `--requests` podczas generowania kontrolera zasobu, aby poinstruować Artisan do wygenerowania [klas żądań formularzy](/docs/{{version}}/validation#form-request-validation) dla metod przechowywania i aktualizacji kontrolera:

```shell
php artisan make:controller PhotoController --model=Photo --resource --requests
```

<a name="restful-partial-resource-routes"></a>
### Częściowe trasy zasobów

Podczas deklarowania trasy zasobu możesz określić podzbiór akcji, które kontroler powinien obsługiwać zamiast pełnego zestawu domyślnych akcji:

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->only([
    'index', 'show'
]);

Route::resource('photos', PhotoController::class)->except([
    'create', 'store', 'update', 'destroy'
]);
```

<a name="api-resource-routes"></a>
#### Trasy zasobów API

Podczas deklarowania tras zasobów, które będą wykorzystywane przez API, zazwyczaj będziesz chciał wykluczyć trasy prezentujące szablony HTML, takie jak `create` i `edit`. Dla wygody możesz użyć metody `apiResource`, aby automatycznie wykluczyć te dwie trasy:

```php
use App\Http\Controllers\PhotoController;

Route::apiResource('photos', PhotoController::class);
```

Możesz zarejestrować wiele kontrolerów zasobów API jednocześnie, przekazując tablicę do metody `apiResources`:

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\PostController;

Route::apiResources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

Aby szybko wygenerować kontroler zasobu API, który nie zawiera metod `create` ani `edit`, użyj przełącznika `--api` podczas wykonywania polecenia `make:controller`:

```shell
php artisan make:controller PhotoController --api
```

<a name="restful-nested-resources"></a>
### Zagnieżdżone zasoby

Czasami może być konieczne zdefiniowanie tras do zagnieżdżonego zasobu. Na przykład zasób zdjęcia może mieć wiele komentarzy, które mogą być dołączone do zdjęcia. Aby zagnieździć kontrolery zasobów, możesz użyć notacji "kropkowej" w deklaracji trasy:

```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class);
```

Ta trasa zarejestruje zagnieżdżony zasób, do którego można uzyskać dostęp za pomocą URI takich jak następujące:

```text
/photos/{photo}/comments/{comment}
```

<a name="scoping-nested-resources"></a>
#### Zakres zagnieżdżonych zasobów

Funkcja [niejawnego wiązania modelu](/docs/{{version}}/routing#implicit-model-binding-scoping) Laravel może automatycznie określać zakres zagnieżdżonych wiązań w taki sposób, że rozwiązany model potomny zostanie potwierdzony jako należący do modelu rodzica. Używając metody `scoped` podczas definiowania zagnieżdżonego zasobu, możesz włączyć automatyczne określanie zakresu, a także poinstruować Laravel, przez które pole powinien zostać pobrany zasób potomny. Aby uzyskać więcej informacji o tym, jak to osiągnąć, zobacz dokumentację dotyczącą [określania zakresu tras zasobów](#restful-scoping-resource-routes).

<a name="shallow-nesting"></a>
#### Płytkie zagnieżdżanie

Często nie jest całkowicie konieczne posiadanie zarówno identyfikatora rodzica, jak i potomka w URI, ponieważ identyfikator potomka jest już unikalnym identyfikatorem. Podczas korzystania z unikalnych identyfikatorów, takich jak autoinkrementujące klucze podstawowe do identyfikowania modeli w segmentach URI, możesz wybrać "płytkie zagnieżdżanie":

```php
use App\Http\Controllers\CommentController;

Route::resource('photos.comments', CommentController::class)->shallow();
```

Ta definicja trasy zdefiniuje następujące trasy:

<div class="overflow-auto">

| Czasownik | URI                               | Akcja   | Nazwa trasy            |
| --------- | --------------------------------- | ------- | ---------------------- |
| GET       | `/photos/{photo}/comments`        | index   | photos.comments.index  |
| GET       | `/photos/{photo}/comments/create` | create  | photos.comments.create |
| POST      | `/photos/{photo}/comments`        | store   | photos.comments.store  |
| GET       | `/comments/{comment}`             | show    | comments.show          |
| GET       | `/comments/{comment}/edit`        | edit    | comments.edit          |
| PUT/PATCH | `/comments/{comment}`             | update  | comments.update        |
| DELETE    | `/comments/{comment}`             | destroy | comments.destroy       |

</div>

<a name="restful-naming-resource-routes"></a>
### Nazywanie tras zasobów

Domyślnie wszystkie akcje kontrolera zasobów mają nazwę trasy; możesz jednak nadpisać te nazwy, przekazując tablicę `names` z żądanymi nazwami tras:

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->names([
    'create' => 'photos.build'
]);
```

<a name="restful-naming-resource-route-parameters"></a>
### Nazywanie parametrów tras zasobów

Domyślnie `Route::resource` utworzy parametry trasy dla tras zasobów na podstawie "pojedynczej" wersji nazwy zasobu. Możesz łatwo nadpisać to dla każdego zasobu, używając metody `parameters`. Tablica przekazana do metody `parameters` powinna być tablicą asocjacyjną nazw zasobów i nazw parametrów:

```php
use App\Http\Controllers\AdminUserController;

Route::resource('users', AdminUserController::class)->parameters([
    'users' => 'admin_user'
]);
```

Powyższy przykład generuje następujące URI dla trasy `show` zasobu:

```text
/users/{admin_user}
```

<a name="restful-scoping-resource-routes"></a>
### Zakres tras zasobów

Funkcja [ograniczonego niejawnego wiązania modelu](/docs/{{version}}/routing#implicit-model-binding-scoping) Laravel może automatycznie określać zakres zagnieżdżonych wiązań w taki sposób, że rozwiązany model potomny zostanie potwierdzony jako należący do modelu rodzica. Używając metody `scoped` podczas definiowania zagnieżdżonego zasobu, możesz włączyć automatyczne określanie zakresu, a także poinstruować Laravel, przez które pole powinien zostać pobrany zasób potomny:

```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class)->scoped([
    'comment' => 'slug',
]);
```

Ta trasa zarejestruje zagnieżdżony zasób z zakresem, do którego można uzyskać dostęp za pomocą URI takich jak następujące:

```text
/photos/{photo}/comments/{comment:slug}
```

Podczas używania niestandardowego wiązania niejawnego z kluczem jako parametru trasy zagnieżdżonej, Laravel automatycznie określi zakres zapytania do pobierania zagnieżdżonego modelu według jego rodzica, używając konwencji do odgadnięcia nazwy relacji w rodzicu. W tym przypadku założy się, że model `Photo` ma relację o nazwie `comments` (liczba mnoga nazwy parametru trasy), której można użyć do pobierania modelu `Comment`.

<a name="restful-localizing-resource-uris"></a>
### Lokalizowanie URI zasobów

Domyślnie `Route::resource` utworzy URI zasobów używając angielskich czasowników i reguł liczby mnogiej. Jeśli musisz zlokalizować czasowniki akcji `create` i `edit`, możesz użyć metody `Route::resourceVerbs`. Można to zrobić na początku metody `boot` w `App\Providers\AppServiceProvider` Twojej aplikacji:

```php
/**
 * Zainicjuj dowolne usługi aplikacji.
 */
public function boot(): void
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
}
```

Pluralizator Laravel obsługuje [kilka różnych języków, które możesz skonfigurować zgodnie ze swoimi potrzebami](/docs/{{version}}/localization#pluralization-language). Po dostosowaniu czasowników i języka liczby mnogiej, rejestracja trasy zasobu, taka jak `Route::resource('publicacion', PublicacionController::class)`, wygeneruje następujące URI:

```text
/publicacion/crear

/publicacion/{publicaciones}/editar
```

<a name="restful-supplementing-resource-controllers"></a>
### Uzupełnianie kontrolerów zasobów

Jeśli musisz dodać dodatkowe trasy do kontrolera zasobów poza domyślnym zestawem tras zasobów, powinieneś zdefiniować te trasy przed wywołaniem metody `Route::resource`; w przeciwnym razie trasy zdefiniowane przez metodę `resource` mogą nieumyślnie mieć pierwszeństwo przed Twoimi dodatkowymi trasami:

```php
use App\Http\Controller\PhotoController;

Route::get('/photos/popular', [PhotoController::class, 'popular']);
Route::resource('photos', PhotoController::class);
```

> [!NOTE]
> Pamiętaj, aby utrzymywać swoje kontrolery skoncentrowane. Jeśli rutynowo potrzebujesz metod spoza typowego zestawu akcji zasobów, rozważ podzielenie kontrolera na dwa mniejsze kontrolery.

<a name="singleton-resource-controllers"></a>
### Kontrolery zasobów singleton

Czasami Twoja aplikacja będzie miała zasoby, które mogą mieć tylko jedną instancję. Na przykład "profil" użytkownika może być edytowany lub aktualizowany, ale użytkownik może nie mieć więcej niż jednego "profilu". Podobnie obraz może mieć jedną "miniaturę". Te zasoby są nazywane "zasobami singleton", co oznacza, że ​​może istnieć jedna i tylko jedna instancja zasobu. W tych scenariuszach możesz zarejestrować kontroler zasobów "singleton":

```php
use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;

Route::singleton('profile', ProfileController::class);
```

Powyższa definicja zasobu singleton zarejestruje następujące trasy. Jak widać, trasy "tworzenia" nie są rejestrowane dla zasobów singleton, a zarejestrowane trasy nie akceptują identyfikatora, ponieważ może istnieć tylko jedna instancja zasobu:

<div class="overflow-auto">

| Czasownik | URI             | Akcja  | Nazwa trasy    |
| --------- | --------------- | ------ | -------------- |
| GET       | `/profile`      | show   | profile.show   |
| GET       | `/profile/edit` | edit   | profile.edit   |
| PUT/PATCH | `/profile`      | update | profile.update |

</div>

Zasoby singleton mogą być również zagnieżdżone w standardowym zasobie:

```php
Route::singleton('photos.thumbnail', ThumbnailController::class);
```

W tym przykładzie zasób `photos` otrzymałby wszystkie [standardowe trasy zasobów](#actions-handled-by-resource-controllers); jednak zasób `thumbnail` byłby zasobem singleton z następującymi trasami:

<div class="overflow-auto">

| Czasownik | URI                              | Akcja  | Nazwa trasy             |
| --------- | -------------------------------- | ------ | ----------------------- |
| GET       | `/photos/{photo}/thumbnail`      | show   | photos.thumbnail.show   |
| GET       | `/photos/{photo}/thumbnail/edit` | edit   | photos.thumbnail.edit   |
| PUT/PATCH | `/photos/{photo}/thumbnail`      | update | photos.thumbnail.update |

</div>

<a name="creatable-singleton-resources"></a>
#### Zasoby singleton z możliwością tworzenia

Czasami możesz chcieć zdefiniować trasy tworzenia i przechowywania dla zasobu singleton. Aby to osiągnąć, możesz wywołać metodę `creatable` podczas rejestrowania trasy zasobu singleton:

```php
Route::singleton('photos.thumbnail', ThumbnailController::class)->creatable();
```

W tym przykładzie zostaną zarejestrowane następujące trasy. Jak widać, trasa `DELETE` również zostanie zarejestrowana dla zasobów singleton z możliwością tworzenia:

<div class="overflow-auto">

| Czasownik | URI                                | Akcja   | Nazwa trasy              |
| --------- | ---------------------------------- | ------- | ------------------------ |
| GET       | `/photos/{photo}/thumbnail/create` | create  | photos.thumbnail.create  |
| POST      | `/photos/{photo}/thumbnail`        | store   | photos.thumbnail.store   |
| GET       | `/photos/{photo}/thumbnail`        | show    | photos.thumbnail.show    |
| GET       | `/photos/{photo}/thumbnail/edit`   | edit    | photos.thumbnail.edit    |
| PUT/PATCH | `/photos/{photo}/thumbnail`        | update  | photos.thumbnail.update  |
| DELETE    | `/photos/{photo}/thumbnail`        | destroy | photos.thumbnail.destroy |

</div>

Jeśli chcesz, aby Laravel zarejestrował trasę `DELETE` dla zasobu singleton, ale nie rejestrował tras tworzenia lub przechowywania, możesz użyć metody `destroyable`:

```php
Route::singleton(...)->destroyable();
```

<a name="api-singleton-resources"></a>
#### Zasoby singleton API

Metoda `apiSingleton` może być użyta do zarejestrowania zasobu singleton, który będzie manipulowany za pośrednictwem API, dzięki czemu trasy `create` i `edit` nie są potrzebne:

```php
Route::apiSingleton('profile', ProfileController::class);
```

Oczywiście zasoby singleton API mogą być również `creatable`, co zarejestruje trasy `store` i `destroy` dla zasobu:

```php
Route::apiSingleton('photos.thumbnail', ProfileController::class)->creatable();
```
<a name="middleware-and-resource-controllers"></a>
### Middleware i kontrolery zasobów

Laravel pozwala przypisać middleware do wszystkich lub tylko określonych metod tras zasobów, używając metod `middleware`, `middlewareFor` i `withoutMiddlewareFor`. Te metody zapewniają precyzyjną kontrolę nad tym, które middleware jest stosowane do każdej akcji zasobu.

#### Stosowanie middleware do wszystkich metod

Możesz użyć metody `middleware`, aby przypisać middleware do wszystkich tras generowanych przez trasę zasobu lub trasę zasobu singleton:

```php
Route::resource('users', UserController::class)
    ->middleware(['auth', 'verified']);

Route::singleton('profile', ProfileController::class)
    ->middleware('auth');
```

#### Stosowanie middleware do określonych metod

Możesz użyć metody `middlewareFor`, aby przypisać middleware do jednej lub więcej określonych metod danego kontrolera zasobów:

```php
Route::resource('users', UserController::class)
    ->middlewareFor('show', 'auth');

Route::apiResource('users', UserController::class)
    ->middlewareFor(['show', 'update'], 'auth');

Route::resource('users', UserController::class)
    ->middlewareFor('show', 'auth')
    ->middlewareFor('update', 'auth');

Route::apiResource('users', UserController::class)
    ->middlewareFor(['show', 'update'], ['auth', 'verified']);
```

Metoda `middlewareFor` może być również używana w połączeniu z kontrolerami zasobów singleton i zasobów singleton API:

```php
Route::singleton('profile', ProfileController::class)
    ->middlewareFor('show', 'auth');

Route::apiSingleton('profile', ProfileController::class)
    ->middlewareFor(['show', 'update'], 'auth');
```

#### Wykluczanie middleware z określonych metod

Możesz użyć metody `withoutMiddlewareFor`, aby wykluczyć middleware z określonych metod kontrolera zasobów:

```php
Route::middleware(['auth', 'verified', 'subscribed'])->group(function () {
    Route::resource('users', UserController::class)
        ->withoutMiddlewareFor('index', ['auth', 'verified'])
        ->withoutMiddlewareFor(['create', 'store'], 'verified')
        ->withoutMiddlewareFor('destroy', 'subscribed');
});
```

<a name="dependency-injection-and-controllers"></a>
## Wstrzykiwanie zależności i kontrolery

<a name="constructor-injection"></a>
#### Wstrzykiwanie w konstruktorze

[Kontener usług](/docs/{{version}}/container) Laravel jest używany do rozwiązywania wszystkich kontrolerów Laravel. W rezultacie możesz wskazać wszelkie zależności, których może potrzebować Twój kontroler w jego konstruktorze. Zadeklarowane zależności zostaną automatycznie rozwiązane i wstrzyknięte do instancji kontrolera:

```php
<?php

namespace App\Http\Controllers;

use App\Repositories\UserRepository;

class UserController extends Controller
{
    /**
     * Utwórz nową instancję kontrolera.
     */
    public function __construct(
        protected UserRepository $users,
    ) {}
}
```

<a name="method-injection"></a>
#### Wstrzykiwanie w metodzie

Oprócz wstrzykiwania w konstruktorze możesz również wskazywać zależności w metodach swojego kontrolera. Typowym przypadkiem użycia wstrzykiwania w metodzie jest wstrzyknięcie instancji `Illuminate\Http\Request` do metod kontrolera:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Zapisz nowego użytkownika.
     */
    public function store(Request $request): RedirectResponse
    {
        $name = $request->name;

        // Zapisz użytkownika...

        return redirect('/users');
    }
}
```

Jeśli Twoja metoda kontrolera oczekuje również danych wejściowych z parametru trasy, wypisz argumenty trasy po innych zależnościach. Na przykład, jeśli Twoja trasa jest zdefiniowana w ten sposób:

```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```

Możesz nadal wskazać `Illuminate\Http\Request` i uzyskać dostęp do swojego parametru `id`, definiując metodę kontrolera w następujący sposób:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Zaktualizuj danego użytkownika.
     */
    public function update(Request $request, string $id): RedirectResponse
    {
        // Zaktualizuj użytkownika...

        return redirect('/users');
    }
}
```
