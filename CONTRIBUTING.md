# 🤝 Wkład w projekt

Dziękujemy za zainteresowanie projektem tłumaczenia dokumentacji Laravel na język polski! Twój wkład jest bardzo cenny dla polskiej społeczności deweloperów.

## 📋 Jak pomóc?

Istnieje wiele sposobów, aby przyczynić się do rozwoju projektu:

### 🐛 Zgłaszanie błędów

Jeśli znalazłeś błąd w tłumaczeniu:

1. Sprawdź, czy błąd nie został już zgłoszony w [Issues](../../issues)
2. Otwórz nowe zgłoszenie opisujące:
   - Lokalizację błędu (plik i sekcja)
   - Obecne tłumaczenie
   - Proponowaną poprawkę
   - Uzasadnienie (jeśli potrzebne)

### ✨ Proponowanie poprawek

Możesz zaproponować poprawki poprzez Pull Request:

1. Zforkuj repozytorium
2. Stwórz nowy branch: `git checkout -b poprawa/nazwa-sekcji`
3. Wprowadź zmiany
4. Commituj z opisowym komunikatem: `git commit -m 'Poprawa tłumaczenia w sekcji routing'`
5. Wypchnij zmiany: `git push origin poprawa/nazwa-sekcji`
6. Otwórz Pull Request

### 📝 Tłumaczenie nowych sekcji

Jeśli chcesz przetłumaczyć nową sekcję lub zaktualizować istniejącą:

1. Sprawdź [Issues](../../issues), czy ktoś już nad tym nie pracuje
2. Opcjonalnie otwórz Issue informujące o zamiarze tłumaczenia
3. Pracuj nad tłumaczeniem zgodnie z wytycznymi poniżej
4. Wyślij Pull Request

## 📖 Zasady tłumaczenia

### Ogólne wytyczne

- **Zachowaj naturalność języka** - tłumacz w sposób zrozumiały dla polskich deweloperów
- **Spójność terminologii** - używaj ustalonych terminów (patrz słownik poniżej)
- **Nie tłumacz kodu** - kod źródłowy pozostaje w języku angielskim
- **Nie tłumacz nazw własnych** - nazwy klas, metod, zmiennych, pakietów pozostają oryginalne

### 🔤 Słownik terminów

| Angielski | Polski |
|-----------|--------|
| routing | routing (bez tłumaczenia) |
| controller | kontroler |
| middleware | middleware (bez tłumaczenia) |
| request | żądanie |
| response | odpowiedź |
| view | widok |
| model | model |
| migration | migracja |
| seeding | seedowanie / wypełnianie bazy |
| factory | fabryka |
| query | zapytanie |
| eloquent | eloquent (bez tłumaczenia) |
| blade | blade (bez tłumaczenia) |
| artisan | artisan (bez tłumaczenia) |
| queue | kolejka |
| job | zadanie |
| event | zdarzenie |
| listener | nasłuchiwacz |
| service provider | dostawca usług |
| container | kontener |
| facade | fasada |
| trait | cecha / trait |
| namespace | przestrzeń nazw |

### Formatowanie

- Zachowuj formatowanie Markdown z oryginału
- Nie zmieniaj struktury nagłówków
- Zachowaj wszystkie linki i odniesienia
- Komentarze w kodzie mogą pozostać po angielsku lub być przetłumaczone (opcjonalnie)

### Przykłady kodu

```php
// ✅ DOBRZE - kod bez zmian, komentarz opcjonalnie po polsku
Route::get('/users', function () {
    // Pobierz wszystkich użytkowników
    return User::all();
});
```

```php
// ❌ ŹLE - nie tłumaczymy kodu!
Trasa::pobierz('/uzytkownicy', function () {
    return Uzytkownik::wszystkie();
});
```

## 🔍 Proces weryfikacji

Pull Request będzie zweryfikowany pod kątem:

- Poprawności tłumaczenia
- Spójności z resztą dokumentacji
- Jakości językowej
- Zgodności z wytycznymi

## 💬 Komunikacja

- **Issues** - zgłaszanie błędów i propozycji
- **Pull Requests** - konkretne zmiany w dokumentacji
- **Discussions** - pytania i dyskusje ogólne

## ⚖️ Licencja

Uczestnicząc w projekcie, zgadzasz się na udostępnienie swojego wkładu na licencji MIT, zgodnie z licencją całego projektu.

## 🙏 Podziękowania

Dziękujemy wszystkim, którzy poświęcają swój czas na poprawę dokumentacji dla polskiej społeczności Laravel!

---

**Masz pytania?** Nie wahaj się zadać ich w [Discussions](../../discussions).
