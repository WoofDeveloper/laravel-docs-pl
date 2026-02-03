# Laravel Valet

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
    - [Aktualizacja Valeta](#upgrading-valet)
- [Obsługa Stron](#serving-sites)
    - [Komenda "Park"](#the-park-command)
    - [Komenda "Link"](#the-link-command)
    - [Zabezpieczanie Stron za pomocą TLS](#securing-sites)
    - [Obsługa Domyślnej Strony](#serving-a-default-site)
    - [Wersje PHP dla Poszczególnych Stron](#per-site-php-versions)
- [Udostępnianie Stron](#sharing-sites)
    - [Udostępnianie Stron w Sieci Lokalnej](#sharing-sites-on-your-local-network)
- [Zmienne Środowiskowe Specyficzne dla Strony](#site-specific-environment-variables)
- [Proxy Serwisów](#proxying-services)
- [Niestandardowe Sterowniki Valeta](#custom-valet-drivers)
    - [Sterowniki Lokalne](#local-drivers)
- [Inne Komendy Valeta](#other-valet-commands)
- [Katalogi i Pliki Valeta](#valet-directories-and-files)
    - [Dostęp do Dysku](#disk-access)

<a name="introduction"></a>
## Wprowadzenie

> [!NOTE]
> Szukasz jeszcze łatwiejszego sposobu na rozwijanie aplikacji Laravel na macOS lub Windows? Sprawdź [Laravel Herd](https://herd.laravel.com). Herd zawiera wszystko, czego potrzebujesz, aby zacząć rozwój Laravel, w tym Valet, PHP i Composer.

[Laravel Valet](https://github.com/laravel/valet) to środowisko deweloperskie dla minimalistów macOS. Laravel Valet konfiguruje twojego Maca tak, aby zawsze uruchamiał [Nginx](https://www.nginx.com/) w tle, gdy twój komputer się uruchamia. Następnie, używając [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq), Valet przekierowuje wszystkie żądania w domenie `*.test`, aby wskazywały na strony zainstalowane na twoim lokalnym komputerze.

Innymi słowy, Valet to błyskawiczne środowisko deweloperskie Laravel, które używa około 7 MB RAM-u. Valet nie jest pełnym zastępstwem dla [Saila](/docs/{{version}}/sail) lub [Homesteada](/docs/{{version}}/homestead), ale stanowi świetną alternatywę, jeśli chcesz elastycznych podstaw, wolisz ekstremalną szybkość lub pracujesz na komputerze z ograniczoną ilością RAM-u.

Gotowe wsparcie Valeta obejmuje między innymi:

<style>
    #valet-support > ul {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        line-height: 1.9;
    }
</style>

<div id="valet-support" markdown="1">

- [Laravel](https://laravel.com)
- [Bedrock](https://roots.io/bedrock/)
- [CakePHP 3](https://cakephp.org)
- [ConcreteCMS](https://www.concretecms.com/)
- [Contao](https://contao.org/en/)
- [Craft](https://craftcms.com)
- [Drupal](https://www.drupal.org/)
- [ExpressionEngine](https://www.expressionengine.com/)
- [Jigsaw](https://jigsaw.tighten.co)
- [Joomla](https://www.joomla.org/)
- [Katana](https://github.com/themsaid/katana)
- [Kirby](https://getkirby.com/)
- [Magento](https://magento.com/)
- [OctoberCMS](https://octobercms.com/)
- [Sculpin](https://sculpin.io/)
- [Slim](https://www.slimframework.com)
- [Statamic](https://statamic.com)
- Static HTML
- [Symfony](https://symfony.com)
- [WordPress](https://wordpress.org)
- [Zend](https://framework.zend.com)

</div>

Jednak możesz rozszerzyć Valeta własnymi [niestandardowymi sterownikami](#custom-valet-drivers).

<a name="installation"></a>
## Instalacja

> [!WARNING]
> Valet wymaga macOS i [Homebrew](https://brew.sh/). Przed instalacją upewnij się, że żaden inny program, taki jak Apache lub Nginx, nie wiąże się z portem 80 twojego lokalnego komputera.

Aby zacząć, musisz najpierw upewnić się, że Homebrew jest aktualny, używając komendy `update`:

```shell
brew update
```

Następnie powinieneś użyć Homebrew do zainstalowania PHP:

```shell
brew install php
```

Po zainstalowaniu PHP jesteś gotowy do zainstalowania [menedżera pakietów Composer](https://getcomposer.org). Ponadto powinieneś upewnić się, że katalog `$HOME/.composer/vendor/bin` znajduje się w "PATH" twojego systemu. Po zainstalowaniu Composera możesz zainstalować Laravel Valet jako globalny pakiet Composer:

```shell
composer global require laravel/valet
```

Na koniec możesz wykonać komendę `install` Valeta. To skonfiguruje i zainstaluje Valeta oraz DnsMasq. Ponadto demony, od których zależy Valet, zostaną skonfigurowane do uruchamiania przy starcie systemu:

```shell
valet install
```

Po zainstalowaniu Valeta spróbuj zrobić ping dowolnej domeny `*.test` w terminalu za pomocą komendy takiej jak `ping foobar.test`. Jeśli Valet jest zainstalowany poprawnie, powinieneś zobaczyć tę domenę odpowiadającą na `127.0.0.1`.

Valet będzie automatycznie uruchamiać wymagane usługi za każdym razem, gdy twój komputer się uruchomi.

<a name="php-versions"></a>
#### Wersje PHP

> [!NOTE]
> Zamiast modyfikować globalną wersję PHP, możesz polecić Valetowi używanie wersji PHP dla poszczególnych stron za pomocą komendy `isolate` [command](#per-site-php-versions).

Valet pozwala przełączać wersje PHP używając komendy `valet use php@version`. Valet zainstaluje określoną wersję PHP za pośrednictwem Homebrew, jeśli nie jest już zainstalowana:

```shell
valet use php@8.2

valet use php
```

Możesz również utworzyć plik `.valetrc` w katalogu głównym swojego projektu. Plik `.valetrc` powinien zawierać wersję PHP, której strona powinna używać:

```shell
php=php@8.2
```

Po utworzeniu tego pliku możesz po prostu wykonać komendę `valet use`, a komenda określi preferowany wersję PHP strony, odczytując plik.

> [!WARNING]
> Valet obsługuje tylko jedną wersję PHP w tym samym czasie, nawet jeśli masz zainstalowanych wiele wersji PHP.

<a name="database"></a>
#### Baza Danych

Jeśli twoja aplikacja potrzebuje bazy danych, sprawdź [DBngin](https://dbngin.com), który zapewnia darmowe, kompleksowe narzędzie do zarządzania bazami danych, w tym MySQL, PostgreSQL i Redis. Po zainstalowaniu DBngin możesz połączyć się ze swoją bazą danych pod adresem `127.0.0.1` używając nazwy użytkownika `root` i pustego ciągu znaków jako hasła.

<a name="resetting-your-installation"></a>
#### Resetowanie Instalacji

Jeśli masz problemy z prawidłowym działaniem instalacji Valeta, wykonanie komendy `composer global require laravel/valet`, a następnie `valet install` zresetuje instalację i może rozwiązać różne problemy. W rzadkich przypadkach może być konieczne "twarde resetowanie" Valeta poprzez wykonanie `valet uninstall --force`, a następnie `valet install`.

<a name="upgrading-valet"></a>
### Aktualizacja Valeta

Możesz zaktualizować instalację Valeta wykonując komendę `composer global require laravel/valet` w terminalu. Po aktualizacji dobrze jest uruchomić komendę `valet install`, aby Valet mógł wykonać dodatkowe aktualizacje plików konfiguracyjnych, jeśli to konieczne.

<a name="upgrading-to-valet-4"></a>
#### Aktualizacja do Valeta 4

Jeśli aktualizujesz z Valeta 3 do Valeta 4, wykonaj następujące kroki, aby prawidłowo zaktualizować instalację Valeta:

<div class="content-list" markdown="1">

- Jeśli dodałeś pliki `.valetphprc` w celu dostosowania wersji PHP strony, zmień nazwę każdego pliku `.valetphprc` na `.valetrc`. Następnie dodaj `php=` przed istniejącą zawartością pliku `.valetrc`.
- Zaktualizuj wszystkie niestandardowe sterowniki, aby odpowiadały przestrzeni nazw, rozszerzeniu, podpowiedziom typów i zwrotom typów nowego systemu sterowników. Możesz skonsultować się z [SampleValetDriver](https://github.com/laravel/valet/blob/d7787c025e60abc24a5195dc7d4c5c6f2d984339/cli/stubs/SampleValetDriver.php) Valeta jako przykład.
- Jeśli używasz PHP 7.1 - 7.4 do obsługi swoich stron, upewnij się, że nadal używasz Homebrew do zainstalowania wersji PHP 8.0 lub wyższej, gdyż Valet użyje tej wersji, nawet jeśli nie jest to twoja główna połączona wersja, do uruchamiania niektórych skryptów.

</div>

<a name="serving-sites"></a>
## Obsługa Stron

Po zainstalowaniu Valeta jesteś gotowy do rozpoczęcia obsługi aplikacji Laravel. Valet udostępnia dwie komendy, które pomogą ci obsługiwać aplikacje: `park` i `link`.

<a name="the-park-command"></a>
### Komenda `park`

Komenda `park` rejestruje katalog na twoim komputerze, który zawiera twoje aplikacje. Po "zaparkowaniu" katalogu w Valecie wszystkie katalogi w tym katalogu będą dostępne w przeglądarce internetowej pod adresem `http://<nazwa-katalogu>.test`:

```shell
cd ~/Sites

valet park
```

To wszystko, czego potrzeba. Teraz każda aplikacja utworzona w "zaparkowanym" katalogu będzie automatycznie obsługiwana przy użyciu konwencji `http://<nazwa-katalogu>.test`. Więc jeśli twój zaparkowany katalog zawiera katalog o nazwie "laravel", aplikacja w tym katalogu będzie dostępna pod adresem `http://laravel.test`. Ponadto Valet automatycznie pozwala ci uzyskiwać dostęp do strony używając wieloznacznych poddomen (`http://foo.laravel.test`).

<a name="the-link-command"></a>
### Komenda `link`

Komenda `link` może być również użyta do obsługi aplikacji Laravel. Ta komenda jest przydatna, jeśli chcesz obsłużyć pojedynczą stronę w katalogu, a nie cały katalog:

```shell
cd ~/Sites/laravel

valet link
```

Po połączeniu aplikacji z Valetem za pomocą komendy `link` możesz uzyskiwać dostęp do aplikacji używając nazwy jej katalogu. Więc strona, która została połączona w powyższym przykładzie, może być dostępna pod adresem `http://laravel.test`. Ponadto Valet automatycznie pozwala ci uzyskiwać dostęp do strony używając wieloznacznych poddomen (`http://foo.laravel.test`).

Jeśli chcesz obsłużyć aplikację pod inną nazwą hosta, możesz przekazać nazwę hosta do komendy `link`. Na przykład możesz uruchomić następującą komendę, aby udostępnić aplikację pod adresem `http://application.test`:

```shell
cd ~/Sites/laravel

valet link application
```

Oczywiście możesz również obsługiwać aplikacje na poddomenach używając komendy `link`:

```shell
valet link api.application
```

Możesz wykonać komendę `links`, aby wyświetlić listę wszystkich połączonych katalogów:

```shell
valet links
```

Komenda `unlink` może być użyta do zniszczenia dowiązania symbolicznego strony:

```shell
cd ~/Sites/laravel

valet unlink
```

<a name="securing-sites"></a>
### Zabezpieczanie Stron za pomocą TLS

Domyślnie Valet obsługuje strony przez HTTP. Jednak jeśli chcesz obsługiwać stronę przez szyfrowane TLS używając HTTP/2, możesz użyć komendy `secure`. Na przykład, jeśli twoja strona jest obsługiwana przez Valeta w domenie `laravel.test`, powinieneś uruchomić następującą komendę, aby ją zabezpieczyć:

```shell
valet secure laravel
```

Aby "odzabezpieczyć" stronę i powrócić do obsługi jej ruchu przez zwykły HTTP, użyj komendy `unsecure`. Podobnie jak komenda `secure`, ta komenda akceptuje nazwę hosta, który chcesz odzabezpieczyć:

```shell
valet unsecure laravel
```

<a name="serving-a-default-site"></a>
### Obsługa Domyślnej Strony

Czasami możesz chcieć skonfigurować Valeta, aby obsługiwał "domyślną" stronę zamiast `404` podczas odwiedzania nieznanej domeny `test`. Aby to osiągnąć, możesz dodać opcję `default` do pliku konfiguracyjnego `~/.config/valet/config.json`, zawierającą ścieżkę do strony, która powinna służyć jako twoja domyślna strona:

    "default": "/Users/Sally/Sites/example-site",

<a name="per-site-php-versions"></a>
### Wersje PHP dla Poszczególnych Stron

Domyślnie Valet używa twojej globalnej instalacji PHP do obsługi stron. Jednak jeśli musisz obsługiwać wiele wersji PHP na różnych stronach, możesz użyć komendy `isolate`, aby określić, której wersji PHP powinna używać określona strona. Komenda `isolate` konfiguruje Valeta, aby używał określonej wersji PHP dla strony znajdującej się w twoim bieżącym katalogu roboczym:

```shell
cd ~/Sites/example-site

valet isolate php@8.0
```

Jeśli nazwa twojej strony nie odpowiada nazwie katalogu, który ją zawiera, możesz określić nazwę strony używając opcji `--site`:

```shell
valet isolate php@8.0 --site="site-name"
```

Dla wygody możesz używać komend `valet php`, `composer` i `which-php` do przekierowywania wywołań do odpowiedniego CLI PHP lub narzędzia na podstawie skonfigurowanej wersji PHP strony:

```shell
valet php
valet composer
valet which-php
```

Możesz wykonać komendę `isolated`, aby wyświetlić listę wszystkich odizolowanych stron i ich wersji PHP:

```shell
valet isolated
```

Aby przywrócić stronę do globalnie zainstalowanej wersji PHP Valeta, możesz wywołać komendę `unisolate` z katalogu głównego strony:

```shell
valet unisolate
```

<a name="sharing-sites"></a>
## Udostępnianie Stron

Valet zawiera komendę do udostępniania lokalnych stron światu, zapewniając łatwy sposób testowania strony na urządzeniach mobilnych lub udostępniania jej członkom zespołu i klientom.

Gotowo, Valet obsługuje udostępnianie stron przez ngrok lub Expose. Przed udostępnieniem strony powinieneś zaktualizować konfigurację Valeta używając komendy `share-tool`, określając `ngrok`, `expose` lub `cloudflared`:

```shell
valet share-tool ngrok
```

Jeśli wybierzesz narzędzie i nie masz go zainstalowanego przez Homebrew (dla ngrok i cloudflared) lub Composer (dla Expose), Valet automatycznie poprosi cię o jego zainstalowanie. Oczywiście oba narzędzia wymagają uwierzytelnienia twojego konta ngrok lub Expose przed rozpoczęciem udostępniania stron.

Aby udostępnić stronę, przejdź do katalogu strony w terminalu i uruchom komendę `share` Valeta. Publicznie dostępny URL zostanie umieszczony w schowku i jest gotowy do wklejenia bezpośrednio do przeglądarki lub udostępnienia zespołowi:

```shell
cd ~/Sites/laravel

valet share
```

Aby zatrzymać udostępnianie strony, możesz nacisnąć `Control + C`.

> [!WARNING]
> Jeśli używasz niestandardowego serwera DNS (takiego jak `1.1.1.1`), udostępnianie ngrok może nie działać poprawnie. Jeśli tak jest na twoim komputerze, otwórz ustawienia systemowe Maca, przejdź do ustawień Sieci, otwórz ustawienia Zaawansowane, następnie przejdź do zakładki DNS i dodaj `127.0.0.1` jako pierwszy serwer DNS.

<a name="sharing-sites-via-ngrok"></a>
#### Udostępnianie Stron przez Ngrok

Udostępnianie strony przy użyciu ngrok wymaga [utworzenia konta ngrok](https://dashboard.ngrok.com/signup) i [skonfigurowania tokena uwierzytelniającego](https://dashboard.ngrok.com/get-started/your-authtoken). Po uzyskaniu tokena uwierzytelniającego możesz zaktualizować konfigurację Valeta o ten token:

```shell
valet set-ngrok-token YOUR_TOKEN_HERE
```

> [!NOTE]
> Możesz przekazać dodatkowe parametry ngrok do komendy share, takie jak `valet share --region=eu`. Więcej informacji znajdziesz w [dokumentacji ngrok](https://ngrok.com/docs).

<a name="sharing-sites-via-expose"></a>
#### Udostępnianie Stron przez Expose

Udostępnianie strony przy użyciu Expose wymaga [utworzenia konta Expose](https://expose.dev/register) i [uwierzytelnienia się w Expose za pomocą tokena uwierzytelniającego](https://expose.dev/docs/getting-started/getting-your-token).

Możesz sprawdzić [dokumentację Expose](https://expose.dev/docs), aby uzyskać informacje o dodatkowych parametrach wiersza poleceń, które obsługuje.

<a name="sharing-sites-on-your-local-network"></a>
### Udostępnianie Stron w Sieci Lokalnej

Valet domyślnie ogranicza przychodzący ruch do wewnętrznego interfejsu `127.0.0.1`, aby twój komputer deweloperski nie był narażony na zagrożenia bezpieczeństwa z Internetu.

Jeśli chcesz pozwolić innym urządzeniom w twojej sieci lokalnej na dostęp do stron Valeta na twoim komputerze poprzez adres IP twojego komputera (np. `192.168.1.10/application.test`), będziesz musiał ręcznie edytować odpowiedni plik konfiguracyjny Nginx dla tej strony, aby usunąć ograniczenie dyrektywy `listen`. Powinieneś usunąć prefiks `127.0.0.1:` w dyrektywie `listen` dla portów 80 i 443.

Jeśli nie uruchomiłeś `valet secure` w projekcie, możesz otworzyć dostęp sieciowy dla wszystkich stron non-HTTPS, edytując plik `/usr/local/etc/nginx/valet/valet.conf`. Jednak jeśli obsługujesz stronę projektu przez HTTPS (uruchomiłeś `valet secure` dla strony), powinieneś edytować plik `~/.config/valet/Nginx/app-name.test`.

Po zaktualizowaniu konfiguracji Nginx uruchom komendę `valet restart`, aby zastosować zmiany w konfiguracji.

<a name="site-specific-environment-variables"></a>
## Zmienne Środowiskowe Specyficzne dla Strony

Niektóre aplikacje używające innych frameworków mogą zależeć od zmiennych środowiskowych serwera, ale nie zapewniają sposobu na skonfigurowanie tych zmiennych w projekcie. Valet pozwala skonfigurować zmienne środowiskowe specyficzne dla strony, dodając plik `.valet-env.php` w katalogu głównym projektu. Ten plik powinien zwrócić tablicę par strona / zmienna środowiskowa, które zostaną dodane do globalnej tablicy `$_SERVER` dla każdej strony określonej w tablicy:

```php
<?php

return [
    // Ustaw $_SERVER['key'] na "value" dla strony laravel.test...
    'laravel' => [
        'key' => 'value',
    ],

    // Ustaw $_SERVER['key'] na "value" dla wszystkich stron...
    '*' => [
        'key' => 'value',
    ],
];
```

<a name="proxying-services"></a>
## Proxy Serwisów

Czasami możesz chcieć przekierować domenę Valeta do innej usługi na swoim lokalnym komputerze. Na przykład, możesz czasami uruchamiać Valeta podczas uruchamiania oddzielnej strony w Dockerze; jednak Valet i Docker nie mogą jednocześnie wiązać się z portem 80.

Aby to rozwiązać, możesz użyć komendy `proxy` do wygenerowania proxy. Na przykład możesz przekierować cały ruch z `http://elasticsearch.test` do `http://127.0.0.1:9200`:

```shell
# Proxy przez HTTP...
valet proxy elasticsearch http://127.0.0.1:9200

# Proxy przez TLS + HTTP/2...
valet proxy elasticsearch http://127.0.0.1:9200 --secure
```

Możesz usunąć proxy używając komendy `unproxy`:

```shell
valet unproxy elasticsearch
```

Możesz użyć komendy `proxies`, aby wyświetlić wszystkie konfiguracje stron, które są przekierowywane:

```shell
valet proxies
```

<a name="custom-valet-drivers"></a>
## Niestandardowe Sterowniki Valeta

Możesz napisać własny "sterownik" Valeta do obsługi aplikacji PHP działających na frameworku lub CMS, który nie jest natywnie obsługiwany przez Valeta. Podczas instalacji Valeta tworzony jest katalog `~/.config/valet/Drivers`, który zawiera plik `SampleValetDriver.php`. Ten plik zawiera przykładową implementację sterownika, aby zademonstrować, jak napisać niestandardowy sterownik. Pisanie sterownika wymaga zaimplementowania tylko trzech metod: `serves`, `isStaticFile` i `frontControllerPath`.

Wszystkie trzy metody otrzymują wartości `$sitePath`, `$siteName` i `$uri` jako argumenty. `$sitePath` to w pełni kwalifikowana ścieżka do strony obsługiwanej na twoim komputerze, na przykład `/Users/Lisa/Sites/my-project`. `$siteName` to część "host" / "nazwa strony" domeny (`my-project`). `$uri` to przychodzący URI żądania (`/foo/bar`).

Po ukończeniu niestandardowego sterownika Valeta umieść go w katalogu `~/.config/valet/Drivers` używając konwencji nazewnictwa `FrameworkValetDriver.php`. Na przykład, jeśli piszesz niestandardowy sterownik Valeta dla WordPressa, nazwa pliku powinna być `WordPressValetDriver.php`.

Przyjrzyjmy się przykładowej implementacji każdej metody, którą powinien implementować twój niestandardowy sterownik Valeta.

<a name="the-serves-method"></a>
#### Metoda `serves`

Metoda `serves` powinna zwrócić `true`, jeśli twój sterownik powinien obsłużyć przychodzące żądanie. W przeciwnym razie metoda powinna zwrócić `false`. Więc w tej metodzie powinieneś spróbować określić, czy podane `$sitePath` zawiera projekt typu, który próbujesz obsłużyć.

Na przykład, wyobraźmy sobie, że piszemy `WordPressValetDriver`. Nasza metoda `serves` może wyglądać mniej więcej tak:

```php
/**
 * Określ, czy sterownik obsługuje żądanie.
 */
public function serves(string $sitePath, string $siteName, string $uri): bool
{
    return is_dir($sitePath.'/wp-admin');
}
```

<a name="the-isstaticfile-method"></a>
#### Metoda `isStaticFile`

Metoda `isStaticFile` powinna określić, czy przychodzące żądanie dotyczy pliku, który jest "statyczny", takiego jak obraz lub arkusz stylów. Jeśli plik jest statyczny, metoda powinna zwrócić w pełni kwalifikowaną ścieżkę do pliku statycznego na dysku. Jeśli przychodzące żądanie nie dotyczy pliku statycznego, metoda powinna zwrócić `false`:

```php
/**
 * Określ, czy przychodzące żądanie dotyczy pliku statycznego.
 *
 * @return string|false
 */
public function isStaticFile(string $sitePath, string $siteName, string $uri)
{
    if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
        return $staticFilePath;
    }

    return false;
}
```

> [!WARNING]
> Metoda `isStaticFile` zostanie wywołana tylko wtedy, gdy metoda `serves` zwraca `true` dla przychodzącego żądania i URI żądania nie jest `/`.

<a name="the-frontcontrollerpath-method"></a>
#### Metoda `frontControllerPath`

Metoda `frontControllerPath` powinna zwrócić w pełni rozwiązaną ścieżkę do "front controllera" aplikacji, który zazwyczaj jest plikiem "index.php" lub równoważnym:

```php
/**
 * Pobierz w pełni rozwiązaną ścieżkę do front controllera aplikacji.
 */
public function frontControllerPath(string $sitePath, string $siteName, string $uri): string
{
    return $sitePath.'/public/index.php';
}
```

<a name="local-drivers"></a>
### Sterowniki Lokalne

Jeśli chcesz zdefiniować niestandardowy sterownik Valeta dla pojedynczej aplikacji, utwórz plik `LocalValetDriver.php` w katalogu głównym aplikacji. Twój niestandardowy sterownik może rozszerzać bazową klasę `ValetDriver` lub rozszerzać istniejący sterownik specyficzny dla aplikacji, taki jak `LaravelValetDriver`:

```php
use Valet\Drivers\LaravelValetDriver;

class LocalValetDriver extends LaravelValetDriver
{
    /**
     * Określ, czy sterownik obsługuje żądanie.
     */
    public function serves(string $sitePath, string $siteName, string $uri): bool
    {
        return true;
    }

    /**
     * Pobierz w pełni rozwiązaną ścieżkę do front controllera aplikacji.
     */
    public function frontControllerPath(string $sitePath, string $siteName, string $uri): string
    {
        return $sitePath.'/public_html/index.php';
    }
}
```

<a name="other-valet-commands"></a>
## Inne Komendy Valeta

<div class="overflow-auto">

| Komenda | Opis |
| --- | --- |
| `valet list` | Wyświetl listę wszystkich komend Valeta. |
| `valet diagnose` | Wyprowadź diagnostykę w celu wspomagania debugowania Valeta. |
| `valet directory-listing` | Określ zachowanie listowania katalogów. Domyślnie "off", który renderuje stronę 404 dla katalogów. |
| `valet forget` | Uruchom tę komendę z "zaparkowanego" katalogu, aby usunąć go z listy zaparkowanych katalogów. |
| `valet log` | Wyświetl listę logów zapisanych przez usługi Valeta. |
| `valet paths` | Wyświetl wszystkie twoje "zaparkowane" ścieżki. |
| `valet restart` | Uruchom ponownie demony Valeta. |
| `valet start` | Uruchom demony Valeta. |
| `valet stop` | Zatrzymaj demony Valeta. |
| `valet trust` | Dodaj pliki sudoers dla Brew i Valeta, aby umożliwić uruchamianie komend Valeta bez monitowania o hasło. |
| `valet uninstall` | Odinstaluj Valeta: pokaż instrukcje ręcznej dezinstalacji. Przekaz opcję `--force`, aby agresywnie usunąć wszystkie zasoby Valeta. |

</div>

<a name="valet-directories-and-files"></a>
## Katalogi i Pliki Valeta

Następujące informacje o katalogach i plikach mogą być pomocne podczas rozwiązywania problemów z środowiskiem Valeta:

#### `~/.config/valet`

Zawiera całą konfigurację Valeta. Możesz chcieć zachować kopię zapasową tego katalogu.

#### `~/.config/valet/dnsmasq.d/`

Ten katalog zawiera konfigurację DNSMasq.

#### `~/.config/valet/Drivers/`

Ten katalog zawiera sterowniki Valeta. Sterowniki określają, jak obsługiwany jest określony framework / CMS.

#### `~/.config/valet/Nginx/`

Ten katalog zawiera wszystkie konfiguracje stron Nginx Valeta. Te pliki są przebudowywane podczas uruchamiania komend `install` i `secure`.

#### `~/.config/valet/Sites/`

Ten katalog zawiera wszystkie dowiązania symboliczne dla twoich [połączonych projektów](#the-link-command).

#### `~/.config/valet/config.json`

Ten plik jest głównym plikiem konfiguracyjnym Valeta.

#### `~/.config/valet/valet.sock`

Ten plik to gniazdo PHP-FPM używane przez instalację Nginx Valeta. Będzie istnieć tylko wtedy, gdy PHP działa prawidłowo.

#### `~/.config/valet/Log/fpm-php.www.log`

Ten plik to log użytkownika dla błędów PHP.

#### `~/.config/valet/Log/nginx-error.log`

Ten plik to log użytkownika dla błędów Nginx.

#### `/usr/local/var/log/php-fpm.log`

Ten plik to log systemowy dla błędów PHP-FPM.

#### `/usr/local/var/log/nginx`

Ten katalog zawiera logi dostępu i błędów Nginx.

#### `/usr/local/etc/php/X.X/conf.d`

Ten katalog zawiera pliki `*.ini` dla różnych ustawień konfiguracyjnych PHP.

#### `/usr/local/etc/php/X.X/php-fpm.d/valet-fpm.conf`

Ten plik to plik konfiguracyjny puli PHP-FPM.

#### `~/.composer/vendor/laravel/valet/cli/stubs/secure.valet.conf`

Ten plik to domyślna konfiguracja Nginx używana do budowania certyfikatów SSL dla twoich stron.

<a name="disk-access"></a>
### Dostęp do Dysku

Od macOS 10.14, [dostęp do niektórych plików i katalogów jest domyślnie ograniczony](https://manuals.info.apple.com/MANUALS/1000/MA1902/en_US/apple-platform-security-guide.pdf). Te ograniczenia obejmują katalogi Desktop, Documents i Downloads. Ponadto dostęp do wolumenów sieciowych i wolumenów wymiennych jest ograniczony. Dlatego Valet zaleca, aby foldery stron były zlokalizowane poza tymi chronionymi lokalizacjami.

Jednak jeśli chcesz obsługiwać strony z jedną z tych lokalizacji, będziesz musiał nadać Nginx "Pełny dostęp do dysku". W przeciwnym razie możesz napotkać błędy serwera lub inne nieprzewidywalne zachowanie Nginx, zwłaszcza podczas obsługi zasobów statycznych. Zazwyczaj macOS automatycznie poprosi cię o przyznanie Nginx pełnego dostępu do tych lokalizacji. Lub możesz to zrobić ręcznie poprzez `System Preferences` > `Security & Privacy` > `Privacy` i wybranie `Full Disk Access`. Następnie włącz wszystkie wpisy `nginx` w głównym panelu okna.
