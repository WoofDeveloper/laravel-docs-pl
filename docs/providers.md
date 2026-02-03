# Dostawcy Usług

- [Wprowadzenie](#introduction)
- [Pisanie Dostawców Usług](#writing-service-providers)
    - [Metoda Register](#the-register-method)
    - [Metoda Boot](#the-boot-method)
- [Rejestrowanie Dostawców](#registering-providers)
- [Dostawcy Odroczeni](#deferred-providers)

<a name="introduction"></a>
## Wprowadzenie

Dostawcy usług są centralnym miejscem uruchamiania całej aplikacji Laravel. Twoja własna aplikacja, jak również wszystkie podstawowe usługi Laravel, są uruchamiane za pomocą dostawców usług.

Ale co mamy na myśli przez "uruchamianie"? Ogólnie rzecz biorąc, mamy na myśli **rejestrowanie** rzeczy, w tym rejestrowanie powiązań kontenera usług, nasłuchiwaczy zdarzeń, middleware, a nawet tras. Dostawcy usług są centralnym miejscem do konfigurowania aplikacji.

Laravel używa wewnętrznie dziesiątków dostawców usług do uruchamiania swoich podstawowych usług, takich jak mailer, kolejka, pamięć podręczna i inne. Wiele z tych dostawców to dostawcy "odroczeni", co oznacza, że nie będą ładowani przy każdym żądaniu, ale tylko wtedy, gdy usługi, które dostarczają, są rzeczywiście potrzebne.

Wszyscy zdefiniowani przez użytkownika dostawcy usług są rejestrowani w pliku `bootstrap/providers.php`. W poniższej dokumentacji dowiesz się, jak pisać własnych dostawców usług i rejestrować ich w aplikacji Laravel.

> [!NOTE]
> Jeśli chcesz dowiedzieć się więcej o tym, jak Laravel obsługuje żądania i działa wewnętrznie, sprawdź naszą dokumentację na temat [cyklu życia żądania](/docs/{{version}}/lifecycle) Laravel.

<a name="writing-service-providers"></a>
## Pisanie Dostawców Usług

Wszyscy dostawcy usług rozszerzają klasę `Illuminate\Support\ServiceProvider`. Większość dostawców usług zawiera metodę `register` i `boot`. W metodzie `register` powinieneś **tylko wiązać rzeczy do [kontenera usług](/docs/{{version}}/container)**. Nigdy nie powinieneś próbować rejestrować żadnych nasłuchiwaczy zdarzeń, tras ani żadnej innej funkcjonalności w metodzie `register`.

Artisan CLI może wygenerować nowego dostawcę za pomocą polecenia `make:provider`. Laravel automatycznie zarejestruje twojego nowego dostawcę w pliku `bootstrap/providers.php` aplikacji:

```shell
php artisan make:provider RiakServiceProvider
```

<a name="the-register-method"></a>
### Metoda Register

Jak wspomniano wcześniej, w metodzie `register` powinieneś tylko wiązać rzeczy do [kontenera usług](/docs/{{version}}/container). Nigdy nie powinieneś próbować rejestrować żadnych nasłuchiwaczy zdarzeń, tras ani żadnej innej funkcjonalności w metodzie `register`. W przeciwnym razie możesz przypadkowo użyć usługi, która jest dostarczana przez dostawcę usług, który jeszcze nie został załadowany.

Spójrzmy na podstawowego dostawcę usług. W ramach dowolnej z metod twojego dostawcy usług zawsze masz dostęp do właściwości `$app`, która zapewnia dostęp do kontenera usług:

```php
<?php

namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection(config('riak'));
        });
    }
}
```

Ten dostawca usług definiuje tylko metodę `register` i używa tej metody do zdefiniowania implementacji `App\Services\Riak\Connection` w kontenerze usług. Jeśli nie znasz jeszcze kontenera usług Laravel, sprawdź [jego dokumentację](/docs/{{version}}/container).

<a name="the-bindings-and-singletons-properties"></a>
#### Właściwości `bindings` i `singletons`

Jeśli twój dostawca usług rejestruje wiele prostych powiązań, możesz użyć właściwości `bindings` i `singletons` zamiast ręcznego rejestrowania każdego powiązania kontenera. Gdy dostawca usług jest ładowany przez framework, automatycznie sprawdzi te właściwości i zarejestruje ich powiązania:

```php
<?php

namespace App\Providers;

use App\Contracts\DowntimeNotifier;
use App\Contracts\ServerProvider;
use App\Services\DigitalOceanServerProvider;
use App\Services\PingdomDowntimeNotifier;
use App\Services\ServerToolsProvider;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * All of the container bindings that should be registered.
     *
     * @var array
     */
    public $bindings = [
        ServerProvider::class => DigitalOceanServerProvider::class,
    ];

    /**
     * All of the container singletons that should be registered.
     *
     * @var array
     */
    public $singletons = [
        DowntimeNotifier::class => PingdomDowntimeNotifier::class,
        ServerProvider::class => ServerToolsProvider::class,
    ];
}
```

<a name="the-boot-method"></a>
### Metoda Boot

A co, jeśli musimy zarejestrować [komponowalny widok](/docs/{{version}}/views#view-composers) w naszym dostawcy usług? Powinno to być zrobione w metodzie `boot`. **Ta metoda jest wywoływana po zarejestrowaniu wszystkich innych dostawców usług**, co oznacza, że masz dostęp do wszystkich innych usług, które zostały zarejestrowane przez framework:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        View::composer('view', function () {
            // ...
        });
    }
}
```

<a name="boot-method-dependency-injection"></a>
#### Wstrzykiwanie Zależności w Metodzie Boot

Możesz określić typy zależności dla metody `boot` twojego dostawcy usług. [Kontener usług](/docs/{{version}}/container) automatycznie wstrzyknie wszelkie potrzebne zależności:

```php
use Illuminate\Contracts\Routing\ResponseFactory;

/**
 * Bootstrap any application services.
 */
public function boot(ResponseFactory $response): void
{
    $response->macro('serialized', function (mixed $value) {
        // ...
    });
}
```

<a name="registering-providers"></a>
## Rejestrowanie Dostawców

Wszyscy dostawcy usług są rejestrowani w pliku konfiguracyjnym `bootstrap/providers.php`. Ten plik zwraca tablicę zawierającą nazwy klas dostawców usług twojej aplikacji:

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
];
```

Gdy wywołujesz polecenie Artisan `make:provider`, Laravel automatycznie doda wygenerowanego dostawcę do pliku `bootstrap/providers.php`. Jednak jeśli ręcznie utworzyłeś klasę dostawcy, powinieneś ręcznie dodać klasę dostawcy do tablicy:

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
    App\Providers\ComposerServiceProvider::class, // [tl! add]
];
```

<a name="deferred-providers"></a>
## Dostawcy Odroczeni

Jeśli twój dostawca **tylko** rejestruje powiązania w [kontenerze usług](/docs/{{version}}/container), możesz zdecydować się na odroczenie jego rejestracji do momentu, gdy jedno z zarejestrowanych powiązań będzie rzeczywiście potrzebne. Odroczenie ładowania takiego dostawcy poprawi wydajność twojej aplikacji, ponieważ nie jest ładowany z systemu plików przy każdym żądaniu.

Laravel kompiluje i przechowuje listę wszystkich usług dostarczanych przez odroczonych dostawców usług, wraz z nazwą klasy jego dostawcy usług. Następnie tylko wtedy, gdy próbujesz rozwiązać jedną z tych usług, Laravel ładuje dostawcę usług.

Aby odroczyć ładowanie dostawcy, zaimplementuj interfejs `\Illuminate\Contracts\Support\DeferrableProvider` i zdefiniuj metodę `provides`. Metoda `provides` powinna zwracać powiązania kontenera usług zarejestrowane przez dostawcę:

```php
<?php

namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Contracts\Support\DeferrableProvider;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection($app['config']['riak']);
        });
    }

    /**
     * Get the services provided by the provider.
     *
     * @return array<int, string>
     */
    public function provides(): array
    {
        return [Connection::class];
    }
}
```
