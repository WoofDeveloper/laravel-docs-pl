# Frontend

- [Wprowadzenie](#introduction)
- [Używanie PHP](#using-php)
    - [PHP i Blade](#php-and-blade)
    - [Livewire](#livewire)
    - [Zestawy startowe](#php-starter-kits)
- [Używanie React lub Vue](#using-react-or-vue)
    - [Inertia](#inertia)
    - [Zestawy startowe](#inertia-starter-kits)
- [Bundlowanie zasobów](#bundling-assets)

<a name="introduction"></a>
## Wprowadzenie

Laravel to framework backendowy, który zapewnia wszystkie funkcje potrzebne do budowania nowoczesnych aplikacji webowych, takie jak [routing](/docs/{{version}}/routing), [walidacja](/docs/{{version}}/validation), [cachowanie](/docs/{{version}}/cache), [kolejki](/docs/{{version}}/queues), [przechowywanie plików](/docs/{{version}}/filesystem) i wiele innych. Wierzymy jednak, że ważne jest, aby oferować deweloperom piękne doświadczenie full-stack, w tym potężne podejścia do budowania frontendu aplikacji.

Istnieją dwa główne sposoby podejścia do rozwoju frontendu podczas budowania aplikacji z Laravel, a wybór podejścia zależy od tego, czy chcesz zbudować swój frontend wykorzystując PHP, czy używając frameworków JavaScript, takich jak Vue i React. Omówimy obie te opcje poniżej, abyś mógł podjąć świadomą decyzję dotyczącą najlepszego podejścia do rozwoju frontendu dla swojej aplikacji.

<a name="using-php"></a>
## Używanie PHP

<a name="php-and-blade"></a>
### PHP i Blade

W przeszłości większość aplikacji PHP renderowała HTML do przeglądarki za pomocą prostych szablonów HTML przeplatanych instrukcjami PHP `echo`, które renderują dane pobrane z bazy danych podczas żądania:

```blade
<div>
    <?php foreach ($users as $user): ?>
        Hello, <?php echo $user->name; ?> <br />
    <?php endforeach; ?>
</div>
```

W Laravel to podejście do renderowania HTML może być nadal osiągnięte za pomocą [widoków](/docs/{{version}}/views) i [Blade](/docs/{{version}}/blade). Blade to niezwykle lekki język szablonów, który zapewnia wygodne, krótkie składnie do wyświetlania danych, iterowania po danych i więcej:

```blade
<div>
    @foreach ($users as $user)
        Hello, {{ $user->name }} <br />
    @endforeach
</div>
```

Podczas budowania aplikacji w ten sposób, wysyłanie formularzy i inne interakcje na stronie zazwyczaj otrzymują całkowicie nowy dokument HTML z serwera, a cała strona jest ponownie renderowana przez przeglądarkę. Nawet dzisiaj wiele aplikacji może być idealnie dopasowanych do posiadania frontendów skonstruowanych w ten sposób za pomocą prostych szablonów Blade.

<a name="growing-expectations"></a>
#### Rosnące oczekiwania

Jednak wraz z dojrzewaniem oczekiwań użytkowników dotyczących aplikacji webowych, wielu programistów znalazło potrzebę budowania bardziej dynamicznych frontendów z interakcjami, które wydają się bardziej wypolerowane. W związku z tym niektórzy deweloperzy decydują się rozpocząć budowanie frontendu swojej aplikacji za pomocą frameworków JavaScript, takich jak Vue i React.

Inni, preferujący trzymanie się języka backendowego, z którym czują się komfortowo, rozwinęli rozwiązania, które pozwalają na budowę nowoczesnych interfejsów użytkownika aplikacji webowych, przy nadal głównym wykorzystaniu ich języka backendowego. Na przykład w ekosystemie [Rails](https://rubyonrails.org/) doprowadziło to do powstania bibliotek takich jak [Turbo](https://turbo.hotwired.dev/) [Hotwire](https://hotwired.dev/) i [Stimulus](https://stimulus.hotwired.dev/).

W ekosystemie Laravel potrzeba tworzenia nowoczesnych, dynamicznych frontendów głównie przy użyciu PHP doprowadziła do powstania [Laravel Livewire](https://livewire.laravel.com) i [Alpine.js](https://alpinejs.dev/).

<a name="livewire"></a>
### Livewire

[Laravel Livewire](https://livewire.laravel.com) to framework do budowania frontendów zasilanych przez Laravel, które czują się dynamiczne, nowoczesne i żywe, tak jak frontendy zbudowane za pomocą nowoczesnych frameworków JavaScript, takich jak Vue i React.

Podczas używania Livewire będziesz tworzyć "komponenty" Livewire, które renderują dyskretny fragment Twojego interfejsu użytkownika i udostępniają metody i dane, które mogą być wywoływane i współdziałać z frontendem Twojej aplikacji. Na przykład prosty komponent "Counter" może wyglądać następująco:

```php
<?php

namespace App\Http\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public $count = 0;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

A odpowiadający szablon dla licznika byłby napisany tak:

```blade
<div>
    <button wire:click="increment">+</button>
    <h1>{{ $count }}</h1>
</div>
```

Jak widać, Livewire umożliwia pisanie nowych atrybutów HTML, takich jak `wire:click`, które łączą frontend i backend Twojej aplikacji Laravel. Dodatkowo możesz renderować bieżący stan swojego komponentu za pomocą prostych wyrażeń Blade.

Dla wielu Livewire zrewolucjonizował rozwiój frontendu z Laravel, pozwalając im pozostać w komforcie Laravel podczas konstruowania nowoczesnych, dynamicznych aplikacji webowych. Zazwyczaj deweloperzy używający Livewire będą również wykorzystywać [Alpine.js](https://alpinejs.dev/) do "posypywania" JavaScript na swoim frontendzie tylko tam, gdzie jest to potrzebne, na przykład w celu renderowania okna dialogowego.

Jeśli jesteś nowy w Laravel, zalecamy zapoznanie się z podstawowym użyciem [widoków](/docs/{{version}}/views) i [Blade](/docs/{{version}}/blade). Następnie zapoznaj się z oficjalną [dokumentacją Laravel Livewire](https://livewire.laravel.com/docs), aby dowiedzieć się, jak przenieść swoją aplikację na następny poziom za pomocą interaktywnych komponentów Livewire.

<a name="php-starter-kits"></a>
### Zestawy startowe

Jeśli chcesz zbudować swój frontend za pomocą PHP i Livewire, możesz skorzystać z naszego [zestawu startowego Livewire](/docs/{{version}}/starter-kits), aby szybko rozpocząć rozwój swojej aplikacji.

<a name="using-react-or-vue"></a>
## Używanie React lub Vue

Chociaż możliwe jest budowanie nowoczesnych frontendów za pomocą Laravel i Livewire, wielu programistów nadal preferuje wykorzystanie mocy frameworka JavaScript, takiego jak React lub Vue. Pozwala to deweloperom na wykorzystanie bogatego ekosystemu pakietów i narzędzi JavaScript dostępnych przez NPM.

Jednak bez dodatkowych narzędzi połączenie Laravel z React lub Vue pozostawiłoby nas z koniecznością rozwiązania różnych skomplikowanych problemów, takich jak routing po stronie klienta, hydratacja danych i uwierzytelnianie. Routing po stronie klienta jest często uproszczony przez użycie opiniotworzych frameworków React / Vue, takich jak [Next](https://nextjs.org/) i [Nuxt](https://nuxt.com/); jednak hydratacja danych i uwierzytelnianie pozostają skomplikowanymi i uciążliwymi problemami do rozwiązania podczas łączenia frameworka backendowego, takiego jak Laravel, z tymi frameworkami frontendowymi.

Ponadto deweloperzy muszą utrzymywać dwa oddzielne repozytoria kodu, często wymagając koordynacji konserwacji, wydania i wdrożeń w obu repozytoriach. Chociaż te problemy nie są nie do pokonania, nie wierzymy, że jest to produktywny lub przyjemny sposób rozwijania aplikacji.

<a name="inertia"></a>
### Inertia

Na szczęście Laravel oferuje to, co najlepsze z obu światów. [Inertia](https://inertiajs.com) wypełnia lukę między Twoją aplikacją Laravel a nowoczesnym frontendem React lub Vue, pozwalając budować pełnoprawne, nowoczesne frontendy za pomocą React lub Vue, wykorzystując jednocześnie trasy i kontrolery Laravel do routingu, hydratacji danych i uwierzytelniania — wszystko w ramach jednego repozytorium kodu. Przy tym podejściu możesz cieszyć się pełną mocą zarówno Laravel, jak i React / Vue, bez og przemożliwych możliwości żadnego z narzędzi.

Po zainstalowaniu Inertia w swojej aplikacji Laravel, będziesz pisać trasy i kontrolery normalnie. Jednak zamiast zwracania szablonu Blade z kontrolera, zwrócisz stronę Inertia:

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Inertia\Inertia;
use Inertia\Response;

class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     */
    public function show(string $id): Response
    {
        return Inertia::render('users/show', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

Strona Inertia odpowiada komponentowi React lub Vue, zazwyczaj przechowywanym w katalogu `resources/js/pages` Twojej aplikacji. Dane przekazane do strony za pomocą metody `Inertia::render` będą użyte do hydratacji "props" komponentu strony:

```jsx
import Layout from '@/layouts/authenticated';
import { Head } from '@inertiajs/react';

export default function Show({ user }) {
    return (
        <Layout>
            <Head title="Welcome" />
            <h1>Welcome</h1>
            <p>Hello {user.name}, welcome to Inertia.</p>
        </Layout>
    )
}
```

Jak widać, Inertia pozwala wykorzystać pełną moc React lub Vue podczas budowania frontendu, zapewniając jednocześnie lekki most między Twoim backendem zasilanym przez Laravel a frontendem zasilanym przez JavaScript.

#### Renderowanie po stronie serwera

Jeśli martwisz się o zanurzenie się w Inertia, ponieważ Twoja aplikacja wymaga renderowania po stronie serwera, nie martw się. Inertia oferuje [wsparcie dla renderowania po stronie serwera](https://inertiajs.com/server-side-rendering). A podczas wdrażania aplikacji przez [Laravel Cloud](https://cloud.laravel.com) lub [Laravel Forge](https://forge.laravel.com), to pestka upewnić się, że proces renderowania po stronie serwera Inertia zawsze działa.

<a name="inertia-starter-kits"></a>
### Zestawy startowe

Jeśli chcesz zbudować swój frontend za pomocą Inertia i Vue / React, możesz skorzystać z naszych [zestawów startowych aplikacji React lub Vue](/docs/{{version}}/starter-kits), aby szybko rozpocząć rozwój swojej aplikacji. Oba te zestawy startowe zapewniają szkielet dla backendu i frontendu Twojej aplikacji, w tym przepływ uwierzytelniania, za pomocą Inertia, Vue / React, [Tailwind](https://tailwindcss.com) i [Vite](https://vitejs.dev), abyś mógł rozpocząć budowanie swojego następnego wielkiego pomysłu.

<a name="bundling-assets"></a>
## Bundlowanie zasobów

Niezależnie od tego, czy zdecydujesz się rozwijać swój frontend za pomocą Blade i Livewire, czy Vue / React i Inertia, prawdopodobnie będziesz musiał zbundlować CSS swojej aplikacji do zasobów gotowych do produkcji. Oczywiście, jeśli zdecydujesz się zbudować frontend swojej aplikacji za pomocą Vue lub React, będziesz również musiał zbundlować swoje komponenty do zasobów JavaScript gotowych dla przeglądarki.

Domyślnie Laravel wykorzystuje [Vite](https://vitejs.dev) do bundlowania zasobów. Vite zapewnia błyskawiczne czasy budowania i niemal natychmiastowe Hot Module Replacement (HMR) podczas lokalnego rozwoju. We wszystkich nowych aplikacjach Laravel, w tym tych używających naszych [zestawów startowych](/docs/{{version}}/starter-kits), znajdziesz plik `vite.config.js`, który ładuje nasz lekki plugin Laravel Vite, który sprawia, że używanie Vite z aplikacjami Laravel jest przyjemnością.

Najszybszym sposobem na rozpoczęcie pracy z Laravel i Vite jest rozpoczęcie rozwoju aplikacji za pomocą [naszych zestawów startowych aplikacji](/docs/{{version}}/starter-kits), które szybko rozpoczynają Twoją aplikację, zapewniając szkielet uwierzytelniania frontendowego i backendowego.

> [!NOTE]
> Aby uzyskać bardziej szczegółową dokumentację dotyczącą wykorzystania Vite z Laravel, zapoznaj się z naszą [dedykowaną dokumentacją dotyczącą bundlowania i kompilowania zasobów](/docs/{{version}}/vite).
