# Mockowanie

- [Wprowadzenie](#introduction)
- [Mockowanie obiektów](#mocking-objects)
- [Mockowanie fasad](#mocking-facades)
    - [Szpiegowanie fasad](#facade-spies)
- [Interakcja z czasem](#interacting-with-time)

<a name="introduction"></a>
## Wprowadzenie

Podczas testowania aplikacji Laravel możesz chcieć "zamockować" pewne aspekty swojej aplikacji, aby nie były one faktycznie wykonywane podczas danego testu. Na przykład, podczas testowania kontrolera, który wysyła zdarzenie, możesz chcieć zamockować nasłuchiwacze zdarzeń, aby nie były one faktycznie wykonywane podczas testu. Pozwala to na testowanie tylko odpowiedzi HTTP kontrolera bez martwienia się o wykonanie nasłuchiwaczy zdarzeń, ponieważ nasłuchiwacze zdarzeń mogą być testowane w ich własnych przypadkach testowych.

Laravel dostarcza pomocne metody do mockowania zdarzeń, zadań i innych fasad od razu po wyjęciu z pudełka. Te helpery przede wszystkim zapewniają warstwę wygody nad Mockery, dzięki czemu nie musisz ręcznie wykonywać skomplikowanych wywołań metod Mockery.

<a name="mocking-objects"></a>
## Mockowanie obiektów

Podczas mockowania obiektu, który będzie wstrzykiwany do Twojej aplikacji za pośrednictwem [kontenera usług](/docs/{{version}}/container) Laravel, będziesz musiał powiązać swoją zamockowaną instancję z kontenerem jako wiązanie `instance`. Spowoduje to, że kontener użyje Twojej zamockowanej instancji obiektu zamiast konstruować sam obiekt:

```php tab=Pest
use App\Service;
use Mockery;
use Mockery\MockInterface;

test('something can be mocked', function () {
    $this->instance(
        Service::class,
        Mockery::mock(Service::class, function (MockInterface $mock) {
            $mock->expects('process');
        })
    );
});
```

```php tab=PHPUnit
use App\Service;
use Mockery;
use Mockery\MockInterface;

public function test_something_can_be_mocked(): void
{
    $this->instance(
        Service::class,
        Mockery::mock(Service::class, function (MockInterface $mock) {
            $mock->expects('process');
        })
    );
}
```

Aby uczynić to wygodniejszym, możesz użyć metody `mock` dostarczanej przez bazową klasę przypadku testowego Laravel. Na przykład, poniższy przykład jest równoważny z powyższym przykładem:

```php
use App\Service;
use Mockery\MockInterface;

$mock = $this->mock(Service::class, function (MockInterface $mock) {
    $mock->expects('process');
});
```

Możesz użyć metody `partialMock`, gdy musisz zamockować tylko kilka metod obiektu. Metody, które nie są zamockowane, będą wykonywane normalnie po wywołaniu:

```php
use App\Service;
use Mockery\MockInterface;

$mock = $this->partialMock(Service::class, function (MockInterface $mock) {
    $mock->expects('process');
});
```

Podobnie, jeśli chcesz [szpiegować](http://docs.mockery.io/en/latest/reference/spies.html) obiekt, bazowa klasa przypadku testowego Laravel oferuje metodę `spy` jako wygodną nakładkę na metodę `Mockery::spy`. Szpiedzy są podobni do mocków; jednak szpiedzy rejestrują każdą interakcję między szpiegiem a testowanym kodem, pozwalając Ci na wykonanie asercji po wykonaniu kodu:

```php
use App\Service;

$spy = $this->spy(Service::class);

// ...

$spy->shouldHaveReceived('process');
```

<a name="mocking-facades"></a>
## Mockowanie fasad

W przeciwieństwie do tradycyjnych wywołań metod statycznych, [fasady](/docs/{{version}}/facades) (w tym [fasady w czasie rzeczywistym](/docs/{{version}}/facades#real-time-facades)) mogą być mockowane. Daje to ogromną przewagę nad tradycyjnymi metodami statycznymi i zapewnia taką samą testowalność, jaką miałbyś, gdybyś używał tradycyjnego wstrzykiwania zależności. Podczas testowania często możesz chcieć zamockować wywołanie fasady Laravel, które występuje w jednym z Twoich kontrolerów. Na przykład, rozważ następującą akcję kontrolera:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * Retrieve a list of all users of the application.
     */
    public function index(): array
    {
        $value = Cache::get('key');

        return [
            // ...
        ];
    }
}
```

Możemy zamockować wywołanie fasady `Cache` za pomocą metody `expects`, która zwróci instancję mocka [Mockery](https://github.com/padraic/mockery). Ponieważ fasady są faktycznie rozwiązywane i zarządzane przez [kontener usług](/docs/{{version}}/container) Laravel, mają one znacznie większą testowalność niż typowa klasa statyczna. Na przykład zamockujmy nasze wywołanie metody `get` fasady `Cache`:

```php tab=Pest
<?php

use Illuminate\Support\Facades\Cache;

test('get index', function () {
    Cache::expects('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/users');

    // ...
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Illuminate\Support\Facades\Cache;
use Tests\TestCase;

class UserControllerTest extends TestCase
{
    public function test_get_index(): void
    {
        Cache::expects('get')
            ->with('key')
            ->andReturn('value');

        $response = $this->get('/users');

        // ...
    }
}
```

> [!WARNING]
> Nie powinieneś mockować fasady `Request`. Zamiast tego przekaż dane wejściowe, których potrzebujesz, do [metod testowania HTTP](/docs/{{version}}/http-tests) takich jak `get` i `post` podczas uruchamiania testu. Podobnie, zamiast mockować fasadę `Config`, wywołaj metodę `Config::set` w swoich testach.

<a name="facade-spies"></a>
### Szpiegowanie fasad

Jeśli chcesz [szpiegować](http://docs.mockery.io/en/latest/reference/spies.html) fasadę, możesz wywołać metodę `spy` na odpowiedniej fasadzie. Szpiedzy są podobni do mocków; jednak szpiedzy rejestrują każdą interakcję między szpiegiem a testowanym kodem, pozwalając Ci na wykonanie asercji po wykonaniu kodu:

```php tab=Pest
<?php

use Illuminate\Support\Facades\Cache;

test('values are stored in cache', function () {
    Cache::spy();

    $response = $this->get('/');

    $response->assertStatus(200);

    Cache::shouldHaveReceived('put')->with('name', 'Taylor', 10);
});
```

```php tab=PHPUnit
use Illuminate\Support\Facades\Cache;

public function test_values_are_stored_in_cache(): void
{
    Cache::spy();

    $response = $this->get('/');

    $response->assertStatus(200);

    Cache::shouldHaveReceived('put')->with('name', 'Taylor', 10);
}
```

<a name="interacting-with-time"></a>
## Interakcja z czasem

Podczas testowania możesz czasami potrzebować zmodyfikować czas zwracany przez helpery takie jak `now` lub `Illuminate\Support\Carbon::now()`. Na szczęście bazowa klasa testów funkcjonalnych Laravel zawiera helpery, które pozwalają Ci manipulować bieżącym czasem:

```php tab=Pest
test('time can be manipulated', function () {
    // Podróż w przyszłość...
    $this->travel(5)->milliseconds();
    $this->travel(5)->seconds();
    $this->travel(5)->minutes();
    $this->travel(5)->hours();
    $this->travel(5)->days();
    $this->travel(5)->weeks();
    $this->travel(5)->years();

    // Podróż w przeszłość...
    $this->travel(-5)->hours();

    // Podróż do konkretnego czasu...
    $this->travelTo(now()->minus(hours: 6));

    // Powrót do obecnego czasu...
    $this->travelBack();
});
```

```php tab=PHPUnit
public function test_time_can_be_manipulated(): void
{
    // Podróż w przyszłość...
    $this->travel(5)->milliseconds();
    $this->travel(5)->seconds();
    $this->travel(5)->minutes();
    $this->travel(5)->hours();
    $this->travel(5)->days();
    $this->travel(5)->weeks();
    $this->travel(5)->years();

    // Podróż w przeszłość...
    $this->travel(-5)->hours();

    // Podróż do konkretnego czasu...
    $this->travelTo(now()->minus(hours: 6));

    // Powrót do obecnego czasu...
    $this->travelBack();
}
```

Możesz również przekazać closure do różnych metod podróży w czasie. Closure zostanie wywołane z zamrożonym czasem w określonym czasie. Po wykonaniu closure czas będzie kontynuowany normalnie:

```php
$this->travel(5)->days(function () {
    // Test something five days into the future...
});

$this->travelTo(now()->mins(days: 10), function () {
    // Test something during a given moment...
});
```

Metoda `freezeTime` może być używana do zamrożenia bieżącego czasu. Podobnie, metoda `freezeSecond` zamrozi bieżący czas, ale na początku bieżącej sekundy:

```php
use Illuminate\Support\Carbon;

// Zamroź czas i wznów normalny czas po wykonaniu closure...
$this->freezeTime(function (Carbon $time) {
    // ...
});

// Zamroź czas na początku bieżącej sekundy i wznów normalny czas po wykonaniu closure...
$this->freezeSecond(function (Carbon $time) {
    // ...
})
```

Jak można się spodziewać, wszystkie omówione powyżej metody są przede wszystkim przydatne do testowania zachowania aplikacji wrażliwych na czas, takiego jak blokowanie nieaktywnych postów na forum dyskusyjnym:

```php tab=Pest
use App\Models\Thread;

test('forum threads lock after one week of inactivity', function () {
    $thread = Thread::factory()->create();

    $this->travel(1)->week();

    expect($thread->isLockedByInactivity())->toBeTrue();
});
```

```php tab=PHPUnit
use App\Models\Thread;

public function test_forum_threads_lock_after_one_week_of_inactivity()
{
    $thread = Thread::factory()->create();

    $this->travel(1)->week();

    $this->assertTrue($thread->isLockedByInactivity());
}
```
