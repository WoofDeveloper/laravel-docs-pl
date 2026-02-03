# Informacje o wydaniach

- [Schemat wersjonowania](#versioning-scheme)
- [Polityka wsparcia](#support-policy)
- [Laravel 12](#laravel-12)

<a name="versioning-scheme"></a>
## Schemat wersjonowania

Laravel i jego inne oficjalne pakiety stosują [Wersjonowanie semantyczne](https://semver.org). Główne wydania frameworka są publikowane co roku (~Q1), podczas gdy wydania mniejsze i poprawki mogą być publikowane nawet co tydzień. Wydania mniejsze i poprawki **nigdy** nie powinny zawierać zmian łamiących wsteczną kompatybilność.

Odwołując się do frameworka Laravel lub jego komponentów z twojej aplikacji lub pakietu, zawsze powinieneś używać ograniczenia wersji takiego jak `^12.0`, ponieważ główne wydania Laravel zawierają zmiany łamiące wsteczną kompatybilność. Jednak staramy się zawsze zapewnić, że możesz zaktualizować do nowego głównego wydania w jeden dzień lub krócej.

<a name="named-arguments"></a>
#### Nazwane argumenty

[Nazwane argumenty](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments) nie są objęte wytycznymi wstecznej kompatybilności Laravel. Możemy zdecydować się na zmianę nazw argumentów funkcji, gdy jest to konieczne w celu poprawy kodu Laravel. Dlatego używanie nazwanych argumentów podczas wywoływania metod Laravel powinno być wykonywane ostrożnie i ze świadomością, że nazwy parametrów mogą się zmienić w przyszłości.

<a name="support-policy"></a>
## Polityka wsparcia

Dla wszystkich wydań Laravel poprawki błędów są dostarczane przez 18 miesięcy, a poprawki bezpieczeństwa przez 2 lata. Dla wszystkich dodatkowych bibliotek tylko najnowsze główne wydanie otrzymuje poprawki błędów. Dodatkowo przejrzyj wersje baz danych [obsługiwane przez Laravel](/docs/{{version}}/database#introduction).

<div class="overflow-auto">

| Wersja | PHP (*)   | Wydanie             | Poprawki błędów do  | Poprawki bezpieczeństwa do |
| ------- |-----------| ------------------- | ------------------- | -------------------- |
| 10      | 8.1 - 8.3 | 14 lutego 2023 | 6 sierpnia 2024    | 4 lutego 2025   |
| 11      | 8.2 - 8.4 | 12 marca 2024    | 3 września 2025 | 12 marca 2026     |
| 12      | 8.2 - 8.5 | 24 lutego 2025 | 13 sierpnia 2026   | 24 lutego 2027  |
| 13      | 8.3 - 8.5 | Q1 2026             | Q3 2027             | Q1 2028              |

</div>

<div class="version-colors">
    <div class="end-of-life">
        <div class="color-box"></div>
        <div>Koniec wsparcia</div>
    </div>
    <div class="security-fixes">
        <div class="color-box"></div>
        <div>Tylko poprawki bezpieczeństwa</div>
    </div>
</div>

(*) Obsługiwane wersje PHP

<a name="laravel-12"></a>
## Laravel 12

Laravel 12 kontynuuje ulepszenia wprowadzone w Laravel 11.x poprzez aktualizację zależności upstream i wprowadzenie nowych zestawów startowych dla React, Vue i Livewire, w tym opcję użycia [WorkOS AuthKit](https://authkit.com) do uwierzytelniania użytkowników. Wariant WorkOS naszych zestawów startowych oferuje uwierzytelnianie społecznościowe, passkeys i obsługę SSO.

<a name="minimal-breaking-changes"></a>
### Minimalne zmiany łamiące

Większość naszego skupienia podczas tego cyklu wydania polegała na minimalizowaniu zmian łamiących. Zamiast tego poświęciliśmy się dostarczaniu ciągłych ulepszeń jakości życia przez cały rok, które nie łamią istniejących aplikacji.

Dlatego wydanie Laravel 12 jest stosunkowo niewielkim "wydaniem konserwacyjnym" w celu aktualizacji istniejących zależności. W świetle tego większość aplikacji Laravel może zaktualizować się do Laravel 12 bez zmiany jakiegokolwiek kodu aplikacji.

<a name="new-application-starter-kits"></a>
### Nowe zestawy startowe aplikacji

Laravel 12 wprowadza nowe [zestawy startowe aplikacji](/docs/{{version}}/starter-kits) dla React, Vue i Livewire. Zestawy startowe React i Vue wykorzystują Inertia 2, TypeScript, [shadcn/ui](https://ui.shadcn.com) i Tailwind, podczas gdy zestawy startowe Livewire wykorzystują opartą na Tailwind bibliotekę komponentów [Flux UI](https://fluxui.dev) oraz Laravel Volt.

Zestawy startowe React, Vue i Livewire wykorzystują wbudowany system uwierzytelniania Laravel, aby oferować logowanie, rejestrację, resetowanie hasła, weryfikację e-mail i więcej. Dodatkowo wprowadzamy wariant każdego zestawu startowego [napędzany przez WorkOS AuthKit](https://authkit.com), oferujący uwierzytelnianie społecznościowe, passkeys i obsługę SSO. WorkOS oferuje darmowe uwierzytelnianie dla aplikacji do 1 miliona miesięcznie aktywnych użytkowników.

Z wprowadzeniem naszych nowych zestawów startowych aplikacji, Laravel Breeze i Laravel Jetstream nie będą już otrzymywać dodatkowych aktualizacji.

Aby rozpocząć pracę z naszymi nowymi zestawami startowymi, zapoznaj się z [dokumentacją zestawów startowych](/docs/{{version}}/starter-kits).
