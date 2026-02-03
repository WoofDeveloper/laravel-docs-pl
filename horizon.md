# Laravel Horizon

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
    - [Konfiguracja](#configuration)
    - [Autoryzacja panelu](#dashboard-authorization)
    - [Maksymalna liczba prób zadania](#max-job-attempts)
    - [Limit czasu zadania](#job-timeout)
    - [Wycofanie zadania](#job-backoff)
    - [Wyciszone zadania](#silenced-jobs)
- [Strategie równoważenia](#balancing-strategies)
    - [Automatyczne równoważenie](#auto-balancing)
    - [Proste równoważenie](#simple-balancing)
    - [Bez równoważenia](#no-balancing)
- [Aktualizacja Horizon](#upgrading-horizon)
- [Uruchamianie Horizon](#running-horizon)
    - [Wdrażanie Horizon](#deploying-horizon)
- [Tagi](#tags)
- [Powiadomienia](#notifications)
- [Metryki](#metrics)
- [Usuwanie nieudanych zadań](#deleting-failed-jobs)
- [Czyszczenie zadań z kolejek](#clearing-jobs-from-queues)

<a name="introduction"></a>
## Wprowadzenie

> [!NOTE]
> Przed zagłębieniem się w Laravel Horizon, powinieneś zapoznać się z podstawowymi [usługami kolejek](/docs/{{version}}/queues) Laravel. Horizon rozszerza kolejki Laravel o dodatkowe funkcje, które mogą być mylące, jeśli nie znasz już podstawowych funkcji kolejek oferowanych przez Laravel.

[Laravel Horizon](https://github.com/laravel/horizon) zapewnia piękny panel i konfigurację sterowaną kodem dla Twoich [kolejek Redis](/docs/{{version}}/queues) obsługiwanych przez Laravel. Horizon pozwala łatwo monitorować kluczowe metryki Twojego systemu kolejek, takie jak przepustowość zadań, czas wykonania i błędy zadań.

Korzystając z Horizon, cała konfiguracja workerów kolejek jest przechowywana w pojedynczym, prostym pliku konfiguracyjnym. Definiując konfigurację workerów aplikacji w pliku kontrolowanym wersją, możesz łatwo skalować lub modyfikować workery kolejek aplikacji podczas jej wdrażania.

<img src="https://laravel.com/img/docs/horizon-example.png">

<a name="installation"></a>
## Instalacja

> [!WARNING]
> Laravel Horizon wymaga używania [Redis](https://redis.io) do obsługi kolejek. Dlatego powinieneś upewnić się, że połączenie kolejki jest ustawione na `redis` w pliku konfiguracyjnym `config/queue.php` Twojej aplikacji. Horizon nie jest w tej chwili kompatybilny z Redis Cluster.

Możesz zainstalować Horizon w swoim projekcie używając menedżera pakietów Composer:

```shell
composer require laravel/horizon
```

Po zainstalowaniu Horizon, opublikuj jego zasoby używając polecenia Artisan `horizon:install`:

```shell
php artisan horizon:install
```

<a name="configuration"></a>
### Konfiguracja

Po opublikowaniu zasobów Horizon, główny plik konfiguracyjny będzie zlokalizowany w `config/horizon.php`. Ten plik konfiguracyjny pozwala skonfigurować opcje workerów kolejek dla Twojej aplikacji. Każda opcja konfiguracyjna zawiera opis swojego przeznaczenia, więc pamiętaj, aby dokładnie przejrzeć ten plik.

> [!WARNING]
> Horizon używa wewnętrznie połączenia Redis o nazwie `horizon`. Ta nazwa połączenia Redis jest zarezerwowana i nie powinna być przypisana do innego połączenia Redis w pliku konfiguracyjnym `database.php` ani jako wartość opcji `use` w pliku konfiguracyjnym `horizon.php`.

<a name="environments"></a>
#### Środowiska

Po instalacji, główna opcja konfiguracyjna Horizon, z którą powinieneś się zapoznać, to opcja konfiguracyjna `environments`. Ta opcja konfiguracyjna to tablica środowisk, w których działa Twoja aplikacja i definiuje opcje procesów workerów dla każdego środowiska. Domyślnie ten wpis zawiera środowisko `production` i `local`. Jednak możesz dodać więcej środowisk według potrzeb:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            'maxProcesses' => 10,
            'balanceMaxShift' => 1,
            'balanceCooldown' => 3,
        ],
    ],

    'local' => [
        'supervisor-1' => [
            'maxProcesses' => 3,
        ],
    ],
],
```

Możesz również zdefiniować wieloznaczne środowisko (`*`), które będzie używane, gdy nie zostanie znalezione żadne inne pasujące środowisko:

```php
'environments' => [
    // ...

    '*' => [
        'supervisor-1' => [
            'maxProcesses' => 3,
        ],
    ],
],
```

Kiedy uruchamiasz Horizon, użyje on opcji konfiguracyjnych procesów workerów dla środowiska, w którym działa Twoja aplikacja. Zazwyczaj środowisko jest określane przez wartość [zmiennej środowiskowej](/docs/{{version}}/configuration#determining-the-current-environment) `APP_ENV`. Na przykład, domyślne środowisko `local` Horizon jest skonfigurowane do uruchomienia trzech procesów workerów i automatycznego równoważenia liczby procesów workerów przypisanych do każdej kolejki. Domyślne środowisko `production` jest skonfigurowane do uruchomienia maksymalnie 10 procesów workerów i automatycznego równoważenia liczby procesów workerów przypisanych do każdej kolejki.

> [!WARNING]
> Powinieneś upewnić się, że sekcja `environments` w pliku konfiguracyjnym `horizon` zawiera wpis dla każdego [środowiska](/docs/{{version}}/configuration#environment-configuration), w którym planujesz uruchamiać Horizon.

<a name="supervisors"></a>
#### Nadzorcy

Jak możesz zobaczyć w domyślnym pliku konfiguracyjnym Horizon, każde środowisko może zawierać jednego lub więcej "nadzorców" (supervisors). Domyślnie plik konfiguracyjny definiuje tego nadzorcę jako `supervisor-1`; jednak możesz nazwać swoich nadzorców jak chcesz. Każdy nadzorca jest zasadniczo odpowiedzialny za "nadzorowanie" grupy procesów workerów i dba o równoważenie procesów workerów pomiędzy kolejkami.

Możesz dodać dodatkowych nadzorców do danego środowiska, jeśli chcesz zdefiniować nową grupę procesów workerów, które powinny działać w tym środowisku. Możesz to zrobić, jeśli chcesz zdefiniować inną strategię równoważenia lub liczbę procesów workerów dla danej kolejki używanej przez Twoją aplikację.

<a name="maintenance-mode"></a>
#### Tryb konserwacji

Kiedy Twoja aplikacja jest w [trybie konserwacji](/docs/{{version}}/configuration#maintenance-mode), zadania w kolejce nie będą przetwarzane przez Horizon, chyba że opcja `force` nadzorcy jest zdefiniowana jako `true` w pliku konfiguracyjnym Horizon:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'force' => true,
        ],
    ],
],
```

<a name="default-values"></a>
#### Wartości domyślne

W domyślnym pliku konfiguracyjnym Horizon zauważysz opcję konfiguracyjną `defaults`. Ta opcja konfiguracyjna określa domyślne wartości dla [nadzorców](#supervisors) Twojej aplikacji. Domyślne wartości konfiguracyjne nadzorcy zostaną scalone z konfiguracją nadzorcy dla każdego środowiska, pozwalając uniknąć niepotrzebnego powtarzania podczas definiowania nadzorców.

<a name="dashboard-authorization"></a>
### Autoryzacja panelu

Panel Horizon może być dostępny przez trasę `/horizon`. Domyślnie będziesz mógł uzyskać dostęp do tego panelu tylko w środowisku `local`. Jednak w pliku `app/Providers/HorizonServiceProvider.php` znajduje się definicja [bramy autoryzacji](/docs/{{version}}/authorization#gates). Ta brama autoryzacji kontroluje dostęp do Horizon w środowiskach **nie-lokalnych**. Możesz modyfikować tę bramę według potrzeb, aby ograniczyć dostęp do instalacji Horizon:

```php
/**
 * Register the Horizon gate.
 *
 * This gate determines who can access Horizon in non-local environments.
 */
protected function gate(): void
{
    Gate::define('viewHorizon', function (User $user) {
        return in_array($user->email, [
            'taylor@laravel.com',
        ]);
    });
}
```

<a name="alternative-authentication-strategies"></a>
#### Alternatywne strategie uwierzytelniania

Pamiętaj, że Laravel automatycznie wstrzykuje uwierzytelnionego użytkownika do domknięcia bramy. Jeśli Twoja aplikacja zapewnia bezpieczeństwo Horizon za pomocą innej metody, takiej jak ograniczenia IP, Twoi użytkownicy Horizon mogą nie potrzebować "logowania". Dlatego będziesz musiał zmienić sygnaturę domknięcia `function (User $user)` powyżej na `function (User $user = null)`, aby wymusić, że Laravel nie wymaga uwierzytelniania.

<a name="max-job-attempts"></a>
### Maksymalna liczba prób zadania

> [!NOTE]
> Przed dostępowaniem tych opcji, upewnij się, że znasz domyślne [usługi kolejek](/docs/{{version}}/queues#max-job-attempts-and-timeout) Laravel i koncepcję 'prób'.

Możesz zdefiniować maksymalną liczbę prób, jaką może zużyć zadanie, w konfiguracji nadzorcy:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'tries' => 10,
        ],
    ],
],
```

> [!NOTE]
> Ta opcja jest podobna do opcji `--tries` podczas używania polecenia Artisan do przetwarzania kolejek.

Dostosowanie opcji `tries` jest istotne przy używaniu middlewareów takich jak `WithoutOverlapping` lub `RateLimited`, ponieważ zużywają one próby. Aby to obsłużyć, dostosuj wartość konfiguracyjną `tries` na poziomie nadzorcy lub definiując właściwość `$tries` w klasie zadania.

Jeśli nie ustawisz opcji `tries`, Horizon domyślnie użyje pojedynczej próby, chyba że klasa zadania definiuje `$tries`, która ma pierwszęństwo przed konfiguracją Horizon.

Ustawienie `tries` lub `$tries` na 0 pozwala na nieograniczoną liczbę prób, co jest idealne, gdy liczba prób jest niepewna. Aby zapobiec nieskończonym niepowodzeniom, możesz ograniczyć liczbę dozwolonych wyjątków, ustawiając właściwość `$maxExceptions` w klasie zadania.

<a name="job-timeout"></a>
### Limit czasu zadania

Podobnie, możesz ustawić wartość `timeout` na poziomie nadzorcy, która określa, ile sekund proces workera może wykonywać zadanie, zanim zostanie przymusowo zakończony. Po zakończeniu, zadanie zostanie albo ponownie próbowane, albo oznaczone jako nieudane, w zależności od konfiguracji kolejki:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...¨
            'timeout' => 60,
        ],
    ],
],
```

> [!WARNING]
> Podczas używania strategii równoważenia `auto`, Horizon będzie uważał workery w trakcie wykonywania za "zawieszonych" i wymusi ich zakończenie po przekroczeniu limitu czasu Horizon podczas skalowania w dół. Zawsze upewnij się, że limit czasu Horizon jest większy niż każdy limit czasu na poziomie zadania, w przeciwnym razie zadania mogą zostać zakończone w trakcie wykonywania. Ponadto, wartość `timeout` powinna zawsze być co najmniej kilka sekund krótsza niż wartość `retry_after` zdefiniowana w pliku konfiguracyjnym `config/queue.php`. W przeciwnym razie Twoje zadania mogą zostać przetworzone dwukrotnie.

<a name="job-backoff"></a>
### Wycofanie zadania

Możesz zdefiniować wartość `backoff` na poziomie nadzorcy, aby określić, jak długo Horizon powinien czekać przed ponowną próbą zadania, które napotkało nieobsłużony wyjątek:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'backoff' => 10,
        ],
    ],
],
```

Możesz również skonfigurować "wykładnicze" wycofania, używając tablicy dla wartości `backoff`. W tym przykładzie, opóźnienie ponownej próby wyniesie 1 sekundę dla pierwszej próby, 5 sekund dla drugiej próby, 10 sekund dla trzeciej próby i 10 sekund dla każdej kolejnej próby, jeśli pozostają jeszcze próby:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'backoff' => [1, 5, 10],
        ],
    ],
],
```

<a name="silenced-jobs"></a>
### Wyciszone zadania

Czasami możesz nie być zainteresowany wyświetlaniem niektórych zadań wysyłanych przez Twoją aplikację lub pakiety stron trzecich. Zamiast zajmowania miejsca przez te zadania na liście "Zakończonych zadań", możesz je wyciszyć. Aby zacząć, dodaj nazwę klasy zadania do opcji konfiguracyjnej `silenced` w pliku konfiguracyjnym `horizon` Twojej aplikacji:

```php
'silenced' => [
    App\Jobs\ProcessPodcast::class,
],
```

Oprócz wyciszania poszczególnych klas zadań, Horizon obsługuje również wyciszanie zadań na podstawie [tagów](#tags). Może to być przydatne, jeśli chcesz ukryć wiele zadań, które mają wspólny tag:

```php
'silenced_tags' => [
    'notifications'
],
```

Alternatywnie, zadanie, które chcesz wyciszyć, może implementować interfejs `Laravel\Horizon\Contracts\Silenced`. Jeśli zadanie implementuje ten interfejs, zostanie automatycznie wyciszone, nawet jeśli nie jest obecne w tablicy konfiguracyjnej `silenced`:

```php
use Laravel\Horizon\Contracts\Silenced;

class ProcessPodcast implements ShouldQueue, Silenced
{
    use Queueable;

    // ...
}
```

<a name="balancing-strategies"></a>
## Strategie równoważenia

Każdy nadzorca może przetwarzać jedną lub więcej kolejek, ale w przeciwieństwie do domyślnego systemu kolejek Laravel, Horizon pozwala wybrać spośród trzech strategii równoważenia workerów: `auto`, `simple` i `false`.

<a name="auto-balancing"></a>
### Automatyczne równoważenie

Strategia `auto`, która jest strategią domyślną, dostosowuje liczbę procesów workerów na kolejkę na podstawie bieżącego obciążenia kolejki. Na przykład, jeśli Twoja kolejka `notifications` ma 1000 oczekujących zadań, a kolejka `default` jest pusta, Horizon przydzieli więcej workerów do kolejki `notifications`, aż kolejka zostanie opróżniona.

Uzywając strategii `auto`, możesz również skonfigurować opcje konfiguracyjne `minProcesses` i `maxProcesses`:

<div class="content-list" markdown="1">

- `minProcesses` definiuje minimalną liczbę procesów workerów na kolejkę. Ta wartość musi być większa lub równa 1.
- `maxProcesses` definiuje maksymalną całkowitą liczbę procesów workerów, do których Horizon może skalować się we wszystkich kolejkach. Ta wartość powinna zazwyczaj być większa niż liczba kolejek pomnożona przez wartość `minProcesses`. Aby uniemożliwić nadzórcy tworzenie jakichkolwiek procesów, możesz ustawić tę wartość na 0.

</div>

Na przykład, możesz skonfigurować Horizon, aby utrzymywał co najmniej jeden proces na kolejkę i skalował się do maksymalnie 10 procesów workerów:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            'connection' => 'redis',
            'queue' => ['default', 'notifications'],
            'balance' => 'auto',
            'autoScalingStrategy' => 'time',
            'minProcesses' => 1,
            'maxProcesses' => 10,
            'balanceMaxShift' => 1,
            'balanceCooldown' => 3,
        ],
    ],
],
```

Opcja konfiguracyjna `autoScalingStrategy` określa, jak Horizon będzie przydzielać więcej procesów workerów do kolejek. Możesz wybrać między dwiema strategiami:

<div class="content-list" markdown="1">

- Strategia `time` będzie przydzielać workery na podstawie całkowitego szacowanego czasu, jaki zajmie wyczyszczenie kolejki.
- Strategia `size` będzie przydzielać workery na podstawie całkowitej liczby zadań w kolejce.

</div>

Wartości konfiguracyjne `balanceMaxShift` i `balanceCooldown` określają, jak szybko Horizon będzie skalować się, aby spełnić zapotrzebowanie na workery. W powyższym przykładzie, maksymalnie jeden nowy proces zostanie utworzony lub zniszczony co trzy sekundy. Możesz dowolnie dostosować te wartości w zależności od potrzeb Twojej aplikacji.

<a name="auto-queue-priorities"></a>
#### Priorytety kolejek i automatyczne równoważenie

Podczas używania strategii równoważenia `auto`, Horizon nie wymusza ścisłego priorytetu między kolejkami. Kolejność kolejek w konfiguracji nadzorcy nie wpływa na to, jak procesy workerów są przydzielane. Zamiast tego Horizon polega na wybranej strategii `autoScalingStrategy`, aby dynamicznie przydzielać procesy workerów na podstawie obciążenia kolejki.

Na przykład, w następującej konfiguracji, kolejka high nie jest priorytetowa względem kolejki default, mimo że pojawia się pierwsza na liście:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['high', 'default'],
            'minProcesses' => 1,
            'maxProcesses' => 10,
        ],
    ],
],
```

Jeśli chcesz wymusić względny priorytet między kolejkami, możesz zdefiniować wielu nadzorców i jawnie przydzielić zasoby przetwarzania:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default'],
            'minProcesses' => 1,
            'maxProcesses' => 10,
        ],
        'supervisor-2' => [
            // ...
            'queue' => ['images'],
            'minProcesses' => 1,
            'maxProcesses' => 1,
        ],
    ],
],
```

W tym przykładzie, kolejka `default` może skalować się do 10 procesów, podczas gdy kolejka `images` jest ograniczona do jednego procesu. Ta konfiguracja zapewnia, że Twoje kolejki mogą skalować się niezależnie.

> [!NOTE]
> Podczas wysyłania zadań wymagających dużych zasobów, czasami najlepiej przydzielić je do dedykowanej kolejki z ograniczoną wartością `maxProcesses`. W przeciwnym razie te zadania mogłyby zużyć nadmierne zasoby CPU i przeciążyć system.

<a name="simple-balancing"></a>
### Proste równoważenie

Strategia `simple` rozdziela procesy workerów równomiernie między określone kolejki. Przy tej strategii Horizon nie skaluje automatycznie liczby procesów workerów. Zamiast tego używa stałej liczby procesów:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default', 'notifications'],
            'balance' => 'simple',
            'processes' => 10,
        ],
    ],
],
```

W powyższym przykładzie Horizon przydzieli 5 procesów do każdej kolejki, dzieląc całość 10 równo.

Jeśli chcesz kontrolować liczbę procesów workerów przydzielonych do każdej kolejki indywidualnie, możesz zdefiniować wielu nadzorców:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default'],
            'balance' => 'simple',
            'processes' => 10,
        ],
        'supervisor-notifications' => [
            // ...
            'queue' => ['notifications'],
            'balance' => 'simple',
            'processes' => 2,
        ],
    ],
],
```

W tej konfiguracji Horizon przydzieli 10 procesów do kolejki `default` i 2 procesy do kolejki `notifications`.

<a name="no-balancing"></a>
### Bez równoważenia

Kiedy opcja `balance` jest ustawiona na `false`, Horizon przetwarza kolejki ściśle w kolejności, w jakiej są wymienione, podobnie jak domyślny system kolejek Laravel. Jednak nadal będzie skalować liczbę procesów workerów, jeśli zadania zaczną się gromadzić:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default', 'notifications'],
            'balance' => false,
            'minProcesses' => 1,
            'maxProcesses' => 10,
        ],
    ],
],
```

W powyższym przykładzie zadania w kolejce `default` są zawsze priorytetowe względem zadań w kolejce `notifications`. Na przykład, jeśli jest 1000 zadań w `default` i tylko 10 w `notifications`, Horizon w pełni przetworzy wszystkie zadania `default` przed obsłużeniem jakichkolwiek z `notifications`.

Możesz kontrolować możliwość skalowania procesów workerów przez Horizon, używając opcji `minProcesses` i `maxProcesses`:

<div class="content-list" markdown="1">

- `minProcesses` definiuje minimalną liczbę procesów workerów łącznie. Ta wartość musi być większa lub równa 1.
- `maxProcesses` definiuje maksymalną całkowitą liczbę procesów workerów, do których Horizon może się skalować.

</div>

<a name="upgrading-horizon"></a>
## Aktualizacja Horizon

Podczas aktualizacji do nowej głównej wersji Horizon, ważne jest, aby dokładnie przejrzeć [przewodnik po aktualizacji](https://github.com/laravel/horizon/blob/master/UPGRADE.md).

<a name="running-horizon"></a>
## Uruchamianie Horizon

Po skonfigurowaniu nadzorców i workerów w pliku konfiguracyjnym `config/horizon.php` Twojej aplikacji, możesz uruchomić Horizon za pomocą polecenia Artisan `horizon`. To pojedyncze polecenie uruchomi wszystkie skonfigurowane procesy workerów dla bieżącego środowiska:

```shell
php artisan horizon
```

Możesz wstrzymać proces Horizon i polecić mu kontynuować przetwarzanie zadań, używając poleceń Artisan `horizon:pause` i `horizon:continue`:

```shell
php artisan horizon:pause

php artisan horizon:continue
```

Możesz również wstrzymać i wznowić konkretnych [nadzorców](#supervisors) Horizon, używając poleceń Artisan `horizon:pause-supervisor` i `horizon:continue-supervisor`:

```shell
php artisan horizon:pause-supervisor supervisor-1

php artisan horizon:continue-supervisor supervisor-1
```

Możesz sprawdzić bieżący status procesu Horizon, używając polecenia Artisan `horizon:status`:

```shell
php artisan horizon:status
```

Możesz sprawdzić bieżący status konkretnego [nadzorcy](#supervisors) Horizon, używając polecenia Artisan `horizon:supervisor-status`:

```shell
php artisan horizon:supervisor-status supervisor-1
```

Możesz łagodnie zakończyć proces Horizon, używając polecenia Artisan `horizon:terminate`. Wszelkie zadania, które są aktualnie przetwarzane, zostaną zakończone, a następnie Horizon przestanie się wykonywać:

```shell
php artisan horizon:terminate
```

<a name="automatically-restarting-horizon"></a>
#### Automatyczne restartowanie Horizon

Podczas lokalnego rozwijania aplikacji możesz uruchomić polecenie `horizon:listen`. Podczas używania polecenia `horizon:listen` nie musisz ręcznie restartować Horizon, gdy chcesz przeładować zaktualizowany kod. Przed użyciem tej funkcji powinieneś upewnić się, że [Node](https://nodejs.org) jest zainstalowany w Twoim lokalnym środowisku deweloperskim. Ponadto powinieneś zainstalować bibliotekę do obserwowania plików [Chokidar](https://github.com/paulmillr/chokidar) w swoim projekcie:

```shell
npm install --save-dev chokidar
```

Po zainstalowaniu Chokidar możesz uruchomić Horizon za pomocą polecenia `horizon:listen`:

```shell
php artisan horizon:listen
```

Podczas uruchamiania w Dockerze lub Vagrant powinieneś użyć opcji `--poll`:

```shell
php artisan horizon:listen --poll
```

Możesz skonfigurować katalogi i pliki, które powinny być obserwowane, używając opcji konfiguracyjnej `watch` w pliku konfiguracyjnym `config/horizon.php` Twojej aplikacji:

```php
'watch' => [
    'app',
    'bootstrap',
    'config',
    'database',
    'public/**/*.php',
    'resources/**/*.php',
    'routes',
    'composer.lock',
    '.env',
],
```

<a name="deploying-horizon"></a>
### Wdrażanie Horizon

Kiedy jesteś gotowy wdrożyć Horizon na rzeczywistym serwerze aplikacji, powinieneś skonfigurować monitor procesów do monitorowania polecenia `php artisan horizon` i restartowania go, jeśli nieoczekiwanie się zakończy. Nie martw się, poniżej omówimy, jak zainstalować monitor procesów.

Podczas procesu wdrażania aplikacji powinieneś polecić procesowi Horizon zakończenie działania, aby został on ponownie uruchomiony przez monitor procesów i otrzymał zmiany w kodzie:

```shell
php artisan horizon:terminate
```

<a name="installing-supervisor"></a>
#### Instalacja Supervisora

Supervisor to monitor procesów dla systemu operacyjnego Linux i automatycznie restartuje proces `horizon`, jeśli przestanie się wykonywać. Aby zainstalować Supervisora na Ubuntu, możesz użyć następującego polecenia. Jeśli nie używasz Ubuntu, możesz prawdopodobnie zainstalować Supervisora używając menedżera pakietów Twojego systemu operacyjnego:

```shell
sudo apt-get install supervisor
```

> [!NOTE]
> Jeśli samodzielne konfigurowanie Supervisora wydaje się przytłaczające, rozważ użycie [Laravel Cloud](https://cloud.laravel.com), które może zarządzać procesami w tle dla Twoich aplikacji Laravel.

<a name="supervisor-configuration"></a>
#### Konfiguracja Supervisora

Pliki konfiguracyjne Supervisora są zazwyczaj przechowywane w katalogu `/etc/supervisor/conf.d` Twojego serwera. W tym katalogu możesz utworzyć dowolną liczbę plików konfiguracyjnych, które instruują supervisora, jak monitorować Twoje procesy. Na przykład, stwórzmy plik `horizon.conf`, który uruchamia i monitoruje proces `horizon`:

```ini
[program:horizon]
process_name=%(program_name)s
command=php /home/forge/example.com/artisan horizon
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/home/forge/example.com/horizon.log
stopwaitsecs=3600
```

Definiując konfigurację Supervisora, powinieneś upewnić się, że wartość `stopwaitsecs` jest większa niż liczba sekund zużywanych przez najdłużej wykonujące się zadanie. W przeciwnym razie Supervisor może zabić zadanie, zanim skończy przetwarzanie.

> [!WARNING]
> Choć powyższe przykłady są ważne dla serwerów opartych na Ubuntu, lokalizacja i rozszerzenie pliku oczekiwane przez pliki konfiguracyjne Supervisora mogą się różnić w zależności od innych systemów operacyjnych serwerów. Skonsultuj się z dokumentacją swojego serwera, aby uzyskać więcej informacji.

<a name="starting-supervisor"></a>
#### Uruchamianie Supervisora

Po utworzeniu pliku konfiguracyjnego możesz zaktualizować konfigurację Supervisora i uruchomić monitorowane procesy za pomocą następujących poleceń:

```shell
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start horizon
```

> [!NOTE]
> Więcej informacji o uruchamianiu Supervisora znajdziesz w [dokumentacji Supervisora](http://supervisord.org/index.html).

<a name="tags"></a>
## Tagi

Horizon pozwala przypisywać "tagi" do zadań, w tym do mailables, zdarzeń broadcast, powiadomień i nasłuchiwaczy zdarzeń w kolejce. W rzeczywistości Horizon inteligentnie i automatycznie oznaczy większość zadań w zależności od modeli Eloquent, które są dołączone do zadania. Na przykład, spójrz na następujące zadanie:

```php
<?php

namespace App\Jobs;

use App\Models\Video;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class RenderVideo implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Video $video,
    ) {}

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        // ...
    }
}
```

Jeśli to zadanie zostanie dodane do kolejki z instancją `App\Models\Video`, która ma atrybut `id` o wartości `1`, automatycznie otrzyma tag `App\Models\Video:1`. Dzieje się tak, ponieważ Horizon przeszukuje właściwości zadania w poszukiwaniu modeli Eloquent. Jeśli znalezione zostaną modele Eloquent, Horizon inteligentnie oznaczy zadanie, używając nazwy klasy modelu i klucza głównego:

```php
use App\Jobs\RenderVideo;
use App\Models\Video;

$video = Video::find(1);

RenderVideo::dispatch($video);
```

<a name="manually-tagging-jobs"></a>
#### Ręczne oznaczanie zadań

Jeśli chcesz ręcznie zdefiniować tagi dla jednego ze swoich obiektów kolejkowalnych, możesz zdefiniować metodę `tags` w klasie:

```php
class RenderVideo implements ShouldQueue
{
    /**
     * Get the tags that should be assigned to the job.
     *
     * @return array<int, string>
     */
    public function tags(): array
    {
        return ['render', 'video:'.$this->video->id];
    }
}
```

<a name="manually-tagging-event-listeners"></a>
#### Ręczne oznaczanie nasłuchiwaczy zdarzeń

Podczas pobierania tagów dla nasłuchiwacza zdarzeń w kolejce, Horizon automatycznie przekazuje instancję zdarzenia do metody `tags`, pozwalając dodać dane zdarzenia do tagów:

```php
class SendRenderNotifications implements ShouldQueue
{
    /**
     * Get the tags that should be assigned to the listener.
     *
     * @return array<int, string>
     */
    public function tags(VideoRendered $event): array
    {
        return ['video:'.$event->video->id];
    }
}
```

<a name="notifications"></a>
## Powiadomienia

> [!WARNING]
> Podczas konfigurowania Horizon do wysyłania powiadomień Slack lub SMS, powinieneś przejrzeć [wymagania wstępne dla odpowiedniego kanału powiadomień](/docs/{{version}}/notifications).

Jeśli chcesz otrzymywać powiadomienia, gdy jedna z Twoich kolejek ma długi czas oczekiwania, możesz użyć metod `Horizon::routeMailNotificationsTo`, `Horizon::routeSlackNotificationsTo` i `Horizon::routeSmsNotificationsTo`. Możesz wywołać te metody z metody `boot` dostawcy `App\Providers\HorizonServiceProvider` Twojej aplikacji:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    parent::boot();

    Horizon::routeSmsNotificationsTo('15556667777');
    Horizon::routeMailNotificationsTo('example@example.com');
    Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
}
```

<a name="configuring-notification-wait-time-thresholds"></a>
#### Konfiguracja progów czasu oczekiwania powiadomień

Możesz skonfigurować, ile sekund jest uwazażanych za "długie oczekiwanie" w pliku konfiguracyjnym `config/horizon.php` Twojej aplikacji. Opcja konfiguracyjna `waits` w tym pliku pozwala kontrolować próg długiego oczekiwania dla każdej kombinacji połączenia / kolejki. Wszystkie niezdefiniowane kombinacje połączenia / kolejki będą domyślnie miały próg długiego oczekiwania wynoszący 60 sekund:

```php
'waits' => [
    'redis:critical' => 30,
    'redis:default' => 60,
    'redis:batch' => 120,
],
```

Ustawienie progu kolejki na `0` wyłączy powiadomienia o długim oczekiwaniu dla tej kolejki.

<a name="metrics"></a>
## Metryki

Horizon zawiera panel metryk, który dostarcza informacji o czasach oczekiwania i przepustowości Twoich zadań i kolejek. Aby wypełnić ten panel, powinieneś skonfigurować polecenie Artisan `snapshot` Horizon tak, aby uruchamiało się co pięć minut w pliku `routes/console.php` Twojej aplikacji:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('horizon:snapshot')->everyFiveMinutes();
```

Jeśli chcesz usunąć wszystkie dane metryk, możesz wywołać polecenie Artisan `horizon:clear-metrics`:

```shell
php artisan horizon:clear-metrics
```

<a name="deleting-failed-jobs"></a>
## Usuwanie nieudanych zadań

Jeśli chcesz usunąć nieudane zadanie, możesz użyć polecenia `horizon:forget`. Polecenie `horizon:forget` akceptuje ID lub UUID nieudanego zadania jako jedyny argument:

```shell
php artisan horizon:forget 5
```

Jeśli chcesz usunąć wszystkie nieudane zadania, możesz przekazać opcję `--all` do polecenia `horizon:forget`:

```shell
php artisan horizon:forget --all
```

<a name="clearing-jobs-from-queues"></a>
## Czyszczenie zadań z kolejek

Jeśli chcesz usunąć wszystkie zadania z domyślnej kolejki aplikacji, możesz to zrobić za pomocą polecenia Artisan `horizon:clear`:

```shell
php artisan horizon:clear
```

Możesz podać opcję `queue`, aby usunąć zadania z konkretnej kolejki:

```shell
php artisan horizon:clear --queue=emails
```
