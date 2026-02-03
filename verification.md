# Weryfikacja adresu e-mail

- [Wprowadzenie](#introduction)
    - [Przygotowanie modelu](#model-preparation)
    - [Przygotowanie bazy danych](#database-preparation)
- [Routing](#verification-routing)
    - [Powiadomienie o weryfikacji e-mail](#the-email-verification-notice)
    - [Obsługa weryfikacji e-mail](#the-email-verification-handler)
    - [Ponowne wysłanie e-maila weryfikacyjnego](#resending-the-verification-email)
    - [Ochrona tras](#protecting-routes)
- [Dostosowywanie](#customization)
- [Zdarzenia](#events)

<a name="introduction"></a>
## Wprowadzenie

Wiele aplikacji internetowych wymaga od użytkowników weryfikacji adresów e-mail przed rozpoczęciem korzystania z aplikacji. Zamiast zmuszać Cię do ponownego implementowania tej funkcji ręcznie dla każdej tworzonej aplikacji, Laravel zapewnia wygodne wbudowane usługi do wysyłania i weryfikacji żądań weryfikacji e-mail.

> [!NOTE]
> Chcesz szybko zacząć? Zainstaluj jeden z [zestawów startowych aplikacji Laravel](/docs/{{version}}/starter-kits) w świeżej aplikacji Laravel. Zestawy startowe zajmą się utworzeniem całego systemu uwierzytelniania, włącznie ze wsparciem weryfikacji e-mail.

<a name="model-preparation"></a>
### Przygotowanie modelu

Przed rozpoczęciem sprawdź, czy Twój model `App\Models\User` implementuje kontrakt `Illuminate\Contracts\Auth\MustVerifyEmail`:

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable implements MustVerifyEmail
{
    use Notifiable;

    // ...
}
```

Po dodaniu tego interfejsu do modelu, nowo zarejestrowani użytkownicy automatycznie otrzymają e-mail zawierający link weryfikacyjny. Dzieje się to bezproblemowo, ponieważ Laravel automatycznie rejestruje [nasłuchiwacz](/docs/{{version}}/events) `Illuminate\Auth\Listeners\SendEmailVerificationNotification` dla zdarzenia `Illuminate\Auth\Events\Registered`.

Jeśli ręcznie implementujesz rejestrację w swojej aplikacji zamiast używać [zestawu startowego](/docs/{{version}}/starter-kits), powinieneś upewnić się, że wysyłasz zdarzenie `Illuminate\Auth\Events\Registered` po pomyślnej rejestracji użytkownika:

```php
use Illuminate\Auth\Events\Registered;

event(new Registered($user));
```

<a name="database-preparation"></a>
### Przygotowanie bazy danych

Następnie tabela `users` musi zawierać kolumnę `email_verified_at` do przechowywania daty i czasu weryfikacji adresu e-mail użytkownika. Zazwyczaj jest to uwzględnione w domyślnej migracji bazy danych Laravel `0001_01_01_000000_create_users_table.php`.

<a name="verification-routing"></a>
## Routing

Aby prawidłowo zaimplementować weryfikację e-mail, należy zdefiniować trzy trasy. Po pierwsze, potrzebna będzie trasa do wyświetlenia powiadomienia użytkownikowi, że powinien kliknąć link weryfikacyjny w e-mailu weryfikacyjnym, który Laravel wysłał im po rejestracji.

Po drugie, potrzebna będzie trasa do obsługi żądań generowanych, gdy użytkownik kliknie link weryfikacyjny w e-mailu.

Po trzecie, potrzebna będzie trasa do ponownego wysłania linku weryfikacyjnego, jeśli użytkownik przypadkowo straci pierwszy link weryfikacyjny.

<a name="the-email-verification-notice"></a>
### Powiadomienie o weryfikacji e-mail

Jak wspomniano wcześniej, należy zdefiniować trasę, która zwróci widok instruujący użytkownika, aby kliknął link weryfikacyjny e-mail, który został wysłany do niego przez Laravel po rejestracji. Ten widok będzie wyświetlany użytkownikom, gdy spróbują uzyskać dostęp do innych części aplikacji bez uprzedniej weryfikacji adresu e-mail. Pamiętaj, że link jest automatycznie wysyłany do użytkownika, o ile Twój model `App\Models\User` implementuje interfejs `MustVerifyEmail`:

```php
Route::get('/email/verify', function () {
    return view('auth.verify-email');
})->middleware('auth')->name('verification.notice');
```

Trasa zwracająca powiadomienie o weryfikacji e-mail powinna nazywać się `verification.notice`. Ważne jest, aby trasa miała dokładnie tę nazwę, ponieważ middleware `verified` [dołączony do Laravel](#protecting-routes) automatycznie przekieruje do tej nazwy trasy, jeśli użytkownik nie zweryfikował swojego adresu e-mail.

> [!NOTE]
> Podczas ręcznego wdrażania weryfikacji e-mail musisz samodzielnie zdefiniować zawartość widoku powiadomienia o weryfikacji. Jeśli chcesz rusztowanie zawierające wszystkie niezbędne widoki uwierzytelniania i weryfikacji, sprawdź [zestawy startowe aplikacji Laravel](/docs/{{version}}/starter-kits).

<a name="the-email-verification-handler"></a>
### Obsługa weryfikacji e-mail

Następnie musimy zdefiniować trasę, która będzie obsługiwać żądania generowane, gdy użytkownik kliknie link weryfikacyjny e-mail, który został do niego wysłany. Ta trasa powinna nazywać się `verification.verify` i mieć przypisane middleware `auth` i `signed`:

```php
use Illuminate\Foundation\Auth\EmailVerificationRequest;

Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
    $request->fulfill();

    return redirect('/home');
})->middleware(['auth', 'signed'])->name('verification.verify');
```

Zanim przejdziemy dalej, przyjrzyjmy się bliżej tej trasie. Po pierwsze, zauważysz, że używamy typu żądania `EmailVerificationRequest` zamiast typowej instancji `Illuminate\Http\Request`. `EmailVerificationRequest` jest [żądaniem formularza](/docs/{{version}}/validation#form-request-validation) dołączonym do Laravel. To żądanie automatycznie zadba o walidację parametrów `id` i `hash` żądania.

Następnie możemy przejść bezpośrednio do wywołania metody `fulfill` na żądaniu. Ta metoda wywoła metodę `markEmailAsVerified` na uwierzytelnionym użytkowniku i wyśle zdarzenie `Illuminate\Auth\Events\Verified`. Metoda `markEmailAsVerified` jest dostępna dla domyślnego modelu `App\Models\User` poprzez klasę bazową `Illuminate\Foundation\Auth\User`. Po zweryfikowaniu adresu e-mail użytkownika możesz przekierować go dokądkolwiek chcesz.

<a name="resending-the-verification-email"></a>
### Ponowne wysyłanie e-maila weryfikacyjnego

Czasami użytkownik może zgubić lub przypadkowo usunąć e-mail weryfikacyjny adresu e-mail. Aby to uwzględnić, możesz zdefiniować trasę pozwalającą użytkownikowi zażądać ponownego wysłania e-maila weryfikacyjnego. Możesz następnie wykonać żądanie do tej trasy, umieszczając prosty przycisk wysyłania formularza w swoim [widoku powiadomienia o weryfikacji](#the-email-verification-notice):

```php
use Illuminate\Http\Request;

Route::post('/email/verification-notification', function (Request $request) {
    $request->user()->sendEmailVerificationNotification();

    return back()->with('message', 'Verification link sent!');
})->middleware(['auth', 'throttle:6,1'])->name('verification.send');
```

<a name="protecting-routes"></a>
### Ochrona tras

[Middleware trasy](/docs/{{version}}/middleware) można użyć, aby zezwolić tylko zweryfikowanym użytkownikom na dostęp do danej trasy. Laravel zawiera [alias middleware](/docs/{{version}}/middleware#middleware-aliases) `verified`, który jest aliasem dla klasy middleware `Illuminate\Auth\Middleware\EnsureEmailIsVerified`. Ponieważ ten alias jest już automatycznie zarejestrowany przez Laravel, wszystko, co musisz zrobić, to dołączyć middleware `verified` do definicji trasy. Zazwyczaj ten middleware jest łączony z middleware `auth`:

```php
Route::get('/profile', function () {
    // Only verified users may access this route...
})->middleware(['auth', 'verified']);
```

Jeśli niezweryfikowany użytkownik spróbuje uzyskać dostęp do trasy, która ma przypisany ten middleware, zostanie automatycznie przekierowany do [nazwanej trasy](/docs/{{version}}/routing#named-routes) `verification.notice`.

<a name="customization"></a>
## Dostosowywanie

<a name="verification-email-customization"></a>
#### Dostosowywanie e-maila weryfikacyjnego

Choć domyślne powiadomienie o weryfikacji e-mail powinno spełniać wymagania większości aplikacji, Laravel pozwala dostosować sposób konstruowania wiadomości e-mail weryfikacyjnej.

Aby rozpocząć, przekaż domknięcie do metody `toMailUsing` dostarczonej przez powiadomienie `Illuminate\Auth\Notifications\VerifyEmail`. Domknięcie otrzyma instancję modelu powiadamianego, który otrzymuje powiadomienie, oraz podpisany URL weryfikacji e-mail, który użytkownik musi odwiedzić, aby zweryfikować swój adres e-mail. Domknięcie powinno zwrócić instancję `Illuminate\Notifications\Messages\MailMessage`. Zazwyczaj powinieneś wywołać metodę `toMailUsing` z metody `boot` klasy `AppServiceProvider` Twojej aplikacji:

```php
use Illuminate\Auth\Notifications\VerifyEmail;
use Illuminate\Notifications\Messages\MailMessage;

/**
 * Inicjalizuje dowolne usługi aplikacji.
 */
public function boot(): void
{
    // ...

    VerifyEmail::toMailUsing(function (object $notifiable, string $url) {
        return (new MailMessage)
            ->subject('Verify Email Address')
            ->line('Click the button below to verify your email address.')
            ->action('Verify Email Address', $url);
    });
}
```

> [!NOTE]
> Aby dowiedzieć się więcej o powiadomieniach e-mail, zapoznaj się z [dokumentacją powiadomień e-mail](/docs/{{version}}/notifications#mail-notifications).

<a name="events"></a>
## Zdarzenia

Podczas korzystania z [zestawów startowych aplikacji Laravel](/docs/{{version}}/starter-kits), Laravel wysyła [zdarzenie](/docs/{{version}}/events) `Illuminate\Auth\Events\Verified` podczas procesu weryfikacji e-mail. Jeśli ręcznie obsługujesz weryfikację e-mail dla swojej aplikacji, możesz chcieć ręcznie wysłać te zdarzenia po zakończeniu weryfikacji.
