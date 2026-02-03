# Konsola Artisan

- [Wprowadzenie](#introduction)
    - [Tinker (REPL)](#tinker)
- [Pisanie Komend](#writing-commands)
    - [Generowanie Komend](#generating-commands)
    - [Struktura Komendy](#command-structure)
    - [Komendy Closure](#closure-commands)
    - [Komendy Izolowalce](#isolatable-commands)
- [Definiowanie Oczekiwań Wejściowych](#defining-input-expectations)
    - [Argumenty](#arguments)
    - [Opcje](#options)
    - [Tablice Wejściowe](#input-arrays)
    - [Opisy Wejść](#input-descriptions)
    - [Pytanie o Brakujące Dane Wejściowe](#prompting-for-missing-input)
- [Wejście/Wyjście Komendy](#command-io)
    - [Pobieranie Wejścia](#retrieving-input)
    - [Pytanie o Dane Wejściowe](#prompting-for-input)
    - [Wypisywanie Wyjścia](#writing-output)
- [Rejestrowanie Komend](#registering-commands)
- [Programowe Wykonywanie Komend](#programmatically-executing-commands)
    - [Wywoływanie Komend z Innych Komend](#calling-commands-from-other-commands)
- [Obsługa Sygnałów](#signal-handling)
- [Dostosowywanie Stubów](#stub-customization)
- [Zdarzenia](#events)

<a name="introduction"></a>
## Wprowadzenie

Artisan to interfejs wiersza poleceń dołączony do Laravel. Artisan znajduje się w katalogu głównym aplikacji jako skrypt `artisan` i udostępnia szereg pomocnych komend, które mogą Ci pomóc podczas budowania aplikacji. Aby zobaczyć listę wszystkich dostępnych komend Artisan, możesz użyć komendy `list`:

```shell
php artisan list
```

Każda komenda zawiera również ekran "help", który wyświetla i opisuje dostępne argumenty i opcje komendy. Aby zobaczyć ekran pomocy, poprzedź nazwę komendy słowem `help`:

```shell
php artisan help migrate
```

<a name="laravel-sail"></a>
#### Laravel Sail

Jeśli używasz [Laravel Sail](/docs/{{version}}/sail) jako lokalnego środowiska programistycznego, pamiętaj, aby używać wiersza poleceń `sail` do wywoływania komend Artisan. Sail wykona Twoje komendy Artisan w kontenerach Docker Twojej aplikacji:

```shell
./vendor/bin/sail artisan list
```

<a name="tinker"></a>
### Tinker (REPL)

[Laravel Tinker](https://github.com/laravel/tinker) to potężny REPL dla frameworka Laravel, zasilany przez pakiet [PsySH](https://github.com/bobthecow/psysh).

<a name="installation"></a>
#### Instalacja

Wszystkie aplikacje Laravel zawierają Tinker domyślnie. Jednak możesz zainstalować Tinker za pomocą Composera, jeśli wcześniej usunąłeś go ze swojej aplikacji:

```shell
composer require laravel/tinker
```

> [!NOTE]
> Szukasz hot reloading, wieloliniowej edycji kodu i autouzupełniania podczas interakcji z Twoją aplikacją Laravel? Sprawdź [Tinkerwell](https://tinkerwell.app)!

<a name="usage"></a>
#### Użycie

Tinker pozwala na interakcję z całą aplikacją Laravel w wierszu poleceń, włącznie z modelami Eloquent, zadaniami, zdarzeniami i więcej. Aby wejść do środowiska Tinker, uruchom komendę Artisan `tinker`:

```shell
php artisan tinker
```

Możesz opublikować plik konfiguracyjny Tinker za pomocą komendy `vendor:publish`:

```shell
php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"
```

> [!WARNING]
> Funkcja pomocnicza `dispatch` i metoda `dispatch` klasy `Dispatchable` polegają na garbage collection, aby umieścić zadanie w kolejce. Dlatego podczas używania Tinker powinieneś użyć `Bus::dispatch` lub `Queue::push` do wysyłania zadań.

<a name="command-allow-list"></a>
#### Lista Dozwolonych Komend

Tinker używa listy "dozwolonych" aby określić, które komendy Artisan mogą być uruchamiane w jego powłoce. Domyślnie możesz uruchamiać komendy `clear-compiled`, `down`, `env`, `inspire`, `migrate`, `migrate:install`, `up` i `optimize`. Jeśli chcesz zezwolić na więcej komend, możesz dodać je do tablicy `commands` w pliku konfiguracyjnym `tinker.php`:

```php
'commands' => [
    // App\Console\Commands\ExampleCommand::class,
],
```

<a name="classes-that-should-not-be-aliased"></a>
#### Klasy, które nie powinny być aliasowane

Zwykle Tinker automatycznie tworzy aliasy klas podczas interakcji z nimi w Tinker. Jednak możesz nigdy nie chcieć aliasować niektórych klas. Możesz to osiągnąć, wymieniając klasy w tablicy `dont_alias` w pliku konfiguracyjnym `tinker.php`:

```php
'dont_alias' => [
    App\Models\User::class,
],
```

<a name="writing-commands"></a>
## Pisanie Komend

Oprócz komend dostarczanych z Artisan, możesz tworzyć własne niestandardowe komendy. Komendy są zazwyczaj przechowywane w katalogu `app/Console/Commands`; jednak masz swobodę wyboru własnej lokalizacji przechowywania, o ile poinstruujesz Laravel, aby [skanował inne katalogi w poszukiwaniu komend Artisan](#registering-commands).

<a name="generating-commands"></a>
### Generowanie Komend

Aby stworzyć nową komendę, możesz użyć komendy Artisan `make:command`. Ta komenda utworzy nową klasę komendy w katalogu `app/Console/Commands`. Nie martw się, jeśli ten katalog nie istnieje w Twojej aplikacji - zostanie utworzony przy pierwszym uruchomieniu komendy Artisan `make:command`:

```shell
php artisan make:command SendEmails
```

<a name="command-structure"></a>
### Struktura Komendy

Po wygenerowaniu komendy powinieneś zdefiniować odpowiednie wartości dla właściwości `signature` i `description` klasy. Te właściwości będą używane podczas wyświetlania komendy na ekranie `list`. Właściwość `signature` pozwala również definiować [oczekiwania wejściowe komendy](#defining-input-expectations). Metoda `handle` zostanie wywołana podczas wykonywania komendy. Możesz umieścić logikę komendy w tej metodzie.

Spojrzmy na przykładową komendę. Zauważ, że możemy żądać wszelkich zależności, których potrzebujemy przez metodę `handle` komendy. [Kontener usług](/docs/{{version}}/container) Laravel automatycznie wstrzyknie wszystkie zależności, które są oznaczone typem w sygnaturze tej metody:

```php
<?php

namespace App\Console\Commands;

use App\Models\User;
use App\Support\DripEmailer;
use Illuminate\Console\Command;

class SendEmails extends Command
{
    /**
     * Nazwa i sygnatura komendy konsolowej.
     *
     * @var string
     */
    protected $signature = 'mail:send {user}';

    /**
     * Opis komendy konsolowej.
     *
     * @var string
     */
    protected $description = 'Wyślij email marketingowy do użytkownika';

    /**
     * Wykonaj komendę konsolową.
     */
    public function handle(DripEmailer $drip): void
    {
        $drip->send(User::find($this->argument('user')));
    }
}
```

> [!NOTE]
> Dla większego ponownego użycia kodu, dobrym zwyczajem jest utrzymywanie komend konsolowych lekkimi i pozwolenie im na delegowanie zadań do usług aplikacji. W powyższym przykładzie zauważ, że wstrzykujemy klasę usługi, aby wykonać "ciężką pracę" wysyłania e-maili.

<a name="exit-codes"></a>
#### Kody Wyjścia

Jeśli nic nie jest zwracane z metody `handle` i komenda wykona się pomyślnie, komenda zakończy się kodem wyjścia `0`, wskazującym sukces. Jednak metoda `handle` może opcjonalnie zwrócić liczbę całkowitą, aby ręcznie określić kod wyjścia komendy:

```php
$this->error('Coś poszło nie tak.');

return 1;
```

Jeśli chcesz "nie udać" komendy z jakiejkolwiek metody wewnątrz komendy, możesz użyć metody `fail`. Metoda `fail` natychmiast zakończy wykonanie komendy i zwróci kod wyjścia `1`:

```php
$this->fail('Coś poszło nie tak.');
```

<a name="closure-commands"></a>
### Komendy Closure

Komendy oparte na closure stanowią alternatywę dla definiowania komend konsolowych jako klas. W ten sam sposób, w jaki closure tras są alternatywą dla kontrolerów, myśl o closure komend jako alternatywie dla klas komend.

Chocიაż plik `routes/console.php` nie definiuje tras HTTP, definiuje punkty wejścia (trasy) oparte na konsoli do Twojej aplikacji. W tym pliku możesz zdefiniować wszystkie swoje komendy konsolowe oparte na closure za pomocą metody `Artisan::command`. Metoda `command` przyjmuje dwa argumenty: [sygnaturę komendy](#defining-input-expectations) oraz closure, które otrzymuje argumenty i opcje komendy:

```php
Artisan::command('mail:send {user}', function (string $user) {
    $this->info("Wysyłanie emaila do: {$user}!");
});
```

Closure jest powiązane z podstawową instancją komendy, więc masz pełny dostęp do wszystkich metod pomocniczych, do których zwykle miałbyś dostęp w pełnej klasie komendy.

<a name="type-hinting-dependencies"></a>
#### Oznaczanie Zależności Typem

Oprócz otrzymywania argumentów i opcji komendy, closure komend mogą również oznaczać typem dodatkowe zależności, które chcesz rozstrzygnąć z [kontenera usług](/docs/{{version}}/container):

```php
use App\Models\User;
use App\Support\DripEmailer;
use Illuminate\Support\Facades\Artisan;

Artisan::command('mail:send {user}', function (DripEmailer $drip, string $user) {
    $drip->send(User::find($user));
});
```

<a name="closure-command-descriptions"></a>
#### Opisy Komend Closure

Podczas definiowania komendy opartej na closure, możesz użyć metody `purpose`, aby dodać opis do komendy. Ten opis będzie wyświetlany podczas uruchamiania komend `php artisan list` lub `php artisan help`:

```php
Artisan::command('mail:send {user}', function (string $user) {
    // ...
})->purpose('Wyślij email marketingowy do użytkownika');
```

<a name="isolatable-commands"></a>
### Komendy Izolowalne

> [!WARNING]
> Aby użyć tej funkcji, Twoja aplikacja musi używać sterownika cache `memcached`, `redis`, `dynamodb`, `database`, `file` lub `array` jako domyślnego sterownika cache aplikacji. Ponadto wszystkie serwery muszą komunikować się z tym samym centralnym serwerem cache.

Czasami możesz chcieć upewnić się, że tylko jedna instancja komendy może być uruchomiona w danym czasie. Aby to osiągnąć, możesz zaimplementować interfejs `Illuminate\Contracts\Console\Isolatable` w swojej klasie komendy:

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Contracts\Console\Isolatable;

class SendEmails extends Command implements Isolatable
{
    // ...
}
```

Gdy oznaczysz komendę jako `Isolatable`, Laravel automatycznie udostępnia opcję `--isolated` dla komendy bez konieczności jawnego definiowania jej w opcjach komendy. Gdy komenda jest wywoływana z tą opcją, Laravel upewni się, że żadne inne instancje tej komendy już nie działają. Laravel osiąga to, próbując uzyskać blokadę atomową przy użyciu domyślnego sterownika cache aplikacji. Jeśli inne instancje komendy działają, komenda nie zostanie wykonana; jednak komenda nadal zakończy się z kodem statusu sukcesu:

```shell
php artisan mail:send 1 --isolated
```

Jeśli chcesz określić kod statusu wyjścia, który komenda powinna zwrócić, jeśli nie może zostać wykonana, możesz podać żądany kod statusu przez opcję `isolated`:

```shell
php artisan mail:send 1 --isolated=12
```

<a name="lock-id"></a>
#### ID Blokady

Domyślnie Laravel użyje nazwy komendy do wygenerowania klucza łańcucha, który jest używany do uzyskania blokady atomowej w cache aplikacji. Jednak możesz dostosować ten klucz, definiując metodę `isolatableId` w swojej klasie komendy Artisan, pozwalając zintegrować argumenty lub opcje komendy z kluczem:

```php
/**
 * Pobierz ID izolacji dla komendy.
 */
public function isolatableId(): string
{
    return $this->argument('user');
}
```

<a name="lock-expiration-time"></a>
#### Czas Wygaśnięcia Blokady

Domyślnie blokady izolacji wygaśają po zakończeniu komendy. Lub, jeśli komenda zostanie przerwana i nie może się zakończyć, blokada wygasa po godzinie. Jednak możesz dostosować czas wygaśnięcia blokady, definiując metodę `isolationLockExpiresAt` w swojej komendzie:

```php
use DateTimeInterface;
use DateInterval;

/**
 * Określ, kiedy blokada izolacji wygasa dla komendy.
 */
public function isolationLockExpiresAt(): DateTimeInterface|DateInterval
{
    return now()->plus(minutes: 5);
}
```

<a name="defining-input-expectations"></a>
## Definiowanie Oczekiwań Wejściowych

Podczas pisania komend konsolowych, powszechne jest zbieranie danych wejściowych od użytkownika poprzez argumenty lub opcje. Laravel sprawia, że bardzo wygodnie można określić dane wejściowe, których oczekujesz od użytkownika, używając właściwości `signature` w swoich komendach. Właściwość `signature` pozwala określić nazwę, argumenty i opcje komendy w pojedynczej, ekspresyjnej składni podobnej do tras.

<a name="arguments"></a>
### Argumenty

Wszystkie argumenty i opcje dostarczone przez użytkownika są owinione w nawiasy klamrowe. W poniższym przykładzie komenda definiuje jeden wymagany argument: `user`:

```php
/**
 * Nazwa i sygnatura komendy konsolowej.
 *
 * @var string
 */
protected $signature = 'mail:send {user}';
```

Możesz również uczynić argumenty opcjonalnymi lub zdefiniować domyślne wartości dla argumentów:

```php
// Opcjonalny argument...
'mail:send {user?}'

// Opcjonalny argument z domyślną wartością...
'mail:send {user=foo}'
```

<a name="options"></a>
### Opcje

Opcje, podobnie jak argumenty, są inną formą danych wejściowych użytkownika. Opcje są poprzedzone dwoma myślnikami (`--`), gdy są podawane przez wiersz poleceń. Istnieją dwa typy opcji: te, które otrzymują wartość i te, które nie. Opcje, które nie otrzymują wartości, służą jako "przełącznik" logiczny. Spójrzmy na przykład tego typu opcji:

```php
/**
 * Nazwa i sygnatura komendy konsolowej.
 *
 * @var string
 */
protected $signature = 'mail:send {user} {--queue}';
```

W tym przykładzie przełącznik `--queue` może zostać określony podczas wywoływania komendy Artisan. Jeśli przełącznik `--queue` zostanie przekazany, wartość opcji będzie `true`. W przeciwnym razie wartość będzie `false`:

```shell
php artisan mail:send 1 --queue
```

<a name="options-with-values"></a>
#### Opcje Z Wartościami

Następnie spójrzmy na opcję, która oczekuje wartości. Jeśli użytkownik musi określić wartość dla opcji, powinieneś dodać znak `=` do nazwy opcji:

```php
/**
 * Nazwa i sygnatura komendy konsolowej.
 *
 * @var string
 */
protected $signature = 'mail:send {user} {--queue=}';
```

W tym przykładzie użytkownik może przekazać wartość dla opcji w ten sposób. Jeśli opcja nie zostanie określona podczas wywoływania komendy, jej wartością będzie `null`:

```shell
php artisan mail:send 1 --queue=default
```

Możesz przypisać domyślne wartości do opcji, określając domyślną wartość po nazwie opcji. Jeśli użytkownik nie przekazuje wartości opcji, zostanie użyta wartość domyślna:

```php
'mail:send {user} {--queue=default}'
```

<a name="option-shortcuts"></a>
#### Skróty Opcji

Aby przypisać skrót podczas definiowania opcji, możesz określić go przed nazwą opcji i użyć znaku `|` jako separatora, aby oddzielić skrót od pełnej nazwy opcji:

```php
'mail:send {user} {--Q|queue=}'
```

Podczas wywoływania komendy w terminalu, skróty opcji powinny być poprzedzone pojedynczym myślnikiem i nie powinien być zawarty znak `=` podczas określania wartości dla opcji:

```shell
php artisan mail:send 1 -Qdefault
```

<a name="input-arrays"></a>
### Tablice Wejściowe

Jeśli chcesz zdefiniować argumenty lub opcje, aby oczekiwały wielu wartości wejściowych, możesz użyć znaku `*`. Najpierw spójrzmy na przykład, który określa taki argument:

```php
'mail:send {user*}'
```

Podczas uruchamiania tej komendy argumenty `user` mogą zostać przekazane w kolejności do wiersza poleceń. Na przykład poniższa komenda ustawi wartość `user` na tablicę z `1` i `2` jako jej wartościami:

```shell
php artisan mail:send 1 2
```

Ten znak `*` może być połączony z opcjonalną definicją argumentu, aby pozwolić na zero lub więcej instancji argumentu:

```php
'mail:send {user?*}'
```

<a name="option-arrays"></a>
#### Tablice Opcji

Podczas definiowania opcji, która oczekuje wielu wartości wejściowych, każda wartość opcji przekazana do komendy powinna być poprzedzona nazwą opcji:

```php
'mail:send {--id=*}'
```

Taka komenda może zostać wywołana przez przekazanie wielu argumentów `--id`:

```shell
php artisan mail:send --id=1 --id=2
```

<a name="input-descriptions"></a>
### Opisy Wejść

Możesz przypisać opisy do argumentów i opcji wejściowych, oddzielając nazwę argumentu od opisu dwukropkiem. Jeśli potrzebujesz trochę więcej miejsca na zdefiniowanie swojej komendy, możesz swobodnie rozciągnąć definicję na wiele linii:

```php
/**
 * Nazwa i sygnatura komendy konsolowej.
 *
 * @var string
 */
protected $signature = 'mail:send
                        {user : ID użytkownika}
                        {--queue : Czy zadanie powinno zostać umieszczone w kolejce}';
```

<a name="prompting-for-missing-input"></a>
### Pytanie o Brakujące Dane Wejściowe

Jeśli Twoja komenda zawiera wymagane argumenty, użytkownik otrzyma komunikat o błędzie, gdy nie zostaną one dostarczone. Alternatywnie możesz skonfigurować swoją komendę tak, aby automatycznie pytała użytkownika, gdy brakuje wymaganych argumentów, implementując interfejs `PromptsForMissingInput`:

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Contracts\Console\PromptsForMissingInput;

class SendEmails extends Command implements PromptsForMissingInput
{
    /**
     * Nazwa i sygnatura komendy konsolowej.
     *
     * @var string
     */
    protected $signature = 'mail:send {user}';

    // ...
}
```

Jeśli Laravel musi zebrać wymagany argument od użytkownika, automatycznie zapyta użytkownika o argument, inteligentnie formując pytanie przy użyciu nazwy argumentu lub opisu. Jeśli chcesz dostosować pytanie używane do zebrania wymaganego argumentu, możesz zaimplementować metodę `promptForMissingArgumentsUsing`, zwracającą tablicę pytań indeksowanych nazwami argumentów:

```php
/**
 * Zapytaj o brakujące argumenty wejściowe używając zwracanych pytań.
 *
 * @return array<string, string>
 */
protected function promptForMissingArgumentsUsing(): array
{
    return [
        'user' => 'Który ID użytkownika powinien otrzymać email?',
    ];
}
```

Możesz również dostarczyć tekst zastępczy, używając krotki zawierającej pytanie i zastępnik:

```php
return [
    'user' => ['Który ID użytkownika powinien otrzymać email?', 'Np. 123'],
];
```

Jeśli chcesz mieć pełną kontrolę nad promptem, możesz dostarczyć closure, które powinno zapytać użytkownika i zwrócić jego odpowiedź:

```php
use App\Models\User;
use function Laravel\Prompts\search;

// ...

return [
    'user' => fn () => search(
        label: 'Wyszukaj użytkownika:',
        placeholder: 'Np. Taylor Otwell',
        options: fn ($value) => strlen($value) > 0
            ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
            : []
    ),
];
```

> [!NOTE]
Wyczerpująca dokumentacja [Laravel Prompts](/docs/{{version}}/prompts) zawiera dodatkowe informacje o dostępnych promptach i ich użyciu.

Jeśli chcesz poprosić użytkownika o wybór lub wprowadzenie [opcji](#options), możesz uwzględnić prompty w metodzie `handle` swojej komendy. Jednak jeśli chcesz tylko pytać użytkownika, gdy został również automatycznie poproszony o brakujące argumenty, możesz zaimplementować metodę `afterPromptingForMissingArguments`:

```php
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use function Laravel\Prompts\confirm;

// ...

/**
 * Wykonaj akcje po tym, jak użytkownik został poproszony o brakujące argumenty.
 */
protected function afterPromptingForMissingArguments(InputInterface $input, OutputInterface $output): void
{
    $input->setOption('queue', confirm(
        label: 'Czy chcesz umieścić email w kolejce?',
        default: $this->option('queue')
    ));
}
```

<a name="command-io"></a>
## Wejście/Wyjście Komendy

<a name="retrieving-input"></a>
### Pobieranie Wejścia

Podczas wykonywania komendy prawdopodobnie będziesz musiał uzyskać dostęp do wartości argumentów i opcji akceptowanych przez Twoją komendę. Aby to zrobić, możesz użyć metod `argument` i `option`. Jeśli argument lub opcja nie istnieje, zwracane jest `null`:

```php
/**
 * Wykonaj komendę konsolową.
 */
public function handle(): void
{
    $userId = $this->argument('user');
}
```

Jeśli musisz pobrać wszystkie argumenty jako tablicę (`array`), wywołaj metodę `arguments`:

```php
$arguments = $this->arguments();
```

Opcje mogą być pobierane równie latwo jak argumenty za pomocą metody `option`. Aby pobrać wszystkie opcje jako tablicę, wywołaj metodę `options`:

```php
// Pobierz konkretną opcję...
$queueName = $this->option('queue');

// Pobierz wszystkie opcje jako tablicę...
$options = $this->options();
```

<a name="prompting-for-input"></a>
### Pytanie o Dane Wejściowe

> [!NOTE]
> [Laravel Prompts](/docs/{{version}}/prompts) to pakiet PHP służący do dodawania pięknych i przyjaznych użytkownikowi formularzy do aplikacji wiersza poleceń, z funkcjami podobnymi do przeglądarki, w tym tekstem zastępczym i walidacją.

Oprócz wyświetlania wyjścia, możesz również poprosić użytkownika o podanie danych wejściowych podczas wykonywania komendy. Metoda `ask` zapyta użytkownika podanym pytaniem, zaakceptuje jego dane wejściowe, a następnie zwróci dane wejściowe użytkownika z powrotem do komendy:

```php
/**
 * Wykonaj komendę konsolową.
 */
public function handle(): void
{
    $name = $this->ask('Jak się nazywasz?');

    // ...
}
```

Metoda `ask` akceptuje również opcjonalny drugi argument, który określa domyślną wartość, która powinna zostać zwrócona, jeśli nie podano danych wejściowych użytkownika:

```php
$name = $this->ask('Jak się nazywasz?', 'Taylor');
```

Metoda `secret` jest podobna do `ask`, ale dane wejściowe użytkownika nie będą widoczne podczas wpisywania w konsoli. Ta metoda jest przydatna podczas pytania o wrażliwe informacje, takie jak hasła:

```php
$password = $this->secret('Jakie jest hasło?');
```

<a name="asking-for-confirmation"></a>
#### Pytanie o Potwierdzenie

Jeśli musisz zapytać użytkownika o proste potwierdzenie "tak lub nie", możesz użyć metody `confirm`. Domyślnie ta metoda zwróci `false`. Jednak jeśli użytkownik wprowadzi `y` lub `yes` w odpowiedzi na prompt, metoda zwróci `true`.

```php
if ($this->confirm('Czy chcesz kontynuować?')) {
    // ...
}
```

Jeśli to konieczne, możesz określić, że prompt potwierdzenia powinien domyślnie zwracać `true`, przekazując `true` jako drugi argument do metody `confirm`:

```php
if ($this->confirm('Czy chcesz kontynuować?', true)) {
    // ...
}
```

<a name="auto-completion"></a>
#### Autouzupełnianie

Metoda `anticipate` może być używana do zapewnienia autouzupełniania dla możliwych wyborów. Użytkownik nadal może podać jakąkolwiek odpowiedź, niezależnie od podpowiedzi autouzupełniania:

```php
$name = $this->anticipate('Jak się nazywasz?', ['Taylor', 'Dayle']);
```

Alternatywnie możesz przekazać closure jako drugi argument do metody `anticipate`. Closure będzie wywoływane za każdym razem, gdy użytkownik wpisze znak. Closure powinno przyjąć parametr łańcucha zawierający dotychczasowe dane wejściowe użytkownika i zwrócić tablicę opcji do autouzupełniania:

```php
use App\Models\Address;

$name = $this->anticipate('Jaki jest twój adres?', function (string $input) {
    return Address::whereLike('name', "{$input}%")
        ->limit(5)
        ->pluck('name')
        ->all();
});
```

<a name="multiple-choice-questions"></a>
#### Pytania Wielokrotnego Wyboru

Jeśli musisz dać użytkownikowi predefiniowany zestaw wyborów podczas zadawania pytania, możesz użyć metody `choice`. Możesz ustawić indeks tablicy domyślnej wartości, która ma zostać zwrócona, jeśli nie wybrano żadnej opcji, przekazując indeks jako trzeci argument do metody:

```php
$name = $this->choice(
    'Jak się nazywasz?',
    ['Taylor', 'Dayle'],
    $defaultIndex
);
```

Ponadto metoda `choice` akceptuje opcjonalne czwarty i piąty argumenty do określenia maksymalnej liczby prób wybrania poprawnej odpowiedzi oraz tego, czy dozwolone są wielokrotne wybory:

```php
$name = $this->choice(
    'Jak się nazywasz?',
    ['Taylor', 'Dayle'],
    $defaultIndex,
    $maxAttempts = null,
    $allowMultipleSelections = false
);
```

<a name="writing-output"></a>
### Wypisywanie Wyjścia

Aby wysłać wyjście do konsoli, możesz użyć metod `line`, `newLine`, `info`, `comment`, `question`, `warn`, `alert` i `error`. Każda z tych metod użyje odpowiednich kolorów ANSI zgodnie z ich przeznaczeniem. Na przykład wyświetlmy niektóre ogólne informacje użytkownikowi. Zazwyczaj metoda `info` wyświetli się w konsoli jako zielony tekst:

```php
/**
 * Wykonaj komendę konsolową.
 */
public function handle(): void
{
    // ...

    $this->info('Komenda zakończyła się sukcesem!');
}
```

Aby wyświetlić komunikat o błędzie, użyj metody `error`. Tekst komunikatu o błędzie jest zazwyczaj wyświetlany na czerwono:

```php
$this->error('Coś poszło nie tak!');
```

Możesz użyć metody `line`, aby wyświetlić zwykły, niekolorowy tekst:

```php
$this->line('Wyświetl to na ekranie');
```

Możesz użyć metody `newLine`, aby wyświetlić pustą linię:

```php
// Napisz jedną pustą linię...
$this->newLine();

// Napisz trzy puste linie...
$this->newLine(3);
```

<a name="tables"></a>
#### Tabele

Metoda `table` ułatwia poprawne formatowanie wielu wierszy / kolumn danych. Wszystko, co musisz zrobić, to podać nazwy kolumn i dane dla tabeli, a Laravel automatycznie obliczy odpowiednią szerokość i wysokość tabeli:

```php
use App\Models\User;

$this->table(
    ['Name', 'Email'],
    User::all(['name', 'email'])->toArray()
);
```

<a name="progress-bars"></a>
#### Paski Postępu

Dla długotrwałych zadań może być pomocne wyświetlenie paska postępu, który informuje użytkowników o stopniu zakończenia zadania. Używając metody `withProgressBar`, Laravel wyświetli pasek postępu i przesunie jego postęp dla każdej iteracji nad daną wartością iterowalną:

```php
use App\Models\User;

$users = $this->withProgressBar(User::all(), function (User $user) {
    $this->performTask($user);
});
```

Czasami możesz potrzebować więcej ręcznej kontroli nad tym, jak pasek postępu jest przesuwany. Najpierw zdefiniuj całkowitą liczbę kroków, przez które proces będzie iterować. Następnie przesuń pasek postępu po przetworzeniu każdego elementu:

```php
$users = App\Models\User::all();

$bar = $this->output->createProgressBar(count($users));

$bar->start();

foreach ($users as $user) {
    $this->performTask($user);

    $bar->advance();
}

$bar->finish();
```

> [!NOTE]
> Aby poznać bardziej zaawansowane opcje, sprawdź [dokumentację komponentu Symfony Progress Bar](https://symfony.com/doc/current/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>
## Rejestrowanie Komend

Domyślnie Laravel automatycznie rejestruje wszystkie komendy w katalogu `app/Console/Commands`. Jednak możesz polecić Laravel, aby skanował inne katalogi w poszukiwaniu komend Artisan, używając metody `withCommands` w pliku `bootstrap/app.php` Twojej aplikacji:

```php
->withCommands([
    __DIR__.'/../app/Domain/Orders/Commands',
])
```

Jeśli to konieczne, możesz również ręcznie rejestrować komendy, podając nazwę klasy komendy do metody `withCommands`:

```php
use App\Domain\Orders\Commands\SendEmails;

->withCommands([
    SendEmails::class,
])
```

Podczas uruchamiania Artisan wszystkie komendy w Twojej aplikacji zostaną rozstrzygnięte przez [kontener usług](/docs/{{version}}/container) i zarejestrowane w Artisan.

<a name="programmatically-executing-commands"></a>
## Programowe Wykonywanie Komend

Czasami możesz chcieć wykonać komendę Artisan poza CLI. Na przykład możesz chcieć wykonać komendę Artisan z trasy lub kontrolera. Możesz użyć metody `call` na fasadzie `Artisan`, aby to osiągnąć. Metoda `call` przyjmuje jako pierwszy argument nazwę sygnatury komendy lub nazwę klasy, a jako drugi argument tablicę parametrów komendy. Zwracany jest kod wyjścia:

```php
use Illuminate\Support\Facades\Artisan;
use Illuminate\Support\Facades\Route;

Route::post('/user/{user}/mail', function (string $user) {
    $exitCode = Artisan::call('mail:send', [
        'user' => $user, '--queue' => 'default'
    ]);

    // ...
});
```

Alternatywnie możesz przekazać całą komendę Artisan do metody `call` jako łańcuch:

```php
Artisan::call('mail:send 1 --queue=default');
```

<a name="passing-array-values"></a>
#### Przekazywanie Wartości Tablicowych

Jeśli Twoja komenda definiuje opcję, która akceptuje tablicę, możesz przekazać tablicę wartości do tej opcji:

```php
use Illuminate\Support\Facades\Artisan;
use Illuminate\Support\Facades\Route;

Route::post('/mail', function () {
    $exitCode = Artisan::call('mail:send', [
        '--id' => [5, 13]
    ]);
});
```

<a name="passing-boolean-values"></a>
#### Przekazywanie Wartości Logicznych

Jeśli musisz określić wartość opcji, która nie akceptuje wartości łańcuchowych, takich jak flaga `--force` w komendzie `migrate:refresh`, powinieneś przekazać `true` lub `false` jako wartość opcji:

```php
$exitCode = Artisan::call('migrate:refresh', [
    '--force' => true,
]);
```

<a name="queueing-artisan-commands"></a>
#### Umieszczanie Komend Artisan w Kolejce

Używając metody `queue` na fasadzie `Artisan`, możesz nawet umieścić komendy Artisan w kolejce, aby były przetwarzane w tle przez Twoich [workerów kolejek](/docs/{{version}}/queues). Przed użyciem tej metody upewnij się, że skonfigurowałeś swoją kolejkę i uruchomiłeś nasłuchiwacza kolejek:

```php
use Illuminate\Support\Facades\Artisan;
use Illuminate\Support\Facades\Route;

Route::post('/user/{user}/mail', function (string $user) {
    Artisan::queue('mail:send', [
        'user' => $user, '--queue' => 'default'
    ]);

    // ...
});
```

Używając metod `onConnection` i `onQueue`, możesz określić połączenie lub kolejkę, do której komenda Artisan powinna zostać wysłana:

```php
Artisan::queue('mail:send', [
    'user' => 1, '--queue' => 'default'
])->onConnection('redis')->onQueue('commands');
```

<a name="calling-commands-from-other-commands"></a>
### Wywoływanie Komend z Innych Komend

Czasami możesz chcieć wywołać inne komendy z istniejącej komendy Artisan. Możesz to zrobić, używając metody `call`. Ta metoda `call` przyjmuje nazwę komendy oraz tablicę argumentów / opcji komendy:

```php
/**
 * Wykonaj komendę konsolową.
 */
public function handle(): void
{
    $this->call('mail:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    // ...
}
```

Jeśli chcesz wywołać inną komendę konsolową i pominąć całe jej wyjście, możesz użyć metody `callSilently`. Metoda `callSilently` ma tą samą sygnaturę co metoda `call`:

```php
$this->callSilently('mail:send', [
    'user' => 1, '--queue' => 'default'
]);
```

<a name="signal-handling"></a>
## Obsługa Sygnałów

Jak zapewne wiesz, systemy operacyjne pozwalają na wysyłanie sygnałów do uruchomionych procesów. Na przykład sygnał `SIGTERM` to sposób, w jaki systemy operacyjne proszą program o zakończenie działania. Jeśli chcesz nasłuchiwać sygnałów w swoich komendach konsolowych Artisan i wykonywać kod, gdy wystąpią, możesz użyć metody `trap`:

```php
/**
 * Wykonaj komendę konsolową.
 */
public function handle(): void
{
    $this->trap(SIGTERM, fn () => $this->shouldKeepRunning = false);

    while ($this->shouldKeepRunning) {
        // ...
    }
}
```

Aby nasłuchiwać wielu sygnałów jednocześnie, możesz dostarczyć tablicę sygnałów do metody `trap`:

```php
$this->trap([SIGTERM, SIGQUIT], function (int $signal) {
    $this->shouldKeepRunning = false;

    dump($signal); // SIGTERM / SIGQUIT
});
```

<a name="stub-customization"></a>
## Dostosowywanie Stubów

Komendy `make` konsoli Artisan są używane do tworzenia różnych klas, takich jak kontrolery, zadania, migracje i testy. Te klasy są generowane przy użyciu plików "stub", które są wypełniane wartościami na podstawie Twoich danych wejściowych. Jednak możesz chcieć dokonać niewielkich zmian w plikach generowanych przez Artisan. Aby to osiągnąć, możesz użyć komendy `stub:publish`, aby opublikować najczęściej używane stuby do swojej aplikacji, aby móc je dostosować:

```shell
php artisan stub:publish
```

Opublikowane stuby zostaną umieszczone w katalogu `stubs` w katalogu głównym Twojej aplikacji. Wszelkie zmiany wprowadzone w tych stubach zostaną odzwierciedlone podczas generowania odpowiednich klas za pomocą komend `make` Artisan.

<a name="events"></a>
## Zdarzenia

Artisan wysyła trzy zdarzenia podczas uruchamiania komend: `Illuminate\Console\Events\ArtisanStarting`, `Illuminate\Console\Events\CommandStarting` i `Illuminate\Console\Events\CommandFinished`. Zdarzenie `ArtisanStarting` jest wysyłane natychmiast po rozpoczęciu działania Artisan. Następnie zdarzenie `CommandStarting` jest wysyłane natychmiast przed uruchomieniem komendy. Na koniec zdarzenie `CommandFinished` jest wysyłane po zakończeniu wykonywania komendy.
