# Laravel Folio

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
    - [Ścieżki stron / URI](#page-paths-uris)
    - [Routing subdomen](#subdomain-routing)
- [Tworzenie tras](#creating-routes)
    - [Zagnieżdżone trasy](#nested-routes)
    - [Trasy indeksowe](#index-routes)
- [Parametry tras](#route-parameters)
- [Wiązanie modeli tras](#route-model-binding)
    - [Modele usunięte programowo](#soft-deleted-models)
- [Hooki renderowania](#render-hooks)
- [Nazwane trasy](#named-routes)
- [Middleware](#middleware)
- [Cachowanie tras](#route-caching)

<a name="introduction"></a>
## Wprowadzenie

[Laravel Folio](https://github.com/laravel/folio) to potężny router oparty na stronach, zaprojektowany w celu uproszczenia routingu w aplikacjach Laravel. Dzięki Laravel Folio generowanie trasy staje się tak proste jak tworzenie szablonu Blade w katalogu `resources/views/pages` Twojej aplikacji.

Na przykład, aby utworzyć stronę dostępną pod adresem URL `/greeting`, wystarczy utworzyć plik `greeting.blade.php` w katalogu `resources/views/pages` Twojej aplikacji:

```php
<div>
    Hello World
</div>
```

<a name="installation"></a>
## Instalacja

Aby rozpocząć, zainstaluj Folio w swoim projekcie za pomocą menedżera pakietów Composer:

```shell
composer require laravel/folio
```

Po zainstalowaniu Folio możesz wykonać polecenie Artisan `folio:install`, które zainstaluje dostawcę usług Folio w Twojej aplikacji. Ten dostawca usług rejestruje katalog, w którym Folio będzie szukać tras / stron:

```shell
php artisan folio:install
```

<a name="page-paths-uris"></a>
### Ścieżki stron / URI

Domyślnie Folio serwuje strony z katalogu `resources/views/pages` Twojej aplikacji, ale możesz dostosować te katalogi w metodzie `boot` dostawcy usług Folio.

Na przykład, czasami może być wygodne określenie wielu ścieżek Folio w tej samej aplikacji Laravel. Możesz chcieć mieć oddzielny katalog stron Folio dla obszaru "admin" Twojej aplikacji, używając innego katalogu dla pozostałych stron aplikacji.

Możesz to osiągnąć za pomocą metod `Folio::path` i `Folio::uri`. Metoda `path` rejestruje katalog, który Folio będzie skanować w poszukiwaniu stron podczas routowania przychodzących żądań HTTP, podczas gdy metoda `uri` określa "bazowy URI" dla tego katalogu stron:

```php
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages/guest'))->uri('/');

Folio::path(resource_path('views/pages/admin'))
    ->uri('/admin')
    ->middleware([
        '*' => [
            'auth',
            'verified',

            // ...
        ],
    ]);
```

<a name="subdomain-routing"></a>
### Routing subdomen

Możesz również kierować do stron na podstawie subdomeny przychodzącego żądania. Na przykład, możesz chcieć kierować żądania z `admin.example.com` do innego katalogu stron niż pozostałe strony Folio. Możesz to osiągnąć wywołując metodę `domain` po wywołaniu metody `Folio::path`:

```php
use Laravel\Folio\Folio;

Folio::domain('admin.example.com')
    ->path(resource_path('views/pages/admin'));
```

Metoda `domain` pozwala również przechwytywać części domeny lub subdomeny jako parametry. Te parametry zostaną wstrzyknięte do szablonu Twojej strony:

```php
use Laravel\Folio\Folio;

Folio::domain('{account}.example.com')
    ->path(resource_path('views/pages/admin'));
```

<a name="creating-routes"></a>
## Tworzenie tras

Możesz utworzyć trasę Folio umieszczając szablon Blade w dowolnym z zamontowanych katalogów Folio. Domyślnie Folio montuje katalog `resources/views/pages`, ale możesz dostosować te katalogi w metodzie `boot` dostawcy usług Folio.

Po umieszczeniu szablonu Blade w zamontowanym katalogu Folio możesz natychmiast uzyskać do niego dostęp przez przeglądarkę. Na przykład, strona umieszczona w `pages/schedule.blade.php` może być dostępna w przeglądarce pod adresem `http://example.com/schedule`.

Aby szybko wyświetlić listę wszystkich stron / tras Folio, możesz wywołać polecenie Artisan `folio:list`:

```shell
php artisan folio:list
```

<a name="nested-routes"></a>
### Zagnieżdżone trasy

Możesz utworzyć zagnieżdżoną trasę tworząc jeden lub więcej katalogów w jednym z katalogów Folio. Na przykład, aby utworzyć stronę dostępną przez `/user/profile`, utwórz szablon `profile.blade.php` w katalogu `pages/user`:

```shell
php artisan folio:page user/profile

# pages/user/profile.blade.php → /user/profile
```

<a name="index-routes"></a>
### Trasy indeksowe

Czasami możesz chcieć uczynić daną stronę "indeksem" katalogu. Umieszczając szablon `index.blade.php` w katalogu Folio, wszystkie żądania do głównego katalogu będą kierowane do tej strony:

```shell
php artisan folio:page index
# pages/index.blade.php → /

php artisan folio:page users/index
# pages/users/index.blade.php → /users
```

<a name="route-parameters"></a>
## Parametry tras

Często będziesz potrzebować, aby segmenty adresu URL przychodzącego żądania były wstrzykiwane do Twojej strony, abyś mógł z nimi wchodzić w interakcję. Na przykład, możesz potrzebować dostępu do "ID" użytkownika, którego profil jest wyświetlany. Aby to osiągnąć, możesz ująć segment nazwy pliku strony w nawiasy kwadratowe:

```shell
php artisan folio:page "users/[id]"

# pages/users/[id].blade.php → /users/1
```

Przechwycone segmenty mogą być dostępne jako zmienne w szablonie Blade:

```html
<div>
    User {{ $id }}
</div>
```

Aby przechwycić wiele segmentów, możesz poprzedzić ujęty segment trzema kropkami `...`:

```shell
php artisan folio:page "users/[...ids]"

# pages/users/[...ids].blade.php → /users/1/2/3
```

Podczas przechwytywania wielu segmentów, przechwycone segmenty zostaną wstrzyknięte do strony jako tablica:

```html
<ul>
    @foreach ($ids as $id)
        <li>User {{ $id }}</li>
    @endforeach
</ul>
```

<a name="route-model-binding"></a>
## Wiązanie modeli tras

Jeśli segment wieloznaczny nazwy pliku szablonu Twojej strony odpowiada jednemu z modeli Eloquent Twojej aplikacji, Folio automatycznie wykorzysta możliwości wiązania modeli tras Laravel i spróbuje wstrzyknąć rozwiązaną instancję modelu do Twojej strony:

```shell
php artisan folio:page "users/[User]"

# pages/users/[User].blade.php → /users/1
```

Przechwycone modele mogą być dostępne jako zmienne w szablonie Blade. Nazwa zmiennej modelu zostanie przekonwertowana na "camel case":

```html
<div>
    User {{ $user->id }}
</div>
```

#### Dostosowywanie klucza

Czasami możesz chcieć rozwiązać powiązane modele Eloquent używając kolumny innej niż `id`. Aby to zrobić, możesz określić kolumnę w nazwie pliku strony. Na przykład, strona z nazwą pliku `[Post:slug].blade.php` spróbuje rozwiązać powiązany model przez kolumnę `slug` zamiast kolumny `id`.

W systemie Windows powinieneś użyć `-` do oddzielenia nazwy modelu od klucza: `[Post-slug].blade.php`.

#### Lokalizacja modelu

Domyślnie Folio będzie szukać Twojego modelu w katalogu `app/Models` Twojej aplikacji. Jednak w razie potrzeby możesz określić w pełni kwalifikowaną nazwę klasy modelu w nazwie pliku szablonu:

```shell
php artisan folio:page "users/[.App.Models.User]"

# pages/users/[.App.Models.User].blade.php → /users/1
```

<a name="soft-deleted-models"></a>
### Modele usunięte programowo

Domyślnie modele, które zostały usunięte programowo (soft deleted), nie są pobierane podczas rozwiązywania niejawnych wiązań modeli. Jednak jeśli chcesz, możesz poinstruować Folio, aby pobierał usunięte programowo modele wywołując funkcję `withTrashed` w szablonie strony:

```php
<?php

use function Laravel\Folio\{withTrashed};

withTrashed();

?>

<div>
    User {{ $user->id }}
</div>
```

<a name="render-hooks"></a>
## Hooki renderowania

Domyślnie Folio zwróci zawartość szablonu Blade strony jako odpowiedź na przychodzące żądanie. Jednak możesz dostosować odpowiedź wywołując funkcję `render` w szablonie strony.

Funkcja `render` przyjmuje domknięcie, które otrzyma instancję `View` renderowaną przez Folio, umożliwiając dodanie dodatkowych danych do widoku lub dostosowanie całej odpowiedzi. Oprócz otrzymania instancji `View`, wszelkie dodatkowe parametry trasy lub wiązania modeli również zostaną dostarczone do domknięcia `render`:

```php
<?php

use App\Models\Post;
use Illuminate\Support\Facades\Auth;
use Illuminate\View\View;

use function Laravel\Folio\render;

render(function (View $view, Post $post) {
    if (! Auth::user()->can('view', $post)) {
        return response('Unauthorized', 403);
    }

    return $view->with('photos', $post->author->photos);
}); ?>

<div>
    {{ $post->content }}
</div>

<div>
    This author has also taken {{ count($photos) }} photos.
</div>
```

<a name="named-routes"></a>
## Nazwane trasy

Możesz określić nazwę dla trasy danej strony używając funkcji `name`:

```php
<?php

use function Laravel\Folio\name;

name('users.index');
```

Podobnie jak w nazwanych trasach Laravel, możesz użyć funkcji `route` do generowania adresów URL do stron Folio, którym przypisano nazwę:

```php
<a href="{{ route('users.index') }}">
    All Users
</a>
```

Jeśli strona ma parametry, możesz po prostu przekazać ich wartości do funkcji `route`:

```php
route('users.show', ['user' => $user]);
```

<a name="middleware"></a>
## Middleware

Możesz zastosować middleware do konkretnej strony wywołując funkcję `middleware` w szablonie strony:

```php
<?php

use function Laravel\Folio\{middleware};

middleware(['auth', 'verified']);

?>

<div>
    Dashboard
</div>
```

Lub, aby przypisać middleware do grupy stron, możesz połączyć metodę `middleware` po wywołaniu metody `Folio::path`.

Aby określić, do których stron należy zastosować middleware, tablica middleware może być indeksowana za pomocą odpowiednich wzorców URL stron, do których mają być zastosowane. Znak `*` może być używany jako znak wieloznaczny:

```php
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages'))->middleware([
    'admin/*' => [
        'auth',
        'verified',

        // ...
    ],
]);
```

Możesz uwzględnić domknięcia w tablicy middleware, aby zdefiniować inline, anonimowy middleware:

```php
use Closure;
use Illuminate\Http\Request;
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages'))->middleware([
    'admin/*' => [
        'auth',
        'verified',

        function (Request $request, Closure $next) {
            // ...

            return $next($request);
        },
    ],
]);
```

<a name="route-caching"></a>
## Cachowanie tras

Podczas używania Folio zawsze powinieneś korzystać z [możliwości cachowania tras Laravel](/docs/{{version}}/routing#route-caching). Folio nasłuchuje polecenia Artisan `route:cache`, aby upewnić się, że definicje stron Folio i nazwy tras są prawidłowo cache'owane dla maksymalnej wydajności.
