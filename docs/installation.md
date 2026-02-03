# Instalacja

- [Poznaj Laravel](#meet-laravel)
    - [Dlaczego Laravel?](#why-laravel)
- [Tworzenie aplikacji Laravel](#creating-a-laravel-project)
    - [Instalacja PHP i instalatora Laravel](#installing-php)
    - [Tworzenie aplikacji](#creating-an-application)
- [Konfiguracja początkowa](#initial-configuration)
    - [Konfiguracja oparta na środowisku](#environment-based-configuration)
    - [Bazy danych i migracje](#databases-and-migrations)
    - [Konfiguracja katalogów](#directory-configuration)
- [Instalacja przy użyciu Herd](#installation-using-herd)
    - [Herd na macOS](#herd-on-macos)
    - [Herd na Windows](#herd-on-windows)
- [Wsparcie IDE](#ide-support)
- [Laravel i AI](#laravel-and-ai)
    - [Instalacja Laravel Boost](#installing-laravel-boost)
- [Następne kroki](#next-steps)
    - [Laravel jako framework Full Stack](#laravel-the-fullstack-framework)
    - [Laravel jako backend API](#laravel-the-api-backend)

<a name="meet-laravel"></a>
## Poznaj Laravel

Laravel to framework aplikacji webowych o ekspresyjnej, eleganckiej składni. Framework webowy zapewnia strukturę i punkt wyjścia do tworzenia aplikacji, pozwalając Ci skupić się na tworzeniu czegoś niesamowitego, podczas gdy my zajmujemy się szczegółami.

Laravel dąży do zapewnienia niesamowitego doświadczenia programistycznego, oferując jednocześnie potężne funkcje, takie jak dokładna iniekcja zależności, wyrazista warstwa abstrakcji bazy danych, kolejki i zaplanowane zadania, testy jednostkowe i integracyjne oraz wiele więcej.

Niezależnie od tego, czy jesteś nowy w świecie frameworków PHP, czy masz lata doświadczenia, Laravel to framework, który może rosnąć razem z Tobą. Pomożemy Ci postawić pierwsze kroki jako web developer lub damy Ci wsparcie, gdy będziesz rozwijać swoją wiedzę na wyższy poziom. Nie możemy się doczekać, co zbudujesz.

<a name="why-laravel"></a>
### Dlaczego Laravel?

Istnieje wiele narzędzi i frameworków dostępnych podczas tworzenia aplikacji webowej. Jednak wierzymy, że Laravel to najlepszy wybór do budowania nowoczesnych, pełnowymiarowych aplikacji webowych.

#### Progresywny framework

Lubimy nazywać Laravel "progresywnym" frameworkiem. Oznacza to, że Laravel rośnie wraz z Tobą. Jeśli dopiero stawiasz pierwsze kroki w rozwoju aplikacji webowych, ogromna biblioteka dokumentacji, przewodników i [tutoriali wideo](https://laracasts.com) Laravel pomoże Ci nauczyć się podstaw bez przytłaczania.

Jeśli jesteś doświadczonym programistą, Laravel daje Ci solidne narzędzia do [iniekcji zależności](/docs/{{version}}/container), [testów jednostkowych](/docs/{{version}}/testing), [kolejek](/docs/{{version}}/queues), [zdarzeń w czasie rzeczywistym](/docs/{{version}}/broadcasting) i więcej. Laravel jest dopracowany do budowania profesjonalnych aplikacji webowych i gotowy do obsługi obciążeń korporacyjnych.

#### Skalowalny framework

Laravel jest niezwykle skalowalny. Dzięki przyjaznemu do skalowania charakterowi PHP oraz wbudowanemu wsparciu Laravel dla szybkich, rozproszonych systemów cache, takich jak Redis, skalowanie horyzontalne z Laravel to pestka. Faktycznie, aplikacje Laravel zostały łatwo przeskalowane do obsługi setek milionów zapytań miesięcznie.

Potrzebujesz ekstremalnego skalowania? Platformy takie jak [Laravel Cloud](https://cloud.laravel.com) pozwalają uruchamiać aplikację Laravel na niemal nieograniczoną skalę.

#### Framework gotowy na agentów AI

Opiniowane konwencje Laravel i dobrze zdefiniowana struktura czynią go idealnym frameworkiem do [rozwoju wspomaganego przez AI](/docs/{{version}}/ai) przy użyciu narzędzi takich jak Cursor i Claude Code. Kiedy poprosisz agenta AI o dodanie kontrolera, wie dokładnie, gdzie go umieścić. Kiedy potrzebujesz nowej migracji, konwencje nazewnictwa i lokalizacje plików są przewidywalne. Ta spójność eliminuje zgadywanie, które często sprawia problemy narzędziom AI w bardziej elastycznych frameworkach.

Poza organizacją plików, wyrazista składnia Laravel i kompleksowa dokumentacja dają agentom AI kontekst potrzebny do generowania dokładnego, idiomatycznego kodu. Funkcje takie jak relacje Eloquent, żądania formularzy i middleware podążają za wzorcami, które agenci mogą niezawodnie rozumieć i replikować. Rezultatem jest kod generowany przez AI, który wygląda, jakby został napisany przez doświadczonego programistę Laravel, a nie sklejony z ogólnych fragmentów PHP.

Aby dowiedzieć się więcej o tym, dlaczego Laravel jest idealnym wyborem do rozwoju wspomaganego przez AI, sprawdź naszą dokumentację na temat [rozwoju agentowego](/docs/{{version}}/ai).

#### Framework społecznościowy

Laravel łączy najlepsze pakiety w ekosystemie PHP, aby zaoferować najbardziej solidny i przyjazny dla programistów framework. Ponadto tysiące utalentowanych programistów z całego świata [przyczyniły się do rozwoju frameworka](https://github.com/laravel/framework). Kto wie, może nawet staniesz się współtwórcą Laravel.

<a name="creating-a-laravel-project"></a>
## Tworzenie aplikacji Laravel

<a name="installing-php"></a>
### Instalacja PHP i instalatora Laravel

Przed utworzeniem pierwszej aplikacji Laravel upewnij się, że na Twoim lokalnym komputerze zainstalowane są [PHP](https://php.net), [Composer](https://getcomposer.org) oraz [instalator Laravel](https://github.com/laravel/installer). Ponadto powinieneś zainstalować [Node i NPM](https://nodejs.org) lub [Bun](https://bun.sh/), aby móc kompilować zasoby frontendowe swojej aplikacji.

Jeśli nie masz zainstalowanego PHP i Composera na swoim lokalnym komputerze, następujące polecenia zainstalują PHP, Composer i instalator Laravel na macOS, Windows lub Linux:

```shell tab=macOS
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"
```

```shell tab=Windows PowerShell
# Run as administrator...
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.4'))
```

```shell tab=Linux
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
```

Po uruchomieniu jednego z powyższych poleceń powinieneś ponownie uruchomić sesję terminala. Aby zaktualizować PHP, Composer i instalator Laravel po zainstalowaniu ich przez `php.new`, możesz ponownie uruchomić polecenie w swoim terminalu.

Jeśli masz już zainstalowane PHP i Composer, możesz zainstalować instalator Laravel przez Composer:

```shell
composer global require laravel/installer
```

> [!NOTE]
> Aby uzyskać w pełni funkcjonalne, graficzne doświadczenie instalacji i zarządzania PHP, sprawdź [Laravel Herd](#installation-using-herd).

<a name="creating-an-application"></a>
### Tworzenie aplikacji

Po zainstalowaniu PHP, Composera i instalatora Laravel jesteś gotowy do utworzenia nowej aplikacji Laravel. Instalator Laravel poprosi Cię o wybranie preferowanego frameworka testowego, bazy danych i zestawu startowego:

```shell
laravel new example-app
```

Po utworzeniu aplikacji możesz uruchomić lokalny serwer deweloperski Laravel, worker kolejki i serwer deweloperski Vite używając skryptu Composera `dev`:

```shell
cd example-app
npm install && npm run build
composer run dev
```

Po uruchomieniu serwera deweloperskiego Twoja aplikacja będzie dostępna w przeglądarce pod adresem [http://localhost:8000](http://localhost:8000). Następnie jesteś gotowy, aby [rozpocząć kolejne kroki w ekosystemie Laravel](#next-steps). Oczywiście możesz również [skonfigurować bazę danych](#databases-and-migrations).

> [!NOTE]
> Jeśli chciałbyś mieć dobry start podczas tworzenia aplikacji Laravel, rozważ użycie jednego z naszych [zestawów startowych](/docs/{{version}}/starter-kits). Zestawy startowe Laravel zapewniają szkielet uwierzytelniania backendu i frontendu dla Twojej nowej aplikacji Laravel.

<a name="initial-configuration"></a>
## Konfiguracja początkowa

Wszystkie pliki konfiguracyjne dla frameworka Laravel są przechowywane w katalogu `config`. Każda opcja jest udokumentowana, więc śmiało przeglądaj pliki i zapoznawaj się z dostępnymi opcjami.

Laravel nie wymaga prawie żadnej dodatkowej konfiguracji od razu po instalacji. Możesz swobodnie zacząć programować! Jednak możesz chcieć przejrzeć plik `config/app.php` i jego dokumentację. Zawiera on kilka opcji, takich jak `url` i `locale`, które możesz chcieć zmienić zgodnie ze swoją aplikacją.

<a name="environment-based-configuration"></a>
### Konfiguracja oparta na środowisku

Ponieważ wiele wartości opcji konfiguracyjnych Laravel może się różnić w zależności od tego, czy aplikacja działa na lokalnym komputerze czy na produkcyjnym serwerze webowym, wiele ważnych wartości konfiguracyjnych jest definiowanych przy użyciu pliku `.env`, który znajduje się w katalogu głównym aplikacji.

Twój plik `.env` nie powinien być zatwierdzany do kontroli źródła aplikacji, ponieważ każdy programista / serwer używający Twojej aplikacji może wymagać innej konfiguracji środowiska. Ponadto byłoby to zagrożeniem bezpieczeństwa w przypadku, gdyby intruz uzyskał dostęp do repozytorium kontroli źródła, ponieważ wszelkie wrażliwe dane uwierzytelniające zostałyby ujawnione.

> [!NOTE]
> Aby uzyskać więcej informacji o pliku `.env` i konfiguracji opartej na środowisku, sprawdź pełną [dokumentację konfiguracji](/docs/{{version}}/configuration#environment-configuration).

<a name="databases-and-migrations"></a>
### Bazy danych i migracje

Teraz, gdy utworzyłeś swoją aplikację Laravel, prawdopodobnie chcesz przechowywać pewne dane w bazie danych. Domyślnie plik konfiguracyjny `.env` Twojej aplikacji określa, że Laravel będzie współdziałać z bazą danych SQLite.

Podczas tworzenia aplikacji Laravel utworzył plik `database/database.sqlite` i uruchomił niezbędne migracje w celu utworzenia tabel bazy danych aplikacji.

Jeśli wolisz użyć innego sterownika bazy danych, takiego jak MySQL lub PostgreSQL, możesz zaktualizować swój plik konfiguracyjny `.env`, aby użyć odpowiedniej bazy danych. Na przykład, jeśli chcesz użyć MySQL, zaktualizuj zmienne `DB_*` w pliku konfiguracyjnym `.env` w następujący sposób:

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

Jeśli wybierzesz bazę danych inną niż SQLite, musisz utworzyć bazę danych i uruchomić [migracje bazy danych](/docs/{{version}}/migrations) swojej aplikacji:

```shell
php artisan migrate
```

> [!NOTE]
> Jeśli rozwijasz aplikację na macOS lub Windows i musisz zainstalować MySQL, PostgreSQL lub Redis lokalnie, rozważ użycie [Herd Pro](https://herd.laravel.com/#plans) lub [DBngin](https://dbngin.com/).

<a name="directory-configuration"></a>
### Konfiguracja katalogów

Laravel powinien zawsze być serwowany z katalogu głównego "katalogu webowego" skonfigurowanego dla Twojego serwera webowego. Nie powinieneś próbować serwować aplikacji Laravel z podkatalogu "katalogu webowego". Próba tego może ujawnić wrażliwe pliki obecne w Twojej aplikacji.

<a name="installation-using-herd"></a>
## Instalacja przy użyciu Herd

[Laravel Herd](https://herd.laravel.com) to błyskawicznie szybkie, natywne środowisko programistyczne Laravel i PHP dla macOS i Windows. Herd zawiera wszystko, czego potrzebujesz, aby zacząć programować w Laravel, w tym PHP i Nginx.

Po zainstalowaniu Herd jesteś gotowy, aby zacząć programować w Laravel. Herd zawiera narzędzia wiersza poleceń dla `php`, `composer`, `laravel`, `expose`, `node`, `npm` i `nvm`.

> [!NOTE]
> [Herd Pro](https://herd.laravel.com/#plans) rozszerza Herd o dodatkowe potężne funkcje, takie jak możliwość tworzenia i zarządzania lokalnymi bazami danych MySQL, Postgres i Redis, a także przeglądanie lokalnej poczty i monitorowanie logów.

<a name="herd-on-macos"></a>
### Herd na macOS

Jeśli rozwijasz aplikacje na macOS, możesz pobrać instalator Herd ze [strony Herd](https://herd.laravel.com). Instalator automatycznie pobiera najnowszą wersję PHP i konfiguruje Twojego Maca, aby zawsze uruchamiał [Nginx](https://www.nginx.com/) w tle.

Herd dla macOS używa [dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq) do obsługi "zaparkowanych" katalogów. Każda aplikacja Laravel w zaparkowanym katalogu będzie automatycznie serwowana przez Herd. Domyślnie Herd tworzy zaparkowany katalog w `~/Herd` i możesz uzyskać dostęp do dowolnej aplikacji Laravel w tym katalogu na domenie `.test` używając nazwy katalogu.

Po zainstalowaniu Herd najszybszym sposobem utworzenia nowej aplikacji Laravel jest użycie CLI Laravel, które jest dołączone do Herd:

```shell
cd ~/Herd
laravel new my-app
cd my-app
herd open
```

Oczywiście zawsze możesz zarządzać swoimi zaparkowanymi katalogami i innymi ustawieniami PHP za pośrednictwem interfejsu użytkownika Herd, który można otworzyć z menu Herd w zasobniku systemowym.

Możesz dowiedzieć się więcej o Herd, sprawdzając [dokumentację Herd](https://herd.laravel.com/docs).

<a name="herd-on-windows"></a>
### Herd na Windows

Możesz pobrać instalator Windows dla Herd ze [strony Herd](https://herd.laravel.com/windows). Po zakończeniu instalacji możesz uruchomić Herd, aby zakończyć proces wprowadzania i uzyskać dostęp do interfejsu użytkownika Herd po raz pierwszy.

Interfejs użytkownika Herd jest dostępny po kliknięciu lewym przyciskiem myszy na ikonę Herd w zasobniku systemowym. Kliknięcie prawym przyciskiem otwiera szybkie menu z dostępem do wszystkich narzędzi, których potrzebujesz na co dzień.

Podczas instalacji Herd tworzy "zaparkowany" katalog w Twoim katalogu domowym pod adresem `%USERPROFILE%\Herd`. Każda aplikacja Laravel w zaparkowanym katalogu będzie automatycznie serwowana przez Herd i możesz uzyskać dostęp do dowolnej aplikacji Laravel w tym katalogu na domenie `.test` używając nazwy katalogu.

Po zainstalowaniu Herd najszybszym sposobem utworzenia nowej aplikacji Laravel jest użycie CLI Laravel, które jest dołączone do Herd. Aby zacząć, otwórz Powershell i uruchom następujące polecenia:

```shell
cd ~\Herd
laravel new my-app
cd my-app
herd open
```

Możesz dowiedzieć się więcej o Herd, sprawdzając [dokumentację Herd dla Windows](https://herd.laravel.com/docs/windows).

<a name="ide-support"></a>
## Wsparcie IDE

Możesz swobodnie używać dowolnego edytora kodu podczas tworzenia aplikacji Laravel. Jeśli szukasz lekkich i rozszerzalnych edytorów, [VS Code](https://code.visualstudio.com) lub [Cursor](https://cursor.com) w połączeniu z oficjalnym [rozszerzeniem Laravel VS Code](https://marketplace.visualstudio.com/items?itemName=laravel.vscode-laravel) oferują doskonałe wsparcie dla Laravel z funkcjami takimi jak podświetlanie składni, snippety, integracja poleceń artisan oraz inteligentne autouzupełnianie dla modeli Eloquent, tras, middleware, zasobów, konfiguracji i Inertia.js.

Aby uzyskać rozległe i solidne wsparcie dla Laravel, sprawdź [PhpStorm](https://www.jetbrains.com/phpstorm/laravel/?utm_source=laravel.com&utm_medium=link&utm_campaign=laravel-2025&utm_content=partner&ref=laravel-2025), IDE od JetBrains. Wbudowane wsparcie frameworka Laravel w PhpStorm obejmuje szablony Blade, inteligentne autouzupełnianie dla modeli Eloquent, tras, widoków, tłumaczeń i komponentów, wraz z potężnym generowaniem kodu i nawigacją po projektach Laravel.

Dla tych, którzy szukają doświadczenia programistycznego w chmurze, [Firebase Studio](https://firebase.studio/) zapewnia natychmiastowy dostęp do budowania z Laravel bezpośrednio w przeglądarce. Bez wymaganej konfiguracji Firebase Studio ułatwia rozpoczęcie budowania aplikacji Laravel z dowolnego urządzenia.

<a name="laravel-and-ai"></a>
## Laravel i AI

[Laravel Boost](https://github.com/laravel/boost) to potężne narzędzie, które wypełnia lukę między agentami kodującymi AI a aplikacjami Laravel. Boost zapewnia agentom AI kontekst specyficzny dla Laravel, narzędzia i wytyczne, dzięki czemu mogą generować bardziej dokładny kod specyficzny dla wersji, który podąża za konwencjami Laravel.

Gdy zainstalujesz Boost w swojej aplikacji Laravel, agenci AI uzyskują dostęp do ponad 15 specjalistycznych narzędzi, w tym możliwość sprawdzenia, które pakiety używasz, zapytania do bazy danych, wyszukiwania w dokumentacji Laravel, odczytu logów przeglądarki, generowania testów i wykonywania kodu za pośrednictwem Tinker.

Ponadto Boost daje agentom AI dostęp do ponad 17 000 wektoryzowanych fragmentów dokumentacji ekosystemu Laravel, specyficznych dla zainstalowanych wersji pakietów. Oznacza to, że agenci mogą dostarczać wskazówki dostosowane do dokładnych wersji używanych w Twoim projekcie.

Boost zawiera również wytyczne AI utrzymywane przez Laravel, które pomagają agentom podążać za konwencjami frameworka, pisać odpowiednie testy i unikać typowych pułapek podczas generowania kodu Laravel.

<a name="installing-laravel-boost"></a>
### Instalacja Laravel Boost

Boost można zainstalować w aplikacjach Laravel 10, 11 i 12 działających na PHP 8.1 lub wyższym. Aby rozpocząć, zainstaluj Boost jako zależność deweloperską:

```shell
composer require laravel/boost --dev
```

Po zainstalowaniu uruchom interaktywny instalator:

```shell
php artisan boost:install
```

Instalator automatycznie wykryje Twoje IDE i agentów AI, pozwalając Ci wybrać funkcje, które mają sens dla Twojego projektu. Boost szanuje istniejące konwencje projektu i domyślnie nie wymusza opiniotwórczych reguł stylu.

> [!NOTE]
> Aby dowiedzieć się więcej o Boost, sprawdź [repozytorium Laravel Boost na GitHub](https://github.com/laravel/boost).

<a name="adding-custom-ai-guidelines"></a>
#### Dodawanie niestandardowych wytycznych AI

Aby rozszerzyć Laravel Boost o własne niestandardowe wytyczne AI, dodaj pliki `.blade.php` lub `.md` do katalogu `.ai/guidelines/*` w Twojej aplikacji. Te pliki zostaną automatycznie uwzględnione wraz z wytycznymi Laravel Boost, gdy uruchomisz `boost:install`.

<a name="next-steps"></a>
## Następne kroki

Teraz, gdy utworzyłeś swoją aplikację Laravel, możesz się zastanawiać, czego nauczyć się dalej. Po pierwsze, zdecydowanie zalecamy zapoznanie się z tym, jak działa Laravel, czytając następującą dokumentację:

<div class="content-list" markdown="1">

- [Cykl życia żądania](/docs/{{version}}/lifecycle)
- [Konfiguracja](/docs/{{version}}/configuration)
- [Struktura katalogów](/docs/{{version}}/structure)
- [Frontend](/docs/{{version}}/frontend)
- [Kontener usług](/docs/{{version}}/container)
- [Fasady](/docs/{{version}}/facades)

</div>

Sposób, w jaki chcesz używać Laravel, będzie również determinował kolejne kroki w Twojej podróży. Istnieje wiele sposobów używania Laravel i poniżej zbadamy dwa główne przypadki użycia frameworka.

<a name="laravel-the-fullstack-framework"></a>
### Laravel jako framework Full Stack

Laravel może służyć jako framework full stack. Przez framework "full stack" mamy na myśli, że będziesz używać Laravel do kierowania żądań do swojej aplikacji i renderowania frontendu za pomocą [szablonów Blade](/docs/{{version}}/blade) lub technologii hybrydowej jednostronicowej aplikacji, takiej jak [Inertia](https://inertiajs.com). To najczęstszy sposób używania frameworka Laravel i, naszym zdaniem, najbardziej produktywny sposób używania Laravel.

Jeśli tak planujesz używać Laravel, możesz sprawdzić naszą dokumentację na temat [rozwoju frontendu](/docs/{{version}}/frontend), [routingu](/docs/{{version}}/routing), [widoków](/docs/{{version}}/views) lub [ORM Eloquent](/docs/{{version}}/eloquent). Ponadto możesz być zainteresowany poznaniem pakietów społecznościowych, takich jak [Livewire](https://livewire.laravel.com) i [Inertia](https://inertiajs.com). Te pakiety pozwalają używać Laravel jako frameworka full-stack, jednocześnie ciesząc się wieloma korzyściami UI oferowanymi przez jednostronicowe aplikacje JavaScript.

Jeśli używasz Laravel jako frameworka full stack, zdecydowanie zachęcamy również do nauczenia się, jak kompilować CSS i JavaScript aplikacji za pomocą [Vite](/docs/{{version}}/vite).

> [!NOTE]
> Jeśli chcesz mieć dobry start w budowaniu swojej aplikacji, sprawdź jeden z naszych oficjalnych [zestawów startowych aplikacji](/docs/{{version}}/starter-kits).

<a name="laravel-the-api-backend"></a>
### Laravel jako backend API

Laravel może również służyć jako backend API do jednostronicowej aplikacji JavaScript lub aplikacji mobilnej. Na przykład możesz używać Laravel jako backendu API dla swojej aplikacji [Next.js](https://nextjs.org). W tym kontekście możesz używać Laravel do zapewnienia [uwierzytelniania](/docs/{{version}}/sanctum) oraz przechowywania / pobierania danych dla swojej aplikacji, jednocześnie korzystając z potężnych usług Laravel, takich jak kolejki, e-maile, powiadomienia i więcej.

Jeśli tak planujesz używać Laravel, możesz sprawdzić naszą dokumentację na temat [routingu](/docs/{{version}}/routing), [Laravel Sanctum](/docs/{{version}}/sanctum) i [ORM Eloquent](/docs/{{version}}/eloquent).
