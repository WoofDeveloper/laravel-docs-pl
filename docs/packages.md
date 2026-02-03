# Tworzenie Pakietów

- [Wprowadzenie](#introduction)
    - [Uwaga o Fasadach](#a-note-on-facades)
- [Wykrywanie Pakietów](#package-discovery)
- [Dostawcy Usług](#service-providers)
- [Zasoby](#resources)
    - [Konfiguracja](#configuration)
    - [Trasy](#routes)
    - [Migracje](#migrations)
    - [Pliki Językowe](#language-files)
    - [Widoki](#views)
    - [Komponenty Widoków](#view-components)
    - [Polecenie Artisan "About"](#about-artisan-command)
- [Polecenia](#commands)
    - [Polecenia Optymalizacji](#optimize-commands)
    - [Polecenia Przeładowania](#reload-commands)
- [Zasoby Publiczne](#public-assets)
- [Publikowanie Grup Plików](#publishing-file-groups)

<a name="introduction"></a>
## Wprowadzenie

Pakiety są podstawowym sposobem dodawania funkcjonalności do Laravel. Pakiety mogą być czymkolwiek, od świetnego sposobu pracy z datami, takiego jak [Carbon](https://github.com/briannesbitt/Carbon), po pakiet pozwalający powiązać pliki z modelami Eloquent, jak [Laravel Media Library](https://github.com/spatie/laravel-medialibrary) od Spatie.

Istnieją różne rodzaje pakietów. Niektóre pakiety są samodzielne, co oznacza, że działają z dowolnym frameworkiem PHP. Carbon i Pest to przykłady pakietów samodzielnych. Każdy z tych pakietów może być używany z Laravel poprzez dodanie ich do pliku `composer.json`.

Z drugiej strony, inne pakiety są specjalnie przeznaczone do użycia z Laravel. Takie pakiety mogą mieć trasy, kontrolery, widoki i konfigurację specjalnie przygotowaną do rozszerzenia aplikacji Laravel. Ten przewodnik skupia się głównie na rozwoju pakietów przeznaczonych specjalnie dla Laravel.

<a name="a-note-on-facades"></a>
### Uwaga o Fasadach

Podczas pisania aplikacji Laravel, zazwyczaj nie ma znaczenia, czy używasz kontraktów czy fasad, ponieważ oba zapewniają zasadniczo równe poziomy testowalności. Jednak podczas pisania pakietów, Twój pakiet zazwyczaj nie będzie miał dostępu do wszystkich pomocników testowych Laravel. Jeśli chciałbyś móc pisać testy pakietu tak, jakby pakiet był zainstalowany wewnątrz typowej aplikacji Laravel, możesz użyć pakietu [Orchestral Testbench](https://github.com/orchestral/testbench).

<a name="package-discovery"></a>
## Wykrywanie Pakietów

Plik `bootstrap/providers.php` aplikacji Laravel zawiera listę dostawców usług, którzy powinni zostać załadowani przez Laravel. Jednak zamiast wymagać od użytkowników ręcznego dodawania Twojego dostawcy usług do listy, możesz zdefiniować dostawcę w sekcji `extra` pliku `composer.json` Twojego pakietu, aby był automatycznie ładowany przez Laravel. Oprócz dostawców usług, możesz również wylistować wszystkie [fasady](/docs/{{version}}/facades), które chciałbyś zarejestrować:

```json
"extra": {
    "laravel": {
        "providers": [
            "Barryvdh\\Debugbar\\ServiceProvider"
        ],
        "aliases": {
            "Debugbar": "Barryvdh\\Debugbar\\Facade"
        }
    }
},
```

Po skonfigurowaniu pakietu do wykrywania, Laravel automatycznie zarejestruje jego dostawców usług i fasady podczas instalacji, tworząc wygodne doświadczenie instalacyjne dla użytkowników Twojego pakietu.

<a name="opting-out-of-package-discovery"></a>
#### Rezygnacja z Wykrywania Pakietów

Jeśli jesteś użytkownikiem pakietu i chciałbyś wyłączyć wykrywanie pakietów dla danego pakietu, możesz wymienić nazwę pakietu w sekcji `extra` pliku `composer.json` Twojej aplikacji:

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "barryvdh/laravel-debugbar"
        ]
    }
},
```

Możesz wyłączyć wykrywanie pakietów dla wszystkich pakietów używając znaku `*` wewnątrz dyrektywy `dont-discover` Twojej aplikacji:

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "*"
        ]
    }
},
```

<a name="service-providers"></a>
## Dostawcy Usług

[Dostawcy usług](/docs/{{version}}/providers) są punktem połączenia między Twoim pakietem a Laravel. Dostawca usług jest odpowiedzialny za wiązanie elementów do [kontenera usług](/docs/{{version}}/container) Laravel i informowanie Laravel, gdzie załadować zasoby pakietu takie jak widoki, konfiguracja i pliki językowe.

Dostawca usług rozszerza klasę `Illuminate\Support\ServiceProvider` i zawiera dwie metody: `register` i `boot`. Bazowa klasa `ServiceProvider` znajduje się w pakiecie Composer `illuminate/support`, który powinieneś dodać do zależności swojego pakietu. Aby dowiedzieć się więcej o strukturze i przeznaczeniu dostawców usług, sprawdź [ich dokumentację](/docs/{{version}}/providers).

<a name="resources"></a>
## Zasoby

<a name="configuration"></a>
### Konfiguracja

Zazwyczaj będziesz musiał opublikować plik konfiguracyjny swojego pakietu do katalogu `config` aplikacji. Pozwoli to użytkownikom Twojego pakietu łatwo nadpisać Twoje domyślne opcje konfiguracyjne. Aby umożliwić publikację plików konfiguracyjnych, wywołaj metodę `publishes` z metody `boot` Twojego dostawcy usług:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->publishes([
        __DIR__.'/../config/courier.php' => config_path('courier.php'),
    ]);
}
```

Teraz, gdy użytkownicy Twojego pakietu wykonają polecenie Laravel `vendor:publish`, Twój plik zostanie skopiowany do określonej lokalizacji publikacji. Po opublikowaniu konfiguracji, jej wartości mogą być dostępne jak każdy inny plik konfiguracyjny:

```php
$value = config('courier.option');
```

> [!WARNING]
> Nie powinieneś definiować domknięć w plikach konfiguracyjnych. Nie mogą one być poprawnie serializowane, gdy użytkownicy wykonują polecenie Artisan `config:cache`.

<a name="default-package-configuration"></a>
#### Domyślna Konfiguracja Pakietu

Możesz również scalić własny plik konfiguracyjny pakietu z opublikowaną kopią aplikacji. Pozwoli to użytkownikom zdefiniować tylko te opcje, które faktycznie chcą nadpisać w opublikowanej kopii pliku konfiguracyjnego. Aby scalić wartości pliku konfiguracyjnego, użyj metody `mergeConfigFrom` w metodzie `register` dostawcy usług.

Metoda `mergeConfigFrom` przyjmuje jako pierwszy argument ścieżkę do pliku konfiguracyjnego Twojego pakietu, a jako drugi argument nazwę kopii pliku konfiguracyjnego aplikacji:

```php
/**
 * Register any package services.
 */
public function register(): void
{
    $this->mergeConfigFrom(
        __DIR__.'/../config/courier.php', 'courier'
    );
}
```

> [!WARNING]
> Ta metoda scala tylko pierwszy poziom tablicy konfiguracyjnej. Jeśli użytkownicy częściowo zdefiniują wielowymiarową tablicę konfiguracyjną, brakujące opcje nie zostaną scalone.

<a name="routes"></a>
### Trasy

Jeśli Twój pakiet zawiera trasy, możesz je załadować używając metody `loadRoutesFrom`. Ta metoda automatycznie określi, czy trasy aplikacji są w pamięci podręcznej i nie załaduje Twojego pliku tras, jeśli trasy zostały już zapisane w pamięci podręcznej:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->loadRoutesFrom(__DIR__.'/../routes/web.php');
}
```

<a name="migrations"></a>
### Migracje

Jeśli Twój pakiet zawiera [migracje bazy danych](/docs/{{version}}/migrations), możesz użyć metody `publishesMigrations`, aby poinformować Laravel, że dany katalog lub plik zawiera migracje. Gdy Laravel publikuje migracje, automatycznie zaktualizuje znacznik czasu w nazwie pliku, aby odzwierciedlić bieżącą datę i godzinę:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->publishesMigrations([
        __DIR__.'/../database/migrations' => database_path('migrations'),
    ]);
}
```

<a name="language-files"></a>
### Pliki Językowe

Jeśli Twój pakiet zawiera [pliki językowe](/docs/{{version}}/localization), możesz użyć metody `loadTranslationsFrom`, aby poinformować Laravel, jak je załadować. Na przykład, jeśli Twój pakiet nazywa się `courier`, powinieneś dodać następujący kod do metody `boot` dostawcy usług:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->loadTranslationsFrom(__DIR__.'/../lang', 'courier');
}
```

Linie tłumaczeniowe pakietów są przywoływane przy użyciu konwencji składni `package::file.line`. Więc możesz załadować linię `welcome` pakietu `courier` z pliku `messages` w następujący sposób:

```php
echo trans('courier::messages.welcome');
```

Możesz zarejestrować pliki tłumaczeń JSON dla swojego pakietu używając metody `loadJsonTranslationsFrom`. Ta metoda przyjmuje ścieżkę do katalogu zawierającego pliki tłumaczeń JSON Twojego pakietu:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->loadJsonTranslationsFrom(__DIR__.'/../lang');
}
```

<a name="publishing-language-files"></a>
#### Publikowanie Plików Językowych

Jeśli chciałbyś opublikować pliki językowe swojego pakietu do katalogu `lang/vendor` aplikacji, możesz użyć metody `publishes` dostawcy usług. Metoda `publishes` przyjmuje tablicę ścieżek pakietu i ich pożądanych lokalizacji publikacji. Na przykład, aby opublikować pliki językowe dla pakietu `courier`, możesz zrobić następująco:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->loadTranslationsFrom(__DIR__.'/../lang', 'courier');

    $this->publishes([
        __DIR__.'/../lang' => $this->app->langPath('vendor/courier'),
    ]);
}
```

Teraz, gdy użytkownicy Twojego pakietu wykonają polecenie Artisan `vendor:publish` Laravel, pliki językowe Twojego pakietu zostaną opublikowane w określonej lokalizacji publikacji.

<a name="views"></a>
### Widoki

Aby zarejestrować [widoki](/docs/{{version}}/views) swojego pakietu w Laravel, musisz powiedzieć Laravel, gdzie znajdują się widoki. Możesz to zrobić używając metody `loadViewsFrom` dostawcy usług. Metoda `loadViewsFrom` przyjmuje dwa argumenty: ścieżkę do szablonów widoków i nazwę Twojego pakietu. Na przykład, jeśli nazwa Twojego pakietu to `courier`, dodasz następujący kod do metody `boot` dostawcy usług:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->loadViewsFrom(__DIR__.'/../resources/views', 'courier');
}
```

Widoki pakietów są przywoływane przy użyciu konwencji składni `package::view`. Więc, gdy ścieżka widoku jest zarejestrowana w dostawcy usług, możesz załadować widok `dashboard` z pakietu `courier` w następujący sposób:

```php
Route::get('/dashboard', function () {
    return view('courier::dashboard');
});
```

<a name="overriding-package-views"></a>
#### Nadpisywanie Widoków Pakietu

Kiedy używasz metody `loadViewsFrom`, Laravel faktycznie rejestruje dwie lokalizacje dla Twoich widoków: katalog `resources/views/vendor` aplikacji i katalog, który określiłeś. Więc, używając pakietu `courier` jako przykładu, Laravel najpierw sprawdzi, czy niestandardowa wersja widoku została umieszczona w katalogu `resources/views/vendor/courier` przez programistę. Następnie, jeśli widok nie został dostosowany, Laravel przeszuka katalog widoków pakietu określony w wywołaniu `loadViewsFrom`. Umożliwia to użytkownikom pakietu łatwe dostosowanie/nadpisanie widoków Twojego pakietu.

<a name="publishing-views"></a>
#### Publikowanie Widoków

Jeśli chciałbyś udostępnić swoje widoki do publikacji w katalogu `resources/views/vendor` aplikacji, możesz użyć metody `publishes` dostawcy usług. Metoda `publishes` przyjmuje tablicę ścieżek widoków pakietu i ich pożądanych lokalizacji publikacji:

```php
/**
 * Bootstrap the package services.
 */
public function boot(): void
{
    $this->loadViewsFrom(__DIR__.'/../resources/views', 'courier');

    $this->publishes([
        __DIR__.'/../resources/views' => resource_path('views/vendor/courier'),
    ]);
}
```

Teraz, gdy użytkownicy Twojego pakietu wykonają polecenie Artisan `vendor:publish` Laravel, widoki Twojego pakietu zostaną skopiowane do określonej lokalizacji publikacji.

<a name="view-components"></a>
### Komponenty Widoków

Jeśli budujesz pakiet wykorzystujący komponenty Blade lub umieszczasz komponenty w nietradycyjnych katalogach, będziesz musiał ręcznie zarejestrować klasę komponentu i jego alias znacznika HTML, aby Laravel wiedział, gdzie znaleźć komponent. Zazwyczaj powinieneś rejestrować swoje komponenty w metodzie `boot` dostawcy usług pakietu:

```php
use Illuminate\Support\Facades\Blade;
use VendorPackage\View\Components\AlertComponent;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::component('package-alert', AlertComponent::class);
}
```

Po zarejestrowaniu komponentu, może zostać renderowany przy użyciu jego aliasu znacznika:

```blade
<x-package-alert/>
```

<a name="autoloading-package-components"></a>
#### Automatyczne Ładowanie Komponentów Pakietu

Alternatywnie, możesz użyć metody `componentNamespace`, aby automatycznie ładować klasy komponentów według konwencji. Na przykład, pakiet `Nightshade` może mieć komponenty `Calendar` i `ColorPicker`, które znajdują się w przestrzeni nazw `Nightshade\Views\Components`:

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

Umożliwi to używanie komponentów pakietu przez ich przestrzeń nazw dostawcy przy użyciu składni `package-name::`:

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade automatycznie wykryje klasę połączoną z tym komponentem poprzez konwersję nazwy komponentu na format PascalCase. Podkatalogi są również obsługiwane przy użyciu notacji kropkowej.

<a name="anonymous-components"></a>
#### Komponenty Anonimowe

Jeśli Twój pakiet zawiera komponenty anonimowe, muszą one zostać umieszczone w katalogu `components` katalogu "views" Twojego pakietu (jak określono przez [metodę loadViewsFrom](#views)). Następnie możesz je renderować, poprzedzając nazwę komponentu przestrzenią nazw widoku pakietu:

```blade
<x-courier::alert />
```

<a name="about-artisan-command"></a>
### Polecenie Artisan "About"

Wbudowane polecenie Artisan `about` Laravel dostarcza podsumowanie środowiska i konfiguracji aplikacji. Pakiety mogą dodawać dodatkowe informacje do wyjścia tego polecenia za pośrednictwem klasy `AboutCommand`. Zazwyczaj te informacje mogą być dodane z metody `boot` dostawcy usług pakietu:

```php
use Illuminate\Foundation\Console\AboutCommand;

/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    AboutCommand::add('My Package', fn () => ['Version' => '1.0.0']);
}
```

<a name="commands"></a>
## Polecenia

Aby zarejestrować polecenia Artisan swojego pakietu w Laravel, możesz użyć metody `commands`. Ta metoda oczekuje tablicy nazw klas poleceń. Po zarejestrowaniu poleceń, możesz je wykonywać używając [Artisan CLI](/docs/{{version}}/artisan):

```php
use Courier\Console\Commands\InstallCommand;
use Courier\Console\Commands\NetworkCommand;

/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    if ($this->app->runningInConsole()) {
        $this->commands([
            InstallCommand::class,
            NetworkCommand::class,
        ]);
    }
}
```

<a name="optimize-commands"></a>
### Polecenia Optymalizacji

Polecenie [optimize](/docs/{{version}}/deployment#optimization) Laravel zapisuje w pamięci podręcznej konfigurację, zdarzenia, trasy i widoki aplikacji. Używając metody `optimizes`, możesz zarejestrować własne polecenia Artisan pakietu, które powinny zostać wywołane podczas wykonywania poleceń `optimize` i `optimize:clear`:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    if ($this->app->runningInConsole()) {
        $this->optimizes(
            optimize: 'package:optimize',
            clear: 'package:clear-optimizations',
        );
    }
}
```

<a name="reload-commands"></a>
### Polecenia Przeładowania

Polecenie [reload](/docs/{{version}}/deployment#reloading-services) Laravel kończy działanie wszystkich uruchomionych usług, aby mogły zostać automatycznie ponownie uruchomione przez monitor procesów systemowych. Używając metody `reloads`, możesz zarejestrować własne polecenia Artisan pakietu, które powinny zostać wywołane podczas wykonywania polecenia `reload`:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    if ($this->app->runningInConsole()) {
        $this->reloads('package:reload');
    }
}
```

<a name="public-assets"></a>
## Zasoby Publiczne

Twój pakiet może mieć zasoby takie jak JavaScript, CSS i obrazy. Aby opublikować te zasoby do katalogu `public` aplikacji, użyj metody `publishes` dostawcy usług. W tym przykładzie dodamy również tag grupy zasobów `public`, który może być użyty do łatwego publikowania grup powiązanych zasobów:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->publishes([
        __DIR__.'/../public' => public_path('vendor/courier'),
    ], 'public');
}
```

Teraz, gdy użytkownicy Twojego pakietu wykonają polecenie `vendor:publish`, Twoje zasoby zostaną skopiowane do określonej lokalizacji publikacji. Ponieważ użytkownicy zazwyczaj będą musieli nadpisywać zasoby za każdym razem, gdy pakiet zostanie zaktualizowany, mogą użyć flagi `--force`:

```shell
php artisan vendor:publish --tag=public --force
```

<a name="publishing-file-groups"></a>
## Publikowanie Grup Plików

Możesz chcieć publikować grupy zasobów i zasobów pakietu osobno. Na przykład, możesz chcieć pozwolić użytkownikom publikować pliki konfiguracyjne Twojego pakietu bez konieczności publikowania zasobów pakietu. Możesz to zrobić, "tagując" je podczas wywoływania metody `publishes` z dostawcy usług pakietu. Na przykład, użyjmy tagów do zdefiniowania dwóch grup publikacji dla pakietu `courier` (`courier-config` i `courier-migrations`) w metodzie `boot` dostawcy usług pakietu:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->publishes([
        __DIR__.'/../config/package.php' => config_path('package.php')
    ], 'courier-config');

    $this->publishesMigrations([
        __DIR__.'/../database/migrations/' => database_path('migrations')
    ], 'courier-migrations');
}
```

Teraz użytkownicy mogą publikować te grupy osobno, odwołując się do ich tagu podczas wykonywania polecenia `vendor:publish`:

```shell
php artisan vendor:publish --tag=courier-config
```

Użytkownicy mogą również opublikować wszystkie pliki możliwe do opublikowania zdefiniowane przez dostawcę usług Twojego pakietu, używając flagi `--provider`:

```shell
php artisan vendor:publish --provider="Your\Package\ServiceProvider"
```
