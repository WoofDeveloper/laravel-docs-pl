# Laravel Boost

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
    - [Aktualizowanie zasobów Boost](#keeping-boost-resources-updated)
    - [Konfiguracja agentów](#set-up-your-agents)
- [Serwer MCP](#mcp-server)
    - [Dostępne narzędzia MCP](#available-mcp-tools)
    - [Ręczna rejestracja serwera MCP](#manually-registering-the-mcp-server)
- [Wytyczne AI](#ai-guidelines)
    - [Dostępne wytyczne AI](#available-ai-guidelines)
    - [Dodawanie własnych wytycznych AI](#adding-custom-ai-guidelines)
    - [Nadpisywanie wytycznych Boost AI](#overriding-boost-ai-guidelines)
    - [Wytyczne AI pakietów zewnętrznych](#third-party-package-ai-guidelines)
- [Umiejętności agentów](#agent-skills)
    - [Dostępne umiejętności](#available-skills)
    - [Własne umiejętności](#custom-skills)
    - [Nadpisywanie umiejętności](#overriding-skills)
    - [Umiejętności pakietów zewnętrznych](#third-party-package-skills)
- [Wytyczne vs. Umiejętności](#guidelines-vs-skills)
- [API dokumentacji](#documentation-api)
- [Rozszerzanie Boost](#extending-boost)
    - [Dodawanie wsparcia dla innych IDE / Agentów AI](#adding-support-for-other-ides-ai-agents)

<a name="introduction"></a>
## Wprowadzenie

Laravel Boost przyspiesza rozwój wspomagany przez AI, dostarczając niezbędne wytyczne i umiejętności agentów, które pomagają agentom AI tworzyć wysokiej jakości aplikacje Laravel zgodne z najlepszymi praktykami Laravel.

Boost zapewnia również potężne API dokumentacji ekosystemu Laravel, które łączy wbudowane narzędzie MCP z obszerną bazą wiedzy zawierającą ponad 17 000 informacji specyficznych dla Laravel, wszystkie wzbogacone możliwościami wyszukiwania semantycznego przy użyciu embeddingów dla precyzyjnych, kontekstowych wyników. Boost instruuje agentów AI, takich jak Claude Code i Cursor, aby korzystali z tego API w celu poznania najnowszych funkcji Laravel i najlepszych praktyk.

<a name="installation"></a>
## Instalacja

Laravel Boost można zainstalować za pomocą Composera:

```shell
composer require laravel/boost --dev
```

Następnie zainstaluj serwer MCP i wytyczne kodowania:

```shell
php artisan boost:install
```

Polecenie `boost:install` wygeneruje odpowiednie pliki wytycznych i umiejętności agentów dla agentów kodowania wybranych podczas procesu instalacji.

Po zainstalowaniu Laravel Boost jesteś gotowy do rozpoczęcia kodowania z Cursor, Claude Code lub wybranym agentem AI.

> [!NOTE]
> Możesz dodać wygenerowany plik konfiguracyjny MCP (`.mcp.json`), pliki wytycznych (`CLAUDE.md`, `AGENTS.md`, `junie/`, itp.) oraz plik konfiguracyjny `boost.json` do `.gitignore` swojej aplikacji, ponieważ pliki te są automatycznie regenerowane podczas uruchamiania `boost:install` i `boost:update`.

<a name="set-up-your-agents"></a>
### Konfiguracja agentów

#### Cursor

1. Otwórz paletę poleceń (`Cmd+Shift+P` lub `Ctrl+Shift+P`)
2. Naciśnij `enter` na "/open MCP Settings"
3. Włącz przełącznik dla `laravel-boost`

#### Claude Code

Wsparcie dla Claude Code jest zazwyczaj włączane automatycznie. Jeśli okaże się, że nie jest, otwórz powłokę w katalogu projektu i uruchom następujące polecenie:

```shell
claude mcp add -s local -t stdio laravel-boost php artisan boost:mcp
```

#### Codex

Wsparcie dla Codex jest zazwyczaj włączane automatycznie. Jeśli okaże się, że nie jest, otwórz powłokę w katalogu projektu i uruchom następujące polecenie:

```shell
codex mcp add laravel-boost -- php "artisan" "boost:mcp"
```

#### Gemini CLI

Wsparcie dla Gemini CLI jest zazwyczaj włączane automatycznie. Jeśli okaże się, że nie jest, otwórz powłokę w katalogu projektu i uruchom następujące polecenie:

```shell
gemini mcp add -s project -t stdio laravel-boost php artisan boost:mcp
```

#### GitHub Copilot (VS Code)

1. Otwórz paletę poleceń (`Cmd+Shift+P` lub `Ctrl+Shift+P`)
2. Naciśnij `enter` na "MCP: List Servers"
3. Przejdź do `laravel-boost` i naciśnij `enter`
4. Wybierz "Start server"

#### Junie

1. Naciśnij `shift` dwukrotnie, aby otworzyć paletę poleceń
2. Wyszukaj "MCP Settings" i naciśnij `enter`
3. Zaznacz pole wyboru obok `laravel-boost`
4. Kliknij "Apply" w prawym dolnym rogu

<a name="keeping-boost-resources-updated"></a>
### Aktualizowanie zasobów Boost

Możesz chcieć okresowo aktualizować lokalne zasoby Boost (wytyczne AI i umiejętności), aby upewnić się, że odzwierciedlają najnowsze wersje zainstalowanych pakietów ekosystemu Laravel. Aby to zrobić, możesz użyć polecenia Artisan `boost:update`.

```shell
php artisan boost:update
```

Możesz również zautomatyzować ten proces, dodając go do skryptów Composera "post-update-cmd":

```json
{
  "scripts": {
    "post-update-cmd": [
      "@php artisan boost:update --ansi"
    ]
  }
}
```

<a name="mcp-server"></a>
## Serwer MCP

Laravel Boost zapewnia serwer MCP (Model Context Protocol), który udostępnia narzędzia dla agentów AI do interakcji z aplikacją Laravel. Te narzędzia dają agentom możliwość inspekcji struktury aplikacji, odpytywania bazy danych, wykonywania kodu i nie tylko.

<a name="available-mcp-tools"></a>
### Dostępne narzędzia MCP

| Nazwa                      | Opis                                                                                                           |
|----------------------------|----------------------------------------------------------------------------------------------------------------|
| Application Info           | Odczytuje wersje PHP i Laravel, silnik bazy danych, listę pakietów ekosystemu z wersjami i modele Eloquent    |
| Browser Logs               | Odczytuje logi i błędy z przeglądarki                                                                          |
| Database Connections       | Sprawdza dostępne połączenia z bazą danych, w tym domyślne połączenie                                          |
| Database Query             | Wykonuje zapytanie do bazy danych                                                                              |
| Database Schema            | Odczytuje schemat bazy danych                                                                                  |
| Get Absolute URL           | Konwertuje względne ścieżki URI na bezwzględne, aby agenci generowali prawidłowe URL                           |
| Get Config                 | Pobiera wartość z plików konfiguracyjnych używając notacji "kropkowej"                                         |
| Last Error                 | Odczytuje ostatni błąd z plików logów aplikacji                                                                |
| List Artisan Commands      | Sprawdza dostępne polecenia Artisan                                                                            |
| List Available Config Keys | Sprawdza dostępne klucze konfiguracyjne                                                                        |
| List Available Env Vars    | Sprawdza dostępne klucze zmiennych środowiskowych                                                              |
| List Routes                | Sprawdza trasy aplikacji                                                                                       |
| Read Log Entries           | Odczytuje ostatnie N wpisów logów                                                                              |
| Search Docs                | Odpytuje usługę API dokumentacji Laravel w celu pobrania dokumentacji na podstawie zainstalowanych pakietów    |
| Tinker                     | Wykonuje dowolny kod w kontekście aplikacji                                                                    |

<a name="manually-registering-the-mcp-server"></a>
### Ręczna rejestracja serwera MCP

Czasami może być konieczna ręczna rejestracja serwera MCP Laravel Boost w wybranym edytorze. Powinieneś zarejestrować serwer MCP używając następujących szczegółów:

<table>
<tr><td><strong>Command</strong></td><td><code>php</code></td></tr>
<tr><td><strong>Args</strong></td><td><code>artisan boost:mcp</code></td></tr>
</table>

Przykład JSON:

```json
{
    "mcpServers": {
        "laravel-boost": {
            "command": "php",
            "args": ["artisan", "boost:mcp"]
        }
    }
}
```

<a name="ai-guidelines"></a>
## Wytyczne AI

Wytyczne AI to komponowalne pliki instrukcji, które są ładowane z góry, aby zapewnić agentom AI niezbędny kontekst dotyczący pakietów ekosystemu Laravel. Te wytyczne zawierają podstawowe konwencje, najlepsze praktyki i wzorce specyficzne dla frameworka, które pomagają agentom generować spójny, wysokiej jakości kod.

<a name="available-ai-guidelines"></a>
### Dostępne wytyczne AI

Laravel Boost zawiera wytyczne AI dla następujących pakietów i frameworków. Wytyczne `core` zapewniają ogólne, uogólnione porady dla AI dotyczące danego pakietu, które mają zastosowanie we wszystkich wersjach.

| Pakiet             | Wersje obsługiwane     |
|--------------------|------------------------|
| Core & Boost       | core                   |
| Laravel Framework  | core, 10.x, 11.x, 12.x |
| Livewire           | core, 2.x, 3.x, 4.x    |
| Flux UI            | core, free, pro        |
| Folio              | core                   |
| Herd               | core                   |
| Inertia Laravel    | core, 1.x, 2.x         |
| Inertia React      | core, 1.x, 2.x         |
| Inertia Vue        | core, 1.x, 2.x         |
| Inertia Svelte     | core, 1.x, 2.x         |
| MCP                | core                   |
| Pennant            | core                   |
| Pest               | core, 3.x, 4.x         |
| PHPUnit            | core                   |
| Pint               | core                   |
| Sail               | core                   |
| Tailwind CSS       | core, 3.x, 4.x         |
| Livewire Volt      | core                   |
| Wayfinder          | core                   |
| Enforce Tests      | conditional            |

> **Uwaga:** Aby zaktualizować swoje wytyczne AI, zobacz sekcję [Aktualizowanie zasobów Boost](#keeping-boost-resources-updated).

<a name="adding-custom-ai-guidelines"></a>
### Dodawanie własnych wytycznych AI

Aby rozszerzyć Laravel Boost o własne wytyczne AI, dodaj pliki `.blade.php` lub `.md` do katalogu `.ai/guidelines/*` swojej aplikacji. Te pliki zostaną automatycznie włączone wraz z wytycznymi Laravel Boost, gdy uruchomisz `boost:install`.

<a name="overriding-boost-ai-guidelines"></a>
### Nadpisywanie wytycznych Boost AI

Możesz nadpisać wbudowane wytyczne AI Boost, tworząc własne wytyczne z pasującymi ścieżkami plików. Gdy utworzysz niestandardową wytyczną, która pasuje do istniejącej ścieżki wytycznej Boost, Boost użyje Twojej niestandardowej wersji zamiast wbudowanej.

Na przykład, aby nadpisać wytyczne Boost "Inertia React v2 Form Guidance", utwórz plik w `.ai/guidelines/inertia-react/2/forms.blade.php`. Gdy uruchomisz `boost:install`, Boost dołączy Twoją niestandardową wytyczną zamiast domyślnej.

<a name="third-party-package-ai-guidelines"></a>
### Wytyczne AI pakietów zewnętrznych

Jeśli utrzymujesz pakiet zewnętrzny i chciałbyś, aby Boost zawierał wytyczne AI dla niego, możesz to zrobić, dodając plik `resources/boost/guidelines/core.blade.php` do swojego pakietu. Gdy użytkownicy Twojego pakietu uruchomią `php artisan boost:install`, Boost automatycznie załaduje Twoje wytyczne.

Wytyczne AI powinny zawierać krótki przegląd tego, co robi Twój pakiet, określać wymaganą strukturę plików lub konwencje oraz wyjaśniać, jak tworzyć lub używać jego głównych funkcji (z przykładowymi poleceniami lub fragmentami kodu). Trzymaj je zwięzłe, praktyczne i skoncentrowane na najlepszych praktykach, aby AI mogło generować poprawny kod dla Twoich użytkowników. Oto przykład:

```php
## Nazwa pakietu

Ten pakiet zapewnia [krótki opis funkcjonalności].

### Funkcje

- Funkcja 1: [jasny i krótki opis].
- Funkcja 2: [jasny i krótki opis]. Przykładowe użycie:

@verbatim
<code-snippet name="Jak używać Funkcji 2" lang="php">
$result = PackageName::featureTwo($param1, $param2);
</code-snippet>
@endverbatim
```

<a name="agent-skills"></a>
## Umiejętności agentów

[Agent Skills](https://agentskills.io/home) to lekkie, ukierunkowane moduły wiedzy, które agenci mogą aktywować na żądanie podczas pracy w określonych domenach. W przeciwieństwie do wytycznych, które są ładowane z góry, umiejętności pozwalają na ładowanie szczegółowych wzorców i najlepszych praktyk tylko wtedy, gdy jest to istotne, redukując nadmiar kontekstu i poprawiając trafność kodu generowanego przez AI.

Gdy uruchomisz `boost:install` i wybierzesz umiejętności jako funkcję, umiejętności są automatycznie instalowane na podstawie pakietów wykrytych w Twoim `composer.json`. Na przykład, jeśli Twój projekt zawiera `livewire/livewire`, umiejętność `livewire-development` zostanie zainstalowana automatycznie.

<a name="available-skills"></a>
### Dostępne umiejętności

| Skill                        | Package      |
|------------------------------|--------------|
| fluxui-development           | Flux UI      |
| folio-routing                | Folio        |
| inertia-react-development    | Inertia React|
| inertia-svelte-development   | Inertia Svelte|
| inertia-vue-development      | Inertia Vue  |
| livewire-development         | Livewire     |
| mcp-development              | MCP          |
| pennant-development          | Pennant      |
| pest-testing                 | Pest         |
| tailwindcss-development      | Tailwind CSS |
| volt-development             | Volt         |
| wayfinder-development        | Wayfinder    |

> **Uwaga:** Aby zaktualizować swoje umiejętności, zobacz sekcję [Aktualizowanie zasobów Boost](#keeping-boost-resources-updated).

<a name="custom-skills"></a>
### Własne umiejętności

Aby stworzyć własne niestandardowe umiejętności, dodaj plik `SKILL.md` do katalogu `.ai/skills/{skill-name}/` swojej aplikacji. Gdy uruchomisz `boost:update`, Twoje niestandardowe umiejętności zostaną zainstalowane obok wbudowanych umiejętności Boost.

Na przykład, aby utworzyć niestandardową umiejętność dla logiki domenowej aplikacji:

```
.ai/skills/creating-invoices/SKILL.md
```

<a name="overriding-skills"></a>
### Nadpisywanie umiejętności

Możesz nadpisać wbudowane umiejętności Boost, tworząc własne niestandardowe umiejętności o pasujących nazwach. Gdy utworzysz niestandardową umiejętność, która pasuje do istniejącej nazwy umiejętności Boost, Boost użyje Twojej niestandardowej wersji zamiast wbudowanej.

Na przykład, aby nadpisać umiejętność `livewire-development` Boost, utwórz plik w `.ai/skills/livewire-development/SKILL.md`. Gdy uruchomisz `boost:update`, Boost dołączy Twoją niestandardową umiejętność zamiast domyślnej.

<a name="third-party-package-skills"></a>
### Umiejętności pakietów zewnętrznych

Jeśli utrzymujesz pakiet zewnętrzny i chciałbyś, aby Boost zawierał umiejętności dla niego, możesz to zrobić, dodając plik `resources/boost/skills/{skill-name}/SKILL.md` do swojego pakietu. Gdy użytkownicy Twojego pakietu uruchomią `php artisan boost:install`, Boost automatycznie zainstaluje Twoje umiejętności na podstawie preferencji użytkownika.

Boost Skills obsługuje [format Agent Skills](https://agentskills.io/what-are-skills) i powinny być ustrukturyzowane jako folder zawierający plik `SKILL.md` z nagłówkiem YAML i instrukcjami Markdown. Plik `SKILL.md` musi zawierać wymagany nagłówek (`name` i `description`) i może opcjonalnie zawierać skrypty, szablony i materiały referencyjne.

Umiejętności powinny określać wymaganą strukturę plików lub konwencje oraz wyjaśniać, jak tworzyć lub używać jego głównych funkcji (z przykładowymi poleceniami lub fragmentami kodu). Trzymaj je zwięzłe, praktyczne i skoncentrowane na najlepszych praktykach, aby AI mogło generować poprawny kod dla Twoich użytkowników:

```markdown
---
name: package-name-development
description: Tworzenie i praca z funkcjami PackageName, w tym komponentami i przepływami pracy.
---

# Rozwój Package Name

## Kiedy używać tej umiejętności
Używaj tej umiejętności podczas pracy z funkcjami PackageName...

## Funkcje

- Funkcja 1: [jasny i krótki opis].
- Funkcja 2: [jasny i krótki opis]. Przykładowe użycie:

$result = PackageName::featureTwo($param1, $param2);
```

<a name="guidelines-vs-skills"></a>
## Wytyczne vs. Umiejętności

Laravel Boost zapewnia dwa odrębne sposoby dostarczania agentom AI kontekstu o Twojej aplikacji: **wytyczne** i **umiejętności**.

**Wytyczne** są ładowane z góry, gdy agent AI się uruchamia, zapewniając niezbędny kontekst dotyczący konwencji Laravel i najlepszych praktyk, które mają szerokie zastosowanie w całej bazie kodu.

**Umiejętności** są aktywowane na żądanie podczas pracy nad konkretnymi zadaniami, zawierając szczegółowe wzorce dla określonych dziedzin (takich jak komponenty Livewire lub testy Pest). Ładowanie umiejętności tylko wtedy, gdy są istotne, redukuje nadmiar kontekstu i poprawia jakość kodu.

| Aspekt | Wytyczne | Umiejętności |
|--------|------------|--------|
| **Ładowanie** | Z góry, zawsze obecne | Na żądanie, gdy istotne |
| **Zakres** | Szeroki, fundamentalny | Skoncentrowany, specyficzny dla zadania |
| **Cel** | Główne konwencje i najlepsze praktyki | Szczegółowe wzorce implementacji |

<a name="documentation-api"></a>
## API dokumentacji

Laravel Boost zawiera API dokumentacji, które zapewnia agentom AI dostęp do obszernej bazy wiedzy zawierającej ponad 17 000 informacji specyficznych dla Laravel. API wykorzystuje wyszukiwanie semantyczne z embeddingami do dostarczania precyzyjnych, kontekstowych wyników.

Narzędzie MCP `Search Docs` umożliwia agentom odpytywanie usługi API dokumentacji Laravel w celu pobrania dokumentacji na podstawie zainstalowanych pakietów. Wytyczne AI i umiejętności Boost automatycznie poinstruują Twojego agenta kodowania, aby używał tego API.

| Pakiet            | Wersje obsługiwane |
|-------------------|--------------------||
| Laravel Framework | 10.x, 11.x, 12.x   ||
| Filament          | 2.x, 3.x, 4.x, 5.x ||
| Flux UI           | 2.x Free, 2.x Pro  ||
| Inertia           | 1.x, 2.x           ||
| Livewire          | 1.x, 2.x, 3.x, 4.x ||
| Nova              | 4.x, 5.x           ||
| Pest              | 3.x, 4.x           ||
| Tailwind CSS      | 3.x, 4.x           ||

<a name="extending-boost"></a>
## Rozszerzanie Boost

Boost działa z wieloma popularnymi IDE i agentami AI od razu po wyjęciu z pudełka. Jeśli Twoje narzędzie kodowania nie jest jeszcze obsługiwane, możesz utworzyć własnego agenta i zintegrować go z Boost.

<a name="adding-support-for-other-ides-ai-agents"></a>
### Dodawanie wsparcia dla innych IDE / Agentów AI

Aby dodać wsparcie dla nowego IDE lub agenta AI, utwórz klasę, która rozszerza `Laravel\Boost\Install\Agents\Agent` i zaimplementuj jeden lub więcej z następujących kontraktów w zależności od tego, czego potrzebujesz:

- `Laravel\Boost\Contracts\SupportsGuidelines` - Dodaje wsparcie dla wytycznych AI.
- `Laravel\Boost\Contracts\SupportsMcp` - Dodaje wsparcie dla MCP.
- `Laravel\Boost\Contracts\SupportsSkills` - Dodaje wsparcie dla umiejętności agentów.

<a name="writing-the-agent"></a>
#### Pisanie agenta

```php
<?php

declare(strict_types=1);

namespace App;

use Laravel\Boost\Contracts\SupportsGuidelines;
use Laravel\Boost\Contracts\SupportsMcp;
use Laravel\Boost\Contracts\SupportsSkills;
use Laravel\Boost\Install\Agents\Agent;

class CustomAgent extends Agent implements SupportsGuidelines, SupportsMcp, SupportsSkills
{
    // Twoja implementacja...
}
```

Przykład implementacji znajdziesz w [ClaudeCode.php](https://github.com/laravel/boost/blob/main/src/Install/Agents/ClaudeCode.php).

<a name="registering-the-agent"></a>
#### Rejestrowanie agenta

Zarejestruj swojego niestandardowego agenta w metodzie `boot` klasy `App\Providers\AppServiceProvider` swojej aplikacji:

```php
use Laravel\Boost\Boost;

public function boot(): void
{
    Boost::registerAgent('customagent', CustomAgent::class);
}
```

Po zarejestrowaniu Twój agent będzie dostępny do wyboru podczas uruchamiania `php artisan boost:install`.
