# Przewodnik współtworzenia

- [Raporty o błędach](#bug-reports)
- [Pytania o wsparcie](#support-questions)
- [Dyskusja o rozwoju rdzenia](#core-development-discussion)
- [Która gałąź?](#which-branch)
- [Skompilowane zasoby](#compiled-assets)
- [Luki bezpieczeństwa](#security-vulnerabilities)
- [Styl kodowania](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)
- [Kodeks postępowania](#code-of-conduct)

<a name="bug-reports"></a>
## Raporty o błędach

Aby zachęcić do aktywnej współpracy, Laravel zdecydowanie zachęca do pull requestów, a nie tylko raportów o błędach. Pull requesty będą przeglądane tylko wtedy, gdy są oznaczone jako "ready for review" (nie w stanie "draft") i wszystkie testy dla nowych funkcjonalności przechodzą pomyślnie. Nieaktywne pull requesty pozostawione w stanie "draft" zostaną zamknięte po kilku dniach.

Jednakże, jeśli zgłaszasz raport o błędzie, Twoje zgłoszenie powinno zawierać tytuł i jasny opis problemu. Powinieneś również dołączyć jak najwięcej istotnych informacji i przykład kodu, który demonstruje problem. Celem raportu o błędzie jest ułatwienie sobie - i innym - replikacji błędu i opracowania poprawki.

Pamiętaj, raporty o błędach są tworzone w nadziei, że inni z tym samym problemem będą mogli współpracować z Tobą nad jego rozwiązaniem. Nie oczekuj, że raport o błędzie automatycznie zobaczy jakąkolwiek aktywność lub że inni rzucą się go naprawiać. Tworzenie raportu o błędzie służy pomocy sobie i innym w rozpoczęciu drogi do naprawienia problemu. Jeśli chcesz pomóc, możesz to zrobić naprawiając [dowolne błędy wymienione w naszych systemach śledzenia problemów](https://github.com/issues?q=is%3Aopen+is%3Aissue+label%3Abug+user%3Alaravel). Musisz być uwierzytelniony w GitHub, aby zobaczyć wszystkie problemy Laravel.

Jeśli zauważysz nieprawidłowe DocBlock, PHPStan lub ostrzeżenia IDE podczas używania Laravel, nie twórz problemu GitHub. Zamiast tego prześlij pull request naprawiający problem.

Kod źródłowy Laravel jest zarządzany na GitHub, a dla każdego z projektów Laravel istnieją repozytoria:

<div class="content-list" markdown="1">

- [Aplikacja Laravel](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel Boost](https://github.com/laravel/boost)
- [Dokumentacja Laravel](https://github.com/laravel/docs)
- [Laravel Dusk](https://github.com/laravel/dusk)
- [Laravel Cashier Stripe](https://github.com/laravel/cashier)
- [Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle)
- [Laravel Echo](https://github.com/laravel/echo)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Folio](https://github.com/laravel/folio)
- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Homestead](https://github.com/laravel/homestead) ([Build Scripts](https://github.com/laravel/settler))
- [Laravel Horizon](https://github.com/laravel/horizon)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Pennant](https://github.com/laravel/pennant)
- [Laravel Pint](https://github.com/laravel/pint)
- [Laravel Prompts](https://github.com/laravel/prompts)
- [Laravel Reverb](https://github.com/laravel/reverb)
- [Laravel Sail](https://github.com/laravel/sail)
- [Laravel Sanctum](https://github.com/laravel/sanctum)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Telescope](https://github.com/laravel/telescope)
- [Laravel Livewire Starter Kit](https://github.com/laravel/livewire-starter-kit)
- [Laravel React Starter Kit](https://github.com/laravel/react-starter-kit)
- [Laravel Vue Starter Kit](https://github.com/laravel/vue-starter-kit)

</div>

<a name="support-questions"></a>
## Pytania o wsparcie

Systemy śledzenia problemów GitHub w Laravel nie są przeznaczone do udzielania pomocy lub wsparcia dla Laravel. Zamiast tego użyj jednego z następujących kanałów:

<div class="content-list" markdown="1">

- [GitHub Discussions](https://github.com/laravel/framework/discussions)
- [Laracasts Forums](https://laracasts.com/discuss)
- [Laravel.io Forums](https://laravel.io/forum)
- [StackOverflow](https://stackoverflow.com/questions/tagged/laravel)
- [Discord](https://discord.gg/laravel)
- [Larachat](https://larachat.co)
- [IRC](https://web.libera.chat/?nick=artisan&channels=#laravel)

</div>

<a name="core-development-discussion"></a>
## Dyskusja o rozwoju rdzenia

Możesz proponować nowe funkcje lub ulepszenia istniejącego zachowania Laravel na [tablicy dyskusyjnej GitHub](https://github.com/laravel/framework/discussions) repozytorium Laravel framework. Jeśli proponujesz nową funkcję, bądź gotowy do zaimplementowania przynajmniej części kodu, który będzie potrzebny do ukończenia funkcji.

Nieformalna dyskusja dotycząca błędów, nowych funkcji i implementacji istniejących funkcji odbywa się na kanale `#internals` [serwera Discord Laravel](https://discord.gg/laravel). Taylor Otwell, opiekun Laravel, jest zazwyczaj obecny na kanale w dni powszednie od 8:00 do 17:00 (UTC-06:00 lub America/Chicago) i sporadycznie obecny na kanale w innych godzinach.

<a name="which-branch"></a>
## Która gałąź?

**Wszystkie** poprawki błędów powinny być wysyłane do najnowszej wersji, która obsługuje poprawki błędów (obecnie `12.x`). Poprawki błędów **nigdy** nie powinny być wysyłane do gałęzi `master`, chyba że naprawiają funkcje, które istnieją tylko w nadchodzącej wersji.

**Mniejsze** funkcje, które są **w pełni wstecznie kompatybilne** z bieżącą wersją, mogą być wysyłane do najnowszej stabilnej gałęzi (obecnie `12.x`).

**Większe** nowe funkcje lub funkcje ze zmianami niezgodnymi wstecz powinny zawsze być wysyłane do gałęzi `master`, która zawiera nadchodzącą wersję.

<a name="compiled-assets"></a>
## Skompilowane zasoby

Jeśli przesyłasz zmianę, która będzie miała wpływ na skompilowany plik, taki jak większość plików w `resources/css` lub `resources/js` repozytorium `laravel/laravel`, nie commituj skompilowanych plików. Ze względu na ich duży rozmiar nie mogą być realistycznie sprawdzone przez opiekuna. Może to zostać wykorzystane jako sposób na wstrzyknięcie złośliwego kodu do Laravel. W celu zapobiegawczego zapobieżenia temu, wszystkie skompilowane pliki będą generowane i commitowane przez opiekunów Laravel.

<a name="security-vulnerabilities"></a>
## Luki bezpieczeństwa

Jeśli odkryjesz lukę bezpieczeństwa w Laravel, wyślij e-mail do Taylor Otwell na adres <a href="mailto:taylor@laravel.com">taylor@laravel.com</a>. Wszystkie luki bezpieczeństwa zostaną niezwłocznie rozwiązane.

<a name="coding-style"></a>
## Styl kodowania

Laravel przestrzega standardu kodowania [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) oraz standardu autoładowania [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md).

<a name="phpdoc"></a>
### PHPDoc

Poniżej znajduje się przykład prawidłowego bloku dokumentacji Laravel. Zauważ, że atrybut `@param` jest następowany przez dwie spacje, typ argumentu, dwie kolejne spacje i na końcu nazwę zmiennej:

```php
/**
 * Register a binding with the container.
 *
 * @param  string|array  $abstract
 * @param  \Closure|string|null  $concrete
 * @param  bool  $shared
 * @return void
 *
 * @throws \Exception
 */
public function bind($abstract, $concrete = null, $shared = false)
{
    // ...
}
```

Gdy atrybuty `@param` lub `@return` są zbędne ze względu na użycie typów natywnych, można je usunąć:

```php
/**
 * Execute the job.
 */
public function handle(AudioProcessor $processor): void
{
    // ...
}
```

Jednak gdy typ natywny jest generyczny, określ typ generyczny za pomocą atrybutów `@param` lub `@return`:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromStorage('/path/to/file'),
    ];
}
```

<a name="styleci"></a>
### StyleCI

Nie martw się, jeśli styl Twojego kodu nie jest idealny! [StyleCI](https://styleci.io/) automatycznie scali wszelkie poprawki stylu do repozytorium Laravel po scaleniu pull requestów. Pozwala nam to skupić się na treści wkładu, a nie na stylu kodu.

<a name="code-of-conduct"></a>
## Kodeks postępowania

Kodeks postępowania Laravel wywodzi się z kodeksu postępowania Ruby. Wszelkie naruszenia kodeksu postępowania mogą być zgłaszane do Taylor Otwell (taylor@laravel.com):

<div class="content-list" markdown="1">

- Uczestnicy będą tolerancyjni wobec przeciwnych poglądów.
- Uczestnicy muszą zapewnić, że ich język i działania są wolne od osobistych ataków i uwłaczających uwag osobistych.
- Interpretując słowa i działania innych, uczestnicy powinni zawsze zakładać dobre intencje.
- Zachowanie, które można rozsądnie uznać za nękanie, nie będzie tolerowane.

</div>
