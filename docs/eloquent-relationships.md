# Eloquent: Relacje

- [Wprowadzenie](#Wprowadzenie)
- [Definiowanie relacji](#definiowanie-relacje)
    - [Jeden do jednego / Has One](#one-to-one)
    - [Jeden do wielu / Has Many](#one-to-many)
    - [Jeden do wielu (odwrotna) / Belongs To](#one-to-many-inverse)
    - [Ma jedną z wielu](#has-one-of-many)
    - [Ma jedną przez](#has-one-through)
    - [Ma wiele przez](#has-many-through)
- [Relacje z zakresem](#scoped-relacje)
- [Relacje wiele do wielu](#many-to-many)
    - [Pobieranie kolumn tabeli pośredniej](#pobieranie-intermediate-tabela-kolumny)
    - [Filtrowanie zapytań przez kolumny tabeli pośredniej](#filtering-zapytania-via-intermediate-tabela-kolumny)
    - [Sortowanie zapytań przez kolumny tabeli pośredniej](#ordering-zapytania-via-intermediate-tabela-kolumny)
    - [Definiowanie niestandardowych modeli tabeli pośredniej](#definiowanie-custom-intermediate-tabela-modele)
- [Relacje polimorficzne](#polymorphic-relacje)
    - [Jeden do jednego](#one-to-one-polymorphic-relations)
    - [Jeden do wielu](#one-to-many-polymorphic-relations)
    - [Jeden z wielu](#one-of-many-polymorphic-relations)
    - [Wiele do wielu](#many-to-many-polymorphic-relations)
    - [Niestandardowe typy polimorficzne](#custom-polymorphic-typy)
- [Relacje dynamiczne](#dynamic-relacje)
- [Odpytywanie relacji](#querying-relations)
    - [Metody relacji vs. właściwości dynamiczne](#relacja-metody-vs-dynamic-właściwości)
    - [Odpytywanie o istnienie relacji](#querying-relacja-existence)
    - [Odpytywanie o brak relacji](#querying-relacja-absence)
    - [Odpytywanie relacji Morph To](#querying-morph-to-relacje)
- [Agregowanie powiązanych modeli](#aggregating-related-modele)
    - [Liczenie powiązanych modeli](#counting-related-modele)
    - [Inne funkcje agregujące](#other-aggregate-functions)
    - [Liczenie powiązanych modeli w relacjach Morph To](#counting-related-modele-on-morph-to-relacje)
- [Ładowanie zachłanne](#eager-loading)
    - [Ograniczanie ładowania zachłannego](#constraining-eager-loads)
    - [Leniwe ładowanie zachłanne](#lazy-eager-loading)
    - [Automatyczne ładowanie zachłanne](#automatic-eager-loading)
    - [Zapobieganie leniwemu ładowaniu](#preventing-lazy-loading)
- [Wstawianie i aktualizowanie powiązanych modeli](#inserting-and-updating-related-modele)
    - [Metoda `save`](#the-save-metoda)
    - [Metoda `create`](#the-create-metoda)
    - [Relacje Belongs To](#updating-belongs-to-relacje)
    - [Relacje wiele do wielu](#updating-many-to-many-relacje)
- [Aktualizowanie znaczników czasu rodzica](#touching-parent-timestamps)

<a name="Wprowadzenie"></a>
## Wprowadzenie

Tabele bazy danych są często powiązane ze sobą. Na przykład, wpis na blogu może mieć wiele komentarzy lub zamówienie może być powiązane z użytkownikiem, który je złożył. Eloquent ułatwia zarządzanie i pracę z tymi relacjami, i wspiera różnorodne popularne relacje:

<div class="content-list" markdown="1">

- [Jeden do jednego](#one-to-one)
- [Jeden do wielu](#one-to-many)
- [Wiele do wielu](#many-to-many)
- [Ma jedną przez](#has-one-through)
- [Ma wiele przez](#has-many-through)
- [Jeden do jednego (Polymorphic)](#one-to-one-polymorphic-relations)
- [Jeden do wielu (Polymorphic)](#one-to-many-polymorphic-relations)
- [Wiele do wielu (Polymorphic)](#many-to-many-polymorphic-relations)

</div>

<a name="definiowanie-relacje"></a>
## Definiowanie relacji

Relacje Eloquent są definiowane jako metody w klasach modeli Eloquent. Ponieważ relacje służą również jako potężne [konstruktory zapytań](/docs/{{version}}/queries), definiowanie relacji jako metod zapewnia możliwości łańcuchowania metod i wykonywania zapytań. Na przykład możemy dodać dodatkowe ograniczenia zapytań do tej relacji `posts`:

```php
$user->posts()->where('active', 1)->get();
```

Ale zanim zagłębimy się w używanie relacji, nauczmy się, jak definiować każdy typ relacji obsługiwany przez Eloquent.

<a name="one-to-one"></a>
### Jeden do jednego / Has One

Relacja jeden do jednego jest bardzo podstawowym typem relacji bazodanowej. Na przykład model `User` może być powiązany z jednym modelem `Phone`. Aby zdefiniować tę relację, umieścimy metodę `phone` w modelu `User`. Metoda `phone` powinna wywołać metodę `hasOne` i zwrócić jej wynik. Metoda `hasOne` jest dostępna dla Twojego modelu poprzez klasę bazową `Illuminate\Database\Eloquent\Model`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOne;

class User extends Model
{
    /**
     * Pobierz telefon powiązany z użytkownikiem.
     */
    public function phone(): HasOne
    {
        return $this->hasOne(Phone::class);
    }
}
```

Pierwszy argument przekazany do metody `hasOne` to nazwa klasy powiązanego modelu. Po zdefiniowaniu relacji możemy pobrać powiązany rekord używając właściwości dynamicznych Eloquent. Właściwości dynamiczne pozwalają uzyskiwać dostęp do metod relacji tak, jakby były właściwościami zdefiniowanymi w modelu:

```php
$phone = User::find(1)->phone;
```

Eloquent określa klucz obcy relacji na podstawie nazwy modelu nadrzędnego. W tym przypadku automatycznie zakłada się, że model `Phone` ma klucz obcy `user_id`. Jeśli chcesz nadrzędzić tę konwencję, możesz przekazać drugi argument do metody `hasOne`:

```php
return $this->hasOne(Phone::class, 'foreign_key');
```

Dodatkowo Eloquent zakłada, że klucz obcy powinien mieć wartość odpowiadającą kolumnie klucza głównego rodzica. Innymi słowy, Eloquent będzie szukać wartości kolumny `id` użytkownika w kolumnie `user_id` rekordu `Phone`. Jeśli chcesz, aby relacja używała wartości klucza podstawowego innej niż `id` lub właściwości `$primaryKey` Twojego modelu, możesz przekazać trzeci argument do metody `hasOne`:

```php
return $this->hasOne(Phone::class, 'foreign_key', 'local_key');
```

<a name="one-to-one-definiowanie-the-inverse-of-the-relacja"></a>
#### Definiowanie odwrotnej relacji

Więc możemy uzyskać dostęp do modelu `Phone` z naszego modelu `User`. Następnie zdefiniujmy relację w modelu `Phone`, która pozwoli nam uzyskać dostęp do użytkownika posiadającego telefon. Możemy zdefiniować odwrotną relację do `hasOne` używając metody `belongsTo`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Phone extends Model
{
    /**
     * Pobierz użytkownika, który jest właścicielem telefonu.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

Podczas wywoływania metody `user`, Eloquent spróbuje znaleźć model `User`, który ma `id` pasujące do kolumny `user_id` w modelu `Phone`.

Eloquent określa nazwę klucza obcego badając nazwę metody relacji i dodając sufiks `_id` do nazwy metody. Tak więc w tym przypadku Eloquent zakłada, że model `Phone` ma kolumnę `user_id`. Jednak jeśli klucz obcy w modelu `Phone` nie jest `user_id`, możesz przekazać własną nazwę klucza jako drugi argument metody `belongsTo`:

```php
/**
 * Pobierz użytkownika, który jest właścicielem telefonu.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key');
}
```

Jeśli model nadrzędny nie używa `id` jako klucza głównego lub chcesz znaleźć powiązany model przy użyciu innej kolumny, możesz przekazać trzeci argument do metody `belongsTo`, określając niestandardowy klucz tabeli nadrzędnej:

```php
/**
 * Pobierz użytkownika, który jest właścicielem telefonu.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key', 'owner_key');
}
```

<a name="one-to-many"></a>
### Jeden do wielu / Has Many

Relacja jeden do wielu jest używana do definiowania relacji, w których pojedynczy model jest rodzicem jednego lub wielu modeli podrzędnych. Na przykład wpis na blogu może mieć nieskończoną liczbę komentarzy. Podobnie jak wszystkie inne relacje Eloquent, relacje jeden do wielu są definiowane poprzez zdefiniowanie metody w modelu Eloquent:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    /**
     * Pobierz komentarze dla wpisu na blogu.
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
}
```

Pamiętaj, że Eloquent automatycznie okre\u015bli odpowiedni\u0105 kolumn\u0119 klucza obcego dla modelu `Comment`. Zgodnie z konwencj\u0105, Eloquent weźmie nazw\u0119 modelu nadrz\u0119dnego w notacji "snake case" i doda sufiks `_id`. Tak wi\u0119c w tym przyk\u0142adzie Eloquent za\u0142o\u017cy, \u017ce kolumn\u0105 klucza obcego w modelu `Comment` jest `post_id`.

Po zdefiniowaniu metody relacji mo\u017cemy uzyska\u0107 dost\u0119p do [kolekcji](/docs/{{version}}/eloquent-collections) powi\u0105zanych komentarzy poprzez dost\u0119p do w\u0142a\u015bciwo\u015bci `comments`. Pami\u0119taj, \u017ce poniewa\u017c Eloquent zapewnia "dynamiczne w\u0142a\u015bciwo\u015bci relacji", mo\u017cemy uzyskiwa\u0107 dost\u0119p do metod relacji tak, jakby by\u0142y one zdefiniowane jako w\u0142a\u015bciwo\u015bci modelu:

```php
use App\\Models\\Post;

$comments = Post::find(1)->comments;

foreach ($comments as $comment) {
    // ...
}
```

Poniewa\u017c wszystkie relacje s\u0142u\u017c\u0105 r\u00f3wnie\u017c jako konstruktory zapyta\u0144, mo\u017cesz doda\u0107 dodatkowe ograniczenia do zapytania relacji, wywo\u0142uj\u0105c metod\u0119 `comments` i kontynuuj\u0105c \u0142a\u0144cuchowanie warunk\u00f3w do zapytania:

```php
$comment = Post::find(1)->comments()
    ->where('title', 'foo')
    ->first();
```

Podobnie jak w przypadku metody `hasOne`, mo\u017cesz r\u00f3wnie\u017c nadpisa\u0107 klucze obce i lokalne, przekazuj\u0105c dodatkowe argumenty do metody `hasMany`:

```php
return $this->hasMany(Comment::class, 'foreign_key');

return $this->hasMany(Comment::class, 'foreign_key', 'local_key');
```

<a name="automatically-hydrating-parent-modele-on-children"></a>
#### Automatyczne wypełnianie modeli nadrz\u0119dnych w podrz\u0119dnych

Nawet podczas u\u017cywania \u0142adowania zach\u0142annego Eloquent, problemy zapyta\u0144 "N + 1" mog\u0105 pojawi\u0107 si\u0119, je\u015bli pr\u00f3bujesz uzyskać dostęp do modelu nadrz\u0119dnego z modelu podrz\u0119dnego podczas iteracji przez modele podrz\u0119dne:

```php
$posts = Post::with('comments')->get();

foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->post->title;
    }
}
```

W powyższym przyk\u0142adzie wprowadzono problem zapyta\u0144 "N + 1", poniewa\u017c mimo \u017ce komentarze zosta\u0142y zach\u0142annie za\u0142adowane dla ka\u017cdego modelu `Post`, Eloquent nie wype\u0142nia automatycznie nadrz\u0119dnego `Post` na ka\u017cdym podrz\u0119dnym modelu `Comment`.

Je\u015bli chcesz, aby Eloquent automatycznie wype\u0142nia\u0142 modele nadrz\u0119dne w ich potomkach, mo\u017cesz wywo\u0142a\u0107 metod\u0119 `chaperone` podczas definiowania relacji `hasMany`:

```php
<?php

namespace App\\Models;

use Illuminate\\Database\\Eloquent\\Model;
use Illuminate\\Database\\Eloquent\\Relations\\HasMany;

class Post extends Model
{
    /**
     * Pobierz komentarze dla wpisu na blogu.
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class)->chaperone();
    }
}
```

Lub, je\u015bli chcesz wybra\u0107 automatyczne wype\u0142nianie rodzica w czasie wykonywania, mo\u017cesz wywo\u0142a\u0107 metod\u0119 `chaperone` podczas \u0142adowania zach\u0142annego relacji:

```php
use App\\Models\\Post;

$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

<a name="one-to-many-inverse"></a>
### Jeden do wielu (odwrotna) / Belongs To

Teraz, gdy mo\u017cemy uzyska\u0107 dost\u0119p do wszystkich komentarzy wpisu, zdefiniujmy relacj\u0119, aby komentarz m\u00f3g\u0142 uzyska\u0107 dost\u0119p do swojego wpisu nadrz\u0119dnego. Aby zdefiniowa\u0107 odwrotn\u0105 relacj\u0119 do `hasMany`, zdefiniuj metod\u0119 relacji w modelu podrz\u0119dnym, kt\u00f3ra wywo\u0142uje metod\u0119 `belongsTo`:

```php
<?php

namespace App\\Models;

use Illuminate\\Database\\Eloquent\\Model;
use Illuminate\\Database\\Eloquent\\Relations\\BelongsTo;

class Comment extends Model
{
    /**
     * Pobierz wpis, do kt\u00f3rego nale\u017cy komentarz.
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

Po zdefiniowaniu relacji mo\u017cemy pobra\u0107 wpis nadrz\u0119dny komentarza, uzyskuj\u0105c dost\u0119p do dynamicznej w\u0142a\u015bciwo\u015bci relacji `post`:

```php
use App\\Models\\Comment;

$comment = Comment::find(1);

return $comment->post->title;
```

W powyższym przyk\u0142adzie Eloquent spr\u00f3buje znale\u017a\u0107 model `Post`, kt\u00f3ry ma `id` pasuj\u0105ce do kolumny `post_id` w modelu `Comment`.

Eloquent okre\u015bla domy\u015bln\u0105 nazw\u0119 klucza obcego badaj\u0105c nazw\u0119 metody relacji i dodaj\u0105c do nazwy metody `_`, a nast\u0119pnie nazw\u0119 kolumny klucza g\u0142\u00f3wnego modelu nadrz\u0119dnego. Tak wi\u0119c w tym przyk\u0142adzie Eloquent za\u0142o\u017cy, \u017ce klucz obcy modelu `Post` w tabeli `comments` to `post_id`.

Jednak je\u015bli klucz obcy dla Twojej relacji nie jest zgodny z t\u0105 konwencj\u0105, mo\u017cesz przekaza\u0107 w\u0142asn\u0105 nazw\u0119 klucza obcego jako drugi argument do metody `belongsTo`:

```php
/**
 * Pobierz wpis, do kt\u00f3rego nale\u017cy komentarz.
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key');
}
```

Je\u015bli model nadrz\u0119dny nie u\u017cywa `id` jako klucza podstawowego lub chcesz znale\u017a\u0107 powi\u0105zany model u\u017cywaj\u0105c innej kolumny, mo\u017cesz przekaza\u0107 trzeci argument do metody `belongsTo` okre\u015blaj\u0105cy w\u0142asny klucz tabeli nadrz\u0119dnej:

```php
/**
 * Pobierz wpis, do kt\u00f3rego nale\u017cy komentarz.
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key', 'owner_key');
}
```

<a name="default-modele"></a>
#### Modele domy\u015blne

Relacje `belongsTo`, `hasOne`, `hasOneThrough` i `morphOne` pozwalaj\u0105 zdefiniowa\u0107 model domy\u015blny, kt\u00f3ry zostanie zwr\u00f3cony, je\u015bli dana relacja jest `null`. Ten wzorzec jest cz\u0119sto nazywany [wzorcem obiektu zerowego](https://en.wikipedia.org/wiki/Null_Object_pattern) i mo\u017ce pom\u00f3c usun\u0105\u0107 warunkowe sprawdzenia w kodzie. W poni\u017cszym przyk\u0142adzie relacja `user` zwr\u00f3ci pusty model `App\\Models\\User`, je\u015bli \u017caden u\u017cytkownik nie jest do\u0142\u0105czony do modelu `Post`:

```php
/**
 * Pobierz autora wpisu.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault();
}
```

Aby wype\u0142ni\u0107 model domy\u015blny atrybutami, mo\u017cesz przekaza\u0107 tablic\u0119 lub zamkni\u0119cie do metody `withDefault`:

```php
/**
 * Pobierz autora wpisu.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault([
        'name' => 'Go\u015b\u0107 Autor',
    ]);
}

/**
 * Pobierz autora wpisu.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault(function (User $user, Post $post) {
        $user->name = 'Go\u015b\u0107 Autor';
    });
}
```

<a name="querying-belongs-to-relacje"></a>
#### Odpytywanie relacji Belongs To

Podczas odpytywania potomk\u00f3w relacji "belongs to" mo\u017cesz r\u0119cznie zbudowa\u0107 klauzul\u0119 `where`, aby pobra\u0107 odpowiadaj\u0105ce modele Eloquent:

```php
use App\\Models\\Post;

$posts = Post::where('user_id', $user->id)->get();
```

Jednak mo\u017cesz uzna\u0107 za wygodniejsze u\u017cycie metody `whereBelongsTo`, kt\u00f3ra automatycznie okre\u015bli odpowiedni\u0105 relacj\u0119 i klucz obcy dla danego modelu:

```php
$posts = Post::whereBelongsTo($user)->get();
```

Mo\u017cesz r\u00f3wnie\u017c przekaza\u0107 instancj\u0119 [kolekcji](/docs/{{version}}/eloquent-collections) do metody `whereBelongsTo`. Podczas tego Laravel pobierze modele nale\u017c\u0105ce do któregokolwiek z modeli nadrz\u0119dnych w kolekcji:

```php
$users = User::where('vip', true)->get();

$posts = Post::whereBelongsTo($users)->get();
```

Domy\u015blnie Laravel okre\u015bli relacj\u0119 powi\u0105zan\u0105 z danym modelem na podstawie nazwy klasy modelu; jednak mo\u017cesz okre\u015bli\u0107 nazw\u0119 relacji r\u0119cznie, podaj\u0105c j\u0105 jako drugi argument metody `whereBelongsTo`:

```php
$posts = Post::whereBelongsTo($user, 'author')->get();
```

<a name="has-one-of-many"></a>
### Ma jedną z wielu

Czasami model może mieć wiele powiązanych modeli, ale chcesz łatwo pobrać "najnowszy" lub "najstarszy" powiązany model relacji. Na przykład, model `User` może być powiązany z wieloma modelami `Order`, ale chcesz zdefiniować wygodny sposób interakcji z najnowszym zamówieniem złożonym przez użytkownika. Możesz to osiągnąć używając typu relacji `hasOne` w połączeniu z metodą `ofMany`:

```php
/**
 * Get the user's most recent order.
 */
public function latestOrder(): HasOne
{
    return $this->hasOne(Order::class)->latestOfMany();
}
```

Podobnie możesz zdefiniować metodę do pobrania "najstarszego", czyli pierwszego, powiązanego modelu relacji:

```php
/**
 * Get the user's oldest order.
 */
public function oldestOrder(): HasOne
{
    return $this->hasOne(Order::class)->oldestOfMany();
}
```

Domyślnie metody `latestOfMany` i `oldestOfMany` pobierają najnowszy lub najstarszy powiązany model na podstawie klucza głównego modelu, który musi być sortowalny. Jednak czasami możesz chcieć pobrać pojedynczy model z większej relacji używając innych kryteriów sortowania.

Na przykład, używając metody `ofMany`, możesz pobrać najdroższe zamówienie użytkownika. Metoda `ofMany` przyjmuje jako pierwszy argument kolumnę sortowalną oraz określa funkcję agregującą (`min` lub `max`) do zastosowania podczas odpytywania o powiązany model:

```php
/**
 * Get the user's largest order.
 */
public function largestOrder(): HasOne
{
    return $this->hasOne(Order::class)->ofMany('price', 'max');
}
```

> [!WARNING]
> Ponieważ PostgreSQL nie obsługuje wykonywania funkcji `MAX` na kolumnach UUID, obecnie nie jest możliwe używanie relacji jeden-z-wielu w połączeniu z kolumnami UUID PostgreSQL.

<a name="converting-many-relacje-to-has-one-relacje"></a>
#### Konwersja relacji "Wiele" na relację "Ma jeden"

Często podczas pobierania pojedynczego modelu przy użyciu metod `latestOfMany`, `oldestOfMany` lub `ofMany`, masz już zdefiniowaną relację "ma wiele" dla tego samego modelu. Dla wygody Laravel pozwala łatwo przekonwertować tę relację na relację "ma jeden" poprzez wywołanie metody `one` na relacji:

```php
/**
 * Get the user's orders.
 */
public function orders(): HasMany
{
    return $this->hasMany(Order::class);
}

/**
 * Get the user's largest order.
 */
public function largestOrder(): HasOne
{
    return $this->orders()->one()->ofMany('price', 'max');
}
```

Możesz również użyć metody `one` do konwersji relacji `HasManyThrough` na relacje `HasOneThrough`:

```php
public function latestDeployment(): HasOneThrough
{
    return $this->deployments()->one()->latestOfMany();
}
```

<a name="advanced-has-one-of-many-relacje"></a>
#### Zaawansowane relacje "Ma jedną z wielu"

Możliwe jest tworzenie bardziej zaawansowanych relacji "Ma jedną z wielu". Na przykład model `Product` może mieć wiele powiązanych modeli `Price`, które są zachowywane w systemie nawet po opublikowaniu nowych cen. Dodatkowo nowe dane cenowe produktu mogą być publikowane z wyprzedzeniem, aby wejść w życie w przyszłości za pomocą kolumny `published_at`.

Podsumowując, musimy pobrać najnowszą opublikowaną cenę, gdzie data publikacji nie jest w przyszłości. Dodatkowo, jeśli dwie ceny mają tę samą datę publikacji, preferujemy cenę z wyższym ID. Aby to osiągnąć, musimy przekazać tablicę do metody `ofMany`, która zawiera kolumny sortowalne określające najnowszą cenę. Dodatkowo domknięcie zostanie dostarczone jako drugi argument metody `ofMany`. To domknięcie będzie odpowiedzialne za dodanie dodatkowych ograniczeń daty publikacji do zapytania relacji:

```php
/**
 * Get the current pricing for the product.
 */
public function currentPricing(): HasOne
{
    return $this->hasOne(Price::class)->ofMany([
        'published_at' => 'max',
        'id' => 'max',
    ], function (Builder $query) {
        $query->where('published_at', '<', now());
    });
}
```

<a name="has-one-through"></a>
### Ma jedną przez

Relacja "has-one-through" definiuje relację jeden-do-jednego z innym modelem. Jednak ta relacja wskazuje, że deklarujący model może być dopasowany do jednej instancji innego modelu poprzez przejście _przez_ trzeci model.

Na przykład w aplikacji warsztatu samochodowego, każdy model `Mechanic` może być powiązany z jednym modelem `Car`, a każdy model `Car` może być powiązany z jednym modelem `Owner`. Podczas gdy mechanik i właściciel nie mają bezpośredniej relacji w bazie danych, mechanik może uzyskać dostęp do właściciela _przez_ model `Car`. Przyj rzyjmy się tabelom niezbędnym do zdefiniowania tej relacji:

```text
mechanics
    id - integer
    name - string

cars
    id - integer
    model - string
    mechanic_id - integer

owners
    id - integer
    name - string
    car_id - integer
```

Teraz, gdy zbadaliśmy strukturę tabeli dla relacji, zdefiniujmy relację w modelu `Mechanic`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOneThrough;

class Mechanic extends Model
{
    /**
     * Get the car's owner.
     */
    public function carOwner(): HasOneThrough
    {
        return $this->hasOneThrough(Owner::class, Car::class);
    }
}
```

Pierwszy argument przekazany do metody `hasOneThrough` to nazwa końcowego modelu, do którego chcemy uzyskać dostęp, podczas gdy drugi argument to nazwa modelu pośredniego.

Lub, jeśli odpowiednie relacje zostały już zdefiniowane we wszystkich modelach zaangażowanych w relację, możesz płynnie zdefiniować relację "has-one-through" poprzez wywołanie metody `through` i dostarczenie nazw tych relacji. Na przykład, jeśli model `Mechanic` ma relację `cars`, a model `Car` ma relację `owner`, możesz zdefiniować relację "has-one-through" łączącą mechanika i właściciela w następujący sposób:

```php
// String based syntax...
return $this->through('cars')->has('owner');

// Dynamic syntax...
return $this->throughCars()->hasOwner();
```

<a name="has-one-through-key-conventions"></a>
#### Konwencje kluczy

Typowe konwencje kluczy obcych Eloquent będą używane podczas wykonywania zapytań relacji. Jeśli chcesz dostosować klucze relacji, możesz przekazać je jako trzeci i czwarty argument metody `hasOneThrough`. Trzeci argument to nazwa klucza obcego w modelu pośrednim. Czwarty argument to nazwa klucza obcego w modelu końcowym. Piąty argument to klucz lokalny, podczas gdy szósty argument to klucz lokalny modelu pośredniego:

```php
class Mechanic extends Model
{
    /**
     * Get the car's owner.
     */
    public function carOwner(): HasOneThrough
    {
        return $this->hasOneThrough(
            Owner::class,
            Car::class,
            'mechanic_id', // Foreign key on the cars tabela...
            'car_id', // Foreign key on the owners tabela...
            'id', // Local key on the mechanics tabela...
            'id' // Local key on the cars tabela...
        );
    }
}
```

Lub, jak omówiono wcześniej, jeśli odpowiednie relacje zostały już zdefiniowane we wszystkich modelach zaangażowanych w relację, możesz płynnie zdefiniować relację "has-one-through" poprzez wywołanie metody `through` i dostarczenie nazw tych relacji. To podejście oferuje zaletę ponownego wykorzystania konwencji kluczy już zdefiniowanych w istniejących relacjach:

```php
// String based syntax...
return $this->through('cars')->has('owner');

// Dynamic syntax...
return $this->throughCars()->hasOwner();
```

<a name="has-many-through"></a>
### Ma wiele przez

Relacja "has-many-through" zapewnia wygodny sposób dostępu do odległych relacji poprzez relację pośrednią. Na przykład załóżmy, że budujemy platformę wdrożeniową jak [Laravel Cloud](https://cloud.laravel.com). Model `Application` może uzyskiwać dostęp do wielu modeli `Deployment` przez pośredni model `Environment`. Używając tego przykładu, możesz łatwo zebrać wszystkie wdrożenia dla danej aplikacji. Przyjrzyjmy się tabelom wymaganym do zdefiniowania tej relacji:

```text
applications
    id - integer
    name - string

environments
    id - integer
    application_id - integer
    name - string

deployments
    id - integer
    environment_id - integer
    commit_hash - string
```

Teraz, gdy zbadaliśmy strukturę tabeli dla relacji, zdefiniujmy relację w modelu `Application`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasManyThrough;

class Application extends Model
{
    /**
     * Get all of the deployments for the application.
     */
    public function deployments(): HasManyThrough
    {
        return $this->hasManyThrough(Deployment::class, Environment::class);
    }
}
```

Pierwszy argument przekazany do metody `hasManyThrough` to nazwa końcowego modelu, do którego chcemy uzyskać dostęp, podczas gdy drugi argument to nazwa modelu pośredniego.

Lub, jeśli odpowiednie relacje zostały już zdefiniowane we wszystkich modelach zaangażowanych w relację, możesz płynnie zdefiniować relację "has-many-through" poprzez wywołanie metody `through` i dostarczenie nazw tych relacji. Na przykład, jeśli model `Application` ma relację `environments`, a model `Environment` ma relację `deployments`, możesz zdefiniować relację "has-many-through" łączącą aplikację i wdrożenia w następujący sposób:

```php
// String based syntax...
zwróć $this->through('environments')->has('deployments');

// Dynamic syntax...
zwróć $this->throughEnvironments()->hasDeployments();
```

Chociaż tabela modelu `Deployment` nie zawiera kolumny `application_id`, relacja `hasManyThrough` zapewnia dostęp do wdrożeń aplikacji poprzez `$application->deployments`. Aby pobrać te modele, Eloquent sprawdza kolumnę `application_id` w tabeli pośredniego modelu `Environment`. Po znalezieniu odpowiednich ID środowisk, są one używane do zapytania tabeli modelu `Deployment`.

<a name="has-many-through-key-conventions"></a>
#### Key Conventions

Typowe konwencje kluczy obcych Eloquent będą używane podczas wykonywania zapytań relacji. Jeśli chcesz dostosować klucze relacji, możesz przekazać je jako trzeci i czwarty argument do metody `hasManyThrough`. Trzeci argument to nazwa klucza obcego w modelu pośrednim. Czwarty argument to nazwa klucza obcego w modelu końcowym. Piąty argument to klucz lokalny, a szósty argument to klucz lokalny modelu pośredniego:

```php
class Application extends Model
{
    public function deployments(): HasManyThrough
    {
        return $this->hasManyThrough(
            Deployment::class,
            Environment::class,
            'application_id', // Foreign key on the environments table...
            'environment_id', // Foreign key on the deployments table...
            'id', // Local key on the applications table...
            'id' // Local key on the environments table...
        );
    }
}
```

Lub, jak omówiono wcześniej, jeśli odpowiednie relacje zostały już zdefiniowane na wszystkich modelach zaangażowanych w relację, możesz płynnie zdefiniować relację "has-many-through" wywołując metodę `through` i podając nazwy tych relacji. To podejście oferuje tę zaletę, że wykorzystuje konwencje kluczy już zdefiniowane w istniejących relacjach:

```php
// String based syntax...
return $this->through('environments')->has('deployments');

// Dynamic syntax...
return $this->throughEnvironments()->hasDeployments();
```

<a name="scoped-relacje"></a>
### Relacje z zakresem

Często dodaje się dodatkowe metody do modeli, które ograniczają relacje. Na przykład, możesz dodać metodę `featuredPosts` do modelu `User`, która ogranicza szerszą relację `posts` dodatkowym warunkiem `where`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * Get the user's posts.
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class)->latest();
    }

    /**
     * Get the user's featured posts.
     */
    public function featuredPosts(): HasMany
    {
        return $this->posts()->where('featured', true);
    }
}
```

Jednak, jeśli spróbujesz utworzyć model za pomocą metody `featuredPosts`, jego atrybut `featured` nie zostanie ustawiony na `true`. Jeśli chcesz tworzyć modele poprzez metody relacji i również określić atrybuty, które powinny być dodane do wszystkich modeli tworzonych za pomocą tej relacji, możesz użyć metody `withAttributes` podczas budowania zapytania relacji:

```php
/**
 * Get the user's featured posts.
 */
public function featuredPosts(): HasMany
{
    return $this->posts()->withAttributes(['featured' => true]);
}
```

Metoda `withAttributes` doda warunki `where` do zapytania używając podanych atrybutów, a także doda te atrybuty do wszystkich modeli tworzonych za pomocą metody relacji:

```php
$post = $user->featuredPosts()->create(['title' => 'Featured Post']);

$post->featured; // true
```

Aby poinstruować metodę `withAttributes`, żeby nie dodawała warunków `where` do zapytania, możesz ustawić argument `asConditions` na `false`:

```php
return $this->posts()->withAttributes(['featured' => true], asConditions: false);
```

<a name="many-to-many"></a>
## Relacje wiele do wielu

Relacje wiele-do-wielu są nieco bardziej skomplikowane niż relacje `hasOne` i `hasMany`. Przykładem relacji wiele-do-wielu jest użytkownik, który ma wiele ról, a te role są również współdzielone przez innych użytkowników w aplikacji. Na przykład, użytkownikowi może być przypisana rola "Author" i "Editor"; jednak te role mogą być również przypisane innym użytkownikom. Tak więc użytkownik ma wiele ról, a rola ma wielu użytkowników.

<a name="many-to-many-tabela-structure"></a>
#### tabela Structure

Aby zdefiniować tę relację, potrzebne są trzy tabele bazy danych: `users`, `roles` i `role_user`. Tabela `role_user` pochodzi od alfabetycznej kolejności nazw powiązanych modeli i zawiera kolumny `user_id` i `role_id`. Ta tabela jest używana jako tabela pośrednia łącząca użytkowników i role.

Pamiętaj, ponieważ rola może należeć do wielu użytkowników, nie możemy po prostu umieścić kolumny `user_id` w tabeli `roles`. Oznaczałoby to, że rola może należeć tylko do jednego użytkownika. Aby zapewnić obsługę przypisywania ról do wielu użytkowników, potrzebna jest tabela `role_user`. Możemy podsumować strukturę tabel relacji w następujący sposób:

```text
users
    id - integer
    name - string

roles
    id - integer
    name - string

role_user
    user_id - integer
    role_id - integer
```

<a name="many-to-many-model-structure"></a>
#### model Structure

Relacje wiele-do-wielu są definiowane przez napisanie metody, która zwraca wynik metody `belongsToMany`. Metoda `belongsToMany` jest dostarczana przez klasę bazową `Illuminate\Database\Eloquent\Model`, która jest używana przez wszystkie modele Eloquent Twojej aplikacji. Na przykład, zdefiniujmy metodę `roles` w naszym modelu `User`. Pierwszy argument przekazany do tej metody to nazwa powiązanej klasy modelu:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class User extends Model
{
    /**
     * The roles that belong to the user.
     */
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }
}
```

Po zdefiniowaniu relacji, możesz uzyskać dostęp do ról użytkownika używając dynamicznej właściwości relacji `roles`:

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->roles as $role) {
    // ...
}
```

Ponieważ wszystkie relacje służą również jako konstruktory zapytań, możesz dodać dalsze ograniczenia do zapytania relacji wywołując metodę `roles` i kontynuując łańcuchowe dodawanie warunków do zapytania:

```php
$roles = User::find(1)->roles()->orderBy('name')->get();
```

Aby określić nazwę tabeli pośredniej relacji, Eloquent połączy nazwy dwóch powiązanych modeli w kolejności alfabetycznej. Jednak możesz swobodnie nadpisać tę konwencję. Możesz to zrobić przekazując drugi argument do metody `belongsToMany`:

```php
return $this->belongsToMany(Role::class, 'role_user');
```

Oprócz dostosowania nazwy tabeli pośredniej, możesz również dostosować nazwy kolumn kluczy w tabeli przekazując dodatkowe argumenty do metody `belongsToMany`. Trzeci argument to nazwa klucza obcego modelu, w którym definiujesz relację, a czwarty argument to nazwa klucza obcego modelu, do którego się łączysz:

```php
return $this->belongsToMany(Role::class, 'role_user', 'user_id', 'role_id');
```

<a name="many-to-many-definiowanie-the-inverse-of-the-relacja"></a>
#### Definiowanie odwrotności relacji

Aby zdefiniować "odwrotność" relacji wiele-do-wielu, powinieneś zdefiniować metodę w powiązanym modelu, która również zwraca wynik metody `belongsToMany`. Aby dokończyć nasz przykład użytkownik / rola, zdefiniujmy metodę `users` w modelu `Role`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Role extends Model
{
    /**
     * The users that belong to the role.
     */
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class);
    }
}
```

Jak widać, relacja jest zdefiniowana dokładnie tak samo jak jej odpowiednik w modelu `User`, z wyjątkiem odniesienia do modelu `App\Models\User`. Ponieważ używamy ponownie metody `belongsToMany`, wszystkie zwykłe opcje dostosowania tabeli i kluczy są dostępne podczas definiowania "odwrotności" relacji wiele-do-wielu.

<a name="pobieranie-intermediate-tabela-kolumny"></a>
### Pobieranie kolumn tabeli pośredniej

Jak już się nauczyłeś, praca z relacjami wiele-do-wielu wymaga obecności tabeli pośredniej. Eloquent zapewnia bardzo pomocne sposoby interakcji z tą tabelą. Na przykład, załóżmy, że nasz model `User` ma wiele modeli `Role`, z którymi jest powiązany. Po uzyskaniu dostępu do tej relacji, możemy uzyskać dostęp do tabeli pośredniej używając atrybutu `pivot` na modelach:

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}
```

Zauważ, że każdy model `Role`, który pobieramy, jest automatycznie przypisywany do atrybutu `pivot`. Ten atrybut zawiera model reprezentujący tabelę pośrednią.

Domyślnie tylko klucze modelu będą obecne w modelu `pivot`. Jeśli twoja tabela pośrednia zawiera dodatkowe atrybuty, musisz je określić podczas definiowania relacji:

```php
return $this->belongsToMany(Role::class)->withPivot('active', 'created_by');
```

Jeśli chcesz, aby twoja tabela pośrednia miała znaczniki czasu `created_at` i `updated_at`, które są automatycznie utrzymywane przez Eloquent, wywołaj metodę `withTimestamps` podczas definiowania relacji:

```php
return $this->belongsToMany(Role::class)->withTimestamps();
```

> [!WARNING]
> Tabele pośrednie, które wykorzystują automatycznie utrzymywane znaczniki czasu Eloquent, muszą mieć obie kolumny znaczników czasu `created_at` i `updated_at`.

<a name="customizing-the-pivot-atrybut-name"></a>
#### Customizing the `pivot` atrybut Name

Jak wspomniano wcześniej, atrybuty z tabeli pośredniej mogą być dostępne na modelach poprzez atrybut `pivot`. Jednak możesz swobodnie dostosować nazwę tego atrybutu, aby lepiej odzwierciedlała jego przeznaczenie w Twojej aplikacji.

Na przykład, jeśli Twoja aplikacja zawiera użytkowników, którzy mogą subskrybować podcasty, prawdopodobnie masz relację wiele-do-wielu między użytkownikami a podcastami. W takim przypadku możesz chcieć zmienić nazwę atrybutu tabeli pośredniej na `subscription` zamiast `pivot`. Można to zrobić używając metody `as` podczas definiowania relacji:

```php
return $this->belongsToMany(Podcast::class)
    ->as('subscription')
    ->withTimestamps();
```

Po określeniu niestandardowego atrybutu tabeli pośredniej, możesz uzyskać dostęp do danych tabeli pośredniej używając dostosowanej nazwy:

```php
$users = User::with('podcasts')->get();

foreach ($users->flatMap->podcasts as $podcast) {
    echo $podcast->subscription->created_at;
}
```

<a name="filtering-zapytania-via-intermediate-tabela-kolumny"></a>
### Filtrowanie zapytań przez kolumny tabeli pośredniej

Możesz również filtrować wyniki zwracane przez zapytania relacji `belongsToMany` używając metod `wherePivot`, `wherePivotIn`, `wherePivotNotIn`, `wherePivotBetween`, `wherePivotNotBetween`, `wherePivotNull` i `wherePivotNotNull` podczas definiowania relacji:

```php
return $this->belongsToMany(Role::class)
    ->wherePivot('approved', 1);

return $this->belongsToMany(Role::class)
    ->wherePivotIn('priority', [1, 2]);

return $this->belongsToMany(Role::class)
    ->wherePivotNotIn('priority', [1, 2]);

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotBetween('created_at', ['2020-01-01 00:00:00', '2020-12-31 00:00:00']);

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotNotBetween('created_at', ['2020-01-01 00:00:00', '2020-12-31 00:00:00']);

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotNull('expired_at');

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotNotNull('expired_at');
```

Metoda `wherePivot` dodaje ograniczenie klauzuli where do zapytania, ale nie dodaje określonej wartości podczas tworzenia nowych modeli za pomocą zdefiniowanej relacji. Jeśli potrzebujesz zarówno zapytać, jak i utworzyć relacje z określoną wartością pivot, możesz użyć metody `withPivotValue`:

```php
return $this->belongsToMany(Role::class)
    ->withPivotValue('approved', 1);
```

<a name="ordering-zapytania-via-intermediate-tabela-kolumny"></a>
### Sortowanie zapytań przez kolumny tabeli pośredniej

Możesz sortować wyniki zwracane przez zapytania relacji `belongsToMany` używając metody `orderByPivot`. W poniższym przykładzie pobierzemy wszystkie najnowsze odznaki użytkownika:

```php
return $this->belongsToMany(Badge::class)
    ->where('rank', 'gold')
    ->orderByPivot('created_at', 'desc');
```

<a name="definiowanie-custom-intermediate-tabela-modele"></a>
### Definiowanie niestandardowych modeli tabeli pośredniej

Jeśli chcesz zdefiniować niestandardowy model do reprezentowania tabeli pośredniej Twojej relacji wiele-do-wielu, możesz wywołać metodę `using` podczas definiowania relacji. Niestandardowe modele pivot dają Ci możliwość zdefiniowania dodatkowego zachowania na modelu pivot, takiego jak metody i rzutowania.

Niestandardowe modele pivot wiele-do-wielu powinny rozszerzać klasę `Illuminate\Database\Eloquent\Relations\Pivot`, podczas gdy niestandardowe polimorficzne modele pivot wiele-do-wielu powinny rozszerzać klasę `Illuminate\Database\Eloquent\Relations\MorphPivot`. Na przykład, możemy zdefiniować model `Role`, który używa niestandardowego modelu pivot `RoleUser`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Role extends Model
{
    /**
     * The users that belong to the role.
     */
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class)->using(RoleUser::class);
    }
}
```

Podczas definiowania modelu `RoleUser`, powinieneś rozszerzyć klasę `Illuminate\Database\Eloquent\Relations\Pivot`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Relations\Pivot;

class RoleUser extends Pivot
{
    // ...
}
```

> [!WARNING]
> Modele pivot nie mogą używać traitu `SoftDeletes`. Jeśli potrzebujesz soft delete dla rekordów pivot, rozważ przekonwertowanie swojego modelu pivot na rzeczywisty model Eloquent.

<a name="custom-pivot-modele-and-incrementing-ids"></a>
#### Niestandardowe modele Pivot i autoinkrementujące ID

Jeśli zdefiniowałeś relację wiele-do-wielu, która używa niestandardowego modelu pivot, i ten model pivot ma autoinkrementujący klucz podstawowy, powinieneś upewnić się, że Twoja klasa niestandardowego modelu pivot definiuje właściwość `incrementing` ustawioną na `true`.

```php
/**
 * Indicates if the IDs are auto-incrementing.
 *
 * @var bool
 */
public $incrementing = true;
```

<a name="polymorphic-relacje"></a>
## Relacje polimorficzne

Relacja polimorficzna pozwala modelowi potomnemu należeć do więcej niż jednego typu modelu przy użyciu pojedynczego powiązania. Na przykład, wyobraź sobie, że budujesz aplikację, która pozwala użytkownikom udostępniać posty na blogu i filmy. W takiej aplikacji model `Comment` może należeć zarówno do modelu `Post`, jak i `Video`.

<a name="one-to-one-polymorphic-relations"></a>
### Jeden do jednego (Polymorphic)

<a name="one-to-one-polymorphic-tabela-structure"></a>
#### tabela Structure

Relacja jeden-do-jednego polimorficzna jest podobna do typowej relacji jeden-do-jednego; jednak model potomny może należeć do więcej niż jednego typu modelu przy użyciu pojedynczego powiązania. Na przykład, post na blogu `Post` i `User` mogą współdzielić relację polimorficzną do modelu `Image`. Używając relacji jeden-do-jednego polimorficznej, możesz mieć pojedynczą tabelę unikalnych obrazów, które mogą być powiązane z postami i użytkownikami. Po pierwsze, przeanalizujmy strukturę tabeli:

```text
posts
    id - integer
    name - string

users
    id - integer
    name - string

images
    id - integer
    url - string
    imageable_id - integer
    imageable_type - string
```

Zauważ kolumny `imageable_id` i `imageable_type` w tabeli `images`. Kolumna `imageable_id` będzie zawierać wartość ID posta lub użytkownika, podczas gdy kolumna `imageable_type` będzie zawierać nazwę klasy modelu nadrzędnego. Kolumna `imageable_type` jest używana przez Eloquent do określenia, który "typ" modelu nadrzędnego zwrócić podczas dostępu do relacji `imageable`. W tym przypadku kolumna zawierałaby albo `App\Models\Post` albo `App\Models\User`.

<a name="one-to-one-polymorphic-model-structure"></a>
#### Struktura modelu

Następnie, przeanalizujmy definicje modeli potrzebne do zbudowania tej relacji:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class Image extends Model
{
    /**
     * Get the parent imageable model (user or post).
     */
    public function imageable(): MorphTo
    {
        return $this->morphTo();
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphOne;

class Post extends Model
{
    /**
     * Get the post's image.
     */
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphOne;

class User extends Model
{
    /**
     * Get the user's image.
     */
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}
```

<a name="one-to-one-polymorphic-pobieranie-the-relacja"></a>
#### Pobieranie relacji

Po zdefiniowaniu tabeli bazy danych i modeli, możesz uzyskać dostęp do relacji poprzez swoje modele. Na przykład, aby pobrać obraz dla posta, możemy uzyskać dostęp do dynamicznej właściwości relacji `image`:

```php
use App\Models\Post;

$post = Post::find(1);

$image = $post->image;
```

Możesz pobrać rodzica modelu polimorficznego uzyskując dostęp do nazwy metody, która wykonuje wywołanie `morphTo`. W tym przypadku jest to metoda `imageable` w modelu `Image`. Więc uzyskamy dostęp do tej metody jako dynamicznej właściwości relacji:

```php
use App\Models\Image;

$image = Image::find(1);

$imageable = $image->imageable;
```

Relacja `imageable` w modelu `Image` zwróci albo instancję `Post`, albo `User`, w zależności od tego, który typ modelu posiada obraz.

<a name="morph-one-to-one-key-conventions"></a>
#### Konwencje kluczy

W razie potrzeby, możesz określić nazwę kolumn "id" i "typ" używanych przez Twój polimorficzny model potomny. Jeśli to zrobisz, upewnij się, że zawsze przekazujesz nazwę relacji jako pierwszy argument do metody `morphTo`. Zazwyczaj ta wartość powinna odpowiadać nazwie metody, więc możesz użyć stałej PHP `__FUNCTION__`:

```php
/**
 * Get the model that the image belongs to.
 */
public function imageable(): MorphTo
{
    return $this->morphTo(__FUNCTION__, 'imageable_type', 'imageable_id');
}
```

<a name="one-to-many-polymorphic-relations"></a>
### Jeden do wielu (Polymorphic)

<a name="one-to-many-polymorphic-tabela-structure"></a>
#### tabela Structure

Relacja jeden-do-wielu polimorficzna jest podobna do typowej relacji jeden-do-wielu; jednak model potomny może należeć do więcej niż jednego typu modelu przy użyciu pojedynczego powiązania. Na przykład, wyobraź sobie, że użytkownicy Twojej aplikacji mogą "komentować" posty i filmy. Używając relacji polimorficznych, możesz użyć pojedynczej tabeli `comments` do przechowywania komentarzy zarówno dla postów, jak i filmów. Po pierwsze, przeanalizujmy strukturę tabel wymaganą do zbudowania tej relacji:

```text
posts
    id - integer
    title - string
    body - text

videos
    id - integer
    title - string
    url - string

comments
    id - integer
    body - text
    commentable_id - integer
    commentable_type - string
```

<a name="one-to-many-polymorphic-model-structure"></a>
#### Struktura modelu

Następnie, przeanalizujmy definicje modeli potrzebne do zbudowania tej relacji:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class Comment extends Model
{
    /**
     * Get the parent commentable model (post or video).
     */
    public function commentable(): MorphTo
    {
        return $this->morphTo();
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphMany;

class Post extends Model
{
    /**
     * Get all of the post's comments.
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphMany;

class Video extends Model
{
    /**
     * Get all of the video's comments.
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}
```

<a name="one-to-many-polymorphic-pobieranie-the-relacja"></a>
#### Pobieranie relacji

Po zdefiniowaniu tabel i modeli bazy danych, możesz uzyskać dostęp do relacji za pomocą dynamicznych właściwości relacji modelu. Na przykład, aby uzyskać dostęp do wszystkich komentarzy dla posta, możemy użyć dynamicznej właściwości `comments`:

```php
use App\Models\Post;

$post = Post::find(1);

foreach ($post->comments as $comment) {
    // ...
}
```

Możesz również pobrać rodzica polimorficznego modelu potomnego, uzyskując dostęp do nazwy metody, która wykonuje wywołanie `morphTo`. W tym przypadku jest to metoda `commentable` modelu `Comment`. Zatem uzyskamy dostęp do tej metody jako dynamicznej właściwości relacji, aby uzyskać dostęp do modelu rodzica komentarza:

```php
use App\Models\Comment;

$comment = Comment::find(1);

$commentable = $comment->commentable;
```

Relacja `commentable` modelu `Comment` zwróci instancję `Post` lub `Video`, w zależności od tego, jaki typ modelu jest rodzicem komentarza.

<a name="polymorphic-automatically-hydrating-parent-modele-on-children"></a>
#### Automatyczna hydracja modeli rodzica na dzieciach

Nawet podczas korzystania z eager loading Eloquent, problemy zapytań "N + 1" mogą wystąpić, jeśli spróbujesz uzyskać dostęp do modelu rodzica z modelu dziecka podczas przeglądania modeli dziecka:

```php
$posts = Post::with('comments')->get();

foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->commentable->title;
    }
}
```

W powyższym przykładzie problem zapytania "N + 1" został wprowadzony, ponieważ mimo że komentarze zostały załadowane zachłannie dla każdego modelu `Post`, Eloquent nie hydruje automatycznie rodzica `Post` dla każdego dziecka modelu `Comment`.

Jeśli chcesz, aby Eloquent automatycznie hydrował modele rodzica na ich dzieci, możesz wywołać metodę `chaperone` podczas definiowania relacji `morphMany`:

```php
class Post extends Model
{
    /**
     * Get all of the post's comments.
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable')->chaperone();
    }
}
```

Lub, jeśli chcesz włączyć automatyczną hydrację rodzica w czasie wykonywania, możesz wywołać metodę `chaperone` podczas załadowania zachłannego relacji:

```php
use App\Models\Post;

$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

<a name="one-of-many-polymorphic-relations"></a>
### Jeden z wielu (Polymorphic)

Czasami model może mieć wiele powiązanych modeli, ale chcesz łatwo pobrać "najnowszy" lub "najstarszy" powiązany model relacji. Na przykład model `User` może być powiązany z wieloma modelami `Image`, ale chcesz zdefiniować wygodny sposób interakcji z najnowszym obrazem, który użytkownik przesłał. Możesz to osiągnąć używając typu relacji `morphOne` połączonego z metodami `ofMany`:

```php
/**
 * Get the user's most recent image.
 */
public function latestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->latestOfMany();
}
```

Podobnie możesz zdefiniować metodę do pobrania "najstarszego", czyli pierwszego, powiązanego modelu relacji:

```php
/**
 * Get the user's oldest image.
 */
public function oldestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->oldestOfMany();
}
```

Domyślnie metody `latestOfMany` i `oldestOfMany` pobiorą najnowszy lub najstarszy powiązany model na podstawie klucza podstawowego modelu, który musi być sortowalny. Jednak czasami możesz chcieć pobrać pojedynczy model z większej relacji używając innych kryteriów sortowania.

Na przykład, używając metody `ofMany`, możesz pobrać najbardziej "polubiony" obraz użytkownika. Metoda `ofMany` przyjmuje kolumnę sortowalną jako pierwszy argument oraz określa, którą funkcję agregującą (`min` lub `max`) zastosować podczas odpytywania powiązanego modelu:

```php
/**
 * Get the user's most popular image.
 */
public function bestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->ofMany('likes', 'max');
}
```

> [!NOTE]
> Możliwe jest konstruowanie bardziej zaawansowanych relacji "Jeden z wielu". Aby uzyskać więcej informacji, zapoznaj się z [dokumentacją Ma jedną z wielu](#advanced-has-one-of-many-relacje).

<a name="many-to-many-polymorphic-relations"></a>
### Wiele do wielu (Polymorphic)

<a name="many-to-many-polymorphic-tabela-structure"></a>
#### Struktura tabeli

Relacje polimorficzne wiele do wielu są nieco bardziej skomplikowane niż relacje "morph one" i "morph many". Na przykład model `Post` i model `Video` mogą współdzielić relację polimorficzną z modelem `Tag`. Używając relacji polimorficznej wiele do wielu w tej sytuacji, Twoja aplikacja może mieć pojedynczą tabelę unikalnych tagów, które mogą być powiązane z postami lub filmami. Najpierw przyjrzyjmy się strukturze tabeli wymaganej do zbudowania tej relacji:

```text
posts
    id - integer
    name - string

videos
    id - integer
    name - string

tags
    id - integer
    name - string

taggables
    tag_id - integer
    taggable_id - integer
    taggable_type - string
```

> [!NOTE]
> Przed zagłębieniem się w polimorficzne relacje wiele do wielu, możesz skorzystać z przeczytania dokumentacji o typowych [relacjach wiele do wielu](#many-to-many).

<a name="many-to-many-polymorphic-model-structure"></a>
#### Struktura modelu

Następnie jesteśmy gotowi do zdefiniowania relacji w modelach. Modele `Post` i `Video` będą zawierały metodę `tags`, która wywołuje metodę `morphToMany` dostarczoną przez podstawową klasę modelu Eloquent.

Metoda `morphToMany` przyjmuje nazwę powiązanego modelu oraz "nazwę relacji". Na podstawie nazwy, którą przypisaliśmy do naszej tabeli pośredniczącej i kluczy, które zawiera, będziemy odwoływać się do relacji jako "taggable":

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphToMany;

class Post extends Model
{
    /**
     * Get all of the tags for the post.
     */
    public function tags(): MorphToMany
    {
        return $this->morphToMany(Tag::class, 'taggable');
    }
}
```

<a name="many-to-many-polymorphic-definiowanie-the-inverse-of-the-relacja"></a>
#### Definiowanie odwrotności relacji

Następnie w modelu `Tag` powinieneś zdefiniować metodę dla każdego z możliwych modeli rodzica. Zatem w tym przykładzie zdefiniujemy metodę `posts` i metodę `videos`. Obie te metody powinny zwrócić wynik metody `morphedByMany`.

Metoda `morphedByMany` przyjmuje nazwę powiązanego modelu oraz "nazwę relacji". Na podstawie nazwy, którą przypisaliśmy do naszej tabeli pośredniczącej i kluczy, które zawiera, będziemy odwoływać się do relacji jako "taggable":

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphToMany;

class Tag extends Model
{
    /**
     * Get all of the posts that are assigned this tag.
     */
    public function posts(): MorphToMany
    {
        return $this->morphedByMany(Post::class, 'taggable');
    }

    /**
     * Get all of the videos that are assigned this tag.
     */
    public function videos(): MorphToMany
    {
        return $this->morphedByMany(Video::class, 'taggable');
    }
}
```

<a name="many-to-many-polymorphic-pobieranie-the-relacja"></a>
#### Pobieranie relacji

Po zdefiniowaniu tabel bazy danych i modeli, możesz uzyskać dostęp do relacji za pomocą swoich modeli. Na przykład, aby uzyskać dostęp do wszystkich tagów posta, możesz użyć dynamicznej właściwości relacji `tags`:

```php
use App\Models\Post;

$post = Post::find(1);

foreach ($post->tags as $tag) {
    // ...
}
```

Możesz pobrać rodzica relacji polimorficznej z polimorficznego modelu potomnego, uzyskując dostęp do nazwy metody, która wykonuje wywołanie `morphedByMany`. W tym przypadku są to metody `posts` lub `videos` modelu `Tag`:

```php
use App\Models\Tag;

$tag = Tag::find(1);

foreach ($tag->posts as $post) {
    // ...
}

foreach ($tag->videos as $video) {
    // ...
}
```

<a name="custom-polymorphic-typy"></a>
### Niestandardowe typy polimorficzne

Domyślnie, Laravel will użyj the fully qualified class name to przechowaj the "typ" of the related model. For instancja, given the one-to-many relacja przykład above where a `Comment` model may belong to a `Post` or a `Video` model, the default `commentable_type` would be either `App\modele\Post` or `App\modele\Video`, respectively. Jednak, you may wish to decouple these values from your application's internal structure.

Na przykład, zamiast używać nazw modeli jako "typ", możemy użyć prostych ciągów znaków, takich jak `post` i `video`. W ten sposób wartości kolumny polimorficznego "typu" w naszej bazie danych pozostaną prawidłowe, nawet jeśli modele zostaną zmienione:

```php
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::enforceMorphMap([
    'post' => 'App\Models\Post',
    'video' => 'App\Models\Video',
]);
```

Możesz wywołać metodę `enforceMorphMap` w metodzie `boot` klasy `App\Providers\AppServiceProvider` lub utworzyć oddzielnego dostawcę usług, jeśli chcesz.

Możesz określić alias morph danego modelu w czasie wykonywania, używając metody `getMorphClass` modelu. Odwrotnie, możesz określić w pełni kwalifikowaną nazwę klasy powiązaną z aliasem morph, używając metody `Relation::getMorphedModel`:

```php
use Illuminate\Database\Eloquent\Relations\Relation;

$alias = $post->getMorphClass();

$class = Relation::getMorphedModel($alias);
```

> [!WARNING]
> Podczas dodawania "mapy morph" do istniejącej aplikacji, każda wartość kolumny morfowanej `*_type` w bazie danych, która nadal zawiera w pełni kwalifikowaną klasę, będzie musiała zostać przekonwertowana na jej nazwę "mapy".

<a name="dynamic-relacje"></a>
### Relacje dynamiczne

Możesz użyć metody `resolveRelationUsing` do definiowania relacji między modelami Eloquent w czasie wykonywania. Chociaż zwykle nie jest to zalecane dla normalnego rozwoju aplikacji, może to być czasami przydatne podczas rozwijania pakietów Laravel.

Metoda `resolveRelationUsing` przyjmuje żądaną nazwę relacji jako swój pierwszy argument. Drugi argument przekazany do metody powinien być domknięciem, które przyjmuje instancję modelu i zwraca prawidłową definicję relacji Eloquent. Zazwyczaj powinieneś konfigurować relacje dynamiczne w metodzie boot [dostawcy usług](/docs/{{version}}/providers):

```php
use App\Models\Order;
use App\Models\Customer;

Order::resolveRelationUsing('customer', function (Order $orderModel) {
    return $orderModel->belongsTo(Customer::class, 'customer_id');
});
```

> [!WARNING]
> Podczas definiowania relacji dynamicznych zawsze podawaj jawne argumenty nazw kluczy do metod relacji Eloquent.

<a name="querying-relations"></a>
## Odpytywanie relacji

Ponieważ wszystkie relacje Eloquent są definiowane za pomocą metod, możesz wywołać te metody, aby uzyskać instancję relacji bez faktycznego wykonywania zapytania w celu załadowania powiązanych modeli. Ponadto wszystkie typy relacji Eloquent działają również jako [konstruktory zapytań](/docs/{{version}}/zapytania), umożliwiając dalsze łączenie ograniczeń do zapytania relacji przed ostatecznym wykonaniem zapytania SQL względem bazy danych.

Na przykład wyobraź sobie aplikację blogową, w której model `User` ma wiele powiązanych modeli `Post`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * Get all of the posts for the user.
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}
```

Możesz zapytać relację `posts` i dodać dodatkowe ograniczenia do relacji w taki sposób:

```php
use App\Models\User;

$user = User::find(1);

$user->posts()->where('active', 1)->get();
```

Możesz użyć dowolnej z metod [konstruktora zapytań](/docs/{{version}}/zapytania) Laravela dla relacji, więc koniecznie zapoznaj się z dokumentacją konstruktora zapytań, aby dowiedzieć się o wszystkich dostępnych metodach.

<a name="chaining-orwhere-clauses-Po-relacje"></a>
#### Łączenie klauzul `orWhere` po relacjach

Jak pokazano w powyższym przykładzie, możesz swobodnie dodawać dodatkowe ograniczenia do relacji podczas ich odpytywania. Jednak używaj ostrożnie podczas łączenia klauzul `orWhere` z relacją, ponieważ klauzule `orWhere` będą logicznie zgrupowane na tym samym poziomie co ograniczenie relacji:

```php
$user->posts()
    ->where('active', 1)
    ->orWhere('votes', '>=', 100)
    ->get();
```

Powyższy przykład wygeneruje następujące SQL. Jak widać, klauzula `or` nakazuje zapytaniu zwrócić _dowolny_ post z więcej niż 100 głosami. Zapytanie nie jest już ograniczone do konkretnego użytkownika:

```sql
select *
from posts
where user_id = ? and active = 1 or votes >= 100
```

W większości sytuacji powinieneś użyć [grup logicznych](/docs/{{version}}/zapytania#logical-grouping), aby pogrupować sprawdzenia warunkowe w nawiasach:

```php
use Illuminate\Database\Eloquent\Builder;

$user->posts()
    ->where(function (Builder $zapytanie) {
        return $zapytanie->where('active', 1)
            ->orWhere('votes', '>=', 100);
    })
    ->get();
```

Powyższy przykład wygeneruje następujące SQL. Zauważ, że grupowanie logiczne poprawnie pogrupowało ograniczenia, a zapytanie pozostaje ograniczone do konkretnego użytkownika:

```sql
select *
from posts
where user_id = ? and (active = 1 or votes >= 100)
```

<a name="relacja-metody-vs-dynamic-właściwości"></a>
### Metody relacji vs. właściwości dynamiczne

Jeśli nie musisz dodawać dodatkowych ograniczeń do zapytania relacji Eloquent, możesz uzyskać dostęp do relacji tak, jakby była właściwością. Na przykład, kontynuując używanie naszych przykładowych modeli `User` i `Post`, możemy uzyskać dostęp do wszystkich postów użytkownika w ten sposób:

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->posts as $post) {
    // ...
}
```

Dynamiczne właściwości relacji wykonują "lazy loading", co oznacza, że załadują swoje dane relacji dopiero, gdy faktycznie uzyskasz do nich dostęp. Z tego powodu programiści często używają [ładowania zachłannego](#eager-loading) do wstępnego załadowania relacji, o których wiedzą, że będą dostępne po załadowaniu modelu. Ładowanie zachłanne zapewnia znaczącą redukcję zapytań SQL, które muszą zostać wykonane w celu załadowania relacji modelu.

<a name="querying-relacja-existence"></a>
### Odpytywanie o istnienie relacji

Podczas pobierania rekordów modelu możesz chcieć ograniczyć wyniki na podstawie istnienia relacji. Na przykład wyobraź sobie, że chcesz pobrać wszystkie posty na blogu, które mają co najmniej jeden komentarz. Aby to zrobić, możesz przekazać nazwę relacji do metod `has` i `orHas`:

```php
use App\Models\Post;

// pobierz wszystkie posty, które mają co najmniej jeden komentarz...
$posts = Post::has('comments')->get();
```

You may also określ an operator and count wartość to further customize the zapytanie:

```php
// pobierz all posts that have three or more comments...
$posts = Post::has('comments', '>=', 3)->get();
```

Nested `has` statements may be constructed używając "dot" notation. Na przykład, you may pobierz all posts that have at least one comment that has at least one image:

```php
// pobierz posts that have at least one comment with images...
$posts = Post::has('comments.images')->get();
```

Jeśli potrzebujesz jeszcze więcej możliwości, możesz użyć metod `whereHas` i `orWhereHas` do zdefiniowania dodatkowych ograniczeń zapytania dla Twoich zapytań `has`, takich jak inspekcja treści komentarza:

```php
use Illuminate\Database\Eloquent\Builder;

// pobierz posty z co najmniej jednym komentarzem zawierającym słowa jak code%...
$posts = Post::whereHas('comments', function (Builder $zapytanie) {
    $zapytanie->where('content', 'like', 'code%');
})->get();

// pobierz posty z co najmniej dziesięcioma komentarzami zawierającymi słowa jak code%...
$posts = Post::whereHas('comments', function (Builder $zapytanie) {
    $zapytanie->where('content', 'like', 'code%');
}, '>=', 10)->get();
```

> [!WARNING]
> Eloquent obecnie nie obsługuje odpytywania o istnienie relacji między bazami danych. Relacje muszą istnieć w tej samej bazie danych.

<a name="many-to-many-relacja-existence-zapytania"></a>
#### Zapytania o istnienie relacji wiele do wielu

Metoda `whereAttachedTo` może być użyta do odpytywania modeli, które mają załącznik wiele do wielu do modelu lub kolekcji modeli:

```php
$users = User::whereAttachedTo($role)->get();
```

Możesz również przekazać instancję [kolekcji](/docs/{{version}}/eloquent-collections) do metody `whereAttachedTo`. Podczas tego Laravel pobierze modele, które są załączone do któregokolwiek z modeli w kolekcji:

```php
$tags = Tag::whereLike('name', '%laravel%')->get();

$posts = Post::whereAttachedTo($tags)->get();
```

<a name="inline-relacja-existence-zapytania"></a>
#### Inline relacja Existence zapytania

If you would like to zapytanie for a relacja's existence with a single, simple where condition attached to the relacja zapytanie, you may find it more convenient to użyj the `whereRelation`, `orWhereRelation`, `whereMorphRelation`, and `orWhereMorphRelation` metody. Na przykład, we may zapytanie for all posts that have unapproved comments:

```php
użyj App\modele\Post;

$posts = Post::whereRelation('comments', 'is_approved', false)->get();
```

Of course, like calls to the zapytanie builder's `where` metoda, you may also określ an operator:

```php
$posts = Post::whereRelation(
    'comments', 'created_at', '>=', now()->minus(hours: 1)
)->get();
```

<a name="querying-relacja-absence"></a>
### Odpytywanie o brak relacji

Podczas pobierania rekordów modelu możesz chcieć ograniczyć wyniki na podstawie braku relacji. Na przykład wyobraź sobie, że chcesz pobrać wszystkie posty na blogu, które **nie** mają żadnych komentarzy. Aby to zrobić, możesz przekazać nazwę relacji do metod `doesntHave` i `orDoesntHave`:

```php
use App\Models\Post;

$posts = Post::doesntHave('comments')->get();
```

Jeśli potrzebujesz jeszcze więcej możliwości, możesz użyć metod `whereDoesntHave` i `orWhereDoesntHave` do dodania dodatkowych ograniczeń zapytania do zapytań `doesntHave`, takich jak inspekcja treści komentarza:

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::whereDoesntHave('comments', function (Builder $zapytanie) {
    $zapytanie->where('content', 'like', 'code%');
})->get();
```

Możesz użyć notacji "kropkowej" do wykonania zapytania względem zagnieżdżonej relacji. Na przykład następujące zapytanie pobierze wszystkie posty, które nie mają komentarzy, oraz posty, które mają komentarze, gdzie żaden z komentarzy nie pochodzi od zbanowanych użytkowników:

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::whereDoesntHave('comments.author', function (Builder $zapytanie) {
    $zapytanie->where('banned', 1);
})->get();
```

<a name="querying-morph-to-relacje"></a>
### Odpytywanie relacji Morph To

Aby zapytać o istnienie relacji "morph to", możesz użyć metod `whereHasMorph` i `whereDoesntHaveMorph`. Te metody przyjmują nazwę relacji jako pierwszy argument. Następnie metody przyjmują nazwy powiązanych modeli, które chcesz uwzględnić w zapytaniu. Na koniec możesz przekazać domknięcie, które dostosowuje zapytanie relacji:

```php
use App\Models\Comment;
use App\Models\Post;
use App\Models\Video;
use Illuminate\Database\Eloquent\Builder;

// pobierz komentarze powiązane z postami lub filmami z tytułem jak code%...
$comments = Comment::whereHasMorph(
    'commentable',
    [Post::class, Video::class],
    function (Builder $zapytanie) {
        $zapytanie->where('title', 'like', 'code%');
    }
)->get();

// pobierz komentarze powiązane z postami z tytułem nie jak code%...
$comments = Comment::whereDoesntHaveMorph(
    'commentable',
    Post::class,
    function (Builder $zapytanie) {
        $zapytanie->where('title', 'like', 'code%');
    }
)->get();
```

Czasami możesz potrzebować dodać ograniczenia zapytania na podstawie "typu" powiązanego modelu polimorficznego. Domknięcie przekazane do metody `whereHasMorph` może otrzymać wartość `$type` jako drugi argument. Ten argument pozwala Ci zbadać "typ" budowanego zapytania:

```php
use Illuminate\Database\Eloquent\Builder;

$comments = Comment::whereHasMorph(
    'commentable',
    [Post::class, Video::class],
    function (Builder $zapytanie, string $typ) {
        $kolumna = $typ === Post::class ? 'content' : 'title';

        $zapytanie->where($kolumna, 'like', 'code%');
    }
)->get();
```

Czasami możesz chcieć zapytać o dzieci relacji "morph to" rodzica. Możesz to osiągnąć używając metod `whereMorphedTo` i `whereNotMorphedTo`, które automatycznie określą odpowiednie mapowanie typu morph dla danego modelu. Te metody przyjmują nazwę relacji `morphTo` jako pierwszy argument i powiązany model rodzica jako drugi argument:

```php
$comments = Comment::whereMorphedTo('commentable', $post)
    ->orWhereMorphedTo('commentable', $video)
    ->get();
```

<a name="querying-all-morph-to-related-modele"></a>
#### Odpytywanie wszystkich powiązanych modeli

Zamiast przekazywania tablicy możliwych modeli polimorficznych, możesz przekazać `*` jako wartość wieloznaczną. To nakazuje Laravelowi pobrać wszystkie możliwe typy polimorficzne z bazy danych. Laravel wykona dodatkowe zapytanie w celu wykonania tej operacji:

```php
use Illuminate\Database\Eloquent\Builder;

$comments = Comment::whereHasMorph('commentable', '*', function (Builder $zapytanie) {
    $zapytanie->where('title', 'like', 'foo%');
})->get();
```

<a name="aggregating-related-modele"></a>
## Agregowanie powiązanych modeli

<a name="counting-related-modele"></a>
### Liczenie powiązanych modeli

Czasami możesz chcieć policzyć liczbę powiązanych modeli dla danej relacji bez faktycznego ładowania modeli. Aby to osiągnąć, możesz użyć metody `withCount`. Metoda `withCount` umieści atrybut `{relation}_count` na wynikowych modelach:

```php
use App\Models\Post;

$posts = Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo $post->comments_count;
}
```

Przez przekazanie tablicy do metody `withCount`, możesz dodać "liczby" dla wielu relacji, a także dodać dodatkowe ograniczenia do zapytań:

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::withCount(['votes', 'comments' => function (Builder $query) {
    $query->where('content', 'like', 'code%');
}])->get();

echo $posts[0]->votes_count;
echo $posts[0]->comments_count;
```

Możesz również nadać alias wynikom liczenia relacji, umożliwiając wiele liczników na tej samej relacji:

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::withCount([
    'comments',
    'comments as pending_comments_count' => function (Builder $query) {
        $query->where('approved', false);
    },
])->get();

echo $posts[0]->comments_count;
echo $posts[0]->pending_comments_count;
```

<a name="deferred-count-loading"></a>
#### Odroczone ładowanie liczników

Korzystając z metody `loadCount`, możesz załadować licznik relacji po tym, jak model rodzica został już pobrany:

```php
$book = Book::first();

$book->loadCount('genres');
```

Jeśli musisz ustawić dodatkowe ograniczenia zapytania na zapytaniu liczącym, możesz przekazać tablicę z kluczami będącymi relacjami, które chcesz policzyć. Wartości tablicy powinny być funkcjami anonimowymi, które otrzymają instancję konstruktora zapytań:

```php
$book->loadCount(['reviews' => function (Builder $query) {
    $query->where('rating', 5);
}])
```

<a name="relacja-counting-and-custom-select-statements"></a>
#### Liczenie relacji i niestandardowe instrukcje Select

Jeśli łączysz `withCount` z instrukcją `select`, upewnij się, że wywołujesz `withCount` po metodzie `select`:

```php
$posts = Post::select(['title', 'body'])
    ->withCount('comments')
    ->get();
```

<a name="other-aggregate-functions"></a>
### Inne funkcje agregujące

Oprócz metody `withCount`, Eloquent zapewnia metody `withMin`, `withMax`, `withAvg`, `withSum` oraz `withExists`. Te metody umieszczą atrybut `{relation}_{function}_{column}` w wynikowych modelach:

```php
use App\Models\Post;

$posts = Post::withSum('comments', 'votes')->get();

foreach ($posts as $post) {
    echo $post->comments_sum_votes;
}
```

Jeśli chcesz uzyskać dostęp do wyniku funkcji agregującej przy użyciu innej nazwy, możesz określić własny alias:

```php
$posts = Post::withSum('comments as total_comments', 'votes')->get();

foreach ($posts as $post) {
    echo $post->total_comments;
}
```

Podobnie jak metoda `loadCount`, dostępne są również odroczone wersje tych metod. Te dodatkowe operacje agregujące mogą być wykonywane na modelach Eloquent, które zostały już pobrane:

```php
$post = Post::first();

$post->loadSum('comments', 'votes');
```

Jeśli łączysz te metody agregujące z instrukcją `select`, upewnij się, że wywołujesz metody agregujące po metodzie `select`:

```php
$posts = Post::select(['title', 'body'])
    ->withExists('comments')
    ->get();
```

<a name="counting-related-modele-on-morph-to-relacje"></a>
### Liczenie powiązanych modeli w relacjach Morph To

Jeśli chcesz załadować zachłannie relację "morph to", a także liczby powiązanych modeli dla różnych encji, które mogą być zwrócone przez tę relację, możesz użyć metody `with` w połączeniu z metodą `morphWithCount` relacji `morphTo`.

W tym przykładzie załóżmy, że modele `Photo` i `Post` mogą tworzyć modele `ActivityFeed`. Załóżmy, że model `ActivityFeed` definiuje relację "morph to" o nazwie `parentable`, która pozwala nam pobrać rodzica `Photo` lub `Post` dla danej instancji `ActivityFeed`. Dodatkowo załóżmy, że modele `Photo` "mają wiele" modeli `Tag`, a modele `Post` "mają wiele" modeli `Comment`.

Teraz załóżmy, że chcemy pobrać instancje `ActivityFeed` i załadować zachłannie modele rodzica `parentable` dla każdej instancji `ActivityFeed`. Dodatkowo chcemy pobrać liczbę tagów powiązanych z każdym zdjęciem rodzica oraz liczbę komentarzy powiązanych z każdym postem rodzica:

```php
use Illuminate\Database\Eloquent\Relations\MorphTo;

$activities = ActivityFeed::with([
    'parentable' => function (MorphTo $morphTo) {
        $morphTo->morphWithCount([
            Photo::class => ['tags'],
            Post::class => ['comments'],
        ]);
    }])->get();
```

<a name="morph-to-deferred-count-loading"></a>
#### Odroczone ładowanie liczników

Załóżmy, że już pobraliśmy zestaw modeli `ActivityFeed` i teraz chcielibyśmy załadować zagnieżdżone liczniki relacji dla różnych modeli `parentable` powiązanych z kanałami aktywności. Możesz użyć metody `loadMorphCount`, aby to osiągnąć:

```php
$activities = ActivityFeed::with('parentable')->get();

$activities->loadMorphCount('parentable', [
    Photo::class => ['tags'],
    Post::class => ['comments'],
]);
```

<a name="eager-loading"></a>
## Ładowanie zachłanne

Podczas dostępu do relacji Eloquent jako właściwości, powiązane modele są "ładowane leniwie". Oznacza to, że dane relacji nie są faktycznie ładowane, dopóki nie uzyskasz dostępu do właściwości po raz pierwszy. Jednak Eloquent może "załadować zachłannie" relacje w momencie wykonywania zapytania o model rodzica. Ładowanie zachłanne łagodzi problem zapytań "N + 1". Aby zilustrować problem zapytań N + 1, rozważmy model `Book`, który "należy do" modelu `Author`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Book extends Model
{
    /**
     * Get the author that wrote the book.
     */
    public function author(): BelongsTo
    {
        return $this->belongsTo(Author::class);
    }
}
```

Teraz pobierzmy wszystkie książki i ich autorów:

```php
use App\Models\Book;

$books = Book::all();

foreach ($books as $book) {
    echo $book->author->name;
}
```

Ta pętla wykona jedno zapytanie, aby pobrać wszystkie książki z tabeli bazy danych, a następnie kolejne zapytanie dla każdej książki w celu pobrania autora książki. Więc jeśli mamy 25 książek, powyższy kod wykona 26 zapytań: jedno dla oryginalnych książek i 25 dodatkowych zapytań, aby pobrać autora każdej książki.

Na szczęście możemy użyć ładowania zachłannego, aby zredukować tę operację do zaledwie dwóch zapytań. Podczas budowania zapytania możesz określić, które relacje powinny być ładowane zachłannie za pomocą metody `with`:

```php
$books = Book::with('author')->get();

foreach ($books as $book) {
    echo $book->author->name;
}
```

Dla tej operacji zostaną wykonane tylko dwa zapytania - jedno zapytanie do pobrania wszystkich książek i jedno zapytanie do pobrania wszystkich autorów dla wszystkich książek:

```sql
select * from books

select * from authors where id in (1, 2, 3, 4, 5, ...)
```

<a name="eager-loading-multiple-relacje"></a>
#### Ładowanie zachłanne wielu relacji

Czasami możesz potrzebować załadować zachłannie kilka różnych relacji. Aby to zrobić, po prostu przekaż tablicę relacji do metody `with`:

```php
$books = Book::with(['author', 'publisher'])->get();
```

<a name="nested-eager-loading"></a>
#### Nested Ładowanie zachłanne

To eager load a relacja's relacje, you may użyj "dot" syntax. Na przykład, let's eager load all of the book's authors and all of the author's personal contacts:

```php
$books = Book::with('author.contacts')->get();
```

Alternatywnie możesz określić zagnieżdżone relacje ładowane zachłannie, przekazując zagnieżdżoną tablicę do metody `with`, co może być wygodne podczas ładowania zachłannego wielu zagnieżdżonych relacji:

```php
$books = Book::with([
    'author' => [
        'contacts',
        'publisher',
    ],
])->get();
```

<a name="nested-eager-loading-morphto-relacje"></a>
#### Zagnieżdżone ładowanie zachłanne relacji `morphTo`

Jeśli chcesz załadować zachłannie relację `morphTo`, a także zagnieżdżone relacje dla różnych encji, które mogą być zwrócone przez tę relację, możesz użyć metody `with` w połączeniu z metodą `morphWith` relacji `morphTo`. Aby zilustrować tę metodę, rozważmy następujący model:

```php
<?php

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class ActivityFeed extends Model
{
    /**
     * Get the parent of the activity feed record.
     */
    public function parentable(): MorphTo
    {
        return $this->morphTo();
    }
}
```

W tym przykładzie załóżmy, że modele `Event`, `Photo` i `Post` mogą tworzyć modele `ActivityFeed`. Dodatkowo załóżmy, że modele `Event` należą do modelu `Calendar`, modele `Photo` są powiązane z modelami `Tag`, a modele `Post` należą do modelu `Author`.

Korzystając z tych definicji modeli i relacji, możemy pobrać instancje modelu `ActivityFeed` i załadować zachłannie wszystkie modele `parentable` oraz ich odpowiednie zagnieżdżone relacje:

```php
use Illuminate\Database\Eloquent\Relations\MorphTo;

$activities = ActivityFeed::query()
    ->with(['parentable' => function (MorphTo $morphTo) {
        $morphTo->morphWith([
            Event::class => ['calendar'],
            Photo::class => ['tags'],
            Post::class => ['author'],
        ]);
    }])->get();
```

<a name="eager-loading-specific-kolumny"></a>
#### Ładowanie zachłanne określonych kolumn

Nie zawsze możesz potrzebować każdej kolumny z pobieranych relacji. Z tego powodu Eloquent pozwala określić, które kolumny relacji chciałbyś pobrać:

```php
$books = Book::with('author:id,name,book_id')->get();
```

> [!WARNING]
> Podczas korzystania z tej funkcji zawsze powinieneś uwzględnić kolumnę `id` oraz wszelkie odpowiednie kolumny kluczy obcych na liście kolumn, które chcesz pobrać.

<a name="eager-loading-by-default"></a>
#### Ładowanie zachłanne domyślnie

Czasami możesz chcieć zawsze ładować niektóre relacje podczas pobierania modelu. Aby to osiągnąć, możesz zdefiniować właściwość `$with` w modelu:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Book extends Model
{
    /**
     * The relationships that should always be loaded.
     *
     * @var array
     */
    protected $with = ['author'];

    /**
     * Get the author that wrote the book.
     */
    public function author(): BelongsTo
    {
        return $this->belongsTo(Author::class);
    }

    /**
     * Get the genre of the book.
     */
    public function genre(): BelongsTo
    {
        return $this->belongsTo(Genre::class);
    }
}
```

Jeśli chcesz usunąć element z właściwości `$with` dla pojedynczego zapytania, możesz użyć metody `without`:

```php
$books = Book::without('author')->get();
```

Jeśli chcesz zastąpić wszystkie elementy w właściwości `$with` dla pojedynczego zapytania, możesz użyć metody `withOnly`:

```php
$books = Book::withOnly('genre')->get();
```

<a name="constraining-eager-loads"></a>
### Ograniczanie ładowania zachłannego

Czasami możesz chcieć załadować zachłannie relację, ale także określić dodatkowe warunki zapytania dla zapytania ładowania zachłannego. Możesz to osiągnąć, przekazując tablicę relacji do metody `with`, gdzie kluczem tablicy jest nazwa relacji, a wartością tablicy jest funkcja anonimowa, która dodaje dodatkowe ograniczenia do zapytania ładowania zachłannego:

```php
use App\Models\User;

$users = User::with(['posts' => function ($query) {
    $query->where('title', 'like', '%code%');
}])->get();
```

W tym przykładzie Eloquent załaduje zachłannie tylko posty, w których kolumna `title` posta zawiera słowo `code`. Możesz wywołać inne metody [konstruktora zapytań](/docs/{{version}}/zapytania), aby dodatkowo dostosować operację ładowania zachłannego:

```php
$users = User::with(['posts' => function ($query) {
    $query->orderBy('created_at', 'desc');
}])->get();
```

<a name="constraining-eager-loading-of-morph-to-relacje"></a>
#### Ograniczanie ładowania zachłannego relacji `morphTo`

Jeśli ładujesz zachłannie relację `morphTo`, Eloquent wykona wiele zapytań, aby pobrać każdy typ powiązanego modelu. Możesz dodać dodatkowe ograniczenia do każdego z tych zapytań, używając metody `constrain` relacji `MorphTo`:

```php
use Illuminate\Database\Eloquent\Relations\MorphTo;

$comments = Comment::with(['commentable' => function (MorphTo $morphTo) {
    $morphTo->constrain([
        Post::class => function ($query) {
            $query->whereNull('hidden_at');
        },
        Video::class => function ($query) {
            $query->where('type', 'educational');
        },
    ]);
}])->get();
```

W tym przykładzie Eloquent załaduje zachłannie tylko posty, które nie zostały ukryte, oraz filmy, które mają wartość `type` równą "educational".

<a name="constraining-eager-loads-with-relacja-existence"></a>
#### Ograniczanie ładowania zachłannego z istnieniem relacji

Czasami możesz zauważyć, że musisz sprawdzić istnienie relacji, jednocześnie ładując relację na podstawie tych samych warunków. Na przykład możesz chcieć pobrać tylko modele `User`, które mają podrzędne modele `Post` spełniające dany warunek zapytania, jednocześnie ładując zachłannie pasujące posty. Możesz to osiągnąć, używając metody `withWhereHas`:

```php
use App\Models\User;

$users = User::withWhereHas('posts', function ($query) {
    $query->where('featured', true);
})->get();
```

<a name="lazy-eager-loading"></a>
### Leniwe ładowanie zachłanne

Czasami możesz potrzebować załadować zachłannie relację po tym, jak model rodzica został już pobrany. Na przykład może to być przydatne, jeśli musisz dynamicznie zdecydować, czy załadować powiązane modele:

```php
use App\Models\Book;

$books = Book::all();

if ($condition) {
    $books->load('author', 'publisher');
}
```

Jeśli musisz ustawić dodatkowe ograniczenia zapytania na zapytaniu ładowania zachłannego, możesz przekazać tablicę z kluczami będącymi relacjami, które chcesz załadować. Wartości tablicy powinny być instancjami funkcji anonimowych, które otrzymają instancję zapytania:

```php
$author->load(['books' => function ($query) {
    $query->orderBy('published_date', 'asc');
}]);
```

Aby załadować relację tylko wtedy, gdy nie została jeszcze załadowana, użyj metody `loadMissing`:

```php
$book->loadMissing('author');
```

<a name="nested-lazy-eager-loading-morphto"></a>
#### Zagnieżdżone leniwe ładowanie zachłanne i `morphTo`

Jeśli chcesz załadować zachłannie relację `morphTo`, a także zagnieżdżone relacje dla różnych encji, które mogą być zwrócone przez tę relację, możesz użyć metody `loadMorph`.

Ta metoda przyjmuje nazwę relacji `morphTo` jako pierwszy argument oraz tablicę par model/relacja jako drugi argument. Aby zilustrować tę metodę, rozważmy następujący model:

```php
<?php

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class ActivityFeed extends Model
{
    /**
     * Get the parent of the activity feed record.
     */
    public function parentable(): MorphTo
    {
        return $this->morphTo();
    }
}
```

W tym przykładzie załóżmy, że modele `Event`, `Photo` i `Post` mogą tworzyć modele `ActivityFeed`. Dodatkowo załóżmy, że modele `Event` należą do modelu `Calendar`, modele `Photo` są powiązane z modelami `Tag`, a modele `Post` należą do modelu `Author`.

Korzystając z tych definicji modeli i relacji, możemy pobrać instancje modelu `ActivityFeed` i załadować zachłannie wszystkie modele `parentable` oraz ich odpowiednie zagnieżdżone relacje:

```php
$activities = ActivityFeed::with('parentable')
    ->get()
    ->loadMorph('parentable', [
        Event::class => ['calendar'],
        Photo::class => ['tags'],
        Post::class => ['author'],
    ]);
```

<a name="automatic-eager-loading"></a>
### Automatyczne ładowanie zachłanne

> [!WARNING]
> Ta funkcja jest obecnie w wersji beta w celu zebrania opinii społeczności. Zachowanie i funkcjonalność tej funkcji mogą się zmieniać nawet w wydaniach łatkowych.

W wielu przypadkach Laravel może automatycznie załadować zachłannie relacje, do których uzyskujesz dostęp. Aby włączyć automatyczne ładowanie zachłanne, powinieneś wywołać metodę `Model::automaticallyEagerLoadRelationships` w metodzie `boot` dostawcy usług `AppServiceProvider` Twojej aplikacji:

```php
use Illuminate\Database\Eloquent\Model;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Model::automaticallyEagerLoadRelationships();
}
```

Kiedy ta funkcja jest włączona, Laravel będzie próbował automatycznie załadować wszystkie relacje, do których uzyskujesz dostęp, które nie zostały wcześniej załadowane. Na przykład rozważ następujący scenariusz:

```php
use App\Models\User;

$users = User::all();

foreach ($users as $user) {
    foreach ($user->posts as $post) {
        foreach ($post->comments as $comment) {
            echo $comment->content;
        }
    }
}
```

Zazwyczaj powyższy kod wykonałby zapytanie dla każdego użytkownika w celu pobrania jego postów, a także zapytanie dla każdego posta w celu pobrania jego komentarzy. Jednak gdy funkcja `automaticallyEagerLoadRelationships` została włączona, Laravel automatycznie [leniwie załaduje zachłannie](#lazy-eager-loading) posty dla wszystkich użytkowników w kolekcji użytkowników, gdy spróbujesz uzyskać dostęp do postów któregokolwiek z pobranych użytkowników. Podobnie, gdy spróbujesz uzyskać dostęp do komentarzy dla któregokolwiek pobranego posta, wszystkie komentarze zostaną leniwie załadowane zachłannie dla wszystkich postów, które zostały pierwotnie pobrane.

Jeśli nie chcesz globalnie włączać automatycznego ładowania zachłannego, nadal możesz włączyć tę funkcję dla pojedynczej instancji kolekcji Eloquent, wywołując metodę `withRelationshipAutoloading` na kolekcji:

```php
$users = User::where('vip', true)->get();

return $users->withRelationshipAutoloading();
```

<a name="preventing-lazy-loading"></a>
### Zapobieganie leniwemu ładowaniu

Jak wcześniej omówiono, ładowanie zachłanne relacji często może zapewnić znaczne korzyści wydajnościowe dla Twojej aplikacji. Dlatego, jeśli chcesz, możesz polecić Laravel zawsze zapobiegać leniwemu ładowaniu relacji. Aby to osiągnąć, możesz wywołać metodę `preventLazyLoading` oferowaną przez bazową klasę modelu Eloquent. Zazwyczaj powinieneś wywołać tę metodę w metodzie `boot` klasy `AppServiceProvider` Twojej aplikacji.

Metoda `preventLazyLoading` przyjmuje opcjonalny argument boolean, który wskazuje, czy leniwe ładowanie powinno być zabronione. Na przykład możesz chcieć wyłączyć leniwe ładowanie tylko w środowiskach nieprodukcyjnych, tak aby Twoje środowisko produkcyjne nadal działało normalnie, nawet jeśli leniwie załadowana relacja jest przypadkowo obecna w kodzie produkcyjnym:

```php
use Illuminate\Database\Eloquent\Model;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Model::preventLazyLoading(! $this->app->isProduction());
}
```

Po zapobieganiu leniwemu ładowaniu Eloquent rzuci wyjątek `Illuminate\Database\LazyLoadingViolationException`, gdy Twoja aplikacja spróbuje leniwie załadować jakąkolwiek relację Eloquent.

Możesz dostosować zachowanie naruszeń leniwego ładowania, używając metody `handleLazyLoadingViolationsUsing`. Na przykład, używając tej metody, możesz polecić, aby naruszenia leniwego ładowania były tylko logowane zamiast przerywać wykonanie aplikacji za pomocą wyjątków:

```php
Model::handleLazyLoadingViolationUsing(function (Model $model, string $relation) {
    $class = $model::class;

    info("Attempted to lazy load [{$relation}] on model [{$class}].");
});
```

<a name="inserting-and-updating-related-modele"></a>
## Wstawianie i aktualizowanie powiązanych modeli

<a name="the-save-metoda"></a>
### Metoda `save`

Eloquent zapewnia wygodne metody dodawania nowych modeli do relacji. Na przykład być może musisz dodać nowy komentarz do posta. Zamiast ręcznie ustawiać atrybut `post_id` w modelu `Comment`, możesz wstawić komentarz, używając metody `save` relacji:

```php
use App\Models\Comment;
use App\Models\Post;

$comment = new Comment(['message' => 'A new comment.']);

$post = Post::find(1);

$post->comments()->save($comment);
```

Zauważ, że nie uzyskaliśmy dostępu do relacji `comments` jako właściwości dynamicznej. Zamiast tego wywołaliśmy metodę `comments`, aby uzyskać instancję relacji. Metoda `save` automatycznie doda odpowiednią wartość `post_id` do nowego modelu `Comment`.

Jeśli musisz zapisać wiele powiązanych modeli, możesz użyć metody `saveMany`:

```php
$post = Post::find(1);

$post->comments()->saveMany([
    new Comment(['message' => 'A new comment.']),
    new Comment(['message' => 'Another new comment.']),
]);
```

The `save` and `saveMany` metody will persist the given model instancje, but will not add the newly persisted modele to any in-memory relacje that are already loaded onto the parent model. If you plan on dostępu the relacja Po używając the `save` or `saveMany` metody, you may wish to użyj the `refresh` metoda to reload the model and its relacje:

```php
$post->comments()->save($comment);

$post->refresh();

// All comments, including the newly saved comment...
$post->comments;
```

<a name="the-push-metoda"></a>
#### Rekursywne zapisywanie modeli i relacji

Jeśli chciałbyś zapisać (`save`) swój model i wszystkie powiązane z nim relacje, możesz użyć metody `push`. W tym przykładzie model `Post` zostanie zapisany, a także jego komentarze i autorzy komentarzy:

```php
$post = Post::find(1);

$post->comments[0]->message = 'Message';
$post->comments[0]->author->name = 'Author Name';

$post->push();
```

Metoda `pushQuietly` może być użyta do zapisania modelu i jego powiązanych relacji bez wywoływania żadnych zdarzeń:

```php
$post->pushQuietly();
```

<a name="the-create-metoda"></a>
### Metoda `create`

Oprócz metod `save` i `saveMany`, możesz również użyć metody `create`, która przyjmuje tablicę atrybutów, tworzy model i wstawia go do bazy danych. Różnica między `save` a `create` polega na tym, że `save` przyjmuje pełną instancję modelu Eloquent, podczas gdy `create` przyjmuje zwykłą tablicę PHP (`array`). Nowo utworzony model zostanie zwrócony przez metodę `create`:

```php
use App\Models\Post;

$post = Post::find(1);

$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);
```

Możesz użyć metody `createMany`, aby utworzyć wiele powiązanych modeli:

```php
$post = Post::find(1);

$post->comments()->createMany([
    ['message' => 'A new comment.'],
    ['message' => 'Another new comment.'],
]);
```

Metody `createQuietly` i `createManyQuietly` mogą być użyte do utworzenia modelu (modeli) bez wysyłania żadnych zdarzeń:

```php
$user = User::find(1);

$user->posts()->createQuietly([
    'title' => 'Post title.',
]);

$user->posts()->createManyQuietly([
    ['title' => 'First post.'],
    ['title' => 'Second post.'],
]);
```

You may also użyj the `findOrNew`, `firstOrNew`, `firstOrCreate`, and `updateOrCreate` metody to [create and update modele on relacje](/docs/{{version}}/eloquent#upserts).

> [!NOTE]
> Przed używając Metoda `create`, be sure to review the [mass assignment](/docs/{{version}}/eloquent#mass-assignment) documentation.

<a name="updating-belongs-to-relacje"></a>
### Relacje Belongs To

Jeśli chciałbyś przypisać model potomny do nowego modelu rodzica, możesz użyć metody `associate`. W tym przykładzie model `User` definiuje relację `belongsTo` do modelu `Account`. Ta metoda `associate` ustawi klucz obcy w modelu potomnym:

```php
use App\Models\Account;

$account = Account::find(10);

$user->account()->associate($account);

$user->save();
```

Aby usunąć model rodzica z modelu potomnego, możesz użyć metody `dissociate`. Ta metoda ustawi klucz obcy relacji na `null`:

```php
$user->account()->dissociate();

$user->save();
```

<a name="updating-many-to-many-relacje"></a>
### Relacje wiele do wielu

<a name="attaching-detaching"></a>
#### Dołączanie / Odłączanie

Eloquent zapewnia również metody, które ułatwiają pracę z relacjami wiele do wielu. Na przykład wyobraźmy sobie, że użytkownik może mieć wiele ról, a rola może mieć wielu użytkowników. Możesz użyć metody `attach`, aby dołączyć rolę do użytkownika, wstawiając rekord w tabeli pośredniej relacji:

```php
use App\Models\User;

$user = User::find(1);

$user->roles()->attach($roleId);
```

Podczas dołączania relacji do modelu możesz również przekazać tablicę dodatkowych danych do wstawienia do tabeli pośredniej:

```php
$user->roles()->attach($roleId, ['expires' => $expires]);
```

Czasami może być konieczne usunięcie roli od użytkownika. Aby usunąć rekord relacji wiele do wielu, użyj metody `detach`. Metoda `detach` usunie odpowiedni rekord z tabeli pośredniej; jednak oba modele pozostaną w bazie danych:

```php
// Detach a single role from the user...
$user->roles()->detach($roleId);

// Detach all roles from the user...
$user->roles()->detach();
```

Dla wygody metody `attach` i `detach` akceptują również tablice identyfikatorów jako dane wejściowe:

```php
$user = User::find(1);

$user->roles()->detach([1, 2, 3]);

$user->roles()->attach([
    1 => ['expires' => $expires],
    2 => ['expires' => $expires],
]);
```

<a name="syncing-associations"></a>
#### Synchronizacja powiązań

Możesz również użyć metody `sync` do konstruowania powiązań wiele do wielu. Metoda `sync` przyjmuje tablicę identyfikatorów do umieszczenia w tabeli pośredniej. Wszystkie identyfikatory, które nie znajdują się w podanej tablicy, zostaną usunięte z tabeli pośredniej. Tak więc po zakończeniu tej operacji tylko identyfikatory w podanej tablicy będą istnieć w tabeli pośredniej:

```php
$user->roles()->sync([1, 2, 3]);
```

Możesz również przekazać dodatkowe wartości tabeli pośredniej wraz z identyfikatorami:

```php
$user->roles()->sync([1 => ['expires' => true], 2, 3]);
```

Jeśli chciałbyś wstawić te same wartości tabeli pośredniej dla każdego z zsynchronizowanych identyfikatorów modeli, możesz użyć metody `syncWithPivotValues`:

```php
$user->roles()->syncWithPivotValues([1, 2, 3], ['active' => true]);
```

Jeśli nie chcesz odłączać istniejących identyfikatorów, które brakują w podanej tablicy, możesz użyć metody `syncWithoutDetaching`:

```php
$user->roles()->syncWithoutDetaching([1, 2, 3]);
```

<a name="toggling-associations"></a>
#### Przełączanie powiązań

Relacja wiele do wielu zapewnia również metodę `toggle`, która "przełącza" status dołączenia podanych identyfikatorów powiązanych modeli. Jeśli podany identyfikator jest obecnie dołączony, zostanie odłączony. Podobnie, jeśli jest obecnie odłączony, zostanie dołączony:

```php
$user->roles()->toggle([1, 2, 3]);
```

Możesz również przekazać dodatkowe wartości tabeli pośredniej wraz z identyfikatorami:

```php
$user->roles()->toggle([
    1 => ['expires' => true],
    2 => ['expires' => true],
]);
```

<a name="updating-a-record-on-the-intermediate-tabela"></a>
#### Aktualizowanie rekordu w tabeli pośredniej

Jeśli musisz zaktualizować istniejący wiersz w tabeli pośredniej relacji, możesz użyć metody `updateExistingPivot`. Ta metoda przyjmuje klucz obcy rekordu pośredniego oraz tablicę atrybutów do zaktualizowania:

```php
$user = User::find(1);

$user->roles()->updateExistingPivot($roleId, [
    'active' => false,
]);
```

<a name="touching-parent-timestamps"></a>
## Aktualizowanie znaczników czasu rodzica

Kiedy model definiuje relację `belongsTo` lub `belongsToMany` do innego modelu, tak jak `Comment` należący do `Post`, czasami pomocne jest zaktualizowanie znacznika czasu rodzica, gdy model potomny jest aktualizowany.

Na przykład, gdy model `Comment` jest aktualizowany, możesz chcieć automatycznie "dotknąć" znacznika czasu `updated_at` posiadającego go `Post`, tak aby został ustawiony na bieżącą datę i godzinę. Aby to osiągnąć, możesz dodać właściwość `touches` do swojego modelu potomnego zawierającą nazwy relacji, które powinny mieć zaktualizowane znaczniki czasu `updated_at`, gdy model potomny jest aktualizowany:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Comment extends Model
{
    /**
     * All of the relationships to be touched.
     *
     * @var array
     */
    protected $touches = ['post'];

    /**
     * Get the post that the comment belongs to.
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

> [!WARNING]
> Znaczniki czasu modelu rodzica zostaną zaktualizowane tylko wtedy, gdy model potomny zostanie zaktualizowany za pomocą metody `save` Eloquenta.
