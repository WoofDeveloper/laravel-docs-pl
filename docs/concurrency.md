# Współbieżność

- [Wprowadzenie](#introduction)
- [Uruchamianie zadań współbieżnych](#running-concurrent-tasks)
- [Odraczanie zadań współbieżnych](#deferring-concurrent-tasks)

<a name="introduction"></a>
## Wprowadzenie

Czasami może być konieczne wykonanie kilku wolnych zadań, które nie zależą od siebie nawzajem. W wielu przypadkach znaczną poprawę wydajności można osiągnąć wykonując zadania współbieżnie. Fasada `Concurrency` w Laravelu zapewnia proste i wygodne API do wykonywania domknięć współbieżnie.

<a name="how-it-works"></a>
#### Jak to działa

Laravel osiąga współbieżność poprzez serializację podanych domknięć i wysyłanie ich do ukrytego polecenia Artisan CLI, które deserializuje domknięcia i wywołuje je w swoim własnym procesie PHP. Po wywołaniu domknięcia, wartość zwracana jest serializowana z powrotem do procesu nadrzędnego.

Fasada `Concurrency` obsługuje trzy sterowniki: `process` (domyślny), `fork` i `sync`.

Sterownik `fork` oferuje lepszą wydajność w porównaniu z domyślnym sterownikiem `process`, ale może być używany tylko w kontekście CLI PHP, ponieważ PHP nie obsługuje forkowania podczas żądań webowych. Przed użyciem sterownika `fork`, należy zainstalować pakiet `spatie/fork`:

```shell
composer require spatie/fork
```

Sterownik `sync` jest przydatny głównie podczas testowania, gdy chcesz wyłączyć całą współbieżność i po prostu wykonać podane domknięcia sekwencyjnie w procesie nadrzędnym.

<a name="running-concurrent-tasks"></a>
## Uruchamianie zadań współbieżnych

Aby uruchomić zadania współbieżne, możesz wywołać metodę `run` fasady `Concurrency`. Metoda `run` przyjmuje tablicę domknięć, które powinny być wykonane jednocześnie w podrzędnych procesach PHP:

```php
use Illuminate\Support\Facades\Concurrency;
use Illuminate\Support\Facades\DB;

[$userCount, $orderCount] = Concurrency::run([
    fn () => DB::table('users')->count(),
    fn () => DB::table('orders')->count(),
]);
```

Aby użyć konkretnego sterownika, możesz użyć metody `driver`:

```php
$results = Concurrency::driver('fork')->run(...);
```

Lub, aby zmienić domyślny sterownik współbieżności, należy opublikować plik konfiguracyjny `concurrency` za pomocą polecenia Artisan `config:publish` i zaktualizować opcję `default` w pliku:

```shell
php artisan config:publish concurrency
```

<a name="deferring-concurrent-tasks"></a>
## Odraczanie zadań współbieżnych

Jeśli chcesz wykonać tablicę domknięć współbieżnie, ale nie interesują Cię wyniki zwracane przez te domknięcia, powinieneś rozważyć użycie metody `defer`. Gdy metoda `defer` jest wywoływana, podane domknięcia nie są wykonywane natychmiast. Zamiast tego, Laravel wykona domknięcia współbieżnie po wysłaniu odpowiedzi HTTP do użytkownika:

```php
use App\Services\Metrics;
use Illuminate\Support\Facades\Concurrency;

Concurrency::defer([
    fn () => Metrics::report('users'),
    fn () => Metrics::report('orders'),
]);
```
