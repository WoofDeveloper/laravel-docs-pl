# Baza Danych: Migracje

- [Wprowadzenie](#introduction)
- [Generowanie Migracji](#generating-migrations)
    - [Zgniatanie Migracji](#squashing-migrations)
- [Struktura Migracji](#migration-structure)
- [Uruchamianie Migracji](#running-migrations)
    - [Wycofywanie Migracji](#rolling-back-migrations)
- [Tabele](#tables)
    - [Tworzenie Tabel](#creating-tables)
    - [Aktualizowanie Tabel](#updating-tables)
    - [Zmiana Nazw / Usuwanie Tabel](#renaming-and-dropping-tables)
- [Kolumny](#columns)
    - [Tworzenie Kolumn](#creating-columns)
    - [Dostępne Typy Kolumn](#available-column-types)
    - [Modyfikatory Kolumn](#column-modifiers)
    - [Modyfikowanie Kolumn](#modifying-columns)
    - [Zmiana Nazw Kolumn](#renaming-columns)
    - [Usuwanie Kolumn](#dropping-columns)
- [Indeksy](#indexes)
    - [Tworzenie Indeksów](#creating-indexes)
    - [Zmiana Nazw Indeksów](#renaming-indexes)
    - [Usuwanie Indeksów](#dropping-indexes)
    - [Ograniczenia Klucza Obcego](#foreign-key-constraints)
- [Zdarzenia](#events)

<a name="introduction"></a>
## Wprowadzenie

Migracje są jak kontrola wersji dla bazy danych, pozwalając zespołowi definiować i udostępniać definicję schematu bazy danych aplikacji. Jeśli kiedykolwiek musiałeś powiedzieć członkowi zespołu, aby ręcznie dodał kolumnę do lokalnego schematu bazy danych po pobraniu zmian z kontroli wersji, stanąłeś przed problemem, który rozwiązują migracje bazy danych.

[Fasada](/docs/{{version}}/facades) `Schema` Laravel zapewnia obsługę niezależną od bazy danych do tworzenia i manipulowania tabelami we wszystkich obsługiwanych systemach bazodanowych Laravel. Zazwyczaj migracje będą używać tej fasady do tworzenia i modyfikowania tabel i kolumn bazy danych.

<a name="generating-migrations"></a>
## Generowanie Migracji

Możesz użyć [polecenia Artisan](/docs/{{version}}/artisan) `make:migration` do wygenerowania migracji bazy danych. Nowa migracja zostanie umieszczona w katalogu `database/migrations`. Każda nazwa pliku migracji zawiera znacznik czasu, który pozwala Laravel określić kolejność migracji:

```shell
php artisan make:migration create_flights_table
```

Laravel będzie używać nazwy migracji do próby odgadnięcia nazwy tabeli i tego, czy migracja będzie tworzyć nową tabelę. Jeśli Laravel jest w stanie określić nazwę tabeli z nazwy migracji, Laravel wypełni wygenerowany plik migracji określoną tabelą. W przeciwnym razie możesz po prostu określić tabelę w pliku migracji ręcznie.

Jeśli chcesz określić niestandardową ścieżkę dla wygenerowanej migracji, możesz użyć opcji `--path` podczas wykonywania polecenia `make:migration`. Podana ścieżka powinna być względna w stosunku do ścieżki bazowej aplikacji.

> [!NOTE]
> Szablony migracji mogą być dostosowane za pomocą [publikowania szablonów](/docs/{{version}}/artisan#stub-customization).

<a name="squashing-migrations"></a>
### Zgniatanie Migracji

Podczas budowania aplikacji z czasem możesz zgromadzić coraz więcej migracji. Może to prowadzić do przepełnienia katalogu `database/migrations` potencjalnie setkami migracji. Jeśli chcesz, możesz "zgnieść" swoje migracje do jednego pliku SQL. Aby zacząć, wykonaj polecenie `schema:dump`:

```shell
php artisan schema:dump

# Dump the current database schema and prune all existing migrations...
php artisan schema:dump --prune
```

Gdy wykonasz to polecenie, Laravel zapisze plik "schema" do katalogu `database/schema` aplikacji. Nazwa pliku schematu będzie odpowiadać połączeniu bazy danych. Teraz, gdy spróbujesz zmigrować bazę danych i żadne inne migracje nie zostały wykonane, Laravel najpierw wykona instrukcje SQL w pliku schematu połączenia bazy danych, którego używasz. Po wykonaniu instrukcji SQL pliku schematu, Laravel wykona wszelkie pozostałe migracje, które nie były częścią zrzutu schematu.

Jeśli testy aplikacji używają innego połączenia bazy danych niż to, którego zazwyczaj używasz podczas lokalnego rozwoju, powinieneś upewnić się, że zrzuciłeś plik schematu używając tego połączenia bazy danych, aby testy mogły zbudować bazę danych. Możesz chcieć to zrobić po zrzuceniu połączenia bazy danych, którego zazwyczaj używasz podczas lokalnego rozwoju:

```shell
php artisan schema:dump
php artisan schema:dump --database=testing --prune
```

Powinieneś zatwierdzić plik schematu bazy danych do kontroli wersji, aby inni nowi programiści w zespole mogli szybko utworzyć początkową strukturę bazy danych aplikacji.

> [!WARNING]
> Zgniatanie migracji jest dostępne tylko dla baz danych MariaDB, MySQL, PostgreSQL i SQLite oraz wykorzystuje klienta wiersza poleceń bazy danych.

<a name="migration-structure"></a>
## Struktura Migracji

Klasa migracji zawiera dwie metody: `up` i `down`. Metoda `up` jest używana do dodawania nowych tabel, kolumn lub indeksów do bazy danych, podczas gdy metoda `down` powinna odwrócić operacje wykonane przez metodę `up`.

W obu tych metodach możesz użyć constructora schematów Laravel do ekspresyjnego tworzenia i modyfikowania tabel. Aby dowiedzieć się o wszystkich metodach dostępnych w konstruktorze `Schema`, [sprawdź jego dokumentację](#creating-tables). Na przykład, następująca migracja tworzy tabelę `flights`:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::drop('flights');
    }
};
```

<a name="setting-the-migration-connection"></a>
#### Ustawianie Połączenia Migracji

Jeśli twoja migracja będzie współdziałać z połączeniem bazy danych innym niż domyślne połączenie bazy danych aplikacji, powinieneś ustawić właściwość `$connection` migracji:

```php
/**
 * The database connection that should be used by the migration.
 *
 * @var string
 */
protected $connection = 'pgsql';

/**
 * Run the migrations.
 */
public function up(): void
{
    // ...
}
```

<a name="skipping-migrations"></a>
#### Pomijanie Migracji

Czasami migracja może być przeznaczona do obsługi funkcji, która nie jest jeszcze aktywna i nie chcesz, aby była jeszcze uruchamiana. W takim przypadku możesz zdefiniować metodę `shouldRun` w migracji. Jeśli metoda `shouldRun` zwróci `false`, migracja zostanie pominięta:

```php
use App\Models\Flight;
use Laravel\Pennant\Feature;

/**
 * Determine if this migration should run.
 */
public function shouldRun(): bool
{
    return Feature::active(Flight::class);
}
```

<a name="running-migrations"></a>
## Uruchamianie Migracji

Aby uruchomić wszystkie zaległe migracje, wykonaj polecenie Artisan `migrate`:

```shell
php artisan migrate
```

Jeśli chcesz zobaczyć, które migracje zostały już uruchomione, a które oczekują, możesz użyć polecenia Artisan `migrate:status`:

```shell
php artisan migrate:status
```

Jeśli chcesz zobaczyć instrukcje SQL, które zostaną wykonane przez migracje bez ich faktycznego uruchomienia, możesz dodać flagę `--pretend` do polecenia `migrate`:

```shell
php artisan migrate --pretend
```

<a name="isolating-migration-execution"></a>
#### Izolowanie Wykonywania Migracji

Jeśli wdrażasz aplikację na wielu serwerach i uruchamiasz migracje jako część procesu wdrożenia, prawdopodobnie nie chcesz, aby dwa serwery próbowały jednocześnie migrować bazę danych. Aby tego uniknąć, możesz użyć opcji `isolated` podczas wywoływania polecenia `migrate`.

Kiedy podana jest opcja `isolated`, Laravel uzyska blokadę atomową przy użyciu sterownika cache aplikacji przed próbą uruchomienia migracji. Wszystkie inne próby uruchomienia polecenia `migrate`, gdy ta blokada jest utrzymywana, nie zostaną wykonane; jednak polecenie nadal zakończy się z pomyslnym kodem wyjścia:

```shell
php artisan migrate --isolated
```

> [!WARNING]
> Aby użyć tej funkcji, aplikacja musi używać sterownika cache `memcached`, `redis`, `dynamodb`, `database`, `file` lub `array` jako domyślnego sterownika cache aplikacji. Ponadto wszystkie serwery muszą komunikować się z tym samym centralnym serwerem cache.

<a name="forcing-migrations-to-run-in-production"></a>
#### Wymuszanie Migracji w Produkcji

Niektóre operacje migracyjne są destrukcyjne, co oznacza, że mogą spowodować utratę danych. Aby chronić cię przed uruchomieniem tych poleceń w produkcyjnej bazie danych, przed wykonaniem poleceń zostaniesz poproszony o potwierdzenie. Aby wymusić uruchomienie poleceń bez monitu, użyj flagi `--force`:

```shell
php artisan migrate --force
```

<a name="rolling-back-migrations"></a>
### Wycofywanie Migracji

Aby wycofać najnowszą operację migracji, możesz użyć polecenia Artisan `rollback`. To polecenie wycofuje ostatnią "partię" migracji, która może zawierać wiele plików migracji:

```shell
php artisan migrate:rollback
```

Możesz wycofać ograniczoną liczbę migracji, podając opcję `step` do polecenia `rollback`. Na przykład, następujące polecenie wycofa ostatnie pięć migracji:

```shell
php artisan migrate:rollback --step=5
```

Możesz wycofać określoną "partię" migracji, podając opcję `batch` do polecenia `rollback`, gdzie opcja `batch` odpowiada wartości partii w tabeli `migrations` bazy danych aplikacji. Na przykład, następujące polecenie wycofa wszystkie migracje z partii trzeciej:

```shell
php artisan migrate:rollback --batch=3
```

Jeśli chcesz zobaczyć instrukcje SQL, które zostaną wykonane przez migracje bez ich faktycznego uruchomienia, możesz dodać flagę `--pretend` do polecenia `migrate:rollback`:

```shell
php artisan migrate:rollback --pretend
```

Polecenie `migrate:reset` wycofa wszystkie migracje aplikacji:

```shell
php artisan migrate:reset
```

<a name="roll-back-migrate-using-a-single-command"></a>
#### Wycofywanie i Migrowanie Za Pomocą Jednego Polecenia

Polecenie `migrate:refresh` wycofa wszystkie migracje, a następnie wykona polecenie `migrate`. To polecenie skutecznie odtworzy całą bazę danych:

```shell
php artisan migrate:refresh

# Refresh the database and run all database seeds...
php artisan migrate:refresh --seed
```

Możesz wycofać i ponownie zmigrować ograniczoną liczbę migracji, podając opcję `step` do polecenia `refresh`. Na przykład, następujące polecenie wycofa i ponownie zmigruje ostatnie pięć migracji:

```shell
php artisan migrate:refresh --step=5
```

<a name="drop-all-tables-migrate"></a>
#### Usunięcie Wszystkich Tabel i Migrowanie

Polecenie `migrate:fresh` usunie wszystkie tabele z bazy danych, a następnie wykona polecenie `migrate`:

```shell
php artisan migrate:fresh

php artisan migrate:fresh --seed
```

Domyślnie polecenie `migrate:fresh` usuwa tabele tylko z domyślnego połączenia bazy danych. Jednak możesz użyć opcji `--database`, aby określić połączenie bazy danych, które powinno zostać zmigrowane. Nazwa połączenia bazy danych powinna odpowiadać połączeniu zdefiniowanemu w [pliku konfiguracyjnym](/docs/{{version}}/configuration) `database` aplikacji:

```shell
php artisan migrate:fresh --database=admin
```

> [!WARNING]
> Polecenie `migrate:fresh` usunie wszystkie tabele bazy danych niezależnie od ich prefiksu. To polecenie powinno być używane ostrożnie podczas pracy na bazie danych wspólnej z innymi aplikacjami.

<a name="tables"></a>
## Tabele

<a name="creating-tables"></a>
### Tworzenie Tabel

Aby utworzyć nową tabelę bazy danych, użyj metody `create` na fasadzie `Schema`. Metoda `create` przyjmuje dwa argumenty: pierwszy to nazwa tabeli, a drugi to zamknięcie, które otrzymuje obiekt `Blueprint`, który może być użyty do zdefiniowania nowej tabeli:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email');
    $table->timestamps();
});
```

Podczas tworzenia tabeli możesz użyć dowolnej z [metod kolumn](#creating-columns) constructora schematów, aby zdefiniować kolumny tabeli.

<a name="determining-table-column-existence"></a>
#### Określanie Istnienia Tabeli / Kolumny

Możesz określić istnienie tabeli, kolumny lub indeksu za pomocą metod `hasTable`, `hasColumn` i `hasIndex`:

```php
if (Schema::hasTable('users')) {
    // The "users" table exists...
}

if (Schema::hasColumn('users', 'email')) {
    // The "users" table exists and has an "email" column...
}

if (Schema::hasIndex('users', ['email'], 'unique')) {
    // The "users" table exists and has a unique index on the "email" column...
}
```

<a name="database-connection-table-options"></a>
#### Połączenie Bazy Danych i Opcje Tabeli

Jeśli chcesz wykonać operację schematu na połączeniu bazy danych, które nie jest domyślnym połączeniem aplikacji, użyj metody `connection`:

```php
Schema::connection('sqlite')->create('users', function (Blueprint $table) {
    $table->id();
});
```

Ponadto kilka innych właściwości i metod może być używanych do definiowania innych aspektów tworzenia tabeli. Właściwość `engine` może być używana do określenia silnika pamięci tabeli podczas używania MariaDB lub MySQL:

```php
Schema::create('users', function (Blueprint $table) {
    $table->engine('InnoDB');

    // ...
});
```

Właściwości `charset` i `collation` mogą być używane do określenia zestawu znaków i sortowania dla tworzonej tabeli podczas używania MariaDB lub MySQL:

```php
Schema::create('users', function (Blueprint $table) {
    $table->charset('utf8mb4');
    $table->collation('utf8mb4_unicode_ci');

    // ...
});
```

Metoda `temporary` może być używana do wskazania, że tabela powinna być "tymczasowa". Tabele tymczasowe są widoczne tylko dla sesji bazy danych bieżącego połączenia i są automatycznie usuwane po zamknięciu połączenia:

```php
Schema::create('calculations', function (Blueprint $table) {
    $table->temporary();

    // ...
});
```

Jeśli chcesz dodać "komentarz" do tabeli bazy danych, możesz wywołać metodę `comment` na instancji tabeli. Komentarze do tabel są obecnie obsługiwane tylko przez MariaDB, MySQL i PostgreSQL:

```php
Schema::create('calculations', function (Blueprint $table) {
    $table->comment('Business calculations');

    // ...
});
```

<a name="updating-tables"></a>
### Aktualizowanie Tabel

Metoda `table` na fasadzie `Schema` może być używana do aktualizowania istniejących tabel. Podobnie jak metoda `create`, metoda `table` przyjmuje dwa argumenty: nazwę tabeli i zamknięcie, które otrzymuje instancję `Blueprint`, której możesz użyć do dodawania kolumn lub indeksów do tabeli:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

<a name="renaming-and-dropping-tables"></a>
### Zmiana Nazw / Usuwanie Tabel

Aby zmienić nazwę istniejącej tabeli bazy danych, użyj metody `rename`:

```php
use Illuminate\Support\Facades\Schema;

Schema::rename($from, $to);
```

Aby usunąć istniejącą tabelę, możesz użyć metod `drop` lub `dropIfExists`:

```php
Schema::drop('users');

Schema::dropIfExists('users');
```

<a name="renaming-tables-with-foreign-keys"></a>
#### Zmiana Nazw Tabel z Kluczami Obcymi

Przed zmianą nazwy tabeli powinieneś zweryfikować, że wszystkie ograniczenia klucza obcego w tabeli mają wyraźną nazwę w plikach migracji zamiast pozwalać Laravel przypisywać nazwę zgodnie z konwencją. W przeciwnym razie nazwa ograniczenia klucza obcego będzie odnosić się do starej nazwy tabeli.

<a name="columns"></a>
## Kolumny

<a name="creating-columns"></a>
### Tworzenie Kolumn

Metoda `table` na fasadzie `Schema` może być używana do aktualizowania istniejących tabel. Podobnie jak metoda `create`, metoda `table` przyjmuje dwa argumenty: nazwę tabeli i zamknięcie, które otrzymuje instancję `Illuminate\Database\Schema\Blueprint`, której możesz użyć do dodawania kolumn do tabeli:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

<a name="available-column-types"></a>
### Dostępne Typy Kolumn

Constructor blueprint schema builder oferuje różnorodne metody odpowiadające różnym typom kolumn, które możesz dodać do tabel bazy danych. Każda z dostępnych metod jest wymieniona w poniższej tabeli:

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

    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="booleans-method-list"></a>
#### Typy Logiczne

<div class="collection-method-list" markdown="1">

[boolean](#column-method-boolean)

</div>

<a name="strings-and-texts-method-list"></a>
#### Typy Tekstowe

<div class="collection-method-list" markdown="1">

[char](#column-method-char)
[longText](#column-method-longText)
[mediumText](#column-method-mediumText)
[string](#column-method-string)
[text](#column-method-text)
[tinyText](#column-method-tinyText)

</div>

<a name="numbers--method-list"></a>
#### Typy Numeryczne

<div class="collection-method-list" markdown="1">

[bigIncrements](#column-method-bigIncrements)
[bigInteger](#column-method-bigInteger)
[decimal](#column-method-decimal)
[double](#column-method-double)
[float](#column-method-float)
[id](#column-method-id)
[increments](#column-method-increments)
[integer](#column-method-integer)
[mediumIncrements](#column-method-mediumIncrements)
[mediumInteger](#column-method-mediumInteger)
[smallIncrements](#column-method-smallIncrements)
[smallInteger](#column-method-smallInteger)
[tinyIncrements](#column-method-tinyIncrements)
[tinyInteger](#column-method-tinyInteger)
[unsignedBigInteger](#column-method-unsignedBigInteger)
[unsignedInteger](#column-method-unsignedInteger)
[unsignedMediumInteger](#column-method-unsignedMediumInteger)
[unsignedSmallInteger](#column-method-unsignedSmallInteger)
[unsignedTinyInteger](#column-method-unsignedTinyInteger)

</div>

<a name="dates-and-times-method-list"></a>
#### Typy Daty i Czasu

<div class="collection-method-list" markdown="1">

[dateTime](#column-method-dateTime)
[dateTimeTz](#column-method-dateTimeTz)
[date](#column-method-date)
[time](#column-method-time)
[timeTz](#column-method-timeTz)
[timestamp](#column-method-timestamp)
[timestamps](#column-method-timestamps)
[timestampsTz](#column-method-timestampsTz)
[softDeletes](#column-method-softDeletes)
[softDeletesTz](#column-method-softDeletesTz)
[year](#column-method-year)

</div>

<a name="binaries-method-list"></a>
#### Typy Binarne

<div class="collection-method-list" markdown="1">

[binary](#column-method-binary)

</div>

<a name="object-and-jsons-method-list"></a>
#### Typy Obiektów i JSON

<div class="collection-method-list" markdown="1">

[json](#column-method-json)
[jsonb](#column-method-jsonb)

</div>

<a name="uuids-and-ulids-method-list"></a>
#### Typy UUID i ULID

<div class="collection-method-list" markdown="1">

[ulid](#column-method-ulid)
[ulidMorphs](#column-method-ulidMorphs)
[uuid](#column-method-uuid)
[uuidMorphs](#column-method-uuidMorphs)
[nullableUlidMorphs](#column-method-nullableUlidMorphs)
[nullableUuidMorphs](#column-method-nullableUuidMorphs)

</div>

<a name="spatials-method-list"></a>
#### Typy Przestrzenne

<div class="collection-method-list" markdown="1">

[geography](#column-method-geography)
[geometry](#column-method-geometry)

</div>

<a name="relationship-method-list"></a>
#### Typy Relacji

<div class="collection-method-list" markdown="1">

[foreignId](#column-method-foreignId)
[foreignIdFor](#column-method-foreignIdFor)
[foreignUlid](#column-method-foreignUlid)
[foreignUuid](#column-method-foreignUuid)
[morphs](#column-method-morphs)
[nullableMorphs](#column-method-nullableMorphs)

</div>

<a name="spacifics-method-list"></a>
#### Typy Specjalne

<div class="collection-method-list" markdown="1">

[enum](#column-method-enum)
[set](#column-method-set)
[macAddress](#column-method-macAddress)
[ipAddress](#column-method-ipAddress)
[rememberToken](#column-method-rememberToken)
[vector](#column-method-vector)

</div>

<a name="column-method-bigIncrements"></a>
#### `bigIncrements()` {.collection-method .first-collection-method}

Metoda `bigIncrements` tworzy autoinkrementacyjną kolumnę równoważną `UNSIGNED BIGINT` (klucz główny):

```php
$table->bigIncrements('id');
```

<a name="column-method-bigInteger"></a>
#### `bigInteger()` {.collection-method}

Metoda `bigInteger` tworzy kolumnę równoważną `BIGINT`:

```php
$table->bigInteger('votes');
```

<a name="column-method-binary"></a>
#### `binary()` {.collection-method}

Metoda `binary` tworzy kolumnę równoważną `BLOB`:

```php
$table->binary('photo');
```

Podczas używania MySQL, MariaDB lub SQL Server, możesz przekazać argumenty `length` i `fixed`, aby utworzyć kolumnę równoważną `VARBINARY` lub `BINARY`:

```php
$table->binary('data', length: 16); // VARBINARY(16)

$table->binary('data', length: 16, fixed: true); // BINARY(16)
```

<a name="column-method-boolean"></a>
#### `boolean()` {.collection-method}

Metoda `boolean` tworzy kolumnę równoważną `BOOLEAN`:

```php
$table->boolean('confirmed');
```

<a name="column-method-char"></a>
#### `char()` {.collection-method}

Metoda `char` tworzy kolumnę równoważną `CHAR` o podanej długości:

```php
$table->char('name', length: 100);
```

<a name="column-method-dateTimeTz"></a>
#### `dateTimeTz()` {.collection-method}

Metoda `dateTimeTz` tworzy kolumnę równoważną `DATETIME` (ze strefą czasową) z opcjonalną precyzją sekund ułamkowych:

```php
$table->dateTimeTz('created_at', precision: 0);
```

<a name="column-method-dateTime"></a>
#### `dateTime()` {.collection-method}

Metoda `dateTime` tworzy kolumnę równoważną `DATETIME` z opcjonalną precyzją sekund ułamkowych:

```php
$table->dateTime('created_at', precision: 0);
```

<a name="column-method-date"></a>
#### `date()` {.collection-method}

Metoda `date` tworzy kolumnę równoważną `DATE`:

```php
$table->date('created_at');
```

<a name="column-method-decimal"></a>
#### `decimal()` {.collection-method}

Metoda `decimal` tworzy kolumnę równoważną `DECIMAL` z podaną precyzją (całkowita liczba cyfr) i skalą (cyfry dziesiętne):

```php
$table->decimal('amount', total: 8, places: 2);
```

<a name="column-method-double"></a>
#### `double()` {.collection-method}

Metoda `double` tworzy kolumnę równoważną `DOUBLE`:

```php
$table->double('amount');
```

<a name="column-method-enum"></a>
#### `enum()` {.collection-method}

Metoda `enum` tworzy kolumnę równoważną `ENUM` z podanymi poprawnymi wartościami:

```php
$table->enum('difficulty', ['easy', 'hard']);
```

Oczywiście możesz użyć metody `Enum::cases()` zamiast ręcznego definiowania tablicy dozwolonych wartości:

```php
use App\Enums\Difficulty;

$table->enum('difficulty', Difficulty::cases());
```

<a name="column-method-float"></a>
#### `float()` {.collection-method}

Metoda `float` tworzy kolumnę równoważną `FLOAT` z podaną precyzją:

```php
$table->float('amount', precision: 53);
```

<a name="column-method-foreignId"></a>
#### `foreignId()` {.collection-method}

Metoda `foreignId` tworzy kolumnę równoważną `UNSIGNED BIGINT`:

```php
$table->foreignId('user_id');
```

<a name="column-method-foreignIdFor"></a>
#### `foreignIdFor()` {.collection-method}

Metoda `foreignIdFor` dodaje kolumnę równoważną `{column}_id` dla podanej klasy modelu. Typ kolumny będzie `UNSIGNED BIGINT`, `CHAR(36)` lub `CHAR(26)` w zależności od typu klucza modelu:

```php
$table->foreignIdFor(User::class);
```

<a name="column-method-foreignUlid"></a>
#### `foreignUlid()` {.collection-method}

Metoda `foreignUlid` tworzy kolumnę równoważną `ULID`:

```php
$table->foreignUlid('user_id');
```

<a name="column-method-foreignUuid"></a>
#### `foreignUuid()` {.collection-method}

Metoda `foreignUuid` tworzy kolumnę równoważną `UUID`:

```php
$table->foreignUuid('user_id');
```

<a name="column-method-geography"></a>
#### `geography()` {.collection-method}

Metoda `geography` tworzy kolumnę równoważną `GEOGRAPHY` z podanym typem przestrzennym i SRID (Spatial Reference System Identifier):

```php
$table->geography('coordinates', subtype: 'point', srid: 4326);
```

> [!NOTE]
> Obsługa typów przestrzennych zależy od sterownika bazy danych. Zapoznaj się z dokumentacją bazy danych. Jeśli aplikacja używa bazy danych PostgreSQL, musisz zainstalować rozszerzenie [PostGIS](https://postgis.net) przed użyciem metody `geography`.

<a name="column-method-geometry"></a>
#### `geometry()` {.collection-method}

Metoda `geometry` tworzy kolumnę równoważną `GEOMETRY` z podanym typem przestrzennym i SRID (Spatial Reference System Identifier):

```php
$table->geometry('positions', subtype: 'point', srid: 0);
```

> [!NOTE]
> Obsługa typów przestrzennych zależy od sterownika bazy danych. Zapoznaj się z dokumentacją bazy danych. Jeśli aplikacja używa bazy danych PostgreSQL, musisz zainstalować rozszerzenie [PostGIS](https://postgis.net) przed użyciem metody `geometry`.

<a name="column-method-id"></a>
#### `id()` {.collection-method}

Metoda `id` jest aliasem metody `bigIncrements`. Domyślnie metoda utworzy kolumnę `id`; jednak możesz przekazać nazwę kolumny, jeśli chcesz przypisać inną nazwę kolumnie:

```php
$table->id();
```

<a name="column-method-increments"></a>
#### `increments()` {.collection-method}

Metoda `increments` tworzy autoinkrementacyjną kolumnę równoważną `UNSIGNED INTEGER` jako klucz główny:

```php
$table->increments('id');
```

<a name="column-method-integer"></a>
#### `integer()` {.collection-method}

Metoda `integer` tworzy kolumnę równoważną `INTEGER`:

```php
$table->integer('votes');
```

<a name="column-method-ipAddress"></a>
#### `ipAddress()` {.collection-method}

Metoda `ipAddress` tworzy kolumnę równoważną `VARCHAR`:

```php
$table->ipAddress('visitor');
```

Podczas używania PostgreSQL zostanie utworzona kolumna `INET`.

<a name="column-method-json"></a>
#### `json()` {.collection-method}

Metoda `json` tworzy kolumnę równoważną `JSON`:

```php
$table->json('options');
```

Podczas używania SQLite zostanie utworzona kolumna `TEXT`.

<a name="column-method-jsonb"></a>
#### `jsonb()` {.collection-method}

Metoda `jsonb` tworzy kolumnę równoważną `JSONB`:

```php
$table->jsonb('options');
```

Podczas używania SQLite zostanie utworzona kolumna `TEXT`.

<a name="column-method-longText"></a>
#### `longText()` {.collection-method}

Metoda `longText` tworzy kolumnę równoważną `LONGTEXT`:

```php
$table->longText('description');
```

Podczas używania MySQL lub MariaDB możesz zastosować zestaw znaków `binary` do kolumny, aby utworzyć kolumnę równoważną `LONGBLOB`:

```php
$table->longText('data')->charset('binary'); // LONGBLOB
```

<a name="column-method-macAddress"></a>
#### `macAddress()` {.collection-method}

Metoda `macAddress` tworzy kolumnę przeznaczoną do przechowywania adresu MAC. Niektóre systemy bazodanowe, takie jak PostgreSQL, mają dedykowany typ kolumny dla tego typu danych. Inne systemy bazodanowe użyją kolumny równoważnej tekstowej:

```php
$table->macAddress('device');
```

<a name="column-method-mediumIncrements"></a>
#### `mediumIncrements()` {.collection-method}

Metoda `mediumIncrements` tworzy autoinkrementacyjną kolumnę równoważną `UNSIGNED MEDIUMINT` jako klucz główny:

```php
$table->mediumIncrements('id');
```

<a name="column-method-mediumInteger"></a>
#### `mediumInteger()` {.collection-method}

Metoda `mediumInteger` tworzy kolumnę równoważną `MEDIUMINT`:

```php
$table->mediumInteger('votes');
```

<a name="column-method-mediumText"></a>
#### `mediumText()` {.collection-method}

Metoda `mediumText` tworzy kolumnę równoważną `MEDIUMTEXT`:

```php
$table->mediumText('description');
```

Podczas używania MySQL lub MariaDB możesz zastosować zestaw znaków `binary` do kolumny, aby utworzyć kolumnę równoważną `MEDIUMBLOB`:

```php
$table->mediumText('data')->charset('binary'); // MEDIUMBLOB
```

<a name="column-method-morphs"></a>
#### `morphs()` {.collection-method}

Metoda `morphs` jest metodą wygody, która dodaje kolumnę równoważną `{column}_id` i kolumnę równoważną `{column}_type` `VARCHAR`. Typ kolumny dla `{column}_id` będzie `UNSIGNED BIGINT`, `CHAR(36)` lub `CHAR(26)` w zależności od typu klucza modelu.

Ta metoda jest przeznaczona do użycia podczas definiowania kolumn niezbędnych dla polimorficznej [relacji Eloquent](/docs/{{version}}/eloquent-relationships). W poniższym przykładzie zostaną utworzone kolumny `taggable_id` i `taggable_type`:

```php
$table->morphs('taggable');
```

<a name="column-method-nullableMorphs"></a>
#### `nullableMorphs()` {.collection-method}

Metoda jest podobna do metody [morphs](#column-method-morphs); jednak utworzone kolumny będą "nullable":

```php
$table->nullableMorphs('taggable');
```

<a name="column-method-nullableUlidMorphs"></a>
#### `nullableUlidMorphs()` {.collection-method}

Metoda jest podobna do metody [ulidMorphs](#column-method-ulidMorphs); jednak utworzone kolumny będą "nullable":

```php
$table->nullableUlidMorphs('taggable');
```

<a name="column-method-nullableUuidMorphs"></a>
#### `nullableUuidMorphs()` {.collection-method}

Metoda jest podobna do metody [uuidMorphs](#column-method-uuidMorphs); jednak utworzone kolumny będą "nullable":

```php
$table->nullableUuidMorphs('taggable');
```

<a name="column-method-rememberToken"></a>
#### `rememberToken()` {.collection-method}

Metoda `rememberToken` tworzy nullable kolumnę równoważną `VARCHAR(100)`, która jest przeznaczona do przechowywania bieżącego [tokenu uwierzytelniania](/docs/{{version}}/authentication#remembering-users) "zapamiętaj mnie":

```php
$table->rememberToken();
```

<a name="column-method-set"></a>
#### `set()` {.collection-method}

Metoda `set` tworzy kolumnę równoważną `SET` z podaną listą poprawnych wartości:

```php
$table->set('flavors', ['strawberry', 'vanilla']);
```

<a name="column-method-smallIncrements"></a>
#### `smallIncrements()` {.collection-method}

Metoda `smallIncrements` tworzy autoinkrementacyjną kolumnę równoważną `UNSIGNED SMALLINT` jako klucz główny:

```php
$table->smallIncrements('id');
```

<a name="column-method-smallInteger"></a>
#### `smallInteger()` {.collection-method}

Metoda `smallInteger` tworzy kolumnę równoważną `SMALLINT`:

```php
$table->smallInteger('votes');
```

<a name="column-method-softDeletesTz"></a>
#### `softDeletesTz()` {.collection-method}

Metoda `softDeletesTz` dodaje nullable kolumnę równoważną `deleted_at` `TIMESTAMP` (ze strefą czasową) z opcjonalną precyzją sekund ułamkowych. Ta kolumna jest przeznaczona do przechowywania znacznika czasu `deleted_at` potrzebnego dla funkcjonalności "soft delete" Eloquent:

```php
$table->softDeletesTz('deleted_at', precision: 0);
```

<a name="column-method-softDeletes"></a>
#### `softDeletes()` {.collection-method}

Metoda `softDeletes` dodaje nullable kolumnę równoważną `deleted_at` `TIMESTAMP` z opcjonalną precyzją sekund ułamkowych. Ta kolumna jest przeznaczona do przechowywania znacznika czasu `deleted_at` potrzebnego dla funkcjonalności "soft delete" Eloquent:

```php
$table->softDeletes('deleted_at', precision: 0);
```

<a name="column-method-string"></a>
#### `string()` {.collection-method}

Metoda `string` tworzy kolumnę równoważną `VARCHAR` o podanej długości:

```php
$table->string('name', length: 100);
```

<a name="column-method-text"></a>
#### `text()` {.collection-method}

Metoda `text` tworzy kolumnę równoważną `TEXT`:

```php
$table->text('description');
```

Podczas używania MySQL lub MariaDB możesz zastosować zestaw znaków `binary` do kolumny, aby utworzyć kolumnę równoważną `BLOB`:

```php
$table->text('data')->charset('binary'); // BLOB
```

<a name="column-method-timeTz"></a>
#### `timeTz()` {.collection-method}

Metoda `timeTz` tworzy kolumnę równoważną `TIME` (ze strefą czasową) z opcjonalną precyzją sekund ułamkowych:

```php
$table->timeTz('sunrise', precision: 0);
```

<a name="column-method-time"></a>
#### `time()` {.collection-method}

Metoda `time` tworzy kolumnę równoważną `TIME` z opcjonalną precyzją sekund ułamkowych:

```php
$table->time('sunrise', precision: 0);
```

<a name="column-method-timestampTz"></a>
#### `timestampTz()` {.collection-method}

Metoda `timestampTz` tworzy kolumnę równoważną `TIMESTAMP` (ze strefą czasową) z opcjonalną precyzją sekund ułamkowych:

```php
$table->timestampTz('added_at', precision: 0);
```

<a name="column-method-timestamp"></a>
#### `timestamp()` {.collection-method}

Metoda `timestamp` tworzy kolumnę równoważną `TIMESTAMP` z opcjonalną precyzją sekund ułamkowych:

```php
$table->timestamp('added_at', precision: 0);
```

<a name="column-method-timestampsTz"></a>
#### `timestampsTz()` {.collection-method}

Metoda `timestampsTz` tworzy kolumny równoważne `created_at` i `updated_at` `TIMESTAMP` (ze strefą czasową) z opcjonalną precyzją sekund ułamkowych:

```php
$table->timestampsTz(precision: 0);
```

<a name="column-method-timestamps"></a>
#### `timestamps()` {.collection-method}

Metoda `timestamps` tworzy kolumny równoważne `created_at` i `updated_at` `TIMESTAMP` z opcjonalną precyzją sekund ułamkowych:

```php
$table->timestamps(precision: 0);
```

<a name="column-method-tinyIncrements"></a>
#### `tinyIncrements()` {.collection-method}

Metoda `tinyIncrements` tworzy autoinkrementacyjną kolumnę równoważną `UNSIGNED TINYINT` jako klucz główny:

```php
$table->tinyIncrements('id');
```

<a name="column-method-tinyInteger"></a>
#### `tinyInteger()` {.collection-method}

Metoda `tinyInteger` tworzy kolumnę równoważną `TINYINT`:

```php
$table->tinyInteger('votes');
```

<a name="column-method-tinyText"></a>
#### `tinyText()` {.collection-method}

Metoda `tinyText` tworzy kolumnę równoważną `TINYTEXT`:

```php
$table->tinyText('notes');
```

Podczas używania MySQL lub MariaDB możesz zastosować zestaw znaków `binary` do kolumny, aby utworzyć kolumnę równoważną `TINYBLOB`:

```php
$table->tinyText('data')->charset('binary'); // TINYBLOB
```

<a name="column-method-unsignedBigInteger"></a>
#### `unsignedBigInteger()` {.collection-method}

Metoda `unsignedBigInteger` tworzy kolumnę równoważną `UNSIGNED BIGINT`:

```php
$table->unsignedBigInteger('votes');
```

<a name="column-method-unsignedInteger"></a>
#### `unsignedInteger()` {.collection-method}

Metoda `unsignedInteger` tworzy kolumnę równoważną `UNSIGNED INTEGER`:

```php
$table->unsignedInteger('votes');
```

<a name="column-method-unsignedMediumInteger"></a>
#### `unsignedMediumInteger()` {.collection-method}

Metoda `unsignedMediumInteger` tworzy kolumnę równoważną `UNSIGNED MEDIUMINT`:

```php
$table->unsignedMediumInteger('votes');
```

<a name="column-method-unsignedSmallInteger"></a>
#### `unsignedSmallInteger()` {.collection-method}

Metoda `unsignedSmallInteger` tworzy kolumnę równoważną `UNSIGNED SMALLINT`:

```php
$table->unsignedSmallInteger('votes');
```

<a name="column-method-unsignedTinyInteger"></a>
#### `unsignedTinyInteger()` {.collection-method}

Metoda `unsignedTinyInteger` tworzy kolumnę równoważną `UNSIGNED TINYINT`:

```php
$table->unsignedTinyInteger('votes');
```

<a name="column-method-ulidMorphs"></a>
#### `ulidMorphs()` {.collection-method}

Metoda `ulidMorphs` jest metodą wygody, która dodaje kolumnę równoważną `{column}_id` `CHAR(26)` i kolumnę równoważną `{column}_type` `VARCHAR`.

Ta metoda jest przeznaczona do użycia podczas definiowania kolumn niezbędnych dla polimorficznej [relacji Eloquent](/docs/{{version}}/eloquent-relationships), która używa identyfikatorów ULID. W poniższym przykładzie zostaną utworzone kolumny `taggable_id` i `taggable_type`:

```php
$table->ulidMorphs('taggable');
```

<a name="column-method-uuidMorphs"></a>
#### `uuidMorphs()` {.collection-method}

Metoda `uuidMorphs` jest metodą wygody, która dodaje kolumnę równoważną `{column}_id` `CHAR(36)` i kolumnę równoważną `{column}_type` `VARCHAR`.

Ta metoda jest przeznaczona do użycia podczas definiowania kolumn niezbędnych dla [polimorficznej relacji Eloquent](/docs/{{version}}/eloquent-relationships#polymorphic-relationships), która używa identyfikatorów UUID. W poniższym przykładzie zostaną utworzone kolumny `taggable_id` i `taggable_type`:

```php
$table->uuidMorphs('taggable');
```

<a name="column-method-ulid"></a>
#### `ulid()` {.collection-method}

Metoda `ulid` tworzy kolumnę równoważną `ULID`:

```php
$table->ulid('id');
```

<a name="column-method-uuid"></a>
#### `uuid()` {.collection-method}

Metoda `uuid` tworzy kolumnę równoważną `UUID`:

```php
$table->uuid('id');
```

<a name="column-method-vector"></a>
#### `vector()` {.collection-method}

Metoda `vector` tworzy kolumnę równoważną `vector`:

```php
$table->vector('embedding', dimensions: 100);
```

<a name="column-method-year"></a>
#### `year()` {.collection-method}

Metoda `year` tworzy kolumnę równoważną `YEAR`:

```php
$table->year('birth_year');
```

<a name="column-modifiers"></a>
### Modyfikatory Kolumn

Oprócz typów kolumn wymienionych powyżej, istnieje kilka "modyfikatorów" kolumn, których możesz użyć podczas dodawania kolumny do tabeli bazy danych. Na przykład, aby uczynić kolumnę "nullable", możesz użyć metody `nullable`:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->string('email')->nullable();
});
```

Poniższa tabela zawiera wszystkie dostępne modyfikatory kolumn. Ta lista nie zawiera [modyfikatorów indeksów](#creating-indexes):

<div class="overflow-auto">

| Modyfikator                             | Opis                                                                                                           |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `->after('column')`                 | Umieść kolumnę "po" innej kolumnie (MariaDB / MySQL).                                                        |
| `->autoIncrement()`                 | Ustaw kolumny `INTEGER` jako autoinkrementacyjne (klucz główny).                                                |
| `->charset('utf8mb4')`              | Określ zestaw znaków dla kolumny (MariaDB / MySQL).                                                          |
| `->collation('utf8mb4_unicode_ci')` | Określ sortowanie dla kolumny.                                                                                 |
| `->comment('my comment')`           | Dodaj komentarz do kolumny (MariaDB / MySQL / PostgreSQL).                                                    |
| `->default($value)`                 | Określ "domyślną" wartość dla kolumny.                                                                      |
| `->first()`                         | Umieść kolumnę jako "pierwszą" w tabeli (MariaDB / MySQL).                                                   |
| `->from($integer)`                  | Ustaw wartość początkową pola autoinkrementacyjnego (MariaDB / MySQL / PostgreSQL).                         |
| `->instant()`                       | Dodaj lub zmodyfikuj kolumnę używając operacji natychmiastowej (MySQL).                                       |
| `->invisible()`                     | Uczyń kolumnę "niewidzialną" dla zapytań `SELECT *` (MariaDB / MySQL).                                    |
| `->lock($mode)`                     | Określ tryb blokady dla operacji kolumny (MySQL).                                                             |
| `->nullable($value = true)`         | Pozwól na wstawianie wartości `NULL` do kolumny.                                                              |
| `->storedAs($expression)`           | Utwórz przechowywaną wygenerowane kolumnę (MariaDB / MySQL / PostgreSQL / SQLite).                         |
| `->unsigned()`                      | Ustaw kolumny `INTEGER` jako `UNSIGNED` (MariaDB / MySQL).                                                    |
| `->useCurrent()`                    | Ustaw kolumny `TIMESTAMP` aby używały `CURRENT_TIMESTAMP` jako wartości domyślnej.                         |
| `->useCurrentOnUpdate()`            | Ustaw kolumny `TIMESTAMP` aby używały `CURRENT_TIMESTAMP` gdy rekord jest aktualizowany (MariaDB / MySQL). |
| `->virtualAs($expression)`          | Utwórz wirtualną wygenerowane kolumnę (MariaDB / MySQL / SQLite).                                           |
| `->generatedAs($expression)`        | Utwórz kolumnę tożsamości z określonymi opcjami sekwencji (PostgreSQL).                                    |
| `->always()`                        | Definiuje pierwszeństwo wartości sekwencji nad wejściem dla kolumny tożsamości (PostgreSQL).              |

</div>

<a name="default-expressions"></a>
#### Wyrażenia Domyślne

Modyfikator `default` akceptuje wartość lub instancję `Illuminate\Database\Query\Expression`. Użycie instancji `Expression` zapobiegnie owinięciu wartości w cudzysłowy przez Laravel i pozwoli na użycie funkcji specyficznych dla bazy danych. Jedna sytuacja, w której jest to szczególnie przydatne, to przypisywanie wartości domyślnych do kolumn JSON:

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Query\Expression;
use Illuminate\Database\Migrations\Migration;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->json('movies')->default(new Expression('(JSON_ARRAY())'));
            $table->timestamps();
        });
    }
};
```

> [!WARNING]
> Obsługa wyrażeń domyślnych zależy od sterownika bazy danych, wersji bazy danych i typu pola. Zapoznaj się z dokumentacją bazy danych.

<a name="column-order"></a>
#### Kolejność Kolumn

Podczas używania bazy danych MariaDB lub MySQL, metoda `after` może być używana do dodawania kolumn po istniejącej kolumnie w schemacie:

```php
$table->after('password', function (Blueprint $table) {
    $table->string('address_line1');
    $table->string('address_line2');
    $table->string('city');
});
```

<a name="instant-column-operations"></a>
#### Operacje Natychmiastowe na Kolumnach

Podczas używania MySQL możesz użyć modyfikatora `instant` w łańcuchu definicji kolumny, aby wskazać, że kolumna powinna zostać dodana lub zmodyfikowana za pomocą algorytmu "instant" MySQL. Ten algorytm pozwala na wykonanie niektórych zmian schematu bez pełnej przebudowy tabeli, czyniąc je niemal natychmiastowymi niezależnie od rozmiaru tabeli:

```php
$table->string('name')->nullable()->instant();
```

Natychmiastowe dodawanie kolumn może tylko dodać kolumny na końcu tabeli, więc modyfikator `instant` nie może być łączony z modyfikatorami `after` lub `first`. Ponadto algorytm nie obsługuje wszystkich typów kolumn ani operacji. Jeśli żądana operacja jest niekompatybilna, MySQL zgłosi błąd.

Zapoznaj się z [dokumentacją MySQL](https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-operations.html), aby określić, które operacje są kompatybilne z natychmiastowymi modyfikacjami kolumn.

<a name="ddl-locking"></a>
#### Blokowanie DDL

Podczas używania MySQL możesz użyć modyfikatora `lock` w łańcuchu definicji kolumny, indeksu lub klucza obcego, aby kontrolować blokowanie tabeli podczas operacji schematu. MySQL obsługuje kilka trybów blokady: `none` pozwala na równoczesne odczyty i zapisy, `shared` pozwala na równoczesne odczyty, ale blokuje zapisy, `exclusive` blokuje cały równoczesny dostęp, a `default` pozwala MySQL wybrać najbardziej odpowiedni tryb:

```php
$table->string('name')->lock('none');

$table->index('email')->lock('shared');
```

Jeśli żądany tryb blokady jest niekompatybilny z operacją, MySQL zgłosi błąd. Modyfikator `lock` może być łączony z modyfikatorem `instant`, aby dalej optymalizować zmiany schematu:

```php
$table->string('name')->instant()->lock('none');
```

<a name="modifying-columns"></a>
### Modyfikowanie Kolumn

Metoda `change` pozwala modyfikować typ i atrybuty istniejących kolumn. Na przykład, możesz chcieć zwiększyć rozmiar kolumny `string`. Aby zobaczyć metodę `change` w akcji, zwiększmy rozmiar kolumny `name` z 25 do 50. Aby to osiągnąć, po prostu definiujemy nowy stan kolumny, a następnie wywołujemy metodę `change`:

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->change();
});
```

Podczas modyfikowania kolumny musisz wyraźnie uwzględnić wszystkie modyfikatory, które chcesz zachować w definicji kolumny - wszelkie brakujące atrybuty zostaną usunięte. Na przykład, aby zachować atrybuty `unsigned`, `default` i `comment`, musisz wywołać każdy modyfikator wyraźnie podczas zmieniania kolumny:

```php
Schema::table('users', function (Blueprint $table) {
    $table->integer('votes')->unsigned()->default(1)->comment('my comment')->change();
});
```

Metoda `change` nie zmienia indeksów kolumny. Dlatego możesz użyć modyfikatorów indeksu, aby wyraźnie dodać lub usunąć indeks podczas modyfikowania kolumny:

```php
// Add an index...
$table->bigIncrements('id')->primary()->change();

// Drop an index...
$table->char('postal_code', 10)->unique(false)->change();
```

<a name="renaming-columns"></a>
### Zmiana Nazw Kolumn

Aby zmienić nazwę kolumny, możesz użyć metody `renameColumn` dostarczonej przez constructor schematów:

```php
Schema::table('users', function (Blueprint $table) {
    $table->renameColumn('from', 'to');
});
```

<a name="dropping-columns"></a>
### Usuwanie Kolumn

Aby usunąć kolumnę, możesz użyć metody `dropColumn` na constructorze schematów:

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('votes');
});
```

Możesz usunąć wiele kolumn z tabeli, przekazując tablicę nazw kolumn do metody `dropColumn`:

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```

<a name="available-command-aliases"></a>
#### Dostępne Aliasy Poleceń

Laravel dostarcza kilka wygodnych metod związanych z usuwaniem popularnych typów kolumn. Każda z tych metod jest opisana w poniższej tabeli:

<div class="overflow-auto">

| Polecenie                               | Opis                                                          |
| --------------------------------------- | ------------------------------------------------------------- |
| `$table->dropMorphs('morphable');`  | Usuń kolumny `morphable_id` i `morphable_type`. |
| `$table->dropRememberToken();`      | Usuń kolumnę `remember_token`.                     |
| `$table->dropSoftDeletes();`        | Usuń kolumnę `deleted_at`.                         |
| `$table->dropSoftDeletesTz();`      | Alias metody `dropSoftDeletes()`.                  |
| `$table->dropTimestamps();`         | Usuń kolumny `created_at` i `updated_at`.       |
| `$table->dropTimestampsTz();`       | Alias metody `dropTimestamps()`.                   |

</div>

<a name="indexes"></a>
## Indeksy

<a name="creating-indexes"></a>
### Tworzenie Indeksów

Constructor schematów Laravel obsługuje kilka typów indeksów. Poniższy przykład tworzy nową kolumnę `email` i określa, że jej wartości powinny być unikalne. Aby utworzyć indeks, możemy użyć metody `unique` w łańcuchu definicji kolumny:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->string('email')->unique();
});
```

Alternatywnie możesz utworzyć indeks po zdefiniowaniu kolumny. Aby to zrobić, powinieneś wywołać metodę `unique` na blueprint constructora schematów. Ta metoda akceptuje nazwę kolumny, która powinna otrzymać unikalny indeks:

```php
$table->unique('email');
```

Możesz nawet przekazać tablicę kolumn do metody indeksu, aby utworzyć indeks złożony (lub kompozytowy):

```php
$table->index(['account_id', 'created_at']);
```

Podczas tworzenia indeksu Laravel automatycznie wygeneruje nazwę indeksu na podstawie nazwy tabeli, nazw kolumn i typu indeksu, ale możesz przekazać drugi argument do metody, aby określić nazwę indeksu samodzielnie:

```php
$table->unique('email', 'unique_email');
```

<a name="available-index-types"></a>
#### Dostępne Typy Indeksów

Klasa blueprint constructora schematów Laravel zapewnia metody do tworzenia każdego typu indeksu obsługiwanego przez Laravel. Każda metoda indeksu akceptuje opcjonalny drugi argument określający nazwę indeksu. Jeśli zostanie pominięty, nazwa będzie pochodzić od nazw tabeli i kolumn używanych dla indeksu, a także typu indeksu. Każda z dostępnych metod indeksu jest opisana w poniższej tabeli:

<div class="overflow-auto">

| Polecenie                                            | Opis                                                           |
| ---------------------------------------------------- | -------------------------------------------------------------- |
| `$table->primary('id');`                         | Dodaje klucz główny.                                            |
| `$table->primary(['id', 'parent_id']);`          | Dodaje klucze kompozytowe.                                           |
| `$table->unique('email');`                       | Dodaje unikalny indeks.                                           |
| `$table->index('state');`                        | Dodaje indeks.                                                 |
| `$table->fullText('body');`                      | Dodaje indeks pełnotekstowy (MariaDB / MySQL / PostgreSQL).         |
| `$table->fullText('body')->language('english');` | Dodaje indeks pełnotekstowy określonego języka (PostgreSQL). |
| `$table->spatialIndex('location');`              | Dodaje indeks przestrzenny (z wyjątkiem SQLite).                          |

</div>

<a name="online-index-creation"></a>
#### Tworzenie Indeksów Online

Domyślnie tworzenie indeksu na dużej tabeli może zablokować tabelę i blokować odczyty lub zapisy podczas budowania indeksu. Podczas używania PostgreSQL lub SQL Server możesz użyć metody `online` w łańcuchu definicji indeksu, aby utworzyć indeks bez blokowania tabeli, pozwalając aplikacji kontynuować odczyt i zapis danych podczas tworzenia indeksu:

```php
$table->string('email')->unique()->online();
```

Podczas używania PostgreSQL dodaje to opcję `CONCURRENTLY` do instrukcji tworzenia indeksu. Podczas używania SQL Server dodaje to opcję `WITH (online = on)`.

<a name="renaming-indexes"></a>
### Zmiana Nazw Indeksów

Aby zmienić nazwę indeksu, możesz użyć metody `renameIndex` dostarczonej przez blueprint constructora schematów. Ta metoda akceptuje bieżącą nazwę indeksu jako pierwszy argument i żądaną nazwę jako drugi argument:

```php
$table->renameIndex('from', 'to')
```

<a name="dropping-indexes"></a>
### Usuwanie Indeksów

Aby usunąć indeks, musisz określić nazwę indeksu. Domyślnie Laravel automatycznie przypisuje nazwę indeksu na podstawie nazwy tabeli, nazwy indeksowanej kolumny i typu indeksu. Oto kilka przykładów:

<div class="overflow-auto">

| Polecenie                                                  | Opis                                                 |
| ---------------------------------------------------------- | ---------------------------------------------------- |
| `$table->dropPrimary('users_id_primary');`               | Usuń klucz główny z tabeli "users".                  |
| `$table->dropUnique('users_email_unique');`              | Usuń unikalny indeks z tabeli "users".                 |
| `$table->dropIndex('geo_state_index');`                  | Usuń podstawowy indeks z tabeli "geo".                    |
| `$table->dropFullText('posts_body_fulltext');`           | Usuń indeks pełnotekstowy z tabeli "posts".              |
| `$table->dropSpatialIndex('geo_location_spatialindex');` | Usuń indeks przestrzenny z tabeli "geo" (z wyjątkiem SQLite). |

</div>

Jeśli przekazujesz tablicę kolumn do metody usuwającej indeksy, konwencyjna nazwa indeksu zostanie wygenerowana na podstawie nazwy tabeli, kolumn i typu indeksu:

```php
Schema::table('geo', function (Blueprint $table) {
    $table->dropIndex(['state']); // Drops index 'geo_state_index'
});
```

<a name="foreign-key-constraints"></a>
### Ograniczenia Klucza Obcego

Laravel zapewnia również obsługę tworzenia ograniczeń klucza obcego, które są używane do wymuszania integralności referencyjnej na poziomie bazy danych. Na przykład zdefiniujmy kolumnę `user_id` w tabeli `posts`, która odnosi się do kolumny `id` w tabeli `users`:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('posts', function (Blueprint $table) {
    $table->unsignedBigInteger('user_id');

    $table->foreign('user_id')->references('id')->on('users');
});
```

Ponieważ ta składnia jest dość rozwlekła, Laravel zapewnia dodatkowe, krótsze metody, które używają konwencji, aby zapewnić lepszą jakość pracy programisty. Podczas używania metody `foreignId` do tworzenia kolumny, powyższy przykład może być przepisany tak:

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained();
});
```

Metoda `foreignId` tworzy kolumnę równoważną `UNSIGNED BIGINT`, podczas gdy metoda `constrained` użyje konwencji do określenia tabeli i kolumny, do której następuje odniesienie. Jeśli nazwa tabeli nie pasuje do konwencji Laravel, możesz ręcznie przekazać ją do metody `constrained`. Ponadto można również określić nazwę, która powinna zostać przypisana do wygenerowanego indeksu:

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained(
        table: 'users', indexName: 'posts_user_id'
    );
});
```

Możesz również określić żądaną akcję dla właściwości "on delete" i "on update" ograniczenia:

```php
$table->foreignId('user_id')
    ->constrained()
    ->onUpdate('cascade')
    ->onDelete('cascade');
```

Dostępna jest również alternatywna, ekspresyjna składnia dla tych akcji:

<div class="overflow-auto">

| Metoda                        | Opis                                       |
| ----------------------------- | ------------------------------------------ |
| `$table->cascadeOnUpdate();`  | Aktualizacje powinny kaskadowo się rozprzestrzeniać.                           |
| `$table->restrictOnUpdate();` | Aktualizacje powinny być ograniczone.                     |
| `$table->nullOnUpdate();`     | Aktualizacje powinny ustawić wartość klucza obcego na null. |
| `$table->noActionOnUpdate();` | Brak akcji przy aktualizacjach.                             |
| `$table->cascadeOnDelete();`  | Usunięcia powinny kaskadowo się rozprzestrzeniać.                           |
| `$table->restrictOnDelete();` | Usunięcia powinny być ograniczone.                     |
| `$table->nullOnDelete();`     | Usunięcia powinny ustawić wartość klucza obcego na null. |
| `$table->noActionOnDelete();` | Zapobiega usunięciom, jeśli istnieją rekordy potomne.          |

</div>

Wszelkie dodatkowe [modyfikatory kolumn](#column-modifiers) muszą zostać wywołane przed metodą `constrained`:

```php
$table->foreignId('user_id')
    ->nullable()
    ->constrained();
```

<a name="dropping-foreign-keys"></a>
#### Dropping Foreign Keys

To drop a foreign key, you may use the `dropForeign` method, passing the name of the foreign key constraint to be deleted as an argument. Foreign key constraints use the same naming convention as indexes. In other words, the foreign key constraint name is based on the name of the table and the columns in the constraint, followed by a "\_foreign" suffix:

```php
$table->dropForeign('posts_user_id_foreign');
```

Alternatywnie możesz przekazać tablicę zawierającą nazwę kolumny zawierającej klucz obcy do metody `dropForeign`. Tablica zostanie przekonwertowana na nazwę ograniczenia klucza obcego używając konwencji nazewnictwa Laravel:

```php
$table->dropForeign(['user_id']);
```

<a name="toggling-foreign-key-constraints"></a>
#### Przełączanie Ograniczeń Klucza Obcego

Możesz włączyć lub wyłączyć ograniczenia klucza obcego w swoich migracjach za pomocą następujących metod:

```php
Schema::enableForeignKeyConstraints();

Schema::disableForeignKeyConstraints();

Schema::withoutForeignKeyConstraints(function () {
    // Constraints disabled within this closure...
});
```

> [!WARNING]
> SQLite domyślnie wyłącza ograniczenia klucza obcego. Podczas używania SQLite upewnij się, że [włączyłeś obsługę klucza obcego](/docs/{{version}}/database#configuration) w konfiguracji bazy danych przed próbą utworzenia ich w migracjach.

<a name="events"></a>
## Zdarzenia

Dla wygody każda operacja migracji wysyła [zdarzenie](/docs/{{version}}/events). Wszystkie poniższe zdarzenia rozszerzają bazowoklasy `Illuminate\Database\Events\MigrationEvent`:

<div class="overflow-auto">

| Klasa                                            | Opis                                      |
| ------------------------------------------------ | ----------------------------------------- |
| `Illuminate\Database\Events\MigrationsStarted`   | Partia migracji zostanie wykonana.   |
| `Illuminate\Database\Events\MigrationsEnded`     | Partia migracji zakończyła się.    |
| `Illuminate\Database\Events\MigrationStarted`    | Pojedyncza migracja zostanie wykonana.      |
| `Illuminate\Database\Events\MigrationEnded`      | Pojedyncza migracja zakończyła się.       |
| `Illuminate\Database\Events\NoPendingMigrations` | Polecenie migracji nie znalazło oczekujących migracji. |
| `Illuminate\Database\Events\SchemaDumped`        | Zrzut schematu bazy danych został zakończony.            |
| `Illuminate\Database\Events\SchemaLoaded`        | Istniejący zrzut schematu bazy danych został załadowany.     |

</div>
