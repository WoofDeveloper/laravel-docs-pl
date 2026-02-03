# Laravel Cashier (Paddle)

- [Wprowadzenie](#introduction)
- [Aktualizacja Cashier](#upgrading-cashier)
- [Instalacja](#installation)
    - [Paddle Sandbox](#paddle-sandbox)
- [Konfiguracja](#configuration)
    - [Model rozliczeniowy](#billable-model)
    - [Klucze API](#api-keys)
    - [Paddle JS](#paddle-js)
    - [Konfiguracja waluty](#currency-configuration)
    - [Nadpisywanie domyślnych modeli](#overriding-default-models)
- [Szybki start](#quickstart)
    - [Sprzedaż produktów](#quickstart-selling-products)
    - [Sprzedaż subskrypcji](#quickstart-selling-subscriptions)
- [Sesje kasy](#checkout-sessions)
    - [Checkout nakładkowy](#overlay-checkout)
    - [Checkout wbudowany](#inline-checkout)
    - [Checkout dla gości](#guest-checkouts)
- [Podgląd cen](#price-previews)
    - [Podgląd cen dla klientów](#customer-price-previews)
    - [Rabaty](#price-discounts)
- [Klienci](#customers)
    - [Domyślne dane klientów](#customer-defaults)
    - [Pobieranie klientów](#retrieving-customers)
    - [Tworzenie klientów](#creating-customers)
- [Subskrypcje](#subscriptions)
    - [Tworzenie subskrypcji](#creating-subscriptions)
    - [Sprawdzanie statusu subskrypcji](#checking-subscription-status)
    - [Jednorazowe opłaty subskrypcyjne](#subscription-single-charges)
    - [Aktualizacja informacji o płatności](#updating-payment-information)
    - [Zmiana planów](#changing-plans)
    - [Ilość subskrypcji](#subscription-quantity)
    - [Subskrypcje z wieloma produktami](#subscriptions-with-multiple-products)
    - [Wiele subskrypcji](#multiple-subscriptions)
    - [Wstrzymywanie subskrypcji](#pausing-subscriptions)
    - [Anulowanie subskrypcji](#canceling-subscriptions)
- [Okresy próbne subskrypcji](#subscription-trials)
    - [Z metodą płatności z góry](#with-payment-method-up-front)
    - [Bez metody płatności z góry](#without-payment-method-up-front)
    - [Przedłużanie lub aktywacja okresu próbnego](#extend-or-activate-a-trial)
- [Obsługa webhooków Paddle](#handling-paddle-webhooks)
    - [Definiowanie obsługi zdarzeń webhook](#defining-webhook-event-handlers)
    - [Weryfikacja podpisów webhook](#verifying-webhook-signatures)
- [Pojedyncze opłaty](#single-charges)
    - [Pobieranie opłat za produkty](#charging-for-products)
    - [Zwroty transakcji](#refunding-transactions)
    - [Uznanie transakcji](#crediting-transactions)
- [Transakcje](#transactions)
    - [Przeszłe i nadchodzące płatności](#past-and-upcoming-payments)
- [Testowanie](#testing)

<a name="introduction"></a>
## Wprowadzenie

> [!WARNING]
> Ta dokumentacja dotyczy integracji Cashier Paddle 2.x z Paddle Billing. Jeśli nadal używasz Paddle Classic, powinieneś użyć [Cashier Paddle 1.x](https://github.com/laravel/cashier-paddle/tree/1.x).

[Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle) zapewnia wyrazisty, płynny interfejs do usług rozliczeniowych subskrypcji [Paddle](https://paddle.com). Obsługuje prawie cały boilerplate kod rozliczeniowy subskrypcji, którego się obawiasz. Oprócz podstawowego zarządzania subskrypcjami, Cashier może obsługiwać: zamianę subskrypcji, "ilości" subskrypcji, wstrzymywanie subskrypcji, okresy karencji po anulowaniu i wiele więcej.

Przed zagłębieniem się w Cashier Paddle zalecamy również zapoznanie się z [przewodnikami po koncepcjach](https://developer.paddle.com/concepts/overview) Paddle i [dokumentacją API](https://developer.paddle.com/api-reference/overview).

<a name="upgrading-cashier"></a>
## Aktualizacja Cashier

Podczas aktualizacji do nowej wersji Cashier ważne jest, aby dokładnie przejrzeć [przewodnik po aktualizacji](https://github.com/laravel/cashier-paddle/blob/master/UPGRADE.md).

<a name="installation"></a>
## Instalacja

Najpierw zainstaluj pakiet Cashier dla Paddle za pomocą menedżera pakietów Composer:

```shell
composer require laravel/cashier-paddle
```

Następnie opublikuj pliki migracji Cashier za pomocą polecenia Artisan `vendor:publish`:

```shell
php artisan vendor:publish --tag="cashier-migrations"
```

Następnie uruchom migracje bazy danych swojej aplikacji. Migracje Cashier utworzą nową tabelę `customers`. Dodatkowo zostaną utworzone nowe tabele `subscriptions` i `subscription_items` do przechowywania wszystkich subskrypcji klientów. Na koniec zostanie utworzona nowa tabela `transactions` do przechowywania wszystkich transakcji Paddle związanych z Twoimi klientami:

```shell
php artisan migrate
```

> [!WARNING]
> Aby upewnić się, że Cashier prawidłowo obsługuje wszystkie zdarzenia Paddle, pamiętaj o [skonfigurowaniu obsługi webhooków Cashier](#handling-paddle-webhooks).

<a name="paddle-sandbox"></a>
### Paddle Sandbox

Podczas lokalnego i stagingowego rozwoju powinieneś [zarejestrować konto Paddle Sandbox](https://sandbox-login.paddle.com/signup). To konto zapewni Ci środowisko piaskownicy do testowania i rozwijania aplikacji bez dokonywania rzeczywistych płatności. Możesz użyć [testowych numerów kart](https://developer.paddle.com/concepts/payment-methods/credit-debit-card#test-payment-method) Paddle do symulacji różnych scenariuszy płatności.

Podczas korzystania ze środowiska Paddle Sandbox powinieneś ustawić zmienną środowiskową `PADDLE_SANDBOX` na `true` w pliku `.env` swojej aplikacji:

```ini
PADDLE_SANDBOX=true
```

Po zakończeniu rozwijania aplikacji możesz [ubiegać się o konto sprzedawcy Paddle](https://paddle.com). Zanim Twoja aplikacja zostanie umieszczona w produkcji, Paddle będzie musiał zatwierdzić domenę Twojej aplikacji.

<a name="configuration"></a>
## Konfiguracja

<a name="billable-model"></a>
### Model rozliczeniowy

Przed użyciem Cashier musisz dodać trait `Billable` do definicji modelu użytkownika. Ten trait dostarcza różne metody, które pozwalają wykonywać typowe zadania rozliczeniowe, takie jak tworzenie subskrypcji i aktualizacja informacji o metodzie płatności:

```php
use Laravel\Paddle\Billable;

class User extends Authenticatable
{
    use Billable;
}
```

Jeśli masz jednostki rozliczeniowe, które nie są użytkownikami, możesz również dodać trait do tych klas:

```php
use Illuminate\Database\Eloquent\Model;
use Laravel\Paddle\Billable;

class Team extends Model
{
    use Billable;
}
```

<a name="api-keys"></a>
### Klucze API

Następnie powinieneś skonfigurować klucze Paddle w pliku `.env` swojej aplikacji. Możesz pobrać klucze API Paddle z panelu kontrolnego Paddle:

```ini
PADDLE_CLIENT_SIDE_TOKEN=your-paddle-client-side-token
PADDLE_API_KEY=your-paddle-api-key
PADDLE_RETAIN_KEY=your-paddle-retain-key
PADDLE_WEBHOOK_SECRET="your-paddle-webhook-secret"
PADDLE_SANDBOX=true
```

Zmienna środowiskowa `PADDLE_SANDBOX` powinna być ustawiona na `true`, gdy korzystasz ze [środowiska Paddle Sandbox](#paddle-sandbox). Zmienna `PADDLE_SANDBOX` powinna być ustawiona na `false`, jeśli wdrażasz aplikację do produkcji i korzystasz z aktywnego środowiska sprzedawcy Paddle.

`PADDLE_RETAIN_KEY` jest opcjonalny i powinien być ustawiony tylko wtedy, gdy używasz Paddle z [Retain](https://developer.paddle.com/concepts/retain/overview).

<a name="paddle-js"></a>
### Paddle JS

Paddle polega na własnej bibliotece JavaScript do inicjowania widgetu kasy Paddle. Możesz załadować bibliotekę JavaScript umieszczając dyrektywę Blade `@paddleJS` tuż przed zamykającym tagiem `</head>` w układzie aplikacji:

```blade
<head>
    ...

    @paddleJS
</head>
```

<a name="currency-configuration"></a>
### Konfiguracja waluty

Możesz określić locale, które będzie używane podczas formatowania wartości pieniężnych do wyświetlania na fakturach. Wewnętrznie Cashier wykorzystuje [klasę PHP `NumberFormatter`](https://www.php.net/manual/en/class.numberformatter.php) do ustawienia locale waluty:

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> [!WARNING]
> Aby używać locale innych niż `en`, upewnij się, że rozszerzenie PHP `ext-intl` jest zainstalowane i skonfigurowane na serwerze.

<a name="overriding-default-models"></a>
### Nadpisywanie domyślnych modeli

Możesz swobodnie rozszerzać modele używane wewnętrznie przez Cashier, definiując własny model i rozszerzając odpowiedni model Cashier:

```php
use Laravel\Paddle\Subscription as CashierSubscription;

class Subscription extends CashierSubscription
{
    // ...
}
```

Po zdefiniowaniu modelu możesz polecić Cashier używać niestandardowego modelu poprzez klasę `Laravel\Paddle\Cashier`. Zazwyczaj powinieneś poinformować Cashier o niestandardowych modelach w metodzie `boot` klasy `App\Providers\AppServiceProvider` swojej aplikacji:

```php
use App\Models\Cashier\Subscription;
use App\Models\Cashier\Transaction;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::useSubscriptionModel(Subscription::class);
    Cashier::useTransactionModel(Transaction::class);
}
```

<a name="quickstart"></a>
## Szybki start

<a name="quickstart-selling-products"></a>
### Sprzedaż produktów

> [!NOTE]
> Przed skorzystaniem z Paddle Checkout powinieneś zdefiniować Produkty ze stałymi cenami w panelu Paddle. Dodatkowo powinieneś [skonfigurować obsługę webhooków Paddle](#handling-paddle-webhooks).

Oferowanie produktów i rozliczeń subskrypcyjnych za pośrednictwem aplikacji może być onieśmielające. Jednak dzięki Cashier i [Paddle's Checkout Overlay](https://developer.paddle.com/concepts/sell/overlay-checkout) możesz łatwo budować nowoczesne, solidne integracje płatności.

Aby obciążyć klientów za produkty jednorazowe, niepowtarzające się, wykorzystamy Cashier do pobierania opłat od klientów za pomocą Paddle's Checkout Overlay, gdzie podadzą szczegóły płatności i potwierdzą zakup. Po dokonaniu płatności za pośrednictwem Checkout Overlay, klient zostanie przekierowany na wybrany przez Ciebie adres URL sukcesu w aplikacji:

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $request->user()->checkout('pri_deluxe_album')
        ->returnTo(route('dashboard'));

    return view('buy', ['checkout' => $checkout]);
})->name('checkout');
```

Jak widać w powyższym przykładzie, wykorzystamy dostarczaną przez Cashier metodę `checkout` do utworzenia obiektu kasy, aby zaprezentować klientowi Paddle Checkout Overlay dla danego "identyfikatora ceny". Przy użyciu Paddle, "ceny" odnoszą się do [zdefiniowanych cen dla konkretnych produktów](https://developer.paddle.com/build/products/create-products-prices).

Jeśli to konieczne, metoda `checkout` automatycznie utworzy klienta w Paddle i połączy ten rekord klienta Paddle z odpowiednim użytkownikiem w bazie danych aplikacji. Po zakończeniu sesji kasy klient zostanie przekierowany na dedykowaną stronę sukcesu, gdzie możesz wyświetlić wiadomość informacyjną dla klienta.

W widoku `buy` dodamy przycisk do wyświetlenia Checkout Overlay. Komponent Blade `paddle-button` jest dołączony do Cashier Paddle; możesz jednak również [ręcznie renderować checkout nakładkowy](#manually-rendering-an-overlay-checkout):

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Buy Product
</x-paddle-button>
```

<a name="providing-meta-data-to-paddle-checkout"></a>
#### Dostarczanie meta danych do Paddle Checkout

Podczas sprzedaży produktów powszechne jest śledzenie ukończonych zamówień i zakupionych produktów za pomocą modeli `Cart` i `Order` zdefiniowanych przez własną aplikację. Podczas przekierowywania klientów do Paddle's Checkout Overlay w celu ukończenia zakupu może być konieczne podanie istniejącego identyfikatora zamówienia, aby móc powiązać ukończony zakup z odpowiednim zamówieniem, gdy klient zostanie przekierowany z powrotem do aplikacji.

Aby to osiągnąć, możesz przekazać tablicę niestandardowych danych do metody `checkout`. Wyobraźmy sobie, że oczekujące `Order` jest tworzone w naszej aplikacji, gdy użytkownik rozpoczyna proces kasy. Pamiętaj, że modele `Cart` i `Order` w tym przykładzie są ilustracyjne i nie są dostarczane przez Cashier. Możesz swobodnie implementować te koncepcje w zależności od potrzeb własnej aplikacji:

```php
use App\Models\Cart;
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/cart/{cart}/checkout', function (Request $request, Cart $cart) {
    $order = Order::create([
        'cart_id' => $cart->id,
        'price_ids' => $cart->price_ids,
        'status' => 'incomplete',
    ]);

    $checkout = $request->user()->checkout($order->price_ids)
        ->customData(['order_id' => $order->id]);

    return view('billing', ['checkout' => $checkout]);
})->name('checkout');
```

Jak widać w powyższym przykładzie, gdy użytkownik rozpoczyna proces kasy, podamy wszystkie powiązane identyfikatory cen Paddle koszyka/zamówienia do metody `checkout`. Oczywiście Twoja aplikacja jest odpowiedzialna za powiązanie tych elementów z "koszykiem" lub zamówieniem, gdy klient je dodaje. Przekazujemy również ID zamówienia do Paddle Checkout Overlay za pomocą metody `customData`.

Oczywiście prawdopodobnie będziesz chciał oznaczyć zamówienie jako "ukończone", gdy klient zakończy proces kasy. Aby to osiągnąć, możesz nasłuchiwać webhooków wysyłanych przez Paddle i wywoływanych przez zdarzenia w Cashier, aby przechowywać informacje o zamówieniu w bazie danych.

Aby rozpocząć, nasłuchuj zdarzenia `TransactionCompleted` wysyłanego przez Cashier. Zazwyczaj powinieneś zarejestrować nasłuchiwacz zdarzeń w metodzie `boot` klasy `AppServiceProvider` swojej aplikacji:

```php
use App\Listeners\CompleteOrder;
use Illuminate\Support\Facades\Event;
use Laravel\Paddle\Events\TransactionCompleted;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(TransactionCompleted::class, CompleteOrder::class);
}
```

W tym przykładzie nasłuchiwacz `CompleteOrder` może wyglądać następująco:

```php
namespace App\Listeners;

use App\Models\Order;
use Laravel\Paddle\Cashier;
use Laravel\Paddle\Events\TransactionCompleted;

class CompleteOrder
{
    /**
     * Handle the incoming Cashier webhook event.
     */
    public function handle(TransactionCompleted $event): void
    {
        $orderId = $event->payload['data']['custom_data']['order_id'] ?? null;

        $order = Order::findOrFail($orderId);

        $order->update(['status' => 'completed']);
    }
}
```

Zapoznaj się z dokumentacją Paddle, aby uzyskać więcej informacji o [danych zawartych w zdarzeniu `transaction.completed`](https://developer.paddle.com/webhooks/transactions/transaction-completed).

<a name="quickstart-selling-subscriptions"></a>
### Sprzedaż subskrypcji

> [!NOTE]
> Przed skorzystaniem z Paddle Checkout powinieneś zdefiniować Produkty ze stałymi cenami w panelu Paddle. Dodatkowo powinieneś [skonfigurować obsługę webhooków Paddle](#handling-paddle-webhooks).

Oferowanie produktów i rozliczeń subskrypcyjnych za pośrednictwem aplikacji może być onieśmielające. Jednak dzięki Cashier i [Paddle's Checkout Overlay](https://developer.paddle.com/concepts/sell/overlay-checkout) możesz łatwo budować nowoczesne, solidne integracje płatności.

Aby dowiedzieć się, jak sprzedawać subskrypcje za pomocą Cashier i Paddle's Checkout Overlay, rozważmy prosty scenariusz usługi subskrypcyjnej z podstawowym planem miesięcznym (`price_basic_monthly`) i rocznym (`price_basic_yearly`). Te dwie ceny mogą być zgrupowane w ramach produktu "Basic" (`pro_basic`) w naszym panelu Paddle. Ponadto nasza usługa subskrypcyjna może oferować plan "Expert" jako `pro_expert`.

Na początek odkryjmy, jak klient może subskrybować nasze usługi. Oczywiście możesz sobie wyobrazić, że klient może kliknąć przycisk "subskrybuj" dla planu Basic na stronie cenowej naszej aplikacji. Ten przycisk wywoła Paddle Checkout Overlay dla wybranego planu. Aby rozpocząć, zainicjujmy sesję kasy za pomocą metody `checkout`:

```php
use Illuminate\Http\Request;

Route::get('/subscribe', function (Request $request) {
    $checkout = $request->user()->checkout('price_basic_monthly')
        ->returnTo(route('dashboard'));

    return view('subscribe', ['checkout' => $checkout]);
})->name('subscribe');
```

W widoku `subscribe` dodamy przycisk do wyświetlenia Checkout Overlay. Komponent Blade `paddle-button` jest dołączony do Cashier Paddle; możesz jednak również [ręcznie renderować checkout nakładkowy](#manually-rendering-an-overlay-checkout):

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

Teraz, gdy przycisk Subscribe zostanie kliknięty, klient będzie mógł wprowadzić szczegóły płatności i zainicjować subskrypcję. Aby wiedzieć, kiedy ich subskrypcja rzeczywiście się rozpoczęła (ponieważ niektóre metody płatności wymagają kilku sekund przetwarzania), powinieneś również [skonfigurować obsługę webhooków Cashier](#handling-paddle-webhooks).

Teraz, gdy klienci mogą rozpoczynać subskrypcje, musimy ograniczyć dostęp do niektórych części naszej aplikacji, aby tylko subskrybowani użytkownicy mogli do nich uzyskać dostęp. Oczywiście zawsze możemy określić aktualny status subskrypcji użytkownika za pomocą metody `subscribed` dostarczonej przez trait `Billable` Cashier:

```blade
@if ($user->subscribed())
    <p>You are subscribed.</p>
@endif
```

Możemy nawet łatwo określić, czy użytkownik jest subskrybentem konkretnego produktu lub ceny:

```blade
@if ($user->subscribedToProduct('pro_basic'))
    <p>You are subscribed to our Basic product.</p>
@endif

@if ($user->subscribedToPrice('price_basic_monthly'))
    <p>You are subscribed to our monthly Basic plan.</p>
@endif
```

<a name="quickstart-building-a-subscribed-middleware"></a>
#### Tworzenie middleware dla subskrybentów

Dla wygody możesz chcieć utworzyć [middleware](/docs/{{version}}/middleware), który określi, czy przychodzące żądanie pochodzi od subskrybowanego użytkownika. Po zdefiniowaniu tego middleware możesz łatwo przypisać go do trasy, aby uniemożliwić użytkownikom, którzy nie są subskrybentami, dostęp do trasy:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class Subscribed
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user()?->subscribed()) {
            // Przekieruj użytkownika do strony rozliczeniowej i poprosi go o subskrypcję...
            return redirect('/subscribe');
        }

        return $next($request);
    }
}
```

Po zdefiniowaniu middleware możesz przypisać go do trasy:

```php
use App\Http\Middleware\Subscribed;

Route::get('/dashboard', function () {
    // ...
})->middleware([Subscribed::class]);
```

<a name="quickstart-allowing-customers-to-manage-their-billing-plan"></a>
#### Umożliwienie klientom zarządzania planem rozliczeniowym

Oczywiście klienci mogą chcieć zmienić swój plan subskrypcji na inny produkt lub "poziom". W naszym przykładzie z góry chcielibyśmy pozwolić klientowi zmienić swój plan z miesięcznej subskrypcji na subskrypcję roczną. W tym celu będziesz musiał zaimplementować coś w rodzaju przycisku, który prowadzi do poniższej trasy:

```php
use Illuminate\Http\Request;

Route::put('/subscription/{price}/swap', function (Request $request, $price) {
    $user->subscription()->swap($price); // Z "$price" będącym "price_basic_yearly" dla tego przykładu.

    return redirect()->route('dashboard');
})->name('subscription.swap');
```

Oprócz zmiany planów będziesz musiał również pozwolić swoim klientom anulować subskrypcję. Podobnie jak przy zamianie planów, udostępnij przycisk prowadzący do następującej trasy:

```php
use Illuminate\Http\Request;

Route::put('/subscription/cancel', function (Request $request, $price) {
    $user->subscription()->cancel();

    return redirect()->route('dashboard');
})->name('subscription.cancel');
```

I teraz Twoja subskrypcja zostanie anulowana na końcu okresu rozliczeniowego.

> [!NOTE]
> Dopóki masz skonfigurowaną obsługę webhooków Cashier, Cashier automatycznie będzie utrzymywał synchronizację tabel bazodanowych związanych z Cashier w Twojej aplikacji, sprawdzając przychodzące webhooki z Paddle. Na przykład, gdy anulujesz subskrypcję klienta przez panel Paddle, Cashier otrzyma odpowiedni webhook i oznaczy subskrypcję jako "anulowaną" w bazie danych Twojej aplikacji.

<a name="checkout-sessions"></a>
## Sesje kasy

Większość operacji rozliczeniowych dla klientów jest wykonywana przy użyciu "kas" przez [widget Checkout Overlay](https://developer.paddle.com/build/checkout/build-overlay-checkout) Paddle lub wykorzystując [checkout wbudowany](https://developer.paddle.com/build/checkout/build-branded-inline-checkout).

Przed przetwarzaniem płatności kasy przy użyciu Paddle powinieneś zdefiniować [domyślny link płatności](https://developer.paddle.com/build/transactions/default-payment-link#set-default-link) swojej aplikacji w panelu ustawień kasy Paddle.

<a name="overlay-checkout"></a>
### Checkout nakładkowy

Przed wyświetleniem widgetu Checkout Overlay musisz wygenerować sesję kasy za pomocą Cashier. Sesja kasy poinformuje widget kasy o operacji rozliczeniowej, która powinna być wykonana:

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

Cashier zawiera [komponent Blade](/docs/{{version}}/blade#components) `paddle-button`. Możesz przekazać sesję kasy do tego komponentu jako "prop". Następnie, gdy ten przycisk zostanie kliknięty, zostanie wyświetlony widget kasy Paddle:

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

Domyślnie wyświetli to widget używając domyślnego stylu Paddle. Możesz dostosować widget, dodając [atrybuty wspierane przez Paddle](https://developer.paddle.com/paddlejs/html-data-attributes), takie jak atrybut `data-theme='light'` do komponentu:

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4" data-theme="light">
    Subscribe
</x-paddle-button>
```

Widget kasy Paddle jest asynchroniczny. Po utworzeniu subskrypcji przez użytkownika w widgecie, Paddle wyśle webhook do Twojej aplikacji, abyś mógł prawidłowo zaktualizować stan subskrypcji w bazie danych aplikacji. Dlatego ważne jest, abyś odpowiednio [skonfigurował webhooki](#handling-paddle-webhooks), aby uwzględnić zmiany stanu z Paddle.

> [!WARNING]
> Po zmianie stanu subskrypcji opóźnienie w otrzymaniu odpowiedniego webhooka jest zazwyczaj minimalne, ale powinieneś uwzględnić to w swojej aplikacji, biorąc pod uwagę, że subskrypcja użytkownika może nie być natychmiast dostępna po zakończeniu kasy.

<a name="manually-rendering-an-overlay-checkout"></a>
#### Ręczne renderowanie checkout nakładkowego

Możesz również ręcznie renderować checkout nakładkowy bez używania wbudowanych komponentów Blade Laravel. Aby rozpocząć, wygeneruj sesję kasy [jak pokazano w poprzednich przykładach](#overlay-checkout):

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

Następnie możesz użyć Paddle.js do zainicjowania kasy. W tym przykładzie utworzymy link, któremu przypisana jest klasa `paddle_button`. Paddle.js wykryje tę klasę i wyświetli kasę nakładkową po kliknięciu linku:

```blade
<?php
$items = $checkout->getItems();
$customer = $checkout->getCustomer();
$custom = $checkout->getCustomData();
?>

<a
    href='#!'
    class='paddle_button'
    data-items='{!! json_encode($items) !!}'
    @if ($customer) data-customer-id='{{ $customer->paddle_id }}' @endif
    @if ($custom) data-custom-data='{{ json_encode($custom) }}' @endif
    @if ($returnUrl = $checkout->getReturnUrl()) data-success-url='{{ $returnUrl }}' @endif
>
    Buy Product
</a>
```

<a name="inline-checkout"></a>
### Checkout wbudowany

Jeśli nie chcesz korzystać z widgetu kasy w stylu "nakładki" Paddle, Paddle zapewnia również opcję wyświetlania widgetu wbudowanego. Chociaż to podejście nie pozwala na dostosowanie żadnych pól HTML kasy, umożliwia osadzenie widgetu w aplikacji.

Aby ułatwić Ci rozpoczęcie pracy z checkout wbudowanym, Cashier zawiera komponent Blade `paddle-checkout`. Aby rozpocząć, powinieneś [wygenerować sesję kasy](#overlay-checkout):

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

Następnie możesz przekazać sesję kasy do atrybutu `checkout` komponentu:

```blade
<x-paddle-checkout :checkout="$checkout" class="w-full" />
```

Aby dostosować wysokość komponentu inline checkout, możesz przekazać atrybut `height` do komponentu Blade:

```blade
<x-paddle-checkout :checkout="$checkout" class="w-full" height="500" />
```

Zapoznaj się z [przewodnikiem Paddle po Inline Checkout](https://developer.paddle.com/build/checkout/build-branded-inline-checkout) i [dostępnymi ustawieniami kasy](https://developer.paddle.com/build/checkout/set-up-checkout-default-settings), aby uzyskać więcej szczegółów na temat opcji dostosowywania inline checkout.

<a name="manually-rendering-an-inline-checkout"></a>
#### Ręczne renderowanie checkout wbudowanego

Możesz również ręcznie renderować checkout wbudowany bez używania wbudowanych komponentów Blade Laravel. Aby rozpocząć, wygeneruj sesję kasy [jak pokazano w poprzednich przykładach](#inline-checkout):

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

Następnie możesz użyć Paddle.js do zainicjowania kasy. W tym przykładzie zademonstrujemy to używając [Alpine.js](https://github.com/alpinejs/alpine); możesz jednak swobodnie zmodyfikować ten przykład dla własnego stosu frontendowego:

```blade
<?php
$options = $checkout->options();

$options['settings']['frameTarget'] = 'paddle-checkout';
$options['settings']['frameInitialHeight'] = 366;
?>

<div class="paddle-checkout" x-data="{}" x-init="
    Paddle.Checkout.open(@json($options));
">
</div>
```

<a name="guest-checkouts"></a>
### Checkout dla gości

Czasami możesz potrzebować utworzyć sesję kasy dla użytkowników, którzy nie potrzebują konta w Twojej aplikacji. Aby to zrobić, możesz użyć metody `guest`:

```php
use Illuminate\Http\Request;
use Laravel\Paddle\Checkout;

Route::get('/buy', function (Request $request) {
    $checkout = Checkout::guest(['pri_34567'])
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

Następnie możesz przekazać sesję kasy do komponentów Blade [przycisku Paddle](#overlay-checkout) lub [checkout wbudowanego](#inline-checkout).

<a name="price-previews"></a>
## Podgląd cen

Paddle pozwala dostosowywać ceny dla każdej waluty, zasadniczo umożliwiając konfigurację różnych cen dla różnych krajów. Cashier Paddle pozwala pobrać wszystkie te ceny za pomocą metody `previewPrices`. Ta metoda akceptuje identyfikatory cen, dla których chcesz pobrać ceny:

```php
use Laravel\Paddle\Cashier;

$prices = Cashier::previewPrices(['pri_123', 'pri_456']);
```

Waluta zostanie określona na podstawie adresu IP żądania; możesz jednak opcjonalnie podać konkretny kraj, dla którego chcesz pobrać ceny:

```php
use Laravel\Paddle\Cashier;

$prices = Cashier::previewPrices(['pri_123', 'pri_456'], ['address' => [
    'country_code' => 'BE',
    'postal_code' => '1234',
]]);
```

Po pobraniu cen możesz wyświetlić je w dowolny sposób:

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->total() }}</li>
    @endforeach
</ul>
```

Możesz również wyświetlić cenę netto i kwotę podatku osobno:

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->subtotal() }} (+ {{ $price->tax() }} tax)</li>
    @endforeach
</ul>
```

Aby uzyskać więcej informacji, [sprawdź dokumentację API Paddle dotyczącą podglądu cen](https://developer.paddle.com/api-reference/pricing-preview/preview-prices).

<a name="customer-price-previews"></a>
### Podgląd cen dla klientów

Jeśli użytkownik jest już klientem i chcesz wyświetlić ceny dotyczące tego klienta, możesz to zrobić, pobierając ceny bezpośrednio z instancji klienta:

```php
use App\Models\User;

$prices = User::find(1)->previewPrices(['pri_123', 'pri_456']);
```

Wewnętrznie Cashier użyje identyfikatora klienta użytkownika do pobrania cen w jego walucie. Na przykład użytkownik mieszkający w Stanach Zjednoczonych zobaczy ceny w dolarach amerykańskich, podczas gdy użytkownik w Belgii zobaczy ceny w euro. Jeśli nie zostanie znaleziona pasująca waluta, zostanie użyta domyślna waluta produktu. Możesz dostosować wszystkie ceny produktu lub planu subskrypcji w panelu kontrolnym Paddle.

<a name="price-discounts"></a>
### Rabaty

Możesz również wyświetlić ceny po rabacie. Podczas wywoływania metody `previewPrices` podaj identyfikator rabatu za pomocą opcji `discount_id`:

```php
use Laravel\Paddle\Cashier;

$prices = Cashier::previewPrices(['pri_123', 'pri_456'], [
    'discount_id' => 'dsc_123'
]);
```

Następnie wyświetl obliczone ceny:

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->total() }}</li>
    @endforeach
</ul>
```

<a name="customers"></a>
## Klienci

<a name="customer-defaults"></a>
### Domyślne dane klientów

Cashier pozwala zdefiniować przydatne wartości domyślne dla klientów podczas tworzenia sesji kasy. Ustawienie tych wartości domyślnych pozwala na wstępne wypełnienie adresu e-mail i nazwy klienta, aby mógł on natychmiast przejść do części płatności widgetu kasy. Możesz ustawić te wartości domyślne, nadpisując następujące metody w modelu rozliczeniowym:

```php
/**
 * Pobierz nazwę klienta do powiązania z Paddle.
 */
public function paddleName(): string|null
{
    return $this->name;
}

/**
 * Pobierz adres e-mail klienta do powiązania z Paddle.
 */
public function paddleEmail(): string|null
{
    return $this->email;
}
```

Te wartości domyślne będą używane dla każdej akcji w Cashier, która generuje [sesję kasy](#checkout-sessions).

<a name="retrieving-customers"></a>
### Pobieranie klientów

Możesz pobrać klienta po jego identyfikatorze klienta Paddle za pomocą metody `Cashier::findBillable`. Ta metoda zwróci instancję modelu rozliczeniowego:

```php
use Laravel\Paddle\Cashier;

$user = Cashier::findBillable($customerId);
```

<a name="creating-customers"></a>
### Tworzenie klientów

Okazjonalnie możesz chcieć utworzyć klienta Paddle bez rozpoczynania subskrypcji. Możesz to osiągnąć używając metody `createAsCustomer`:

```php
$customer = $user->createAsCustomer();
```

Zwracana jest instancja `Laravel\Paddle\Customer`. Po utworzeniu klienta w Paddle możesz rozpocząć subskrypcję w późniejszym terminie. Możesz podać opcjonalną tablicę `$options`, aby przekazać dodatkowe [parametry tworzenia klienta obsługiwane przez API Paddle](https://developer.paddle.com/api-reference/customers/create-customer):

```php
$customer = $user->createAsCustomer($options);
```

<a name="subscriptions"></a>
## Subskrypcje

<a name="creating-subscriptions"></a>
### Tworzenie subskrypcji

Aby utworzyć subskrypcję, najpierw pobierz instancję modelu rozliczeniowego z bazy danych, która zazwyczaj będzie instancją `App\Models\User`. Po pobraniu instancji modelu możesz użyć metody `subscribe` do utworzenia sesji kasy modelu:

```php
use Illuminate\Http\Request;

Route::get('/user/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe($premium = 'pri_123', 'default')
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

Pierwszy argument przekazany do metody `subscribe` to konkretna cena, na którą subskrybuje użytkownik. Ta wartość powinna odpowiadać identyfikatorowi ceny w Paddle. Metoda `returnTo` akceptuje adres URL, na który użytkownik zostanie przekierowany po pomyślnym zakończeniu kasy. Drugi argument przekazany do metody `subscribe` powinien być wewnętrznym "typem" subskrypcji. Jeśli Twoja aplikacja oferuje tylko jedną subskrypcję, możesz nazwać ją `default` lub `primary`. Ten typ subskrypcji jest przeznaczony tylko do wewnętrznego użytku aplikacji i nie powinien być wyświetlany użytkownikom. Ponadto nie powinien zawierać spacji i nigdy nie powinien być zmieniany po utworzeniu subskrypcji.

Możesz również podać tablicę niestandardowych metadanych dotyczących subskrypcji za pomocą metody `customData`:

```php
$checkout = $request->user()->subscribe($premium = 'pri_123', 'default')
    ->customData(['key' => 'value'])
    ->returnTo(route('home'));
```

Po utworzeniu sesji kasy subskrypcji, sesja kasy może zostać przekazana do [komponentu Blade](#overlay-checkout) `paddle-button`, który jest dołączony do Cashier Paddle:

```blade
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

Po zakończeniu kasy przez użytkownika, webhook `subscription_created` zostanie wysłany z Paddle. Cashier otrzyma ten webhook i skonfiguruje subskrypcję dla Twojego klienta. Aby upewnić się, że wszystkie webhooki są prawidłowo odbierane i obsługiwane przez Twoją aplikację, upewnij się, że prawidłowo [skonfigurowałeś obsługę webhooków](#handling-paddle-webhooks).

<a name="checking-subscription-status"></a>
### Sprawdzanie statusu subskrypcji

Po subskrybowaniu aplikacji przez użytkownika możesz sprawdzić status jego subskrypcji za pomocą różnych wygodnych metod. Po pierwsze, metoda `subscribed` zwraca `true`, jeśli użytkownik ma ważną subskrypcję, nawet jeśli subskrypcja jest obecnie w okresie próbnym:

```php
if ($user->subscribed()) {
    // ...
}
```

Jeśli Twoja aplikacja oferuje wiele subskrypcji, możesz określić subskrypcję podczas wywoływania metody `subscribed`:

```php
if ($user->subscribed('default')) {
    // ...
}
```

Metoda `subscribed` jest również doskonałym kandydatem na [middleware trasy](/docs/{{version}}/middleware), umożliwiając filtrowanie dostępu do tras i kontrolerów na podstawie statusu subskrypcji użytkownika:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserIsSubscribed
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->user() && ! $request->user()->subscribed()) {
            // Ten użytkownik nie jest płacącym klientem...
            return redirect('/billing');
        }

        return $next($request);
    }
}
```

Jeśli chcesz sprawdzić, czy użytkownik nadal jest w okresie próbnym, możesz użyć metody `onTrial`. Ta metoda może być przydatna do określenia, czy powinieneś wyświetlić ostrzeżenie użytkownikowi, że nadal jest w okresie próbnym:

```php
if ($user->subscription()->onTrial()) {
    // ...
}
```

Metoda `subscribedToPrice` może być użyta do określenia, czy użytkownik jest subskrybentem danego planu na podstawie danego identyfikatora ceny Paddle. W tym przykładzie określimy, czy subskrypcja `default` użytkownika jest aktywnie subskrybowana na cenę miesięczną:

```php
if ($user->subscribedToPrice($monthly = 'pri_123', 'default')) {
    // ...
}
```

Metoda `recurring` może być użyta do określenia, czy użytkownik jest obecnie w aktywnej subskrypcji i nie jest już w okresie próbnym ani w okresie karencji:

```php
if ($user->subscription()->recurring()) {
    // ...
}
```

<a name="canceled-subscription-status"></a>
#### Status Anulowanej Subskrypcji

Aby określić, czy użytkownik był kiedyś aktywnym subskrybentem, ale anulował swoją subskrypcję, możesz użyć metody `canceled`:

```php
if ($user->subscription()->canceled()) {
    // ...
}
```

Możesz również określić, czy użytkownik anulował swoją subskrypcję, ale nadal jest w "okresie karencji" do momentu pełnego wygaśnięcia subskrypcji. Na przykład, jeśli użytkownik anuluje subskrypcję 5 marca, która pierwotnie miała wygasnąć 10 marca, użytkownik jest w "okresie karencji" do 10 marca. Ponadto metoda `subscribed` nadal będzie zwracać `true` w tym czasie:

```php
if ($user->subscription()->onGracePeriod()) {
    // ...
}
```

<a name="past-due-status"></a>
#### Status Przeterminowany

Jeśli płatność za subskrypcję nie powiedzie się, zostanie ona oznaczona jako `past_due`. Gdy subskrypcja jest w tym stanie, nie będzie aktywna, dopóki klient nie zaktualizuje swoich informacji płatniczych. Możesz określić, czy subskrypcja jest przeterminowana, używając metody `pastDue` na instancji subskrypcji:

```php
if ($user->subscription()->pastDue()) {
    // ...
}
```

Gdy subskrypcja jest przeterminowana, powinieneś polecić użytkownikowi [aktualizację informacji o płatności](#updating-payment-information).

Jeśli chcesz, aby subskrypcje były nadal uważane za ważne, gdy są `past_due`, możesz użyć metody `keepPastDueSubscriptionsActive` dostarczonej przez Cashier. Zazwyczaj ta metoda powinna być wywołana w metodzie `register` Twojego `AppServiceProvider`:

```php
use Laravel\Paddle\Cashier;

/**
 * Register any application services.
 */
public function register(): void
{
    Cashier::keepPastDueSubscriptionsActive();
}
```

> [!WARNING]
> Gdy subskrypcja jest w stanie `past_due`, nie może być zmieniona, dopóki informacje o płatności nie zostaną zaktualizowane. Dlatego metody `swap` i `updateQuantity` rzucą wyjątek, gdy subskrypcja jest w stanie `past_due`.

<a name="subscription-scopes"></a>
#### Zakresy Subskrypcji

Większość stanów subskrypcji jest również dostępna jako zakresy zapytań, więc możesz łatwo odpytać swoją bazę danych o subskrypcje, które są w danym stanie:

```php
// Pobierz wszystkie ważne subskrypcje...
$subscriptions = Subscription::query()->valid()->get();

// Pobierz wszystkie anulowane subskrypcje dla użytkownika...
$subscriptions = $user->subscriptions()->canceled()->get();
```

Pełna lista dostępnych zakresów jest dostępna poniżej:

```php
Subscription::query()->valid();
Subscription::query()->onTrial();
Subscription::query()->expiredTrial();
Subscription::query()->notOnTrial();
Subscription::query()->active();
Subscription::query()->recurring();
Subscription::query()->pastDue();
Subscription::query()->paused();
Subscription::query()->notPaused();
Subscription::query()->onPausedGracePeriod();
Subscription::query()->notOnPausedGracePeriod();
Subscription::query()->canceled();
Subscription::query()->notCanceled();
Subscription::query()->onGracePeriod();
Subscription::query()->notOnGracePeriod();
```

<a name="subscription-single-charges"></a>
### Jednorazowe Opłaty Subskrypcyjne

Jednorazowe opłaty subskrypcyjne pozwalają na obciążenie subskrybentów jednorazową opłatą ponad ich subskrypcje. Musisz podać jeden lub więcej identyfikatorów cen podczas wywoływania metody `charge`:

```php
// Charge a single price...
$response = $user->subscription()->charge('pri_123');

// Charge multiple prices at once...
$response = $user->subscription()->charge(['pri_123', 'pri_456']);
```

Metoda `charge` faktycznie nie obciąży klienta, dopóki nie nastąpi następny okres rozliczeniowy ich subskrypcji. Jeśli chcesz obciążyć klienta natychmiast, możesz zamiast tego użyć metody `chargeAndInvoice`:

```php
$response = $user->subscription()->chargeAndInvoice('pri_123');
```

<a name="updating-payment-information"></a>
### Aktualizacja Informacji o Płatności

Paddle zawsze zapisuje metodę płatności na subskrypcję. Jeśli chcesz zaktualizować domyślną metodę płatności dla subskrypcji, powinieneś przekierować klienta na hostowany przez Paddle stronę aktualizacji metody płatności, używając metody `redirectToUpdatePaymentMethod` na modelu subskrypcji:

```php
use Illuminate\Http\Request;

Route::get('/update-payment-method', function (Request $request) {
    $user = $request->user();

    return $user->subscription()->redirectToUpdatePaymentMethod();
});
```

Gdy użytkownik zakończy aktualizację swoich informacji, webhook `subscription_updated` zostanie wysłany przez Paddle, a szczegóły subskrypcji zostaną zaktualizowane w bazie danych aplikacji.

<a name="changing-plans"></a>
### Zmiana Planów

Po zasubskrybowaniu aplikacji przez użytkownika, może on czasami chcieć zmienić na nowy plan subskrypcji. Aby zaktualizować plan subskrypcji dla użytkownika, powinieneś przekazać identyfikator ceny Paddle do metody `swap` subskrypcji:

```php
use App\Models\User;

$user = User::find(1);

$user->subscription()->swap($premium = 'pri_456');
```

Jeśli chcesz zamienić plany i natychmiast wystawić fakturę użytkownikowi zamiast czekać na następny cykl rozliczeniowy, możesz użyć metody `swapAndInvoice`:

```php
$user = User::find(1);

$user->subscription()->swapAndInvoice($premium = 'pri_456');
```

<a name="prorations"></a>
#### Proporcje

Domyślnie Paddle proporcjonalnie rozlicza opłaty podczas zamiany między planami. Metoda `noProrate` może być użyta do aktualizacji subskrypcji bez proporcjonalnego rozliczania opłat:

```php
$user->subscription('default')->noProrate()->swap($premium = 'pri_456');
```

Jeśli chcesz wyłączyć proporcjonalne rozliczanie i natychmiast wystawić fakturę klientom, możesz użyć metody `swapAndInvoice` w połączeniu z `noProrate`:

```php
$user->subscription('default')->noProrate()->swapAndInvoice($premium = 'pri_456');
```

Lub, aby nie obciążać klienta za zmianę subskrypcji, możesz użyć metody `doNotBill`:

```php
$user->subscription('default')->doNotBill()->swap($premium = 'pri_456');
```

Aby uzyskać więcej informacji na temat zasad proporcjonalnego rozliczania Paddle, zapoznaj się z [dokumentacją proporcjonalnego rozliczania](https://developer.paddle.com/concepts/subscriptions/proration) Paddle.

<a name="subscription-quantity"></a>
### Ilość Subskrypcji

Czasami subskrypcje są uzależnione od "ilości". Na przykład aplikacja do zarządzania projektami może pobierać 10 USD miesięcznie za projekt. Aby łatwo zwiększyć lub zmniejszyć ilość subskrypcji, użyj metod `incrementQuantity` i `decrementQuantity`:

```php
$user = User::find(1);

$user->subscription()->incrementQuantity();

// Add five to the subscription's current quantity...
$user->subscription()->incrementQuantity(5);

$user->subscription()->decrementQuantity();

// Subtract five from the subscription's current quantity...
$user->subscription()->decrementQuantity(5);
```

Alternatywnie, możesz ustawić konkretną ilość używając metody `updateQuantity`:

```php
$user->subscription()->updateQuantity(10);
```

Metoda `noProrate` może być użyta do aktualizacji ilości subskrypcji bez proporcjonalnego rozliczania opłat:

```php
$user->subscription()->noProrate()->updateQuantity(10);
```

<a name="quantities-for-subscription-with-multiple-products"></a>
#### Ilości dla Subskrypcji z Wieloma Produktami

Jeśli Twoja subskrypcja jest [subskrypcją z wieloma produktami](#subscriptions-with-multiple-products), powinieneś przekazać identyfikator ceny, której ilość chcesz zwiększyć lub zmniejszyć jako drugi argument do metod inkrementacji/dekrementacji:

```php
$user->subscription()->incrementQuantity(1, 'price_chat');
```

<a name="subscriptions-with-multiple-products"></a>
### Subskrypcje z Wieloma Produktami

[Subskrypcje z wieloma produktami](https://developer.paddle.com/build/subscriptions/add-remove-products-prices-addons) pozwalają przypisać wiele produktów rozliczeniowych do pojedynczej subskrypcji. Na przykład wyobraź sobie, że budujesz aplikację "helpdesk" obsługi klienta, która ma bazowaną cenę subskrypcji 10 USD miesięcznie, ale oferuje dodatek czatu na żywo za dodatkowe 15 USD miesięcznie.

Podczas tworzenia sesji kasy subskrypcji, możesz określić wiele produktów dla danej subskrypcji, przekazując tablicę cen jako pierwszy argument do metody `subscribe`:

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe([
        'price_monthly',
        'price_chat',
    ]);

    return view('billing', ['checkout' => $checkout]);
});
```

W powyższym przykładzie klient będzie miał dwie ceny przypisane do swojej subskrypcji `default`. Obie ceny będą pobierane w odpowiednich odstępach rozliczeniowych. W razie potrzeby możesz przekazać tablicę asocjacyjną par klucz/wartość, aby wskazać konkretną ilość dla każdej ceny:

```php
$user = User::find(1);

$checkout = $user->subscribe('default', ['price_monthly', 'price_chat' => 5]);
```

Jeśli chcesz dodać kolejną cenę do istniejącej subskrypcji, musisz użyć metody `swap` subskrypcji. Podczas wywoływania metody `swap`, powinieneś również uwzględnić bieżące ceny i ilości subskrypcji:

```php
$user = User::find(1);

$user->subscription()->swap(['price_chat', 'price_original' => 2]);
```

Powyższy przykład doda nową cenę, ale klient nie zostanie za nią obciążony aż do następnego cyklu rozliczeniowego. Jeśli chcesz obciążyć klienta natychmiast, możesz użyć metody `swapAndInvoice`:

```php
$user->subscription()->swapAndInvoice(['price_chat', 'price_original' => 2]);
```

Możesz usunąć ceny z subskrypcji używając metody `swap` i pomijając cenę, którą chcesz usunąć:

```php
$user->subscription()->swap(['price_original' => 2]);
```

> [!WARNING]
> Nie możesz usunąć ostatniej ceny z subskrypcji. Zamiast tego powinieneś po prostu anulować subskrypcję.

<a name="multiple-subscriptions"></a>
### Wiele Subskrypcji

Paddle pozwala Twoim klientom mieć wiele subskrypcji jednocześnie. Na przykład możesz prowadzić siłownię, która oferuje subskrypcję pływania i subskrypcję podnoszenia ciężarów, a każda subskrypcja może mieć różne ceny. Oczywiście klienci powinni móc subskrybować jeden lub oba plany.

Gdy Twoja aplikacja tworzy subskrypcje, możesz podać typ subskrypcji do metody `subscribe` jako drugi argument. Typ może być dowolnym ciągiem znaków reprezentującym typ subskrypcji, którą użytkownik rozpoczyna:

```php
use Illuminate\Http\Request;

Route::post('/swimming/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe($swimmingMonthly = 'pri_123', 'swimming');

    return view('billing', ['checkout' => $checkout]);
});
```

W tym przykładzie zainicjowaliśmy miesięczną subskrypcję pływania dla klienta. Jednak może on później chcieć zmienić na subskrypcję roczną. Podczas dostosowywania subskrypcji klienta możemy po prostu zamienić cenę na subskrypcji `swimming`:

```php
$user->subscription('swimming')->swap($swimmingYearly = 'pri_456');
```

Oczywiście możesz również całkowicie anulować subskrypcję:

```php
$user->subscription('swimming')->cancel();
```

<a name="pausing-subscriptions"></a>
### Wstrzymywanie Subskrypcji

Aby wstrzymać subskrypcję, wywołaj metodę `pause` na subskrypcji użytkownika:

```php
$user->subscription()->pause();
```

Gdy subskrypcja jest wstrzymana, Cashier automatycznie ustawi kolumnę `paused_at` w bazie danych. Ta kolumna jest używana do określenia, kiedy metoda `paused` powinna zacząć zwracać `true`. Na przykład, jeśli klient wstrzyma subskrypcję 1 marca, ale subskrypcja nie była zaplanowana do odnowienia do 5 marca, metoda `paused` będzie nadal zwracać `false` do 5 marca. Dzieje się tak, ponieważ użytkownik może zazwyczaj nadal korzystać z aplikacji do końca okresu rozliczeniowego.

Domyślnie wstrzymanie następuje w następnym okresie rozliczeniowym, aby klient mógł korzystać z pozostałej części okresu, za który zapłacił. Jeśli chcesz natychmiast wstrzymać subskrypcję, możesz użyć metody `pauseNow`:

```php
$user->subscription()->pauseNow();
```

Używając metody `pauseUntil`, możesz wstrzymać subskrypcję do określonego momentu w czasie:

```php
$user->subscription()->pauseUntil(now()->plus(months: 1));
```

Lub możesz użyć metody `pauseNowUntil`, aby natychmiast wstrzymać subskrypcję do określonego momentu w czasie:

```php
$user->subscription()->pauseNowUntil(now()->plus(months: 1));
```

Możesz określić, czy użytkownik wstrzymał swoją subskrypcję, ale nadal jest w "okresie karencji", używając metody `onPausedGracePeriod`:

```php
if ($user->subscription()->onPausedGracePeriod()) {
    // ...
}
```

Aby wznowić wstrzymaną subskrypcję, możesz wywołać metodę `resume` na subskrypcji:

```php
$user->subscription()->resume();
```

> [!WARNING]
> Subskrypcja nie może być modyfikowana, gdy jest wstrzymana. Jeśli chcesz zmienić na inny plan lub zaktualizować ilości, musisz najpierw wznowić subskrypcję.

<a name="canceling-subscriptions"></a>
### Anulowanie Subskrypcji

Aby anulować subskrypcję, wywołaj metodę `cancel` na subskrypcji użytkownika:

```php
$user->subscription()->cancel();
```

Gdy subskrypcja jest anulowana, Cashier automatycznie ustawi kolumnę `ends_at` w bazie danych. Ta kolumna jest używana do określenia, kiedy metoda `subscribed` powinna zacząć zwracać `false`. Na przykład, jeśli klient anuluje subskrypcję 1 marca, ale subskrypcja nie była zaplanowana do zakończenia do 5 marca, metoda `subscribed` będzie nadal zwracać `true` do 5 marca. Dzieje się tak, ponieważ użytkownik może zazwyczaj nadal korzystać z aplikacji do końca okresu rozliczeniowego.

Możesz określić, czy użytkownik anulował swoją subskrypcję, ale nadal jest w "okresie karencji", używając metody `onGracePeriod`:

```php
if ($user->subscription()->onGracePeriod()) {
    // ...
}
```

Jeśli chcesz natychmiast anulować subskrypcję, możesz wywołać metodę `cancelNow` na subskrypcji:

```php
$user->subscription()->cancelNow();
```

Aby zatrzymać anulowanie subskrypcji w okresie karencji, możesz wywołać metodę `stopCancelation`:

```php
$user->subscription()->stopCancelation();
```

> [!WARNING]
> Subskrypcje Paddle nie mogą być wznowione po anulowaniu. Jeśli Twój klient chce wznowić swoją subskrypcję, będzie musiał utworzyć nową subskrypcję.

<a name="subscription-trials"></a>
## Okresy Próbne Subskrypcji

<a name="with-payment-method-up-front"></a>
### Z Metodą Płatności z Góry

Jeśli chcesz zaoferować okresy próbne swoim klientom, jednocześnie zbierając informacje o metodzie płatności z góry, powinieneś ustawić czas próbny w pulpicie Paddle na cenie, którą subskrybuje klient. Następnie zainicjuj sesję kasy jak zwykle:

```php
use Illuminate\Http\Request;

Route::get('/user/subscribe', function (Request $request) {
    $checkout = $request->user()
        ->subscribe('pri_monthly')
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

Gdy aplikacja otrzyma zdarzenie `subscription_created`, Cashier ustawi datę zakończenia okresu próbnego w rekordzie subskrypcji w bazie danych aplikacji, a także poinstruuje Paddle, aby nie rozpoczynać obciążania klienta przed tą datą.

> [!WARNING]
> Jeśli subskrypcja klienta nie zostanie anulowana przed datą zakończenia okresu próbnego, zostanie on obciążony, gdy tylko okres próbny wygaśnie, więc powinieneś upewnić się, że poinformujesz użytkowników o dacie zakończenia okresu próbnego.

Możesz określić, czy użytkownik jest w okresie próbnym, używając metody `onTrial` na instancji użytkownika:

```php
if ($user->onTrial()) {
    // ...
}
```

Aby określić, czy istniejący okres próbny wygasł, możesz użyć metody `hasExpiredTrial`:

```php
if ($user->hasExpiredTrial()) {
    // ...
}
```

Aby określić, czy użytkownik jest w okresie próbnym dla konkretnego typu subskrypcji, możesz podać typ do metod `onTrial` lub `hasExpiredTrial`:

```php
if ($user->onTrial('default')) {
    // ...
}

if ($user->hasExpiredTrial('default')) {
    // ...
}
```

<a name="without-payment-method-up-front"></a>
### Bez Metody Płatności z Góry

Jeśli chcesz zaoferować okresy próbne bez zbierania informacji o metodzie płatności użytkownika z góry, możesz ustawić kolumnę `trial_ends_at` w rekordzie klienta związanym z użytkownikiem na żądaną datę zakończenia próby. Zazwyczaj robi się to podczas rejestracji użytkownika:

```php
use App\Models\User;

$user = User::create([
    // ...
]);

$user->createAsCustomer([
    'trial_ends_at' => now()->plus(days: 10)
]);
```

Cashier odnosi się do tego typu próby jako "ogólnej próby", ponieważ nie jest ona związana z żadną istniejącą subskrypcją. Metoda `onTrial` w instancji `User` zwróci `true`, jeśli bieżąca data nie przekroczyła wartości `trial_ends_at`:

```php
if ($user->onTrial()) {
    // User is within their trial period...
}
```

Gdy będziesz gotowy do utworzenia faktycznej subskrypcji dla użytkownika, możesz użyć metody `subscribe` jak zwykle:

```php
use Illuminate\Http\Request;

Route::get('/user/subscribe', function (Request $request) {
    $checkout = $request->user()
        ->subscribe('pri_monthly')
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

Aby pobrać datę zakończenia próby użytkownika, możesz użyć metody `trialEndsAt`. Ta metoda zwróci instancję daty Carbon, jeśli użytkownik jest w okresie próbnym, lub `null`, jeśli nie jest. Możesz również przekazać opcjonalny parametr typu subskrypcji, jeśli chcesz uzyskać datę zakończenia próby dla określonej subskrypcji innej niż domyślna:

```php
if ($user->onTrial('default')) {
    $trialEndsAt = $user->trialEndsAt();
}
```

Możesz użyć metody `onGenericTrial`, jeśli chcesz wiedzieć konkretnie, że użytkownik jest w swoim "ogólnym" okresie próbnym i nie utworzył jeszcze faktycznej subskrypcji:

```php
if ($user->onGenericTrial()) {
    // User is within their "generic" trial period...
}
```

<a name="extend-or-activate-a-trial"></a>
### Przedłużanie lub Aktywacja Okresu Próbnego

Możesz przedłużyć istniejący okres próbny subskrypcji, wywołując metodę `extendTrial` i określając moment, w którym okres próbny powinien się zakończyć:

```php
$user->subscription()->extendTrial(now()->plus(days: 5));
```

Lub możesz natychmiast aktywować subskrypcję, kończąc jej okres próbny, wywołując metodę `activate` na subskrypcji:

```php
$user->subscription()->activate();
```

<a name="handling-paddle-webhooks"></a>
## Obsługa Webhooków Paddle

Paddle może powiadamiać Twoją aplikację o różnych zdarzeniach za pośrednictwem webhooków. Domyślnie trasa wskazująca na kontroler webhooków Cashier jest rejestrowana przez dostawcę usług Cashier. Ten kontroler będzie obsługiwał wszystkie przychodzące żądania webhooków.

Domyślnie ten kontroler automatycznie obsłuży anulowanie subskrypcji, które mają zbyt wiele nieudanych opłat, aktualizacje subskrypcji i zmiany metod płatności; jednak, jak wkrótce odkryjemy, możesz rozszerzyć ten kontroler, aby obsługiwać dowolne zdarzenie webhooka Paddle, które lubisz.

Aby upewnić się, że Twoja aplikacja może obsługiwać webhooki Paddle, upewnij się, że [skonfigurowałeś adres URL webhooka w panelu kontrolnym Paddle](https://vendors.paddle.com/notifications-v2). Domyślnie kontroler webhooków Cashier odpowiada na ścieżkę URL `/paddle/webhook`. Pełna lista wszystkich webhooków, które powinieneś włączyć w panelu kontrolnym Paddle, to:

- Customer Updated
- Transaction Completed
- Transaction Updated
- Subscription Created
- Subscription Updated
- Subscription Paused
- Subscription Canceled

> [!WARNING]
> Upewnij się, że chronisz przychodzące żądania za pomocą zawartego w Cashier middleware [weryfikacji podpisu webhook](/docs/{{version}}/cashier-paddle#verifying-webhook-signatures).

<a name="webhooks-csrf-protection"></a>
#### Webhooki i Ochrona CSRF

Ponieważ webhooki Paddle muszą ominąć [ochronę CSRF](/docs/{{version}}/csrf) Laravel, powinieneś upewnić się, że Laravel nie próbuje weryfikować tokena CSRF dla przychodzących webhooków Paddle. Aby to osiągnąć, powinieneś wykluczyć `paddle/*` z ochrony CSRF w pliku `bootstrap/app.php` swojej aplikacji:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->validateCsrfTokens(except: [
        'paddle/*',
    ]);
})
```

<a name="webhooks-local-development"></a>
#### Webhooki i Lokalny Rozwój

Aby Paddle mógł wysyłać webhooki do Twojej aplikacji podczas lokalnego rozwoju, będziesz musiał ujawnić swoją aplikację za pośrednictwem usługi udostępniania witryn, takiej jak [Ngrok](https://ngrok.com/) lub [Expose](https://expose.dev/docs/introduction). Jeśli rozwijasz swoją aplikację lokalnie za pomocą [Laravel Sail](/docs/{{version}}/sail), możesz użyć [komendy udostępniania witryny](/docs/{{version}}/sail#sharing-your-site) Sail.

<a name="defining-webhook-event-handlers"></a>
### Definiowanie Obsługi Zdarzeń Webhook

Cashier automatycznie obsługuje anulowanie subskrypcji przy nieudanych opłatach i innych typowych webhookach Paddle. Jednak jeśli masz dodatkowe zdarzenia webhooków, które chciałbyś obsłużyć, możesz to zrobić, nasłuchując następujących zdarzeń, które są wysyłane przez Cashier:

- `Laravel\Paddle\Events\WebhookReceived`
- `Laravel\Paddle\Events\WebhookHandled`

Oba zdarzenia zawierają pełny payload webhooka Paddle. Na przykład, jeśli chcesz obsłużyć webhook `transaction.billed`, możesz zarejestrować [nasłuchiwacz](/docs/{{version}}/events#defining-listeners), który obsłuży zdarzenie:

```php
<?php

namespace App\Listeners;

use Laravel\Paddle\Events\WebhookReceived;

class PaddleEventListener
{
    /**
     * Handle received Paddle webhooks.
     */
    public function handle(WebhookReceived $event): void
    {
        if ($event->payload['event_type'] === 'transaction.billed') {
            // Handle the incoming event...
        }
    }
}
```

Cashier emituje również zdarzenia dedykowane typowi otrzymanego webhooka. Oprócz pełnego payloadu z Paddle, zawierają one również odpowiednie modele, które zostały użyte do przetworzenia webhooka, takie jak model rozliczeniowy, subskrypcja lub paragon:

<div class="content-list" markdown="1">

- `Laravel\Paddle\Events\CustomerUpdated`
- `Laravel\Paddle\Events\TransactionCompleted`
- `Laravel\Paddle\Events\TransactionUpdated`
- `Laravel\Paddle\Events\SubscriptionCreated`
- `Laravel\Paddle\Events\SubscriptionUpdated`
- `Laravel\Paddle\Events\SubscriptionPaused`
- `Laravel\Paddle\Events\SubscriptionCanceled`

</div>

Możesz również nadrisać domyślną, wbudowaną trasę webhooka, definiując zmienną środowiskową `CASHIER_WEBHOOK` w pliku `.env` swojej aplikacji. Ta wartość powinna być pełnym adresem URL Twojej trasy webhooka i musi odpowiadać adresowi URL ustawionemu w panelu kontrolnym Paddle:

```ini
CASHIER_WEBHOOK=https://example.com/my-paddle-webhook-url
```

<a name="verifying-webhook-signatures"></a>
### Weryfikacja Podpisów Webhook

Aby zabezpieczyć swoje webhooki, możesz użyć [podpisów webhooków Paddle](https://developer.paddle.com/webhooks/signature-verification). Dla wygody Cashier automatycznie zawiera middleware, który sprawdza, czy przychodzące żądanie webhooka Paddle jest prawidłowe.

Aby włączyć weryfikację webhooka, upewnij się, że zmienna środowiskowa `PADDLE_WEBHOOK_SECRET` jest zdefiniowana w pliku `.env` Twojej aplikacji. Sekret webhooka można pobrać z panelu konta Paddle.

<a name="single-charges"></a>
## Pojedyncze Opłaty

<a name="charging-for-products"></a>
### Pobieranie Opłat za Produkty

Jeśli chcesz zainicjować zakup produktu dla klienta, możesz użyć metody `checkout` na instancji modelu rozliczeniowego, aby wygenerować sesję kasy dla zakupu. Metoda `checkout` akceptuje jeden lub więcej identyfikatorów cen. W razie potrzeby można użyć tablicy asocjacyjnej, aby podać ilość kupowanego produktu:

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $request->user()->checkout(['pri_tshirt', 'pri_socks' => 5]);

    return view('buy', ['checkout' => $checkout]);
});
```

Po wygenerowaniu sesji kasy możesz użyć dostarczonego przez Cashier [komponentu Blade](#overlay-checkout) `paddle-button`, aby umożliwić użytkownikowi oglądanie widgetu kasy Paddle i zakończenie zakupu:

```blade
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Buy
</x-paddle-button>
```

Sesja kasy ma metodę `customData`, która pozwala przekazywać dowolne niestandardowe dane do podstawowego tworzenia transakcji. Zapoznaj się z [dokumentacją Paddle](https://developer.paddle.com/build/transactions/custom-data), aby dowiedzieć się więcej o opcjach dostępnych podczas przekazywania niestandardowych danych:

```php
$checkout = $user->checkout('pri_tshirt')
    ->customData([
        'custom_option' => $value,
    ]);
```

<a name="refunding-transactions"></a>
### Zwroty Transakcji

Zwroty transakcji zwracają zwracaną kwotę na metodę płatności klienta, która była używana w momencie zakupu. Jeśli musisz zwracać zakup Paddle, możesz użyć metody `refund` na modelu `Cashier\Paddle\Transaction`. Ta metoda akceptuje powód jako pierwszy argument, jeden lub więcej identyfikatorów cen do zwrócenia z opcjonalnymi kwotami jako tablica asocjacyjna. Możesz pobrać transakcje dla danego modelu rozliczeniowego za pomocą metody `transactions`.

Na przykład wyobraźmy sobie, że chcemy zwrócić konkretną transakcję dla cen `pri_123` i `pri_456`. Chcemy w pełni zwrócić `pri_123`, ale tylko zwrócić dwa dolary za `pri_456`:

```php
use App\Models\User;

$user = User::find(1);

$transaction = $user->transactions()->first();

$response = $transaction->refund('Accidental charge', [
    'pri_123', // W pełni zwróć tę cenę...
    'pri_456' => 200, // Tylko częściowo zwróć tę cenę...
]);
```

Powyższy przykład zwraca konkretne pozycje w transakcji. Jeśli chcesz zwrócić całą transakcję, po prostu podaj powód:

```php
$response = $transaction->refund('Accidental charge');
```

Aby uzyskać więcej informacji o zwrotach, zapoznaj się z [dokumentacją zwrotów Paddle](https://developer.paddle.com/build/transactions/create-transaction-adjustments).

> [!WARNING]
> Zwroty muszą zawsze być zatwierdzone przez Paddle przed pełnym przetworzeniem.

<a name="crediting-transactions"></a>
### Uznanie Transakcji

Podobnie jak zwroty, możesz również uznawać transakcje. Uznawanie transakcji doda środki do salda klienta, aby mogły zostać użyte do przyszłych zakupów. Uznawanie transakcji może być wykonywane tylko dla transakcji zbieranych ręcznie, a nie dla transakcji zbieranych automatycznie (takich jak subskrypcje), ponieważ Paddle sam obsługuje uznania subskrypcji:

```php
$transaction = $user->transactions()->first();

// Credit a specific line item fully...
$response = $transaction->credit('Compensation', 'pri_123');
```

Aby uzyskać więcej informacji, [zobacz dokumentację Paddle dotyczącą uznania](https://developer.paddle.com/build/transactions/create-transaction-adjustments).

> [!WARNING]
> Uznania mogą być stosowane tylko dla transakcji zbieranych ręcznie. Transakcje zbierane automatycznie są uznawane przez sam Paddle.

<a name="transactions"></a>
## Transakcje

Możesz łatwo pobrać tablicę transakcji modelu rozliczeniowego za pośrednictwem właściwości `transactions`:

```php
use App\Models\User;

$user = User::find(1);

$transactions = $user->transactions;
```

Transakcje reprezentują płatności za Twoje produkty i zakupy i są towarzyszy fakturami. Tylko zakończone transakcje są przechowywane w bazie danych aplikacji.

Podczas wyświetlania transakcji dla klienta możesz użyć metod instancji transakcji do wyświetlenia odpowiednich informacji płatniczych. Na przykład możesz chcieć wymienić każdą transakcję w tabeli, umożliwiając użytkownikowi łatwe pobieranie dowolnej faktury:

```html
<table>
    @foreach ($transactions as $transaction)
        <tr>
            <td>{{ $transaction->billed_at->toFormattedDateString() }}</td>
            <td>{{ $transaction->total() }}</td>
            <td>{{ $transaction->tax() }}</td>
            <td><a href="{{ route('download-invoice', $transaction->id) }}" target="_blank">Download</a></td>
        </tr>
    @endforeach
</table>
```

Trasa `download-invoice` może wyglądać następująco:

```php
use Illuminate\Http\Request;
use Laravel\Paddle\Transaction;

Route::get('/download-invoice/{transaction}', function (Request $request, Transaction $transaction) {
    return $transaction->redirectToInvoicePdf();
})->name('download-invoice');
```

<a name="past-and-upcoming-payments"></a>
### Przeszłe i Nadchodzące Płatności

Możesz użyć metod `lastPayment` i `nextPayment`, aby pobrać i wyświetlić przeszłe lub nadchodzące płatności klienta dla subskrypcji cyklicznych:

```php
use App\Models\User;

$user = User::find(1);

$subscription = $user->subscription();

$lastPayment = $subscription->lastPayment();
$nextPayment = $subscription->nextPayment();
```

Obie te metody zwracają instancję `Laravel\Paddle\Payment`; jednak `lastPayment` zwróci `null`, gdy transakcje nie zostały jeszcze zsynchronizowane przez webhooki, podczas gdy `nextPayment` zwróci `null`, gdy cykl rozliczeniowy się zakończył (na przykład gdy subskrypcja została anulowana):

```blade
Next payment: {{ $nextPayment->amount() }} due on {{ $nextPayment->date()->format('d/m/Y') }}
```

<a name="testing"></a>
## Testowanie

Podczas testowania powinieneś ręcznie testować swój przepływ rozliczeń, aby upewnić się, że Twoja integracja działa zgodnie z oczekiwaniami.

Dla testów automatycznych, w tym tych wykonywanych w środowisku CI, możesz użyć [Klienta HTTP Laravel](/docs/{{version}}/http-client#testing) do fałszowania wywołań HTTP do Paddle. Chociaż nie testuje to rzeczywistych odpowiedzi z Paddle, zapewnia sposób testowania aplikacji bez faktycznego wywoływania API Paddle.
