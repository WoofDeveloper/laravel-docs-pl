# Laravel Cashier (Stripe)

- [Wprowadzenie](#introduction)
- [Aktualizacja Cashier](#upgrading-cashier)
- [Instalacja](#installation)
- [Konfiguracja](#configuration)
    - [Model rozliczeniowy](#billable-model)
    - [Klucze API](#api-keys)
    - [Konfiguracja waluty](#currency-configuration)
    - [Konfiguracja podatkowa](#tax-configuration)
    - [Logowanie](#logging)
    - [Używanie własnych modeli](#using-custom-models)
- [Szybki start](#quickstart)
    - [Sprzedaż produktów](#quickstart-selling-products)
    - [Sprzedaż subskrypcji](#quickstart-selling-subscriptions)
- [Klienci](#customers)
    - [Pobieranie klientów](#retrieving-customers)
    - [Tworzenie klientów](#creating-customers)
    - [Aktualizacja klientów](#updating-customers)
    - [Salda](#balances)
    - [Numery identyfikacji podatkowej](#tax-ids)
    - [Synchronizacja danych klienta ze Stripe](#syncing-customer-data-with-stripe)
    - [Portal rozliczeniowy](#billing-portal)
- [Metody płatności](#payment-methods)
    - [Przechowywanie metod płatności](#storing-payment-methods)
    - [Pobieranie metod płatności](#retrieving-payment-methods)
    - [Obecność metody płatności](#payment-method-presence)
    - [Aktualizacja domyślnej metody płatności](#updating-the-default-payment-method)
    - [Dodawanie metod płatności](#adding-payment-methods)
    - [Usuwanie metod płatności](#deleting-payment-methods)
- [Subskrypcje](#subscriptions)
    - [Tworzenie subskrypcji](#creating-subscriptions)
    - [Sprawdzanie statusu subskrypcji](#checking-subscription-status)
    - [Zmiana cen](#changing-prices)
    - [Ilość subskrypcji](#subscription-quantity)
    - [Subskrypcje z wieloma produktami](#subscriptions-with-multiple-products)
    - [Wiele subskrypcji](#multiple-subscriptions)
    - [Rozliczanie oparte na użyciu](#usage-based-billing)
    - [Podatki od subskrypcji](#subscription-taxes)
    - [Data kotwicy subskrypcji](#subscription-anchor-date)
    - [Anulowanie subskrypcji](#cancelling-subscriptions)
    - [Wznawianie subskrypcji](#resuming-subscriptions)
- [Okresy próbne subskrypcji](#subscription-trials)
    - [Z metodą płatności z góry](#with-payment-method-up-front)
    - [Bez metody płatności z góry](#without-payment-method-up-front)
    - [Przedłużanie okresów próbnych](#extending-trials)
- [Obsługa webhooków Stripe](#handling-stripe-webhooks)
    - [Definiowanie procedur obsługi zdarzeń webhook](#defining-webhook-event-handlers)
    - [Weryfikacja podpisów webhook](#verifying-webhook-signatures)
- [Pojedyncze opłaty](#single-charges)
    - [Prosta opłata](#simple-charge)
    - [Opłata z fakturą](#charge-with-invoice)
    - [Tworzenie intencji płatności](#creating-payment-intents)
    - [Zwroty opłat](#refunding-charges)
- [Faktury](#invoices)
    - [Pobieranie faktur](#retrieving-invoices)
    - [Nadchodzące faktury](#upcoming-invoices)
    - [Podgląd faktur subskrypcji](#previewing-subscription-invoices)
    - [Generowanie plików PDF faktur](#generating-invoice-pdfs)
- [Kasa](#checkout)
    - [Kasy produktów](#product-checkouts)
    - [Kasy pojedynczej opłaty](#single-charge-checkouts)
    - [Kasy subskrypcji](#subscription-checkouts)
    - [Zbieranie numerów identyfikacji podatkowej](#collecting-tax-ids)
    - [Kasy dla gości](#guest-checkouts)
- [Obsługa nieudanych płatności](#handling-failed-payments)
    - [Potwierdzanie płatności](#confirming-payments)
- [Silne uwierzytelnianie klienta (SCA)](#strong-customer-authentication)
    - [Płatności wymagające dodatkowego potwierdzenia](#payments-requiring-additional-confirmation)
    - [Powiadomienia o płatnościach poza sesją](#off-session-payment-notifications)
- [Stripe SDK](#stripe-sdk)
- [Testowanie](#testing)

<a name="introduction"></a>
## Wprowadzenie

[Laravel Cashier Stripe](https://github.com/laravel/cashier-stripe) zapewnia wyrazisty, płynny interfejs do usług [Stripe](https://stripe.com) rozliczania subskrypcji. Obsługuje niemal cały kod boilerplate rozliczania subskrypcji, którego pisania się obawiasz. Oprócz podstawowego zarządzania subskrypcjami, Cashier może obsługiwać kupony, zamianę subskrypcji, "ilości" subskrypcji, okresy karencji anulowania, a nawet generować pliki PDF faktur.

<a name="upgrading-cashier"></a>
## Aktualizacja Cashier

Podczas aktualizacji do nowej wersji Cashier ważne jest, aby dokładnie przejrzeć [przewodnik po aktualizacji](https://github.com/laravel/cashier-stripe/blob/master/UPGRADE.md).

> [!WARNING]
> Aby zapobiec zmianom wprowadzającym niezgodności, Cashier używa stałej wersji API Stripe. Cashier 16 wykorzystuje wersję API Stripe `2025-06-30.basil`. Wersja API Stripe będzie aktualizowana w wersjach pomocniczych, aby korzystać z nowych funkcji i ulepszeń Stripe.

<a name="installation"></a>
## Instalacja

Najpierw zainstaluj pakiet Cashier dla Stripe za pomocą menedżera pakietów Composer:

```shell
composer require laravel/cashier
```

Po zainstalowaniu pakietu opublikuj migracje Cashier za pomocą polecenia Artisan `vendor:publish`:

```shell
php artisan vendor:publish --tag="cashier-migrations"
```

Następnie wykonaj migrację bazy danych:

```shell
php artisan migrate
```

Migracje Cashier dodadzą kilka kolumn do tabeli `users`. Utworzą również nową tabelę `subscriptions` do przechowywania wszystkich subskrypcji klientów oraz tabelę `subscription_items` dla subskrypcji z wieloma cenami.

Jeśli chcesz, możesz również opublikować plik konfiguracyjny Cashier za pomocą polecenia Artisan `vendor:publish`:

```shell
php artisan vendor:publish --tag="cashier-config"
```

Na koniec, aby upewnić się, że Cashier prawidłowo obsługuje wszystkie zdarzenia Stripe, pamiętaj o [skonfigurowaniu obsługi webhooków Cashier](#handling-stripe-webhooks).

> [!WARNING]
> Stripe zaleca, aby każda kolumna używana do przechowywania identyfikatorów Stripe była wrażliwa na wielkość liter. Dlatego należy upewnić się, że sortowanie kolumny `stripe_id` jest ustawione na `utf8_bin` podczas korzystania z MySQL. Więcej informacji na ten temat można znaleźć w [dokumentacji Stripe](https://stripe.com/docs/upgrades#what-changes-does-stripe-consider-to-be-backwards-compatible).

<a name="configuration"></a>
## Konfiguracja

<a name="billable-model"></a>
### Model rozliczeniowy

Przed użyciem Cashier dodaj cechę `Billable` do definicji modelu rozliczeniowego. Zazwyczaj będzie to model `App\Models\User`. Ta cecha udostępnia różne metody umożliwiające wykonywanie typowych zadań rozliczeniowych, takich jak tworzenie subskrypcji, stosowanie kuponów i aktualizowanie informacji o metodzie płatności:

```php
use Laravel\Cashier\Billable;

class User extends Authenticatable
{
    use Billable;
}
```

Cashier zakłada, że model rozliczeniowy będzie klasą `App\Models\User` dostarczaną z Laravel. Jeśli chcesz to zmienić, możesz określić inny model za pomocą metody `useCustomerModel`. Ta metoda powinna być zazwyczaj wywoływana w metodzie `boot` klasy `AppServiceProvider`:

```php
use App\Models\Cashier\User;
use Laravel\Cashier\Cashier;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::useCustomerModel(User::class);
}
```

> [!WARNING]
> Jeśli używasz modelu innego niż dostarczany przez Laravel model `App\Models\User`, będziesz musiał opublikować i zmienić dostarczone [migracje Cashier](#installation), aby dopasować je do nazwy tabeli alternatywnego modelu.

<a name="api-keys"></a>
### Klucze API

Następnie powinieneś skonfigurować klucze API Stripe w pliku `.env` aplikacji. Możesz pobrać klucze API Stripe z panelu kontrolnego Stripe:

```ini
STRIPE_KEY=your-stripe-key
STRIPE_SECRET=your-stripe-secret
STRIPE_WEBHOOK_SECRET=your-stripe-webhook-secret
```

> [!WARNING]
> Powinieneś upewnić się, że zmienna środowiskowa `STRIPE_WEBHOOK_SECRET` jest zdefiniowana w pliku `.env` aplikacji, ponieważ ta zmienna służy do zapewnienia, że przychodzące webhooki faktycznie pochodzą ze Stripe.

<a name="currency-configuration"></a>
### Konfiguracja waluty

Domyślną walutą Cashier jest dolar amerykański (USD). Możesz zmienić domyślną walutę, ustawiając zmienną środowiskową `CASHIER_CURRENCY` w pliku `.env` aplikacji:

```ini
CASHIER_CURRENCY=eur
```

Oprócz konfigurowania waluty Cashier, możesz również określić ustawienia regionalne, które będą używane podczas formatowania wartości pieniężnych do wyświetlania na fakturach. Wewnętrznie Cashier wykorzystuje [klasę PHP `NumberFormatter`](https://www.php.net/manual/en/class.numberformatter.php) do ustawiania ustawień regionalnych waluty:

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> [!WARNING]
> Aby używać ustawień regionalnych innych niż `en`, upewnij się, że rozszerzenie PHP `ext-intl` jest zainstalowane i skonfigurowane na serwerze.

<a name="tax-configuration"></a>
### Konfiguracja podatkowa

Dzięki [Stripe Tax](https://stripe.com/tax) możliwe jest automatyczne obliczanie podatków dla wszystkich faktur generowanych przez Stripe. Możesz włączyć automatyczne obliczanie podatków, wywołując metodę `calculateTaxes` w metodzie `boot` klasy `App\Providers\AppServiceProvider` aplikacji:

```php
use Laravel\Cashier\Cashier;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::calculateTaxes();
}
```

Po włączeniu obliczania podatków wszystkie nowe subskrypcje i wszystkie jednorazowe faktury, które zostaną wygenerowane, otrzymają automatyczne obliczanie podatków.

Aby ta funkcja działała poprawnie, dane rozliczeniowe klienta, takie jak nazwa, adres i numer identyfikacji podatkowej, muszą być zsynchronizowane ze Stripe. Możesz użyć metod [synchronizacji danych klienta](#syncing-customer-data-with-stripe) i [numeru identyfikacji podatkowej](#tax-ids) oferowanych przez Cashier, aby to osiągnąć.

<a name="logging"></a>
### Logowanie

Cashier pozwala określić kanał dziennika, który ma być używany podczas rejestrowania krytycznych błędów Stripe. Możesz określić kanał dziennika, definiując zmienną środowiskową `CASHIER_LOGGER` w pliku `.env` aplikacji:

```ini
CASHIER_LOGGER=stack
```

Wyjątki generowane przez wywołania API do Stripe będą rejestrowane przez domyślny kanał dziennika aplikacji.

<a name="using-custom-models"></a>
### Używanie własnych modeli

Możesz swobodnie rozszerzać modele używane wewnętrznie przez Cashier, definiując własny model i rozszerzając odpowiedni model Cashier:

```php
use Laravel\Cashier\Subscription as CashierSubscription;

class Subscription extends CashierSubscription
{
    // ...
}
```

Po zdefiniowaniu modelu możesz poinstruować Cashier, aby używał niestandardowego modelu za pomocą klasy `Laravel\Cashier\Cashier`. Zazwyczaj powinieneś poinformować Cashier o niestandardowych modelach w metodzie `boot` klasy `App\Providers\AppServiceProvider` aplikacji:

```php
use App\Models\Cashier\Subscription;
use App\Models\Cashier\SubscriptionItem;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::useSubscriptionModel(Subscription::class);
    Cashier::useSubscriptionItemModel(SubscriptionItem::class);
}
```

<a name="quickstart"></a>
## Szybki start

<a name="quickstart-selling-products"></a>
### Sprzedaż produktów

> [!NOTE]
> Przed wykorzystaniem Stripe Checkout powinieneś zdefiniować produkty ze stałymi cenami w panelu Stripe. Ponadto powinieneś [skonfigurować obsługę webhooków Cashier](#handling-stripe-webhooks).

Oferowanie rozliczeń za produkty i subskrypcje za pośrednictwem aplikacji może być onieśmielające. Jednak dzięki Cashier i [Stripe Checkout](https://stripe.com/payments/checkout) możesz łatwo budować nowoczesne, solidne integracje płatności.

Aby obciążyć klientów za niepowtarzające się, jednorazowe produkty, użyjemy Cashier, aby skierować klientów do Stripe Checkout, gdzie podadzą swoje dane płatnicze i potwierdzą zakup. Po dokonaniu płatności przez Checkout klient zostanie przekierowany na adres URL sukcesu wybrany przez Ciebie w aplikacji:

```php
use Illuminate\Http\Request;

Route::get('/checkout', function (Request $request) {
    $stripePriceId = 'price_deluxe_album';

    $quantity = 1;

    return $request->user()->checkout([$stripePriceId => $quantity], [
        'success_url' => route('checkout-success'),
        'cancel_url' => route('checkout-cancel'),
    ]);
})->name('checkout');

Route::view('/checkout/success', 'checkout.success')->name('checkout-success');
Route::view('/checkout/cancel', 'checkout.cancel')->name('checkout-cancel');
```

Jak widać w powyższym przykładzie, użyjemy dostarczonej przez Cashier metody `checkout`, aby przekierować klienta do Stripe Checkout dla danego "identyfikatora ceny". Przy użyciu Stripe "ceny" odnoszą się do [zdefiniowanych cen dla konkretnych produktów](https://stripe.com/docs/products-prices/how-products-and-prices-work).

Jeśli to konieczne, metoda `checkout` automatycznie utworzy klienta w Stripe i połączy ten rekord klienta Stripe z odpowiednim użytkownikiem w bazie danych aplikacji. Po zakończeniu sesji realizacji transakcji klient zostanie przekierowany na dedykowaną stronę sukcesu lub anulowania, gdzie możesz wyświetlić komunikat informacyjny dla klienta.

<a name="providing-meta-data-to-stripe-checkout"></a>
#### Dostarczanie metadanych do Stripe Checkout

Podczas sprzedaży produktów powszechną praktyką jest śledzenie zrealizowanych zamówień i zakupionych produktów za pomocą modeli `Cart` i `Order` zdefiniowanych we własnej aplikacji. Podczas przekierowywania klientów do Stripe Checkout w celu zakończenia zakupu może być konieczne podanie istniejącego identyfikatora zamówienia, aby móc powiązać zakończony zakup z odpowiednim zamówieniem, gdy klient zostanie przekierowany z powrotem do aplikacji.

Aby to osiągnąć, możesz przekazać tablicę `metadata` do metody `checkout`. Załóżmy, że oczekujące `Order` jest tworzone w naszej aplikacji, gdy użytkownik rozpoczyna proces realizacji transakcji. Pamiętaj, że modele `Cart` i `Order` w tym przykładzie są ilustracyjne i nie są dostarczane przez Cashier. Możesz swobodnie implementować te koncepcje w oparciu o potrzeby własnej aplikacji:

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

    return $request->user()->checkout($order->price_ids, [
        'success_url' => route('checkout-success').'?session_id={CHECKOUT_SESSION_ID}',
        'cancel_url' => route('checkout-cancel'),
        'metadata' => ['order_id' => $order->id],
    ]);
})->name('checkout');
```

Jak widać w powyższym przykładzie, gdy użytkownik rozpoczyna proces realizacji transakcji, przekazujemy wszystkie powiązane identyfikatory cen Stripe koszyka / zamówienia do metody `checkout`. Oczywiście Twoja aplikacja jest odpowiedzialna za powiązanie tych elementów z "koszykiem" lub zamówieniem, gdy klient je dodaje. Przekazujemy również ID zamówienia do sesji Stripe Checkout za pośrednictwem tablicy `metadata`. Na koniec dodaliśmy zmienną szablonu `CHECKOUT_SESSION_ID` do trasy sukcesu Checkout. Gdy Stripe przekierowuje klientów z powrotem do aplikacji, ta zmienna szablonu zostanie automatycznie wypełniona ID sesji Checkout.

Następnie zbudujmy trasę sukcesu Checkout. Jest to trasa, do której użytkownicy zostaną przekierowani po zakończeniu zakupu przez Stripe Checkout. W ramach tej trasy możemy pobrać ID sesji Stripe Checkout i powiązaną instancję Stripe Checkout, aby uzyskać dostęp do naszych dostarczonych metadanych i odpowiednio zaktualizować zamówienie klienta:

```php
use App\Models\Order;
use Illuminate\Http\Request;
use Laravel\Cashier\Cashier;

Route::get('/checkout/success', function (Request $request) {
    $sessionId = $request->get('session_id');

    if ($sessionId === null) {
        return;
    }

    $session = Cashier::stripe()->checkout->sessions->retrieve($sessionId);

    if ($session->payment_status !== 'paid') {
        return;
    }

    $orderId = $session['metadata']['order_id'] ?? null;

    $order = Order::findOrFail($orderId);

    $order->update(['status' => 'completed']);

    return view('checkout-success', ['order' => $order]);
})->name('checkout-success');
```

Zapoznaj się z dokumentacją Stripe, aby uzyskać więcej informacji o [danych zawartych w obiekcie sesji Checkout](https://stripe.com/docs/api/checkout/sessions/object).

<a name="quickstart-selling-subscriptions"></a>
### Sprzedaż subskrypcji

> [!NOTE]
> Przed wykorzystaniem Stripe Checkout powinieneś zdefiniować produkty ze stałymi cenami w panelu Stripe. Ponadto powinieneś [skonfigurować obsługę webhooków Cashier](#handling-stripe-webhooks).

Oferowanie rozliczeń za produkty i subskrypcje za pośrednictwem aplikacji może być onieśmielające. Jednak dzięki Cashier i [Stripe Checkout](https://stripe.com/payments/checkout) możesz łatwo budować nowoczesne, solidne integracje płatności.

Aby nauczyć się, jak sprzedawać subskrypcje za pomocą Cashier i Stripe Checkout, rozważmy prosty scenariusz usługi subskrypcyjnej z podstawowym planem miesięcznym (`price_basic_monthly`) i rocznym (`price_basic_yearly`). Te dwie ceny mogą być zgrupowane w produkcie "Basic" (`pro_basic`) w naszym panelu Stripe. Ponadto nasza usługa subskrypcyjna może oferować plan Expert jako `pro_expert`.

Po pierwsze, odkryjmy, jak klient może zasubskrybować nasze usługi. Oczywiście możesz sobie wyobrazić, że klient może kliknąć przycisk "subskrybuj" dla planu Basic na stronie cenowej naszej aplikacji. Ten przycisk lub link powinien kierować użytkownika do trasy Laravel, która tworzy sesję Stripe Checkout dla wybranego planu:

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_basic_monthly')
        ->trialDays(5)
        ->allowPromotionCodes()
        ->checkout([
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
});
```

Jak widać w powyższym przykładzie, przekierujemy klienta do sesji Stripe Checkout, która pozwoli mu zasubskrybować nasz plan Basic. Po udanej realizacji transakcji lub anulowaniu klient zostanie przekierowany z powrotem na adres URL, który przekazaliśmy do metody `checkout`. Aby wiedzieć, kiedy ich subskrypcja faktycznie się rozpoczęła (ponieważ niektóre metody płatności wymagają kilku sekund przetwarzania), będziemy musieli również [skonfigurować obsługę webhooków Cashier](#handling-stripe-webhooks).

Teraz, gdy klienci mogą rozpocząć subskrypcje, musimy ograniczyć dostęp do niektórych części naszej aplikacji, aby tylko zasubskrybowani użytkownicy mogli do nich uzyskiwać dostęp. Oczywiście zawsze możemy określić aktualny status subskrypcji użytkownika za pomocą metody `subscribed` dostarczonej przez cechę `Billable` Cashier:

```blade
@if ($user->subscribed())
    <p>Masz aktywną subskrypcję.</p>
@endif
```

Możemy nawet łatwo określić, czy użytkownik jest zasubskrybowany do konkretnego produktu lub ceny:

```blade
@if ($user->subscribedToProduct('pro_basic'))
    <p>Jesteś zasubskrybowany do naszego produktu Basic.</p>
@endif

@if ($user->subscribedToPrice('price_basic_monthly'))
    <p>Jesteś zasubskrybowany do naszego miesięcznego planu Basic.</p>
@endif
```

<a name="quickstart-building-a-subscribed-middleware"></a>
#### Tworzenie middleware dla zasubskrybowanych użytkowników

Dla wygody możesz chcieć utworzyć [middleware](/docs/{{version}}/middleware), który określa, czy przychodzące żądanie pochodzi od zasubskrybowanego użytkownika. Po zdefiniowaniu tego middleware możesz łatwo przypisać go do trasy, aby zapobiec dostępowi użytkowników, którzy nie są zasubskrybowani:

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
            // Przekieruj użytkownika na stronę rozliczeń i poproś go o subskrypcję...
            return redirect('/billing');
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

Oczywiście klienci mogą chcieć zmienić swój plan subskrypcji na inny produkt lub "poziom". Najprosts way to allow this is by directing customers to Stripe's [Customer Billing Portal](https://stripe.com/docs/no-code/customer-portal), which provides a hosted user interface that allows customers to download invoices, update their payment method, and change subscription plans.

First, define a link or button within your application that directs users to a Laravel route which we will utilize to initiate a Billing Portal session:

```blade
<a href="{{ route('billing') }}">
    Rozliczenia
</a>
```

Następnie zdefiniujmy trasę, która inicjuje sesję Portalu rozliczeniowego klienta Stripe i przekierowuje użytkownika do Portalu. Metoda `redirectToBillingPortal` przyjmuje adres URL, do którego użytkownicy powinni zostać zwróceni po wyjściu z Portalu:

```php
use Illuminate\Http\Request;

Route::get('/billing', function (Request $request) {
    return $request->user()->redirectToBillingPortal(route('dashboard'));
})->middleware(['auth'])->name('billing');
```

> [!NOTE]
> Dopóki masz skonfigurowaną obsługę webhooków Cashier, Cashier będzie automatycznie synchronizował tabele bazy danych związane z Cashier w aplikacji, sprawdzając przychodzące webhooki ze Stripe. Na przykład, gdy użytkownik anuluje subskrypcję za pośrednictwem Portalu rozliczeniowego klienta Stripe, Cashier otrzyma odpowiedni webhook i oznaczy subskrypcję jako "anulowaną" w bazie danych aplikacji.

<a name="customers"></a>
## Klienci

<a name="retrieving-customers"></a>
### Pobieranie klientów

Możesz pobrać klienta po jego identyfikatorze Stripe za pomocą metody `Cashier::findBillable`. Ta metoda zwróci instancję modelu rozliczeniowego:

```php
use Laravel\Cashier\Cashier;

$user = Cashier::findBillable($stripeId);
```

<a name="creating-customers"></a>
### Tworzenie klientów

Okazjonalnie możesz chcieć utworzyć klienta Stripe bez rozpoczynania subskrypcji. Możesz to osiągnąć, używając metody `createAsStripeCustomer`:

```php
$stripeCustomer = $user->createAsStripeCustomer();
```

Po utworzeniu klienta w Stripe możesz rozpocząć subskrypcję w późniejszym terminie. Możesz podać opcjonalną tablicę `$options`, aby przekazać dowolne dodatkowe [parametry tworzenia klienta obsługiwane przez API Stripe](https://stripe.com/docs/api/customers/create):

```php
$stripeCustomer = $user->createAsStripeCustomer($options);
```

Możesz użyć metody `asStripeCustomer`, jeśli chcesz zwrócić obiekt klienta Stripe dla modelu rozliczeniowego:

```php
$stripeCustomer = $user->asStripeCustomer();
```

Metoda `createOrGetStripeCustomer` może być użyta, jeśli chcesz pobrać obiekt klienta Stripe dla danego modelu rozliczeniowego, ale nie jesteś pewien, czy model rozliczeniowy jest już klientem w Stripe. Ta metoda utworzy nowego klienta w Stripe, jeśli jeszcze nie istnieje:

```php
$stripeCustomer = $user->createOrGetStripeCustomer();
```

<a name="updating-customers"></a>
### Aktualizacja klientów

Okazjonalnie możesz chcieć bezpośrednio zaktualizować klienta Stripe dodatkowymi informacjami. Możesz to osiągnąć, używając metody `updateStripeCustomer`. Ta metoda przyjmuje tablicę [opcji aktualizacji klienta obsługiwanych przez API Stripe](https://stripe.com/docs/api/customers/update):

```php
$stripeCustomer = $user->updateStripeCustomer($options);
```

<a name="balances"></a>
### Salda

Stripe umożliwia uznanie lub obciążenie "salda" klienta. Później to saldo zostanie uznane lub obciążone na nowych fakturach. Aby sprawdzić całkowite saldo klienta, możesz użyć metody `balance` dostępnej w modelu rozliczeniowym. Metoda `balance` zwróci sformatowaną reprezentację ciągu znaków salda w walucie klienta:

```php
$balance = $user->balance();
```

Aby uznać saldo klienta, możesz podać wartość do metody `creditBalance`. Jeśli chcesz, możesz również podać opis:

```php
$user->creditBalance(500, 'Dopłata dla klienta premium.');
```

Podanie wartości do metody `debitBalance` obciąży saldo klienta:

```php
$user->debitBalance(300, 'Kara za złe użycie.');
```

Metoda `applyBalance` utworzy nowe transakcje salda klienta dla klienta. Możesz pobrać te rekordy transakcji za pomocą metody `balanceTransactions`, co może być przydatne w celu zapewnienia dziennika uznania i obciążenia do przeglądu przez klienta:

```php
// Pobierz wszystkie transakcje...
$transactions = $user->balanceTransactions();

foreach ($transactions as $transaction) {
    // Kwota transakcji...
    $amount = $transaction->amount(); // $2.31

    // Pobierz powiązaną fakturę, gdy jest dostępna...
    $invoice = $transaction->invoice();
}
```

<a name="tax-ids"></a>
### Numery identyfikacji podatkowej

Cashier oferuje łatwy sposób zarządzania numerami identyfikacji podatkowej klienta. Na przykład metoda `taxIds` może być użyta do pobrania wszystkich [numerów identyfikacji podatkowej](https://stripe.com/docs/api/customer_tax_ids/object), które są przypisane do klienta jako kolekcja:

```php
$taxIds = $user->taxIds();
```

Możesz również pobrać określony numer identyfikacji podatkowej dla klienta po jego identyfikatorze:

```php
$taxId = $user->findTaxId('txi_belgium');
```

Możesz utworzyć nowy numer identyfikacji podatkowej, podając prawidłowy [typ](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-type) i wartość do metody `createTaxId`:

```php
$taxId = $user->createTaxId('eu_vat', 'BE0123456789');
```

Metoda `createTaxId` natychmiast doda numer VAT do konta klienta. [Weryfikacja numerów VAT jest również wykonywana przez Stripe](https://stripe.com/docs/invoicing/customer/tax-ids#validation); jednak jest to proces asynchroniczny. Możesz być powiadamiany o aktualizacjach weryfikacji, subskrybując zdarzenie webhooka `customer.tax_id.updated` i sprawdzając [parametr `verification` numerów VAT](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-verification). Więcej informacji na temat obsługi webhooków można znaleźć w [dokumentacji definiowania procedur obsługi webhooków](#handling-stripe-webhooks).

Możesz usunąć numer identyfikacji podatkowej za pomocą metody `deleteTaxId`:

```php
$user->deleteTaxId('txi_belgium');
```

<a name="syncing-customer-data-with-stripe"></a>
### Synchronizacja danych klienta ze Stripe

Zazwyczaj, gdy użytkownicy aplikacji aktualizują swoje imię, adres e-mail lub inne informacje, które są również przechowywane przez Stripe, powinieneś poinformować Stripe o aktualizacjach. Dzięki temu kopia informacji Stripe będzie zsynchronizowana z aplikacją.

Aby to zautomatyzować, możesz zdefiniować nasłuchiwacz zdarzeń w modelu rozliczeniowym, który reaguje na zdarzenie `updated` modelu. Następnie, w ramach nasłuchiwacza zdarzeń, możesz wywołać metodę `syncStripeCustomerDetails` na modelu:

```php
use App\Models\User;
use function Illuminate\Events\queueable;

/**
 * The "booted" method of the model.
 */
protected static function booted(): void
{
    static::updated(queueable(function (User $customer) {
        if ($customer->hasStripeId()) {
            $customer->syncStripeCustomerDetails();
        }
    }));
}
```

Teraz za każdym razem, gdy model klienta zostanie zaktualizowany, jego informacje zostaną zsynchronizowane ze Stripe. Dla wygody Cashier automatycznie zsynchronizuje informacje klienta ze Stripe podczas początkowego tworzenia klienta.

Możesz dostosować kolumny używane do synchronizacji informacji o kliencie ze Stripe, nadpisując różne metody dostarczone przez Cashier. Na przykład możesz nadpisać metodę `stripeName`, aby dostosować atrybut, który powinien być uważany za "imię" klienta, gdy Cashier synchronizuje informacje o kliencie ze Stripe:

```php
/**
 * Get the customer name that should be synced to Stripe.
 */
public function stripeName(): string|null
{
    return $this->company_name;
}
```

Podobnie możesz nadpisać metody `stripeEmail`, `stripePhone` (maksymalnie 20 znaków), `stripeAddress` i `stripePreferredLocales`. Te metody będą synchronizować informacje z odpowiednimi parametrami klienta podczas [aktualizowania obiektu klienta Stripe](https://stripe.com/docs/api/customers/update). Jeśli chcesz przejąć pełną kontrolę nad procesem synchronizacji informacji o kliencie, możesz nadpisać metodę `syncStripeCustomerDetails`.

<a name="billing-portal"></a>
### Portal rozliczeniowy

Stripe oferuje [łatwy sposób skonfigurowania portalu rozliczeniowego](https://stripe.com/docs/billing/subscriptions/customer-portal), aby klient mógł zarządzać subskrypcją, metodami płatności i przeglądać historię rozliczeń. Możesz przekierować użytkowników do portalu rozliczeniowego, wywołując metodę `redirectToBillingPortal` na modelu rozliczeniowym z kontrolera lub trasy:

```php
use Illuminate\Http\Request;

Route::get('/billing-portal', function (Request $request) {
    return $request->user()->redirectToBillingPortal();
});
```

Domyślnie, gdy użytkownik zakończy zarządzanie subskrypcją, będzie mógł powrócić do trasy `home` aplikacji za pośrednictwem linku w portalu rozliczeniowym Stripe. Możesz podać niestandardowy adres URL, do którego użytkownik powinien powrócić, przekazując adres URL jako argument do metody `redirectToBillingPortal`:

```php
use Illuminate\Http\Request;

Route::get('/billing-portal', function (Request $request) {
    return $request->user()->redirectToBillingPortal(route('billing'));
});
```

Jeśli chcesz wygenerować adres URL do portalu rozliczeniowego bez generowania odpowiedzi przekierowania HTTP, możesz wywołać metodę `billingPortalUrl`:

```php
$url = $request->user()->billingPortalUrl(route('billing'));
```

<a name="payment-methods"></a>
## Metody płatności

<a name="storing-payment-methods"></a>
### Przechowywanie metod płatności

Aby tworzyć subskrypcje lub wykonywać jednorazowe opłaty za pomocą Stripe, musisz przechować metodę płatności i pobrać jej identyfikator ze Stripe. Podejście użyte do osiągnięcia tego różni się w zależności od tego, czy planujesz używać metody płatności do subskrypcji czy pojedynczych opłat, więc zbadamy obie poniżej.

<a name="payment-methods-for-subscriptions"></a>
#### Metody płatności dla subskrypcji

Podczas przechowywania informacji o karcie kredytowej klienta do przyszłego wykorzystania przez subskrypcję, należy użyć API Stripe "Setup Intents" w celu bezpiecznego zebrania danych metody płatności klienta. "Setup Intent" wskazuje Stripe zamiar obciążenia metody płatności klienta. Cecha `Billable` w Cashier zawiera metodę `createSetupIntent` do łatwego tworzenia nowego Setup Intent. Powinieneś wywołać tę metodę z trasy lub kontrolera, który będzie renderował formularz zbierający szczegóły metody płatności klienta:

```php
return view('update-payment-method', [
    'intent' => $user->createSetupIntent()
]);
```

Po utworzeniu Setup Intent i przekazaniu go do widoku, powinieneś dołączyć jego sekret do elementu, który będzie zbierał metodę płatności. Na przykład, rozważ ten formularz "aktualizacji metody płatności":

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button" data-secret="{{ $intent->client_secret }}">
    Update Payment Method
</button>
```

Następnie biblioteka Stripe.js może być użyta do dołączenia [Stripe Element](https://stripe.com/docs/stripe-js) do formularza i bezpiecznego zebrania danych płatniczych klienta:

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

Następnie karta może zostać zweryfikowana, a bezpieczny "identyfikator metody płatności" może zostać pobrany ze Stripe za pomocą [metody `confirmCardSetup` Stripe](https://stripe.com/docs/js/setup_intents/confirm_card_setup):

```js
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');
const clientSecret = cardButton.dataset.secret;

cardButton.addEventListener('click', async (e) => {
    const { setupIntent, error } = await stripe.confirmCardSetup(
        clientSecret, {
            payment_method: {
                card: cardElement,
                billing_details: { name: cardHolderName.value }
            }
        }
    );

    if (error) {
        // Display "error.message" to the user...
    } else {
        // The card has been verified successfully...
    }
});
```

Po zweryfikowaniu karty przez Stripe, możesz przekazać wynikowy identyfikator `setupIntent.payment_method` do aplikacji Laravel, gdzie może zostać przypisany do klienta. Metoda płatności może być [dodana jako nowa metoda płatności](#adding-payment-methods) lub [użyta do aktualizacji domyślnej metody płatności](#updating-the-default-payment-method). Możesz również natychmiast użyć identyfikatora metody płatności do [utworzenia nowej subskrypcji](#creating-subscriptions).

> [!NOTE]
> Jeśli chcesz dowiedzieć się więcej o Setup Intents i zbieraniu danych płatniczych klienta, zapoznaj się z [tym przeglądem dostarczonym przez Stripe](https://stripe.com/docs/payments/save-and-reuse#php).

<a name="payment-methods-for-single-charges"></a>
#### Metody płatności dla pojedynczych opłat

Oczywiście, podczas dokonywania pojedynczej opłaty z metody płatności klienta, będziemy musieli użyć identyfikatora metody płatności tylko raz. Ze względu na ograniczenia Stripe, nie możesz używać przechowywanej domyślnej metody płatności klienta do pojedynczych opłat. Musisz pozwolić klientowi wprowadzić dane metody płatności za pomocą biblioteki Stripe.js. Na przykład, rozważ następujący formularz:

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button">
    Process Payment
</button>
```

Po zdefiniowaniu takiego formularza biblioteka Stripe.js może zostać użyta do przypisania [Stripe Element](https://stripe.com/docs/stripe-js) do formularza i bezpiecznego zebrania danych płatniczych klienta:

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

Następnie karta może zostać zweryfikowana, a bezpieczny "identyfikator metody płatności" może zostać pobrany ze Stripe za pomocą [metody `createPaymentMethod` Stripe](https://stripe.com/docs/stripe-js/reference#stripe-create-payment-method):

```js
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');

cardButton.addEventListener('click', async (e) => {
    const { paymentMethod, error } = await stripe.createPaymentMethod(
        'card', cardElement, {
            billing_details: { name: cardHolderName.value }
        }
    );

    if (error) {
        // Wyświetl "error.message" użytkownikowi...
    } else {
        // Karta została pomyślnie zweryfikowana...
    }
});
```

Jeśli karta zostanie pomyślnie zweryfikowana, możesz przekazać `paymentMethod.id` do aplikacji Laravel i przetworzyć [pojedynczą opłatę](#simple-charge).

<a name="retrieving-payment-methods"></a>
### Pobieranie metod płatności

Metoda `paymentMethods` na instancji modelu rozliczeniowego zwraca kolekcję instancji `Laravel\Cashier\PaymentMethod`:

```php
$paymentMethods = $user->paymentMethods();
```

Domyślnie ta metoda zwraca metody płatności wszystkich typów. Aby pobrać metody płatności określonego typu, możesz przekazać `type` jako argument do metody:

```php
$paymentMethods = $user->paymentMethods('sepa_debit');
```

Aby pobrać domyślną metodę płatności klienta, można użyć metody `defaultPaymentMethod`:

```php
$paymentMethod = $user->defaultPaymentMethod();
```

Możesz pobrać konkretną metodę płatności, która jest przypisana do modelu rozliczeniowego, używając metody `findPaymentMethod`:

```php
$paymentMethod = $user->findPaymentMethod($paymentMethodId);
```

<a name="payment-method-presence"></a>
### Obecność metody płatności

Aby określić, czy model rozliczeniowy ma domyślną metodę płatności przypiętą do swojego konta, wywołaj metodę `hasDefaultPaymentMethod`:

```php
if ($user->hasDefaultPaymentMethod()) {
    // ...
}
```

Możesz użyć metody `hasPaymentMethod`, aby określić, czy model rozliczeniowy ma przynajmniej jedną metodę płatności przypiętą do swojego konta:

```php
if ($user->hasPaymentMethod()) {
    // ...
}
```

Ta metoda określi, czy model rozliczeniowy ma w ogóle jakąkolwiek metodę płatności. Aby określić, czy istnieje metoda płatności określonego typu dla modelu, możesz przekazać `type` jako argument do metody:

```php
if ($user->hasPaymentMethod('sepa_debit')) {
    // ...
}
```

<a name="updating-the-default-payment-method"></a>
### Aktualizacja domyślnej metody płatności

Metoda `updateDefaultPaymentMethod` może być użyta do aktualizacji informacji o domyślnej metodzie płatności klienta. Ta metoda akceptuje identyfikator metody płatności Stripe i przypisze nową metodę płatności jako domyślną metodę płatności do rozliczeń:

```php
$user->updateDefaultPaymentMethod($paymentMethod);
```

Aby zsynchronizować informacje o domyślnej metodzie płatności z informacjami o domyślnej metodzie płatności klienta w Stripe, możesz użyć metody `updateDefaultPaymentMethodFromStripe`:

```php
$user->updateDefaultPaymentMethodFromStripe();
```

> [!WARNING]
> Domyślna metoda płatności klienta może być używana tylko do fakturowania i tworzenia nowych subskrypcji. Ze względu na ograniczenia narzucone przez Stripe, nie może być używana do pojedynczych opłat.

<a name="adding-payment-methods"></a>
### Dodawanie metod płatności

Aby dodać nową metodę płatności, możesz wywołać metodę `addPaymentMethod` na modelu rozliczeniowym, przekazując identyfikator metody płatności:

```php
$user->addPaymentMethod($paymentMethod);
```

> [!NOTE]
> Aby dowiedzieć się, jak pobrać identyfikatory metod płatności, zapoznaj się z [dokumentacją przechowywania metod płatności](#storing-payment-methods).

<a name="deleting-payment-methods"></a>
### Usuwanie metod płatności

Aby usunąć metodę płatności, możesz wywołać metodę `delete` na instancji `Laravel\Cashier\PaymentMethod`, którą chcesz usunąć:

```php
$paymentMethod->delete();
```

Metoda `deletePaymentMethod` usunie konkretną metodę płatności z modelu rozliczeniowego:

```php
$user->deletePaymentMethod('pm_visa');
```

Metoda `deletePaymentMethods` usunie wszystkie informacje o metodach płatności dla modelu rozliczeniowego:

```php
$user->deletePaymentMethods();
```

Domyślnie ta metoda usunie metody płatności wszystkich typów. Aby usunąć metody płatności określonego typu, możesz przekazać `type` jako argument do metody:

```php
$user->deletePaymentMethods('sepa_debit');
```

> [!WARNING]
> Jeśli użytkownik ma aktywną subskrypcję, aplikacja nie powinna pozwalać mu usunąć domyślnej metody płatności.

<a name="subscriptions"></a>
## Subskrypcje

Subskrypcje zapewniają sposób na skonfigurowanie cyklicznych płatności dla klientów. Subskrypcje Stripe zarządzane przez Cashier zapewniają obsługę wielu cen subskrypcji, ilości subskrypcji, okresów próbnych i więcej.

<a name="creating-subscriptions"></a>
### Tworzenie subskrypcji

Aby utworzyć subskrypcję, najpierw pobierz instancję modelu rozliczeniowego, która zazwyczaj będzie instancją `App\Models\User`. Po pobraniu instancji modelu możesz użyć metody `newSubscription` do utworzenia subskrypcji modelu:

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription(
        'default', 'price_monthly'
    )->create($request->paymentMethodId);

    // ...
});
```

Pierwszy argument przekazany do metody `newSubscription` powinien być wewnętrznym typem subskrypcji. Jeśli aplikacja oferuje tylko jedną subskrypcję, możesz nazwać ją `default` lub `primary`. Ten typ subskrypcji jest przeznaczony tylko do wewnętrznego użytku aplikacji i nie powinien być pokazywany użytkownikom. Ponadto nie powinien zawierać spacji i nie powinien być nigdy zmieniany po utworzeniu subskrypcji. Drugi argument to konkretna cena, którą użytkownik subskrybuje. Ta wartość powinna odpowiadać identyfikatorowi ceny w Stripe.

Metoda `create`, która akceptuje [identyfikator metody płatności Stripe](#storing-payment-methods) lub obiekt Stripe `PaymentMethod`, rozpocznie subskrypcję, a także zaktualizuje bazę danych o identyfikatorze klienta Stripe modelu rozliczeniowego i innych istotnych informacjach rozliczeniowych.

> [!WARNING]
> Przekazanie identyfikatora metody płatności bezpośrednio do metody `create` subskrypcji automatycznie doda ją do przechowywanych metod płatności użytkownika.

<a name="collecting-recurring-payments-via-invoice-emails"></a>
#### Zbieranie cyklicznych płatności przez faktury e-mail

Zamiast automatycznego zbierania cyklicznych płatności klienta, możesz poinstruować Stripe, aby wysyłał fakturę e-mailem do klienta za każdym razem, gdy jego cykliczna płatność jest należna. Następnie klient może ręcznie opłacić fakturę po jej otrzymaniu. Klient nie musi podawać metody płatności z góry podczas zbierania cyklicznych płatności przez faktury:

```php
$user->newSubscription('default', 'price_monthly')->createAndSendInvoice();
```

Ilość czasu, jaką klient ma na zapłacenie faktury przed anulowaniem subskrypcji, jest określona przez opcję `days_until_due`. Domyślnie jest to 30 dni; jednakże możesz podać konkretną wartość dla tej opcji, jeśli chcesz:

```php
$user->newSubscription('default', 'price_monthly')->createAndSendInvoice([], [
    'days_until_due' => 30
]);
```

<a name="subscription-quantities"></a>
#### Ilości

Jeśli chcesz ustawić określoną [ilość](https://stripe.com/docs/billing/subscriptions/quantities) dla ceny podczas tworzenia subskrypcji, powinieneś wywołać metodę `quantity` na konstruktorze subskrypcji przed utworzeniem subskrypcji:

```php
$user->newSubscription('default', 'price_monthly')
    ->quantity(5)
    ->create($paymentMethod);
```

<a name="additional-details"></a>
#### Dodatkowe szczegóły

Jeśli chcesz określić dodatkowe opcje [klienta](https://stripe.com/docs/api/customers/create) lub [subskrypcji](https://stripe.com/docs/api/subscriptions/create) obsługiwane przez Stripe, możesz to zrobić, przekazując je jako drugi i trzeci argument do metody `create`:

```php
$user->newSubscription('default', 'price_monthly')->create($paymentMethod, [
    'email' => $email,
], [
    'metadata' => ['note' => 'Some extra information.'],
]);
```

<a name="coupons"></a>
#### Kupony

Jeśli chcesz zastosować kupon podczas tworzenia subskrypcji, możesz użyć metody `withCoupon`:

```php
$user->newSubscription('default', 'price_monthly')
    ->withCoupon('code')
    ->create($paymentMethod);
```

Lub, jeśli chcesz zastosować [kod promocyjny Stripe](https://stripe.com/docs/billing/subscriptions/discounts/codes), możesz użyć metody `withPromotionCode`:

```php
$user->newSubscription('default', 'price_monthly')
    ->withPromotionCode('promo_code_id')
    ->create($paymentMethod);
```

Podany identyfikator kodu promocyjnego powinien być identyfikatorem API Stripe przypisanym do kodu promocyjnego, a nie kodem promocyjnym widocznym dla klienta. Jeśli musisz znaleźć identyfikator kodu promocyjnego na podstawie danego kodu promocyjnego widocznego dla klienta, możesz użyć metody `findPromotionCode`:

```php
// Find a promotion code ID by its customer facing code...
$promotionCode = $user->findPromotionCode('SUMMERSALE');

// Find an active promotion code ID by its customer facing code...
$promotionCode = $user->findActivePromotionCode('SUMMERSALE');
```

W powyższym przykładzie zwrócony obiekt `$promotionCode` jest instancją `Laravel\Cashier\PromotionCode`. Ta klasa dekoruje bazowy obiekt `Stripe\PromotionCode`. Możesz pobrać kupon powiązany z kodem promocyjnym, wywołując metodę `coupon`:

```php
$coupon = $user->findPromotionCode('SUMMERSALE')->coupon();
```

Instancja kuponu pozwala określić kwotę rabatu i czy kupon reprezentuje rabat stały, czy procentowy:

```php
if ($coupon->isPercentage()) {
    return $coupon->percentOff().'%'; // 21.5%
} else {
    return $coupon->amountOff(); // $5.99
}
```

Możesz również pobrać rabaty, które są aktualnie zastosowane do klienta lub subskrypcji:

```php
$discount = $billable->discount();

$discount = $subscription->discount();
```

Zwracane instancje `Laravel\Cashier\Discount` dekorują podstawową instancję obiektu `Stripe\Discount`. Możesz pobrać kupon związany z tym rabatem, wywołując metodę `coupon`:

```php
$coupon = $subscription->discount()->coupon();
```

Jeśli chcesz zastosować nowy kupon lub kod promocyjny do klienta lub subskrypcji, możesz to zrobić za pomocą metod `applyCoupon` lub `applyPromotionCode`:

```php
$billable->applyCoupon('coupon_id');
$billable->applyPromotionCode('promotion_code_id');

$subscription->applyCoupon('coupon_id');
$subscription->applyPromotionCode('promotion_code_id');
```

Pamiętaj, że powinieneś używać identyfikatora API Stripe przypisanego do kodu promocyjnego, a nie kodu promocyjnego widocznego dla klienta. Do klienta lub subskrypcji można zastosować tylko jeden kupon lub kod promocyjny w danym momencie.

Więcej informacji na ten temat można znaleźć w dokumentacji Stripe dotyczącej [kuponów](https://stripe.com/docs/billing/subscriptions/coupons) i [kodów promocyjnych](https://stripe.com/docs/billing/subscriptions/coupons/codes).

<a name="adding-subscriptions"></a>
#### Dodawanie subskrypcji

Jeśli chcesz dodać subskrypcję do klienta, który już ma domyślną metodę płatności, możesz wywołać metodę `add` na konstruktorze subskrypcji:

```php
use App\Models\User;

$user = User::find(1);

$user->newSubscription('default', 'price_monthly')->add();
```

<a name="creating-subscriptions-from-the-stripe-dashboard"></a>
#### Tworzenie subskrypcji z dashboardu Stripe

Możesz również tworzyć subskrypcje bezpośrednio z dashboardu Stripe. W takim przypadku Cashier zsynchronizuje nowo dodane subskrypcje i przypisze im typ `default`. Aby dostosować typ subskrypcji przypisywany do subskrypcji utworzonych w dashboardzie, [zdefiniuj obsługę zdarzeń webhook](#defining-webhook-event-handlers).

Ponadto możesz utworzyć tylko jeden typ subskrypcji za pośrednictwem dashboardu Stripe. Jeśli aplikacja oferuje wiele subskrypcji, które używają różnych typów, tylko jeden typ subskrypcji może zostać dodany przez dashboard Stripe.

Na koniec zawsze upewnij się, że dodajesz tylko jedną aktywną subskrypcję na typ subskrypcji oferowany przez aplikację. Jeśli klient ma dwie subskrypcje `default`, tylko ostatnio dodana subskrypcja będzie używana przez Cashier, nawet jeśli obie zostałyby zsynchronizowane z bazą danych aplikacji.

<a name="checking-subscription-status"></a>
### Sprawdzanie statusu subskrypcji

Po zasubskrybowaniu aplikacji przez klienta możesz łatwo sprawdzić jego status subskrypcji za pomocą różnych wygodnych metod. Po pierwsze, metoda `subscribed` zwraca `true`, jeśli klient ma aktywną subskrypcję, nawet jeśli subskrypcja jest aktualnie w okresie próbnym. Metoda `subscribed` akceptuje typ subskrypcji jako pierwszy argument:

```php
if ($user->subscribed('default')) {
    // ...
}
```

Metoda `subscribed` również jest doskonałym kandydatem na [middleware trasy](/docs/{{version}}/middleware), pozwalając filtrować dostęp do tras i kontrolerów w oparciu o status subskrypcji użytkownika:

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
        if ($request->user() && ! $request->user()->subscribed('default')) {
            // Ten użytkownik nie jest płacącym klientem...
            return redirect('/billing');
        }

        return $next($request);
    }
}
```

Jeśli chcesz określić, czy użytkownik jest nadal w okresie próbnym, możesz użyć metody `onTrial`. Ta metoda może być przydatna do określenia, czy należy wyświetlić użytkownikowi ostrzeżenie, że nadal jest w okresie próbnym:

```php
if ($user->subscription('default')->onTrial()) {
    // ...
}
```

Metoda `subscribedToProduct` może być użyta do określenia, czy użytkownik jest zasubskrybowany do danego produktu w oparciu o identyfikator produktu Stripe. W Stripe produkty są kolekcjami cen. W tym przykładzie określimy, czy subskrypcja `default` użytkownika jest aktywnie zasubskrybowana do produktu "premium" aplikacji. Podany identyfikator produktu Stripe powinien odpowiadać jednemu z identyfikatorów Twoich produktów w dashboardzie Stripe:

```php
if ($user->subscribedToProduct('prod_premium', 'default')) {
    // ...
}
```

Przez przekazanie tablicy do metody `subscribedToProduct`, możesz określić, czy subskrypcja `default` użytkownika jest aktywnie zasubskrybowana do produktu "basic" lub "premium" aplikacji:

```php
if ($user->subscribedToProduct(['prod_basic', 'prod_premium'], 'default')) {
    // ...
}
```

Metoda `subscribedToPrice` może być użyta do określenia, czy subskrypcja klienta odpowiada danemu identyfikatorowi ceny:

```php
if ($user->subscribedToPrice('price_basic_monthly', 'default')) {
    // ...
}
```

Metoda `recurring` może być użyta do określenia, czy użytkownik jest aktualnie zasubskrybowany i nie jest już w okresie próbnym:

```php
if ($user->subscription('default')->recurring()) {
    // ...
}
```

> [!WARNING]
> Jeśli użytkownik ma dwie subskrypcje o tym samym typie, metoda `subscription` zawsze zwróci najnowszą subskrypcję. Na przykład użytkownik może mieć dwa rekordy subskrypcji typu `default`; jednak jedna z subskrypcji może być starą, wygasłą subskrypcją, podczas gdy druga jest aktualną, aktywną subskrypcją. Najnowsza subskrypcja zawsze będzie zwracana, podczas gdy starsze subskrypcje są przechowywane w bazie danych do celów historycznych.

<a name="cancelled-subscription-status"></a>
#### Status anulowanej subskrypcji

Aby określić, czy użytkownik był kiedyś aktywnym subskrybentem, ale anulował swoją subskrypcję, możesz użyć metody `canceled`:

```php
if ($user->subscription('default')->canceled()) {
    // ...
}
```

Możesz również określić, czy użytkownik anulował swoją subskrypcję, ale nadal jest w "okresie łaski", dopóki subskrypcja nie wygaśnie całkowicie. Na przykład, jeśli użytkownik anuluje subskrypcję 5 marca, która pierwotnie miała wygasnąć 10 marca, użytkownik jest w "okresie łaski" do 10 marca. Należy zauważyć, że metoda `subscribed` nadal zwraca `true` w tym czasie:

```php
if ($user->subscription('default')->onGracePeriod()) {
    // ...
}
```

Aby określić, czy użytkownik anulował swoją subskrypcję i nie jest już w "okresie łaski", możesz użyć metody `ended`:

```php
if ($user->subscription('default')->ended()) {
    // ...
}
```

<a name="incomplete-and-past-due-status"></a>
#### Status niekompletny i zaległy

Jeśli subskrypcja wymaga dodatkowej akcji płatności po utworzeniu, subskrypcja zostanie oznaczona jako `incomplete`. Statusy subskrypcji są przechowywane w kolumnie `stripe_status` tabeli bazy danych `subscriptions` w Cashier.

Podobnie, jeśli dodatkowa akcja płatności jest wymagana podczas zmiany cen, subskrypcja zostanie oznaczona jako `past_due`. Gdy Twoja subskrypcja jest w którymkolwiek z tych stanów, nie będzie aktywna, dopóki klient nie potwierdzi swojej płatności. Określenie, czy subskrypcja ma niekompletną płatność, można wykonać za pomocą metody `hasIncompletePayment` na modelu płatniczym lub instancji subskrypcji:

```php
if ($user->hasIncompletePayment('default')) {
    // ...
}

if ($user->subscription('default')->hasIncompletePayment()) {
    // ...
}
```

Gdy subskrypcja ma niekompletną płatność, powinieneś skierować użytkownika na stronę potwierdzenia płatności Cashier, przekazując identyfikator `latestPayment`. Możesz użyć metody `latestPayment` dostępnej w instancji subskrypcji, aby pobrać ten identyfikator:

```html
<a href="{{ route('cashier.payment', $subscription->latestPayment()->id) }}">
    Please confirm your payment.
</a>
```

Jeśli chcesz, aby subskrypcja była nadal uważana za aktywną, gdy jest w stanie `past_due` lub `incomplete`, możesz użyć metod `keepPastDueSubscriptionsActive` i `keepIncompleteSubscriptionsActive` dostarczonych przez Cashier. Zazwyczaj te metody powinny być wywoływane w metodzie `register` Twojego `App\Providers\AppServiceProvider`:

```php
use Laravel\Cashier\Cashier;

/**
 * Register any application services.
 */
public function register(): void
{
    Cashier::keepPastDueSubscriptionsActive();
    Cashier::keepIncompleteSubscriptionsActive();
}
```

> [!WARNING]
> Gdy subskrypcja jest w stanie `incomplete`, nie może być zmieniona do momentu potwierdzenia płatności. Dlatego metody `swap` i `updateQuantity` zgłoszą wyjątek, gdy subskrypcja jest w stanie `incomplete`.

<a name="subscription-scopes"></a>
#### Zakresy subskrypcji

Większość stanów subskrypcji jest również dostępna jako zakresy zapytań, dzięki czemu możesz łatwo przeszukiwać bazę danych w poszukiwaniu subskrypcji znajdujących się w danym stanie:

```php
// Get all active subscriptions...
$subscriptions = Subscription::query()->active()->get();

// Get all of the canceled subscriptions for a user...
$subscriptions = $user->subscriptions()->canceled()->get();
```

Pełna lista dostępnych zakresów jest dostępna poniżej:

```php
Subscription::query()->active();
Subscription::query()->canceled();
Subscription::query()->ended();
Subscription::query()->incomplete();
Subscription::query()->notCanceled();
Subscription::query()->notOnGracePeriod();
Subscription::query()->notOnTrial();
Subscription::query()->onGracePeriod();
Subscription::query()->onTrial();
Subscription::query()->pastDue();
Subscription::query()->recurring();
```

<a name="changing-prices"></a>
### Zmiana cen

Po zasubskrybowaniu aplikacji przez klienta, może on czasami chcieć zmienić na nową cenę subskrypcji. Aby zmienić klientowi cenę na nową, przekaż identyfikator ceny Stripe do metody `swap`. Podczas zmiany cen zakłada się, że użytkownik chce ponownie aktywować swoją subskrypcję, jeśli została wcześniej anulowana. Podany identyfikator ceny powinien odpowiadać identyfikatorowi ceny Stripe dostępnemu w dashboardzie Stripe:

```php
use App\Models\User;

$user = App\Models\User::find(1);

$user->subscription('default')->swap('price_yearly');
```

Jeśli klient jest w okresie próbnym, okres próbny zostanie zachowany. Dodatkowo, jeśli dla subskrypcji istnieje "ilość", ta ilość również zostanie zachowana.

Jeśli chcesz zmienić ceny i anulować dowolny okres próbny, w którym aktualnie znajduje się klient, możesz wywołać metodę `skipTrial`:

```php
$user->subscription('default')
    ->skipTrial()
    ->swap('price_yearly');
```

Jeśli chcesz zmienić ceny i natychmiast wystawić fakturę klientowi zamiast czekać na jego następny cykl rozliczeniowy, możesz użyć metody `swapAndInvoice`:

```php
$user = User::find(1);

$user->subscription('default')->swapAndInvoice('price_yearly');
```

<a name="prorations"></a>
#### Proporcjonalne opłaty

Domyślnie Stripe proporcjonalnie przelicza opłaty podczas zmiany między cenami. Metoda `noProrate` może być użyta do aktualizacji ceny subskrypcji bez proporcjonalnego przeliczania opłat:

```php
$user->subscription('default')->noProrate()->swap('price_yearly');
```

Więcej informacji na temat proporcjonalnych opłat za subskrypcję można znaleźć w [dokumentacji Stripe](https://stripe.com/docs/billing/subscriptions/prorations).

> [!WARNING]
> Wykonanie metody `noProrate` przed metodą `swapAndInvoice` nie będzie miało wpływu na proporcjonalne opłaty. Faktura zawsze zostanie wystawiona.

<a name="subscription-quantity"></a>
### Ilość subskrypcji

Czasami subskrypcje są zależne od "ilości". Na przykład aplikacja do zarządzania projektami może pobierać $10 miesięcznie za projekt. Możesz użyć metod `incrementQuantity` i `decrementQuantity`, aby łatwo zwiększyć lub zmniejszyć ilość subskrypcji:

```php
use App\Models\User;

$user = User::find(1);

$user->subscription('default')->incrementQuantity();

// Add five to the subscription's current quantity...
$user->subscription('default')->incrementQuantity(5);

$user->subscription('default')->decrementQuantity();

// Subtract five from the subscription's current quantity...
$user->subscription('default')->decrementQuantity(5);
```

Alternatywnie możesz ustawić konkretną ilość używając metody `updateQuantity`:

```php
$user->subscription('default')->updateQuantity(10);
```

Metoda `noProrate` może być użyta do aktualizacji ilości subskrypcji bez proporcjonalnego przeliczania opłat:

```php
$user->subscription('default')->noProrate()->updateQuantity(10);
```

Więcej informacji na temat ilości subskrypcji można znaleźć w [dokumentacji Stripe](https://stripe.com/docs/subscriptions/quantities).

<a name="quantities-for-subscription-with-multiple-products"></a>
#### Ilości dla subskrypcji z wieloma produktami

Jeśli Twoja subskrypcja jest [subskrypcją z wieloma produktami](#subscriptions-with-multiple-products), powinieneś przekazać ID ceny, której ilość chcesz zwiększyć lub zmniejszyć, jako drugi argument do metod increment / decrement:

```php
$user->subscription('default')->incrementQuantity(1, 'price_chat');
```

<a name="subscriptions-with-multiple-products"></a>
### Subskrypcje z wieloma produktami

[Subskrypcje z wieloma produktami](https://stripe.com/docs/billing/subscriptions/multiple-products) pozwalają przypisać wiele produktów rozliczeniowych do jednej subskrypcji. Na przykład wyobraź sobie, że budujesz aplikację obsługi klienta "helpdesk", która ma podstawową cenę subskrypcji $10 miesięcznie, ale oferuje dodatek czatu na żywo za dodatkowe $15 miesięcznie. Informacje o subskrypcjach z wieloma produktami są przechowywane w tabeli bazy danych `subscription_items` w Cashier.

Możesz określić wiele produktów dla danej subskrypcji, przekazując tablicę cen jako drugi argument do metody `newSubscription`:

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription('default', [
        'price_monthly',
        'price_chat',
    ])->create($request->paymentMethodId);

    // ...
});
```

W powyższym przykładzie klient będzie miał dwie ceny przypisane do swojej subskrypcji `default`. Obie ceny będą pobierane w ich odpowiednich interwałach rozliczeniowych. W razie potrzeby możesz użyć metody `quantity`, aby wskazać konkretną ilość dla każdej ceny:

```php
$user = User::find(1);

$user->newSubscription('default', ['price_monthly', 'price_chat'])
    ->quantity(5, 'price_chat')
    ->create($paymentMethod);
```

Jeśli chcesz dodać kolejną cenę do istniejącej subskrypcji, możesz wywołać metodę `addPrice` subskrypcji:

```php
$user = User::find(1);

$user->subscription('default')->addPrice('price_chat');
```

Powyższy przykład doda nową cenę, a klient zostanie za nią obciążony w następnym cyklu rozliczeniowym. Jeśli chcesz obciążyć klienta natychmiast, możesz użyć metody `addPriceAndInvoice`:

```php
$user->subscription('default')->addPriceAndInvoice('price_chat');
```

Jeśli chcesz dodać cenę z określoną ilością, możesz przekazać ilość jako drugi argument metod `addPrice` lub `addPriceAndInvoice`:

```php
$user = User::find(1);

$user->subscription('default')->addPrice('price_chat', 5);
```

Możesz usuwać ceny z subskrypcji używając metody `removePrice`:

```php
$user->subscription('default')->removePrice('price_chat');
```

> [!WARNING]
> Nie możesz usunąć ostatniej ceny w subskrypcji. Zamiast tego powinieneś po prostu anulować subskrypcję.

<a name="swapping-prices"></a>
#### Zamiana cen

Możesz również zmienić ceny przypisane do subskrypcji z wieloma produktami. Na przykład wyobraź sobie, że klient ma subskrypcję `price_basic` z dodatkiem `price_chat` i chcesz zaktualizować klienta z ceny `price_basic` na cenę `price_pro`:

```php
use App\Models\User;

$user = User::find(1);

$user->subscription('default')->swap(['price_pro', 'price_chat']);
```

Podczas wykonywania powyższego przykładu, podstawowy element subskrypcji z `price_basic` jest usuwany, a ten z `price_chat` jest zachowywany. Dodatkowo tworzony jest nowy element subskrypcji dla `price_pro`.

Możesz również określić opcje elementu subskrypcji, przekazując tablicę par klucz/wartość do metody `swap`. Na przykład, możesz potrzebować określić ilości dla cen subskrypcji:

```php
$user = User::find(1);

$user->subscription('default')->swap([
    'price_pro' => ['quantity' => 5],
    'price_chat'
]);
```

Jeśli chcesz zamienić pojedynczą cenę w subskrypcji, możesz to zrobić używając metody `swap` na samym elemencie subskrypcji. To podejście jest szczególnie przydatne, jeśli chcesz zachować wszystkie istniejące metadane dla innych cen subskrypcji:

```php
$user = User::find(1);

$user->subscription('default')
    ->findItemOrFail('price_basic')
    ->swap('price_pro');
```

<a name="proration"></a>
#### Proporcjonalność

Domyślnie Stripe proporcjonalnie rozlicza opłaty podczas dodawania lub usuwania cen z subskrypcji z wieloma produktami. Jeśli chcesz dokonać korekty ceny bez proporcjonalnego rozliczenia, powinieneś dołączyć metodę `noProrate` do operacji na cenie:

```php
$user->subscription('default')->noProrate()->removePrice('price_chat');
```

<a name="swapping-quantities"></a>
#### Ilości

Jeśli chcesz zaktualizować ilości dla poszczególnych cen subskrypcji, możesz to zrobić używając [istniejących metod ilości](#subscription-quantity), przekazując ID ceny jako dodatkowy argument do metody:

```php
$user = User::find(1);

$user->subscription('default')->incrementQuantity(5, 'price_chat');

$user->subscription('default')->decrementQuantity(3, 'price_chat');

$user->subscription('default')->updateQuantity(10, 'price_chat');
```

> [!WARNING]
> Gdy subskrypcja ma wiele cen, atrybuty `stripe_price` i `quantity` w modelu `Subscription` będą `null`. Aby uzyskać dostęp do atrybutów poszczególnych cen, należy użyć relacji `items` dostępnej w modelu `Subscription`.

<a name="subscription-items"></a>
#### Elementy Subskrypcji

When a subscription has multiple prices, it will have multiple subscription "items" stored in your database's `subscription_items` table. You may access these via the `items` relationship on the subscription:

```php
use App\Models\User;

$user = User::find(1);

$subscriptionItem = $user->subscription('default')->items->first();

// Retrieve the Stripe price and quantity for a specific item...
$stripePrice = $subscriptionItem->stripe_price;
$quantity = $subscriptionItem->quantity;
```

Możesz również pobrać określoną cenę używając metody `findItemOrFail`:

```php
$user = User::find(1);

$subscriptionItem = $user->subscription('default')->findItemOrFail('price_chat');
```

<a name="multiple-subscriptions"></a>
### Wiele Subskrypcji

Stripe pozwala Twoim klientom posiadać wiele subskrypcji jednocześnie. Na przykład, możesz prowadzić siłownię, która oferuje subskrypcję pływania i subskrypcję ćwiczeń siłowych, a każda subskrypcja może mieć różne ceny. Oczywiście klienci powinni mieć możliwość subskrybowania jednego lub obu planów.

Kiedy Twoja aplikacja tworzy subskrypcje, możesz podać typ subskrypcji do metody `newSubscription`. Typ może być dowolnym ciągiem znaków reprezentującym typ subskrypcji, którą użytkownik rozpoczyna:

```php
use Illuminate\Http\Request;

Route::post('/swimming/subscribe', function (Request $request) {
    $request->user()->newSubscription('swimming')
        ->price('price_swimming_monthly')
        ->create($request->paymentMethodId);

    // ...
});
```

W tym przykładzie zainicjowaliśmy miesięczną subskrypcję pływania dla klienta. Jednak mogą oni chcieć przejść na subskrypcję roczną w późniejszym czasie. Podczas dostosowywania subskrypcji klienta, możemy po prostu zamienić cenę w subskrypcji `swimming`:

```php
$user->subscription('swimming')->swap('price_swimming_yearly');
```

Oczywiście możesz również całkowicie anulować subskrypcję:

```php
$user->subscription('swimming')->cancel();
```

<a name="usage-based-billing"></a>
### Rozliczenia Oparte na Użyciu

[Rozliczenia oparte na użyciu](https://stripe.com/docs/billing/subscriptions/metered-billing) pozwalają na obciążanie klientów na podstawie ich użycia produktu podczas cyklu rozliczeniowego. Na przykład, możesz obciążać klientów na podstawie liczby wiadomości tekstowych lub e-maili, które wysyłają miesięcznie.

To start using usage billing, you will first need to create a new product in your Stripe dashboard with a [usage based billing model](https://docs.stripe.com/billing/subscriptions/usage-based/implementation-guide) and a [meter](https://docs.stripe.com/billing/subscriptions/usage-based/recording-usage#configure-meter). After creating the meter, store the associated event name and meter ID, which you will need to report and retrieve usage. Then, use the `meteredPrice` method to add the metered price ID to a customer subscription:

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription('default')
        ->meteredPrice('price_metered')
        ->create($request->paymentMethodId);

    // ...
});
```

Możesz również rozpocząć mierzoną subskrypcję przez [Stripe Checkout](#checkout):

```php
$checkout = Auth::user()
    ->newSubscription('default', [])
    ->meteredPrice('price_metered')
    ->checkout();

return view('your-checkout-view', [
    'checkout' => $checkout,
]);
```

<a name="reporting-usage"></a>
#### Raportowanie Użycia

W miarę jak Twój klient korzysta z Twojej aplikacji, będziesz raportować jego użycie do Stripe, aby mógł być rozliczany dokładnie. Aby zgłosić użycie mierzonego zdarzenia, możesz użyć metody `reportMeterEvent` na swoim modelu `Billable`:

```php
$user = User::find(1);

$user->reportMeterEvent('emails-sent');
```

Domyślnie "ilość użycia" wynosząca 1 jest dodawana do okresu rozliczeniowego. Alternatywnie możesz przekazać określoną ilość "użycia", która zostanie dodana do użycia klienta w okresie rozliczeniowym:

```php
$user = User::find(1);

$user->reportMeterEvent('emails-sent', quantity: 15);
```

Aby pobrać podsumowanie zdarzeń klienta dla licznika, możesz użyć metody `meterEventSummaries` instancji `Billable`:

```php
$user = User::find(1);

$meterUsage = $user->meterEventSummaries($meterId);

$meterUsage->first()->aggregated_value // 10
```

Zapoznaj się z [dokumentacją obiektu podsumowania zdarzeń licznika Stripe](https://docs.stripe.com/api/billing/meter-event_summary/object), aby uzyskać więcej informacji o podsumowaniach zdarzeń licznika.

Aby [wyświetlić wszystkie liczniki](https://docs.stripe.com/api/billing/meter/list), możesz użyć metody `meters` instancji `Billable`:

```php
$user = User::find(1);

$user->meters();
```

<a name="subscription-taxes"></a>
### Podatki od Subskrypcji

> [!WARNING]
> Zamiast ręcznego obliczania stawek podatkowych, możesz [automatycznie obliczać podatki używając Stripe Tax](#tax-configuration)

Aby określić stawki podatkowe, które użytkownik płaci za subskrypcję, powinieneś zaimplementować metodę `taxRates` w swoim modelu rozliczeniowym i zwrócić tablicę zawierającą ID stawek podatkowych Stripe. Możesz zdefiniować te stawki podatkowe w [swoim panelu Stripe](https://dashboard.stripe.com/test/tax-rates):

```php
/**
 * The tax rates that should apply to the customer's subscriptions.
 *
 * @return array<int, string>
 */
public function taxRates(): array
{
    return ['txr_id'];
}
```

Metoda `taxRates` umożliwia zastosowanie stawki podatkowej dla każdego klienta osobno, co może być pomocne dla bazy użytkowników obejmującej wiele krajów i stawek podatkowych.

If you're offering subscriptions with multiple products, you may define different tax rates for each price by implementing a `priceTaxRates` method on your billable model:

```php
/**
 * The tax rates that should apply to the customer's subscriptions.
 *
 * @return array<string, array<int, string>>
 */
public function priceTaxRates(): array
{
    return [
        'price_monthly' => ['txr_id'],
    ];
}
```

> [!WARNING]
> Metoda `taxRates` dotyczy tylko opłat subskrypcyjnych. Jeśli używasz Cashier do dokonywania jednorazowych opłat, będziesz musiał ręcznie określić stawkę podatkową w tym czasie.

<a name="syncing-tax-rates"></a>
#### Synchronizacja Stawek Podatkowych

Podczas zmiany zakodowanych na stałe ID stawek podatkowych zwracanych przez metodę `taxRates`, ustawienia podatkowe dla wszystkich istniejących subskrypcji użytkownika pozostaną takie same. Jeśli chcesz zaktualizować wartość podatku dla istniejących subskrypcji nowymi wartościami `taxRates`, powinieneś wywołać metodę `syncTaxRates` na instancji subskrypcji użytkownika:

```php
$user->subscription('default')->syncTaxRates();
```

Spowoduje to również synchronizację wszystkich stawek podatkowych dla elementów subskrypcji z wieloma produktami. Jeśli Twoja aplikacja oferuje subskrypcje z wieloma produktami, powinieneś upewnić się, że Twój model rozliczeniowy implementuje metodę `priceTaxRates` [omówioną powyżej](#subscription-taxes).

<a name="tax-exemption"></a>
#### Zwolnienie Podatkowe

Cashier oferuje również metody `isNotTaxExempt`, `isTaxExempt` i `reverseChargeApplies` do określenia, czy klient jest zwolniony z podatku. Te metody wywołują API Stripe, aby określić status zwolnienia podatkowego klienta:

```php
use App\Models\User;

$user = User::find(1);

$user->isTaxExempt();
$user->isNotTaxExempt();
$user->reverseChargeApplies();
```

> [!WARNING]
> Te metody są również dostępne dla każdego obiektu `Laravel\Cashier\Invoice`. Jednak gdy są wywoływane na obiekcie `Invoice`, metody określą status zwolnienia w momencie utworzenia faktury.

<a name="subscription-anchor-date"></a>
### Data Zakotwiczenia Subskrypcji

By default, the billing cycle anchor is the date the subscription was created or, if a trial period is used, the date that the trial ends. If you would like to modify the billing anchor date, you may use the `anchorBillingCycleOn` method:

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $anchor = Carbon::parse('first day of next month');

    $request->user()->newSubscription('default', 'price_monthly')
        ->anchorBillingCycleOn($anchor->startOfDay())
        ->create($request->paymentMethodId);

    // ...
});
```

Aby uzyskać więcej informacji na temat zarządzania cyklami rozliczeniowymi subskrypcji, zapoznaj się z [dokumentacją cyklu rozliczeniowego Stripe](https://stripe.com/docs/billing/subscriptions/billing-cycle)

<a name="cancelling-subscriptions"></a>
### Anulowanie Subskrypcji

To cancel a subscription, call the `cancel` method on the user's subscription:

```php
$user->subscription('default')->cancel();
```

Gdy subskrypcja jest anulowana, Cashier automatycznie ustawi kolumnę `ends_at` w tabeli bazy danych `subscriptions`. Ta kolumna jest używana do określenia, kiedy metoda `subscribed` powinna zacząć zwracać `false`.

Na przykład, jeśli klient anuluje subskrypcję 1 marca, ale subskrypcja nie była zaplanowana do zakończenia do 5 marca, metoda `subscribed` będzie nadal zwracać `true` do 5 marca. Dzieje się tak, ponieważ użytkownik zazwyczaj może kontynuować korzystanie z aplikacji do końca swojego cyklu rozliczeniowego.

Możesz określić, czy użytkownik anulował swoją subskrypcję, ale nadal jest w "okresie karencji", używając metody `onGracePeriod`:

```php
if ($user->subscription('default')->onGracePeriod()) {
    // ...
}
```

Jeśli chcesz natychmiast anulować subskrypcję, wywołaj metodę `cancelNow` na subskrypcji użytkownika:

```php
$user->subscription('default')->cancelNow();
```

Jeśli chcesz natychmiast anulować subskrypcję i zafakturować wszelkie pozostałe niezafakturowane mierzone użycie lub nowe / oczekujące pozycje faktury proporcjonalnej, wywołaj metodę `cancelNowAndInvoice` na subskrypcji użytkownika:

```php
$user->subscription('default')->cancelNowAndInvoice();
```

Możesz również wybrać anulowanie subskrypcji w określonym momencie:

```php
$user->subscription('default')->cancelAt(
    now()->plus(days: 10)
);
```

Na koniec, zawsze powinieneś anulować subskrypcje użytkownika przed usunięciem powiązanego modelu użytkownika:

```php
$user->subscription('default')->cancelNow();

$user->delete();
```

<a name="resuming-subscriptions"></a>
### Wznowienie Subskrypcji

Jeśli klient anulował swoją subskrypcję i chcesz ją wznowić, możesz wywołać metodę `resume` na subskrypcji. Klient musi nadal znajdować się w "okresie karencji", aby móc wznowić subskrypcję:

```php
$user->subscription('default')->resume();
```

Jeśli klient anuluje subskrypcję, a następnie wznowi tę subskrypcję zanim subskrypcja całkowicie wygaśnie, klient nie zostanie natychmiast obciążony. Zamiast tego jego subskrypcja zostanie reaktywowana i zostanie obciążony w oryginalnym cyklu rozliczeniowym.

<a name="subscription-trials"></a>
## Okresy Próbne Subskrypcji

<a name="with-payment-method-up-front"></a>
### Z Metodą Płatności z Góry

Jeśli chcesz oferować okresy próbne swoim klientom, jednocześnie zbierając informacje o metodzie płatności z góry, powinieneś użyć metody `trialDays` podczas tworzenia subskrypcji:

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription('default', 'price_monthly')
        ->trialDays(10)
        ->create($request->paymentMethodId);

    // ...
});
```

Ta metoda ustawi datę zakończenia okresu próbnego w rekordzie subskrypcji w bazie danych i poinstruuje Stripe, aby nie rozpoczynać rozliczania klienta aż do tej daty. Podczas używania metody `trialDays`, Cashier nadpisze każdy domyślny okres próbny skonfigurowany dla ceny w Stripe.

> [!WARNING]
> Jeśli subskrypcja klienta nie zostanie anulowana przed datą zakończenia okresu próbnego, zostanie obciążony zaraz po wygaśnięciu okresu próbnego, więc powinieneś upewnić się, że powiadomisz użytkowników o dacie zakończenia okresu próbnego.

Metoda `trialUntil` pozwala na podanie instancji `DateTime`, która określa, kiedy okres próbny powinien się zakończyć:

```php
use Illuminate\Support\Carbon;

$user->newSubscription('default', 'price_monthly')
    ->trialUntil(Carbon::now()->plus(days: 10))
    ->create($paymentMethod);
```

Możesz określić, czy użytkownik znajduje się w okresie próbnym, używając metody `onTrial` instancji użytkownika lub metody `onTrial` instancji subskrypcji. Poniższe dwa przykłady są równoważne:

```php
if ($user->onTrial('default')) {
    // ...
}

if ($user->subscription('default')->onTrial()) {
    // ...
}
```

Możesz użyć metody `endTrial`, aby natychmiast zakończyć okres próbny subskrypcji:

```php
$user->subscription('default')->endTrial();
```

Aby określić, czy istniejący okres próbny wygasł, możesz użyć metod `hasExpiredTrial`:

```php
if ($user->hasExpiredTrial('default')) {
    // ...
}

if ($user->subscription('default')->hasExpiredTrial()) {
    // ...
}
```

<a name="defining-trial-days-in-stripe-cashier"></a>
#### Definiowanie Dni Próbnych w Stripe / Cashier

Możesz zdecydować, ile dni próbnych otrzymają Twoje ceny w panelu Stripe lub zawsze przekazywać je jawnie za pomocą Cashier. Jeśli zdecydujesz się zdefiniować dni próbne swojej ceny w Stripe, powinieneś być świadomy, że nowe subskrypcje, w tym nowe subskrypcje dla klienta, który miał subskrypcję w przeszłości, zawsze otrzymają okres próbny, chyba że jawnie wywołasz metodę `skipTrial()`.

<a name="without-payment-method-up-front"></a>
### Bez metody płatności z góry

Jeśli chcesz oferować okresy próbne bez zbierania informacji o metodzie płatności użytkownika z góry, możesz ustawić kolumnę `trial_ends_at` w rekordzie użytkownika na żądaną datę zakończenia okresu próbnego. Zazwyczaj robi się to podczas rejestracji użytkownika:

```php
use App\Models\User;

$user = User::create([
    // ...
    'trial_ends_at' => now()->plus(days: 10),
]);
```

> [!WARNING]
> Upewnij się, że dodałeś [rzutowanie daty](/docs/{{version}}/eloquent-mutators#date-casting) dla atrybutu `trial_ends_at` w definicji klasy modelu rozliczeniowego.

Cashier określa ten typ okresu próbnego jako "ogólny okres próbny", ponieważ nie jest on powiązany z żadną istniejącą subskrypcją. Metoda `onTrial` na instancji modelu rozliczeniowego zwróci `true`, jeśli bieżąca data nie przekroczyła wartości `trial_ends_at`:

```php
if ($user->onTrial()) {
    // Użytkownik jest w okresie próbnym...
}
```

Gdy będziesz gotowy do utworzenia rzeczywistej subskrypcji dla użytkownika, możesz użyć metody `newSubscription` jak zwykle:

```php
$user = User::find(1);

$user->newSubscription('default', 'price_monthly')->create($paymentMethod);
```

Aby pobrać datę zakończenia okresu próbnego użytkownika, możesz użyć metody `trialEndsAt`. Ta metoda zwróci instancję daty Carbon, jeśli użytkownik jest w okresie próbnym, lub `null`, jeśli nie jest. Możesz również przekazać opcjonalny parametr typu subskrypcji, jeśli chcesz uzyskać datę zakończenia okresu próbnego dla konkretnej subskrypcji innej niż domyślna:

```php
if ($user->onTrial()) {
    $trialEndsAt = $user->trialEndsAt('main');
}
```

Możesz również użyć metody `onGenericTrial`, jeśli chcesz wiedzieć konkretnie, że użytkownik jest w swoim "ogólnym" okresie próbnym i nie utworzył jeszcze rzeczywistej subskrypcji:

```php
if ($user->onGenericTrial()) {
    // Użytkownik jest w swoim "ogólnym" okresie próbnym...
}
```

<a name="extending-trials"></a>
### Przedłużanie okresów próbnych

Metoda `extendTrial` pozwala przedłużyć okres próbny subskrypcji po utworzeniu subskrypcji. Jeśli okres próbny już wygasł, a klient jest już obciążany za subskrypcję, nadal możesz zaoferować mu przedłużony okres próbny. Czas spędzony w okresie próbnym zostanie odjęty od następnej faktury klienta:

```php
use App\Models\User;

$subscription = User::find(1)->subscription('default');

// Zakończ okres próbny za 7 dni...
$subscription->extendTrial(
    now()->plus(days: 7)
);

// Dodaj dodatkowe 5 dni do okresu próbnego...
$subscription->extendTrial(
    $subscription->trial_ends_at->plus(days: 5)
);
```

<a name="handling-stripe-webhooks"></a>
## Obsługa webhooków Stripe

> [!NOTE]
> Możesz użyć [Stripe CLI](https://stripe.com/docs/stripe-cli), aby pomóc testować webhooki podczas lokalnego rozwoju.

Stripe może powiadamiać aplikację o różnych zdarzeniach za pośrednictwem webhooków. Domyślnie trasa wskazująca na kontroler webhooków Cashier jest automatycznie rejestrowana przez dostawcę usług Cashier. Ten kontroler obsłuży wszystkie przychodzące żądania webhooków.

Domyślnie kontroler webhooków Cashier automatycznie obsłuży anulowanie subskrypcji, które mają zbyt wiele nieudanych opłat (zgodnie z ustawieniami Stripe), aktualizacje klientów, usuwanie klientów, aktualizacje subskrypcji i zmiany metod płatności; jednak, jak wkrótce odkryjemy, możesz rozszerzyć ten kontroler, aby obsłużyć dowolne zdarzenie webhooków Stripe, które lubisz.

Aby upewnić się, że aplikacja może obsługiwać webhooki Stripe, skonfiguruj URL webhooka w panelu kontrolnym Stripe. Domyślnie kontroler webhooków Cashier odpowiada na ścieżkę URL `/stripe/webhook`. Pełna lista wszystkich webhooków, które powinieś włączyć w panelu kontrolnym Stripe to:

- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `customer.updated`
- `customer.deleted`
- `payment_method.automatically_updated`
- `invoice.payment_action_required`
- `invoice.payment_succeeded`

Dla wygody Cashier zawiera polecenie Artisan `cashier:webhook`. To polecenie utworzy webhook w Stripe, który nasłuchuje wszystkich zdarzeń wymaganych przez Cashier:

```shell
php artisan cashier:webhook
```

Domyślnie utworzony webhook wskazuje na URL zdefiniowany przez zmienną środowiskową `APP_URL` i trasę `cashier.webhook`, która jest dołączona do Cashier. Możesz podać opcję `--url` podczas wywoływania polecenia, jeśli chcesz użyć innego URL:

```shell
php artisan cashier:webhook --url "https://example.com/stripe/webhook"
```

Utworzony webhook będzie używać wersji API Stripe, która jest kompatybilna z Twoją wersją Cashier. Jeśli chcesz użyć innej wersji Stripe, możesz podać opcję `--api-version`:

```shell
php artisan cashier:webhook --api-version="2019-12-03"
```

Po utworzeniu webhook będzie natychmiast aktywny. Jeśli chcesz utworzyć webhook, ale mieć go wyłączonego do czasu gotowości, możesz podać opcję `--disabled` podczas wywoływania polecenia:

```shell
php artisan cashier:webhook --disabled
```

> [!WARNING]
> Upewnij się, że chronisz przychodzące żądania webhooków Stripe za pomocą dołączonego middleware [weryfikacji podpisu webhooków](#verifying-webhook-signatures) Cashier.

<a name="webhooks-csrf-protection"></a>
#### Webhooki a ochrona CSRF

Ponieważ webhooki Stripe muszą ominąć [ochronę CSRF](/docs/{{version}}/csrf) Laravel, powinieś upewnić się, że Laravel nie próbuje weryfikować tokenu CSRF dla przychodzących webhooków Stripe. Aby to osiągnąć, powinieś wykluczyć `stripe/*` z ochrony CSRF w pliku `bootstrap/app.php` aplikacji:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->validateCsrfTokens(except: [
        'stripe/*',
    ]);
})
```

<a name="defining-webhook-event-handlers"></a>
### Definiowanie procedur obsługi zdarzeń webhook

Cashier automatycznie obsługuje anulowania subskrypcji dla nieudanych opłat i innych typowych zdarzeń webhooków Stripe. Jednak jeśli masz dodatkowe zdarzenia webhooków, które chciałbyś obsłużyć, możesz to zrobić, nasłuchując następujących zdarzeń wysyłanych przez Cashier:

- `Laravel\Cashier\Events\WebhookReceived`
- `Laravel\Cashier\Events\WebhookHandled`

Oba zdarzenia zawierają pełny payload webhooka Stripe. Na przykład, jeśli chcesz obsłużyć webhook `invoice.payment_succeeded`, możesz zarejestrować [nasłuchiwacza](/docs/{{version}}/events#defining-listeners), który obsłuży zdarzenie:

```php
<?php

namespace App\Listeners;

use Laravel\Cashier\Events\WebhookReceived;

class StripeEventListener
{
    /**
     * Obsługuj otrzymane webhooki Stripe.
     */
    public function handle(WebhookReceived $event): void
    {
        if ($event->payload['type'] === 'invoice.payment_succeeded') {
            // Obsłuż przychodzące zdarzenie...
        }
    }
}
```

<a name="verifying-webhook-signatures"></a>
### Weryfikacja podpisów webhook

Aby zabezpieczyć webhooki, możesz użyć [podpisów webhooków Stripe](https://stripe.com/docs/webhooks/signatures). Dla wygody Cashier automatycznie zawiera middleware, które weryfikuje, czy przychodzące żądanie webhooka Stripe jest prawidłowe.

Aby włączyć weryfikację webhooków, upewnij się, że zmienna środowiskowa `STRIPE_WEBHOOK_SECRET` jest ustawiona w pliku `.env` aplikacji. Sekret webhooka można pobrać z panelu konta Stripe.

<a name="single-charges"></a>
## Pojedyncze opłaty

<a name="simple-charge"></a>
### Prosta opłata

Jeśli chcesz dokonać jednorazowej opłaty wobec klienta, możesz użyć metody `charge` na instancji modelu rozliczeniowego. Będziesz musiał [podać identyfikator metody płatności](#payment-methods-for-single-charges) jako drugi argument do metody `charge`:

```php
use Illuminate\Http\Request;

Route::post('/purchase', function (Request $request) {
    $stripeCharge = $request->user()->charge(
        100, $request->paymentMethodId
    );

    // ...
});
```

Metoda `charge` przyjmuje tablicę jako trzeci argument, pozwalając przekazać dowolne opcje do podstawowego tworzenia opłaty Stripe. Więcej informacji na temat opcji dostępnych podczas tworzenia opłat można znaleźć w [dokumentacji Stripe](https://stripe.com/docs/api/charges/create):

```php
$user->charge(100, $paymentMethod, [
    'custom_option' => $value,
]);
```

Możesz również użyć metody `charge` bez podstawowego klienta lub użytkownika. Aby to osiągnąć, wywołaj metodę `charge` na nowej instancji modelu rozliczeniowego aplikacji:

```php
use App\Models\User;

$stripeCharge = (new User)->charge(100, $paymentMethod);
```

Metoda `charge` zgłosi wyjątek, jeśli opłata się nie powiedzie. Jeśli opłata zaksięgowana zostanie pomyślnie, instancja `Laravel\Cashier\Payment` zostanie zwrócona z metody:

```php
try {
    $payment = $user->charge(100, $paymentMethod);
} catch (Exception $e) {
    // ...
}
```

> [!WARNING]
> Metoda `charge` przyjmuje kwotę płatności w najniższej jednostce waluty używanej przez aplikację. Na przykład, jeśli klienci płacą w dolarach amerykańskich, kwoty powinny być określone w groszach.

<a name="charge-with-invoice"></a>
### Opłata z fakturą

Czasami możesz potrzebować dokonać jednorazowej opłaty i zaoferować klientowi fakturę PDF. Metoda `invoicePrice` pozwala na to. Na przykład, zafakturujmy klientowi pięć nowych koszulek:

```php
$user->invoicePrice('price_tshirt', 5);
```

Faktura zostanie natychmiast obciążona domyślną metodą płatności użytkownika. Metoda `invoicePrice` przyjmuje również tablicę jako trzeci argument. Ta tablica zawiera opcje rozliczeniowe dla pozycji faktury. Czwarty argument akceptowany przez metodę to również tablica, która powinna zawierać opcje rozliczeniowe dla samej faktury:

```php
$user->invoicePrice('price_tshirt', 5, [
    'discounts' => [
        ['coupon' => 'SUMMER21SALE']
    ],
], [
    'default_tax_rates' => ['txr_id'],
]);
```

Podobnie do `invoicePrice`, możesz użyć metody `tabPrice`, aby utworzyć jednorazową opłatę dla wielu przedmiotów (do 250 pozycji na fakturę), dodając je do "rachunku" klienta, a następnie fakturując klienta. Na przykład możemy zafakturować klientowi pięć koszulek i dwa kubki:

```php
$user->tabPrice('price_tshirt', 5);
$user->tabPrice('price_mug', 2);
$user->invoice();
```

Alternatywnie możesz użyć metody `invoiceFor`, aby dokonać "jednorazowej" opłaty wobec domyślnej metody płatności klienta:

```php
$user->invoiceFor('Jednorazowa opłata', 500);
```

Chociaż metoda `invoiceFor` jest dostępna do użytku, zaleca się używanie metod `invoicePrice` i `tabPrice` ze wstępnie zdefiniowanymi cenami. Dzięki temu będziesz mieć dostęp do lepszych analiz i danych w panelu Stripe dotyczących sprzedaży w przeliczeniu na produkt.

> [!WARNING]
> Metody `invoice`, `invoicePrice` i `invoiceFor` utworzą fakturę Stripe, która będzie ponawiać nieudane próby rozliczenia. Jeśli nie chcesz, aby faktury ponawiały nieudane opłaty, będziesz musiał je zamknąć za pomocą API Stripe po pierwszej nieudanej opłacie.

<a name="creating-payment-intents"></a>
### Tworzenie intencji płatności

Możesz utworzyć nową intencję płatności Stripe, wywołując metodę `pay` na instancji modelu rozliczeniowego. Wywołanie tej metody utworzy intencję płatności, która jest opakowana w instancję `Laravel\Cashier\Payment`:

```php
use Illuminate\Http\Request;

Route::post('/pay', function (Request $request) {
    $payment = $request->user()->pay(
        $request->get('amount')
    );

    return $payment->client_secret;
});
```

Po utworzeniu intencji płatności możesz zwrócić sekret klienta do frontendu aplikacji, aby użytkownik mógł zakończyć płatność w przeglądarce. Aby dowiedzieć się więcej o budowaniu całych przepływów płatności przy użyciu intencji płatności Stripe, zapoznaj się z [dokumentacją Stripe](https://stripe.com/docs/payments/accept-a-payment?platform=web).

Podczas używania metody `pay` domyślne metody płatności, które są włączone w panelu Stripe, będą dostępne dla klienta. Alternatywnie, jeśli chcesz zezwolić tylko na określone metody płatności, możesz użyć metody `payWith`:

```php
use Illuminate\Http\Request;

Route::post('/pay', function (Request $request) {
    $payment = $request->user()->payWith(
        $request->get('amount'), ['card', 'bancontact']
    );

    return $payment->client_secret;
});
```

> [!WARNING]
> Metody `pay` i `payWith` przyjmują kwotę płatności w najniższej jednostce waluty używanej przez aplikację. Na przykład, jeśli klienci płacą w dolarach amerykańskich, kwoty powinny być określone w groszach.

<a name="refunding-charges"></a>
### Zwroty opłat

Jeśli musisz zwrócić opłatę Stripe, możesz użyć metody `refund`. Ta metoda przyjmuje [identyfikator intencji płatności](#payment-methods-for-single-charges) Stripe jako pierwszy argument:

```php
$payment = $user->charge(100, $paymentMethodId);

$user->refund($payment->id);
```

<a name="invoices"></a>
## Faktury

<a name="retrieving-invoices"></a>
### Pobieranie faktur

Możesz łatwo pobrać tablicę faktur modelu rozliczeniowego za pomocą metody `invoices`. Metoda `invoices` zwraca kolekcję instancji `Laravel\Cashier\Invoice`:

```php
$invoices = $user->invoices();
```

Jeśli chcesz uwzględnić oczekujące faktury w wynikach, możesz użyć metody `invoicesIncludingPending`:

```php
$invoices = $user->invoicesIncludingPending();
```

Możesz użyć metody `findInvoice`, aby pobrać konkretną fakturę po jej ID:

```php
$invoice = $user->findInvoice($invoiceId);
```

<a name="displaying-invoice-information"></a>
#### Wyświetlanie informacji o fakturze

Podczas wypisywania faktur dla klienta możesz użyć metod faktury do wyświetlania odpowiednich informacji o fakturze. Na przykład możesz chcieć wymienić każdą fakturę w tabeli, pozwalając użytkownikowi łatwo pobrać dowolny z nich:

```blade
<table>
    @foreach ($invoices as $invoice)
        <tr>
            <td>{{ $invoice->date()->toFormattedDateString() }}</td>
            <td>{{ $invoice->total() }}</td>
            <td><a href="/user/invoice/{{ $invoice->id }}">Pobierz</a></td>
        </tr>
    @endforeach
</table>
```

<a name="upcoming-invoices"></a>
### Nadchodzące faktury

Aby pobrać nadchodzącą fakturę dla klienta, możesz użyć metody `upcomingInvoice`:

```php
$invoice = $user->upcomingInvoice();
```

Podobnie, jeśli klient ma wiele subskrypcji, możesz również pobrać nadchodzącą fakturę dla konkretnej subskrypcji:

```php
$invoice = $user->subscription('default')->upcomingInvoice();
```

<a name="previewing-subscription-invoices"></a>
### Podgląd faktur subskrypcji

Używając metody `previewInvoice`, możesz podejrzeć fakturę przed dokonaniem zmian cen. Pozwoli to określić, jak będzie wyglądała faktura klienta, gdy zostanie dokonana dana zmiana ceny:

```php
$invoice = $user->subscription('default')->previewInvoice('price_yearly');
```

Możesz przekazać tablicę cen do metody `previewInvoice`, aby podejrzeć faktury z wieloma nowymi cenami:

```php
$invoice = $user->subscription('default')->previewInvoice(['price_yearly', 'price_metered']);
```

<a name="generating-invoice-pdfs"></a>
### Generowanie plików PDF faktur

Przed generowaniem plików PDF faktur powinieś użyć Composer do zainstalowania biblioteki Dompdf, która jest domyślnym rendererem faktur dla Cashier:

```shell
composer require dompdf/dompdf
```

Z poziomu trasy lub kontrolera możesz użyć metody `downloadInvoice`, aby wygenerować plik PDF danej faktury do pobrania. Ta metoda automatycznie wygeneruje odpowiednią odpowiedź HTTP potrzebną do pobrania faktury:

```php
use Illuminate\Http\Request;

Route::get('/user/invoice/{invoice}', function (Request $request, string $invoiceId) {
    return $request->user()->downloadInvoice($invoiceId);
});
```

Domyślnie wszystkie dane na fakturze pochodzą z danych klienta i faktury przechowywanych w Stripe. Nazwa pliku opiera się na wartości konfiguracji `app.name`. Możesz jednak dostosować część tych danych, podając tablicę jako drugi argument metody `downloadInvoice`. Ta tablica pozwala dostosować informacje, takie jak szczegóły firmy i produktu:

```php
return $request->user()->downloadInvoice($invoiceId, [
    'vendor' => 'Twoja Firma',
    'product' => 'Twój Produkt',
    'street' => 'Główna 1',
    'location' => '00-001 Warszawa, Polska',
    'phone' => '+48 123 456 789',
    'email' => 'info@example.com',
    'url' => 'https://example.com',
    'vendorVat' => 'PL1234567890',
]);
```

Metoda `downloadInvoice` również pozwala na własną nazwę pliku za pomocą trzeciego argumentu. Ta nazwa pliku zostanie automatycznie opatrzona sufiksem `.pdf`:

```php
return $request->user()->downloadInvoice($invoiceId, [], 'moja-faktura');
```

<a name="custom-invoice-render"></a>
#### Niestandardowy renderer faktur

Cashier umożliwia również używanie niestandardowego renderera faktur. Domyślnie Cashier używa implementacji `DompdfInvoiceRenderer`, która wykorzystuje bibliotekę PHP [dompdf](https://github.com/dompdf/dompdf) do generowania faktur Cashier. Możesz jednak użyć dowolnego renderera, implementując interfejs `Laravel\Cashier\Contracts\InvoiceRenderer`. Na przykład możesz chcieć renderować fakturę PDF za pomocą wywołania API do zewnętrznej usługi renderowania PDF:

```php
use Illuminate\Support\Facades\Http;
use Laravel\Cashier\Contracts\InvoiceRenderer;
use Laravel\Cashier\Invoice;

class ApiInvoiceRenderer implements InvoiceRenderer
{
    /**
     * Renderuj daną fakturę i zwróć surowe bajty PDF.
     */
    public function render(Invoice $invoice, array $data = [], array $options = []): string
    {
        $html = $invoice->view($data)->render();

        return Http::get('https://example.com/html-to-pdf', ['html' => $html])->get()->body();
    }
}
```

Po zaimplementowaniu kontraktu renderera faktur powinieś zaktualizować wartość konfiguracji `cashier.invoices.renderer` w pliku konfiguracyjnym `config/cashier.php` aplikacji. Ta wartość konfiguracyjna powinna być ustawiona na nazwę klasy niestandardowej implementacji renderera.

<a name="checkout"></a>
## Kasa

Cashier Stripe zapewnia również obsługę [Stripe Checkout](https://stripe.com/payments/checkout). Stripe Checkout eliminuje ból implementowania niestandardowych stron do akceptowania płatności, dostarczając wstępnie zbudowaną, hostowaną stronę płatności.

Następująca dokumentacja zawiera informacje o tym, jak zacząć używać Stripe Checkout z Cashier. Aby dowiedzieć się więcej o Stripe Checkout, powinieś również zapoznać się z [własną dokumentacją Stripe dotyczącą Checkout](https://stripe.com/docs/payments/checkout).

<a name="product-checkouts"></a>
### Kasy produktów

Możesz przeprowadzić kasę dla istniejącego produktu, który został utworzony w panelu Stripe, używając metody `checkout` na modelu rozliczeniowym. Metoda `checkout` rozpocznie nową sesję Stripe Checkout. Domyślnie wymagane jest przekazanie identyfikatora ceny Stripe:

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout('price_tshirt');
});
```

W razie potrzeby możesz również określić ilość produktu:

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout(['price_tshirt' => 15]);
});
```

Kiedy klient odwiedzi tę trasę, zostanie przekierowany na stronę Checkout Stripe. Domyślnie, gdy użytkownik pomyślnie zakończy lub anuluje zakup, zostanie przekierowany do lokalizacji trasy `home`, ale możesz określić niestandardowe adresy URL callback, używając opcji `success_url` i `cancel_url`:

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout(['price_tshirt' => 1], [
        'success_url' => route('your-success-route'),
        'cancel_url' => route('your-cancel-route'),
    ]);
});
```

Podczas definiowania opcji kasy `success_url` możesz polecić Stripe dodanie identyfikatora sesji checkout jako parametru query string podczas wywoływania URL. Aby to zrobić, dodaj dosłowny ciąg `{CHECKOUT_SESSION_ID}` do query string `success_url`. Stripe zastąpi ten placeholder faktycznym identyfikatorem sesji checkout:

```php
use Illuminate\Http\Request;
use Stripe\Checkout\Session;
use Stripe\Customer;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout(['price_tshirt' => 1], [
        'success_url' => route('checkout-success').'?session_id={CHECKOUT_SESSION_ID}',
        'cancel_url' => route('checkout-cancel'),
    ]);
});

Route::get('/checkout-success', function (Request $request) {
    $checkoutSession = $request->user()->stripe()->checkout->sessions->retrieve($request->get('session_id'));

    return view('checkout.success', ['checkoutSession' => $checkoutSession]);
})->name('checkout-success');
```

<a name="checkout-promotion-codes"></a>
#### Kody promocyjne

Domyślnie Stripe Checkout nie zezwala na [kody promocyjne wykupywane przez użytkownika](https://stripe.com/docs/billing/subscriptions/discounts/codes). Na szczęście istnieje łatwy sposób, aby włączyć je na stronie Checkout. Aby to zrobić, możesz wywołać metodę `allowPromotionCodes`:

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()
        ->allowPromotionCodes()
        ->checkout('price_tshirt');
});
```

<a name="single-charge-checkouts"></a>
### Kasy pojedynczej opłaty

Możesz również wykonać prostą opłatę za produkt ad-hoc, który nie został utworzony w panelu Stripe. Aby to zrobić, możesz użyć metody `checkoutCharge` na modelu rozliczeniowym i przekazać jej kwotę do obciążenia, nazwę produktu i opcjonalną ilość. Gdy klient odwiedzi tę trasę, zostanie przekierowany na stronę Checkout Stripe:

```php
use Illuminate\Http\Request;

Route::get('/charge-checkout', function (Request $request) {
    return $request->user()->checkoutCharge(1200, 'Koszulka', 5);
});
```

> [!WARNING]
> Podczas używania metody `checkoutCharge` Stripe zawsze utworzy nowy produkt i cenę w panelu Stripe. Dlatego zalecamy tworzenie produktów z wyprzedzeniem w panelu Stripe i zamiast tego używanie metody `checkout`.

<a name="subscription-checkouts"></a>
### Kasy subskrypcji

> [!WARNING]
> Używanie Stripe Checkout dla subskrypcji wymaga włączenia webhooka `customer.subscription.created` w panelu Stripe. Ten webhook utworzy rekord subskrypcji w bazie danych i zapisze wszystkie odpowiednie elementy subskrypcji.

Możesz również użyć Stripe Checkout do rozpoczynania subskrypcji. Po zdefiniowaniu subskrypcji za pomocą metod konstruktora subskrypcji Cashier możesz wywołać metodę `checkout`. Gdy klient odwiedzi tę trasę, zostanie przekierowany na stronę Checkout Stripe:

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_monthly')
        ->checkout();
});
```

Tak jak w przypadku kas produktów, możesz dostosować adresy URL sukcesu i anulowania:

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_monthly')
        ->checkout([
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
});
```

Oczywiście możesz również włączyć kody promocyjne dla kas subskrypcji:

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_monthly')
        ->allowPromotionCodes()
        ->checkout();
});
```

> [!WARNING]
> Niestety Stripe Checkout nie obsługuje wszystkich opcji rozliczeniowych subskrypcji podczas rozpoczynania subskrypcji. Używanie metody `anchorBillingCycleOn` w konstruktorze subskrypcji, ustawianie zachowania pro-rata lub ustawianie zachowania płatności nie będzie miało żadnego efektu podczas sesji Stripe Checkout. Zapoznaj się z [dokumentacją API sesji Stripe Checkout](https://stripe.com/docs/api/checkout/sessions/create), aby sprawdzić, które parametry są dostępne.

<a name="stripe-checkout-trial-periods"></a>
#### Stripe Checkout i okresy próbne

Oczywiście możesz zdefiniować okres próbny podczas budowania subskrypcji, która zostanie zakończona za pomocą Stripe Checkout:

```php
$checkout = Auth::user()->newSubscription('default', 'price_monthly')
    ->trialDays(3)
    ->checkout();
```

Jednak okres próbny musi wynosić co najmniej 48 godzin, co jest minimalną ilością czasu próbnego obsługiwaną przez Stripe Checkout.

<a name="stripe-checkout-subscriptions-and-webhooks"></a>
#### Subskrypcje i webhooki

Pamiętaj, że Stripe i Cashier aktualizują statusy subskrypcji za pośrednictwem webhooków, więc istnieje możliwość, że subskrypcja może jeszcze nie być aktywna, gdy klient powróci do aplikacji po wprowadzeniu informacji o płatności. Aby obsłużyć ten scenariusz, możesz chcieć wyświetlić komunikat informujący użytkownika, że jego płatność lub subskrypcja jest w trakcie realizacji.

<a name="collecting-tax-ids"></a>
### Zbieranie numerów identyfikacji podatkowej

Checkout obsługuje również zbieranie numeru identyfikacji podatkowej klienta. Aby włączyć to w sesji checkout, wywołaj metodę `collectTaxIds` podczas tworzenia sesji:

```php
$checkout = $user->collectTaxIds()->checkout('price_tshirt');
```

Kiedy ta metoda jest wywołana, nowe pole wyboru będzie dostępne dla klienta, które pozwala mu wskazać, czy dokonuje zakupu jako firma. Jeśli tak, będą mieli możliwość podania swojego numeru identyfikacji podatkowej.

> [!WARNING]
> Jeśli już skonfigurowałeś [automatyczne pobieranie podatków](#tax-configuration) w dostawcy usług aplikacji, ta funkcja będzie włączona automatycznie i nie ma potrzeby wywoływania metody `collectTaxIds`.

<a name="guest-checkouts"></a>
### Kasy dla gości

Używając metody `Checkout::guest`, możesz inicjować sesje checkout dla gości aplikacji, którzy nie mają "konta":

```php
use Illuminate\Http\Request;
use Laravel\Cashier\Checkout;

Route::get('/product-checkout', function (Request $request) {
    return Checkout::guest()->create('price_tshirt', [
        'success_url' => route('your-success-route'),
        'cancel_url' => route('your-cancel-route'),
    ]);
});
```

Podobnie jak podczas tworzenia sesji checkout dla istniejących użytkowników, możesz wykorzystać dodatkowe metody dostępne w instancji `Laravel\Cashier\CheckoutBuilder`, aby dostosować sesję checkout gościa:

```php
use Illuminate\Http\Request;
use Laravel\Cashier\Checkout;

Route::get('/product-checkout', function (Request $request) {
    return Checkout::guest()
        ->withPromotionCode('promo-code')
        ->create('price_tshirt', [
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
});
```

Po zakończeniu kasy gościa Stripe może wysłać zdarzenie webhooka `checkout.session.completed`, więc upewnij się, że [skonfigurowałeś webhooka Stripe](https://dashboard.stripe.com/webhooks), aby faktycznie wysłać to zdarzenie do aplikacji. Po włączeniu webhooka w panelu Stripe możesz [obsłużyć webhooka za pomocą Cashier](#handling-stripe-webhooks). Obiekt zawarty w payload webhooka będzie [obiektem checkout](https://stripe.com/docs/api/checkout/sessions/object), który możesz sprawdzić, aby zrealizować zamówienie klienta.

<a name="handling-failed-payments"></a>
## Obsługa nieudanych płatności

Czasami płatności za subskrypcje lub pojedyncze opłaty mogą się nie powieść. Kiedy to się dzieje, Cashier zgłosi wyjątek `Laravel\Cashier\Exceptions\IncompletePayment`, który informuje, że to się wydarzyło. Po złapaniu tego wyjątku masz dwie opcje, jak postąpić.

Po pierwsze, możesz przekierować klienta na dedykowaną stronę potwierdzenia płatności, która jest dołączona do Cashier. Ta strona ma już powiązaną nazwaną trasę, która jest zarejestrowana przez dostawcę usług Cashier. Możesz więc złapać wyjątek `IncompletePayment` i przekierować użytkownika na stronę potwierdzenia płatności:

```php
use Laravel\Cashier\Exceptions\IncompletePayment;

try {
    $subscription = $user->newSubscription('default', 'price_monthly')
        ->create($paymentMethod);
} catch (IncompletePayment $exception) {
    return redirect()->route(
        'cashier.payment',
        [$exception->payment->id, 'redirect' => route('home')]
    );
}
```

Na stronie potwierdzenia płatności klient zostanie poproszony o ponowne wprowadzenie danych karty kredytowej i wykonanie wszelkich dodatkowych działań wymaganych przez Stripe, takich jak potwierdzenie "3D Secure". Po potwierdzeniu płatności użytkownik zostanie przekierowany na adres URL podany przez parametr `redirect` określony powyżej. Po przekierowaniu zmienne query string `message` (string) i `success` (integer) zostaną dodane do adresu URL. Strona płatności obsługuje obecnie następujące typy metod płatności:

<div class="content-list" markdown="1">

- Karty kredytowe
- Alipay
- Bancontact
- BECS Direct Debit
- EPS
- Giropay
- iDEAL
- SEPA Direct Debit

</div>

Alternatywnie możesz pozwolić Stripe obsługiwać potwierdzenie płatności za Ciebie. W takim przypadku zamiast przekierowywania na stronę potwierdzenia płatności możesz [skonfigurować automatyczne e-maile rozliczeniowe Stripe](https://dashboard.stripe.com/account/billing/automatic) w panelu Stripe. Jednak jeśli zostanie złapany wyjątek `IncompletePayment`, nadal powinieś poinformować użytkownika, że otrzyma e-mail z dalszymi instrukcjami dotyczącymi potwierdzenia płatności.

Wyjątki płatności mogą zostać zgłoszone dla następujących metod: `charge`, `invoiceFor` i `invoice` na modelach używających cechy `Billable`. Podczas interakcji z subskrypcjami metoda `create` w `SubscriptionBuilder` oraz metody `incrementAndInvoice` i `swapAndInvoice` na modelach `Subscription` i `SubscriptionItem` mogą zgłaszać wyjątki niekompletnych płatności.

Określenie, czy istniejąca subskrypcja ma niekompletną płatność, można wykonać za pomocą metody `hasIncompletePayment` na modelu rozliczeniowym lub instancji subskrypcji:

```php
if ($user->hasIncompletePayment('default')) {
    // ...
}

if ($user->subscription('default')->hasIncompletePayment()) {
    // ...
}
```

Możesz wyprowadzić konkretny status niekompletnej płatności, sprawdzając właściwość `payment` na instancji wyjątku:

```php
use Laravel\Cashier\Exceptions\IncompletePayment;

try {
    $user->charge(1000, 'pm_card_threeDSecure2Required');
} catch (IncompletePayment $exception) {
    // Pobierz status intencji płatności...
    $exception->payment->status;

    // Sprawdź konkretne warunki...
    if ($exception->payment->requiresPaymentMethod()) {
        // ...
    } elseif ($exception->payment->requiresConfirmation()) {
        // ...
    }
}
```

<a name="confirming-payments"></a>
### Potwierdzanie płatności

Niektóre metody płatności wymagają dodatkowych danych w celu potwierdzenia płatności. Na przykład metody płatności SEPA wymagają dodatkowych danych "mandatu" podczas procesu płatności. Możesz dostarczyć te dane do Cashier za pomocą metody `withPaymentConfirmationOptions`:

```php
$subscription->withPaymentConfirmationOptions([
    'mandate_data' => '...',
])->swap('price_xxx');
```

Możesz zapoznać się z [dokumentacją API Stripe](https://stripe.com/docs/api/payment_intents/confirm), aby przejrzeć wszystkie opcje akceptowane podczas potwierdzania płatności.

<a name="strong-customer-authentication"></a>
## Strong Customer Authentication

If your business or one of your customers is based in Europe you will need to abide by the EU's Strong Customer Authentication (SCA) regulations. These regulations were imposed in September 2019 by the European Union to prevent payment fraud. Luckily, Stripe and Cashier are prepared for building SCA compliant applications.

> [!WARNING]
> Before getting started, review [Stripe's guide on PSD2 and SCA](https://stripe.com/guides/strong-customer-authentication) as well as their [documentation on the new SCA APIs](https://stripe.com/docs/strong-customer-authentication).

<a name="payments-requiring-additional-confirmation"></a>
### Płatności wymagające dodatkowego potwierdzenia

Przepisy SCA często wymagają dodatkowej weryfikacji w celu potwierdzenia i przetworzenia płatności. Kiedy to się dzieje, Cashier zgłosi wyjątek `Laravel\Cashier\Exceptions\IncompletePayment`, który informuje, że wymagana jest dodatkowa weryfikacja. Więcej informacji o tym, jak obsługiwać te wyjątki, można znaleźć w dokumentacji dotyczącej [obsługi nieudanych płatności](#handling-failed-payments).

Ekrany potwierdzania płatności prezentowane przez Stripe lub Cashier mogą być dostosowane do przepływu płatności konkretnego banku lub wystawcy karty i mogą obejmować dodatkowe potwierdzenie karty, tymczasową niewielką opłatę, oddzielne uwierzytelnianie urządzenia lub inne formy weryfikacji.

<a name="incomplete-and-past-due-state"></a>
#### Stan niekompletny i zaległy

Kiedy płatność wymaga dodatkowego potwierdzenia, subskrypcja pozostanie w stanie `incomplete` lub `past_due`, jak wskazuje kolumna bazy danych `stripe_status`. Cashier automatycznie aktywuje subskrypcję klienta, gdy tylko potwierdzenie płatności zostanie zakończone, a aplikacja zostanie powiadomiona przez Stripe za pośrednictwem webhooka o jego zakończeniu.

Aby uzyskać więcej informacji na temat stanów `incomplete` i `past_due`, zapoznaj się z [naszą dodatkową dokumentacją dotyczącą tych stanów](#incomplete-and-past-due-status).

<a name="off-session-payment-notifications"></a>
### Powiadomienia o płatnościach poza sesją

Ponieważ przepisy SCA wymagają od klientów okresowej weryfikacji danych płatności, nawet gdy ich subskrypcja jest aktywna, Cashier może wysłać powiadomienie do klienta, gdy wymagane jest potwierdzenie płatności poza sesją. Na przykład może to wystąpić podczas odnawiania subskrypcji. Powiadomienie o płatności Cashier można włączyć, ustawiając zmienną środowiskową `CASHIER_PAYMENT_NOTIFICATION` na klasę powiadomienia. Domyślnie to powiadomienie jest wyłączone. Oczywiście Cashier zawiera klasę powiadomienia, której możesz użyć w tym celu, ale możesz swobodnie dostarczyć własną klasę powiadomienia, jeśli chcesz:

```ini
CASHIER_PAYMENT_NOTIFICATION=Laravel\Cashier\Notifications\ConfirmPayment
```

Aby upewnić się, że powiadomienia o potwierdzeniu płatności poza sesją są dostarczane, sprawdź, czy [webhooki Stripe są skonfigurowane](#handling-stripe-webhooks) dla aplikacji i czy webhook `invoice.payment_action_required` jest włączony w panelu Stripe. Ponadto model `Billable` powinien również używać cechy Laravel `Illuminate\Notifications\Notifiable`.

> [!WARNING]
> Powiadomienia będą wysyłane nawet wtedy, gdy klienci ręcznie dokonują płatności wymagającej dodatkowego potwierdzenia. Niestety Stripe nie ma możliwości, aby dowiedzieć się, czy płatność została wykonana ręcznie, czy "poza sesją". Jednak klient zobaczy po prostu komunikat "Płatność zakończona sukcesem", jeśli odwiedzi stronę płatności po potwierdzeniu płatności. Klient nie będzie mógł przypadkowo potwierdzić tej samej płatności dwa razy i zostać obciążony przypadkowo po raz drugi.

<a name="stripe-sdk"></a>
## Stripe SDK

Wiele obiektów Cashier to opakowania wokół obiektów Stripe SDK. Jeśli chcesz wchodzić w interakcję z obiektami Stripe bezpośrednio, możesz wygodnie je pobrać za pomocą metody `asStripe`:

```php
$stripeSubscription = $subscription->asStripeSubscription();

$stripeSubscription->application_fee_percent = 5;

$stripeSubscription->save();
```

Możesz również użyć metody `updateStripeSubscription`, aby bezpośrednio zaktualizować subskrypcję Stripe:

```php
$subscription->updateStripeSubscription(['application_fee_percent' => 5]);
```

Możesz wywołać metodę `stripe` na klasie `Cashier`, jeśli chcesz użyć klienta `Stripe\StripeClient` bezpośrednio. Na przykład możesz użyć tej metody, aby uzyskać dostęp do instancji `StripeClient` i pobrać listę cen ze swojego konta Stripe:

```php
use Laravel\Cashier\Cashier;

$prices = Cashier::stripe()->prices->all();
```

<a name="testing"></a>
## Testowanie

Podczas testowania aplikacji korzystającej z Cashier możesz mockować rzeczywiste żądania HTTP do API Stripe; jednak wymaga to częściowej re-implementacji własnego zachowania Cashier. Dlatego zalecamy, aby Twoje testy trafiały do rzeczywistego API Stripe. Chociaż jest to wolniejsze, zapewnia większą pewność, że aplikacja działa zgodnie z oczekiwaniami, a wszelkie wolne testy mogą być umieszczone we własnej grupie testowej Pest / PHPUnit.

Podczas testowania pamiętaj, że sam Cashier ma już świetny zestaw testów, więc powinieneś skupić się tylko na testowaniu przepływu subskrypcji i płatności swojej aplikacji, a nie na każdym podstawowym zachowaniu Cashier.

Aby rozpocząć, dodaj **testową** wersję swojego sekretu Stripe do pliku `phpunit.xml`:

```xml
<env name="STRIPE_SECRET" value="sk_test_<your-key>"/>
```

Teraz, za każdym razem, gdy wchodzisz w interakcję z Cashier podczas testowania, będzie wysyłał rzeczywiste żądania API do Twojego środowiska testowego Stripe. Dla wygody powinieneś wstępnie wypełnić swoje konto testowe Stripe subskrypcjami / cenami, których możesz używać podczas testowania.

> [!NOTE]
> Aby przetestować różne scenariusze rozliczeniowe, takie jak odmowa i niepowodzenie karty kredytowej, możesz użyć szerokiej gamy [testowych numerów kart i tokenów](https://stripe.com/docs/testing) dostarczonych przez Stripe.
