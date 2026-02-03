# Uwierzytelnianie

- [Wprowadzenie](#introduction)
    - [Zestawy Startowe](#starter-kits)
    - [Uwagi dotyczące bazy danych](#introduction-database-considerations)
    - [Przegląd Ekosystemu](#ecosystem-overview)
- [Szybki start uwierzytelniania](#authentication-quickstart)
    - [Zainstaluj Zestaw Startowy](#install-a-starter-kit)
    - [Pobieranie Uwierzytelnionego Użytkownika](#retrieving-the-authenticated-user)
    - [Ochrona Tras](#protecting-routes)
    - [Ograniczanie logowania](#login-throttling)
- [Ręczne uwierzytelnianie użytkowników](#authenticating-users)
    - [Zapamiętywanie Użytkowników](#remembering-users)
    - [Inne Metody Uwierzytelniania](#other-authentication-methods)
- [Uwierzytelnianie HTTP Basic](#http-basic-authentication)
    - [Bezstanowe uwierzytelnianie HTTP Basic](#stateless-http-basic-authentication)
- [Wylogowanie](#logging-out)
    - [Unieważnianie Sesji na Innych Urządzeniach](#invalidating-sessions-on-other-devices)
- [Potwierdzanie Hasła](#password-confirmation)
    - [Konfiguracja](#password-confirmation-configuration)
    - [Routing](#password-confirmation-routing)
    - [Ochrona Tras](#password-confirmation-protecting-routes)
- [Dodawanie Niestandardowych Guardów](#adding-custom-guards)
    - [Guardy Closure Request](#closure-request-guards)
- [Dodawanie Niestandardowych Dostawców Użytkowników](#adding-custom-user-providers)
    - [Kontrakt Dostawcy Użytkownika](#the-user-provider-contract)
    - [Kontrakt Authenticatable](#the-authenticatable-contract)
- [Automatyczne ponowne haszowanie hasła](#automatic-password-rehashing)
- [Uwierzytelnianie społecznościowe](/docs/{{version}}/socialite)
- [Zdarzenia](#events)

<a name="introduction"></a>
## Wprowadzenie

Wiele aplikacji internetowych udostępnia swoim użytkownikom sposób uwierzytelniania w aplikacji i "logowania". Implementacja tej funkcji w aplikacjach internetowych może być złożonym i potencjalnie ryzykownym przedsięwzięciem. Z tego powodu Laravel stara się dostarczyć narzędzia potrzebne do szybkiej, bezpiecznej i łatwej implementacji uwierzytelniania.

W swojej istocie mechanizmy uwierzytelniania Laravel składają się z "guardów" i "dostawców". Guardy definiują sposób uwierzytelniania użytkowników dla każdego żądania. Na przykład Laravel zawiera guard `session`, który utrzymuje stan za pomocą magazynu sesji i plików cookie.

Dostawcy definiują sposób pobierania użytkowników z trwałego magazynu. Laravel zawiera wsparcie dla pobierania użytkowników za pomocą [Eloquent](/docs/{{version}}/eloquent) i konstruktora zapytań bazy danych. Możesz jednak dowolnie definiować dodatkowych dostawców według potrzeb aplikacji.

Plik konfiguracyjny uwierzytelniania aplikacji znajduje się w `config/auth.php`. Ten plik zawiera kilka dobrze udokumentowanych opcji dostosowywania zachowania usług uwierzytelniania Laravel.

> [!NOTE]
> Guardy i dostawcy nie powinny być mylone z "rolami" i "uprawnieniami". Aby dowiedzieć się więcej o autoryzacji działań użytkowników za pomocą uprawnień, zapoznaj się z dokumentacją [autoryzacji](/docs/{{version}}/authorization).

<a name="starter-kits"></a>
### Zestawy Startowe

Chcesz zacząć szybko? Zainstaluj [zestaw startowy aplikacji Laravel](/docs/{{version}}/starter-kits) w świeżej aplikacji Laravel. Po migracji bazy danych przejdź przeglądarką do `/register` lub dowolnego innego adresu URL przypisanego do aplikacji. Zestawy startowe zajmą się przygotowaniem całego systemu uwierzytelniania!

**Nawet jeśli zdecydujesz się nie używać zestawu startowego w ostatecznej aplikacji Laravel, zainstalowanie [zestawu startowego](/docs/{{version}}/starter-kits) może być doskonałą okazją do nauczenia się, jak zaimplementować wszystkie funkcje uwierzytelniania Laravel w prawdziwym projekcie Laravel.** Ponieważ zestawy startowe Laravel zawierają kontrolery uwierzytelniania, trasy i widoki, możesz sprawdzić kod w tych plikach, aby dowiedzieć się, jak można zaimplementować funkcje uwierzytelniania Laravel.

<a name="introduction-database-considerations"></a>
### Uwagi dotyczące bazy danych

Domyślnie Laravel zawiera model [Eloquent](/docs/{{version}}/eloquent) `App\Models\User` w katalogu `app/Models`. Ten model może być używany z domyślnym sterownikiem uwierzytelniania Eloquent.

Jeśli aplikacja nie używa Eloquent, możesz użyć dostawcy uwierzytelniania `database`, który uśywa konstruktora zapytań Laravel. Jeśli aplikacja używa MongoDB, sprawdź oficjalną [dokumentację uwierzytelniania użytkowników Laravel](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/user-authentication/) dla MongoDB.

Podczas budowania schematu bazy danych dla modelu `App\Models\User`, upewnij się, że kolumna password ma co najmniej 60 znaków długości. Oczywiście migracja tabeli `users`, która jest dołączona do nowych aplikacji Laravel, już tworzy kolumnę przekraczającą tę długość.

Ponadto, powinieneś zweryfikować, że tabela `users` (lub równoważna) zawiera nulowalną, tekstową kolumnę `remember_token` o długości 100 znaków. Ta kolumna będzie używana do przechowywania tokena dla użytkowników, którzy wybiorą opcję "zapamiętaj mnie" podczas logowania do aplikacji. Ponownie, domyślna migracja tabeli `users`, która jest dołączona do nowych aplikacji Laravel, już zawiera tę kolumnę.

<a name="ecosystem-overview"></a>
### Przegląd Ekosystemu

Laravel oferuje kilka pakietów związanych z uwierzytelnianiem. Zanim będziemy kontynuować, dokonamy przeglądu ogólnego ekosystemu uwierzytelniania w Laravel i omówimy przeznaczenie każdego pakietu.

Po pierwsze, zastanowmy się, jak działa uwierzytelnianie. Korzystając z przeglądarki internetowej, użytkownik podaje nazwę użytkownika i hasło za pośrednictwem formularza logowania. Jeśli te dane uwierzytelniające są prawidłowe, aplikacja przechowa informacje o uwierzytelnionym użytkowniku w [sesji](/docs/{{version}}/session) użytkownika. Plik cookie wystawiony przeglądarce zawiera identyfikator sesji, dzięki czemu kolejne żądania do aplikacji mogą kojarzyć użytkownika z prawidłową sesją. Po odebraniu pliku cookie sesji aplikacja pobierze dane sesji na podstawie identyfikatora sesji, zauważy, że informacje uwierzytelniające zostały zapisane w sesji i uzna użytkownika za "uwierzytelnionego".

Kiedy zdalna usługa musi się uwierzytelnić, aby uzyskać dostęp do interfejsu API, pliki cookie zazwyczaj nie są używane do uwierzytelniania, ponieważ nie ma przeglądarki internetowej. Zamiast tego zdalna usługa wysyła token API do interfejsu API z każdym żądaniem. Aplikacja może zweryfikować przychodzący token w tabeli poprawnych tokenów API i "uwierzytelnić" żądanie jako wykonywane przez użytkownika powiązanego z tym tokenem API.

<a name="laravels-built-in-browser-authentication-services"></a>
#### Wbudowane usługi uwierzytelniania przeglądarki Laravel

Laravel zawiera wbudowane usługi uwierzytelniania i sesji, do których zazwyczaj uzyskuje się dostęp za pośrednictwem fasad `Auth` i `Session`. Te funkcje zapewniają uwierzytelnianie oparte na plikach cookie dla żądań inicjowanych z przeglądarek internetowych. Udostępniają metody umożliwiające weryfikację danych uwierzytelniających użytkownika i uwierzytelnienie użytkownika. Ponadto usługi te automatycznie zapiszą prawidłowe dane uwierzytelniające w sesji użytkownika i wydadzą plik cookie sesji użytkownika. Omówienie sposobu korzystania z tych usług zawiera ta dokumentacja.

**Zestawy Startowe Aplikacji**

Jak omówiono w tej dokumentacji, możesz współdziałać z tymi usługami uwierzytelniania ręcznie, aby zbudować własną warstwę uwierzytelniania aplikacji. Jednak aby pomóc Ci zacząć szybciej, udostępniliśmy [darmowe zestawy startowe](/docs/{{version}}/starter-kits), które zapewniają solidne, nowoczesne rusztowanie całej warstwy uwierzytelniania.

<a name="laravels-api-authentication-services"></a>
#### Usługi uwierzytelniania API Laravel

Laravel udostępnia dwa opcjonalne pakiety pomagające w zarządzaniu tokenami API i uwierzytelnianiu żądań wykonywanych za pomocą tokenów API: [Passport](/docs/{{version}}/passport) i [Sanctum](/docs/{{version}}/sanctum). Należy zauważyć, że te biblioteki i wbudowane biblioteki uwierzytelniania oparte na plikach cookie Laravel nie wykluczają się wzajemnie. Te biblioteki koncentrują się głównie na uwierzytelnianiu tokenów API, podczas gdy wbudowane usługi uwierzytelniania koncentrują się na uwierzytelnianiu przeglądarki opartym na plikach cookie. Wiele aplikacji będzie korzystać zarówno z wbudowanych usług uwierzytelniania opartych na plikach cookie Laravel, jak i jednego z pakietów uwierzytelniania API Laravel.

**Passport**

Passport to dostawca uwierzytelniania OAuth2, oferujący różne "typy grantów" OAuth2, które umożliwiają wydawanie różnych typów tokenów. Ogólnie rzecz biorąc, jest to solidny i złożony pakiet do uwierzytelniania API. Jednak większość aplikacji nie wymaga złożonych funkcji oferowanych przez specyfikację OAuth2, które mogą być mylące zarówno dla użytkowników, jak i programistów. Ponadto programiści byli historycznie zdezorientowani co do sposobu uwierzytelniania aplikacji SPA lub aplikacji mobilnych przy użyciu dostawców uwierzytelniania OAuth2, takich jak Passport.

**Sanctum**

W odpowiedzi na złożoność OAuth2 i zamieszanie programistów postanowiliśmy zbudować prostszy, bardziej usprawniony pakiet uwierzytelniania, który mógłby obsłużyć zarówno żądania internetowe pierwszej strony z przeglądarki internetowej, jak i żądania API za pośrednictwem tokenów. Ten cel został zrealizowany wraz z wydaniem [Laravel Sanctum](/docs/{{version}}/sanctum), który powinien być uznawany za preferowany i zalecany pakiet uwierzytelniania dla aplikacji, które będą oferować interfejs webowy pierwszej strony oprócz interfejsu API lub będą zasilane przez aplikację jednostronicową (SPA), która istnieje oddzielnie od backendowej aplikacji Laravel, lub aplikacje oferujące klienta mobilnego.

Laravel Sanctum to hybrydowy pakiet uwierzytelniania web / API, który może zarządzać całym procesem uwierzytelniania aplikacji. Jest to możliwe, ponieważ gdy aplikacje oparte na Sanctum otrzymują żądanie, Sanctum najpierw określi, czy żądanie zawiera plik cookie sesji odwołujący się do uwierzytelnionej sesji. Sanctum realizuje to, wywołując wbudowane usługi uwierzytelniania Laravel, które omówiliśmy wcześniej. Jeśli żądanie nie jest uwierzytelniane za pośrednictwem pliku cookie sesji, Sanctum sprawdzi żądanie pod kątem tokena API. Jeśli token API jest obecny, Sanctum uwierzytelni żądanie za pomocą tego tokena. Aby dowiedzieć się więcej o tym procesie, zapoznaj się z dokumentacją Sanctum ["jak to działa"](/docs/{{version}}/sanctum#how-it-works).

<a name="summary-choosing-your-stack"></a>
#### Podsumowanie i wybór stosu

Podsumowując, jeśli aplikacja będzie dostępna za pomocą przeglądarki i budujesz monolityczną aplikację Laravel, aplikacja będzie korzystać z wbudowanych usług uwierzytelniania Laravel.

Następnie, jeśli aplikacja oferuje interfejs API, który będzie używany przez strony trzecie, wybierzesz między [Passport](/docs/{{version}}/passport) a [Sanctum](/docs/{{version}}/sanctum), aby zapewnić uwierzytelnianie tokena API dla aplikacji. Ogólnie rzecz biorąc, Sanctum powinien być preferowany, gdy jest to możliwe, ponieważ jest prostym, kompletnym rozwiązaniem dla uwierzytelniania API, uwierzytelniania SPA i uwierzytelniania mobilnego, w tym z obsługą "zakresów" lub "zdolności".

Jeśli budujesz aplikację jednostronicową (SPA), która będzie zasilana przez backend Laravel, powinieneś użyć [Laravel Sanctum](/docs/{{version}}/sanctum). Korzystając z Sanctum, będziesz musiał [ręcznie zaimplementować własne trasy uwierzytelniania backendu](#authenticating-users) lub wykorzystać [Laravel Fortify](/docs/{{version}}/fortify) jako bezgłową usługę uwierzytelniania backendu, która zapewnia trasy i kontrolery dla funkcji, takich jak rejestracja, resetowanie hasła, weryfikacja e-mail i inne.

Passport może zostać wybrany, gdy aplikacja absolutnie potrzebuje wszystkich funkcji zapewnianych przez specyfikację OAuth2.

A jeśli chcesz zacząć szybko, z przyjemnością polecamy [nasze zestawy startowe aplikacji](/docs/{{version}}/starter-kits) jako szybki sposób uruchomienia nowej aplikacji Laravel, która już korzysta z naszego preferowanego stosu uwierzytelniania wbudowanych usług uwierzytelniania Laravel.

<a name="authentication-quickstart"></a>
## Szybki start uwierzytelniania

> [!WARNING]
> Ta część dokumentacji omówia uwierzytelnianie użytkowników za pomocą [zestawów startowych aplikacji Laravel](/docs/{{version}}/starter-kits), które obejmują rusztowanie interfejsu użytkownika, które pomóg Ci szybko zacząć. Jeśli chcesz zintegrować się bezpośrednio z systemami uwierzytelniania Laravel, sprawdź dokumentację dotyczącą [ręcznego uwierzytelniania użytkowników](#authenticating-users).

<a name="install-a-starter-kit"></a>
### Zainstaluj Zestaw Startowy

Po pierwsze, powinieneś [zainstalować zestaw startowy aplikacji Laravel](/docs/{{version}}/starter-kits). Nasze zestawy startowe oferują pięknie zaprojektowane punkty wyjścia do włączenia uwierzytelniania do świeżej aplikacji Laravel.

<a name="retrieving-the-authenticated-user"></a>
### Pobieranie Uwierzytelnionego Użytkownika

Po utworzeniu aplikacji z zestawu startowego i umożliwieniu użytkownikom rejestracji i uwierzytelniania w aplikacji, często będziesz musiał współdziałać z aktualnie uwierzytelnionym użytkownikiem. Podczas obsługi przychodzącego żądania możesz uzyskać dostęp do uwierzytelnionego użytkownika za pośrednictwem metody `user` fasady `Auth`:

```php
use Illuminate\Support\Facades\Auth;

// Pobierz aktualnie uwierzytelnionego użytkownika...
$user = Auth::user();

// Pobierz ID aktualnie uwierzytelnionego użytkownika...
$id = Auth::id();
```

Alternatywnie, gdy użytkownik jest uwierzytelniony, możesz uzyskać dostęp do uwierzytelnionego użytkownika za pośrednictwem instancji `Illuminate\Http\Request`. Pamiętaj, że klasy ze wskazanymi typami będą automatycznie wstrzykiwane do metod kontrolera. Wskazując typ obiektu `Illuminate\Http\Request`, możesz uzyskać wygodny dostęp do uwierzytelnionego użytkownika z dowolnej metody kontrolera w aplikacji za pośrednictwem metody `user` żądania:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class FlightController extends Controller
{
    /**
     * Zaktualizuj informacje o locie dla istniejącego lotu.
     */
    public function update(Request $request): RedirectResponse
    {
        $user = $request->user();

        // ...

        return redirect('/flights');
    }
}
```

<a name="determining-if-the-current-user-is-authenticated"></a>
#### Określanie, czy bieżący użytkownik jest uwierzytelniony

Aby określić, czy użytkownik wykonujący przychodzące żądanie HTTP jest uwierzytelniony, możesz użyć metody `check` w fasadzie `Auth`. Ta metoda zwróci `true`, jeśli użytkownik jest uwierzytelniony:

```php
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // Użytkownik jest zalogowany...
}
```

> [!NOTE]
> Choć możliwe jest określenie, czy użytkownik jest uwierzytelniony za pomocą metody `check`, zazwyczaj będziesz używać middleware do weryfikacji, czy użytkownik jest uwierzytelniony, zanim zezwolisz użytkownikowi na dostęp do określonych tras / kontrolerów. Aby dowiedzieć się więcej na ten temat, sprawdź dokumentację dotyczącą [ochrony tras](/docs/{{version}}/authentication#protecting-routes).

<a name="protecting-routes"></a>
### Ochrona Tras

[Middleware tras](/docs/{{version}}/middleware) może być używany do zezwalania tylko uwierzytelnionym użytkownikom na dostęp do danej trasy. Laravel zawiera middleware `auth`, który jest [aliasem middleware](/docs/{{version}}/middleware#middleware-aliases) dla klasy `Illuminate\Auth\Middleware\Authenticate`. Ponieważ ten middleware jest już aliasowany wewnętrznie przez Laravel, wszystko, co musisz zrobić, to podłączyć middleware do definicji trasy:

```php
Route::get('/flights', function () {
    // Tylko uwierzytelnieni użytkownicy mogą uzyskać dostęp do tej trasy...
})->middleware('auth');
```

<a name="redirecting-unauthenticated-users"></a>
#### Przekierowanie nieuwierzytelnionych użytkowników

Kiedy middleware `auth` wykryje nieuwierzytelnionego użytkownika, przekieruje użytkownika do [nazwanej trasy](/docs/{{version}}/routing#named-routes) `login`. Możesz zmodyfikować to zachowanie za pomocą metody `redirectGuestsTo` w pliku `bootstrap/app.php` aplikacji:

```php
use Illuminate\Http\Request;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->redirectGuestsTo('/login');

    // Używając closure...
    $middleware->redirectGuestsTo(fn (Request $request) => route('login'));
})
```

<a name="redirecting-authenticated-users"></a>
#### Przekierowanie uwierzytelnionych użytkowników

Kiedy middleware `guest` wykryje uwierzytelnionego użytkownika, przekieruje użytkownika do nazwanej trasy `dashboard` lub `home`. Możesz zmodyfikować to zachowanie za pomocą metody `redirectUsersTo` w pliku `bootstrap/app.php` aplikacji:

```php
use Illuminate\Http\Request;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->redirectUsersTo('/panel');

    // Używając closure...
    $middleware->redirectUsersTo(fn (Request $request) => route('panel'));
})
```

<a name="specifying-a-guard"></a>
#### Określanie Guard

Przypinając middleware `auth` do trasy, możesz również określić, który "guard" powinien być używany do uwierzytelnienia użytkownika. Określony guard powinien odpowiadać jednemu z kluczy w tablicy `guards` pliku konfiguracyjnego `auth.php`:

```php
Route::get('/flights', function () {
    // Tylko uwierzytelnieni użytkownicy mogą uzyskać dostęp do tej trasy...
})->middleware('auth:admin');
```

<a name="login-throttling"></a>
### Ograniczanie logowania

Jeśli korzystasz z jednego z naszych [zestawów startowych aplikacji](/docs/{{version}}/starter-kits), ograniczanie szybkości będzie automatycznie stosowane do prób logowania. Domyślnie użytkownik nie będzie mógł się zalogować przez jedną minutę, jeśli nie poda poprawnych danych uwierzytelniających po kilku próbach. Ograniczanie jest unikalne dla nazwy użytkownika / adresu e-mail użytkownika i ich adresu IP.

> [!NOTE]
> Jeśli chcesz ograniczyć szybkość innych tras w aplikacji, sprawdź [dokumentację ograniczania szybkości](/docs/{{version}}/routing#rate-limiting).

<a name="authenticating-users"></a>
## Ręczne uwierzytelnianie użytkowników

Nie musisz używać rusztowania uwierzytelniania dołączonego do [zestawów startowych aplikacji](/docs/{{version}}/starter-kits) Laravel. Jeśli zdecydujesz się nie używać tego rusztowania, będziesz musiał zarządzać uwierzytelnianiem użytkownika bezpośrednio za pomocą klas uwierzytelniania Laravel. Nie martw się, to pestka!

Uzyskamy dostęp do usług uwierzytelniania Laravel za pośrednictwem [fasady](/docs/{{version}}/facades) `Auth`, więc będziemy musieli upewnić się, że zaimportujemy fasadę `Auth` na początku klasy. Następnie sprawdźmy metodę `attempt`. Metoda `attempt` jest normalnie używana do obsługi prób uwierzytelnienia z formularza "logowania" aplikacji. Jeśli uwierzytelnienie się powiedzie, powinieneś zregenerować [sesję](/docs/{{version}}/session) użytkownika, aby zapobiec [fiksacji sesji](https://en.wikipedia.org/wiki/Session_fixation):

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    /**
     * Obsłuż próbę uwierzytelnienia.
     */
    public function authenticate(Request $request): RedirectResponse
    {
        $credentials = $request->validate([
            'email' => ['required', 'email'],
            'password' => ['required'],
        ]);

        if (Auth::attempt($credentials)) {
            $request->session()->regenerate();

            return redirect()->intended('dashboard');
        }

        return back()->withErrors([
            'email' => 'Podane dane uwierzytelniające nie pasują do naszych rekordów.',
        ])->onlyInput('email');
    }
}
```

Metoda `attempt` akceptuje tablicę par klucz / wartość jako pierwszy argument. Wartości w tablicy będą używane do znalezienia użytkownika w tabeli bazy danych. Więc w powyższym przykładzie użytkownik zostanie pobrany przez wartość kolumny `email`. Jeśli użytkownik zostanie znaleziony, zahaszowane hasło zapisane w bazie danych zostanie porównane z wartością `password` przekazaną do metody przez tablicę. Nie powinieneś haszować wartości `password` przychodzącego żądania, ponieważ framework automatycznie zahaszuje wartość przed porównaniem jej z zahaszowanym hasłem w bazie danych. Uwierzytelniona sesja zostanie rozpoczęta dla użytkownika, jeśli dwa zahaszowane hasła pasują.

Pamiętaj, że usługi uwierzytelniania Laravel będą pobierać użytkowników z bazy danych na podstawie konfiguracji "dostawcy" guarda uwierzytelniania. W domyślnym pliku konfiguracyjnym `config/auth.php` określony jest dostawca użytkownika Eloquent i jest poinstruowany, aby używać modelu `App\Models\User` podczas pobierania użytkowników. Możesz zmienić te wartości w pliku konfiguracyjnym na podstawie potrzeb aplikacji.

Metoda `attempt` zwróci `true`, jeśli uwierzytelnienie było udane. W przeciwnym razie zostanie zwrócona wartość `false`.

Metoda `intended` dostarczona przez przekierowanie Laravel przekieruje użytkownika na adres URL, do którego próbował uzyskać dostęp przed przechwyceniem przez middleware uwierzytelniania. W przypadku, gdy zamierzone miejsce docelowe nie jest dostępne, można podać rezerwowy URI do tej metody.

<a name="specifying-additional-conditions"></a>
#### Określanie dodatkowych warunków

Jeśli chcesz, możesz również dodać dodatkowe warunki zapytania do zapytania uwierzytelniającego oprócz adresu e-mail i hasła użytkownika. Aby to osiągnąć, możemy po prostu dodać warunki zapytania do tablicy przekazanej do metody `attempt`. Na przykład możemy zweryfikować, czy użytkownik jest oznaczony jako "aktywny":

```php
if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
    // Uwierzytelnienie było udane...
}
```

W przypadku złożonych warunków zapytania możesz podać closure w tablicy danych uwierzytelniających. To closure zostanie wywołane z instancją zapytania, pozwalając dostosować zapytanie zgodnie z potrzebami aplikacji:

```php
use Illuminate\Database\Eloquent\Builder;

if (Auth::attempt([
    'email' => $email,
    'password' => $password,
    fn (Builder $query) => $query->has('activeSubscription'),
])) {
    // Uwierzytelnienie było udane...
}
```

> [!WARNING]
> W tych przykładach `email` nie jest wymaganym parametrem, jest używany jedynie jako przykład. Powinieneś użyć dowolnej nazwy kolumny, która odpowiada "nazwie użytkownika" w tabeli bazy danych.

Metoda `attemptWhen`, która otrzymuje closure jako drugi argument, może być używana do przeprowadzania bardziej rozległej inspekcji potencjalnego użytkownika przed faktycznym uwierzytelnieniem użytkownika. Closure otrzymuje potencjalnego użytkownika i powinno zwrócić `true` lub `false`, aby wskazać, czy użytkownik może zostać uwierzytelniony:

```php
if (Auth::attemptWhen([
    'email' => $email,
    'password' => $password,
], function (User $user) {
    return $user->isNotBanned();
})) {
    // Uwierzytelnienie było udane...
}
```

<a name="accessing-specific-guard-instances"></a>
#### Dostęp do konkretnych instancji Guard

Za pomocą metody `guard` fasady `Auth` możesz określić, której instancji guard chciałbyś użyć podczas uwierzytelniania użytkownika. Pozwala to zarządzać uwierzytelnianiem oddzielnych części aplikacji przy użyciu zupełnie oddzielnych modeli uwierzytelnianych lub tabel użytkowników.

Nazwa guard przekazana do metody `guard` powinna odpowiadać jednemu z guardów skonfigurowanych w pliku konfiguracyjnym `auth.php`:

```php
if (Auth::guard('admin')->attempt($credentials)) {
    // ...
}
```

<a name="remembering-users"></a>
### Zapamiętywanie Użytkowników

Wiele aplikacji internetowych udostępnia pole wyboru "zapamiętaj mnie" w formularzu logowania. Jeśli chcesz zapewnić funkcjonalność "zapamiętaj mnie" w aplikacji, możesz przekazać wartość logiczną jako drugi argument metody `attempt`.

Kiedy ta wartość wynosi `true`, Laravel będzie utrzymywać użytkownika uwierzytelnionym w nieskończoność lub do momentu ręcznego wylogowania. Tabela `users` musi zawierać kolumnę ciągu znaków `remember_token`, która będzie używana do przechowywania tokena "zapamiętaj mnie". Migracja tabeli `users` dołączona do nowych aplikacji Laravel już zawiera tę kolumnę:

```php
use Illuminate\Support\Facades\Auth;

if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // Użytkownik jest zapamiętywany...
}
```

Jeśli aplikacja oferuje funkcjonalność "zapamiętaj mnie", możesz użyć metody `viaRemember`, aby określić, czy aktualnie uwierzytelniony użytkownik został uwierzytelniony za pomocą pliku cookie "zapamiętaj mnie":

```php
use Illuminate\Support\Facades\Auth;

if (Auth::viaRemember()) {
    // ...
}
```

<a name="other-authentication-methods"></a>
### Inne Metody Uwierzytelniania

<a name="authenticate-a-user-instance"></a>
#### Uwierzytelnienie instancji użytkownika

Jeśli musisz ustawić istniejącą instancję użytkownika jako aktualnie uwierzytelnionego użytkownika, możesz przekazać instancję użytkownika do metody `login` fasady `Auth`. Podana instancja użytkownika musi być implementacją [kontraktu](/docs/{{version}}/contracts) `Illuminate\Contracts\Auth\Authenticatable`. Model `App\Models\User` dołączony do Laravel już implementuje ten interfejs. Ta metoda uwierzytelniania jest przydatna, gdy masz już poprawną instancję użytkownika, na przykład bezpośrednio po zarejestrowaniu użytkownika w aplikacji:

```php
use Illuminate\Support\Facades\Auth;

Auth::login($user);
```

Możesz przekazać wartość logiczną jako drugi argument metody `login`. Ta wartość wskazuje, czy funkcjonalność "zapamiętaj mnie" jest pożądana dla uwierzytelnionej sesji. Pamiętaj, że oznacza to, że sesja będzie uwierzytelniana w nieskończoność lub do momentu ręcznego wylogowania użytkownika z aplikacji:

```php
Auth::login($user, $remember = true);
```

Jeśli to konieczne, możesz określić guard uwierzytelniania przed wywołaniem metody `login`:

```php
Auth::guard('admin')->login($user);
```

<a name="authenticate-a-user-by-id"></a>
#### Uwierzytelnienie użytkownika po ID

Aby uwierzytelnić użytkownika przy użyciu klucza głównego rekordu bazy danych, możesz użyć metody `loginUsingId`. Ta metoda akceptuje klucz główny użytkownika, którego chcesz uwierzytelnić:

```php
Auth::loginUsingId(1);
```

Możesz przekazać wartość logiczną do argumentu `remember` metody `loginUsingId`. Ta wartość wskazuje, czy funkcjonalność "zapamiętaj mnie" jest pożądana dla uwierzytelnionej sesji. Pamiętaj, że oznacza to, że sesja będzie uwierzytelniana w nieskończoność lub do momentu ręcznego wylogowania użytkownika z aplikacji:

```php
Auth::loginUsingId(1, remember: true);
```

<a name="authenticate-a-user-once"></a>
#### Jednorazowe uwierzytelnienie użytkownika

Możesz użyć metody `once`, aby uwierzytelnić użytkownika w aplikacji dla pojedynczego żądania. Żadne sesje ani pliki cookie nie będą wykorzystywane podczas wywoływania tej metody, a zdarzenie `Login` nie zostanie wywołane:

```php
if (Auth::once($credentials)) {
    // ...
}
```

<a name="http-basic-authentication"></a>
## Uwierzytelnianie HTTP Basic

[Uwierzytelnianie HTTP Basic](https://en.wikipedia.org/wiki/Basic_access_authentication) zapewnia szybki sposób uwierzytelniania użytkowników aplikacji bez konfigurowania dedykowanej strony "logowania". Aby zacząć, dołącz [middleware](/docs/{{version}}/middleware) `auth.basic` do trasy. Middleware `auth.basic` jest dołączony do frameworka Laravel, więc nie musisz go definiować:

```php
Route::get('/profile', function () {
    // Tylko uwierzytelnieni użytkownicy mogą uzyskać dostęp do tej trasy...
})->middleware('auth.basic');
```

Po przyłączeniu middleware do trasy zostaniesz automatycznie poproszony o podanie danych uwierzytelniających podczas uzyskiwania dostępu do trasy w przeglądarce. Domyślnie middleware `auth.basic` zakłada, że kolumna `email` w tabeli bazy danych `users` jest "nazwą użytkownika" użytkownika.

<a name="a-note-on-fastcgi"></a>
#### Uwaga dotycząca FastCGI

Jeśli używasz [PHP FastCGI](https://www.php.net/manual/en/install.fpm.php) i Apache do obsługi aplikacji Laravel, uwierzytelnianie HTTP Basic może nie działać poprawnie. Aby naprawić te problemy, do pliku `.htaccess` aplikacji można dodać następujące linie:

```apache
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

<a name="stateless-http-basic-authentication"></a>
### Bezstanowe uwierzytelnianie HTTP Basic

Możesz również używać uwierzytelniania HTTP Basic bez ustawiania pliku cookie identyfikatora użytkownika w sesji. Jest to głównie przydatne, jeśli zdecydujesz się użyć uwierzytelniania HTTP do uwierzytelniania żądań do interfejsu API aplikacji. Aby to osiągnąć, [zdefiniuj middleware](/docs/{{version}}/middleware), który wywołuje metodę `onceBasic`. Jeśli żadna odpowiedź nie zostanie zwrócona przez metodę `onceBasic`, żądanie może zostać przekazane dalej do aplikacji:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Symfony\Component\HttpFoundation\Response;

class AuthenticateOnceWithBasicAuth
{
    /**
     * Obsłuż przychodzące żądanie.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        return Auth::onceBasic() ?: $next($request);
    }

}
```

Następnie dołącz middleware do trasy:

```php
Route::get('/api/user', function () {
    // Tylko uwierzytelnieni użytkownicy mogą uzyskać dostęp do tej trasy...
})->middleware(AuthenticateOnceWithBasicAuth::class);
```

<a name="logging-out"></a>
## Wylogowanie

Aby ręcznie wylogować użytkowników z aplikacji, możesz użyć metody `logout` dostarczonej przez fasadę `Auth`. Spowoduje to usunięcie informacji uwierzytelniających z sesji użytkownika, tak aby kolejne żądania nie były uwierzytelniane.

Oprócz wywołania metody `logout` zaleca się unieważnienie sesji użytkownika i wygenerowanie [tokena CSRF](/docs/{{version}}/csrf). Po wylogowaniu użytkownika zazwyczaj przekierujesz użytkownika do głównego katalogu aplikacji:

```php
use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Auth;

/**
 * Wyloguj użytkownika z aplikacji.
 */
public function logout(Request $request): RedirectResponse
{
    Auth::logout();

    $request->session()->invalidate();

    $request->session()->regenerateToken();

    return redirect('/');
}
```

<a name="invalidating-sessions-on-other-devices"></a>
### Unieważnianie Sesji na Innych Urządzeniach

Laravel zapewnia również mechanizm unieważniania i "wylogowywania" sesji użytkownika, które są aktywne na innych urządzeniach bez unieważniania sesji na ich bieżącym urządzeniu. Ta funkcja jest zazwyczaj wykorzystywana, gdy użytkownik zmienia lub aktualizuje swoje hasło i chcesz unieważnić sesje na innych urządzeniach, zachowując uwierzytelnienie bieżącego urządzenia.

Zanim zaczniesz, upewnij się, że middleware `Illuminate\Session\Middleware\AuthenticateSession` jest dołączony do tras, które powinny otrzymać uwierzytelnianie sesji. Zazwyczaj powinieneś umieścić ten middleware w definicji grupy tras, aby mógł być stosowany do większości tras aplikacji. Domyślnie middleware `AuthenticateSession` może być dołączony do trasy przy użyciu [aliasu middleware](/docs/{{version}}/middleware#middleware-aliases) `auth.session`:

```php
Route::middleware(['auth', 'auth.session'])->group(function () {
    Route::get('/', function () {
        // ...
    });
});
```

Następnie możesz użyć metody `logoutOtherDevices` dostarczonej przez fasadę `Auth`. Ta metoda wymaga potwierdzenia przez użytkownika jego aktualnego hasła, które aplikacja powinna zaakceptować za pośrednictwem formularza wejściowego:

```php
use Illuminate\Support\Facades\Auth;

Auth::logoutOtherDevices($currentPassword);
```

Kiedy wywołana zostanie metoda `logoutOtherDevices`, inne sesje użytkownika zostaną całkowicie unieważnione, co oznacza, że zostaną "wylogowani" ze wszystkich guardów, za pomocą których byli wcześniej uwierzytelnieni.

<a name="password-confirmation"></a>
## Potwierdzanie Hasła

Podczas tworzenia aplikacji możesz od czasu do czasu wykonywać działania, które powinny wymagać od użytkownika potwierdzenia hasła przed wykonaniem działania lub przed przekierowaniem użytkownika do wrażliwego obszaru aplikacji. Laravel zawiera wbudowany middleware, który ułatwia ten proces. Wdrożenie tej funkcji wymaga zdefiniowania dwóch tras: jednej trasy wyświetlającej widok prosjący użytkownika o potwierdzenie hasła i drugiej trasy potwierdzającej poprawność hasła i przekierowującej użytkownika do zamierzonego miejsca docelowego.

> [!NOTE]
> Poniższa dokumentacja omówia, jak zintegrować się bezpośrednio z funkcjami potwierdzania hasła Laravel; jednak jeśli chcesz zacząć szybciej, [zestawy startowe aplikacji Laravel](/docs/{{version}}/starter-kits) obejmują obsługę tej funkcji!

<a name="password-confirmation-configuration"></a>
### Konfiguracja

Po potwierdzeniu hasła użytkownik nie będzie proszony o ponowne potwierdzenie hasła przez trzy godziny. Możesz jednak skonfigurować czas, po którym użytkownik zostanie ponownie poproszony o podanie hasła, zmieniając wartość konfiguracyjną `password_timeout` w pliku konfiguracyjnym `config/auth.php` aplikacji.

<a name="password-confirmation-routing"></a>
### Routing

<a name="the-password-confirmation-form"></a>
#### Formularz potwierdzania hasła

Po pierwsze, zdefiniujemy trasę wyświetlającą widok prosjący użytkownika o potwierdzenie hasła:

```php
Route::get('/confirm-password', function () {
    return view('auth.confirm-password');
})->middleware('auth')->name('password.confirm');
```

Jak możesz się spodziewać, widok zwracany przez tę trasę powinien zawierać formularz zawierający pole `password`. Ponadto możesz dowolnie umieścić w widoku tekst wyjaśniający, że użytkownik wchodzi do chronionego obszaru aplikacji i musi potwierdzić swoje hasło.

<a name="confirming-the-password"></a>
#### Potwierdzanie hasła

Następnie zdefiniujemy trasę, która będzie obsługiwać żądanie formularza z widoku "potwierdź hasło". Ta trasa będzie odpowiedzialna za walidację hasła i przekierowanie użytkownika do zamierzonego miejsca docelowego:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

Route::post('/confirm-password', function (Request $request) {
    if (! Hash::check($request->password, $request->user()->password)) {
        return back()->withErrors([
            'password' => ['Podane hasło nie pasuje do naszych rekordów.']
        ]);
    }

    $request->session()->passwordConfirmed();

    return redirect()->intended();
})->middleware(['auth', 'throttle:6,1']);
```

Zanim przejdziemy dalej, przyjrzyjmy się tej trasie bardziej szczegółowo. Najpierw określane jest, czy pole `password` żądania faktycznie pasuje do hasła uwierzytelnionego użytkownika. Jeśli hasło jest poprawne, musimy poinformować sesję Laravel, że użytkownik potwierdził swoje hasło. Metoda `passwordConfirmed` ustawi znacznik czasu w sesji użytkownika, którego Laravel może użyć do określenia, kiedy użytkownik ostatnio potwierdził swoje hasło. Na koniec możemy przekierować użytkownika do zamierzonego miejsca docelowego.

<a name="password-confirmation-protecting-routes"></a>
### Ochrona Tras

Powinieneś upewnić się, że każda trasa wykonująca działanie wymagające niedawnego potwierdzenia hasła ma przypisany middleware `password.confirm`. Ten middleware jest dołączony do domyślnej instalacji Laravel i automatycznie zapisze zamierzone miejsce docelowe użytkownika w sesji, aby użytkownik mógł zostać przekierowany do tej lokalizacji po potwierdzeniu hasła. Po zapisaniu zamierzonego miejsca docelowego użytkownika w sesji middleware przekieruje użytkownika do [nazwanej trasy](/docs/{{version}}/routing#named-routes) `password.confirm`:

```php
Route::get('/settings', function () {
    // ...
})->middleware(['password.confirm']);

Route::post('/settings', function () {
    // ...
})->middleware(['password.confirm']);
```

<a name="adding-custom-guards"></a>
## Dodawanie Niestandardowych Guardów

Możesz zdefiniować własne guardy uwierzytelniania za pomocą metody `extend` w fasadzie `Auth`. Powinieneś umieścić wywołanie metody `extend` w [dostawcy usług](/docs/{{version}}/providers). Ponieważ Laravel już zawiera `AppServiceProvider`, możemy umieścić kod w tym dostawcy:

```php
<?php

namespace App\Providers;

use App\Services\Auth\JwtGuard;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    // ...

    /**
     * Zainicjuj dowolne usługi aplikacji.
     */
    public function boot(): void
    {
        Auth::extend('jwt', function (Application $app, string $name, array $config) {
            // Zwróć instancję Illuminate\Contracts\Auth\Guard...

            return new JwtGuard(Auth::createUserProvider($config['provider']));
        });
    }
}
```

Jak widać w powyższym przykładzie, callback przekazane do metody `extend` powinno zwrócić implementację `Illuminate\Contracts\Auth\Guard`. Ten interfejs zawiera kilka metod, które musisz zaimplementować, aby zdefiniować niestandardowy guard. Po zdefiniowaniu niestandardowego guard możesz odwołać się do niego w konfiguracji `guards` pliku konfiguracyjnego `auth.php`:

```php
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

<a name="closure-request-guards"></a>
### Guardy Closure Request

Najprostszym sposobem wdrożenia niestandardowego systemu uwierzytelniania opartego na żądaniach HTTP jest użycie metody `Auth::viaRequest`. Ta metoda pozwala szybko zdefiniować proces uwierzytelniania za pomocą pojedynczego closure.

Aby zacząć, wywołaj metodę `Auth::viaRequest` w metodzie `boot` klasy `AppServiceProvider` aplikacji. Metoda `viaRequest` akceptuje nazwę sterownika uwierzytelniania jako pierwszy argument. Ta nazwa może być dowolną nazwą opisującą niestandardowy guard. Drugi argument przekazany do metody powinien być closure, które otrzymuje przychodzące żądanie HTTP i zwraca instancję użytkownika lub, jeśli uwierzytelnianie się nie powiedzie, `null`:

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

/**
 * Zainicjuj dowolne usługi aplikacji.
 */
public function boot(): void
{
    Auth::viaRequest('custom-token', function (Request $request) {
        return User::where('token', (string) $request->token)->first();
    });
}
```

Po zdefiniowaniu niestandardowego sterownika uwierzytelniania możesz skonfigurować go jako sterownik w konfiguracji `guards` pliku konfiguracyjnego `auth.php`:

```php
'guards' => [
    'api' => [
        'driver' => 'custom-token',
    ],
],
```

Na koniec możesz odwołać się do guarda podczas przypisywania middleware uwierzytelniania do trasy:

```php
Route::middleware('auth:api')->group(function () {
    // ...
});
```

<a name="adding-custom-user-providers"></a>
## Dodawanie Niestandardowych Dostawców Użytkowników

Jeśli nie używasz tradycyjnej relacyjnej bazy danych do przechowywania użytkowników, będziesz musiał rozszerzyć Laravel o własnego dostawcę użytkowników uwierzytelniania. Użyjemy metody `provider` w fasadzie `Auth`, aby zdefiniować niestandardowego dostawcę użytkowników. Resolver dostawcy użytkownika powinien zwrócić implementację `Illuminate\Contracts\Auth\UserProvider`:

```php
<?php

namespace App\Providers;

use App\Extensions\MongoUserProvider;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    // ...

    /**
     * Zainicjuj dowolne usługi aplikacji.
     */
    public function boot(): void
    {
        Auth::provider('mongo', function (Application $app, array $config) {
            // Zwróć instancję Illuminate\Contracts\Auth\UserProvider...

            return new MongoUserProvider($app->make('mongo.connection'));
        });
    }
}
```

Po zarejestrowaniu dostawcy za pomocą metody `provider`, możesz przełączyć się na nowego dostawcę użytkownika w pliku konfiguracyjnym `auth.php`. Najpierw zdefiniuj `provider`, który używa nowego sterownika:

```php
'providers' => [
    'users' => [
        'driver' => 'mongo',
    ],
],
```

Na koniec możesz odwołać się do tego dostawcy w konfiguracji `guards`:

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
],
```

<a name="the-user-provider-contract"></a>
### Kontrakt Dostawcy Użytkownika

Implementacje `Illuminate\Contracts\Auth\UserProvider` są odpowiedzialne za pobieranie implementacji `Illuminate\Contracts\Auth\Authenticatable` z trwałego systemu przechowywania, takiego jak MySQL, MongoDB, itp. Te dwa interfejsy pozwalają mechanizmom uwierzytelniania Laravel kontynuować działanie niezależnie od sposobu przechowywania danych użytkownika lub typu klasy używanej do reprezentowania uwierzytelnionego użytkownika:

Spójrzmy na kontrakt `Illuminate\Contracts\Auth\UserProvider`:

```php
<?php

namespace Illuminate\Contracts\Auth;

interface UserProvider
{
    public function retrieveById($identifier);
    public function retrieveByToken($identifier, $token);
    public function updateRememberToken(Authenticatable $user, $token);
    public function retrieveByCredentials(array $credentials);
    public function validateCredentials(Authenticatable $user, array $credentials);
    public function rehashPasswordIfRequired(Authenticatable $user, array $credentials, bool $force = false);
}
```

Funkcja `retrieveById` zazwyczaj otrzymuje klucz reprezentujący użytkownika, taki jak automatycznie inkrementowane ID z bazy danych MySQL. Implementacja `Authenticatable` pasująca do ID powinna zostać pobrana i zwrócona przez metodę.

Funkcja `retrieveByToken` pobiera użytkownika po jego unikalnym `$identifier` i tokenie "zapamiętaj mnie" `$token`, zazwyczaj przechowywanych w kolumnie bazy danych takiej jak `remember_token`. Podobnie jak w poprzedniej metodzie, implementacja `Authenticatable` z pasującą wartością tokena powinna zostać zwrócona przez tę metodę.

Metoda `updateRememberToken` aktualizuje `remember_token` instancji `$user` nowym `$token`. Świeży token jest przypisywany użytkownikom po udanej próbie uwierzytelnienia "zapamiętaj mnie" lub gdy użytkownik się wylogowuje.

Metoda `retrieveByCredentials` otrzymuje tablicę danych uwierzytelniających przekazanych do metody `Auth::attempt` podczas próby uwierzytelnienia w aplikacji. Metoda powinna następnie "odpytać" bazowy trwały magazyn w poszukiwaniu użytkownika pasującego do tych danych. Zazwyczaj ta metoda wykona zapytanie z warunkiem "where", które szuka rekordu użytkownika z "nazwą użytkownika" pasującą do wartości `$credentials['username']`. Metoda powinna zwrócić implementację `Authenticatable`. **Ta metoda nie powinna próbować przeprowadzać walidacji ani uwierzytelniania hasła.**

Metoda `validateCredentials` powinna porównać podanego `$user` z `$credentials`, aby uwierzytelnić użytkownika. Na przykład, ta metoda zazwyczaj użyje metody `Hash::check` do porównania wartości `$user->getAuthPassword()` z wartością `$credentials['password']`. Ta metoda powinna zwrócić `true` lub `false` wskazując, czy hasło jest poprawne.

Metoda `rehashPasswordIfRequired` powinna ponownie zahaszować hasło podanego `$user`, jeśli jest to wymagane i obsługiwane. Na przykład, ta metoda zazwyczaj użyje metody `Hash::needsRehash`, aby określić, czy wartość `$credentials['password']` wymaga ponownego haszowania. Jeśli hasło wymaga ponownego haszowania, metoda powinna użyć metody `Hash::make` do ponownego zahaszowania hasła i zaktualizowania rekordu użytkownika w bazowym trwałym magazynie.

<a name="the-authenticatable-contract"></a>
### Kontrakt Authenticatable

Teraz, gdy zbadaliśmy każdą z metod `UserProvider`, spójrzmy na kontrakt `Authenticatable`. Pamiętaj, że dostawcy użytkowników powinni zwracać implementacje tego interfejsu z metod `retrieveById`, `retrieveByToken` i `retrieveByCredentials`:

```php
<?php

namespace Illuminate\Contracts\Auth;

interface Authenticatable
{
    public function getAuthIdentifierName();
    public function getAuthIdentifier();
    public function getAuthPasswordName();
    public function getAuthPassword();
    public function getRememberToken();
    public function setRememberToken($value);
    public function getRememberTokenName();
}
```

Ten interfejs jest prosty. Metoda `getAuthIdentifierName` powinna zwrócić nazwę kolumny "klucza głównego" dla użytkownika, a metoda `getAuthIdentifier` powinna zwrócić "klucz główny" użytkownika. Podczas używania backendu MySQL prawdopodobnie byłby to automatycznie inkrementowany klucz główny przypisany do rekordu użytkownika. Metoda `getAuthPasswordName` powinna zwrócić nazwę kolumny hasła użytkownika. Metoda `getAuthPassword` powinna zwrócić zahaszowane hasło użytkownika.

Ten interfejs pozwala systemowi uwierzytelniania pracować z dowolną klasą "user", niezależnie od używanego ORM-a lub warstwy abstrakcji magazynu. Domyślnie Laravel zawiera klasę `App\Models\User` w katalogu `app/Models`, która implementuje ten interfejs.

<a name="automatic-password-rehashing"></a>
## Automatyczne ponowne haszowanie hasła

Domyślnym algorytmem haszowania haseł Laravel jest bcrypt. "Współczynnik pracy" dla hasheỳ bcrypt można dostosować za pomocą pliku konfiguracyjnego `config/hashing.php` aplikacji lub zmiennej środowiskowej `BCRYPT_ROUNDS`.

Zazwyczaj współczynnik pracy bcrypt powinien być zwiększany wraz z czasem, gdy rośnie moc obliczeniowa CPU / GPU. Jeśli zwiększysz współczynnik pracy bcrypt dla swojej aplikacji, Laravel będzie z wdziękiem i automatycznie ponownie haszować hasła użytkowników, gdy użytkownicy będą się uwierzytelniać w aplikacji za pomocą zestawów startowych Laravel lub gdy [ręcznie uwierzytelniasz użytkowników](#authenticating-users) za pomocą metody `attempt`.

Zazwyczaj automatyczne ponowne haszowanie haseł nie powinno zakłócać działania aplikacji; jednak możesz wyłączyć to zachowanie, publikując plik konfiguracyjny `hashing`:

```shell
php artisan config:publish hashing
```

Po opublikowaniu pliku konfiguracyjnego możesz ustawić wartość konfiguracyjną `rehash_on_login` na `false`:

```php
'rehash_on_login' => false,
```

<a name="events"></a>
## Zdarzenia

Laravel wywołuje różne [zdarzenia](/docs/{{version}}/events) podczas procesu uwierzytelniania. Możesz [zdefiniować nasłuchiwacze](/docs/{{version}}/events) dla dowolnego z następujących zdarzeń:

<div class="overflow-auto">

| Event Name                                     |
| ---------------------------------------------- |
| `Illuminate\Auth\Events\Registered`            |
| `Illuminate\Auth\Events\Attempting`            |
| `Illuminate\Auth\Events\Authenticated`         |
| `Illuminate\Auth\Events\Login`                 |
| `Illuminate\Auth\Events\Failed`                |
| `Illuminate\Auth\Events\Validated`             |
| `Illuminate\Auth\Events\Verified`              |
| `Illuminate\Auth\Events\Logout`                |
| `Illuminate\Auth\Events\CurrentDeviceLogout`   |
| `Illuminate\Auth\Events\OtherDeviceLogout`     |
| `Illuminate\Auth\Events\Lockout`               |
| `Illuminate\Auth\Events\PasswordReset`         |
| `Illuminate\Auth\Events\PasswordResetLinkSent` |

</div>
