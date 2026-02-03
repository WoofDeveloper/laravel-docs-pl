# 🚀 Instrukcja publikacji na GitHub

## Krok 1: Utwórz repozytorium na GitHub

1. Zaloguj się na [GitHub.com](https://github.com)
2. Kliknij przycisk "+" w prawym górnym rogu i wybierz "New repository"
3. Wypełnij formularz:
   - **Repository name**: `laravel-docs-pl` (lub inną wybraną nazwę)
   - **Description**: `📘 Nieoficjalne polskie tłumaczenie dokumentacji Laravel 12.x - kompletny przewodnik po najpopularniejszym frameworku PHP dla polskich deweloperów`
   - **Public/Private**: Wybierz **Public**
   - **NIE zaznaczaj** żadnych opcji (README, .gitignore, license) - mamy już te pliki
4. Kliknij "Create repository"

## Krok 2: Połącz lokalne repozytorium z GitHub

GitHub wyświetli instrukcje. Użyj tych dla **"…or push an existing repository from the command line"**:

```powershell
cd C:\Users\Kacper\source\repos\docs-12.x
git remote add origin https://github.com/TWOJA-NAZWA/laravel-docs-pl.git
git branch -M main
git push -u origin main
```

**Zamień `TWOJA-NAZWA`** na swoją nazwę użytkownika GitHub!

## Krok 3: Skonfiguruj repozytorium na GitHub

### 3.1 Dodaj opis i tagi

1. Przejdź do swojego repozytorium na GitHub
2. Kliknij ikonę koła zębatego (⚙️) obok "About" po prawej stronie
3. Dodaj:
   - **Description**: `📘 Nieoficjalne polskie tłumaczenie dokumentacji Laravel 12.x`
   - **Website** (opcjonalnie): `https://laravel.com`
   - **Topics**: Dodaj tagi, po jednym na raz:
     ```
     laravel
     laravel-12
     php
     dokumentacja
     polish
     translation
     polska
     laravel-docs
     php-framework
     web-development
     ```

### 3.2 Włącz Issues i Discussions

1. Przejdź do **Settings** (Ustawienia)
2. W sekcji **Features** zaznacz:
   - ✅ Issues
   - ✅ Discussions (jeśli chcesz umożliwić dyskusje)

## Krok 4: (Opcjonalnie) Włącz GitHub Pages

Aby udostępnić dokumentację jako stronę internetową:

1. Przejdź do **Settings** → **Pages**
2. W sekcji **Source** wybierz:
   - Branch: `main`
   - Folder: `/ (root)`
3. Kliknij **Save**
4. Po kilku minutach dokumentacja będzie dostępna pod:
   `https://TWOJA-NAZWA.github.io/laravel-docs-pl/`

## Krok 5: Dodaj logo i banner (opcjonalnie)

### Przygotuj social preview:

1. Stwórz obrazek 1280x640px z logo Laravel i napisem "Laravel 12.x - Dokumentacja PL"
2. Przejdź do **Settings** → **Options** → **Social preview**
3. Kliknij **Edit** i prześlij obrazek

## Krok 6: Udostępnij społeczności!

Podziel się linkiem do repozytorium:
- 🇵🇱 Polska społeczność Laravel na Facebook
- 💬 Discord/Slack dla programistów PHP
- 🐦 Twitter/X z hashtag #Laravel #PHP #PolskieIT
- 💼 LinkedIn

---

## 📝 Przydatne komendy Git

### Aktualizacja repozytorium po zmianach:

```powershell
cd C:\Users\Kacper\source\repos\docs-12.x

# Sprawdź status
git status

# Dodaj zmiany
git add .

# Commituj
git commit -m "Opis zmian"

# Wypchnij na GitHub
git push
```

### Tworzenie branch dla nowych funkcji:

```powershell
# Stwórz nowy branch
git checkout -b nowa-funkcja

# Wprowadź zmiany...
git add .
git commit -m "Dodanie nowej funkcji"

# Wypchnij branch
git push -u origin nowa-funkcja
```

Następnie na GitHub stwórz Pull Request!

---

## 🎉 Gotowe!

Twoje repozytorium jest teraz dostępne publicznie na GitHub. Gratulacje! 🎊

**Następne kroki:**
1. Dodaj badge ze statusem do README
2. Zachęć innych do współpracy (Issues, Pull Requests)
3. Promuj projekt w społeczności
4. Regularnie aktualizuj tłumaczenie

---

## 🆘 Potrzebujesz pomocy?

- [Dokumentacja GitHub](https://docs.github.com)
- [Podstawy Git](https://git-scm.com/book/pl/v2)
- [GitHub Desktop](https://desktop.github.com/) - alternatywa dla wiersza poleceń
