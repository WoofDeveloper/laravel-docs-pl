# Laravel Telescope

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
    - [Instalacja Tylko Lokalna](#local-only-installation)
    - [Konfiguracja](#configuration)
    - [Przycinanie Danych](#data-pruning)
    - [Autoryzacja Panelu](#dashboard-authorization)
- [Aktualizacja Telescope](#upgrading-telescope)
- [Filtrowanie](#filtering)
    - [Wpisy](#filtering-entries)
    - [Partie](#filtering-batches)
- [Tagowanie](#tagging)
- [Dostępne Obserwatory](#available-watchers)
    - [Obserwator Partii](#batch-watcher)
    - [Obserwator Cache](#cache-watcher)
    - [Obserwator Poleceń](#command-watcher)
    - [Obserwator Dump](#dump-watcher)
    - [Obserwator Zdarzeń](#event-watcher)
    - [Obserwator Wyjątków](#exception-watcher)
    - [Obserwator Bram](#gate-watcher)
    - [Obserwator Klienta HTTP](#http-client-watcher)
    - [Obserwator Zadań](#job-watcher)
    - [Obserwator Logów](#log-watcher)
    - [Obserwator Poczty](#mail-watcher)
    - [Obserwator Modeli](#model-watcher)
    - [Obserwator Powiadomień](#notification-watcher)
    - [Obserwator Zapytań](#query-watcher)
    - [Obserwator Redis](#redis-watcher)
    - [Obserwator Żądań](#request-watcher)
    - [Obserwator Harmonogramu](#schedule-watcher)
    - [Obserwator Widoków](#view-watcher)
- [Wyświetlanie Awatarów Użytkowników](#displaying-user-avatars)

<a name="introduction"></a>
## Wprowadzenie

[Laravel Telescope](https://github.com/laravel/telescope) jest wspaniałym towarzyszem dla Twojego lokalnego środowiska deweloperskiego Laravel. Telescope zapewnia wgląd w żądania przychodzące do Twojej aplikacji, wyjątki, wpisy logów, zapytania do bazy danych, zadania w kolejce, wiadomości e-mail, powiadomienia, operacje cache, zaplanowane zadania, zrzuty zmiennych i wiele więcej.

<img src="https://laravel.com/img/docs/telescope-example.png">

<a name="installation"></a>
## Instalacja

Możesz użyć menedżera pakietów Composer, aby zainstalować Telescope w swoim projekcie Laravel:

```shell
composer require laravel/telescope
```

Po zainstalowaniu Telescope opublikuj jego zasoby i migracje za pomocą polecenia Artisan `telescope:install`. Po zainstalowaniu Telescope powinieneś również uruchomić polecenie `migrate`, aby utworzyć tabele potrzebne do przechowywania danych Telescope:

```shell
php artisan telescope:install

php artisan migrate
```

Wreszcie, możesz uzyskać dostęp do panelu Telescope poprzez trasę `/telescope`.

<a name="local-only-installation"></a>
### Instalacja Tylko Lokalna

Jeśli planujesz używać Telescope tylko do wspomagania lokalnego rozwoju, możesz zainstalować Telescope używając flagi `--dev`:

```shell
composer require laravel/telescope --dev

php artisan telescope:install

php artisan migrate
```

Po uruchomieniu `telescope:install`, powinieneś usunąć rejestrację dostawcy usług `TelescopeServiceProvider` z pliku konfiguracyjnego `bootstrap/providers.php` Twojej aplikacji. Zamiast tego ręcznie zarejestruj dostawców usług Telescope w metodzie `register` klasy `App\Providers\AppServiceProvider`. Upewnimy się, że obecne środowisko to `local` przed zarejestrowaniem dostawców:

```php
/**
 * Register any application services.
 */
public function register(): void
{
    if ($this->app->environment('local') && class_exists(\Laravel\Telescope\TelescopeServiceProvider::class)) {
        $this->app->register(\Laravel\Telescope\TelescopeServiceProvider::class);
        $this->app->register(TelescopeServiceProvider::class);
    }
}
```

Wreszcie, powinieneś również zapobiec [automatycznemu wykrywaniu](/docs/{{version}}/packages#package-discovery) pakietu Telescope, dodając następujące do pliku `composer.json`:

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "laravel/telescope"
        ]
    }
},
```

<a name="configuration"></a>
### Konfiguracja

Po opublikowaniu zasobów Telescope, jego główny plik konfiguracyjny będzie znajdował się w `config/telescope.php`. Ten plik konfiguracyjny umożliwia skonfigurowanie [opcji obserwatora](#available-watchers). Każda opcja konfiguracyjna zawiera opis swojego przeznaczenia, więc koniecznie dokładnie zbadaj ten plik.

Jeśli chcesz, możesz całkowicie wyłączyć zbieranie danych przez Telescope za pomocą opcji konfiguracyjnej `enabled`:

```php
'enabled' => env('TELESCOPE_ENABLED', true),
```

<a name="data-pruning"></a>
### Przycinanie Danych

Bez przycinania, tabela `telescope_entries` może bardzo szybko gromadzić rekordy. Aby temu zaradzić, powinieneś [zaplanować](/docs/{{version}}/scheduling) codzienne uruchamianie polecenia Artisan `telescope:prune`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('telescope:prune')->daily();
```

Domyślnie wszystkie wpisy starsze niż 24 godziny zostaną przycięte. Możesz użyć opcji `hours` podczas wywoływania polecenia, aby określić, jak długo zachowywać dane Telescope. Na przykład, następujące polecenie usunie wszystkie rekordy utworzone ponad 48 godzin temu:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('telescope:prune --hours=48')->daily();
```

<a name="dashboard-authorization"></a>
### Autoryzacja Panelu

Panel Telescope może być dostępny przez trasę `/telescope`. Domyślnie będziesz mógł uzyskać dostęp do tego panelu tylko w środowisku `local`. W pliku `app/Providers/TelescopeServiceProvider.php` znajduje się definicja [bramy autoryzacji](/docs/{{version}}/authorization#gates). Ta brama autoryzacji kontroluje dostęp do Telescope w środowiskach **nie-lokalnych**. Możesz modyfikować tę bramę według potrzeb, aby ograniczyć dostęp do Twojej instalacji Telescope:

```php
use App\Models\User;

/**
 * Register the Telescope gate.
 *
 * This gate determines who can access Telescope in non-local environments.
 */
protected function gate(): void
{
    Gate::define('viewTelescope', function (User $user) {
        return in_array($user->email, [
            'taylor@laravel.com',
        ]);
    });
}
```

> [!WARNING]
> Powinieneś upewnić się, że zmienisz swoją zmienną środowiskową `APP_ENV` na `production` w środowisku produkcyjnym. W przeciwnym razie Twoja instalacja Telescope będzie publicznie dostępna.

<a name="upgrading-telescope"></a>
## Aktualizacja Telescope

Podczas aktualizacji do nowej głównej wersji Telescope, ważne jest, aby dokładnie przejrzeć [przewodnik aktualizacji](https://github.com/laravel/telescope/blob/master/UPGRADE.md).

Ponadto, podczas aktualizacji do dowolnej nowej wersji Telescope, powinieneś ponownie opublikować zasoby Telescope:

```shell
php artisan telescope:publish
```

Aby zasoby były aktualne i uniknąć problemów w przyszłych aktualizacjach, możesz dodać polecenie `vendor:publish --tag=laravel-assets` do skryptów `post-update-cmd` w pliku `composer.json` Twojej aplikacji:

```json
{
    "scripts": {
        "post-update-cmd": [
            "@php artisan vendor:publish --tag=laravel-assets --ansi --force"
        ]
    }
}
```

<a name="filtering"></a>
## Filtrowanie

<a name="filtering-entries"></a>
### Wpisy

Możesz filtrować dane rejestrowane przez Telescope za pomocą zamknięcia `filter`, które jest zdefiniowane w klasie `App\Providers\TelescopeServiceProvider`. Domyślnie to zamknięcie rejestruje wszystkie dane w środowisku `local` oraz wyjątki, nieudane zadania, zaplanowane zadania i dane z monitorowanymi tagami we wszystkich innych środowiskach:

```php
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->hideSensitiveRequestDetails();

    Telescope::filter(function (IncomingEntry $entry) {
        if ($this->app->environment('local')) {
            return true;
        }

        return $entry->isReportableException() ||
            $entry->isFailedJob() ||
            $entry->isScheduledTask() ||
            $entry->isSlowQuery() ||
            $entry->hasMonitoredTag();
    });
}
```

<a name="filtering-batches"></a>
### Partie

Choc zamknięcie `filter` filtruje dane dla pojedynczych wpisów, możesz użyć metody `filterBatch`, aby zarejestrować zamknięcie filtrujące wszystkie dane dla danego żądania lub polecenia konsolowego. Jeśli zamknięcie zwróci `true`, wszystkie wpisy są rejestrowane przez Telescope:

```php
use Illuminate\Support\Collection;
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->hideSensitiveRequestDetails();

    Telescope::filterBatch(function (Collection $entries) {
        if ($this->app->environment('local')) {
            return true;
        }

        return $entries->contains(function (IncomingEntry $entry) {
            return $entry->isReportableException() ||
                $entry->isFailedJob() ||
                $entry->isScheduledTask() ||
                $entry->isSlowQuery() ||
                $entry->hasMonitoredTag();
            });
    });
}
```

<a name="tagging"></a>
## Tagowanie

Telescope umożliwia wyszukiwanie wpisów po "tagu". Często tagi to nazwy klas modeli Eloquent lub identyfikatory uwierzytelnionych użytkowników, które Telescope automatycznie dodaje do wpisów. Czasami możesz chcieć dołączyć własne niestandardowe tagi do wpisów. Aby to osiągnąć, możesz użyć metody `Telescope::tag`. Metoda `tag` akceptuje zamknięcie, które powinno zwrócić tablicę tagów. Tagi zwrócone przez zamknięcie zostaną połączone z wszelkimi tagami, które Telescope automatycznie dołączy do wpisu. Zazwyczaj powinieneś wywołać metodę `tag` w metodzie `register` klasy `App\Providers\TelescopeServiceProvider`:

```php
use Laravel\Telescope\EntryType;
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->hideSensitiveRequestDetails();

    Telescope::tag(function (IncomingEntry $entry) {
        return $entry->type === EntryType::REQUEST
            ? ['status:'.$entry->content['response_status']]
            : [];
    });
}
```

<a name="available-watchers"></a>
## Dostępne Obserwatory

"Obserwatory" Telescope zbierają dane aplikacji, gdy żądanie lub polecenie konsolowe jest wykonywane. Możesz dostosować listę obserwatorów, które chcesz włączyć, w pliku konfiguracyjnym `config/telescope.php`:

```php
'watchers' => [
    Watchers\CacheWatcher::class => true,
    Watchers\CommandWatcher::class => true,
    // ...
],
```

Niektóre obserwatory pozwalają również na dodatkowe opcje dostosowania:

```php
'watchers' => [
    Watchers\QueryWatcher::class => [
        'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
        'slow' => 100,
    ],
    // ...
],
```

<a name="batch-watcher"></a>
### Obserwator Partii

Obserwator partii rejestruje informacje o kolejkowanych [partiach](/docs/{{version}}/queues#job-batching), w tym informacje o zadaniu i połączeniu.

<a name="cache-watcher"></a>
### Obserwator Cache

Obserwator cache rejestruje dane, gdy klucz cache zostanie trafiony, chybiony, zaktualizowany lub zapomniany.

<a name="command-watcher"></a>
### Obserwator Poleceń

Obserwator poleceń rejestruje argumenty, opcje, kod wyjścia i wyjście za każdym razem, gdy polecenie Artisan jest wykonywane. Jeśli chcesz wykluczyć pewne polecenia z rejestrowania przez obserwatora, możesz określić polecenie w opcji `ignore` w pliku `config/telescope.php`:

```php
'watchers' => [
    Watchers\CommandWatcher::class => [
        'enabled' => env('TELESCOPE_COMMAND_WATCHER', true),
        'ignore' => ['key:generate'],
    ],
    // ...
],
```

<a name="dump-watcher"></a>
### Obserwator Dump

Obserwator dump rejestruje i wyświetla zrzuty zmiennych w Telescope. Podczas używania Laravel, zmienne mogą być zrzucane za pomocą globalnej funkcji `dump`. Zakładka obserwatora dump musi być otwarta w przeglądarce, aby zrzut został zarejestrowany, w przeciwnym razie zrzuty zostaną zignorowane przez obserwatora.

<a name="event-watcher"></a>
### Obserwator Zdarzeń

Obserwator zdarzeń rejestruje payload, nasłuchiwacze i dane rozgłoszeniowe dla wszelkich [zdarzeń](/docs/{{version}}/events) wysyłanych przez Twoją aplikację. Wewnętrzne zdarzenia frameworka Laravel są ignorowane przez obserwatora zdarzeń.

<a name="exception-watcher"></a>
### Obserwator Wyjątków

Obserwator wyjątków rejestruje dane i ślad stosu dla wszelkich wyjątków, które są zgłaszane przez Twoją aplikację.

<a name="gate-watcher"></a>
### Obserwator Bram

Obserwator bram rejestruje dane i wynik sprawdzeń [bram i polityk](/docs/{{version}}/authorization) przez Twoją aplikację. Jeśli chcesz wykluczyć pewne uprawnienia z rejestrowania przez obserwatora, możesz określić je w opcji `ignore_abilities` w pliku `config/telescope.php`:

```php
'watchers' => [
    Watchers\GateWatcher::class => [
        'enabled' => env('TELESCOPE_GATE_WATCHER', true),
        'ignore_abilities' => ['viewNova'],
    ],
    // ...
],
```

<a name="http-client-watcher"></a>
### Obserwator Klienta HTTP

Obserwator klienta HTTP rejestruje wychodzące [żądania klienta HTTP](/docs/{{version}}/http-client) wykonywane przez Twoją aplikację.

<a name="job-watcher"></a>
### Obserwator Zadań

Obserwator zadań rejestruje dane i status wszelkich [zadań](/docs/{{version}}/queues) wysyłanych przez Twoją aplikację.

<a name="log-watcher"></a>
### Obserwator Logów

Obserwator logów rejestruje [dane logów](/docs/{{version}}/logging) dla wszelkich logów zapisywanych przez Twoją aplikację.

Domyślnie Telescope będzie rejestrowaM tylko logi na poziomie `error` i powyżej. Możesz jednak zmodyfikować opcję `level` w pliku konfiguracyjnym `config/telescope.php` Twojej aplikacji, aby zmienić to zachowanie:

```php
'watchers' => [
    Watchers\LogWatcher::class => [
        'enabled' => env('TELESCOPE_LOG_WATCHER', true),
        'level' => 'debug',
    ],

    // ...
],
```

<a name="mail-watcher"></a>
### Obserwator Poczty

Obserwator poczty umożliwia podgląd [wiadomości e-mail](/docs/{{version}}/mail) wysyłanych przez Twoją aplikację w przeglądarce wraz z powiązanymi danymi. Możesz również pobrać wiadomość e-mail jako plik `.eml`.

<a name="model-watcher"></a>
### Obserwator Modeli

Obserwator modeli rejestruje zmiany modeli, gdy zostanie wysyłane [zdarzenie modelu](/docs/{{version}}/eloquent#events) Eloquent. Możesz określić, które zdarzenia modeli powinny być rejestrowane za pomocą opcji `events` obserwatora:

```php
'watchers' => [
    Watchers\ModelWatcher::class => [
        'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
        'events' => ['eloquent.created*', 'eloquent.updated*'],
    ],
    // ...
],
```

Jeśli chcesz rejestrować liczbę modeli uwodnionych podczas danego żądania, włącz opcję `hydrations`:

```php
'watchers' => [
    Watchers\ModelWatcher::class => [
        'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
        'events' => ['eloquent.created*', 'eloquent.updated*'],
        'hydrations' => true,
    ],
    // ...
],
```

<a name="notification-watcher"></a>
### Obserwator Powiadomień

Obserwator powiadomień rejestruje wszystkie [powiadomienia](/docs/{{version}}/notifications) wysyłane przez Twoją aplikację. Jeśli powiadomienie wywołuje wiadomość e-mail i masz włączonego obserwatora poczty, wiadomość e-mail będzie również dostępna do podglądu na ekranie obserwatora poczty.

<a name="query-watcher"></a>
### Obserwator Zapytań

Obserwator zapytań rejestruje surowy SQL, powiązania i czas wykonania dla wszystkich zapytań, które są wykonywane przez Twoją aplikację. Obserwator również oznacza wszelkie zapytania wolniejsze niż 100 milisekund jako `slow`. Możesz dostosować próg wolnego zapytania za pomocą opcji `slow` obserwatora:

```php
'watchers' => [
    Watchers\QueryWatcher::class => [
        'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
        'slow' => 50,
    ],
    // ...
],
```

<a name="redis-watcher"></a>
### Obserwator Redis

Obserwator Redis rejestruje wszystkie polecenia [Redis](/docs/{{version}}/redis) wykonywane przez Twoją aplikację. Jeśli używasz Redis do cache'owania, polecenia cache będą również rejestrowane przez obserwatora Redis.

<a name="request-watcher"></a>
### Obserwator Żądań

Obserwator żądań rejestruje żądanie, nagłówki, sesję i dane odpowiedzi powiązane z wszelkimi żądaniami obsługiwanymi przez aplikację. Możesz ograniczyć rejestrowane dane odpowiedzi za pomocą opcji `size_limit` (w kilobajtach):

```php
'watchers' => [
    Watchers\RequestWatcher::class => [
        'enabled' => env('TELESCOPE_REQUEST_WATCHER', true),
        'size_limit' => env('TELESCOPE_RESPONSE_SIZE_LIMIT', 64),
    ],
    // ...
],
```

<a name="schedule-watcher"></a>
### Obserwator Harmonogramu

Obserwator harmonogramu rejestruje polecenie i wyjście wszelkich [zaplanowanych zadań](/docs/{{version}}/scheduling) uruchamianych przez Twoją aplikację.

<a name="view-watcher"></a>
### Obserwator Widoków

Obserwator widoków rejestruje nazwę, ścieżkę, dane i "kompozytorów" [widoku](/docs/{{version}}/views) używanych podczas renderowania widoków.

<a name="displaying-user-avatars"></a>
## Wyświetlanie Awatarów Użytkowników

Panel Telescope wyświetla awatar użytkownika, który był uwierzytelniony, gdy dany wpis został zapisany. Domyślnie Telescope będzie pobierać awatary za pomocą usługi internetowej Gravatar. Możesz jednak dostosować URL awatara, rejestrując callback w klasie `App\Providers\TelescopeServiceProvider`. Callback otrzyma identyfikator i adres e-mail użytkownika i powinien zwrócić URL obrazu awatara użytkownika:

```php
use App\Models\User;
use Laravel\Telescope\Telescope;

/**
 * Register any application services.
 */
public function register(): void
{
    // ...

    Telescope::avatar(function (?string $id, ?string $email) {
        return ! is_null($id)
            ? '/avatars/'.User::find($id)->avatar_path
            : '/generic-avatar.jpg';
    });
}
```
