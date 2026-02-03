# Eloquent: Pierwsze Kroki

- [Wprowadzenie](#introduction)
- [Generowanie Klas Modeli](#generating-model-classes)
- [Konwencje Modelu Eloquent](#eloquent-model-conventions)
    - [Nazwy Tabel](#table-names)
    - [Klucze Główne](#primary-keys)
    - [Klucze UUID i ULID](#uuid-i-ulid-keys)
    - [Znaczniki Czasu](#timestamps)
    - [Połączenia z Bazą Danych](#database-connections)
    - [Domyślne Wartości Atrybutów](#default-attribute-values)
    - [Konfigurowanie Rygorystyczności Eloquent](#configuring-eloquent-strictness)
- [Pobieranie Modeli](#retrieving-models)
    - [Kolekcje](#collections)
    - [Dzielenie Wyników](#chunking-results)
    - [Dzielenie przy Użyciu Leniwych Kolekcji](#chunking-using-lazy-collections)
    - [Kursory](#cursors)
    - [Zaawansowane Podzapytania](#advanced-subqueries)
- [Pobieranie Pojedynczych Modeli / Agregatów](#retrieving-single-models)
    - [Pobieranie lub Tworzenie Modeli](#retrieving-or-creating-models)
    - [Pobieranie Agregatów](#retrieving-aggregates)
- [Wstawianie i Aktualizowanie Modeli](#inserting-and-updating-models)
    - [Wstawianie](#inserts)
    - [Aktualizowanie](#updates)
    - [Przypisanie Masowe](#mass-assignment)
    - [Upserty](#upserts)
- [Usuwanie Modeli](#deleting-models)
    - [Usuwanie Miękkie](#soft-deleting)
    - [Odpytywanie Miękko Usuniętych Modeli](#querying-soft-deleted-models)
- [Przycinanie Modeli](#pruning-models)
- [Replikowanie Modeli](#replicating-models)
- [Zakresy Zapytań](#query-scopes)
    - [Zakresy Globalne](#global-scopes)
    - [Zakresy Lokalne](#local-scopes)
    - [Oczekujące Atrybuty](#pending-attributes)
- [Porównywanie Modeli](#comparing-models)
- [Zdarzenia](#events)
    - [Używanie Zamknięć](#events-using-closures)
    - [Obserwatorzy](#observers)
    - [Wyciszanie Zdarzeń](#muting-events)

<a name="introduction"></a>
## Wprowadzenie

Laravel zawiera Eloquent, obiektowo-relacyjny mapper (ORM), który sprawia, że interakcja z bazą danych staje się przyjemna. Podczas używania Eloquent, każda tabela w bazie danych ma odpowiadający jej "Model", który służy do interakcji z tą tabelą. Oprócz pobierania rekordów z tabeli bazy danych, modele Eloquent pozwalają również na wstawianie, aktualizowanie i usuwanie rekordów z tabeli.

> [!NOTE]
> Przed rozpoczęciem upewnij się, że skonfigurowałeś połączenie z bazą danych w pliku konfiguracyjnym `config/database.php` swojej aplikacji. Aby uzyskać więcej informacji na temat konfigurowania bazy danych, sprawdź [dokumentację konfiguracji bazy danych](/docs/{{version}}/database#configuration).

<a name="generating-model-classes"></a>
## Generowanie Klas Modeli

Aby rozpocząć, utwórzmy model Eloquent. Modele zazwyczaj znajdują się w katalogu `app\Models` i rozszerzają klasę `Illuminate\Database\Eloquent\Model`. Możesz użyć [polecenia Artisan](/docs/{{version}}/artisan) `make:model`, aby wygenerować nowy model:

```shell
php artisan make:model Flight
```

Jeśli chcesz wygenerować [migrację bazy danych](/docs/{{version}}/migrations) podczas generowania modelu, możesz użyć opcji `--migration` lub `-m` :

```shell
php artisan make:model Flight --migration
```

Możesz wygenerować różne inne typy klas podczas generowania modelu, takie jak fabryki, seeder-y, polityki, kontrolery i żądania formularzy. Ponadto, te opcje mogą być łączone, aby utworzyć wiele klas jednocześnie:

```shell
# Wygeneruj model i klasę FlightFactory...
php artisan make:model Flight --factory
php artisan make:model Flight -f

# Wygeneruj model i klasę FlightSeeder...
php artisan make:model Flight --seed
php artisan make:model Flight -s

# Wygeneruj model i klasę FlightController...
php artisan make:model Flight --controller
php artisan make:model Flight -c

# Wygeneruj model, klasę zasobów FlightController oraz klasy żądań formularza...
php artisan make:model Flight --controller --resource --requests
php artisan make:model Flight -crR

# Wygeneruj model i klasę FlightPolicy...
php artisan make:model Flight --policy

# Wygeneruj model wraz z migracją, fabryką, seederem i kontrolerem...
php artisan make:model Flight -mfsc

# Skrót do wygenerowania modelu, migracji, fabryki, seedera, polityki, kontrolera i żądań formularza...
php artisan make:model Flight --all
php artisan make:model Flight -a

# Wygeneruj model piwotu...
php artisan make:model Member --pivot
php artisan make:model Member -p
```

<a name="inspecting-models"></a>
#### Inspekcja Modeli

Czasami może być trudno określić wszystkie dostępne atrybuty i relacje modelu tylko poprzez przeglądanie jego kodu. Zamiast tego spróbuj użyć polecenia Artisan `model:show`, które zapewnia wygodny przegląd wszystkich atrybutów i relacji modelu:

```shell
php artisan model:show Flight
```

<a name="eloquent-model-conventions"></a>
## Konwencje Modelu Eloquent

Modele wygenerowane przez polecenie `make:model` będą umieszczone w katalogu `app/Models`. Przyjrzyjmy się podstawowej klasie modelu i omówmy niektóre kluczowe konwencje Eloquent:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    // ...
}
```

<a name="table-names"></a>
### Nazwy Tabel

Po spojrzeniu na powyższy przykład, mogłeś zauważyć, że nie powiedzieliśmy Eloquent, która tabela bazy danych odpowiada naszemu modelowi `Flight`. Według konwencji, nazwa klasy w formacie "snake case" w liczbie mnogiej będzie używana jako nazwa tabeli, chyba że zostanie wyraźnie określona inna nazwa. Tak więc, w tym przypadku, Eloquent przyjmie, że model `Flight` przechowuje rekordy w tabeli `flights`, podczas gdy model `AirTrafficController` przechowywałby rekordy w tabeli `air_traffic_controllers`.

Jeśli odpowiadająca modelowi tabela bazy danych nie pasuje do tej konwencji, możesz ręcznie określić nazwę tabeli modelu, definiując właściwość `table` w modelu:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Tabela powiązana z modelem.
     *
     * @var string
     */
    protected $table = 'my_flights';
}
```

<a name="primary-keys"></a>
### Klucze Główne

Eloquent zakłada również, że każda odpowiadająca modelowi tabela bazy danych ma kolumnę klucza głównego o nazwie `id`. W razie potrzeby możesz zdefiniować chronioną właściwość `$primaryKey` w swoim modelu, aby określić inną kolumnę, która służy jako klucz główny modelu:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Klucz główny powiązany z tabelą.
     *
     * @var string
     */
    protected $primaryKey = 'flight_id';
}
```

Ponadto, Eloquent zakłada, że klucz główny jest rosnącą wartością całkowitą, co oznacza, że Eloquent automatycznie rzutuje klucz główny na liczbę całkowitą. Jeśli chcesz użyć nierosnącego lub nienumerycznego klucza głównego, musisz zdefiniować publiczną właściwość `$incrementing` w swoim modelu, która jest ustawiona na `false`:

```php
<?php

class Flight extends Model
{
    /**
     * Wskazuje, czy ID modelu jest automatycznie inkrementowane.
     *
     * @var bool
     */
    public $incrementing = false;
}
```

Jeśli klucz główny twojego modelu nie jest liczbą całkowitą, powinieneś zdefiniować chronioną właściwość `$keyType` w swoim modelu. Ta właściwość powinna mieć wartość `string`:

```php
<?php

class Flight extends Model
{
    /**
     * Typ danych ID klucza głównego.
     *
     * @var string
     */
    protected $keyType = 'string';
}
```

<a name="composite-primary-keys"></a>
#### "Złożone" Klucze Główne

Eloquent wymaga, aby każdy model miał przynajmniej jedno jednoznacznie identyfikujące "ID", które może służyć jako jego klucz główny. "Złożone" klucze główne nie są obsługiwane przez modele Eloquent. Możesz jednak swobodnie dodawać dodatkowe, wielokolumnowe, unikalne indeksy do swoich tabel bazy danych, oprócz jednoznacznie identyfikującego klucza głównego tabeli.

<a name="uuid-i-ulid-keys"></a>
### Klucze UUID i ULID

Zamiast używać automatycznie rosnących liczb całkowitych jako kluczy głównych twojego modelu Eloquent, możesz wybrać zamiast tego UUID. UUID to uniwersalnie unikalne identyfikatory alfanumeryczne o długości 36 znaków.

Jeśli chcesz, aby model używał klucza UUID zamiast automatycznie rosnącego klucza całkowitego, możesz użyć cechy `Illuminate\Database\Eloquent\Concerns\HasUuids` w modelu. Oczywiście powinieneś upewnić się, że model ma [kolumnę klucza głównego równoważną UUID](/docs/{{version}}/migrations#column-method-uuid):

```php
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    use HasUuids;

    // ...
}

$article = Article::create(['title' => 'Traveling to Europe']);

$article->id; // "8f8e8478-9035-4d23-b9a7-62f4d2612ce5"
```

Domyślnie, cecha `HasUuids` wygeneruje ["uporządkowane" UUID](/docs/{{version}}/strings#method-str-ordered-uuid) dla twoich modeli. Te UUID są bardziej wydajne dla indeksowanego przechowywania w bazie danych, ponieważ mogą być sortowane leksykograficznie.

Możesz nadpisać proces generowania UUID dla danego modelu, definiując metodę `newUniqueId` w modelu. Ponadto możesz określić, które kolumny powinny otrzymać UUID, definiując metodę `uniqueIds` w modelu:

```php
use Ramsey\Uuid\Uuid;

/**
 * Wygeneruj nowy UUID dla modelu.
 */
public function newUniqueId(): string
{
    return (string) Uuid::uuid4();
}

/**
 * Pobierz kolumny, które powinny otrzymać unikalny identyfikator.
 *
 * @return array<int, string>
 */
public function uniqueIds(): array
{
    return ['id', 'discount_code'];
}
```

Jeśli chcesz, możesz zdecydować się na wykorzystanie "ULID" zamiast UUID. ULID są podobne do UUID; jednak mają tylko 26 znaków długości. Podobnie jak uporządkowane UUID, ULID są leksykograficznie sortowaln i wydajne dla indeksowania bazy danych. Aby wykorzystać ULID, powinieneś użyć cechy `Illuminate\Database\Eloquent\Concerns\HasUlids` w swoim modelu. Powinieneś również upewnić się, że model ma [kolumnę klucza głównego równoważną ULID](/docs/{{version}}/migrations#column-method-ulid):

```php
use Illuminate\Database\Eloquent\Concerns\HasUlids;
use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    use HasUlids;

    // ...
}

$article = Article::create(['title' => 'Traveling to Asia']);

$article->id; // "01gd4d3tgrrfqeda94gdbtdk5c"
```

<a name="timestamps"></a>
### Znaczniki Czasu

Domyślnie, Eloquent oczekuje kolumn `created_at` i `updated_at` w odpowiadającej modelowi tabeli bazy danych. Eloquent automatycznie ustawi wartości tych kolumn, gdy modele są tworzone lub aktualizowane. Jeśli nie chcesz, aby te kolumny były automatycznie zarządzane przez Eloquent, powinieneś zdefiniować właściwość `$timestamps` w swoim modelu z wartością `false`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Wskazuje, czy model powinien mieć znaczniki czasu.
     *
     * @var bool
     */
    public $timestamps = false;
}
```

Jeśli musisz dostosować format znaczników czasu twojego modelu, ustaw właściwość `$dateFormat` w swoim modelu. Ta właściwość określa, jak atrybuty daty są przechowywane w bazie danych, a także ich format, gdy model jest serializowany do tablicy lub JSON:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Format przechowywania kolumn dat modelu.
     *
     * @var string
     */
    protected $dateFormat = 'U';
}
```

Jeśli musisz dostosować nazwy kolumn używanych do przechowywania znaczników czasu, możesz zdefiniować stałe `CREATED_AT` i `UPDATED_AT` w swoim modelu:

```php
<?php

class Flight extends Model
{
    /**
     * Nazwa kolumny "created at".
     *
     * @var string|null
     */
    public const CREATED_AT = 'creation_date';

    /**
     * Nazwa kolumny "updated at".
     *
     * @var string|null
     */
    public const UPDATED_AT = 'updated_date';
}
```

Jeśli chcesz wykonać operacje na modelu bez modyfikowania jego kolumny `updated_at`, możesz operować na modelu w zamknięciu przekazanym do metody `withoutTimestamps`:

```php
Model::withoutTimestamps(fn () => $post->increment('reads'));
```

<a name="database-connections"></a>
### Połączenia z Bazą Danych

Domyślnie wszystkie modele Eloquent będą używać domyślnego połączenia z bazą danych skonfigurowanego dla twojej aplikacji. Jeśli chcesz określić inne połączenie, które powinno być używane podczas interakcji z konkretnym modelem, powinieneś zdefiniować właściwość `$connection` w modelu:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Połączenie z bazą danych, które powinno być używane przez model.
     *
     * @var string
     */
    protected $connection = 'mysql';
}
```

<a name="default-attribute-values"></a>
### Domyślne Wartości Atrybutów

Domyślnie nowo utworzona instancja modelu nie będzie zawierać żadnych wartości atrybutów. Jeśli chcesz zdefiniować domyślne wartości dla niektórych atrybutów twojego modelu, możesz zdefiniować właściwość `$attributes` w swoim modelu. Wartości atrybutów umieszczone we właściwości `$attributes` powinny być w swoim surowym, "przechowywalnym" formacie, tak jakby zostały właśnie odczytane z bazy danych:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Domyślne wartości atrybutów modelu.
     *
     * @var array
     */
    protected $attributes = [
        'options' => '[]',
        'delayed' => false,
    ];
}
```

<a name="configuring-eloquent-strictness"></a>
### Konfigurowanie Rygorystyczności Eloquent

Laravel oferuje kilka metod, które pozwalają skonfigurować zachowanie Eloquent i "rygorystyczność" w różnych sytuacjach.

Po pierwsze, metoda `preventLazyLoading` przyjmuje opcjonalny argument logiczny wskazujący, czy leniwe ładowanie powinno być zapobiegane. Na przykład, możesz chcieć wyłączyć leniwe ładowanie tylko w środowiskach nieprodukcyjnych, aby Twoje środowisko produkcyjne mogło nadal funkcjonować normalnie, nawet jeśli leniwie ładowana relacja przypadkowo pojawi się w kodzie produkcyjnym. Zazwyczaj ta metoda powinna być wywołana w metodzie `boot` klasy `AppServiceProvider`:

```php
use Illuminate\Database\Eloquent\Model;

/**
 * Załaduj dowolne serwisy aplikacji.
 */
public function boot(): void
{
    Model::preventLazyLoading(! $this->app->isProduction());
}
```

Możesz również poinstruować Laravel, aby rzucał wyjątek podczas próby wypełnienia niewypełnialnego atrybutu, wywołując metodę `preventSilentlyDiscardingAttributes`. Może to pomóc zapobiec nieoczekiwanym błędom podczas lokalnego rozwoju podczas próby ustawienia atrybutu, który nie został dodany do tablicy `fillable` modelu:

```php
Model::preventSilentlyDiscardingAttributes(! $this->app->isProduction());
```

<a name="retrieving-models"></a>
## Pobieranie Modeli

Po utworzeniu modelu i [powiązanej z nim tabeli bazy danych](/docs/{{version}}/migrations#generating-migrations), jesteś gotowy, aby rozpocząć pobieranie danych z bazy danych. Możesz myśleć o każdym modelu Eloquent jako o potężnym [kreatorze zapytań](/docs/{{version}}/queries) umożliwiającym płynne odpytywanie tabeli bazy danych powiązanej z modelem. Metoda `all` pobierze wszystkie rekordy z powiązanej z modelem tabeli bazy danych:

```php
use App\Models\Flight;

foreach (Flight::all() as $flight) {
    echo $flight->name;
}
```

<a name="building-queries"></a>
#### Budowanie Zapytań

Metoda Eloquent `all` zwróci wszystkie wyniki w modelu. Jednak ponieważ każdy model Eloquent służy jako [kreator zapytań](/docs/{{version}}/queries), możesz dodać dodatkowe ograniczenia do zapytań, a następnie wywołać metodę `get`, aby pobrać wyniki:

```php
$flights = Flight::where('active', 1)
    ->orderBy('name')
    ->limit(10)
    ->get();
```

> [!NOTE]
> Ponieważ modele Eloquent są kreatorami zapytań, powinieneś przejrzeć wszystkie metody dostarczane przez [kreatora zapytań](/docs/{{version}}/queries). Możesz użyć dowolnej z tych metod podczas pisania zapytań Eloquent.

<a name="refreshing-models"></a>
#### Odświeżanie Modeli

Jeśli masz już instancję modelu Eloquent, która została pobrana z bazy danych, możesz "odświeżyć" model używając metod `fresh` i `refresh`. Metoda `fresh` ponownie pobierze model z bazy danych. Istniejąca instancja modelu nie zostanie naruszona:

```php
$flight = Flight::where('number', 'FR 900')->first();

$freshFlight = $flight->fresh();
```

Metoda `refresh` ponownie uwodni istniejący model używając świeżych danych z bazy danych. Ponadto wszystkie jego załadowane relacje również zostaną odświeżone:

```php
$flight = Flight::where('number', 'FR 900')->first();

$flight->number = 'FR 456';

$flight->refresh();

$flight->number; // "FR 900"
```

<a name="collections"></a>
### Kolekcje

Jak widzieliśmy, metody Eloquent takie jak `all` i `get` pobierają wiele rekordów z bazy danych. Jednak te metody nie zwracają zwykłej tablicy PHP. Zamiast tego zwracana jest instancja `Illuminate\Database\Eloquent\Collection`.

Klasa Eloquent `Collection` rozszerza bazową klasę Laravel `Illuminate\Support\Collection`, która dostarcza [różnorodne przydatne metody](/docs/{{version}}/collections#available-methods) do interakcji z kolekcjami danych. Na przykład, metoda `reject` może być użyta do usunięcia modeli z kolekcji na podstawie wyników wywołanego zamknięcia:

```php
$flights = Flight::where('destination', 'Paris')->get();

$flights = $flights->reject(function (Flight $flight) {
    return $flight->cancelled;
});
```

Oprócz metod dostarczanych przez bazową klasę kolekcji Laravel, klasa kolekcji Eloquent dostarcza [kilka dodatkowych metod](/docs/{{version}}/eloquent-collections#available-methods), które są specjalnie przeznaczone do interakcji z kolekcjami modeli Eloquent.

Ponieważ wszystkie kolekcje Laravel implementują interfejsy iterowalne PHP, możesz iterować po kolekcjach tak, jakby były tablicą:

```php
foreach ($flights as $flight) {
    echo $flight->name;
}
```

<a name="chunking-results"></a>
### Dzielenie Wyników

Twoja aplikacja może skończyć pamięć, jeśli spróbujesz załadować dziesiątki tysięcy rekordów Eloquent za pomocą metod `all` lub `get`. Zamiast używać tych metod, metoda `chunk` może być użyta do przetwarzania dużej liczby modeli bardziej wydajnie.

Metoda `chunk` pobierze podzbiór modeli Eloquent, przekazując je do zamknięcia do przetworzenia. Ponieważ tylko bieżący fragment modeli Eloquent jest pobierany na raz, metoda `chunk` zapewni znacznie zmniejszone zużycie pamięci podczas pracy z dużą liczbą modeli:

```php
use App\Models\Flight;
use Illuminate\Database\Eloquent\Collection;

Flight::chunk(200, function (Collection $flights) {
    foreach ($flights as $flight) {
        // ...
    }
});
```

Pierwszy argument przekazany do metody `chunk` to liczba rekordów, które chcesz otrzymać na "fragment". Zamknięcie przekazane jako drugi argument będzie wywołane dla każdego fragmentu pobranego z bazy danych. Zapytanie do bazy danych zostanie wykonane w celu pobrania każdego fragmentu rekordów przekazanych do zamknięcia.

Jeśli filtrujesz wyniki metody `chunk` na podstawie kolumny, którą również będziesz aktualizować podczas iteracji po wynikach, powinieneś użyć metody `chunkById`. Używanie metody `chunk` w tych scenariuszach mogłoby prowadzić do nieoczekiwanych i niespójnych wyników. Wewnętrznie, metoda `chunkById` zawsze pobierze modele z kolumną `id` większą niż ostatni model w poprzednim fragmencie:

```php
Flight::where('departed', true)
    ->chunkById(200, function (Collection $flights) {
        $flights->each->update(['departed' => false]);
    }, column: 'id');
```

Ponieważ metody `chunkById` i `lazyById` dodają własne "where" do wykonywanego zapytania, powinieneś zazwyczaj [logicznie zgrupować](/docs/{{version}}/queries#logical-grouping) swoje własne warunki w zamknięciu:

```php
Flight::where(function ($query) {
    $query->where('delayed', true)->orWhere('cancelled', true);
})->chunkById(200, function (Collection $flights) {
    $flights->each->update([
        'departed' => false,
        'cancelled' => true
    ]);
}, column: 'id');
```

<a name="chunking-using-lazy-collections"></a>
### Dzielenie przy Użyciu Leniwych Kolekcji

Metoda `lazy` działa podobnie do [metody `chunk`](#chunking-results) w tym sensie, że za kulisami wykonuje zapytanie w fragmentach. Jednak zamiast przekazywać każdy fragment bezpośrednio do callbacku, metoda `lazy` zwraca spłaszczoną [LazyCollection](/docs/{{version}}/collections#lazy-collections) modeli Eloquent, która pozwala interaktować z wynikami jako pojedynczym strumieniem:

```php
use App\Models\Flight;

foreach (Flight::lazy() as $flight) {
    // ...
}
```

Jeśli filtrujesz wyniki metody `lazy` na podstawie kolumny, którą również będziesz aktualizować podczas iteracji po wynikach, powinieneś użyć metody `lazyById`. Wewnętrznie, metoda `lazyById` zawsze pobierze modele z kolumną `id` większą niż ostatni model w poprzednim fragmencie:

```php
Flight::where('departed', true)
    ->lazyById(200, column: 'id')
    ->each->update(['departed' => false]);
```

Możesz filtrować wyniki na podstawie malejącego porządku `id` używając metody `lazyByIdDesc`.

<a name="cursors"></a>
### Kursory

Podobnie do metody `lazy`, metoda `cursor` może być użyta do znacznego zmniejszenia zużycia pamięci twojej aplikacji podczas iteracji przez dziesiątki tysięcy rekordów modeli Eloquent.

Metoda `cursor` wykona tylko jedno zapytanie do bazy danych; jednak poszczególne modele Eloquent nie będą hydratowane, dopóki nie będą faktycznie iterowane. W związku z tym, tylko jeden model Eloquent jest przechowywany w pamięci w dowolnym momencie podczas iteracji po kursorze.

> [!WARNING]
> Ponieważ metoda `cursor` przechowuje tylko jeden model Eloquent w pamięci na raz, nie może ładować chętnie relacji. Jeśli musisz ładować chętnie relacje, rozważ użycie [metody `lazy`](#chunking-using-lazy-collections) zamiast tego.

Wewnętrznie, metoda `cursor` używa [generatorów PHP](https://www.php.net/manual/en/language.generators.overview.php) do zaimplementowania tej funkcjonalności:

```php
use App\Models\Flight;

foreach (Flight::where('destination', 'Zurich')->cursor() as $flight) {
    // ...
}
```

Metoda `cursor` zwraca instancję `Illuminate\Support\LazyCollection`. [Leniwe kolekcje](/docs/{{version}}/collections#lazy-collections) pozwalają używać wielu metod kolekcji dostępnych w typowych kolekcjach Laravel, przy jednoczesnym ładowaniu tylko jednego modelu do pamięci na raz:

```php
use App\Models\User;

$users = User::cursor()->filter(function (User $user) {
    return $user->id > 500;
});

foreach ($users as $user) {
    echo $user->id;
}
```

Choć metoda `cursor` używa znacznie mniej pamięci niż zwykłe zapytanie (przechowując tylko jeden model Eloquent w pamięci na raz), nadal w końcu skończy się pamięć. Jest to spowodowane [wewnętrznym buforowaniem wszystkich surowych wyników zapytań przez sterownik PDO PHP](https://www.php.net/manual/en/mysqlinfo.concepts.buffering.php). Jeśli masz do czynienia z bardzo dużą liczbą rekordów Eloquent, rozważ użycie [metody `lazy`](#chunking-using-lazy-collections) zamiast tego.

<a name="advanced-subqueries"></a>
### Zaawansowane Podzapytania

<a name="subquery-selects"></a>
#### Podzapytania Select

Eloquent oferuje również zaawansowane wsparcie dla podzapytań, które pozwala wyciągnąć informacje z powiązanych tabel w jednym zapytaniu. Na przykład, załóżmy, że mamy tabelę `destinations` lotów i tabelę `flights` do miejsc docelowych. Tabela `flights` zawiera kolumnę `arrived_at`, która wskazuje, kiedy lot przyleciał do miejsca docelowego.

Używając funkcjonalności podzapytań dostępnej w metodach `select` i `addSelect` kreatora zapytań, możemy wybrać wszystkie `destinations` i nazwę lotu, który ostatnio przyleciał do tego miejsca docelowego, używając jednego zapytania:

```php
use App\Models\Destination;
use App\Models\Flight;

return Destination::addSelect(['last_flight' => Flight::select('name')
    ->whereColumn('destination_id', 'destinations.id')
    ->orderByDesc('arrived_at')
    ->limit(1)
])->get();
```

<a name="subquery-ordering"></a>
#### Sortowanie Podzapytań

Ponadto, funkcja `orderBy` kreatora zapytań obsługuje podzapytania. Kontynuując korzystanie z naszego przykładu lotu, możemy użyć tej funkcjonalności do sortowania wszystkich miejsc docelowych na podstawie tego, kiedy ostatni lot przyleciał do tego miejsca docelowego. Ponownie, można to zrobić wykonując pojedyncze zapytanie do bazy danych:

```php
return Destination::orderByDesc(
    Flight::select('arrived_at')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderByDesc('arrived_at')
        ->limit(1)
)->get();
```

<a name="retrieving-single-models"></a>
## Pobieranie Pojedynczych Modeli / Agregatów

Oprócz pobierania wszystkich rekordów pasujących do danego zapytania, możesz również pobierać pojedyncze rekordy używając metod `find`, `first` lub `firstWhere`. Zamiast zwracać kolekcję modeli, te metody zwracają pojedynczą instancję modelu:

```php
use App\Models\Flight;

// Pobierz model po jego kluczu głównym...
$flight = Flight::find(1);

// Pobierz pierwszy model pasujący do warunków zapytania...
$flight = Flight::where('active', 1)->first();

// Alternatywa dla pobierania pierwszego modelu pasującego do warunków zapytania...
$flight = Flight::firstWhere('active', 1);
```

Czasami możesz chcieć wykonać jakąś inną akcję, jeśli nie zostaną znalezione żadne wyniki. Metody `findOr` i `firstOr` zwrócą pojedynczą instancję modelu lub, jeśli nie zostaną znalezione żadne wyniki, wykonają podane zamknięcie. Wartość zwracana przez zamknięcie będzie uważana za wynik metody:

```php
$flight = Flight::findOr(1, function () {
    // ...
});

$flight = Flight::where('legs', '>', 3)->firstOr(function () {
    // ...
});
```

<a name="not-found-exceptions"></a>
#### Wyjątki Nie Znaleziono

Czasami możesz chcieć rzucić wyjątek, jeśli model nie zostanie znaleziony. Jest to szczególnie przydatne w trasach lub kontrolerach. Metody `findOrFail` i `firstOrFail` pobiorą pierwszy wynik zapytania; jednak, jeśli nie zostanie znaleziony żaden wynik, zostanie rzucony wyjątek `Illuminate\Database\Eloquent\ModelNotFoundException`:

```php
$flight = Flight::findOrFail(1);

$flight = Flight::where('legs', '>', 3)->firstOrFail();
```

Jeśli `ModelNotFoundException` nie zostanie złapany, odpowiedź HTTP 404 zostanie automatycznie wysłana z powrotem do klienta:

```php
use App\Models\Flight;

Route::get('/api/flights/{id}', function (string $id) {
    return Flight::findOrFail($id);
});
```

<a name="retrieving-or-creating-models"></a>
### Pobieranie lub Tworzenie Modeli

Metoda `firstOrCreate` spróbuje zlokalizować rekord bazy danych używając podanych par kolumna / wartość. Jeśli model nie może zostać znaleziony w bazie danych, zostanie wstawiony rekord z atrybutami wynikającymi z połączenia pierwszego argumentu tablicowego z opcjonalnym drugim argumentem tablicowym.

Metoda `firstOrNew`, podobnie jak `firstOrCreate`, spróbuje zlokalizować rekord w bazie danych pasujący do podanych atrybutów. Jednak, jeśli model nie zostanie znaleziony, zostanie zwrócona nowa instancja modelu. Zauważ, że model zwrócony przez `firstOrNew` nie został jeszcze utrwalony w bazie danych. Będziesz musiał ręcznie wywołać metodę `save`, aby go utrwalić:

```php
use App\Models\Flight;

// Pobierz lot po nazwie lub utwórz go, jeśli nie istnieje...
$flight = Flight::firstOrCreate([
    'name' => 'London to Paris'
]);

// Pobierz lot po nazwie lub utwórz go z atrybutami name, delayed i arrival_time...
$flight = Flight::firstOrCreate(
    ['name' => 'London to Paris'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);

// Pobierz lot po nazwie lub utwórz nową instancję Flight...
$flight = Flight::firstOrNew([
    'name' => 'London to Paris'
]);

// Pobierz lot po nazwie lub utwórz instancję z atrybutami name, delayed i arrival_time...
$flight = Flight::firstOrNew(
    ['name' => 'Tokyo to Sydney'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);
```

<a name="retrieving-aggregates"></a>
### Pobieranie Agregatów

Podczas interakcji z modelami Eloquent, możesz również używać metod `count`, `sum`, `max` i innych [metod agregujących](/docs/{{version}}/queries#aggregates) dostarczanych przez Laravel [kreator zapytań](/docs/{{version}}/queries). Jak możesz się spodziewać, te metody zwracają wartość skalarną zamiast instancji modelu Eloquent:

```php
$count = Flight::where('active', 1)->count();

$max = Flight::where('active', 1)->max('price');
```

<a name="inserting-and-updating-models"></a>
## Wstawianie i Aktualizowanie Modeli

<a name="inserts"></a>
### Wstawianie

Oczywiście, podczas używania Eloquent, nie tylko musimy pobierać modele z bazy danych. Musimy również wstawiać nowe rekordy. Na szczęście, Eloquent to ułatwia. Aby wstawić nowy rekord do bazy danych, powinieneś utworzyć instancję nowego modelu i ustawić atrybuty na modelu. Następnie wywołaj metodę `save` na instancji modelu:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Flight;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class FlightController extends Controller
{
    /**
     * Zapisz nowy lot w bazie danych.
     */
    public function store(Request $request): RedirectResponse
    {
        // Zwaliduj żądanie...

        $flight = new Flight;

        $flight->name = $request->name;

        $flight->save();

        return redirect('/flights');
    }
}
```

W tym przykładzie, przypisujemy pole `name` z przychodzącego żądania HTTP do atrybutu `name` modelu `App\Models\Flight`. Gdy wywołujemy metodę `save`, rekord zostanie wstawiony do bazy danych. Znaczniki czasu `created_at` i `updated_at` zostaną automatycznie ustawione, gdy metoda `save` zostanie wywołana, więc nie ma potrzeby ustawiać ich ręcznie.

Alternatywnie, możesz użyć metody `create` do "zapisania" nowego modelu używając pojedynczej instrukcji PHP. Wstawiona instancja modelu zostanie zwrócona przez metodę `create`:

```php
use App\Models\Flight;

$flight = Flight::create([
    'name' => 'London to Paris',
]);
```

Jednak przed użyciem metody `create`, będziesz musiał określić właściwość `fillable` lub `guarded` w swoim modelu. Te właściwości są wymagane, ponieważ wszystkie modele Eloquent są domyślnie chronione przed podatnościami na przypisanie masowe. Aby dowiedzieć się więcej o przypisaniu masowym, zapoznaj się z [dokumentacją przypisania masowego](#mass-assignment).

<a name="updates"></a>
### Aktualizowanie

Metoda `save` może również być używana do aktualizowania modeli, które już istnieją w bazie danych. Aby zaktualizować model, powinieneś go pobrać i ustawić wszelkie atrybuty, które chcesz zaktualizować. Następnie powinieneś wywołać metodę `save` modelu. Ponownie, znacznik czasu `updated_at` zostanie automatycznie zaktualizowany, więc nie ma potrzeby ręcznie ustawiać jego wartości:

```php
use App\Models\Flight;

$flight = Flight::find(1);

$flight->name = 'Paris to London';

$flight->save();
```

Czasami możesz potrzebować zaktualizować istniejący model lub utworzyć nowy model, jeśli nie istnieje pasujący model. Podobnie jak metoda `firstOrCreate`, metoda `updateOrCreate` utrzymuje model, więc nie ma potrzeby ręcznego wywoływania metody `save`.

W poniższym przykładzie, jeśli istnieje lot z lokalizacją `departure` `Oakland` i lokalizacją `destination` `San Diego`, jego kolumny `price` i `discounted` zostaną zaktualizowane. Jeśli taki lot nie istnieje, zostanie utworzony nowy lot, który ma atrybuty wynikające z połączenia pierwszej tablicy argumentów z drugą tablicą argumentów:

```php
$flight = Flight::updateOrCreate(
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    ['price' => 99, 'discounted' => 1]
);
```

Podczas używania metod takich jak `firstOrCreate` lub `updateOrCreate`, możesz nie wiedzieć, czy nowy model został utworzony, czy istniejący został zaktualizowany. Właściwość `wasRecentlyCreated` wskazuje, czy model został utworzony podczas jego bieżącego cyklu życia:

```php
$flight = Flight::updateOrCreate(
    // ...
);

if ($flight->wasRecentlyCreated) {
    // Nowy rekord lotu został wstawiony...
}
```

<a name="mass-updates"></a>
#### Masowe Aktualizowanie

Aktualizacje mogą również być wykonywane na modelach, które pasują do danego zapytania. W tym przykładzie wszystkie loty, które są `active` i mają `destination` `San Diego`, zostaną oznaczone jako opóźnione:

```php
Flight::where('active', 1)
    ->where('destination', 'San Diego')
    ->update(['delayed' => 1]);
```

Metoda `update` oczekuje tablicy par kolumna i wartość, reprezentujących kolumny, które powinny być zaktualizowane. Metoda `update` zwraca liczbę naruszonych wierszy.

> [!WARNING]
> Podczas wydawania masowej aktualizacji za pomocą Eloquent, zdarzenia modelu `saving`, `saved`, `updating` i `updated` nie będą wywoływane dla zaktualizowanych modeli. Jest to spowodowane tym, że modele nigdy nie są faktycznie pobierane podczas wydawania masowej aktualizacji.

<a name="examining-attribute-changes"></a>
#### Badanie Zmian Atrybutów

Eloquent udostępnia metody `isDirty`, `isClean` i `wasChanged` do badania wewnętrznego stanu twojego modelu i określania, jak jego atrybuty zmieniły się od momentu, gdy model został pierwotnie pobrany.

Metoda `isDirty` określa, czy którykolwiek z atrybutów modelu został zmieniony od momentu pobrania modelu. Możesz przekazać konkretną nazwę atrybutu lub tablicę atrybutów do metody `isDirty`, aby określić, czy którykolwiek z atrybutów jest "brudny". Metoda `isClean` określi, czy atrybut pozostał niezmieniony od momentu pobrania modelu. Ta metoda również akceptuje opcjonalny argument atrybutu:

```php
use App\Models\User;

$user = User::create([
    'first_name' => 'Taylor',
    'last_name' => 'Otwell',
    'title' => 'Developer',
]);

$user->title = 'Painter';

$user->isDirty(); // true
$user->isDirty('title'); // true
$user->isDirty('first_name'); // false
$user->isDirty(['first_name', 'title']); // true

$user->isClean(); // false
$user->isClean('title'); // false
$user->isClean('first_name'); // true
$user->isClean(['first_name', 'title']); // false

$user->save();

$user->isDirty(); // false
$user->isClean(); // true
```

Metoda `wasChanged` określa, czy którekolwiek atrybuty zostały zmienione, gdy model został ostatnio zapisany w bieżącym cyklu żądania. Jeśli to konieczne, możesz przekazać nazwę atrybutu, aby zobaczyć, czy dany atrybut został zmieniony:

```php
$user = User::create([
    'first_name' => 'Taylor',
    'last_name' => 'Otwell',
    'title' => 'Developer',
]);

$user->title = 'Painter';

$user->save();

$user->wasChanged(); // true
$user->wasChanged('title'); // true
$user->wasChanged(['title', 'slug']); // true
$user->wasChanged('first_name'); // false
$user->wasChanged(['first_name', 'title']); // true
```

Metoda `getOriginal` zwraca tablicę zawierającą oryginalne atrybuty modelu, niezależnie od jakichkolwiek zmian w modelu od momentu jego pobrania. Jeśli to konieczne, możesz przekazać konkretną nazwę atrybutu, aby uzyskać oryginalną wartość danego atrybutu:

```php
$user = User::find(1);

$user->name; // John
$user->email; // john@example.com

$user->name = 'Jack';
$user->name; // Jack

$user->getOriginal('name'); // John
$user->getOriginal(); // Array of original attributes...
```

Metoda `getChanges` zwraca tablicę zawierającą atrybuty, które zmieniły się, gdy model został ostatnio zapisany, podczas gdy metoda `getPrevious` zwraca tablicę zawierającą oryginalne wartości atrybutów przed ostatnim zapisaniem modelu:

```php
$user = User::find(1);

$user->name; // John
$user->email; // john@example.com

$user->update([
    'name' => 'Jack',
    'email' => 'jack@example.com',
]);

$user->getChanges();

/*
    [
        'name' => 'Jack',
        'email' => 'jack@example.com',
    ]
*/

$user->getPrevious();

/*
    [
        'name' => 'John',
        'email' => 'john@example.com',
    ]
*/
```

<a name="mass-assignment"></a>
### Przypisanie Masowe

Możesz użyć metody `create`, aby "zapisać" nowy model za pomocą pojedynczej instrukcji PHP. Wstawiona instancja modelu zostanie zwrócona przez tę metodę:

```php
use App\Models\Flight;

$flight = Flight::create([
    'name' => 'London to Paris',
]);
```

Jednak przed użyciem metody `create` będziesz musiał określić właściwość `fillable` lub `guarded` w swoim modelu. Te właściwości są wymagane, ponieważ wszystkie modele Eloquent są domyślnie chronione przed podatnościami na przypisanie masowe.

Podatność na przypisanie masowe występuje, gdy użytkownik przekazuje nieoczekiwane pole żądania HTTP i to pole zmienia kolumnę w bazie danych, której nie oczekiwałeś. Na przykład, złośliwy użytkownik mógłby wysłać parametr `is_admin` za pośrednictwem żądania HTTP, który jest następnie przekazywany do metody `create` twojego modelu, pozwalając użytkownikowi na eskalację swoich uprawnień do administratora.

Więc, aby zacząć, powinieneś zdefiniować, które atrybuty modelu chcesz uczynić przypisywalnymi masowo. Możesz to zrobić, używając właściwości `$fillable` w modelu. Na przykład, uczyńmy atrybut `name` naszego modelu `Flight` przypisywalnym masowo:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Atrybuty, które są przypisywalne masowo.
     *
     * @var array<int, string>
     */
    protected $fillable = ['name'];
}
```

Po określeniu, które atrybuty są przypisywalne masowo, możesz użyć metody `create`, aby wstawić nowy rekord do bazy danych. Metoda `create` zwraca nowo utworzony model:

```php
$flight = Flight::create(['name' => 'London to Paris']);
```

Jeśli masz już instancję modelu, możesz użyć metody `fill`, aby wypełnić ją tablicą atrybutów:

```php
$flight->fill(['name' => 'Amsterdam to Frankfurt']);
```

<a name="mass-assignment-json-columns"></a>
#### Przypisanie Masowe i Kolumny JSON

Podczas przypisywania kolumn JSON, klucz przypisywalny masowo każdej kolumny musi być określony w tablicy `$fillable` twojego modelu. Ze względów bezpieczeństwa Laravel nie obsługuje aktualizowania zagnieżdżonych atrybutów JSON podczas używania właściwości `guarded`:

```php
/**
 * Atrybuty, które są przypisywalne masowo.
 *
 * @var array<int, string>
 */
protected $fillable = [
    'options->enabled',
];
```

<a name="allowing-mass-assignment"></a>
#### Zezwolenie na Przypisanie Masowe

Jeśli chcesz uczynić wszystkie swoje atrybuty przypisywalnymi masowo, możesz zdefiniować właściwość `$guarded` swojego modelu jako pustą tablicę. Jeśli zdecydujesz się wyłączyć ochronę swojego modelu, powinieneś szczególnie zadbać o to, aby zawsze ręcznie tworzyć tablice przekazywane do metod `fill`, `create` i `update` Eloquent:

```php
/**
 * Atrybuty, które nie są przypisywalne masowo.
 *
 * @var array<string>|bool
 */
protected $guarded = [];
```

<a name="mass-assignment-exceptions"></a>
#### Wyjątki Przypisania Masowego

Domyślnie atrybuty, które nie są uwzględnione w tablicy `$fillable`, są po cichu odrzucane podczas wykonywania operacji przypisania masowego. W środowisku produkcyjnym jest to oczekiwane zachowanie; jednak podczas lokalnego rozwoju może to prowadzić do zamieszania, dlaczego zmiany modelu nie mają efektu.

Jeśli chcesz, możesz polecić Laravel, aby rzucał wyjątek podczas próby wypełnienia niewypełnialnego atrybutu, wywołując metodę `preventSilentlyDiscardingAttributes`. Zazwyczaj ta metoda powinna być wywołana w metodzie `boot` klasy `AppServiceProvider`:

```php
use Illuminate\Database\Eloquent\Model;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Model::preventSilentlyDiscardingAttributes($this->app->isLocal());
}
```

<a name="upserts"></a>
### Upserty

Metoda `upsert` Eloquent może być użyta do aktualizowania lub tworzenia rekordów w pojedynczej, atomowej operacji. Pierwszy argument metody składa się z wartości do wstawienia lub aktualizacji, podczas gdy drugi argument wymienia kolumny, które jednoznacznie identyfikują rekordy w powiązanej tabeli. Trzeci i ostatni argument metody to tablica kolumn, które powinny być zaktualizowane, jeśli pasujący rekord już istnieje w bazie danych. Metoda `upsert` automatycznie ustawi znaczniki czasu `created_at` i `updated_at`, jeśli znaczniki czasu są włączone w modelu:

```php
Flight::upsert([
    ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
    ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
], uniqueBy: ['departure', 'destination'], update: ['price']);
```

> [!WARNING]
> Wszystkie bazy danych z wyjątkiem SQL Server wymagają, aby kolumny w drugim argumencie metody `upsert` miały indeks "główny" lub "unikalny". Ponadto sterowniki baz danych MariaDB i MySQL ignorują drugi argument metody `upsert` i zawsze używają indeksów "głównych" i "unikalnych" tabeli do wykrywania istniejących rekordów.

<a name="deleting-models"></a>
## Usuwanie Modeli

Aby usunąć model, możesz wywołać metodę `delete` na instancji modelu:

```php
use App\Models\Flight;

$flight = Flight::find(1);

$flight->delete();
```

<a name="deleting-an-existing-model-by-its-primary-key"></a>
#### Usuwanie Istniejącego Modelu po Kluczu Głównym

W powyższym przykładzie pobieramy model z bazy danych przed wywołaniem metody `delete`. Jednak jeśli znasz klucz główny modelu, możesz usunąć model bez jawnego jego pobierania, wywołując metodę `destroy`. Oprócz akceptowania pojedynczego klucza głównego, metoda `destroy` akceptuje również wiele kluczy głównych, tablicę kluczy głównych lub [kolekcję](/docs/{{version}}/collections) kluczy głównych:

```php
Flight::destroy(1);

Flight::destroy(1, 2, 3);

Flight::destroy([1, 2, 3]);

Flight::destroy(collect([1, 2, 3]));
```

Jeśli korzystasz z [miękkiego usuwania modeli](#soft-deleting), możesz trwale usunąć modele za pomocą metody `forceDestroy`:

```php
Flight::forceDestroy(1);
```

> [!WARNING]
> Metoda `destroy` ładuje każdy model indywidualnie i wywołuje metodę `delete`, dzięki czemu zdarzenia `deleting` i `deleted` są prawidłowo wysyłane dla każdego modelu.

<a name="deleting-models-using-queries"></a>
#### Usuwanie Modeli za Pomocą Zapytań

Oczywiście możesz zbudować zapytanie Eloquent, aby usunąć wszystkie modele pasujące do kryteriów twojego zapytania. W tym przykładzie usuniemy wszystkie loty, które są oznaczone jako nieaktywne. Podobnie jak masowe aktualizacje, masowe usunięcia nie będą wysyłać zdarzeń modelu dla usuwanych modeli:

```php
$deleted = Flight::where('active', 0)->delete();
```

Aby usunąć wszystkie modele w tabeli, powinieneś wykonać zapytanie bez dodawania jakichkolwiek warunków:

```php
$deleted = Flight::query()->delete();
```

> [!WARNING]
> Podczas wykonywania instrukcji masowego usunięcia za pomocą Eloquent, zdarzenia modelu `deleting` i `deleted` nie będą wysyłane dla usuniętych modeli. Jest to spowodowane tym, że modele nigdy nie są faktycznie pobierane podczas wykonywania instrukcji usuwania.

<a name="soft-deleting"></a>
### Usuwanie Miękkie

Oprócz faktycznego usuwania rekordów z bazy danych, Eloquent może również "miękko usuwać" modele. Gdy modele są miękko usuwane, nie są one faktycznie usuwane z bazy danych. Zamiast tego atrybut `deleted_at` jest ustawiany na modelu, wskazując datę i czas, w którym model został "usunięty". Aby włączyć miękkie usuwanie dla modelu, dodaj cechę `Illuminate\Database\Eloquent\SoftDeletes` do modelu:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model
{
    use SoftDeletes;
}
```

> [!NOTE]
> Cecha `SoftDeletes` automatycznie rzutuje atrybut `deleted_at` na instancję `DateTime` / `Carbon`.

Powinieneś również dodać kolumnę `deleted_at` do swojej tabeli bazy danych. Laravel [kreator schematu](/docs/{{version}}/migrations) zawiera metodę pomocniczą do utworzenia tej kolumny:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('flights', function (Blueprint $table) {
    $table->softDeletes();
});

Schema::table('flights', function (Blueprint $table) {
    $table->dropSoftDeletes();
});
```

Teraz, gdy wywołasz metodę `delete` na instancji modelu, kolumna `deleted_at` zostanie ustawiona na bieżącą datę i czas. Jednak rekord bazy danych modelu zostanie pozostał w tabeli. Podczas odpytywania modelu, który używa miękkiego usuwania, miękko usunięte modele będą automatycznie wykluczone ze wszystkich wyników zapytania.

Aby określić, czy dana instancja modelu została miękko usunięta, możesz użyć metody `trashed`:

```php
if ($flight->trashed()) {
    // ...
}
```

<a name="restoring-soft-deleted-models"></a>
#### Przywracanie Mięko Usuniętych Modeli

Czasami możesz chcieć "cofnąć usunięcie" mięko usuniętego modelu. Aby przywrócić mięko usunięty model, możesz wywołać metodę `restore` na instancji modelu. Metoda `restore` ustawi kolumnę `deleted_at` modelu na `null`:

```php
$flight->restore();
```

Możesz również użyć metody `restore` w zapytaniu, aby przywrócić wiele modeli. Ponownie, podobnie jak inne operacje "masowe", nie wywoła to żadnych zdarzeń modelu dla przywracanych modeli:

```php
Flight::withTrashed()
    ->where('airline_id', 1)
    ->restore();
```

Metoda `restore` może być również użyta podczas budowania zapytań [relacji](/docs/{{version}}/eloquent-relationships):

```php
$flight->history()->restore();
```

<a name="permanently-deleting-models"></a>
#### Trwałe Usuwanie Modeli

Czasami może być konieczne prawdziwe usunięcie modelu z bazy danych. Możesz użyć metody `forceDelete`, aby trwale usunąć miękko usunięty model z tabeli bazy danych:

```php
$flight->forceDelete();
```

Możesz również użyć metody `forceDelete` podczas budowania zapytań relacji Eloquent:

```php
$flight->history()->forceDelete();
```

<a name="querying-soft-deleted-models"></a>
### Odpytywanie Mięko Usuniętych Modeli

<a name="including-soft-deleted-models"></a>
#### Uwzględnianie Mięko Usuniętych Modeli

Jak zaznaczono powyżej, mięko usunięte modele zostaną automatycznie wykluczone z wyników zapytania. Jednak możesz wymusić, aby mięko usunięte modele zostały uwzględnione w wynikach zapytania, wywołując metodę `withTrashed` na zapytaniu:

```php
use App\Models\Flight;

$flights = Flight::withTrashed()
    ->where('account_id', 1)
    ->get();
```

Metoda `withTrashed` może być również wywołana podczas budowania zapytania [relacji](/docs/{{version}}/eloquent-relationships):

```php
$flight->history()->withTrashed()->get();
```

<a name="retrieving-only-soft-deleted-models"></a>
#### Pobieranie Tylko Mięko Usuniętych Modeli

Metoda `onlyTrashed` pobierze **tylko** mięko usunięte modele:

```php
$flights = Flight::onlyTrashed()
    ->where('airline_id', 1)
    ->get();
```

<a name="pruning-models"></a>
## Przycinanie Modeli

Czasami możesz chcieć okresowo usuwać modele, które nie są już potrzebne. Aby to osiągnąć, możesz dodać cechę `Illuminate\Database\Eloquent\Prunable` lub `Illuminate\Database\Eloquent\MassPrunable` do modeli, które chcesz okresowo przycinać. Po dodaniu jednej z cech do modelu, zaimplementuj metodę `prunable`, która zwraca kreator zapytań Eloquent, który rozpoznaje modele, które nie są już potrzebne:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Prunable;

class Flight extends Model
{
    use Prunable;

    /**
     * Pobierz zapytanie modelu do przycinania.
     */
    public function prunable(): Builder
    {
        return static::where('created_at', '<=', now()->minus(months: 1));
    }
}
```

Podczas oznaczania modeli jako `Prunable`, możesz również zdefiniować metodę `pruning` w modelu. Ta metoda zostanie wywołana przed usunięciem modelu. Ta metoda może być przydatna do usuwania wszelkich dodatkowych zasobów powiązanych z modelem, takich jak przechowywane pliki, zanim model zostanie trwale usunięty z bazy danych:

```php
/**
 * Przygotuj model do przycinania.
 */
protected function pruning(): void
{
    // ...
}
```

Po skonfigurowaniu modelu do przycinania, powinieneś zaplanować polecenie Artisan `model:prune` w pliku `routes/console.php`. Możesz wybrać odpowiedni interwał, w którym to polecenie powinno być uruchamiane:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('model:prune')->daily();
```

W tle, polecenie `model:prune` automatycznie wykryje modele "Prunable" w katalogu `app/Models`. Jeśli twoje modele znajdują się w innej lokalizacji, możesz użyć opcji `--model`, aby określić nazwy klas modeli:

```php
Schedule::command('model:prune', [
    '--model' => [Address::class, Flight::class],
])->daily();
```

Jeśli chcesz wykluczyć pewne modele z przycinania podczas przycinania wszystkich wykrytych modeli, możesz użyć opcji `--except`:

```php
Schedule::command('model:prune', [
    '--except' => [Address::class, Flight::class],
])->daily();
```

Możesz przetestować swoje zapytanie `prunable`, wykonując polecenie `model:prune` z opcją `--pretend`. Podczas udawania, polecenie `model:prune` po prostu zgłosi, ile rekordów zostałoby przyciętych, gdyby polecenie zostało faktycznie uruchomione:

```shell
php artisan model:prune --pretend
```

> [!WARNING]
> Mięko usuwane modele zostaną trwale usunięte (`forceDelete`), jeśli będą pasować do zapytania przycinania.

<a name="mass-pruning"></a>
#### Przycinanie Masowe

Gdy modele są oznaczone cechą `Illuminate\Database\Eloquent\MassPrunable`, modele są usuwane z bazy danych za pomocą zapytań masowego usuwania. W związku z tym, metoda `pruning` nie zostanie wywołana, ani zdarzenia modelu `deleting` i `deleted` nie będą wysyłane. Jest to spowodowane tym, że modele nigdy nie są faktycznie pobierane przed usunięciem, co sprawia, że proces przycinania jest znacznie bardziej wydajny:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\MassPrunable;

class Flight extends Model
{
    use MassPrunable;

    /**
     * Pobierz zapytanie modelu do przycinania.
     */
    public function prunable(): Builder
    {
        return static::where('created_at', '<=', now()->minus(months: 1));
    }
}
```

<a name="replicating-models"></a>
## Replikowanie Modeli

Możesz utworzyć niezapisaną kopię istniejącej instancji modelu, używając metody `replicate`. Ta metoda jest szczególnie przydatna, gdy masz instancje modeli, które dzielą wiele tych samych atrybutów:

```php
use App\Models\Address;

$shipping = Address::create([
    'type' => 'shipping',
    'line_1' => '123 Example Street',
    'city' => 'Victorville',
    'state' => 'CA',
    'postcode' => '90001',
]);

$billing = $shipping->replicate()->fill([
    'type' => 'billing'
]);

$billing->save();
```

Aby wykluczyć jeden lub więcej atrybutów z replikacji do nowego modelu, możesz przekazać tablicę do metody `replicate`:

```php
$flight = Flight::create([
    'destination' => 'LAX',
    'origin' => 'LHR',
    'last_flown' => '2020-03-04 11:00:00',
    'last_pilot_id' => 747,
]);

$flight = $flight->replicate([
    'last_flown',
    'last_pilot_id'
]);
```

<a name="query-scopes"></a>
## Zakresy Zapytań

<a name="global-scopes"></a>
### Zakresy Globalne

Zakresy globalne pozwalają na dodanie ograniczeń do wszystkich zapytań dla danego modelu. Własna funkcjonalność Laravel [miękkiego usuwania](#soft-deleting) wykorzystuje zakresy globalne, aby pobierać tylko modele "nie-usunięte" z bazy danych. Pisanie własnych zakresów globalnych może zapewnić wygodny, łatwy sposób, aby upewnić się, że każde zapytanie dla danego modelu otrzymuje określone ograniczenia.

<a name="generating-scopes"></a>
#### Generowanie Zakresów

Aby wygenerować nowy zakres globalny, możesz wywołać polecenie Artisan `make:scope`, które umieści wygenerowany zakres w katalogu `app/Models/Scopes`:

```shell
php artisan make:scope AncientScope
```

<a name="writing-global-scopes"></a>
#### Pisanie Zakresów Globalnych

Pisanie zakresu globalnego jest proste. Najpierw użyj polecenia `make:scope`, aby wygenerować klasę, która implementuje interfejs `Illuminate\Database\Eloquent\Scope`. Interfejs `Scope` wymaga zaimplementowania jednej metody: `apply`. Metoda `apply` może dodawać ograniczenia `where` lub inne typy klauzul do zapytania według potrzeb:

```php
<?php

namespace App\Models\Scopes;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;

class AncientScope implements Scope
{
    /**
     * Zastosuj zakres do danego kreatora zapytań Eloquent.
     */
    public function apply(Builder $builder, Model $model): void
    {
        $builder->where('created_at', '<', now()->minus(years: 2000));
    }
}
```

> [!NOTE]
> Jeśli twój zakres globalny dodaje kolumny do klauzuli select zapytania, powinieneś użyć metody `addSelect` zamiast `select`. Zapobiegnie to niezamierzonemu zastąpieniu istniejącej klauzuli select zapytania.

<a name="applying-global-scopes"></a>
#### Stosowanie Zakresów Globalnych

Aby przypisać zakres globalny do modelu, możesz po prostu umieścić atrybut `ScopedBy` w modelu:

```php
<?php

namespace App\Models;

use App\Models\Scopes\AncientScope;
use Illuminate\Database\Eloquent\Attributes\ScopedBy;

#[ScopedBy([AncientScope::class])]
class User extends Model
{
    //
}
```

Lub możesz ręcznie zarejestrować zakres globalny, nadpisując metodę `booted` modelu i wywołując metodę `addGlobalScope` modelu. Metoda `addGlobalScope` przyjmuje instancję twojego zakresu jako jedyny argument:

```php
<?php

namespace App\Models;

use App\Models\Scopes\AncientScope;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Metoda "booted" modelu.
     */
    protected static function booted(): void
    {
        static::addGlobalScope(new AncientScope);
    }
}
```

Po dodaniu zakresu z przykładu powyżej do modelu `App\Models\User`, wywołanie metody `User::all()` wykona następujące zapytanie SQL:

```sql
select * from `users` where `created_at` < 0021-02-18 00:00:00
```

<a name="anonymous-global-scopes"></a>
#### Anonimowe Zakresy Globalne

Eloquent pozwala również na definiowanie zakresów globalnych za pomocą zamknięć, co jest szczególnie przydatne dla prostych zakresów, które nie wymagają oddzielnej klasy. Podczas definiowania zakresu globalnego za pomocą zamknięcia, powinieneś podać nazwę zakresu według własnego wyboru jako pierwszy argument metody `addGlobalScope`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Metoda "booted" modelu.
     */
    protected static function booted(): void
    {
        static::addGlobalScope('ancient', function (Builder $builder) {
            $builder->where('created_at', '<', now()->minus(years: 2000));
        });
    }
}
```

<a name="removing-global-scopes"></a>
#### Usuwanie Zakresów Globalnych

Jeśli chcesz usunąć zakres globalny dla danego zapytania, możesz użyć metody `withoutGlobalScope`. Metoda ta przyjmuje nazwę klasy zakresu globalnego jako jedyny argument:

```php
User::withoutGlobalScope(AncientScope::class)->get();
```

Lub, jeśli zdefiniowałeś zakres globalny za pomocą zamknięcia, powinieneś przekazać nazwę tekstową, którą przypisałeś do zakresu globalnego:

```php
User::withoutGlobalScope('ancient')->get();
```

Jeśli chcesz usunąć kilka lub nawet wszystkie zakresy globalne zapytania, możesz użyć metod `withoutGlobalScopes` i `withoutGlobalScopesExcept`:

```php
// Usuń wszystkie zakresy globalne...
User::withoutGlobalScopes()->get();

// Usuń niektóre zakresy globalne...
User::withoutGlobalScopes([
    FirstScope::class, SecondScope::class
])->get();

// Usuń wszystkie zakresy globalne z wyjątkiem podanych...
User::withoutGlobalScopesExcept([
    SecondScope::class,
])->get();
```

<a name="local-scopes"></a>
### Zakresy Lokalne

Zakresy lokalne pozwalają na definiowanie wspólnych zestawów ograniczeń zapytań, które możesz łatwo ponownie używać w całej aplikacji. Na przykład, możesz musić często pobierać wszystkich użytkowników, którzy są uważani za "popularnych". Aby zdefiniować zakres, dodaj atrybut `Scope` do metody Eloquent.

Zakresy powinny zawsze zwracać tego samego kreatora zapytań lub `void`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Scope;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Ogranicz zapytanie do popularnych użytkowników.
     */
    #[Scope]
    protected function popular(Builder $query): void
    {
        $query->where('votes', '>', 100);
    }

    /**
     * Ogranicz zapytanie do aktywnych użytkowników.
     */
    #[Scope]
    protected function active(Builder $query): void
    {
        $query->where('active', 1);
    }
}
```

<a name="utilizing-a-local-scope"></a>
#### Wykorzystywanie Zakresu Lokalnego

Po zdefiniowaniu zakresu możesz wywołać metody zakresu podczas odpytywania modelu. Możesz nawet łączyć wywołania różnych zakresów:

```php
use App\Models\User;

$users = User::popular()->active()->orderBy('created_at')->get();
```

Łączenie wielu zakresów modelu Eloquent za pomocą operatora zapytania `or` może wymagać użycia zamknięć, aby osiągnąć poprawne [grupowanie logiczne](/docs/{{version}}/queries#logical-grouping):

```php
$users = User::popular()->orWhere(function (Builder $query) {
    $query->active();
})->get();
```

Jednak ponieważ może to być uciążliwe, Laravel udostępnia metodę "wyższego rzędu" `orWhere`, która pozwala płynnie łączyć zakresy bez używania zamknięć:

```php
$users = User::popular()->orWhere->active()->get();
```

<a name="dynamic-scopes"></a>
#### Dynamiczne Zakresy

Czasami możesz chcieć zdefiniować zakres, który akceptuje parametry. Aby rozpocząć, po prostu dodaj dodatkowe parametry do sygnatury metody zakresu. Parametry zakresu powinny być zdefiniowane po parametrze `$query`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Scope;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Ogranicz zapytanie do użytkowników danego typu.
     */
    #[Scope]
    protected function ofType(Builder $query, string $type): void
    {
        $query->where('type', $type);
    }
}
```

Po dodaniu oczekiwanych argumentów do sygnatury metody zakresu, możesz przekazywać argumenty podczas wywoływania zakresu:

```php
$users = User::ofType('admin')->get();
```

<a name="pending-attributes"></a>
### Oczekujące Atrybuty

Jeśli chcesz użyć zakresów do tworzenia modeli, które mają te same atrybuty co te używane do ograniczenia zakresu, możesz użyć metody `withAttributes` podczas budowania zapytania zakresu:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Scope;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    /**
     * Ogranicz zapytanie tylko do wersji roboczych.
     */
    #[Scope]
    protected function draft(Builder $query): void
    {
        $query->withAttributes([
            'hidden' => true,
        ]);
    }
}
```

Metoda `withAttributes` doda warunki `where` do zapytania, używając podanych atrybutów, a także doda podane atrybuty do wszelkich modeli utworzonych za pomocą zakresu:

```php
$draft = Post::draft()->create(['title' => 'In Progress']);

$draft->hidden; // true
```

Aby poinstruować metodę `withAttributes`, aby nie dodawała warunków `where` do zapytania, możesz ustawić argument `asConditions` na `false`:

```php
$query->withAttributes([
    'hidden' => true,
], asConditions: false);
```

<a name="comparing-models"></a>
## Porównywanie Modeli

Czasami możesz musić określić, czy dwa modele są "takie same" czy nie. Metody `is` i `isNot` mogą być użyte do szybkiego zweryfikowania, czy dwa modele mają ten sam klucz główny, tabelę i połączenie z bazą danych, czy nie:

```php
if ($post->is($anotherPost)) {
    // ...
}

if ($post->isNot($anotherPost)) {
    // ...
}
```

Metody `is` i `isNot` są również dostępne podczas używania relacji `belongsTo`, `hasOne`, `morphTo` i `morphOne` [relationships](/docs/{{version}}/eloquent-relationships). Ta metoda jest szczególnie pomocna, gdy chcesz porównać powiązany model bez wysyłania zapytania w celu pobrania tego modelu:

```php
if ($post->author()->is($user)) {
    // ...
}
```

<a name="events"></a>
## Zdarzenia

> [!NOTE]
> Chcesz wysyłać zdarzenia Eloquent bezpośrednio do aplikacji po stronie klienta? Sprawdź [wysyłanie zdarzeń modelu](/docs/{{version}}/broadcasting#model-broadcasting) w Laravel.

Modele Eloquent wysyłają kilka zdarzeń, pozwalając na podłączenie się do następujących momentów w cyklu życia modelu: `retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `trashed`, `forceDeleting`, `forceDeleted`, `restoring`, `restored` i `replicating`.

Zdarzenie `retrieved` zostanie wysłane, gdy istniejący model zostanie pobrany z bazy danych. Gdy nowy model zostanie zapisany po raz pierwszy, zdarzenia `creating` i `created` zostaną wysłane. Zdarzenia `updating` / `updated` zostaną wysłane, gdy istniejący model zostanie zmodyfikowany i zostanie wywołana metoda `save`. Zdarzenia `saving` / `saved` zostaną wysłane, gdy model zostanie utworzony lub zaktualizowany - nawet jeśli atrybuty modelu nie zostały zmienione. Nazwy zdarzeń kończące się na `-ing` są wysyłane przed utrwaleniem jakichkolwiek zmian w modelu, podczas gdy zdarzenia kończące się na `-ed` są wysyłane po utrwaleniu zmian w modelu.

Aby zacząć nasłuchiwać zdarzeń modelu, zdefiniuj właściwość `$dispatchesEvents` w swoim modelu Eloquent. Ta właściwość mapuje różne punkty cyklu życia modelu Eloquent na twoje własne [klasy zdarzeń](/docs/{{version}}/events). Każda klasa zdarzenia modelu powinna oczekiwać otrzymania instancji dotyczącego modelu przez swój konstruktor:

```php
<?php

namespace App\Models;

use App\Events\UserDeleted;
use App\Events\UserSaved;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Mapa zdarzeń dla modelu.
     *
     * @var array<string, string>
     */
    protected $dispatchesEvents = [
        'saved' => UserSaved::class,
        'deleted' => UserDeleted::class,
    ];
}
```

Po zdefiniowaniu i zmapowaniu zdarzeń Eloquent, możesz używać [nasłuchiwaczy zdarzeń](/docs/{{version}}/events#defining-listeners) do obsługi zdarzeń.

> [!WARNING]
> Podczas wysyłania zapytania masowej aktualizacji lub usuwania za pośrednictwem Eloquent, zdarzenia modelu `saved`, `updated`, `deleting` i `deleted` nie będą wysyłane dla dotkniętych modeli. Jest to spowodowane tym, że modele nigdy nie są faktycznie pobierane podczas wykonywania masowych aktualizacji lub usunięć.

<a name="events-using-closures"></a>
### Używanie Zamknięć

Zamiast używać niestandardowych klas zdarzeń, możesz zarejestrować zamknięcia, które są wykonywane, gdy różne zdarzenia modelu są wysyłane. Zazwyczaj powinieneś rejestrować te zamknięcia w metodzie `booted` swojego modelu:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Metoda "booted" modelu.
     */
    protected static function booted(): void
    {
        static::created(function (User $user) {
            // ...
        });
    }
}
```

Jeśli potrzebujesz, możesz używać [kolejkowalnych anonimowych nasłuchiwaczy zdarzeń](/docs/{{version}}/events#queuable-anonymous-event-listeners) podczas rejestrowania zdarzeń modelu. To poinstruuje Laravel, aby wykonywał nasłuchiwacz zdarzeń modelu w tle, używając [kolejki](/docs/{{version}}/queues) aplikacji:

```php
use function Illuminate\Events\queueable;

static::created(queueable(function (User $user) {
    // ...
}));
```

<a name="observers"></a>
### Obserwatorzy

<a name="defining-observers"></a>
#### Definiowanie Obserwatorów

Jeśli nasłuchujesz wielu zdarzeń na danym modelu, możesz użyć obserwatorów, aby zgrupować wszystkich nasłuchiwaczy w pojedynczej klasie. Klasy obserwatorów mają nazwy metod, które odzwierciedlają zdarzenia Eloquent, których chcesz nasłuchiwać. Każda z tych metod otrzymuje dotknięty model jako jedyny argument. Polecenie Artisan `make:observer` to najłatwiejszy sposób na utworzenie nowej klasy obserwatora:

```shell
php artisan make:observer UserObserver --model=User
```

To polecenie umieści nowego obserwatora w katalogu `app/Observers`. Jeśli ten katalog nie istnieje, Artisan utworzy go dla ciebie. Twój świeży obserwator będzie wyglądał następująco:

```php
<?php

namespace App\Observers;

use App\Models\User;

class UserObserver
{
    /**
     * Obsługa zdarzenia User "created".
     */
    public function created(User $user): void
    {
        // ...
    }

    /**
     * Obsługa zdarzenia User "updated".
     */
    public function updated(User $user): void
    {
        // ...
    }

    /**
     * Obsługa zdarzenia User "deleted".
     */
    public function deleted(User $user): void
    {
        // ...
    }

    /**
     * Obsługa zdarzenia User "restored".
     */
    public function restored(User $user): void
    {
        // ...
    }

    /**
     * Obsługa zdarzenia User "forceDeleted".
     */
    public function forceDeleted(User $user): void
    {
        // ...
    }
}
```

Aby zarejestrować obserwatora, możesz umieścić atrybut `ObservedBy` w odpowiednim modelu:

```php
use App\Observers\UserObserver;
use Illuminate\Database\Eloquent\Attributes\ObservedBy;

#[ObservedBy([UserObserver::class])]
class User extends Authenticatable
{
    //
}
```

Lub możesz ręcznie zarejestrować obserwatora, wywołując metodę `observe` w modelu, który chcesz obserwować. Możesz zarejestrować obserwatorów w metodzie `boot` klasy `AppServiceProvider`:

```php
use App\Models\User;
use App\Observers\UserObserver;

/**
 * Zainicjuj dowolne usługi aplikacji.
 */
public function boot(): void
{
    User::observe(UserObserver::class);
}
```

> [!NOTE]
> Istnieją dodatkowe zdarzenia, których obsługi może nasłuchiwać obserwator, takie jak `saving` i `retrieved`. Te zdarzenia są opisane w dokumentacji [zdarzeń](#events).

<a name="observers-and-database-transactions"></a>
#### Obserwatorzy i Transakcje Bazy Danych

Kiedy modele są tworzone w ramach transakcji bazodanowej, możesz chcieć poinstruować obserwatora, aby wykonywał swoje obsługi zdarzeń dopiero po zatwierdzeniu transakcji bazodanowej. Możesz to osiągnąć, implementując interfejs `ShouldHandleEventsAfterCommit` w swoim obserwatorze. Jeśli transakcja bazodanowa nie jest w toku, obsługi zdarzeń zostaną wykonane natychmiast:

```php
<?php

namespace App\Observers;

use App\Models\User;
use Illuminate\Contracts\Events\ShouldHandleEventsAfterCommit;

class UserObserver implements ShouldHandleEventsAfterCommit
{
    /**
     * Obsługa zdarzenia User "created".
     */
    public function created(User $user): void
    {
        // ...
    }
}
```

<a name="muting-events"></a>
### Wyciszanie Zdarzeń

Czasami możesz potrzebować tymczasowo "wyciszyć" wszystkie zdarzenia wysyłane przez model. Możesz to osiągnąć, używając metody `withoutEvents`. Metoda `withoutEvents` przyjmuje zamknięcie jako jedyny argument. Wszelki kod wykonany wewnątrz tego zamknięcia nie będzie wysyłał zdarzeń modelu, a wszelka wartość zwracana przez zamknięcie zostanie zwrócona przez metodę `withoutEvents`:

```php
use App\Models\User;

$user = User::withoutEvents(function () {
    User::findOrFail(1)->delete();

    return User::find(2);
});
```

<a name="saving-a-single-model-without-events"></a>
#### Zapisywanie Pojedynczego Modelu Bez Zdarzeń

Czasami możesz chcieć "zapisać" dany model bez wysyłania jakichkolwiek zdarzeń. Możesz to osiągnąć, używając metody `saveQuietly`:

```php
$user = User::findOrFail(1);

$user->name = 'Victoria Faith';

$user->saveQuietly();
```

Możesz również "zaktualizować", "usunąć", "miękko usunąć", "przywrócić" i "replikować" dany model bez wysyłania jakichkolwiek zdarzeń:

```php
$user->deleteQuietly();
$user->forceDeleteQuietly();
$user->restoreQuietly();
```
