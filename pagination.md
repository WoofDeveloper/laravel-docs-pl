# Baza Danych: Paginacja

- [Wprowadzenie](#introduction)
- [Podstawowe Użycie](#basic-usage)
    - [Paginacja Wyników Query Builder](#paginating-query-builder-results)
    - [Paginacja Wyników Eloquent](#paginating-eloquent-results)
    - [Paginacja Kursorowa](#cursor-pagination)
    - [Ręczne Tworzenie Paginatora](#manually-creating-a-paginator)
    - [Dostosowywanie Adresów URL Paginacji](#customizing-pagination-urls)
- [Wyświetlanie Wyników Paginacji](#displaying-pagination-results)
    - [Dostosowywanie Okna Linków Paginacji](#adjusting-the-pagination-link-window)
    - [Konwersja Wyników do JSON](#converting-results-to-json)
- [Dostosowywanie Widoku Paginacji](#customizing-the-pagination-view)
    - [Używanie Bootstrap](#using-bootstrap)
- [Metody Instancji Paginator i LengthAwarePaginator](#paginator-instance-methods)
- [Metody Instancji Cursor Paginator](#cursor-paginator-instance-methods)

<a name="introduction"></a>
## Wprowadzenie

W innych frameworkach paginacja może być bardzo bolesna. Mamy nadzieję, że podejście Laravel do paginacji będzie świeżym powiewem. Paginator Laravel jest zintegrowany z [konstruktorem zapytań](/docs/{{version}}/queries) i [Eloquent ORM](/docs/{{version}}/eloquent) i zapewnia wygodne, łatwe w użyciu paginowanie rekordów bazy danych bez konieczności konfiguracji.

Domyślnie HTML generowany przez paginator jest kompatybilny z [frameworkiem Tailwind CSS](https://tailwindcss.com/); jednak dostępna jest również obsługa paginacji Bootstrap.

<a name="tailwind"></a>
#### Tailwind

Jeśli używasz domyślnych widoków paginacji Laravel z Tailwind 4.x, plik `resources/css/app.css` Twojej aplikacji będzie już prawidłowo skonfigurowany do `@source` widoków paginacji Laravel:

```css
@import 'tailwindcss';

@source '../../vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php';
```

<a name="basic-usage"></a>
## Podstawowe Użycie

<a name="paginating-query-builder-results"></a>
### Paginacja Wyników Query Builder

Istnieje kilka sposobów paginowania elementów. Najprostszy to użycie metody `paginate` na [konstruktorze zapytań](/docs/{{version}}/queries) lub [zapytaniu Eloquent](/docs/{{version}}/eloquent). Metoda `paginate` automatycznie dba o ustawienie "limit" i "offset" zapytania na podstawie bieżącej strony oglądanej przez użytkownika. Domyślnie bieżąca strona jest wykrywana przez wartość argumentu ciągu zapytania `page` w żądaniu HTTP. Wartość ta jest automatycznie wykrywana przez Laravel i również automatycznie wstawiana do linków generowanych przez paginator.

W tym przykładzie jedynym argumentem przekazanym do metody `paginate` jest liczba elementów, które chcesz wyświetlić "na stronę". W tym przypadku określmy, że chcemy wyświetlić `15` elementów na stronę:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show all application users.
     */
    public function index(): View
    {
        return view('user.index', [
            'users' => DB::table('users')->paginate(15)
        ]);
    }
}
```

<a name="simple-pagination"></a>
#### Prosta Paginacja

Metoda `paginate` liczy całkowitą liczbę rekordów pasujących do zapytania przed pobraniem rekordów z bazy danych. Robi się to po to, aby paginator wiedział, ile całkowitych stron rekordów istnieje. Jednak jeśli nie planujesz pokazywać całkowitej liczby stron w interfejsie użytkownika aplikacji, zapytanie o liczbę rekordów jest niepotrzebne.

Dlatego, jeśli potrzebujesz wyświetlić tylko proste linki "Następna" i "Poprzednia" w interfejsie użytkownika aplikacji, możesz użyć metody `simplePaginate`, aby wykonać pojedyncze, wydajne zapytanie:

```php
$users = DB::table('users')->simplePaginate(15);
```

<a name="paginating-eloquent-results"></a>
### Paginacja Wyników Eloquent

Możesz również paginować zapytania [Eloquent](/docs/{{version}}/eloquent). W tym przykładzie będziemy paginować model `App\Models\User` i wskazać, że planujemy wyświetlić 15 rekordów na stronę. Jak widać, składnia jest niemal identyczna z paginacją wyników konstruktora zapytań:

```php
use App\Models\User;

$users = User::paginate(15);
```

Oczywiście możesz wywołać metodę `paginate` po ustawieniu innych ograniczeń zapytania, takich jak klauzule `where`:

```php
$users = User::where('votes', '>', 100)->paginate(15);
```

Możesz również użyć metody `simplePaginate` podczas paginacji modeli Eloquent:

```php
$users = User::where('votes', '>', 100)->simplePaginate(15);
```

Podobnie, możesz użyć metody `cursorPaginate`, aby wykonać paginację kursorową modeli Eloquent:

```php
$users = User::where('votes', '>', 100)->cursorPaginate(15);
```

<a name="multiple-paginator-instances-per-page"></a>
#### Wiele Instancji Paginatora na Stronę

Czasami możesz potrzebować wyrenderować dwa oddzielne paginatory na jednym ekranie renderowanym przez Twoją aplikację. Jednak jeśli obie instancje paginatora używają parametru ciągu zapytania `page` do przechowywania bieżącej strony, dwa paginatory będą ze sobą kolidować. Aby rozwiązać ten konflikt, możesz przekazać nazwę parametru ciągu zapytania, którego chcesz użyć do przechowywania bieżącej strony paginatora przez trzeci argument dostarczany do metod `paginate`, `simplePaginate` i `cursorPaginate`:

```php
use App\Models\User;

$users = User::where('votes', '>', 100)->paginate(
    $perPage = 15, $columns = ['*'], $pageName = 'users'
);
```

<a name="cursor-pagination"></a>
### Paginacja Kursorowa

Podczas gdy `paginate` i `simplePaginate` tworzą zapytania używając klauzuli SQL "offset", paginacja kursorowa działa poprzez konstruowanie klauzul "where", które porównują wartości uporządkowanych kolumn zawartych w zapytaniu, zapewniając najwydajniejszą wydajność bazy danych spośród wszystkich metod paginacji Laravel. Ta metoda paginacji jest szczególnie odpowiednia dla dużych zbiorów danych i interfejsów użytkownika z "nieskończonym" przewijaniem.

W przeciwieństwie do paginacji opartej na offset, która zawiera numer strony w ciągu zapytania adresów URL generowanych przez paginator, paginacja oparta na kursorze umieszcza ciąg "cursor" w ciągu zapytania. Kursor jest zakodowanym ciągiem zawierającym lokalizację, od której następne zapytanie paginacji powinno zacząć paginację oraz kierunek, w którym powinno paginować:

```text
http://localhost/users?cursor=eyJpZCI6MTUsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX0
```

Możesz utworzyć instancję paginatora opartego na kursorze za pomocą metody `cursorPaginate` oferowanej przez konstruktor zapytań. Ta metoda zwraca instancję `Illuminate\Pagination\CursorPaginator`:

```php
$users = DB::table('users')->orderBy('id')->cursorPaginate(15);
```

Po pobraniu instancji paginatora kursorowego możesz [wyświetlić wyniki paginacji](#displaying-pagination-results) tak, jak zazwyczaj robiłbyś to używając metod `paginate` i `simplePaginate`. Więcej informacji o metodach instancji oferowanych przez paginator kursorowy można znaleźć w [dokumentacji metod instancji paginatora kursorowego](#cursor-paginator-instance-methods).

> [!WARNING]
> Twoje zapytanie musi zawierać klauzulę "order by", aby skorzystać z paginacji kursorowej. Dodatkowo, kolumny, według których zapytanie jest sortowane, muszą należeć do tabeli, którą paginujesz.

<a name="cursor-vs-offset-pagination"></a>
#### Paginacja Kursorowa vs. Paginacja Offsetowa

Aby zilustrować różnice między paginacją offsetową a paginacją kursorową, zbadajmy przykładowe zapytania SQL. Oba z poniższych zapytań wyświetlą "drugą stronę" wyników dla tabeli `users` uporządkowanej według `id`:

```sql
# Offset Pagination...
select * from users order by id asc limit 15 offset 15;

# Cursor Pagination...
select * from users where id > 15 order by id asc limit 15;
```

Zapytanie z paginacją kursorową oferuje następujące zalety w porównaniu z paginacją offsetową:

- W przypadku dużych zbiorów danych paginacja kursorowa zapewni lepsze wydajności, jeśli kolumny "order by" są indeksowane. Dzieje się tak, ponieważ klauzula "offset" przeszukuje wszystkie wcześniej dopasowane dane.
- W przypadku zbiorów danych z częstymi zapisami, paginacja offsetowa może pomijać rekordy lub pokazywać duplikaty, jeśli wyniki zostały niedawno dodane lub usunięte ze strony, którą użytkownik aktualnie ogląda.

Jednak paginacja kursorowa ma następujące ograniczenia:

- Podobnie jak `simplePaginate`, paginacja kursorowa może być używana tylko do wyświetlania linków "Następna" i "Poprzednia" i nie obsługuje generowania linków z numerami stron.
- Wymaga, aby sortowanie było oparte na co najmniej jednej unikalnej kolumnie lub kombinacji kolumn, które są unikalne. Kolumny z wartościami `null` nie są obsługiwane.
- Wyrażenia zapytań w klauzulach "order by" są obsługiwane tylko wtedy, gdy mają alias i zostały dodane również do klauzuli "select".
- Wyrażenia zapytań z parametrami nie są obsługiwane.

<a name="manually-creating-a-paginator"></a>
### Ręczne Tworzenie Paginatora

Czasami możesz chcieć utworzyć instancję paginacji ręcznie, przekazując jej tablicę elementów, które już masz w pamięci. Możesz to zrobić, tworząc instancję `Illuminate\Pagination\Paginator`, `Illuminate\Pagination\LengthAwarePaginator` lub `Illuminate\Pagination\CursorPaginator`, w zależności od Twoich potrzeb.

Klasy `Paginator` i `CursorPaginator` nie muszą znać całkowitej liczby elementów w zestawie wyników; jednak z tego powodu te klasy nie mają metod do pobierania indeksu ostatniej strony. `LengthAwarePaginator` przyjmuje niemal te same argumenty co `Paginator`; jednak wymaga policzenia całkowitej liczby elementów w zestawie wyników.

Innymi słowy, `Paginator` odpowiada metodzie `simplePaginate` konstruktora zapytań, `CursorPaginator` odpowiada metodzie `cursorPaginate`, a `LengthAwarePaginator` odpowiada metodzie `paginate`.

> [!WARNING]
> Podczas ręcznego tworzenia instancji paginatora, powinieneś ręcznie "pokroić" tablicę wyników, którą przekazujesz do paginatora. Jeśli nie jesteś pewien, jak to zrobić, sprawdź funkcję PHP [array_slice](https://secure.php.net/manual/en/function.array-slice.php).

<a name="customizing-pagination-urls"></a>
### Dostosowywanie Adresów URL Paginacji

Domyślnie linki generowane przez paginator będą pasować do URI bieżącego żądania. Jednak metoda `withPath` paginatora pozwala dostosować URI używany przez paginator podczas generowania linków. Na przykład, jeśli chcesz, aby paginator generował linki takie jak `http://example.com/admin/users?page=N`, powinieneś przekazać `/admin/users` do metody `withPath`:

```php
use App\Models\User;

Route::get('/users', function () {
    $users = User::paginate(15);

    $users->withPath('/admin/users');

    // ...
});
```

<a name="appending-query-string-values"></a>
#### Dodawanie Wartości Ciągu Zapytania

Możesz dodać do ciągu zapytania linków paginacji używając metody `appends`. Na przykład, aby dodać `sort=votes` do każdego linku paginacji, powinieneś wykonać następujące wywołanie `appends`:

```php
use App\Models\User;

Route::get('/users', function () {
    $users = User::paginate(15);

    $users->appends(['sort' => 'votes']);

    // ...
});
```

Możesz użyć metody `withQueryString`, jeśli chcesz dodać wszystkie wartości ciągu zapytania bieżącego żądania do linków paginacji:

```php
$users = User::paginate(15)->withQueryString();
```

<a name="appending-hash-fragments"></a>
#### Dodawanie Fragmentów Hasha

Jeśli musisz dodać "fragment hasha" do adresów URL generowanych przez paginator, możesz użyć metody `fragment`. Na przykład, aby dodać `#users` na końcu każdego linku paginacji, powinieneś wywołać metodę `fragment` w następujący sposób:

```php
$users = User::paginate(15)->fragment('users');
```

<a name="displaying-pagination-results"></a>
## Wyświetlanie Wyników Paginacji

Podczas wywoływania metody `paginate` otrzymasz instancję `Illuminate\Pagination\LengthAwarePaginator`, podczas gdy wywołanie metody `simplePaginate` zwraca instancję `Illuminate\Pagination\Paginator`. I wreszcie, wywołanie metody `cursorPaginate` zwraca instancję `Illuminate\Pagination\CursorPaginator`.

Te obiekty udostępniają kilka metod opisujących zestaw wyników. Oprócz tych metod pomocniczych, instancje paginatora są iteratorami i mogą być przetwarzane jako tablica. Więc, po pobraniu wyników, możesz wyświetlić wyniki i renderować linki stron używając [Blade](/docs/{{version}}/blade):

```blade
<div class="container">
    @foreach ($users as $user)
        {{ $user->name }}
    @endforeach
</div>

{{ $users->links() }}
```

Metoda `links` wyrenderuje linki do pozostałych stron w zestawie wyników. Każdy z tych linków będzie już zawierał właściwą zmienną ciągu zapytania `page`. Pamiętaj, że HTML generowany przez metodę `links` jest kompatybilny z [frameworkiem Tailwind CSS](https://tailwindcss.com).

<a name="adjusting-the-pagination-link-window"></a>
### Dostosowywanie Okna Linków Paginacji

Kiedy paginator wyświetla linki paginacji, numer bieżącej strony jest wyświetlany wraz z linkami do trzech stron przed i po bieżącej stronie. Używając metody `onEachSide`, możesz kontrolować, ile dodatkowych linków jest wyświetlanych po każdej stronie bieżącej strony w środkowym, przesuwnym oknie linków generowanych przez paginator:

```blade
{{ $users->onEachSide(5)->links() }}
```

<a name="converting-results-to-json"></a>
### Konwersja Wyników do JSON

Klasy paginatora Laravel implementują kontrakt interfejsu `Illuminate\Contracts\Support\Jsonable` i udostępniają metodę `toJson`, więc bardzo łatwo jest przekonwertować wyniki paginacji do JSON. Możesz również przekonwertować instancję paginatora do JSON, zwracając ją z trasy lub akcji kontrolera:

```php
use App\Models\User;

Route::get('/users', function () {
    return User::paginate();
});
```

JSON z paginatora będzie zawierał meta informacje takie jak `total`, `current_page`, `last_page` i więcej. Rekordy wyników są dostępne poprzez klucz `data` w tablicy JSON. Oto przykład JSON utworzonego przez zwrócenie instancji paginatora z trasy:

```json
{
   "total": 50,
   "per_page": 15,
   "current_page": 1,
   "last_page": 4,
   "current_page_url": "http://laravel.app?page=1",
   "first_page_url": "http://laravel.app?page=1",
   "last_page_url": "http://laravel.app?page=4",
   "next_page_url": "http://laravel.app?page=2",
   "prev_page_url": null,
   "path": "http://laravel.app",
   "from": 1,
   "to": 15,
   "data":[
        {
            // Record...
        },
        {
            // Record...
        }
   ]
}
```

<a name="customizing-the-pagination-view"></a>
## Dostosowywanie Widoku Paginacji

Domyślnie widoki renderowane do wyświetlania linków paginacji są kompatybilne z frameworkiem [Tailwind CSS](https://tailwindcss.com). Jednak jeśli nie używasz Tailwind, możesz swobodnie zdefiniować własne widoki do renderowania tych linków. Podczas wywoływania metody `links` na instancji paginatora, możesz przekazać nazwę widoku jako pierwszy argument metody:

```blade
{{ $paginator->links('view.name') }}

<!-- Passing additional data to the view... -->
{{ $paginator->links('view.name', ['foo' => 'bar']) }}
```

Jednak najłatwiejszym sposobem dostosowania widoków paginacji jest wyeksportowanie ich do katalogu `resources/views/vendor` używając polecenia `vendor:publish`:

```shell
php artisan vendor:publish --tag=laravel-pagination
```

To polecenie umieści widoki w katalogu `resources/views/vendor/pagination` Twojej aplikacji. Plik `tailwind.blade.php` w tym katalogu odpowiada domyślnemu widokowi paginacji. Możesz edytować ten plik, aby zmodyfikować HTML paginacji.

Jeśli chciałbyś wyznaczyć inny plik jako domyślny widok paginacji, możesz wywołać metody `defaultView` i `defaultSimpleView` paginatora w metodzie `boot` Twojej klasy `App\Providers\AppServiceProvider`:

```php
<?php

namespace App\Providers;

use Illuminate\Pagination\Paginator;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Paginator::defaultView('view-name');

        Paginator::defaultSimpleView('view-name');
    }
}
```

<a name="using-bootstrap"></a>
### Używanie Bootstrap

Laravel zawiera widoki paginacji zbudowane przy użyciu [Bootstrap CSS](https://getbootstrap.com/). Aby użyć tych widoków zamiast domyślnych widoków Tailwind, możesz wywołać metody `useBootstrapFour` lub `useBootstrapFive` paginatora w metodzie `boot` Twojej klasy `App\Providers\AppServiceProvider`:

```php
use Illuminate\Pagination\Paginator;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Paginator::useBootstrapFive();
    Paginator::useBootstrapFour();
}
```

<a name="paginator-instance-methods"></a>
## Metody Instancji Paginator / LengthAwarePaginator

Każda instancja paginatora dostarcza dodatkowych informacji o paginacji za pośrednictwem następujących metod:

<div class="overflow-auto">

| Metoda                                  | Opis                                                                                                         |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `$paginator->count()`                   | Pobierz liczbę elementów dla bieżącej strony.                                                                |
| `$paginator->currentPage()`             | Pobierz numer bieżącej strony.                                                                                 |
| `$paginator->firstItem()`               | Pobierz numer wyniku pierwszego elementu w wynikach.                                                      |
| `$paginator->getOptions()`              | Pobierz opcje paginatora.                                                                                   |
| `$paginator->getUrlRange($start, $end)` | Utwórz zakres adresów URL paginacji.                                                                           |
| `$paginator->hasPages()`                | Określ, czy jest wystarczająco dużo elementów, aby podzielić je na wiele stron.                                            |
| `$paginator->hasMorePages()`            | Określ, czy jest więcej elementów w magazynie danych.                                                         |
| `$paginator->items()`                   | Pobierz elementy dla bieżącej strony.                                                                          |
| `$paginator->lastItem()`                | Pobierz numer wyniku ostatniego elementu w wynikach.                                                       |
| `$paginator->lastPage()`                | Pobierz numer strony ostatniej dostępnej strony. (Niedostępne przy użyciu `simplePaginate`).                 |
| `$paginator->nextPageUrl()`             | Pobierz adres URL następnej strony.                                                                               |
| `$paginator->onFirstPage()`             | Określ, czy paginator jest na pierwszej stronie.                                                             |
| `$paginator->onLastPage()`              | Określ, czy paginator jest na ostatniej stronie.                                                              |
| `$paginator->perPage()`                 | Liczba elementów do wyświetlenia na stronę.                                                                    |
| `$paginator->previousPageUrl()`         | Pobierz adres URL poprzedniej strony.                                                                           |
| `$paginator->total()`                   | Określ całkowitą liczbę pasujących elementów w magazynie danych. (Niedostępne przy użyciu `simplePaginate`). |
| `$paginator->url($page)`                | Pobierz adres URL dla danego numeru strony.                                                                         |
| `$paginator->getPageName()`             | Pobierz zmienną ciągu zapytania używaną do przechowywania strony.                                                        |
| `$paginator->setPageName($name)`        | Ustaw zmienną ciągu zapytania używaną do przechowywania strony.                                                        |
| `$paginator->through($callback)`        | Przekształć każdy element przy użyciu callbacku.                                                                        |

</div>

<a name="cursor-paginator-instance-methods"></a>
## Metody Instancji Cursor Paginator

Każda instancja paginatora kursorowego dostarcza dodatkowych informacji o paginacji za pośrednictwem następujących metod:

<div class="overflow-auto">

| Metoda                          | Opis                                                       |
| ------------------------------- | ----------------------------------------------------------------- |
| `$paginator->count()`           | Pobierz liczbę elementów dla bieżącej strony.                     |
| `$paginator->cursor()`          | Pobierz bieżącą instancję kursora.                                  |
| `$paginator->getOptions()`      | Pobierz opcje paginatora.                                        |
| `$paginator->hasPages()`        | Określ, czy jest wystarczająco dużo elementów, aby podzielić je na wiele stron. |
| `$paginator->hasMorePages()`    | Określ, czy jest więcej elementów w magazynie danych.              |
| `$paginator->getCursorName()`   | Pobierz zmienną ciągu zapytania używaną do przechowywania kursora.           |
| `$paginator->items()`           | Pobierz elementy dla bieżącej strony.                               |
| `$paginator->nextCursor()`      | Pobierz instancję kursora dla następnego zestawu elementów.                |
| `$paginator->nextPageUrl()`     | Pobierz adres URL następnej strony.                                    |
| `$paginator->onFirstPage()`     | Określ, czy paginator jest na pierwszej stronie.                  |
| `$paginator->onLastPage()`      | Określ, czy paginator jest na ostatniej stronie.                   |
| `$paginator->perPage()`         | Liczba elementów do wyświetlenia na stronę.                         |
| `$paginator->previousCursor()`  | Pobierz instancję kursora dla poprzedniego zestawu elementów.            |
| `$paginator->previousPageUrl()` | Pobierz adres URL poprzedniej strony.                                |
| `$paginator->setCursorName()`   | Ustaw zmienną ciągu zapytania używaną do przechowywania kursora.           |
| `$paginator->url($cursor)`      | Pobierz adres URL dla danej instancji kursora.                          |

</div>
