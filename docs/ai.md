# Rozwój wspomagany AI

- [Wprowadzenie](#introduction)
    - [Dlaczego Laravel do rozwoju z AI?](#why-laravel-for-ai-development)
- [Laravel Boost](#laravel-boost)
    - [Instalacja](#installation)
    - [Dostępne narzędzia](#available-tools)
    - [Wytyczne AI](#ai-guidelines)
    - [Wyszukiwanie dokumentacji](#documentation-search)
    - [Integracja z IDE](#ide-integration)
- [Niestandardowe wytyczne Boost](#custom-guidelines)
    - [Dodawanie wytycznych projektu](#adding-project-guidelines)
    - [Wytyczne pakietów](#package-guidelines)

<a name="introduction"></a>
## Wprowadzenie

Laravel jest wyjątkowo dobrze przygotowany do bycia najlepszym frameworkiem dla rozwoju wspomaganego przez AI i agentów. Rozwój agentów kodujących AI, takich jak [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [OpenCode](https://opencode.ai), [Cursor](https://cursor.com) i [GitHub Copilot](https://github.com/features/copilot), zmienił sposób, w jaki programiści piszą kod. Te narzędzia mogą generować całe funkcjonalności, debugować złożone problemy i refaktoryzować kod z bezprecedensową szybkością - ale ich efektywność w dużej mierze zależy od tego, jak dobrze rozumieją Twoją bazę kodu.

<a name="why-laravel-for-ai-development"></a>
### Dlaczego Laravel do rozwoju z AI?

Opiniowane konwencje Laravel i dobrze zdefiniowana struktura sprawiają, że jest to idealny framework do rozwoju wspomaganego przez AI. Gdy poprosisz agenta AI o dodanie kontrolera, wie dokładnie, gdzie go umieścić. Gdy potrzebujesz nowej migracji, konwencje nazewnictwa i lokalizacje plików są przewidywalne. Ta spójność eliminuje zgadywanie, które często sprawia kłopoty narzędziom AI w bardziej elastycznych frameworkach.

Poza organizacją plików, ekspresywna składnia Laravel i kompleksowa dokumentacja dają agentom AI kontekst potrzebny do generowania dokładnego, idiomatycznego kodu. Funkcje takie jak relacje Eloquent, żądania formularzy i middleware podążają za wzorcami, które agenci mogą niezawodnie zrozumieć i replikować. Rezultatem jest kod generowany przez AI, który wygląda tak, jakby został napisany przez doświadczonego programistę Laravel, a nie poskładany z generycznych fragmentów PHP.

<a name="laravel-boost"></a>
## Laravel Boost

[Laravel Boost](https://github.com/laravel/boost) wypełnia lukę między agentami kodującymi AI a Twoją aplikacją Laravel. Boost to serwer MCP (Model Context Protocol) wyposażony w ponad 15 wyspecjalizowanych narzędzi, które zapewniają agentom AI głęboki wgląd w strukturę aplikacji, bazę danych, trasy i wiele więcej. Po zainstalowaniu Boost, Twój agent AI przekształca się z ogólnego asystenta kodu w eksperta Laravel, który rozumie Twoją konkretną aplikację.

Boost zapewnia trzy główne możliwości: zestaw narzędzi MCP do inspekcji i interakcji z Twoją aplikacją, komponowalne wytyczne AI stworzone specjalnie dla ekosystemu Laravel oraz potężne API dokumentacji zawierające ponad 17 000 fragmentów wiedzy specyficznej dla Laravel.

<a name="installation"></a>
### Instalacja

Boost może być zainstalowany w aplikacjach Laravel 10, 11 i 12 działających na PHP 8.1 lub nowszym. Aby rozpocząć, zainstaluj Boost jako zależność deweloperską:

```shell
composer require laravel/boost --dev
```

Po zainstalowaniu uruchom interaktywny instalator:

```shell
php artisan boost:install
```

Instalator automatycznie wykryje Twoje IDE i agentów AI, umożliwiając wybór integracji odpowiednich dla Twojego projektu. Boost wygeneruje niezbędne pliki konfiguracyjne, takie jak `.mcp.json` dla edytorów zgodnych z MCP oraz pliki wytycznych dla kontekstu AI.

> [!NOTE]
> Wygenerowane pliki konfiguracyjne, takie jak `.mcp.json`, `CLAUDE.md` i `boost.json`, mogą być bezpiecznie dodane do `.gitignore`, jeśli wolisz, aby każdy programista konfigurował własne środowisko.

<a name="available-tools"></a>
### Dostępne narzędzia

Boost udostępnia kompleksowy zestaw narzędzi agentom AI za pośrednictwem Model Context Protocol. Te narzędzia umożliwiają agentom głębokie zrozumienie i interakcję z Twoją aplikacją Laravel:

<div class="content-list" markdown="1">

- **Introspekcja aplikacji** - Sprawdzaj wersje PHP i Laravel, listuj zainstalowane pakiety i inspekcjonuj konfigurację aplikacji oraz zmienne środowiskowe.
- **Narzędzia bazy danych** - Inspekcjonuj schemat bazy danych, wykonuj zapytania tylko do odczytu i zrozum strukturę danych bez opuszczania konwersacji.
- **Inspekcja tras** - Wyświetlaj wszystkie zarejestrowane trasy z ich middleware, kontrolerami i parametrami.
- **Polecenia Artisan** - Odkrywaj dostępne polecenia Artisan i ich argumenty, umożliwiając agentom sugerowanie i wykonywanie właściwych poleceń dla Twojego zadania.
- **Analiza logów** - Czytaj i analizuj pliki logów aplikacji, aby pomóc w debugowaniu problemów.
- **Logi przeglądarki** - Uzyskuj dostęp do logów konsoli przeglądarki i błędów podczas rozwijania z narzędziami frontendowymi Laravel.
- **Integracja Tinker** - Wykonuj kod PHP w kontekście Twojej aplikacji za pomocą Laravel Tinker, umożliwiając agentom testowanie hipotez i weryfikację zachowania.
- **Wyszukiwanie dokumentacji** - Przeszukuj dokumentację ekosystemu Laravel z wynikami dostosowanymi do Twoich zainstalowanych wersji pakietów.

</div>

<a name="ai-guidelines"></a>
### Wytyczne AI

Boost zawiera kompleksowy zestaw wytycznych AI specjalnie przygotowanych dla ekosystemu Laravel. Te wytyczne uczą agentów AI, jak pisać idiomatyczny kod Laravel, przestrzegać konwencji frameworka i unikać typowych pułapek. Wytyczne są komponowalne i świadome wersji, co oznacza, że agenci otrzymują instrukcje odpowiednie dla Twoich dokładnych wersji pakietów.

Wytyczne są dostępne dla samego Laravel i ponad 16 pakietów w ekosystemie Laravel, w tym:

<div class="content-list" markdown="1">

- Livewire (2.x, 3.x i 4.x)
- Inertia.js (warianty React i Vue)
- Tailwind CSS (3.x i 4.x)
- Filament (3.x i 4.x)
- PHPUnit
- Pest PHP
- Laravel Pint
- I wiele więcej

</div>

Gdy uruchomisz `boost:install`, Boost automatycznie wykrywa, które pakiety używa Twoja aplikacja i składa odpowiednie wytyczne w plikach kontekstu AI Twojego projektu.

<a name="documentation-search"></a>
### Wyszukiwanie dokumentacji

Boost zawiera potężne API dokumentacji, które daje agentom AI dostęp do ponad 17 000 fragmentów dokumentacji ekosystemu Laravel. W przeciwieństwie do ogólnych wyszukiwań internetowych, ta dokumentacja jest indeksowana, wektoryzowana i filtrowana tak, aby pasowała do Twoich dokładnych wersji pakietów.

Gdy agent potrzebuje zrozumieć, jak działa funkcjonalność, może przeszukać API dokumentacji Boost i otrzymać dokładne, specyficzne dla wersji informacje. To eliminuje powszechny problem agentów AI sugerujących przestarzałe metody lub składnię ze starszych wersji frameworka.

<a name="ide-integration"></a>
### Integracja z IDE

Boost integruje się z popularnymi IDE i narzędziami AI, które obsługują Model Context Protocol. Oto jak włączyć Boost w kilku popularnych edytorach:

```text tab="Claude Code"
// torchlight! {"lineNumbers": false}
Boost jest zazwyczaj wykrywany automatycznie. Jeśli potrzebna jest ręczna konfiguracja:

1. Otwórz terminal w katalogu swojego projektu
2. Uruchom: claude mcp add laravel-boost -- php artisan boost:mcp
```

```text tab=Cursor
// torchlight! {"lineNumbers": false}
1. Otwórz paletę poleceń (Cmd+Shift+P lub Ctrl+Shift+P)
2. Wybierz "MCP: Open Settings"
3. Włącz opcję "laravel-boost"
```

```text tab="VS Code"
// torchlight! {"lineNumbers": false}
1. Otwórz paletę poleceń (Cmd+Shift+P lub Ctrl+Shift+P)
2. Wybierz "MCP: List Servers"
3. Przejdź do "laravel-boost" i naciśnij enter
4. Wybierz "Start server"
```

```text tab=PhpStorm
// torchlight! {"lineNumbers": false}
1. Naciśnij Shift dwukrotnie, aby otworzyć Search Everywhere
2. Wyszukaj "MCP Settings" i naciśnij enter
3. Włącz checkbox obok "laravel-boost"
4. Kliknij "Apply"
```

```text tab=Codex
// torchlight! {"lineNumbers": false}
Boost jest zazwyczaj wykrywany automatycznie. Jeśli potrzebna jest ręczna konfiguracja:

1. Otwórz terminal w katalogu swojego projektu
2. Uruchom: codex mcp add -- php artisan boost:mcp
```

```text tab=Gemini
// torchlight! {"lineNumbers": false}
Boost jest zazwyczaj wykrywany automatycznie. Jeśli potrzebna jest ręczna konfiguracja:

1. Otwórz terminal w katalogu swojego projektu
2. Uruchom: gemini mcp add laravel-boost -- php artisan boost:mcp
```

<a name="custom-guidelines"></a>
## Niestandardowe wytyczne Boost

Chociaż wbudowane wytyczne Boost kompleksowo obejmują ekosystem Laravel, możesz chcieć dodać instrukcje specyficzne dla projektu dla swoich agentów AI.

<a name="adding-project-guidelines"></a>
### Dodawanie wytycznych projektu

Aby dodać niestandardowe wytyczne dla swojego projektu, utwórz pliki `.blade.php` lub `.md` w katalogu `.ai/guidelines` Twojej aplikacji:

```text
.ai/
└── guidelines/
    └── api-conventions.md
    ├── architecture.md
    ├── testing-standards.blade.php
```

Te pliki będą automatycznie uwzględnione, gdy uruchomisz `boost:install`. Użyj tych wytycznych do udokumentowania standardów kodowania Twojego zespołu, decyzji architektonicznych, terminologii specyficznej dla domeny lub jakiegokolwiek innego kontekstu, który pomógłby agentom AI pisać lepszy kod dla Twojego projektu.

<a name="package-guidelines"></a>
### Wytyczne pakietów

Jeśli utrzymujesz pakiet Laravel i chcesz dostarczyć wytyczne AI dla swoich użytkowników, możesz umieścić wytyczne w katalogu `resources/boost/guidelines` swojego pakietu:

```text
resources/
└── boost/
    └── guidelines/
        └── core.blade.php
```

Wytyczne AI powinny zawierać krótki przegląd tego, co robi Twój pakiet, opisywać wymaganą strukturę plików lub konwencje i wyjaśniać, jak tworzyć lub używać jego głównych funkcjonalności (z przykładowymi poleceniami lub fragmentami kodu). Utrzymuj je zwięzłe, wykonalne i skupione na najlepszych praktykach, aby AI mógł generować poprawny kod dla Twoich użytkowników. Oto przykład:

```md
## Nazwa pakietu

Ten pakiet zapewnia [krótki opis funkcjonalności].

### Funkcjonalności

- Funkcjonalność 1: [jasny i krótki opis].
- Funkcjonalność 2: [jasny i krótki opis]. Przykład użycia:

@verbatim
<code-snippet name="Jak używać Funkcjonalności 2" lang="php">
$result = PackageName::featureTwo($param1, $param2);
</code-snippet>
@endverbatim
```

Gdy użytkownicy zainstalują Boost w aplikacji zawierającej Twój pakiet, Twoje wytyczne zostaną automatycznie odkryte i uwzględnione w ich kontekście AI. To pozwala autorom pakietów pomóc agentom AI zrozumieć, jak prawidłowo używać ich pakietów.
