# Baza danych: Wprowadzenie

- [Wprowadzenie](#introduction)
    - [Konfiguracja](#configuration)
    - [Połączenia odczytu i zapisu](#read-and-write-connections)
- [Wykonywanie zapytań SQL](#running-queries)
    - [Używanie wielu połączeń z bazą danych](#using-multiple-database-connections)
    - [Nasłuchiwanie zdarzeń zapytań](#listening-for-query-events)
    - [Monitorowanie skumulowanego czasu zapytań](#monitoring-cumulative-query-time)
- [Transakcje bazodanowe](#database-transactions)
- [Łączenie z CLI bazy danych](#connecting-to-the-database-cli)
- [Inspekcja baz danych](#inspecting-your-databases)
- [Monitorowanie baz danych](#monitoring-your-databases)

<a name="introduction"></a>
## Wprowadzenie

Prawie każda nowoczesna aplikacja webowa współpracuje z bazą danych. Laravel sprawia, że interakcja z bazami danych jest niezwykle prosta w przypadku różnych obsługiwanych baz danych, używając surowego SQL, [płynnego kreatora zapytań](/docs/{{version}}/queries) oraz [Eloquent ORM](/docs/{{version}}/eloquent). Obecnie Laravel zapewnia natywne wsparcie dla pięciu baz danych:

<div class="content-list" markdown="1">

- MariaDB 10.3+ ([Version Policy](https://mariadb.org/about/#maintenance-policy))
- MySQL 5.7+ ([Version Policy](https://en.wikipedia.org/wiki/MySQL#Release_history))
- PostgreSQL 10.0+ ([Version Policy](https://www.postgresql.org/support/versioning/))
- SQLite 3.26.0+
- SQL Server 2017+ ([Version Policy](https://docs.microsoft.com/en-us/lifecycle/products/?products=sql-server))

</div>

Dodatkowo MongoDB jest obsługiwany przez pakiet `mongodb/laravel-mongodb`, który jest oficjalnie utrzymywany przez MongoDB. Sprawdź dokumentację [Laravel MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/) po więcej informacji.

<a name="configuration"></a>
### Konfiguracja

Konfiguracja usług bazy danych Laravel znajduje się w pliku konfiguracyjnym `config/database.php` Twojej aplikacji. W tym pliku możesz zdefiniować wszystkie swoje połączenia z bazą danych, a także określić, które połączenie powinno być używane domyślnie. Większość opcji konfiguracji w tym pliku jest sterowana przez wartości zmiennych środowiskowych Twojej aplikacji. W tym pliku znajdują się przykłady dla większości systemów baz danych obsługiwanych przez Laravel.

Domyślnie przykładowa [konfiguracja środowiskowa](/docs/{{version}}/configuration#environment-configuration) Laravel jest gotowa do użycia z [Laravel Sail](/docs/{{version}}/sail), który jest konfiguracją Docker do tworzenia aplikacji Laravel na lokalnej maszynie. Możesz jednak dowolnie modyfikować konfigurację swojej bazy danych zgodnie z potrzebami Twojej lokalnej bazy danych.

<a name="sqlite-configuration"></a>
#### Konfiguracja SQLite

Bazy danych SQLite są zawarte w pojedynczym pliku w Twoim systemie plików. Możesz utworzyć nową bazę danych SQLite używając polecenia `touch` w terminalu: `touch database/database.sqlite`. Po utworzeniu bazy danych możesz łatwo skonfigurować swoje zmienne środowiskowe, aby wskazywały na tę bazę danych, umieszczając bezwzględną ścieżkę do bazy danych w zmiennej środowiskowej `DB_DATABASE`:

```ini
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
```

Domyślnie ograniczenia klucza obcego są włączone dla połączeń SQLite. Jeśli chcesz je wyłączyć, powinieneś ustawić zmienną środowiskową `DB_FOREIGN_KEYS` na `false`:

```ini
DB_FOREIGN_KEYS=false
```

> [!NOTE]
> Jeśli używasz [instalatora Laravel](/docs/{{version}}/installation#creating-a-laravel-project) do utworzenia swojej aplikacji Laravel i wybierzesz SQLite jako swoją bazę danych, Laravel automatycznie utworzy plik `database/database.sqlite` i uruchomi domyślne [migracje bazy danych](/docs/{{version}}/migrations) za Ciebie.

<a name="mssql-configuration"></a>
#### Konfiguracja Microsoft SQL Server

Aby użyć bazy danych Microsoft SQL Server, powinieneś upewnić się, że masz zainstalowane rozszerzenia PHP `sqlsrv` i `pdo_sqlsrv`, a także wszelkie zależności, których mogą wymagać, takie jak sterownik Microsoft SQL ODBC.

<a name="configuration-using-urls"></a>
#### Konfiguracja przy użyciu adresów URL

Zazwyczaj połączenia z bazą danych są konfigurowane przy użyciu wielu wartości konfiguracji, takich jak `host`, `database`, `username`, `password` itp. Każda z tych wartości konfiguracji ma swoją odpowiednią zmienną środowiskową. Oznacza to, że podczas konfigurowania informacji o połączeniu z bazą danych na serwerze produkcyjnym musisz zarządzać kilkoma zmiennymi środowiskowymi.

Niektórzy zarządzani dostawcy baz danych, tacy jak AWS i Heroku, udostępniają pojedynczy "URL" bazy danych, który zawiera wszystkie informacje o połączeniu dla bazy danych w jednym ciągu. Przykładowy adres URL bazy danych może wyglądać następująco:

```html
mysql://root:password@127.0.0.1/forge?charset=UTF-8
```

Te adresy URL zazwyczaj podlegają standardowej konwencji schematu:

```html
driver://username:password@host:port/database?options
```

Dla wygody Laravel obsługuje te adresy URL jako alternatywę dla konfigurowania bazy danych za pomocą wielu opcji konfiguracji. Jeśli opcja konfiguracyjna `url` (lub odpowiadająca jej zmienna środowiskowa `DB_URL`) jest obecna, zostanie użyta do wyodrębnienia informacji o połączeniu z bazą danych i poświadczeń.

<a name="read-and-write-connections"></a>
### Połączenia odczytu i zapisu

Czasami możesz chcieć użyć jednego połączenia z bazą danych dla instrukcji SELECT, a innego dla instrukcji INSERT, UPDATE i DELETE. Laravel sprawia, że jest to dziecinnie proste, a odpowiednie połączenia będą zawsze używane, niezależnie od tego, czy używasz surowych zapytań, kreatora zapytań czy Eloquent ORM.

Aby zobaczyć, jak należy skonfigurować połączenia odczytu/zapisu, spójrzmy na ten przykład:

```php
'mysql' => [
    'driver' => 'mysql',
    
    'read' => [
        'host' => [
            '192.168.1.1',
            '196.168.1.2',
        ],
    ],
    'write' => [
        'host' => [
            '192.168.1.3',
        ],
    ],
    'sticky' => true,
    
    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'unix_socket' => env('DB_SOCKET', ''),
    'charset' => env('DB_CHARSET', 'utf8mb4'),
    'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
    'prefix' => '',
    'prefix_indexes' => true,
    'strict' => true,
    'engine' => null,
    'options' => extension_loaded('pdo_mysql') ? array_filter([
        (PHP_VERSION_ID >= 80500 ? \Pdo\Mysql::ATTR_SSL_CA : \PDO::MYSQL_ATTR_SSL_CA) => env('MYSQL_ATTR_SSL_CA'),
    ]) : [],
],
```

Zauważ, że trzy klucze zostały dodane do tablicy konfiguracji: `read`, `write` i `sticky`. Klucze `read` i `write` mają wartości tablicowe zawierające pojedynczy klucz: `host`. Pozostałe opcje bazy danych dla połączeń `read` i `write` zostaną scalone z głównej tablicy konfiguracji `mysql`.

Musisz umieścić elementy w tablicach `read` i `write` tylko wtedy, gdy chcesz nadpisać wartości z głównej tablicy `mysql`. Więc w tym przypadku `192.168.1.1` będzie używany jako host dla połączenia "read", podczas gdy `192.168.1.3` będzie używany dla połączenia "write". Poświadczenia bazy danych, prefiks, zestaw znaków i wszystkie inne opcje w głównej tablicy `mysql` będą współdzielone przez oba połączenia. Gdy w tablicy konfiguracji `host` istnieje wiele wartości, host bazy danych będzie losowo wybierany dla każdego żądania.

<a name="the-sticky-option"></a>
#### Opcja `sticky`

Opcja `sticky` jest *opcjonalną* wartością, która może być użyta do umożliwienia natychmiastowego odczytu rekordów, które zostały zapisane w bazie danych podczas bieżącego cyklu żądania. Jeśli opcja `sticky` jest włączona i podczas bieżącego cyklu żądania wykonana została operacja "write" w bazie danych, wszelkie dalsze operacje "read" będą używać połączenia "write". Zapewnia to, że wszelkie dane zapisane podczas cyklu żądania mogą być natychmiast odczytane z bazy danych podczas tego samego żądania. To od Ciebie zależy, czy jest to pożądane zachowanie dla Twojej aplikacji.

<a name="running-queries"></a>
## Wykonywanie zapytań SQL

Po skonfigurowaniu połączenia z bazą danych możesz uruchamiać zapytania przy użyciu fasady `DB`. Fasada `DB` dostarcza metody dla każdego typu zapytania: `select`, `update`, `insert`, `delete` i `statement`.

<a name="running-a-select-query"></a>
#### Wykonywanie zapytania Select

Aby uruchomić podstawowe zapytanie SELECT, możesz użyć metody `select` na fasadzie `DB`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Pokaż listę wszystkich użytkowników aplikacji.
     */
    public function index(): View
    {
        $users = DB::select('select * from users where active = ?', [1]);

        return view('user.index', ['users' => $users]);
    }
}
```

Pierwszy argument przekazany do metody `select` to zapytanie SQL, podczas gdy drugi argument to wszelkie powiązania parametrów, które muszą być powiązane z zapytaniem. Zazwyczaj są to wartości ograniczeń klauzuli `where`. Wiązanie parametrów zapewnia ochronę przed atakami typu SQL injection.

Metoda `select` zawsze zwróci `array` wyników. Każdy wynik w tablicy będzie obiektem PHP `stdClass` reprezentującym rekord z bazy danych:

```php
use Illuminate\Support\Facades\DB;

$users = DB::select('select * from users');

foreach ($users as $user) {
    echo $user->name;
}
```

<a name="selecting-scalar-values"></a>
#### Wybieranie wartości skalarnych

Czasami Twoje zapytanie do bazy danych może zwrócić pojedynczą wartość skalarną. Zamiast być zmuszonym do pobierania skalarnego wyniku zapytania z obiektu rekordu, Laravel pozwala Ci pobrać tę wartość bezpośrednio, używając metody `scalar`:

```php
$burgers = DB::scalar(
    "select count(case when food = 'burger' then 1 end) as burgers from menu"
);
```

<a name="selecting-multiple-result-sets"></a>
#### Wybieranie wielu zestawów wyników

Jeśli Twoja aplikacja wywołuje procedury składowane, które zwracają wiele zestawów wyników, możesz użyć metody `selectResultSets`, aby pobrać wszystkie zestawy wyników zwrócone przez procedurę składowaną:

```php
[$options, $notifications] = DB::selectResultSets(
    "CALL get_user_options_and_notifications(?)", $request->user()->id
);
```

<a name="using-named-bindings"></a>
#### Używanie nazwanych powiązań

Zamiast używać `?` do reprezentowania wiązań parametrów, możesz wykonać zapytanie używając nazwanych powiązań:

```php
$results = DB::select('select * from users where id = :id', ['id' => 1]);
```

<a name="running-an-insert-statement"></a>
#### Wykonywanie instrukcji Insert

Aby wykonać instrukcję `insert`, możesz użyć metody `insert` na fasadzie `DB`. Podobnie jak `select`, ta metoda przyjmuje zapytanie SQL jako pierwszy argument i powiązania jako drugi argument:

```php
use Illuminate\Support\Facades\DB;

DB::insert('insert into users (id, name) values (?, ?)', [1, 'Marc']);
```

<a name="running-an-update-statement"></a>
#### Wykonywanie instrukcji Update

Metoda `update` powinna być używana do aktualizacji istniejących rekordów w bazie danych. Liczba wierszy dotkniętych przez instrukcję jest zwracana przez metodę:

```php
use Illuminate\Support\Facades\DB;

$affected = DB::update(
    'update users set votes = 100 where name = ?',
    ['Anita']
);
```

<a name="running-a-delete-statement"></a>
#### Wykonywanie instrukcji Delete

Metoda `delete` powinna być używana do usuwania rekordów z bazy danych. Podobnie jak `update`, liczba dotkniętych wierszy zostanie zwrócona przez metodę:

```php
use Illuminate\Support\Facades\DB;

$deleted = DB::delete('delete from users');
```

<a name="running-a-general-statement"></a>
#### Wykonywanie instrukcji ogólnej

Niektóre instrukcje bazy danych nie zwracają żadnej wartości. Dla tego typu operacji możesz użyć metody `statement` na fasadzie `DB`:

```php
DB::statement('drop table users');
```

<a name="running-an-unprepared-statement"></a>
#### Wykonywanie nieprzygotowanej instrukcji

Czasami możesz chcieć wykonać instrukcję SQL bez wiązania żadnych wartości. Możesz użyć metody `unprepared` fasady `DB`, aby to osiągnąć:

```php
DB::unprepared('update users set votes = 100 where name = "Dries"');
```

> [!WARNING]
> Ponieważ nieprzygotowane instrukcje nie wiążą parametrów, mogą być podatne na ataki SQL injection. Nigdy nie powinieneś zezwalać na wartości kontrolowane przez użytkownika w nieprzygotowanej instrukcji.

<a name="implicit-commits-in-transactions"></a>
#### Niejawne zatwierdzenia w transakcjach

Podczas używania metod `statement` i `unprepared` fasady `DB` w transakcjach musisz uważać, aby unikać instrukcji, które powodują [niejawne zatwierdzenia](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html). Te instrukcje spowodują, że silnik bazy danych pośrednio zatwierdzi całą transakcję, pozostawiając Laravel nieświadomy poziomu transakcji bazy danych. Przykładem takiej instrukcji jest utworzenie tabeli bazy danych:

```php
DB::unprepared('create table a (col varchar(1) null)');
```

Sprawdź w podręczniku MySQL [listę wszystkich instrukcji](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html), które wyzwalają niejawne zatwierdzenia.

<a name="using-multiple-database-connections"></a>
### Używanie wielu połączeń z bazą danych

Jeśli Twoja aplikacja definiuje wiele połączeń w pliku konfiguracyjnym `config/database.php`, możesz uzyskać dostęp do każdego połączenia za pomocą metody `connection` dostarczonej przez fasadę `DB`. Nazwa połączenia przekazana do metody `connection` powinna odpowiadać jednemu z połączeń wymienionych w pliku konfiguracyjnym `config/database.php` lub skonfigurowanemu w czasie wykonywania za pomocą helpera `config`:

```php
use Illuminate\Support\Facades\DB;

$users = DB::connection('sqlite')->select(/* ... */);
```

Możesz uzyskać dostęp do surowej, podstawowej instancji PDO połączenia, używając metody `getPdo` na instancji połączenia:

```php
$pdo = DB::connection()->getPdo();
```

<a name="listening-for-query-events"></a>
### Nasłuchiwanie zdarzeń zapytań

Jeśli chcesz określić zamknięcie, które jest wywoływane dla każdego zapytania SQL wykonanego przez Twoją aplikację, możesz użyć metody `listen` fasady `DB`. Ta metoda może być przydatna do rejestrowania zapytań lub debugowania. Możesz zarejestrować swoje zamknięcie nasłuchujące zapytania w metodzie `boot` [dostawcy usług](/docs/{{version}}/providers):

```php
<?php

namespace App\Providers;

use Illuminate\Database\Events\QueryExecuted;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Zarejestruj wszelkie usługi aplikacji.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Uruchom wszelkie usługi aplikacji.
     */
    public function boot(): void
    {
        DB::listen(function (QueryExecuted $query) {
            // $query->sql;
            // $query->bindings;
            // $query->time;
            // $query->toRawSql();
        });
    }
}
```

<a name="monitoring-cumulative-query-time"></a>
### Monitorowanie skumulowanego czasu zapytań

Częstym wąskim gardłem wydajności nowoczesnych aplikacji webowych jest czas, który spędzają na odpytywaniu baz danych. Na szczęście Laravel może wywołać zamknięcie lub callback Twojego wyboru, gdy spędza zbyt dużo czasu na odpytywaniu bazy danych podczas pojedynczego żądania. Aby rozpocząć, podaj próg czasu zapytania (w milisekundach) i zamknięcie do metody `whenQueryingForLongerThan`. Możesz wywołać tę metodę w metodzie `boot` [dostawcy usług](/docs/{{version}}/providers):

```php
<?php

namespace App\Providers;

use Illuminate\Database\Connection;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;
use Illuminate\Database\Events\QueryExecuted;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Zarejestruj wszelkie usługi aplikacji.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Uruchom wszelkie usługi aplikacji.
     */
    public function boot(): void
    {
        DB::whenQueryingForLongerThan(500, function (Connection $connection, QueryExecuted $event) {
            // Powiadom zespół deweloperski...
        });
    }
}
```

<a name="database-transactions"></a>
## Transakcje bazodanowe

Możesz użyć metody `transaction` dostarczonej przez fasadę `DB`, aby uruchomić zestaw operacji w ramach transakcji bazodanowej. Jeśli w zamknięciu transakcji zostanie zgłoszony wyjątek, transakcja zostanie automatycznie wycofana, a wyjątek zostanie ponownie zgłoszony. Jeśli zamknięcie zostanie wykonane pomyślnie, transakcja zostanie automatycznie zatwierdzona. Nie musisz martwić się o ręczne wycofywanie lub zatwierdzanie podczas korzystania z metody `transaction`:

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::update('update users set votes = 1');

    DB::delete('delete from posts');
});
```

<a name="handling-deadlocks"></a>
#### Obsługa zakleszczeń

Metoda `transaction` przyjmuje opcjonalny drugi argument, który definiuje, ile razy transakcja powinna być ponawiana, gdy wystąpi zakleszczenie. Po wyczerpaniu tych prób zostanie zgłoszony wyjątek:

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::update('update users set votes = 1');

    DB::delete('delete from posts');
}, attempts: 5);
```

<a name="manually-using-transactions"></a>
#### Ręczne używanie transakcji

Jeśli chcesz rozpocząć transakcję ręcznie i mieć pełną kontrolę nad wycofywaniem i zatwierdzaniem, możesz użyć metody `beginTransaction` dostarczonej przez fasadę `DB`:

```php
use Illuminate\Support\Facades\DB;

DB::beginTransaction();
```

Możesz wycofać transakcję za pomocą metody `rollBack`:

```php
DB::rollBack();
```

Na koniec możesz zatwierdzić transakcję za pomocą metody `commit`:

```php
DB::commit();
```

> [!NOTE]
> Metody transakcji fasady `DB` kontrolują transakcje zarówno dla [kreatora zapytań](/docs/{{version}}/queries), jak i [Eloquent ORM](/docs/{{version}}/eloquent).

<a name="connecting-to-the-database-cli"></a>
## Łączenie z CLI bazy danych

Jeśli chcesz połączyć się z CLI swojej bazy danych, możesz użyć polecenia Artisan `db`:

```shell
php artisan db
```

W razie potrzeby możesz określić nazwę połączenia z bazą danych, aby połączyć się z połączeniem z bazą danych, które nie jest połączeniem domyślnym:

```shell
php artisan db mysql
```

<a name="inspecting-your-databases"></a>
## Inspekcja baz danych

Za pomocą poleceń Artisan `db:show` i `db:table` możesz uzyskać cenny wgląd w swoją bazę danych i powiązane z nią tabele. Aby zobaczyć przegląd swojej bazy danych, w tym jej rozmiar, typ, liczbę otwartych połączeń i podsumowanie jej tabel, możesz użyć polecenia `db:show`:

```shell
php artisan db:show
```

Możesz określić, które połączenie z bazą danych powinno być sprawdzone, podając nazwę połączenia z bazą danych do polecenia za pomocą opcji `--database`:

```shell
php artisan db:show --database=pgsql
```

Jeśli chcesz uwzględnić liczbę wierszy tabeli i szczegóły widoków bazy danych w danych wyjściowych polecenia, możesz podać odpowiednio opcje `--counts` i `--views`. W dużych bazach danych pobieranie liczby wierszy i szczegółów widoków może być wolne:

```shell
php artisan db:show --counts --views
```

Ponadto możesz użyć następujących metod `Schema` do inspekcji swojej bazy danych:

```php
use Illuminate\Support\Facades\Schema;

$tables = Schema::getTables();
$views = Schema::getViews();
$columns = Schema::getColumns('users');
$indexes = Schema::getIndexes('users');
$foreignKeys = Schema::getForeignKeys('users');
```

Jeśli chcesz sprawdzić połączenie z bazą danych, które nie jest domyślnym połączeniem Twojej aplikacji, możesz użyć metody `connection`:

```php
$columns = Schema::connection('sqlite')->getColumns('users');
```

<a name="table-overview"></a>
#### Przegląd tabeli

Jeśli chcesz uzyskać przegląd pojedynczej tabeli w swojej bazie danych, możesz wykonać polecenie Artisan `db:table`. To polecenie zapewnia ogólny przegląd tabeli bazy danych, w tym jej kolumny, typy, atrybuty, klucze i indeksy:

```shell
php artisan db:table users
```

<a name="monitoring-your-databases"></a>
## Monitorowanie baz danych

Za pomocą polecenia Artisan `db:monitor` możesz nakazać Laravel wysłanie zdarzenia `Illuminate\Database\Events\DatabaseBusy`, jeśli Twoja baza danych zarządza więcej niż określoną liczbą otwartych połączeń.

Aby rozpocząć, powinieneś zaplanować uruchamianie polecenia `db:monitor` [co minutę](/docs/{{version}}/scheduling). Polecenie przyjmuje nazwy konfiguracji połączeń z bazą danych, które chcesz monitorować, a także maksymalną liczbę otwartych połączeń, która powinna być tolerowana przed wysłaniem zdarzenia:

```shell
php artisan db:monitor --databases=mysql,pgsql --max=100
```

Samo zaplanowanie tego polecenia nie wystarcza, aby wywołać powiadomienie informujące o liczbie otwartych połączeń. Gdy polecenie napotka bazę danych, która ma liczbę otwartych połączeń przekraczającą Twój próg, zostanie wysłane zdarzenie `DatabaseBusy`. Powinieneś nasłuchiwać tego zdarzenia w `AppServiceProvider` swojej aplikacji, aby wysłać powiadomienie do Ciebie lub Twojego zespołu deweloperskiego:

```php
use App\Notifications\DatabaseApproachingMaxConnections;
use Illuminate\Database\Events\DatabaseBusy;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Notification;

/**
 * Uruchom wszelkie usługi aplikacji.
 */
public function boot(): void
{
    Event::listen(function (DatabaseBusy $event) {
        Notification::route('mail', 'dev@example.com')
            ->notify(new DatabaseApproachingMaxConnections(
                $event->connectionName,
                $event->connections
            ));
    });
}
```
