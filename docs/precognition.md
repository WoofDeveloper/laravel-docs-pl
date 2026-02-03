# Precognition

- [Wprowadzenie](#introduction)
- [Walidacja na Żywo](#live-validation)
    - [Używanie Vue](#using-vue)
    - [Używanie React](#using-react)
    - [Używanie Alpine i Blade](#using-alpine)
    - [Konfiguracja Axios](#configuring-axios)
- [Walidacja Tablic](#validating-arrays)
- [Dostosowywanie Reguł Walidacji](#customizing-validation-rules)
- [Obsługa Przesyłania Plików](#handling-file-uploads)
- [Zarządzanie Efektami Ubocznymi](#managing-side-effects)
- [Testowanie](#testing)

<a name="introduction"></a>
## Wprowadzenie

Laravel Precognition pozwala przewidywać wynik przyszłego żądania HTTP. Jednym z głównych przypadków użycia Precognition jest możliwość zapewnienia walidacji "na żywo" dla frontendu JavaScript bez konieczności duplikowania reguł walidacji backendu aplikacji.

Gdy Laravel otrzymuje "żądanie precognitive", wykona całe middleware trasy i rozwiąże zależności kontrolera trasy, włączając walidację [żądań formularzy](/docs/{{version}}/validation#form-request-validation) - ale faktycznie nie wykona metody kontrolera trasy.

> [!NOTE]
> Od Inertia 2.3, wsparcie dla Precognition jest wbudowane. Proszę skonsultować się z [dokumentacją formularzy Inertia](https://inertiajs.com/docs/v2/the-basics/forms) dla więcej informacji. Wcześniejsze wersje Inertia wymagają Precognition 0.x.

<a name="live-validation"></a>
## Walidacja na Żywo

<a name="using-vue"></a>
### Używanie Vue

Używając Laravel Precognition, możesz oferować doświadczenia walidacji na żywo swoim użytkownikom bez konieczności duplikowania reguł walidacji w aplikacji frontendowej Vue. Aby zilustrować, jak to działa, zbudujmy formularz do tworzenia nowych użytkowników w naszej aplikacji.

Najpierw, aby włączyć Precognition dla trasy, middleware `HandlePrecognitiveRequests` powinien zostać dodany do definicji trasy. Powinieneś także utworzyć [form request](/docs/{{version}}/validation#form-request-validation) aby pomieścić reguły walidacji trasy:

```php
use App\Http\Requests\StoreUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

Następnie powinieneś zainstalować helpery frontend Laravel Precognition dla Vue przez NPM:

```shell
npm install laravel-precognition-vue
```

Z zainstalowanym pakietem Laravel Precognition, możesz teraz utworzyć obiekt formularza używając funkcji `useForm` Precognition, podając metodę HTTP (`post`), docelowy URL (`/users`) oraz początkowe dane formularza.

Następnie, aby włączyć walidację na żywo, wywołaj metodę `validate` formularza na zdarzeniu `change` każdego pola, podając nazwę pola:

```vue
<script setup>
import { useForm } from 'laravel-precognition-vue';

const form = useForm('post', '/users', {
    name: '',
    email: '',
});

const submit = () => form.submit();
</script>

<template>
    <form @submit.prevent="submit">
        <label for="name">Nazwa</label>
        <input
            id="name"
            v-model="form.name"
            @change="form.validate('name')"
        />
        <div v-if="form.invalid('name')">
            {{ form.errors.name }}
        </div>

        <label for="email">Email</label>
        <input
            id="email"
            type="email"
            v-model="form.email"
            @change="form.validate('email')"
        />
        <div v-if="form.invalid('email')">
            {{ form.errors.email }}
        </div>

        <button :disabled="form.processing">
            Utwórz Użytkownika
        </button>
    </form>
</template>
```

Teraz, gdy formularz jest wypełniany przez użytkownika, Precognition zapewni walidację na żywo opartą na regułach walidacji w żądaniu formularza trasy. Gdy pola formularza są zmieniane, opóźnione żądanie walidacji "precognitive" zostanie wysłane do aplikacji Laravel. Możesz skonfigurować opóźnienie wywołując funkcję `setValidationTimeout` formularza:

```js
form.setValidationTimeout(3000);
```

Gdy żądanie walidacji jest w trakcie, właściwość `validating` formularza będzie `true`:

```html
<div v-if="form.validating">
    Walidacja...
</div>
```

Wszelkie błędy walidacji zwrócone podczas żądania walidacji lub przesłania formularza automatycznie wypełnią obiekt `errors` formularza:

```html
<div v-if="form.invalid('email')">
    {{ form.errors.email }}
</div>
```

Możesz określić, czy formularz ma jakiekolwiek błędy używając właściwości `hasErrors` formularza:

```html
<div v-if="form.hasErrors">
    <!-- ... -->
</div>
```

Możesz również określić, czy pole przeszło lub nie przeszło walidacji, przekazując nazwę pola do funkcji `valid` i `invalid` formularza:

```html
<span v-if="form.valid('email')">
    ✅
</span>

<span v-else-if="form.invalid('email')">
    ❌
</span>
```

> [!WARNING]
> Pole formularza pojawi się jako poprawne lub niepoprawne dopiero po jego zmianie i otrzymaniu odpowiedzi walidacji.

Jeśli walidujesz podzbiór pól formularza za pomocą Precognition, może być przydatne ręczne wyczyszczenie błędów. Możesz użyć funkcji `forgetError` formularza, aby to osiągnąć:

```html
<input
    id="avatar"
    type="file"
    @change="(e) => {
        form.avatar = e.target.files[0]

        form.forgetError('avatar')
    }"
>
```

Jak widzieliśmy, możesz podpiąć się pod zdarzenie `change` pola i walidować pojedyncze pola, gdy użytkownik z nimi wchodzi w interakcje; jednak możesz potrzebować walidować pola, z którymi użytkownik jeszcze nie wchodził w interakcje. Jest to częste przy tworzeniu "kreatora", gdzie chcesz walidować wszystkie widoczne pola, niezależnie od tego, czy użytkownik z nimi wchodził w interakcje, przed przejściem do następnego kroku.

Aby to zrobić za pomocą Precognition, powinieneś wywołać metodę `validate`, przekazując nazwy pól, które chcesz walidować, do klucza konfiguracji `only`. Możesz obsłużyć wynik walidacji za pomocą callbacków `onSuccess` lub `onValidationError`:

```html
<button
    type="button"
    @click="form.validate({
        only: ['name', 'email', 'phone'],
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })"
>Następny Krok</button>
```

Oczywiście możesz również wykonać kod w reakcji na odpowiedź przesłania formularza. Funkcja `submit` formularza zwraca obietnicę żądania Axios. Zapewnia to wygodny sposób na dostęp do zawartości odpowiedzi, resetowanie pól formularza po pomyślnym przesłaniu lub obsługę nieudanego żądania:

```js
const submit = () => form.submit()
    .then(response => {
        form.reset();

        alert('Użytkownik utworzony.');
    })
    .catch(error => {
        alert('Wystąpił błąd.');
    });
```

Możesz określić, czy żądanie przesłania formularza jest w trakcie, sprawdzając właściwość `processing` formularza:

```html
<button :disabled="form.processing">
    Wyślij
</button>
```

<a name="using-react"></a>
### Używanie React

Używając Laravel Precognition, możesz oferować doświadczenia walidacji na żywo swoim użytkownikom bez konieczności duplikowania reguł walidacji w aplikacji frontendowej React. Aby zilustrować, jak to działa, zbudujmy formularz do tworzenia nowych użytkowników w naszej aplikacji.

Najpierw, aby włączyć Precognition dla trasy, middleware `HandlePrecognitiveRequests` powinien zostać dodany do definicji trasy. Powinieneś także utworzyć [form request](/docs/{{version}}/validation#form-request-validation) aby pomieścić reguły walidacji trasy:

```php
use App\Http\Requests\StoreUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

Następnie powinieneś zainstalować helpery frontend Laravel Precognition dla React przez NPM:

```shell
npm install laravel-precognition-react
```

Z zainstalowanym pakietem Laravel Precognition, możesz teraz utworzyć obiekt formularza używając funkcji `useForm` Precognition, podając metodę HTTP (`post`), docelowy URL (`/users`) oraz początkowe dane formularza.

Aby włączyć walidację na żywo, powinieneś nasłuchiwać zdarzeń `change` i `blur` każdego pola. W handlerze zdarzenia `change` powinieneś ustawić dane formularza za pomocą funkcji `setData`, przekazując nazwę pola i nową wartość. Następnie w handlerze zdarzenia `blur` wywołaj metodę `validate` formularza, podając nazwę pola:

```jsx
import { useForm } from 'laravel-precognition-react';

export default function Form() {
    const form = useForm('post', '/users', {
        name: '',
        email: '',
    });

    const submit = (e) => {
        e.preventDefault();

        form.submit();
    };

    return (
        <form onWyślij={submit}>
            <label htmlFor="name">Nazwa</label>
            <input
                id="name"
                value={form.data.name}
                onChange={(e) => form.setData('name', e.target.value)}
                onBlur={() => form.validate('name')}
            />
            {form.invalid('name') && <div>{form.errors.name}</div>}

            <label htmlFor="email">Email</label>
            <input
                id="email"
                value={form.data.email}
                onChange={(e) => form.setData('email', e.target.value)}
                onBlur={() => form.validate('email')}
            />
            {form.invalid('email') && <div>{form.errors.email}</div>}

            <button disabled={form.processing}>
                Utwórz Użytkownika
            </button>
        </form>
    );
};
```

Teraz, gdy formularz jest wypełniany przez użytkownika, Precognition zapewni walidację na żywo powered by the validation rules in the route's form request. When the form's inputs are changed, a debounced "precognitive" validation request will be sent to your Laravel application. You may configure the debounce timeout by calling the form's `setValidationTimeout` function:

```js
form.setValidationTimeout(3000);
```

Gdy żądanie walidacji jest w trakcie, the form's `validating` property will be `true`:

```jsx
{form.validating && <div>Walidacja...</div>}
```

Wszelkie błędy walidacji zwrócone during a validation request or a form submission will automatically populate the form's `errors` object:

```jsx
{form.invalid('email') && <div>{form.errors.email}</div>}
```

Możesz określić, czy formularz ma jakiekolwiek błędy using the form's `hasErrors` property:

```jsx
{form.hasErrors && <div><!-- ... --></div>}
```

You may also determine if an input has passed or failed validation by passing the input's name to the form's `valid` and `invalid` functions, respectively:

```jsx
{form.valid('email') && <span>✅</span>}

{form.invalid('email') && <span>❌</span>}
```

> [!WARNING]
> A form input will only appear as valid or invalid once it has changed and a validation response has been received.

If you are validating a subset of a form's inputs with Precognition, it can be useful to manually clear errors. You may use the form's `forgetError` function to achieve this:

```jsx
<input
    id="avatar"
    type="file"
    onChange={(e) => {
        form.setData('avatar', e.target.files[0]);

        form.forgetError('avatar');
    }}
>
```

Jak widzieliśmy, możesz podpiąć się pod zdarzenie `blur` pola i walidować pojedyncze pola, gdy użytkownik z nimi wchodzi w interakcje; jednak możesz potrzebować walidować pola, z którymi użytkownik jeszcze nie wchodził w interakcje. Jest to częste przy tworzeniu "kreatora", gdzie chcesz walidować wszystkie widoczne pola, niezależnie od tego, czy użytkownik z nimi wchodził w interakcje, przed przejściem do następnego kroku.

Aby to zrobić za pomocą Precognition, powinieneś wywołać metodę `validate`, przekazując nazwy pól, które chcesz walidować, do klucza konfiguracji `only`. Możesz obsłużyć wynik walidacji za pomocą callbacków `onSuccess` lub `onValidationError`:

```jsx
<button
    type="button"
    onClick={() => form.validate({
        only: ['name', 'email', 'phone'],
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })}
>Następny Krok</button>
```

Oczywiście możesz również wykonać kod w reakcji na odpowiedź przesłania formularza. Funkcja `submit` formularza zwraca obietnicę żądania Axios. Zapewnia to wygodny sposób na dostęp do zawartości odpowiedzi, resetowanie pól formularza po pomyślnym przesłaniu lub obsługę nieudanego żądania:

```js
const submit = (e) => {
    e.preventDefault();

    form.submit()
        .then(response => {
            form.reset();

            alert('Użytkownik utworzony.');
        })
        .catch(error => {
            alert('Wystąpił błąd.');
        });
};
```

Możesz określić, czy żądanie przesłania formularza jest w trakcie, sprawdzając właściwość `processing` formularza:

```html
<button disabled={form.processing}>
    Wyślij
</button>
```

<a name="using-alpine"></a>
### Używanie Alpine i Blade

Używając Laravel Precognition, możesz oferować doświadczenia walidacji na żywo swoim użytkownikom bez konieczności duplikowania reguł walidacji w aplikacji frontendowej Alpine. Aby zilustrować, jak to działa, zbudujmy formularz do tworzenia nowych użytkowników w naszej aplikacji.

Najpierw, aby włączyć Precognition dla trasy, middleware `HandlePrecognitiveRequests` powinien zostać dodany do definicji trasy. Powinieneś także utworzyć [form request](/docs/{{version}}/validation#form-request-validation) aby pomieścić reguły walidacji trasy:

```php
use App\Http\Requests\CreateUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (CreateUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

Następnie powinieneś zainstalować helpery frontend Laravel Precognition dla Alpine przez NPM:

```shell
npm install laravel-precognition-alpine
```

Następnie zarejestruj wtyczkę Precognition w Alpine w pliku `resources/js/app.js`:

```js
import Alpine from 'alpinejs';
import Precognition from 'laravel-precognition-alpine';

window.Alpine = Alpine;

Alpine.plugin(Precognition);
Alpine.start();
```

Z zainstalowanym i zarejestrowanym pakietem Laravel Precognition, możesz teraz utworzyć obiekt formularza używając "magicznej" funkcji `$form` Precognition, podając metodę HTTP (`post`), docelowy URL (`/users`) oraz początkowe dane formularza.

Aby włączyć walidację na żywo, powinieneś powiązać dane formularza z odpowiednimi polami, a następnie nasłuchiwać zdarzenia `change` każdego pola. W handlerze zdarzenia `change` powinieneś wywołać metodę `validate` formularza, podając nazwę pola:

```html
<form x-data="{
    form: $form('post', '/register', {
        name: '',
        email: '',
    }),
}">
    @csrf
    <label for="name">Nazwa</label>
    <input
        id="name"
        name="name"
        x-model="form.name"
        @change="form.validate('name')"
    />
    <template x-if="form.invalid('name')">
        <div x-text="form.errors.name"></div>
    </template>

    <label for="email">Email</label>
    <input
        id="email"
        name="email"
        x-model="form.email"
        @change="form.validate('email')"
    />
    <template x-if="form.invalid('email')">
        <div x-text="form.errors.email"></div>
    </template>

    <button :disabled="form.processing">
        Utwórz Użytkownika
    </button>
</form>
```

Teraz, gdy formularz jest wypełniany przez użytkownika, Precognition zapewni walidację na żywo powered by the validation rules in the route's form request. When the form's inputs are changed, a debounced "precognitive" validation request will be sent to your Laravel application. You may configure the debounce timeout by calling the form's `setValidationTimeout` function:

```js
form.setValidationTimeout(3000);
```

Gdy żądanie walidacji jest w trakcie, the form's `validating` property will be `true`:

```html
<template x-if="form.validating">
    <div>Walidacja...</div>
</template>
```

Wszelkie błędy walidacji zwrócone during a validation request or a form submission will automatically populate the form's `errors` object:

```html
<template x-if="form.invalid('email')">
    <div x-text="form.errors.email"></div>
</template>
```

Możesz określić, czy formularz ma jakiekolwiek błędy using the form's `hasErrors` property:

```html
<template x-if="form.hasErrors">
    <div><!-- ... --></div>
</template>
```

You may also determine if an input has passed or failed validation by passing the input's name to the form's `valid` and `invalid` functions, respectively:

```html
<template x-if="form.valid('email')">
    <span>✅</span>
</template>

<template x-if="form.invalid('email')">
    <span>❌</span>
</template>
```

> [!WARNING]
> Pole formularza pojawi się jako poprawne lub niepoprawne dopiero po jego zmianie i otrzymaniu odpowiedzi walidacji.

Jak widzieliśmy, możesz podpiąć się pod zdarzenie `change` pola i walidować pojedyncze pola, gdy użytkownik z nimi wchodzi w interakcje; jednak możesz potrzebować walidować pola, z którymi użytkownik jeszcze nie wchodził w interakcje. Jest to częste przy tworzeniu "kreatora", gdzie chcesz walidować wszystkie widoczne pola, niezależnie od tego, czy użytkownik z nimi wchodził w interakcje, przed przejściem do następnego kroku.

Aby to zrobić za pomocą Precognition, powinieneś wywołać metodę `validate`, przekazując nazwy pól, które chcesz walidować, do klucza konfiguracji `only`. Możesz obsłużyć wynik walidacji za pomocą callbacków `onSuccess` lub `onValidationError`:

```html
<button
    type="button"
    @click="form.validate({
        only: ['name', 'email', 'phone'],
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })"
>Następny Krok</button>
```

Możesz określić, czy żądanie przesłania formularza jest w trakcie, sprawdzając właściwość `processing` formularza:

```html
<button :disabled="form.processing">
    Wyślij
</button>
```

<a name="repopulating-old-form-data"></a>
#### Uzupełnianie Starych Danych Formularza

W przykładzie tworzenia użytkownika omówionym powyżej używamy Precognition do wykonywania walidacji na żywo; jednak wykonujemy tradycyjne przesłanie formularza po stronie serwera. Dlatego formularz powinien być wypełniony wszelkimi "starymi" danymi wejściowymi i błędami walidacji zwróconymi z przesłania formularza po stronie serwera:

```html
<form x-data="{
    form: $form('post', '/register', {
        name: '{{ old('name') }}',
        email: '{{ old('email') }}',
    }).setErrors({{ Js::from($errors->messages()) }}),
}">
```

Alternatywnie, jeśli chcesz przesłać formularz przez XHR, możesz użyć funkcji `submit` formularza, która zwraca obietnicę żądania Axios:

```html
<form
    x-data="{
        form: $form('post', '/register', {
            name: '',
            email: '',
        }),
        submit() {
            this.form.submit()
                .then(response => {
                    this.form.reset();

                    alert('Użytkownik utworzony.')
                })
                .catch(error => {
                    alert('Wystąpił błąd.');
                });
        },
    }"
    @submit.prevent="submit"
>
```

<a name="configuring-axios"></a>
### Konfiguracja Axios

Biblioteki walidacji Precognition używają klienta HTTP [Axios](https://github.com/axios/axios) do wysyłania żądań do backendu aplikacji. Dla wygody instancja Axios może być dostosowana, jeśli wymaga tego aplikacja. Na przykład, gdy używasz biblioteki `laravel-precognition-vue`, możesz dodać dodatkowe nagłówki żądania do każdego wychodzącego żądania w pliku `resources/js/app.js` aplikacji:

```js
import { client } from 'laravel-precognition-vue';

client.axios().defaults.headers.common['Authorization'] = authToken;
```

Lub, jeśli masz już skonfigurowaną instancję Axios dla swojej aplikacji, możesz powiedzieć Precognition, aby zamiast tego używała tej instancji:

```js
import Axios from 'axios';
import { client } from 'laravel-precognition-vue';

window.axios = Axios.create()
window.axios.defaults.headers.common['Authorization'] = authToken;

client.use(window.axios)
```

<a name="validating-arrays"></a>
## Walidacja Tablic

Możesz użyć wieloznaczników do walidacji pól w tablicach lub zagnieżdżonych obiektach. Każdy `*` dopasowuje pojedynczy segment ścieżki:

```js
// Waliduj email dla wszystkich użytkowników w tablicy...
form.validate('users.*.email');

// Waliduj wszystkie pola w obiekcie profilu...
form.validate('profile.*');

// Waliduj wszystkie pola dla wszystkich użytkowników...
form.validate('users.*.*');
```

<a name="customizing-validation-rules"></a>
## Dostosowywanie Reguł Walidacji

Możliwe jest dostosowanie reguł walidacji wykonywanych podczas żądania precognitive za pomocą metody `isPrecognitive` żądania.

Na przykład w formularzu tworzenia użytkownika możemy chcieć walidować, czy hasło jest "niezagrożone" tylko przy końcowym przesłaniu formularza. Dla żądań walidacji precognitive po prostu zwalidujemy, że hasło jest wymagane i ma minimum 8 znaków. Używając metody `isPrecognitive`, możemy dostosować reguły zdefiniowane przez nasze żądanie formularza:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rules\Password;

class StoreUserRequest extends FormRequest
{
    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    protected function rules()
    {
        return [
            'password' => [
                'required',
                $this->isPrecognitive()
                    ? Password::min(8)
                    : Password::min(8)->uncompromised(),
            ],
            // ...
        ];
    }
}
```

<a name="handling-file-uploads"></a>
## Obsługa Przesyłania Plików

Domyślnie Laravel Precognition nie przesyła ani nie waliduje plików podczas żądania walidacji precognitive. Zapewnia to, że duże pliki nie są niepotrzebnie przesyłane wiele razy.

Z powodu tego zachowania powinieneś upewnić się, że aplikacja [dostosowuje odpowiednie reguły walidacji żądania formularza](#customizing-validation-rules), aby określić, że pole jest wymagane tylko dla pełnych przesłań formularza:

```php
/**
 * Get the validation rules that apply to the request.
 *
 * @return array
 */
protected function rules()
{
    return [
        'avatar' => [
            ...$this->isPrecognitive() ? [] : ['required'],
            'image',
            'mimes:jpg,png',
            'dimensions:ratio=3/2',
        ],
        // ...
    ];
}
```

Jeśli chcesz dołączyć pliki w każdym żądaniu walidacji, możesz wywołać funkcję `validateFiles` na instancji formularza po stronie klienta:

```js
form.validateFiles();
```

<a name="managing-side-effects"></a>
## Zarządzanie Efektami Ubocznymi

Dodając middleware `HandlePrecognitiveRequests` do trasy, powinieneś rozważyć, czy są jakiekolwiek efekty uboczne w _innych_ middleware, które powinny być pominięte podczas żądania precognitive.

Na przykład możesz mieć middleware, które zwiększa całkowitą liczbę "interakcji" każdego użytkownika z aplikacją, ale możesz nie chcieć, aby żądania precognitive były liczone jako interakcja. Aby to osiągnąć, możemy sprawdzić metodę `isPrecognitive` żądania przed zwiększeniem licznika interakcji:

```php
<?php

namespace App\Http\Middleware;

use App\Facades\Interaction;
use Closure;
use Illuminate\Http\Request;

class InteractionMiddleware
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): mixed
    {
        if (! $request->isPrecognitive()) {
            Interaction::incrementFor($request->user());
        }

        return $next($request);
    }
}
```

<a name="testing"></a>
## Testowanie

Jeśli chcesz wykonywać żądania precognitive w testach, `TestCase` Laravel zawiera helper `withPrecognition`, który doda nagłówek żądania `Precognition`.

Dodatkowo, jeśli chcesz sprawdzić, czy żądanie precognitive było pomyślne, np. nie zwróciło żadnych błędów walidacji, możesz użyć metody `assertSuccessfulPrecognition` na odpowiedzi:

```php tab=Pest
it('validates registration form with precognition', function () {
    $response = $this->withPrecognition()
        ->post('/register', [
            'name' => 'Taylor Otwell',
        ]);

    $response->assertSuccessfulPrecognition();

    expect(User::count())->toBe(0);
});
```

```php tab=PHPUnit
public function test_it_validates_registration_form_with_precognition()
{
    $response = $this->withPrecognition()
        ->post('/register', [
            'name' => 'Taylor Otwell',
        ]);

    $response->assertSuccessfulPrecognition();
    $this->assertSame(0, User::count());
}
```
