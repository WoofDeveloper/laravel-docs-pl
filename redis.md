# Redis

- [Wprowadzenie](#introduction)
- [Konfiguracja](#configuration)
    - [Klastry](#clusters)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [Interakcja z Redis](#interacting-with-redis)
    - [Transakcje](#transactions)
    - [Przetwarzanie Potokowe Komend](#pipelining-commands)
- [Pub / Sub](#pubsub)

<a name="introduction"></a>
## Wprowadzenie

[Redis](https://redis.io) to otwartoźródłowy, zaawansowany magazyn klucz-wartość. Jest często określany jako serwer struktur danych, ponieważ klucze mogą zawierać [ciągi znaków](https://redis.io/docs/latest/develop/data-types/strings/), [hasze](https://redis.io/docs/latest/develop/data-types/hashes/), [listy](https://redis.io/docs/latest/develop/data-types/lists/), [zbiory](https://redis.io/docs/latest/develop/data-types/sets/) i [posortowane zbiory](https://redis.io/docs/latest/develop/data-types/sorted-sets/).

Przed użyciem Redis z Laravelem, zachęcamy do zainstalowania i użycia rozszerzenia PHP [PhpRedis](https://github.com/phpredis/phpredis) przez PECL. Rozszerzenie jest bardziej skomplikowane w instalacji w porównaniu do pakietów PHP "user-land", ale może zapewnić lepszą wydajność dla aplikacji intensywnie korzystających z Redis. Jeśli używasz [Laravel Sail](/docs/{{version}}/sail), to rozszerzenie jest już zainstalowane w kontenerze Docker twojej aplikacji.

Jeśli nie możesz zainstalować rozszerzenia PhpRedis, możesz zainstalować pakiet `predis/predis` przez Composer. Predis to klient Redis napisany całkowicie w PHP i nie wymaga żadnych dodatkowych rozszerzeń:

```shell
composer require predis/predis
```

<a name="configuration"></a>
## Konfiguracja

Możesz skonfigurować ustawienia Redis twojej aplikacji za pomocą pliku konfiguracyjnego `config/database.php`. W tym pliku zobaczysz tablicę `redis` zawierającą serwery Redis używane przez twoją aplikację:

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'default' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'username' => env('REDIS_USERNAME'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
    ],

    'cache' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'username' => env('REDIS_USERNAME'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_CACHE_DB', '1'),
    ],

],
```

Każdy serwer Redis zdefiniowany w pliku konfiguracyjnym musi mieć nazwę, hosta i port, chyba że zdefiniujesz pojedynczy URL reprezentujący połączenie Redis:

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'default' => [
        'url' => 'tcp://127.0.0.1:6379?database=0',
    ],

    'cache' => [
        'url' => 'tls://user:password@127.0.0.1:6380?database=1',
    ],

],
```

<a name="configuring-the-connection-scheme"></a>
#### Konfiguracja Schematu Połączenia

Domyślnie klienci Redis będą używać schematu `tcp` podczas łączenia się z serwerami Redis; jednak możesz użyć szyfrowania TLS / SSL, określając opcję konfiguracyjną `scheme` w tablicy konfiguracyjnej serwera Redis:

```php
'default' => [
    'scheme' => 'tls',
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
],
```

<a name="clusters"></a>
### Klastry

Jeśli twoja aplikacja używa klastra serwerów Redis, powinieneś zdefiniować te klastry w kluczu `clusters` twojej konfiguracji Redis. Ten klucz konfiguracyjny nie istnieje domyślnie, więc będziesz musiał go utworzyć w pliku konfiguracyjnym `config/database.php` twojej aplikacji:

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'clusters' => [
        'default' => [
            [
                'url' => env('REDIS_URL'),
                'host' => env('REDIS_HOST', '127.0.0.1'),
                'username' => env('REDIS_USERNAME'),
                'password' => env('REDIS_PASSWORD'),
                'port' => env('REDIS_PORT', '6379'),
                'database' => env('REDIS_DB', '0'),
            ],
        ],
    ],

    // ...
],
```

Domyślnie Laravel będzie używać natywnego klastrowania Redis, ponieważ wartość konfiguracyjna `options.cluster` jest ustawiona na `redis`. Klastrowanie Redis to doskonała domyślna opcja, ponieważ sprawnie obsługuje awarie.

Laravel obsługuje również sharding po stronie klienta podczas używania Predis. Jednak sharding po stronie klienta nie obsługuje awarii; dlatego jest przede wszystkim odpowiedni dla przejściowych danych z cache, które są dostępne z innego głównego magazynu danych.

Jeśli chcesz użyć shardingu po stronie klienta zamiast natywnego klastrowania Redis, możesz usunąć wartość konfiguracyjną `options.cluster` w pliku konfiguracyjnym `config/database.php` twojej aplikacji:

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'clusters' => [
        // ...
    ],

    // ...
],
```

<a name="predis"></a>
### Predis

Jeśli chcesz, aby twoja aplikacja wchodziła w interakcję z Redis przez pakiet Predis, powinieneś upewnić się, że wartość zmiennej środowiskowej `REDIS_CLIENT` to `predis`:

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'predis'),

    // ...
],
```

Oprócz domyślnych opcji konfiguracyjnych, Predis obsługuje dodatkowe [parametry połączenia](https://github.com/nrk/predis/wiki/Connection-Parameters), które mogą zostać zdefiniowane dla każdego z twoich serwerów Redis. Aby użyć tych dodatkowych opcji konfiguracyjnych, dodaj je do konfiguracji serwera Redis w pliku konfiguracyjnym `config/database.php` twojej aplikacji:

```php
'default' => [
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'read_write_timeout' => 60,
],
```

<a name="phpredis"></a>
### PhpRedis

Domyślnie Laravel będzie używać rozszerzenia PhpRedis do komunikacji z Redis. Klient, którego Laravel użyje do komunikacji z Redis, jest określony przez wartość opcji konfiguracyjnej `redis.client`, która zazwyczaj odzwierciedla wartość zmiennej środowiskowej `REDIS_CLIENT`:

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    // ...
],
```

Oprócz domyślnych opcji konfiguracyjnych, PhpRedis obsługuje następujące dodatkowe parametry połączenia: `name`, `persistent`, `persistent_id`, `prefix`, `read_timeout`, `retry_interval`, `max_retries`, `backoff_algorithm`, `backoff_base`, `backoff_cap`, `timeout` i `context`. Możesz dodać którekolwiek z tych opcji do konfiguracji serwera Redis w pliku konfiguracyjnym `config/database.php`:

```php
'default' => [
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'read_timeout' => 60,
    'context' => [
        // 'auth' => ['username', 'secret'],
        // 'stream' => ['verify_peer' => false],
    ],
],
```

<a name="retry-and-backoff-configuration"></a>
#### Konfiguracja Ponownych Prób i Backoff

Opcje `retry_interval`, `max_retries`, `backoff_algorithm`, `backoff_base` i `backoff_cap` mogą zostać użyte do skonfigurowania, jak klient PhpRedis powinien próbować ponownie połączyć się z serwerem Redis. Obsługiwane są następujące algorytmy backoff: `default`, `decorrelated_jitter`, `equal_jitter`, `exponential`, `uniform` i `constant`:

```php
'default' => [
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'max_retries' => env('REDIS_MAX_RETRIES', 3),
    'backoff_algorithm' => env('REDIS_BACKOFF_ALGORITHM', 'decorrelated_jitter'),
    'backoff_base' => env('REDIS_BACKOFF_BASE', 100),
    'backoff_cap' => env('REDIS_BACKOFF_CAP', 1000),
],
```

<a name="unix-socket-connections"></a>
#### Połączenia Unix Socket

Połączenia Redis mogą również być skonfigurowane do używania gniazd Unix zamiast TCP. Może to zapewnić lepszą wydajność poprzez wyeliminowanie narzutu TCP dla połączeń z instancjami Redis na tym samym serwerze co twoja aplikacja. Aby skonfigurować Redis do używania gniazda Unix, ustaw zmienną środowiskową `REDIS_HOST` na ścieżkę gniazda Redis, a zmienną środowiskową `REDIS_PORT` na `0`:

```env
REDIS_HOST=/run/redis/redis.sock
REDIS_PORT=0
```

<a name="phpredis-serialization"></a>
#### Serializacja i Kompresja PhpRedis

Rozszerzenie PhpRedis może również być skonfigurowane do używania różnych serializatorów i algorytmów kompresji. Te algorytmy mogą zostać skonfigurowane za pomocą tablicy `options` twojej konfiguracji Redis:

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        'serializer' => Redis::SERIALIZER_MSGPACK,
        'compression' => Redis::COMPRESSION_LZ4,
    ],

    // ...
],
```

Aktualnie obsługiwane serializatory obejmują: `Redis::SERIALIZER_NONE` (domyślny), `Redis::SERIALIZER_PHP`, `Redis::SERIALIZER_JSON`, `Redis::SERIALIZER_IGBINARY` i `Redis::SERIALIZER_MSGPACK`.

Obsługiwane algorytmy kompresji obejmują: `Redis::COMPRESSION_NONE` (domyślny), `Redis::COMPRESSION_LZF`, `Redis::COMPRESSION_ZSTD` i `Redis::COMPRESSION_LZ4`.

<a name="interacting-with-redis"></a>
## Interakcja z Redis

Możesz wchodzić w interakcję z Redis, wywołując różne metody na [fasadzie](/docs/{{version}}/facades) `Redis`. Fasada `Redis` obsługuje metody dynamiczne, co oznacza, że możesz wywoływać dowolne [polecenia Redis](https://redis.io/commands) na fasadzie, a polecenie zostanie przekazane bezpośrednio do Redis. W tym przykładzie wywołamy polecenie Redis `GET`, wywołując metodę `get` na fasadzie `Redis`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Redis;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Pokaż profil dla danego użytkownika.
     */
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => Redis::get('user:profile:'.$id)
        ]);
    }
}
```

Jak wspomniano powyżej, możesz wywoływać dowolne polecenia Redis na fasadzie `Redis`. Laravel używa metod magicznych do przekazywania komend do serwera Redis. Jeśli polecenie Redis oczekuje argumentów, powinieneś przekazać je do odpowiedniej metody fasady:

```php
use Illuminate\Support\Facades\Redis;

Redis::set('name', 'Taylor');

$values = Redis::lrange('names', 5, 10);
```

Alternatywnie możesz przekazywać polecenia do serwera za pomocą metody `command` fasady `Redis`, która przyjmuje nazwę polecenia jako pierwszy argument i tablicę wartości jako drugi argument:

```php
$values = Redis::command('lrange', ['name', 5, 10]);
```

<a name="using-multiple-redis-connections"></a>
#### Używanie Wielu Połączeń Redis

Plik konfiguracyjny `config/database.php` twojej aplikacji pozwala zdefiniować wiele połączeń / serwerów Redis. Możesz uzyskać połączenie z konkretnym połączeniem Redis za pomocą metody `connection` fasady `Redis`:

```php
$redis = Redis::connection('connection-name');
```

Aby uzyskać instancję domyślnego połączenia Redis, możesz wywołać metodę `connection` bez żadnych dodatkowych argumentów:

```php
$redis = Redis::connection();
```

<a name="transactions"></a>
### Transakcje

Metoda `transaction` fasady `Redis` zapewnia wygodny wrapper wokół natywnych komend Redis `MULTI` i `EXEC`. Metoda `transaction` akceptuje closure jako jedyny argument. To closure otrzyma instancję połączenia Redis i może wydać dowolne polecenia do tej instancji. Wszystkie polecenia Redis wydane w ramach closure zostaną wykonane w jednej, atomowej transakcji:

```php
use Redis;
use Illuminate\Support\Facades;

Facades\Redis::transaction(function (Redis $redis) {
    $redis->incr('user_visits', 1);
    $redis->incr('total_visits', 1);
});
```

> [!WARNING]
> Podczas definiowania transakcji Redis nie możesz pobierać żadnych wartości z połączenia Redis. Pamiętaj, że twoja transakcja jest wykonywana jako jedna, atomowa operacja i ta operacja nie jest wykonywana, dopóki całe closure nie zakończy wykonywania swoich komend.

#### Skrypty Lua

Metoda `eval` zapewnia inną metodę wykonywania wielu komend Redis w jednej, atomowej operacji. Jednak metoda `eval` ma tę zaletę, że może wchodzić w interakcję i sprawdzać wartości kluczy Redis podczas tej operacji. Skrypty Redis są napisane w [języku programowania Lua](https://www.lua.org).

Metoda `eval` może wydawać się na początku trochę przestraszająca, ale zbadamy podstawowy przykład, aby przerwać lody. Metoda `eval` oczekuje kilku argumentów. Najpierw powinieneś przekazać skrypt Lua (jako ciąg znaków) do metody. Po drugie, powinieneś przekazać liczbę kluczy (jako liczbę całkowitą), z którymi wchodzi w interakcję skrypt. Po trzecie, powinieneś przekazać nazwy tych kluczy. Na koniec możesz przekazać wszelkie inne dodatkowe argumenty, których potrzebujesz, aby uzyskać dostęp w swoim skrypcie.

W tym przykładzie zwiększymy licznik, sprawdzimy jego nową wartość i zwiększymy drugi licznik, jeśli wartość pierwszego licznika jest większa niż pięć. Na koniec zwrócimy wartość pierwszego licznika:

```php
$value = Redis::eval(<<<'LUA'
    local counter = redis.call("incr", KEYS[1])

    if counter > 5 then
        redis.call("incr", KEYS[2])
    end

    return counter
LUA, 2, 'first-counter', 'second-counter');
```

> [!WARNING]
> Więcej informacji na temat skryptowania Redis znajdziesz w [dokumentacji Redis](https://redis.io/commands/eval).

<a name="pipelining-commands"></a>
### Przetwarzanie Potokowe Komend

Czasami możesz potrzebować wykonać dziesiątki komend Redis. Zamiast wykonywać połączenie sieciowe z serwerem Redis dla każdej komendy, możesz użyć metody `pipeline`. Metoda `pipeline` akceptuje jeden argument: closure, które otrzymuje instancję Redis. Możesz wydać wszystkie swoje polecenia do tej instancji Redis i wszystkie zostaną wysłane do serwera Redis w tym samym czasie, aby zmniejszyć połączenia sieciowe z serwerem. Polecenia zostaną nadal wykonane w kolejności, w jakiej zostały wydane:

```php
use Redis;
use Illuminate\Support\Facades;

Facades\Redis::pipeline(function (Redis $pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```

<a name="pubsub"></a>
## Pub / Sub

Laravel zapewnia wygodny interfejs do komend Redis `publish` i `subscribe`. Te polecenia Redis pozwalają nasłuchiwać wiadomości na danym "kanale". Możesz publikować wiadomości na kanale z innej aplikacji, a nawet używając innego języka programowania, umożliwiając łatwą komunikację między aplikacjami i procesami.

Najpierw skonfigurujmy nasłuchiwanie kanału za pomocą metody `subscribe`. Umieścimy to wywołanie metody w [poleceniu Artisan](/docs/{{version}}/artisan), ponieważ wywołanie metody `subscribe` rozpoczyna długotrwały proces:

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Redis;

class RedisSubscribe extends Command
{
    /**
     * Nazwa i sygnatura komendy konsolowej.
     *
     * @var string
     */
    protected $signature = 'redis:subscribe';

    /**
     * Opis komendy konsolowej.
     *
     * @var string
     */
    protected $description = 'Subskrybuj kanał Redis';

    /**
     * Wykonaj komendę konsolową.
     */
    public function handle(): void
    {
        Redis::subscribe(['test-channel'], function (string $message) {
            echo $message;
        });
    }
}
```

Teraz możemy publikować wiadomości na kanale za pomocą metody `publish`:

```php
use Illuminate\Support\Facades\Redis;

Route::get('/publish', function () {
    // ...

    Redis::publish('test-channel', json_encode([
        'name' => 'Adam Wathan'
    ]));
});
```

<a name="wildcard-subscriptions"></a>
#### Subskrypcje Wieloznaczne

Używając metody `psubscribe`, możesz subskrybować kanał wieloznaczny, co może być przydatne do przechwytywania wszystkich wiadomości na wszystkich kanałach. Nazwa kanału zostanie przekazana jako drugi argument do dostarczonego closure:

```php
Redis::psubscribe(['*'], function (string $message, string $channel) {
    echo $message;
});

Redis::psubscribe(['users.*'], function (string $message, string $channel) {
    echo $message;
});
```
