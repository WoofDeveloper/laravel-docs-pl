# Prompts

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
- [Dostępne Zapytania](#available-prompts)
    - [Text](#text)
    - [Textarea](#textarea)
    - [Number](#number)
    - [Password](#password)
    - [Confirm](#confirm)
    - [Select](#select)
    - [Multi-select](#multiselect)
    - [Suggest](#suggest)
    - [Search](#search)
    - [Multi-search](#multisearch)
    - [Pause](#pause)
- [Transforming Input Before Validation](#transforming-input-before-validation)
- [Forms](#forms)
- [Informational Messages](#informational-messages)
- [Tables](#tables)
- [Spin](#spin)
- [Progress Bar](#progress)
- [Clearing the Terminal](#clear)
- [Terminal Considerations](#terminal-considerations)
- [Unsupported Environments and Fallbacks](#fallbacks)
- [Testing](#testing)

<a name="introduction"></a>
## Wprowadzenie

[Laravel Prompts](https://github.com/laravel/prompts) to pakiet PHP do dodawania pięknych i przyjaznych dla użytkownika formularzy do aplikacji wiersza poleceń, z funkcjami podobnymi do przeglądarki, w tym tekstem zastępczym i walidacją.

<img src="https://laravel.com/img/docs/prompts-example.png">

Laravel Prompts idealnie nadaje się do przyjmowania danych wejściowych użytkownika w [poleceniach konsoli Artisan](/docs/{{version}}/artisan#writing-commands), ale może być również używany w dowolnym projekcie PHP wiersza poleceń.

> [!NOTE]
> Laravel Prompts obsługuje macOS, Linux i Windows z WSL. Aby uzyskać więcej informacji, zobacz naszą dokumentację dotyczącą [nieobsługiwanych środowisk i rozwiązań awaryjnych](#fallbacks).

<a name="installation"></a>
## Instalacja

Laravel Prompts jest już dołączony do najnowszej wersji Laravel.

Laravel Prompts może być również zainstalowany w innych projektach PHP za pomocą menedżera pakietów Composer:

```shell
composer require laravel/prompts
```

<a name="available-prompts"></a>
## Dostępne zapytania

<a name="text"></a>
### Tekst

Funkcja `text` wyświetli użytkownikowi podane pytanie, zaakceptuje jego dane wejściowe, a następnie je zwróci:

```php
use function Laravel\Prompts\text;

$name = text('What is your name?');
```

Możesz również dołączyć tekst zastępczy, wartość domyślną i podpowiedź informacyjną:

```php
$name = text(
    label: 'What is your name?',
    placeholder: 'E.g. Taylor Otwell',
    default: $user?->name,
    hint: 'This will be displayed on your profile.'
);
```

<a name="text-required"></a>
#### Wymagane wartości

Jeśli wymagasz wprowadzenia wartości, możesz przekazać argument `required`:

```php
$name = text(
    label: 'What is your name?',
    required: true
);
```

Jeśli chcesz dostosować komunikat walidacji, możesz również przekazać ciąg znaków:

```php
$name = text(
    label: 'What is your name?',
    required: 'Your name is required.'
);
```

<a name="text-validation"></a>
#### Dodatkowa walidacja

Na koniec, jeśli chcesz wykonać dodatkową logikę walidacji, możesz przekazać domknięcie do argumentu `validate`:

```php
$name = text(
    label: 'What is your name?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

Domknięcie otrzyma wprowadzoną wartość i może zwrócić komunikat błędu lub `null`, jeśli walidacja przejdzie.

Alternatywnie możesz wykorzystać moc [walidatora](/docs/{{version}}/validation) Laravel. Aby to zrobić, podaj tablicę zawierającą nazwę atrybutu i żądane reguły walidacji do argumentu `validate`:

```php
$name = text(
    label: 'What is your name?',
    validate: ['name' => 'required|max:255|unique:users']
);
```

<a name="textarea"></a>
### Tekstarea

Funkcja `textarea` wyświetli użytkownikowi podane pytanie, zaakceptuje jego dane wejściowe przez wielowierszowe pole tekstowe, a następnie je zwróci:

```php
use function Laravel\Prompts\textarea;

$story = textarea('Tell me a story.');
```

Możesz również dołączyć tekst zastępczy, wartość domyślną i podpowiedź informacyjną:

```php
$story = textarea(
    label: 'Tell me a story.',
    placeholder: 'This is a story about...',
    hint: 'This will be displayed on your profile.'
);
```

<a name="textarea-required"></a>
#### Wymagane wartości

Jeśli wymagasz wprowadzenia wartości, możesz przekazać argument `required`:

```php
$story = textarea(
    label: 'Tell me a story.',
    required: true
);
```

Jeśli chcesz dostosować komunikat walidacji, możesz również przekazać ciąg znaków:

```php
$story = textarea(
    label: 'Tell me a story.',
    required: 'A story is required.'
);
```

<a name="textarea-validation"></a>
#### Dodatkowa walidacja

Na koniec, jeśli chcesz wykonać dodatkową logikę walidacji, możesz przekazać domknięcie do argumentu `validate`:

```php
$story = textarea(
    label: 'Tell me a story.',
    validate: fn (string $value) => match (true) {
        strlen($value) < 250 => 'The story must be at least 250 characters.',
        strlen($value) > 10000 => 'The story must not exceed 10,000 characters.',
        default => null
    }
);
```

Domknięcie otrzyma wprowadzoną wartość i może zwrócić komunikat błędu lub `null`, jeśli walidacja przejdzie.

Alternatywnie możesz wykorzystać moc [walidatora](/docs/{{version}}/validation) Laravel. Aby to zrobić, podaj tablicę zawierającą nazwę atrybutu i żądane reguły walidacji do argumentu `validate`:

```php
$story = textarea(
    label: 'Tell me a story.',
    validate: ['story' => 'required|max:10000']
);
```

<a name="number"></a>
### Liczba

Funkcja `number` wyświetli użytkownikowi podane pytanie, zaakceptuje jego dane liczbowe, a następnie je zwróci. Funkcja `number` pozwala użytkownikowi używać klawiszy strzałek w górę i w dół do manipulowania liczbą:

```php
use function Laravel\Prompts\number;

$number = number('How many copies would you like?');
```

Możesz również dołączyć tekst zastępczy, wartość domyślną i podpowiedź informacyjną:

```php
$name = number(
    label: 'How many copies would you like?',
    placeholder: '5',
    default: 1,
    hint: 'This will be determine how many copies to create.'
);
```

<a name="number-required"></a>
#### Wymagane wartości

Jeśli wymagasz wprowadzenia wartości, możesz przekazać argument `required`:

```php
$copies = number(
    label: 'How many copies would you like?',
    required: true
);
```

Jeśli chcesz dostosować komunikat walidacji, możesz również przekazać ciąg znaków:

```php
$copies = number(
    label: 'How many copies would you like?',
    required: 'A number of copies is required.'
);
```

<a name="number-validation"></a>
#### Dodatkowa walidacja

Na koniec, jeśli chcesz wykonać dodatkową logikę walidacji, możesz przekazać domknięcie do argumentu `validate`:

```php
$copies = number(
    label: 'How many copies would you like?',
    validate: fn (?int $value) => match (true) {
        $value < 1 => 'At least one copy is required.',
        $value > 100 => 'You may not create more than 100 copies.',
        default => null
    }
);
```

Domknięcie otrzyma wprowadzoną wartość i może zwrócić komunikat błędu lub `null`, jeśli walidacja przejdzie.

Alternatywnie możesz wykorzystać moc [walidatora](/docs/{{version}}/validation) Laravel. Aby to zrobić, podaj tablicę zawierającą nazwę atrybutu i żądane reguły walidacji do argumentu `validate`:

```php
$copies = number(
    label: 'How many copies would you like?',
    validate: ['copies' => 'required|integer|min:1|max:100']
);
```

<a name="password"></a>
### Hasło

Funkcja `password` jest podobna do funkcji `text`, ale dane wejściowe użytkownika będą ukryte podczas wpisywania w konsoli. Jest to przydatne przy pytaniu o poufne informacje, takie jak hasła:

```php
use function Laravel\Prompts\password;

$password = password('What is your password?');
```

Możesz również dołączyć tekst zastępczy i podpowiedź informacyjną:

```php
$password = password(
    label: 'What is your password?',
    placeholder: 'password',
    hint: 'Minimum 8 characters.'
);
```

<a name="password-required"></a>
#### Wymagane wartości

Jeśli wymagasz wprowadzenia wartości, możesz przekazać argument `required`:

```php
$password = password(
    label: 'What is your password?',
    required: true
);
```

Jeśli chcesz dostosować komunikat walidacji, możesz również przekazać ciąg znaków:

```php
$password = password(
    label: 'What is your password?',
    required: 'The password is required.'
);
```

<a name="password-validation"></a>
#### Dodatkowa walidacja

Na koniec, jeśli chcesz wykonać dodatkową logikę walidacji, możesz przekazać domknięcie do argumentu `validate`:

```php
$password = password(
    label: 'What is your password?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 8 => 'The password must be at least 8 characters.',
        default => null
    }
);
```

Domknięcie otrzyma wprowadzoną wartość i może zwrócić komunikat błędu lub `null`, jeśli walidacja przejdzie.

Alternatywnie możesz wykorzystać moc [walidatora](/docs/{{version}}/validation) Laravel. Aby to zrobić, podaj tablicę zawierającą nazwę atrybutu i żądane reguły walidacji do argumentu `validate`:

```php
$password = password(
    label: 'What is your password?',
    validate: ['password' => 'min:8']
);
```

<a name="confirm"></a>
### Potwierdzenie

Jeśli musisz zapytać użytkownika o potwierdzenie "tak lub nie", możesz użyć funkcji `confirm`. Użytkownicy mogą używać klawiszy strzałek lub nacisnąć `y` lub `n`, aby wybrać swoją odpowiedź. Ta funkcja zwróci `true` lub `false`.

```php
use function Laravel\Prompts\confirm;

$confirmed = confirm('Do you accept the terms?');
```

Możesz również dołączyć wartość domyślną, dostosowane sformułowania dla etykiet "Tak" i "Nie" oraz podpowiedź informacyjną:

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    default: false,
    yes: 'I accept',
    no: 'I decline',
    hint: 'The terms must be accepted to continue.'
);
```

<a name="confirm-required"></a>
#### Wymaganie "Tak"

W razie potrzeby możesz wymagać od użytkowników wybrania "Tak", przekazując argument `required`:

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    required: true
);
```

Jeśli chcesz dostosować komunikat walidacji, możesz również przekazać ciąg znaków:

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    required: 'You must accept the terms to continue.'
);
```

<a name="select"></a>
### Wybór

Jeśli potrzebujesz, aby użytkownik wybrał z predefiniowanego zestawu wyborów, możesz użyć funkcji `select`:

```php
use function Laravel\Prompts\select;

$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner']
);
```

Możesz również określić domyślny wybór i podpowiedź informacyjną:

```php
$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner'],
    default: 'Owner',
    hint: 'The role may be changed at any time.'
);
```

Możesz również przekazać tablicę asocjacyjną do argumentu `options`, aby zwrócić wybrany klucz zamiast jego wartości:

```php
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner',
    ],
    default: 'owner'
);
```

Do pięciu opcji będzie wyświetlanych przed rozpoczęciem przewijania listy. Możesz to dostosować, przekazując argument `scroll`:

```php
$role = select(
    label: 'Which category would you like to assign?',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

<a name="select-validation"></a>
#### Dodatkowa walidacja

W przeciwieństwie do innych funkcji zapytań, funkcja `select` nie akceptuje argumentu `required`, ponieważ nie można wybrać niczego. Możesz jednak przekazać domknięcie do argumentu `validate`, jeśli musisz przedstawić opcję, ale zapobiec jej wybraniu:

```php
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner',
    ],
    validate: fn (string $value) =>
        $value === 'owner' && User::where('role', 'owner')->exists()
            ? 'An owner already exists.'
            : null
);
```

Jeśli argument `options` jest tablicą asocjacyjną, domknięcie otrzyma wybrany klucz, w przeciwnym razie otrzyma wybraną wartość. Domknięcie może zwrócić komunikat błędu lub `null`, jeśli walidacja przejdzie.

<a name="multiselect"></a>
### Wielokrotny wybór

Jeśli potrzebujesz, aby użytkownik mógł wybrać wiele opcji, możesz użyć funkcji `multiselect`:

```php
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: ['Read', 'Create', 'Update', 'Delete']
);
```

Możesz również określić domyślne wybory i podpowiedź informacyjną:

```php
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: ['Read', 'Create', 'Update', 'Delete'],
    default: ['Read', 'Create'],
    hint: 'Permissions may be updated at any time.'
);
```

You may also pass an associative array to the `options` argument to return the selected options' keys instead of their values:

```php
$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: [
        'read' => 'Read',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete',
    ],
    default: ['read', 'create']
);
```

Do pięciu opcji będzie wyświetlanych przed rozpoczęciem przewijania listy. Możesz to dostosować, przekazując argument `scroll`:

```php
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

<a name="multiselect-required"></a>
#### Wymaganie wartości

Domyślnie użytkownik może wybrać zero lub więcej opcji. Możesz przekazać argument `required`, aby wymuszać jedną lub więcej opcji:

```php
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    required: true
);
```

Jeśli chcesz dostosować komunikat walidacji, możesz przekazać ciąg znaków do argumentu `required`:

```php
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    required: 'You must select at least one category'
);
```

<a name="multiselect-validation"></a>
#### Dodatkowa walidacja

Możesz przekazać domknięcie do argumentu `validate`, jeśli musisz przedstawić opcję, ale zapobiec jej wybraniu:

```php
$permissions = multiselect(
    label: 'What permissions should the user have?',
    options: [
        'read' => 'Read',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete',
    ],
    validate: fn (array $values) => ! in_array('read', $values)
        ? 'All users require the read permission.'
        : null
);
```

Jeśli argument `options` jest tablicą asocjacyjną, domknięcie otrzyma wybrane klucze, w przeciwnym razie otrzyma wybrane wartości. Domknięcie może zwrócić komunikat błędu lub `null`, jeśli walidacja przejdzie.

<a name="suggest"></a>
### Sugestia

Funkcja `suggest` może być używana do zapewnienia autouzupełniania dla możliwych wyborów. Użytkownik nadal może podać dowolną odpowiedź, niezależnie od podpowiedzi autouzupełniania:

```php
use function Laravel\Prompts\suggest;

$name = suggest('What is your name?', ['Taylor', 'Dayle']);
```

Alternatywnie możesz przekazać domknięcie jako drugi argument do funkcji `suggest`. Domknięcie zostanie wywołane za każdym razem, gdy użytkownik wpisze znak. Domknięcie powinno akceptować parametr ciągu zawierający dotychczasowe dane wejściowe użytkownika i zwracać tablicę opcji do autouzupełniania:

```php
$name = suggest(
    label: 'What is your name?',
    options: fn ($value) => collect(['Taylor', 'Dayle'])
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
)
```

Możesz również dołączyć tekst zastępczy, wartość domyślną i podpowiedź informacyjną:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    placeholder: 'E.g. Taylor',
    default: $user?->name,
    hint: 'This will be displayed on your profile.'
);
```

<a name="suggest-required"></a>
#### Wymagane wartości

Jeśli wymagasz wprowadzenia wartości, możesz przekazać argument `required`:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    required: true
);
```

Jeśli chcesz dostosować komunikat walidacji, możesz również przekazać ciąg znaków:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    required: 'Your name is required.'
);
```

<a name="suggest-validation"></a>
#### Dodatkowa walidacja

Na koniec, jeśli chcesz wykonać dodatkową logikę walidacji, możesz przekazać domknięcie do argumentu `validate`:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

Domknięcie otrzyma wprowadzoną wartość i może zwrócić komunikat błędu lub `null`, jeśli walidacja przejdzie.

Alternatywnie możesz wykorzystać moc [walidatora](/docs/{{version}}/validation) Laravel. Aby to zrobić, podaj tablicę zawierającą nazwę atrybutu i żądane reguły walidacji do argumentu `validate`:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    validate: ['name' => 'required|min:3|max:255']
);
```

<a name="search"></a>
### Wyszukiwanie

Jeśli masz wiele opcji do wyboru dla użytkownika, funkcja `search` pozwala użytkownikowi wpisać zapytanie wyszukiwania, aby przefiltrować wyniki przed użyciem klawiszy strzałek do wybrania opcji:

```php
use function Laravel\Prompts\search;

$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

Domknięcie otrzyma tekst, który został dotychczas wpisany przez użytkownika i musi zwrócić tablicę opcji. Jeśli zwrócisz tablicę asocjacyjną, zwrócony zostanie klucz wybranej opcji, w przeciwnym razie zwrócona zostanie jej wartość.

Podczas filtrowania tablicy, w której zamierzasz zwrócić wartość, powinieneś użyć funkcji `array_values` lub metody `values` kolekcji, aby upewnić się, że tablica nie stanie się asocjacyjna:

```php
$names = collect(['Taylor', 'Abigail']);

$selected = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => $names
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
        ->values()
        ->all(),
);
```

Możesz również dołączyć tekst zastępczy i podpowiedź informacyjną:

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    placeholder: 'E.g. Taylor Otwell',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'The user will receive an email immediately.'
);
```

Do pięciu opcji będzie wyświetlanych przed rozpoczęciem przewijania listy. Możesz to dostosować, przekazując argument `scroll`:

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

<a name="search-validation"></a>
#### Dodatkowa walidacja

Jeśli chcesz wykonać dodatkową logikę walidacji, możesz przekazać domknięcie do argumentu `validate`:

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (int|string $value) {
        $user = User::findOrFail($value);

        if ($user->opted_out) {
            return 'This user has opted-out of receiving mail.';
        }
    }
);
```

Jeśli argument `options` zwraca tablicę asocjacyjną, domknięcie otrzyma wybrany klucz, w przeciwnym razie otrzyma wybraną wartość. Domknięcie może zwrócić komunikat błędu lub `null`, jeśli walidacja przejdzie.

<a name="multisearch"></a>
### Wielokrotne wyszukiwanie

Jeśli masz wiele opcji wyszukiwania i potrzebujesz, aby użytkownik mógł wybrać wiele elementów, funkcja `multisearch` pozwala użytkownikowi wpisać zapytanie wyszukiwania, aby przefiltrować wyniki przed użyciem klawiszy strzałek i spacji do wybrania opcji:

```php
use function Laravel\Prompts\multisearch;

$ids = multisearch(
    'Search for the users that should receive the mail',
    fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

Domknięcie otrzyma tekst, który został dotychczas wpisany przez użytkownika i musi zwrócić tablicę opcji. Jeśli zwrócisz tablicę asocjacyjną, zwrócone zostaną klucze wybranych opcji; w przeciwnym razie zwrócone zostaną ich wartości.

Podczas filtrowania tablicy, w której zamierzasz zwrócić wartość, powinieneś użyć funkcji `array_values` lub metody `values` kolekcji, aby upewnić się, że tablica nie stanie się asocjacyjna:

```php
$names = collect(['Taylor', 'Abigail']);

$selected = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => $names
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
        ->values()
        ->all(),
);
```

Możesz również dołączyć tekst zastępczy i podpowiedź informacyjną:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    placeholder: 'E.g. Taylor Otwell',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'The user will receive an email immediately.'
);
```

Do pięciu opcji będzie wyświetlanych przed rozpoczęciem przewijania listy. Możesz to dostosować, przekazując argument `scroll`:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

<a name="multisearch-required"></a>
#### Wymaganie wartości

Domyślnie użytkownik może wybrać zero lub więcej opcji. Możesz przekazać argument `required`, aby wymuszać jedną lub więcej opcji:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: true
);
```

Jeśli chcesz dostosować komunikat walidacji, możesz również przekazać ciąg znaków do argumentu `required`:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: 'You must select at least one user.'
);
```

<a name="multisearch-validation"></a>
#### Dodatkowa walidacja

Jeśli chcesz wykonać dodatkową logikę walidacji, możesz przekazać domknięcie do argumentu `validate`:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (array $values) {
        $optedOut = User::whereLike('name', '%a%')->findMany($values);

        if ($optedOut->isNotEmpty()) {
            return $optedOut->pluck('name')->join(', ', ', and ').' have opted out.';
        }
    }
);
```

Jeśli domknięcie `options` zwraca tablicę asocjacyjną, domknięcie otrzyma wybrane klucze; w przeciwnym razie otrzyma wybrane wartości. Domknięcie może zwrócić komunikat błędu lub `null`, jeśli walidacja przejdzie.

<a name="pause"></a>
### Pauza

Funkcja `pause` może być użyta do wyświetlenia tekstu informacyjnego użytkownikowi i oczekiwania na potwierdzenie chęci kontynuowania przez naciśnięcie klawisza Enter / Return:

```php
use function Laravel\Prompts\pause;

pause('Press ENTER to continue.');
```

<a name="transforming-input-before-validation"></a>
## Przekształcanie danych wejściowych przed walidacją

Czasami możesz chcieć przekształcić dane wejściowe zapytania przed walidacją. Na przykład możesz chcieć usunąć białe znaki z dowolnych dostarczonych ciągów. Aby to osiągnąć, wiele funkcji zapytań zapewnia argument `transform`, który akceptuje domknięcie:

```php
$name = text(
    label: 'What is your name?',
    transform: fn (string $value) => trim($value),
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

<a name="forms"></a>
## Formularze

Często będziesz miał wiele zapytań, które będą wyświetlane kolejno w celu zebrania informacji przed wykonaniem dodatkowych działań. Możesz użyć funkcji `form`, aby utworzyć zgrupowany zestaw zapytań do wypełnienia przez użytkownika:

```php
use function Laravel\Prompts\form;

$responses = form()
    ->text('What is your name?', required: true)
    ->password('What is your password?', validate: ['password' => 'min:8'])
    ->confirm('Do you accept the terms?')
    ->submit();
```

Metoda `submit` zwróci tablicę indeksowaną numerycznie zawierającą wszystkie odpowiedzi z zapytań formularza. Możesz jednak podać nazwę dla każdego zapytania za pomocą argumentu `name`. Gdy nazwa jest podana, odpowiedź nazwanego zapytania może być dostępna przez tę nazwę:

```php
use App\Models\User;
use function Laravel\Prompts\form;

$responses = form()
    ->text('What is your name?', required: true, name: 'name')
    ->password(
        label: 'What is your password?',
        validate: ['password' => 'min:8'],
        name: 'password'
    )
    ->confirm('Do you accept the terms?')
    ->submit();

User::create([
    'name' => $responses['name'],
    'password' => $responses['password'],
]);
```

Główną korzyścią z używania funkcji `form` jest możliwość powrotu użytkownika do poprzednich zapytań w formularzu za pomocą `CTRL + U`. Pozwala to użytkownikowi naprawić błędy lub zmienić wybory bez konieczności anulowania i ponownego uruchamiania całego formularza.

Jeśli potrzebujesz bardziej szczegółowej kontroli nad zapytaniem w formularzu, możesz wywołać metodę `add` zamiast bezpośredniego wywołania jednej z funkcji zapytań. Metoda `add` otrzymuje wszystkie poprzednie odpowiedzi dostarczone przez użytkownika:

```php
use function Laravel\Prompts\form;
use function Laravel\Prompts\outro;
use function Laravel\Prompts\text;

$responses = form()
    ->text('What is your name?', required: true, name: 'name')
    ->add(function ($responses) {
        return text("How old are you, {$responses['name']}?");
    }, name: 'age')
    ->submit();

outro("Your name is {$responses['name']} and you are {$responses['age']} years old.");
```

<a name="informational-messages"></a>
## Komunikaty informacyjne

Funkcje `note`, `info`, `warning`, `error` i `alert` mogą być użyte do wyświetlenia komunikatów informacyjnych:

```php
use function Laravel\Prompts\info;

info('Package installed successfully.');
```

<a name="tables"></a>
## Tabele

Funkcja `table` ułatwia wyświetlanie wielu wierszy i kolumn danych. Wszystko, co musisz zrobić, to podać nazwy kolumn i dane dla tabeli:

```php
use function Laravel\Prompts\table;

table(
    headers: ['Name', 'Email'],
    rows: User::all(['name', 'email'])->toArray()
);
```

<a name="spin"></a>
## Spinner

Funkcja `spin` wyświetla spinner wraz z opcjonalnym komunikatem podczas wykonywania określonego wywołania zwrotnego. Służy do wskazania trwających procesów i zwraca wyniki wywołania zwrotnego po zakończeniu:

```php
use function Laravel\Prompts\spin;

$response = spin(
    callback: fn () => Http::get('http://example.com'),
    message: 'Fetching response...'
);
```

> [!WARNING]
> Funkcja `spin` wymaga rozszerzenia PHP [PCNTL](https://www.php.net/manual/en/book.pcntl.php) do animowania spinnera. Gdy to rozszerzenie nie jest dostępne, zamiast tego pojawi się statyczna wersja spinnera.

<a name="progress"></a>
## Paski postępu

Dla długotrwałych zadań pomocne może być pokazanie paska postępu, który informuje użytkowników, jak bardzo zadanie jest ukończone. Używając funkcji `progress`, Laravel wyświetli pasek postępu i będzie rozwijał jego postęp dla każdej iteracji nad daną wartością iterowalną:

```php
use function Laravel\Prompts\progress;

$users = progress(
    label: 'Updating users',
    steps: User::all(),
    callback: fn ($user) => $this->performTask($user)
);
```

Funkcja `progress` działa jak funkcja map i zwróci tablicę zawierającą wartość zwracaną każdej iteracji Twojego wywołania zwrotnego.

Wywołanie zwrotne może również akceptować instancję `Laravel\Prompts\Progress`, pozwalając Ci modyfikować etykietę i podpowiedź w każdej iteracji:

```php
$users = progress(
    label: 'Updating users',
    steps: User::all(),
    callback: function ($user, $progress) {
        $progress
            ->label("Updating {$user->name}")
            ->hint("Created on {$user->created_at}");

        return $this->performTask($user);
    },
    hint: 'This may take some time.'
);
```

Czasami możesz potrzebować bardziej ręcznej kontroli nad tym, jak pasek postępu jest rozwijany. Najpierw zdefiniuj całkowitą liczbę kroków, przez które proces będzie iterował. Następnie rozwijaj pasek postępu za pomocą metody `advance` po przetworzeniu każdego elementu:

```php
$progress = progress(label: 'Updating users', steps: 10);

$users = User::all();

$progress->start();

foreach ($users as $user) {
    $this->performTask($user);

    $progress->advance();
}

$progress->finish();
```

<a name="clear"></a>
## Czyszczenie terminala

Funkcja `clear` może być użyta do wyczyszczenia terminala użytkownika:

```php
use function Laravel\Prompts\clear;

clear();
```

<a name="terminal-considerations"></a>
## Uwagi dotyczące terminala

<a name="terminal-width"></a>
#### Szerokość terminala

Jeśli długość jakiejkolwiek etykiety, opcji lub komunikatu walidacji przekracza liczbę "kolumn" w terminalu użytkownika, zostanie automatycznie obcięta, aby pasować. Rozważ minimalizację długości tych ciągów, jeśli Twoi użytkownicy mogą używać węższych terminali. Typowa bezpieczna maksymalna długość to 74 znaki, aby wspierać 80-znakowy terminal.

<a name="terminal-height"></a>
#### Wysokość terminala

Dla wszystkich zapytań, które akceptują argument `scroll`, skonfigurowana wartość zostanie automatycznie zmniejszona, aby pasować do wysokości terminala użytkownika, w tym miejsca na komunikat walidacji.

<a name="fallbacks"></a>
## Nieobsługiwane środowiska i rozwiązania awaryjne

Laravel Prompts obsługuje macOS, Linux i Windows z WSL. Ze względu na ograniczenia w wersji PHP dla Windows, obecnie nie jest możliwe używanie Laravel Prompts w systemie Windows poza WSL.

Z tego powodu Laravel Prompts obsługuje powrót do alternatywnej implementacji, takiej jak [Symfony Console Question Helper](https://symfony.com/doc/current/components/console/helpers/questionhelper.html).

> [!NOTE]
> Podczas używania Laravel Prompts z frameworkiem Laravel, rozwiązania awaryjne dla każdego zapytania zostały skonfigurowane dla Ciebie i zostaną automatycznie włączone w nieobsługiwanych środowiskach.

<a name="fallback-conditions"></a>
#### Warunki awaryjne

Jeśli nie używasz Laravel lub musisz dostosować, kiedy zachowanie awaryjne jest używane, możesz przekazać wartość logiczną do statycznej metody `fallbackWhen` na klasie `Prompt`:

```php
use Laravel\Prompts\Prompt;

Prompt::fallbackWhen(
    ! $input->isInteractive() || windows_os() || app()->runningUnitTests()
);
```

<a name="fallback-behavior"></a>
#### Zachowanie awaryjne

Jeśli nie używasz Laravel lub musisz dostosować zachowanie awaryjne, możesz przekazać domknięcie do statycznej metody `fallbackUsing` na każdej klasie zapytania:

```php
use Laravel\Prompts\TextPrompt;
use Symfony\Component\Console\Question\Question;
use Symfony\Component\Console\Style\SymfonyStyle;

TextPrompt::fallbackUsing(function (TextPrompt $prompt) use ($input, $output) {
    $question = (new Question($prompt->label, $prompt->default ?: null))
        ->setValidator(function ($answer) use ($prompt) {
            if ($prompt->required && $answer === null) {
                throw new \RuntimeException(
                    is_string($prompt->required) ? $prompt->required : 'Required.'
                );
            }

            if ($prompt->validate) {
                $error = ($prompt->validate)($answer ?? '');

                if ($error) {
                    throw new \RuntimeException($error);
                }
            }

            return $answer;
        });

    return (new SymfonyStyle($input, $output))
        ->askQuestion($question);
});
```

Rozwiązania awaryjne muszą być skonfigurowane indywidualnie dla każdej klasy zapytania. Domknięcie otrzyma instancję klasy zapytania i musi zwrócić odpowiedni typ dla zapytania.

<a name="testing"></a>
## Testowanie

Laravel zapewnia różne metody testowania, że Twoje polecenie wyświetla oczekiwane komunikaty zapytań:

```php tab=Pest
test('report generation', function () {
    $this->artisan('report:generate')
        ->expectsPromptsInfo('Welcome to the application!')
        ->expectsPromptsWarning('This action cannot be undone')
        ->expectsPromptsError('Something went wrong')
        ->expectsPromptsAlert('Important notice!')
        ->expectsPromptsIntro('Starting process...')
        ->expectsPromptsOutro('Process completed!')
        ->expectsPromptsTable(
            headers: ['Name', 'Email'],
            rows: [
                ['Taylor Otwell', 'taylor@example.com'],
                ['Jason Beggs', 'jason@example.com'],
            ]
        )
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
public function test_report_generation(): void
{
    $this->artisan('report:generate')
        ->expectsPromptsInfo('Welcome to the application!')
        ->expectsPromptsWarning('This action cannot be undone')
        ->expectsPromptsError('Something went wrong')
        ->expectsPromptsAlert('Important notice!')
        ->expectsPromptsIntro('Starting process...')
        ->expectsPromptsOutro('Process completed!')
        ->expectsPromptsTable(
            headers: ['Name', 'Email'],
            rows: [
                ['Taylor Otwell', 'taylor@example.com'],
                ['Jason Beggs', 'jason@example.com'],
            ]
        )
        ->assertExitCode(0);
}
```
