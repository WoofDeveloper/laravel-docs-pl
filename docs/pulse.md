# Laravel Pulse

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
    - [Konfiguracja](#configuration)
- [Panel](#dashboard)
    - [Autoryzacja](#dashboard-authorization)
    - [Dostosowywanie](#dashboard-customization)
    - [Rozwiązywanie Użytkowników](#dashboard-resolving-users)
    - [Karty](#dashboard-cards)
- [Przechwytywanie Wpisów](#capturing-entries)
    - [Rejestratory](#recorders)
    - [Filtrowanie](#filtering)
- [Wydajność](#performance)
    - [Używanie Innej Bazy Danych](#using-a-different-database)
    - [Przetwarzanie Redis](#ingest)
    - [Próbkowanie](#sampling)
    - [Przycinanie](#trimming)
    - [Obsługa Wyjątków Pulse](#pulse-exceptions)
- [Niestandardowe Karty](#custom-cards)
    - [Komponenty Kart](#custom-card-components)
    - [Stylowanie](#custom-card-styling)
    - [Przechwytywanie i Agregacja Danych](#custom-card-data)

<a name="introduction"></a>
## Wprowadzenie

[Laravel Pulse](https://github.com/laravel/pulse) dostarcza błyskawicznych wglądów w wydajność i użycie twojej aplikacji. Za pomocą Pulse możesz wytropić wąskie gardła, takie jak wolne zadania i punkty końcowe, znaleźć najbardziej aktywnych użytkowników i więcej.

Do szczegółowego debugowania poszczególnych zdarzeń, sprawdź [Laravel Telescope](/docs/{{version}}/telescope).

<a name="installation"></a>
## Instalacja

> [!WARNING]
> Pierwszorzędna implementacja przechowywania danych Pulse wymaga obecnie bazy danych MySQL, MariaDB lub PostgreSQL. Jeśli używasz innego silnika bazy danych, będziesz potrzebować osobnej bazy danych MySQL, MariaDB lub PostgreSQL dla danych Pulse.

Możesz zainstalować Pulse używając menedżera pakietów Composer:

```shell
composer require laravel/pulse
```

Następnie powinieneś opublikować pliki konfiguracyjne i migracyjne Pulse za pomocą polecenia Artisan `vendor:publish`:

```shell
php artisan vendor:publish --provider="Laravel\Pulse\PulseServiceProvider"
```

Na koniec powinieneś uruchomić polecenie `migrate`, aby utworzyć tabele potrzebne do przechowywania danych Pulse:

```shell
php artisan migrate
```

Po uruchomieniu migracji bazy danych Pulse możesz uzyskać dostęp do panelu Pulse poprzez trasę `/pulse`.

> [!NOTE]
> Jeśli nie chcesz przechowywać danych Pulse w głównej bazie danych aplikacji, możesz [określić dedykowane połączenie z bazą danych](#using-a-different-database).

<a name="configuration"></a>
### Konfiguracja

Wiele opcji konfiguracyjnych Pulse można kontrolować za pomocą zmiennych środowiskowych. Aby zobaczyć dostępne opcje, zarejestrować nowe rejestratory lub skonfigurować zaawansowane opcje, możesz opublikować plik konfiguracyjny `config/pulse.php`:

```shell
php artisan vendor:publish --tag=pulse-config
```

<a name="dashboard"></a>
## Panel

<a name="dashboard-authorization"></a>
### Autoryzacja

Panel Pulse może być dostępny poprzez trasę `/pulse`. Domyślnie będziesz mógł uzyskać dostęp do tego panelu tylko w środowisku `local`, więc będziesz musiał skonfigurować autoryzację dla środowisk produkcyjnych, dostosowując bramę autoryzacji `'viewPulse'`. Możesz to zrobić w pliku `app/Providers/AppServiceProvider.php` swojej aplikacji:

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Gate::define('viewPulse', function (User $user) {
        return $user->isAdmin();
    });

    // ...
}
```

<a name="dashboard-customization"></a>
### Dostosowywanie

Karty i układ panelu Pulse można skonfigurować, publikując widok panelu. Widok panelu zostanie opublikowany w `resources/views/vendor/pulse/dashboard.blade.php`:

```shell
php artisan vendor:publish --tag=pulse-dashboard
```

Panel jest zasilany przez [Livewire](https://livewire.laravel.com/) i pozwala na dostosowanie kart i układu bez konieczności przebudowywania zasobów JavaScript.

W tym pliku komponent `<x-pulse>` jest odpowiedzialny za renderowanie panelu i zapewnia układ siatki dla kart. Jeśli chcesz, aby panel obejmował pełną szerokość ekranu, możesz dodać właściwość `full-width` do komponentu:

```blade
<x-pulse full-width>
    ...
</x-pulse>
```

Domyślnie komponent `<x-pulse>` utworzy siatkę 12-kolumnową, ale możesz to dostosować za pomocą właściwości `cols`:

```blade
<x-pulse cols="16">
    ...
</x-pulse>
```

Każda karta akceptuje właściwości `cols` i `rows`, aby kontrolować przestrzeń i pozycjonowanie:

```blade
<livewire:pulse.usage cols="4" rows="2" />
```

Większość kart akceptuje również właściwość `expand`, aby wyświetlić pełną kartę zamiast przewijania:

```blade
<livewire:pulse.slow-queries expand />
```

<a name="dashboard-resolving-users"></a>
### Rozwiązywanie Użytkowników

Dla kart, które wyświetlają informacje o użytkownikach, takich jak karta Użycia Aplikacji, Pulse będzie rejestorwać tylko identyfikator użytkownika. Podczas renderowania panelu Pulse rozwiąże pola `name` i `email` z domyślnego modelu `Authenticatable` i wyświetli awatary za pomocą usługi internetowej Gravatar.

Możesz dostosować pola i awatar, wywołując metodę `Pulse::user` w klasie `App\Providers\AppServiceProvider` swojej aplikacji.

Metoda `user` akceptuje domknięcie, które otrzyma model `Authenticatable` do wyświetlenia i powinno zwrócić tablicę zawierającą informacje `name`, `extra` i `avatar` dla użytkownika:

```php
use Laravel\Pulse\Facades\Pulse;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Pulse::user(fn ($user) => [
        'name' => $user->name,
        'extra' => $user->email,
        'avatar' => $user->avatar_url,
    ]);

    // ...
}
```

> [!NOTE]
> Możesz całkowicie dostosować sposób przechwytywania i pobierania uwierzytelnionego użytkownika, implementując kontrakt `Laravel\Pulse\Contracts\ResolvesUsers` i wiążąc go w [kontenerze usług](/docs/{{version}}/container#binding-a-singleton) Laravel.

<a name="dashboard-cards"></a>
### Karty

<a name="servers-card"></a>
#### Serwery

Karta `<livewire:pulse.servers />` wyświetla zużycie zasobów systemowych dla wszystkich serwerów uruchamiających polecenie `pulse:check`. Zapoznaj się z dokumentacją dotyczącą [rejestratora serwerów](#servers-recorder), aby uzyskać więcej informacji na temat raportowania zasobów systemowych.

Jeśli wymienisz serwer w swojej infrastrukturze, możesz chcieć przestać wyświetlać nieaktywny serwer w panelu Pulse po określonym czasie. Możesz to zrobić, używając właściwości `ignore-after`, która akceptuje liczbę sekund, po których nieaktywne serwery powinny zostać usunięte z panelu Pulse. Alternatywnie możesz podać względny czas sformatowany jako ciąg, taki jak `1 hour` lub `3 days and 1 hour`:

```blade
<livewire:pulse.servers ignore-after="3 hours" />
```

<a name="application-usage-card"></a>
#### Użycie Aplikacji

Karta `<livewire:pulse.usage />` wyświetla 10 najlepszych użytkowników wykonujących żądania do aplikacji, wysyłających zadania i doświadczających wolnych żądań.

Jeśli chcesz wyświetlić wszystkie metryki użycia na ekranie w tym samym czasie, możesz dołączyć kartę wiele razy i określić atrybut `type`:

```blade
<livewire:pulse.usage type="requests" />
<livewire:pulse.usage type="slow_requests" />
<livewire:pulse.usage type="jobs" />
```

Aby dowiedzieć się, jak dostosować sposób, w jaki Pulse pobiera i wyświetla informacje o użytkownikach, zapoznaj się z naszą dokumentacją na temat [rozwiązywania użytkowników](#dashboard-resolving-users).

> [!NOTE]
> Jeśli twoja aplikacja otrzymuje wiele żądań lub wysyła wiele zadań, możesz chcieć włączyć [próbkowanie](#sampling). Zobacz dokumentację [rejestratora żądań użytkowników](#user-requests-recorder), [rejestratora zadań użytkowników](#user-jobs-recorder) i [rejestratora wolnych zadań](#slow-jobs-recorder), aby uzyskać więcej informacji.

<a name="exceptions-card"></a>
#### Wyjątki

Karta `<livewire:pulse.exceptions />` pokazuje częstotliwość i czas występowania wyjątków w aplikacji. Domyślnie wyjątki są grupowane na podstawie klasy wyjątku i lokalizacji, w której wystąpiły. Zobacz dokumentację [rejestratora wyjątków](#exceptions-recorder), aby uzyskać więcej informacji.

<a name="queues-card"></a>
#### Kolejki

Karta `<livewire:pulse.queues />` pokazuje przepustowość kolejek w aplikacji, w tym liczbę zadań w kolejce, przetwarzanych, przetworzonych, zwolnionych i nieudanych. Zobacz dokumentację [rejestratora kolejek](#queues-recorder), aby uzyskać więcej informacji.

<a name="slow-requests-card"></a>
#### Wolne Żądania

Karta `<livewire:pulse.slow-requests />` pokazuje przychodzące żądania do aplikacji, które przekraczają skonfigurowaną wartość progową, która domyślnie wynosi 1000 ms. Zobacz dokumentację [rejestratora wolnych żądań](#slow-requests-recorder), aby uzyskać więcej informacji.

<a name="slow-jobs-card"></a>
#### Wolne Zadania

Karta `<livewire:pulse.slow-jobs />` pokazuje zadania w kolejce w aplikacji, które przekraczają skonfigurowaną wartość progową, która domyślnie wynosi 1000 ms. Zobacz dokumentację [rejestratora wolnych zadań](#slow-jobs-recorder), aby uzyskać więcej informacji.

<a name="slow-queries-card"></a>
#### Wolne Zapytania

Karta `<livewire:pulse.slow-queries />` pokazuje zapytania do bazy danych w aplikacji, które przekraczają skonfigurowaną wartość progową, która domyślnie wynosi 1000 ms.

Domyślnie wolne zapytania są grupowane na podstawie zapytania SQL (bez powiązań) i lokalizacji, w której wystąpiło, ale możesz wybrać, aby nie przechwytywać lokalizacji, jeśli chcesz grupować tylko na podstawie zapytania SQL.

Jeśli napotkasz problemy z wydajnością renderowania z powodu niezwykle dużych zapytań SQL otrzymujących podświetlanie składni, możesz wyłączyć podświetlanie, dodając właściwość `without-highlighting`:

```blade
<livewire:pulse.slow-queries without-highlighting />
```

Zobacz dokumentację [rejestratora wolnych zapytań](#slow-queries-recorder), aby uzyskać więcej informacji.

<a name="slow-outgoing-requests-card"></a>
#### Wolne Żądania Wychodzące

Karta `<livewire:pulse.slow-outgoing-requests />` pokazuje żądania wychodzące wykonane za pomocą [klienta HTTP](/docs/{{version}}/http-client) Laravel, które przekraczają skonfigurowaną wartość progową, która domyślnie wynosi 1000 ms.

Domyślnie wpisy będą grupowane według pełnego adresu URL. Jednak możesz chcieć znormalizować lub pogrupować podobne żądania wychodzące za pomocą wyrażeń regularnych. Zobacz dokumentację [rejestratora wolnych żądań wychodzących](#slow-outgoing-requests-recorder), aby uzyskać więcej informacji.

<a name="cache-card"></a>
#### Pamięć Podręczna

Karta `<livewire:pulse.cache />` pokazuje statystyki trafień i chybień pamięci podręcznej dla aplikacji, zarówno globalnie, jak i dla poszczególnych kluczy.

Domyślnie wpisy będą grupowane według klucza. Jednak możesz chcieć znormalizować lub pogrupować podobne klucze za pomocą wyrażeń regularnych. Zobacz dokumentację [rejestratora interakcji z pamięcią podręczną](#cache-interactions-recorder), aby uzyskać więcej informacji.

<a name="capturing-entries"></a>
## Przechwytywanie Wpisów

Większość rejestratorów Pulse automatycznie przechwyci wpisy na podstawie zdarzeń frameworka wysyłanych przez Laravel. Jednak [rejestrator serwerów](#servers-recorder) i niektóre karty innych firm muszą regularnie odpytywać informacje. Aby użyć tych kart, musisz uruchomić demon `pulse:check` na wszystkich poszczególnych serwerach aplikacji:

```php
php artisan pulse:check
```

> [!NOTE]
> Aby proces `pulse:check` działał permanentnie w tle, powinieneś użyć monitora procesów, takiego jak Supervisor, aby upewnić się, że polecenie nie przestanie działać.

Ponieważ polecenie `pulse:check` jest procesem długotrwałym, nie zobaczy zmian w kodzie bez ponownego uruchomienia. Powinieneś łagodnie uruchomić ponownie polecenie, wywołując polecenie `pulse:restart` podczas procesu wdrażania aplikacji:

```shell
php artisan pulse:restart
```

> [!NOTE]
> Pulse używa [pamięci podręcznej](/docs/{{version}}/cache) do przechowywania sygnałów ponownego uruchomienia, więc powinieneś sprawdzić, czy sterownik pamięci podręcznej jest prawidłowo skonfigurowany dla aplikacji przed użyciem tej funkcji.

<a name="recorders"></a>
### Rejestratory

Rejestratory są odpowiedzialne za przechwytywanie wpisów z aplikacji, które mają być zarejestrowane w bazie danych Pulse. Rejestratory są rejestrowane i konfigurowane w sekcji `recorders` [pliku konfiguracyjnego](#configuration) Pulse.

<a name="cache-interactions-recorder"></a>
#### Interakcje z Pamięcią Podręczną

Rejestrator `CacheInteractions` przechwytuje informacje o trafieniach i chybieniach [pamięci podręcznej](/docs/{{version}}/cache) występujących w aplikacji, aby wyświetlić je na karcie [Pamięć Podręczna](#cache-card).

Możesz opcjonalnie dostosować [współczynnik próbkowania](#sampling) i ignorowane wzorce kluczy.

Możesz również skonfigurować grupowanie kluczy, tak aby podobne klucze były grupowane jako pojedynczy wpis. Na przykład możesz chcieć usunąć unikalne identyfikatory z kluczy buforujących ten sam typ informacji. Grupy są konfigurowane za pomocą wyrażenia regularnego do "znajdowania i zamiany" części klucza. Przykład jest zawarty w pliku konfiguracyjnym:

```php
Recorders\CacheInteractions::class => [
    // ...
    'groups' => [
        // '/:d+/' => ':*',
    ],
],
```

Używany będzie pierwszy pasujący wzorzec. Jeśli żaden wzorzec nie pasuje, klucz zostanie przechwycony w takiej postaci, w jakiej jest.

<a name="exceptions-recorder"></a>
#### Wyjątki

Rejestrator `Exceptions` przechwytuje informacje o raportowanych wyjątkach występujących w aplikacji, aby wyświetlić je na karcie [Wyjątki](#exceptions-card).

Możesz opcjonalnie dostosować [współczynnik próbkowania](#sampling) i ignorowane wzorce wyjątków. Możesz również skonfigurować, czy przechwytywać lokalizację, z której pochodził wyjątek. Przechwycona lokalizacja zostanie wyświetlona w panelu Pulse, co może pomóc w śledzeniu pochodzenia wyjątku; jednak jeśli ten sam wyjątek wystąpi w wielu lokalizacjach, pojawi się wiele razy dla każdej unikalnej lokalizacji.

<a name="queues-recorder"></a>
#### Kolejki

Rejestrator `Queues` przechwytuje informacje o kolejkach aplikacji, aby wyświetlić je na karcie [Kolejki](#queues-card).

Możesz opcjonalnie dostosować [współczynnik próbkowania](#sampling) i ignorowane wzorce zadań.

<a name="slow-jobs-recorder"></a>
#### Wolne Zadania

Rejestrator `SlowJobs` przechwytuje informacje o wolnych zadaniach występujących w aplikacji, aby wyświetlić je na karcie [Wolne Zadania](#slow-jobs-recorder).

Możesz opcjonalnie dostosować próg wolnych zadań, [współczynnik próbkowania](#sampling) i ignorowane wzorce zadań.

Możesz mieć niektóre zadania, które oczekujesz, że zajmą dłużej niż inne. W takich przypadkach możesz skonfigurować progi dla poszczególnych zadań:

```php
Recorders\SlowJobs::class => [
    // ...
    'threshold' => [
        '#^App\\Jobs\\GenerateYearlyReports$#' => 5000,
        'default' => env('PULSE_SLOW_JOBS_THRESHOLD', 1000),
    ],
],
```

Jeśli żaden wzór wyrażenia regularnego nie pasuje do nazwy klasy zadania, używana jest wartość `'default'`.

<a name="slow-outgoing-requests-recorder"></a>
#### Wolne Żądania Wychodzące

Rejestrator `SlowOutgoingRequests` przechwytuje informacje o wychodzących żądaniach HTTP wykonanych za pomocą [klienta HTTP](/docs/{{version}}/http-client) Laravel, które przekraczają skonfigurowaną wartość progową, aby wyświetlić je na karcie [Wolne Żądania Wychodzące](#slow-outgoing-requests-card).

Możesz opcjonalnie dostosować próg wolnych żądań wychodzących, [współczynnik próbkowania](#sampling) i ignorowane wzorce URL.

Możesz mieć niektóre żądania wychodzące, które oczekujesz, że zajmą dłużej niż inne. W takich przypadkach możesz skonfigurować progi dla poszczególnych żądań:

```php
Recorders\SlowOutgoingRequests::class => [
    // ...
    'threshold' => [
        '#backup.zip$#' => 5000,
        'default' => env('PULSE_SLOW_OUTGOING_REQUESTS_THRESHOLD', 1000),
    ],
],
```

Jeśli żaden wzór wyrażenia regularnego nie pasuje do adresu URL żądania, używana jest wartość `'default'`.

Możesz również skonfigurować grupowanie URL, aby podobne adresy URL były grupowane jako pojedynczy wpis. Na przykład możesz chcieć usunąć unikalne identyfikatory ze ścieżek URL lub grupować tylko według domeny. Grupy są konfigurowane za pomocą wyrażenia regularnego do "znajdowania i zamiany" części URL. Niektóre przykłady są zawarte w pliku konfiguracyjnym:

```php
Recorders\SlowOutgoingRequests::class => [
    // ...
    'groups' => [
        // '#^https://api\.github\.com/repos/.*$#' => 'api.github.com/repos/*',
        // '#^https?://([^/]*).*$#' => '\1',
        // '#/\d+#' => '/*',
    ],
],
```

Używany będzie pierwszy pasujący wzorzec. Jeśli żaden wzorzec nie pasuje, adres URL zostanie przechwycony w takiej postaci, w jakiej jest.

<a name="slow-queries-recorder"></a>
#### Wolne Zapytania

Rejestrator `SlowQueries` przechwytuje wszelkie zapytania do bazy danych w aplikacji, które przekraczają skonfigurowaną wartość progową, aby wyświetlić je na karcie [Wolne Zapytania](#slow-queries-card).

Możesz opcjonalnie dostosować próg wolnych zapytań, [współczynnik próbkowania](#sampling) i ignorowane wzorce zapytań. Możesz również skonfigurować, czy przechwytywać lokalizację zapytania. Przechwycona lokalizacja zostanie wyświetlona w panelu Pulse, co może pomóc w śledzeniu pochodzenia zapytania; jednak jeśli to samo zapytanie jest wykonywane w wielu lokalizacjach, pojawi się wiele razy dla każdej unikalnej lokalizacji.

Możesz mieć niektóre zapytania, które oczekujesz, że zajmą dłużej niż inne. W takich przypadkach możesz skonfigurować progi dla poszczególnych zapytań:

```php
Recorders\SlowQueries::class => [
    // ...
    'threshold' => [
        '#^insert into `yearly_reports`#' => 5000,
        'default' => env('PULSE_SLOW_QUERIES_THRESHOLD', 1000),
    ],
],
```

Jeśli żaden wzór wyrażenia regularnego nie pasuje do SQL zapytania, używana jest wartość `'default'`.

<a name="slow-requests-recorder"></a>
#### Wolne Żądania

Rejestrator `Requests` przechwytuje informacje o żądaniach wykonanych do aplikacji, aby wyświetlić je na kartach [Wolne Żądania](#slow-requests-card) i [Użycie Aplikacji](#application-usage-card).

Możesz opcjonalnie dostosować próg wolnej trasy, [współczynnik próbkowania](#sampling) i ignorowane ścieżki.

Możesz mieć niektóre żądania, które oczekujesz, że zajmą dłużej niż inne. W takich przypadkach możesz skonfigurować progi dla poszczególnych żądań:

```php
Recorders\SlowRequests::class => [
    // ...
    'threshold' => [
        '#^/admin/#' => 5000,
        'default' => env('PULSE_SLOW_REQUESTS_THRESHOLD', 1000),
    ],
],
```

Jeśli żaden wzór wyrażenia regularnego nie pasuje do adresu URL żądania, używana jest wartość `'default'`.

<a name="servers-recorder"></a>
#### Serwery

Rejestrator `Servers` przechwytuje zużycie procesora, pamięci i przechowywania danych serwerów zasilających aplikację, aby wyświetlić je na karcie [Serwery](#servers-card). Ten rejestrator wymaga uruchomienia [polecenia pulse:check](#capturing-entries) na każdym z serwerów, które chcesz monitorować.

Każdy raportujący serwer musi mieć unikalna nazwę. Domyślnie Pulse użyje wartości zwracanej przez funkcję PHP `gethostname`. Jeśli chcesz to dostosować, możesz ustawić zmienną środowiskową `PULSE_SERVER_NAME`:

```env
PULSE_SERVER_NAME=load-balancer
```

Plik konfiguracyjny Pulse pozwala również dostosować katalogi, które są monitorowane.

<a name="user-jobs-recorder"></a>
#### Zadania Użytkowników

Rejestrator `UserJobs` przechwytuje informacje o użytkownikach wysyłających zadania w aplikacji, aby wyświetlić je na karcie [Użycie Aplikacji](#application-usage-card).

Możesz opcjonalnie dostosować [współczynnik próbkowania](#sampling) i ignorowane wzorce zadań.

<a name="user-requests-recorder"></a>
#### Żądania Użytkowników

Rejestrator `UserRequests` przechwytuje informacje o użytkownikach wykonujących żądania do aplikacji, aby wyświetlić je na karcie [Użycie Aplikacji](#application-usage-card).

Możesz opcjonalnie dostosować [współczynnik próbkowania](#sampling) i ignorowane wzorce URL.

<a name="filtering"></a>
### Filtrowanie

Jak widzieliśmy, wiele [rejestratorów](#recorders) oferuje możliwość "ignorowania" przychodzących wpisów w oparciu o ich wartość, taką jak adres URL żądania, poprzez konfigurację. Jednak czasami może być przydatne odfiltrowanie rekordów na podstawie innych czynników, takich jak aktualnie uwierzytelniony użytkownik. Aby odfiltrować te rekordy, możesz przekazać domknięcie do metody `filter` Pulse. Zazwyczaj metodę `filter` należy wywołać w metodzie `boot` `AppServiceProvider` aplikacji:

```php
use Illuminate\Support\Facades\Auth;
use Laravel\Pulse\Entry;
use Laravel\Pulse\Facades\Pulse;
use Laravel\Pulse\Value;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Pulse::filter(function (Entry|Value $entry) {
        return Auth::user()->isNotAdmin();
    });

    // ...
}
```

<a name="performance"></a>
## Wydajność

Pulse został zaprojektowany tak, aby mógł być dodany do istniejącej aplikacji bez konieczności dodatkowej infrastruktury. Jednak dla aplikacji o dużym ruchu istnieje kilka sposobów usunięcia wpływu, jaki Pulse może mieć na wydajność aplikacji.

<a name="using-a-different-database"></a>
### Używanie Innej Bazy Danych

Dla aplikacji o dużym ruchu możesz wolać użyć dedykowanego połączenia z bazą danych dla Pulse, aby uniknąć wpływu na bazę danych aplikacji.

Możesz dostosować [połączenie z bazą danych](/docs/{{version}}/database#configuration) używane przez Pulse, ustawiając zmienną środowiskową `PULSE_DB_CONNECTION`.

```env
PULSE_DB_CONNECTION=pulse
```

<a name="ingest"></a>
### Przetwarzanie Redis

> [!WARNING]
> Przetwarzanie Redis wymaga Redis 6.2 lub nowszego oraz `phpredis` lub `predis` jako skonfigurowanego sterownika klienta Redis aplikacji.

Domyślnie Pulse będzie przechowywać wpisy bezpośrednio do [skonfigurowanego połączenia z bazą danych](#using-a-different-database) po wysłaniu odpowiedzi HTTP do klienta lub przetworzeniu zadania; jednak możesz użyć sterownika przetwarzania Redis Pulse, aby zamiast tego wysyłać wpisy do strumienia Redis. Można to włączyć, konfigurując zmienną środowiskową `PULSE_INGEST_DRIVER`:

```ini
PULSE_INGEST_DRIVER=redis
```

Pulse będzie domyślnie używać domyślnego [połączenia Redis](/docs/{{version}}/redis#configuration), ale możesz to dostosować za pomocą zmiennej środowiskowej `PULSE_REDIS_CONNECTION`:

```ini
PULSE_REDIS_CONNECTION=pulse
```

> [!WARNING]
> Podczas korzystania ze sterownika przetwarzania Redis, instalacja Pulse powinna zawsze używać innego połączenia Redis niż kolejka zasilana przez Redis, jeśli dotyczy.

Podczas używania przetwarzania Redis będziesz musiał uruchomić polecenie `pulse:work`, aby monitorować strumień i przenoсиć wpisy z Redis do tabel bazy danych Pulse.

```php
php artisan pulse:work
```

> [!NOTE]
> Aby proces `pulse:work` działał permanentnie w tle, powinieneś użyć monitora procesów, takiego jak Supervisor, aby upewnić się, że worker Pulse nie przestanie działać.

As the `pulse:work` command is a long-lived process, it will not see changes to your codebase without being restarted. You should gracefully restart the command by calling the `pulse:restart` command during your application's deployment process:

```shell
php artisan pulse:restart
```

> [!NOTE]
> Pulse uses the [cache](/docs/{{version}}/cache) to store restart signals, so you should verify that a cache driver is properly configured for your application before using this feature.

<a name="sampling"></a>
### Próbkowanie

Domyślnie Pulse przechwyci każde istotne zdarzenie, które występuje w aplikacji. Dla aplikacji o dużym ruchu może to skutkować koniecznością agregacji milionów wierszy bazy danych w panelu, zwłaszcza dla dłuższych okresów czasu.

Możesz zamiast tego wybrać włączenie "próbkowania" dla niektórych rejestratorów danych Pulse. Na przykład ustawienie współczynnika próbkowania na `0.1` w rejestratorze [Żądania Użytkowników](#user-requests-recorder) oznacza, że rejestrujesz tylko około 10% żądań do aplikacji. W panelu wartości zostaną przeskalowane i poprzedzone prefiksem `~`, aby wskazać, że są przybliżeniem.

Ogólnie rzecz biorąc, im więcej wpisów masz dla określonej metryki, tym niższy możesz bezpiecznie ustawić współczynnik próbkowania bez utraty zbyt dużej dokładności.

<a name="trimming"></a>
### Przycinanie

Pulse automatycznie przytnie swoje przechowywane wpisy, gdy wykraczają poza okno panelu. Przycinanie następuje podczas przetwarzania danych za pomocą systemu loterii, który może być dostosowany w [pliku konfiguracyjnym](#configuration) Pulse.

<a name="pulse-exceptions"></a>
### Obsługa Wyjątków Pulse

Jeśli wystąpi wyjątek podczas przechwytywania danych Pulse, na przykład niemożność połączenia się z bazą danych przechowywania, Pulse ciągle zakończy się niepowodzeniem, aby uniknąć wpływu na aplikację.

Jeśli chcesz dostosować sposób obsługi tych wyjątków, możesz podać domknięcie do metody `handleExceptionsUsing`:

```php
use Laravel\Pulse\Facades\Pulse;
use Illuminate\Support\Facades\Log;

Pulse::handleExceptionsUsing(function ($e) {
    Log::debug('An exception happened in Pulse', [
        'message' => $e->getMessage(),
        'stack' => $e->getTraceAsString(),
    ]);
});
```

<a name="custom-cards"></a>
## Niestandardowe Karty

Pulse pozwala na tworzenie niestandardowych kart do wyświetlania danych istotnych dla specyficznych potrzeb aplikacji. Pulse używa [Livewire](https://livewire.laravel.com), więc możesz chcieć [zapoznać się z jego dokumentacją](https://livewire.laravel.com/docs) przed zbudowaniem pierwszej niestandardowej karty.

<a name="custom-card-components"></a>
### Komponenty Kart

Tworzenie niestandardowej karty w Laravel Pulse rozpoczyna się od rozszerzenia bazowego komponentu Livewire `Card` i zdefiniowania odpowiadającego widoku:

```php
namespace App\Livewire\Pulse;

use Laravel\Pulse\Livewire\Card;
use Livewire\Attributes\Lazy;

#[Lazy]
class TopSellers extends Card
{
    public function render()
    {
        return view('livewire.pulse.top-sellers');
    }
}
```

Podczas używania funkcji [leniwego ładowania](https://livewire.laravel.com/docs/lazy) Livewire, komponent `Card` automatycznie dostarczy symbol zastępczy, który respektuje atrybuty `cols` i `rows` przekazane do komponentu.

Podczas pisania odpowiadającego widoku karty Pulse możesz wykorzystać komponenty Blade Pulse, aby uzyskać spójny wygląd i styl:

```blade
<x-pulse::card :cols="$cols" :rows="$rows" :class="$class" wire:poll.5s="">
    <x-pulse::card-header name="Top Sellers">
        <x-slot:icon>
            ...
        </x-slot:icon>
    </x-pulse::card-header>

    <x-pulse::scroll :expand="$expand">
        ...
    </x-pulse::scroll>
</x-pulse::card>
```

Zmienne `$cols`, `$rows`, `$class` i `$expand` powinny zostać przekazane do odpowiednich komponentów Blade, aby układ karty mógł być dostosowany z widoku panelu. Możesz również chcieć dołączyć atrybut `wire:poll.5s=""` w swoim widoku, aby karta automatycznie się aktualizowała.

Po zdefiniowaniu komponentu Livewire i szablonu, karta może zostać dołączona do [widoku panelu](#dashboard-customization):

```blade
<x-pulse>
    ...

    <livewire:pulse.top-sellers cols="4" />
</x-pulse>
```

> [!NOTE]
> Jeśli karta jest zawarta w pakiecie, będziesz musiał zarejestrować komponent w Livewire, używając metody `Livewire::component`.

<a name="custom-card-styling"></a>
### Stylowanie

Jeśli karta wymaga dodatkowego stylowania poza klasami i komponentami dołączonymi do Pulse, istnieje kilka opcji dołączenia niestandardowego CSS dla kart.

<a name="custom-card-styling-vite"></a>
#### Integracja z Laravel Vite

Jeśli niestandardowa karta znajduje się w kodzie aplikacji i używasz [integracji Vite](/docs/{{version}}/vite) Laravel, możesz zaktualizować plik `vite.config.js`, aby dołączyć dedykowany punkt wejścia CSS dla karty:

```js
laravel({
    input: [
        'resources/css/pulse/top-sellers.css',
        // ...
    ],
}),
```

Następnie możesz użyć dyrektywy Blade `@vite` w [widoku panelu](#dashboard-customization), określając punkt wejścia CSS dla karty:

```blade
<x-pulse>
    @vite('resources/css/pulse/top-sellers.css')

    ...
</x-pulse>
```

<a name="custom-card-styling-css"></a>
#### Pliki CSS

W przypadku innych przypadków użycia, w tym kart Pulse zawartych w pakiecie, możesz polecić Pulse załadowanie dodatkowych arkuszy stylów, definiując metodę `css` w komponencie Livewire, która zwraca ścieżkę do pliku CSS:

```php
class TopSellers extends Card
{
    // ...

    protected function css()
    {
        return __DIR__.'/../../dist/top-sellers.css';
    }
}
```

Gdy ta karta jest dołączona do panelu, Pulse automatycznie dołączy zawartość tego pliku w tagu `<style>`, więc nie musi być publikowana w katalogu `public`.

<a name="custom-card-styling-tailwind"></a>
#### Tailwind CSS

Podczas używania Tailwind CSS powinieneś utworzyć dedykowany punkt wejścia CSS. Poniższy przykład wyklucza bazowe style [Preflight](https://tailwindcss.com/docs/preflight) Tailwind, które są już dołączone przez Pulse, i zakresuje Tailwind za pomocą selektora CSS, aby uniknąć konfliktów z klasami Tailwind Pulse:

```css
@import "tailwindcss/theme.css";

@custom-variant dark (&:where(.dark, .dark *));
@source "./../../views/livewire/pulse/top-sellers.blade.php";

@theme {
  /* ... */
}

#top-sellers {
  @import "tailwindcss/utilities.css" source(none);
}
```

Będziesz również musiał dołączyć atrybut `id` lub `class` w widoku karty, który pasuje do selektora CSS w punkcie wejścia:

```blade
<x-pulse::card id="top-sellers" :cols="$cols" :rows="$rows" class="$class">
    ...
</x-pulse::card>
```

<a name="custom-card-data"></a>
### Przechwytywanie i Agregacja Danych

Niestandardowe karty mogą pobierać i wyświetlać dane z dowolnego miejsca; jednak możesz chcieć wykorzystać potężny i wydajny system rejestrowania i agregacji danych Pulse.

<a name="custom-card-data-capture"></a>
#### Przechwytywanie Wpisów

Pulse pozwala rejestrować "wpisy" za pomocą metody `Pulse::record`:

```php
use Laravel\Pulse\Facades\Pulse;

Pulse::record('user_sale', $user->id, $sale->amount)
    ->sum()
    ->count();
```

Pierwszy argument przekazany do metody `record` to `type` dla wpisu, który rejestrujesz, podczas gdy drugi argument to `key`, który określa, jak zagregowane dane powinny być grupowane. Dla większości metod agregacji będziesz również musiał określić `value` do zagregowania. W powyższym przykładzie agregowana wartość to `$sale->amount`. Następnie możesz wywołać jedną lub więcej metod agregacji (takich jak `sum`), aby Pulse mógł przechwycić wstępnie zagregowane wartości do "kubków" w celu wydajnego pobierania później.

Dostępne metody agregacji to:

* `avg`
* `count`
* `max`
* `min`
* `sum`

> [!NOTE]
> Podczas tworzenia pakietu karty, który przechwytuje aktualnie uwierzytelniony identyfikator użytkownika, powinieneś użyć metody `Pulse::resolveAuthenticatedUserId()`, która respektuje wszelkie [dostosowania resolvera użytkownika](#dashboard-resolving-users) dokonane w aplikacji.

<a name="custom-card-data-retrieval"></a>
#### Pobieranie Zagregowanych Danych

Podczas rozszerzania komponentu Livewire `Card` Pulse możesz użyć metody `aggregate`, aby pobrać zagregowane dane dla okresu przeglądanego w panelu:

```php
class TopSellers extends Card
{
    public function render()
    {
        return view('livewire.pulse.top-sellers', [
            'topSellers' => $this->aggregate('user_sale', ['sum', 'count'])
        ]);
    }
}
```

Metoda `aggregate` zwraca kolekcję obiektów PHP `stdClass`. Każdy obiekt będzie zawierać właściwość `key` przechwyconą wcześniej, wraz z kluczami dla każdego z żądanych agregatów:

```blade
@foreach ($topSellers as $seller)
    {{ $seller->key }}
    {{ $seller->sum }}
    {{ $seller->count }}
@endforeach
```

Pulse będzie głównie pobierać dane z wstępnie zagregowanych kubków; dlatego określone agregaty muszą zostać przechwycone z wyprzedzeniem za pomocą metody `Pulse::record`. Najstarszy kubek zazwyczaj będzie częściowo wykraczać poza okres, więc Pulse zagreguje najstarsze wpisy, aby wypełnić lukę i dać dokładną wartość dla całego okresu, bez konieczności agregowania całego okresu przy każdym żądaniu odpytywania.

Możesz również pobrać całkowitą wartość dla danego typu, używając metody `aggregateTotal`. Na przykład następująca metoda pobierze sumę wszystkich sprzedaży użytkowników zamiast grupowania ich według użytkownika.

```php
$total = $this->aggregateTotal('user_sale', 'sum');
```

<a name="custom-card-displaying-users"></a>
#### Wyświetlanie Użytkowników

Podczas pracy z agregatami, które rejestrują identyfikator użytkownika jako klucz, możesz rozwiązać klucze do rekordów użytkowników za pomocą metody `Pulse::resolveUsers`:

```php
$aggregates = $this->aggregate('user_sale', ['sum', 'count']);

$users = Pulse::resolveUsers($aggregates->pluck('key'));

return view('livewire.pulse.top-sellers', [
    'sellers' => $aggregates->map(fn ($aggregate) => (object) [
        'user' => $users->find($aggregate->key),
        'sum' => $aggregate->sum,
        'count' => $aggregate->count,
    ])
]);
```

Metoda `find` zwraca obiekt zawierający klucze `name`, `extra` i `avatar`, które możesz opcjonalnie przekazać bezpośrednio do komponentu Blade `<x-pulse::user-card>`:

```blade
<x-pulse::user-card :user="{{ $seller->user }}" :stats="{{ $seller->sum }}" />
```

<a name="custom-recorders"></a>
#### Niestandardowe Rejestratory

Autorzy pakietów mogą chcieć dostarczać klasy rejestratora, aby umożliwić użytkownikom konfigurację przechwytywania danych.

Rejestratory są rejestrowane w sekcji `recorders` pliku konfiguracyjnego `config/pulse.php` aplikacji:

```php
[
    // ...
    'recorders' => [
        Acme\Recorders\Deployments::class => [
            // ...
        ],

        // ...
    ],
]
```

Rejestratory mogą nasłuchiwać zdarzeń, określając właściwość `$listen`. Pulse automatycznie zarejestruje nasłuchiwacze i wywoła metodę `record` rejestratorów:

```php
<?php

namespace Acme\Recorders;

use Acme\Events\Deployment;
use Illuminate\Support\Facades\Config;
use Laravel\Pulse\Facades\Pulse;

class Deployments
{
    /**
     * The events to listen for.
     *
     * @var array<int, class-string>
     */
    public array $listen = [
        Deployment::class,
    ];

    /**
     * Record the deployment.
     */
    public function record(Deployment $event): void
    {
        $config = Config::get('pulse.recorders.'.static::class);

        Pulse::record(
            // ...
        );
    }
}
```
