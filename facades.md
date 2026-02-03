# Fasady

- [Wprowadzenie](#introduction)
- [Kiedy używać fasad](#when-to-use-facades)
    - [Fasady vs. Wstrzykiwanie zależności](#facades-vs-dependency-injection)
    - [Fasady vs. Funkcje pomocnicze](#facades-vs-helper-functions)
- [Jak działają fasady](#how-facades-work)
- [Fasady w czasie rzeczywistym](#real-time-facades)
- [Dokumentacja klas fasad](#facade-class-reference)

<a name="introduction"></a>
## Wprowadzenie

W całej dokumentacji Laravel zobaczysz przykłady kodu, który wchodzi w interakcję z funkcjami Laravel za pomocą "fasad". Fasady zapewniają interfejs "statyczny" do klas, które są dostępne w [kontenerze usług](/docs/{{version}}/container) aplikacji. Laravel dostarcza wiele fasad, które zapewniają dostęp do niemal wszystkich funkcji Laravel.

Fasady Laravel służą jako "statyczne proxy" do podstawowych klas w kontenerze usług, zapewniając korzyści zwięzłej, wyrazistej składni przy jednoczesnym zachowaniu większej testowalności i elastyczności niż tradycyjne metody statyczne. Jest całkowicie w porządku, jeśli nie do końca rozumiesz, jak działają fasady - po prostu płyń z nurtem i kontynuuj naukę o Laravel.

Wszystkie fasady Laravel są zdefiniowane w przestrzeni nazw `Illuminate\Support\Facades`. Możemy więc łatwo uzyskać dostęp do fasady w następujący sposób:

```php
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Route;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

W całej dokumentacji Laravel wiele przykładów będzie używać fasad do demonstrowania różnych funkcji frameworka.

<a name="helper-functions"></a>
#### Funkcje pomocnicze

Jako uzupełnienie fasad, Laravel oferuje różnorodne globalne "funkcje pomocnicze", które ułatwiają interakcję ze wspólnymi funkcjami Laravel. Niektóre z popularnych funkcji pomocniczych, z którymi możesz się zetknąć, to `view`, `response`, `url`, `config` i wiele innych. Każda funkcja pomocnicza oferowana przez Laravel jest udokumentowana wraz z odpowiednią funkcjonalnością; jednak pełna lista jest dostępna w dedykowanej [dokumentacji pomocników](/docs/{{version}}/helpers).

Na przykład, zamiast używać fasady `Illuminate\Support\Facades\Response` do generowania odpowiedzi JSON, możemy po prostu użyć funkcji `response`. Ponieważ funkcje pomocnicze są dostępne globalnie, nie musisz importować żadnych klas, aby ich używać:

```php
use Illuminate\Support\Facades\Response;

Route::get('/users', function () {
    return Response::json([
        // ...
    ]);
});

Route::get('/users', function () {
    return response()->json([
        // ...
    ]);
});
```

<a name="when-to-use-facades"></a>
## Kiedy używać fasad

Fasady mają wiele korzyści. Zapewniają zwięzłą, łatwą do zapamiętania składnię, która pozwala korzystać z funkcji Laravel bez zapamiętywania długich nazw klas, które muszą być wstrzykiwane lub konfigurowane ręcznie. Co więcej, dzięki ich unikalnemu wykorzystaniu dynamicznych metod PHP, są łatwe do testowania.

Jednak należy zachować ostrożność podczas używania fasad. Głównym zagrożeniem związanym z fasadami jest "rozrost zakresu" klasy. Ponieważ fasady są tak łatwe w użyciu i nie wymagają wstrzykiwania, może być łatwo pozwolić, aby klasy rozrastały się i używały wielu fasad w jednej klasie. Przy użyciu wstrzykiwania zależności, ten potencjał jest łagodzony przez wizualną informację zwrotną, jaką daje Ci duży konstruktor, że Twoja klasa staje się zbyt duża. Dlatego podczas używania fasad zwracaj szczególną uwagę na rozmiar swojej klasy, aby jej zakres odpowiedzialności pozostał wąski. Jeśli Twoja klasa staje się zbyt duża, rozważ podzielenie jej na wiele mniejszych klas.

<a name="facades-vs-dependency-injection"></a>
### Fasady vs. Wstrzykiwanie zależności

Jedną z głównych korzyści wstrzykiwania zależności jest możliwość zamiany implementacji wstrzykiwanej klasy. Jest to przydatne podczas testowania, ponieważ możesz wstrzyknąć mock lub stub i potwierdzić, że różne metody zostały wywołane na stubie.

Zazwyczaj nie byłoby możliwe mockowanie lub stubowanie prawdziwie statycznej metody klasy. Jednakże, ponieważ fasady używają dynamicznych metod do przekierowywania wywołań metod do obiektów rozwiązanych z kontenera usług, możemy faktycznie testować fasady tak samo, jak testowalibyśmy instancję wstrzykiwanej klasy. Na przykład, mając następującą trasę:

```php
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

Używając metod testowania fasad Laravel, możemy napisać następujący test, aby zweryfikować, że metoda `Cache::get` została wywołana z oczekiwanym argumentem:

```php tab=Pest
use Illuminate\Support\Facades\Cache;

test('basic example', function () {
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
});
```

```php tab=PHPUnit
use Illuminate\Support\Facades\Cache;

/**
 * Podstawowy przykład testu funkcjonalnego.
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
}
```

<a name="facades-vs-helper-functions"></a>
### Fasady vs. Funkcje pomocnicze

Oprócz fasad, Laravel zawiera różnorodne funkcje "pomocnicze", które mogą wykonywać typowe zadania, takie jak generowanie widoków, wywoływanie zdarzeń, wysyłanie zadań lub wysyłanie odpowiedzi HTTP. Wiele z tych funkcji pomocniczych wykonuje tę samą funkcję co odpowiadająca im fasada. Na przykład, to wywołanie fasady i wywołanie helpera są równoważne:

```php
return Illuminate\Support\Facades\View::make('profile');

return view('profile');
```

Nie ma absolutnie żadnej praktycznej różnicy między fasadami a funkcjami pomocniczymi. Podczas używania funkcji pomocniczych, nadal możesz je testować dokładnie tak samo, jak odpowiadającą im fasadę. Na przykład, mając następującą trasę:

```php
Route::get('/cache', function () {
    return cache('key');
});
```

Helper `cache` będzie wywoływać metodę `get` na klasie bazowej fasady `Cache`. Więc, nawet jeśli używamy funkcji pomocniczej, możemy napisać następujący test, aby zweryfikować, że metoda została wywołana z oczekiwanym argumentem:

```php
use Illuminate\Support\Facades\Cache;

/**
 * Podstawowy przykład testu funkcjonalnego.
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
}
```

<a name="how-facades-work"></a>
## Jak działają fasady

W aplikacji Laravel fasada jest klasą, która zapewnia dostęp do obiektu z kontenera. Mechanizm, który sprawia, że to działa, znajduje się w klasie `Facade`. Fasady Laravel oraz wszelkie niestandardowe fasady, które utworzysz, będą rozszerzać klasę bazową `Illuminate\Support\Facades\Facade`.

Klasa bazowa `Facade` wykorzystuje magiczną metodę `__callStatic()` do odraczania wywołań z Twojej fasady do obiektu rozwiązanego z kontenera. W poniższym przykładzie dokonywane jest wywołanie do systemu cache Laravel. Patrząc na ten kod, można założyć, że statyczna metoda `get` jest wywoływana na klasie `Cache`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Pokaż profil dla danego użytkownika.
     */
    public function showProfile(string $id): View
    {
        $user = Cache::get('user:'.$id);

        return view('profile', ['user' => $user]);
    }
}
```

Zauważ, że blisko góry pliku "importujemy" fasadę `Cache`. Ta fasada służy jako proxy do uzyskiwania dostępu do podstawowej implementacji interfejsu `Illuminate\Contracts\Cache\Factory`. Wszelkie wywołania, które wykonujemy za pomocą fasady, zostaną przekazane do podstawowej instancji usługi cache Laravel.

Jeśli spojrzymy na klasę `Illuminate\Support\Facades\Cache`, zobaczysz, że nie ma tam statycznej metody `get`:

```php
class Cache extends Facade
{
    /**
     * Pobierz zarejestrowaną nazwę komponentu.
     */
    protected static function getFacadeAccessor(): string
    {
        return 'cache';
    }
}
```

Zamiast tego, fasada `Cache` rozszerza klasę bazową `Facade` i definiuje metodę `getFacadeAccessor()`. Zadaniem tej metody jest zwrócenie nazwy wiązania kontenera usług. Gdy użytkownik odwołuje się do dowolnej statycznej metody fasady `Cache`, Laravel rozwiązuje wiązanie `cache` z [kontenera usług](/docs/{{version}}/container) i uruchamia żądaną metodę (w tym przypadku `get`) na tym obiekcie.

<a name="real-time-facades"></a>
## Fasady w czasie rzeczywistym

Używając fasad w czasie rzeczywistym, możesz traktować dowolną klasę w swojej aplikacji tak, jakby była fasadą. Aby zilustrować, jak można to wykorzystać, najpierw zbadajmy kod, który nie wykorzystuje fasad czasu rzeczywistego. Na przykład załóżmy, że nasz model `Podcast` ma metodę `publish`. Jednak, aby opublikować podcast, musimy wstrzyknąć instancję `Publisher`:

```php
<?php

namespace App\Models;

use App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Opublikuj podcast.
     */
    public function publish(Publisher $publisher): void
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this);
    }
}
```

Wstrzyknięcie implementacji publishera do metody pozwala nam łatwo testować metodę w izolacji, ponieważ możemy zmockować wstrzykniętego publishera. Jednak wymaga to, aby zawsze przekazywać instancję publishera za każdym razem, gdy wywołujemy metodę `publish`. Używając fasad w czasie rzeczywistym, możemy utrzymać tę samą testowalność bez konieczności jawnego przekazywania instancji `Publisher`. Aby wygenerować fasadę w czasie rzeczywistym, przedrostek przestrzeni nazw zaimportowanej klasy z `Facades`:

```php
<?php

namespace App\Models;

use App\Contracts\Publisher; // [tl! remove]
use Facades\App\Contracts\Publisher; // [tl! add]
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Opublikuj podcast.
     */
    public function publish(Publisher $publisher): void // [tl! remove]
    public function publish(): void // [tl! add]
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this); // [tl! remove]
        Publisher::publish($this); // [tl! add]
    }
}
```

Gdy używana jest fasada w czasie rzeczywistym, implementacja publishera zostanie rozwiązana z kontenera usług przy użyciu części interfejsu lub nazwy klasy, która pojawia się po prefiksie `Facades`. Podczas testowania możemy użyć wbudowanych w Laravel helperów testowych dla fasad, aby zmockować to wywołanie metody:

```php tab=Pest
<?php

