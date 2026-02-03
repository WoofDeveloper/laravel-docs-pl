# Broadcasting

- [Wprowadzenie](#introduction)
- [Szybki start](#quickstart)
- [Instalacja po stronie serwera](#server-side-installation)
    - [Reverb](#reverb)
    - [Pusher Channels](#pusher-channels)
    - [Ably](#ably)
- [Instalacja po stronie klienta](#client-side-installation)
    - [Reverb](#client-reverb)
    - [Pusher Channels](#client-pusher-channels)
    - [Ably](#client-ably)
- [Przegląd koncepcji](#concept-overview)
    - [Korzystanie z przykładowej aplikacji](#using-example-application)
- [Definiowanie zdarzeń rozgłoszeniowych](#defining-broadcast-events)
    - [Nazwa rozgłoszenia](#broadcast-name)
    - [Dane rozgłoszenia](#broadcast-data)
    - [Kolejka rozgłoszenia](#broadcast-queue)
    - [Warunki rozgłaszania](#broadcast-conditions)
    - [Rozgłaszanie i transakcje bazodanowe](#broadcasting-and-database-transactions)
- [Autoryzacja kanałów](#authorizing-channels)
    - [Definiowanie callbacków autoryzacji](#defining-authorization-callbacks)
    - [Definiowanie klas kanałów](#defining-channel-classes)
- [Rozgłaszanie zdarzeń](#broadcasting-events)
    - [Tylko do innych](#only-to-others)
    - [Dostosowywanie połączenia](#customizing-the-connection)
    - [Zdarzenia anonimowe](#anonymous-events)
    - [Ratowanie rozgłoszeń](#rescuing-broadcasts)
- [Odbieranie rozgłoszeń](#receiving-broadcasts)
    - [Nasłuchiwanie zdarzeń](#listening-for-events)
    - [Opuszczanie kanału](#leaving-a-channel)
    - [Przestrzenie nazw](#namespaces)
    - [Używanie React lub Vue](#using-react-or-vue)
- [Kanały obecności](#presence-channels)
    - [Autoryzacja kanałów obecności](#authorizing-presence-channels)
    - [Dołączanie do kanałów obecności](#joining-presence-channels)
    - [Rozgłaszanie do kanałów obecności](#broadcasting-to-presence-channels)
- [Rozgłaszanie modeli](#model-broadcasting)
    - [Konwencje rozgłaszania modeli](#model-broadcasting-conventions)
    - [Nasłuchiwanie rozgłoszeń modeli](#listening-for-model-broadcasts)
- [Zdarzenia klienckie](#client-events)
- [Powiadomienia](#notifications)

<a name="introduction"></a>
## Wprowadzenie

W wielu nowoczesnych aplikacjach internetowych WebSockets są używane do implementacji interfejsów użytkownika aktualizowanych w czasie rzeczywistym. Gdy dane są aktualizowane na serwerze, wiadomość jest zazwyczaj wysyłana przez połączenie WebSocket, aby mogła być obsłużona przez klienta. WebSockets zapewniają bardziej efektywną alternatywę dla ciągłego odpytywania serwera aplikacji o zmiany danych, które powinny być odzwierciedlone w interfejsie użytkownika.

Na przykład wyobraź sobie, że Twoja aplikacja może wyeksportować dane użytkownika do pliku CSV i wysłać je do niego e-mailem. Jednak utworzenie tego pliku CSV zajmuje kilka minut, więc wybierasz utworzenie i wysłanie CSV w ramach [zadania w kolejce](/docs/{{version}}/queues). Gdy CSV zostanie utworzony i wysłany do użytkownika, możemy użyć rozgłaszania zdarzeń, aby wysłać zdarzenie `App\Events\UserDataExported`, które zostanie odebrane przez JavaScript naszej aplikacji. Po odebraniu zdarzenia możemy wyświetlić użytkownikowi wiadomość, że jego CSV został wysłany e-mailem, bez konieczności odświeżania strony.

Aby pomóc Ci w budowaniu tego typu funkcji, Laravel ułatwia "rozgłaszanie" Twoich serwerowych [zdarzeń](/docs/{{version}}/events) Laravel przez połączenie WebSocket. Rozgłaszanie zdarzeń Laravel pozwala na współdzielenie tych samych nazw zdarzeń i danych między serwerową aplikacją Laravel a klienckąaplikacją JavaScript.

Podstawowe koncepcje rozgłaszania są proste: klienci łączą się z nazwanymi kanałami po stronie frontendu, podczas gdy Twoja aplikacja Laravel rozgłasza zdarzenia do tych kanałów po stronie backendu. Te zdarzenia mogą zawierać wszelkie dodatkowe dane, które chcesz udostępnić frontendowi.

<a name="supported-drivers"></a>
#### Obsługiwane sterowniki

Domyślnie Laravel zawiera trzy sterowniki rozgłaszania po stronie serwera do wyboru: [Laravel Reverb](https://reverb.laravel.com), [Pusher Channels](https://pusher.com/channels) i [Ably](https://ably.com).

> [!NOTE]
> Przed zagłębieniem się w rozgłaszanie zdarzeń upewnij się, że przeczytałeś dokumentację Laravel dotyczącą [zdarzeń i listenerów](/docs/{{version}}/events).

<a name="quickstart"></a>
## Szybki start

Domyślnie rozgłaszanie nie jest włączone w nowych aplikacjach Laravel. Możesz włączyć rozgłaszanie za pomocą polecenia Artisan `install:broadcasting`:

```shell
php artisan install:broadcasting
```

Polecenie `install:broadcasting` zapyta Cię, której usługi rozgłaszania zdarzeń chciałbyś użyć. Dodatkowo utworzy plik konfiguracyjny `config/broadcasting.php` i plik `routes/channels.php`, w którym możesz zarejestrować trasy i callbacki autoryzacji rozgłaszania aplikacji.

Laravel obsługuje kilka sterowników rozgłaszania od razu: [Laravel Reverb](/docs/{{version}}/reverb), [Pusher Channels](https://pusher.com/channels), [Ably](https://ably.com) oraz sterownik `log` do lokalnego rozwoju i debugowania. Dodatkowo dołączony jest sterownik `null`, który pozwala wyłączyć rozgłaszanie podczas testowania. Przykład konfiguracji dla każdego z tych sterowników znajduje się w pliku konfiguracyjnym `config/broadcasting.php`.

Cała konfiguracja rozgłaszania zdarzeń aplikacji jest przechowywana w pliku konfiguracyjnym `config/broadcasting.php`. Nie martw się, jeśli ten plik nie istnieje w Twojej aplikacji; zostanie utworzony po uruchomieniu polecenia Artisan `install:broadcasting`.

<a name="quickstart-next-steps"></a>
#### Kolejne kroki

Po włączeniu rozgłaszania zdarzeń jesteś gotowy, aby dowiedzieć się więcej o [definiowaniu zdarzeń rozgłoszeniowych](#defining-broadcast-events) i [nasłuchiwaniu zdarzeń](#listening-for-events). Jeśli używasz [starter kits](/docs/{{version}}/starter-kits) React lub Vue, możesz nasłuchiwać zdarzeń za pomocą [hooka useEcho](#using-react-or-vue) Echo.

> [!NOTE]
> Przed rozgłoszeniem jakichkolwiek zdarzeń powinieneś najpierw skonfigurować i uruchomić [worker kolejki](/docs/{{version}}/queues). Wszystkie rozgłaszanie zdarzeń odbywa się za pośrednictwem zadań w kolejce, dzięki czemu czas odpowiedzi Twojej aplikacji nie jest poważnie wpływany przez rozgłaszane zdarzenia.

<a name="server-side-installation"></a>
## Instalacja po stronie serwera

Aby rozpocząć korzystanie z rozgłaszania zdarzeń Laravel, musimy przeprowadzić pewną konfigurację w aplikacji Laravel, a także zainstalować kilka pakietów.

Rozgłaszanie zdarzeń jest realizowane przez sterownik rozgłaszania po stronie serwera, który rozgłasza Twoje zdarzenia Laravel, tak aby Laravel Echo (biblioteka JavaScript) mogła je odbierać w kliencie przeglądarki. Nie martw się - przeprowadzimy Cię przez każdą część procesu instalacji krok po kroku.

<a name="reverb"></a>
### Reverb

Aby szybko włączyć obsługę funkcji rozgłaszania Laravel podczas używania Reverb jako rozgłaszacza zdarzeń, wywołaj polecenie Artisan `install:broadcasting` z opcją `--reverb`. To polecenie Artisan zainstaluje wymagane pakiety Composer i NPM dla Reverb oraz zaktualizuje plik `.env` Twojej aplikacji o odpowiednie zmienne:

```shell
php artisan install:broadcasting --reverb
```

<a name="reverb-manual-installation"></a>
#### Instalacja ręczna

Podczas uruchamiania polecenia `install:broadcasting` zostaniesz poproszony o zainstalowanie [Laravel Reverb](/docs/{{version}}/reverb). Oczywiście możesz również zainstalować Reverb ręcznie za pomocą menedżera pakietów Composer:

```shell
composer require laravel/reverb
```

Po zainstalowaniu pakietu możesz uruchomić polecenie instalacyjne Reverb, aby opublikować konfigurację, dodać wymagane zmienne środowiskowe Reverb i włączyć rozgłaszanie zdarzeń w aplikacji:

```shell
php artisan reverb:install
```

Szczegółowe instrukcje instalacji i użycia Reverb można znaleźć w [dokumentacji Reverb](/docs/{{version}}/reverb).

<a name="pusher-channels"></a>
### Pusher Channels

Aby szybko włączyć obsługę funkcji rozgłaszania Laravel podczas używania Pushera jako rozgłaszacza zdarzeń, wywołaj polecenie Artisan `install:broadcasting` z opcją `--pusher`. To polecenie Artisan zapyta Cię o dane uwierzytelniające Pushera, zainstaluje SDK PHP i JavaScript Pushera oraz zaktualizuje plik `.env` Twojej aplikacji o odpowiednie zmienne:

```shell
php artisan install:broadcasting --pusher
```

<a name="pusher-manual-installation"></a>
#### Instalacja ręczna

Aby zainstalować obsługę Pushera ręcznie, powinieneś zainstalować SDK PHP Pusher Channels za pomocą menedżera pakietów Composer:

```shell
composer require pusher/pusher-php-server
```

Następnie powinieneś skonfigurować dane uwierzytelniające Pusher Channels w pliku konfiguracyjnym `config/broadcasting.php`. Przykładowa konfiguracja Pusher Channels jest już zawarta w tym pliku, co pozwala szybko określić klucz, sekret i ID aplikacji. Zazwyczaj powinieneś skonfigurować dane uwierzytelniające Pusher Channels w pliku `.env` swojej aplikacji:

```ini
PUSHER_APP_ID="your-pusher-app-id"
PUSHER_APP_KEY="your-pusher-key"
PUSHER_APP_SECRET="your-pusher-secret"
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME="https"
PUSHER_APP_CLUSTER="mt1"
```

Konfiguracja `pusher` w pliku `config/broadcasting.php` pozwala również określić dodatkowe `opcje` obsługiwane przez Channels, takie jak cluster.

Następnie ustaw zmienną środowiskową `BROADCAST_CONNECTION` na `pusher` w pliku `.env` Twojej aplikacji:

```ini
BROADCAST_CONNECTION=pusher
```

Na koniec jesteś gotowy, aby zainstalować i skonfigurować [Laravel Echo](#client-side-installation), który będzie odbierał zdarzenia rozgłoszeniowe po stronie klienta.

<a name="ably"></a>
### Ably

> [!NOTE]
> Poniższa dokumentacja omawia, jak używać Ably w trybie "kompatybilności z Pusherem". Jednak zespół Ably zaleca i utrzymuje broadcaster oraz klienta Echo, który może wykorzystać unikalne możliwości oferowane przez Ably. Więcej informacji na temat używania sterowników utrzymywanych przez Ably znajdziesz w [dokumentacji Laravel broadcastera Ably](https://github.com/ably/laravel-broadcaster).

Aby szybko włączyć obsługę funkcji rozgłaszania Laravel podczas używania [Ably](https://ably.com) jako rozgłaszacza zdarzeń, wywołaj polecenie Artisan `install:broadcasting` z opcją `--ably`. To polecenie Artisan zapyta Cię o dane uwierzytelniające Ably, zainstaluje SDK PHP i JavaScript Ably oraz zaktualizuje plik `.env` Twojej aplikacji o odpowiednie zmienne:

```shell
php artisan install:broadcasting --ably
```

**Przed kontynuowaniem powinieneś włączyć obsługę protokołu Pusher w ustawieniach aplikacji Ably. Możesz włączyć tę funkcję w sekcji "Protocol Adapter Settings" panelu ustawień aplikacji Ably.**

<a name="ably-manual-installation"></a>
#### Instalacja ręczna

Aby zainstalować obsługę Ably ręcznie, powinieneś zainstalować SDK PHP Ably za pomocą menedżera pakietów Composer:

```shell
composer require ably/ably-php
```

Następnie powinieneś skonfigurować dane uwierzytelniające Ably w pliku konfiguracyjnym `config/broadcasting.php`. Przykładowa konfiguracja Ably jest już zawarta w tym pliku, co pozwala szybko określić klucz. Zazwyczaj ta wartość powinna być ustawiona za pomocą [zmiennej środowiskowej](/docs/{{version}}/configuration#environment-configuration) `ABLY_KEY`:

```ini
ABLY_KEY=your-ably-key
```

Następnie ustaw zmienną środowiskową `BROADCAST_CONNECTION` na `ably` w pliku `.env` Twojej aplikacji:

```ini
BROADCAST_CONNECTION=ably
```

Na koniec jesteś gotowy, aby zainstalować i skonfigurować [Laravel Echo](#client-side-installation), który będzie odbierał zdarzenia rozgłoszeniowe po stronie klienta.

<a name="client-side-installation"></a>
## Instalacja po stronie klienta

<a name="client-reverb"></a>
### Reverb

[Laravel Echo](https://github.com/laravel/echo) to biblioteka JavaScript, która ułatwia subskrypcję kanałów i nasłuchiwanie zdarzeń rozgłaszanych przez sterownik rozgłaszania po stronie serwera.

Podczas instalacji Laravel Reverb za pomocą polecenia Artisan `install:broadcasting`, szkielet i konfiguracja Reverb i Echo zostaną automatycznie wstrzyknięte do Twojej aplikacji. Jednak jeśli chcesz ręcznie skonfigurować Laravel Echo, możesz to zrobić, postępując zgodnie z poniższymi instrukcjami.

<a name="reverb-client-manual-installation"></a>
#### Instalacja ręczna

Aby ręcznie skonfigurować Laravel Echo dla frontendu aplikacji, najpierw zainstaluj pakiet `pusher-js`, ponieważ Reverb wykorzystuje protokół Pusher dla subskrypcji WebSocket, kanałów i wiadomości:

```shell
npm install --save-dev laravel-echo pusher-js
```

Po zainstalowaniu Echo jesteś gotowy, aby utworzyć nową instancję Echo w JavaScript aplikacji. Świetnym miejscem do tego jest dolna część pliku `resources/js/bootstrap.js`, który jest dołączony do frameworka Laravel:

```js tab=JavaScript
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT ?? 80,
    wssPort: import.meta.env.VITE_REVERB_PORT ?? 443,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    enabledTransports: ['ws', 'wss'],
});
```

```js tab=React
import { configureEcho } from "@laravel/echo-react";

configureEcho({
    broadcaster: "reverb",
    // key: import.meta.env.VITE_REVERB_APP_KEY,
    // wsHost: import.meta.env.VITE_REVERB_HOST,
    // wsPort: import.meta.env.VITE_REVERB_PORT,
    // wssPort: import.meta.env.VITE_REVERB_PORT,
    // forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    // enabledTransports: ['ws', 'wss'],
});
```

```js tab=Vue
import { configureEcho } from "@laravel/echo-vue";

configureEcho({
    broadcaster: "reverb",
    // key: import.meta.env.VITE_REVERB_APP_KEY,
    // wsHost: import.meta.env.VITE_REVERB_HOST,
    // wsPort: import.meta.env.VITE_REVERB_PORT,
    // wssPort: import.meta.env.VITE_REVERB_PORT,
    // forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    // enabledTransports: ['ws', 'wss'],
});
```

Następnie powinieneś skompilować zasoby aplikacji:

```shell
npm run build
```

> [!WARNING]
> Broadcaster Laravel Echo `reverb` wymaga laravel-echo v1.16.0+.

<a name="client-pusher-channels"></a>
### Pusher Channels

[Laravel Echo](https://github.com/laravel/echo) to biblioteka JavaScript, która ułatwia subskrypcję kanałów i nasłuchiwanie zdarzeń rozgłaszanych przez sterownik rozgłaszania po stronie serwera.

Podczas instalacji obsługi rozgłaszania za pomocą polecenia Artisan `install:broadcasting --pusher`, szkielet i konfiguracja Pushera i Echo zostaną automatycznie wstrzyknięte do Twojej aplikacji. Jednak jeśli chcesz ręcznie skonfigurować Laravel Echo, możesz to zrobić, postępując zgodnie z poniższymi instrukcjami.

<a name="pusher-client-manual-installation"></a>
#### Instalacja ręczna

Aby ręcznie skonfigurować Laravel Echo dla frontendu aplikacji, najpierw zainstaluj pakiety `laravel-echo` i `pusher-js`, które wykorzystują protokół Pusher dla subskrypcji WebSocket, kanałów i wiadomości:

```shell
npm install --save-dev laravel-echo pusher-js
```

Po zainstalowaniu Echo jesteś gotowy, aby utworzyć nową instancję Echo w pliku `resources/js/bootstrap.js` swojej aplikacji:

```js tab=JavaScript
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    forceTLS: true
});
```

```js tab=React
import { configureEcho } from "@laravel/echo-react";

configureEcho({
    broadcaster: "pusher",
    // key: import.meta.env.VITE_PUSHER_APP_KEY,
    // cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    // forceTLS: true,
    // wsHost: import.meta.env.VITE_PUSHER_HOST,
    // wsPort: import.meta.env.VITE_PUSHER_PORT,
    // wssPort: import.meta.env.VITE_PUSHER_PORT,
    // enabledTransports: ["ws", "wss"],
});
```

```js tab=Vue
import { configureEcho } from "@laravel/echo-vue";

configureEcho({
    broadcaster: "pusher",
    // key: import.meta.env.VITE_PUSHER_APP_KEY,
    // cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    // forceTLS: true,
    // wsHost: import.meta.env.VITE_PUSHER_HOST,
    // wsPort: import.meta.env.VITE_PUSHER_PORT,
    // wssPort: import.meta.env.VITE_PUSHER_PORT,
    // enabledTransports: ["ws", "wss"],
});
```

Następnie powinieneś zdefiniować odpowiednie wartości dla zmiennych środowiskowych Pushera w pliku `.env` swojej aplikacji. Jeśli tych zmiennych jeszcze nie ma w pliku `.env`, powinieneś je dodać:

```ini
PUSHER_APP_ID="your-pusher-app-id"
PUSHER_APP_KEY="your-pusher-key"
PUSHER_APP_SECRET="your-pusher-secret"
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME="https"
PUSHER_APP_CLUSTER="mt1"

VITE_APP_NAME="${APP_NAME}"
VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
VITE_PUSHER_HOST="${PUSHER_HOST}"
VITE_PUSHER_PORT="${PUSHER_PORT}"
VITE_PUSHER_SCHEME="${PUSHER_SCHEME}"
VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```

Po dostosowaniu konfiguracji Echo do potrzeb aplikacji możesz skompilować zasoby aplikacji:

```shell
npm run build
```

> [!NOTE]
> Aby dowiedzieć się więcej o kompilowaniu zasobów JavaScript aplikacji, zapoznaj się z dokumentacją dotyczącą [Vite](/docs/{{version}}/vite).

<a name="using-an-existing-client-instance"></a>
#### Używanie istniejącej instancji klienta

Jeśli masz już wstępnie skonfigurowaną instancję klienta Pusher Channels, której chciałbyś użyć w Echo, możesz przekazać ją do Echo za pomocą opcji konfiguracyjnej `client`:

```js
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

const options = {
    broadcaster: 'pusher',
    key: import.meta.env.VITE_PUSHER_APP_KEY
}

window.Echo = new Echo({
    ...options,
    client: new Pusher(options.key, options)
});
```

<a name="client-ably"></a>
### Ably

> [!NOTE]
> Poniższa dokumentacja omawia, jak używać Ably w trybie "kompatybilności z Pusherem". Jednak zespół Ably zaleca i utrzymuje broadcaster oraz klienta Echo, który może wykorzystać unikalne możliwości oferowane przez Ably. Więcej informacji na temat używania sterowników utrzymywanych przez Ably znajdziesz w [dokumentacji Laravel broadcastera Ably](https://github.com/ably/laravel-broadcaster).

[Laravel Echo](https://github.com/laravel/echo) to biblioteka JavaScript, która ułatwia subskrypcję kanałów i nasłuchiwanie zdarzeń rozgłaszanych przez sterownik rozgłaszania po stronie serwera.

Podczas instalacji obsługi rozgłaszania za pomocą polecenia Artisan `install:broadcasting --ably`, szkielet i konfiguracja Ably i Echo zostaną automatycznie wstrzyknięte do Twojej aplikacji. Jednak jeśli chcesz ręcznie skonfigurować Laravel Echo, możesz to zrobić, postępując zgodnie z poniższymi instrukcjami.

<a name="ably-client-manual-installation"></a>
#### Instalacja ręczna

Aby ręcznie skonfigurować Laravel Echo dla frontendu aplikacji, najpierw zainstaluj pakiety `laravel-echo` i `pusher-js`, które wykorzystują protokół Pusher dla subskrypcji WebSocket, kanałów i wiadomości:

```shell
npm install --save-dev laravel-echo pusher-js
```

**Przed kontynuowaniem powinieneś włączyć obsługę protokołu Pusher w ustawieniach aplikacji Ably. Możesz włączyć tę funkcję w sekcji "Protocol Adapter Settings" panelu ustawień aplikacji Ably.**

Po zainstalowaniu Echo jesteś gotowy, aby utworzyć nową instancję Echo w pliku `resources/js/bootstrap.js` swojej aplikacji:

```js tab=JavaScript
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_ABLY_PUBLIC_KEY,
    wsHost: 'realtime-pusher.ably.io',
    wsPort: 443,
    disableStats: true,
    encrypted: true,
});
```

```js tab=React
import { configureEcho } from "@laravel/echo-react";

configureEcho({
    broadcaster: "ably",
    // key: import.meta.env.VITE_ABLY_PUBLIC_KEY,
    // wsHost: "realtime-pusher.ably.io",
    // wsPort: 443,
    // disableStats: true,
    // encrypted: true,
});
```

```js tab=Vue
import { configureEcho } from "@laravel/echo-vue";

configureEcho({
    broadcaster: "ably",
    // key: import.meta.env.VITE_ABLY_PUBLIC_KEY,
    // wsHost: "realtime-pusher.ably.io",
    // wsPort: 443,
    // disableStats: true,
    // encrypted: true,
});
```

Możesz zauważyć, że nasza konfiguracja Echo dla Ably odwołuje się do zmiennej środowiskowej `VITE_ABLY_PUBLIC_KEY`. Wartość tej zmiennej powinna być Twoim kluczem publicznym Ably. Twój klucz publiczny to część klucza Ably, która występuje przed znakiem `:`.

Po dostosowaniu konfiguracji Echo do swoich potrzeb możesz skompilować zasoby aplikacji:

```shell
npm run dev
```

> [!NOTE]
> Aby dowiedzieć się więcej o kompilowaniu zasobów JavaScript aplikacji, zapoznaj się z dokumentacją dotyczącą [Vite](/docs/{{version}}/vite).

<a name="concept-overview"></a>
## Przegląd koncepcji

Rozgłaszanie zdarzeń Laravel umożliwia rozgłaszanie serwerowych zdarzeń Laravel do klienckiej aplikacji JavaScript przy użyciu sterownikowego podejścia do WebSockets. Obecnie Laravel jest dostarczany ze sterownikami [Laravel Reverb](https://reverb.laravel.com), [Pusher Channels](https://pusher.com/channels) i [Ably](https://ably.com). Zdarzenia mogą być łatwo konsumowane po stronie klienta przy użyciu pakietu JavaScript [Laravel Echo](#client-side-installation).

Zdarzenia są rozgłaszane przez "kanały", które mogą być określone jako publiczne lub prywatne. Każdy odwiedzający Twoją aplikację może zasubskrybować publiczny kanał bez jakiejkolwiek uwierzytelnienia lub autoryzacji; jednak aby zasubskrybować prywatny kanał, użytkownik musi być uwierzytelniony i autoryzowany do nasłuchiwania na tym kanale.

<a name="using-example-application"></a>
### Korzystanie z przykładowej aplikacji

Przed zagłębieniem się w każdy komponent rozgłaszania zdarzeń przyjrzyjmy się ogólnemu przeglądowi na przykładzie sklepu e-commerce.

W naszej aplikacji załóżmy, że mamy stronę, która pozwala użytkownikom przeglądać status wysyłki ich zamówień. Załóżmy również, że zdarzenie `OrderShipmentStatusUpdated` jest wywoływane, gdy aktualizacja statusu wysyłki jest przetwarzana przez aplikację:

```php
use App\Events\OrderShipmentStatusUpdated;

OrderShipmentStatusUpdated::dispatch($order);
```

<a name="the-shouldbroadcast-interface"></a>
#### Interfejs `ShouldBroadcast`

Kiedy użytkownik ogląda jedno ze swoich zamówień, nie chcemy, aby musiał odświeżać stronę, aby zobaczyć aktualizacje statusu. Zamiast tego chcemy rozgłaszać aktualizacje do aplikacji w momencie ich utworzenia. Więc musimy oznaczyć zdarzenie `OrderShipmentStatusUpdated` interfejsem `ShouldBroadcast`. To poinstruuje Laravela, aby rozgłosić zdarzenie, gdy zostanie wywołane:

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class OrderShipmentStatusUpdated implements ShouldBroadcast
{
    /**
     * The order instance.
     *
     * @var \App\Models\Order
     */
    public $order;
}
```

Interfejs `ShouldBroadcast` wymaga, aby nasze zdarzenie definiowało metodę `broadcastOn`. Ta metoda jest odpowiedzialna za zwracanie kanałów, na których zdarzenie powinno być rozgłaszane. Pusty szkielet tej metody jest już zdefiniowany w generowanych klasach zdarzeń, więc musimy tylko wypełnić jej szczegóły. Chcemy, aby tylko twórca zamówienia mógł oglądać aktualizacje statusu, więc rozgłosimy zdarzenie na prywatnym kanale powiązanym z zamówieniem:

```php
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\PrivateChannel;

/**
 * Get the channel the event should broadcast on.
 */
public function broadcastOn(): Channel
{
    return new PrivateChannel('orders.'.$this->order->id);
}
```

Jeśli chcesz, aby zdarzenie było rozgłaszane na wielu kanałach, możesz zamiast tego zwrócić `tablicę`:

```php
use Illuminate\Broadcasting\PrivateChannel;

/**
 * Get the channels the event should broadcast on.
 *
 * @return array<int, \Illuminate\Broadcasting\Channel>
 */
public function broadcastOn(): array
{
    return [
        new PrivateChannel('orders.'.$this->order->id),
        // ...
    ];
}
```

<a name="example-application-authorizing-channels"></a>
#### Autoryzacja kanałów

Pamiętaj, że użytkownicy muszą być autoryzowani do nasłuchiwania na prywatnych kanałach. Możemy zdefiniować reguły autoryzacji kanałów w pliku `routes/channels.php` naszej aplikacji. W tym przykładzie musimy sprawdzić, czy każdy użytkownik próbujący nasłuchiwać na prywatnym kanale `orders.1` jest faktycznie twórcą zamówienia:

```php
use App\Models\Order;
use App\Models\User;

Broadcast::channel('orders.{orderId}', function (User $user, int $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

Metoda `channel` przyjmuje dwa argumenty: nazwę kanału i callback, który zwraca `true` lub `false` wskazując, czy użytkownik jest autoryzowany do nasłuchiwania na kanale.

Wszystkie callbacki autoryzacji otrzymują aktualnie uwierzytelnionego użytkownika jako swój pierwszy argument oraz wszelkie dodatkowe parametry wieloznaczne jako kolejne argumenty. W tym przykładzie używamy symbolu wieloznacznego `{orderId}`, aby wskazać, że część "ID" nazwy kanału jest wieloznacznikiem.

<a name="listening-for-event-broadcasts"></a>
#### Nasłuchiwanie rozgłoszeń zdarzeń

Następnie wszystko, co pozostało, to nasłuchiwać zdarzenia w naszej aplikacji JavaScript. Możemy to zrobić za pomocą [Laravel Echo](#client-side-installation). Wbudowane hooki React i Vue Laravel Echo ułatwiają rozpoczęcie pracy, a domyślnie wszystkie publiczne właściwości zdarzenia będą zawarte w rozgłoszonym zdarzeniu:

```js tab=React
import { useEcho } from "@laravel/echo-react";

useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);
```

```vue tab=Vue
<script setup lang="ts">
import { useEcho } from "@laravel/echo-vue";

useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);
</script>
```

<a name="defining-broadcast-events"></a>
## Definiowanie zdarzeń rozgłoszeniowych

Aby poinformować Laravela, że dane zdarzenie powinno być rozgłaszane, musisz zaimplementować interfejs `Illuminate\Contracts\Broadcasting\ShouldBroadcast` w klasie zdarzenia. Ten interfejs jest już zaimportowany do wszystkich klas zdarzeń generowanych przez framework, więc możesz go łatwo dodać do swoich zdarzeń.

Interfejs `ShouldBroadcast` wymaga zaimplementowania jednej metody: `broadcastOn`. Metoda `broadcastOn` powinna zwracać kanał lub tablicę kanałów, na których zdarzenie powinno być rozgłaszane. Kanały powinny być instancjami `Channel`, `PrivateChannel` lub `PresenceChannel`. Instancje `Channel` reprezentują publiczne kanały, do których może zasubskrybować każdy użytkownik, podczas gdy `PrivateChannels` i `PresenceChannels` reprezentują prywatne kanały, które wymagają [autoryzacji kanału](#authorizing-channels):

```php
<?php

namespace App\Events;

use App\Models\User;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class ServerCreated implements ShouldBroadcast
{
    use SerializesModels;

    /**
     * Create a new event instance.
     */
    public function __construct(
        public User $user,
    ) {}

    /**
     * Get the channels the event should broadcast on.
     *
     * @return array<int, \Illuminate\Broadcasting\Channel>
     */
    public function broadcastOn(): array
    {
        return [
            new PrivateChannel('user.'.$this->user->id),
        ];
    }
}
```

Po zaimplementowaniu interfejsu `ShouldBroadcast` musisz tylko [wywołać zdarzenie](/docs/{{version}}/events) jak zwykle. Po wywołaniu zdarzenia [zadanie w kolejce](/docs/{{version}}/queues) automatycznie rozgłosi zdarzenie przy użyciu określonego sterownika rozgłaszania.

<a name="broadcast-name"></a>
### Nazwa rozgłoszenia

Domyślnie Laravel rozgłosi zdarzenie przy użyciu nazwy klasy zdarzenia. Jednak możesz dostosować nazwę rozgłoszenia, definiując metodę `broadcastAs` w zdarzeniu:

```php
/**
 * The event's broadcast name.
 */
public function broadcastAs(): string
{
    return 'server.created';
}
```

Jeśli dostosujesz nazwę rozgłoszenia za pomocą metody `broadcastAs`, powinieneś upewnić się, że zarejestrujesz swój listener z wiodącym znakiem `.`. To poinstruuje Echo, aby nie dodawał przedrostka przestrzeni nazw aplikacji do zdarzenia:

```javascript
.listen('.server.created', function (e) {
    // ...
});
```

<a name="broadcast-data"></a>
### Dane rozgłoszenia

Kiedy zdarzenie jest rozgłaszane, wszystkie jego właściwości `public` są automatycznie serializowane i rozgłaszane jako ładunek zdarzenia, co pozwala na dostęp do dowolnych jego publicznych danych z aplikacji JavaScript. Więc na przykład, jeśli Twoje zdarzenie ma jedną publiczną właściwość `$user`, która zawiera model Eloquent, ładunek rozgłoszeniowy zdarzenia będzie:

```json
{
    "user": {
        "id": 1,
        "name": "Patrick Stewart"
        ...
    }
}
```

Jednak jeśli chcesz mieć większą kontrolę nad ładunkiem rozgłoszeniowym, możesz dodać metodę `broadcastWith` do swojego zdarzenia. Ta metoda powinna zwracać tablicę danych, które chcesz rozgłaszać jako ładunek zdarzenia:

```php
/**
 * Get the data to broadcast.
 *
 * @return array<string, mixed>
 */
public function broadcastWith(): array
{
    return ['id' => $this->user->id];
}
```

<a name="broadcast-queue"></a>
### Kolejka rozgłoszenia

Domyślnie każde zdarzenie rozgłoszeniowe jest umieszczane w domyślnej kolejce dla domyślnego połączenia kolejki określonego w pliku konfiguracyjnym `queue.php`. Możesz dostosować połączenie kolejki i nazwę używaną przez broadcaster, definiując właściwości `connection` i `queue` w klasie zdarzenia:

```php
/**
 * The name of the queue connection to use when broadcasting the event.
 *
 * @var string
 */
public $connection = 'redis';

/**
 * The name of the queue on which to place the broadcasting job.
 *
 * @var string
 */
public $queue = 'default';
```

Alternatywnie możesz dostosować nazwę kolejki, definiując metodę `broadcastQueue` w swoim zdarzeniu:

```php
/**
 * The name of the queue on which to place the broadcasting job.
 */
public function broadcastQueue(): string
{
    return 'default';
}
```

Jeśli chcesz rozgłosić swoje zdarzenie przy użyciu kolejki `sync` zamiast domyślnego sterownika kolejki, możesz zaimplementować interfejs `ShouldBroadcastNow` zamiast `ShouldBroadcast`:

```php
<?php

namespace App\Events;

use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class OrderShipmentStatusUpdated implements ShouldBroadcastNow
{
    // ...
}
```

<a name="broadcast-conditions"></a>
### Warunki rozgłaszania

Czasami chcesz rozgłaszać swoje zdarzenie tylko wtedy, gdy spełniony jest określony warunek. Możesz zdefiniować te warunki, dodając metodę `broadcastWhen` do swojej klasy zdarzenia:

```php
/**
 * Determine if this event should broadcast.
 */
public function broadcastWhen(): bool
{
    return $this->order->value > 100;
}
```

<a name="broadcasting-and-database-transactions"></a>
#### Rozgłaszanie i transakcje bazodanowe

Kiedy zdarzenia rozgłoszeniowe są wysyłane w ramach transakcji bazodanowych, mogą być przetworzone przez kolejkę zanim transakcja bazodanowa została zatwierdzona. Gdy to się dzieje, wszelkie aktualizacje wprowadzone do modeli lub rekordów bazy danych podczas transakcji bazodanowej mogą jeszcze nie być odzwierciedlone w bazie danych. Ponadto wszelkie modele lub rekordy bazy danych utworzone w ramach transakcji mogą nie istnieć w bazie danych. Jeśli Twoje zdarzenie zależy od tych modeli, mogą wystąpić nieoczekiwane błędy podczas przetwarzania zadania rozgłaszającego zdarzenie.

Jeśli opcja konfiguracji `after_commit` Twojego połączenia kolejki jest ustawiona na `false`, nadal możesz wskazać, że określone zdarzenie rozgłoszeniowe powinno być wysłane po zatwierdzeniu wszystkich otwartych transakcji bazodanowych, implementując interfejs `ShouldDispatchAfterCommit` w klasie zdarzenia:

```php
<?php

namespace App\Events;

use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
use Illuminate\Queue\SerializesModels;

class ServerCreated implements ShouldBroadcast, ShouldDispatchAfterCommit
{
    use SerializesModels;
}
```

> [!NOTE]
> Aby dowiedzieć się więcej na temat obchodzenia tych problemów, zapoznaj się z dokumentacją dotyczącą [zadań w kolejce i transakcji bazodanowych](/docs/{{version}}/queues#jobs-and-database-transactions).

<a name="authorizing-channels"></a>
## Autoryzacja kanałów

Kanały prywatne wymagają autoryzacji, że obecnie uwierzytelniony użytkownik może faktycznie nasłuchiwać na kanale. Odbywa się to poprzez wykonanie żądania HTTP do aplikacji Laravel z nazwą kanału i pozwolenie aplikacji określić, czy użytkownik może nasłuchiwać na tym kanale. Podczas korzystania z [Laravel Echo](#client-side-installation), żądanie HTTP do autoryzacji subskrypcji kanałów prywatnych będzie wykonywane automatycznie.

Gdy rozgłaszanie jest instalowane, Laravel próbuje automatycznie zarejestrować trasę `/broadcasting/auth` do obsługi żądań autoryzacji. Jeśli Laravel nie zarejestruje tych tras automatycznie, możesz je zarejestrować ręcznie w pliku `/bootstrap/app.php` swojej aplikacji:

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    channels: __DIR__.'/../routes/channels.php',
    health: '/up',
)
```

<a name="defining-authorization-callbacks"></a>
### Definiowanie callbacków autoryzacji

Następnie musimy zdefiniować logikę, która faktycznie określi, czy obecnie uwierzytelniony użytkownik może nasłuchiwać na danym kanale. Odbywa się to w pliku `routes/channels.php`, który został utworzony przez polecenie Artisan `install:broadcasting`. W tym pliku możesz użyć metody `Broadcast::channel` do rejestrowania callbacków autoryzacji kanałów:

```php
use App\Models\User;

Broadcast::channel('orders.{orderId}', function (User $user, int $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

Metoda `channel` przyjmuje dwa argumenty: nazwę kanału i callback, który zwraca `true` lub `false`, wskazując, czy użytkownik jest upoważniony do nasłuchiwania na kanale.

Wszystkie callbacki autoryzacji otrzymują obecnie uwierzytelnionego użytkownika jako pierwszy argument i wszelkie dodatkowe parametry wieloznaczne jako kolejne argumenty. W tym przykładzie używamy znacznika `{orderId}`, aby wskazać, że część "ID" nazwy kanału jest wieloznaczna.

Możesz wyświetlić listę callbacków autoryzacji rozgłaszania swojej aplikacji za pomocą polecenia Artisan `channel:list`:

```shell
php artisan channel:list
```

<a name="authorization-callback-model-binding"></a>
#### Wiązanie modeli w callbackach autoryzacji

Podobnie jak trasy HTTP, trasy kanałów mogą również korzystać z jawnego i niejawnego [wiązania modeli tras](/docs/{{version}}/routing#route-model-binding). Na przykład, zamiast otrzymywać ciąg znaków lub numeryczne ID zamówienia, możesz zażądać rzeczywistej instancji modelu `Order`:

```php
use App\Models\Order;
use App\Models\User;

Broadcast::channel('orders.{order}', function (User $user, Order $order) {
    return $user->id === $order->user_id;
});
```

> [!WARNING]
> W przeciwieństwie do wiązania modeli tras HTTP, wiązanie modeli kanałów nie obsługuje automatycznego [niejawnego zakresowania wiązania modeli](/docs/{{version}}/routing#implicit-model-binding-scoping). Jednak rzadko stanowi to problem, ponieważ większość kanałów można zakresować na podstawie unikalnego klucza podstawowego pojedynczego modelu.

<a name="authorization-callback-authentication"></a>
#### Uwierzytelnianie w callbackach autoryzacji

Prywatne i kanały obecności uwierzytelniają bieżącego użytkownika poprzez domyślną straż uwierzytelniania aplikacji. Jeśli użytkownik nie jest uwierzytelniony, autoryzacja kanału jest automatycznie odrzucana, a callback autoryzacji nigdy nie jest wykonywany. Jednak możesz przypisać wiele niestandardowych straży, które powinny uwierzytelniać przychodzące żądanie, jeśli to konieczne:

```php
Broadcast::channel('channel', function () {
    // ...
}, ['guards' => ['web', 'admin']]);
```

<a name="defining-channel-classes"></a>
### Definiowanie klas kanałów

Jeśli Twoja aplikacja korzysta z wielu różnych kanałów, Twój plik `routes/channels.php` może stać się obszerny. Dlatego, zamiast używać zamknięć do autoryzacji kanałów, możesz użyć klas kanałów. Aby wygenerować klasę kanału, użyj polecenia Artisan `make:channel`. To polecenie umieści nową klasę kanału w katalogu `App/Broadcasting`.

```shell
php artisan make:channel OrderChannel
```

Następnie zarejestruj swój kanał w pliku `routes/channels.php`:

```php
use App\Broadcasting\OrderChannel;

Broadcast::channel('orders.{order}', OrderChannel::class);
```

Na koniec możesz umieścić logikę autoryzacji dla swojego kanału w metodzie `join` klasy kanału. Ta metoda `join` będzie zawierać tę samą logikę, którą zazwyczaj umieściłbyś w zamknięciu autoryzacji kanału. Możesz również korzystać z wiązania modeli kanałów:

```php
<?php

namespace App\Broadcasting;

use App\Models\Order;
use App\Models\User;

class OrderChannel
{
    /**
     * Create a new channel instance.
     */
    public function __construct() {}

    /**
     * Authenticate the user's access to the channel.
     */
    public function join(User $user, Order $order): array|bool
    {
        return $user->id === $order->user_id;
    }
}
```

> [!NOTE]
> Podobnie jak wiele innych klas w Laravel, klasy kanałów będą automatycznie rozwiązywane przez [kontener serwisów](/docs/{{version}}/container). Możesz więc wskazywać wszelkie zależności wymagane przez Twój kanał w jego konstruktorze.

<a name="broadcasting-events"></a>
## Rozgłaszanie zdarzeń

Gdy już zdefiniujesz zdarzenie i oznaczysz je interfejsem `ShouldBroadcast`, wystarczy wywołać zdarzenie za pomocą metody dispatch zdarzenia. Dyspozytor zdarzeń zauważy, że zdarzenie jest oznaczone interfejsem `ShouldBroadcast` i umieści zdarzenie w kolejce do rozgłoszenia:

```php
use App\Events\OrderShipmentStatusUpdated;

OrderShipmentStatusUpdated::dispatch($order);
```

<a name="only-to-others"></a>
### Tylko do innych

Podczas budowania aplikacji wykorzystującej rozgłaszanie zdarzeń, czasami może być konieczne rozgłoszenie zdarzenia do wszystkich subskrybentów danego kanału z wyjątkiem bieżącego użytkownika. Możesz to osiągnąć za pomocą helpera `broadcast` i metody `toOthers`:

```php
use App\Events\OrderShipmentStatusUpdated;

broadcast(new OrderShipmentStatusUpdated($update))->toOthers();
```

Aby lepiej zrozumieć, kiedy możesz chcieć użyć metody `toOthers`, wyobraźmy sobie aplikację do listy zadań, w której użytkownik może utworzyć nowe zadanie, wprowadzając nazwę zadania. Aby utworzyć zadanie, Twoja aplikacja może wykonać żądanie do URL `/task`, które rozgłasza utworzenie zadania i zwraca reprezentację JSON nowego zadania. Gdy Twoja aplikacja JavaScript otrzyma odpowiedź z punktu końcowego, może bezpośrednio wstawić nowe zadanie do swojej listy zadań w następujący sposób:

```js
axios.post('/task', task)
    .then((response) => {
        this.tasks.push(response.data);
    });
```

Jednak pamiętaj, że rozgłaszamy również utworzenie zadania. Jeśli Twoja aplikacja JavaScript również nasłuchuje tego zdarzenia, aby dodawać zadania do listy zadań, będziesz mieć zduplikowane zadania na liście: jedno z punktu końcowego i jedno z rozgłoszenia. Możesz rozwiązać ten problem, używając metody `toOthers`, aby poinstruować rozgłaszacz, aby nie rozgłaszał zdarzenia do bieżącego użytkownika.

> [!WARNING]
> Twoje zdarzenie musi używać cechy `Illuminate\Broadcasting\InteractsWithSockets`, aby móc wywołać metodę `toOthers`.

<a name="only-to-others-configuration"></a>
#### Konfiguracja

Gdy inicjalizujesz instancję Laravel Echo, do połączenia jest przypisywany ID gniazda. Jeśli używasz globalnej instancji [Axios](https://github.com/axios/axios) do wykonywania żądań HTTP z aplikacji JavaScript, ID gniazda będzie automatycznie dołączany do każdego wychodzącego żądania jako nagłówek `X-Socket-ID`. Następnie, gdy wywołasz metodę `toOthers`, Laravel wyodrębni ID gniazda z nagłówka i poinstruuje rozgłaszacz, aby nie rozgłaszał do żadnych połączeń z tym ID gniazda.

Jeśli nie używasz globalnej instancji Axios, musisz ręcznie skonfigurować swoją aplikację JavaScript, aby wysyłała nagłówek `X-Socket-ID` ze wszystkimi wychodzącymi żądaniami. Możesz pobrać ID gniazda za pomocą metody `Echo.socketId`:

```js
var socketId = Echo.socketId();
```

<a name="customizing-the-connection"></a>
### Dostosowywanie połączenia

Jeśli Twoja aplikacja współdziała z wieloma połączeniami rozgłaszania i chcesz rozgłosić zdarzenie przy użyciu rozgłaszacza innego niż domyślny, możesz określić, do którego połączenia przekazać zdarzenie, używając metody `via`:

```php
use App\Events\OrderShipmentStatusUpdated;

broadcast(new OrderShipmentStatusUpdated($update))->via('pusher');
```

Alternatywnie możesz określić połączenie rozgłoszenia zdarzenia, wywołując metodę `broadcastVia` w konstruktorze zdarzenia. Jednak przed tym powinieneś upewnić się, że klasa zdarzenia używa cechy `InteractsWithBroadcasting`:

```php
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithBroadcasting;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class OrderShipmentStatusUpdated implements ShouldBroadcast
{
    use InteractsWithBroadcasting;

    /**
     * Create a new event instance.
     */
    public function __construct()
    {
        $this->broadcastVia('pusher');
    }
}
```

<a name="anonymous-events"></a>
### Zdarzenia anonimowe

Czasami możesz chcieć rozgłosić proste zdarzenie do frontendu swojej aplikacji bez tworzenia dedykowanej klasy zdarzenia. Aby to umożliwić, fasada `Broadcast` pozwala na rozgłaszanie "zdarzeń anonimowych":

```php
Broadcast::on('orders.'.$order->id)->send();
```

Powyższy przykład rozgłosi następujące zdarzenie:

```json
{
    "event": "AnonymousEvent",
    "data": "[]",
    "channel": "orders.1"
}
```

Używając metod `as` i `with`, możesz dostosować nazwę i dane zdarzenia:

```php
Broadcast::on('orders.'.$order->id)
    ->as('OrderPlaced')
    ->with($order)
    ->send();
```

Powyższy przykład rozgłosi zdarzenie podobne do następującego:

```json
{
    "event": "OrderPlaced",
    "data": "{ id: 1, total: 100 }",
    "channel": "orders.1"
}
```

Jeśli chcesz rozgłosić zdarzenie anonimowe na kanale prywatnym lub kanałach obecności, możesz użyć metod `private` i `presence`:

```php
Broadcast::private('orders.'.$order->id)->send();
Broadcast::presence('channels.'.$channel->id)->send();
```

Rozgłaszanie zdarzenia anonimowego przy użyciu metody `send` wysyła zdarzenie do [kolejki](/docs/{{version}}/queues) aplikacji do przetworzenia. Jednak jeśli chcesz rozgłosić zdarzenie natychmiast, możesz użyć metody `sendNow`:

```php
Broadcast::on('orders.'.$order->id)->sendNow();
```

Aby rozgłosić zdarzenie do wszystkich subskrybentów kanału z wyjątkiem aktualnie uwierzytelnionego użytkownika, możesz wywołać metodę `toOthers`:

```php
Broadcast::on('orders.'.$order->id)
    ->toOthers()
    ->send();
```

<a name="rescuing-broadcasts"></a>
### Ratowanie rozgłoszeń

Gdy serwer kolejki aplikacji jest niedostępny lub Laravel napotka błąd podczas rozgłaszania zdarzenia, wyrzucany jest wyjątek, który zwykle powoduje, że użytkownik końcowy widzi błąd aplikacji. Ponieważ rozgłaszanie zdarzeń jest często dodatkową funkcjonalnością aplikacji, możesz zapobiec zakłócaniu doświadczenia użytkownika przez te wyjątki, implementując interfejs `ShouldRescue` w swoich zdarzeniach.

Zdarzenia implementujące interfejs `ShouldRescue` automatycznie wykorzystują [funkcję helpera rescue](/docs/{{version}}/helpers#method-rescue) Laravela podczas prób rozgłaszania. Ten helper wyłapuje wszelkie wyjątki, raportuje je do obsługi wyjątków aplikacji w celu logowania i pozwala aplikacji kontynuować normalne wykonywanie bez przerywania procesu pracy użytkownika:

```php
<?php

namespace App\Events;

use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Contracts\Broadcasting\ShouldRescue;

class ServerCreated implements ShouldBroadcast, ShouldRescue
{
    // ...
}
```

<a name="receiving-broadcasts"></a>
## Odbieranie rozgłoszeń

<a name="listening-for-events"></a>
### Nasłuchiwanie zdarzeń

Po [zainstalowaniu i zinstancjonowaniu Laravel Echo](#client-side-installation), jesteś gotowy, aby zacząć nasłuchiwać zdarzeń, które są rozgłaszane z Twojej aplikacji Laravel. Najpierw użyj metody `channel`, aby uzyskać instancję kanału, następnie wywołaj metodę `listen`, aby nasłuchiwać określone zdarzenie:

```js
Echo.channel(`orders.${this.order.id}`)
    .listen('OrderShipmentStatusUpdated', (e) => {
        console.log(e.order.name);
    });
```

Jeśli chcesz nasłuchiwać zdarzeń na kanale prywatnym, użyj zamiast tego metody `private`. Możesz kontynuować łańcuchowe wywołania metody `listen`, aby nasłuchiwać wielu zdarzeń na jednym kanale:

```js
Echo.private(`orders.${this.order.id}`)
    .listen(/* ... */)
    .listen(/* ... */)
    .listen(/* ... */);
```

<a name="stop-listening-for-events"></a>
#### Zatrzymywanie nasłuchiwania zdarzeń

Jeśli chcesz przestać nasłuchiwać danego zdarzenia bez [opuszczania kanału](#leaving-a-channel), możesz użyć metody `stopListening`:

```js
Echo.private(`orders.${this.order.id}`)
    .stopListening('OrderShipmentStatusUpdated');
```

<a name="leaving-a-channel"></a>
### Opuszczanie kanału

Aby opuścić kanał, możesz wywołać metodę `leaveChannel` na instancji Echo:

```js
Echo.leaveChannel(`orders.${this.order.id}`);
```

Jeśli chcesz opuścić kanał oraz powiązane z nim kanały prywatne i kanały obecności, możesz wywołać metodę `leave`:

```js
Echo.leave(`orders.${this.order.id}`);
```
<a name="namespaces"></a>
### Przestrzenie nazw

Możesz zauważyć w powyższych przykładach, że nie określaliśmy pełnej przestrzeni nazw `App\Events` dla klas zdarzeń. Dzieje się tak, ponieważ Echo automatycznie zakłada, że zdarzenia znajdują się w przestrzeni nazw `App\Events`. Jednak możesz skonfigurować główną przestrzeń nazw podczas instancjonowania Echo, przekazując opcję konfiguracji `namespace`:

```js
window.Echo = new Echo({
    broadcaster: 'pusher',
    // ...
    namespace: 'App.Other.Namespace'
});
```

Alternatywnie możesz prefiksować klasy zdarzeń kropką `.`, subskrybując je za pomocą Echo. Pozwoli Ci to zawsze określać pełną kwalifikowaną nazwę klasy:

```js
Echo.channel('orders')
    .listen('.Namespace\\Event\\Class', (e) => {
        // ...
    });
```

<a name="using-react-or-vue"></a>
### Używanie React lub Vue

Laravel Echo zawiera hooki React i Vue, które ułatwiają nasłuchiwanie zdarzeń. Aby zacząć, wywołaj hook `useEcho`, który służy do nasłuchiwania prywatnych zdarzeń. Hook `useEcho` automatycznie opuści kanały, gdy komponent konsumujący zostanie odmontowany:

```js tab=React
import { useEcho } from "@laravel/echo-react";

useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);
```

```vue tab=Vue
<script setup lang="ts">
import { useEcho } from "@laravel/echo-vue";

useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);
</script>
```

Możesz nasłuchiwać wielu zdarzeń, przekazując tablicę zdarzeń do `useEcho`:

```js
useEcho(
    `orders.${orderId}`,
    ["OrderShipmentStatusUpdated", "OrderShipped"],
    (e) => {
        console.log(e.order);
    },
);
```

Możesz również określić kształt danych ładunku zdarzenia rozgłoszeniowego, zapewniając większe bezpieczeństwo typów i wygodę edycji:

```ts
type OrderData = {
    order: {
        id: number;
        user: {
            id: number;
            name: string;
        };
        created_at: string;
    };
};

useEcho<OrderData>(`orders.${orderId}`, "OrderShipmentStatusUpdated", (e) => {
    console.log(e.order.id);
    console.log(e.order.user.id);
});
```

Hook `useEcho` automatycznie opuści kanały, gdy komponent konsumujący zostanie odmontowany; jednak możesz wykorzystać zwrócone funkcje do ręcznego zatrzymania / rozpoczęcia nasłuchiwania kanałów programowo, gdy zajdzie taka potrzeba:

```js tab=React
import { useEcho } from "@laravel/echo-react";

const { leaveChannel, leave, stopListening, listen } = useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);

// Stop listening without leaving channel...
stopListening();

// Start listening again...
listen();

// Leave channel...
leaveChannel();

// Leave a channel and also its associated private and presence channels...
leave();
```

```vue tab=Vue
<script setup lang="ts">
import { useEcho } from "@laravel/echo-vue";

const { leaveChannel, leave, stopListening, listen } = useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);

// Stop listening without leaving channel...
stopListening();

// Start listening again...
listen();

// Leave channel...
leaveChannel();

// Leave a channel and also its associated private and presence channels...
leave();
</script>
```

<a name="react-vue-connecting-to-public-channels"></a>
#### Podłączanie do kanałów publicznych

Aby połączyć się z kanałem publicznym, możesz użyć hooka `useEchoPublic`:

```js tab=React
import { useEchoPublic } from "@laravel/echo-react";

useEchoPublic("posts", "PostPublished", (e) => {
    console.log(e.post);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoPublic } from "@laravel/echo-vue";

useEchoPublic("posts", "PostPublished", (e) => {
    console.log(e.post);
});
</script>
```

<a name="react-vue-connecting-to-presence-channels"></a>
#### Podłączanie do kanałów obecności

Aby połączyć się z kanałem obecności, możesz użyć hooka `useEchoPresence`:

```js tab=React
import { useEchoPresence } from "@laravel/echo-react";

useEchoPresence("posts", "PostPublished", (e) => {
    console.log(e.post);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoPresence } from "@laravel/echo-vue";

useEchoPresence("posts", "PostPublished", (e) => {
    console.log(e.post);
});
</script>
```

<a name="react-vue-connection-status"></a>
#### Status połączenia

Możesz pobrać bieżący status połączenia WebSocket za pomocą hooka `useConnectionStatus`, który zapewnia reaktywny status, który automatycznie aktualizuje się, gdy zmienia się stan połączenia:

```js tab=React
import { useConnectionStatus } from "@laravel/echo-react";

function ConnectionIndicator() {
    const status = useConnectionStatus();

    return <div>Connection: {status}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useConnectionStatus } from "@laravel/echo-vue";

const status = useConnectionStatus();
</script>

<template>
    <div>Connection: {{ status }}</div>
</template>
```

Możliwe wartości statusu to:

<div class="content-list" markdown="1">

- `connected` - Pomyślnie połączono z serwerem WebSocket.
- `connecting` - Próba początkowego połączenia w toku.
- `reconnecting` - Próba ponownego połączenia po rozłączeniu.
- `disconnected` - Nie połączony i nie próbujący połączyć się ponownie.
- `failed` - Połączenie nie powiodło się i nie będzie ponawiane.

</div>

<a name="presence-channels"></a>
## Kanały obecności

Kanały obecności opierają się na bezpieczeństwie kanałów prywatnych, jednocześnie udostępniając dodatkową funkcję świadomości, kto jest zasubskrybowany na kanale. Ułatwia to budowanie potężnych, współpracujących funkcji aplikacji, takich jak powiadamianie użytkowników, gdy inny użytkownik ogląda tę samą stronę lub wypisywanie uczestników pokoju czatu.

<a name="authorizing-presence-channels"></a>
### Autoryzacja kanałów obecności

Wszystkie kanały obecności są również kanałami prywatnymi; dlatego użytkownicy muszą być [autoryzowani do dostępu do nich](#authorizing-channels). Jednak podczas definiowania callbacków autoryzacji dla kanałów obecności, nie zwrócisz `true`, jeśli użytkownik jest autoryzowany do dołączenia do kanału. Zamiast tego powinieneś zwrócić tablicę danych o użytkowniku.

Dane zwrócone przez callback autoryzacji będą dostępne dla nasłuchujących zdarzeń kanałów obecności w aplikacji JavaScript. Jeśli użytkownik nie jest autoryzowany do dołączenia do kanału obecności, powinieneś zwrócić `false` lub `null`:

```php
use App\Models\User;

Broadcast::channel('chat.{roomId}', function (User $user, int $roomId) {
    if ($user->canJoinRoom($roomId)) {
        return ['id' => $user->id, 'name' => $user->name];
    }
});
```

<a name="joining-presence-channels"></a>
### Dołączanie do kanałów obecności

Aby dołączyć do kanału obecności, możesz użyć metody `join` Echo. Metoda `join` zwróci implementację `PresenceChannel`, która, wraz z udostępnianiem metody `listen`, pozwala subskrybować zdarzenia `here`, `joining` i `leaving`.

```js
Echo.join(`chat.${roomId}`)
    .here((users) => {
        // ...
    })
    .joining((user) => {
        console.log(user.name);
    })
    .leaving((user) => {
        console.log(user.name);
    })
    .error((error) => {
        console.error(error);
    });
```

Callback `here` zostanie wykonany natychmiast po pomyślnym dołączeniu do kanału i otrzyma tablicę zawierającą informacje o użytkownikach dla wszystkich innych użytkowników obecnie zasubskrybowanych na kanale. Metoda `joining` zostanie wykonana, gdy nowy użytkownik dołączy do kanału, podczas gdy metoda `leaving` zostanie wykonana, gdy użytkownik opuści kanał. Metoda `error` zostanie wykonana, gdy punkt końcowy uwierzytelniania zwróci kod statusu HTTP inny niż 200 lub gdy wystąpi problem z parsowaniem zwróconego JSON.

<a name="broadcasting-to-presence-channels"></a>
### Rozgłaszanie do kanałów obecności

Kanały obecności mogą odbierać zdarzenia tak samo jak kanały publiczne lub prywatne. Używając przykładu pokoju czatu, możemy chcieć rozgłosić zdarzenia `NewMessage` do kanału obecności pokoju. Aby to zrobić, zwrócimy instancję `PresenceChannel` z metody `broadcastOn` zdarzenia:

```php
/**
 * Get the channels the event should broadcast on.
 *
 * @return array<int, \Illuminate\Broadcasting\Channel>
 */
public function broadcastOn(): array
{
    return [
        new PresenceChannel('chat.'.$this->message->room_id),
    ];
}
```

Podobnie jak w przypadku innych zdarzeń, możesz użyć helpera `broadcast` i metody `toOthers`, aby wykluczyć bieżącego użytkownika z otrzymania rozgłoszenia:

```php
broadcast(new NewMessage($message));

broadcast(new NewMessage($message))->toOthers();
```

Jak to typowe dla innych typów zdarzeń, możesz nasłuchiwać zdarzeń wysyłanych do kanałów obecności za pomocą metody `listen` Echo:

```js
Echo.join(`chat.${roomId}`)
    .here(/* ... */)
    .joining(/* ... */)
    .leaving(/* ... */)
    .listen('NewMessage', (e) => {
        // ...
    });
```

<a name="model-broadcasting"></a>
## Model Broadcasting

> [!WARNING]
> Przed przeczytaniem poniższej dokumentacji o rozgłaszaniu modeli, zalecamy zapoznanie się z ogólnymi koncepcjami usług rozgłaszania modeli Laravel oraz sposobem ręcznego tworzenia i nasłuchiwania zdarzeń rozgłoszeniowych.

Często zdarza się rozgłaszać zdarzenia, gdy [modele Eloquent](/docs/{{version}}/eloquent) Twojej aplikacji są tworzone, aktualizowane lub usuwane. Oczywiście można to łatwo osiągnąć, ręcznie [definiując niestandardowe zdarzenia dla zmian stanu modeli Eloquent](/docs/{{version}}/eloquent#events) i oznaczając te zdarzenia interfejsem `ShouldBroadcast`.

Jednak jeśli nie używasz tych zdarzeń do żadnych innych celów w swojej aplikacji, tworzenie klas zdarzeń wyłącznie w celu ich rozgłaszania może być uciążliwe. Aby temu zaradzić, Laravel pozwala wskazać, że model Eloquent powinien automatycznie rozgłaszać swoje zmiany stanu.

Aby rozpocząć, Twój model Eloquent powinien używać cechy `Illuminate\Database\Eloquent\BroadcastsEvents`. Dodatkowo model powinien definiować metodę `broadcastOn`, która zwróci tablicę kanałów, na których zdarzenia modelu powinny być rozgłaszane:

```php
<?php

namespace App\Models;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Database\Eloquent\BroadcastsEvents;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Post extends Model
{
    use BroadcastsEvents, HasFactory;

    /**
     * Get the user that the post belongs to.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Get the channels that model events should broadcast on.
     *
     * @return array<int, \Illuminate\Broadcasting\Channel|\Illuminate\Database\Eloquent\Model>
     */
    public function broadcastOn(string $event): array
    {
        return [$this, $this->user];
    }
}
```

Gdy Twój model zawiera tę cechę i definiuje swoje kanały rozgłaszania, automatycznie rozpocznie rozgłaszanie zdarzeń, gdy instancja modelu jest tworzona, aktualizowana, usuwana, przenoszona do kosza lub przywracana.

Dodatkowo możesz zauważyć, że metoda `broadcastOn` otrzymuje argument typu string `$event`. Ten argument zawiera typ zdarzenia, które wystąpiło na modelu i będzie miał wartość `created`, `updated`, `deleted`, `trashed` lub `restored`. Sprawdzając wartość tej zmiennej, możesz określić, na które kanały (jeśli w ogóle) model powinien rozgłaszać dla konkretnego zdarzenia:

```php
/**
 * Get the channels that model events should broadcast on.
 *
 * @return array<string, array<int, \Illuminate\Broadcasting\Channel|\Illuminate\Database\Eloquent\Model>>
 */
public function broadcastOn(string $event): array
{
    return match ($event) {
        'deleted' => [],
        default => [$this, $this->user],
    };
}
```

<a name="customizing-model-broadcasting-event-creation"></a>
#### Dostosowywanie tworzenia zdarzeń rozgłaszania modeli

Od czasu do czasu możesz chcieć dostosować sposób, w jaki Laravel tworzy podstawowe zdarzenie rozgłaszania modelu. Możesz to osiągnąć, definiując metodę `newBroadcastableEvent` w swoim modelu Eloquent. Ta metoda powinna zwracać instancję `Illuminate\Database\Eloquent\BroadcastableModelEventOccurred`:

```php
use Illuminate\Database\Eloquent\BroadcastableModelEventOccurred;

/**
 * Create a new broadcastable model event for the model.
 */
protected function newBroadcastableEvent(string $event): BroadcastableModelEventOccurred
{
    return (new BroadcastableModelEventOccurred(
        $this, $event
    ))->dontBroadcastToCurrentUser();
}
```

<a name="model-broadcasting-conventions"></a>
### Konwencje rozgłaszania modeli

<a name="model-broadcasting-channel-conventions"></a>
#### Konwencje kanałów

Jak możesz zauważyć, metoda `broadcastOn` w powyższym przykładzie modelu nie zwracała instancji `Channel`. Zamiast tego modele Eloquent zostały zwrócone bezpośrednio. Jeśli instancja modelu Eloquent jest zwracana przez metodę `broadcastOn` Twojego modelu (lub jest zawarta w tablicy zwróconej przez metodę), Laravel automatycznie utworzy instancję kanału prywatnego dla modelu, używając nazwy klasy modelu i identyfikatora klucza podstawowego jako nazwy kanału.

Tak więc model `App\Models\User` z `id` równym `1` zostanie przekonwertowany na instancję `Illuminate\Broadcasting\PrivateChannel` z nazwą `App.Models.User.1`. Oczywiście, oprócz zwracania instancji modeli Eloquent z metody `broadcastOn` Twojego modelu, możesz zwracać pełne instancje `Channel`, aby mieć pełną kontrolę nad nazwami kanałów modelu:

```php
use Illuminate\Broadcasting\PrivateChannel;

/**
 * Get the channels that model events should broadcast on.
 *
 * @return array<int, \Illuminate\Broadcasting\Channel>
 */
public function broadcastOn(string $event): array
{
    return [
        new PrivateChannel('user.'.$this->id)
    ];
}
```

Jeśli planujesz jawnie zwrócić instancję kanału z metody `broadcastOn` Twojego modelu, możesz przekazać instancję modelu Eloquent do konstruktora kanału. Przy tym Laravel użyje konwencji kanałów modeli omówionych powyżej, aby przekonwertować model Eloquent na ciąg nazwy kanału:

```php
return [new Channel($this->user)];
```

Jeśli musisz określić nazwę kanału modelu, możesz wywołać metodę `broadcastChannel` na dowolnej instancji modelu. Na przykład ta metoda zwraca ciąg `App.Models.User.1` dla modelu `App\Models\User` z `id` równym `1`:

```php
$user->broadcastChannel();
```

<a name="model-broadcasting-event-conventions"></a>
#### Konwencje zdarzeń

Ponieważ zdarzenia rozgłaszania modeli nie są powiązane z "rzeczywistym" zdarzeniem w katalogu `App\Events` Twojej aplikacji, są im przypisywane nazwy i ładunki na podstawie konwencji. Konwencją Laravela jest rozgłaszanie zdarzenia przy użyciu nazwy klasy modelu (bez przestrzeni nazw) oraz nazwy zdarzenia modelu, które wywołało rozgłoszenie.

Tak więc na przykład aktualizacja modelu `App\Models\Post` rozgłosiłaby zdarzenie do aplikacji klienckiej jako `PostUpdated` z następującym ładunkiem:

```json
{
    "model": {
        "id": 1,
        "title": "My first post"
        ...
    },
    ...
    "socket": "someSocketId"
}
```

Usunięcie modelu `App\Models\User` rozgłosiłoby zdarzenie o nazwie `UserDeleted`.

Jeśli chcesz, możesz zdefiniować niestandardową nazwę rozgłoszenia i ładunek, dodając metodę `broadcastAs` i `broadcastWith` do swojego modelu. Te metody otrzymują nazwę zdarzenia / operacji modelu, która ma miejsce, umożliwiając dostosowanie nazwy i ładunku zdarzenia dla każdej operacji modelu. Jeśli `null` zostanie zwrócone z metody `broadcastAs`, Laravel użyje konwencji nazw zdarzeń rozgłaszania modeli omówionych powyżej podczas rozgłaszania zdarzenia:

```php
/**
 * The model event's broadcast name.
 */
public function broadcastAs(string $event): string|null
{
    return match ($event) {
        'created' => 'post.created',
        default => null,
    };
}

/**
 * Get the data to broadcast for the model.
 *
 * @return array<string, mixed>
 */
public function broadcastWith(string $event): array
{
    return match ($event) {
        'created' => ['title' => $this->title],
        default => ['model' => $this],
    };
}
```

<a name="listening-for-model-broadcasts"></a>
### Nasłuchiwanie rozgłoszeń modeli

Gdy dodasz cechę `BroadcastsEvents` do swojego modelu i zdefiniujesz metodę `broadcastOn` modelu, jesteś gotowy, aby zacząć nasłuchiwać rozgłaszanych zdarzeń modeli w aplikacji klienckiej. Przed rozpoczęciem możesz chcieć zapoznać się z kompletną dokumentacją dotyczącą [nasłuchiwania zdarzeń](#listening-for-events).

Najpierw użyj metody `private`, aby pobrać instancję kanału, a następnie wywołaj metodę `listen`, aby nasłuchiwać określonego zdarzenia. Zazwyczaj nazwa kanału przekazana do metody `private` powinna odpowiadać [konwencjom rozgłaszania modeli](#model-broadcasting-conventions) Laravela.

Gdy uzyskasz instancję kanału, możesz użyć metody `listen`, aby nasłuchiwać konkretnego zdarzenia. Ponieważ zdarzenia rozgłaszania modeli nie są powiązane z "rzeczywistym" zdarzeniem w katalogu `App\Events` Twojej aplikacji, [nazwa zdarzenia](#model-broadcasting-event-conventions) musi być poprzedzona kropką `.`, aby wskazać, że nie należy do konkretnej przestrzeni nazw. Każde zdarzenie rozgłaszania modelu ma właściwość `model`, która zawiera wszystkie rozgłaszalne właściwości modelu:

```js
Echo.private(`App.Models.User.${this.user.id}`)
    .listen('.UserUpdated', (e) => {
        console.log(e.model);
    });
```

<a name="model-broadcasts-with-react-or-vue"></a>
#### Używanie React lub Vue

Jeśli używasz React lub Vue, możesz użyć dołączonego hooka `useEchoModel` Laravel Echo, aby łatwo nasłuchiwać rozgłoszeń modeli:

```js tab=React
import { useEchoModel } from "@laravel/echo-react";

useEchoModel("App.Models.User", userId, ["UserUpdated"], (e) => {
    console.log(e.model);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoModel } from "@laravel/echo-vue";

useEchoModel("App.Models.User", userId, ["UserUpdated"], (e) => {
    console.log(e.model);
});
</script>
```

Możesz również określić kształt danych ładunku zdarzenia modelu, zapewniając większe bezpieczeństwo typów i wygodę edycji:

```ts
type User = {
    id: number;
    name: string;
    email: string;
};

useEchoModel<User, "App.Models.User">("App.Models.User", userId, ["UserUpdated"], (e) => {
    console.log(e.model.id);
    console.log(e.model.name);
});
```

<a name="client-events"></a>
## Zdarzenia klienckie

> [!NOTE]
> Podczas korzystania z [Pusher Channels](https://pusher.com/channels), musisz włączyć opcję "Client Events" w sekcji "App Settings" [panelu aplikacji](https://dashboard.pusher.com/), aby móc wysyłać zdarzenia klienckie.

Czasami możesz chcieć rozgłosić zdarzenie do innych podłączonych klientów bez wcale dotykania Twojej aplikacji Laravel. Może to być szczególnie przydatne w przypadku takich rzeczy jak powiadomienia o "pisaniu", w których chcesz powiadomić użytkowników aplikacji, że inny użytkownik pisze wiadomość na danym ekranie.

Aby rozgłaszać zdarzenia klienckie, możesz użyć metody `whisper` Echo:

```js tab=JavaScript
Echo.private(`chat.${roomId}`)
    .whisper('typing', {
        name: this.user.name
    });
```

```js tab=React
import { useEcho } from "@laravel/echo-react";

const { channel } = useEcho(`chat.${roomId}`, ['update'], (e) => {
    console.log('Chat event received:', e);
});

channel().whisper('typing', { name: user.name });
```

```vue tab=Vue
<script setup lang="ts">
import { useEcho } from "@laravel/echo-vue";

const { channel } = useEcho(`chat.${roomId}`, ['update'], (e) => {
    console.log('Chat event received:', e);
});

channel().whisper('typing', { name: user.name });
</script>
```

Aby nasłuchiwać zdarzeń klienckich, możesz użyć metody `listenForWhisper`:

```js tab=JavaScript
Echo.private(`chat.${roomId}`)
    .listenForWhisper('typing', (e) => {
        console.log(e.name);
    });
```

```js tab=React
import { useEcho } from "@laravel/echo-react";

const { channel } = useEcho(`chat.${roomId}`, ['update'], (e) => {
    console.log('Chat event received:', e);
});

channel().listenForWhisper('typing', (e) => {
    console.log(e.name);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEcho } from "@laravel/echo-vue";

const { channel } = useEcho(`chat.${roomId}`, ['update'], (e) => {
    console.log('Chat event received:', e);
});

channel().listenForWhisper('typing', (e) => {
    console.log(e.name);
});
</script>
```

<a name="notifications"></a>
## Powiadomienia

Łącząc rozgłaszanie zdarzeń z [powiadomieniami](/docs/{{version}}/notifications), Twoja aplikacja JavaScript może odbierać nowe powiadomienia w momencie ich wystąpienia bez konieczności odświeżania strony. Przed rozpoczęciem upewnij się, że przeczytałeś dokumentację dotyczącą korzystania z [kanału powiadomień rozgłoszeniowych](/docs/{{version}}/notifications#broadcast-notifications).

Gdy skonfigurujesz powiadomienie do korzystania z kanału rozgłoszeniowego, możesz nasłuchiwać zdarzeń rozgłoszeniowych za pomocą metody `notification` Echo. Pamiętaj, że nazwa kanału powinna odpowiadać nazwie klasy encji otrzymującej powiadomienia:

```js tab=JavaScript
Echo.private(`App.Models.User.${userId}`)
    .notification((notification) => {
        console.log(notification.type);
    });
```

```js tab=React
import { useEchoModel } from "@laravel/echo-react";

const { channel } = useEchoModel('App.Models.User', userId);

channel().notification((notification) => {
    console.log(notification.type);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoModel } from "@laravel/echo-vue";

const { channel } = useEchoModel('App.Models.User', userId);

channel().notification((notification) => {
    console.log(notification.type);
});
</script>
```

W tym przykładzie wszystkie powiadomienia wysłane do instancji `App\Models\User` przez kanał `broadcast` byłyby odbierane przez callback. Callback autoryzacji kanału dla kanału `App.Models.User.{id}` jest zawarty w pliku `routes/channels.php` Twojej aplikacji.

<a name="stop-listening-for-notifications"></a>
#### Zatrzymanie nasłuchiwania powiadomień

Jeśli chcesz przestać nasłuchiwać powiadomień bez [opuszczania kanału](#leaving-a-channel), możesz użyć metody `stopListeningForNotification`:

```js
const callback = (notification) => {
    console.log(notification.type);
}

// Start listening...
Echo.private(`App.Models.User.${userId}`)
    .notification(callback);

// Stop listening (callback must be the same)...
Echo.private(`App.Models.User.${userId}`)
    .stopListeningForNotification(callback);
```
