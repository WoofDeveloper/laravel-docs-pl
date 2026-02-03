# Laravel Reverb

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
- [Konfiguracja](#configuration)
    - [Dane uwierzytelniające aplikacji](#application-credentials)
    - [Dozwolone źródła](#allowed-origins)
    - [Dodatkowe aplikacje](#additional-applications)
    - [SSL](#ssl)
- [Uruchamianie serwera](#running-server)
    - [Debugowanie](#debugging)
    - [Restartowanie](#restarting)
- [Monitorowanie](#monitoring)
- [Uruchamianie Reverb w produkcji](#production)
    - [Otwarte pliki](#open-files)
    - [Pętla zdarzeń](#event-loop)
    - [Serwer WWW](#web-server)
    - [Porty](#ports)
    - [Zarządzanie procesami](#process-management)
    - [Skalowanie](#scaling)
- [Zdarzenia](#events)

<a name="introduction"></a>
## Wprowadzenie

[Laravel Reverb](https://github.com/laravel/reverb) zapewnia błyskawiczną i skalowalną komunikację WebSocket w czasie rzeczywistym bezpośrednio do Twojej aplikacji Laravel i zapewnia płynną integrację z istniejącym zestawem [narzędzi do broadcastingu zdarzeń](/docs/{{version}}/broadcasting) Laravela.

<a name="installation"></a>
## Instalacja

Możesz zainstalować Reverb używając komendy Artisan `install:broadcasting`:

```shell
php artisan install:broadcasting
```

<a name="configuration"></a>
## Konfiguracja

W tle komenda Artisan `install:broadcasting` uruchomi komendę `reverb:install`, która zainstaluje Reverb z rozsądnym zestawem domyślnych opcji konfiguracyjnych. Jeśli chcesz wprowadzić jakiekolwiek zmiany w konfiguracji, możesz to zrobić, aktualizując zmienne środowiskowe Reverb lub aktualizując plik konfiguracyjny `config/reverb.php`.

<a name="application-credentials"></a>
### Dane uwierzytelniające aplikacji

Aby nawiązać połączenie z Reverb, musi zostać wymieniony zestaw danych uwierzytelniających "aplikacji" Reverb między klientem a serwerem. Te dane uwierzytelniające są konfigurowane na serwerze i służą do weryfikacji żądania od klienta. Możesz zdefiniować te dane uwierzytelniające za pomocą następujących zmiennych środowiskowych:

```ini
REVERB_APP_ID=my-app-id
REVERB_APP_KEY=my-app-key
REVERB_APP_SECRET=my-app-secret
```

<a name="allowed-origins"></a>
### Dozwolone źródła

Możesz również zdefiniować źródła, z których mogą pochodzić żądania klientów, aktualizując wartość konfiguracji `allowed_origins` w sekcji `apps` pliku konfiguracyjnego `config/reverb.php`. Wszelkie żądania ze źródła niewymienionych w dozwolonych źródłach zostaną odrzucone. Możesz zezwolić na wszystkie źródła używając `*`:

```php
'apps' => [
    [
        'app_id' => 'my-app-id',
        'allowed_origins' => ['laravel.com'],
        // ...
    ]
]
```

<a name="additional-applications"></a>
### Dodatkowe aplikacje

Zazwyczaj Reverb zapewnia serwer WebSocket dla aplikacji, w której jest zainstalowany. Jednak możliwe jest obsługiwanie więcej niż jednej aplikacji przy użyciu pojedynczej instalacji Reverb.

Na przykład możesz chcieć utrzymywać jedną aplikację Laravel, która, za pośrednictwem Reverb, zapewnia łączność WebSocket dla wielu aplikacji. Można to osiągnąć poprzez zdefiniowanie wielu `apps` w pliku konfiguracyjnym `config/reverb.php` aplikacji:

```php
'apps' => [
    [
        'app_id' => 'my-app-one',
        // ...
    ],
    [
        'app_id' => 'my-app-two',
        // ...
    ],
],
```

<a name="ssl"></a>
### SSL

W większości przypadków bezpieczne połączenia WebSocket są obsługiwane przez upstream serwer WWW (Nginx, itp.) zanim żądanie zostanie przekazane do serwera Reverb.

Jednak czasami może być przydatne, na przykład podczas lokalnego rozwoju, aby serwer Reverb obsługiwał bezpieczne połączenia bezpośrednio. Jeśli używasz funkcji bezpiecznej witryny [Laravel Herd](https://herd.laravel.com) lub używasz [Laravel Valet](/docs/{{version}}/valet) i uruchomiłeś [komendę secure](/docs/{{version}}/valet#securing-sites) dla swojej aplikacji, możesz użyć certyfikatu Herd / Valet wygenerowanego dla Twojej witryny, aby zabezpieczyć połączenia Reverb. Aby to zrobić, ustaw zmienną środowiskową `REVERB_HOST` na nazwę hosta Twojej witryny lub jawnie przekaz nazwę hosta podczas uruchamiania serwera Reverb:

```shell
php artisan reverb:start --host="0.0.0.0" --port=8080 --hostname="laravel.test"
```

Ponieważ domeny Herd i Valet rozpoznają się jako `localhost`, uruchomienie powyższej komendy spowoduje, że serwer Reverb będzie dostępny przez bezpieczny protokół WebSocket (`wss`) pod adresem `wss://laravel.test:8080`.

Możesz również ręcznie wybrać certyfikat, definiując opcje `tls` w pliku konfiguracyjnym `config/reverb.php` aplikacji. W tablicy opcji `tls` możesz podać dowolne opcje obsługiwane przez [opcje kontekstu SSL PHP](https://www.php.net/manual/en/context.ssl.php):

```php
'options' => [
    'tls' => [
        'local_cert' => '/path/to/cert.pem'
    ],
],
```

<a name="running-server"></a>
## Uruchamianie serwera

Serwer Reverb może zostać uruchomiony za pomocą komendy Artisan `reverb:start`:

```shell
php artisan reverb:start
```

Domyślnie serwer Reverb zostanie uruchomiony na `0.0.0.0:8080`, co czyni go dostępnym ze wszystkich interfejsów sieciowych.

Jeśli musisz określić niestandardowy host lub port, możesz to zrobić za pomocą opcji `--host` i `--port` podczas uruchamiania serwera:

```shell
php artisan reverb:start --host=127.0.0.1 --port=9000
```

Alternatywnie możesz zdefiniować zmienne środowiskowe `REVERB_SERVER_HOST` i `REVERB_SERVER_PORT` w pliku konfiguracyjnym `.env` aplikacji.

Zmienne środowiskowe `REVERB_SERVER_HOST` i `REVERB_SERVER_PORT` nie powinny być mylone z `REVERB_HOST` i `REVERB_PORT`. Te pierwsze określają host i port, na których ma działać sam serwer Reverb, podczas gdy para ta druga instruuje Laravela, gdzie wysyłać komunikaty broadcast. Na przykład w środowisku produkcyjnym możesz kierować żądania z publicznej nazwy hosta Reverb na porcie `443` do serwera Reverb działającego na `0.0.0.0:8080`. W tym scenariuszu zmienne środowiskowe byłyby zdefiniowane następująco:

```ini
REVERB_SERVER_HOST=0.0.0.0
REVERB_SERVER_PORT=8080

REVERB_HOST=ws.laravel.com
REVERB_PORT=443
```

<a name="debugging"></a>
### Debugowanie

Aby poprawić wydajność, Reverb domyślnie nie wypisuje żadnych informacji debugowania. Jeśli chcesz zobaczyć strumień danych przechodzący przez serwer Reverb, możesz podać opcję `--debug` do komendy `reverb:start`:

```shell
php artisan reverb:start --debug
```

<a name="restarting"></a>
### Restartowanie

Ponieważ Reverb jest procesem długotrwałym, zmiany w kodzie nie będą odzwierciedlone bez restartu serwera za pomocą komendy Artisan `reverb:restart`.

Komenda `reverb:restart` zapewnia, że wszystkie połączenia zostaną grzecznie zakończone przed zatrzymaniem serwera. Jeśli uruchamiasz Reverb z menedżerem procesów, takim jak Supervisor, serwer zostanie automatycznie ponownie uruchomiony przez menedżer procesów po zakończeniu wszystkich połączeń:

```shell
php artisan reverb:restart
```

<a name="monitoring"></a>
## Monitorowanie

Reverb może być monitorowany za pomocą integracji z [Laravel Pulse](/docs/{{version}}/pulse). Włączając integrację Pulse z Reverb, możesz śledzić liczbę połączeń i wiadomości obsługiwanych przez serwer.

Aby włączyć integrację, najpierw upewnij się, że [zainstalowałeś Pulse](/docs/{{version}}/pulse#installation). Następnie dodaj dowolne z rejestratorów Reverb do pliku konfiguracyjnego `config/pulse.php` aplikacji:

```php
use Laravel\Reverb\Pulse\Recorders\ReverbConnections;
use Laravel\Reverb\Pulse\Recorders\ReverbMessages;

'recorders' => [
    ReverbConnections::class => [
        'sample_rate' => 1,
    ],

    ReverbMessages::class => [
        'sample_rate' => 1,
    ],

    // ...
],
```

Następnie dodaj karty Pulse dla każdego rejestratora do [pulpitu Pulse](/docs/{{version}}/pulse#dashboard-customization):

```blade
<x-pulse>
    <livewire:reverb.connections cols="full" />
    <livewire:reverb.messages cols="full" />
    ...
</x-pulse>
```

Aktywność połączeń jest rejestrowana przez odpytywanie o nowe aktualizacje w sposób okresowy. Aby upewnić się, że te informacje są prawidłowo renderowane na pulpicie Pulse, musisz uruchomić demona `pulse:check` na serwerze Reverb. Jeśli uruchamiasz Reverb w konfiguracji [skalowanej horyzontalnie](#scaling), powinieneś uruchomić ten demon tylko na jednym z serwerów.

<a name="production"></a>
## Uruchamianie Reverb w produkcji

Ze względu na długotrwały charakter serwerów WebSocket, możesz potrzebować wprowadzić pewne optymalizacje na serwerze i środowisku hostingowym, aby upewnić się, że serwer Reverb może skutecznie obsługiwać optymalną liczbę połączeń dla zasobów dostępnych na serwerze.

> [!NOTE]
> [Laravel Cloud](https://cloud.laravel.com) oferuje w pełni zarządzaną infrastrukturę WebSocket zasilaną przez klastry Laravel Reverb, umożliwiając skalowanie i dostarczanie aplikacji obsługujących Reverb bez zarządzania infrastrukturą.

<a name="open-files"></a>
### Otwarte pliki

Każde połączenie WebSocket jest przechowywane w pamięci, dopóki klient lub serwer się nie rozłączy. W środowiskach Unix i podobnych do Unix, każde połączenie jest reprezentowane przez plik. Jednak często istnieją limity liczby dozwolonych otwartych plików zarówno na poziomie systemu operacyjnego, jak i aplikacji.

<a name="operating-system"></a>
#### System operacyjny

W systemie operacyjnym opartym na Unixie możesz określić dowoloną liczbę otwartych plików za pomocą komendy `ulimit`:

```shell
ulimit -n
```

Ta komenda wyświetli limity otwartych plików dozwolone dla różnych użytkowników. Możesz zaktualizować te wartości, edytując plik `/etc/security/limits.conf`. Na przykład, aktualizacja maksymalnej liczby otwartych plików do 10 000 dla użytkownika `forge` wyglądałaby następująco:

```ini
# /etc/security/limits.conf
forge        soft  nofile  10000
forge        hard  nofile  10000
```

<a name="event-loop"></a>
### Pętla zdarzeń

W tle Reverb używa pętli zdarzeń ReactPHP do zarządzania połączeniami WebSocket na serwerze. Domyślnie ta pętla zdarzeń jest zasilana przez `stream_select`, który nie wymaga żadnych dodatkowych rozszerzeń. Jednak `stream_select` jest zazwyczaj ograniczony do 1024 otwartych plików. W związku z tym, jeśli planujesz obsługiwać więcej niż 1000 jednoczesnych połączeń, musisz użyć alternatywnej pętli zdarzeń, która nie jest związana z tymi samymi ograniczeniami.

Reverb automatycznie przełączy się na pętlę zasilaną `ext-uv`, gdy będzie dostępna. To rozszerzenie PHP jest dostępne do instalacji przez PECL:

```shell
pecl install uv
```

<a name="web-server"></a>
### Serwer WWW

W większości przypadków Reverb działa na porcie niepublicznym na serwerze. Aby więc kierować ruch do Reverb, powinieneś skonfigurować reverse proxy. Zakładając, że Reverb działa na hoście `0.0.0.0` i porcie `8080`, a Twój serwer wykorzystuje serwer WWW Nginx, reverse proxy może zostać zdefiniowane dla serwera Reverb za pomocą następującej konfiguracji witryny Nginx:

```nginx
server {
    ...

    location / {
        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";

        proxy_pass http://0.0.0.0:8080;
    }

    ...
}
```

> [!WARNING]
> Reverb nasłuchuje połączeń WebSocket pod `/app` i obsługuje żądania API pod `/apps`. Powinieneś się upewnić, że serwer WWW obsługujący żądania Reverb może obsługiwać oba te URI. Jeśli używasz [Laravel Forge](https://forge.laravel.com) do zarządzania swoimi serwerami, Twój serwer Reverb zostanie prawidłowo skonfigurowany domyślnie.

Zazwyczaj serwery WWW są skonfigurowane tak, aby ograniczać liczbę dozwolonych połączeń w celu zapobiegania przeciążeniu serwera. Aby zwiększyć liczbę dozwolonych połączeń na serwerze WWW Nginx do 10 000, wartości `worker_rlimit_nofile` i `worker_connections` w pliku `nginx.conf` powinny zostać zaktualizowane:

```nginx
user forge;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
worker_rlimit_nofile 10000;

events {
  worker_connections 10000;
  multi_accept on;
}
```

Powyższa konfiguracja pozwoli na utworzenie do 10 000 workerów Nginx na proces. Ponadto ta konfiguracja ustawia limit otwartych plików Nginx na 10 000.

<a name="ports"></a>
### Porty

Systemy operacyjne oparte na Unixie zazwyczaj ograniczają liczbę portów, które mogą być otwarte na serwerze. Możesz zobaczyć bieżący dozwolony zakres za pomocą następującej komendy:

```shell
cat /proc/sys/net/ipv4/ip_local_port_range
# 32768	60999
```

Powyższe wyjście pokazuje, że serwer może obsługiwać maksymalnie 28 231 (60 999 - 32 768) połączeń, ponieważ każde połączenie wymaga wolnego portu. Choć zalecamy [skalowanie horyzontalne](#scaling), aby zwiększyć liczbę dozwolonych połączeń, możesz zwiększyć liczbę dostępnych otwartych portów, aktualizując dozwolony zakres portów w pliku konfiguracyjnym `/etc/sysctl.conf` serwera.

<a name="process-management"></a>
### Zarządzanie procesami

W większości przypadków powinieneś używać menedżera procesów, takiego jak Supervisor, aby zapewnić, że serwer Reverb będzie działać nieprzerwanie. Jeśli używasz Supervisora do uruchamiania Reverb, powinieneś zaktualizować ustawienie `minfds` w pliku `supervisor.conf` serwera, aby upewnić się, że Supervisor jest w stanie otworzyć pliki wymagane do obsługi połączeń do serwera Reverb:

```ini
[supervisord]
...
minfds=10000
```

<a name="scaling"></a>
### Skalowanie

Jeśli musisz obsłużyć więcej połączeń niż jeden serwer może obsłużyć, możesz skalować swój serwer Reverb horyzontalnie. Wykorzystując możliwości publikowania / subskrybowania Redis, Reverb jest w stanie zarządzać połączeniami na wielu serwerach. Gdy wiadomość zostanie odebrana przez jeden z serwerów Reverb aplikacji, serwer użyje Redisa do opublikowania przychodzącej wiadomości na wszystkich innych serwerach.

Aby włączyć skalowanie horyzontalne, powinieneś ustawić zmienną środowiskową `REVERB_SCALING_ENABLED` na `true` w pliku konfiguracyjnym `.env` aplikacji:

```env
REVERB_SCALING_ENABLED=true
```

Następnie powinieneś mieć dedykowany, centralny serwer Redis, z którym będą komunikować się wszystkie serwery Reverb. Reverb użyje [domyślnego połączenia Redis skonfigurowanego dla aplikacji](/docs/{{version}}/redis#configuration) do publikowania wiadomości na wszystkich serwerach Reverb.

Po włączeniu opcji skalowania Reverb i skonfigurowaniu serwera Redis, możesz po prostu wywołać komendę `reverb:start` na wielu serwerach, które mogą komunikować się z Twoim serwerem Redis. Te serwery Reverb powinny zostać umieszczone za load balancerem, który równomiernie rozdziela przychodzące żądania między serwerami.

<a name="events"></a>
## Zdarzenia

Reverb wysyła wewnętrzne zdarzenia podczas cyklu życia połączenia i obsługi wiadomości. Możesz [nasłuchiwać tych zdarzeń](/docs/{{version}}/events), aby wykonywać akcje, gdy połączenia są zarządzane lub wiadomości są wymieniane.

Następujące zdarzenia są wysyłane przez Reverb:

#### `Laravel\Reverb\Events\ChannelCreated`

Wysyłane, gdy kanał zostaje utworzony. Zazwyczaj następuje to, gdy pierwsze połączenie subskrybuje określony kanał. Zdarzenie otrzymuje instancję `Laravel\Reverb\Protocols\Pusher\Channel`.

#### `Laravel\Reverb\Events\ChannelRemoved`

Wysyłane, gdy kanał zostaje usunięty. Zazwyczaj następuje to, gdy ostatnie połączenie wypisuje się z kanału. Zdarzenie otrzymuje instancję `Laravel\Reverb\Protocols\Pusher\Channel`.

#### `Laravel\Reverb\Events\ConnectionPruned`

Wysyłane, gdy nieaktualne połączenie jest usuwane przez serwer. Zdarzenie otrzymuje instancję `Laravel\Reverb\Contracts\Connection`.

#### `Laravel\Reverb\Events\MessageReceived`

Wysyłane, gdy wiadomość zostanie odebrana od połączenia klienta. Zdarzenie otrzymuje instancję `Laravel\Reverb\Contracts\Connection` i surowy ciąg `$message`.

#### `Laravel\Reverb\Events\MessageSent`

Wysyłane, gdy wiadomość zostanie wysłana do połączenia klienta. Zdarzenie otrzymuje instancję `Laravel\Reverb\Contracts\Connection` i surowy ciąg `$message`.
