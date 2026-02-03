# Laravel Sail

- [Wprowadzenie](#introduction)
- [Instalacja i Konfiguracja](#installation)
    - [Instalacja Sail w Istniejących Aplikacjach](#installing-sail-into-existing-applications)
    - [Przebudowa Obrazów Sail](#rebuilding-sail-images)
    - [Konfiguracja Aliasu Powłoki](#configuring-a-shell-alias)
- [Uruchamianie i Zatrzymywanie Sail](#starting-and-stopping-sail)
- [Wykonywanie Poleceń](#executing-sail-commands)
    - [Wykonywanie Poleceń PHP](#executing-php-commands)
    - [Wykonywanie Poleceń Composer](#executing-composer-commands)
    - [Wykonywanie Poleceń Artisan](#executing-artisan-commands)
    - [Wykonywanie Poleceń Node / NPM](#executing-node-npm-commands)
- [Interakcja z Bazami Danych](#interacting-with-sail-databases)
    - [MySQL](#mysql)
    - [MongoDB](#mongodb)
    - [Redis](#redis)
    - [Valkey](#valkey)
    - [Meilisearch](#meilisearch)
    - [Typesense](#typesense)
- [Przechowywanie Plików](#file-storage)
- [Uruchamianie Testów](#running-tests)
    - [Laravel Dusk](#laravel-dusk)
- [Podgląd E-maili](#previewing-emails)
- [CLI Kontenera](#sail-container-cli)
- [Wersje PHP](#sail-php-versions)
- [Wersje Node](#sail-node-versions)
- [Udostępnianie Witryny](#sharing-your-site)
- [Debugowanie z Xdebug](#debugging-with-xdebug)
  - [Użycie Xdebug CLI](#xdebug-cli-usage)
  - [Użycie Xdebug w Przeglądarce](#xdebug-browser-usage)
- [Dostosowanie](#sail-customization)

<a name="introduction"></a>
## Wprowadzenie

[Laravel Sail](https://github.com/laravel/sail) to lekki interfejs wiersza poleceń do interakcji z domyślnym środowiskiem programistycznym Docker Laravel. Sail zapewnia świetny punkt wyjścia do budowania aplikacji Laravel przy użyciu PHP, MySQL i Redis bez wymagania wcześniejszego doświadczenia z Docker.

W swoje sercu Sail to plik `compose.yaml` i skrypt `sail`, który jest przechowywany w głównym katalogu projektu. Skrypt `sail` zapewnia CLI z wygodnymi metodami do interakcji z kontenerami Docker zdefiniowanymi w pliku `compose.yaml`.

Laravel Sail jest obsługiwany na macOS, Linux i Windows (poprzez [WSL2](https://docs.microsoft.com/en-us/windows/wsl/about)).

<a name="installation"></a>
## Instalacja i Konfiguracja

Laravel Sail jest automatycznie instalowany ze wszystkimi nowymi aplikacjami Laravel, więc możesz zacząć korzystać z niego natychmiast.

<a name="installing-sail-into-existing-applications"></a>
### Instalacja Sail w Istniejących Aplikacjach

Jeśli jesteś zainteresowany użyciem Sail z istniejącą aplikacją Laravel, możesz po prostu zainstalować Sail używając menedżera pakietów Composer. Oczywiście, te kroki zakładają, że Twoje istniejące lokalne środowisko programistyczne pozwala na instalowanie zależności Composer:

```shell
composer require laravel/sail --dev
```

Po zainstalowaniu Sail, możesz uruchomić polecenie Artisan `sail:install`. To polecenie opublikuje plik `compose.yaml` Sail w głównym katalogu Twojej aplikacji i zmodyfikuje plik `.env` niezbędnymi zmiennymi środowiskowymi w celu połączenia z usługami Docker:

```shell
php artisan sail:install
```

W końcu możesz uruchomić Sail. Aby kontynuować naukę korzystania z Sail, przeczytaj pozostałą część tej dokumentacji:

```shell
./vendor/bin/sail up
```

> [!WARNING]
> Jeśli używasz Docker Desktop dla Linuxa, powinieneś użyć domyślnego kontekstu Docker, wykonując następujące polecenie: `docker context use default`. Dodatkowo, jeśli napotkasz błędy uprawnień plików w kontenerach, możesz potrzebować ustawić zmienną środowiskową `SUPERVISOR_PHP_USER` na `root`.

<a name="adding-additional-services"></a>
#### Dodawanie Dodatkowych Usług

Jeśli chcesz dodać dodatkowoą usługę do istniejącej instalacji Sail, możesz uruchomić polecenie Artisan `sail:add`:

```shell
php artisan sail:add
```

<a name="using-devcontainers"></a>
#### Używanie Devcontainerów

Jeśli chcesz programować w [Devcontainerze](https://code.visualstudio.com/docs/remote/containers), możesz przekazać opcję `--devcontainer` do polecenia `sail:install`. Opcja `--devcontainer` poleci poleceniu `sail:install` opublikować domyślny plik `.devcontainer/devcontainer.json` w głównym katalogu Twojej aplikacji:

```shell
php artisan sail:install --devcontainer
```

<a name="rebuilding-sail-images"></a>
### Przebudowa Obrazów Sail

Czasami możesz chcieć całkowicie przebudować swoje obrazy Sail, aby upewnić się, że wszystkie pakiety i oprogramowanie obrazu są aktualne. Możesz to osiągnąć używając polecenia `build`:

```shell
docker compose down -v

sail build --no-cache

sail up
```

<a name="configuring-a-shell-alias"></a>
### Konfiguracja Aliasu Powłoki

Domyślnie polecenia Sail są wywoływane używając skryptu `vendor/bin/sail`, który jest dołączony do wszystkich nowych aplikacji Laravel:

```shell
./vendor/bin/sail up
```

Jednak zamiast wielokrotnie wpisywać `vendor/bin/sail`, aby wykonywać polecenia Sail, możesz skonfigurować alias powłoki, który pozwoli na łatwiejsze wykonywanie poleceń Sail:

```shell
alias sail='sh $([ -f sail ] && echo sail || echo vendor/bin/sail)'
```

Aby upewnić się, że jest to zawsze dostępne, możesz dodać to do pliku konfiguracyjnego powłoki w katalogu domowym, takiego jak `~/.zshrc` lub `~/.bashrc`, a następnie uruchomić ponownie powłokę.

Po skonfigurowaniu aliasu powłoki, możesz wykonywać polecenia Sail, po prostu wpisując `sail`. Pozostałe przykłady w tej dokumentacji zakładają, że skonfigurowałeś ten alias:

```shell
sail up
```

<a name="starting-and-stopping-sail"></a>
## Uruchamianie i Zatrzymywanie Sail

Plik `compose.yaml` Laravel Sail definiuje różnorodne kontenery Docker, które współpracują, aby pomóc Ci budować aplikacje Laravel. Każdy z tych kontenerów to wpis w konfiguracji `services` Twojego pliku `compose.yaml`. Kontener `laravel.test` to główny kontener aplikacji, który będzie obsługiwać Twoją aplikację.

Przed uruchomieniem Sail powinieneś upewnić się, że żadne inne serwery WWW ani bazy danych nie są uruchomione na Twoim lokalnym komputerze. Aby uruchomić wszystkie kontenery Docker zdefiniowane w pliku `compose.yaml` Twojej aplikacji, powinieneś wykonać polecenie `up`:

```shell
sail up
```

Aby uruchomić wszystkie kontenery Docker w tle, możesz uruchomić Sail w trybie "detached":

```shell
sail up -d
```

Po uruchomieniu kontenerów aplikacji możesz uzyskać dostęp do projektu w przeglądarce internetowej pod adresem: http://localhost.

Aby zatrzymać wszystkie kontenery, możesz po prostu nacisnąć Control + C, aby zatrzymać wykonywanie kontenera. Lub, jeśli kontenery działają w tle, możesz użyć polecenia `stop`:

```shell
sail stop
```

<a name="executing-sail-commands"></a>
## Wykonywanie Poleceń

Podczas używania Laravel Sail, Twoja aplikacja jest wykonywana w kontenerze Docker i jest odizolowana od Twojego lokalnego komputera. Jednak Sail zapewnia wygodny sposób uruchamiania różnych poleceń na Twojej aplikacji, takich jak dowolne polecenia PHP, polecenia Artisan, polecenia Composer i polecenia Node / NPM.

**Podczas czytania dokumentacji Laravel często będziesz widzieć odniesienia do poleceń Composer, Artisan i Node / NPM, które nie odwołują się do Sail.** Te przykłady zakładają, że te narzędzia są zainstalowane na Twoim lokalnym komputerze. Jeśli używasz Sail dla swojego lokalnego środowiska programistycznego Laravel, powinieneś wykonywać te polecenia używając Sail:

```shell
# Uruchamianie poleceń Artisan lokalnie...
php artisan queue:work

# Uruchamianie poleceń Artisan w Laravel Sail...
sail artisan queue:work
```

<a name="executing-php-commands"></a>
### Wykonywanie Poleceń PHP

Polecenia PHP mogą być wykonywane za pomocą polecenia `php`. Oczywiście, te polecenia będą wykonywane przy użyciu wersji PHP, która jest skonfigurowana dla Twojej aplikacji. Aby dowiedzieć się więcej o wersjach PHP dostępnych dla Laravel Sail, zapoznaj się z [dokumentacją wersji PHP](#sail-php-versions):

```shell
sail php --version

sail php script.php
```

<a name="executing-composer-commands"></a>
### Wykonywanie Poleceń Composer

Polecenia Composer mogą być wykonywane za pomocą polecenia `composer`. Kontener aplikacji Laravel Sail zawiera instalację Composer:

```shell
sail composer require laravel/sanctum
```

<a name="executing-artisan-commands"></a>
### Wykonywanie Poleceń Artisan

Polecenia Laravel Artisan mogą być wykonywane za pomocą polecenia `artisan`:

```shell
sail artisan queue:work
```

<a name="executing-node-npm-commands"></a>
### Wykonywanie Poleceń Node / NPM

Polecenia Node mogą być wykonywane za pomocą polecenia `node`, podczas gdy polecenia NPM mogą być wykonywane za pomocą polecenia `npm`:

```shell
sail node --version

sail npm run dev
```

Jeśli chcesz, możesz użyć Yarn zamiast NPM:

```shell
sail yarn
```

<a name="interacting-with-sail-databases"></a>
## Interakcja z Bazami Danych

<a name="mysql"></a>
### MySQL

Jak możesz zauważyć, plik `compose.yaml` Twojej aplikacji zawiera wpis dla kontenera MySQL. Ten kontener używa [wolumenu Docker](https://docs.docker.com/storage/volumes/), aby dane przechowywane w bazie danych były utrwalane nawet po zatrzymaniu i ponownym uruchomieniu kontenerów.

Ponadto, przy pierwszym uruchomieniu kontenera MySQL, utworzy on dla Ciebie dwie bazy danych. Pierwsza baza danych jest nazwana używając wartości zmiennej środowiskowej `DB_DATABASE` i jest przeznaczona dla lokalnego programowania. Druga to dedykowana baza danych testowa o nazwie `testing`, która zapewni, że Twoje testy nie będą interferowar ze z danymi programistycznymi.

Po uruchomieniu kontenerów możesz połączyć się z instancją MySQL w swojej aplikacji, ustawiając zmienną środowiskową `DB_HOST` w pliku `.env` aplikacji na `mysql`.

Aby połączyć się z bazą danych MySQL aplikacji z lokalnego komputera, możesz użyć graficznej aplikacji do zarządzania bazą danych, takiej jak [TablePlus](https://tableplus.com). Domyślnie baza danych MySQL jest dostępna na `localhost` na porcie 3306, a dane uwierzytelniające odpowiadają wartościom zmiennych środowiskowych `DB_USERNAME` i `DB_PASSWORD`. Lub możesz połączyć się jako użytkownik `root`, który również używa wartości zmiennej środowiskowej `DB_PASSWORD` jako hasła.

<a name="mongodb"></a>
### MongoDB

Jeśli zdecydowałeś się zainstalować usługę [MongoDB](https://www.mongodb.com/) podczas instalacji Sail, plik `compose.yaml` Twojej aplikacji zawiera wpis dla kontenera [MongoDB Atlas Local](https://www.mongodb.com/docs/atlas/cli/current/atlas-cli-local-cloud/), który zapewnia bazę danych dokumentową MongoDB z funkcjami Atlas, takimi jak [Search Indexes](https://www.mongodb.com/docs/atlas/atlas-search/). Ten kontener używa [wolumenu Docker](https://docs.docker.com/storage/volumes/), aby dane przechowywane w bazie danych były utrwalane nawet po zatrzymaniu i ponownym uruchomieniu kontenerów.

Po uruchomieniu kontenerów możesz połączyć się z instancją MongoDB w swojej aplikacji, ustawiając zmienną środowiskową `MONGODB_URI` w pliku `.env` aplikacji na `mongodb://mongodb:27017`. Uwierzytelnianie jest domyślnie wyłączone, ale możesz ustawić zmienne środowiskowe `MONGODB_USERNAME` i `MONGODB_PASSWORD`, aby włączyć uwierzytelnianie przed uruchomieniem kontenera `mongodb`. Następnie dodaj dane uwierzytelniające do ciągu połączenia:

```ini
MONGODB_USERNAME=user
MONGODB_PASSWORD=laravel
MONGODB_URI=mongodb://${MONGODB_USERNAME}:${MONGODB_PASSWORD}@mongodb:27017
```

Dla bezproblemowej integracji MongoDB z Twoją aplikacją możesz zainstalować [oficjalny pakiet utrzymywany przez MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/).

Aby połączyć się z bazą danych MongoDB aplikacji z lokalnego komputera, możesz użyć graficznego interfejsu, takiego jak [Compass](https://www.mongodb.com/products/tools/compass). Domyślnie baza danych MongoDB jest dostępna na `localhost` na porcie `27017`.

<a name="redis"></a>
### Redis

Plik `compose.yaml` Twojej aplikacji zawiera również wpis dla kontenera [Redis](https://redis.io). Ten kontener używa [wolumenu Docker](https://docs.docker.com/storage/volumes/), aby dane przechowywane w instancji Redis były utrwalane nawet po zatrzymaniu i ponownym uruchomieniu kontenerów. Po uruchomieniu kontenerów możesz połączyć się z instancją Redis w swojej aplikacji, ustawiając zmienną środowiskową `REDIS_HOST` w pliku `.env` aplikacji na `redis`.

Aby połączyć się z bazą danych Redis aplikacji z lokalnego komputera, możesz użyć graficznej aplikacji do zarządzania bazą danych, takiej jak [TablePlus](https://tableplus.com). Domyślnie baza danych Redis jest dostępna na `localhost` na porcie 6379.

<a name="valkey"></a>
### Valkey

Jeśli zdecydowałeś się zainstalować usługę Valkey podczas instalacji Sail, plik `compose.yaml` Twojej aplikacji będzie zawierał wpis dla [Valkey](https://valkey.io/). Ten kontener używa [wolumenu Docker](https://docs.docker.com/storage/volumes/), aby dane przechowywane w instancji Valkey były utrwalane nawet po zatrzymaniu i ponownym uruchomieniu kontenerów. Możesz połączyć się z tym kontenerem w swojej aplikacji, ustawiając zmienną środowiskową `REDIS_HOST` w pliku `.env` aplikacji na `valkey`.

Aby połączyć się z bazą danych Valkey aplikacji z lokalnego komputera, możesz użyć graficznej aplikacji do zarządzania bazą danych, takiej jak [TablePlus](https://tableplus.com). Domyślnie baza danych Valkey jest dostępna na `localhost` na porcie 6379.

<a name="meilisearch"></a>
### Meilisearch

Jeśli zdecydowałeś się zainstalować usługę [Meilisearch](https://www.meilisearch.com) podczas instalacji Sail, plik `compose.yaml` Twojej aplikacji będzie zawierał wpis dla tego potężnego silnika wyszukiwania, który jest zintegrowany z [Laravel Scout](/docs/{{version}}/scout). Po uruchomieniu kontenerów możesz połączyć się z instancją Meilisearch w swojej aplikacji, ustawiając zmienną środowiskową `MEILISEARCH_HOST` na `http://meilisearch:7700`.

Z lokalnego komputera możesz uzyskać dostęp do webowego panelu administracyjnego Meilisearch, przechodząc do `http://localhost:7700` w przeglądarce internetowej.

<a name="typesense"></a>
### Typesense

Jeśli zdecydowałeś się zainstalować usługę [Typesense](https://typesense.org) podczas instalacji Sail, plik `compose.yaml` Twojej aplikacji będzie zawierał wpis dla tego błyskawicznie szybkiego, open-source silnika wyszukiwania, który jest natywnie zintegrowany z [Laravel Scout](/docs/{{version}}/scout#typesense). Po uruchomieniu kontenerów możesz połączyć się z instancją Typesense w swojej aplikacji, ustawiając następujące zmienne środowiskowe:

```ini
TYPESENSE_HOST=typesense
TYPESENSE_PORT=8108
TYPESENSE_PROTOCOL=http
TYPESENSE_API_KEY=xyz
```

Z lokalnego komputera możesz uzyskać dostęp do API Typesense poprzez `http://localhost:8108`.

<a name="file-storage"></a>
## Przechowywanie Plików

Jeśli planujesz używać Amazon S3 do przechowywania plików podczas uruchamiania aplikacji w środowisku produkcyjnym, możesz rozważyć zainstalowanie usługi [RustFS](https://rustfs.com) podczas instalacji Sail. RustFS zapewnia API kompatybilne z S3, którego możesz użyć do lokalnego tworzenia aplikacji przy użyciu sterownika przechowywania plików `s3` Laravel bez tworzenia "testowych" pojemników pamięci w produkcyjnym środowisku S3. Jeśli zdecydujesz się zainstalować RustFS podczas instalacji Sail, sekcja konfiguracyjna RustFS zostanie dodana do pliku `compose.yaml` Twojej aplikacji.

Domyślnie plik konfiguracyjny `filesystems` Twojej aplikacji zawiera już konfigurację dysku dla dysku `s3`. Oprócz używania tego dysku do interakcji z Amazon S3, możesz go użyć do interakcji z dowolną usługą przechowywania plików kompatybilną z S3, taką jak RustFS, po prostu modyfikując powiązane zmienne środowiskowe, które kontrolują jego konfigurację. Na przykład, podczas używania RustFS, konfiguracja zmiennych środowiskowych systemu plików powinna być zdefiniowana następująco:

```ini
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=sail
AWS_SECRET_ACCESS_KEY=password
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=local
AWS_ENDPOINT=http://rustfs:9000
AWS_USE_PATH_STYLE_ENDPOINT=true
```

<a name="running-tests"></a>
## Uruchamianie Testów

Laravel zapewnia niesamowite wsparcie dla testów od razu po instalacji, a możesz użyć polecenia `test` Sail, aby uruchomić [testy funkcjonalne i jednostkowe](/docs/{{version}}/testing) swojej aplikacji. Wszelkie opcje CLI, które są akceptowane przez Pest / PHPUnit, mogą również być przekazane do polecenia `test`:

```shell
sail test

sail test --group orders
```

Polecenie `test` Sail odpowiada uruchomieniu polecenia Artisan `test`:

```shell
sail artisan test
```

Domyślnie Sail utworzy dedykowaną bazę danych `testing`, aby Twoje testy nie interferowar ze z bieżącym stanem bazy danych. W domyślnej instalacji Laravel, Sail skonfiguruje również plik `phpunit.xml`, aby używać tej bazy danych podczas wykonywania testów:

```xml
<env name="DB_DATABASE" value="testing"/>
```

<a name="laravel-dusk"></a>
### Laravel Dusk

[Laravel Dusk](/docs/{{version}}/dusk) zapewnia ekspresywne, łatwe w użyciu API do automatyzacji i testowania przeglądarki. Dzięki Sail możesz uruchamiać te testy bez instalowania Selenium lub innych narzędzi na lokalnym komputerze. Aby rozpocząć, odkomentuj usługę Selenium w pliku `compose.yaml` Twojej aplikacji:

```yaml
selenium:
    image: 'selenium/standalone-chrome'
    extra_hosts:
      - 'host.docker.internal:host-gateway'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

Następnie upewnij się, że usługa `laravel.test` w pliku `compose.yaml` Twojej aplikacji ma wpis `depends_on` dla `selenium`:

```yaml
depends_on:
    - mysql
    - redis
    - selenium
```

W końcu możesz uruchomić swoje testy Dusk, uruchamiając Sail i wykonując polecenie `dusk`:

```shell
sail dusk
```

<a name="selenium-on-apple-silicon"></a>
#### Selenium na Apple Silicon

Jeśli Twój lokalny komputer zawiera chip Apple Silicon, Twoja usługa `selenium` musi używać obrazu `selenium/standalone-chromium`:

```yaml
selenium:
    image: 'selenium/standalone-chromium'
    extra_hosts:
        - 'host.docker.internal:host-gateway'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

<a name="previewing-emails"></a>
## Podgląd E-maili

Domyślny plik `compose.yaml` Laravel Sail zawiera wpis usługi dla [Mailpit](https://github.com/axllent/mailpit). Mailpit przechwytuje e-maile wysyłane przez Twoją aplikację podczas lokalnego tworzenia i zapewnia wygodny interfejs webowy, dzięki któremu możesz przeglądać wiadomości e-mail w przeglądarce. Podczas używania Sail, domyślny host Mailpit to `mailpit` i jest dostępny przez port 1025:

```ini
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_ENCRYPTION=null
```

Gdy Sail jest uruchomiony, możesz uzyskać dostęp do interfejsu webowego Mailpit pod adresem: http://localhost:8025

<a name="sail-container-cli"></a>
## CLI Kontenera

Czasami możesz chcieć uruchomić sesję Bash w kontenerze swojej aplikacji. Możesz użyć polecenia `shell`, aby połączyć się z kontenerem aplikacji, co pozwala na sprawdzenie jego plików i zainstalowanych usług oraz wykonywanie dowolnych poleceń powłoki w kontenerze:

```shell
sail shell

sail root-shell
```

Aby uruchomić nową sesję [Laravel Tinker](https://github.com/laravel/tinker), możesz wykonać polecenie `tinker`:

```shell
sail tinker
```

<a name="sail-php-versions"></a>
## Wersje PHP

Sail obecnie obsługuje obsługę Twojej aplikacji za pomocą PHP 8.5, 8.4, 8.3, 8.2, 8.1 lub PHP 8.0. Domyślną wersją PHP używaną przez Sail jest obecnie PHP 8.4. Aby zmienić wersję PHP używaną do obsługi Twojej aplikacji, powinieś zaktualizować definicję `build` kontenera `laravel.test` w pliku `compose.yaml` Twojej aplikacji:

```yaml
# PHP 8.5
context: ./vendor/laravel/sail/runtimes/8.5

# PHP 8.4
context: ./vendor/laravel/sail/runtimes/8.4

# PHP 8.3
context: ./vendor/laravel/sail/runtimes/8.3

# PHP 8.2
context: ./vendor/laravel/sail/runtimes/8.2

# PHP 8.1
context: ./vendor/laravel/sail/runtimes/8.1

# PHP 8.0
context: ./vendor/laravel/sail/runtimes/8.0
```

Ponadto możesz chcieć zaktualizować nazwę `image`, aby odzwierciedlała wersję PHP używaną przez Twoją aplikację. Ta opcja jest również zdefiniowana w pliku `compose.yaml` Twojej aplikacji:

```yaml
image: sail-8.2/app
```

Po zaktualizowaniu pliku `compose.yaml` Twojej aplikacji powinieś przebudować obrazy kontenerów:

```shell
sail build --no-cache

sail up
```

<a name="sail-node-versions"></a>
## Wersje Node

Sail instaluje Node 22 domyślnie. Aby zmienić wersję Node, która jest instalowana podczas budowania obrazów, możesz zaktualizować definicję `build.args` usługi `laravel.test` w pliku `compose.yaml` Twojej aplikacji:

```yaml
build:
    args:
        WWWGROUP: '${WWWGROUP}'
        NODE_VERSION: '18'
```

Po zaktualizowaniu pliku `compose.yaml` Twojej aplikacji powinieś przebudować obrazy kontenerów:

```shell
sail build --no-cache

sail up
```

<a name="sharing-your-site"></a>
## Udostępnianie Witryny

Czasami możesz potrzebować udostępnić swoją witrynę publicznie, aby podejrzeć ją dla współpracownika lub przetestować integracje webhook z Twoją aplikacją. Aby udostępnić swoją witrynę, możesz użyć polecenia `share`. Po wykonaniu tego polecenia otrzymasz losowy URL `laravel-sail.site`, którego możesz użyć do uzyskania dostępu do Twojej aplikacji:

```shell
sail share
```

Podczas udostępniania witryny za pomocą polecenia `share` powinieś skonfigurować zaufane proxy swojej aplikacji, używając metody middleware `trustProxies` w pliku `bootstrap/app.php` Twojej aplikacji. W przeciwnym razie pomocnicy generowania URL, takie jak `url` i `route`, nie będą mogli określić poprawnego hosta HTTP, który powinien być używany podczas generowania URL:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustProxies(at: '*');
})
```

Jeśli chcesz wybrać subdomenę dla swojej udostępnianej witryny, możesz podać opcję `subdomain` podczas wykonywania polecenia `share`:

```shell
sail share --subdomain=my-sail-site
```

> [!NOTE]
> Polecenie `share` jest obsługiwane przez [Expose](https://github.com/beyondcode/expose), usługę tunelowania open source od [BeyondCode](https://beyondco.de).

<a name="debugging-with-xdebug"></a>
## Debugowanie z Xdebug

Konfiguracja Docker Laravel Sail zawiera wsparcie dla [Xdebug](https://xdebug.org/), popularnego i potężnego debugera dla PHP. Aby włączyć Xdebug, upewnij się, że [opublikowałeś konfigurację Sail](#sail-customization). Następnie dodaj następujące zmienne do pliku `.env` Twojej aplikacji, aby skonfigurować Xdebug:

```ini
SAIL_XDEBUG_MODE=develop,debug,coverage
```

Następnie upewnij się, że Twój opublikowany plik `php.ini` zawiera następującą konfigurację, aby Xdebug był aktywowany w określonych trybach:

```ini
[xdebug]
xdebug.mode=${XDEBUG_MODE}
```

Po zmodyfikowaniu pliku `php.ini`, pamiętaj o przebudowaniu obrazów Docker, aby Twoje zmiany w pliku `php.ini` zostały wprowadzone:

```shell
sail build --no-cache
```

#### Konfiguracja Linux Host IP

Wewnętrznie zmienna środowiskowa `XDEBUG_CONFIG` jest zdefiniowana jako `client_host=host.docker.internal`, aby Xdebug był poprawnie skonfigurowany dla Mac i Windows (WSL2). Jeśli Twój lokalny komputer działa na Linuxie i używasz Dockera 20.10+, `host.docker.internal` jest dostępny i nie jest wymagana żadna ręczna konfiguracja.

Dla wersji Dockera starszych niż 20.10, `host.docker.internal` nie jest obsługiwany na Linuxie i będziesz musiał ręcznie zdefiniować IP hosta. Aby to zrobić, skonfiguruj statyczne IP dla swojego kontenera, definiując niestandardową sieć w pliku `compose.yaml`:

```yaml
networks:
  custom_network:
    ipam:
      config:
        - subnet: 172.20.0.0/16

services:
  laravel.test:
    networks:
      custom_network:
        ipv4_address: 172.20.0.2
```

Po ustawieniu statycznego IP, zdefiniuj zmienną SAIL_XDEBUG_CONFIG w pliku .env Twojej aplikacji:

```ini
SAIL_XDEBUG_CONFIG="client_host=172.20.0.2"
```

<a name="xdebug-cli-usage"></a>
### Użycie Xdebug CLI

Polecenie `sail debug` może być używane do rozpoczęcia sesji debugowania podczas uruchamiania polecenia Artisan:

```shell
# Uruchom polecenie Artisan bez Xdebug...
sail artisan migrate

# Uruchom polecenie Artisan z Xdebug...
sail debug migrate
```

<a name="xdebug-browser-usage"></a>
### Użycie Xdebug w Przeglądarce

Aby debugować swoją aplikację podczas interakcji z aplikacją przez przeglądarkę internetową, postępuj zgodnie z [instrukcjami dostarczonymi przez Xdebug](https://xdebug.org/docs/step_debug#web-application) w celu zainicjowania sesji Xdebug z przeglądarki internetowej.

Jeśli używasz PhpStorm, przejrzyj dokumentację JetBrains dotyczącą [debugowania bez konfiguracji](https://www.jetbrains.com/help/phpstorm/zero-configuration-debugging.html).

> [!WARNING]
> Laravel Sail polega na `artisan serve`, aby obsługiwać Twoją aplikację. Polecenie `artisan serve` akceptuje tylko zmienne `XDEBUG_CONFIG` i `XDEBUG_MODE` od wersji Laravel 8.53.0. Starsze wersje Laravel (8.52.0 i niżej) nie obsługują tych zmiennych i nie będą akceptować połączeń debugowania.

<a name="sail-customization"></a>
## Dostosowanie

Ponieważ Sail to po prostu Docker, możesz dowolnie dostosować niemal wszystko w nim. Aby opublikować własne pliki Dockerfile Sail, możesz wykonać polecenie `sail:publish`:

```shell
sail artisan sail:publish
```

Po uruchomieniu tego polecenia pliki Dockerfile i inne pliki konfiguracyjne używane przez Laravel Sail zostaną umieszczone w katalogu `docker` w głównym katalogu Twojej aplikacji. Po dostosowaniu instalacji Sail możesz chcieć zmienić nazwę obrazu dla kontenera aplikacji w pliku `compose.yaml` Twojej aplikacji. Po wykonaniu tego przebuduj kontenery aplikacji za pomocą polecenia `build`. Przypisanie unikalnej nazwy obrazowi aplikacji jest szczególnie ważne, jeśli używasz Sail do tworzenia wielu aplikacji Laravel na jednym komputerze:

```shell
sail build --no-cache
```
