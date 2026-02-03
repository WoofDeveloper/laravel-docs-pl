# Autoryzacja

- [Wprowadzenie](#introduction)
- [Bramki](#gates)
    - [Tworzenie bramek](#writing-gates)
    - [Autoryzowanie akcji](#authorizing-actions-via-gates)
    - [Odpowiedzi bramek](#gate-responses)
    - [Przechwytywanie sprawdzeń bramek](#intercepting-gate-checks)
    - [Autoryzacja inline](#inline-authorization)
- [Tworzenie zasad](#creating-policies)
    - [Generowanie zasad](#generating-policies)
    - [Rejestrowanie zasad](#registering-policies)
- [Pisanie zasad](#writing-policies)
    - [Metody zasad](#policy-methods)
    - [Odpowiedzi zasad](#policy-responses)
    - [Metody bez modeli](#methods-without-models)
    - [Użytkownicy-goście](#guest-users)
    - [Filtry zasad](#policy-filters)
- [Autoryzowanie akcji za pomocą zasad](#authorizing-actions-using-policies)
    - [Poprzez model użytkownika](#via-the-user-model)
    - [Poprzez fasadę Gate](#via-the-gate-facade)
    - [Poprzez middleware](#via-middleware)
    - [Poprzez szablony Blade](#via-blade-templates)
    - [Dostarczanie dodatkowego kontekstu](#supplying-additional-context)
- [Autoryzacja i Inertia](#authorization-and-inertia)

<a name="introduction"></a>
## Wprowadzenie

Oprócz wbudowanych usług [uwierzytelniania](/docs/{{version}}/authentication), Laravel zapewnia również prosty sposób autoryzacji akcji użytkowników na danym zasobie. Na przykład, nawet jeśli użytkownik jest uwierzytelniony, może nie być upoważniony do aktualizacji lub usuwania określonych modeli Eloquent lub rekordów bazy danych zarządzanych przez aplikację. Funkcje autoryzacji Laravel zapewniają łatwy, zorganizowany sposób zarządzania tego typu sprawdzeniami autoryzacji.

Laravel zapewnia dwa podstawowe sposoby autoryzacji akcji: [bramki](#gates) i [zasady](#creating-policies). Pomyśl o bramkach i zasadach jak o trasach i kontrolerach. Bramki zapewniają proste, oparte na zamknięciach podejście do autoryzacji, podczas gdy zasady, podobnie jak kontrolery, grupują logikę wokół konkretnego modelu lub zasobu. W tej dokumentacji najpierw zbadamy bramki, a następnie przyjrzymy się zasadom.

Nie musisz wybierać między wyłącznym używaniem bramek lub wyłącznym używaniem zasad podczas budowania aplikacji. Większość aplikacji najprawdopodobniej będzie zawierała mieszankę bramek i zasad, co jest całkowicie w porządku! Bramki są najbardziej odpowiednie dla akcji, które nie są powiązane z żadnym modelem lub zasobem, takich jak wyświetlanie panelu administratora. Z kolei zasady powinny być używane, gdy chcesz autoryzować akcję dla konkretnego modelu lub zasobu.

<a name="gates"></a>
## Bramki

<a name="writing-gates"></a>
### Tworzenie bramek

> [!WARNING]
> Bramki to świetny sposób na poznanie podstaw funkcji autoryzacji Laravel; jednak podczas budowania solidnych aplikacji Laravel powinieneś rozważyć użycie [zasad](#creating-policies) do organizacji reguł autoryzacji.

Bramki to po prostu zamknięcia, które określają, czy użytkownik jest upoważniony do wykonania danej akcji. Zazwyczaj bramki są definiowane w metodzie `boot` klasy `App\Providers\AppServiceProvider` za pomocą fasady `Gate`. Bramki zawsze otrzymują instancję użytkownika jako pierwszy argument i mogą opcjonalnie otrzymać dodatkowe argumenty, takie jak odpowiedni model Eloquent.

W tym przykładzie zdefiniujemy bramkę, aby określić, czy użytkownik może zaktualizować dany model `App\Models\Post`. Bramka osiągnie to porównując `id` użytkownika z `user_id` użytkownika, który utworzył post:

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Support\Facades\Gate;

/**
 * Bootstrapuj dowolne usługi aplikacji.
 */
public function boot(): void
{
    Gate::define('update-post', function (User $user, Post $post) {
        return $user->id === $post->user_id;
    });
}
```

Podobnie jak kontrolery, bramki mogą być również definiowane za pomocą tablicy zwrotnej klasy:

```php
use App\Policies\PostPolicy;
use Illuminate\Support\Facades\Gate;

/**
 * Bootstrapuj dowolne usługi aplikacji.
 */
public function boot(): void
{
    Gate::define('update-post', [PostPolicy::class, 'update']);
}
```

<a name="authorizing-actions-via-gates"></a>
### Autoryzowanie akcji

Aby autoryzować akcję za pomocą bramek, powinieneś użyć metod `allows` lub `denies` dostarczonych przez fasadę `Gate`. Zauważ, że nie musisz przekazywać aktualnie uwierzytelnionego użytkownika do tych metod. Laravel automatycznie zajmie się przekazaniem użytkownika do zamknięcia bramki. Typowo wywołuje się metody autoryzacji bramek w kontrolerach aplikacji przed wykonaniem akcji wymagającej autoryzacji:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

class PostController extends Controller
{
    /**
     * Update the given post.
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        if (! Gate::allows('update-post', $post)) {
            abort(403);
        }

        // Aktualizuj post...

        return redirect('/posts');
    }
}
```

Jeśli chcesz określić, czy użytkownik inny niż aktualnie uwierzytelniony jest upoważniony do wykonania akcji, możesz użyć metody `forUser` na fasadzie `Gate`:

```php
if (Gate::forUser($user)->allows('update-post', $post)) {
    // Użytkownik może zaktualizować post...
}

if (Gate::forUser($user)->denies('update-post', $post)) {
    // Użytkownik nie może zaktualizować posta...
}
```

Możesz autoryzować wiele akcji naraz za pomocą metod `any` lub `none`:

```php
if (Gate::any(['update-post', 'delete-post'], $post)) {
    // Użytkownik może zaktualizować lub usunąć post...
}

if (Gate::none(['update-post', 'delete-post'], $post)) {
    // Użytkownik nie może zaktualizować ani usunąć posta...
}
```

<a name="authorizing-or-throwing-exceptions"></a>
#### Autoryzacja lub rzucanie wyjątków

Jeśli chcesz spróbować autoryzować akcję i automatycznie rzucić wyjątek `Illuminate\Auth\Access\AuthorizationException`, jeśli użytkownik nie ma uprawnień do wykonania danej akcji, możesz użyć metody `authorize` fasady `Gate`. Instancje `AuthorizationException` są automatycznie konwertowane na odpowiedź HTTP 403 przez Laravel:

```php
Gate::authorize('update-post', $post);

// Akcja jest autoryzowana...
```

<a name="gates-supplying-additional-context"></a>
#### Dostarczanie dodatkowego kontekstu

Metody bramek do autoryzacji uprawnień (`allows`, `denies`, `check`, `any`, `none`, `authorize`, `can`, `cannot`) oraz [dyrektywy Blade](#via-blade-templates) autoryzacji (`@can`, `@cannot`, `@canany`) mogą otrzymać tablicę jako drugi argument. Elementy tej tablicy są przekazywane jako parametry do zamknięcia bramki i mogą być użyte do dodatkowego kontekstu podczas podejmowania decyzji autoryzacyjnych:

```php
use App\Models\Category;
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::define('create-post', function (User $user, Category $category, bool $pinned) {
    if (! $user->canPublishToGroup($category->group)) {
        return false;
    } elseif ($pinned && ! $user->canPinPosts()) {
        return false;
    }

    return true;
});

if (Gate::check('create-post', [$category, $pinned])) {
    // Użytkownik może utworzyć post...
}
```

<a name="gate-responses"></a>
### Odpowiedzi bramek

Do tej pory zbadaliśmy tylko bramki, które zwracają proste wartości logiczne. Jednakże czasami możesz chcieć zwrócić bardziej szczegółową odpowiedź, w tym komunikat o błędzie. Aby to zrobić, możesz zwrócić `Illuminate\Auth\Access\Response` ze swojej bramki:

```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::deny('Musisz być administratorem.');
});
```

Nawet gdy zwracasz odpowiedź autoryzacji ze swojej bramki, metoda `Gate::allows` nadal zwróci prostą wartość logiczną; jednakże możesz użyć metody `Gate::inspect`, aby uzyskać pełną odpowiedź autoryzacji zwróconą przez bramkę:

```php
$response = Gate::inspect('edit-settings');

if ($response->allowed()) {
    // Akcja jest autoryzowana...
} else {
    echo $response->message();
}
```

Podczas używania metody `Gate::authorize`, która rzuca `AuthorizationException`, jeśli akcja nie jest autoryzowana, komunikat o błędzie dostarczony przez odpowiedź autoryzacji zostanie przekazany do odpowiedzi HTTP:

```php
Gate::authorize('edit-settings');

// Akcja jest autoryzowana...
```

<a name="customizing-gate-response-status"></a>
#### Dostosowywanie statusu odpowiedzi HTTP

Gdy akcja jest odrzucona przez bramkę, zwracana jest odpowiedź HTTP `403`; jednakże czasami może być przydatne zwrócenie alternatywnego kodu statusu HTTP. Możesz dostosować kod statusu HTTP zwracany dla nieudanej kontroli autoryzacji za pomocą statycznego konstruktora `denyWithStatus` klasy `Illuminate\Auth\Access\Response`:

```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::denyWithStatus(404);
});
```

Ponieważ ukrywanie zasobów poprzez odpowiedź `404` jest tak powszechnym wzorcem dla aplikacji webowych, dla wygody oferowana jest metoda `denyAsNotFound`:

```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::denyAsNotFound();
});
```

<a name="intercepting-gate-checks"></a>
### Przechwytywanie sprawdzeń bramek

Czasami możesz chcieć przyznać wszystkie uprawnienia określonemu użytkownikowi. Możesz użyć metody `before`, aby zdefiniować zamknięcie, które jest uruchamiane przed wszystkimi innymi sprawdzeniami autoryzacji:

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::before(function (User $user, string $ability) {
    if ($user->isAdministrator()) {
        return true;
    }
});
```

Jeśli zamknięcie `before` zwróci wynik różny od null, ten wynik zostanie uznany za wynik sprawdzenia autoryzacji.

Możesz użyć metody `after`, aby zdefiniować zamknięcie, które zostanie wykonane po wszystkich innych sprawdzeniach autoryzacji:

```php
use App\Models\User;

Gate::after(function (User $user, string $ability, bool|null $result, mixed $arguments) {
    if ($user->isAdministrator()) {
        return true;
    }
});
```

Wartości zwracane przez zamknięcia `after` nie nadpiszą wyniku sprawdzenia autoryzacji, chyba że bramka lub zasada zwróciła `null`.

<a name="inline-authorization"></a>
### Autoryzacja inline

Czasami możesz chcieć określić, czy aktualnie uwierzytelniony użytkownik jest upoważniony do wykonania danej akcji bez pisania dedykowanej bramki odpowiadającej akcji. Laravel pozwala na wykonywanie tego typu "inline" sprawdzeń autoryzacji poprzez metody `Gate::allowIf` i `Gate::denyIf`. Autoryzacja inline nie wykonuje żadnych zdefiniowanych ["before" lub "after" hooków autoryzacji](#intercepting-gate-checks):

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::allowIf(fn (User $user) => $user->isAdministrator());

Gate::denyIf(fn (User $user) => $user->banned());
```

Jeśli akcja nie jest autoryzowana lub jeśli żaden użytkownik nie jest obecnie uwierzytelniony, Laravel automatycznie rzuci wyjątek `Illuminate\Auth\Access\AuthorizationException`. Instancje `AuthorizationException` są automatycznie konwertowane na odpowiedź HTTP 403 przez obsługę wyjątków Laravel.

<a name="creating-policies"></a>
## Tworzenie zasad

<a name="generating-policies"></a>
### Generowanie zasad

Zasady to klasy, które organizują logikę autoryzacji wokół konkretnego modelu lub zasobu. Na przykład, jeśli Twoja aplikacja jest blogiem, możesz mieć model `App\Models\Post` i odpowiadającą mu zasadę `App\Policies\PostPolicy` do autoryzacji akcji użytkowników, takich jak tworzenie lub aktualizacja postów.

Możesz wygenerować zasadę za pomocą komendy Artisan `make:policy`. Wygenerowana zasada zostanie umieszczona w katalogu `app/Policies`. Jeśli ten katalog nie istnieje w Twojej aplikacji, Laravel utworzy go za Ciebie:

```shell
php artisan make:policy PostPolicy
```

Komenda `make:policy` wygeneruje pustą klasę zasad. Jeśli chcesz wygenerować klasę z przykładowymi metodami zasad związanymi z wyświetlaniem, tworzeniem, aktualizowaniem i usuwaniem zasobu, możesz podać opcję `--model` podczas wykonywania komendy:

```shell
php artisan make:policy PostPolicy --model=Post
```

<a name="registering-policies"></a>
### Rejestrowanie zasad

<a name="policy-discovery"></a>
#### Wykrywanie zasad

Domyślnie Laravel automatycznie wykrywa zasady, o ile model i zasady przestrzegają standardowych konwencji nazewnictwa Laravel. Konkretnie, zasady muszą znajdować się w katalogu `Policies` na poziomie lub powyżej katalogu, który zawiera Twoje modele. Na przykład, modele mogą być umieszczone w katalogu `app/Models`, podczas gdy zasady mogą być umieszczone w katalogu `app/Policies`. W tej sytuacji Laravel sprawdzi zasady w `app/Models/Policies`, a następnie w `app/Policies`. Ponadto nazwa zasady musi odpowiadać nazwie modelu i mieć sufiks `Policy`. Tak więc model `User` odpowiadałby klasie zasad `UserPolicy`.

Jeśli chcesz zdefiniować własną logikę wykrywania zasad, możesz zarejestrować niestandardowe wywołanie zwrotne wykrywania zasad za pomocą metody `Gate::guessPolicyNamesUsing`. Zazwyczaj ta metoda powinna być wywoływana z metody `boot` `AppServiceProvider` Twojej aplikacji:

```php
use Illuminate\Support\Facades\Gate;

Gate::guessPolicyNamesUsing(function (string $modelClass) {
    // Zwróć nazwę klasy zasad dla danego modelu...
});
```

<a name="manually-registering-policies"></a>
#### Ręczne rejestrowanie zasad

Używając fasady `Gate`, możesz ręcznie zarejestrować zasady i odpowiadające im modele w metodzie `boot` `AppServiceProvider` Twojej aplikacji:

```php
use App\Models\Order;
use App\Policies\OrderPolicy;
use Illuminate\Support\Facades\Gate;

/**
 * Bootstrapuj dowolne usługi aplikacji.
 */
public function boot(): void
{
    Gate::policy(Order::class, OrderPolicy::class);
}
```

Alternatywnie możesz umieścić atrybut `UsePolicy` na klasie modelu, aby poinformować Laravel o odpowiadających zasadach modelu:

```php
<?php

namespace App\Models;

use App\Policies\OrderPolicy;
use Illuminate\Database\Eloquent\Attributes\UsePolicy;
use Illuminate\Database\Eloquent\Model;

#[UsePolicy(OrderPolicy::class)]
class Order extends Model
{
    //
}
```

<a name="writing-policies"></a>
## Pisanie zasad

<a name="policy-methods"></a>
### Metody zasad

Po zarejestrowaniu klasy zasad możesz dodać metody dla każdej akcji, którą autoryzuje. Na przykład, zdefiniujmy metodę `update` w naszej `PostPolicy`, która określa, czy dany użytkownik `App\Models\User` może zaktualizować daną instancję `App\Models\Post`.

Metoda `update` otrzyma instancję `User` i `Post` jako argumenty i powinna zwrócić `true` lub `false` wskazując, czy użytkownik jest upoważniony do aktualizacji danego `Post`. Tak więc w tym przykładzie zweryfikujemy, czy `id` użytkownika pasuje do `user_id` w poście:

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * Określ, czy dany post może zostać zaktualizowany przez użytkownika.
     */
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

Możesz kontynuować definiowanie dodatkowych metod w zasadach w zależności od potrzeb dla różnych akcji, które autoryzuje. Na przykład możesz zdefiniować metody `view` lub `delete`, aby autoryzować różne akcje związane z `Post`, ale pamiętaj, że możesz nadać metodom zasad dowolne nazwy, jakie lubisz.

Jeśli użyłeś opcji `--model` podczas generowania zasad przez konsolę Artisan, będzie już zawierać metody dla akcji `viewAny`, `view`, `create`, `update`, `delete`, `restore` i `forceDelete`.

> [!NOTE]
> Wszystkie zasady są rozwiązywane przez Laravel [kontener usług](/docs/{{version}}/container), pozwalając na wskazanie typu dowolnych potrzebnych zależności w konstruktorze zasad, aby były one automatycznie wstrzykiwane.

<a name="policy-responses"></a>
### Odpowiedzi zasad

Do tej pory zbadaliśmy tylko metody zasad, które zwracają proste wartości logiczne. Jednakże czasami możesz chcieć zwrócić bardziej szczegółową odpowiedź, w tym komunikat o błędzie. Aby to zrobić, możesz zwrócić instancję `Illuminate\Auth\Access\Response` ze swojej metody zasad:

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * Określ, czy dany post może zostać zaktualizowany przez użytkownika.
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::deny('Nie jesteś właścicielem tego posta.');
}
```

Podczas zwracania odpowiedzi autoryzacji ze swoich zasad, metoda `Gate::allows` nadal zwróci prostą wartość logiczną; jednakże możesz użyć metody `Gate::inspect`, aby uzyskać pełną odpowiedź autoryzacji zwróconą przez bramkę:

```php
use Illuminate\Support\Facades\Gate;

$response = Gate::inspect('update', $post);

if ($response->allowed()) {
    // Akcja jest autoryzowana...
} else {
    echo $response->message();
}
```

Podczas używania metody `Gate::authorize`, która rzuca `AuthorizationException`, jeśli akcja nie jest autoryzowana, komunikat o błędzie dostarczony przez odpowiedź autoryzacji zostanie przekazany do odpowiedzi HTTP:

```php
Gate::authorize('update', $post);

// Akcja jest autoryzowana...
```

<a name="customizing-policy-response-status"></a>
#### Dostosowywanie statusu odpowiedzi HTTP

Gdy akcja jest odrzucana przez metodę zasad, zwracana jest odpowiedź HTTP `403`; jednakże czasami może być przydatne zwrócenie alternatywnego kodu statusu HTTP. Możesz dostosować kod statusu HTTP zwracany dla nieudanej kontroli autoryzacji za pomocą statycznego konstruktora `denyWithStatus` klasy `Illuminate\Auth\Access\Response`:

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * Określ, czy dany post może zostać zaktualizowany przez użytkownika.
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::denyWithStatus(404);
}
```

Ponieważ ukrywanie zasobów poprzez odpowiedź `404` jest tak powszechnym wzorcem dla aplikacji webowych, dla wygody oferowana jest metoda `denyAsNotFound`:

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * Określ, czy dany post może być zaktualizowany przez użytkownika.
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::denyAsNotFound();
}
```

<a name="methods-without-models"></a>
### Metody bez modeli

Niektóre metody zasad otrzymują tylko instancję aktualnie uwierzytelnionego użytkownika. Ta sytuacja jest najczęstsza podczas autoryzacji akcji `create`. Na przykład, jeśli tworzysz bloga, możesz chcieć określić, czy użytkownik jest upoważniony do tworzenia jakichkolwiek postów. W takich sytuacjach Twoja metoda zasad powinna oczekiwać tylko instancji użytkownika:

```php
/**
 * Określ, czy dany użytkownik może tworzyć posty.
 */
public function create(User $user): bool
{
    return $user->role == 'writer';
}
```

<a name="guest-users"></a>
### Użytkownicy-goście

Domyślnie wszystkie bramki i zasady automatycznie zwracają `false`, jeśli przychodzące żądanie HTTP nie zostało zainicjowane przez uwierzytelnionego użytkownika. Jednakże możesz pozwolić tym sprawdzeniom autoryzacji przejść do Twoich bramek i zasad, deklarując "opcjonalny" typ lub podając domyślną wartość `null` dla definicji argumentu użytkownika:

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * Określ, czy dany post może być zaktualizowany przez użytkownika.
     */
    public function update(?User $user, Post $post): bool
    {
        return $user?->id === $post->user_id;
    }
}
```

<a name="policy-filters"></a>
### Filtry zasad

Dla niektórych użytkowników możesz chcieć autoryzować wszystkie akcje w danej zasadzie. Aby to osiągnąć, zdefiniuj metodę `before` w zasadzie. Metoda `before` zostanie wykonana przed jakimikolwiek innymi metodami w zasadzie, dając Ci możliwość autoryzacji akcji przed faktycznym wywołaniem zamierzonej metody zasad. Ta funkcja jest najczęściej używana do autoryzacji administratorów aplikacji do wykonywania dowolnych akcji:

```php
use App\Models\User;

/**
 * Wykonaj sprawdzenia przed autoryzacją.
 */
public function before(User $user, string $ability): bool|null
{
    if ($user->isAdministrator()) {
        return true;
    }

    return null;
}
```

Jeśli chcesz odrzucić wszystkie sprawdzenia autoryzacji dla określonego typu użytkownika, możesz zwrócić `false` z metody `before`. Jeśli zostanie zwrócony `null`, sprawdzenie autoryzacji przejdzie do metody zasad.

> [!WARNING]
> Metoda `before` klasy zasad nie zostanie wywołana, jeśli klasa nie zawiera metody o nazwie odpowiadającej nazwie sprawdzanej zdolności.

<a name="authorizing-actions-using-policies"></a>
## Autoryzowanie akcji za pomocą zasad

<a name="via-the-user-model"></a>
### Poprzez model użytkownika

Model `App\Models\User`, który jest dołączony do Twojej aplikacji Laravel, zawiera dwie pomocne metody do autoryzacji akcji: `can` i `cannot`. Metody `can` i `cannot` otrzymują nazwę akcji, którą chcesz autoryzować, oraz odpowiedni model. Na przykład określmy, czy użytkownik jest upoważniony do aktualizacji danego modelu `App\Models\Post`. Zazwyczaj będzie to wykonywane w metodzie kontrolera:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * Update the given post.
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        if ($request->user()->cannot('update', $post)) {
            abort(403);
        }

        // Aktualizuj post...

        return redirect('/posts');
    }
}
```

Jeśli [zasada jest zarejestrowana](#registering-policies) dla danego modelu, metoda `can` automatycznie wywoła odpowiednią zasadę i zwróci wynik logiczny. Jeśli żadna zasada nie jest zarejestrowana dla modelu, metoda `can` spróbuje wywołać bramkę opartą na zamknięciu odpowiadającą danej nazwie akcji.

<a name="user-model-actions-that-dont-require-models"></a>
#### Akcje, które nie wymagają modeli

Pamiętaj, że niektóre akcje mogą odpowiadać metodom zasad takim jak `create`, które nie wymagają instancji modelu. W takich sytuacjach możesz przekazać nazwę klasy do metody `can`. Nazwa klasy zostanie użyta do określenia, której zasady użyć podczas autoryzacji akcji:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * Create a post.
     */
    public function store(Request $request): RedirectResponse
    {
        if ($request->user()->cannot('create', Post::class)) {
            abort(403);
        }

        // Utwórz post...

        return redirect('/posts');
    }
}
```

<a name="via-the-gate-facade"></a>
### Poprzez fasadę `Gate`

Oprócz pomocnych metod dostarczonych modelowi `App\Models\User`, zawsze możesz autoryzować akcje poprzez metodę `authorize` fasady `Gate`.

Podobnie jak metoda `can`, ta metoda przyjmuje nazwę akcji, którą chcesz autoryzować, oraz odpowiedni model. Jeśli akcja nie jest autoryzowana, metoda `authorize` rzuci wyjątek `Illuminate\Auth\Access\AuthorizationException`, który obsługa wyjątków Laravel automatycznie przekonwertuje na odpowiedź HTTP z kodem statusu 403:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

class PostController extends Controller
{
    /**
     * Aktualizuj dany post na blogu.
     *
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        Gate::authorize('update', $post);

        // Obecny użytkownik może zaktualizować post na blogu...

        return redirect('/posts');
    }
}
```

<a name="controller-actions-that-dont-require-models"></a>
#### Akcje, które nie wymagają modeli

Jak wcześniej omówiono, niektóre metody zasad takie jak `create` nie wymagają instancji modelu. W takich sytuacjach powinieneś przekazać nazwę klasy do metody `authorize`. Nazwa klasy zostanie użyta do określenia, której zasady użyć podczas autoryzacji akcji:

```php
use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

/**
 * Utwórz nowy post na blogu.
 *
 * @throws \Illuminate\Auth\Access\AuthorizationException
 */
public function create(Request $request): RedirectResponse
{
    Gate::authorize('create', Post::class);

    // Obecny użytkownik może tworzyć posty na blogu...

    return redirect('/posts');
}
```

<a name="via-middleware"></a>
### Poprzez middleware

Laravel zawiera middleware, który może autoryzować akcje jeszcze zanim przychodzące żądanie dotrze do Twoich tras lub kontrolerów. Domyślnie middleware `Illuminate\Auth\Middleware\Authorize` może być dołączony do trasy za pomocą [aliasu middleware](/docs/{{version}}/middleware#middleware-aliases) `can`, który jest automatycznie rejestrowany przez Laravel. Przyjrzyjmy się przykładowi użycia middleware `can` do autoryzacji, że użytkownik może zaktualizować post:

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // Obecny użytkownik może zaktualizować post...
})->middleware('can:update,post');
```

W tym przykładzie przekazujemy middleware `can` dwa argumenty. Pierwszy to nazwa akcji, którą chcemy autoryzować, a drugi to parametr trasy, który chcemy przekazać do metody zasad. W tym przypadku, ponieważ używamy [niejawnego wiązania modelu](/docs/{{version}}/routing#implicit-binding), model `App\Models\Post` zostanie przekazany do metody zasad. Jeśli użytkownik nie jest upoważniony do wykonania danej akcji, odpowiedź HTTP z kodem statusu 403 zostanie zwrócona przez middleware.

Dla wygody możesz także dołączyć middleware `can` do swojej trasy za pomocą metody `can`:

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // Obecny użytkownik może zaktualizować post...
})->can('update', 'post');
```

<a name="middleware-actions-that-dont-require-models"></a>
#### Akcje, które nie wymagają modeli

Ponownie, niektóre metody zasad takie jak `create` nie wymagają instancji modelu. W takich sytuacjach możesz przekazać nazwę klasy do middleware. Nazwa klasy zostanie użyta do określenia, której zasady użyć podczas autoryzacji akcji:

```php
Route::post('/post', function () {
    // Obecny użytkownik może tworzyć posty...
})->middleware('can:create,App\Models\Post');
```

Określanie całej nazwy klasy w definicji middleware jako ciąg może stać się uciążliwe. Z tego powodu możesz wybrać dołączenie middleware `can` do swojej trasy za pomocą metody `can`:

```php
use App\Models\Post;

Route::post('/post', function () {
    // Obecny użytkownik może tworzyć posty...
})->can('create', Post::class);
```

<a name="via-blade-templates"></a>
### Poprzez szablony Blade

Podczas pisania szablonów Blade możesz chcieć wyświetlić część strony tylko wtedy, gdy użytkownik jest upoważniony do wykonania danej akcji. Na przykład możesz chcieć pokazać formularz aktualizacji posta na blogu tylko wtedy, gdy użytkownik może faktycznie zaktualizować post. W tej sytuacji możesz użyć dyrektyw `@can` i `@cannot`:

```blade
@can('update', $post)
    <!-- Obecny użytkownik może zaktualizować post... -->
@elsecan('create', App\Models\Post::class)
    <!-- Obecny użytkownik może tworzyć nowe posty... -->
@else
    <!-- ... -->
@endcan

@cannot('update', $post)
    <!-- Obecny użytkownik nie może zaktualizować posta... -->
@elsecannot('create', App\Models\Post::class)
    <!-- Obecny użytkownik nie może tworzyć nowych postów... -->
@endcannot
```

Te dyrektywy są wygodnymi skrótami do pisania instrukcji `@if` i `@unless`. Instrukcje `@can` i `@cannot` powyżej są równoważne następującym instrukcjom:

```blade
@if (Auth::user()->can('update', $post))
    <!-- Obecny użytkownik może zaktualizować post... -->
@endif

@unless (Auth::user()->can('update', $post))
    <!-- Obecny użytkownik nie może zaktualizować posta... -->
@endunless
```

Możesz również określić, czy użytkownik jest upoważniony do wykonania dowolnej akcji z danej tablicy akcji. Aby to osiągnąć, użyj dyrektywy `@canany`:

```blade
@canany(['update', 'view', 'delete'], $post)
    <!-- Obecny użytkownik może zaktualizować, wyświetlić lub usunąć post... -->
@elsecanany(['create'], \App\Models\Post::class)
    <!-- Obecny użytkownik może utworzyć post... -->
@endcanany
```

<a name="blade-actions-that-dont-require-models"></a>
#### Akcje, które nie wymagają modeli

Podobnie jak większość innych metod autoryzacji, możesz przekazać nazwę klasy do dyrektyw `@can` i `@cannot`, jeśli akcja nie wymaga instancji modelu:

```blade
@can('create', App\Models\Post::class)
    <!-- Obecny użytkownik może tworzyć posty... -->
@endcan

@cannot('create', App\Models\Post::class)
    <!-- Obecny użytkownik nie może tworzyć postów... -->
@endcannot
```

<a name="supplying-additional-context"></a>
### Dostarczanie dodatkowego kontekstu

Podczas autoryzacji akcji za pomocą zasad możesz przekazać tablicę jako drugi argument do różnych funkcji i pomocników autoryzacji. Pierwszy element tablicy zostanie użyty do określenia, która zasada powinna być wywołana, podczas gdy pozostałe elementy tablicy są przekazywane jako parametry do metody zasad i mogą być użyte do dodatkowego kontekstu podczas podejmowania decyzji autoryzacyjnych. Na przykład rozważmy następującą definicję metody `PostPolicy`, która zawiera dodatkowy parametr `$category`:

```php
/**
 * Określ, czy dany post może być zaktualizowany przez użytkownika.
 */
public function update(User $user, Post $post, int $category): bool
{
    return $user->id === $post->user_id &&
           $user->canUpdateCategory($category);
}
```

Podczas próby określenia, czy uwierzytelniony użytkownik może zaktualizować dany post, możemy wywołać tę metodę zasad w następujący sposób:

```php
/**
 * Aktualizuj dany post na blogu.
 *
 * @throws \Illuminate\Auth\Access\AuthorizationException
 */
public function update(Request $request, Post $post): RedirectResponse
{
    Gate::authorize('update', [$post, $request->category]);

    // Obecny użytkownik może zaktualizować post na blogu...

    return redirect('/posts');
}
```

<a name="authorization-and-inertia"></a>
## Autoryzacja i Inertia

Chociaż autoryzacja musi być zawsze obsługiwana na serwerze, często może być wygodne dostarczenie aplikacji frontendowej danych autoryzacji w celu prawidłowego renderowania interfejsu użytkownika aplikacji. Laravel nie definiuje wymaganej konwencji dla udostępniania informacji autoryzacyjnych do frontendu obsługiwanego przez Inertia.

Jednakże jeśli używasz jednego z [zestawów startowych](/docs/{{version}}/starter-kits) Laravel opartych na Inertia, Twoja aplikacja już zawiera middleware `HandleInertiaRequests`. W metodzie `share` tego middleware możesz zwrócić udostępnione dane, które zostaną dostarczone do wszystkich stron Inertia w Twojej aplikacji. Te udostępnione dane mogą służyć jako wygodna lokalizacja do definiowania informacji autoryzacyjnych dla użytkownika:

```php
<?php

namespace App\Http\Middleware;

use App\Models\Post;
use Illuminate\Http\Request;
use Inertia\Middleware;

class HandleInertiaRequests extends Middleware
{
    // ...

    /**
     * Zdefiniuj właściwości, które są domyślnie udostępniane.
     *
     * @return array<string, mixed>
     */
    public function share(Request $request)
    {
        return [
            ...parent::share($request),
            'auth' => [
                'user' => $request->user(),
                'permissions' => [
                    'post' => [
                        'create' => $request->user()->can('create', Post::class),
                    ],
                ],
            ],
        ];
    }
}
```
