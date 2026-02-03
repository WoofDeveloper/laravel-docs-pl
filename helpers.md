# Helpery

- [Wprowadzenie](#introduction)
- [Dostępne metody](#available-methods)
- [Inne narzędzia](#other-utilities)
    - [Benchmarking](#benchmarking)
    - [Daty i czas](#dates)
    - [Funkcje odroczone](#deferred-functions)
    - [Loteria](#lottery)
    - [Pipeline](#pipeline)
    - [Sleep](#sleep)
    - [Timebox](#timebox)
    - [URI](#uri)

<a name="introduction"></a>
## Wprowadzenie

Laravel zawiera różnorodne globalne funkcje "helperowe" PHP. Wiele z tych funkcji jest używanych przez sam framework; jednak możesz swobodnie używać ich w swoich własnych aplikacjach, jeśli uznasz je za wygodne.

<a name="available-methods"></a>
## Dostępne metody

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

<a name="arrays-and-objects-method-list"></a>
### Tablice i obiekty

<div class="collection-method-list" markdown="1">

[Arr::accessible](#method-array-accessible)
[Arr::add](#method-array-add)
[Arr::array](#method-array-array)
[Arr::boolean](#method-array-boolean)
[Arr::collapse](#method-array-collapse)
[Arr::crossJoin](#method-array-crossjoin)
[Arr::divide](#method-array-divide)
[Arr::dot](#method-array-dot)
[Arr::every](#method-array-every)
[Arr::except](#method-array-except)
[Arr::exceptValues](#method-array-except-values)
[Arr::exists](#method-array-exists)
[Arr::first](#method-array-first)
[Arr::flatten](#method-array-flatten)
[Arr::float](#method-array-float)
[Arr::forget](#method-array-forget)
[Arr::from](#method-array-from)
[Arr::get](#method-array-get)
[Arr::has](#method-array-has)
[Arr::hasAll](#method-array-hasall)
[Arr::hasAny](#method-array-hasany)
[Arr::integer](#method-array-integer)
[Arr::isAssoc](#method-array-isassoc)
[Arr::isList](#method-array-islist)
[Arr::join](#method-array-join)
[Arr::keyBy](#method-array-keyby)
[Arr::last](#method-array-last)
[Arr::map](#method-array-map)
[Arr::mapSpread](#method-array-map-spread)
[Arr::mapWithKeys](#method-array-map-with-keys)
[Arr::only](#method-array-only)
[Arr::onlyValues](#method-array-only-values)
[Arr::partition](#method-array-partition)
[Arr::pluck](#method-array-pluck)
[Arr::prepend](#method-array-prepend)
[Arr::prependKeysWith](#method-array-prependkeyswith)
[Arr::pull](#method-array-pull)
[Arr::push](#method-array-push)
[Arr::query](#method-array-query)
[Arr::random](#method-array-random)
[Arr::reject](#method-array-reject)
[Arr::select](#method-array-select)
[Arr::set](#method-array-set)
[Arr::shuffle](#method-array-shuffle)
[Arr::sole](#method-array-sole)
[Arr::some](#method-array-some)
[Arr::sort](#method-array-sort)
[Arr::sortDesc](#method-array-sort-desc)
[Arr::sortRecursive](#method-array-sort-recursive)
[Arr::string](#method-array-string)
[Arr::take](#method-array-take)
[Arr::toCssClasses](#method-array-to-css-classes)
[Arr::toCssStyles](#method-array-to-css-styles)
[Arr::undot](#method-array-undot)
[Arr::where](#method-array-where)
[Arr::whereNotNull](#method-array-where-not-null)
[Arr::wrap](#method-array-wrap)
[data_fill](#method-data-fill)
[data_get](#method-data-get)
[data_set](#method-data-set)
[data_forget](#method-data-forget)
[head](#method-head)
[last](#method-last)
</div>

<a name="numbers-method-list"></a>
### Liczby

<div class="collection-method-list" markdown="1">

[Number::abbreviate](#method-number-abbreviate)
[Number::clamp](#method-number-clamp)
[Number::currency](#method-number-currency)
[Number::defaultCurrency](#method-default-currency)
[Number::defaultLocale](#method-default-locale)
[Number::fileSize](#method-number-file-size)
[Number::forHumans](#method-number-for-humans)
[Number::format](#method-number-format)
[Number::ordinal](#method-number-ordinal)
[Number::pairs](#method-number-pairs)
[Number::parseInt](#method-number-parse-int)
[Number::parseFloat](#method-number-parse-float)
[Number::percentage](#method-number-percentage)
[Number::spell](#method-number-spell)
[Number::spellOrdinal](#method-number-spell-ordinal)
[Number::trim](#method-number-trim)
[Number::useLocale](#method-number-use-locale)
[Number::withLocale](#method-number-with-locale)
[Number::useCurrency](#method-number-use-currency)
[Number::withCurrency](#method-number-with-currency)

</div>

<a name="paths-method-list"></a>
### Ścieżki

<div class="collection-method-list" markdown="1">

[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[lang_path](#method-lang-path)
[public_path](#method-public-path)
[resource_path](#method-resource-path)
[storage_path](#method-storage-path)

</div>

<a name="urls-method-list"></a>
### URLs

<div class="collection-method-list" markdown="1">

[action](#method-action)
[asset](#method-asset)
[route](#method-route)
[secure_asset](#method-secure-asset)
[secure_url](#method-secure-url)
[to_action](#method-to-action)
[to_route](#method-to-route)
[uri](#method-uri)
[url](#method-url)

</div>

<a name="miscellaneous-method-list"></a>
### Różne

<div class="collection-method-list" markdown="1">

[abort](#method-abort)
[abort_if](#method-abort-if)
[abort_unless](#method-abort-unless)
[app](#method-app)
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[blank](#method-blank)
[broadcast](#method-broadcast)
[broadcast_if](#method-broadcast-if)
[broadcast_unless](#method-broadcast-unless)
[cache](#method-cache)
[class_uses_recursive](#method-class-uses-recursive)
[collect](#method-collect)
[config](#method-config)
[context](#method-context)
[cookie](#method-cookie)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[decrypt](#method-decrypt)
[dd](#method-dd)
[dispatch](#method-dispatch)
[dispatch_sync](#method-dispatch-sync)
[dump](#method-dump)
[encrypt](#method-encrypt)
[env](#method-env)
[event](#method-event)
[fake](#method-fake)
[filled](#method-filled)
[info](#method-info)
[literal](#method-literal)
[logger](#method-logger)
[method_field](#method-method-field)
[now](#method-now)
[old](#method-old)
[once](#method-once)
[optional](#method-optional)
[policy](#method-policy)
[redirect](#method-redirect)
[report](#method-report)
[report_if](#method-report-if)
[report_unless](#method-report-unless)
[request](#method-request)
[rescue](#method-rescue)
[resolve](#method-resolve)
[response](#method-response)
[retry](#method-retry)
[session](#method-session)
[tap](#method-tap)
[throw_if](#method-throw-if)
[throw_unless](#method-throw-unless)
[today](#method-today)
[trait_uses_recursive](#method-trait-uses-recursive)
[transform](#method-transform)
[validator](#method-validator)
[value](#method-value)
[view](#method-view)
[with](#method-with)
[when](#method-when)

</div>

<a name="arrays"></a>
## Tablice i obiekty

<a name="method-array-accessible"></a>
#### `Arr::accessible()` {.collection-method .first-collection-method}

Metoda `Arr::accessible` określa, czy dana wartość jest dostępna jako tablica:

```php
use Illuminate\Support\Arr;
use Illuminate\Support\Collection;

$isAccessible = Arr::accessible(['a' => 1, 'b' => 2]);

// true

$isAccessible = Arr::accessible(new Collection);

// true

$isAccessible = Arr::accessible('abc');

// false

$isAccessible = Arr::accessible(new stdClass);

// false
```

<a name="method-array-add"></a>
#### `Arr::add()` {.collection-method}

Metoda `Arr::add` dodaje daną parę klucz/wartość do tablicy, jeśli dany klucz nie istnieje już w tablicy lub jest ustawiony na `null`:

```php
use Illuminate\Support\Arr;

$array = Arr::add(['name' => 'Desk'], 'price', 100);

// ['name' => 'Desk', 'price' => 100]

$array = Arr::add(['name' => 'Desk', 'price' => null], 'price', 100);

// ['name' => 'Desk', 'price' => 100]
```

<a name="method-array-array"></a>
#### `Arr::array()` {.collection-method}

Metoda `Arr::array` pobiera wartość z głęboko zagnieżdżonej tablicy używając notacji "kropkowej" (tak samo jak [Arr::get()](#method-array-get)), ale rzuca wyjątek `InvalidArgumentException`, jeśli żądana wartość nie jest typu `array`:

```
use Illuminate\Support\Arr;

$array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

$value = Arr::array($array, 'languages');

// ['PHP', 'Ruby']

$value = Arr::array($array, 'name');

// rzuca InvalidArgumentException
```

<a name="method-array-boolean"></a>
#### `Arr::boolean()` {.collection-method}

Metoda `Arr::boolean` pobiera wartość z głęboko zagnieżdżonej tablicy używając notacji "kropkowej" (tak samo jak [Arr::get()](#method-array-get)), ale rzuca wyjątek `InvalidArgumentException`, jeśli żądana wartość nie jest typu `boolean`:

```
use Illuminate\Support\Arr;

$array = ['name' => 'Joe', 'available' => true];

$value = Arr::boolean($array, 'available');

// true

$value = Arr::boolean($array, 'name');

// rzuca InvalidArgumentException
```


<a name="method-array-collapse"></a>
#### `Arr::collapse()` {.collection-method}

Metoda `Arr::collapse` zwija tablicę tablic lub kolekcji w jedną tablicę:

```php
use Illuminate\Support\Arr;

$array = Arr::collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

<a name="method-array-crossjoin"></a>
#### `Arr::crossJoin()` {.collection-method}

Metoda `Arr::crossJoin` wykonuje iloczyn kartezjański podanych tablic, zwracając wszystkie możliwe permutacje:

```php
use Illuminate\Support\Arr;

$matrix = Arr::crossJoin([1, 2], ['a', 'b']);

/*
    [
        [1, 'a'],
        [1, 'b'],
        [2, 'a'],
        [2, 'b'],
    ]
*/

$matrix = Arr::crossJoin([1, 2], ['a', 'b'], ['I', 'II']);

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

<a name="method-array-divide"></a>
#### `Arr::divide()` {.collection-method}

Metoda `Arr::divide` zwraca dwie tablice: jedną zawierającą klucze, a drugą zawierającą wartości danej tablicy:

```php
use Illuminate\Support\Arr;

[$keys, $values] = Arr::divide(['name' => 'Desk']);

// $keys: ['name']

// $values: ['Desk']
```

<a name="method-array-dot"></a>
#### `Arr::dot()` {.collection-method}

Metoda `Arr::dot` spłaszcza wielowymiarową tablicę do tablicy jednowymiarowej, która używa notacji "kropkowej" do wskazania głębokości:

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

$flattened = Arr::dot($array);

// ['products.desk.price' => 100]
```

<a name="method-array-every"></a>
#### `Arr::every()` {.collection-method}

Metoda `Arr::every` sprawdza, czy wszystkie wartości w tablicy przechodzą dany test prawdziwości:

```php
use Illuminate\Support\Arr;

$array = [1, 2, 3];

Arr::every($array, fn ($i) => $i > 0);

// true

Arr::every($array, fn ($i) => $i > 2);

// false
```

<a name="method-array-except"></a>
#### `Arr::except()` {.collection-method}

Metoda `Arr::except` usuwa podane pary klucz/wartość z tablicy:

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Desk', 'price' => 100];

$filtered = Arr::except($array, ['price']);

// ['name' => 'Desk']
```

<a name="method-array-except-values"></a>
#### `Arr::exceptValues()` {.collection-method}

Metoda `Arr::exceptValues` usuwa określone wartości z tablicy:

```php
use Illuminate\Support\Arr;

$array = ['foo', 'bar', 'baz', 'qux'];

$filtered = Arr::exceptValues($array, ['foo', 'baz']);

// ['bar', 'qux']
```

Możesz również przekazać `true` do argumentu `strict`, aby używać ścisłego porównania typów podczas filtrowania:

```php
use Illuminate\Support\Arr;

$array = [1, '1', 2, '2'];

$filtered = Arr::exceptValues($array, [1, 2], strict: true);

// ['1', '2']
```

<a name="method-array-exists"></a>
#### `Arr::exists()` {.collection-method}

Metoda `Arr::exists` sprawdza, czy dany klucz istnieje w podanej tablicy:

```php
use Illuminate\Support\Arr;

$array = ['name' => 'John Doe', 'age' => 17];

$exists = Arr::exists($array, 'name');

// true

$exists = Arr::exists($array, 'salary');

// false
```

<a name="method-array-first"></a>
#### `Arr::first()` {.collection-method}

Metoda `Arr::first` zwraca pierwszy element tablicy przechodzący dany test prawdziwości:

```php
use Illuminate\Support\Arr;

$array = [100, 200, 300];

$first = Arr::first($array, function (int $value, int $key) {
    return $value >= 150;
});

// 200
```

Wartość domyślną można również przekazać jako trzeci parametr do metody. Ta wartość zostanie zwrócona, jeśli żadna wartość nie przejdzie testu prawdziwości:

```php
use Illuminate\Support\Arr;

$first = Arr::first($array, $callback, $default);
```

<a name="method-array-flatten"></a>
#### `Arr::flatten()` {.collection-method}

Metoda `Arr::flatten` spłaszcza wielowymiarową tablicę do tablicy jednowymiarowej:

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

$flattened = Arr::flatten($array);

// ['Joe', 'PHP', 'Ruby']
```

<a name="method-array-float"></a>
#### `Arr::float()` {.collection-method}

Metoda `Arr::float` pobiera wartość z głęboko zagnieżdżonej tablicy używając notacji "kropkowej" (tak samo jak [Arr::get()](#method-array-get)), ale rzuca wyjątek `InvalidArgumentException`, jeśli żądana wartość nie jest typu `float`:

```
use Illuminate\Support\Arr;

$array = ['name' => 'Joe', 'balance' => 123.45];

$value = Arr::float($array, 'balance');

// 123.45

$value = Arr::float($array, 'name');

// rzuca InvalidArgumentException
```

<a name="method-array-forget"></a>
#### `Arr::forget()` {.collection-method}

Metoda `Arr::forget` usuwa daną parę klucz/wartość z głęboko zagnieżdżonej tablicy używając notacji "kropkowej":

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

Arr::forget($array, 'products.desk');

// ['products' => []]
```

<a name="method-array-from"></a>
#### `Arr::from()` {.collection-method}

Metoda `Arr::from` konwertuje różne typy wejściowe na zwykłą tablicę PHP. Obsługuje szereg typów wejściowych, w tym tablice, obiekty i kilka popularnych interfejsów Laravel, takich jak `Arrayable`, `Enumerable`, `Jsonable` i `JsonSerializable`. Dodatkowo obsługuje instancje `Traversable` i `WeakMap`:

```php
use Illuminate\Support\Arr;

Arr::from((object) ['foo' => 'bar']); // ['foo' => 'bar']

class TestJsonableObject implements Jsonable
{
    public function toJson($options = 0)
    {
        return json_encode(['foo' => 'bar']);
    }
}

Arr::from(new TestJsonableObject); // ['foo' => 'bar']
```

<a name="method-array-get"></a>
#### `Arr::get()` {.collection-method}

Metoda `Arr::get` pobiera wartość z głęboko zagnieżdżonej tablicy używając notacji "kropkowej":

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

$price = Arr::get($array, 'products.desk.price');

// 100
```

Metoda `Arr::get` akceptuje również wartość domyślną, która zostanie zwrócona, jeśli określony klucz nie jest obecny w tablicy:

```php
use Illuminate\Support\Arr;

$discount = Arr::get($array, 'products.desk.discount', 0);

// 0
```

<a name="method-array-has"></a>
#### `Arr::has()` {.collection-method}

Metoda `Arr::has` sprawdza, czy dany element lub elementy istnieją w tablicy używając notacji "kropkowej":

```php
use Illuminate\Support\Arr;

$array = ['product' => ['name' => 'Desk', 'price' => 100]];

$contains = Arr::has($array, 'product.name');

// true

$contains = Arr::has($array, ['product.price', 'product.discount']);

// false
```

<a name="method-array-hasall"></a>
#### `Arr::hasAll()` {.collection-method}

Metoda `Arr::hasAll` określa, czy wszystkie określone klucze istnieją w danej tablicy używając notacji "kropkowej":

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Taylor', 'language' => 'PHP'];

Arr::hasAll($array, ['name']); // true
Arr::hasAll($array, ['name', 'language']); // true
Arr::hasAll($array, ['name', 'IDE']); // false
```

<a name="method-array-hasany"></a>
#### `Arr::hasAny()` {.collection-method}

Metoda `Arr::hasAny` sprawdza, czy którykolwiek element z danego zestawu istnieje w tablicy używając notacji "kropkowej":

```php
use Illuminate\Support\Arr;

$array = ['product' => ['name' => 'Desk', 'price' => 100]];

$contains = Arr::hasAny($array, 'product.name');

// true

$contains = Arr::hasAny($array, ['product.name', 'product.discount']);

// true

$contains = Arr::hasAny($array, ['category', 'product.discount']);

// false
```

<a name="method-array-integer"></a>
#### `Arr::integer()` {.collection-method}

Metoda `Arr::integer` pobiera wartość z głęboko zagnieżdżonej tablicy używając notacji "kropkowej" (tak samo jak [Arr::get()](#method-array-get)), ale rzuca wyjątek `InvalidArgumentException`, jeśli żądana wartość nie jest typu `int`:

```
use Illuminate\Support\Arr;

$array = ['name' => 'Joe', 'age' => 42];

$value = Arr::integer($array, 'age');

// 42

$value = Arr::integer($array, 'name');

// rzuca InvalidArgumentException
```

<a name="method-array-isassoc"></a>
#### `Arr::isAssoc()` {.collection-method}

Metoda `Arr::isAssoc` zwraca `true`, jeśli dana tablica jest tablicą asocjacyjną. Tablica jest uważana za "asocjacyjną", jeśli nie ma sekwencyjnych kluczy numerycznych zaczynających się od zera:

```php
use Illuminate\Support\Arr;

$isAssoc = Arr::isAssoc(['product' => ['name' => 'Desk', 'price' => 100]]);

// true

$isAssoc = Arr::isAssoc([1, 2, 3]);

// false
```

<a name="method-array-islist"></a>
#### `Arr::isList()` {.collection-method}

Metoda `Arr::isList` zwraca `true`, jeśli klucze danej tablicy są sekwencyjnymi liczbami całkowitymi zaczynającymi się od zera:

```php
use Illuminate\Support\Arr;

$isList = Arr::isList(['foo', 'bar', 'baz']);

// true

$isList = Arr::isList(['product' => ['name' => 'Desk', 'price' => 100]]);

// false
```

<a name="method-array-join"></a>
#### `Arr::join()` {.collection-method}

Metoda `Arr::join` łączy elementy tablicy ze sobą za pomocą łańcucha znaków. Używając trzeciego argumentu tej metody, możesz również określić łańcuch łączący dla ostatniego elementu tablicy:

```php
use Illuminate\Support\Arr;

$array = ['Tailwind', 'Alpine', 'Laravel', 'Livewire'];

$joined = Arr::join($array, ', ');

// Tailwind, Alpine, Laravel, Livewire

$joined = Arr::join($array, ', ', ', and ');

// Tailwind, Alpine, Laravel, and Livewire
```

<a name="method-array-keyby"></a>
#### `Arr::keyBy()` {.collection-method}

Metoda `Arr::keyBy` indeksuje tablicę według podanego klucza. Jeśli wiele elementów ma ten sam klucz, w nowej tablicy pojawi się tylko ostatni z nich:

```php
use Illuminate\Support\Arr;

$array = [
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
];

$keyed = Arr::keyBy($array, 'product_id');

/*
    [
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]
*/
```

<a name="method-array-last"></a>
#### `Arr::last()` {.collection-method}

Metoda `Arr::last` zwraca ostatni element tablicy przechodzący dany test prawdziwości:

```php
use Illuminate\Support\Arr;

$array = [100, 200, 300, 110];

$last = Arr::last($array, function (int $value, int $key) {
    return $value >= 150;
});

// 300
```

Wartość domyślną można przekazać jako trzeci argument do metody. Ta wartość zostanie zwrócona, jeśli żadna wartość nie przejdzie testu prawdziwości:

```php
use Illuminate\Support\Arr;

$last = Arr::last($array, $callback, $default);
```

<a name="method-array-map"></a>
#### `Arr::map()` {.collection-method}

Metoda `Arr::map` iteruje przez tablicę i przekazuje każdą wartość oraz klucz do podanego callbacka. Wartość w tablicy jest zastępowana wartością zwróconą przez callback:

```php
use Illuminate\Support\Arr;

$array = ['first' => 'james', 'last' => 'kirk'];

$mapped = Arr::map($array, function (string $value, string $key) {
    return ucfirst($value);
});

// ['first' => 'James', 'last' => 'Kirk']
```

<a name="method-array-map-spread"></a>
#### `Arr::mapSpread()` {.collection-method}

Metoda `Arr::mapSpread` iteruje przez tablicę, przekazując każdą zagnieżdżoną wartość elementu do podanego domknięcia. Domknięcie może swobodnie modyfikować element i go zwrócić, tworząc w ten sposób nową tablicę zmodyfikowanych elementów:

```php
use Illuminate\Support\Arr;

$array = [
    [0, 1],
    [2, 3],
    [4, 5],
    [6, 7],
    [8, 9],
];

$mapped = Arr::mapSpread($array, function (int $even, int $odd) {
    return $even + $odd;
});

/*
    [1, 5, 9, 13, 17]
*/
```

<a name="method-array-map-with-keys"></a>
#### `Arr::mapWithKeys()` {.collection-method}

Metoda `Arr::mapWithKeys` iteruje przez tablicę i przekazuje każdą wartość do podanego wywołania zwrotnego. Wywołanie zwrotne powinno zwrócić tablicę asocjacyjną zawierającą pojedynczą parę klucz / wartość:

```php
use Illuminate\Support\Arr;

$array = [
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
];

$mapped = Arr::mapWithKeys($array, function (array $item, int $key) {
    return [$item['email'] => $item['name']];
});

/*
    [
        'john@example.com' => 'John',
        'jane@example.com' => 'Jane',
    ]
*/
```

<a name="method-array-only"></a>
#### `Arr::only()` {.collection-method}

Metoda `Arr::only` zwraca tylko określone pary klucz / wartość z podanej tablicy:

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

$slice = Arr::only($array, ['name', 'price']);

// ['name' => 'Desk', 'price' => 100]
```

<a name="method-array-only-values"></a>
#### `Arr::onlyValues()` {.collection-method}

Metoda `Arr::onlyValues` zwraca tylko określone wartości z tablicy:

```php
use Illuminate\Support\Arr;

$array = ['foo', 'bar', 'baz', 'qux'];

$filtered = Arr::onlyValues($array, ['foo', 'baz']);

// ['foo', 'baz']
```

Możesz również przekazać `true` do argumentu `strict`, aby użyć ścisłego porównania typów podczas filtrowania:

```php
use Illuminate\Support\Arr;

$array = [1, '1', 2, '2'];

$filtered = Arr::onlyValues($array, [1, 2], strict: true);

// [1, 2]
```

<a name="method-array-partition"></a>
#### `Arr::partition()` {.collection-method}

Metoda `Arr::partition` może być połączona z destrukturyzacją tablicy PHP, aby oddzielić elementy, które przechodzą dany test prawdy, od tych, które nie przechodzą:

```php
<?php

use Illuminate\Support\Arr;

$numbers = [1, 2, 3, 4, 5, 6];

[$underThree, $equalOrAboveThree] = Arr::partition($numbers, function (int $i) {
    return $i < 3;
});

dump($underThree);

// [1, 2]

dump($equalOrAboveThree);

// [3, 4, 5, 6]
```

<a name="method-array-pluck"></a>
#### `Arr::pluck()` {.collection-method}

Metoda `Arr::pluck` pobiera wszystkie wartości dla podanego klucza z tablicy:

```php
use Illuminate\Support\Arr;

$array = [
    ['developer' => ['id' => 1, 'name' => 'Taylor']],
    ['developer' => ['id' => 2, 'name' => 'Abigail']],
];

$names = Arr::pluck($array, 'developer.name');

// ['Taylor', 'Abigail']
```

Możesz również określić, jak chcesz, aby wynikowa lista była indeksowana:

```php
use Illuminate\Support\Arr;

$names = Arr::pluck($array, 'developer.name', 'developer.id');

// [1 => 'Taylor', 2 => 'Abigail']
```

<a name="method-array-prepend"></a>
#### `Arr::prepend()` {.collection-method}

Metoda `Arr::prepend` doda element na początek tablicy:

```php
use Illuminate\Support\Arr;

$array = ['one', 'two', 'three', 'four'];

$array = Arr::prepend($array, 'zero');

// ['zero', 'one', 'two', 'three', 'four']
```

Jeśli to konieczne, możesz określić klucz, który powinien być użyty dla wartości:

```php
use Illuminate\Support\Arr;

$array = ['price' => 100];

$array = Arr::prepend($array, 'Desk', 'name');

// ['name' => 'Desk', 'price' => 100]
```

<a name="method-array-prependkeyswith"></a>
#### `Arr::prependKeysWith()` {.collection-method}

Metoda `Arr::prependKeysWith` dodaje do wszystkich nazw kluczy tablicy asocjacyjnej podany prefiks:

```php
use Illuminate\Support\Arr;

$array = [
    'name' => 'Desk',
    'price' => 100,
];

$keyed = Arr::prependKeysWith($array, 'product.');

/*
    [
        'product.name' => 'Desk',
        'product.price' => 100,
    ]
*/
```

<a name="method-array-pull"></a>
#### `Arr::pull()` {.collection-method}

Metoda `Arr::pull` zwraca i usuwa parę klucz / wartość z tablicy:

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Desk', 'price' => 100];

$name = Arr::pull($array, 'name');

// $name: Desk

// $array: ['price' => 100]
```

Wartość domyślna może być przekazana jako trzeci argument metody. Ta wartość zostanie zwrócona, jeśli klucz nie istnieje:

```php
use Illuminate\Support\Arr;

$value = Arr::pull($array, $key, $default);
```

<a name="method-array-push"></a>
#### `Arr::push()` {.collection-method}

Metoda `Arr::push` dodaje element do tablicy używając notacji "kropkowej". Jeśli tablica nie istnieje pod podanym kluczem, zostanie utworzona:

```php
use Illuminate\Support\Arr;

$array = [];

Arr::push($array, 'office.furniture', 'Desk');

// $array: ['office' => ['furniture' => ['Desk']]]
```

<a name="method-array-query"></a>
#### `Arr::query()` {.collection-method}

Metoda `Arr::query` konwertuje tablicę na ciąg zapytania:

```php
use Illuminate\Support\Arr;

$array = [
    'name' => 'Taylor',
    'order' => [
        'column' => 'created_at',
        'direction' => 'desc'
    ]
];

Arr::query($array);

// name=Taylor&order[column]=created_at&order[direction]=desc
```

<a name="method-array-random"></a>
#### `Arr::random()` {.collection-method}

Metoda `Arr::random` zwraca losową wartość z tablicy:

```php
use Illuminate\Support\Arr;

$array = [1, 2, 3, 4, 5];

$random = Arr::random($array);

// 4 - (retrieved randomly)
```

Możesz również określić liczbę elementów do zwrócenia jako opcjonalny drugi argument. Zauważ, że podanie tego argumentu zwróci tablicę, nawet jeśli pożądany jest tylko jeden element:

```php
use Illuminate\Support\Arr;

$items = Arr::random($array, 2);

// [2, 5] - (retrieved randomly)
```

<a name="method-array-reject"></a>
#### `Arr::reject()` {.collection-method}

Metoda `Arr::reject` usuwa elementy z tablicy za pomocą podanego domknięcia:

```php
use Illuminate\Support\Arr;

$array = [100, '200', 300, '400', 500];

$filtered = Arr::reject($array, function (string|int $value, int $key) {
    return is_string($value);
});

// [0 => 100, 2 => 300, 4 => 500]
```

<a name="method-array-select"></a>
#### `Arr::select()` {.collection-method}

Metoda `Arr::select` wybiera tablicę wartości z tablicy:

```php
use Illuminate\Support\Arr;

$array = [
    ['id' => 1, 'name' => 'Desk', 'price' => 200],
    ['id' => 2, 'name' => 'Table', 'price' => 150],
    ['id' => 3, 'name' => 'Chair', 'price' => 300],
];

Arr::select($array, ['name', 'price']);

// [['name' => 'Desk', 'price' => 200], ['name' => 'Table', 'price' => 150], ['name' => 'Chair', 'price' => 300]]
```

<a name="method-array-set"></a>
#### `Arr::set()` {.collection-method}

Metoda `Arr::set` ustawia wartość wewnątrz głęboko zagnieżdżonej tablicy używając notacji "kropkowej":

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

Arr::set($array, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 200]]]
```

<a name="method-array-shuffle"></a>
#### `Arr::shuffle()` {.collection-method}

Metoda `Arr::shuffle` losowo tasuje elementy w tablicy:

```php
use Illuminate\Support\Arr;

$array = Arr::shuffle([1, 2, 3, 4, 5]);

// [3, 2, 5, 1, 4] - (generated randomly)
```

<a name="method-array-sole"></a>
#### `Arr::sole()` {.collection-method}

Metoda `Arr::sole` pobiera pojedynczą wartość z tablicy za pomocą podanego domknięcia. Jeśli więcej niż jedna wartość w tablicy pasuje do podanego testu prawdy, zostanie rzucony wyjątek `Illuminate\Support\MultipleItemsFoundException`. Jeśli żadna wartość nie pasuje do testu prawdy, zostanie rzucony wyjątek `Illuminate\Support\ItemNotFoundException`:

```php
use Illuminate\Support\Arr;

$array = ['Desk', 'Table', 'Chair'];

$value = Arr::sole($array, fn (string $value) => $value === 'Desk');

// 'Desk'
```

<a name="method-array-some"></a>
#### `Arr::some()` {.collection-method}

Metoda `Arr::some` zapewnia, że przynajmniej jedna z wartości w tablicy przechodzi podany test prawdy:

```php
use Illuminate\Support\Arr;

$array = [1, 2, 3];

Arr::some($array, fn ($i) => $i > 2);

// true
```

<a name="method-array-sort"></a>
#### `Arr::sort()` {.collection-method}

Metoda `Arr::sort` sortuje tablicę według jej wartości:

```php
use Illuminate\Support\Arr;

$array = ['Desk', 'Table', 'Chair'];

$sorted = Arr::sort($array);

// ['Chair', 'Desk', 'Table']
```

Możesz również posortować tablicę według wyników podanego domknięcia:

```php
use Illuminate\Support\Arr;

$array = [
    ['name' => 'Desk'],
    ['name' => 'Table'],
    ['name' => 'Chair'],
];

$sorted = array_values(Arr::sort($array, function (array $value) {
    return $value['name'];
}));

/*
    [
        ['name' => 'Chair'],
        ['name' => 'Desk'],
        ['name' => 'Table'],
    ]
*/
```

<a name="method-array-sort-desc"></a>
#### `Arr::sortDesc()` {.collection-method}

Metoda `Arr::sortDesc` sortuje tablicę w kolejności malejącej według jej wartości:

```php
use Illuminate\Support\Arr;

$array = ['Desk', 'Table', 'Chair'];

$sorted = Arr::sortDesc($array);

// ['Table', 'Desk', 'Chair']
```

Możesz również posortować tablicę według wyników podanego domknięcia:

```php
use Illuminate\Support\Arr;

$array = [
    ['name' => 'Desk'],
    ['name' => 'Table'],
    ['name' => 'Chair'],
];

$sorted = array_values(Arr::sortDesc($array, function (array $value) {
    return $value['name'];
}));

/*
    [
        ['name' => 'Table'],
        ['name' => 'Desk'],
        ['name' => 'Chair'],
    ]
*/
```

<a name="method-array-sort-recursive"></a>
#### `Arr::sortRecursive()` {.collection-method}

Metoda `Arr::sortRecursive` rekursywnie sortuje tablicę używając funkcji `sort` dla podtablic indeksowanych numerycznie i funkcji `ksort` dla podtablic asocjacyjnych:

```php
use Illuminate\Support\Arr;

$array = [
    ['Roman', 'Taylor', 'Li'],
    ['PHP', 'Ruby', 'JavaScript'],
    ['one' => 1, 'two' => 2, 'three' => 3],
];

$sorted = Arr::sortRecursive($array);

/*
    [
        ['JavaScript', 'PHP', 'Ruby'],
        ['one' => 1, 'three' => 3, 'two' => 2],
        ['Li', 'Roman', 'Taylor'],
    ]
*/
```

Jeśli chcesz, aby wyniki były posortowane w kolejności malejącej, możesz użyć metody `Arr::sortRecursiveDesc`.

```php
$sorted = Arr::sortRecursiveDesc($array);
```

<a name="method-array-string"></a>
#### `Arr::string()` {.collection-method}

Metoda `Arr::string` pobiera wartość z głęboko zagnieżdżonej tablicy używając notacji "kropkowej" (tak samo jak [Arr::get()](#method-array-get)), ale rzuca wyjątek `InvalidArgumentException`, jeśli żądana wartość nie jest `string`:

```
use Illuminate\Support\Arr;

$array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

$value = Arr::string($array, 'name');

// Joe

$value = Arr::string($array, 'languages');

// throws InvalidArgumentException
```

<a name="method-array-take"></a>
#### `Arr::take()` {.collection-method}

Metoda `Arr::take` zwraca nową tablicę z określoną liczbą elementów:

```php
use Illuminate\Support\Arr;

$array = [0, 1, 2, 3, 4, 5];

$chunk = Arr::take($array, 3);

// [0, 1, 2]
```

Możesz również przekazać liczbę ujemną, aby pobrać określoną liczbę elementów z końca tablicy:

```php
$array = [0, 1, 2, 3, 4, 5];

$chunk = Arr::take($array, -2);

// [4, 5]
```

<a name="method-array-to-css-classes"></a>
#### `Arr::toCssClasses()` {.collection-method}

Metoda `Arr::toCssClasses` warunkowo kompiluje łańcuch klasy CSS. Metoda przyjmuje tablicę klas, w której klucz tablicy zawiera klasę lub klasy, które chcesz dodać, a wartość jest wyrażeniem boolean. Jeśli element tablicy ma klucz numeryczny, zawsze będzie uwzględniony na renderowanej liście klas:

```php
use Illuminate\Support\Arr;

$isActive = false;
$hasError = true;

$array = ['p-4', 'font-bold' => $isActive, 'bg-red' => $hasError];

$classes = Arr::toCssClasses($array);

/*
    'p-4 bg-red'
*/
```

<a name="method-array-to-css-styles"></a>
#### `Arr::toCssStyles()` {.collection-method}

Metoda `Arr::toCssStyles` warunkowo kompiluje łańcuch stylów CSS. Metoda przyjmuje tablicę deklaracji CSS, w której klucz tablicy zawiera deklarację CSS, którą chcesz dodać, a wartość jest wyrażeniem boolean. Jeśli element tablicy ma klucz numeryczny, zawsze będzie uwzględniony w skompilowanym łańcuchu stylów CSS:

```php
use Illuminate\Support\Arr;

$hasColor = true;

$array = ['background-color: blue', 'color: blue' => $hasColor];

$classes = Arr::toCssStyles($array);

/*
    'background-color: blue; color: blue;'
*/
```

Ta metoda zasila funkcjonalność Laravela umożliwiającą [łączenie klas z workiem atrybutów komponentu Blade](/docs/{{version}}/blade#conditionally-merge-classes), a także dyrektywę `@class` [Blade](/docs/{{version}}/blade#conditional-classes).

<a name="method-array-undot"></a>
#### `Arr::undot()` {.collection-method}

Metoda `Arr::undot` rozwija jednowymiarową tablicę, która używa notacji "kropkowej", w tablicę wielowymiarową:

```php
use Illuminate\Support\Arr;

$array = [
    'user.name' => 'Kevin Malone',
    'user.occupation' => 'Accountant',
];

$array = Arr::undot($array);

// ['user' => ['name' => 'Kevin Malone', 'occupation' => 'Accountant']]
```

<a name="method-array-where"></a>
#### `Arr::where()` {.collection-method}

Metoda `Arr::where` filtruje tablicę za pomocą podanego domknięcia:

```php
use Illuminate\Support\Arr;

$array = [100, '200', 300, '400', 500];

$filtered = Arr::where($array, function (string|int $value, int $key) {
    return is_string($value);
});

// [1 => '200', 3 => '400']
```

<a name="method-array-where-not-null"></a>
#### `Arr::whereNotNull()` {.collection-method}

Metoda `Arr::whereNotNull` usuwa wszystkie wartości `null` z podanej tablicy:

```php
use Illuminate\Support\Arr;

$array = [0, null];

$filtered = Arr::whereNotNull($array);

// [0 => 0]
```

<a name="method-array-wrap"></a>
#### `Arr::wrap()` {.collection-method}

Metoda `Arr::wrap` opakowuje podaną wartość w tablicę. Jeśli podana wartość jest już tablicą, zostanie zwrócona bez modyfikacji:

```php
use Illuminate\Support\Arr;

$string = 'Laravel';

$array = Arr::wrap($string);

// ['Laravel']
```

Jeśli podana wartość to `null`, zostanie zwrócona pusta tablica:

```php
use Illuminate\Support\Arr;

$array = Arr::wrap(null);

// []
```

<a name="method-data-fill"></a>
#### `data_fill()` {.collection-method}

Funkcja `data_fill` ustawia brakującą wartość wewnątrz zagnieżdżonej tablicy lub obiektu używając notacji "kropkowej":

```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_fill($data, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 100]]]

data_fill($data, 'products.desk.discount', 10);

// ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]
```

Ta funkcja również akceptuje gwiazdki jako symbole wieloznaczne i odpowiednio wypełni cel:

```php
$data = [
    'products' => [
        ['name' => 'Desk 1', 'price' => 100],
        ['name' => 'Desk 2'],
    ],
];

data_fill($data, 'products.*.price', 200);

/*
    [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 200],
        ],
    ]
*/
```

<a name="method-data-get"></a>
#### `data_get()` {.collection-method}

Funkcja `data_get` pobiera wartość z zagnieżdżonej tablicy lub obiektu używając notacji "kropkowej":

```php
$data = ['products' => ['desk' => ['price' => 100]]];

$price = data_get($data, 'products.desk.price');

// 100
```

Funkcja `data_get` akceptuje również wartość domyślną, która zostanie zwrócona, jeśli określony klucz nie zostanie znaleziony:

```php
$discount = data_get($data, 'products.desk.discount', 0);

// 0
```

Funkcja akceptuje również symbole wieloznaczne używające gwiazdek, które mogą targetować dowolny klucz tablicy lub obiektu:

```php
$data = [
    'product-one' => ['name' => 'Desk 1', 'price' => 100],
    'product-two' => ['name' => 'Desk 2', 'price' => 150],
];

data_get($data, '*.name');

// ['Desk 1', 'Desk 2'];
```

Symbole zastępcze `{first}` i `{last}` mogą być używane do pobierania pierwszych lub ostatnich elementów w tablicy:

```php
$flight = [
    'segments' => [
        ['from' => 'LHR', 'departure' => '9:00', 'to' => 'IST', 'arrival' => '15:00'],
        ['from' => 'IST', 'departure' => '16:00', 'to' => 'PKX', 'arrival' => '20:00'],
    ],
];

data_get($flight, 'segments.{first}.arrival');

// 15:00
```

<a name="method-data-set"></a>
#### `data_set()` {.collection-method}

Funkcja `data_set` ustawia wartość wewnątrz zagnieżdżonej tablicy lub obiektu używając notacji "kropkowej":

```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_set($data, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 200]]]
```

Ta funkcja również akceptuje symbole wieloznaczne używające gwiazdek i odpowiednio ustawi wartości w celu:

```php
$data = [
    'products' => [
        ['name' => 'Desk 1', 'price' => 100],
        ['name' => 'Desk 2', 'price' => 150],
    ],
];

data_set($data, 'products.*.price', 200);

/*
    [
        'products' => [
            ['name' => 'Desk 1', 'price' => 200],
            ['name' => 'Desk 2', 'price' => 200],
        ],
    ]
*/
```

Domyślnie wszystkie istniejące wartości są nadpisywane. Jeśli chcesz ustawić wartość tylko wtedy, gdy nie istnieje, możesz przekazać `false` jako czwarty argument funkcji:

```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_set($data, 'products.desk.price', 200, overwrite: false);

// ['products' => ['desk' => ['price' => 100]]]
```

<a name="method-data-forget"></a>
#### `data_forget()` {.collection-method}

Funkcja `data_forget` usuwa wartość wewnątrz zagnieżdżonej tablicy lub obiektu używając notacji "kropkowej":

```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_forget($data, 'products.desk.price');

// ['products' => ['desk' => []]]
```

Ta funkcja również akceptuje symbole wieloznaczne używające gwiazdek i odpowiednio usunie wartości z celu:

```php
$data = [
    'products' => [
        ['name' => 'Desk 1', 'price' => 100],
        ['name' => 'Desk 2', 'price' => 150],
    ],
];

data_forget($data, 'products.*.price');

/*
    [
        'products' => [
            ['name' => 'Desk 1'],
            ['name' => 'Desk 2'],
        ],
    ]
*/
```

<a name="method-head"></a>
#### `head()` {.collection-method}

Funkcja `head` zwraca pierwszy element z podanej tablicy. Jeśli tablica jest pusta, zwrócone zostanie `false`:

```php
$array = [100, 200, 300];

$first = head($array);

// 100
```

<a name="method-last"></a>
#### `last()` {.collection-method}

Funkcja `last` zwraca ostatni element z podanej tablicy. Jeśli tablica jest pusta, zwrócone zostanie `false`:

```php
$array = [100, 200, 300];

$last = last($array);

// 300
```

<a name="numbers"></a>
## Liczby

<a name="method-number-abbreviate"></a>
#### `Number::abbreviate()` {.collection-method}

Metoda `Number::abbreviate` zwraca czytelny dla człowieka format podanej wartości numerycznej, ze skrótem jednostek:

```php
use Illuminate\Support\Number;

$number = Number::abbreviate(1000);

// 1K

$number = Number::abbreviate(489939);

// 490K

$number = Number::abbreviate(1230000, precision: 2);

// 1.23M
```

<a name="method-number-clamp"></a>
#### `Number::clamp()` {.collection-method}

Metoda `Number::clamp` zapewnia, że podana liczba pozostaje w określonym zakresie. Jeśli liczba jest niższa niż minimum, zwracana jest wartość minimalna. Jeśli liczba jest wyższa niż maksimum, zwracana jest wartość maksymalna:

```php
use Illuminate\Support\Number;

$number = Number::clamp(105, min: 10, max: 100);

// 100

$number = Number::clamp(5, min: 10, max: 100);

// 10

$number = Number::clamp(10, min: 10, max: 100);

// 10

$number = Number::clamp(20, min: 10, max: 100);

// 20
```

<a name="method-number-currency"></a>
#### `Number::currency()` {.collection-method}

Metoda `Number::currency` zwraca reprezentację walutową podanej wartości jako string:

```php
use Illuminate\Support\Number;

$currency = Number::currency(1000);

// $1,000.00

$currency = Number::currency(1000, in: 'EUR');

// €1,000.00

$currency = Number::currency(1000, in: 'EUR', locale: 'de');

// 1.000,00 €

$currency = Number::currency(1000, in: 'EUR', locale: 'de', precision: 0);

// 1.000 €
```

<a name="method-default-currency"></a>
#### `Number::defaultCurrency()` {.collection-method}

Metoda `Number::defaultCurrency` zwraca domyślną walutę używaną przez klasę `Number`:

```php
use Illuminate\Support\Number;

$currency = Number::defaultCurrency();

// USD
```

<a name="method-default-locale"></a>
#### `Number::defaultLocale()` {.collection-method}

Metoda `Number::defaultLocale` zwraca domyślne ustawienia regionalne używane przez klasę `Number`:

```php
use Illuminate\Support\Number;

$locale = Number::defaultLocale();

// en
```

<a name="method-number-file-size"></a>
#### `Number::fileSize()` {.collection-method}

Metoda `Number::fileSize` zwraca reprezentację rozmiaru pliku dla podanej wartości w bajtach jako string:

```php
use Illuminate\Support\Number;

$size = Number::fileSize(1024);

// 1 KB

$size = Number::fileSize(1024 * 1024);

// 1 MB

$size = Number::fileSize(1024, precision: 2);

// 1.00 KB
```

<a name="method-number-for-humans"></a>
#### `Number::forHumans()` {.collection-method}

Metoda `Number::forHumans` zwraca czytelny dla człowieka format podanej wartości numerycznej:

```php
use Illuminate\Support\Number;

$number = Number::forHumans(1000);

// 1 thousand

$number = Number::forHumans(489939);

// 490 thousand

$number = Number::forHumans(1230000, precision: 2);

// 1.23 million
```

<a name="method-number-format"></a>
#### `Number::format()` {.collection-method}

Metoda `Number::format` formatuje podaną liczbę na string zgodny z ustawieniami regionalnymi:

```php
use Illuminate\Support\Number;

$number = Number::format(100000);

// 100,000

$number = Number::format(100000, precision: 2);

// 100,000.00

$number = Number::format(100000.123, maxPrecision: 2);

// 100,000.12

$number = Number::format(100000, locale: 'de');

// 100.000
```

<a name="method-number-ordinal"></a>
#### `Number::ordinal()` {.collection-method}

Metoda `Number::ordinal` zwraca reprezentację porządkową liczby:

```php
use Illuminate\Support\Number;

$number = Number::ordinal(1);

// 1st

$number = Number::ordinal(2);

// 2nd

$number = Number::ordinal(21);

// 21st
```

<a name="method-number-pairs"></a>
#### `Number::pairs()` {.collection-method}

Metoda `Number::pairs` generuje tablicę par liczb (podzakresów) na podstawie określonego zakresu i wartości kroku. Ta metoda może być przydatna do dzielenia większego zakresu liczb na mniejsze, łatwe w zarządzaniu podzakresy do takich rzeczy jak paginacja czy grupowanie zadań. Metoda `pairs` zwraca tablicę tablic, gdzie każda wewnętrzna tablica reprezentuje parę (podzakres) liczb:

```php
use Illuminate\Support\Number;

$result = Number::pairs(25, 10);

// [[0, 9], [10, 19], [20, 25]]

$result = Number::pairs(25, 10, offset: 0);

// [[0, 10], [10, 20], [20, 25]]
```

<a name="method-number-parse-int"></a>
#### `Number::parseInt()` {.collection-method}

Metoda `Number::parseInt` parsuje string na liczbę całkowitą zgodnie z określonymi ustawieniami regionalnymi:

```php
use Illuminate\Support\Number;

$result = Number::parseInt('10.123');

// (int) 10

$result = Number::parseInt('10,123', locale: 'fr');

// (int) 10
```

<a name="method-number-parse-float"></a>
#### `Number::parseFloat()` {.collection-method}

Metoda `Number::parseFloat` parsuje string na liczbę zmiennoprzecinkową zgodnie z określonymi ustawieniami regionalnymi:

```php
use Illuminate\Support\Number;

$result = Number::parseFloat('10');

// (float) 10.0

$result = Number::parseFloat('10', locale: 'fr');

// (float) 10.0
```

<a name="method-number-percentage"></a>
#### `Number::percentage()` {.collection-method}

Metoda `Number::percentage` zwraca reprezentację procentową podanej wartości jako string:

```php
use Illuminate\Support\Number;

$percentage = Number::percentage(10);

// 10%

$percentage = Number::percentage(10, precision: 2);

// 10.00%

$percentage = Number::percentage(10.123, maxPrecision: 2);

// 10.12%

$percentage = Number::percentage(10, precision: 2, locale: 'de');

// 10,00%
```

<a name="method-number-spell"></a>
#### `Number::spell()` {.collection-method}

Metoda `Number::spell` przekształca podaną liczbę na string słów:

```php
use Illuminate\Support\Number;

$number = Number::spell(102);

// one hundred and two

$number = Number::spell(88, locale: 'fr');

// quatre-vingt-huit
```

Argument `after` pozwala określić wartość, po której wszystkie liczby powinny być wypisane słownie:

```php
$number = Number::spell(10, after: 10);

// 10

$number = Number::spell(11, after: 10);

// eleven
```

Argument `until` pozwala określić wartość, przed którą wszystkie liczby powinny być wypisane słownie:

```php
$number = Number::spell(5, until: 10);

// five

$number = Number::spell(10, until: 10);

// 10
```

<a name="method-number-spell-ordinal"></a>
#### `Number::spellOrdinal()` {.collection-method}

Metoda `Number::spellOrdinal` zwraca reprezentację porządkową liczby jako string słów:

```php
use Illuminate\Support\Number;

$number = Number::spellOrdinal(1);

// first

$number = Number::spellOrdinal(2);

// second

$number = Number::spellOrdinal(21);

// twenty-first
```

<a name="method-number-trim"></a>
#### `Number::trim()` {.collection-method}

Metoda `Number::trim` usuwa końcowe zera po przecinku dziesiętnym z podanej liczby:

```php
use Illuminate\Support\Number;

$number = Number::trim(12.0);

// 12

$number = Number::trim(12.30);

// 12.3
```

<a name="method-number-use-locale"></a>
#### `Number::useLocale()` {.collection-method}

Metoda `Number::useLocale` ustawia domyślne ustawienia regionalne dla liczb globalnie, co wpływa na sposób formatowania liczb i walut przez kolejne wywołania metod klasy `Number`:

```php
use Illuminate\Support\Number;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Number::useLocale('de');
}
```

<a name="method-number-with-locale"></a>
#### `Number::withLocale()` {.collection-method}

Metoda `Number::withLocale` wykonuje podane domknięcie używając określonych ustawień regionalnych, a następnie przywraca oryginalne ustawienia regionalne po wykonaniu callbacku:

```php
use Illuminate\Support\Number;

$number = Number::withLocale('de', function () {
    return Number::format(1500);
});
```

<a name="method-number-use-currency"></a>
#### `Number::useCurrency()` {.collection-method}

Metoda `Number::useCurrency` ustawia domyślną walutę globalnie, co wpływa na sposób formatowania waluty przez kolejne wywołania metod klasy `Number`:

```php
use Illuminate\Support\Number;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Number::useCurrency('GBP');
}
```

<a name="method-number-with-currency"></a>
#### `Number::withCurrency()` {.collection-method}

Metoda `Number::withCurrency` wykonuje podane domknięcie używając określonej waluty, a następnie przywraca oryginalną walutę po wykonaniu callbacku:

```php
use Illuminate\Support\Number;

$number = Number::withCurrency('GBP', function () {
    // ...
});
```

<a name="paths"></a>
## Ścieżki

<a name="method-app-path"></a>
#### `app_path()` {.collection-method}

Funkcja `app_path` zwraca pełną kwalifikowaną ścieżkę do katalogu `app` Twojej aplikacji. Możesz również użyć funkcji `app_path` do wygenerowania pełnej kwalifikowanej ścieżki do pliku względem katalogu aplikacji:

```php
$path = app_path();

$path = app_path('Http/Controllers/Controller.php');
```

<a name="method-base-path"></a>
#### `base_path()` {.collection-method}

Funkcja `base_path` zwraca pełną kwalifikowaną ścieżkę do katalogu głównego Twojej aplikacji. Możesz również użyć funkcji `base_path` do wygenerowania pełnej kwalifikowanej ścieżki do podanego pliku względem katalogu głównego projektu:

```php
$path = base_path();

$path = base_path('vendor/bin');
```

<a name="method-config-path"></a>
#### `config_path()` {.collection-method}

Funkcja `config_path` zwraca pełną kwalifikowaną ścieżkę do katalogu `config` Twojej aplikacji. Możesz również użyć funkcji `config_path` do wygenerowania pełnej kwalifikowanej ścieżki do podanego pliku w katalogu konfiguracji aplikacji:

```php
$path = config_path();

$path = config_path('app.php');
```

<a name="method-database-path"></a>
#### `database_path()` {.collection-method}

Funkcja `database_path` zwraca pełną kwalifikowaną ścieżkę do katalogu `database` Twojej aplikacji. Możesz również użyć funkcji `database_path` do wygenerowania pełnej kwalifikowanej ścieżki do podanego pliku w katalogu bazy danych:

```php
$path = database_path();

$path = database_path('factories/UserFactory.php');
```

<a name="method-lang-path"></a>
#### `lang_path()` {.collection-method}

Funkcja `lang_path` zwraca pełną kwalifikowaną ścieżkę do katalogu `lang` Twojej aplikacji. Możesz również użyć funkcji `lang_path` do wygenerowania pełnej kwalifikowanej ścieżki do podanego pliku w tym katalogu:

```php
$path = lang_path();

$path = lang_path('en/messages.php');
```

> [!NOTE]
> Domyślnie szkielet aplikacji Laravel nie zawiera katalogu `lang`. Jeśli chcesz dostosować pliki językowe Laravel, możesz je opublikować za pomocą komendy Artisan `lang:publish`.

<a name="method-public-path"></a>
#### `public_path()` {.collection-method}

Funkcja `public_path` zwraca pełną kwalifikowaną ścieżkę do katalogu `public` Twojej aplikacji. Możesz również użyć funkcji `public_path` do wygenerowania pełnej kwalifikowanej ścieżki do podanego pliku w katalogu publicznym:

```php
$path = public_path();

$path = public_path('css/app.css');
```

<a name="method-resource-path"></a>
#### `resource_path()` {.collection-method}

Funkcja `resource_path` zwraca pełną kwalifikowaną ścieżkę do katalogu `resources` Twojej aplikacji. Możesz również użyć funkcji `resource_path` do wygenerowania pełnej kwalifikowanej ścieżki do podanego pliku w katalogu zasobów:

```php
$path = resource_path();

$path = resource_path('sass/app.scss');
```

<a name="method-storage-path"></a>
#### `storage_path()` {.collection-method}

Funkcja `storage_path` zwraca pełną kwalifikowaną ścieżkę do katalogu `storage` Twojej aplikacji. Możesz również użyć funkcji `storage_path` do wygenerowania pełnej kwalifikowanej ścieżki do podanego pliku w katalogu storage:

```php
$path = storage_path();

$path = storage_path('app/file.txt');
```

<a name="urls"></a>
## Adresy URL

<a name="method-action"></a>
#### `action()` {.collection-method}

Funkcja `action` generuje URL dla podanej akcji kontrolera:

```php
use App\Http\Controllers\HomeController;

$url = action([HomeController::class, 'index']);
```

Jeśli metoda akceptuje parametry trasy, możesz przekazać je jako drugi argument do metody:

```php
$url = action([UserController::class, 'profile'], ['id' => 1]);
```

<a name="method-asset"></a>
#### `asset()` {.collection-method}

Funkcja `asset` generuje URL dla zasobu używając bieżącego schematu żądania (HTTP lub HTTPS):

```php
$url = asset('img/photo.jpg');
```

Możesz skonfigurować hosta URL zasobów ustawiając zmienną `ASSET_URL` w pliku `.env`. Może to być przydatne, jeśli hostujesz swoje zasoby na zewnętrznej usłudze takiej jak Amazon S3 lub innym CDN:

```php
// ASSET_URL=http://example.com/assets

$url = asset('img/photo.jpg'); // http://example.com/assets/img/photo.jpg
```

<a name="method-route"></a>
#### `route()` {.collection-method}

Funkcja `route` generuje URL dla podanej [nazwanej trasy](/docs/{{version}}/routing#named-routes):

```php
$url = route('route.name');
```

Jeśli trasa akceptuje parametry, możesz przekazać je jako drugi argument do funkcji:

```php
$url = route('route.name', ['id' => 1]);
```

Domyślnie funkcja `route` generuje bezwzględny URL. Jeśli chcesz wygenerować względny URL, możesz przekazać `false` jako trzeci argument do funkcji:

```php
$url = route('route.name', ['id' => 1], false);
```

<a name="method-secure-asset"></a>
#### `secure_asset()` {.collection-method}

Funkcja `secure_asset` generuje URL dla zasobu używając HTTPS:

```php
$url = secure_asset('img/photo.jpg');
```

<a name="method-secure-url"></a>
#### `secure_url()` {.collection-method}

Funkcja `secure_url` generuje pełny kwalifikowany URL HTTPS do podanej ścieżki. Dodatkowe segmenty URL mogą być przekazane w drugim argumencie funkcji:

```php
$url = secure_url('user/profile');

$url = secure_url('user/profile', [1]);
```

<a name="method-to-action"></a>
#### `to_action()` {.collection-method}

Funkcja `to_action` generuje [odpowiedź HTTP z przekierowaniem](/docs/{{version}}/responses#redirects) dla podanej akcji kontrolera:

```php
use App\Http\Controllers\UserController;

return to_action([UserController::class, 'show'], ['user' => 1]);
```

Jeśli to konieczne, możesz przekazać kod statusu HTTP, który powinien być przypisany do przekierowania oraz wszelkie dodatkowe nagłówki odpowiedzi jako trzeci i czwarty argument do metody `to_action`:

```php
return to_action(
    [UserController::class, 'show'],
    ['user' => 1],
    302,
    ['X-Framework' => 'Laravel']
);
```

<a name="method-to-route"></a>
#### `to_route()` {.collection-method}

Funkcja `to_route` generuje [odpowiedź HTTP z przekierowaniem](/docs/{{version}}/responses#redirects) dla podanej [nazwanej trasy](/docs/{{version}}/routing#named-routes):

```php
return to_route('users.show', ['user' => 1]);
```

Jeśli to konieczne, możesz przekazać kod statusu HTTP, który powinien być przypisany do przekierowania oraz wszelkie dodatkowe nagłówki odpowiedzi jako trzeci i czwarty argument do metody `to_route`:

```php
return to_route('users.show', ['user' => 1], 302, ['X-Framework' => 'Laravel']);
```

<a name="method-uri"></a>
#### `uri()` {.collection-method}

Funkcja `uri` generuje [płynną instancję URI](#uri) dla podanego URI:

```php
$uri = uri('https://example.com')
    ->withPath('/users')
    ->withQuery(['page' => 1]);
```

Jeśli funkcja `uri` otrzyma tablicę zawierającą parę kontrolera i metody wywoływalnej, funkcja utworzy instancję `Uri` dla ścieżki trasy metody kontrolera:

```php
use App\Http\Controllers\UserController;

$uri = uri([UserController::class, 'show'], ['user' => $user]);
```

Jeśli kontroler jest wywoływalny, możesz po prostu podać nazwę klasy kontrolera:

```php
use App\Http\Controllers\UserIndexController;

$uri = uri(UserIndexController::class);
```

Jeśli wartość podana funkcji `uri` pasuje do nazwy [nazwanej trasy](/docs/{{version}}/routing#named-routes), instancja `Uri` zostanie wygenerowana dla ścieżki tej trasy:

```php
$uri = uri('users.show', ['user' => $user]);
```

<a name="method-url"></a>
#### `url()` {.collection-method}

Funkcja `url` generuje pełny kwalifikowany URL do podanej ścieżki:

```php
$url = url('user/profile');

$url = url('user/profile', [1]);
```

Jeśli nie podano ścieżki, zwracana jest instancja `Illuminate\Routing\UrlGenerator`:

```php
$current = url()->current();

$full = url()->full();

$previous = url()->previous();
```

Aby uzyskać więcej informacji na temat pracy z funkcją `url`, zapoznaj się z [dokumentacją generowania URL](/docs/{{version}}/urls#generating-urls).

<a name="miscellaneous"></a>
## Różne

<a name="method-abort"></a>
#### `abort()` {.collection-method}

Funkcja `abort` rzuca [wyjątek HTTP](/docs/{{version}}/errors#http-exceptions), który zostanie wyrenderowany przez [obsługę wyjątków](/docs/{{version}}/errors#handling-exceptions):

```php
abort(403);
```

Możesz również podać komunikat wyjątku i niestandardowe nagłówki odpowiedzi HTTP, które powinny być wysłane do przeglądarki:

```php
abort(403, 'Unauthorized.', $headers);
```

<a name="method-abort-if"></a>
#### `abort_if()` {.collection-method}

Funkcja `abort_if` rzuca wyjątek HTTP, jeśli podane wyrażenie boolowskie zwraca `true`:

```php
abort_if(! Auth::user()->isAdmin(), 403);
```

Podobnie jak metoda `abort`, możesz również podać tekst odpowiedzi wyjątku jako trzeci argument oraz tablicę niestandardowych nagłówków odpowiedzi jako czwarty argument funkcji.

<a name="method-abort-unless"></a>
#### `abort_unless()` {.collection-method}

Funkcja `abort_unless` rzuca wyjątek HTTP, jeśli podane wyrażenie boolowskie zwraca `false`:

```php
abort_unless(Auth::user()->isAdmin(), 403);
```

Podobnie jak metoda `abort`, możesz również podać tekst odpowiedzi wyjątku jako trzeci argument oraz tablicę niestandardowych nagłówków odpowiedzi jako czwarty argument funkcji.

<a name="method-app"></a>
#### `app()` {.collection-method}

Funkcja `app` zwraca instancję [kontenera usług](/docs/{{version}}/container):

```php
$container = app();
```

Możesz przekazać nazwę klasy lub interfejsu, aby rozwiązać go z kontenera:

```php
$api = app('HelpSpot\API');
```

<a name="method-auth"></a>
#### `auth()` {.collection-method}

Funkcja `auth` zwraca instancję [uwierzytelniania](/docs/{{version}}/authentication). Możesz jej użyć jako alternatywy dla fasady `Auth`:

```php
$user = auth()->user();
```

Jeśli to konieczne, możesz określić, do której instancji straży chcesz uzyskać dostęp:

```php
$user = auth('admin')->user();
```

<a name="method-back"></a>
#### `back()` {.collection-method}

Funkcja `back` generuje [odpowiedź HTTP z przekierowaniem](/docs/{{version}}/responses#redirects) do poprzedniej lokalizacji użytkownika:

```php
return back($status = 302, $headers = [], $fallback = '/');

return back();
```

<a name="method-bcrypt"></a>
#### `bcrypt()` {.collection-method}

Funkcja `bcrypt` [haszuje](/docs/{{version}}/hashing) podaną wartość używając Bcrypt. Możesz użyć tej funkcji jako alternatywy dla fasady `Hash`:

```php
$password = bcrypt('my-secret-password');
```

<a name="method-blank"></a>
#### `blank()` {.collection-method}

Funkcja `blank` określa, czy podana wartość jest "pusta":

```php
blank('');
blank('   ');
blank(null);
blank(collect());

// true

blank(0);
blank(true);
blank(false);

// false
```

Dla odwrotności funkcji `blank`, zobacz funkcję [filled](#method-filled).

<a name="method-broadcast"></a>
#### `broadcast()` {.collection-method}

Funkcja `broadcast` [emituje](/docs/{{version}}/broadcasting) dane [zdarzenie](/docs/{{version}}/events) do jego odbiorców:

```php
broadcast(new UserRegistered($user));

broadcast(new UserRegistered($user))->toOthers();
```

<a name="method-broadcast-if"></a>
#### `broadcast_if()` {.collection-method}

Funkcja `broadcast_if` [emituje](/docs/{{version}}/broadcasting) dane [zdarzenie](/docs/{{version}}/events) do jego odbiorców, jeśli podane wyrażenie boolowskie jest równe `true`:

```php
broadcast_if($user->isActive(), new UserRegistered($user));

broadcast_if($user->isActive(), new UserRegistered($user))->toOthers();
```

<a name="method-broadcast-unless"></a>
#### `broadcast_unless()` {.collection-method}

Funkcja `broadcast_unless` [emituje](/docs/{{version}}/broadcasting) dane [zdarzenie](/docs/{{version}}/events) do jego odbiorców, jeśli podane wyrażenie boolowskie jest równe `false`:

```php
broadcast_unless($user->isBanned(), new UserRegistered($user));

broadcast_unless($user->isBanned(), new UserRegistered($user))->toOthers();
```

<a name="method-cache"></a>
#### `cache()` {.collection-method}

Funkcja `cache` może być użyta do pobierania wartości z [cache](/docs/{{version}}/cache). Jeśli podany klucz nie istnieje w cache, zostanie zwrócona opcjonalna wartość domyślna:

```php
$value = cache('key');

$value = cache('key', 'default');
```

Możesz dodawać elementy do cache, przekazując do funkcji tablicę par klucz/wartość. Powinieneś również przekazać liczbę sekund lub czas trwania, przez jaki wartość cache powinna być uznawana za prawidłową:

```php
cache(['key' => 'value'], 300);

cache(['key' => 'value'], now()->plus(seconds: 10));
```

<a name="method-class-uses-recursive"></a>
#### `class_uses_recursive()` {.collection-method}

Funkcja `class_uses_recursive` zwraca wszystkie traity używane przez klasę, włączając traity używane przez wszystkie jej klasy nadrzędne:

```php
$traits = class_uses_recursive(App\Models\User::class);
```

<a name="method-collect"></a>
#### `collect()` {.collection-method}

Funkcja `collect` tworzy instancję [kolekcji](/docs/{{version}}/collections) z podanej wartości:

```php
$collection = collect(['Taylor', 'Abigail']);
```

<a name="method-config"></a>
#### `config()` {.collection-method}

Funkcja `config` pobiera wartość zmiennej [konfiguracyjnej](/docs/{{version}}/configuration). Wartości konfiguracyjne mogą być dostępne przy użyciu składni "kropkowej", która zawiera nazwę pliku i opcję, do której chcesz uzyskać dostęp. Możesz również podać wartość domyślną, która zostanie zwrócona, jeśli opcja konfiguracyjna nie istnieje:

```php
$value = config('app.timezone');

$value = config('app.timezone', $default);
```

Możesz ustawiać zmienne konfiguracyjne w czasie wykonywania, przekazując tablicę par klucz/wartość. Należy jednak pamiętać, że funkcja ta wpływa tylko na wartość konfiguracji dla bieżącego żądania i nie aktualizuje rzeczywistych wartości konfiguracyjnych:

```php
config(['app.debug' => true]);
```

<a name="method-context"></a>
#### `context()` {.collection-method}

Funkcja `context` pobiera wartość z bieżącego [kontekstu](/docs/{{version}}/context). Możesz również podać wartość domyślną, która zostanie zwrócona, jeśli klucz kontekstu nie istnieje:

```php
$value = context('trace_id');

$value = context('trace_id', $default);
```

Możesz ustawiać wartości kontekstu, przekazując tablicę par klucz/wartość:

```php
use Illuminate\Support\Str;

context(['trace_id' => Str::uuid()->toString()]);
```

<a name="method-cookie"></a>
#### `cookie()` {.collection-method}

Funkcja `cookie` tworzy nową instancję [cookie](/docs/{{version}}/requests#cookies):

```php
$cookie = cookie('name', 'value', $minutes);
```

<a name="method-csrf-field"></a>
#### `csrf_field()` {.collection-method}

Funkcja `csrf_field` generuje ukryte pole wejściowe HTML zawierające wartość tokenu CSRF. Na przykład, używając [składni Blade](/docs/{{version}}/blade):

```blade
{{ csrf_field() }}
```

<a name="method-csrf-token"></a>
#### `csrf_token()` {.collection-method}

Funkcja `csrf_token` pobiera wartość bieżącego tokenu CSRF:

```php
$token = csrf_token();
```

<a name="method-decrypt"></a>
#### `decrypt()` {.collection-method}

Funkcja `decrypt` [odszyfrowuje](/docs/{{version}}/encryption) podaną wartość. Możesz użyć tej funkcji jako alternatywy dla fasady `Crypt`:

```php
$password = decrypt($value);
```

Dla odwrotności funkcji `decrypt`, zobacz funkcję [encrypt](#method-encrypt).

<a name="method-dd"></a>
#### `dd()` {.collection-method}

Funkcja `dd` wyświetla podane zmienne i kończy wykonywanie skryptu:

```php
dd($value);

dd($value1, $value2, $value3, ...);
```

Jeśli nie chcesz zatrzymać wykonywania skryptu, użyj zamiast tego funkcji [dump](#method-dump).

<a name="method-dispatch"></a>
#### `dispatch()` {.collection-method}

Funkcja `dispatch` umieszcza dane [zadanie](/docs/{{version}}/queues#creating-jobs) w [kolejce zadań](/docs/{{version}}/queues) Laravel:

```php
dispatch(new App\Jobs\SendEmails);
```

<a name="method-dispatch-sync"></a>
#### `dispatch_sync()` {.collection-method}

Funkcja `dispatch_sync` umieszcza dane zadanie w kolejce [sync](/docs/{{version}}/queues#synchronous-dispatching), aby zostało przetworzone natychmiast:

```php
dispatch_sync(new App\Jobs\SendEmails);
```

<a name="method-dump"></a>
#### `dump()` {.collection-method}

Funkcja `dump` wyświetla podane zmienne:

```php
dump($value);

dump($value1, $value2, $value3, ...);
```

Jeśli chcesz zatrzymać wykonywanie skryptu po wyświetleniu zmiennych, użyj zamiast tego funkcji [dd](#method-dd).

<a name="method-encrypt"></a>
#### `encrypt()` {.collection-method}

Funkcja `encrypt` [szyfruje](/docs/{{version}}/encryption) podaną wartość. Możesz użyć tej funkcji jako alternatywy dla fasady `Crypt`:

```php
$secret = encrypt('my-secret-value');
```

Dla odwrotności funkcji `encrypt`, zobacz funkcję [decrypt](#method-decrypt).

<a name="method-env"></a>
#### `env()` {.collection-method}

Funkcja `env` pobiera wartość [zmiennej środowiskowej](/docs/{{version}}/configuration#environment-configuration) lub zwraca wartość domyślną:

```php
$env = env('APP_ENV');

$env = env('APP_ENV', 'production');
```

> [!WARNING]
> Jeśli wykonujesz polecenie `config:cache` podczas procesu wdrażania, powinieneś upewnić się, że wywołujesz funkcję `env` tylko z poziomu plików konfiguracyjnych. Po zapisaniu konfiguracji w cache, plik `.env` nie zostanie załadowany, a wszystkie wywołania funkcji `env` zwrócą zewnętrzne zmienne środowiskowe, takie jak zmienne środowiskowe na poziomie serwera lub systemu, lub `null`.

<a name="method-event"></a>
#### `event()` {.collection-method}

Funkcja `event` wysyła dane [zdarzenie](/docs/{{version}}/events) do jego odbiorców:

```php
event(new UserRegistered($user));
```

<a name="method-fake"></a>
#### `fake()` {.collection-method}

Funkcja `fake` rozwiązuje singleton [Faker](https://github.com/FakerPHP/Faker) z kontenera, co może być przydatne podczas tworzenia fałszywych danych w fabrykach modeli, seedowaniu bazy danych, testach i prototypowaniu widoków:

```blade
@for ($i = 0; $i < 10; $i++)
    <dl>
        <dt>Name</dt>
        <dd>{{ fake()->name() }}</dd>

        <dt>Email</dt>
        <dd>{{ fake()->unique()->safeEmail() }}</dd>
    </dl>
@endfor
```

Domyślnie funkcja `fake` będzie korzystać z opcji konfiguracyjnej `app.faker_locale` w Twojej konfiguracji `config/app.php`. Zazwyczaj ta opcja konfiguracyjna jest ustawiana za pomocą zmiennej środowiskowej `APP_FAKER_LOCALE`. Możesz również określić ustawienia regionalne, przekazując je do funkcji `fake`. Każde ustawienie regionalne rozwiąże indywidualny singleton:

```php
fake('nl_NL')->name()
```

<a name="method-filled"></a>
#### `filled()` {.collection-method}

Funkcja `filled` określa, czy podana wartość nie jest "pusta":

```php
filled(0);
filled(true);
filled(false);

// true

filled('');
filled('   ');
filled(null);
filled(collect());

// false
```

Dla odwrotności funkcji `filled`, zobacz funkcję [blank](#method-blank).

<a name="method-info"></a>
#### `info()` {.collection-method}

Funkcja `info` zapisze informacje do [dziennika](/docs/{{version}}/logging) Twojej aplikacji:

```php
info('Some helpful information!');
```

Do funkcji można również przekazać tablicę danych kontekstowych:

```php
info('User login attempt failed.', ['id' => $user->id]);
```

<a name="method-literal"></a>
#### `literal()` {.collection-method}

Funkcja `literal` tworzy nową instancję [stdClass](https://www.php.net/manual/en/class.stdclass.php) z podanymi nazwanymi argumentami jako właściwościami:

```php
$obj = literal(
    name: 'Joe',
    languages: ['PHP', 'Ruby'],
);

$obj->name; // 'Joe'
$obj->languages; // ['PHP', 'Ruby']
```

<a name="method-logger"></a>
#### `logger()` {.collection-method}

Funkcja `logger` może być użyta do zapisania wiadomości na poziomie `debug` do [dziennika](/docs/{{version}}/logging):

```php
logger('Debug message');
```

Do funkcji można również przekazać tablicę danych kontekstowych:

```php
logger('User has logged in.', ['id' => $user->id]);
```

Instancja [loggera](/docs/{{version}}/logging) zostanie zwrócona, jeśli do funkcji nie zostanie przekazana żadna wartość:

```php
logger()->error('You are not allowed here.');
```

<a name="method-method-field"></a>
#### `method_field()` {.collection-method}

Funkcja `method_field` generuje ukryte pole wejściowe HTML zawierające sfałszowaną wartość czasownika HTTP formularza. Na przykład, używając [składni Blade](/docs/{{version}}/blade):

```blade
<form method="POST">
    {{ method_field('DELETE') }}
</form>
```

<a name="method-now"></a>
#### `now()` {.collection-method}

Funkcja `now` tworzy nową instancję `Illuminate\Support\Carbon` dla bieżącego czasu:

```php
$now = now();
```

<a name="method-old"></a>
#### `old()` {.collection-method}

Funkcja `old` [pobiera](/docs/{{version}}/requests#retrieving-input) wartość [starego wejścia](/docs/{{version}}/requests#old-input) zapisaną w sesji:

```php
$value = old('value');

$value = old('value', 'default');
```

Ponieważ "wartość domyślna" przekazywana jako drugi argument do funkcji `old` jest często atrybutem modelu Eloquent, Laravel pozwala po prostu przekazać cały model Eloquent jako drugi argument do funkcji `old`. Podczas wykonywania tej operacji Laravel przyjmie, że pierwszy argument przekazany do funkcji `old` to nazwa atrybutu Eloquent, który powinien być traktowany jako "wartość domyślna":

```blade
{{ old('name', $user->name) }}

// Is equivalent to...

{{ old('name', $user) }}
```

<a name="method-once"></a>
#### `once()` {.collection-method}

Funkcja `once` wykonuje podane wywołanie zwrotne i zapisuje wynik w pamięci na czas trwania żądania. Wszystkie kolejne wywołania funkcji `once` z tym samym wywołaniem zwrotnym zwrócą wcześniej zapisany w cache wynik:

```php
function random(): int
{
    return once(function () {
        return random_int(1, 1000);
    });
}

random(); // 123
random(); // 123 (cached result)
random(); // 123 (cached result)
```

Gdy funkcja `once` jest wykonywana z poziomu instancji obiektu, zapisany w cache wynik będzie unikalny dla tej instancji obiektu:

```php
<?php

class NumberService
{
    public function all(): array
    {
        return once(fn () => [1, 2, 3]);
    }
}

$service = new NumberService;

$service->all();
$service->all(); // (cached result)

$secondService = new NumberService;

$secondService->all();
$secondService->all(); // (cached result)
```
<a name="method-optional"></a>
#### `optional()` {.collection-method}

Funkcja `optional` akceptuje dowolny argument i pozwala na dostęp do właściwości lub wywołanie metod na tym obiekcie. Jeśli podany obiekt to `null`, właściwości i metody zwrócą `null` zamiast powodować błąd:

```php
return optional($user->address)->street;

{!! old('name', optional($user)->name) !!}
```

Funkcja `optional` akceptuje również domknięcie jako drugi argument. Domknięcie zostanie wywołane, jeśli wartość przekazana jako pierwszy argument nie jest null:

```php
return optional(User::find($id), function (User $user) {
    return $user->name;
});
```

<a name="method-policy"></a>
#### `policy()` {.collection-method}

Metoda `policy` pobiera instancję [polityki](/docs/{{version}}/authorization#creating-policies) dla danej klasy:

```php
$policy = policy(App\Models\User::class);
```

<a name="method-redirect"></a>
#### `redirect()` {.collection-method}

Funkcja `redirect` zwraca [odpowiedź HTTP przekierowania](/docs/{{version}}/responses#redirects) lub zwraca instancję redirectora, jeśli zostanie wywołana bez argumentów:

```php
return redirect($to = null, $status = 302, $headers = [], $secure = null);

return redirect('/home');

return redirect()->route('route.name');
```

<a name="method-report"></a>
#### `report()` {.collection-method}

Funkcja `report` zgłosi wyjątek za pomocą Twojego [obsługiwacza wyjątków](/docs/{{version}}/errors#handling-exceptions):

```php
report($e);
```

Funkcja `report` akceptuje również ciąg znaków jako argument. Gdy do funkcji przekazany zostanie ciąg znaków, funkcja utworzy wyjątek z podanym ciągiem jako jego wiadomością:

```php
report('Something went wrong.');
```

<a name="method-report-if"></a>
#### `report_if()` {.collection-method}

Funkcja `report_if` zgłosi wyjątek za pomocą Twojego [obsługiwacza wyjątków](/docs/{{version}}/errors#handling-exceptions), jeśli podane wyrażenie boolowskie jest równe `true`:

```php
report_if($shouldReport, $e);

report_if($shouldReport, 'Something went wrong.');
```

<a name="method-report-unless"></a>
#### `report_unless()` {.collection-method}

Funkcja `report_unless` zgłosi wyjątek za pomocą Twojego [obsługiwacza wyjątków](/docs/{{version}}/errors#handling-exceptions), jeśli podane wyrażenie boolowskie jest równe `false`:

```php
report_unless($reportingDisabled, $e);

report_unless($reportingDisabled, 'Something went wrong.');
```

<a name="method-request"></a>
#### `request()` {.collection-method}

Funkcja `request` zwraca bieżącą instancję [żądania](/docs/{{version}}/requests) lub pobiera wartość pola wejściowego z bieżącego żądania:

```php
$request = request();

$value = request('key', $default);
```

<a name="method-rescue"></a>
#### `rescue()` {.collection-method}

Funkcja `rescue` wykonuje podane domknięcie i przechwytuje wszelkie wyjątki, które wystąpią podczas jego wykonywania. Wszystkie przechwycone wyjątki zostaną wysłane do Twojego [obsługiwacza wyjątków](/docs/{{version}}/errors#handling-exceptions); jednak żądanie będzie kontynuowane:

```php
return rescue(function () {
    return $this->method();
});
```

Możesz również przekazać drugi argument do funkcji `rescue`. Ten argument będzie wartością "domyślną", która powinna zostać zwrócona, jeśli wystąpi wyjątek podczas wykonywania domknięcia:

```php
return rescue(function () {
    return $this->method();
}, false);

return rescue(function () {
    return $this->method();
}, function () {
    return $this->failure();
});
```

Argument `report` może zostać przekazany do funkcji `rescue`, aby określić, czy wyjątek powinien być zgłoszony za pomocą funkcji `report`:

```php
return rescue(function () {
    return $this->method();
}, report: function (Throwable $throwable) {
    return $throwable instanceof InvalidArgumentException;
});
```

<a name="method-resolve"></a>
#### `resolve()` {.collection-method}

Funkcja `resolve` rozwiązuje daną nazwę klasy lub interfejsu na instancję za pomocą [kontenera usług](/docs/{{version}}/container):

```php
$api = resolve('HelpSpot\API');
```

<a name="method-response"></a>
#### `response()` {.collection-method}

Funkcja `response` tworzy instancję [odpowiedzi](/docs/{{version}}/responses) lub pobiera instancję fabryki odpowiedzi:

```php
return response('Hello World', 200, $headers);

return response()->json(['foo' => 'bar'], 200, $headers);
```

<a name="method-retry"></a>
#### `retry()` {.collection-method}

Funkcja `retry` próbuje wykonać podane wywołanie zwrotne do momentu osiągnięcia maksymalnej liczby prób. Jeśli wywołanie zwrotne nie zgłosi wyjątku, jego wartość zwracana zostanie zwrócona. Jeśli wywołanie zwrotne zgłosi wyjątek, zostanie automatycznie ponowione. Jeśli maksymalna liczba prób zostanie przekroczona, wyjątek zostanie zgłoszony:

```php
return retry(5, function () {
    // Attempt 5 times while resting 100ms between attempts...
}, 100);
```

Jeśli chcesz ręcznie obliczyć liczbę milisekund do uśpienia między próbami, możesz przekazać domknięcie jako trzeci argument do funkcji `retry`:

```php
use Exception;

return retry(5, function () {
    // ...
}, function (int $attempt, Exception $exception) {
    return $attempt * 100;
});
```

Dla wygody możesz przekazać tablicę jako pierwszy argument do funkcji `retry`. Ta tablica będzie użyta do określenia, ile milisekund należy odczekać między kolejnymi próbami:

```php
return retry([100, 200], function () {
    // Sleep for 100ms on first retry, 200ms on second retry...
});
```

Aby ponowić próbę tylko w określonych warunkach, możesz przekazać domknięcie jako czwarty argument do funkcji `retry`:

```php
use App\Exceptions\TemporaryException;
use Exception;

return retry(5, function () {
    // ...
}, 100, function (Exception $exception) {
    return $exception instanceof TemporaryException;
});
```

<a name="method-session"></a>
#### `session()` {.collection-method}

Funkcja `session` może być użyta do pobierania lub ustawiania wartości [sesji](/docs/{{version}}/session):

```php
$value = session('key');
```

Możesz ustawiać wartości, przekazując tablicę par klucz/wartość do funkcji:

```php
session(['chairs' => 7, 'instruments' => 3]);
```

Magazyn sesji zostanie zwrócony, jeśli do funkcji nie zostanie przekazana żadna wartość:

```php
$value = session()->get('key');

session()->put('key', $value);
```

<a name="method-tap"></a>
#### `tap()` {.collection-method}

Funkcja `tap` akceptuje dwa argumenty: dowolną wartość `$value` i domknięcie. Wartość `$value` zostanie przekazana do domknięcia, a następnie zostanie zwrócona przez funkcję `tap`. Wartość zwracana przez domknięcie jest nieistotna:

```php
$user = tap(User::first(), function (User $user) {
    $user->name = 'Taylor';

    $user->save();
});
```

Jeśli do funkcji `tap` nie zostanie przekazane domknięcie, możesz wywołać dowolną metodę na podanej wartości `$value`. Wartość zwracana przez wywoływaną metodę zawsze będzie `$value`, niezależnie od tego, co metoda faktycznie zwraca w swojej definicji. Na przykład metoda `update` Eloquent zazwyczaj zwraca liczbę całkowitą. Możemy jednak wymusić, aby metoda zwróciła sam model, łącząc wywołanie metody `update` przez funkcję `tap`:

```php
$user = tap($user)->update([
    'name' => $name,
    'email' => $email,
]);
```

Aby dodać metodę `tap` do klasy, możesz dodać trait `Illuminate\Support\Traits\Tappable` do klasy. Metoda `tap` tego traita akceptuje domknięcie jako jedyny argument. Instancja obiektu zostanie przekazana do domknięcia, a następnie zostanie zwrócona przez metodę `tap`:

```php
return $user->tap(function (User $user) {
    // ...
});
```

<a name="method-throw-if"></a>
#### `throw_if()` {.collection-method}

Funkcja `throw_if` zgłasza podany wyjątek, jeśli podane wyrażenie boolowskie jest równe `true`:

```php
throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

throw_if(
    ! Auth::user()->isAdmin(),
    AuthorizationException::class,
    'You are not allowed to access this page.'
);
```

<a name="method-throw-unless"></a>
#### `throw_unless()` {.collection-method}

Funkcja `throw_unless` zgłasza podany wyjątek, jeśli podane wyrażenie boolowskie jest równe `false`:

```php
throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

throw_unless(
    Auth::user()->isAdmin(),
    AuthorizationException::class,
    'You are not allowed to access this page.'
);
```

<a name="method-today"></a>
#### `today()` {.collection-method}

Funkcja `today` tworzy nową instancję `Illuminate\Support\Carbon` dla bieżącej daty:

```php
$today = today();
```

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()` {.collection-method}

Funkcja `trait_uses_recursive` zwraca wszystkie traity używane przez trait:

```php
$traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);
```

<a name="method-transform"></a>
#### `transform()` {.collection-method}

Funkcja `transform` wykonuje domknięcie na podanej wartości, jeśli wartość nie jest [pusta](#method-blank), a następnie zwraca wartość zwracaną przez domknięcie:

```php
$callback = function (int $value) {
    return $value * 2;
};

$result = transform(5, $callback);

// 10
```

Wartość domyślna lub domknięcie może zostać przekazane jako trzeci argument do funkcji. Ta wartość zostanie zwrócona, jeśli podana wartość jest pusta:

```php
$result = transform(null, $callback, 'The value is blank');

// The value is blank
```

<a name="method-validator"></a>
#### `validator()` {.collection-method}

Funkcja `validator` tworzy nową instancję [walidatora](/docs/{{version}}/validation) z podanymi argumentami. Możesz jej użyć jako alternatywy dla fasady `Validator`:

```php
$validator = validator($data, $rules, $messages);
```

<a name="method-value"></a>
#### `value()` {.collection-method}

Funkcja `value` zwraca wartość, która została jej podana. Jednakże, jeśli przekażesz domknięcie do funkcji, domknięcie zostanie wykonane i zwrócona zostanie jego wartość:

```php
$result = value(true);

// true

$result = value(function () {
    return false;
});

// false
```

Dodatkowe argumenty mogą zostać przekazane do funkcji `value`. Jeśli pierwszy argument to domknięcie, dodatkowe parametry zostaną przekazane do domknięcia jako argumenty, w przeciwnym razie zostaną zignorowane:

```php
$result = value(function (string $name) {
    return $name;
}, 'Taylor');

// 'Taylor'
```

<a name="method-view"></a>
#### `view()` {.collection-method}

Funkcja `view` pobiera instancję [widoku](/docs/{{version}}/views):

```php
return view('auth.login');
```

<a name="method-with"></a>
#### `with()` {.collection-method}

Funkcja `with` zwraca wartość, która została jej podana. Jeśli domknięcie zostanie przekazane jako drugi argument do funkcji, domknięcie zostanie wykonane i zwrócona zostanie jego wartość:

```php
$callback = function (mixed $value) {
    return is_numeric($value) ? $value * 2 : 0;
};

$result = with(5, $callback);

// 10

$result = with(null, $callback);

// 0

$result = with(5, null);

// 5
```

<a name="method-when"></a>
#### `when()` {.collection-method}

Funkcja `when` zwraca wartość, która została jej podana, jeśli podany warunek jest równy `true`. W przeciwnym razie zwracana jest wartość `null`. Jeśli domknięcie zostanie przekazane jako drugi argument do funkcji, domknięcie zostanie wykonane i zwrócona zostanie jego wartość:

```php
$value = when(true, 'Hello World');

$value = when(true, fn () => 'Hello World');
```

Funkcja `when` jest szczególnie przydatna do warunkowego renderowania atrybutów HTML:

```blade
<div {!! when($condition, 'wire:poll="calculate"') !!}>
    ...
</div>
```

<a name="other-utilities"></a>
## Inne Narzędzia

<a name="benchmarking"></a>
### Benchmarking

Czasami możesz chcieć szybko przetestować wydajność określonych części swojej aplikacji. W takich sytuacjach możesz użyć klasy pomocniczej `Benchmark` do zmierzenia liczby milisekund, jakie zajmuje wykonanie podanych wywołań zwrotnych:

```php
<?php

use App\Models\User;
use Illuminate\Support\Benchmark;

Benchmark::dd(fn () => User::find(1)); // 0.1 ms

Benchmark::dd([
    'Scenario 1' => fn () => User::count(), // 0.5 ms
    'Scenario 2' => fn () => User::all()->count(), // 20.0 ms
]);
```

Domyślnie podane wywołania zwrotne zostaną wykonane raz (jedna iteracja), a ich czas trwania zostanie wyświetlony w przeglądarce / konsoli.

Aby wywołać wywołanie zwrotne więcej niż raz, możesz określić liczbę iteracji jako drugi argument metody. Podczas wykonywania wywołania zwrotnego więcej niż raz, klasa `Benchmark` zwróci średnią liczbę milisekund potrzebną na wykonanie wywołania zwrotnego we wszystkich iteracjach:

```php
Benchmark::dd(fn () => User::count(), iterations: 10); // 0.5 ms
```

Czasami możesz chcieć zmierzyć wykonanie wywołania zwrotnego, jednocześnie otrzymując wartość zwróconą przez wywołanie zwrotne. Metoda `value` zwróci krotkę zawierającą wartość zwróconą przez wywołanie zwrotne oraz liczbę milisekund potrzebną na jego wykonanie:

```php
[$count, $duration] = Benchmark::value(fn () => User::count());
```

<a name="dates"></a>
### Daty i Czas

Laravel zawiera [Carbon](https://carbon.nesbot.com/docs/), potężną bibliotekę do manipulacji datami i czasem. Aby utworzyć nową instancję `Carbon`, możesz wywołać funkcję `now`. Ta funkcja jest globalnie dostępna w Twojej aplikacji Laravel:

```php
$now = now();
```

Możesz też utworzyć nową instancję `Carbon` używając klasy `Illuminate\Support\Carbon`:

```php
use Illuminate\Support\Carbon;

$now = Carbon::now();
```

Laravel rozszerza również instancje `Carbon` o metody `plus` i `minus`, umożliwiając łatwą manipulację datą i czasem instancji:

```php
return now()->plus(minutes: 5);
return now()->plus(hours: 8);
return now()->plus(weeks: 4);

return now()->minus(minutes: 5);
return now()->minus(hours: 8);
return now()->minus(weeks: 4);
```

Aby zapoznać się z dokładnym omówieniem Carbon i jego funkcji, zapoznaj się z [oficjalną dokumentacją Carbon](https://carbon.nesbot.com/docs/).

<a name="interval-functions"></a>
#### Funkcje Interwałów

Laravel oferuje również funkcje `milliseconds`, `seconds`, `minutes`, `hours`, `days`, `weeks`, `months` i `years`, które zwracają instancje `CarbonInterval` rozszerzające klasę PHP [DateInterval](https://www.php.net/manual/en/class.dateinterval.php). Te funkcje mogą być używane wszędzie tam, gdzie Laravel akceptuje instancję `DateInterval`:

```php
use Illuminate\Support\Facades\Cache;

use function Illuminate\Support\{minutes};

Cache::put('metrics', $metrics, minutes(10));
```

<a name="deferred-functions"></a>
### Funkcje Odroczone

Chociaż [zadania kolejkowane](/docs/{{version}}/queues) Laravel pozwalają kolejkować zadania do przetwarzania w tle, czasami możesz mieć proste zadania, które chciałbyś odroczyć bez konfigurowania lub utrzymywania długo działającego workera kolejki.

Funkcje odroczone pozwalają odroczyć wykonanie domknięcia do czasu, aż odpowiedź HTTP zostanie wysłana do użytkownika, dzięki czemu Twoja aplikacja pozostaje szybka i responsywna. Aby odroczyć wykonanie domknięcia, po prostu przekaż domknięcie do funkcji `Illuminate\Support\defer`:

```php
use App\Services\Metrics;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use function Illuminate\Support\defer;

Route::post('/orders', function (Request $request) {
    // Create order...

    defer(fn () => Metrics::reportOrder($order));

    return $order;
});
```

Domyślnie funkcje odroczone będą wykonywane tylko wtedy, gdy odpowiedź HTTP, polecenie Artisan lub zadanie kolejkowane, z którego wywołano `Illuminate\Support\defer`, zakończy się pomyślnie. Oznacza to, że funkcje odroczone nie będą wykonywane, jeśli żądanie zakończy się odpowiedzią HTTP `4xx` lub `5xx`. Jeśli chcesz, aby funkcja odroczona zawsze była wykonywana, możesz dołączyć metodę `always` do swojej funkcji odroczonej:

```php
defer(fn () => Metrics::reportOrder($order))->always();
```

> [!WARNING]
> Jeśli masz zainstalowane [rozszerzenie Swoole PHP](https://www.php.net/manual/en/book.swoole.php), funkcja `defer` Laravel może kolidować z własną globalną funkcją `defer` Swoole, prowadząc do błędów serwera WWW. Upewnij się, że wywołujesz helper `defer` Laravel poprzez jawne wskazanie przestrzeni nazw: `use function Illuminate\Support\defer;`

<a name="cancelling-deferred-functions"></a>
#### Anulowanie Funkcji Odroczonych

Jeśli musisz anulować funkcję odroczoną przed jej wykonaniem, możesz użyć metody `forget`, aby anulować funkcję po jej nazwie. Aby nazwać funkcję odroczoną, podaj drugi argument do funkcji `Illuminate\Support\defer`:

```php
defer(fn () => Metrics::report(), 'reportMetrics');

defer()->forget('reportMetrics');
```

<a name="disabling-deferred-functions-in-tests"></a>
#### Wyłączanie Funkcji Odroczonych w Testach

Podczas pisania testów może być przydatne wyłączenie funkcji odroczonych. Możesz wywołać `withoutDefer` w swoim teście, aby polecić Laravel natychmiastowe wywołanie wszystkich funkcji odroczonych:

```php tab=Pest
test('without defer', function () {
    $this->withoutDefer();

    // ...
});
```

```php tab=PHPUnit
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_without_defer(): void
    {
        $this->withoutDefer();

        // ...
    }
}
```

Jeśli chcesz wyłączyć funkcje odroczone dla wszystkich testów w ramach przypadku testowego, możesz wywołać metodę `withoutDefer` z metody `setUp` w podstawowej klasie `TestCase`:

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    protected function setUp(): void// [tl! add:start]
    {
        parent::setUp();

        $this->withoutDefer();
    }// [tl! add:end]
}
```

<a name="lottery"></a>
### Loteria

Klasa loteryjna Laravel może być używana do wykonywania wywołań zwrotnych na podstawie zestawu podanych prawdopodobieństw. Może to być szczególnie przydatne, gdy chcesz wykonać kod tylko dla określonego procenta przychodzących żądań:

```php
use Illuminate\Support\Lottery;

Lottery::odds(1, 20)
    ->winner(fn () => $user->won())
    ->loser(fn () => $user->lost())
    ->choose();
```

Możesz łączyć klasę loteryjną Laravel z innymi funkcjami Laravel. Na przykład możesz chcieć raportować tylko niewielki procent wolnych zapytań do swojego handlera wyjątków. A ponieważ klasa loteryjna jest wywoływalna, możemy przekazać instancję klasy do dowolnej metody, która akceptuje wywoływalne obiekty:

```php
use Carbon\CarbonInterval;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Lottery;

DB::whenQueryingForLongerThan(
    CarbonInterval::seconds(2),
    Lottery::odds(1, 100)->winner(fn () => report('Querying > 2 seconds.')),
);
```

<a name="testing-lotteries"></a>
#### Testowanie Loterii

Laravel udostępnia kilka prostych metod, które pozwalają łatwo testować wywołania loterii w Twojej aplikacji:

```php
// Lottery will always win...
Lottery::alwaysWin();

// Lottery will always lose...
Lottery::alwaysLose();

// Lottery will win then lose, and finally return to normal behavior...
Lottery::fix([true, false]);

// Lottery will return to normal behavior...
Lottery::determineResultsNormally();
```

<a name="pipeline"></a>
### Pipeline (Potok)

Fasada `Pipeline` Laravel zapewnia wygodny sposób "przepuszczenia" danego wejścia przez serię wywoływalnych klas, domknięć lub wywoływalnych obiektów, dając każdej klasie możliwość sprawdzenia lub modyfikacji wejścia i wywołania następnego wywoływalnego obiektu w potoku:

```php
use Closure;
use App\Models\User;
use Illuminate\Support\Facades\Pipeline;

$user = Pipeline::send($user)
    ->through([
        function (User $user, Closure $next) {
            // ...

            return $next($user);
        },
        function (User $user, Closure $next) {
            // ...

            return $next($user);
        },
    ])
    ->then(fn (User $user) => $user);
```

Jak widać, każda wywoływalna klasa lub domknięcie w potoku otrzymuje wejście i domknięcie `$next`. Wywołanie domknięcia `$next` wywoła następny wywoływalny obiekt w potoku. Jak mogłeś zauważyć, jest to bardzo podobne do [middleware](/docs/{{version}}/middleware).

Gdy ostatni wywoływalny obiekt w potoku wywoła domknięcie `$next`, zostanie wywołany wywoływalny obiekt podany w metodzie `then`. Zazwyczaj ten wywoływalny obiekt po prostu zwróci podane wejście. Dla wygody, jeśli po prostu chcesz zwrócić wejście po jego przetworzeniu, możesz użyć metody `thenReturn`.

Oczywiście, jak wcześniej omówiono, nie jesteś ograniczony do przekazywania domknięć do swojego potoku. Możesz również dostarczać wywoływalne klasy. Jeśli zostanie podana nazwa klasy, klasa zostanie utworzona za pomocą [kontenera usług](/docs/{{version}}/container) Laravel, umożliwiając wstrzyknięcie zależności do wywoływalnej klasy:

```php
$user = Pipeline::send($user)
    ->through([
        GenerateProfilePhoto::class,
        ActivateSubscription::class,
        SendWelcomeEmail::class,
    ])
    ->thenReturn();
```

Metoda `withinTransaction` może być wywołana na potoku, aby automatycznie opakować wszystkie kroki potoku w pojedynczą transakcję bazodanową:

```php
$user = Pipeline::send($user)
    ->withinTransaction()
    ->through([
        ProcessOrder::class,
        TransferFunds::class,
        UpdateInventory::class,
    ])
    ->thenReturn();
```

<a name="sleep"></a>
### Uśpienie (Sleep)

Klasa `Sleep` Laravel jest lekkim opakowaniem natywnych funkcji PHP `sleep` i `usleep`, oferującym większą testowalność, jednocześnie udostępniając przyjazne dla programistów API do pracy z czasem:

```php
use Illuminate\Support\Sleep;

$waiting = true;

while ($waiting) {
    Sleep::for(1)->second();

    $waiting = /* ... */;
}
```

Klasa `Sleep` oferuje różnorodne metody, które pozwalają pracować z różnymi jednostkami czasu:

```php
// Return a value after sleeping...
$result = Sleep::for(1)->second()->then(fn () => 1 + 1);

// Sleep while a given value is true...
Sleep::for(1)->second()->while(fn () => shouldKeepSleeping());

// Pause execution for 90 seconds...
Sleep::for(1.5)->minutes();

// Pause execution for 2 seconds...
Sleep::for(2)->seconds();

// Pause execution for 500 milliseconds...
Sleep::for(500)->milliseconds();

// Pause execution for 5,000 microseconds...
Sleep::for(5000)->microseconds();

// Pause execution until a given time...
Sleep::until(now()->plus(minutes: 1));

// Alias of PHP's native "sleep" function...
Sleep::sleep(2);

// Alias of PHP's native "usleep" function...
Sleep::usleep(5000);
```

Aby łatwo łączyć jednostki czasu, możesz użyć metody `and`:

```php
Sleep::for(1)->second()->and(10)->milliseconds();
```

<a name="testing-sleep"></a>
#### Testowanie Uśpienia

Podczas testowania kodu, który wykorzystuje klasę `Sleep` lub natywne funkcje uśpienia PHP, Twój test wstrzyma wykonanie. Jak można się spodziewać, to znacznie spowalnia zestaw testów. Na przykład wyobraź sobie, że testujesz następujący kod:

```php
$waiting = /* ... */;

$seconds = 1;

while ($waiting) {
    Sleep::for($seconds++)->seconds();

    $waiting = /* ... */;
}
```

Zazwyczaj testowanie tego kodu zajęłoby _co najmniej_ jedną sekundę. Na szczęście klasa `Sleep` pozwala nam "udawać" uśpienie, dzięki czemu nasz zestaw testów pozostaje szybki:

```php tab=Pest
it('waits until ready', function () {
    Sleep::fake();

    // ...
});
```

```php tab=PHPUnit
public function test_it_waits_until_ready()
{
    Sleep::fake();

    // ...
}
```

Podczas udawania klasy `Sleep`, rzeczywista pauza wykonania jest omijana, co prowadzi do znacznie szybszego testu.

Po sfałszowaniu klasy `Sleep` możliwe jest tworzenie asercji dotyczących oczekiwanych "uśpień", które powinny wystąpić. Aby to zilustrować, wyobraźmy sobie, że testujemy kod, który wstrzymuje wykonanie trzy razy, przy czym każda pauza zwiększa się o jedną sekundę. Używając metody `assertSequence`, możemy sprawdzić, czy nasz kod "spał" przez odpowiednią ilość czasu, jednocześnie utrzymując nasz test szybkim:

```php tab=Pest
it('checks if ready three times', function () {
    Sleep::fake();

    // ...

    Sleep::assertSequence([
        Sleep::for(1)->second(),
        Sleep::for(2)->seconds(),
        Sleep::for(3)->seconds(),
    ]);
}
```

```php tab=PHPUnit
public function test_it_checks_if_ready_three_times()
{
    Sleep::fake();

    // ...

    Sleep::assertSequence([
        Sleep::for(1)->second(),
        Sleep::for(2)->seconds(),
        Sleep::for(3)->seconds(),
    ]);
}
```

Oczywiście klasa `Sleep` oferuje różne inne asercje, których możesz użyć podczas testowania:

```php
use Carbon\CarbonInterval as Duration;
use Illuminate\Support\Sleep;

// Assert that sleep was called 3 times...
Sleep::assertSleptTimes(3);

// Assert against the duration of sleep...
Sleep::assertSlept(function (Duration $duration): bool {
    return /* ... */;
}, times: 1);

// Assert that the Sleep class was never invoked...
Sleep::assertNeverSlept();

// Assert that, even if Sleep was called, no execution paused occurred...
Sleep::assertInsomniac();
```

Czasami może być przydatne wykonanie akcji za każdym razem, gdy wystąpi fałszywe uśpienie. Aby to osiągnąć, możesz dostarczyć wywołanie zwrotne do metody `whenFakingSleep`. W poniższym przykładzie używamy [helperów manipulacji czasem](/docs/{{version}}/mocking#interacting-with-time) Laravel, aby natychmiast przesunąć czas o czas trwania każdego uśpienia:

```php
use Carbon\CarbonInterval as Duration;

$this->freezeTime();

Sleep::fake();

Sleep::whenFakingSleep(function (Duration $duration) {
    // Progress time when faking sleep...
    $this->travel($duration->totalMilliseconds)->milliseconds();
});
```

Ponieważ przesuwanie czasu jest powszechnym wymaganiem, metoda `fake` akceptuje argument `syncWithCarbon`, aby utrzymać Carbon w synchronizacji podczas uśpienia w teście:

```php
Sleep::fake(syncWithCarbon: true);

$start = now();

Sleep::for(1)->second();

$start->diffForHumans(); // 1 second ago
```

Laravel używa klasy `Sleep` wewnętrznie za każdym razem, gdy wstrzymuje wykonanie. Na przykład helper [retry](#method-retry) używa klasy `Sleep` podczas uśpienia, umożliwiając lepszą testowalność podczas korzystania z tego helpera.

<a name="timebox"></a>
### Timebox (Ramka Czasowa)

Klasa `Timebox` Laravel zapewnia, że podane wywołanie zwrotne zawsze zajmuje stałą ilość czasu na wykonanie, nawet jeśli jego rzeczywiste wykonanie zakończy się wcześniej. Jest to szczególnie przydatne w operacjach kryptograficznych i sprawdzaniu uwierzytelniania użytkowników, gdzie atakujący mogą wykorzystywać różnice w czasie wykonania do wnioskowania wrażliwych informacji.

Jeśli wykonanie przekroczy stały czas trwania, `Timebox` nie ma efektu. To do developera należy wybór wystarczająco długiego czasu jako stały czas trwania, aby uwzględnić scenariusze najgorszego przypadku.

Metoda call akceptuje domknięcie i limit czasu w mikrosekundach, a następnie wykonuje domknięcie i czeka, aż limit czasu zostanie osiągnięty:

```php
use Illuminate\Support\Timebox;

(new Timebox)->call(function ($timebox) {
    // ...
}, microseconds: 10000);
```

Jeśli w domknięciu zostanie zgłoszony wyjątek, ta klasa przestrzega zdefiniowanego opóźnienia i ponownie zgłasza wyjątek po opóźnieniu.

<a name="uri"></a>
### URI

Klasa `Uri` Laravel zapewnia wygodny i płynny interfejs do tworzenia i manipulowania URI. Ta klasa owija funkcjonalność dostarczoną przez podstawowy pakiet League URI i bezproblemowo integruje się z systemem routingu Laravel.

Możesz łatwo utworzyć instancję `Uri` używając metod statycznych:

```php
use App\Http\Controllers\UserController;
use App\Http\Controllers\InvokableController;
use Illuminate\Support\Uri;

// Generate a URI instance from the given string...
$uri = Uri::of('https://example.com/path');

// Generate URI instances to paths, named routes, or controller actions...
$uri = Uri::to('/dashboard');
$uri = Uri::route('users.show', ['user' => 1]);
$uri = Uri::signedRoute('users.show', ['user' => 1]);
$uri = Uri::temporarySignedRoute('user.index', now()->plus(minutes: 5));
$uri = Uri::action([UserController::class, 'index']);
$uri = Uri::action(InvokableController::class);

// Generate a URI instance from the current request URL...
$uri = $request->uri();
```

Po uzyskaniu instancji URI możesz ją płynnie modyfikować:

```php
$uri = Uri::of('https://example.com')
    ->withScheme('http')
    ->withHost('test.com')
    ->withPort(8000)
    ->withPath('/users')
    ->withQuery(['page' => 2])
    ->withFragment('section-1');
```

<a name="inspecting-uris"></a>
#### Inspekcja URI

Klasa `Uri` pozwala również łatwo sprawdzać różne komponenty podstawowego URI:

```php
$scheme = $uri->scheme();
$host = $uri->host();
$port = $uri->port();
$path = $uri->path();
$segments = $uri->pathSegments();
$query = $uri->query();
$fragment = $uri->fragment();
```

<a name="manipulating-query-strings"></a>
#### Manipulowanie Ciągami Zapytań

Klasa `Uri` oferuje kilka metod, które mogą być użyte do manipulowania ciągiem zapytania URI. Metoda `withQuery` może być użyta do scalenia dodatkowych parametrów ciągu zapytania z istniejącym ciągiem zapytania:

```php
$uri = $uri->withQuery(['sort' => 'name']);
```

Metoda `withQueryIfMissing` może być użyta do scalenia dodatkowych parametrów ciągu zapytania z istniejącym ciągiem zapytania, jeśli podane klucze nie istnieją jeszcze w ciągu zapytania:

```php
$uri = $uri->withQueryIfMissing(['page' => 1]);
```

Metoda `replaceQuery` może być użyta do całkowitego zastąpienia istniejącego ciągu zapytania nowym:

```php
$uri = $uri->replaceQuery(['page' => 1]);
```

Metoda `pushOntoQuery` może być użyta do dodania dodatkowych parametrów do parametru ciągu zapytania, który ma wartość tablicy:

```php
$uri = $uri->pushOntoQuery('filter', ['active', 'pending']);
```

Metoda `withoutQuery` może być użyta do usunięcia parametrów z ciągu zapytania:

```php
$uri = $uri->withoutQuery(['page']);
```

<a name="generating-responses-from-uris"></a>
#### Generowanie Odpowiedzi z URI

Metoda `redirect` może być użyta do wygenerowania instancji `RedirectResponse` do podanego URI:

```php
$uri = Uri::of('https://example.com');

return $uri->redirect();
```

Możesz też po prostu zwrócić instancję `Uri` z trasy lub akcji kontrolera, co automatycznie wygeneruje odpowiedź przekierowania do zwróconego URI:

```php
use Illuminate\Support\Facades\Route;
use Illuminate\Support\Uri;

Route::get('/redirect', function () {
    return Uri::to('/index')
        ->withQuery(['sort' => 'name']);
});
```
