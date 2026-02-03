# Ochrona CSRF

- [Wprowadzenie](#csrf-introduction)
- [Zapobieganie żądaniom CSRF](#preventing-csrf-requests)
    - [Wykluczanie URI](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## Wprowadzenie

Podrabianie żądań między witrynami (Cross-site request forgery) to rodzaj złośliwego exploitu, w którym nieautoryzowane polecenia są wykonywane w imieniu uwierzytelnionego użytkownika. Na szczęście Laravel ułatwia ochronę aplikacji przed atakami [cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF).

<a name="csrf-explanation"></a>
#### Wyjaśnienie podatności

Jeśli nie znasz podrabiania żądań między witrynami, omówmy przykład, jak można wykorzystać tę podatność. Wyobraź sobie, że Twoja aplikacja ma trasę `/user/email`, która akceptuje żądanie `POST` w celu zmiany adresu e-mail uwierzytelnionego użytkownika. Prawdopodobnie ta trasa oczekuje, że pole wejściowe `email` będzie zawierać adres e-mail, którego użytkownik chce zacząć używać.

Bez ochrony CSRF złośliwa witryna może stworzyć formularz HTML, który wskazuje na trasę `/user/email` Twojej aplikacji i przesyła własny adres e-mail złośliwego użytkownika:

```blade
<form action="https://your-application.com/user/email" method="POST">
    <input type="email" value="malicious-email@example.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

Jeśli złośliwa witryna automatycznie przesyła formularz po załadowaniu strony, złośliwy użytkownik musi jedynie zwabić nieostrożnego użytkownika Twojej aplikacji do odwiedzenia jego witryny, a ich adres e-mail zostanie zmieniony w Twojej aplikacji.

Aby zapobiec tej podatności, musimy sprawdzić każde przychodzące żądanie `POST`, `PUT`, `PATCH` lub `DELETE` pod kątem tajnej wartości sesji, do której złośliwa aplikacja nie ma dostępu.

<a name="preventing-csrf-requests"></a>
## Zapobieganie żądaniom CSRF

Laravel automatycznie generuje "token" CSRF dla każdej aktywnej [sesji użytkownika](/docs/{{version}}/session) zarządzanej przez aplikację. Token ten służy do weryfikacji, że uwierzytelniony użytkownik jest osobą faktycznie wykonującą żądania do aplikacji. Ponieważ ten token jest przechowywany w sesji użytkownika i zmienia się za każdym razem, gdy sesja jest regenerowana, złośliwa aplikacja nie może uzyskać do niego dostępu.

Token CSRF bieżącej sesji można uzyskać za pomocą sesji żądania lub za pomocą funkcji pomocniczej `csrf_token`:

```php
use Illuminate\Http\Request;

Route::get('/token', function (Request $request) {
    $token = $request->session()->token();

    $token = csrf_token();

    // ...
});
```

Za każdym razem, gdy definiujesz formularz HTML "POST", "PUT", "PATCH" lub "DELETE" w swojej aplikacji, powinieneś umieścić w formularzu ukryte pole CSRF `_token`, aby middleware ochrony CSRF mógł zweryfikować żądanie. Dla wygody możesz użyć dyrektywy Blade `@csrf`, aby wygenerować ukryte pole wejściowe tokenu:

```blade
<form method="POST" action="/profile">
    @csrf

    <!-- Equivalent to... -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

[Middleware](/docs/{{version}}/middleware) `Illuminate\Foundation\Http\Middleware\ValidateCsrfToken`, który jest domyślnie zawarty w grupie middleware `web`, automatycznie zweryfikuje, czy token w danych wejściowych żądania pasuje do tokenu przechowywanego w sesji. Gdy te dwa tokeny pasują, wiemy, że uwierzytelniony użytkownik jest tym, który inicjuje żądanie.

<a name="csrf-tokens-and-spas"></a>
### Tokeny CSRF i SPA

Jeśli budujesz SPA, która wykorzystuje Laravel jako backend API, powinieneś skonsultować się z [dokumentacją Laravel Sanctum](/docs/{{version}}/sanctum), aby uzyskać informacje na temat uwierzytelniania z API i ochrony przed podatnościami CSRF.

<a name="csrf-excluding-uris"></a>
### Wykluczanie URI z ochrony CSRF

Czasami możesz chcieć wykluczyć zestaw URI z ochrony CSRF. Na przykład, jeśli używasz [Stripe](https://stripe.com) do przetwarzania płatności i korzystasz z ich systemu webhooków, będziesz musiał wykluczyć trasę obsługi webhooka Stripe z ochrony CSRF, ponieważ Stripe nie będzie wiedział, jaki token CSRF wysłać do Twoich tras.

Zazwyczaj powinieneś umieścić tego rodzaju trasy poza grupą middleware `web`, którą Laravel stosuje do wszystkich tras w pliku `routes/web.php`. Możesz jednak również wykluczyć konkretne trasy, podając ich URI do metody `validateCsrfTokens` w pliku `bootstrap/app.php` Twojej aplikacji:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->validateCsrfTokens(except: [
        'stripe/*',
        'http://example.com/foo/bar',
        'http://example.com/foo/*',
    ]);
})
```

> [!NOTE]
> Dla wygody middleware CSRF jest automatycznie wyłączany dla wszystkich tras podczas [uruchamiania testów](/docs/{{version}}/testing).

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

Oprócz sprawdzania tokenu CSRF jako parametru POST, middleware `Illuminate\Foundation\Http\Middleware\ValidateCsrfToken`, który jest domyślnie zawarty w grupie middleware `web`, sprawdzi również nagłówek żądania `X-CSRF-TOKEN`. Możesz na przykład przechowywać token w tagu HTML `meta`:

```blade
<meta name="csrf-token" content="{{ csrf_token() }}">
```

Następnie możesz poinstruować bibliotekę taką jak jQuery, aby automatycznie dodawała token do wszystkich nagłówków żądań. Zapewnia to prostą i wygodną ochronę CSRF dla aplikacji opartych na AJAX wykorzystujących starszą technologię JavaScript:

```js
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel przechowuje bieżący token CSRF w zaszyfrowanym pliku cookie `XSRF-TOKEN`, który jest dołączany do każdej odpowiedzi generowanej przez framework. Możesz użyć wartości tego pliku cookie, aby ustawić nagłówek żądania `X-XSRF-TOKEN`.

Ten plik cookie jest wysyłany głównie dla wygody programistów, ponieważ niektóre frameworki i biblioteki JavaScript, takie jak Angular i Axios, automatycznie umieszczają jego wartość w nagłówku `X-XSRF-TOKEN` przy żądaniach tego samego pochodzenia.

> [!NOTE]
> Domyślnie plik `resources/js/bootstrap.js` zawiera bibliotekę HTTP Axios, która automatycznie wyśle nagłówek `X-XSRF-TOKEN` za Ciebie.
