# Zestawy Startowe

- [Wprowadzenie](#introduction)
- [Tworzenie Aplikacji przy Użyciu Zestawu Startowego](#creating-an-application)
- [Dostępne Zestawy Startowe](#available-starter-kits)
    - [React](#react)
    - [Vue](#vue)
    - [Livewire](#livewire)
- [Dostosowywanie Zestawu Startowego](#starter-kit-customization)
    - [React](#react-customization)
    - [Vue](#vue-customization)
    - [Livewire](#livewire-customization)
- [Uwierzytelnianie](#authentication)
    - [Włączanie i Wyłączanie Funkcji](#enabling-and-disabling-features)
    - [Dostosowywanie Tworzenia Użytkowników i Resetowania Hasła](#customizing-actions)
    - [Uwierzytelnianie Dwuskładnikowe](#two-factor-authentication)
    - [Ograniczanie Szybkości Żądań](#rate-limiting)
- [Uwierzytelnianie WorkOS AuthKit](#workos)
- [Inertia SSR](#inertia-ssr)
- [Zestawy Startowe Utrzymywane przez Społeczność](#community-maintained-starter-kits)
- [Najczęściej Zadawane Pytania](#faqs)

<a name="introduction"></a>
## Wprowadzenie

Aby dać ci dobry start w budowaniu nowej aplikacji Laravel, z przyjemnością oferujemy [zestawy startowe aplikacji](https://laravel.com/starter-kits). Te zestawy startowe dają ci przewagę w budowaniu kolejnej aplikacji Laravel i zawierają trasy, kontrolery i widoki potrzebne do rejestracji i uwierzytelniania użytkowników twojej aplikacji. Zestawy startowe wykorzystują [Laravel Fortify](/docs/{{version}}/fortify) do zapewnienia uwierzytelniania.

Chociaż możesz korzystać z tych zestawów startowych, nie są one wymagane. Możesz swobodnie budować własną aplikację od podstaw, po prostu instalując świeżą kopię Laravela. Tak czy inaczej, wiemy, że zbudujesz coś wspaniałego!

<a name="creating-an-application"></a>
## Tworzenie Aplikacji przy Użyciu Zestawu Startowego

Aby utworzyć nową aplikację Laravel przy użyciu jednego z naszych zestawów startowych, najpierw powinieneś [zainstalować PHP i narzędzie CLI Laravel](/docs/{{version}}/installation#installing-php). Jeśli masz już zainstalowane PHP i Composer, możesz zainstalować narzędzie CLI instalatora Laravel przez Composer:

```shell
composer global require laravel/installer
```

Następnie utwórz nową aplikację Laravel używając CLI instalatora Laravel. Instalator Laravel poprosi cię o wybór preferowanego zestawu startowego:

```shell
laravel new my-app
```

Po utworzeniu aplikacji Laravel wystarczy zainstalować jej zależności frontendowe przez NPM i uruchomić serwer deweloperski Laravel:

```shell
cd my-app
npm install && npm run build
composer run dev
```

Po uruchomieniu serwera deweloperskiego Laravel, twoja aplikacja będzie dostępna w przeglądarce internetowej pod adresem [http://localhost:8000](http://localhost:8000).

<a name="available-starter-kits"></a>
## Dostępne Zestawy Startowe

<a name="react"></a>
### React

Nasz zestaw startowy React zapewnia solidny, nowoczesny punkt wyjścia do budowania aplikacji Laravel z frontendem React przy użyciu [Inertia](https://inertiajs.com).

Inertia pozwala budować nowoczesne, jednostronicowe aplikacje React używając klasycznego routingu po stronie serwera i kontrolerów. To pozwala cieszyć się mocą frontendu React połączoną z niewiarygodną produktywnością backendu Laravel i błyskawiczną kompilacją Vite.

Zestaw startowy React wykorzystuje React 19, TypeScript, Tailwind oraz bibliotekę komponentów [shadcn/ui](https://ui.shadcn.com).

<a name="vue"></a>
### Vue

Nasz zestaw startowy Vue zapewnia świetny punkt wyjścia do budowania aplikacji Laravel z frontendem Vue przy użyciu [Inertia](https://inertiajs.com).

Inertia pozwala budować nowoczesne, jednostronicowe aplikacje Vue używając klasycznego routingu po stronie serwera i kontrolerów. To pozwala cieszyć się mocą frontendu Vue połączoną z niewiarygodną produktywnością backendu Laravel i błyskawiczną kompilacją Vite.

Zestaw startowy Vue wykorzystuje Vue Composition API, TypeScript, Tailwind oraz bibliotekę komponentów [shadcn-vue](https://www.shadcn-vue.com/).

<a name="livewire"></a>
### Livewire

Nasz zestaw startowy Livewire zapewnia idealny punkt wyjścia do budowania aplikacji Laravel z frontendem [Laravel Livewire](https://livewire.laravel.com).

Livewire to potężny sposób budowania dynamicznych, reaktywnych interfejsów użytkownika frontendowych przy użyciu tylko PHP. To świetne rozwiązanie dla zespołów, które głównie używają szablonów Blade i szukają prostszej alternatywy dla frameworków SPA opartych na JavaScript, takich jak React i Vue.

Zestaw startowy Livewire wykorzystuje Livewire, Tailwind oraz bibliotekę komponentów [Flux UI](https://fluxui.dev).

<a name="starter-kit-customization"></a>
## Dostosowywanie Zestawu Startowego

<a name="react-customization"></a>
### React

Nasz zestaw startowy React został zbudowany z Inertia 2, React 19, Tailwind 4 i [shadcn/ui](https://ui.shadcn.com). Podobnie jak wszystkie nasze zestawy startowe, cały kod backendu i frontendu znajduje się w twojej aplikacji, co pozwala na pełne dostosowanie.

Większość kodu frontendowego znajduje się w katalogu `resources/js`. Możesz swobodnie modyfikować dowolny kod, aby dostosować wygląd i zachowanie swojej aplikacji:

```text
resources/js/
├── components/    # Komponenty React wielokrotnego użytku
├── hooks/         # Hooki React
├── layouts/       # Layouty aplikacji
├── lib/           # Funkcje użytkowe i konfiguracja
├── pages/         # Komponenty stron
└── types/         # Definicje TypeScript
```

Aby opublikować dodatkowe komponenty shadcn, najpierw [znajdź komponent, który chcesz opublikować](https://ui.shadcn.com). Następnie opublikuj komponent używając `npx`:

```shell
npx shadcn@latest add switch
```

W tym przykładzie polecenie opublikuje komponent Switch w `resources/js/components/ui/switch.tsx`. Po opublikowaniu komponentu możesz użyć go na dowolnej ze swoich stron:

```jsx
import { Switch } from "@/components/ui/switch"

const MyPage = () => {
  return (
    <div>
      <Switch />
    </div>
  );
};

export default MyPage;
```

<a name="react-available-layouts"></a>
#### Dostępne Layouty

Zestaw startowy React zawiera dwa różne główne layouty do wyboru: layout "sidebar" (boczny pasek) i layout "header" (nagłówek). Layout sidebar jest domyślny, ale możesz przełączyć się na layout header modyfikując layout importowany na początku pliku `resources/js/layouts/app-layout.tsx` twojej aplikacji:

```js
import AppLayoutTemplate from '@/layouts/app/app-sidebar-layout'; // [tl! remove]
import AppLayoutTemplate from '@/layouts/app/app-header-layout'; // [tl! add]
```

<a name="react-sidebar-variants"></a>
#### Warianty Sidebar

Layout sidebar zawiera trzy różne warianty: domyślny wariant sidebar, wariant "inset" i wariant "floating". Możesz wybrać wariant, który najbardziej ci odpowiada, modyfikując komponent `resources/js/components/app-sidebar.tsx`:

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

<a name="react-authentication-page-layout-variants"></a>
#### Warianty Layoutu Stron Uwierzytelniania

Strony uwierzytelniania dołączone do zestawu startowego React, takie jak strona logowania i strona rejestracji, oferują również trzy różne warianty layoutu: "simple", "card" i "split".

Aby zmienić layout uwierzytelniania, zmodyfikuj layout importowany na początku pliku `resources/js/layouts/auth-layout.tsx` twojej aplikacji:

```js
import AuthLayoutTemplate from '@/layouts/auth/auth-simple-layout'; // [tl! remove]
import AuthLayoutTemplate from '@/layouts/auth/auth-split-layout'; // [tl! add]
```

<a name="vue-customization"></a>
### Vue

Nasz zestaw startowy Vue został zbudowany z Inertia 2, Vue 3 Composition API, Tailwind i [shadcn-vue](https://www.shadcn-vue.com/). Podobnie jak wszystkie nasze zestawy startowe, cały kod backendu i frontendu znajduje się w twojej aplikacji, co pozwala na pełne dostosowanie.

Większość kodu frontendowego znajduje się w katalogu `resources/js`. Możesz swobodnie modyfikować dowolny kod, aby dostosować wygląd i zachowanie swojej aplikacji:

```text
resources/js/
├── components/    # Komponenty Vue wielokrotnego użytku
├── composables/   # Vue composables / hooki
├── layouts/       # Layouty aplikacji
├── lib/           # Funkcje użytkowe i konfiguracja
├── pages/         # Komponenty stron
└── types/         # Definicje TypeScript
```

Aby opublikować dodatkowe komponenty shadcn-vue, najpierw [znajdź komponent, który chcesz opublikować](https://www.shadcn-vue.com). Następnie opublikuj komponent używając `npx`:

```shell
npx shadcn-vue@latest add switch
```

W tym przykładzie polecenie opublikuje komponent Switch w `resources/js/components/ui/Switch.vue`. Po opublikowaniu komponentu możesz użyć go na dowolnej ze swoich stron:

```vue
<script setup lang="ts">
import { Switch } from '@/Components/ui/switch'
</script>

<template>
    <div>
        <Switch />
    </div>
</template>
```

<a name="vue-available-layouts"></a>
#### Dostępne Layouty

Zestaw startowy Vue zawiera dwa różne główne layouty do wyboru: layout "sidebar" (boczny pasek) i layout "header" (nagłówek). Layout sidebar jest domyślny, ale możesz przełączyć się na layout header modyfikując layout importowany na początku pliku `resources/js/layouts/AppLayout.vue` twojej aplikacji:

```js
import AppLayout from '@/layouts/app/AppSidebarLayout.vue'; // [tl! remove]
import AppLayout from '@/layouts/app/AppHeaderLayout.vue'; // [tl! add]
```

<a name="vue-sidebar-variants"></a>
#### Warianty Sidebar

Layout sidebar zawiera trzy różne warianty: domyślny wariant sidebar, wariant "inset" i wariant "floating". Możesz wybrać wariant, który najbardziej ci odpowiada, modyfikując komponent `resources/js/components/AppSidebar.vue`:

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

<a name="vue-authentication-page-layout-variants"></a>
#### Warianty Layoutu Stron Uwierzytelniania

Strony uwierzytelniania dołączone do zestawu startowego Vue, takie jak strona logowania i strona rejestracji, oferują również trzy różne warianty layoutu: "simple", "card" i "split".

Aby zmienić layout uwierzytelniania, zmodyfikuj layout importowany na początku pliku `resources/js/layouts/AuthLayout.vue` twojej aplikacji:

```js
import AuthLayout from '@/layouts/auth/AuthSimpleLayout.vue'; // [tl! remove]
import AuthLayout from '@/layouts/auth/AuthSplitLayout.vue'; // [tl! add]
```

<a name="livewire-customization"></a>
### Livewire

Nasz zestaw startowy Livewire został zbudowany z Livewire 4, Tailwind i [Flux UI](https://fluxui.dev/). Podobnie jak wszystkie nasze zestawy startowe, cały kod backendu i frontendu znajduje się w twojej aplikacji, co pozwala na pełne dostosowanie.

Większość kodu frontendowego znajduje się w katalogu `resources/views`. Możesz swobodnie modyfikować dowolny kod, aby dostosować wygląd i zachowanie swojej aplikacji:

```text
resources/views
├── components            # Komponenty wielokrotnego użytku
├── flux                  # Dostosowane komponenty Flux
├── layouts               # Layouty aplikacji
├── pages                 # Strony Livewire
├── partials              # Części Blade wielokrotnego użytku
├── dashboard.blade.php   # Dashboard uwierzytelnionego użytkownika
├── welcome.blade.php     # Strona powitalna dla gości
```

<a name="livewire-available-layouts"></a>
#### Dostępne Layouty

Zestaw startowy Livewire zawiera dwa różne główne layouty do wyboru: layout "sidebar" (boczny pasek) i layout "header" (nagłówek). Layout sidebar jest domyślny, ale możesz przełączyć się na layout header modyfikując layout używany przez plik `resources/views/layouts/app.blade.php` twojej aplikacji. Ponadto powinieneś dodać atrybut `container` do głównego komponentu Flux:

```blade
<x-layouts::app.header>
    <flux:main container>
        {{ $slot }}
    </flux:main>
</x-layouts::app.header>
```

<a name="livewire-authentication-page-layout-variants"></a>
#### Warianty Layoutu Stron Uwierzytelniania

Strony uwierzytelniania dołączone do zestawu startowego Livewire, takie jak strona logowania i strona rejestracji, oferują również trzy różne warianty layoutu: "simple", "card" i "split".

Aby zmienić layout uwierzytelniania, zmodyfikuj layout używany przez plik `resources/views/layouts/auth.blade.php` twojej aplikacji:

```blade
<x-layouts::auth.split>
    {{ $slot }}
</x-layouts::auth.split>
```

<a name="authentication"></a>
## Uwierzytelnianie

Wszystkie zestawy startowe używają [Laravel Fortify](/docs/{{version}}/fortify) do obsługi uwierzytelniania. Fortify dostarcza trasy, kontrolery i logikę dla logowania, rejestracji, resetowania hasła, weryfikacji e-mail i innych.

Fortify automatycznie rejestruje następujące trasy uwierzytelniania na podstawie funkcji, które są włączone w pliku konfiguracyjnym `config/fortify.php` twojej aplikacji:

| Trasa                              | Metoda | Opis                         |
| ---------------------------------- | ------ | ----------------------------------- |
| `/login`                           | `GET`    | Wyświetl formularz logowania                  |
| `/login`                           | `POST`   | Uwierzytelnij użytkownika                   |
| `/logout`                          | `POST`   | Wyloguj użytkownika                        |
| `/register`                        | `GET`    | Wyświetl formularz rejestracji           |
| `/register`                        | `POST`   | Utwórz nowego użytkownika                     |
| `/forgot-password`                 | `GET`    | Wyświetl formularz żądania resetowania hasła |
| `/forgot-password`                 | `POST`   | Wyślij link resetowania hasła            |
| `/reset-password/{token}`          | `GET`    | Wyświetl formularz resetowania hasła         |
| `/reset-password`                  | `POST`   | Zaktualizuj hasło                     |
| `/email/verify`                    | `GET`    | Wyświetl powiadomienie o weryfikacji e-mail   |
| `/email/verify/{id}/{hash}`        | `GET`    | Zweryfikuj adres e-mail                |
| `/email/verification-notification` | `POST`   | Wyślij ponownie e-mail weryfikacyjny           |
| `/user/confirm-password`           | `GET`    | Wyświetl formularz potwierdzenia hasła  |
| `/user/confirm-password`           | `POST`   | Potwierdź hasło                    |
| `/two-factor-challenge`            | `GET`    | Wyświetl formularz wyzwania 2FA          |
| `/two-factor-challenge`            | `POST`   | Zweryfikuj kod 2FA                     |

Polecenie Artisan `php artisan route:list` może zostać użyte do wyświetlenia wszystkich tras w twojej aplikacji.

<a name="enabling-and-disabling-features"></a>
### Włączanie i Wyłączanie Funkcji

Możesz kontrolować, które funkcje Fortify są włączone w pliku konfiguracyjnym `config/fortify.php` twojej aplikacji:

```php
use Laravel\Fortify\Features;

'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
    Features::twoFactorAuthentication([
        'confirm' => true,
        'confirmPassword' => true,
    ]),
],
```

Aby wyłączyć funkcję, zakomentuj lub usuń ten wpis funkcji z tablicy `features`. Na przykład, usuń `Features::registration()`, aby wyłączyć publiczną rejestrację.

Korzystając z zestawów startowych [React](#react) lub [Vue](#vue), będziesz również musiał usunąć wszelkie odwołania do tras wyłączonej funkcji w kodzie frontendowym. Na przykład, jeśli wyłączysz weryfikację e-mail, powinieneś usunąć importy i odwołania do tras `verification` w swoich komponentach Vue lub React. Jest to konieczne, ponieważ te zestawy startowe używają Wayfinder dla bezpiecznego typowo routingu, który generuje definicje tras w czasie budowania. Jeśli będziesz odwoływać się do tras, które już nie istnieją, twoja aplikacja nie skończy budowania.

<a name="customizing-actions"></a>
### Dostosowywanie Tworzenia Użytkowników i Resetowania Hasła

Gdy użytkownik rejestruje się lub resetuje hasło, Fortify wywołuje klasy akcji znajdujące się w katalogu `app/Actions/Fortify` twojej aplikacji:

| Plik                          | Opis                           |
| ----------------------------- | ------------------------------------- |
| `CreateNewUser.php`           | Waliduje i tworzy nowych użytkowników       |
| `ResetUserPassword.php`       | Waliduje i aktualizuje hasła użytkowników  |
| `PasswordValidationRules.php` | Definiuje reguły walidacji haseł     |

Na przykład, aby dostosować logikę rejestracji twojej aplikacji, powinieneś edytować akcję `CreateNewUser`:

```php
public function create(array $input): User
{
    Validator::make($input, [
        'name' => ['required', 'string', 'max:255'],
        'email' => ['required', 'email', 'max:255', 'unique:users'],
        'phone' => ['required', 'string', 'max:20'], // [tl! add]
        'password' => $this->passwordRules(),
    ])->validate();

    return User::create([
        'name' => $input['name'],
        'email' => $input['email'],
        'phone' => $input['phone'], // [tl! add]
        'password' => Hash::make($input['password']),
    ]);
}
```

<a name="two-factor-authentication"></a>
### Uwierzytelnianie Dwuskadnikowe

Zestawy startowe zawierają wbudowane uwierzytelnianie dwuskładnikowe (2FA), umożliwiające użytkownikom zabezpieczenie swoich kont przy użyciu dowolnej aplikacji uwierzytelniającej kompatybilnej z TOTP. 2FA jest włączone domyślnie przez `Features::twoFactorAuthentication()` w pliku konfiguracyjnym `config/fortify.php` twojej aplikacji.

Opcja `confirm` wymaga od użytkowników weryfikacji kodu przed pełnym włączeniem 2FA, podczas gdy `confirmPassword` wymaga potwierdzenia hasła przed włączeniem lub wyłączeniem 2FA. Więcej szczegółów znajdziesz w [dokumentacji uwierzytelniania dwuskładnikowego Fortify](/docs/{{version}}/fortify#two-factor-authentication).

<a name="rate-limiting"></a>
### Ograniczanie Szybkości Żądań

Ograniczanie szybkości żądań zapobiega atakom siłowym i powtórzonym próbom logowania przed przytłoczeniem twoich punktów końcowych uwierzytelniania. Możesz dostosować zachowanie ograniczania szybkości żądań Fortify w `FortifyServiceProvider` twojej aplikacji:

```php
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Cache\RateLimiting\Limit;

RateLimiter::for('login', function ($request) {
    return Limit::perMinute(5)->by($request->email.$request->ip());
});
```

<a name="workos"></a>
## Uwierzytelnianie WorkOS AuthKit

Domyślnie zestawy startowe React, Vue i Livewire wykorzystują wbudowany system uwierzytelniania Laravel, aby oferować logowanie, rejestrację, resetowanie hasła, weryfikację e-mail i inne. Ponadto oferujemy również wariant każdego zestawu startowego zasilanego przez [WorkOS AuthKit](https://authkit.com), który oferuje:

<div class="content-list" markdown="1">

- Uwierzytelnianie społecznościowe (Google, Microsoft, GitHub i Apple)
- Uwierzytelnianie passkey
- "Magic Auth" oparty na e-mailu
- SSO

</div>

Używanie WorkOS jako dostawcy uwierzytelniania [wymaga konta WorkOS](https://workos.com). WorkOS oferuje bezpłatne uwierzytelnianie dla aplikacji o do 1 miliona miesięcznych aktywnych użytkowników.

Aby użyć WorkOS AuthKit jako dostawcy uwierzytelniania twojej aplikacji, wybierz opcję WorkOS podczas tworzenia nowej aplikacji zasilanej zestawem startowym za pomocą `laravel new`.

### Konfigurowanie Twojego Zestawu Startowego WorkOS

Po utworzeniu nowej aplikacji przy użyciu zestawu startowego zasilanego przez WorkOS, powinieneś ustawić zmienne środowiskowe `WORKOS_CLIENT_ID`, `WORKOS_API_KEY` i `WORKOS_REDIRECT_URL` w pliku `.env` twojej aplikacji. Te zmienne powinny odpowiadać wartościom dostarczonym w panelu WorkOS dla twojej aplikacji:

```ini
WORKOS_CLIENT_ID=your-client-id
WORKOS_API_KEY=your-api-key
WORKOS_REDIRECT_URL="${APP_URL}/authenticate"
```

Ponadto powinieneś skonfigurować URL strony głównej aplikacji w swoim panelu WorkOS. Ten URL to miejsce, do którego użytkownicy zostaną przekierowani po wylogowaniu się z twojej aplikacji.

<a name="configuring-authkit-authentication-methods"></a>
#### Konfigurowanie Metod Uwierzytelniania AuthKit

Korzystając z zestawu startowego zasilanego przez WorkOS, zalecamy wyłączenie uwierzytelniania "E-mail + Hasło" w ustawieniach konfiguracji WorkOS AuthKit twojej aplikacji, pozwalając użytkownikom uwierzytelniać się tylko za pośrednictwem dostawców uwierzytelniania społecznościowego, passkey, "Magic Auth" i SSO. To pozwala twojej aplikacji całkowicie uniknąć obsługi haseł użytkowników.

<a name="configuring-authkit-session-timeouts"></a>
#### Konfigurowanie Limitów Czasu Sesji AuthKit

Ponadto zalecamy skonfigurowanie limitu czasu nieaktywnopci sesji WorkOS AuthKit tak, aby odpowiadał skonfigurowanemu progowi limitu czasu sesji twojej aplikacji Laravel, który zwykle wynosi dwie godziny.

<a name="inertia-ssr"></a>
### Inertia SSR

Zestawy startowe React i Vue są kompatybilne z możliwościami [renderowania po stronie serwera](https://inertiajs.com/server-side-rendering) Inertia. Aby zbudować paczkę kompatybilną z Inertia SSR dla twojej aplikacji, uruchom polecenie `build:ssr`:

```shell
npm run build:ssr
```

Dla wygody dostępne jest również polecenie `composer dev:ssr`. To polecenie uruchomi serwer deweloperski Laravel i serwer Inertia SSR po zbudowaniu paczki kompatybilnej z SSR dla twojej aplikacji, pozwalając ci testować aplikację lokalnie przy użyciu silnika renderowania po stronie serwera Inertia:

```shell
composer dev:ssr
```

<a name="community-maintained-starter-kits"></a>
### Zestawy Startowe Utrzymywane przez Społeczność

Tworząc nową aplikację Laravel przy użyciu instalatora Laravel, możesz podać dowolny zestaw startowy utrzymywany przez społeczność, dostępny na Packagist, do flagi `--using`:

```shell
laravel new my-app --using=example/starter-kit
```

<a name="creating-starter-kits"></a>
#### Tworzenie Zestawów Startowych

Aby upewnić się, że twój zestaw startowy jest dostępny dla innych, będziesz musiał opublikować go na [Packagist](https://packagist.org). Twój zestaw startowy powinien definiować wymagane zmienne środowiskowe w swoim pliku `.env.example`, a wszelkie niezbędne polecenia po instalacji powinny być wymienione w tablicy `post-create-project-cmd` pliku `composer.json` zestawu startowego.

<a name="faqs"></a>
### Najczęściej Zadawane Pytania

<a name="faq-upgrade"></a>
#### Jak zaktualizować?

Każdy zestaw startowy daje ci solidny punkt wyjścia dla twojej następnej aplikacji. Mając pełną własność kodu, możesz modyfikować, dostosować i budować aplikację dokładnie tak, jak sobie wyobrażasz. Jednak nie ma potrzeby aktualizować samego zestawu startowego.

<a name="faq-enable-email-verification"></a>
#### Jak włączyć weryfikację e-mail?

Weryfikację e-mail można dodać, odkomentowując import `MustVerifyEmail` w modelu `App/Models/User.php` i upewniając się, że model implementuje interfejs `MustVerifyEmail`:

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;
// ...

class User extends Authenticatable implements MustVerifyEmail
{
    // ...
}
```

Po rejestracji użytkownicy otrzymają e-mail weryfikacyjny. Aby ograniczyć dostęp do określonych tras, dopóki adres e-mail użytkownika nie zostanie zweryfikowany, dodaj middleware `verified` do tras:

```php
Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('dashboard', function () {
        return Inertia::render('dashboard');
    })->name('dashboard');
});
```

> [!NOTE]
> Weryfikacja e-mail nie jest wymagana w przypadku używania wariantu [WorkOS](#workos) zestawów startowych.

<a name="faq-modify-email-template"></a>
#### Jak zmodyfikować domyślny szablon e-mail?

Możesz chcieć dostosować domyślny szablon e-mail, aby lepiej dostosować go do brandingu twojej aplikacji. Aby zmodyfikować ten szablon, powinieneś opublikować widoki e-mail do swojej aplikacji za pomocą następującego polecenia:

```
php artisan vendor:publish --tag=laravel-mail
```

To wygeneruje kilka plików w `resources/views/vendor/mail`. Możesz modyfikować dowolny z tych plików, a także plik `resources/views/vendor/mail/themes/default.css`, aby zmienić wygląd domyślnego szablonu e-mail.
