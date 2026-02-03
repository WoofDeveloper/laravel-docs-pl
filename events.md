# Zdarzenia

- [Wprowadzenie](#introduction)
- [Generowanie zdarzeń i nasłuchiwaczy](#generating-events-and-listeners)
- [Rejestrowanie zdarzeń i nasłuchiwaczy](#registering-events-and-listeners)
    - [Wykrywanie zdarzeń](#event-discovery)
    - [Ręczne rejestrowanie zdarzeń](#manually-registering-events)
    - [Nasłuchiwacze domknięć](#closure-listeners)
- [Definiowanie zdarzeń](#defining-events)
- [Definiowanie nasłuchiwaczy](#defining-listeners)
- [Kolejkowane nasłuchiwacze zdarzeń](#queued-event-listeners)
    - [Ręczna interakcja z kolejką](#manually-interacting-with-the-queue)
    - [Kolejkowane nasłuchiwacze zdarzeń a transakcje bazodanowe](#queued-event-listeners-and-database-transactions)
    - [Middleware dla kolejkowanych nasłuchiwaczy](#queued-listener-middleware)
    - [Szyfrowane kolejkowane nasłuchiwacze](#encrypted-queued-listeners)
    - [Obsługa nieudanych zadań](#handling-failed-jobs)
- [Wysyłanie zdarzeń](#dispatching-events)
    - [Wysyłanie zdarzeń po transakcjach bazodanowych](#dispatching-events-after-database-transactions)
    - [Odkładanie zdarzeń](#deferring-events)
- [Subskrybenci zdarzeń](#event-subscribers)
    - [Pisanie subskrybentów zdarzeń](#writing-event-subscribers)
    - [Rejestrowanie subskrybentów zdarzeń](#registering-event-subscribers)
- [Testowanie](#testing)
    - [Symulowanie podzbioru zdarzeń](#faking-a-subset-of-events)
    - [Symulacje zdarzeń w zakresie](#scoped-event-fakes)

<a name="introduction"></a>
## Wprowadzenie

System zdarzeń Laravela zapewnia prostą implementację wzorca obserwatora, umożliwiając subskrybowanie i nasłuchiwanie różnych zdarzeń zachodzących w aplikacji. Klasy zdarzeń są zazwyczaj przechowywane w katalogu `app/Events`, podczas gdy ich nasłuchiwacze znajdują się w `app/Listeners`. Nie martw się, jeśli nie widzisz tych katalogów w swojej aplikacji, ponieważ zostaną utworzone automatycznie podczas generowania zdarzeń i nasłuchiwaczy za pomocą poleceń konsoli Artisan.

Zdarzenia służą jako świetny sposób rozdzielenia różnych aspektów aplikacji, ponieważ pojedyncze zdarzenie może mieć wielu nasłuchiwaczy, którzy nie są od siebie zależni. Na przykład, możesz chcieć wysłać powiadomienie Slack do użytkownika za każdym razem, gdy zamówienie zostanie wysłane. Zamiast łączyć kod przetwarzania zamówień z kodem powiadomień Slack, możesz wywołać zdarzenie `App\Events\OrderShipped`, które nasłuchiwacz może odebrać i użyć do wysłania powiadomienia Slack.

<a name="generating-events-and-listeners"></a>
## Generowanie zdarzeń i nasłuchiwaczy

Aby szybko wygenerować zdarzenia i nasłuchiwaczy, możesz użyć poleceń Artisan `make:event` i `make:listener`:

```shell
php artisan make:event PodcastProcessed

php artisan make:listener SendPodcastNotification --event=PodcastProcessed
```

Dla wygody, możesz również wywołać polecenia Artisan `make:event` i `make:listener` bez dodatkowych argumentów. Kiedy to zrobisz, Laravel automatycznie poprosi Cię o nazwę klasy, a podczas tworzenia nasłuchiwacza - o zdarzenie, którego ma nasłuchiwać:

```shell
php artisan make:event

php artisan make:listener
```

<a name="registering-events-and-listeners"></a>
## Rejestrowanie zdarzeń i nasłuchiwaczy

<a name="event-discovery"></a>
### Wykrywanie zdarzeń

Domyślnie Laravel automatycznie znajdzie i zarejestruje nasłuchiwaczy zdarzeń poprzez skanowanie katalogu `Listeners` aplikacji. Gdy Laravel znajdzie jakąkolwiek metodę klasy nasłuchiwacza, która zaczyna się od `handle` lub `__invoke`, Laravel zarejestruje te metody jako nasłuchiwaczy zdarzeń dla zdarzenia, które jest określone typem w sygnaturze metody:

```php
use App\Events\PodcastProcessed;

class SendPodcastNotification
{
    /**
     * Handle the event.
     */
    public function handle(PodcastProcessed $event): void
    {
        // ...
    }
}
```

Możesz nasłuchiwać wielu zdarzeń używając typów unii PHP:

```php
/**
 * Handle the event.
 */
public function handle(PodcastProcessed|PodcastPublished $event): void
{
    // ...
}
```

Jeśli planujesz przechowywać nasłuchiwaczy w innym katalogu lub w wielu katalogach, możesz poinstruować Laravela, aby skanował te katalogi używając metody `withEvents` w pliku `bootstrap/app.php` aplikacji:

```php
->withEvents(discover: [
    __DIR__.'/../app/Domain/Orders/Listeners',
])
```

Możesz skanować nasłuchiwaczy w wielu podobnych katalogach używając znaku `*` jako wieloznacznika:

```php
->withEvents(discover: [
    __DIR__.'/../app/Domain/*/Listeners',
])
])
```

Polecenie `event:list` może być użyte do wylistowania wszystkich nasłuchiwaczy zarejestrowanych w aplikacji:

```shell
php artisan event:list
```

<a name="event-discovery-in-production"></a>
#### Wykrywanie zdarzeń w produkcji

Aby przyspieszyć działanie aplikacji, powinieneś buforować manifest wszystkich nasłuchiwaczy aplikacji za pomocą poleceń Artisan `optimize` lub `event:cache`. Zazwyczaj to polecenie powinno być uruchamiane jako część [procesu wdrażania](/docs/{{version}}/deployment#optimization) aplikacji. Ten manifest będzie używany przez framework do przyspieszenia procesu rejestracji zdarzeń. Polecenie `event:clear` może być użyte do usunięcia bufora zdarzeń.

<a name="manually-registering-events"></a>
### Ręczne rejestrowanie zdarzeń

Za pomocą fasady `Event` możesz ręcznie zarejestrować zdarzenia i odpowiadające im nasłuchiwaczy w metodzie `boot` klasy `AppServiceProvider` aplikacji:

```php
use App\Domain\Orders\Events\PodcastProcessed;
use App\Domain\Orders\Listeners\SendPodcastNotification;
use Illuminate\Support\Facades\Event;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(
        PodcastProcessed::class,
        SendPodcastNotification::class,
    );
}
```

Polecenie `event:list` może być użyte do wylistowania wszystkich nasłuchiwaczy zarejestrowanych w aplikacji:

```shell
php artisan event:list
```

<a name="closure-listeners"></a>
### Nasłuchiwacze domknięć

Zazwyczaj nasłuchiwacze są definiowane jako klasy; jednakże możesz również ręcznie zarejestrować nasłuchiwaczy opartych na domknięciach w metodzie `boot` klasy `AppServiceProvider` aplikacji:

```php
use App\Events\PodcastProcessed;
use Illuminate\Support\Facades\Event;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(function (PodcastProcessed $event) {
        // ...
    });
}
```

<a name="queuable-anonymous-event-listeners"></a>
#### Kolejkowane anonimowe nasłuchiwacze zdarzeń

Podczas rejestrowania nasłuchiwaczy opartych na domknięciach, możesz opakować domknięcie nasłuchiwacza w funkcję `Illuminate\Events\queueable`, aby poinstruować Laravela, aby wykonał nasłuchiwacza za pomocą [kolejki](/docs/{{version}}/queues):

```php
use App\Events\PodcastProcessed;
use function Illuminate\Events\queueable;
use Illuminate\Support\Facades\Event;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(queueable(function (PodcastProcessed $event) {
        // ...
    }));
}
```

Podobnie jak w przypadku kolejkowanych zadań, możesz użyć metod `onConnection`, `onQueue` i `delay` do dostosowania wykonywania kolejkowanego nasłuchiwacza:

```php
Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->onConnection('redis')->onQueue('podcasts')->delay(now()->plus(seconds: 10)));
```

Jeśli chcesz obsługiwać awarie anonimowych kolejkowanych nasłuchiwaczy, możesz przekazać domknięcie do metody `catch` podczas definiowania nasłuchiwacza `queueable`. To domknięcie otrzyma instancję zdarzenia i instancję `Throwable`, która spowodowała awarię nasłuchiwacza:

```php
use App\Events\PodcastProcessed;
use function Illuminate\Events\queueable;
use Illuminate\Support\Facades\Event;
use Throwable;

Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->catch(function (PodcastProcessed $event, Throwable $e) {
    // Kolejkowany nasłuchiwacz zawiódł...
}));
```

<a name="wildcard-event-listeners"></a>
#### Nasłuchiwacze zdarzeń z wieloznacznkiem

Możesz również zarejestrować nasłuchiwaczy używając znaku `*` jako parametru wieloznacznego, pozwalającego przechwycić wiele zdarzeń na tym samym nasłuchiwaczu. Nasłuchiwacze wieloznacznikowe otrzymują nazwę zdarzenia jako pierwszy argument i całą tablicę danych zdarzenia jako drugi argument:

```php
Event::listen('event.*', function (string $eventName, array $data) {
    // ...
});
```

<a name="defining-events"></a>
## Definiowanie zdarzeń

Klasa zdarzenia to zasadniczo kontener danych, który przechowuje informacje związane ze zdarzeniem. Na przykład, załóżmy, że zdarzenie `App\Events\OrderShipped` otrzymuje obiekt [Eloquent ORM](/docs/{{version}}/eloquent):

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    /**
     * Create a new event instance.
     */
    public function __construct(
        public Order $order,
    ) {}
}
```

Jak widzisz, ta klasa zdarzenia nie zawiera żadnej logiki. Jest to kontener dla instancji `App\Models\Order`, która została zakupiona. Cecha `SerializesModels` używana przez zdarzenie będzie elegancko serializować wszystkie modele Eloquent, jeśli obiekt zdarzenia zostanie zserializowany przy użyciu funkcji `serialize` PHP, na przykład podczas wykorzystywania [kolejkowanych nasłuchiwaczy](#queued-event-listeners).

<a name="defining-listeners"></a>
## Definiowanie nasłuchiwaczy

Następnie przyjrzyjmy się nasłuchiwaczowi dla naszego przykładowego zdarzenia. Nasłuchiwacze zdarzeń otrzymują instancje zdarzeń w swojej metodzie `handle`. Polecenie Artisan `make:listener`, gdy jest wywoływane z opcją `--event`, automatycznie zaimportuje odpowiednią klasę zdarzenia i określi typ zdarzenia w metodzie `handle`. W metodzie `handle` możesz wykonać dowolne akcje niezbędne do odpowiedzi na zdarzenie:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    /**
     * Create the event listener.
     */
    public function __construct() {}

    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        // Dostęp do zamówienia używając $event->order...
    }
}
```

> [!NOTE]
> Twoje nasłuchiwacze zdarzeń mogą również określać w konstruktorach typy wszystkich zależności, których potrzebują. Wszyscy nasłuchiwacze zdarzeń są rozwiązywani przez [kontener usług](/docs/{{version}}/container) Laravela, więc zależności będą wstrzykiwane automatycznie.

<a name="stopping-the-propagation-of-an-event"></a>
#### Zatrzymywanie propagacji zdarzenia

Czasami możesz chcieć zatrzymać propagację zdarzenia do innych nasłuchiwaczy. Możesz to zrobić, zwracając `false` z metody `handle` nasłuchiwacza.

<a name="queued-event-listeners"></a>
## Kolejkowane nasłuchiwacze zdarzeń

Kolejkowanie nasłuchiwaczy może być korzystne, jeśli Twój nasłuchiwacz zamierza wykonać wolne zadanie, takie jak wysyłanie e-maila lub wykonywanie żądania HTTP. Przed użyciem kolejkowanych nasłuchiwaczy upewnij się, że [skonfigurowałeś kolejkę](/docs/{{version}}/queues) i uruchomiłeś worker kolejki na swoim serwerze lub lokalnym środowisku programistycznym.

Aby określić, że nasłuchiwacz powinien być kolejkowany, dodaj interfejs `ShouldQueue` do klasy nasłuchiwacza. Nasłuchiwacze wygenerowani przez polecenie Artisan `make:listener` mają już ten interfejs zaimportowany do bieżącej przestrzeni nazw, więc możesz go od razu użyć:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```

To wszystko! Teraz, gdy zdarzenie obsługiwane przez tego nasłuchiwacza zostanie wysłane, nasłuchiwacz zostanie automatycznie kolejkowany przez dyspozytor zdarzeń za pomocą [systemu kolejek](/docs/{{version}}/queues) Laravela. Jeśli żadne wyjątki nie zostaną zgłoszone podczas wykonywania nasłuchiwacza przez kolejkę, kolejkowane zadanie zostanie automatycznie usunięte po zakończeniu przetwarzania.

<a name="customizing-the-queue-connection-queue-name"></a>
#### Dostosowywanie połączenia kolejki, nazwy i opóźnienia

Jeśli chcesz dostosować połączenie kolejki, nazwę kolejki lub czas opóźnienia zdarzenia nasłuchiwacza, możesz zdefiniować właściwości `$connection`, `$queue` lub `$delay` w swojej klasie nasłuchiwacza:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    /**
     * Nazwa połączenia, do którego zadanie powinno zostać wysłane.
     *
     * @var string|null
     */
    public $connection = 'sqs';

    /**
     * Nazwa kolejki, do której zadanie powinno zostać wysłane.
     *
     * @var string|null
     */
    public $queue = 'listeners';

    /**
     * Czas (w sekundach) przed przetworzeniem zadania.
     *
     * @var int
     */
    public $delay = 60;
}
```

Jeśli chcesz zdefiniować połączenie kolejki, nazwę kolejki lub opóźnienie nasłuchiwacza w czasie wykonywania, możesz zdefiniować metody `viaConnection`, `viaQueue` lub `withDelay` w nasłuchiwaczu:

```php
/**
 * Pobierz nazwę połączenia kolejki nasłuchiwacza.
 */
public function viaConnection(): string
{
    return 'sqs';
}

/**
 * Pobierz nazwę kolejki nasłuchiwacza.
 */
public function viaQueue(): string
{
    return 'listeners';
}

/**
 * Pobierz liczbę sekund przed przetworzeniem zadania.
 */
public function withDelay(OrderShipped $event): int
{
    return $event->highPriority ? 0 : 60;
}
```

<a name="conditionally-queueing-listeners"></a>
#### Warunkowe kolejkowanie nasłuchiwaczy

Czasami może być konieczne określenie, czy nasłuchiwacz powinien być kolejkowany na podstawie niektórych danych, które są dostępne tylko w czasie wykonywania. Aby to osiągnąć, można dodać metodę `shouldQueue` do nasłuchiwacza w celu określenia, czy nasłuchiwacz powinien być kolejkowany. Jeśli metoda `shouldQueue` zwróci `false`, nasłuchiwacz nie zostanie kolejkowany:

```php
<?php

namespace App\Listeners;

use App\Events\OrderCreated;
use Illuminate\Contracts\Queue\ShouldQueue;

class RewardGiftCard implements ShouldQueue
{
    /**
     * Nagrodzenie klienta kartą podarunkową.
     */
    public function handle(OrderCreated $event): void
    {
        // ...
    }

    /**
     * Określ, czy nasłuchiwacz powinien być kolejkowany.
     */
    public function shouldQueue(OrderCreated $event): bool
    {
        return $event->order->subtotal >= 5000;
    }
}
```

<a name="manually-interacting-with-the-queue"></a>
### Ręczna interakcja z kolejką

Jeśli musisz ręcznie uzyskać dostęp do metod `delete` i `release` podstawowego zadania kolejki nasłuchiwacza, możesz to zrobić używając cechy `Illuminate\Queue\InteractsWithQueue`. Ta cecha jest domyślnie importowana w wygenerowanych nasłuchiwaczach i zapewnia dostęp do tych metod:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        if ($condition) {
            $this->release(30);
        }
    }
}
```

<a name="queued-event-listeners-and-database-transactions"></a>
### Kolejkowane nasłuchiwacze zdarzeń a transakcje bazodanowe

Gdy kolejkowane nasłuchiwacze są wysyłane w ramach transakcji bazodanowych, mogą być przetwarzane przez kolejkę przed zatwierdzeniem transakcji bazodanowej. Gdy to się stanie, wszelkie aktualizacje dokonane w modelach lub rekordach bazy danych podczas transakcji bazodanowej mogą jeszcze nie być odzwierciedlone w bazie danych. Dodatkowo, wszelkie modele lub rekordy bazy danych utworzone w ramach transakcji mogą nie istnieć w bazie danych. Jeśli Twój nasłuchiwacz zależy od tych modeli, nieoczekiwane błędy mogą wystąpić, gdy zadanie wysyłające kolejkowanego nasłuchiwacza jest przetwarzane.

Jeśli opcja konfiguracyjna `after_commit` połączenia kolejki jest ustawiona na `false`, możesz nadal wskazać, że dany kolejkowany nasłuchiwacz powinien być wysłany po zatwierdzeniu wszystkich otwartych transakcji bazodanowych, implementując interfejs `ShouldQueueAfterCommit` w klasie nasłuchiwacza:

```php
<?php

namespace App\Listeners;

use Illuminate\Contracts\Queue\ShouldQueueAfterCommit;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueueAfterCommit
{
    use InteractsWithQueue;
}
```

> [!NOTE]
> Aby dowiedzieć się więcej o obejściach tych problemów, zapoznaj się z dokumentacją dotyczącą [kolejkowanych zadań i transakcji bazodanowych](/docs/{{version}}/queues#jobs-and-database-transactions).

<a name="queued-listener-middleware"></a>
### Middleware dla kolejkowanych nasłuchiwaczy

Kolejkowane nasłuchiwacze mogą również wykorzystywać [middleware zadań](/docs/{{version}}/queues#job-middleware). Middleware zadań umożliwiają zawinięcie niestandardowej logiki wokół wykonywania kolejkowanych nasłuchiwaczy, redukując powtarzalny kod w samych nasłuchiwaczach. Po utworzeniu middleware zadań mogą być dołączone do nasłuchiwacza, zwracając je z metody `middleware` nasłuchiwacza:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use App\Jobs\Middleware\RateLimited;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    /**
     * Obsłuż zdarzenie.
     */
    public function handle(OrderShipped $event): void
    {
        // Przetwórz zdarzenie...
    }

    /**
     * Pobierz middleware, przez które nasłuchiwacz powinien przejść.
     *
     * @return array<int, object>
     */
    public function middleware(OrderShipped $event): array
    {
        return [new RateLimited];
    }
}
```

<a name="encrypted-queued-listeners"></a>
#### Szyfrowane kolejkowane nasłuchiwacze

Laravel umożliwia zapewnienie prywatności i integralności danych kolejkowanego nasłuchiwacza poprzez [szyfrowanie](/docs/{{version}}/encryption). Aby rozpocząć, po prostu dodaj interfejs `ShouldBeEncrypted` do klasy nasłuchiwacza. Po dodaniu tego interfejsu do klasy, Laravel automatycznie zaszyfruje Twojego nasłuchiwacza przed umieszczeniem go w kolejce:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue, ShouldBeEncrypted
{
    // ...
}
```

<a name="handling-failed-jobs"></a>
### Obsługa nieudanych zadań

Czasami Twoi kolejkowane nasłuchiwacze zdarzeń mogą zawieść. Jeśli kolejkowany nasłuchiwacz przekroczy maksymalną liczbę prób zdefiniowaną przez Twojego workera kolejki, metoda `failed` zostanie wywołana na Twoim nasłuchiwaczu. Metoda `failed` otrzymuje instancję zdarzenia i `Throwable`, który spowodował awarię:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Throwable;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        // ...
    }

    /**
     * Obsłuż awarię zadania.
     */
    public function failed(OrderShipped $event, Throwable $exception): void
    {
        // ...
    }
}
```

<a name="specifying-queued-listener-maximum-attempts"></a>
#### Określanie maksymalnej liczby prób kolejkowanego nasłuchiwacza

Jeśli jeden z Twoich kolejkowanych nasłuchiwaczy napotyka błąd, prawdopodobnie nie chcesz, aby próbował w nieskończoność. Dlatego Laravel zapewnia różne sposoby określenia, ile razy lub jak długo nasłuchiwacz może być próbowany.

Możesz zdefiniować właściwość lub metodę `tries` w swojej klasie nasłuchiwacza, aby określić, ile razy nasłuchiwacz może być próbowany, zanim zostanie uznany za nieudany:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Liczba prób ponowienia kolejkowanego nasłuchiwacza.
     *
     * @var int
     */
    public $tries = 5;
}
```

Jako alternatywę dla definiowania, ile razy nasłuchiwacz może być próbowany przed awarią, możesz zdefiniować czas, po którym nasłuchiwacz nie powinien być już próbowany. Pozwala to nasłuchiwaczowi być próbowanym dowolną liczbę razy w określonym przedziale czasowym. Aby zdefiniować czas, po którym nasłuchiwacz nie powinien być już próbowany, dodaj metodę `retryUntil` do swojej klasy nasłuchiwacza. Ta metoda powinna zwracać instancję `DateTime`:

```php
use DateTime;

/**
 * Określ czas, po którym nasłuchiwacz powinien przekroczyć limit czasu.
 */
public function retryUntil(): DateTime
{
    return now()->plus(minutes: 5);
}
```

Jeśli zarówno `retryUntil`, jak i `tries` są zdefiniowane, Laravel daje pierwszeństwo metodzie `retryUntil`.

<a name="specifying-queued-listener-backoff"></a>
#### Określanie wycofania kolejkowanego nasłuchiwacza

Jeśli chcesz skonfigurować, ile sekund Laravel powinien czekać przed ponowną próbą nasłuchiwacza, który napotkał wyjątek, możesz to zrobić, definiując właściwość `backoff` w swojej klasie nasłuchiwacza:

```php
/**
 * Liczba sekund oczekiwania przed ponowieniem próby kolejkowanego nasłuchiwacza.
 *
 * @var int
 */
public $backoff = 3;
```

Jeśli potrzebujesz bardziej złożonej logiki do określania czasu wycofania nasłuchiwacza, możesz zdefiniować metodę `backoff` w swojej klasie nasłuchiwacza:

```php
/**
 * Oblicz liczbę sekund oczekiwania przed ponowieniem próby kolejkowanego nasłuchiwacza.
 */
public function backoff(OrderShipped $event): int
{
    return 3;
}
```

Możesz łatwo skonfigurować "wykładnicze" wycofania, zwracając tablicę wartości wycofania z metody `backoff`. W tym przykładzie opóźnienie ponowienia będzie wynosić 1 sekundę dla pierwszej próby, 5 sekund dla drugiej próby, 10 sekund dla trzeciej próby i 10 sekund dla każdej kolejnej próby, jeśli pozostały więcej prób:

```php
/**
 * Oblicz liczbę sekund oczekiwania przed ponowieniem próby kolejkowanego nasłuchiwacza.
 *
 * @return list<int>
 */
public function backoff(OrderShipped $event): array
{
    return [1, 5, 10];
}
```

<a name="specifying-queued-listener-max-exceptions"></a>
#### Określanie maksymalnej liczby wyjątków kolejkowanego nasłuchiwacza

Czasami możesz chcieć określić, że kolejkowany nasłuchiwacz może być próbowany wielokrotnie, ale powinien zawieść, jeśli ponowne próby zostały wywołane przez daną liczbę nieobsłużonych wyjątków (w przeciwieństwie do bycia zwolnionym bezpośrednio przez metodę `release`). Aby to osiągnąć, możesz zdefiniować właściwość `maxExceptions` w swojej klasie nasłuchiwacza:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Liczba prób ponowienia kolejkowanego nasłuchiwacza.
     *
     * @var int
     */
    public $tries = 25;

    /**
     * Maksymalna liczba nieobsłużonych wyjątków przed porażką.
     *
     * @var int
     */
    public $maxExceptions = 3;

    /**
     * Obsłuż zdarzenie.
     */
    public function handle(OrderShipped $event): void
    {
        // Przetwórz zdarzenie...
    }
}
```

W tym przykładzie nasłuchiwacz będzie ponawiany do 25 razy. Jednak nasłuchiwacz zawiedzie, jeśli trzy nieobsłużone wyjątki zostaną zgłoszone przez nasłuchiwacza.

<a name="specifying-queued-listener-timeout"></a>
#### Określanie limitu czasu kolejkowanego nasłuchiwacza

Często wiesz mniej więcej, jak długo oczekujesz, że Twoi kolejkowani nasłuchiwacze będą działać. Z tego powodu Laravel pozwala określić wartość "timeout". Jeśli nasłuchiwacz przetwarza dłużej niż liczba sekund określona przez wartość timeout, worker przetwarzający nasłuchiwacza zakończy działanie z błędem. Możesz zdefiniować maksymalną liczbę sekund, przez którą nasłuchiwacz może działać, definiując właściwość `timeout` w swojej klasie nasłuchiwacza:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    /**
     * Liczba sekund, przez które nasłuchiwacz może działać przed przekroczeniem limitu czasu.
     *
     * @var int
     */
    public $timeout = 120;
}
```

Jeśli chcesz wskazać, że nasłuchiwacz powinien być oznaczony jako nieudany po przekroczeniu limitu czasu, możesz zdefiniować właściwość `failOnTimeout` w klasie nasłuchiwacza:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    /**
     * Wskaż, czy nasłuchiwacz powinien zostać oznaczony jako nieudany po przekroczeniu limitu czasu.
     *
     * @var bool
     */
    public $failOnTimeout = true;
}
```

<a name="dispatching-events"></a>
## Wysyłanie zdarzeń

Aby wysłać zdarzenie, możesz wywołać statyczną metodę `dispatch` na zdarzeniu. Ta metoda jest udostępniana w zdarzeniu przez cechę `Illuminate\Foundation\Events\Dispatchable`. Wszelkie argumenty przekazane do metody `dispatch` będą przekazane do konstruktora zdarzenia:

```php
<?php

namespace App\Http\Controllers;

use App\Events\OrderShipped;
use App\Models\Order;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class OrderShipmentController extends Controller
{
    /**
     * Wysłanie danego zamówienia.
     */
    public function store(Request $request): RedirectResponse
    {
        $order = Order::findOrFail($request->order_id);

        // Logika wysyłki zamówienia...

        OrderShipped::dispatch($order);

        return redirect('/orders');
    }
}
```

Jeśli chcesz warunkowo wysłać zdarzenie, możesz użyć metod `dispatchIf` i `dispatchUnless`:

```php
OrderShipped::dispatchIf($condition, $order);

OrderShipped::dispatchUnless($condition, $order);
```

> [!NOTE]
> Podczas testowania może być pomocne potwierdzenie, że pewne zdarzenia zostały wysłane bez faktycznego uruchamiania ich nasłuchiwaczy. [Wbudowane helpery testowe](#testing) Laravela ułatwiają to zadanie.

<a name="dispatching-events-after-database-transactions"></a>
### Wysyłanie zdarzeń po transakcjach bazodanowych

Czasami możesz chcieć poinstruować Laravela, aby wysłał zdarzenie dopiero po zatwierdzeniu aktywnej transakcji bazodanowej. Aby to zrobić, możesz zaimplementować interfejs `ShouldDispatchAfterCommit` w klasie zdarzenia.

Ten interfejs instruuje Laravela, aby nie wysyłał zdarzenia, dopóki bieżąca transakcja bazodanowa nie zostanie zatwierdzona. Jeśli transakcja zawiedzie, zdarzenie zostanie odrzucone. Jeśli nie ma transakcji bazodanowej w toku, gdy zdarzenie jest wysyłane, zdarzenie zostanie wysłane natychmiast:

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped implements ShouldDispatchAfterCommit
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    /**
     * Create a new event instance.
     */
    public function __construct(
        public Order $order,
    ) {}
}
```

<a name="deferring-events"></a>
### Odkładanie zdarzeń

Odroczone zdarzenia pozwalają opóźnić wysyłanie zdarzeń modelu i wykonanie nasłuchiwaczy zdarzeń do momentu zakończenia określonego bloku kodu. Jest to szczególnie przydatne, gdy musisz upewnić się, że wszystkie powiązane rekordy są tworzone przed uruchomieniem nasłuchiwaczy zdarzeń.

Aby odroczyć zdarzenia, przekaż domknięcie do metody `Event::defer()`:

```php
use App\Models\User;
use Illuminate\Support\Facades\Event;

Event::defer(function () {
    $user = User::create(['name' => 'Victoria Otwell']);

    $user->posts()->create(['title' => 'My first post!']);
});
```

Wszystkie zdarzenia uruchomione w domknięciu zostaną wysłane po wykonaniu domknięcia. Zapewnia to, że nasłuchiwacze zdarzeń mają dostęp do wszystkich powiązanych rekordów, które zostały utworzone podczas odroczonego wykonywania. Jeśli wystąpi wyjątek w domknięciu, odroczone zdarzenia nie zostaną wysłane.

Aby odroczyć tylko określone zdarzenia, przekaż tablicę zdarzeń jako drugi argument do metody `defer`:

```php
use App\Models\User;
use Illuminate\Support\Facades\Event;

Event::defer(function () {
    $user = User::create(['name' => 'Victoria Otwell']);

    $user->posts()->create(['title' => 'My first post!']);
}, ['eloquent.created: '.User::class]);
```

<a name="event-subscribers"></a>
## Subskrybenci zdarzeń

<a name="writing-event-subscribers"></a>
### Pisanie subskrybentów zdarzeń

Subskrybenci zdarzeń to klasy, które mogą subskrybować wiele zdarzeń z poziomu samej klasy subskrybenta, umożliwiając definiowanie kilku handlerów zdarzeń w ramach jednej klasy. Subskrybenci powinni definiować metodę `subscribe`, która otrzymuje instancję dyspozytora zdarzeń. Możesz wywołać metodę `listen` na danym dyspozytorze, aby zarejestrować nasłuchiwaczy zdarzeń:

```php
<?php

namespace App\Listeners;

use Illuminate\Auth\Events\Login;
use Illuminate\Auth\Events\Logout;
use Illuminate\Events\Dispatcher;

class UserEventSubscriber
{
    /**
     * Handle user login events.
     */
    public function handleUserLogin(Login $event): void {}

    /**
     * Handle user logout events.
     */
    public function handleUserLogout(Logout $event): void {}

    /**
     * Zarejestruj nasłuchiwaczy dla subskrybenta.
     */
    public function subscribe(Dispatcher $events): void
    {
        $events->listen(
            Login::class,
            [UserEventSubscriber::class, 'handleUserLogin']
        );

        $events->listen(
            Logout::class,
            [UserEventSubscriber::class, 'handleUserLogout']
        );
    }
}
```

Jeśli metody nasłuchiwacza zdarzeń są zdefiniowane w samym subskrybencie, może być wygodniej zwrócić tablicę zdarzeń i nazw metod z metody `subscribe` subskrybenta. Laravel automatycznie określi nazwę klasy subskrybenta podczas rejestrowania nasłuchiwaczy zdarzeń:

```php
<?php

namespace App\Listeners;

use Illuminate\Auth\Events\Login;
use Illuminate\Auth\Events\Logout;
use Illuminate\Events\Dispatcher;

class UserEventSubscriber
{
    /**
     * Handle user login events.
     */
    public function handleUserLogin(Login $event): void {}

    /**
     * Handle user logout events.
     */
    public function handleUserLogout(Logout $event): void {}

    /**
     * Zarejestruj nasłuchiwaczy dla subskrybenta.
     *
     * @return array<string, string>
     */
    public function subscribe(Dispatcher $events): array
    {
        return [
            Login::class => 'handleUserLogin',
            Logout::class => 'handleUserLogout',
        ];
    }
}
```

<a name="registering-event-subscribers"></a>
### Rejestrowanie subskrybentów zdarzeń

Po napisaniu subskrybenta, Laravel automatycznie zarejestruje metody handlera w subskrybencie, jeśli przestrzegają [konwencji wykrywania zdarzeń](#event-discovery) Laravela. W przeciwnym razie możesz ręcznie zarejestrować swojego subskrybenta za pomocą metody `subscribe` fasady `Event`. Zazwyczaj powinno to być wykonane w metodzie `boot` Twojego `AppServiceProvider` aplikacji:

```php
<?php

namespace App\Providers;

use App\Listeners\UserEventSubscriber;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Event::subscribe(UserEventSubscriber::class);
    }
}
```

<a name="testing"></a>
## Testowanie

Podczas testowania kodu, który wysyła zdarzenia, możesz chcieć poinstruować Laravela, aby faktycznie nie wykonywał nasłuchiwaczy zdarzenia, ponieważ kod nasłuchiwacza może być testowany bezpośrednio i oddzielnie od kodu, który wysyła odpowiednie zdarzenie. Oczywiście, aby przetestować sam nasłuchiwacz, możesz utworzyć instancję nasłuchiwacza i wywołać metodę `handle` bezpośrednio w swoim teście.

Używając metody `fake` fasady `Event`, możesz zapobiec wykonywaniu nasłuchiwaczy, wykonać kod testowany, a następnie potwierdzić, które zdarzenia zostały wysłane przez aplikację za pomocą metod `assertDispatched`, `assertNotDispatched` i `assertNothingDispatched`:

```php tab=Pest
<?php

use App\Events\OrderFailedToShip;
use App\Events\OrderShipped;
use Illuminate\Support\Facades\Event;

test('orders can be shipped', function () {
    Event::fake();

    // Wykonaj wysyłkę zamówienia...

    // Potwierdź, że zdarzenie zostało wysłane...
    Event::assertDispatched(OrderShipped::class);

    // Potwierdź, że zdarzenie zostało wysłane dwa razy...
    Event::assertDispatched(OrderShipped::class, 2);

    // Potwierdź, że zdarzenie zostało wysłane raz...
    Event::assertDispatchedOnce(OrderShipped::class);

    // Potwierdź, że zdarzenie nie zostało wysłane...
    Event::assertNotDispatched(OrderFailedToShip::class);

    // Potwierdź, że żadne zdarzenia nie zostały wysłane...
    Event::assertNothingDispatched();
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Events\OrderFailedToShip;
use App\Events\OrderShipped;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Testowanie wysyłki zamówień.
     */
    public function test_orders_can_be_shipped(): void
    {
        Event::fake();

        // Wykonaj wysyłkę zamówienia...

        // Potwierdź, że zdarzenie zostało wysłane...
        Event::assertDispatched(OrderShipped::class);

        // Potwierdź, że zdarzenie zostało wysłane dwa razy...
        Event::assertDispatched(OrderShipped::class, 2);

        // Potwierdź, że zdarzenie zostało wysłane raz...
        Event::assertDispatchedOnce(OrderShipped::class);

        // Potwierdź, że zdarzenie nie zostało wysłane...
        Event::assertNotDispatched(OrderFailedToShip::class);

        // Potwierdź, że żadne zdarzenia nie zostały wysłane...
        Event::assertNothingDispatched();
    }
}
```

Możesz przekazać domknięcie do metod `assertDispatched` lub `assertNotDispatched`, aby sprawdzić, czy zdarzenie zostało wysłane, które przechodzi dany "test prawdy". Jeśli co najmniej jedno zdarzenie zostało wysłane, które przechodzi dany test prawdy, to asercja zakończy się sukcesem:

```php
Event::assertDispatched(function (OrderShipped $event) use ($order) {
    return $event->order->id === $order->id;
});
```

Jeśli po prostu chcesz sprawdzić, czy nasłuchiwacz zdarzeń nasłuchuje danego zdarzenia, możesz użyć metody `assertListening`:

```php
Event::assertListening(
    OrderShipped::class,
    SendShipmentNotification::class
);
```

> [!WARNING]
> Po wywołaniu `Event::fake()`, żadne nasłuchiwacze zdarzeń nie zostaną wykonane. Więc jeśli Twoje testy używają fabryk modeli, które polegają na zdarzeniach, takich jak tworzenie UUID podczas zdarzenia `creating` modelu, powinieneś wywołać `Event::fake()` **po** użyciu swoich fabryk.

<a name="faking-a-subset-of-events"></a>
### Symulowanie podzbioru zdarzeń

Jeśli chcesz symulować nasłuchiwaczy zdarzeń tylko dla określonego zestawu zdarzeń, możesz przekazać je do metody `fake` lub `fakeFor`:

```php tab=Pest
test('orders can be processed', function () {
    Event::fake([
        OrderCreated::class,
    ]);

    $order = Order::factory()->create();

    Event::assertDispatched(OrderCreated::class);

    // Inne zdarzenia są wysyłane normalnie...
    $order->update([
        // ...
    ]);
});
```

```php tab=PHPUnit
/**
 * Testowanie przetwarzania zamówień.
 */
public function test_orders_can_be_processed(): void
{
    Event::fake([
        OrderCreated::class,
    ]);

    $order = Order::factory()->create();

    Event::assertDispatched(OrderCreated::class);

    // Inne zdarzenia są wysyłane normalnie...
    $order->update([
        // ...
    ]);
}
```

Możesz symulować wszystkie zdarzenia z wyjątkiem określonego zestawu zdarzeń używając metody `except`:

```php
Event::fake()->except([
    OrderCreated::class,
]);
```

<a name="scoped-event-fakes"></a>
### Symulacje zdarzeń w zakresie

Jeśli chcesz symulować nasłuchiwaczy zdarzeń tylko dla części testu, możesz użyć metody `fakeFor`:

```php tab=Pest
<?php

use App\Events\OrderCreated;
use App\Models\Order;
use Illuminate\Support\Facades\Event;

test('orders can be processed', function () {
    $order = Event::fakeFor(function () {
        $order = Order::factory()->create();

        Event::assertDispatched(OrderCreated::class);

        return $order;
    });

    // Zdarzenia są wysyłane normalnie, a obserwatorzy będą działać...
    $order->update([
        // ...
    ]);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Events\OrderCreated;
use App\Models\Order;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Testowanie przetwarzania zamówień.
     */
    public function test_orders_can_be_processed(): void
    {
        $order = Event::fakeFor(function () {
            $order = Order::factory()->create();

            Event::assertDispatched(OrderCreated::class);

            return $order;
        });

        // Zdarzenia są wysyłane normalnie, a obserwatorzy będą działać...
        $order->update([
            // ...
        ]);
    }
}
```
