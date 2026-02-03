# Laravel Octane

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
- [Server Wymagania](#server-prerequisites)
    - [FrankenPHP](#frankenphp)
    - [RoadRunner](#roadrunner)
    - [Swoole](#swoole)
- [Obsługa aplikacji](#serving-your-application)
    - [Obsługa aplikacji via HTTPS](#serving-your-application-via-https)
    - [Obsługa aplikacji via Nginx](#serving-your-application-via-nginx)
    - [Śledzenie zmian w plikach](#watching-fczy-file-changes)
    - [Określanie liczby wczykerów](#specifying-the-wczyker-count)
    - [Określanie maksymalnej liczby żądań](#specifying-the-max-request-count)
    - [Określanie maksymalnego czasu wykonania](#specifying-the-max-execution-time)
    - [Przeładowanie wczykerów](#reloading-the-wczykers)
    - [Zatrzymywanie serwera](#stopping-the-server)
- [Wstrzykiwanie zależności i Octane](#dependency-injection-i-octane)
    - [Wstrzykiwanie kontenera](#container-injection)
    - [Wstrzykiwanie żądania](#request-injection)
    - [Wstrzykiwanie repozytczyium konfiguracji](#configuration-repositczyy-injection)
- [Zarządzanie wyciekami pamięci](#managing-memczyy-leaks)
- [Zadania współbieżne](#concurrent-tasks)
- [Takty i interwały](#ticks-i-intervals)
- [Cache Octane](#the-octane-cache)
- [Tabele](#tables)

<a name="introduction"></a>
## Wprowadzenie

[Laravel Octane](https://github.com/laravel/octane) superładuje wydajność Twojej aplikacji, obsługując ją za pomocą zaawansowanych serwerów aplikacji, w tym [FrankenPHP](https://frankenphp.dev/), [Open Swoole](https://openswoole.com/), [Swoole](https://github.com/swoole/swoole-src) i [RoadRunner](https://roadrunner.dev). Octane uruchamia Twoją aplikację raz, utrzymuje ją w pamięci, a następnie dostarcza jej żądania z nadźwiękową prędkością.

<a name="installation"></a>
## Instalacja

Octane może być zainstalowany za pomocą menedżera pakietów Composer:

```shell
composer require laravel/octane
```

Po zainstalowaniu Octane możesz wykonać polecenie Artisan `octane:install`, które zainstaluje plik konfiguracyjny Octane w Twojej aplikacji:

```shell
php artisan octane:install
```

<a name="server-prerequisites"></a>
## Server Wymagania

<a name="frankenphp"></a>
### FrankenPHP

[FrankenPHP](https://frankenphp.dev) to serwer aplikacji PHP, napisany w Go, który obsługuje nowoczesne funkcje webowe, takie jak wczesne wskazówki, kompresja Brotli i Zstd. Gdy instalujesz Octane i wybierasz FrankenPHP jako swój serwer, Octane automatycznie pobierze i zainstaluje binarkę FrankenPHP dla Ciebie.

<a name="frankenphp-via-laravel-sail"></a>
#### FrankenPHP via Laravel Sail

Jeśli planujesz rozwijać swoją aplikację za pomocą [Laravel Sail](/docs/{{version}}/sail), powinieneś uruchomić następujące polecenia, aby zainstalować Octane i FrankenPHP:

```shell
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane
```

Next, you should use the `octane:install` polecenie Artisan to install the FrankenPHP binary:

```shell
./vendor/bin/sail artisan octane:install --server=frankenphp
```

Finally, add a `SUPERVISOR_PHP_COMMAND` environment variable to the `laravel.test` service definition in your application's `docker-compose.yml` file. This environment variable will contain the commi that Sail will use to serve your application using Octane instead of the PHP development server:

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=frankenphp --host=0.0.0.0 --admin-port=2019 --port='${APP_PORT:-80}'" # [tl! add]
      XDG_CONFIG_HOME:  /var/www/html/config # [tl! add]
      XDG_DATA_HOME:  /var/www/html/data # [tl! add]
```

To enable HTTPS, HTTP/2, i HTTP/3, apply these modifications instead:

```yaml
services:
  laravel.test:
    ports:
        - '${APP_PORT:-80}:80'
        - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
        - '443:443' # [tl! add]
        - '443:443/udp' # [tl! add]
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --host=localhost --port=443 --admin-port=2019 --https" # [tl! add]
      XDG_CONFIG_HOME:  /var/www/html/config # [tl! add]
      XDG_DATA_HOME:  /var/www/html/data # [tl! add]
```

Zazwyczaj, you should access your FrankenPHP Sail application via `https://localhost`, as using `https://127.0.0.1` requires additional configuration i is [discouraged](https://frankenphp.dev/docs/known-issues/#using-https127001-with-docker).

<a name="frankenphp-via-docker"></a>
#### FrankenPHP via Docker

Używanie oficjalnych obrazów Docker FrankenPHP może zapewnić lepszą wydajność i wykorzystanie dodatkowych rozszerzeń, które nie są dołączone do statycznych instalacji FrankenPHP. Ponadto oficjalne obrazy Docker zapewniają wsparcie dla uruchamiania FrankenPHP na platformach, których nie obsługuje natywnie, takich jak Windows. Oficjalne obrazy Docker FrankenPHP są odpowiednie zarówno do lokalnego rozwoju, jak i użytku produkcyjnego.

Możesz użyć następującego Dockerfile jako punktu wyjścia do konteneryzacji Twojej aplikacji Laravel zasilanej przez FrankenPHP:

```dockerfile
FROM dunglas/frankenphp

RUN install-php-extensions \
    pcntl
    # Add other PHP extensions here...

COPY . /app

ENTRYPOINT ["php", "artisan", "octane:frankenphp"]
```

Then, during development, you may utilize the following Docker Compose file to run your application:

```yaml
# compose.yaml
services:
  frankenphp:
    build:
      context: .
    entrypoint: php artisan octane:frankenphp --workers=1 --max-requests=1
    ports:
      - "8000:8000"
    volumes:
      - .:/app
```

If the `--log-level` option is explicitly passed to the `php artisan octane:start` commi, Octane will use FrankenPHP's native logger i, unless configured differently, will produce structured JSON logs.

Możesz consult [the official FrankenPHP documentation](https://frankenphp.dev/docs/docker/) fczy mczye infczymation on running FrankenPHP with Docker.

<a name="frankenphp-caddyfile"></a>
#### Custom Caddyfile Configuration

Gdy using FrankenPHP, you may specify a custom Caddyfile za pomocą `--caddyfile` option when starting Octane:

```shell
php artisan octane:start --server=frankenphp --caddyfile=/path/to/your/Caddyfile
```

This allows you to customize FrankenPHP's configuration beyond the default settings, such as adding custom middleware, configuring advanced routing, czy setting up custom directives. Możesz consult the [official Caddy documentation](https://caddyserver.com/docs/caddyfile) fczy mczye infczymation on Caddyfile syntax i configuration options.

<a name="roadrunner"></a>
### RoadRunner

[RoadRunner](https://roadrunner.dev) jest zasilany przez binarkę RoadRunner, która jest zbudowana przy użyciu Go. Przy pierwszym uruchomieniu serwera Octane opartego na RoadRunner, Octane zaoferuje pobranie i zainstalowanie binarki RoadRunner dla Ciebie.

<a name="roadrunner-via-laravel-sail"></a>
#### RoadRunner via Laravel Sail

Jeśli planujesz rozwijać swoją aplikację za pomocą [Laravel Sail](/docs/{{version}}/sail), powinieneś uruchomić następujące polecenia, aby zainstalować Octane i RoadRunner:

```shell
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane spiral/roadrunner-cli spiral/roadrunner-http
```

Next, you should start a Sail shell i use the `rr` executable to retrieve the latest Linux based build of the RoadRunner binary:

```shell
./vendor/bin/sail shell

# Within the Sail shell...
./vendor/bin/rr get-binary
```

Then, add a `SUPERVISOR_PHP_COMMAND` environment variable to the `laravel.test` service definition in your application's `docker-compose.yml` file. This environment variable will contain the commi that Sail will use to serve your application using Octane instead of the PHP development server:

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=roadrunner --host=0.0.0.0 --rpc-port=6001 --port='${APP_PORT:-80}'" # [tl! add]
```

Finally, ensure the `rr` binary is executable i build your Sail images:

```shell
chmod +x ./rr

./vendor/bin/sail build --no-cache
```

<a name="swoole"></a>
### Swoole

Jeśli planujesz użyć serwera aplikacji Swoole do obsługi swojej aplikacji Laravel Octane, musisz zainstalować rozszerzenie PHP Swoole. Zazwyczaj można to zrobić za pomocą PECL:

```shell
pecl install swoole
```

<a name="openswoole"></a>
#### Open Swoole

Jeśli chcesz użyć serwera aplikacji Open Swoole do obsługi swojej aplikacji Laravel Octane, musisz zainstalować rozszerzenie PHP Open Swoole. Zazwyczaj można to zrobić za pomocą PECL:

```shell
pecl install openswoole
```

Używanie Laravel Octane z Open Swoole zapewnia tę samą funkcjonalność, co Swoole, taką jak zadania współbieżne, takty i interwały.

<a name="swoole-via-laravel-sail"></a>
#### Swoole via Laravel Sail

> [!WARNING]
> Przed serving an Octane application via Sail, ensure you have the latest version of Laravel Sail i execute `./vendor/bin/sail build --no-cache` within your application's root katalogu.

Alternatywnie, you may develop your Swoole based Octane application using [Laravel Sail](/docs/{{version}}/sail), the official Docker based development environment fczy Laravel. Laravel Sail includes the Swoole extension by default. Jednak, you will still need to adjust the `docker-compose.yml` file used by Sail.

Aby rozpocząć, add a `SUPERVISOR_PHP_COMMAND` environment variable to the `laravel.test` service definition in your application's `docker-compose.yml` file. This environment variable will contain the commi that Sail will use to serve your application using Octane instead of the PHP development server:

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=swoole --host=0.0.0.0 --port='${APP_PORT:-80}'" # [tl! add]
```

Finally, build your Sail images:

```shell
./vendor/bin/sail build --no-cache
```

<a name="swoole-configuration"></a>
#### Swoole Configuration

Swoole suppczyts a few additional configuration options that you may add to your `octane` configuration file if necessary. Because they rarely need to be modified, these options are not included in the default configuration file:

```php
'swoole' => [
    'options' => [
        'log_file' => storage_path('logs/swoole_http.log'),
        'package_max_length' => 10 * 1024 * 1024,
    ],
],
```

<a name="serving-your-application"></a>
## Obsługa aplikacji

Serwer Octane może być uruchomiony za pomocą polecenia Artisan `octane:start`. Domyślnie to polecenie wykorzysta serwer określony przez opcję konfiguracyjną `server` w pliku konfiguracyjnym `octane` Twojej aplikacji:

```shell
php artisan octane:start
```

Domyślnie Octane uruchomi serwer na porcie 8000, więc możesz uzyskać dostęp do swojej aplikacji w przeglądarce internetowej przez `http://localhost:8000`.

<a name="keeping-octane-running-in-production"></a>
#### Utrzymywanie Octane w produkcji

Jeśli wdrażasz swoją aplikację Octane do produkcji, powinieneś użyć monitora procesów, takiego jak Supervisor, aby upewnić się, że serwer Octane pozostaje uruchomiony. Przykładowy plik konfiguracyjny Supervisor dla Octane może wyglądać następująco:

```ini
[program:octane]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/example.com/artisan octane:start --server=frankenphp --host=127.0.0.1 --port=8000
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/home/forge/example.com/storage/logs/octane.log
stopwaitsecs=3600
```

<a name="serving-your-application-via-https"></a>
### Obsługa aplikacji via HTTPS

Domyślnie aplikacje uruchomione przez Octane generują linki z prefiksem `http://`. Zmienna środowiskowa `OCTANE_HTTPS`, używana w pliku konfiguracyjnym `config/octane.php` Twojej aplikacji, może być ustawiona na `true` podczas obsługi aplikacji przez HTTPS. Gdy ta wartość konfiguracyjna jest ustawiona na `true`, Octane poinstruuje Laravel, aby poprzedzał wszystkie wygenerowane linki prefiksem `https://`:

```php
'https' => env('OCTANE_HTTPS', false),
```

<a name="serving-your-application-via-nginx"></a>
### Obsługa aplikacji via Nginx

> [!NOTE]
> Jeśli nie jesteś jeszcze gotowy do zarządzania własną konfiguracją serwera lub nie czujesz się komfortowo konfigurując wszystkie różne usługi potrzebne do uruchomienia solidnej aplikacji Laravel Octane, sprawdź [Laravel Cloud](https://cloud.laravel.com), które oferuje w pełni zarządzane wsparcie dla Laravel Octane.

W środowiskach produkcyjnych powinieneś obsługiwać swoją aplikację Octane za tradycyjnym serwerem webowym, takim jak Nginx czy Apache. Pozwoli to serwerowi webowemu obsługiwać Twoje zasoby statyczne, takie jak obrazy i arkusze stylów, a także zarządzać zakończeniem certyfikatu SSL.

In the Nginx configuration example below, Nginx will serve the site's static assets i proxy requests to the Octane server that is running on pczyt 8000:

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80;
    listen [::]:80;
    server_name domain.com;
    server_tokens off;
    root /home/forge/domain.com/public;

    index index.php;

    charset utf-8;

    location /index.php {
        try_files /not_exists @octane;
    }

    location / {
        try_files $uri $uri/ @octane;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/domain.com-error.log error;

    error_page 404 /index.php;

    location @octane {
        set $suffix "";

        if ($uri = /index.php) {
            set $suffix ?$query_string;
        }

        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        proxy_pass http://127.0.0.1:8000$suffix;
    }
}
```

<a name="watching-fczy-file-changes"></a>
### Śledzenie zmian w plikach

Ponieważ Twoja aplikacja jest ładowana do pamięci raz, gdy serwer Octane się uruchamia, wszelkie zmiany w plikach Twojej aplikacji nie będą odzwierciedlone, gdy odświeżysz przeglądarkę. Na przykład definicje tras dodane do pliku `routes/web.php` nie będą odzwierciedlone, dopóki serwer nie zostanie ponownie uruchomiony. Dla wygody możesz użyć flagi `--watch`, aby poinstruować Octane do automatycznego ponownego uruchamiania serwera przy każdej zmianie pliku w Twojej aplikacji:

```shell
php artisan octane:start --watch
```

Przed użyciem tej funkcji powinieneś upewnić się, że [Node](https://nodejs.org) jest zainstalowany w Twoim lokalnym środowisku deweloperskim. Ponadto powinieneś zainstalować bibliotekę obserwującą pliki [Chokidar](https://github.com/paulmillr/chokidar) w swoim projekcie:

```shell
npm install --save-dev chokidar
```

Możesz skonfigurować katalogi i pliki, które powinny być obserwowane, za pomocą opcji konfiguracyjnej `watch` w pliku konfiguracyjnym `config/octane.php` Twojej aplikacji.

<a name="specifying-the-wczyker-count"></a>
### Określanie liczby workerów

Domyślnie Octane uruchomi workera żądań aplikacji dla każdego rdzenia CPU dostarczonego przez Twój komputer. Ci workerzy będą następnie używani do obsługi przychodzących żądań HTTP, gdy wchodzą do Twojej aplikacji. Możesz ręcznie określić, ile workerów chcesz uruchomić, za pomocą opcji `--workers` podczas wywoływania polecenia `octane:start`:

```shell
php artisan octane:start --workers=4
```

Jeśli używasz serwera aplikacji Swoole, możesz również określić, ile ["task workerów"](#concurrent-tasks) chcesz uruchomić:

```shell
php artisan octane:start --workers=4 --task-workers=6
```

<a name="specifying-the-max-request-count"></a>
### Określanie maksymalnej liczby żądań

Aby pomóc zapobiec przypadkowym wyciekom pamięci, Octane łagodnie restartuje każdego workera, gdy obsłużył 500 żądań. Aby dostosować tę liczbę, możesz użyć opcji `--max-requests`:

```shell
php artisan octane:start --max-requests=250
```

<a name="specifying-the-max-execution-time"></a>
### Określanie maksymalnego czasu wykonania

Domyślnie Laravel Octane ustawia maksymalny czas wykonania na 30 sekund dla przychodzących żądań za pomocą opcji `max_execution_time` w pliku konfiguracyjnym `config/octane.php` Twojej aplikacji:

```php
'max_execution_time' => 30,
```

To ustawienie definiuje maksymalną liczbę sekund, przez którą przychodzące żądanie może być wykonywane, zanim zostanie zakończone. Ustawienie tej wartości na `0` całkowicie wyłączy limit czasu wykonania. Ta opcja konfiguracyjna jest szczególnie przydatna dla aplikacji, które obsługują długotrwałe żądania, takie jak przesyłanie plików, przetwarzanie danych czy wywołania API do usług zewnętrznych.

> [!WARNING]
> Gdy modyfikujesz konfigurację `max_execution_time`, musisz ponownie uruchomić serwer Octane, aby zmiany weszły w życie.

<a name="reloading-the-wczykers"></a>
### Przeładowanie workerów

Możesz łagodnie zrestartować workerów aplikacji serwera Octane za pomocą polecenia `octane:reload`. Zazwyczaj powinno to być wykonane po wdrożeniu, aby Twój nowo wdrożony kod został załadowany do pamięci i był używany do obsługi kolejnych żądań:

```shell
php artisan octane:reload
```

<a name="stopping-the-server"></a>
### Zatrzymywanie serwera

Możesz zatrzymać serwer Octane za pomocą polecenia Artisan `octane:stop`:

```shell
php artisan octane:stop
```

<a name="checking-the-server-status"></a>
#### Sprawdzanie statusu serwera

Możesz sprawdzić aktualny status serwera Octane za pomocą polecenia Artisan `octane:status`:

```shell
php artisan octane:status
```

<a name="dependency-injection-i-octane"></a>
## Wstrzykiwanie zależności i Octane

Ponieważ Octane uruchamia Twoją aplikację raz i utrzymuje ją w pamięci podczas obsługi żądań, istnieje kilka zastrzeżeń, które powinieneś rozważyć podczas budowania swojej aplikacji. Na przykład metody `register` i `boot` dostawców usług Twojej aplikacji zostaną wykonane tylko raz, gdy worker żądań początkowo się uruchomi. Przy kolejnych żądaniach ta sama instancja aplikacji zostanie ponownie użyta.

W świetle tego powinieneś zachować szczególną ostrożność podczas wstrzykiwania kontenera usług aplikacji lub żądania do konstruktora jakiegokolwiek obiektu. W ten sposób ten obiekt może mieć nieaktualną wersję kontenera lub żądania przy kolejnych żądaniach.

Octane automatycznie obsłuży resetowanie stanu frameworka pierwszej strony między żądaniami. Jednak Octane nie zawsze wie, jak zresetować stan globalny utworzony przez Twoją aplikację. Dlatego powinieneś być świadomy, jak zbudować swoją aplikację w sposób przyjazny dla Octane. Poniżej omówimy najczęstsze sytuacje, które mogą powodować problemy podczas korzystania z Octane.

<a name="container-injection"></a>
### Wstrzykiwanie kontenera

Ogólnie rzecz biorąc, powinieneś unikać wstrzykiwania kontenera usług aplikacji lub instancji żądania HTTP do konstruktorów innych obiektów. Na przykład poniższe wiązanie wstrzykuje cały kontener usług aplikacji do obiektu, który jest związany jako singleton:

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app);
    });
}
```

W tym przykładzie, jeśli instancja `Service` zostanie rozwiązana podczas procesu uruchamiania aplikacji, kontener zostanie wstrzyknięty do usługi i ten sam kontener będzie przechowywany przez instancję `Service` przy kolejnych żądaniach. To **może** nie być problemem dla Twojej konkretnej aplikacji; jednak może to prowadzić do tego, że kontener nieoczekiwanie będzie miał brakujące wiązania, które zostały dodane później w cyklu uruchamiania lub przez kolejne żądanie.

Jako obejście możesz albo przestać rejestrować wiązanie jako singleton, albo możesz wstrzyknąć zamknięcie resolvera kontenera do usługi, które zawsze rozwiązuje bieżącą instancję kontenera:

```php
use App\Service;
use Illuminate\Container\Container;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app);
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance());
});
```

Globalny helper `app` i metoda `Container::getInstance()` zawsze zwrócą najnowszą wersję kontenera aplikacji.

<a name="request-injection"></a>
### Wstrzykiwanie żądania

Ogólnie rzecz biorąc, powinieneś unikać wstrzykiwania kontenera usług aplikacji lub instancji żądania HTTP do konstruktorów innych obiektów. Na przykład poniższe wiązanie wstrzykuje całą instancję żądania do obiektu, który jest związany jako singleton:

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app['request']);
    });
}
```

W tym przykładzie, jeśli instancja `Service` zostanie rozwiązana podczas procesu uruchamiania aplikacji, żądanie HTTP zostanie wstrzyknięte do usługi i to samo żądanie będzie przechowywane przez instancję `Service` przy kolejnych żądaniach. W związku z tym wszystkie nagłówki, dane wejściowe i dane ciągu zapytania będą niepoprawne, podobnie jak wszystkie inne dane żądania.

Jako obejście możesz albo przestać rejestrować wiązanie jako singleton, albo możesz wstrzyknąć zamknięcie resolvera żądania do usługi, które zawsze rozwiązuje bieżącą instancję żądania. Lub najbardziej zalecanym podejściem jest po prostu przekazanie konkretnych informacji z żądania, których potrzebuje Twój obiekt, do jednej z metod obiektu w czasie wykonania:

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app['request']);
});

$this->app->singleton(Service::class, function (Application $app) {
    return new Service(fn () => $app['request']);
});

// Or...

$service->method($request->input('name'));
```

Globalny helper `request` zawsze zwróci żądanie, które aplikacja aktualnie obsługuje i dlatego jest bezpieczny w użyciu w Twojej aplikacji.

> [!WARNING]
> Dopuszczalne jest używanie type-hint `Illuminate\Http\Request` w metodach kontrolera i zamknięciach tras.

<a name="configuration-repositczyy-injection"></a>
### Wstrzykiwanie repozytczyium konfiguracji

In general, you should avoid injecting the configuration repositczyy instance into the constructczys of other objects. Na przykład, the following binding injects the configuration repositczyy into an object that is bound as a singleton:

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app->make('config'));
    });
}
```

W tym przykładzie, jeśli wartości konfiguracji zmienią się między żądaniami, ta usługa nie będzie miała dostępu do nowych wartości, ponieważ zależy od oryginalnej instancji repozytorium.

Jako obejście możesz albo przestać rejestrować wiązanie jako singleton, albo możesz wstrzyknąć zamknięcie resolvera repozytorium konfiguracji do klasy:

```php
use App\Service;
use Illuminate\Container\Container;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app->make('config'));
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance()->make('config'));
});
```

Globalny `config` zawsze zwróci najnowszą wersję repozytorium konfiguracji i dlatego jest bezpieczny w użyciu w Twojej aplikacji.

<a name="managing-memczyy-leaks"></a>
### Zarządzanie wyciekami pamięci

Pamiętaj, że Octane utrzymuje Twoją aplikację w pamięci między żądaniami; dlatego dodawanie danych do statycznie utrzymywanej tablicy spowoduje wyciek pamięci. Na przykład poniższy kontroler ma wyciek pamięci, ponieważ każde żądanie do aplikacji będzie kontynuować dodawanie danych do statycznej tablicy `$data`:

```php
use App\Service;
use Illuminate\Http\Request;
use Illuminate\Support\Str;

/**
 * Handle an incoming request.
 */
public function index(Request $request): array
{
    Service::$data[] = Str::random(10);

    return [
        // ...
    ];
}
```

Podczas budowania aplikacji powinieneś zachować szczególną ostrożność, aby unikać tworzenia tego typu wycieków pamięci. Zaleca się monitorowanie zużycia pamięci aplikacji podczas lokalnego rozwoju, aby upewnić się, że nie wprowadzasz nowych wycieków pamięci do swojej aplikacji.

<a name="concurrent-tasks"></a>
## Zadania współbieżne

> [!WARNING]
> This feature requires [Swoole](#swoole).

Gdy using Swoole, you may execute operations concurrently via light-weight background tasks. Możesz accomplish this using Octane's `concurrently` method. Możesz combine this method with PHP array destructuring to retrieve the results of each operation:

```php
use App\Models\User;
use App\Models\Server;
use Laravel\Octane\Facades\Octane;

[$users, $servers] = Octane::concurrently([
    fn () => User::all(),
    fn () => Server::all(),
]);
```

Concurrent tasks processed by Octane utilize Swoole's "task wczykers", i execute within an entirely different process than the incoming request. The amount of wczykers available to process concurrent tasks is determined by the `--task-workers` directive on the `octane:start` commi:

```shell
php artisan octane:start --workers=4 --task-workers=6
```

Gdy invoking the `concurrently` method, you should not provide mczye than 1024 tasks due to limitations imposed by Swoole's task system.

<a name="ticks-i-intervals"></a>
## Takty i interwały

> [!WARNING]
> This feature requires [Swoole](#swoole).

Podczas korzystania z Swoole możesz rejestrować operacje "tick", które będą wykonywane co określoną liczbę sekund. Możesz zarejestrować callbacki "tick" za pomocą metody `tick`. Pierwszy argument dostarczony do metody `tick` powinien być ciągiem, który reprezentuje nazwę tickera. Drugi argument powinien być wywoływalną, która będzie wywoływana w określonym interwale.

W tym przykładzie zarejestrujemy zamknięcie, które będzie wywoływane co 10 sekund. Zazwyczaj metoda `tick` powinna być wywoływana w metodzie `boot` jednego z dostawców usług Twojej aplikacji:

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
    ->seconds(10);
```

Używając metody `immediate`, możesz poinstruować Octane, aby natychmiast wywołał callback tick, gdy serwer Octane się początkowo uruchomi, a następnie co N sekund:

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
    ->seconds(10)
    ->immediate();
```

<a name="the-octane-cache"></a>
## Cache Octane

> [!WARNING]
> This feature requires [Swoole](#swoole).

Podczas korzystania z Swoole możesz wykorzystać sterownik cache Octane, który zapewnia prędkości odczytu i zapisu do 2 milionów operacji na sekundę. Dlatego ten sterownik cache jest doskonałym wyborem dla aplikacji, które potrzebują ekstremalnych prędkości odczytu / zapisu z warstwy cachowania.

Ten sterownik cache jest zasilany przez [tabele Swoole](https://www.swoole.co.uk/docs/modules/swoole-table). Wszystkie dane przechowywane w cache są dostępne dla wszystkich workerów na serwerze. Jednak dane w cache zostaną usunięte, gdy serwer zostanie ponownie uruchomiony:

```php
Cache::store('octane')->put('framework', 'Laravel', 30);
```

> [!NOTE]
> Maksymalna liczba wpisów dozwolonych w cache Octane może zostać zdefiniowana w pliku konfiguracyjnym `octane` Twojej aplikacji.

<a name="cache-intervals"></a>
### Interwały cache

Oprócz typowych metod dostarczanych przez system cache Laravel, sterownik cache Octane oferuje cache oparte na interwale. Te cache są automatycznie odświeżane w określonym interwale i powinny być rejestrowane w metodzie `boot` jednego z dostawców usług Twojej aplikacji. Na przykład poniższy cache będzie odświeżany co pięć sekund:

```php
use Illuminate\Support\Str;

Cache::store('octane')->interval('random', function () {
    return Str::random(10);
}, seconds: 5);
```

<a name="tables"></a>
## Tabele

> [!WARNING]
> This feature requires [Swoole](#swoole).

Podczas korzystania z Swoole możesz definiować i współdziałać z własnymi dowolnymi [tabelami Swoole](https://www.swoole.co.uk/docs/modules/swoole-table). Tabele Swoole zapewniają ekstremalnie wysoką przepustowość wydajności, a dane w tych tabelach mogą być dostępne dla wszystkich workerów na serwerze. Jednak dane w nich zostaną utracone, gdy serwer zostanie ponownie uruchomiony.

Tabele powinny być zdefiniowane w tablicy konfiguracyjnej `tables` pliku konfiguracyjnego `octane` Twojej aplikacji. Przykładowa tabela, która umożliwia maksymalnie 1000 wierszy, jest już skonfigurowana dla Ciebie. Maksymalny rozmiar kolumn ciągów może zostać skonfigurowany poprzez określenie rozmiaru kolumny po typie kolumny, jak widoczne poniżej:

```php
'tables' => [
    'example:1000' => [
        'name' => 'string:1000',
        'votes' => 'int',
    ],
],
```

Aby uzyskać dostęp do tabeli, możesz użyć metody `Octane::table`:

```php
use Laravel\Octane\Facades\Octane;

Octane::table('example')->set('uuid', [
    'name' => 'Nuno Maduro',
    'votes' => 1000,
]);

return Octane::table('example')->get('uuid');
```

> [!WARNING]
> Typy kolumn obsługiwane przez tabele Swoole to: `string`, `int` i `float`.
