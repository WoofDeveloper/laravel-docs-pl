# Cykl życia żądania

- [Wprowadzenie](#introduction)
- [Przegląd cyklu życia](#lifecycle-overview)
    - [Pierwsze kroki](#first-steps)
    - [Jądra HTTP / Console](#http-console-kernels)
    - [Dostawcy usług](#service-providers)
    - [Routing](#routing)
    - [Zakończenie](#finishing-up)
- [Skupienie na dostawcach usług](#focus-on-service-providers)

<a name="introduction"></a>
## Wprowadzenie

Kiedy używasz dowolnego narzędzia w "prawdziwym świecie", czujesz się pewniej, jeśli rozumiesz, jak to narzędzie działa. Rozwój aplikacji nie jest inny. Gdy rozumiesz, jak działają Twoje narzędzia programistyczne, czujesz się bardziej komfortowo i pewnie, używając ich.

Celem tego dokumentu jest danie Ci dobrego, wysokopoziomowego przeglądu tego, jak działa framework Laravel. Poznając ogólny framework lepiej, wszystko wydaje się mniej "magiczne" i będziesz bardziej pewny podczas budowania swoich aplikacji. Jeśli nie rozumiesz wszystkich terminów od razu, nie trać serca! Po prostu spróbuj uzyskać podstawowe zrozumienie tego, co się dzieje, a Twoja wiedza będzie rosła, gdy będziesz eksplorować inne sekcje dokumentacji.

<a name="lifecycle-overview"></a>
## Przegląd cyklu życia

<a name="first-steps"></a>
### Pierwsze kroki

Punktem wejścia dla wszystkich żądań do aplikacji Laravel jest plik `public/index.php`. Wszystkie żądania są kierowane do tego pliku przez konfigurację serwera webowego (Apache / Nginx). Plik `index.php` nie zawiera dużo kodu. Jest raczej punktem startowym do ładowania reszty frameworka.

Plik `index.php` ładuje definicję autoloadera wygenerowaną przez Composer, a następnie pobiera instancję aplikacji Laravel z `bootstrap/app.php`. Pierwszą akcją podjętą przez sam Laravel jest utworzenie instancji aplikacji / [kontenera usług](/docs/{{version}}/container).

<a name="http-console-kernels"></a>
### Jądra HTTP / Console

Następnie przychodzące żądanie jest wysyłane do jądra HTTP lub jądra konsolowego, używając metod `handleRequest` lub `handleCommand` instancji aplikacji, w zależności od typu żądania wchodzącego do aplikacji. Te dwa jądra służą jako centralna lokalizacja, przez którą przepływają wszystkie żądania. Na razie skupmy się na jądrze HTTP, które jest instancją `Illuminate\Foundation\Http\Kernel`.

Jądro HTTP definiuje tablicę `bootstrappers`, które zostaną uruchomione przed wykonaniem żądania. Te bootstrappery konfigurują obsługę błędów, konfigurują logowanie, [wykrywają środowisko aplikacji](/docs/{{version}}/configuration#environment-configuration) i wykonują inne zadania, które muszą zostać wykonane przed faktyczną obsługą żądania. Zazwyczaj te klasy obsługują wewnętrzną konfigurację Laravel, o którą nie musisz się martwić.

Jądro HTTP jest również odpowiedzialne za przekazanie żądania przez stos middleware aplikacji. Te middleware obsługują odczyt i zapis [sesji HTTP](/docs/{{version}}/session), określanie, czy aplikacja jest w trybie konserwacji, [weryfikację tokenu CSRF](/docs/{{version}}/csrf) i więcej. Porozmawiamy o tym więcej wkrótce.

Sygnatura metody `handle` jądra HTTP jest dość prosta: otrzymuje `Request` i zwraca `Response`. Pomyśl o jądrze jak o dużej czarnej skrzynce reprezentującej całą aplikację. Podaj jej żądania HTTP, a zwróci odpowiedzi HTTP.

<a name="service-providers"></a>
### Dostawcy usług

Jedną z najważniejszych akcji uruchamiania jądra jest ładowanie [dostawców usług](/docs/{{version}}/providers) dla Twojej aplikacji. Dostawcy usług są odpowiedzialni za uruchamianie wszystkich różnych komponentów frameworka, takich jak komponenty bazy danych, kolejki, walidacji i routingu.

Laravel przejdzie przez tę listę dostawców i utworzy instancję każdego z nich. Po utworzeniu instancji dostawców, metoda `register` zostanie wywołana na wszystkich dostawcach. Następnie, gdy wszyscy dostawcy zostaną zarejestrowani, metoda `boot` zostanie wywołana na każdym dostawcy. Dzieje się tak, aby dostawcy usług mogli zależeć od każdego powiązania kontenera, które zostało zarejestrowane i jest dostępne w momencie wykonania ich metody `boot`.

Zasadniczo każda główna funkcja oferowana przez Laravel jest uruchamiana i konfigurowana przez dostawcę usług. Ponieważ uruchamiają i konfigurują tak wiele funkcji oferowanych przez framework, dostawcy usług są najważniejszym aspektem całego procesu uruchamiania Laravel.

Podczas gdy framework wewnętrznie używa dziesiątek dostawców usług, masz również możliwość utworzenia własnych. Listę dostawców usług zdefiniowanych przez użytkownika lub stron trzecich, których używa Twoja aplikacja, możesz znaleźć w pliku `bootstrap/providers.php`.

<a name="routing"></a>
### Routing

Po uruchomieniu aplikacji i zarejestrowaniu wszystkich dostawców usług, `Request` zostanie przekazany do routera w celu wysłania. Router wyśle żądanie do trasy lub kontrolera, a także uruchomi wszelkie middleware specyficzne dla trasy.

Middleware zapewniają wygodny mechanizm filtrowania lub badania żądań HTTP wchodzących do Twojej aplikacji. Na przykład Laravel zawiera middleware, które weryfikuje, czy użytkownik aplikacji jest uwierzytelniony. Jeśli użytkownik nie jest uwierzytelniony, middleware przekieruje użytkownika do ekranu logowania. Jeśli jednak użytkownik jest uwierzytelniony, middleware pozwoli żądaniu przejść dalej do aplikacji. Niektóre middleware są przypisane do wszystkich tras w aplikacji, takie jak `PreventRequestsDuringMaintenance`, podczas gdy niektóre są przypisane tylko do określonych tras lub grup tras. Możesz dowiedzieć się więcej o middleware, czytając pełną [dokumentację middleware](/docs/{{version}}/middleware).

Jeśli żądanie przejdzie przez wszystkie przypisane middleware dopasowanej trasy, zostanie wykonana metoda trasy lub kontrolera, a odpowiedź zwrócona przez metodę trasy lub kontrolera zostanie wysłana z powrotem przez łańcuch middleware trasy.

<a name="finishing-up"></a>
### Zakończenie

Gdy metoda trasy lub kontrolera zwróci odpowiedź, odpowiedź będzie podróżować z powrotem na zewnątrz przez middleware trasy, dając aplikacji szansę na modyfikację lub zbadanie wychodzącej odpowiedzi.

Na koniec, gdy odpowiedź przejdzie z powrotem przez middleware, metoda `handle` jądra HTTP zwraca obiekt odpowiedzi do `handleRequest` instancji aplikacji, a ta metoda wywołuje metodę `send` na zwróconej odpowiedzi. Metoda `send` wysyła zawartość odpowiedzi do przeglądarki użytkownika. Ukończyliśmy właśnie naszą podróż przez cały cykl życia żądania Laravel!

<a name="focus-on-service-providers"></a>
## Skupienie na dostawcach usług

Dostawcy usług są naprawdę kluczem do uruchomienia aplikacji Laravel. Instancja aplikacji jest tworzona, dostawcy usług są rejestrowani, a żądanie jest przekazywane do uruchomionej aplikacji. To naprawdę takie proste!

Posiadanie mocnego zrozumienia tego, jak aplikacja Laravel jest budowana i uruchamiana za pośrednictwem dostawców usług, jest bardzo cenne. Dostawcy usług zdefiniowani przez użytkownika w Twojej aplikacji są przechowywani w katalogu `app/Providers`.

Domyślnie `AppServiceProvider` jest dość pusty. Ten dostawca to świetne miejsce do dodania własnego uruchamiania aplikacji i powiązań kontenera usług. W przypadku dużych aplikacji możesz chcieć utworzyć kilku dostawców usług, z których każdy będzie miał bardziej szczegółowe uruchamianie dla konkretnych usług używanych przez Twoją aplikację.
