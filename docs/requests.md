# Żądania HTTP

- [Wprowadzenie](#introduction)
- [Interakcja z żądaniem](#interacting-with-the-request)
    - [Dostęp do żądania](#accessing-the-request)
    - [Ścieżka, host i metoda żądania](#request-path-and-method)
    - [Nagłówki żądania](#request-headers)
    - [Adres IP żądania](#request-ip-address)
    - [Negocjacja zawartości](#content-negotiation)
    - [Żądania PSR-7](#psr7-requests)
- [Dane wejściowe](#input)
    - [Pobieranie danych wejściowych](#retrieving-input)
    - [Obecność danych wejściowych](#input-presence)
    - [Łączenie dodatkowych danych wejściowych](#merging-additional-input)
    - [Stare dane wejściowe](#old-input)
    - [Ciasteczka](#cookies)
    - [Przycinanie i normalizacja danych wejściowych](#input-trimming-and-normalization)
- [Pliki](#files)
    - [Pobieranie przesłanych plików](#retrieving-uploaded-files)
    - [Przechowywanie przesłanych plików](#storing-uploaded-files)
- [Konfiguracja zaufanych proxy](#configuring-trusted-proxies)
- [Konfiguracja zaufanych hostów](#configuring-trusted-hosts)

<a name="introduction"></a>
## Wprowadzenie

Klasa `Illuminate\Http\Request` w Laravel zapewnia obiektowy sposób interakcji z bieżącym żądaniem HTTP obsługiwanym przez twoją aplikację, a także pobierania danych wejściowych, ciasteczek i plików, które zostały przesłane wraz z żądaniem.

<a name="interacting-with-the-request"></a>
## Interakcja z żądaniem

<a name="accessing-the-request"></a>
### Dostęp do żądania

Aby uzyskać instancję bieżącego żądania HTTP poprzez wstrzykiwanie zależności, powinieneś dodać type-hint klasy `Illuminate\Http\Request` do twojego domknięcia trasy lub metody kontrolera. Instancja przychodzącego żądania zostanie automatycznie wstrzyknięta przez [kontener usług](/docs/{{version}}/container) Laravel:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Store a new user.
     */
    public function store(Request $request): RedirectResponse
    {
        $name = $request->input('name');

        // Store the user...

        return redirect('/users');
    }
}
```

Jak wspomniano, możesz również dodać type-hint klasy `Illuminate\Http\Request` do domknięcia trasy. Kontener usług automatycznie wstrzyknie przychodzące żądanie do domknięcia, gdy zostanie ono wykonane:

```php
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    // ...
});
```

<a name="dependency-injection-route-parameters"></a>
#### Wstrzykiwanie zależności i parametry trasy

Jeśli twoja metoda kontrolera również oczekuje danych wejściowych z parametru trasy, powinieneś wypisać parametry trasy po innych zależnościach. Na przykład, jeśli twoja trasa jest zdefiniowana w ten sposób:

```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```

Nadal możesz dodać type-hint dla `Illuminate\Http\Request` i uzyskać dostęp do parametru trasy `id`, definiując metodę kontrolera w następujący sposób:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Update the specified user.
     */
    public function update(Request $request, string $id): RedirectResponse
    {
        // Update the user...

        return redirect('/users');
    }
}
```

<a name="request-path-and-method"></a>
### Ścieżka, host i metoda żądania

Instancja `Illuminate\Http\Request` zapewnia wiele metod do badania przychodzącego żądania HTTP i rozszerza klasę `Symfony\Component\HttpFoundation\Request`. Omówimy kilka najważniejszych metod poniżej.

<a name="retrieving-the-request-path"></a>
#### Pobieranie ścieżki żądania

Metoda `path` zwraca informacje o ścieżce żądania. Więc jeśli przychodzące żądanie jest skierowane do `http://example.com/foo/bar`, metoda `path` zwróci `foo/bar`:

```php
$uri = $request->path();
```

<a name="inspecting-the-request-path"></a>
#### Sprawdzanie ścieżki / trasy żądania

Metoda `is` pozwala zweryfikować, czy ścieżka przychodzącego żądania pasuje do podanego wzorca. Możesz użyć znaku `*` jako wieloznacznika podczas korzystania z tej metody:

```php
if ($request->is('admin/*')) {
    // ...
}
```

Używając metody `routeIs`, możesz sprawdzić, czy przychodzące żądanie pasuje do [nazwanej trasy](/docs/{{version}}/routing#named-routes):

```php
if ($request->routeIs('admin.*')) {
    // ...
}
```

<a name="retrieving-the-request-url"></a>
#### Pobieranie adresu URL żądania

Aby pobrać pełny adres URL dla przychodzącego żądania, możesz użyć metod `url` lub `fullUrl`. Metoda `url` zwróci adres URL bez ciągu zapytania, podczas gdy metoda `fullUrl` zawiera ciąg zapytania:

```php
$url = $request->url();

$urlWithQueryString = $request->fullUrl();
```

Jeśli chciałbyś dodać dane ciągu zapytania do bieżącego adresu URL, możesz wywołać metodę `fullUrlWithQuery`. Ta metoda łączy podaną tablicę zmiennych ciągu zapytania z bieżącym ciągiem zapytania:

```php
$request->fullUrlWithQuery(['type' => 'phone']);
```

Jeśli chciałbyś uzyskać bieżący adres URL bez podanego parametru ciągu zapytania, możesz użyć metody `fullUrlWithoutQuery`:

```php
$request->fullUrlWithoutQuery(['type']);
```

<a name="retrieving-the-request-host"></a>
#### Pobieranie hosta żądania

Możesz pobrać "host" przychodzącego żądania za pomocą metod `host`, `httpHost` i `schemeAndHttpHost`:

```php
$request->host();
$request->httpHost();
$request->schemeAndHttpHost();
```

<a name="retrieving-the-request-method"></a>
#### Pobieranie metody żądania

Metoda `method` zwróci czasownik HTTP dla żądania. Możesz użyć metody `isMethod`, aby zweryfikować, że czasownik HTTP pasuje do podanego ciągu:

```php
$method = $request->method();

if ($request->isMethod('post')) {
    // ...
}
```

<a name="request-headers"></a>
### Nagłówki żądania

Możesz pobrać nagłówek żądania z instancji `Illuminate\Http\Request` za pomocą metody `header`. Jeśli nagłówek nie jest obecny w żądaniu, zostanie zwrócone `null`. Jednak metoda `header` przyjmuje opcjonalny drugi argument, który zostanie zwrócony, jeśli nagłówek nie jest obecny w żądaniu:

```php
$value = $request->header('X-Header-Name');

$value = $request->header('X-Header-Name', 'default');
```

Metoda `hasHeader` może być użyta do sprawdzenia, czy żądanie zawiera podany nagłówek:

```php
if ($request->hasHeader('X-Header-Name')) {
    // ...
}
```

Dla wygody metoda `bearerToken` może być użyta do pobrania tokena bearer z nagłówka `Authorization`. Jeśli taki nagłówek nie jest obecny, zostanie zwrócony pusty ciąg:

```php
$token = $request->bearerToken();
```

<a name="request-ip-address"></a>
### Adres IP żądania

Metoda `ip` może być użyta do pobrania adresu IP klienta, który wykonał żądanie do twojej aplikacji:

```php
$ipAddress = $request->ip();
```

Jeśli chciałbyś pobrać tablicę adresów IP, w tym wszystkie adresy IP klientów, które zostały przekazane przez proxy, możesz użyć metody `ips`. "Oryginalny" adres IP klienta będzie na końcu tablicy:

```php
$ipAddresses = $request->ips();
```

Ogólnie rzecz biorąc, adresy IP powinny być traktowane jako niezaufane, kontrolowane przez użytkownika dane wejściowe i używane wyłącznie do celów informacyjnych.

<a name="content-negotiation"></a>
### Negocjacja zawartości

Laravel zapewnia kilka metod do sprawdzania żądanych typów zawartości przychodzącego żądania za pomocą nagłówka `Accept`. Najpierw metoda `getAcceptableContentTypes` zwróci tablicę zawierającą wszystkie typy zawartości akceptowane przez żądanie:

```php
$contentTypes = $request->getAcceptableContentTypes();
```

Metoda `accepts` przyjmuje tablicę typów zawartości i zwraca `true`, jeśli którykolwiek z typów zawartości jest akceptowany przez żądanie. W przeciwnym razie zostanie zwrócone `false`:

```php
if ($request->accepts(['text/html', 'application/json'])) {
    // ...
}
```

Możesz użyć metody `prefers`, aby określić, który typ zawartości z podanej tablicy typów zawartości jest najbardziej preferowany przez żądanie. Jeśli żaden z dostarczonych typów zawartości nie jest akceptowany przez żądanie, zostanie zwrócone `null`:

```php
$preferred = $request->prefers(['text/html', 'application/json']);
```

Ponieważ wiele aplikacji obsługuje tylko HTML lub JSON, możesz użyć metody `expectsJson`, aby szybko określić, czy przychodzące żądanie oczekuje odpowiedzi JSON:

```php
if ($request->expectsJson()) {
    // ...
}
```

<a name="psr7-requests"></a>
### Żądania PSR-7

[Standard PSR-7](https://www.php-fig.org/psr/psr-7/) określa interfejsy dla komunikatów HTTP, w tym żądań i odpowiedzi. Jeśli chciałbyś uzyskać instancję żądania PSR-7 zamiast żądania Laravel, najpierw musisz zainstalować kilka bibliotek. Laravel używa komponentu *Symfony HTTP Message Bridge* do konwersji typowych żądań i odpowiedzi Laravel na implementacje kompatybilne z PSR-7:

```shell
composer require symfony/psr-http-message-bridge
composer require nyholm/psr7
```

Po zainstalowaniu tych bibliotek możesz uzyskać żądanie PSR-7, dodając type-hint interfejsu żądania do domknięcia trasy lub metody kontrolera:

```php
use Psr\Http\Message\ServerRequestInterface;

Route::get('/', function (ServerRequestInterface $request) {
    // ...
});
```

> [!NOTE]
> Jeśli zwrócisz instancję odpowiedzi PSR-7 z trasy lub kontrolera, zostanie ona automatycznie przekonwertowana z powrotem na instancję odpowiedzi Laravel i wyświetlona przez framework.

<a name="input"></a>
## Dane wejściowe

<a name="retrieving-input"></a>
### Pobieranie danych wejściowych

<a name="retrieving-all-input-data"></a>
#### Pobieranie wszystkich danych wejściowych

Możesz pobrać wszystkie dane wejściowe przychodzącego żądania jako `array` za pomocą metody `all`. Ta metoda może być używana niezależnie od tego, czy przychodzące żądanie pochodzi z formularza HTML, czy jest żądaniem XHR:

```php
$input = $request->all();
```

Używając metody `collect`, możesz pobrać wszystkie dane wejściowe przychodzącego żądania jako [kolekcję](/docs/{{version}}/collections):

```php
$input = $request->collect();
```

Metoda `collect` pozwala również pobrać podzbiór danych wejściowych przychodzącego żądania jako kolekcję:

```php
$request->collect('users')->each(function (string $user) {
    // ...
});
```

<a name="retrieving-an-input-value"></a>
#### Pobieranie wartości danych wejściowych

Używając kilku prostych metod, możesz uzyskać dostęp do wszystkich danych wejściowych użytkownika z instancji `Illuminate\Http\Request` bez martwienia się o to, który czasownik HTTP został użyty dla żądania. Niezależnie od czasownika HTTP, metoda `input` może być użyta do pobrania danych wejściowych użytkownika:

```php
$name = $request->input('name');
```

Możesz przekazać wartość domyślną jako drugi argument do metody `input`. Ta wartość zostanie zwrócona, jeśli żądana wartość wejściowa nie jest obecna w żądaniu:

```php
$name = $request->input('name', 'Sally');
```

Podczas pracy z formularzami zawierającymi dane wejściowe tablicowe, użyj notacji "kropkowej", aby uzyskać dostęp do tablic:

```php
$name = $request->input('products.0.name');

$names = $request->input('products.*.name');
```

Możesz wywołać metodę `input` bez żadnych argumentów, aby pobrać wszystkie wartości wejściowe jako tablicę asocjacyjną:

```php
$input = $request->input();
```

<a name="retrieving-input-from-the-query-string"></a>
#### Pobieranie danych wejściowych z ciągu zapytania

Podczas gdy metoda `input` pobiera wartości z całego ładunku żądania (w tym ciągu zapytania), metoda `query` pobierze wartości tylko z ciągu zapytania:

```php
$name = $request->query('name');
```

Jeśli żądane dane ciągu zapytania nie są obecne, zostanie zwrócony drugi argument tej metody:

```php
$name = $request->query('name', 'Helen');
```

Możesz wywołać metodę `query` bez żadnych argumentów, aby pobrać wszystkie wartości ciągu zapytania jako tablicę asocjacyjną:

```php
$query = $request->query();
```

<a name="retrieving-json-input-values"></a>
#### Pobieranie wartości wejściowych JSON

Podczas wysyłania żądań JSON do twojej aplikacji, możesz uzyskać dostęp do danych JSON za pomocą metody `input`, o ile nagłówek `Content-Type` żądania jest prawidłowo ustawiony na `application/json`. Możesz nawet użyć składni "kropkowej", aby pobrać wartości zagnieżdżone w tablicach / obiektach JSON:

```php
$name = $request->input('user.name');
```

<a name="retrieving-stringable-input-values"></a>
#### Pobieranie wartości wejściowych jako ciągi

Zamiast pobierać dane wejściowe żądania jako prymitywny `string`, możesz użyć metody `string`, aby pobrać dane żądania jako instancję [Illuminate\Support\Stringable](/docs/{{version}}/strings):

```php
$name = $request->string('name')->trim();
```

<a name="retrieving-integer-input-values"></a>
#### Pobieranie wartości wejściowych jako liczby całkowite

Aby pobrać wartości wejściowe jako liczby całkowite, możesz użyć metody `integer`. Ta metoda spróbuje rzutować wartość wejściową na liczbę całkowitą. Jeśli dane wejściowe nie są obecne lub rzutowanie nie powiedzie się, zwróci wartość domyślną, którą określisz. Jest to szczególnie przydatne dla paginacji lub innych danych wejściowych numerycznych:

```php
$perPage = $request->integer('per_page');
```

<a name="retrieving-boolean-input-values"></a>
#### Pobieranie wartości wejściowych jako wartości logiczne

Podczas pracy z elementami HTML, takimi jak pola wyboru, twoja aplikacja może otrzymywać wartości "truthy", które w rzeczywistości są ciągami. Na przykład "true" lub "on". Dla wygody możesz użyć metody `boolean`, aby pobrać te wartości jako wartości logiczne. Metoda `boolean` zwraca `true` dla 1, "1", true, "true", "on" i "yes". Wszystkie inne wartości zwrócą `false`:

```php
$archived = $request->boolean('archived');
```

<a name="retrieving-array-input-values"></a>
#### Pobieranie wartości wejściowych jako tablice

Wartości wejściowe zawierające tablice mogą być pobierane za pomocą metody `array`. Ta metoda zawsze rzutuje wartość wejściową na tablicę. Jeśli żądanie nie zawiera wartości wejściowej o podanej nazwie, zostanie zwrócona pusta tablica:

```php
$versions = $request->array('versions');
```

<a name="retrieving-date-input-values"></a>
#### Pobieranie wartości wejściowych jako daty

Dla wygody wartości wejściowe zawierające daty / czasy mogą być pobierane jako instancje Carbon za pomocą metody `date`. Jeśli żądanie nie zawiera wartości wejściowej o podanej nazwie, zostanie zwrócone `null`:

```php
$birthday = $request->date('birthday');
```

Drugi i trzeci argument akceptowany przez metodę `date` może być użyty do określenia odpowiednio formatu daty i strefy czasowej:

```php
$elapsed = $request->date('elapsed', '!H:i', 'Europe/Madrid');
```

Jeśli wartość wejściowa jest obecna, ale ma nieprawidłowy format, zostanie zgłoszony `InvalidArgumentException`; dlatego zaleca się walidację danych wejściowych przed wywołaniem metody `date`.

<a name="retrieving-enum-input-values"></a>
#### Pobieranie wartości wejściowych jako enum

Wartości wejściowe, które odpowiadają [enumom PHP](https://www.php.net/manual/en/language.types.enumerations.php), mogą być również pobierane z żądania. Jeśli żądanie nie zawiera wartości wejściowej o podanej nazwie lub enum nie ma wartości bazowej pasującej do wartości wejściowej, zostanie zwrócone `null`. Metoda `enum` przyjmuje nazwę wartości wejściowej i klasę enum jako swoje pierwsze i drugie argumenty:

```php
use App\Enums\Status;

$status = $request->enum('status', Status::class);
```

Możesz również podać wartość domyślną, która zostanie zwrócona, jeśli wartość jest brakująca lub nieprawidłowa:

```php
$status = $request->enum('status', Status::class, Status::Pending);
```

Jeśli wartość wejściowa jest tablicą wartości, które odpowiadają enumowi PHP, możesz użyć metody `enums`, aby pobrać tablicę wartości jako instancje enum:

```php
use App\Enums\Product;

$products = $request->enums('products', Product::class);
```

<a name="retrieving-input-via-dynamic-properties"></a>
#### Pobieranie danych wejściowych za pomocą właściwości dynamicznych

Możesz również uzyskać dostęp do danych wejściowych użytkownika za pomocą właściwości dynamicznych na instancji `Illuminate\Http\Request`. Na przykład, jeśli jeden z formularzy twojej aplikacji zawiera pole `name`, możesz uzyskać dostęp do wartości pola w ten sposób:

```php
$name = $request->name;
```

Podczas używania właściwości dynamicznych Laravel najpierw będzie szukał wartości parametru w ładunku żądania. Jeśli nie jest on obecny, Laravel będzie szukał pola w parametrach dopasowanej trasy.

<a name="retrieving-a-portion-of-the-input-data"></a>
#### Pobieranie części danych wejściowych

Jeśli potrzebujesz pobrać podzbiór danych wejściowych, możesz użyć metod `only` i `except`. Obie te metody przyjmują pojedynczą `array` lub dynamiczną listę argumentów:

```php
$input = $request->only(['username', 'password']);

$input = $request->only('username', 'password');

$input = $request->except(['credit_card']);

$input = $request->except('credit_card');
```

> [!WARNING]
> Metoda `only` zwraca wszystkie pary klucz / wartość, o które prosisz; jednak nie zwróci par klucz / wartość, które nie są obecne w żądaniu.

<a name="input-presence"></a>
### Obecność danych wejściowych

Możesz użyć metody `has`, aby określić, czy wartość jest obecna w żądaniu. Metoda `has` zwraca `true`, jeśli wartość jest obecna w żądaniu:

```php
if ($request->has('name')) {
    // ...
}
```

Gdy podano tablicę, metoda `has` określi, czy wszystkie określone wartości są obecne:

```php
if ($request->has(['name', 'email'])) {
    // ...
}
```

Metoda `hasAny` zwraca `true`, jeśli którakolwiek z określonych wartości jest obecna:

```php
if ($request->hasAny(['name', 'email'])) {
    // ...
}
```

Metoda `whenHas` wykona podane domknięcie, jeśli wartość jest obecna w żądaniu:

```php
$request->whenHas('name', function (string $input) {
    // ...
});
```

Drugie domknięcie może być przekazane do metody `whenHas`, które zostanie wykonane, jeśli określona wartość nie jest obecna w żądaniu:

```php
$request->whenHas('name', function (string $input) {
    // The "name" value is present...
}, function () {
    // The "name" value is not present...
});
```

Jeśli chciałbyś określić, czy wartość jest obecna w żądaniu i nie jest pustym ciągiem, możesz użyć metody `filled`:

```php
if ($request->filled('name')) {
    // ...
}
```

Jeśli chciałbyś określić, czy wartość jest brakująca w żądaniu lub jest pustym ciągiem, możesz użyć metody `isNotFilled`:

```php
if ($request->isNotFilled('name')) {
    // ...
}
```

Gdy podano tablicę, metoda `isNotFilled` określi, czy wszystkie określone wartości są brakujące lub puste:

```php
if ($request->isNotFilled(['name', 'email'])) {
    // ...
}
```

Metoda `anyFilled` zwraca `true`, jeśli którakolwiek z określonych wartości nie jest pustym ciągiem:

```php
if ($request->anyFilled(['name', 'email'])) {
    // ...
}
```

Metoda `whenFilled` wykona podane domknięcie, jeśli wartość jest obecna w żądaniu i nie jest pustym ciągiem:

```php
$request->whenFilled('name', function (string $input) {
    // ...
});
```

Drugie domknięcie może być przekazane do metody `whenFilled`, które zostanie wykonane, jeśli określona wartość nie jest "wypełniona":

```php
$request->whenFilled('name', function (string $input) {
    // The "name" value is filled...
}, function () {
    // The "name" value is not filled...
});
```

Aby określić, czy dany klucz jest nieobecny w żądaniu, możesz użyć metod `missing` i `whenMissing`:

```php
if ($request->missing('name')) {
    // ...
}

$request->whenMissing('name', function () {
    // The "name" value is missing...
}, function () {
    // The "name" value is present...
});
```

<a name="merging-additional-input"></a>
### Łączenie dodatkowych danych wejściowych

Czasami możesz potrzebować ręcznie połączyć dodatkowe dane wejściowe z istniejącymi danymi wejściowymi żądania. Aby to osiągnąć, możesz użyć metody `merge`. Jeśli dany klucz wejściowy już istnieje w żądaniu, zostanie on nadpisany przez dane dostarczone do metody `merge`:

```php
$request->merge(['votes' => 0]);
```

Metoda `mergeIfMissing` może być użyta do łączenia danych wejściowych w żądaniu, jeśli odpowiadające klucze nie istnieją jeszcze w danych wejściowych żądania:

```php
$request->mergeIfMissing(['votes' => 0]);
```

<a name="old-input"></a>
### Stare dane wejściowe

Laravel pozwala zachować dane wejściowe z jednego żądania podczas następnego żądania. Ta funkcja jest szczególnie przydatna do ponownego wypełniania formularzy po wykryciu błędów walidacji. Jednak jeśli używasz wbudowanych [funkcji walidacji](/docs/{{version}}/validation) Laravel, prawdopodobnie nie będziesz musiał ręcznie używać tych metod błyskawicznego przechowywania danych wejściowych sesji, ponieważ niektóre wbudowane narzędzia walidacyjne Laravel wywołają je automatycznie.

<a name="flashing-input-to-the-session"></a>
#### Błyskawiczne przechowywanie danych wejściowych w sesji

Metoda `flash` w klasie `Illuminate\Http\Request` przechowa bieżące dane wejściowe w [sesji](/docs/{{version}}/session), aby były dostępne podczas następnego żądania użytkownika do aplikacji:

```php
$request->flash();
```

Możesz również użyć metod `flashOnly` i `flashExcept`, aby przechować podzbiór danych żądania w sesji. Te metody są przydatne do przechowywania wrażliwych informacji, takich jak hasła, poza sesją:

```php
$request->flashOnly(['username', 'email']);

$request->flashExcept('password');
```

<a name="flashing-input-then-redirecting"></a>
#### Błyskawiczne przechowywanie danych wejściowych, a następnie przekierowanie

Ponieważ często będziesz chciał przechować dane wejściowe w sesji, a następnie przekierować do poprzedniej strony, możesz łatwo połączyć przechowywanie danych wejściowych z przekierowaniem, używając metody `withInput`:

```php
return redirect('/form')->withInput();

return redirect()->route('user.create')->withInput();

return redirect('/form')->withInput(
    $request->except('password')
);
```

<a name="retrieving-old-input"></a>
#### Pobieranie starych danych wejściowych

Aby pobrać przechowywane dane wejściowe z poprzedniego żądania, wywołaj metodę `old` na instancji `Illuminate\Http\Request`. Metoda `old` pobierze wcześniej przechowywane dane wejściowe z [sesji](/docs/{{version}}/session):

```php
$username = $request->old('username');
```

Laravel zapewnia również globalną funkcję pomocniczą `old`. Jeśli wyświetlasz stare dane wejściowe w [szablonie Blade](/docs/{{version}}/blade), wygodniej jest użyć funkcji pomocniczej `old` do ponownego wypełnienia formularza. Jeśli nie istnieją stare dane wejściowe dla podanego pola, zostanie zwrócone `null`:

```blade
<input type="text" name="username" value="{{ old('username') }}">
```

<a name="cookies"></a>
### Ciasteczka

<a name="retrieving-cookies-from-requests"></a>
#### Pobieranie ciasteczek z żądań

Wszystkie ciasteczka utworzone przez framework Laravel są szyfrowane i podpisane kodem uwierzytelniającym, co oznacza, że będą uznawane za nieprawidłowe, jeśli zostały zmienione przez klienta. Aby pobrać wartość ciasteczka z żądania, użyj metody `cookie` na instancji `Illuminate\Http\Request`:

```php
$value = $request->cookie('name');
```

<a name="input-trimming-and-normalization"></a>
## Przycinanie i normalizacja danych wejściowych

Domyślnie Laravel zawiera oprogramowanie pośredniczące `Illuminate\Foundation\Http\Middleware\TrimStrings` i `Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull` w globalnym stosie oprogramowania pośredniczącego twojej aplikacji. Te oprogramowania pośredniczące automatycznie przytnają wszystkie przychodzące pola ciągu w żądaniu, a także przekonwertują wszystkie puste pola ciągu na `null`. Pozwala to nie martwić się o te problemy normalizacji w trasach i kontrolerach.

#### Wyłączanie normalizacji danych wejściowych

Jeśli chciałbyś wyłączyć to zachowanie dla wszystkich żądań, możesz usunąć oba oprogramowania pośredniczące ze stosu oprogramowania pośredniczącego twojej aplikacji, wywołując metodę `$middleware->remove` w pliku `bootstrap/app.php` twojej aplikacji:

```php
use Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull;
use Illuminate\Foundation\Http\Middleware\TrimStrings;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->remove([
        ConvertEmptyStringsToNull::class,
        TrimStrings::class,
    ]);
})
```

Jeśli chciałbyś wyłączyć przycinanie ciągów i konwersję pustych ciągów dla podzbioru żądań do twojej aplikacji, możesz użyć metod oprogramowania pośredniczącego `trimStrings` i `convertEmptyStringsToNull` w pliku `bootstrap/app.php` twojej aplikacji. Obie metody przyjmują tablicę domknięć, które powinny zwrócić `true` lub `false`, aby wskazać, czy normalizacja danych wejściowych powinna zostać pominięta:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->convertEmptyStringsToNull(except: [
        fn (Request $request) => $request->is('admin/*'),
    ]);

    $middleware->trimStrings(except: [
        fn (Request $request) => $request->is('admin/*'),
    ]);
})
```

<a name="files"></a>
## Pliki

<a name="retrieving-uploaded-files"></a>
### Pobieranie przesłanych plików

Możesz pobrać przesłane pliki z instancji `Illuminate\Http\Request` za pomocą metody `file` lub używając właściwości dynamicznych. Metoda `file` zwraca instancję klasy `Illuminate\Http\UploadedFile`, która rozszerza klasę PHP `SplFileInfo` i zapewnia wiele metod do interakcji z plikiem:

```php
$file = $request->file('photo');

$file = $request->photo;
```

Możesz określić, czy plik jest obecny w żądaniu, używając metody `hasFile`:

```php
if ($request->hasFile('photo')) {
    // ...
}
```

<a name="validating-successful-uploads"></a>
#### Walidacja udanych przesyłań

Oprócz sprawdzania, czy plik jest obecny, możesz zweryfikować, że nie było problemów z przesłaniem pliku za pomocą metody `isValid`:

```php
if ($request->file('photo')->isValid()) {
    // ...
}
```

<a name="file-paths-extensions"></a>
#### Ścieżki i rozszerzenia plików

Klasa `UploadedFile` zawiera również metody do uzyskiwania dostępu do w pełni kwalifikowanej ścieżki pliku i jego rozszerzenia. Metoda `extension` spróbuje odgadnąć rozszerzenie pliku na podstawie jego zawartości. To rozszerzenie może różnić się od rozszerzenia dostarczonego przez klienta:

```php
$path = $request->photo->path();

$extension = $request->photo->extension();
```

<a name="other-file-methods"></a>
#### Inne metody plików

Na instancjach `UploadedFile` dostępnych jest wiele innych metod. Sprawdź [dokumentację API dla klasy](https://github.com/symfony/symfony/blob/6.0/src/Symfony/Component/HttpFoundation/File/UploadedFile.php), aby uzyskać więcej informacji na temat tych metod.

<a name="storing-uploaded-files"></a>
### Przechowywanie przesłanych plików

Aby przechować przesłany plik, zazwyczaj użyjesz jednego z skonfigurowanych [systemów plików](/docs/{{version}}/filesystem). Klasa `UploadedFile` ma metodę `store`, która przeniesie przesłany plik na jeden z twoich dysków, który może być lokalizacją w lokalnym systemie plików lub lokalizacją przechowywania w chmurze, taką jak Amazon S3.

Metoda `store` przyjmuje ścieżkę, gdzie plik powinien być przechowywany względem skonfigurowanego katalogu głównego systemu plików. Ta ścieżka nie powinna zawierać nazwy pliku, ponieważ unikalny identyfikator zostanie automatycznie wygenerowany, aby służyć jako nazwa pliku.

Metoda `store` przyjmuje również opcjonalny drugi argument dla nazwy dysku, który powinien być użyty do przechowania pliku. Metoda zwróci ścieżkę pliku względem katalogu głównego dysku:

```php
$path = $request->photo->store('images');

$path = $request->photo->store('images', 's3');
```

Jeśli nie chcesz, aby nazwa pliku była generowana automatycznie, możesz użyć metody `storeAs`, która przyjmuje ścieżkę, nazwę pliku i nazwę dysku jako argumenty:

```php
$path = $request->photo->storeAs('images', 'filename.jpg');

$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```

> [!NOTE]
> Aby uzyskać więcej informacji o przechowywaniu plików w Laravel, zapoznaj się z kompletną [dokumentacją przechowywania plików](/docs/{{version}}/filesystem).

<a name="configuring-trusted-proxies"></a>
## Konfiguracja zaufanych proxy

Podczas uruchamiania aplikacji za load balancerem, który kończy certyfikaty TLS / SSL, możesz zauważyć, że twoja aplikacja czasami nie generuje linków HTTPS podczas korzystania z helpera `url`. Zazwyczaj dzieje się tak, ponieważ twoja aplikacja otrzymuje ruch przekierowany z load balancera na porcie 80 i nie wie, że powinna generować bezpieczne linki.

Aby to rozwiązać, możesz włączyć oprogramowanie pośredniczące `Illuminate\Http\Middleware\TrustProxies`, które jest zawarte w twojej aplikacji Laravel, co pozwala szybko dostosować load balancery lub proxy, którym powinna ufać twoja aplikacja. Twoje zaufane proxy powinny być określone za pomocą metody oprogramowania pośredniczącego `trustProxies` w pliku `bootstrap/app.php` twojej aplikacji:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustProxies(at: [
        '192.168.1.1',
        '10.0.0.0/8',
    ]);
})
```

Oprócz konfigurowania zaufanych proxy możesz również skonfigurować nagłówki proxy, którym należy ufać:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustProxies(headers: Request::HEADER_X_FORWARDED_FOR |
        Request::HEADER_X_FORWARDED_HOST |
        Request::HEADER_X_FORWARDED_PORT |
        Request::HEADER_X_FORWARDED_PROTO |
        Request::HEADER_X_FORWARDED_AWS_ELB
    );
})
```

> [!NOTE]
> Jeśli używasz AWS Elastic Load Balancing, wartość `headers` powinna być `Request::HEADER_X_FORWARDED_AWS_ELB`. Jeśli twój load balancer używa standardowego nagłówka `Forwarded` z [RFC 7239](https://www.rfc-editor.org/rfc/rfc7239#section-4), wartość `headers` powinna być `Request::HEADER_FORWARDED`. Aby uzyskać więcej informacji o stałych, które mogą być używane w wartości `headers`, sprawdź dokumentację Symfony dotyczącą [ufania proxy](https://symfony.com/doc/current/deployment/proxies.html).

<a name="trusting-all-proxies"></a>
#### Ufanie wszystkim proxy

Jeśli używasz Amazon AWS lub innego dostawcy load balancera "w chmurze", możesz nie znać adresów IP swoich rzeczywistych load balancerów. W takim przypadku możesz użyć `*`, aby ufać wszystkim proxy:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustProxies(at: '*');
})
```

<a name="configuring-trusted-hosts"></a>
## Konfiguracja zaufanych hostów

Domyślnie Laravel odpowie na wszystkie żądania, które otrzyma, niezależnie od zawartości nagłówka HTTP `Host` żądania. Ponadto wartość nagłówka `Host` będzie używana podczas generowania bezwzględnych adresów URL do twojej aplikacji podczas żądania web.

Zazwyczaj powinieneś skonfigurować swój serwer web, taki jak Nginx lub Apache, aby wysyłał żądania do twojej aplikacji tylko wtedy, gdy pasują do podanej nazwy hosta. Jednak jeśli nie masz możliwości bezpośredniego dostosowania swojego serwera web i musisz poinstruować Laravel, aby odpowiadał tylko na określone nazwy hostów, możesz to zrobić, włączając oprogramowanie pośredniczące `Illuminate\Http\Middleware\TrustHosts` dla twojej aplikacji.

Aby włączyć oprogramowanie pośredniczące `TrustHosts`, powinieneś wywołać metodę oprogramowania pośredniczącego `trustHosts` w pliku `bootstrap/app.php` twojej aplikacji. Używając argumentu `at` tej metody, możesz określić nazwy hostów, na które twoja aplikacja powinna odpowiadać. Ciąg nazwy hosta jest traktowany jako wyrażenie regularne. Przychodzące żądania z innymi nagłówkami `Host` zostaną odrzucone:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustHosts(at: ['^laravel\.test$']);
})
```

Domyślnie żądania pochodzące z subdomen adresu URL aplikacji są również automatycznie uznawane za zaufane. Jeśli chciałbyś wyłączyć to zachowanie, możesz użyć argumentu `subdomains`:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustHosts(at: ['^laravel\.test$'], subdomains: false);
})
```

Jeśli potrzebujesz dostępu do plików konfiguracyjnych lub bazy danych twojej aplikacji, aby określić zaufane hosty, możesz przekazać domknięcie do argumentu `at`:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustHosts(at: fn () => config('app.trusted_hosts'));
})
```
