# Testowanie Bazy Danych

- [Wprowadzenie](#introduction)
    - [Resetowanie Bazy Danych Po Każdym Teście](#resetting-the-database-after-each-test)
- [Fabryki Modeli](#model-factories)
- [Uruchamianie Seederów](#running-seeders)
- [Dostępne Asercje](#available-assertions)

<a name="introduction"></a>
## Wprowadzenie

Laravel dostarcza różnorodne przydatne narzędzia i asercje, które ułatwiają testowanie aplikacji opartych na bazie danych. Dodatkowo, fabryki modeli Laravel i seedery sprawiają, że tworzenie testowych rekordów bazy danych przy użyciu modeli Eloquent i relacji jest bezbolesne. Omówimy wszystkie te potężne funkcje w poniższej dokumentacji.

<a name="resetting-the-database-after-each-test"></a>
### Resetowanie Bazy Danych Po Każdym Teście

Zanim przejdziemy dalej, omówmy sposób resetowania bazy danych po każdym z Twoich testów, aby dane z poprzedniego testu nie zakłócały kolejnych testów. Dołączona w Laravel cecha `Illuminate\Foundation\Testing\RefreshDatabase` zajmie się tym za Ciebie. Po prostu użyj tej cechy w swojej klasie testowej:

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('podstawowy przykład', function () {
    $response = $this->get('/');

    // ...
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * Podstawowy przykład testu funkcjonalnego.
     */
    public function test_basic_example(): void
    {
        $response = $this->get('/');

        // ...
    }
}
```

Cecha `Illuminate\Foundation\Testing\RefreshDatabase` nie migruje Twojej bazy danych, jeśli Twój schemat jest aktualny. Zamiast tego, wykona test tylko w ramach transakcji bazy danych. W związku z tym, wszelkie rekordy dodane do bazy danych przez przypadki testowe, które nie używają tej cechy, mogą nadal istnieć w bazie danych.

Jeśli chcesz całkowicie zresetować bazę danych, możesz użyć cech `Illuminate\Foundation\Testing\DatabaseMigrations` lub `Illuminate\Foundation\Testing\DatabaseTruncation`. Jednak obie te opcje są znacznie wolniejsze niż cecha `RefreshDatabase`.

<a name="model-factories"></a>
## Fabryki Modeli

Podczas testowania możesz potrzebować wstawić kilka rekordów do bazy danych przed wykonaniem testu. Zamiast ręcznego określania wartości każdej kolumny podczas tworzenia tych danych testowych, Laravel pozwala zdefiniować zestaw domyślnych atrybutów dla każdego z Twoich [modeli Eloquent](/docs/{{version}}/eloquent) za pomocą [fabryk modeli](/docs/{{version}}/eloquent-factories).

Aby dowiedzieć się więcej o tworzeniu i wykorzystywaniu fabryk modeli do tworzenia modeli, zapoznaj się z pełną [dokumentacją fabryk modeli](/docs/{{version}}/eloquent-factories). Gdy już zdefiniujesz fabrykę modelu, możesz wykorzystać fabrykę w swoim teście do tworzenia modeli:

```php tab=Pest
use App\Models\User;

test('modele mogą być tworzone', function () {
    $user = User::factory()->create();

    // ...
});
```

```php tab=PHPUnit
use App\Models\User;

public function test_models_can_be_instantiated(): void
{
    $user = User::factory()->create();

    // ...
}
```

<a name="running-seeders"></a>
## Uruchamianie Seederów

Jeśli chcesz użyć [seederów bazy danych](/docs/{{version}}/seeding) do wypełnienia bazy danych podczas testu funkcjonalnego, możesz wywołać metodę `seed`. Domyślnie metoda `seed` wykona `DatabaseSeeder`, który powinien wykonać wszystkie Twoje pozostałe seedery. Alternatywnie, możesz przekazać nazwę konkretnej klasy seedera do metody `seed`:

```php tab=Pest
<?php

use Database\Seeders\OrderStatusSeeder;
use Database\Seeders\TransactionStatusSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('zamówienia mogą być tworzone', function () {
    // Uruchom DatabaseSeeder...
    $this->seed();

    // Uruchom konkretny seeder...
    $this->seed(OrderStatusSeeder::class);

    // ...

    // Uruchom tablicę konkretnych seederów...
    $this->seed([
        OrderStatusSeeder::class,
        TransactionStatusSeeder::class,
        // ...
    ]);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Database\Seeders\OrderStatusSeeder;
use Database\Seeders\TransactionStatusSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * Test tworzenia nowego zamówienia.
     */
    public function test_orders_can_be_created(): void
    {
        // Uruchom DatabaseSeeder...
        $this->seed();

        // Uruchom konkretny seeder...
        $this->seed(OrderStatusSeeder::class);

        // ...

        // Uruchom tablicę konkretnych seederów...
        $this->seed([
            OrderStatusSeeder::class,
            TransactionStatusSeeder::class,
            // ...
        ]);
    }
}
```

Alternatywnie, możesz poinstruować Laravel, aby automatycznie wypełniał bazę danych przed każdym testem, który używa cechy `RefreshDatabase`. Możesz to osiągnąć, definiując właściwość `$seed` w swojej bazowej klasie testowej:

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    /**
     * Wskazuje, czy domyślny seeder powinien być uruchamiany przed każdym testem.
     *
     * @var bool
     */
    protected $seed = true;
}
```

Gdy właściwość `$seed` jest ustawiona na `true`, test uruchomi klasę `Database\Seeders\DatabaseSeeder` przed każdym testem, który używa cechy `RefreshDatabase`. Jednak możesz określić konkretny seeder, który powinien być wykonany, definiując właściwość `$seeder` w swojej klasie testowej:

```php
use Database\Seeders\OrderStatusSeeder;

/**
 * Uruchom konkretny seeder przed każdym testem.
 *
 * @var string
 */
protected $seeder = OrderStatusSeeder::class;
```

<a name="available-assertions"></a>
## Dostępne Asercje

Laravel dostarcza kilka asercji bazy danych dla Twoich testów funkcjonalnych [Pest](https://pestphp.com) lub [PHPUnit](https://phpunit.de). Omówimy każdą z tych asercji poniżej.

<a name="assert-database-count"></a>
#### assertDatabaseCount

Potwierdź, że tabela w bazie danych zawiera podaną liczbę rekordów:

```php
$this->assertDatabaseCount('users', 5);
```

<a name="assert-database-empty"></a>
#### assertDatabaseEmpty

Potwierdź, że tabela w bazie danych nie zawiera żadnych rekordów:

```php
$this->assertDatabaseEmpty('users');
```

<a name="assert-database-has"></a>
#### assertDatabaseHas

Potwierdź, że tabela w bazie danych zawiera rekordy pasujące do podanych ograniczeń zapytania klucz / wartość:

```php
$this->assertDatabaseHas('users', [
    'email' => 'sally@example.com',
]);
```

<a name="assert-database-missing"></a>
#### assertDatabaseMissing

Potwierdź, że tabela w bazie danych nie zawiera rekordów pasujących do podanych ograniczeń zapytania klucz / wartość:

```php
$this->assertDatabaseMissing('users', [
    'email' => 'sally@example.com',
]);
```

<a name="assert-deleted"></a>
#### assertSoftDeleted

Metoda `assertSoftDeleted` może być użyta do potwierdzenia, że dany model Eloquent został "miękko usunięty":

```php
$this->assertSoftDeleted($user);
```

<a name="assert-not-deleted"></a>
#### assertNotSoftDeleted

Metoda `assertNotSoftDeleted` może być użyta do potwierdzenia, że dany model Eloquent nie został "miękko usunięty":

```php
$this->assertNotSoftDeleted($user);
```

<a name="assert-model-exists"></a>
#### assertModelExists

Potwierdź, że dany model lub kolekcja modeli istnieje w bazie danych:

```php
use App\Models\User;

$user = User::factory()->create();

$this->assertModelExists($user);
```

<a name="assert-model-missing"></a>
#### assertModelMissing

Potwierdź, że dany model lub kolekcja modeli nie istnieje w bazie danych:

```php
use App\Models\User;

$user = User::factory()->create();

$user->delete();

$this->assertModelMissing($user);
```

<a name="expects-database-query-count"></a>
#### expectsDatabaseQueryCount

Metoda `expectsDatabaseQueryCount` może być wywołana na początku testu, aby określić całkowitą liczbę zapytań do bazy danych, które oczekujesz, że zostaną wykonane podczas testu. Jeśli rzeczywista liczba wykonanych zapytań nie będzie dokładnie odpowiadać temu oczekiwaniu, test zakończy się niepowodzeniem:

```php
$this->expectsDatabaseQueryCount(5);

// Test...
```
