# Laravel Fortify

- [Wprowadzenie](#introduction)
    - [Czym jest Fortify?](#what-is-fortify)
    - [Kiedy powinienem używać Fortify?](#when-should-i-use-fortify)
- [Instalacja](#installation)
    - [Funkcje Fortify](#fortify-features)
    - [Wyłączanie widoków](#disabling-views)
- [Uwierzytelnianie](#authentication)
    - [Dostosowywanie uwierzytelniania użytkownika](#customizing-user-authentication)
    - [Dostosowywanie pipeline'u uwierzytelniania](#customizing-the-authentication-pipeline)
    - [Dostosowywanie przekierowań](#customizing-authentication-redirects)
- [Uwierzytelnianie dwuskładnikowe](#two-factor-authentication)
    - [Włączanie uwierzytelniania dwuskładnikowego](#enabling-two-factor-authentication)
    - [Uwierzytelnianie z użyciem uwierzytelniania dwuskładnikowego](#authenticating-with-two-factor-authentication)
    - [Wyłączanie uwierzytelniania dwuskładnikowego](#disabling-two-factor-authentication)
- [Rejestracja](#registration)
    - [Dostosowywanie rejestracji](#customizing-registration)
- [Resetowanie hasła](#password-reset)
    - [Żądanie linku resetowania hasła](#requesting-a-password-reset-link)
    - [Resetowanie hasła](#resetting-the-password)
    - [Dostosowywanie resetowania hasła](#customizing-password-resets)
- [Weryfikacja e-mail](#email-verification)
    - [Ochrona tras](#protecting-routes)
- [Potwierdzanie hasła](#password-confirmation)

<a name="introduction"></a>
## Wprowadzenie

[Laravel Fortify](https://github.com/laravel/fortify) to niezależna od frontendu implementacja backendu uwierzytelniania dla Laravel. Fortify rejestruje trasy i kontrolery potrzebne do implementacji wszystkich funkcji uwierzytelniania Laravel, w tym logowania, rejestracji, resetowania hasła, weryfikacji e-mail i innych. Po zainstalowaniu Fortify możesz uruchomić polecenie Artisan `route:list`, aby zobaczyć trasy zarejestrowane przez Fortify.

Ponieważ Fortify nie dostarcza własnego interfejsu użytkownika, jest przeznaczony do współpracy z Twoim własnym interfejsem użytkownika, który wysyła żądania do rejestrowanych przez niego tras. Omówimy dokładnie, jak wysyłać żądania do tych tras w dalszej części tej dokumentacji.

> [!NOTE]
> Pamiętaj, że Fortify to pakiet, który ma dać Ci dobry start w implementacji funkcji uwierzytelniania Laravel. **Nie musisz go używać.** Zawsze możesz ręcznie korzystać z usług uwierzytelniania Laravel, postępując zgodnie z dokumentacją dostępną w sekcjach [uwierzytelnianie](/docs/{{version}}/authentication), [resetowanie hasła](/docs/{{version}}/passwords) i [weryfikacja e-mail](/docs/{{version}}/verification).

<a name="what-is-fortify"></a>
### Czym jest Fortify?

Jak wspomniano wcześniej, Laravel Fortify to niezależna od frontendu implementacja backendu uwierzytelniania dla Laravel. Fortify rejestruje trasy i kontrolery potrzebne do implementacji wszystkich funkcji uwierzytelniania Laravel, w tym logowania, rejestracji, resetowania hasła, weryfikacji e-mail i innych.

**Nie musisz używać Fortify, aby korzystać z funkcji uwierzytelniania Laravel.** Zawsze możesz ręcznie korzystać z usług uwierzytelniania Laravel, postępując zgodnie z dokumentacją dostępną w sekcjach [uwierzytelnianie](/docs/{{version}}/authentication), [resetowanie hasła](/docs/{{version}}/passwords) i [weryfikacja e-mail](/docs/{{version}}/verification).

Jeśli jesteś nowy w Laravel, możesz chcieć zapoznać się z [naszymi zestawami startowymi aplikacji](/docs/{{version}}/starter-kits). Zestawy startowe aplikacji Laravel używają Fortify wewnętrznie, aby zapewnić szkielet uwierzytelniania dla Twojej aplikacji, który zawiera interfejs użytkownika zbudowany za pomocą [Tailwind CSS](https://tailwindcss.com). Pozwala to na naukę i oswojenie się z funkcjami uwierzytelniania Laravel.

Laravel Fortify zasadniczo pobiera trasy i kontrolery z naszych zestawów startowych aplikacji i oferuje je jako pakiet, który nie zawiera interfejsu użytkownika. Pozwala to nadal szybko zbudować implementację backendu warstwy uwierzytelniania aplikacji bez bycia związanym z konkretnymi opiniami dotyczącymi frontendu.

<a name="when-should-i-use-fortify"></a>
### Kiedy powinienem używać Fortify?

Możesz się zastanawiać, kiedy właściwe jest użycie Laravel Fortify. Po pierwsze, jeśli używasz jednego z [zestawów startowych aplikacji](/docs/{{version}}/starter-kits) Laravel, nie musisz instalować Laravel Fortify, ponieważ wszystkie zestawy startowe aplikacji Laravel używają Fortify i już zapewniają pełną implementację uwierzytelniania.

Jeśli nie używasz zestawu startowego aplikacji, a Twoja aplikacja potrzebuje funkcji uwierzytelniania, masz dwie opcje: ręcznie zaimplementować funkcje uwierzytelniania swojej aplikacji lub użyć Laravel Fortify do zapewnienia implementacji backendu tych funkcji.

Jeśli zdecydujesz się zainstalować Fortify, Twój interfejs użytkownika będzie wysyłał żądania do tras uwierzytelniania Fortify, które są szczegółowo opisane w tej dokumentacji, w celu uwierzytelniania i rejestrowania użytkowników.

Jeśli zdecydujesz się ręcznie korzystać z usług uwierzytelniania Laravel zamiast używać Fortify, możesz to zrobić, postępując zgodnie z dokumentacją dostępną w sekcjach [uwierzytelnianie](/docs/{{version}}/authentication), [resetowanie hasła](/docs/{{version}}/passwords) i [weryfikacja e-mail](/docs/{{version}}/verification).

<a name="laravel-fortify-and-laravel-sanctum"></a>
#### Laravel Fortify i Laravel Sanctum

Niektórzy deweloperzy mylą się co do różnicy między [Laravel Sanctum](/docs/{{version}}/sanctum) a Laravel Fortify. Ponieważ te dwa pakiety rozwiązują dwa różne, ale powiązane problemy, Laravel Fortify i Laravel Sanctum nie są wzajemnie wykluczające się ani konkurencyjne pakiety.

Laravel Sanctum zajmuje się tylko zarządzaniem tokenami API i uwierzytelnianiem istniejących użytkowników za pomocą ciasteczek sesji lub tokenów. Sanctum nie dostarcza żadnych tras obsługujących rejestrację użytkowników, resetowanie hasła itp.

Jeśli próbujesz ręcznie zbudować warstwę uwierzytelniania dla aplikacji, która oferuje API lub służy jako backend dla aplikacji jednostronicowej, całkowicie możliwe jest, że będziesz używać zarówno Laravel Fortify (do rejestracji użytkowników, resetowania hasła itp.) jak i Laravel Sanctum (zarządzanie tokenami API, uwierzytelnianie sesji).

<a name="installation"></a>
## Instalacja

Aby rozpocząć, zainstaluj Fortify za pomocą menedżera pakietów Composer:

```shell
composer require laravel/fortify
```

Następnie opublikuj zasoby Fortify za pomocą polecenia Artisan `fortify:install`:

```shell
php artisan fortify:install
```

To polecenie opublikuje akcje Fortify do katalogu `app/Actions`, który zostanie utworzony, jeśli nie istnieje. Dodatkowo zostanie opublikowany `FortifyServiceProvider`, plik konfiguracyjny i wszystkie niezbędne migracje bazy danych.

Następnie powinieneś przeprowadzić migrację bazy danych:

```shell
php artisan migrate
```

<a name="fortify-features"></a>
### Funkcje Fortify

Plik konfiguracyjny `fortify` zawiera tablicę konfiguracyjną `features`. Ta tablica definiuje, które trasy backendu / funkcje Fortify będzie domyślnie udostępniać. Zalecamy włączenie tylko następujących funkcji, które są podstawowymi funkcjami uwierzytelniania zapewnianymi przez większość aplikacji Laravel:

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
],
```

<a name="disabling-views"></a>
### Wyłączanie widoków

Domyślnie Fortify definiuje trasy, które mają zwracać widoki, takie jak ekran logowania lub ekran rejestracji. Jeśli jednak budujesz aplikację jednostronicową sterowaną przez JavaScript, możesz nie potrzebować tych tras. Z tego powodu możesz całkowicie wyłączyć te trasy, ustawiając wartość konfiguracji `views` w pliku konfiguracyjnym `config/fortify.php` Twojej aplikacji na `false`:

```php
'views' => false,
```

<a name="disabling-views-and-password-reset"></a>
#### Wyłączanie widoków i resetowanie hasła

Jeśli zdecydujesz się wyłączyć widoki Fortify i będziesz implementować funkcje resetowania hasła dla swojej aplikacji, nadal powinieneś zdefiniować trasę o nazwie `password.reset`, która jest odpowiedzialna za wyświetlanie widoku "resetuj hasło" Twojej aplikacji. Jest to konieczne, ponieważ powiadomienie `Illuminate\Auth\Notifications\ResetPassword` Laravel wygeneruje URL resetowania hasła za pomocą nazwanej trasy `password.reset`.

<a name="authentication"></a>
## Uwierzytelnianie

Aby rozpocząć, musimy poinstruować Fortify, jak zwrócić nasz widok "logowania". Pamiętaj, że Fortify to biblioteka uwierzytelniania bez interfejsu użytkownika. Jeśli chcesz mieć gotową implementację frontendową funkcji uwierzytelniania Laravel, powinieneś użyć [zestawu startowego aplikacji](/docs/{{version}}/starter-kits).

Wszystkie logiki renderowania widoków uwierzytelniania mogą być dostosowane za pomocą odpowiednich metod dostępnych przez klasę `Laravel\Fortify\Fortify`. Zazwyczaj powinieneś wywołać tę metodę z metody `boot` klasy `App\Providers\FortifyServiceProvider` swojej aplikacji. Fortify zajmie się zdefiniowaniem trasy `/login`, która zwraca ten widok:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::loginView(function () {
        return view('auth.login');
    });

    // ...
}
```

Twój szablon logowania powinien zawierać formularz, który wysyła żądanie POST do `/login`. Endpoint `/login` oczekuje ciągu `email` / `username` i `password`. Nazwa pola email / username powinna odpowiadać wartości `username` w pliku konfiguracyjnym `config/fortify.php`. Dodatkowo może być dostarczony boolean `remember`, aby wskazać, że użytkownik chce używać funkcjonalności "zapamiętaj mnie" zapewnionej przez Laravel.

Jeśli próba logowania zakończy się sukcesem, Fortify przekieruje Cię do URI skonfigurowanego przez opcję konfiguracji `home` w pliku konfiguracyjnym `fortify` Twojej aplikacji. Jeśli żądanie logowania było żądaniem XHR, zwrócona zostanie odpowiedź HTTP 200.

Jeśli żądanie nie powiodło się, użytkownik zostanie przekierowany z powrotem na ekran logowania, a błędy walidacji będą dostępne przez współdzieloną [zmienną szablonu Blade](/docs/{{version}}/validation#quick-displaying-the-validation-errors) `$errors`. W przypadku żądania XHR błędy walidacji zostaną zwrócone z odpowiedzią HTTP 422.

<a name="customizing-user-authentication"></a>
### Dostosowywanie uwierzytelniania użytkownika

Fortify automatycznie pobierze i uwierzytelni użytkownika na podstawie dostarczonych poświadczeń i straży uwierzytelniania skonfigurowanej dla Twojej aplikacji. Możesz jednak czasami chcieć mieć pełną kontrolę nad tym, jak poświadczenia logowania są uwierzytelniane i użytkownicy są pobierani. Na szczęście Fortify pozwala łatwo to osiągnąć za pomocą metody `Fortify::authenticateUsing`.

Ta metoda akceptuje closure, które otrzymuje przychodzące żądanie HTTP. Closure jest odpowiedzialne za walidację poświadczeń logowania dołączonych do żądania i zwrócenie powiązanej instancji użytkownika. Jeśli poświadczenia są nieprawidłowe lub nie można znaleźć użytkownika, `null` lub `false` powinno zostać zwrócone przez closure. Zazwyczaj ta metoda powinna być wywołana z metody `boot` Twojego `FortifyServiceProvider`:

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::authenticateUsing(function (Request $request) {
        $user = User::where('email', $request->email)->first();

        if ($user &&
            Hash::check($request->password, $user->password)) {
            return $user;
        }
    });

    // ...
}
```

<a name="authentication-guard"></a>
#### Straż uwierzytelniania

Możesz dostosować straż uwierzytelniania używaną przez Fortify w pliku konfiguracyjnym `fortify` swojej aplikacji. Powinieneś jednak upewnić się, że skonfigurowana straż jest implementacją `Illuminate\Contracts\Auth\StatefulGuard`. Jeśli próbujesz użyć Laravel Fortify do uwierzytelnienia SPA, powinieneś użyć domyślnej straży `web` Laravel w połączeniu z [Laravel Sanctum](https://laravel.com/docs/sanctum).

<a name="customizing-the-authentication-pipeline"></a>
### Dostosowywanie pipeline'u uwierzytelniania

Laravel Fortify uwierzytelnia żądania logowania poprzez pipeline wywoływalnych klas. Jeśli chcesz, możesz zdefiniować własny pipeline klas, przez który powinny być przekazywane żądania logowania. Każda klasa powinna mieć metodę `__invoke`, która otrzymuje przychodzącą instancję `Illuminate\Http\Request` i, podobnie jak [middleware](/docs/{{version}}/middleware), zmienną `$next`, która jest wywoływana w celu przekazania żądania do następnej klasy w pipeline.

Aby zdefiniować własny pipeline, możesz użyć metody `Fortify::authenticateThrough`. Ta metoda akceptuje closure, które powinno zwrócić tablicę klas, przez które ma być przekazane żądanie logowania. Zazwyczaj ta metoda powinna być wywołana z metody `boot` klasy `App\Providers\FortifyServiceProvider`.

Poniższy przykład zawiera domyślną definicję pipeline, której możesz użyć jako punktu wyjścia podczas dokonywania własnych modyfikacji:

```php
use Laravel\Fortify\Actions\AttemptToAuthenticate;
use Laravel\Fortify\Actions\CanonicalizeUsername;
use Laravel\Fortify\Actions\EnsureLoginIsNotThrottled;
use Laravel\Fortify\Actions\PrepareAuthenticatedSession;
use Laravel\Fortify\Actions\RedirectIfTwoFactorAuthenticatable;
use Laravel\Fortify\Features;
use Laravel\Fortify\Fortify;
use Illuminate\Http\Request;

Fortify::authenticateThrough(function (Request $request) {
    return array_filter([
            config('fortify.limiters.login') ? null : EnsureLoginIsNotThrottled::class,
            config('fortify.lowercase_usernames') ? CanonicalizeUsername::class : null,
            Features::enabled(Features::twoFactorAuthentication()) ? RedirectIfTwoFactorAuthenticatable::class : null,
            AttemptToAuthenticate::class,
            PrepareAuthenticatedSession::class,
    ]);
});
```

#### Throttling uwierzytelniania

Domyślnie Fortify będzie throttlować próby uwierzytelnienia za pomocą middleware `EnsureLoginIsNotThrottled`. To middleware throttluje próby, które są unikalne dla kombinacji nazwy użytkownika i adresu IP.

Niektóre aplikacje mogą wymagać innego podejścia do throttlingu prób uwierzytelnienia, takiego jak throttling tylko według adresu IP. Dlatego Fortify pozwala określić własny [rate limiter](/docs/{{version}}/routing#rate-limiting) poprzez opcję konfiguracji `fortify.limiters.login`. Oczywiście ta opcja konfiguracji znajduje się w pliku konfiguracyjnym `config/fortify.php` Twojej aplikacji.

> [!NOTE]
> Używanie kombinacji throttlingu, [uwierzytelniania dwuskładnikowego](/docs/{{version}}/fortify#two-factor-authentication) i zewnętrznej zapory aplikacji webowej (WAF) zapewni najbardziej solidną obronę dla prawdziwych użytkowników Twojej aplikacji.

<a name="customizing-authentication-redirects"></a>
### Dostosowywanie przekierowań

Jeśli próba logowania zakończy się sukcesem, Fortify przekieruje Cię do URI skonfigurowanego przez opcję konfiguracji `home` w pliku konfiguracyjnym `fortify` Twojej aplikacji. Jeśli żądanie logowania było żądaniem XHR, zwrócona zostanie odpowiedź HTTP 200. Po wylogowaniu się użytkownika z aplikacji, użytkownik zostanie przekierowany do URI `/`.

Jeśli potrzebujesz zaawansowanego dostosowania tego zachowania, możesz związać implementacje kontraktów `LoginResponse` i `LogoutResponse` w [kontenerze usług](/docs/{{version}}/container) Laravel. Zazwyczaj powinno to zostać zrobione w metodzie `register` klasy `App\Providers\FortifyServiceProvider` Twojej aplikacji:

```php
use Laravel\Fortify\Contracts\LogoutResponse;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->instance(LogoutResponse::class, new class implements LogoutResponse {
        public function toResponse($request)
        {
            return redirect('/');
        }
    });
}
```

<a name="two-factor-authentication"></a>
## Uwierzytelnianie dwuskładnikowe

Gdy funkcja uwierzytelniania dwuskładnikowego Fortify jest włączona, użytkownik musi wprowadzić sześcio cyfrowy token numeryczny podczas procesu uwierzytelniania. Ten token jest generowany przy użyciu jednorazowego hasła opartego na czasie (TOTP), które można pobrać z dowolnej mobilnej aplikacji uwierzytelniającej zgodnej z TOTP, takiej jak Google Authenticator.

Przed rozpoczęciem powinieneś się upewnić, że model `App\Models\User` Twojej aplikacji używa traitu `Laravel\Fortify\TwoFactorAuthenticatable`:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Fortify\TwoFactorAuthenticatable;

class User extends Authenticatable
{
    use Notifiable, TwoFactorAuthenticatable;
}
```

Następnie powinieneś zbudować ekran w swojej aplikacji, na którym użytkownicy mogą zarządzać swoimi ustawieniami uwierzytelniania dwuskładnikowego. Ten ekran powinien pozwalać użytkownikowi włączać i wyłączać uwierzytelnianie dwuskładnikowe, a także regenerować kody odzyskiwania uwierzytelniania dwuskładnikowego.

> Domyślnie tablica `features` pliku konfiguracyjnego `fortify` nakazuje, aby ustawienia uwierzytelniania dwuskładnikowego Fortify wymagały potwierdzenia hasła przed modyfikacją. Dlatego Twoja aplikacja powinna zaimplementować funkcję [potwierdzania hasła](#password-confirmation) Fortify przed kontynuowaniem.

<a name="enabling-two-factor-authentication"></a>
### Włączanie uwierzytelniania dwuskładnikowego

Aby rozpocząć włączanie uwierzytelniania dwuskładnikowego, Twoja aplikacja powinna wysłać żądanie POST do endpointu `/user/two-factor-authentication` zdefiniowanego przez Fortify. Jeśli żądanie zakończy się sukcesem, użytkownik zostanie przekierowany z powrotem na poprzedni URL, a zmienna sesji `status` zostanie ustawiona na `two-factor-authentication-enabled`. Możesz wykryć tę zmienną sesji `status` w swoich szablonach, aby wyświetlić odpowiedni komunikat o sukcesie. Jeśli żądanie było żądaniem XHR, zwrócona zostanie odpowiedź HTTP `200`.

Po wybraniu włączenia uwierzytelniania dwuskładnikowego, użytkownik musi nadal "potwierdzić" swoją konfigurację uwierzytelniania dwuskładnikowego, dostarczając prawidłowy kod uwierzytelniania dwuskładnikowego. Tak więc Twój komunikat "sukcesu" powinien informować użytkownika, że potwierdzenie uwierzytelniania dwuskładnikowego jest nadal wymagane:

```html
@if (session('status') == 'two-factor-authentication-enabled')
    <div class="mb-4 font-medium text-sm">
        Proszę zakończyć konfigurację uwierzytelniania dwuskładnikowego poniżej.
    </div>
@endif
```

Następnie powinieneś wyświetlić kod QR uwierzytelniania dwuskładnikowego, aby użytkownik mógł go zeskanować do swojej aplikacji uwierzytelniającej. Jeśli używasz Blade do renderowania frontendu swojej aplikacji, możesz pobrać SVG kodu QR za pomocą metody `twoFactorQrCodeSvg` dostępnej na instancji użytkownika:

```php
$request->user()->twoFactorQrCodeSvg();
```

Jeśli budujesz frontend zasilany przez JavaScript, możesz wykonać żądanie XHR GET do endpointu `/user/two-factor-qr-code`, aby pobrać kod QR uwierzytelniania dwuskładnikowego użytkownika. Ten endpoint zwróci obiekt JSON zawierający klucz `svg`.

<a name="confirming-two-factor-authentication"></a>
#### Potwierdzanie uwierzytelniania dwuskładnikowego

Oprócz wyświetlania kodu QR uwierzytelniania dwuskładnikowego użytkownika, powinieneś udostępnić pole tekstowe, w którym użytkownik może podać prawidłowy kod uwierzytelniający, aby "potwierdzić" swoją konfigurację uwierzytelniania dwuskładnikowego. Ten kod powinien zostać dostarczony do aplikacji Laravel poprzez żądanie POST do endpointu `/user/confirmed-two-factor-authentication` zdefiniowanego przez Fortify.

Jeśli żądanie zakończy się sukcesem, użytkownik zostanie przekierowany z powrotem na poprzedni URL, a zmienna sesji `status` zostanie ustawiona na `two-factor-authentication-confirmed`:

```html
@if (session('status') == 'two-factor-authentication-confirmed')
    <div class="mb-4 font-medium text-sm">
        Uwierzytelnianie dwuskładnikowe potwierdzone i włączone pomyślnie.
    </div>
@endif
```

Jeśli żądanie do endpointu potwierdzenia uwierzytelniania dwuskładnikowego zostało wykonane przez żądanie XHR, zwrócona zostanie odpowiedź HTTP `200`.

<a name="displaying-the-recovery-codes"></a>
#### Wyświetlanie kodów odzyskiwania

Powinieneś również wyświetlić kody odzyskiwania uwierzytelniania dwuskładnikowego użytkownika. Te kody odzyskiwania pozwalają użytkownikowi uwierzytelnić się, jeśli straci dostęp do swojego urządzenia mobilnego. Jeśli używasz Blade do renderowania frontendu swojej aplikacji, możesz uzyskać dostęp do kodów odzyskiwania przez instancję uwierzytelnionego użytkownika:

```php
(array) $request->user()->recoveryCodes()
```

Jeśli budujesz frontend zasilany przez JavaScript, możesz wykonać żądanie XHR GET do endpointu `/user/two-factor-recovery-codes`. Ten endpoint zwróci tablicę JSON zawierającą kody odzyskiwania użytkownika.

Aby zregenerować kody odzyskiwania użytkownika, Twoja aplikacja powinna wysłać żądanie POST do endpointu `/user/two-factor-recovery-codes`.

<a name="authenticating-with-two-factor-authentication"></a>
### Uwierzytelnianie z użyciem uwierzytelniania dwuskładnikowego

Podczas procesu uwierzytelniania Fortify automatycznie przekieruje użytkownika na ekran wyzwania uwierzytelniania dwuskładnikowego Twojej aplikacji. Jeśli jednak Twoja aplikacja wykonuje żądanie logowania XHR, odpowiedź JSON zwrócona po pomyślnej próbie uwierzytelnienia będzie zawierać obiekt JSON, który ma właściwość boolean `two_factor`. Powinieneś sprawdzić tę wartość, aby wiedzieć, czy powinieneś przekierować na ekran wyzwania uwierzytelniania dwuskładnikowego swojej aplikacji.

Aby rozpocząć implementację funkcjonalności uwierzytelniania dwuskładnikowego, musimy poinstruować Fortify, jak zwrócić nasz widok wyzwania uwierzytelniania dwuskładnikowego. Cała logika renderowania widoków uwierzytelniania Fortify może być dostosowana za pomocą odpowiednich metod dostępnych przez klasę `Laravel\Fortify\Fortify`. Zazwyczaj powinieneś wywołać tę metodę z metody `boot` klasy `App\Providers\FortifyServiceProvider` swojej aplikacji:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::twoFactorChallengeView(function () {
        return view('auth.two-factor-challenge');
    });

    // ...
}
```

Fortify zajmie się zdefiniowaniem trasy `/two-factor-challenge`, która zwraca ten widok. Twój szablon `two-factor-challenge` powinien zawierać formularz, który wysyła żądanie POST do endpointu `/two-factor-challenge`. Akcja `/two-factor-challenge` oczekuje pola `code` zawierającego prawidłowy token TOTP lub pola `recovery_code` zawierającego jeden z kodów odzyskiwania użytkownika.

Jeśli próba logowania zakończy się sukcesem, Fortify przekieruje użytkownika do URI skonfigurowanego przez opcję konfiguracji `home` w pliku konfiguracyjnym `fortify` Twojej aplikacji. Jeśli żądanie logowania było żądaniem XHR, zwrócona zostanie odpowiedź HTTP 204.

Jeśli żądanie nie powiodło się, użytkownik zostanie przekierowany z powrotem na ekran wyzwania dwuskładnikowego, a błędy walidacji będą dostępne przez współdzieloną [zmienną szablonu Blade](/docs/{{version}}/validation#quick-displaying-the-validation-errors) `$errors`. W przypadku żądania XHR błędy walidacji zostaną zwrócone z odpowiedzią HTTP 422.

<a name="disabling-two-factor-authentication"></a>
### Wyłączanie uwierzytelniania dwuskładnikowego

Aby wyłączyć uwierzytelnianie dwuskładnikowe, Twoja aplikacja powinna wysłać żądanie DELETE do endpointu `/user/two-factor-authentication`. Pamiętaj, że endpointy uwierzytelniania dwuskładnikowego Fortify wymagają [potwierdzenia hasła](#password-confirmation) przed wywołaniem.

<a name="registration"></a>
## Rejestracja

Aby rozpocząć implementację funkcjonalności rejestracji naszej aplikacji, musimy poinstruować Fortify, jak zwrócić nasz widok "rejestracji". Pamiętaj, że Fortify to biblioteka uwierzytelniania bez interfejsu użytkownika. Jeśli chcesz mieć gotową implementację frontendową funkcji uwierzytelniania Laravel, powinieneś użyć [zestawu startowego aplikacji](/docs/{{version}}/starter-kits).

Wszystkie logiki renderowania widoków Fortify mogą być dostosowane za pomocą odpowiednich metod dostępnych przez klasę `Laravel\Fortify\Fortify`. Zazwyczaj powinieneś wywołać tę metodę z metody `boot` klasy `App\Providers\FortifyServiceProvider`:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::registerView(function () {
        return view('auth.register');
    });

    // ...
}
```

Fortify zajmie się zdefiniowaniem trasy `/register`, która zwraca ten widok. Twój szablon `register` powinien zawierać formularz, który wysyła żądanie POST do endpointu `/register` zdefiniowanego przez Fortify.

Endpoint `/register` oczekuje ciągu `name`, adresu email / nazwy użytkownika typu string, pól `password` i `password_confirmation`. Nazwa pola email / nazwy użytkownika powinna odpowiadać wartości konfiguracji `username` zdefiniowanej w pliku konfiguracyjnym `fortify` Twojej aplikacji.

Jeśli próba rejestracji zakończy się sukcesem, Fortify przekieruje użytkownika do URI skonfigurowanego przez opcję konfiguracji `home` w pliku konfiguracyjnym `fortify` Twojej aplikacji. Jeśli żądanie było żądaniem XHR, zwrócona zostanie odpowiedź HTTP 201.

Jeśli żądanie nie powiodło się, użytkownik zostanie przekierowany z powrotem na ekran rejestracji, a błędy walidacji będą dostępne przez współdzieloną [zmienną szablonu Blade](/docs/{{version}}/validation#quick-displaying-the-validation-errors) `$errors`. W przypadku żądania XHR błędy walidacji zostaną zwrócone z odpowiedzią HTTP 422.

<a name="customizing-registration"></a>
### Dostosowywanie rejestracji

Proces walidacji i tworzenia użytkownika może być dostosowany poprzez modyfikację akcji `App\Actions\Fortify\CreateNewUser`, która została wygenerowana podczas instalacji Laravel Fortify.

<a name="password-reset"></a>
## Resetowanie hasła

<a name="requesting-a-password-reset-link"></a>
### Żądanie linku resetowania hasła

Aby rozpocząć implementację funkcjonalności resetowania hasła naszej aplikacji, musimy poinstruować Fortify, jak zwrócić nasz widok "zapomnianego hasła". Pamiętaj, że Fortify to biblioteka uwierzytelniania bez interfejsu użytkownika. Jeśli chcesz mieć gotową implementację frontendową funkcji uwierzytelniania Laravel, powinieneś użyć [zestawu startowego aplikacji](/docs/{{version}}/starter-kits).

Wszystkie logiki renderowania widoków Fortify mogą być dostosowane za pomocą odpowiednich metod dostępnych przez klasę `Laravel\Fortify\Fortify`. Zazwyczaj powinieneś wywołać tę metodę z metody `boot` klasy `App\Providers\FortifyServiceProvider` swojej aplikacji:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::requestPasswordResetLinkView(function () {
        return view('auth.forgot-password');
    });

    // ...
}
```

Fortify zajmie się zdefiniowaniem endpointu `/forgot-password`, który zwraca ten widok. Twój szablon `forgot-password` powinien zawierać formularz, który wysyła żądanie POST do endpointu `/forgot-password`.

Endpoint `/forgot-password` oczekuje pola `email` typu string. Nazwa tego pola / kolumny bazy danych powinna odpowiadać wartości konfiguracji `email` w pliku konfiguracyjnym `fortify` Twojej aplikacji.

<a name="handling-the-password-reset-link-request-response"></a>
#### Obsługa odpowiedzi żądania linku resetowania hasła

Jeśli żądanie linku resetowania hasła było pomyślne, Fortify przekieruje użytkownika z powrotem do endpointu `/forgot-password` i wyśle email do użytkownika z bezpiecznym linkiem, którego może użyć do zresetowania swojego hasła. Jeśli żądanie było żądaniem XHR, zwrócona zostanie odpowiedź HTTP 200.

Po przekierowaniu z powrotem do endpointu `/forgot-password` po pomyślnym żądaniu, zmienna sesji `status` może być użyta do wyświetlenia statusu próby żądania linku resetowania hasła.

Wartość zmiennej sesji `$status` będzie odpowiadać jednemu z ciągów tłumaczenia zdefiniowanych w [pliku językowym](/docs/{{version}}/localization) `passwords` Twojej aplikacji. Jeśli chcesz dostosować tę wartość i nie opublikowałeś plików językowych Laravel, możesz to zrobić za pomocą polecenia Artisan `lang:publish`:

```html
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

Jeśli żądanie nie powiodło się, użytkownik zostanie przekierowany z powrotem na ekran żądania linku resetowania hasła, a błędy walidacji będą dostępne przez współdzieloną [zmienną szablonu Blade](/docs/{{version}}/validation#quick-displaying-the-validation-errors) `$errors`. W przypadku żądania XHR błędy walidacji zostaną zwrócone z odpowiedzią HTTP 422.

<a name="resetting-the-password"></a>
### Resetowanie hasła

Aby zakończyć implementację funkcjonalności resetowania hasła naszej aplikacji, musimy poinstruować Fortify, jak zwrócić nasz widok "resetowania hasła".

Wszystkie logiki renderowania widoków Fortify mogą być dostosowane za pomocą odpowiednich metod dostępnych przez klasę `Laravel\Fortify\Fortify`. Zazwyczaj powinieneś wywołać tę metodę z metody `boot` klasy `App\Providers\FortifyServiceProvider` swojej aplikacji:

```php
use Laravel\Fortify\Fortify;
use Illuminate\Http\Request;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::resetPasswordView(function (Request $request) {
        return view('auth.reset-password', ['request' => $request]);
    });

    // ...
}
```

Fortify zajmie się zdefiniowaniem trasy do wyświetlenia tego widoku. Twój szablon `reset-password` powinien zawierać formularz, który wysyła żądanie POST do `/reset-password`.

Endpoint `/reset-password` oczekuje pola `email` typu string, pola `password`, pola `password_confirmation` oraz ukrytego pola o nazwie `token`, które zawiera wartość `request()->route('token')`. Nazwa pola "email" / kolumny bazy danych powinna odpowiadać wartości konfiguracji `email` zdefiniowanej w pliku konfiguracyjnym `fortify` Twojej aplikacji.

<a name="handling-the-password-reset-response"></a>
#### Obsługa odpowiedzi resetowania hasła

Jeśli żądanie resetowania hasła było pomyślne, Fortify przekieruje z powrotem do trasy `/login`, aby użytkownik mógł zalogować się swoim nowym hasłem. Dodatkowo zmienna sesji `status` zostanie ustawiona, abyś mógł wyświetlić pomyślny status resetu na ekranie logowania:

```blade
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

Jeśli żądanie było żądaniem XHR, zwrócona zostanie odpowiedź HTTP 200.

Jeśli żądanie nie powiodło się, użytkownik zostanie przekierowany z powrotem na ekran resetowania hasła, a błędy walidacji będą dostępne przez współdzieloną [zmienną szablonu Blade](/docs/{{version}}/validation#quick-displaying-the-validation-errors) `$errors`. W przypadku żądania XHR błędy walidacji zostaną zwrócone z odpowiedzią HTTP 422.

<a name="customizing-password-resets"></a>
### Dostosowywanie resetowania hasła

Proces resetowania hasła może być dostosowany poprzez modyfikację akcji `App\Actions\ResetUserPassword`, która została wygenerowana podczas instalacji Laravel Fortify.

<a name="email-verification"></a>
## Weryfikacja e-mail

Po rejestracji możesz chcieć, aby użytkownicy zweryfikowali swój adres email przed kontynuacją korzystania z Twojej aplikacji. Aby rozpocząć, upewnij się, że funkcja `emailVerification` jest włączona w tablicy `features` pliku konfiguracyjnego `fortify`. Następnie powinieneś się upewnić, że Twoja klasa `App\Models\User` implementuje interfejs `Illuminate\Contracts\Auth\MustVerifyEmail`.

Po zakończeniu tych dwóch kroków konfiguracji, nowo zarejestrowani użytkownicy otrzymają email proszący ich o weryfikację własności swojego adresu email. Musimy jednak poinformować Fortify, jak wyświetlić ekran weryfikacji emaila, który informuje użytkownika, że musi kliknąć link weryfikacyjny w emailu.

Wszystkie logiki renderowania widoków Fortify mogą być dostosowane za pomocą odpowiednich metod dostępnych przez klasę `Laravel\Fortify\Fortify`. Zazwyczaj powinieneś wywołać tę metodę z metody `boot` klasy `App\Providers\FortifyServiceProvider` swojej aplikacji:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::verifyEmailView(function () {
        return view('auth.verify-email');
    });

    // ...
}
```

Fortify zajmie się zdefiniowaniem trasy, która wyświetla ten widok, gdy użytkownik zostanie przekierowany do endpointu `/email/verify` przez wbudowane middleware `verified` Laravel.

Twój szablon `verify-email` powinien zawierać komunikat informacyjny instruujący użytkownika, aby kliknął link weryfikacyjny emaila, który został wysłany na jego adres email.

<a name="resending-email-verification-links"></a>
#### Ponowne wysyłanie linków weryfikacji email

Jeśli chcesz, możesz dodać przycisk do szablonu `verify-email` swojej aplikacji, który wywołuje żądanie POST do endpointu `/email/verification-notification`. Gdy ten endpoint otrzyma żądanie, nowy link weryfikacyjny email zostanie wysłany do użytkownika, pozwalając użytkownikowi uzyskać nowy link weryfikacyjny, jeśli poprzedni został przypadkowo usunięty lub zgubiony.

Jeśli żądanie ponownego wysłania emaila z linkiem weryfikacyjnym było pomyślne, Fortify przekieruje użytkownika z powrotem do endpointu `/email/verify` ze zmienną sesji `status`, pozwalając na wyświetlenie komunikatu informacyjnego użytkownikowi, że operacja powiodła się. Jeśli żądanie było żądaniem XHR, zwrócona zostanie odpowiedź HTTP 202:

```blade
@if (session('status') == 'verification-link-sent')
    <div class="mb-4 font-medium text-sm text-green-600">
        Nowy link weryfikacyjny email został wysłany na Twój adres!
    </div>
@endif
```

<a name="protecting-routes"></a>
### Ochrona tras

Aby określić, że trasa lub grupa tras wymaga, aby użytkownik zweryfikował swój adres email, powinieneś dołączyć wbudowane middleware `verified` Laravel do trasy. Alias middleware `verified` jest automatycznie rejestrowany przez Laravel i służy jako alias dla middleware `Illuminate\Auth\Middleware\EnsureEmailIsVerified`:

```php
Route::get('/dashboard', function () {
    // ...
})->middleware(['verified']);
```

<a name="password-confirmation"></a>
## Potwierdzanie hasła

Podczas budowania aplikacji możesz czasami mieć akcje, które powinny wymagać od użytkownika potwierdzenia swojego hasła przed wykonaniem akcji. Zazwyczaj te trasy są chronione przez wbudowane middleware `password.confirm` Laravel.

Aby rozpocząć implementację funkcjonalności potwierdzania hasła, musimy poinstruować Fortify, jak zwrócić widok "potwierdzenia hasła" naszej aplikacji. Pamiętaj, że Fortify to biblioteka uwierzytelniania bez interfejsu użytkownika. Jeśli chcesz mieć gotową implementację frontendową funkcji uwierzytelniania Laravel, powinieneś użyć [zestawu startowego aplikacji](/docs/{{version}}/starter-kits).

Wszystkie logiki renderowania widoków Fortify mogą być dostosowane za pomocą odpowiednich metod dostępnych przez klasę `Laravel\Fortify\Fortify`. Zazwyczaj powinieneś wywołać tę metodę z metody `boot` klasy `App\Providers\FortifyServiceProvider` swojej aplikacji:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::confirmPasswordView(function () {
        return view('auth.confirm-password');
    });

    // ...
}
```

Fortify zajmie się zdefiniowaniem endpointu `/user/confirm-password`, który zwraca ten widok. Twój szablon `confirm-password` powinien zawierać formularz, który wysyła żądanie POST do endpointu `/user/confirm-password`. Endpoint `/user/confirm-password` oczekuje pola `password` zawierającego bieżące hasło użytkownika.

Jeśli hasło pasuje do bieżącego hasła użytkownika, Fortify przekieruje użytkownika na trasę, do której próbował uzyskać dostęp. Jeśli żądanie było żądaniem XHR, zwrócona zostanie odpowiedź HTTP 201.

Jeśli żądanie nie powiodło się, użytkownik zostanie przekierowany z powrotem na ekran potwierdzania hasła, a błędy walidacji będą dostępne przez współdzieloną zmienną szablonu Blade `$errors`. W przypadku żądania XHR błędy walidacji zostaną zwrócone z odpowiedzią HTTP 422.
