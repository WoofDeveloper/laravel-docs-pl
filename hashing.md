# Haszowanie

- [Wprowadzenie](#introduction)
- [Konfiguracja](#configuration)
- [Podstawowe użycie](#basic-usage)
    - [Haszowanie haseł](#hashing-passwords)
    - [Weryfikacja czy hasło pasuje do hasza](#verifying-that-a-password-matches-a-hash)
    - [Określanie czy hasło wymaga ponownego zahaszowania](#determining-if-a-password-needs-to-be-rehashed)
- [Weryfikacja algorytmu haszowania](#hash-algorithm-verification)

<a name="introduction"></a>
## Wprowadzenie

[Fasada](/docs/{{version}}/facades) `Hash` Laravel zapewnia bezpieczne haszowanie Bcrypt i Argon2 do przechowywania haseł użytkowników. Jeśli korzystasz z jednego z [zestawów startowych aplikacji Laravel](/docs/{{version}}/starter-kits), Bcrypt będzie domyślnie używany do rejestracji i uwierzytelniania.

Bcrypt to doskonały wybór do haszowania haseł, ponieważ jego "współczynnik pracy" jest regulowany, co oznacza, że czas potrzebny do wygenerowania hasza można zwiększyć wraz ze wzrostem mocy sprzętowej. Podczas haszowania haseł, wolne jest dobre. Im dłużej algorytm haszuje hasło, tym dłużej trwa złośliwym użytkownikom generowanie "tęczowych tablic" wszystkich możliwych wartości hasza ciągów znaków, które mogą być użyte w atakach brutalnej siły na aplikacje.

<a name="configuration"></a>
## Konfiguracja

Domyślnie Laravel używa sterownika haszowania `bcrypt` podczas haszowania danych. Jednak obsługiwane są również inne sterowniki haszowania, w tym [argon](https://en.wikipedia.org/wiki/Argon2) i [argon2id](https://en.wikipedia.org/wiki/Argon2).

Możesz określić sterownik haszowania aplikacji za pomocą zmiennej środowiskowej `HASH_DRIVER`. Ale jeśli chcesz dostosować wszystkie opcje sterownika haszowania Laravel, powinieneś opublikować kompletny plik konfiguracyjny `hashing` za pomocą komendy Artisan `config:publish`:

```shell
php artisan config:publish hashing
```

<a name="basic-usage"></a>
## Podstawowe użycie

<a name="hashing-passwords"></a>
### Haszowanie haseł

Możesz zahaszować hasło wywołując metodę `make` na fasadzie `Hash`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

class PasswordController extends Controller
{
    /**
     * Aktualizuje hasło użytkownika.
     */
    public function update(Request $request): RedirectResponse
    {
        // Walidacja długości nowego hasła...

        $request->user()->fill([
            'password' => Hash::make($request->newPassword)
        ])->save();

        return redirect('/profile');
    }
}
```

<a name="adjusting-the-bcrypt-work-factor"></a>
#### Dostosowanie współczynnika pracy Bcrypt

Jeśli używasz algorytmu Bcrypt, metoda `make` pozwala zarządzać współczynnikiem pracy algorytmu za pomocą opcji `rounds`; jednak domyślny współczynnik pracy zarządzany przez Laravel jest akceptowalny dla większości aplikacji:

```php
$hashed = Hash::make('password', [
    'rounds' => 12,
]);
```

<a name="adjusting-the-argon2-work-factor"></a>
#### Dostosowanie współczynnika pracy Argon2

Jeśli używasz algorytmu Argon2, metoda `make` pozwala zarządzać współczynnikiem pracy algorytmu za pomocą opcji `memory`, `time` i `threads`; jednak domyślne wartości zarządzane przez Laravel są akceptowalne dla większości aplikacji:

```php
$hashed = Hash::make('password', [
    'memory' => 1024,
    'time' => 2,
    'threads' => 2,
]);
```

> [!NOTE]
> Aby uzyskać więcej informacji na temat tych opcji, zapoznaj się z [oficjalną dokumentacją PHP dotyczącą haszowania Argon](https://secure.php.net/manual/en/function.password-hash.php).

<a name="verifying-that-a-password-matches-a-hash"></a>
### Weryfikacja czy hasło pasuje do hasza

Metoda `check` dostarczona przez fasadę `Hash` pozwala zweryfikować, czy dany ciąg tekstowy odpowiada danemu haszowi:

```php
if (Hash::check('plain-text', $hashedPassword)) {
    // Hasła pasują...
}
```

<a name="determining-if-a-password-needs-to-be-rehashed"></a>
### Określanie czy hasło wymaga ponownego zahaszowania

Metoda `needsRehash` dostarczona przez fasadę `Hash` pozwala określić, czy współczynnik pracy używany przez hasher zmienił się od czasu zahaszowania hasła. Niektóre aplikacje decydują się na wykonanie tego sprawdzenia podczas procesu uwierzytelniania aplikacji:

```php
if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('plain-text');
}
```

<a name="hash-algorithm-verification"></a>
## Weryfikacja algorytmu haszowania

Aby zapobiec manipulacji algorytmem haszowania, metoda `Hash::check` Laravel najpierw zweryfikuje, czy dany hasz został wygenerowany przy użyciu wybranego algorytmu haszowania aplikacji. Jeśli algorytmy są różne, zostanie zgłoszony wyjątek `RuntimeException`.

Jest to oczekiwane zachowanie dla większości aplikacji, gdzie nie oczekuje się zmiany algorytmu haszowania, a różne algorytmy mogą wskazywać na złośliwy atak. Jednak jeśli musisz obsługiwać wiele algorytmów haszowania w swojej aplikacji, np. podczas migracji z jednego algorytmu na drugi, możesz wyłączyć weryfikację algorytmu haszowania ustawiając zmienną środowiskową `HASH_VERIFY` na `false`:

```ini
HASH_VERIFY=false
```
