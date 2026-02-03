# Laravel Pennant

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
- [Konfiguracja](#configuration)
- [Definiowanie Funkcji](#defining-features)
    - [Funkcje Oparte na Klasach](#class-based-features)
- [Sprawdzanie Funkcji](#checking-features)
    - [Warunkowe Wykonywanie](#conditional-execution)
    - [Trait `HasFeatures`](#the-has-features-trait)
    - [Dyrektywa Blade](#blade-directive)
    - [Middleware](#middleware)
    - [Przechwytywanie Sprawdzeń Funkcji](#intercepting-feature-checks)
    - [Cache w Pamięci](#in-memory-cache)
- [Zakres](#scope)
    - [Określanie Zakresu](#specifying-the-scope)
    - [Domyślny Zakres](#default-scope)
    - [Zakres Nullable](#nullable-scope)
    - [Identyfikowanie Zakresu](#identifying-scope)
    - [Serializacja Zakresu](#serializing-scope)
- [Bogate Wartości Funkcji](#rich-feature-values)
- [Pobieranie Wielu Funkcji](#retrieving-multiple-features)
- [Eager Loading](#eager-loading)
- [Aktualizacja Wartości](#updating-values)
    - [Aktualizacje Zbiorcze](#bulk-updates)
    - [Czyszczenie Funkcji](#purging-features)
- [Testowanie](#testing)
- [Dodawanie Niestandardowych Sterowników Pennant](#adding-custom-pennant-drivers)
    - [Implementacja Sterownika](#implementing-the-driver)
    - [Rejestracja Sterownika](#registering-the-driver)
    - [Definiowanie Funkcji Zewnętrznie](#defining-features-externally)
- [Zdarzenia](#events)

<a name="introduction"></a>
## Wprowadzenie

[Laravel Pennant](https://github.com/laravel/pennant) to prosty i lekki pakiet flag funkcji - bez zbędnych dodatków. Flagi funkcji umożliwiają stopniowe wdrażanie nowych funkcji aplikacji z pewnością, testowanie A/B nowych projektów interfejsu, uzupełnienie strategii rozwoju opartej na trunk-based development i wiele więcej.

<a name="installation"></a>
## Instalacja

Najpierw zainstaluj Pennant w swoim projekcie za pomocą menedżera pakietów Composer:

```shell
composer require laravel/pennant
```

Następnie opublikuj pliki konfiguracji i migracji Pennant używając polecenia Artisan `vendor:publish`:

```shell
php artisan vendor:publish --provider="Laravel\Pennant\PennantServiceProvider"
```

Na koniec uruchom migracje bazy danych aplikacji. To utworzy tabelę `features`, której Pennant używa do zasilania swojego sterownika `database`:

```shell
php artisan migrate
```

<a name="configuration"></a>
## Konfiguracja

Po opublikowaniu zasobów Pennant, jego plik konfiguracyjny będzie znajdował się w `config/pennant.php`. Ten plik konfiguracyjny pozwala określić domyślny mechanizm przechowywania, który będzie używany przez Pennant do przechowywania rozwiązanych wartości flag funkcji.

Pennant zawiera wsparcie dla przechowywania rozwiązanych wartości flag funkcji w tablicy w pamięci za pomocą sterownika `array`. Lub Pennant może przechowywać rozwiązane wartości flag funkcji trwale w relacyjnej bazie danych za pomocą sterownika `database`, który jest domyślnym mechanizmem przechowywania używanym przez Pennant.

<a name="defining-features"></a>
## Definiowanie Funkcji

Aby zdefiniować funkcję, możesz użyć metody `define` oferowanej przez fasadę `Feature`. Musisz podać nazwę funkcji oraz domknięcie, które zostanie wywołane w celu rozwiązania początkowej wartości funkcji.

Typowo funkcje są definiowane w dostawcy usług za pomocą fasady `Feature`. Domknięcie otrzyma "zakres" dla sprawdzenia funkcji. Najczęściej zakresem jest aktualnie uwierzytelniony użytkownik. W tym przykładzie zdefiniujemy funkcję do stopniowego wdrażania nowego API dla użytkowników naszej aplikacji:

```php
<?php

namespace App\Providers;

use App\Models\User;
use Illuminate\Support\Lottery;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrapuj wszelkie usługi aplikacji.
     */
    public function boot(): void
    {
        Feature::define('new-api', fn (User $user) => match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        });
    }
}
```

Jak widać, mamy następujące reguły dla naszej funkcji:

- Wszyscy członkowie wewnętrznego zespołu powinni używać nowego API.
- Żaden klient o wysokim ruchu nie powinien używać nowego API.
- W przeciwnym razie funkcja powinna być losowo przypisana użytkownikom z szansą 1 na 100 bycia aktywną.

Pierwszym razem, gdy funkcja `new-api` jest sprawdzana dla danego użytkownika, wynik domknięcia zostanie zapisany przez sterownik przechowywania. Następnym razem, gdy funkcja jest sprawdzana dla tego samego użytkownika, wartość zostanie pobrana z przechowywania i domknięcie nie zostanie wywołane.

Dla wygody, jeśli definicja funkcji zwraca tylko loterię, możesz całkowicie pominąć domknięcie:

    Feature::define('site-redesign', Lottery::odds(1, 1000));

<a name="class-based-features"></a>
### Funkcje Oparte na Klasach

Pennant pozwala również definiować funkcje oparte na klasach. W przeciwieństwie do definicji funkcji opartych na domknięciach, nie ma potrzeby rejestrowania funkcji opartej na klasie w dostawcy usług. Aby utworzyć funkcję opartą na klasie, możesz wywołać polecenie Artisan `pennant:feature`. Domyślnie klasa funkcji zostanie umieszczona w katalogu `app/Features` Twojej aplikacji:

```shell
php artisan pennant:feature NewApi
```

Pisząc klasę funkcji, musisz tylko zdefiniować metodę `resolve`, która zostanie wywołana w celu rozwiązania początkowej wartości funkcji dla danego zakresu. Ponownie, zakresem będzie zazwyczaj aktualnie uwierzytelniony użytkownik:

```php
<?php

namespace App\Features;

use App\Models\User;
use Illuminate\Support\Lottery;

class NewApi
{
    /**
     * Rozwiąż początkową wartość funkcji.
     */
    public function resolve(User $user): mixed
    {
        return match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        };
    }
}
```

Jeśli chcesz ręcznie rozwiązać instancję funkcji opartej na klasie, możesz wywołać metodę `instance` na fasadzie `Feature`:

```php
use Illuminate\Support\Facades\Feature;

$instance = Feature::instance(NewApi::class);
```

> [!NOTE]
> Klasy funkcji są rozwiązywane przez [kontener](/docs/{{version}}/container), więc możesz wstrzykiwać zależności do konstruktora klasy funkcji gdy potrzeba.

#### Dostosowywanie Przechowywanej Nazwy Funkcji

Domyślnie Pennant będzie przechowywał w pełni kwalifikowaną nazwę klasy funkcji. Jeśli chcesz oddzielić przechowywaną nazwę funkcji od wewnętrznej struktury aplikacji, możesz określić właściwość `$name` w klasie funkcji. Wartość tej właściwości zostanie zapisana zamiast nazwy klasy:

```php
<?php

namespace App\Features;

class NewApi
{
    /**
     * Przechowywana nazwa funkcji.
     *
     * @var string
     */
    public $name = 'new-api';

    // ...
}
```

<a name="checking-features"></a>
## Sprawdzanie Funkcji

Aby określić, czy funkcja jest aktywna, możesz użyć metody `active` na fasadzie `Feature`. Domyślnie funkcje są sprawdzane względem aktualnie uwierzytelnionego użytkownika:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     * Wyświetl listę zasobów.
     */
    public function index(Request $request): Response
    {
        return Feature::active('new-api')
            ? $this->resolveNewApiResponse($request)
            : $this->resolveLegacyApiResponse($request);
    }

    // ...
}
```

Choć funkcje są domyślnie sprawdzane względem aktualnie uwierzytelnionego użytkownika, możesz łatwo sprawdzić funkcję względem innego użytkownika lub [zakresu](#scope). Aby to osiągnąć, użyj metody `for` oferowanej przez fasadę `Feature`:

```php
return Feature::for($user)->active('new-api')
    ? $this->resolveNewApiResponse($request)
    : $this->resolveLegacyApiResponse($request);
```

Pennant oferuje również kilka dodatkowych wygodnych metod, które mogą okazać się przydatne przy określaniu, czy funkcja jest aktywna czy nie:

```php
// Określ, czy wszystkie podane funkcje są aktywne...
Feature::allAreActive(['new-api', 'site-redesign']);

// Określ, czy którakolwiek z podanych funkcji jest aktywna...
Feature::someAreActive(['new-api', 'site-redesign']);

// Określ, czy funkcja jest nieaktywna...
Feature::inactive('new-api');

// Określ, czy wszystkie podane funkcje są nieaktywne...
Feature::allAreInactive(['new-api', 'site-redesign']);

// Określ, czy którakolwiek z podanych funkcji jest nieaktywna...
Feature::someAreInactive(['new-api', 'site-redesign']);
```

> [!NOTE]
> Używając Pennant poza kontekstem HTTP, takim jak polecenie Artisan lub zadanie w kolejce, powinieneś zazwyczaj [jawnie określić zakres funkcji](#specifying-the-scope). Alternatywnie możesz zdefiniować [domyślny zakres](#default-scope), który uwzględnia zarówno uwierzytelnione konteksty HTTP, jak i nieuwierzytelnione konteksty.

<a name="checking-class-based-features"></a>
#### Sprawdzanie Funkcji Opartych na Klasach

Dla funkcji opartych na klasach powinieneś podać nazwę klasy podczas sprawdzania funkcji:

```php
<?php

namespace App\Http\Controllers;

use App\Features\NewApi;
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     * Wyświetl listę zasobów.
     */
    public function index(Request $request): Response
    {
        return Feature::active(NewApi::class)
            ? $this->resolveNewApiResponse($request)
            : $this->resolveLegacyApiResponse($request);
    }

    // ...
}
```

<a name="conditional-execution"></a>
### Warunkowe Wykonywanie

Metoda `when` może być użyta do płynnego wykonania danego domknięcia, jeśli funkcja jest aktywna. Dodatkowo może być podane drugie domknięcie, które zostanie wykonane, jeśli funkcja jest nieaktywna:

```php
<?php

namespace App\Http\Controllers;

use App\Features\NewApi;
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     * Wyświetl listę zasobów.
     */
    public function index(Request $request): Response
    {
        return Feature::when(NewApi::class,
            fn () => $this->resolveNewApiResponse($request),
            fn () => $this->resolveLegacyApiResponse($request),
        );
    }

    // ...
}
```

Metoda `unless` służy jako odwrotność metody `when`, wykonując pierwsze domknięcie, jeśli funkcja jest nieaktywna:

```php
return Feature::unless(NewApi::class,
    fn () => $this->resolveLegacyApiResponse($request),
    fn () => $this->resolveNewApiResponse($request),
);
```

<a name="the-has-features-trait"></a>
### Trait `HasFeatures`

Trait `HasFeatures` Pennanta może być dodany do modelu `User` Twojej aplikacji (lub dowolnego innego modelu, który ma funkcje), aby zapewnić płynny i wygodny sposób sprawdzania funkcji bezpośrednio z modelu:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Laravel\Pennant\Concerns\HasFeatures;

class User extends Authenticatable
{
    use HasFeatures;

    // ...
}
```

Po dodaniu traita do modelu możesz łatwo sprawdzać funkcje wywołując metodę `features`:

```php
if ($user->features()->active('new-api')) {
    // ...
}
```

Oczywiście metoda `features` zapewnia dostęp do wielu innych wygodnych metod do interakcji z funkcjami:

```php
// Wartości...
$value = $user->features()->value('purchase-button')
$values = $user->features()->values(['new-api', 'purchase-button']);

// Stan...
$user->features()->active('new-api');
$user->features()->allAreActive(['new-api', 'server-api']);
$user->features()->someAreActive(['new-api', 'server-api']);

$user->features()->inactive('new-api');
$user->features()->allAreInactive(['new-api', 'server-api']);
$user->features()->someAreInactive(['new-api', 'server-api']);

// Warunkowe wykonywanie...
$user->features()->when('new-api',
    fn () => /* ... */,
    fn () => /* ... */,
);

$user->features()->unless('new-api',
    fn () => /* ... */,
    fn () => /* ... */,
);
```

<a name="blade-directive"></a>
### Dyrektywa Blade

Aby sprawdzanie funkcji w Blade było bezproblemowe, Pennant oferuje dyrektywy `@feature` i `@featureany`:

```blade
@feature('site-redesign')
    <!-- 'site-redesign' jest aktywny -->
@else
    <!-- 'site-redesign' jest nieaktywny -->
@endfeature

@featureany(['site-redesign', 'beta'])
    <!-- 'site-redesign' lub `beta` jest aktywny -->
@endfeatureany
```

<a name="middleware"></a>
### Middleware

Pennant zawiera również [middleware](/docs/{{version}}/middleware), które może być użyte do weryfikacji, czy aktualnie uwierzytelniony użytkownik ma dostęp do funkcji, zanim trasa zostanie wywołana. Możesz przypisać middleware do trasy i określić funkcje, które są wymagane do dostępu do trasy. Jeśli którakolwiek z określonych funkcji jest nieaktywna dla aktualnie uwierzytelnionego użytkownika, trasa zwróci odpowiedź HTTP `400 Bad Request`. Wiele funkcji może być przekazanych do statycznej metody `using`.

```php
use Illuminate\Support\Facades\Route;
use Laravel\Pennant\Middleware\EnsureFeaturesAreActive;

Route::get('/api/servers', function () {
    // ...
})->middleware(EnsureFeaturesAreActive::using('new-api', 'servers-api'));
```

<a name="customizing-the-response"></a>
#### Dostosowywanie Odpowiedzi

Jeśli chcesz dostosować odpowiedź zwracaną przez middleware, gdy jedna z wymienionych funkcji jest nieaktywna, możesz użyć metody `whenInactive` udostępnianej przez middleware `EnsureFeaturesAreActive`. Zazwyczaj ta metoda powinna być wywołana w metodzie `boot` jednego z dostawców usług Twojej aplikacji:

```php
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Middleware\EnsureFeaturesAreActive;

/**
 * Bootstrapuj wszelkie usługi aplikacji.
 */
public function boot(): void
{
    EnsureFeaturesAreActive::whenInactive(
        function (Request $request, array $features) {
            return new Response(status: 403);
        }
    );

    // ...
}
```

<a name="intercepting-feature-checks"></a>
### Przechwytywanie Sprawdzeń Funkcji

Czasami może być przydatne wykonanie niektórych sprawdzeń w pamięci przed pobraniem przechowywanej wartości danej funkcji. Wyobraź sobie, że rozwijasz nowe API za flagą funkcji i chcesz mieć możliwość wyłączenia nowego API bez utraty żadnych rozwiązanych wartości funkcji w przechowywaniu. Jeśli zauważysz błąd w nowym API, możesz łatwo wyłączyć je dla wszystkich z wyjątkiem członków wewnętrznego zespołu, naprawić błąd, a następnie ponownie włączyć nowe API dla użytkowników, którzy wcześniej mieli dostęp do funkcji.

Możesz to osiągnąć za pomocą metody `before` [funkcji opartej na klasie](#class-based-features). Gdy jest obecna, metoda `before` jest zawsze uruchamiana w pamięci przed pobraniem wartości z przechowywania. Jeśli z metody zostanie zwrócona wartość inna niż `null`, zostanie ona użyta zamiast przechowywanej wartości funkcji na czas trwania żądania:

```php
<?php

namespace App\Features;

use App\Models\User;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Lottery;

class NewApi
{
    /**
     * Uruchom sprawdzenie zawsze w pamięci przed pobraniem przechowywanej wartości.
     */
    public function before(User $user): mixed
    {
        if (Config::get('features.new-api.disabled')) {
            return $user->isInternalTeamMember();
        }
    }

    /**
     * Rozwiąż początkową wartość funkcji.
     */
    public function resolve(User $user): mixed
    {
        return match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        };
    }
}
```

Możesz również użyć tej funkcji do zaplanowania globalnego wdrożenia funkcji, która wcześniej była za flagą funkcji:

```php
<?php

namespace App\Features;

use Illuminate\Support\Carbon;
use Illuminate\Support\Facades\Config;

class NewApi
{
    /**
     * Uruchom sprawdzenie zawsze w pamięci przed pobraniem przechowywanej wartości.
     */
    public function before(User $user): mixed
    {
        if (Config::get('features.new-api.disabled')) {
            return $user->isInternalTeamMember();
        }

        if (Carbon::parse(Config::get('features.new-api.rollout-date'))->isPast()) {
            return true;
        }
    }

    // ...
}
```

<a name="in-memory-cache"></a>
### Cache w Pamięci

Podczas sprawdzania funkcji Pennant utworzy cache w pamięci z wynikiem. Jeśli używasz sterownika `database`, oznacza to, że ponowne sprawdzenie tej samej flagi funkcji w ramach pojedynczego żądania nie spowoduje dodatkowych zapytań do bazy danych. Zapewnia to również, że funkcja ma spójny wynik przez czas trwania żądania.

Jeśli musisz ręcznie wyczyścić cache w pamięci, możesz użyć metody `flushCache` oferowanej przez fasadę `Feature`:

```php
Feature::flushCache();
```

<a name="scope"></a>
## Zakres

<a name="specifying-the-scope"></a>
### Określanie Zakresu

Jak omówiono, funkcje są zazwyczaj sprawdzane względem aktualnie uwierzytelnionego użytkownika. Jednak może to nie zawsze odpowiadać Twoim potrzebom. Dlatego możliwe jest określenie zakresu, względem którego chcesz sprawdzić daną funkcję, za pomocą metody `for` fasady `Feature`:

```php
return Feature::for($user)->active('new-api')
    ? $this->resolveNewApiResponse($request)
    : $this->resolveLegacyApiResponse($request);
```

Oczywiście zakresy funkcji nie są ograniczone do "użytkowników". Wyobraź sobie, że zbudowałeś nowe doświadczenie rozliczeniowe, które wdrażasz dla całych zespołów, a nie indywidualnych użytkowników. Być może chcesz, aby starsze zespoły miały wolniejsze wdrożenie niż nowsze zespoły. Twoje domknięcie rozwiązywania funkcji może wyglądać mniej więcej tak:

```php
use App\Models\Team;
use Illuminate\Support\Carbon;
use Illuminate\Support\Lottery;
use Laravel\Pennant\Feature;

Feature::define('billing-v2', function (Team $team) {
    if ($team->created_at->isAfter(new Carbon('1st Jan, 2023'))) {
        return true;
    }

    if ($team->created_at->isAfter(new Carbon('1st Jan, 2019'))) {
        return Lottery::odds(1 / 100);
    }

    return Lottery::odds(1 / 1000);
});
```

Zauważysz, że zdefiniowane przez nas domknięcie nie oczekuje `User`, ale zamiast tego oczekuje modelu `Team`. Aby określić, czy ta funkcja jest aktywna dla zespołu użytkownika, powinieneś przekazać zespół do metody `for` oferowanej przez fasadę `Feature`:

```php
if (Feature::for($user->team)->active('billing-v2')) {
    return redirect('/billing/v2');
}

// ...
```

<a name="default-scope"></a>
### Domyślny Zakres

Możliwe jest również dostosowanie domyślnego zakresu, którego Pennant używa do sprawdzania funkcji. Na przykład, może wszystkie Twoje funkcje są sprawdzane względem zespołu aktualnie uwierzytelnionego użytkownika zamiast użytkownika. Zamiast wywoływać `Feature::for($user->team)` za każdym razem, gdy sprawdzasz funkcję, możesz zamiast tego określić zespół jako domyślny zakres. Zazwyczaj powinno to być wykonane w jednym z dostawców usług Twojej aplikacji:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Auth;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrapuj wszelkie usługi aplikacji.
     */
    public function boot(): void
    {
        Feature::resolveScopeUsing(fn ($driver) => Auth::user()?->team);

        // ...
    }
}
```

Jeśli żaden zakres nie jest jawnie podany przez metodę `for`, sprawdzenie funkcji będzie teraz używać zespołu aktualnie uwierzytelnionego użytkownika jako domyślnego zakresu:

```php
Feature::active('billing-v2');

// Jest teraz równoważne...

Feature::for($user->team)->active('billing-v2');
```

<a name="nullable-scope"></a>
### Zakres Nullable

Jeśli zakres, który podajesz podczas sprawdzania funkcji, jest `null` i definicja funkcji nie obsługuje `null` przez typ nullable lub przez włączenie `null` w typ union, Pennant automatycznie zwróci `false` jako wartość wyniku funkcji.

Więc jeśli zakres, który przekazujesz do funkcji, jest potencjalnie `null` i chcesz, aby resolver wartości funkcji został wywołany, powinieneś to uwzględnić w definicji funkcji. Zakres `null` może wystąpić, jeśli sprawdzasz funkcję w poleceniu Artisan, zadaniu w kolejce lub nieuwierzytelnionej trasie. Ponieważ zazwyczaj nie ma uwierzytelnionego użytkownika w tych kontekstach, domyślny zakres będzie `null`.

Jeśli nie zawsze [jawnie określasz zakres funkcji](#specifying-the-scope), powinieneś upewnić się, że typ zakresu jest "nullable" i obsłużyć wartość zakresu `null` w logice definicji funkcji:

```php
use App\Models\User;
use Illuminate\Support\Lottery;
use Laravel\Pennant\Feature;

Feature::define('new-api', fn (User $user) => match (true) {// [tl! remove]
Feature::define('new-api', fn (User|null $user) => match (true) {// [tl! add]
    $user === null => true,// [tl! add]
    $user->isInternalTeamMember() => true,
    $user->isHighTrafficCustomer() => false,
    default => Lottery::odds(1 / 100),
});
```

<a name="identifying-scope"></a>
### Identyfikowanie Zakresu

Wbudowane sterowniki `array` i `database` Pennanta wiedzą, jak prawidłowo przechowywać identyfikatory zakresu dla wszystkich typów danych PHP, a także modeli Eloquent. Jednak jeśli Twoja aplikacja wykorzystuje sterownik Pennant innej firmy, ten sterownik może nie wiedzieć, jak prawidłowo przechowywać identyfikator dla modelu Eloquent lub innych niestandardowych typów w Twojej aplikacji.

W świetle tego Pennant pozwala formatować wartości zakresu do przechowywania przez implementację kontraktu `FeatureScopeable` na obiektach w Twojej aplikacji, które są używane jako zakresy Pennant.

Na przykład wyobraź sobie, że używasz dwóch różnych sterowników funkcji w jednej aplikacji: wbudowanego sterownika `database` i sterownika "Flag Rocket" innej firmy. Sterownik "Flag Rocket" nie wie, jak prawidłowo przechowywać model Eloquent. Zamiast tego wymaga instancji `FlagRocketUser`. Implementując `toFeatureIdentifier` zdefiniowany przez kontrakt `FeatureScopeable`, możemy dostosować wartość zakresu do przechowywania, dostarczaną każdemu sterownikowi używanemu przez naszą aplikację:

```php
<?php

namespace App\Models;

use FlagRocket\FlagRocketUser;
use Illuminate\Database\Eloquent\Model;
use Laravel\Pennant\Contracts\FeatureScopeable;

class User extends Model implements FeatureScopeable
{
    /**
     * Rzutuj obiekt na identyfikator zakresu funkcji dla danego sterownika.
     */
    public function toFeatureIdentifier(string $driver): mixed
    {
        return match($driver) {
            'database' => $this,
            'flag-rocket' => FlagRocketUser::fromId($this->flag_rocket_id),
        };
    }
}
```

<a name="serializing-scope"></a>
### Serializacja Zakresu

Domyślnie Pennant będzie używać w pełni kwalifikowanej nazwy klasy podczas przechowywania funkcji powiązanej z modelem Eloquent. Jeśli już używasz [mapy morph Eloquent](/docs/{{version}}/eloquent-relationships#custom-polymorphic-types), możesz wybrać, aby Pennant również używał mapy morph, aby oddzielić przechowywaną funkcję od struktury Twojej aplikacji.

Aby to osiągnąć, po zdefiniowaniu mapy morph Eloquent w dostawcy usług, możesz wywołać metodę `useMorphMap` fasady `Feature`:

```php
use Illuminate\Database\Eloquent\Relations\Relation;
use Laravel\Pennant\Feature;

Relation::enforceMorphMap([
    'post' => 'App\Models\Post',
    'video' => 'App\Models\Video',
]);

Feature::useMorphMap();
```

<a name="rich-feature-values"></a>
## Bogate Wartości Funkcji

Do tej pory pokazywaliśmy głównie funkcje jako będące w stanie binarnym, co oznacza, że są "aktywne" lub "nieaktywne", ale Pennant pozwala również przechowywać bogate wartości.

Na przykład wyobraź sobie, że testujesz trzy nowe kolory dla przycisku "Kup teraz" Twojej aplikacji. Zamiast zwracać `true` lub `false` z definicji funkcji, możesz zamiast tego zwrócić ciąg znaków:

```php
use Illuminate\Support\Arr;
use Laravel\Pennant\Feature;

Feature::define('purchase-button', fn (User $user) => Arr::random([
    'blue-sapphire',
    'seafoam-green',
    'tart-orange',
]));
```

Możesz pobrać wartość funkcji `purchase-button` używając metody `value`:

```php
$color = Feature::value('purchase-button');
```

Dołączona dyrektywa Blade Pennanta również ułatwia warunkowe renderowanie treści na podstawie aktualnej wartości funkcji:

```blade
@feature('purchase-button', 'blue-sapphire')
    <!-- 'blue-sapphire' jest aktywny -->
@elsefeature('purchase-button', 'seafoam-green')
    <!-- 'seafoam-green' jest aktywny -->
@elsefeature('purchase-button', 'tart-orange')
    <!-- 'tart-orange' jest aktywny -->
@endfeature
```

> [!NOTE]
> Używając bogatych wartości, ważne jest, aby wiedzieć, że funkcja jest uważana za "aktywną", gdy ma jakąkolwiek wartość inną niż `false`.

Podczas wywoływania [warunkowej metody `when`](#conditional-execution), bogata wartość funkcji zostanie przekazana do pierwszego domknięcia:

```php
Feature::when('purchase-button',
    fn ($color) => /* ... */,
    fn () => /* ... */,
);
```

Podobnie, podczas wywoływania warunkowej metody `unless`, bogata wartość funkcji zostanie przekazana do opcjonalnego drugiego domknięcia:

```php
Feature::unless('purchase-button',
    fn () => /* ... */,
    fn ($color) => /* ... */,
);
```

<a name="retrieving-multiple-features"></a>
## Pobieranie Wielu Funkcji

Metoda `values` pozwala na pobieranie wielu funkcji dla danego zakresu:

```php
Feature::values(['billing-v2', 'purchase-button']);

// [
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
// ]
```

Lub możesz użyć metody `all`, aby pobrać wartości wszystkich zdefiniowanych funkcji dla danego zakresu:

```php
Feature::all();

// [
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
//     'site-redesign' => true,
// ]
```

Jednak funkcje oparte na klasach są dynamicznie rejestrowane i nie są znane Pennantowi, dopóki nie zostaną jawnie sprawdzone. Oznacza to, że funkcje oparte na klasach Twojej aplikacji mogą nie pojawić się w wynikach zwróconych przez metodę `all`, jeśli nie zostały już sprawdzone podczas bieżącego żądania.

Jeśli chcesz zapewnić, że klasy funkcji są zawsze uwzględniane podczas używania metody `all`, możesz użyć możliwości odkrywania funkcji Pennanta. Aby rozpocząć, wywołaj metodę `discover` w jednym z dostawców usług Twojej aplikacji:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrapuj wszelkie usługi aplikacji.
     */
    public function boot(): void
    {
        Feature::discover();

        // ...
    }
}
```

Metoda `discover` zarejestruje wszystkie klasy funkcji w katalogu `app/Features` Twojej aplikacji. Metoda `all` będzie teraz uwzględniać te klasy w swoich wynikach, niezależnie od tego, czy zostały sprawdzone podczas bieżącego żądania:

```php
Feature::all();

// [
//     'App\Features\NewApi' => true,
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
//     'site-redesign' => true,
// ]
```

<a name="eager-loading"></a>
## Eager Loading

Chociaż Pennant utrzymuje cache w pamięci wszystkich rozwiązanych funkcji dla pojedynczego żądania, nadal możliwe jest napotkanie problemów z wydajnością. Aby temu zaradzić, Pennant oferuje możliwość eager loadingu wartości funkcji.

Aby to zilustrować, wyobraź sobie, że sprawdzamy, czy funkcja jest aktywna w pętli:

```php
use Laravel\Pennant\Feature;

foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) {
        $user->notify(new RegistrationSuccess);
    }
}
```

Zakładając, że używamy sterownika bazy danych, ten kod wykona zapytanie do bazy danych dla każdego użytkownika w pętli - potencjalnie wykonując setki zapytań. Jednak używając metody `load` Pennanta, możemy usunąć to potencjalne wąskie gardło wydajności przez eager loading wartości funkcji dla kolekcji użytkowników lub zakresów:

```php
Feature::for($users)->load(['notifications-beta']);

foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) {
        $user->notify(new RegistrationSuccess);
    }
}
```

Aby załadować wartości funkcji tylko wtedy, gdy nie zostały jeszcze załadowane, możesz użyć metody `loadMissing`:

```php
Feature::for($users)->loadMissing([
    'new-api',
    'purchase-button',
    'notifications-beta',
]);
```

Możesz załadować wszystkie zdefiniowane funkcje używając metody `loadAll`:

```php
Feature::for($users)->loadAll();
```

<a name="updating-values"></a>
## Aktualizacja Wartości

Gdy wartość funkcji jest rozwiązywana po raz pierwszy, podstawowy sterownik zapisze wynik w przechowywaniu. Jest to często konieczne, aby zapewnić spójne doświadczenie dla użytkowników w różnych żądaniach. Jednak czasami możesz chcieć ręcznie zaktualizować przechowywaną wartość funkcji.

Aby to osiągnąć, możesz użyć metod `activate` i `deactivate`, aby przełączać funkcję "włącz" lub "wyłącz":

```php
use Laravel\Pennant\Feature;

// Aktywuj funkcję dla domyślnego zakresu...
Feature::activate('new-api');

// Dezaktywuj funkcję dla danego zakresu...
Feature::for($user->team)->deactivate('billing-v2');
```

Możliwe jest również ręczne ustawienie bogatej wartości dla funkcji przez podanie drugiego argumentu do metody `activate`:

```php
Feature::activate('purchase-button', 'seafoam-green');
```

Aby poinstruować Pennant, aby zapomniał przechowywaną wartość funkcji, możesz użyć metody `forget`. Gdy funkcja zostanie ponownie sprawdzona, Pennant rozwiąże wartość funkcji z jej definicji funkcji:

```php
Feature::forget('purchase-button');
```

<a name="bulk-updates"></a>
### Aktualizacje Zbiorcze

Aby zaktualizować przechowywane wartości funkcji zbiorczo, możesz użyć metod `activateForEveryone` i `deactivateForEveryone`.

Na przykład wyobraź sobie, że jesteś teraz pewien stabilności funkcji `new-api` i ustaliłeś najlepszy kolor `'purchase-button'` dla przepływu kasy - możesz odpowiednio zaktualizować przechowywaną wartość dla wszystkich użytkowników:

```php
use Laravel\Pennant\Feature;

Feature::activateForEveryone('new-api');

Feature::activateForEveryone('purchase-button', 'seafoam-green');
```

Alternatywnie możesz dezaktywować funkcję dla wszystkich użytkowników:

```php
Feature::deactivateForEveryone('new-api');
```

> [!NOTE]
> To zaktualizuje tylko rozwiązane wartości funkcji, które zostały zapisane przez sterownik przechowywania Pennanta. Będziesz także musiał zaktualizować definicję funkcji w swojej aplikacji.

<a name="purging-features"></a>
### Czyszczenie Funkcji

Czasami może być przydatne wyczyszczenie całej funkcji z przechowywania. Jest to zazwyczaj konieczne, jeśli usunąłeś funkcję ze swojej aplikacji lub dokonałeś poprawek w definicji funkcji, które chcesz wdrożyć dla wszystkich użytkowników.

Możesz usunąć wszystkie przechowywane wartości funkcji używając metody `purge`:

```php
// Czyszczenie pojedynczej funkcji...
Feature::purge('new-api');

// Czyszczenie wielu funkcji...
Feature::purge(['new-api', 'purchase-button']);
```

Jeśli chcesz wyczyścić _wszystkie_ funkcje z przechowywania, możesz wywołać metodę `purge` bez żadnych argumentów:

```php
Feature::purge();
```

Ponieważ może być przydatne czyszczenie funkcji w ramach pipeline'u wdrożenia aplikacji, Pennant zawiera polecenie Artisan `pennant:purge`, które wyczyści podane funkcje z przechowywania:

```shell
php artisan pennant:purge new-api

php artisan pennant:purge new-api purchase-button
```

Możliwe jest również wyczyszczenie wszystkich funkcji _z wyjątkiem_ tych na danej liście funkcji. Na przykład wyobraź sobie, że chcesz wyczyścić wszystkie funkcje, ale zachować wartości dla funkcji "new-api" i "purchase-button" w przechowywaniu. Aby to osiągnąć, możesz przekazać te nazwy funkcji do opcji `--except`:

```shell
php artisan pennant:purge --except=new-api --except=purchase-button
```

Dla wygody polecenie `pennant:purge` obsługuje również flagę `--except-registered`. Ta flaga wskazuje, że wszystkie funkcje z wyjątkiem tych jawnie zarejestrowanych w dostawcy usług powinny zostać wyczyszczone:

```shell
php artisan pennant:purge --except-registered
```

<a name="testing"></a>
## Testowanie

Podczas testowania kodu, który współdziała z flagami funkcji, najłatwiejszym sposobem kontrolowania zwracanej wartości flagi funkcji w testach jest po prostu ponowne zdefiniowanie funkcji. Na przykład wyobraź sobie, że masz następującą funkcję zdefiniowaną w jednym z dostawców usług Twojej aplikacji:

```php
use Illuminate\Support\Arr;
use Laravel\Pennant\Feature;

Feature::define('purchase-button', fn () => Arr::random([
    'blue-sapphire',
    'seafoam-green',
    'tart-orange',
]));
```

Aby zmodyfikować zwracaną wartość funkcji w testach, możesz ponownie zdefiniować funkcję na początku testu. Następujący test zawsze przejdzie, nawet jeśli implementacja `Arr::random()` nadal jest obecna w dostawcy usług:

```php tab=Pest
use Laravel\Pennant\Feature;

test('it can control feature values', function () {
    Feature::define('purchase-button', 'seafoam-green');

    expect(Feature::value('purchase-button'))->toBe('seafoam-green');
});
```

```php tab=PHPUnit
use Laravel\Pennant\Feature;

public function test_it_can_control_feature_values()
{
    Feature::define('purchase-button', 'seafoam-green');

    $this->assertSame('seafoam-green', Feature::value('purchase-button'));
}
```

To samo podejście może być użyte dla funkcji opartych na klasach:

```php tab=Pest
use Laravel\Pennant\Feature;

test('it can control feature values', function () {
    Feature::define(NewApi::class, true);

    expect(Feature::value(NewApi::class))->toBeTrue();
});
```

```php tab=PHPUnit
use App\Features\NewApi;
use Laravel\Pennant\Feature;

public function test_it_can_control_feature_values()
{
    Feature::define(NewApi::class, true);

    $this->assertTrue(Feature::value(NewApi::class));
}
```

Jeśli Twoja funkcja zwraca instancję `Lottery`, dostępnych jest kilka przydatnych [helperów testowych](/docs/{{version}}/helpers#testing-lotteries).

<a name="store-configuration"></a>
#### Konfiguracja Store

Możesz skonfigurować store, który Pennant będzie używać podczas testowania, definiując zmienną środowiskową `PENNANT_STORE` w pliku `phpunit.xml` Twojej aplikacji:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit colors="true">
    <!-- ... -->
    <php>
        <env name="PENNANT_STORE" value="array"/>
        <!-- ... -->
    </php>
</phpunit>
```

<a name="adding-custom-pennant-drivers"></a>
## Dodawanie Niestandardowych Sterowników Pennant

<a name="implementing-the-driver"></a>
#### Implementacja Sterownika

Jeśli żaden z istniejących sterowników przechowywania Pennanta nie pasuje do potrzeb Twojej aplikacji, możesz napisać własny sterownik przechowywania. Twój niestandardowy sterownik powinien implementować interfejs `Laravel\Pennant\Contracts\Driver`:

```php
<?php

namespace App\Extensions;

use Laravel\Pennant\Contracts\Driver;

class RedisFeatureDriver implements Driver
{
    public function define(string $feature, callable $resolver): void {}
    public function defined(): array {}
    public function getAll(array $features): array {}
    public function get(string $feature, mixed $scope): mixed {}
    public function set(string $feature, mixed $scope, mixed $value): void {}
    public function setForAllScopes(string $feature, mixed $value): void {}
    public function delete(string $feature, mixed $scope): void {}
    public function purge(array|null $features): void {}
}
```

Teraz musimy tylko zaimplementować każdą z tych metod używając połączenia Redis. Aby zobaczyć przykład, jak zaimplementować każdą z tych metod, spójrz na `Laravel\Pennant\Drivers\DatabaseDriver` w [kodzie źródłowym Pennant](https://github.com/laravel/pennant/blob/1.x/src/Drivers/DatabaseDriver.php)

> [!NOTE]
> Laravel nie jest dostarczany z katalogiem do zawierania Twoich rozszerzeń. Możesz umieścić je gdziekolwiek chcesz. W tym przykładzie utworzyliśmy katalog `Extensions`, aby pomieścić `RedisFeatureDriver`.

<a name="registering-the-driver"></a>
#### Rejestracja Sterownika

Po zaimplementowaniu sterownika jesteś gotowy, aby zarejestrować go w Laravel. Aby dodać dodatkowe sterowniki do Pennant, możesz użyć metody `extend` dostarczonej przez fasadę `Feature`. Powinieneś wywołać metodę `extend` z metody `boot` jednego z [dostawców usług](/docs/{{version}}/providers) Twojej aplikacji:

```php
<?php

namespace App\Providers;

use App\Extensions\RedisFeatureDriver;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Zarejestruj wszelkie usługi aplikacji.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrapuj wszelkie usługi aplikacji.
     */
    public function boot(): void
    {
        Feature::extend('redis', function (Application $app) {
            return new RedisFeatureDriver($app->make('redis'), $app->make('events'), []);
        });
    }
}
```

Po zarejestrowaniu sterownika możesz użyć sterownika `redis` w pliku konfiguracyjnym `config/pennant.php` Twojej aplikacji:

```php
'stores' => [

    'redis' => [
        'driver' => 'redis',
        'connection' => null,
    ],

    // ...

],
```

<a name="defining-features-externally"></a>
### Definiowanie Funkcji Zewnętrznie

Jeśli Twój sterownik jest wrapperem wokół platformy flag funkcji innej firmy, prawdopodobnie zdefiniujesz funkcje na platformie, a nie używając metody `Feature::define` Pennanta. Jeśli tak jest, Twój niestandardowy sterownik powinien również implementować interfejs `Laravel\Pennant\Contracts\DefinesFeaturesExternally`:

```php
<?php

namespace App\Extensions;

use Laravel\Pennant\Contracts\Driver;
use Laravel\Pennant\Contracts\DefinesFeaturesExternally;

class FeatureFlagServiceDriver implements Driver, DefinesFeaturesExternally
{
    /**
     * Pobierz funkcje zdefiniowane dla danego zakresu.
     */
    public function definedFeaturesForScope(mixed $scope): array {}

    /* ... */
}
```

Metoda `definedFeaturesForScope` powinna zwracać listę nazw funkcji zdefiniowanych dla podanego zakresu.

<a name="events"></a>
## Zdarzenia

Pennant wysyła różnorodne zdarzenia, które mogą być przydatne podczas śledzenia flag funkcji w całej aplikacji.

### `Laravel\Pennant\Events\FeatureRetrieved`

To zdarzenie jest wysyłane za każdym razem, gdy [funkcja jest sprawdzana](#checking-features). To zdarzenie może być przydatne do tworzenia i śledzenia metryk względem użycia flagi funkcji w całej aplikacji.

### `Laravel\Pennant\Events\FeatureResolved`

To zdarzenie jest wysyłane za pierwszym razem, gdy wartość funkcji jest rozwiązywana dla określonego zakresu.

### `Laravel\Pennant\Events\UnknownFeatureResolved`

To zdarzenie jest wysyłane za pierwszym razem, gdy nieznana funkcja jest rozwiązywana dla określonego zakresu. Słuchanie tego zdarzenia może być przydatne, jeśli zamierzałeś usunąć flagę funkcji, ale przypadkowo zostawiłeś przypadkowe odniesienia do niej w całej aplikacji:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Log;
use Laravel\Pennant\Events\UnknownFeatureResolved;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrapuj wszelkie usługi aplikacji.
     */
    public function boot(): void
    {
        Event::listen(function (UnknownFeatureResolved $event) {
            Log::error("Resolving unknown feature [{$event->feature}].");
        });
    }
}
```

### `Laravel\Pennant\Events\DynamicallyRegisteringFeatureClass`

To zdarzenie jest wysyłane, gdy [funkcja oparta na klasie](#class-based-features) jest dynamicznie sprawdzana po raz pierwszy podczas żądania.

### `Laravel\Pennant\Events\UnexpectedNullScopeEncountered`

To zdarzenie jest wysyłane, gdy zakres `null` jest przekazywany do definicji funkcji, która [nie obsługuje null](#nullable-scope).

Ta sytuacja jest obsługiwana gracefully i funkcja zwróci `false`. Jednak jeśli chcesz zrezygnować z tego domyślnego zachowania graceful funkcji, możesz zarejestrować listener dla tego zdarzenia w metodzie `boot` `AppServiceProvider` Twojej aplikacji:

```php
use Illuminate\Support\Facades\Log;
use Laravel\Pennant\Events\UnexpectedNullScopeEncountered;

/**
 * Bootstrapuj wszelkie usługi aplikacji.
 */
public function boot(): void
{
    Event::listen(UnexpectedNullScopeEncountered::class, fn () => abort(500));
}
```

### `Laravel\Pennant\Events\FeatureUpdated`

To zdarzenie jest wysyłane podczas aktualizacji funkcji dla zakresu, zazwyczaj przez wywołanie `activate` lub `deactivate`.

### `Laravel\Pennant\Events\FeatureUpdatedForAllScopes`

To zdarzenie jest wysyłane podczas aktualizacji funkcji dla wszystkich zakresów, zazwyczaj przez wywołanie `activateForEveryone` lub `deactivateForEveryone`.

### `Laravel\Pennant\Events\FeatureDeleted`

To zdarzenie jest wysyłane podczas usuwania funkcji dla zakresu, zazwyczaj przez wywołanie `forget`.

### `Laravel\Pennant\Events\FeaturesPurged`

To zdarzenie jest wysyłane podczas czyszczenia określonych funkcji.

### `Laravel\Pennant\Events\AllFeaturesPurged`

To zdarzenie jest wysyłane podczas czyszczenia wszystkich funkcji.