use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('podcast can be published', function () {
    $podcast = Podcast::factory()->create();

    Publisher::shouldReceive('publish')->once()->with($podcast);

    $podcast->publish();
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class PodcastTest extends TestCase
{
    use RefreshDatabase;

    /**
     * Przykład testu.
     */
    public function test_podcast_can_be_published(): void
    {
        $podcast = Podcast::factory()->create();

        Publisher::shouldReceive('publish')->once()->with($podcast);

        $podcast->publish();
    }
}
```

<a name="facade-class-reference"></a>
## Dokumentacja klas fasad

Poniżej znajdziesz każdą fasadę i jej podstawową klasę. Jest to przydatne narzędzie do szybkiego zagłębiania się w dokumentację API dla danego źródła fasady. Klucz [wiązania kontenera usług](/docs/{{version}}/container) jest również uwzględniony tam, gdzie ma to zastosowanie.

<div class="overflow-auto">

| Fasada | Klasa | Wiązanie kontenera usług |
| --- | --- | --- |
| App | [Illuminate\Foundation\Application](https://api.laravel.com/docs/{{version}}/Illuminate/Foundation/Application.html) | `app` |
| Artisan | [Illuminate\Contracts\Console\Kernel](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Console/Kernel.html) | `artisan` |
| Auth (Instancja) | [Illuminate\Contracts\Auth\Guard](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Auth/Guard.html) | `auth.driver` |
| Auth | [Illuminate\Auth\AuthManager](https://api.laravel.com/docs/{{version}}/Illuminate/Auth/AuthManager.html) | `auth` |
| Blade | [Illuminate\View\Compilers\BladeCompiler](https://api.laravel.com/docs/{{version}}/Illuminate/View/Compilers/BladeCompiler.html) | `blade.compiler` |
| Broadcast (Instancja) | [Illuminate\Contracts\Broadcasting\Broadcaster](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Broadcasting/Broadcaster.html) | &nbsp; |
| Broadcast | [Illuminate\Contracts\Broadcasting\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Broadcasting/Factory.html) | &nbsp; |
| Bus | [Illuminate\Contracts\Bus\Dispatcher](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html) | &nbsp; |
| Cache (Instancja) | [Illuminate\Cache\Repository](https://api.laravel.com/docs/{{version}}/Illuminate/Cache/Repository.html) | `cache.store` |
| Cache | [Illuminate\Cache\CacheManager](https://api.laravel.com/docs/{{version}}/Illuminate/Cache/CacheManager.html) | `cache` |
| Config | [Illuminate\Config\Repository](https://api.laravel.com/docs/{{version}}/Illuminate/Config/Repository.html) | `config` |
| Context | [Illuminate\Log\Context\Repository](https://api.laravel.com/docs/{{version}}/Illuminate/Log/Context/Repository.html) | &nbsp; |
| Cookie | [Illuminate\Cookie\CookieJar](https://api.laravel.com/docs/{{version}}/Illuminate/Cookie/CookieJar.html) | `cookie` |
| Crypt | [Illuminate\Encryption\Encrypter](https://api.laravel.com/docs/{{version}}/Illuminate/Encryption/Encrypter.html) | `encrypter` |
| Date | [Illuminate\Support\DateFactory](https://api.laravel.com/docs/{{version}}/Illuminate/Support/DateFactory.html) | `date` |
| DB (Instancja) | [Illuminate\Database\Connection](https://api.laravel.com/docs/{{version}}/Illuminate/Database/Connection.html) | `db.connection` |
| DB | [Illuminate\Database\DatabaseManager](https://api.laravel.com/docs/{{version}}/Illuminate/Database/DatabaseManager.html) | `db` |
| Event | [Illuminate\Events\Dispatcher](https://api.laravel.com/docs/{{version}}/Illuminate/Events/Dispatcher.html) | `events` |
| Exceptions (Instancja) | [Illuminate\Contracts\Debug\ExceptionHandler](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Debug/ExceptionHandler.html) | &nbsp; |
| Exceptions | [Illuminate\Foundation\Exceptions\Handler](https://api.laravel.com/docs/{{version}}/Illuminate/Foundation/Exceptions/Handler.html) | &nbsp; |
| File | [Illuminate\Filesystem\Filesystem](https://api.laravel.com/docs/{{version}}/Illuminate/Filesystem/Filesystem.html) | `files` |
| Gate | [Illuminate\Contracts\Auth\Access\Gate](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html) | &nbsp; |
| Hash | [Illuminate\Contracts\Hashing\Hasher](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Hashing/Hasher.html) | `hash` |
| Http | [Illuminate\Http\Client\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Http/Client/Factory.html) | &nbsp; |
| Lang | [Illuminate\Translation\Translator](https://api.laravel.com/docs/{{version}}/Illuminate/Translation/Translator.html) | `translator` |
| Log | [Illuminate\Log\LogManager](https://api.laravel.com/docs/{{version}}/Illuminate/Log/LogManager.html) | `log` |
| Mail | [Illuminate\Mail\Mailer](https://api.laravel.com/docs/{{version}}/Illuminate/Mail/Mailer.html) | `mailer` |
| Notification | [Illuminate\Notifications\ChannelManager](https://api.laravel.com/docs/{{version}}/Illuminate/Notifications/ChannelManager.html) | &nbsp; |
| Password (Instancja) | [Illuminate\Auth\Passwords\PasswordBroker](https://api.laravel.com/docs/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html) | `auth.password.broker` |
| Password | [Illuminate\Auth\Passwords\PasswordBrokerManager](https://api.laravel.com/docs/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html) | `auth.password` |
| Pipeline (Instancja) | [Illuminate\Pipeline\Pipeline](https://api.laravel.com/docs/{{version}}/Illuminate/Pipeline/Pipeline.html) | &nbsp; |
| Process | [Illuminate\Process\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Process/Factory.html) | &nbsp; |
| Queue (Klasa bazowa) | [Illuminate\Queue\Queue](https://api.laravel.com/docs/{{version}}/Illuminate/Queue/Queue.html) | &nbsp; |
| Queue (Instancja) | [Illuminate\Contracts\Queue\Queue](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Queue/Queue.html) | `queue.connection` |
| Queue | [Illuminate\Queue\QueueManager](https://api.laravel.com/docs/{{version}}/Illuminate/Queue/QueueManager.html) | `queue` |
| RateLimiter | [Illuminate\Cache\RateLimiter](https://api.laravel.com/docs/{{version}}/Illuminate/Cache/RateLimiter.html) | &nbsp; |
| Redirect | [Illuminate\Routing\Redirector](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/Redirector.html) | `redirect` |
| Redis (Instancja) | [Illuminate\Redis\Connections\Connection](https://api.laravel.com/docs/{{version}}/Illuminate/Redis/Connections/Connection.html) | `redis.connection` |
| Redis | [Illuminate\Redis\RedisManager](https://api.laravel.com/docs/{{version}}/Illuminate/Redis/RedisManager.html) | `redis` |
| Request | [Illuminate\Http\Request](https://api.laravel.com/docs/{{version}}/Illuminate/Http/Request.html) | `request` |
| Response (Instancja) | [Illuminate\Http\Response](https://api.laravel.com/docs/{{version}}/Illuminate/Http/Response.html) | &nbsp; |
| Response | [Illuminate\Contracts\Routing\ResponseFactory](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html) | &nbsp; |
| Route | [Illuminate\Routing\Router](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/Router.html) | `router` |
| Schedule | [Illuminate\Console\Scheduling\Schedule](https://api.laravel.com/docs/{{version}}/Illuminate/Console/Scheduling/Schedule.html) | &nbsp; |
| Schema | [Illuminate\Database\Schema\Builder](https://api.laravel.com/docs/{{version}}/Illuminate/Database/Schema/Builder.html) | &nbsp; |
| Session (Instancja) | [Illuminate\Session\Store](https://api.laravel.com/docs/{{version}}/Illuminate/Session/Store.html) | `session.store` |
| Session | [Illuminate\Session\SessionManager](https://api.laravel.com/docs/{{version}}/Illuminate/Session/SessionManager.html) | `session` |
| Storage (Instancja) | [Illuminate\Contracts\Filesystem\Filesystem](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Filesystem/Filesystem.html) | `filesystem.disk` |
| Storage | [Illuminate\Filesystem\FilesystemManager](https://api.laravel.com/docs/{{version}}/Illuminate/Filesystem/FilesystemManager.html) | `filesystem` |
| URL | [Illuminate\Routing\UrlGenerator](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/UrlGenerator.html) | `url` |
| Validator (Instancja) | [Illuminate\Validation\Validator](https://api.laravel.com/docs/{{version}}/Illuminate/Validation/Validator.html) | &nbsp; |
| Validator | [Illuminate\Validation\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Validation/Factory.html) | `validator` |
| View (Instancja) | [Illuminate\View\View](https://api.laravel.com/docs/{{version}}/Illuminate/View/View.html) | &nbsp; |
| View | [Illuminate\View\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/View/Factory.html) | `view` |
| Vite | [Illuminate\Foundation\Vite](https://api.laravel.com/docs/{{version}}/Illuminate/Foundation/Vite.html) | &nbsp; |

</div>
