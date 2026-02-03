# Planowanie zadań

- [Wprowadzenie](#introduction)
- [Definiowanie harmonogramów](#defining-schedules)
    - [Planowanie poleceń Artisan](#scheduling-artisan-commands)
    - [Planowanie zadań kolejkowanych](#scheduling-queued-jobs)
    - [Planowanie poleceń powłoki](#scheduling-shell-commands)
    - [Opcje częstotliwości harmonogramu](#schedule-frequency-options)
    - [Strefy czasowe](#timezones)
    - [Zapobieganie nakładaniu się zadań](#preventing-task-overlaps)
    - [Uruchamianie zadań na jednym serwerze](#running-tasks-on-one-server)
    - [Zadania w tle](#background-tasks)
    - [Tryb konserwacji](#maintenance-mode)
    - [Grupy harmonogramów](#schedule-groups)
- [Uruchamianie harmonogramu](#running-the-scheduler)
    - [Zadania zaplanowane poniżej minuty](#sub-minute-scheduled-tasks)
    - [Lokalne uruchamianie harmonogramu](#running-the-scheduler-locally)
- [Wyjście zadań](#task-output)
- [Haki zadań](#task-hooks)
- [Wydarzenia](#events)

<a name="introduction"></a>
## Wprowadzenie

W przeszłości mogłeś pisać wpis konfiguracyjny crona dla każdego zadania, które musiałeś zaplanować na swoim serwerze. Jednak może to szybko stać się uciążliwe, ponieważ harmonogram Twoich zadań nie jest już w kontroli źródła i musisz się połączyć przez SSH z serwerem, aby zobaczyć istniejące wpisy crona lub dodać dodatkowe wpisy.

Harmonogram poleceń Laravel oferuje świeże podejście do zarządzania zaplanowanymi zadaniami na Twoim serwerze. Harmonogram pozwala płynnie i wyraźnie definiować harmonogram poleceń w samej aplikacji Laravel. Podczas korzystania z harmonogramu na serwerze potrzebny jest tylko jeden wpis crona. Harmonogram Twoich zadań jest zazwyczaj definiowany w pliku `routes/console.php` Twojej aplikacji.

<a name="defining-schedules"></a>
## Definiowanie harmonogramów

Możesz zdefiniować wszystkie swoje zaplanowane zadania w pliku `routes/console.php` Twojej aplikacji. Aby rozpocząć, spójrzmy na przykład. W tym przykładzie zaplanujemy wywołanie zamknięcia każdego dnia o północy. W zamknięciu wykonamy zapytanie do bazy danych, aby wyczyścić tabelę:

```php
<?php

use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->daily();
```

Oprócz planowania przy użyciu zamknięć, możesz również planować [obiekty wywołowalne](https://secure.php.net/manual/en/language.oop5.magic.php#object.invoke). Obiekty wywołowalne to proste klasy PHP, które zawierają metodę `__invoke`:

```php
Schedule::call(new DeleteRecentUsers)->daily();
```

Jeśli wolisz zarezerwować plik `routes/console.php` tylko dla definicji poleceń, możesz użyć metody `withSchedule` w pliku `bootstrap/app.php` Twojej aplikacji, aby zdefiniować swoje zaplanowane zadania. Ta metoda akceptuje zamknięcie, które otrzymuje instancję harmonogramu:

```php
use Illuminate\Console\Scheduling\Schedule;

->withSchedule(function (Schedule $schedule) {
    $schedule->call(new DeleteRecentUsers)->daily();
})
```

Jeśli chcesz zobaczyć przegląd swoich zaplanowanych zadań i następny czas ich uruchomienia, możesz użyć polecenia Artisan `schedule:list`:

```shell
php artisan schedule:list
```

<a name="scheduling-artisan-commands"></a>
### Planowanie poleceń Artisan

Oprócz planowania zamknięć, możesz również planować [polecenia Artisan](/docs/{{version}}/artisan) i polecenia systemowe. Na przykład możesz użyć metody `command`, aby zaplanować polecenie Artisan przy użyciu nazwy lub klasy polecenia.

Podczas planowania poleceń Artisan przy użyciu nazwy klasy polecenia, możesz przekazać tablicę dodatkowych argumentów wiersza poleceń, które powinny być dostarczone do polecenia, gdy zostanie wywołane:

```php
use App\Console\Commands\SendEmailsCommand;
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send Taylor --force')->daily();

Schedule::command(SendEmailsCommand::class, ['Taylor', '--force'])->daily();
```

<a name="scheduling-artisan-closure-commands"></a>
#### Planowanie poleceń Artisan zdefiniowanych przez zamknięcia

Jeśli chcesz zaplanować polecenie Artisan zdefiniowane przez zamknięcie, możesz połączyć łańcuchowo metody związane z planowaniem po definicji polecenia:

```php
Artisan::command('delete:recent-users', function () {
    DB::table('recent_users')->delete();
})->purpose('Delete recent users')->daily();
```

Jeśli musisz przekazać argumenty do polecenia zamknięcia, możesz je dostarczyć do metody `schedule`:

```php
Artisan::command('emails:send {user} {--force}', function ($user) {
    // ...
})->purpose('Send emails to the specified user')->schedule(['Taylor', '--force'])->daily();
```

<a name="scheduling-queued-jobs"></a>
### Planowanie zadań kolejkowanych

Metoda `job` może być używana do planowania [zadania kolejkowanego](/docs/{{version}}/queues). Ta metoda zapewnia wygodny sposób planowania zadań kolejkowanych bez konieczności używania metody `call` do definiowania zamknięć w celu dodania zadania do kolejki:

```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

Schedule::job(new Heartbeat)->everyFiveMinutes();
```

Opcjonalne drugi i trzeci argument mogą być dostarczone do metody `job`, które określają nazwę kolejki i połączenie kolejki, które powinny być używane do dodania zadania do kolejki:

```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

// Dispatch the job to the "heartbeats" queue on the "sqs" connection...
Schedule::job(new Heartbeat, 'heartbeats', 'sqs')->everyFiveMinutes();
```

<a name="scheduling-shell-commands"></a>
### Planowanie poleceń powłoki

Metoda `exec` może być używana do wydawania polecenia systemowi operacyjnemu:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::exec('node /home/forge/script.js')->daily();
```

<a name="schedule-frequency-options"></a>
### Opcje częstotliwości harmonogramu

Widzieliśmy już kilka przykładów, jak możesz skonfigurować zadanie do uruchamiania w określonych odstępach czasu. Jednak jest znacznie więcej częstotliwości harmonogramu zadań, które możesz przypisać do zadania:

<div class="overflow-auto">

| Metoda                                 | Opis                                                             |
| -------------------------------------- | ---------------------------------------------------------------- |
| `->cron('* * * * *');`                 | Uruchom zadanie według niestandardowego harmonogramu cron.       |
| `->everySecond();`                     | Uruchom zadanie co sekundę.                                      |
| `->everyTwoSeconds();`                 | Uruchom zadanie co dwie sekundy.                                 |
| `->everyFiveSeconds();`                | Uruchom zadanie co pięć sekund.                                  |
| `->everyTenSeconds();`                 | Uruchom zadanie co dziesięć sekund.                              |
| `->everyFifteenSeconds();`             | Uruchom zadanie co piętnaście sekund.                            |
| `->everyTwentySeconds();`              | Uruchom zadanie co dwadzieścia sekund.                           |
| `->everyThirtySeconds();`              | Uruchom zadanie co trzydzieści sekund.                           |
| `->everyMinute();`                     | Uruchom zadanie co minutę.                                       |
| `->everyTwoMinutes();`                 | Uruchom zadanie co dwie minuty.                                  |
| `->everyThreeMinutes();`               | Uruchom zadanie co trzy minuty.                                  |
| `->everyFourMinutes();`                | Uruchom zadanie co cztery minuty.                                |
| `->everyFiveMinutes();`                | Uruchom zadanie co pięć minut.                                   |
| `->everyTenMinutes();`                 | Uruchom zadanie co dziesięć minut.                               |
| `->everyFifteenMinutes();`             | Uruchom zadanie co piętnaście minut.                             |
| `->everyThirtyMinutes();`              | Uruchom zadanie co trzydzieści minut.                            |
| `->hourly();`                          | Uruchom zadanie co godzinę.                                      |
| `->hourlyAt(17);`                      | Uruchom zadanie co godzinę o 17 minut po pełnej godzinie.        |
| `->everyOddHour($minutes = 0);`        | Uruchom zadanie co nieparzystą godzinę.                          |
| `->everyTwoHours($minutes = 0);`       | Uruchom zadanie co dwie godziny.                                 |
| `->everyThreeHours($minutes = 0);`     | Uruchom zadanie co trzy godziny.                                 |
| `->everyFourHours($minutes = 0);`      | Uruchom zadanie co cztery godziny.                               |
| `->everySixHours($minutes = 0);`       | Uruchom zadanie co sześć godzin.                                 |
| `->daily();`                           | Uruchom zadanie codziennie o północy.                            |
| `->dailyAt('13:00');`                  | Uruchom zadanie codziennie o 13:00.                              |
| `->twiceDaily(1, 13);`                 | Uruchom zadanie codziennie o 1:00 i 13:00.                       |
| `->twiceDailyAt(1, 13, 15);`           | Uruchom zadanie codziennie o 1:15 i 13:15.                       |
| `->daysOfMonth([1, 10, 20]);`          | Uruchom zadanie w określone dni miesiąca.                        |
| `->weekly();`                          | Uruchom zadanie co niedzielę o 00:00.                            |
| `->weeklyOn(1, '8:00');`               | Uruchom zadanie co tydzień w poniedziałek o 8:00.                |
| `->monthly();`                         | Uruchom zadanie pierwszego dnia każdego miesiąca o 00:00.        |
| `->monthlyOn(4, '15:00');`             | Uruchom zadanie co miesiąc czwartego dnia o 15:00.               |
| `->twiceMonthly(1, 16, '13:00');`      | Uruchom zadanie miesięcznie pierwszego i szesnastego dnia o 13:00.|
| `->lastDayOfMonth('15:00');`           | Uruchom zadanie ostatniego dnia miesiąca o 15:00.                |
| `->quarterly();`                       | Uruchom zadanie pierwszego dnia każdego kwartału o 00:00.        |
| `->quarterlyOn(4, '14:00');`           | Uruchom zadanie co kwartał czwartego dnia o 14:00.               |
| `->yearly();`                          | Uruchom zadanie pierwszego dnia każdego roku o 00:00.            |
| `->yearlyOn(6, 1, '17:00');`           | Uruchom zadanie co roku 1 czerwca o 17:00.                       |
| `->timezone('America/New_York');`      | Ustaw strefę czasową dla zadania.                                |

</div>

Te metody mogą być łączone z dodatkowymi ograniczeniami, aby tworzyć jeszcze bardziej precyzyjne harmonogramy, które działają tylko w określone dni tygodnia. Na przykład możesz zaplanować uruchomienie polecenia raz w tygodniu w poniedziałek:

```php
use Illuminate\Support\Facades\Schedule;

// Run once per week on Monday at 1 PM...
Schedule::call(function () {
    // ...
})->weekly()->mondays()->at('13:00');

// Run hourly from 8 AM to 5 PM on weekdays...
Schedule::command('foo')
    ->weekdays()
    ->hourly()
    ->timezone('America/Chicago')
    ->between('8:00', '17:00');
```

Lista dodatkowych ograniczeń harmonogramu znajduje się poniżej:

<div class="overflow-auto">

| Metoda                                   | Opis                                                          |
| ---------------------------------------- | ------------------------------------------------------------- |
| `->weekdays();`                          | Ogranicz zadanie do dni roboczych.                            |
| `->weekends();`                          | Ogranicz zadanie do weekendów.                                |
| `->sundays();`                           | Ogranicz zadanie do niedzieli.                                |
| `->mondays();`                           | Ogranicz zadanie do poniedziałku.                             |
| `->tuesdays();`                          | Ogranicz zadanie do wtorku.                                   |
| `->wednesdays();`                        | Ogranicz zadanie do środy.                                    |
| `->thursdays();`                         | Ogranicz zadanie do czwartku.                                 |
| `->fridays();`                           | Ogranicz zadanie do piątku.                                   |
| `->saturdays();`                         | Ogranicz zadanie do soboty.                                   |
| `->days(array\|mixed);`                  | Ogranicz zadanie do określonych dni.                          |
| `->between($startTime, $endTime);`       | Ogranicz zadanie do uruchomienia między czasem rozpoczęcia i zakończenia.|
| `->unlessBetween($startTime, $endTime);` | Ogranicz zadanie, aby nie uruchamiać się między czasem rozpoczęcia i zakończenia.|
| `->when(Closure);`                       | Ogranicz zadanie na podstawie testu prawdziwości.             |
| `->environments($env);`                  | Ogranicz zadanie do określonych środowisk.                    |

</div>

<a name="day-constraints"></a>
#### Ograniczenia dni

Metoda `days` może być używana do ograniczenia wykonywania zadania do określonych dni tygodnia. Na przykład możesz zaplanować uruchomienie polecenia co godzinę w niedziele i środy:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->hourly()
    ->days([0, 3]);
```

Alternatywnie możesz użyć stałych dostępnych w klasie `Illuminate\Console\Scheduling\Schedule` podczas definiowania dni, w które zadanie powinno się uruchomić:

```php
use Illuminate\Support\Facades;
use Illuminate\Console\Scheduling\Schedule;

Facades\Schedule::command('emails:send')
    ->hourly()
    ->days([Schedule::SUNDAY, Schedule::WEDNESDAY]);
```

<a name="between-time-constraints"></a>
#### Ograniczenia czasowe między

Metoda `between` może być używana do ograniczenia wykonywania zadania w zależności od pory dnia:

```php
Schedule::command('emails:send')
    ->hourly()
    ->between('7:00', '22:00');
```

Podobnie metoda `unlessBetween` może być użyta do wykluczenia wykonywania zadania przez pewien okres czasu:

```php
Schedule::command('emails:send')
    ->hourly()
    ->unlessBetween('23:00', '4:00');
```

<a name="truth-test-constraints"></a>
#### Ograniczenia testu prawdziwości

Metoda `when` może być używana do ograniczenia wykonywania zadania na podstawie wyniku danego testu prawdziwości. Innymi słowy, jeśli dane zamknięcie zwróci `true`, zadanie zostanie wykonane, o ile żadne inne warunki ograniczające nie uniemożliwią uruchomienia zadania:

```php
Schedule::command('emails:send')->daily()->when(function () {
    return true;
});
```

Metoda `skip` może być postrzegana jako odwrotność `when`. Jeśli metoda `skip` zwróci `true`, zaplanowane zadanie nie zostanie wykonane:

```php
Schedule::command('emails:send')->daily()->skip(function () {
    return true;
});
```

Podczas korzystania z połączonych łańcuchowo metod `when`, zaplanowane polecenie zostanie wykonane tylko wtedy, gdy wszystkie warunki `when` zwrócą `true`.

<a name="environment-constraints"></a>
#### Ograniczenia środowiskowe

Metoda `environments` może być używana do wykonywania zadań tylko w danych środowiskach (zgodnie ze [zmienną środowiskową](/docs/{{version}}/configuration#environment-configuration) `APP_ENV`):

```php
Schedule::command('emails:send')
    ->daily()
    ->environments(['staging', 'production']);
```

<a name="timezones"></a>
### Strefy czasowe

Używając metody `timezone`, możesz określić, że czas zaplanowanego zadania powinien być interpretowany w danej strefie czasowej:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->timezone('America/New_York')
    ->at('2:00')
```

Jeśli wielokrotnie przypisujesz tę samą strefę czasową do wszystkich zaplanowanych zadań, możesz określić, która strefa czasowa powinna być przypisana do wszystkich harmonogramów, definiując opcję `schedule_timezone` w pliku konfiguracyjnym `app` Twojej aplikacji:

```php
'timezone' => 'UTC',

'schedule_timezone' => 'America/Chicago',
```

> [!WARNING]
> Pamiętaj, że niektóre strefy czasowe wykorzystują czas letni. Gdy następują zmiany czasu letniego, Twoje zaplanowane zadanie może uruchomić się dwa razy lub nawet wcale nie uruchomić się. Z tego powodu zalecamy unikanie planowania według stref czasowych, gdy jest to możliwe.

<a name="preventing-task-overlaps"></a>
### Zapobieganie nakładaniu się zadań

Domyślnie zaplanowane zadania będą uruchamiane, nawet jeśli poprzednia instancja zadania nadal działa. Aby temu zapobiec, możesz użyć metody `withoutOverlapping`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')->withoutOverlapping();
```

W tym przykładzie [polecenie Artisan](/docs/{{version}}/artisan) `emails:send` będzie uruchamiane co minutę, jeśli nie jest już uruchomione. Metoda `withoutOverlapping` jest szczególnie przydatna, jeśli masz zadania, które znacznie się różnią czasem wykonywania, uniemożliwiając przewidzenie dokładnie, ile czasu zajmie dane zadanie.

W razie potrzeby możesz określić, ile minut musi upłynąć, zanim blokada "bez nakładania się" wygaśnie. Domyślnie blokada wygaśnie po 24 godzinach:

```php
Schedule::command('emails:send')->withoutOverlapping(10);
```

Za kulisami metoda `withoutOverlapping` wykorzystuje [pamięć podręczną](/docs/{{version}}/cache) Twojej aplikacji do uzyskania blokad. W razie potrzeby możesz wyczyścić te blokady pamięci podręcznej za pomocą polecenia Artisan `schedule:clear-cache`. Jest to zazwyczaj konieczne tylko wtedy, gdy zadanie utknie z powodu nieoczekiwanego problemu z serwerem.

<a name="running-tasks-on-one-server"></a>
### Uruchamianie zadań na jednym serwerze

> [!WARNING]
> Aby korzystać z tej funkcji, Twoja aplikacja musi używać sterownika pamięci podręcznej `database`, `memcached`, `dynamodb` lub `redis` jako domyślnego sterownika pamięci podręcznej Twojej aplikacji. Ponadto wszystkie serwery muszą komunikować się z tym samym centralnym serwerem pamięci podręcznej.

Jeśli harmonogram Twojej aplikacji działa na wielu serwerach, możesz ograniczyć zaplanowane zadanie do wykonywania tylko na jednym serwerze. Na przykład załóżmy, że masz zaplanowane zadanie, które generuje nowy raport każdego piątku wieczorem. Jeśli harmonogram zadań działa na trzech serwerach roboczych, zaplanowane zadanie zostanie uruchomione na wszystkich trzech serwerach i wygeneruje raport trzy razy. Niedobrze!

Aby wskazać, że zadanie powinno działać tylko na jednym serwerze, użyj metody `onOneServer` podczas definiowania zaplanowanego zadania. Pierwszy serwer, który uzyska zadanie, zabezpieczy atomową blokadę zadania, aby zapobiec uruchomieniu tego samego zadania w tym samym czasie na innych serwerach:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->fridays()
    ->at('17:00')
    ->onOneServer();
```

Możesz użyć metody `useCache`, aby dostosować magazyn pamięci podręcznej używany przez harmonogram do uzyskania atomowych blokad niezbędnych dla zadań jednoserwerowych:

```php
Schedule::useCache('database');
```

<a name="naming-unique-jobs"></a>
#### Nazywanie zadań jednoserwerowych

Czasami może być konieczne zaplanowanie tego samego zadania do wysłania z różnymi parametrami, jednocześnie instruując Laravel, aby uruchomić każdą permutację zadania na jednym serwerze. Aby to osiągnąć, możesz przypisać każdej definicji harmonogramu unikalną nazwę za pomocą metody `name`:

```php
Schedule::job(new CheckUptime('https://laravel.com'))
    ->name('check_uptime:laravel.com')
    ->everyFiveMinutes()
    ->onOneServer();

Schedule::job(new CheckUptime('https://vapor.laravel.com'))
    ->name('check_uptime:vapor.laravel.com')
    ->everyFiveMinutes()
    ->onOneServer();
```

Podobnie, zaplanowane zamknięcia muszą mieć przypisaną nazwę, jeśli mają być uruchomione na jednym serwerze:

```php
Schedule::call(fn () => User::resetApiRequestCount())
    ->name('reset-api-request-count')
    ->daily()
    ->onOneServer();
```

<a name="background-tasks"></a>
### Zadania w tle

<a name="background-tasks"></a>
### Zadania w tle

Domyślnie wiele zadań zaplanowanych w tym samym czasie będzie wykonywanych sekwencyjnie w oparciu o kolejność, w jakiej zostały zdefiniowane w Twojej metodzie `schedule`. Jeśli masz długo działające zadania, może to spowodować, że kolejne zadania rozpoczną się znacznie później niż przewidywano. Jeśli chcesz uruchamiać zadania w tle, aby mogły wszystkie działać jednocześnie, możesz użyć metody `runInBackground`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('analytics:report')
    ->daily()
    ->runInBackground();
```

> [!WARNING]
> Metoda `runInBackground` może być używana tylko podczas planowania zadań za pomocą metod `command` i `exec`.

<a name="maintenance-mode"></a>
### Tryb konserwacji

Zaplanowane zadania Twojej aplikacji nie będą uruchamiane, gdy aplikacja jest w [trybie konserwacji](/docs/{{version}}/configuration#maintenance-mode), ponieważ nie chcemy, aby Twoje zadania zakłócały jakąkolwiek niedokończoną konserwację, którą możesz wykonywać na swoim serwerze. Jednak jeśli chcesz wymusić uruchomienie zadania nawet w trybie konserwacji, możesz wywołać metodę `evenInMaintenanceMode` podczas definiowania zadania:

```php
Schedule::command('emails:send')->evenInMaintenanceMode();
```

<a name="schedule-groups"></a>
### Grupy harmonogramów

Podczas definiowania wielu zaplanowanych zadań o podobnych konfiguracjach możesz użyć funkcji grupowania zadań Laravel, aby uniknąć powtarzania tych samych ustawień dla każdego zadania. Grupowanie zadań upraszcza Twój kod i zapewnia spójność między powiązanymi zadaniami.

Aby utworzyć grupę zaplanowanych zadań, wywołaj żądane metody konfiguracji zadań, a następnie metodę `group`. Metoda `group` akceptuje zamknięcie, które odpowiada za definiowanie zadań współdzielących określoną konfigurację:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::daily()
    ->onOneServer()
    ->timezone('America/New_York')
    ->group(function () {
        Schedule::command('emails:send --force');
        Schedule::command('emails:prune');
    });
```

<a name="running-the-scheduler"></a>
## Uruchamianie harmonogramu

Teraz, gdy nauczyliśmy się, jak definiować zaplanowane zadania, omówmy, jak faktycznie uruchamiać je na naszym serwerze. Polecenie Artisan `schedule:run` oceni wszystkie Twoje zaplanowane zadania i określi, czy muszą zostać uruchomione na podstawie bieżącego czasu serwera.

Tak więc podczas korzystania z harmonogramu Laravel potrzebujemy tylko dodać jeden wpis konfiguracyjny crona do naszego serwera, który uruchamia polecenie `schedule:run` co minutę. Jeśli nie wiesz, jak dodać wpisy crona do swojego serwera, rozważ użycie platformy zarządzanej, takiej jak [Laravel Cloud](https://cloud.laravel.com), która może zarządzać wykonywaniem zaplanowanych zadań za Ciebie:

```shell
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

<a name="sub-minute-scheduled-tasks"></a>
### Zadania zaplanowane poniżej minuty

W większości systemów operacyjnych zadania cron są ograniczone do uruchamiania maksymalnie raz na minutę. Jednak harmonogram Laravel pozwala zaplanować zadania do uruchamiania w częstszych odstępach czasu, nawet tak często jak raz na sekundę:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->everySecond();
```

Gdy zadania poniżej minuty są zdefiniowane w Twojej aplikacji, polecenie `schedule:run` będzie kontynuować działanie do końca bieżącej minuty zamiast natychmiastowego zakończenia. Pozwala to poleceniu wywołać wszystkie wymagane zadania poniżej minuty przez całą minutę.

Ponieważ zadania poniżej minuty, które trwają dłużej niż oczekiwano, mogą opóźnić wykonanie późniejszych zadań poniżej minuty, zaleca się, aby wszystkie zadania poniżej minuty wysyłały zadania kolejkowane lub polecenia w tle do obsługi faktycznego przetwarzania zadań:

```php
use App\Jobs\DeleteRecentUsers;

Schedule::job(new DeleteRecentUsers)->everyTenSeconds();

Schedule::command('users:delete')->everyTenSeconds()->runInBackground();
```

<a name="interrupting-sub-minute-tasks"></a>
#### Przerywanie zadań poniżej minuty

Ponieważ polecenie `schedule:run` działa przez całą minutę wywołania, gdy zdefiniowane są zadania poniżej minuty, czasami może być konieczne przerwanie polecenia podczas wdrażania aplikacji. W przeciwnym razie instancja polecenia `schedule:run`, która już działa, będzie nadal używać wcześniej wdrożonego kodu Twojej aplikacji, aż do końca bieżącej minuty.

Aby przerwać trwające wywołania `schedule:run`, możesz dodać polecenie `schedule:interrupt` do skryptu wdrażania swojej aplikacji. To polecenie powinno zostać wywołane po zakończeniu wdrażania aplikacji:

```shell
php artisan schedule:interrupt
```

<a name="running-the-scheduler-locally"></a>
### Lokalne uruchamianie harmonogramu

Zazwyczaj nie dodałbyś wpisu crona harmonogramu do swojej lokalnej maszyny rozwojowej. Zamiast tego możesz użyć polecenia Artisan `schedule:work`. To polecenie będzie działać na pierwszym planie i wywoływać harmonogram co minutę, aż zakończysz polecenie. Gdy zdefiniowane są zadania poniżej minuty, harmonogram będzie kontynuował działanie w każdej minucie, aby przetworzyć te zadania:

```shell
php artisan schedule:work
```

<a name="task-output"></a>
## Wyjście zadań

Harmonogram Laravel zapewnia kilka wygodnych metod do pracy z wyjściem generowanym przez zaplanowane zadania. Po pierwsze, za pomocą metody `sendOutputTo`, możesz wysłać wyjście do pliku w celu późniejszej inspekcji:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->daily()
    ->sendOutputTo($filePath);
```

Jeśli chcesz dołączyć wyjście do danego pliku, możesz użyć metody `appendOutputTo`:

```php
Schedule::command('emails:send')
    ->daily()
    ->appendOutputTo($filePath);
```

Za pomocą metody `emailOutputTo` możesz wysłać wyjście e-mailem na wybrany adres e-mail. Przed wysłaniem wyjścia zadania e-mailem powinieneś skonfigurować [usługi e-mail](/docs/{{version}}/mail) Laravel:

```php
Schedule::command('report:generate')
    ->daily()
    ->sendOutputTo($filePath)
    ->emailOutputTo('taylor@example.com');
```

Jeśli chcesz wysłać e-mailem wyjście tylko wtedy, gdy zaplanowane polecenie Artisan lub systemowe zakończy się niezerowym kodem wyjścia, użyj metody `emailOutputOnFailure`:

```php
Schedule::command('report:generate')
    ->daily()
    ->emailOutputOnFailure('taylor@example.com');
```

> [!WARNING]
> Metody `emailOutputTo`, `emailOutputOnFailure`, `sendOutputTo` i `appendOutputTo` są wyłączne dla metod `command` i `exec`.

<a name="task-hooks"></a>
## Haki zadań

Za pomocą metod `before` i `after` możesz określić kod, który ma zostać wykonany przed i po wykonaniu zaplanowanego zadania:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->daily()
    ->before(function () {
        // The task is about to execute...
    })
    ->after(function () {
        // The task has executed...
    });
```

Metody `onSuccess` i `onFailure` pozwalają określić kod, który ma zostać wykonany, jeśli zaplanowane zadanie się powiedzie lub nie powiedzie. Niepowodzenie oznacza, że zaplanowane polecenie Artisan lub systemowe zakończyło się niezerowym kodem wyjścia:

```php
Schedule::command('emails:send')
    ->daily()
    ->onSuccess(function () {
        // The task succeeded...
    })
    ->onFailure(function () {
        // The task failed...
    });
```

Jeśli wyjście jest dostępne z Twojego polecenia, możesz uzyskać do niego dostęp w swoich hakach `after`, `onSuccess` lub `onFailure`, wskazując typ instancji `Illuminate\Support\Stringable` jako argument `$output` definicji zamknięcia Twojego haka:

```php
use Illuminate\Support\Stringable;

Schedule::command('emails:send')
    ->daily()
    ->onSuccess(function (Stringable $output) {
        // The task succeeded...
    })
    ->onFailure(function (Stringable $output) {
        // The task failed...
    });
```

<a name="pinging-urls"></a>
#### Pingowanie URL-i

Za pomocą metod `pingBefore` i `thenPing`, harmonogram może automatycznie pingować dany URL przed lub po wykonaniu zadania. Ta metoda jest przydatna do powiadamiania zewnętrznej usługi, takiej jak [Envoyer](https://envoyer.io), że Twoje zaplanowane zadanie rozpoczyna się lub zakończyło wykonywanie:

```php
Schedule::command('emails:send')
    ->daily()
    ->pingBefore($url)
    ->thenPing($url);
```

Metody `pingOnSuccess` i `pingOnFailure` mogą być używane do pingowania danego URL tylko wtedy, gdy zadanie się powiedzie lub nie powiedzie. Niepowodzenie oznacza, że zaplanowane polecenie Artisan lub systemowe zakończyło się niezerowym kodem wyjścia:

```php
Schedule::command('emails:send')
    ->daily()
    ->pingOnSuccess($successUrl)
    ->pingOnFailure($failureUrl);
```

Metody `pingBeforeIf`, `thenPingIf`, `pingOnSuccessIf` i `pingOnFailureIf` mogą być używane do pingowania danego URL tylko wtedy, gdy dany warunek jest `true`:

```php
Schedule::command('emails:send')
    ->daily()
    ->pingBeforeIf($condition, $url)
    ->thenPingIf($condition, $url);

Schedule::command('emails:send')
    ->daily()
    ->pingOnSuccessIf($condition, $successUrl)
    ->pingOnFailureIf($condition, $failureUrl);
```

<a name="events"></a>
## Wydarzenia

Laravel wysyła różne [wydarzenia](/docs/{{version}}/events) podczas procesu planowania. Możesz [zdefiniować nasłuchiwacze](/docs/{{version}}/events) dla dowolnego z następujących wydarzeń:

<div class="overflow-auto">

| Nazwa wydarzenia                                            |
| ----------------------------------------------------------- |
| `Illuminate\Console\Events\ScheduledTaskStarting`           |
| `Illuminate\Console\Events\ScheduledTaskFinished`           |
| `Illuminate\Console\Events\ScheduledBackgroundTaskFinished` |
| `Illuminate\Console\Events\ScheduledTaskSkipped`            |
| `Illuminate\Console\Events\ScheduledTaskFailed`             |

</div>
