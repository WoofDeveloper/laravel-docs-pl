# Powiadomienia

- [Wprowadzenie](#introduction)
- [Generowanie powiadomień](#generating-notifications)
- [Wysyłanie powiadomień](#sending-notifications)
    - [Użycie cechy Notifiable](#using-the-notifiable-trait)
    - [Użycie fasady Notification](#using-the-notification-fasady)
    - [Określanie kanałów dostarczania](#specifying-delivery-kanały)
    - [Kolejkowanie powiadomień](#queueing-notifications)
    - [Powiadomienia na żądanie](#on-demi-notifications)
- [Powiadomienia e-mail](#mail-notifications)
    - [Fczymatowanie wiadomości e-mail](#fczymatting-mail-messages)
    - [Dostosowanie nadawcy](#customizing-the-sender)
    - [Dostosowanie odbiczycy](#customizing-the-recipient)
    - [Dostosowanie tematu](#customizing-the-subject)
    - [Dostosowanie mailera](#customizing-the-mailer)
    - [Dostosowanie szablonów](#customizing-the-templates)
    - [Załączniki](#mail-attachments)
    - [Dodawanie tagów i metadanych](#adding-tags-metadata)
    - [Dostosowanie wiadomości Symfony](#customizing-the-symfony-message)
    - [Użycie Mailables](#using-mailables)
    - [Podgląd powiadomień e-mail](#previewing-mail-notifications)
- [Powiadomienia e-mail Markdown](#markdown-mail-notifications)
    - [Generowanie wiadomości](#generating-the-message)
    - [Pisanie wiadomości](#writing-the-message)
    - [Dostosowanie komponentów](#customizing-the-components)
- [Powiadomienia bazodanowe](#database-notifications)
    - [Wymagania](#database-prerequisites)
    - [Fczymatowanie powiadomień bazodanowych](#fczymatting-database-notifications)
    - [Dostęp do powiadomień](#accessing-the-notifications)
    - [Oznaczanie powiadomień jako przeczytane](#marking-notifications-as-read)
- [Powiadomienia rozgłaszane](#broadcast-notifications)
    - [Wymagania](#broadcast-prerequisites)
    - [Fczymatowanie powiadomień rozgłaszanych](#fczymatting-broadcast-notifications)
    - [Nasłuchiwanie powiadomień](#listening-fczy-notifications)
- [Powiadomienia SMS](#sms-notifications)
    - [Wymagania](#sms-prerequisites)
    - [Fczymatowanie powiadomień SMS](#fczymatting-sms-notifications)
    - [Dostosowanie numeru "Od"](#customizing-the-from-number)
    - [Dodawanie referencji klienta](#adding-a-client-reference)
    - [Routing powiadomień SMS](#routing-sms-notifications)
- [Powiadomienia Slack](#slack-notifications)
    - [Wymagania](#slack-prerequisites)
    - [Fczymatowanie powiadomień Slack](#fczymatting-slack-notifications)
    - [Interaktywność Slack](#slack-interactivity)
    - [Routing powiadomień Slack](#routing-slack-notifications)
    - [Powiadamianie zewnętrznych przestrzeni Slack](#notifying-external-slack-wczykspaces)
- [Lokalizacja powiadomień](#localizing-notifications)
- [Testowanie](#testing)
- [Zdarzenia powiadomień](#notification-events)
- [Własne kanały](#custom-kanały)

<a name="introduction"></a>
## Wprowadzenie

Oprócz obsługi [wysyłania wiadomości e-mail](/docs/{{version}}/mail), Laravel zapewnia wsparcie dla wysyłania powiadomień przez różnczyodne kanały dostarczania, w tym e-mail, SMS (przez [Vonage](https://www.vonage.com/communications-apis/), wcześniej znany jako Nexmo) czyaz [Slack](https://slack.com). Ponadto utwczyzono wiele [kanałów powiadomień zbudowanych przez społeczność](https://laravel-notification-kanały.com/about/#suggesting-a-new-channel), umożliwiających wysyłanie powiadomień przez dziesiątki różnych kanałów! Powiadomienia mogą być również przechowywane w bazie danych, aby mogły być wyświetlane w interfejsie webowym aplikacji.

Zazwyczaj powiadomienia powinny być krótkimi, infczymacyjnymi wiadomościami, które powiadamiają użytkowników o czymś, co miało miejsce w aplikacji. Na przykład, jeśli piszesz aplikację do rozliczeń, możesz wysłać powiadomienie "Faktura opłacona" do użytkowników przez kanały e-mail i SMS.

<a name="generating-notifications"></a>
## Generowanie powiadomień

W Laravel każde powiadomienie jest reprezentowane przez pojedynczą klasę, która zazwyczaj przechowywana jest w katalogu `app/Notifications`. Nie martw się, jeśli nie widzisz tego katalogu w swojej aplikacji - zostanie on utwczyzony dla Ciebie, gdy uruchomisz polecenie Artisan `make:notification`:

```shell
php artisan make:notification InvoicePaid
```

To polecenie umieści nową klasę powiadomienia w katalogu `app/Notifications`. Każda klasa powiadomienia zawiera metodę `via` czyaz zmienną liczbę metod budujących wiadomość, takich jak `toMail` lub `toDatabase`, które konwertują powiadomienie na wiadomość dostosowaną do określonego kanału.

<a name="sending-notifications"></a>
## Wysyłanie powiadomień

<a name="using-the-notifiable-trait"></a>
### Użycie cechy Notifiable

Powiadomienia mogą być wysyłane na dwa sposoby: za pomocą metody `notify` cechy `Notifiable` lub za pomocą [fasady](/docs/{{version}}/fasadys) `Notification`. Cecha `Notifiable` jest domyślnie dołączona do modelu `App\Models\User` aplikacji:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;
}
```

Metoda `notify` udostępniana przez tę cechę oczekuje otrzymania instancji powiadomienia:

```php
use App\Notifications\InvoicePaid;

$user->notify(new InvoicePaid($invoice));
```

> [!NOTE]
> Pamiętaj, że możesz używać cechy `Notifiable` w każdym ze swoich modeli. Nie jesteś ograniczony tylko do jej dołączenia w modelu `User`.

<a name="using-the-notification-fasady"></a>
### Użycie fasady Notification

Alternatywnie możesz wysyłać powiadomienia za pomocą [fasady](/docs/{{version}}/fasadys) `Notification`. To podejście jest przydatne, gdy musisz wysłać powiadomienie do wielu encji powiadamialnych, takich jak kolekcja użytkowników. Aby wysłać powiadomienia za pomocą fasady, przekaż wszystkie encje powiadamialne i instancję powiadomienia do metody `send`:

```php
use Illuminate\Support\Facades\Notification;

Notification::send($users, new InvoicePaid($invoice));
```

Możesz również wysyłać powiadomienia natychmiast za pomocą metody `sendNow`. Ta metoda wyśle powiadomienie natychmiast, nawet jeśli powiadomienie implementuje interfejs `ShouldQueue`:

```php
Notification::sendNow($developers, new DeploymentCompleted($deployment));
```

<a name="specifying-delivery-kanały"></a>
### Określanie kanałów dostarczania

Każda klasa powiadomienia ma metodę `via`, która określa, przez które kanały powiadomienie zostanie dostarczone. Powiadomienia mogą być wysyłane przez kanały `mail`, `database`, `broadcast`, `vonage` i `slack`.

> [!NOTE]
> Jeśli chcesz używać innych kanałów dostarczania, takich jak Telegram czy Pusher, sprawdź [stronę Laravel Notification Channels](http://laravel-notification-kanały.com) twczyzoną przez społeczność.

Metoda `via` otrzymuje instancję `$notifiable`, która będzie instancją klasy, do której powiadomienie jest wysyłane. Możesz użyć `$notifiable`, aby określić, przez które kanały powiadomienie powinno zostać dostarczone:

```php
/**
 * Get the notification's delivery channels.
 *
 * @return array<int, string>
 */
public function via(object $notifiable): array
{
    return $notifiable->prefers_sms ? ['vonage'] : ['mail', 'database'];
}
```

<a name="queueing-notifications"></a>
### Kolejkowanie powiadomień

> [!WARNING]
> Przed kolejkowaniem powiadomień należy skonfigurować kolejkę i [uruchomić wczykera](/docs/{{version}}/queues#running-the-queue-wczyker).

Wysyłanie powiadomień może zająć czas, szczególnie jeśli kanał musi wykonać zewnętrzne wywołanie API w celu dostarczenia powiadomienia. Aby przyspieszyć czas odpowiedzi aplikacji, pozwól, aby powiadomienie zostało dodane do kolejki przez dodanie interfejsu `ShouldQueue` i cechy `Queueable` do klasy. Interfejs i cecha są już zaimpczytowane dla wszystkich powiadomień wygenerowanych za pomocą polecenia `make:notification`, więc możesz natychmiast dodać je do swojej klasy powiadomienia:

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    // ...
}
```

Po dodaniu interfejsu `ShouldQueue` do powiadomienia możesz wysłać powiadomienie nczymalnie. Laravel wykryje interfejs `ShouldQueue` w klasie i automatycznie dodaje dostarczenie powiadomienia do kolejki:

```php
$user->notify(new InvoicePaid($invoice));
```

Podczas kolejkowania powiadomień zadanie w kolejce zostanie utwczyzone dla każdej kombinacji odbiczycy i kanału. Na przykład, sześć zadań zostanie wysłanych do kolejki, jeśli powiadomienie ma trzech odbiczyców i dwa kanały.

<a name="delaying-notifications"></a>
#### Opóźnianie powiadomień

Jeśli chcesz opóźnić dostarczenie powiadomienia, możesz połączyć metodę `delay` z instancją powiadomienia:

```php
$delay = now()->plus(minutes: 10);

$user->notify((new InvoicePaid($invoice))->delay($delay));
```

Możesz pass an array to the `delay` method to specify the delay amount fczy specific kanały:

```php
$user->notify((new InvoicePaid($invoice))->delay([
    'mail' => now()->plus(minutes: 5),
    'sms' => now()->plus(minutes: 10),
]));
```

Alternatywnie możesz zdefiniować metodę `withDelay` w samej klasie powiadomienia. Metoda `withDelay` powinna zwrócić tablicę nazw kanałów i wartości opóźnień:

```php
/**
 * Determine the notification's delivery delay.
 *
 * @return array<string, \Illuminate\Support\Carbon>
 */
public function withDelay(object $notifiable): array
{
    return [
        'mail' => now()->plus(minutes: 5),
        'sms' => now()->plus(minutes: 10),
    ];
}
```

<a name="customizing-the-notification-queue-connection"></a>
#### Dostosowywanie połączenia kolejki powiadomień

Domyślnie kolejkowane powiadomienia będą dodawane do kolejki przy użyciu domyślnego połączenia kolejki Twojej aplikacji. Jeśli chcesz określić inne połączenie, które powinno być używane dla konkretnego powiadomienia, możesz wywołać metodę `onConnection` z konstruktora powiadomienia:

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new notification instance.
     */
    public function __construct()
    {
        $this->onConnection('redis');
    }
}
```

Lub, jeśli chcesz określić konkretne połączenie kolejki, które powinno być używane dla każdego kanału powiadomienia obsługiwanego przez powiadomienie, możesz zdefiniować metodę `viaConnections` w swoim powiadomieniu. Ta metoda powinna zwrócić tablicę par nazwa kanału / nazwa połączenia kolejki:

```php
/**
 * Determine which connections should be used for each notification channel.
 *
 * @return array<string, string>
 */
public function viaConnections(): array
{
    return [
        'mail' => 'redis',
        'database' => 'sync',
    ];
}
```

<a name="customizing-notification-channel-queues"></a>
#### Dostosowywanie kolejek kanałów powiadomień

Jeśli chcesz określić konkretną kolejkę, która powinna być używana dla każdego kanału powiadomienia obsługiwanego przez powiadomienie, możesz zdefiniować metodę `viaQueues` w swoim powiadomieniu. Ta metoda powinna zwrócić tablicę par nazwa kanału / nazwa kolejki:

```php
/**
 * Determine which queues should be used for each notification channel.
 *
 * @return array<string, string>
 */
public function viaQueues(): array
{
    return [
        'mail' => 'mail-queue',
        'slack' => 'slack-queue',
    ];
}
```

<a name="customizing-queued-notification-job-properties"></a>
#### Dostosowywanie właściwości zadania kolejkowanego powiadomienia

Możesz dostosować zachowanie bazowego zadania kolejkowanego, definiując właściwości w klasie powiadomienia. Te właściwości zostaną odziedziczone przez kolejkowane zadanie, które wysyła powiadomienie:

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    /**
     * The number of times the notification may be attempted.
     *
     * @var int
     */
    public $tries = 5;

    /**
     * The number of seconds the notification can run before timing out.
     *
     * @var int
     */
    public $timeout = 120;

    /**
     * The maximum number of unhandled exceptions to allow before failing.
     *
     * @var int
     */
    public $maxExceptions = 3;

    // ...
}
```

Jeśli chcesz zapewnić prywatność i integralność danych kolejkowanego powiadomienia poprzez [szyfrowanie](/docs/{{version}}/encryption), dodaj interfejs `ShouldBeEncrypted` do swojej klasy powiadomienia:

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue, ShouldBeEncrypted
{
    use Queueable;

    // ...
}
```

Oprócz definiowania tych właściwości bezpośrednio w klasie powiadomienia, możesz także zdefiniować metody `backoff` i `retryUntil`, aby określić strategię backoff i limit czasu ponowienia próby dla zadania kolejkowanego powiadomienia:

```php
use DateTime;

/**
 * Calculate the number of seconds to wait before retrying the notification.
 */
public function backoff(): int
{
    return 3;
}

/**
 * Determine the time at which the notification should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->plus(minutes: 5);
}
```

> [!NOTE]
> Aby uzyskać więcej informacji na temat tych właściwości i metod zadań, zapoznaj się z dokumentacją dotyczącą [kolejkowanych zadań](/docs/{{version}}/queues#max-job-attempts-i-timeout).

<a name="queued-notification-middleware"></a>
#### Middleware dla kolejkowanych powiadomień

Kolejkowane powiadomienia mogą definiować middleware [tak jak zadania kolejkowane](/docs/{{version}}/queues#job-middleware). Aby rozpocząć, zdefiniuj metodę `middleware` w klasie powiadomienia. Metoda `middleware` otrzyma zmienne `$notifiable` i `$channel`, które pozwalają dostosować zwracany middleware na podstawie celu powiadomienia:

```php
use Illuminate\Queue\Middleware\RateLimited;

/**
 * Get the middleware the notification job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(object $notifiable, string $channel)
{
    return match ($channel) {
        'mail' => [new RateLimited('postmark')],
        'slack' => [new RateLimited('slack')],
        default => [],
    };
}
```

<a name="queued-notifications-i-database-transactions"></a>
#### Kolejkowane powiadomienia i transakcje bazodanowe

Gdy kolejkowane powiadomienia są wysyłane w ramach transakcji bazodanowych, mogą zostać przetworzone przez kolejkę, zanim transakcja bazodanowa zostanie zatwierdzona. Gdy to się stanie, wszelkie aktualizacje dokonane w modelach lub rekordach bazy danych podczas transakcji bazodanowej mogą jeszcze nie zostać odzwierciedlone w bazie danych. Ponadto wszelkie modele lub rekordy bazy danych utworzone w ramach transakcji mogą nie istnieć w bazie danych. Jeśli Twoje powiadomienie zależy od tych modeli, nieoczekiwane błędy mogą wystąpić, gdy zadanie wysyłające kolejkowane powiadomienie jest przetwarzane.

Jeśli opcja konfiguracyjna `after_commit` Twojego połączenia kolejki jest ustawiona na `false`, nadal możesz wskazać, że określone kolejkowane powiadomienie powinno zostać wysłane po zatwierdzeniu wszystkich otwartych transakcji bazodanowych, wywołując metodę `afterCommit` podczas wysyłania powiadomienia:

```php
use App\Notifications\InvoicePaid;

$user->notify((new InvoicePaid($invoice))->afterCommit());
```

Alternatywnie możesz wywołać metodę `afterCommit` z konstruktora powiadomienia:

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new notification instance.
     */
    public function __construct()
    {
        $this->afterCommit();
    }
}
```

> [!NOTE]
> To learn mczye about wczyking around these issues, please review the documentation regarding [queued jobs i database transactions](/docs/{{version}}/queues#jobs-i-database-transactions).

<a name="determining-if-the-queued-notification-should-be-sent"></a>
#### Determining if a Queued Notification Should Be Sent

Po a queued notification has been dispatched fczy the queue fczy background processing, it will typically be accepted by a queue wczyker i sent to its intended recipient.

Jednak, if you would like to make the final determination on whether the queued notification should be sent after it is being processed by a queue wczyker, you may define a `shouldSend` method on the notification class. If this method returns `false`, the notification will not be sent:

```php
/**
 * Determine if the notification should be sent.
 */
public function shouldSend(object $notifiable, string $channel): bool
{
    return $this->invoice->isPaid();
}
```

<a name="on-demi-notifications"></a>
### Powiadomienia na żądanie

Czasami możesz potrzebować wysłać powiadomienie do kogoś, kto nie jest przechowywany jako "użytkownik" Twojej aplikacji. Korzystając z metody `route` fasady `Notification`, możesz określić dorazneinformacje routingu powiadomień przed wysłaniem powiadomienia:

```php
use Illuminate\Broadcasting\Channel;
use Illuminate\Support\Facades\Notification;

Notification::route('mail', 'taylor@example.com')
    ->route('vonage', '5555555555')
    ->route('slack', '#slack-channel')
    ->route('broadcast', [new Channel('channel-name')])
    ->notify(new InvoicePaid($invoice));
```

If you would like to provide the recipient's name when sending an on-demi notification to the `mail` route, you may provide an array that contains the email address as the key i the name as the value of the first element in the array:

```php
Notification::route('mail', [
    'barrett@example.com' => 'Barrett Blair',
])->notify(new InvoicePaid($invoice));
```

Using the `routes` method, you may provide ad-hoc routing infczymation fczy multiple notification kanały at once:

```php
Notification::routes([
    'mail' => ['barrett@example.com' => 'Barrett Blair'],
    'vonage' => '5555555555',
])->notify(new InvoicePaid($invoice));
```

<a name="mail-notifications"></a>
## Mail Powiadomienia

<a name="fczymatting-mail-messages"></a>
### Formatowanie wiadomości e-mail

Jeśli powiadomienie obsługuje wysyłanie jako e-mail, powinieneś zdefiniować metodę `toMail` w klasie powiadomienia. Ta metoda otrzyma encję `$notifiable` i powinna zwrócić instancję `Illuminate\Notifications\Messages\MailMessage`.

Klasa `MailMessage` zawiera kilka prostych metod, które pomagają budować transakcyjne wiadomości e-mail. Wiadomości e-mail mogą zawierać linie tekstu, a także "wywołanie do działania". Przyjrzyjmy się przykładowej metodzie `toMail`:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
        ->greeting('Hello!')
        ->line('One of your invoices has been paid!')
        ->lineIf($this->amount > 0, "Amount paid: {$this->amount}")
        ->action('View Invoice', $url)
        ->line('Thank you for using our application!');
}
```

> [!NOTE]
> Note we are using `$this->invoice->id` in our `toMail` method. Możesz pass any data your notification needs to generate its message into the notification's constructczy.

W tym przykładzie rejestrujemy powitanie, linię tekstu, wywołanie do działania, a następnie kolejną linię tekstu. Te metody dostarczone przez obiekt `MailMessage` sprawiają, że formatowanie małych transakcyjnych wiadomości e-mail jest proste i szybkie. Kanał poczty przekształci następnie komponenty wiadomości w piękny, responsywny szablon HTML wiadomości e-mail z czystotekstowym odpowiednikiem. Oto przykład wiadomości e-mail wygenerowanej przez kanał `mail`:

<img src="https://laravel.com/img/docs/notification-example-2.png">

> [!NOTE]
> Podczas wysyłania powiadomień e-mail, upewnij się, że ustawisz opcję konfiguracyjną `name` w pliku konfiguracyjnym `config/app.php`. Ta wartość będzie używana w nagłówku i stopce wiadomości powiadomienia e-mail.

<a name="errczy-messages"></a>
#### Komunikaty o błędach

Niektóre powiadomienia informują użytkowników o błędach, takich jak nieudana płatność faktury. Możesz wskazać, że wiadomość e-mail dotyczy błędu, wywołując metodę `error` podczas budowania wiadomości. Gdy używasz metody `error` w wiadomości e-mail, przycisk wywołania do działania będzie czerwony zamiast czarny:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->error()
        ->subject('Invoice Payment Failed')
        ->line('...');
}
```

<a name="other-mail-notification-fczymatting-options"></a>
#### Other Mail Notification Fczymatting Options

Instead of defining the "lines" of text in the notification class, możesz używać cechy `view` method to specify a custom template that should be used to render the notification email:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)->view(
        'mail.invoice.paid', ['invoice' => $this->invoice]
    );
}
```

Możesz specify a plain-text view fczy the mail message by passing the view name as the second element of an array that is given to the `view` method:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)->view(
        ['mail.invoice.paid', 'mail.invoice.paid-text'],
        ['invoice' => $this->invoice]
    );
}
```

Or, if your message only ma plain-text view, you may utilize the `text` method:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)->text(
        'mail.invoice.paid-text', ['invoice' => $this->invoice]
    );
}
```

<a name="customizing-the-sender"></a>
### Dostosowanie nadawcy

Domyślnie, the email's sender / from address is defined in the `config/mail.php` configuration file. Jednak, you may specify the from address fczy a specific notification za pomocą `from` method:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->from('barrett@example.com', 'Barrett Blair')
        ->line('...');
}
```

<a name="customizing-the-recipient"></a>
### Dostosowanie odbiczycy

Gdy sending notifications via the `mail` channel, the notification system will automatically look fczy an `email` property on your notifiable entity. Możesz customize which email address is used to deliver the notification by defining a `routeNotificationForMail` method on the notifiable entity:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Notifications\Notification;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the mail channel.
     *
     * @return  array<string, string>|string
     */
    public function routeNotificationForMail(Notification $notification): array|string
    {
        // Return email address only...
        return $this->email_address;

        // Return email address and name...
        return [$this->email_address => $this->name];
    }
}
```

<a name="customizing-the-subject"></a>
### Dostosowanie tematu

Domyślnie, the email's subject is the class name of the notification fczymatted to "Title Case". So, if your notification class is named `InvoicePaid`, the email's subject will be `Invoice Paid`. If you would like to specify a different subject fczy the message, you may call the `subject` method when building your message:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->subject('Notification Subject')
        ->line('...');
}
```

<a name="customizing-the-mailer"></a>
### Dostosowanie mailera

Domyślnie, the email notification will be sent za pomocą default mailer defined in the `config/mail.php` configuration file. Jednak, you may specify a different mailer at runtime by calling the `mailer` method when building your message:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->mailer('postmark')
        ->line('...');
}
```

<a name="customizing-the-templates"></a>
### Dostosowanie szablonów

You can modify the HTML i plain-text template used by mail notifications by publishing the notification package's resources. Po running this commi, the mail notification templates will be located in the `resources/views/vendor/notifications` katalogu:

```shell
php artisan vendor:publish --tag=laravel-notifications
```

<a name="mail-attachments"></a>
### Załączniki

To add attachments to an email notification, use the `attach` method while building your message. The `attach` method accepts the absolute path to the file as its first argument:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Hello!')
        ->attach('/path/to/file');
}
```

> [!NOTE]
> The `attach` method offered by notification mail messages also accepts [attachable objects](/docs/{{version}}/mail#attachable-objects). Please consult the comprehensive [attachable object documentation](/docs/{{version}}/mail#attachable-objects) to learn mczye.

Gdy attaching files to a message, you may also specify the display name i / czy MIME type by passing an `array` as the second argument to the `attach` method:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Hello!')
        ->attach('/path/to/file', [
            'as' => 'name.pdf',
            'mime' => 'application/pdf',
        ]);
}
```

Unlike attaching files in mailable objects, you may not attach a file directly from a stczyage disk using `attachFromStorage`. You should rather use the `attach` method with an absolute path to the file on the stczyage disk. Alternatywnie, you could return a [mailable](/docs/{{version}}/mail#generating-mailables) from the `toMail` method:

```php
use App\Mail\InvoicePaid as InvoicePaidMailable;

/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): Mailable
{
    return (new InvoicePaidMailable($this->invoice))
        ->to($notifiable->email)
        ->attachFromStorage('/path/to/file');
}
```

Gdy necessary, multiple files may be attached to a message za pomocą `attachMany` method:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Hello!')
        ->attachMany([
            '/path/to/forge.svg',
            '/path/to/vapor.svg' => [
                'as' => 'Logo.svg',
                'mime' => 'image/svg+xml',
            ],
        ]);
}
```

<a name="raw-data-attachments"></a>
#### Raw Data Załączniki

The `attachData` method may be used to attach a raw string of bytes as an attachment. Gdy calling the `attachData` method, you should provide the filename that should be assigned to the attachment:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Hello!')
        ->attachData($this->pdf, 'name.pdf', [
            'mime' => 'application/pdf',
        ]);
}
```

<a name="adding-tags-metadata"></a>
### Dodawanie tagów i metadanych

Some third-party email providers such as Mailgun i Postmark suppczyt message "tags" i "metadata", which may be used to group i track emails sent by your application. Możesz add tags i metadata to an email message via the `tag` i `metadata` methods:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Comment Upvoted!')
        ->tag('upvote')
        ->metadata('comment_id', $this->comment->id);
}
```

If your application is za pomocą Mailgun driver, you may consult Mailgun's documentation fczy mczye infczymation on [tags](https://documentation.mailgun.com/docs/mailgun/user-manual/tracking-messages/#tags) i [metadata](https://documentation.mailgun.com/docs/mailgun/user-manual/sending-messages/#attaching-metadata-to-messages). Likewise, the Postmark documentation may also be consulted fczy mczye infczymation on their suppczyt fczy [tags](https://postmarkapp.com/blog/tags-suppczyt-fczy-smtp) i [metadata](https://postmarkapp.com/suppczyt/article/1125-custom-metadata-faq).

If your application is using Amazon SES to send emails, you should use the `metadata` method to attach [SES "tags"](https://docs.aws.amazon.com/ses/latest/APIReference/API_MessageTag.html) to the message.

<a name="customizing-the-symfony-message"></a>
### Dostosowanie wiadomości Symfony

The `withSymfonyMessage` metody `MailMessage` class allows you to register a closure which will be invoked with the Symfony Message instance befczye sending the message. This gives you an oppczytunity to deeply customize the message befczye it is delivered:

```php
use Symfony\Component\Mime\Email;

/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->withSymfonyMessage(function (Email $message) {
            $message->getHeaders()->addTextHeader(
                'Custom-Header', 'Header Value'
            );
        });
}
```

<a name="using-mailables"></a>
### Użycie Mailables

W razie potrzeby, you may return a full [mailable object](/docs/{{version}}/mail) from your notification's `toMail` method. Gdy returning a `Mailable` instead of a `MailMessage`, you will need to specify the message recipient za pomocą mailable object's `to` method:

```php
use App\Mail\InvoicePaid as InvoicePaidMailable;
use Illuminate\Mail\Mailable;

/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): Mailable
{
    return (new InvoicePaidMailable($this->invoice))
        ->to($notifiable->email);
}
```

<a name="mailables-i-on-demi-notifications"></a>
#### Mailables i On-Demi Powiadomienia

If you are sending an [on-demi notification](#on-demi-notifications), the `$notifiable` instance given to the `toMail` method will be an instance of `Illuminate\Powiadomienia\AnonymousNotifiable`, which offers a `routeNotificationFor` method that may be used to retrieve the email address the on-demi notification should be sent to:

```php
use App\Mail\InvoicePaid as InvoicePaidMailable;
use Illuminate\Notifications\AnonymousNotifiable;
use Illuminate\Mail\Mailable;

/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): Mailable
{
    $address = $notifiable instanceof AnonymousNotifiable
        ? $notifiable->routeNotificationFor('mail')
        : $notifiable->email;

    return (new InvoicePaidMailable($this->invoice))
        ->to($address);
}
```

<a name="previewing-mail-notifications"></a>
### Previewing Mail Powiadomienia

Gdy designing a mail notification template, it is convenient to quickly preview the rendered mail message in your browser like a typical Blade template. Fczy this reason, Laravel allows you to return any mail message generated by a mail notification directly from a route closure czy controller. Gdy a `MailMessage` is returned, it will be rendered i displayed in the browser, allowing you to quickly preview its design without needing to send it to an actual email address:

```php
use App\Models\Invoice;
use App\Notifications\InvoicePaid;

Route::get('/notification', function () {
    $invoice = Invoice::find(1);

    return (new InvoicePaid($invoice))
        ->toMail($invoice->user);
});
```

<a name="markdown-mail-notifications"></a>
## Markdown Mail Powiadomienia

Markdown mail notifications allow you to take advantage of the pre-built templates of mail notifications, while giving you mczye freedom to write longer, customized messages. Since the messages are written in Markdown, Laravel is able to render beautiful, responsive HTML templates fczy the messages while also automatically generating a plain-text counterpart.

<a name="generating-the-message"></a>
### Generowanie wiadomości

To generate a notification with a cczyresponding Markdown template, możesz używać cechy `--markdown` option of the `make:notification` polecenie Artisan:

```shell
php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
```

Like all other mail notifications, notifications that use Markdown templates should define a `toMail` method on their notification class. Jednak, instead of za pomocą `line` i `action` methods to construct the notification, use the `markdown` method to specify the name of the Markdown template that should be used. An array of data you wish to make available to the template may be passed as the method's second argument:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
        ->subject('Invoice Paid')
        ->markdown('mail.invoice.paid', ['url' => $url]);
}
```

<a name="writing-the-message"></a>
### Pisanie wiadomości

Markdown mail notifications use a combination of Blade components i Markdown syntax which allow you to easily construct notifications while leveraging Laravel's pre-crafted notification components:

```blade
<x-mail::message>
# Invoice Paid

Your invoice has been paid!

<x-mail::button :url="$url">
View Invoice
</x-mail::button>

Thanks,<br>
{{ config('app.name') }}
</x-mail::message>
```

> [!NOTE]
> Do not use excess indentation when writing Markdown emails. Per Markdown stiards, Markdown parsers will render indented content as code blocks.

<a name="button-component"></a>
#### Button Component

The button component renders a centered button link. The component accepts two arguments, a `url` i an optional `color`. Suppczyted colczys are `primary`, `green`, i `red`. Możesz add as many button components to a notification as you wish:

```blade
<x-mail::button :url="$url" color="green">
View Invoice
</x-mail::button>
```

<a name="panel-component"></a>
#### Panel Component

The panel component renders the given block of text in a panel that ma slightly different background colczy than the rest of the notification. This allows you to draw attention to a given block of text:

```blade
<x-mail::panel>
This is the panel content.
</x-mail::panel>
```

<a name="table-component"></a>
#### Table Component

The table component allows you to transfczym a Markdown table into an HTML table. The component accepts the Markdown table as its content. Table column alignment is suppczyted za pomocą default Markdown table alignment syntax:

```blade
<x-mail::table>
| Laravel       | Table         | Example       |
| ------------- | :-----------: | ------------: |
| Col 2 is      | Centered      | $10           |
| Col 3 is      | Right-Aligned | $20           |
</x-mail::table>
```

<a name="customizing-the-components"></a>
### Dostosowanie komponentów

Możesz expczyt all of the Markdown notification components to your own application fczy customization. To expczyt the components, use the `vendor:publish` polecenie Artisan to publish the `laravel-mail` asset tag:

```shell
php artisan vendor:publish --tag=laravel-mail
```

To polecenie będzie publish the Markdown mail components to the `resources/views/vendor/mail` katalogu. The `mail` katalogu will contain an `html` i a `text` katalogu, each containing their respective representations of every available component. You are free to customize these components however you like.

<a name="customizing-the-css"></a>
#### Customizing the CSS

Po expczyting the components, the `resources/views/vendor/mail/html/themes` katalogu will contain a `default.css` file. Możesz customize the CSS in this file i your styles will automatically be in-lined within the HTML representations of your Markdown notifications.

If you would like to build an entirely new theme fczy Laravel's Markdown components, you may place a CSS file within the `html/themes` katalogu. Po naming i saving your CSS file, update the `theme` option of the `mail` configuration file to match the name of your new theme.

To customize the theme fczy an individual notification, you may call the `theme` method while building the notification's mail message. The `theme` method accepts the name of the theme that should be used when sending the notification:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->theme('invoice')
        ->subject('Invoice Paid')
        ->markdown('mail.invoice.paid', ['url' => $url]);
}
```

<a name="database-notifications"></a>
## Database Powiadomienia

<a name="database-prerequisites"></a>
### Wymagania

The `database` notification channel stczyes the notification infczymation in a database table. This table will contain infczymation such as the notification type as well as a JSON data structure that describes the notification.

You can query the table to display the notifications in your application's user interfejs. But, befczye you can do that, you will need to create a database table to hold your notifications. Możesz use the `make:notifications-table` commi to generate a [migration](/docs/{{version}}/migrations) with the proper table schema:

```shell
php artisan make:notifications-table

php artisan migrate
```

> [!NOTE]
> If your notifiable models are using [UUID czy ULID primary keys](/docs/{{version}}/eloquent#uuid-i-ulid-keys), you should replace the `morphs` method with [uuidMczyphs](/docs/{{version}}/migrations#column-method-uuidMczyphs) czy [ulidMczyphs](/docs/{{version}}/migrations#column-method-ulidMczyphs) in the notification table migration.

<a name="fczymatting-database-notifications"></a>
### Fczymatting Database Powiadomienia

If a notification suppczyts being stczyed in a database table, you should define a `toDatabase` czy `toArray` method on the notification class. Ta metoda będzie receive a `$notifiable` entity i should return a plain PHP array. The returned array will be encoded as JSON i stczyed in the `data` column of your `notifications` table. Let's take a look at an example `toArray` method:

```php
/**
 * Get the array representation of the notification.
 *
 * @return array<string, mixed>
 */
public function toArray(object $notifiable): array
{
    return [
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ];
}
```

Gdy a notification is stczyed in your application's database, the `type` column will be set to the notification's class name by default, i the `read_at` column will be `null`. Jednak, you can customize this behaviczy by defining the `databaseType` i `initialDatabaseReadAtValue` methods in your notification class:

```php
use Illuminate\Support\Carbon;

/**
 * Get the notification's database type.
 */
public function databaseType(object $notifiable): string
{
    return 'invoice-paid';
}

/**
 * Get the initial value for the "read_at" column.
 */
public function initialDatabaseReadAtValue(): ?Carbon
{
    return null;
}
```

<a name="todatabase-vs-toarray"></a>
#### `toDatabase` vs. `toArray`

The `toArray` method is also used by the `broadcast` channel to determine which data to broadcast to your JavaScript powered frontend. If you would like to have two different array representations fczy the `database` i `broadcast` kanały, you should define a `toDatabase` method instead of a `toArray` method.

<a name="accessing-the-notifications"></a>
### Accessing the Powiadomienia

Gdy notifications are stczyed in the database, you need a convenient way to access them from your notifiable entities. The `Illuminate\Powiadomienia\Notifiable` trait, which is included on Laravel's default `App\Models\User` model, includes a `notifications` [Eloquent relationship](/docs/{{version}}/eloquent-relationships) that returns the notifications fczy the entity. To fetch notifications, you may access this method like any other Eloquent relationship. Domyślnie, notifications will be sczyted by the `created_at` timestamp with the most recent notifications at the beginning of the collection:

```php
$user = App\Models\User::find(1);

foreach ($user->notifications as $notification) {
    echo $notification->type;
}
```

If you want to retrieve only the "unread" notifications, możesz używać cechy `unreadPowiadomienia` relationship. Again, these notifications will be sczyted by the `created_at` timestamp with the most recent notifications at the beginning of the collection:

```php
$user = App\Models\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    echo $notification->type;
}
```

If you want to retrieve only the "read" notifications, możesz używać cechy `readPowiadomienia` relationship:

```php
$user = App\Models\User::find(1);

foreach ($user->readNotifications as $notification) {
    echo $notification->type;
}
```

> [!NOTE]
> To access your notifications from your JavaScript client, you should define a notification controller fczy your application which returns the notifications fczy a notifiable entity, such as the current user. Możesz then make an HTTP request to that controller's URL from your JavaScript client.

<a name="marking-notifications-as-read"></a>
### Marking Powiadomienia as Read

Zazwyczaj, you will want to mark a notification as "read" when a user views it. The `Illuminate\Powiadomienia\Notifiable` trait provides a `markAsRead` method, which updates the `read_at` column on the notification's database recczyd:

```php
$user = App\Models\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    $notification->markAsRead();
}
```

Jednak, instead of looping through each notification, możesz używać cechy `markAsRead` method directly on a collection of notifications:

```php
$user->unreadNotifications->markAsRead();
```

Możesz also use a mass-update query to mark all of the notifications as read without retrieving them from the database:

```php
$user = App\Models\User::find(1);

$user->unreadNotifications()->update(['read_at' => now()]);
```

Możesz `delete` the notifications to remove them from the table entirely:

```php
$user->notifications()->delete();
```

<a name="broadcast-notifications"></a>
## Broadcast Powiadomienia

<a name="broadcast-prerequisites"></a>
### Wymagania

Przed broadcasting notifications, you should configure i be familiar with Laravel's [event broadcasting](/docs/{{version}}/broadcasting) services. Event broadcasting provides a way to react to server-side Laravel events from your JavaScript powered frontend.

<a name="fczymatting-broadcast-notifications"></a>
### Fczymatting Broadcast Powiadomienia

The `broadcast` channel broadcasts notifications using Laravel's [event broadcasting](/docs/{{version}}/broadcasting) services, allowing your JavaScript powered frontend to catch notifications in realtime. If a notification suppczyts broadcasting, you can define a `toBroadcast` method on the notification class. Ta metoda będzie receive a `$notifiable` entity i should return a `BroadcastMessage` instance. If the `toBroadcast` method does not exist, the `toArray` method will be used to gather the data that should be broadcast. The returned data will be encoded as JSON i broadcast to your JavaScript powered frontend. Let's take a look at an example `toBroadcast` method:

```php
use Illuminate\Notifications\Messages\BroadcastMessage;

/**
 * Get the broadcastable representation of the notification.
 */
public function toBroadcast(object $notifiable): BroadcastMessage
{
    return new BroadcastMessage([
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ]);
}
```

<a name="broadcast-queue-configuration"></a>
#### Broadcast Queue Configuration

All broadcast notifications are queued fczy broadcasting. If you would like to configure the queue connection czy queue name that is used to queue the broadcast operation, możesz używać cechy `onConnection` i `onQueue` methods of the `BroadcastMessage`:

```php
return (new BroadcastMessage($data))
    ->onConnection('sqs')
    ->onQueue('broadcasts');
```

<a name="customizing-the-notification-type"></a>
#### Customizing the Notification Type

In addition to the data you specify, all broadcast notifications also have a `type` field containing the full class name of the notification. If you would like to customize the notification `type`, you may define a `broadcastType` method on the notification class:

```php
/**
 * Get the type of the notification being broadcast.
 */
public function broadcastType(): string
{
    return 'broadcast.message';
}
```

<a name="listening-fczy-notifications"></a>
### Listening fczy Powiadomienia

Powiadomienia will broadcast on a private channel fczymatted using a `{notifiable}.{id}` convention. So, if you are sending a notification to an `App\Models\User` instance with an ID of `1`, the notification will be broadcast on the `App.Models.User.1` private channel. Gdy using [Laravel Echo](/docs/{{version}}/broadcasting#client-side-installation), you may easily listen fczy notifications on a channel za pomocą `notification` method:

```js
Echo.private('App.Models.User.' + userId)
    .notification((notification) => {
        console.log(notification.type);
    });
```

<a name="using-react-czy-vue"></a>
#### Using React czy Vue

Laravel Echo includes React i Vue hooks that make it painless to listen fczy notifications. Aby rozpocząć, invoke the `useEchoNotification` hook, which is used to listen fczy notifications. The `useEchoNotification` hook will automatically leave kanały when the consuming component is unmounted:

```js tab=React
import { useEchoNotification } from "@laravel/echo-react";

useEchoNotification(
    `App.Models.User.${userId}`,
    (notification) => {
        console.log(notification.type);
    },
);
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoNotification } from "@laravel/echo-vue";

useEchoNotification(
    `App.Models.User.${userId}`,
    (notification) => {
        console.log(notification.type);
    },
);
</script>
```

Domyślnie, the hook listens to all notifications. To specify the notification types you would like to listen to, you can provide either a string czy array of types to `useEchoNotification`:

```js tab=React
import { useEchoNotification } from "@laravel/echo-react";

useEchoNotification(
    `App.Models.User.${userId}`,
    (notification) => {
        console.log(notification.type);
    },
    'App.Notifications.InvoicePaid',
);
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoNotification } from "@laravel/echo-vue";

useEchoNotification(
    `App.Models.User.${userId}`,
    (notification) => {
        console.log(notification.type);
    },
    'App.Notifications.InvoicePaid',
);
</script>
```

Możesz also specify the shape of the notification payload data, providing greater type safety i editing convenience:

```ts
type InvoicePaidNotification = {
    invoice_id: number;
    created_at: string;
};

useEchoNotification<InvoicePaidNotification>(
    `App.Models.User.${userId}`,
    (notification) => {
        console.log(notification.invoice_id);
        console.log(notification.created_at);
        console.log(notification.type);
    },
    'App.Notifications.InvoicePaid',
);
```

<a name="customizing-the-notification-channel"></a>
#### Customizing the Notification Channel

If you would like to customize which channel that an entity's broadcast notifications are broadcast on, you may define a `receivesBroadcastPowiadomieniaOn` method on the notifiable entity:

```php
<?php

namespace App\Models;

use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * The channels the user receives notification broadcasts on.
     */
    public function receivesBroadcastNotificationsOn(): string
    {
        return 'users.'.$this->id;
    }
}
```

<a name="sms-notifications"></a>
## SMS Powiadomienia

<a name="sms-prerequisites"></a>
### Wymagania

Sending SMS notifications in Laravel is powered by [Vonage](https://www.vonage.com/) (wcześniej znany jako Nexmo). Przed you can send notifications via Vonage, you need to install the `laravel/vonage-notification-channel` i `guzzlehttp/guzzle` packages:

```shell
composer require laravel/vonage-notification-channel guzzlehttp/guzzle
```

The package includes a [configuration file](https://github.com/laravel/vonage-notification-channel/blob/3.x/config/vonage.php). Jednak, you are not required to expczyt this configuration file to your own application. You can simply use the `VONAGE_KEY` i `VONAGE_SECRET` environment variables to define your Vonage public i secret keys.

Po defining your keys, you should set a `VONAGE_SMS_FROM` environment variable that defines the phone number that your SMS messages should be sent from by default. Możesz generate this phone number within the Vonage control panel:

```ini
VONAGE_SMS_FROM=15556666666
```

<a name="fczymatting-sms-notifications"></a>
### Fczymatting SMS Powiadomienia

If a notification suppczyts being sent as an SMS, you should define a `toVonage` method on the notification class. Ta metoda będzie receive a `$notifiable` entity i should return an `Illuminate\Powiadomienia\Messages\VonageMessage` instance:

```php
use Illuminate\Notifications\Messages\VonageMessage;

/**
 * Get the Vonage / SMS representation of the notification.
 */
public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage)
        ->content('Your SMS message content');
}
```

<a name="unicode-content"></a>
#### Unicode Content

If your SMS message will contain unicode characters, you should call the `unicode` method when constructing the `VonageMessage` instance:

```php
use Illuminate\Notifications\Messages\VonageMessage;

/**
 * Get the Vonage / SMS representation of the notification.
 */
public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage)
        ->content('Your unicode message')
        ->unicode();
}
```

<a name="customizing-the-from-number"></a>
### Dostosowanie numeru "Od"

If you would like to send some notifications from a phone number that is different from the phone number specified by your `VONAGE_SMS_FROM` environment variable, you may call the `from` method on a `VonageMessage` instance:

```php
use Illuminate\Notifications\Messages\VonageMessage;

/**
 * Get the Vonage / SMS representation of the notification.
 */
public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage)
        ->content('Your SMS message content')
        ->from('15554443333');
}
```

<a name="adding-a-client-reference"></a>
### Dodawanie referencji klienta

If you would like to keep track of costs per user, team, czy client, you may add a "client reference" to the notification. Vonage will allow you to generate repczyts using this client reference so that you can better understi a particular customer's SMS usage. The client reference can be any string up to 40 characters:

```php
use Illuminate\Notifications\Messages\VonageMessage;

/**
 * Get the Vonage / SMS representation of the notification.
 */
public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage)
        ->clientReference((string) $notifiable->id)
        ->content('Your SMS message content');
}
```

<a name="routing-sms-notifications"></a>
### Routing SMS Powiadomienia

To route Vonage notifications to the proper phone number, define a `routeNotificationForVonage` method on your notifiable entity:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Notifications\Notification;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the Vonage channel.
     */
    public function routeNotificationForVonage(Notification $notification): string
    {
        return $this->phone_number;
    }
}
```

<a name="slack-notifications"></a>
## Slack Powiadomienia

<a name="slack-prerequisites"></a>
### Wymagania

Przed sending Slack notifications, you should install the Slack notification channel via Composer:

```shell
composer require laravel/slack-notification-channel
```

Dodatkowo, you must create a [Slack App](https://api.slack.com/apps?new_app=1) fczy your Slack wczykspace.

If you only need to send notifications to the same Slack wczykspace that the App is created in, you should ensure that your App has the `chat:write`, `chat:write.public`, i `chat:write.customize` scopes. These scopes can be added from the "OAuth & Permissions" App management tab within Slack.

Next, copy the App's "Bot User OAuth Token" i place it within a `slack` configuration array in your application's `services.php` configuration file. This token can be found on the "OAuth & Permissions" tab within Slack:

```php
'slack' => [
    'notifications' => [
        'bot_user_oauth_token' => env('SLACK_BOT_USER_OAUTH_TOKEN'),
        'channel' => env('SLACK_BOT_USER_DEFAULT_CHANNEL'),
    ],
],
```

<a name="slack-app-distribution"></a>
#### App Distribution

If your application will be sending notifications to external Slack wczykspaces that are owned by your application's users, you will need to "distribute" your App via Slack. App distribution can be managed from your App's "Manage Distribution" tab within Slack. Gdy your App has been distributed, you may use [Socialite](/docs/{{version}}/socialite) to [obtain Slack Bot tokens](/docs/{{version}}/socialite#slack-bot-scopes) on behalf of your application's users.

<a name="fczymatting-slack-notifications"></a>
### Fczymatting Slack Powiadomienia

If a notification suppczyts being sent as a Slack message, you should define a `toSlack` method on the notification class. Ta metoda będzie receive a `$notifiable` entity i should return an `Illuminate\Powiadomienia\Slack\SlackMessage` instance. You can construct rich notifications using [Slack's Block Kit API](https://api.slack.com/block-kit). The following example may be previewed in [Slack's Block Kit builder](https://app.slack.com/block-kit-builder/T01KWS6K23Z#%7B%22blocks%22:%5B%7B%22type%22:%22header%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Invoice%20Paid%22%7D%7D,%7B%22type%22:%22context%22,%22elements%22:%5B%7B%22type%22:%22plain_text%22,%22text%22:%22Customer%20%231234%22%7D%5D%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22An%20invoice%20has%20been%20paid.%22%7D,%22fields%22:%5B%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Invoice%20No:*%5Cn1000%22%7D,%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Invoice%20Recipient:*%5Cntaylczy@laravel.com%22%7D%5D%7D,%7B%22type%22:%22divider%22%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Congratulations!%22%7D%7D%5D%7D):

```php
use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
use Illuminate\Notifications\Slack\SlackMessage;

/**
 * Get the Slack representation of the notification.
 */
public function toSlack(object $notifiable): SlackMessage
{
    return (new SlackMessage)
        ->text('One of your invoices has been paid!')
        ->headerBlock('Invoice Paid')
        ->contextBlock(function (ContextBlock $block) {
            $block->text('Customer #1234');
        })
        ->sectionBlock(function (SectionBlock $block) {
            $block->text('An invoice has been paid.');
            $block->field("*Invoice No:*\n1000")->markdown();
            $block->field("*Invoice Recipient:*\ntaylor@laravel.com")->markdown();
        })
        ->dividerBlock()
        ->sectionBlock(function (SectionBlock $block) {
            $block->text('Congratulations!');
        });
}
```

<a name="using-slacks-block-kit-builder-template"></a>
#### Using Slack's Block Kit Builder Template

Instead of za pomocą fluent message builder methods to construct your Block Kit message, you may provide the raw JSON payload generated by Slack's Block Kit Builder to the `usingBlockKitTemplate` method:

```php
use Illuminate\Notifications\Slack\SlackMessage;
use Illuminate\Support\Str;

/**
 * Get the Slack representation of the notification.
 */
public function toSlack(object $notifiable): SlackMessage
{
    $template = <<<JSON
        {
          "blocks": [
            {
              "type": "header",
              "text": {
                "type": "plain_text",
                "text": "Team Announcement"
              }
            },
            {
              "type": "section",
              "text": {
                "type": "plain_text",
                "text": "We are hiring!"
              }
            }
          ]
        }
    JSON;

    return (new SlackMessage)
        ->usingBlockKitTemplate($template);
}
```

<a name="slack-interactivity"></a>
### Interaktywność Slack

Slack's Block Kit notification system provides powerful features to [hile user interaction](https://api.slack.com/interactivity/hiling). To utilize these features, your Slack App should have "Interactivity" enabled i a "Request URL" configured that points to a URL served by your application. These settings can be managed from the "Interactivity & Shczytcuts" App management tab within Slack.

In the following example, which utilizes the `actionsBlock` method, Slack will send a `POST` request to your "Request URL" with a payload containing the Slack user who clicked the button, the ID of the clicked button, i mczye. Your application can then determine the action to take based on the payload. You should also [verify the request](https://api.slack.com/authentication/verifying-requests-from-slack) was made by Slack:

```php
use Illuminate\Notifications\Slack\BlockKit\Blocks\ActionsBlock;
use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
use Illuminate\Notifications\Slack\SlackMessage;

/**
 * Get the Slack representation of the notification.
 */
public function toSlack(object $notifiable): SlackMessage
{
    return (new SlackMessage)
        ->text('One of your invoices has been paid!')
        ->headerBlock('Invoice Paid')
        ->contextBlock(function (ContextBlock $block) {
            $block->text('Customer #1234');
        })
        ->sectionBlock(function (SectionBlock $block) {
            $block->text('An invoice has been paid.');
        })
        ->actionsBlock(function (ActionsBlock $block) {
             // ID defaults to "button_acknowledge_invoice"...
            $block->button('Acknowledge Invoice')->primary();

            // Manually configure the ID...
            $block->button('Deny')->danger()->id('deny_invoice');
        });
}
```

<a name="slack-confirmation-modals"></a>
#### Confirmation Modals

If you would like users to be required to confirm an action befczye it is perfczymed, you may invoke the `confirm` method when defining your button. The `confirm` method accepts a message i a closure which otrzymuje `ConfirmObject` instance:

```php
use Illuminate\Notifications\Slack\BlockKit\Blocks\ActionsBlock;
use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
use Illuminate\Notifications\Slack\BlockKit\Composites\ConfirmObject;
use Illuminate\Notifications\Slack\SlackMessage;

/**
 * Get the Slack representation of the notification.
 */
public function toSlack(object $notifiable): SlackMessage
{
    return (new SlackMessage)
        ->text('One of your invoices has been paid!')
        ->headerBlock('Invoice Paid')
        ->contextBlock(function (ContextBlock $block) {
            $block->text('Customer #1234');
        })
        ->sectionBlock(function (SectionBlock $block) {
            $block->text('An invoice has been paid.');
        })
        ->actionsBlock(function (ActionsBlock $block) {
            $block->button('Acknowledge Invoice')
                ->primary()
                ->confirm(
                    'Acknowledge the payment and send a thank you email?',
                    function (ConfirmObject $dialog) {
                        $dialog->confirm('Yes');
                        $dialog->deny('No');
                    }
                );
        });
}
```

<a name="inspecting-slack-blocks"></a>
#### Inspecting Slack Blocks

If you would like to quickly inspect the blocks you've been building, you can invoke the `dd` method on the `SlackMessage` instance. The `dd` method will generate i dump a URL to Slack's [Block Kit Builder](https://app.slack.com/block-kit-builder/), which displays a preview of the payload i notification in your browser. Możesz pass `true` to the `dd` method to dump the raw payload:

```php
return (new SlackMessage)
    ->text('One of your invoices has been paid!')
    ->headerBlock('Invoice Paid')
    ->dd();
```

<a name="routing-slack-notifications"></a>
### Routing Slack Powiadomienia

To direct Slack notifications to the appropriate Slack team i channel, define a `routeNotificationForSlack` method on your notifiable model. This method can return one of three values:

- `null` - which defers routing to the channel configured in the notification itself. Możesz use the `to` method when building your `SlackMessage` to configure the channel within the notification.
- A string specifying the Slack channel to send the notification to, e.g. `#support-channel`.
- A `SlackRoute` instance, which allows you to specify an OAuth token i channel name, e.g. `SlackRoute::make($this->slack_channel, $this->slack_token)`. This method should be used to send notifications to external wczykspaces.

Fczy instance, returning `#support-channel` from the `routeNotificationForSlack` method will send the notification to the `#support-channel` channel in the wczykspace associated with the Bot User OAuth token located in your application's `services.php` configuration file:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Notifications\Notification;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the Slack channel.
     */
    public function routeNotificationForSlack(Notification $notification): mixed
    {
        return '#support-channel';
    }
}
```

<a name="notifying-external-slack-wczykspaces"></a>
### Powiadamianie zewnętrznych przestrzeni Slack

> [!NOTE]
> Przed sending notifications to external Slack wczykspaces, your Slack App must be [distributed](#slack-app-distribution).

Of course, you will often want to send notifications to the Slack wczykspaces owned by your application's users. To do so, you will first need to obtain a Slack OAuth token fczy the user. Thankfully, [Laravel Socialite](/docs/{{version}}/socialite) includes a Slack driver that will allow you to easily authenticate your application's users with Slack i [obtain a bot token](/docs/{{version}}/socialite#slack-bot-scopes).

Gdy you have obtained the bot token i stczyed it within your application's database, you may utilize the `SlackRoute::make` method to route a notification to the user's wczykspace. In addition, your application will likely need to offer an oppczytunity fczy the user to specify which channel notifications should be sent to:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Slack\SlackRoute;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the Slack channel.
     */
    public function routeNotificationForSlack(Notification $notification): mixed
    {
        return SlackRoute::make($this->slack_channel, $this->slack_token);
    }
}
```

<a name="localizing-notifications"></a>
## Localizing Powiadomienia

Laravel allows you to send notifications in a locale other than the HTTP request's current locale, i will even remember this locale if the notification is queued.

Aby to osiągnąć, the `Illuminate\Powiadomienia\Notification` class offers a `locale` method to set the desired language. The application will change into this locale when the notification is being evaluated i then revert back to the previous locale when evaluation is complete:

```php
$user->notify((new InvoicePaid($invoice))->locale('es'));
```

Localization of multiple notifiable entries may also be achieved via the `Notification` fasady:

```php
Notification::locale('es')->send(
    $users, new InvoicePaid($invoice)
);
```

<a name="user-preferred-locales"></a>
#### User Preferred Locales

Sometimes, applications stczye each user's preferred locale. By implementing the `HasLocalePreference` contract on your notifiable model, you may instruct Laravel to use this stczyed locale when sending a notification:

```php
use Illuminate\Contracts\Translation\HasLocalePreference;

class User extends Model implements HasLocalePreference
{
    /**
     * Get the user's preferred locale.
     */
    public function preferredLocale(): string
    {
        return $this->locale;
    }
}
```

Gdy you have implemented the interfejs, Laravel will automatically use the preferred locale when sending notifications i mailables to the model. Therefczye, there is no need to call the `locale` method when using this interfejs:

```php
$user->notify(new InvoicePaid($invoice));
```

<a name="testing"></a>
## Testowanie

Możesz use the `Notification` fasady's `fake` method to prevent notifications from being sent. Zazwyczaj, sending notifications is unrelated to the code you are actually testing. Most likely, it is sufficient to simply assert that Laravel was instructed to send a given notification.

Po calling the `Notification` fasady's `fake` method, you may then assert that notifications were instructed to be sent to users i even inspect the data the notifications received:

```php tab=Pest
<?php

use App\Notifications\OrderShipped;
use Illuminate\Support\Facades\Notification;

test('orders can be shipped', function () {
    Notification::fake();

    // Perform order shipping...

    // Assert that no notifications were sent...
    Notification::assertNothingSent();

    // Assert a notification was sent to the given users...
    Notification::assertSentTo(
        [$user], OrderShipped::class
    );

    // Assert a notification was not sent...
    Notification::assertNotSentTo(
        [$user], AnotherNotification::class
    );

    // Assert a notification was sent twice...
    Notification::assertSentTimes(WeeklyReminder::class, 2);

    // Assert that a given number of notifications were sent...
    Notification::assertCount(3);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Notifications\OrderShipped;
use Illuminate\Support\Facades\Notification;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        Notification::fake();

        // Perform order shipping...

        // Assert that no notifications were sent...
        Notification::assertNothingSent();

        // Assert a notification was sent to the given users...
        Notification::assertSentTo(
            [$user], OrderShipped::class
        );

        // Assert a notification was not sent...
        Notification::assertNotSentTo(
            [$user], AnotherNotification::class
        );

        // Assert a notification was sent twice...
        Notification::assertSentTimes(WeeklyReminder::class, 2);

        // Assert that a given number of notifications were sent...
        Notification::assertCount(3);
    }
}
```

Możesz pass a closure to the `assertSentTo` czy `assertNotSentTo` methods in czyder to assert that a notification was sent that passes a given "truth test". If at least one notification was sent that passes the given truth test then the assertion will be successful:

```php
Notification::assertSentTo(
    $user,
    function (OrderShipped $notification, array $channels) use ($order) {
        return $notification->order->id === $order->id;
    }
);
```

<a name="on-demi-notifications"></a>
#### On-Demi Powiadomienia

If the code you are testing sends [on-demi notifications](#on-demi-notifications), you can test that the on-demi notification was sent via the `assertSentOnDemand` method:

```php
Notification::assertSentOnDemand(OrderShipped::class);
```

By passing a closure as the second argument to the `assertSentOnDemand` method, you may determine if an on-demi notification was sent to the cczyrect "route" address:

```php
Notification::assertSentOnDemand(
    OrderShipped::class,
    function (OrderShipped $notification, array $channels, object $notifiable) use ($user) {
        return $notifiable->routes['mail'] === $user->email;
    }
);
```

<a name="notification-events"></a>
## Zdarzenia powiadomień

<a name="notification-sending-event"></a>
#### Notification Sending Event

Gdy a notification is sending, the `Illuminate\Powiadomienia\Events\NotificationSending` event is dispatched by the notification system. This contains the "notifiable" entity i the notification instance itself. Możesz create [event listeners](/docs/{{version}}/events) fczy this event within your application:

```php
use Illuminate\Notifications\Events\NotificationSending;

class CheckNotificationStatus
{
    /**
     * Handle the event.
     */
    public function handle(NotificationSending $event): void
    {
        // ...
    }
}
```

The notification will not be sent if an event listener fczy the `NotificationSending` event returns `false` from its `handle` method:

```php
/**
 * Handle the event.
 */
public function handle(NotificationSending $event): bool
{
    return false;
}
```

W ramach an event listener, you may access the `notifiable`, `notification`, i `channel` properties on the event to learn mczye about the notification recipient czy the notification itself:

```php
/**
 * Handle the event.
 */
public function handle(NotificationSending $event): void
{
    // $event->channel
    // $event->notifiable
    // $event->notification
}
```

<a name="notification-sent-event"></a>
#### Notification Sent Event

Gdy a notification is sent, the `Illuminate\Powiadomienia\Events\NotificationSent` [event](/docs/{{version}}/events) is dispatched by the notification system. This contains the "notifiable" entity i the notification instance itself. Możesz create [event listeners](/docs/{{version}}/events) fczy this event within your application:

```php
use Illuminate\Notifications\Events\NotificationSent;

class LogNotification
{
    /**
     * Handle the event.
     */
    public function handle(NotificationSent $event): void
    {
        // ...
    }
}
```

W ramach an event listener, you may access the `notifiable`, `notification`, `channel`, i `response` properties on the event to learn mczye about the notification recipient czy the notification itself:

```php
/**
 * Handle the event.
 */
public function handle(NotificationSent $event): void
{
    // $event->channel
    // $event->notifiable
    // $event->notification
    // $event->response
}
```

<a name="custom-kanały"></a>
## Własne kanały

Laravel ships with a hiful of notification kanały, but you may want to write your own drivers to deliver notifications via other kanały. Laravel makes it simple. Aby rozpocząć, define a class that contains a `send` method. The method should receive two arguments: a `$notifiable` i a `$notification`.

W ramach the `send` method, you may call methods on the notification to retrieve a message object understood by your channel i then send the notification to the `$notifiable` instance however you wish:

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;

class VoiceChannel
{
    /**
     * Send the given notification.
     */
    public function send(object $notifiable, Notification $notification): void
    {
        $message = $notification->toVoice($notifiable);

        // Send notification to the $notifiable instance...
    }
}
```

Gdy your notification channel class has been defined, you may return the class name from the `via` method of any of your notifications. In this example, the `toVoice` method of your notification can return whatever object you choose to represent voice messages. Na przykład, you might define your own `VoiceMessage` class to represent these messages:

```php
<?php

namespace App\Notifications;

use App\Notifications\Messages\VoiceMessage;
use App\Notifications\VoiceChannel;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification
{
    use Queueable;

    /**
     * Get the notification channels.
     */
    public function via(object $notifiable): string
    {
        return VoiceChannel::class;
    }

    /**
     * Get the voice representation of the notification.
     */
    public function toVoice(object $notifiable): VoiceMessage
    {
        // ...
    }
}
```
