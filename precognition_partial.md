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

Używając Laravel Precognition, możesz oferować doświadczenia walidacji na żywo swoim użytkownikom bez konieczności duplikowania reguł walidacji w aplikacji frontend Vue. Aby zilustrować, jak to działa, zbudujmy formularz do tworzenia nowych użytkowników w naszej aplikacji.

Najpierw, aby włączyć Precognition dla trasy, middleware HandlePrecognitiveRequests powinien zostać dodany do definicji trasy. Powinieneś także utworzyć [żądanie formularza](/docs/{{version}}/validation#form-request-validation), aby pomieścić reguły walidacji trasy:
