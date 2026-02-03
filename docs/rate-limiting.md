# Limitowanie Częstotliwości

- [Wprowadzenie](#introduction)
    - [Konfiguracja Cache](#cache-configuration)
- [Podstawowe Użycie](#basic-usage)
    - [Ręczne Zwiększanie Prób](#manually-incrementing-attempts)
    - [Czyszczenie Prób](#clearing-attempts)

<a name="introduction"></a>
## Wprowadzenie

Laravel zawiera prostą w użyciu abstrakcję limitowania częstotliwości, która w połączeniu z [cache](cache) twojej aplikacji, zapewnia łatwy sposób na ograniczenie dowolnej akcji w określonym oknie czasowym.

> [!NOTE]
> Jeśli jesteś zainteresowany limitowaniem częstotliwości przychodzących żądań HTTP, sprawdź [dokumentację middleware limitera częstotliwości](/docs/{{version}}/routing#rate-limiting).

<a name="cache-configuration"></a>
### Konfiguracja Cache

Zazwyczaj limiter częstotliwości wykorzystuje domyślny cache aplikacji, zdefiniowany przez klucz `default` w pliku konfiguracyjnym `cache` twojej aplikacji. Możesz jednak określić, którego sterownika cache powinien używać limiter częstotliwości, definiując klucz `limiter` w pliku konfiguracyjnym `cache` twojej aplikacji:

```php
'default' => env('CACHE_STORE', 'database'),

'limiter' => 'redis', // [tl! add]
```

<a name="basic-usage"></a>
## Podstawowe Użycie

Fasada `Illuminate\Support\Facades\RateLimiter` może być używana do interakcji z limiterem częstotliwości. Najprostsza metoda oferowana przez limiter częstotliwości to metoda `attempt`, która limituje częstotliwość danego callbacka przez określoną liczbę sekund.

Metoda `attempt` zwraca `false`, gdy callback nie ma już dostępnych prób; w przeciwnym razie metoda `attempt` zwróci wynik callbacka lub `true`. Pierwszy argument akceptowany przez metodę `attempt` to "klucz" limitera częstotliwości, który może być dowolnym ciągiem znaków reprezentującym limitowaną akcję:

```php
use Illuminate\Support\Facades\RateLimiter;

$executed = RateLimiter::attempt(
    'send-message:'.$user->id,
    $perMinute = 5,
    function() {
        // Wyślij wiadomość...
    }
);

if (! $executed) {
    return 'Wysłano zbyt wiele wiadomości!';
}
```

W razie potrzeby możesz podać czwarty argument dla metody `attempt`, którym jest "współczynnik zaniku" (decay rate), czyli liczba sekund, po których dostępne próby są resetowane. Na przykład możemy zmodyfikować powyższy przykład, aby umożliwić pięć prób co dwie minuty:

```php
$executed = RateLimiter::attempt(
    'send-message:'.$user->id,
    $perTwoMinutes = 5,
    function() {
        // Wyślij wiadomość...
    },
    $decayRate = 120,
);
```

<a name="manually-incrementing-attempts"></a>
### Ręczne Zwiększanie Prób

Jeśli chcesz ręcznie wchodzić w interakcję z limiterem częstotliwości, dostępnych jest wiele innych metod. Na przykład możesz wywołać metodę `tooManyAttempts`, aby określić, czy dany klucz limitera częstotliwości przekroczył maksymalną liczbę dozwolonych prób na minutę:

```php
use Illuminate\Support\Facades\RateLimiter;

if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
    return 'Zbyt wiele prób!';
}

RateLimiter::increment('send-message:'.$user->id);

// Wyślij wiadomość...
```

Alternatywnie możesz użyć metody `remaining`, aby uzyskać liczbę pozostałych prób dla danego klucza. Jeśli dany klucz ma pozostałe próby, możesz wywołać metodę `increment`, aby zwiększyć liczbę całkowitych prób:

```php
use Illuminate\Support\Facades\RateLimiter;

if (RateLimiter::remaining('send-message:'.$user->id, $perMinute = 5)) {
    RateLimiter::increment('send-message:'.$user->id);

    // Wyślij wiadomość...
}
```

Jeśli chcesz zwiększyć wartość dla danego klucza limitera częstotliwości o więcej niż jeden, możesz podać żądaną wartość do metody `increment`:

```php
RateLimiter::increment('send-message:'.$user->id, amount: 5);
```

<a name="determining-limiter-availability"></a>
#### Określanie Dostępności Limitera

Gdy klucz nie ma już więcej prób, metoda `availableIn` zwraca liczbę sekund pozostałych do momentu, gdy więcej prób będzie dostępnych:

```php
use Illuminate\Support\Facades\RateLimiter;

if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
    $seconds = RateLimiter::availableIn('send-message:'.$user->id);

    return 'Możesz spróbować ponownie za '.$seconds.' sekund.';
}

RateLimiter::increment('send-message:'.$user->id);

// Wyślij wiadomość...
```

<a name="clearing-attempts"></a>
### Czyszczenie Prób

Możesz zresetować liczbę prób dla danego klucza limitera częstotliwości za pomocą metody `clear`. Na przykład możesz zresetować liczbę prób, gdy dana wiadomość zostanie przeczytana przez odbiorcę:

```php
use App\Models\Message;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Oznacz wiadomość jako przeczytaną.
 */
public function read(Message $message): Message
{
    $message->markAsRead();

    RateLimiter::clear('send-message:'.$message->user_id);

    return $message;
}
```
