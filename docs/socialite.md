# Laravel Socialite

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
- [Aktualizacja Socialite](#upgrading-socialite)
- [Konfiguracja](#configuration)
- [Uwierzytelnianie](#authentication)
    - [Trasowanie](#routing)
    - [Uwierzytelnianie i przechowywanie](#authentication-and-storage)
    - [Zakresy dostępu](#access-scopes)
    - [Zakresy botów Slack](#slack-bot-scopes)
    - [Parametry opcjonalne](#optional-parameters)
- [Pobieranie szczegółów użytkownika](#retrieving-user-details)
- [Testowanie](#testing)

<a name="introduction"></a>
## Wprowadzenie

Oprócz typowego uwierzytelniania opartego na formularzach, Laravel zapewnia również prosty i wygodny sposób uwierzytelniania za pomocą dostawców OAuth przy użyciu [Laravel Socialite](https://github.com/laravel/socialite). Socialite aktualnie obsługuje uwierzytelnianie przez Facebook, X, LinkedIn, Google, GitHub, GitLab, Bitbucket i Slack.

> [!NOTE]
> Adaptery dla innych platform są dostępne poprzez stronę [Socialite Providers](https://socialiteproviders.com/) napędzaną przez społeczność.

<a name="installation"></a>
## Instalacja

Aby rozpocząć pracę z Socialite, użyj menedżera pakietów Composer, aby dodać pakiet do zależności Twojego projektu:

```shell
composer require laravel/socialite
```

<a name="upgrading-socialite"></a>
## Aktualizacja Socialite

Podczas aktualizacji do nowej wersji głównej Socialite, ważne jest, aby dokładnie przejrzeć [przewodnik aktualizacji](https://github.com/laravel/socialite/blob/master/UPGRADE.md).

<a name="configuration"></a>
## Konfiguracja

Przed użyciem Socialite, musisz dodać dane uwierzytelniające dla dostawców OAuth wykorzystywanych przez Twoją aplikację. Zazwyczaj te dane uwierzytelniające mogą być pobrane poprzez utworzenie "aplikacji dewelopera" w panelu usługi, z którą będziesz się uwierzytelniać.

Te dane uwierzytelniające powinny zostać umieszczone w pliku konfiguracyjnym Twojej aplikacji `config/services.php` i powinny używać klucza `facebook`, `x`, `linkedin-openid`, `google`, `github`, `gitlab`, `bitbucket`, `slack` lub `slack-openid`, w zależności od dostawców wymaganych przez Twoją aplikację:

```php
'github' => [
    'client_id' => env('GITHUB_CLIENT_ID'),
    'client_secret' => env('GITHUB_CLIENT_SECRET'),
    'redirect' => 'http://example.com/callback-url',
],
```

> [!NOTE]
> Jeśli opcja `redirect` zawiera ścieżkę względną, zostanie automatycznie rozwiązana do w pełni kwalifikowanego adresu URL.

<a name="authentication"></a>
## Uwierzytelnianie

<a name="routing"></a>
### Trasowanie

Aby uwierzytelnić użytkowników za pomocą dostawcy OAuth, będziesz potrzebować dwóch tras: jednej do przekierowania użytkownika do dostawcy OAuth i drugiej do odbierania callbacku od dostawcy po uwierzytelnieniu. Poniższe przykładowe trasy demonstrują implementację obu tras:

```php
use Laravel\Socialite\Socialite;

Route::get('/auth/redirect', function () {
    return Socialite::driver('github')->redirect();
});

Route::get('/auth/callback', function () {
    $user = Socialite::driver('github')->user();

    // $user->token
});
```

Metoda `redirect` dostarczona przez fasadę `Socialite` zajmuje się przekierowaniem użytkownika do dostawcy OAuth, podczas gdy metoda `user` zbada przychodzące żądanie i pobierze informacje o użytkowniku od dostawcy po zatwierdzeniu żądania uwierzytelnienia.

<a name="authentication-and-storage"></a>
### Uwierzytelnianie i przechowywanie

Po pobraniu użytkownika od dostawcy OAuth, możesz określić, czy użytkownik istnieje w bazie danych Twojej aplikacji i [uwierzytelnić użytkownika](/docs/{{version}}/authentication#authenticate-a-user-instance). Jeśli użytkownik nie istnieje w bazie danych Twojej aplikacji, zazwyczaj utworzysz nowy rekord w bazie danych, aby reprezentować użytkownika:

```php
use App\Models\User;
use Illuminate\Support\Facades\Auth;
use Laravel\Socialite\Socialite;

Route::get('/auth/callback', function () {
    $githubUser = Socialite::driver('github')->user();

    $user = User::updateOrCreate([
        'github_id' => $githubUser->id,
    ], [
        'name' => $githubUser->name,
        'email' => $githubUser->email,
        'github_token' => $githubUser->token,
        'github_refresh_token' => $githubUser->refreshToken,
    ]);

    Auth::login($user);

    return redirect('/dashboard');
});
```

> [!NOTE]
> Aby uzyskać więcej informacji na temat dostępnych informacji o użytkowniku od konkretnych dostawców OAuth, zapoznaj się z dokumentacją na temat [pobierania szczegółów użytkownika](#retrieving-user-details).

<a name="access-scopes"></a>
### Zakresy dostępu

Przed przekierowaniem użytkownika, możesz użyć metody `scopes`, aby określić "zakresy", które powinny być uwzględnione w żądaniu uwierzytelnienia. Ta metoda połączy wszystkie wcześniej określone zakresy z zakresami, które określasz:

```php
use Laravel\Socialite\Socialite;

return Socialite::driver('github')
    ->scopes(['read:user', 'public_repo'])
    ->redirect();
```

Możesz nadpisać wszystkie istniejące zakresy w żądaniu uwierzytelnienia, używając metody `setScopes`:

```php
return Socialite::driver('github')
    ->setScopes(['read:user', 'public_repo'])
    ->redirect();
```

<a name="slack-bot-scopes"></a>
### Zakresy botów Slack

API Slack udostępnia [różne typy tokenów dostępu](https://api.slack.com/authentication/token-types), każdy z własnym zestawem [zakresów uprawnień](https://api.slack.com/scopes). Socialite jest kompatybilny z oboma następującymi typami tokenów dostępu Slack:

<div class="content-list" markdown="1">

- Bot (z prefiksem `xoxb-`)
- User (z prefiksem `xoxp-`)

</div>

Domyślnie sterownik `slack` wygeneruje token `user`, a wywołanie metody `user` sterownika zwróci szczegóły użytkownika.

Tokeny botów są głównie przydatne, jeśli Twoja aplikacja będzie wysyłać powiadomienia do zewnętrznych przestrzeni roboczych Slack, które należą do użytkowników Twojej aplikacji. Aby wygenerować token bota, wywołaj metodę `asBotUser` przed przekierowaniem użytkownika do Slack w celu uwierzytelnienia:

```php
return Socialite::driver('slack')
    ->asBotUser()
    ->setScopes(['chat:write', 'chat:write.public', 'chat:write.customize'])
    ->redirect();
```

Ponadto musisz wywołać metodę `asBotUser` przed wywołaniem metody `user` po tym, jak Slack przekieruje użytkownika z powrotem do Twojej aplikacji po uwierzytelnieniu:

```php
$user = Socialite::driver('slack')->asBotUser()->user();
```

Podczas generowania tokenu bota, metoda `user` nadal zwróci instancję `Laravel\Socialite\Two\User`; jednak tylko właściwość `token` zostanie wypełniona. Ten token może zostać zapisany w celu [wysyłania powiadomień do przestrzeni roboczych Slack uwierzytelnionego użytkownika](/docs/{{version}}/notifications#notifying-external-slack-workspaces).

<a name="optional-parameters"></a>
### Parametry opcjonalne

Wiele dostawców OAuth obsługuje inne opcjonalne parametry w żądaniu przekierowania. Aby uwzględnić dowolne opcjonalne parametry w żądaniu, wywołaj metodę `with` z tablicą asocjacyjną:

```php
use Laravel\Socialite\Socialite;

return Socialite::driver('google')
    ->with(['hd' => 'example.com'])
    ->redirect();
```

> [!WARNING]
> Podczas używania metody `with`, uważaj, aby nie przekazywać żadnych zarezerwowanych słów kluczowych, takich jak `state` lub `response_type`.

<a name="retrieving-user-details"></a>
## Pobieranie szczegółów użytkownika

Po przekierowaniu użytkownika z powrotem do trasy callbacku uwierzytelniania Twojej aplikacji, możesz pobrać szczegóły użytkownika, używając metody `user` Socialite. Obiekt użytkownika zwraca przez metodę `user` zapewnia różne właściwości i metody, których możesz użyć do przechowywania informacji o użytkowniku we własnej bazie danych.

Różne właściwości i metody mogą być dostępne dla tego obiektu w zależności od tego, czy dostawca OAuth, z którym się uwierzytelniasz, obsługuje OAuth 1.0 czy OAuth 2.0:

```php
use Laravel\Socialite\Socialite;

Route::get('/auth/callback', function () {
    $user = Socialite::driver('github')->user();

    // OAuth 2.0 providers...
    $token = $user->token;
    $refreshToken = $user->refreshToken;
    $expiresIn = $user->expiresIn;

    // OAuth 1.0 providers...
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // All providers...
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();
});
```

<a name="retrieving-user-details-from-a-token-oauth2"></a>
#### Pobieranie szczegółów użytkownika z tokenu

Jeśli masz już ważny token dostępu dla użytkownika, możesz pobrać jego szczegóły używając metody `userFromToken` Socialite:

```php
use Laravel\Socialite\Socialite;

$user = Socialite::driver('github')->userFromToken($token);
```

Jeśli używasz Facebook Limited Login przez aplikację iOS, Facebook zwróci token OIDC zamiast tokenu dostępu. Podobnie jak token dostępu, token OIDC może zostać przekazany do metody `userFromToken` w celu pobrania szczegółów użytkownika.

<a name="stateless-authentication"></a>
#### Uwierzytelnianie bezstanowe

Metoda `stateless` może być użyta do wyłączenia weryfikacji stanu sesji. Jest to przydatne podczas dodawania uwierzytelniania społecznościowego do bezstanowego API, które nie wykorzystuje sesji opartych na ciasteczkach:

```php
use Laravel\Socialite\Socialite;

return Socialite::driver('google')->stateless()->user();
```

<a name="testing"></a>
## Testowanie

Laravel Socialite zapewnia wygodny sposób testowania przepływów uwierzytelniania OAuth bez wykonywania faktycznych żądań do dostawców OAuth. Metoda `fake` pozwala na mockowanie zachowania dostawcy OAuth i definiowanie danych użytkownika, które powinny zostać zwrócone.

<a name="faking-the-redirect"></a>
#### Mockowanie przekierowania

Aby przetestować, że Twoja aplikacja poprawnie przekierowuje użytkowników do dostawcy OAuth, możesz wywołać metodę `fake` przed wykonaniem żądania do trasy przekierowania. Spowoduje to, że Socialite zwróci przekierowanie do fałszywego adresu URL autoryzacji zamiast przekierowania do rzeczywistego dostawcy OAuth:

```php
use Laravel\Socialite\Socialite;

test('user is redirected to github', function () {
    Socialite::fake('github');

    $response = $this->get('/auth/github/redirect');

    $response->assertRedirect();
});
```

<a name="faking-the-callback"></a>
#### Mockowanie callbacku

Aby przetestować trasę callbacku Twojej aplikacji, możesz wywołać metodę `fake` i dostarczyć instancję `User`, która powinna zostać zwrócona, gdy Twoja aplikacja poprosi o szczegóły użytkownika od dostawcy. Instancja `User` może zostać utworzona przy użyciu metody `map`:

```php
use Laravel\Socialite\Socialite;
use Laravel\Socialite\Two\User;

test('user can login with github', function () {
    Socialite::fake('github', (new User)->map([
        'id' => 'github-123',
        'name' => 'Jason Beggs',
        'email' => 'jason@example.com',
    ]));

    $response = $this->get('/auth/github/callback');

    $response->assertRedirect('/dashboard');

    $this->assertDatabaseHas('users', [
        'name' => 'Jason Beggs',
        'email' => 'jason@example.com',
        'github_id' => 'github-123',
    ]);
});
```

Domyślnie instancja `User` będzie również zawierać właściwość `token`. Jeśli potrzeba, możesz ręcznie określić dodatkowe właściwości instancji `User`:

```php
$fakeUser = (new User)->map([
    'id' => 'github-123',
    'name' => 'Jason Beggs',
    'email' => 'jason@example.com',
])->setToken('fake-token')
  ->setRefreshToken('fake-refresh-token')
  ->setExpiresIn(3600)
  ->setApprovedScopes(['read', 'write'])
```
