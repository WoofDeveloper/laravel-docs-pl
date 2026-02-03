# Logowanie

- [Wprowadzenie](#introduction)
- [Konfiguracja](#configuration)
    - [Dostępne sterowniki kanałów](#available-channel-drivers)
    - [Wymagania wstępne kanałów](#channel-prerequisites)
    - [Logowanie ostrzeżeń o przestarzałych funkcjach](#logging-deprecation-warnings)
- [Budowanie stosów logów](#building-log-stacks)
- [Zapisywanie komunikatów logów](#writing-log-messages)
    - [Informacje kontekstowe](#contextual-information)
    - [Zapisywanie do konkretnych kanałów](#writing-to-specific-channels)
- [Dostosowywanie kanałów Monolog](#monolog-channel-customization)
    - [Dostosowywanie Monolog dla kanałów](#customizing-monolog-for-channels)
    - [Tworzenie kanałów obsługi Monolog](#creating-monolog-handler-channels)
    - [Tworzenie niestandardowych kanałów za pomocą fabryk](#creating-custom-channels-via-factories)
- [Śledzenie komunikatów logów za pomocą Pail](#tailing-log-messages-using-pail)
    - [Instalacja](#pail-installation)
    - [Użycie](#pail-usage)
    - [Filtrowanie logów](#pail-filtering-logs)

<a name="introduction"></a>
## Wprowadzenie

Aby pomóc Ci dowiedzieć się więcej o tym, co dzieje się w Twojej aplikacji, Laravel zapewnia solidne usługi logowania, które pozwalają na zapisywanie komunikatów do plików, do dziennika błędów systemu, a nawet do Slacka, aby powiadomić cały Twój zespół.

Logowanie Laravel opiera się na "kanałach". Każdy kanał reprezentuje konkretny sposób zapisywania informacji logowania. Na przykład kanał `single` zapisuje pliki logów do pojedynczego pliku logów, podczas gdy kanał `slack` wysyła komunikaty logów do Slacka. Komunikaty logów mogą być zapisywane do wielu kanałów w zależności od ich wagi.

Pod maską Laravel wykorzystuje bibliotekę [Monolog](https://github.com/Seldaek/monolog), która zapewnia wsparcie dla różnych potężnych programów obsługi logów. Laravel ułatwia konfigurację tych programów obsługi, pozwalając na ich mieszanie i dopasowywanie w celu dostosowania obsługi logowania aplikacji.

<a name="configuration"></a>
## Konfiguracja

Wszystkie opcje konfiguracyjne kontrolujące zachowanie logowania Twojej aplikacji znajdują się w pliku konfiguracyjnym `config/logging.php`. Ten plik pozwala na skonfigurowanie kanałów logów aplikacji, więc upewnij się, że przejrzysz każdy z dostępnych kanałów i ich opcji. Poniżej omówimy kilka popularnych opcji.

Domyślnie Laravel będzie używał kanału `stack` podczas logowania komunikatów. Kanał `stack` służy do agregowania wielu kanałów logów w jeden kanał. Aby uzyskać więcej informacji na temat budowania stosów, sprawdź [dokumentację poniżej](#building-log-stacks).

<a name="available-channel-drivers"></a>
### Dostępne sterowniki kanałów

Każdy kanał logów jest zasilany przez "sterownik". Sterownik określa, jak i gdzie komunikat logów jest faktycznie rejestrowany. Następujące sterowniki kanałów logów są dostępne w każdej aplikacji Laravel. Wpis dla większości z tych sterowników jest już obecny w pliku konfiguracyjnym `config/logging.php` Twojej aplikacji, więc upewnij się, że przejrzysz ten plik, aby zapoznać się z jego zawartością:

<div class="overflow-auto">

| Nazwa        | Opis                                                                         |
| ------------ | ---------------------------------------------------------------------------- |
| `custom`     | Sterownik, który wywołuje określoną fabrykę do utworzenia kanału.           |
| `daily`      | Sterownik Monolog oparty na `RotatingFileHandler`, który rotuje codziennie. |
| `errorlog`   | Sterownik Monolog oparty na `ErrorLogHandler`.                               |
| `monolog`    | Sterownik fabryki Monolog, który może używać dowolnego obsługiwanego programu obsługi Monolog. |
| `papertrail` | Sterownik Monolog oparty na `SyslogUdpHandler`.                              |
| `single`     | Kanał loggera oparty na pojedynczym pliku lub ścieżce (`StreamHandler`).    |
| `slack`      | Sterownik Monolog oparty na `SlackWebhookHandler`.                           |
| `stack`      | Wrapper ułatwiający tworzenie kanałów "wielokanałowych".                     |
| `syslog`     | Sterownik Monolog oparty na `SyslogHandler`.                                 |

</div>

> [!NOTE]
> Sprawdź dokumentację dotyczącą [zaawansowanego dostosowywania kanałów](#monolog-channel-customization), aby dowiedzieć się więcej o sterownikach `monolog` i `custom`.

<a name="configuring-the-channel-name"></a>
#### Konfigurowanie nazwy kanału

Domyślnie Monolog jest instancjonowany z "nazwą kanału", która odpowiada bieżącemu środowisku, takim jak `production` lub `local`. Aby zmienić tę wartość, możesz dodać opcję `name` do konfiguracji Twojego kanału:

```php
'stack' => [
    'driver' => 'stack',
    'name' => 'channel-name',
    'channels' => ['single', 'slack'],
],
```

<a name="channel-prerequisites"></a>
### Wymagania wstępne kanałów

<a name="configuring-the-single-and-daily-channels"></a>
#### Konfigurowanie kanałów Single i Daily

Kanały `single` i `daily` mają trzy opcjonalne opcje konfiguracyjne: `bubble`, `permission` i `locking`.

<div class="overflow-auto">

| Nazwa        | Opis                                                                            | Domyślnie |
| ------------ | ------------------------------------------------------------------------------- | --------- |
| `bubble`     | Wskazuje, czy komunikaty powinny przechodzić do innych kanałów po obsłudze.    | `true`    |
| `locking`    | Próbuj zablokować plik logów przed zapisaniem do niego.                         | `false`   |
| `permission` | Uprawnienia pliku logów.                                                        | `0644`    |

</div>

Dodatkowo polityka przechowywania dla kanału `daily` może być skonfigurowana za pomocą zmiennej środowiskowej `LOG_DAILY_DAYS` lub przez ustawienie opcji konfiguracyjnej `days`.

<div class="overflow-auto">

| Nazwa  | Opis                                                                | Domyślnie |
| ------ | ------------------------------------------------------------------- | --------- |
| `days` | Liczba dni, przez które dzienne pliki logów powinny być zachowane.  | `14`      |

</div>

<a name="configuring-the-papertrail-channel"></a>
#### Konfigurowanie kanału Papertrail

Kanał `papertrail` wymaga opcji konfiguracyjnych `host` i `port`. Mogą one być zdefiniowane za pomocą zmiennych środowiskowych `PAPERTRAIL_URL` i `PAPERTRAIL_PORT`. Możesz uzyskać te wartości z [Papertrail](https://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-php-apps/#send-events-from-php-app).

<a name="configuring-the-slack-channel"></a>
#### Konfigurowanie kanału Slack

Kanał `slack` wymaga opcji konfiguracyjnej `url`. Ta wartość może być zdefiniowana za pomocą zmiennej środowiskowej `LOG_SLACK_WEBHOOK_URL`. Ten URL powinien odpowiadać adresowi URL dla [przychodzącego webhooka](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks), który skonfigurowałeś dla swojego zespołu Slack.

Domyślnie Slack będzie otrzymywał tylko logi na poziomie `critical` i wyższym; jednak możesz dostosować to za pomocą zmiennej środowiskowej `LOG_LEVEL` lub modyfikując opcję konfiguracyjną `level` w tablicy konfiguracyjnej kanału logów Slack.

<a name="logging-deprecation-warnings"></a>
### Logowanie ostrzeżeń o przestarzałych funkcjach

PHP, Laravel i inne biblioteki często powiadamiają swoich użytkowników, że niektóre z ich funkcji zostały przestarzałe i zostaną usunięte w przyszłej wersji. Jeśli chcesz rejestrować te ostrzeżenia o przestarzałych funkcjach, możesz określić preferowany kanał logów `deprecations` za pomocą zmiennej środowiskowej `LOG_DEPRECATIONS_CHANNEL` lub w pliku konfiguracyjnym `config/logging.php` Twojej aplikacji:

```php
'deprecations' => [
    'channel' => env('LOG_DEPRECATIONS_CHANNEL', 'null'),
    'trace' => env('LOG_DEPRECATIONS_TRACE', false),
],

'channels' => [
    // ...
]
```

Lub możesz zdefiniować kanał logów o nazwie `deprecations`. Jeśli kanał logów o tej nazwie istnieje, zawsze będzie używany do logowania przestarzałych funkcji:

```php
'channels' => [
    'deprecations' => [
        'driver' => 'single',
        'path' => storage_path('logs/php-deprecation-warnings.log'),
    ],
],
```

<a name="building-log-stacks"></a>
## Budowanie stosów logów

Jak wspomniano wcześniej, sterownik `stack` umożliwia łączenie wielu kanałów w jeden kanał logów dla wygody. Aby zilustrować, jak używać stosów logów, spójrzmy na przykładową konfigurację, którą możesz zobaczyć w aplikacji produkcyjnej:

```php
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['syslog', 'slack'], // [tl! add]
        'ignore_exceptions' => false,
    ],

    'syslog' => [
        'driver' => 'syslog',
        'level' => env('LOG_LEVEL', 'debug'),
        'facility' => env('LOG_SYSLOG_FACILITY', LOG_USER),
        'replace_placeholders' => true,
    ],

    'slack' => [
        'driver' => 'slack',
        'url' => env('LOG_SLACK_WEBHOOK_URL'),
        'username' => env('LOG_SLACK_USERNAME', 'Laravel Log'),
        'emoji' => env('LOG_SLACK_EMOJI', ':boom:'),
        'level' => env('LOG_LEVEL', 'critical'),
        'replace_placeholders' => true,
    ],
],
```

Przeanalizujmy tę konfigurację. Po pierwsze, zauważ, że nasz kanał `stack` agreguje dwa inne kanały za pomocą swojej opcji `channels`: `syslog` i `slack`. Więc podczas logowania komunikatów, oba te kanały będą miały możliwość zalogowania komunikatu. Jednak, jak zobaczymy poniżej, to czy te kanały faktycznie zalogują komunikat, może być określone przez wagę / "poziom" komunikatu.

<a name="log-levels"></a>
#### Poziomy logów

Zwróć uwagę na opcję konfiguracyjną `level` obecną w konfiguracjach kanałów `syslog` i `slack` w powyższym przykładzie. Ta opcja określa minimalny "poziom", jaki musi mieć komunikat, aby został zalogowany przez kanał. Monolog, który zasila usługi logowania Laravel, oferuje wszystkie poziomy logów zdefiniowane w [specyfikacji RFC 5424](https://tools.ietf.org/html/rfc5424). W malejącej kolejności wagi, te poziomy logów to: **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** i **debug**.

Więc wyobraź sobie, że logujemy komunikat za pomocą metody `debug`:

```php
Log::debug('An informational message.');
```

Biorąc pod uwagę naszą konfigurację, kanał `syslog` zapisze komunikat do dziennika systemu; jednak ponieważ komunikat o błędzie nie jest na poziomie `critical` lub wyższym, nie zostanie wysłany do Slacka. Jednak jeśli zalogujemy komunikat `emergency`, zostanie wysłany zarówno do dziennika systemu, jak i do Slacka, ponieważ poziom `emergency` jest powyżej naszego minimalnego progu poziomu dla obu kanałów:

```php
Log::emergency('The system is down!');
```

<a name="writing-log-messages"></a>
## Zapisywanie komunikatów logów

Możesz zapisywać informacje do logów za pomocą [fasady](/docs/{{version}}/facades) `Log`. Jak wspomniano wcześniej, logger zapewnia osiem poziomów logowania zdefiniowanych w [specyfikacji RFC 5424](https://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** i **debug**:

```php
use Illuminate\Support\Facades\Log;

Log::emergency($message);
Log::alert($message);
Log::critical($message);
Log::error($message);
Log::warning($message);
Log::notice($message);
Log::info($message);
Log::debug($message);
```

Możesz wywołać dowolną z tych metod, aby zalogować komunikat dla odpowiedniego poziomu. Domyślnie komunikat zostanie zapisany do domyślnego kanału logów zgodnie z konfiguracją w pliku konfiguracyjnym `logging`:

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Support\Facades\Log;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     */
    public function show(string $id): View
    {
        Log::info('Showing the user profile for user: {id}', ['id' => $id]);

        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

<a name="contextual-information"></a>
### Informacje kontekstowe

Tablica danych kontekstowych może być przekazana do metod logów. Te dane kontekstowe zostaną sformatowane i wyświetlone wraz z komunikatem logów:

```php
use Illuminate\Support\Facades\Log;

Log::info('User {id} failed to login.', ['id' => $user->id]);
```

Czasami możesz chcieć określić pewne informacje kontekstowe, które powinny być uwzględnione we wszystkich kolejnych wpisach logów w konkretnym kanale. Na przykład możesz chcieć zalogować identyfikator żądania powiązany z każdym przychodzącym żądaniem do Twojej aplikacji. Aby to osiągnąć, możesz wywołać metodę `withContext` fasady `Log`:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;

class AssignRequestId
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        $requestId = (string) Str::uuid();

        Log::withContext([
            'request-id' => $requestId
        ]);

        $response = $next($request);

        $response->headers->set('Request-Id', $requestId);

        return $response;
    }
}
```

Jeśli chcesz udostępnić informacje kontekstowe _wszystkim_ kanałom logowania, możesz wywołać metodę `Log::shareContext()`. Ta metoda dostarczy informacje kontekstowe do wszystkich utworzonych kanałów i wszelkich kanałów, które zostaną utworzone później:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;

class AssignRequestId
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        $requestId = (string) Str::uuid();

        Log::shareContext([
            'request-id' => $requestId
        ]);

        // ...
    }
}
```

> [!NOTE]
> Jeśli musisz udostępnić kontekst logów podczas przetwarzania zadań w kolejce, możesz skorzystać z [middleware zadań](/docs/{{version}}/queues#job-middleware).

<a name="writing-to-specific-channels"></a>
### Zapisywanie do konkretnych kanałów

Czasami możesz chcieć zalogować komunikat do kanału innego niż domyślny kanał Twojej aplikacji. Możesz użyć metody `channel` w fasadzie `Log`, aby pobrać i zalogować do dowolnego kanału zdefiniowanego w Twoim pliku konfiguracyjnym:

```php
use Illuminate\Support\Facades\Log;

Log::channel('slack')->info('Something happened!');
```

Jeśli chcesz utworzyć stos logowania na żądanie, składający się z wielu kanałów, możesz użyć metody `stack`:

```php
Log::stack(['single', 'slack'])->info('Something happened!');
```

<a name="on-demand-channels"></a>
#### Kanały na żądanie

Możliwe jest również utworzenie kanału na żądanie, dostarczając konfigurację w czasie wykonywania bez konieczności obecności tej konfiguracji w pliku konfiguracyjnym `logging` Twojej aplikacji. Aby to osiągnąć, możesz przekazać tablicę konfiguracyjną do metody `build` fasady `Log`:

```php
use Illuminate\Support\Facades\Log;

Log::build([
  'driver' => 'single',
  'path' => storage_path('logs/custom.log'),
])->info('Something happened!');
```

Możesz również chcieć uwzględnić kanał na żądanie w stosie logowania na żądanie. Można to osiągnąć, włączając instancję Twojego kanału na żądanie w tablicy przekazanej do metody `stack`:

```php
use Illuminate\Support\Facades\Log;

$channel = Log::build([
  'driver' => 'single',
  'path' => storage_path('logs/custom.log'),
]);

Log::stack(['slack', $channel])->info('Something happened!');
```

<a name="monolog-channel-customization"></a>
## Dostosowywanie kanałów Monolog

<a name="customizing-monolog-for-channels"></a>
### Dostosowywanie Monolog dla kanałów

Czasami możesz potrzebować pełnej kontroli nad konfiguracją Monolog dla istniejącego kanału. Na przykład możesz chcieć skonfigurować niestandardową implementację `FormatterInterface` Monolog dla wbudowanego kanału `single` Laravel.

Aby rozpocząć, zdefiniuj tablicę `tap` w konfiguracji kanału. Tablica `tap` powinna zawierać listę klas, które powinny mieć możliwość dostosowania (lub "tap into") instancji Monolog po jej utworzeniu. Nie ma konwencjonalnej lokalizacji, w której te klasy powinny być umieszczone, więc możesz utworzyć katalog w swojej aplikacji, aby zawierał te klasy:

```php
'single' => [
    'driver' => 'single',
    'tap' => [App\Logging\CustomizeFormatter::class],
    'path' => storage_path('logs/laravel.log'),
    'level' => env('LOG_LEVEL', 'debug'),
    'replace_placeholders' => true,
],
```

Po skonfigurowaniu opcji `tap` na swoim kanale, jesteś gotowy do zdefiniowania klasy, która dostosuje Twoją instancję Monolog. Ta klasa potrzebuje tylko jednej metody: `__invoke`, która otrzymuje instancję `Illuminate\Log\Logger`. Instancja `Illuminate\Log\Logger` przekazuje wszystkie wywołania metod do podstawowej instancji Monolog:

```php
<?php

namespace App\Logging;

use Illuminate\Log\Logger;
use Monolog\Formatter\LineFormatter;

class CustomizeFormatter
{
    /**
     * Customize the given logger instance.
     */
    public function __invoke(Logger $logger): void
    {
        foreach ($logger->getHandlers() as $handler) {
            $handler->setFormatter(new LineFormatter(
                '[%datetime%] %channel%.%level_name%: %message% %context% %extra%'
            ));
        }
    }
}
```

> [!NOTE]
> Wszystkie Twoje klasy "tap" są rozwiązywane przez [kontener usług](/docs/{{version}}/container), więc wszelkie zależności konstruktora, których wymagają, zostaną automatycznie wstrzyknięte.

<a name="creating-monolog-handler-channels"></a>
### Tworzenie kanałów obsługi Monolog

Monolog ma różnorodne [dostępne programy obsługi](https://github.com/Seldaek/monolog/tree/main/src/Monolog/Handler), a Laravel nie zawiera wbudowanego kanału dla każdego z nich. W niektórych przypadkach możesz chcieć utworzyć niestandardowy kanał, który jest jedynie instancją konkretnego programu obsługi Monolog, który nie ma odpowiadającego sterownika logów Laravel. Te kanały można łatwo utworzyć za pomocą sterownika `monolog`.

Podczas używania sterownika `monolog`, opcja konfiguracyjna `handler` służy do określenia, który program obsługi zostanie utworzony. Opcjonalnie, wszelkie parametry konstruktora, których potrzebuje program obsługi, można określić za pomocą opcji konfiguracyjnej `handler_with`:

```php
'logentries' => [
    'driver'  => 'monolog',
    'handler' => Monolog\Handler\SyslogUdpHandler::class,
    'handler_with' => [
        'host' => 'my.logentries.internal.datahubhost.company.com',
        'port' => '10000',
    ],
],
```

<a name="monolog-formatters"></a>
#### Formatery Monolog

Podczas używania sterownika `monolog`, Monolog `LineFormatter` będzie używany jako domyślny formater. Możesz jednak dostosować typ formatera przekazywanego do programu obsługi za pomocą opcji konfiguracyjnych `formatter` i `formatter_with`:

```php
'browser' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\BrowserConsoleHandler::class,
    'formatter' => Monolog\Formatter\HtmlFormatter::class,
    'formatter_with' => [
        'dateFormat' => 'Y-m-d',
    ],
],
```

Jeśli używasz programu obsługi Monolog, który jest w stanie dostarczyć własny formater, możesz ustawić wartość opcji konfiguracyjnej `formatter` na `default`:

```php
'newrelic' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\NewRelicHandler::class,
    'formatter' => 'default',
],
```

<a name="monolog-processors"></a>
#### Procesory Monolog

Monolog może również przetwarzać komunikaty przed ich zalogowaniem. Możesz tworzyć własne procesory lub używać [istniejących procesorów oferowanych przez Monolog](https://github.com/Seldaek/monolog/tree/main/src/Monolog/Processor).

Jeśli chcesz dostosować procesory dla sterownika `monolog`, dodaj wartość konfiguracyjną `processors` do konfiguracji Twojego kanału:

```php
'memory' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\StreamHandler::class,
    'handler_with' => [
        'stream' => 'php://stderr',
    ],
    'processors' => [
        // Simple syntax...
        Monolog\Processor\MemoryUsageProcessor::class,

        // With options...
        [
            'processor' => Monolog\Processor\PsrLogMessageProcessor::class,
            'with' => ['removeUsedContextFields' => true],
        ],
    ],
],
```

<a name="creating-custom-channels-via-factories"></a>
### Tworzenie niestandardowych kanałów za pomocą fabryk

Jeśli chcesz zdefiniować całkowicie niestandardowy kanał, w którym masz pełną kontrolę nad tworzeniem i konfiguracją Monolog, możesz określić typ sterownika `custom` w swoim pliku konfiguracyjnym `config/logging.php`. Twoja konfiguracja powinna zawierać opcję `via`, która zawiera nazwę klasy fabryki, która zostanie wywołana w celu utworzenia instancji Monolog:

```php
'channels' => [
    'example-custom-channel' => [
        'driver' => 'custom',
        'via' => App\Logging\CreateCustomLogger::class,
    ],
],
```

Po skonfigurowaniu kanału sterownika `custom`, jesteś gotowy do zdefiniowania klasy, która utworzy Twoją instancję Monolog. Ta klasa potrzebuje tylko jednej metody `__invoke`, która powinna zwrócić instancję loggera Monolog. Metoda otrzyma tablicę konfiguracyjną kanałów jako jedyny argument:

```php
<?php

namespace App\Logging;

use Monolog\Logger;

class CreateCustomLogger
{
    /**
     * Create a custom Monolog instance.
     */
    public function __invoke(array $config): Logger
    {
        return new Logger(/* ... */);
    }
}
```

<a name="tailing-log-messages-using-pail"></a>
## Śledzenie komunikatów logów za pomocą Pail

Często możesz potrzebować śledzić logi swojej aplikacji w czasie rzeczywistym. Na przykład podczas debugowania problemu lub monitorowania logów aplikacji pod kątem określonych typów błędów.

Laravel Pail to pakiet, który pozwala łatwo zagłębić się w pliki logów aplikacji Laravel bezpośrednio z wiersza poleceń. W przeciwieństwie do standardowego polecenia `tail`, Pail jest zaprojektowany do pracy z dowolnym sterownikiem logów, w tym Sentry czy Flare. Ponadto Pail zapewnia zestaw przydatnych filtrów, które pomogą Ci szybko znaleźć to, czego szukasz.

<img src="https://laravel.com/img/docs/pail-example.png">

<a name="pail-installation"></a>
### Instalacja

> [!WARNING]
> Laravel Pail wymaga rozszerzenia PHP [PCNTL](https://www.php.net/manual/en/book.pcntl.php).

Aby rozpocząć, zainstaluj Pail w swoim projekcie za pomocą menedżera pakietów Composer:

```shell
composer require --dev laravel/pail
```

<a name="pail-usage"></a>
### Użycie

Aby rozpocząć śledzenie logów, uruchom polecenie `pail`:

```shell
php artisan pail
```

Aby zwiększyć szczegółowość wyjścia i uniknąć skracania (…), użyj opcji `-v`:

```shell
php artisan pail -v
```

Dla maksymalnej szczegółowości i wyświetlania śladów stosu wyjątków, użyj opcji `-vv`:

```shell
php artisan pail -vv
```

Aby zatrzymać śledzenie logów, naciśnij `Ctrl+C` w dowolnym momencie.

<a name="pail-filtering-logs"></a>
### Filtrowanie logów

<a name="pail-filtering-logs-filter-option"></a>
#### `--filter`

Możesz użyć opcji `--filter`, aby filtrować logi według ich typu, pliku, komunikatu i zawartości śladu stosu:

```shell
php artisan pail --filter="QueryException"
```

<a name="pail-filtering-logs-message-option"></a>
#### `--message`

Aby filtrować logi tylko według ich komunikatu, możesz użyć opcji `--message`:

```shell
php artisan pail --message="User created"
```

<a name="pail-filtering-logs-level-option"></a>
#### `--level`

Opcja `--level` może być użyta do filtrowania logów według ich [poziomu logów](#log-levels):

```shell
php artisan pail --level=error
```

<a name="pail-filtering-logs-user-option"></a>
#### `--user`

Aby wyświetlać tylko logi, które zostały zapisane, gdy dany użytkownik był uwierzytelniony, możesz podać identyfikator użytkownika do opcji `--user`:

```shell
php artisan pail --user=1
```
