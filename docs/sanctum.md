# Laravel Sanctum

- [Wprowadzenie](#introduction)
    - [Jak to działa](#how-it-works)
- [Instalacja](#installation)
- [Konfiguracja](#configuration)
    - [Nadpisywanie domyślnych modeli](#overriding-default-models)
- [Uwierzytelnianie tokenami API](#api-token-authentication)
    - [Wydawanie tokenów API](#issuing-api-tokens)
    - [Uprawnienia tokenów](#token-abilities)
    - [Zabezpieczanie tras](#protecting-routes)
    - [Cofanie tokenów](#revoking-tokens)
    - [Wygasanie tokenów](#token-expiration)
- [Uwierzytelnianie SPA](#spa-authentication)
    - [Konfiguracja](#spa-configuration)
    - [Uwierzytelnianie](#spa-authenticating)
    - [Zabezpieczanie tras](#protecting-spa-routes)
    - [Autoryzacja prywatnych kanałów rozgłoszeniowych](#authorizing-private-broadcast-channels)
- [Uwierzytelnianie aplikacji mobilnych](#mobile-application-authentication)
    - [Wydawanie tokenów API](#issuing-mobile-api-tokens)
    - [Zabezpieczanie tras](#protecting-mobile-api-routes)
    - [Cofanie tokenów](#revoking-mobile-api-tokens)
- [Testowanie](#testing)

<a name="introduction"></a>
## Wprowadzenie

[Laravel Sanctum](https://github.com/laravel/sanctum) zapewnia lekki system uwierzytelniania dla aplikacji SPA (aplikacje jednostronicowe), aplikacji mobilnych oraz prostych interfejsów API opartych na tokenach. Sanctum pozwala każdemu użytkownikowi Twojej aplikacji generować wiele tokenów API dla swojego konta. Tokeny te mogą otrzymać uprawnienia/zakresy, które określają, jakie akcje tokeny mogą wykonywać.

<a name="how-it-works"></a>
### Jak to działa

Laravel Sanctum istnieje, aby rozwiązać dwa odrębne problemy. Omówmy każdy z nich, zanim zagłębimy się w bibliotekę.

<a name="how-it-works-api-tokens"></a>
#### Tokeny API

Po pierwsze, Sanctum to prosty pakiet, którego możesz użyć do wydawania tokenów API swoim użytkownikom bez komplikacji OAuth. Ta funkcja jest inspirowana przez GitHub i inne aplikacje, które wydają "tokeny dostępu osobistego". Na przykład, wyobraź sobie, że "ustawienia konta" Twojej aplikacji mają ekran, na którym użytkownik może wygenerować token API dla swojego konta. Możesz użyć Sanctum do generowania i zarządzania tymi tokenami. Te tokeny zazwyczaj mają bardzo długi czas wygaśnięcia (lata), ale mogą być ręcznie cofnięte przez użytkownika w dowolnym momencie.

Laravel Sanctum oferuje tę funkcję, przechowując tokeny API użytkowników w pojedynczej tabeli bazy danych i uwierzytelniając przychodzące żądania HTTP za pomocą nagłówka `Authorization`, który powinien zawierać prawidłowy token API.

<a name="how-it-works-spa-authentication"></a>
#### Uwierzytelnianie SPA

Po drugie, Sanctum istnieje, aby zaoferować prosty sposób uwierzytelniania aplikacji jednostronicowych (SPA), które muszą komunikować się z API zasilanym przez Laravel. Te aplikacje SPA mogą istnieć w tym samym repozytorium co Twoja aplikacja Laravel lub mogą być całkowicie oddzielnym repozytorium, takim jak SPA utworzone przy użyciu Next.js lub Nuxt.

Dla tej funkcji Sanctum nie używa tokenów żadnego rodzaju. Zamiast tego Sanctum używa wbudowanych w Laravel usług uwierzytelniania sesji opartych na plikach cookie. Zazwyczaj Sanctum wykorzystuje guard uwierzytelniania `web` Laravel, aby to osiągnąć. Zapewnia to korzyści w postaci ochrony CSRF, uwierzytelniania sesji, a także chroni przed wyciekiem danych uwierzytelniających przez XSS.

Sanctum podejmie próbę uwierzytelnienia przy użyciu plików cookie tylko wtedy, gdy przychodzące żądanie pochodzi z Twojego własnego frontendu SPA. Gdy Sanctum bada przychodzące żądanie HTTP, najpierw sprawdzi obecność pliku cookie uwierzytelniającego, a jeśli go nie ma, Sanctum sprawdzi nagłówek `Authorization` w poszukiwaniu prawidłowego tokenu API.

> [!NOTE]
> Całkowicie w porządku jest używanie Sanctum tylko do uwierzytelniania tokenami API lub tylko do uwierzytelniania SPA. To, że używasz Sanctum, nie oznacza, że musisz używać obu funkcji, które oferuje.

<a name="installation"></a>
## Instalacja

Możesz zainstalować Laravel Sanctum za pomocą polecenia Artisan `install:api`:

```shell
php artisan install:api
```

Następnie, jeśli planujesz wykorzystać Sanctum do uwierzytelniania SPA, zapoznaj się z sekcją [Uwierzytelnianie SPA](#spa-authentication) tej dokumentacji.

<a name="configuration"></a>
## Konfiguracja

<a name="overriding-default-models"></a>
### Nadpisywanie domyślnych modeli

Chociaż zazwyczaj nie jest to wymagane, możesz rozszerzyć model `PersonalAccessToken` używany wewnętrznie przez Sanctum:

```php
use Laravel\Sanctum\PersonalAccessToken as SanctumPersonalAccessToken;

class PersonalAccessToken extends SanctumPersonalAccessToken
{
    // ...
}
```

Następnie możesz poinstruować Sanctum, aby używał Twojego niestandardowego modelu za pomocą metody `usePersonalAccessTokenModel` dostarczonej przez Sanctum. Zazwyczaj powinieneś wywołać tę metodę w metodzie `boot` pliku `AppServiceProvider` Twojej aplikacji:

```php
use App\Models\Sanctum\PersonalAccessToken;
use Laravel\Sanctum\Sanctum;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Sanctum::usePersonalAccessTokenModel(PersonalAccessToken::class);
}
```

<a name="api-token-authentication"></a>
## Uwierzytelnianie tokenami API

> [!NOTE]
> [!NOTE]
> Nie powinieneś używać tokenów API do uwierzytelniania swojej własnej aplikacji SPA pierwszej strony. Zamiast tego użyj wbudowanych funkcji [uwierzytelniania SPA](#spa-authentication) Sanctum.

<a name="issuing-api-tokens"></a>
### Wydawanie tokenów API

Sanctum pozwala wydawać tokeny API / tokeny dostępu osobistego, które mogą być używane do uwierzytelniania żądań API do Twojej aplikacji. Podczas składania żądań przy użyciu tokenów API, token powinien być zawarty w nagłówku `Authorization` jako token `Bearer`.

Aby rozpocząć wydawanie tokenów dla użytkowników, Twój model User powinien używać traita `Laravel\Sanctum\HasApiTokens`:

```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

Aby wydać token, możesz użyć metody `createToken`. Metoda `createToken` zwraca instancję `Laravel\Sanctum\NewAccessToken`. Tokeny API są hashowane przy użyciu hashowania SHA-256 przed zapisaniem w bazie danych, ale możesz uzyskać dostęp do wartości zwykłego tekstu tokenu za pomocą właściwości `plainTextToken` instancji `NewAccessToken`. Powinieneś wyświetlić tę wartość użytkownikowi natychmiast po utworzeniu tokenu:

```php
use Illuminate\Http\Request;

Route::post('/tokens/create', function (Request $request) {
    $token = $request->user()->createToken($request->token_name);

    return ['token' => $token->plainTextToken];
});
```

Możesz uzyskać dostęp do wszystkich tokenów użytkownika za pomocą relacji Eloquent `tokens` dostarczonej przez trait `HasApiTokens`:

```php
foreach ($user->tokens as $token) {
    // ...
}
```

<a name="token-abilities"></a>
### Uprawnienia tokenów

Sanctum pozwala przypisywać "uprawnienia" do tokenów. Uprawnienia służą podobnemu celowi co "zakresy" OAuth. Możesz przekazać tablicę stringowych uprawnień jako drugi argument do metody `createToken`:

```php
return $user->createToken('token-name', ['server:update'])->plainTextToken;
```

Podczas obsługi przychodzącego żądania uwierzytelnionego przez Sanctum, możesz określić, czy token ma dane uprawnienie, używając metod `tokenCan` lub `tokenCant`:

```php
if ($user->tokenCan('server:update')) {
    // ...
}

if ($user->tokenCant('server:update')) {
    // ...
}
```

<a name="token-ability-middleware"></a>
#### Middleware uprawnień tokenów

Sanctum zawiera również dwa middleware, które mogą być używane do weryfikacji, czy przychodzące żądanie jest uwierzytelnione tokenem, któremu przyznano dane uprawnienie. Aby rozpocząć, zdefiniuj następujące aliasy middleware w pliku `bootstrap/app.php` swojej aplikacji:

```php
use Laravel\Sanctum\Http\Middleware\CheckAbilities;
use Laravel\Sanctum\Http\Middleware\CheckForAnyAbility;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'abilities' => CheckAbilities::class,
        'ability' => CheckForAnyAbility::class,
    ]);
})
```

Middleware `abilities` może być przypisany do trasy, aby zweryfikować, że token przychodzącego żądania ma wszystkie wymienione uprawnienia:

```php
Route::get('/orders', function () {
    // Token has both "check-status" and "place-orders" abilities...
})->middleware(['auth:sanctum', 'abilities:check-status,place-orders']);
```

Middleware `ability` może być przypisany do trasy, aby zweryfikować, że token przychodzącego żądania ma *przynajmniej jedno* z wymienionych uprawnień:

```php
Route::get('/orders', function () {
    // Token has the "check-status" or "place-orders" ability...
})->middleware(['auth:sanctum', 'ability:check-status,place-orders']);
```

<a name="first-party-ui-initiated-requests"></a>
#### Żądania zainicjowane przez interfejs użytkownika pierwszej strony

Dla wygody, metoda `tokenCan` zawsze zwróci `true`, jeśli przychodzące uwierzytelnione żądanie pochodzi z Twojej aplikacji SPA pierwszej strony i używasz wbudowanego [uwierzytelniania SPA](#spa-authentication) Sanctum.

Jednak nie oznacza to koniecznie, że Twoja aplikacja musi pozwolić użytkownikowi wykonać akcję. Zazwyczaj [polityki autoryzacji](/docs/{{version}}/authorization#creating-policies) Twojej aplikacji określą, czy tokenowi przyznano uprawnienie do wykonywania uprawnień, a także sprawdzą, czy sama instancja użytkownika powinna móc wykonać akcję.

Na przykład, jeśli wyobrazimy sobie aplikację zarządzającą serwerami, może to oznaczać sprawdzenie, czy token jest autoryzowany do aktualizacji serwerów **i** że serwer należy do użytkownika:

```php
return $request->user()->id === $server->user_id &&
       $request->user()->tokenCan('server:update')
```

Na początku, pozwolenie na wywołanie metody `tokenCan` i zawsze zwracanie `true` dla żądań zainicjowanych przez interfejs użytkownika pierwszej strony może wydawać się dziwne; jednak jest to wygodne, aby móc zawsze założyć, że token API jest dostępny i może być sprawdzony za pomocą metody `tokenCan`. Przyjmując takie podejście, możesz zawsze wywołać metodę `tokenCan` w politykach autoryzacji swojej aplikacji bez martwienia się, czy żądanie zostało wyzwolone z interfejsu użytkownika Twojej aplikacji, czy zostało zainicjowane przez jednego z zewnętrznych konsumentów Twojego API.

<a name="protecting-routes"></a>
### Zabezpieczanie tras

Aby zabezpieczyć trasy tak, aby wszystkie przychodzące żądania musiały być uwierzytelnione, powinieneś dołączyć guard uwierzytelniania `sanctum` do swoich chronionych tras w plikach tras `routes/web.php` i `routes/api.php`. Ten guard zapewni, że przychodzące żądania są uwierzytelnione jako stanowe żądania uwierzytelnione przez pliki cookie lub zawierają prawidłowy nagłówek tokenu API, jeśli żądanie pochodzi od strony trzeciej.

Możesz się zastanawiać, dlaczego sugerujemy uwierzytelnianie tras w pliku `routes/web.php` Twojej aplikacji za pomocą guarda `sanctum`. Pamiętaj, że Sanctum najpierw spróbuje uwierzytelnić przychodzące żądania za pomocą typowego pliku cookie uwierzytelniającego sesję Laravel. Jeśli tego pliku cookie nie ma, Sanctum spróbuje uwierzytelnić żądanie za pomocą tokenu w nagłówku `Authorization` żądania. Ponadto uwierzytelnianie wszystkich żądań za pomocą Sanctum zapewnia, że zawsze możemy wywołać metodę `tokenCan` na obecnie uwierzytelnionej instancji użytkownika:

```php
use Illuminate\Http\Request;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

<a name="revoking-tokens"></a>
### Cofanie tokenów

Możesz "cofnąć" tokeny, usuwając je z bazy danych za pomocą relacji `tokens` dostarczonej przez trait `Laravel\Sanctum\HasApiTokens`:

```php
// Revoke all tokens...
$user->tokens()->delete();

// Revoke the token that was used to authenticate the current request...
$request->user()->currentAccessToken()->delete();

// Revoke a specific token...
$user->tokens()->where('id', $tokenId)->delete();
```

<a name="token-expiration"></a>
### Wygasanie tokenów

Domyślnie tokeny Sanctum nigdy nie wygasają i mogą być unieważnione tylko przez [cofnięcie tokenu](#revoking-tokens). Jednak jeśli chcesz skonfigurować czas wygaśnięcia tokenów API swojej aplikacji, możesz to zrobić za pomocą opcji konfiguracji `expiration` zdefiniowanej w pliku konfiguracyjnym `sanctum` Twojej aplikacji. Ta opcja konfiguracji definiuje liczbę minut, po której wydany token zostanie uznany za wygasły:

```php
'expiration' => 525600,
```

Jeśli chcesz określić czas wygaśnięcia każdego tokenu niezależnie, możesz to zrobić, podając czas wygaśnięcia jako trzeci argument metody `createToken`:

```php
return $user->createToken(
    'token-name', ['*'], now()->plus(weeks: 1)
)->plainTextToken;
```

Jeśli skonfigurowałeś czas wygaśnięcia tokenu dla swojej aplikacji, możesz również [zaplanować zadanie](/docs/{{version}}/scheduling) aby wyczyścić wygasłe tokeny Twojej aplikacji. Na szczęście Sanctum zawiera polecenie Artisan `sanctum:prune-expired`, które możesz użyć, aby to osiągnąć. Na przykład możesz skonfigurować zaplanowane zadanie, aby usunąć wszystkie wygasłe rekordy tokenów z bazy danych, które wygasły przynajmniej 24 godziny temu:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('sanctum:prune-expired --hours=24')->daily();
```

<a name="spa-authentication"></a>
## Uwierzytelnianie SPA

Sanctum istnieje również, aby zapewnić prostą metodę uwierzytelniania aplikacji jednostronicowych (SPA), które muszą komunikować się z API zasilanym przez Laravel. Te aplikacje SPA mogą istnieć w tym samym repozytorium co Twoja aplikacja Laravel lub mogą być całkowicie oddzielnym repozytorium.

Dla tej funkcji Sanctum nie używa tokenów żadnego rodzaju. Zamiast tego Sanctum używa wbudowanych w Laravel usług uwierzytelniania sesji opartych na plikach cookie. To podejście do uwierzytelniania zapewnia korzyści w postaci ochrony CSRF, uwierzytelniania sesji, a także chroni przed wyciekiem danych uwierzytelniających przez XSS.

> [!WARNING]
> Aby się uwierzytelnić, Twoja SPA i API muszą współdzielić tę samą domenę najwyższego poziomu. Mogą jednak być umieszczone w różnych subdomenach. Dodatkowo powinieneś upewnić się, że wysyłasz nagłówek `Accept: application/json` oraz nagłówek `Referer` lub `Origin` z Twoim żądaniem.

<a name="spa-configuration"></a>
### Konfiguracja

<a name="configuring-your-first-party-domains"></a>
#### Konfigurowanie domen pierwszej strony

Po pierwsze, powinieneś skonfigurować domeny, z których Twoja SPA będzie składać żądania. Możesz skonfigurować te domeny za pomocą opcji konfiguracji `stateful` w pliku konfiguracyjnym `sanctum`. To ustawienie konfiguracji określa, które domeny będą utrzymywać uwierzytelnianie "stanowe" przy użyciu plików cookie sesji Laravel podczas składania żądań do Twojego API.

Aby pomóc Ci w skonfigurowaniu stanowych domen pierwszej strony, Sanctum dostarcza dwie funkcje pomocnicze, które możesz umieścić w konfiguracji. Po pierwsze, `Sanctum::currentApplicationUrlWithPort()` zwróci bieżący URL aplikacji ze zmiennej środowiskowej `APP_URL`, a `Sanctum::currentRequestHost()` wstawi symbol zastępczy do listy stanowych domen, który w czasie wykonywania zostanie zastąpiony hostem z bieżącego żądania, tak aby wszystkie żądania z tej samej domeny były uznawane za stanowe.

> [!WARNING]
> Jeśli uzyskujesz dostęp do swojej aplikacji za pomocą URL zawierającego port (`127.0.0.1:8000`), powinieneś upewnić się, że uwzględniasz numer portu z domeną.

<a name="sanctum-middleware"></a>
#### Middleware Sanctum

Następnie powinieneś poinstruować Laravel, że przychodzące żądania z Twojej SPA mogą uwierzytelniać się za pomocą plików cookie sesji Laravel, jednocześnie pozwalając na uwierzytelnianie żądań od stron trzecich lub aplikacji mobilnych za pomocą tokenów API. Można to łatwo osiągnąć, wywołując metodę middleware `statefulApi` w pliku `bootstrap/app.php` Twojej aplikacji:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->statefulApi();
})
```

<a name="cors-and-cookies"></a>
#### CORS i pliki cookie

Jeśli masz problemy z uwierzytelnianiem swojej aplikacji z SPA, która wykonuje się w oddzielnej subdomenie, prawdopodobnie błędnie skonfigurowałeś ustawienia CORS (Cross-Origin Resource Sharing) lub plików cookie sesji.

Plik konfiguracyjny `config/cors.php` nie jest publikowany domyślnie. Jeśli musisz dostosować opcje CORS Laravel, powinieneś opublikować kompletny plik konfiguracyjny `cors` za pomocą polecenia Artisan `config:publish`:

```shell
php artisan config:publish cors
```

Następnie powinieneś upewnić się, że konfiguracja CORS Twojej aplikacji zwraca nagłówek `Access-Control-Allow-Credentials` z wartością `True`. Można to osiągnąć, ustawiając opcję `supports_credentials` w pliku konfiguracyjnym `config/cors.php` Twojej aplikacji na `true`.

Ponadto powinieneś włączyć opcje `withCredentials` i `withXSRFToken` w globalnej instancji `axios` Twojej aplikacji. Zazwyczaj powinno to być wykonane w pliku `resources/js/bootstrap.js`. Jeśli nie używasz Axios do wykonywania żądań HTTP z Twojego frontendu, powinieneś wykonać równoważną konfigurację na swoim własnym kliencie HTTP:

```js
axios.defaults.withCredentials = true;
axios.defaults.withXSRFToken = true;
```

Na koniec powinieneś upewnić się, że konfiguracja domeny pliku cookie sesji Twojej aplikacji obsługuje dowolną subdomenę Twojej domeny głównej. Możesz to osiągnąć, dodając wiodącą `.` przed domeną w pliku konfiguracyjnym `config/session.php` Twojej aplikacji:

```php
'domain' => '.domain.com',
```

<a name="spa-authenticating"></a>
### Uwierzytelnianie

<a name="csrf-protection"></a>
#### Ochrona CSRF

Aby uwierzytelnić swoją SPA, strona "logowania" Twojej SPA powinna najpierw wysłać żądanie do punktu końcowego `/sanctum/csrf-cookie`, aby zainicjować ochronę CSRF dla aplikacji:

```js
axios.get('/sanctum/csrf-cookie').then(response => {
    // Login...
});
```

Podczas tego żądania Laravel ustawi plik cookie `XSRF-TOKEN` zawierający bieżący token CSRF. Token ten powinien następnie zostać zdekodowany z URL i przekazany w nagłówku `X-XSRF-TOKEN` w kolejnych żądaniach, co niektóre biblioteki klientów HTTP, takie jak Axios i Angular HttpClient, zrobią automatycznie. Jeśli Twoja biblioteka HTTP JavaScript nie ustawia wartości automatycznie, będziesz musiał ręcznie ustawić nagłówek `X-XSRF-TOKEN` tak, aby pasował do wartości zdekodowanej z URL pliku cookie `XSRF-TOKEN`, który jest ustawiany przez tę trasę.

<a name="logging-in"></a>
#### Logowanie

Po zainicjowaniu ochrony CSRF powinieneś wysłać żądanie `POST` do trasy `/login` Twojej aplikacji Laravel. Ta trasa `/login` może być [zaimplementowana ręcznie](/docs/{{version}}/authentication#authenticating-users) lub przy użyciu bezgłowego pakietu uwierzytelniającego, takiego jak [Laravel Fortify](/docs/{{version}}/fortify).

Jeśli żądanie logowania się powiedzie, zostaniesz uwierzytelniony, a kolejne żądania do tras Twojej aplikacji będą automatycznie uwierzytelniane za pomocą pliku cookie sesji, który aplikacja Laravel wydała Twojemu klientowi. Ponadto, ponieważ Twoja aplikacja już wysłała żądanie do trasy `/sanctum/csrf-cookie`, kolejne żądania powinny automatycznie otrzymać ochronę CSRF, o ile Twój klient HTTP JavaScript wysyła wartość pliku cookie `XSRF-TOKEN` w nagłówku `X-XSRF-TOKEN`.

Oczywiście, jeśli sesja Twojego użytkownika wygaśnie z powodu braku aktywności, kolejne żądania do aplikacji Laravel mogą otrzymać odpowiedź błędu HTTP 401 lub 419. W tym przypadku powinieneś przekierować użytkownika na stronę logowania Twojej SPA.

> [!WARNING]
> Możesz napisać własny punkt końcowy `/login`; jednak powinieneś upewnić się, że uwierzytelnia on użytkownika przy użyciu standardowych [usług uwierzytelniania opartych na sesji, które dostarcza Laravel](/docs/{{version}}/authentication#authenticating-users). Zazwyczaj oznacza to użycie guarda uwierzytelniania `web`.

<a name="protecting-spa-routes"></a>
### Zabezpieczanie tras

Aby zabezpieczyć trasy tak, aby wszystkie przychodzące żądania musiały być uwierzytelnione, powinieneś dołączyć guard uwierzytelniania `sanctum` do swoich tras API w pliku `routes/api.php`. Ten guard zapewni, że przychodzące żądania są uwierzytelnione jako stanowe uwierzytelnione żądania z Twojej SPA lub zawierają prawidłowy nagłówek tokenu API, jeśli żądanie pochodzi od strony trzeciej:

```php
use Illuminate\Http\Request;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

<a name="authorizing-private-broadcast-channels"></a>
### Autoryzacja prywatnych kanałów rozgłoszeniowych

Jeśli Twoja SPA musi uwierzytelniać się z [prywatnymi / obecnościowymi kanałami rozgłoszeniowymi](/docs/{{version}}/broadcasting#authorizing-channels), powinieneś usunąć wpis `channels` z metody `withRouting` zawartej w pliku `bootstrap/app.php` Twojej aplikacji. Zamiast tego powinieneś wywołać metodę `withBroadcasting`, aby móc określić odpowiedni middleware dla tras rozgłoszeniowych Twojej aplikacji:

```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        // ...
    )
    ->withBroadcasting(
        __DIR__.'/../routes/channels.php',
        ['prefix' => 'api', 'middleware' => ['api', 'auth:sanctum']],
    )
```

Następnie, aby żądania autoryzacji Pusher się powiodły, będziesz musiał dostarczyć niestandardowy `authorizer` Pusher podczas inicjowania [Laravel Echo](/docs/{{version}}/broadcasting#client-side-installation). Pozwala to Twojej aplikacji skonfigurować Pusher do używania instancji `axios`, która jest [prawidłowo skonfigurowana dla żądań międzydomenowych](#cors-and-cookies):

```js
window.Echo = new Echo({
    broadcaster: "pusher",
    cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    encrypted: true,
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    authorizer: (channel, options) => {
        return {
            authorize: (socketId, callback) => {
                axios.post('/api/broadcasting/auth', {
                    socket_id: socketId,
                    channel_name: channel.name
                })
                .then(response => {
                    callback(false, response.data);
                })
                .catch(error => {
                    callback(true, error);
                });
            }
        };
    },
})
```

<a name="mobile-application-authentication"></a>
## Uwierzytelnianie aplikacji mobilnych

Możesz również użyć tokenów Sanctum do uwierzytelniania żądań Twojej aplikacji mobilnej do Twojego API. Proces uwierzytelniania żądań aplikacji mobilnej jest podobny do uwierzytelniania żądań API stron trzecich; jednak istnieją małe różnice w sposobie wydawania tokenów API.

<a name="issuing-mobile-api-tokens"></a>
### Wydawanie tokenów API

Aby rozpocząć, utwórz trasę, która akceptuje email / nazwę użytkownika, hasło i nazwę urządzenia użytkownika, a następnie wymienia te dane uwierzytelniające na nowy token Sanctum. "Nazwa urządzenia" przekazana do tego punktu końcowego służy celom informacyjnym i może być dowolną wartością. Ogólnie rzecz biorąc, wartość nazwy urządzenia powinna być nazwą, którą użytkownik rozpozna, taką jak "iPhone 12 Nuno".

Zazwyczaj wyślesz żądanie do punktu końcowego tokenu z ekranu "logowania" Twojej aplikacji mobilnej. Punkt końcowy zwróci token API w postaci zwykłego tekstu, który może następnie zostać zapisany na urządzeniu mobilnym i używany do wysyłania dodatkowych żądań API:

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

Route::post('/sanctum/token', function (Request $request) {
    $request->validate([
        'email' => 'required|email',
        'password' => 'required',
        'device_name' => 'required',
    ]);

    $user = User::where('email', $request->email)->first();

    if (! $user || ! Hash::check($request->password, $user->password)) {
        throw ValidationException::withMessages([
            'email' => ['The provided credentials are incorrect.'],
        ]);
    }

    return $user->createToken($request->device_name)->plainTextToken;
});
```

Gdy aplikacja mobilna używa tokenu do wysłania żądania API do Twojej aplikacji, powinna przekazać token w nagłówku `Authorization` jako token `Bearer`.

> [!NOTE]
> Podczas wydawania tokenów dla aplikacji mobilnej możesz również określić [uprawnienia tokenów](#token-abilities).

<a name="protecting-mobile-api-routes"></a>
### Zabezpieczanie tras

Jak wcześniej udokumentowano, możesz zabezpieczyć trasy tak, aby wszystkie przychodzące żądania musiały być uwierzytelnione, dołączając guard uwierzytelniania `sanctum` do tras:

```php
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

<a name="revoking-mobile-api-tokens"></a>
### Cofanie tokenów

Aby umożliwić użytkownikom cofanie tokenów API wydanych urządzeniom mobilnym, możesz wyświetlić je według nazwy wraz z przyciskiem "Cofnij" w sekcji "ustawienia konta" interfejsu użytkownika Twojej aplikacji webowej. Gdy użytkownik kliknie przycisk "Cofnij", możesz usunąć token z bazy danych. Pamiętaj, że możesz uzyskać dostęp do tokenów API użytkownika za pomocą relacji `tokens` dostarczonej przez trait `Laravel\Sanctum\HasApiTokens`:

```php
// Revoke all tokens...
$user->tokens()->delete();

// Revoke a specific token...
$user->tokens()->where('id', $tokenId)->delete();
```

<a name="testing"></a>
## Testowanie

Podczas testowania metoda `Sanctum::actingAs` może być używana do uwierzytelnienia użytkownika i określenia, jakie uprawnienia powinny zostać przyznane jego tokenowi:

```php tab=Pest
use App\Models\User;
use Laravel\Sanctum\Sanctum;

test('task list can be retrieved', function () {
    Sanctum::actingAs(
        User::factory()->create(),
        ['view-tasks']
    );

    $response = $this->get('/api/task');

    $response->assertOk();
});
```

```php tab=PHPUnit
use App\Models\User;
use Laravel\Sanctum\Sanctum;

public function test_task_list_can_be_retrieved(): void
{
    Sanctum::actingAs(
        User::factory()->create(),
        ['view-tasks']
    );

    $response = $this->get('/api/task');

    $response->assertOk();
}
```

Jeśli chcesz przyznać wszystkie uprawnienia tokenowi, powinieneś uwzględnić `*` na liście uprawnień przekazanej do metody `actingAs`:

```php
Sanctum::actingAs(
    User::factory()->create(),
    ['*']
);
```
