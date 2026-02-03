# Testowanie: Pierwsze Kroki

- [Wprowadzenie](#introduction)
- [Środowisko](#environment)
- [Tworzenie Testów](#creating-tests)
- [Uruchamianie Testów](#running-tests)
    - [Równoległe Uruchamianie Testów](#running-tests-in-parallel)
    - [Raportowanie Pokrycia Testami](#reporting-test-coverage)
    - [Profilowanie Testów](#profiling-tests)
- [Cachowanie Konfiguracji](#configuration-caching)

<a name="introduction"></a>
## Wprowadzenie

Laravel został zbudowany z myślą o testowaniu. W rzeczywistości, wsparcie dla testowania z [Pest](https://pestphp.com) i [PHPUnit](https://phpunit.de) jest dołączone od razu, a plik `phpunit.xml` jest już skonfigurowany dla Twojej aplikacji. Framework zawiera również wygodne metody pomocnicze, które pozwalają na ekspresyjne testowanie Twoich aplikacji.

Domyślnie katalog `tests` Twojej aplikacji zawiera dwa katalogi: `Feature` i `Unit`. Testy jednostkowe to testy, które skupiają się na bardzo małej, izolowanej części Twojego kodu. W rzeczywistości, większość testów jednostkowych prawdopodobnie skupia się na pojedynczej metodzie. Testy w katalogu "Unit" nie uruchamiają Twojej aplikacji Laravel i dlatego nie mają dostępu do bazy danych aplikacji ani innych usług frameworka.

Testy funkcjonalne mogą testować większą część Twojego kodu, w tym jak kilka obiektów współdziała ze sobą lub nawet pełne żądanie HTTP do endpointu JSON. **Generalnie, większość Twoich testów powinna być testami funkcjonalnymi. Te typy testów zapewniają największą pewność, że Twój system jako całość działa zgodnie z zamierzeniami.**

Plik `ExampleTest.php` jest dostarczony zarówno w katalogach testowych `Feature`, jak i `Unit`. Po zainstalowaniu nowej aplikacji Laravel, wykonaj polecenia `vendor/bin/pest`, `vendor/bin/phpunit` lub `php artisan test`, aby uruchomić swoje testy.

<a name="environment"></a>
## Środowisko

Podczas uruchamiania testów, Laravel automatycznie ustawi [środowisko konfiguracji](/docs/{{version}}/configuration#environment-configuration) na `testing` ze względu na zmienne środowiskowe zdefiniowane w pliku `phpunit.xml`. Laravel również automatycznie konfiguruje sesję i cache na sterownik `array`, dzięki czemu żadne dane sesji ani cache nie będą zachowywane podczas testowania.

Możesz swobodnie definiować inne wartości konfiguracji środowiska testowego, jeśli to konieczne. Zmienne środowiskowe `testing` mogą być skonfigurowane w pliku `phpunit.xml` Twojej aplikacji, ale upewnij się, że wyczyścisz cache konfiguracji używając polecenia Artisan `config:clear` przed uruchomieniem testów!

<a name="the-env-testing-environment-file"></a>
#### Plik Środowiska `.env.testing`

Dodatkowo możesz utworzyć plik `.env.testing` w katalogu głównym Twojego projektu. Ten plik będzie używany zamiast pliku `.env` podczas uruchamiania testów Pest i PHPUnit lub wykonywania poleceń Artisan z opcją `--env=testing`.

<a name="creating-tests"></a>
## Tworzenie Testów

Aby utworzyć nowy przypadek testowy, użyj polecenia Artisan `make:test`. Domyślnie testy zostaną umieszczone w katalogu `tests/Feature`:

```shell
php artisan make:test UserTest
```

Jeśli chcesz utworzyć test w katalogu `tests/Unit`, możesz użyć opcji `--unit` podczas wykonywania polecenia `make:test`:

```shell
php artisan make:test UserTest --unit
```

> [!NOTE]
> Szablony testów mogą być dostosowane za pomocą [publikowania szablonów](/docs/{{version}}/artisan#stub-customization).

Po wygenerowaniu testu możesz zdefiniować test tak, jak normalnie robisz to za pomocą Pest lub PHPUnit. Aby uruchomić testy, wykonaj polecenie `vendor/bin/pest`, `vendor/bin/phpunit` lub `php artisan test` z terminala:

```php tab=Pest
<?php

test('basic', function () {
    expect(true)->toBeTrue();
});
```

```php tab=PHPUnit
<?php

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     */
    public function test_basic_test(): void
    {
        $this->assertTrue(true);
    }
}
```

> [!WARNING]
> Jeśli definiujesz własne metody `setUp` / `tearDown` w klasie testowej, upewnij się, że wywołujesz odpowiednie metody `parent::setUp()` / `parent::tearDown()` w klasie nadrzędnej. Zazwyczaj powinieneś wywołać `parent::setUp()` na początku własnej metody `setUp`, a `parent::tearDown()` na końcu swojej metody `tearDown`.

<a name="running-tests"></a>
## Uruchamianie Testów

Jak wspomniano wcześniej, po napisaniu testów możesz je uruchomić za pomocą `pest` lub `phpunit`:

```shell tab=Pest
./vendor/bin/pest
```

```shell tab=PHPUnit
./vendor/bin/phpunit
```

Oprócz poleceń `pest` lub `phpunit`, możesz użyć polecenia Artisan `test` do uruchamiania testów. Runner testów Artisan zapewnia szczegółowe raporty testów, aby ułatwić rozwój i debugowanie:

```shell
php artisan test
```

Wszystkie argumenty, które mogą być przekazane do poleceń `pest` lub `phpunit`, mogą być również przekazane do polecenia Artisan `test`:

```shell
php artisan test --testsuite=Feature --stop-on-failure
```

<a name="running-tests-in-parallel"></a>
### Równoległe Uruchamianie Testów

Domyślnie Laravel i Pest / PHPUnit wykonują testy sekwencyjnie w pojedynczym procesie. Możesz jednak znacznie skrócić czas potrzebny na uruchomienie testów, uruchamiając testy jednocześnie w wielu procesach. Aby rozpocząć, zainstaluj pakiet Composer `brianium/paratest` jako zależność "dev". Następnie dołącz opcję `--parallel` podczas wykonywania polecenia Artisan `test`:

```shell
composer require brianium/paratest --dev

php artisan test --parallel
```

Domyślnie Laravel utworzy tyle procesów, ile jest dostępnych rdzeni CPU na Twoim komputerze. Możesz jednak dostosować liczbę procesów używając opcji `--processes`:

```shell
php artisan test --parallel --processes=4
```

> [!WARNING]
> Podczas uruchamiania testów równolegle, niektóre opcje Pest / PHPUnit (takie jak `--do-not-cache-result`) mogą być niedostępne.

<a name="parallel-testing-and-databases"></a>
#### Testowanie Równoległe i Bazy Danych

O ile masz skonfigurowane główne połączenie z bazą danych, Laravel automatycznie obsługuje tworzenie i migrację bazy danych testowej dla każdego równoległego procesu, który uruchamia Twoje testy. Bazy danych testowe będą miały sufiks z tokenem procesu, który jest unikalny dla każdego procesu. Na przykład, jeśli masz dwa równoległe procesy testowe, Laravel utworzy i użyje baz danych testowych `your_db_test_1` i `your_db_test_2`.

Domyślnie bazy danych testowe utrzymują się między wywołaniami polecenia Artisan `test`, aby mogły być ponownie użyte przez kolejne wywołania `test`. Możesz jednak je odtworzyć używając opcji `--recreate-databases`:

```shell
php artisan test --parallel --recreate-databases
```

<a name="parallel-testing-hooks"></a>
#### Hooki Testowania Równoległego

Czasami możesz potrzebować przygotować pewne zasoby używane przez testy Twojej aplikacji, aby mogły być bezpiecznie używane przez wiele procesów testowych.

Używając fasady `ParallelTesting`, możesz określić kod, który ma być wykonany podczas `setUp` i `tearDown` procesu lub przypadku testowego. Podane domknięcia otrzymują zmienne `$token` i `$testCase`, które zawierają odpowiednio token procesu i bieżący przypadek testowy:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Artisan;
use Illuminate\Support\Facades\ParallelTesting;
use Illuminate\Support\ServiceProvider;
use PHPUnit\Framework\TestCase;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        ParallelTesting::setUpProcess(function (int $token) {
            // ...
        });

        ParallelTesting::setUpTestCase(function (int $token, TestCase $testCase) {
            // ...
        });

        // Wykonywane podczas tworzenia bazy danych testowej...
        ParallelTesting::setUpTestDatabase(function (string $database, int $token) {
            Artisan::call('db:seed');
        });

        ParallelTesting::tearDownTestCase(function (int $token, TestCase $testCase) {
            // ...
        });

        ParallelTesting::tearDownProcess(function (int $token) {
            // ...
        });
    }
}
```

<a name="accessing-the-parallel-testing-token"></a>
#### Dostęp do Tokenu Testowania Równoległego

Jeśli chcesz uzyskać dostęp do "tokenu" bieżącego procesu równoległego z dowolnego innego miejsca w kodzie testowym Twojej aplikacji, możesz użyć metody `token`. Ten token jest unikalnym identyfikatorem ciągu dla pojedynczego procesu testowego i może być użyty do segmentacji zasobów w procesach testowych równoległych. Na przykład, Laravel automatycznie dodaje ten token na końcu baz danych testowych tworzonych przez każdy proces testowania równoległego:

    $token = ParallelTesting::token();

<a name="reporting-test-coverage"></a>
### Raportowanie Pokrycia Testami

> [!WARNING]
> Ta funkcja wymaga [Xdebug](https://xdebug.org) lub [PCOV](https://pecl.php.net/package/pcov).

Podczas uruchamiania testów aplikacji możesz chcieć określić, czy Twoje przypadki testowe rzeczywiście pokrywają kod aplikacji i ile kodu aplikacji jest używane podczas uruchamiania testów. Aby to osiągnąć, możesz podać opcję `--coverage` podczas wywoływania polecenia `test`:

```shell
php artisan test --coverage
```

<a name="enforcing-a-minimum-coverage-threshold"></a>
#### Wymuszanie Minimalnego Progu Pokrycia

Możesz użyć opcji `--min`, aby zdefiniować minimalny próg pokrycia testami dla Twojej aplikacji. Zestaw testów zakończy się niepowodzeniem, jeśli ten próg nie zostanie osiągnięty:

```shell
php artisan test --coverage --min=80.3
```

<a name="profiling-tests"></a>
### Profilowanie Testów

Runner testów Artisan zawiera również wygodny mechanizm do wyświetlania najwolniejszych testów Twojej aplikacji. Wywołaj polecenie `test` z opcją `--profile`, aby otrzymać listę dziesięciu najwolniejszych testów, co pozwala łatwo zbadać, które testy można ulepszyć, aby przyspieszyć Twój zestaw testów:

```shell
php artisan test --profile
```

<a name="configuration-caching"></a>
## Cachowanie Konfiguracji

Podczas uruchamiania testów Laravel uruchamia aplikację dla każdej indywidualnej metody testowej. Bez cachowanego pliku konfiguracji, każdy plik konfiguracyjny w Twojej aplikacji musi być załadowany na początku testu. Aby zbudować konfigurację raz i użyć jej ponownie dla wszystkich testów w jednym uruchomieniu, możesz użyć traitu `Illuminate\Foundation\Testing\WithCachedConfig`:

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\WithCachedConfig;

pest()->use(WithCachedConfig::class);

// ...
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\WithCachedConfig;
use Tests\TestCase;

class ConfigTest extends TestCase
{
    use WithCachedConfig;

    // ...
}
```
