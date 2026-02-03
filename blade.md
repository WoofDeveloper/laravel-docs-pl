# Szablony Blade

- [Wprowadzenie](#introduction)
    - [Wzmacnianie Blade za pomocą Livewire](#supercharging-blade-with-livewire)
- [Wyświetlanie danych](#displaying-data)
    - [Kodowanie encji HTML](#html-entity-encoding)
    - [Blade i frameworki JavaScript](#blade-and-javascript-frameworks)
- [Dyrektywy Blade](#blade-directives)
    - [Instrukcje warunkowe If](#if-statements)
    - [Instrukcje Switch](#switch-statements)
    - [Pętle](#loops)
    - [Zmienna Loop](#the-loop-variable)
    - [Warunkowe klasy](#conditional-classes)
    - [Dodatkowe atrybuty](#additional-attributes)
    - [Dołączanie podwidoków](#including-subviews)
    - [Dyrektywa `@once`](#the-once-directive)
    - [Czysty PHP](#raw-php)
    - [Komentarze](#comments)
- [Komponenty](#components)
    - [Renderowanie komponentów](#rendering-components)
    - [Komponenty indeksowe](#index-components)
    - [Przekazywanie danych do komponentów](#passing-data-to-components)
    - [Atrybuty komponentów](#component-attributes)
    - [Słowa kluczowe zarezerwowane](#reserved-keywords)
    - [Sloty](#slots)
    - [Widoki komponentów inline](#inline-component-views)
    - [Komponenty dynamiczne](#dynamic-components)
    - [Ręczna rejestracja komponentów](#manually-registering-components)
- [Komponenty anonimowe](#anonymous-components)
    - [Anonimowe komponenty indeksowe](#anonymous-index-components)
    - [Właściwości danych / Atrybuty](#data-properties-attributes)
    - [Dostęp do danych nadrzędnych](#accessing-parent-data)
    - [Ścieżki komponentów anonimowych](#anonymous-component-paths)
- [Budowanie layoutów](#building-layouts)
    - [Layouty za pomocą komponentów](#layouts-using-components)
    - [Layouty za pomocą dziedziczenia szablonów](#layouts-using-template-inheritance)
- [Formularze](#forms)
    - [Pole CSRF](#csrf-field)
    - [Pole Method](#method-field)
    - [Błędy walidacji](#validation-errors)
- [Stosy](#stacks)
- [Wstrzykiwanie serwisów](#service-injection)
- [Renderowanie inline szablonów Blade](#rendering-inline-blade-templates)
- [Renderowanie fragmentów Blade](#rendering-blade-fragments)
- [Rozszerzanie Blade](#extending-blade)
    - [Niestandardowe handlery echo](#custom-echo-handlers)
    - [Niestandardowe instrukcje warunkowe](#custom-if-statements)

<a name="introduction"></a>
## Wprowadzenie

Blade to prosty, ale potężny silnik szablonów dołączony do Laravela. W przeciwieństwie do niektórych silników szablonów PHP, Blade nie ogranicza używania czystego kodu PHP w szablonach. W rzeczywistości wszystkie szablony Blade są kompilowane do czystego kodu PHP i buforowane do momentu ich modyfikacji, co oznacza, że Blade praktycznie nie dodaje żadnych narzutów do aplikacji. Pliki szablonów Blade używają rozszerzenia `.blade.php` i są zwykle przechowywane w katalogu `resources/views`.

Widoki Blade mogą być zwracane z tras lub kontrolerów za pomocą globalnego helpera `view`. Oczywiście, jak wspomniano w dokumentacji dotyczącej [widoków](/docs/{{version}}/views), dane mogą być przekazywane do widoku Blade za pomocą drugiego argumentu helpera `view`:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'Finn']);
});
```

<a name="supercharging-blade-with-livewire"></a>
### Wzmacnianie Blade za pomocą Livewire

Chcesz przenieść swoje szablony Blade na wyższy poziom i z łatwością budować dynamiczne interfejsy? Sprawdź [Laravel Livewire](https://livewire.laravel.com). Livewire pozwala pisać komponenty Blade wzbogacone o dynamiczną funkcjonalność, która zazwyczaj byłaby możliwa tylko za pomocą frameworków frontendowych takich jak React czy Vue, zapewniając świetne podejście do budowania nowoczesnych, reaktywnych frontendów bez złożoności, renderowania po stronie klienta czy kroków budowania wielu frameworków JavaScript.

<a name="displaying-data"></a>
## Wyświetlanie danych

Możesz wyświetlać dane, które są przekazywane do widoków Blade, owijając zmienną w nawiasy klamrowe. Na przykład, mając następującą trasę:

```php
Route::get('/', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```

Możesz wyświetlić zawartość zmiennej `name` w następujący sposób:

```blade
Hello, {{ $name }}.
```

> [!NOTE]
> Wyrażenia echo `{{ }}` Blade są automatycznie przekazywane przez funkcję PHP `htmlspecialchars`, aby zapobiec atakom XSS.

Nie jesteś ograniczony do wyświetlania zawartości zmiennych przekazanych do widoku. Możesz również wyprowadzać wyniki dowolnej funkcji PHP. W rzeczywistości możesz umieścić dowolny kod PHP w instrukcji echo Blade:

```blade
The current UNIX timestamp is {{ time() }}.
```

<a name="html-entity-encoding"></a>
### Kodowanie encji HTML

Domyślnie Blade (i funkcja Laravela `e`) podwójnie koduje encje HTML. Jeśli chcesz wyłączyć podwójne kodowanie, wywołaj metodę `Blade::withoutDoubleEncoding` z metody `boot` Twojego `AppServiceProvider`:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Blade::withoutDoubleEncoding();
    }
}
```

<a name="displaying-unescaped-data"></a>
#### Wyświetlanie danych bez escape'owania

Domyślnie instrukcje Blade `{{ }}` są automatycznie przekazywane przez funkcję PHP `htmlspecialchars`, aby zapobiec atakom XSS. Jeśli nie chcesz, aby Twoje dane były escapowane, możesz użyć następującej składni:

```blade
Hello, {!! $name !!}.
```

> [!WARNING]
> Zachowaj szczególną ostrożność podczas wyprowadzania treści dostarczonej przez użytkowników Twojej aplikacji. Powinie-neś zwykle używać escapowanej składni z podwójnymi nawiasami klamrowymi, aby zapobiec atakom XSS podczas wyświetlania danych dostarczonych przez użytkownika.

<a name="blade-and-javascript-frameworks"></a>
### Blade i frameworki JavaScript

Ponieważ wiele frameworków JavaScript również używa "nawiasow klamrowych" do wskazania, że dane wyrażenie powinno zostać wyświetlone w przeglądarce, możesz użyć symbolu `@`, aby poinformować silnik renderowania Blade, że wyrażenie powinno pozostać nietknięte. Na przykład:

```blade
<h1>Laravel</h1>

Hello, @{{ name }}.
```

W tym przykładzie symbol `@` zostanie usunięty przez Blade; jednak wyrażenie `{{ name }}` pozostanie nietknięte przez silnik Blade, pozwalając na jego renderowanie przez Twój framework JavaScript.

Symbol `@` może również być używany do escapowania dyrektyw Blade:

```blade
{{-- Blade template --}}
@@if()

<!-- HTML output -->
@if()
```

<a name="rendering-json"></a>
#### Renderowanie JSON

Czasami możesz przekazać tablicę do swojego widoku z zamiarem renderowania jej jako JSON w celu zainicjowania zmiennej JavaScript. Na przykład:

```php
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

Jednak zamiast ręcznie wywoływać `json_encode`, możesz użyć metody `Illuminate\Support\Js::from`. Metoda `from` przyjmuje te same argumenty co funkcja PHP `json_encode`; jednak będzie zapewniać, że wynikowy JSON został poprawnie escapowany do umieszczenia w cudzysłowach HTML. Metoda `from` zwróci łańcuch instrukcji JavaScript `JSON.parse`, który przekonwertuje dany obiekt lub tablicę na prawidłowy obiekt JavaScript:

```blade
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```

Najnowsze wersje szkieletu aplikacji Laravel zawierają fasadę `Js`, która zapewnia wygodny dostęp do tej funkcjonalności w szablonach Blade:

```blade
<script>
    var app = {{ Js::from($array) }};
</script>
```

> [!WARNING]
> Powinie-neś używać metody `Js::from` tylko do renderowania istniejących zmiennych jako JSON. Szablonowanie Blade opiera się na wyrażeniach regularnych i próby przekazania złożonego wyrażenia do dyrektywy mogą spowodować nieoczekiwane niepowodzenia.

<a name="the-at-verbatim-directive"></a>
#### Dyrektywa `@verbatim`

Jeśli wyświetlasz zmienne JavaScript w dużej części szablonu, możesz owinąć HTML w dyrektywie `@verbatim`, aby nie musiał poprzedzać każdej instrukcji echo Blade symbolem `@`:

```blade
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

<a name="blade-directives"></a>
## Dyrektywy Blade

Oprócz dziedziczenia szablonów i wyświetlania danych, Blade zapewnia również wygodne skróty dla popularnych struktur kontrolnych PHP, takich jak instrukcje warunkowe i pętle. Te skróty zapewniają bardzo czysty, zwięzły sposób pracy ze strukturami kontrolnymi PHP, pozostając jednocześnie znajome dla ich odpowiedników PHP.

<a name="if-statements"></a>
### Instrukcje warunkowe If

Możesz konstruować instrukcje `if` używając dyrektyw `@if`, `@elseif`, `@else` i `@endif`. Te dyrektywy działają identycznie jak ich odpowiedniki PHP:

```blade
@if (count($records) === 1)
    Mam jeden rekord!
@elseif (count($records) > 1)
    Mam wiele rekordów!
@else
    Nie mam żadnych rekordów!
@endif
```

Dla wygody Blade zapewnia również dyrektywę `@unless`:

```blade
@unless (Auth::check())
    Nie jesteś zalogowany.
@endunless
```

Oprócz już omowionych dyrektyw warunkowych, dyrektywy `@isset` i `@empty` mogą być używane jako wygodne skróty dla swoich odpowiednich funkcji PHP:

```blade
@isset($records)
    // $records jest zdefiniowane i nie jest null...
@endisset

@empty($records)
    // $records jest "puste"...
@endempty
```

<a name="authentication-directives"></a>
#### Dyrektywy uwierzytelniania

Dyrektywy `@auth` i `@guest` mogą być używane do szybkiego określenia, czy aktualny użytkownik jest [uwierzytelniony](/docs/{{version}}/authentication), czy jest gościem:

```blade
@auth
    // Użytkownik jest uwierzytelniony...
@endauth

@guest
    // Użytkownik nie jest uwierzytelniony...
@endguest
```

Jeśli potrzeba, możesz określić guard uwierzytelniania, który powinien zostać sprawdzony podczas używania dyrektyw `@auth` i `@guest`:

```blade
@auth('admin')
    // Użytkownik jest uwierzytelniony...
@endauth

@guest('admin')
    // Użytkownik nie jest uwierzytelniony...
@endguest
```

<a name="environment-directives"></a>
#### Dyrektywy środowiska

Możesz sprawdzić, czy aplikacja działa w środowisku produkcyjnym, używając dyrektywy `@production`:

```blade
@production
    // Treść specyficzna dla produkcji...
@endproduction
```

Lub możesz określić, czy aplikacja działa w konkretnym środowisku, używając dyrektywy `@env`:

```blade
@env('staging')
    // Aplikacja działa w "staging"...
@endenv

@env(['staging', 'production'])
    // Aplikacja działa w "staging" lub "production"...
@endenv
```

<a name="section-directives"></a>
#### Dyrektywy sekcji

Możesz określić, czy sekcja dziedziczenia szablonów ma zawartość, używając dyrektywy `@hasSection`:

```blade
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```

Możesz użyć dyrektywy `sectionMissing`, aby określić, czy sekcja nie ma zawartości:

```blade
@sectionMissing('navigation')
    <div class="pull-right">
        @include('default-navigation')
    </div>
@endif
```

<a name="session-directives"></a>
#### Dyrektywy sesji

Dyrektywa `@session` może być używana do określenia, czy wartość [sesji](/docs/{{version}}/session) istnieje. Jeśli wartość sesji istnieje, zawartość szablonu między dyrektywami `@session` i `@endsession` zostanie wyewaluowana. W treści dyrektywy `@session` możesz wyprowadzić zmienną `$value`, aby wyświetlić wartość sesji:

```blade
@session('status')
    <div class="p-4 bg-green-100">
        {{ $value }}
    </div>
@endsession
```

<a name="context-directives"></a>
#### Dyrektywy kontekstu

Dyrektywa `@context` może być używana do określenia, czy wartość [kontekstu](/docs/{{version}}/context) istnieje. Jeśli wartość kontekstu istnieje, zawartość szablonu między dyrektywami `@context` i `@endcontext` zostanie wyewaluowana. W treści dyrektywy `@context` możesz wyprowadzić zmienną `$value`, aby wyświetlić wartość kontekstu:

```blade
@context('canonical')
    <link href="{{ $value }}" rel="canonical">
@endcontext
```

<a name="switch-statements"></a>
### Instrukcje Switch

Instrukcje switch mogą być konstruowane za pomocą dyrektyw `@switch`, `@case`, `@break`, `@default` i `@endswitch`:

```blade
@switch($i)
    @case(1)
        Pierwszy przypadek...
        @break

    @case(2)
        Drugi przypadek...
        @break

    @default
        Domyślny przypadek...
@endswitch
```
        Default case...
@endswitch
```

<a name="loops"></a>
### Pętle

Oprócz instrukcji warunkowych, Blade zapewnia proste dyrektywy do pracy ze strukturami pętli PHP. Ponownie, każda z tych dyrektyw działa identycznie jak ich odpowiedniki PHP:

```blade
@for ($i = 0; $i < 10; $i++)
    Aktualna wartość to {{ $i }}
@endfor

@foreach ($users as $user)
    <p>To jest użytkownik {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>Brak użytkowników</p>
@endforelse

@while (true)
    <p>Pętlę się w nieskończoność.</p>
@endwhile
```

> [!NOTE]
> Podczas iteracji przez pętlę `foreach` możesz użyć [zmiennej loop](#the-loop-variable), aby uzyskać cenne informacje o pętli, takie jak to, czy jesteś w pierwszej czy ostatniej iteracji przez pętlę.

Podczas używania pętli możesz również pominąć bieżącą iterację lub zakończyć pętlę za pomocą dyrektyw `@continue` i `@break`:

```blade
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach
```

Możesz również uwzględnić warunek kontynuacji lub przerwania w deklaracji dyrektywy:

```blade
@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
```

<a name="the-loop-variable"></a>
### Zmienna Loop

Podczas iteracji przez pętlę `foreach`, zmienna `$loop` będzie dostępna wewnątrz Twojej pętli. Ta zmienna zapewnia dostęp do niektórych przydatnych informacji, takich jak aktualny indeks pętli i czy jest to pierwsza czy ostatnia iteracja przez pętlę:

```blade
@foreach ($users as $user)
    @if ($loop->first)
        To jest pierwsza iteracja.
    @endif

    @if ($loop->last)
        To jest ostatnia iteracja.
    @endif

    <p>To jest użytkownik {{ $user->id }}</p>
@endforeach
```

Jeśli znajdujesz się w zagnieżdżonej pętli, możesz uzyskać dostęp do zmiennej `$loop` pętli nadrzędnej za pomocą właściwości `parent`:

```blade
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            To jest pierwsza iteracja pętli nadrzędnej.
        @endif
    @endforeach
@endforeach
```

Zmienna `$loop` zawiera również różne inne przydatne właściwości:

<div class="overflow-auto">

| Właściwość           | Opis                                            |
| ------------------ | ------------------------------------------------------ |
| `$loop->index`     | Indeks aktualnej iteracji pętli (zaczyna się od 0). |
| `$loop->iteration` | Aktualna iteracja pętli (zaczyna się od 1).              |
| `$loop->remaining` | Pozostałe iteracje w pętli.                  |
| `$loop->count`     | Całkowita liczba elementów w iterowanej tablicy. |
| `$loop->first`     | Czy jest to pierwsza iteracja przez pętlę.  |
| `$loop->last`      | Czy jest to ostatnia iteracja przez pętlę.   |
| `$loop->even`      | Czy jest to parzysta iteracja przez pętlę.    |
| `$loop->odd`       | Czy jest to nieparzysta iteracja przez pętlę.     |
| `$loop->depth`     | Poziom zagnieżdżenia aktualnej pętli.                 |
| `$loop->parent`    | W zagnieżdżonej pętli, zmienna loop pętli nadrzędnej.     |

</div>

<a name="conditional-classes"></a>
### Warunkowe klasy i style

Dyrektywa `@class` warunkowo kompiluje łańcuch klas CSS. Dyrektywa przyjmuje tablicę klas, gdzie klucz tablicy zawiera klasę lub klasy, które chcesz dodać, podczas gdy wartość jest wyrażeniem logicznym. Jeśli element tablicy ma klucz numeryczny, zawsze będzie uwzględniony na liście renderowanych klas:

```blade
@php
    $isActive = false;
    $hasError = true;
@endphp

<span @class([
    'p-4',
    'font-bold' => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>

<span class="p-4 text-gray-500 bg-red"></span>
```

Podobnie dyrektywa `@style` może być używana do warunkowego dodawania inline stylów CSS do elementu HTML:

```blade
@php
    $isActive = true;
@endphp

<span @style([
    'background-color: red',
    'font-weight: bold' => $isActive,
])></span>

<span style="background-color: red; font-weight: bold;"></span>
```

<a name="additional-attributes"></a>
### Dodatkowe atrybuty

Dla wygody możesz użyć dyrektywy `@checked`, aby łatwo wskazać, czy dany checkbox HTML jest "zaznaczony". Ta dyrektywa wyprowadzi `checked`, jeśli podany warunek zostanie oceniony jako `true`:

```blade
<input
    type="checkbox"
    name="active"
    value="active"
    @checked(old('active', $user->active))
/>
```

Podobnie dyrektywa `@selected` może być używana do wskazania, czy dana opcja select powinna być "wybrana":

```blade
<select name="version">
    @foreach ($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>
```

Dodatkowo dyrektywa `@disabled` może być używana do wskazania, czy dany element powinien być "wyłączony":

```blade
<button type="submit" @disabled($errors->isNotEmpty())>Submit</button>
```

Ponadto dyrektywa `@readonly` może być używana do wskazania, czy dany element powinien być "tylko do odczytu":

```blade
<input
    type="email"
    name="email"
    value="email@laravel.com"
    @readonly($user->isNotAdmin())
/>
```

Ponadto dyrektywa `@required` może być używana do wskazania, czy dany element powinien być "wymagany":

```blade
<input
    type="text"
    name="title"
    value="title"
    @required($user->isAdmin())
/>
```

<a name="including-subviews"></a>
### Dołączanie podwidoków

> [!NOTE]
> Choć możesz swobodnie używać dyrektywy `@include`, [komponenty](#components) Blade zapewniają podobną funkcjonalność i oferują kilka korzyści w porównaniu z dyrektywą `@include`, takich jak wiązanie danych i atrybutów.

Dyrektywa `@include` Blade pozwala dołączyć widok Blade z innego widoku. Wszystkie zmienne dostępne dla widoku nadrzędnego będą udostępnione dołączanemu widokowi:

```blade
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

Choć dołączany widok odziedziczy wszystkie dane dostępne w widoku nadrzędnym, możesz również przekazać tablicę dodatkowych danych, które powinny być udostępnione dołączanemu widokowi:

```blade
@include('view.name', ['status' => 'complete'])
```

Jeśli próbujesz użyć `@include` dla widoku, który nie istnieje, Laravel zgłosi błąd. Jeśli chcesz dołączyć widok, który może istnieć lub nie, powinie-neś użyć dyrektywy `@includeIf`:

```blade
@includeIf('view.name', ['status' => 'complete'])
```

Jeśli chcesz użyć `@include` dla widoku, jeśli dane wyrażenie logiczne zostanie ocenione jako `true` lub `false`, możesz użyć dyrektyw `@includeWhen` i `@includeUnless`:

```blade
@includeWhen($boolean, 'view.name', ['status' => 'complete'])

@includeUnless($boolean, 'view.name', ['status' => 'complete'])
```

Aby dołączyć pierwszy widok, który istnieje z danej tablicy widoków, możesz użyć dyrektywy `includeFirst`:

```blade
@includeFirst(['custom.admin', 'admin'], ['status' => 'complete'])
```

Jeśli chcesz dołączyć widok bez dziedziczenia jakichkolwiek zmiennych z widoku nadrzędnego, możesz użyć dyrektywy `@includeIsolated`. Dołączany widok będzie miał dostęp tylko do zmiennych, które jawnie przekazujesz:

```blade
@includeIsolated('view.name', ['user' => $user])
```

> [!WARNING]
> Powinie-neś unikać używania stałych `__DIR__` i `__FILE__` w swoich widokach Blade, ponieważ będą one odnosić się do lokalizacji buforowanego, skompilowanego widoku.

<a name="rendering-views-for-collections"></a>
#### Renderowanie widoków dla kolekcji

Możesz połączyć pętle i include'y w jedną linię za pomocą dyrektywy `@each` Blade:

```blade
@each('view.name', $jobs, 'job')
```

Pierwszy argument dyrektywy `@each` to widok do renderowania dla każdego elementu w tablicy lub kolekcji. Drugi argument to tablica lub kolekcja, przez którą chcesz iterować, a trzeci argument to nazwa zmiennej, która zostanie przypisana do aktualnej iteracji w widoku. Na przykład, jeśli iterujesz przez tablicę `jobs`, zazwyczaj będziesz chciał uzyskiwać dostęp do każdego zadania jako zmiennej `job` w widoku. Klucz tablicy dla bieżącej iteracji będzie dostępny jako zmienna `key` w widoku.

Możesz również przekazać czwarty argument do dyrektywy `@each`. Ten argument określa widok, który zostanie wyrenderowany, jeśli dana tablica jest pusta.

```blade
@each('view.name', $jobs, 'job', 'view.empty')
```

> [!WARNING]
> Widoki renderowane za pomocą `@each` nie dziedziczą zmiennych z widoku nadrzędnego. Jeśli widok potomny wymaga tych zmiennych, powinie-neś użyć zamiast tego dyrektyw `@foreach` i `@include`.

<a name="the-once-directive"></a>
### Dyrektywa `@once`

Dyrektywa `@once` pozwala zdefiniować część szablonu, która zostanie wyewaluowana tylko raz podczas cyklu renderowania. Może to być przydatne do przesyłania danej części JavaScript do nagłówka strony za pomocą [stosów](#stacks). Na przykład, jeśli renderujesz dany [komponent](#components) w pętli, możesz chcieć przesłać JavaScript do nagłówka tylko za pierwszym razem, gdy komponent jest renderowany:

```blade
@once
    @push('scripts')
        <script>
            // Your custom JavaScript...
        </script>
    @endpush
@endonce
```

Ponieważ dyrektywa `@once` jest często używana w połączeniu z dyrektywami `@push` lub `@prepend`, dla Twojej wygody dostępne są dyrektywy `@pushOnce` i `@prependOnce`:

```blade
@pushOnce('scripts')
    <script>
        // Your custom JavaScript...
    </script>
@endPushOnce
```

Jeśli przesyłasz zduplikowaną zawartość z dwóch osobnych szablonów Blade, powinie-neś podać unikalny identyfikator jako drugi argument do dyrektywy `@pushOnce`, aby zapewnić, że zawartość zostanie wyrenderowana tylko raz:

```blade
<!-- pie-chart.blade.php -->
@pushOnce('scripts', 'chart.js')
    <script src="/chart.js"></script>
@endPushOnce

<!-- line-chart.blade.php -->
@pushOnce('scripts', 'chart.js')
    <script src="/chart.js"></script>
@endPushOnce
```

<a name="raw-php"></a>
### Czysty PHP

W niektórych sytuacjach przydatne jest osadzenie kodu PHP w widokach. Możesz użyć dyrektywy Blade `@php`, aby wykonać blok zwykłego PHP w szablonie:

```blade
@php
    $counter = 1;
@endphp
```

Lub, jeśli potrzebujesz użyć PHP tylko do zaimportowania klasy, możesz użyć dyrektywy `@use`:

```blade
@use('App\Models\Flight')
```

Drugi argument może zostać przekazany do dyrektywy `@use`, aby ustawić alias zaimportowanej klasy:

```blade
@use('App\Models\Flight', 'FlightModel')
```

Jeśli masz wiele klas w tej samej przestrzeni nazw, możesz pogrupować importy tych klas:

```blade
@use('App\Models\{Flight, Airport}')
```

Dyrektywa `@use` obsługuje również importowanie funkcji i stałych PHP poprzez poprzedzenie ścieżki importu modyfikatorem `function` lub `const`:

```blade
@use(function App\Helpers\format_currency)
@use(const App\Constants\MAX_ATTEMPTS)
```

Podobnie jak importy klas, aliasy są obsługiwane również dla funkcji i stałych:

```blade
@use(function App\Helpers\format_currency, 'formatMoney')
@use(const App\Constants\MAX_ATTEMPTS, 'MAX_TRIES')
```

Grupowane importy są również obsługiwane z modyfikatorami function i const, umożliwiając importowanie wielu symbolów z tej samej przestrzeni nazw w jednej dyrektywie:

```blade
@use(function App\Helpers\{format_currency, format_date})
@use(const App\Constants\{MAX_ATTEMPTS, DEFAULT_TIMEOUT})
```

<a name="comments"></a>
### Komentarze

Blade pozwala również definiować komentarze w widokach. Jednak w przeciwieństwie do komentarzy HTML, komentarze Blade nie są uwzględniane w HTML zwracanym przez aplikację:

```blade
{{-- Ten komentarz nie będzie obecny w renderowanym HTML --}}
```

<a name="components"></a>
## Komponenty

Komponenty i sloty zapewniają podobne korzyści jak sekcje, layouty i include'y; jednak niektórzy mogą uznać model mentalny komponentów i slotów za łatwiejszy do zrozumienia. Istnieją dwa podejścia do pisania komponentów: komponenty oparte na klasach i komponenty anonimowe.

Aby utworzyć komponent oparty na klasie, możesz użyć polecenia Artisan `make:component`. Aby zilustrować, jak używać komponentów, utworzymy prosty komponent `Alert`. Polecenie `make:component` umieści komponent w katalogu `app/View/Components`:

```shell
php artisan make:component Alert
```

Polecenie `make:component` utworzy również szablon widoku dla komponentu. Widok zostanie umieszczony w katalogu `resources/views/components`. Podczas pisania komponentów dla własnej aplikacji, komponenty są automatycznie wykrywane w katalogach `app/View/Components` i `resources/views/components`, więc zazwyczaj nie jest wymagana dalsza rejestracja komponentów.

Możesz również tworzyć komponenty w podkatalogach:

```shell
php artisan make:component Forms/Input
```

Powyższe polecenie utworzy komponent `Input` w katalogu `app/View/Components/Forms`, a widok zostanie umieszczony w katalogu `resources/views/components/forms`.

<a name="manually-registering-package-components"></a>
#### Ręczne rejestrowanie komponentów pakietów

Podczas pisania komponentów dla własnej aplikacji, komponenty są automatycznie wykrywane w katalogach `app/View/Components` i `resources/views/components`.

Jednak jeśli tworzysz pakiet, który wykorzystuje komponenty Blade, będziesz musiał ręcznie zarejestrować klasę komponentu i jej alias tagu HTML. Zazwyczaj powinie-neś rejestrować swoje komponenty w metodzie `boot` dostawcy usług pakietu:

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::component('package-alert', Alert::class);
}
```

Po zarejestrowaniu komponentu może on być renderowany za pomocą aliasu tagu:

```blade
<x-package-alert/>
```

Alternatywnie możesz użyć metody `componentNamespace`, aby automatycznie ładować klasy komponentów zgodnie z konwencją. Na przykład pakiet `Nightshade` może mieć komponenty `Calendar` i `ColorPicker`, które znajdują się w przestrzeni nazw `Package\Views\Components`:

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

To umo\u017cliwi u\u017cywanie komponent\u00f3w pakietu przez ich przestrze\u0144 nazw dostawcy za pomoc\u0105 sk\u0142adni `package-name::`:

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade automatycznie wykryje klas\u0119, kt\u00f3ra jest po\u0142\u0105czona z tym komponentem, przekszta\u0142caj\u0105c nazw\u0119 komponentu na PascalCase. Podkatalogi s\u0105 r\u00f3wnie\u017c obs\u0142ugiwane za pomoc\u0105 notacji \"kropkowej\".

<a name="rendering-components"></a>
### Renderowanie komponent\u00f3w

Aby wy\u015bwietli\u0107 komponent, mo\u017cesz u\u017cy\u0107 tagu komponentu Blade w jednym ze swoich szablon\u00f3w Blade. Tagi komponent\u00f3w Blade zaczynaj\u0105 si\u0119 od \u0142a\u0144cucha `x-`, po kt\u00f3rym nast\u0119puje nazwa klasy komponentu w formacie kebab-case:

```blade
<x-alert/>

<x-user-profile/>
```

Jeśli klasa komponentu jest zagnieżdżona głębiej w katalogu `app/View/Components`, możesz użyć znaku `.`, aby wskazać zagnieżdżenie katalogów. Na przykład, jeśli założymy, że komponent znajduje się w `app/View/Components/Inputs/Button.php`, możemy go wyrenderować w następujący sposób:

```blade
<x-inputs.button/>
```

Jeśli chcesz warunkowo renderować swój komponent, możesz zdefiniować metodę `shouldRender` w klasie komponentu. Jeśli metoda `shouldRender` zwróci `false`, komponent nie zostanie wyrenderowany:

```php
use Illuminate\Support\Str;

/**
 * Whether the component should be rendered
 */
public function shouldRender(): bool
{
    return Str::length($this->message) > 0;
}
```

<a name="index-components"></a>
### Komponenty indeksowe

Czasami komponenty są częścią grupy komponentów i możesz chcieć pogrupować powiązane komponenty w jednym katalogu. Na przykład wyobraź sobie komponent "card" z następującą strukturą klas:

```text
App\Views\Components\Card\Card
App\Views\Components\Card\Header
App\Views\Components\Card\Body
```

Ponieważ główny komponent `Card` jest zagnieżdżony w katalogu `Card`, możesz się spodziewać, że będziesz musiał renderować komponent za pomocą `<x-card.card>`. Jednak gdy nazwa pliku komponentu odpowiada nazwie katalogu komponentu, Laravel automatycznie zakłada, że komponent jest komponentem "głównym" i pozwala renderować komponent bez powtarzania nazwy katalogu:

```blade
<x-card>
    <x-card.header>...</x-card.header>
    <x-card.body>...</x-card.body>
</x-card>
```

<a name="passing-data-to-components"></a>
### Przekazywanie danych do komponentów

Możesz przekazywać dane do komponentów Blade za pomocą atrybutów HTML. Zakodowane na stałe, prymitywne wartości mogą być przekazywane do komponentu za pomocą prostych łańcuchów atrybutów HTML. Wyrażenia PHP i zmienne powinny być przekazywane do komponentu za pomocą atrybutów używających znaku `:` jako przedrostka:

```blade
<x-alert type="error" :message="$message"/>
```

Powinie-neś zdefiniować wszystkie atrybuty danych komponentu w jego konstruktorze klasy. Wszystkie właściwości publiczne komponentu będą automatycznie udostępnione widokowi komponentu. Nie jest konieczne przekazywanie danych do widoku z metody `render` komponentu:

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;
use Illuminate\View\View;

class Alert extends Component
{
    /**
     * Create the component instance.
     */
    public function __construct(
        public string $type,
        public string $message,
    ) {}

    /**
     * Get the view / contents that represent the component.
     */
    public function render(): View
    {
        return view('components.alert');
    }
}
```

Gdy komponent jest renderowany, możesz wyświetlić zawartość publicznych zmiennych komponentu, wyprowadzając zmienne po nazwie:

```blade
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

<a name="casing"></a>
#### Forma liter

Argumenty konstruktora komponentów powinny być określane przy użyciu `camelCase`, podczas gdy `kebab-case` powinien być używany podczas odwoływania się do nazw argumentów w atrybutach HTML. Na przykład, mając następujący konstruktor komponentu:

```php
/**
 * Create the component instance.
 */
public function __construct(
    public string $alertType,
) {}
```

Argument `$alertType` może zostać przekazany do komponentu w następujący sposób:

```blade
<x-alert alert-type="danger" />
```

<a name="short-attribute-syntax"></a>
#### Skrócona składnia atrybutów

Podczas przekazywania atrybutów do komponentów możesz również użyć "skróconej składni atrybutów". Jest to często wygodne, ponieważ nazwy atrybutów często odpowiadają nazwom zmiennych, którym odpowiadają:

```blade
{{-- Short attribute syntax... --}}
<x-profile :$userId :$name />

{{-- Is equivalent to... --}}
<x-profile :user-id="$userId" :name="$name" />
```

<a name="escaping-attribute-rendering"></a>
#### Escapowanie renderowania atrybutów

Ponieważ niektóre frameworki JavaScript, takie jak Alpine.js, również używają atrybutów z przedrostkiem dwukropka, możesz użyć przedrostka podwójnego dwukropka (`::`) aby poinformować Blade, że atrybut nie jest wyrażeniem PHP. Na przykład, mając następujący komponent:

```blade
<x-button ::class="{ danger: isDeleting }">
    Submit
</x-button>
```

Następujący HTML zostanie wyrenderowany przez Blade:

```blade
<button :class="{ danger: isDeleting }">
    Submit
</button>
```

<a name="component-methods"></a>
#### Metody komponentów

Oprócz publicznych zmiennych dostępnych dla szablonu komponentu, wszelkie metody publiczne komponentu mogą zostać wywołane. Na przykład wyobraź sobie komponent, który ma metodę `isSelected`:

```php
/**
 * Determine if the given option is the currently selected option.
 */
public function isSelected(string $option): bool
{
    return $option === $this->selected;
}
```

Możesz wykonać tę metodę ze swojego szablonu komponentu, wywołując zmienną pasującą do nazwy metody:

```blade
<option {{ $isSelected($value) ? 'selected' : '' }} value="{{ $value }}">
    {{ $label }}
</option>
```

<a name="using-attributes-slots-within-component-class"></a>
#### Dostęp do atrybutów i slotów w klasach komponentów

Komponenty Blade pozwalają również na dostęp do nazwy komponentu, atrybutów i slotu wewnątrz metody render klasy. Jednak aby uzyskać dostęp do tych danych, powinie-neś zwrócić domknięcie (closure) z metody `render` komponentu:

```php
use Closure;

/**
 * Get the view / contents that represent the component.
 */
public function render(): Closure
{
    return function () {
        return '<div {{ $attributes }}>Components content</div>';
    };
}
```

Domknięcie zwracane przez metodę `render` komponentu może również otrzymywać tablicę `$data` jako jedyny argument. Ta tablica będzie zawierać kilka elementów, które dostarczą informacji o komponencie:

```php
return function (array $data) {
    // $data['componentName'];
    // $data['attributes'];
    // $data['slot'];

    return '<div {{ $attributes }}>Components content</div>';
}
```

> [!WARNING]
> Elementy w tablicy `$data` nigdy nie powinny być bezpośrednio osadzane w łańcuchu Blade zwracanym przez metodę `render`, ponieważ takie postępowanie mogłoby umożliwić zdalne wykonanie kodu za pomocą złośliwej zawartości atrybutów.

`componentName` jest równy nazwie używanej w tagu HTML po przedrostku `x-`. Więc `componentName` dla `<x-alert />` będzie `alert`. Element `attributes` będzie zawierać wszystkie atrybuty, które były obecne w tagu HTML. Element `slot` jest instancją `Illuminate\Support\HtmlString` z zawartością slotu komponentu.

Domknięcie powinno zwrócić łańcuch. Jeśli zwrócony łańcuch odpowiada istniejącemu widokowi, ten widok zostanie wyrenderowany; w przeciwnym razie zwrócony łańcuch zostanie oceniony jako inline widok Blade.

<a name="additional-dependencies"></a>
#### Dodatkowe zależności

Jeśli Twój komponent wymaga zależności z [kontenera serwisów](/docs/{{version}}/container) Laravela, możesz je wymienić przed dowolnymi atrybutami danych komponentu, a zostaną one automatycznie wstrzyknięte przez kontener:

```php
use App\Services\AlertCreator;

/**
 * Create the component instance.
 */
public function __construct(
    public AlertCreator $creator,
    public string $type,
    public string $message,
) {}
```

<a name="hiding-attributes-and-methods"></a>
#### Ukrywanie atrybutów / metod

Jeśli chcesz uniemożliwić eksponowanie niektórych metod lub właściwości publicznych jako zmiennych w szablonie komponentu, możesz dodać je do właściwości tablicy `$except` w komponencie:

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    /**
     * The properties / methods that should not be exposed to the component template.
     *
     * @var array
     */
    protected $except = ['type'];

    /**
     * Create the component instance.
     */
    public function __construct(
        public string $type,
    ) {}
}
```

<a name="component-attributes"></a>
### Atrybuty komponentów

Już zbadaliśmy, jak przekazywać atrybuty danych do komponentu; jednak czasami możesz potrzebować określić dodatkowe atrybuty HTML, takie jak `class`, które nie są częścią danych wymaganych do działania komponentu. Zazwyczaj chcesz przekazać te dodatkowe atrybuty do elementu głównego szablonu komponentu. Na przykład wyobraź sobie, że chcemy wyrenderować komponent `alert` w następujący sposób:

```blade
<x-alert type="error" :message="$message" class="mt-4"/>
```

Wszystkie atrybuty, które nie są częścią konstruktora komponentu, zostaną automatycznie dodane do "worka atrybutów" komponentu. Ten worek atrybutów jest automatycznie udostępniany komponentowi za pomocą zmiennej `$attributes`. Wszystkie atrybuty mogą zostać wyrenderowane w komponencie przez wyprowadzenie tej zmiennej:

```blade
<div {{ $attributes }}>
    <!-- Component content -->
</div>
```

> [!WARNING]
> Używanie dyrektyw takich jak `@env` wewnątrz tagów komponentów nie jest obecnie obsługiwane. Na przykład `<x-alert :live="@env('production')"/>` nie zostanie skompilowane.

<a name="default-merged-attributes"></a>
#### Domyślne / łączone atrybuty

Czasami możesz potrzebować określić wartości domyślne dla atrybutów lub połączyć dodatkowe wartości z niektórymi atrybutami komponentu. Aby to osiągnąć, możesz użyć metody `merge` worka atrybutów. Ta metoda jest szczególnie przydatna do definiowania zestawu domyślnych klas CSS, które powinny być zawsze stosowane do komponentu:

```blade
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

Jeśli założymy, że ten komponent jest używany w następujący sposób:

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

Końcowy, wyrenderowany HTML komponentu będzie wyglądać następująco:

```blade
<div class="alert alert-error mb-4">
    <!-- Contents of the $message variable -->
</div>
```

<a name="conditionally-merge-classes"></a>
#### Warunkowe łączenie klas

Czasami możesz chcieć połączyć klasy, jeśli dany warunek jest `true`. Możesz to osiągnąć za pomocą metody `class`, która przyjmuje tablicę klas, gdzie klucz tablicy zawiera klasę lub klasy, które chcesz dodać, podczas gdy wartość jest wyrażeniem logicznym. Jeśli element tablicy ma klucz numeryczny, zawsze będzie uwzględniony na liście renderowanych klas:

```blade
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
    {{ $message }}
</div>
```

Jeśli musisz połączyć inne atrybuty z komponentem, możesz połączyć metodę `merge` z metodą `class`:

```blade
<button {{ $attributes->class(['p-4'])->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

> [!NOTE]
> Jeśli musisz warunkowo kompilować klasy na innych elementach HTML, które nie powinny otrzymywać połączonych atrybutów, możesz użyć [dyrektywy @class](#conditional-classes).

<a name="non-class-attribute-merging"></a>
#### Łączenie atrybutów innych niż class

Podczas łączenia atrybutów, które nie są atrybutami `class`, wartości przekazane do metody `merge` będą uznawane za "domyślne" wartości atrybutu. Jednak w przeciwieństwie do atrybutu `class`, te atrybuty nie będą łączone z wstrzykniętymi wartościami atrybutów. Zamiast tego zostaną nadpisane. Na przykład implementacja komponentu `button` może wyglądać następująco:

```blade
<button {{ $attributes->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

Aby wyrenderować komponent przycisku z niestandardowym `type`, może on zostać określony podczas konsumowania komponentu. Jeśli żaden typ nie zostanie określony, zostanie użyty typ `button`:

```blade
<x-button type="submit">
    Submit
</x-button>
```

Wyrenderowany HTML komponentu `button` w tym przykładzie byłby:

```blade
<button type="submit">
    Submit
</button>
```

Jeśli chcesz, aby atrybut inny niż `class` miał swoją wartość domyślną i wstrzyknięte wartości połączone razem, możesz użyć metody `prepends`. W tym przykładzie atrybut `data-controller` zawsze będzie zaczynać się od `profile-controller`, a wszelkie dodatkowe wstrzyknięte wartości `data-controller` zostaną umieszczone po tej wartości domyślnej:

```blade
<div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
    {{ $slot }}
</div>
```

<a name="filtering-attributes"></a>
#### Pobieranie i filtrowanie atrybutów

Możesz filtrować atrybuty za pomocą metody `filter`. Ta metoda przyjmuje domknięcie, które powinno zwrócić `true`, jeśli chcesz zachować atrybut w worku atrybutów:

```blade
{{ $attributes->filter(fn (string $value, string $key) => $key == 'foo') }}
```

Dla wygody możesz użyć metody `whereStartsWith`, aby pobrać wszystkie atrybuty, których klucze zaczynają się od danego łańcucha:

```blade
{{ $attributes->whereStartsWith('wire:model') }}
```

Odwrotnie, metoda `whereDoesntStartWith` może być użyta do wykluczenia wszystkich atrybutów, których klucze zaczynają się od danego łańcucha:

```blade
{{ $attributes->whereDoesntStartWith('wire:model') }}
```

Używając metody `first`, możesz wyrenderować pierwszy atrybut w danym worku atrybutów:

```blade
{{ $attributes->whereStartsWith('wire:model')->first() }}
```

Jeśli chcesz sprawdzić, czy atrybut jest obecny w komponencie, możesz użyć metody `has`. Ta metoda przyjmuje nazwę atrybutu jako jedyny argument i zwraca wartość logiczną wskazującą, czy atrybut jest obecny:

```blade
@if ($attributes->has('class'))
    <div>Class attribute is present</div>
@endif
```

Jeśli tablica zostanie przekazana do metody `has`, metoda określi, czy wszystkie podane atrybuty są obecne w komponencie:

```blade
@if ($attributes->has(['name', 'class']))
    <div>All of the attributes are present</div>
@endif
```

Metoda `hasAny` może być użyta do określenia, czy którykolwiek z podanych atrybutów jest obecny w komponencie:

```blade
@if ($attributes->hasAny(['href', ':href', 'v-bind:href']))
    <div>One of the attributes is present</div>
@endif
```

Możesz pobrać wartość konkretnego atrybutu za pomocą metody `get`:

```blade
{{ $attributes->get('class') }}
```

Metoda `only` może być użyta do pobrania tylko atrybutów z podanymi kluczami:

```blade
{{ $attributes->only(['class']) }}
```

Metoda `except` może być użyta do pobrania wszystkich atrybutów oprócz tych z podanymi kluczami:

```blade
{{ $attributes->except(['class']) }}
```

<a name="reserved-keywords"></a>
### Słowa kluczowe zarezerwowane

Domyślnie niektóre słowa kluczowe są zarezerwowane do użytku wewnętrznego Blade w celu renderowania komponentów. Następujące słowa kluczowe nie mogą być definiowane jako właściwości publiczne lub nazwy metod w Twoich komponentach:

<div class="content-list" markdown="1">

- `data`
- `render`
- `resolve`
- `resolveView`
- `shouldRender`
- `view`
- `withAttributes`
- `withName`

</div>

<a name="slots"></a>
### Sloty

Często będziesz potrzebować przekazać dodatkową zawartość do komponentu za pomocą "slotów". Sloty komponentów są renderowane przez wyprowadzenie zmiennej `$slot`. Aby zbadać tę koncepcję, wyobraźmy sobie, że komponent `alert` ma następujące oznakowanie:

```blade
<!-- /resources/views/components/alert.blade.php -->

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

Możemy przekazać zawartość do `slotu`, wstrzykując zawartość do komponentu:

```blade
<x-alert>
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

Czasami komponent może potrzebować renderować wiele różnych slotów w różnych lokalizacjach wewnątrz komponentu. Zmodyfikujmy nasz komponent alertu, aby umożliwić wstrzyknięcie slotu "title":

```blade
<!-- /resources/views/components/alert.blade.php -->

<span class="alert-title">{{ $title }}</span>

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

Możesz zdefiniować zawartość nazwanego slotu za pomocą tagu `x-slot`. Jakakolwiek zawartość nie znajdująca się w jawnym tagu `x-slot` zostanie przekazana do komponentu w zmiennej `$slot`:

```xml
<x-alert>
    <x-slot:title>
        Server Error
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

Możesz wywołać metodę `isEmpty` slotu, aby określić, czy slot zawiera zawartość:

```blade
<span class="alert-title">{{ $title }}</span>

<div class="alert alert-danger">
    @if ($slot->isEmpty())
        This is default content if the slot is empty.
    @else
        {{ $slot }}
    @endif
</div>
```

Dodatkowo metoda `hasActualContent` może być użyta do określenia, czy slot zawiera jakąkolwiek "rzeczywistą" zawartość, która nie jest komentarzem HTML:

```blade
@if ($slot->hasActualContent())
    The scope has non-comment content.
@endif
```

<a name="scoped-slots"></a>
#### Sloty zakresowe

Jeśli używałeś frameworka JavaScript takiego jak Vue, możesz być zaznajomiony ze "slotami zakresowymi", które pozwalają uzyskać dostęp do danych lub metod z komponentu w slocie. Możesz osiągnąć podobne zachowanie w Laravelu, definiując metody lub właściwości publiczne w komponencie i uzyskując dostęp do komponentu w slocie za pomocą zmiennej `$component`. W tym przykładzie założymy, że komponent `x-alert` ma publiczną metodę `formatAlert` zdefiniowa-ną w swojej klasie komponentu:

```blade
<x-alert>
    <x-slot:title>
        {{ $component->formatAlert('Server Error') }}
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

<a name="slot-attributes"></a>
#### Atrybuty slotów

Podobnie jak komponenty Blade, możesz przypisać dodatkowe [atrybuty](#component-attributes) do slotów, takie jak nazwy klas CSS:

```xml
<x-card class="shadow-sm">
    <x-slot:heading class="font-bold">
        Heading
    </x-slot>

    Content

    <x-slot:footer class="text-sm">
        Footer
    </x-slot>
</x-card>
```

Aby wchodzić w interakcje z atrybutami slotu, możesz uzyskać dostęp do właściwości `attributes` zmiennej slotu. Więcej informacji o interakcji z atrybutami można znaleźć w dokumentacji dotyczącej [atrybutów komponentów](#component-attributes):

```blade
@props([
    'heading',
    'footer',
])

<div {{ $attributes->class(['border']) }}>
    <h1 {{ $heading->attributes->class(['text-lg']) }}>
        {{ $heading }}
    </h1>

    {{ $slot }}

    <footer {{ $footer->attributes->class(['text-gray-700']) }}>
        {{ $footer }}
    </footer>
</div>
```

<a name="inline-component-views"></a>
### Widoki komponentów inline

Dla bardzo małych komponentów zarządzanie zarówno klasą komponentu, jak i szablonem widoku komponentu może wydawać się uciążliwe. Z tego powodu możesz zwrócić oznakowanie komponentu bezpośrednio z metody `render`:

```php
/**
 * Get the view / contents that represent the component.
 */
public function render(): string
{
    return <<<'blade'
        <div class="alert alert-danger">
            {{ $slot }}
        </div>
    blade;
}
```

<a name="generating-inline-view-components"></a>
#### Generowanie inline komponentów widoku

Aby utworzyć komponent, który renderuje widok inline, możesz użyć opcji `inline` podczas wykonywania polecenia `make:component`:

```shell
php artisan make:component Alert --inline
```

<a name="dynamic-components"></a>
### Komponenty dynamiczne

Czasami możesz potrzebować wyrenderować komponent, ale nie wiedzieć, który komponent powinien zostać wyrenderowany do czasu wykonania. W tej sytuacji możesz użyć wbudowanego komponentu `dynamic-component` Laravela, aby wyrenderować komponent w oparciu o wartość lub zmienną w czasie wykonania:

```blade
// $componentName = "secondary-button";

<x-dynamic-component :component="$componentName" class="mt-4" />
```

<a name="manually-registering-components"></a>
### Ręczna rejestracja komponentów

> [!WARNING]
> Następująca dokumentacja dotycząca ręcznej rejestracji komponentów jest głównie skierowana do osób piszucących pakiety Laravel, które zawierają komponenty widoku. Jeśli nie piszesz pakietu, ta część dokumentacji komponentów może nie być dla Ciebie istotna.

Podczas pisania komponentów dla własnej aplikacji, komponenty są automatycznie wykrywane w katalogach `app/View/Components` i `resources/views/components`.

Jednak jeśli tworzysz pakiet, który wykorzystuje komponenty Blade, lub umieszczasz komponenty w niekonwencjonalnych katalogach, będziesz musiał ręcznie zarejestrować klasę komponentu i jej alias tagu HTML, aby Laravel wiedział, gdzie znaleźć komponent. Zazwyczaj powinie-neś rejestrować swoje komponenty w metodzie `boot` dostawcy usług pakietu:

```php
use Illuminate\Support\Facades\Blade;
use VendorPackage\View\Components\AlertComponent;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::component('package-alert', AlertComponent::class);
}
```

Po zarejestrowaniu komponentu może on być renderowany za pomocą aliasu tagu:

```blade
<x-package-alert/>
```

#### Automatyczne ładowanie komponentów pakietu

Alternatywnie możesz użyć metody `componentNamespace`, aby automatycznie ładować klasy komponentów zgodnie z konwencją. Na przykład pakiet `Nightshade` może mieć komponenty `Calendar` i `ColorPicker`, które znajdują się w przestrzeni nazw `Package\Views\Components`:

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

To umożliwi użycie komponentów pakietu przez ich przestrzeń nazw dostawcy za pomocą składni `package-name::`:

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade automatycznie wykryje klasę, która jest powiązana z tym komponentem, przekształcając nazwę komponentu na PascalCase. Podkatalogi są również obsługiwane za pomocą notacji "kropkowej".

<a name="anonymous-components"></a>
## Komponenty anonimowe

Podobnie jak komponenty inline, komponenty anonimowe zapewniają mechanizm zarządzania komponentem za pomocą pojedynczego pliku. Jednak komponenty anonimowe wykorzystują pojedynczy plik widoku i nie mają powiązanej klasy. Aby zdefiniować komponent anonimowy, wystarczy umieścić szablon Blade w katalogu `resources/views/components`. Na przykład, zakładając, że zdefiniowałeś komponent w `resources/views/components/alert.blade.php`, możesz po prostu go wyrenderować w następujący sposób:

```blade
<x-alert/>
```

Możesz użyć znaku `.`, aby wskazać, czy komponent jest zagnieżdżony głębiej w katalogu `components`. Na przykład, zakładając, że komponent jest zdefiniowany w `resources/views/components/inputs/button.blade.php`, możesz go wyrenderować w następujący sposób:

```blade
<x-inputs.button/>
```

Aby utworzyć komponent anonimowy za pomocą Artisan, możesz użyć flagi `--view` podczas wywoływania polecenia `make:component`:

```shell
php artisan make:component forms.input --view
```

Powyższe polecenie utworzy plik Blade w `resources/views/components/forms/input.blade.php`, który może być renderowany jako komponent za pomocą `<x-forms.input />`.

<a name="anonymous-index-components"></a>
### Anonimowe komponenty indeksowe

Czasami, gdy komponent składa się z wielu szablonów Blade, możesz chcieć pogrupować szablony danego komponentu w jednym katalogu. Na przykład wyobraź sobie komponent "accordion" z następującą strukturą katalogów:

```text
/resources/views/components/accordion.blade.php
/resources/views/components/accordion/item.blade.php
```

Ta struktura katalogów pozwala renderować komponent accordion i jego element w następujący sposób:

```blade
<x-accordion>
    <x-accordion.item>
        ...
    </x-accordion.item>
</x-accordion>
```

Jednak, aby wyrenderować komponent accordion za pomocą `x-accordion`, byliśmy zmuszeni umieścić "indeksowy" szablon komponentu accordion w katalogu `resources/views/components` zamiast zagnieździć go w katalogu `accordion` z innymi szablonami związanymi z accordion.

Na szczęście Blade pozwala umieścić plik pasujący do nazwy katalogu komponentu w samym katalogu komponentu. Gdy ten szablon istnieje, może być renderowany jako "główny" element komponentu, nawet jeśli jest zagnieżdżony w katalogu. Możemy więc nadal używać tej samej składni Blade podanej w powyższym przykładzie; jednak dostosujemy naszą strukturę katalogów w następujący sposób:

```text
/resources/views/components/accordion/accordion.blade.php
/resources/views/components/accordion/item.blade.php
```

<a name="data-properties-attributes"></a>
### Właściwości danych / Atrybuty

Ponieważ komponenty anonimowe nie mają żadnej powiązanej klasy, możesz się zastanawiać, jak rozróżnić, które dane powinny być przekazane do komponentu jako zmienne, a które atrybuty powinny być umieszczone w [worku atrybutów](#component-attributes) komponentu.

Możesz określić, które atrybuty powinny być uznane za zmienne danych, używając dyrektywy `@props` na górze szablonu Blade komponentu. Wszystkie inne atrybuty komponentu będą dostępne za pośrednictwem worka atrybutów komponentu. Jeśli chcesz nadać zmiennej danych wartość domyślną, możesz określić nazwę zmiennej jako klucz tablicy, a wartość domyślną jako wartość tablicy:

```blade
<!-- /resources/views/components/alert.blade.php -->

@props(['type' => 'info', 'message'])

<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

Mając powyższą definicję komponentu, możemy renderować komponent w następujący sposób:

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

<a name="accessing-parent-data"></a>
### Dostęp do danych nadrzędnych

Czasami możesz chcieć uzyskać dostęp do danych z komponentu nadrzędnego wewnątrz komponentu potomnego. W takich przypadkach możesz użyć dyrektywy `@aware`. Na przykład wyobraź sobie, że budujemy złożony komponent menu składający się z nadrzędnego `<x-menu>` i potomnego `<x-menu.item>`:

```blade
<x-menu color="purple">
    <x-menu.item>...</x-menu.item>
    <x-menu.item>...</x-menu.item>
</x-menu>
```

Komponent `<x-menu>` może mieć implementację taką jak poniżej:

```blade
<!-- /resources/views/components/menu/index.blade.php -->

@props(['color' => 'gray'])

<ul {{ $attributes->merge(['class' => 'bg-'.$color.'-200']) }}>
    {{ $slot }}
</ul>
```

Ponieważ właściwość `color` została przekazana tylko do rodzica (`<x-menu>`), nie będzie dostępna wewnątrz `<x-menu.item>`. Jednak jeśli użyjemy dyrektywy `@aware`, możemy ją udostępnić również wewnątrz `<x-menu.item>`:

```blade
<!-- /resources/views/components/menu/item.blade.php -->

@aware(['color' => 'gray'])

<li {{ $attributes->merge(['class' => 'text-'.$color.'-800']) }}>
    {{ $slot }}
</li>
```

> [!WARNING]
> Dyrektywa `@aware` nie może uzyskać dostępu do danych nadrzędnych, które nie są jawnie przekazane do komponentu nadrzędnego za pomocą atrybutów HTML. Domyślne wartości `@props`, które nie są jawnie przekazane do komponentu nadrzędnego, nie mogą być dostępne przez dyrektywę `@aware`.

<a name="anonymous-component-paths"></a>
### Ścieżki komponentów anonimowych

Jak wcześniej omówiono, komponenty anonimowe są zazwyczaj definiowane przez umieszczenie szablonu Blade w katalogu `resources/views/components`. Jednak czasami możesz chcieć zarejestrować inne ścieżki komponentów anonimowych w Laravelu oprócz domyślnej ścieżki.

Metoda `anonymousComponentPath` przyjmuje "ścieżkę" do lokalizacji komponentu anonimowego jako pierwszy argument oraz opcjonalną "przestrzeń nazw", w której powinny być umieszczone komponenty, jako drugi argument. Zazwyczaj ta metoda powinna być wywoływana z metody `boot` jednego z [dostawców usług](/docs/{{version}}/providers) Twojej aplikacji:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Blade::anonymousComponentPath(__DIR__.'/../components');
}
```

Gdy ścieżki komponentów są rejestrowane bez określonego przedrostka, jak w powyższym przykładzie, mogą być renderowane w komponentach Blade również bez odpowiadającego przedrostka. Na przykład, jeśli komponent `panel.blade.php` istnieje w ścieżce zarejestrowanej powyżej, może być renderowany w następujący sposób:

```blade
<x-panel />
```

"Przestrzenie nazw" z przedrostkiem mogą być podane jako drugi argument metody `anonymousComponentPath`:

```php
Blade::anonymousComponentPath(__DIR__.'/../components', 'dashboard');
```

Gdy podany jest przedrostek, komponenty w tej "przestrzeni nazw" mogą być renderowane przez dodanie przedrostka przestrzeni nazw komponentu do nazwy komponentu podczas renderowania komponentu:

```blade
<x-dashboard::panel />
```

<a name="building-layouts"></a>
## Budowanie layoutów

<a name="layouts-using-components"></a>
### Layouty za pomocą komponentów

Większość aplikacji webowych utrzymuje ten sam ogólny układ na różnych stronach. Byłoby niezwykle uciążliwe i trudne w utrzymaniu naszej aplikacji, gdybyśmy musieli powtarzać cały HTML layoutu w każdym widoku, który tworzymy. Na szczęście wygodne jest zdefiniowanie tego layoutu jako pojedynczego [komponentu Blade](#components), a następnie używanie go w całej naszej aplikacji.

<a name="defining-the-layout-component"></a>
#### Definiowanie komponentu layoutu

Na przykład wyobraź sobie, że budujemy aplikację do listy zadań "todo". Możemy zdefiniować komponent `layout`, który wygląda następująco:

```blade
<!-- resources/views/components/layout.blade.php -->

<html>
    <head>
        <title>{{ $title ?? 'Todo Manager' }}</title>
    </head>
    <body>
        <h1>Todos</h1>
        <hr/>
        {{ $slot }}
    </body>
</html>
```

<a name="applying-the-layout-component"></a>
#### Stosowanie komponentu layoutu

Po zdefiniowaniu komponentu `layout` możemy utworzyć widok Blade, który wykorzystuje komponent. W tym przykładzie zdefiniujemy prosty widok, który wyświetla naszą listę zadań:

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    @foreach ($tasks as $task)
        <div>{{ $task }}</div>
    @endforeach
</x-layout>
```

Pamiętaj, że zawartość wstrzykiwana do komponentu zostanie dostarczona do domyślnej zmiennej `$slot` w naszym komponencie `layout`. Jak mogłeś zauważyć, nasz `layout` również respektuje slot `$title`, jeśli jest podany; w przeciwnym razie wyświetlany jest domyślny tytuł. Możemy wstrzyknąć niestandardowy tytuł z naszego widoku listy zadań, używając standardowej składni slotów omówionej w [dokumentacji komponentów](#components):

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    <x-slot:title>
        Custom Title
    </x-slot>

    @foreach ($tasks as $task)
        <div>{{ $task }}</div>
    @endforeach
</x-layout>
```

Teraz, gdy zdefiniowaliśmy nasze widoki layoutu i listy zadań, musimy tylko zwrócić widok `task` z trasy:

```php
use App\Models\Task;

Route::get('/tasks', function () {
    return view('tasks', ['tasks' => Task::all()]);
});
```

<a name="layouts-using-template-inheritance"></a>
### Layouty za pomocą dziedziczenia szablonów

<a name="defining-a-layout"></a>
#### Definiowanie layoutu

Layouty mogą być również tworzone za pomocą "dziedziczenia szablonów". To był główny sposób budowania aplikacji przed wprowadzeniem [komponentów](#components).

Aby zacząć, spójrzmy na prosty przykład. Najpierw zbadamy layout strony. Ponieważ większość aplikacji webowych utrzymuje ten sam ogólny układ na różnych stronach, wygodnie jest zdefiniować ten layout jako pojedynczy widok Blade:

```blade
<!-- resources/views/layouts/app.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

Jak widać, ten plik zawiera typowe oznakowanie HTML. Jednak zwróć uwagę na dyrektywy `@section` i `@yield`. Dyrektywa `@section`, jak sama nazwa wskazuje, definiuje sekcję zawartości, podczas gdy dyrektywa `@yield` służy do wyświetlania zawartości danej sekcji.

Teraz, gdy zdefiniowaliśmy layout dla naszej aplikacji, zdefiniujmy stronę potomną, która dziedziczy layout.

<a name="extending-a-layout"></a>
#### Rozszerzanie layoutu

Podczas definiowania widoku potomnego użyj dyrektywy Blade `@extends`, aby określić, który layout widok potomny powinien "dziedziczyć". Widoki, które rozszerzają layout Blade, mogą wstrzykiwać zawartość do sekcji layoutu za pomocą dyrektyw `@section`. Pamiętaj, jak widać w powyższym przykładzie, zawartość tych sekcji zostanie wyświetlona w layoucie za pomocą `@yield`:

```blade
<!-- resources/views/child.blade.php -->

@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @@parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

W tym przykładzie sekcja `sidebar` wykorzystuje dyrektywę `@@parent` do dołączania (zamiast nadpisywania) zawartości do paska bocznego layoutu. Dyrektywa `@@parent` zostanie zastąpiona zawartością layoutu podczas renderowania widoku.

> [!NOTE]
> W przeciwieństwie do poprzedniego przykładu, ta sekcja `sidebar` kończy się `@endsection` zamiast `@show`. Dyrektywa `@endsection` tylko zdefiniuje sekcję, podczas gdy `@show` zdefiniuje i **natychmiast wyprowadzi** sekcję.

Dyrektywa `@yield` akceptuje również wartość domyślną jako drugi parametr. Ta wartość zostanie wyrenderowana, jeśli wyprowadzana sekcja jest niezdefiniowana:

```blade
@yield('content', 'Default content')
```

<a name="forms"></a>
## Formularze

<a name="csrf-field"></a>
### Pole CSRF

Za każdym razem, gdy definiujesz formularz HTML w swojej aplikacji, powinieneś dołączyć ukryte pole tokena CSRF w formularzu, aby middleware [ochrony CSRF](/docs/{{version}}/csrf) mógł zwalidować żądanie. Możesz użyć dyrektywy Blade `@csrf`, aby wygenerować pole tokena:

```blade
<form method="POST" action="/profile">
    @csrf

    ...
</form>
```

<a name="method-field"></a>
### Pole Method

Ponieważ formularze HTML nie mogą wykonywać żądań `PUT`, `PATCH` lub `DELETE`, będziesz musiał dodać ukryte pole `_method`, aby zasymulować te czasowniki HTTP. Dyrektywa Blade `@method` może utworzyć to pole za Ciebie:

```blade
<form action="/foo/bar" method="POST">
    @method('PUT')

    ...
</form>
```

<a name="validation-errors"></a>
### Błędy walidacji

Dyrektywa `@error` może być użyta do szybkiego sprawdzenia, czy istnieją [komunikaty błędów walidacji](/docs/{{version}}/validation#quick-displaying-the-validation-errors) dla danego atrybutu. Wewnątrz dyrektywy `@error` możesz wyprowadzić zmienną `$message`, aby wyświetlić komunikat błędu:

```blade
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input
    id="title"
    type="text"
    class="@error('title') is-invalid @enderror"
/>

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

Ponieważ dyrektywa `@error` kompiluje się do instrukcji "if", możesz użyć dyrektywy `@else`, aby renderować zawartość, gdy nie ma błędu dla atrybutu:

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input
    id="email"
    type="email"
    class="@error('email') is-invalid @else is-valid @enderror"
/>
```

Możesz przekazać [nazwę konkretnego worka błędów](/docs/{{version}}/validation#named-error-bags) jako drugi parametr do dyrektywy `@error`, aby pobrać komunikaty błędów walidacji na stronach zawierających wiele formularzy:

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input
    id="email"
    type="email"
    class="@error('email', 'login') is-invalid @enderror"
/>

@error('email', 'login')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

<a name="stacks"></a>
## Stosy

Blade pozwala przekazywać do nazwanych stosów, które mogą być renderowane gdzieś indziej w innym widoku lub layoucie. Może to być szczególnie przydatne do określania bibliotek JavaScript wymaganych przez widoki potomne:

```blade
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

Jeśli chcesz użyć `@push` do zawartości, jeśli dane wyrażenie logiczne zostanie ocenione jako `true`, możesz użyć dyrektywy `@pushIf`:

```blade
@pushIf($shouldPush, 'scripts')
    <script src="/example.js"></script>
@endPushIf
```

Możesz przekazywać do stosu tyle razy, ile potrzebujesz. Aby wyrenderować pełną zawartość stosu, przekaż nazwę stosu do dyrektywy `@stack`:

```blade
<head>
    <!-- Head Contents -->

    @stack('scripts')
</head>
```

Jeśli chcesz dodać zawartość na początku stosu, powinieneś użyć dyrektywy `@prepend`:

```blade
@push('scripts')
    This will be second...
@endpush

// Later...

@prepend('scripts')
    This will be first...
@endprepend
```

Dyrektywa `@hasstack` może być użyta do określenia, czy stos jest pusty:

```blade
@hasstack('list')
    <ul>
        @stack('list')
    </ul>
@endif
```

<a name="service-injection"></a>
## Wstrzykiwanie serwisów

Dyrektywa `@inject` może być użyta do pobierania serwisu z [kontenera serwisów](/docs/{{version}}/container) Laravela. Pierwszy argument przekazany do `@inject` to nazwa zmiennej, do której zostanie umieszczony serwis, podczas gdy drugi argument to nazwa klasy lub interfejsu serwisu, który chcesz rozwiązać:

```blade
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

<a name="rendering-inline-blade-templates"></a>
## Renderowanie inline szablonów Blade

Czasami możesz potrzebować przekształcić surowy łańcuch szablonu Blade w prawidłowy HTML. Możesz to osiągnąć, używając metody `render` dostarczonej przez fasadę `Blade`. Metoda `render` przyjmuje łańcuch szablonu Blade i opcjonalną tablicę danych do dostarczenia do szablonu:

```php
use Illuminate\Support\Facades\Blade;

return Blade::render('Hello, {{ $name }}', ['name' => 'Julian Bashir']);
```

Laravel renderuje inline szablony Blade, zapisując je do katalogu `storage/framework/views`. Jeśli chcesz, aby Laravel usunął te tymczasowe pliki po wyrenderowaniu szablonu Blade, możesz przekazać argument `deleteCachedView` do metody:

```php
return Blade::render(
    'Hello, {{ $name }}',
    ['name' => 'Julian Bashir'],
    deleteCachedView: true
);
```

<a name="rendering-blade-fragments"></a>
## Renderowanie fragmentów Blade

Podczas korzystania z frameworków frontendowych, takich jak [Turbo](https://turbo.hotwired.dev/) i [htmx](https://htmx.org/), możesz czasami potrzebować zwrócić tylko część szablonu Blade w swojej odpowiedzi HTTP. "Fragmenty" Blade pozwalają Ci to zrobić. Aby zacząć, umieść część swojego szablonu Blade w dyrektywach `@fragment` i `@endfragment`:

```blade
@fragment('user-list')
    <ul>
        @foreach ($users as $user)
            <li>{{ $user->name }}</li>
        @endforeach
    </ul>
@endfragment
```

Następnie, podczas renderowania widoku, który wykorzystuje ten szablon, możesz wywołać metodę `fragment`, aby określić, że tylko określony fragment powinien być uwzględniony w wychodzącej odpowiedzi HTTP:

```php
return view('dashboard', ['users' => $users])->fragment('user-list');
```

Metoda `fragmentIf` pozwala warunkowo zwrócić fragment widoku na podstawie danego warunku. W przeciwnym razie zostanie zwrócony cały widok:

```php
return view('dashboard', ['users' => $users])
    ->fragmentIf($request->hasHeader('HX-Request'), 'user-list');
```

Metody `fragments` i `fragmentsIf` pozwalają zwrócić wiele fragmentów widoku w odpowiedzi. Fragmenty zostaną połączone razem:

```php
view('dashboard', ['users' => $users])
    ->fragments(['user-list', 'comment-list']);

view('dashboard', ['users' => $users])
    ->fragmentsIf(
        $request->hasHeader('HX-Request'),
        ['user-list', 'comment-list']
    );
```

<a name="extending-blade"></a>
## Rozszerzanie Blade

Blade pozwala definiować własne niestandardowe dyrektywy za pomocą metody `directive`. Gdy kompilator Blade napotka niestandardową dyrektywę, wywoła dostarczone wywołanie zwrotne (callback) z wyrażeniem, które zawiera dyrektywa.

Poniższy przykład tworzy dyrektywę `@datetime($var)`, która formatuje daną zmienną `$var`, która powinna być instancją `DateTime`:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Blade::directive('datetime', function (string $expression) {
            return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
        });
    }
}
```

Jak widać, połączymy metodę `format` z jakimkolwiek wyrażeniem przekazanym do dyrektywy. Tak więc w tym przykładzie końcowy kod PHP wygenerowany przez tę dyrektywę będzie wyglądać następująco:

```php
<?php echo ($var)->format('m/d/Y H:i'); ?>
```

> [!WARNING]
> Po zaktualizowaniu logiki dyrektywy Blade będziesz musiał usunąć wszystkie buforowane widoki Blade. Buforowane widoki Blade można usunąć za pomocą polecenia Artisan `view:clear`.

<a name="custom-echo-handlers"></a>
### Niestandardowe handlery echo

Jeśli spróbujesz "wyprowadzić" obiekt za pomocą Blade, zostanie wywołana metoda `__toString` obiektu. Metoda [__toString](https://www.php.net/manual/en/language.oop5.magic.php#object.tostring) jest jedną z wbudowanych "magicznych metod" PHP. Jednak czasami możesz nie mieć kontroli nad metodą `__toString` danej klasy, na przykład gdy klasa, z którą wchodzisz w interakcję, należy do biblioteki innej firmy.

W takich przypadkach Blade pozwala zarejestrować niestandardowy handler echo dla tego konkretnego typu obiektu. Aby to osiągnąć, powinieneś wywołać metodę `stringable` Blade. Metoda `stringable` przyjmuje domknięcie. To domknięcie powinno wskazywać typ obiektu, za renderowanie którego jest odpowiedzialne. Zazwyczaj metoda `stringable` powinna być wywoływana w metodzie `boot` klasy `AppServiceProvider` Twojej aplikacji:

```php
use Illuminate\Support\Facades\Blade;
use Money\Money;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Blade::stringable(function (Money $money) {
        return $money->formatTo('en_GB');
    });
}
```

Po zdefiniowaniu niestandardowego handlera echo możesz po prostu wyprowadzić obiekt w swoim szablonie Blade:

```blade
Cost: {{ $money }}
```

<a name="custom-if-statements"></a>
### Niestandardowe instrukcje warunkowe

Programowanie niestandardowej dyrektywy jest czasami bardziej złożone niż to konieczne podczas definiowania prostych, niestandardowych instrukcji warunkowych. Z tego powodu Blade zapewnia metodę `Blade::if`, która pozwala szybko definiować niestandardowe dyrektywy warunkowe za pomocą domknięć. Na przykład zdefiniujmy niestandardowy warunek, który sprawdza skonfigurowany domyślny "dysk" dla aplikacji. Możemy to zrobić w metodzie `boot` naszego `AppServiceProvider`:

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Blade::if('disk', function (string $value) {
        return config('filesystems.default') === $value;
    });
}
```

Po zdefiniowaniu niestandardowego warunku możesz go używać w swoich szablonach:

```blade
@disk('local')
    <!-- The application is using the local disk... -->
@elsedisk('s3')
    <!-- The application is using the s3 disk... -->
@else
    <!-- The application is using some other disk... -->
@enddisk

@unlessdisk('local')
    <!-- The application is not using the local disk... -->
@enddisk
```
