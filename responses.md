# Odpowiedzi HTTP

- [Tworzenie odpowiedzi](#creating-responses)
    - [Dołączanie nagłówków do odpowiedzi](#attaching-headers-to-responses)
    - [Dołączanie ciasteczek do odpowiedzi](#attaching-cookies-to-responses)
    - [Ciasteczka i szyfrowanie](#cookies-and-encryption)
- [Przekierowania](#redirects)
    - [Przekierowywanie do nazwanych tras](#redirecting-named-routes)
    - [Przekierowywanie do akcji kontrolera](#redirecting-controller-actions)
    - [Przekierowywanie do zewnętrznych domen](#redirecting-external-domains)
    - [Przekierowywanie z danymi sesji flash](#redirecting-with-flashed-session-data)
- [Inne typy odpowiedzi](#other-response-types)
    - [Odpowiedzi widoku](#view-responses)
    - [Odpowiedzi JSON](#json-responses)
    - [Pobieranie plików](#file-downloads)
    - [Odpowiedzi plików](#file-responses)
- [Odpowiedzi strumieniowe](#streamed-responses)
    - [Konsumowanie odpowiedzi strumieniowych](#consuming-streamed-responses)
    - [Strumieniowe odpowiedzi JSON](#streamed-json-responses)
    - [Strumienie zdarzeń (SSE)](#event-streams)
    - [Strumieniowe pobieranie plików](#streamed-downloads)
- [Makra odpowiedzi](#response-macros)

<a name="creating-responses"></a>
## Tworzenie odpowiedzi

<a name="strings-arrays"></a>
#### Ciągi znaków i tablice

Wszystkie trasy i kontrolery powinny zwracać odpowiedź, która zostanie wysłana do przeglądarki użytkownika. Laravel zapewnia kilka różnych sposobów zwracania odpowiedzi. Najbardziej podstawową odpowiedzią jest zwrócenie ciągu znaków z trasy lub kontrolera. Framework automatycznie przekonwertuje ciąg znaków na pełną odpowiedź HTTP:

```php
Route::get('/', function () {
    return 'Hello World';
});
```

Oprócz zwracania ciągów znaków z tras i kontrolerów, możesz również zwracać tablice. Framework automatycznie przekonwertuje tablicę na odpowiedź JSON:

```php
Route::get('/', function () {
    return [1, 2, 3];
});
```

> [!NOTE]
> Czy wiesz, że możesz również zwracać [kolekcje Eloquent](/docs/{{version}}/eloquent-collections) z tras lub kontrolerów? Zostaną one automatycznie przekonwertowane na JSON. Wypróbuj to!

<a name="response-objects"></a>
#### Obiekty odpowiedzi

Zazwyczaj nie będziesz zwracać tylko prostych ciągów znaków lub tablic z akcji tras. Zamiast tego będziesz zwracać pełne instancje `Illuminate\Http\Response` lub [widoki](/docs/{{version}}/views).

Zwracanie pełnej instancji `Response` pozwala dostosować kod statusu HTTP odpowiedzi i nagłówki. Instancja `Response` dziedziczy z klasy `Symfony\Component\HttpFoundation\Response`, która zapewnia różnorodne metody do budowania odpowiedzi HTTP:

```php
Route::get('/home', function () {
    return response('Hello World', 200)
        ->header('Content-Type', 'text/plain');
});
```

<a name="eloquent-models-and-collections"></a>
#### Modele i kolekcje Eloquent

Możesz również zwracać modele i kolekcje [Eloquent ORM](/docs/{{version}}/eloquent) bezpośrednio z tras i kontrolerów. Kiedy to robisz, Laravel automatycznie przekonwertuje modele i kolekcje na odpowiedzi JSON, respektując [ukryte atrybuty](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json) modelu:

```php
use App\Models\User;

Route::get('/user/{user}', function (User $user) {
    return $user;
});
```

<a name="attaching-headers-to-responses"></a>
### Dołączanie nagłówków do odpowiedzi

Pamiętaj, że większość metod odpowiedzi jest łańcuchowych, co pozwala na płynne konstruowanie instancji odpowiedzi. Na przykład możesz użyć metody `header`, aby dodać serię nagłówków do odpowiedzi przed wysłaniem jej do użytkownika:

```php
return response($content)
    ->header('Content-Type', $type)
    ->header('X-Header-One', 'Header Value')
    ->header('X-Header-Two', 'Header Value');
```

Lub możesz użyć metody `withHeaders`, aby określić tablicę nagłówków do dodania do odpowiedzi:

```php
return response($content)
    ->withHeaders([
        'Content-Type' => $type,
        'X-Header-One' => 'Header Value',
        'X-Header-Two' => 'Header Value',
    ]);
```

<a name="cache-control-middleware"></a>
#### Middleware kontroli cache

Laravel zawiera middleware `cache.headers`, który może być używany do szybkiego ustawiania nagłówka `Cache-Control` dla grupy tras. Dyrektywy powinny być podawane w odpowiedniku "snake case" odpowiedniej dyrektywy cache-control i powinny być oddzielone średnikiem. Jeśli `etag` jest określone na liście dyrektyw, hash MD5 zawartości odpowiedzi zostanie automatycznie ustawiony jako identyfikator ETag:

```php
Route::middleware('cache.headers:public;max_age=30;s_maxage=300;stale_while_revalidate=600;etag')->group(function () {
    Route::get('/privacy', function () {
        // ...
    });

    Route::get('/terms', function () {
        // ...
    });
});
```

<a name="attaching-cookies-to-responses"></a>
### Dołączanie ciasteczek do odpowiedzi

Możesz dołączyć ciasteczko do wychodzącej instancji `Illuminate\Http\Response` używając metody `cookie`. Powinieneś przekazać nazwę, wartość i liczbę minut, przez które ciasteczko powinno być ważne, do tej metody:

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

Metoda `cookie` akceptuje również kilka dodatkowych argumentów, które są używane rzadziej. Ogólnie rzecz biorąc, te argumenty mają ten sam cel i znaczenie co argumenty przekazywane do natywnej metody PHP [setcookie](https://secure.php.net/manual/en/function.setcookie.php):

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

Jeśli chcesz się upewnić, że ciasteczko zostanie wysłane z wychodzącą odpowiedzią, ale nie masz jeszcze instancji tej odpowiedzi, możesz użyć fasady `Cookie`, aby "zakole jkować" ciasteczka do dołączenia do odpowiedzi, gdy zostanie ona wysłana. Metoda `queue` akceptuje argumenty niezbędne do utworzenia instancji ciasteczka. Te ciasteczka zostaną dołączone do wychodzącej odpowiedzi przed wysłaniem jej do przeglądarki:

```php
use Illuminate\Support\Facades\Cookie;

Cookie::queue('name', 'value', $minutes);
```

<a name="generating-cookie-instances"></a>
#### Generowanie instancji ciasteczek

Jeśli chcesz wygenerować instancję `Symfony\Component\HttpFoundation\Cookie`, którą można dołączyć do instancji odpowiedzi później, możesz użyć globalnego pomocnika `cookie`. To ciasteczko nie zostanie odesłane do klienta, chyba że zostanie dołączone do instancji odpowiedzi:

```php
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

<a name="expiring-cookies-early"></a>
#### Przedwczesne wygasanie ciasteczek

Możesz usunąć ciasteczko, wyga szając je za pomocą metody `withoutCookie` wychodzącej odpowiedzi:

```php
return response('Hello World')->withoutCookie('name');
```

Jeśli nie masz jeszcze instancji wychodzącej odpowiedzi, możesz użyć metody `expire` fasady `Cookie`, aby wygasić ciasteczko:

```php
Cookie::expire('name');
```

<a name="cookies-and-encryption"></a>
### Ciasteczka i szyfrowanie

Domyślnie, dzięki middleware `Illuminate\Cookie\Middleware\EncryptCookies`, wszystkie ciasteczka generowane przez Laravela są szyfrowane i podpisane, więc nie mogą być modyfikowane ani odczytywane przez klienta. Jeśli chcesz wyłączyć szyfrowanie dla podzbioru ciasteczek generowanych przez aplikację, możesz użyć metody `encryptCookies` w pliku `bootstrap/app.php` aplikacji:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->encryptCookies(except: [
        'cookie_name',
    ]);
})
```

> [!NOTE]
> Ogólnie rzecz biorąc, szyfrowanie ciasteczek nigdy nie powinno być wyłączone, ponieważ naraża to ciasteczka na potencjalne ujawnienie danych po stronie klienta i manipulację.

<a name="redirects"></a>
## Przekierowania

Odpowiedzi przekierowania są instancjami klasy `Illuminate\Http\RedirectResponse` i zawierają odpowiednie nagłówki niezbędne do przekierowania użytkownika na inny URL. Istnieje kilka sposobów wygenerowania instancji `RedirectResponse`. Najprostszą metodą jest użycie globalnego pomocnika `redirect`:

```php
Route::get('/dashboard', function () {
    return redirect('/home/dashboard');
});
```

Czasami możesz chcieć przekierować użytkownika do poprzedniej lokalizacji, na przykład gdy przesłany formularz jest niepoprawny. Możesz to zrobić, używając globalnej funkcji pomocniczej `back`. Ponieważ ta funkcja wykorzystuje [sesję](/docs/{{version}}/session), upewnij się, że trasa wywołująca funkcję `back` używa grupy middleware `web`:

```php
Route::post('/user/profile', function () {
    // Walidacja żądania...

    return back()->withInput();
});
```

<a name="redirecting-named-routes"></a>
### Przekierowywanie do nazwanych tras

Gdy wywołujesz pomocnika `redirect` bez parametrów, zwracana jest instancja `Illuminate\Routing\Redirector`, pozwalająca wywołać dowolne metody na instancji `Redirector`. Na przykład, aby wygenerować `RedirectResponse` do nazwanej trasy, możesz użyć metody `route`:

```php
return redirect()->route('login');
```

Jeśli Twoja trasa ma parametry, możesz przekazać je jako drugi argument do metody `route`:

```php
// Dla trasy z następującym URI: /profile/{id}

return redirect()->route('profile', ['id' => 1]);
```

<a name="populating-parameters-via-eloquent-models"></a>
#### Wypełnianie parametrów poprzez modele Eloquent

Jeśli przekierowujesz do trasy z parametrem "ID", który jest wypełniany z modelu Eloquent, możesz przekazać sam model. ID zostanie automatycznie wyodrębnione:

```php
// Dla trasy z następującym URI: /profile/{id}

return redirect()->route('profile', [$user]);
```

Jeśli chcesz dostosować wartość, która jest umieszczana w parametrze trasy, możesz określić kolumnę w definicji parametru trasy (`/profile/{id:slug}`) lub możesz nadpisać metodę `getRouteKey` w modelu Eloquent:

```php
/**
 * Pobierz wartość klucza trasy modelu.
 */
public function getRouteKey(): mixed
{
    return $this->slug;
}
```

<a name="redirecting-controller-actions"></a>
### Przekierowywanie do akcji kontrolera

Możesz również generować przekierowania do [akcji kontrolera](/docs/{{version}}/controllers). Aby to zrobić, przekaz kontroler i nazwę akcji do metody `action`:

```php
use App\Http\Controllers\UserController;

return redirect()->action([UserController::class, 'index']);
```

Jeśli trasa kontrolera wymaga parametrów, możesz przekazać je jako drugi argument do metody `action`:

```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```

<a name="redirecting-external-domains"></a>
### Przekierowywanie do zewnętrznych domen

Czasami możesz potrzebować przekierować do domeny poza aplikacją. Możesz to zrobić, wywołując metodę `away`, która tworzy `RedirectResponse` bez dodatkowego kodowania, walidacji czy weryfikacji URL:

```php
return redirect()->away('https://www.google.com');
```

<a name="redirecting-with-flashed-session-data"></a>
### Przekierowywanie z danymi sesji flash

Przekierowywanie do nowego adresu URL i [zapisywanie danych flash do sesji](/docs/{{version}}/session#flash-data) zazwyczaj odbywa się w tym samym czasie. Zazwyczaj robi się to po pomyślnym wykonaniu akcji, gdy zapisujesz komunikat o sukcesie do sesji. Dla wygody możesz utworzyć instancję `RedirectResponse` i zapisać dane flash do sesji w jednym, płynnym łańcuchu metod:

```php
Route::post('/user/profile', function () {
    // ...

    return redirect('/dashboard')->with('status', 'Profil zaktualizowany!');
});
```

Po przekierowaniu użytkownika możesz wyświetlić wiadomość flash z [sesji](/docs/{{version}}/session). Na przykład, używając [składni Blade](/docs/{{version}}/blade):

```blade
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

<a name="redirecting-with-input"></a>
#### Przekierowywanie z danymi wejściowymi

Możesz użyć metody `withInput` dostarczonej przez instancję `RedirectResponse`, aby zapisać dane wejściowe bieżącego żądania do sesji przed przekierowaniem użytkownika do nowej lokalizacji. Zazwyczaj robi się to, gdy użytkownik napotkał błąd walidacji. Gdy dane wejściowe zostaną zapisane flash do sesji, możesz łatwo [je pobrać](/docs/{{version}}/requests#retrieving-old-input) podczas następnego żądania, aby ponownie wypełnić formularz:

```php
return back()->withInput();
```

<a name="other-response-types"></a>
## Inne typy odpowiedzi

Pomocnik `response` może być używany do generowania innych typów instancji odpowiedzi. Gdy pomocnik `response` jest wywoływany bez argumentów, zwracana jest implementacja [kontraktu](/docs/{{version}}/contracts) `Illuminate\Contracts\Routing\ResponseFactory`. Ten kontrakt zapewnia kilka pomocnych metod do generowania odpowiedzi.

<a name="view-responses"></a>
### Odpowiedzi widoku

Jeśli potrzebujesz kontroli nad statusem i nagłówkami odpowiedzi, ale także musisz zwrócić [widok](/docs/{{version}}/views) jako zawartość odpowiedzi, powinieneś użyć metody `view`:

```php
return response()
    ->view('hello', $data, 200)
    ->header('Content-Type', $type);
```

Oczy wiście, jeśli nie musisz przekazywać niestandardowego kodu statusu HTTP lub niestandardowych nagłówków, możesz użyć globalnej funkcji pomocniczej `view`.

<a name="json-responses"></a>
### Odpowiedzi JSON

Metoda `json` automatycznie ustawi nagłówek `Content-Type` na `application/json`, a także przekonwertuje podaną tablicę na JSON za pomocą funkcji PHP `json_encode`:

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);
```

Jeśli chcesz utworzyć odpowiedź JSONP, możesz użyć metody `json` w połączeniu z metodą `withCallback`:

```php
return response()
    ->json(['name' => 'Abigail', 'state' => 'CA'])
    ->withCallback($request->input('callback'));
```

<a name="file-downloads"></a>
### Pobieranie plików

Metoda `download` może być użyta do wygenerowania odpowiedzi, która wymusza pobranie przez przeglądarkę użytkownika pliku ze ścieżki podanej. Metoda `download` akceptuje nazwę pliku jako drugi argument, która określi nazwę pliku widoczną przez użytkownika pobierającego plik. Na koniec możesz przekazać tablicę nagłówków HTTP jako trzeci argument:

```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```

> [!WARNING]
> Symfony HttpFoundation, które zarządza pobieraniem plików, wymaga, aby pobierany plik miał nazwę ASCII.

<a name="file-responses"></a>
### Odpowiedzi plików

Metoda `file` może być użyta do wyświetlenia pliku, takiego jak obraz lub PDF, bezpośrednio w przeglądarce użytkownika zamiast inicjowania pobierania. Ta metoda akceptuje bezwzględną ścieżkę do pliku jako pierwszy argument i tablicę nagłówków jako drugi argument:

```php
return response()->file($pathToFile);

return response()->file($pathToFile, $headers);
```

<a name="streamed-responses"></a>
## Odpowiedzi strumieniowe

Przez strumieniowanie danych do klienta w miarę ich generowania, możesz znacznie zmniejszyć zużycie pamięci i poprawić wydajność, zwłaszcza dla bardzo dużych odpowiedzi. Odpowiedzi strumieniowe pozwalają klientowi rozpocząć przetwarzanie danych, zanim serwer skończy je wysyłać:

```php
Route::get('/stream', function () {
    return response()->stream(function (): void {
        foreach (['developer', 'admin'] as $string) {
            echo $string;
            ob_flush();
            flush();
            sleep(2); // Symulacja opóźnienia między kawałkami...
        }
    }, 200, ['X-Accel-Buffering' => 'no']);
});
```

Dla wygody, jeśli domknięcie przekazane do metody `stream` zwraca [Generator](https://www.php.net/manual/en/language.generators.overview.php), Laravel automatycznie opróżni bufor wyjściowy między ciągami zwracanymi przez generator, a także wyłączy buforowanie wyjścia Nginx:

```php
Route::post('/chat', function () {
    return response()->stream(function (): Generator {
        $stream = OpenAI::client()->chat()->createStreamed(...);

        foreach ($stream as $response) {
            yield $response->choices[0];
        }
    });
});
```

<a name="consuming-streamed-responses"></a>
### Konsumowanie odpowiedzi strumieniowych

Odpowiedzi strumieniowe mogą być konsumowane za pomocą pakietu npm `stream` Laravela, który zapewnia wygodne API do interakcji ze strumieniami odpowiedzi i zdarzeń Laravela. Aby rozpocząć, zainstaluj pakiet `@laravel/stream-react` lub `@laravel/stream-vue`:

```shell tab=React
npm install @laravel/stream-react
```

```shell tab=Vue
npm install @laravel/stream-vue
```

Następnie `useStream` może być użyte do konsumowania strumienia zdarzeń. Po podaniu adresu URL strumienia, hook automatycznie zaktualizuje `data` o połączoną odpowiedź w miarę zwracania zawartości z aplikacji Laravel:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, isFetching, isStreaming, send } = useStream("chat");

    const sendMessage = () => {
        send({
            message: `Current timestamp: ${Date.now()}`,
        });
    };

    return (
        <div>
            <div>{data}</div>
            {isFetching && <div>Connecting...</div>}
            {isStreaming && <div>Generating...</div>}
            <button onClick={sendMessage}>Send Message</button>
        </div>
    );
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data, isFetching, isStreaming, send } = useStream("chat");

const sendMessage = () => {
    send({
        message: `Current timestamp: ${Date.now()}`,
    });
};
</script>

<template>
    <div>
        <div>{{ data }}</div>
        <div v-if="isFetching">Connecting...</div>
        <div v-if="isStreaming">Generating...</div>
        <button @click="sendMessage">Send Message</button>
    </div>
</template>
```

Kiedy wysyłasz dane z powrotem do strumienia za pomocą `send`, aktywne połączenie do strumienia zostaje anulowane przed wysłaniem nowych danych. Wszystkie żądania są wysyłane jako żądania JSON `POST`.

> [!WARNING]
> Ponieważ hook `useStream` wykonuje żądanie `POST` do aplikacji, wymagany jest ważny token CSRF. Najłatwiejszym sposobem na dostarczenie tokena CSRF jest [dołączenie go przez tag meta w nagłówku layoutu aplikacji](/docs/{{version}}/csrf#csrf-x-csrf-token).

Drugi argument przekazywany do `useStream` to obiekt opcji, którego możesz użyć do dostosowania zachowania konsumpcji strumienia. Domyślne wartości dla tego obiektu są pokazane poniżej:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data } = useStream("chat", {
        id: undefined,
        initialInput: undefined,
        headers: undefined,
        csrfToken: undefined,
        onResponse: (response: Response) => void,
        onData: (data: string) => void,
        onCancel: () => void,
        onFinish: () => void,
        onError: (error: Error) => void,
    });

    return <div>{data}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data } = useStream("chat", {
    id: undefined,
    initialInput: undefined,
    headers: undefined,
    csrfToken: undefined,
    onResponse: (response: Response) => void,
    onData: (data: string) => void,
    onCancel: () => void,
    onFinish: () => void,
    onError: (error: Error) => void,
});
</script>

<template>
    <div>{{ data }}</div>
</template>
```

`onResponse` jest wyzwalane po pomyślnej początkowej odpowiedzi ze strumienia i surowa [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response) jest przekazywana do callbacku. `onData` jest wywoływana, gdy każdy kawałek zostanie odebrany - bieżący kawałek jest przekazywany do callbacku. `onFinish` jest wywoływana, gdy strumień został zakończony i gdy błąd zostanie zgłoszony podczas cyklu fetch / read.

Domyślnie żądanie nie jest wysyłane do strumienia podczas inicjalizacji. Możesz przekazać początkowy payload do strumienia, używając opcji `initialInput`:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data } = useStream("chat", {
        initialInput: {
            message: "Introduce yourself.",
        },
    });

    return <div>{data}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data } = useStream("chat", {
    initialInput: {
        message: "Introduce yourself.",
    },
});
</script>

<template>
    <div>{{ data }}</div>
</template>
```

Aby ręcznie anulować strumień, możesz użyć metody `cancel` zwracanej z hooka:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, cancel } = useStream("chat");

    return (
        <div>
            <div>{data}</div>
            <button onClick={cancel}>Cancel</button>
        </div>
    );
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data, cancel } = useStream("chat");
</script>

<template>
    <div>
        <div>{{ data }}</div>
        <button @click="cancel">Cancel</button>
    </div>
</template>
```

Każde użycie hooka `useStream` generuje losowe `id` do identyfikacji strumienia. Jest ono wysyłane z powrotem do serwera z każdym żądaniem w nagłówku `X-STREAM-ID`. Podczas konsumowania tego samego strumienia z wielu komponentów, możesz czytać i zapisywać do strumienia, podając własne `id`:

```tsx tab=React
// App.tsx
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, id } = useStream("chat");

    return (
        <div>
            <div>{data}</div>
            <StreamStatus id={id} />
        </div>
    );
}

// StreamStatus.tsx
import { useStream } from "@laravel/stream-react";

function StreamStatus({ id }) {
    const { isFetching, isStreaming } = useStream("chat", { id });

    return (
        <div>
            {isFetching && <div>Connecting...</div>}
            {isStreaming && <div>Generating...</div>}
        </div>
    );
}
```

```vue tab=Vue
<!-- App.vue -->
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";
import StreamStatus from "./StreamStatus.vue";

const { data, id } = useStream("chat");
</script>

<template>
    <div>
        <div>{{ data }}</div>
        <StreamStatus :id="id" />
    </div>
</template>

<!-- StreamStatus.vue -->
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const props = defineProps<{
    id: string;
}>();

const { isFetching, isStreaming } = useStream("chat", { id: props.id });
</script>

<template>
    <div>
        <div v-if="isFetching">Connecting...</div>
        <div v-if="isStreaming">Generating...</div>
    </div>
</template>
```

<a name="streamed-json-responses"></a>
### Strumieniowe odpowiedzi JSON

Jeśli musisz strumieniować dane JSON przyrostowo, możesz użyć metody `streamJson`. Ta metoda jest szczególnie przydatna dla dużych zbiorów danych, które muszą być wysyłane postępowo do przeglądarki w formacie, który może być łatwo parsowany przez JavaScript:

```php
use App\Models\User;

Route::get('/users.json', function () {
    return response()->streamJson([
        'users' => User::cursor(),
    ]);
});
```

Hook `useJsonStream` jest identyczny z [hookiem useStream](#consuming-streamed-responses), z wyjątkiem tego, że spróbuje sparsować dane jako JSON, gdy skończy strumieniowanie:

```tsx tab=React
import { useJsonStream } from "@laravel/stream-react";

type User = {
    id: number;
    name: string;
    email: string;
};

function App() {
    const { data, send } = useJsonStream<{ users: User[] }>("users");

    const loadUsers = () => {
        send({
            query: "taylor",
        });
    };

    return (
        <div>
            <ul>
                {data?.users.map((user) => (
                    <li>
                        {user.id}: {user.name}
                    </li>
                ))}
            </ul>
            <button onClick={loadUsers}>Load Users</button>
        </div>
    );
}
```

```vue tab=Vue
<script setup lang="ts">
import { useJsonStream } from "@laravel/stream-vue";

type User = {
    id: number;
    name: string;
    email: string;
};

const { data, send } = useJsonStream<{ users: User[] }>("users");

const loadUsers = () => {
    send({
        query: "taylor",
    });
};
</script>

<template>
    <div>
        <ul>
            <li v-for="user in data?.users" :key="user.id">
                {{ user.id }}: {{ user.name }}
            </li>
        </ul>
        <button @click="loadUsers">Load Users</button>
    </div>
</template>
```

<a name="event-streams"></a>
### Strumienie zdarzeń (SSE)

Metoda `eventStream` może być użyta do zwracania strumieniowej odpowiedzi server-sent events (SSE) używając typu zawartości `text/event-stream`. Metoda `eventStream` akceptuje domknięcie, które powinno [zwracać](https://www.php.net/manual/en/language.generators.overview.php) odpowiedzi do strumienia w miarę ich dostępności:

```php
Route::get('/chat', function () {
    return response()->eventStream(function () {
        $stream = OpenAI::client()->chat()->createStreamed(...);

        foreach ($stream as $response) {
            yield $response->choices[0];
        }
    });
});
```

Jeśli chcesz dostosować nazwę zdarzenia, możesz zwracać instancję klasy `StreamedEvent`:

```php
use Illuminate\Http\StreamedEvent;

yield new StreamedEvent(
    event: 'update',
    data: $response->choices[0],
);
```

<a name="consuming-event-streams"></a>
#### Konsumowanie strumieni zdarzeń

Strumienie zdarzeń mogą być konsumowane za pomocą pakietu npm `stream` Laravela, który zapewnia wygodne API do interakcji ze strumieniami zdarzeń Laravela. Aby rozpocząć, zainstaluj pakiet `@laravel/stream-react` lub `@laravel/stream-vue`:

```shell tab=React
npm install @laravel/stream-react
```

```shell tab=Vue
npm install @laravel/stream-vue
```

Następnie `useEventStream` może być użyte do konsumowania strumienia zdarzeń. Po podaniu adresu URL strumienia, hook automatycznie zaktualizuje `message` o połączoną odpowiedź w miarę zwracania wiadomości z aplikacji Laravel:

```jsx tab=React
import { useEventStream } from "@laravel/stream-react";

function App() {
  const { message } = useEventStream("/chat");

  return <div>{message}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useEventStream } from "@laravel/stream-vue";

const { message } = useEventStream("/chat");
</script>

<template>
  <div>{{ message }}</div>
</template>
```

Drugi argument przekazywany do `useEventStream` to obiekt opcji, którego możesz użyć do dostosowania zachowania konsumpcji strumienia. Domyślne wartości dla tego obiektu są pokazane poniżej:

```jsx tab=React
import { useEventStream } from "@laravel/stream-react";

function App() {
  const { message } = useEventStream("/stream", {
    eventName: "update",
    onMessage: (message) => {
      //
    },
    onError: (error) => {
      //
    },
    onComplete: () => {
      //
    },
    endSignal: "</stream>",
    glue: " ",
  });

  return <div>{message}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useEventStream } from "@laravel/stream-vue";

const { message } = useEventStream("/chat", {
  eventName: "update",
  onMessage: (message) => {
    // ...
  },
  onError: (error) => {
    // ...
  },
  onComplete: () => {
    // ...
  },
  endSignal: "</stream>",
  glue: " ",
});
</script>
```

Strumienie zdarzeń mogą również być ręcznie konsumowane za pomocą obiektu [EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) przez frontend aplikacji. Metoda `eventStream` automatycznie wyśle aktualizację `</stream>` do strumienia zdarzeń, gdy strumień zostanie zakończony:

```js
const source = new EventSource('/chat');

source.addEventListener('update', (event) => {
    if (event.data === '</stream>') {
        source.close();

        return;
    }

    console.log(event.data);
});
```

Aby dostosować końcowe zdarzenie wysyłane do strumienia zdarzeń, możesz podać instancję `StreamedEvent` do argumentu `endStreamWith` metody `eventStream`:

```php
return response()->eventStream(function () {
    // ...
}, endStreamWith: new StreamedEvent(event: 'update', data: '</stream>'));
```

<a name="streamed-downloads"></a>
### Strumieniowe pobieranie plików

Czasami możesz chcieć zamienić odpowiedź tekstową danej operacji na odpowiedź do pobrania bez konieczności zapisywania zawartości operacji na dysku. Możesz użyć metody `streamDownload` w tym scenariuszu. Ta metoda akceptuje callback, nazwę pliku i opcjonalną tablicę nagłówków jako argumenty:

```php
use App\Services\GitHub;

return response()->streamDownload(function () {
    echo GitHub::api('repo')
        ->contents()
        ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');
```

<a name="response-macros"></a>
## Makra odpowiedzi

Jeśli chcesz zdefiniować niestandardową odpowiedź, którą możesz ponownie użyć w różnych trasach i kontrolerach, możesz użyć metody `macro` na fasadzie `Response`. Zazwyczaj powinieneś wywołać tę metodę z metody `boot` jednego z [dostawców usług](/docs/{{version}}/providers) aplikacji, takiego jak dostawca usług `App\Providers\AppServiceProvider`:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Response;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Response::macro('caps', function (string $value) {
            return Response::make(strtoupper($value));
        });
    }
}
```

Funkcja `macro` akceptuje nazwę jako pierwszy argument i domknięcie jako drugi argument. Domknięcie makra zostanie wykonane podczas wywoływania nazwy makra z implementacji `ResponseFactory` lub pomocnika `response`:

```php
return response()->caps('foo');
```
