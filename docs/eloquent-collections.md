# Eloquent: Kolekcje

- [Wprowadzenie](#introduction)
- [Dostępne Metody](#available-.)
- [Custom Kolekcje](#custom-collections)

<a name="introduction"></a>
## Wprowadzenie

Wszystkie metody Eloquent, które zwracają więcej niż jeden wynik modelu, zwrócą instancje klasy `Illuminate\Database\Eloquent\Collection`, w tym wyniki pobrane za pomocą metody `get` lub dostępne przez relację. Obiekt kolekcji Eloquent rozszerza [bazową kolekcję](/docs/{{version}}/collections) Laravela, więc naturalnie dziedziczy dziesiątki metod używanych do płynnej pracy z podstawową tablicą modeli Eloquent. Koniecznie zapoznaj się z dokumentacją kolekcji Laravel, aby dowiedzieć się wszystkiego o tych pomocnych metodach!

Wszystkie kolekcje służą również jako iteratory, pozwalając na przechodzenie przez nie, jakby były zwykłymi tablicami PHP:

```php
use App\Models\User;

$users = User::where('active', 1)->get();

foreach ($users as $user) {
    echo $user->name;
}
```

Jednak, jak wcześniej wspomniano, kolekcje są znacznie potężniejsze niż tablice i udostępniają różnorodne operacje map/reduce, które można łączyć w łańcuch przy użyciu intuicyjnego interfejsu. Na przykład możemy usunąć wszystkie nieaktywne modele i następnie pobrać imię dla każdego pozostałego użytkownika:

```php
$names = User::all()->reject(function (User $user) {
    return $user->active === false;
})->map(function (User $user) {
    return $user->name;
});
```

<a name="eloquent-collection-conversion"></a>
#### Konwersja Kolekcji Eloquent

Podczas gdy większość metod kolekcji Eloquent zwraca nową instancję kolekcji Eloquent, metody `collapse`, `flatten`, `flip`, `keys`, `pluck` i `zip` zwracają [bazową kolekcję](/docs/{{version}}/collections). Podobnie, jeśli operacja `map` zwraca kolekcję, która nie zawiera żadnych modeli Eloquent, zostanie ona przekonwertowana na bazową kolekcję.

<a name="available-."></a>
## Dostępne Metody

Wszystkie kolekcje Eloquent rozszerzają bazowy obiekt [kolekcji Laravel](/docs/{{version}}/collections#available-methods); w związku z tym dziedziczą wszystkie potężne metody dostarczone przez bazową kolekcję.

Ponadto klasa `Illuminate\Database\Eloquent\Collection` zapewnia nadzbiór metod pomagających w zarządzaniu kolekcjami modeli. Większość metod zwraca instancje `Illuminate\Database\Eloquent\Collection`; jednak niektóre metody, takie jak `modelKeys`, zwracają `Illuminate\Support\Collection`.

<style>
    .collection-method-list > p {
        columns: 14.4em 1; -moz-columns: 14.4em 1; -webkit-columns: 14.4em 1;
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

<div class="collection-method-list" markdown="1">

[append](#method-append)
[contains](#method-contains)
[diff](#method-diff)
[except](#method-except)
[find](#method-find)
[findOrFail](#method-find-or-fail)
[fresh](#method-fresh)
[intersect](#method-intersect)
[load](#method-load)
[loadMissing](#method-loadMissing)
[modelKeys](#method-modelKeys)
[makeVisible](#method-makeVisible)
[makeHidden](#method-makeHidden)
[mergeVisible](#method-mergeVisible)
[mergeHidden](#method-mergeHidden)
[only](#method-only)
[partition](#method-partition)
[setAppends](#method-setAppends)
[setVisible](#method-setVisible)
[setHidden](#method-setHidden)
[toQuery](#method-toquery)
[unique](#method-unique)
[withoutAppends](#method-withoutAppends)

</div>

<a name="method-append"></a>
#### `append($attributes)` {.collection-method .first-collection-method}

Metoda `append` może być użyta do wskazania, że atrybut powinien być [dołączony](/docs/{{version}}/eloquent-serialization#appending-values-to-json) dla każdego modelu w kolekcji. Ta metoda przyjmuje tablicę atrybutów lub pojedynczy atrybut:

```php
$users->append('team');

$users->append(['team', 'is_admin']);
```

<a name="method-contains"></a>
#### `contains($key, $operator = null, $value = null)` {.collection-method}

Metoda `contains` może być użyta do określenia, czy dana instancja modelu znajduje się w kolekcji. Ta metoda przyjmuje klucz główny lub instancję modelu:

```php
$users->contains(1);

$users->contains(User::find(1));
```

<a name="method-diff"></a>
#### `diff($items)` {.collection-method}

Metoda `diff` zwraca wszystkie modele, które nie są obecne w podanej kolekcji:

```php
use App\Models\User;

$users = $users->diff(User::whereIn('id', [1, 2, 3])->get());
```

<a name="method-except"></a>
#### `except($keys)` {.collection-method}

Metoda `except` zwraca wszystkie modele, które nie mają podanych kluczy głównych:

```php
$users = $users->except([1, 2, 3]);
```

<a name="method-find"></a>
#### `find($key)` {.collection-method}

Metoda `find` zwraca model, który ma klucz główny pasujący do podanego klucza. Jeśli `$key` jest instancją modelu, `find` spróbuje zwrócić model pasujący do klucza głównego. Jeśli `$key` jest tablicą kluczy, `find` zwróci wszystkie modele, które mają klucz główny w podanej tablicy:

```php
$users = User::all();

$user = $users->find(1);
```

<a name="method-find-or-fail"></a>
#### `findOrFail($key)` {.collection-method}

Metoda `findOrFail` zwraca model, który ma klucz główny pasujący do podanego klucza lub rzuca wyjątek `Illuminate\Database\Eloquent\ModelNotFoundException`, jeśli nie można znaleźć pasującego modelu w kolekcji:

```php
$users = User::all();

$user = $users->findOrFail(1);
```

<a name="method-fresh"></a>
#### `fresh($with = [])` {.collection-method}

Metoda `fresh` pobiera świeżą instancję każdego modelu w kolekcji z bazy danych. Ponadto wszystkie określone relacje zostaną wczytane z wyprzedzeniem:

```php
$users = $users->fresh();

$users = $users->fresh('comments');
```

<a name="method-intersect"></a>
#### `intersect($items)` {.collection-method}

Metoda `intersect` zwraca wszystkie modele, które są również obecne w podanej kolekcji:

```php
use App\Models\User;

$users = $users->intersect(User::whereIn('id', [1, 2, 3])->get());
```

<a name="method-load"></a>
#### `load($relations)` {.collection-method}

Metoda `load` wczytuje z wyprzedzeniem podane relacje dla wszystkich modeli w kolekcji:

```php
$users->load(['comments', 'posts']);

$users->load('comments.author');

$users->load(['comments', 'posts' => fn ($query) => $query->where('active', 1)]);
```

<a name="method-loadMissing"></a>
#### `loadMissing($relations)` {.collection-method}

Metoda `loadMissing` wczytuje z wyprzedzeniem podane relacje dla wszystkich modeli w kolekcji, jeśli relacje nie są jeszcze załadowane:

```php
$users->loadMissing(['comments', 'posts']);

$users->loadMissing('comments.author');

$users->loadMissing(['comments', 'posts' => fn ($query) => $query->where('active', 1)]);
```

<a name="method-modelKeys"></a>
#### `modelKeys()` {.collection-method}

Metoda `modelKeys` zwraca klucze główne dla wszystkich modeli w kolekcji:

```php
$users->modelKeys();

// [1, 2, 3, 4, 5]
```

<a name="method-makeVisible"></a>
#### `makeVisible($attributes)` {.collection-method}

Metoda `makeVisible` [czyni atrybuty widocznymi](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json), które są zazwyczaj "ukryte" w każdym modelu w kolekcji:

```php
$users = $users->makeVisible(['address', 'phone_number']);
```

<a name="method-makeHidden"></a>
#### `makeHidden($attributes)` {.collection-method}

Metoda `makeHidden` [ukrywa atrybuty](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json), które są zazwyczaj "widoczne" w każdym modelu w kolekcji:

```php
$users = $users->makeHidden(['address', 'phone_number']);
```

<a name="method-mergeVisible"></a>
#### `mergeVisible($attributes)` {.collection-method}

Metoda `mergeVisible` [czyni dodatkowe atrybuty widocznymi](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json), zachowując istniejące widoczne atrybuty:

```php
$users = $users->mergeVisible(['middle_name']);
```

<a name="method-mergeHidden"></a>
#### `mergeHidden($attributes)` {.collection-method}

Metoda `mergeHidden` [ukrywa dodatkowe atrybuty](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json), zachowując istniejące ukryte atrybuty:

```php
$users = $users->mergeHidden(['last_login_at']);
```

<a name="method-only"></a>
#### `only($keys)` {.collection-method}

Metoda `only` zwraca wszystkie modele, które mają podane klucze główne:

```php
$users = $users->only([1, 2, 3]);
```

<a name="method-partition"></a>
#### `partition` {.collection-method}

Metoda `partition` zwraca instancję `Illuminate\Support\Collection` zawierającą instancje kolekcji `Illuminate\Database\Eloquent\Collection`:

```php
$partition = $users->partition(fn ($user) => $user->age > 18);

dump($partition::class);    // Illuminate\Support\Collection
dump($partition[0]::class); // Illuminate\Database\Eloquent\Collection
dump($partition[1]::class); // Illuminate\Database\Eloquent\Collection
```

<a name="method-setAppends"></a>
#### `setAppends($attributes)` {.collection-method}

Metoda `setAppends` tymczasowo nadpisuje wszystkie [dołączone atrybuty](/docs/{{version}}/eloquent-serialization#appending-values-to-json) w każdym modelu w kolekcji:

```php
$users = $users->setAppends(['is_admin']);
```

<a name="method-setVisible"></a>
#### `setVisible($attributes)` {.collection-method}

Metoda `setVisible` [tymczasowo nadpisuje](/docs/{{version}}/eloquent-serialization#temporarily-modifying-attribute-visibility) wszystkie widoczne atrybuty w każdym modelu w kolekcji:

```php
$users = $users->setVisible(['id', 'name']);
```

<a name="method-setHidden"></a>
#### `setHidden($attributes)` {.collection-method}

Metoda `setHidden` [tymczasowo nadpisuje](/docs/{{version}}/eloquent-serialization#temporarily-modifying-attribute-visibility) wszystkie ukryte atrybuty w każdym modelu w kolekcji:

```php
$users = $users->setHidden(['email', 'password', 'remember_token']);
```

<a name="method-toquery"></a>
#### `toQuery()` {.collection-method}

Metoda `toQuery` zwraca instancję kreatora zapytań Eloquent zawierającą ograniczenie `whereIn` na klucze główne modeli kolekcji:

```php
use App\Models\User;

$users = User::where('status', 'VIP')->get();

$users->toQuery()->update([
    'status' => 'Administrator',
]);
```

<a name="method-unique"></a>
#### `unique($key = null, $strict = false)` {.collection-method}

Metoda `unique` zwraca wszystkie unikalne modele w kolekcji. Wszystkie modele z tym samym kluczem głównym co inny model w kolekcji są usuwane:

```php
$users = $users->unique();
```

<a name="method-withoutAppends"></a>
#### `withoutAppends($attributes)` {.collection-method}

Metoda `withoutAppends` tymczasowo usuwa wszystkie [dołączone atrybuty](/docs/{{version}}/eloquent-serialization#appending-values-to-json) z każdego modelu w kolekcji:

```php
$users = $users->withoutAppends();
```

<a name="custom-collections"></a>
## Custom Kolekcje

Jeśli chcesz użyć niestandardowego obiektu `Collection` podczas interakcji z danym modelem, możesz dodać atrybut `CollectedBy` do swojego modelu:

```php
<?php

namespace App\Models;

use App\Support\UserCollection;
use Illuminate\Database\Eloquent\Attributes\CollectedBy;
use Illuminate\Database\Eloquent\Model;

#[CollectedBy(UserCollection::class)]
class User extends Model
{
    // ...
}
```

Alternatywnie możesz zdefiniować metodę `newCollection` w swoim modelu:

```php
<?php

namespace App\Models;

use App\Support\UserCollection;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Create a new Eloquent Collection instance.
     *
     * @param  array<int, \Illuminate\Database\Eloquent\Model>  $models
     * @return \Illuminate\Database\Eloquent\Collection<int, \Illuminate\Database\Eloquent\Model>
     */
    public function newCollection(array $models = []): Collection
    {
        $collection = new UserCollection($models);

        if (Model::isAutomaticallyEagerLoadingRelationships()) {
            $collection->withRelationshipAutoloading();
        }

        return $collection;
    }
}
```

Po zdefiniowaniu metody `newCollection` lub dodaniu atrybutu `CollectedBy` do swojego modelu, otrzymasz instancję swojej niestandardowej kolekcji za każdym razem, gdy Eloquent normalnie zwróciłby instancję `Illuminate\Database\Eloquent\Collection`.

Jeśli chcesz użyć niestandardowej kolekcji dla każdego modelu w swojej aplikacji, powinieneś zdefiniować metodę `newCollection` w bazowej klasie modelu, która jest rozszerzana przez wszystkie modele Twojej aplikacji.
