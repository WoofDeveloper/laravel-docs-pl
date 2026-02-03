# Resetowanie haseł

- [Wprowadzenie](#introduction)
    - [Konfiguracja](#configuration)
    - [Wymagania sterownika](#driver-prerequisites)
    - [Przygotowanie modelu](#model-preparation)
    - [Konfiguracja zaufanych hostów](#configuring-trusted-hosts)
- [Trasy](#routing)
    - [Żądanie linku resetowania hasła](#requesting-the-password-reset-link)
    - [Resetowanie hasła](#resetting-the-password)
- [Usuwanie wygasłych tokenów](#deleting-expired-tokens)
- [Dostosowywanie](#password-customization)

<a name="introduction"></a>
## Wprowadzenie

Większość aplikacji webowych zapewnia użytkownikom sposób na zresetowanie zapomnianych haseł. Zamiast zmuszać Cię do ponownego implementowania tego ręcznie dla każdej tworzonej aplikacji, Laravel zapewnia wygodne usługi do wysyłania linków resetowania hasła i bezpiecznego resetowania haseł.

> [!NOTE]
> Chcesz szybko zacząć? Zainstaluj [zestaw startowy aplikacji](/docs/{{version}}/starter-kits) Laravel w nowej aplikacji Laravel. Zestawy startowe Laravel zadbają o stworzenie szkieletu całego systemu uwierzytelniania, w tym resetowania zapomnianych haseł.

<a name="configuration"></a>
### Konfiguracja

Plik konfiguracyjny resetowania hasła Twojej aplikacji znajduje się w `config/auth.php`. Upewnij się, że przejrzałeś dostępne opcje w tym pliku. Domyślnie Laravel jest skonfigurowany do używania sterownika resetowania hasła `database`.

Opcja konfiguracji `driver` resetowania hasła określa, gdzie będą przechowywane dane resetowania hasła. Laravel zawiera dwa sterowniki:

<div class="content-list" markdown="1">

- `database` - dane resetowania hasła są przechowywane w relacyjnej bazie danych.
- `cache` - dane resetowania hasła są przechowywane w jednym z magazynów opartych na pamięci podręcznej.

</div>

<a name="driver-prerequisites"></a>
### Wymagania sterownika

<a name="database"></a>
#### Baza danych

Podczas używania domyślnego sterownika `database`, należy utworzyć tabelę do przechowywania tokenów resetowania hasła Twojej aplikacji. Zazwyczaj jest to zawarte w domyślnej migracji bazy danych Laravel `0001_01_01_000000_create_users_table.php`.

<a name="cache"></a>
#### Cache

Dostępny jest również sterownik pamięci podręcznej do obsługi resetowania haseł, który nie wymaga dedykowanej tabeli bazy danych. Wpisy są indeksowane według adresu e-mail użytkownika, więc upewnij się, że nie używasz adresów e-mail jako klucza pamięci podręcznej w innym miejscu w swojej aplikacji:

```php
'passwords' => [
    'users' => [
        'driver' => 'cache',
        'provider' => 'users',
        'store' => 'passwords', // Optional...
        'expire' => 60,
        'throttle' => 60,
    ],
],
```

Aby zapobiec wywołaniu `artisan cache:clear` od opróżnienia danych resetowania hasła, możesz opcjonalnie określić osobny magazyn pamięci podręcznej za pomocą klucza konfiguracji `store`. Wartość powinna odpowiadać magazynowi skonfigurowanemu w pliku konfiguracyjnym `config/cache.php`.

<a name="model-preparation"></a>
### Przygotowanie modelu

Przed użyciem funkcji resetowania hasła Laravel, model `App\Models\User` Twojej aplikacji musi używać traitu `Illuminate\Notifications\Notifiable`. Zazwyczaj ten trait jest już zawarty w domyślnym modelu `App\Models\User`, który jest tworzony z nowymi aplikacjami Laravel.

Następnie sprawdź, czy Twój model `App\Models\User` implementuje kontrakt `Illuminate\Contracts\Auth\CanResetPassword`. Model `App\Models\User` zawarty w frameworku już implementuje ten interfejs i używa traitu `Illuminate\Auth\Passwords\CanResetPassword`, aby zawrzeć metody potrzebne do implementacji interfejsu.

<a name="configuring-trusted-hosts"></a>
### Konfiguracja zaufanych hostów

Domyślnie Laravel odpowie na wszystkie żądania, które otrzyma, niezależnie od zawartości nagłówka `Host` żądania HTTP. Ponadto wartość nagłówka `Host` będzie używana podczas generowania bezwzględnych adresów URL do Twojej aplikacji podczas żądania webowego.

Zazwyczaj powinieneś skonfigurować swój serwer webowy, taki jak Nginx lub Apache, aby wysyłał żądania do Twojej aplikacji tylko wtedy, gdy pasują do danej nazwy hosta. Jednak jeśli nie masz możliwości bezpośredniego dostosowania swojego serwera webowego i musisz poinstruować Laravel, aby odpowiadał tylko na określone nazwy hostów, możesz to zrobić, używając metody middleware `trustHosts` w pliku `bootstrap/app.php` Twojej aplikacji. Jest to szczególnie ważne, gdy Twoja aplikacja oferuje funkcjonalność resetowania hasła.

Aby dowiedzieć się więcej o tej metodzie middleware, zapoznaj się z [dokumentacją middleware TrustHosts](/docs/{{version}}/requests#configuring-trusted-hosts).

<a name="routing"></a>
## Trasy

Aby prawidłowo zaimplementować obsługę resetowania haseł przez użytkowników, musimy zdefiniować kilka tras. Po pierwsze, będziemy potrzebować pary tras do obsługi umożliwienia użytkownikowi żądania linku resetowania hasła za pomocą swojego adresu e-mail. Po drugie, będziemy potrzebować pary tras do obsługi faktycznego resetowania hasła, gdy użytkownik odwiedzi link resetowania hasła wysłany do niego e-mailem i wypełni formularz resetowania hasła.

<a name="requesting-the-password-reset-link"></a>
### Żądanie linku resetowania hasła

<a name="the-password-reset-link-request-form"></a>
#### Formularz żądania linku resetowania hasła

Najpierw zdefiniujemy trasy, które są potrzebne do żądania linków resetowania hasła. Na początek zdefiniujemy trasę, która zwraca widok z formularzem żądania linku resetowania hasła:

```php
Route::get('/forgot-password', function () {
    return view('auth.forgot-password');
})->middleware('guest')->name('password.request');
```

Widok zwracany przez tę trasę powinien zawierać formularz zawierający pole `email`, które pozwoli użytkownikowi zażądać linku resetowania hasła dla danego adresu e-mail.

<a name="password-reset-link-handling-the-form-submission"></a>
#### Obsługa przesłania formularza

Następnie zdefiniujemy trasę, która obsługuje żądanie przesłania formularza z widoku "forgot password". Ta trasa będzie odpowiedzialna za walidację adresu e-mail i wysłanie żądania resetowania hasła do odpowiedniego użytkownika:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Password;

Route::post('/forgot-password', function (Request $request) {
    $request->validate(['email' => 'required|email']);

    $status = Password::sendResetLink(
        $request->only('email')
    );

    return $status === Password::ResetLinkSent
        ? back()->with(['status' => __($status)])
        : back()->withErrors(['email' => __($status)]);
})->middleware('guest')->name('password.email');
```

Przed przejściem dalej przeanalizujmy tę trasę bardziej szczegółowo. Najpierw walidowany jest atrybut `email` żądania. Następnie użyjemy wbudowanego "brokera hasła" Laravel (za pośrednictwem fasady `Password`), aby wysłać link resetowania hasła do użytkownika. Broker hasła zajmie się pobraniem użytkownika po danym polu (w tym przypadku adresie e-mail) i wysłaniem użytkownikowi linku resetowania hasła za pomocą wbudowanego [systemu powiadomień](/docs/{{version}}/notifications) Laravel.

Metoda `sendResetLink` zwraca slug "statusu". Ten status może być przetłumaczony za pomocą helperów [lokalizacji](/docs/{{version}}/localization) Laravel w celu wyświetlenia przyjaznej dla użytkownika wiadomości dotyczącej statusu ich żądania. Tłumaczenie statusu resetowania hasła jest określane przez plik językowy `lang/{lang}/passwords.php` Twojej aplikacji. Wpis dla każdej możliwej wartości slugu statusu znajduje się w pliku językowym `passwords`.

> [!NOTE]
> Domyślnie szkielet aplikacji Laravel nie zawiera katalogu `lang`. Jeśli chcesz dostosować pliki językowe Laravel, możesz je opublikować za pomocą polecenia Artisan `lang:publish`.

Możesz się zastanawiać, skąd Laravel wie, jak pobrać rekord użytkownika z bazy danych Twojej aplikacji podczas wywoływania metody `sendResetLink` fasady `Password`. Broker hasła Laravel wykorzystuje "dostawców użytkowników" Twojego systemu uwierzytelniania do pobierania rekordów z bazy danych. Dostawca użytkowników używany przez broker hasła jest skonfigurowany w tablicy konfiguracyjnej `passwords` pliku konfiguracyjnego `config/auth.php`. Aby dowiedzieć się więcej o pisaniu niestandardowych dostawców użytkowników, zapoznaj się z [dokumentacją uwierzytelniania](/docs/{{version}}/authentication#adding-custom-user-providers).

> [!NOTE]
> Podczas ręcznego implementowania resetowania haseł musisz sam zdefiniować zawartość widoków i tras. Jeśli chcesz szkielet zawierający całą niezbędną logikę uwierzytelniania i weryfikacji, sprawdź [zestawy startowe aplikacji Laravel](/docs/{{version}}/starter-kits).

<a name="resetting-the-password"></a>
### Resetowanie hasła

<a name="the-password-reset-form"></a>
#### Formularz resetowania hasła

Następnie zdefiniujemy trasy niezbędne do faktycznego zresetowania hasła, gdy użytkownik kliknie link resetowania hasła, który został do niego wysłany e-mailem i poda nowe hasło. Najpierw zdefiniujmy trasę, która będzie wyświetlać formularz resetowania hasła, który jest wyświetlany, gdy użytkownik kliknie link resetowania hasła. Ta trasa otrzyma parametr `token`, którego użyjemy później do weryfikacji żądania resetowania hasła:

```php
Route::get('/reset-password/{token}', function (string $token) {
    return view('auth.reset-password', ['token' => $token]);
})->middleware('guest')->name('password.reset');
```

Widok zwracany przez tę trasę powinien wyświetlać formularz zawierający pole `email`, pole `password`, pole `password_confirmation` oraz ukryte pole `token`, które powinno zawierać wartość tajnego `$token` otrzymanego przez naszą trasę.

<a name="password-reset-handling-the-form-submission"></a>
#### Obsługa przesłania formularza

Oczywiście musimy zdefiniować trasę do faktycznej obsługi przesłania formularza resetowania hasła. Ta trasa będzie odpowiedzialna za walidację przychodzącego żądania i aktualizację hasła użytkownika w bazie danych:

```php
use App\Models\User;
use Illuminate\Auth\Events\PasswordReset;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Password;
use Illuminate\Support\Str;

Route::post('/reset-password', function (Request $request) {
    $request->validate([
        'token' => 'required',
        'email' => 'required|email',
        'password' => 'required|min:8|confirmed',
    ]);

    $status = Password::reset(
        $request->only('email', 'password', 'password_confirmation', 'token'),
        function (User $user, string $password) {
            $user->forceFill([
                'password' => Hash::make($password)
            ])->setRememberToken(Str::random(60));

            $user->save();

            event(new PasswordReset($user));
        }
    );

    return $status === Password::PasswordReset
        ? redirect()->route('login')->with('status', __($status))
        : back()->withErrors(['email' => [__($status)]]);
})->middleware('guest')->name('password.update');
```

Przed przejściem dalej przeanalizujmy tę trasę bardziej szczegółowo. Najpierw walidowane są atrybuty `token`, `email` i `password` żądania. Następnie użyjemy wbudowanego "brokera hasła" Laravel (za pośrednictwem fasady `Password`) do walidacji danych uwierzytelniających żądania resetowania hasła.

Jeśli token, adres e-mail i hasło przekazane brokerowi hasła są prawidłowe, wywołane zostanie zamknięcie przekazane do metody `reset`. W ramach tego zamknięcia, które otrzymuje instancję użytkownika i hasło w postaci zwykłego tekstu dostarczone do formularza resetowania hasła, możemy zaktualizować hasło użytkownika w bazie danych.

Metoda `reset` zwraca slug "statusu". Ten status może być przetłumaczony za pomocą helperów [lokalizacji](/docs/{{version}}/localization) Laravel w celu wyświetlenia przyjaznej dla użytkownika wiadomości dotyczącej statusu ich żądania. Tłumaczenie statusu resetowania hasła jest określane przez plik językowy `lang/{lang}/passwords.php` Twojej aplikacji. Wpis dla każdej możliwej wartości slugu statusu znajduje się w pliku językowym `passwords`. Jeśli Twoja aplikacja nie zawiera katalogu `lang`, możesz go utworzyć za pomocą polecenia Artisan `lang:publish`.

Przed przejściem dalej możesz się zastanawiać, skąd Laravel wie, jak pobrać rekord użytkownika z bazy danych Twojej aplikacji podczas wywoływania metody `reset` fasady `Password`. Broker hasła Laravel wykorzystuje "dostawców użytkowników" Twojego systemu uwierzytelniania do pobierania rekordów z bazy danych. Dostawca użytkowników używany przez broker hasła jest skonfigurowany w tablicy konfiguracyjnej `passwords` pliku konfiguracyjnego `config/auth.php`. Aby dowiedzieć się więcej o pisaniu niestandardowych dostawców użytkowników, zapoznaj się z [dokumentacją uwierzytelniania](/docs/{{version}}/authentication#adding-custom-user-providers).

<a name="deleting-expired-tokens"></a>
## Usuwanie wygasłych tokenów

Jeśli używasz sterownika `database`, tokeny resetowania hasła, które wygasły, nadal będą obecne w Twojej bazie danych. Jednak możesz łatwo usunąć te rekordy za pomocą polecenia Artisan `auth:clear-resets`:

```shell
php artisan auth:clear-resets
```

Jeśli chcesz zautomatyzować ten proces, rozważ dodanie polecenia do [harmonogramu](/docs/{{version}}/scheduling) Twojej aplikacji:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('auth:clear-resets')->everyFifteenMinutes();
```

<a name="password-customization"></a>
## Dostosowywanie

<a name="reset-link-customization"></a>
#### Dostosowywanie linku resetowania

Możesz dostosować URL linku resetowania hasła za pomocą metody `createUrlUsing` dostarczonej przez klasę powiadomienia `ResetPassword`. Ta metoda akceptuje zamknięcie, które otrzymuje instancję użytkownika otrzymującego powiadomienie oraz token linku resetowania hasła. Zazwyczaj powinieneś wywołać tę metodę z metody `boot` `AppServiceProvider` Twojej aplikacji:

```php
use App\Models\User;
use Illuminate\Auth\Notifications\ResetPassword;

/**
 * Zainicjalizuj dowolne usługi aplikacji.
 */
public function boot(): void
{
    ResetPassword::createUrlUsing(function (User $user, string $token) {
        return 'https://example.com/reset-password?token='.$token;
    });
}
```

<a name="reset-email-customization"></a>
#### Dostosowywanie e-maila resetowania

Możesz łatwo zmodyfikować klasę powiadomienia używaną do wysłania linku resetowania hasła do użytkownika. Aby rozpocząć, nadpisz metodę `sendPasswordResetNotification` w swoim modelu `App\Models\User`. W ramach tej metody możesz wysłać powiadomienie za pomocą dowolnej [klasy powiadomienia](/docs/{{version}}/notifications) własnego autorstwa. Token resetowania hasła `$token` jest pierwszym argumentem otrzymywanym przez metodę. Możesz użyć tego `$token`, aby zbudować URL resetowania hasła według własnego uznania i wysłać swoje powiadomienie do użytkownika:

```php
use App\Notifications\ResetPasswordNotification;

/**
 * Wyślij powiadomienie o resetowaniu hasła do użytkownika.
 *
 * @param  string  $token
 */
public function sendPasswordResetNotification($token): void
{
    $url = 'https://example.com/reset-password?token='.$token;

    $this->notify(new ResetPasswordNotification($url));
}
```
