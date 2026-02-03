# Pakowanie zasobów (Vite)

- [Wprowadzenie](#introduction)
- [Instalacja i konfiguracja](#installation)
  - [Instalacja Node](#installing-node)
  - [Instalacja Vite i wtyczki Laravel](#installing-vite-and-laravel-plugin)
  - [Konfiguracja Vite](#configuring-vite)
  - [Ładowanie skryptów i stylów](#loading-your-scripts-and-styles)
- [Uruchamianie Vite](#running-vite)
- [Praca z JavaScript](#working-with-scripts)
  - [Aliasy](#aliases)
  - [Vue](#vue)
  - [React](#react)
  - [Inertia](#inertia)
  - [Przetwarzanie URL](#url-processing)
- [Praca z arkuszami stylów](#working-with-stylesheets)
- [Praca z Blade i trasami](#working-with-blade-and-routes)
  - [Przetwarzanie zasobów statycznych za pomocą Vite](#blade-processing-static-assets)
  - [Odświeżanie przy zapisie](#blade-refreshing-on-save)
  - [Aliasy](#blade-aliases)
- [Pobieranie zasobów z wyprzedzeniem](#asset-prefetching)
- [Niestandardowe podstawowe adresy URL](#custom-base-urls)
- [Zmienne środowiskowe](#environment-variables)
- [Wyłączanie Vite w testach](#disabling-vite-in-tests)
- [Renderowanie po stronie serwera (SSR)](#ssr)
- [Atrybuty tagów Script i Style](#script-and-style-attributes)
  - [Nonce polityki bezpieczeństwa treści (CSP)](#content-security-policy-csp-nonce)
  - [Integralność podzasobów (SRI)](#subresource-integrity-sri)
  - [Dowolne atrybuty](#arbitrary-attributes)
- [Zaawansowana personalizacja](#advanced-customization)
  - [Współdzielenie zasobów między różnymi źródłami (CORS) serwera deweloperskiego](#cors)
  - [Korygowanie adresów URL serwera deweloperskiego](#correcting-dev-server-urls)

<a name="introduction"></a>
## Wprowadzenie

[Vite](https://vitejs.dev) to nowoczesne narzędzie do budowania frontendu, które zapewnia niezwykle szybkie środowisko programistyczne i pakuje Twój kod do produkcji. Podczas budowania aplikacji z Laravel zazwyczaj będziesz używać Vite do pakowania plików CSS i JavaScript Twojej aplikacji w zasoby gotowe do produkcji.

Laravel bezproblemowo integruje się z Vite, zapewniając oficjalną wtyczkę i dyrektywę Blade do ładowania zasobów dla środowiska deweloperskiego i produkcyjnego.

<a name="installation"></a>
## Instalacja i konfiguracja

> [!NOTE]
> Poniższa dokumentacja omawia, jak ręcznie zainstalować i skonfigurować wtyczkę Laravel Vite. Jednak [zestawy startowe](/docs/{{version}}/starter-kits) Laravel już zawierają całe to rusztowanie i są najszybszym sposobem na rozpoczęcie pracy z Laravel i Vite.

<a name="installing-node"></a>
### Instalacja Node

Musisz upewnić się, że Node.js (16+) i NPM są zainstalowane przed uruchomieniem Vite i wtyczki Laravel:

```shell
node -v
npm -v
```

Możesz łatwo zainstalować najnowszą wersję Node i NPM używając prostych instalatorów graficznych z [oficjalnej strony Node](https://nodejs.org/en/download/). Lub, jeśli używasz [Laravel Sail](https://laravel.com/docs/{{version}}/sail), możesz wywołać Node i NPM przez Sail:

```shell
./vendor/bin/sail node -v
./vendor/bin/sail npm -v
```

<a name="installing-vite-and-laravel-plugin"></a>
### Instalacja Vite i wtyczki Laravel

W nowej instalacji Laravel znajdziesz plik `package.json` w głównym katalogu struktury Twojej aplikacji. Domyślny plik `package.json` już zawiera wszystko, czego potrzebujesz, aby rozpocząć korzystanie z Vite i wtyczki Laravel. Możesz zainstalować zależności frontendowe Twojej aplikacji przez NPM:

```shell
npm install
```

<a name="configuring-vite"></a>
### Konfiguracja Vite

Vite jest konfigurowany przez plik `vite.config.js` w głównym katalogu Twojego projektu. Możesz dowolnie dostosować ten plik do swoich potrzeb, a także możesz zainstalować inne wtyczki wymagane przez Twoją aplikację, takie jak `@vitejs/plugin-vue` lub `@vitejs/plugin-react`.

Wtyczka Laravel Vite wymaga określenia punktów wejścia dla Twojej aplikacji. Mogą to być pliki JavaScript lub CSS i obejmują przetworzone języki, takie jak TypeScript, JSX, TSX i Sass.

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css',
            'resources/js/app.js',
        ]),
    ],
});
```

Jeśli budujesz SPA, w tym aplikacje zbudowane przy użyciu Inertia, Vite działa najlepiej bez punktów wejścia CSS:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css', // [tl! remove]
            'resources/js/app.js',
        ]),
    ],
});
```

Zamiast tego powinieneś importować swój CSS przez JavaScript. Zazwyczaj byłoby to wykonane w pliku `resources/js/app.js` Twojej aplikacji:

```js
import './bootstrap';
import '../css/app.css'; // [tl! add]
```

Wtyczka Laravel obsługuje również wiele punktów wejścia i zaawansowane opcje konfiguracji, takie jak [punkty wejścia SSR](#ssr).

<a name="working-with-a-secure-development-server"></a>
#### Praca z bezpiecznym serwerem deweloperskim

Jeśli lokalny serwer webowy deweloperski obsługuje Twoją aplikację przez HTTPS, możesz napotkać problemy z połączeniem z serwerem deweloperskim Vite.

Jeśli używasz [Laravel Herd](https://herd.laravel.com) i zabezpieczyłeś witrynę lub używasz [Laravel Valet](/docs/{{version}}/valet) i uruchomiłeś [komendę secure](/docs/{{version}}/valet#securing-sites) dla swojej aplikacji, wtyczka Laravel Vite automatycznie wykryje i użyje wygenerowanego certyfikatu TLS za Ciebie.

Jeśli zabezpieczyłeś witrynę używając hosta, który nie pasuje do nazwy katalogu aplikacji, możesz ręcznie określić hosta w pliku `vite.config.js` Twojej aplikacji:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            detectTls: 'my-app.test', // [tl! add]
        }),
    ],
});
```

Podczas korzystania z innego serwera WWW powinieneś wygenerować zaufany certyfikat i ręcznie skonfigurować Vite, aby używał wygenerowanych certyfikatów:

```js
// ...
import fs from 'fs'; // [tl! add]

const host = 'my-app.test'; // [tl! add]

export default defineConfig({
    // ...
    server: { // [tl! add]
        host, // [tl! add]
        hmr: { host }, // [tl! add]
        https: { // [tl! add]
            key: fs.readFileSync(`/path/to/${host}.key`), // [tl! add]
            cert: fs.readFileSync(`/path/to/${host}.crt`), // [tl! add]
        }, // [tl! add]
    }, // [tl! add]
});
```

Jeśli nie możesz wygenerować zaufanego certyfikatu dla swojego systemu, możesz zainstalować i skonfigurować [wtyczkę @vitejs/plugin-basic-ssl](https://github.com/vitejs/vite-plugin-basic-ssl). Podczas korzystania z niezaufanych certyfikatów będziesz musiał zaakceptować ostrzeżenie o certyfikacie dla serwera deweloperskiego Vite w przeglądarce, klikając link "Local" w konsoli podczas uruchamiania komendy `npm run dev`.

<a name="configuring-hmr-in-sail-on-wsl2"></a>
#### Uruchamianie serwera deweloperskiego w Sail na WSL2

Podczas uruchamiania serwera deweloperskiego Vite w [Laravel Sail](/docs/{{version}}/sail) w podsystemie Windows dla systemu Linux 2 (WSL2), powinieneś dodać następującą konfigurację do pliku `vite.config.js`, aby upewnić się, że przeglądarka może komunikować się z serwerem deweloperskim:

```js
// ...

export default defineConfig({
    // ...
    server: { // [tl! add:start]
        hmr: {
            host: 'localhost',
        },
    }, // [tl! add:end]
});
```

Jeśli zmiany w plikach nie są odzwierciedlane w przeglądarce podczas działania serwera deweloperskiego, może być konieczne skonfigurowanie [opcji server.watch.usePolling](https://vitejs.dev/config/server-options.html#server-watch) Vite.

<a name="loading-your-scripts-and-styles"></a>
### Ładowanie skryptów i stylów

Po skonfigurowaniu punktów wejścia Vite możesz teraz odwoływać się do nich w dyrektywie Blade `@vite()`, którą dodajesz do `<head>` głównego szablonu Twojej aplikacji:

```blade
<!DOCTYPE html>
<head>
    {{-- ... --}}

    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
```

Jeśli importujesz swój CSS przez JavaScript, musisz tylko dołączyć punkt wejścia JavaScript:

```blade
<!DOCTYPE html>
<head>
    {{-- ... --}}

    @vite('resources/js/app.js')
</head>
```

Dyrektywa `@vite` automatycznie wykryje serwer deweloperski Vite i wstrzyknie klienta Vite, aby włączyć Hot Module Replacement. W trybie budowania dyrektywa załaduje Twoje skompilowane i wersjonowane zasoby, w tym wszelkie zaimportowane CSS.

W razie potrzeby możesz również określić ścieżkę budowania skompilowanych zasobów podczas wywoływania dyrektywy `@vite`:

```blade
<!doctype html>
<head>
    {{-- Given build path is relative to public path. --}}

    @vite('resources/js/app.js', 'vendor/courier/build')
</head>
```

<a name="inline-assets"></a>
#### Zasoby wbudowane

Czasami może być konieczne dołączenie surowej zawartości zasobów zamiast linkowania do wersjonowanego adresu URL zasobu. Na przykład może być konieczne dołączenie zawartości zasobu bezpośrednio do strony podczas przekazywania zawartości HTML do generatora PDF. Możesz wyprowadzić zawartość zasobów Vite używając metody `content` dostarczonej przez fasadę `Vite`:

```blade
@use('Illuminate\Support\Facades\Vite')

<!doctype html>
<head>
    {{-- ... --}}

    <style>
        {!! Vite::content('resources/css/app.css') !!}
    </style>
    <script>
        {!! Vite::content('resources/js/app.js') !!}
    </script>
</head>
```

<a name="running-vite"></a>
## Uruchamianie Vite

Istnieją dwa sposoby uruchamiania Vite. Możesz uruchomić serwer deweloperski za pomocą komendy `dev`, co jest przydatne podczas lokalnego programowania. Serwer deweloperski automatycznie wykryje zmiany w plikach i natychmiast odzwierciedli je we wszystkich otwartych oknach przeglądarki.

Lub uruchomienie komendy `build` wersjonuje i pakuje zasoby Twojej aplikacji i przygotowuje je do wdrożenia w produkcji:

```shell
# Run the Vite development server...
npm run dev

# Build and version the assets for production...
npm run build
```

Jeśli uruchamiasz serwer deweloperski w [Sail](/docs/{{version}}/sail) na WSL2, może być potrzebna pewna [dodatkowa konfiguracja](#configuring-hmr-in-sail-on-wsl2).

<a name="working-with-scripts"></a>
## Praca z JavaScript

<a name="aliases"></a>
### Aliasy

Domyślnie wtyczka Laravel zapewnia wspólny alias, aby pomóc Ci szybko rozpocząć i wygodnie importować zasoby Twojej aplikacji:

```js
{
    '@' => '/resources/js'
}
```

Możesz nadpisać alias `'@'`, dodając własny do pliku konfiguracyjnego `vite.config.js`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel(['resources/ts/app.tsx']),
    ],
    resolve: {
        alias: {
            '@': '/resources/ts',
        },
    },
});
```

<a name="vue"></a>
### Vue

Jeśli chcesz zbudować swój frontend używając frameworka [Vue](https://vuejs.org/), będziesz również musiał zainstalować wtyczkę `@vitejs/plugin-vue`:

```shell
npm install --save-dev @vitejs/plugin-vue
```

Możesz następnie dołączyć wtyczkę do pliku konfiguracyjnego `vite.config.js`. Istnieje kilka dodatkowych opcji, których będziesz potrzebować podczas korzystania z wtyczki Vue z Laravel:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.js']),
        vue({
            template: {
                transformAssetUrls: {
                    // The Vue plugin will re-write asset URLs, when referenced
                    // in Single File Components, to point to the Laravel web
                    // server. Setting this to `null` allows the Laravel plugin
                    // to instead re-write asset URLs to point to the Vite
                    // server instead.
                    base: null,

                    // The Vue plugin will parse absolute URLs and treat them
                    // as absolute paths to files on disk. Setting this to
                    // `false` will leave absolute URLs un-touched so they can
                    // reference assets in the public directory as expected.
                    includeAbsolute: false,
                },
            },
        }),
    ],
});
```

> [!NOTE]
> [Zestawy startowe](/docs/{{version}}/starter-kits) Laravel już zawierają odpowiednią konfigurację Laravel, Vue i Vite. Te zestawy startowe oferują najszybszy sposób na rozpoczęcie pracy z Laravel, Vue i Vite.

<a name="react"></a>
### React

Jeśli chcesz zbudować swój frontend używając frameworka [React](https://reactjs.org/), będziesz również musiał zainstalować wtyczkę `@vitejs/plugin-react`:

```shell
npm install --save-dev @vitejs/plugin-react
```

Możesz następnie dołączyć wtyczkę do pliku konfiguracyjnego `vite.config.js`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.jsx']),
        react(),
    ],
});
```

Będziesz musiał upewnić się, że wszystkie pliki zawierające JSX mają rozszerzenie `.jsx` lub `.tsx`, pamiętając o aktualizacji punktu wejścia, jeśli jest to wymagane, jak [pokazano powyżej](#configuring-vite).

Będziesz również musiał dołączyć dodatkową dyrektywę Blade `@viteReactRefresh` wraz z istniejącą dyrektywą `@vite`.

```blade
@viteReactRefresh
@vite('resources/js/app.jsx')
```

Dyrektywa `@viteReactRefresh` musi być wywołana przed dyrektywą `@vite`.

> [!NOTE]
> [Zestawy startowe](/docs/{{version}}/starter-kits) Laravel już zawierają odpowiednią konfigurację Laravel, React i Vite. Te zestawy startowe oferują najszybszy sposób na rozpoczęcie pracy z Laravel, React i Vite.

<a name="inertia"></a>
### Inertia

Wtyczka Laravel Vite zapewnia wygodną funkcję `resolvePageComponent`, która pomoże Ci rozwiązać komponenty strony Inertia. Poniżej znajduje się przykład użycia pomocnika z Vue 3; jednak możesz również wykorzystać funkcję w innych frameworkach, takich jak React:

```js
import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';

createInertiaApp({
  resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
});
```

Jeśli używasz funkcji dzielenia kodu Vite z Inertia, zalecamy skonfigurowanie [pobierania zasobów z wyprzedzeniem](#asset-prefetching).

> [!NOTE]
> [Zestawy startowe](/docs/{{version}}/starter-kits) Laravel już zawierają odpowiednią konfigurację Laravel, Inertia i Vite. Te zestawy startowe oferują najszybszy sposób na rozpoczęcie pracy z Laravel, Inertia i Vite.

<a name="url-processing"></a>
### Przetwarzanie URL

Podczas korzystania z Vite i odwoływania się do zasobów w HTML, CSS lub JS Twojej aplikacji, istnieje kilka zastrzeżeń do rozważenia. Po pierwsze, jeśli odwołujesz się do zasobów za pomocą ścieżki bezwzględnej, Vite nie dołączy zasobu do budowania; dlatego powinieneś upewnić się, że zasób jest dostępny w Twoim katalogu publicznym. Powinieneś unikać używania ścieżek bezwzględnych podczas korzystania z [dedykowanego punktu wejścia CSS](#configuring-vite), ponieważ podczas programowania przeglądarki będą próbowały załadować te ścieżki z serwera deweloperskiego Vite, gdzie hostowany jest CSS, a nie z Twojego katalogu publicznego.

Podczas odwoływania się do względnych ścieżek zasobów powinieneś pamiętać, że ścieżki są względne do pliku, w którym są odwoływane. Wszelkie zasoby odwołane za pomocą ścieżki względnej zostaną przepisane, wersjonowane i spakowane przez Vite.

Rozważ następującą strukturę projektu:

```text
public/
  taylor.png
resources/
  js/
    Pages/
      Welcome.vue
  images/
    abigail.png
```

Poniższy przykład pokazuje, jak Vite będzie traktować względne i bezwzględne adresy URL:

```html
<!-- This asset is not handled by Vite and will not be included in the build -->
<img src="/taylor.png">

<!-- This asset will be re-written, versioned, and bundled by Vite -->
<img src="../../images/abigail.png">
```

<a name="working-with-stylesheets"></a>
## Praca z arkuszami stylów

> [!NOTE]
> [Zestawy startowe](/docs/{{version}}/starter-kits) Laravel już zawierają odpowiednią konfigurację Tailwind i Vite. Lub, jeśli chcesz używać Tailwind i Laravel bez używania jednego z naszych zestawów startowych, sprawdź [przewodnik instalacji Tailwind dla Laravel](https://tailwindcss.com/docs/guides/laravel).

Wszystkie aplikacje Laravel już zawierają Tailwind i prawidłowo skonfigurowany plik `vite.config.js`. Więc musisz tylko uruchomić serwer deweloperski Vite lub uruchomić komendę Composer `dev`, która uruchomi zarówno serwery deweloperskie Laravel, jak i Vite:

```shell
composer run dev
```

CSS Twojej aplikacji może być umieszczony w pliku `resources/css/app.css`.

<a name="working-with-blade-and-routes"></a>
## Praca z Blade i trasami

<a name="blade-processing-static-assets"></a>
### Przetwarzanie zasobów statycznych za pomocą Vite

Podczas odwoływania się do zasobów w JavaScript lub CSS, Vite automatycznie je przetwarza i wersjonuje. Dodatkowo, podczas budowania aplikacji opartych na Blade, Vite może również przetwarzać i wersjonować zasoby statyczne, do których odwołujesz się wyłącznie w szablonach Blade.

Jednak w celu osiągnięcia tego musisz uświadomić Vite o swoich zasobach, importując zasoby statyczne do punktu wejścia aplikacji. Na przykład, jeśli chcesz przetworzyć i wersjonować wszystkie obrazy przechowywane w `resources/images` i wszystkie czcionki przechowywane w `resources/fonts`, powinieneś dodać następujące w punkcie wejścia `resources/js/app.js` Twojej aplikacji:

```js
import.meta.glob([
  '../images/**',
  '../fonts/**',
]);
```

Te zasoby będą teraz przetwarzane przez Vite podczas uruchamiania `npm run build`. Możesz następnie odwoływać się do tych zasobów w szablonach Blade używając metody `Vite::asset`, która zwróci wersjonowany adres URL dla danego zasobu:

```blade
<img src="{{ Vite::asset('resources/images/logo.png') }}">
```

<a name="blade-refreshing-on-save"></a>
### Odświeżanie przy zapisie

Gdy Twoja aplikacja jest zbudowana przy użyciu tradycyjnego renderowania po stronie serwera z Blade, Vite może poprawić Twój przepływ pracy deweloperskiej, automatycznie odświeżając przeglądarkę, gdy wprowadzasz zmiany do plików widoku w aplikacji. Aby rozpocząć, możesz po prostu określić opcję `refresh` jako `true`.

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: true,
        }),
    ],
});
```

Gdy opcja `refresh` jest `true`, zapisywanie plików w następujących katalogach spowoduje, że przeglądarka wykona pełne odświeżenie strony podczas uruchamiania `npm run dev`:

- `app/Livewire/**`
- `app/View/Components/**`
- `lang/**`
- `resources/lang/**`
- `resources/views/**`
- `routes/**`

Obserwowanie katalogu `routes/**` jest przydatne, jeśli wykorzystujesz [Ziggy](https://github.com/tighten/ziggy) do generowania linków tras w frontendzie Twojej aplikacji.

Jeśli te domyślne ścieżki nie odpowiadają Twoim potrzebom, możesz określić własną listę ścieżek do obserwowania:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: ['resources/views/**'],
        }),
    ],
});
```

Pod maską wtyczka Laravel Vite używa pakietu [vite-plugin-full-reload](https://github.com/ElMassimo/vite-plugin-full-reload), który oferuje niektóre zaawansowane opcje konfiguracji do dostrojenia zachowania tej funkcji. Jeśli potrzebujesz tego poziomu dostosowania, możesz podać definicję `config`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: [{
                paths: ['path/to/watch/**'],
                config: { delay: 300 }
            }],
        }),
    ],
});
```

<a name="blade-aliases"></a>
### Aliasy

W aplikacjach JavaScript często [tworzy się aliasy](#aliases) do regularnie odwoływanych katalogów. Ale możesz również tworzyć aliasy do użycia w Blade, używając metody `macro` klasy `Illuminate\Support\Facades\Vite`. Zazwyczaj "makra" powinny być zdefiniowane w metodzie `boot` [dostawcy usług](/docs/{{version}}/providers):

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Vite::macro('image', fn (string $asset) => $this->asset("resources/images/{$asset}"));
}
```

Po zdefiniowaniu makra może być wywołane w Twoich szablonach. Na przykład możemy użyć makra `image` zdefiniowanego powyżej, aby odwołać się do zasobu znajdującego się w `resources/images/logo.png`:

```blade
<img src="{{ Vite::image('logo.png') }}" alt="Laravel Logo">
```

<a name="asset-prefetching"></a>
## Pobieranie zasobów z wyprzedzeniem

Podczas budowania SPA przy użyciu funkcji dzielenia kodu Vite, wymagane zasoby są pobierane przy każdej nawigacji strony. To zachowanie może prowadzić do opóźnionego renderowania interfejsu użytkownika. Jeśli jest to problem dla wybranego frameworka frontendowego, Laravel oferuje możliwość intensywnego wstępnego pobierania zasobów JavaScript i CSS Twojej aplikacji przy początkowym ładowaniu strony.

Możesz poinstruować Laravel, aby intensywnie wstępnie pobierał Twoje zasoby, wywołując metodę `Vite::prefetch` w metodzie `boot` [dostawcy usług](/docs/{{version}}/providers):

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Vite;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Vite::prefetch(concurrency: 3);
    }
}
```

W powyższym przykładzie zasoby będą wstępnie pobierane z maksymalnie `3` równoczesnymi pobieraniami przy każdym ładowaniu strony. Możesz zmodyfikować współbieżność, aby dopasować ją do potrzeb Twojej aplikacji lub określić brak limitu współbieżności, jeśli aplikacja powinna pobrać wszystkie zasoby jednocześnie:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Vite::prefetch();
}
```

Domyślnie pobieranie z wyprzedzeniem rozpocznie się, gdy zostanie uruchomione [zdarzenie _load_ strony](https://developer.mozilla.org/en-US/docs/Web/API/Window/load_event). Jeśli chcesz dostosować, kiedy rozpoczyna się pobieranie z wyprzedzeniem, możesz określić zdarzenie, którego będzie słuchał Vite:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Vite::prefetch(event: 'vite:prefetch');
}
```

Biorąc pod uwagę powyższy kod, pobieranie z wyprzedzeniem rozpocznie się teraz, gdy ręcznie wyślesz zdarzenie `vite:prefetch` do obiektu `window`. Na przykład możesz uruchomić pobieranie z wyprzedzeniem trzy sekundy po załadowaniu strony:

```html
<script>
    addEventListener('load', () => setTimeout(() => {
        dispatchEvent(new Event('vite:prefetch'))
    }, 3000))
</script>
```

<a name="custom-base-urls"></a>
## Niestandardowe podstawowe adresy URL

Jeśli Twoje skompilowane zasoby Vite są wdrażane w domenie oddzielnej od Twojej aplikacji, na przykład za pośrednictwem CDN, musisz określić zmienną środowiskową `ASSET_URL` w pliku `.env` Twojej aplikacji:

```env
ASSET_URL=https://cdn.example.com
```

Po skonfigurowaniu adresu URL zasobu wszystkie przepisane adresy URL do Twoich zasobów zostaną poprzedzone skonfigurowaną wartością:

```text
https://cdn.example.com/build/assets/app.9dce8d17.js
```

Pamiętaj, że [bezwzględne adresy URL nie są przepisywane przez Vite](#url-processing), więc nie zostaną poprzedzone prefiksem.

<a name="environment-variables"></a>
## Zmienne środowiskowe

Możesz wstrzyknąć zmienne środowiskowe do swojego JavaScript, poprzedzając je `VITE_` w pliku `.env` Twojej aplikacji:

```env
VITE_SENTRY_DSN_PUBLIC=http://example.com
```

Możesz uzyskać dostęp do wstrzykniętych zmiennych środowiskowych przez obiekt `import.meta.env`:

```js
import.meta.env.VITE_SENTRY_DSN_PUBLIC
```

<a name="disabling-vite-in-tests"></a>
## Wyłączanie Vite w testach

Integracja Laravel z Vite będzie próbowała rozwiązać Twoje zasoby podczas uruchamiania testów, co wymaga uruchomienia serwera deweloperskiego Vite lub zbudowania zasobów.

Jeśli wolisz mockować Vite podczas testowania, możesz wywołać metodę `withoutVite`, która jest dostępna dla wszystkich testów rozszerzających klasę `TestCase` Laravel:

```php tab=Pest
test('without vite example', function () {
    $this->withoutVite();

    // ...
});
```

```php tab=PHPUnit
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_without_vite_example(): void
    {
        $this->withoutVite();

        // ...
    }
}
```

Jeśli chcesz wyłączyć Vite dla wszystkich testów, możesz wywołać metodę `withoutVite` z metody `setUp` w swojej podstawowej klasie `TestCase`:

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    protected function setUp(): void// [tl! add:start]
    {
        parent::setUp();

        $this->withoutVite();
    }// [tl! add:end]
}
```

<a name="ssr"></a>
## Renderowanie po stronie serwera (SSR)

Wtyczka Laravel Vite czyni bezbolesnym konfigurowanie renderowania po stronie serwera z Vite. Aby rozpocząć, utwórz punkt wejścia SSR w `resources/js/ssr.js` i określ punkt wejścia, przekazując opcję konfiguracji do wtyczki Laravel:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            ssr: 'resources/js/ssr.js',
        }),
    ],
});
```

Aby upewnić się, że nie zapomnisz przebudować punktu wejścia SSR, zalecamy rozszerzenie skryptu "build" w `package.json` Twojej aplikacji, aby utworzyć budowanie SSR:

```json
"scripts": {
     "dev": "vite",
     "build": "vite build" // [tl! remove]
     "build": "vite build && vite build --ssr" // [tl! add]
}
```

Następnie, aby zbudować i uruchomić serwer SSR, możesz uruchomić następujące komendy:

```shell
npm run build
node bootstrap/ssr/ssr.js
```

Jeśli używasz [SSR z Inertia](https://inertiajs.com/server-side-rendering), możesz zamiast tego użyć komendy Artisan `inertia:start-ssr`, aby uruchomić serwer SSR:

```shell
php artisan inertia:start-ssr
```

> [!NOTE]
> [Zestawy startowe](/docs/{{version}}/starter-kits) Laravel już zawierają odpowiednią konfigurację Laravel, Inertia SSR i Vite. Te zestawy startowe oferują najszybszy sposób na rozpoczęcie pracy z Laravel, Inertia SSR i Vite.

<a name="script-and-style-attributes"></a>
## Atrybuty tagów Script i Style

<a name="content-security-policy-csp-nonce"></a>
### Nonce polityki bezpieczeństwa treści (CSP)

Jeśli chcesz dołączyć [atrybut nonce](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce) do tagów script i style jako część [polityki bezpieczeństwa treści](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP), możesz wygenerować lub określić nonce używając metody `useCspNonce` w niestandardowym [middleware](/docs/{{version}}/middleware):

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Vite;
use Symfony\Component\HttpFoundation\Response;

class AddContentSecurityPolicyHeaders
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        Vite::useCspNonce();

        return $next($request)->withHeaders([
            'Content-Security-Policy' => "script-src 'nonce-".Vite::cspNonce()."'",
        ]);
    }
}
```

Po wywołaniu metody `useCspNonce`, Laravel automatycznie dołączy atrybuty `nonce` do wszystkich wygenerowanych tagów script i style.

Jeśli musisz określić nonce w innym miejscu, w tym [dyrektywę Ziggy `@route`](https://github.com/tighten/ziggy#using-routes-with-a-content-security-policy) dołączoną do [zestawów startowych](/docs/{{version}}/starter-kits) Laravel, możesz go pobrać używając metody `cspNonce`:

```blade
@routes(nonce: Vite::cspNonce())
```

Jeśli już masz nonce, którego chcesz użyć w Laravel, możesz przekazać nonce do metody `useCspNonce`:

```php
Vite::useCspNonce($nonce);
```

<a name="subresource-integrity-sri"></a>
### Integralność podzasobów (SRI)

Jeśli Twój manifest Vite zawiera hashe `integrity` dla Twoich zasobów, Laravel automatycznie doda atrybut `integrity` do wszystkich generowanych tagów script i style, aby wymusić [integralność podzasobów](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity). Domyślnie Vite nie zawiera hasza `integrity` w swoim manifeście, ale możesz go włączyć, instalując wtyczkę NPM [vite-plugin-manifest-sri](https://www.npmjs.com/package/vite-plugin-manifest-sri):

```shell
npm install --save-dev vite-plugin-manifest-sri
```

Możesz następnie włączyć tę wtyczkę w pliku `vite.config.js`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import manifestSRI from 'vite-plugin-manifest-sri';// [tl! add]

export default defineConfig({
    plugins: [
        laravel({
            // ...
        }),
        manifestSRI(),// [tl! add]
    ],
});
```

W razie potrzeby możesz również dostosować klucz manifestu, w którym można znaleźć hash integralności:

```php
use Illuminate\Support\Facades\Vite;

Vite::useIntegrityKey('custom-integrity-key');
```

Jeśli chcesz całkowicie wyłączyć to automatyczne wykrywanie, możesz przekazać `false` do metody `useIntegrityKey`:

```php
Vite::useIntegrityKey(false);
```

<a name="arbitrary-attributes"></a>
### Dowolne atrybuty

Jeśli musisz dołączyć dodatkowe atrybuty do tagów script i style, takie jak atrybut [data-turbo-track](https://turbo.hotwired.dev/handbook/drive#reloading-when-assets-change), możesz je określić za pomocą metod `useScriptTagAttributes` i `useStyleTagAttributes`. Zazwyczaj te metody powinny być wywoływane z [dostawcy usług](/docs/{{version}}/providers):

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes([
    'data-turbo-track' => 'reload', // Specify a value for the attribute...
    'async' => true, // Specify an attribute without a value...
    'integrity' => false, // Exclude an attribute that would otherwise be included...
]);

Vite::useStyleTagAttributes([
    'data-turbo-track' => 'reload',
]);
```

Jeśli musisz warunkowo dodać atrybuty, możesz przekazać wywołanie zwrotne, które otrzyma ścieżkę źródłową zasobu, jego adres URL, jego fragment manifestu i cały manifest:

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $src === 'resources/js/app.js' ? 'reload' : false,
]);

Vite::useStyleTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $chunk && $chunk['isEntry'] ? 'reload' : false,
]);
```

> [!WARNING]
> Argumenty `$chunk` i `$manifest` będą `null` podczas działania serwera deweloperskiego Vite.

<a name="advanced-customization"></a>
## Zaawansowana personalizacja

Po wyjęciu z pudełka wtyczka Vite Laravel używa rozsądnych konwencji, które powinny działać dla większości aplikacji; jednak czasami może być konieczne dostosowanie zachowania Vite. Aby włączyć dodatkowe opcje dostosowania, oferujemy następujące metody i opcje, które mogą być używane zamiast dyrektywy Blade `@vite`:

```blade
<!doctype html>
<head>
    {{-- ... --}}

    {{
        Vite::useHotFile(storage_path('vite.hot')) // Customize the "hot" file...
            ->useBuildDirectory('bundle') // Customize the build directory...
            ->useManifestFilename('assets.json') // Customize the manifest filename...
            ->withEntryPoints(['resources/js/app.js']) // Specify the entry points...
            ->createAssetPathsUsing(function (string $path, ?bool $secure) { // Customize the backend path generation for built assets...
                return "https://cdn.example.com/{$path}";
            })
    }}
</head>
```

W pliku `vite.config.js` powinieneś następnie określić tę samą konfigurację:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            hotFile: 'storage/vite.hot', // Customize the "hot" file...
            buildDirectory: 'bundle', // Customize the build directory...
            input: ['resources/js/app.js'], // Specify the entry points...
        }),
    ],
    build: {
      manifest: 'assets.json', // Customize the manifest filename...
    },
});
```

<a name="cors"></a>
### Współdzielenie zasobów między różnymi źródłami (CORS) serwera deweloperskiego

Jeśli doświadczasz problemów ze współdzieleniem zasobów między różnymi źródłami (CORS) w przeglądarce podczas pobierania zasobów z serwera deweloperskiego Vite, może być konieczne przyznanie niestandardowemu źródłu dostępu do serwera deweloperskiego. Vite w połączeniu z wtyczką Laravel pozwala na następujące źródła bez żadnej dodatkowej konfiguracji:

- `::1`
- `127.0.0.1`
- `localhost`
- `*.test`
- `*.localhost`
- `APP_URL` in the project's `.env`

Najłatwiejszym sposobem na zezwolenie na niestandardowe źródło dla Twojego projektu jest upewnienie się, że zmienna środowiskowa `APP_URL` Twojej aplikacji pasuje do źródła, które odwiedzasz w przeglądarce. Na przykład, jeśli odwiedzasz `https://my-app.laravel`, powinieneś zaktualizować swój `.env`, aby pasował:

```env
APP_URL=https://my-app.laravel
```

Jeśli potrzebujesz bardziej szczegółowej kontroli nad źródłami, takiej jak obsługa wielu źródeł, powinieneś wykorzystać [kompleksową i elastyczną wbudowaną konfigurację serwera CORS Vite](https://vite.dev/config/server-options.html#server-cors). Na przykład możesz określić wiele źródeł w opcji konfiguracji `server.cors.origin` w pliku `vite.config.js` projektu:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            refresh: true,
        }),
    ],
    server: {  // [tl! add]
        cors: {  // [tl! add]
            origin: [  // [tl! add]
                'https://backend.laravel',  // [tl! add]
                'http://admin.laravel:8566',  // [tl! add]
            ],  // [tl! add]
        },  // [tl! add]
    },  // [tl! add]
});
```

Możesz również dołączyć wzorce regex, które mogą być pomocne, jeśli chcesz zezwolić na wszystkie źródła dla danej domeny najwyższego poziomu, takiej jak `*.laravel`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            refresh: true,
        }),
    ],
    server: {  // [tl! add]
        cors: {  // [tl! add]
            origin: [ // [tl! add]
                // Supports: SCHEME://DOMAIN.laravel[:PORT] [tl! add]
                /^https?:\/\/.*\.laravel(:\d+)?$/, //[tl! add]
            ], // [tl! add]
        }, // [tl! add]
    }, // [tl! add]
});
```

<a name="correcting-dev-server-urls"></a>
### Korygowanie adresów URL serwera deweloperskiego

Niektóre wtyczki w ekosystemie Vite zakładają, że adresy URL, które zaczynają się od ukośnika, zawsze będą wskazywać na serwer deweloperski Vite. Jednak ze względu na charakter integracji Laravel nie jest to przypadek.

Na przykład wtyczka `vite-imagetools` wyprowadza adresy URL takie jak poniższe, gdy Vite obsługuje Twoje zasoby:

```html
<img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520">
```

Wtyczka `vite-imagetools` oczekuje, że wyjściowy adres URL zostanie przechwycony przez Vite, a wtyczka może następnie obsłużyć wszystkie adresy URL zaczynające się od `/@imagetools`. Jeśli używasz wtyczek oczekujących tego zachowania, będziesz musiał ręcznie poprawić adresy URL. Możesz to zrobić w pliku `vite.config.js`, używając opcji `transformOnServe`.

W tym konkretnym przykładzie dodamy adres URL serwera deweloperskiego do wszystkich wystąpień `/@imagetools` w wygenerowanym kodzie:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import { imagetools } from 'vite-imagetools';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            transformOnServe: (code, devServerUrl) => code.replaceAll('/@imagetools', devServerUrl+'/@imagetools'),
        }),
        imagetools(),
    ],
});
```

Teraz, podczas gdy Vite obsługuje zasoby, wyprowadzi adresy URL wskazujące na serwer deweloperski Vite:

```html
- <img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! remove] -->
+ <img src="http://[::1]:5173/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! add] -->
```
