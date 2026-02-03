# Kolejki

- [Wprowadzenie](#introduction)
    - [Połączenia a kolejki](#connections-vs-queues)
    - [Uwagi o sterownikach i wymagania wstępne](#driver-prerequisites)
- [Tworzenie zadań](#creating-jobs)
    - [Generowanie klas zadań](#generating-job-classes)
    - [Struktura klasy](#class-structure)
    - [Unikalne zadania](#unique-jobs)
    - [Zaszyfrowane zadania](#encrypted-jobs)
- [Middleware zadań](#job-middleware)
    - [Ograniczanie częstotliwości](#rate-limiting)
    - [Zapobieganie nakładaniu się zadań](#preventing-job-overlaps)
    - [Ograniczanie wyjątków](#throttling-exceptions)
    - [Pomijanie zadań](#skipping-jobs)
- [Wysyłanie zadań](#dispatching-jobs)
    - [Opóźnione wysyłanie](#delayed-dispatching)
    - [Synchroniczne wysyłanie](#synchronous-dispatching)
    - [Zadania i transakcje bazodanowe](#jobs-and-database-transactions)
    - [Łączenie zadań w łańcuch](#job-chaining)
    - [Dostosowywanie kolejki i połączenia](#customizing-the-queue-and-connection)
    - [Określanie maksymalnych prób / wartości limitu czasu](#max-job-attempts-and-timeout)
    - [SQS FIFO i sprawiedliwe kolejki](#sqs-fifo-and-fair-queues)
    - [Przełączanie awaryjne kolejki](#queue-failover)
    - [Obsługa błędów](#error-handling)
- [Porcjowanie zadań](#job-batching)
    - [Defining Batchable Jobs](#defining-batchable-jobs)
    - [Dispatching Batches](#dispatching-batches)
    - [Chains and Batches](#chains-and-batches)
    - [Adding Jobs to Batches](#adding-jobs-to-batches)
    - [Inspecting Batches](#inspecting-batches)
    - [Cancelling Batches](#cancelling-batches)
    - [Batch Failures](#batch-failures)
    - [Pruning Batches](#pruning-batches)
    - [Storing Batches in DynamoDB](#storing-batches-in-dynamodb)
- [Kolejkowanie domknięć](#queueing-closures)
- [Uruchamianie workera kolejki](#running-the-queue-worker)
    - [The `queue:work` Command](#the-queue-work-command)
    - [Queue Priorities](#queue-priorities)
    - [Queue Workers and Deployment](#queue-workers-and-deployment)
    - [Job Expirations and Timeouts](#job-expirations-and-timeouts)
    - [Pausing and Resuming Queue Workers](#pausing-and-resuming-queue-workers)
- [Konfiguracja Supervisora](#supervisor-configuration)
- [Obsługa nieudanych zadań](#dealing-with-failed-jobs)
    - [Cleaning Up After Failed Jobs](#cleaning-up-after-failed-jobs)
    - [Retrying Failed Jobs](#retrying-failed-jobs)
    - [Ignoring Missing Models](#ignoring-missing-models)
    - [Pruning Failed Jobs](#pruning-failed-jobs)
    - [Storing Failed Jobs in DynamoDB](#storing-failed-jobs-in-dynamodb)
    - [Disabling Failed Job Storage](#disabling-failed-job-storage)
    - [Failed Job Events](#failed-job-events)
- [Czyszczenie zadań z kolejek](#clearing-jobs-from-queues)
- [Monitorowanie kolejek](#monitoring-your-queues)
- [Testowanie](#testing)
    - [Faking a Subset of Jobs](#faking-a-subset-of-jobs)
    - [Testing Job Chains](#testing-job-chains)
    - [Testing Job Batches](#testing-job-batches)
    - [Testing Job / Queue Interactions](#testing-job-queue-interactions)
- [Zdarzenia zadań](#job-events)

<a name="introduction"></a>
## Wprowadzenie

Podczas budowania aplikacji internetowej możesz mieć niektóre zadania, takie jak parsowanie i przechowywanie przesłanego pliku CSV, które trwają zbyt długo, aby wykonać je podczas typowego żądania internetowego. Na szczęście Laravel umożliwia łatwe tworzenie zadań kolejkowanych, które mogą być przetwarzane w tle. Przenosząc czasochłonne zadania do kolejki, Twoja aplikacja może odpowiadać na żądania internetowe z błyskawiczną szybkością i zapewniać lepsze doświadczenie użytkownika dla Twoich klientów.

Kolejki Laravel zapewniają zunifikowane API kolejkowania w różnych backendach kolejek, takich jak [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io), lub nawet relacyjna baza danych.

Opcje konfiguracji kolejki Laravel są przechowywane w pliku konfiguracyjnym `config/queue.php` Twojej aplikacji. W tym pliku znajdziesz konfiguracje połączeń dla każdego ze sterowników kolejek, które są dołączone do frameworka, w tym bazy danych, [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io), and [Beanstalkd](https://beanstalkd.github.io/) oraz sterownik synchroniczny, który będzie wykonywał zadania natychmiast (do użytku podczas rozwoju lub testowania). Dołączony jest również sterownik kolejki `null`, który odrzuca zadania kolejkowane.

> [!NOTE]
> Laravel Horizon to piękny panel administracyjny i system konfiguracji dla kolejek zasilanych przez Redis. Sprawdź pełną [dokumentację Horizon](/docs/{{version}}/horizon) aby uzyskać więcej informacji.

<a name="connections-vs-queues"></a>
### Połączenia a kolejki

Przed rozpoczęciem pracy z kolejkami Laravel ważne jest zrozumienie rozróżnienia między "połączeniami" a "kolejkami". W pliku konfiguracyjnym `config/queue.php` znajduje się tablica konfiguracyjna `connections`. Ta opcja definiuje połączenia do usług kolejek backendowych, takich jak Amazon SQS, Beanstalk lub Redis. Jednak każde połączenie kolejki może mieć wiele "kolejek", które można uznać za różne stosy lub grupy zadań kolejkowanych.

Zauważ, że każdy przykład konfiguracji połączenia w pliku konfiguracyjnym `queue` zawiera atrybut `queue`. Jest to domyślna kolejka, do której zadania będą wysyłane, gdy są wysyłane do danego połączenia. Innymi słowy, jeśli wyślesz zadanie bez jawnego zdefiniowania, do której kolejki powinno być wysłane, zadanie zostanie umieszczone w kolejce zdefiniowanej w atrybucie `queue` konfiguracji połączenia:

```php
use App\Jobs\ProcessPodcast;

// This job is sent to the default connection's default queue...
ProcessPodcast::dispatch();

// This job is sent to the default connection's "emails" queue...
ProcessPodcast::dispatch()->onQueue('emails');
```

Niektóre aplikacje mogą nigdy nie potrzebować przesyłania zadań do wielu kolejek, zamiast tego preferując jedną prostą kolejkę. Jednak przesyłanie zadań do wielu kolejek może być szczególnie przydatne dla aplikacji, które chcą ustalać priorytety lub segmentować sposób przetwarzania zadań, ponieważ worker kolejki Laravel pozwala określić, które kolejki powinien przetwarzać według priorytetu. Na przykład, jeśli przesyłasz zadania do kolejki `high`, możesz uruchomić workera, który da im wyższy priorytet przetwarzania:

```shell
php artisan queue:work --queue=high,default
```

<a name="driver-prerequisites"></a>
### Uwagi o sterownikach i wymagania wstępne

<a name="database"></a>
#### Baza danych

Aby użyć sterownika kolejki `database`, potrzebujesz tabeli bazy danych do przechowywania zadań. Zazwyczaj jest to uwzględnione w domyślnej `0001_01_01_000002_create_jobs_table.php` [migracji bazy danych](/docs/{{version}}/migrations); jednak jeśli Twoja aplikacja nie zawiera tej migracji, możesz użyć polecenia Artisan `make:queue-table`, aby ją utworzyć:

```shell
php artisan make:queue-table

php artisan migrate
```

<a name="redis"></a>
#### Redis

Aby użyć sterownika kolejki `redis`, powinieneś skonfigurować połączenie z bazą danych Redis w pliku konfiguracyjnym `config/database.php`.

> [!WARNING]
> Opcje `serializer` i `compression` Redis nie są obsługiwane przez sterownik kolejki `redis`.

<a name="redis-cluster"></a>
##### Klaster Redis

Jeśli Twoje połączenie kolejki Redis używa [Redis Cluster](https://redis.io/docs/latest/operate/rs/databases/durability-ha/clustering), Twoje nazwy kolejek muszą zawierać [tag hash klucza](https://redis.io/docs/latest/develop/using-commands/keyspace/#hashtags). Jest to wymagane, aby upewnić się, że wszystkie klucze Redis dla danej kolejki są umieszczone w tym samym slocie hash:

```php
'redis' => [
    'driver' => 'redis',
    'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
    'queue' => env('REDIS_QUEUE', '{default}'),
    'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
    'block_for' => null,
    'after_commit' => false,
],
```

<a name="blocking"></a>
##### Blokowanie

Podczas korzystania z kolejki Redis możesz użyć opcji konfiguracji `block_for`, aby określić, jak długo sterownik powinien czekać na dostępność zadania przed iteracją przez pętlę workera i ponownym odpytywaniem bazy danych Redis.

Dostosowanie tej wartości na podstawie obciążenia kolejki może być bardziej wydajne niż ciągłe odpytywanie bazy danych Redis o nowe zadania. Na przykład możesz ustawić wartość na `5`, aby wskazać, że sterownik powinien blokować przez pięć sekund podczas oczekiwania na dostępność zadania:

```php
'redis' => [
    'driver' => 'redis',
    'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
    'queue' => env('REDIS_QUEUE', 'default'),
    'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
    'block_for' => 5,
    'after_commit' => false,
],
```

> [!WARNING]
> Ustawienie `block_for` na `0` spowoduje, że workery kolejki będą blokować się w nieskończoność, dopóki zadanie nie będzie dostępne. Zapobiegnie to również obsłudze sygnałów takich jak `SIGTERM` do czasu przetworzenia następnego zadania.

<a name="other-driver-prerequisites"></a>
#### Inne wymagania wstępne sterowników

Następujące zależności są potrzebne dla wymienionych sterowników kolejek. Te zależności mogą być zainstalowane przez menedżera pakietów Composer:

<div class="content-list" markdown="1">

- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~5.0`
- Redis: `predis/predis ~2.0` or phpredis PHP extension
- [MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/queues/): `mongodb/laravel-mongodb`

</div>

<a name="creating-jobs"></a>
## Tworzenie zadań

<a name="generating-job-classes"></a>
### Generowanie klas zadań

Domyślnie wszystkie zadania kolejkowane dla Twojej aplikacji są przechowywane w katalogu `app/Jobs`. Jeśli katalog `app/Jobs` nie istnieje, zostanie utworzony po uruchomieniu polecenia Artisan `make:job`:

```shell
php artisan make:job ProcessPodcast
```

Wygenerowana klasa zaimplementuje interfejs `Illuminate\Contracts\Queue\ShouldQueue`, wskazując Laravel, że zadanie powinno zostać przesłane do kolejki, aby działać asynchronicznie.

> [!NOTE]
> Szkice zadań mogą być dostosowywane za pomocą [publikowania szkiców](/docs/{{version}}/artisan#stub-customization).

<a name="class-structure"></a>
### Struktura klasy

Klasy zadań są bardzo proste, zwykle zawierają tylko metodę `handle`, która jest wywoływana, gdy zadanie jest przetwarzane przez kolejkę. Aby zacząć, przyjrzyjmy się przykładowej klasie zadania. W tym przykładzie udamy, że zarządzamy usługą publikowania podcastów i musimy przetworzyć przesłane pliki podcastów przed ich opublikowaniem:

```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Podcast $podcast,
    ) {}

    /**
     * Execute the job.
     */
    public function handle(AudioProcessor $processor): void
    {
        // Process uploaded podcast...
    }
}
```

W tym przykładzie zauważ, że mogliśmy przekazać [model Eloquent](/docs/{{version}}/eloquent) bezpośrednio do konstruktora zadania kolejkowanego. Dzięki cechie `Queueable`, której używa zadanie, modele Eloquent i ich załadowane relacje zostaną elegancko zserializowane i zdeserializowane podczas przetwarzania zadania.

Jeśli Twoje zadanie kolejkowane przyjmuje model Eloquent w swoim konstruktorze, tylko identyfikator modelu zostanie zserializowany do kolejki. Gdy zadanie jest rzeczywiście obsługiwane, system kolejek automatycznie ponownie pobierze pełną instancję modelu i jej załadowane relacje z bazy danych. To podejście do serializacji modelu pozwala na wysyłanie znacznie mniejszych ładunków zadań do sterownika kolejki.

<a name="handle-method-dependency-injection"></a>
#### Wstrzykiwanie zależności w metodzie `handle`

Metoda `handle` jest wywoływana, gdy zadanie jest przetwarzane przez kolejkę. Zauważ, że możemy wskazać zależności typów w metodzie `handle` zadania. Laravel [kontener usług](/docs/{{version}}/container) automatycznie wstrzykuje te zależności.

Jeśli chciałbyś mieć pełną kontrolę nad tym, jak kontener wstrzykuje zależności do metody `handle`, możesz użyć metody `bindMethod` kontenera. Metoda `bindMethod` akceptuje callback, który otrzymuje zadanie i kontener. W callbacku możesz swobodnie wywołać metodę `handle` w dowolny sposób. Zazwyczaj powinieneś wywołać tę metodę z metody `boot` Twojego `App\Providers\AppServiceProvider` [dostawcy usług](/docs/{{version}}/providers):

```php
use App\Jobs\ProcessPodcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Foundation\Application;

$this->app->bindMethod([ProcessPodcast::class, 'handle'], function (ProcessPodcast $job, Application $app) {
    return $job->handle($app->make(AudioProcessor::class));
});
```

> [!WARNING]
> Dane binarne, takie jak surowa zawartość obrazu, powinny być przekazane przez funkcję `base64_encode` przed przekazaniem do zadania kolejkowanego. W przeciwnym razie zadanie może nie zostać prawidłowo zserializowane do JSON przy umieszczaniu w kolejce.

<a name="handling-relationships"></a>
#### Kolejkowane relacje

Ponieważ wszystkie załadowane relacje modelu Eloquent są również serializowane, gdy zadanie jest dodawane do kolejki, zserializowany ciąg zadania może czasami stać się dość duży. Ponadto, gdy zadanie jest deserializowane, a relacje modelu są ponownie pobierane z bazy danych, zostaną pobrane w całości. Wszelkie poprzednie ograniczenia relacji, które zostały zastosowane przed serializacją modelu podczas procesu kolejkowania zadania, nie zostaną zastosowane, gdy zadanie jest deserializowane. Dlatego, jeśli chcesz pracować z podzbiorem danej relacji, powinieneś ponownie ograniczyć tę relację w swoim zadaniu kolejkowanym.

Lub, aby zapobiec serializacji relacji, możesz wywołać metodę `withoutRelations` na modelu podczas ustawiania wartości właściwości. Ta metoda zwróci instancję modelu bez załadowanych relacji:

```php
/**
 * Create a new job instance.
 */
public function __construct(
    Podcast $podcast,
) {
    $this->podcast = $podcast->withoutRelations();
}
```

Jeśli używasz [promocji właściwości konstruktora PHP](https://www.php.net/manual/en/language.oop5.decon.php#language.oop5.decon.constructor.promotion) i chcesz wskazać, że model Eloquent nie powinien mieć serializowanych relacji, możesz użyć atrybutu `WithoutRelations`:

```php
use Illuminate\Queue\Attributes\WithoutRelations;

/**
 * Create a new job instance.
 */
public function __construct(
    #[WithoutRelations]
    public Podcast $podcast,
) {}
```

Dla wygody, jeśli chcesz serializować wszystkie modele bez relacji, możesz zastosować atrybut `WithoutRelations` do całej klasy zamiast stosować atrybut do każdego modelu:

```php
<?php

namespace App\Jobs;

use App\Models\DistributionPlatform;
use App\Models\Podcast;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Queue\Attributes\WithoutRelations;

#[WithoutRelations]
class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Podcast $podcast,
        public DistributionPlatform $platform,
    ) {}
}
```

Jeśli zadanie otrzymuje kolekcję lub tablicę modeli Eloquent zamiast pojedynczego modelu, modele w tej kolekcji nie będą miały przywróconych relacji, gdy zadanie zostanie zdeserializowane i wykonane. Ma to na celu zapobieganie nadmiernemu zużyciu zasobów w zadaniach, które obsługują dużą liczbę modeli.

<a name="unique-jobs"></a>
### Unikalne zadania

> [!WARNING]
> Unikalne zadania wymagają sterownika pamięci podręcznej, który obsługuje [blokady](/docs/{{version}}/cache#atomic-blokady). Obecnie sterowniki pamięci podręcznej `memcached`, `redis`, `dynamodb`, `database`, `file` i `array` obsługują blokady atomowe.

> [!WARNING]
> Ograniczenia unikalnych zadań nie dotyczą zadań w partiach.

Czasami możesz chcieć upewnić się, że tylko jedna instancja określonego zadania znajduje się w kolejce w danym momencie. Możesz to zrobić, implementując interfejs `ShouldBeUnique` w swojej klasie zadania. Ten interfejs nie wymaga definiowania żadnych dodatkowych metod w Twojej klasie:

```php
<?php

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    // ...
}
```

W powyższym przykładzie zadanie `UpdateSearchIndex` jest unikalne. Więc zadanie nie zostanie wysłane, jeśli inna instancja zadania jest już w kolejce i nie zakończyła przetwarzania.

W niektórych przypadkach możesz chcieć zdefiniować konkretny "klucz", który sprawia, że zadanie jest unikalne, lub możesz chcieć określić limit czasu, po którym zadanie nie jest już unikalne. Aby to osiągnąć, możesz zdefiniować właściwości lub metody `uniqueId` i `uniqueFor` w swojej klasie zadania:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    /**
     * The product instance.
     *
     * @var \App\Models\Product
     */
    public $product;

    /**
     * The number of seconds after which the job's unique lock will be released.
     *
     * @var int
     */
    public $uniqueFor = 3600;

    /**
     * Get the unique ID for the job.
     */
    public function uniqueId(): string
    {
        return $this->product->id;
    }
}
```

W powyższym przykładzie zadanie `UpdateSearchIndex` jest unikalne według identyfikatora produktu. Więc wszelkie nowe wysyłki zadania z tym samym identyfikatorem produktu będą ignorowane, dopóki istniejące zadanie nie zakończy przetwarzania. Ponadto, jeśli istniejące zadanie nie zostanie przetworzone w ciągu godziny, unikalna blokada zostanie zwolniona i kolejne zadanie z tym samym unikalnym kluczem może zostać wysłane do kolejki.

> [!WARNING]
> Jeśli Twoja aplikacja wysyła zadania z wielu serwerów internetowych lub kontenerów, powinieneś upewnić się, że wszystkie Twoje serwery komunikują się z tym samym centralnym serwerem pamięci podręcznej, aby Laravel mógł dokładnie określić, czy zadanie jest unikalne.

<a name="keeping-jobs-unique-until-processing-begins"></a>
#### Zachowanie unikalności zadań do rozpoczęcia przetwarzania

Domyślnie unikalne zadania są "odblokowane" po zakończeniu przetwarzania zadania lub po niepowodzeniu wszystkich prób ponowienia. Jednak mogą być sytuacje, w których chciałbyś, aby Twoje zadanie zostało odblokowane natychmiast przed przetworzeniem. Aby to osiągnąć, Twoje zadanie powinno implementować kontrakt `ShouldBeUniqueUntilProcessing` zamiast kontraktu `ShouldBeUnique`:

```php
<?php

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUniqueUntilProcessing
{
    // ...
}
```

<a name="unique-job-blokady"></a>
#### Blokady unikalnych zadań

W tle, gdy zadanie `ShouldBeUnique` jest wysyłane, Laravel próbuje uzyskać [blokadę](/docs/{{version}}/cache#atomic-blokady) z kluczem `uniqueId`. Jeśli blokada jest już utrzymywana, zadanie nie jest wysyłane. Ta blokada jest zwalniana, gdy zadanie zakończy przetwarzanie lub nie powiedzie się we wszystkich próbach ponowienia. Domyślnie Laravel użyje domyślnego sterownika pamięci podręcznej do uzyskania tej blokady. Jednak jeśli chcesz użyć innego sterownika do uzyskania blokady, możesz zdefiniować metodę `uniqueVia`, która zwraca sterownik pamięci podręcznej, który powinien być używany:

```php
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Support\Facades\Cache;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    // ...

    /**
     * Get the cache driver for the unique job lock.
     */
    public function uniqueVia(): Repository
    {
        return Cache::driver('redis');
    }
}
```

> [!NOTE]
> Jeśli potrzebujesz tylko ograniczyć współbieżne przetwarzanie zadania, użyj zamiast tego middleware zadania [WithoutOverlapping](/docs/{{version}}/queues#preventing-job-overlaps).

<a name="encrypted-jobs"></a>
### Zaszyfrowane zadania

Laravel pozwala zapewnić prywatność i integralność danych zadania poprzez [szyfrowanie](/docs/{{version}}/encryption). Aby rozpocząć, po prostu dodaj interfejs `ShouldBeEncrypted` do klasy zadania. Po dodaniu tego interfejsu do klasy, Laravel automatycznie zaszyfruje Twoje zadanie przed umieszczeniem go w kolejce:

```php
<?php

use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;

class UpdateSearchIndex implements ShouldQueue, ShouldBeEncrypted
{
    // ...
}
```

<a name="job-middleware"></a>
## Middleware zadań

Middleware zadań pozwala na owinięcie niestandardowej logiki wokół wykonywania zadań kolejkowych, redukując szablon w samych zadaniach. Na przykład rozważ następującą metodę `handle`, która wykorzystuje funkcje ograniczania częstotliwości Redis w Laravel, aby pozwolić na przetwarzanie tylko jednego zadania co pięć sekund:

```php
use Illuminate\Support\Facades\Redis;

/**
 * Execute the job.
 */
public function handle(): void
{
    Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
        info('Lock obtained...');

        // Handle job...
    }, function () {
        // Could not obtain lock...

        return $this->release(5);
    });
}
```

Chociaż ten kod jest prawidłowy, implementacja metody `handle` staje się hałaśliwa, ponieważ jest zaśmiecona logiką ograniczania częstotliwości Redis. Ponadto ta logika ograniczania częstotliwości musi być duplikowana dla wszystkich innych zadań, które chcemy ograniczyć. Zamiast ograniczać częstotliwość w metodzie handle, moglibyśmy zdefiniować middleware zadania, które obsługuje ograniczanie częstotliwości:

```php
<?php

namespace App\Jobs\Middleware;

use Closure;
use Illuminate\Support\Facades\Redis;

class RateLimited
{
    /**
     * Process the queued job.
     *
     * @param  \Closure(object): void  $next
     */
    public function handle(object $job, Closure $next): void
    {
        Redis::throttle('key')
            ->block(0)->allow(1)->every(5)
            ->then(function () use ($job, $next) {
                // Lock obtained...

                $next($job);
            }, function () use ($job) {
                // Could not obtain lock...

                $job->release(5);
            });
    }
}
```

Jak widać, podobnie jak [middleware tras](/docs/{{version}}/middleware), middleware zadań otrzymuje zadanie do przetworzenia i callback, który powinien zostać wywołany, aby kontynuować przetwarzanie zadania.

Możesz wygenerować nową klasę middleware zadania za pomocą polecenia Artisan `make:job-middleware`. Po utworzeniu middleware zadania, mogą one zostać dołączone do zadania, zwracając je z metody `middleware` zadania. Ta metoda nie istnieje w zadaniach utworzonych przez polecenie Artisan `make:job`, więc będziesz musiał ręcznie dodać ją do swojej klasy zadania:

```php
use App\Jobs\Middleware\RateLimited;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new RateLimited];
}
```

> [!NOTE]
> Middleware zadań może być również przypisane do [kolejkowalnych nasłuchiwaczy zdarzeń](/docs/{{version}}/events#queued-event-listeners), [mailables](/docs/{{version}}/mail#queueing-mail) i [powiadomień](/docs/{{version}}/notifications#queueing-notifications).

<a name="rate-limiting"></a>
### Ograniczanie częstotliwości

Chociaż właśnie pokazaliśmy, jak napisać własne middleware ograniczania częstotliwości zadań, Laravel faktycznie zawiera middleware ograniczania częstotliwości, którego możesz użyć do ograniczania zadań. Podobnie jak [ograniczniki częstotliwości tras](/docs/{{version}}/routing#defining-rate-limiters), ograniczniki częstotliwości zadań są definiowane za pomocą metody `for` fasady `RateLimiter`.

Na przykład możesz chcieć pozwolić użytkownikom na tworzenie kopii zapasowych ich danych raz na godzinę, nie nakładając takiego limitu na klientów premium. Aby to osiągnąć, możesz zdefiniować `RateLimiter` w metodzie `boot` Twojego `AppServiceProvider`:

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    RateLimiter::for('backups', function (object $job) {
        return $job->user->vipCustomer()
            ? Limit::none()
            : Limit::perHour(1)->by($job->user->id);
    });
}
```

W powyższym przykładzie zdefiniowaliśmy godzinowy limit częstotliwości; jednak możesz łatwo zdefiniować limit częstotliwości na podstawie minut, używając metody `perMinute`. Ponadto możesz przekazać dowolną wartość do metody `by` limitu częstotliwości; jednak ta wartość jest najczęściej używana do segmentacji limitów częstotliwości według klienta:

```php
return Limit::perMinute(50)->by($job->user->id);
```

Po zdefiniowaniu limitu częstotliwości możesz dołączyć ogranicznik częstotliwości do swojego zadania za pomocą middleware `Illuminate\Queue\Middleware\RateLimited`. Za każdym razem, gdy zadanie przekroczy limit częstotliwości, to middleware zwolni zadanie z powrotem do kolejki z odpowiednim opóźnieniem na podstawie czasu trwania limitu częstotliwości:

```php
use Illuminate\Queue\Middleware\RateLimited;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new RateLimited('backups')];
}
```

Zwolnienie zadania z limitem częstotliwości z powrotem do kolejki nadal zwiększy całkowitą liczbę `attempts` zadania. Możesz chcieć dostosować właściwości `tries` i `maxExceptions` w swojej klasie zadania odpowiednio. Lub możesz użyć [metody retryUntil](#time-based-attempts), aby zdefiniować czas, do którego zadanie nie powinno być próbowane.

Używając metody `releaseAfter`, możesz również określić liczbę sekund, które muszą upłynąć przed ponowną próbą zwolnionego zadania:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new RateLimited('backups'))->releaseAfter(60)];
}
```

Jeśli nie chcesz, aby zadanie było ponawiane, gdy jest ograniczone częstotliwością, możesz użyć metody `dontRelease`:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new RateLimited('backups'))->dontRelease()];
}
```

> [!NOTE]
> Jeśli używasz Redis, możesz użyć middleware `Illuminate\Queue\Middleware\RateLimitedWithRedis`, które jest dostrojone dla Redis i bardziej wydajne niż podstawowe middleware ograniczania częstotliwości.

<a name="preventing-job-overlaps"></a>
### Zapobieganie nakładaniu się zadań

Laravel zawiera middleware `Illuminate\Queue\Middleware\WithoutOverlapping`, które pozwala zapobiegać nakładaniu się zadań na podstawie dowolnego klucza. Może to być pomocne, gdy zadanie kolejkowane modyfikuje zasób, który powinien być modyfikowany tylko przez jedno zadanie naraz.

Na przykład wyobraźmy sobie, że masz zadanie kolejkowane, które aktualizuje ocenę kredytową użytkownika i chcesz zapobiec nakładaniu się zadań aktualizacji oceny kredytowej dla tego samego identyfikatora użytkownika. Aby to osiągnąć, możesz zwrócić middleware `WithoutOverlapping` z metody `middleware` Twojego zadania:

```php
use Illuminate\Queue\Middleware\WithoutOverlapping;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new WithoutOverlapping($this->user->id)];
}
```

Zwolnienie nakładającego się zadania z powrotem do kolejki nadal zwiększy całkowitą liczbę prób zadania. Możesz chcieć dostosować właściwości `tries` i `maxExceptions` w swojej klasie zadania odpowiednio. Na przykład pozostawienie właściwości `tries` jako 1, jak jest domyślnie, zapobiegnie ponownemu próbowaniu dowolnego nakładającego się zadania później.

Wszystkie nakładające się zadania tego samego typu zostaną zwolnione z powrotem do kolejki. Możesz również określić liczbę sekund, które muszą upłynąć przed ponowną próbą zwolnionego zadania:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->releaseAfter(60)];
}
```

Jeśli chcesz natychmiast usunąć wszystkie nakładające się zadania, aby nie były ponawiane, możesz użyć metody `dontRelease`:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->dontRelease()];
}
```

Middleware `WithoutOverlapping` jest zasilane przez funkcję blokady atomowej Laravel. Czasami Twoje zadanie może nieoczekiwanie się nie powieść lub przekroczyć limit czasu w taki sposób, że blokada nie zostanie zwolniona. Dlatego możesz jawnie zdefiniować czas wygaśnięcia blokady za pomocą metody `expireAfter`. Na przykład poniższy przykład poinstruuje Laravel, aby zwolnił blokadę `WithoutOverlapping` trzy minuty po rozpoczęciu przetwarzania zadania:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->expireAfter(180)];
}
```

> [!WARNING]
> Middleware `WithoutOverlapping` wymaga sterownika pamięci podręcznej, który obsługuje [blokady](/docs/{{version}}/cache#atomic-blokady). Obecnie sterowniki pamięci podręcznej `memcached`, `redis`, `dynamodb`, `database`, `file` i `array` obsługują blokady atomowe.

<a name="sharing-lock-keys"></a>
#### Współdzielenie kluczy blokad między klasami zadań

Domyślnie middleware `WithoutOverlapping` zapobiegnie nakładaniu się tylko zadań tej samej klasy. Więc, chociaż dwie różne klasy zadań mogą używać tego samego klucza blokady, nie będą one powstrzymane przed nakładaniem się. Jednak możesz poinstruować Laravel, aby zastosował klucz we wszystkich klasach zadań, używając metody `shared`:

```php
use Illuminate\Queue\Middleware\WithoutOverlapping;

class ProviderIsDown
{
    // ...

    public function middleware(): array
    {
        return [
            (new WithoutOverlapping("status:{$this->provider}"))->shared(),
        ];
    }
}

class ProviderIsUp
{
    // ...

    public function middleware(): array
    {
        return [
            (new WithoutOverlapping("status:{$this->provider}"))->shared(),
        ];
    }
}
```

<a name="throttling-exceptions"></a>
### Ograniczanie wyjątków

Laravel zawiera middleware `Illuminate\Queue\Middleware\ThrottlesExceptions`, które pozwala ograniczać wyjątki. Po zgłoszeniu przez zadanie określonej liczby wyjątków, wszystkie dalsze próby wykonania zadania są opóźnione do momentu upłynięcia określonego przedziału czasu. To middleware jest szczególnie przydatne dla zadań, które współdziałają z usługami stron trzecich, które są niestabilne.

Na przykład wyobraźmy sobie zadanie kolejkowane, które współdziała z API strony trzeciej, które zaczyna zgłaszać wyjątki. Aby ograniczyć wyjątki, możesz zwrócić middleware `ThrottlesExceptions` z metody `middleware` Twojego zadania. Zazwyczaj to middleware powinno być sparowane z zadaniem, które implementuje [próby oparte na czasie](#time-based-attempts):

```php
use DateTime;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new ThrottlesExceptions(10, 5 * 60)];
}

/**
 * Determine the time at which the job should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->plus(minutes: 30);
}
```

Pierwszy argument konstruktora akceptowany przez middleware to liczba wyjątków, które zadanie może zgłosić przed ograniczeniem, podczas gdy drugi argument konstruktora to liczba sekund, które powinny upłynąć przed ponowną próbą zadania po ograniczeniu. W powyższym przykładzie kodu, jeśli zadanie zgłosi 10 kolejnych wyjątków, będziemy czekać 5 minut przed ponowną próbą zadania, ograniczone 30-minutowym limitem czasu.

Kiedy zadanie zgłasza wyjątek, ale próg wyjątków nie został jeszcze osiągnięty, zadanie będzie zazwyczaj ponawiane natychmiast. Jednak możesz określić liczbę minut, o które takie zadanie powinno być opóźnione, wywołując metodę `backoff` podczas dołączania middleware do zadania:

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 5 * 60))->backoff(5)];
}
```

Wewnętrznie to middleware używa systemu cache Laravel do implementacji ograniczania częstotliwości, a nazwa klasy zadania jest używana jako "klucz" cache. Możesz nadpisać ten klucz, wywołując metodę `by` podczas dołączania middleware do swojego zadania. Może to być przydatne, jeśli masz wiele zadań współdziałających z tą samą usługą strony trzeciej i chcesz, aby współdzieliły wspólny "kosz" ograniczania, zapewniając, że respektują pojedynczy współdzielony limit:

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->by('key')];
}
```

Domyślnie to middleware ograniczy każdy wyjątek. Możesz zmodyfikować to zachowanie, wywołując metodę `when` podczas dołączania middleware do swojego zadania. Wyjątek zostanie wtedy ograniczony tylko wtedy, gdy domknięcie dostarczone do metody `when` zwróci `true`:

```php
use Illuminate\Http\Client\HttpClientException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->when(
        fn (Throwable $throwable) => $throwable instanceof HttpClientException
    )];
}
```

W przeciwieństwie do metody `when`, która zwalnia zadanie z powrotem do kolejki lub zgłasza wyjątek, metoda `deleteWhen` pozwala całkowicie usunąć zadanie, gdy wystąpi dany wyjątek:

```php
use App\Exceptions\CustomerDeletedException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(2, 10 * 60))->deleteWhen(CustomerDeletedException::class)];
}
```

Jeśli chciałbyś, aby ograniczone wyjątki były zgłaszane do obsługi wyjątków Twojej aplikacji, możesz to zrobić, wywołując metodę `report` podczas dołączania middleware do swojego zadania. Opcjonalnie możesz dostarczyć domknięcie do metody `report`, a wyjątek zostanie zgłoszony tylko wtedy, gdy dane domknięcie zwróci `true`:

```php
use Illuminate\Http\Client\HttpClientException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->report(
        fn (Throwable $throwable) => $throwable instanceof HttpClientException
    )];
}
```

> [!NOTE]
> Jeśli używasz Redis, możesz użyć middleware `Illuminate\Queue\Middleware\ThrottlesExceptionsWithRedis`, które jest dostrojone dla Redis i bardziej wydajne niż podstawowe middleware ograniczania wyjątków.

<a name="skipping-jobs"></a>
### Pomijanie zadań

Middleware `Skip` pozwala określić, że zadanie powinno być pominięte / usunięte bez konieczności modyfikowania logiki zadania. Metoda `Skip::when` usunie zadanie, jeśli podany warunek zostanie oceniony jako `true`, podczas gdy metoda `Skip::unless` usunie zadanie, jeśli warunek zostanie oceniony jako `false`:

```php
use Illuminate\Queue\Middleware\Skip;

/**
 * Get the middleware the job should pass through.
 */
public function middleware(): array
{
    return [
        Skip::when($condition),
    ];
}
```

Możesz również przekazać `Closure` do metod `when` i `unless` w celu bardziej złożonej oceny warunkowej:

```php
use Illuminate\Queue\Middleware\Skip;

/**
 * Get the middleware the job should pass through.
 */
public function middleware(): array
{
    return [
        Skip::when(function (): bool {
            return $this->shouldSkip();
        }),
    ];
}
```

<a name="dispatching-jobs"></a>
## Wysyłanie zadań

Po napisaniu klasy zadania możesz je wysłać za pomocą metody `dispatch` na samym zadaniu. Argumenty przekazane do metody `dispatch` zostaną przekazane do konstruktora zadania:

```php
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // ...

        ProcessPodcast::dispatch($podcast);

        return redirect('/podcasts');
    }
}
```

Jeśli chcesz warunkowo wysłać zadanie, możesz użyć metod `dispatchIf` i `dispatchUnless`:

```php
ProcessPodcast::dispatchIf($accountActive, $podcast);

ProcessPodcast::dispatchUnless($accountSuspended, $podcast);
```

W nowych aplikacjach Laravel połączenie `database` jest zdefiniowane jako domyślna kolejka. Możesz określić inne domyślne połączenie kolejki, zmieniając zmienną środowiskową `QUEUE_CONNECTION` w pliku `.env` Twojej aplikacji.

<a name="delayed-dispatching"></a>
### Opóźnione wysyłanie

Jeśli chciałbyś określić, że zadanie nie powinno być natychmiast dostępne do przetwarzania przez workera kolejki, możesz użyć metody `delay` podczas wysyłania zadania. Na przykład określmy, że zadanie nie powinno być dostępne do przetwarzania przed upływem 10 minut od jego wysłania:

```php
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // ...

        ProcessPodcast::dispatch($podcast)
            ->delay(now()->plus(minutes: 10));

        return redirect('/podcasts');
    }
}
```

W niektórych przypadkach zadania mogą mieć skonfigurowane domyślne opóźnienie. Jeśli musisz ominąć to opóźnienie i wysłać zadanie do natychmiastowego przetwarzania, możesz użyć metody `withoutDelay`:

```php
ProcessPodcast::dispatch($podcast)->withoutDelay();
```

> [!WARNING]
> Usługa kolejki Amazon SQS ma maksymalny czas opóźnienia wynoszący 15 minut.

<a name="synchronous-dispatching"></a>
### Synchroniczne wysyłanie

Jeśli chciałbyś wysłać zadanie natychmiast (synchronicznie), możesz użyć metody `dispatchSync`. Podczas korzystania z tej metody zadanie nie zostanie dodane do kolejki i zostanie wykonane natychmiast w bieżącym procesie:

```php
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // Create podcast...

        ProcessPodcast::dispatchSync($podcast);

        return redirect('/podcasts');
    }
}
```

<a name="deferred-dispatching"></a>
#### Odroczone wysyłanie

Korzystając z odroczonego synchronicznego wysyłania, możesz wysłać zadanie do przetworzenia podczas bieżącego procesu, ale po wysłaniu odpowiedzi HTTP do użytkownika. Pozwala to na przetwarzanie zadań "kolejkowanych" synchronicznie bez spowalniania doświadczenia użytkownika w aplikacji. Aby odroczyć wykonanie zadania synchronicznego, wyślij zadanie do połączenia `deferred`:

```php
RecordDelivery::dispatch($order)->onConnection('deferred');
```

Połączenie `deferred` służy również jako domyślna [kolejka awaryjna](#queue-failover).

Podobnie połączenie `background` przetwarza zadania po wysłaniu odpowiedzi HTTP do użytkownika; jednak zadanie jest przetwarzane w oddzielnie utworzonym procesie PHP, co pozwala PHP-FPM / workerowi aplikacji być dostępnym do obsługi kolejnego przychodzącego żądania HTTP:

```php
RecordDelivery::dispatch($order)->onConnection('background');
```

<a name="jobs-and-database-transactions"></a>
### Zadania i transakcje bazodanowe

Chociaż wysyłanie zadań w ramach transakcji bazodanowych jest całkowicie w porządku, powinieneś szczególnie zadbać o to, aby Twoje zadanie faktycznie mogło zostać pomyślnie wykonane. Podczas wysyłania zadania w ramach transakcji możliwe jest, że zadanie zostanie przetworzone przez workera przed zatwierdzeniem transakcji nadrzędnej. Kiedy to się stanie, wszelkie aktualizacje modeli lub rekordów bazy danych dokonane podczas transakcji bazodanowych mogą jeszcze nie być odzwierciedlone w bazie danych. Ponadto wszelkie modele lub rekordy bazy danych utworzone w ramach transakcji mogą nie istnieć w bazie danych.

Na szczęście Laravel zapewnia kilka metod radzenia sobie z tym problemem. Po pierwsze, możesz ustawić opcję połączenia `after_commit` w tablicy konfiguracji połączenia kolejki:

```php
'redis' => [
    'driver' => 'redis',
    // ...
    'after_commit' => true,
],
```

Gdy opcja `after_commit` jest ustawiona na `true`, możesz wysyłać zadania w ramach transakcji bazodanowych; jednak Laravel poczeka, aż otwarte nadrzędne transakcje bazodanowe zostaną zatwierdzone, zanim faktycznie wyśle zadanie. Oczywiście, jeśli obecnie nie ma otwartych transakcji bazodanowych, zadanie zostanie wysłane natychmiast.

Jeśli transakcja zostanie wycofana z powodu wyjątku, który wystąpił podczas transakcji, zadania, które zostały wysłane podczas tej transakcji, zostaną odrzucone.

> [!NOTE]
> Ustawienie opcji konfiguracji `after_commit` na `true` spowoduje również, że wszystkie kolejkowane słuchacze zdarzeń, mailables, powiadomienia i zdarzenia broadcast będą wysyłane po zatwierdzeniu wszystkich otwartych transakcji bazodanowych.

<a name="specifying-commit-dispatch-behavior-inline"></a>
#### Określanie zachowania wysyłki zatwierdzenia w linii

Jeśli nie ustawisz opcji konfiguracji połączenia kolejki `after_commit` na `true`, nadal możesz wskazać, że określone zadanie powinno zostać wysłane po zatwierdzeniu wszystkich otwartych transakcji bazodanowych. Aby to osiągnąć, możesz łączyć metodę `afterCommit` z operacją wysyłki:

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatch($podcast)->afterCommit();
```

Podobnie, jeśli opcja konfiguracji `after_commit` jest ustawiona na `true`, możesz wskazać, że określone zadanie powinno zostać wysłane natychmiast, bez czekania na zatwierdzenie jakichkolwiek otwartych transakcji bazodanowych:

```php
ProcessPodcast::dispatch($podcast)->beforeCommit();
```

<a name="job-chaining"></a>
### Łączenie zadań w łańcuch

Łączenie zadań w łańcuch pozwala określić listę kolejkowanych zadań, które powinny być uruchamiane sekwencyjnie po pomyślnym wykonaniu zadania głównego. Jeśli jedno zadanie w sekwencji nie powiedzie się, pozostałe zadania nie zostaną uruchomione. Aby wykonać łańcuch kolejkowanych zadań, możesz użyć metody `chain` dostarczonej przez fasadę `Bus`. Magistrala poleceń Laravela jest komponentem niższego poziomu, na którym zbudowane jest wysyłanie kolejkowanych zadań:

```php
use App\Jobs\OptimizePodcast;
use App\Jobs\ProcessPodcast;
use App\Jobs\ReleasePodcast;
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->dispatch();
```

Oprócz łączenia w łańcuch instancji klas zadań, możesz również łączyć domknięcia:

```php
Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    function () {
        Podcast::update(/* ... */);
    },
])->dispatch();
```

> [!WARNING]
> Usuwanie zadań za pomocą metody `$this->delete()` w ramach zadania nie uniemożliwi przetwarzania zadań łańcuchowych. Łańcuch przestanie się wykonywać tylko wtedy, gdy zadanie w łańcuchu nie powiedzie się.

<a name="chain-connection-queue"></a>
#### Łańcuch połączeń i kolejka

Jeśli chcesz określić połączenie i kolejkę, które powinny być używane dla zadań łańcuchowych, możesz użyć metod `onConnection` i `onQueue`. Te metody określają połączenie kolejki i nazwę kolejki, które powinny być używane, chyba że kolejkowane zadanie jest jawnie przypisane do innego połączenia / kolejki:

```php
Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->onConnection('redis')->onQueue('podcasts')->dispatch();
```

<a name="adding-jobs-to-the-chain"></a>
#### Dodawanie zadań do łańcucha

Czasami może być konieczne dodanie zadania na początku lub na końcu istniejącego łańcucha zadań z poziomu innego zadania w tym łańcuchu. Możesz to osiągnąć używając metod `prependToChain` i `appendToChain`:

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    // ...

    // Prepend to the current chain, run job immediately after current job...
    $this->prependToChain(new TranscribePodcast);

    // Append to the current chain, run job at end of chain...
    $this->appendToChain(new TranscribePodcast);
}
```

<a name="chain-failures"></a>
#### Niepowodzenia łańcucha

Podczas łączenia zadań w łańcuch możesz użyć metody `catch`, aby określić domknięcie, które powinno zostać wywołane, jeśli zadanie w łańcuchu nie powiedzie się. Podane wywołanie zwrotne otrzyma instancję `Throwable`, która spowodowała niepowodzenie zadania:

```php
use Illuminate\Support\Facades\Bus;
use Throwable;

Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->catch(function (Throwable $e) {
    // A job within the chain has failed...
})->dispatch();
```

> [!WARNING]
> Ponieważ wywołania zwrotne łańcucha są serializowane i wykonywane później przez kolejkę Laravel, nie powinieneś używać zmiennej `$this` w wywołaniach zwrotnych łańcucha.

<a name="customizing-the-queue-and-connection"></a>
### Dostosowywanie kolejki i połączenia

<a name="dispatching-to-a-particular-queue"></a>
#### Wysyłanie do określonej kolejki

Umieszczając zadania w różnych kolejkach, możesz "kategoryzować" swoje kolejkowane zadania, a nawet ustalać priorytety, ile workerów przypisać do różnych kolejek. Pamiętaj, że nie umieszcza to zadań w różnych "połączeniach" kolejki, jak zdefiniowano w pliku konfiguracji kolejki, ale tylko w określonych kolejkach w ramach jednego połączenia. Aby określić kolejkę, użyj metody `onQueue` podczas wysyłania zadania:

```php
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // Create podcast...

        ProcessPodcast::dispatch($podcast)->onQueue('processing');

        return redirect('/podcasts');
    }
}
```

Alternatywnie możesz określić kolejkę zadania, wywołując metodę `onQueue` w konstruktorze zadania:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct()
    {
        $this->onQueue('processing');
    }
}
```

<a name="dispatching-to-a-particular-connection"></a>
#### Wysyłanie do określonego połączenia

Jeśli Twoja aplikacja współdziała z wieloma połączeniami kolejki, możesz określić, do którego połączenia wysłać zadanie, używając metody `onConnection`:

```php
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // Create podcast...

        ProcessPodcast::dispatch($podcast)->onConnection('sqs');

        return redirect('/podcasts');
    }
}
```

Możesz połączyć metody `onConnection` i `onQueue`, aby określić połączenie i kolejkę dla zadania:

```php
ProcessPodcast::dispatch($podcast)
    ->onConnection('sqs')
    ->onQueue('processing');
```

Alternatywnie możesz określić połączenie zadania, wywołując metodę `onConnection` w konstruktorze zadania:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct()
    {
        $this->onConnection('sqs');
    }
}
```

<a name="max-job-attempts-and-timeout"></a>
### Określanie maksymalnych prób / wartości limitu czasu

<a name="max-attempts"></a>
#### Maksymalne próby

Próby zadań są podstawową koncepcją systemu kolejek Laravel i napędzają wiele zaawansowanych funkcji. Choć mogą wydawać się mylące na początku, ważne jest zrozumienie, jak działają, zanim zmodyfikujesz domyślną konfigurację.

Kiedy zadanie jest wysyłane, jest umieszczane w kolejce. Worker następnie je pobiera i próbuje je wykonać. To jest próba zadania.

Jednak próba niekoniecznie oznacza, że metoda `handle` zadania została wykonana. Próby mogą być również "zużyte" na kilka sposobów:

<div class="content-list" markdown="1">

- Zadanie napotyka nieobsłużony wyjątek podczas wykonywania.
- Zadanie jest ręcznie zwalniane z powrotem do kolejki za pomocą `$this->release()`.
- Middleware takie jak `WithoutOverlapping` lub `RateLimited` nie udaje się uzyskać blokady i zwalnia zadanie.
- Zadanie przekroczyło limit czasu.
- Metoda `handle` zadania uruchamia się i kończy bez rzucania wyjątku.

</div>

Prawdopodobnie nie chcesz próbować wykonywać zadania w nieskończoność. Dlatego Laravel zapewnia różne sposoby określania, ile razy lub jak długo można próbować wykonać zadanie.

> [!NOTE]
> Domyślnie Laravel spróbuje wykonać zadanie tylko raz. Jeśli Twoje zadanie używa middleware takiego jak `WithoutOverlapping` lub `RateLimited`, lub jeśli ręcznie zwalniasz zadania, prawdopodobnie będziesz musiał zwiększyć liczbę dozwolonych prób za pomocą opcji `tries`.

Jednym ze sposobów określania maksymalnej liczby prób wykonania zadania jest przełącznik `--tries` w wierszu poleceń Artisan. Będzie to miało zastosowanie do wszystkich zadań przetwarzanych przez workera, chyba że przetwarzane zadanie określa liczbę prób, które może wykonać:

```shell
php artisan queue:work --tries=3
```

Jeśli zadanie przekroczy maksymalną liczbę prób, zostanie uznane za zadanie "nieudane". Aby uzyskać więcej informacji na temat obsługi nieudanych zadań, zapoznaj się z [dokumentacją nieudanych zadań](#dealing-with-failed-jobs). Jeśli do polecenia `queue:work` zostanie podane `--tries=0`, zadanie będzie ponawiane w nieskończoność.

Możesz przyjąć bardziej szczegółowe podejście, definiując maksymalną liczbę prób wykonania zadania w samej klasie zadania. Jeśli maksymalna liczba prób jest określona w zadaniu, będzie ona miała pierwszeństwo przed wartością `--tries` podaną w wierszu poleceń:

```php
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * The number of times the job may be attempted.
     *
     * @var int
     */
    public $tries = 5;
}
```

Jeśli potrzebujesz dynamicznej kontroli nad maksymalną liczbą prób danego zadania, możesz zdefiniować metodę `tries` w zadaniu:

```php
/**
 * Determine number of times the job may be attempted.
 */
public function tries(): int
{
    return 5;
}
```

<a name="time-based-attempts"></a>
#### Próby oparte na czasie

Jako alternatywę dla definiowania, ile razy można próbować wykonać zadanie, zanim się nie powiedzie, możesz zdefiniować czas, po którym nie należy już próbować wykonywać zadania. Pozwala to na wykonanie dowolnej liczby prób w danym przedziale czasu. Aby zdefiniować czas, po którym nie należy już próbować wykonywać zadania, dodaj metodę `retryUntil` do swojej klasy zadania. Ta metoda powinna zwrócić instancję `DateTime`:

```php
use DateTime;

/**
 * Determine the time at which the job should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->plus(minutes: 10);
}
```

Jeśli zdefiniowane są zarówno `retryUntil`, jak i `tries`, Laravel daje pierwszeństwo metodzie `retryUntil`.

> [!NOTE]
> Możesz również zdefiniować właściwość `tries` lub metodę `retryUntil` w swoich [kolejkowanych słuchaczach zdarzeń](/docs/{{version}}/events#queued-event-listeners) i [kolejkowanych powiadomieniach](/docs/{{version}}/notifications#queueing-notifications).

<a name="max-exceptions"></a>
#### Maksymalna liczba wyjątków

Czasami możesz chcieć określić, że zadanie może być próbowane wiele razy, ale powinno się nie powieść, jeśli ponowne próby są wyzwalane przez określoną liczbę nieobsłużonych wyjątków (w przeciwieństwie do bezpośredniego zwolnienia metodą `release`). Aby to osiągnąć, możesz zdefiniować właściwość `maxExceptions` w swojej klasie zadania:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Support\Facades\Redis;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * The number of times the job may be attempted.
     *
     * @var int
     */
    public $tries = 25;

    /**
     * The maximum number of unhandled exceptions to allow before failing.
     *
     * @var int
     */
    public $maxExceptions = 3;

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        Redis::throttle('key')->allow(10)->every(60)->then(function () {
            // Lock obtained, process the podcast...
        }, function () {
            // Unable to obtain lock...
            return $this->release(10);
        });
    }
}
```

W tym przykładzie zadanie jest zwalniane na dziesięć sekund, jeśli aplikacja nie może uzyskać blokady Redis i będzie kontynuować ponowne próby do 25 razy. Jednak zadanie zakończy się niepowodzeniem, jeśli zostaną zgłoszone trzy nieobsłużone wyjątki przez zadanie.

<a name="timeout"></a>
#### Limit czasu

Często wiesz w przybliżeniu, jak długo powinno zająć wykonanie kolejkowanych zadań. Z tego powodu Laravel pozwala określić wartość "timeout". Domyślnie wartość limitu czasu wynosi 60 sekund. Jeśli zadanie jest przetwarzane dłużej niż liczba sekund określona przez wartość limitu czasu, worker przetwarzający zadanie zakończy się z błędem. Zazwyczaj worker zostanie automatycznie ponownie uruchomiony przez [menedżer procesów skonfigurowany na serwerze](#supervisor-configuration).

Maksymalna liczba sekund, przez które mogą działać zadania, może być określona za pomocą przełącznika `--timeout` w wierszu poleceń Artisan:

```shell
php artisan queue:work --timeout=30
```

Jeśli zadanie przekroczy maksymalną liczbę prób, ciągle przekraczając limit czasu, zostanie oznaczone jako nieudane.

Możesz również zdefiniować maksymalną liczbę sekund, przez które zadanie powinno móc działać, w samej klasie zadania. Jeśli limit czasu jest określony w zadaniu, będzie on miał pierwszeństwo przed jakimkolwiek limitem czasu określonym w wierszu poleceń:

```php
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * The number of seconds the job can run before timing out.
     *
     * @var int
     */
    public $timeout = 120;
}
```

Czasami procesy blokujące IO, takie jak gniazda lub wychodzące połączenia HTTP, mogą nie respektować określonego limitu czasu. Dlatego, używając tych funkcji, zawsze powinieneś próbować określić limit czasu za pomocą ich interfejsów API. Na przykład, używając [Guzzle](https://docs.guzzlephp.org), zawsze powinieneś określić wartość limitu czasu połączenia i żądania.

> [!WARNING]
> Rozszerzenie PHP [PCNTL](https://www.php.net/manual/en/book.pcntl.php) musi być zainstalowane, aby określić limity czasu zadań. Ponadto wartość "timeout" zadania powinna zawsze być mniejsza niż jego wartość ["retry after"](#job-expiration). W przeciwnym razie zadanie może być ponowione, zanim faktycznie zakończy wykonywanie lub przekroczy limit czasu.

<a name="failing-on-timeout"></a>
#### Niepowodzenie po przekroczeniu limitu czasu

Jeśli chcesz wskazać, że zadanie powinno zostać oznaczone jako [nieudane](#dealing-with-failed-jobs) po przekroczeniu limitu czasu, możesz zdefiniować właściwość `$failOnTimeout` w klasie zadania:

```php
/**
 * Indicate if the job should be marked as failed on timeout.
 *
 * @var bool
 */
public $failOnTimeout = true;
```

> [!NOTE]
> Domyślnie, gdy zadanie przekroczy limit czasu, zużywa jedną próbę i jest zwalniane z powrotem do kolejki (jeśli ponowne próby są dozwolone). Jednak jeśli skonfigurujesz zadanie do niepowodzenia po przekroczeniu limitu czasu, nie będzie ono ponawiane, niezależnie od wartości ustawionej dla tries.

<a name="sqs-fifo-and-fair-queues"></a>
### SQS FIFO i sprawiedliwe kolejki

Laravel obsługuje kolejki [Amazon SQS FIFO (First-In-First-Out)](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-fifo-queues.html), umożliwiając przetwarzanie zadań w dokładnej kolejności, w jakiej zostały wysłane, przy jednoczesnym zapewnieniu przetwarzania dokładnie-jeden-raz poprzez deduplikację wiadomości.

Kolejki FIFO wymagają identyfikatora grupy wiadomości, aby określić, które zadania mogą być przetwarzane równolegle. Zadania z tym samym identyfikatorem grupy są przetwarzane sekwencyjnie, podczas gdy wiadomości z różnymi identyfikatorami grup mogą być przetwarzane jednocześnie.

Laravel zapewnia płynną metodę `onGroup` do określania identyfikatora grupy wiadomości podczas wysyłania zadań:

```php
ProcessOrder::dispatch($order)
    ->onGroup("customer-{$order->customer_id}");
```

Kolejki SQS FIFO obsługują deduplikację wiadomości, aby zapewnić przetwarzanie dokładnie-jeden-raz. Zaimplementuj metodę `deduplicationId` w swojej klasie zadania, aby zapewnić niestandardowy identyfikator deduplikacji:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessSubscriptionRenewal implements ShouldQueue
{
    use Queueable;

    // ...

    /**
     * Get the job's deduplication ID.
     */
    public function deduplicationId(): string
    {
        return "renewal-{$this->subscription->id}";
    }
}
```

<a name="fifo-listeners-mail-and-notifications"></a>
#### Słuchacze FIFO, poczta i powiadomienia

Korzystając z kolejek FIFO, będziesz również musiał zdefiniować grupy wiadomości dla słuchaczy, poczty i powiadomień. Alternatywnie możesz wysyłać kolejkowane instancje tych obiektów do kolejki nie-FIFO.

Aby zdefiniować grupę wiadomości dla [kolejkowanego słuchacza zdarzeń](/docs/{{version}}/events#queued-event-listeners), zdefiniuj metodę `messageGroup` w słuchaczu. Możesz również opcjonalnie zdefiniować metodę `deduplicationId`:

```php
<?php

namespace App\Listeners;

class SendShipmentNotification
{
    // ...

    /**
     * Get the job's message group.
     */
    public function messageGroup(): string
    {
        return 'shipments';
    }

    /**
     * Get the job's deduplication ID.
     */
    public function deduplicationId(): string
    {
        return "shipment-notification-{$this->shipment->id}";
    }
}
```

Podczas wysyłania [wiadomości e-mail](/docs/{{version}}/mail), która ma być kolejkowana w kolejce FIFO, powinieneś wywołać metodę `onGroup` i opcjonalnie metodę `withDeduplicator` podczas wysyłania powiadomienia:

```php
use App\Mail\InvoicePaid;
use Illuminate\Support\Facades\Mail;

$invoicePaid = (new InvoicePaid($invoice))
    ->onGroup('invoices')
    ->withDeduplicator(fn () => 'invoices-'.$invoice->id);

Mail::to($request->user())->send($invoicePaid);
```

Podczas wysyłania [powiadomienia](/docs/{{version}}/notifications), które ma być kolejkowane w kolejce FIFO, powinieneś wywołać metodę `onGroup` i opcjonalnie metodę `withDeduplicator` podczas wysyłania powiadomienia:

```php
use App\Notifications\InvoicePaid;

$invoicePaid = (new InvoicePaid($invoice))
    ->onGroup('invoices')
    ->withDeduplicator(fn () => 'invoices-'.$invoice->id);

$user->notify($invoicePaid);
```

<a name="queue-failover"></a>
### Przełączanie awaryjne kolejki

Sterownik kolejki `failover` zapewnia automatyczną funkcjonalność przełączania awaryjnego podczas umieszczania zadań w kolejce. Jeśli podstawowe połączenie kolejki konfiguracji `failover` nie powiedzie się z jakiegokolwiek powodu, Laravel automatycznie spróbuje umieścić zadanie w następnym skonfigurowanym połączeniu z listy. Jest to szczególnie przydatne do zapewnienia wysokiej dostępności w środowiskach produkcyjnych, gdzie niezawodność kolejki jest kluczowa.

Aby skonfigurować połączenie kolejki przełączania awaryjnego, określ sterownik `failover` i podaj tablicę nazw połączeń do próbowania w kolejności. Domyślnie Laravel zawiera przykładową konfigurację przełączania awaryjnego w pliku konfiguracyjnym `config/queue.php` Twojej aplikacji:

```php
'failover' => [
    'driver' => 'failover',
    'connections' => [
        'redis',
        'database',
        'sync',
    ],
],
```

Po skonfigurowaniu połączenia, które używa sterownika `failover`, będziesz musiał ustawić połączenie przełączania awaryjnego jako domyślne połączenie kolejki w pliku `.env` aplikacji, aby skorzystać z funkcjonalności przełączania awaryjnego:

```ini
QUEUE_CONNECTION=failover
```

Następnie uruchom co najmniej jednego workera dla każdego połączenia na liście połączeń przełączania awaryjnego:

```bash
php artisan queue:work redis
php artisan queue:work database
```

> [!NOTE]
> Nie musisz uruchamiać workera dla połączeń używających sterowników kolejki `sync`, `background` lub `deferred`, ponieważ te sterowniki przetwarzają zadania w bieżącym procesie PHP.

Gdy operacja połączenia kolejki nie powiedzie się i zostanie aktywowane przełączanie awaryjne, Laravel wyśle zdarzenie `Illuminate\Queue\Events\QueueFailedOver`, umożliwiając zgłoszenie lub zarejestrowanie, że połączenie kolejki nie powiodło się.

> [!NOTE]
> Jeśli używasz Laravel Horizon, pamiętaj, że Horizon zarządza tylko kolejkami Redis. Jeśli Twoja lista przełączania awaryjnego zawiera `database`, powinieneś uruchomić zwykły proces `php artisan queue:work database` obok Horizon.

<a name="error-handling"></a>
### Obsługa błędów

Jeśli wyjątek zostanie zgłoszony podczas przetwarzania zadania, zadanie zostanie automatycznie zwolnione z powrotem do kolejki, aby można było podjąć kolejną próbę. Zadanie będzie nadal zwalniane, dopóki nie zostanie podjęta maksymalna liczba prób dozwolona przez aplikację. Maksymalna liczba prób jest definiowana przez przełącznik `--tries` używany w poleceniu Artisan `queue:work`. Alternatywnie maksymalna liczba prób może być zdefiniowana w samej klasie zadania. Więcej informacji na temat uruchamiania workera kolejki [można znaleźć poniżej](#running-the-queue-worker).

<a name="manually-releasing-a-job"></a>
#### Ręczne zwalnianie zadania

Czasami możesz chcieć ręcznie zwolnić zadanie z powrotem do kolejki, aby można było podjąć kolejną próbę w późniejszym czasie. Możesz to osiągnąć, wywołując metodę `release`:

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    // ...

    $this->release();
}
```

Domyślnie metoda `release` zwolni zadanie z powrotem do kolejki do natychmiastowego przetworzenia. Jednak możesz poinstruować kolejkę, aby nie udostępniała zadania do przetworzenia, dopóki nie upłynie określona liczba sekund, przekazując liczbę całkowitą lub instancję daty do metody `release`:

```php
$this->release(10);

$this->release(now()->plus(seconds: 10));
```

<a name="manually-failing-a-job"></a>
#### Ręczne oznaczanie zadania jako nieudane

Czasami może być konieczne ręczne oznaczenie zadania jako "nieudane". Aby to zrobić, możesz wywołać metodę `fail`:

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    // ...

    $this->fail();
}
```

Jeśli chcesz oznaczyć swoje zadanie jako nieudane z powodu przechwytanego wyjątku, możesz przekazać wyjątek do metody `fail`. Lub, dla wygody, możesz przekazać komunikat błędu w postaci ciągu znaków, który zostanie przekonwertowany na wyjątek:

```php
$this->fail($exception);

$this->fail('Something went wrong.');
```

> [!NOTE]
> Aby uzyskać więcej informacji na temat nieudanych zadań, sprawdź [dokumentację dotyczącą obsługi nieudanych zadań](#dealing-with-failed-jobs).

<a name="fail-jobs-on-exceptions"></a>
#### Niepowodzenie zadań przy określonych wyjątkach

Middleware zadań `FailOnException` [job middleware](#job-middleware) pozwala na przerwanie ponownych prób, gdy zgłaszane są określone wyjątki. Pozwala to na ponowne próby w przypadku przejściowych wyjątków, takich jak błędy zewnętrznych API, ale trwałe niepowodzenie zadania w przypadku trwałych wyjątków, takich jak cofnięcie uprawnień użytkownika:

```php
<?php

namespace App\Jobs;

use App\Models\User;
use Illuminate\Auth\Access\AuthorizationException;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Queue\Middleware\FailOnException;
use Illuminate\Support\Facades\Http;

class SyncChatHistory implements ShouldQueue
{
    use Queueable;

    public $tries = 3;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public User $user,
    ) {}

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        $this->user->authorize('sync-chat-history');

        $response = Http::throw()->get(
            "https://chat.laravel.test/?user={$this->user->uuid}"
        );

        // ...
    }

    /**
     * Get the middleware the job should pass through.
     */
    public function middleware(): array
    {
        return [
            new FailOnException([AuthorizationException::class])
        ];
    }
}
```

<a name="job-batching"></a>
## Job Batching

Funkcja wsadowego przetwarzania zadań Laravel pozwala łatwo wykonać partię zadań, a następnie wykonać jakąś akcję po zakończeniu wykonywania partii zadań. Przed rozpoczęciem należy utworzyć migrację bazy danych, aby zbudować tabelę, która będzie zawierać metainformacje o Twoich partiach zadań, takie jak procent ukończenia. Ta migracja może zostać wygenerowana za pomocą polecenia Artisan `make:queue-batches-table`:

```shell
php artisan make:queue-batches-table

php artisan migrate
```

<a name="defining-batchable-jobs"></a>
### Defining Batchable Jobs

Aby zdefiniować zadanie wsadowe, powinieneś [utworzyć kolejkowane zadanie](#creating-jobs) jak zwykle; jednak powinieneś dodać cechę `Illuminate\Bus\Batchable` do klasy zadania. Ta cecha zapewnia dostęp do metody `batch`, która może być używana do pobrania bieżącej partii, w ramach której wykonywane jest zadanie:

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Batchable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ImportCsv implements ShouldQueue
{
    use Batchable, Queueable;

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        if ($this->batch()->cancelled()) {
            // Determine if the batch has been cancelled...

            return;
        }

        // Import a portion of the CSV file...
    }
}
```

<a name="dispatching-batches"></a>
### Dispatching Batches

Aby wysłać partię zadań, powinieneś użyć metody `batch` fasady `Bus`. Oczywiście wsadowe przetwarzanie jest szczególnie przydatne w połączeniu z wywołaniami zwrotnymi zakończenia. Możesz więc użyć metod `then`, `catch` i `finally`, aby zdefiniować wywołania zwrotne zakończenia dla partii. Każde z tych wywołań zwrotnych otrzyma instancję `Illuminate\Bus\Batch` po wywołaniu.

Podczas uruchamiania wielu workerów kolejki, zadania w partii będą przetwarzane równolegle. Dlatego kolejność, w jakiej zadania się kończą, może nie być taka sama jak kolejność, w jakiej zostały dodane do partii. Zapoznaj się z naszą dokumentacją dotyczącą [łańcuchów zadań i partii](#chains-and-batches), aby uzyskać informacje o tym, jak uruchomić serię zadań w sekwencji.

W tym przykładzie wyobrazimy sobie, że kolejkujemy partię zadań, z których każde przetwarza określoną liczbę wierszy z pliku CSV:

```php
use App\Jobs\ImportCsv;
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;
use Throwable;

$batch = Bus::batch([
    new ImportCsv(1, 100),
    new ImportCsv(101, 200),
    new ImportCsv(201, 300),
    new ImportCsv(301, 400),
    new ImportCsv(401, 500),
])->before(function (Batch $batch) {
    // The batch has been created but no jobs have been added...
})->progress(function (Batch $batch) {
    // A single job has completed successfully...
})->then(function (Batch $batch) {
    // All jobs completed successfully...
})->catch(function (Batch $batch, Throwable $e) {
    // First batch job failure detected...
})->finally(function (Batch $batch) {
    // The batch has finished executing...
})->dispatch();

return $batch->id;
```

Identyfikator partii, do którego można uzyskać dostęp za pomocą właściwości `$batch->id`, może być używany do [zapytania magistrali poleceń Laravel](#inspecting-batches) o informacje o partii po jej wysłaniu.

> [!WARNING]
> Ponieważ wywołania zwrotne partii są serializowane i wykonywane później przez kolejkę Laravel, nie powinieneś używać zmiennej `$this` w wywołaniach zwrotnych. Ponadto, ponieważ zadania wsadowe są opakowane w transakcje bazodanowe, instrukcje bazy danych, które wyzwalają niejawne zatwierdzenia, nie powinny być wykonywane w zadaniach.

<a name="naming-batches"></a>
#### Naming Batches

Niektóre narzędzia, takie jak [Laravel Horizon](/docs/{{version}}/horizon) i [Laravel Telescope](/docs/{{version}}/telescope), mogą dostarczać bardziej przyjazne dla użytkownika informacje debugowania dla partii, jeśli partie są nazwane. Aby przypisać dowolną nazwę partii, możesz wywołać metodę `name` podczas definiowania partii:

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->name('Import CSV')->dispatch();
```

<a name="batch-connection-queue"></a>
#### Batch Connection and Queue

Jeśli chcesz określić połączenie i kolejkę, które powinny być używane dla zadań wsadowych, możesz użyć metod `onConnection` i `onQueue`. Wszystkie zadania wsadowe muszą być wykonywane w tym samym połączeniu i kolejce:

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->onConnection('redis')->onQueue('imports')->dispatch();
```

<a name="chains-and-batches"></a>
### Chains and Batches

Możesz zdefiniować zestaw [połączonych zadań](#job-chaining) w partii, umieszczając połączone zadania w tablicy. Na przykład możemy wykonać dwa łańcuchy zadań równolegle i wykonać wywołanie zwrotne, gdy oba łańcuchy zadań zakończą przetwarzanie:

```php
use App\Jobs\ReleasePodcast;
use App\Jobs\SendPodcastReleaseNotification;
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;

Bus::batch([
    [
        new ReleasePodcast(1),
        new SendPodcastReleaseNotification(1),
    ],
    [
        new ReleasePodcast(2),
        new SendPodcastReleaseNotification(2),
    ],
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->dispatch();
```

Odwrotnie, możesz uruchamiać partie zadań w ramach [łańcucha](#job-chaining), definiując partie w łańcuchu. Na przykład możesz najpierw uruchomić partię zadań, aby wydać wiele podcastów, a następnie partię zadań do wysyłania powiadomień o wydaniu:

```php
use App\Jobs\FlushPodcastCache;
use App\Jobs\ReleasePodcast;
use App\Jobs\SendPodcastReleaseNotification;
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new FlushPodcastCache,
    Bus::batch([
        new ReleasePodcast(1),
        new ReleasePodcast(2),
    ]),
    Bus::batch([
        new SendPodcastReleaseNotification(1),
        new SendPodcastReleaseNotification(2),
    ]),
])->dispatch();
```

<a name="adding-jobs-to-batches"></a>
### Adding Jobs to Batches

Czasami może być przydatne dodanie dodatkowych zadań do partii z poziomu zadania wsadowego. Ten wzorzec może być przydatny, gdy musisz partii tysiące zadań, których wysłanie może zająć zbyt długo podczas żądania internetowego. Zamiast tego możesz wysłać początkową partię zadań "ładujących", które wypełnią partię jeszcze większą liczbą zadań:

```php
$batch = Bus::batch([
    new LoadImportBatch,
    new LoadImportBatch,
    new LoadImportBatch,
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->name('Import Contacts')->dispatch();
```

W tym przykładzie użyjemy zadania `LoadImportBatch`, aby wypełnić partię dodatkowymi zadaniami. Aby to osiągnąć, możemy użyć metody `add` na instancji partii, do której można uzyskać dostęp za pomocą metody `batch` zadania:

```php
use App\Jobs\ImportContacts;
use Illuminate\Support\Collection;

/**
 * Execute the job.
 */
public function handle(): void
{
    if ($this->batch()->cancelled()) {
        return;
    }

    $this->batch()->add(Collection::times(1000, function () {
        return new ImportContacts;
    }));
}
```

> [!WARNING]
> Możesz dodawać zadania do partii tylko z poziomu zadania, które należy do tej samej partii.

<a name="inspecting-batches"></a>
### Inspecting Batches

Instancja `Illuminate\Bus\Batch`, która jest dostarczana do wywołań zwrotnych zakończenia partii, ma różnorodne właściwości i metody, które pomogą Ci w interakcji i inspekcji danej partii zadań:

```php
// The UUID of the batch...
$batch->id;

// The name of the batch (if applicable)...
$batch->name;

// The number of jobs assigned to the batch...
$batch->totalJobs;

// The number of jobs that have not been processed by the queue...
$batch->pendingJobs;

// The number of jobs that have failed...
$batch->failedJobs;

// The number of jobs that have been processed thus far...
$batch->processedJobs();

// The completion percentage of the batch (0-100)...
$batch->progress();

// Indicates if the batch has finished executing...
$batch->finished();

// Cancel the execution of the batch...
$batch->cancel();

// Indicates if the batch has been cancelled...
$batch->cancelled();
```

<a name="returning-batches-from-routes"></a>
#### Returning Batches From Routes

Wszystkie instancje `Illuminate\Bus\Batch` są serializowalne do JSON, co oznacza, że możesz zwrócić je bezpośrednio z jednej z tras aplikacji, aby pobrać ładunek JSON zawierający informacje o partii, w tym postęp jej ukończenia. To ułatwia wyświetlanie informacji o postępie ukończenia partii w interfejsie użytkownika aplikacji.

Aby pobrać partię według jej identyfikatora, możesz użyć metody `findBatch` fasady `Bus`:

```php
use Illuminate\Support\Facades\Bus;
use Illuminate\Support\Facades\Route;

Route::get('/batch/{batchId}', function (string $batchId) {
    return Bus::findBatch($batchId);
});
```

<a name="cancelling-batches"></a>
### Cancelling Batches

Czasami może być konieczne anulowanie wykonania danej partii. Można to osiągnąć, wywołując metodę `cancel` na instancji `Illuminate\Bus\Batch`:

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    if ($this->user->exceedsImportLimit()) {
        $this->batch()->cancel();

        return;
    }

    if ($this->batch()->cancelled()) {
        return;
    }
}
```

Jak można zauważyć w poprzednich przykładach, zadania wsadowe powinny zazwyczaj określić, czy ich odpowiednia partia została anulowana, przed kontynuowaniem wykonywania. Jednak dla wygody możesz przypisać middleware `SkipIfBatchCancelled` [middleware](#job-middleware) do zadania. Jak wskazuje jego nazwa, to middleware poinstruuje Laravel, aby nie przetwarzał zadania, jeśli jego odpowiednia partia została anulowana:

```php
use Illuminate\Queue\Middleware\SkipIfBatchCancelled;

/**
 * Get the middleware the job should pass through.
 */
public function middleware(): array
{
    return [new SkipIfBatchCancelled];
}
```

<a name="batch-failures"></a>
### Batch Failures

Gdy zadanie wsadowe nie powiedzie się, wywołanie zwrotne `catch` (jeśli jest przypisane) zostanie wywołane. To wywołanie zwrotne jest wywoływane tylko dla pierwszego zadania, które nie powiedzie się w partii.

<a name="allowing-failures"></a>
#### Allowing Failures

Gdy zadanie w partii nie powiedzie się, Laravel automatycznie oznaczy partię jako "anulowana". Jeśli chcesz, możesz wyłączyć to zachowanie, aby niepowodzenie zadania nie oznaczało automatycznie partii jako anulowanej. Można to osiągnąć, wywołując metodę `allowFailures` podczas wysyłania partii:

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->allowFailures()->dispatch();
```

Możesz opcjonalnie dostarczyć domknięcie do metody `allowFailures`, które zostanie wykonane przy każdym niepowodzeniu zadania:

```php
$batch = Bus::batch([
    // ...
])->allowFailures(function (Batch $batch, $exception) {
    // Handle individual job failures...
})->dispatch();
```

<a name="retrying-failed-batch-jobs"></a>
#### Retrying Failed Batch Jobs

Dla wygody Laravel zapewnia polecenie Artisan `queue:retry-batch`, które pozwala łatwo ponowić wszystkie nieudane zadania dla danej partii. To polecenie przyjmuje UUID partii, której nieudane zadania powinny być ponowione:

```shell
php artisan queue:retry-batch 32dbc76c-4f82-4749-b610-a639fe0099b5
```

<a name="pruning-batches"></a>
### Czyszczenie Partii

Bez czyszczenia, tabela `job_batches` może bardzo szybko gromadzić rekordy. Aby temu zapobiec, powinieneś [zaplanować](/docs/{{version}}/scheduling) codzienne uruchamianie polecenia Artisan `queue:prune-batches`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches')->daily();
```

Domyślnie wszystkie zakończone partie starsze niż 24 godziny zostaną usunięte. Możesz użyć opcji `hours` podczas wywoływania polecenia, aby określić, jak długo przechowywać dane partii. Na przykład następujące polecenie usunie wszystkie partie, które zakończyły się ponad 48 godzin temu:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches --hours=48')->daily();
```

Czasami tabela `jobs_batches` może gromadzić rekordy partii dla partii, które nigdy nie zakończyły się pomyślnie, takich jak partie, w których zadanie nie powiodło się i to zadanie nigdy nie zostało ponowione z sukcesem. Możesz nakazać poleceniu `queue:prune-batches` usuwanie tych niedokończonych rekordów partii za pomocą opcji `unfinished`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches --hours=48 --unfinished=72')->daily();
```

Podobnie, tabela `jobs_batches` może również gromadzić rekordy partii dla anulowanych partii. Możesz nakazać poleceniu `queue:prune-batches` usuwanie tych anulowanych rekordów partii za pomocą opcji `cancelled`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches --hours=48 --cancelled=72')->daily();
```

<a name="storing-batches-in-dynamodb"></a>
### Przechowywanie Partii w DynamoDB

Laravel zapewnia również obsługę przechowywania metainformacji o partiach w [DynamoDB](https://aws.amazon.com/dynamodb) zamiast w relacyjnej bazie danych. Jednak będziesz musiał ręcznie utworzyć tabelę DynamoDB do przechowywania wszystkich rekordów partii.

Typowo, ta tabela powinna nazywać się `job_batches`, ale powinieneś nazwać tabelę na podstawie wartości konfiguracji `queue.batching.table` w pliku konfiguracyjnym `queue` twojej aplikacji.

<a name="dynamodb-batch-table-configuration"></a>
#### Konfiguracja Tabeli Partii DynamoDB

Tabela `job_batches` powinna mieć ciąg znaków jako klucz partycji podstawowej o nazwie `application` oraz ciąg znaków jako klucz sortowania podstawowego o nazwie `id`. Część `application` klucza będzie zawierać nazwę twojej aplikacji zdefiniowaną przez wartość konfiguracji `name` w pliku konfiguracyjnym `app` twojej aplikacji. Ponieważ nazwa aplikacji jest częścią klucza tabeli DynamoDB, możesz użyć tej samej tabeli do przechowywania partii zadań dla wielu aplikacji Laravel.

Ponadto możesz zdefiniować atrybut `ttl` dla swojej tabeli, jeśli chcesz skorzystać z [automatycznego czyszczenia partii](#pruning-batches-in-dynamodb).

<a name="dynamodb-configuration"></a>
#### Konfiguracja DynamoDB

Następnie zainstaluj AWS SDK, aby twoja aplikacja Laravel mogła komunikować się z Amazon DynamoDB:

```shell
composer require aws/aws-sdk-php
```

Następnie ustaw wartość opcji konfiguracji `queue.batching.driver` na `dynamodb`. Ponadto powinieneś zdefiniować opcje konfiguracji `key`, `secret` i `region` w tablicy konfiguracji `batching`. Te opcje będą używane do uwierzytelniania z AWS. Podczas korzystania ze sterownika `dynamodb`, opcja konfiguracji `queue.batching.database` jest niepotrzebna:

```php
'batching' => [
    'driver' => env('QUEUE_BATCHING_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'job_batches',
],
```

<a name="pruning-batches-in-dynamodb"></a>
#### Czyszczenie Partii w DynamoDB

Podczas korzystania z [DynamoDB](https://aws.amazon.com/dynamodb) do przechowywania informacji o partiach zadań, typowe polecenia czyszczące używane do usuwania partii przechowywanych w relacyjnej bazie danych nie będą działać. Zamiast tego możesz wykorzystać [natywną funkcjonalność TTL DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html), aby automatycznie usuwać rekordy starych partii.

Jeśli zdefiniowałeś swoją tabelę DynamoDB z atrybutem `ttl`, możesz zdefiniować parametry konfiguracji, aby poinstruować Laravel, jak czyścić rekordy partii. Wartość konfiguracji `queue.batching.ttl_attribute` definiuje nazwę atrybutu przechowującego TTL, podczas gdy wartość konfiguracji `queue.batching.ttl` definiuje liczbę sekund, po których rekord partii może zostać usunięty z tabeli DynamoDB, względem ostatniego czasu aktualizacji rekordu:

```php
'batching' => [
    'driver' => env('QUEUE_FAILED_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'job_batches',
    'ttl_attribute' => 'ttl',
    'ttl' => 60 * 60 * 24 * 7, // 7 days...
],
```

<a name="queueing-closures"></a>
## Kolejkowanie Zamknięć

Zamiast wysyłać klasę zadania do kolejki, możesz również wysłać zamknięcie. Jest to świetne dla szybkich, prostych zadań, które muszą być wykonane poza bieżącym cyklem żądania. Podczas wysyłania zamknięć do kolejki, zawartość kodu zamknięcia jest kryptograficznie podpisana, aby nie mogła być modyfikowana w tranzycie:

```php
use App\Models\Podcast;

$podcast = Podcast::find(1);

dispatch(function () use ($podcast) {
    $podcast->publish();
});
```

Aby przypisać nazwę do zamknięcia w kolejce, która może być używana przez pulpity raportowania kolejki, a także wyświetlana przez polecenie `queue:work`, możesz użyć metody `name`:

```php
dispatch(function () {
    // ...
})->name('Publish Podcast');
```

Korzystając z metody `catch`, możesz dostarczyć zamknięcie, które powinno być wykonane, jeśli zamknięcie w kolejce nie uda się zakończyć pomyślnie po wyczerpaniu wszystkich [skonfigurowanych prób ponowienia](#max-job-attempts-and-timeout) twojej kolejki:

```php
use Throwable;

dispatch(function () use ($podcast) {
    $podcast->publish();
})->catch(function (Throwable $e) {
    // This job has failed...
});
```

> [!WARNING]
> Since `catch` callbacks are serialized and executed at a later time by the Laravel queue, you should not use the `$this` variable within `catch` callbacks.

<a name="running-the-queue-worker"></a>
## Uruchamianie Workera Kolejki

<a name="the-queue-work-command"></a>
### Polecenie `queue:work`

Laravel zawiera polecenie Artisan, które uruchomi workera kolejki i przetworzy nowe zadania, gdy są one dodawane do kolejki. Możesz uruchomić workera za pomocą polecenia Artisan `queue:work`. Zauważ, że po uruchomieniu polecenia `queue:work` będzie ono działać, dopóki nie zostanie ręcznie zatrzymane lub nie zamkniesz terminala:

```shell
php artisan queue:work
```

> [!NOTE]
> To keep the `queue:work` process running permanently in the background, you should use a process monitor such as [Supervisor](#supervisor-configuration) to ensure that the queue worker does not stop running.

Możesz dołączyć flagę `-v` podczas wywoływania polecenia `queue:work`, jeśli chcesz, aby przetworzone ID zadań, nazwy połączeń i nazwy kolejek były zawarte w wyjściu polecenia:

```shell
php artisan queue:work -v
```

Pamiętaj, że workery kolejek to długotrwałe procesy, które przechowują stan uruchomionej aplikacji w pamięci. W rezultacie nie zauważą zmian w twojej bazie kodu po ich uruchomieniu. Więc podczas procesu wdrażania upewnij się, że [restartujesz swoje workery kolejek](#queue-workers-and-deployment). Ponadto pamiętaj, że każdy stan statyczny utworzony lub zmodyfikowany przez twoją aplikację nie zostanie automatycznie zresetowany między zadaniami.

Alternatywnie możesz uruchomić polecenie `queue:listen`. Podczas korzystania z polecenia `queue:listen` nie musisz ręcznie restartować workera, gdy chcesz przeładować zaktualizowany kod lub zresetować stan aplikacji; jednak to polecenie jest znacznie mniej wydajne niż polecenie `queue:work`:

```shell
php artisan queue:listen
```

<a name="running-multiple-queue-workers"></a>
#### Uruchamianie Wielu Workerów Kolejki

Aby przypisać wielu workerów do kolejki i przetwarzać zadania jednocześnie, powinieneś po prostu uruchomić wiele procesów `queue:work`. Można to zrobić lokalnie poprzez wiele zakładek w terminalu lub w produkcji za pomocą ustawień konfiguracji menedżera procesów. [Podczas korzystania z Supervisora](#supervisor-configuration), możesz użyć wartości konfiguracji `numprocs`.

<a name="specifying-the-connection-queue"></a>
#### Określanie Połączenia i Kolejki

Możesz również określić, którego połączenia kolejki powinien używać worker. Nazwa połączenia przekazana do polecenia `work` powinna odpowiadać jednemu z połączeń zdefiniowanych w pliku konfiguracyjnym `config/queue.php`:

```shell
php artisan queue:work redis
```

Domyślnie polecenie `queue:work` przetwarza tylko zadania dla domyślnej kolejki w danym połączeniu. Jednak możesz jeszcze bardziej dostosować swojego workera kolejki, przetwarzając tylko określone kolejki dla danego połączenia. Na przykład, jeśli wszystkie twoje e-maile są przetwarzane w kolejce `emails` na połączeniu kolejki `redis`, możesz wydać następujące polecenie, aby uruchomić workera, który przetwarza tylko tę kolejkę:

```shell
php artisan queue:work redis --queue=emails
```

<a name="processing-a-specified-number-of-jobs"></a>
#### Przetwarzanie Określonej Liczby Zadań

Opcja `--once` może być użyta do poinstruowania workera, aby przetworzył tylko jedno zadanie z kolejki:

```shell
php artisan queue:work --once
```

Opcja `--max-jobs` może być użyta do poinstruowania workera, aby przetworzył określoną liczbę zadań, a następnie zakończył działanie. Ta opcja może być użyteczna w połączeniu z [Supervisorem](#supervisor-configuration), aby twoi workerzy byli automatycznie restartowani po przetworzeniu określonej liczby zadań, zwalniając pamięć, którą mogli zgromadzić:

```shell
php artisan queue:work --max-jobs=1000
```

<a name="processing-all-queued-jobs-then-exiting"></a>
#### Przetwarzanie Wszystkich Zadań w Kolejce, a Następnie Wyjście

Opcja `--stop-when-empty` może być użyta do poinstruowania workera, aby przetworzył wszystkie zadania, a następnie zakończył działanie w sposób płynny. Ta opcja może być użyteczna podczas przetwarzania kolejek Laravel w kontenerze Docker, jeśli chcesz zamknąć kontener po opróżnieniu kolejki:

```shell
php artisan queue:work --stop-when-empty
```

<a name="processing-jobs-for-a-given-number-of-seconds"></a>
#### Przetwarzanie Zadań Przez Określoną Liczbę Sekund

Opcja `--max-time` może być użyta do poinstruowania workera, aby przetwarzał zadania przez określoną liczbę sekund, a następnie zakończył działanie. Ta opcja może być użyteczna w połączeniu z [Supervisorem](#supervisor-configuration), aby twoi workerzy byli automatycznie restartowani po przetworzeniu zadań przez określony czas, zwalniając pamięć, którą mogli zgromadzić:

```shell
# Process jobs for one hour and then exit...
php artisan queue:work --max-time=3600
```

<a name="worker-sleep-duration"></a>
#### Czas Uśpienia Workera

Kiedy zadania są dostępne w kolejce, worker będzie kontynuował przetwarzanie zadań bez opóźnień między zadaniami. Jednak opcja `sleep` określa, ile sekund worker będzie "spał", jeśli nie ma dostępnych zadań. Oczywiście podczas spania worker nie będzie przetwarzał żadnych nowych zadań:

```shell
php artisan queue:work --sleep=3
```

<a name="maintenance-mode-queues"></a>
#### Tryb Konserwacji i Kolejki

Podczas gdy twoja aplikacja jest w [trybie konserwacji](/docs/{{version}}/configuration#maintenance-mode), żadne zadania w kolejce nie będą obsługiwane. Zadania będą nadal obsługiwane normalnie, gdy aplikacja wyjdzie z trybu konserwacji.

Aby zmusić workerów kolejki do przetwarzania zadań, nawet jeśli tryb konserwacji jest włączony, możesz użyć opcji `--force`:

```shell
php artisan queue:work --force
```

<a name="resource-considerations"></a>
#### Rozważania dotyczące Zasobów

Workerzy kolejki działający jako demony nie "restartują" frameworka przed przetworzeniem każdego zadania. Dlatego powinieneś zwolnić wszelkie ciężkie zasoby po zakończeniu każdego zadania. Na przykład, jeśli wykonujesz manipulację obrazem za pomocą [biblioteki GD](https://www.php.net/manual/en/book.image.php), powinieneś zwolnić pamięć za pomocą `imagedestroy`, gdy skończysz przetwarzanie obrazu.

<a name="queue-priorities"></a>
### Priorytety Kolejek

Czasami możesz chcieć nadać priorytet sposobowi przetwarzania kolejek. Na przykład w pliku konfiguracyjnym `config/queue.php` możesz ustawić domyślną `queue` dla połączenia `redis` na `low`. Jednak czasami możesz chcieć wysłać zadanie do kolejki o `high` priorytecie w taki sposób:

```php
dispatch((new Job)->onQueue('high'));
```

Aby uruchomić workera, który sprawdza, czy wszystkie zadania z kolejki `high` są przetwarzane przed kontynuacją jakichkolwiek zadań z kolejki `low`, przekaż oddzieloną przecinkami listę nazw kolejek do polecenia `work`:

```shell
php artisan queue:work --queue=high,low
```

<a name="queue-workers-and-deployment"></a>
### Workerzy Kolejki i Wdrażanie

Ponieważ workerzy kolejki to długotrwałe procesy, nie zauważą zmian w twoim kodzie bez ponownego uruchomienia. Więc najprostszym sposobem na wdrożenie aplikacji korzystającej z workerów kolejki jest restart workerów podczas procesu wdrażania. Możesz płynnie zrestartować wszystkich workerów, wydając polecenie `queue:restart`:

```shell
php artisan queue:restart
```

To polecenie nakaże wszystkim workerom kolejki płynne wyjście po zakończeniu przetwarzania ich bieżącego zadania, aby żadne istniejące zadania nie zostały utracone. Ponieważ workerzy kolejki zakończą działanie, gdy zostanie wykonane polecenie `queue:restart`, powinieneś uruchomić menedżera procesów, takiego jak [Supervisor](#supervisor-configuration), aby automatycznie restartować workerów kolejki.

> [!NOTE]
> Kolejka używa [pamięci podręcznej](/docs/{{version}}/cache) do przechowywania sygnałów restartu, więc powinieneś sprawdzić, czy sterownik pamięci podręcznej jest poprawnie skonfigurowany dla twojej aplikacji przed użyciem tej funkcji.

<a name="job-expirations-and-timeouts"></a>
### Wygasanie Zadań i Limity Czasu

<a name="job-expiration"></a>
#### Wygasanie Zadań

W pliku konfiguracyjnym `config/queue.php` każde połączenie kolejki definiuje opcję `retry_after`. Ta opcja określa, ile sekund połączenie kolejki powinno czekać przed ponowną próbą zadania, które jest przetwarzane. Na przykład, jeśli wartość `retry_after` jest ustawiona na `90`, zadanie zostanie zwrócone do kolejki, jeśli było przetwarzane przez 90 sekund bez zwolnienia lub usunięcia. Typowo powinieneś ustawić wartość `retry_after` na maksymalną liczbę sekund, jaką twoje zadania powinny rozsądnie zająć na zakończenie przetwarzania.

> [!WARNING]
> The only queue connection which does not contain a `retry_after` value is Amazon SQS. SQS will retry the job based on the [Default Visibility Timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html) which is managed within the AWS console.

<a name="worker-timeouts"></a>
#### Limity Czasu Workera

Polecenie Artisan `queue:work` udostępnia opcję `--timeout`. Domyślnie wartość `--timeout` wynosi 60 sekund. Jeśli zadanie jest przetwarzane dłużej niż liczba sekund określona przez wartość limitu czasu, worker przetwarzający zadanie zakończy działanie z błędem. Typowo worker zostanie automatycznie zrestartowany przez [menedżera procesów skonfigurowanego na twoim serwerze](#supervisor-configuration):

```shell
php artisan queue:work --timeout=60
```

Opcja konfiguracji `retry_after` i opcja CLI `--timeout` są różne, ale działają razem, aby zapewnić, że zadania nie zostaną utracone i że zadania są pomyślnie przetwarzane tylko raz.

> [!WARNING]
> Wartość `--timeout` powinna być zawsze co najmniej kilka sekund krótsza niż wartość konfiguracji `retry_after`. Zapewni to, że worker przetwarzający zamrożone zadanie jest zawsze kończony przed ponowną próbą zadania. Jeśli twoja opcja `--timeout` jest dłuższa niż wartość konfiguracji `retry_after`, twoje zadania mogą być przetwarzane dwukrotnie.

<a name="pausing-and-resuming-queue-workers"></a>
### Wstrzymywanie i Wznawianie Workerów Kolejki

Czasami możesz potrzebować tymczasowo zapobiec przetwarzaniu nowych zadań przez workera kolejki bez całkowitego zatrzymywania workera. Na przykład możesz chcieć wstrzymać przetwarzanie zadań podczas konserwacji systemu. Laravel udostępnia polecenia Artisan `queue:pause` i `queue:continue` do wstrzymywania i wznawiania workerów kolejki.

Aby wstrzymać określoną kolejkę, podaj nazwę połączenia kolejki i nazwę kolejki:

```shell
php artisan queue:pause database:default
```

W tym przykładzie `database` to nazwa połączenia kolejki, a `default` to nazwa kolejki. Po wstrzymaniu kolejki, wszelcy workerzy przetwarzający zadania z tej kolejki będą kontynuować kończenie swoich bieżących zadań, ale nie podejmą żadnych nowych zadań, dopóki kolejka nie zostanie wznowiona.

Aby wznowić przetwarzanie zadań w wstrzymanej kolejce, użyj polecenia `queue:continue`:

```shell
php artisan queue:continue database:default
```

Po wznowieniu kolejki workerzy natychmiast zaczną przetwarzać nowe zadania z tej kolejki. Zauważ, że wstrzymanie kolejki nie zatrzymuje samego procesu workera - tylko zapobiega przetwarzaniu nowych zadań z określonej kolejki przez workera.

<a name="worker-restart-and-pause-signals"></a>
#### Sygnały Restartu i Wstrzymania Workera

Domyślnie workerzy kolejki sprawdzają sterownik pamięci podręcznej pod kątem sygnałów restartu i wstrzymania przy każdej iteracji zadania. Choć to sprawdzanie jest niezbędne do reagowania na polecenia `queue:restart` i `queue:pause`, wprowadza niewielki narzut wydajności.

Jeśli potrzebujesz zoptymalizować wydajność i nie potrzebujesz tych funkcji przerywania, możesz wyłączyć to sprawdzanie globalnie, wywołując metodę `withoutInterruptionPolling` na fasadzie `Queue`. Powinno to zazwyczaj być wykonane w metodzie `boot` twojego `AppServiceProvider`:

```php
use Illuminate\Support\Facades\Queue;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Queue::withoutInterruptionPolling();
}
```

Alternatywnie możesz wyłączyć sprawdzanie restartu lub wstrzymania indywidualnie, ustawiając statyczne właściwości `$restartable` lub `$pausable` w klasie `Illuminate\Queue\Worker`:

```php
use Illuminate\Queue\Worker;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Worker::$restartable = false;
    Worker::$pausable = false;
}
```

> [!WARNING]
> Gdy sprawdzanie przerwań jest wyłączone, workerzy nie będą reagować na polecenia `queue:restart` lub `queue:pause` (w zależności od tego, które funkcje są wyłączone).

<a name="supervisor-configuration"></a>
## Konfiguracja Supervisora

W produkcji potrzebujesz sposobu, aby utrzymać działanie procesów `queue:work`. Proces `queue:work` może przestać działać z różnych powodów, takich jak przekroczenie limitu czasu workera lub wykonanie polecenia `queue:restart`.

Z tego powodu musisz skonfigurować monitor procesów, który może wykryć, gdy twoje procesy `queue:work` zakończą działanie i automatycznie je zrestartować. Ponadto monitory procesów mogą pozwolić ci określić, ile procesów `queue:work` chcesz uruchomić jednocześnie. Supervisor to monitor procesów powszechnie używany w środowiskach Linux i omówimy, jak go skonfigurować w poniższej dokumentacji.

<a name="installing-supervisor"></a>
#### Instalacja Supervisora

Supervisor to monitor procesów dla systemu operacyjnego Linux, który automatycznie zrestartuje twoje procesy `queue:work`, jeśli zawiodą. Aby zainstalować Supervisora na Ubuntu, możesz użyć następującego polecenia:

```shell
sudo apt-get install supervisor
```

> [!NOTE]
> Jeśli samodzielne konfigurowanie i zarządzanie Supervisorem wydaje się przytłaczające, rozważ użycie [Laravel Cloud](https://cloud.laravel.com), który zapewnia w pełni zarządzaną platformę do uruchamiania workerów kolejki Laravel.

<a name="configuring-supervisor"></a>
#### Konfiguracja Supervisora

Pliki konfiguracyjne Supervisora są zazwyczaj przechowywane w katalogu `/etc/supervisor/conf.d`. W tym katalogu możesz utworzyć dowolną liczbę plików konfiguracyjnych, które instruują supervisora, jak twoje procesy powinny być monitorowane. Na przykład utwórzmy plik `laravel-worker.conf`, który uruchamia i monitoruje procesy `queue:work`:

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
stopwaitsecs=3600
```

W tym przykładzie dyrektywa `numprocs` nakaże Supervisorowi uruchomienie ośmiu procesów `queue:work` i monitorowanie ich wszystkich, automatycznie restartując je, jeśli zawiodą. Powinieneś zmienić dyrektywę `command` konfiguracji, aby odzwierciedlać żądane połączenie kolejki i opcje workera.

> [!WARNING]
> Powinieneś upewnić się, że wartość `stopwaitsecs` jest większa niż liczba sekund zużywana przez twoje najdłużej działające zadanie. W przeciwnym razie Supervisor może zabić zadanie przed zakończeniem przetwarzania.

<a name="starting-supervisor"></a>
#### Uruchamianie Supervisora

Po utworzeniu pliku konfiguracyjnego możesz zaktualizować konfigurację Supervisora i uruchomić procesy za pomocą następujących poleceń:

```shell
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start "laravel-worker:*"
```

Aby uzyskać więcej informacji na temat Supervisora, zapoznaj się z [dokumentacją Supervisora](http://supervisord.org/index.html).

<a name="dealing-with-failed-jobs"></a>
## Obsługa Nieudanych Zadań

Czasami twoje zadania w kolejce zawiodą. Nie martw się, rzeczy nie zawsze idą zgodnie z planem! Laravel zawiera wygodny sposób [określania maksymalnej liczby prób wykonania zadania](#max-job-attempts-and-timeout). Po przekroczeniu tej liczby prób przez zadanie asynchroniczne zostanie ono wstawione do tabeli bazy danych `failed_jobs`. [Zadania wysyłane synchronicznie](/docs/{{version}}/queues#synchronous-dispatching), które zawiodą, nie są przechowywane w tej tabeli, a ich wyjątki są natychmiast obsługiwane przez aplikację.

Migracja do utworzenia tabeli `failed_jobs` jest zazwyczaj już obecna w nowych aplikacjach Laravel. Jednak jeśli twoja aplikacja nie zawiera migracji dla tej tabeli, możesz użyć polecenia `make:queue-failed-table`, aby utworzyć migrację:

```shell
php artisan make:queue-failed-table

php artisan migrate
```

Podczas uruchamiania procesu [workera kolejki](#running-the-queue-worker) możesz określić maksymalną liczbę prób wykonania zadania za pomocą przełącznika `--tries` w poleceniu `queue:work`. Jeśli nie określisz wartości dla opcji `--tries`, zadania będą próbowane tylko raz lub tyle razy, ile określono we właściwości `$tries` klasy zadania:

```shell
php artisan queue:work redis --tries=3
```

Używając opcji `--backoff`, możesz określić, ile sekund Laravel powinien poczekać przed ponowną próbą zadania, które napotkało wyjątek. Domyślnie zadanie jest natychmiast zwracane do kolejki, aby można było ponownie go spróbować:

```shell
php artisan queue:work redis --tries=3 --backoff=3
```

Jeśli chcesz skonfigurować, ile sekund Laravel powinien poczekać przed ponowną próbą zadania, które napotkało wyjątek, dla każdego zadania osobno, możesz to zrobić, definiując właściwość `backoff` w swojej klasie zadania:

```php
/**
 * The number of seconds to wait before retrying the job.
 *
 * @var int
 */
public $backoff = 3;
```

Jeśli potrzebujesz bardziej złożonej logiki do określania czasu backoff zadania, możesz zdefiniować metodę `backoff` w swojej klasie zadania:

```php
/**
 * Calculate the number of seconds to wait before retrying the job.
 */
public function backoff(): int
{
    return 3;
}
```

Możesz łatwo skonfigurować "wykładnicze" backoff, zwracając tablicę wartości backoff z metody `backoff`. W tym przykładzie opóźnienie ponowienia będzie wynosiło 1 sekundę dla pierwszej próby, 5 sekund dla drugiej próby, 10 sekund dla trzeciej próby i 10 sekund dla każdej kolejnej próby, jeśli pozostały więcej prób:

```php
/**
 * Calculate the number of seconds to wait before retrying the job.
 *
 * @return array<int, int>
 */
public function backoff(): array
{
    return [1, 5, 10];
}
```

<a name="cleaning-up-after-failed-jobs"></a>
### Sprzątanie Po Nieudanych Zadaniach

Kiedy określone zadanie zawiedzie, możesz chcieć wysłać alert do swoich użytkowników lub odwrócić wszelkie akcje, które zostały częściowo wykonane przez zadanie. Aby to osiągnąć, możesz zdefiniować metodę `failed` w swojej klasie zadania. Instancja `Throwable`, która spowodowała niepowodzenie zadania, zostanie przekazana do metody `failed`:

```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Throwable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Podcast $podcast,
    ) {}

    /**
     * Execute the job.
     */
    public function handle(AudioProcessor $processor): void
    {
        // Process uploaded podcast...
    }

    /**
     * Handle a job failure.
     */
    public function failed(?Throwable $exception): void
    {
        // Send user notification of failure, etc...
    }
}
```

> [!WARNING]
> Nowa instancja zadania jest tworzona przed wywołaniem metody `failed`; dlatego wszelkie modyfikacje właściwości klasy, które mogły wystąpić w metodzie `handle`, zostaną utracone.

Nieudane zadanie to niekoniecznie takie, które napotkało nieobsłużony wyjątek. Zadanie może być również uznane za nieudane, gdy wyczerpie wszystkie dozwolone próby. Te próby mogą zostać zużyte na kilka sposobów:

<div class="content-list" markdown="1">

- Zadanie przekroczyło limit czasu.
- Zadanie napotkało nieobsłużony wyjątek podczas wykonywania.
- Zadanie zostało zwolnione z powrotem do kolejki ręcznie lub przez middleware.

</div>

Jeśli ostatnia próba zawiedzie z powodu wyjątku rzuconego podczas wykonywania zadania, ten wyjątek zostanie przekazany do metody `failed` zadania. Jednak jeśli zadanie zawiedzie, ponieważ osiągnęło maksymalną liczbę dozwolonych prób, `$exception` będzie instancją `Illuminate\Queue\MaxAttemptsExceededException`. Podobnie, jeśli zadanie zawiedzie z powodu przekroczenia skonfigurowanego limitu czasu, `$exception` będzie instancją `Illuminate\Queue\TimeoutExceededException`.

<a name="retrying-failed-jobs"></a>
### Ponowne Próbowanie Nieudanych Zadań

Aby zobaczyć wszystkie nieudane zadania, które zostały wstawione do twojej tabeli bazy danych `failed_jobs`, możesz użyć polecenia Artisan `queue:failed`:

```shell
php artisan queue:failed
```

Polecenie `queue:failed` wyszczególni ID zadania, połączenie, kolejkę, czas niepowodzenia i inne informacje o zadaniu. ID zadania może być użyte do ponowienia nieudanego zadania. Na przykład, aby ponowić nieudane zadanie o ID `ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece`, wydaj następujące polecenie:

```shell
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece
```

Jeśli to konieczne, możesz przekazać wiele ID do polecenia:

```shell
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece 91401d2c-0784-4f43-824c-34f94a33c24d
```

Możesz również ponowić wszystkie nieudane zadania dla określonej kolejki:

```shell
php artisan queue:retry --queue=name
```

Aby ponowić wszystkie nieudane zadania, wykonaj polecenie `queue:retry` i przekaż `all` jako ID:

```shell
php artisan queue:retry all
```

Jeśli chcesz usunąć nieudane zadanie, możesz użyć polecenia `queue:forget`:

```shell
php artisan queue:forget 91401d2c-0784-4f43-824c-34f94a33c24d
```

> [!NOTE]
> Podczas korzystania z [Horizon](/docs/{{version}}/horizon), powinieneś użyć polecenia `horizon:forget`, aby usunąć nieudane zadanie zamiast polecenia `queue:forget`.

Aby usunąć wszystkie nieudane zadania z tabeli `failed_jobs`, możesz użyć polecenia `queue:flush`:

```shell
php artisan queue:flush
```

Polecenie `queue:flush` usuwa wszystkie rekordy nieudanych zadań z twojej kolejki, bez względu na to, jak stare jest nieudane zadanie. Możesz użyć opcji `--hours`, aby usunąć tylko zadania, które zawiodły określoną liczbę godzin temu lub wcześniej:

```shell
php artisan queue:flush --hours=48
```

<a name="ignoring-missing-models"></a>
### Ignorowanie Brakujących Modeli

Podczas wstrzykiwania modelu Eloquent do zadania, model jest automatycznie serializowany przed umieszczeniem w kolejce i ponownie pobierany z bazy danych, gdy zadanie jest przetwarzane. Jednak jeśli model został usunięty podczas oczekiwania zadania na przetworzenie przez workera, twoje zadanie może zawięść z `ModelNotFoundException`.

Dla wygody możesz wybrać automatyczne usuwanie zadań z brakującymi modelami, ustawiając właściwość `deleteWhenMissingModels` swojego zadania na `true`. Gdy ta właściwość jest ustawiona na `true`, Laravel po cichu odrzuci zadanie bez rzucania wyjątku:

```php
/**
 * Delete the job if its models no longer exist.
 *
 * @var bool
 */
public $deleteWhenMissingModels = true;
```

<a name="pruning-failed-jobs"></a>
### Czyszczenie Nieudanych Zadań

Możesz oczyścić rekordy w tabeli `failed_jobs` swojej aplikacji, wywołując polecenie Artisan `queue:prune-failed`:

```shell
php artisan queue:prune-failed
```

Domyślnie wszystkie rekordy nieudanych zadań starsze niż 24 godziny zostaną usunięte. Jeśli podasz opcję `--hours` do polecenia, tylko rekordy nieudanych zadań, które zostały wstawione w ciągu ostatnich N godzin, zostaną zachowane. Na przykład następujące polecenie usunie wszystkie rekordy nieudanych zadań, które zostały wstawione ponad 48 godzin temu:

```shell
php artisan queue:prune-failed --hours=48
```

<a name="storing-failed-jobs-in-dynamodb"></a>
### Przechowywanie Nieudanych Zadań w DynamoDB

Laravel zapewnia również obsługę przechowywania rekordów nieudanych zadań w [DynamoDB](https://aws.amazon.com/dynamodb) zamiast w tabeli relacyjnej bazy danych. Jednak musisz ręcznie utworzyć tabelę DynamoDB do przechowywania wszystkich rekordów nieudanych zadań. Typowo, ta tabela powinna nazywać się `failed_jobs`, ale powinieneś nazwać tabelę na podstawie wartości konfiguracji `queue.failed.table` w pliku konfiguracyjnym `queue` twojej aplikacji.

Tabela `failed_jobs` powinna mieć ciąg znaków jako klucz partycji podstawowej o nazwie `application` oraz ciąg znaków jako klucz sortowania podstawowego o nazwie `uuid`. Część `application` klucza będzie zawierać nazwę twojej aplikacji zdefiniowaną przez wartość konfiguracji `name` w pliku konfiguracyjnym `app` twojej aplikacji. Ponieważ nazwa aplikacji jest częścią klucza tabeli DynamoDB, możesz użyć tej samej tabeli do przechowywania nieudanych zadań dla wielu aplikacji Laravel.

Ponadto upewnij się, że zainstalujesz AWS SDK, aby twoja aplikacja Laravel mogła komunikować się z Amazon DynamoDB:

```shell
composer require aws/aws-sdk-php
```

Następnie ustaw wartość opcji konfiguracji `queue.failed.driver` na `dynamodb`. Ponadto powinieneś zdefiniować opcje konfiguracji `key`, `secret` i `region` w tablicy konfiguracji nieudanych zadań. Te opcje będą używane do uwierzytelniania z AWS. Podczas korzystania ze sterownika `dynamodb`, opcja konfiguracji `queue.failed.database` jest niepotrzebna:

```php
'failed' => [
    'driver' => env('QUEUE_FAILED_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'failed_jobs',
],
```

<a name="disabling-failed-job-storage"></a>
### Wyłączanie Przechowywania Nieudanych Zadań

Możesz nakazać Laravel odrzucanie nieudanych zadań bez ich przechowywania, ustawiając wartość opcji konfiguracji `queue.failed.driver` na `null`. Typowo może to być zrealizowane przez zmienną środowiskową `QUEUE_FAILED_DRIVER`:

```ini
QUEUE_FAILED_DRIVER=null
```

<a name="failed-job-events"></a>
### Zdarzenia Nieudanych Zadań

Jeśli chcesz zarejestrować listener zdarzeń, który zostanie wywołany, gdy zadanie zawiedzie, mośesz użyć metody `failing` fasady `Queue`. Na przykład możemy dołączyć closure do tego zdarzenia z metody `boot` `AppServiceProvider`, który jest dołączony do Laravel:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Queue;
use Illuminate\Support\ServiceProvider;
use Illuminate\Queue\Events\JobFailed;

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
        Queue::failing(function (JobFailed $event) {
            // $event->connectionName
            // $event->job
            // $event->exception
        });
    }
}
```

<a name="clearing-jobs-from-queues"></a>
## Czyszczenie Zadań z Kolejek

> [!NOTE]
> Podczas korzystania z [Horizon](/docs/{{version}}/horizon), powinieneś użyć polecenia `horizon:clear`, aby czyścić zadania z kolejki zamiast polecenia `queue:clear`.

Jeśli chcesz usunąć wszystkie zadania z domyślnej kolejki domyślnego połączenia, możesz to zrobić za pomocą polecenia Artisan `queue:clear`:

```shell
php artisan queue:clear
```

Możesz również podać argument `connection` i opcję `queue`, aby usunąć zadania z określonego połączenia i kolejki:

```shell
php artisan queue:clear redis --queue=emails
```

> [!WARNING]
> Czyszczenie zadań z kolejek jest dostępne tylko dla sterowników kolejki SQS, Redis i bazy danych. Ponadto proces usuwania komunikatów SQS trwa do 60 sekund, więc zadania wysłane do kolejki SQS do 60 sekund po wyczyszczeniu kolejki mogą również zostać usunięte.

<a name="monitoring-your-queues"></a>
## Monitorowanie Kolejek

Jeśli twoja kolejka otrzyma nagły napływ zadań, może stać się przeciążona, prowadząc do długiego czasu oczekiwania na zakończenie zadań. Jeśli chcesz, Laravel może ostrzegać cię, gdy liczba zadań w kolejce przekroczy określony próg.

Aby zacząć, powinieneś zaplanować [uruchamianie co minutę](/docs/{{version}}/scheduling) polecenia `queue:monitor`. Polecenie akceptuje nazwy kolejek, które chcesz monitorować, a także pożądany próg liczby zadań:

```shell
php artisan queue:monitor redis:default,redis:deployments --max=100
```

Samodzielne zaplanowanie tego polecenia nie wystarczy, aby wywołać powiadomienie o przeciążeniu kolejki. Gdy polecenie napotka kolejkę, która ma liczbę zadań przekraczającą twój próg, zostanie wywołane zdarzenie `Illuminate\Queue\Events\QueueBusy`. Możesz nasłuchiwać tego zdarzenia w `AppServiceProvider` swojej aplikacji, aby wysłać powiadomienie do ciebie lub twojego zespołu deweloperskiego:

```php
use App\Notifications\QueueHasLongWaitTime;
use Illuminate\Queue\Events\QueueBusy;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Notification;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(function (QueueBusy $event) {
        Notification::route('mail', 'dev@example.com')
            ->notify(new QueueHasLongWaitTime(
                $event->connection,
                $event->queue,
                $event->size
            ));
    });
}
```

<a name="testing"></a>
## Testowanie

Podczas testowania kodu, który wysyła zadania, możesz chcieć nakazać Laravel, aby nie wykonywał rzeczywiście samego zadania, ponieważ kod zadania może być testowany bezpośrednio i niezależnie od kodu, który go wysyła. Oczywiście, aby przetestować samo zadanie, możesz utworzyć instancję zadania i bezpośrednio wywołać metodę `handle` w swoim teście.

Możesz użyć metody `fake` fasady `Queue`, aby zapobiec rzeczywistemu dodawaniu zadań do kolejki. Po wywołaniu metody `fake` fasady `Queue` możesz następnie utrzymać, że aplikacja próbowała dodać zadania do kolejki:

```php tab=Pest
<?php

use App\Jobs\AnotherJob;
use App\Jobs\ShipOrder;
use Illuminate\Support\Facades\Queue;

test('orders can be shipped', function () {
    Queue::fake();

    // Perform order shipping...

    // Assert that no jobs were pushed...
    Queue::assertNothingPushed();

    // Assert a job was pushed to a given queue...
    Queue::assertPushedOn('queue-name', ShipOrder::class);

    // Assert a job was pushed twice...
    Queue::assertPushed(ShipOrder::class, 2);

    // Assert a job was not pushed...
    Queue::assertNotPushed(AnotherJob::class);

    // Assert that a closure was pushed to the queue...
    Queue::assertClosurePushed();

    // Assert that a closure was not pushed...
    Queue::assertClosureNotPushed();

    // Assert the total number of jobs that were pushed...
    Queue::assertCount(3);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Jobs\AnotherJob;
use App\Jobs\ShipOrder;
use Illuminate\Support\Facades\Queue;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        Queue::fake();

        // Perform order shipping...

        // Assert that no jobs were pushed...
        Queue::assertNothingPushed();

        // Assert a job was pushed to a given queue...
        Queue::assertPushedOn('queue-name', ShipOrder::class);

        // Assert a job was pushed twice...
        Queue::assertPushed(ShipOrder::class, 2);

        // Assert a job was not pushed...
        Queue::assertNotPushed(AnotherJob::class);

        // Assert that a closure was pushed to the queue...
        Queue::assertClosurePushed();

        // Assert that a closure was not pushed...
        Queue::assertClosureNotPushed();

        // Assert the total number of jobs that were pushed...
        Queue::assertCount(3);
    }
}
```

You may pass a closure to the `assertPushed`, `assertNotPushed`, `assertClosurePushed`, or `assertClosureNotPushed` methods in order to assert that a job was pushed that passes a given "truth test". If at least one job was pushed that passes the given truth test then the assertion will be successful:

```php
use Illuminate\Queue\CallQueuedClosure;

Queue::assertPushed(function (ShipOrder $job) use ($order) {
    return $job->order->id === $order->id;
});

Queue::assertClosurePushed(function (CallQueuedClosure $job) {
    return $job->name === 'validate-order';
});
```

<a name="faking-a-subset-of-jobs"></a>
### Faking a Subset of Jobs

If you only need to fake specific jobs while allowing your other jobs to execute normally, you may pass the class names of the jobs that should be faked to the `fake` method:

```php tab=Pest
test('orders can be shipped', function () {
    Queue::fake([
        ShipOrder::class,
    ]);

    // Perform order shipping...

    // Assert a job was pushed twice...
    Queue::assertPushed(ShipOrder::class, 2);
});
```

```php tab=PHPUnit
public function test_orders_can_be_shipped(): void
{
    Queue::fake([
        ShipOrder::class,
    ]);

    // Perform order shipping...

    // Assert a job was pushed twice...
    Queue::assertPushed(ShipOrder::class, 2);
}
```

You may fake all jobs except for a set of specified jobs using the `except` method:

```php
Queue::fake()->except([
    ShipOrder::class,
]);
```

<a name="testing-job-chains"></a>
### Testing Job Chains

To test job chains, you will need to utilize the `Bus` facade's faking capabilities. The `Bus` facade's `assertChained` method may be used to assert that a [chain of jobs](/docs/{{version}}/queues#job-chaining) was dispatched. The `assertChained` method accepts an array of chained jobs as its first argument:

```php
use App\Jobs\RecordShipment;
use App\Jobs\ShipOrder;
use App\Jobs\UpdateInventory;
use Illuminate\Support\Facades\Bus;

Bus::fake();

// ...

Bus::assertChained([
    ShipOrder::class,
    RecordShipment::class,
    UpdateInventory::class
]);
```

As you can see in the example above, the array of chained jobs may be an array of the job's class names. However, you may also provide an array of actual job instances. When doing so, Laravel will ensure that the job instances are of the same class and have the same property values of the chained jobs dispatched by your application:

```php
Bus::assertChained([
    new ShipOrder,
    new RecordShipment,
    new UpdateInventory,
]);
```

You may use the `assertDispatchedWithoutChain` method to assert that a job was pushed without a chain of jobs:

```php
Bus::assertDispatchedWithoutChain(ShipOrder::class);
```

<a name="testing-chain-modifications"></a>
#### Testing Chain Modifications

If a chained job [prepends or appends jobs to an existing chain](#adding-jobs-to-the-chain), you may use the job's `assertHasChain` method to assert that the job has the expected chain of remaining jobs:

```php
$job = new ProcessPodcast;

$job->handle();

$job->assertHasChain([
    new TranscribePodcast,
    new OptimizePodcast,
    new ReleasePodcast,
]);
```

The `assertDoesntHaveChain` method may be used to assert that the job's remaining chain is empty:

```php
$job->assertDoesntHaveChain();
```

<a name="testing-chained-batches"></a>
#### Testing Chained Batches

If your job chain [contains a batch of jobs](#chains-and-batches), you may assert that the chained batch matches your expectations by inserting a `Bus::chainedBatch` definition within your chain assertion:

```php
use App\Jobs\ShipOrder;
use App\Jobs\UpdateInventory;
use Illuminate\Bus\PendingBatch;
use Illuminate\Support\Facades\Bus;

Bus::assertChained([
    new ShipOrder,
    Bus::chainedBatch(function (PendingBatch $batch) {
        return $batch->jobs->count() === 3;
    }),
    new UpdateInventory,
]);
```

<a name="testing-job-batches"></a>
### Testing Job Batches

The `Bus` facade's `assertBatched` method may be used to assert that a [batch of jobs](/docs/{{version}}/queues#job-batching) was dispatched. The closure given to the `assertBatched` method receives an instance of `Illuminate\Bus\PendingBatch`, which may be used to inspect the jobs within the batch:

```php
use Illuminate\Bus\PendingBatch;
use Illuminate\Support\Facades\Bus;

Bus::fake();

// ...

Bus::assertBatched(function (PendingBatch $batch) {
    return $batch->name == 'Import CSV' &&
           $batch->jobs->count() === 10;
});
```

You may use the `assertBatchCount` method to assert that a given number of batches were dispatched:

```php
Bus::assertBatchCount(3);
```

You may use `assertNothingBatched` to assert that no batches were dispatched:

```php
Bus::assertNothingBatched();
```

<a name="testing-job-batch-interaction"></a>
#### Testing Job / Batch Interaction

Dodatkowo, możesz czasami potrzebować przetestować interakcję pojedynczego zadania z jego bazową partią. Na przykład, możesz potrzebować sprawdzić, czy zadanie anulowało dalsze przetwarzanie swojej partii. Aby to osiągnąć, musisz przypisać fałszywą partię do zadania za pomocą metody `withFakeBatch`. Metoda `withFakeBatch` zwraca krotkę zawierającą instancję zadania i fałszywą partię:

```php
[$job, $batch] = (new ShipOrder)->withFakeBatch();

$job->handle();

$this->assertTrue($batch->cancelled());
$this->assertEmpty($batch->added);
```

<a name="testing-job-queue-interactions"></a>
### Testowanie Interakcji Zadania / Kolejki

Czasami możesz potrzebować przetestować, czy zadanie w kolejce [zwalnia się z powrotem do kolejki](#manually-releasing-a-job). Lub możesz potrzebować przetestować, czy zadanie usunęło się samo. Możesz przetestować te interakcje kolejki poprzez utworzenie instancji zadania i wywołanie metody `withFakeQueueInteractions`.

Po sfałszowaniu interakcji zadania z kolejką, możesz wywołać metodę `handle` na zadaniu. Po wywołaniu zadania dostępne są różne metody asercji do weryfikacji interakcji zadania z kolejką:

```php
use App\Exceptions\CorruptedAudioException;
use App\Jobs\ProcessPodcast;

$job = (new ProcessPodcast)->withFakeQueueInteractions();

$job->handle();

$job->assertReleased(delay: 30);
$job->assertDeleted();
$job->assertNotDeleted();
$job->assertFailed();
$job->assertFailedWith(CorruptedAudioException::class);
$job->assertNotFailed();
```

<a name="job-events"></a>
## Zdarzenia Zadań

Używając metod `before` i `after` na [fasadzie](/docs/{{version}}/facades) `Queue`, możesz określić callbacki do wykonania przed lub po przetworzeniu zadania w kolejce. Te callbacki są świetną okazją do wykonania dodatkowego logowania lub inkrementacji statystyk dla dashboardu. Zazwyczaj powinieneś wywoływać te metody z metody `boot` [dostawcy usług](/docs/{{version}}/providers). Na przykład, możemy użyć `AppServiceProvider`, który jest dołączony do Laravel:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Queue;
use Illuminate\Support\ServiceProvider;
use Illuminate\Queue\Events\JobProcessed;
use Illuminate\Queue\Events\JobProcessing;

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
        Queue::before(function (JobProcessing $event) {
            // $event->connectionName
            // $event->job
            // $event->job->payload()
        });

        Queue::after(function (JobProcessed $event) {
            // $event->connectionName
            // $event->job
            // $event->job->payload()
        });
    }
}
```

Używając metody `looping` na [fasadzie](/docs/{{version}}/facades) `Queue`, możesz określić callbacki, które są wykonywane przed próbą pobrania przez worker zadania z kolejki. Na przykład, możesz zarejestrować closure do wycofania wszelkich transakcji, które zostały pozostawione otwarte przez wcześniej nieudane zadanie:

```php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Queue;

Queue::looping(function () {
    while (DB::transactionLevel() > 0) {
        DB::rollBack();
    }
});
```
