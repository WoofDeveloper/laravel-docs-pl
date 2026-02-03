# Konfiguracja

- [Wprowadzenie](#introduction)
- [Konfiguracja środowiska](#environment-configuration)
    - [Typy zmiennych środowiskowych](#environment-variable-types)
    - [Pobieranie konfiguracji środowiska](#retrieving-environment-configuration)
    - [Określanie bieżącego środowiska](#determining-the-current-environment)
    - [Szyfrowanie plików środowiska](#encrypting-environment-files)
- [Dostęp do wartości konfiguracji](#accessing-configuration-values)
- [Buforowanie konfiguracji](#configuration-caching)
- [Publikowanie konfiguracji](#configuration-publishing)
- [Tryb debugowania](#debug-mode)
- [Tryb konserwacji](#maintenance-mode)

<a name="introduction"></a>
## Wprowadzenie

Wszystkie pliki konfiguracyjne frameworka Laravel są przechowywane w katalogu `config`. Każda opcja jest udokumentowana, więc możesz swobodnie przeglądać pliki i zapoznać się z dostępnymi opcjami.

Te pliki konfiguracyjne pozwalają na skonfigurowanie takich rzeczy jak informacje o połączeniu z bazą danych, informacje o serwerze pocztowym, a także różne inne podstawowe wartości konfiguracyjne, takie jak adres URL aplikacji i klucz szyfrowania.

<a name="the-about-command"></a>
#### Polecenie `about`

Laravel może wyświetlić przegląd konfiguracji aplikacji, sterowników i środowiska za pomocą polecenia Artisan `about`.

```shell
php artisan about
```

Jeśli interesujesz się tylko określoną sekcją wyjścia przeglądu aplikacji, możesz filtrować tę sekcję za pomocą opcji `--only`:

```shell
php artisan about --only=environment
```

Lub, aby szczegółowo przeanalizować wartości konkretnego pliku konfiguracyjnego, możesz użyć polecenia Artisan `config:show`:

```shell
php artisan config:show database
```

<a name="environment-configuration"></a>
## Konfiguracja środowiska

Często przydatne jest posiadanie różnych wartości konfiguracyjnych w zależności od środowiska, w którym działa aplikacja. Na przykład możesz chcieć używać innego sterownika pamięci podręcznej lokalnie niż na serwerze produkcyjnym.

Aby to ułatwić, Laravel wykorzystuje bibliotekę PHP [DotEnv](https://github.com/vlucas/phpdotenv). W świeżej instalacji Laravela katalog główny aplikacji będzie zawierał plik `.env.example`, który definiuje wiele typowych zmiennych środowiskowych. Podczas procesu instalacji Laravela plik ten zostanie automatycznie skopiowany do `.env`.

Domyślny plik `.env` Laravela zawiera niektóre typowe wartości konfiguracyjne, które mogą się różnić w zależności od tego, czy aplikacja działa lokalnie czy na produkcyjnym serwerze internetowym. Wartości te są następnie odczytywane przez pliki konfiguracyjne w katalogu `config` za pomocą funkcji `env` Laravela.

Jeśli pracujesz w zespole, możesz chcieć kontynuować dołączanie i aktualizowanie pliku `.env.example` wraz z aplikacją. Umieszczając wartości zastępcze w przykładowym pliku konfiguracyjnym, inni deweloperzy w zespole mogą wyraźnie zobaczyć, które zmienne środowiskowe są potrzebne do uruchomienia aplikacji.

> [!NOTE]
> Każda zmienna w pliku `.env` może zostać nadpisana przez zewnętrzne zmienne środowiskowe, takie jak zmienne środowiskowe na poziomie serwera lub systemu.

<a name="environment-file-security"></a>
#### Bezpieczeństwo pliku środowiska

Twój plik `.env` nie powinien być commitowany do systemu kontroli wersji aplikacji, ponieważ każdy deweloper/serwer używający aplikacji może wymagać innej konfiguracji środowiska. Ponadto stanowiłoby to zagrożenie bezpieczeństwa w przypadku, gdyby intruz uzyskał dostęp do repozytorium kontroli wersji, ponieważ wszelkie poufne dane uwierzytelniające zostałyby ujawnione.

Jednakże możliwe jest zaszyfrowanie pliku środowiska za pomocą wbudowanego [szyfrowania środowiska](#encrypting-environment-files) Laravela. Zaszyfrowane pliki środowiska mogą być bezpiecznie umieszczone w kontroli wersji.

<a name="additional-environment-files"></a>
#### Dodatkowe pliki środowiska

Przed załadowaniem zmiennych środowiskowych aplikacji Laravel określa, czy zmienna środowiskowa `APP_ENV` została dostarczona z zewnątrz lub czy określono argument CLI `--env`. Jeśli tak, Laravel spróbuje załadować plik `.env.[APP_ENV]`, jeśli istnieje. Jeśli nie istnieje, zostanie załadowany domyślny plik `.env`.

<a name="environment-variable-types"></a>
### Typy zmiennych środowiskowych

Wszystkie zmienne w plikach `.env` są zazwyczaj parsowane jako ciągi znaków, więc utworzono pewne wartości zarezerwowane, aby umożliwić zwracanie szerszego zakresu typów z funkcji `env()`:

<div class="overflow-auto">

| Wartość `.env` | Wartość `env()` |
| ------------ | ------------- |
| true         | (bool) true   |
| (true)       | (bool) true   |
| false        | (bool) false  |
| (false)      | (bool) false  |
| empty        | (string) ''   |
| (empty)      | (string) ''   |
| null         | (null) null   |
| (null)       | (null) null   |

</div>

Jeśli musisz zdefiniować zmienną środowiskową z wartością zawierającą spacje, możesz to zrobić, otaczając wartość podwójnymi cudzysłowami:

```ini
APP_NAME="My Application"
```

<a name="retrieving-environment-configuration"></a>
### Pobieranie konfiguracji środowiska

Wszystkie zmienne wymienione w pliku `.env` zostaną załadowane do superglobalnej zmiennej PHP `$_ENV`, gdy aplikacja otrzyma żądanie. Możesz jednak użyć funkcji `env`, aby pobrać wartości z tych zmiennych w plikach konfiguracyjnych. W rzeczywistości, jeśli przejrzysz pliki konfiguracyjne Laravela, zauważysz, że wiele opcji już używa tej funkcji:

```php
'debug' => (bool) env('APP_DEBUG', false),
```

Druga wartość przekazana do funkcji `env` to "wartość domyślna". Ta wartość zostanie zwrócona, jeśli nie istnieje żadna zmienna środowiskowa dla danego klucza.

<a name="determining-the-current-environment"></a>
### Określanie bieżącego środowiska

Bieżące środowisko aplikacji jest określane za pomocą zmiennej `APP_ENV` z pliku `.env`. Możesz uzyskać dostęp do tej wartości za pomocą metody `environment` na [fasadzie](/docs/{{version}}/facades) `App`:

```php
use Illuminate\Support\Facades\App;

$environment = App::environment();
```

Możesz również przekazać argumenty do metody `environment`, aby określić, czy środowisko pasuje do danej wartości. Metoda zwróci `true`, jeśli środowisko pasuje do którejkolwiek z podanych wartości:

```php
if (App::environment('local')) {
    // Środowisko to local
}

if (App::environment(['local', 'staging'])) {
    // Środowisko to local LUB staging...
}
```

> [!NOTE]
> Wykrywanie bieżącego środowiska aplikacji może zostać nadpisane przez zdefiniowanie zmiennej środowiskowej `APP_ENV` na poziomie serwera.

<a name="encrypting-environment-files"></a>
### Szyfrowanie plików środowiska

Niezaszyfrowane pliki środowiska nigdy nie powinny być przechowywane w kontroli wersji. Jednak Laravel pozwala na szyfrowanie plików środowiska, tak aby mogły być bezpiecznie dodane do kontroli wersji wraz z resztą aplikacji.

<a name="encryption"></a>
#### Szyfrowanie

Aby zaszyfrować plik środowiska, możesz użyć polecenia `env:encrypt`:

```shell
php artisan env:encrypt
```

Uruchomienie polecenia `env:encrypt` zaszyfruje plik `.env` i umieści zaszyfrowaną zawartość w pliku `.env.encrypted`. Klucz deszyfrowania jest prezentowany w wyniku polecenia i powinien być przechowywany w bezpiecznym menedżerze haseł. Jeśli chcesz dostarczyć własny klucz szyfrowania, możesz użyć opcji `--key` podczas wywoływania polecenia:

```shell
php artisan env:encrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

> [!NOTE]
> Długość dostarczonego klucza powinna odpowiadać długości klucza wymaganej przez używany szyfr szyfrowania. Domyślnie Laravel użyje szyfru `AES-256-CBC`, który wymaga klucza o długości 32 znaków. Możesz swobodnie używać dowolnego szyfru obsługiwanego przez [enkrypter](/docs/{{version}}/encryption) Laravela, przekazując opcję `--cipher` podczas wywoływania polecenia.

Jeśli aplikacja ma wiele plików środowiska, takich jak `.env` i `.env.staging`, możesz określić plik środowiska, który powinien być zaszyfrowany, podając nazwę środowiska za pomocą opcji `--env`:

```shell
php artisan env:encrypt --env=staging
```

<a name="readable-variable-names"></a>
#### Czytelne nazwy zmiennych

Podczas szyfrowania pliku środowiska możesz użyć opcji `--readable`, aby zachować widoczne nazwy zmiennych podczas szyfrowania ich wartości:

```shell
php artisan env:encrypt --readable
```

Spowoduje to utworzenie zaszyfrowanego pliku w następującym formacie:

```ini
APP_NAME=eyJpdiI6...
APP_ENV=eyJpdiI6...
APP_KEY=eyJpdiI6...
APP_DEBUG=eyJpdiI6...
APP_URL=eyJpdiI6...
```

Używanie czytelnego formatu pozwala zobaczyć, które zmienne środowiskowe istnieją bez ujawniania poufnych danych. Ułatwia to również przeglądanie pull requestów, ponieważ możesz zobaczyć, które zmienne zostały dodane, usunięte lub przemianowane bez konieczności deszyfrowania pliku.

Podczas deszyfrowania plików środowiska Laravel automatycznie wykrywa, który format został użyty, więc nie są potrzebne żadne dodatkowe opcje dla polecenia `env:decrypt`.

> [!NOTE]
> Podczas używania opcji `--readable` komentarze i puste linie z oryginalnego pliku środowiska nie są uwzględniane w zaszyfrowanym wyjściu.

<a name="decryption"></a>
#### Deszyfrowanie

Aby odszyfrować plik środowiska, możesz użyć polecenia `env:decrypt`. To polecenie wymaga klucza deszyfrowania, który Laravel pobierze ze zmiennej środowiskowej `LARAVEL_ENV_ENCRYPTION_KEY`:

```shell
php artisan env:decrypt
```

Lub klucz może być dostarczony bezpośrednio do polecenia za pomocą opcji `--key`:

```shell
php artisan env:decrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

Gdy wywoływane jest polecenie `env:decrypt`, Laravel odszyfrowuje zawartość pliku `.env.encrypted` i umieszcza odszyfrowaną zawartość w pliku `.env`.

Opcja `--cipher` może zostać dostarczona do polecenia `env:decrypt`, aby użyć niestandardowego szyfru szyfrowania:

```shell
php artisan env:decrypt --key=qUWuNRdfuImXcKxZ --cipher=AES-128-CBC
```

Jeśli aplikacja ma wiele plików środowiska, takich jak `.env` i `.env.staging`, możesz określić plik środowiska, który powinien być odszyfrowany, podając nazwę środowiska za pomocą opcji `--env`:

```shell
php artisan env:decrypt --env=staging
```

Aby nadpisać istniejący plik środowiska, możesz podać opcję `--force` do polecenia `env:decrypt`:

```shell
php artisan env:decrypt --force
```

<a name="accessing-configuration-values"></a>
## Dostęp do wartości konfiguracji

Możesz łatwo uzyskać dostęp do wartości konfiguracyjnych za pomocą fasady `Config` lub globalnej funkcji `config` z dowolnego miejsca w aplikacji. Dostęp do wartości konfiguracyjnych można uzyskać za pomocą składni "kropkowej", która zawiera nazwę pliku i opcji, do której chcesz uzyskać dostęp. Można również określić wartość domyślną, która zostanie zwrócona, jeśli opcja konfiguracyjna nie istnieje:

```php
use Illuminate\Support\Facades\Config;

$value = Config::get('app.timezone');

$value = config('app.timezone');

// Pobierz wartość domyślną, jeśli wartość konfiguracyjna nie istnieje...
$value = config('app.timezone', 'Asia/Seoul');
```

Aby ustawić wartości konfiguracyjne w czasie wykonywania, możesz wywołać metodę `set` fasady `Config` lub przekazać tablicę do funkcji `config`:

```php
Config::set('app.timezone', 'America/Chicago');

config(['app.timezone' => 'America/Chicago']);
```

Aby pomóc w statycznej analizie, fasada `Config` zapewnia również typowane metody pobierania konfiguracji. Jeśli pobrana wartość konfiguracyjna nie pasuje do oczekiwanego typu, zostanie zgłoszony wyjątek:

```php
Config::string('config-key');
Config::integer('config-key');
Config::float('config-key');
Config::boolean('config-key');
Config::array('config-key');
Config::collection('config-key');
```

<a name="configuration-caching"></a>
## Buforowanie konfiguracji

Aby przyspieszyć działanie aplikacji, powinieneś buforować wszystkie pliki konfiguracyjne w jeden plik za pomocą polecenia Artisan `config:cache`. Spowoduje to połączenie wszystkich opcji konfiguracyjnych aplikacji w jeden plik, który może być szybko załadowany przez framework.

Zazwyczaj powinieneś uruchomić polecenie `php artisan config:cache` jako część procesu wdrażania produkcyjnego. Polecenie nie powinno być uruchamiane podczas lokalnego developmentu, ponieważ opcje konfiguracyjne będą musiały być często zmieniane podczas rozwoju aplikacji.

Gdy konfiguracja zostanie zbuforowana, plik `.env` aplikacji nie będzie ładowany przez framework podczas żądań lub poleceń Artisan; dlatego funkcja `env` zwróci tylko zewnętrzne zmienne środowiskowe na poziomie systemu.

Z tego powodu powinieneś upewnić się, że wywołujesz funkcję `env` tylko z poziomu plików konfiguracyjnych (`config`) aplikacji. Możesz zobaczyć wiele przykładów tego, badając domyślne pliki konfiguracyjne Laravela. Wartości konfiguracyjne mogą być dostępne z dowolnego miejsca w aplikacji za pomocą funkcji `config` [opisanej powyżej](#accessing-configuration-values).

Polecenie `config:clear` może być użyte do wyczyszczenia zbuforowanej konfiguracji:

```shell
php artisan config:clear
```

> [!WARNING]
> Jeśli wykonujesz polecenie `config:cache` podczas procesu wdrażania, powinieneś upewnić się, że wywołujesz funkcję `env` tylko z poziomu plików konfiguracyjnych. Gdy konfiguracja zostanie zbuforowana, plik `.env` nie będzie ładowany; dlatego funkcja `env` zwróci tylko zewnętrzne zmienne środowiskowe na poziomie systemu.

<a name="configuration-publishing"></a>
## Publikowanie konfiguracji

Większość plików konfiguracyjnych Laravela jest już opublikowana w katalogu `config` aplikacji; jednak niektóre pliki konfiguracyjne, takie jak `cors.php` i `view.php`, nie są domyślnie publikowane, ponieważ większość aplikacji nigdy nie będzie musiała ich modyfikować.

Możesz jednak użyć polecenia Artisan `config:publish`, aby opublikować wszystkie pliki konfiguracyjne, które nie są domyślnie publikowane:

```shell
php artisan config:publish

php artisan config:publish --all
```

<a name="debug-mode"></a>
## Tryb debugowania

Opcja `debug` w pliku konfiguracyjnym `config/app.php` określa, ile informacji o błędzie jest faktycznie wyświetlanych użytkownikowi. Domyślnie ta opcja jest ustawiona tak, aby respektować wartość zmiennej środowiskowej `APP_DEBUG`, która jest przechowywana w pliku `.env`.

> [!WARNING]
> Do lokalnego developmentu powinieneś ustawić zmienną środowiskową `APP_DEBUG` na `true`. **W środowisku produkcyjnym ta wartość powinna zawsze wynosić `false`. Jeśli zmienna jest ustawiona na `true` w produkcji, ryzykujesz ujawnienie poufnych wartości konfiguracyjnych użytkownikom końcowym aplikacji.**

<a name="maintenance-mode"></a>
## Tryb konserwacji

Gdy aplikacja jest w trybie konserwacji, niestandardowy widok będzie wyświetlany dla wszystkich żądań do aplikacji. Ułatwia to "wyłączenie" aplikacji podczas jej aktualizacji lub wykonywania konserwacji. Sprawdzanie trybu konserwacji jest zawarte w domyślnym stosie middleware dla aplikacji. Jeśli aplikacja jest w trybie konserwacji, zostanie zgłoszony wyjątek `Symfony\Component\HttpKernel\Exception\HttpException` z kodem statusu 503.

Aby włączyć tryb konserwacji, wykonaj polecenie Artisan `down`:

```shell
php artisan down
```

Jeśli chcesz, aby nagłówek HTTP `Refresh` był wysyłany ze wszystkimi odpowiedziami trybu konserwacji, możesz podać opcję `refresh` podczas wywoływania polecenia `down`. Nagłówek `Refresh` poinstruuje przeglądarkę, aby automatycznie odświeżyła stronę po określonej liczbie sekund:

```shell
php artisan down --refresh=15
```

Możesz również podać opcję `retry` do polecenia `down`, która zostanie ustawiona jako wartość nagłówka HTTP `Retry-After`, chociaż przeglądarki zazwyczaj ignorują ten nagłówek:

```shell
php artisan down --retry=60
```

<a name="bypassing-maintenance-mode"></a>
#### Omijanie trybu konserwacji

Aby umożliwić omijanie trybu konserwacji za pomocą tajnego tokenu, możesz użyć opcji `secret`, aby określić token omijania trybu konserwacji:

```shell
php artisan down --secret="1630542a-246b-4b66-afa1-dd72a4c43515"
```

Po umieszczeniu aplikacji w trybie konserwacji możesz przejść do adresu URL aplikacji pasującego do tego tokenu, a Laravel wyda cookie omijające tryb konserwacji Twojej przeglądarce:

```shell
https://example.com/1630542a-246b-4b66-afa1-dd72a4c43515
```

Jeśli chcesz, aby Laravel wygenerował dla Ciebie tajny token, możesz użyć opcji `with-secret`. Sekret zostanie wyświetlony po przejściu aplikacji w tryb konserwacji:

```shell
php artisan down --with-secret
```

Po uzyskaniu dostępu do tej ukrytej trasy zostaniesz przekierowany do trasy `/` aplikacji. Gdy cookie zostanie wydany przeglądarce, będziesz mógł normalnie przeglądać aplikację tak, jakby nie była w trybie konserwacji.

> [!NOTE]
> Twój tajny tryb konserwacji powinien zazwyczaj składać się ze znaków alfanumerycznych i, opcjonalnie, myślników. Powinieneś unikać używania znaków, które mają specjalne znaczenie w adresach URL, takich jak `?` lub `&`.

<a name="maintenance-mode-on-multiple-servers"></a>
#### Tryb konserwacji na wielu serwerach

Domyślnie Laravel określa, czy aplikacja jest w trybie konserwacji, za pomocą systemu opartego na plikach. Oznacza to, że aby aktywować tryb konserwacji, polecenie `php artisan down` musi zostać wykonane na każdym serwerze hostującym aplikację.

Alternatywnie, Laravel oferuje metodę opartą na pamięci podręcznej do obsługi trybu konserwacji. Ta metoda wymaga uruchomienia polecenia `php artisan down` tylko na jednym serwerze. Aby użyć tego podejścia, zmodyfikuj zmienne trybu konserwacji w pliku `.env` aplikacji. Powinieneś wybrać `store` pamięci podręcznej, który jest dostępny dla wszystkich serwerów. Zapewnia to, że status trybu konserwacji jest konsekwentnie utrzymywany na każdym serwerze:

```ini
APP_MAINTENANCE_DRIVER=cache
APP_MAINTENANCE_STORE=database
```

<a name="pre-rendering-the-maintenance-mode-view"></a>
#### Wstępne renderowanie widoku trybu konserwacji

Jeśli korzystasz z polecenia `php artisan down` podczas wdrażania, użytkownicy mogą nadal czasami napotykać błędy, jeśli uzyskają dostęp do aplikacji podczas aktualizowania zależności Composer lub innych komponentów infrastruktury. Dzieje się tak, ponieważ znaczna część frameworka Laravel musi uruchomić się, aby określić, że aplikacja jest w trybie konserwacji i renderować widok trybu konserwacji za pomocą silnika szablonów.

Z tego powodu Laravel umożliwia wstępne renderowanie widoku trybu konserwacji, który zostanie zwrócony na samym początku cyklu żądania. Ten widok jest renderowany przed załadowaniem jakichkolwiek zależności aplikacji. Możesz wstępnie renderować szablon według własnego wyboru, używając opcji `render` polecenia `down`:

```shell
php artisan down --render="errors::503"
```

<a name="redirecting-maintenance-mode-requests"></a>
#### Przekierowywanie żądań trybu konserwacji

Będąc w trybie konserwacji, Laravel wyświetli widok trybu konserwacji dla wszystkich adresów URL aplikacji, do których użytkownik próbuje uzyskać dostęp. Jeśli chcesz, możesz poinstruować Laravela, aby przekierowywał wszystkie żądania do określonego adresu URL. Można to osiągnąć za pomocą opcji `redirect`. Na przykład możesz chcieć przekierować wszystkie żądania do URI `/`:

```shell
php artisan down --redirect=/
```

<a name="disabling-maintenance-mode"></a>
#### Wyłączanie trybu konserwacji

Aby wyłączyć tryb konserwacji, użyj polecenia `up`:

```shell
php artisan up
```

> [!NOTE]
> Możesz dostosować domyślny szablon trybu konserwacji, definiując własny szablon w `resources/views/errors/503.blade.php`.

<a name="maintenance-mode-queues"></a>
#### Tryb konserwacji i kolejki

Gdy aplikacja jest w trybie konserwacji, żadne [zadania w kolejce](/docs/{{version}}/queues) nie będą obsługiwane. Zadania będą obsługiwane normalnie, gdy aplikacja wyjdzie z trybu konserwacji.

<a name="alternatives-to-maintenance-mode"></a>
#### Alternatywy dla trybu konserwacji

Ponieważ tryb konserwacji wymaga kilku sekund przestoju aplikacji, rozważ uruchamianie aplikacji na w pełni zarządzanej platformie, takiej jak [Laravel Cloud](https://cloud.laravel.com), aby osiągnąć wdrożenie bez przestoju z Laravelem.
