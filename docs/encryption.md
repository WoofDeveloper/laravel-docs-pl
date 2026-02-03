# Szyfrowanie

- [Wprowadzenie](#introduction)
- [Konfiguracja](#configuration)
    - [Płynna Rotacja Kluczy Szyfrowania](#gracefully-rotating-encryption-keys)
- [Używanie Encryptera](#using-the-encrypter)

<a name="introduction"></a>
## Wprowadzenie

Usługi szyfrowania Laravel zapewniają prosty i wygodny interfejs do szyfrowania i deszyfrowania tekstu za pomocą OpenSSL przy użyciu szyfrowania AES-256 i AES-128. Wszystkie zaszyfrowane wartości Laravel są podpisywane kodem uwierzytelniania wiadomości (MAC), dzięki czemu ich podstawowa wartość nie może zostać zmodyfikowana ani zmanipulowana po zaszyfrowaniu.

<a name="configuration"></a>
## Konfiguracja

Przed użyciem mechanizmu szyfrowania Laravel musisz ustawić opcję konfiguracyjną `key` w pliku konfiguracyjnym `config/app.php`. Ta wartość konfiguracyjna jest sterowana przez zmienną środowiskową `APP_KEY`. Powinieneś użyć polecenia `php artisan key:generate`, aby wygenerować wartość tej zmiennej, ponieważ polecenie `key:generate` użyje generatora bezpiecznych losowych bajtów PHP do zbudowania kryptograficznie bezpiecznego klucza dla Twojej aplikacji. Zazwyczaj wartość zmiennej środowiskowej `APP_KEY` zostanie wygenerowana dla Ciebie podczas [instalacji Laravel](/docs/{{version}}/installation).

<a name="gracefully-rotating-encryption-keys"></a>
### Płynna Rotacja Kluczy Szyfrowania

Jeśli zmienisz klucz szyfrowania swojej aplikacji, wszystkie uwierzytelnione sesje użytkowników zostaną wylogowane z Twojej aplikacji. Dzieje się tak, ponieważ każde cookie, w tym pliki cookie sesji, są szyfrowane przez Laravel. Ponadto nie będzie już możliwe odszyfrowanie żadnych danych, które zostały zaszyfrowane przy użyciu Twojego poprzedniego klucza szyfrowania.

Aby złagodzić ten problem, Laravel pozwala na wylistowanie poprzednich kluczy szyfrowania w zmiennej środowiskowej `APP_PREVIOUS_KEYS` Twojej aplikacji. Ta zmienna może zawierać listę wszystkich poprzednich kluczy szyfrowania oddzielonych przecinkami:

```ini
APP_KEY="base64:J63qRTDLub5NuZvP+kb8YIorGS6qFYHKVo6u7179stY="
APP_PREVIOUS_KEYS="base64:2nLsGFGzyoae2ax3EF2Lyq/hH6QghBGLIq5uL+Gp8/w="
```

Po ustawieniu tej zmiennej środowiskowej Laravel zawsze będzie używał "bieżącego" klucza szyfrowania podczas szyfrowania wartości. Jednak podczas deszyfrowania wartości Laravel najpierw spróbuje użyć bieżącego klucza, a jeśli deszyfrowanie nie powiedzie się przy użyciu bieżącego klucza, Laravel spróbuje wszystkich poprzednich kluczy, aż jeden z kluczy będzie w stanie odszyfrować wartość.

To podejście do płynnego deszyfrowania pozwala użytkownikom na nieprzerwane korzystanie z Twojej aplikacji, nawet jeśli klucz szyfrowania zostanie zmieniony.

<a name="using-the-encrypter"></a>
## Używanie Encryptera

<a name="encrypting-a-value"></a>
#### Szyfrowanie Wartości

Możesz zaszyfrować wartość za pomocą metody `encryptString` dostarczonej przez fasadę `Crypt`. Wszystkie zaszyfrowane wartości są szyfrowane przy użyciu OpenSSL i szyfru AES-256-CBC. Ponadto wszystkie zaszyfrowane wartości są podpisywane kodem uwierzytelniania wiadomości (MAC). Zintegrowany kod uwierzytelniania wiadomości zapobiegnie odszyfrowaniu jakichkolwiek wartości, które zostały zmanipulowane przez złośliwych użytkowników:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Crypt;

class DigitalOceanTokenController extends Controller
{
    /**
     * Store a DigitalOcean API token for the user.
     */
    public function store(Request $request): RedirectResponse
    {
        $request->user()->fill([
            'token' => Crypt::encryptString($request->token),
        ])->save();

        return redirect('/secrets');
    }
}
```

<a name="decrypting-a-value"></a>
#### Deszyfrowanie Wartości

Możesz odszyfrować wartości za pomocą metody `decryptString` dostarczonej przez fasadę `Crypt`. Jeśli wartość nie może być poprawnie odszyfrowana, na przykład gdy kod uwierzytelniania wiadomości jest nieprawidłowy, zostanie zgłoszony wyjątek `Illuminate\Contracts\Encryption\DecryptException`:

```php
use Illuminate\Contracts\Encryption\DecryptException;
use Illuminate\Support\Facades\Crypt;

try {
    $decrypted = Crypt::decryptString($encryptedValue);
} catch (DecryptException $e) {
    // ...
}
```
