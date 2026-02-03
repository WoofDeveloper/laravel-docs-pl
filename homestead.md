# Laravel Homestead

- [Wprowadzenie](#introduction)
- [Instalacja i konfiguracja](#installation-and-setup)
    - [Pierwsze kroki](#first-steps)
    - [Konfiguracja Homestead](#configuring-homestead)
    - [Konfiguracja stron Nginx](#configuring-nginx-sites)
    - [Konfiguracja usług](#configuring-services)
    - [Uruchamianie Vagrant Box](#launching-the-vagrant-box)
    - [Instalacja per projekt](#per-project-installation)
    - [Instalacja opcjonalnych funkcji](#installing-optional-features)
    - [Aliasy](#aliases)
- [Aktualizacja Homestead](#updating-homestead)
- [Codzienne użytkowanie](#daily-usage)
    - [Łączenie przez SSH](#connecting-via-ssh)
    - [Dodawanie dodatkowych stron](#adding-additional-sites)
    - [Zmienne środowiskowe](#environment-variables)
    - [Porty](#ports)
    - [Wersje PHP](#php-versions)
    - [Łączenie z bazami danych](#connecting-to-databases)
    - [Kopie zapasowe baz danych](#database-backups)
    - [Konfiguracja harmonogramów Cron](#configuring-cron-schedules)
    - [Konfiguracja Mailpit](#configuring-mailpit)
    - [Konfiguracja Minio](#configuring-minio)
    - [Laravel Dusk](#laravel-dusk)
    - [Udostępnianie środowiska](#sharing-your-environment)
- [Debugowanie i profilowanie](#debugging-and-profiling)
    - [Debugowanie żądań web z Xdebug](#debugging-web-requests)
    - [Debugowanie aplikacji CLI](#debugging-cli-applications)
    - [Profilowanie aplikacji z Blackfire](#profiling-applications-with-blackfire)
- [Interfejsy sieciowe](#network-interfaces)
- [Rozszerzanie Homestead](#extending-homestead)
- [Ustawienia specyficzne dla dostawcy](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## Wprowadzenie

> [!WARNING]
> Laravel Homestead to starszy pakiet, który nie jest już aktywnie utrzymywany. [Laravel Sail](/docs/{{version}}/sail) może być używany jako nowoczesna alternatywa.

Laravel dąży do tego, aby całe doświadczenie związane z rozwojem PHP było przyjemne, włączając w to Twoje lokalne środowisko deweloperskie. [Laravel Homestead](https://github.com/laravel/homestead) to oficjalny, wstępnie skonfigurowany Vagrant box, który zapewnia wspaniałe środowisko deweloperskie bez wymagania instalowania PHP, serwera web lub innego oprogramowania serwerowego na Twoim lokalnym komputerze.

[Vagrant](https://www.vagrantup.com) zapewnia prosty, elegancki sposób zarządzania i konfigurowania maszyn wirtualnych. Vagrant boxy są całkowicie jednorazowe. Jeśli coś pójdzie nie tak, możesz zniszczyć i odtworzyć box w kilka minut!

Homestead działa na każdym systemie Windows, macOS lub Linux i zawiera Nginx, PHP, MySQL, PostgreSQL, Redis, Memcached, Node i wszystkie inne oprogramowanie potrzebne do tworzenia niesamowitych aplikacji Laravel.

> [!WARNING]
> Jeśli używasz Windows, możesz potrzebować włączyć wirtualizację sprzętową (VT-x). Zazwyczaj można ją włączyć przez BIOS. Jeśli używasz Hyper-V na systemie UEFI, możesz dodatkowo potrzebować wyłączyć Hyper-V, aby uzyskać dostęp do VT-x.

<a name="included-software"></a>
### Dołączone oprogramowanie

<style>
    #software-list > ul {
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style>

<div id="software-list" markdown="1">

- Ubuntu 22.04
- Git
- PHP 8.3
- PHP 8.2
- PHP 8.1
- PHP 8.0
- PHP 7.4
- PHP 7.3
- PHP 7.2
- PHP 7.1
- PHP 7.0
- PHP 5.6
- Nginx
- MySQL 8.0
- lmm
- Sqlite3
- PostgreSQL 15
- Composer
- Docker
- Node (With Yarn, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd
- Mailpit
- avahi
- ngrok
- Xdebug
- XHProf / Tideways / XHGui
- wp-cli

</div>

<a name="optional-software"></a>
### Opcjonalne oprogramowanie

<style>
    #software-list > ul {
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style>

<div id="software-list" markdown="1">

- Apache
- Blackfire
- Cassandra
- Chronograf
- CouchDB
- Crystal & Lucky Framework
- Elasticsearch
- EventStoreDB
- Flyway
- Gearman
- Go
- Grafana
- InfluxDB
- Logstash
- MariaDB
- Meilisearch
- MinIO
- MongoDB
- Neo4j
- Oh My Zsh
- Open Resty
- PM2
- Python
- R
- RabbitMQ
- Rust
- RVM (Ruby Version Manager)
- Solr
- TimescaleDB
- Trader <small>(rozszerzenie PHP)</small>
- Webdriver & Laravel Dusk Utilities

</div>

<a name="installation-and-setup"></a>
## Instalacja i konfiguracja

<a name="first-steps"></a>
### Pierwsze kroki

Przed uruchomieniem środowiska Homestead musisz zainstalować [Vagrant](https://developer.hashicorp.com/vagrant/downloads) oraz jednego z następujących obsługiwanych dostawców:

- [VirtualBox 6.1.x](https://www.virtualbox.org/wiki/Download_Old_Builds_6_1)
- [Parallels](https://www.parallels.com/products/desktop/)

Wszystkie te pakiety oprogramowania zapewniają łatwe w użyciu wizualne instalatory dla wszystkich popularnych systemów operacyjnych.

Aby użyć dostawcy Parallels, musisz zainstalować [wtyczkę Parallels Vagrant](https://github.com/Parallels/vagrant-parallels). Jest to bezpłatne.

<a name="installing-homestead"></a>
#### Instalacja Homestead

Możesz zainstalować Homestead klonując repozytorium Homestead na swoją maszynę host. Rozważ sklonowanie repozytorium do folderu `Homestead` w Twoim katalogu "domowym", ponieważ maszyna wirtualna Homestead będzie służyć jako host dla wszystkich Twoich aplikacji Laravel. W tej dokumentacji będziemy nazywać ten katalog "katalogiem Homestead":

```shell
git clone https://github.com/laravel/homestead.git ~/Homestead
```

Po sklonowaniu repozytorium Laravel Homestead powinieneś przełączyć się na gałąź `release`. Ta gałąź zawsze zawiera najnowsze stabilne wydanie Homestead:

```shell
cd ~/Homestead

git checkout release
```

Następnie wykonaj polecenie `bash init.sh` z katalogu Homestead, aby utworzyć plik konfiguracyjny `Homestead.yaml`. Plik `Homestead.yaml` to miejsce, w którym skonfigurujesz wszystkie ustawienia dla instalacji Homestead. Ten plik zostanie umieszczony w katalogu Homestead:

```shell
# macOS / Linux...
bash init.sh

# Windows...
init.bat
```

<a name="configuring-homestead"></a>
### Konfiguracja Homestead

<a name="setting-your-provider"></a>
#### Ustawianie dostawcy

Klucz `provider` w pliku `Homestead.yaml` wskazuje, który dostawca Vagrant powinien być użyty: `virtualbox` lub `parallels`:

    provider: virtualbox

> [!WARNING]
> Jeśli używasz Apple Silicon, wymagany jest dostawca Parallels.

<a name="configuring-shared-folders"></a>
#### Konfiguracja udostępnionych folderów

Właściwość `folders` w pliku `Homestead.yaml` wymienia wszystkie foldery, które chcesz udostępnić środowisku Homestead. W miarę zmiany plików w tych folderach będą one synchronizowane między Twoim lokalnym komputerem a wirtualnym środowiskiem Homestead. Możesz skonfigurować tyle udostępnionych folderów, ile potrzebujesz:

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
```

> [!WARNING]
> Użytkownicy Windows nie powinni używać składni ścieżki `~/`, zamiast tego powinni używać pełnej ścieżki do swojego projektu, takiej jak `C:\Users\user\Code\project1`.

Zawsze powinieneś mapować poszczególne aplikacje do ich własnych mapowań folderów zamiast mapowania jednego dużego katalogu zawierającego wszystkie Twoje aplikacje. Gdy mapujesz folder, maszyna wirtualna musi śledzić wszystkie operacje dyskowe dla *każdego* pliku w folderze. Możesz doświadczyć obniżonej wydajności, jeśli masz dużą liczbę plików w folderze:

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
    - map: ~/code/project2
      to: /home/vagrant/project2
```

> [!WARNING]
> Nigdy nie powinieneś montować `.` (bieżący katalog) podczas używania Homestead. To powoduje, że Vagrant nie mapuje bieżącego folderu do `/vagrant` i złamie opcjonalne funkcje oraz spowoduje nieoczekiwane wyniki podczas provisioningu.

Aby włączyć [NFS](https://developer.hashicorp.com/vagrant/docs/synced-folders/nfs), możesz dodać opcję `type` do mapowania folderu:

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "nfs"
```

> [!WARNING]
> Podczas używania NFS na Windows powinieneś rozważyć zainstalowanie wtyczki [vagrant-winnfsd](https://github.com/winnfsd/vagrant-winnfsd). Ta wtyczka będzie utrzymywać prawidłowe uprawnienia użytkownika / grupy dla plików i katalogów w maszynie wirtualnej Homestead.

Możesz również przekazać wszelkie opcje obsługiwane przez [Synced Folders](https://developer.hashicorp.com/vagrant/docs/synced-folders/basic_usage) Vagrant, wymieniając je pod kluczem `options`:

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "rsync"
      options:
          rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
          rsync__exclude: ["node_modules"]
```

<a name="configuring-nginx-sites"></a>
### Konfiguracja stron Nginx

Nie znasz Nginx? Nie ma problemu. Właściwość `sites` Twojego pliku `Homestead.yaml` pozwala łatwo mapować "domenę" do folderu w środowisku Homestead. Przykładowa konfiguracja strony jest zawarta w pliku `Homestead.yaml`. Ponownie, możesz dodać tyle stron do środowiska Homestead, ile potrzebujesz. Homestead może służyć jako wygodne, zwirtualizowane środowisko dla każdej aplikacji Laravel, nad którą pracujesz:

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
```

Jeśli zmienisz właściwość `sites` po prowizjonowaniu maszyny wirtualnej Homestead, powinieneś wykonać polecenie `vagrant reload --provision` w terminalu, aby zaktualizować konfigurację Nginx na maszynie wirtualnej.

> [!WARNING]
> Skrypty Homestead zostały zbudowane tak, aby były jak najbardziej idempotentne. Jednak jeśli doświadczasz problemów podczas provisioningu, powinieneś zniszczyć i odbudować maszynę, wykonując polecenie `vagrant destroy && vagrant up`.

<a name="hostname-resolution"></a>
#### Rozpoznawanie nazw hostów

Homestead publikuje nazwy hostów za pomocą `mDNS` do automatycznego rozpoznawania hostów. Jeśli ustawisz `hostname: homestead` w pliku `Homestead.yaml`, host będzie dostępny pod adresem `homestead.local`. Dystrybucje desktopowe macOS, iOS i Linux zawierają domyślnie obsługę `mDNS`. Jeśli używasz Windows, musisz zainstalować [Bonjour Print Services for Windows](https://support.apple.com/kb/DL999?viewlocale=en_US&locale=en_US).

Używanie automatycznych nazw hostów działa najlepiej dla [instalacji per projekt](#per-project-installation) Homestead. Jeśli hostujesz wiele stron na jednej instancji Homestead, możesz dodać "domeny" dla swoich stron internetowych do pliku `hosts` na swojej maszynie. Plik `hosts` przekieruje żądania dla stron Homestead do Twojej maszyny wirtualnej Homestead. Na macOS i Linuxie, ten plik znajduje się w `/etc/hosts`. Na Windows znajduje się w `C:\\Windows\\System32\\drivers\\etc\\hosts`. Linie, które dodajesz do tego pliku, będą wyglądać następująco:

```text
192.168.56.56  homestead.test
```

Upewnij się, że adres IP wymieniony jest tym ustawionym w pliku `Homestead.yaml`. Po dodaniu domeny do pliku `hosts` i uruchomieniu Vagrant box, będziesz mógł uzyskać dostęp do strony przez przeglądarkę internetową:

```shell
http://homestead.test
```

<a name="configuring-services"></a>
### Konfiguracja usług

Homestead domyślnie uruchamia kilka usług; jednak możesz dostosować, które usługi są włączone lub wyłączone podczas provisioningu. Na przykład, możesz włączyć PostgreSQL i wyłączyć MySQL, modyfikując opcję `services` w pliku `Homestead.yaml`:

```yaml
services:
    - enabled:
        - "postgresql"
    - disabled:
        - "mysql"
```

Określone usługi będą uruchamiane lub zatrzymywane na podstawie ich kolejności w dyrektywach `enabled` i `disabled`.

<a name="launching-the-vagrant-box"></a>
### Uruchamianie Vagrant Box

Po edycji pliku `Homestead.yaml` według swoich potrzeb, uruchom polecenie `vagrant up` z katalogu Homestead. Vagrant uruchomi maszynę wirtualną i automatycznie skonfiguruje Twoje udostępnione foldery i strony Nginx.

Aby zniszczyć maszynę, możesz użyć polecenia `vagrant destroy`.

<a name="per-project-installation"></a>
### Instalacja per projekt

Zamiast instalować Homestead globalnie i udostępniać tę samą maszynę wirtualną Homestead we wszystkich projektach, możesz zamiast tego skonfigurować instancję Homestead dla każdego zarządzanego projektu. Instalowanie Homestead per projekt może być korzystne, jeśli chcesz dołączyć `Vagrantfile` do swojego projektu, pozwalając innym pracującym nad projektem na wykonanie `vagrant up` natychmiast po sklonowaniu repozytorium projektu.

Możesz zainstalować Homestead w swoim projekcie używając menedżera pakietów Composer:

```shell
composer require laravel/homestead --dev
```

Po zainstalowaniu Homestead, wywołaj polecenie `make` Homestead, aby wygenerować pliki `Vagrantfile` i `Homestead.yaml` dla swojego projektu. Te pliki zostaną umieszczone w głównym katalogu Twojego projektu. Polecenie `make` automatycznie skonfiguruje dyrektywy `sites` i `folders` w pliku `Homestead.yaml`:

```shell
# macOS / Linux...
php vendor/bin/homestead make

# Windows...
vendor\\bin\\homestead make
```

Następnie uruchom polecenie `vagrant up` w terminalu i uzyskaj dostęp do swojego projektu pod adresem `http://homestead.test` w przeglądarce. Pamiętaj, że nadal będziesz musiał dodać wpis `/etc/hosts` dla `homestead.test` lub domeny według własnego wyboru, jeśli nie używasz automatycznego [rozpoznawania nazw hostów](#hostname-resolution).

<a name="installing-optional-features"></a>
### Instalacja opcjonalnych funkcji

Opcjonalne oprogramowanie jest instalowane za pomocą opcji `features` w pliku `Homestead.yaml`. Większość funkcji może być włączana lub wyłączana za pomocą wartości boolean, podczas gdy niektóre funkcje pozwalają na wiele opcji konfiguracyjnych:

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
    - cassandra: true
    - chronograf: true
    - couchdb: true
    - crystal: true
    - dragonflydb: true
    - elasticsearch:
        version: 7.9.0
    - eventstore: true
        version: 21.2.0
    - flyway: true
    - gearman: true
    - golang: true
    - grafana: true
    - influxdb: true
    - logstash: true
    - mariadb: true
    - meilisearch: true
    - minio: true
    - mongodb: true
    - neo4j: true
    - ohmyzsh: true
    - openresty: true
    - pm2: true
    - python: true
    - r-base: true
    - rabbitmq: true
    - rustc: true
    - rvm: true
    - solr: true
    - timescaledb: true
    - trader: true
    - webdriver: true
```

<a name="elasticsearch"></a>
#### Elasticsearch

Możesz określić obsługiwaną wersję Elasticsearch, która musi być dokładnym numerem wersji (major.minor.patch). Domyślna instalacja stworzy klaster o nazwie 'homestead'. Nigdy nie powinieś przydzielać Elasticsearch więcej niż połowę pamięci systemu operacyjnego, więc upewnij się, że Twoja maszyna wirtualna Homestead ma co najmniej dwukrotność alokacji Elasticsearch.

> [!NOTE]
> Sprawdź [dokumentację Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current), aby dowiedzieć się, jak dostosować konfigurację.

<a name="mariadb"></a>
#### MariaDB

Włączenie MariaDB usunie MySQL i zainstaluje MariaDB. MariaDB zazwyczaj służy jako zastępstwo dla MySQL, więc powinieś nadal używać sterownika bazy danych `mysql` w konfiguracji bazy danych swojej aplikacji.

<a name="mongodb"></a>
#### MongoDB

Domyślna instalacja MongoDB ustawi nazwę użytkownika bazy danych na `homestead`, a odpowiadające hasło na `secret`.

<a name="neo4j"></a>
#### Neo4j

Domyślna instalacja Neo4j ustawi nazwę użytkownika bazy danych na `homestead`, a odpowiadające hasło na `secret`. Aby uzyskać dostęp do przeglądarki Neo4j, odwiedź `http://homestead.test:7474` w przeglądarce internetowej. Porty `7687` (Bolt), `7474` (HTTP) i `7473` (HTTPS) są gotowe do obsługi żądań od klienta Neo4j.

<a name="aliases"></a>
### Aliasy

Możesz dodać aliasy Bash do swojej maszyny wirtualnej Homestead, modyfikując plik `aliases` w katalogu Homestead:

```shell
alias c='clear'
alias ..='cd ..'
```

Po zaktualizowaniu pliku `aliases` powinieś ponownie prowizjonować maszynę wirtualną Homestead za pomocą polecenia `vagrant reload --provision`. To zapewni, że Twoje nowe aliasy będą dostępne na maszynie.

<a name="updating-homestead"></a>
## Aktualizacja Homestead

Zanim zaczniesz aktualizować Homestead, powinieś się upewnić, że usunąłeś swoją bieżącą maszynę wirtualną, uruchamiając następujące polecenie w katalogu Homestead:

```shell
vagrant destroy
```

Następnie musisz zaktualizować kod źródłowy Homestead. Jeśli sklonowałeś repozytorium, możesz wykonać następujące polecenia w lokalizacji, w której pierwotnie sklonowałeś repozytorium:

```shell
git fetch

git pull origin release
```

Te polecenia pobierają najnowszy kod Homestead z repozytorium GitHub, pobierają najnowsze tagi, a następnie sprawdzają najnowsze oznaczone wydanie. Możesz znaleźć najnowszą stabilną wersję wydania na [stronie wydania GitHub](https://github.com/laravel/homestead/releases) Homestead.

Jeśli zainstalowałeś Homestead przez plik `composer.json` swojego projektu, powinieś się upewnić, że plik `composer.json` zawiera `"laravel/homestead": "^12"` i zaktualizować zależności:

```shell
composer update
```

Następnie powinieś zaktualizować Vagrant box za pomocą polecenia `vagrant box update`:

```shell
vagrant box update
```

Po zaktualizowaniu Vagrant box powinieś uruchomić polecenie `bash init.sh` z katalogu Homestead, aby zaktualizować dodatkowe pliki konfiguracyjne Homestead. Zostaniesz zapytany, czy chcesz nadpisać istniejące pliki `Homestead.yaml`, `after.sh` i `aliases`:

```shell
# macOS / Linux...
bash init.sh

# Windows...
init.bat
```

W końcu będziesz musiał ponownie wygenerować swoją maszynę wirtualną Homestead, aby używać najnowszej instalacji Vagrant:

```shell
vagrant up
```

<a name="daily-usage"></a>
## Codzienne użytkowanie

<a name="connecting-via-ssh"></a>
### Łączenie przez SSH

Możesz połączyć się SSH z maszyną wirtualną, wykonując polecenie terminala `vagrant ssh` z katalogu Homestead.

<a name="adding-additional-sites"></a>
### Dodawanie dodatkowych stron

Po provizjonowaniu i uruchomieniu środowiska Homestead możesz chcieć dodać dodatkowe strony Nginx dla innych projektów Laravel. Możesz uruchomić tyle projektów Laravel, ile chcesz, w jednym środowisku Homestead. Aby dodać dodatkową stronę, dodaj stronę do pliku `Homestead.yaml`.

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
    - map: another.test
      to: /home/vagrant/project2/public
```

> [!WARNING]
> Powinieś się upewnić, że skonfigurowałeś [mapowanie folderów](#configuring-shared-folders) dla katalogu projektu przed dodaniem strony.

Jeśli Vagrant nie zarządza automatycznie Twoim plikiem "hosts", możesz również potrzebować dodać nową stronę do tego pliku. Na macOS i Linuxie ten plik znajduje się w `/etc/hosts`. Na Windows znajduje się w `C:\Windows\System32\drivers\etc\hosts`:

```text
192.168.56.56  homestead.test
192.168.56.56  another.test
```

Po dodaniu strony wykonaj polecenie terminala `vagrant reload --provision` z katalogu Homestead.

<a name="site-types"></a>
#### Typy stron

Homestead obsługuje kilka "typów" stron, które pozwalają łatwo uruchamiać projekty, które nie są oparte na Laravel. Na przykład możemy łatwo dodać aplikację Statamic do Homestead, używając typu strony `statamic`:

```yaml
sites:
    - map: statamic.test
      to: /home/vagrant/my-symfony-project/web
      type: "statamic"
```

Dostępne typy stron to: `apache`, `apache-proxy`, `apigility`, `expressive`, `laravel` (domyślnie), `proxy` (dla nginx), `silverstripe`, `statamic`, `symfony2`, `symfony4` i `zf`.

<a name="site-parameters"></a>
#### Parametry strony

Możesz dodać dodatkowe wartości `fastcgi_param` Nginx do swojej strony poprzez dyrektywy `params` strony:

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      params:
          - key: FOO
            value: BAR
```

<a name="environment-variables"></a>
### Zmienne środowiskowe

Możesz zdefiniować globalne zmienne środowiskowe, dodając je do pliku `Homestead.yaml`:

```yaml
variables:
    - key: APP_ENV
      value: local
    - key: FOO
      value: bar
```

Po zaktualizowaniu pliku `Homestead.yaml` upewnij się, że ponownie prowizjonujesz maszynę, wykonując polecenie `vagrant reload --provision`. To zaktualizuje konfigurację PHP-FPM dla wszystkich zainstalowanych wersji PHP, a także zaktualizuje środowisko dla użytkownika `vagrant`.

<a name="ports"></a>
### Porty

Domyślnie następujące porty są przekierowywane do środowiska Homestead:

<div class="content-list" markdown="1">

- **HTTP:** 8000 &rarr; Przekierowuje do 80
- **HTTPS:** 44300 &rarr; Przekierowuje do 443

</div>

<a name="forwarding-additional-ports"></a>
#### Przekierowywanie dodatkowych portów

Jeśli chcesz, możesz przekierować dodatkowe porty do Vagrant box, definiując wpis konfiguracyjny `ports` w pliku `Homestead.yaml`. Po zaktualizowaniu pliku `Homestead.yaml` upewnij się, że ponownie prowizjonujesz maszynę, wykonując polecenie `vagrant reload --provision`:

```yaml
ports:
    - send: 50000
      to: 5000
    - send: 7777
      to: 777
      protocol: udp
```

Poniżej znajduje się lista dodatkowych portów usług Homestead, które możesz chcieć zmapować z maszyny hosta do Vagrant box:

<div class="content-list" markdown="1">

- **SSH:** 2222 &rarr; Do 22
- **ngrok UI:** 4040 &rarr; Do 4040
- **MySQL:** 33060 &rarr; Do 3306
- **PostgreSQL:** 54320 &rarr; Do 5432
- **MongoDB:** 27017 &rarr; Do 27017
- **Mailpit:** 8025 &rarr; Do 8025
- **Minio:** 9600 &rarr; Do 9600

</div>

<a name="php-versions"></a>
### Wersje PHP

Homestead obsługuje uruchamianie wielu wersji PHP na tej samej maszynie wirtualnej. Możesz określić, która wersja PHP ma być używana dla danej strony w pliku `Homestead.yaml`. Dostępne wersje PHP to: "5.6", "7.0", "7.1", "7.2", "7.3", "7.4", "8.0", "8.1", "8.2" i "8.3" (domyślnie):

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      php: "7.1"
```

[W maszynie wirtualnej Homestead](#connecting-via-ssh) możesz używać dowolnej z obsługiwanych wersji PHP poprzez CLI:

```shell
php5.6 artisan list
php7.0 artisan list
php7.1 artisan list
php7.2 artisan list
php7.3 artisan list
php7.4 artisan list
php8.0 artisan list
php8.1 artisan list
php8.2 artisan list
php8.3 artisan list
```

Możesz zmienić domyślną wersję PHP używaną przez CLI, wydając następujące polecenia z maszyny wirtualnej Homestead:

```shell
php56
php70
php71
php72
php73
php74
php80
php81
php82
php83
```

<a name="connecting-to-databases"></a>
### Łączenie z bazami danych

Baza danych `homestead` jest skonfigurowana dla zarówno MySQL, jak i PostgreSQL od razu. Aby połączyć się z bazą danych MySQL lub PostgreSQL z klienta bazy danych na maszynie hosta, powinieś połączyć się z `127.0.0.1` na porcie `33060` (MySQL) lub `54320` (PostgreSQL). Nazwa użytkownika i hasło dla obu baz danych to `homestead` / `secret`.

> [!WARNING]
> Powinieś używać tylko tych niestandardowych portów podczas łączenia się z bazami danych z maszyny hosta. Użyjesz domyślnych portów 3306 i 5432 w pliku konfiguracyjnym `database` aplikacji Laravel, ponieważ Laravel działa _wewnątrz_ maszyny wirtualnej.

<a name="database-backups"></a>
### Kopie zapasowe baz danych

Homestead może automatycznie tworzyć kopie zapasowe bazy danych, gdy Twoja maszyna wirtualna Homestead jest niszczona. Aby użyć tej funkcji, musisz używać Vagrant 2.1.0 lub nowszej wersji. Lub, jeśli używasz starszej wersji Vagrant, musisz zainstalować wtyczkę `vagrant-triggers`. Aby włączyć automatyczne kopie zapasowe baz danych, dodaj następującą linię do pliku `Homestead.yaml`:

```yaml
backup: true
```

Po skonfigurowaniu Homestead wyeksportuje Twoje bazy danych do katalogów `.backup/mysql_backup` i `.backup/postgres_backup`, gdy zostanie wykonane polecenie `vagrant destroy`. Te katalogi można znaleźć w folderze, w którym zainstalowałeś Homestead lub w katalogu głównym Twojego projektu, jeśli używasz metody [instalacji per projekt](#per-project-installation).

<a name="configuring-cron-schedules"></a>
### Konfiguracja harmonogramów Cron

Laravel zapewnia wygodny sposób [planowania zadań cron](/docs/{{version}}/scheduling), planując pojedyncze polecenie Artisan `schedule:run` do uruchamiania co minutę. Polecenie `schedule:run` zbada harmonogram zadań zdefiniowany w pliku `routes/console.php`, aby określić, które zaplanowane zadania uruchomić.

Jeśli chcesz, aby polecenie `schedule:run` było uruchamiane dla strony Homestead, możesz ustawić opcję `schedule` na `true` podczas definiowania strony:

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      schedule: true
```

Zadanie cron dla strony zostanie zdefiniowane w katalogu `/etc/cron.d` maszyny wirtualnej Homestead.

<a name="configuring-mailpit"></a>
### Konfiguracja Mailpit

[Mailpit](https://github.com/axllent/mailpit) pozwala przechwytywać wychodzącą pocztę e-mail i badać ją bez faktycznego wysyłania poczty do odbiorców. Aby zacząć, zaktualizuj plik `.env` swojej aplikacji, aby używać następujących ustawień poczty:

```ini
MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

Po skonfigurowaniu Mailpit możesz uzyskać dostęp do pulpitu Mailpit pod adresem `http://localhost:8025`.

<a name="configuring-minio"></a>
### Konfiguracja Minio

[Minio](https://github.com/minio/minio) to serwer przechowywania obiektów open source z API kompatybilnym z Amazon S3. Aby zainstalować Minio, zaktualizuj plik `Homestead.yaml` następującą opcją konfiguracyjną w sekcji [features](#installing-optional-features):

    minio: true

Domyślnie Minio jest dostępny na porcie 9600. Możesz uzyskać dostęp do panelu sterowania Minio, odwiedzając `http://localhost:9600`. Domyślny klucz dostępu to `homestead`, a domyślny klucz tajny to `secretkey`. Podczas uzyskiwania dostępu do Minio zawsze powinieś używać regionu `us-east-1`.

Aby używać Minio, upewnij się, że plik `.env` ma następujące opcje:

```ini
AWS_USE_PATH_STYLE_ENDPOINT=true
AWS_ENDPOINT=http://localhost:9600
AWS_ACCESS_KEY_ID=homestead
AWS_SECRET_ACCESS_KEY=secretkey
AWS_DEFAULT_REGION=us-east-1
```

Aby prowizjonować pojemniki "S3" zasilane przez Minio, dodaj dyrektywy `buckets` do pliku `Homestead.yaml`. Po zdefiniowaniu pojemników powinieś wykonać polecenie `vagrant reload --provision` w terminalu:

```yaml
buckets:
    - name: your-bucket
      policy: public
    - name: your-private-bucket
      policy: none
```

Obsługiwane wartości `policy` to: `none`, `download`, `upload` i `public`.

<a name="laravel-dusk"></a>
### Laravel Dusk

Aby uruchomić testy [Laravel Dusk](/docs/{{version}}/dusk) w Homestead, powinieś włączyć [funkcję webdriver](#installing-optional-features) w konfiguracji Homestead:

```yaml
features:
    - webdriver: true
```

Po włączeniu funkcji `webdriver` powinieś wykonać polecenie `vagrant reload --provision` w terminalu.

<a name="sharing-your-environment"></a>
### Udostępnianie środowiska

Czasami możesz chcieć udostępnić to, nad czym aktualnie pracujesz, współpracownikom lub klientowi. Vagrant ma wbudowaną obsługę tego poprzez polecenie `vagrant share`; jednak nie będzie to działać, jeśli masz wiele stron skonfigurowanych w pliku `Homestead.yaml`.

Aby rozwiązać ten problem, Homestead zawiera własne polecenie `share`. Aby zacząć, [połącz się SSH z maszyną wirtualną Homestead](#connecting-via-ssh) poprzez `vagrant ssh` i wykonaj polecenie `share homestead.test`. To polecenie udostępni stronę `homestead.test` z pliku konfiguracyjnego `Homestead.yaml`. Możesz podstawić dowolną z innych skonfigurowanych stron za `homestead.test`:

```shell
share homestead.test
```

Po uruchomieniu polecenia zobaczysz ekran Ngrok, który zawiera dziennik aktywności i publicznie dostępne adresy URL dla udostępnianej strony. Jeśli chcesz określić niestandardowy region, subdomenę lub inną opcję uruchomieniową Ngrok, możesz dodać je do polecenia `share`:

```shell
share homestead.test -region=eu -subdomain=laravel
```

Jeśli musisz udostępnić treść przez HTTPS zamiast HTTP, użycie polecenia `sshare` zamiast `share` umożliwi Ci to.

> [!WARNING]
> Pamiętaj, że Vagrant jest z natury niezabezpieczony i udostępniasz swoją maszynę wirtualną Internetowi, uruchamiając polecenie `share`.

<a name="debugging-and-profiling"></a>
## Debugowanie i profilowanie

<a name="debugging-web-requests"></a>
### Debugowanie żądań web z Xdebug

Homestead zawiera obsługę debugowania krokowego przy użyciu [Xdebug](https://xdebug.org). Na przykład możesz uzyskać dostęp do strony w przeglądarce, a PHP połączy się z Twoim IDE, aby umożliwić inspekcję i modyfikację uruchomionego kodu.

Domyślnie Xdebug jest już uruchomiony i gotowy do akceptowania połączeń. Jeśli musisz włączyć Xdebug w CLI, wykonaj polecenie `sudo phpenmod xdebug` w maszynie wirtualnej Homestead. Następnie postępuj zgodnie z instrukcjami swojego IDE, aby włączyć debugowanie. Na koniec skonfiguruj przeglądarkę, aby wywoływać Xdebug za pomocą rozszerzenia lub [bookmarkletu](https://www.jetbrains.com/phpstorm/marklets/).

> [!WARNING]
> Xdebug powoduje, że PHP działa znacznie wolniej. Aby wyłączyć Xdebug, uruchom `sudo phpdismod xdebug` w maszynie wirtualnej Homestead i zrestartuj usługę FPM.

<a name="autostarting-xdebug"></a>
#### Automatyczne uruchamianie Xdebug

Podczas debugowania testów funkcjonalnych, które wysyłają żądania do serwera web, łatwiej jest automatycznie uruchomić debugowanie, zamiast modyfikować testy, aby przekazywały niestandardowy nagłówek lub ciasteczko w celu uruchomienia debugowania. Aby wymusić automatyczne uruchamianie Xdebug, zmodyfikuj plik `/etc/php/7.x/fpm/conf.d/20-xdebug.ini` w maszynie wirtualnej Homestead i dodaj następującą konfigurację:

```ini
; Jeśli Homestead.yaml zawiera inną podsieć dla adresu IP, ten adres może być inny...
xdebug.client_host = 192.168.10.1
xdebug.mode = debug
xdebug.start_with_request = yes
```

<a name="debugging-cli-applications"></a>
### Debugowanie aplikacji CLI

Aby debugować aplikację PHP CLI, użyj aliasu powłoki `xphp` wewnątrz maszyny wirtualnej Homestead:

```shell
xphp /path/to/script
```

<a name="profiling-applications-with-blackfire"></a>
### Profilowanie aplikacji z Blackfire

[Blackfire](https://blackfire.io/docs/introduction) to usługa do profilowania żądań web i aplikacji CLI. Oferuje interaktywny interfejs użytkownika, który wyświetla dane profilu w wykresach wywołań i osiach czasu. Jest zbudowany do użycia w rozwoju, stagingu i produkcji, bez narzutu dla użytkowników końcowych. Ponadto Blackfire zapewnia kontrole wydajności, jakości i bezpieczeństwa kodu oraz ustawień konfiguracyjnych `php.ini`.

[Blackfire Player](https://blackfire.io/docs/player/index) to aplikacja do indeksowania Web, testowania Web i skrobania Web typu open-source, która może współpracować z Blackfire w celu skryptowania scenariuszy profilowania.

Aby włączyć Blackfire, użyj ustawienia "features" w pliku konfiguracyjnym Homestead:

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
```

Dane uwierzytelniające serwera Blackfire i dane uwierzytelniające klienta [wymagają konta Blackfire](https://blackfire.io/signup). Blackfire oferuje różne opcje profilowania aplikacji, w tym narzędzie CLI i rozszerzenie przeglądarki. Proszę [zapoznać się z dokumentacją Blackfire, aby uzyskać więcej szczegółów](https://blackfire.io/docs/php/integrations/laravel/index).

<a name="network-interfaces"></a>
## Interfejsy sieciowe

Właściwość `networks` w pliku `Homestead.yaml` konfiguruje interfejsy sieciowe dla maszyny wirtualnej Homestead. Możesz skonfigurować tyle interfejsów, ile potrzebujesz:

```yaml
networks:
    - type: "private_network"
      ip: "192.168.10.20"
```

Aby włączyć interfejs [mostkowany](https://developer.hashicorp.com/vagrant/docs/networking/public_network), skonfiguruj ustawienie `bridge` dla sieci i zmień typ sieci na `public_network`:

```yaml
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
```

Aby włączyć [DHCP](https://developer.hashicorp.com/vagrant/docs/networking/public_network#dhcp), po prostu usuń opcję `ip` z konfiguracji:

```yaml
networks:
    - type: "public_network"
      bridge: "en1: Wi-Fi (AirPort)"
```

Aby zaktualizować urządzenie, którego używa sieć, możesz dodać opcję `dev` do konfiguracji sieci. Domyślną wartością `dev` jest `eth0`:

```yaml
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
      dev: "enp2s0"
```

<a name="extending-homestead"></a>
## Rozszerzanie Homestead

Możesz rozszerzyć Homestead za pomocą skryptu `after.sh` w katalogu głównym katalogu Homestead. W tym pliku możesz dodać wszelkie polecenia powłoki, które są niezbędne do prawidłowej konfiguracji i dostosowania maszyny wirtualnej.

Podczas dostosowywania Homestead Ubuntu może zapytać, czy chcesz zachować oryginalną konfigurację pakietu, czy nadpisać ją nowym plikiem konfiguracyjnym. Aby tego uniknąć, powinieś użyć następującego polecenia podczas instalowania pakietów, aby uniknąć nadpisywania jakiejkolwiek konfiguracji wcześniej napisanej przez Homestead:

```shell
sudo apt-get -y \
    -o Dpkg::Options::="--force-confdef" \
    -o Dpkg::Options::="--force-confold" \
    install package-name
```

<a name="user-customizations"></a>
### Dostosowania użytkownika

Podczas używania Homestead z zespołem możesz chcieć dostosować Homestead, aby lepiej dopasować go do swojego osobistego stylu rozwoju. Aby to osiągnąć, możesz stworzyć plik `user-customizations.sh` w katalogu głównym katalogu Homestead (tym samym katalogu zawierającym plik `Homestead.yaml`). W tym pliku możesz wprowadzić dowolne dostosowania; jednak `user-customizations.sh` nie powinien być kontrolowany przez wersjonowanie.

<a name="provider-specific-settings"></a>
## Ustawienia specyficzne dla dostawcy

<a name="provider-specific-virtualbox"></a>
### VirtualBox

<a name="natdnshostresolver"></a>
#### `natdnshostresolver`

Domyślnie Homestead konfiguruje ustawienie `natdnshostresolver` na `on`. To pozwala Homestead używać ustawień DNS Twojego systemu operacyjnego hosta. Jeśli chcesz nadpisać to zachowanie, dodaj następujące opcje konfiguracyjne do pliku `Homestead.yaml`:

```yaml
provider: virtualbox
natdnshostresolver: 'off'
```
