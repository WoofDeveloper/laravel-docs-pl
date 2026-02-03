# Laravel Scout

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
    - [Kolejkowanie](#queueing)
- [Wymagania sterowników](#driver-prerequisites)
    - [Algolia](#algolia)
    - [Meilisearch](#meilisearch)
    - [Typesense](#typesense)
- [Konfiguracja](#configuration)
    - [Konfigurowanie indeksów modeli](#configuring-model-indexes)
    - [Konfigurowanie danych przeszukiwalnych](#configuring-searchable-data)
    - [Konfigurowanie ID modelu](#configuring-the-model-id)
    - [Konfigurowanie silników wyszukiwania dla modelu](#configuring-search-engines-per-model)
    - [Identyfikowanie użytkowników](#identifying-users)
- [Silniki bazy danych / kolekcji](#database-and-collection-engines)
    - [Silnik bazy danych](#database-engine)
    - [Silnik kolekcji](#collection-engine)
- [Indeksowanie](#indexing)
    - [Import wsadowy](#batch-import)
    - [Dodawanie rekordów](#adding-records)
    - [Aktualizowanie rekordów](#updating-records)
    - [Usuwanie rekordów](#removing-records)
    - [Wstrzymywanie indeksowania](#pausing-indexing)
    - [Warunkowo przeszukiwalne instancje modeli](#conditionally-searchable-model-instances)
- [Wyszukiwanie](#searching)
    - [Klauzule where](#where-clauses)
    - [Paginacja](#pagination)
    - [Usuwanie miękkie](#soft-deleting)
    - [Dostosowywanie wyszukiwania silnika](#customizing-engine-searches)
- [Własne silniki](#custom-engines)

<a name="introduction"></a>
## Wprowadzenie

[Laravel Scout](https://github.com/laravel/scout) zapewnia proste, oparte na sterownikach rozwiązanie do dodawania wyszukiwania pełnotekstowego do Twoich [modeli Eloquent](/docs/{{version}}/eloquent). Używając obserwatorów modeli, Scout automatycznie synchronizuje Twoje indeksy wyszukiwania z rekordami Eloquent.

Obecnie Scout jest dostarczany ze sterownikami [Algolia](https://www.algolia.com/), [Meilisearch](https://www.meilisearch.com), [Typesense](https://typesense.org) oraz MySQL / PostgreSQL (`database`). Ponadto Scout zawiera sterownik "collection", który jest przeznaczony do użytku w lokalnym środowisku deweloperskim i nie wymaga żadnych zewnętrznych zależności ani usług stron trzecich. Co więcej, pisanie własnych sterowników jest proste i możesz swobodnie rozszerzać Scout o własne implementacje wyszukiwania.

<a name="installation"></a>
## Instalacja

Najpierw zainstaluj Scout za pomocą menedżera pakietów Composer:

```shell
composer require laravel/scout
```

Po zainstalowaniu Scout powinieneś opublikować plik konfiguracyjny Scout za pomocą polecenia Artisan `vendor:publish`. To polecenie opublikuje plik konfiguracyjny `scout.php` w katalogu `config` Twojej aplikacji:

```shell
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```

Na koniec dodaj trait `Laravel\Scout\Searchable` do modelu, który chcesz uczynić przeszukiwalnym. Ten trait zarejestruje obserwatora modelu, który automatycznie będzie synchronizował model z Twoim sterownikiem wyszukiwania:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;
}
```

<a name="queueing"></a>
### Kolejkowanie

Podczas korzystania z silnika innego niż `database` lub `collection`, powinieneś zdecydowanie rozważyć skonfigurowanie [sterownika kolejek](/docs/{{version}}/queues) przed użyciem biblioteki. Uruchomienie workera kolejek pozwoli Scout kolejkować wszystkie operacje synchronizujące informacje o Twoich modelach z indeksami wyszukiwania, zapewniając znacznie lepsze czasy odpowiedzi dla interfejsu internetowego Twojej aplikacji.

Po skonfigurowaniu sterownika kolejek ustaw wartość opcji `queue` w pliku konfiguracyjnym `config/scout.php` na `true`:

```php
'queue' => true,
```

Nawet gdy opcja `queue` jest ustawiona na `false`, ważne jest, aby pamiętać, że niektóre sterowniki Scout, takie jak Algolia i Meilisearch, zawsze indeksują rekordy asynchronicznie. Oznacza to, że nawet jeśli operacja indeksowania została zakończona w Twojej aplikacji Laravel, sam silnik wyszukiwania może nie odzwierciedlać nowych i zaktualizowanych rekordów natychmiast.

Aby określić połączenie i kolejkę, z których korzystają zadania Scout, możesz zdefiniować opcję konfiguracyjną `queue` jako tablicę:

```php
'queue' => [
    'connection' => 'redis',
    'queue' => 'scout'
],
```

Oczywiście, jeśli dostosujesz połączenie i kolejkę, z których korzystają zadania Scout, powinieneś uruchomić workera kolejek, aby przetwarzał zadania na tym połączeniu i w tej kolejce:

```shell
php artisan queue:work redis --queue=scout
```

<a name="driver-prerequisites"></a>
## Wymagania sterowników

<a name="algolia"></a>
### Algolia

Podczas korzystania ze sterownika Algolia powinieneś skonfigurować swoje dane uwierzytelniające Algolia `id` i `secret` w pliku konfiguracyjnym `config/scout.php`. Po skonfigurowaniu danych uwierzytelniających będziesz musiał również zainstalować Algolia PHP SDK za pomocą menedżera pakietów Composer:

```shell
composer require algolia/algoliasearch-client-php
```

<a name="meilisearch"></a>
### Meilisearch

[Meilisearch](https://www.meilisearch.com) to błyskawicznie szybki silnik wyszukiwania o otwartym kodzie źródłowym. Jeśli nie masz pewności, jak zainstalować Meilisearch na swoim lokalnym komputerze, możesz użyć [Laravel Sail](/docs/{{version}}/sail#meilisearch), oficjalnie wspieranego przez Laravel środowiska deweloperskiego Docker.

Podczas korzystania ze sterownika Meilisearch będziesz musiał zainstalować Meilisearch PHP SDK za pomocą menedżera pakietów Composer:

```shell
composer require meilisearch/meilisearch-php http-interop/http-factory-guzzle
```

Następnie ustaw zmienną środowiskową `SCOUT_DRIVER`, a także dane uwierzytelniające `host` i `key` Meilisearch w pliku `.env` Twojej aplikacji:

```ini
SCOUT_DRIVER=meilisearch
MEILISEARCH_HOST=http://127.0.0.1:7700
MEILISEARCH_KEY=masterKey
```

Aby uzyskać więcej informacji na temat Meilisearch, zapoznaj się z [dokumentacją Meilisearch](https://docs.meilisearch.com/learn/getting_started/quick_start.html).

Ponadto powinieneś upewnić się, że instalujesz wersję `meilisearch/meilisearch-php` kompatybilną z Twoją wersją binarną Meilisearch, przeglądając [dokumentację Meilisearch dotyczącą kompatybilności binarnej](https://github.com/meilisearch/meilisearch-php#-compatibility-with-meilisearch).

> [!WARNING]
> Podczas aktualizacji Scout w aplikacji korzystającej z Meilisearch zawsze powinieneś [przejrzeć wszelkie dodatkowe zmiany łamiące](https://github.com/meilisearch/Meilisearch/releases) w samej usłudze Meilisearch.

<a name="typesense"></a>
### Typesense

[Typesense](https://typesense.org) to błyskawiczny silnik wyszukiwania o otwartym kodzie źródłowym, który obsługuje wyszukiwanie słów kluczowych, wyszukiwanie semantyczne, wyszukiwanie geograficzne i wyszukiwanie wektorowe.

Możesz [samodzielnie hostować](https://typesense.org/docs/guide/install-typesense.html#option-2-local-machine-self-hosting) Typesense lub użyć [Typesense Cloud](https://cloud.typesense.org).

Aby rozpocząć korzystanie z Typesense ze Scout, zainstaluj Typesense PHP SDK za pomocą menedżera pakietów Composer:

```shell
composer require typesense/typesense-php
```

Następnie ustaw zmienną środowiskową `SCOUT_DRIVER`, a także dane uwierzytelniające hosta i klucza API Typesense w pliku .env Twojej aplikacji:

```ini
SCOUT_DRIVER=typesense
TYPESENSE_API_KEY=masterKey
TYPESENSE_HOST=localhost
```

Jeśli korzystasz z [Laravel Sail](/docs/{{version}}/sail), możesz potrzebować dostosować zmienną środowiskową `TYPESENSE_HOST`, aby odpowiadała nazwie kontenera Docker. Możesz również opcjonalnie określić port, ścieżkę i protokół instalacji:

```ini
TYPESENSE_PORT=8108
TYPESENSE_PATH=
TYPESENSE_PROTOCOL=http
```

Dodatkowe ustawienia i definicje schematu dla kolekcji Typesense można znaleźć w pliku konfiguracyjnym `config/scout.php` Twojej aplikacji. Aby uzyskać więcej informacji na temat Typesense, zapoznaj się z [dokumentacją Typesense](https://typesense.org/docs/guide/#quick-start).

<a name="preparing-data-for-storage-in-typesense"></a>
#### Przygotowywanie danych do przechowywania w Typesense

Podczas korzystania z Typesense Twoje przeszukiwalne modele muszą definiować metodę `toSearchableArray`, która rzutuje klucz podstawowy modelu na ciąg znaków, a datę utworzenia na znacznik czasu UNIX:

```php
/**
 * Get the indexable data array for the model.
 *
 * @return array<string, mixed>
 */
public function toSearchableArray(): array
{
    return array_merge($this->toArray(),[
        'id' => (string) $this->id,
        'created_at' => $this->created_at->timestamp,
    ]);
}
```

Powinieneś również zdefiniować schematy kolekcji Typesense w pliku `config/scout.php` Twojej aplikacji. Schemat kolekcji opisuje typy danych każdego pola, które można przeszukiwać za pomocą Typesense. Aby uzyskać więcej informacji na temat wszystkich dostępnych opcji schematu, zapoznaj się z [dokumentacją Typesense](https://typesense.org/docs/latest/api/collections.html#schema-parameters).

Jeśli musisz zmienić schemat kolekcji Typesense po jego zdefiniowaniu, możesz uruchomić `scout:flush` i `scout:import`, co usunie wszystkie istniejące zaindeksowane dane i ponownie utworzy schemat. Możesz też użyć API Typesense do modyfikacji schematu kolekcji bez usuwania żadnych zaindeksowanych danych.

Jeśli Twój przeszukiwalny model obsługuje usuwanie miękkie, powinieneś zdefiniować pole `__soft_deleted` w odpowiednim schemacie Typesense modelu w pliku konfiguracyjnym `config/scout.php` Twojej aplikacji:

```php
User::class => [
    'collection-schema' => [
        'fields' => [
            // ...
            [
                'name' => '__soft_deleted',
                'type' => 'int32',
                'optional' => true,
            ],
        ],
    ],
],
```

<a name="typesense-dynamic-search-parameters"></a>
#### Dynamiczne parametry wyszukiwania

Typesense umożliwia dynamiczną modyfikację [parametrów wyszukiwania](https://typesense.org/docs/latest/api/search.html#search-parameters) podczas wykonywania operacji wyszukiwania za pomocą metody `options`:

```php
use App\Models\Todo;

Todo::search('Groceries')->options([
    'query_by' => 'title, description'
])->get();
```

<a name="configuration"></a>
## Konfiguracja

<a name="configuring-model-indexes"></a>
### Konfigurowanie indeksów modeli

Każdy model Eloquent jest synchronizowany z danym "indeksem" wyszukiwania, który zawiera wszystkie przeszukiwalne rekordy dla tego modelu. Innymi słowy, możesz myśleć o każdym indeksie jak o tabeli MySQL. Domyślnie każdy model będzie przechowywany w indeksie odpowiadającym typowej nazwie "tabeli" modelu. Zwykle jest to forma mnoga nazwy modelu; możesz jednak swobodnie dostosować indeks modelu, nadpisując metodę `searchableAs` w modelu:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;

    /**
     * Get the name of the index associated with the model.
     */
    public function searchableAs(): string
    {
        return 'posts_index';
    }
}
```

<a name="configuring-searchable-data"></a>
### Konfigurowanie danych przeszukiwalnych

Domyślnie cała forma `toArray` danego modelu będzie przechowywana w jego indeksie wyszukiwania. Jeśli chcesz dostosować dane synchronizowane z indeksem wyszukiwania, możesz nadpisać metodę `toSearchableArray` w modelu:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;

    /**
     * Get the indexable data array for the model.
     *
     * @return array<string, mixed>
     */
    public function toSearchableArray(): array
    {
        $array = $this->toArray();

        // Customize the data array...

        return $array;
    }
}
```

Niektóre silniki wyszukiwania, takie jak Meilisearch, wykonują operacje filtrowania (`>`, `<`, itp.) tylko na danych odpowiedniego typu. Dlatego podczas korzystania z tych silników wyszukiwania i dostosowywania przeszukiwalnych danych powinieneś upewnić się, że wartości numeryczne są rzutowane na właściwy typ:

```php
public function toSearchableArray()
{
    return [
        'id' => (int) $this->id,
        'name' => $this->name,
        'price' => (float) $this->price,
    ];
}
```

<a name="configuring-indexes-for-algolia"></a>
#### Konfigurowanie ustawień indeksu (Algolia)

Czasami możesz chcieć skonfigurować dodatkowe ustawienia dla indeksów Algolia. Chociaż możesz zarządzać tymi ustawieniami za pośrednictwem interfejsu Algolia, czasami bardziej efektywne jest zarządzanie pożądanym stanem konfiguracji indeksu bezpośrednio z pliku konfiguracyjnego `config/scout.php` Twojej aplikacji.

To podejście pozwala wdrożyć te ustawienia za pomocą automatycznego potoku wdrożenia aplikacji, unikając ręcznej konfiguracji i zapewniając spójność w wielu środowiskach. Możesz skonfigurować atrybuty filtrowalne, ranking, fasetowanie lub [dowolne inne obsługiwane ustawienia](https://www.algolia.com/doc/rest-api/search/#tag/Indices/operation/setSettings).

Aby rozpocząć, dodaj ustawienia dla każdego indeksu w pliku konfiguracyjnym `config/scout.php` Twojej aplikacji:

```php
use App\Models\User;
use App\Models\Flight;

'algolia' => [
    'id' => env('ALGOLIA_APP_ID', ''),
    'secret' => env('ALGOLIA_SECRET', ''),
    'index-settings' => [
        User::class => [
            'searchableAttributes' => ['id', 'name', 'email'],
            'attributesForFaceting'=> ['filterOnly(email)'],
            // Other settings fields...
        ],
        Flight::class => [
            'searchableAttributes'=> ['id', 'destination'],
        ],
    ],
],
```

Jeśli model leżący u podstaw danego indeksu obsługuje usuwanie miękkie i jest zawarty w tablicy `index-settings`, Scout automatycznie dołączy wsparcie dla fasetowania dla modeli usuniętych miękkow na tym indeksie. Jeśli nie masz innych atrybutów fasetowania do zdefiniowania dla indeksu modelu z usuwaniem miękkim, możesz po prostu dodać pusty wpis do tablicy `index-settings` dla tego modelu:

```php
'index-settings' => [
    Flight::class => []
],
```

Po skonfigurowaniu ustawień indeksu aplikacji musisz wywołać polecenie Artisan `scout:sync-index-settings`. To polecenie poinformuje Algolię o aktualnie skonfigurowanych ustawieniach indeksu. Dla wygody możesz uczynić to polecenie częścią procesu wdrażania:

```shell
php artisan scout:sync-index-settings
```

<a name="configuring-filterable-data-for-meilisearch"></a>
#### Konfigurowanie danych filtrowalnych i ustawień indeksu (Meilisearch)

W przeciwieństwie do innych sterowników Scout, Meilisearch wymaga wstępnego zdefiniowania ustawień wyszukiwania indeksu, takich jak atrybuty filtrowalne, atrybuty sortowalne i [inne obsługiwane pola ustawień](https://docs.meilisearch.com/reference/api/settings.html).

Atrybuty filtrowalne to wszelkie atrybuty, które planujesz filtrować podczas wywoływania metody `where` Scout, podczas gdy atrybuty sortowalne to wszelkie atrybuty, według których planujesz sortować podczas wywoływania metody `orderBy` Scout. Aby zdefiniować ustawienia indeksu, dostosuj część `index-settings` wpisu konfiguracyjnego `meilisearch` w pliku konfiguracyjnym `scout` Twojej aplikacji:

```php
use App\Models\User;
use App\Models\Flight;

'meilisearch' => [
    'host' => env('MEILISEARCH_HOST', 'http://localhost:7700'),
    'key' => env('MEILISEARCH_KEY', null),
    'index-settings' => [
        User::class => [
            'filterableAttributes'=> ['id', 'name', 'email'],
            'sortableAttributes' => ['created_at'],
            // Other settings fields...
        ],
        Flight::class => [
            'filterableAttributes'=> ['id', 'destination'],
            'sortableAttributes' => ['updated_at'],
        ],
    ],
],
```

Jeśli model leżący u podstaw danego indeksu obsługuje usuwanie miękkie i jest zawarty w tablicy `index-settings`, Scout automatycznie dołączy wsparcie dla filtrowania modeli usuniętych miękko na tym indeksie. Jeśli nie masz innych atrybutów filtrowalnych ani sortowalnych do zdefiniowania dla indeksu modelu z usuwaniem miękkim, możesz po prostu dodać pusty wpis do tablicy `index-settings` dla tego modelu:

```php
'index-settings' => [
    Flight::class => []
],
```

Po skonfigurowaniu ustawień indeksu aplikacji musisz wywołać polecenie Artisan `scout:sync-index-settings`. To polecenie poinformuje Meilisearch o aktualnie skonfigurowanych ustawieniach indeksu. Dla wygody możesz uczynić to polecenie częścią procesu wdrażania:

```shell
php artisan scout:sync-index-settings
```

<a name="configuring-the-model-id"></a>
### Konfigurowanie ID modelu

Domyślnie Scout użyje klucza podstawowego modelu jako unikalnego ID/klucza modelu przechowywanego w indeksie wyszukiwania. Jeśli musisz dostosować to zachowanie, możesz nadpisać metody `getScoutKey` i `getScoutKeyName` w modelu:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class User extends Model
{
    use Searchable;

    /**
     * Get the value used to index the model.
     */
    public function getScoutKey(): mixed
    {
        return $this->email;
    }

    /**
     * Get the key name used to index the model.
     */
    public function getScoutKeyName(): mixed
    {
        return 'email';
    }
}
```

<a name="configuring-search-engines-per-model"></a>
### Konfigurowanie silników wyszukiwania dla modelu

Podczas wyszukiwania Scout zazwyczaj używa domyślnego silnika wyszukiwania określonego w pliku konfiguracyjnym `scout` Twojej aplikacji. Jednak silnik wyszukiwania dla konkretnego modelu można zmienić, nadpisując metodę `searchableUsing` w modelu:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Engines\Engine;
use Laravel\Scout\Scout;
use Laravel\Scout\Searchable;

class User extends Model
{
    use Searchable;

    /**
     * Get the engine used to index the model.
     */
    public function searchableUsing(): Engine
    {
        return Scout::engine('meilisearch');
    }
}
```

<a name="identifying-users"></a>
### Identyfikowanie użytkowników

Scout umożliwia również automatyczną identyfikację użytkowników podczas korzystania z [Algolia](https://algolia.com). Powiązanie uwierzytelnionego użytkownika z operacjami wyszukiwania może być pomocne podczas przeglądania analiz wyszukiwania w panelu Algolia. Możesz włączyć identyfikację użytkownika, definiując zmienną środowiskową `SCOUT_IDENTIFY` jako `true` w pliku `.env` Twojej aplikacji:

```ini
SCOUT_IDENTIFY=true
```

Włączenie tej funkcji spowoduje również przekazanie adresu IP żądania i głównego identyfikatora uwierzytelnionego użytkownika do Algolii, dzięki czemu te dane będą powiązane z każdym żądaniem wyszukiwania wykonywanym przez użytkownika.

<a name="database-and-collection-engines"></a>
## Silniki bazy danych / kolekcji

<a name="database-engine"></a>
### Silnik bazy danych

> [!WARNING]
> Silnik bazy danych obecnie obsługuje MySQL i PostgreSQL.

Silnik `database` to najszybszy sposób na rozpoczęcie pracy z Laravel Scout i wykorzystuje indeksy pełnotekstowe MySQL/PostgreSQL oraz klauzule "where like" podczas filtrowania wyników z istniejącej bazy danych w celu określenia odpowiednich wyników wyszukiwania dla Twojego zapytania.

Aby użyć silnika bazy danych, możesz po prostu ustawić wartość zmiennej środowiskowej `SCOUT_DRIVER` na `database` lub określić sterownik `database` bezpośrednio w pliku konfiguracyjnym `scout` Twojej aplikacji:

```ini
SCOUT_DRIVER=database
```

Po określeniu silnika bazy danych jako preferowanego sterownika musisz [skonfigurować przeszukiwalne dane](#configuring-searchable-data). Następnie możesz rozpocząć [wykonywanie zapytań wyszukiwania](#searching) względem swoich modeli. Indeksowanie silnika wyszukiwania, takie jak indeksowanie potrzebne do zasiewania indeksów Algolia, Meilisearch lub Typesense, jest niepotrzebne podczas korzystania z silnika bazy danych.

#### Dostosowywanie strategii wyszukiwania w bazie danych

Domyślnie silnik bazy danych wykona zapytanie "where like" względem każdego atrybutu modelu, który [skonfigurowałeś jako przeszukiwalny](#configuring-searchable-data). Jednak w niektórych sytuacjach może to prowadzić do słabej wydajności. Dlatego strategia wyszukiwania silnika bazy danych może być skonfigurowana tak, aby niektóre określone kolumny wykorzystywały zapytania wyszukiwania pełnotekstowego lub używały tylko ograniczeń "where like" do przeszukiwania prefiksów ciągów (`example%`) zamiast przeszukiwania w całym ciągu (`%example%`).

Aby zdefiniować to zachowanie, możesz przypisać atrybuty PHP do metody `toSearchableArray` modelu. Wszystkie kolumny, którym nie przypisano dodatkowego zachowania strategii wyszukiwania, będą nadal używać domyślnej strategii "where like":

```php
use Laravel\Scout\Attributes\SearchUsingFullText;
use Laravel\Scout\Attributes\SearchUsingPrefix;

/**
 * Get the indexable data array for the model.
 *
 * @return array<string, mixed>
 */
#[SearchUsingPrefix(['id', 'email'])]
#[SearchUsingFullText(['bio'])]
public function toSearchableArray(): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'bio' => $this->bio,
    ];
}
```

> [!WARNING]
> Przed określeniem, że kolumna powinna używać ograniczeń zapytania pełnotekstowego, upewnij się, że kolumna ma przypisany [indeks pełnotekstowy](/docs/{{version}}/migrations#available-index-types).

<a name="collection-engine"></a>
### Silnik kolekcji

Chociaż możesz swobodnie używać silników wyszukiwania Algolia, Meilisearch lub Typesense podczas lokalnego rozwoju, możesz uznać za wygodniejsze rozpoczęcie od silnika "collection". Silnik collection użyje klauzul "where" i filtrowania kolekcji wyników z istniejącej bazy danych, aby określić odpowiednie wyniki wyszukiwania dla Twojego zapytania. Podczas korzystania z tego silnika nie jest konieczne "indeksowanie" przeszukiwalnych modeli, ponieważ zostaną one po prostu pobrane z lokalnej bazy danych.

Aby użyć silnika collection, możesz po prostu ustawić wartość zmiennej środowiskowej `SCOUT_DRIVER` na `collection` lub określić sterownik `collection` bezpośrednio w pliku konfiguracyjnym `scout` Twojej aplikacji:

```ini
SCOUT_DRIVER=collection
```

Po określeniu sterownika collection jako preferowanego sterownika możesz rozpocząć [wykonywanie zapytań wyszukiwania](#searching) względem swoich modeli. Indeksowanie silnika wyszukiwania, takie jak indeksowanie potrzebne do zasiewania indeksów Algolia, Meilisearch lub Typesense, jest niepotrzebne podczas korzystania z silnika collection.

#### Różnice w stosunku do silnika bazy danych

Na pierwszy rzut oka silniki "database" i "collections" są dość podobne. Oba współdziałają bezpośrednio z bazą danych, aby pobrać wyniki wyszukiwania. Jednak silnik collection nie wykorzystuje indeksów pełnotekstowych ani klauzul `LIKE` do znajdowania pasujących rekordów. Zamiast tego pobiera wszystkie możliwe rekordy i używa helpera `Str::is` Laravel, aby określić, czy ciąg wyszukiwania istnieje w wartościach atrybutów modelu.

Silnik collection jest najbardziej przenośnym silnikiem wyszukiwania, ponieważ działa we wszystkich relacyjnych bazach danych obsługiwanych przez Laravel (w tym SQLite i SQL Server); jednak jest znacznie mniej wydajny niż silnik bazy danych Scout.

<a name="indexing"></a>
## Indeksowanie

<a name="batch-import"></a>
### Import wsadowy

Jeśli instalujesz Scout w istniejącym projekcie, możesz już mieć rekordy bazy danych, które musisz zaimportować do swoich indeksów. Scout zapewnia polecenie Artisan `scout:import`, które możesz użyć do zaimportowania wszystkich istniejących rekordów do indeksów wyszukiwania:

```shell
php artisan scout:import "App\Models\Post"
```

Polecenie `scout:queue-import` może być użyte do zaimportowania wszystkich istniejących rekordów przy użyciu [kolejkowanych zadań](/docs/{{version}}/queues):

```shell
php artisan scout:queue-import "App\Models\Post" --chunk=500
```

Polecenie `flush` może być użyte do usunięcia wszystkich rekordów modelu z indeksów wyszukiwania:

```shell
php artisan scout:flush "App\Models\Post"
```

<a name="modifying-the-import-query"></a>
#### Modyfikowanie zapytania importu

Jeśli chcesz zmodyfikować zapytanie używane do pobierania wszystkich modeli do importu wsadowego, możesz zdefiniować metodę `makeAllSearchableUsing` w swoim modelu. To świetne miejsce na dodanie dowolnego ładowania relacji eager, które może być konieczne przed importem modeli:

```php
use Illuminate\Database\Eloquent\Builder;

/**
 * Modify the query used to retrieve models when making all of the models searchable.
 */
protected function makeAllSearchableUsing(Builder $query): Builder
{
    return $query->with('author');
}
```

> [!WARNING]
> Metoda `makeAllSearchableUsing` może nie mieć zastosowania podczas używania kolejki do wsadowego importu modeli. Relacje [nie są przywracane](/docs/{{version}}/queues#handling-relationships), gdy kolekcje modeli są przetwarzane przez zadania.

<a name="adding-records"></a>
### Dodawanie rekordów

Po dodaniu traitu `Laravel\Scout\Searchable` do modelu wszystko, co musisz zrobić, to wykonać `save` lub `create` instancji modelu, a zostanie ona automatycznie dodana do Twojego indeksu wyszukiwania. Jeśli skonfigurowałeś Scout do [używania kolejek](#queueing), ta operacja zostanie wykonana w tle przez Twojego workera kolejek:

```php
use App\Models\Order;

$order = new Order;

// ...

$order->save();
```

<a name="adding-records-via-query"></a>
#### Dodawanie rekordów via Query

Jeśli chcesz dodać kolekcję modeli do indeksu wyszukiwania za pomocą zapytania Eloquent, możesz połączyć metodę `searchable` z zapytaniem Eloquent. Metoda `searchable` [podzieli wyniki](/docs/{{version}}/eloquent#chunking-results) zapytania na fragmenty i doda rekordy do indeksu wyszukiwania. Ponownie, jeśli skonfigurowałeś Scout do używania kolejek, wszystkie fragmenty zostaną zaimportowane w tle przez Twoich workerów kolejek:

```php
use App\Models\Order;

Order::where('price', '>', 100)->searchable();
```

Możesz również wywołać metodę `searchable` na instancji relacji Eloquent:

```php
$user->orders()->searchable();
```

Lub, jeśli masz już kolekcję modeli Eloquent w pamięci, możesz wywołać metodę `searchable` na instancji kolekcji, aby dodać instancje modeli do ich odpowiednich indeksów:

```php
$orders->searchable();
```

> [!NOTE]
> Metoda `searchable` może być traktowana jako operacja "upsert". Innymi słowy, jeśli rekord modelu jest już w Twoim indeksie, zostanie zaktualizowany. Jeśli nie istnieje w indeksie wyszukiwania, zostanie dodany do indeksu.

<a name="updating-records"></a>
### Aktualizowanie rekordów

Aby zaktualizować przeszukiwalny model, musisz tylko zaktualizować właściwości instancji modelu i wykonać `save` modelu do swojej bazy danych. Scout automatycznie utrwali zmiany w Twoim indeksie wyszukiwania:

```php
use App\Models\Order;

$order = Order::find(1);

// Update the order...

$order->save();
```

Możesz również wywołać metodę `searchable` na instancji zapytania Eloquent, aby zaktualizować kolekcję modeli. Jeśli modele nie istnieją w Twoim indeksie wyszukiwania, zostaną utworzone:

```php
Order::where('price', '>', 100)->searchable();
```

Jeśli chcesz zaktualizować rekordy indeksu wyszukiwania dla wszystkich modeli w relacji, możesz wywołać `searchable` na instancji relacji:

```php
$user->orders()->searchable();
```

Lub, jeśli masz już kolekcję modeli Eloquent w pamięci, możesz wywołać metodę `searchable` na instancji kolekcji, aby zaktualizować instancje modeli w ich odpowiednich indeksach:

```php
$orders->searchable();
```

<a name="modifying-records-before-importing"></a>
#### Modyfikowanie rekordów przed importem

Czasami możesz potrzebować przygotować kolekcję modeli, zanim staną się przeszukiwalne. Na przykład możesz chcieć eager load relacji, aby dane relacji mogły być efektywnie dodane do Twojego indeksu wyszukiwania. Aby to osiągnąć, zdefiniuj metodę `makeSearchableUsing` w odpowiednim modelu:

```php
use Illuminate\Database\Eloquent\Collection;

/**
 * Modify the collection of models being made searchable.
 */
public function makeSearchableUsing(Collection $models): Collection
{
    return $models->load('author');
}
```

<a name="removing-records"></a>
### Usuwanie rekordów

Aby usunąć rekord z indeksu, możesz po prostu wykonać `delete` modelu z bazy danych. Można to zrobić nawet jeśli używasz modeli [usuwanych miękko](/docs/{{version}}/eloquent#soft-deleting):

```php
use App\Models\Order;

$order = Order::find(1);

$order->delete();
```

Jeśli nie chcesz pobierać modelu przed usunięciem rekordu, możesz użyć metody `unsearchable` na instancji zapytania Eloquent:

```php
Order::where('price', '>', 100)->unsearchable();
```

Jeśli chcesz usunąć rekordy indeksu wyszukiwania dla wszystkich modeli w relacji, możesz wywołać `unsearchable` na instancji relacji:

```php
$user->orders()->unsearchable();
```

Lub, jeśli masz już kolekcję modeli Eloquent w pamięci, możesz wywołać metodę `unsearchable` na instancji kolekcji, aby usunąć instancje modeli z ich odpowiednich indeksów:

```php
$orders->unsearchable();
```

Aby usunąć wszystkie rekordy modelu z ich odpowiednich indeksów, możesz wywołać metodę `removeAllFromSearch`:

```php
Order::removeAllFromSearch();
```

<a name="pausing-indexing"></a>
### Wstrzymywanie indeksowania

Czasami możesz potrzebować wykonać wsadową operację Eloquent na modelu bez synchronizacji danych modelu z indeksem wyszukiwania. Możesz to zrobić za pomocą metody `withoutSyncingToSearch`. Ta metoda akceptuje pojedyncze zamknięcie, które zostanie natychmiast wykonane. Wszelkie operacje na modelu, które wystąpią w zamknięciu, nie zostaną zsynchronizowane z indeksem modelu:

```php
use App\Models\Order;

Order::withoutSyncingToSearch(function () {
    // Perform model actions...
});
```

<a name="conditionally-searchable-model-instances"></a>
### Warunkowo przeszukiwalne instancje modeli

Czasami możesz potrzebować uczynić model przeszukiwalnym tylko pod pewnymi warunkami. Na przykład wyobraź sobie, że masz model `App\Models\Post`, który może być w jednym z dwóch stanów: "szkic" i "opublikowany". Możesz chcieć zezwolić tylko na przeszukiwanie postów "opublikowanych". Aby to osiągnąć, możesz zdefiniować metodę `shouldBeSearchable` w swoim modelu:

```php
/**
 * Determine if the model should be searchable.
 */
public function shouldBeSearchable(): bool
{
    return $this->isPublished();
}
```

Metoda `shouldBeSearchable` jest stosowana tylko podczas manipulowania modelami za pomocą metod `save` i `create`, zapytań lub relacji. Bezpośrednie czynienie modeli lub kolekcji przeszukiwalnymi za pomocą metody `searchable` nadpisze wynik metody `shouldBeSearchable`.

> [!WARNING]
> Metoda `shouldBeSearchable` nie ma zastosowania podczas korzystania z silnika "database" Scout, ponieważ wszystkie przeszukiwalne dane są zawsze przechowywane w bazie danych. Aby osiągnąć podobne zachowanie podczas korzystania z silnika bazy danych, powinieneś zamiast tego użyć [klauzul where](#where-clauses).

<a name="searching"></a>
## Wyszukiwanie

Możesz rozpocząć przeszukiwanie modelu za pomocą metody `search`. Metoda search akceptuje pojedynczy ciąg, który zostanie użyty do przeszukiwania Twoich modeli. Następnie powinieneś połączyć metodę `get` z zapytaniem wyszukiwania, aby pobrać modele Eloquent pasujące do danego zapytania wyszukiwania:

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->get();
```

Ponieważ wyszukiwania Scout zwracają kolekcję modeli Eloquent, możesz nawet zwrócić wyniki bezpośrednio z trasy lub kontrolera i zostaną one automatycznie przekonwertowane na JSON:

```php
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/search', function (Request $request) {
    return Order::search($request->search)->get();
});
```

Jeśli chcesz uzyskać surowe wyniki wyszukiwania, zanim zostaną przekonwertowane na modele Eloquent, możesz użyć metody `raw`:

```php
$orders = Order::search('Star Trek')->raw();
```

<a name="custom-indexes"></a>
#### Własne indeksy

Zapytania wyszukiwania będą zazwyczaj wykonywane na indeksie określonym przez metodę modelu [searchableAs](#configuring-model-indexes). Możesz jednak użyć metody `within`, aby określić niestandardowy indeks, który powinien zostać przeszukany zamiast tego:

```php
$orders = Order::search('Star Trek')
    ->within('tv_shows_popularity_desc')
    ->get();
```

<a name="where-clauses"></a>
### Klauzule where

Scout pozwala dodawać proste klauzule "where" do zapytań wyszukiwania. Obecnie te klauzule obsługują tylko podstawowe sprawdzanie równości numerycznej i są przede wszystkim przydatne do ograniczania zapytań wyszukiwania według ID właściciela:

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->where('user_id', 1)->get();
```

Dodatkowo metoda `whereIn` może być użyta do sprawdzenia, czy wartość danej kolumny jest zawarta w podanej tablicy:

```php
$orders = Order::search('Star Trek')->whereIn(
    'status', ['open', 'paid']
)->get();
```

Metoda `whereNotIn` sprawdza, czy wartość danej kolumny nie jest zawarta w podanej tablicy:

```php
$orders = Order::search('Star Trek')->whereNotIn(
    'status', ['closed']
)->get();
```

Ponieważ indeks wyszukiwania nie jest relacyjną bazą danych, bardziej zaawansowane klauzule "where" nie są obecnie obsługiwane.

> [!WARNING]
> Jeśli Twoja aplikacja używa Meilisearch, musisz skonfigurować [atrybuty filtrowalne](#configuring-filterable-data-for-meilisearch) swojej aplikacji przed wykorzystaniem klauzul "where" Scout.

<a name="pagination"></a>
### Paginacja

Oprócz pobierania kolekcji modeli możesz paginować wyniki wyszukiwania za pomocą metody `paginate`. Ta metoda zwróci instancję `Illuminate\Pagination\LengthAwarePaginator`, tak jak w przypadku [paginacji tradycyjnego zapytania Eloquent](/docs/{{version}}/pagination):

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->paginate();
```

Możesz określić, ile modeli pobrać na stronę, przekazując ilość jako pierwszy argument do metody `paginate`:

```php
$orders = Order::search('Star Trek')->paginate(15);
```

Po pobraniu wyników możesz wyświetlić wyniki i renderować linki stron za pomocą [Blade](/docs/{{version}}/blade), tak jak w przypadku paginacji tradycyjnego zapytania Eloquent:

```html
<div class="container">
    @foreach ($orders as $order)
        {{ $order->price }}
    @endforeach
</div>

{{ $orders->links() }}
```

Oczywiście, jeśli chcesz pobrać wyniki paginacji jako JSON, możesz zwrócić instancję paginatora bezpośrednio z trasy lub kontrolera:

```php
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/orders', function (Request $request) {
    return Order::search($request->input('query'))->paginate(15);
});
```

> [!WARNING]
> Ponieważ silniki wyszukiwania nie są świadome definicji globalnych zakresów modeli Eloquent, nie powinieneś używać globalnych zakresów w aplikacjach korzystających z paginacji Scout. Lub powinieneś odtworzyć ograniczenia globalnego zakresu podczas wyszukiwania za pomocą Scout.

<a name="soft-deleting"></a>
### Usuwanie miękkie

Jeśli Twoje zaindeksowane modele są [usuwane miękko](/docs/{{version}}/eloquent#soft-deleting) i musisz przeszukiwać modele usunięte miękko, ustaw opcję `soft_delete` w pliku konfiguracyjnym `config/scout.php` na `true`:

```php
'soft_delete' => true,
```

Gdy ta opcja konfiguracyjna jest `true`, Scout nie usunie modeli usuniętych miękko z indeksu wyszukiwania. Zamiast tego ustawi ukryty atrybut `__soft_deleted` w zaindeksowanym rekordzie. Następnie możesz użyć metod `withTrashed` lub `onlyTrashed`, aby pobrać rekordy usunięte miękko podczas wyszukiwania:

```php
use App\Models\Order;

// Include trashed records when retrieving results...
$orders = Order::search('Star Trek')->withTrashed()->get();

// Only include trashed records when retrieving results...
$orders = Order::search('Star Trek')->onlyTrashed()->get();
```

> [!NOTE]
> Gdy model usunięty miękko jest trwale usuwany za pomocą `forceDelete`, Scout automatycznie usunie go z indeksu wyszukiwania.

<a name="customizing-engine-searches"></a>
### Dostosowywanie wyszukiwania silnika

Jeśli musisz wykonać zaawansowane dostosowanie zachowania wyszukiwania silnika, możesz przekazać zamknięcie jako drugi argument do metody `search`. Na przykład możesz użyć tego callbacka, aby dodać dane geolokalizacyjne do opcji wyszukiwania, zanim zapytanie wyszukiwania zostanie przekazane do Algolii:

```php
use Algolia\AlgoliaSearch\SearchIndex;
use App\Models\Order;

Order::search(
    'Star Trek',
    function (SearchIndex $algolia, string $query, array $options) {
        $options['body']['query']['bool']['filter']['geo_distance'] = [
            'distance' => '1000km',
            'location' => ['lat' => 36, 'lon' => 111],
        ];

        return $algolia->search($query, $options);
    }
)->get();
```

<a name="customizing-the-eloquent-results-query"></a>
#### Dostosowywanie zapytania wyników Eloquent

Po tym, jak Scout pobierze listę pasujących modeli Eloquent z silnika wyszukiwania Twojej aplikacji, Eloquent jest używany do pobrania wszystkich pasujących modeli według ich kluczy podstawowych. Możesz dostosować to zapytanie, wywołując metodę `query`. Metoda `query` akceptuje zamknięcie, które otrzyma instancję konstruktora zapytań Eloquent jako argument:

```php
use App\Models\Order;
use Illuminate\Database\Eloquent\Builder;

$orders = Order::search('Star Trek')
    ->query(fn (Builder $query) => $query->with('invoices'))
    ->get();
```

Ponieważ ten callback jest wywoływany po tym, jak odpowiednie modele zostały już pobrane z silnika wyszukiwania Twojej aplikacji, metoda `query` nie powinna być używana do "filtrowania" wyników. Zamiast tego powinieneś użyć [klauzul where Scout](#where-clauses).

<a name="custom-engines"></a>
## Własne silniki

<a name="writing-the-engine"></a>
#### Pisanie silnika

Jeśli żaden z wbudowanych silników wyszukiwania Scout nie odpowiada Twoim potrzebom, możesz napisać własny niestandardowy silnik i zarejestrować go w Scout. Twój silnik powinien rozszerzać abstrakcyjną klasę `Laravel\Scout\Engines\Engine`. Ta abstrakcyjna klasa zawiera osiem metod, które Twój niestandardowy silnik musi zaimplementować:

```php
use Laravel\Scout\Builder;

abstract public function update($models);
abstract public function delete($models);
abstract public function search(Builder $builder);
abstract public function paginate(Builder $builder, $perPage, $page);
abstract public function mapIds($results);
abstract public function map(Builder $builder, $results, $model);
abstract public function getTotalCount($results);
abstract public function flush($model);
```

Możesz uznać za przydatne przejrzenie implementacji tych metod w klasie `Laravel\Scout\Engines\AlgoliaEngine`. Ta klasa zapewni Ci dobry punkt wyjścia do nauki, jak zaimplementować każdą z tych metod w swoim własnym silniku.

<a name="registering-the-engine"></a>
#### Rejestrowanie silnika

Po napisaniu własnego silnika możesz zarejestrować go w Scout za pomocą metody `extend` menedżera silników Scout. Menedżer silników Scout może być rozwiązany z kontenera usług Laravel. Powinieneś wywołać metodę `extend` z metody `boot` klasy `App\Providers\AppServiceProvider` lub dowolnego innego dostawcy usług używanego przez Twoją aplikację:

```php
use App\ScoutExtensions\MySqlSearchEngine;
use Laravel\Scout\EngineManager;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    resolve(EngineManager::class)->extend('mysql', function () {
        return new MySqlSearchEngine;
    });
}
```

Po zarejestrowaniu silnika możesz określić go jako domyślny `driver` Scout w pliku konfiguracyjnym `config/scout.php` Twojej aplikacji:

```php
'driver' => 'mysql',
```
