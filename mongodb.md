# MongoDB

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
    - [Sterownik MongoDB](#mongodb-driver)
    - [Uruchamianie serwera MongoDB](#starting-a-mongodb-server)
    - [Instalacja pakietu Laravel MongoDB](#install-the-laravel-mongodb-package)
- [Konfiguracja](#configuration)
- [Funkcje](#features)

<a name="introduction"></a>
## Wprowadzenie

[MongoDB](https://www.mongodb.com/resources/products/fundamentals/why-use-mongodb) jest jedną z najpopularniejszych baz danych NoSQL zorientowanych na dokumenty, używaną ze względu na jej wysokie obciążenie zapisu (przydatne do analityki lub IoT) i wysoką dostępność (łatwe do ustawienia zestawy replik z automatycznym przełączaniem awaryjnym). Może również łatwo shardować bazę danych w celu zapewnienia skalowalności poziomej i ma potężny język zapytań do wykonywania agregacji, wyszukiwania tekstowego lub zapytań geoprzestrzennych.

Zamiast przechowywać dane w tabelach wierszy lub kolumn, jak bazy danych SQL, każdy rekord w bazie danych MongoDB jest dokumentem opisanym w BSON, binarnej reprezentacji danych. Aplikacje mogą następnie pobierać te informacje w formacie JSON. Obsługuje szeroką gamę typów danych, w tym dokumenty, tablice, dokumenty osadzone i dane binarne.

Przed użyciem MongoDB z Laravel zalecamy zainstalowanie i użycie pakietu `mongodb/laravel-mongodb` za pomocą Composera. Pakiet `laravel-mongodb` jest oficjalnie utrzymywany przez MongoDB i chociaż MongoDB jest natywnie obsługiwany przez PHP za pośrednictwem sterownika MongoDB, pakiet [Laravel MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/) zapewnia bogatszą integrację z Eloquent i innymi funkcjami Laravel:

```shell
composer require mongodb/laravel-mongodb
```

<a name="installation"></a>
## Instalacja

<a name="mongodb-driver"></a>
### Sterownik MongoDB

Aby połączyć się z bazą danych MongoDB, wymagane jest rozszerzenie PHP `mongodb`. Jeśli rozwijasz lokalnie za pomocą [Laravel Herd](https://herd.laravel.com) lub zainstalowałeś PHP przez `php.new`, masz już to rozszerzenie zainstalowane w swoim systemie. Jeśli jednak musisz zainstalować rozszerzenie ręcznie, możesz to zrobić za pomocą PECL:

```shell
pecl install mongodb
```

Aby uzyskać więcej informacji na temat instalacji rozszerzenia MongoDB PHP, sprawdź [instrukcje instalacji rozszerzenia MongoDB PHP](https://www.php.net/manual/en/mongodb.installation.php).

<a name="starting-a-mongodb-server"></a>
### Uruchamianie serwera MongoDB

MongoDB Community Server może być używany do lokalnego uruchomienia MongoDB i jest dostępny do instalacji na systemach Windows, macOS, Linux lub jako kontener Docker. Aby dowiedzieć się, jak zainstalować MongoDB, zapoznaj się z [oficjalnym przewodnikiem instalacji MongoDB Community](https://docs.mongodb.com/manual/administration/install-community/).

Łańcuch połączenia dla serwera MongoDB można ustawić w pliku `.env`:

```ini
MONGODB_URI="mongodb://localhost:27017"
MONGODB_DATABASE="laravel_app"
```

W przypadku hostowania MongoDB w chmurze rozważ użycie [MongoDB Atlas](https://www.mongodb.com/cloud/atlas).
Aby uzyskać dostęp do klastra MongoDB Atlas lokalnie z Twojej aplikacji, musisz [dodać swój własny adres IP w ustawieniach sieci klastra](https://www.mongodb.com/docs/atlas/security/add-ip-address-to-list/) do listy dostępu IP projektu.

Łańcuch połączenia dla MongoDB Atlas można również ustawić w pliku `.env`:

```ini
MONGODB_URI="mongodb+srv://<username>:<password>@<cluster>.mongodb.net/<dbname>?retryWrites=true&w=majority"
MONGODB_DATABASE="laravel_app"
```

<a name="install-the-laravel-mongodb-package"></a>
### Instalacja pakietu Laravel MongoDB

Na koniec użyj Composera, aby zainstalować pakiet Laravel MongoDB:

```shell
composer require mongodb/laravel-mongodb
```

> [!NOTE]
> Ta instalacja pakietu nie powiedzie się, jeśli rozszerzenie PHP `mongodb` nie jest zainstalowane. Konfiguracja PHP może się różnić między CLI a serwerem internetowym, więc upewnij się, że rozszerzenie jest włączone w obu konfiguracjach.

<a name="configuration"></a>
## Konfiguracja

Możesz skonfigurować połączenie MongoDB za pomocą pliku konfiguracyjnego `config/database.php` Twojej aplikacji. W tym pliku dodaj połączenie `mongodb`, które wykorzystuje sterownik `mongodb`:

```php
'connections' => [
    'mongodb' => [
        'driver' => 'mongodb',
        'dsn' => env('MONGODB_URI', 'mongodb://localhost:27017'),
        'database' => env('MONGODB_DATABASE', 'laravel_app'),
    ],
],
```

<a name="features"></a>
## Funkcje

Po zakończeniu konfiguracji możesz użyć pakietu i połączenia z bazą danych `mongodb` w swojej aplikacji, aby wykorzystać różne potężne funkcje:

- [Używanie Eloquent](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/eloquent-models/), modele mogą być przechowywane w kolekcjach MongoDB. Oprócz standardowych funkcji Eloquent, pakiet Laravel MongoDB zapewnia dodatkowe funkcje, takie jak relacje osadzone. Pakiet zapewnia również bezpośredni dostęp do sterownika MongoDB, który może być używany do wykonywania operacji takich jak surowe zapytania i potoki agregacji.
- [Pisanie złożonych zapytań](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/query-builder/) za pomocą konstruktora zapytań.
- [Sterownik cache](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/cache/) `mongodb` jest zoptymalizowany do używania funkcji MongoDB, takich jak indeksy TTL, aby automatycznie usuwać wygasłe wpisy cache.
- [Wysyłanie i przetwarzanie zadań w kolejce](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/queues/) za pomocą sterownika kolejki `mongodb`.
- [Przechowywanie plików w GridFS](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/filesystems/) za pomocą [adaptera GridFS dla Flysystem](https://flysystem.thephpleague.com/docs/adapter/gridfs/).
- Większość pakietów stron trzecich korzystających z połączenia z bazą danych lub Eloquent może być używana z MongoDB.

Aby kontynuować naukę korzystania z MongoDB i Laravel, zapoznaj się z [przewodnikiem Szybki start](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/quick-start/) MongoDB.
