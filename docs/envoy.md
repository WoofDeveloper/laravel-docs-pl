# Laravel Envoy

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
- [Pisanie zadań](#writing-tasks)
    - [Definiowanie zadań](#defining-tasks)
    - [Wiele serwerów](#multiple-servers)
    - [Konfiguracja](#setup)
    - [Zmienne](#variables)
    - [Historie](#stories)
    - [Hooki](#completion-hooks)
- [Uruchamianie zadań](#running-tasks)
    - [Potwierdzanie wykonania zadania](#confirming-task-execution)
- [Powiadomienia](#notifications)
    - [Slack](#slack)
    - [Discord](#discord)
    - [Telegram](#telegram)
    - [Microsoft Teams](#microsoft-teams)

<a name="introduction"></a>
## Wprowadzenie

[Laravel Envoy](https://github.com/laravel/envoy) jest narzędziem do wykonywania typowych zadań, które uruchamiasz na swoich zdalnych serwerach. Używając składni w stylu [Blade](/docs/{{version}}/blade), możesz łatwo skonfigurować zadania do wdrażania, poleceń Artisan i więcej. Obecnie Envoy obsługuje tylko systemy operacyjne Mac i Linux. Jednak obsługa Windows jest możliwa przy użyciu [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

<a name="installation"></a>
## Instalacja

Najpierw zainstaluj Envoy w swoim projekcie używając menedżera pakietów Composer:

```shell
composer require laravel/envoy --dev
```

Po zainstalowaniu Envoy, plik binarny Envoy będzie dostępny w katalogu `vendor/bin` Twojej aplikacji:

```shell
php vendor/bin/envoy
```

<a name="writing-tasks"></a>
## Pisanie zadań

<a name="defining-tasks"></a>
### Definiowanie zadań

Zadania są podstawowym elementem składowym Envoy. Zadania definiują polecenia powłoki, które powinny zostać wykonane na Twoich zdalnych serwerach, gdy zadanie jest wywoływane. Na przykład, możesz zdefiniować zadanie, które wykonuje polecenie `php artisan queue:restart` na wszystkich serwerach procesów kolejki Twojej aplikacji.

Wszystkie Twoje zadania Envoy powinny być zdefiniowane w pliku `Envoy.blade.php` w katalogu głównym Twojej aplikacji. Oto przykład dla Ciebie na początek:

```blade
@servers(['web' => ['user@192.168.1.1'], 'workers' => ['user@192.168.1.2']])

@task('restart-queues', ['on' => 'workers'])
    cd /home/user/example.com
    php artisan queue:restart
@endtask
```

Jak widać, tablica `@servers` jest zdefiniowana na górze pliku, umożliwiając Ci odniesienie się do tych serwerów za pomocą opcji `on` w deklaracjach zadań. Deklaracja `@servers` powinna zawsze być umieszczona w jednej linii. Wewnątrz deklaracji `@task` powinieneś umieścić polecenia powłoki, które powinny zostać wykonane na Twoich serwerach, gdy zadanie jest wywoływane.

<a name="local-tasks"></a>
#### Zadania lokalne

Możesz wymusić uruchomienie skryptu na swoim lokalnym komputerze, określając adres IP serwera jako `127.0.0.1`:

```blade
@servers(['localhost' => '127.0.0.1'])
```

<a name="importing-envoy-tasks"></a>
#### Importowanie zadań Envoy

Używając dyrektywy `@import`, możesz zaimportować inne pliki Envoy, tak aby ich historie i zadania zostały dodane do Twoich. Po zaimportowaniu plików możesz wykonywać zawarte w nich zadania tak, jakby były zdefiniowane w Twoim własnym pliku Envoy:

```blade
@import('vendor/package/Envoy.blade.php')
```

<a name="multiple-servers"></a>
### Wiele serwerów

Envoy pozwala łatwo uruchomić zadanie na wielu serwerach. Najpierw dodaj dodatkowe serwery do swojej deklaracji `@servers`. Każdemu serwerowi powinna być przypisana unikalna nazwa. Po zdefiniowaniu dodatkowych serwerów możesz wymienić każdy z serwerów w tablicy `on` zadania:

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2']])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

<a name="parallel-execution"></a>
#### Wykonywanie równoległe

Domyślnie zadania będą wykonywane na każdym serwerze szeregowo. Innymi słowy, zadanie zakończy działanie na pierwszym serwerze przed przejściem do wykonania na drugim serwerze. Jeśli chcesz uruchomić zadanie na wielu serwerach równolegle, dodaj opcję `parallel` do deklaracji zadania:

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

<a name="setup"></a>
### Konfiguracja

Czasami możesz potrzebować wykonać dowolny kod PHP przed uruchomieniem swoich zadań Envoy. Możesz użyć dyrektywy `@setup`, aby zdefiniować blok kodu PHP, który powinien zostać wykonany przed Twoimi zadaniami:

```php
@setup
    $now = new DateTime;
@endsetup
```

Jeśli potrzebujesz dołączyć inne pliki PHP przed wykonaniem zadania, możesz użyć dyrektywy `@include` na górze swojego pliku `Envoy.blade.php`:

```blade
@include('vendor/autoload.php')

@task('restart-queues')
    # ...
@endtask
```

<a name="variables"></a>
### Zmienne

Jeśli to konieczne, możesz przekazywać argumenty do zadań Envoy, określając je w wierszu poleceń podczas wywoływania Envoy:

```shell
php vendor/bin/envoy run deploy --branch=master
```

Możesz uzyskać dostęp do opcji w swoich zadaniach używając składni "echo" Blade. Możesz również definiować instrukcje `if` Blade i pętle wewnątrz swoich zadań. Na przykład, zweryfikujmy obecność zmiennej `$branch` przed wykonaniem polecenia `git pull`:

```blade
@servers(['web' => ['user@192.168.1.1']])

@task('deploy', ['on' => 'web'])
    cd /home/user/example.com

    @if ($branch)
        git pull origin {{ $branch }}
    @endif

    php artisan migrate --force
@endtask
```

<a name="stories"></a>
### Historie

Historie grupują zestaw zadań pod jedną, wygodną nazwą. Na przykład historia `deploy` może uruchamiać zadania `update-code` i `install-dependencies`, wymieniając nazwy zadań w swojej definicji:

```blade
@servers(['web' => ['user@192.168.1.1']])

@story('deploy')
    update-code
    install-dependencies
@endstory

@task('update-code')
    cd /home/user/example.com
    git pull origin master
@endtask

@task('install-dependencies')
    cd /home/user/example.com
    composer install
@endtask
```

Po napisaniu historii możesz ją wywołać w taki sam sposób, jak wywołałbyś zadanie:

```shell
php vendor/bin/envoy run deploy
```

<a name="completion-hooks"></a>
### Hooki

Gdy zadania i historie są uruchamiane, wykonywanych jest wiele hooków. Typy hooków obsługiwane przez Envoy to `@before`, `@after`, `@error`, `@success` i `@finished`. Cały kod w tych hookach jest interpretowany jako PHP i wykonywany lokalnie, a nie na zdalnych serwerach, z którymi wchodzą w interakcję Twoje zadania.

Możesz zdefiniować tyle każdego z tych hooków, ile chcesz. Będą one wykonywane w kolejności, w jakiej pojawiają się w Twoim skrypcie Envoy.

<a name="hook-before"></a>
#### `@before`

Przed każdym wykonaniem zadania, wszystkie hooki `@before` zarejestrowane w Twoim skrypcie Envoy zostaną wykonane. Hooki `@before` otrzymują nazwę zadania, które zostanie wykonane:

```blade
@before
    if ($task === 'deploy') {
        // ...
    }
@endbefore
```

<a name="completion-after"></a>
#### `@after`

Po każdym wykonaniu zadania, wszystkie hooki `@after` zarejestrowane w Twoim skrypcie Envoy zostaną wykonane. Hooki `@after` otrzymują nazwę zadania, które zostało wykonane:

```blade
@after
    if ($task === 'deploy') {
        // ...
    }
@endafter
```

<a name="completion-error"></a>
#### `@error`

Po każdym niepowodzeniu zadania (kończy się z kodem statusu większym niż `0`), wszystkie hooki `@error` zarejestrowane w Twoim skrypcie Envoy zostaną wykonane. Hooki `@error` otrzymują nazwę zadania, które zostało wykonane:

```blade
@error
    if ($task === 'deploy') {
        // ...
    }
@enderror
```

<a name="completion-success"></a>
#### `@success`

Jeśli wszystkie zadania zostały wykonane bez błędów, wszystkie hooki `@success` zarejestrowane w Twoim skrypcie Envoy zostaną wykonane:

```blade
@success
    // ...
@endsuccess
```

<a name="completion-finished"></a>
#### `@finished`

Po wykonaniu wszystkich zadań (niezależnie od statusu wyjścia), wszystkie hooki `@finished` zostaną wykonane. Hooki `@finished` otrzymują kod statusu ukończonego zadania, który może być `null` lub `integer` większym lub równym `0`:

```blade
@finished
    if ($exitCode > 0) {
        // Wystąpiły błędy w jednym z zadań...
    }
@endfinished
```

<a name="running-tasks"></a>
## Uruchamianie zadań

Aby uruchomić zadanie lub historię zdefiniowaną w pliku `Envoy.blade.php` Twojej aplikacji, wykonaj polecenie `run` Envoy, przekazując nazwę zadania lub historii, którą chcesz wykonać. Envoy wykona zadanie i wyświetli wyjście z Twoich zdalnych serwerów w trakcie działania zadania:

```shell
php vendor/bin/envoy run deploy
```

<a name="confirming-task-execution"></a>
### Potwierdzanie wykonania zadania

Jeśli chcesz zostać poproszony o potwierdzenie przed uruchomieniem danego zadania na Twoich serwerach, powinieneś dodać dyrektywę `confirm` do deklaracji zadania. Ta opcja jest szczególnie przydatna dla operacji destrukcyjnych:

```blade
@task('deploy', ['on' => 'web', 'confirm' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate
@endtask
```

<a name="notifications"></a>
## Powiadomienia

<a name="slack"></a>
### Slack

Envoy obsługuje wysyłanie powiadomień do [Slack](https://slack.com) po wykonaniu każdego zadania. Dyrektywa `@slack` przyjmuje URL hooka Slack i nazwę kanału / użytkownika. Możesz pobrać swój URL webhooka tworząc integrację "Incoming WebHooks" w swoim panelu sterowania Slack.

Powinieneś przekazać cały URL webhooka jako pierwszy argument podany do dyrektywy `@slack`. Drugi argument podany do dyrektywy `@slack` powinien być nazwą kanału (`#channel`) lub nazwą użytkownika (`@user`):

```blade
@finished
    @slack('webhook-url', '#bots')
@endfinished
```

Domyślnie powiadomienia Envoy będą wysyłać wiadomość do kanału powiadomień opisującą zadanie, które zostało wykonane. Jednak możesz nadpisać tę wiadomość własną niestandardową wiadomością, przekazując trzeci argument do dyrektywy `@slack`:

```blade
@finished
    @slack('webhook-url', '#bots', 'Hello, Slack.')
@endfinished
```

<a name="discord"></a>
### Discord

Envoy obsługuje również wysyłanie powiadomień do [Discord](https://discord.com) po wykonaniu każdego zadania. Dyrektywa `@discord` przyjmuje URL hooka Discord i wiadomość. Możesz pobrać swój URL webhooka tworząc "Webhook" w ustawieniach serwera i wybierając kanał, na który webhook powinien publikować. Powinieneś przekazać cały URL webhooka do dyrektywy `@discord`:

```blade
@finished
    @discord('discord-webhook-url')
@endfinished
```

<a name="telegram"></a>
### Telegram

Envoy obsługuje również wysyłanie powiadomień do [Telegram](https://telegram.org) po wykonaniu każdego zadania. Dyrektywa `@telegram` przyjmuje ID bota Telegram i ID czatu. Możesz pobrać swój ID bota tworząc nowego bota za pomocą [BotFather](https://t.me/botfather). Możesz pobrać prawidłowy ID czatu używając [@username_to_id_bot](https://t.me/username_to_id_bot). Powinieneś przekazać cały ID bota i ID czatu do dyrektywy `@telegram`:

```blade
@finished
    @telegram('bot-id','chat-id')
@endfinished
```

<a name="microsoft-teams"></a>
### Microsoft Teams

Envoy obsługuje również wysyłanie powiadomień do [Microsoft Teams](https://www.microsoft.com/en-us/microsoft-teams) po wykonaniu każdego zadania. Dyrektywa `@microsoftTeams` przyjmuje webhook Teams (wymagany), wiadomość, kolor motywu (success, info, warning, error) i tablicę opcji. Możesz pobrać swój webhook Teams tworząc nowy [incoming webhook](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook). API Teams ma wiele innych atrybutów do dostosowania skrzynki wiadomości, takich jak tytuł, podsumowanie i sekcje. Więcej informacji znajdziesz w [dokumentacji Microsoft Teams](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using?tabs=cURL#example-of-connector-message). Powinieneś przekazać cały URL webhooka do dyrektywy `@microsoftTeams`:

```blade
@finished
    @microsoftTeams('webhook-url')
@endfinished
```
