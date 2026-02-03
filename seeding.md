# Baza danych: Seedowanie

- [Wprowadzenie](#introduction)
- [Pisanie seederów](#writing-seeders)
    - [Używanie fabryk modeli](#using-model-factories)
    - [Wywoływanie dodatkowych seederów](#calling-additional-seeders)
    - [Wyciszanie zdarzeń modeli](#muting-model-events)
- [Uruchamianie seederów](#running-seeders)

<a name="introduction"></a>
## Wprowadzenie

Laravel zawiera możliwość wypełniania bazy danych danymi za pomocą klas seed. Wszystkie klasy seed są przechowywane w katalogu `database/seeders`. Domyślnie zdefiniowana jest dla Ciebie klasa `DatabaseSeeder`. Z tej klasy możesz użyć metody `call`, aby uruchomić inne klasy seed, co pozwala kontrolować kolejność seedowania.

> [!NOTE]
> [Ochrona przed masowym przypisaniem](/docs/{{version}}/eloquent#mass-assignment) jest automatycznie wyłączana podczas seedowania bazy danych.

<a name="writing-seeders"></a>
## Pisanie seederów

Aby wygenerować seeder, wykonaj [polecenie Artisan](/docs/{{version}}/artisan) `make:seeder`. Wszystkie seedery wygenerowane przez framework zostaną umieszczone w katalogu `database/seeders`:

```shell
php artisan make:seeder UserSeeder
```

Klasa seedera domyślnie zawiera tylko jedną metodę: `run`. Ta metoda jest wywoływana, gdy wykonywane jest [polecenie Artisan](/docs/{{version}}/artisan) `db:seed`. W metodzie `run` możesz wstawiać dane do swojej bazy danych w dowolny sposób. Możesz użyć [konstruktora zapytań](/docs/{{version}}/queries) do ręcznego wstawiania danych lub możesz użyć [fabryk modeli Eloquent](/docs/{{version}}/eloquent-factories).

Jako przykład zmodyfikujmy domyślną klasę `DatabaseSeeder` i dodajmy instrukcję wstawiania do bazy danych do metody `run`:

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeders.
     */
    public function run(): void
    {
        DB::table('users')->insert([
            'name' => Str::random(10),
            'email' => Str::random(10).'@example.com',
            'password' => Hash::make('password'),
        ]);
    }
}
```

> [!NOTE]
> Możesz określić type-hint dla dowolnych zależności, których potrzebujesz w sygnaturze metody `run`. Będą one automatycznie rozwiązane przez [kontener usług](/docs/{{version}}/container) Laravel.

<a name="using-model-factories"></a>
### Używanie fabryk modeli

Oczywiście ręczne określanie atrybutów dla każdego seedu modelu jest uciążliwe. Zamiast tego możesz użyć [fabryk modeli](/docs/{{version}}/eloquent-factories), aby wygodnie wygenerować duże ilości rekordów bazy danych. Najpierw przejrzyj [dokumentację fabryk modeli](/docs/{{version}}/eloquent-factories), aby dowiedzieć się, jak definiować swoje fabryki.

Na przykład stwórzmy 50 użytkowników, z których każdy ma jeden powiązany post:

```php
use App\Models\User;

/**
 * Run the database seeders.
 */
public function run(): void
{
    User::factory()
        ->count(50)
        ->hasPosts(1)
        ->create();
}
```

<a name="calling-additional-seeders"></a>
### Wywoływanie dodatkowych seederów

W klasie `DatabaseSeeder` możesz użyć metody `call`, aby wykonać dodatkowe klasy seed. Użycie metody `call` pozwala podzielić seedowanie bazy danych na wiele plików, tak aby żadna pojedyncza klasa seedera nie stała się zbyt duża. Metoda `call` akceptuje tablicę klas seedera, które powinny zostać wykonane:

```php
/**
 * Run the database seeders.
 */
public function run(): void
{
    $this->call([
        UserSeeder::class,
        PostSeeder::class,
        CommentSeeder::class,
    ]);
}
```

<a name="muting-model-events"></a>
### Wyciszanie zdarzeń modeli

Podczas uruchamiania seedów możesz chcieć zapobiec wysyłaniu zdarzeń przez modele. Możesz to osiągnąć, używając traitu `WithoutModelEvents`. Gdy jest używany, trait `WithoutModelEvents` zapewnia, że żadne zdarzenia modelu nie są wysyłane, nawet jeśli dodatkowe klasy seed są wykonywane za pomocą metody `call`:

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Database\Console\Seeds\WithoutModelEvents;

class DatabaseSeeder extends Seeder
{
    use WithoutModelEvents;

    /**
     * Run the database seeders.
     */
    public function run(): void
    {
        $this->call([
            UserSeeder::class,
        ]);
    }
}
```

<a name="running-seeders"></a>
## Uruchamianie seederów

Możesz wykonać polecenie Artisan `db:seed`, aby zaseedować bazę danych. Domyślnie polecenie `db:seed` uruchamia klasę `Database\Seeders\DatabaseSeeder`, która z kolei może wywoływać inne klasy seed. Możesz jednak użyć opcji `--class`, aby określić konkretną klasę seedera do uruchomienia indywidualnie:

```shell
php artisan db:seed

php artisan db:seed --class=UserSeeder
```

Możesz również zaseedować bazę danych za pomocą polecenia `migrate:fresh` w połączeniu z opcją `--seed`, co usunie wszystkie tabele i ponownie uruchomi wszystkie Twoje migracje. To polecenie jest przydatne do całkowitego przebudowania bazy danych. Opcja `--seeder` może być użyta do określenia konkretnego seedera do uruchomienia:

```shell
php artisan migrate:fresh --seed

php artisan migrate:fresh --seed --seeder=UserSeeder
```

<a name="forcing-seeding-production"></a>
#### Wymuszanie uruchamiania seederów w produkcji

Niektóre operacje seedowania mogą spowodować zmianę lub utratę danych. Aby chronić Cię przed uruchamianiem poleceń seedowania na Twojej produkcyjnej bazie danych, zostaniesz poproszony o potwierdzenie, zanim seedery zostaną wykonane w środowisku `production`. Aby wymusić uruchomienie seederów bez monitu, użyj flagi `--force`:

```shell
php artisan db:seed --force
```
