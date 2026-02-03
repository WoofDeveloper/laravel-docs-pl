# Eloquent: Fabryki

- [Wprowadzenie](#introduction)
- [Definiowanie Fabryk Modeli](#defining-model-factories)
    - [Generowanie Fabryk](#generating-factories)
    - [Stany Fabryk](#factory-states)
    - [Callbacki Fabryk](#factory-callbacks)
- [Tworzenie Modeli za Pomocą Fabryk](#creating-models-using-factories)
    - [Tworzenie Instancji Modeli](#instantiating-models)
    - [Utrwalanie Modeli](#persisting-models)
    - [Sekwencje](#sequences)
- [Relacje Fabryk](#factory-relationships)
    - [Relacje Jeden do Wielu](#has-many-relationships)
    - [Relacje Należy Do](#belongs-to-relationships)
    - [Relacje Wiele do Wielu](#many-to-many-relationships)
    - [Relacje Polimorficzne](#polymorphic-relationships)
    - [Definiowanie Relacji w Fabrykach](#defining-relationships-within-factories)
    - [Recykling Istniejącego Modelu dla Relacji](#recycling-an-existing-model-for-relationships)

<a name="introduction"></a>
## Wprowadzenie

Podczas testowania aplikacji lub wypełniania bazy danych, możesz potrzebować wstawić kilka rekordów do bazy danych. Zamiast ręcznego określania wartości każdej kolumny, Laravel pozwala zdefiniować zestaw domyślnych atrybutów dla każdego z Twoich [modeli Eloquent](/docs/{{version}}/eloquent) przy użyciu fabryk modeli.

Aby zobaczyć przykład jak napisać fabrykę, spójrz na plik `database/factories/UserFactory.php` w swojej aplikacji. Ta fabryka jest dołączona do wszystkich nowych aplikacji Laravel i zawiera następującą definicję fabryki:

```php
namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

/**
 * @extends \Illuminate\Database\Eloquent\Factories\Factory<\App\Models\User>
 */
class UserFactory extends Factory
{
    /**
     * The current password being used by the factory.
     */
    protected static ?string $password;

    /**
     * Define the model's default state.
     *
     * @return array<string, mixed>
     */
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => static::$password ??= Hash::make('password'),
            'remember_token' => Str::random(10),
        ];
    }

    /**
     * Indicate that the model's email address should be unverified.
     */
    public function unverified(): static
    {
        return $this->state(fn (array $attributes) => [
            'email_verified_at' => null,
        ]);
    }
}
```

Jak widzisz, w swojej najbardziej podstawowej formie, fabryki są klasami, które rozszerzają bazową klasę fabryki Laravel i definiują metodę `definition`. Metoda `definition` zwraca domyślny zestaw wartości atrybutów, które powinny być zastosowane podczas tworzenia modelu za pomocą fabryki.

Za pomocą helpera `fake`, fabryki mają dostęp do biblioteki PHP [Faker](https://github.com/FakerPHP/Faker), która pozwala wygodnie generować różnego rodzaju losowe dane do testowania i wypełniania bazy.

> [!NOTE]
> Możesz zmienić ustawienia regionalne Fakera swojej aplikacji, aktualizując opcję `faker_locale` w pliku konfiguracyjnym `config/app.php`.

<a name="defining-model-factories"></a>
## Definiowanie Fabryk Modeli

<a name="generating-factories"></a>
### Generowanie Fabryk

Aby utworzyć fabrykę, wykonaj [polecenie Artisan](/docs/{{version}}/artisan) `make:factory`:

```shell
php artisan make:factory PostFactory
```

Nowa klasa fabryki zostanie umieszczona w katalogu `database/factories`.

<a name="factory-and-model-discovery-conventions"></a>
#### Konwencje Odkrywania Modeli i Fabryk

Po zdefiniowaniu swoich fabryk, możesz użyć statycznej metody `factory` dostarczonej do Twoich modeli przez cechę `Illuminate\Database\Eloquent\Factories\HasFactory`, aby utworzyć instancję fabryki dla tego modelu.

Metoda `factory` cechy `HasFactory` użyje konwencji, aby określić odpowiednią fabrykę dla modelu, do którego przypisana jest cecha. Konkretnie, metoda będzie szukać fabryki w przestrzeni nazw `Database\Factories`, która ma nazwę klasy pasującą do nazwy modelu i jest zakończona sufiksem `Factory`. Jeśli te konwencje nie mają zastosowania do Twojej konkretnej aplikacji lub fabryki, możesz dodać atrybut `UseFactory` do modelu, aby ręcznie określić fabrykę modelu:

```php
use Illuminate\Database\Eloquent\Attributes\UseFactory;
use Database\Factories\Administration\FlightFactory;

#[UseFactory(FlightFactory::class)]
class Flight extends Model
{
    // ...
}
```

Alternatywnie, możesz nadpisać metodę `newFactory` w swoim modelu, aby zwrócić instancję odpowiadającej fabryki modelu bezpośrednio:

```php
use Database\Factories\Administration\FlightFactory;

/**
 * Create a new factory instance for the model.
 */
protected static function newFactory()
{
    return FlightFactory::new();
}
```

Następnie zdefiniuj właściwość `model` w odpowiedniej fabryce:

```php
use App\Administration\Flight;
use Illuminate\Database\Eloquent\Factories\Factory;

class FlightFactory extends Factory
{
    /**
     * The name of the factory's corresponding model.
     *
     * @var class-string<\Illuminate\Database\Eloquent\Model>
     */
    protected $model = Flight::class;
}
```

<a name="factory-states"></a>
### Stany Fabryk

Metody manipulacji stanem pozwalają zdefiniować dyskretne modyfikacje, które można zastosować do fabryk modeli w dowolnej kombinacji. Na przykład, Twoja fabryka `Database\Factories\UserFactory` może zawierać metodę stanu `suspended`, która modyfikuje jedną z jej domyślnych wartości atrybutów.

Metody transformacji stanu zazwyczaj wywołują metodę `state` dostarczoną przez bazową fabrykę Laravel. Metoda `state` przyjmuje closure, które otrzyma tablicę surowych atrybutów zdefiniowanych dla fabryki i powinno zwrócić tablicę atrybutów do modyfikacji:

```php
use Illuminate\Database\Eloquent\Factories\Factory;

/**
 * Indicate that the user is suspended.
 */
public function suspended(): Factory
{
    return $this->state(function (array $attributes) {
        return [
            'account_status' => 'suspended',
        ];
    });
}
```

<a name="trashed-state"></a>
#### Stan "Usunięty"

Jeśli Twój model Eloquent może być [usunięty miękko](/docs/{{version}}/eloquent#soft-deleting), możesz wywołać wbudowaną metodę stanu `trashed`, aby wskazać, że utworzony model powinien już być "usunięty miękko". Nie musisz ręcznie definiować stanu `trashed`, ponieważ jest on automatycznie dostępny dla wszystkich fabryk:

```php
use App\Models\User;

$user = User::factory()->trashed()->create();
```

<a name="factory-callbacks"></a>
### Callbacki Fabryk

Callbacki fabryk są rejestrowane za pomocą metod `afterMaking` i `afterCreating` i pozwalają wykonać dodatkowe zadania po utworzeniu lub wykonaniu modelu. Powinieneś zarejestrować te callbacki definiując metodę `configure` w swojej fabryce. Ta metoda zostanie automatycznie wywołana przez Laravel, gdy fabryka jest instancjonowana:

```php
namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

class UserFactory extends Factory
{
    /**
     * Configure the model factory.
     */
    public function configure(): static
    {
        return $this->afterMaking(function (User $user) {
            // ...
        })->afterCreating(function (User $user) {
            // ...
        });
    }

    // ...
}
```

Możesz również zarejestrować callbacki fabryk w metodach stanu, aby wykonać dodatkowe zadania specyficzne dla danego stanu:

```php
use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

/**
 * Indicate that the user is suspended.
 */
public function suspended(): Factory
{
    return $this->state(function (array $attributes) {
        return [
            'account_status' => 'suspended',
        ];
    })->afterMaking(function (User $user) {
        // ...
    })->afterCreating(function (User $user) {
        // ...
    });
}
```

<a name="creating-models-using-factories"></a>
## Tworzenie Modeli za Pomocą Fabryk

<a name="instantiating-models"></a>
### Tworzenie Instancji Modeli

Po zdefiniowaniu swoich fabryk, możesz użyć statycznej metody `factory` dostarczonej do Twoich modeli przez cechę `Illuminate\Database\Eloquent\Factories\HasFactory`, aby utworzyć instancję fabryki dla tego modelu. Przyjrzyjmy się kilku przykładom tworzenia modeli. Po pierwsze, użyjemy metody `make`, aby utworzyć modele bez zapisywania ich do bazy danych:

```php
use App\Models\User;

$user = User::factory()->make();
```

Możesz utworzyć kolekcję wielu modeli za pomocą metody `count`:

```php
$users = User::factory()->count(3)->make();
```

<a name="applying-states"></a>
#### Stosowanie Stanów

Możesz również zastosować dowolne [stany](#factory-states) do modeli. Jeśli chcesz zastosować wiele transformacji stanu do modeli, możesz po prostu wywołać metody transformacji stanu bezpośrednio:

```php
$users = User::factory()->count(5)->suspended()->make();
```

<a name="overriding-attributes"></a>
#### Nadpisywanie Atrybutów

Jeśli chcesz nadpisać niektóre z domyślnych wartości swoich modeli, możesz przekazać tablicę wartości do metody `make`. Tylko określone atrybuty zostaną zastąpione, podczas gdy reszta atrybutów pozostanie ustawiona na ich domyślne wartości określone przez fabrykę:

```php
$user = User::factory()->make([
    'name' => 'Abigail Otwell',
]);
```

Alternatywnie, metoda `state` może być wywołana bezpośrednio na fabryce, aby wykonać inline transformację stanu:

```php
$user = User::factory()->state([
    'name' => 'Abigail Otwell',
])->make();
```

> [!NOTE]
> [Ochrona przed masowym przypisaniem](/docs/{{version}}/eloquent#mass-assignment) jest automatycznie wyłączona podczas tworzenia modeli za pomocą fabryk.

<a name="persisting-models"></a>
### Utrwalanie Modeli

Metoda `create` tworzy instancje modeli i zapisuje je do bazy danych za pomocą metody `save` Eloquent:

```php
use App\Models\User;

// Create a single App\Models\User instance...
$user = User::factory()->create();

// Create three App\Models\User instances...
$users = User::factory()->count(3)->create();
```

Możesz nadpisać domyślne atrybuty modelu fabryki, przekazując tablicę atrybutów do metody `create`:

```php
$user = User::factory()->create([
    'name' => 'Abigail',
]);
```

<a name="sequences"></a>
### Sekwencje

Czasami możesz chcieć zmieniać wartość danego atrybutu modelu dla każdego utworzonego modelu. Możesz to osiągnąć, definiując transformację stanu jako sekwencję. Na przykład, możesz chcieć zmieniać wartość kolumny `admin` między `Y` i `N` dla każdego utworzonego użytkownika:

```php
use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Sequence;

$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        ['admin' => 'Y'],
        ['admin' => 'N'],
    ))
    ->create();
```

W tym przykładzie pięciu użytkowników zostanie utworzonych z wartością `admin` równą `Y`, a pięciu użytkowników zostanie utworzonych z wartością `admin` równą `N`.

W razie potrzeby możesz uwzględnić closure jako wartość sekwencji. Closure będzie wywoływane za każdym razem, gdy sekwencja potrzebuje nowej wartości:

```php
use Illuminate\Database\Eloquent\Factories\Sequence;

$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        fn (Sequence $sequence) => ['role' => UserRoles::all()->random()],
    ))
    ->create();
```

W closure sekwencji możesz uzyskać dostęp do właściwości `$index` na instancji sekwencji, która jest wstrzykiwana do closure. Właściwość `$index` zawiera liczbę iteracji przez sekwencję, które miały miejsce do tej pory:

```php
$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        fn (Sequence $sequence) => ['name' => 'Name '.$sequence->index],
    ))
    ->create();
```

Dla wygody sekwencje mogą być również stosowane za pomocą metody `sequence`, która po prostu wywołuje metodę `state` wewnętrznie. Metoda `sequence` przyjmuje closure lub tablice sekwencjonowanych atrybutów:

```php
$users = User::factory()
    ->count(2)
    ->sequence(
        ['name' => 'First User'],
        ['name' => 'Second User'],
    )
    ->create();
```

<a name="factory-relationships"></a>
## Relacje Fabryk

<a name="has-many-relationships"></a>
### Relacje Jeden do Wielu

Następnie, zbadajmy budowanie relacji modeli Eloquent za pomocą płynnych metod fabryk Laravel. Po pierwsze, załóżmy, że nasza aplikacja ma model `App\Models\User` i model `App\Models\Post`. Załóżmy również, że model `User` definiuje relację `hasMany` z `Post`. Możemy utworzyć użytkownika, który ma trzy posty, używając metody `has` dostarczonej przez fabryki Laravel. Metoda `has` przyjmuje instancję fabryki:

```php
use App\Models\Post;
use App\Models\User;

$user = User::factory()
    ->has(Post::factory()->count(3))
    ->create();
```

Według konwencji, przekazując model `Post` do metody `has`, Laravel założy, że model `User` musi mieć metodę `posts`, która definiuje relację. Jeśli to konieczne, możesz jawnie określić nazwę relacji, którą chcesz manipulować:

```php
$user = User::factory()
    ->has(Post::factory()->count(3), 'posts')
    ->create();
```

Oczywiście możesz wykonywać manipulacje stanem na powiązanych modelach. Ponadto możesz przekazać transformację stanu opartą na closure, jeśli zmiana stanu wymaga dostępu do modelu nadrzędnego:

```php
$user = User::factory()
    ->has(
        Post::factory()
            ->count(3)
            ->state(function (array $attributes, User $user) {
                return ['user_type' => $user->type];
            })
        )
    ->create();
```

<a name="has-many-relationships-using-magic-methods"></a>
#### Używanie Magicznych Metod

Dla wygody możesz użyć magicznych metod relacji fabryk Laravel do budowania relacji. Na przykład, poniższy przykład użyje konwencji, aby określić, że powiązane modele powinny być tworzone za pomocą metody relacji `posts` w modelu `User`:

```php
$user = User::factory()
    ->hasPosts(3)
    ->create();
```

Używając magicznych metod do tworzenia relacji fabryk, możesz przekazać tablicę atrybutów do nadpisania w powiązanych modelach:

```php
$user = User::factory()
    ->hasPosts(3, [
        'published' => false,
    ])
    ->create();
```

Możesz zapewnić transformację stanu opartą na closure, jeśli zmiana stanu wymaga dostępu do modelu nadrzędnego:

```php
$user = User::factory()
    ->hasPosts(3, function (array $attributes, User $user) {
        return ['user_type' => $user->type];
    })
    ->create();
```

<a name="belongs-to-relationships"></a>
### Relacje Należy Do

Teraz, gdy zbadaliśmy, jak budować relacje "jeden do wielu" za pomocą fabryk, zbadajmy odwrotność relacji. Metoda `for` może być użyta do zdefiniowania modelu nadrzędnego, do którego należą modele utworzone przez fabrykę. Na przykład, możemy utworzyć trzy instancje modelu `App\Models\Post`, które należą do jednego użytkownika:

```php
use App\Models\Post;
use App\Models\User;

$posts = Post::factory()
    ->count(3)
    ->for(User::factory()->state([
        'name' => 'Jessica Archer',
    ]))
    ->create();
```

Jeśli masz już instancję modelu nadrzędnego, która powinna być skojarzona z modelami, które tworzysz, możesz przekazać instancję modelu do metody `for`:

```php
$user = User::factory()->create();

$posts = Post::factory()
    ->count(3)
    ->for($user)
    ->create();
```

<a name="belongs-to-relationships-using-magic-methods"></a>
#### Używanie Magicznych Metod

Dla wygody możesz użyć magicznych metod relacji fabryk Laravel do definiowania relacji "należy do". Na przykład, poniższy przykład użyje konwencji, aby określić, że trzy posty powinny należeć do relacji `user` w modelu `Post`:

```php
$posts = Post::factory()
    ->count(3)
    ->forUser([
        'name' => 'Jessica Archer',
    ])
    ->create();
```

<a name="many-to-many-relationships"></a>
### Relacje Wiele do Wielu

Podobnie jak [relacje jeden do wielu](#has-many-relationships), relacje "wiele do wielu" mogą być tworzone za pomocą metody `has`:

```php
use App\Models\Role;
use App\Models\User;

$user = User::factory()
    ->has(Role::factory()->count(3))
    ->create();
```

<a name="pivot-table-attributes"></a>
#### Atrybuty Tabeli Pivot

Jeśli musisz zdefiniować atrybuty, które powinny być ustawione w tabeli pivot/pośredniczącej łączącej modele, możesz użyć metody `hasAttached`. Ta metoda przyjmuje tablicę nazw i wartości atrybutów tabeli pivot jako drugi argument:

```php
use App\Models\Role;
use App\Models\User;

$user = User::factory()
    ->hasAttached(
        Role::factory()->count(3),
        ['active' => true]
    )
    ->create();
```

Możesz zapewnić transformację stanu opartą na closure, jeśli zmiana stanu wymaga dostępu do powiązanego modelu:

```php
$user = User::factory()
    ->hasAttached(
        Role::factory()
            ->count(3)
            ->state(function (array $attributes, User $user) {
                return ['name' => $user->name.' Role'];
            }),
        ['active' => true]
    )
    ->create();
```

Jeśli masz już instancje modeli, które chcesz dołączyć do modeli, które tworzysz, możesz przekazać instancje modeli do metody `hasAttached`. W tym przykładzie te same trzy role zostaną dołączone do wszystkich trzech użytkowników:

```php
$roles = Role::factory()->count(3)->create();

$users = User::factory()
    ->count(3)
    ->hasAttached($roles, ['active' => true])
    ->create();
```

<a name="many-to-many-relationships-using-magic-methods"></a>
#### Używanie Magicznych Metod

Dla wygody możesz użyć magicznych metod relacji fabryk Laravel do definiowania relacji wiele do wielu. Na przykład, poniższy przykład użyje konwencji, aby określić, że powiązane modele powinny być tworzone za pomocą metody relacji `roles` w modelu `User`:

```php
$user = User::factory()
    ->hasRoles(1, [
        'name' => 'Editor'
    ])
    ->create();
```

<a name="polymorphic-relationships"></a>
### Relacje Polimorficzne

[Relacje polimorficzne](/docs/{{version}}/eloquent-relationships#polymorphic-relationships) mogą być również tworzone za pomocą fabryk. Polimorficzne relacje "morph many" są tworzone w ten sam sposób, co typowe relacje "jeden do wielu". Na przykład, jeśli model `App\Models\Post` ma relację `morphMany` z modelem `App\Models\Comment`:

```php
use App\Models\Post;

$post = Post::factory()->hasComments(3)->create();
```

<a name="morph-to-relationships"></a>
#### Relacje Morph To

Magiczne metody nie mogą być użyte do tworzenia relacji `morphTo`. Zamiast tego metoda `for` musi być użyta bezpośrednio, a nazwa relacji musi być jawnie podana. Na przykład, wyobraź sobie, że model `Comment` ma metodę `commentable`, która definiuje relację `morphTo`. W tej sytuacji możemy utworzyć trzy komentarze, które należą do jednego posta, używając metody `for` bezpośrednio:

```php
$comments = Comment::factory()->count(3)->for(
    Post::factory(), 'commentable'
)->create();
```

<a name="polymorphic-many-to-many-relationships"></a>
#### Polimorficzne Relacje Wiele do Wielu

Polimorficzne relacje "wiele do wielu" (`morphToMany` / `morphedByMany`) mogą być tworzone tak samo jak niepolimorficzne relacje "wiele do wielu":

```php
use App\Models\Tag;
use App\Models\Video;

$video = Video::factory()
    ->hasAttached(
        Tag::factory()->count(3),
        ['public' => true]
    )
    ->create();
```

Oczywiście magiczna metoda `has` może być również użyta do tworzenia polimorficznych relacji "wiele do wielu":

```php
$video = Video::factory()
    ->hasTags(3, ['public' => true])
    ->create();
```

<a name="defining-relationships-within-factories"></a>
### Definiowanie Relacji w Fabrykach

Aby zdefiniować relację w swojej fabryce modelu, zazwyczaj przypisujesz nową instancję fabryki do klucza obcego relacji. Jest to zazwyczaj robione dla relacji "odwrotnych", takich jak `belongsTo` i `morphTo`. Na przykład, jeśli chcesz utworzyć nowego użytkownika podczas tworzenia posta, możesz zrobić następująco:

```php
use App\Models\User;

/**
 * Define the model's default state.
 *
 * @return array<string, mixed>
 */
public function definition(): array
{
    return [
        'user_id' => User::factory(),
        'title' => fake()->title(),
        'content' => fake()->paragraph(),
    ];
}
```

Jeśli kolumny relacji zależą od fabryki, która ją definiuje, możesz przypisać closure do atrybutu. Closure otrzyma tablicę ocenionych atrybutów modelu:

```php
/**
 * Define the model's default state.
 *
 * @return array<string, mixed>
 */
public function definition(): array
{
    return [
        'user_id' => User::factory(),
        'user_type' => function (array $attributes) {
            return User::find($attributes['user_id'])->type;
        },
        'title' => fake()->title(),
        'content' => fake()->paragraph(),
    ];
}
```

<a name="recycling-an-existing-model-for-relationships"></a>
### Recykling Istniejącego Modelu dla Relacji

Jeśli masz modele, które dzielą wspólną relację z innym modelem, możesz użyć metody `recycle`, aby upewnić się, że jedna instancja powiązanego modelu jest ponownie używana dla wszystkich relacji utworzonych przez fabrykę.

Na przykład, wyobraź sobie, że masz modele `Airline`, `Flight` i `Ticket`, gdzie bilet należy do linii lotniczej i lotu, a lot również należy do linii lotniczej. Podczas tworzenia biletów prawdopodobnie będziesz chciał mieć tę samą linię lotniczą zarówno dla biletu, jak i dla lotu, więc możesz przekazać instancję linii lotniczej do metody `recycle`:

```php
Ticket::factory()
    ->recycle(Airline::factory()->create())
    ->create();
```

Możesz uznać metodę `recycle` za szczególnie użyteczną, jeśli masz modele należące do wspólnego użytkownika lub zespołu.

Metoda `recycle` przyjmuje również kolekcję istniejących modeli. Gdy kolekcja jest przekazana do metody `recycle`, losowy model z kolekcji zostanie wybrany, gdy fabryka potrzebuje modelu tego typu:

```php
Ticket::factory()
    ->recycle($airlines)
    ->create();
```
