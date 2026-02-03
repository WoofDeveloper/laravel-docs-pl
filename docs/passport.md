# Laravel Passport

- [Wprowadzenie](#introduction)
    - [Passport czy Sanctum?](#passport-or-sanctum)
- [Instalacja](#installation)
    - [Wdrażanie Passport](#deploying-passport)
    - [Aktualizacja Passport](#upgrading-passport)
- [Konfiguracja](#configuration)
    - [Czas życia tokenów](#token-lifetimes)
    - [Nadpisywanie domyślnych modeli](#overriding-default-models)
    - [Nadpisywanie tras](#overriding-routes)
- [Authorization Code Grant](#authorization-code-grant)
    - [Zarządzanie klientami](#managing-clients)
    - [Żądanie tokenów](#requesting-tokens)
    - [Zarządzanie tokenami](#managing-tokens)
    - [Odświeżanie tokenów](#refreshing-tokens)
    - [Odwoływanie tokenów](#revoking-tokens)
    - [Czyszczenie tokenów](#purging-tokens)
- [Authorization Code Grant z PKCE](#code-grant-pkce)
    - [Tworzenie klienta](#creating-a-auth-pkce-grant-client)
    - [Żądanie tokenów](#requesting-auth-pkce-grant-tokens)
- [Device Authorization Grant](#device-authorization-grant)
    - [Tworzenie klienta Device Code Grant](#creating-a-device-authorization-grant-client)
    - [Żądanie tokenów](#requesting-device-authorization-grant-tokens)
- [Password Grant](#password-grant)
    - [Tworzenie klienta Password Grant](#creating-a-password-grant-client)
    - [Żądanie tokenów](#requesting-password-grant-tokens)
    - [Żądanie wszystkich zakresów](#requesting-all-scopes)
    - [Dostosowywanie dostawcy użytkowników](#customizing-the-user-provider)
    - [Dostosowywanie pola nazwy użytkownika](#customizing-the-username-field)
    - [Dostosowywanie walidacji hasła](#customizing-the-password-validation)
- [Implicit Grant](#implicit-grant)
- [Client Credentials Grant](#client-credentials-grant)
- [Tokeny dostępu osobistego](#personal-access-tokens)
    - [Tworzenie klienta dostępu osobistego](#creating-a-personal-access-client)
    - [Dostosowywanie dostawcy użytkowników](#customizing-the-user-provider-for-pat)
    - [Zarządzanie tokenami dostępu osobistego](#managing-personal-access-tokens)
- [Ochrona tras](#protecting-routes)
    - [Za pomocą middleware](#via-middleware)
    - [Przekazywanie tokenu dostępu](#passing-the-access-token)
- [Zakresy tokenów](#token-scopes)
    - [Definiowanie zakresów](#defining-scopes)
    - [Zakres domyślny](#default-scope)
    - [Przypisywanie zakresów do tokenów](#assigning-scopes-to-tokens)
    - [Sprawdzanie zakresów](#checking-scopes)
- [Uwierzytelnianie SPA](#spa-authentication)
- [Zdarzenia](#events)
- [Testowanie](#testing)

<a name="introduction"></a>
## Wprowadzenie

[Laravel Passport](https://github.com/laravel/passport) zapewnia pełną implementację serwera OAuth2 dla Twojej aplikacji Laravel w ciągu kilku minut. Passport jest zbudowany na bazie [League OAuth2 server](https://github.com/thephpleague/oauth2-server), który jest utrzymywany przez Andy'ego Millingtona i Simona Hampa.

> [!NOTE]
> Ta dokumentacja zakłada, że znasz już OAuth2. Jeśli nie wiesz nic o OAuth2, rozważ zapoznanie się z ogólną [terminologią](https://oauth2.thephpleague.com/terminology/) i funkcjami OAuth2 przed kontynuowaniem.

<a name="passport-or-sanctum"></a>
### Passport czy Sanctum?

Przed rozpoczęciem możesz chcieć określić, czy Twoja aplikacja będzie lepiej obsługiwana przez Laravel Passport czy [Laravel Sanctum](/docs/{{version}}/sanctum). Jeśli Twoja aplikacja absolutnie musi obsługiwać OAuth2, powinieneś użyć Laravel Passport.

Jednak jeśli próbujesz uwierzytelnić aplikację jednostronicową, aplikację mobilną lub wydać tokeny API, powinieneś użyć [Laravel Sanctum](/docs/{{version}}/sanctum). Laravel Sanctum nie obsługuje OAuth2; jednak zapewnia znacznie prostsze doświadczenie w tworzeniu uwierzytelniania API.

<a name="installation"></a>
## Instalacja

Możesz zainstalować Laravel Passport za pomocą polecenia Artisan `install:api`:

```shell
php artisan install:api --passport
```

To polecenie opublikuje i uruchomi migracje bazy danych niezbędne do utworzenia tabel, których Twoja aplikacja potrzebuje do przechowywania klientów OAuth2 i tokenów dostępu. Polecenie utworzy również klucze szyfrowania wymagane do generowania bezpiecznych tokenów dostępu.

Po uruchomieniu polecenia `install:api` dodaj trait `Laravel\Passport\HasApiTokens` i interfejs `Laravel\Passport\Contracts\OAuthenticatable` do swojego modelu `App\Models\User`. Ten trait zapewni kilka pomocniczych metod do Twojego modelu, które pozwolą Ci sprawdzić token i zakresy uwierzytelnionego użytkownika:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

Wreszcie, w pliku konfiguracyjnym `config/auth.php` Twojej aplikacji, powinieneś zdefiniować strażnika uwierzytelniania `api` i ustawić opcję `driver` na `passport`. To poinstruuje Twoją aplikację, aby używała `TokenGuard` Passport podczas uwierzytelniania przychodzących żądań API:

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

<a name="deploying-passport"></a>
### Wdrażanie Passport

Podczas pierwszego wdrażania Passport na serwery Twojej aplikacji, prawdopodobnie będziesz musiał uruchomić polecenie `passport:keys`. To polecenie generuje klucze szyfrowania, których Passport potrzebuje do generowania tokenów dostępu. Wygenerowane klucze zazwyczaj nie są przechowywane w kontroli źródła:

```shell
php artisan passport:keys
```

Jeśli to konieczne, możesz określić ścieżkę, z której powinny być ładowane klucze Passport. Możesz użyć metody `Passport::loadKeysFrom`, aby to osiągnąć. Zazwyczaj ta metoda powinna być wywołana z metody `boot` klasy `App\Providers\AppServiceProvider` Twojej aplikacji:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::loadKeysFrom(__DIR__.'/../secrets/oauth');
}
```

<a name="loading-keys-from-the-environment"></a>
#### Ładowanie kluczy ze środowiska

Alternatywnie, możesz opublikować plik konfiguracyjny Passport za pomocą polecenia Artisan `vendor:publish`:

```shell
php artisan vendor:publish --tag=passport-config
```

Po opublikowaniu pliku konfiguracyjnego, możesz załadować klucze szyfrowania swojej aplikacji, definiując je jako zmienne środowiskowe:

```ini
PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
<private key here>
-----END RSA PRIVATE KEY-----"

PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
<public key here>
-----END PUBLIC KEY-----"
```

<a name="upgrading-passport"></a>
### Aktualizacja Passport

Podczas aktualizacji do nowej głównej wersji Passport, ważne jest, aby dokładnie przejrzeć [przewodnik aktualizacji](https://github.com/laravel/passport/blob/master/UPGRADE.md).

<a name="configuration"></a>
## Konfiguracja

<a name="token-lifetimes"></a>
### Czas życia tokenów

Domyślnie Passport wydaje długotrwałe tokeny dostępu, które wygasają po roku. Jeśli chcesz skonfigurować dłuższy / krótszy czas życia tokena, możesz użyć metod `tokensExpireIn`, `refreshTokensExpireIn` i `personalAccessTokensExpireIn`. Te metody powinny być wywołane z metody `boot` klasy `App\Providers\AppServiceProvider` Twojej aplikacji:

```php
use Carbon\CarbonInterval;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::tokensExpireIn(CarbonInterval::days(15));
    Passport::refreshTokensExpireIn(CarbonInterval::days(30));
    Passport::personalAccessTokensExpireIn(CarbonInterval::months(6));
}
```

> [!WARNING]
> Kolumny `expires_at` w tabelach bazy danych Passport są tylko do odczytu i służą tylko do wyświetlania. Podczas wydawania tokenów, Passport przechowuje informacje o wygasnięciu w podpisanych i zaszyfrowanych tokenach. Jeśli musisz unieważnić token, powinieneś [go odwołać](#revoking-tokens).

<a name="overriding-default-models"></a>
### Nadpisywanie domyślnych modeli

Możesz swobodnie rozszerzyć modele używane wewnętrznie przez Passport, definiując własny model i rozszerzając odpowiedni model Passport:

```php
use Laravel\Passport\Client as PassportClient;

class Client extends PassportClient
{
    // ...
}
```

Po zdefiniowaniu modelu, możesz poinstruować Passport, aby używał Twojego niestandardowego modelu za pomocą klasy `Laravel\Passport\Passport`. Zazwyczaj powinieneś poinformować Passport o swoich niestandardowych modelach w metodzie `boot` klasy `App\Providers\AppServiceProvider` Twojej aplikacji:

```php
use App\Models\Passport\AuthCode;
use App\Models\Passport\Client;
use App\Models\Passport\DeviceCode;
use App\Models\Passport\RefreshToken;
use App\Models\Passport\Token;
use Laravel\Passport\Passport;

/**
 * Zainicjalizuj dowolne usługi aplikacji.
 */
public function boot(): void
{
    Passport::useTokenModel(Token::class);
    Passport::useRefreshTokenModel(RefreshToken::class);
    Passport::useAuthCodeModel(AuthCode::class);
    Passport::useClientModel(Client::class);
    Passport::useDeviceCodeModel(DeviceCode::class);
}
```

<a name="overriding-routes"></a>
### Nadpisywanie tras

Czasami możesz chcieć dostosować trasy zdefiniowane przez Passport. Aby to osiągnąć, najpierw musisz zignorować trasy rejestrowane przez Passport, dodając `Passport::ignoreRoutes` do metody `register` `AppServiceProvider` Twojej aplikacji:

```php
use Laravel\Passport\Passport;

/**
 * Zarejestruj dowolne usługi aplikacji.
 */
public function register(): void
{
    Passport::ignoreRoutes();
}
```

Następnie możesz skopiować trasy zdefiniowane przez Passport w [jego pliku tras](https://github.com/laravel/passport/blob/master/routes/web.php) do pliku `routes/web.php` Twojej aplikacji i zmodyfikować je według własnych potrzeb:

```php
Route::group([
    'as' => 'passport.',
    'prefix' => config('passport.path', 'oauth'),
    'namespace' => '\Laravel\Passport\Http\Controllers',
], function () {
    // Passport routes...
});
```

<a name="authorization-code-grant"></a>
## Authorization Code Grant

Używanie OAuth2 za pomocą kodów autoryzacyjnych to sposób, z którym większość programistów jest zaznajomiona w OAuth2. Podczas używania kodów autoryzacyjnych aplikacja kliencka przekieruje użytkownika do Twojego serwera, gdzie albo zaakceptuje, albo odrzuci żądanie wydania tokenu dostępu dla klienta.

Aby rozpocząć, musimy poinstruować Passport, jak zwrócić nasz widok "autoryzacji".

Cała logika renderowania widoku autoryzacji może być dostosowana za pomocą odpowiednich metod dostępnych za pośrednictwem klasy `Laravel\Passport\Passport`. Zazwyczaj powinieneś wywołać tę metodę z metody `boot` klasy `App\Providers\AppServiceProvider` Twojej aplikacji:

```php
use Inertia\Inertia;
use Laravel\Passport\Passport;

/**
 * Zainicjalizuj dowolne usługi aplikacji.
 */
public function boot(): void
{
    // Przez podanie nazwy widoku...
    Passport::authorizationView('auth.oauth.authorize');

    // Przez podanie zamknięcia...
    Passport::authorizationView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Authorize', [
            'request' => $parameters['request'],
            'authToken' => $parameters['authToken'],
            'client' => $parameters['client'],
            'user' => $parameters['user'],
            'scopes' => $parameters['scopes'],
        ])
    );
}
```

Passport automatycznie zdefiniuje trasę `/oauth/authorize`, która zwraca ten widok. Twój szablon `auth.oauth.authorize` powinien zawierać formularz, który wykonuje żądanie POST do trasy `passport.authorizations.approve`, aby zatwierdzić autoryzację oraz formularz, który wykonuje żądanie DELETE do trasy `passport.authorizations.deny`, aby odrzucić autoryzację. Trasy `passport.authorizations.approve` i `passport.authorizations.deny` oczekują pól `state`, `client_id` i `auth_token`.

<a name="managing-clients"></a>
### Zarządzanie klientami

Programiści budujący aplikacje, które muszą współdziałać z API Twojej aplikacji, będą musieli zarejestrować swoją aplikację u Ciebie, tworząc "klienta". Zazwyczaj polega to na podaniu nazwy ich aplikacji i URI, na który Twoja aplikacja może przekierować po tym, jak użytkownicy zaakceptują ich żądanie autoryzacji.

<a name="managing-first-party-clients"></a>
#### Klienci pierwszej strony

Najprostszym sposobem utworzenia klienta jest użycie polecenia Artisan `passport:client`. To polecenie może być użyte do tworzenia klientów pierwszej strony lub testowania Twojej funkcjonalności OAuth2. Kiedy uruchomisz polecenie `passport:client`, Passport poprosi Cię o więcej informacji o Twoim kliencie i dostarczy Ci identyfikator klienta i sekret:

```shell
php artisan passport:client
```

Jeśli chcesz zezwolić na wiele URI przekierowania dla swojego klienta, możesz je określić za pomocą listy rozdzielanej przecinkami, gdy zostaniesz poproszony o URI przez polecenie `passport:client`. Wszystkie URI zawierające przecinki powinny być zakodowane jako URI:

```shell
https://third-party-app.com/callback,https://example.com/oauth/redirect
```

<a name="managing-third-party-clients"></a>
#### Klienci trzeciej strony

Ponieważ użytkownicy Twojej aplikacji nie będą mogli używać polecenia `passport:client`, możesz użyć metody `createAuthorizationCodeGrantClient` klasy `Laravel\Passport\ClientRepository`, aby zarejestrować klienta dla danego użytkownika:

```php
use App\Models\User;
use Laravel\Passport\ClientRepository;

$user = User::find($userId);

// Tworzenie klienta aplikacji OAuth, który należy do danego użytkownika...
$client = app(ClientRepository::class)->createAuthorizationCodeGrantClient(
    user: $user,
    name: 'Example App',
    redirectUris: ['https://third-party-app.com/callback'],
    confidential: false,
    enableDeviceFlow: true
);

// Pobieranie wszystkich klientów aplikacji OAuth, które należą do użytkownika...
$clients = $user->oauthApps()->get();
```

Metoda `createAuthorizationCodeGrantClient` zwraca instancję `Laravel\Passport\Client`. Możesz wyświetlić `$client->id` jako identyfikator klienta i `$client->plainSecret` jako sekret klienta użytkownikowi.

<a name="requesting-tokens"></a>
### Żądanie tokenów

<a name="requesting-tokens-redirecting-for-authorization"></a>
#### Przekierowanie w celu autoryzacji

Po utworzeniu klienta programiści mogą użyć swojego identyfikatora klienta i sekretu, aby zażądać kodu autoryzacyjnego i tokenu dostępu z Twojej aplikacji. Najpierw aplikacja konsumująca powinna wykonać żądanie przekierowania do trasy `/oauth/authorize` Twojej aplikacji w następujący sposób:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
        'state' => $state,
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

Parametr `prompt` może być używany do określenia zachowania uwierzytelniania aplikacji Passport.

Jeśli wartością `prompt` jest `none`, Passport zawsze zgłosi błąd uwierzytelniania, jeśli użytkownik nie jest już uwierzytelniony w aplikacji Passport. Jeśli wartością jest `consent`, Passport zawsze wyświetli ekran zatwierdzania autoryzacji, nawet jeśli wszystkie zakresy zostały wcześniej przyznane aplikacji konsumującej. Gdy wartością jest `login`, aplikacja Passport zawsze poprosi użytkownika o ponowne zalogowanie do aplikacji, nawet jeśli ma już istniejącą sesję.

Jeśli nie podano wartości `prompt`, użytkownik zostanie poproszony o autoryzację tylko wtedy, gdy wcześniej nie autoryzowal dostępu do aplikacji konsumującej dla żądanych zakresów.

> [!NOTE]
> Pamiętaj, że trasa `/oauth/authorize` jest już zdefiniowana przez Passport. Nie musisz ręcznie definiować tej trasy.

<a name="approving-the-request"></a>
#### Zatwierdzanie żądania

Podczas otrzymywania żądań autoryzacji, Passport automatycznie odpowie na podstawie wartości parametru `prompt` (jeśli obecny) i może wyświetlić szablon użytkownikowi, umożliwiając mu zatwierdzenie lub odrzucenie żądania autoryzacji. Jeśli zaakceptują żądanie, zostaną przekierowani z powrotem do `redirect_uri`, który został określony przez aplikację konsumującą. `redirect_uri` musi pasować do adresu URL `redirect`, który został określony podczas tworzenia klienta.

Czasami możesz chcieć pominąć monit o autoryzację, na przykład podczas autoryzacji klienta pierwszej strony. Możesz to osiągnąć, [rozszerzając model `Client`](#overriding-default-models) i definiując metodę `skipsAuthorization`. Jeśli `skipsAuthorization` zwraca `true`, klient zostanie zatwierdzony, a użytkownik zostanie natychmiast przekierowany z powrotem do `redirect_uri`, chyba że aplikacja konsumująca wyraźnie ustawiła parametr `prompt` podczas przekierowywania w celu autoryzacji:

```php
<?php

namespace App\Models\Passport;

use Illuminate\Contracts\Auth\Authenticatable;
use Laravel\Passport\Client as BaseClient;

class Client extends BaseClient
{
    /**
     * Określ, czy klient powinien pominąć monit o autoryzację.
     *
     * @param  \Laravel\Passport\Scope[]  $scopes
     */
    public function skipsAuthorization(Authenticatable $user, array $scopes): bool
    {
        return $this->firstParty();
    }
}
```

<a name="requesting-tokens-converting-authorization-codes-to-access-tokens"></a>
#### Konwersja kodów autoryzacyjnych na tokeny dostępu

Jeśli użytkownik zatwierdzi żądanie autoryzacji, zostanie przekierowany z powrotem do aplikacji konsumującej. Konsument powinien najpierw zweryfikować parametr `state` względem wartości, która została przechowana przed przekierowaniem. Jeśli parametr state się zgadza, konsument powinien wysłać żądanie `POST` do Twojej aplikacji, aby zażądać tokenu dostępu. Żądanie powinno zawierać kod autoryzacyjny, który został wydany przez Twoją aplikację, gdy użytkownik zatwierdził żądanie autoryzacji:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class,
        'Invalid state value.'
    );

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'code' => $request->code,
    ]);

    return $response->json();
});
```

Ta trasa `/oauth/token` zwróci odpowiedź JSON zawierającą atrybuty `access_token`, `refresh_token` i `expires_in`. Atrybut `expires_in` zawiera liczbę sekund do wygasnięcia tokenu dostępu.

> [!NOTE]
> Podobnie jak trasa `/oauth/authorize`, trasa `/oauth/token` jest zdefiniowana dla Ciebie przez Passport. Nie ma potrzeby ręcznego definiowania tej trasy.

<a name="managing-tokens"></a>
### Zarządzanie tokenami

Możesz pobrać autoryzowane tokeny użytkownika, używając metody `tokens` traitu `Laravel\Passport\HasApiTokens`. Na przykład może to być użyte do zaoferowania użytkownikom panelu do śledzenia ich połączeń z aplikacjami trzecimi:

```php
use App\Models\User;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Support\Facades\Date;
use Laravel\Passport\Token;

$user = User::find($userId);

// Pobieranie wszystkich ważnych tokenów użytkownika...
$tokens = $user->tokens()
    ->where('revoked', false)
    ->where('expires_at', '>', Date::now())
    ->get();

// Pobieranie wszystkich połączeń użytkownika z klientami aplikacji OAuth trzecich...
$connections = $tokens->load('client')
    ->reject(fn (Token $token) => $token->client->firstParty())
    ->groupBy('client_id')
    ->map(fn (Collection $tokens) => [
        'client' => $tokens->first()->client,
        'scopes' => $tokens->pluck('scopes')->flatten()->unique()->values()->all(),
        'tokens_count' => $tokens->count(),
    ])
    ->values();
```

<a name="refreshing-tokens"></a>
### Odświeżanie tokenów

Jeśli Twoja aplikacja wydaje krótkotrwałe tokeny dostępu, użytkownicy będą musieli odświeżyć swoje tokeny dostępu za pomocą tokenu odświeżającego, który został im dostarczony podczas wydawania tokenu dostępu:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'refresh_token',
    'refresh_token' => 'the-refresh-token',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // Required for confidential clients only...
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

Ta trasa `/oauth/token` zwróci odpowiedź JSON zawierającą atrybuty `access_token`, `refresh_token` i `expires_in`. Atrybut `expires_in` zawiera liczbę sekund do wygasnięcia tokenu dostępu.

<a name="revoking-tokens"></a>
### Odwoływanie tokenów

Możesz odwołać token, używając metody `revoke` na modelu `Laravel\Passport\Token`. Możesz odwołać token odświeżający tokena, używając metody `revoke` na modelu `Laravel\Passport\RefreshToken`:

```php
use Laravel\Passport\Passport;
use Laravel\Passport\Token;

$token = Passport::token()->find($tokenId);

// Odwoływanie tokenu dostępu...
$token->revoke();

// Odwoływanie tokenu odświeżającego tokena...
$token->refreshToken?->revoke();

// Odwoływanie wszystkich tokenów użytkownika...
User::find($userId)->tokens()->each(function (Token $token) {
    $token->revoke();
    $token->refreshToken?->revoke();
});
```

<a name="purging-tokens"></a>
### Czyszczenie tokenów

Gdy tokeny zostały odwołane lub wygasły, możesz chcieć usunąć je z bazy danych. Dołączone polecenie Artisan `passport:purge` Passport może to dla Ciebie zrobić:

```shell
# Czyszczenie odwołanych i wygasłych tokenów, kodów autoryzacyjnych i kodów urządzeń...
php artisan passport:purge

# Czyszczenie tylko tokenów wygasłych ponad 6 godzin temu...
php artisan passport:purge --hours=6

# Czyszczenie tylko odwołanych tokenów, kodów autoryzacyjnych i kodów urządzeń...
php artisan passport:purge --revoked

# Czyszczenie tylko wygasłych tokenów, kodów autoryzacyjnych i kodów urządzeń...
php artisan passport:purge --expired
```

Możesz również skonfigurować [zaplanowane zadanie](/docs/{{version}}/scheduling) w pliku `routes/console.php` Twojej aplikacji, aby automatycznie czyścić tokeny zgodnie z harmonogramem:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('passport:purge')->hourly();
```

<a name="code-grant-pkce"></a>
## Authorization Code Grant z PKCE

Autoryzacja za pomocą kodu autoryzacyjnego z "Proof Key for Code Exchange" (PKCE) to bezpieczny sposób uwierzytelniania aplikacji jednostronicowych lub aplikacji mobilnych w celu uzyskania dostępu do Twojego API. To przyznanie powinno być używane, gdy nie możesz zagwarantować, że sekret klienta będzie przechowywany poufnie lub w celu złagodzenia zagrożenia przechwycenia kodu autoryzacyjnego.

<a name="creating-a-auth-pkce-grant-client"></a>
### Tworzenie klienta

Zanim Twoja aplikacja będzie mogła wydawać tokeny przez autoryzację z kodem z PKCE, będziesz musiał utworzyć klienta z włączonym PKCE. Możesz to zrobić za pomocą polecenia Artisan `passport:client` z opcją `--public`:

```shell
php artisan passport:client --public
```

<a name="requesting-auth-pkce-grant-tokens"></a>
### Żądanie tokenów

<a name="code-verifier-code-challenge"></a>
#### Weryfikator kodu i wyzwanie kodowe

Ponieważ to przyznanie autoryzacji nie zapewnia sekretu klienta, programiści będą musieli wygenerować kombinację weryfikatora kodu i wyzwania kodowego, aby zażądać tokena.

Weryfikator kodu powinien być losowym ciągiem znaków o długości od 43 do 128 znaków, zawierającym litery, cyfry i znaki `"-"`, `"."`, `"_"`, `"~"`, zgodnie z definicją w [specyfikacji RFC 7636](https://tools.ietf.org/html/rfc7636).

Wyzwanie kodowe powinno być ciągiem zakodowanym w Base64 z bezpiecznymi dla URL i nazw plików znakami. Końcowe znaki `'='` powinny zostać usunięte i nie powinny być obecne żadne znaki nowej linii, białe znaki ani inne dodatkowe znaki.

```php
$encoded = base64_encode(hash('sha256', $codeVerifier, true));

$codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');
```

<a name="code-grant-pkce-redirecting-for-authorization"></a>
#### Przekierowanie w celu autoryzacji

Po utworzeniu klienta możesz użyć identyfikatora klienta oraz wygenerowanego weryfikatora kodu i wyzwania kodowego, aby zażądać kodu autoryzacyjnego i tokenu dostępu z Twojej aplikacji. Najpierw aplikacja konsumująca powinna wykonać żądanie przekierowania do trasy `/oauth/authorize` Twojej aplikacji:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $request->session()->put(
        'code_verifier', $codeVerifier = Str::random(128)
    );

    $codeChallenge = strtr(rtrim(
        base64_encode(hash('sha256', $codeVerifier, true))
    , '='), '+/', '-_');

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
        'state' => $state,
        'code_challenge' => $codeChallenge,
        'code_challenge_method' => 'S256',
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

<a name="code-grant-pkce-converting-authorization-codes-to-access-tokens"></a>
#### Konwersja kodów autoryzacyjnych na tokeny dostępu

Jeśli użytkownik zatwierdzi żądanie autoryzacji, zostanie przekierowany z powrotem do aplikacji konsumującej. Konsument powinien zweryfikować parametr `state` względem wartości, która została przechowana przed przekierowaniem, tak jak w standardowym Authorization Code Grant.

Jeśli parametr state się zgadza, konsument powinien wysłać żądanie `POST` do Twojej aplikacji, aby zażądać tokenu dostępu. Żądanie powinno zawierać kod autoryzacyjny, który został wydany przez Twoją aplikację, gdy użytkownik zatwierdził żądanie autoryzacji wraz z pierwotnie wygenerowanym weryfikatorem kodu:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    $codeVerifier = $request->session()->pull('code_verifier');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class
    );

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'code_verifier' => $codeVerifier,
        'code' => $request->code,
    ]);

    return $response->json();
});
```

<a name="device-authorization-grant"></a>
## Device Authorization Grant

Przyznanie autoryzacji urządzenia OAuth2 umożliwia urządzeniom bez przeglądarki lub z ograniczonym wprowadzaniem danych, takim jak telewizory i konsole do gier, uzyskanie tokenu dostępu poprzez wymianę "kodu urządzenia". Podczas korzystania z przepływu urządzenia klient urządzenia poinstruuje użytkownika, aby użył urządzenia pomocniczego, takiego jak komputer lub smartfon, i połączył się z Twoim serwerem, gdzie wprowadzi dostarczony "kod użytkownika" i albo zatwierdzi, albo odrzuci żądanie dostępu.

To get started, we need to instruct Passport how to return our "user code" and "authorization" views.

All the authorization view's rendering logic may be customized using the appropriate methods available via the `Laravel\Passport\Passport` class. Typically, you should call this method from the `boot` method of your application's `App\Providers\AppServiceProvider` class.

```php
use Inertia\Inertia;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    // By providing a view name...
    Passport::deviceUserCodeView('auth.oauth.device.user-code');
    Passport::deviceAuthorizationView('auth.oauth.device.authorize');

    // By providing a closure...
    Passport::deviceUserCodeView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Device/UserCode')
    );

    Passport::deviceAuthorizationView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Device/Authorize', [
            'request' => $parameters['request'],
            'authToken' => $parameters['authToken'],
            'client' => $parameters['client'],
            'user' => $parameters['user'],
            'scopes' => $parameters['scopes'],
        ])
    );

    // ...
}
```

Passport will automatically define routes that return these views. Your `auth.oauth.device.user-code` template should include a form that makes a GET request to the `passport.device.authorizations.authorize` route. The `passport.device.authorizations.authorize` route expects a `user_code` query parameter.

Your `auth.oauth.device.authorize` template should include a form that makes a POST request to the `passport.device.authorizations.approve` route to approve the authorization and a form that makes a DELETE request to the `passport.device.authorizations.deny` route to deny the authorization. The `passport.device.authorizations.approve` and `passport.device.authorizations.deny` routes expect `state`, `client_id`, and `auth_token` fields.

<a name="creating-a-device-authorization-grant-client"></a>
### Creating a Device Authorization Grant Client

Before your application can issue tokens via the device authorization grant, you will need to create a device flow enabled client. You may do this using the `passport:client` Artisan command with the `--device` option. This command will create a first-party device flow enabled client and provide you with a client ID and secret:

```shell
php artisan passport:client --device
```

Additionally, you may use `createDeviceAuthorizationGrantClient` method on the `ClientRepository` class to register a third-party client that belongs to the given user:

```php
use App\Models\User;
use Laravel\Passport\ClientRepository;

$user = User::find($userId);

$client = app(ClientRepository::class)->createDeviceAuthorizationGrantClient(
    user: $user,
    name: 'Example Device',
    confidential: false,
);
```

<a name="requesting-device-authorization-grant-tokens"></a>
### Requesting Tokens

<a name="device-code"></a>
#### Requesting a Device Code

Once a client has been created, developers may use their client ID to request a device code from your application. First, the consuming device should make a `POST` request to your application's `/oauth/device/code` route to request a device code:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/device/code', [
    'client_id' => 'your-client-id',
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

This will return a JSON response containing `device_code`, `user_code`, `verification_uri`, `interval`, and `expires_in` attributes. The `expires_in` attribute contains the number of seconds until the device code expires. The `interval` attribute contains the number of seconds the consuming device should wait between requests when polling `/oauth/token` route to avoid rate limit errors.

> [!NOTE]
> Remember, the `/oauth/device/code` route is already defined by Passport. You do not need to manually define this route.

<a name="user-code"></a>
#### Displaying the Verification URI and User Code

Once a device code request has been obtained, the consuming device should instruct the user to use another device and visit the provided `verification_uri` and enter the `user_code` in order to approve the authorization request.

<a name="polling-token-request"></a>
#### Polling Token Request

Since the user will be using a separate device to grant (or deny) access, the consuming device should poll your application's `/oauth/token` route to determine when the user has responded to the request. The consuming device should use the minimum polling `interval` provided in the JSON response when requesting device code to avoid rate limit errors:

```php
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Sleep;

$interval = 5;

do {
    Sleep::for($interval)->seconds();

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'urn:ietf:params:oauth:grant-type:device_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret', // Required for confidential clients only...
        'device_code' => 'the-device-code',
    ]);

    if ($response->json('error') === 'slow_down') {
        $interval += 5;
    }
} while (in_array($response->json('error'), ['authorization_pending', 'slow_down']));

return $response->json();
```

If the user has approved the authorization request, this will return a JSON response containing `access_token`, `refresh_token`, and `expires_in` attributes. The `expires_in` attribute contains the number of seconds until the access token expires.

<a name="password-grant"></a>
## Password Grant

> [!WARNING]
> Nie zalecamy już używania tokenów przyznania hasła. Zamiast tego powinieneś wybrać [typ przyznania aktualnie zalecany przez OAuth2 Server](https://oauth2.thephpleague.com/authorization-server/which-grant/).

Przyznanie hasła OAuth2 pozwala Twoim innym klientom pierwszej strony, takim jak aplikacja mobilna, uzyskać token dostępu za pomocą adresu e-mail / nazwy użytkownika i hasła. Pozwala to bezpiecznie wydawać tokeny dostępu klientom pierwszej strony bez wymagania od użytkowników przejścia przez cały przepływ przekierowania kodu autoryzacyjnego OAuth2.

Aby włączyć przyznanie hasła, wywołaj metodę `enablePasswordGrant` w metodzie `boot` klasy `App\Providers\AppServiceProvider` Twojej aplikacji:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::enablePasswordGrant();
}
```

<a name="creating-a-password-grant-client"></a>
### Creating a Password Grant Client

Before your application can issue tokens via the password grant, you will need to create a password grant client. You may do this using the `passport:client` Artisan command with the `--password` option.

```shell
php artisan passport:client --password
```

<a name="requesting-password-grant-tokens"></a>
### Requesting Tokens

Once you have enabled the grant and have created a password grant client, you may request an access token by issuing a `POST` request to the `/oauth/token` route with the user's email address and password. Remember, this route is already registered by Passport so there is no need to define it manually. If the request is successful, you will receive an `access_token` and `refresh_token` in the JSON response from the server:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // Required for confidential clients only...
    'username' => 'taylor@laravel.com',
    'password' => 'my-password',
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

> [!NOTE]
> Remember, access tokens are long-lived by default. However, you are free to [configure your maximum access token lifetime](#configuration) if needed.

<a name="requesting-all-scopes"></a>
### Requesting All Scopes

When using the password grant or client credentials grant, you may wish to authorize the token for all of the scopes supported by your application. You can do this by requesting the `*` scope. If you request the `*` scope, the `can` method on the token instance will always return `true`. This scope may only be assigned to a token that is issued using the `password` or `client_credentials` grant:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // Required for confidential clients only...
    'username' => 'taylor@laravel.com',
    'password' => 'my-password',
    'scope' => '*',
]);
```

<a name="customizing-the-user-provider"></a>
### Customizing the User Provider

If your application uses more than one [authentication user provider](/docs/{{version}}/authentication#introduction), you may specify which user provider the password grant client uses by providing a `--provider` option when creating the client via the `artisan passport:client --password` command. The given provider name should match a valid provider defined in your application's `config/auth.php` configuration file. You can then [protect your route using middleware](#multiple-authentication-guards) to ensure that only users from the guard's specified provider are authorized.

<a name="customizing-the-username-field"></a>
### Customizing the Username Field

When authenticating using the password grant, Passport will use the `email` attribute of your authenticatable model as the "username". However, you may customize this behavior by defining a `findForPassport` method on your model:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\Bridge\Client;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * Find the user instance for the given username.
     */
    public function findForPassport(string $username, Client $client): User
    {
        return $this->where('username', $username)->first();
    }
}
```

<a name="customizing-the-password-validation"></a>
### Customizing the Password Validation

When authenticating using the password grant, Passport will use the `password` attribute of your model to validate the given password. If your model does not have a `password` attribute or you wish to customize the password validation logic, you can define a `validateForPassportPasswordGrant` method on your model:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Support\Facades\Hash;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * Validate the password of the user for the Passport password grant.
     */
    public function validateForPassportPasswordGrant(string $password): bool
    {
        return Hash::check($password, $this->password);
    }
}
```

<a name="implicit-grant"></a>
## Implicit Grant

> [!WARNING]
> We no longer recommend using implicit grant tokens. Instead, you should choose [a grant type that is currently recommended by OAuth2 Server](https://oauth2.thephpleague.com/authorization-server/which-grant/).

The implicit grant is similar to the authorization code grant; however, the token is returned to the client without exchanging an authorization code. This grant is most commonly used for JavaScript or mobile applications where the client credentials can't be securely stored. To enable the grant, call the `enableImplicitGrant` method in the `boot` method of your application's `App\Providers\AppServiceProvider` class:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::enableImplicitGrant();
}
```

Before your application can issue tokens via the implicit grant, you will need to create an implicit grant client. You may do this using the `passport:client` Artisan command with the `--implicit` option.

```shell
php artisan passport:client --implicit
```

Once the grant has been enabled and an implicit client has been created, developers may use their client ID to request an access token from your application. The consuming application should make a redirect request to your application's `/oauth/authorize` route like so:

```php
use Illuminate\Http\Request;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'token',
        'scope' => 'user:read orders:create',
        'state' => $state,
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

> [!NOTE]
> Remember, the `/oauth/authorize` route is already defined by Passport. You do not need to manually define this route.

<a name="client-credentials-grant"></a>
## Client Credentials Grant

The client credentials grant is suitable for machine-to-machine authentication. For example, you might use this grant in a scheduled job which is performing maintenance tasks over an API.

Before your application can issue tokens via the client credentials grant, you will need to create a client credentials grant client. You may do this using the `--client` option of the `passport:client` Artisan command:

```shell
php artisan passport:client --client
```

Next, assign the `Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner` middleware to a route:

```php
use Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner;

Route::get('/orders', function (Request $request) {
    // Access token is valid and the client is resource owner...
})->middleware(EnsureClientIsResourceOwner::class);
```

To restrict access to the route to specific scopes, you may provide a list of the required scopes to the `using` method`:

```php
Route::get('/orders', function (Request $request) {
    // Access token is valid, the client is resource owner, and has both "servers:read" and "servers:create" scopes...
})->middleware(EnsureClientIsResourceOwner::using('servers:read', 'servers:create'));
```

<a name="retrieving-tokens"></a>
### Retrieving Tokens

To retrieve a token using this grant type, make a request to the `oauth/token` endpoint:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'client_credentials',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret',
    'scope' => 'servers:read servers:create',
]);

return $response->json()['access_token'];
```

<a name="personal-access-tokens"></a>
## Personal Access Tokens

Sometimes, your users may want to issue access tokens to themselves without going through the typical authorization code redirect flow. Allowing users to issue tokens to themselves via your application's UI can be useful for allowing users to experiment with your API or may serve as a simpler approach to issuing access tokens in general.

> [!NOTE]
> If your application is using Passport primarily to issue personal access tokens, consider using [Laravel Sanctum](/docs/{{version}}/sanctum), Laravel's light-weight first-party library for issuing API access tokens.

<a name="creating-a-personal-access-client"></a>
### Creating a Personal Access Client

Before your application can issue personal access tokens, you will need to create a personal access client. You may do this by executing the `passport:client` Artisan command with the `--personal` option. If you have already run the `passport:install` command, you do not need to run this command:

```shell
php artisan passport:client --personal
```

<a name="customizing-the-user-provider-for-pat"></a>
### Customizing the User Provider

If your application uses more than one [authentication user provider](/docs/{{version}}/authentication#introduction), you may specify which user provider the personal access grant client uses by providing a `--provider` option when creating the client via the `artisan passport:client --personal` command. The given provider name should match a valid provider defined in your application's `config/auth.php` configuration file. You can then [protect your route using middleware](#multiple-authentication-guards) to ensure that only users from the guard's specified provider are authorized.

<a name="managing-personal-access-tokens"></a>
### Managing Personal Access Tokens

Once you have created a personal access client, you may issue tokens for a given user using the `createToken` method on the `App\Models\User` model instance. The `createToken` method accepts the name of the token as its first argument and an optional array of [scopes](#token-scopes) as its second argument:

```php
use App\Models\User;
use Illuminate\Support\Facades\Date;
use Laravel\Passport\Token;

$user = User::find($userId);

// Creating a token without scopes...
$token = $user->createToken('My Token')->accessToken;

// Creating a token with scopes...
$token = $user->createToken('My Token', ['user:read', 'orders:create'])->accessToken;

// Creating a token with all scopes...
$token = $user->createToken('My Token', ['*'])->accessToken;

// Retrieving all the valid personal access tokens that belong to the user...
$tokens = $user->tokens()
    ->with('client')
    ->where('revoked', false)
    ->where('expires_at', '>', Date::now())
    ->get()
    ->filter(fn (Token $token) => $token->client->hasGrantType('personal_access'));
```

<a name="protecting-routes"></a>
## Protecting Routes

<a name="via-middleware"></a>
### Via Middleware

Passport includes an [authentication guard](/docs/{{version}}/authentication#adding-custom-guards) that will validate access tokens on incoming requests. Once you have configured the `api` guard to use the `passport` driver, you only need to specify the `auth:api` middleware on any routes that should require a valid access token:

```php
Route::get('/user', function () {
    // Only API authenticated users may access this route...
})->middleware('auth:api');
```

> [!WARNING]
> If you are using the [client credentials grant](#client-credentials-grant), you should use [the `Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner` middleware](#client-credentials-grant) to protect your routes instead of the `auth:api` middleware.

<a name="multiple-authentication-guards"></a>
#### Multiple Authentication Guards

If your application authenticates different types of users that perhaps use entirely different Eloquent models, you will likely need to define a guard configuration for each user provider type in your application. This allows you to protect requests intended for specific user providers. For example, given the following guard configuration the `config/auth.php` configuration file:

```php
'guards' => [
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],

    'api-customers' => [
        'driver' => 'passport',
        'provider' => 'customers',
    ],
],
```

The following route will utilize the `api-customers` guard, which uses the `customers` user provider, to authenticate incoming requests:

```php
Route::get('/customer', function () {
    // ...
})->middleware('auth:api-customers');
```

> [!NOTE]
> For more information on using multiple user providers with Passport, please consult the [personal access tokens documentation](#customizing-the-user-provider-for-pat) and [password grant documentation](#customizing-the-user-provider).

<a name="passing-the-access-token"></a>
### Passing the Access Token

When calling routes that are protected by Passport, your application's API consumers should specify their access token as a `Bearer` token in the `Authorization` header of their request. For example, when using the `Http` Facade:

```php
use Illuminate\Support\Facades\Http;

$response = Http::withHeaders([
    'Accept' => 'application/json',
    'Authorization' => "Bearer $accessToken",
])->get('https://passport-app.test/api/user');

return $response->json();
```

<a name="token-scopes"></a>
## Token Scopes

Scopes allow your API clients to request a specific set of permissions when requesting authorization to access an account. For example, if you are building an e-commerce application, not all API consumers will need the ability to place orders. Instead, you may allow the consumers to only request authorization to access order shipment statuses. In other words, scopes allow your application's users to limit the actions a third-party application can perform on their behalf.

<a name="defining-scopes"></a>
### Defining Scopes

You may define your API's scopes using the `Passport::tokensCan` method in the `boot` method of your application's `App\Providers\AppServiceProvider` class. The `tokensCan` method accepts an array of scope names and scope descriptions. The scope description may be anything you wish and will be displayed to users on the authorization approval screen:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::tokensCan([
        'user:read' => 'Retrieve the user info',
        'orders:create' => 'Place orders',
        'orders:read:status' => 'Check order status',
    ]);
}
```

<a name="default-scope"></a>
### Default Scope

If a client does not request any specific scopes, you may configure your Passport server to attach default scopes to the token using the `defaultScopes` method. Typically, you should call this method from the `boot` method of your application's `App\Providers\AppServiceProvider` class:

```php
use Laravel\Passport\Passport;

Passport::tokensCan([
    'user:read' => 'Retrieve the user info',
    'orders:create' => 'Place orders',
    'orders:read:status' => 'Check order status',
]);

Passport::defaultScopes([
    'user:read',
    'orders:create',
]);
```

<a name="assigning-scopes-to-tokens"></a>
### Assigning Scopes to Tokens

<a name="when-requesting-authorization-codes"></a>
#### When Requesting Authorization Codes

When requesting an access token using the authorization code grant, consumers should specify their desired scopes as the `scope` query string parameter. The `scope` parameter should be a space-delimited list of scopes:

```php
Route::get('/redirect', function () {
    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

<a name="when-issuing-personal-access-tokens"></a>
#### When Issuing Personal Access Tokens

If you are issuing personal access tokens using the `App\Models\User` model's `createToken` method, you may pass the array of desired scopes as the second argument to the method:

```php
$token = $user->createToken('My Token', ['orders:create'])->accessToken;
```

<a name="checking-scopes"></a>
### Checking Scopes

Passport includes two middleware that may be used to verify that an incoming request is authenticated with a token that has been granted a given scope.

<a name="check-for-all-scopes"></a>
#### Check For All Scopes

The `Laravel\Passport\Http\Middleware\CheckToken` middleware may be assigned to a route to verify that the incoming request's access token has all the listed scopes:

```php
use Laravel\Passport\Http\Middleware\CheckToken;

Route::get('/orders', function () {
    // Access token has both "orders:read" and "orders:create" scopes...
})->middleware(['auth:api', CheckToken::using('orders:read', 'orders:create')]);
```

<a name="check-for-any-scopes"></a>
#### Check for Any Scopes

The `Laravel\Passport\Http\Middleware\CheckTokenForAnyScope` middleware may be assigned to a route to verify that the incoming request's access token has *at least one* of the listed scopes:

```php
use Laravel\Passport\Http\Middleware\CheckTokenForAnyScope;

Route::get('/orders', function () {
    // Access token has either "orders:read" or "orders:create" scope...
})->middleware(['auth:api', CheckTokenForAnyScope::using('orders:read', 'orders:create')]);
```

<a name="checking-scopes-on-a-token-instance"></a>
#### Checking Scopes on a Token Instance

Once an access token authenticated request has entered your application, you may still check if the token has a given scope using the `tokenCan` method on the authenticated `App\Models\User` instance:

```php
use Illuminate\Http\Request;

Route::get('/orders', function (Request $request) {
    if ($request->user()->tokenCan('orders:create')) {
        // ...
    }
});
```

<a name="additional-scope-methods"></a>
#### Additional Scope Methods

The `scopeIds` method will return an array of all defined IDs / names:

```php
use Laravel\Passport\Passport;

Passport::scopeIds();
```

The `scopes` method will return an array of all defined scopes as instances of `Laravel\Passport\Scope`:

```php
Passport::scopes();
```

The `scopesFor` method will return an array of `Laravel\Passport\Scope` instances matching the given IDs / names:

```php
Passport::scopesFor(['user:read', 'orders:create']);
```

You may determine if a given scope has been defined using the `hasScope` method:

```php
Passport::hasScope('orders:create');
```

<a name="spa-authentication"></a>
## SPA Authentication

When building an API, it can be extremely useful to be able to consume your own API from your JavaScript application. This approach to API development allows your own application to consume the same API that you are sharing with the world. The same API may be consumed by your web application, mobile applications, third-party applications, and any SDKs that you may publish on various package managers.

Typically, if you want to consume your API from your JavaScript application, you would need to manually send an access token to the application and pass it with each request to your application. However, Passport includes a middleware that can handle this for you. All you need to do is append the `CreateFreshApiToken` middleware to the `web` middleware group in your application's `bootstrap/app.php` file:

```php
use Laravel\Passport\Http\Middleware\CreateFreshApiToken;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->web(append: [
        CreateFreshApiToken::class,
    ]);
})
```

> [!WARNING]
> You should ensure that the `CreateFreshApiToken` middleware is the last middleware listed in your middleware stack.

This middleware will attach a `laravel_token` cookie to your outgoing responses. This cookie contains an encrypted JWT that Passport will use to authenticate API requests from your JavaScript application. The JWT has a lifetime equal to your `session.lifetime` configuration value. Now, since the browser will automatically send the cookie with all subsequent requests, you may make requests to your application's API without explicitly passing an access token:

```js
axios.get('/api/user')
    .then(response => {
        console.log(response.data);
    });
```

<a name="customizing-the-cookie-name"></a>
#### Customizing the Cookie Name

If needed, you can customize the `laravel_token` cookie's name using the `Passport::cookie` method. Typically, this method should be called from the `boot` method of your application's `App\Providers\AppServiceProvider` class:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::cookie('custom_name');
}
```

<a name="csrf-protection"></a>
#### CSRF Protection

When using this method of authentication, you will need to ensure a valid CSRF token header is included in your requests. The default Laravel JavaScript scaffolding included with the skeleton application and all starter kits includes an [Axios](https://github.com/axios/axios) instance, which will automatically use the encrypted `XSRF-TOKEN` cookie value to send an `X-XSRF-TOKEN` header on same-origin requests.

> [!NOTE]
> If you choose to send the `X-CSRF-TOKEN` header instead of `X-XSRF-TOKEN`, you will need to use the unencrypted token provided by `csrf_token()`.

<a name="events"></a>
## Events

Passport raises events when issuing access tokens and refresh tokens. You may [listen for these events](/docs/{{version}}/events) to prune or revoke other access tokens in your database:

<div class="overflow-auto">

| Event Name                                    |
| --------------------------------------------- |
| `Laravel\Passport\Events\AccessTokenCreated`  |
| `Laravel\Passport\Events\AccessTokenRevoked`  |
| `Laravel\Passport\Events\RefreshTokenCreated` |

</div>

<a name="testing"></a>
## Testing

Passport's `actingAs` method may be used to specify the currently authenticated user as well as its scopes. The first argument given to the `actingAs` method is the user instance and the second is an array of scopes that should be granted to the user's token:

```php tab=Pest
use App\Models\User;
use Laravel\Passport\Passport;

test('orders can be created', function () {
    Passport::actingAs(
        User::factory()->create(),
        ['orders:create']
    );

    $response = $this->post('/api/orders');

    $response->assertStatus(201);
});
```

```php tab=PHPUnit
use App\Models\User;
use Laravel\Passport\Passport;

public function test_orders_can_be_created(): void
{
    Passport::actingAs(
        User::factory()->create(),
        ['orders:create']
    );

    $response = $this->post('/api/orders');

    $response->assertStatus(201);
}
```

Passport's `actingAsClient` method may be used to specify the currently authenticated client as well as its scopes. The first argument given to the `actingAsClient` method is the client instance and the second is an array of scopes that should be granted to the client's token:

```php tab=Pest
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

test('servers can be retrieved', function () {
    Passport::actingAsClient(
        Client::factory()->create(),
        ['servers:read']
    );

    $response = $this->get('/api/servers');

    $response->assertStatus(200);
});
```

```php tab=PHPUnit
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

public function test_servers_can_be_retrieved(): void
{
    Passport::actingAsClient(
        Client::factory()->create(),
        ['servers:read']
    );

    $response = $this->get('/api/servers');

    $response->assertStatus(200);
}
```
