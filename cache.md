# Cache

- [Wprowadzenie](#introduction)
- [Konfiguracja](#configuration)
    - [Wymagania sterowników](#driver-prerequisites)
- [Użycie cache](#cache-usage)
    - [Uzyskiwanie instancji cache](#obtaining-a-cache-instance)
    - [Pobieranie elementów z cache](#retrieving-items-from-the-cache)
    - [Przechowywanie elementów w cache](#storing-items-in-the-cache)
    - [Usuwanie elementów z cache](#removing-items-from-the-cache)
    - [Memoizacja cache](#cache-memoization)
    - [Helper cache](#the-cache-helper)
- [Tagi cache](#cache-tags)
- [Blokady atomowe](#atomic-locks)
    - [Zarządzanie blokadami](#managing-locks)
    - [Zarządzanie blokadami między procesami](#managing-locks-across-processes)
    - [Blokady i wywołania funkcji](#locks-and-function-invocations)
- [Failover cache](#cache-failover)
- [Dodawanie niestandardowych sterowników cache](#adding-custom-cache-drivers)
    - [Pisanie sterownika](#writing-the-driver)
    - [Rejestrowanie sterownika](#registering-the-driver)
- [Zdarzenia](#events)

<a name="introduction"></a>
## Wprowadzenie

Niektóre zadania pobierania lub przetwarzania danych wykonywane przez Twoją aplikację mogą być intensywne dla procesora lub trwać kilka sekund. W takim przypadku powszechną praktyką jest tymczasowe przechowywanie pobranych danych w cache, aby można je było szybko pobrać przy kolejnych żądaniach tych samych danych. Dane z cache są zwykle przechowywane w bardzo szybkim magazynie danych, takim jak [Memcached](https://memcached.org) lub [Redis](https://redis.io).

Na szczęście Laravel zapewnia ekspresyjne, zunifikowane API dla różnych backendów cache, umożliwiając wykorzystanie ich błyskawicznie szybkiego pobierania danych i przyspieszenie aplikacji webowej.

<a name="configuration"></a>
## Konfiguracja

Plik konfiguracyjny cache Twojej aplikacji znajduje się w `config/cache.php`. W tym pliku możesz określić, który magazyn cache ma być domyślnie używany w całej aplikacji. Laravel wspiera popularne backendy cache, takie jak [Memcached](https://memcached.org), [Redis](https://redis.io), [DynamoDB](https://aws.amazon.com/dynamodb) oraz relacyjne bazy danych. Ponadto dostępny jest sterownik cache oparty na plikach, podczas gdy sterowniki `array` i `null` zapewniają wygodne backendy cache dla testów automatycznych.

Plik konfiguracyjny cache zawiera również wiele innych opcji, które możesz przejrzeć. Domyślnie Laravel jest skonfigurowany do używania sterownika cache `database`, który przechowuje zserializowane obiekty cache w bazie danych aplikacji.

<a name="driver-prerequisites"></a>
### Wymagania sterowników

<a name="prerequisites-database"></a>
#### Baza danych

Podczas używania sterownika cache `database` będziesz potrzebować tabeli bazy danych do przechowywania danych cache. Zazwyczaj jest to zawarte w domyślnej [migracji bazy danych](/docs/{{version}}/migrations) Laravel `0001_01_01_000001_create_cache_table.php`; jednak jeśli Twoja aplikacja nie zawiera tej migracji, możesz użyć polecenia Artisan `make:cache-table`, aby ją utworzyć:

```shell
php artisan make:cache-table

php artisan migrate
```

<a name="memcached"></a>
#### Memcached

Używanie sterownika Memcached wymaga zainstalowania [pakietu Memcached PECL](https://pecl.php.net/package/memcached). Możesz wypisać wszystkie swoje serwery Memcached w pliku konfiguracyjnym `config/cache.php`. Ten plik już zawiera wpis `memcached.servers`, aby Ci pomóc:

```php
'memcached' => [
    // ...

    'servers' => [
        [
            'host' => env('MEMCACHED_HOST', '127.0.0.1'),
            'port' => env('MEMCACHED_PORT', 11211),
            'weight' => 100,
        ],
    ],
],
```

Jeśli to konieczne, możesz ustawić opcję `host` na ścieżkę gniazda UNIX. Jeśli to zrobisz, opcja `port` powinna być ustawiona na `0`:

```php
'memcached' => [
    // ...

    'servers' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],
],
```

<a name="redis"></a>
#### Redis

Przed użyciem cache Redis z Laravel musisz zainstalować rozszerzenie PHP PhpRedis przez PECL lub zainstalować pakiet `predis/predis` (~2.0) przez Composer. [Laravel Sail](/docs/{{version}}/sail) już zawiera to rozszerzenie. Ponadto oficjalne platformy aplikacji Laravel, takie jak [Laravel Cloud](https://cloud.laravel.com) i [Laravel Forge](https://forge.laravel.com), mają domyślnie zainstalowane rozszerzenie PhpRedis.

Aby uzyskać więcej informacji na temat konfigurowania Redis, zapoznaj się z [jego stroną dokumentacji Laravel](/docs/{{version}}/redis#configuration).

<a name="dynamodb"></a>
#### DynamoDB

Przed użyciem sterownika cache [DynamoDB](https://aws.amazon.com/dynamodb) musisz utworzyć tabelę DynamoDB do przechowywania wszystkich danych cache. Zazwyczaj ta tabela powinna nazywać się `cache`. Jednak powinieneś nazwać tabelę na podstawie wartości konfiguracji `stores.dynamodb.table` w pliku konfiguracyjnym `cache`. Nazwa tabeli może być również ustawiona przez zmienną środowiskową `DYNAMODB_CACHE_TABLE`.

Ta tabela powinna również mieć klucz partycji typu string o nazwie odpowiadającej wartości elementu konfiguracyjnego `stores.dynamodb.attributes.key` w pliku konfiguracyjnym `cache` Twojej aplikacji. Domyślnie klucz partycji powinien nazywać się `key`.

Zazwyczaj DynamoDB nie usuwa proaktywnie wygasłych elementów z tabeli. Dlatego powinieneś [włączyć Time to Live (TTL)](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html) dla tabeli. Konfigurując ustawienia TTL tabeli, powinieneś ustawić nazwę atrybutu TTL na `expires_at`.

Następnie zainstaluj AWS SDK, aby Twoja aplikacja Laravel mogła komunikować się z DynamoDB:

```shell
composer require aws/aws-sdk-php
```

Ponadto powinieneś upewnić się, że wartości są dostarczone dla opcji konfiguracyjnych magazynu cache DynamoDB. Zazwyczaj te opcje, takie jak `AWS_ACCESS_KEY_ID` i `AWS_SECRET_ACCESS_KEY`, powinny być zdefiniowane w pliku konfiguracyjnym `.env` Twojej aplikacji:

```php
'dynamodb' => [
    'driver' => 'dynamodb',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => env('DYNAMODB_CACHE_TABLE', 'cache'),
    'endpoint' => env('DYNAMODB_ENDPOINT'),
],
```

<a name="mongodb"></a>
#### MongoDB

Jeśli używasz MongoDB, sterownik cache `mongodb` jest dostarczany przez oficjalny pakiet `mongodb/laravel-mongodb` i może być skonfigurowany przy użyciu połączenia bazy danych `mongodb`. MongoDB wspiera indeksy TTL, które mogą być używane do automatycznego czyszczenia wygasłych elementów cache.

Aby uzyskać więcej informacji na temat konfigurowania MongoDB, zapoznaj się z [dokumentacją Cache and Locks](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/cache/) MongoDB.

<a name="cache-usage"></a>
## Użycie cache

<a name="obtaining-a-cache-instance"></a>
### Uzyskiwanie instancji cache

Aby uzyskać instancję magazynu cache, możesz użyć fasady `Cache`, której będziemy używać w całej tej dokumentacji. Fasada `Cache` zapewnia wygodny, zwięzły dostęp do podstawowych implementacji kontraktów cache Laravel:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * Show a list of all users of the application.
     */
    public function index(): array
    {
        $value = Cache::get('key');

        return [
            // ...
        ];
    }
}
```

<a name="accessing-multiple-cache-stores"></a>
#### Dostęp do wielu magazynów cache

Używając fasady `Cache`, możesz uzyskiwać dostęp do różnych magazynów cache za pomocą metody `store`. Klucz przekazany do metody `store` powinien odpowiadać jednemu z magazynów wymienionych w tablicy konfiguracyjnej `stores` w pliku konfiguracyjnym `cache`:

```php
$value = Cache::store('file')->get('foo');

Cache::store('redis')->put('bar', 'baz', 600); // 10 minut
```

<a name="retrieving-items-from-the-cache"></a>
### Pobieranie elementów z cache

Metoda `get` fasady `Cache` służy do pobierania elementów z cache. Jeśli element nie istnieje w cache, zostanie zwrócona wartość `null`. Jeśli chcesz, możesz przekazać drugi argument do metody `get`, określając wartość domyślną, która ma zostać zwrócona, jeśli element nie istnieje:

```php
$value = Cache::get('key');

$value = Cache::get('key', 'default');
```

Możesz nawet przekazać domknięcie jako wartość domyślną. Wynik domknięcia zostanie zwrócony, jeśli określony element nie istnieje w cache. Przekazanie domknięcia pozwala opóźnić pobieranie wartości domyślnych z bazy danych lub innej usługi zewnętrznej:

```php
$value = Cache::get('key', function () {
    return DB::table(/* ... */)->get();
});
```

<a name="determining-item-existence"></a>
#### Sprawdzanie istnienia elementu

Metoda `has` może być użyta do określenia, czy element istnieje w cache. Ta metoda zwróci również `false`, jeśli element istnieje, ale jego wartość to `null`:

```php
if (Cache::has('key')) {
    // ...
}
```

<a name="incrementing-decrementing-values"></a>
#### Inkrementacja / Dekrementacja wartości

Metody `increment` i `decrement` mogą być użyte do dostosowania wartości elementów liczb całkowitych w cache. Obie te metody akceptują opcjonalny drugi argument wskazujący wielkość, o którą należy zwiększyć lub zmniejszyć wartość elementu:

```php
// Inicjalizuj wartość, jeśli nie istnieje...
Cache::add('key', 0, now()->plus(hours: 4));

// Inkrementuj lub dekrementuj wartość...
Cache::increment('key');
Cache::increment('key', $amount);
Cache::decrement('key');
Cache::decrement('key', $amount);
```

<a name="retrieve-store"></a>
#### Pobierz i przechowaj

Czasami możesz chcieć pobrać element z cache, ale również przechowywać wartość domyślną, jeśli żądany element nie istnieje. Na przykład możesz chcieć pobrać wszystkich użytkowników z cache lub, jeśli nie istnieją, pobrać ich z bazy danych i dodać do cache. Możesz to zrobić używając metody `Cache::remember`:

```php
$value = Cache::remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

Jeśli element nie istnieje w cache, domknięcie przekazane do metody `remember` zostanie wykonane, a jego wynik zostanie umieszczony w cache.

Możesz użyć metody `rememberForever`, aby pobrać element z cache lub przechowywać go na zawsze, jeśli nie istnieje:

```php
$value = Cache::rememberForever('users', function () {
    return DB::table('users')->get();
});
```

<a name="swr"></a>
#### Stale While Revalidate (Nieaktualne podczas rewalidacji)

Podczas używania metody `Cache::remember` niektórzy użytkownicy mogą doświadczać wolnych czasów odpowiedzi, jeśli wartość cache wygasła. Dla niektórych typów danych może być przydatne pozwolić na serwowanie częściowo nieaktualnych danych, podczas gdy wartość cache jest przeliczana w tle, zapobiegając wolnym czasom odpowiedzi dla niektórych użytkowników podczas obliczania wartości cache. Nazywa się to często wzorcem "stale-while-revalidate", a metoda `Cache::flexible` zapewnia implementację tego wzorca.

Metoda flexible akceptuje tablicę, która określa, jak długo wartość cache jest uważana za "świeżą" i kiedy staje się "nieaktualna". Pierwsza wartość w tablicy reprezentuje liczbę sekund, przez które cache jest uważany za świeży, podczas gdy druga wartość określa, jak długo może być serwowany jako nieaktualne dane przed koniecznością przeliczenia.

Jeśli żądanie jest wykonane w okresie świeżym (przed pierwszą wartością), cache jest zwracany natychmiast bez przeliczenia. Jeśli żądanie jest wykonane podczas okresu nieaktualności (między dwiema wartościami), nieaktualna wartość jest serwowana użytkownikowi, a [odroczona funkcja](/docs/{{version}}/helpers#deferred-functions) jest rejestrowana, aby odświeżyć wartość cache po wysłaniu odpowiedzi do użytkownika. Jeśli żądanie jest wykonane po drugiej wartości, cache jest uważany za wygasły, a wartość jest przeliczana natychmiast, co może skutkować wolniejszą odpowiedzią dla użytkownika:

```php
$value = Cache::flexible('users', [5, 10], function () {
    return DB::table('users')->get();
});
```

<a name="retrieve-delete"></a>
#### Pobierz i usuń

Jeśli musisz pobrać element z cache, a następnie usunąć element, możesz użyć metody `pull`. Podobnie jak metoda `get`, wartość `null` zostanie zwrócona, jeśli element nie istnieje w cache:

```php
$value = Cache::pull('key');

$value = Cache::pull('key', 'default');
```

<a name="storing-items-in-the-cache"></a>
### Przechowywanie elementów w cache

Możesz użyć metody `put` fasady `Cache`, aby przechowywać elementy w cache:

```php
Cache::put('key', 'value', $seconds = 10);
```

Jeśli czas przechowywania nie jest przekazany do metody `put`, element będzie przechowywany w nieskończoność:

```php
Cache::put('key', 'value');
```

Zamiast przekazywać liczbę sekund jako liczbę całkowitą, możesz również przekazać instancję `DateTime` reprezentującą pożądany czas wygaśnięcia elementu cache:

```php
Cache::put('key', 'value', now()->plus(minutes: 10));
```

<a name="store-if-not-present"></a>
#### Przechowaj jeśli nie istnieje

Metoda `add` doda element do cache tylko wtedy, gdy nie istnieje jeszcze w magazynie cache. Metoda zwróci `true`, jeśli element zostanie rzeczywiście dodany do cache. W przeciwnym razie metoda zwróci `false`. Metoda `add` jest operacją atomową:

```php
Cache::add('key', 'value', $seconds);
```

<a name="storing-items-forever"></a>
#### Przechowywanie elementów na zawsze

Metoda `forever` może być użyta do przechowywania elementu w cache na stałe. Ponieważ te elementy nie wygasły, muszą być ręcznie usunięte z cache za pomocą metody `forget`:

```php
Cache::forever('key', 'value');
```

> [!NOTE]
> Jeśli używasz sterownika Memcached, elementy, które są przechowywane "na zawsze", mogą zostać usunięte, gdy cache osiągnie swoją granice rozmiaru.

<a name="removing-items-from-the-cache"></a>
### Usuwanie elementów z cache

Możesz usunąć elementy z cache używając metody `forget`:

```php
Cache::forget('key');
```

Możesz również usunąć elementy, podając zerową lub ujemną liczbę sekund wygaśnięcia:

```php
Cache::put('key', 'value', 0);

Cache::put('key', 'value', -5);
```

Możesz wyczyścić cały cache używając metody `flush`:

```php
Cache::flush();
```

> [!WARNING]
> Czyszczenie cache nie uwzględnia skonfigurowanego "prefiksu" cache i usunie wszystkie wpisy z cache. Rozważ to ostrożnie podczas czyszczenia cache, który jest współdzielony przez inne aplikacje.

<a name="cache-memoization"></a>
### Memoizacja cache

Sterownik cache `memo` Laravel pozwala tymczasowo przechowywać rozwiązane wartości cache w pamięci podczas pojedynczego żądania lub wykonania zadania. Zapobiega to powtórzonym odwołaniom do cache w tym samym wykonaniu, znacznie poprawiając wydajność.

Aby użyć cache z memoizacją, wywołaj metodę `memo`:

```php
use Illuminate\Support\Facades\Cache;

$value = Cache::memo()->get('key');
```

Metoda `memo` opcjonalnie akceptuje nazwę magazynu cache, który określa podstawowy magazyn cache, który zostanie ozdobiony przez sterownik z memoizacją:

```php
// Używając domyślnego magazynu cache...
$value = Cache::memo()->get('key');

// Używając magazynu cache Redis...
$value = Cache::memo('redis')->get('key');
```

Pierwsze wywołanie `get` dla danego klucza pobiera wartość z magazynu cache, ale kolejne wywołania w tym samym żądaniu lub zadaniu pobiorą wartość z pamięci:

```php
// Odwołuje się do cache...
$value = Cache::memo()->get('key');

// Nie odwołuje się do cache, zwraca zmemoralizowaną wartość...
$value = Cache::memo()->get('key');
```

Podczas wywoływania metod, które modyfikują wartości cache (takich jak `put`, `increment`, `remember` itp.), cache z memoizacją automatycznie zapomina zmemoralizowaną wartość i deleguje wywołanie metody modyfikującej do podstawowego magazynu cache:

```php
Cache::memo()->put('name', 'Taylor'); // Zapisuje do podstawowego cache...
Cache::memo()->get('name');           // Odwołuje się do podstawowego cache...
Cache::memo()->get('name');           // Zmemoralizowane, nie odwołuje się do cache...

Cache::memo()->put('name', 'Tim');    // Zapomina zmemoralizowaną wartość, zapisuje nową wartość...
Cache::memo()->get('name');           // Odwołuje się ponownie do podstawowego cache...
```

<a name="the-cache-helper"></a>
### Helper cache

Oprócz używania fasady `Cache`, możesz również użyć globalnej funkcji `cache` do pobierania i przechowywania danych przez cache. Gdy funkcja `cache` jest wywoływana z pojedynczym argumentem typu string, zwróci wartość danego klucza:

```php
$value = cache('key');
```

Jeśli podasz tablicę par klucz / wartość oraz czas wygaśnięcia do funkcji, przechowywa ona wartości w cache przez określony czas:

```php
cache(['key' => 'value'], $seconds);

cache(['key' => 'value'], now()->plus(minutes: 10));
```

Gdy funkcja `cache` jest wywoływana bez żadnych argumentów, zwraca instancję implementacji `Illuminate\Contracts\Cache\Factory`, pozwalając wywoływać inne metody cache:

```php
cache()->remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

> [!NOTE]
> Podczas testowania wywołań globalnej funkcji `cache`, możesz użyć metody `Cache::shouldReceive` tak jakbyś [testował fasadę](/docs/{{version}}/mocking#mocking-facades).

<a name="cache-tags"></a>
## Tagi cache

> [!WARNING]
> Tagi cache nie są obsługiwane podczas używania sterowników cache `file`, `dynamodb` lub `database`.

<a name="storing-tagged-cache-items"></a>
### Przechowywanie elementów cache z tagami

Tagi cache pozwalają tagować powiązane elementy w cache, a następnie opuścić wszystkie wartości cache, którym przypisano dany tag. Możesz uzyskać dostęp do cache z tagami, przekazując uporządkowaną tablicę nazw tagów. Na przykład uzyskajmy dostęp do cache z tagami i `put` wartość do cache:

```php
use Illuminate\Support\Facades\Cache;

Cache::tags(['people', 'artists'])->put('John', $john, $seconds);
Cache::tags(['people', 'authors'])->put('Anne', $anne, $seconds);
```

<a name="accessing-tagged-cache-items"></a>
### Dostęp do elementów cache z tagami

Elementy przechowywane przez tagi nie mogą być dostępne bez podania tagów, które zostały użyte do przechowywania wartości. Aby pobrać element cache z tagami, przekaż tę samą uporządkowaną listę tagów do metody `tags`, a następnie wywołaj metodę `get` z kluczem, który chcesz pobrać:

```php
$john = Cache::tags(['people', 'artists'])->get('John');

$anne = Cache::tags(['people', 'authors'])->get('Anne');
```

<a name="removing-tagged-cache-items"></a>
### Usuwanie elementów cache z tagami

Możesz opuścić wszystkie elementy, którym przypisano tag lub listę tagów. Na przykład poniższy kod usunąłby wszystkie cache otagowane `people`, `authors` lub obydwoma. Więc zarówno `Anne`, jak i `John` zostaliby usunięci z cache:

```php
Cache::tags(['people', 'authors'])->flush();
```

Z kolei poniższy kod usunąłby tylko wartości cache otagowane `authors`, więc `Anne` zostanie usunięta, ale nie `John`:

```php
Cache::tags('authors')->flush();
```

<a name="atomic-locks"></a>
## Blokady atomowe

> [!WARNING]
> Aby wykorzystać tę funkcję, Twoja aplikacja musi używać sterownika cache `memcached`, `redis`, `dynamodb`, `database`, `file` lub `array` jako domyślnego sterownika cache aplikacji. Ponadto wszystkie serwery muszą komunikować się z tym samym centralnym serwerem cache.

<a name="managing-locks"></a>
### Zarządzanie blokadami

Blokady atomowe pozwalają na manipulację rozproszonymi blokadami bez martwienia się o race conditions. Na przykład [Laravel Cloud](https://cloud.laravel.com) używa blokad atomowych, aby zapewnić, że tylko jedno zdalne zadanie jest wykonywane na serwerze w danym czasie. Możesz tworzyć i zarządzać blokadami używając metody `Cache::lock`:

```php
use Illuminate\Support\Facades\Cache;

$lock = Cache::lock('foo', 10);

if ($lock->get()) {
    // Blokada nabyta na 10 sekund...

    $lock->release();
}
```

Metoda `get` akceptuje również domknięcie. Po wykonaniu domknięcia Laravel automatycznie zwolni blokadę:

```php
Cache::lock('foo', 10)->get(function () {
    // Blokada nabyta na 10 sekund i automatycznie zwolniona...
});
```

Jeśli blokada nie jest dostępna w momencie, gdy ją żądasz, możesz polecić Laravel, aby czekał przez określoną liczbę sekund. Jeśli blokada nie może zostać nabyta w określonym limicie czasu, zostanie zrzucony wyjątek `Illuminate\Contracts\Cache\LockTimeoutException`:

```php
use Illuminate\Contracts\Cache\LockTimeoutException;

$lock = Cache::lock('foo', 10);

try {
    $lock->block(5);

    // Blokada nabyta po oczekiwaniu maksymalnie 5 sekund...
} catch (LockTimeoutException $e) {
    // Nie udało się nabyć blokady...
} finally {
    $lock->release();
}
```

Powyższy przykład może zostać uproszczony przez przekazanie domknięcia do metody `block`. Gdy domknięcie jest przekazane do tej metody, Laravel spróbuje nabyć blokadę przez określoną liczbę sekund i automatycznie zwolni blokadę po wykonaniu domknięcia:

```php
Cache::lock('foo', 10)->block(5, function () {
    // Blokada nabyta na 10 sekund po oczekiwaniu maksymalnie 5 sekund...
});
```

<a name="managing-locks-across-processes"></a>
### Zarządzanie blokadami między procesami

Czasami możesz chcieć nabyć blokadę w jednym procesie i zwolnić ją w innym procesie. Na przykład możesz nabyć blokadę podczas żądania webowego i chcieć zwolnić blokadę na końcu zadania w kolejce, które jest wyzwalane przez to żądanie. W tym scenariuszu powinieneś przekazać zakresowy "token właściciela" blokady do zadania w kolejce, aby zadanie mogło ponownie zinstancjonować blokadę używając danego tokena.

W poniższym przykładzie wysyłamy zadanie w kolejce, jeśli blokada zostanie pomyślnie nabyta. Ponadto przekazujemy token właściciela blokady do zadania w kolejce za pomocą metody `owner` blokady:

```php
$podcast = Podcast::find($id);

$lock = Cache::lock('processing', 120);

if ($lock->get()) {
    ProcessPodcast::dispatch($podcast, $lock->owner());
}
```

W zadaniu `ProcessPodcast` naszej aplikacji możemy przywrócić i zwolnić blokadę używając tokena właściciela:

```php
Cache::restoreLock('processing', $this->owner)->release();
```

Jeśli chcesz zwolnić blokadę bez przestrzegania jej obecnego właściciela, możesz użyć metody `forceRelease`:

```php
Cache::lock('processing')->forceRelease();
```

<a name="locks-and-function-invocations"></a>
### Blokady i wywołania funkcji

Metoda `withoutOverlapping` zapewnia prostą składnię do wykonywania danego domknięcia podczas utrzymywania blokady atomowej, pozwalając zapewnić, że tylko jedna instancja domknięcia działa w danym czasie w całej infrastrukturze:

```php
Cache::withoutOverlapping('foo', function () {
    // Blokada nabyta po oczekiwaniu maksymalnie 10 sekund...
});
```

Domyślnie blokada nie zostanie zwolniona, dopóki domknięcie nie zakończy wykonywania, a metoda będzie czekać do 10 sekund na nabycie blokady. Możesz dostosować te wartości, przekazując dodatkowe argumenty do metody:

```php
Cache::withoutOverlapping('foo', function () {
    // Blokada nabyta na 120 sekund po oczekiwaniu maksymalnie 5 sekund...
}, lockSeconds: 120, waitSeconds: 5);
```

Jeśli blokada nie może zostać nabyta w określonym czasie oczekiwania, zostanie zrzucony wyjątek `Illuminate\Contracts\Cache\LockTimeoutException`.

<a name="cache-failover"></a>
## Failover cache

Sterownik cache `failover` zapewnia automatyczną funkcjonalność failover podczas interakcji z cache. Jeśli główny magazyn cache magazynu `failover` zawiedzie z jakiegokolwiek powodu, Laravel automatycznie spróbuje użyć następnego skonfigurowanego magazynu na liście. Jest to szczególnie przydatne do zapewnienia wysokiej dostępności w środowiskach produkcyjnych, gdzie niezawodność cache jest krytyczna.

Aby skonfigurować magazyn cache failover, określ sterownik `failover` i podaj tablicę nazw magazynów do próbowania w kolejności. Domyślnie Laravel zawiera przykładową konfigurację failover w pliku konfiguracyjnym `config/cache.php` Twojej aplikacji:

```php
'failover' => [
    'driver' => 'failover',
    'stores' => [
        'database',
        'array',
    ],
],
```

Po skonfigurowaniu magazynu, który używa sterownika `failover`, musisz ustawić magazyn failover jako domyślny magazyn cache w pliku `.env` Twojej aplikacji, aby wykorzystać funkcjonalność failover:

```ini
CACHE_STORE=failover
```

Gdy operacja magazynu cache zawiedzie i failover zostanie aktywowany, Laravel wysłe zdarzenie `Illuminate\Cache\Events\CacheFailedOver`, pozwalając Ci zgłaszać lub logować, że magazyn cache zawiedł.

<a name="adding-custom-cache-drivers"></a>
## Dodawanie niestandardowych sterowników cache

<a name="writing-the-driver"></a>
### Pisanie sterownika

Aby utworzyć nasz niestandardowy sterownik cache, najpierw musimy zaimplementować [kontrakt](/docs/{{version}}/contracts) `Illuminate\Contracts\Cache\Store`. Więc implementacja cache MongoDB mogłaby wyglądać mniej więcej tak:

```php
<?php

namespace App\Extensions;

use Illuminate\Contracts\Cache\Store;

class MongoStore implements Store
{
    public function get($key) {}
    public function many(array $keys) {}
    public function put($key, $value, $seconds) {}
    public function putMany(array $values, $seconds) {}
    public function increment($key, $value = 1) {}
    public function decrement($key, $value = 1) {}
    public function forever($key, $value) {}
    public function forget($key) {}
    public function flush() {}
    public function getPrefix() {}
}
```

Po prostu musimy zaimplementować każdą z tych metod używając połączenia MongoDB. Aby uzyskać przykład implementacji każdej z tych metod, spójrz na `Illuminate\Cache\MemcachedStore` w [kodzie źródłowym frameworka Laravel](https://github.com/laravel/framework). Po zakończeniu naszej implementacji możemy zakończyć rejestrację naszego niestandardowego sterownika, wywołując metodę `extend` fasady `Cache`:

```php
Cache::extend('mongo', function (Application $app) {
    return Cache::repository(new MongoStore);
});
```

> [!NOTE]
> Jeśli zastanawiasz się, gdzie umieścić kod niestandardowego sterownika cache, możesz utworzyć przestrzeń nazw `Extensions` w katalogu `app`. Jednak pamiętaj, że Laravel nie ma sztywnej struktury aplikacji i możesz swobodnie organizować aplikację zgodnie z własnymi preferencjami.

<a name="registering-the-driver"></a>
### Rejestrowanie sterownika

Aby zarejestrować niestandardowy sterownik cache z Laravel, użyjemy metody `extend` fasady `Cache`. Ponieważ inni dostawcy usług mogą próbować odczytywać wartości cache w ich metodzie `boot`, zarejestrujemy nasz niestandardowy sterownik w callbacku `booting`. Używając callbacku `booting`, możemy zapewnić, że niestandardowy sterownik jest zarejestrowany tuż przed wywołaniem metody `boot` w dostawcach usług naszej aplikacji, ale po wywołaniu metody `register` we wszystkich dostawcach usług. Zarejestrujemy nasz callback `booting` w metodzie `register` klasy `App\Providers\AppServiceProvider` naszej aplikacji:

```php
<?php

namespace App\Providers;

use App\Extensions\MongoStore;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->booting(function () {
             Cache::extend('mongo', function (Application $app) {
                 return Cache::repository(new MongoStore);
             });
         });
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        // ...
    }
}
```

Pierwszy argument przekazany do metody `extend` to nazwa sterownika. Będzie to odpowiadać Twojej opcji `driver` w pliku konfiguracyjnym `config/cache.php`. Drugi argument to domknięcie, które powinno zwrócić instancję `Illuminate\Cache\Repository`. Domknięcie otrzyma instancję `$app`, która jest instancją [kontenera usług](/docs/{{version}}/container).

Po zarejestrowaniu rozszerzenia zaktualizuj zmienną środowiskową `CACHE_STORE` lub opcję `default` w pliku konfiguracyjnym `config/cache.php` Twojej aplikacji na nazwę Twojego rozszerzenia.

<a name="events"></a>
## Zdarzenia

Aby wykonać kod przy każdej operacji cache, możesz nasłuchiwać różnych [zdarzeń](/docs/{{version}}/events) wysyłanych przez cache:

<div class="overflow-auto">

| Nazwa zdarzenia                              |
|----------------------------------------------|
| `Illuminate\Cache\Events\CacheFlushed`       |
| `Illuminate\Cache\Events\CacheFlushing`      |
| `Illuminate\Cache\Events\CacheHit`           |
| `Illuminate\Cache\Events\CacheMissed`        |
| `Illuminate\Cache\Events\ForgettingKey`      |
| `Illuminate\Cache\Events\KeyForgetFailed`    |
| `Illuminate\Cache\Events\KeyForgotten`       |
| `Illuminate\Cache\Events\KeyWriteFailed`     |
| `Illuminate\Cache\Events\KeyWritten`         |
| `Illuminate\Cache\Events\RetrievingKey`      |
| `Illuminate\Cache\Events\RetrievingManyKeys` |
| `Illuminate\Cache\Events\WritingKey`         |
| `Illuminate\Cache\Events\WritingManyKeys`    |

</div>

Aby zwiększyć wydajność, możesz wyłączyć zdarzenia cache, ustawiając opcję konfiguracyjną `events` na `false` dla danego magazynu cache w pliku konfiguracyjnym `config/cache.php` Twojej aplikacji:

```php
'database' => [
    'driver' => 'database',
    // ...
    'events' => false,
],
```
