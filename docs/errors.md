# Obsługa Błędów

- [Wprowadzenie](#introduction)
- [Konfiguracja](#configuration)
- [Obsługa Wyjątków](#handling-exceptions)
    - [Raportowanie Wyjątków](#reporting-exceptions)
    - [Poziomy Logowania Wyjątków](#exception-log-levels)
    - [Ignorowanie Wyjątków według Typu](#ignoring-exceptions-by-type)
    - [Renderowanie Wyjątków](#rendering-exceptions)
    - [Wyjątki Raportowalne i Renderowalne](#renderable-exceptions)
- [Throttling Raportowanych Wyjątków](#throttling-reported-exceptions)
- [Wyjątki HTTP](#http-exceptions)
    - [Niestandardowe Strony Błędów HTTP](#custom-http-error-pages)

<a name="introduction"></a>
## Wprowadzenie

Kiedy rozpoczynasz nowy projekt Laravel, obsługa błędów i wyjątków jest już skonfigurowana dla Ciebie; jednak w dowolnym momencie możesz użyć metody `withExceptions` w pliku `bootstrap/app.php` swojej aplikacji, aby zarządzać tym, jak wyjątki są raportowane i renderowane przez Twoją aplikację.

Obiekt `$exceptions` dostarczony do domknięcia `withExceptions` jest instancją `Illuminate\Foundation\Configuration\Exceptions` i jest odpowiedzialny za zarządzanie obsługą wyjątków w Twojej aplikacji. Zagłębimy się w ten obiekt w dalszej części tej dokumentacji.

<a name="configuration"></a>
## Konfiguracja

Opcja `debug` w pliku konfiguracyjnym `config/app.php` określa, ile informacji o błędzie jest faktycznie wyświetlane użytkownikowi. Domyślnie ta opcja jest ustawiona tak, aby respektować wartość zmiennej środowiskowej `APP_DEBUG`, która jest przechowywana w Twoim pliku `.env`.

Podczas lokalnego rozwoju powinieneś ustawić zmienną środowiskową `APP_DEBUG` na `true`. **W środowisku produkcyjnym ta wartość powinna zawsze wynosić `false`. Jeśli wartość jest ustawiona na `true` w produkcji, ryzykujesz ujawnienie wrażliwych wartości konfiguracyjnych użytkownikom końcowym Twojej aplikacji.**

<a name="handling-exceptions"></a>
## Obsługa Wyjątków

<a name="reporting-exceptions"></a>
### Raportowanie Wyjątków

W Laravel raportowanie wyjątków służy do logowania wyjątków lub wysyłania ich do zewnętrznej usługi, takiej jak [Sentry](https://github.com/getsentry/sentry-laravel) lub [Flare](https://flareapp.io). Domyślnie wyjątki będą logowane na podstawie Twojej konfiguracji [logowania](/docs/{{version}}/logging). Jednak możesz swobodnie logować wyjątki w dowolny sposób.

Jeśli potrzebujesz raportować różne typy wyjątków w różny sposób, możesz użyć metody wyjątku `report` w pliku `bootstrap/app.php` swojej aplikacji, aby zarejestrować domknięcie, które powinno zostać wykonane, gdy wyjątek danego typu wymaga raportowania. Laravel określi, jaki typ wyjątku raportuje domknięcie, badając type-hint domknięcia:

```php
use App\Exceptions\InvalidOrderException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    });
})
```

Kiedy rejestrujesz niestandardowe wywołanie zwrotne raportowania wyjątków za pomocą metody `report`, Laravel nadal będzie logować wyjątek za pomocą domyślnej konfiguracji logowania aplikacji. Jeśli chcesz zatrzymać propagację wyjątku do domyślnego stosu logowania, możesz użyć metody `stop` podczas definiowania swojego wywołania zwrotnego raportowania lub zwrócić `false` z wywołania zwrotnego:

```php
use App\Exceptions\InvalidOrderException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    })->stop();

    $exceptions->report(function (InvalidOrderException $e) {
        return false;
    });
})
```

> [!NOTE]
> Aby dostosować raportowanie wyjątków dla danego wyjątku, możesz również wykorzystać [wyjątki raportowalne](/docs/{{version}}/errors#renderable-exceptions).

<a name="global-log-context"></a>
#### Globalny Kontekst Logu

Jeśli dostępne, Laravel automatycznie dodaje ID bieżącego użytkownika do każdej wiadomości logowania wyjątku jako danych kontekstowych. Możesz zdefiniować własne globalne dane kontekstowe za pomocą metody wyjątku `context` w pliku `bootstrap/app.php` Twojej aplikacji. Te informacje zostaną uwzględnione w każdej wiadomości logowania wyjątku zapisywanej przez Twoją aplikację:

```php
->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->context(fn () => [
        'foo' => 'bar',
    ]);
})
```

<a name="exception-log-context"></a>
#### Kontekst Logu Wyjątku

Choć dodawanie kontekstu do każdej wiadomości logowania może być przydatne, czasami konkretny wyjątek może mieć unikalny kontekst, który chcesz uwzględnić w swoich logach. Definiując metodę `context` na jednym z wyjątków Twojej aplikacji, możesz określić dowolne dane istotne dla tego wyjątku, które powinny zostać dodane do wpisu logowania wyjątku:

```php
<?php

namespace App\Exceptions;

use Exception;

class InvalidOrderException extends Exception
{
    // ...

    /**
     * Pobierz informacje kontekstowe wyjątku.
     *
     * @return array<string, mixed>
     */
    public function context(): array
    {
        return ['order_id' => $this->orderId];
    }
}
```

<a name="the-report-helper"></a>
#### Helper `report`

Czasami możesz potrzebować zgłosić wyjątek, ale kontynuować obsługę bieżącego żądania. Funkcja pomocnicza `report` pozwala szybko zgłosić wyjątek bez renderowania strony błędu użytkownikowi:

```php
public function isValid(string $value): bool
{
    try {
        // Waliduj wartość...
    } catch (Throwable $e) {
        report($e);

        return false;
    }
}
```

<a name="deduplicating-reported-exceptions"></a>
#### Deduplikacja Raportowanych Wyjątków

Jeśli używasz funkcji `report` w całej aplikacji, możesz czasami raportować ten sam wyjątek wiele razy, tworząc zduplikowane wpisy w logach.

Jeśli chcesz zapewnić, że pojedyncza instancja wyjątku jest raportowana tylko raz, możesz wywołać metodę wyjątku `dontReportDuplicates` w pliku `bootstrap/app.php` swojej aplikacji:

```php
->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->dontReportDuplicates();
})
```

Teraz, gdy helper `report` jest wywoływany z tą samą instancją wyjątku, tylko pierwsze wywołanie zostanie zaraportowane:

```php
$original = new RuntimeException('Whoops!');

report($original); // raportowany

try {
    throw $original;
} catch (Throwable $caught) {
    report($caught); // zignorowany
}

report($original); // zignorowany
report($caught); // zignorowany
```

<a name="exception-log-levels"></a>
### Poziomy Logowania Wyjątków

Kiedy wiadomości są zapisywane do [logów](/docs/{{version}}/logging) Twojej aplikacji, wiadomości są zapisywane na określonym [poziomie logu](/docs/{{version}}/logging#log-levels), który wskazuje na wagę lub ważność logowanej wiadomości.

Jak wspomniano powyżej, nawet jeśli zarejestrujesz niestandardowe wywołanie zwrotne raportowania wyjątków za pomocą metody `report`, Laravel nadal będzie logować wyjątek za pomocą domyślnej konfiguracji logowania aplikacji; jednakże, ponieważ poziom logu może czasami wpływać na kanały, na których wiadomość jest logowana, możesz chcieć skonfigurować poziom logu, na którym pewne wyjątki są logowane.

Aby to osiągnąć, możesz użyć metody wyjątku `level` w pliku `bootstrap/app.php` Twojej aplikacji. Ta metoda przyjmuje typ wyjątku jako pierwszy argument, a poziom logu jako drugi argument:

```php
use PDOException;
use Psr\Log\LogLevel;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->level(PDOException::class, LogLevel::CRITICAL);
})
```

<a name="ignoring-exceptions-by-type"></a>
### Ignorowanie Wyjątków według Typu

Podczas budowania aplikacji będą pewne typy wyjątków, których nigdy nie chcesz raportować. Aby zignorować te wyjątki, możesz użyć metody wyjątku `dontReport` w pliku `bootstrap/app.php` Twojej aplikacji. Żadna klasa dostarczona do tej metody nigdy nie będzie raportowana; jednak nadal mogą mieć niestandardową logikę renderowania:

```php
use App\Exceptions\InvalidOrderException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->dontReport([
        InvalidOrderException::class,
    ]);
})
```

Alternatywnie, możesz po prostu "oznaczyć" klasę wyjątku interfejsem `Illuminate\Contracts\Debug\ShouldntReport`. Kiedy wyjątek jest oznaczony tym interfejsem, nigdy nie będzie raportowany przez obsługę wyjątków Laravel:

```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Contracts\Debug\ShouldntReport;

class PodcastProcessingException extends Exception implements ShouldntReport
{
    //
}
```

Jeśli potrzebujesz jeszcze większej kontroli nad tym, kiedy konkretny typ wyjątku jest ignorowany, możesz przekazać domknięcie do metody `dontReportWhen`:

```php
use App\Exceptions\InvalidOrderException;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->dontReportWhen(function (Throwable $e) {
        return $e instanceof PodcastProcessingException &&
               $e->reason() === 'Subscription expired';
    });
})
```

Wewnętrznie Laravel już ignoruje dla Ciebie niektóre typy błędów, takie jak wyjątki wynikające z błędów HTTP 404 lub odpowiedzi HTTP 419 generowanych przez nieprawidłowe tokeny CSRF. Jeśli chcesz poinstruować Laravel, aby przestał ignorować dany typ wyjątku, możesz użyć metody wyjątku `stopIgnoring` w pliku `bootstrap/app.php` Twojej aplikacji:

```php
use Symfony\Component\HttpKernel\Exception\HttpException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->stopIgnoring(HttpException::class);
})
```

<a name="rendering-exceptions"></a>
### Renderowanie Wyjątków

Domyślnie obsługa wyjątków Laravel przekonwertuje wyjątki na odpowiedź HTTP dla Ciebie. Jednak możesz swobodnie zarejestrować niestandardowe domknięcie renderowania dla wyjątków danego typu. Możesz to osiągnąć za pomocą metody wyjątku `render` w pliku `bootstrap/app.php` Twojej aplikacji.

Domknięcie przekazane do metody `render` powinno zwrócić instancję `Illuminate\Http\Response`, która może być wygenerowana za pomocą helpera `response`. Laravel określi, jaki typ wyjątku renderuje domknięcie, badając type-hint domknięcia:

```php
use App\Exceptions\InvalidOrderException;
use Illuminate\Http\Request;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->render(function (InvalidOrderException $e, Request $request) {
        return response()->view('errors.invalid-order', status: 500);
    });
})
```

Możesz również użyć metody `render`, aby nadpisać zachowanie renderowania dla wbudowanych wyjątków Laravel lub Symfony, takich jak `NotFoundHttpException`. Jeśli domknięcie przekazane do metody `render` nie zwróci wartości, zostanie użyte domyślne renderowanie wyjątków Laravel:

```php
use Illuminate\Http\Request;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->render(function (NotFoundHttpException $e, Request $request) {
        if ($request->is('api/*')) {
            return response()->json([
                'message' => 'Record not found.'
            ], 404);
        }
    });
})
```

<a name="rendering-exceptions-as-json"></a>
#### Renderowanie Wyjątków jako JSON

Podczas renderowania wyjątku Laravel automatycznie określi, czy wyjątek powinien zostać wyrenderowany jako odpowiedź HTML czy JSON na podstawie nagłówka `Accept` żądania. Jeśli chcesz dostosować sposób, w jaki Laravel określa, czy renderować odpowiedzi na wyjątki HTML czy JSON, możesz użyć metody `shouldRenderJsonWhen`:

```php
use Illuminate\Http\Request;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->shouldRenderJsonWhen(function (Request $request, Throwable $e) {
        if ($request->is('admin/*')) {
            return true;
        }

        return $request->expectsJson();
    });
})
```

<a name="customizing-the-exception-response"></a>
#### Dostosowywanie Odpowiedzi Wyjątku

Rzadko możesz potrzebować dostosować całą odpowiedź HTTP renderowaną przez obsługę wyjątków Laravel. Aby to osiągnąć, możesz zarejestrować domknięcie dostosowania odpowiedzi za pomocą metody `respond`:

```php
use Symfony\Component\HttpFoundation\Response;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->respond(function (Response $response) {
        if ($response->getStatusCode() === 419) {
            return back()->with([
                'message' => 'The page expired, please try again.',
            ]);
        }

        return $response;
    });
})
```

<a name="renderable-exceptions"></a>
### Wyjątki Raportowalne i Renderowalne

Zamiast definiować niestandardowe zachowanie raportowania i renderowania w pliku `bootstrap/app.php` Twojej aplikacji, możesz zdefiniować metody `report` i `render` bezpośrednio na wyjątkach Twojej aplikacji. Kiedy te metody istnieją, zostaną automatycznie wywołane przez framework:

```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Http\Request;
use Illuminate\Http\Response;

class InvalidOrderException extends Exception
{
    /**
     * Zgłoś wyjątek.
     */
    public function report(): void
    {
        // ...
    }

    /**
     * Renderuj wyjątek jako odpowiedź HTTP.
     */
    public function render(Request $request): Response
    {
        return response(/* ... */);
    }
}
```

Jeśli Twój wyjątek rozszerza wyjątek, który jest już renderowalny, taki jak wbudowany wyjątek Laravel lub Symfony, możesz zwrócić `false` z metody `render` wyjątku, aby wyrenderować domyślną odpowiedź HTTP wyjątku:

```php
/**
 * Renderuj wyjątek jako odpowiedź HTTP.
 */
public function render(Request $request): Response|bool
{
    if (/** Określ, czy wyjątek wymaga niestandardowego renderowania */) {

        return response(/* ... */);
    }

    return false;
}
```

Jeśli Twój wyjątek zawiera niestandardową logikę raportowania, która jest konieczna tylko wtedy, gdy spełnione są określone warunki, możesz potrzebować poinstruować Laravel, aby czasami raportował wyjątek za pomocą domyślnej konfiguracji obsługi wyjątków. Aby to osiągnąć, możesz zwrócić `false` z metody `report` wyjątku:

```php
/**
 * Zgłoś wyjątek.
 */
public function report(): bool
{
    if (/** Określ, czy wyjątek wymaga niestandardowego raportowania */) {

        // ...

        return true;
    }

    return false;
}
```

> [!NOTE]
> Możesz użyć type-hint dla dowolnych wymaganych zależności metody `report`, a zostaną one automatycznie wstrzyknięte do metody przez [kontener usług](/docs/{{version}}/container) Laravel.

<a name="throttling-reported-exceptions"></a>
### Throttling Raportowanych Wyjątków

Jeśli Twoja aplikacja raportuje bardzo dużą liczbę wyjątków, możesz chcieć ograniczyć, ile wyjątków jest faktycznie logowanych lub wysyłanych do zewnętrznej usługi śledzenia błędów Twojej aplikacji.

Aby wziąć losową próbkę wyjątków, możesz użyć metody wyjątku `throttle` w pliku `bootstrap/app.php` Twojej aplikacji. Metoda `throttle` otrzymuje domknięcie, które powinno zwrócić instancję `Lottery`:

```php
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        return Lottery::odds(1, 1000);
    });
})
```

Możliwe jest również warunkowe próbkowanie na podstawie typu wyjątku. Jeśli chcesz próbkować tylko instancje określonej klasy wyjątku, możesz zwrócić instancję `Lottery` tylko dla tej klasy:

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof ApiMonitoringException) {
            return Lottery::odds(1, 1000);
        }
    });
})
```

Możesz również ograniczyć wyjątki logowane lub wysyłane do zewnętrznej usługi śledzenia błędów, zwracając instancję `Limit` zamiast `Lottery`. Jest to przydatne, jeśli chcesz chronić się przed nagłymi skokami wyjątków zalewającymi Twoje logi, na przykład, gdy usługa strony trzeciej używana przez Twoją aplikację jest niedostępna:

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof BroadcastException) {
            return Limit::perMinute(300);
        }
    });
})
```

Domyślnie limity będą używać klasy wyjątku jako klucza limitu szybkości. Możesz to dostosować, określając własny klucz za pomocą metody `by` na `Limit`:

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof BroadcastException) {
            return Limit::perMinute(300)->by($e->getMessage());
        }
    });
})
```

Oczywiście możesz zwrócić mieszankę instancji `Lottery` i `Limit` dla różnych wyjątków:

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        return match (true) {
            $e instanceof BroadcastException => Limit::perMinute(300),
            $e instanceof ApiMonitoringException => Lottery::odds(1, 1000),
            default => Limit::none(),
        };
    });
})
```

<a name="http-exceptions"></a>
## Wyjątki HTTP

Niektóre wyjątki opisują kody błędów HTTP z serwera. Na przykład może to być błąd "strona nie znaleziona" (404), "błąd nieautoryzowany" (401), a nawet błąd 500 wygenerowany przez programistę. Aby wygenerować taką odpowiedź z dowolnego miejsca w aplikacji, możesz użyć helpera `abort`:

```php
abort(404);
```

<a name="custom-http-error-pages"></a>
### Niestandardowe Strony Błędów HTTP

Laravel ułatwia wyświetlanie niestandardowych stron błędów dla różnych kodów statusu HTTP. Na przykład, aby dostosować stronę błędu dla kodów statusu HTTP 404, utwórz szablon widoku `resources/views/errors/404.blade.php`. Ten widok będzie renderowany dla wszystkich błędów 404 generowanych przez Twoją aplikację. Widoki w tym katalogu powinny być nazwane tak, aby odpowiadały kodowi statusu HTTP. Instancja `Symfony\Component\HttpKernel\Exception\HttpException` wywołana przez funkcję `abort` zostanie przekazana do widoku jako zmienna `$exception`:

```blade
<h2>{{ $exception->getMessage() }}</h2>
```

Możesz opublikować domyślne szablony stron błędów Laravel za pomocą polecenia Artisan `vendor:publish`. Po opublikowaniu szablonów możesz je dostosować według własnych upodobań:

```shell
php artisan vendor:publish --tag=laravel-errors
```

<a name="fallback-http-error-pages"></a>
#### Zapasowe Strony Błędów HTTP

Możesz również zdefiniować "zapasową" stronę błędu dla danej serii kodów statusu HTTP. Ta strona będzie renderowana, jeśli nie ma odpowiedniej strony dla konkretnego kodu statusu HTTP, który wystąpił. Aby to osiągnąć, zdefiniuj szablon `4xx.blade.php` i szablon `5xx.blade.php` w katalogu `resources/views/errors` Twojej aplikacji.

Podczas definiowania zapasowych stron błędów, strony zapasowe nie będą wpływać na odpowiedzi błędów `404`, `500` i `503`, ponieważ Laravel ma wewnętrzne, dedykowane strony dla tych kodów statusu. Aby dostosować strony renderowane dla tych kodów statusu, powinieneś zdefiniować niestandardową stronę błędu dla każdego z nich indywidualnie.
