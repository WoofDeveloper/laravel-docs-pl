# Kolekcje

- [Wprowadzenie](#introduction)
    - [Tworzenie kolekcji](#creating-collections)
    - [Rozszerzanie kolekcji](#extending-collections)
- [Dostępne metody](#available-methods)
- [Komunikaty wyższego rzędu](#higher-order-messages)
- [Leniwe kolekcje](#lazy-collections)
    - [Wprowadzenie](#lazy-collection-introduction)
    - [Tworzenie leniwych kolekcji](#creating-lazy-collections)
    - [Kontrakt Enumerable](#the-enumerable-contract)
    - [Metody leniwych kolekcji](#lazy-collection-methods)

<a name="introduction"></a>
## Wprowadzenie

Klasa `Illuminate\Support\Collection` zapewnia płynną, wygodną otoczkę do pracy z tablicami danych. Na przykład, spójrz na poniższy kod. Użyjemy pomocnika `collect` do utworzenia nowej instancji kolekcji z tablicy, uruchomimy funkcję `strtoupper` na każdym elemencie, a następnie usuniemy wszystkie puste elementy:

```php
$collection = collect(['Taylor', 'Abigail', null])->map(function (?string $name) {
    return strtoupper($name);
})->reject(function (string $name) {
    return empty($name);
});
```

Jak widzisz, klasa `Collection` pozwala na łączenie jej metod w łańcuch w celu wykonania płynnego mapowania i redukcji tablicy bazowej. Ogólnie rzecz biorąc, kolekcje są niezmienne, co oznacza, że każda metoda `Collection` zwraca całkowicie nową instancję `Collection`.

<a name="creating-collections"></a>
### Tworzenie kolekcji

Jak wspomniano powyżej, helper `collect` zwraca nową instancję `Illuminate\Support\Collection` dla podanej tablicy. Więc tworzenie kolekcji jest tak proste jak:

```php
$collection = collect([1, 2, 3]);
```

Możesz również utworzyć kolekcję używając metod [make](#method-make) i [fromJson](#method-fromjson).

> [!NOTE]
> Wyniki zapytań [Eloquent](/docs/{{version}}/eloquent) są zawsze zwracane jako instancje `Collection`.

<a name="extending-collections"></a>
### Rozszerzanie kolekcji

Kolekcje są "makrowalme", co pozwala na dodanie dodatkowych metod do klasy `Collection` w czasie wykonywania. Metoda `macro` klasy `Illuminate\Support\Collection` akceptuje zamknięcie, które zostanie wykonane, gdy Twoje makro zostanie wywołane. Zamknięcie makra może uzyskać dostęp do innych metod kolekcji poprzez `$this`, tak jakby była to prawdziwa metoda klasy kolekcji. Na przykład poniższy kod dodaje metodę `toUpper` do klasy `Collection`:

```php
use Illuminate\Support\Collection;
use Illuminate\Support\Str;

Collection::macro('toUpper', function () {
    return $this->map(function (string $value) {
        return Str::upper($value);
    });
});

$collection = collect(['first', 'second']);

$upper = $collection->toUpper();

// ['FIRST', 'SECOND']
```

Zazwyczaj powinieneś deklarować makra kolekcji w metodzie `boot` [dostawcy usług](/docs/{{version}}/providers).

<a name="macro-arguments"></a>
#### Argumenty makr

W razie potrzeby możesz zdefiniować makra, które akceptują dodatkowe argumenty:

```php
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\Lang;

Collection::macro('toLocale', function (string $locale) {
    return $this->map(function (string $value) use ($locale) {
        return Lang::get($value, [], $locale);
    });
});

$collection = collect(['first', 'second']);

$translated = $collection->toLocale('es');

// ['primero', 'segundo'];
```

<a name="available-methods"></a>
## Dostępne metody

W większości pozostałej dokumentacji kolekcji omówimy każdą metodę dostępną w klasie `Collection`. Pamiętaj, że wszystkie te metody mogą być łączone w łańcuch, aby płynnie manipulować tablicą bazową. Ponadto prawie każda metoda zwraca nową instancję `Collection`, co pozwala na zachowanie oryginalnej kopii kolekcji w razie potrzeby:

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
</style>

<div class="collection-method-list" markdown="1">

[after](#method-after)
[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[before](#method-before)
[chunk](#method-chunk)
[chunkWhile](#method-chunkwhile)
[collapse](#method-collapse)
[collapseWithKeys](#method-collapsewithkeys)
[collect](#method-collect)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsManyItems](#method-containsmanyitems)
[containsStrict](#method-containsstrict)
[count](#method-count)
[countBy](#method-countBy)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffAssocUsing](#method-diffassocusing)
[diffKeys](#method-diffkeys)
[doesntContain](#method-doesntcontain)
[doesntContainStrict](#method-doesntcontainstrict)
[dot](#method-dot)
[dump](#method-dump)
[duplicates](#method-duplicates)
[duplicatesStrict](#method-duplicatesstrict)
[each](#method-each)
[eachSpread](#method-eachspread)
[ensure](#method-ensure)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[firstOrFail](#method-first-or-fail)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[fromJson](#method-fromjson)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[hasAny](#method-hasany)
[hasSole](#method-hassole)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectUsing](#method-intersectusing)
[intersectAssoc](#method-intersectAssoc)
[intersectAssocUsing](#method-intersectassocusing)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[join](#method-join)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[lazy](#method-lazy)
[macro](#method-macro)
[make](#method-make)
[map](#method-map)
[mapInto](#method-mapinto)
[mapSpread](#method-mapspread)
[mapToGroups](#method-maptogroups)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[mergeRecursive](#method-mergerecursive)
[min](#method-min)
[mode](#method-mode)
[multiply](#method-multiply)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[percentage](#method-percentage)
[pipe](#method-pipe)
[pipeInto](#method-pipeinto)
[pipeThrough](#method-pipethrough)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[range](#method-range)
[reduce](#method-reduce)
[reduceSpread](#method-reduce-spread)
[reject](#method-reject)
[replace](#method-replace)
[replaceRecursive](#method-replacerecursive)
[reverse](#method-reverse)
[search](#method-search)
[select](#method-select)
[shift](#method-shift)
[shuffle](#method-shuffle)
[skip](#method-skip)
[skipUntil](#method-skipuntil)
[skipWhile](#method-skipwhile)
[slice](#method-slice)
[sliding](#method-sliding)
[sole](#method-sole)
[some](#method-some)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[sortDesc](#method-sortdesc)
[sortKeys](#method-sortkeys)
[sortKeysDesc](#method-sortkeysdesc)
[sortKeysUsing](#method-sortkeysusing)
[splice](#method-splice)
[split](#method-split)
[splitIn](#method-splitin)
[sum](#method-sum)
[take](#method-take)
[takeUntil](#method-takeuntil)
[takeWhile](#method-takewhile)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[toPrettyJson](#method-to-pretty-json)
[transform](#method-transform)
[undot](#method-undot)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unlessEmpty](#method-unlessempty)
[unlessNotEmpty](#method-unlessnotempty)
[unwrap](#method-unwrap)
[value](#method-value)
[values](#method-values)
[when](#method-when)
[whenEmpty](#method-whenempty)
[whenNotEmpty](#method-whennotempty)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereBetween](#method-wherebetween)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereInstanceOf](#method-whereinstanceof)
[whereNotBetween](#method-wherenotbetween)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[whereNotNull](#method-wherenotnull)
[whereNull](#method-wherenull)
[wrap](#method-wrap)
[zip](#method-zip)

</div>

<a name="method-listing"></a>
## Lista metod

<style>
    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="method-after"></a>
#### `after()` {.collection-method .first-collection-method}

Metoda `after` zwraca element po podanym elemencie. Zwracany jest `null`, jeśli podany element nie zostanie znaleziony lub jest ostatnim elementem:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->after(3);

// 4

$collection->after(5);

// null
```

Ta metoda wyszukuje podany element używając "luźnego" porównania, co oznacza, że ciąg znaków zawierający wartość całkowitą będzie uważany za równy liczbie całkowitej o tej samej wartości. Aby użyć "ścisłego" porównania, możesz przekazać argument `strict` do metody:

```php
collect([2, 4, 6, 8])->after('4', strict: true);

// null
```

Alternatywnie możesz przekazać własne zamknięcie, aby wyszukać pierwszy element, który przejdzie dany test prawdziwości:

```php
collect([2, 4, 6, 8])->after(function (int $item, int $key) {
    return $item > 5;
});

// 8
```

<a name="method-all"></a>
#### `all()` {.collection-method}

Metoda `all` zwraca tablicę bazową reprezentowaną przez kolekcję:

```php
collect([1, 2, 3])->all();

// [1, 2, 3]
```

<a name="method-average"></a>
#### `average()` {.collection-method}

Alias dla metody [avg](#method-avg).

<a name="method-avg"></a>
#### `avg()` {.collection-method}

Metoda `avg` zwraca [średnią wartość](https://en.wikipedia.org/wiki/Average) dla podanego klucza:

```php
$average = collect([
    ['foo' => 10],
    ['foo' => 10],
    ['foo' => 20],
    ['foo' => 40]
])->avg('foo');

// 20

$average = collect([1, 1, 2, 4])->avg();

// 2
```

<a name="method-before"></a>
#### `before()` {.collection-method}

Metoda `before` jest przeciwieństwem metody [after](#method-after). Zwraca element przed podanym elementem. Zwracany jest `null`, jeśli podany element nie zostanie znaleziony lub jest pierwszym elementem:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->before(3);

// 2

$collection->before(1);

// null

collect([2, 4, 6, 8])->before('4', strict: true);

// null

collect([2, 4, 6, 8])->before(function (int $item, int $key) {
    return $item > 5;
});

// 4
```

<a name="method-chunk"></a>
#### `chunk()` {.collection-method}

Metoda `chunk` dzieli kolekcję na wiele mniejszych kolekcji o podanym rozmiarze:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7]);

$chunks = $collection->chunk(4);

$chunks->all();

// [[1, 2, 3, 4], [5, 6, 7]]
```

Ta metoda jest szczególnie przydatna w [widokach](/docs/{{version}}/views) podczas pracy z systemem siatki, takim jak [Bootstrap](https://getbootstrap.com/docs/5.3/layout/grid/). Na przykład wyobraź sobie, że masz kolekcję modeli [Eloquent](/docs/{{version}}/eloquent), które chcesz wyświetlić w siatce:

```blade
@foreach ($products->chunk(3) as $chunk)
    <div class="row">
        @foreach ($chunk as $product)
            <div class="col-xs-4">{{ $product->name }}</div>
        @endforeach
    </div>
@endforeach
```

<a name="method-chunkwhile"></a>
#### `chunkWhile()` {.collection-method}

Metoda `chunkWhile` dzieli kolekcję na wiele mniejszych kolekcji w oparciu o ocenę podanego callbacku. Zmienna `$chunk` przekazana do zamknięcia może być użyta do sprawdzenia poprzedniego elementu:

```php
$collection = collect(str_split('AABBCCCD'));

$chunks = $collection->chunkWhile(function (string $value, int $key, Collection $chunk) {
    return $value === $chunk->last();
});

$chunks->all();

// [['A', 'A'], ['B', 'B'], ['C', 'C', 'C'], ['D']]
```

<a name="method-collapse"></a>
#### `collapse()` {.collection-method}

Metoda `collapse` zwija kolekcję tablic lub kolekcji w jedną, płaską kolekcję:

```php
$collection = collect([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]);

$collapsed = $collection->collapse();

$collapsed->all();

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

<a name="method-collapsewithkeys"></a>
#### `collapseWithKeys()` {.collection-method}

Metoda `collapseWithKeys` spłaszcza kolekcję tablic lub kolekcji w jedną kolekcję, zachowując oryginalne klucze. Jeśli kolekcja jest już płaska, zwróci pustą kolekcję:

```php
$collection = collect([
    ['first'  => collect([1, 2, 3])],
    ['second' => [4, 5, 6]],
    ['third'  => collect([7, 8, 9])]
]);

$collapsed = $collection->collapseWithKeys();

$collapsed->all();

// [
//     'first'  => [1, 2, 3],
//     'second' => [4, 5, 6],
//     'third'  => [7, 8, 9],
// ]
```

<a name="method-collect"></a>
#### `collect()` {.collection-method}

Metoda `collect` zwraca nową instancję `Collection` z elementami aktualnie znajdującymi się w kolekcji:

```php
$collectionA = collect([1, 2, 3]);

$collectionB = $collectionA->collect();

$collectionB->all();

// [1, 2, 3]
```

Metoda `collect` jest przede wszystkim przydatna do konwersji [leniwych kolekcji](#lazy-collections) na standardowe instancje `Collection`:

```php
$lazyCollection = LazyCollection::make(function () {
    yield 1;
    yield 2;
    yield 3;
});

$collection = $lazyCollection->collect();

$collection::class;

// 'Illuminate\Support\Collection'

$collection->all();

// [1, 2, 3]
```

> [!NOTE]
> Metoda `collect` jest szczególnie przydatna, gdy masz instancję `Enumerable` i potrzebujesz instancji kolekcji nie-leniwej. Ponieważ `collect()` jest częścią kontraktu `Enumerable`, możesz bezpiecznie jej użyć, aby uzyskać instancję `Collection`.

<a name="method-combine"></a>
#### `combine()` {.collection-method}

Metoda `combine` łączy wartości kolekcji jako klucze z wartościami innej tablicy lub kolekcji:

```php
$collection = collect(['name', 'age']);

$combined = $collection->combine(['George', 29]);

$combined->all();

// ['name' => 'George', 'age' => 29]
```

<a name="method-concat"></a>
#### `concat()` {.collection-method}

Metoda `concat` dodaje wartości podanej tablicy lub kolekcji na koniec innej kolekcji:

```php
$collection = collect(['John Doe']);

$concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

$concatenated->all();

// ['John Doe', 'Jane Doe', 'Johnny Doe']
```

Metoda `concat` numerycznie reindeksuje klucze dla elementów połączonych z oryginalną kolekcją. Aby zachować klucze w kolekcjach asocjacyjnych, zobacz metodę [merge](#method-merge).

<a name="method-contains"></a>
#### `contains()` {.collection-method}

Metoda `contains` określa, czy kolekcja zawiera podany element. Możesz przekazać zamknięcie do metody `contains`, aby określić, czy element istnieje w kolekcji spełniając dany test prawdziwości:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->contains(function (int $value, int $key) {
    return $value > 5;
});

// false
```

Alternatywnie możesz przekazać ciąg znaków do metody `contains`, aby określić, czy kolekcja zawiera podaną wartość elementu:

```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

$collection->contains('Desk');

// true

$collection->contains('New York');

// false
```

Możesz również przekazać parę klucz/wartość do metody `contains`, która określi, czy dana para istnieje w kolekcji:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);

$collection->contains('product', 'Bookcase');

// false
```

Metoda `contains` używa "luźnych" porównań podczas sprawdzania wartości elementów, co oznacza, że ciąg znaków z wartością całkowitą będzie uważany za równy liczbie całkowitej o tej samej wartości. Użyj metody [containsStrict](#method-containsstrict), aby filtrować używając "ścisłych" porównań.

Aby uzyskać odwrotność `contains`, zobacz metodę [doesntContain](#method-doesntcontain).

<a name="method-containsmanyitems"></a>
#### `containsManyItems()` {.collection-method}

Metoda `containsManyItems` określa, czy kolekcja zawiera wiele elementów:

```php
collect([])->containsManyItems(); // false
collect(['1'])->containsManyItems(); // false
collect(['1', '2'])->containsManyItems(); // true
```

Możesz również przekazać callback, aby określić, czy wiele elementów w kolekcji spełnia dany warunek:

```php
collect([1, 2, 3])->containsManyItems(fn (int $item) => $item > 1);

// true

collect([1, 2, 3])->containsManyItems(fn (int $item) => $item > 5);

// false
```

<a name="method-containsstrict"></a>
#### `containsStrict()` {.collection-method}

Ta metoda ma tę samą sygnaturę co metoda [contains](#method-contains); jednakże wszystkie wartości są porównywane używając "ścisłych" porównań.

> [!NOTE]
> Zachowanie tej metody jest modyfikowane podczas używania [kolekcji Eloquent](/docs/{{version}}/eloquent-collections#method-contains).

<a name="method-count"></a>
#### `count()` {.collection-method}

Metoda `count` zwraca całkowitą liczbę elementów w kolekcji:

```php
$collection = collect([1, 2, 3, 4]);

$collection->count();

// 4
```

<a name="method-countBy"></a>
#### `countBy()` {.collection-method}

Metoda `countBy` zlicza wystąpienia wartości w kolekcji. Domyślnie metoda zlicza wystąpienia każdego elementu, pozwalając na zliczanie określonych "typów" elementów w kolekcji:

```php
$collection = collect([1, 2, 2, 2, 3]);

$counted = $collection->countBy();

$counted->all();

// [1 => 1, 2 => 3, 3 => 1]
```

Możesz przekazać zamknięcie do metody `countBy`, aby zliczyć wszystkie elementy według niestandardowej wartości:

```php
$collection = collect(['alice@gmail.com', 'bob@yahoo.com', 'carlos@gmail.com']);

$counted = $collection->countBy(function (string $email) {
    return substr(strrchr($email, '@'), 1);
});

$counted->all();

// ['gmail.com' => 2, 'yahoo.com' => 1]
```

<a name="method-crossjoin"></a>
#### `crossJoin()` {.collection-method}

Metoda `crossJoin` wykonuje iloczyn kartezjański wartości kolekcji pośród podanych tablic lub kolekcji, zwracając iloczyn kartezjański ze wszystkimi możliwymi permutacjami:

```php
$collection = collect([1, 2]);

$matrix = $collection->crossJoin(['a', 'b']);

$matrix->all();

/*
    [
        [1, 'a'],
        [1, 'b'],
        [2, 'a'],
        [2, 'b'],
    ]
*/

$collection = collect([1, 2]);

$matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);

$matrix->all();

/*
    [
        [1, 'a', 'I'],
        [1, 'a', 'II'],
        [1, 'b', 'I'],
        [1, 'b', 'II'],
        [2, 'a', 'I'],
        [2, 'a', 'II'],
        [2, 'b', 'I'],
        [2, 'b', 'II'],
    ]
*/
```

<a name="method-dd"></a>
#### `dd()` {.collection-method}

Metoda `dd` zrzuca elementy kolekcji i kończy wykonywanie skryptu:

```php
$collection = collect(['John Doe', 'Jane Doe']);

$collection->dd();

/*
    array:2 [
        0 => "John Doe"
        1 => "Jane Doe"
    ]
*/
```

Jeśli nie chcesz zatrzymać wykonywania skryptu, użyj zamiast tego metody [dump](#method-dump).

<a name="method-diff"></a>
#### `diff()` {.collection-method}

Metoda `diff` porównuje kolekcję z inną kolekcją lub zwykłą tablicą PHP `array` na podstawie jej wartości. Ta metoda zwróci wartości z oryginalnej kolekcji, które nie są obecne w podanej kolekcji:

```php
$collection = collect([1, 2, 3, 4, 5]);

$diff = $collection->diff([2, 4, 6, 8]);

$diff->all();

// [1, 3, 5]
```

> [!NOTE]
> Zachowanie tej metody jest modyfikowane podczas używania [kolekcji Eloquent](/docs/{{version}}/eloquent-collections#method-diff).

<a name="method-diffassoc"></a>
#### `diffAssoc()` {.collection-method}

Metoda `diffAssoc` porównuje kolekcję z inną kolekcją lub zwykłą tablicą PHP `array` na podstawie jej kluczy i wartości. Ta metoda zwróci pary klucz/wartość z oryginalnej kolekcji, które nie są obecne w podanej kolekcji:

```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6,
]);

$diff = $collection->diffAssoc([
    'color' => 'yellow',
    'type' => 'fruit',
    'remain' => 3,
    'used' => 6,
]);

$diff->all();

// ['color' => 'orange', 'remain' => 6]
```

<a name="method-diffassocusing"></a>
#### `diffAssocUsing()` {.collection-method}

W przeciwieństwie do `diffAssoc`, `diffAssocUsing` akceptuje funkcję callback dostarczoną przez użytkownika do porównywania indeksów:

```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6,
]);

$diff = $collection->diffAssocUsing([
    'Color' => 'yellow',
    'Type' => 'fruit',
    'Remain' => 3,
], 'strnatcasecmp');

$diff->all();

// ['color' => 'orange', 'remain' => 6]
```

Callback musi być funkcją porównującą, która zwraca liczbę całkowitą mniejszą niż, równą lub większą niż zero. Aby uzyskać więcej informacji, zapoznaj się z dokumentacją PHP dotyczącą [array_diff_uassoc](https://www.php.net/array_diff_uassoc#refsect1-function.array-diff-uassoc-parameters), która jest funkcją PHP używaną wewnętrznie przez metodę `diffAssocUsing`.

<a name="method-diffkeys"></a>
#### `diffKeys()` {.collection-method}

Metoda `diffKeys` porównuje kolekcję z inną kolekcją lub zwykłą tablicą PHP `array` na podstawie jej kluczy. Ta metoda zwróci pary klucz/wartość z oryginalnej kolekcji, które nie są obecne w podanej kolekcji:

```php
$collection = collect([
    'one' => 10,
    'two' => 20,
    'three' => 30,
    'four' => 40,
    'five' => 50,
]);

$diff = $collection->diffKeys([
    'two' => 2,
    'four' => 4,
    'six' => 6,
    'eight' => 8,
]);

$diff->all();

// ['one' => 10, 'three' => 30, 'five' => 50]
```

<a name="method-doesntcontain"></a>
#### `doesntContain()` {.collection-method}

Metoda `doesntContain` określa, czy kolekcja nie zawiera podanego elementu. Możesz przekazać zamknięcie do metody `doesntContain`, aby określić, czy element nie istnieje w kolekcji spełniając dany test prawdziwości:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->doesntContain(function (int $value, int $key) {
    return $value < 5;
});

// false
```

Alternatywnie możesz przekazać ciąg znaków do metody `doesntContain`, aby określić, czy kolekcja nie zawiera podanej wartości elementu:

```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

$collection->doesntContain('Table');

// true

$collection->doesntContain('Desk');

// false
```

Możesz również przekazać parę klucz/wartość do metody `doesntContain`, która określi, czy dana para nie istnieje w kolekcji:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);

$collection->doesntContain('product', 'Bookcase');

// true
```

Metoda `doesntContain` używa "luźnych" porównań podczas sprawdzania wartości elementów, co oznacza, że ciąg znaków z wartością całkowitą będzie uważany za równy liczbie całkowitej o tej samej wartości.

<a name="method-doesntcontainstrict"></a>
#### `doesntContainStrict()` {.collection-method}

Ta metoda ma tę samą sygnaturę co metoda [doesntContain](#method-doesntcontain); jednakże wszystkie wartości są porównywane używając "ścisłych" porównań.

<a name="method-dot"></a>
#### `dot()` {.collection-method}

Metoda `dot` spłaszcza wielowymiarową kolekcję w jednopoziomową kolekcję, która używa notacji "kropkowej", aby wskazać głębokość:

```php
$collection = collect(['products' => ['desk' => ['price' => 100]]]);

$flattened = $collection->dot();

$flattened->all();

// ['products.desk.price' => 100]
```

<a name="method-dump"></a>
#### `dump()` {.collection-method}

Metoda `dump` zrzuca elementy kolekcji:

```php
$collection = collect(['John Doe', 'Jane Doe']);

$collection->dump();

/*
    array:2 [
        0 => "John Doe"
        1 => "Jane Doe"
    ]
*/
```

Jeśli chcesz zatrzymać wykonywanie skryptu po zrzucie kolekcji, użyj zamiast tego metody [dd](#method-dd).

<a name="method-duplicates"></a>
#### `duplicates()` {.collection-method}

Metoda `duplicates` pobiera i zwraca zduplikowane wartości z kolekcji:

```php
$collection = collect(['a', 'b', 'a', 'c', 'b']);

$collection->duplicates();

// [2 => 'a', 4 => 'b']
```

Jeśli kolekcja zawiera tablice lub obiekty, możesz przekazać klucz atrybutów, które chcesz sprawdzić pod kątem zduplikowanych wartości:

```php
$employees = collect([
    ['email' => 'abigail@example.com', 'position' => 'Developer'],
    ['email' => 'james@example.com', 'position' => 'Designer'],
    ['email' => 'victoria@example.com', 'position' => 'Developer'],
]);

$employees->duplicates('position');

// [2 => 'Developer']
```

<a name="method-duplicatesstrict"></a>
#### `duplicatesStrict()` {.collection-method}

Ta metoda ma tę samą sygnaturę co metoda [duplicates](#method-duplicates); jednakże wszystkie wartości są porównywane używając "ścisłych" porównań.

<a name="method-each"></a>
#### `each()` {.collection-method}

Metoda `each` iteruje po elementach w kolekcji i przekazuje każdy element do zamknięcia:

```php
$collection = collect([1, 2, 3, 4]);

$collection->each(function (int $item, int $key) {
    // ...
});
```

Jeśli chcesz zatrzymać iterację przez elementy, możesz zwrócić `false` ze swojego zamknięcia:

```php
$collection->each(function (int $item, int $key) {
    if (/* condition */) {
        return false;
    }
});
```

<a name="method-eachspread"></a>
#### `eachSpread()` {.collection-method}

Metoda `eachSpread` iteruje po elementach kolekcji, przekazując każdą zagnieżdżoną wartość elementu do podanego callbacku:

```php
$collection = collect([['John Doe', 35], ['Jane Doe', 33]]);

$collection->eachSpread(function (string $name, int $age) {
    // ...
});
```

Możesz zatrzymać iterację przez elementy, zwracając `false` z callbacku:

```php
$collection->eachSpread(function (string $name, int $age) {
    return false;
});
```

<a name="method-ensure"></a>
#### `ensure()` {.collection-method}

Metoda `ensure` może być użyta do weryfikacji, że wszystkie elementy kolekcji są określonego typu lub listy typów. W przeciwnym razie zostanie rzucony `UnexpectedValueException`:

```php
return $collection->ensure(User::class);

return $collection->ensure([User::class, Customer::class]);
```

Typy prymitywne takie jak `string`, `int`, `float`, `bool` i `array` mogą być również określone:

```php
return $collection->ensure('int');
```

> [!WARNING]
> Metoda `ensure` nie gwarantuje, że elementy różnych typów nie zostaną dodane do kolekcji w późniejszym czasie.

<a name="method-every"></a>
#### `every()` {.collection-method}

Metoda `every` może być użyta do weryfikacji, że wszystkie elementy kolekcji przechodzą dany test prawdziwości:

```php
collect([1, 2, 3, 4])->every(function (int $value, int $key) {
    return $value > 2;
});

// false
```

Jeśli kolekcja jest pusta, metoda `every` zwróci true:

```php
$collection = collect([]);

$collection->every(function (int $value, int $key) {
    return $value > 2;
});

// true
```

<a name="method-except"></a>
#### `except()` {.collection-method}

Metoda `except` zwraca wszystkie elementy w kolekcji z wyjątkiem tych o podanych kluczach:

```php
$collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

$filtered = $collection->except(['price', 'discount']);

$filtered->all();

// ['product_id' => 1]
```

Dla odwrotności `except`, zobacz metodę [only](#method-only).

> [!NOTE]
> This method's behavior is modified when using [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-except).

<a name="method-filter"></a>
#### `filter()` {.collection-method}

Metoda `filter` filtruje kolekcję używając podanego callbacku, zachowując tylko te elementy, które przechodzą dany test prawdziwości:

```php
$collection = collect([1, 2, 3, 4]);

$filtered = $collection->filter(function (int $value, int $key) {
    return $value > 2;
});

$filtered->all();

// [3, 4]
```

Jeśli nie podano callbacku, wszystkie wpisy kolekcji równoważne `false` zostaną usunięte:

```php
$collection = collect([1, 2, 3, null, false, '', 0, []]);

$collection->filter()->all();

// [1, 2, 3]
```

Dla odwrotności `filter`, zobacz metodę [reject](#method-reject).

<a name="method-first"></a>
#### `first()` {.collection-method}

Metoda `first` zwraca pierwszy element w kolekcji, który przechodzi dany test prawdziwości:

```php
collect([1, 2, 3, 4])->first(function (int $value, int $key) {
    return $value > 2;
});

// 3
```

Możesz również wywołać metodę `first` bez argumentów, aby uzyskać pierwszy element w kolekcji. Jeśli kolekcja jest pusta, zwracany jest `null`:

```php
collect([1, 2, 3, 4])->first();

// 1
```

<a name="method-first-or-fail"></a>
#### `firstOrFail()` {.collection-method}

Metoda `firstOrFail` jest identyczna z metodą `first`; jednakże, jeśli nie znaleziono wyniku, zostanie rzucony wyjątek `Illuminate\Support\ItemNotFoundException`:

```php
collect([1, 2, 3, 4])->firstOrFail(function (int $value, int $key) {
    return $value > 5;
});

// Rzuca ItemNotFoundException...
```

Możesz również wywołać metodę `firstOrFail` bez argumentów, aby uzyskać pierwszy element w kolekcji. Jeśli kolekcja jest pusta, zostanie rzucony wyjątek `Illuminate\Support\ItemNotFoundException`:

```php
collect([])->firstOrFail();

// Rzuca ItemNotFoundException...
```

<a name="method-first-where"></a>
#### `firstWhere()` {.collection-method}

Metoda `firstWhere` zwraca pierwszy element w kolekcji z podaną parą klucz/wartość:

```php
$collection = collect([
    ['name' => 'Regena', 'age' => null],
    ['name' => 'Linda', 'age' => 14],
    ['name' => 'Diego', 'age' => 23],
    ['name' => 'Linda', 'age' => 84],
]);

$collection->firstWhere('name', 'Linda');

// ['name' => 'Linda', 'age' => 14]
```

Możesz również wywołać metodę `firstWhere` z operatorem porównania:

```php
$collection->firstWhere('age', '>=', 18);

// ['name' => 'Diego', 'age' => 23]
```

Podobnie jak metoda [where](#method-where), możesz przekazać jeden argument do metody `firstWhere`. W tym scenariuszu metoda `firstWhere` zwróci pierwszy element, gdzie wartość podanego klucza elementu jest "prawdziwa":

```php
$collection->firstWhere('age');

// ['name' => 'Linda', 'age' => 14]
```

<a name="method-flatmap"></a>
#### `flatMap()` {.collection-method}

Metoda `flatMap` iteruje przez kolekcję i przekazuje każdą wartość do podanego zamknięcia. Zamknięcie może swobodnie modyfikować element i go zwrócić, tworząc w ten sposób nową kolekcję zmodyfikowanych elementów. Następnie tablica jest spłaszczana o jeden poziom:

```php
$collection = collect([
    ['name' => 'Sally'],
    ['school' => 'Arkansas'],
    ['age' => 28]
]);

$flattened = $collection->flatMap(function (array $values) {
    return array_map('strtoupper', $values);
});

$flattened->all();

// ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];
```

<a name="method-flatten"></a>
#### `flatten()` {.collection-method}

Metoda `flatten` spłaszcza wielowymiarową kolekcję do jednego wymiaru:

```php
$collection = collect([
    'name' => 'Taylor',
    'languages' => [
        'PHP', 'JavaScript'
    ]
]);

$flattened = $collection->flatten();

$flattened->all();

// ['Taylor', 'PHP', 'JavaScript'];
```

W razie potrzeby możesz przekazać metodzie `flatten` argument "głębokości":

```php
$collection = collect([
    'Apple' => [
        [
            'name' => 'iPhone 6S',
            'brand' => 'Apple'
        ],
    ],
    'Samsung' => [
        [
            'name' => 'Galaxy S7',
            'brand' => 'Samsung'
        ],
    ],
]);

$products = $collection->flatten(1);

$products->values()->all();

/*
    [
        ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
    ]
*/
```

W tym przykładzie wywołanie `flatten` bez podania głębokości spłaszczyłoby również zagnieżdżone tablice, dając w wyniku `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`. Podanie głębokości pozwala określić liczbę poziomów, o które zagnieżdżone tablice zostaną spłaszczone.

<a name="method-flip"></a>
#### `flip()` {.collection-method}

Metoda `flip` zamienia klucze kolekcji z ich odpowiadającymi wartościami:

```php
$collection = collect(['name' => 'Taylor', 'framework' => 'Laravel']);

$flipped = $collection->flip();

$flipped->all();

// ['Taylor' => 'name', 'Laravel' => 'framework']
```

<a name="method-forget"></a>
#### `forget()` {.collection-method}

Metoda `forget` usuwa element z kolekcji według jego klucza:

```php
$collection = collect(['name' => 'Taylor', 'framework' => 'Laravel']);

// Zapomnij pojedynczy klucz...
$collection->forget('name');

// ['framework' => 'Laravel']

// Zapomnij wiele kluczy...
$collection->forget(['name', 'framework']);

// []
```

> [!WARNING]
> W przeciwieństwie do większości innych metod kolekcji, `forget` nie zwraca nowej zmodyfikowanej kolekcji; modyfikuje i zwraca kolekcję, na której została wywołana.

<a name="method-forpage"></a>
#### `forPage()` {.collection-method}

Metoda `forPage` zwraca nową kolekcję zawierającą elementy, które byłyby obecne na danym numerze strony. Metoda akceptuje numer strony jako swój pierwszy argument i liczbę elementów do pokazania na stronie jako swój drugi argument:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

$chunk = $collection->forPage(2, 3);

$chunk->all();

// [4, 5, 6]
```

<a name="method-fromjson"></a>
#### `fromJson()` {.collection-method}

Statyczna metoda `fromJson` tworzy nową instancję kolekcji przez dekodowanie podanego ciągu JSON używając funkcji PHP `json_decode`:

```php
use Illuminate\Support\Collection;

$json = json_encode([
    'name' => 'Taylor Otwell',
    'role' => 'Developer',
    'status' => 'Active',
]);

$collection = Collection::fromJson($json);
```

<a name="method-get"></a>
#### `get()` {.collection-method}

Metoda `get` zwraca element o podanym kluczu. Jeśli klucz nie istnieje, zwracany jest `null`:

```php
$collection = collect(['name' => 'Taylor', 'framework' => 'Laravel']);

$value = $collection->get('name');

// Taylor
```

Możesz opcjonalnie przekazać wartość domyślną jako drugi argument:

```php
$collection = collect(['name' => 'Taylor', 'framework' => 'Laravel']);

$value = $collection->get('age', 34);

// 34
```

Możesz nawet przekazać callback jako wartość domyślną metody. Wynik callbacku zostanie zwrócony, jeśli podany klucz nie istnieje:

```php
$collection->get('email', function () {
    return 'taylor@example.com';
});

// taylor@example.com
```

<a name="method-groupby"></a>
#### `groupBy()` {.collection-method}

Metoda `groupBy` grupuje elementy kolekcji według podanego klucza:

```php
$collection = collect([
    ['account_id' => 'account-x10', 'product' => 'Chair'],
    ['account_id' => 'account-x10', 'product' => 'Bookcase'],
    ['account_id' => 'account-x11', 'product' => 'Desk'],
]);

$grouped = $collection->groupBy('account_id');

$grouped->all();

/*
    [
        'account-x10' => [
            ['account_id' => 'account-x10', 'product' => 'Chair'],
            ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ],
        'account-x11' => [
            ['account_id' => 'account-x11', 'product' => 'Desk'],
        ],
    ]
*/
```

Zamiast przekazywać ciąg znaków `key`, możesz przekazać callback. Callback powinien zwrócić wartość, według której chcesz grupować:

```php
$grouped = $collection->groupBy(function (array $item, int $key) {
    return substr($item['account_id'], -3);
});

$grouped->all();

/*
    [
        'x10' => [
            ['account_id' => 'account-x10', 'product' => 'Chair'],
            ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ],
        'x11' => [
            ['account_id' => 'account-x11', 'product' => 'Desk'],
        ],
    ]
*/
```

Wiele kryteriów grupowania może być przekazanych jako tablica. Każdy element tablicy zostanie zastosowany do odpowiedniego poziomu w tablicy wielowymiarowej:

```php
$data = new Collection([
    10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
    20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
    30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
    40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
]);

$result = $data->groupBy(['skill', function (array $item) {
    return $item['roles'];
}], preserveKeys: true);

/*
[
    1 => [
        'Role_1' => [
            10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
            20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        ],
        'Role_2' => [
            20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        ],
        'Role_3' => [
            10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
        ],
    ],
    2 => [
        'Role_1' => [
            30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
        ],
        'Role_2' => [
            40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
        ],
    ],
];
*/
```

<a name="method-has"></a>
#### `has()` {.collection-method}

Metoda `has` określa, czy podany klucz istnieje w kolekcji:

```php
$collection = collect(['account_id' => 1, 'product' => 'Desk', 'amount' => 5]);

$collection->has('product');

// true

$collection->has(['product', 'amount']);

// true

$collection->has(['amount', 'price']);

// false
```

<a name="method-hasany"></a>
#### `hasAny()` {.collection-method}

Metoda `hasAny` określa, czy którykolwiek z podanych kluczy istnieje w kolekcji:

```php
$collection = collect(['account_id' => 1, 'product' => 'Desk', 'amount' => 5]);

$collection->hasAny(['product', 'price']);

// true

$collection->hasAny(['name', 'price']);

// false
```

<a name="method-hassole"></a>
#### `hasSole()` {.collection-method}

Metoda `hasSole` określa, czy kolekcja zawiera pojedynczy element, opcjonalnie spełniający podane kryteria:

```php
collect([])->hasSole();

// false

collect(['1'])->hasSole();

// true

collect([1, 2, 3])->hasSole(fn (int $item) => $item === 2);

// true
```

<a name="method-implode"></a>
#### `implode()` {.collection-method}

Metoda `implode` łączy elementy w kolekcji. Jej argumenty zależą od typu elementów w kolekcji. Jeśli kolekcja zawiera tablice lub obiekty, powinieneś przekazać klucz atrybutów, które chcesz połączyć, oraz ciąg "kleju", który chcesz umieścić między wartościami:

```php
$collection = collect([
    ['account_id' => 1, 'product' => 'Desk'],
    ['account_id' => 2, 'product' => 'Chair'],
]);

$collection->implode('product', ', ');

// 'Desk, Chair'
```

Jeśli kolekcja zawiera proste ciągi znaków lub wartości numeryczne, powinieneś przekazać "klej" jako jedyny argument do metody:

```php
collect([1, 2, 3, 4, 5])->implode('-');

// '1-2-3-4-5'
```

Możesz przekazać zamknięcie do metody `implode`, jeśli chcesz sformatować wartości, które są łączone:

```php
$collection->implode(function (array $item, int $key) {
    return strtoupper($item['product']);
}, ', ');

// 'DESK, CHAIR'
```

<a name="method-intersect"></a>
#### `intersect()` {.collection-method}

Metoda `intersect` usuwa wszelkie wartości z oryginalnej kolekcji, które nie są obecne w podanej tablicy lub kolekcji. Wynikowa kolekcja zachowa klucze oryginalnej kolekcji:

```php
$collection = collect(['Desk', 'Sofa', 'Chair']);

$intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

$intersect->all();

// [0 => 'Desk', 2 => 'Chair']
```

> [!NOTE]
> Zachowanie tej metody jest modyfikowane podczas używania [kolekcji Eloquent](/docs/{{version}}/eloquent-collections#method-intersect).

<a name="method-intersectusing"></a>
#### `intersectUsing()` {.collection-method}

Metoda `intersectUsing` usuwa wszelkie wartości z oryginalnej kolekcji, które nie są obecne w podanej tablicy lub kolekcji, używając niestandardowego callbacku do porównywania wartości. Wynikowa kolekcja zachowa klucze oryginalnej kolekcji:

```php
$collection = collect(['Desk', 'Sofa', 'Chair']);

$intersect = $collection->intersectUsing(['desk', 'chair', 'bookcase'], function (string $a, string $b) {
    return strcasecmp($a, $b);
});

$intersect->all();

// [0 => 'Desk', 2 => 'Chair']
```

<a name="method-intersectAssoc"></a>
#### `intersectAssoc()` {.collection-method}

Metoda `intersectAssoc` porównuje oryginalną kolekcję z inną kolekcją lub tablicą, zwracając pary klucz/wartość, które są obecne we wszystkich podanych kolekcjach:

```php
$collection = collect([
    'color' => 'red',
    'size' => 'M',
    'material' => 'cotton'
]);

$intersect = $collection->intersectAssoc([
    'color' => 'blue',
    'size' => 'M',
    'material' => 'polyester'
]);

$intersect->all();

// ['size' => 'M']
```

<a name="method-intersectassocusing"></a>
#### `intersectAssocUsing()` {.collection-method}

Metoda `intersectAssocUsing` porównuje oryginalną kolekcję z inną kolekcją lub tablicą, zwracając pary klucz/wartość, które są obecne w obu, używając niestandardowego callbacku porównania do określania równości zarówno kluczy, jak i wartości:

```php
$collection = collect([
    'color' => 'red',
    'Size' => 'M',
    'material' => 'cotton',
]);

$intersect = $collection->intersectAssocUsing([
    'color' => 'blue',
    'size' => 'M',
    'material' => 'polyester',
], function (string $a, string $b) {
    return strcasecmp($a, $b);
});

$intersect->all();

// ['Size' => 'M']
```

<a name="method-intersectbykeys"></a>
#### `intersectByKeys()` {.collection-method}

Metoda `intersectByKeys` usuwa wszelkie klucze i ich odpowiadające wartości z oryginalnej kolekcji, które nie są obecne w podanej tablicy lub kolekcji:

```php
$collection = collect([
    'serial' => 'UX301', 'type' => 'screen', 'year' => 2009,
]);

$intersect = $collection->intersectByKeys([
    'reference' => 'UX404', 'type' => 'tab', 'year' => 2011,
]);

$intersect->all();

// ['type' => 'screen', 'year' => 2009]
```

<a name="method-isempty"></a>
#### `isEmpty()` {.collection-method}

Metoda `isEmpty` zwraca `true`, jeśli kolekcja jest pusta; w przeciwnym razie zwracany jest `false`:

```php
collect([])->isEmpty();

// true
```

<a name="method-isnotempty"></a>
#### `isNotEmpty()` {.collection-method}

Metoda `isNotEmpty` zwraca `true`, jeśli kolekcja nie jest pusta; w przeciwnym razie zwracany jest `false`:

```php
collect([])->isNotEmpty();

// false
```

<a name="method-join"></a>
#### `join()` {.collection-method}

Metoda `join` łączy wartości kolekcji z ciągiem znaków. Używając drugiego argumentu tej metody, możesz również określić, jak ostatni element powinien być dołączony do ciągu:

```php
collect(['a', 'b', 'c'])->join(', '); // 'a, b, c'
collect(['a', 'b', 'c'])->join(', ', ', and '); // 'a, b, and c'
collect(['a', 'b'])->join(', ', ' and '); // 'a and b'
collect(['a'])->join(', ', ' and '); // 'a'
collect([])->join(', ', ' and '); // ''
```

<a name="method-keyby"></a>
#### `keyBy()` {.collection-method}

Metoda `keyBy` indeksuje kolekcję według podanego klucza. Jeśli wiele elementów ma ten sam klucz, tylko ostatni pojawi się w nowej kolekcji:

```php
$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$keyed = $collection->keyBy('product_id');

$keyed->all();

/*
    [
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]
*/
```

Możesz również przekazać callback do metody. Callback powinien zwrócić wartość, według której indeksować kolekcję:

```php
$keyed = $collection->keyBy(function (array $item, int $key) {
    return strtoupper($item['product_id']);
});

$keyed->all();

/*
    [
        'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]
*/
```

<a name="method-keys"></a>
#### `keys()` {.collection-method}

Metoda `keys` zwraca wszystkie klucze kolekcji:

```php
$collection = collect([
    'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
    'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$keys = $collection->keys();

$keys->all();

// ['prod-100', 'prod-200']
```

<a name="method-last"></a>
#### `last()` {.collection-method}

Metoda `last` zwraca ostatni element w kolekcji, który przechodzi dany test prawdziwości:

```php
collect([1, 2, 3, 4])->last(function (int $value, int $key) {
    return $value < 3;
});

// 2
```

Możesz również wywołać metodę `last` bez argumentów, aby uzyskać ostatni element w kolekcji. Jeśli kolekcja jest pusta, zwracany jest `null`:

```php
collect([1, 2, 3, 4])->last();

// 4
```

<a name="method-lazy"></a>
#### `lazy()` {.collection-method}

Metoda `lazy` zwraca nową instancję [LazyCollection](#lazy-collections) z tablicy elementów bazowych:

```php
$lazyCollection = collect([1, 2, 3, 4])->lazy();

$lazyCollection::class;

// Illuminate\Support\LazyCollection

$lazyCollection->all();

// [1, 2, 3, 4]
```

Jest to szczególnie przydatne, gdy musisz wykonać transformacje na ogromnej `Collection`, która zawiera wiele elementów:

```php
$count = $hugeCollection
    ->lazy()
    ->where('country', 'FR')
    ->where('balance', '>', '100')
    ->count();
```

Poprzez konwersję kolekcji na `LazyCollection`, unikamy konieczności alokowania dużej ilości dodatkowej pamięci. Chociaż oryginalna kolekcja nadal przechowuje _swoje_ wartości w pamięci, kolejne filtry nie będą. Dlatego praktycznie żadna dodatkowa pamięć nie zostanie przydzielona podczas filtrowania wyników kolekcji.

<a name="method-macro"></a>
#### `macro()` {.collection-method}

Statyczna metoda `macro` pozwala na dodanie metod do klasy `Collection` w czasie wykonywania. Zapoznaj się z dokumentacją dotyczącą [rozszerzania kolekcji](#extending-collections) po więcej informacji.

<a name="method-make"></a>
#### `make()` {.collection-method}

Statyczna metoda `make` tworzy nową instancję kolekcji. Zobacz sekcję [Tworzenie kolekcji](#creating-collections).

```php
use Illuminate\Support\Collection;

$collection = Collection::make([1, 2, 3]);
```

<a name="method-map"></a>
#### `map()` {.collection-method}

Metoda `map` iteruje przez kolekcję i przekazuje każdą wartość do podanego callbacku. Callback może swobodnie modyfikować element i go zwrócić, tworząc w ten sposób nową kolekcję zmodyfikowanych elementów:

```php
$collection = collect([1, 2, 3, 4, 5]);

$multiplied = $collection->map(function (int $item, int $key) {
    return $item * 2;
});

$multiplied->all();

// [2, 4, 6, 8, 10]
```

> [!WARNING]
> Podobnie jak większość innych metod kolekcji, `map` zwraca nową instancję kolekcji; nie modyfikuje kolekcji, na której została wywołana. Jeśli chcesz przekształcić oryginalną kolekcję, użyj metody [transform](#method-transform).

<a name="method-mapinto"></a>
#### `mapInto()` {.collection-method}

Metoda `mapInto()` iteruje po kolekcji, tworząc nową instancję podanej klasy przez przekazanie wartości do konstruktora:

```php
class Currency
{
    /**
     * Create a new currency instance.
     */
    function __construct(
        public string $code,
    ) {}
}

$collection = collect(['USD', 'EUR', 'GBP']);

$currencies = $collection->mapInto(Currency::class);

$currencies->all();

// [Currency('USD'), Currency('EUR'), Currency('GBP')]
```

<a name="method-mapspread"></a>
#### `mapSpread()` {.collection-method}

Metoda `mapSpread` iteruje po elementach kolekcji, przekazując każdą zagnieżdżoną wartość elementu do podanego zamknięcia. Zamknięcie może swobodnie modyfikować element i go zwrócić, tworząc w ten sposób nową kolekcję zmodyfikowanych elementów:

```php
$collection = collect([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

$chunks = $collection->chunk(2);

$sequence = $chunks->mapSpread(function (int $even, int $odd) {
    return $even + $odd;
});

$sequence->all();

// [1, 5, 9, 13, 17]
```

<a name="method-maptogroups"></a>
#### `mapToGroups()` {.collection-method}

Metoda `mapToGroups` grupuje elementy kolekcji według podanego zamknięcia. Zamknięcie powinno zwrócić tablicę asocjacyjną zawierającą pojedynczą parę klucz/wartość, tworząc w ten sposób nową kolekcję zgrupowanych wartości:

```php
$collection = collect([
    [
        'name' => 'John Doe',
        'department' => 'Sales',
    ],
    [
        'name' => 'Jane Doe',
        'department' => 'Sales',
    ],
    [
        'name' => 'Johnny Doe',
        'department' => 'Marketing',
    ]
]);

$grouped = $collection->mapToGroups(function (array $item, int $key) {
    return [$item['department'] => $item['name']];
});

$grouped->all();

/*
    [
        'Sales' => ['John Doe', 'Jane Doe'],
        'Marketing' => ['Johnny Doe'],
    ]
*/

$grouped->get('Sales')->all();

// ['John Doe', 'Jane Doe']
```

<a name="method-mapwithkeys"></a>
#### `mapWithKeys()` {.collection-method}

Metoda `mapWithKeys` iteruje przez kolekcję i przekazuje każdą wartość do podanego callbacku. Callback powinien zwrócić tablicę asocjacyjną zawierającą pojedynczą parę klucz/wartość:

```php
$collection = collect([
    [
        'name' => 'John',
        'department' => 'Sales',
        'email' => 'john@example.com',
    ],
    [
        'name' => 'Jane',
        'department' => 'Marketing',
        'email' => 'jane@example.com',
    ]
]);

$keyed = $collection->mapWithKeys(function (array $item, int $key) {
    return [$item['email'] => $item['name']];
});

$keyed->all();

/*
    [
        'john@example.com' => 'John',
        'jane@example.com' => 'Jane',
    ]
*/
```

<a name="method-max"></a>
#### `max()` {.collection-method}

Metoda `max` zwraca maksymalną wartość podanego klucza:

```php
$max = collect([
    ['foo' => 10],
    ['foo' => 20]
])->max('foo');

// 20

$max = collect([1, 2, 3, 4, 5])->max();

// 5
```

<a name="method-median"></a>
#### `median()` {.collection-method}

Metoda `median` zwraca [wartość mediany](https://en.wikipedia.org/wiki/Median) podanego klucza:

```php
$median = collect([
    ['foo' => 10],
    ['foo' => 10],
    ['foo' => 20],
    ['foo' => 40]
])->median('foo');

// 15

$median = collect([1, 1, 2, 4])->median();

// 1.5
```

<a name="method-merge"></a>
#### `merge()` {.collection-method}

Metoda `merge` łączy podaną tablicę lub kolekcję z oryginalną kolekcją. Jeśli klucz ciągu znaków w podanych elementach pasuje do klucza ciągu znaków w oryginalnej kolekcji, wartość podanego elementu nadpisze wartość w oryginalnej kolekcji:

```php
$collection = collect(['product_id' => 1, 'price' => 100]);

$merged = $collection->merge(['price' => 200, 'discount' => false]);

$merged->all();

// ['product_id' => 1, 'price' => 200, 'discount' => false]
```

Jeśli klucze podanych elementów są numeryczne, wartości zostaną dodane na końcu kolekcji:

```php
$collection = collect(['Desk', 'Chair']);

$merged = $collection->merge(['Bookcase', 'Door']);

$merged->all();

// ['Desk', 'Chair', 'Bookcase', 'Door']
```

<a name="method-mergerecursive"></a>
#### `mergeRecursive()` {.collection-method}

Metoda `mergeRecursive` łączy podaną tablicę lub kolekcję rekurencyjnie z oryginalną kolekcją. Jeśli klucz ciągu znaków w podanych elementach pasuje do klucza ciągu znaków w oryginalnej kolekcji, to wartości dla tych kluczy są łączone razem w tablicę, i jest to robione rekurencyjnie:

```php
$collection = collect(['product_id' => 1, 'price' => 100]);

$merged = $collection->mergeRecursive([
    'product_id' => 2,
    'price' => 200,
    'discount' => false
]);

$merged->all();

// ['product_id' => [1, 2], 'price' => [100, 200], 'discount' => false]
```

<a name="method-min"></a>
#### `min()` {.collection-method}

Metoda `min` zwraca minimalną wartość podanego klucza:

```php
$min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

// 10

$min = collect([1, 2, 3, 4, 5])->min();

// 1
```

<a name="method-mode"></a>
#### `mode()` {.collection-method}

Metoda `mode` zwraca [wartość modalną](https://en.wikipedia.org/wiki/Mode_(statistics)) podanego klucza:

```php
$mode = collect([
    ['foo' => 10],
    ['foo' => 10],
    ['foo' => 20],
    ['foo' => 40]
])->mode('foo');

// [10]

$mode = collect([1, 1, 2, 4])->mode();

// [1]

$mode = collect([1, 1, 2, 2])->mode();

// [1, 2]
```

<a name="method-multiply"></a>
#### `multiply()` {.collection-method}

Metoda `multiply` tworzy określoną liczbę kopii wszystkich elementów w kolekcji:

```php
$users = collect([
    ['name' => 'User #1', 'email' => 'user1@example.com'],
    ['name' => 'User #2', 'email' => 'user2@example.com'],
])->multiply(3);

/*
    [
        ['name' => 'User #1', 'email' => 'user1@example.com'],
        ['name' => 'User #2', 'email' => 'user2@example.com'],
        ['name' => 'User #1', 'email' => 'user1@example.com'],
        ['name' => 'User #2', 'email' => 'user2@example.com'],
        ['name' => 'User #1', 'email' => 'user1@example.com'],
        ['name' => 'User #2', 'email' => 'user2@example.com'],
    ]
*/
```

<a name="method-nth"></a>
#### `nth()` {.collection-method}

Metoda `nth` tworzy nową kolekcję składającą się z co n-tego elementu:

```php
$collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

$collection->nth(4);

// ['a', 'e']
```

Opcjonalnie możesz przekazać przesunięcie początkowe jako drugi argument:

```php
$collection->nth(4, 1);

// ['b', 'f']
```

<a name="method-only"></a>
#### `only()` {.collection-method}

Metoda `only` zwraca elementy w kolekcji z określonymi kluczami:

```php
$collection = collect([
    'product_id' => 1,
    'name' => 'Desk',
    'price' => 100,
    'discount' => false
]);

$filtered = $collection->only(['product_id', 'name']);

$filtered->all();

// ['product_id' => 1, 'name' => 'Desk']
```

Dla odwrotności metody `only`, zobacz metodę [except](#method-except).

> [!NOTE]
> Zachowanie tej metody jest modyfikowane podczas używania [Kolekcji Eloquent](/docs/{{version}}/eloquent-collections#method-only).

<a name="method-pad"></a>
#### `pad()` {.collection-method}

Metoda `pad` wypełni tablicę podaną wartością, aż tablica osiągnie określony rozmiar. Ta metoda zachowuje się jak funkcja PHP [array_pad](https://secure.php.net/manual/en/function.array-pad.php).

Aby wypełnić po lewej stronie, należy określić ujemny rozmiar. Wypełnienie nie nastąpi, jeśli wartość bezwzględna podanego rozmiaru jest mniejsza lub równa długości tablicy:

```php
$collection = collect(['A', 'B', 'C']);

$filtered = $collection->pad(5, 0);

$filtered->all();

// ['A', 'B', 'C', 0, 0]

$filtered = $collection->pad(-5, 0);

$filtered->all();

// [0, 0, 'A', 'B', 'C']
```

<a name="method-partition"></a>
#### `partition()` {.collection-method}

Metoda `partition` może być połączona z destrukturyzacją tablicy PHP w celu rozdzielenia elementów, które przeją dany test prawdziwości od tych, które go nie przeją:

```php
$collection = collect([1, 2, 3, 4, 5, 6]);

[$underThree, $equalOrAboveThree] = $collection->partition(function (int $i) {
    return $i < 3;
});

$underThree->all();

// [1, 2]

$equalOrAboveThree->all();

// [3, 4, 5, 6]
```

> [!NOTE]
> Zachowanie tej metody jest modyfikowane podczas interakcji z [kolekcjami Eloquent](/docs/{{version}}/eloquent-collections#method-partition).

<a name="method-percentage"></a>
#### `percentage()` {.collection-method}

Metoda `percentage` może być użyta do szybkiego określenia procentu elementów w kolekcji, które przeją dany test prawdziwości:

```php
$collection = collect([1, 1, 2, 2, 2, 3]);

$percentage = $collection->percentage(fn (int $value) => $value === 1);

// 33.33
```

Domyślnie procent zostanie zaokrąglony do dwóch miejsc po przecinku. Jednak możesz dostosować to zachowanie, podając drugi argument do metody:

```php
$percentage = $collection->percentage(fn (int $value) => $value === 1, precision: 3);

// 33.333
```

<a name="method-pipe"></a>
#### `pipe()` {.collection-method}

Metoda `pipe` przekazuje kolekcję do podanego zamknięcia i zwraca wynik wykonanego zamknięcia:

```php
$collection = collect([1, 2, 3]);

$piped = $collection->pipe(function (Collection $collection) {
    return $collection->sum();
});

// 6
```

<a name="method-pipeinto"></a>
#### `pipeInto()` {.collection-method}

Metoda `pipeInto` tworzy nową instancję podanej klasy i przekazuje kolekcję do konstruktora:

```php
class ResourceCollection
{
    /**
     * Create a new ResourceCollection instance.
     */
    public function __construct(
        public Collection $collection,
    ) {}
}

$collection = collect([1, 2, 3]);

$resource = $collection->pipeInto(ResourceCollection::class);

$resource->collection->all();

// [1, 2, 3]
```

<a name="method-pipethrough"></a>
#### `pipeThrough()` {.collection-method}

Metoda `pipeThrough` przekazuje kolekcję do podanej tablicy zamknięć i zwraca wynik wykonanych zamknięć:

```php
use Illuminate\Support\Collection;

$collection = collect([1, 2, 3]);

$result = $collection->pipeThrough([
    function (Collection $collection) {
        return $collection->merge([4, 5]);
    },
    function (Collection $collection) {
        return $collection->sum();
    },
]);

// 15
```

<a name="method-pluck"></a>
#### `pluck()` {.collection-method}

Metoda `pluck` pobiera wszystkie wartości dla podanego klucza:

```php
$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$plucked = $collection->pluck('name');

$plucked->all();

// ['Desk', 'Chair']
```

Możesz również określić, w jaki sposób chcesz, aby wynikowa kolekcja była kluczowana:

```php
$plucked = $collection->pluck('name', 'product_id');

$plucked->all();

// ['prod-100' => 'Desk', 'prod-200' => 'Chair']
```

Metoda `pluck` również obsługuje pobieranie zagnieżdżonych wartości za pomocą notacji "kropkowej":

```php
$collection = collect([
    [
        'name' => 'Laracon',
        'speakers' => [
            'first_day' => ['Rosa', 'Judith'],
        ],
    ],
    [
        'name' => 'VueConf',
        'speakers' => [
            'first_day' => ['Abigail', 'Joey'],
        ],
    ],
]);

$plucked = $collection->pluck('speakers.first_day');

$plucked->all();

// [['Rosa', 'Judith'], ['Abigail', 'Joey']]
```

Jeśli istnieją zduplikowane klucze, ostatni pasujący element zostanie wstawiony do kolekcji wyciągniętej:

```php
$collection = collect([
    ['brand' => 'Tesla',  'color' => 'red'],
    ['brand' => 'Pagani', 'color' => 'white'],
    ['brand' => 'Tesla',  'color' => 'black'],
    ['brand' => 'Pagani', 'color' => 'orange'],
]);

$plucked = $collection->pluck('color', 'brand');

$plucked->all();

// ['Tesla' => 'black', 'Pagani' => 'orange']
```

<a name="method-pop"></a>
#### `pop()` {.collection-method}

Metoda `pop` usuwa i zwraca ostatni element z kolekcji. Jeśli kolekcja jest pusta, zostanie zwrócone `null`:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->pop();

// 5

$collection->all();

// [1, 2, 3, 4]
```

Możesz przekazać liczbę całkowitą do metody `pop`, aby usunąć i zwrócić wiele elementów z końca kolekcji:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->pop(3);

// collect([5, 4, 3])

$collection->all();

// [1, 2]
```

<a name="method-prepend"></a>
#### `prepend()` {.collection-method}

Metoda `prepend` dodaje element na początek kolekcji:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->prepend(0);

$collection->all();

// [0, 1, 2, 3, 4, 5]
```

Możesz również przekazać drugi argument, aby określić klucz poprzedzającego elementu:

```php
$collection = collect(['one' => 1, 'two' => 2]);

$collection->prepend(0, 'zero');

$collection->all();

// ['zero' => 0, 'one' => 1, 'two' => 2]
```

<a name="method-pull"></a>
#### `pull()` {.collection-method}

Metoda `pull` usuwa i zwraca element z kolekcji na podstawie jego klucza:

```php
$collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

$collection->pull('name');

// 'Desk'

$collection->all();

// ['product_id' => 'prod-100']
```

<a name="method-push"></a>
#### `push()` {.collection-method}

Metoda `push` dodaje element na koniec kolekcji:

```php
$collection = collect([1, 2, 3, 4]);

$collection->push(5);

$collection->all();

// [1, 2, 3, 4, 5]
```

Możesz również dodać wiele elementów na koniec kolekcji:

```php
$collection = collect([1, 2, 3, 4]);

$collection->push(5, 6, 7);
 
$collection->all();
 
// [1, 2, 3, 4, 5, 6, 7]
```

<a name="method-put"></a>
#### `put()` {.collection-method}

Metoda `put` ustawia podany klucz i wartość w kolekcji:

```php
$collection = collect(['product_id' => 1, 'name' => 'Desk']);

$collection->put('price', 100);

$collection->all();

// ['product_id' => 1, 'name' => 'Desk', 'price' => 100]
```

<a name="method-random"></a>
#### `random()` {.collection-method}

Metoda `random` zwraca losowy element z kolekcji:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->random();

// 4 - (retrieved randomly)
```

Możesz przekazać liczbę całkowitą do `random`, aby określić, ile elementów chcesz losowo pobrać. Kolekcja elementów jest zawsze zwracana, gdy wyraźnie przekazujesz liczbę elementów, które chcesz otrzymać:

```php
$random = $collection->random(3);

$random->all();

// [2, 4, 5] - (retrieved randomly)
```

Jeśli instancja kolekcji ma mniej elementów niż zażądano, metoda `random` zgłosi wyjątek `InvalidArgumentException`.

Metoda `random` akceptuje również zamknięcie, które otrzyma bieżącą instancję kolekcji:

```php
use Illuminate\Support\Collection;

$random = $collection->random(fn (Collection $items) => min(10, count($items)));

$random->all();

// [1, 2, 3, 4, 5] - (retrieved randomly)
```

<a name="method-range"></a>
#### `range()` {.collection-method}

Metoda `range` zwraca kolekcję zawierającą liczby całkowite w określonym zakresie:

```php
$collection = collect()->range(3, 6);

$collection->all();

// [3, 4, 5, 6]
```

<a name="method-reduce"></a>
#### `reduce()` {.collection-method}

Metoda `reduce` redukuje kolekcję do pojedynczej wartości, przekazując wynik każdej iteracji do następnej iteracji:

```php
$collection = collect([1, 2, 3]);

$total = $collection->reduce(function (?int $carry, int $item) {
    return $carry + $item;
});

// 6
```

Wartość `$carry` w pierwszej iteracji to `null`; jednak możesz określić jej wartość początkową, przekazując drugi argument do `reduce`:

```php
$collection->reduce(function (int $carry, int $item) {
    return $carry + $item;
}, 4);

// 10
```

Metoda `reduce` przekazuje również klucze tablicy do podanego callbacka:

```php
$collection = collect([
    'usd' => 1400,
    'gbp' => 1200,
    'eur' => 1000,
]);

$ratio = [
    'usd' => 1,
    'gbp' => 1.37,
    'eur' => 1.22,
];

$collection->reduce(function (int $carry, int $value, string $key) use ($ratio) {
    return $carry + ($value * $ratio[$key]);
}, 0);

// 4264
```

<a name="method-reduce-spread"></a>
#### `reduceSpread()` {.collection-method}

Metoda `reduceSpread` redukuje kolekcję do tablicy wartości, przekazując wyniki każdej iteracji do następnej iteracji. Ta metoda jest podobna do metody `reduce`; jednak może akceptować wiele wartości początkowych:

```php
[$creditsRemaining, $batch] = Image::where('status', 'unprocessed')
    ->get()
    ->reduceSpread(function (int $creditsRemaining, Collection $batch, Image $image) {
        if ($creditsRemaining >= $image->creditsRequired()) {
            $batch->push($image);

            $creditsRemaining -= $image->creditsRequired();
        }

        return [$creditsRemaining, $batch];
    }, $creditsAvailable, collect());
```

<a name="method-reject"></a>
#### `reject()` {.collection-method}

Metoda `reject` filtruje kolekcję za pomocą podanego zamknięcia. Zamknięcie powinno zwrócić `true`, jeśli element powinien zostać usunięty z wynikowej kolekcji:

```php
$collection = collect([1, 2, 3, 4]);

$filtered = $collection->reject(function (int $value, int $key) {
    return $value > 2;
});

$filtered->all();

// [1, 2]
```

Dla odwrotności metody `reject`, zobacz metodę [filter](#method-filter).

<a name="method-replace"></a>
#### `replace()` {.collection-method}

Metoda `replace` zachowuje się podobnie do `merge`; jednak oprócz nadpisywania pasujących elementów, które mają klucze ciągowe, metoda `replace` nadpisze również elementy w kolekcji, które mają pasujące klucze numeryczne:

```php
$collection = collect(['Taylor', 'Abigail', 'James']);

$replaced = $collection->replace([1 => 'Victoria', 3 => 'Finn']);

$replaced->all();

// ['Taylor', 'Victoria', 'James', 'Finn']
```

<a name="method-replacerecursive"></a>
#### `replaceRecursive()` {.collection-method}

Metoda `replaceRecursive` zachowuje się podobnie do `replace`, ale będzie schodzić w głąb do tablic i stosować ten sam proces zastępowania do wartości wewnętrznych:

```php
$collection = collect([
    'Taylor',
    'Abigail',
    [
        'James',
        'Victoria',
        'Finn'
    ]
]);

$replaced = $collection->replaceRecursive([
    'Charlie',
    2 => [1 => 'King']
]);

$replaced->all();

// ['Charlie', 'Abigail', ['James', 'King', 'Finn']]
```

<a name="method-reverse"></a>
#### `reverse()` {.collection-method}

Metoda `reverse` odwraca kolejność elementów kolekcji, zachowując oryginalne klucze:

```php
$collection = collect(['a', 'b', 'c', 'd', 'e']);

$reversed = $collection->reverse();

$reversed->all();

/*
    [
        4 => 'e',
        3 => 'd',
        2 => 'c',
        1 => 'b',
        0 => 'a',
    ]
*/
```

<a name="method-search"></a>
#### `search()` {.collection-method}

Metoda `search` wyszukuje w kolekcji podaną wartość i zwraca jej klucz, jeśli zostanie znaleziona. Jeśli element nie zostanie znaleziony, zwracane jest `false`:

```php
$collection = collect([2, 4, 6, 8]);

$collection->search(4);

// 1
```

Wyszukiwanie jest wykonywane przy użyciu "luźnego" porównania, co oznacza, że ciąg z wartością całkowitą będzie uważany za równy liczbie całkowitej o tej samej wartości. Aby użyć "ściemu" porównania, przekaz `true` jako drugi argument do metody:

```php
collect([2, 4, 6, 8])->search('4', strict: true);

// false
```

Alternatywnie możesz przekazać własne zamknięcie, aby wyszukać pierwszy element, który przejdzie dany test prawdziwości:

```php
collect([2, 4, 6, 8])->search(function (int $item, int $key) {
    return $item > 5;
});

// 2
```

<a name="method-select"></a>
#### `select()` {.collection-method}

Metoda `select` wybiera podane klucze z kolekcji, podobnie do instrukcji SQL `SELECT`:

```php
$users = collect([
    ['name' => 'Taylor Otwell', 'role' => 'Developer', 'status' => 'active'],
    ['name' => 'Victoria Faith', 'role' => 'Researcher', 'status' => 'active'],
]);

$users->select(['name', 'role']);

/*
    [
        ['name' => 'Taylor Otwell', 'role' => 'Developer'],
        ['name' => 'Victoria Faith', 'role' => 'Researcher'],
    ],
*/
```

<a name="method-shift"></a>
#### `shift()` {.collection-method}

Metoda `shift` usuwa i zwraca pierwszy element z kolekcji:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->shift();

// 1

$collection->all();

// [2, 3, 4, 5]
```

Możesz przekazać liczbę całkowitą do metody `shift`, aby usunąć i zwrócić wiele elementów z początku kolekcji:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->shift(3);

// collect([1, 2, 3])

$collection->all();

// [4, 5]
```

<a name="method-shuffle"></a>
#### `shuffle()` {.collection-method}

Metoda `shuffle` losowo tasuje elementy w kolekcji:

```php
$collection = collect([1, 2, 3, 4, 5]);

$shuffled = $collection->shuffle();

$shuffled->all();

// [3, 2, 5, 1, 4] - (generated randomly)
```

<a name="method-skip"></a>
#### `skip()` {.collection-method}

Metoda `skip` zwraca nową kolekcję z podaną liczbą elementów usuniętych z początku kolekcji:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$collection = $collection->skip(4);

$collection->all();

// [5, 6, 7, 8, 9, 10]
```

<a name="method-skipuntil"></a>
#### `skipUntil()` {.collection-method}

Metoda `skipUntil` pomija elementy z kolekcji, dopóki podany callback zwraca `false`. Gdy callback zwróci `true`, wszystkie pozostałe elementy w kolekcji zostaną zwrócone jako nowa kolekcja:

```php
$collection = collect([1, 2, 3, 4]);

$subset = $collection->skipUntil(function (int $item) {
    return $item >= 3;
});

$subset->all();

// [3, 4]
```

Możesz również przekazać prostą wartość do metody `skipUntil`, aby pominąć wszystkie elementy do momentu znalezienia podanej wartości:

```php
$collection = collect([1, 2, 3, 4]);

$subset = $collection->skipUntil(3);

$subset->all();

// [3, 4]
```

> [!WARNING]
> Jeśli podana wartość nie zostanie znaleziona lub callback nigdy nie zwróci `true`, metoda `skipUntil` zwróci pustą kolekcję.

<a name="method-skipwhile"></a>
#### `skipWhile()` {.collection-method}

Metoda `skipWhile` pomija elementy z kolekcji, dopóki podany callback zwraca `true`. Gdy callback zwróci `false`, wszystkie pozostałe elementy w kolekcji zostaną zwrócone jako nowa kolekcja:

```php
$collection = collect([1, 2, 3, 4]);

$subset = $collection->skipWhile(function (int $item) {
    return $item <= 3;
});

$subset->all();

// [4]
```

> [!WARNING]
> Jeśli callback nigdy nie zwróci `false`, metoda `skipWhile` zwróci pustą kolekcję.

<a name="method-slice"></a>
#### `slice()` {.collection-method}

Metoda `slice` zwraca wycinek kolekcji zaczynający się od podanego indeksu:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$slice = $collection->slice(4);

$slice->all();

// [5, 6, 7, 8, 9, 10]
```

Jeśli chcesz ograniczyć rozmiar zwracanego wycinka, przekaz żądany rozmiar jako drugi argument do metody:

```php
$slice = $collection->slice(4, 2);

$slice->all();

// [5, 6]
```

Zwrócony wycinek domyślnie zachowa klucze. Jeśli nie chcesz zachować oryginalnych kluczy, możesz użyć metody [values](#method-values), aby je przenumerować.

<a name="method-sliding"></a>
#### `sliding()` {.collection-method}

Metoda `sliding` zwraca nową kolekcję fragmentów reprezentujących widok "przesuwnego okna" elementów w kolekcji:

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunks = $collection->sliding(2);

$chunks->toArray();

// [[1, 2], [2, 3], [3, 4], [4, 5]]
```

Jest to szczególnie przydatne w połączeniu z metodą [eachSpread](#method-eachspread):

```php
$transactions->sliding(2)->eachSpread(function (Collection $previous, Collection $current) {
    $current->total = $previous->total + $current->amount;
});
```

Opcjonalnie możesz przekazać drugą wartość "kroku", która określa odległość między pierwszym elementem każdego fragmentu:

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunks = $collection->sliding(3, step: 2);

$chunks->toArray();

// [[1, 2, 3], [3, 4, 5]]
```

<a name="method-sole"></a>
#### `sole()` {.collection-method}

Metoda `sole` zwraca pierwszy element w kolekcji, który przejdzie dany test prawdziwości, ale tylko wtedy, gdy test prawdziwości pasuje dokładnie do jednego elementu:

```php
collect([1, 2, 3, 4])->sole(function (int $value, int $key) {
    return $value === 2;
});

// 2
```

Możesz również przekazać parę klucz / wartość do metody `sole`, która zwróci pierwszy element w kolekcji, który pasuje do podanej pary, ale tylko jeśli dokładnie jeden element pasuje:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);

$collection->sole('product', 'Chair');

// ['product' => 'Chair', 'price' => 100]
```

Alternatywnie możesz również wywołać metodę `sole` bez argumentu, aby otrzymać pierwszy element w kolekcji, jeśli jest tylko jeden element:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
]);

$collection->sole();

// ['product' => 'Desk', 'price' => 200]
```

Jeśli w kolekcji nie ma elementów, które powinny zostać zwrócone przez metodę `sole`, zostanie zgłoszony wyjątek `\Illuminate\Collections\ItemNotFoundException`. Jeśli jest więcej niż jeden element, który powinien zostać zwrócony, zostanie zgłoszony wyjątek `\Illuminate\Collections\MultipleItemsFoundException`.

<a name="method-some"></a>
#### `some()` {.collection-method}

Alias dla metody [contains](#method-contains).

<a name="method-sort"></a>
#### `sort()` {.collection-method}

Metoda `sort` sortuje kolekcję. Posortowana kolekcja zachowuje oryginalne klucze tablicy, więc w poniższym przykładzie użyjemy metody [values](#method-values), aby zresetować klucze do kolejno ponumerowanych indeksów:

```php
$collection = collect([5, 3, 1, 2, 4]);

$sorted = $collection->sort();

$sorted->values()->all();

// [1, 2, 3, 4, 5]
```

Jeśli Twoje potrzeby sortowania są bardziej zaawansowane, możesz przekazać callback do `sort` z własnym algorytmem. Zapoznaj się z dokumentacją PHP dotyczącą [uasort](https://secure.php.net/manual/en/function.uasort.php#refsect1-function.uasort-parameters), której używa wewnętrznie metoda `sort` kolekcji.

> [!NOTE]
> Jeśli musisz posortować kolekcję zagnieżdżonych tablic lub obiektów, zobacz metody [sortBy](#method-sortby) i [sortByDesc](#method-sortbydesc).

<a name="method-sortby"></a>
#### `sortBy()` {.collection-method}

Metoda `sortBy` sortuje kolekcję według podanego klucza. Posortowana kolekcja zachowuje oryginalne klucze tablicy, więc w poniższym przykładzie użyjemy metody [values](#method-values), aby zresetować klucze do kolejno ponumerowanych indeksów:

```php
$collection = collect([
    ['name' => 'Desk', 'price' => 200],
    ['name' => 'Chair', 'price' => 100],
    ['name' => 'Bookcase', 'price' => 150],
]);

$sorted = $collection->sortBy('price');

$sorted->values()->all();

/*
    [
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
        ['name' => 'Desk', 'price' => 200],
    ]
*/
```

Metoda `sortBy` akceptuje [flagi sortowania](https://www.php.net/manual/en/function.sort.php) jako drugi argument:

```php
$collection = collect([
    ['title' => 'Item 1'],
    ['title' => 'Item 12'],
    ['title' => 'Item 3'],
]);

$sorted = $collection->sortBy('title', SORT_NATURAL);

$sorted->values()->all();

/*
    [
        ['title' => 'Item 1'],
        ['title' => 'Item 3'],
        ['title' => 'Item 12'],
    ]
*/
```

Alternatywnie możesz przekazać własne zamknięcie, aby określić, jak sortować wartości kolekcji:

```php
$collection = collect([
    ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
    ['name' => 'Chair', 'colors' => ['Black']],
    ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
]);

$sorted = $collection->sortBy(function (array $product, int $key) {
    return count($product['colors']);
});

$sorted->values()->all();

/*
    [
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]
*/
```

Jeśli chcesz posortować kolekcję według wielu atrybutów, możesz przekazać tablicę operacji sortowania do metody `sortBy`. Każda operacja sortowania powinna być tablicą składającą się z atrybutu, według którego chcesz sortować oraz kierunku żądanego sortowania:

```php
$collection = collect([
    ['name' => 'Taylor Otwell', 'age' => 34],
    ['name' => 'Abigail Otwell', 'age' => 30],
    ['name' => 'Taylor Otwell', 'age' => 36],
    ['name' => 'Abigail Otwell', 'age' => 32],
]);

$sorted = $collection->sortBy([
    ['name', 'asc'],
    ['age', 'desc'],
]);

$sorted->values()->all();

/*
    [
        ['name' => 'Abigail Otwell', 'age' => 32],
        ['name' => 'Abigail Otwell', 'age' => 30],
        ['name' => 'Taylor Otwell', 'age' => 36],
        ['name' => 'Taylor Otwell', 'age' => 34],
    ]
*/
```

Podczas sortowania kolekcji według wielu atrybutów, możesz również podać zamknięcia, które definiują każdą operację sortowania:

```php
$collection = collect([
    ['name' => 'Taylor Otwell', 'age' => 34],
    ['name' => 'Abigail Otwell', 'age' => 30],
    ['name' => 'Taylor Otwell', 'age' => 36],
    ['name' => 'Abigail Otwell', 'age' => 32],
]);

$sorted = $collection->sortBy([
    fn (array $a, array $b) => $a['name'] <=> $b['name'],
    fn (array $a, array $b) => $b['age'] <=> $a['age'],
]);

$sorted->values()->all();

/*
    [
        ['name' => 'Abigail Otwell', 'age' => 32],
        ['name' => 'Abigail Otwell', 'age' => 30],
        ['name' => 'Taylor Otwell', 'age' => 36],
        ['name' => 'Taylor Otwell', 'age' => 34],
    ]
*/
```

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {.collection-method}

Ta metoda ma tę samą sygnaturę co metoda [sortBy](#method-sortby), ale będzie sortować kolekcję w odwrotnej kolejności.

<a name="method-sortdesc"></a>
#### `sortDesc()` {.collection-method}

Ta metoda posortuje kolekcję w odwrotnej kolejności niż metoda [sort](#method-sort):

```php
$collection = collect([5, 3, 1, 2, 4]);

$sorted = $collection->sortDesc();

$sorted->values()->all();

// [5, 4, 3, 2, 1]
```

W przeciwieństwie do `sort`, nie możesz przekazać zamknięcia do `sortDesc`. Zamiast tego należy użyć metody [sort](#method-sort) i odwrócić porównanie.

<a name="method-sortkeys"></a>
#### `sortKeys()` {.collection-method}

Metoda `sortKeys` sortuje kolekcję według kluczy bazowej tablicy asocjacyjnej:

```php
$collection = collect([
    'id' => 22345,
    'first' => 'John',
    'last' => 'Doe',
]);

$sorted = $collection->sortKeys();

$sorted->all();

/*
    [
        'first' => 'John',
        'id' => 22345,
        'last' => 'Doe',
    ]
*/
```

<a name="method-sortkeysdesc"></a>
#### `sortKeysDesc()` {.collection-method}

Ta metoda ma tę samą sygnaturę co metoda [sortKeys](#method-sortkeys), ale będzie sortować kolekcję w odwrotnej kolejności.

<a name="method-sortkeysusing"></a>
#### `sortKeysUsing()` {.collection-method}

Metoda `sortKeysUsing` sortuje kolekcję według kluczy bazowej tablicy asocjacyjnej za pomocą callbacka:

```php
$collection = collect([
    'ID' => 22345,
    'first' => 'John',
    'last' => 'Doe',
]);

$sorted = $collection->sortKeysUsing('strnatcasecmp');

$sorted->all();

/*
    [
        'first' => 'John',
        'ID' => 22345,
        'last' => 'Doe',
    ]
*/
```

Callback musi być funkcją porównawczą, która zwraca liczbę całkowitą mniejszą, równą lub większą od zera. Więcej informacji znajdziesz w dokumentacji PHP dotyczącej [uksort](https://www.php.net/manual/en/function.uksort.php#refsect1-function.uksort-parameters), która jest funkcją PHP wykorzystywaną wewnętrznie przez metodę `sortKeysUsing`.

<a name="method-splice"></a>
#### `splice()` {.collection-method}

Metoda `splice` usuwa i zwraca wycinek elementów rozpoczynający się od określonego indeksu:

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunk = $collection->splice(2);

$chunk->all();

// [3, 4, 5]

$collection->all();

// [1, 2]
```

Możesz przekazać drugi argument, aby ograniczyć rozmiar wynikowej kolekcji:

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunk = $collection->splice(2, 1);

$chunk->all();

// [3]

$collection->all();

// [1, 2, 4, 5]
```

Ponadto możesz przekazać trzeci argument zawierający nowe elementy, które zastąpią elementy usunięte z kolekcji:

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunk = $collection->splice(2, 1, [10, 11]);

$chunk->all();

// [3]

$collection->all();

// [1, 2, 10, 11, 4, 5]
```

<a name="method-split"></a>
#### `split()` {.collection-method}

Metoda `split` dzieli kolekcję na podaną liczbę grup:

```php
$collection = collect([1, 2, 3, 4, 5]);

$groups = $collection->split(3);

$groups->all();

// [[1, 2], [3, 4], [5]]
```

<a name="method-splitin"></a>
#### `splitIn()` {.collection-method}

Metoda `splitIn` dzieli kolekcję na podaną liczbę grup, wypełniając całkowicie grupy niekoncowe przed przydzieleniem pozostałych elementów do końcowej grupy:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$groups = $collection->splitIn(3);

$groups->all();

// [[1, 2, 3, 4], [5, 6, 7, 8], [9, 10]]
```

<a name="method-sum"></a>
#### `sum()` {.collection-method}

Metoda `sum` zwraca sumę wszystkich elementów w kolekcji:

```php
collect([1, 2, 3, 4, 5])->sum();

// 15
```

Jeśli kolekcja zawiera zagnieżdżone tablice lub obiekty, należy przekazać klucz, który będzie używany do określenia, które wartości zsumować:

```php
$collection = collect([
    ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
    ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
]);

$collection->sum('pages');

// 1272
```

Ponadto możesz przekazać własne zamknięcie, aby określić, które wartości kolekcji zsumować:

```php
$collection = collect([
    ['name' => 'Chair', 'colors' => ['Black']],
    ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
    ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
]);

$collection->sum(function (array $product) {
    return count($product['colors']);
});

// 6
```

<a name="method-take"></a>
#### `take()` {.collection-method}

Metoda `take` zwraca nową kolekcję z określoną liczbą elementów:

```php
$collection = collect([0, 1, 2, 3, 4, 5]);

$chunk = $collection->take(3);

$chunk->all();

// [0, 1, 2]
```

Możesz również przekazać ujemną liczbę całkowitą, aby pobrać określoną liczbę elementów z końca kolekcji:

```php
$collection = collect([0, 1, 2, 3, 4, 5]);

$chunk = $collection->take(-2);

$chunk->all();

// [4, 5]
```

<a name="method-takeuntil"></a>
#### `takeUntil()` {.collection-method}

Metoda `takeUntil` zwraca elementy w kolekcji, dopóki podany callback nie zwróci `true`:

```php
$collection = collect([1, 2, 3, 4]);

$subset = $collection->takeUntil(function (int $item) {
    return $item >= 3;
});

$subset->all();

// [1, 2]
```

Możesz również przekazać prostą wartość do metody `takeUntil`, aby pobrać elementy aż do momentu znalezienia podanej wartości:

```php
$collection = collect([1, 2, 3, 4]);

$subset = $collection->takeUntil(3);

$subset->all();

// [1, 2]
```

> [!WARNING]
> Jeśli podana wartość nie zostanie znaleziona lub callback nigdy nie zwróci `true`, metoda `takeUntil` zwróci wszystkie elementy w kolekcji.

<a name="method-takewhile"></a>
#### `takeWhile()` {.collection-method}

Metoda `takeWhile` zwraca elementy w kolekcji, dopóki podany callback nie zwróci `false`:

```php
$collection = collect([1, 2, 3, 4]);

$subset = $collection->takeWhile(function (int $item) {
    return $item < 3;
});

$subset->all();

// [1, 2]
```

> [!WARNING]
> Jeśli callback nigdy nie zwróci `false`, metoda `takeWhile` zwróci wszystkie elementy w kolekcji.

<a name="method-tap"></a>
#### `tap()` {.collection-method}

Metoda `tap` przekazuje kolekcję do podanego callbacka, pozwalając "podejrzeć" kolekcję w określonym punkcie i zrobić coś z elementami, nie wpływając na samą kolekcję. Kolekcja jest następnie zwracana przez metodę `tap`:

```php
collect([2, 4, 3, 1, 5])
    ->sort()
    ->tap(function (Collection $collection) {
        Log::debug('Values after sorting', $collection->values()->all());
    })
    ->shift();

// 1
```

<a name="method-times"></a>
#### `times()` {.collection-method}

Statyczna metoda `times` tworzy nową kolekcję przez wywołanie podanego zamknięcia określoną liczbę razy:

```php
$collection = Collection::times(10, function (int $number) {
    return $number * 9;
});

$collection->all();

// [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]
```

<a name="method-toarray"></a>
#### `toArray()` {.collection-method}

Metoda `toArray` konwertuje kolekcję na zwykłą tablicę PHP `array`. Jeśli wartości kolekcji są modelami [Eloquent](/docs/{{version}}/eloquent), modele również zostaną przekonwertowane na tablice:

```php
$collection = collect(['name' => 'Desk', 'price' => 200]);

$collection->toArray();

/*
    [
        ['name' => 'Desk', 'price' => 200],
    ]
*/
```

> [!WARNING]
> `toArray` konwertuje również wszystkie zagnieżdżone obiekty kolekcji, które są instancją `Arrayable`, na tablicę. Jeśli chcesz otrzymać surową tablicę leżącą u podstaw kolekcji, użyj zamiast tego metody [all](#method-all).

<a name="method-tojson"></a>
#### `toJson()` {.collection-method}

Metoda `toJson` konwertuje kolekcję na serializowany ciąg JSON:

```php
$collection = collect(['name' => 'Desk', 'price' => 200]);

$collection->toJson();

// '{"name":"Desk", "price":200}'
```

<a name="method-to-pretty-json"></a>
#### `toPrettyJson()` {.collection-method}

Metoda `toPrettyJson` konwertuje kolekcję na sformatowany ciąg JSON używając opcji `JSON_PRETTY_PRINT`:

```php
$collection = collect(['name' => 'Desk', 'price' => 200]);

$collection->toPrettyJson();
```

<a name="method-transform"></a>
#### `transform()` {.collection-method}

Metoda `transform` iteruje po kolekcji i wywołuje podany callback z każdym elementem w kolekcji. Elementy w kolekcji zostaną zastąpione wartościami zwracanymi przez callback:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->transform(function (int $item, int $key) {
    return $item * 2;
});

$collection->all();

// [2, 4, 6, 8, 10]
```

> [!WARNING]
> W przeciwieństwie do większości innych metod kolekcji, `transform` modyfikuje samą kolekcję. Jeśli chcesz zamiast tego utworzyć nową kolekcję, użyj metody [map](#method-map).

<a name="method-undot"></a>
#### `undot()` {.collection-method}

Metoda `undot` rozszerza jednowymiarową kolekcję, która używa notacji "kropkowej", na wielowymiarową kolekcję:

```php
$person = collect([
    'name.first_name' => 'Marie',
    'name.last_name' => 'Valentine',
    'address.line_1' => '2992 Eagle Drive',
    'address.line_2' => '',
    'address.suburb' => 'Detroit',
    'address.state' => 'MI',
    'address.postcode' => '48219'
]);

$person = $person->undot();

$person->toArray();

/*
    [
        "name" => [
            "first_name" => "Marie",
            "last_name" => "Valentine",
        ],
        "address" => [
            "line_1" => "2992 Eagle Drive",
            "line_2" => "",
            "suburb" => "Detroit",
            "state" => "MI",
            "postcode" => "48219",
        ],
    ]
*/
```

<a name="method-union"></a>
#### `union()` {.collection-method}

Metoda `union` dodaje podaną tablicę do kolekcji. Jeśli podana tablica zawiera klucze, które już są w oryginalnej kolekcji, będą preferowane wartości z oryginalnej kolekcji:

```php
$collection = collect([1 => ['a'], 2 => ['b']]);

$union = $collection->union([3 => ['c'], 1 => ['d']]);

$union->all();

// [1 => ['a'], 2 => ['b'], 3 => ['c']]
```

<a name="method-unique"></a>
#### `unique()` {.collection-method}

Metoda `unique` zwraca wszystkie unikalne elementy w kolekcji. Zwrócona kolekcja zachowuje oryginalne klucze tablicy, więc w poniższym przykładzie użyjemy metody [values](#method-values), aby zresetować klucze do kolejno ponumerowanych indeksów:

```php
$collection = collect([1, 1, 2, 2, 3, 4, 2]);

$unique = $collection->unique();

$unique->values()->all();

// [1, 2, 3, 4]
```

Podczas pracy z zagnieżdżonymi tablicami lub obiektami, możesz określić klucz używany do określenia unikalności:

```php
$collection = collect([
    ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
    ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
    ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
    ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
    ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
]);

$unique = $collection->unique('brand');

$unique->values()->all();

/*
    [
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
    ]
*/
```

Na koniec możesz również przekazać własne zamknięcie do metody `unique`, aby określić, która wartość powinna określać unikalność elementu:

```php
$unique = $collection->unique(function (array $item) {
    return $item['brand'].$item['type'];
});

$unique->values()->all();

/*
    [
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]
*/
```

Metoda `unique` używa "luźnych" porównań podczas sprawdzania wartości elementów, co oznacza, że ciąg z wartością całkowitą będzie uważany za równy liczbie całkowitej o tej samej wartości. Użyj metody [uniqueStrict](#method-uniquestrict), aby filtrować za pomocą "ściemu" porównania.

> [!NOTE]
> Zachowanie tej metody jest modyfikowane podczas używania [Kolekcji Eloquent](/docs/{{version}}/eloquent-collections#method-unique).

<a name="method-uniquestrict"></a>
#### `uniqueStrict()` {.collection-method}

Ta metoda ma tę samą sygnaturę co metoda [unique](#method-unique); jednak wszystkie wartości są porównywane za pomocą "ściego" porównania.

<a name="method-unless"></a>
#### `unless()` {.collection-method}

Metoda `unless` wykona podany callback, chyba że pierwszy argument przekazany do metody jest oceniany jako `true`. Instancja kolekcji oraz pierwszy argument przekazany do metody `unless` zostaną dostarczone do zamknięcia:

```php
$collection = collect([1, 2, 3]);

$collection->unless(true, function (Collection $collection, bool $value) {
    return $collection->push(4);
});

$collection->unless(false, function (Collection $collection, bool $value) {
    return $collection->push(5);
});

$collection->all();

// [1, 2, 3, 5]
```

Drugi callback może zostać przekazany do metody `unless`. Drugi callback zostanie wykonany, gdy pierwszy argument przekazany do metody `unless` jest oceniany jako `true`:

```php
$collection = collect([1, 2, 3]);

$collection->unless(true, function (Collection $collection, bool $value) {
    return $collection->push(4);
}, function (Collection $collection, bool $value) {
    return $collection->push(5);
});

$collection->all();

// [1, 2, 3, 5]
```

Dla odwrotności metody `unless`, zobacz metodę [when](#method-when).

<a name="method-unlessempty"></a>
#### `unlessEmpty()` {.collection-method}

Alias dla metody [whenNotEmpty](#method-whennotempty).

<a name="method-unlessnotempty"></a>
#### `unlessNotEmpty()` {.collection-method}

Alias dla metody [whenEmpty](#method-whenempty).

<a name="method-unwrap"></a>
#### `unwrap()` {.collection-method}

Statyczna metoda `unwrap` zwraca bazowe elementy kolekcji z podanej wartości, gdy ma to zastosowanie:

```php
Collection::unwrap(collect('John Doe'));

// ['John Doe']

Collection::unwrap(['John Doe']);

// ['John Doe']

Collection::unwrap('John Doe');

// 'John Doe'
```

<a name="method-value"></a>
#### `value()` {.collection-method}

Metoda `value` pobiera podaną wartość z pierwszego elementu kolekcji:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Speaker', 'price' => 400],
]);

$value = $collection->value('price');

// 200
```

<a name="method-values"></a>
#### `values()` {.collection-method}

Metoda `values` zwraca nową kolekcję z kluczami zresetowanymi do kolejnych liczb całkowitych:

```php
$collection = collect([
    10 => ['product' => 'Desk', 'price' => 200],
    11 => ['product' => 'Desk', 'price' => 200],
]);

$values = $collection->values();

$values->all();

/*
    [
        0 => ['product' => 'Desk', 'price' => 200],
        1 => ['product' => 'Desk', 'price' => 200],
    ]
*/
```

<a name="method-when"></a>
#### `when()` {.collection-method}

Metoda `when` wykona podany callback, gdy pierwszy argument przekazany do metody jest oceniany jako `true`. Instancja kolekcji oraz pierwszy argument przekazany do metody `when` zostaną dostarczone do zamknięcia:

```php
$collection = collect([1, 2, 3]);

$collection->when(true, function (Collection $collection, bool $value) {
    return $collection->push(4);
});

$collection->when(false, function (Collection $collection, bool $value) {
    return $collection->push(5);
});

$collection->all();

// [1, 2, 3, 4]
```

Drugi callback może zostać przekazany do metody `when`. Drugi callback zostanie wykonany, gdy pierwszy argument przekazany do metody `when` jest oceniany jako `false`:

```php
$collection = collect([1, 2, 3]);

$collection->when(false, function (Collection $collection, bool $value) {
    return $collection->push(4);
}, function (Collection $collection, bool $value) {
    return $collection->push(5);
});

$collection->all();

// [1, 2, 3, 5]
```

Dla odwrotności metody `when`, zobacz metodę [unless](#method-unless).

<a name="method-whenempty"></a>
#### `whenEmpty()` {.collection-method}

Metoda `whenEmpty` wykona podane callback, gdy kolekcja jest pusta:

```php
$collection = collect(['Michael', 'Tom']);

$collection->whenEmpty(function (Collection $collection) {
    return $collection->push('Adam');
});

$collection->all();

// ['Michael', 'Tom']

$collection = collect();

$collection->whenEmpty(function (Collection $collection) {
    return $collection->push('Adam');
});

$collection->all();

// ['Adam']
```

Drugie domknięcie może być przekazane do metody `whenEmpty`, które zostanie wykonane, gdy kolekcja nie jest pusta:

```php
$collection = collect(['Michael', 'Tom']);

$collection->whenEmpty(function (Collection $collection) {
    return $collection->push('Adam');
}, function (Collection $collection) {
    return $collection->push('Taylor');
});

$collection->all();

// ['Michael', 'Tom', 'Taylor']
```

Dla odwrotności metody `whenEmpty`, zobacz metodę [whenNotEmpty](#method-whennotempty).

<a name="method-whennotempty"></a>
#### `whenNotEmpty()` {.collection-method}

Metoda `whenNotEmpty` wykona podane callback, gdy kolekcja nie jest pusta:

```php
$collection = collect(['Michael', 'Tom']);

$collection->whenNotEmpty(function (Collection $collection) {
    return $collection->push('Adam');
});

$collection->all();

// ['Michael', 'Tom', 'Adam']

$collection = collect();

$collection->whenNotEmpty(function (Collection $collection) {
    return $collection->push('Adam');
});

$collection->all();

// []
```

Drugie domknięcie może być przekazane do metody `whenNotEmpty`, które zostanie wykonane, gdy kolekcja jest pusta:

```php
$collection = collect();

$collection->whenNotEmpty(function (Collection $collection) {
    return $collection->push('Adam');
}, function (Collection $collection) {
    return $collection->push('Taylor');
});

$collection->all();

// ['Taylor']
```

Dla odwrotności metody `whenNotEmpty`, zobacz metodę [whenEmpty](#method-whenempty).

<a name="method-where"></a>
#### `where()` {.collection-method}

Metoda `where` filtruje kolekcję według podanej pary klucz / wartość:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->where('price', 100);

$filtered->all();

/*
    [
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Door', 'price' => 100],
    ]
*/
```

Metoda `where` używa "luźnych" porównań podczas sprawdzania wartości elementów, co oznacza, że ciąg znaków z wartością całkowitą będzie uważany za równy liczbie całkowitej o tej samej wartości. Użyj metody [whereStrict](#method-wherestrict) do filtrowania za pomocą "ścisłych" porównań lub metod [whereNull](#method-wherenull) i [whereNotNull](#method-wherenotnull) do filtrowania wartości `null`.

Opcjonalnie możesz przekazać operator porównania jako drugi parametr. Obsługiwane operatory to: '===', '!==', '!=', '==', '=', '<>', '>', '<', '>=' i '<=':

```php
$collection = collect([
    ['name' => 'Jim', 'platform' => 'Mac'],
    ['name' => 'Sally', 'platform' => 'Mac'],
    ['name' => 'Sue', 'platform' => 'Linux'],
]);

$filtered = $collection->where('platform', '!=', 'Linux');

$filtered->all();

/*
    [
        ['name' => 'Jim', 'platform' => 'Mac'],
        ['name' => 'Sally', 'platform' => 'Mac'],
    ]
*/
```

<a name="method-wherestrict"></a>
#### `whereStrict()` {.collection-method}

Ta metoda ma taki sam podpis jak metoda [where](#method-where); jednakże wszystkie wartości są porównywane za pomocą "ścisłych" porównań.

<a name="method-wherebetween"></a>
#### `whereBetween()` {.collection-method}

Metoda `whereBetween` filtruje kolekcję, określając, czy określona wartość elementu znajduje się w podanym zakresie:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 80],
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Pencil', 'price' => 30],
    ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->whereBetween('price', [100, 200]);

$filtered->all();

/*
    [
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]
*/
```

<a name="method-wherein"></a>
#### `whereIn()` {.collection-method}

Metoda `whereIn` usuwa elementy z kolekcji, które nie mają określonej wartości elementu, która jest zawarta w podanej tablicy:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->whereIn('price', [150, 200]);

$filtered->all();

/*
    [
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Bookcase', 'price' => 150],
    ]
*/
```

Metoda `whereIn` używa "luźnych" porównań podczas sprawdzania wartości elementów, co oznacza, że ciąg znaków z wartością całkowitą będzie uważany za równy liczbie całkowitej o tej samej wartości. Użyj metody [whereInStrict](#method-whereinstrict) do filtrowania za pomocą "ścisłych" porównań.

<a name="method-whereinstrict"></a>
#### `whereInStrict()` {.collection-method}

Ta metoda ma taki sam podpis jak metoda [whereIn](#method-wherein); jednakże wszystkie wartości są porównywane za pomocą "ścisłych" porównań.

<a name="method-whereinstanceof"></a>
#### `whereInstanceOf()` {.collection-method}

Metoda `whereInstanceOf` filtruje kolekcję według podanego typu klasy:

```php
use App\Models\User;
use App\Models\Post;

$collection = collect([
    new User,
    new User,
    new Post,
]);

$filtered = $collection->whereInstanceOf(User::class);

$filtered->all();

// [App\Models\User, App\Models\User]
```

<a name="method-wherenotbetween"></a>
#### `whereNotBetween()` {.collection-method}

Metoda `whereNotBetween` filtruje kolekcję, określając, czy określona wartość elementu znajduje się poza podanym zakresem:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 80],
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Pencil', 'price' => 30],
    ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->whereNotBetween('price', [100, 200]);

$filtered->all();

/*
    [
        ['product' => 'Chair', 'price' => 80],
        ['product' => 'Pencil', 'price' => 30],
    ]
*/
```

<a name="method-wherenotin"></a>
#### `whereNotIn()` {.collection-method}

Metoda `whereNotIn` usuwa elementy z kolekcji, które mają określoną wartość elementu, która jest zawarta w podanej tablicy:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->whereNotIn('price', [150, 200]);

$filtered->all();

/*
    [
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Door', 'price' => 100],
    ]
*/
```

Metoda `whereNotIn` używa "luźnych" porównań podczas sprawdzania wartości elementów, co oznacza, że ciąg znaków z wartością całkowitą będzie uważany za równy liczbie całkowitej o tej samej wartości. Użyj metody [whereNotInStrict](#method-wherenotinstrict) do filtrowania za pomocą "ścisłych" porównań.

<a name="method-wherenotinstrict"></a>
#### `whereNotInStrict()` {.collection-method}

Ta metoda ma taki sam podpis jak metoda [whereNotIn](#method-wherenotin); jednakże wszystkie wartości są porównywane za pomocą "ścisłych" porównań.

<a name="method-wherenotnull"></a>
#### `whereNotNull()` {.collection-method}

Metoda `whereNotNull` zwraca elementy z kolekcji, gdzie podany klucz nie jest `null`:

```php
$collection = collect([
    ['name' => 'Desk'],
    ['name' => null],
    ['name' => 'Bookcase'],
    ['name' => 0],
    ['name' => ''],
]);

$filtered = $collection->whereNotNull('name');

$filtered->all();

/*
    [
        ['name' => 'Desk'],
        ['name' => 'Bookcase'],
        ['name' => 0],
        ['name' => ''],
    ]
*/
```

<a name="method-wherenull"></a>
#### `whereNull()` {.collection-method}

Metoda `whereNull` zwraca elementy z kolekcji, gdzie podany klucz jest `null`:

```php
$collection = collect([
    ['name' => 'Desk'],
    ['name' => null],
    ['name' => 'Bookcase'],
    ['name' => 0],
    ['name' => ''],
]);

$filtered = $collection->whereNull('name');

$filtered->all();

/*
    [
        ['name' => null],
    ]
*/
```

<a name="method-wrap"></a>
#### `wrap()` {.collection-method}

Statyczna metoda `wrap` owija podaną wartość w kolekcję, gdy jest to możliwe:

```php
use Illuminate\Support\Collection;

$collection = Collection::wrap('John Doe');

$collection->all();

// ['John Doe']

$collection = Collection::wrap(['John Doe']);

$collection->all();

// ['John Doe']

$collection = Collection::wrap(collect('John Doe'));

$collection->all();

// ['John Doe']
```

<a name="method-zip"></a>
#### `zip()` {.collection-method}

Metoda `zip` łączy razem wartości podanej tablicy z wartościami oryginalnej kolekcji pod ich odpowiadającymi indeksami:

```php
$collection = collect(['Chair', 'Desk']);

$zipped = $collection->zip([100, 200]);

$zipped->all();

// [['Chair', 100], ['Desk', 200]]
```

<a name="higher-order-messages"></a>
## Komunikaty wyższego rzędu

Kolekcje zapewniają również wsparcie dla "komunikatów wyższego rzędu", które są skrótami do wykonywania wspólnych działań na kolekcjach. Metody kolekcji, które zapewniają komunikaty wyższego rzędu to: [average](#method-average), [avg](#method-avg), [contains](#method-contains), [each](#method-each), [every](#method-every), [filter](#method-filter), [first](#method-first), [flatMap](#method-flatmap), [groupBy](#method-groupby), [keyBy](#method-keyby), [map](#method-map), [max](#method-max), [min](#method-min), [partition](#method-partition), [reject](#method-reject), [skipUntil](#method-skipuntil), [skipWhile](#method-skipwhile), [some](#method-some), [sortBy](#method-sortby), [sortByDesc](#method-sortbydesc), [sum](#method-sum), [takeUntil](#method-takeuntil), [takeWhile](#method-takewhile) i [unique](#method-unique).

Każdy komunikat wyższego rzędu może być dostępny jako dynamiczna właściwość instancji kolekcji. Na przykład użyjmy komunikatu wyższego rzędu `each`, aby wywołać metodę na każdym obiekcie w kolekcji:

```php
use App\Models\User;

$users = User::where('votes', '>', 500)->get();

$users->each->markAsVip();
```

Podobnie możemy użyć komunikatu wyższego rzędu `sum`, aby zgromadzić całkowitą liczbę "głosów" dla kolekcji użytkowników:

```php
$users = User::where('group', 'Development')->get();

return $users->sum->votes;
```

<a name="lazy-collections"></a>
## Leniwe kolekcje

<a name="lazy-collection-introduction"></a>
### Wprowadzenie

> [!WARNING]
> Przed poznaniem leniwych kolekcji Laravela, poświęć trochę czasu na zapoznanie się z [generatorami PHP](https://www.php.net/manual/en/language.generators.overview.php).

Aby uzupełnić już potężną klasę `Collection`, klasa `LazyCollection` wykorzystuje [generatory](https://www.php.net/manual/en/language.generators.overview.php) PHP, aby umożliwić pracę z bardzo dużymi zbiorami danych przy jednoczesnym utrzymaniu niskiego zużycia pamięci.

Na przykład wyobraź sobie, że Twoja aplikacja musi przetworzyć wielogigabajtowy plik dziennika, wykorzystując metody kolekcji Laravela do analizy dzienników. Zamiast wczytywać cały plik do pamięci naraz, leniwe kolekcje mogą być używane do przechowywania tylko małej części pliku w pamięci w danym czasie:

```php
use App\Models\LogEntry;
use Illuminate\Support\LazyCollection;

LazyCollection::make(function () {
    $handle = fopen('log.txt', 'r');

    while (($line = fgets($handle)) !== false) {
        yield $line;
    }

    fclose($handle);
})->chunk(4)->map(function (array $lines) {
    return LogEntry::fromLines($lines);
})->each(function (LogEntry $logEntry) {
    // Przetwarzanie wpisu dziennika...
});
```

Lub wyobraź sobie, że musisz przejść przez 10 000 modeli Eloquent. Używając tradycyjnych kolekcji Laravela, wszystkie 10 000 modeli Eloquent musi zostać załadowanych do pamięci w tym samym czasie:

```php
use App\Models\User;

$users = User::all()->filter(function (User $user) {
    return $user->id > 500;
});
```

Jednakże metoda `cursor` konstruktora zapytań zwraca instancję `LazyCollection`. Pozwala to na uruchomienie tylko jednego zapytania do bazy danych, ale także na utrzymywanie tylko jednego modelu Eloquent załadowanego do pamięci na raz. W tym przykładzie callback `filter` nie jest wykonywany, dopóki nie przejdziemy przez każdego użytkownika indywidualnie, pozwalając na drastyczne zmniejszenie zużycia pamięci:

```php
use App\Models\User;

$users = User::cursor()->filter(function (User $user) {
    return $user->id > 500;
});

foreach ($users as $user) {
    echo $user->id;
}
```

<a name="creating-lazy-collections"></a>
### Tworzenie leniwych kolekcji

Aby utworzyć instancję leniwej kolekcji, należy przekazać funkcję generatora PHP do metody `make` kolekcji:

```php
use Illuminate\Support\LazyCollection;

LazyCollection::make(function () {
    $handle = fopen('log.txt', 'r');

    while (($line = fgets($handle)) !== false) {
        yield $line;
    }

    fclose($handle);
});
```

<a name="the-enumerable-contract"></a>
### Kontrakt Enumerable

Prawie wszystkie metody dostępne w klasie `Collection` są również dostępne w klasie `LazyCollection`. Obie te klasy implementują kontrakt `Illuminate\Support\Enumerable`, który definiuje następujące metody:

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
</style>

<div class="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[chunkWhile](#method-chunkwhile)
[collapse](#method-collapse)
[collect](#method-collect)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsStrict](#method-containsstrict)
[count](#method-count)
[countBy](#method-countBy)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffKeys](#method-diffkeys)
[dump](#method-dump)
[duplicates](#method-duplicates)
[duplicatesStrict](#method-duplicatesstrict)
[each](#method-each)
[eachSpread](#method-eachspread)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[firstOrFail](#method-first-or-fail)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectAssoc](#method-intersectAssoc)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[join](#method-join)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[macro](#method-macro)
[make](#method-make)
[map](#method-map)
[mapInto](#method-mapinto)
[mapSpread](#method-mapspread)
[mapToGroups](#method-maptogroups)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[mergeRecursive](#method-mergerecursive)
[min](#method-min)
[mode](#method-mode)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[pipe](#method-pipe)
[pluck](#method-pluck)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[replace](#method-replace)
[replaceRecursive](#method-replacerecursive)
[reverse](#method-reverse)
[search](#method-search)
[shuffle](#method-shuffle)
[skip](#method-skip)
[slice](#method-slice)
[sole](#method-sole)
[some](#method-some)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[sortKeys](#method-sortkeys)
[sortKeysDesc](#method-sortkeysdesc)
[split](#method-split)
[sum](#method-sum)
[take](#method-take)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unlessEmpty](#method-unlessempty)
[unlessNotEmpty](#method-unlessnotempty)
[unwrap](#method-unwrap)
[values](#method-values)
[when](#method-when)
[whenEmpty](#method-whenempty)
[whenNotEmpty](#method-whennotempty)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereBetween](#method-wherebetween)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereInstanceOf](#method-whereinstanceof)
[whereNotBetween](#method-wherenotbetween)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[wrap](#method-wrap)
[zip](#method-zip)

</div>

> [!WARNING]
> Metody, które mutują kolekcję (takie jak `shift`, `pop`, `prepend` itp.) **nie** są dostępne w klasie `LazyCollection`.

<a name="lazy-collection-methods"></a>
### Metody leniwych kolekcji

Oprócz metod zdefiniowanych w kontrakcie `Enumerable`, klasa `LazyCollection` zawiera następujące metody:

<a name="method-takeUntilTimeout"></a>
#### `takeUntilTimeout()` {.collection-method}

Metoda `takeUntilTimeout` zwraca nową leniwą kolekcję, która będzie wyliczać wartości do określonego czasu. Po tym czasie kolekcja przestanie wyliczać:

```php
$lazyCollection = LazyCollection::times(INF)
    ->takeUntilTimeout(now()->plus(minutes: 1));

$lazyCollection->each(function (int $number) {
    dump($number);

    sleep(1);
});

// 1
// 2
// ...
// 58
// 59
```

Aby zilustrować użycie tej metody, wyobraź sobie aplikację, która przesyła faktury z bazy danych za pomocą kursora. Możesz zdefiniować [zaplanowane zadanie](/docs/{{version}}/scheduling), które uruchamia się co 15 minut i przetwarza faktury przez maksymalnie 14 minut:

```php
use App\Models\Invoice;
use Illuminate\Support\Carbon;

Invoice::pending()->cursor()
    ->takeUntilTimeout(
        Carbon::createFromTimestamp(LARAVEL_START)->add(14, 'minutes')
    )
    ->each(fn (Invoice $invoice) => $invoice->submit());
```

<a name="method-tapEach"></a>
#### `tapEach()` {.collection-method}

While the `each` method calls the given callback for each item in the collection right away, the `tapEach` method only calls the given callback as the items are being pulled out of the list one by one:

```php
// Nothing has been dumped so far...
$lazyCollection = LazyCollection::times(INF)->tapEach(function (int $value) {
    dump($value);
});

// Three items are dumped...
$array = $lazyCollection->take(3)->all();

// 1
// 2
// 3
```

<a name="method-throttle"></a>
#### `throttle()` {.collection-method}

The `throttle` method will throttle the lazy collection such that each value is returned after the specified number of seconds. This method is especially useful for situations where you may be interacting with external APIs that rate limit incoming requests:

```php
use App\Models\User;

User::where('vip', true)
    ->cursor()
    ->throttle(seconds: 1)
    ->each(function (User $user) {
        // Call external API...
    });
```

<a name="method-remember"></a>
#### `remember()` {.collection-method}

The `remember` method returns a new lazy collection that will remember any values that have already been enumerated and will not retrieve them again on subsequent collection enumerations:

```php
// No query has been executed yet...
$users = User::cursor()->remember();

// The query is executed...
// The first 5 users are hydrated from the database...
$users->take(5)->all();

// First 5 users come from the collection's cache...
// The rest are hydrated from the database...
$users->take(20)->all();
```

<a name="method-with-heartbeat"></a>
#### `withHeartbeat()` {.collection-method}

The `withHeartbeat` method allows you to execute a callback at regular time intervals while a lazy collection is being enumerated. This is particularly useful for long-running operations that require periodic maintenance tasks, such as extending locks or sending progress updates:

```php
use Carbon\CarbonInterval;
use Illuminate\Support\Facades\Cache;

$lock = Cache::lock('generate-reports', seconds: 60 * 5);

if ($lock->get()) {
    try {
        Report::where('status', 'pending')
            ->lazy()
            ->withHeartbeat(
                CarbonInterval::minutes(4),
                fn () => $lock->extend(CarbonInterval::minutes(5))
            )
            ->each(fn ($report) => $report->process());
    } finally {
        $lock->release();
    }
}
```
