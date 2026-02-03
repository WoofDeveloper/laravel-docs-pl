# Eloquent: Mutatory i Rzutowanie

- [Wprowadzenie](#Wprowadzenie)
- [Akcesory i mutatory](#accessors-and-mutators)
    - [Definiowanie akcessora](#definiowanie-an-accessor)
    - [Definiowanie mutatora](#definiowanie-a-mutator)
- [Rzutowanie atrybutów](#atrybut-casting)
    - [Rzutowanie tablic i JSON](#tablica-and-json-casting)
    - [Rzutowanie binarne](#binary-casting)
    - [Rzutowanie dat](#date-casting)
    - [Rzutowanie Enum](#enum-casting)
    - [Rzutowanie zaszyfrowane](#encrypted-casting)
    - [Rzutowanie w czasie zapytania](#zapytanie-time-casting)
- [Niestandardowe rzutowania](#custom-rzutowania)
    - [Rzutowanie obiektów wartości](#wartość-obiekt-casting)
    - [Serializacja tablicy / JSON](#tablica-json-serialization)
    - [Rzutowanie przychodzące](#inbound-casting)
    - [Parametry rzutowania](#rzutowanie-parameters)
    - [Porównywanie wartości rzutowanych](#comparing-rzutowanie-values)
    - [Castables](#Castables)

<a name="Wprowadzenie"></a>
## Wprowadzenie

Akcesory, mutatory i rzutowanie atrybutów pozwalają na transformację wartości atrybutów Eloquent podczas ich pobierania lub ustawiania w instancjach modelu. Na przykład możesz chcieć użyć [enkryptera Laravel](/docs/{{version}}/encryption) do zaszyfrowania wartości podczas zapisywania jej w bazie danych, a następnie automatycznie odszyfrować atrybut podczas dostępu do niego w modelu Eloquent. Lub możesz chcieć przekonwertować ciąg JSON przechowywany w bazie danych na tablicę, gdy jest dostępny przez model Eloquent.

<a name="accessors-and-mutators"></a>
## Akcesory i mutatory

<a name="definiowanie-an-accessor"></a>
### Definiowanie akcessora

Akcesor transformuje wartość atrybutu Eloquent podczas dostępu do niego. Aby zdefiniować akcesor, utwórz chronioną metodę w swoim modelu reprezentującą dostępny atrybut. Nazwa tej metody powinna odpowiadać reprezentacji "camel case" rzeczywistego atrybutu modelu / kolumny bazy danych, gdy ma to zastosowanie.

W tym przykładzie zdefiniujemy akcesor dla atrybutu `first_name`. Akcesor zostanie automatycznie wywołany przez Eloquent podczas próby pobrania wartości atrybutu `first_name`. Wszystkie metody akcesorów / mutatorów atrybutów muszą deklarować typ zwracany `Illuminate\Database\Eloquent\Casts\Attribute`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Pobierz imię użytkownika.
     */
    protected function firstName(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => ucfirst($value),
        );
    }
}
```

Wszystkie metody akcesorów zwracają instancję `Attribute`, która definiuje, jak atrybut będzie dostępny i, opcjonalnie, mutowany. W tym przykładzie definiujemy tylko sposób dostępu do atrybutu. Aby to zrobić, przekazujemy argument `get` do konstruktora klasy `Attribute`.

Jak widać, oryginalna wartość kolumny jest przekazywana do akcessora, umożliwiając manipulację i zwrócenie wartości. Aby uzyskać dostęp do wartości akcessora, możesz po prostu uzyskać dostęp do atrybutu `first_name` w instancji modelu:

```php
use App\Models\User;

$user = User::find(1);

$firstName = $user->first_name;
```

> [!NOTE]
> Jeśli chcesz, aby te obliczone wartości były dodawane do reprezentacji tablicy / JSON twojego modelu, [musisz je dołączyć](/docs/{{version}}/eloquent-serialization#appending-values-to-json).

<a name="building-value-objects-from-multiple-attributes"></a>
#### Budowanie obiektów wartości z wielu atrybutów

Czasami akcesor może potrzebować przekształcić wiele atrybutów modelu w pojedynczy "obiekt wartości". Aby to zrobić, domknięcie `get` może przyjąć drugi argument `$attributes`, który zostanie automatycznie przekazany do domknięcia i będzie zawierał tablicę wszystkich bieżących atrybutów modelu:

```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * Interakcja z adresem użytkownika.
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    );
}
```

<a name="accessor-caching"></a>
#### Buforowanie akcesorów

Podczas zwracania obiektów wartości z akcesorów, wszelkie zmiany wprowadzone do obiektu wartości będą automatycznie synchronizowane z powrotem do modelu przed zapisaniem modelu. Jest to możliwe, ponieważ Eloquent zachowuje instancje zwracane przez akcesory, dzięki czemu może zwrócić tę samą instancję przy każdym wywołaniu akcessora:

```php
use App\Models\User;

$user = User::find(1);

$user->address->lineOne = 'Zaktualizowana wartość wiersza adresu 1';
$user->address->lineTwo = 'Zaktualizowana wartość wiersza adresu 2';

$user->save();
```

Jednak czasami możesz chcieć włączyć buforowanie dla wartości prymitywnych, takich jak ciągi znaków i wartości logiczne, szczególnie jeśli są one wymagające obliczeniowo. Aby to osiągnąć, możesz wywołać metodę `shouldCache` podczas definiowania akcessora:

```php
protected function hash(): Attribute
{
    return Attribute::make(
        get: fn (string $value) => bcrypt(gzuncompress($value)),
    )->shouldCache();
}
```

Jeśli chcesz wyłączyć buforowanie obiektów dla atrybutów, możesz wywołać metodę `withoutObjectCaching` podczas definiowania atrybutu:

```php
/**
 * Interakcja z adresem użytkownika.
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    )->withoutObjectCaching();
}
```

<a name="definiowanie-a-mutator"></a>
### Definiowanie mutatora

Mutator przekształca wartość atrybutu Eloquent podczas jego ustawiania. Aby zdefiniować mutator, możesz podać argument `set` podczas definiowania atrybutu. Zdefiniujmy mutator dla atrybutu `first_name`. Ten mutator zostanie automatycznie wywołany, gdy będziemy próbować ustawić wartość atrybutu `first_name` w modelu:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Interakcja z imieniem użytkownika.
     */
    protected function firstName(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => ucfirst($value),
            set: fn (string $value) => strtolower($value),
        );
    }
}
```

Domknięcie mutatora otrzyma wartość, która jest ustawiana na atrybut, umożliwiając manipulację wartością i zwrócenie zmanipulowanej wartości. Aby użyć naszego mutatora, wystarczy ustawić atrybut `first_name` w modelu Eloquent:

```php
use App\Models\User;

$user = User::find(1);

$user->first_name = 'Sally';
```

W tym przykładzie callback `set` zostanie wywołany z wartością `Sally`. Mutator zastosuje funkcję `strtolower` do nazwy i ustawi wynikową wartość w wewnętrznej tablicy `$attributes` modelu.

<a name="mutating-multiple-attributes"></a>
#### Mutowanie wielu atrybutów

Czasami mutator może potrzebować ustawić wiele atrybutów w bazowym modelu. Aby to zrobić, możesz zwrócić tablicę z domknięcia `set`. Każdy klucz w tablicy powinien odpowiadać bazowemu atrybutowi / kolumnie bazy danych związanej z modelem:

```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * Interakcja z adresem użytkownika.
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
        set: fn (Address $value) => [
            'address_line_one' => $value->lineOne,
            'address_line_two' => $value->lineTwo,
        ],
    );
}
```

<a name="atrybut-casting"></a>
## Rzutowanie atrybutów

Rzutowanie atrybutów zapewnia funkcjonalność podobną do akcesorów i mutatorów bez konieczności definiowania dodatkowych metod w modelu. Zamiast tego metoda `casts` modelu zapewnia wygodną metodę konwersji atrybutów na popularne typy danych.

Metoda `casts` powinna zwrócić tablicę, w której kluczem jest nazwa rzutowanego atrybutu, a wartością jest typ, na który chcesz rzutować kolumnę. Obsługiwane typy rzutowania to:

<div class="content-list" markdown="1">

- `array`
- `AsFluent::class`
- `AsStringable::class`
- `AsUri::class`
- `boolean`
- `collection`
- `date`
- `datetime`
- `immutable_date`
- `immutable_datetime`
- <code>decimal:&lt;precision&gt;</code>
- `double`
- `encrypted`
- `encrypted:array`
- `encrypted:collection`
- `encrypted:object`
- `float`
- `hashed`
- `integer`
- `object`
- `real`
- `string`
- `timestamp`

</div>

Aby zademonstruować rzutowanie atrybutów, zrzutujmy atrybut `is_admin`, który jest przechowywany w naszej bazie danych jako liczba całkowita (`0` lub `1`) na wartość logiczną:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Pobierz atrybuty, które powinny być rzutowane.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'is_admin' => 'boolean',
        ];
    }
}
```

Po zdefiniowaniu rzutowania atrybut `is_admin` będzie zawsze rzutowany na wartość logiczną podczas uzyskiwania do niego dostępu, nawet jeśli bazowa wartość jest przechowywana w bazie danych jako liczba całkowita:

```php
$user = App\Models\User::find(1);

if ($user->is_admin) {
    // ...
}
```

Jeśli musisz dodać nowe, tymczasowe rzutowanie w czasie wykonywania, możesz użyć metody `mergeCasts`. Te definicje rzutowania zostaną dodane do wszystkich rzutowań już zdefiniowanych w modelu:

```php
$user->mergeCasts([
    'is_admin' => 'integer',
    'options' => 'obiekt',
]);
```

> [!WARNING]
> Atrybuty, które są `null`, nie zostaną rzutowane. Ponadto nigdy nie powinieneś definiować rzutowania (lub atrybutu), które ma taką samą nazwę jak relacja, ani przypisywać rzutowania do klucza głównego modelu.

<a name="stringable-casting"></a>
#### Rzutowanie Stringable

Możesz użyć klasy rzutowania `Illuminate\Database\Eloquent\Casts\AsStringable`, aby rzutować atrybut modelu na [obiekt Illuminate\Support\Stringable płynny](/docs/{{version}}/strings#fluent-strings-metoda-list):

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\AsStringable;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Pobierz atrybuty, które powinny być rzutowane.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'directory' => AsStringable::class,
        ];
    }
}
```

<a name="tablica-and-json-casting"></a>
### Rzutowanie tablic i JSON

Rzutowanie `array` jest szczególnie przydatne podczas pracy z kolumnami przechowywanymi jako serializowany JSON. Na przykład, jeśli Twoja baza danych ma pole typu `JSON` lub `TEXT`, które zawiera serializowany JSON, dodanie rzutowania `array` do tego atrybutu automatycznie zdeserializuje atrybut do tablicy PHP, gdy uzyskasz do niego dostęp w swoim modelu Eloquent:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Pobierz atrybuty, które powinny być rzutowane.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'options' => 'array',
        ];
    }
}
```

Po zdefiniowaniu rzutowania możesz uzyskać dostęp do atrybutu `options` i zostanie on automatycznie zdeserializowany z JSON do tablicy PHP. Kiedy ustawisz wartość atrybutu `options`, podana tablica zostanie automatycznie z powrotem serializowana do JSON w celu przechowywania:

```php
use App\Models\User;

$user = User::find(1);

$options = $user->options;

$options['key'] = 'value';

$user->options = $options;

$user->save();
```

Aby zaktualizować pojedyncze pole atrybutu JSON za pomocą bardziej zwięzłej składni, możesz [ustawić atrybut jako mass assignable](/docs/{{version}}/eloquent#mass-assignment-json-kolumny) i użyć operatora `->` podczas wywoływania metody `update`:

```php
$user = User::find(1);

$user->update(['options->key' => 'wartość']);
```

<a name="json-and-unicode"></a>
#### JSON i Unicode

Jeśli chcesz przechowywać atrybut tablicy jako JSON z niezescapowanymi znakami Unicode, możesz użyć rzutowania `json:unicode`:

```php
/**
 * Pobierz atrybuty, które powinny być rzutowane.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => 'json:unicode',
    ];
}
```

<a name="tablica-obiekt-and-kolekcja-casting"></a>
#### Rzutowanie obiektu tablicy i kolekcji

Chociaż standardowe rzutowanie `array` jest wystarczające dla wielu aplikacji, ma pewne wady. Ponieważ rzutowanie `array` zwraca typ prymitywny, nie jest możliwe bezpośrednie mutowanie offsetu tablicy. Na przykład następujący kod wywoła błąd PHP:

```php
$user = User::find(1);

$user->options['key'] = $value;
```

Aby to rozwiązać, Laravel oferuje rzutowanie `AsArrayObject`, które rzutuje atrybut JSON na klasę [ArrayObject](https://www.php.net/manual/en/class.arrayobject.php). Ta funkcja jest zaimplementowana przy użyciu implementacji [niestandardowego rzutowania](#custom-rzutowania) Laravel, która pozwala Laravel inteligentnie buforować i przekształcać zmutowany obiekt w taki sposób, aby można było modyfikować poszczególne offsety bez wywoływania błędu PHP. Aby użyć rzutowania `AsArrayObject`, po prostu przypisz je do atrybutu:

```php
use Illuminate\Database\Eloquent\Casts\AsArrayObject;

/**
 * Pobierz atrybuty, które powinny być rzutowane.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsArrayObject::class,
    ];
}
```

Podobnie Laravel oferuje rzutowanie `AsCollection`, które rzutuje atrybut JSON na instancję Laravel [kolekcji](/docs/{{version}}/collections):

```php
use Illuminate\Database\Eloquent\Casts\AsCollection;

/**
 * Pobierz atrybuty, które powinny być rzutowane.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsCollection::class,
    ];
}
```

Jeśli chcesz, aby rzutowanie `AsCollection` tworzyło niestandardową klasę kolekcji zamiast bazowej klasy kolekcji Laravel, możesz podać nazwę klasy kolekcji jako argument rzutowania:

```php
use App\Collections\OptionCollection;
use Illuminate\Database\Eloquent\Casts\AsCollection;

/**
 * Pobierz atrybuty, które powinny być rzutowane.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsCollection::using(OptionCollection::class),
    ];
}
```

Metoda `of` może być używana do wskazania, że elementy kolekcji powinny być mapowane do danej klasy za pośrednictwem [metody mapInto kolekcji](/docs/{{version}}/collections#metoda-mapinto):

```php
use App\ValueObjects\Option;
use Illuminate\Database\Eloquent\Casts\AsCollection;

/**
 * Pobierz atrybuty, które powinny być rzutowane.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsCollection::of(Option::class)
    ];
}
```

Podczas mapowania kolekcji do obiektów, obiekt powinien implementować interfejsy `Illuminate\Contracts\Support\Arrayable` i `JsonSerializable`, aby określić, jak ich instancje powinny być serializowane do bazy danych jako JSON:

```php
<?php

namespace App\ValueObjects;

use Illuminate\Contracts\Support\Arrayable;
use JsonSerializable;

class Option implements Arrayable, JsonSerializable
{
    public string $name;
    public mixed $value;
    public bool $isLocked;

    /**
     * Utwórz nową instancję Option.
     */
    public function __construct(array $data)
    {
        $this->name = $data['name'];
        $this->value = $data['value'];
        $this->isLocked = $data['is_locked'];
    }

    /**
     * Pobierz instancję jako tablicę.
     *
     * @return array{name: string, data: string, is_locked: bool}
     */
    public function toArray(): array
    {
        return [
            'name' => $this->name,
            'value' => $this->value,
            'is_locked' => $this->isLocked,
        ];
    }

    /**
     * Określ dane, które powinny być serializowane do JSON.
     *
     * @return array{name: string, data: string, is_locked: bool}
     */
    public function jsonSerialize(): array
    {
        return $this->toArray();
    }
}
```

<a name="binary-casting"></a>
### Rzutowanie binarne

Jeśli twój model Eloquent ma kolumnę `uuid` lub `ulid` [typu binarnego](/docs/{{version}}/migrations#kolumna-metoda-binary) oprócz kolumny ID modelu z autoinkrementacją, możesz użyć rzutowania `AsBinary`, aby automatycznie rzutować wartość do i z jej reprezentacji binarnej:

```php
use Illuminate\Database\Eloquent\Casts\AsBinary;

/**
 * Pobierz atrybuty, które powinny być rzutowane.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'uuid' => AsBinary::uuid(),
        'ulid' => AsBinary::ulid(),
    ];
}
```

Po zdefiniowaniu rzutowania w modelu możesz ustawić wartość atrybutu UUID / ULID na instancję obiektu lub ciąg znaków. Eloquent automatycznie rzutuje wartość na jej reprezentację binarną. Podczas pobierania wartości atrybutu zawsze otrzymasz wartość w postaci zwykłego tekstu:

```php
use Illuminate\Support\Str;

$user->uuid = Str::uuid();

return $user->uuid;

// "6e8cdeed-2f32-40bd-b109-1e4405be2140"
```

<a name="date-casting"></a>
### Rzutowanie dat

Domyślnie Eloquent rzutuje kolumny `created_at` i `updated_at` na instancje [Carbon](https://github.com/briannesbitt/Carbon), która rozszerza klasę PHP `DateTime` i zapewnia zespół pomocnych metod. Możesz rzutować dodatkowe atrybuty dat, definiując dodatkowe rzutowania dat w metodzie `casts` swojego modelu. Zazwyczaj daty powinny być rzutowane przy użyciu typów rzutowań `datetime` lub `immutable_datetime`.

Podczas definiowania rzutowania `date` lub `datetime` możesz również określić format daty. Ten format będzie używany, gdy [model jest serializowany do tablicy lub JSON](/docs/{{version}}/eloquent-serialization):

```php
/**
 * Pobierz atrybuty, które powinny być rzutowane.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'created_at' => 'datetime:Y-m-d',
    ];
}
```

Gdy kolumna jest rzutowana jako data, możesz ustawić odpowiadającą wartość atrybutu modelu na znacznik czasu UNIX, ciąg daty (`Y-m-d`), ciąg daty i czasu lub instancję `DateTime` / `Carbon`. Wartość daty zostanie poprawnie przekonwertowana i przechowana w bazie danych.

Możesz dostosować domyślny format serializacji dla wszystkich dat twojego modelu, definiując metodę `serializeDate` w swoim modelu. Ta metoda nie wpływa na to, jak twoje daty są formatowane do przechowywania w bazie danych:

```php
/**
 * Przygotuj datę do serializacji tablicy / JSON.
 */
protected function serializeDate(DateTimeInterface $date): string
{
    return $date->format('Y-m-d');
}
```

Aby określić format, który powinien być używany podczas faktycznego przechowywania dat modelu w bazie danych, powinieneś zdefiniować właściwość `$dateFormat` w swoim modelu:

```php
/**
 * Format przechowywania kolumn dat modelu.
 *
 * @var string
 */
protected $dateFormat = 'U';
```

<a name="date-casting-and-timezones"></a>
#### Rzutowanie dat, serializacja i strefy czasowe

Domyślnie, the `date` and `datetime` rzutowania will serializuj dates to a UTC ISO-8601 date string (`YYYY-MM-DDTHH:MM:SS.uuuuuuZ`), regardless of the timezone określony in your application's `timezone` configuration option. You are strongly encouraged to always użyj this serialization format, as well as to przechowaj your application's dates in the UTC timezone by not changing your application's `timezone` configuration option from its default `UTC` wartość. Consistently używając the UTC timezone throughout your application will dostarcz the maximum level of interoperability with other date manipulation libraries written in PHP and JavaScript.

If a custom format is applied to the `date` or `datetime` rzutowanie, such as `datetime:Y-m-d H:i:s`, the inner timezone of the Carbon instancja will be used during date serialization. zazwyczaj, this will be the timezone określony in your application's `timezone` configuration option. Jednak, it's important to Zauważ, że `timestamp` kolumny such as `created_at` and `updated_at` are exempt from this behavior and are always formatted in UTC, regardless of the application's timezone setting.

<a name="enum-casting"></a>
### Rzutowanie Enum

Eloquent umożliwia również rzutowanie wartości atrybutów na [Enumy](https://www.php.net/manual/en/language.enumerations.backed.php) PHP. Aby to osiągnąć, możesz określić atrybut i enum, które chcesz rzutować, w metodzie `casts` swojego modelu:

```php
use App\Enums\ServerStatus;

/**
 * Pobierz atrybuty, które powinny być rzutowane.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'status' => ServerStatus::class,
    ];
}
```

Po zdefiniowaniu rzutowania w swoim modelu, określony atrybut będzie automatycznie rzutowany do i z enum podczas interakcji z tym atrybutem:

```php
if ($server->status == ServerStatus::Provisioned) {
    $server->status = ServerStatus::Ready;

    $server->save();
}
```

<a name="casting-arrays-of-enums"></a>
#### Rzutowanie tablic enumów

Czasami you may need your model to przechowaj an tablica of enum values within a single kolumna. Aby to osiągnąć, you may utilize the `AsEnumArrayObject` or `AsEnumCollection` rzutowania dostarczony by Laravel:

```php
use App\Enums\ServerStatus;
use Illuminate\Database\Eloquent\Casts\AsEnumCollection;

/**
 * Pobierz atrybuty, które powinny być rzutowane.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'statuses' => AsEnumCollection::of(ServerStatus::class),
    ];
}
```

<a name="encrypted-casting"></a>
### Rzutowanie zaszyfrowane

Rzutowanie `encrypted` szyfruje wartość atrybutu modelu przy użyciu wbudowanych funkcji [szyfrowania](/docs/{{version}}/encryption) Laravel. Ponadto rzutowania `encrypted:array`, `encrypted:collection`, `encrypted:object`, `AsEncryptedArrayObject` i `AsEncryptedCollection` działają podobnie jak ich niezaszyfrowane odpowiedniki; jednak, jak można się spodziewać, bazowa wartość jest szyfrowana podczas przechowywania w bazie danych.

Ponieważ końcowa długość zaszyfrowanego tekstu nie jest przewidywalna i jest dłuższa niż jej odpowiednik w postaci zwykłego tekstu, upewnij się, że powiązana kolumna bazy danych jest typu `TEXT` lub większa. Ponadto, ponieważ wartości są szyfrowane w bazie danych, nie będziesz mógł wykonywać zapytań ani wyszukiwać zaszyfrowanych wartości atrybutów.

<a name="key-rotation"></a>
#### Rotacja klucza

Jak możesz wiedzieć, Laravel szyfruje ciągi znaków przy użyciu wartości konfiguracyjnej `key` określonej w pliku konfiguracyjnym `app` Twojej aplikacji. Zazwyczaj ta wartość odpowiada wartości zmiennej środowiskowej `APP_KEY`. Jeśli musisz obrócić klucz szyfrowania swojej aplikacji, możesz [zrobić to w sposób łagodny](/docs/{{version}}/encryption#gracefully-rotating-encryption-keys).

<a name="zapytanie-time-casting"></a>
### Rzutowanie w czasie zapytania

Czasami możesz potrzebować zastosować rzutowania podczas wykonywania zapytania, na przykład podczas wybierania surowej wartości z tabeli. Na przykład rozważ następujące zapytanie:

```php
use App\Models\Post;
use App\Models\User;

$users = User::select([
    'users.*',
    'last_posted_at' => Post::selectRaw('MAX(created_at)')
        ->whereColumn('user_id', 'users.id')
])->get();
```

Atrybut `last_posted_at` w wynikach tego zapytania będzie zwykłym ciągiem znaków. Byłoby wspaniale, gdybyśmy mogli zastosować rzutowanie `datetime` do tego atrybutu podczas wykonywania zapytania. Na szczęście możemy to osiągnąć za pomocą metody `withCasts`:

```php
$users = User::select([
    'users.*',
    'last_posted_at' => Post::selectRaw('MAX(created_at)')
        ->whereColumn('user_id', 'users.id')
])->withCasts([
    'last_posted_at' => 'datetime'
])->get();
```

<a name="custom-rzutowania"></a>
## Niestandardowe rzutowania

Laravel posiada różnorodne wbudowane, pomocne typy rzutowań; jednak czasami możesz potrzebować zdefiniować własne typy rzutowań. Aby utworzyć rzutowanie, wykonaj polecenie Artisan `make:cast`. Nowa klasa rzutowania zostanie umieszczona w katalogu `app/Casts`:

```shell
php artisan make:cast AsJson
```

Wszystkie niestandardowe klasy rzutowania implementują interfejs `CastsAttributes`. Klasy implementujące ten interfejs muszą zdefiniować metodę `get` i `set`. Metoda `get` jest odpowiedzialna za przekształcanie surowej wartości z bazy danych na wartość rzutowaną, podczas gdy metoda `set` powinna przekształcać wartość rzutowaną na surową wartość, która może być przechowywan a w bazie danych. Jako przykład, ponownie zaimplementujemy wbudowany typ rzutowania `json` jako niestandardowy typ rzutowania:

```php
<?php

namespace App\Casts;

use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
use Illuminate\Database\Eloquent\Model;

class AsJson implements CastsAttributes
{
    /**
     * Rzutuj podaną wartość.
     *
     * @param  array<string, mixed>  $attributes
     * @return array<string, mixed>
     */
    public function get(
        Model $model,
        string $key,
        mixed $value,
        array $attributes,
    ): array {
        return json_decode($value, true);
    }

    /**
     * Przygotuj podaną wartość do przechowywania.
     *
     * @param  array<string, mixed>  $attributes
     */
    public function set(
        Model $model,
        string $key,
        mixed $value,
        array $attributes,
    ): string {
        return json_encode($value);
    }
}
```

Po zdefiniowaniu niestandardowego typu rzutowania możesz dołączyć go do atrybutu modelu, używając jego nazwy klasy:

```php
<?php

namespace App\Models;

use App\Casts\AsJson;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Pobierz atrybuty, które powinny być rzutowane.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'options' => AsJson::class,
        ];
    }
}
```

<a name="wartość-obiekt-casting"></a>
### Rzutowanie obiektów wartości

Nie jesteś ograniczony do rzutowania wartości na typy prymitywne. Możesz również rzutować wartości na obiekty. Definiowanie niestandardowych rzutowań, które rzutują wartości na obiekty, jest bardzo podobne do rzutowania na typy prymitywne; jednak jeśli Twój obiekt wartości obejmuje więcej niż jedną kolumnę bazy danych, metoda `set` musi zwrócić tablicę par klucz/wartość, które będą używane do ustawiania surowych, przechowywalnych wartości w modelu. Jeśli Twój obiekt wartości wpływa tylko na jedną kolumnę, powinieneś po prostu zwrócić przechowywalną wartość.

Jako przykład zdefiniujemy niestandardową klasę rzutowania, która rzutuje wiele wartości modelu na pojedynczy obiekt wartości `Address`. Założymy, że obiekt wartości `Address` ma dwie publiczne właściwości: `lineOne` i `lineTwo`:

```php
<?php

namespace App\Casts;

use App\ValueObjects\Address;
use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
use Illuminate\Database\Eloquent\Model;
use InvalidArgumentException;

class AsAddress implements CastsAttributes
{
    /**
     * Rzutuj podaną wartość.
     *
     * @param  array<string, mixed>  $attributes
     */
    public function get(
        Model $model,
        string $key,
        mixed $value,
        array $attributes,
    ): Address {
        return new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two']
        );
    }

    /**
     * Przygotuj podaną wartość do przechowywania.
     *
     * @param  array<string, mixed>  $attributes
     * @return array<string, string>
     */
    public function set(
        Model $model,
        string $key,
        mixed $value,
        array $attributes,
    ): array {
        if (! $value instanceof Address) {
            throw new InvalidArgumentException('Podana wartość nie jest instancją Address.');
        }

        return [
            'address_line_one' => $value->lineOne,
            'address_line_two' => $value->lineTwo,
        ];
    }
}
```

Podczas rzutowania na obiekty wartości, wszelkie zmiany wprowadzone do obiektu wartości będą automatycznie synchronizowane z powrotem do modelu przed zapisaniem modelu:

```php
use App\Models\User;

$user = User::find(1);

$user->address->lineOne = 'Zaktualizowana wartość adresu';

$user->save();
```

> [!NOTE]
> Jeśli planujesz serializować swoje modele Eloquent zawierające obiekty wartości do JSON lub tablic, powinieneś zaimplementować interfejsy `Illuminate\Contracts\Support\Arrayable` i `JsonSerializable` w obiekcie wartości.

<a name="wartość-obiekt-caching"></a>
#### Buforowanie obiektów wartości

Gdy atrybuty rzutowane na obiekty wartości są rozwiązywane, są buforowane przez Eloquent. W związku z tym, ta sama instancja obiektu będzie zwracana, jeśli atrybut zostanie ponownie użyty.

Jeśli chcesz wyłączyć zachowanie buforowania obiektów niestandardowych klas rzutowania, możesz zadeklarować publiczną właściwość `withoutObjectCaching` w swojej niestandardowej klasie rzutowania:

```php
class AsAddress implements CastsAttributes
{
    public bool $withoutObjectCaching = true;

    // ...
}
```

<a name="tablica-json-serialization"></a>
### Serializacja tablicy / JSON

Gdy model Eloquent jest konwertowany na tablicę lub JSON przy użyciu metod `toArray` i `toJson`, Twoje niestandardowe obiekty wartości rzutowania zazwyczaj będą również serializowane, o ile implementują interfejsy `Illuminate\Contracts\Support\Arrayable` i `JsonSerializable`. Jednak podczas używania obiektów wartości dostarczonych przez biblioteki stron trzecich, możesz nie mieć możliwości dodania tych interfejsów do obiektu.

Dlatego możesz określić, że Twoja niestandardowa klasa rzutowania będzie odpowiedzialna za serializację obiektu wartości. Aby to zrobić, Twoja niestandardowa klasa rzutowania powinna implementować interfejs `Illuminate\Contracts\Database\Eloquent\SerializesCastableAttributes`. Ten interfejs stwierdza, że Twoja klasa powinna zawierać metodę `serialize`, która powinna zwrócić zserializowaną formę Twojego obiektu wartości:

```php
/**
 * Pobierz zserializowaną reprezentację wartości.
 *
 * @param  array<string, mixed>  $attributes
 */
public function serialize(
    Model $model,
    string $key,
    mixed $value,
    array $attributes,
): string {
    return (string) $value;
}
```

<a name="inbound-casting"></a>
### Rzutowanie przychodzące

Czasami możesz potrzebować napisać niestandardową klasę rzutowania, która przekształca tylko wartości ustawiane na modelu i nie wykonuje żadnych operacji podczas pobierania atrybutów z modelu.

Niestandardowe rzutowania tylko przychodzące powinny implementować interfejs `CastsInboundAttributes`, który wymaga zdefiniowania tylko metody `set`. Polecenie Artisan `make:cast` może być wywołane z opcją `--inbound`, aby wygenerować klasę rzutowania tylko przychodzącego:

```shell
php artisan make:cast AsHash --inbound
```

Klasyczny przykład rzutowania tylko przychodzącego to rzutowanie "haszujące". Na przykład możemy zdefiniować rzutowanie, które haszuje wartości przychodzące za pomocą danego algorytmu:

```php
<?php

namespace App\Casts;

use Illuminate\Contracts\Database\Eloquent\CastsInboundAttributes;
use Illuminate\Database\Eloquent\Model;

class AsHash implements CastsInboundAttributes
{
    /**
     * Utwórz nową instancję klasy rzutowania.
     */
    public function __construct(
        protected string|null $algorithm = null,
    ) {}

    /**
     * Przygotuj podaną wartość do przechowywania.
     *
     * @param  array<string, mixed>  $attributes
     */
    public function set(
        Model $model,
        string $key,
        mixed $value,
        array $attributes,
    ): string {
        return is_null($this->algorithm)
            ? bcrypt($value)
            : hash($this->algorithm, $value);
    }
}
```

<a name="rzutowanie-parameters"></a>
### Parametry rzutowania

Podczas dołączania niestandardowego rzutowania do modelu, parametry rzutowania mogą być określone przez oddzielenie ich od nazwy klasy za pomocą znaku `:` i rozgraniczenie wielu parametrów przecinkami. Parametry zostaną przekazane do konstruktora klasy rzutowania:

```php
/**
 * Pobierz atrybuty, które powinny być rzutowane.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'secret' => AsHash::class.':sha256',
    ];
}
```

<a name="comparing-rzutowanie-values"></a>
### Porównywanie wartości rzutowanych

Jeśli chcesz zdefiniować, jak dwie podane wartości rzutowane powinny być porównywane w celu ustalenia, czy zostały zmienione, Twoja niestandardowa klasa rzutowania może implementować interfejs `Illuminate\Contracts\Database\Eloquent\ComparesCastableAttributes`. Pozwala to na dokładną kontrolę tego, które wartości Eloquent uważa za zmienione, a tym samym zapisuje do bazy danych podczas aktualizacji modelu.

Ten interfejs stwierdza, że Twoja klasa powinna zawierać metodę `compare`, która powinna zwrócić `true`, jeśli podane wartości są uważane za równe:

```php
/**
 * Określ, czy podane wartości są równe.
 *
 * @param  \Illuminate\Database\Eloquent\Model  $model
 * @param  string  $key
 * @param  mixed  $firstValue
 * @param  mixed  $secondValue
 * @return bool
 */
public function compare(
    Model $model,
    string $key,
    mixed $firstValue,
    mixed $secondValue
): bool {
    return $firstValue === $secondValue;
}
```

<a name="Castables"></a>
### Castables

Możesz chcieć pozwolić obiektom wartości Twojej aplikacji definiować własne niestandardowe klasy rzutowania. Zamiast dołączać niestandardową klasę rzutowania do swojego modelu, możesz alternatywnie dołączyć klasę obiektu wartości, która implementuje interfejs `Illuminate\Contracts\Database\Eloquent\Castable`:

```php
use App\ValueObjects\Address;

protected function casts(): array
{
    return [
        'address' => Address::class,
    ];
}
```

Obiekty implementujące interfejs `Castable` muszą zdefiniować metodę `castUsing`, która zwraca nazwę klasy niestandardowej klasy rzutowania odpowiedzialnej za rzutowanie do i z klasy `Castable`:

```php
<?php

namespace App\ValueObjects;

use Illuminate\Contracts\Database\Eloquent\Castable;
use App\Casts\AsAddress;

class Address implements Castable
{
    /**
     * Pobierz nazwę klasy castera do użycia podczas rzutowania z/do tego celu rzutowania.
     *
     * @param  array<string, mixed>  $arguments
     */
    public static function castUsing(array $arguments): string
    {
        return AsAddress::class;
    }
}
```

Podczas używania klas `Castable` możesz nadal dostarczać argumenty w definicji metody `casts`. Argumenty zostaną przekazane do metody `castUsing`:

```php
use App\ValueObjects\Address;

protected function casts(): array
{
    return [
        'address' => Address::class.':argument',
    ];
}
```

<a name="anonymous-rzutowanie-classes"></a>
#### Castables i anonimowe klasy rzutowania

Połączenie "Castables" z [anonimowymi klasami](https://www.php.net/manual/en/language.oop5.anonymous.php) PHP pozwala zdefiniować obiekt wartości i jego logikę rzutowania jako pojedynczy obiekt rzutowalny. Aby to osiągnąć, zwróć anonimową klasę z metody `castUsing` Twojego obiektu wartości. Anonimowa klasa powinna implementować interfejs `CastsAttributes`:

```php
<?php

namespace App\ValueObjects;

use Illuminate\Contracts\Database\Eloquent\Castable;
use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

class Address implements Castable
{
    // ...

    /**
     * Pobierz klasę castera do użycia podczas rzutowania z/do tego celu rzutowania.
     *
     * @param  array<string, mixed>  $arguments
     */
    public static function castUsing(array $arguments): CastsAttributes
    {
        return new class implements CastsAttributes
        {
            public function get(
                Model $model,
                string $key,
                mixed $value,
                array $attributes,
            ): Address {
                return new Address(
                    $attributes['address_line_one'],
                    $attributes['address_line_two']
                );
            }

            public function set(
                Model $model,
                string $key,
                mixed $value,
                array $attributes,
            ): array {
                return [
                    'address_line_one' => $value->lineOne,
                    'address_line_two' => $value->lineTwo,
                ];
            }
        };
    }
}
```
