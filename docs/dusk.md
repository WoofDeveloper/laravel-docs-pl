# Laravel Dusk

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
    - [Zarządzanie instalacjami ChromeDriver](#managing-chromedriver-installations)
    - [Używanie innych przeglądarek](#using-other-browsers)
- [Pierwsze kroki](#getting-started)
    - [Generowanie testów](#generating-tests)
    - [Resetowanie bazy danych po każdym teście](#resetting-the-database-after-each-test)
    - [Uruchamianie testów](#running-tests)
    - [Obsługa środowiska](#environment-handling)
- [Podstawy przeglądarki](#browser-basics)
    - [Tworzenie przeglądarek](#creating-browsers)
    - [Nawigacja](#navigation)
    - [Zmiana rozmiaru okien przeglądarki](#resizing-browser-windows)
    - [Makra przeglądarki](#browser-macros)
    - [Uwierzytelnianie](#authentication)
    - [Ciasteczka](#cookies)
    - [Wykonywanie JavaScript](#executing-javascript)
    - [Robienie zrzutu ekranu](#taking-a-screenshot)
    - [Zapisywanie wyjścia konsoli na dysk](#storing-console-output-to-disk)
    - [Zapisywanie źródła strony na dysk](#storing-page-source-to-disk)
- [Interakcja z elementami](#interacting-with-elements)
    - [Selektory Dusk](#dusk-selectors)
    - [Tekst, wartości i atrybuty](#text-values-and-attributes)
    - [Interakcja z formularzami](#interacting-with-forms)
    - [Dołączanie plików](#attaching-files)
    - [Naciskanie przycisków](#pressing-buttons)
    - [Klikanie linków](#clicking-links)
    - [Używanie klawiatury](#using-the-keyboard)
    - [Używanie myszy](#using-the-mouse)
    - [Okna dialogowe JavaScript](#javascript-dialogs)
    - [Interakcja z ramkami inline](#interacting-with-iframes)
    - [Określanie zakresu selektorów](#scoping-selectors)
    - [Oczekiwanie na elementy](#waiting-for-elements)
    - [Przewijanie elementu do widoku](#scrolling-an-element-into-view)
- [Dostępne asercje](#available-assertions)
- [Strony](#pages)
    - [Generowanie stron](#generating-pages)
    - [Konfigurowanie stron](#configuring-pages)
    - [Nawigowanie do stron](#navigating-to-pages)
    - [Selektory skrótowe](#shorthand-selectors)
    - [Metody stron](#page-methods)
- [Komponenty](#components)
    - [Generowanie komponentów](#generating-components)
    - [Używanie komponentów](#using-components)
- [Ciągła integracja](#continuous-integration)
    - [Heroku CI](#running-tests-on-heroku-ci)
    - [Travis CI](#running-tests-on-travis-ci)
    - [GitHub Actions](#running-tests-on-github-actions)
    - [Chipper CI](#running-tests-on-chipper-ci)

<a name="introduction"></a>
## Wprowadzenie

> [!WARNING]
> [Pest 4](https://pestphp.com/) zawiera teraz automatyczne testowanie przeglądarki, które oferuje znaczącą poprawę wydajności i użyteczności w porównaniu do Laravel Dusk. Dla nowych projektów zalecamy używanie Pest do testowania przeglądarki.

[Laravel Dusk](https://github.com/laravel/dusk) zapewnia ekspresyjne, łatwe w użyciu API do automatyzacji przeglądarki i testowania. Domyślnie Dusk nie wymaga instalacji JDK ani Selenium na lokalnym komputerze. Zamiast tego Dusk używa samodzielnej instalacji [ChromeDriver](https://sites.google.com/chromium.org/driver). Możesz jednak użyć dowolnego innego sterownika kompatybilnego z Selenium.

<a name="installation"></a>
## Instalacja

Aby rozpocząć, powinieneś zainstalować [Google Chrome](https://www.google.com/chrome) i dodać zależność Composer `laravel/dusk` do swojego projektu:

```shell
composer require laravel/dusk --dev
```

> [!WARNING]
> Jeśli ręcznie rejestrujesz dostawcę usług Dusk, **nigdy** nie powinieneś go rejestrować w środowisku produkcyjnym, ponieważ może to prowadzić do tego, że dowolni użytkownicy będą mogli uwierzytelnić się w Twojej aplikacji.

Po zainstalowaniu pakietu Dusk wykonaj komendę Artisan `dusk:install`. Komenda `dusk:install` utworzy katalog `tests/Browser`, przykładowy test Dusk i zainstaluje binarne pliki Chrome Driver dla Twojego systemu operacyjnego:

```shell
php artisan dusk:install
```

Następnie ustaw zmienną środowiskową `APP_URL` w pliku `.env` Twojej aplikacji. Ta wartość powinna odpowiadać adresowi URL, którego używasz do dostępu do aplikacji w przeglądarce.

> [!NOTE]
> Jeśli używasz [Laravel Sail](/docs/{{version}}/sail) do zarządzania lokalnym środowiskiem programistycznym, zapoznaj się również z dokumentacją Sail dotyczącą [konfigurowania i uruchamiania testów Dusk](/docs/{{version}}/sail#laravel-dusk).

<a name="managing-chromedriver-installations"></a>
### Zarządzanie instalacjami ChromeDriver

Jeśli chcesz zainstalować inną wersję ChromeDriver niż ta zainstalowana przez Laravel Dusk za pomocą komendy `dusk:install`, możesz użyć komendy `dusk:chrome-driver`:

```shell
# Zainstaluj najnowszą wersję ChromeDriver dla Twojego systemu operacyjnego...
php artisan dusk:chrome-driver

# Zainstaluj określoną wersję ChromeDriver dla Twojego systemu operacyjnego...
php artisan dusk:chrome-driver 86

# Zainstaluj określoną wersję ChromeDriver dla wszystkich obsługiwanych systemów operacyjnych...
php artisan dusk:chrome-driver --all

# Zainstaluj wersję ChromeDriver, która odpowiada wykrytej wersji Chrome / Chromium dla Twojego systemu operacyjnego...
php artisan dusk:chrome-driver --detect
```

> [!WARNING]
> Dusk wymaga, aby binaria `chromedriver` były wykonywalne. Jeśli masz problemy z uruchomieniem Dusk, powinieneś upewnić się, że binaria są wykonywalne używając następującej komendy: `chmod -R 0755 vendor/laravel/dusk/bin/`.

<a name="using-other-browsers"></a>
### Używanie innych przeglądarek

Domyślnie Dusk używa Google Chrome i samodzielnej instalacji [ChromeDriver](https://sites.google.com/chromium.org/driver) do uruchamiania testów przeglądarki. Możesz jednak uruchomić własny serwer Selenium i uruchamiać testy na dowolnej przeglądarce.

Aby rozpocząć, otwórz plik `tests/DuskTestCase.php`, który jest podstawową klasą testową Dusk dla Twojej aplikacji. W tym pliku możesz usunąć wywołanie metody `startChromeDriver`. Spowoduje to, że Dusk przestanie automatycznie uruchamiać ChromeDriver:

```php
/**
 * Prepare for Dusk test execution.
 *
 * @beforeClass
 */
public static function prepare(): void
{
    // static::startChromeDriver();
}
```

Następnie możesz zmodyfikować metodę `driver`, aby połączyć się z wybranym adresem URL i portem. Ponadto możesz zmodyfikować "pożądane możliwości" (desired capabilities), które powinny zostać przekazane do WebDriver:

```php
use Facebook\WebDriver\Remote\RemoteWebDriver;

/**
 * Create the RemoteWebDriver instance.
 */
protected function driver(): RemoteWebDriver
{
    return RemoteWebDriver::create(
        'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
    );
}
```

<a name="getting-started"></a>
## Pierwsze kroki

<a name="generating-tests"></a>
### Generowanie testów

Aby wygenerować test Dusk, użyj komendy Artisan `dusk:make`. Wygenerowany test zostanie umieszczony w katalogu `tests/Browser`:

```shell
php artisan dusk:make LoginTest
```

<a name="resetting-the-database-after-each-test"></a>
### Resetowanie bazy danych po każdym teście

Większość testów, które napiszesz, będzie wchodzić w interakcję ze stronami, które pobierają dane z bazy danych Twojej aplikacji; jednak Twoje testy Dusk nigdy nie powinny używać traitu `RefreshDatabase`. Trait `RefreshDatabase` wykorzystuje transakcje bazodanowe, które nie będą miały zastosowania ani nie będą dostępne w różnych żądaniach HTTP. Zamiast tego masz dwie opcje: trait `DatabaseMigrations` i trait `DatabaseTruncation`.

<a name="reset-migrations"></a>
#### Używanie migracji bazy danych

Trait `DatabaseMigrations` uruchomi Twoje migracje bazy danych przed każdym testem. Jednak usuwanie i ponowne tworzenie tabel bazy danych dla każdego testu jest zazwyczaj wolniejsze niż obcinanie tabel:

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;

pest()->use(DatabaseMigrations::class);

//
```

```php tab=PHPUnit
<?php

namespace Tests\Browser;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;

    //
}
```

> [!WARNING]
> Bazy danych SQLite w pamięci nie mogą być używane podczas wykonywania testów Dusk. Ponieważ przeglądarka działa w swoim własnym procesie, nie będzie mogła uzyskać dostępu do baz danych w pamięci innych procesów.

<a name="reset-truncation"></a>
#### Używanie obcinania bazy danych

Trait `DatabaseTruncation` przeprowadzi migrację bazy danych podczas pierwszego testu, aby upewnić się, że tabele bazy danych zostały prawidłowo utworzone. Jednak w kolejnych testach tabele bazy danych będą po prostu obcięte - zapewniając przyspieszenie w porównaniu z ponownym uruchamianiem wszystkich migracji bazy danych:

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\DatabaseTruncation;
use Laravel\Dusk\Browser;

pest()->use(DatabaseTruncation::class);

//
```

```php tab=PHPUnit
<?php

namespace Tests\Browser;

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseTruncation;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseTruncation;

    //
}
```

Domyślnie ten trait obetnie wszystkie tabele z wyjątkiem tabeli `migrations`. Jeśli chcesz dostosować tabele, które powinny zostać obcięte, możesz zdefiniować właściwość `$tablesToTruncate` w swojej klasie testowej:

> [!NOTE]
> Jeśli używasz Pest, powinieneś zdefiniować właściwości lub metody w podstawowej klasie `DuskTestCase` lub w dowolnej klasie, którą rozszerza Twój plik testowy.

```php
/**
 * Wskazuje, które tabele powinny zostać obcięte.
 *
 * @var array
 */
protected $tablesToTruncate = ['users'];
```

Alternatywnie możesz zdefiniować właściwość `$exceptTables` w swojej klasie testowej, aby określić, które tabele powinny być wykluczone z obcinania:

```php
/**
 * Wskazuje, które tabele powinny być wykluczone z obcinania.
 *
 * @var array
 */
protected $exceptTables = ['users'];
```

Aby określić połączenia bazy danych, które powinny mieć obcięte swoje tabele, możesz zdefiniować właściwość `$connectionsToTruncate` w swojej klasie testowej:

```php
/**
 * Wskazuje, które połączenia powinny mieć obcięte swoje tabele.
 *
 * @var array
 */
protected $connectionsToTruncate = ['mysql'];
```

Jeśli chcesz wykonać kod przed lub po obcięciu bazy danych, możesz zdefiniować metody `beforeTruncatingDatabase` lub `afterTruncatingDatabase` w swojej klasie testowej:

```php
/**
 * Wykonaj wszelkie prace, które powinny się odbyć przed rozpoczęciem obcinania bazy danych.
 */
protected function beforeTruncatingDatabase(): void
{
    //
}

/**
 * Wykonaj wszelkie prace, które powinny się odbyć po zakończeniu obcinania bazy danych.
 */
protected function afterTruncatingDatabase(): void
{
    //
}
```

<a name="running-tests"></a>
### Uruchamianie testów

Aby uruchomić testy przeglądarki, wykonaj komendę Artisan `dusk`:

```shell
php artisan dusk
```

Jeśli miałeś niepowodzenia testów podczas ostatniego uruchomienia komendy `dusk`, możesz zaoszczędzić czas, ponownie uruchamiając najpierw nieudane testy za pomocą komendy `dusk:fails`:

```shell
php artisan dusk:fails
```

Komenda `dusk` akceptuje dowolny argument, który jest normalnie akceptowany przez test runner Pest / PHPUnit, na przykład pozwalając uruchomić testy tylko dla danej [grupy](https://docs.phpunit.de/en/10.5/annotations.html#group):

```shell
php artisan dusk --group=foo
```

> [!NOTE]
> If you are using [Laravel Sail](/docs/{{version}}/sail) to manage your local development environment, please consult the Sail documentation on [configuring and running Dusk tests](/docs/{{version}}/sail#laravel-dusk).

<a name="manually-starting-chromedriver"></a>
#### Ręczne uruchamianie ChromeDriver

Domyślnie Dusk automatycznie próbuje uruchomić ChromeDriver. Jeśli to nie działa dla Twojego konkretnego systemu, możesz ręcznie uruchomić ChromeDriver przed uruchomieniem komendy `dusk`. Jeśli zdecydujesz się ręcznie uruchomić ChromeDriver, powinieneś zakomentować następującą linię pliku `tests/DuskTestCase.php`:

```php
/**
 * Prepare for Dusk test execution.
 *
 * @beforeClass
 */
public static function prepare(): void
{
    // static::startChromeDriver();
}
```

Ponadto, jeśli uruchomisz ChromeDriver na porcie innym niż 9515, powinieneś zmodyfikować metodę `driver` tej samej klasy, aby odzwierciedlać prawidłowy port:

```php
use Facebook\WebDriver\Remote\RemoteWebDriver;

/**
 * Create the RemoteWebDriver instance.
 */
protected function driver(): RemoteWebDriver
{
    return RemoteWebDriver::create(
        'http://localhost:9515', DesiredCapabilities::chrome()
    );
}
```

<a name="environment-handling"></a>
### Obsługa środowiska

Aby zmusić Dusk do używania własnego pliku środowiskowego podczas uruchamiania testów, utwórz plik `.env.dusk.{environment}` w katalogu głównym Twojego projektu. Na przykład, jeśli będziesz inicjować komendę `dusk` ze swojego środowiska `local`, powinieneś utworzyć plik `.env.dusk.local`.

Podczas uruchamiania testów Dusk utworzy kopię zapasową Twojego pliku `.env` i zmieni nazwę środowiska Dusk na `.env`. Po zakończeniu testów Twój plik `.env` zostanie przywrócony.

<a name="browser-basics"></a>
## Podstawy przeglądarki

<a name="creating-browsers"></a>
### Tworzenie przeglądarek

Aby rozpocząć, napiszmy test, który weryfikuje, czy możemy zalogować się do naszej aplikacji. Po wygenerowaniu testu możemy go zmodyfikować, aby przejść do strony logowania, wprowadzić pewne dane uwierzytelniające i kliknąć przycisk "Login". Aby utworzyć instancję przeglądarki, możesz wywołać metodę `browse` w ramach swojego testu Dusk:

```php tab=Pest
<?php

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;

pest()->use(DatabaseMigrations::class);

test('basic example', function () {
    $user = User::factory()->create([
        'email' => 'taylor@laravel.com',
    ]);

    $this->browse(function (Browser $browser) use ($user) {
        $browser->visit('/login')
            ->type('email', $user->email)
            ->type('password', 'password')
            ->press('Login')
            ->assertPathIs('/home');
    });
});
```

```php tab=PHPUnit
<?php

namespace Tests\Browser;

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;

    /**
     * A basic browser test example.
     */
    public function test_basic_example(): void
    {
        $user = User::factory()->create([
            'email' => 'taylor@laravel.com',
        ]);

        $this->browse(function (Browser $browser) use ($user) {
            $browser->visit('/login')
                ->type('email', $user->email)
                ->type('password', 'password')
                ->press('Login')
                ->assertPathIs('/home');
        });
    }
}
```

Jak widać w powyższym przykładzie, metoda `browse` akceptuje domknięcie. Instancja przeglądarki zostanie automatycznie przekazana do tego domknięcia przez Dusk i jest głównym obiektem używanym do interakcji i wykonywania asercji względem Twojej aplikacji.

<a name="creating-multiple-browsers"></a>
#### Tworzenie wielu przeglądarek

Niekiedy możesz potrzebować wielu przeglądarek, aby prawidłowo przeprowadzić test. Na przykład wiele przeglądarek może być potrzebnych do przetestowania ekranu czatu, który wchodzi w interakcję z websocketami. Aby utworzyć wiele przeglądarek, po prostu dodaj więcej argumentów przeglądarki do sygnatury closure przekazanego do metody `browse`:

```php
$this->browse(function (Browser $first, Browser $second) {
    $first->loginAs(User::find(1))
        ->visit('/home')
        ->waitForText('Message');

    $second->loginAs(User::find(2))
        ->visit('/home')
        ->waitForText('Message')
        ->type('message', 'Hey Taylor')
        ->press('Send');

    $first->waitForText('Hey Taylor')
        ->assertSee('Jeffrey Way');
});
```

<a name="navigation"></a>
### Nawigacja

Metoda `visit` może być użyta do przejścia do danego URI w Twojej aplikacji:

```php
$browser->visit('/login');
```

Możesz użyć metody `visitRoute`, aby przejść do [nazwanej trasy](/docs/{{version}}/routing#named-routes):

```php
$browser->visitRoute($routeName, $parameters);
```

Możesz nawigować "wstecz" i "naprzód" używając metod `back` i `forward`:

```php
$browser->back();

$browser->forward();
```

Możesz użyć metody `refresh`, aby odświeżyć stronę:

```php
$browser->refresh();
```

<a name="resizing-browser-windows"></a>
### Zmiana rozmiaru okien przeglądarki

Możesz użyć metody `resize`, aby dostosować rozmiar okna przeglądarki:

```php
$browser->resize(1920, 1080);
```

Metoda `maximize` może być użyta do zmaksymalizowania okna przeglądarki:

```php
$browser->maximize();
```

Metoda `fitContent` zmieni rozmiar okna przeglądarki, aby dopasować się do rozmiaru jego treści:

```php
$browser->fitContent();
```

Gdy test się nie powiedzie, Dusk automatycznie zmieni rozmiar przeglądarki, aby dopasować się do treści przed wykonaniem zrzutu ekranu. Możesz wyłączyć tę funkcję, wywołując metodę `disableFitOnFailure` w swoim teście:

```php
$browser->disableFitOnFailure();
```

Możesz użyć metody `move`, aby przesunąć okno przeglądarki w inne miejsce na ekranie:

```php
$browser->move($x = 100, $y = 100);
```

<a name="browser-macros"></a>
### Makra przeglądarki

Jeśli chcesz zdefiniować niestandardową metodę przeglądarki, którą możesz ponownie używać w różnych testach, możesz użyć metody `macro` na klasie `Browser`. Zazwyczaj powinieneś wywołać tę metodę z metody `boot` [dostawcy usług](/docs/{{version}}/providers):

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Dusk\Browser;

class DuskServiceProvider extends ServiceProvider
{
    /**
     * Register Dusk's browser macros.
     */
    public function boot(): void
    {
        Browser::macro('scrollToElement', function (string $element = null) {
            $this->script("$('html, body').animate({ scrollTop: $('$element').offset().top }, 0);");

            return $this;
        });
    }
}
```

Funkcja `macro` przyjmuje nazwę jako pierwszy argument i closure jako drugi. Closure makra zostanie wykonane podczas wywoływania makra jako metody na instancji `Browser`:

```php
$this->browse(function (Browser $browser) use ($user) {
    $browser->visit('/pay')
        ->scrollToElement('#credit-card-details')
        ->assertSee('Enter Credit Card Details');
});
```

<a name="authentication"></a>
### Uwierzytelnianie

Często będziesz testować strony wymagające uwierzytelnienia. Możesz użyć metody `loginAs` Dusk, aby uniknąć interakcji z ekranem logowania Twojej aplikacji podczas każdego testu. Metoda `loginAs` akceptuje klucz główny związany z Twoim modelem uwierzytelniającym lub instancję modelu uwierzytelniającego:

```php
use App\Models\User;
use Laravel\Dusk\Browser;

$this->browse(function (Browser $browser) {
    $browser->loginAs(User::find(1))
        ->visit('/home');
});
```

> [!WARNING]
> Po użyciu metody `loginAs` sesja użytkownika będzie utrzymywana dla wszystkich testów w pliku.

<a name="cookies"></a>
### Ciasteczka

Możesz użyć metody `cookie`, aby pobrać lub ustawić wartość zaszyfrowanego ciasteczka. Domyślnie wszystkie ciasteczka tworzone przez Laravel są zaszyfrowane:

```php
$browser->cookie('name');

$browser->cookie('name', 'Taylor');
```

Możesz użyć metody `plainCookie`, aby pobrać lub ustawić wartość niezaszyfrowanego ciasteczka:

```php
$browser->plainCookie('name');

$browser->plainCookie('name', 'Taylor');
```

Możesz użyć metody `deleteCookie`, aby usunąć podane ciasteczko:

```php
$browser->deleteCookie('name');
```

<a name="executing-javascript"></a>
### Wykonywanie JavaScript

Możesz użyć metody `script`, aby wykonać dowolne instrukcje JavaScript w przeglądarce:

```php
$browser->script('document.documentElement.scrollTop = 0');

$browser->script([
    'document.body.scrollTop = 0',
    'document.documentElement.scrollTop = 0',
]);

$output = $browser->script('return window.location.pathname');
```

<a name="taking-a-screenshot"></a>
### Robienie zrzutu ekranu

Możesz użyć metody `screenshot`, aby zrobić zrzut ekranu i zapisać go pod podaną nazwą pliku. Wszystkie zrzuty ekranu będą przechowywane w katalogu `tests/Browser/screenshots`:

```php
$browser->screenshot('filename');
```

Metoda `responsiveScreenshots` może być użyta do wykonania serii zrzutów ekranu na różnych punktów przerwań (breakpoints):

```php
$browser->responsiveScreenshots('filename');
```

Metoda `screenshotElement` może być użyta do wykonania zrzutu ekranu określonego elementu na stronie:

```php
$browser->screenshotElement('#selector', 'filename');
```

<a name="storing-console-output-to-disk"></a>
### Zapisywanie wyjścia konsoli na dysk

Możesz użyć metody `storeConsoleLog`, aby zapisać wyjście konsoli bieżącej przeglądarki na dysku pod podaną nazwą pliku. Wyjście konsoli będzie przechowywane w katalogu `tests/Browser/console`:

```php
$browser->storeConsoleLog('filename');
```

<a name="storing-page-source-to-disk"></a>
### Zapisywanie źródła strony na dysk

Możesz użyć metody `storeSource`, aby zapisać źródło bieżącej strony na dysku pod podaną nazwą pliku. Źródło strony będzie przechowywane w katalogu `tests/Browser/source`:

```php
$browser->storeSource('filename');
```

<a name="interacting-with-elements"></a>
## Interakcja z elementami

<a name="dusk-selectors"></a>
### Selektory Dusk

Wybieranie dobrych selektorów CSS do interakcji z elementami jest jedną z najtrudniejszych części pisania testów Dusk. Z czasem zmiany w interfejsie użytkownika mogą spowodować, że selektory CSS takie jak poniższe zepsują Twoje testy:

```html
// HTML...

<button>Login</button>
```

```php
// Test...

$browser->click('.login-page .container div > button');
```

Selektory Dusk pozwalają Ci skupić się na pisaniu skutecznych testów zamiast pamiętania selektorów CSS. Aby zdefiniować selektor, dodaj atrybut `dusk` do swojego elementu HTML. Następnie, podczas interakcji z przeglądarką Dusk, dodaj przedrostek `@` do selektora, aby manipulować dołączonym elementem w swoim teście:

```html
// HTML...

<button dusk="login-button">Login</button>
```

```php
// Test...

$browser->click('@login-button');
```

Jeśli chcesz, możesz dostosować atrybut HTML, którego używa selektor Dusk, za pomocą metody `selectorHtmlAttribute`. Zazwyczaj metoda ta powinna być wywoływana z metody `boot` `AppServiceProvider` Twojej aplikacji:

```php
use Laravel\Dusk\Dusk;

Dusk::selectorHtmlAttribute('data-dusk');
```

<a name="text-values-and-attributes"></a>
### Tekst, wartości i atrybuty

<a name="retrieving-setting-values"></a>
#### Pobieranie i ustawianie wartości

Dusk zapewnia kilka metod do interakcji z bieżącą wartością, wyświetlanym tekstem i atrybutami elementów na stronie. Na przykład, aby uzyskać "wartość" elementu, który pasuje do danego selektora CSS lub Dusk, użyj metody `value`:

```php
// Pobierz wartość...
$value = $browser->value('selector');

// Ustaw wartość...
$browser->value('selector', 'value');
```

Możesz użyć metody `inputValue`, aby uzyskać "wartość" elementu input, który ma daną nazwę pola:

```php
$value = $browser->inputValue('field');
```

<a name="retrieving-text"></a>
#### Pobieranie tekstu

Metoda `text` może być użyta do pobrania wyświetlanego tekstu elementu, który pasuje do danego selektora:

```php
$text = $browser->text('selector');
```

<a name="retrieving-attributes"></a>
#### Pobieranie atrybutów

Wreszcie metoda `attribute` może być użyta do pobrania wartości atrybutu elementu pasującego do danego selektora:

```php
$attribute = $browser->attribute('selector', 'value');
```

<a name="interacting-with-forms"></a>
### Interakcja z formularzami

<a name="typing-values"></a>
#### Wpisywanie wartości

Dusk zapewnia różnorodne metody do interakcji z formularzami i elementami input. Najpierw spójrzmy na przykład wpisywania tekstu w pole input:

```php
$browser->type('email', 'taylor@laravel.com');
```

Zauważ, że choć metoda akceptuje jeden, jeśli to konieczne, nie musimy przekazywać selektora CSS do metody `type`. Jeśli selektor CSS nie zostanie podany, Dusk będzie szukać pola `input` lub `textarea` z podanym atrybutem `name`.

Aby dodać tekst do pola bez czyszczenia jego zawartości, możesz użyć metody `append`:

```php
$browser->type('tags', 'foo')
    ->append('tags', ', bar, baz');
```

Możesz wyczyścić wartość pola input używając metody `clear`:

```php
$browser->clear('email');
```

Możesz polecić Dusk, aby pisał wolno, używając metody `typeSlowly`. Domyślnie Dusk będzie czekać 100 milisekund między naciśnięciami klawiszy. Aby dostosować czas między naciśnięciami klawiszy, możesz przekazać odpowiednią liczbę milisekund jako trzeci argument metody:

```php
$browser->typeSlowly('mobile', '+1 (202) 555-5555');

$browser->typeSlowly('mobile', '+1 (202) 555-5555', 300);
```

Możesz użyć metody `appendSlowly`, aby dodać tekst wolno:

```php
$browser->type('tags', 'foo')
    ->appendSlowly('tags', ', bar, baz');
```

<a name="dropdowns"></a>
#### Listy rozwijane

Aby wybrać wartość dostępną w elemencie `select`, możesz użyć metody `select`. Podobnie jak metoda `type`, metoda `select` nie wymaga pełnego selektora CSS. Przekazując wartość do metody `select`, powinieneś przekazać podstawową wartość opcji zamiast wyświetlanego tekstu:

```php
$browser->select('size', 'Large');
```

Możesz wybrać losową opcję, pomijając drugi argument:

```php
$browser->select('size');
```

Dostarczając tablicę jako drugi argument metody `select`, możesz polecić metodzie wybranie wielu opcji:

```php
$browser->select('categories', ['Art', 'Music']);
```

<a name="checkboxes"></a>
#### Pola wyboru

Aby "zaznaczyć" pole wyboru, możesz użyć metody `check`. Podobnie jak wiele innych metod związanych z polami input, pełny selektor CSS nie jest wymagany. Jeśli nie można znaleźć dopasowania selektora CSS, Dusk będzie szukać pola wyboru z pasującym atrybutem `name`:

```php
$browser->check('terms');
```

Metoda `uncheck` może być użyta do "odznaczenia" pola wyboru:

```php
$browser->uncheck('terms');
```

<a name="radio-buttons"></a>
#### Przyciski radiowe

Aby "wybrać" opcję `radio`, możesz użyć metody `radio`. Podobnie jak wiele innych metod związanych z polami input, pełny selektor CSS nie jest wymagany. Jeśli nie można znaleźć dopasowania selektora CSS, Dusk będzie szukać pola `radio` z pasującymi atrybutami `name` i `value`:

```php
$browser->radio('size', 'large');
```

<a name="attaching-files"></a>
### Dołączanie plików

Metoda `attach` może być użyta do dołączenia pliku do elementu input `file`. Podobnie jak wiele innych metod związanych z polami input, pełny selektor CSS nie jest wymagany. Jeśli nie można znaleźć dopasowania selektora CSS, Dusk będzie szukać pola input `file` z pasującym atrybutem `name`:

```php
$browser->attach('photo', __DIR__.'/photos/mountains.png');
```

> [!WARNING]
> Funkcja dołączania wymaga, aby rozszerzenie PHP `Zip` było zainstalowane i włączone na Twoim serwerze.

<a name="pressing-buttons"></a>
### Naciskanie przycisków

Metoda `press` może być użyta do kliknięcia elementu przycisku na stronie. Argument przekazany do metody `press` może być wyświetlanym tekstem przycisku lub selektorem CSS / Dusk:

```php
$browser->press('Login');
```

Podczas przesyłania formularzy wiele aplikacji wyłącza przycisk przesyłania formularza po jego naciśnięciu, a następnie ponownie włącza przycisk po zakończeniu żądania HTTP przesyłania formularza. Aby nacisnąć przycisk i czekać, aż przycisk zostanie ponownie włączony, możesz użyć metody `pressAndWaitFor`:

```php
// Naciśnij przycisk i czekaj maksymalnie 5 sekund, aż zostanie włączony...
$browser->pressAndWaitFor('Save');

// Naciśnij przycisk i czekaj maksymalnie 1 sekundę, aż zostanie włączony...
$browser->pressAndWaitFor('Save', 1);
```

<a name="clicking-links"></a>
### Klikanie linków

Aby kliknąć link, możesz użyć metody `clickLink` na instancji przeglądarki. Metoda `clickLink` kliknie link, który ma podany wyświetlany tekst:

```php
$browser->clickLink($linkText);
```

Możesz użyć metody `seeLink`, aby określić, czy link z podanym wyświetlanym tekstem jest widoczny na stronie:

```php
if ($browser->seeLink($linkText)) {
    // ...
}
```

> [!WARNING]
> Te metody wchodzą w interakcję z jQuery. Jeśli jQuery nie jest dostępne na stronie, Dusk automatycznie wstrzyknie je na stronę, aby było dostępne przez czas trwania testu.

<a name="using-the-keyboard"></a>
### Używanie klawiatury

Metoda `keys` pozwala na dostarczenie bardziej złożonych sekwencji wprowadzania do danego elementu niż normalnie dozwolone przez metodę `type`. Na przykład możesz polecić Dusk, aby przytrzymał klawisze modyfikujące podczas wprowadzania wartości. W tym przykładzie klawisz `shift` będzie przytrzymywany, podczas gdy `taylor` jest wprowadzany do elementu pasującego do danego selektora. Po wpisaniu `taylor`, `swift` zostanie wpisane bez żadnych klawiszy modyfikujących:

```php
$browser->keys('selector', ['{shift}', 'taylor'], 'swift');
```

Innym cennym zastosowaniem metody `keys` jest wysłanie kombinacji "skrótu klawiaturowego" do głównego selektora CSS Twojej aplikacji:

```php
$browser->keys('.app', ['{command}', 'j']);
```

> [!NOTE]
> Wszystkie klawisze modyfikujące, takie jak `{command}`, są opakowane w znaki `{}` i odpowiadają stałym zdefiniowanym w klasie `Facebook\WebDriver\WebDriverKeys`, którą można [znaleźć na GitHub](https://github.com/php-webdriver/php-webdriver/blob/master/lib/WebDriverKeys.php).

<a name="fluent-keyboard-interactions"></a>
#### Płynne interakcje klawiaturowe

Dusk zapewnia również metodę `withKeyboard`, pozwalającą płynnie wykonywać złożone interakcje klawiaturowe poprzez klasę `Laravel\Dusk\Keyboard`. Klasa `Keyboard` udostępnia metody `press`, `release`, `type` i `pause`:

```php
use Laravel\Dusk\Keyboard;

$browser->withKeyboard(function (Keyboard $keyboard) {
    $keyboard->press('c')
        ->pause(1000)
        ->release('c')
        ->type(['c', 'e', 'o']);
});
```

<a name="keyboard-macros"></a>
#### Makra klawiaturowe

Jeśli chcesz zdefiniować niestandardowe interakcje klawiaturowe, które możesz łatwo ponownie używać w całym swoim zestawie testów, możesz użyć metody `macro` dostarczonej przez klasę `Keyboard`. Zazwyczaj powinieneś wywołać tę metodę z metody `boot` [dostawcy usług](/docs/{{version}}/providers):

```php
<?php

namespace App\Providers;

use Facebook\WebDriver\WebDriverKeys;
use Illuminate\Support\ServiceProvider;
use Laravel\Dusk\Keyboard;
use Laravel\Dusk\OperatingSystem;

class DuskServiceProvider extends ServiceProvider
{
    /**
     * Register Dusk's browser macros.
     */
    public function boot(): void
    {
        Keyboard::macro('copy', function (string $element = null) {
            $this->type([
                OperatingSystem::onMac() ? WebDriverKeys::META : WebDriverKeys::CONTROL, 'c',
            ]);

            return $this;
        });

        Keyboard::macro('paste', function (string $element = null) {
            $this->type([
                OperatingSystem::onMac() ? WebDriverKeys::META : WebDriverKeys::CONTROL, 'v',
            ]);

            return $this;
        });
    }
}
```

Funkcja `macro` przyjmuje nazwę jako pierwszy argument i closure jako drugi. Closure makra zostanie wykonane podczas wywoływania makra jako metody na instancji `Keyboard`:

```php
$browser->click('@textarea')
    ->withKeyboard(fn (Keyboard $keyboard) => $keyboard->copy())
    ->click('@another-textarea')
    ->withKeyboard(fn (Keyboard $keyboard) => $keyboard->paste());
```

<a name="using-the-mouse"></a>
### Używanie myszy

<a name="clicking-on-elements"></a>
#### Klikanie na elementy

Metoda `click` może być użyta do kliknięcia elementu pasującego do danego selektora CSS lub Dusk:

```php
$browser->click('.selector');
```

Metoda `clickAtXPath` może być użyta do kliknięcia elementu pasującego do danego wyrażenia XPath:

```php
$browser->clickAtXPath('//div[@class = "selector"]');
```

Metoda `clickAtPoint` może być użyta do kliknięcia najwyższego elementu przy danej parze współrzędnych względem widocznego obszaru przeglądarki:

```php
$browser->clickAtPoint($x = 0, $y = 0);
```

Metoda `doubleClick` może być użyta do symulacji podwójnego kliknięcia myszą:

```php
$browser->doubleClick();

$browser->doubleClick('.selector');
```

Metoda `rightClick` może być użyta do symulacji prawego kliknięcia myszą:

```php
$browser->rightClick();

$browser->rightClick('.selector');
```

Metoda `clickAndHold` może być użyta do symulacji kliknięcia i przytrzymania przycisku myszy. Kolejne wywołanie metody `releaseMouse` cofnie to zachowanie i zwolni przycisk myszy:

```php
$browser->clickAndHold('.selector');

$browser->clickAndHold()
    ->pause(1000)
    ->releaseMouse();
```

Metoda `controlClick` może być użyta do symulacji zdarzenia `ctrl+click` w przeglądarce:

```php
$browser->controlClick();

$browser->controlClick('.selector');
```

<a name="mouseover"></a>
#### Najechanie myszką

Metoda `mouseover` może być użyta, gdy musisz przesunąć mysz nad element pasujący do danego selektora CSS lub Dusk:

```php
$browser->mouseover('.selector');
```

<a name="drag-drop"></a>
#### Przeciąganie i upuszczanie

Metoda `drag` może być użyta do przeciągnięcia elementu pasującego do danego selektora na inny element:

```php
$browser->drag('.from-selector', '.to-selector');
```

Lub możesz przeciągnąć element w jednym kierunku:

```php
$browser->dragLeft('.selector', $pixels = 10);
$browser->dragRight('.selector', $pixels = 10);
$browser->dragUp('.selector', $pixels = 10);
$browser->dragDown('.selector', $pixels = 10);
```

Wreszcie możesz przeciągnąć element o dane przesunięcie:

```php
$browser->dragOffset('.selector', $x = 10, $y = 10);
```

<a name="javascript-dialogs"></a>
### Okna dialogowe JavaScript

Dusk zapewnia różne metody do interakcji z oknami dialogowymi JavaScript. Na przykład możesz użyć metody `waitForDialog`, aby czekać na pojawienie się okna dialogowego JavaScript. Ta metoda akceptuje opcjonalny argument określający, ile sekund czekać na pojawienie się okna dialogowego:

```php
$browser->waitForDialog($seconds = null);
```

Metoda `assertDialogOpened` może być użyta do potwierdzenia, że okno dialogowe zostało wyświetlone i zawiera dany komunikat:

```php
$browser->assertDialogOpened('Dialog message');
```

Jeśli okno dialogowe JavaScript zawiera monit, możesz użyć metody `typeInDialog`, aby wpisać wartość do monitu:

```php
$browser->typeInDialog('Hello World');
```

Aby zamknąć otwarte okno dialogowe JavaScript, klikając przycisk "OK", możesz wywołać metodę `acceptDialog`:

```php
$browser->acceptDialog();
```

Aby zamknąć otwarte okno dialogowe JavaScript, klikając przycisk "Cancel", możesz wywołać metodę `dismissDialog`:

```php
$browser->dismissDialog();
```

<a name="interacting-with-iframes"></a>
### Interakcja z ramkami inline

Jeśli musisz wejść w interakcję z elementami wewnątrz ramki iframe, możesz użyć metody `withinFrame`. Wszystkie interakcje z elementami, które odbywają się w closure przekazanym do metody `withinFrame`, będą objęte zakresem kontekstu określonego iframe:

```php
$browser->withinFrame('#credit-card-details', function ($browser) {
    $browser->type('input[name="cardnumber"]', '4242424242424242')
        ->type('input[name="exp-date"]', '1224')
        ->type('input[name="cvc"]', '123')
        ->press('Pay');
});
```

<a name="scoping-selectors"></a>
### Określanie zakresu selektorów

Niekiedy możesz chcieć wykonać kilka operacji, objętą zakresem wszystkich operacji w danym selektorze. Na przykład możesz chcieć potwierdzić, że jakiś tekst istnieje tylko w tabeli, a następnie kliknąć przycisk w tej tabeli. Możesz użyć metody `with`, aby to osiągnąć. Wszystkie operacje wykonane w closure przekazanym do metody `with` będą objęte zakresem oryginalnego selektora:

```php
$browser->with('.table', function (Browser $table) {
    $table->assertSee('Hello World')
        ->clickLink('Delete');
});
```

Możesz czasami potrzebować wykonać asercje poza bieżącym zakresem. Możesz użyć metod `elsewhere` i `elsewhereWhenAvailable`, aby to osiągnąć:

```php
$browser->with('.table', function (Browser $table) {
    // Bieżący zakres to `body .table`...

    $browser->elsewhere('.page-title', function (Browser $title) {
        // Bieżący zakres to `body .page-title`...
        $title->assertSee('Hello World');
    });

    $browser->elsewhereWhenAvailable('.page-title', function (Browser $title) {
        // Bieżący zakres to `body .page-title`...
        $title->assertSee('Hello World');
    });
});
```

<a name="waiting-for-elements"></a>
### Oczekiwanie na elementy

Podczas testowania aplikacji, które intensywnie wykorzystują JavaScript, często konieczne staje się "oczekiwanie" na dostępność pewnych elementów lub danych przed kontynuowaniem testu. Dusk ułatwia to. Używając różnych metod, możesz czekać, aż elementy staną się widoczne na stronie, a nawet czekać, aż dane wyrażenie JavaScript zostanie obliczone jako `true`.

<a name="waiting"></a>
#### Oczekiwanie

Jeśli po prostu musisz wstrzymać test na daną liczbę milisekund, użyj metody `pause`:

```php
$browser->pause(1000);
```

Jeśli musisz wstrzymać test tylko wtedy, gdy dany warunek jest `true`, użyj metody `pauseIf`:

```php
$browser->pauseIf(App::environment('production'), 1000);
```

Podobnie, jeśli musisz wstrzymać test, chyba że dany warunek jest `true`, możesz użyć metody `pauseUnless`:

```php
$browser->pauseUnless(App::environment('testing'), 1000);
```

<a name="waiting-for-selectors"></a>
#### Oczekiwanie na selektory

Metoda `waitFor` może być użyta do wstrzymania wykonywania testu, aż element pasujący do danego selektora CSS lub Dusk zostanie wyświetlony na stronie. Domyślnie spowoduje to wstrzymanie testu na maksymalnie pięć sekund przed zgłoszeniem wyjątku. Jeśli to konieczne, możesz przekazać niestandardowy próg limitu czasu jako drugi argument metody:

```php
// Czekaj maksymalnie pięć sekund na selektor...
$browser->waitFor('.selector');

// Czekaj maksymalnie jedną sekundę na selektor...
$browser->waitFor('.selector', 1);
```

Możesz również czekać, aż element pasujący do danego selektora będzie zawierał podany tekst:

```php
// Czekaj maksymalnie pięć sekund, aż selektor będzie zawierał podany tekst...
$browser->waitForTextIn('.selector', 'Hello World');

// Czekaj maksymalnie jedną sekundę, aż selektor będzie zawierał podany tekst...
$browser->waitForTextIn('.selector', 'Hello World', 1);
```

Możesz również czekać, aż element pasujący do danego selektora zniknie ze strony:

```php
// Czekaj maksymalnie pięć sekund, aż selektor zniknie...
$browser->waitUntilMissing('.selector');

// Czekaj maksymalnie jedną sekundę, aż selektor zniknie...
$browser->waitUntilMissing('.selector', 1);
```

Lub możesz czekać, aż element pasujący do danego selektora zostanie włączony lub wyłączony:

```php
// Czekaj maksymalnie pięć sekund, aż selektor zostanie włączony...
$browser->waitUntilEnabled('.selector');

// Czekaj maksymalnie jedną sekundę, aż selektor zostanie włączony...
$browser->waitUntilEnabled('.selector', 1);

// Czekaj maksymalnie pięć sekund, aż selektor zostanie wyłączony...
$browser->waitUntilDisabled('.selector');

// Czekaj maksymalnie jedną sekundę, aż selektor zostanie wyłączony...
$browser->waitUntilDisabled('.selector', 1);
```

<a name="scoping-selectors-when-available"></a>
#### Określanie zakresu selektorów gdy dostępne

Okazjonalnie możesz chcieć poczekać na pojawienie się elementu pasującego do danego selektora, a następnie wejść w interakcję z elementem. Na przykład możesz chcieć poczekać, aż okno modalne będzie dostępne, a następnie nacisnąć przycisk "OK" w oknie modalnym. Metoda `whenAvailable` może być użyta do tego celu. Wszystkie operacje na elementach wykonane w danym closure będą objęte zakresem oryginalnego selektora:

```php
$browser->whenAvailable('.modal', function (Browser $modal) {
    $modal->assertSee('Hello World')
        ->press('OK');
});
```

<a name="waiting-for-text"></a>
#### Oczekiwanie na tekst

Metoda `waitForText` może być użyta do oczekiwania, aż podany tekst zostanie wyświetlony na stronie:

```php
// Czekaj maksymalnie pięć sekund na tekst...
$browser->waitForText('Hello World');

// Czekaj maksymalnie jedną sekundę na tekst...
$browser->waitForText('Hello World', 1);
```

Możesz użyć metody `waitUntilMissingText`, aby poczekać, aż wyświetlany tekst zostanie usunięty ze strony:

```php
// Czekaj maksymalnie pięć sekund, aż tekst zostanie usunięty...
$browser->waitUntilMissingText('Hello World');

// Czekaj maksymalnie jedną sekundę, aż tekst zostanie usunięty...
$browser->waitUntilMissingText('Hello World', 1);
```

<a name="waiting-for-links"></a>
#### Oczekiwanie na linki

Metoda `waitForLink` może być użyta do oczekiwania, aż podany tekst linku zostanie wyświetlony na stronie:

```php
// Czekaj maksymalnie pięć sekund na link...
$browser->waitForLink('Create');

// Czekaj maksymalnie jedną sekundę na link...
$browser->waitForLink('Create', 1);
```

<a name="waiting-for-inputs"></a>
#### Oczekiwanie na pola input

Metoda `waitForInput` może być użyta do oczekiwania, aż podane pole input będzie widoczne na stronie:

```php
// Czekaj maksymalnie pięć sekund na input...
$browser->waitForInput($field);

// Czekaj maksymalnie jedną sekundę na input...
$browser->waitForInput($field, 1);
```

<a name="waiting-on-the-page-location"></a>
#### Oczekiwanie na lokalizację strony

Podczas wykonywania asercji ścieżki, takiej jak `$browser->assertPathIs('/home')`, asercja może się nie powieść, jeśli `window.location.pathname` jest aktualizowany asynchronicznie. Możesz użyć metody `waitForLocation`, aby poczekać, aż lokalizacja będzie miała daną wartość:

```php
$browser->waitForLocation('/secret');
```

Metoda `waitForLocation` może również być użyta do oczekiwania, aż bieżąca lokalizacja okna będzie w pełni kwalifikowanym adresem URL:

```php
$browser->waitForLocation('https://example.com/path');
```

Możesz również czekać na lokalizację [nazwanej trasy](/docs/{{version}}/routing#named-routes):

```php
$browser->waitForRoute($routeName, $parameters);
```

<a name="waiting-for-page-reloads"></a>
#### Oczekiwanie na przeładowanie strony

Jeśli musisz poczekać na przeładowanie strony po wykonaniu akcji, użyj metody `waitForReload`:

```php
use Laravel\Dusk\Browser;

$browser->waitForReload(function (Browser $browser) {
    $browser->press('Submit');
})
->assertSee('Success!');
```

Ponieważ potrzeba oczekiwania na przeładowanie strony zazwyczaj występuje po kliknięciu przycisku, możesz użyć metody `clickAndWaitForReload` dla wygody:

```php
$browser->clickAndWaitForReload('.selector')
    ->assertSee('something');
```

<a name="waiting-on-javascript-expressions"></a>
#### Oczekiwanie na wyrażenia JavaScript

Niekiedy możesz chcieć wstrzymać wykonywanie testu, aż dane wyrażenie JavaScript zostanie obliczone jako `true`. Możesz łatwo to osiągnąć za pomocą metody `waitUntil`. Przekazując wyrażenie do tej metody, nie musisz dodawać słowa kluczowego `return` ani końcowego średnika:

```php
// Czekaj maksymalnie pięć sekund, aż wyrażenie będzie prawdziwe...
$browser->waitUntil('App.data.servers.length > 0');

// Czekaj maksymalnie jedną sekundę, aż wyrażenie będzie prawdziwe...
$browser->waitUntil('App.data.servers.length > 0', 1);
```

<a name="waiting-on-vue-expressions"></a>
#### Oczekiwanie na wyrażenia Vue

Metody `waitUntilVue` i `waitUntilVueIsNot` mogą być użyte do oczekiwania, aż atrybut [komponentu Vue](https://vuejs.org) będzie miał daną wartość:

```php
// Czekaj, aż atrybut komponentu będzie zawierał daną wartość...
$browser->waitUntilVue('user.name', 'Taylor', '@user');

// Czekaj, aż atrybut komponentu nie będzie zawierał danej wartości...
$browser->waitUntilVueIsNot('user.name', null, '@user');
```

<a name="waiting-for-javascript-events"></a>
#### Oczekiwanie na zdarzenia JavaScript

Metoda `waitForEvent` może być użyta do wstrzymania wykonywania testu, aż wystąpi zdarzenie JavaScript:

```php
$browser->waitForEvent('load');
```

Nasłuchiwacz zdarzeń jest dołączony do bieżącego zakresu, którym domyślnie jest element `body`. Podczas używania selektora zakresowego, nasłuchiwacz zdarzeń zostanie dołączony do pasującego elementu:

```php
$browser->with('iframe', function (Browser $iframe) {
    // Czekaj na zdarzenie ładowania iframe...
    $iframe->waitForEvent('load');
});
```

Możesz również podać selektor jako drugi argument metody `waitForEvent`, aby dołączyć nasłuchiwacz zdarzeń do konkretnego elementu:

```php
$browser->waitForEvent('load', '.selector');
```

Możesz również czekać na zdarzenia obiektów `document` i `window`:

```php
// Czekaj, aż dokument zostanie przewinięty...
$browser->waitForEvent('scroll', 'document');

// Czekaj maksymalnie pięć sekund, aż rozmiar okna zostanie zmieniony...
$browser->waitForEvent('resize', 'window', 5);
```

<a name="waiting-with-a-callback"></a>
#### Oczekiwanie z callback

Wiele metod "wait" w Dusk opiera się na podstawowej metodzie `waitUsing`. Możesz użyć tej metody bezpośrednio, aby poczekać, aż dane closure zwróci `true`. Metoda `waitUsing` akceptuje maksymalną liczbę sekund oczekiwania, interwał, w którym closure powinno zostać ocenione, closure oraz opcjonalny komunikat o niepowodzeniu:

```php
$browser->waitUsing(10, 1, function () use ($something) {
    return $something->isReady();
}, "Something wasn't ready in time.");
```

<a name="scrolling-an-element-into-view"></a>
### Przewijanie elementu do widoku

Niekiedy możesz nie być w stanie kliknąć na element, ponieważ znajduje się poza widocznym obszarem przeglądarki. Metoda `scrollIntoView` przewinie okno przeglądarki, aż element przy danym selektorze znajdzie się w widoku:

```php
$browser->scrollIntoView('.selector')
    ->click('.selector');
```

<a name="available-assertions"></a>
## Dostępne asercje

Dusk zapewnia różnorodne asercje, które możesz wykonać wobec swojej aplikacji. Wszystkie dostępne asercje są udokumentowane na poniższej liście:

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertTitle](#assert-title)
[assertTitleContains](#assert-title-contains)
[assertUrlIs](#assert-url-is)
[assertSchemeIs](#assert-scheme-is)
[assertSchemeIsNot](#assert-scheme-is-not)
[assertHostIs](#assert-host-is)
[assertHostIsNot](#assert-host-is-not)
[assertPortIs](#assert-port-is)
[assertPortIsNot](#assert-port-is-not)
[assertPathBeginsWith](#assert-path-begins-with)
[assertPathEndsWith](#assert-path-ends-with)
[assertPathContains](#assert-path-contains)
[assertPathIs](#assert-path-is)
[assertPathIsNot](#assert-path-is-not)
[assertRouteIs](#assert-route-is)
[assertQueryStringHas](#assert-query-string-has)
[assertQueryStringMissing](#assert-query-string-missing)
[assertFragmentIs](#assert-fragment-is)
[assertFragmentBeginsWith](#assert-fragment-begins-with)
[assertFragmentIsNot](#assert-fragment-is-not)
[assertHasCookie](#assert-has-cookie)
[assertHasPlainCookie](#assert-has-plain-cookie)
[assertCookieMissing](#assert-cookie-missing)
[assertPlainCookieMissing](#assert-plain-cookie-missing)
[assertCookieValue](#assert-cookie-value)
[assertPlainCookieValue](#assert-plain-cookie-value)
[assertSee](#assert-see)
[assertDontSee](#assert-dont-see)
[assertSeeIn](#assert-see-in)
[assertDontSeeIn](#assert-dont-see-in)
[assertSeeAnythingIn](#assert-see-anything-in)
[assertSeeNothingIn](#assert-see-nothing-in)
[assertCount](#assert-count)
[assertScript](#assert-script)
[assertSourceHas](#assert-source-has)
[assertSourceMissing](#assert-source-missing)
[assertSeeLink](#assert-see-link)
[assertDontSeeLink](#assert-dont-see-link)
[assertInputValue](#assert-input-value)
[assertInputValueIsNot](#assert-input-value-is-not)
[assertChecked](#assert-checked)
[assertNotChecked](#assert-not-checked)
[assertIndeterminate](#assert-indeterminate)
[assertRadioSelected](#assert-radio-selected)
[assertRadioNotSelected](#assert-radio-not-selected)
[assertSelected](#assert-selected)
[assertNotSelected](#assert-not-selected)
[assertSelectHasOptions](#assert-select-has-options)
[assertSelectMissingOptions](#assert-select-missing-options)
[assertSelectHasOption](#assert-select-has-option)
[assertSelectMissingOption](#assert-select-missing-option)
[assertValue](#assert-value)
[assertValueIsNot](#assert-value-is-not)
[assertAttribute](#assert-attribute)
[assertAttributeMissing](#assert-attribute-missing)
[assertAttributeContains](#assert-attribute-contains)
[assertAttributeDoesntContain](#assert-attribute-doesnt-contain)
[assertAriaAttribute](#assert-aria-attribute)
[assertDataAttribute](#assert-data-attribute)
[assertVisible](#assert-visible)
[assertPresent](#assert-present)
[assertNotPresent](#assert-not-present)
[assertMissing](#assert-missing)
[assertInputPresent](#assert-input-present)
[assertInputMissing](#assert-input-missing)
[assertDialogOpened](#assert-dialog-opened)
[assertEnabled](#assert-enabled)
[assertDisabled](#assert-disabled)
[assertButtonEnabled](#assert-button-enabled)
[assertButtonDisabled](#assert-button-disabled)
[assertFocused](#assert-focused)
[assertNotFocused](#assert-not-focused)
[assertAuthenticated](#assert-authenticated)
[assertGuest](#assert-guest)
[assertAuthenticatedAs](#assert-authenticated-as)
[assertVue](#assert-vue)
[assertVueIsNot](#assert-vue-is-not)
[assertVueContains](#assert-vue-contains)
[assertVueDoesntContain](#assert-vue-doesnt-contain)

</div>

<a name="assert-title"></a>
#### assertTitle

Potwierdź, że tytuł strony pasuje do podanego tekstu:

```php
$browser->assertTitle($title);
```

<a name="assert-title-contains"></a>
#### assertTitleContains

Potwierdź, że tytuł strony zawiera podany tekst:

```php
$browser->assertTitleContains($title);
```

<a name="assert-url-is"></a>
#### assertUrlIs

Potwierdź, że bieżący URL (bez ciągu zapytania) pasuje do podanego ciągu:

```php
$browser->assertUrlIs($url);
```

<a name="assert-scheme-is"></a>
#### assertSchemeIs

Potwierdź, że bieżący schemat URL pasuje do podanego schematu:

```php
$browser->assertSchemeIs($scheme);
```

<a name="assert-scheme-is-not"></a>
#### assertSchemeIsNot

Potwierdź, że bieżący schemat URL nie pasuje do podanego schematu:

```php
$browser->assertSchemeIsNot($scheme);
```

<a name="assert-host-is"></a>
#### assertHostIs

Potwierdź, że bieżący host URL pasuje do podanego hosta:

```php
$browser->assertHostIs($host);
```

<a name="assert-host-is-not"></a>
#### assertHostIsNot

Potwierdź, że bieżący host URL nie pasuje do podanego hosta:

```php
$browser->assertHostIsNot($host);
```

<a name="assert-port-is"></a>
#### assertPortIs

Potwierdź, że bieżący port URL pasuje do podanego portu:

```php
$browser->assertPortIs($port);
```

<a name="assert-port-is-not"></a>
#### assertPortIsNot

Potwierdź, że bieżący port URL nie pasuje do podanego portu:

```php
$browser->assertPortIsNot($port);
```

<a name="assert-path-begins-with"></a>
#### assertPathBeginsWith

Potwierdź, że bieżąca ścieżka URL zaczyna się od podanej ścieżki:

```php
$browser->assertPathBeginsWith('/home');
```

<a name="assert-path-ends-with"></a>
#### assertPathEndsWith

Potwierdź, że bieżąca ścieżka URL kończy się podaną ścieżką:

```php
$browser->assertPathEndsWith('/home');
```

<a name="assert-path-contains"></a>
#### assertPathContains

Potwierdź, że bieżąca ścieżka URL zawiera podaną ścieżkę:

```php
$browser->assertPathContains('/home');
```

<a name="assert-path-is"></a>
#### assertPathIs

Potwierdź, że bieżąca ścieżka pasuje do podanej ścieżki:

```php
$browser->assertPathIs('/home');
```

<a name="assert-path-is-not"></a>
#### assertPathIsNot

Potwierdź, że bieżąca ścieżka nie pasuje do podanej ścieżki:

```php
$browser->assertPathIsNot('/home');
```

<a name="assert-route-is"></a>
#### assertRouteIs

Potwierdź, że bieżący URL pasuje do URL podanej [nazwanej trasy](/docs/{{version}}/routing#named-routes):

```php
$browser->assertRouteIs($name, $parameters);
```

<a name="assert-query-string-has"></a>
#### assertQueryStringHas

Potwierdź, że podany parametr ciągu zapytania jest obecny:

```php
$browser->assertQueryStringHas($name);
```

Potwierdź, że podany parametr ciągu zapytania jest obecny i ma daną wartość:

```php
$browser->assertQueryStringHas($name, $value);
```

<a name="assert-query-string-missing"></a>
#### assertQueryStringMissing

Potwierdź, że podany parametr ciągu zapytania jest nieobecny:

```php
$browser->assertQueryStringMissing($name);
```

<a name="assert-fragment-is"></a>
#### assertFragmentIs

Potwierdź, że bieżący fragment hash URL pasuje do podanego fragmentu:

```php
$browser->assertFragmentIs('anchor');
```

<a name="assert-fragment-begins-with"></a>
#### assertFragmentBeginsWith

Potwierdź, że bieżący fragment hash URL zaczyna się od podanego fragmentu:

```php
$browser->assertFragmentBeginsWith('anchor');
```

<a name="assert-fragment-is-not"></a>
#### assertFragmentIsNot

Potwierdź, że bieżący fragment hash URL nie pasuje do podanego fragmentu:

```php
$browser->assertFragmentIsNot('anchor');
```

<a name="assert-has-cookie"></a>
#### assertHasCookie

Potwierdź, że podane zaszyfrowane ciasteczko jest obecne:

```php
$browser->assertHasCookie($name);
```

<a name="assert-has-plain-cookie"></a>
#### assertHasPlainCookie

Potwierdź, że podane niezaszyfrowane ciasteczko jest obecne:

```php
$browser->assertHasPlainCookie($name);
```

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

Potwierdź, że podane zaszyfrowane ciasteczko jest nieobecne:

```php
$browser->assertCookieMissing($name);
```

<a name="assert-plain-cookie-missing"></a>
#### assertPlainCookieMissing

Potwierdź, że podane niezaszyfrowane ciasteczko jest nieobecne:

```php
$browser->assertPlainCookieMissing($name);
```

<a name="assert-cookie-value"></a>
#### assertCookieValue

Potwierdź, że zaszyfrowane ciasteczko ma daną wartość:

```php
$browser->assertCookieValue($name, $value);
```

<a name="assert-plain-cookie-value"></a>
#### assertPlainCookieValue

Potwierdź, że niezaszyfrowane ciasteczko ma daną wartość:

```php
$browser->assertPlainCookieValue($name, $value);
```

<a name="assert-see"></a>
#### assertSee

Potwierdź, że podany tekst jest obecny na stronie:

```php
$browser->assertSee($text);
```

<a name="assert-dont-see"></a>
#### assertDontSee

Potwierdź, że podany tekst jest nieobecny na stronie:

```php
$browser->assertDontSee($text);
```

<a name="assert-see-in"></a>
#### assertSeeIn

Potwierdź, że podany tekst jest obecny w selektorze:

```php
$browser->assertSeeIn($selector, $text);
```

<a name="assert-dont-see-in"></a>
#### assertDontSeeIn

Potwierdź, że podany tekst jest nieobecny w selektorze:

```php
$browser->assertDontSeeIn($selector, $text);
```

<a name="assert-see-anything-in"></a>
#### assertSeeAnythingIn

Potwierdź, że dowolny tekst jest obecny w selektorze:

```php
$browser->assertSeeAnythingIn($selector);
```

<a name="assert-see-nothing-in"></a>
#### assertSeeNothingIn

Potwierdź, że żaden tekst nie jest obecny w selektorze:

```php
$browser->assertSeeNothingIn($selector);
```

<a name="assert-count"></a>
#### assertCount

Potwierdź, że elementy pasujące do podanego selektora pojawiają się określoną liczbę razy:

```php
$browser->assertCount($selector, $count);
```

<a name="assert-script"></a>
#### assertScript

Potwierdź, że podane wyrażenie JavaScript oblicza się do podanej wartości:

```php
$browser->assertScript('window.isLoaded')
    ->assertScript('document.readyState', 'complete');
```

<a name="assert-source-has"></a>
#### assertSourceHas

Potwierdź, że podany kod źródłowy jest obecny na stronie:

```php
$browser->assertSourceHas($code);
```

<a name="assert-source-missing"></a>
#### assertSourceMissing

Potwierdź, że podany kod źródłowy jest nieobecny na stronie:

```php
$browser->assertSourceMissing($code);
```

<a name="assert-see-link"></a>
#### assertSeeLink

Potwierdź, że podany link jest obecny na stronie:

```php
$browser->assertSeeLink($linkText);
```

<a name="assert-dont-see-link"></a>
#### assertDontSeeLink

Potwierdź, że podany link jest nieobecny na stronie:

```php
$browser->assertDontSeeLink($linkText);
```

<a name="assert-input-value"></a>
#### assertInputValue

Potwierdź, że podane pole input ma daną wartość:

```php
$browser->assertInputValue($field, $value);
```

<a name="assert-input-value-is-not"></a>
#### assertInputValueIsNot

Potwierdź, że podane pole input nie ma danej wartości:

```php
$browser->assertInputValueIsNot($field, $value);
```

<a name="assert-checked"></a>
#### assertChecked

Potwierdź, że podane pole wyboru jest zaznaczone:

```php
$browser->assertChecked($field);
```

<a name="assert-not-checked"></a>
#### assertNotChecked

Potwierdź, że podane pole wyboru nie jest zaznaczone:

```php
$browser->assertNotChecked($field);
```

<a name="assert-indeterminate"></a>
#### assertIndeterminate

Potwierdź, że podane pole wyboru jest w stanie nieokreślonym:

```php
$browser->assertIndeterminate($field);
```

<a name="assert-radio-selected"></a>
#### assertRadioSelected

Potwierdź, że podane pole radiowe jest wybrane:

```php
$browser->assertRadioSelected($field, $value);
```

<a name="assert-radio-not-selected"></a>
#### assertRadioNotSelected

Potwierdź, że podane pole radiowe nie jest wybrane:

```php
$browser->assertRadioNotSelected($field, $value);
```

<a name="assert-selected"></a>
#### assertSelected

Potwierdź, że podana lista rozwijana ma wybraną daną wartość:

```php
$browser->assertSelected($field, $value);
```

<a name="assert-not-selected"></a>
#### assertNotSelected

Potwierdź, że podana lista rozwijana nie ma wybranej danej wartości:

```php
$browser->assertNotSelected($field, $value);
```

<a name="assert-select-has-options"></a>
#### assertSelectHasOptions

Potwierdź, że podana tablica wartości jest dostępna do wyboru:

```php
$browser->assertSelectHasOptions($field, $values);
```

<a name="assert-select-missing-options"></a>
#### assertSelectMissingOptions

Potwierdź, że podana tablica wartości nie jest dostępna do wyboru:

```php
$browser->assertSelectMissingOptions($field, $values);
```

<a name="assert-select-has-option"></a>
#### assertSelectHasOption

Potwierdź, że podana wartość jest dostępna do wyboru w podanym polu:

```php
$browser->assertSelectHasOption($field, $value);
```

<a name="assert-select-missing-option"></a>
#### assertSelectMissingOption

Potwierdź, że podana wartość nie jest dostępna do wyboru:

```php
$browser->assertSelectMissingOption($field, $value);
```

<a name="assert-value"></a>
#### assertValue

Potwierdź, że element pasujący do podanego selektora ma daną wartość:

```php
$browser->assertValue($selector, $value);
```

<a name="assert-value-is-not"></a>
#### assertValueIsNot

Potwierdź, że element pasujący do podanego selektora nie ma danej wartości:

```php
$browser->assertValueIsNot($selector, $value);
```

<a name="assert-attribute"></a>
#### assertAttribute

Potwierdź, że element pasujący do podanego selektora ma daną wartość w podanym atrybucie:

```php
$browser->assertAttribute($selector, $attribute, $value);
```

<a name="assert-attribute-missing"></a>
#### assertAttributeMissing

Potwierdź, że element pasujący do podanego selektora nie ma podanego atrybutu:

```php
$browser->assertAttributeMissing($selector, $attribute);
```

<a name="assert-attribute-contains"></a>
#### assertAttributeContains

Potwierdź, że element pasujący do podanego selektora zawiera daną wartość w podanym atrybucie:

```php
$browser->assertAttributeContains($selector, $attribute, $value);
```

<a name="assert-attribute-doesnt-contain"></a>
#### assertAttributeDoesntContain

Potwierdź, że element pasujący do podanego selektora nie zawiera danej wartości w podanym atrybucie:

```php
$browser->assertAttributeDoesntContain($selector, $attribute, $value);
```

<a name="assert-aria-attribute"></a>
#### assertAriaAttribute

Potwierdź, że element pasujący do podanego selektora ma daną wartość w podanym atrybucie aria:

```php
$browser->assertAriaAttribute($selector, $attribute, $value);
```

Na przykład, mając znacznik `<button aria-label="Add"></button>`, możesz potwierdzić atrybut `aria-label` w następujący sposób:

```php
$browser->assertAriaAttribute('button', 'label', 'Add')
```

<a name="assert-data-attribute"></a>
#### assertDataAttribute

Potwierdź, że element pasujący do podanego selektora ma daną wartość w podanym atrybucie data:

```php
$browser->assertDataAttribute($selector, $attribute, $value);
```

Na przykład, mając znacznik `<tr id="row-1" data-content="attendees"></tr>`, możesz potwierdzić atrybut `data-label` w następujący sposób:

```php
$browser->assertDataAttribute('#row-1', 'content', 'attendees')
```

<a name="assert-visible"></a>
#### assertVisible

Potwierdź, że element pasujący do podanego selektora jest widoczny:

```php
$browser->assertVisible($selector);
```

<a name="assert-present"></a>
#### assertPresent

Potwierdź, że element pasujący do podanego selektora jest obecny w źródle:

```php
$browser->assertPresent($selector);
```

<a name="assert-not-present"></a>
#### assertNotPresent

Potwierdź, że element pasujący do podanego selektora nie jest obecny w źródle:

```php
$browser->assertNotPresent($selector);
```

<a name="assert-missing"></a>
#### assertMissing

Potwierdź, że element pasujący do podanego selektora nie jest widoczny:

```php
$browser->assertMissing($selector);
```

<a name="assert-input-present"></a>
#### assertInputPresent

Potwierdź, że pole input o podanej nazwie jest obecne:

```php
$browser->assertInputPresent($name);
```

<a name="assert-input-missing"></a>
#### assertInputMissing

Potwierdź, że pole input o podanej nazwie nie jest obecne w źródle:

```php
$browser->assertInputMissing($name);
```

<a name="assert-dialog-opened"></a>
#### assertDialogOpened

Potwierdź, że okno dialogowe JavaScript z podanym komunikatem zostało otwarte:

```php
$browser->assertDialogOpened($message);
```

<a name="assert-enabled"></a>
#### assertEnabled

Potwierdź, że podane pole jest włączone:

```php
$browser->assertEnabled($field);
```

<a name="assert-disabled"></a>
#### assertDisabled

Potwierdź, że podane pole jest wyłączone:

```php
$browser->assertDisabled($field);
```

<a name="assert-button-enabled"></a>
#### assertButtonEnabled

Potwierdź, że podany przycisk jest włączony:

```php
$browser->assertButtonEnabled($button);
```

<a name="assert-button-disabled"></a>
#### assertButtonDisabled

Potwierdź, że podany przycisk jest wyłączony:

```php
$browser->assertButtonDisabled($button);
```

<a name="assert-focused"></a>
#### assertFocused

Potwierdź, że podane pole ma fokus:

```php
$browser->assertFocused($field);
```

<a name="assert-not-focused"></a>
#### assertNotFocused

Potwierdź, że podane pole nie ma fokusu:

```php
$browser->assertNotFocused($field);
```

<a name="assert-authenticated"></a>
#### assertAuthenticated

Potwierdź, że użytkownik jest uwierzytelniony:

```php
$browser->assertAuthenticated();
```

<a name="assert-guest"></a>
#### assertGuest

Potwierdź, że użytkownik nie jest uwierzytelniony:

```php
$browser->assertGuest();
```

<a name="assert-authenticated-as"></a>
#### assertAuthenticatedAs

Potwierdź, że użytkownik jest uwierzytelniony jako podany użytkownik:

```php
$browser->assertAuthenticatedAs($user);
```

<a name="assert-vue"></a>
#### assertVue

Dusk pozwala nawet na wykonywanie asercji dotyczących stanu danych [komponentów Vue](https://vuejs.org). Na przykład wyobraź sobie, że Twoja aplikacja zawiera następujący komponent Vue:

    // HTML...

    <profile dusk="profile-component"></profile>

    // Definicja komponentu...

    Vue.component('profile', {
        template: '<div>{{ user.name }}</div>',

        data: function () {
            return {
                user: {
                    name: 'Taylor'
                }
            };
        }
    });

Możesz potwierdzić stan komponentu Vue w następujący sposób:

```php tab=Pest
test('vue', function () {
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
            ->assertVue('user.name', 'Taylor', '@profile-component');
    });
});
```

```php tab=PHPUnit
/**
 * Podstawowy przykład testu Vue.
 */
public function test_vue(): void
{
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
            ->assertVue('user.name', 'Taylor', '@profile-component');
    });
}
```

<a name="assert-vue-is-not"></a>
#### assertVueIsNot

Potwierdź, że dana właściwość danych komponentu Vue nie pasuje do podanej wartości:

```php
$browser->assertVueIsNot($property, $value, $componentSelector = null);
```

<a name="assert-vue-contains"></a>
#### assertVueContains

Potwierdź, że dana właściwość danych komponentu Vue jest tablicą i zawiera daną wartość:

```php
$browser->assertVueContains($property, $value, $componentSelector = null);
```

<a name="assert-vue-doesnt-contain"></a>
#### assertVueDoesntContain

Potwierdź, że dana właściwość danych komponentu Vue jest tablicą i nie zawiera danej wartości:

```php
$browser->assertVueDoesntContain($property, $value, $componentSelector = null);
```

<a name="pages"></a>
## Strony

Niekiedy testy wymagają wykonania kilku skomplikowanych akcji w sekwencji. Może to uczynić Twoje testy trudniejszymi do odczytania i zrozumienia. Strony Dusk pozwalają zdefiniować ekspresyjne akcje, które mogą być następnie wykonane na danej stronie za pomocą jednej metody. Strony pozwalają również definiować skróty do popularnych selektorów dla Twojej aplikacji lub dla pojedynczej strony.

<a name="generating-pages"></a>
### Generowanie stron

Aby wygenerować obiekt strony, wykonaj komendę Artisan `dusk:page`. Wszystkie obiekty stron zostaną umieszczone w katalogu `tests/Browser/Pages` Twojej aplikacji:

```shell
php artisan dusk:page Login
```

<a name="configuring-pages"></a>
### Konfigurowanie stron

Domyślnie strony mają trzy metody: `url`, `assert` i `elements`. Omówimy metody `url` i `assert` teraz. Metoda `elements` zostanie [omówiona bardziej szczegółowo poniżej](#shorthand-selectors).

<a name="the-url-method"></a>
#### Metoda `url`

Metoda `url` powinna zwracać ścieżkę URL reprezentującą stronę. Dusk użyje tego URL podczas nawigacji do strony w przeglądarce:

```php
/**
 * Pobierz URL dla strony.
 */
public function url(): string
{
    return '/login';
}
```

<a name="the-assert-method"></a>
#### Metoda `assert`

Metoda `assert` może wykonywać wszelkie asercje niezbędne do potwierdzenia, że przeglądarka faktycznie znajduje się na danej stronie. Nie jest faktycznie konieczne umieszczanie czegokol wiek w tej metodzie; jednak możesz swobodnie wykonywac te asercje, jeśli chcesz. Te asercje zostaną uruchomione automatycznie podczas nawigacji do strony:

```php
/**
 * Assert that the browser is on the page.
 */
public function assert(Browser $browser): void
{
    $browser->assertPathIs($this->url());
}
```

<a name="navigating-to-pages"></a>
### Nawigowanie do stron

Po zdefiniowaniu strony możesz nawigować do niej za pomocą metody `visit`:

```php
use Tests\Browser\Pages\Login;

$browser->visit(new Login);
```

Niekiedy możesz już znajdować się na danej stronie i musisz "ładować" selektory i metody strony do bieżącego kontekstu testu. Jest to powszechne, gdy naciskasz przycisk i jesteś przekierowywany na daną stronę bez wyraźnego nawigowania do niej. W tej sytuacji możesz użyć metody `on`, aby załadować stronę:

```php
use Tests\Browser\Pages\CreatePlaylist;

$browser->visit('/dashboard')
    ->clickLink('Create Playlist')
    ->on(new CreatePlaylist)
    ->assertSee('@create');
```

<a name="shorthand-selectors"></a>
### Selektory skrótowe

Metoda `elements` w klasach stron pozwala zdefiniować szybkie, łatwe do zapamiętania skróty dla dowolnego selektora CSS na Twojej stronie. Na przykład zdefiniujmy skrót dla pola input "email" strony logowania aplikacji:

```php
/**
 * Pobierz skróty elementów dla strony.
 *
 * @return array<string, string>
 */
public function elements(): array
{
    return [
        '@email' => 'input[name=email]',
    ];
}
```

Po zdefiniowaniu skrótu możesz używać selektora skrótowego wszedzie tam, gdzie normalnie używałbyś pełnego selektora CSS:

```php
$browser->type('@email', 'taylor@laravel.com');
```

<a name="global-shorthand-selectors"></a>
#### Globalne selektory skrótowe

Po zainstalowaniu Dusk podstawowa klasa `Page` zostanie umieszczona w katalogu `tests/Browser/Pages`. Ta klasa zawiera metodę `siteElements`, która może być użyta do zdefiniowania globalnych selektorów skrótowych, które powinny być dostępne na każdej stronie w całej aplikacji:

```php
/**
 * Get the global element shortcuts for the site.
 *
 * @return array<string, string>
 */
public static function siteElements(): array
{
    return [
        '@element' => '#selector',
    ];
}
```

<a name="page-methods"></a>
### Metody stron

Oprócz domyślnych metod zdefiniowanych na stronach możesz definiować dodatkowe metody, które mogą być używane w całych testach. Na przykład wyobraźmy sobie, że budujemy aplikację do zarządzania muzyką. Powszechną akcją dla jednej strony aplikacji może być tworzenie playlisty. Zamiast przepisywać logikę tworzenia playlisty w każdym teście, możesz zdefiniować metodę `createPlaylist` w klasie strony:

```php
<?php

namespace Tests\Browser\Pages;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Page;

class Dashboard extends Page
{
    // Inne metody strony...

    /**
     * Utwórz nową playlistę.
     */
    public function createPlaylist(Browser $browser, string $name): void
    {
        $browser->type('name', $name)
            ->check('share')
            ->press('Create Playlist');
    }
}
```

Po zdefiniowaniu metody możesz jej używać w dowolnym teście wykorzystującym stronę. Instancja przeglądarki zostanie automatycznie przekazana jako pierwszy argument do niestandardowych metod stron:

```php
use Tests\Browser\Pages\Dashboard;

$browser->visit(new Dashboard)
    ->createPlaylist('My Playlist')
    ->assertSee('My Playlist');
```

<a name="components"></a>
## Komponenty

Komponenty są podobne do "obiektów strony" Dusk, ale są przeznaczone dla fragmentów interfejsu użytkownika i funkcjonalności, które są ponownie używane w całej aplikacji, takie jak pasek nawigacji lub okno powiadomień. Jako takie, komponenty nie są związane z konkretnymi adresami URL.

<a name="generating-components"></a>
### Generowanie komponentów

Aby wygenerować komponent, wykonaj komendę Artisan `dusk:component`. Nowe komponenty są umieszczane w katalogu `tests/Browser/Components`:

```shell
php artisan dusk:component DatePicker
```

Jak pokazano powyżej, "wybierak daty" jest przykładem komponentu, który może istnieć w Twojej aplikacji na różnych stronach. Może stać się uciążliwe ręczne pisanie logiki automatyzacji przeglądarki do wybierania daty w dziesiątkach testów w całym zestawie testów. Zamiast tego możemy zdefiniować komponent Dusk reprezentujący wybierak daty, co pozwala nam enkapsulować tę logikę w komponencie:

```php
<?php

namespace Tests\Browser\Components;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Component as BaseComponent;

class DatePicker extends BaseComponent
{
    /**
     * Get the root selector for the component.
     */
    public function selector(): string
    {
        return '.date-picker';
    }

    /**
     * Assert that the browser page contains the component.
     */
    public function assert(Browser $browser): void
    {
        $browser->assertVisible($this->selector());
    }

    /**
     * Get the element shortcuts for the component.
     *
     * @return array<string, string>
     */
    public function elements(): array
    {
        return [
            '@date-field' => 'input.datepicker-input',
            '@year-list' => 'div > div.datepicker-years',
            '@month-list' => 'div > div.datepicker-months',
            '@day-list' => 'div > div.datepicker-days',
        ];
    }

    /**
     * Select the given date.
     */
    public function selectDate(Browser $browser, int $year, int $month, int $day): void
    {
        $browser->click('@date-field')
            ->within('@year-list', function (Browser $browser) use ($year) {
                $browser->click($year);
            })
            ->within('@month-list', function (Browser $browser) use ($month) {
                $browser->click($month);
            })
            ->within('@day-list', function (Browser $browser) use ($day) {
                $browser->click($day);
            });
    }
}
```

<a name="using-components"></a>
### Używanie komponentów

Po zdefiniowaniu komponentu możemy łatwo wybrać datę w wybiraku daty z dowolnego testu. Jeśli logika niezbędna do wyboru daty się zmieni, musimy zaktualizować tylko komponent:

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\Browser\Components\DatePicker;

pest()->use(DatabaseMigrations::class);

test('basic example', function () {
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
            ->within(new DatePicker, function (Browser $browser) {
                $browser->selectDate(2019, 1, 30);
            })
            ->assertSee('January');
    });
});
```

```php tab=PHPUnit
<?php

namespace Tests\Browser;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\Browser\Components\DatePicker;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    /**
     * A basic component test example.
     */
    public function test_basic_example(): void
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                ->within(new DatePicker, function (Browser $browser) {
                    $browser->selectDate(2019, 1, 30);
                })
                ->assertSee('January');
        });
    }
}
```

Metoda `component` może być użyta do pobrania instancji przeglądarki objętej zakresem podanego komponentu:

```php
$datePicker = $browser->component(new DatePickerComponent);

$datePicker->selectDate(2019, 1, 30);

$datePicker->assertSee('January');
```

<a name="continuous-integration"></a>
## Ciągła integracja

> [!WARNING]
> Większość konfiguracji ciągłej integracji Dusk oczekuje, że Twoja aplikacja Laravel będzie obsługiwana przy użyciu wbudowanego serwera programistycznego PHP na porcie 8000. Dlatego przed kontynuowaniem powinieneś upewnić się, że Twoje środowisko ciągłej integracji ma wartość zmiennej środowiskowej `APP_URL` ustawioną na `http://127.0.0.1:8000`.

<a name="running-tests-on-heroku-ci"></a>
### Heroku CI

Aby uruchomić testy Dusk na [Heroku CI](https://www.heroku.com/continuous-integration), dodaj następujący buildpack Google Chrome i skrypty do pliku `app.json` Heroku:

```json
{
  "environments": {
    "test": {
      "buildpacks": [
        { "url": "heroku/php" },
        { "url": "https://github.com/heroku/heroku-buildpack-chrome-for-testing" }
      ],
      "scripts": {
        "test-setup": "cp .env.testing .env",
        "test": "nohup bash -c './vendor/laravel/dusk/bin/chromedriver-linux --port=9515 > /dev/null 2>&1 &' && nohup bash -c 'php artisan serve --no-reload > /dev/null 2>&1 &' && php artisan dusk"
      }
    }
  }
}
```

<a name="running-tests-on-travis-ci"></a>
### Travis CI

Aby uruchomić testy Dusk na [Travis CI](https://travis-ci.org), użyj następującej konfiguracji `.travis.yml`. Ponieważ Travis CI nie jest środowiskiem graficznym, będziemy musieli podjąć kilka dodatkowych kroków, aby uruchomić przeglądarkę Chrome. Ponadto użyjemy `php artisan serve`, aby uruchomić wbudowany serwer internetowy PHP:

```yaml
language: php

php:
  - 8.2

addons:
  chrome: stable

install:
  - cp .env.testing .env
  - travis_retry composer install --no-interaction --prefer-dist
  - php artisan key:generate
  - php artisan dusk:chrome-driver

before_script:
  - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
  - php artisan serve --no-reload &

script:
  - php artisan dusk
```

<a name="running-tests-on-github-actions"></a>
### GitHub Actions

Jeśli używasz [GitHub Actions](https://github.com/features/actions) do uruchamiania testów Dusk, możesz użyć następującego pliku konfiguracyjnego jako punktu wyjścia. Podobnie jak TravisCI, użyjemy komendy `php artisan serve`, aby uruchomić wbudowany serwer internetowy PHP:

```yaml
name: CI
on: [push]
jobs:

  dusk-php:
    runs-on: ubuntu-latest
    env:
      APP_URL: "http://127.0.0.1:8000"
      DB_USERNAME: root
      DB_PASSWORD: root
      MAIL_MAILER: log
    steps:
      - uses: actions/checkout@v5
      - name: Prepare The Environment
        run: cp .env.example .env
      - name: Create Database
        run: |
          sudo systemctl start mysql
          mysql --user="root" --password="root" -e "CREATE DATABASE \`my-database\` character set UTF8mb4 collate utf8mb4_bin;"
      - name: Install Composer Dependencies
        run: composer install --no-progress --prefer-dist --optimize-autoloader
      - name: Generate Application Key
        run: php artisan key:generate
      - name: Upgrade Chrome Driver
        run: php artisan dusk:chrome-driver --detect
      - name: Start Chrome Driver
        run: ./vendor/laravel/dusk/bin/chromedriver-linux --port=9515 &
      - name: Run Laravel Server
        run: php artisan serve --no-reload &
      - name: Run Dusk Tests
        run: php artisan dusk
      - name: Upload Screenshots
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: screenshots
          path: tests/Browser/screenshots
      - name: Upload Console Logs
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: console
          path: tests/Browser/console
```

<a name="running-tests-on-chipper-ci"></a>
### Chipper CI

Jeśli używasz [Chipper CI](https://chipperci.com) do uruchamiania testów Dusk, możesz użyć następującego pliku konfiguracyjnego jako punktu wyjścia. Użyjemy wbudowanego serwera PHP do uruchomienia Laravel, abyśmy mogli nasłuchiwać żądań:

```yaml
# file .chipperci.yml
version: 1

environment:
  php: 8.2
  node: 16

# Include Chrome in the build environment
services:
  - dusk

# Build all commits
on:
   push:
      branches: .*

pipeline:
  - name: Setup
    cmd: |
      cp -v .env.example .env
      composer install --no-interaction --prefer-dist --optimize-autoloader
      php artisan key:generate

      # Create a dusk env file, ensuring APP_URL uses BUILD_HOST
      cp -v .env .env.dusk.ci
      sed -i "s@APP_URL=.*@APP_URL=http://$BUILD_HOST:8000@g" .env.dusk.ci

  - name: Compile Assets
    cmd: |
      npm ci --no-audit
      npm run build

  - name: Browser Tests
    cmd: |
      php -S [::0]:8000 -t public 2>server.log &
      sleep 2
      php artisan dusk:chrome-driver $CHROME_DRIVER
      php artisan dusk --env=ci
```

Aby dowiedzieć się więcej o uruchamianiu testów Dusk na Chipper CI, w tym o używaniu baz danych, zapoznaj się z [oficjalną dokumentacją Chipper CI](https://chipperci.com/docs/testing/laravel-dusk-new/).
