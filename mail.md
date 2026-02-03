# Mail

- [Wprowadzenie](#introduction)
    - [Konfiguracja](#configuration)
    - [Wymagania sterowników](#driver-prerequisites)
    - [Konfiguracja przełączania awaryjnego](#failover-configuration)
    - [Konfiguracja Round Robin](#round-robin-configuration)
- [Generowanie klas mailowych](#generating-mailables)
- [Pisanie klas mailowych](#writing-mailables)
    - [Konfiguracja nadawcy](#configuring-the-sender)
    - [Konfiguracja widoku](#configuring-the-view)
    - [Dane widoku](#view-data)
    - [Załączniki](#attachments)
    - [Załączniki inline](#inline-attachments)
    - [Obiekty załączalne](#attachable-objects)
    - [Nagłówki](#headers)
    - [Tagi i metadane](#tags-and-metadata)
    - [Dostosowywanie wiadomości Symfony](#customizing-the-symfony-message)
- [Mailables Markdown](#markdown-mailables)
    - [Generowanie Mailables Markdown](#generating-markdown-mailables)
    - [Pisanie wiadomości Markdown](#writing-markdown-messages)
    - [Dostosowywanie komponentów](#customizing-the-components)
- [Wysyłanie maili](#sending-mail)
    - [Kolejkowanie maili](#queueing-mail)
- [Renderowanie mailables](#rendering-mailables)
    - [Podgląd mailables w przeglądarce](#previewing-mailables-in-the-browser)
- [Lokalizacja mailables](#localizing-mailables)
- [Testowanie](#testing-mailables)
    - [Testowanie zawartości maili](#testing-mailable-content)
    - [Testowanie wysyłania maili](#testing-mailable-sending)
- [Mail i rozwój lokalny](#mail-and-local-development)
- [Zdarzenia](#events)
- [Niestandardowe transporty](#custom-transports)
    - [Dodatkowe transporty Symfony](#additional-symfony-transports)

<a name="introduction"></a>
## Wprowadzenie

Wysyłanie e-maili nie musi być skomplikowane. Laravel zapewnia czysty, prosty API do poczty e-mail oparty na popularnym komponencie [Symfony Mailer](https://symfony.com/doc/current/mailer.html). Laravel i Symfony Mailer dostarczają sterowniki do wysyłania e-maili przez SMTP, Mailgun, Postmark, Resend, Amazon SES oraz `sendmail`, pozwalając na szybkie rozpoczęcie wysyłania poczty przez lokalną lub chmurową usługę według twojego wyboru.

<a name="configuration"></a>
### Konfiguracja

Usługi e-mail Laravel mogą być konfigurowane przez plik konfiguracyjny `config/mail.php` twojej aplikacji. Każdy mailer skonfigurowany w tym pliku może mieć swoją unikalną konfigurację, a nawet własny unikalny "transport", pozwalając twojej aplikacji używać różnych usług e-mail do wysyłania określonych wiadomości. Na przykład, twoja aplikacja może używać Postmark do wysyłania transakcyjnych e-maili, jednocześnie używając Amazon SES do wysyłania masowych e-maili.

W pliku konfiguracyjnym `mail` znajdziesz tablicę konfiguracyjną `mailers`. Ta tablica zawiera przykładowy wpis konfiguracyjny dla każdego z głównych sterowników / transportów poczty obsługiwanych przez Laravel, podczas gdy wartość konfiguracyjna `default` określa, który mailer będzie używany domyślnie, gdy twoja aplikacja potrzebuje wysłać wiadomość e-mail.

<a name="driver-prerequisites"></a>
### Wymagania sterowników / transportów

Sterowniki oparte na API, takie jak Mailgun, Postmark i Resend, są często prostsze i szybsze niż wysyłanie poczty przez serwery SMTP. Kiedy tylko jest to możliwe, zalecamy użycie jednego z tych sterowników.

<a name="mailgun-driver"></a>
#### Sterownik Mailgun

Aby użyć sterownika Mailgun, zainstaluj transport Symfony Mailgun Mailer przez Composer:

```shell
composer require symfony/mailgun-mailer symfony/http-client
```

Następnie musisz wprowadzić dwie zmiany w pliku konfiguracyjnym `config/mail.php` swojej aplikacji. Po pierwsze, ustaw domyślny mailer na `mailgun`:

```php
'default' => env('MAIL_MAILER', 'mailgun'),
```

Po drugie, dodaj następującą tablicę konfiguracyjną do swojej tablicy `mailers`:

```php
'mailgun' => [
    'transport' => 'mailgun',
    // 'client' => [
    //     'timeout' => 5,
    // ],
],
```

Po skonfigurowaniu domyślnego mailera aplikacji dodaj następujące opcje do pliku konfiguracyjnego `config/services.php`:

```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
    'endpoint' => env('MAILGUN_ENDPOINT', 'api.mailgun.net'),
    'scheme' => 'https',
],
```

Jeśli nie używasz [regionu Mailgun](https://documentation.mailgun.com/docs/mailgun/api-reference/#mailgun-regions) Stanów Zjednoczonych, możesz zdefiniować endpoint swojego regionu w pliku konfiguracyjnym `services`:

```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
    'endpoint' => env('MAILGUN_ENDPOINT', 'api.eu.mailgun.net'),
    'scheme' => 'https',
],
```

<a name="postmark-driver"></a>
#### Sterownik Postmark

Aby użyć sterownika [Postmark](https://postmarkapp.com/), zainstaluj transport Symfony Postmark Mailer przez Composer:

```shell
composer require symfony/postmark-mailer symfony/http-client
```

Następnie ustaw opcję `default` w pliku konfiguracyjnym `config/mail.php` swojej aplikacji na `postmark`. Po skonfigurowaniu domyślnego mailera aplikacji upewnij się, że plik konfiguracyjny `config/services.php` zawiera następujące opcje:

```php
'postmark' => [
    'key' => env('POSTMARK_API_KEY'),
],
```

Jeśli chcesz określić strumień wiadomości Postmark, który powinien być używany przez dany mailer, możesz dodać opcję konfiguracyjną `message_stream_id` do tablicy konfiguracyjnej mailera. Tę tablicę konfiguracyjną można znaleźć w pliku konfiguracyjnym `config/mail.php` twojej aplikacji:

```php
'postmark' => [
    'transport' => 'postmark',
    'message_stream_id' => env('POSTMARK_MESSAGE_STREAM_ID'),
    // 'client' => [
    //     'timeout' => 5,
    // ],
],
```

W ten sposób możesz również skonfigurować wiele mailerów Postmark z różnymi strumieniami wiadomości.

<a name="resend-driver"></a>
#### Sterownik Resend

Aby użyć sterownika [Resend](https://resend.com/), zainstaluj PHP SDK Resend przez Composer:

```shell
composer require resend/resend-php
```

Następnie ustaw opcję `default` w pliku konfiguracyjnym `config/mail.php` swojej aplikacji na `resend`. Po skonfigurowaniu domyślnego mailera aplikacji upewnij się, że plik konfiguracyjny `config/services.php` zawiera następujące opcje:

```php
'resend' => [
    'key' => env('RESEND_API_KEY'),
],
```

<a name="ses-driver"></a>
#### Sterownik SES

Aby użyć sterownika Amazon SES, musisz najpierw zainstalować Amazon AWS SDK dla PHP. Możesz zainstalować tę bibliotekę za pomocą menedżera pakietów Composer:

```shell
composer require aws/aws-sdk-php
```

Następnie ustaw opcję `default` w pliku konfiguracyjnym `config/mail.php` na `ses` i sprawdź, czy plik konfiguracyjny `config/services.php` zawiera następujące opcje:

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
],
```

Aby wykorzystać [tymczasowe poświadczenia](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html) AWS poprzez token sesji, możesz dodać klucz `token` do konfiguracji SES swojej aplikacji:

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'token' => env('AWS_SESSION_TOKEN'),
],
```

Aby korzystać z [funkcji zarządzania subskrypcjami](https://docs.aws.amazon.com/ses/latest/dg/sending-email-subscription-management.html) SES, możesz zwrócić nagłówek `X-Ses-List-Management-Options` w tablicy zwróconej przez metodę [headers](#headers) wiadomości e-mail:

```php
/**
 * Get the message headers.
 */
public function headers(): Headers
{
    return new Headers(
        text: [
            'X-Ses-List-Management-Options' => 'contactListName=MyContactList;topicName=MyTopic',
        ],
    );
}
```

Jeśli chcesz zdefiniować [dodatkowe opcje](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-sesv2-2019-09-27.html#sendemail), które Laravel powinien przekazać do metody `SendEmail` AWS SDK podczas wysyłania e-maila, możesz zdefiniować tablicę `options` w konfiguracji `ses`:

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'options' => [
        'ConfigurationSetName' => 'MyConfigurationSet',
        'EmailTags' => [
            ['Name' => 'foo', 'Value' => 'bar'],
        ],
    ],
],
```

<a name="failover-configuration"></a>
### Konfiguracja przełączania awaryjnego

Czasami zewnętrzna usługa skonfigurowana do wysyłania poczty twojej aplikacji może być niedostępna. W takich przypadkach może być użyteczne zdefiniowanie jednej lub więcej zapasowych konfiguracji dostarczania poczty, które będą używane w przypadku, gdy twój główny sterownik dostarczania jest niedostępny.

Aby to osiągnąć, powinieneś zdefiniować mailera w pliku konfiguracyjnym `mail` twojej aplikacji, który używa transportu `failover`. Tablica konfiguracyjna dla mailera `failover` twojej aplikacji powinna zawierać tablicę `mailers`, która określa kolejność, w jakiej skonfigurowane mailery powinny być wybierane do dostarczania:

```php
'mailers' => [
    'failover' => [
        'transport' => 'failover',
        'mailers' => [
            'postmark',
            'mailgun',
            'sendmail',
        ],
        'retry_after' => 60,
    ],

    // ...
],
```

Po skonfigurowaniu mailera używającego transportu `failover`, będziesz musiał ustawić failover mailer jako domyślny mailer w pliku `.env` twojej aplikacji, aby korzystać z funkcjonalności failover:

```ini
MAIL_MAILER=failover
```

<a name="round-robin-configuration"></a>
### Konfiguracja Round Robin

Transport `roundrobin` pozwala na rozdzielenie obciążenia wysyłania poczty między wieloma mailerami. Aby rozpocząć, zdefiniuj mailera w pliku konfiguracyjnym `mail` twojej aplikacji, który używa transportu `roundrobin`. Tablica konfiguracyjna dla mailera `roundrobin` twojej aplikacji powinna zawierać tablicę `mailers`, która określa, które skonfigurowane mailery powinny być używane do dostarczania:

```php
'mailers' => [
    'roundrobin' => [
        'transport' => 'roundrobin',
        'mailers' => [
            'ses',
            'postmark',
        ],
        'retry_after' => 60,
    ],

    // ...
],
```

Po zdefiniowaniu mailera round robin powinieneś ustawić ten mailer jako domyślny mailer używany przez twoją aplikację, określając jego nazwę jako wartość klucza konfiguracyjnego `default` w pliku konfiguracyjnym `mail` twojej aplikacji:

```php
'default' => env('MAIL_MAILER', 'roundrobin'),
```

Transport round robin wybiera losowy mailer z listy skonfigurowanych mailerów, a następnie przechodzi do następnego dostępnego mailera dla każdego kolejnego e-maila. W przeciwieństwie do transportu `failover`, który pomaga osiągnąć *[wysoką dostępność](https://en.wikipedia.org/wiki/High_availability)*, transport `roundrobin` zapewnia *[równoważenie obciążenia](https://en.wikipedia.org/wiki/Load_balancing_(computing))*.

<a name="generating-mailables"></a>
## Generowanie klas mailowych

Podczas budowania aplikacji Laravel, każdy typ e-maila wysyłanego przez twoją aplikację jest reprezentowany jako klasa "mailable". Te klasy są przechowywane w katalogu `app/Mail`. Nie martw się, jeśli nie widzisz tego katalogu w swojej aplikacji, ponieważ zostanie on wygenerowany dla ciebie, gdy utworzysz swoją pierwszą klasę mailable za pomocą polecenia Artisan `make:mail`:

```shell
php artisan make:mail OrderShipped
```

<a name="writing-mailables"></a>
## Pisanie klas mailowych

Po wygenerowaniu klasy mailable otwórz ją, abyśmy mogli zbadać jej zawartość. Konfiguracja klasy mailable odbywa się w kilku metodach, w tym metodach `envelope`, `content` i `attachments`.

Metoda `envelope` zwraca obiekt `Illuminate\Mail\Mailables\Envelope`, który definiuje temat, a czasem odbiorców wiadomości. Metoda `content` zwraca obiekt `Illuminate\Mail\Mailables\Content`, który definiuje [szablon Blade](/docs/{{version}}/blade), który będzie używany do generowania treści wiadomości.

<a name="configuring-the-sender"></a>
### Konfiguracja nadawcy

<a name="using-the-envelope"></a>
#### Używanie Envelope

Po pierwsze, zbadajmy konfigurację nadawcy e-maila. Innymi słowy, od kogo będzie e-mail. Istnieją dwa sposoby konfiguracji nadawcy. Najpierw możesz określić adres "from" w kopercie wiadomości:

```php
use Illuminate\Mail\Mailables\Address;
use Illuminate\Mail\Mailables\Envelope;

/**
 * Get the message envelope.
 */
public function envelope(): Envelope
{
    return new Envelope(
        from: new Address('jeffrey@example.com', 'Jeffrey Way'),
        subject: 'Order Shipped',
    );
}
```

Jeśli chcesz, możesz również określić adres `replyTo`:

```php
return new Envelope(
    from: new Address('jeffrey@example.com', 'Jeffrey Way'),
    replyTo: [
        new Address('taylor@example.com', 'Taylor Otwell'),
    ],
    subject: 'Order Shipped',
);
```

<a name="using-a-global-from-address"></a>
#### Używanie globalnego adresu `from`

Jednakże, jeśli twoja aplikacja używa tego samego adresu "from" dla wszystkich swoich e-maili, może być uciążliwe dodawanie go do każdej generowanej klasy mailable. Zamiast tego możesz określić globalny adres "from" w pliku konfiguracyjnym `config/mail.php`. Ten adres będzie używany, jeśli żaden inny adres "from" nie jest określony w klasie mailable:

```php
'from' => [
    'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
    'name' => env('MAIL_FROM_NAME', 'Example'),
],
```

Ponadto możesz zdefiniować globalny adres "reply_to" w pliku konfiguracyjnym `config/mail.php`:

```php
'reply_to' => [
    'address' => 'example@example.com',
    'name' => 'App Name',
],
```

<a name="configuring-the-view"></a>
### Konfiguracja widoku

W metodzie `content` klasy mailable możesz zdefiniować `view`, czyli szablon, który powinien być używany przy renderowaniu treści e-maila. Ponieważ każdy e-mail zazwyczaj używa [szablonu Blade](/docs/{{version}}/blade) do renderowania swojej zawartości, masz pełną moc i wygodę silnika szablonów Blade podczas budowania HTML-a swojego e-maila:

```php
/**
 * Get the message content definition.
 */
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
    );
}
```

> [!NOTE]
> Możesz chcieć utworzyć katalog `resources/views/mail`, aby pomieścić wszystkie swoje szablony e-mail; jednak możesz umieścić je gdzie chcesz w swoim katalogu `resources/views`.

<a name="plain-text-emails"></a>
#### E-maile tekstowe

Jeśli chcesz zdefiniować wersję tekstową swojego e-maila, możesz określić szablon tekstowy podczas tworzenia definicji `Content` wiadomości. Podobnie jak parametr `view`, parametr `text` powinien być nazwą szablonu, która będzie używana do renderowania treści e-maila. Możesz swobodnie definiować zarówno wersję HTML, jak i tekstową swojej wiadomości:

```php
/**
 * Get the message content definition.
 */
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
        text: 'mail.orders.shipped-text'
    );
}
```

Dla jasności parametr `html` może być używany jako alias parametru `view`:

```php
return new Content(
    html: 'mail.orders.shipped',
    text: 'mail.orders.shipped-text'
);
```

<a name="view-data"></a>
### Dane widoku

<a name="via-public-properties"></a>
#### Przez właściwości publiczne

Zazwyczaj będziesz chciał przekazać pewne dane do swojego widoku, które możesz wykorzystać podczas renderowania HTML-a e-maila. Istnieją dwa sposoby udostępniania danych widokowi. Po pierwsze, każda właściwość publiczna zdefiniowana w klasie mailable będzie automatycznie dostępna w widoku. Tak więc, na przykład, możesz przekazać dane do konstruktora klasy mailable i ustawić te dane jako właściwości publiczne zdefiniowane w klasie:

```php
<?php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * Create a new message instance.
     */
    public function __construct(
        public Order $order,
    ) {}

    /**
     * Get the message content definition.
     */
    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
        );
    }
}
```

Po ustawieniu danych jako właściwość publiczna będą one automatycznie dostępne w widoku, więc możesz uzyskać do nich dostęp tak, jak uzyskujesz dostęp do innych danych w szablonach Blade:

```blade
<div>
    Price: {{ $order->price }}
</div>
```

<a name="via-the-with-parameter"></a>
#### Przez parametr `with`:

Jeśli chcesz dostosować format danych e-maila przed ich wysłaniem do szablonu, możesz ręcznie przekazać dane do widoku poprzez parametr `with` definicji `Content`. Zazwyczaj nadal będziesz przekazywać dane przez konstruktor klasy mailable; jednak powinieneś ustawić te dane jako właściwości `protected` lub `private`, aby dane nie były automatycznie udostępniane szablonowi:

```php
<?php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * Create a new message instance.
     */
    public function __construct(
        protected Order $order,
    ) {}

    /**
     * Get the message content definition.
     */
    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
            with: [
                'orderName' => $this->order->name,
                'orderPrice' => $this->order->price,
            ],
        );
    }
}
```

Po przekazaniu danych poprzez parametr `with` będą one automatycznie dostępne w widoku, więc możesz uzyskać do nich dostęp tak, jak uzyskujesz dostęp do innych danych w szablonach Blade:

```blade
<div>
    Price: {{ $orderPrice }}
</div>
```

<a name="attachments"></a>
### Załączniki

Aby dodać załączniki do e-maila, dodasz załączniki do tablicy zwróconej przez metodę `attachments` wiadomości. Po pierwsze, możesz dodać załącznik, podając ścieżkę pliku do metody `fromPath` dostarczonej przez klasę `Attachment`:

```php
use Illuminate\Mail\Mailables\Attachment;

/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromPath('/path/to/file'),
    ];
}
```

Podczas dołączania plików do wiadomości możesz również określić nazwę wyświetlaną i/lub typ MIME załącznika za pomocą metod `as` i `withMime`:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromPath('/path/to/file')
            ->as('name.pdf')
            ->withMime('application/pdf'),
    ];
}
```

<a name="attaching-files-from-disk"></a>
#### Dołączanie plików z dysku

Jeśli przechowujesz plik na jednym ze swoich [dysków systemu plików](/docs/{{version}}/filesystem), możesz dołączyć go do e-maila za pomocą metody załącznika `fromStorage`:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromStorage('/path/to/file'),
    ];
}
```

Oczywiście możesz również określić nazwę i typ MIME załącznika:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromStorage('/path/to/file')
            ->as('name.pdf')
            ->withMime('application/pdf'),
    ];
}
```

Metoda `fromStorageDisk` może być użyta, jeśli musisz określić dysk magazynowania inny niż domyślny dysk:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromStorageDisk('s3', '/path/to/file')
            ->as('name.pdf')
            ->withMime('application/pdf'),
    ];
}
```

<a name="raw-data-attachments"></a>
#### Załączniki z surowych danych

Metoda załącznika `fromData` może być używana do dołączenia surowego ciągu bajtów jako załącznika. Na przykład, możesz użyć tej metody, jeśli wygenerowałeś PDF w pamięci i chcesz dołączyć go do e-maila bez zapisywania na dysku. Metoda `fromData` akceptuje closure, który rozwiązuje surowe bajty danych, a także nazwę, która powinna być przypisana do załącznika:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromData(fn () => $this->pdf, 'Report.pdf')
            ->withMime('application/pdf'),
    ];
}
```

<a name="inline-attachments"></a>
### Załączniki inline

Osadzanie obrazów inline w e-mailach jest zazwyczaj uciążliwe; jednak Laravel zapewnia wygodny sposób dołączania obrazów do e-maili. Aby osadzić obraz inline, użyj metody `embed` na zmiennej `$message` w szablonie e-maila. Laravel automatycznie udostępnia zmienną `$message` wszystkim szablonom e-mail, więc nie musisz się martwić o ręczne przekazywanie jej:

```blade
<body>
    Here is an image:

    <img src="{{ $message->embed($pathToImage) }}">
</body>
```

> [!WARNING]
> Zmienna `$message` nie jest dostępna w szablonach wiadomości tekstowych, ponieważ wiadomości tekstowe nie wykorzystują załączników inline.

<a name="embedding-raw-data-attachments"></a>
#### Osadzanie załączników z surowych danych

Jeśli masz już surowy ciąg danych obrazu, który chcesz osadzić w szablonie e-maila, możesz wywołać metodę `embedData` na zmiennej `$message`. Podczas wywoływania metody `embedData` musisz podać nazwę pliku, która powinna być przypisana do osadzonego obrazu:

```blade
<body>
    Here is an image from raw data:

    <img src="{{ $message->embedData($data, 'example-image.jpg') }}">
</body>
```

<a name="attachable-objects"></a>
### Obiekty możliwe do załączenia

Chociaż dołączanie plików do wiadomości za pomocą prostych ścieżek tekstowych często wystarcza, w wielu przypadkach encje możliwe do załączenia w twojej aplikacji są reprezentowane przez klasy. Na przykład, jeśli twoja aplikacja dołącza zdjęcie do wiadomości, twoja aplikacja może również mieć model `Photo`, który reprezentuje to zdjęcie. W takim przypadku, czy nie byłoby wygodnie po prostu przekazać model `Photo` do metody `attach`? Obiekty możliwe do załączenia pozwalają ci właśnie na to.

Aby rozpocząć, zaimplementuj interfejs `Illuminate\Contracts\Mail\Attachable` na obiekcie, który będzie możliwy do załączenia do wiadomości. Ten interfejs wymaga, aby twoja klasa definiowała metodę `toMailAttachment`, która zwraca instancję `Illuminate\Mail\Attachment`:

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Mail\Attachable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Mail\Attachment;

class Photo extends Model implements Attachable
{
    /**
     * Get the attachable representation of the model.
     */
    public function toMailAttachment(): Attachment
    {
        return Attachment::fromPath('/path/to/file');
    }
}
```

Po zdefiniowaniu obiektu możliwego do załączenia możesz zwrócić instancję tego obiektu z metody `attachments` podczas budowania wiadomości e-mail:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [$this->photo];
}
```

Oczywiście dane załączników mogą być przechowywane w zdalnej usłudze przechowywania plików, takiej jak Amazon S3. Dlatego Laravel umożliwia również generowanie instancji załączników z danych przechowywanych na jednym z [dysków systemu plików](/docs/{{version}}/filesystem) twojej aplikacji:

```php
// Create an attachment from a file on your default disk...
return Attachment::fromStorage($this->path);

// Create an attachment from a file on a specific disk...
return Attachment::fromStorageDisk('backblaze', $this->path);
```

Ponadto możesz tworzyć instancje załączników za pomocą danych, które masz w pamięci. Aby to osiągnąć, przekaż closure do metody `fromData`. Closure powinien zwrócić surowe dane reprezentujące załącznik:

```php
return Attachment::fromData(fn () => $this->content, 'Photo Name');
```

Laravel udostępnia również dodatkowe metody, których możesz użyć do dostosowania swoich załączników. Na przykład, możesz użyć metod `as` i `withMime`, aby dostosować nazwę pliku i typ MIME:

```php
return Attachment::fromPath('/path/to/file')
    ->as('Photo Name')
    ->withMime('image/jpeg');
```

<a name="headers"></a>
### Nagłówki

Czasami możesz potrzebować dołączyć dodatkowe nagłówki do wysyłanej wiadomości. Na przykład, możesz potrzebować ustawić niestandardowy `Message-Id` lub inne dowolne nagłówki tekstowe.

Aby to osiągnąć, zdefiniuj metodę `headers` na swoim mailable. Metoda `headers` powinna zwrócić instancję `Illuminate\Mail\Mailables\Headers`. Ta klasa akceptuje parametry `messageId`, `references` i `text`. Oczywiście możesz podać tylko parametry, których potrzebujesz dla konkretnej wiadomości:

```php
use Illuminate\Mail\Mailables\Headers;

/**
 * Get the message headers.
 */
public function headers(): Headers
{
    return new Headers(
        messageId: 'custom-message-id@example.com',
        references: ['previous-message@example.com'],
        text: [
            'X-Custom-Header' => 'Custom Value',
        ],
    );
}
```

<a name="tags-and-metadata"></a>
### Tagi i metadane

Niektórzy zewnętrzni dostawcy poczty e-mail, tacy jak Mailgun i Postmark, obsługują "tagi" i "metadane" wiadomości, które mogą być używane do grupowania i śledzenia e-maili wysyłanych przez twoją aplikację. Możesz dodać tagi i metadane do wiadomości e-mail poprzez definicję `Envelope`:

```php
use Illuminate\Mail\Mailables\Envelope;

/**
 * Get the message envelope.
 *
 * @return \Illuminate\Mail\Mailables\Envelope
 */
public function envelope(): Envelope
{
    return new Envelope(
        subject: 'Order Shipped',
        tags: ['shipment'],
        metadata: [
            'order_id' => $this->order->id,
        ],
    );
}
```

Jeśli twoja aplikacja używa sterownika Mailgun, możesz zapoznać się z dokumentacją Mailgun, aby uzyskać więcej informacji na temat [tagów](https://documentation.mailgun.com/docs/mailgun/user-manual/tracking-messages/#tags) i [metadanych](https://documentation.mailgun.com/docs/mailgun/user-manual/sending-messages/#attaching-metadata-to-messages). Podobnie, dokumentacja Postmark może również być przydatna w celu uzyskania więcej informacji na temat ich wsparcia dla [tagów](https://postmarkapp.com/blog/tags-support-for-smtp) i [metadanych](https://postmarkapp.com/support/article/1125-custom-metadata-faq).

Jeśli twoja aplikacja używa Amazon SES do wysyłania e-maili, powinieneś użyć metody `metadata`, aby dołączyć ["tagi" SES](https://docs.aws.amazon.com/ses/latest/APIReference/API_MessageTag.html) do wiadomości.

<a name="customizing-the-symfony-message"></a>
### Dostosowywanie wiadomości Symfony

Możliwości pocztowe Laravel są zasilane przez Symfony Mailer. Laravel pozwala na rejestrowanie niestandardowych callback'ów, które będą wywoływane z instancją Symfony Message przed wysłaniem wiadomości. Daje ci to możliwość głębokiego dostosowania wiadomości przed jej wysłaniem. Aby to osiągnąć, zdefiniuj parametr `using` w swojej definicji `Envelope`:

```php
use Illuminate\Mail\Mailables\Envelope;
use Symfony\Component\Mime\Email;

/**
 * Get the message envelope.
 */
public function envelope(): Envelope
{
    return new Envelope(
        subject: 'Order Shipped',
        using: [
            function (Email $message) {
                // ...
            },
        ]
    );
}
```

<a name="markdown-mailables"></a>
## Mailables Markdown

Wiadomości mailable Markdown pozwalają na wykorzystanie gotowych szablonów i komponentów [powiadomień mailowych](/docs/{{version}}/notifications#mail-notifications) w twoich mailables. Ponieważ wiadomości są pisane w Markdown, Laravel jest w stanie renderować piękne, responsywne szablony HTML dla wiadomości, a także automatycznie generować odpowiednik tekstowy.

<a name="generating-markdown-mailables"></a>
### Generowanie mailables Markdown

Aby wygenerować mailable z odpowiadającym szablonem Markdown, możesz użyć opcji `--markdown` polecenia Artisan `make:mail`:

```shell
php artisan make:mail OrderShipped --markdown=mail.orders.shipped
```

Następnie, podczas konfigurowania definicji `Content` mailable w metodzie `content`, użyj parametru `markdown` zamiast parametru `view`:

```php
use Illuminate\Mail\Mailables\Content;

/**
 * Get the message content definition.
 */
public function content(): Content
{
    return new Content(
        markdown: 'mail.orders.shipped',
        with: [
            'url' => $this->orderUrl,
        ],
    );
}
```

<a name="writing-markdown-messages"></a>
### Pisanie wiadomości Markdown

Markdown mailables wykorzystują kombinację komponentów Blade i składni Markdown, która pozwala łatwo konstruować wiadomości mailowe, jednocześnie wykorzystując gotowe komponenty UI e-mail Laravel:

```blade
<x-mail::message>
# Order Shipped

Your order has been shipped!

<x-mail::button :url="$url">
View Order
</x-mail::button>

Thanks,<br>
{{ config('app.name') }}
</x-mail::message>
```

> [!NOTE]
> Nie używaj nadmiernych wcięć podczas pisania e-maili Markdown. Zgodnie ze standardami Markdown, parsery Markdown będą renderować wcięte treści jako bloki kodu.

<a name="button-component"></a>
#### Komponent przycisku

Komponent przycisku renderuje wyśrodkowany link w formie przycisku. Komponent akceptuje dwa argumenty: `url` i opcjonalny `color`. Obsługiwane kolory to `primary`, `success` i `error`. Możesz dodać do wiadomości tyle komponentów przycisku, ile chcesz:

```blade
<x-mail::button :url="$url" color="success">
View Order
</x-mail::button>
```

<a name="panel-component"></a>
#### Komponent panelu

Komponent panelu renderuje dany blok tekstu w panelu, który ma nieco inny kolor tła niż reszta wiadomości. Pozwala to zwrócić uwagę na dany blok tekstu:

```blade
<x-mail::panel>
To jest treść panelu.
</x-mail::panel>
```

<a name="table-component"></a>
#### Komponent tabeli

Komponent tabeli pozwala przekształcić tabelę Markdown w tabelę HTML. Komponent akceptuje tabelę Markdown jako swoją zawartość. Wyrównanie kolumn tabeli jest obsługiwane przy użyciu domyślnej składni wyrównania tabeli Markdown:

```blade
<x-mail::table>
| Laravel       | Table         | Example       |
| ------------- | :-----------: | ------------: |
| Col 2 is      | Centered      | $10           |
| Col 3 is      | Right-Aligned | $20           |
</x-mail::table>
```

<a name="customizing-the-components"></a>
### Dostosowywanie komponentów

Możesz wyeksportować wszystkie komponenty poczty Markdown do swojej aplikacji w celu dostosowania. Aby wyeksportować komponenty, użyj polecenia Artisan `vendor:publish`, aby opublikować tag zasobów `laravel-mail`:

```shell
php artisan vendor:publish --tag=laravel-mail
```

To polecenie opublikuje komponenty poczty Markdown w katalogu `resources/views/vendor/mail`. Katalog `mail` będzie zawierał katalogi `html` i `text`, z których każdy zawiera odpowiednie reprezentacje wszystkich dostępnych komponentów. Możesz dowolnie dostosowywać te komponenty.

<a name="customizing-the-css"></a>
#### Dostosowywanie CSS

Po wyeksportowaniu komponentów katalog `resources/views/vendor/mail/html/themes` będzie zawierał plik `default.css`. Możesz dostosować CSS w tym pliku, a twoje style zostaną automatycznie przekonwertowane na wbudowane style CSS w reprezentacjach HTML twoich wiadomości mailowych Markdown.

Jeśli chcesz zbudować całkowicie nowy motyw dla komponentów Markdown Laravel, możesz umieścić plik CSS w katalogu `html/themes`. Po nazwaniu i zapisaniu pliku CSS zaktualizuj opcję `theme` w pliku konfiguracyjnym `config/mail.php` swojej aplikacji, aby dopasować nazwę nowego motywu.

Aby dostosować motyw dla indywidualnego mailable, możesz ustawić właściwość `$theme` klasy mailable na nazwę motywu, który powinien być używany podczas wysyłania tego mailable.

<a name="sending-mail"></a>
## Wysyłanie poczty

Aby wysłać wiadomość, użyj metody `to` na [fasadzie](/docs/{{version}}/facades) `Mail`. Metoda `to` akceptuje adres e-mail, instancję użytkownika lub kolekcję użytkowników. Jeśli przekażesz obiekt lub kolekcję obiektów, mailer automatycznie użyje ich właściwości `email` i `name` podczas określania odbiorców e-maila, więc upewnij się, że te atrybuty są dostępne w twoich obiektach. Po określeniu odbiorców możesz przekazać instancję swojej klasy mailable do metody `send`:

```php
<?php

namespace App\Http\Controllers;

use App\Mail\OrderShipped;
use App\Models\Order;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Mail;

class OrderShipmentController extends Controller
{
    /**
     * Ship the given order.
     */
    public function store(Request $request): RedirectResponse
    {
        $order = Order::findOrFail($request->order_id);

        // Ship the order...

        Mail::to($request->user())->send(new OrderShipped($order));

        return redirect('/orders');
    }
}
```

Nie jesteś ograniczony tylko do określania odbiorców "to" podczas wysyłania wiadomości. Możesz swobodnie ustawiać odbiorców "to", "cc" i "bcc", łącząc ich odpowiednie metody razem:

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->send(new OrderShipped($order));
```

<a name="looping-over-recipients"></a>
#### Pętla po odbiorcach

Czasami możesz potrzebować wysłać mailable do listy odbiorców, iterując przez tablicę odbiorców / adresów e-mail. Jednak ponieważ metoda `to` dodaje adresy e-mail do listy odbiorców mailable, każda iteracja przez pętlę wyśle kolejny e-mail do każdego poprzedniego odbiorcy. Dlatego zawsze powinieneś ponownie tworzyć instancję mailable dla każdego odbiorcy:

```php
foreach (['taylor@example.com', 'dries@example.com'] as $recipient) {
    Mail::to($recipient)->send(new OrderShipped($order));
}
```

<a name="sending-mail-via-a-specific-mailer"></a>
#### Wysyłanie poczty przez konkretny mailer

Domyślnie Laravel będzie wysyłać e-maile przy użyciu mailera skonfigurowanego jako `default` mailer w pliku konfiguracyjnym `mail` twojej aplikacji. Jednak możesz użyć metody `mailer`, aby wysłać wiadomość za pomocą konkretnej konfiguracji mailera:

```php
Mail::mailer('postmark')
    ->to($request->user())
    ->send(new OrderShipped($order));
```

<a name="queueing-mail"></a>
### Kolejkowanie poczty

<a name="queueing-a-mail-message"></a>
#### Kolejkowanie wiadomości e-mail

Ponieważ wysyłanie wiadomości e-mail może negatywnie wpłynąć na czas odpowiedzi twojej aplikacji, wielu programistów decyduje się na kolejkowanie wiadomości e-mail w celu wysyłania w tle. Laravel ułatwia to za pomocą wbudowanego [zunifikowanego API kolejek](/docs/{{version}}/queues). Aby kolejkować wiadomość mailową, użyj metody `queue` na fasadzie `Mail` po określeniu odbiorców wiadomości:

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue(new OrderShipped($order));
```

Ta metoda automatycznie zadba o umieszczenie zadania w kolejce, aby wiadomość była wysyłana w tle. Będziesz musiał [skonfigurować swoje kolejki](/docs/{{version}}/queues) przed użyciem tej funkcji.

<a name="delayed-message-queueing"></a>
#### Opóźnione kolejkowanie wiadomości

Jeśli chcesz opóźnić dostarczenie kolejkowanej wiadomości e-mail, możesz użyć metody `later`. Jako pierwszy argument metoda `later` akceptuje instancję `DateTime` wskazującą, kiedy wiadomość powinna zostać wysłana:

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->later(now()->plus(minutes: 10), new OrderShipped($order));
```

<a name="pushing-to-specific-queues"></a>
#### Wypychanie do konkretnych kolejek

Ponieważ wszystkie klasy mailable wygenerowane za pomocą polecenia `make:mail` wykorzystują trait `Illuminate\Bus\Queueable`, możesz wywołać metody `onQueue` i `onConnection` na dowolnej instancji klasy mailable, co pozwala określić połączenie i nazwę kolejki dla wiadomości:

```php
$message = (new OrderShipped($order))
    ->onConnection('sqs')
    ->onQueue('emails');

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue($message);
```

<a name="queueing-by-default"></a>
#### Domyślne kolejkowanie

Jeśli masz klasy mailable, które chcesz, aby zawsze były kolejkowane, możesz zaimplementować kontrakt `ShouldQueue` w klasie. Teraz, nawet jeśli wywołasz metodę `send` podczas wysyłania poczty, mailable nadal będzie kolejkowany, ponieważ implementuje kontrakt:

```php
use Illuminate\Contracts\Queue\ShouldQueue;

class OrderShipped extends Mailable implements ShouldQueue
{
    // ...
}
```

<a name="queued-mailables-and-database-transactions"></a>
#### Kolejkowane mailables i transakcje bazodanowe

Kiedy kolejkowane mailables są wysyłane w obrbie transakcji bazodanowych, mogą być przetwarzane przez kolejkę zanim transakcja bazodanowa została zatwierdzona. Kiedy to się dzieje, wszelkie aktualizacje, które wprowadziłeś do modeli lub rekordów bazy danych podczas transakcji bazodanowej, mogą jeszcze nie być odzwierciedlone w bazie danych. Ponadto wszelkie modele lub rekordy bazy danych utworzone w obrbie transakcji mogą nie istnieć w bazie danych. Jeśli twój mailable zależy od tych modeli, nieoczekiwane błędy mogą wystąpić, gdy zadanie wysyłające kolejkowany mailable jest przetwarzane.

Jeśli opcja konfiguracyjna `after_commit` twojego połączenia kolejki jest ustawiona na `false`, nadal możesz wskazać, że konkretny kolejkowany mailable powinien zostać wysłany po zatwierdzeniu wszystkich otwartych transakcji bazodanowych, wywołując metodę `afterCommit` podczas wysyłania wiadomości mailowej:

```php
Mail::to($request->user())->send(
    (new OrderShipped($order))->afterCommit()
);
```

Alternatywnie możesz wywołać metodę `afterCommit` z konstruktora swojego mailable:

```php
<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable implements ShouldQueue
{
    use Queueable, SerializesModels;

    /**
     * Create a new message instance.
     */
    public function __construct()
    {
        $this->afterCommit();
    }
}
```

> [!NOTE]
> Aby dowiedzieć się więcej o obejściu tych problemów, zapoznaj się z dokumentacją dotyczącą [kolejkowanych zadań i transakcji bazodanowych](/docs/{{version}}/queues#jobs-and-database-transactions).

<a name="queued-email-failures"></a>
#### Niepowodzenia kolejkowanych e-maili

Kiedy kolejkowany e-mail się nie powiedzie, metoda `failed` w klasie kolejkowanego mailable zostanie wywołana, jeśli została zdefiniowana. Instancja `Throwable`, która spowodowała niepowodzenie kolejkowanego e-maila, zostanie przekazana do metody `failed`:

```php
<?php

namespace App\Mail;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;
use Throwable;

class OrderDelayed extends Mailable implements ShouldQueue
{
    use SerializesModels;

    /**
     * Handle a queued email's failure.
     */
    public function failed(Throwable $exception): void
    {
        // ...
    }
}
```

<a name="rendering-mailables"></a>
## Renderowanie mailables

Czasami możesz chcieć przechwycić treść HTML mailable bez jej wysyłania. Aby to osiągnąć, możesz wywołać metodę `render` mailable. Metoda ta zwróci ocenioną treść HTML mailable jako ciąg znaków:

```php
use App\Mail\InvoicePaid;
use App\Models\Invoice;

$invoice = Invoice::find(1);

return (new InvoicePaid($invoice))->render();
```

<a name="previewing-mailables-in-the-browser"></a>
### Podgląd mailables w przeglądarce

Podczas projektowania szablonu mailable wygodnie jest szybko przeglądać wyrenderowany mailable w przeglądarce, tak jak typowy szablon Blade. Z tego powodu Laravel pozwala na zwrócenie dowolnego mailable bezpośrednio z closure trasy lub kontrolera. Kiedy mailable jest zwracany, zostanie wyrenderowany i wyświetlony w przeglądarce, pozwalając szybko obejrzeć jego wygląd bez konieczności wysyłania go na rzeczywisty adres e-mail:

```php
Route::get('/mailable', function () {
    $invoice = App\Models\Invoice::find(1);

    return new App\Mail\InvoicePaid($invoice);
});
```

<a name="localizing-mailables"></a>
## Lokalizacja mailables

Laravel pozwala na wysyłanie mailables w ustawieniach lokalnych innych niż bieżące ustawienia lokalne żądania i nawet zapamięta te ustawienia lokalne, jeśli poczta jest kolejkowana.

Aby to osiągnąć, fasada `Mail` oferuje metodę `locale`, aby ustawić pożądany język. Aplikacja zmieni się na te ustawienia lokalne, gdy szablon mailable jest oceniany, a następnie powróci do poprzednich ustawień lokalnych po zakończeniu oceny:

```php
Mail::to($request->user())->locale('es')->send(
    new OrderShipped($order)
);
```

<a name="user-preferred-locales"></a>
#### Preferowane ustawienia lokalne użytkownika

Czasami aplikacje przechowują preferowane ustawienia lokalne każdego użytkownika. Implementując kontrakt `HasLocalePreference` w jednym lub więcej swoich modelach, możesz polecić Laravel, aby używał tych przechowywanych ustawień lokalnych podczas wysyłania poczty:

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

Po zaimplementowaniu interfejsu Laravel automatycznie użyje preferowanych ustawień lokalnych podczas wysyłania mailables i powiadomień do modelu. Dlatego nie ma potrzeby wywoływania metody `locale` podczas używania tego interfejsu:

```php
Mail::to($request->user())->send(new OrderShipped($order));
```

<a name="testing-mailables"></a>
## Testowanie

<a name="testing-mailable-content"></a>
### Testowanie zawartości mailables

Laravel dostarcza różne metody do inspekcji struktury twojego mailable. Ponadto Laravel zapewnia kilka wygodnych metod do testowania, czy twój mailable zawiera oczekiwaną treść:

```php tab=Pest
use App\Mail\InvoicePaid;
use App\Models\User;

test('mailable content', function () {
    $user = User::factory()->create();

    $mailable = new InvoicePaid($user);

    $mailable->assertFrom('jeffrey@example.com');
    $mailable->assertTo('taylor@example.com');
    $mailable->assertHasCc('abigail@example.com');
    $mailable->assertHasBcc('victoria@example.com');
    $mailable->assertHasReplyTo('tyler@example.com');
    $mailable->assertHasSubject('Invoice Paid');
    $mailable->assertHasTag('example-tag');
    $mailable->assertHasMetadata('key', 'value');

    $mailable->assertSeeInHtml($user->email);
    $mailable->assertDontSeeInHtml('Invoice Not Paid');
    $mailable->assertSeeInOrderInHtml(['Invoice Paid', 'Thanks']);

    $mailable->assertSeeInText($user->email);
    $mailable->assertDontSeeInText('Invoice Not Paid');
    $mailable->assertSeeInOrderInText(['Invoice Paid', 'Thanks']);

    $mailable->assertHasAttachment('/path/to/file');
    $mailable->assertHasAttachment(Attachment::fromPath('/path/to/file'));
    $mailable->assertHasAttachedData($pdfData, 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorage('/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorageDisk('s3', '/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
});
```

```php tab=PHPUnit
use App\Mail\InvoicePaid;
use App\Models\User;

public function test_mailable_content(): void
{
    $user = User::factory()->create();

    $mailable = new InvoicePaid($user);

    $mailable->assertFrom('jeffrey@example.com');
    $mailable->assertTo('taylor@example.com');
    $mailable->assertHasCc('abigail@example.com');
    $mailable->assertHasBcc('victoria@example.com');
    $mailable->assertHasReplyTo('tyler@example.com');
    $mailable->assertHasSubject('Invoice Paid');
    $mailable->assertHasTag('example-tag');
    $mailable->assertHasMetadata('key', 'value');

    $mailable->assertSeeInHtml($user->email);
    $mailable->assertDontSeeInHtml('Invoice Not Paid');
    $mailable->assertSeeInOrderInHtml(['Invoice Paid', 'Thanks']);

    $mailable->assertSeeInText($user->email);
    $mailable->assertDontSeeInText('Invoice Not Paid');
    $mailable->assertSeeInOrderInText(['Invoice Paid', 'Thanks']);

    $mailable->assertHasAttachment('/path/to/file');
    $mailable->assertHasAttachment(Attachment::fromPath('/path/to/file'));
    $mailable->assertHasAttachedData($pdfData, 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorage('/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorageDisk('s3', '/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
}
```

Jak można się spodziewać, asercje "HTML" sprawdzają, czy wersja HTML twojego mailable zawiera dany ciąg znaków, podczas gdy asercje "text" sprawdzają, czy wersja tekstowa twojego mailable zawiera dany ciąg znaków.

<a name="testing-mailable-sending"></a>
### Testowanie wysyłania mailables

Sugerujemy osobne testowanie treści twoich mailables od testów, które sprawdzają, czy dany mailable został "wysłany" do konkretnego użytkownika. Zazwyczaj treść mailables nie jest istotna dla kodu, który testujesz, i wystarczy po prostu stwierdzić, że Laravel otrzymał polecenie wysłania danego mailable.

Możesz użyć metody `fake` fasady `Mail`, aby zapobiec wysyłaniu poczty. Po wywołaniu metody `fake` fasady `Mail` możesz następnie sprawdzić, czy mailables otrzymały polecenie wysłania do użytkowników, a nawet sprawdzić dane, które otrzymały mailables:

```php tab=Pest
<?php

use App\Mail\OrderShipped;
use Illuminate\Support\Facades\Mail;

test('orders can be shipped', function () {
    Mail::fake();

    // Perform order shipping...

    // Assert that no mailables were sent...
    Mail::assertNothingSent();

    // Assert that a mailable was sent...
    Mail::assertSent(OrderShipped::class);

    // Assert a mailable was sent twice...
    Mail::assertSent(OrderShipped::class, 2);

    // Assert a mailable was sent to an email address...
    Mail::assertSent(OrderShipped::class, 'example@laravel.com');

    // Assert a mailable was sent to multiple email addresses...
    Mail::assertSent(OrderShipped::class, ['example@laravel.com', '...']);

    // Assert a mailable was not sent...
    Mail::assertNotSent(AnotherMailable::class);

    // Assert a mailable was sent twice...
    Mail::assertSentTimes(OrderShipped::class, 2);

    // Assert 3 total mailables were sent...
    Mail::assertSentCount(3);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Mail\OrderShipped;
use Illuminate\Support\Facades\Mail;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        Mail::fake();

        // Perform order shipping...

        // Assert that no mailables were sent...
        Mail::assertNothingSent();

        // Assert that a mailable was sent...
        Mail::assertSent(OrderShipped::class);

        // Assert a mailable was sent twice...
        Mail::assertSent(OrderShipped::class, 2);

        // Assert a mailable was sent to an email address...
        Mail::assertSent(OrderShipped::class, 'example@laravel.com');

        // Assert a mailable was sent to multiple email addresses...
        Mail::assertSent(OrderShipped::class, ['example@laravel.com', '...']);

        // Assert a mailable was not sent...
        Mail::assertNotSent(AnotherMailable::class);

        // Assert a mailable was sent twice...
        Mail::assertSentTimes(OrderShipped::class, 2);

        // Assert 3 total mailables were sent...
        Mail::assertSentCount(3);
    }
}
```

Jeśli kolejkujesz mailables do dostarczenia w tle, powinieneś użyć metody `assertQueued` zamiast `assertSent`:

```php
Mail::assertQueued(OrderShipped::class);
Mail::assertNotQueued(OrderShipped::class);
Mail::assertNothingQueued();
Mail::assertQueuedCount(3);
```

Możesz również sprawdzić całkowitą liczbę mailables, które zostały wysłane lub kolejkowane za pomocą metody `assertOutgoingCount`:

```php
Mail::assertOutgoingCount(3);
```

Możesz przekazać closure do metod `assertSent`, `assertNotSent`, `assertQueued` lub `assertNotQueued`, aby stwierdzić, że mailable został wysłany, który przechodzi dany "test prawdy". Jeśli co najmniej jeden mailable został wysłany, który przechodzi dany test prawdy, asercja będzie udana:

```php
Mail::assertSent(function (OrderShipped $mail) use ($order) {
    return $mail->order->id === $order->id;
});
```

Podczas wywoływania metod asercji fasady `Mail`, instancja mailable zaakceptowana przez dostarczone closure udostępnia pomocne metody do badania mailable:

```php
Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) use ($user) {
    return $mail->hasTo($user->email) &&
           $mail->hasCc('...') &&
           $mail->hasBcc('...') &&
           $mail->hasReplyTo('...') &&
           $mail->hasFrom('...') &&
           $mail->hasSubject('...') &&
           $mail->hasMetadata('order_id', $mail->order->id);
           $mail->usesMailer('ses');
});
```

Instancja mailable zawiera również kilka pomocnych metod do badania załączników w mailable:

```php
use Illuminate\Mail\Mailables\Attachment;

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) {
    return $mail->hasAttachment(
        Attachment::fromPath('/path/to/file')
            ->as('name.pdf')
            ->withMime('application/pdf')
    );
});

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) {
    return $mail->hasAttachment(
        Attachment::fromStorageDisk('s3', '/path/to/file')
    );
});

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) use ($pdfData) {
    return $mail->hasAttachment(
        Attachment::fromData(fn () => $pdfData, 'name.pdf')
    );
});
```

Mogłeś zauważyć, że istnieją dwie metody sprawdzania, czy poczta nie została wysłana: `assertNotSent` i `assertNotQueued`. Czasami możesz chcieć sprawdzić, czy żadna poczta nie została wysłana **lub** kolejkowana. Aby to osiągnąć, możesz użyć metod `assertNothingOutgoing` i `assertNotOutgoing`:

```php
Mail::assertNothingOutgoing();

Mail::assertNotOutgoing(function (OrderShipped $mail) use ($order) {
    return $mail->order->id === $order->id;
});
```

<a name="mail-and-local-development"></a>
## Mail i programowanie lokalne

Podczas tworzenia aplikacji, która wysyła e-maile, prawdopodobnie nie chcesz rzeczywiście wysyłać e-maili na żywe adresy e-mail. Laravel zapewnia kilka sposobów "wyłączenia" rzeczywistego wysyłania e-maili podczas rozwoju lokalnego.

<a name="log-driver"></a>
#### Sterownik Log

Zamiast wysyłać e-maile, sterownik poczty `log` zapisze wszystkie wiadomości e-mail do plików dziennika w celu inspekcji. Zazwyczaj ten sterownik byłby używany tylko podczas rozwoju lokalnego. Więcej informacji na temat konfigurowania aplikacji według środowiska znajdziesz w [dokumentacji konfiguracji](/docs/{{version}}/configuration#environment-configuration).

<a name="mailtrap"></a>
#### HELO / Mailtrap / Mailpit

Alternatywnie możesz użyć usługi takiej jak [HELO](https://usehelo.com) lub [Mailtrap](https://mailtrap.io) oraz sterownika `smtp`, aby wysyłać wiadomości e-mail do "fałszywej" skrzynki pocztowej, gdzie możesz je wyświetlić w prawdziwym kliencie poczty e-mail. To podejście ma tę zaletę, że pozwala faktycznie sprawdzić ostateczne e-maile w przeglądarce wiadomości Mailtrap.

Jeśli używasz [Laravel Sail](/docs/{{version}}/sail), możesz przeglądać swoje wiadomości za pomocą [Mailpit](https://github.com/axllent/mailpit). Gdy Sail jest uruchomiony, możesz uzyskać dostęp do interfejsu Mailpit pod adresem: `http://localhost:8025`.

<a name="using-a-global-to-address"></a>
#### Używanie globalnego adresu `to`

Na koniec możesz określić globalny adres "to", wywołując metodę `alwaysTo` oferowanej przez fasadę `Mail`. Zazwyczaj ta metoda powinna być wywoływana z metody `boot` jednego z dostawców usług twojej aplikacji:

```php
use Illuminate\Support\Facades\Mail;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    if ($this->app->environment('local')) {
        Mail::alwaysTo('taylor@example.com');
    }
}
```

Podczas używania metody `alwaysTo` wszelkie dodatkowe adresy "cc" lub "bcc" w wiadomościach mailowych zostaną usunięte.

<a name="events"></a>
## Zdarzenia

Laravel emituje dwa zdarzenia podczas wysyłania wiadomości mailowych. Zdarzenie `MessageSending` jest emitowane przed wysłaniem wiadomości, podczas gdy zdarzenie `MessageSent` jest emitowane po wysłaniu wiadomości. Pamiętaj, że te zdarzenia są emitowane, gdy poczta jest *wysyłana*, a nie gdy jest kolejkowana. Możesz tworzyć [nasłuchiwacze zdarzeń](/docs/{{version}}/events) dla tych zdarzeń w swojej aplikacji:

```php
use Illuminate\Mail\Events\MessageSending;
// use Illuminate\Mail\Events\MessageSent;

class LogMessage
{
    /**
     * Handle the event.
     */
    public function handle(MessageSending $event): void
    {
        // ...
    }
}
```

<a name="custom-transports"></a>
## Niestandardowe transporty

Laravel zawiera różne transporty pocztowe; jednak możesz chcieć napisać własne transporty, aby dostarczać e-maile za pośrednictwem innych usług, które Laravel nie obsługuje od razu. Aby rozpocząć, zdefiniuj klasę, która rozszerza klasę `Symfony\Component\Mailer\Transport\AbstractTransport`. Następnie zaimplementuj metody `doSend` i `__toString` w swoim transporcie:

```php
<?php

namespace App\Mail;

use MailchimpTransactional\ApiClient;
use Symfony\Component\Mailer\SentMessage;
use Symfony\Component\Mailer\Transport\AbstractTransport;
use Symfony\Component\Mime\Address;
use Symfony\Component\Mime\MessageConverter;

class MailchimpTransport extends AbstractTransport
{
    /**
     * Create a new Mailchimp transport instance.
     */
    public function __construct(
        protected ApiClient $client,
    ) {
        parent::__construct();
    }

    /**
     * {@inheritDoc}
     */
    protected function doSend(SentMessage $message): void
    {
        $email = MessageConverter::toEmail($message->getOriginalMessage());

        $this->client->messages->send(['message' => [
            'from_email' => $email->getFrom(),
            'to' => collect($email->getTo())->map(function (Address $email) {
                return ['email' => $email->getAddress(), 'type' => 'to'];
            })->all(),
            'subject' => $email->getSubject(),
            'text' => $email->getTextBody(),
        ]]);
    }

    /**
     * Get the string representation of the transport.
     */
    public function __toString(): string
    {
        return 'mailchimp';
    }
}
```

Po zdefiniowaniu niestandardowego transportu możesz go zarejestrować za pomocą metody `extend` dostarczonej przez fasadę `Mail`. Zazwyczaj powinno to być wykonane w metodzie `boot` dostawcy usług `AppServiceProvider` twojej aplikacji. Argument `$config` zostanie przekazany do closure dostarczonego do metody `extend`. Ten argument będzie zawierać tablicę konfiguracyjną zdefiniowaną dla mailera w pliku konfiguracyjnym `config/mail.php` aplikacji:

```php
use App\Mail\MailchimpTransport;
use Illuminate\Support\Facades\Mail;
use MailchimpTransactional\ApiClient;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Mail::extend('mailchimp', function (array $config = []) {
        $client = new ApiClient;

        $client->setApiKey($config['key']);

        return new MailchimpTransport($client);
    });
}
```

Po zdefiniowaniu i zarejestrowaniu niestandardowego transportu możesz utworzyć definicję mailera w pliku konfiguracyjnym `config/mail.php` swojej aplikacji, która wykorzystuje nowy transport:

```php
'mailchimp' => [
    'transport' => 'mailchimp',
    'key' => env('MAILCHIMP_API_KEY'),
    // ...
],
```

<a name="additional-symfony-transports"></a>
### Dodatkowe transporty Symfony

Laravel zawiera wsparcie dla niektórych istniejących transportów pocztowych utrzymywanych przez Symfony, takich jak Mailgun i Postmark. Jednak możesz chcieć rozszerzyć Laravel o wsparcie dla dodatkowych transportów utrzymywanych przez Symfony. Możesz to zrobić, wymagając niezbędnego mailera Symfony przez Composer i rejestrując transport w Laravel. Na przykład, możesz zainstalować i zarejestrować mailer Symfony "Brevo" (wcześniej "Sendinblue"):

```shell
composer require symfony/brevo-mailer symfony/http-client
```

Po zainstalowaniu pakietu mailera Brevo możesz dodać wpis dla poświadczeń API Brevo do pliku konfiguracyjnego `services` swojej aplikacji:

```php
'brevo' => [
    'key' => env('BREVO_API_KEY'),
],
```

Następnie możesz użyć metody `extend` fasady `Mail`, aby zarejestrować transport w Laravel. Zazwyczaj powinno to być wykonane w metodzie `boot` dostawcy usług:

```php
use Illuminate\Support\Facades\Mail;
use Symfony\Component\Mailer\Bridge\Brevo\Transport\BrevoTransportFactory;
use Symfony\Component\Mailer\Transport\Dsn;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Mail::extend('brevo', function () {
        return (new BrevoTransportFactory)->create(
            new Dsn(
                'brevo+api',
                'default',
                config('services.brevo.key')
            )
        );
    });
}
```

Po zarejestrowaniu transportu możesz utworzyć definicję mailera w pliku konfiguracyjnym `config/mail.php` swojej aplikacji, która wykorzystuje nowy transport:

```php
'brevo' => [
    'transport' => 'brevo',
    // ...
],
```
