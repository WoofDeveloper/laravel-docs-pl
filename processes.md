# Procesy

- [Wprowadzenie](#introduction)
- [Wywoływanie procesów](#invoking-processes)
    - [Opcje procesu](#process-options)
    - [Wyjście procesu](#process-output)
    - [Potoki](#process-pipelines)
- [Procesy asynchroniczne](#asynchronous-processes)
    - [Identyfikatory procesów i sygnały](#process-ids-and-signals)
    - [Wyjście procesów asynchronicznych](#asynchronous-process-output)
    - [Limity czasu procesów asynchronicznych](#asynchronous-process-timeouts)
- [Procesy współbieżne](#concurrent-processes)
    - [Nazywanie procesów w puli](#naming-pool-processes)
    - [Identyfikatory i sygnały procesów w puli](#pool-process-ids-and-signals)
- [Testowanie](#testing)
    - [Mockowanie procesów](#faking-processes)
    - [Mockowanie konkretnych procesów](#faking-specific-processes)
    - [Mockowanie sekwencji procesów](#faking-process-sequences)
    - [Mockowanie cykli życia procesów asynchronicznych](#faking-asynchronous-process-lifecycles)
    - [Dostępne asercje](#available-assertions)
    - [Zapobieganie niechcianym procesom](#preventing-stray-processes)

<a name="introduction"></a>
## Wprowadzenie

Laravel oferuje ekspresywne, minimalistyczne API wokół [komponentu Symfony Process](https://symfony.com/doc/current/components/process.html), pozwalając wygodnie wywoływać zewnętrzne procesy z aplikacji Laravel. Funkcje procesów Laravel skupiają się na najbardziej typowych przypadkach użycia i doskonałym doświadczeniu programisty.

<a name="invoking-processes"></a>
## Wywoływanie procesów

Aby wywołać proces, możesz użyć metod `run` i `start` oferowanych przez fasadę `Process`. Metoda `run` wywoła proces i poczeka na zakończenie jego wykonania, natomiast metoda `start` jest używana do asynchronicznego wykonywania procesów. Zbadamy oba podejścia w tej dokumentacji. Najpierw przyjrzyjmy się, jak wywołać podstawowy, synchroniczny proces i sprawdzić jego wynik:

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

return $result->output();
```

Oczywiście, instancja `Illuminate\Contracts\Process\ProcessResult` zwrócona przez metodę `run` oferuje wiele pomocnych metod, które mogą być użyte do sprawdzenia wyniku procesu:

```php
$result = Process::run('ls -la');

$result->command();
$result->successful();
$result->failed();
$result->output();
$result->errorOutput();
$result->exitCode();
```

<a name="throwing-exceptions"></a>
#### Rzucanie wyjątków

Jeśli masz wynik procesu i chciałbyś rzucić instancję `Illuminate\Process\Exceptions\ProcessFailedException`, gdy kod wyjścia jest większy niż zero (co oznacza niepowodzenie), możesz użyć metod `throw` i `throwIf`. Jeśli proces się nie zakończył niepowodzeniem, zostanie zwrócona instancja `ProcessResult`:

```php
$result = Process::run('ls -la')->throw();

$result = Process::run('ls -la')->throwIf($condition);
```

<a name="process-options"></a>
### Opcje procesu

Oczywiście możesz potrzebować dostosować zachowanie procesu przed jego wywołaniem. Na szczęście Laravel pozwala dostosować różne funkcje procesu, takie jak katalog roboczy, limit czasu i zmienne środowiskowe.

<a name="working-directory-path"></a>
#### Ścieżka katalogu roboczego

Możesz użyć metody `path`, aby określić katalog roboczy procesu. Jeśli ta metoda nie zostanie wywołana, proces odziedziczy katalog roboczy obecnie wykonywanego skryptu PHP:

```php
$result = Process::path(__DIR__)->run('ls -la');
```

<a name="input"></a>
#### Wejście

Możesz dostarczyć wejście poprzez "standardowe wejście" procesu używając metody `input`:

```php
$result = Process::input('Hello World')->run('cat');
```

<a name="timeouts"></a>
#### Limity czasu

Domyślnie procesy rzucą instancję `Illuminate\Process\Exceptions\ProcessTimedOutException` po wykonywaniu przez więcej niż 60 sekund. Możesz jednak dostosować to zachowanie za pomocą metody `timeout`:

```php
$result = Process::timeout(120)->run('bash import.sh');
```

Lub, jeśli chcesz całkowicie wyłączyć limit czasu procesu, możesz wywołać metodę `forever`:

```php
$result = Process::forever()->run('bash import.sh');
```

Metoda `idleTimeout` może być użyta do określenia maksymalnej liczby sekund, przez którą proces może działać bez zwracania żadnego wyjścia:

```php
$result = Process::timeout(60)->idleTimeout(30)->run('bash import.sh');
```

<a name="environment-variables"></a>
#### Zmienne środowiskowe

Zmienne środowiskowe mogą być przekazane do procesu za pomocą metody `env`. Wywołany proces odziedziczy również wszystkie zmienne środowiskowe zdefiniowane przez Twój system:

```php
$result = Process::forever()
    ->env(['IMPORT_PATH' => __DIR__])
    ->run('bash import.sh');
```

Jeśli chcesz usunąć odziedziczoną zmienną środowiskową z wywoływanego procesu, możesz podać tę zmienną środowiskową z wartością `false`:

```php
$result = Process::forever()
    ->env(['LOAD_PATH' => false])
    ->run('bash import.sh');
```

<a name="tty-mode"></a>
#### Tryb TTY

Metoda `tty` może być użyta do włączenia trybu TTY dla Twojego procesu. Tryb TTY łączy wejście i wyjście procesu z wejściem i wyjściem Twojego programu, pozwalając Twojemu procesowi otworzyć edytor taki jak Vim lub Nano jako proces:

```php
Process::forever()->tty()->run('vim');
```

> [!WARNING]
> Tryb TTY nie jest wspierany w systemie Windows.

<a name="process-output"></a>
### Wyjście procesu

Jak wcześniej wspomniano, wyjście procesu może być dostępne za pomocą metod `output` (stdout) i `errorOutput` (stderr) na wyniku procesu:

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

echo $result->output();
echo $result->errorOutput();
```

Jednak wyjście może być również zbierane w czasie rzeczywistym poprzez przekazanie domknięcia jako drugiego argumentu do metody `run`. Domknięcie otrzyma dwa argumenty: "typ" wyjścia (`stdout` lub `stderr`) oraz sam ciąg wyjścia:

```php
$result = Process::run('ls -la', function (string $type, string $output) {
    echo $output;
});
```

Laravel oferuje również metody `seeInOutput` i `seeInErrorOutput`, które zapewniają wygodny sposób określenia, czy dany ciąg był zawarty w wyjściu procesu:

```php
if (Process::run('ls -la')->seeInOutput('laravel')) {
    // ...
}
```

<a name="disabling-process-output"></a>
#### Wyłączanie wyjścia procesu

Jeśli Twój proces zapisuje znaczną ilość wyjścia, którym nie jesteś zainteresowany, możesz zaoszczędzić pamięć, całkowicie wyłączając pobieranie wyjścia. Aby to osiągnąć, wywołaj metodę `quietly` podczas budowania procesu:

```php
use Illuminate\Support\Facades\Process;

$result = Process::quietly()->run('bash import.sh');
```

<a name="process-pipelines"></a>
### Potoki

Czasami możesz chcieć uczynić wyjście jednego procesu wejściem innego procesu. Jest to często określane jako "przekazywanie potokowe" wyjścia procesu do innego. Metoda `pipe` dostarczona przez fasadę `Process` ułatwia to osiągnięcie. Metoda `pipe` wykona procesy potokowe synchronicznie i zwróci wynik procesu dla ostatniego procesu w potoku:

```php
use Illuminate\Process\Pipe;
use Illuminate\Support\Facades\Process;

$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
});

if ($result->successful()) {
    // ...
}
```

Jeśli nie musisz dostosowywać poszczególnych procesów składających się na potok, możesz po prostu przekazać tablicę ciągów poleceń do metody `pipe`:

```php
$result = Process::pipe([
    'cat example.txt',
    'grep -i "laravel"',
]);
```

Wyjście procesu może być zbierane w czasie rzeczywistym poprzez przekazanie domknięcia jako drugiego argumentu do metody `pipe`. Domknięcie otrzyma dwa argumenty: "typ" wyjścia (`stdout` lub `stderr`) oraz sam ciąg wyjścia:

```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
}, function (string $type, string $output) {
    echo $output;
});
```

Laravel pozwala również przypisać klucze łańcuchowe do każdego procesu w potoku za pomocą metody `as`. Ten klucz zostanie również przekazany do domknięcia wyjścia dostarczonego do metody `pipe`, pozwalając Ci określić, do którego procesu należy wyjście:

```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->as('first')->command('cat example.txt');
    $pipe->as('second')->command('grep -i "laravel"');
}, function (string $type, string $output, string $key) {
    // ...
});
```

<a name="asynchronous-processes"></a>
## Procesy asynchroniczne

Podczas gdy metoda `run` wywołuje procesy synchronicznie, metoda `start` może być użyta do asynchronicznego wywołania procesu. Pozwala to Twojej aplikacji kontynuować wykonywanie innych zadań, podczas gdy proces działa w tle. Po wywołaniu procesu możesz użyć metody `running`, aby określić, czy proces nadal działa:

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    // ...
}

$result = $process->wait();
```

Jak mogłeś zauważyć, możesz wywołać metodę `wait`, aby poczekać, aż proces zakończy wykonywanie i pobrać instancję `ProcessResult`:

```php
$process = Process::timeout(120)->start('bash import.sh');

// ...

$result = $process->wait();
```

<a name="process-ids-and-signals"></a>
### Identyfikatory procesów i sygnały

Metoda `id` może być użyta do pobrania identyfikatora procesu przypisanego przez system operacyjny dla działającego procesu:

```php
$process = Process::start('bash import.sh');

return $process->id();
```

Możesz użyć metody `signal`, aby wysłać "sygnał" do działającego procesu. Lista predefiniowanych stałych sygnałów znajduje się w [dokumentacji PHP](https://www.php.net/manual/en/pcntl.constants.php):

```php
$process->signal(SIGUSR2);
```

<a name="asynchronous-process-output"></a>
### Wyjście procesów asynchronicznych

Podczas gdy proces asynchroniczny działa, możesz uzyskać dostęp do całego jego bieżącego wyjścia za pomocą metod `output` i `errorOutput`; możesz jednak użyć metod `latestOutput` i `latestErrorOutput`, aby uzyskać dostęp do wyjścia z procesu, które nastąpiło od czasu ostatniego pobrania wyjścia:

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    echo $process->latestOutput();
    echo $process->latestErrorOutput();

    sleep(1);
}
```

Podobnie jak w przypadku metody `run`, wyjście może być również zbierane w czasie rzeczywistym z procesów asynchronicznych poprzez przekazanie domknięcia jako drugiego argumentu do metody `start`. Domknięcie otrzyma dwa argumenty: "typ" wyjścia (`stdout` lub `stderr`) oraz sam ciąg wyjścia:

```php
$process = Process::start('bash import.sh', function (string $type, string $output) {
    echo $output;
});

$result = $process->wait();
```

Zamiast czekać, aż proces się zakończy, możesz użyć metody `waitUntil`, aby przestać czekać na podstawie wyjścia procesu. Laravel przestanie czekać na zakończenie procesu, gdy domknięcie podane do metody `waitUntil` zwróci `true`:

```php
$process = Process::start('bash import.sh');

$process->waitUntil(function (string $type, string $output) {
    return $output === 'Ready...';
});
```

<a name="asynchronous-process-timeouts"></a>
### Limity czasu procesów asynchronicznych

Podczas gdy proces asynchroniczny działa, możesz sprawdzić, czy proces nie przekroczył limitu czasu za pomocą metody `ensureNotTimedOut`. Ta metoda rzuci [wyjątek limitu czasu](#timeouts), jeśli proces przekroczył limit czasu:

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    $process->ensureNotTimedOut();

    // ...

    sleep(1);
}
```

<a name="concurrent-processes"></a>
## Procesy współbieżne

Laravel również ułatwia zarządzanie pulą współbieżnych, asynchronicznych procesów, pozwalając łatwo wykonywać wiele zadań równocześnie. Aby rozpocząć, wywołaj metodę `pool`, która przyjmuje domknięcie otrzymujące instancję `Illuminate\Process\Pool`.

W tym domknięciu możesz zdefiniować procesy należące do puli. Po uruchomieniu puli procesów za pomocą metody `start`, możesz uzyskać dostęp do [kolekcji](/docs/{{version}}/collections) działających procesów za pomocą metody `running`:

```php
use Illuminate\Process\Pool;
use Illuminate\Support\Facades\Process;

$pool = Process::pool(function (Pool $pool) {
    $pool->path(__DIR__)->command('bash import-1.sh');
    $pool->path(__DIR__)->command('bash import-2.sh');
    $pool->path(__DIR__)->command('bash import-3.sh');
})->start(function (string $type, string $output, int $key) {
    // ...
});

while ($pool->running()->isNotEmpty()) {
    // ...
}

$results = $pool->wait();
```

Jak widać, możesz poczekać na zakończenie wykonywania wszystkich procesów w puli i rozwiązać ich wyniki za pomocą metody `wait`. Metoda `wait` zwraca obiekt dostępny jako tablica, który pozwala uzyskać dostęp do instancji `ProcessResult` każdego procesu w puli przez jego klucz:

```php
$results = $pool->wait();

echo $results[0]->output();
```

Lub, dla wygody, metoda `concurrently` może być użyta do uruchomienia asynchronicznej puli procesów i natychmiastowego oczekiwania na jej wyniki. Może to zapewnić szczególnie ekspresyjną składnię w połączeniu z możliwościami destrukturyzacji tablic PHP:

```php
[$first, $second, $third] = Process::concurrently(function (Pool $pool) {
    $pool->path(__DIR__)->command('ls -la');
    $pool->path(app_path())->command('ls -la');
    $pool->path(storage_path())->command('ls -la');
});

echo $first->output();
```

<a name="naming-pool-processes"></a>
### Nazywanie procesów w puli

Dostęp do wyników puli procesów za pomocą klucza numerycznego nie jest zbyt ekspresyjny; dlatego Laravel pozwala przypisać klucze łańcuchowe do każdego procesu w puli za pomocą metody `as`. Ten klucz zostanie również przekazany do domknięcia dostarczonego do metody `start`, pozwalając Ci określić, do którego procesu należy wyjście:

```php
$pool = Process::pool(function (Pool $pool) {
    $pool->as('first')->command('bash import-1.sh');
    $pool->as('second')->command('bash import-2.sh');
    $pool->as('third')->command('bash import-3.sh');
})->start(function (string $type, string $output, string $key) {
    // ...
});

$results = $pool->wait();

return $results['first']->output();
```

<a name="pool-process-ids-and-signals"></a>
### Identyfikatory i sygnały procesów w puli

Ponieważ metoda `running` puli procesów dostarcza kolekcję wszystkich wywołanych procesów w puli, możesz łatwo uzyskać dostęp do podstawowych identyfikatorów procesów w puli:

```php
$processIds = $pool->running()->each->id();
```

I, dla wygody, możesz wywołać metodę `signal` na puli procesów, aby wysłać sygnał do każdego procesu w puli:

```php
$pool->signal(SIGUSR2);
```

<a name="testing"></a>
## Testowanie

Wiele usług Laravel zapewnia funkcjonalność pomagającą łatwo i wyraziście pisać testy, a usługa procesów Laravel nie jest wyjątkiem. Metoda `fake` fasady `Process` pozwala poinstruować Laravel, aby zwracał zastępcze / sztuczne wyniki, gdy procesy są wywoływane.

<a name="faking-processes"></a>
### Mockowanie procesów

Aby zbadać zdolność Laravel do mockowania procesów, wyobraźmy sobie trasę, która wywołuje proces:

```php
use Illuminate\Support\Facades\Process;
use Illuminate\Support\Facades\Route;

Route::get('/import', function () {
    Process::run('bash import.sh');

    return 'Import complete!';
});
```

Podczas testowania tej trasy możemy poinstruować Laravel, aby zwracał sztuczny, pomyślny wynik procesu dla każdego wywołanego procesu, wywołując metodę `fake` na fasadzie `Process` bez argumentów. Ponadto możemy nawet [potwierdzić](#available-assertions), że dany proces został "uruchomiony":

```php tab=Pest
<?php

use Illuminate\Contracts\Process\ProcessResult;
use Illuminate\Process\PendingProcess;
use Illuminate\Support\Facades\Process;

test('process is invoked', function () {
    Process::fake();

    $response = $this->get('/import');

    // Simple process assertion...
    Process::assertRan('bash import.sh');

    // Or, inspecting the process configuration...
    Process::assertRan(function (PendingProcess $process, ProcessResult $result) {
        return $process->command === 'bash import.sh' &&
               $process->timeout === 60;
    });
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Illuminate\Contracts\Process\ProcessResult;
use Illuminate\Process\PendingProcess;
use Illuminate\Support\Facades\Process;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_process_is_invoked(): void
    {
        Process::fake();

        $response = $this->get('/import');

        // Simple process assertion...
        Process::assertRan('bash import.sh');

        // Or, inspecting the process configuration...
        Process::assertRan(function (PendingProcess $process, ProcessResult $result) {
            return $process->command === 'bash import.sh' &&
                   $process->timeout === 60;
        });
    }
}
```

Jak wspomniano, wywołanie metody `fake` na fasadzie `Process` poinstruuje Laravel, aby zawsze zwracał pomyślny wynik procesu bez wyjścia. Możesz jednak łatwo określić wyjście i kod wyjścia dla mockowanych procesów używając metody `result` fasady `Process`:

```php
Process::fake([
    '*' => Process::result(
        output: 'Test output',
        errorOutput: 'Test error output',
        exitCode: 1,
    ),
]);
```

<a name="faking-specific-processes"></a>
### Mockowanie konkretnych procesów

Jak mogłeś zauważyć w poprzednim przykładzie, fasada `Process` pozwala określić różne sztuczne wyniki dla każdego procesu, przekazując tablicę do metody `fake`.

Klucze tablicy powinny reprezentować wzorce poleceń, które chcesz mockować, i powiązane z nimi wyniki. Znak `*` może być użyty jako znak wieloznaczny. Wszelkie polecenia procesów, które nie zostały zamockowane, zostaną faktycznie wywołane. Możesz użyć metody `result` fasady `Process`, aby skonstruować zastępcze / sztuczne wyniki dla tych poleceń:

```php
Process::fake([
    'cat *' => Process::result(
        output: 'Test "cat" output',
    ),
    'ls *' => Process::result(
        output: 'Test "ls" output',
    ),
]);
```

Jeśli nie musisz dostosowywać kodu wyjścia lub wyjścia błędu mockowanego procesu, możesz uznać za wygodniejsze określenie wyników sztucznego procesu jako prostych ciągów:

```php
Process::fake([
    'cat *' => 'Test "cat" output',
    'ls *' => 'Test "ls" output',
]);
```

<a name="faking-process-sequences"></a>
### Mockowanie sekwencji procesów

Jeśli testowany kod wywołuje wiele procesów z tym samym poleceniem, możesz chcieć przypisać inny sztuczny wynik procesu do każdego wywołania procesu. Możesz to osiągnąć za pomocą metody `sequence` fasady `Process`:

```php
Process::fake([
    'ls *' => Process::sequence()
        ->push(Process::result('First invocation'))
        ->push(Process::result('Second invocation')),
]);
```

<a name="faking-asynchronous-process-lifecycles"></a>
### Mockowanie cykli życia procesów asynchronicznych

Do tej pory głównie omawialiśmy mockowanie procesów wywoływanych synchronicznie za pomocą metody `run`. Jednak jeśli próbujesz przetestować kod, który wchodzi w interakcję z procesami asynchronicznymi wywoływanymi za pomocą `start`, możesz potrzebować bardziej wyrafinowanego podejścia do opisywania swoich sztucznych procesów.

Na przykład wyobraźmy sobie następującą trasę, która wchodzi w interakcję z procesem asynchronicznym:

```php
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Route;

Route::get('/import', function () {
    $process = Process::start('bash import.sh');

    while ($process->running()) {
        Log::info($process->latestOutput());
        Log::info($process->latestErrorOutput());
    }

    return 'Done';
});
```

Aby właściwie zmockować ten proces, musimy być w stanie opisać, ile razy metoda `running` powinna zwrócić `true`. Ponadto możemy chcieć określić wiele linii wyjścia, które powinny być zwrócone w kolejności. Aby to osiągnąć, możemy użyć metody `describe` fasady `Process`:

```php
Process::fake([
    'bash import.sh' => Process::describe()
        ->output('First line of standard output')
        ->errorOutput('First line of error output')
        ->output('Second line of standard output')
        ->exitCode(0)
        ->iterations(3),
]);
```

Zagłębmy się w powyższy przykład. Używając metod `output` i `errorOutput`, możemy określić wiele linii wyjścia, które zostaną zwrócone w kolejności. Metoda `exitCode` może być użyta do określenia końcowego kodu wyjścia sztucznego procesu. Na koniec metoda `iterations` może być użyta do określenia, ile razy metoda `running` powinna zwrócić `true`.

<a name="available-assertions"></a>
### Dostępne asercje

Jak [wcześniej wspomniano](#faking-processes), Laravel zapewnia kilka asercji procesów dla testów funkcjonalnych. Omówimy każdą z tych asercji poniżej.

<a name="assert-process-ran"></a>
#### assertRan

Sprawdź, że dany proces został wywołany:

```php
use Illuminate\Support\Facades\Process;

Process::assertRan('ls -la');
```

Metoda `assertRan` akceptuje również domknięcie, które otrzyma instancję procesu i wynik procesu, pozwalając na sprawdzenie skonfigurowanych opcji procesu. Jeśli to domknięcie zwróci `true`, asercja "przejdzie":

```php
Process::assertRan(fn ($process, $result) =>
    $process->command === 'ls -la' &&
    $process->path === __DIR__ &&
    $process->timeout === 60
);
```

`$process` przekazany do domknięcia `assertRan` jest instancją `Illuminate\Process\PendingProcess`, podczas gdy `$result` jest instancją `Illuminate\Contracts\Process\ProcessResult`.

<a name="assert-process-didnt-run"></a>
#### assertDidntRun

Sprawdź, że dany proces nie został wywołany:

```php
use Illuminate\Support\Facades\Process;

Process::assertDidntRun('ls -la');
```

Podobnie jak metoda `assertRan`, metoda `assertDidntRun` również akceptuje domknięcie, które otrzyma instancję procesu i wynik procesu, pozwalając na sprawdzenie skonfigurowanych opcji procesu. Jeśli to domknięcie zwróci `true`, asercja "nie powiedzie się":

```php
Process::assertDidntRun(fn (PendingProcess $process, ProcessResult $result) =>
    $process->command === 'ls -la'
);
```

<a name="assert-process-ran-times"></a>
#### assertRanTimes

Sprawdź, że dany proces został wywołany określoną liczbę razy:

```php
use Illuminate\Support\Facades\Process;

Process::assertRanTimes('ls -la', times: 3);
```

Metoda `assertRanTimes` również akceptuje domknięcie, które otrzyma instancję `PendingProcess` i `ProcessResult`, pozwalając na sprawdzenie skonfigurowanych opcji procesu. Jeśli to domknięcie zwróci `true` i proces został wywołany określoną liczbę razy, asercja "przejdzie":

```php
Process::assertRanTimes(function (PendingProcess $process, ProcessResult $result) {
    return $process->command === 'ls -la';
}, times: 3);
```

<a name="preventing-stray-processes"></a>
### Zapobieganie niechcianym procesom

Jeśli chcesz się upewnić, że wszystkie wywołane procesy zostały zamockowane w całym pojedynczym teście lub pełnym zestawie testów, możesz wywołać metodę `preventStrayProcesses`. Po wywołaniu tej metody wszelkie procesy, które nie mają odpowiedniego sztucznego wyniku, rzucą wyjątek zamiast uruchomienia rzeczywistego procesu:

```php
use Illuminate\Support\Facades\Process;

Process::preventStrayProcesses();

Process::fake([
    'ls *' => 'Test output...',
]);

// Fake response is returned...
Process::run('ls -la');

// An exception is thrown...
Process::run('bash import.sh');
```
