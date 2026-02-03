# Walidacja

- [Wprowadzenie](#introduction)
- [Szybki start walidacji](#validation-quickstart)
    - [Definiowanie tras](#quick-defining-the-routes)
    - [Tworzenie kontrolera](#quick-creating-the-controller)
    - [Pisanie logiki walidacji](#quick-writing-the-validation-logic)
    - [Wyświetlanie błędów walidacji](#quick-displaying-the-validation-errors)
    - [Ponowne wypełnianie formularzy](#repopulating-forms)
    - [Uwaga o polach opcjonalnych](#a-note-on-optional-fields)
    - [Format odpowiedzi z błędami walidacji](#validation-error-response-format)
- [Walidacja żądań formularzy](#form-request-validation)
    - [Tworzenie żądań formularzy](#creating-form-requests)
    - [Autoryzacja żądań formularzy](#authorizing-form-requests)
    - [Dostosowywanie komunikatów błędów](#customizing-the-error-messages)
    - [Przygotowanie danych wejściowych do walidacji](#preparing-input-for-validation)
- [Ręczne tworzenie walidatorów](#manually-creating-validators)
    - [Automatyczne przekierowanie](#automatic-redirection)
    - [Nazwane worki błędów](#named-error-bags)
    - [Dostosowywanie komunikatów błędów](#manual-customizing-the-error-messages)
    - [Wykonywanie dodatkowej walidacji](#performing-additional-validation)
- [Praca z zwalidowanymi danymi wejściowymi](#working-with-validated-input)
- [Praca z komunikatami błędów](#working-with-error-messages)
    - [Określanie niestandardowych komunikatów w plikach językowych](#specifying-custom-messages-in-language-files)
    - [Określanie atrybutów w plikach językowych](#specifying-attribute-in-language-files)
    - [Określanie wartości w plikach językowych](#specifying-values-in-language-files)
- [Dostępne reguły walidacji](#available-validation-rules)
- [Warunkowe dodawanie reguł](#conditionally-adding-rules)
- [Walidacja tablic](#validating-arrays)
    - [Walidacja zagnieżdżonych danych tablicowych](#validating-nested-array-input)
    - [Indeksy i pozycje w komunikatach błędów](#error-message-indexes-and-positions)
- [Walidacja plików](#validating-files)
- [Walidacja haseł](#validating-passwords)
- [Niestandardowe reguły walidacji](#custom-validation-rules)
    - [Używanie obiektów reguł](#using-rule-objects)
    - [Używanie domknięć](#using-closures)
    - [Reguły niejawne](#implicit-rules)

<a name="introduction"></a>
## Wprowadzenie

Laravel zapewnia kilka różnych podejść do walidacji przychodzących danych aplikacji. Najczęściej używana jest metoda `validate` dostępna dla wszystkich przychodzących żądań HTTP. Omówimy jednak również inne podejścia do walidacji.

Laravel zawiera szeroką gamę wygodnych reguł walidacji, które możesz zastosować do danych, w tym możliwość walidacji, czy wartości są unikalne w danej tabeli bazy danych. Omówimy każdą z tych reguł walidacji szczegółowo, abyś był zaznajomiony ze wszystkimi funkcjami walidacji Laravel.

<a name="validation-quickstart"></a>
## Szybki start walidacji

Aby poznać potężne funkcje walidacji Laravel, przyjrzyjmy się kompletnemu przykładowi walidacji formularza i wyświetlania komunikatów błędów użytkownikowi. Czytając ten ogólny przegląd, będziesz w stanie uzyskać dobre ogólne zrozumienie, jak walidować przychodzące dane żądań za pomocą Laravel:

<a name="quick-defining-the-routes"></a>
### Definiowanie tras

Najpierw załóżmy, że mamy następujące trasy zdefiniowane w naszym pliku `routes/web.php` file:

```php
use App\Http\Controllers\PostController;

Route::get('/post/create', [PostController::class, 'create']);
Route::post('/post', [PostController::class, 'store']);
```

Trasa `GET` wyświetli formularz dla użytkownika do utworzenia nowego posta na blogu, podczas gdy trasa `POST` zapisze nowy post w bazie danych.

<a name="quick-creating-the-controller"></a>
### Tworzenie kontrolera

Następnie przyjrzyjmy się prostemu kontrolerowi, który obsługuje przychodzące żądania do tych tras. Pozostawimy `store` metodę pustą na razie:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\View\View;

class PostController extends Controller
{
    /**
     * Pokaż formularz do utworzenia nowego posta na blogu.
     */
    public function create(): View
    {
        return view('post.create');
    }

    /**
     * Zapisz nowy post na blogu.
     */
    public function store(Request $request): RedirectResponse
    {
        // Zwaliduj i zapisz post na blogu...

        $post = /** ... */

        return to_route('post.show', ['post' => $post->id]);
    }
}
```

<a name="quick-writing-the-validation-logic"></a>
### Pisanie logiki walidacji

Teraz jesteśmy gotowi wypełnić naszą `store` metodę logiką walidacji nowego posta na blogu. Aby to zrobić, użyjemy `validate` metody dostarczonej przez `Illuminate\Http\Request` obiekt. Jeśli reguły walidacji przejdą, Twój kod będzie wykonywany normalnie; jednak jeśli walidacja zawiedzie, `Illuminate\Validation\ValidationException` wyjątek zostanie zgłoszony i odpowiednia odpowiedź z błędem zostanie automatycznie wysłana z powrotem do użytkownika.

Jeśli walidacja zawiedzie podczas tradycyjnego żądania HTTP, zostanie wygenerowana odpowiedź przekierowania do poprzedniego URL. Jeśli przychodzące żądanie jest żądaniem XHR, [odpowiedź JSON zawierająca komunikaty błędów walidacji](#validation-error-response-format) zostanie zwrócona.

Aby lepiej zrozumieć `validate` metodę, wróćmy do `store` metody:

```php
/**
 * Zapisz nowy post na blogu.
 */
public function store(Request $request): RedirectResponse
{
    $validated = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // Post na blogu jest prawidłowy...

    return redirect('/posts');
}
```

Jak widać, reguły walidacji są przekazywane do `validate` metody. Nie martw się - wszystkie dostępne reguły walidacji są [udokumentowane](#available-validation-rules). Ponownie, jeśli walidacja zawiedzie, odpowiednia odpowiedź zostanie automatycznie wygenerowana. Jeśli walidacja przejdzie, nasz kontroler będzie kontynuował wykonywanie normalnie.

Alternatywnie reguły walidacji mogą być określone jako tablice reguł zamiast pojedynczego `|` ciągu rozdzielanego:

```php
$validatedData = $request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

Ponadto możesz użyć `validateWithBag` metody do walidacji żądania i przechowywania komunikatów błędów w [nazwanym worku błędów](#named-error-bags):

```php
$validatedData = $request->validateWithBag('post', [
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

<a name="stopping-on-first-validation-failure"></a>
#### Zatrzymanie przy pierwszym błędzie walidacji

Czasami możesz chcieć zatrzymać uruchamianie reguł walidacji na atrybucie po pierwszym błędzie walidacji. Aby to zrobić, przypisz `bail` regułę do atrybutu:

```php
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

W tym przykładzie, jeśli `unique` reguła na `title` atrybucie zawiedzie, `max` reguła nie zostanie sprawdzona. Reguły będą walidowane w kolejności, w jakiej zostały przypisane.

<a name="a-note-on-nested-attributes"></a>
#### Uwaga o zagnieżdżonych atrybutach

Jeśli przychodzące żądanie HTTP zawiera "zagnieżdżone" dane pól, możesz określić te pola w swoich regułach walidacji używając składni "kropkowej":

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

Z drugiej strony, jeśli nazwa Twojego pola zawiera dosłowną kropkę, możesz jawnie zapobiec interpretowaniu tego jako składni "kropkowej", uciekając kropkę za pomocą ukośnika odwrotnego:

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'v1\.0' => 'required',
]);
```

<a name="quick-displaying-the-validation-errors"></a>
### Wyświetlanie błędów walidacji

Co więc, jeśli pola przychodzącego żądania nie przejdą podanych reguł walidacji? Jak wspomniano wcześniej, Laravel automatycznie przekieruje użytkownika z powrotem do ich poprzedniej lokalizacji. Ponadto wszystkie błędy walidacji i [dane wejściowe żądania](/docs/{{version}}/requests#retrieving-old-input) zostaną automatycznie [przeniesione do sesji](/docs/{{version}}/session#flash-data).

Zmienna `$errors` jest udostępniana wszystkim widokom aplikacji przez middleware `Illuminate\View\Middleware\ShareErrorsFromSession`, który jest dostarczany przez grupę middleware `web`. Gdy ten middleware jest zastosowany, zmienna `$errors` będzie zawsze dostępna w Twoich widokach, pozwalając Ci wygodnie założyć, że zmienna `$errors` jest zawsze zdefiniowana i może być bezpiecznie używana. Zmienna `$errors` będzie instancją `Illuminate\Support\MessageBag`. Więcej informacji na temat pracy z tym obiektem znajdziesz w [jego dokumentacji](#working-with-error-messages).

W naszym przykładzie, użytkownik zostanie przekierowany do metody `create` naszego kontrolera, gdy walidacja zawiedzie, co pozwoli nam wyświetlić komunikaty błędów w widoku:

```blade
<!-- /resources/views/post/create.blade.php -->

<h1>Utwórz post</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Formularz tworzenia posta -->
```

<a name="quick-customizing-the-error-messages"></a>
#### Dostosowywanie komunikatów błędów

Każda wbudowana reguła walidacji Laravel ma komunikat błędu, który znajduje się w pliku `lang/en/validation.php` Twojej aplikacji. Jeśli Twoja aplikacja nie ma katalogu `lang`, możesz polecić Laravel, aby go utworzył, używając polecenia Artisan `lang:publish`.

W pliku `lang/en/validation.php` znajdziesz wpis tłumaczenia dla każdej reguły walidacji. Możesz swobodnie zmieniać lub modyfikować te komunikaty w zależności od potrzeb Twojej aplikacji.

Ponadto możesz skopiować ten plik do innego katalogu językowego, aby przetłumaczyć komunikaty na język Twojej aplikacji. Aby dowiedzieć się więcej o lokalizacji Laravel, zapoznaj się z kompletną [dokumentacją lokalizacji](/docs/{{version}}/localization).

> [!WARNING]
> Domyślnie szkielet aplikacji Laravel nie zawiera katalogu `lang`. Jeśli chcesz dostosować pliki językowe Laravel, możesz je opublikować za pomocą polecenia Artisan `lang:publish`.

<a name="quick-xhr-requests-and-validation"></a>
#### Żądania XHR i walidacja

W tym przykładzie użyliśmy tradycyjnego formularza do wysyłania danych do aplikacji. Jednak wiele aplikacji otrzymuje żądania XHR z frontendu opartego na JavaScript. Podczas używania metody `validate` podczas żądania XHR, Laravel nie wygeneruje odpowiedzi przekierowania. Zamiast tego Laravel generuje [odpowiedź JSON zawierającą wszystkie błędy walidacji](#validation-error-response-format). Ta odpowiedź JSON zostanie wysłana z kodem statusu HTTP 422.

<a name="the-at-error-directive"></a>
#### Dyrektywa `@error`

Możesz użyć dyrektywy [Blade](/docs/{{version}}/blade) `@error`, aby szybko sprawdzić, czy istnieją komunikaty błędów walidacji dla danego atrybutu. W dyrektywie `@error` możesz wyświetlić zmienną `$message`, aby pokazać komunikat błędu:

```blade
<!-- /resources/views/post/create.blade.php -->

<label for="title">Tytuł posta</label>

<input
    id="title"
    type="text"
    name="title"
    class="@error('title') is-invalid @enderror"
/>

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

Jeśli używasz [nazwanych worków błędów](#named-error-bags), możesz przekazać nazwę worka błędów jako drugi argument do dyrektywy `@error`:

```blade
<input ... class="@error('title', 'post') is-invalid @enderror">
```

<a name="repopulating-forms"></a>
### Ponowne wypełnianie formularzy

Gdy Laravel generuje odpowiedź przekierowania z powodu błędu walidacji, framework automatycznie [przenosi wszystkie dane wejściowe żądania do sesji](/docs/{{version}}/session#flash-data). Odbywa się to po to, abyś mógł wygodnie uzyskać dostęp do danych wejściowych podczas następnego żądania i ponownie wypełnić formularz, który użytkownik próbował przesłać.

Aby pobrać przeniesione dane wejściowe z poprzedniego żądania, wywołaj metodę `old` na instancji `Illuminate\Http\Request`. Metoda `old` pobierze wcześniej przeniesione dane wejściowe z [sesji](/docs/{{version}}/session):

```php
$title = $request->old('title');
```

Laravel zapewnia również globalny helper `old`. Jeśli wyświetlasz stare dane wejściowe w [szablonie Blade](/docs/{{version}}/blade), wygodniej jest użyć helpera `old`, aby ponownie wypełnić formularz. Jeśli nie ma starych danych wejściowych dla danego pola, zostanie zwrócone `null`:

```blade
<input type="text" name="title" value="{{ old('title') }}">
```

<a name="a-note-on-optional-fields"></a>
### Uwaga o polach opcjonalnych

Domyślnie Laravel zawiera middleware `TrimStrings` i `ConvertEmptyStringsToNull` w globalnym stosie middleware Twojej aplikacji. Z tego powodu często będziesz musiał oznaczyć swoje "opcjonalne" pola żądania jako `nullable`, jeśli nie chcesz, aby walidator uznawał wartości `null` za nieprawidłowe. Na przykład:

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
    'publish_at' => 'nullable|date',
]);
```

W tym przykładzie określamy, że pole `publish_at` może być albo `null`, albo prawidłową reprezentacją daty. Jeśli modyfikator `nullable` nie zostanie dodany do definicji reguły, walidator uznałby `null` za nieprawidłową datę.

<a name="validation-error-response-format"></a>
### Format odpowiedzi z błędami walidacji

Gdy Twoja aplikacja zgłasza wyjątek `Illuminate\Validation\ValidationException`, a przychodzące żądanie HTTP oczekuje odpowiedzi JSON, Laravel automatycznie sformatuje dla Ciebie komunikaty błędów i zwróci odpowiedź HTTP `422 Unprocessable Entity`.

Poniżej możesz przejrzeć przykład formatu odpowiedzi JSON dla błędów walidacji. Zauważ, że zagnieżdżone klucze błędów są spłaszczone do formatu notacji "kropkowej":

```json
{
    "message": "The team name must be a string. (and 4 more errors)",
    "errors": {
        "team_name": [
            "The team name must be a string.",
            "The team name must be at least 1 characters."
        ],
        "authorization.role": [
            "The selected authorization.role is invalid."
        ],
        "users.0.email": [
            "The users.0.email field is required."
        ],
        "users.2.email": [
            "The users.2.email must be a valid email address."
        ]
    }
}
```

<a name="form-request-validation"></a>
## Walidacja żądań formularzy

<a name="creating-form-requests"></a>
### Tworzenie żądań formularzy

Dla bardziej złożonych scenariuszy walidacji możesz chcieć utworzyć "żądanie formularza". Żądania formularzy to niestandardowe klasy żądań, które hermetyzują własną logikę walidacji i autoryzacji. Aby utworzyć klasę żądania formularza, możesz użyć polecenia CLI Artisan `make:request`:

```shell
php artisan make:request StorePostRequest
```

Wygenerowana klasa żądania formularza zostanie umieszczona w katalogu `app/Http/Requests`. Jeśli ten katalog nie istnieje, zostanie utworzony podczas uruchamiania polecenia `make:request`. Każde żądanie formularza wygenerowane przez Laravel ma dwie metody: `authorize` i `rules`.

Jak możesz się domyślić, metoda `authorize` jest odpowiedzialna za określenie, czy aktualnie uwierzytelniony użytkownik może wykonać akcję reprezentowaną przez żądanie, podczas gdy metoda `rules` zwraca reguły walidacji, które powinny być zastosowane do danych żądania:

```php
/**
 * Get the validation rules that apply to the request.
 *
 * @return array<string, \Illuminate\Contracts\Validation\ValidationRule|array<mixed>|string>
 */
public function rules(): array
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
```

> [!NOTE]
> Możesz wpisać podpowiedź typu dla dowolnych zależności wymaganych w sygnaturze metody `rules`. Zostaną one automatycznie rozwiązane przez [kontener usług](/docs/{{version}}/container) Laravel.

Jak więc są oceniane reguły walidacji? Wszystko, co musisz zrobić, to wpisać podpowiedź typu dla żądania w metodzie kontrolera. Przychodzące żądanie formularza jest walidowane przed wywołaniem metody kontrolera, co oznacza, że nie musisz zaśmiecać kontrolera żadną logiką walidacji:

```php
/**
 * Zapisz nowy post na blogu.
 */
public function store(StorePostRequest $request): RedirectResponse
{
    // Przychodzące żądanie jest prawidłowe...

    // Pobierz zwalidowane dane wejściowe...
    $validated = $request->validated();

    // Pobierz część zwalidowanych danych wejściowych...
    $validated = $request->safe()->only(['name', 'email']);
    $validated = $request->safe()->except(['name', 'email']);

    // Zapisz post na blogu...

    return redirect('/posts');
}
```

Jeśli walidacja nie powiedzie się, zostanie wygenerowana odpowiedź przekierowania, aby wysłać użytkownika z powrotem do ich poprzedniej lokalizacji. Błędy również zostaną przeniesione do sesji, aby były dostępne do wyświetlenia. Jeśli żądanie było żądaniem XHR, odpowiedź HTTP z kodem statusu 422 zostanie zwrócona użytkownikowi, w tym [reprezentacja JSON błędów walidacji](#validation-error-response-format).

> [!NOTE]
> Potrzebujesz dodać walidację żądań formularzy w czasie rzeczywistym do frontendu Laravel opartego na Inertia? Sprawdź [Laravel Precognition](/docs/{{version}}/precognition).

<a name="performing-additional-validation-on-form-requests"></a>
#### Wykonywanie dodatkowej walidacji

Czasami musisz wykonać dodatkową walidację po zakończeniu początkowej walidacji. Możesz to osiągnąć, używając metody `after` żądania formularza.

Metoda `after` powinna zwrócić tablicę wywoływalnych funkcji lub domknięć, które zostaną wywołane po zakończeniu walidacji. Te funkcje otrzymają instancję `Illuminate\Validation\Validator`, pozwalając Ci zgłosić dodatkowe komunikaty błędów, jeśli to konieczne:

```php
use Illuminate\Validation\Validator;

/**
 * Get the "after" validation callables for the request.
 */
public function after(): array
{
    return [
        function (Validator $validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add(
                    'field',
                    'Coś jest nie tak z tym polem!'
                );
            }
        }
    ];
}
```

Jak wspomniano, tablica zwrócona przez metodę `after` może również zawierać klasy wywoływalne. Metoda `__invoke` tych klas otrzyma instancję `Illuminate\Validation\Validator`:

```php
use App\Validation\ValidateShippingTime;
use App\Validation\ValidateUserStatus;
use Illuminate\Validation\Validator;

/**
 * Get the "after" validation callables for the request.
 */
public function after(): array
{
    return [
        new ValidateUserStatus,
        new ValidateShippingTime,
        function (Validator $validator) {
            //
        }
    ];
}
```

<a name="request-stopping-on-first-validation-rule-failure"></a>
#### Zatrzymanie przy pierwszym błędzie walidacji

Dodając właściwość `stopOnFirstFailure` do klasy żądania, możesz poinformować walidator, że powinien przestać walidować wszystkie atrybuty, gdy wystąpi pojedynczy błąd walidacji:

```php
/**
 * Określa, czy walidator powinien zatrzymać się przy pierwszym błędzie reguły.
 *
 * @var bool
 */
protected $stopOnFirstFailure = true;
```

<a name="customizing-the-redirect-location"></a>
#### Dostosowywanie lokalizacji przekierowania

Gdy walidacja żądania formularza zawiedzie, zostanie wygenerowana odpowiedź przekierowania, aby wysłać użytkownika z powrotem do jego poprzedniej lokalizacji. Możesz jednak dowolnie dostosować to zachowanie. W tym celu zdefiniuj właściwość `$redirect` w swoim żądaniu formularza:

```php
/**
 * URI, do którego użytkownicy powinni zostać przekierowani, jeśli walidacja zawiedzie.
 *
 * @var string
 */
protected $redirect = '/dashboard';
```

Lub, jeśli chcesz przekierować użytkowników do nazwanej trasy, możesz zamiast tego zdefiniować właściwość `$redirectRoute`:

```php
/**
 * Trasa, do której użytkownicy powinni zostać przekierowani, jeśli walidacja zawiedzie.
 *
 * @var string
 */
protected $redirectRoute = 'dashboard';
```

<a name="authorizing-form-requests"></a>
### Autoryzacja żądań formularzy

Klasa żądania formularza zawiera również metodę `authorize`. W tej metodzie możesz określić, czy uwierzytelniony użytkownik rzeczywiście ma uprawnienia do aktualizacji danego zasobu. Na przykład możesz określić, czy użytkownik rzeczywiście jest właścicielem komentarza na blogu, który próbuje zaktualizować. Najprawdopodobniej będziesz wchodzić w interakcje ze swoimi [bramami i politykami autoryzacji](/docs/{{version}}/authorization) w tej metodzie:

```php
use App\Models\Comment;

/**
 * Determine if the user is authorized to make this request.
 */
public function authorize(): bool
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```

Ponieważ wszystkie żądania formularzy rozszerzają bazową klasę żądania Laravel, możemy użyć metody `user`, aby uzyskać dostęp do aktualnie uwierzytelnionego użytkownika. Zauważ również wywołanie metody `route` w powyższym przykładzie. Ta metoda daje Ci dostęp do parametrów URI zdefiniowanych w wywoływanej trasie, takich jak parametr `{comment}` w poniższym przykładzie:

```php
Route::post('/comment/{comment}');
```

Dlatego, jeśli Twoja aplikacja korzysta z [wiązania modelu trasy](/docs/{{version}}/routing#route-model-binding), Twój kod może być jeszcze bardziej zwięzły, uzyskując dostęp do rozwiązanego modelu jako właściwości żądania:

```php
return $this->user()->can('update', $this->comment);
```

Jeśli metoda `authorize` zwróci `false`, odpowiedź HTTP z kodem statusu 403 zostanie automatycznie zwrócona, a metoda kontrolera nie zostanie wykonana.

Jeśli planujesz obsługiwać logikę autoryzacji dla żądania w innej części aplikacji, możesz całkowicie usunąć metodę `authorize` lub po prostu zwrócić `true`:

```php
/**
 * Determine if the user is authorized to make this request.
 */
public function authorize(): bool
{
    return true;
}
```

> [!NOTE]
> Możesz wpisać podpowiedź typu dla dowolnych zależności potrzebnych w sygnaturze metody `authorize`. Zostaną one automatycznie rozwiązane przez [kontener usług](/docs/{{version}}/container) Laravel.

<a name="customizing-the-error-messages"></a>
### Dostosowywanie komunikatów błędów

Możesz dostosować komunikaty błędów używane przez żądanie formularza, nadpisując metodę `messages`. Ta metoda powinna zwrócić tablicę par atrybut / reguła i odpowiadających im komunikatów błędów:

```php
/**
 * Get the error messages for the defined validation rules.
 *
 * @return array<string, string>
 */
public function messages(): array
{
    return [
        'title.required' => 'A title is required',
        'body.required' => 'A message is required',
    ];
}
```

<a name="customizing-the-validation-attributes"></a>
#### Dostosowywanie atrybutów walidacji

Wiele wbudowanych komunikatów błędów reguł walidacji Laravel zawiera placeholder `:attribute`. Jeśli chcesz, aby placeholder `:attribute` Twojego komunikatu walidacji został zastąpiony niestandardową nazwą atrybutu, możesz określić niestandardowe nazwy, nadpisując metodę `attributes`. Ta metoda powinna zwrócić tablicę par atrybut / nazwa:

```php
/**
 * Get custom attributes for validator errors.
 *
 * @return array<string, string>
 */
public function attributes(): array
{
    return [
        'email' => 'email address',
    ];
}
```

<a name="preparing-input-for-validation"></a>
### Przygotowanie danych wejściowych do walidacji

Jeśli musisz przygotować lub oczyścić jakiekolwiek dane z żądania przed zastosowaniem reguł walidacji, możesz użyć metody `prepareForValidation`:

```php
use Illuminate\Support\Str;

/**
 * Prepare the data for validation.
 */
protected function prepareForValidation(): void
{
    $this->merge([
        'slug' => Str::slug($this->slug),
    ]);
}
```

Podobnie, jeśli musisz znormalizować jakiekolwiek dane żądania po zakończeniu walidacji, możesz użyć metody `passedValidation`:

```php
/**
 * Handle a passed validation attempt.
 */
protected function passedValidation(): void
{
    $this->replace(['name' => 'Taylor']);
}
```

<a name="manually-creating-validators"></a>
## Ręczne tworzenie walidatorów

Jeśli nie chcesz używać metody `validate` na żądaniu, możesz utworzyć instancję walidatora ręcznie, używając [fasady](/docs/{{version}}/facades) `Validator`. Metoda `make` w fasadzie generuje nową instancję walidatora:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class PostController extends Controller
{
    /**
     * Zapisz nowy post na blogu.
     */
    public function store(Request $request): RedirectResponse
    {
        $validator = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        if ($validator->fails()) {
            return redirect('/post/create')
                ->withErrors($validator)
                ->withInput();
        }

        // Pobierz zwalidowane dane wejściowe...
        $validated = $validator->validated();

        // Pobierz część zwalidowanych danych wejściowych...
        $validated = $validator->safe()->only(['name', 'email']);
        $validated = $validator->safe()->except(['name', 'email']);

        // Zapisz post na blogu...

        return redirect('/posts');
    }
}
```

The first argument passed to the `make` method is the data under validation. The second argument is an array of the validation rules that should be applied to the data.

After determining whether the request validation failed, you may use the `withErrors` method to flash the error messages to the session. When using this method, the `$errors` variable zostaną automatycznie shared with your views after redirection, allowing you to easily display them back to the user. The `withErrors` method accepts a validator, a `MessageBag`, or a PHP `array`.

#### Zatrzymanie przy pierwszym błędzie walidacji

The `stopOnFirstFailure` method will inform the validator that it should stop validating all attributes once a single validation failure has occurred:

```php
if ($validator->stopOnFirstFailure()->fails()) {
    // ...
}
```

<a name="automatic-redirection"></a>
### Automatyczne przekierowanie

If you would like to create a validator instance manually but still take advantage of the automatic redirection offered by the HTTP request's `validate` method, you may call the `validate` method on an existing validator instance. If validation fails, the user zostaną automatycznie redirected or, in the case of an XHR request, a [JSON response will be returned](#validation-error-response-format):

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validate();
```

You may use the `validateWithBag` method to store the error messages in a [nazwanym worku błędów](#named-error-bags) if validation fails:

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validateWithBag('post');
```

<a name="named-error-bags"></a>
### Nazwane worki błędów

If you have multiple forms on a single page, you may wish to name the `MessageBag` containing the validation errors, allowing you to retrieve the error messages for a specific form. To achieve this, pass a name as the second argument to `withErrors`:

```php
return redirect('/register')->withErrors($validator, 'login');
```

You may then access the named `MessageBag` instance from the `$errors` variable:

```blade
{{ $errors->login->first('email') }}
```

<a name="manual-customizing-the-error-messages"></a>
### Dostosowywanie komunikatów błędów

If needed, you may provide custom error messages that a validator instance should use instead of the default error messages provided by Laravel. There are several ways to specify custom messages. First, you may pass the custom messages as the third argument to the `Validator::make` metody:

```php
$validator = Validator::make($input, $rules, $messages = [
    'required' => 'The :attribute field is required.',
]);
```

In this example, the `:attribute` placeholder will be replaced by the actual name of the field under validation. You may also utilize other placeholders in validation messages. For example:

```php
$messages = [
    'same' => 'The :attribute and :other must match.',
    'size' => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute value :input is not between :min - :max.',
    'in' => 'The :attribute must be one of the following types: :values',
];
```

<a name="specifying-a-custom-message-for-a-given-attribute"></a>
#### Określanie niestandardowego komunikatu dla danego atrybutu

Sometimes you may wish to specify a custom error message only for a specific attribute. You may do so using "dot" notation. Specify the attribute's name first, followed by the rule:

```php
$messages = [
    'email.required' => 'We need to know your email address!',
];
```

<a name="specifying-custom-attribute-values"></a>
#### Określanie niestandardowych wartości atrybutów

Many of Laravel's built-in error messages include an `:attribute` placeholder that is replaced with the name of the field or attribute under validation. To customize the values used to replace these placeholders for specific fields, you may pass an array of custom attributes as the fourth argument to the `Validator::make` metody:

```php
$validator = Validator::make($input, $rules, $messages, [
    'email' => 'email address',
]);
```

<a name="performing-additional-validation"></a>
### Wykonywanie dodatkowej walidacji

Sometimes you need to perform additional validation after your initial validation is complete. You can accomplish this using the validator's `after` method. The `after` method accepts a closure or an array of callables which will be invoked after validation is complete. The given callables will receive an `Illuminate\Validation\Validator` instance, allowing you to raise additional error messages if necessary:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make(/* ... */);

$validator->after(function ($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add(
            'field', 'Something is wrong with this field!'
        );
    }
});

if ($validator->fails()) {
    // ...
}
```

As noted, the `after` method also accepts an array of callables, which is particularly convenient if your "after validation" logic is encapsulated in invokable classes, which will receive an `Illuminate\Validation\Validator` instance via their `__invoke` metody:

```php
use App\Validation\ValidateShippingTime;
use App\Validation\ValidateUserStatus;

$validator->after([
    new ValidateUserStatus,
    new ValidateShippingTime,
    function ($validator) {
        // ...
    },
]);
```

<a name="working-with-validated-input"></a>
## Praca z zwalidowanymi danymi wejściowymi

After validating incoming request data using a form request or a manually created validator instance, you may wish to retrieve the incoming request data that actually underwent validation. This can be accomplished in several ways. First, you may call the `validated` method on a form request or validator instance. This method returns an array of the data that was validated:

```php
$validated = $request->validated();

$validated = $validator->validated();
```

Alternatively, you may call the `safe` method on a form request or validator instance. This method returns an instance of `Illuminate\Support\ValidatedInput`. This object exposes `only`, `except`, and `all` methods to retrieve a subset of the validated data or the entire array of validated data:

```php
$validated = $request->safe()->only(['name', 'email']);

$validated = $request->safe()->except(['name', 'email']);

$validated = $request->safe()->all();
```

In addition, the `Illuminate\Support\ValidatedInput` instance may be iterated over and accessed like an array:

```php
// Validated data may be iterated...
foreach ($request->safe() as $key => $value) {
    // ...
}

// Validated data may be accessed as an array...
$validated = $request->safe();

$email = $validated['email'];
```

If you would like to add additional fields to the validated data, you may call the `merge` metody:

```php
$validated = $request->safe()->merge(['name' => 'Taylor Otwell']);
```

If you would like to retrieve the validated data as a [collection](/docs/{{version}}/collections) instance, you may call the `collect` metody:

```php
$collection = $request->safe()->collect();
```

<a name="working-with-error-messages"></a>
## Praca z komunikatami błędów

Po wywołaniu metody `errors` na instancji `Validator` otrzymasz instancję `Illuminate\Support\MessageBag`, która ma różnorodne wygodne metody do pracy z komunikatami błędów. Zmienna `$errors`, która jest automatycznie udostępniana wszystkim widokom, również jest instancją klasy `MessageBag`.

<a name="retrieving-the-first-error-message-for-a-field"></a>
#### Pobieranie pierwszego komunikatu błędu dla pola

Aby pobrać pierwszy komunikat błędu dla danego pola, użyj metody `first`:

```php
$errors = $validator->errors();

echo $errors->first('email');
```

<a name="retrieving-all-error-messages-for-a-field"></a>
#### Pobieranie wszystkich komunikatów błędów dla pola

Jeśli musisz pobrać tablicę wszystkich komunikatów dla danego pola, użyj metody `get`:

```php
foreach ($errors->get('email') as $message) {
    // ...
}
```

Jeśli walidujesz pole formularza będące tablicą, możesz pobrać wszystkie komunikaty dla każdego elementu tablicy, używając znaku `*`:

```php
foreach ($errors->get('attachments.*') as $message) {
    // ...
}
```

<a name="retrieving-all-error-messages-for-all-fields"></a>
#### Pobieranie wszystkich komunikatów błędów dla wszystkich pól

Aby pobrać tablicę wszystkich komunikatów dla wszystkich pól, użyj metody `all`:

```php
foreach ($errors->all() as $message) {
    // ...
}
```

<a name="determining-if-messages-exist-for-a-field"></a>
#### Sprawdzanie, czy istnieją komunikaty dla pola

Metoda `has` może być użyta do określenia, czy istnieją komunikaty błędów dla danego pola:

```php
if ($errors->has('email')) {
    // ...
}
```

<a name="specifying-custom-messages-in-language-files"></a>
### Określanie niestandardowych komunikatów w plikach językowych

Każda wbudowana reguła walidacji Laravel ma komunikat błędu, który znajduje się w pliku `lang/en/validation.php` Twojej aplikacji. Jeśli Twoja aplikacja nie ma katalogu `lang`, możesz polecić Laravel, aby go utworzył, używając polecenia Artisan `lang:publish`.

W pliku `lang/en/validation.php` znajdziesz wpis tłumaczenia dla każdej reguły walidacji. Możesz swobodnie zmieniać lub modyfikować te komunikaty w zależności od potrzeb Twojej aplikacji.

Ponadto możesz skopiować ten plik do innego katalogu językowego, aby przetłumaczyć komunikaty na język Twojej aplikacji. Aby dowiedzieć się więcej o lokalizacji Laravel, zapoznaj się z kompletną [dokumentacją lokalizacji](/docs/{{version}}/localization).

> [!WARNING]
> Domyślnie szkielet aplikacji Laravel nie zawiera katalogu `lang`. Jeśli chcesz dostosować pliki językowe Laravel, możesz je opublikować za pomocą polecenia Artisan `lang:publish`.

<a name="custom-messages-for-specific-attributes"></a>
#### Niestandardowe komunikaty dla określonych atrybutów

Możesz dostosować komunikaty błędów używane dla określonych kombinacji atrybutów i reguł w plikach językowych walidacji Twojej aplikacji. Aby to zrobić, dodaj swoje dostosowania komunikatów do tablicy `custom` w pliku językowym `lang/xx/validation.php` Twojej aplikacji:

```php
'custom' => [
    'email' => [
        'required' => 'Musimy znać Twój adres e-mail!',
        'max' => 'Twój adres e-mail jest za długi!'
    ],
],
```

<a name="specifying-attribute-in-language-files"></a>
### Określanie atrybutów w plikach językowych

Wiele wbudowanych komunikatów błędów Laravel zawiera placeholder `:attribute`, który jest zastępowany nazwą pola lub atrybutu poddawanego walidacji. Jeśli chcesz, aby część `:attribute` Twojego komunikatu walidacji została zastąpiona niestandardową wartością, możesz określić niestandardową nazwę atrybutu w tablicy `attributes` pliku językowego `lang/xx/validation.php`:

```php
'attributes' => [
    'email' => 'adres e-mail',
],
```

> [!WARNING]
> Domyślnie szkielet aplikacji Laravel nie zawiera katalogu `lang`. Jeśli chcesz dostosować pliki językowe Laravel, możesz je opublikować za pomocą polecenia Artisan `lang:publish`.

<a name="specifying-values-in-language-files"></a>
### Określanie wartości w plikach językowych

Niektóre wbudowane komunikaty błędów reguł walidacji Laravel zawierają placeholder `:value`, który jest zastępowany bieżącą wartością atrybutu żądania. Jednak czasami możesz potrzebować, aby część `:value` Twojego komunikatu walidacji została zastąpiona niestandardową reprezentacją wartości. Na przykład rozważ następującą regułę, która określa, że numer karty kredytowej jest wymagany, jeśli `payment_type` ma wartość `cc`:

```php
Validator::make($request->all(), [
    'credit_card_number' => 'required_if:payment_type,cc'
]);
```

Jeśli ta reguła walidacji zawiedzie, wyprodukuje następujący komunikat błędu:

```text
Pole numer karty kredytowej jest wymagane, gdy typ płatności to cc.
```

Zamiast wyświetlać `cc` jako wartość typu płatności, możesz określić bardziej przyjazną dla użytkownika reprezentację wartości w pliku językowym `lang/xx/validation.php`, definiując tablicę `values`:

```php
'values' => [
    'payment_type' => [
        'cc' => 'karta kredytowa'
    ],
],
```

> [!WARNING]
> Domyślnie szkielet aplikacji Laravel nie zawiera katalogu `lang`. Jeśli chcesz dostosować pliki językowe Laravel, możesz je opublikować za pomocą polecenia Artisan `lang:publish`.

Po zdefiniowaniu tej wartości, reguła walidacji wyprodukuje następujący komunikat błędu:

```text
Pole numer karty kredytowej jest wymagane, gdy typ płatności to karta kredytowa.
```

<a name="available-validation-rules"></a>
## Dostępne reguły walidacji

Poniżej znajduje się lista wszystkich dostępnych reguł walidacji i ich funkcji:

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

#### Booleans

<div class="collection-method-list" markdown="1">

[Accepted](#rule-accepted)
[Accepted If](#rule-accepted-if)
[Boolean](#rule-boolean)
[Declined](#rule-declined)
[Declined If](#rule-declined-if)

</div>

#### Strings

<div class="collection-method-list" markdown="1">

[Active URL](#rule-active-url)
[Alpha](#rule-alpha)
[Alpha Dash](#rule-alpha-dash)
[Alpha Numeric](#rule-alpha-num)
[Ascii](#rule-ascii)
[Confirmed](#rule-confirmed)
[Current Password](#rule-current-password)
[Different](#rule-different)
[Doesnt Start With](#rule-doesnt-start-with)
[Doesnt End With](#rule-doesnt-end-with)
[Email](#rule-email)
[Ends With](#rule-ends-with)
[Enum](#rule-enum)
[Hex Color](#rule-hex-color)
[In](#rule-in)
[IP Address](#rule-ip)
[JSON](#rule-json)
[Lowercase](#rule-lowercase)
[MAC Address](#rule-mac)
[Max](#rule-max)
[Min](#rule-min)
[Not In](#rule-not-in)
[Regular Expression](#rule-regex)
[Not Regular Expression](#rule-not-regex)
[Same](#rule-same)
[Size](#rule-size)
[Starts With](#rule-starts-with)
[String](#rule-string)
[Uppercase](#rule-uppercase)
[URL](#rule-url)
[ULID](#rule-ulid)
[UUID](#rule-uuid)

</div>

#### Liczby

<div class="collection-method-list" markdown="1">

[Between](#rule-between)
[Decimal](#rule-decimal)
[Different](#rule-different)
[Digits](#rule-digits)
[Digits Between](#rule-digits-between)
[Greater Than](#rule-gt)
[Greater Than Or Equal](#rule-gte)
[Integer](#rule-integer)
[Less Than](#rule-lt)
[Less Than Or Equal](#rule-lte)
[Max](#rule-max)
[Max Digits](#rule-max-digits)
[Min](#rule-min)
[Min Digits](#rule-min-digits)
[Multiple Of](#rule-multiple-of)
[Numeric](#rule-numeric)
[Same](#rule-same)
[Size](#rule-size)

</div>

#### Tablice

<div class="collection-method-list" markdown="1">

[Array](#rule-array)
[Between](#rule-between)
[Contains](#rule-contains)
[Doesnt Contain](#rule-doesnt-contain)
[Distinct](#rule-distinct)
[In Array](#rule-in-array)
[In Array Keys](#rule-in-array-keys)
[List](#rule-list)
[Max](#rule-max)
[Min](#rule-min)
[Size](#rule-size)

</div>

#### Daty

<div class="collection-method-list" markdown="1">

[After](#rule-after)
[After Or Equal](#rule-after-or-equal)
[Before](#rule-before)
[Before Or Equal](#rule-before-or-equal)
[Date](#rule-date)
[Date Equals](#rule-date-equals)
[Date Format](#rule-date-format)
[Different](#rule-different)
[Timezone](#rule-timezone)

</div>

#### Pliki

<div class="collection-method-list" markdown="1">

[Between](#rule-between)
[Dimensions](#rule-dimensions)
[Encoding](#rule-encoding)
[Extensions](#rule-extensions)
[File](#rule-file)
[Image](#rule-image)
[Max](#rule-max)
[MIME Types](#rule-mimetypes)
[MIME Type By File Extension](#rule-mimes)
[Size](#rule-size)

</div>

#### Baza danych

<div class="collection-method-list" markdown="1">

[Exists](#rule-exists)
[Unique](#rule-unique)

</div>

#### Narzędzia

<div class="collection-method-list" markdown="1">

[Any Of](#rule-anyof)
[Bail](#rule-bail)
[Exclude](#rule-exclude)
[Exclude If](#rule-exclude-if)
[Exclude Unless](#rule-exclude-unless)
[Exclude With](#rule-exclude-with)
[Exclude Without](#rule-exclude-without)
[Filled](#rule-filled)
[Missing](#rule-missing)
[Missing If](#rule-missing-if)
[Missing Unless](#rule-missing-unless)
[Missing With](#rule-missing-with)
[Missing With All](#rule-missing-with-all)
[Nullable](#rule-nullable)
[Present](#rule-present)
[Present If](#rule-present-if)
[Present Unless](#rule-present-unless)
[Present With](#rule-present-with)
[Present With All](#rule-present-with-all)
[Prohibited](#rule-prohibited)
[Prohibited If](#rule-prohibited-if)
[Prohibited If Accepted](#rule-prohibited-if-accepted)
[Prohibited If Declined](#rule-prohibited-if-declined)
[Prohibited Unless](#rule-prohibited-unless)
[Prohibits](#rule-prohibits)
[Required](#rule-required)
[Required If](#rule-required-if)
[Required If Accepted](#rule-required-if-accepted)
[Required If Declined](#rule-required-if-declined)
[Required Unless](#rule-required-unless)
[Required With](#rule-required-with)
[Required With All](#rule-required-with-all)
[Required Without](#rule-required-without)
[Required Without All](#rule-required-without-all)
[Required Array Keys](#rule-required-array-keys)
[Sometimes](#validating-when-present)

</div>

<a name="rule-accepted"></a>
#### accepted

Pole poddawane walidacji musi mieć wartość `"yes"`, `"on"`, `1`, `"1"`, `true` lub `"true"`. Jest to przydatne do walidacji akceptacji "Warunków korzystania z usługi" lub podobnych pól.

<a name="rule-accepted-if"></a>
#### accepted_if:anotherfield,value,...

Pole poddawane walidacji musi mieć wartość `"yes"`, `"on"`, `1`, `"1"`, `true` lub `"true"`, jeśli inne walidowane pole jest równe określonej wartości. Jest to przydatne do walidacji akceptacji "Warunków korzystania z usługi" lub podobnych pól.

<a name="rule-active-url"></a>
#### active_url

Pole poddawane walidacji musi posiadać prawidłowy rekord A lub AAAA zgodnie z funkcją PHP `dns_get_record`. Nazwa hosta z podanego adresu URL jest wyodrębniana za pomocą funkcji PHP `parse_url` przed przekazaniem do `dns_get_record`.

<a name="rule-after"></a>
#### after:_date_

Pole poddawane walidacji musi być wartością późniejszą niż podana data. Daty zostaną przekazane do funkcji PHP `strtotime` w celu konwersji na prawidłową instancję `DateTime`:

```php
'start_date' => 'required|date|after:tomorrow'
```

Zamiast przekazywać ciąg daty do ewaluacji przez `strtotime`, możesz określić inne pole do porównania z datą:

```php
'finish_date' => 'required|date|after:start_date'
```

Dla wygody, reguły oparte na datach mogą być konstruowane przy użyciu płynnego buildera reguł `date`:

```php
use Illuminate\Validation\Rule;

'start_date' => [
    'required',
    Rule::date()->after(today()->addDays(7)),
],
```

Metody `afterToday` i `todayOrAfter` mogą być użyte do płynnego wyrażenia, że data musi być późniejsza niż dzisiaj, lub dzisiaj lub później:

```php
'start_date' => [
    'required',
    Rule::date()->afterToday(),
],
```

<a name="rule-after-or-equal"></a>
#### after\_or\_equal:_date_

Pole poddawane walidacji musi być wartością późniejszą lub równą podanej dacie. Więcej informacji znajdziesz w regule [after](#rule-after).

Dla wygody, reguły oparte na datach mogą być konstruowane przy użyciu płynnego buildera reguł `date`:

```php
use Illuminate\Validation\Rule;

'start_date' => [
    'required',
    Rule::date()->afterOrEqual(today()->addDays(7)),
],
```

<a name="rule-anyof"></a>
#### anyOf

Reguła walidacji `Rule::anyOf` pozwala określić, że pole poddawane walidacji musi spełniać którykolwiek z podanych zestawów reguł walidacji. Na przykład, następująca reguła sprawdzi, czy pole `username` jest adresem e-mail lub ciągiem alfanumerycznym (zawierającym myślniki) o długości co najmniej 6 znaków:

```php
use Illuminate\Validation\Rule;

'username' => [
    'required',
    Rule::anyOf([
        ['string', 'email'],
        ['string', 'alpha_dash', 'min:6'],
    ]),
],
```

<a name="rule-alpha"></a>
#### alpha

Pole poddawane walidacji musi składać się wyłącznie ze znaków alfabetycznych Unicode zawartych w [\p{L}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=) i [\p{M}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=).

Aby ograniczyć tę regułę walidacji do znaków z zakresu ASCII (`a-z` i `A-Z`), możesz podać opcję `ascii` dla reguły walidacji:

```php
'username' => 'alpha:ascii',
```

<a name="rule-alpha-dash"></a>
#### alpha_dash

Pole poddawane walidacji musi składać się wyłącznie ze znaków alfanumerycznych Unicode zawartych w [\p{L}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=), [\p{M}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=), [\p{N}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AN%3A%5D&g=&i=), a także myślników ASCII (`-`) i podkreśleń ASCII (`_`).

Aby ograniczyć tę regułę walidacji do znaków z zakresu ASCII (`a-z`, `A-Z` i `0-9`), możesz podać opcję `ascii` dla reguły walidacji:

```php
'username' => 'alpha_dash:ascii',
```

<a name="rule-alpha-num"></a>
#### alpha_num

Pole poddawane walidacji musi składać się wyłącznie ze znaków alfanumerycznych Unicode zawartych w [\p{L}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=), [\p{M}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=) i [\p{N}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AN%3A%5D&g=&i=).

Aby ograniczyć tę regułę walidacji do znaków z zakresu ASCII (`a-z`, `A-Z` i `0-9`), możesz podać opcję `ascii` dla reguły walidacji:

```php
'username' => 'alpha_num:ascii',
```

<a name="rule-array"></a>
#### array

Pole poddawane walidacji musi być tablicą PHP `array`.

Kiedy dodatkowe wartości są przekazywane do reguły `array`, każdy klucz w tablicy wejściowej musi być obecny na liście wartości przekazanych do reguły. W poniższym przykładzie klucz `admin` w tablicy wejściowej jest nieprawidłowy, ponieważ nie znajduje się na liście wartości przekazanych do reguły `array`:

```php
use Illuminate\Support\Facades\Validator;

$input = [
    'user' => [
        'name' => 'Taylor Otwell',
        'username' => 'taylorotwell',
        'admin' => true,
    ],
];

Validator::make($input, [
    'user' => 'array:name,username',
]);
```

Ogólnie rzecz biorąc, powinieneś zawsze określać klucze tablicy, które mogą być obecne w twojej tablicy.

<a name="rule-ascii"></a>
#### ascii

Pole poddawane walidacji musi składać się wyłącznie ze znaków 7-bitowego ASCII.

<a name="rule-bail"></a>
#### bail

Zatrzymuje wykonywanie reguł walidacji dla pola po pierwszym błędzie walidacji.

Podczas gdy reguła `bail` zatrzyma walidację tylko określonego pola po napotkaniu błędu walidacji, metoda `stopOnFirstFailure` poinformuje walidator, że powinien zatrzymać walidację wszystkich atrybutów po wystąpieniu pojedynczego błędu walidacji:

```php
if ($validator->stopOnFirstFailure()->fails()) {
    // ...
}
```

<a name="rule-before"></a>
#### before:_date_

Pole poddawane walidacji musi być wartością wcześniejszą niż podana data. Daty zostaną przekazane do funkcji PHP `strtotime` w celu konwersji na prawidłową instancję `DateTime`. Dodatkowo, podobnie jak w regule [after](#rule-after), nazwa innego walidowanego pola może być podana jako wartość `date`.

Dla wygody, reguły oparte na datach mogą być również konstruowane przy użyciu płynnego buildera reguł `date`:

```php
use Illuminate\Validation\Rule;

'start_date' => [
    'required',
    Rule::date()->before(today()->subDays(7)),
],
```

Metody `beforeToday` i `todayOrBefore` mogą być użyte do płynnego wyrażenia, że data musi być wcześniejsza niż dzisiaj, lub dzisiaj lub wcześniej:

```php
'start_date' => [
    'required',
    Rule::date()->beforeToday(),
],
```

<a name="rule-before-or-equal"></a>
#### before\_or\_equal:_date_

Pole poddawane walidacji musi być wartością wcześniejszą lub równą podanej dacie. Daty zostaną przekazane do funkcji PHP `strtotime` w celu konwersji na prawidłową instancję `DateTime`. Dodatkowo, podobnie jak w regule [after](#rule-after), nazwa innego walidowanego pola może być podana jako wartość `date`.

Dla wygody, reguły oparte na datach mogą być również konstruowane przy użyciu płynnego buildera reguł `date`:

```php
use Illuminate\Validation\Rule;

'start_date' => [
    'required',
    Rule::date()->beforeOrEqual(today()->subDays(7)),
],
```

<a name="rule-between"></a>
#### between:_min_,_max_

Pole poddawane walidacji musi mieć rozmiar między podanymi wartościami _min_ i _max_ (włącznie). Ciągi znaków, liczby, tablice i pliki są oceniane w ten sam sposób co reguła [size](#rule-size).

<a name="rule-boolean"></a>
#### boolean

Pole poddawane walidacji musi móc być rzutowane na wartość boolean. Akceptowane wartości wejściowe to `true`, `false`, `1`, `0`, `"1"` i `"0"`.

Możesz użyć parametru `strict`, aby uznać pole za prawidłowe tylko wtedy, gdy jego wartość to `true` lub `false`:

```php
'foo' => 'boolean:strict'
```

<a name="rule-confirmed"></a>
#### confirmed

Pole poddawane walidacji musi mieć odpowiadające mu pole `{field}_confirmation`. Na przykład, jeśli walidowanym polem jest `password`, w danych wejściowych musi być obecne odpowiadające mu pole `password_confirmation`.

Możesz również przekazać własną nazwę pola potwierdzenia. Na przykład, `confirmed:repeat_username` będzie oczekiwać, że pole `repeat_username` będzie pasować do walidowanego pola.

<a name="rule-contains"></a>
#### contains:_foo_,_bar_,...

Pole poddawane walidacji musi być tablicą zawierającą wszystkie podane wartości parametrów. Ponieważ ta reguła często wymaga użycia `implode` na tablicy, metoda `Rule::contains` może być użyta do płynnego konstruowania reguły:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'roles' => [
        'required',
        'array',
        Rule::contains(['admin', 'editor']),
    ],
]);
```

<a name="rule-doesnt-contain"></a>
#### doesnt_contain:_foo_,_bar_,...

Pole poddawane walidacji musi być tablicą, która nie zawiera żadnej z podanych wartości parametrów. Ponieważ ta reguła często wymaga użycia `implode` na tablicy, metoda `Rule::doesntContain` może być użyta do płynnego konstruowania reguły:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'roles' => [
        'required',
        'array',
        Rule::doesntContain(['admin', 'editor']),
    ],
]);
```

<a name="rule-current-password"></a>
#### current_password

Pole poddawane walidacji musi pasować do hasła uwierzytelnionego użytkownika. Możesz określić [strażnika uwierzytelniania](/docs/{{version}}/authentication) używając pierwszego parametru reguły:

```php
'password' => 'current_password:api'
```

<a name="rule-date"></a>
#### date

Pole poddawane walidacji musi być prawidłową, nierelatywną datą zgodną z funkcją PHP `strtotime`.

<a name="rule-date-equals"></a>
#### date_equals:_date_

Pole poddawane walidacji musi być równe podanej dacie. Daty zostaną przekazane do funkcji PHP `strtotime` w celu konwersji na prawidłową instancję `DateTime`.

<a name="rule-date-format"></a>
#### date_format:_format_,...

Pole poddawane walidacji musi pasować do jednego z podanych formatów _formats_. Podczas walidacji pola powinieneś użyć **albo** `date` **albo** `date_format`, nie obu. Ta reguła walidacji obsługuje wszystkie formaty obsługiwane przez klasę PHP [DateTime](https://www.php.net/manual/en/class.datetime.php).

Dla wygody, reguły oparte na datach mogą być konstruowane przy użyciu płynnego buildera reguł `date`:

```php
use Illuminate\Validation\Rule;

'start_date' => [
    'required',
    Rule::date()->format('Y-m-d'),
],
```

<a name="rule-decimal"></a>
#### decimal:_min_,_max_

Pole poddawane walidacji musi być numeryczne i musi zawierać określoną liczbę miejsc dziesiętnych:

```php
// Must have exactly two decimal places (9.99)...
'price' => 'decimal:2'

// Must have between 2 and 4 decimal places...
'price' => 'decimal:2,4'
```

<a name="rule-declined"></a>
#### declined

Pole poddawane walidacji musi mieć wartość `"no"`, `"off"`, `0`, `"0"`, `false` lub `"false"`.

<a name="rule-declined-if"></a>
#### declined_if:anotherfield,value,...

Pole poddawane walidacji musi mieć wartość `"no"`, `"off"`, `0`, `"0"`, `false` lub `"false"`, jeśli inne walidowane pole jest równe określonej wartości.

<a name="rule-different"></a>
#### different:_field_

Pole poddawane walidacji musi mieć inną wartość niż pole _field_.

<a name="rule-digits"></a>
#### digits:_value_

Liczba całkowita poddawana walidacji musi mieć dokładną długość _value_.

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

Liczba całkowita poddawana walidacji musi mieć długość między podanymi wartościami _min_ i _max_.

<a name="rule-dimensions"></a>
#### dimensions

Plik poddawany walidacji musi być obrazem spełniającym ograniczenia wymiarów określone przez parametry reguły:

```php
'avatar' => 'dimensions:min_width=100,min_height=200'
```

Dostępne ograniczenia to: _min\_width_, _max\_width_, _min\_height_, _max\_height_, _width_, _height_, _ratio_.

Ograniczenie _ratio_ powinno być reprezentowane jako szerokość podzielona przez wysokość. Może być określone jako ułamek jak `3/2` lub liczba zmiennoprzecinkowa jak `1.5`:

```php
'avatar' => 'dimensions:ratio=3/2'
```

Ponieważ ta reguła wymaga kilku argumentów, często wygodniej jest użyć metody `Rule::dimensions` do płynnego konstruowania reguły:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'avatar' => [
        'required',
        Rule::dimensions()
            ->maxWidth(1000)
            ->maxHeight(500)
            ->ratio(3 / 2),
    ],
]);
```

<a name="rule-distinct"></a>
#### distinct

Podczas walidacji tablic, pole poddawane walidacji nie może zawierać żadnych duplikatów wartości:

```php
'foo.*.id' => 'distinct'
```

Reguła distinct domyślnie używa luźnego porównywania zmiennych. Aby użyć ścisłego porównywania, możesz dodać parametr `strict` do definicji reguły walidacji:

```php
'foo.*.id' => 'distinct:strict'
```

Możesz dodać `ignore_case` do argumentów reguły walidacji, aby reguła ignorowała różnice w wielkości liter:

```php
'foo.*.id' => 'distinct:ignore_case'
```

<a name="rule-doesnt-start-with"></a>
#### doesnt_start_with:_foo_,_bar_,...

Pole poddawane walidacji nie może zaczynać się od żadnej z podanych wartości.

<a name="rule-doesnt-end-with"></a>
#### doesnt_end_with:_foo_,_bar_,...

Pole poddawane walidacji nie może kończyć się żadną z podanych wartości.

<a name="rule-email"></a>
#### email

Pole poddawane walidacji musi być sformatowane jako adres e-mail. Ta reguła walidacji wykorzystuje pakiet [egulias/email-validator](https://github.com/egulias/EmailValidator) do walidacji adresu e-mail. Domyślnie stosowany jest walidator `RFCValidation`, ale możesz również zastosować inne style walidacji:

```php
'email' => 'email:rfc,dns'
```

Powyższy przykład zastosuje walidacje `RFCValidation` i `DNSCheckValidation`. Oto pełna lista stylów walidacji, które możesz zastosować:

<div class="content-list" markdown="1">

- `rfc`: `RFCValidation` - Waliduje adres e-mail zgodnie z [obsługiwanymi RFC](https://github.com/egulias/EmailValidator?tab=readme-ov-file#supported-rfcs).
- `strict`: `NoRFCWarningsValidation` - Waliduje e-mail zgodnie z [obsługiwanymi RFC](https://github.com/egulias/EmailValidator?tab=readme-ov-file#supported-rfcs), niepowodzenie gdy znalezione są ostrzeżenia (np. kropki na końcu i wiele kolejnych kropek).
- `dns`: `DNSCheckValidation` - Zapewnia, że domena adresu e-mail ma prawidłowy rekord MX.
- `spoof`: `SpoofCheckValidation` - Zapewnia, że adres e-mail nie zawiera homografów ani zwodniczych znaków Unicode.
- `filter`: `FilterEmailValidation` - Zapewnia, że adres e-mail jest prawidłowy zgodnie z funkcją PHP `filter_var`.
- `filter_unicode`: `FilterEmailValidation::unicode()` - Zapewnia, że adres e-mail jest prawidłowy zgodnie z funkcją PHP `filter_var`, zezwalając na niektóre znaki Unicode.

</div>

Dla wygody, reguły walidacji e-mail mogą być budowane przy użyciu płynnego buildera reguł:

```php
use Illuminate\Validation\Rule;

$request->validate([
    'email' => [
        'required',
        Rule::email()
            ->rfcCompliant(strict: false)
            ->validateMxRecord()
            ->preventSpoofing()
    ],
]);
```

> [!WARNING]
> Walidatory `dns` i `spoof` wymagają rozszerzenia PHP `intl`.

<a name="rule-encoding"></a>
#### encoding:*encoding_type*

Pole poddawane walidacji musi odpowiadać określonemu kodowaniu znaków. Ta reguła używa funkcji PHP `mb_check_encoding` do weryfikacji kodowania podanego pliku lub wartości ciągu. Dla wygody, reguła `encoding` może być konstruowana przy użyciu płynnego buildera reguł plików Laravel:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rules\File;

Validator::validate($input, [
    'attachment' => [
        'required',
        File::types(['csv'])
            ->encoding('utf-8'),
    ],
]);
```

<a name="rule-ends-with"></a>
#### ends_with:_foo_,_bar_,...

Pole poddawane walidacji musi kończyć się jedną z podanych wartości.

<a name="rule-enum"></a>
#### enum

Reguła `Enum` jest regułą opartą na klasie, która waliduje, czy pole poddawane walidacji zawiera prawidłową wartość enum. Reguła `Enum` przyjmuje nazwę enum jako jedyny argument konstruktora. Podczas walidacji wartości prymitywnych, do reguły `Enum` należy przekazać wspierany Enum:

```php
use App\Enums\ServerStatus;
use Illuminate\Validation\Rule;

$request->validate([
    'status' => [Rule::enum(ServerStatus::class)],
]);
```

Metody `only` i `except` reguły `Enum` mogą być użyte do ograniczenia, które przypadki enum powinny być uznane za prawidłowe:

```php
Rule::enum(ServerStatus::class)
    ->only([ServerStatus::Pending, ServerStatus::Active]);

Rule::enum(ServerStatus::class)
    ->except([ServerStatus::Pending, ServerStatus::Active]);
```

Metoda `when` może być użyta do warunkowej modyfikacji reguły `Enum`:

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\Rule;

Rule::enum(ServerStatus::class)
    ->when(
        Auth::user()->isAdmin(),
        fn ($rule) => $rule->only(...),
        fn ($rule) => $rule->only(...),
    );
```

<a name="rule-exclude"></a>
#### exclude

Pole poddawane walidacji zostanie wykluczone z danych żądania zwracanych przez metody `validate` i `validated`.

<a name="rule-exclude-if"></a>
#### exclude_if:_anotherfield_,_value_

Pole poddawane walidacji zostanie wykluczone z danych żądania zwracanych przez metody `validate` i `validated`, jeśli pole _anotherfield_ jest równe _value_.

Jeśli wymagana jest złożona logika warunkowego wykluczania, możesz użyć metody `Rule::excludeIf`. Metoda ta przyjmuje wartość boolean lub domknięcie. Gdy podane jest domknięcie, powinno ono zwrócić `true` lub `false`, aby wskazać, czy pole poddawane walidacji powinno być wykluczone:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => Rule::excludeIf($request->user()->is_admin),
]);

Validator::make($request->all(), [
    'role_id' => Rule::excludeIf(fn () => $request->user()->is_admin),
]);
```

<a name="rule-exclude-unless"></a>
#### exclude_unless:_anotherfield_,_value_

Pole poddawane walidacji zostanie wykluczone z danych żądania zwracanych przez metody `validate` i `validated`, chyba że pole _anotherfield_ jest równe _value_. Jeśli _value_ jest `null` (`exclude_unless:name,null`), pole poddawane walidacji zostanie wykluczone, chyba że pole porównania jest `null` lub pole porównania nie występuje w danych żądania.

<a name="rule-exclude-with"></a>
#### exclude_with:_anotherfield_

Pole poddawane walidacji zostanie wykluczone z danych żądania zwracanych przez metody `validate` i `validated`, jeśli pole _anotherfield_ jest obecne.

<a name="rule-exclude-without"></a>
#### exclude_without:_anotherfield_

Pole poddawane walidacji zostanie wykluczone z danych żądania zwracanych przez metody `validate` i `validated`, jeśli pole _anotherfield_ nie jest obecne.

<a name="rule-exists"></a>
#### exists:_table_,_column_

Pole poddawane walidacji musi istnieć w podanej tabeli bazy danych.

<a name="basic-usage-of-exists-rule"></a>
#### Podstawowe użycie reguły Exists

```php
'state' => 'exists:states'
```

Jeśli opcja `column` nie jest określona, zostanie użyta nazwa pola. Tak więc w tym przypadku reguła sprawdzi, czy tabela bazy danych `states` zawiera rekord z wartością kolumny `state` pasującą do wartości atrybutu `state` żądania.

<a name="specifying-a-custom-column-name"></a>
#### Określanie niestandardowej nazwy kolumny

Możesz jawnie określić nazwę kolumny bazy danych, która powinna być użyta przez regułę walidacji, umieszczając ją po nazwie tabeli bazy danych:

```php
'state' => 'exists:states,abbreviation'
```

Od czasu do czasu może być konieczne określenie konkretnego połączenia z bazą danych, które ma być użyte dla zapytania `exists`. Możesz to osiągnąć, poprzedzając nazwę tabeli nazwą połączenia:

```php
'email' => 'exists:connection.staff,email'
```

Zamiast bezpośredniego określania nazwy tabeli, możesz określić model Eloquent, który powinien być użyty do określenia nazwy tabeli:

```php
'user_id' => 'exists:App\Models\User,id'
```

Jeśli chcesz dostosować zapytanie wykonywane przez regułę walidacji, możesz użyć klasy `Rule` do płynnego definiowania reguły. W tym przykładzie określimy również reguły walidacji jako tablicę zamiast używać znaku `|` do ich rozdzielania:

```php
use Illuminate\Database\Query\Builder;
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::exists('staff')->where(function (Builder $query) {
            $query->where('account_id', 1);
        }),
    ],
]);
```

Możesz jawnie określić nazwę kolumny bazy danych, która powinna być użyta przez regułę `exists` wygenerowaną przez metodę `Rule::exists`, podając nazwę kolumny jako drugi argument metody `exists`:

```php
'state' => Rule::exists('states', 'abbreviation'),
```

Czasami możesz chcieć sprawdzić, czy tablica wartości istnieje w bazie danych. Możesz to zrobić, dodając zarówno regułę `exists`, jak i [array](#rule-array) do walidowanego pola:

```php
'states' => ['array', Rule::exists('states', 'abbreviation')],
```

Kiedy obie te reguły są przypisane do pola, Laravel automatycznie zbuduje pojedyncze zapytanie, aby określić, czy wszystkie podane wartości istnieją w określonej tabeli.

<a name="rule-extensions"></a>
#### extensions:_foo_,_bar_,...

Plik poddawany walidacji musi mieć przypisane przez użytkownika rozszerzenie odpowiadające jednemu z wymienionych rozszerzeń:

```php
'photo' => ['required', 'extensions:jpg,png'],
```

> [!WARNING]
> Nigdy nie powinieneś polegać wyłącznie na walidacji pliku po jego rozszerzeniu przypisanym przez użytkownika. Ta reguła powinna być zazwyczaj zawsze używana w połączeniu z regułami [mimes](#rule-mimes) lub [mimetypes](#rule-mimetypes).

<a name="rule-file"></a>
#### file

Pole poddawane walidacji musi być pomyślnie przesłanym plikiem.

<a name="rule-filled"></a>
#### filled

Pole poddawane walidacji nie może być puste, gdy jest obecne.

<a name="rule-gt"></a>
#### gt:_field_

Pole poddawane walidacji musi być większe niż podane pole _field_ lub wartość _value_. Oba pola muszą być tego samego typu. Ciągi znaków, liczby, tablice i pliki są oceniane przy użyciu tych samych konwencji co reguła [size](#rule-size).

<a name="rule-gte"></a>
#### gte:_field_

Pole poddawane walidacji musi być większe lub równe podanemu polu _field_ lub wartości _value_. Oba pola muszą być tego samego typu. Ciągi znaków, liczby, tablice i pliki są oceniane przy użyciu tych samych konwencji co reguła [size](#rule-size).

<a name="rule-hex-color"></a>
#### hex_color

Pole poddawane walidacji musi zawierać prawidłową wartość koloru w formacie [szesnastkowym](https://developer.mozilla.org/en-US/docs/Web/CSS/hex-color).

<a name="rule-image"></a>
#### image

Plik poddawany walidacji musi być obrazem (jpg, jpeg, png, bmp, gif lub webp).

> [!WARNING]
> Domyślnie reguła image nie zezwala na pliki SVG ze względu na możliwość podatności XSS. Jeśli musisz zezwolić na pliki SVG, możesz dodać dyrektywę `allow_svg` do reguły `image` (`image:allow_svg`).

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

Pole poddawane walidacji musi być zawarte na podanej liście wartości. Ponieważ ta reguła często wymaga użycia `implode` na tablicy, metoda `Rule::in` może być użyta do płynnego konstruowania reguły:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'zones' => [
        'required',
        Rule::in(['first-zone', 'second-zone']),
    ],
]);
```

Kiedy reguła `in` jest łączona z regułą `array`, każda wartość w tablicy wejściowej musi być obecna na liście wartości przekazanych do reguły `in`. W poniższym przykładzie kod lotniska `LAS` w tablicy wejściowej jest nieprawidłowy, ponieważ nie znajduje się na liście lotnisk przekazanych do reguły `in`:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

$input = [
    'airports' => ['NYC', 'LAS'],
];

Validator::make($input, [
    'airports' => [
        'required',
        'array',
    ],
    'airports.*' => Rule::in(['NYC', 'LIT']),
]);
```

<a name="rule-in-array"></a>
#### in_array:_anotherfield_.*

Pole poddawane walidacji musi istnieć w wartościach pola _anotherfield_.

<a name="rule-in-array-keys"></a>
#### in_array_keys:_value_.*

Pole poddawane walidacji musi być tablicą posiadającą co najmniej jedną z podanych wartości _values_ jako klucz w tablicy:

```php
'config' => 'array|in_array_keys:timezone'
```

<a name="rule-integer"></a>
#### integer

Pole poddawane walidacji musi być liczbą całkowitą.

Możesz użyć parametru `strict`, aby uznać pole za prawidłowe tylko wtedy, gdy jego typ to `integer`. Ciągi znaków z wartościami całkowitymi będą uznane za nieprawidłowe:

```php
'age' => 'integer:strict'
```

> [!WARNING]
> Ta reguła walidacji nie weryfikuje, czy dane wejściowe są typu zmiennej "integer", tylko że dane wejściowe są typu akceptowanego przez regułę PHP `FILTER_VALIDATE_INT`. Jeśli musisz zwalidować dane wejściowe jako liczbę, użyj tej reguły w połączeniu z [regułą walidacji `numeric`](#rule-numeric).

<a name="rule-ip"></a>
#### ip

Pole poddawane walidacji musi być adresem IP.

<a name="ipv4"></a>
#### ipv4

Pole poddawane walidacji musi być adresem IPv4.

<a name="ipv6"></a>
#### ipv6

Pole poddawane walidacji musi być adresem IPv6.

<a name="rule-json"></a>
#### json

Pole poddawane walidacji musi być prawidłowym ciągiem JSON.

<a name="rule-lt"></a>
#### lt:_field_

Pole poddawane walidacji musi być mniejsze niż podane pole _field_. Oba pola muszą być tego samego typu. Ciągi znaków, liczby, tablice i pliki są oceniane przy użyciu tych samych konwencji co reguła [size](#rule-size).

<a name="rule-lte"></a>
#### lte:_field_

Pole poddawane walidacji musi być mniejsze lub równe podanemu polu _field_. Oba pola muszą być tego samego typu. Ciągi znaków, liczby, tablice i pliki są oceniane przy użyciu tych samych konwencji co reguła [size](#rule-size).

<a name="rule-lowercase"></a>
#### lowercase

Pole poddawane walidacji musi być pisane małymi literami.

<a name="rule-list"></a>
#### list

Pole poddawane walidacji musi być tablicą będącą listą. Tablica jest uważana za listę, jeśli jej klucze składają się z kolejnych liczb od 0 do `count($array) - 1`.

<a name="rule-mac"></a>
#### mac_address

Pole poddawane walidacji musi być adresem MAC.

<a name="rule-max"></a>
#### max:_value_

Pole poddawane walidacji musi być mniejsze lub równe maksymalnej wartości _value_. Ciągi znaków, liczby, tablice i pliki są oceniane w ten sam sposób co reguła [size](#rule-size).

<a name="rule-max-digits"></a>
#### max_digits:_value_

Liczba całkowita poddawana walidacji musi mieć maksymalną długość _value_.

<a name="rule-mimetypes"></a>
#### mimetypes:_text/plain_,...

Plik poddawany walidacji musi pasować do jednego z podanych typów MIME:

```php
'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'
```

Aby określić typ MIME przesłanego pliku, zawartość pliku zostanie odczytana, a framework spróbuje odgadnąć typ MIME, który może się różnić od typu MIME dostarczonego przez klienta.

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

Plik poddawany walidacji musi mieć typ MIME odpowiadający jednemu z wymienionych rozszerzeń:

```php
'photo' => 'mimes:jpg,bmp,png'
```

Chociaż musisz określić tylko rozszerzenia, ta reguła faktycznie waliduje typ MIME pliku poprzez odczytanie zawartości pliku i odgadnięcie jego typu MIME. Pełna lista typów MIME i odpowiadających im rozszerzeń znajduje się pod następującym adresem:

[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="mime-types-and-extensions"></a>
#### Typy MIME i rozszerzenia

Ta reguła walidacji nie weryfikuje zgodności między typem MIME a rozszerzeniem przypisanym przez użytkownika do pliku. Na przykład reguła walidacji `mimes:png` uznałaby plik zawierający prawidłową zawartość PNG za prawidłowy obraz PNG, nawet jeśli plik nosi nazwę `photo.txt`. Jeśli chcesz zwalidować rozszerzenie pliku przypisane przez użytkownika, możesz użyć reguły [extensions](#rule-extensions).

<a name="rule-min"></a>
#### min:_value_

Pole poddawane walidacji musi mieć minimalną wartość _value_. Ciągi znaków, liczby, tablice i pliki są oceniane w ten sam sposób co reguła [size](#rule-size).

<a name="rule-min-digits"></a>
#### min_digits:_value_

Liczba całkowita poddawana walidacji musi mieć minimalną długość _value_.

<a name="rule-multiple-of"></a>
#### multiple_of:_value_

Pole poddawane walidacji musi być wielokrotnością wartości _value_.

<a name="rule-missing"></a>
#### missing

Pole poddawane walidacji nie może być obecne w danych wejściowych.

<a name="rule-missing-if"></a>
#### missing_if:_anotherfield_,_value_,...

Pole poddawane walidacji nie może być obecne, jeśli pole _anotherfield_ jest równe jakiejkolwiek wartości _value_.

<a name="rule-missing-unless"></a>
#### missing_unless:_anotherfield_,_value_

Pole poddawane walidacji nie może być obecne, chyba że pole _anotherfield_ jest równe jakiejkolwiek wartości _value_.

<a name="rule-missing-with"></a>
#### missing_with:_foo_,_bar_,...

Pole poddawane walidacji nie może być obecne _tylko jeśli_ którekolwiek z innych określonych pól jest obecne.

<a name="rule-missing-with-all"></a>
#### missing_with_all:_foo_,_bar_,...

Pole poddawane walidacji nie może być obecne _tylko jeśli_ wszystkie inne określone pola są obecne.

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

Pole poddawane walidacji nie może być zawarte na podanej liście wartości. Metoda `Rule::notIn` może być użyta do płynnego konstruowania reguły:

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'toppings' => [
        'required',
        Rule::notIn(['sprinkles', 'cherries']),
    ],
]);
```

<a name="rule-not-regex"></a>
#### not_regex:_pattern_

Pole poddawane walidacji nie może pasować do podanego wyrażenia regularnego.

Wewnętrznie ta reguła używa funkcji PHP `preg_match`. Określony wzorzec powinien być zgodny z tym samym formatowaniem wymaganym przez `preg_match`, a zatem również zawierać prawidłowe ograniczniki. Na przykład: `'email' => 'not_regex:/^.+$/i'`.

> [!WARNING]
> Podczas używania wzorców `regex` / `not_regex` może być konieczne określenie reguł walidacji za pomocą tablicy zamiast używania separatorów `|`, szczególnie jeśli wyrażenie regularne zawiera znak `|`.

<a name="rule-nullable"></a>
#### nullable

Pole podlegające walidacji może być `null`.

<a name="rule-numeric"></a>
#### numeric

Pole podlegające walidacji musi być [numeryczne](https://www.php.net/manual/en/function.is-numeric.php).

Możesz użyć parametru `strict`, aby uznać pole za ważne tylko wtedy, gdy jego wartość jest typu integer lub float. Numeryczne ciągi znaków będą uważane za nieprawidłowe:

```php
'amount' => 'numeric:strict'
```

<a name="rule-present"></a>
#### present

Pole podlegające walidacji musi istnieć w danych wejściowych.

<a name="rule-present-if"></a>
#### present_if:_anotherfield_,_value_,...

Pole podlegające walidacji musi być obecne, jeśli pole _anotherfield_ jest równe dowolnej wartości _value_.

<a name="rule-present-unless"></a>
#### present_unless:_anotherfield_,_value_

Pole podlegające walidacji musi być obecne, chyba że pole _anotherfield_ jest równe dowolnej wartości _value_.

<a name="rule-present-with"></a>
#### present_with:_foo_,_bar_,...

Pole podlegające walidacji musi być obecne _tylko wtedy_, gdy obecne jest którekolwiek z pozostałych określonych pól.

<a name="rule-present-with-all"></a>
#### present_with_all:_foo_,_bar_,...

Pole podlegające walidacji musi być obecne _tylko wtedy_, gdy obecne są wszystkie pozostałe określone pola.

<a name="rule-prohibited"></a>
#### prohibited

Pole podlegające walidacji musi być nieobecne lub puste. Pole jest "puste", jeśli spełnia jedno z następujących kryteriów:

<div class="content-list" markdown="1">

- Wartość to `null`.
- Wartość to pusty ciąg znaków.
- Wartość to pusta tablica lub pusty obiekt `Countable`.
- Wartość to przesłany plik z pustą ścieżką.

</div>

<a name="rule-prohibited-if"></a>
#### prohibited_if:_anotherfield_,_value_,...

Pole podlegające walidacji musi być nieobecne lub puste, jeśli pole _anotherfield_ jest równe dowolnej wartości _value_. Pole jest "puste", jeśli spełnia jedno z następujących kryteriów:

<div class="content-list" markdown="1">

- Wartość to `null`.
- Wartość to pusty ciąg znaków.
- Wartość to pusta tablica lub pusty obiekt `Countable`.
- Wartość to przesłany plik z pustą ścieżką.

</div>

Jeśli wymagana jest złożona logika warunkowego zakazu, możesz użyć metody `Rule::prohibitedIf`. Ta metoda akceptuje wartość boolean lub domknięcie. Gdy podane jest domknięcie, powinno ono zwrócić `true` lub `false`, aby wskazać, czy pole podlegające walidacji powinno być zabronione:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => Rule::prohibitedIf($request->user()->is_admin),
]);

Validator::make($request->all(), [
    'role_id' => Rule::prohibitedIf(fn () => $request->user()->is_admin),
]);
```
<a name="rule-prohibited-if-accepted"></a>
#### prohibited_if_accepted:_anotherfield_,...

Pole podlegające walidacji musi być nieobecne lub puste, jeśli pole _anotherfield_ jest równe `"yes"`, `"on"`, `1`, `"1"`, `true` lub `"true"`.

<a name="rule-prohibited-if-declined"></a>
#### prohibited_if_declined:_anotherfield_,...

Pole podlegające walidacji musi być nieobecne lub puste, jeśli pole _anotherfield_ jest równe `"no"`, `"off"`, `0`, `"0"`, `false` lub `"false"`.

<a name="rule-prohibited-unless"></a>
#### prohibited_unless:_anotherfield_,_value_,...

Pole podlegające walidacji musi być nieobecne lub puste, chyba że pole _anotherfield_ jest równe dowolnej wartości _value_. Pole jest "puste", jeśli spełnia jedno z następujących kryteriów:

<div class="content-list" markdown="1">

- Wartość to `null`.
- Wartość to pusty ciąg znaków.
- Wartość to pusta tablica lub pusty obiekt `Countable`.
- Wartość to przesłany plik z pustą ścieżką.

</div>

<a name="rule-prohibits"></a>
#### prohibits:_anotherfield_,...

Jeśli pole podlegające walidacji nie jest nieobecne ani puste, wszystkie pola w _anotherfield_ muszą być nieobecne lub puste. Pole jest "puste", jeśli spełnia jedno z następujących kryteriów:

<div class="content-list" markdown="1">

- Wartość to `null`.
- Wartość to pusty ciąg znaków.
- Wartość to pusta tablica lub pusty obiekt `Countable`.
- Wartość to przesłany plik z pustą ścieżką.

</div>

<a name="rule-regex"></a>
#### regex:_pattern_

Pole podlegające walidacji musi pasować do podanego wyrażenia regularnego.

Wewnętrznie ta reguła używa funkcji PHP `preg_match`. Określony wzorzec powinien być zgodny z tym samym formatowaniem wymaganym przez `preg_match`, a zatem również zawierać prawidłowe ograniczniki. Na przykład: `'email' => 'regex:/^.+@.+$/i'`.

> [!WARNING]
> Podczas używania wzorców `regex` / `not_regex` może być konieczne określenie reguł w tablicy zamiast używania separatorów `|`, szczególnie jeśli wyrażenie regularne zawiera znak `|`.

<a name="rule-required"></a>
#### required

Pole podlegające walidacji musi być obecne w danych wejściowych i nie może być puste. Pole jest "puste", jeśli spełnia jedno z następujących kryteriów:

<div class="content-list" markdown="1">

- Wartość to `null`.
- Wartość to pusty ciąg znaków.
- Wartość to pusta tablica lub pusty obiekt `Countable`.
- Wartość to przesłany plik bez ścieżki.

</div>

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,...

Pole podlegające walidacji musi być obecne i niepuste, jeśli pole _anotherfield_ jest równe dowolnej wartości _value_.

Jeśli chcesz skonstruować bardziej złożony warunek dla reguły `required_if`, możesz użyć metody `Rule::requiredIf`. Ta metoda akceptuje wartość boolean lub domknięcie. Gdy przekazane jest domknięcie, powinno ono zwrócić `true` lub `false`, aby wskazać, czy pole podlegające walidacji jest wymagane:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => Rule::requiredIf($request->user()->is_admin),
]);

Validator::make($request->all(), [
    'role_id' => Rule::requiredIf(fn () => $request->user()->is_admin),
]);
```

<a name="rule-required-if-accepted"></a>
#### required_if_accepted:_anotherfield_,...

Pole podlegające walidacji musi być obecne i niepuste, jeśli pole _anotherfield_ jest równe `"yes"`, `"on"`, `1`, `"1"`, `true` lub `"true"`.

<a name="rule-required-if-declined"></a>
#### required_if_declined:_anotherfield_,...

Pole podlegające walidacji musi być obecne i niepuste, jeśli pole _anotherfield_ jest równe `"no"`, `"off"`, `0`, `"0"`, `false` lub `"false"`.

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

Pole podlegające walidacji musi być obecne i niepuste, chyba że pole _anotherfield_ jest równe dowolnej wartości _value_. Oznacza to również, że pole _anotherfield_ musi być obecne w danych żądania, chyba że wartość _value_ to `null`. Jeśli wartość _value_ to `null` (`required_unless:name,null`), pole podlegające walidacji będzie wymagane, chyba że pole porównania to `null` lub pole porównania brakuje w danych żądania.

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

Pole podlegające walidacji musi być obecne i niepuste _tylko wtedy_, gdy którekolwiek z pozostałych określonych pól są obecne i niepuste.

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

Pole podlegające walidacji musi być obecne i niepuste _tylko wtedy_, gdy wszystkie pozostałe określone pola są obecne i niepuste.

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

Pole podlegające walidacji musi być obecne i niepuste _tylko wtedy_, gdy którekolwiek z pozostałych określonych pól są puste lub nieobecne.

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

Pole podlegające walidacji musi być obecne i niepuste _tylko wtedy_, gdy wszystkie pozostałe określone pola są puste lub nieobecne.

<a name="rule-required-array-keys"></a>
#### required_array_keys:_foo_,_bar_,...

Pole podlegające walidacji musi być tablicą i musi zawierać co najmniej określone klucze.

<a name="rule-same"></a>
#### same:_field_

Podane pole _field_ musi pasować do pola podlegającego walidacji.

<a name="rule-size"></a>
#### size:_value_

Pole podlegające walidacji musi mieć rozmiar odpowiadający podanej wartości _value_. Dla danych tekstowych _value_ odpowiada liczbie znaków. Dla danych numerycznych _value_ odpowiada określonej wartości całkowitej (atrybut musi również mieć regułę `numeric` lub `integer`). Dla tablicy _size_ odpowiada liczbie elementów (`count`) tablicy. Dla plików _size_ odpowiada rozmiarowi pliku w kilobajtach. Spójrzmy na kilka przykładów:

```php
// Validate that a string is exactly 12 characters long...
'title' => 'size:12';

// Validate that a provided integer equals 10...
'seats' => 'integer|size:10';

// Validate that an array has exactly 5 elements...
'tags' => 'array|size:5';

// Validate that an uploaded file is exactly 512 kilobytes...
'image' => 'file|size:512';
```

<a name="rule-starts-with"></a>
#### starts_with:_foo_,_bar_,...

Pole podlegające walidacji musi zaczynać się od jednej z podanych wartości.

<a name="rule-string"></a>
#### string

Pole podlegające walidacji musi być ciągiem znaków. Jeśli chcesz również zezwolić, aby pole było `null`, powinieneś przypisać regułę `nullable` do pola.

<a name="rule-timezone"></a>
#### timezone

Pole podlegające walidacji musi być prawidłowym identyfikatorem strefy czasowej zgodnie z metodą `DateTimeZone::listIdentifiers`.

Argumenty [akceptowane przez metodę `DateTimeZone::listIdentifiers`](https://www.php.net/manual/en/datetimezone.listidentifiers.php) mogą być również przekazane do tej reguły walidacji:

```php
'timezone' => 'required|timezone:all';

'timezone' => 'required|timezone:Africa';

'timezone' => 'required|timezone:per_country,US';
```

<a name="rule-unique"></a>
#### unique:_table_,_column_

Pole podlegające walidacji nie może istnieć w podanej tabeli bazy danych.

**Określanie niestandardowej nazwy tabeli / kolumny:**

Zamiast bezpośrednio podawać nazwę tabeli, możesz określić model Eloquent, który powinien być użyty do określenia nazwy tabeli:

```php
'email' => 'unique:App\Models\User,email_address'
```

Opcja `column` może być użyta do określenia odpowiadającej kolumny bazy danych dla pola. Jeśli opcja `column` nie jest określona, zostanie użyta nazwa pola podlegającego walidacji.

```php
'email' => 'unique:users,email_address'
```

**Określanie niestandardowego połączenia z bazą danych**

Okazjonalnie możesz potrzebować ustawić niestandardowe połączenie dla zapytań do bazy danych wykonywanych przez Validator. Aby to osiągnąć, możesz dodać nazwę połączenia przed nazwą tabeli:

```php
'email' => 'unique:connection.users,email_address'
```

**Wymuszanie ignorowania określonego ID przez regułę unique:**

Niekiedy możesz chcieć zignorować określone ID podczas walidacji unikalności. Na przykład, rozważ ekran "aktualizacja profilu", który zawiera imię użytkownika, adres e-mail i lokalizację. Prawdopodobnie będziesz chciał zweryfikować, że adres e-mail jest unikalny. Jednak jeśli użytkownik zmieni tylko pole imienia, a nie pole e-mail, nie chcesz, aby został zgłoszony błąd walidacji, ponieważ użytkownik jest już właścicielem tego adresu e-mail.

Aby poinstruować walidator, aby zignorował ID użytkownika, użyjemy klasy `Rule` do płynnego zdefiniowania reguły. W tym przykładzie określimy również reguły walidacji jako tablicę zamiast używać znaku `|` do rozdzielania reguł:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::unique('users')->ignore($user->id),
    ],
]);
```

> [!WARNING]
> Nigdy nie powinieneś przekazywać żadnych danych wejściowych żądania kontrolowanych przez użytkownika do metody `ignore`. Zamiast tego powinieneś przekazać tylko wygenerowany przez system unikalny ID, taki jak automatycznie inkrementowany ID lub UUID z instancji modelu Eloquent. W przeciwnym razie Twoja aplikacja będzie podatna na atak SQL injection.

Zamiast przekazywać wartość klucza modelu do metody `ignore`, możesz również przekazać całą instancję modelu. Laravel automatycznie wyodrębni klucz z modelu:

```php
Rule::unique('users')->ignore($user)
```

Jeśli Twoja tabela używa nazwy kolumny klucza podstawowego innej niż `id`, możesz określić nazwę kolumny podczas wywoływania metody `ignore`:

```php
Rule::unique('users')->ignore($user->id, 'user_id')
```

Domyślnie reguła `unique` sprawdzi unikalność kolumny odpowiadającej nazwie walidowanego atrybutu. Jednak możesz przekazać inną nazwę kolumny jako drugi argument do metody `unique`:

```php
Rule::unique('users', 'email_address')->ignore($user->id)
```

**Dodawanie dodatkowych klauzul Where:**

Możesz określić dodatkowe warunki zapytania, dostosowując zapytanie za pomocą metody `where`. Na przykład, dodajmy warunek zapytania, który ogranicza zapytanie do przeszukiwania tylko rekordów, które mają wartość kolumny `account_id` równą `1`:

```php
'email' => Rule::unique('users')->where(fn (Builder $query) => $query->where('account_id', 1))
```

**Ignorowanie miękko usuniętych rekordów w sprawdzaniu unikalności:**

Domyślnie reguła unique uwzględnia miękko usunięte rekordy podczas określania unikalności. Aby wykluczyć miękko usunięte rekordy ze sprawdzania unikalności, możesz wywołać metodę `withoutTrashed`:

```php
Rule::unique('users')->withoutTrashed();
```

Jeśli Twój model używa nazwy kolumny innej niż `deleted_at` dla miękko usuniętych rekordów, możesz podać nazwę kolumny podczas wywoływania metody `withoutTrashed`:

```php
Rule::unique('users')->withoutTrashed('was_deleted_at');
```

<a name="rule-uppercase"></a>
#### uppercase

Pole podlegające walidacji musi być napisane wielkimi literami.

<a name="rule-url"></a>
#### url

Pole podlegające walidacji musi być prawidłowym adresem URL.

Jeśli chcesz określić protokoły URL, które powinny być uznane za prawidłowe, możesz przekazać protokoły jako parametry reguły walidacji:

```php
'url' => 'url:http,https',

'game' => 'url:minecraft,steam',
```

<a name="rule-ulid"></a>
#### ulid

Pole podlegające walidacji musi być prawidłowym [Uniwersalnie Unikalnym Identyfikatorem Sortowalnym Leksykograficznie](https://github.com/ulid/spec) (ULID).

<a name="rule-uuid"></a>
#### uuid

Pole podlegające walidacji musi być prawidłowym uniwersalnie unikalnym identyfikatorem (UUID) zgodnym z RFC 9562 (wersja 1, 3, 4, 5, 6, 7 lub 8).

Możesz również sprawdzić, czy podany UUID odpowiada specyfikacji UUID według wersji:

```php
'uuid' => 'uuid:4'
```

<a name="conditionally-adding-rules"></a>
## Warunkowe dodawanie reguł

<a name="skipping-validation-when-fields-have-certain-values"></a>
#### Pomijanie walidacji, gdy pola mają określone wartości

Możesz czasami chcieć nie walidować danego pola, jeśli inne pole ma określoną wartość. Możesz to osiągnąć za pomocą reguły walidacji `exclude_if`. W tym przykładzie pola `appointment_date` i `doctor_name` nie będą walidowane, jeśli pole `has_appointment` ma wartość `false`:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($data, [
    'has_appointment' => 'required|boolean',
    'appointment_date' => 'exclude_if:has_appointment,false|required|date',
    'doctor_name' => 'exclude_if:has_appointment,false|required|string',
]);
```

Alternatywnie możesz użyć reguły `exclude_unless`, aby nie walidować danego pola, chyba że inne pole ma określoną wartość:

```php
$validator = Validator::make($data, [
    'has_appointment' => 'required|boolean',
    'appointment_date' => 'exclude_unless:has_appointment,true|required|date',
    'doctor_name' => 'exclude_unless:has_appointment,true|required|string',
]);
```

<a name="validating-when-present"></a>
#### Walidacja gdy obecne

W niektórych sytuacjach możesz chcieć uruchomić sprawdzanie walidacji dla pola **tylko** wtedy, gdy to pole jest obecne w walidowanych danych. Aby szybko to osiągnąć, dodaj regułę `sometimes` do listy reguł:

```php
$validator = Validator::make($data, [
    'email' => 'sometimes|required|email',
]);
```

W powyższym przykładzie pole `email` będzie walidowane tylko wtedy, gdy jest obecne w tablicy `$data`.

> [!NOTE]
> Jeśli próbujesz zwalidować pole, które zawsze powinno być obecne, ale może być puste, sprawdź [tę notatkę o polach opcjonalnych](#a-note-on-optional-fields).

<a name="complex-conditional-validation"></a>
#### Złożona walidacja warunkowa

Niekiedy możesz chcieć dodać reguły walidacji oparte na bardziej złożonej logice warunkowej. Na przykład, możesz chcieć wymagać danego pola tylko wtedy, gdy inne pole ma wartość większą niż 100. Lub możesz potrzebować, aby dwa pola miały określoną wartość tylko wtedy, gdy obecne jest inne pole. Dodawanie tych reguł walidacji nie musi być bolesne. Najpierw utwórz instancję `Validator` ze swoimi _statycznymi regułami_, które nigdy się nie zmieniają:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'email' => 'required|email',
    'games' => 'required|integer|min:0',
]);
```

Załóżmy, że nasza aplikacja webowa jest dla kolekcjonerów gier. Jeśli kolekcjoner gier rejestruje się w naszej aplikacji i posiada więcej niż 100 gier, chcemy, aby wyjaśnił, dlaczego posiada tak wiele gier. Na przykład, może prowadzi sklep z używanymi grami lub po prostu lubi kolekcjonować gry. Aby warunkowo dodać to wymaganie, możemy użyć metody `sometimes` na instancji `Validator`.

```php
use Illuminate\Support\Fluent;

$validator->sometimes('reason', 'required|max:500', function (Fluent $input) {
    return $input->games >= 100;
});
```

Pierwszym argumentem przekazanym do metody `sometimes` jest nazwa pola, które warunkowo walidujemy. Drugim argumentem jest lista reguł, które chcemy dodać. Jeśli domknięcie przekazane jako trzeci argument zwróci `true`, reguły zostaną dodane. Ta metoda sprawia, że budowanie złożonych walidacji warunkowych jest dziecinnie proste. Możesz nawet dodać walidacje warunkowe dla kilku pól naraz:

```php
$validator->sometimes(['reason', 'cost'], 'required', function (Fluent $input) {
    return $input->games >= 100;
});
```

> [!NOTE]
> Parametr `$input` przekazany do Twojego domknięcia będzie instancją `Illuminate\Support\Fluent` i może być użyty do dostępu do Twoich danych wejściowych i plików podlegających walidacji.

<a name="complex-conditional-array-validation"></a>
#### Złożona walidacja tablic warunkowych

Niekiedy możesz chcieć zwalidować pole na podstawie innego pola w tej samej zagnieżdżonej tablicy, którego indeksu nie znasz. W takich sytuacjach możesz zezwolić Twojemu domknięciu na otrzymanie drugiego argumentu, którym będzie aktualny pojedynczy element w walidowanej tablicy:

```php
$input = [
    'channels' => [
        [
            'type' => 'email',
            'address' => 'abigail@example.com',
        ],
        [
            'type' => 'url',
            'address' => 'https://example.com',
        ],
    ],
];

$validator->sometimes('channels.*.address', 'email', function (Fluent $input, Fluent $item) {
    return $item->type === 'email';
});

$validator->sometimes('channels.*.address', 'url', function (Fluent $input, Fluent $item) {
    return $item->type !== 'email';
});
```

Podobnie jak parametr `$input` przekazany do domknięcia, parametr `$item` jest instancją `Illuminate\Support\Fluent`, gdy dane atrybutu są tablicą; w przeciwnym razie jest to ciąg znaków.

<a name="validating-arrays"></a>
## Walidacja tablic

Jak omówiono w [dokumentacji reguły walidacji array](#rule-array), reguła `array` akceptuje listę dozwolonych kluczy tablicy. Jeśli w tablicy znajdują się jakiekolwiek dodatkowe klucze, walidacja nie powiedzie się:

```php
use Illuminate\Support\Facades\Validator;

$input = [
    'user' => [
        'name' => 'Taylor Otwell',
        'username' => 'taylorotwell',
        'admin' => true,
    ],
];

Validator::make($input, [
    'user' => 'array:name,username',
]);
```

Ogólnie rzecz biorąc, zawsze powinieneś określać klucze tablicy, które mogą być obecne w Twojej tablicy. W przeciwnym razie metody `validate` i `validated` walidatora zwrócą wszystkie zwalidowane dane, w tym tablicę i wszystkie jej klucze, nawet jeśli te klucze nie zostały zwalidowane przez inne zagnieżdżone reguły walidacji tablicy.

<a name="validating-nested-array-input"></a>
### Walidacja zagnieżdżonych danych tablicowych

Walidacja zagnieżdżonych pól wejściowych formularza opartych na tablicach nie musi być bolesna. Możesz użyć "notacji kropkowej" do walidacji atrybutów w tablicy. Na przykład, jeśli przychodzące żądanie HTTP zawiera pole `photos[profile]`, możesz je zwalidować w następujący sposób:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'photos.profile' => 'required|image',
]);
```

Możesz również zwalidować każdy element tablicy. Na przykład, aby zwalidować, że każdy e-mail w danym polu wejściowym tablicy jest unikalny, możesz zrobić następująco:

```php
$validator = Validator::make($request->all(), [
    'users.*.email' => 'email|unique:users',
    'users.*.first_name' => 'required_with:users.*.last_name',
]);
```

Podobnie możesz użyć znaku `*` podczas określania [niestandardowych komunikatów walidacji w plikach językowych](#custom-messages-for-specific-attributes), co sprawia, że używanie pojedynczego komunikatu walidacji dla pól opartych na tablicach jest dziecinnie proste:

```php
'custom' => [
    'users.*.email' => [
        'unique' => 'Each user must have a unique email address',
    ]
],
```

<a name="accessing-nested-array-data"></a>
#### Dostęp do zagnieżdżonych danych tablicowych

Niekiedy możesz potrzebować dostępu do wartości danego zagnieżdżonego elementu tablicy podczas przypisywania reguł walidacji do atrybutu. Możesz to osiągnąć za pomocą metody `Rule::forEach`. Metoda `forEach` akceptuje domknięcie, które zostanie wywołane dla każdej iteracji atrybutu tablicy podlegającego walidacji i otrzyma wartość atrybutu oraz jawną, w pełni rozwiniętą nazwę atrybutu. Domknięcie powinno zwrócić tablicę reguł do przypisania do elementu tablicy:

```php
use App\Rules\HasPermission;
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

$validator = Validator::make($request->all(), [
    'companies.*.id' => Rule::forEach(function (string|null $value, string $attribute) {
        return [
            Rule::exists(Company::class, 'id'),
            new HasPermission('manage-company', $value),
        ];
    }),
]);
```

<a name="error-message-indexes-and-positions"></a>
### Indeksy i pozycje w komunikatach błędów

Podczas walidacji tablic możesz chcieć odwołać się do indeksu lub pozycji konkretnego elementu, który nie przeszedł walidacji, w komunikacie błędu wyświetlanym przez Twoją aplikację. Aby to osiągnąć, możesz uwzględnić symbole zastępcze `:index` (zaczyna od `0`), `:position` (zaczyna od `1`) lub `:ordinal-position` (zaczyna od `1st`) w Twoim [niestandardowym komunikacie walidacji](#manual-customizing-the-error-messages):

```php
use Illuminate\Support\Facades\Validator;

$input = [
    'photos' => [
        [
            'name' => 'BeachVacation.jpg',
            'description' => 'A photo of my beach vacation!',
        ],
        [
            'name' => 'GrandCanyon.jpg',
            'description' => '',
        ],
    ],
];

Validator::validate($input, [
    'photos.*.description' => 'required',
], [
    'photos.*.description.required' => 'Please describe photo #:position.',
]);
```

W powyższym przykładzie walidacja nie powiedzie się, a użytkownikowi zostanie przedstawiony następujący błąd: _"Please describe photo #2."_

W razie potrzeby możesz odwoływać się do bardziej zagnieżdżonych indeksów i pozycji za pomocą `second-index`, `second-position`, `third-index`, `third-position` itp.

```php
'photos.*.attributes.*.string' => 'Invalid attribute for photo #:second-position.',
```

<a name="validating-files"></a>
## Walidacja plików

Laravel zapewnia różnorodne reguły walidacji, które mogą być użyte do walidacji przesłanych plików, takie jak `mimes`, `image`, `min` i `max`. Chociaż możesz swobodnie określać te reguły indywidualnie podczas walidacji plików, Laravel oferuje również płynny kreator reguł walidacji plików, który może okazać się wygodny:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rules\File;

Validator::validate($input, [
    'attachment' => [
        'required',
        File::types(['mp3', 'wav'])
            ->min(1024)
            ->max(12 * 1024),
    ],
]);
```

<a name="validating-files-file-types"></a>
#### Walidacja typów plików

Chociaż musisz tylko określić rozszerzenia podczas wywoływania metody `types`, ta metoda faktycznie waliduje typ MIME pliku, czytając zawartość pliku i odgadując jego typ MIME. Pełna lista typów MIME i odpowiadających im rozszerzeń może być znaleziona w następującej lokalizacji:

[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="validating-files-file-sizes"></a>
#### Walidacja rozmiarów plików

Dla wygody minimalne i maksymalne rozmiary plików mogą być określone jako ciąg znaków z sufiksem wskazującym jednostki rozmiaru pliku. Obsługiwane są sufiksy `kb`, `mb`, `gb` i `tb`:

```php
File::types(['mp3', 'wav'])
    ->min('1kb')
    ->max('10mb');
```

<a name="validating-files-image-files"></a>
#### Walidacja plików obrazów

Jeśli Twoja aplikacja akceptuje obrazy przesyłane przez użytkowników, możesz użyć metody konstruktora `image` reguły `File`, aby upewnić się, że plik podlegający walidacji jest obrazem (jpg, jpeg, png, bmp, gif lub webp).

Ponadto reguła `dimensions` może być użyta do ograniczenia wymiarów obrazu:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;
use Illuminate\Validation\Rules\File;

Validator::validate($input, [
    'photo' => [
        'required',
        File::image()
            ->min(1024)
            ->max(12 * 1024)
            ->dimensions(Rule::dimensions()->maxWidth(1000)->maxHeight(500)),
    ],
]);
```

> [!NOTE]
> Więcej informacji dotyczących walidacji wymiarów obrazów można znaleźć w [dokumentacji reguły dimensions](#rule-dimensions).

> [!WARNING]
> Domyślnie reguła `image` nie zezwala na pliki SVG ze względu na możliwość podatności XSS. Jeśli musisz zezwolić na pliki SVG, możesz przekazać `allowSvg: true` do reguły `image`: `File::image(allowSvg: true)`.

<a name="validating-files-image-dimensions"></a>
#### Walidacja wymiarów obrazów

Możesz również zwalidować wymiary obrazu. Na przykład, aby zwalidować, że przesłany obraz ma co najmniej 1000 pikseli szerokości i 500 pikseli wysokości, możesz użyć reguły `dimensions`:

```php
use Illuminate\Validation\Rule;
use Illuminate\Validation\Rules\File;

File::image()->dimensions(
    Rule::dimensions()
        ->maxWidth(1000)
        ->maxHeight(500)
)
```

> [!NOTE]
> Więcej informacji dotyczących walidacji wymiarów obrazów można znaleźć w [dokumentacji reguły dimensions](#rule-dimensions).

<a name="validating-passwords"></a>
## Walidacja haseł

Aby zapewnić, że hasła mają odpowiedni poziom złożoności, możesz użyć obiektu reguły `Password` Laravel:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rules\Password;

$validator = Validator::make($request->all(), [
    'password' => ['required', 'confirmed', Password::min(8)],
]);
```

Obiekt reguły `Password` umożliwia łatwe dostosowanie wymagań dotyczących złożoności haseł dla Twojej aplikacji, takich jak określenie, że hasła wymagają co najmniej jednej litery, cyfry, symbolu lub znaków o mieszanej wielkości liter:

```php
// Require at least 8 characters...
Password::min(8)

// Require at least one letter...
Password::min(8)->letters()

// Require at least one uppercase and one lowercase letter...
Password::min(8)->mixedCase()

// Require at least one number...
Password::min(8)->numbers()

// Require at least one symbol...
Password::min(8)->symbols()
```

Ponadto możesz upewnić się, że hasło nie zostało skompromitowane w publicznym wycieku danych haseł za pomocą metody `uncompromised`:

```php
Password::min(8)->uncompromised()
```

Wewnętrznie obiekt reguły `Password` używa modelu [k-Anonymity](https://en.wikipedia.org/wiki/K-anonymity) do określenia, czy hasło zostało wycieknięte za pośrednictwem usługi [haveibeenpwned.com](https://haveibeenpwned.com) bez poświęcania prywatności lub bezpieczeństwa użytkownika.

Domyślnie, jeśli hasło pojawi się co najmniej raz w wycieku danych, będzie uważane za skompromitowane. Możesz dostosować ten próg za pomocą pierwszego argumentu metody `uncompromised`:

```php
// Ensure the password appears less than 3 times in the same data leak...
Password::min(8)->uncompromised(3);
```

Oczywiście możesz łączyć wszystkie metody w powyższych przykładach:

```php
Password::min(8)
    ->letters()
    ->mixedCase()
    ->numbers()
    ->symbols()
    ->uncompromised()
```

<a name="defining-default-password-rules"></a>
#### Definiowanie domyślnych reguł haseł

Możesz uznać za wygodne określenie domyślnych reguł walidacji haseł w jednym miejscu Twojej aplikacji. Możesz to łatwo osiągnąć za pomocą metody `Password::defaults`, która akceptuje domknięcie. Domknięcie podane metodzie `defaults` powinno zwrócić domyślną konfigurację reguły Password. Zazwyczaj reguła `defaults` powinna być wywoływana w metodzie `boot` jednego z dostawców usług Twojej aplikacji:

```php
use Illuminate\Validation\Rules\Password;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Password::defaults(function () {
        $rule = Password::min(8);

        return $this->app->isProduction()
            ? $rule->mixedCase()->uncompromised()
            : $rule;
    });
}
```

Następnie, gdy chcesz zastosować domyślne reguły do konkretnego hasła podlegającego walidacji, możesz wywołać metodę `defaults` bez argumentów:

```php
'password' => ['required', Password::defaults()],
```

Czasami możesz chcieć dołączyć dodatkowe reguły walidacji do domyślnych reguł walidacji haseł. Możesz użyć metody `rules`, aby to osiągnąć:

```php
use App\Rules\ZxcvbnRule;

Password::defaults(function () {
    $rule = Password::min(8)->rules([new ZxcvbnRule]);

    // ...
});
```

<a name="custom-validation-rules"></a>
## Niestandardowe reguły walidacji

<a name="using-rule-objects"></a>
### Używanie obiektów reguł

Laravel zapewnia różnorodne przydatne reguły walidacji; jednak możesz chcieć określić niektóre własne. Jedną z metod rejestrowania niestandardowych reguł walidacji jest używanie obiektów reguł. Aby wygenerować nowy obiekt reguły, możesz użyć polecenia Artisan `make:rule`. Użyjmy tego polecenia do wygenerowania reguły, która weryfikuje, czy ciąg znaków jest napisany wielkimi literami. Laravel umieści nową regułę w katalogu `app/Rules`. Jeśli ten katalog nie istnieje, Laravel utworzy go podczas wykonywania polecenia Artisan w celu utworzenia Twojej reguły:

```shell
php artisan make:rule Uppercase
```

Po utworzeniu reguły jesteśmy gotowi zdefiniować jej zachowanie. Obiekt reguły zawiera pojedynczą metodę: `validate`. Ta metoda otrzymuje nazwę atrybutu, jego wartość i wywołanie zwrotne, które powinno być wywołane w przypadku niepowodzenia z komunikatem błędu walidacji:

```php
<?php

namespace App\Rules;

use Closure;
use Illuminate\Contracts\Validation\ValidationRule;

class Uppercase implements ValidationRule
{
    /**
     * Run the validation rule.
     */
    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if (strtoupper($value) !== $value) {
            $fail('The :attribute must be uppercase.');
        }
    }
}
```

Po zdefiniowaniu reguły możesz dołączyć ją do walidatora, przekazując instancję obiektu reguły wraz z innymi regułami walidacji:

```php
use App\Rules\Uppercase;

$request->validate([
    'name' => ['required', 'string', new Uppercase],
]);
```

#### Tłumaczenie komunikatów walidacji

Zamiast podawać dosłowny komunikat błędu do domknięcia `$fail`, możesz również podać [klucz ciągu tłumaczenia](/docs/{{version}}/localization) i polecić Laravel, aby przetłumaczył komunikat błędu:

```php
if (strtoupper($value) !== $value) {
    $fail('validation.uppercase')->translate();
}
```

W razie potrzeby możesz podać zastąpienia symboli zastępczych i preferowany język jako pierwszy i drugi argument metody `translate`:

```php
$fail('validation.location')->translate([
    'value' => $this->value,
], 'fr');
```

#### Dostęp do dodatkowych danych

Jeśli Twoja niestandardowa klasa reguły walidacji potrzebuje dostępu do wszystkich innych danych podlegających walidacji, Twoja klasa reguły może zaimplementować interfejs `Illuminate\Contracts\Validation\DataAwareRule`. Ten interfejs wymaga, aby Twoja klasa zdefiniowała metodę `setData`. Ta metoda zostanie automatycznie wywołana przez Laravel (przed rozpoczęciem walidacji) ze wszystkimi danymi podlegającymi walidacji:

```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\DataAwareRule;
use Illuminate\Contracts\Validation\ValidationRule;

class Uppercase implements DataAwareRule, ValidationRule
{
    /**
     * All of the data under validation.
     *
     * @var array<string, mixed>
     */
    protected $data = [];

    // ...

    /**
     * Set the data under validation.
     *
     * @param  array<string, mixed>  $data
     */
    public function setData(array $data): static
    {
        $this->data = $data;

        return $this;
    }
}
```

Lub, jeśli Twoja reguła walidacji wymaga dostępu do instancji walidatora wykonującego walidację, możesz zaimplementować interfejs `ValidatorAwareRule`:

```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\ValidationRule;
use Illuminate\Contracts\Validation\ValidatorAwareRule;
use Illuminate\Validation\Validator;

class Uppercase implements ValidationRule, ValidatorAwareRule
{
    /**
     * The validator instance.
     *
     * @var \Illuminate\Validation\Validator
     */
    protected $validator;

    // ...

    /**
     * Set the current validator.
     */
    public function setValidator(Validator $validator): static
    {
        $this->validator = $validator;

        return $this;
    }
}
```

<a name="using-closures"></a>
### Używanie domknięć

Jeśli potrzebujesz funkcjonalności niestandardowej reguły tylko raz w całej aplikacji, możesz użyć domknięcia zamiast obiektu reguły. Domknięcie otrzymuje nazwę atrybutu, wartość atrybutu i wywołanie zwrotne `$fail`, które powinno być wywołane, jeśli walidacja nie powiedzie się:

```php
use Illuminate\Support\Facades\Validator;
use Closure;

$validator = Validator::make($request->all(), [
    'title' => [
        'required',
        'max:255',
        function (string $attribute, mixed $value, Closure $fail) {
            if ($value === 'foo') {
                $fail("The {$attribute} is invalid.");
            }
        },
    ],
]);
```

<a name="implicit-rules"></a>
### Reguły niejawne

Domyślnie, gdy walidowany atrybut nie jest obecny lub zawiera pusty ciąg znaków, normalne reguły walidacji, w tym reguły niestandardowe, nie są uruchamiane. Na przykład reguła [unique](#rule-unique) nie zostanie uruchomiona dla pustego ciągu znaków:

```php
use Illuminate\Support\Facades\Validator;

$rules = ['name' => 'unique:users,name'];

$input = ['name' => ''];

Validator::make($input, $rules)->passes(); // true
```

Aby niestandardowa reguła była uruchamiana nawet wtedy, gdy atrybut jest pusty, reguła musi sugerować, że atrybut jest wymagany. Aby szybko wygenerować nowy obiekt reguły niejawnej, możesz użyć polecenia Artisan `make:rule` z opcją `--implicit`:

```shell
php artisan make:rule Uppercase --implicit
```

> [!WARNING]
> Reguła "niejawna" tylko _sugeruje_, że atrybut jest wymagany. To, czy faktycznie unieważnia brakujący lub pusty atrybut, zależy od Ciebie.
