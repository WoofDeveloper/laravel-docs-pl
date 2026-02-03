# Baza danych: Konstruktor zapytań

- [Wprowadzenie](#introduction)
- [Uruchamianie zapytań do bazy danych](#running-database-queries)
    - [Porcjowanie wyników](#chunking-results)
    - [Leniwe strumieniowanie wyników](#streaming-results-lazily)
    - [Agregaty](#aggregates)
- [Instrukcje Select](#select-statements)
- [Wyrażenia surowe](#raw-expressions)
- [Złączenia](#joins)
- [Unie](#unions)
- [Podstawowe klauzule Where](#basic-where-clauses)
    - [Klauzule Where](#where-clauses)
    - [Klauzule Or Where](#or-where-clauses)
    - [Klauzule Where Not](#where-not-clauses)
    - [Klauzule Where Any / All / None](#where-any-all-none-clauses)
    - [Klauzule JSON Where](#json-where-clauses)
    - [Dodatkowe klauzule Where](#additional-where-clauses)
    - [Grupowanie logiczne](#logical-grouping)
- [Zaawansowane klauzule Where](#advanced-where-clauses)
    - [Klauzule Where Exists](#where-exists-clauses)
    - [Podzapytania Where](#subquery-where-clauses)
    - [Klauzule pełnotekstowe Where](#full-text-where-clauses)
- [Sortowanie, grupowanie, limit i offset](#ordering-grouping-limit-and-offset)
    - [Sortowanie](#ordering)
    - [Grupowanie](#grouping)
    - [Limit i Offset](#limit-and-offset)
- [Klauzule warunkowe](#conditional-clauses)
- [Instrukcje Insert](#insert-statements)
    - [Upserty](#upserts)
- [Instrukcje Update](#update-statements)
    - [Aktualizowanie kolumn JSON](#updating-json-columns)
    - [Inkrementacja i dekrementacja](#increment-and-decrement)
- [Instrukcje Delete](#delete-statements)
- [Blokowanie pesymistyczne](#pessimistic-locking)
- [Komponenty zapytań wielokrotnego użytku](#reusable-query-components)
- [Debugowanie](#debugging)

<a name="introduction"></a>
## Wprowadzenie

Konstruktor zapytań do bazy danych w Laravel zapewnia wygodny, płynny interfejs do tworzenia i wykonywania zapytań do bazy danych. Może być używany do wykonywania większości operacji na bazie danych w Twojej aplikacji i działa doskonale ze wszystkimi obsługiwanymi przez Laravel systemami baz danych.

Konstruktor zapytań Laravel używa wiązania parametrów PDO, aby chronić Twoją aplikację przed atakami SQL injection. Nie ma potrzeby czyszczenia ani sanityzacji ciągów przekazywanych do konstruktora zapytań jako wiązań.

> [!WARNING]
> PDO nie obsługuje wiązania nazw kolumn. Dlatego nigdy nie należy pozwalać na to, aby dane wejściowe użytkownika dyktowały nazwy kolumn, do których odwołują się Twoje zapytania, w tym kolumny "order by".

<a name="running-database-queries"></a>
## Uruchamianie zapytań do bazy danych

<a name="retrieving-all-rows-from-a-table"></a>
#### Pobieranie wszystkich wierszy z tabeli

Możesz użyć metody `table` dostarczonej przez fasadę `DB`, aby rozpocząć zapytanie. Metoda `table` zwraca instancję płynnego konstruktora zapytań dla danej tabeli, umożliwiając łączenie większej liczby ograniczeń do zapytania, a następnie ostatecznie pobranie wyników zapytania za pomocą metody `get`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show a list of all of the application's users.
     */
    public function index(): View
    {
        $users = DB::table('users')->get();

        return view('user.index', ['users' => $users]);
    }
}
```

Metoda `get` zwraca instancję `Illuminate\Support\Collection` zawierającą wyniki zapytania, gdzie każdy wynik jest instancją klasy PHP `stdClass`. Możesz uzyskać dostęp do wartości każdej kolumny, uzyskując dostęp do kolumny jako właściwości obiektu:

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')->get();

foreach ($users as $user) {
    echo $user->name;
}
```

> [!NOTE]
> Kolekcje Laravel zapewniają wiele niezwykle potężnych metod mapowania i redukcji danych. Aby uzyskać więcej informacji o kolekcjach Laravel, sprawdź [dokumentację kolekcji](/docs/{{version}}/collections).

<a name="retrieving-a-single-row-column-from-a-table"></a>
#### Pobieranie pojedynczego wiersza / kolumny z tabeli

Jeśli potrzebujesz pobrać tylko jeden wiersz z tabeli bazy danych, możesz użyć metody `first` fasady `DB`. Ta metoda zwróci pojedynczy obiekt `stdClass`:

```php
$user = DB::table('users')->where('name', 'John')->first();

return $user->email;
```

Jeśli chcesz pobrać pojedynczy wiersz z tabeli bazy danych, ale zgłosić wyjątek `Illuminate\Database\RecordNotFoundException`, jeśli nie znaleziono pasującego wiersza, możesz użyć metody `firstOrFail`. Jeśli wyjątek `RecordNotFoundException` nie zostanie przechwycony, odpowiedź HTTP 404 jest automatycznie wysyłana do klienta:

```php
$user = DB::table('users')->where('name', 'John')->firstOrFail();
```

Jeśli nie potrzebujesz całego wiersza, możesz wyodrębnić pojedynczą wartość z rekordu za pomocą metody `value`. Ta metoda zwróci wartość kolumny bezpośrednio:

```php
$email = DB::table('users')->where('name', 'John')->value('email');
```

Aby pobrać pojedynczy wiersz według wartości kolumny `id`, użyj metody `find`:

```php
$user = DB::table('users')->find(3);
```

<a name="retrieving-a-list-of-column-values"></a>
#### Pobieranie listy wartości kolumn

Jeśli chcesz pobrać instancję `Illuminate\Support\Collection` zawierającą wartości pojedynczej kolumny, możesz użyć metody `pluck`. W tym przykładzie pobierzemy kolekcję tytułów użytkowników:

```php
use Illuminate\Support\Facades\DB;

$titles = DB::table('users')->pluck('title');

foreach ($titles as $title) {
    echo $title;
}
```

Możesz określić kolumnę, której wynikowa kolekcja powinna używać jako kluczy, podając drugi argument metody `pluck`:

```php
$titles = DB::table('users')->pluck('title', 'name');

foreach ($titles as $name => $title) {
    echo $title;
}
```

<a name="chunking-results"></a>
### Porcjowanie wyników

Jeśli musisz pracować z tysiącami rekordów bazy danych, rozważ użycie metody `chunk` dostarczonej przez fasadę `DB`. Ta metoda pobiera małą porcję wyników na raz i przekazuje każdą porcję do domknięcia w celu przetworzenia. Na przykład pobierzmy całą tabelę `users` w porcjach po 100 rekordów na raz:

```php
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\DB;

DB::table('users')->orderBy('id')->chunk(100, function (Collection $users) {
    foreach ($users as $user) {
        // ...
    }
});
```

Możesz zatrzymać przetwarzanie kolejnych porcji, zwracając `false` z domknięcia:

```php
DB::table('users')->orderBy('id')->chunk(100, function (Collection $users) {
    // Process the records...

    return false;
});
```

Jeśli aktualizujesz rekordy bazy danych podczas porcjowania wyników, Twoje wyniki porcji mogą się zmienić w nieoczekiwany sposób. Jeśli planujesz zaktualizować pobrane rekordy podczas porcjowania, zawsze najlepiej jest użyć metody `chunkById`. Ta metoda automatycznie podzieli wyniki na strony na podstawie klucza głównego rekordu:

```php
DB::table('users')->where('active', false)
    ->chunkById(100, function (Collection $users) {
        foreach ($users as $user) {
            DB::table('users')
                ->where('id', $user->id)
                ->update(['active' => true]);
        }
    });
```

Since the `chunkById` and `lazyById` methods add their own "where" conditions to the query being executed, you should typically [logically group](#logical-grouping) your own conditions within a closure:

```php
DB::table('users')->where(function ($query) {
    $query->where('credits', 1)->orWhere('credits', 2);
})->chunkById(100, function (Collection $users) {
    foreach ($users as $user) {
        DB::table('users')
            ->where('id', $user->id)
            ->update(['credits' => 3]);
    }
});
```

> [!WARNING]
> Podczas aktualizacji lub usuwania rekordów wewnątrz callbacku porcji, wszelkie zmiany w kluczu głównym lub kluczach obcych mogą wpłynąć na zapytanie porcji. Może to potencjalnie spowodować, że rekordy nie będą uwzględnione w wynikach porcji.

<a name="streaming-results-lazily"></a>
### Leniwe strumieniowanie wyników

Metoda `lazy` działa podobnie do [metody chunk](#chunking-results) w tym sensie, że wykonuje zapytanie w porcjach. Jednak zamiast przekazywać każdą porcję do callbacku, metoda `lazy()` zwraca [LazyCollection](/docs/{{version}}/collections#lazy-collections), co pozwala na interakcję z wynikami jako pojedynczym strumieniem:

```php
use Illuminate\Support\Facades\DB;

DB::table('users')->orderBy('id')->lazy()->each(function (object $user) {
    // ...
});
```

Ponownie, jeśli planujesz zaktualizować pobrane rekordy podczas iteracji po nich, najlepiej użyć metod `lazyById` lub `lazyByIdDesc`. Te metody automatycznie podzielą wyniki na strony na podstawie klucza głównego rekordu:

```php
DB::table('users')->where('active', false)
    ->lazyById()->each(function (object $user) {
        DB::table('users')
            ->where('id', $user->id)
            ->update(['active' => true]);
    });
```

> [!WARNING]
> Podczas aktualizacji lub usuwania rekordów podczas iteracji po nich, wszelkie zmiany w kluczu głównym lub kluczach obcych mogą wpłynąć na zapytanie porcji. Może to potencjalnie spowodować, że rekordy nie będą uwzględnione w wynikach.

<a name="aggregates"></a>
### Agregaty

Konstruktor zapytań zapewnia również różne metody pobierania wartości agregowanych, takich jak `count`, `max`, `min`, `avg` i `sum`. Możesz wywołać dowolną z tych metod po zbudowaniu zapytania:

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')->count();

$price = DB::table('orders')->max('price');
```

Oczywiście możesz połączyć te metody z innymi klauzulami, aby dostroić sposób obliczania wartości agregowanej:

```php
$price = DB::table('orders')
    ->where('finalized', 1)
    ->avg('price');
```

<a name="determining-if-records-exist"></a>
#### Określanie, czy rekordy istnieją

Zamiast używać metody `count`, aby określić, czy istnieją jakieś rekordy pasujące do ograniczeń zapytania, możesz użyć metod `exists` i `doesntExist`:

```php
if (DB::table('orders')->where('finalized', 1)->exists()) {
    // ...
}

if (DB::table('orders')->where('finalized', 1)->doesntExist()) {
    // ...
}
```

<a name="select-statements"></a>
## Instrukcje Select

<a name="specifying-a-select-clause"></a>
#### Określanie klauzuli Select

Mo?esz nie zawsze chcie? wybiera? wszystkie kolumny z tabeli bazy danych. U?ywaj?c metody `select`, mo?esz okre?li? niestandardow? klauzul? "select" dla zapytania:

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')
    ->select('name', 'email as user_email')
    ->get();
```

Metoda `distinct` pozwala wymusi? na zapytaniu zwracanie r??nych wynik?w:

```php
$users = DB::table('users')->distinct()->get();
```

Je?li masz ju? instancj? konstruktora zapyta? i chcesz doda? kolumn? do jego istniej?cej klauzuli select, mo?esz u?y? metody `addSelect`:

```php
$query = DB::table('users')->select('name');

$users = $query->addSelect('age')->get();
```

<a name="raw-expressions"></a>
## Wyrażenia surowe

Czasami mo?esz potrzebowa? wstawi? dowolny ci?g do zapytania. Aby utworzy? surowe wyra?enie ci?gu, mo?esz u?y? metody `raw` dostarczonej przez fasad? `DB`:

```php
$users = DB::table('users')
    ->select(DB::raw('count(*) as user_count, status'))
    ->where('status', '<>', 1)
    ->groupBy('status')
    ->get();
```

> [!WARNING]
> Surowe instrukcje b?d? wstrzykiwane do zapytania jako ci?gi, wi?c powiniene? by? bardzo ostro?ny, aby unikn?? tworzenia luk SQL injection.

<a name="raw-methods"></a>
### Metody surowe

Zamiast u?ywa? metody `DB::raw`, mo?esz r?wnie? u?y? nast?puj?cych metod, aby wstawi? surowe wyra?enie do r??nych cz??ci zapytania. **Pami?taj, ?e Laravel nie mo?e zagwarantowa?, ?e ka?de zapytanie u?ywaj?ce surowych wyra?e? jest chronione przed lukami SQL injection.**

<a name="selectraw"></a>
#### `selectRaw`

The `selectRaw` method can be used in place of `addSelect(DB::raw(/* ... */))`. This method accepts an optional array of bindings as its second argument:

```php
$orders = DB::table('orders')
    ->selectRaw('price * ? as price_with_tax', [1.0825])
    ->get();
```

<a name="whereraw-orwhereraw"></a>
#### `whereRaw / orWhereRaw`

The `whereRaw` and `orWhereRaw` methods can be used to inject a raw "where" clause into your query. These methods accept an optional array of bindings as their second argument:

```php
$orders = DB::table('orders')
    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
    ->get();
```

<a name="havingraw-orhavingraw"></a>
#### `havingRaw / orHavingRaw`

The `havingRaw` and `orHavingRaw` methods may be used to provide a raw string as the value of the "having" clause. These methods accept an optional array of bindings as their second argument:

```php
$orders = DB::table('orders')
    ->select('department', DB::raw('SUM(price) as total_sales'))
    ->groupBy('department')
    ->havingRaw('SUM(price) > ?', [2500])
    ->get();
```

<a name="orderbyraw"></a>
#### `orderByRaw`

The `orderByRaw` method may be used to provide a raw string as the value of the "order by" clause:

```php
$orders = DB::table('orders')
    ->orderByRaw('updated_at - created_at DESC')
    ->get();
```

<a name="groupbyraw"></a>
### `groupByRaw`

The `groupByRaw` method may be used to provide a raw string as the value of the `group by` clause:

```php
$orders = DB::table('orders')
    ->select('city', 'state')
    ->groupByRaw('city, state')
    ->get();
```

<a name="joins"></a>
## Złączenia

<a name="inner-join-clause"></a>
#### Klauzula Inner Join

Konstruktor zapyta? mo?e by? r?wnie? u?ywany do dodawania klauzul join do Twoich zapyta?. Aby wykona? podstawowe "inner join", mo?esz u?y? metody `join` na instancji konstruktora zapyta?. Pierwszy argument przekazany do metody `join` to nazwa tabeli, do kt?rej musisz do??czy?, podczas gdy pozosta?e argumenty okre?laj? ograniczenia kolumn dla z??czenia. Mo?esz nawet ??czy? wiele tabel w pojedynczym zapytaniu:

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')
    ->join('contacts', 'users.id', '=', 'contacts.user_id')
    ->join('orders', 'users.id', '=', 'orders.user_id')
    ->select('users.*', 'contacts.phone', 'orders.price')
    ->get();
```

<a name="left-join-right-join-clause"></a>
#### Klauzula Left Join / Right Join

Je?li chcesz wykona? "left join" lub "right join" zamiast "inner join", u?yj metod `leftJoin` lub `rightJoin`. Te metody maj? ten sam podpis co metoda `join`:

```php
$users = DB::table('users')
    ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
    ->get();

$users = DB::table('users')
    ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
    ->get();
```

<a name="cross-join-clause"></a>
#### Klauzula Cross Join

Mo?esz u?y? metody `crossJoin`, aby wykona? "cross join". Z??czenia krzy?owe generuj? iloczyn kartezja?ski mi?dzy pierwsz? tabel? a po??czon? tabel?:

```php
$sizes = DB::table('sizes')
    ->crossJoin('colors')
    ->get();
```

<a name="advanced-join-clauses"></a>
#### Zaawansowane klauzule Join

Mo?esz r?wnie? okre?li? bardziej zaawansowane klauzule join. Aby rozpocz??, przeka? domkni?cie jako drugi argument do metody `join`. Domkni?cie otrzyma instancj? `Illuminate\Database\Query\JoinClause`, kt?ra pozwala okre?li? ograniczenia w klauzuli "join":

```php
DB::table('users')
    ->join('contacts', function (JoinClause $join) {
        $join->on('users.id', '=', 'contacts.user_id')->orOn(/* ... */);
    })
    ->get();
```

If you would like to use a "where" clause on your joins, you may use the `where` and `orWhere` methods provided by the `JoinClause` instance. Instead of comparing two columns, these methods will compare the column against a value:

```php
DB::table('users')
    ->join('contacts', function (JoinClause $join) {
        $join->on('users.id', '=', 'contacts.user_id')
            ->where('contacts.user_id', '>', 5);
    })
    ->get();
```

<a name="subquery-joins"></a>
#### Złączenia z podzapytaniami

You may use the `joinSub`, `leftJoinSub`, and `rightJoinSub` methods to join a query to a subquery. Each of these methods receives three arguments: the subquery, its table alias, and a closure that defines the related columns. In this example, we will retrieve a collection of users where each user record also contains the `created_at` timestamp of the user's most recently published blog post:

```php
$latestPosts = DB::table('posts')
    ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
    ->where('is_published', true)
    ->groupBy('user_id');

$users = DB::table('users')
    ->joinSub($latestPosts, 'latest_posts', function (JoinClause $join) {
        $join->on('users.id', '=', 'latest_posts.user_id');
    })->get();
```

<a name="lateral-joins"></a>
#### Złączenia lateralne

> [!WARNING]
> Lateral joins are currently supported by PostgreSQL, MySQL >= 8.0.14, and SQL Server.

You may use the `joinLateral` and `leftJoinLateral` methods to perform a "lateral join" with a subquery. Each of these methods receives two arguments: the subquery and its table alias. The join condition(s) should be specified within the `where` clause of the given subquery. Lateral joins are evaluated for each row and can reference columns outside the subquery.

In this example, we will retrieve a collection of users as well as the user's three most recent blog posts. Each user can produce up to three rows in the result set: one for each of their most recent blog posts. The join condition is specified with a `whereColumn` clause within the subquery, referencing the current user row:

```php
$latestPosts = DB::table('posts')
    ->select('id as post_id', 'title as post_title', 'created_at as post_created_at')
    ->whereColumn('user_id', 'users.id')
    ->orderBy('created_at', 'desc')
    ->limit(3);

$users = DB::table('users')
    ->joinLateral($latestPosts, 'latest_posts')
    ->get();
```

<a name="unions"></a>
## Unie

Konstruktor zapyta? zapewnia r?wnie? wygodn? metod? "union" do ??czenia dw?ch lub wi?cej zapyta?. Na przyk?ad mo?esz utworzy? pocz?tkowe zapytanie i u?y? metody `union`, aby po??czy? je z wi?ksz? liczb? zapyta?:

```php
use Illuminate\Support\Facades\DB;

$first = DB::table('users')
    ->whereNull('first_name');

$users = DB::table('users')
    ->whereNull('last_name')
    ->union($first)
    ->get();
```

In addition to the `union` method, the query builder provides a `unionAll` method. Queries that are combined using the `unionAll` method will not have their duplicate results removed. The `unionAll` method has the same method signature as the `union` method.

<a name="basic-where-clauses"></a>
## Podstawowe klauzule Where

<a name="where-clauses"></a>
### Klauzule Where

Możesz użyć metody `where` konstruktora zapytań, aby dodać klauzule "where" do zapytania. Najbardziej podstawowe wywołanie metody `where` wymaga trzech argumentów. Pierwszy argument to nazwa kolumny. Drugi argument to operator, którym może być dowolny z obsługiwanych przez bazę danych operatorów. Trzeci argument to wartość do porównania z wartością kolumny.

Na przykład następujące zapytanie pobiera użytkowników, gdzie wartość kolumny `votes` jest równa `100`, a wartość kolumny `age` jest większa niż `35`:

```php
$users = DB::table('users')
    ->where('votes', '=', 100)
    ->where('age', '>', 35)
    ->get();
```

Dla wygody, jeśli chcesz sprawdzić, czy kolumna jest `=` danej wartości, możesz przekazać wartość jako drugi argument metody `where`. Laravel założy, że chciałbyś użyć operatora `=`:

```php
$users = DB::table('users')->where('votes', 100)->get();
```

Możesz również przekazać tablicę asocjacyjną do metody `where`, aby szybko wykonać zapytanie względem wielu kolumn:

```php
$users = DB::table('users')->where([
    'first_name' => 'Jane',
    'last_name' => 'Doe',
])->get();
```

Jak wcześniej wspomniano, możesz użyć dowolnego operatora obsługiwanego przez system bazy danych:

```php
$users = DB::table('users')
    ->where('votes', '>=', 100)
    ->get();

$users = DB::table('users')
    ->where('votes', '<>', 100)
    ->get();

$users = DB::table('users')
    ->where('name', 'like', 'T%')
    ->get();
```

Możesz również przekazać tablicę warunków do funkcji `where`. Każdy element tablicy powinien być tablicą zawierającą trzy argumenty zwykle przekazywane do metody `where`:

```php
$users = DB::table('users')->where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])->get();
```

> [!WARNING]
> PDO nie obsługuje wiązania nazw kolumn. Dlatego nigdy nie należy pozwalać na to, aby dane wejściowe użytkownika dyktowały nazwy kolumn, do których odwołują się Twoje zapytania, w tym kolumny "order by".

> [!WARNING]
> MySQL i MariaDB automatycznie konwertują ciągi na liczby całkowite w porównaniach ciąg-liczba. W tym procesie ciągi nienumeryczne są konwertowane na `0`, co może prowadzić do nieoczekiwanych wyników. Na przykład, jeśli Twoja tabela ma kolumnę `secret` z wartością `aaa` i uruchomisz `User::where('secret', 0)`, ten wiersz zostanie zwrócony. Aby tego uniknąć, upewnij się, że wszystkie wartości są konwertowane na odpowiednie typy przed użyciem ich w zapytaniach.

<a name="or-where-clauses"></a>
### Klauzule Or Where

Podczas łączenia wywołań metody `where` konstruktora zapytań, klauzule "where" będą łączone razem za pomocą operatora `and`. Jednak możesz użyć metody `orWhere`, aby dołączyć klauzulę do zapytania za pomocą operatora `or`. Metoda `orWhere` akceptuje te same argumenty co metoda `where`:

```php
$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhere('name', 'John')
    ->get();
```

Jeśli musisz zgrupować warunek "or" w nawiasach, możesz przekazać domknięcie jako pierwszy argument metody `orWhere`:

```php
use Illuminate\Database\Query\Builder; 

$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhere(function (Builder $query) {
        $query->where('name', 'Abigail')
            ->where('votes', '>', 50);
        })
    ->get();
```

Powyższy przykład wygeneruje następujący kod SQL:

```sql
select * from users where votes > 100 or (name = 'Abigail' and votes > 50)
```

> [!WARNING]
> Zawsze powinieneś grupować wywołania `orWhere`, aby uniknąć nieoczekiwanego zachowania, gdy stosowane są zakresy globalne.

<a name="where-not-clauses"></a>
### Klauzule Where Not

Metody `whereNot` i `orWhereNot` mogą być używane do negacji danej grupy ograniczeń zapytania. Na przykład następujące zapytanie wyklucza produkty, które są na wyprzedaży lub mają cenę mniejszą niż dziesięć:

```php
$products = DB::table('products')
    ->whereNot(function (Builder $query) {
        $query->where('clearance', true)
            ->orWhere('price', '<', 10);
        })
    ->get();
```

<a name="where-any-all-none-clauses"></a>
### Klauzule Where Any / All / None

Czasami możesz potrzebować zastosować te same ograniczenia zapytania do wielu kolumn. Na przykład możesz chcieć pobrać wszystkie rekordy, w których dowolne kolumny z danej listy są `LIKE` danej wartości. Możesz to osiągnąć za pomocą metody `whereAny`:

```php
$users = DB::table('users')
    ->where('active', true)
    ->whereAny([
        'name',
        'email',
        'phone',
    ], 'like', 'Example%')
    ->get();
```

Powyższe zapytanie spowoduje utworzenie następującego SQL:

```sql
SELECT *
FROM users
WHERE active = true AND (
    name LIKE 'Example%' OR
    email LIKE 'Example%' OR
    phone LIKE 'Example%'
)
```

Podobnie metoda `whereAll` może być używana do pobierania rekordów, w których wszystkie podane kolumny pasują do danego ograniczenia:

```php
$posts = DB::table('posts')
    ->where('published', true)
    ->whereAll([
        'title',
        'content',
    ], 'like', '%Laravel%')
    ->get();
```

Powyższe zapytanie spowoduje utworzenie następującego SQL:

```sql
SELECT *
FROM posts
WHERE published = true AND (
    title LIKE '%Laravel%' AND
    content LIKE '%Laravel%'
)
```

Metoda `whereNone` może być używana do pobierania rekordów, w których żadna z podanych kolumn nie pasuje do danego ograniczenia:

```php
$albums = DB::table('albums')
    ->where('published', true)
    ->whereNone([
        'title',
        'lyrics',
        'tags',
    ], 'like', '%explicit%')
    ->get();
```

Powyższe zapytanie spowoduje utworzenie następującego SQL:

```sql
SELECT *
FROM albums
WHERE published = true AND NOT (
    title LIKE '%explicit%' OR
    lyrics LIKE '%explicit%' OR
    tags LIKE '%explicit%'
)
```

<a name="json-where-clauses"></a>
### Klauzule JSON Where

Laravel obsługuje również zapytania o typy kolumn JSON w bazach danych, które zapewniają wsparcie dla typów kolumn JSON. Obecnie obejmuje to MariaDB 10.3+, MySQL 8.0+, PostgreSQL 12.0+, SQL Server 2017+ i SQLite 3.39.0+. Aby odpytać kolumnę JSON, użyj operatora `->`:

```php
$users = DB::table('users')
    ->where('preferences->dining->meal', 'salad')
    ->get();

$users = DB::table('users')
    ->whereIn('preferences->dining->meal', ['pasta', 'salad', 'sandwiches'])
    ->get();
```

Możesz użyć metod `whereJsonContains` i `whereJsonDoesntContain`, aby odpytać tablice JSON:

```php
$users = DB::table('users')
    ->whereJsonContains('options->languages', 'en')
    ->get();

$users = DB::table('users')
    ->whereJsonDoesntContain('options->languages', 'en')
    ->get();
```

Jeśli Twoja aplikacja używa baz danych MariaDB, MySQL lub PostgreSQL, możesz przekazać tablicę wartości do metod `whereJsonContains` i `whereJsonDoesntContain`:

```php
$users = DB::table('users')
    ->whereJsonContains('options->languages', ['en', 'de'])
    ->get();

$users = DB::table('users')
    ->whereJsonDoesntContain('options->languages', ['en', 'de'])
    ->get();
```

Ponadto możesz użyć metod `whereJsonContainsKey` lub `whereJsonDoesntContainKey`, aby pobrać wyniki, które zawierają lub nie zawierają klucza JSON:

```php
$users = DB::table('users')
    ->whereJsonContainsKey('preferences->dietary_requirements')
    ->get();

$users = DB::table('users')
    ->whereJsonDoesntContainKey('preferences->dietary_requirements')
    ->get();
```

Na koniec możesz użyć metody `whereJsonLength`, aby odpytać tablice JSON według ich długości:

```php
$users = DB::table('users')
    ->whereJsonLength('options->languages', 0)
    ->get();

$users = DB::table('users')
    ->whereJsonLength('options->languages', '>', 1)
    ->get();
```

<a name="additional-where-clauses"></a>
### Dodatkowe klauzule Where

**whereLike / orWhereLike / whereNotLike / orWhereNotLike**

Metoda `whereLike` pozwala dodać klauzule "LIKE" do zapytania do dopasowywania wzorców. Te metody zapewniają niezależny od bazy danych sposób wykonywania zapytań dopasowywania ciągów z możliwością przełączania wrażliwości na wielkość liter. Domyślnie dopasowywanie ciągów jest niewrażliwe na wielkość liter:

```php
$users = DB::table('users')
    ->whereLike('name', '%John%')
    ->get();
```

Możesz włączyć wyszukiwanie wrażliwe na wielkość liter za pomocą argumentu `caseSensitive`:

```php
$users = DB::table('users')
    ->whereLike('name', '%John%', caseSensitive: true)
    ->get();
```

Metoda `orWhereLike` pozwala dodać klauzulę "or" z warunkiem LIKE:

```php
$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhereLike('name', '%John%')
    ->get();
```

Metoda `whereNotLike` pozwala dodać klauzule "NOT LIKE" do zapytania:

```php
$users = DB::table('users')
    ->whereNotLike('name', '%John%')
    ->get();
```

Podobnie możesz użyć `orWhereNotLike`, aby dodać klauzulę "or" z warunkiem NOT LIKE:

```php
$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhereNotLike('name', '%John%')
    ->get();
```

> [!WARNING]
> Opcja wyszukiwania z rozróżnianiem wielkości liter `whereLike` nie jest obecnie obsługiwana w SQL Server.

**whereIn / whereNotIn / orWhereIn / orWhereNotIn**

Metoda `whereIn` weryfikuje, że wartość danej kolumny jest zawarta w danej tablicy:

```php
$users = DB::table('users')
    ->whereIn('id', [1, 2, 3])
    ->get();
```

Metoda `whereNotIn` weryfikuje, że wartość danej kolumny nie jest zawarta w danej tablicy:

```php
$users = DB::table('users')
    ->whereNotIn('id', [1, 2, 3])
    ->get();
```

Możesz również przekazać obiekt zapytania jako drugi argument metody `whereIn`:

```php
$activeUsers = DB::table('users')->select('id')->where('is_active', 1);

$comments = DB::table('comments')
    ->whereIn('user_id', $activeUsers)
    ->get();
```

Powyższy przykład wygeneruje następujący kod SQL:

```sql
select * from comments where user_id in (
    select id
    from users
    where is_active = 1
)
```

> [!WARNING]
> Jeśli dodajesz dużą tablicę wiązań całkowitych do zapytania, metody `whereIntegerInRaw` lub `whereIntegerNotInRaw` mogą być użyte do znacznego zmniejszenia zużycia pamięci.

**whereBetween / orWhereBetween**

Metoda `whereBetween` weryfikuje, że wartość kolumny znajduje się między dwiema wartościami:

```php
$users = DB::table('users')
    ->whereBetween('votes', [1, 100])
    ->get();
```

**whereNotBetween / orWhereNotBetween**

Metoda `whereNotBetween` weryfikuje, że wartość kolumny leży poza dwiema wartościami:

```php
$users = DB::table('users')
    ->whereNotBetween('votes', [1, 100])
    ->get();
```

**whereBetweenColumns / whereNotBetweenColumns / orWhereBetweenColumns / orWhereNotBetweenColumns**

Metoda `whereBetweenColumns` weryfikuje, że wartość kolumny znajduje się między dwiema wartościami dwóch kolumn w tym samym wierszu tabeli:

```php
$patients = DB::table('patients')
    ->whereBetweenColumns('weight', ['minimum_allowed_weight', 'maximum_allowed_weight'])
    ->get();
```

The `whereNotBetweenColumns` method verifies that a column's value lies outside the two values of two columns in the same table row:

```php
$patients = DB::table('patients')
    ->whereNotBetweenColumns('weight', ['minimum_allowed_weight', 'maximum_allowed_weight'])
    ->get();
```

**whereValueBetween / whereValueNotBetween / orWhereValueBetween / orWhereValueNotBetween**

The `whereValueBetween` method verifies that a given value is between the values of two columns of the same type in the same table row:

```php
$patients = DB::table('products')
    ->whereValueBetween(100, ['min_price', 'max_price'])
    ->get();
```

The `whereValueNotBetween` method verifies that a value lies outside the values of two columns in the same table row:

```php
$patients = DB::table('products')
    ->whereValueNotBetween(100, ['min_price', 'max_price'])
    ->get();
```

**whereNull / whereNotNull / orWhereNull / orWhereNotNull**

Metoda `whereNull` weryfikuje, że wartość danej kolumny to `NULL`:

```php
$users = DB::table('users')
    ->whereNull('updated_at')
    ->get();
```

Metoda `whereNotNull` weryfikuje, że wartość kolumny nie jest `NULL`:

```php
$users = DB::table('users')
    ->whereNotNull('updated_at')
    ->get();
```

**whereDate / whereMonth / whereDay / whereYear / whereTime**

Metoda `whereDate` może być używana do porównywania wartości kolumny z datą:

```php
$users = DB::table('users')
    ->whereDate('created_at', '2016-12-31')
    ->get();
```

Metoda `whereMonth` może być używana do porównywania wartości kolumny z określonym miesiącem:

```php
$users = DB::table('users')
    ->whereMonth('created_at', '12')
    ->get();
```

Metoda `whereDay` może być używana do porównywania wartości kolumny z określonym dniem miesiąca:

```php
$users = DB::table('users')
    ->whereDay('created_at', '31')
    ->get();
```

Metoda `whereYear` może być używana do porównywania wartości kolumny z określonym rokiem:

```php
$users = DB::table('users')
    ->whereYear('created_at', '2016')
    ->get();
```

Metoda `whereTime` może być używana do porównywania wartości kolumny z określonym czasem:

```php
$users = DB::table('users')
    ->whereTime('created_at', '=', '11:20:45')
    ->get();
```

**wherePast / whereFuture / whereToday / whereBeforeToday / whereAfterToday**

The `wherePast` and `whereFuture` methods may be used to determine if a column's value is in the past or future:

```php
$invoices = DB::table('invoices')
    ->wherePast('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereFuture('due_at')
    ->get();
```

The `whereNowOrPast` and `whereNowOrFuture` methods may be used to determine if a column's value is in the past or future, inclusive of the current date and time:

```php
$invoices = DB::table('invoices')
    ->whereNowOrPast('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereNowOrFuture('due_at')
    ->get();
```

The `whereToday`, `whereBeforeToday`, and `whereAfterToday` methods may be used to determine if a column's value is today, before today, or after today, respectively:

```php
$invoices = DB::table('invoices')
    ->whereToday('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereBeforeToday('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereAfterToday('due_at')
    ->get();
```

Similarly, the `whereTodayOrBefore` and `whereTodayOrAfter` methods may be used to determine if a column's value is before today or after today, inclusive of today's date:

```php
$invoices = DB::table('invoices')
    ->whereTodayOrBefore('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereTodayOrAfter('due_at')
    ->get();
```

**whereColumn / orWhereColumn**

Metoda `whereColumn` może być używana do weryfikacji, że dwie kolumny są równe:

```php
$users = DB::table('users')
    ->whereColumn('first_name', 'last_name')
    ->get();
```

Możesz również przekazać operator porównania do metody `whereColumn`:

```php
$users = DB::table('users')
    ->whereColumn('updated_at', '>', 'created_at')
    ->get();
```

Możesz również przekazać tablicę porównań kolumn do metody `whereColumn`. Te warunki zostaną połączone za pomocą operatora `and`:

```php
$users = DB::table('users')
    ->whereColumn([
        ['first_name', '=', 'last_name'],
        ['updated_at', '>', 'created_at'],
    ])->get();
```

<a name="logical-grouping"></a>
### Grupowanie logiczne

Czasami możesz potrzebować zgrupować kilka klauzul "where" w nawiasach, aby osiągnąć pożądane logiczne grupowanie zapytania. W rzeczywistości generalnie powinieneś zawsze grupować wywołania metody `orWhere` w nawiasach, aby uniknąć nieoczekiwanego zachowania zapytania. Aby to osiągnąć, możesz przekazać domknięcie do metody `where`:

```php
$users = DB::table('users')
    ->where('name', '=', 'John')
    ->where(function (Builder $query) {
        $query->where('votes', '>', 100)
            ->orWhere('title', '=', 'Admin');
    })
    ->get();
```

Jak widać, przekazanie domknięcia do metody `where` instruuje konstruktor zapytań, aby rozpoczął grupę ograniczeń. Domknięcie otrzyma instancję konstruktora zapytań, której możesz użyć do ustawienia ograniczeń, które powinny być zawarte w grupie nawiasowej. Powyższy przykład wygeneruje następujący kod SQL:

```sql
select * from users where name = 'John' and (votes > 100 or title = 'Admin')
```

> [!WARNING]
> Zawsze powinieneś grupować wywołania `orWhere`, aby uniknąć nieoczekiwanego zachowania, gdy stosowane są zakresy globalne.

<a name="advanced-where-clauses"></a>
## Zaawansowane klauzule Where

<a name="where-exists-clauses"></a>
### Klauzule Where Exists

Metoda `whereExists` pozwala pisać klauzule SQL "where exists". Metoda `whereExists` akceptuje domknięcie, które otrzyma instancję konstruktora zapytań, umożliwiając zdefiniowanie zapytania, które powinno być umieszczone wewnątrz klauzuli "exists":

```php
$users = DB::table('users')
    ->whereExists(function (Builder $query) {
        $query->select(DB::raw(1))
            ->from('orders')
            ->whereColumn('orders.user_id', 'users.id');
    })
    ->get();
```

Alternatywnie możesz przekazać obiekt zapytania do metody `whereExists` zamiast domknięcia:

```php
$orders = DB::table('orders')
    ->select(DB::raw(1))
    ->whereColumn('orders.user_id', 'users.id');

$users = DB::table('users')
    ->whereExists($orders)
    ->get();
```

Oba powyższe przykłady wygenerują następujący SQL:

```sql
select * from users
where exists (
    select 1
    from orders
    where orders.user_id = users.id
)
```

<a name="subquery-where-clauses"></a>
### Podzapytania Where

Czasami możesz potrzebować skonstruować klauzulę "where", która porównuje wyniki podzapytania z daną wartością. Możesz to osiągnąć, przekazując domknięcie i wartość do metody `where`. Na przykład następujące zapytanie pobierze wszystkich użytkowników, którzy mają niedawne "członkostwo" danego typu;

```php
use App\Models\User;
use Illuminate\Database\Query\Builder;

$users = User::where(function (Builder $query) {
    $query->select('type')
        ->from('membership')
        ->whereColumn('membership.user_id', 'users.id')
        ->orderByDesc('membership.start_date')
        ->limit(1);
}, 'Pro')->get();
```

Lub możesz potrzebować skonstruować klauzulę "where", która porównuje kolumnę z wynikami podzapytania. Możesz to osiągnąć, przekazując kolumnę, operator i domknięcie do metody `where`. Na przykład następujące zapytanie pobierze wszystkie rekordy dochodów, gdzie kwota jest mniejsza niż średnia;

```php
use App\Models\Income;
use Illuminate\Database\Query\Builder;

$incomes = Income::where('amount', '<', function (Builder $query) {
    $query->selectRaw('avg(i.amount)')->from('incomes as i');
})->get();
```

<a name="full-text-where-clauses"></a>
### Klauzule pełnotekstowe Where

> [!WARNING]
> Klauzule pełnotekstowe where są obecnie obsługiwane przez MariaDB, MySQL i PostgreSQL.

Metody `whereFullText` i `orWhereFullText` mogą być używane do dodawania pełnotekstowych klauzul "where" do zapytania dla kolumn, które mają [indeksy pełnotekstowe](/docs/{{version}}/migrations#available-index-types). Te metody zostaną przekształcone w odpowiedni SQL dla podstawowego systemu bazy danych przez Laravel. Na przykład klauzula `MATCH AGAINST` zostanie wygenerowana dla aplikacji wykorzystujących MariaDB lub MySQL:

```php
$users = DB::table('users')
    ->whereFullText('bio', 'web developer')
    ->get();
```

<a name="ordering-grouping-limit-and-offset"></a>
## Sortowanie, grupowanie, limit i offset

<a name="ordering"></a>
### Sortowanie

<a name="orderby"></a>
#### Metoda `orderBy`

The `orderBy` method allows you to sort the results of the query by a given column. The first argument accepted by the `orderBy` method should be the column you wish to sort by, while the second argument determines the direction of the sort and may be either `asc` or `desc`:

```php
$users = DB::table('users')
    ->orderBy('name', 'desc')
    ->get();
```

To sort by multiple columns, you may simply invoke `orderBy` as many times as necessary:

```php
$users = DB::table('users')
    ->orderBy('name', 'desc')
    ->orderBy('email', 'asc')
    ->get();
```

The sort direction is optional, and is ascending by default. If you want to sort in descending order, you can specify the second parameter for the `orderBy` method, or just use `orderByDesc`:

```php
$users = DB::table('users')
    ->orderByDesc('verified_at')
    ->get();
```

Finally, using the `->` operator, the results can be sorted by a value within a JSON column:

```php
$corporations = DB::table('corporations')
    ->where('country', 'US')
    ->orderBy('location->state')
    ->get();
```

<a name="latest-oldest"></a>
#### Metody `latest` i `oldest`

The `latest` and `oldest` methods allow you to easily order results by date. By default, the result will be ordered by the table's `created_at` column. Or, you may pass the column name that you wish to sort by:

```php
$user = DB::table('users')
    ->latest()
    ->first();
```

<a name="random-ordering"></a>
#### Losowe sortowanie

The `inRandomOrder` method may be used to sort the query results randomly. For example, you may use this method to fetch a random user:

```php
$randomUser = DB::table('users')
    ->inRandomOrder()
    ->first();
```

<a name="removing-existing-orderings"></a>
#### Usuwanie istniejących sortowań

The `reorder` method removes all of the "order by" clauses that have previously been applied to the query:

```php
$query = DB::table('users')->orderBy('name');

$unorderedUsers = $query->reorder()->get();
```

You may pass a column and direction when calling the `reorder` method in order to remove all existing "order by" clauses and apply an entirely new order to the query:

```php
$query = DB::table('users')->orderBy('name');

$usersOrderedByEmail = $query->reorder('email', 'desc')->get();
```

For convenience, you may use the `reorderDesc` method to reorder the query results in descending order:

```php
$query = DB::table('users')->orderBy('name');

$usersOrderedByEmail = $query->reorderDesc('email')->get();
```

<a name="grouping"></a>
### Grupowanie

<a name="groupby-having"></a>
#### Metody `groupBy` i `having`

As you might expect, the `groupBy` and `having` methods may be used to group the query results. The `having` method's signature is similar to that of the `where` method:

```php
$users = DB::table('users')
    ->groupBy('account_id')
    ->having('account_id', '>', 100)
    ->get();
```

You can use the `havingBetween` method to filter the results within a given range:

```php
$report = DB::table('orders')
    ->selectRaw('count(id) as number_of_orders, customer_id')
    ->groupBy('customer_id')
    ->havingBetween('number_of_orders', [5, 15])
    ->get();
```

You may pass multiple arguments to the `groupBy` method to group by multiple columns:

```php
$users = DB::table('users')
    ->groupBy('first_name', 'status')
    ->having('account_id', '>', 100)
    ->get();
```

To build more advanced `having` statements, see the [havingRaw](#raw-methods) method.

<a name="limit-and-offset"></a>
### Limit i Offset

You may use the `limit` and `offset` methods to limit the number of results returned from the query or to skip a given number of results in the query:

```php
$users = DB::table('users')
    ->offset(10)
    ->limit(5)
    ->get();
```

<a name="conditional-clauses"></a>
## Klauzule warunkowe

Sometimes you may want certain query clauses to apply to a query based on another condition. For instance, you may only want to apply a `where` statement if a given input value is present on the incoming HTTP request. You may accomplish this using the `when` method:

```php
$role = $request->input('role');

$users = DB::table('users')
    ->when($role, function (Builder $query, string $role) {
        $query->where('role_id', $role);
    })
    ->get();
```

The `when` method only executes the given closure when the first argument is `true`. If the first argument is `false`, the closure will not be executed. So, in the example above, the closure given to the `when` method will only be invoked if the `role` field is present on the incoming request and evaluates to `true`.

You may pass another closure as the third argument to the `when` method. This closure will only execute if the first argument evaluates as `false`. To illustrate how this feature may be used, we will use it to configure the default ordering of a query:

```php
$sortByVotes = $request->boolean('sort_by_votes');

$users = DB::table('users')
    ->when($sortByVotes, function (Builder $query, bool $sortByVotes) {
        $query->orderBy('votes');
    }, function (Builder $query) {
        $query->orderBy('name');
    })
    ->get();
```

<a name="insert-statements"></a>
## Instrukcje Insert

The query builder also provides an `insert` method that may be used to insert records into the database table. The `insert` method accepts an array of column names and values:

```php
DB::table('users')->insert([
    'email' => 'kayla@example.com',
    'votes' => 0
]);
```

You may insert several records at once by passing an array of arrays. Each array represents a record that should be inserted into the table:

```php
DB::table('users')->insert([
    ['email' => 'picard@example.com', 'votes' => 0],
    ['email' => 'janeway@example.com', 'votes' => 0],
]);
```

The `insertOrIgnore` method will ignore errors while inserting records into the database. When using this method, you should be aware that duplicate record errors will be ignored and other types of errors may also be ignored depending on the database engine. For example, `insertOrIgnore` will [bypass MySQL's strict mode](https://dev.mysql.com/doc/refman/en/sql-mode.html#ignore-effect-on-execution):

```php
DB::table('users')->insertOrIgnore([
    ['id' => 1, 'email' => 'sisko@example.com'],
    ['id' => 2, 'email' => 'archer@example.com'],
]);
```

The `insertUsing` method will insert new records into the table while using a subquery to determine the data that should be inserted:

```php
DB::table('pruned_users')->insertUsing([
    'id', 'name', 'email', 'email_verified_at'
], DB::table('users')->select(
    'id', 'name', 'email', 'email_verified_at'
)->where('updated_at', '<=', now()->minus(months: 1)));
```

<a name="auto-incrementing-ids"></a>
#### Automatycznie inkrementowane identyfikatory

If the table has an auto-incrementing id, use the `insertGetId` method to insert a record and then retrieve the ID:

```php
$id = DB::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);
```

> [!WARNING]
> When using PostgreSQL the `insertGetId` method expects the auto-incrementing column to be named `id`. If you would like to retrieve the ID from a different "sequence", you may pass the column name as the second parameter to the `insertGetId` method.

<a name="upserts"></a>
### Upserty

The `upsert` method will insert records that do not exist and update the records that already exist with new values that you may specify. The method's first argument consists of the values to insert or update, while the second argument lists the column(s) that uniquely identify records within the associated table. The method's third and final argument is an array of columns that should be updated if a matching record already exists in the database:

```php
DB::table('flights')->upsert(
    [
        ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
        ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
    ],
    ['departure', 'destination'],
    ['price']
);
```

In the example above, Laravel will attempt to insert two records. If a record already exists with the same `departure` and `destination` column values, Laravel will update that record's `price` column.

> [!WARNING]
> All databases except SQL Server require the columns in the second argument of the `upsert` method to have a "primary" or "unique" index. In addition, the MariaDB and MySQL database drivers ignore the second argument of the `upsert` method and always use the "primary" and "unique" indexes of the table to detect existing records.

<a name="update-statements"></a>
## Instrukcje Update

In addition to inserting records into the database, the query builder can also update existing records using the `update` method. The `update` method, like the `insert` method, accepts an array of column and value pairs indicating the columns to be updated. The `update` method returns the number of affected rows. You may constrain the `update` query using `where` clauses:

```php
$affected = DB::table('users')
    ->where('id', 1)
    ->update(['votes' => 1]);
```

<a name="update-or-insert"></a>
#### Aktualizacja lub wstawienie

Sometimes you may want to update an existing record in the database or create it if no matching record exists. In this scenario, the `updateOrInsert` method may be used. The `updateOrInsert` method accepts two arguments: an array of conditions by which to find the record, and an array of column and value pairs indicating the columns to be updated.

The `updateOrInsert` method will attempt to locate a matching database record using the first argument's column and value pairs. If the record exists, it will be updated with the values in the second argument. If the record cannot be found, a new record will be inserted with the merged attributes of both arguments:

```php
DB::table('users')
    ->updateOrInsert(
        ['email' => 'john@example.com', 'name' => 'John'],
        ['votes' => '2']
    );
```

You may provide a closure to the `updateOrInsert` method to customize the attributes that are updated or inserted into the database based on the existence of a matching record:

```php
DB::table('users')->updateOrInsert(
    ['user_id' => $user_id],
    fn ($exists) => $exists ? [
        'name' => $data['name'],
        'email' => $data['email'],
    ] : [
        'name' => $data['name'],
        'email' => $data['email'],
        'marketable' => true,
    ],
);
```

<a name="updating-json-columns"></a>
### Aktualizowanie kolumn JSON

When updating a JSON column, you should use `->` syntax to update the appropriate key in the JSON object. This operation is supported on MariaDB 10.3+, MySQL 5.7+, and PostgreSQL 9.5+:

```php
$affected = DB::table('users')
    ->where('id', 1)
    ->update(['options->enabled' => true]);
```

<a name="increment-and-decrement"></a>
### Inkrementacja i dekrementacja

The query builder also provides convenient methods for incrementing or decrementing the value of a given column. Both of these methods accept at least one argument: the column to modify. A second argument may be provided to specify the amount by which the column should be incremented or decremented:

```php
DB::table('users')->increment('votes');

DB::table('users')->increment('votes', 5);

DB::table('users')->decrement('votes');

DB::table('users')->decrement('votes', 5);
```

If needed, you may also specify additional columns to update during the increment or decrement operation:

```php
DB::table('users')->increment('votes', 1, ['name' => 'John']);
```

In addition, you may increment or decrement multiple columns at once using the `incrementEach` and `decrementEach` methods:

```php
DB::table('users')->incrementEach([
    'votes' => 5,
    'balance' => 100,
]);
```

<a name="delete-statements"></a>
## Instrukcje Delete

The query builder's `delete` method may be used to delete records from the table. The `delete` method returns the number of affected rows. You may constrain `delete` statements by adding "where" clauses before calling the `delete` method:

```php
$deleted = DB::table('users')->delete();

$deleted = DB::table('users')->where('votes', '>', 100)->delete();
```

<a name="pessimistic-locking"></a>
## Blokowanie pesymistyczne

The query builder also includes a few functions to help you achieve "pessimistic locking" when executing your `select` statements. To execute a statement with a "shared lock", you may call the `sharedLock` method. A shared lock prevents the selected rows from being modified until your transaction is committed:

```php
DB::table('users')
    ->where('votes', '>', 100)
    ->sharedLock()
    ->get();
```

Alternatively, you may use the `lockForUpdate` method. A "for update" lock prevents the selected records from being modified or from being selected with another shared lock:

```php
DB::table('users')
    ->where('votes', '>', 100)
    ->lockForUpdate()
    ->get();
```

While not obligatory, it is recommended to wrap pessimistic locks within a [transaction](/docs/{{version}}/database#database-transactions). This ensures that the data retrieved remains unaltered in the database until the entire operation completes. In case of a failure, the transaction will roll back any changes and release the locks automatically:

```php
DB::transaction(function () {
    $sender = DB::table('users')
        ->lockForUpdate()
        ->find(1);

    $receiver = DB::table('users')
        ->lockForUpdate()
        ->find(2);

    if ($sender->balance < 100) {
        throw new RuntimeException('Balance too low.');
    }

    DB::table('users')
        ->where('id', $sender->id)
        ->update([
            'balance' => $sender->balance - 100
        ]);

    DB::table('users')
        ->where('id', $receiver->id)
        ->update([
            'balance' => $receiver->balance + 100
        ]);
});
```

<a name="reusable-query-components"></a>
## Komponenty zapytań wielokrotnego użytku

If you have repeated query logic throughout your application, you may extract the logic into reusable objects using the query builder's `tap` and `pipe` methods. Imagine you have these two different queries in your application:

```php
use Illuminate\Database\Query\Builder;
use Illuminate\Support\Facades\DB;

$destination = $request->query('destination');

DB::table('flights')
    ->when($destination, function (Builder $query, string $destination) {
        $query->where('destination', $destination);
    })
    ->orderByDesc('price')
    ->get();

// ...

$destination = $request->query('destination');

DB::table('flights')
    ->when($destination, function (Builder $query, string $destination) {
        $query->where('destination', $destination);
    })
    ->where('user', $request->user()->id)
    ->orderBy('destination')
    ->get();
```

You may like to extract the destination filtering that is common between the queries into a reusable object:

```php
<?php

namespace App\Scopes;

use Illuminate\Database\Query\Builder;

class DestinationFilter
{
    public function __construct(
        private ?string $destination,
    ) {
        //
    }

    public function __invoke(Builder $query): void
    {
        $query->when($this->destination, function (Builder $query) {
            $query->where('destination', $this->destination);
        });
    }
}
```

Then, you can use the query builder's `tap` method to apply the object's logic to the query:

```php
use App\Scopes\DestinationFilter;
use Illuminate\Database\Query\Builder;
use Illuminate\Support\Facades\DB;

DB::table('flights')
    ->when($destination, function (Builder $query, string $destination) { // [tl! remove]
        $query->where('destination', $destination); // [tl! remove]
    }) // [tl! remove]
    ->tap(new DestinationFilter($destination)) // [tl! add]
    ->orderByDesc('price')
    ->get();

// ...

DB::table('flights')
    ->when($destination, function (Builder $query, string $destination) { // [tl! remove]
        $query->where('destination', $destination); // [tl! remove]
    }) // [tl! remove]
    ->tap(new DestinationFilter($destination)) // [tl! add]
    ->where('user', $request->user()->id)
    ->orderBy('destination')
    ->get();
```

<a name="query-pipes"></a>
#### Potoki zapytań

The `tap` method will always return the query builder. If you would like to extract an object that executes the query and returns another value, you may use the `pipe` method instead.

Consider the following query object that contains shared [pagination](/docs/{{version}}/pagination) logic used throughout an application. Unlike the `DestinationFilter`, which applies query conditions to the query, the `Paginate` object executes the query and returns a paginator instance:

```php
<?php

namespace App\Scopes;

use Illuminate\Contracts\Pagination\LengthAwarePaginator;
use Illuminate\Database\Query\Builder;

class Paginate
{
    public function __construct(
        private string $sortBy = 'timestamp',
        private string $sortDirection = 'desc',
        private int $perPage = 25,
    ) {
        //
    }

    public function __invoke(Builder $query): LengthAwarePaginator
    {
        return $query->orderBy($this->sortBy, $this->sortDirection)
            ->paginate($this->perPage, pageName: 'p');
    }
}
```

Using the query builder's `pipe` method, we can leverage this object to apply our shared pagination logic:

```php
$flights = DB::table('flights')
    ->tap(new DestinationFilter($destination))
    ->pipe(new Paginate);
```

<a name="debugging"></a>
## Debugowanie

You may use the `dd` and `dump` methods while building a query to dump the current query bindings and SQL. The `dd` method will display the debug information and then stop executing the request. The `dump` method will display the debug information but allow the request to continue executing:

```php
DB::table('users')->where('votes', '>', 100)->dd();

DB::table('users')->where('votes', '>', 100)->dump();
```

The `dumpRawSql` and `ddRawSql` methods may be invoked on a query to dump the query's SQL with all parameter bindings properly substituted:

```php
DB::table('users')->where('votes', '>', 100)->dumpRawSql();

DB::table('users')->where('votes', '>', 100)->ddRawSql();
```
