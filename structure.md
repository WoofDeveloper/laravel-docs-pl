# Struktura Katalogów

- [Wprowadzenie](#introduction)
- [Katalog Główny](#the-root-directory)
    - [Katalog `app`](#the-root-app-directory)
    - [Katalog `bootstrap`](#the-bootstrap-directory)
    - [Katalog `config`](#the-config-directory)
    - [Katalog `database`](#the-database-directory)
    - [Katalog `public`](#the-public-directory)
    - [Katalog `resources`](#the-resources-directory)
    - [Katalog `routes`](#the-routes-directory)
    - [Katalog `storage`](#the-storage-directory)
    - [Katalog `tests`](#the-tests-directory)
    - [Katalog `vendor`](#the-vendor-directory)
- [Katalog App](#the-app-directory)
    - [Katalog `Broadcasting`](#the-broadcasting-directory)
    - [Katalog `Console`](#the-console-directory)
    - [Katalog `Events`](#the-events-directory)
    - [Katalog `Exceptions`](#the-exceptions-directory)
    - [Katalog `Http`](#the-http-directory)
    - [Katalog `Jobs`](#the-jobs-directory)
    - [Katalog `Listeners`](#the-listeners-directory)
    - [Katalog `Mail`](#the-mail-directory)
    - [Katalog `Models`](#the-models-directory)
    - [Katalog `Notifications`](#the-notifications-directory)
    - [Katalog `Policies`](#the-policies-directory)
    - [Katalog `Providers`](#the-providers-directory)
    - [Katalog `Rules`](#the-rules-directory)

<a name="introduction"></a>
## Wprowadzenie

Domyślna struktura aplikacji Laravel ma na celu zapewnienie świetnego punktu wyjścia zarówno dla dużych, jak i małych aplikacji. Możesz jednak dowolnie organizować swoją aplikację. Laravel nie nakłada prawie żadnych ograniczeń co do lokalizacji danej klasy - o ile Composer może ją automatycznie załadować.

<a name="the-root-directory"></a>
## Katalog Główny

<a name="the-root-app-directory"></a>
### Katalog App

Katalog `app` zawiera podstawowy kod Twojej aplikacji. Wkrótce szczegółowo omówimy ten katalog; jednak prawie wszystkie klasy w Twojej aplikacji będą znajdować się w tym katalogu.

<a name="the-bootstrap-directory"></a>
### Katalog Bootstrap

Katalog `bootstrap` zawiera plik `app.php`, który inicjalizuje framework. Ten katalog zawiera również katalog `cache`, który przechowuje pliki generowane przez framework w celu optymalizacji wydajności, takie jak pliki cache tras i usług.

<a name="the-config-directory"></a>
### Katalog Config

Katalog `config`, jak sama nazwa wskazuje, zawiera wszystkie pliki konfiguracyjne Twojej aplikacji. To dobry pomysł, aby przeczytać wszystkie te pliki i zapoznać się ze wszystkimi dostępnymi opcjami.

<a name="the-database-directory"></a>
### Katalog Database

Katalog `database` zawiera migracje bazy danych, fabryki modeli i seedy. Jeśli chcesz, możesz również użyć tego katalogu do przechowywania bazy danych SQLite.

<a name="the-public-directory"></a>
### Katalog Public

Katalog `public` zawiera plik `index.php`, który jest punktem wejścia dla wszystkich żądań wchodzących do Twojej aplikacji i konfiguruje automatyczne ładowanie. Ten katalog zawiera również Twoje zasoby, takie jak obrazy, JavaScript i CSS.

<a name="the-resources-directory"></a>
### Katalog Resources

Katalog `resources` zawiera Twoje [widoki](/docs/{{version}}/views), a także nieprzetworzone, nieskompilowane zasoby, takie jak CSS lub JavaScript.

<a name="the-routes-directory"></a>
### Katalog Routes

Katalog `routes` zawiera wszystkie definicje tras dla Twojej aplikacji. Domyślnie Laravel zawiera dwa pliki tras: `web.php` i `console.php`.

Plik `web.php` zawiera trasy, które Laravel umieszcza w grupie middleware `web`, która zapewnia stan sesji, ochronę CSRF i szyfrowanie plików cookie. Jeśli Twoja aplikacja nie oferuje bezstanowego, RESTful API, to prawdopodobnie wszystkie Twoje trasy będą zdefiniowane w pliku `web.php`.

Plik `console.php` to miejsce, w którym możesz zdefiniować wszystkie swoje polecenia konsolowe oparte na zamknięciach. Każde zamknięcie jest powiązane z instancją polecenia, co umożliwia prosty dostęp do metod IO każdego polecenia. Chociaż ten plik nie definiuje tras HTTP, definiuje punkty wejścia (trasy) do Twojej aplikacji oparte na konsoli. Możesz również [planować](/docs/{{version}}/scheduling) zadania w pliku `console.php`.

Opcjonalnie możesz zainstalować dodatkowe pliki tras dla tras API (`api.php`) i kanałów broadcasting (`channels.php`), za pomocą poleceń Artisan `install:api` i `install:broadcasting`.

Plik `api.php` zawiera trasy, które mają być bezstanowe, więc żądania wchodzące do aplikacji przez te trasy mają być uwierzytelniane [za pomocą tokenów](/docs/{{version}}/sanctum) i nie będą miały dostępu do stanu sesji.

Plik `channels.php` to miejsce, w którym możesz zarejestrować wszystkie kanały [rozgłaszania zdarzeń](/docs/{{version}}/broadcasting), które obsługuje Twoja aplikacja.

<a name="the-storage-directory"></a>
### Katalog Storage

Katalog `storage` zawiera Twoje logi, skompilowane szablony Blade, sesje oparte na plikach, pliki cache i inne pliki generowane przez framework. Ten katalog jest podzielony na katalogi `app`, `framework` i `logs`. Katalog `app` może być używany do przechowywania wszelkich plików generowanych przez Twoją aplikację. Katalog `framework` służy do przechowywania plików i pamięci podręcznych generowanych przez framework. Wreszcie, katalog `logs` zawiera pliki logów Twojej aplikacji.

Katalog `storage/app/public` może być używany do przechowywania plików generowanych przez użytkowników, takich jak awatary profilowe, które powinny być publicznie dostępne. Powinieneś utworzyć link symboliczny w `public/storage`, który wskazuje na ten katalog. Możesz utworzyć link za pomocą polecenia Artisan `php artisan storage:link`.

<a name="the-tests-directory"></a>
### Katalog Tests

Katalog `tests` zawiera Twoje testy automatyczne. Przykładowe testy jednostkowe i funkcjonalne [Pest](https://pestphp.com) lub [PHPUnit](https://phpunit.de/) są dostarczone od razu. Każda klasa testowa powinna mieć przyrostek `Test`. Możesz uruchomić testy za pomocą poleceń `/vendor/bin/pest` lub `/vendor/bin/phpunit`. Lub, jeśli chcesz uzyskać bardziej szczegółową i elegancką reprezentację wyników testów, możesz uruchomić testy za pomocą polecenia Artisan `php artisan test`.

<a name="the-vendor-directory"></a>
### Katalog Vendor

Katalog `vendor` zawiera Twoje zależności [Composer](https://getcomposer.org).

<a name="the-app-directory"></a>
## Katalog App

Większość Twojej aplikacji znajduje się w katalogu `app`. Domyślnie ten katalog jest umieszczony w przestrzeni nazw `App` i jest automatycznie ładowany przez Composer przy użyciu [standardu autoładowania PSR-4](https://www.php-fig.org/psr/psr-4/).

Domyślnie katalog `app` zawiera katalogi `Http`, `Models` i `Providers`. Jednak z czasem wiele innych katalogów zostanie wygenerowanych wewnątrz katalogu app, gdy będziesz używać poleceń make Artisan do generowania klas. Na przykład, katalog `app/Console` nie będzie istniał, dopóki nie wykonasz polecenia Artisan `make:command` w celu wygenerowania klasy polecenia.

Zarówno katalog `Console`, jak i `Http` są szczegółowo wyjaśnione w odpowiednich sekcjach poniżej, ale pomyśl o katalogach `Console` i `Http` jako o zapewnianiu API do rdzenia Twojej aplikacji. Protokół HTTP i CLI są mechanizmami do interakcji z Twoją aplikacją, ale nie zawierają logiki aplikacji. Innymi słowy, są to dwa sposoby wydawania poleceń Twojej aplikacji. Katalog `Console` zawiera wszystkie Twoje polecenia Artisan, podczas gdy katalog `Http` zawiera Twoje kontrolery, middleware i żądania.

> [!NOTE]
> Wiele klas w katalogu `app` może być wygenerowanych przez Artisan za pomocą poleceń. Aby przejrzeć dostępne polecenia, uruchom polecenie `php artisan list make` w terminalu.

<a name="the-broadcasting-directory"></a>
### Katalog Broadcasting

Katalog `Broadcasting` zawiera wszystkie klasy kanałów rozgłaszania dla Twojej aplikacji. Te klasy są generowane za pomocą polecenia `make:channel`. Ten katalog nie istnieje domyślnie, ale zostanie utworzony dla Ciebie, gdy utworzysz swój pierwszy kanał. Aby dowiedzieć się więcej o kanałach, sprawdź dokumentację dotyczącą [rozgłaszania zdarzeń](/docs/{{version}}/broadcasting).

<a name="the-console-directory"></a>
### Katalog Console

Katalog `Console` zawiera wszystkie niestandardowe polecenia Artisan dla Twojej aplikacji. Te polecenia mogą być generowane za pomocą polecenia `make:command`.

<a name="the-events-directory"></a>
### Katalog Events

Ten katalog nie istnieje domyślnie, ale zostanie utworzony dla Ciebie przez polecenia Artisan `event:generate` i `make:event`. Katalog `Events` zawiera [klasy zdarzeń](/docs/{{version}}/events). Zdarzenia mogą być używane do powiadamiania innych części Twojej aplikacji, że nastąpiła dana akcja, zapewniając dużą elastyczność i luźne powiązanie.

<a name="the-exceptions-directory"></a>
### Katalog Exceptions

Katalog `Exceptions` zawiera wszystkie niestandardowe wyjątki dla Twojej aplikacji. Te wyjątki mogą być generowane za pomocą polecenia `make:exception`.

<a name="the-http-directory"></a>
### Katalog Http

Katalog `Http` zawiera Twoje kontrolery, middleware i żądania formularzy. Prawie cała logika do obsługi żądań wchodzących do Twojej aplikacji będzie umieszczona w tym katalogu.

<a name="the-jobs-directory"></a>
### Katalog Jobs

Ten katalog nie istnieje domyślnie, ale zostanie utworzony dla Ciebie, jeśli wykonasz polecenie Artisan `make:job`. Katalog `Jobs` zawiera [zadania możliwe do kolejkowania](/docs/{{version}}/queues) dla Twojej aplikacji. Zadania mogą być umieszczane w kolejce przez Twoją aplikację lub uruchamiane synchronicznie w ramach bieżącego cyklu życia żądania. Zadania uruchamiane synchronicznie podczas bieżącego żądania są czasami nazywane "poleceniami", ponieważ są implementacją [wzorca polecenia](https://en.wikipedia.org/wiki/Command_pattern).

<a name="the-listeners-directory"></a>
### Katalog Listeners

Ten katalog nie istnieje domyślnie, ale zostanie utworzony dla Ciebie, jeśli wykonasz polecenia Artisan `event:generate` lub `make:listener`. Katalog `Listeners` zawiera klasy, które obsługują Twoje [zdarzenia](/docs/{{version}}/events). Nasłuchiwacze zdarzeń otrzymują instancję zdarzenia i wykonują logikę w odpowiedzi na wyzwolenie zdarzenia. Na przykład, zdarzenie `UserRegistered` może być obsługiwane przez nasłuchiwacz `SendWelcomeEmail`.

<a name="the-mail-directory"></a>
### Katalog Mail

Ten katalog nie istnieje domyślnie, ale zostanie utworzony dla Ciebie, jeśli wykonasz polecenie Artisan `make:mail`. Katalog `Mail` zawiera wszystkie Twoje [klasy reprezentujące e-maile](/docs/{{version}}/mail) wysyłane przez Twoją aplikację. Obiekty Mail pozwalają na enkapsulację całej logiki budowania wiadomości e-mail w jednej, prostej klasie, która może być wysłana za pomocą metody `Mail::send`.

<a name="the-models-directory"></a>
### Katalog Models

Katalog `Models` zawiera wszystkie Twoje [klasy modeli Eloquent](/docs/{{version}}/eloquent). ORM Eloquent dołączony do Laravel zapewnia piękną, prostą implementację ActiveRecord do pracy z Twoją bazą danych. Każda tabela bazy danych ma odpowiadający jej "Model", który służy do interakcji z tą tabelą. Modele pozwalają na wykonywanie zapytań o dane w Twoich tabelach, a także wstawianie nowych rekordów do tabeli.

<a name="the-notifications-directory"></a>
### Katalog Notifications

Ten katalog nie istnieje domyślnie, ale zostanie utworzony dla Ciebie, jeśli wykonasz polecenie Artisan `make:notification`. Katalog `Notifications` zawiera wszystkie "transakcyjne" [powiadomienia](/docs/{{version}}/notifications), które są wysyłane przez Twoją aplikację, takie jak proste powiadomienia o zdarzeniach zachodzących w Twojej aplikacji. Funkcja powiadomień Laravel abstrahuje wysyłanie powiadomień przez różne sterowniki, takie jak e-mail, Slack, SMS lub przechowywane w bazie danych.

<a name="the-policies-directory"></a>
### Katalog Policies

Ten katalog nie istnieje domyślnie, ale zostanie utworzony dla Ciebie, jeśli wykonasz polecenie Artisan `make:policy`. Katalog `Policies` zawiera [klasy polityk autoryzacji](/docs/{{version}}/authorization) dla Twojej aplikacji. Polityki są używane do określenia, czy użytkownik może wykonać daną akcję na zasobie.

<a name="the-providers-directory"></a>
### Katalog Providers

Katalog `Providers` zawiera wszystkich [dostawców usług](/docs/{{version}}/providers) dla Twojej aplikacji. Dostawcy usług inicjalizują Twoją aplikację, wiążąc usługi w kontenerze usług, rejestrując zdarzenia lub wykonując inne zadania w celu przygotowania aplikacji na przychodzące żądania.

W świeżej aplikacji Laravel ten katalog będzie już zawierał `AppServiceProvider`. Możesz dodawać własnych dostawców do tego katalogu według potrzeb.

<a name="the-rules-directory"></a>
### Katalog Rules

Ten katalog nie istnieje domyślnie, ale zostanie utworzony dla Ciebie, jeśli wykonasz polecenie Artisan `make:rule`. Katalog `Rules` zawiera niestandardowe obiekty reguł walidacji dla Twojej aplikacji. Reguły są używane do enkapsulacji skomplikowanej logiki walidacji w prostym obiekcie. Aby uzyskać więcej informacji, sprawdź [dokumentację walidacji](/docs/{{version}}/validation).
