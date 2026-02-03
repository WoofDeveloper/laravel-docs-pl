# Wdrożenie

- [Wprowadzenie](#introduction)
- [Wymagania serwera](#server-requirements)
- [Konfiguracja serwera](#server-configuration)
    - [Nginx](#nginx)
    - [FrankenPHP](#frankenphp)
    - [Uprawnienia do katalogów](#directory-permissions)
- [Optymalizacja](#optimization)
    - [Cachowanie konfiguracji](#optimizing-configuration-loading)
    - [Cachowanie zdarzeń](#caching-events)
    - [Cachowanie tras](#optimizing-route-loading)
    - [Cachowanie widoków](#optimizing-view-loading)
- [Przeładowywanie usług](#reloading-services)
- [Tryb debugowania](#debug-mode)
- [Trasa Health](#the-health-route)
- [Wdrażanie z Laravel Cloud lub Forge](#deploying-with-cloud-or-forge)

<a name="introduction"></a>
## Wprowadzenie

Kiedy jesteś gotowy do wdrożenia swojej aplikacji Laravel na produkcję, istnieje kilka ważnych rzeczy, które możesz zrobić, aby upewnić się, że Twoja aplikacja działa tak wydajnie, jak to możliwe. W tym dokumencie omówimy kilka doskonałych punktów startowych, aby upewnić się, że Twoja aplikacja Laravel jest prawidłowo wdrożona.

<a name="server-requirements"></a>
## Wymagania serwera

Framework Laravel ma kilka wymagań systemowych. Powinieneś upewnić się, że Twój serwer WWW ma następującą minimalną wersję PHP i rozszerzenia:

<div class="content-list" markdown="1">

- PHP >= 8.2
- Ctype PHP Extension
- cURL PHP Extension
- DOM PHP Extension
- Fileinfo PHP Extension
- Filter PHP Extension
- Hash PHP Extension
- Mbstring PHP Extension
- OpenSSL PHP Extension
- PCRE PHP Extension
- PDO PHP Extension
- Session PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension

</div>

<a name="server-configuration"></a>
## Konfiguracja serwera

<a name="nginx"></a>
### Nginx

Jeśli wdrażasz swoją aplikację na serwerze, na którym działa Nginx, możesz użyć następującego pliku konfiguracyjnego jako punktu wyjścia do konfiguracji swojego serwera WWW. Najprawdopodobniej ten plik będzie musiał być dostosowany w zależności od konfiguracji Twojego serwera. **Jeśli chciałbyś uzyskać pomoc w zarządzaniu serwerem, rozważ użycie w pełni zarządzanej platformy Laravel, takiej jak [Laravel Cloud](https://cloud.laravel.com).**

Upewnij się, jak w poniższej konfiguracji, że Twój serwer WWW kieruje wszystkie żądania do pliku `public/index.php` Twojej aplikacji. Nigdy nie powinieneś próbować przenosić pliku `index.php` do katalogu głównego projektu, ponieważ obsługa aplikacji z katalogu głównego projektu spowoduje ujawnienie wielu wrażliwych plików konfiguracyjnych w publicznym Internecie:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    root /srv/example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ ^/index\.php(/|$) {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

<a name="frankenphp"></a>
### FrankenPHP

[FrankenPHP](https://frankenphp.dev/) może być również używany do obsługi aplikacji Laravel. FrankenPHP to nowoczesny serwer aplikacji PHP napisany w Go. Aby obsługiwać aplikację PHP Laravel za pomocą FrankenPHP, możesz po prostu wywołać jego polecenie `php-server`:

```shell
frankenphp php-server -r public/
```

Aby skorzystać z bardziej zaawansowanych funkcji obsługiwanych przez FrankenPHP, takich jak integracja z [Laravel Octane](/docs/{{version}}/octane), HTTP/3, nowoczesna kompresja lub możliwość pakowania aplikacji Laravel jako samodzielnych plików binarnych, zapoznaj się z [dokumentacją Laravel](https://frankenphp.dev/docs/laravel/) FrankenPHP.

<a name="directory-permissions"></a>
### Uprawnienia do katalogów

Laravel będzie musiał zapisywać do katalogów `bootstrap/cache` i `storage`, więc powinieneś upewnić się, że właściciel procesu serwera WWW ma uprawnienia do zapisu w tych katalogach.

<a name="optimization"></a>
## Optymalizacja

Podczas wdrażania aplikacji na produkcję, istnieje wiele plików, które powinny być cachowane, w tym konfiguracja, zdarzenia, trasy i widoki. Laravel zapewnia pojedyncze, wygodne polecenie Artisan `optimize`, które będzie cachować wszystkie te pliki. To polecenie powinno być zazwyczaj wywoływane jako część procesu wdrażania aplikacji:

```shell
php artisan optimize
```

Metoda `optimize:clear` może być użyta do usunięcia wszystkich plików cache wygenerowanych przez polecenie `optimize`, a także wszystkich kluczy w domyślnym sterowniku cache:

```shell
php artisan optimize:clear
```

W następującej dokumentacji omówimy każde z szczegółowych poleceń optymalizacji, które są wykonywane przez polecenie `optimize`.

<a name="optimizing-configuration-loading"></a>
### Cachowanie konfiguracji

Podczas wdrażania aplikacji na produkcję, powinieneś upewnić się, że uruchamiasz polecenie Artisan `config:cache` podczas procesu wdrażania:

```shell
php artisan config:cache
```

To polecenie połączy wszystkie pliki konfiguracyjne Laravel w jeden cachowany plik, co znacznie zmniejsza liczbę odwołań, które framework musi wykonać do systemu plików podczas ładowania wartości konfiguracyjnych.

> [!WARNING]
> Jeśli wykonujesz polecenie `config:cache` podczas procesu wdrażania, powinieneś upewnić się, że wywołujesz funkcję `env` tylko z poziomu plików konfiguracyjnych. Po cachowaniu konfiguracji plik `.env` nie będzie ładowany, a wszystkie wywołania funkcji `env` dla zmiennych `.env` zwrócą `null`.

<a name="caching-events"></a>
### Cachowanie zdarzeń

Powinieneś cachować automatycznie wykryte mapowania zdarzeń do nasłuchiwaczy w swojej aplikacji podczas procesu wdrażania. Można to osiągnąć, wywołując polecenie Artisan `event:cache` podczas wdrażania:

```shell
php artisan event:cache
```

<a name="optimizing-route-loading"></a>
### Cachowanie tras

Jeśli budujesz dużą aplikację z wieloma trasami, powinieneś upewnić się, że uruchamiasz polecenie Artisan `route:cache` podczas procesu wdrażania:

```shell
php artisan route:cache
```

To polecenie redukuje wszystkie rejestracje tras do pojedynczego wywołania metody w cachowanym pliku, poprawiając wydajność rejestracji tras przy rejestrowaniu setek tras.

<a name="optimizing-view-loading"></a>
### Cachowanie widoków

Podczas wdrażania aplikacji na produkcję, powinieneś upewnić się, że uruchamiasz polecenie Artisan `view:cache` podczas procesu wdrażania:

```shell
php artisan view:cache
```

To polecenie prekompiluje wszystkie Twoje widoki Blade, dzięki czemu nie są one kompilowane na żądanie, poprawiając wydajność każdego żądania, które zwraca widok.

<a name="reloading-services"></a>
## Przeładowywanie usług

> [!NOTE]
> Podczas wdrażania do [Laravel Cloud](https://cloud.laravel.com) nie jest konieczne używanie polecenia `reload`, ponieważ płynne przeładowanie wszystkich usług jest obsługiwane automatycznie.

Po wdrożeniu nowej wersji aplikacji, wszystkie długo działające usługi, takie jak workery kolejek, Laravel Reverb lub Laravel Octane, powinny być przeładowane / zrestartowane, aby używać nowego kodu. Laravel zapewnia pojedyncze polecenie Artisan `reload`, które zakończy te usługi:

```shell
php artisan reload
```

Jeśli nie używasz [Laravel Cloud](https://cloud.laravel.com), powinieneś ręcznie skonfigurować monitor procesów, który może wykryć, kiedy Twoje procesy do przeładowania zakończą się i automatycznie je zrestartować.

<a name="debug-mode"></a>
## Tryb debugowania

Opcja debugowania w pliku konfiguracyjnym `config/app.php` określa, ile informacji o błędzie jest faktycznie wyświetlanych użytkownikowi. Domyślnie ta opcja jest ustawiona tak, aby respektować wartość zmiennej środowiskowej `APP_DEBUG`, która jest przechowywana w pliku `.env` Twojej aplikacji.

> [!WARNING]
> **W środowisku produkcyjnym ta wartość powinna zawsze wynosić `false`. Jeśli zmienna `APP_DEBUG` jest ustawiona na `true` w produkcji, ryzykujesz ujawnienie wrażliwych wartości konfiguracyjnych użytkownikom końcowym Twojej aplikacji.**

<a name="the-health-route"></a>
## Trasa Health

Laravel zawiera wbudowaną trasę sprawdzania stanu zdrowia, która może być używana do monitorowania statusu Twojej aplikacji. W produkcji ta trasa może być używana do raportowania statusu aplikacji do monitora czasu działania, load balancera lub systemu orkiestracji, takiego jak Kubernetes.

Domyślnie trasa sprawdzania stanu zdrowia jest obsługiwana pod `/up` i zwróci odpowiedź HTTP 200, jeśli aplikacja została uruchomiona bez wyjątków. W przeciwnym razie zostanie zwrócona odpowiedź HTTP 500. Możesz skonfigurować URI dla tej trasy w pliku `bootstrap/app` Twojej aplikacji:

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up', // [tl! remove]
    health: '/status', // [tl! add]
)
```

Kiedy żądania HTTP są kierowane do tej trasy, Laravel wyśle również zdarzenie `Illuminate\Foundation\Events\DiagnosingHealth`, umożliwiając wykonanie dodatkowych kontroli stanu zdrowia odpowiednich dla Twojej aplikacji. W ramach [nasłuchiwacza](/docs/{{version}}/events) dla tego zdarzenia możesz sprawdzić status bazy danych lub cache aplikacji. Jeśli wykryjesz problem z aplikacją, możesz po prostu rzucić wyjątek z nasłuchiwacza.

<a name="deploying-with-cloud-or-forge"></a>
## Wdrażanie z Laravel Cloud lub Forge

<a name="laravel-cloud"></a>
#### Laravel Cloud

Jeśli chciałbyś w pełni zarządzaną, automatycznie skalującą się platformę wdrożeniową dostrojoną do Laravel, sprawdź [Laravel Cloud](https://cloud.laravel.com). Laravel Cloud to solidna platforma wdrożeniowa dla Laravel, oferująca zarządzane obliczenia, bazy danych, cache i pamięć obiektową.

Uruchom swoją aplikację Laravel w Cloud i zakochaj się w skalowalnej prostocie. Laravel Cloud jest precyzyjnie dostrojony przez twórców Laravel do bezproblemowej pracy z frameworkiem, dzięki czemu możesz dalej pisać swoje aplikacje Laravel dokładnie tak, jak jesteś przyzwyczajony.

<a name="laravel-forge"></a>
#### Laravel Forge

Jeśli wolisz zarządzać własnymi serwerami, ale nie czujesz się komfortowo z konfiguracją wszystkich różnych usług potrzebnych do uruchomienia solidnej aplikacji Laravel, [Laravel Forge](https://forge.laravel.com) to platforma zarządzania serwerami VPS dla aplikacji Laravel.

Laravel Forge może tworzyć serwery u różnych dostawców infrastruktury, takich jak DigitalOcean, Linode, AWS i innych. Ponadto Forge instaluje i zarządza wszystkimi narzędziami potrzebnymi do budowy solidnych aplikacji Laravel, takimi jak Nginx, MySQL, Redis, Memcached, Beanstalk i innymi.
