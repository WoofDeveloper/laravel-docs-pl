# Przechowywanie plików

- [Wprowadzenie](#introduction)
- [Konfiguracja](#configuration)
    - [Sterownik lokalny](#the-local-driver)
    - [Dysk publiczny](#the-public-disk)
    - [Wymagania sterowników](#driver-prerequisites)
    - [Systemy plików z zakresem i tylko do odczytu](#scoped-and-read-only-filesystems)
    - [Systemy plików kompatybilne z Amazon S3](#amazon-s3-compatible-filesystems)
- [Pobieranie instancji dysków](#obtaining-disk-instances)
    - [Dyski na żądanie](#on-demand-disks)
- [Pobieranie plików](#retrieving-files)
    - [Pobieranie plików](#downloading-files)
    - [Adresy URL plików](#file-urls)
    - [Tymczasowe adresy URL](#temporary-urls)
    - [Metadane plików](#file-metadata)
- [Przechowywanie plików](#storing-files)
    - [Dodawanie na początku i na końcu plików](#prepending-appending-to-files)
    - [Kopiowanie i przenoszenie plików](#copying-moving-files)
    - [Automatyczne streamowanie](#automatic-streaming)
    - [Przesyłanie plików](#file-uploads)
    - [Widoczność plików](#file-visibility)
- [Usuwanie plików](#deleting-files)
- [Katalogi](#directories)
- [Testowanie](#testing)
- [Niestandardowe systemy plików](#custom-filesystems)

<a name="introduction"></a>
## Wprowadzenie

Laravel zapewnia potężną abstrakcję systemu plików dzięki wspaniałemu pakietowi PHP [Flysystem](https://github.com/thephpleague/flysystem) autorstwa Franka de Jonge. Integracja Laravel z Flysystem zapewnia proste sterowniki do pracy z lokalnymi systemami plików, SFTP i Amazon S3. Co więcej, jest niezwykle łatwo przełączać się między tymi opcjami przechowywania między lokalną maszyną programistyczną a serwerem produkcyjnym, ponieważ API pozostaje takie samo dla każdego systemu.

<a name="configuration"></a>
## Konfiguracja

Plik konfiguracyjny systemu plików Laravel znajduje się w `config/filesystems.php`. W tym pliku możesz skonfigurować wszystkie swoje "dyski" systemu plików. Każdy dysk reprezentuje konkretny sterownik przechowywania i lokalizację przechowywania. Przykładowe konfiguracje dla każdego obsługiwanego sterownika są zawarte w pliku konfiguracyjnym, więc możesz zmodyfikować konfigurację, aby odzwierciedlić swoje preferencje przechowywania i dane uwierzytelniające.

Sterownik `local` współdziała z plikami przechowywanymi lokalnie na serwerze, na którym działa aplikacja Laravel, podczas gdy sterownik `sftp` jest używany dla FTP opartego na kluczach SSH. Sterownik `s3` służy do zapisywania do usługi Amazon S3 w chmurze.

> [!NOTE]
> Możesz skonfigurować tyle dysków, ile chcesz, a nawet możesz mieć wiele dysków używających tego samego sterownika.

<a name="the-local-driver"></a>
### Sterownik lokalny

Podczas używania sterownika `local` wszystkie operacje na plikach są względne do katalogu `root` zdefiniowanego w pliku konfiguracyjnym `filesystems`. Domyślnie ta wartość jest ustawiona na katalog `storage/app/private`. Dlatego następująca metoda zapisałaby do `storage/app/private/example.txt`:

```php
use Illuminate\Support\Facades\Storage;

Storage::disk('local')->put('example.txt', 'Contents');
```

<a name="the-public-disk"></a>
### Dysk publiczny

Dysk `public` zawarty w pliku konfiguracyjnym `filesystems` Twojej aplikacji jest przeznaczony dla plików, które będą publicznie dostępne. Domyślnie dysk `public` używa sterownika `local` i przechowuje swoje pliki w `storage/app/public`.

Jeśli Twój dysk `public` używa sterownika `local` i chcesz, aby te pliki były dostępne z sieci web, powinieneś utworzyć dowiązanie symboliczne z katalogu źródłowego `storage/app/public` do katalogu docelowego `public/storage`:

Aby utworzyć dowiązanie symboliczne, możesz użyć komendy Artisan `storage:link`:

```shell
php artisan storage:link
```

Po zapisaniu pliku i utworzeniu dowiązania symbolicznego możesz utworzyć URL do plików za pomocą helpera `asset`:

```php
echo asset('storage/file.txt');
```

Możesz skonfigurować dodatkowe dowiązania symboliczne w pliku konfiguracyjnym `filesystems`. Każde ze skonfigurowanych linków zostanie utworzone po uruchomieniu komendy `storage:link`:

```php
'links' => [
    public_path('storage') => storage_path('app/public'),
    public_path('images') => storage_path('app/images'),
],
```

Komenda `storage:unlink` może być użyta do usunięcia skonfigurowanych dowiązań symbolicznych:

```shell
php artisan storage:unlink
```

<a name="driver-prerequisites"></a>
### Wymagania sterowników

<a name="s3-driver-configuration"></a>
#### Konfiguracja sterownika S3

Przed użyciem sterownika S3 musisz zainstalować pakiet Flysystem S3 za pomocą menedżera pakietów Composer:

```shell
composer require league/flysystem-aws-s3-v3 "^3.0" --with-all-dependencies
```

Tablica konfiguracyjna dysku S3 znajduje się w pliku konfiguracyjnym `config/filesystems.php`. Zazwyczaj powinieneś skonfigurować swoje informacje S3 i dane uwierzytelniające za pomocą następujących zmiennych środowiskowych, do których odwołuje się plik konfiguracyjny `config/filesystems.php`:

```ini
AWS_ACCESS_KEY_ID=<your-key-id>
AWS_SECRET_ACCESS_KEY=<your-secret-access-key>
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=<your-bucket-name>
AWS_USE_PATH_STYLE_ENDPOINT=false
```

Dla wygody te zmienne środowiskowe odpowiadają konwencji nazewnictwa używanej przez AWS CLI.

<a name="ftp-driver-configuration"></a>
#### Konfiguracja sterownika FTP

Przed użyciem sterownika FTP musisz zainstalować pakiet Flysystem FTP za pomocą menedżera pakietów Composer:

```shell
composer require league/flysystem-ftp "^3.0"
```

Integracje Flysystem w Laravel świetnie współpracują z FTP; jednak przykładowa konfiguracja nie jest zawarta w domyślnym pliku konfiguracyjnym `config/filesystems.php` frameworka. Jeśli potrzebujesz skonfigurować system plików FTP, możesz użyć poniższego przykładu konfiguracji:

```php
'ftp' => [
    'driver' => 'ftp',
    'host' => env('FTP_HOST'),
    'username' => env('FTP_USERNAME'),
    'password' => env('FTP_PASSWORD'),

    // Opcjonalne ustawienia FTP...
    // 'port' => env('FTP_PORT', 21),
    // 'root' => env('FTP_ROOT'),
    // 'passive' => true,
    // 'ssl' => true,
    // 'timeout' => 30,
],
```

<a name="sftp-driver-configuration"></a>
#### Konfiguracja sterownika SFTP

Przed użyciem sterownika SFTP musisz zainstalować pakiet Flysystem SFTP za pomocą menedżera pakietów Composer:

```shell
composer require league/flysystem-sftp-v3 "^3.0"
```

Integracje Flysystem w Laravel świetnie współpracują z SFTP; jednak przykładowa konfiguracja nie jest zawarta w domyślnym pliku konfiguracyjnym `config/filesystems.php` frameworka. Jeśli potrzebujesz skonfigurować system plików SFTP, możesz użyć poniższego przykładu konfiguracji:

```php
'sftp' => [
    'driver' => 'sftp',
    'host' => env('SFTP_HOST'),

    // Ustawienia dla podstawowego uwierzytelniania...
    'username' => env('SFTP_USERNAME'),
    'password' => env('SFTP_PASSWORD'),

    // Ustawienia dla uwierzytelniania opartego na kluczu SSH z hasłem szyfrowania...
    'privateKey' => env('SFTP_PRIVATE_KEY'),
    'passphrase' => env('SFTP_PASSPHRASE'),

    // Ustawienia dla uprawnień plików / katalogów...
    'visibility' => 'private', // `private` = 0600, `public` = 0644
    'directory_visibility' => 'private', // `private` = 0700, `public` = 0755

    // Opcjonalne ustawienia SFTP...
    // 'hostFingerprint' => env('SFTP_HOST_FINGERPRINT'),
    // 'maxTries' => 4,
    // 'passphrase' => env('SFTP_PASSPHRASE'),
    // 'port' => env('SFTP_PORT', 22),
    // 'root' => env('SFTP_ROOT', ''),
    // 'timeout' => 30,
    // 'useAgent' => true,
],
```

<a name="scoped-and-read-only-filesystems"></a>
### Systemy plików z zakresem i tylko do odczytu

Dyski z zakresem pozwalają zdefiniować system plików, w którym wszystkie ścieżki są automatycznie poprzedzane danym prefiksem ścieżki. Przed utworzeniem dysku z zakresem musisz zainstalować dodatkowy pakiet Flysystem za pomocą menedżera pakietów Composer:

```shell
composer require league/flysystem-path-prefixing "^3.0"
```

Możesz utworzyć instancję o zakresie ścieżki dowolnego istniejącego dysku systemu plików, definiując dysk wykorzystujący sterownik `scoped`. Na przykład możesz utworzyć dysk, który zakresuje istniejący dysk `s3` do określonego prefiksu ścieżki, a następnie każda operacja na plikach używająca dysku z zakresem będzie używać określonego prefiksu:

```php
's3-videos' => [
    'driver' => 'scoped',
    'disk' => 's3',
    'prefix' => 'path/to/videos',
],
```

Dyski "tylko do odczytu" pozwalają tworzyć dyski systemu plików, które nie pozwalają na operacje zapisu. Przed użyciem opcji konfiguracyjnej `read-only` musisz zainstalować dodatkowy pakiet Flysystem za pomocą menedżera pakietów Composer:

```shell
composer require league/flysystem-read-only "^3.0"
```

Następnie możesz dodać opcję konfiguracyjną `read-only` do jednej lub więcej tablic konfiguracyjnych dysku:

```php
's3-videos' => [
    'driver' => 's3',
    // ...
    'read-only' => true,
],
```

<a name="amazon-s3-compatible-filesystems"></a>
### Systemy plików kompatybilne z Amazon S3

Domyślnie plik konfiguracyjny `filesystems` Twojej aplikacji zawiera konfigurację dysku dla dysku `s3`. Oprócz używania tego dysku do interakcji z [Amazon S3](https://aws.amazon.com/s3/), możesz go używać do interakcji z dowolną usługą przechowywania plików kompatybilną z S3, taką jak [RustFS](https://github.com/rustfs/rustfs), [DigitalOcean Spaces](https://www.digitalocean.com/products/spaces/), [Vultr Object Storage](https://www.vultr.com/products/object-storage/), [Cloudflare R2](https://www.cloudflare.com/developer-platform/products/r2/) lub [Hetzner Cloud Storage](https://www.hetzner.com/storage/object-storage/).

Zazwyczaj po zaktualizowaniu danych uwierzytelniających dysku, aby pasowały do danych uwierzytelniających usługi, której planujesz używać, wystarczy zaktualizować wartość opcji konfiguracyjnej `endpoint`. Wartość tej opcji jest zazwyczaj definiowana za pomocą zmiennej środowiskowej `AWS_ENDPOINT`:

```php
'endpoint' => env('AWS_ENDPOINT', 'https://rustfs:9000'),
```

<a name="obtaining-disk-instances"></a>
## Pobieranie instancji dysków

Fasada `Storage` może być używana do interakcji z dowolnym ze skonfigurowanych dysków. Na przykład możesz użyć metody `put` na fasadzie, aby zapisać awatar na domyślnym dysku. Jeśli wywołasz metody na fasadzie `Storage` bez wcześniejszego wywołania metody `disk`, metoda zostanie automatycznie przekazana do domyślnego dysku:

```php
use Illuminate\Support\Facades\Storage;

Storage::put('avatars/1', $content);
```

Jeśli Twoja aplikacja współdziała z wieloma dyskami, możesz użyć metody `disk` na fasadzie `Storage`, aby pracować z plikami na konkretnym dysku:

```php
Storage::disk('s3')->put('avatars/1', $content);
```

<a name="on-demand-disks"></a>
### Dyski na żądanie

Czasami możesz chcieć utworzyć dysk w czasie wykonywania przy użyciu danej konfiguracji bez tej konfiguracji faktycznie obecnej w pliku konfiguracyjnym `filesystems` Twojej aplikacji. Aby to osiągnąć, możesz przekazać tablicę konfiguracyjną do metody `build` fasady `Storage`:

```php
use Illuminate\Support\Facades\Storage;

$disk = Storage::build([
    'driver' => 'local',
    'root' => '/path/to/root',
]);

$disk->put('image.jpg', $content);
```

<a name="retrieving-files"></a>
## Pobieranie plików

Metoda `get` może być użyta do pobrania zawartości pliku. Surowa zawartość tekstowa pliku zostanie zwrócona przez metodę. Pamiętaj, że wszystkie ścieżki plików powinny być określone względem lokalizacji "root" dysku:

```php
$contents = Storage::get('file.jpg');
```

Jeśli pobierany plik zawiera JSON, możesz użyć metody `json`, aby pobrać plik i zdekodować jego zawartość:

```php
$orders = Storage::json('orders.json');
```

Metoda `exists` może być użyta do określenia, czy plik istnieje na dysku:

```php
if (Storage::disk('s3')->exists('file.jpg')) {
    // ...
}
```

Metoda `missing` może być użyta do określenia, czy plik brakuje na dysku:

```php
if (Storage::disk('s3')->missing('file.jpg')) {
    // ...
}
```

<a name="downloading-files"></a>
### Pobieranie plików

Metoda `download` może być użyta do wygenerowania odpowiedzi, która wymusza na przeglądarce użytkownika pobranie pliku pod daną ścieżką. Metoda `download` akceptuje nazwę pliku jako drugi argument metody, który określi nazwę pliku widoczną dla użytkownika pobierającego plik. Na koniec możesz przekazać tablicę nagłówków HTTP jako trzeci argument metody:

```php
return Storage::download('file.jpg');

return Storage::download('file.jpg', $name, $headers);
```

<a name="file-urls"></a>
### Adresy URL plików

Możesz użyć metody `url`, aby uzyskać URL dla danego pliku. Jeśli używasz sterownika `local`, zazwyczaj po prostu dodaje to `/storage` do podanej ścieżki i zwraca względny URL do pliku. Jeśli używasz sterownika `s3`, zostanie zwrócony pełny zdalny URL:

```php
use Illuminate\Support\Facades\Storage;

$url = Storage::url('file.jpg');
```

Podczas używania sterownika `local` wszystkie pliki, które powinny być publicznie dostępne, powinny zostać umieszczone w katalogu `storage/app/public`. Ponadto powinieneś [utworzyć dowiązanie symboliczne](#the-public-disk) w `public/storage`, które wskazuje na katalog `storage/app/public`.

> [!WARNING]
> Podczas używania sterownika `local` wartość zwracana przez `url` nie jest zakodowana w URL. Z tego powodu zalecamy zawsze przechowywanie plików przy użyciu nazw, które utworzą prawidłowe adresy URL.

<a name="url-host-customization"></a>
#### Dostosowanie hosta URL

Jeśli chcesz zmodyfikować hosta dla adresów URL generowanych za pomocą fasady `Storage`, możesz dodać lub zmienić opcję `url` w tablicy konfiguracyjnej dysku:

```php
'public' => [
    'driver' => 'local',
    'root' => storage_path('app/public'),
    'url' => env('APP_URL').'/storage',
    'visibility' => 'public',
    'throw' => false,
],
```

<a name="temporary-urls"></a>
### Tymczasowe adresy URL

Używając metody `temporaryUrl`, możesz tworzyć tymczasowe adresy URL do plików przechowywanych za pomocą sterowników `local` i `s3`. Ta metoda akceptuje ścieżkę i instancję `DateTime` określającą, kiedy URL powinien wygasł:

```php
use Illuminate\Support\Facades\Storage;

$url = Storage::temporaryUrl(
    'file.jpg', now()->plus(minutes: 5)
);
```

<a name="enabling-local-temporary-urls"></a>
#### Włączanie lokalnych tymczasowych adresów URL

Jeśli zacząłeś rozwijać swoją aplikację przed wprowadzeniem obsługi tymczasowych adresów URL do sterownika `local`, możesz potrzebować włączyć lokalne tymczasowe adresy URL. Aby to zrobić, dodaj opcję `serve` do tablicy konfiguracyjnej dysku `local` w pliku konfiguracyjnym `config/filesystems.php`:

```php
'local' => [
    'driver' => 'local',
    'root' => storage_path('app/private'),
    'serve' => true, // [tl! add]
    'throw' => false,
],
```

<a name="s3-request-parameters"></a>
#### Parametry żądania S3

Jeśli potrzebujesz określić dodatkowe [parametry żądania S3](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectGET.html#RESTObjectGET-requests), możesz przekazać tablicę parametrów żądania jako trzeci argument do metody `temporaryUrl`:

```php
$url = Storage::temporaryUrl(
    'file.jpg',
    now()->plus(minutes: 5),
    [
        'ResponseContentType' => 'application/octet-stream',
        'ResponseContentDisposition' => 'attachment; filename=file2.jpg',
    ]
);
```

<a name="customizing-temporary-urls"></a>
#### Dostosowywanie tymczasowych adresów URL

Jeśli potrzebujesz dostosować sposób tworzenia tymczasowych adresów URL dla określonego dysku przechowywania, możesz użyć metody `buildTemporaryUrlsUsing`. Na przykład może to być przydatne, jeśli masz kontroler, który pozwala pobierać pliki przechowywane na dysku, który zazwyczaj nie obsługuje tymczasowych adresów URL. Zazwyczaj ta metoda powinna być wywołana z metody `boot` dostawcy usług:

```php
<?php

namespace App\Providers;

use DateTime;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Facades\URL;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Uruchom dowolne usługi aplikacji.
     */
    public function boot(): void
    {
        Storage::disk('local')->buildTemporaryUrlsUsing(
            function (string $path, DateTime $expiration, array $options) {
                return URL::temporarySignedRoute(
                    'files.download',
                    $expiration,
                    array_merge($options, ['path' => $path])
                );
            }
        );
    }
}
```

<a name="temporary-upload-urls"></a>
#### Tymczasowe adresy URL przesyłania

> [!WARNING]
> Możliwość generowania tymczasowych adresów URL przesyłania jest obsługiwana tylko przez sterownik `s3`.

Jeśli potrzebujesz wygenerować tymczasowy URL, który może być użyty do przesłania pliku bezpośrednio z aplikacji po stronie klienta, możesz użyć metody `temporaryUploadUrl`. Ta metoda akceptuje ścieżkę i instancję `DateTime` określającą, kiedy URL powinien wygasł. Metoda `temporaryUploadUrl` zwraca tablicę asocjacyjną, która może zostać zdestrukturyzowana na URL przesyłania i nagłówki, które powinny zostać dołączone do żądania przesyłania:

```php
use Illuminate\Support\Facades\Storage;

['url' => $url, 'headers' => $headers] = Storage::temporaryUploadUrl(
    'file.jpg', now()->plus(minutes: 5)
);
```

Ta metoda jest przede wszystkim przydatna w środowiskach bezserwerowych, które wymagają, aby aplikacja po stronie klienta bezpośrednio przesyłała pliki do systemu przechowywania w chmurze, takiego jak Amazon S3.

<a name="file-metadata"></a>
### Metadane plików

Oprócz odczytu i zapisu plików, Laravel może również dostarczać informacji o samych plikach. Na przykład metoda `size` może być użyta do uzyskania rozmiaru pliku w bajtach:

```php
use Illuminate\Support\Facades\Storage;

$size = Storage::size('file.jpg');
```

Metoda `lastModified` zwraca znacznik czasu UNIX ostatniej modyfikacji pliku:

```php
$time = Storage::lastModified('file.jpg');
```

Typ MIME danego pliku można uzyskać za pomocą metody `mimeType`:

```php
$mime = Storage::mimeType('file.jpg');
```

<a name="file-paths"></a>
#### Ścieżki plików

Możesz użyć metody `path`, aby uzyskać ścieżkę dla danego pliku. Jeśli używasz sterownika `local`, zwróci to bezwzględną ścieżkę do pliku. Jeśli używasz sterownika `s3`, ta metoda zwróci względną ścieżkę do pliku w buckecie S3:

```php
use Illuminate\Support\Facades\Storage;

$path = Storage::path('file.jpg');
```

<a name="storing-files"></a>
## Przechowywanie plików

Metoda `put` może być użyta do przechowywania zawartości plików na dysku. Możesz również przekazać zasob PHP (`resource`) do metody `put`, która użyje bazowej obsługi strumieni Flysystem. Pamiętaj, że wszystkie ścieżki plików powinny być określone względem lokalizacji "root" skonfigurowanej dla dysku:

```php
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents);

Storage::put('file.jpg', $resource);
```

<a name="failed-writes"></a>
#### Nieudane zapisy

Jeśli metoda `put` (lub inne operacje "zapisu") nie może zapisać pliku na dysku, zostanie zwrócona wartość `false`:

```php
if (! Storage::put('file.jpg', $contents)) {
    // Plik nie mógł zostać zapisany na dysku...
}
```

Jeśli chcesz, możesz zdefiniować opcję `throw` w tablicy konfiguracyjnej dysku systemu plików. Gdy ta opcja jest zdefiniowana jako `true`, metody "zapisu", takie jak `put`, zrzucą instancję `League\Flysystem\UnableToWriteFile`, gdy operacje zapisu zawiodą:

```php
'public' => [
    'driver' => 'local',
    // ...
    'throw' => true,
],
```

<a name="prepending-appending-to-files"></a>
### Dodawanie na początku i na końcu plików

Metody `prepend` i `append` pozwalają zapisać na początku lub na końcu pliku:

```php
Storage::prepend('file.log', 'Prepended Text');

Storage::append('file.log', 'Appended Text');
```

<a name="copying-moving-files"></a>
### Kopiowanie i przenoszenie plików

Metoda `copy` może być użyta do skopiowania istniejącego pliku do nowej lokalizacji na dysku, podczas gdy metoda `move` może być użyta do zmiany nazwy lub przeniesienia istniejącego pliku do nowej lokalizacji:

```php
Storage::copy('old/file.jpg', 'new/file.jpg');

Storage::move('old/file.jpg', 'new/file.jpg');
```

<a name="automatic-streaming"></a>
### Automatyczne streamowanie

Streamowanie plików do przechowywania oferuje znacznie zmniejszone zużycie pamięci. Jeśli chcesz, aby Laravel automatycznie zarządzał streamowaniem danego pliku do Twojej lokalizacji przechowywania, możesz użyć metody `putFile` lub `putFileAs`. Ta metoda akceptuje instancję `Illuminate\Http\File` lub `Illuminate\Http\UploadedFile` i automatycznie przestreamuje plik do żądanej lokalizacji:

```php
use Illuminate\Http\File;
use Illuminate\Support\Facades\Storage;

// Automatycznie wygeneruj unikalny identyfikator dla nazwy pliku...
$path = Storage::putFile('photos', new File('/path/to/photo'));

// Ręcznie określ nazwę pliku...
$path = Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');
```

Est kilka ważnych rzeczy do zanotowania na temat metody `putFile`. Zauważ, że określiliśmy tylko nazwę katalogu, a nie nazwę pliku. Domyślnie metoda `putFile` wygeneruje unikalny identyfikator, który będzie służyć jako nazwa pliku. Rozszerzenie pliku zostanie określone poprzez zbadanie typu MIME pliku. Ścieżka do pliku zostanie zwrócona przez metodę `putFile`, więc możesz zapisać ścieżkę, w tym wygenerowanej nazwie pliku, w bazie danych.

Metody `putFile` i `putFileAs` akceptują również argument do określenia "widoczności" przechowywanego pliku. Jest to szczególnie przydatne, jeśli przechowujesz plik na dysku w chmurze, takim jak Amazon S3, i chcesz, aby plik był publicznie dostępny za pośrednictwem generowanych URL:

```php
Storage::putFile('photos', new File('/path/to/photo'), 'public');
```

<a name="file-uploads"></a>
### Przesyłanie plików

W aplikacjach internetowych jednym z najczęstszych przypadków użycia przechowywania plików jest przechowywanie plików przesyłanych przez użytkowników, takich jak zdjęcia i dokumenty. Laravel znacznie ułatwia przechowywanie przesłanych plików za pomocą metody `store` na instancji przesłanego pliku. Wywołaj metodę `store` ze ścieżką, pod którą chcesz przechowywać przesłany plik:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserAvatarController extends Controller
{
    /**
     * Zaktualizuj awatar użytkownika.
     */
    public function update(Request $request): string
    {
        $path = $request->file('avatar')->store('avatars');

        return $path;
    }
}
```

Est kilka ważnych rzeczy do zanotowania o tym przykładzie. Zauważ, że określiliśmy tylko nazwę katalogu, a nie nazwę pliku. Domyślnie metoda `store` wygeneruje unikalny identyfikator, który będzie służyć jako nazwa pliku. Rozszerzenie pliku zostanie określone poprzez zbadanie typu MIME pliku. Ścieżka do pliku zostanie zwrócona przez metodę `store`, więc możesz zapisać ścieżkę, w tym wygenerowanej nazwie pliku, w bazie danych.

Możesz również wywołać metodę `putFile` na fasadzie `Storage`, aby wykonać tę samą operację przechowywania plików, co w powyższym przykładzie:

```php
$path = Storage::putFile('avatars', $request->file('avatar'));
```

<a name="specifying-a-file-name"></a>
#### Określanie nazwy pliku

Jeśli nie chcesz, aby nazwa pliku była automatycznie przypisywana do przechowywanego pliku, możesz użyć metody `storeAs`, która otrzymuje ścieżkę, nazwę pliku i (opcjonalnie) dysk jako swoje argumenty:

```php
$path = $request->file('avatar')->storeAs(
    'avatars', $request->user()->id
);
```

Możesz również użyć metody `putFileAs` na fasadzie `Storage`, która wykona tę samą operację przechowywania plików, co powyższy przykład:

```php
$path = Storage::putFileAs(
    'avatars', $request->file('avatar'), $request->user()->id
);
```

> [!WARNING]
> Niedrukowalne i nieprawidłowe znaki unicode zostaną automatycznie usunięte ze ścieżek plików. Dlatego możesz chcieć oczyścić ścieżki plików przed przekazaniem ich do metod przechowywania plików Laravel. Ścieżki plików są normalizowane przy użyciu metody `League\Flysystem\WhitespacePathNormalizer::normalizePath`.

<a name="specifying-a-disk"></a>
#### Określanie dysku

Domyślnie metoda `store` przesłanego pliku użyje Twojego domyślnego dysku. Jeśli chcesz określić inny dysk, przekazanie nazwy dysku jako drugi argument do metody `store`:

```php
$path = $request->file('avatar')->store(
    'avatars/'.$request->user()->id, 's3'
);
```

Jeśli używasz metody `storeAs`, możesz przekazać nazwę dysku jako trzeci argument do metody:

```php
$path = $request->file('avatar')->storeAs(
    'avatars',
    $request->user()->id,
    's3'
);
```

<a name="other-uploaded-file-information"></a>
#### Inne informacje o przesłanym pliku

Jeśli chcesz uzyskać oryginalną nazwę i rozszerzenie przesłanego pliku, możesz to zrobić za pomocą metod `getClientOriginalName` i `getClientOriginalExtension`:

```php
$file = $request->file('avatar');

$name = $file->getClientOriginalName();
$extension = $file->getClientOriginalExtension();
```

Jednak pamiętaj, że metody `getClientOriginalName` i `getClientOriginalExtension` są uznawane za niebezpieczne, ponieważ nazwa pliku i rozszerzenie mogą zostać zmanipulowane przez złośliwego użytkownika. Z tego powodu zazwyczaj powinieneś preferować metody `hashName` i `extension`, aby uzyskać nazwę i rozszerzenie dla danego przesyłania pliku:

```php
$file = $request->file('avatar');

$name = $file->hashName(); // Wygeneruj unikalną, losową nazwę...
$extension = $file->extension(); // Określ rozszerzenie pliku na podstawie typu MIME pliku...
```

<a name="file-visibility"></a>
### Widoczność plików

W integracji Laravel z Flysystem "widoczność" jest abstrakcją uprawnień plików na wielu platformach. Pliki mogą zostać zadeklarowane jako `public` lub `private`. Gdy plik jest zadeklarowany jako `public`, wskazujesz, że plik powinien być ogólnie dostępny dla innych. Na przykład, gdy używasz sterownika S3, możesz pobierać adresy URL dla plików `public`.

Możesz ustawić widoczność podczas zapisywania pliku za pomocą metody `put`:

```php
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents, 'public');
```

Jeśli plik został już zapisany, jego widoczność może zostać pobrana i ustawiona za pomocą metod `getVisibility` i `setVisibility`:

```php
$visibility = Storage::getVisibility('file.jpg');

Storage::setVisibility('file.jpg', 'public');
```

Podczas interakcji z przesyłanymi plikami możesz użyć metod `storePublicly` i `storePubliclyAs`, aby przechowywać przesłany plik z widocznością `public`:

```php
$path = $request->file('avatar')->storePublicly('avatars', 's3');

$path = $request->file('avatar')->storePubliclyAs(
    'avatars',
    $request->user()->id,
    's3'
);
```

<a name="local-files-and-visibility"></a>
#### Lokalne pliki i widoczność

Podczas używania sterownika `local`, [widoczność](#file-visibility) `public` transluje się na uprawnienia `0755` dla katalogów i `0644` dla plików. Możesz zmodyfikować mapowania uprawnień w pliku konfiguracyjnym `filesystems` Twojej aplikacji:

```php
'local' => [
    'driver' => 'local',
    'root' => storage_path('app'),
    'permissions' => [
        'file' => [
            'public' => 0644,
            'private' => 0600,
        ],
        'dir' => [
            'public' => 0755,
            'private' => 0700,
        ],
    ],
    'throw' => false,
],
```

<a name="deleting-files"></a>
## Usuwanie plików

Metoda `delete` akceptuje pojedynczą nazwę pliku lub tablicę plików do usunięcia:

```php
use Illuminate\Support\Facades\Storage;

Storage::delete('file.jpg');

Storage::delete(['file.jpg', 'file2.jpg']);
```

Jeśli to konieczne, możesz określić dysk, z którego plik powinien zostać usunięty:

```php
use Illuminate\Support\Facades\Storage;

Storage::disk('s3')->delete('path/file.jpg');
```

<a name="directories"></a>
## Katalogi

<a name="get-all-files-within-a-directory"></a>
#### Pobierz wszystkie pliki w katalogu

Metoda `files` zwraca tablicę wszystkich plików w danym katalogu. Jeśli chcesz pobrać listę wszystkich plików w danym katalogu, w tym podkatalogów, możesz użyć metody `allFiles`:

```php
use Illuminate\Support\Facades\Storage;

$files = Storage::files($directory);

$files = Storage::allFiles($directory);
```

<a name="get-all-directories-within-a-directory"></a>
#### Pobierz wszystkie katalogi w katalogu

Metoda `directories` zwraca tablicę wszystkich katalogów w danym katalogu. Jeśli chcesz pobrać listę wszystkich katalogów w danym katalogu, w tym podkatalogów, możesz użyć metody `allDirectories`:

```php
$directories = Storage::directories($directory);

$directories = Storage::allDirectories($directory);
```

<a name="create-a-directory"></a>
#### Utwórz katalog

Metoda `makeDirectory` utworzy dany katalog, w tym wszystkie wymagane podkatalogi:

```php
Storage::makeDirectory($directory);
```

<a name="delete-a-directory"></a>
#### Usuń katalog

Na koniec metoda `deleteDirectory` może być użyta do usunięcia katalogu i wszystkich jego plików:

```php
Storage::deleteDirectory($directory);
```

<a name="testing"></a>
## Testowanie

Metoda `fake` fasady `Storage` pozwala łatwo wygenerować fałszywy dysk, który, połączony z narzędziami generowania plików klasy `Illuminate\Http\UploadedFile`, znacznie upraszcza testowanie przesyłania plików. Na przykład:

```php tab=Pest
<?php

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;

test('albums can be uploaded', function () {
    Storage::fake('photos');

    $response = $this->json('POST', '/photos', [
        UploadedFile::fake()->image('photo1.jpg'),
        UploadedFile::fake()->image('photo2.jpg')
    ]);

    // Sprawdź, czy jeden lub więcej plików zostało zapisanych...
    Storage::disk('photos')->assertExists('photo1.jpg');
    Storage::disk('photos')->assertExists(['photo1.jpg', 'photo2.jpg']);

    // Sprawdź, czy jeden lub więcej plików nie zostało zapisanych...
    Storage::disk('photos')->assertMissing('missing.jpg');
    Storage::disk('photos')->assertMissing(['missing.jpg', 'non-existing.jpg']);

    // Sprawdź, czy liczba plików w danym katalogu odpowiada oczekiwanej liczbie...
    Storage::disk('photos')->assertCount('/wallpapers', 2);

    // Sprawdź, czy dany katalog jest pusty...
    Storage::disk('photos')->assertDirectoryEmpty('/wallpapers');
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_albums_can_be_uploaded(): void
    {
        Storage::fake('photos');

        $response = $this->json('POST', '/photos', [
            UploadedFile::fake()->image('photo1.jpg'),
            UploadedFile::fake()->image('photo2.jpg')
        ]);

        // Sprawdź, czy jeden lub więcej plików zostało zapisanych...
        Storage::disk('photos')->assertExists('photo1.jpg');
        Storage::disk('photos')->assertExists(['photo1.jpg', 'photo2.jpg']);

        // Sprawdź, czy jeden lub więcej plików nie zostało zapisanych...
        Storage::disk('photos')->assertMissing('missing.jpg');
        Storage::disk('photos')->assertMissing(['missing.jpg', 'non-existing.jpg']);

        // Sprawdź, czy liczba plików w danym katalogu odpowiada oczekiwanej liczbie...
        Storage::disk('photos')->assertCount('/wallpapers', 2);

        // Sprawdź, czy dany katalog jest pusty...
        Storage::disk('photos')->assertDirectoryEmpty('/wallpapers');
    }
}
```

Domyślnie metoda `fake` usunie wszystkie pliki w swoim katalogu tymczasowym. Jeśli chcesz zachować te pliki, możesz użyć metody "persistentFake". Więcej informacji na temat testowania przesyłania plików możesz znaleźć w [dokumentacji testowania HTTP na temat przesyłania plików](/docs/{{version}}/http-tests#testing-file-uploads).

> [!WARNING]
> Metoda `image` wymaga [rozszerzenia GD](https://www.php.net/manual/en/book.image.php).

<a name="custom-filesystems"></a>
## Niestandardowe systemy plików

Integracja Laravel z Flysystem zapewnia obsługę kilku "sterowników" od razu; jednak Flysystem nie jest ograniczony do tych i ma adaptery dla wielu innych systemów przechowywania. Możesz utworzyć niestandardowy sterownik, jeśli chcesz użyć jednego z tych dodatkowych adapterów w swojej aplikacji Laravel.

Aby zdefiniować niestandardowy system plików, będziesz potrzebować adaptera Flysystem. Dodajmy utrzymywany przez społeczność adapter Dropbox do naszego projektu:

```shell
composer require spatie/flysystem-dropbox
```

Następnie możesz zarejestrować sterownik w metodzie `boot` jednego z [dostawców usług](/docs/{{version}}/providers) Twojej aplikacji. Aby to osiągnąć, powinieneś użyć metody `extend` fasady `Storage`:

```php
<?php

namespace App\Providers;

use Illuminate\Contracts\Foundation\Application;
use Illuminate\Filesystem\FilesystemAdapter;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\ServiceProvider;
use League\Flysystem\Filesystem;
use Spatie\Dropbox\Client as DropboxClient;
use Spatie\FlysystemDropbox\DropboxAdapter;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Zarejestruj dowolne usługi aplikacji.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Uruchom dowolne usługi aplikacji.
     */
    public function boot(): void
    {
        Storage::extend('dropbox', function (Application $app, array $config) {
            $adapter = new DropboxAdapter(new DropboxClient(
                $config['authorization_token']
            ));

            return new FilesystemAdapter(
                new Filesystem($adapter, $config),
                $adapter,
                $config
            );
        });
    }
}
```

Pierwszy argument metody `extend` to nazwa sterownika, a drugi to closure, który otrzymuje zmienne `$app` i `$config`. Closure musi zwrócić instancję `Illuminate\Filesystem\FilesystemAdapter`. Zmienna `$config` zawiera wartości zdefiniowane w `config/filesystems.php` dla określonego dysku.

Po utworzeniu i zarejestrowaniu dostawcy usług rozszerzenia możesz użyć sterownika `dropbox` w pliku konfiguracyjnym `config/filesystems.php`.
