# Klient HTTP

- [Wprowadzenie](#introduction)
- [Wysyłanie żądań](#making-requests)
    - [Dane żądania](#request-data)
    - [Nagłówki](#headers)
    - [Uwierzytelnianie](#authentication)
    - [Timeout](#timeout)
    - [Ponowne próby](#retries)
    - [Obsługa błędów](#error-handling)
    - [Middleware Guzzle](#guzzle-middleware)
    - [Opcje Guzzle](#guzzle-options)
- [Równoległe żądania](#concurrent-requests)
    - [Pula żądań](#request-pooling)
    - [Wsadowe żądania](#request-batching)
- [Makra](#macros)
- [Testowanie](#testing)
    - [Fałszowanie odpowiedzi](#faking-responses)
    - [Inspekcja żądań](#inspecting-requests)
    - [Zapobieganie przypadkowym żądaniom](#preventing-stray-requests)
- [Zdarzenia](#events)

<a name="introduction"></a>
## Wprowadzenie

Laravel zapewnia wyraziste, minimalistyczne API wokół [klienta HTTP Guzzle](http://docs.guzzlephp.org/en/stable/), pozwalając na szybkie wysyłanie wychodzących żądań HTTP do komunikacji z innymi aplikacjami internetowymi. Opakowanie Laravel wokół Guzzle koncentruje się na najczęstszych przypadkach użycia i wspaniałym doświadczeniu deweloperskim.

<a name="making-requests"></a>
## Wysyłanie żądań

Aby wysyłać żądania, możesz użyć metod `head`, `get`, `post`, `put`, `patch` i `delete` dostarczonych przez fasadę `Http`. Najpierw przyjrzyjmy się, jak wykonać podstawowe żądanie `GET` do innego adresu URL:

```php
use Illuminate\Support\Facades\Http;

$response = Http::get('http://example.com');
```

Metoda `get` zwraca instancję `Illuminate\Http\Client\Response`, która dostarcza różnych metod, które mogą być użyte do inspekcji odpowiedzi:

```php
$response->body() : string;
$response->json($key = null, $default = null) : mixed;
$response->object() : object;
$response->collect($key = null) : Illuminate\Support\Collection;
$response->resource() : resource;
$response->status() : int;
$response->successful() : bool;
$response->redirect(): bool;
$response->failed() : bool;
$response->clientError() : bool;
$response->header($header) : string;
$response->headers() : array;
```

Obiekt `Illuminate\Http\Client\Response` implementuje również interfejs PHP `ArrayAccess`, umożliwiając bezpośredni dostęp do danych odpowiedzi JSON w odpowiedzi:

```php
return Http::get('http://example.com/users/1')['name'];
```

Oprócz wymienionych powyżej metod odpowiedzi, następujące metody mogą być użyte do określenia, czy odpowiedź ma konkretny kod statusu:

```php
$response->ok() : bool;                  // 200 OK
$response->created() : bool;             // 201 Created
$response->accepted() : bool;            // 202 Accepted
$response->noContent() : bool;           // 204 No Content
$response->movedPermanently() : bool;    // 301 Moved Permanently
$response->found() : bool;               // 302 Found
$response->badRequest() : bool;          // 400 Bad Request
$response->unauthorized() : bool;        // 401 Unauthorized
$response->paymentRequired() : bool;     // 402 Payment Required
$response->forbidden() : bool;           // 403 Forbidden
$response->notFound() : bool;            // 404 Not Found
$response->requestTimeout() : bool;      // 408 Request Timeout
$response->conflict() : bool;            // 409 Conflict
$response->unprocessableEntity() : bool; // 422 Unprocessable Entity
$response->tooManyRequests() : bool;     // 429 Too Many Requests
$response->serverError() : bool;         // 500 Internal Server Error
```

<a name="uri-templates"></a>
#### Szablony URI

Klient HTTP pozwala również na konstruowanie adresów URL żądań przy użyciu [specyfikacji szablonów URI](https://www.rfc-editor.org/rfc/rfc6570). Aby zdefiniować parametry URL, które mogą być rozwinięte przez szablon URI, możesz użyć metody `withUrlParameters`:

```php
Http::withUrlParameters([
    'endpoint' => 'https://laravel.com',
    'page' => 'docs',
    'version' => '12.x',
    'topic' => 'validation',
])->get('{+endpoint}/{page}/{version}/{topic}');
```

<a name="dumping-requests"></a>
#### Zrzucanie żądań

If you would like to dump the outgoing request instance before it is sent and terminate the script's execution, you may add the `dd` method na początku definicji żądania:

```php
return Http::dd()->get('http://example.com');
```

<a name="request-data"></a>
### Dane żądania

Oczywiście, podczas wysyłania żądań `POST`, `PUT`, and `PATCH` requests to send additional data with your request, so these methods accept an array of data as their second argument. Domyślnie komunikaty data will be sent using the `application/json` :

```php
use Illuminate\Support\Facades\Http;

$response = Http::post('http://example.com/users', [
    'name' => 'Steve',
    'role' => 'Network Administrator',
]);
```

<a name="get-request-query-parameters"></a>
#### Parametry zapytania żądania GET

When making `GET` requests, you may either append a query string to the URL directly or pass an array of key / value pairs as the second do metody `get` :

```php
$response = Http::get('http://example.com/users', [
    'name' => 'Taylor',
    'page' => 1,
]);
```

Alternatywnie, może być użyta metoda `withQueryParameters` method :

```php
Http::retry(3, 100)->withQueryParameters([
    'name' => 'Taylor',
    'page' => 1,
])->get('http://example.com/users');
```

<a name="sending-form-url-encoded-requests"></a>
#### Wysyłanie żądań zakodowanych jako formularz URL

Jeśli chcesz wysłać dane przy użyciu typu zawartości `application/x-www-form-urlencoded` content type, powinieneś wywołać metodę `asForm` method przed wykonaniem żądania:

```php
$response = Http::asForm()->post('http://example.com/users', [
    'name' => 'Sara',
    'role' => 'Privacy Consultant',
]);
```

<a name="sending-a-raw-request-body"></a>
#### Wysyłanie surowego ciała żądania

You may use the `withBody` method if you would like to provide a raw request body when making a request. The content type may be provided via the method's second argument:

```php
$response = Http::withBody(
    base64_encode($photo), 'image/jpeg'
)->post('http://example.com/photo');
```

<a name="multi-part-requests"></a>
#### Żądania wieloczęściowe

If you would like to send files as multi-part requests, powinieneś wywołać metodę `attach` method before making your request. This method accepts the name of the file and its contents. If needed, you may provide a third argument which will be considered the file's filename, while a fourth argument may be used to provide headers associated with the :

```php
$response = Http::attach(
    'attachment', file_get_contents('photo.jpg'), 'photo.jpg', ['Content-Type' => 'image/jpeg']
)->post('http://example.com/attachments');
```

Zamiast przekazywać surową zawartość pliku, możesz przekazać zasób strumienia:

```php
$photo = fopen('photo.jpg', 'r');

$response = Http::attach(
    'attachment', $photo, 'photo.jpg'
)->post('http://example.com/attachments');
```

<a name="headers"></a>
### Nagłówki

Nagłówki mogą być dodawane do żądań przy użyciu metody `withHeaders` method. This `withHeaders` method accepts an array of key / value pairs:

```php
$response = Http::withHeaders([
    'X-First' => 'foo',
    'X-Second' => 'bar'
])->post('http://example.com/users', [
    'name' => 'Taylor',
]);
```

You may use the `accept` method to specify the content type that your application is expecting w odpowiedzi na żądanie:

```php
$response = Http::accept('application/json')->get('http://example.com/users');
```

Dla wygody możesz użyć metody `acceptJson` method , aby szybko określić, że twoja aplikacja oczekuje typu zawartości `application/json` content type w odpowiedzi na żądanie:

```php
$response = Http::acceptJson()->get('http://example.com/users');
```

The `withHeaders` method merges new headers into the request's existing headers. If needed, you may replace all of the headers entirely using the `replaceHeaders` :

```php
$response = Http::withHeaders([
    'X-Original' => 'foo',
])->replaceHeaders([
    'X-Replacement' => 'bar',
])->post('http://example.com/users', [
    'name' => 'Taylor',
]);
```

<a name="authentication"></a>
### Uwierzytelnianie

You may specify basic and digest authentication credentials using the `withBasicAuth` and `withDigestAuth` methods, respectively:

```php
// Basic authentication...
$response = Http::withBasicAuth('taylor@laravel.com', 'secret')->post(/* ... */);

// Digest authentication...
$response = Http::withDigestAuth('taylor@laravel.com', 'secret')->post(/* ... */);
```

<a name="bearer-tokens"></a>
#### Tokeny Bearer

If you would like to quickly add a bearer token to the request's `Authorization` header, you may use the `withToken` :

```php
$response = Http::withToken('token')->post(/* ... */);
```

<a name="timeout"></a>
### Timeout

The `timeout` method may be used to specify the maximum number of seconds to wait for a response. Domyślnie komunikaty the HTTP client will timeout after 30 seconds:

```php
$response = Http::timeout(3)->get(/* ... */);
```

Jeśli podany timeout zostanie przekroczony, zostanie rzucona instancja `Illuminate\Http\Client\ConnectionException` will  be thrown.

Możesz określić maksymalną liczbę sekund oczekiwania podczas próby połączenia z serwerem przy użyciu metody `connectTimeout` method. The default is 10 seconds:

```php
$response = Http::connectTimeout(3)->get(/* ... */);
```

<a name="retries"></a>
### Ponowne próby

Jeśli chcesz, aby klient HTTP automatycznie ponawiał żądanie w przypadku błędu klienta lub serwera, możesz użyć metody `retry` method. The `retry` method akceptuje maksymalną liczbę prób wykonania żądania oraz liczbę milisekund, które Laravel powinien czekać między próbami:

```php
$response = Http::retry(3, 100)->post(/* ... */);
```

Jeśli chcesz ręcznie obliczyć liczbę milisekund oczekiwania między próbami, możesz przekazać domknięcie jako drugi argument do metody `retry` :

```php
use Exception;

$response = Http::retry(3, function (int $attempt, Exception $exception) {
    return $attempt * 100;
})->post(/* ... */);
```

Dla wygody możesz również przekazać tablicę jako pierwszy argument do metody `retry` method. This array will be used to determine how many milliseconds to sleep between subsequent attempts:

```php
$response = Http::retry([100, 200])->post(/* ... */);
```

W razie potrzeby możesz przekazać trzeci argument do metody `retry` method. The third argument should be a callable that determines if the retries should actually be attempted. For example, you may wish to only retry the request if the initial request encounters an `ConnectionException`:

```php
use Exception;
use Illuminate\Http\Client\PendingRequest;

$response = Http::retry(3, 100, function (Exception $exception, PendingRequest $request) {
    return $exception instanceof ConnectionException;
})->post(/* ... */);
```

If a request attempt fails, you may wish to make a change to the request before a new attempt is made. You can achieve this by modifying the request argument provided to the callable you provided to the `retry` method. For example, you might want to retry the request with a new authorization token if the first attempt returned an authentication error:

```php
use Exception;
use Illuminate\Http\Client\PendingRequest;
use Illuminate\Http\Client\RequestException;

$response = Http::withToken($this->getToken())->retry(2, 0, function (Exception $exception, PendingRequest $request) {
    if (! $exception instanceof RequestException || $exception->response->status() !== 401) {
        return false;
    }

    $request->withToken($this->getNewToken());

    return true;
})->post(/* ... */);
```

Jeśli wszystkie żądania się nie powiodą, zostanie rzucona instancja `Illuminate\Http\Client\RequestException` will be thrown. If you would like to disable this behavior, you may provide a `throw` z wartością `false`. When disabled, the last response received by the client will be returned after all retries have been attempted:

```php
$response = Http::retry(3, 100, throw: false)->post(/* ... */);
```

> [!WARNING]
> If all of the requests fail because of a connection issue, a `Illuminate\Http\Client\ConnectionException` nadal zostanie rzucony, nawet gdy argument `throw` jest ustawiony na `false`.

<a name="error-handling"></a>
### Obsługa błędów

Unlike Guzzle's default behavior, Laravel's HTTP client wrapper does not throw exceptions on client or server errors (`400` and `500` level responses from servers). You may determine if one of these errors was returned using the `successful`, `clientError`, or `serverError` methods:

```php
// Determine if the status code is >= 200 and < 300...
$response->successful();

// Determine if the status code is >= 400...
$response->failed();

// Determine if the response has a 400 level status code...
$response->clientError();

// Determine if the response has a 500 level status code...
$response->serverError();

// Immediately execute the given callback if there was a client or server error...
$response->onError(callable $callback);
```

<a name="throwing-exceptions"></a>
#### Rzucanie wyjątków

Jeśli masz instancję odpowiedzi i chcesz rzucić instancję `Illuminate\Http\Client\RequestException` , jeśli kod statusu odpowiedzi wskazuje na błąd klienta lub serwera, możesz użyć metod `throw` or `throwIf` methods:

```php
use Illuminate\Http\Client\Response;

$response = Http::post(/* ... */);

// Throw an exception if a client or server error occurred...
$response->throw();

// Throw an exception if an error occurred and the given condition is true...
$response->throwIf($condition);

// Throw an exception if an error occurred and the given closure resolves to true...
$response->throwIf(fn (Response $response) => true);

// Throw an exception if an error occurred and the given condition is false...
$response->throwUnless($condition);

// Throw an exception if an error occurred and the given closure resolves to false...
$response->throwUnless(fn (Response $response) => false);

// Throw an exception if the response has a specific status code...
$response->throwIfStatus(403);

// Throw an exception unless the response has a specific status code...
$response->throwUnlessStatus(200);

return $response['user']['id'];
```

The `Illuminate\Http\Client\RequestException` ma publiczną właściwość `$response` property which will allow you to inspect the returned response.

The `throw` method zwraca instancję odpowiedzi, jeśli nie wystąpił błąd, umożliwiając łączenie innych operacji z metodą `throw` :

```php
return Http::post(/* ... */)->throw()->json();
```

Jeśli chcesz wykonać dodatkową logikę przed rzuceniem wyjątku, możesz przekazać domknięcie do metody `throw` method. The exception will be thrown automatically after the closure is invoked, so you do not need to re-throw the exception from within the closure:

```php
use Illuminate\Http\Client\Response;
use Illuminate\Http\Client\RequestException;

return Http::post(/* ... */)->throw(function (Response $response, RequestException $e) {
    // ...
})->json();
```

Domyślnie komunikaty `RequestException` messages are truncated to 120 characters when logged or reported. To customize or disable this behavior, you may utilize the `truncateAt` and `dontTruncate` methods when configuring your application's registered behavior in your `bootstrap/app.php` :

```php
use Illuminate\Http\Client\RequestException;

->registered(function (): void {
    // Truncate request exception messages to 240 characters...
    RequestException::truncateAt(240);

    // Disable request exception message truncation...
    RequestException::dontTruncate();
})
```

Alternatywnie możesz dostosować zachowanie skracania wyjątków dla pojedynczego żądania przy użyciu metody `truncateExceptionsAt` :

```php
return Http::truncateExceptionsAt(240)->post(/* ... */);
```

<a name="guzzle-middleware"></a>
### Middleware Guzzle

Since Laravel's HTTP client is powered by Guzzle, you may take advantage of [Guzzle Middleware](https://docs.guzzlephp.org/en/stable/handlers-and-middleware.html) to manipulate the outgoing request or inspect the incoming response. To manipulate the outgoing request, register a Guzzle middleware via the `withRequestMiddleware` :

```php
use Illuminate\Support\Facades\Http;
use Psr\Http\Message\RequestInterface;

$response = Http::withRequestMiddleware(
    function (RequestInterface $request) {
        return $request->withHeader('X-Example', 'Value');
    }
)->get('http://example.com');
```

Podobnie możesz zbadać przychodzącą odpowiedź HTTP, rejestrując middleware za pomocą metody `withResponseMiddleware` :

```php
use Illuminate\Support\Facades\Http;
use Psr\Http\Message\ResponseInterface;

$response = Http::withResponseMiddleware(
    function (ResponseInterface $response) {
        $header = $response->getHeader('X-Example');

        // ...

        return $response;
    }
)->get('http://example.com');
```

<a name="global-middleware"></a>
#### Globalne middleware

Sometimes, you may want to register a middleware that applies to every outgoing request and incoming response. To accomplish this, you may use the `globalRequestMiddleware` and `globalResponseMiddleware` methods. Typically, these methods should be invoked in the `boot` method of your application's `AppServiceProvider`:

```php
use Illuminate\Support\Facades\Http;

Http::globalRequestMiddleware(fn ($request) => $request->withHeader(
    'User-Agent', 'Example Application/1.0'
));

Http::globalResponseMiddleware(fn ($response) => $response->withHeader(
    'X-Finished-At', now()->toDateTimeString()
));
```

<a name="guzzle-options"></a>
### Opcje Guzzle

Możesz określić dodatkowe [Guzzle request options](http://docs.guzzlephp.org/en/stable/request-options.html) dla wychodzącego żądania przy użyciu metody `withOptions` method. The `withOptions` method accepts an array of key / value pairs:

```php
$response = Http::withOptions([
    'debug' => true,
])->get('http://example.com/users');
```

<a name="global-options"></a>
#### Opcje globalne

Aby skonfigurować domyślne opcje dla każdego wychodzącego żądania, możesz użyć metody `globalOptions` method. Typically, this method should be invoked from the `boot` method of your application's `AppServiceProvider`:

```php
use Illuminate\Support\Facades\Http;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Http::globalOptions([
        'allow_redirects' => false,
    ]);
}
```

<a name="concurrent-requests"></a>
## Równoległe żądania

Sometimes, you may wish to make multiple HTTP requests concurrently. In other words, you want several requests to be dispatched at the same time instead of issuing the requests sequentially. This can lead to substantial performance improvements when interacting with slow HTTP APIs.

<a name="request-pooling"></a>
### Pula żądań

Na szczęście możesz to osiągnąć przy użyciu metody `pool` method. The `pool` method akceptuje domknięcie, które otrzymuje instancję `Illuminate\Http\Client\Pool` , umożliwiając łatwe dodawanie żądań do puli żądań w celu wysłania:

```php
use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$responses = Http::pool(fn (Pool $pool) => [
    $pool->get('http://localhost/first'),
    $pool->get('http://localhost/second'),
    $pool->get('http://localhost/third'),
]);

return $responses[0]->ok() &&
       $responses[1]->ok() &&
       $responses[2]->ok();
```

As you can see, each response instance can be accessed based on the order it was added to the pool. If you wish, you can name the requests using the `as` method, co pozwala na dostęp do odpowiadających odpowiedzi po nazwie:

```php
use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$responses = Http::pool(fn (Pool $pool) => [
    $pool->as('first')->get('http://localhost/first'),
    $pool->as('second')->get('http://localhost/second'),
    $pool->as('third')->get('http://localhost/third'),
]);

return $responses['first']->ok();
```

Maksymalna współbieżność puli żądań może być kontrolowana przez podanie argumentu `concurrency` do metody `pool` method. This value determines the maximum number of HTTP requests that may be concurrently in-flight while processing the request pool:

```php
$responses = Http::pool(fn (Pool $pool) => [
    // ...
], concurrency: 5);
```

<a name="customizing-concurrent-requests"></a>
#### Dostosowywanie równoległych żądań

The `pool` method nie może być łączone z innymi metodami klienta HTTP, takimi jak `withHeaders` or `middleware` methods. If you want to apply custom headers or middleware to pooled requests, you should configure those options on each request in the pool:

```php
use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$headers = [
    'X-Example' => 'example',
];

$responses = Http::pool(fn (Pool $pool) => [
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
]);
```

<a name="request-batching"></a>
### Wsadowe żądania

Innym sposobem pracy z równoległymi żądaniami w Laravel jest użycie metody `batch` method. Like the `pool` method, it akceptuje domknięcie, które otrzymuje instancję `Illuminate\Http\Client\Batch` instance, umożliwiając łatwe dodawanie żądań do puli żądań w celu wysłania, ale pozwala również na zdefiniowanie callbacków zakończenia:

```php
use Illuminate\Http\Client\Batch;
use Illuminate\Http\Client\ConnectionException;
use Illuminate\Http\Client\RequestException;
use Illuminate\Http\Client\Response;
use Illuminate\Support\Facades\Http;

$responses = Http::batch(fn (Batch $batch) => [
    $batch->get('http://localhost/first'),
    $batch->get('http://localhost/second'),
    $batch->get('http://localhost/third'),
])->before(function (Batch $batch) {
    // The batch has been created but no requests have been initialized...
})->progress(function (Batch $batch, int|string $key, Response $response) {
    // An individual request has completed successfully...
})->then(function (Batch $batch, array $results) {
    // All requests completed successfully...
})->catch(function (Batch $batch, int|string $key, Response|RequestException|ConnectionException $response) {
    // First batch request failure detected...
})->finally(function (Batch $batch, array $results) {
    // The batch has finished executing...
})->send();
```

Like the `pool` method, możesz użyć metody `as` method , aby nazwać żądania:

```php
$responses = Http::batch(fn (Batch $batch) => [
    $batch->as('first')->get('http://localhost/first'),
    $batch->as('second')->get('http://localhost/second'),
    $batch->as('third')->get('http://localhost/third'),
])->send();
```

Po uruchomieniu `batch` przez wywołanie metody `send` method, you can't add new requests to it. Trying to do so will result in a `Illuminate\Http\Client\BatchInProgressException` exception being thrown.

Maksymalna współbieżność wsadu żądań może być kontrolowana przez metodę `concurrency` method. This value determines the maximum number of HTTP requests that may be concurrently in-flight while processing the request batch:

```php
$responses = Http::batch(fn (Batch $batch) => [
    // ...
])->concurrency(5)->send();
```

<a name="inspecting-batches"></a>
#### Inspekcja wsadów

The `Illuminate\Http\Client\Batch` dostarczona do callbacków zakończenia wsadu ma różne właściwości i metody, które pomogą ci w interakcji z danym wsadem żądań i jego inspekcji:

```php
// The number of requests assigned to the batch...
$batch->totalRequests;
 
// The number of requests that have not been processed yet...
$batch->pendingRequests;
 
// The number of requests that have failed...
$batch->failedRequests;

// The number of requests that have been processed thus far...
$batch->processedRequests();

// Indicates if the batch has finished executing...
$batch->finished();

// Indicates if the batch has request failures...
$batch->hasFailures();
```
<a name="deferring-batches"></a>
#### Odraczanie wsadów

Gdy wywoływana jest metoda `defer` method is invoked, the batch of requests is not executed immediately. Instead, Laravel will execute the batch after the current application request's HTTP response has been sent to the user, keeping your application feeling fast and responsive:

```php
use Illuminate\Http\Client\Batch;
use Illuminate\Support\Facades\Http;

$responses = Http::batch(fn (Batch $batch) => [
    $batch->get('http://localhost/first'),
    $batch->get('http://localhost/second'),
    $batch->get('http://localhost/third'),
])->then(function (Batch $batch, array $results) {
    // All requests completed successfully...
})->defer();
```

<a name="macros"></a>
## Makra

The Laravel HTTP client allows you to define "macros", which can serve as a fluent, expressive mechanism to configure common request paths and headers when interacting with services throughout your application. To get started, you may define the macro within the `boot` method of your application's `App\Providers\AppServiceProvider` :

```php
use Illuminate\Support\Facades\Http;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Http::macro('github', function () {
        return Http::withHeaders([
            'X-Example' => 'example',
        ])->baseUrl('https://github.com');
    });
}
```

Po skonfigurowaniu makra możesz je wywołać z dowolnego miejsca w aplikacji, aby utworzyć oczekujące żądanie z określoną konfiguracją:

```php
$response = Http::github()->get('/');
```

<a name="testing"></a>
## Testowanie

Many Laravel services provide functionality to help you easily and expressively write tests, and Laravel's HTTP client is no exception. The `Http` facade's `fake` method allows you to instruct the HTTP client to return stubbed / dummy responses when requests are made.

<a name="faking-responses"></a>
### Fałszowanie odpowiedzi

Na przykład, aby poinstruować klienta HTTP, aby zwracał puste odpowiedzi z kodem statusu `200` dla każdego żądania, możesz wywołać metodę `fake` method bez argumentów:

```php
use Illuminate\Support\Facades\Http;

Http::fake();

$response = Http::post(/* ... */);
```

<a name="faking-specific-urls"></a>
#### Fałszowanie konkretnych adresów URL

Alternatywnie możesz przekazać tablicę do metody `fake` method. The array's keys should represent URL patterns that you wish to fake and their associated responses. The `*` character may be used as a wildcard character. You may use the `Http` facade's `response` method , aby skonstruować odpowiedzi stub / fake dla tych punktów końcowych:

```php
Http::fake([
    // Stub a JSON response for GitHub endpoints...
    'github.com/*' => Http::response(['foo' => 'bar'], 200, $headers),

    // Stub a string response for Google endpoints...
    'google.com/*' => Http::response('Hello World', 200, $headers),
]);
```

Any requests made to URLs that have not been faked will actually be executed. If you would like to specify a fallback URL pattern that will stub all unmatched URLs, you may use a single `*` :

```php
Http::fake([
    // Stub a JSON response for GitHub endpoints...
    'github.com/*' => Http::response(['foo' => 'bar'], 200, ['Headers']),

    // Stub a string response for all other endpoints...
    '*' => Http::response('Hello World', 200, ['Headers']),
]);
```

Dla wygody proste odpowiedzi tekstowe, JSON i puste mogą być generowane przez podanie łańcucha, tablicy lub liczby całkowitej jako odpowiedzi:

```php
Http::fake([
    'google.com/*' => 'Hello World',
    'github.com/*' => ['foo' => 'bar'],
    'chatgpt.com/*' => 200,
]);
```

<a name="faking-connection-exceptions"></a>
#### Fałszowanie wyjątków

Sometimes you may need to test your application's behavior if the HTTP client encounters an `Illuminate\Http\Client\ConnectionException` when attempting to make a request. You can instruct the HTTP client to throw a connection exception using the `failedConnection` :

```php
Http::fake([
    'github.com/*' => Http::failedConnection(),
]);
```

To test your application's behavior if a `Illuminate\Http\Client\RequestException` , możesz użyć metody `failedRequest` :

```php
$this->mock(GithubService::class);
    ->shouldReceive('getUser')
    ->andThrow(
        Http::failedRequest(['code' => 'not_found'], 404)
    );
```

<a name="faking-response-sequences"></a>
#### Fałszowanie sekwencji odpowiedzi

Sometimes you may need to specify that a single URL should return a series of fake responses in a specific order. You may accomplish this using the `Http::sequence` method , aby zbudować odpowiedzi:

```php
Http::fake([
    // Stub a series of responses for GitHub endpoints...
    'github.com/*' => Http::sequence()
        ->push('Hello World', 200)
        ->push(['foo' => 'bar'], 200)
        ->pushStatus(404),
]);
```

When all the responses in a response sequence have been consumed, any further requests will cause the response sequence to throw an exception. If you would like to specify a default response that should be returned when a sequence is empty, you may use the `whenEmpty` :

```php
Http::fake([
    // Stub a series of responses for GitHub endpoints...
    'github.com/*' => Http::sequence()
        ->push('Hello World', 200)
        ->push(['foo' => 'bar'], 200)
        ->whenEmpty(Http::response()),
]);
```

Jeśli chcesz sfałszować sekwencję odpowiedzi, ale nie musisz określać konkretnego wzorca URL, który powinien być sfałszowany, możesz użyć metody `Http::fakeSequence` :

```php
Http::fakeSequence()
    ->push('Hello World', 200)
    ->whenEmpty(Http::response());
```

<a name="fake-callback"></a>
#### Callback fałszowania

Jeśli wymagasz bardziej skomplikowanej logiki do określenia, jakie odpowiedzi zwrócić dla określonych punktów końcowych, możesz przekazać domknięcie do metody `fake` method. This closure will receive an instance of `Illuminate\Http\Client\Request` and should return a response instance. Within your closure, you may perform whatever logic is necessary to determine what type of response to return:

```php
use Illuminate\Http\Client\Request;

Http::fake(function (Request $request) {
    return Http::response('Hello World', 200);
});
```

<a name="inspecting-requests"></a>
### Inspekcja żądań

When faking responses, you may occasionally wish to inspect the requests the client receives in order to make sure your application is sending the correct data or headers. You may accomplish this by calling the `Http::assertSent` method po wywołaniu `Http::fake`.

The `assertSent` method akceptuje domknięcie, które otrzyma instancję `Illuminate\Http\Client\Request` instance and should return a boolean value indicating if the request matches your expectations. In order for the test to pass, at least one request must have been issued matching the given expectations:

```php
use Illuminate\Http\Client\Request;
use Illuminate\Support\Facades\Http;

Http::fake();

Http::withHeaders([
    'X-First' => 'foo',
])->post('http://example.com/users', [
    'name' => 'Taylor',
    'role' => 'Developer',
]);

Http::assertSent(function (Request $request) {
    return $request->hasHeader('X-First', 'foo') &&
           $request->url() == 'http://example.com/users' &&
           $request['name'] == 'Taylor' &&
           $request['role'] == 'Developer';
});
```

W razie potrzeby możesz zweryfikować, że konkretne żądanie nie zostało wysłane, używając metody `assertNotSent` :

```php
use Illuminate\Http\Client\Request;
use Illuminate\Support\Facades\Http;

Http::fake();

Http::post('http://example.com/users', [
    'name' => 'Taylor',
    'role' => 'Developer',
]);

Http::assertNotSent(function (Request $request) {
    return $request->url() === 'http://example.com/posts';
});
```

You may use the `assertSentCount` method , aby zweryfikować, ile żądań zostało "wysłanych" podczas testu:

```php
Http::fake();

Http::assertSentCount(5);
```

Lub możesz użyć metody `assertNothingSent` method , aby zweryfikować, że żadne żądania nie zostały wysłane podczas testu:

```php
Http::fake();

Http::assertNothingSent();
```

<a name="recording-requests-and-responses"></a>
#### Nagrywanie żądań / odpowiedzi

You may use the `recorded` method to gather all requests and their corresponding responses. The `recorded` method zwraca kolekcję tablic zawierających instancje `Illuminate\Http\Client\Request` and `Illuminate\Http\Client\Response`:

```php
Http::fake([
    'https://laravel.com' => Http::response(status: 500),
    'https://nova.laravel.com/' => Http::response(),
]);

Http::get('https://laravel.com');
Http::get('https://nova.laravel.com/');

$recorded = Http::recorded();

[$request, $response] = $recorded[0];
```

Dodatkowo metoda `recorded` method akceptuje domknięcie, które otrzyma instancję instance of `Illuminate\Http\Client\Request` and `Illuminate\Http\Client\Response` i może być użyta do filtrowania par żądanie / odpowiedź na podstawie twoich oczekiwań:

```php
use Illuminate\Http\Client\Request;
use Illuminate\Http\Client\Response;

Http::fake([
    'https://laravel.com' => Http::response(status: 500),
    'https://nova.laravel.com/' => Http::response(),
]);

Http::get('https://laravel.com');
Http::get('https://nova.laravel.com/');

$recorded = Http::recorded(function (Request $request, Response $response) {
    return $request->url() !== 'https://laravel.com' &&
           $response->successful();
});
```

<a name="preventing-stray-requests"></a>
### Zapobieganie przypadkowym żądaniom

Jeśli chcesz upewnić się, że wszystkie żądania wysłane przez klienta HTTP zostały sfałszowane w całym indywidualnym teście lub kompletnym pakiecie testów, możesz wywołać metodę `preventStrayRequests` method. After calling this method, any requests that do not have a corresponding fake response will throw an exception rather than making the actual HTTP request:

```php
use Illuminate\Support\Facades\Http;

Http::preventStrayRequests();

Http::fake([
    'github.com/*' => Http::response('ok'),
]);

// An "ok" response is returned...
Http::get('https://github.com/laravel/framework');

// An exception is thrown...
Http::get('https://laravel.com');
```

Sometimes, you may wish to prevent most stray requests while still allowing specific requests to execute. To accomplish this, you may pass an array of URL patterns to the `allowStrayRequests` method. Any request matching one of the given patterns will be allowed, while all other requests will continue to throw an exception:

```php
use Illuminate\Support\Facades\Http;

Http::preventStrayRequests();

Http::allowStrayRequests([
    'http://127.0.0.1:5000/*',
]);

// This request is executed...
Http::get('http://127.0.0.1:5000/generate');

// An exception is thrown...
Http::get('https://laravel.com');
```

<a name="events"></a>
## Zdarzenia

Laravel fires three events during the process of sending HTTP requests. The `RequestSending` jest uruchamiane przed wysłaniem żądania, podczas gdy zdarzenie `ResponseReceived` event is fired after a response is received for a given request. The `ConnectionFailed` event is fired if no response is received for a given request.

The `RequestSending` and `ConnectionFailed` zawierają publiczną właściwość `$request` property , którą możesz użyć do zbadania instancji `Illuminate\Http\Client\Request` instance. Likewise, the `ResponseReceived` zawiera właściwość `$request` property oraz właściwość `$response` property , które mogą być użyte do zbadania instancji `Illuminate\Http\Client\Response` instance. You may create [event listeners](/docs/{{version}}/events) dla tych zdarzeń w swojej aplikacji:

```php
use Illuminate\Http\Client\Events\RequestSending;

class LogRequest
{
    /**
     * Handle the event.
     */
    public function handle(RequestSending $event): void
    {
        // $event->request ...
    }
}
```
