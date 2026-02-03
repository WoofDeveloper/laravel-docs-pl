# Kontener Usług

- [Wprowadzenie](#introduction)
    - [Rozwiązywanie Bez Konfiguracji](#zero-configuration-resolution)
    - [Kiedy Używać Kontenera](#when-to-use-the-container)
- [Wiązanie](#binding)
    - [Podstawy Wiązania](#binding-basics)
    - [Wiązanie Interfejsów z Implementacjami](#binding-interfaces-to-implementations)
    - [Wiązanie Kontekstowe](#contextual-binding)
    - [Atrybuty Kontekstowe](#contextual-attributes)
    - [Wiązanie Typów Prymitywnych](#binding-primitives)
    - [Wiązanie Zmiennej Liczby Argumentów](#binding-typed-variadics)
    - [Tagowanie](#tagging)
    - [Rozszerzanie Wiązań](#extending-bindings)
- [Rozwiązywanie](#resolving)
    - [Metoda Make](#the-make-method)
    - [Automatyczne Wstrzykiwanie](#automatic-injection)
- [Wywołanie Metody i Wstrzykiwanie](#method-invocation-and-injection)
- [Zdarzenia Kontenera](#container-events)
    - [Ponowne Wiązanie](#rebinding)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## Wprowadzenie

Kontener usług Laravel to potężne narzędzie do zarządzania zależnościami klas i wykonywania wstrzykiwania zależności. Wstrzykiwanie zależności to wymyślne określenie, które w istocie oznacza: zależności klas są "wstrzykiwane" do klasy przez konstruktor lub, w niektórych przypadkach, metody "setter".

Spójrzmy na prosty przykład:

```php
<?php

namespace App\Http\Controllers;

use App\Services\AppleMusic;
use Illuminate\View\View;

class PodcastController extends Controller
{
    /**
     * Create a new controller instance.
     */
    public function __construct(
        protected AppleMusic $apple,
    ) {}

    /**
     * Show information about the given podcast.
     */
    public function show(string $id): View
    {
        return view('podcasts.show', [
            'podcast' => $this->apple->findPodcast($id)
        ]);
    }
}
```

W tym przykładzie `PodcastController` musi pobierać podcasty ze źródła danych, takiego jak Apple Music. Dlatego **wstrzykniemy** usługę, która jest w stanie pobierać podcasty. Ponieważ usługa jest wstrzykiwana, możemy łatwo "mockować" lub utworzyć makietę implementacji usługi `AppleMusic` podczas testowania naszej aplikacji.

Głębokie zrozumienie kontenera usług Laravel jest niezbędne do budowy potężnej, dużej aplikacji, a także do współtworzenia samego rdzenia Laravel.

<a name="zero-configuration-resolution"></a>
### Rozwiązywanie Bez Konfiguracji

Jeśli klasa nie ma zależności lub zależy tylko od innych konkretnych klas (a nie interfejsów), kontener nie musi być instruowany, jak rozwiązać tę klasę. Na przykład, możesz umieścić następujący kod w pliku `routes/web.php`:

```php
<?php

class Service
{
    // ...
}

Route::get('/', function (Service $service) {
    dd($service::class);
});
```

W tym przykładzie, odwiedzenie trasy `/` Twojej aplikacji automatycznie rozwiąże klasę `Service` i wstrzyknie ją do obsługi trasy. To zmienia zasady gry. Oznacza to, że możesz rozwijać swoją aplikację i korzystać z wstrzykiwania zależności bez martwienia się o rozdęte pliki konfiguracyjne.

Na szczęście, wiele klas, które będziesz pisać podczas tworzenia aplikacji Laravel, automatycznie otrzymuje swoje zależności poprzez kontener, w tym [kontrolery](/docs/{{version}}/controllers), [słuchacze zdarzeń](/docs/{{version}}/events), [middleware](/docs/{{version}}/middleware) i wiele innych. Dodatkowo, możesz określać zależności w metodzie `handle` [zadań kolejkowanych](/docs/{{version}}/queues). Gdy raz zasmakujesz mocy automatycznego wstrzykiwania zależności bez konfiguracji, wydaje się niemożliwe rozwijać aplikacje bez tego.

<a name="when-to-use-the-container"></a>
### Kiedy Używać Kontenera

Dzięki rozwiązywaniu bez konfiguracji, często będziesz określać zależności w trasach, kontrolerach, słuchaczach zdarzeń i gdzie indziej, bez ręcznej interakcji z kontenerem. Na przykład, możesz określić obiekt `Illuminate\Http\Request` w definicji trasy, aby łatwo uzyskać dostęp do bieżącego żądania. Mimo że nigdy nie musimy wchodzić w interakcję z kontenerem, aby napisać ten kod, zarządza on wstrzykiwaniem tych zależności za kulisami:

```php
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    // ...
});
```

W wielu przypadkach, dzięki automatycznemu wstrzykiwaniu zależności i [fasadom](/docs/{{version}}/facades), możesz budować aplikacje Laravel bez **konieczności** ręcznego wiązania lub rozwiązywania czegokolwiek z kontenera. **Więc kiedy kiedykolwiek ręcznie wchodzisz w interakcję z kontenerem?** Przyjrzyjmy się dwóm sytuacjom.

Po pierwsze, jeśli piszesz klasę, która implementuje interfejs i chcesz określić ten interfejs w trasie lub konstruktorze klasy, musisz [powiedzieć kontenerowi, jak rozwiązać ten interfejs](#binding-interfaces-to-implementations). Po drugie, jeśli [piszesz pakiet Laravel](/docs/{{version}}/packages), który planujesz udostępnić innym programistom Laravel, możesz potrzebować związać usługi swojego pakietu z kontenerem.

<a name="binding"></a>
## Wiązanie

<a name="binding-basics"></a>
### Podstawy Wiązania

<a name="simple-bindings"></a>
#### Proste Wiązania

Niemal wszystkie wiązania kontenera usług będą rejestrowane w ramach [dostawców usług](/docs/{{version}}/providers), więc większość tych przykładów zademonstruje użycie kontenera w tym kontekście.

W ramach dostawcy usług zawsze masz dostęp do kontenera poprzez właściwość `$this->app`. Możemy zarejestrować wiązanie używając metody `bind`, przekazując nazwę klasy lub interfejsu, którą chcemy zarejestrować, wraz z domknięciem, które zwraca instancję klasy:

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

Zauważ, że otrzymujemy sam kontener jako argument resolvera. Możemy następnie użyć kontenera do rozwiązania podzależności obiektu, który budujemy.

Jak wspomniano, zazwyczaj będziesz wchodzić w interakcję z kontenerem w ramach dostawców usług; jednakże, jeśli chcesz wchodzić w interakcję z kontenerem poza dostawcą usług, możesz to zrobić za pośrednictwem [fasady](/docs/{{version}}/facades) `App`:

```php
use App\Services\Transistor;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\App;

App::bind(Transistor::class, function (Application $app) {
    // ...
});
```

Możesz użyć metody `bindIf`, aby zarejestrować wiązanie kontenera tylko wtedy, gdy wiązanie nie zostało już zarejestrowane dla danego typu:

```php
$this->app->bindIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

Dla wygody, możesz pominąć podawanie nazwy klasy lub interfejsu, którą chcesz zarejestrować jako osobny argument i zamiast tego pozwolić Laravel wywnioskować typ z typu zwracanego domknięcia, które przekazujesz do metody `bind`:

```php
App::bind(function (Application $app): Transistor {
    return new Transistor($app->make(PodcastParser::class));
});
```

> [!NOTE]
> Nie ma potrzeby wiązania klas z kontenerem, jeśli nie zależą od żadnych interfejsów. Kontener nie musi być instruowany, jak budować te obiekty, ponieważ może automatycznie rozwiązywać te obiekty za pomocą refleksji.

<a name="binding-a-singleton"></a>
#### Wiązanie Singletona

Metoda `singleton` wiąże klasę lub interfejs z kontenerem, który powinien zostać rozwiązany tylko raz. Po rozwiązaniu wiązania singleton, ta sama instancja obiektu będzie zwracana przy kolejnych wywołaniach kontenera:

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->singleton(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

Możesz użyć metody `singletonIf`, aby zarejestrować wiązanie singleton kontenera tylko wtedy, gdy wiązanie nie zostało już zarejestrowane dla danego typu:

```php
$this->app->singletonIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

<a name="singleton-attribute"></a>
#### Atrybut Singleton

Alternatywnie, możesz oznaczyć interfejs lub klasę atrybutem `#[Singleton]`, aby wskazać kontenerowi, że powinien być rozwiązany tylko raz:

```php
<?php

namespace App\Services;

use Illuminate\Container\Attributes\Singleton;

#[Singleton]
class Transistor
{
    // ...
}
```

<a name="binding-scoped"></a>
#### Wiązanie Zakresowych Singletonów

Metoda `scoped` wiąże klasę lub interfejs z kontenerem, który powinien zostać rozwiązany tylko raz w ramach danego cyklu życia żądania / zadania Laravel. Choć ta metoda jest podobna do metody `singleton`, instancje zarejestrowane za pomocą metody `scoped` będą czyszczone za każdym razem, gdy aplikacja Laravel rozpoczyna nowy "cykl życia", na przykład gdy worker [Laravel Octane](/docs/{{version}}/octane) przetwarza nowe żądanie lub gdy worker [kolejki](/docs/{{version}}/queues) Laravel przetwarza nowe zadanie:

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->scoped(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

Możesz użyć metody `scopedIf`, aby zarejestrować zakresowe wiązanie kontenera tylko wtedy, gdy wiązanie nie zostało już zarejestrowane dla danego typu:

```php
$this->app->scopedIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

<a name="scoped-attribute"></a>
#### Atrybut Scoped

Alternatywnie, możesz oznaczyć interfejs lub klasę atrybutem `#[Scoped]`, aby wskazać kontenerowi, że powinien być rozwiązany tylko raz w ramach danego cyklu życia żądania / zadania Laravel:

```php
<?php

namespace App\Services;

use Illuminate\Container\Attributes\Scoped;

#[Scoped]
class Transistor
{
    // ...
}
```

<a name="binding-instances"></a>
#### Wiązanie Instancji

Możesz również związać istniejącą instancję obiektu z kontenerem za pomocą metody `instance`. Podana instancja zawsze będzie zwracana przy kolejnych wywołaniach kontenera:

```php
use App\Services\Transistor;
use App\Services\PodcastParser;

$service = new Transistor(new PodcastParser);

$this->app->instance(Transistor::class, $service);
```

<a name="binding-interfaces-to-implementations"></a>
### Wiązanie Interfejsów z Implementacjami

Bar­zo potężną funkcją kontenera usług jest jego zdolność do wiązania interfejsu z daną implementacją. Na przykład, załóźmy, że mamy interfejs `EventPusher` i implementację `RedisEventPusher`. Po zakodowaniu naszej implementacji `RedisEventPusher` tego interfejsu, możemy zarejestrować ją w kontenerze usług w następujący sposób:

```php
use App\Contracts\EventPusher;
use App\Services\RedisEventPusher;

$this->app->bind(EventPusher::class, RedisEventPusher::class);
```

Ta instrukcja mówi kontenerowi, że powinien wstrzyknąć `RedisEventPusher`, gdy klasa potrzebuje implementacji `EventPusher`. Teraz możemy określić interfejs `EventPusher` w konstruktorze klasy, która jest rozwiązywana przez kontener. Pamiętaj, że kontrolery, słuchacze zdarzeń, middleware i różne inne typy klas w aplikacjach Laravel są zawsze rozwiązywane za pomocą kontenera:

```php
use App\Contracts\EventPusher;

/**
 * Create a new class instance.
 */
public function __construct(
    protected EventPusher $pusher,
) {}
```

<a name="bind-attribute"></a>
#### Atrybut Bind

Laravel zapewnia również atrybut `Bind` dla dodatkowej wygody. Możesz zastosować ten atrybut do dowolnego interfejsu, aby powiedzieć Laravel, która implementacja powinna być automatycznie wstrzykiwana za każdym razem, gdy ten interfejs jest żądany. Podczas używania atrybutu `Bind` nie ma potrzeby wykonywania żadnej dodatkowej rejestracji usług w dostawcach usług aplikacji.

Ponadto, wiele atrybutów `Bind` można umieścić na interfejsie, aby skonfigurować inną implementację, która powinna być wstrzykiwana dla danego zestawu środowisk:

```php
<?php

namespace App\Contracts;

use App\Services\FakeEventPusher;
use App\Services\RedisEventPusher;
use Illuminate\Container\Attributes\Bind;

#[Bind(RedisEventPusher::class)]
#[Bind(FakeEventPusher::class, environments: ['local', 'testing'])]
interface EventPusher
{
    // ...
}
```

Ponadto, atrybuty [Singleton](#singleton-attribute) i [Scoped](#scoped-attribute) mogą być stosowane, aby wskazać, czy wiązania kontenera powinny być rozwiązywane raz lub raz na żądanie / cykl życia zadania:

```php
use App\Services\RedisEventPusher;
use Illuminate\Container\Attributes\Bind;
use Illuminate\Container\Attributes\Singleton;

#[Bind(RedisEventPusher::class)]
#[Singleton]
interface EventPusher
{
    // ...
}
```

<a name="contextual-binding"></a>
### Wiązanie Kontekstowe

Czasami możesz mieć dwie klasy, które wykorzystują ten sam interfejs, ale chcesz wstrzyknąć różne implementacje do każdej klasy. Na przykład, dwa kontrolery mogą zależeć od różnych implementacji [kontraktu](/docs/{{version}}/contracts) `Illuminate\Contracts\Filesystem\Filesystem`. Laravel zapewnia prosty, płynny interfejs do definiowania tego zachowania:

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\UploadController;
use App\Http\Controllers\VideoController;
use Illuminate\Contracts\Filesystem\Filesystem;
use Illuminate\Support\Facades\Storage;

$this->app->when(PhotoController::class)
    ->needs(Filesystem::class)
    ->give(function () {
        return Storage::disk('local');
    });

$this->app->when([VideoController::class, UploadController::class])
    ->needs(Filesystem::class)
    ->give(function () {
        return Storage::disk('s3');
    });
```

<a name="contextual-attributes"></a>
### Atrybuty Kontekstowe

Ponieważ wiązanie kontekstowe jest często używane do wstrzykiwania implementacji sterowników lub wartości konfiguracyjnych, Laravel oferuje różnorodne atrybuty wiązania kontekstowego, które pozwalają wstrzykiwać te typy wartości bez ręcznego definiowania wiązań kontekstowych w dostawcach usług.

Na przykład, atrybut `Storage` może być używany do wstrzykiwania konkretnego [dysku magazynu](/docs/{{version}}/filesystem):

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Container\Attributes\Storage;
use Illuminate\Contracts\Filesystem\Filesystem;

class PhotoController extends Controller
{
    public function __construct(
        #[Storage('local')] protected Filesystem $filesystem
    ) {
        // ...
    }
}
```

Oprócz atrybutu `Storage`, Laravel oferuje atrybuty `Auth`, `Cache`, `Config`, `Context`, `DB`, `Give`, `Log`, `RouteParameter` i [Tag](#tagging):

```php
<?php

namespace App\Http\Controllers;

use App\Contracts\UserRepository;
use App\Models\Photo;
use App\Repositories\DatabaseRepository;
use Illuminate\Container\Attributes\Auth;
use Illuminate\Container\Attributes\Cache;
use Illuminate\Container\Attributes\Config;
use Illuminate\Container\Attributes\Context;
use Illuminate\Container\Attributes\DB;
use Illuminate\Container\Attributes\Give;
use Illuminate\Container\Attributes\Log;
use Illuminate\Container\Attributes\RouteParameter;
use Illuminate\Container\Attributes\Tag;
use Illuminate\Contracts\Auth\Guard;
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Database\Connection;
use Psr\Log\LoggerInterface;

class PhotoController extends Controller
{
    public function __construct(
        #[Auth('web')] protected Guard $auth,
        #[Cache('redis')] protected Repository $cache,
        #[Config('app.timezone')] protected string $timezone,
        #[Context('uuid')] protected string $uuid,
        #[Context('ulid', hidden: true)] protected string $ulid,
        #[DB('mysql')] protected Connection $connection,
        #[Give(DatabaseRepository::class)] protected UserRepository $users,
        #[Log('daily')] protected LoggerInterface $log,
        #[RouteParameter('photo')] protected Photo $photo,
        #[Tag('reports')] protected iterable $reports,
    ) {
        // ...
    }
}
```

Ponadto, Laravel zapewnia atrybut `CurrentUser` do wstrzykiwania aktualnie uwierzytelnionego użytkownika do danej trasy lub klasy:

```php
use App\Models\User;
use Illuminate\Container\Attributes\CurrentUser;

Route::get('/user', function (#[CurrentUser] User $user) {
    return $user;
})->middleware('auth');
```

<a name="defining-custom-attributes"></a>
#### Definiowanie Własnych Atrybutów

Możesz tworzyć własne atrybuty kontekstowe, implementując kontrakt `Illuminate\Contracts\Container\ContextualAttribute`. Kontener wywoła metodę `resolve` Twojego atrybutu, która powinna rozwiązać wartość, która powinna zostać wstrzyknięta do klasy używającej tego atrybutu. W poniższym przykładzie ponownie zaimplementujemy wbudowany atrybut `Config` Laravel:

```php
<?php

namespace App\Attributes;

use Attribute;
use Illuminate\Contracts\Container\Container;
use Illuminate\Contracts\Container\ContextualAttribute;

#[Attribute(Attribute::TARGET_PARAMETER)]
class Config implements ContextualAttribute
{
    /**
     * Create a new attribute instance.
     */
    public function __construct(public string $key, public mixed $default = null)
    {
    }

    /**
     * Resolve the configuration value.
     *
     * @param  self  $attribute
     * @param  \Illuminate\Contracts\Container\Container  $container
     * @return mixed
     */
    public static function resolve(self $attribute, Container $container)
    {
        return $container->make('config')->get($attribute->key, $attribute->default);
    }
}
```

<a name="binding-primitives"></a>
### Wiązanie Typów Prymitywnych

Czasami możesz mieć klasę, która otrzymuje niektóre wstrzyknięte klasy, ale także potrzebuje wstrzykniętej wartości prymitywnej, takiej jak liczba całkowita. Możesz łatwo użyć wiązania kontekstowego, aby wstrzyknąć dowolną wartość, której potrzebuje Twoja klasa:

```php
use App\Http\Controllers\UserController;

$this->app->when(UserController::class)
    ->needs('$variableName')
    ->give($value);
```

Czasami klasa może zależeć od tablicy [otagowanych](#tagging) instancji. Używając metody `giveTagged`, możesz łatwo wstrzyknąć wszystkie wiązania kontenera z danym tagiem:

```php
$this->app->when(ReportAggregator::class)
    ->needs('$reports')
    ->giveTagged('reports');
```

Jeśli musisz wstrzyknąć wartość z jednego z plików konfiguracyjnych aplikacji, możesz użyć metody `giveConfig`:

```php
$this->app->when(ReportAggregator::class)
    ->needs('$timezone')
    ->giveConfig('app.timezone');
```

<a name="binding-typed-variadics"></a>
### Wiązanie Zmiennej Liczby Argumentów

Okazjonalnie możesz mieć klasę, która otrzymuje tablicę obiektów danego typu przy użyciu wariadycznego argumentu konstruktora:

```php
<?php

use App\Models\Filter;
use App\Services\Logger;

class Firewall
{
    /**
     * The filter instances.
     *
     * @var array
     */
    protected $filters;

    /**
     * Create a new class instance.
     */
    public function __construct(
        protected Logger $logger,
        Filter ...$filters,
    ) {
        $this->filters = $filters;
    }
}
```

Używając wiązania kontekstowego, możesz rozwiązać tę zależność, dostarczając metodzie `give` domknięcie, które zwraca tablicę rozwiązanych instancji `Filter`:

```php
$this->app->when(Firewall::class)
    ->needs(Filter::class)
    ->give(function (Application $app) {
          return [
              $app->make(NullFilter::class),
              $app->make(ProfanityFilter::class),
              $app->make(TooLongFilter::class),
          ];
    });
```

Dla wygody, możesz również po prostu podać tablicę nazw klas do rozwiązania przez kontener, gdy `Firewall` potrzebuje instancji `Filter`:

```php
$this->app->when(Firewall::class)
    ->needs(Filter::class)
    ->give([
        NullFilter::class,
        ProfanityFilter::class,
        TooLongFilter::class,
    ]);
```

<a name="variadic-tag-dependencies"></a>
#### Zależności Wariadycznych Tagów

Czasami klasa może mieć zależność wariadyczną, która jest wskazana jako dana klasa (`Report ...$reports`). Używając metod `needs` i `giveTagged`, możesz łatwo wstrzyknąć wszystkie wiązania kontenera z danym [tagiem](#tagging) dla danej zależności:

```php
$this->app->when(ReportAggregator::class)
    ->needs(Report::class)
    ->giveTagged('reports');
```

<a name="tagging"></a>
### Tagowanie

Okazjonalnie możesz potrzebować rozwiązać wszystkie wiązania określonej "kategorii". Na przykład, może budujesz analizator raportów, który otrzymuje tablicę wielu różnych implementacji interfejsu `Report`. Po zarejestrowaniu implementacji `Report`, możesz przypisać im tag używając metody `tag`:

```php
$this->app->bind(CpuReport::class, function () {
    // ...
});

$this->app->bind(MemoryReport::class, function () {
    // ...
});

$this->app->tag([CpuReport::class, MemoryReport::class], 'reports');
```

Po otagowaniu usług, możesz łatwo rozwiązać je wszystkie za pomocą metody `tagged` kontenera:

```php
$this->app->bind(ReportAnalyzer::class, function (Application $app) {
    return new ReportAnalyzer($app->tagged('reports'));
});
```

<a name="extending-bindings"></a>
### Rozszerzanie Wiązań

Metoda `extend` pozwala na modyfikację rozwiązanych usług. Na przykład, gdy usługa jest rozwiązywana, możesz uruchomić dodatkowy kod, aby udekorować lub skonfigurować usługę. Metoda `extend` przyjmuje dwa argumenty: klasę usługi, którą rozszerzasz, oraz domknięcie, które powinno zwrócić zmodyfikowaną usługę. Domknięcie otrzymuje rozwiązywaną usługę i instancję kontenera:

```php
$this->app->extend(Service::class, function (Service $service, Application $app) {
    return new DecoratedService($service);
});
```

<a name="resolving"></a>
## Rozwiązywanie

<a name="the-make-method"></a>
### Metoda `make`

Możesz użyć metody `make`, aby rozwiązać instancję klasy z kontenera. Metoda `make` przyjmuje nazwę klasy lub interfejsu, który chcesz rozwiązać:

```php
use App\Services\Transistor;

$transistor = $this->app->make(Transistor::class);
```

Jeśli niektóre zależności Twojej klasy nie mogą zostać rozwiązane przez kontener, możesz je wstrzyknąć, przekazując je jako tablicę asocjacyjną do metody `makeWith`. Na przykład, możemy ręcznie przekazać argument konstruktora `$id` wymagany przez usługę `Transistor`:

```php
use App\Services\Transistor;

$transistor = $this->app->makeWith(Transistor::class, ['id' => 1]);
```

Metoda `bound` może być użyta do określenia, czy klasa lub interfejs został jawnie związany w kontenerze:

```php
if ($this->app->bound(Transistor::class)) {
    // ...
}
```

Jeśli jesteś poza dostawcą usług w miejscu kodu, które nie ma dostępu do zmiennej `$app`, możesz użyć [fasady](/docs/{{version}}/facades) `App` lub [helpera](/docs/{{version}}/helpers#method-app) `app`, aby rozwiązać instancję klasy z kontenera:

```php
use App\Services\Transistor;
use Illuminate\Support\Facades\App;

$transistor = App::make(Transistor::class);

$transistor = app(Transistor::class);
```

Jeśli chcesz, aby sama instancja kontenera Laravel została wstrzyknięta do klasy, która jest rozwiązywana przez kontener, mośeś określić klasę `Illuminate\Container\Container` w konstruktorze swojej klasy:

```php
use Illuminate\Container\Container;

/**
 * Create a new class instance.
 */
public function __construct(
    protected Container $container,
) {}
```

<a name="automatic-injection"></a>
### Automatyczne Wstrzykiwanie

Alternatywnie, i co ważne, możesz określić zależność w konstruktorze klasy, która jest rozwiązywana przez kontener, w tym [kontrolerów](/docs/{{version}}/controllers), [słuchaczy zdarzeń](/docs/{{version}}/events), [middleware](/docs/{{version}}/middleware) i innych. Dodatkowo, możesz określić zależności w metodzie `handle` [zadań kolejkowanych](/docs/{{version}}/queues). W praktyce, w ten sposób powinna być rozwiązywana większość Twoich obiektów przez kontener.

Na przykład, możesz określić usługę zdefiniowaą przez Twoją aplikację w konstruktorze kontrolera. Usługa zostanie automatycznie rozwiązana i wstrzyknięta do klasy:

```php
<?php

namespace App\Http\Controllers;

use App\Services\AppleMusic;

class PodcastController extends Controller
{
    /**
     * Create a new controller instance.
     */
    public function __construct(
        protected AppleMusic $apple,
    ) {}

    /**
     * Show information about the given podcast.
     */
    public function show(string $id): Podcast
    {
        return $this->apple->findPodcast($id);
    }
}
```

<a name="method-invocation-and-injection"></a>
## Wywołanie Metody i Wstrzykiwanie

Czasami możesz chcieć wywołać metodę na instancji obiektu, pozwalając kontenerowi automatycznie wstrzyknąć zależności tej metody. Na przykład, mając następującą klasę:

```php
<?php

namespace App;

use App\Services\AppleMusic;

class PodcastStats
{
    /**
     * Generate a new podcast stats report.
     */
    public function generate(AppleMusic $apple): array
    {
        return [
            // ...
        ];
    }
}
```

Możesz wywołać metodę `generate` poprzez kontener w następujący sposób:

```php
use App\PodcastStats;
use Illuminate\Support\Facades\App;

$stats = App::call([new PodcastStats, 'generate']);
```

Metoda `call` przyjmuje dowolną wywoływalność PHP. Metoda `call` kontenera może być nawet użyta do wywołania domknięcia, automatycznie wstrzykując jego zależności:

```php
use App\Services\AppleMusic;
use Illuminate\Support\Facades\App;

$result = App::call(function (AppleMusic $apple) {
    // ...
});
```

<a name="container-events"></a>
## Zdarzenia Kontenera

Kontener usług wywołuje zdarzenie za każdym razem, gdy rozwiązuje obiekt. Możesz nasłuchiwać tego zdarzenia, używając metody `resolving`:

```php
use App\Services\Transistor;
use Illuminate\Contracts\Foundation\Application;

$this->app->resolving(Transistor::class, function (Transistor $transistor, Application $app) {
    // Wywoływane, gdy kontener rozwiązuje obiekty typu "Transistor"...
});

$this->app->resolving(function (mixed $object, Application $app) {
    // Wywoływane, gdy kontener rozwiązuje obiekt dowolnego typu...
});
```

Jak widzisz, rozwiązywany obiekt zostanie przekazany do callbacku, pozwalając ustawić wszelkie dodatkowe właściwości obiektu, zanim zostanie przekazany konsumentowi.

<a name="rebinding"></a>
### Ponowne Wiązanie

Metoda `rebinding` pozwala nasłuchiwać, gdy usługa jest ponownie wiązana z kontenerem, co oznacza, że jest rejestrowana ponownie lub nadpisywana po jej początkowym wiązaniu. Może to być przydatne, gdy musisz zaktualizować zależności lub zmodyfikować zachowanie za każdym razem, gdy konkretne wiązanie jest aktualizowane:

```php
use App\Contracts\PodcastPublisher;
use App\Services\SpotifyPublisher;
use App\Services\TransistorPublisher;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(PodcastPublisher::class, SpotifyPublisher::class);

$this->app->rebinding(
    PodcastPublisher::class,
    function (Application $app, PodcastPublisher $newInstance) {
        //
    },
);

// Nowe wiązanie wywoła domknięcie rebinding...
$this->app->bind(PodcastPublisher::class, TransistorPublisher::class);
```

<a name="psr-11"></a>
## PSR-11

Kontener usług Laravel implementuje interfejs [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md). Dlatego możesz określić interfejs kontenera PSR-11, aby uzyskać instancję kontenera Laravel:

```php
use App\Services\Transistor;
use Psr\Container\ContainerInterface;

Route::get('/', function (ContainerInterface $container) {
    $service = $container->get(Transistor::class);

    // ...
});
```

Wyjątek jest rzucany, jeśli podany identyfikator nie może zostać rozwiązany. Wyjątek będzie instancją `Psr\Container\NotFoundExceptionInterface`, jeśli identyfikator nigdy nie został związany. Jeśli identyfikator został związany, ale nie mógł zostać rozwiązany, zostanie rzucona instancja `Psr\Container\ContainerExceptionInterface`.
