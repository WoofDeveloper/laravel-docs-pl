# Przewodnik Aktualizacji

- [Aktualizacja Do 12.0 Z 11.x](#upgrade-12.0)

<a name="high-impact-changes"></a>
## Zmiany o Wysokim Wpływie

<div class="content-list" markdown="1">

- [Aktualizacja Zależności](#updating-dependencies)
- [Aktualizacja Instalatora Laravel](#updating-the-laravel-installer)

</div>

<a name="medium-impact-changes"></a>
## Zmiany o Średnim Wpływie

<div class="content-list" markdown="1">

- [Modele i UUIDv7](#models-and-uuidv7)

</div>

<a name="low-impact-changes"></a>
## Zmiany o Niskim Wpływie

<div class="content-list" markdown="1">

- [Carbon 3](#carbon-3)
- [Mapowanie Indeksów Wyników Concurrency](#concurrency-result-index-mapping)
- [Rozwiązywanie Zależności Klas Kontenera](#container-class-dependency-resolution)
- [Walidacja Obrazów Teraz Wyklucza SVG](#image-validation)
- [Domyślna Ścieżka Root Dysku Lokalnego Systemu Plików](#local-filesystem-disk-default-root-path)
- [Inspekcja Baz Danych Wieloschemowych](#multi-schema-database-inspecting)
- [Scalanie Zagnieżdżonych Tablic Żądań](#nested-array-request-merging)

</div>

<a name="upgrade-12.0"></a>
## Aktualizacja Do 12.0 Z 11.x

#### Szacowany Czas Aktualizacji: 5 Minut

> [!NOTE]
> Staramy się dokumentować każdą możliwą zmianę przełamującą. Ponieważ niektóre z tych zmian znajdują się w mało znanych częściach frameworka, tylko część z nich może faktycznie wpłynąć na Twoją aplikację. Chcesz zaoszczędzić czas? Możesz użyć [Laravel Shift](https://laravelshift.com/), aby pomóc zautomatyzować aktualizacje aplikacji.

<a name="updating-dependencies"></a>
### Aktualizacja Zależności

**Prawdopodobieństwo Wpływu: Wysokie**

Powinieneś zaktualizować następujące zależności w pliku `composer.json` Twojej aplikacji:

<div class="content-list" markdown="1">

- `laravel/framework` do `^12.0`
- `phpunit/phpunit` do `^11.0`
- `pestphp/pest` do `^3.0`

</div>

<a name="carbon-3"></a>
#### Carbon 3

**Prawdopodobieństwo Wpływu: Niskie**

Wsparcie dla Carbon 2.x zostało usunięte. Wszystkie aplikacje Laravel 12 wymagają teraz [Carbon 3.x](https://carbon.nesbot.com/guide/getting-started/migration.html).

<a name="updating-the-laravel-installer"></a>
### Aktualizacja Instalatora Laravel

Je jesli używasz narzędzia CLI instalatora Laravel do tworzenia nowych aplikacji Laravel, powinieneś zaktualizować instalację instalatora, aby była kompatybilna z Laravel 12.x i [nowymi startowymi zestawami Laravel](https://laravel.com/starter-kits). Jeśli zainstalowałeś instalator Laravel przez `composer global require`, możesz zaktualizować instalator używając `composer global update`:

```shell
composer global update laravel/installer
```

Jeśli pierwotnie zainstalowałeś PHP i Laravel przez `php.new`, możesz po prostu ponownie uruchomić polecenia instalacji `php.new` dla Twojego systemu operacyjnego, aby zainstalować najnowszą wersję PHP i instalatora Laravel:

```shell tab=macOS
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"
```

```shell tab=Windows PowerShell
# Uruchom jako administrator...
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.4'))
```

```shell tab=Linux
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
```

Lub, jeśli używasz dołączonej kopii instalatora Laravel z [Laravel Herd](https://herd.laravel.com), powinieneś zaktualizować instalację Herd do najnowszej wersji.

<a name="authentication"></a>
### Uwierzytelnianie

<a name="updated-databasetokenrepository-constructor-signature"></a>
#### Zaktualizowana Sygnatura Konstruktora `DatabaseTokenRepository`

**Prawdopodobieństwo Wpływu: Bardzo Niskie**

Konstruktor klasy `Illuminate\Auth\Passwords\DatabaseTokenRepository` oczekuje teraz, że parametr `$expires` będzie podany w sekundach, a nie minutach.

<a name="concurrency"></a>
### Współbieżność

<a name="concurrency-result-index-mapping"></a>
#### Mapowanie Indeksów Wyników Concurrency

**Prawdopodobieństwo Wpływu: Niskie**

Podczas wywoływania metody `Concurrency::run` z tablicą asocjacyjną, wyniki operacji współbieżnych są teraz zwracane z ich powiązanymi kluczami:

```php
$result = Concurrency::run([
    'task-1' => fn () => 1 + 1,
    'task-2' => fn () => 2 + 2,
]);

// ['task-1' => 2, 'task-2' => 4]
```

<a name="container"></a>
### Kontener

<a name="container-class-dependency-resolution"></a>
#### Rozwiązywanie Zależności Klas Kontenera

**Prawdopodobieństwo Wpływu: Niskie**

Kontener wstrzykiwania zależności respektuje teraz wartości domyślne właściwości klas podczas rozwiązywania instancji klasy. Jeśli wcześniej polegałeś na kontenerze do rozwiązywania instancji klasy bez wartości domyślnej, możesz potrzebować dostosować swą aplikację do tego nowego zachowania:

```php
class Example
{
    public function __construct(public ?Carbon $date = null) {}
}

$example = resolve(Example::class);

// <= 11.x
$example->date instanceof Carbon;

// >= 12.x
$example->date === null;
```

<a name="database"></a>
### Baza Danych

<a name="multi-schema-database-inspecting"></a>
#### Inspekcja Baz Danych Wieloschemowych

**Prawdopodobieństwo Wpływu: Niskie**

Metody `Schema::getTables()`, `Schema::getViews()` i `Schema::getTypes()` zawierają teraz domyślnie wyniki ze wszystkich schematów. Możesz przekazać argument `schema`, aby uzyskać wynik tylko dla danego schematu:

```php
// Wszystkie tabele ze wszystkich schematów...
$tables = Schema::getTables();

// Wszystkie tabele ze schematu 'main'...
$tables = Schema::getTables(schema: 'main');

// Wszystkie tabele ze schematów 'main' i 'blog'...
$tables = Schema::getTables(schema: ['main', 'blog']);
```

Metoda `Schema::getTableListing()` zwraca teraz domyślnie nazwy tabel kwalifikowane schematem. Możesz przekazać argument `schemaQualified`, aby zmienić zachowanie według życzenia:

```php
$tables = Schema::getTableListing();
// ['main.migrations', 'main.users', 'blog.posts']

$tables = Schema::getTableListing(schema: 'main');
// ['main.migrations', 'main.users']

$tables = Schema::getTableListing(schema: 'main', schemaQualified: false);
// ['migrations', 'users']
```

Polecenia `db:table` i `db:show` zwracają teraz wyniki wszystkich schematów w MySQL, MariaDB i SQLite, podobnie jak PostgreSQL i SQL Server.

<a name="database-constructor-signature-changes"></a>
#### Zmiany Sygnatury Konstruktora Bazy Danych

**Prawdopodobieństwo Wpływu: Bardzo Niskie**

W Laravel 12, kilka niskopoziomowych klas bazy danych wymaga teraz instancji `Illuminate\Database\Connection` dostarczonej przez ich konstruktory.

**Te zmiany dotyczą głównie opiekunów pakietów bazodanowych - jest bardzo mało prawdopodobne, aby którakolwiek z tych zmian wpłynęła na normalne tworzenie aplikacji.**

`Illuminate\Database\Schema\Blueprint`

Konstruktor klasy `Illuminate\Database\Schema\Blueprint` oczekuje teraz instancji `Connection` jako pierwszego argumentu. Dotyczy to głównie aplikacji lub pakietów, które ręcznie tworzą instancje `Blueprint`.

`Illuminate\Database\Grammar`

Konstruktor klasy `Illuminate\Database\Grammar` wymaga teraz również instancji `Connection`. W poprzednich wersjach połączenie było przypisywane po konstrukcji za pomocą metody `setConnection()`. Ta metoda została usunięta w Laravel 12:

```php
// Laravel <= 11.x
$grammar = new MySqlGrammar;
$grammar->setConnection($connection);

// Laravel >= 12.x
$grammar = new MySqlGrammar($connection);
```

Ponadto następujące API zostały usunięte lub zdeprecjonowane:

<div class="content-list" markdown="1">

- Metoda `Blueprint::getPrefix()` jest zdeprecjonowana.
- Metoda `Connection::withTablePrefix()` została usunięta.
- Metody `Grammar::getTablePrefix()` i `setTablePrefix()` są zdeprecjonowane.
- Metoda `Grammar::setConnection()` została usunięta.

</div>

Pracując z prefiksami tabel, powinieneś teraz pobierać je bezpośrednio z połączenia z bazą danych:

```php
$prefix = $connection->getTablePrefix();
```

Jeśli utrzymujesz własne sterowniki baz danych, builderzy schematów lub implementacje gramatyki, powinieneś przejrzeć ich konstruktory i upewnić się, że instancja `Connection` jest dostarczana.

<a name="eloquent"></a>
### Eloquent

<a name="models-and-uuidv7"></a>
#### Modele i UUIDv7

**Prawdopodobieństwo Wpływu: Średnie**

Trait `HasUuids` zwraca teraz UUID, które są kompatybilne z wersją 7 specyfikacji UUID (uporządkowane UUID). Jeśli chcesz nadal używać uporządkowanych ciągów UUIDv4 dla ID Twojego modelu, powinieneś teraz użyć traitu `HasVersion4Uuids`:

```php
use Illuminate\Database\Eloquent\Concerns\HasUuids; // [tl! remove]
use Illuminate\Database\Eloquent\Concerns\HasVersion4Uuids as HasUuids; // [tl! add]
```

Trait `HasVersion7Uuids` został usunięty. Jeśli wcześniej używałeś tego traitu, powinieneś użyć zamiast niego traitu `HasUuids`, który teraz zapewnia to samo zachowanie.

<a name="requests"></a>
### Żądania

<a name="nested-array-request-merging"></a>
#### Scalanie Zagnieżdżonych Tablic Żądań

**Prawdopodobieństwo Wpływu: Niskie**

Metoda `$request->mergeIfMissing()` pozwala teraz na scalanie zagnieżdżonych danych tablicowych przy użyciu notacji "kropkowej". Jeśli wcześniej polegałeś na tej metodzie do tworzenia klucza tablicy najwyższego poziomu zawierającego wersję klucza w notacji "kropkowej", możesz potrzebować dostosować swą aplikację do tego nowego zachowania:

```php
$request->mergeIfMissing([
    'user.last_name' => 'Otwell',
]);
```

<a name="storage"></a>
### Przechowywanie

<a name="local-filesystem-disk-default-root-path"></a>
#### Domyślna Ścieżka Root Dysku Lokalnego Systemu Plików

**Prawdopodobieństwo Wpływu: Niskie**

Jeśli Twoja aplikacja nie definiuje wyraźnie dysku `local` w konfiguracji filesystems, Laravel użyje teraz domyślnie katalogu głównego dysku lokalnego `storage/app/private`. W poprzednich wersjach domyślnie było to `storage/app`. W rezultacie wywołania `Storage::disk('local')` będą odczytywały z i zapisywały do `storage/app/private`, chyba że skonfigurowano inaczej. Aby przywrócić poprzednie zachowanie, możesz zdefiniować dysk `local` ręcznie i ustawić żądaną ścieżkę główną.

<a name="validation"></a>
### Walidacja

<a name="image-validation"></a>
#### Walidacja Obrazów Teraz Wyklucza SVG

**Prawdopodobieństwo Wpływu: Niskie**

Reguła walidacji `image` nie pozwala już domyślnie na obrazy SVG. Jeśli chcesz zezwolić na SVG podczas używania reguły `image`, musisz wyraźnie na nie zezwolić:

```php
use Illuminate\Validation\Rules\File;

'photo' => 'required|image:allow_svg'

// Lub...
'photo' => ['required', File::image(allowSvg: true)],
```

<a name="miscellaneous"></a>
### Różne

Zachęcamy również do przeglądania zmian w [repozytorium GitHub](https://github.com/laravel/laravel) `laravel/laravel`. Choć wiele z tych zmian nie jest wymaganych, możesz chcieć zsynchronizować te pliki z Twoją aplikacją. Niektóre z tych zmian zostaną omawiane w tym przewodniku aktualizacji, ale inne, takie jak zmiany w plikach konfiguracyjnych lub komentarzach, nie będą. Możesz łatwo zobaczyć zmiany za pomocą [narzędzia porównawczego GitHub](https://github.com/laravel/laravel/compare/11.x...12.x) i wybrać, które aktualizacje są dla Ciebie ważne.
