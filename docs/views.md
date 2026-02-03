# Widoki

- [Wprowadzenie](#introduction)
    - [Pisanie widoków w React / Vue](#writing-views-in-react-or-vue)
- [Tworzenie i renderowanie widoków](#creating-and-rendering-views)
    - [Zagnieżdżone katalogi widoków](#nested-view-directories)
    - [Tworzenie pierwszego dostępnego widoku](#creating-the-first-available-view)
    - [Sprawdzanie czy widok istnieje](#determining-if-a-view-exists)
- [Przekazywanie danych do widoków](#passing-data-to-views)
    - [Udostępnianie danych wszystkim widokom](#sharing-data-with-all-views)
- [Kompozytory widoków](#view-composers)
    - [Kreatory widoków](#view-creators)
- [Optymalizacja widoków](#optimizing-views)

<a name="introduction"></a>
## Wprowadzenie

Oczywiście nie jest praktyczne zwracanie całych ciągów dokumentów HTML bezpośrednio z tras i kontrolerów. Na szczęście widoki zapewniają wygodny sposób na umieszczenie całego HTML-a w osobnych plikach.

Widoki oddzielają logikę kontrolera / aplikacji od logiki prezentacji i są przechowywane w katalogu `resources/views`. Podczas używania Laravel, szablony widoków są zwykle pisane przy użyciu [języka szablonów Blade](/docs/{{version}}/blade). Prosty widok może wyglądać mniej więcej tak:

```blade
<!-- View stored in resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

Ponieważ ten widok jest przechowywany w `resources/views/greeting.blade.php`, możemy go zwrócić używając globalnej funkcji pomocniczej `view` w następujący sposób:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

> [!NOTE]
> Szukasz więcej informacji o tym, jak pisać szablony Blade? Sprawdź pełną [dokumentację Blade](/docs/{{version}}/blade), aby rozpocząć.

<a name="writing-views-in-react-or-vue"></a>
### Pisanie widoków w React / Vue

Zamiast pisać swoje szablony frontendowe w PHP za pomocą Blade, wielu programistów zaczęło preferować pisanie swoich szablonów przy użyciu React lub Vue. Laravel czyni to bezbolesnym dzięki [Inertia](https://inertiajs.com/), bibliotece, która ułatwia połączenie frontendu React / Vue z backendem Laravel bez typowych złożoności budowania SPA.

Nasze [zestawy startowe aplikacji React i Vue](/docs/{{version}}/starter-kits) dają Ci świetny punkt wyjścia dla Twojej następnej aplikacji Laravel zasilanej przez Inertia.

<a name="creating-and-rendering-views"></a>
## Tworzenie i renderowanie widoków

Możesz utworzyć widok umieszczając plik z rozszerzeniem `.blade.php` w katalogu `resources/views` Twojej aplikacji lub używając komendy Artisan `make:view`:

```shell
php artisan make:view greeting
```

Rozszerzenie `.blade.php` informuje framework, że plik zawiera [szablon Blade](/docs/{{version}}/blade). Szablony Blade zawierają HTML oraz dyrektywy Blade, które pozwalają łatwo wyświetlać wartości, tworzyć instrukcje "if", iterować po danych i więcej.

Po utworzeniu widoku możesz go zwrócić z jednej z tras lub kontrolerów Twojej aplikacji używając globalnej funkcji pomocniczej `view`:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

Widoki mogą być również zwracane przy użyciu fasady `View`:

```php
use Illuminate\Support\Facades\View;

return View::make('greeting', ['name' => 'James']);
```

Jak widać, pierwszy argument przekazany do funkcji pomocniczej `view` odpowiada nazwie pliku widoku w katalogu `resources/views`. Drugi argument to tablica danych, które powinny być dostępne dla widoku. W tym przypadku przekazujemy zmienną `name`, która jest wyświetlana w widoku przy użyciu [składni Blade](/docs/{{version}}/blade).

<a name="nested-view-directories"></a>
### Zagnieżdżone katalogi widoków

Widoki mogą być również zagnieżdżone w podkatalogach katalogu `resources/views`. Notacja "kropkowa" może być użyta do odwoływania się do zagnieżdżonych widoków. Na przykład, jeśli Twój widok jest przechowywany w `resources/views/admin/profile.blade.php`, możesz go zwrócić z jednej z tras / kontrolerów Twojej aplikacji w następujący sposób:

```php
return view('admin.profile', $data);
```

> [!WARNING]
> Nazwy katalogów widoków nie powinny zawierać znaku `.`.

<a name="creating-the-first-available-view"></a>
### Tworzenie pierwszego dostępnego widoku

Używając metody `first` fasady `View`, możesz utworzyć pierwszy widok, który istnieje w danej tablicy widoków. Może to być przydatne, jeśli Twoja aplikacja lub pakiet pozwala na dostosowywanie lub nadpisywanie widoków:

```php
use Illuminate\Support\Facades\View;

return View::first(['custom.admin', 'admin'], $data);
```

<a name="determining-if-a-view-exists"></a>
### Sprawdzanie czy widok istnieje

Jeśli musisz sprawdzić, czy widok istnieje, możesz użyć fasady `View`. Metoda `exists` zwróci `true`, jeśli widok istnieje:

```php
use Illuminate\Support\Facades\View;

if (View::exists('admin.profile')) {
    // ...
}
```

<a name="passing-data-to-views"></a>
## Przekazywanie danych do widoków

Jak widziałeś w poprzednich przykładach, możesz przekazać tablicę danych do widoków, aby udostępnić te dane widokowi:

```php
return view('greetings', ['name' => 'Victoria']);
```

Podczas przekazywania informacji w ten sposób, dane powinny być tablicą z parami klucz / wartość. Po dostarczeniu danych do widoku możesz następnie uzyskać dostęp do każdej wartości w swoim widoku używając kluczy danych, takich jak `<?php echo $name; ?>`.

Jako alternatywę dla przekazywania kompletnej tablicy danych do funkcji pomocniczej `view`, możesz użyć metody `with`, aby dodać poszczególne fragmenty danych do widoku. Metoda `with` zwraca instancję obiektu widoku, dzięki czemu możesz kontynuować łańcuchowanie metod przed zwróceniem widoku:

```php
return view('greeting')
    ->with('name', 'Victoria')
    ->with('occupation', 'Astronaut');
```

<a name="sharing-data-with-all-views"></a>
### Udostępnianie danych wszystkim widokom

Czasami możesz potrzebować udostępnić dane wszystkim widokom renderowanym przez Twoją aplikację. Możesz to zrobić używając metody `share` fasady `View`. Zazwyczaj powinieneś umieszczać wywołania metody `share` w metodzie `boot` dostawcy usług. Możesz dodać je do klasy `App\Providers\AppServiceProvider` lub wygenerować osobnego dostawcę usług, aby je przechowywać:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;

class AppServiceProvider extends ServiceProvider
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
        View::share('key', 'value');
    }
}
```

<a name="view-composers"></a>
## Kompozytory widoków

Kompozytory widoków to wywołania zwrotne lub metody klas, które są wywoływane, gdy widok jest renderowany. Jeśli masz dane, które chcesz powiązać z widokiem za każdym razem, gdy widok jest renderowany, kompozytor widoków może pomóc Ci zorganizować tę logikę w jednym miejscu. Kompozytory widoków mogą okazać się szczególnie przydatne, jeśli ten sam widok jest zwracany przez wiele tras lub kontrolerów w Twojej aplikacji i zawsze potrzebuje określonego fragmentu danych.

Zazwyczaj kompozytory widoków będą rejestrowane w jednym z [dostawców usług](/docs/{{version}}/providers) Twojej aplikacji. W tym przykładzie założymy, że `App\Providers\AppServiceProvider` będzie zawierać tę logikę.

Użyjemy metody `composer` fasady `View`, aby zarejestrować kompozytor widoków. Laravel nie zawiera domyślnego katalogu dla kompozytorów widoków opartych na klasach, więc możesz je organizować w dowolny sposób. Na przykład możesz utworzyć katalog `app/View/Composers`, aby przechowywać wszystkie kompozytory widoków Twojej aplikacji:

```php
<?php

namespace App\Providers;

use App\View\Composers\ProfileComposer;
use Illuminate\Support\Facades;
use Illuminate\Support\ServiceProvider;
use Illuminate\View\View;

class AppServiceProvider extends ServiceProvider
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
        // Using class-based composers...
        Facades\View::composer('profile', ProfileComposer::class);

        // Using closure-based composers...
        Facades\View::composer('welcome', function (View $view) {
            // ...
        });

        Facades\View::composer('dashboard', function (View $view) {
            // ...
        });
    }
}
```

Teraz, gdy zarejestrowaliśmy kompozytor, metoda `compose` klasy `App\View\Composers\ProfileComposer` będzie wykonywana za każdym razem, gdy widok `profile` jest renderowany. Spójrzmy na przykład klasy kompozytora:

```php
<?php

namespace App\View\Composers;

use App\Repositories\UserRepository;
use Illuminate\View\View;

class ProfileComposer
{
    /**
     * Create a new profile composer.
     */
    public function __construct(
        protected UserRepository $users,
    ) {}

    /**
     * Bind data to the view.
     */
    public function compose(View $view): void
    {
        $view->with('count', $this->users->count());
    }
}
```

Jak widać, wszystkie kompozytory widoków są rozwiązywane przez [kontener usług](/docs/{{version}}/container), więc możesz wpisać wskazówki typu dla dowolnych zależności, których potrzebujesz w konstruktorze kompozytora.

<a name="attaching-a-composer-to-multiple-views"></a>
#### Dołączanie kompozytora do wielu widoków

Możesz dołączyć kompozytor widoku do wielu widoków jednocześnie, przekazując tablicę widoków jako pierwszy argument do metody `composer`:

```php
use App\Views\Composers\MultiComposer;
use Illuminate\Support\Facades\View;

View::composer(
    ['profile', 'dashboard'],
    MultiComposer::class
);
```

Metoda `composer` akceptuje również znak `*` jako symbol wieloznaczny, pozwalając dołączyć kompozytor do wszystkich widoków:

```php
use Illuminate\Support\Facades;
use Illuminate\View\View;

Facades\View::composer('*', function (View $view) {
    // ...
});
```

<a name="view-creators"></a>
### Kreatory widoków

"Kreatory" widoków są bardzo podobne do kompozytorów widoków; jednak są wykonywane natychmiast po utworzeniu instancji widoku, zamiast czekać, aż widok ma być renderowany. Aby zarejestrować kreator widoku, użyj metody `creator`:

```php
use App\View\Creators\ProfileCreator;
use Illuminate\Support\Facades\View;

View::creator('profile', ProfileCreator::class);
```

<a name="optimizing-views"></a>
## Optymalizacja widoków

Domyślnie widoki szablonów Blade są kompilowane na żądanie. Gdy wykonywane jest żądanie renderujące widok, Laravel określi, czy istnieje skompilowana wersja widoku. Jeśli plik istnieje, Laravel określi następnie, czy nieskompilowany widok został zmodyfikowany później niż skompilowany widok. Jeśli skompilowany widok nie istnieje lub nieskompilowany widok został zmodyfikowany, Laravel przekompiluje widok.

Kompilowanie widoków podczas żądania może mieć niewielki negatywny wpływ na wydajność, więc Laravel zapewnia komendę Artisan `view:cache`, aby prekompilować wszystkie widoki używane przez Twoją aplikację. Dla zwiększonej wydajności możesz chcieć uruchomić tę komendę jako część procesu wdrażania:

```shell
php artisan view:cache
```

Możesz użyć komendy `view:clear`, aby wyczyścić pamięć podręczną widoków:

```shell
php artisan view:clear
```
