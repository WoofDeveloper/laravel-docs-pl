# Lokalizacja

- [Wprowadzenie](#introduction)
    - [Publikowanie plików językowych](#publishing-the-language-files)
    - [Konfiguracja ustawień regionalnych](#configuring-the-locale)
    - [Język liczby mnogiej](#pluralization-language)
- [Definiowanie ciągów tłumaczeń](#defining-translation-strings)
    - [Używanie krótkich kluczy](#using-short-keys)
    - [Używanie ciągów tłumaczeń jako kluczy](#using-translation-strings-as-keys)
- [Pobieranie ciągów tłumaczeń](#retrieving-translation-strings)
    - [Zastępowanie parametrów w ciągach tłumaczeń](#replacing-parameters-in-translation-strings)
    - [Liczba mnoga](#pluralization)
- [Nadpisywanie plików językowych pakietów](#overriding-package-language-files)

<a name="introduction"></a>
## Wprowadzenie

> [!NOTE]
> Domyślnie szkielet aplikacji Laravel nie zawiera katalogu `lang`. Jeśli chcesz dostosować pliki językowe Laravel, możesz je opublikować za pomocą polecenia Artisan `lang:publish`.

Funkcje lokalizacji Laravel zapewniają wygodny sposób pobierania ciągów w różnych językach, umożliwiając łatwą obsługę wielu języków w Twojej aplikacji.

Laravel zapewnia dwa sposoby zarządzania ciągami tłumaczeń. Po pierwsze, ciągi językowe mogą być przechowywane w plikach w katalogu `lang` aplikacji. W tym katalogu mogą znajdować się podkatalogi dla każdego języka obsługiwanego przez aplikację. Jest to podejście, którego Laravel używa do zarządzania ciągami tłumaczeń dla wbudowanych funkcji Laravel, takich jak komunikaty o błędach walidacji:

```text
/lang
    /en
        messages.php
    /es
        messages.php
```

Lub ciągi tłumaczeń mogą być zdefiniowane w plikach JSON umieszczonych w katalogu `lang`. Przy tym podejściu każdy język obsługiwany przez aplikację miałby odpowiadający plik JSON w tym katalogu. To podejście jest zalecane dla aplikacji z dużą liczbą ciągów do przetłumaczenia:

```text
/lang
    en.json
    es.json
```

Omówimy każde podejście do zarządzania ciągami tłumaczeń w tej dokumentacji.

<a name="publishing-the-language-files"></a>
### Publikowanie plików językowych

Domyślnie szkielet aplikacji Laravel nie zawiera katalogu `lang`. Jeśli chcesz dostosować pliki językowe Laravel lub stworzyć własne, powinieneś utworzyć katalog `lang` za pomocą polecenia Artisan `lang:publish`. Polecenie `lang:publish` utworzy katalog `lang` w Twojej aplikacji i opublikuje domyślny zestaw plików językowych używanych przez Laravel:

```shell
php artisan lang:publish
```

<a name="configuring-the-locale"></a>
### Konfiguracja ustawień regionalnych

Domyślny język aplikacji jest przechowywany w opcji konfiguracji `locale` pliku konfiguracyjnego `config/app.php`, która jest zazwyczaj ustawiana za pomocą zmiennej środowiskowej `APP_LOCALE`. Możesz swobodnie zmodyfikować tę wartość, aby dostosować ją do potrzeb Twojej aplikacji.

Możesz również skonfigurować "język zapasowy", który będzie używany, gdy domyślny język nie zawiera danego ciągu tłumaczenia. Podobnie jak domyślny język, język zapasowy jest również konfigurowany w pliku konfiguracyjnym `config/app.php`, a jego wartość jest zazwyczaj ustawiana za pomocą zmiennej środowiskowej `APP_FALLBACK_LOCALE`.

Możesz zmodyfikować domyślny język dla pojedynczego żądania HTTP w czasie wykonywania, używając metody `setLocale` dostępnej w fasadzie `App`:

```php
use Illuminate\Support\Facades\App;

Route::get('/greeting/{locale}', function (string $locale) {
    if (! in_array($locale, ['en', 'es', 'fr'])) {
        abort(400);
    }

    App::setLocale($locale);

    // ...
});
```

<a name="determining-the-current-locale"></a>
#### Określanie bieżących ustawień regionalnych

Możesz użyć metod `currentLocale` i `isLocale` w fasadzie `App`, aby określić bieżące ustawienia regionalne lub sprawdzić, czy ustawienia regionalne mają daną wartość:

```php
use Illuminate\Support\Facades\App;

$locale = App::currentLocale();

if (App::isLocale('en')) {
    // ...
}
```

<a name="pluralization-language"></a>
### Język liczby mnogiej

<style>
.code-list-no-flex-break code {
    display: contents !important;
}
</style>

<div class="code-list-no-flex-break">

Możesz poinstruować "pluralizer" Laravel, który jest używany przez Eloquent i inne części frameworka do konwersji ciągów w liczbie pojedynczej na liczbę mnogą, aby używał innego języka niż angielski. Można to osiągnąć, wywołując metodę `useLanguage` w metodzie `boot` jednego z dostawców usług Twojej aplikacji. Obecnie obsługiwane języki pluralizera to: `french`, `norwegian-bokmal`, `portuguese`, `spanish` i `turkish`:

</div>

```php
use Illuminate\Support\Pluralizer;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Pluralizer::useLanguage('spanish');

    // ...
}
```

> [!WARNING]
> Jeśli dostosujesz język pluralizera, powinieneś jawnie zdefiniować [nazwy tabel](/docs/{{version}}/eloquent#table-names) Twoich modeli Eloquent.

<a name="defining-translation-strings"></a>
## Definiowanie ciągów tłumaczeń

<a name="using-short-keys"></a>
### Używanie krótkich kluczy

Zazwyczaj ciągi tłumaczeń są przechowywane w plikach w katalogu `lang`. W tym katalogu powinien znajdować się podkatalog dla każdego języka obsługiwanego przez Twoją aplikację. Jest to podejście, którego Laravel używa do zarządzania ciągami tłumaczeń dla wbudowanych funkcji Laravel, takich jak komunikaty o błędach walidacji:

```text
/lang
    /en
        messages.php
    /es
        messages.php
```

Wszystkie pliki językowe zwracają tablicę ciągów z kluczami. Na przykład:

```php
<?php

// lang/en/messages.php

return [
    'welcome' => 'Welcome to our application!',
];
```

> [!WARNING]
> Dla języków różniących się terytorium, powinieneś nazywać katalogi językowe zgodnie z ISO 15897. Na przykład "en_GB" powinno być używane dla brytyjskiego angielskiego zamiast "en-gb".

<a name="using-translation-strings-as-keys"></a>
### Używanie ciągów tłumaczeń jako kluczy

Dla aplikacji z dużą liczbą ciągów do przetłumaczenia, definiowanie każdego ciągu za pomocą "krótkiego klucza" może być mylące podczas odwoływania się do kluczy w widokach i może być uciążliwe ciągłe wymyślanie kluczy dla każdego ciągu tłumaczenia obsługiwanego przez aplikację.

Z tego powodu Laravel zapewnia również wsparcie dla definiowania ciągów tłumaczeń za pomocą "domyślnego" tłumaczenia ciągu jako klucza. Pliki językowe, które używają ciągów tłumaczeń jako kluczy, są przechowywane jako pliki JSON w katalogu `lang`. Na przykład, jeśli Twoja aplikacja ma hiszpańskie tłumaczenie, powinieneś utworzyć plik `lang/es.json`:

```json
{
    "I love programming.": "Me encanta programar."
}
```

#### Konflikty kluczy / plików

Nie powinieneś definiować kluczy ciągów tłumaczeń, które kolidują z innymi nazwami plików tłumaczeń. Na przykład, tłumaczenie `__('Action')` dla regionu "NL", podczas gdy istnieje plik `nl/action.php`, ale nie istnieje plik `nl.json`, spowoduje, że translator zwróci całą zawartość `nl/action.php`.

<a name="retrieving-translation-strings"></a>
## Pobieranie ciągów tłumaczeń

Możesz pobierać ciągi tłumaczeń ze swoich plików językowych za pomocą funkcji pomocniczej `__`. Jeśli używasz "krótkich kluczy" do definiowania ciągów tłumaczeń, powinieneś przekazać plik zawierający klucz i sam klucz do funkcji `__` używając składni "kropkowej". Na przykład pobierzmy ciąg tłumaczenia `welcome` z pliku językowego `lang/en/messages.php`:

```php
echo __('messages.welcome');
```

Jeśli określony ciąg tłumaczenia nie istnieje, funkcja `__` zwróci klucz ciągu tłumaczenia. Więc, używając powyższego przykładu, funkcja `__` zwróciłaby `messages.welcome`, jeśli ciąg tłumaczenia nie istnieje.

Jeśli używasz [domyślnych ciągów tłumaczeń jako kluczy tłumaczeń](#using-translation-strings-as-keys), powinieneś przekazać domyślne tłumaczenie Twojego ciągu do funkcji `__`;

```php
echo __('I love programming.');
```

Ponownie, jeśli ciąg tłumaczenia nie istnieje, funkcja `__` zwróci klucz ciągu tłumaczenia, który otrzymała.

Jeśli używasz [silnika szablonów Blade](/docs/{{version}}/blade), możesz użyć składni echo `{{ }}`, aby wyświetlić ciąg tłumaczenia:

```blade
{{ __('messages.welcome') }}
```

<a name="replacing-parameters-in-translation-strings"></a>
### Zastępowanie parametrów w ciągach tłumaczeń

Jeśli chcesz, możesz zdefiniować symbole zastępcze w swoich ciągach tłumaczeń. Wszystkie symbole zastępcze są poprzedzone znakiem `:`. Na przykład możesz zdefiniować komunikat powitalny z symbolem zastępczym nazwy:

```php
'welcome' => 'Welcome, :name',
```

Aby zastąpić symbole zastępcze podczas pobierania ciągu tłumaczenia, możesz przekazać tablicę zamienników jako drugi argument do funkcji `__`:

```php
echo __('messages.welcome', ['name' => 'dayle']);
```

Jeśli Twój symbol zastępczy zawiera wszystkie wielkie litery lub ma tylko pierwszą literę wielką, przetłumaczona wartość będzie odpowiednio skapitalizowana:

```php
'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle
```

<a name="object-replacement-formatting"></a>
#### Formatowanie zamiany obiektów

Jeśli próbujesz podać obiekt jako symbol zastępczy tłumaczenia, zostanie wywołana metoda `__toString` obiektu. Metoda [__toString](https://www.php.net/manual/en/language.oop5.magic.php#object.tostring) jest jedną z wbudowanych "magicznych metod" PHP. Jednak czasami możesz nie mieć kontroli nad metodą `__toString` danej klasy, na przykład gdy klasa, z którą wchodzisz w interakcję, należy do biblioteki innej firmy.

W takich przypadkach Laravel pozwala zarejestrować niestandardowy program obsługi formatowania dla tego konkretnego typu obiektu. Aby to osiągnąć, powinieneś wywołać metodę `stringable` translatora. Metoda `stringable` przyjmuje domknięcie, które powinno wskazywać typ obiektu, za którego formatowanie jest odpowiedzialne. Zazwyczaj metoda `stringable` powinna być wywoływana w metodzie `boot` klasy `AppServiceProvider` Twojej aplikacji:

```php
use Illuminate\Support\Facades\Lang;
use Money\Money;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Lang::stringable(function (Money $money) {
        return $money->formatTo('en_GB');
    });
}
```

<a name="pluralization"></a>
### Liczba mnoga

Liczba mnoga to skomplikowany problem, ponieważ różne języki mają różnorodne złożone zasady dla liczby mnogiej; jednak Laravel może pomóc w tłumaczeniu ciągów na różne sposoby w zależności od zdefiniowanych przez Ciebie zasad liczby mnogiej. Używając znaku `|`, możesz rozróżnić formy pojedyncze i mnogie ciągu:

```php
'apples' => 'There is one apple|There are many apples',
```

Oczywiście liczba mnoga jest również obsługiwana podczas [używania ciągów tłumaczeń jako kluczy](#using-translation-strings-as-keys):

```json
{
    "There is one apple|There are many apples": "Hay una manzana|Hay muchas manzanas"
}
```

Możesz nawet tworzyć bardziej złożone zasady liczby mnogiej, które określają ciągi tłumaczeń dla wielu zakresów wartości:

```php
'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',
```

Po zdefiniowaniu ciągu tłumaczenia z opcjami liczby mnogiej możesz użyć funkcji `trans_choice`, aby pobrać linię dla danej "liczby". W tym przykładzie, ponieważ liczba jest większa niż jeden, zwracana jest forma mnoga ciągu tłumaczenia:

```php
echo trans_choice('messages.apples', 10);
```

Możesz również zdefiniować atrybuty zastępcze w ciągach liczby mnogiej. Te symbole zastępcze mogą być zastąpione przez przekazanie tablicy jako trzeciego argumentu do funkcji `trans_choice`:

```php
'minutes_ago' => '{1} :value minute ago|[2,*] :value minutes ago',

echo trans_choice('time.minutes_ago', 5, ['value' => 5]);
```

Jeśli chcesz wyświetlić wartość całkowitą, która została przekazana do funkcji `trans_choice`, możesz użyć wbudowanego symbolu zastępczego `:count`:

```php
'apples' => '{0} There are none|{1} There is one|[2,*] There are :count',
```

<a name="overriding-package-language-files"></a>
## Nadpisywanie plików językowych pakietów

Niektóre pakiety mogą dostarczać własne pliki językowe. Zamiast zmieniać podstawowe pliki pakietu, aby poprawić te linie, możesz je nadpisać, umieszczając pliki w katalogu `lang/vendor/{package}/{locale}`.

Więc na przykład, jeśli musisz nadpisać angielskie ciągi tłumaczeń w `messages.php` dla pakietu o nazwie `skyrim/hearthfire`, powinieneś umieścić plik językowy w: `lang/vendor/hearthfire/en/messages.php`. W tym pliku powinieneś zdefiniować tylko te ciągi tłumaczeń, które chcesz nadpisać. Wszelkie ciągi tłumaczeń, których nie nadpiszesz, nadal będą ładowane z oryginalnych plików językowych pakietu.
