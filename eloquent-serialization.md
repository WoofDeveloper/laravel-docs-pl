# Eloquent: Serializacja

- [Wprowadzenie](#introduction)
- [Serializacja Modeli i Kolekcji](#serializing-models-i-collections)
    - [Serializacja do Tablic](#serializing-to-arrays)
    - [Serializacja do JSON](#serializing-to-json)
- [Ukrywanie Atrybutów w JSON](#hiding-attributes-from-json)
- [Dołączanie Wartości do JSON](#appending-values-to-json)
- [Serializacja Dat](#date-serialization)

<a name="introduction"></a>
## Wprowadzenie

Podczas tworzenia API za pomocą Laravel często będziesz musiał konwertować swoje modele i relacje do tablic lub JSON. Eloquent zawiera wygodne metody do wykonywania tych konwersji, a także kontrolowania, które atrybuty są uwzględnione w serializowanej reprezentacji Twoich modeli.

> [!NOTE]
> Aby uzyskać jeszcze bardziej solidny sposób obsługi serializacji JSON modeli Eloquent i kolekcji, sprawdź dokumentację dotyczącą [zasobów API Eloquent](/docs/{{version}}/eloquent-resources).

<a name="serializing-models-i-collections"></a>
## Serializacja Modeli i Kolekcji

<a name="serializing-to-arrays"></a>
### Serializacja do Tablic

Aby przekonwertować model i jego załadowane [relacje](/docs/{{version}}/eloquent-relationships) do tablicy, powinieneś użyć metody `toArray` Ta metoda jest rekurencyjna, więc wszystkie atrybuty i wszystkie relacje (w tym relacje relacji) zostaną przekonwertowane na tablice:

```php
use App\Models\User;

$user = User::with('roles')->first();

return $user->toArray();
```

Metoda `attributesToArray` może być użyta do konwersji atrybutów modelu do tablicy, ale nie jego relacji:

```php
$user = User::first();

return $user->attributesToArray();
```

Możesz również przekonwertować całe [kolekcje](/docs/{{version}}/eloquent-collections) modeli do tablic wywołując metodę `toArray` na instancji kolekcji:

```php
$users = User::all();

return $users->toArray();
```

<a name="serializing-to-json"></a>
### Serializacja do JSON

Aby przekonwertować model do JSON, powinieneś użyć metody `toJson` Podobnie jak `toArray`metoda `toJson` jest rekurencyjna, więc wszystkie atrybuty i relacje zostaną przekonwertowane na JSON. Możesz również określić dowolne opcje kodowania JSON [obsługiwane przez PHP](https://secure.php.net/manual/en/function.json-encode.php):

```php
use App\Models\User;

$user = User::find(1);

return $user->toJson();

return $user->toJson(JSON_PRETTY_PRINT);
```

Alternatywnie, możesz rzutować model lub kolekcję na string, co automatycznie wywoła metodę `toJson` na modelu lub kolekcji:

```php
return (string) User::find(1);
```

Ponieważ modele i kolekcje są konwertowane na JSON podczas rzutowania na string, możesz zwracać obiekty Eloquent bezpośrednio z tras lub kontrolerów Twojej aplikacji. Laravel automatycznie serializuje Twoje modele Eloquent i kolekcje do JSON, gdy są zwracane z tras lub kontrolerów:

```php
Route::get('/users', function () {
    return User::all();
});
```

<a name="relationships"></a>
#### Relacje

Gdy model Eloquent jest konwertowany na JSON, jego załadowane relacje zostaną automatycznie uwzględnione jako atrybuty w obiekcie JSON. Ponadto, chociaż metody relacji Eloquent są definiowane przy użyciu nazw metod "camel case", atrybut JSON relacji będzie w "snake case".

<a name="hiding-attributes-from-json"></a>
## Ukrywanie Atrybutów w JSON

Czasami możesz chcieć ograniczyć atrybuty, takie jak hasła, które są uwzględnione w tablicy lub reprezentacji JSON Twojego modelu. Aby to zrobić, dodaj właściwość `$hidden` do Twojego modelu. Atrybuty, które są wymienione w tablicy właściwości `$hidden` nie będą uwzględnione w serializowanej reprezentacji Twojego modelu:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Atrybuty, które powinny być ukryte dla serializacji.
     *
     * @var array<string>
     */
    protected $hidden = ['password'];
}
```

> [!NOTE]
> Aby ukryć relacje, dodaj nazwę metody relacji do właściwości `$hidden` Twojego modelu Eloquent.

Alternatywnie, możesz użyć właściwości `visible` aby zdefiniować "białą listę" atrybutów, które powinny być uwzględnione w tablicy i reprezentacji JSON Twojego modelu. Wszystkie atrybuty, które nie są obecne w tablicy `$visible` będą ukryte, gdy model jest konwertowany na tablicę lub JSON:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Atrybuty, które powinny być widoczne w tablicach.
     *
     * @var array
     */
    protected $visible = ['first_name', 'last_name'];
}
```

<a name="temporarily-modifying-attribute-visibility"></a>
#### Tymczasowa Modyfikacja Widoczności Atrybutów

Jeśli chcesz uczynić niektóre zazwyczaj ukryte atrybuty widocznymi w danej instancji modelu, możesz użyć metod `makeVisible` lub `mergeVisible` Metoda `makeVisible` zwraca instancję modelu:

```php
return $user->makeVisible('attribute')->toArray();

return $user->mergeVisible(['name', 'email'])->toArray();
```

Podobnie, jeśli chcesz ukryć niektóre atrybuty, które są zazwyczaj widoczne, możesz użyć metod `makeHidden` lub `mergeHidden` 

```php
return $user->makeHidden('attribute')->toArray();

return $user->mergeHidden(['name', 'email'])->toArray();
```

Jeśli chcesz tymczasowo nadpisać wszystkie widoczne lub ukryte atrybuty, możesz użyć odpowiednio metod `setVisible` i `setHidden`:

```php
return $user->setVisible(['id', 'name'])->toArray();

return $user->setHidden(['email', 'password', 'remember_token'])->toArray();
```

<a name="appending-values-to-json"></a>
## Dołączanie Wartości do JSON

Czasami podczas konwertowania modeli do tablic lub JSON możesz chcieć dodać atrybuty, które nie mają odpowiadającej kolumny w bazie danych. Aby to zrobić, najpierw zdefiniuj [akcesor](/docs/{{version}}/eloquent-mutators) dla wartości:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Określ, czy użytkownik jest administratorem.
     */
    protected function isAdmin(): Attribute
    {
        return new Attribute(
            get: fn () => 'yes',
        );
    }
}
```

Jeśli chcesz, aby akcesor był zawsze dołączony do tablicowych i JSON reprezentacji Twojego modelu, możesz dodać nazwę atrybutu do właściwości `appends` Twojego modelu. Zauważ, że nazwy atrybutów są zazwyczaj przywoływane za pomocą ich serializowanej reprezentacji "snake case", nawet jeśli metoda PHP akcesora jest zdefiniowana przy użyciu "camel case":

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Akcesory do dołączenia do tablicowej formy modelu.
     *
     * @var array
     */
    protected $appends = ['is_admin'];
}
```

Po dodaniu atrybutu do listy `appends`, będzie on uwzględniony zarówno w tablicowej, jak i JSON reprezentacji modelu. Atrybuty w tablicy `appends` również będą respektować ustawienia `visible` i `hidden` skonfigurowane w modelu.

<a name="appending-at-run-time"></a>
#### Dołączanie w Czasie Wykonania

W czasie wykonania możesz nakazać instancji modelu dołączenie dodatkowych atrybutów za pomocą metod `append` lub `mergeAppends` Lub możesz użyć metody `setAppends`, aby nadpisać całą tablicę dołączonych właściwości dla danej instancji modelu:

```php
return $user->append('is_admin')->toArray();

return $user->mergeAppends(['is_admin', 'status'])->toArray();

return $user->setAppends(['is_admin'])->toArray();
```

<a name="date-serialization"></a>
## Serializacja Dat

<a name="customizing-the-default-date-format"></a>
#### Dostosowywanie Domyślnego Formatu Daty

Możesz dostosować domyślny format serializacji, nadpisując metodę `serializeDate`. Ta metoda nie wpływa na to, jak Twoje daty są formatowane do przechowywania w bazie danych:

```php
/**
 * Przygotuj datę do serializacji tablicy / JSON.
 */
protected function serializeDate(DateTimeInterface $date): string
{
    return $date->format('Y-m-d');
}
```

<a name="customizing-the-date-format-per-attribute"></a>
#### Dostosowywanie Formatu Daty dla Każdego Atrybutu

Możesz dostosować format serializacji poszczególnych atrybutów dat Eloquent, określając format daty w [deklaracjach rzutowania](/docs/{{version}}/eloquent-mutators#attribute-casting) modelu:

```php
protected function casts(): array
{
    return [
        'birthday' => 'date:Y-m-d',
        'joined_at' => 'datetime:Y-m-d H:00',
    ];
}
```
