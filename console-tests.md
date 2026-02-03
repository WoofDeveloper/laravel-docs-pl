# Testy Konsolowe

- [Wprowadzenie](#introduction)
- [Oczekiwania Sukcesu / Porażki](#success-failure-expectations)
- [Oczekiwania Wejścia / Wyjścia](#input-output-expectations)
- [Zdarzenia Konsolowe](#console-events)

<a name="introduction"></a>
## Wprowadzenie

Oprócz uproszczenia testowania HTTP, Laravel zapewnia proste API do testowania [niestandardowych poleceń konsolowych](/docs/{{version}}/artisan) Twojej aplikacji.

<a name="success-failure-expectations"></a>
## Oczekiwania Sukcesu / Porażki

Na początek przyjrzyjmy się, jak wykonywać asercje dotyczące kodu wyjścia polecenia Artisan. Aby to osiągnąć, użyjemy metody `artisan` do wywołania polecenia Artisan z naszego testu. Następnie użyjemy metody `assertExitCode`, aby potwierdzić, że polecenie zakończyło się z określonym kodem wyjścia:

```php tab=Pest
test('console command', function () {
    $this->artisan('inspire')->assertExitCode(0);
});
```

```php tab=PHPUnit
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('inspire')->assertExitCode(0);
}
```

Możesz użyć metody `assertNotExitCode`, aby potwierdzić, że polecenie nie zakończyło się określonym kodem wyjścia:

```php
$this->artisan('inspire')->assertNotExitCode(1);
```

Oczywiście wszystkie polecenia terminala zazwyczaj kończą się kodem statusu `0`, gdy są pomyślne, oraz kodem niezerowym, gdy nie są pomyślne. Dlatego dla wygody możesz użyć asercji `assertSuccessful` i `assertFailed`, aby potwierdzić, że dane polecenie zakończyło się pomyślnym kodem wyjścia lub nie:

```php
$this->artisan('inspire')->assertSuccessful();

$this->artisan('inspire')->assertFailed();
```

<a name="input-output-expectations"></a>
## Oczekiwania Wejścia / Wyjścia

Laravel pozwala łatwo "mockować" dane wejściowe użytkownika dla poleceń konsolowych za pomocą metody `expectsQuestion`. Ponadto możesz określić kod wyjścia i tekst, który oczekujesz, że zostanie wyświetlony przez polecenie konsolowe, używając metod `assertExitCode` i `expectsOutput`. Na przykład rozważ następujące polecenie konsolowe:

```php
Artisan::command('question', function () {
    $name = $this->ask('What is your name?');

    $language = $this->choice('Which language do you prefer?', [
        'PHP',
        'Ruby',
        'Python',
    ]);

    $this->line('Your name is '.$name.' and you prefer '.$language.'.');
});
```

Możesz przetestować to polecenie za pomocą następującego testu:

```php tab=Pest
test('console command', function () {
    $this->artisan('question')
        ->expectsQuestion('What is your name?', 'Taylor Otwell')
        ->expectsQuestion('Which language do you prefer?', 'PHP')
        ->expectsOutput('Your name is Taylor Otwell and you prefer PHP.')
        ->doesntExpectOutput('Your name is Taylor Otwell and you prefer Ruby.')
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('question')
        ->expectsQuestion('What is your name?', 'Taylor Otwell')
        ->expectsQuestion('Which language do you prefer?', 'PHP')
        ->expectsOutput('Your name is Taylor Otwell and you prefer PHP.')
        ->doesntExpectOutput('Your name is Taylor Otwell and you prefer Ruby.')
        ->assertExitCode(0);
}
```

Jeśli korzystasz z funkcji `search` lub `multisearch` dostarczanych przez [Laravel Prompts](/docs/{{version}}/prompts), możesz użyć asercji `expectsSearch`, aby mockować dane wejściowe użytkownika, wyniki wyszukiwania i wybór:

```php tab=Pest
test('console command', function () {
    $this->artisan('example')
        ->expectsSearch('What is your name?', search: 'Tay', answers: [
            'Taylor Otwell',
            'Taylor Swift',
            'Darian Taylor'
        ], answer: 'Taylor Otwell')
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('example')
        ->expectsSearch('What is your name?', search: 'Tay', answers: [
            'Taylor Otwell',
            'Taylor Swift',
            'Darian Taylor'
        ], answer: 'Taylor Otwell')
        ->assertExitCode(0);
}
```

Możesz również potwierdzić, że polecenie konsolowe nie generuje żadnych danych wyjściowych, używając metody `doesntExpectOutput`:

```php tab=Pest
test('console command', function () {
    $this->artisan('example')
        ->doesntExpectOutput()
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('example')
        ->doesntExpectOutput()
        ->assertExitCode(0);
}
```

Metody `expectsOutputToContain` i `doesntExpectOutputToContain` mogą być użyte do wykonania asercji dotyczących fragmentu danych wyjściowych:

```php tab=Pest
test('console command', function () {
    $this->artisan('example')
        ->expectsOutputToContain('Taylor')
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('example')
        ->expectsOutputToContain('Taylor')
        ->assertExitCode(0);
}
```

<a name="confirmation-expectations"></a>
#### Oczekiwania Potwierdzenia

Podczas pisania polecenia, które oczekuje potwierdzenia w formie odpowiedzi "tak" lub "nie", możesz użyć metody `expectsConfirmation`:

```php
$this->artisan('module:import')
    ->expectsConfirmation('Do you really wish to run this command?', 'no')
    ->assertExitCode(1);
```

<a name="table-expectations"></a>
#### Oczekiwania Tabeli

Jeśli Twoje polecenie wyświetla tabelę informacji za pomocą metody `table` Artisan, może być uciążliwe pisanie oczekiwań wyjściowych dla całej tabeli. Zamiast tego możesz użyć metody `expectsTable`. Ta metoda przyjmuje nagłówki tabeli jako pierwszy argument, a dane tabeli jako drugi argument:

```php
$this->artisan('users:all')
    ->expectsTable([
        'ID',
        'Email',
    ], [
        [1, 'taylor@example.com'],
        [2, 'abigail@example.com'],
    ]);
```

<a name="console-events"></a>
## Zdarzenia Konsolowe

Domyślnie zdarzenia `Illuminate\Console\Events\CommandStarting` i `Illuminate\Console\Events\CommandFinished` nie są wysyłane podczas uruchamiania testów Twojej aplikacji. Możesz jednak włączyć te zdarzenia dla danej klasy testowej, dodając trait `Illuminate\Foundation\Testing\WithConsoleEvents` do klasy:

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\WithConsoleEvents;

pest()->use(WithConsoleEvents::class);

// ...
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\WithConsoleEvents;
use Tests\TestCase;

class ConsoleEventTest extends TestCase
{
    use WithConsoleEvents;

    // ...
}
```
