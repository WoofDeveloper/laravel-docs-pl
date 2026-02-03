# Laravel Pint

- [Wprowadzenie](#introduction)
- [Instalacja](#installation)
- [Uruchamianie Pint](#running-pint)
- [Konfiguracja Pint](#configuring-pint)
    - [Presety](#presets)
    - [Reguły](#rules)
    - [Wykluczanie Plików / Folderów](#excluding-files-or-folders)
- [Ciągła Integracja](#continuous-integration)
    - [GitHub Actions](#running-tests-on-github-actions)

<a name="introduction"></a>
## Wprowadzenie

[Laravel Pint](https://github.com/laravel/pint) to opiniowany narzędzie do naprawiania stylu kodu PHP dla minimalistów. Pint jest zbudowany na bazie [PHP CS Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer) i ułatwia zapewnienie, że Twój styl kodu pozostaje czysty i spójny.

Pint jest automatycznie instalowany wraz ze wszystkimi nowymi aplikacjami Laravel, więc możesz zacząć go używać natychmiast. Domyślnie Pint nie wymaga żadnej konfiguracji i naprawi problemy ze stylem kodu w Twoim kodzie, zgodnie z opiniotwórczym stylem kodowania Laravel.

<a name="installation"></a>
## Instalacja

Pint jest dołączony do najnowszych wydań frameworka Laravel, więc instalacja jest zazwyczaj niepotrzebna. Jednak dla starszych aplikacji możesz zainstalować Laravel Pint za pomocą Composer:

```shell
composer require laravel/pint --dev
```

<a name="running-pint"></a>
## Uruchamianie Pint

Możesz poinstruować Pint, aby naprawił problemy ze stylem kodu, wywołując plik binarny `pint`, który jest dostępny w katalogu `vendor/bin` Twojego projektu:

```shell
./vendor/bin/pint
```

Jeśli chcesz, aby Pint działał w trybie równoległym (eksperymentalnym) dla lepszej wydajności, możesz użyć opcji `--parallel`:

```shell
./vendor/bin/pint --parallel
```

Tryb równoległy pozwala również określić maksymalną liczbę procesów do uruchomienia za pomocą opcji `--max-processes`. Jeśli ta opcja nie jest podana, Pint użyje każdego dostępnego rdzenia na Twojej maszynie:

```shell
./vendor/bin/pint --parallel --max-processes=4
```

Możesz również uruchomić Pint na konkretnych plikach lub katalogach:

```shell
./vendor/bin/pint app/Models

./vendor/bin/pint app/Models/User.php
```

Pint wyświetli dokładną listę wszystkich plików, które aktualizuje. Możesz zobaczyć jeszcze więcej szczegółów o zmianach Pint, podając opcję `-v` podczas wywoływania Pint:

```shell
./vendor/bin/pint -v
```

Jeśli chcesz, aby Pint po prostu sprawdził Twój kod pod kątem błędów stylu bez faktycznej zmiany plików, możesz użyć opcji `--test`. Pint zwróci kod wyjścia różny od zera, jeśli zostaną znalezione jakiekolwiek błędy stylu kodu:

```shell
./vendor/bin/pint --test
```

Jeśli chcesz, aby Pint modyfikował tylko pliki, które różnią się od podanej gałęzi według Git, możesz użyć opcji `--diff=[branch]`. Może to być skutecznie wykorzystane w środowisku CI (jak GitHub Actions) do oszczędzenia czasu przez sprawdzanie tylko nowych lub zmodyfikowanych plików:

```shell
./vendor/bin/pint --diff=main
```

Jeśli chcesz, aby Pint modyfikował tylko pliki, które mają niezatwierdzone zmiany według Git, możesz użyć opcji `--dirty`:

```shell
./vendor/bin/pint --dirty
```

Jeśli chcesz, aby Pint naprawił wszelkie pliki z błędami stylu kodu, ale także wyszedł z kodem wyjścia różnym od zera, jeśli jakiekolwiek błędy zostały naprawione, możesz użyć opcji `--repair`:

```shell
./vendor/bin/pint --repair
```

<a name="configuring-pint"></a>
## Konfiguracja Pint

Jak wspomniano wcześniej, Pint nie wymaga żadnej konfiguracji. Jednak jeśli chcesz dostosować presety, reguły lub sprawdzane foldery, możesz to zrobić, tworząc plik `pint.json` w głównym katalogu projektu:

```json
{
    "preset": "laravel"
}
```

Dodatkowo, jeśli chcesz użyć `pint.json` z konkretnego katalogu, możesz podać opcję `--config` podczas wywoływania Pint:

```shell
./vendor/bin/pint --config vendor/my-company/coding-style/pint.json
```

<a name="presets"></a>
### Presety

Presety definiują zestaw reguł, które mogą być użyte do naprawiania problemów ze stylem kodu w Twoim kodzie. Domyślnie Pint używa presetu `laravel`, który naprawia problemy zgodnie z opinioteórczym stylem kodowania Laravel. Jednak możesz określić inny preset, podając opcję `--preset` do Pint:

```shell
./vendor/bin/pint --preset psr12
```

Jeśli chcesz, możesz również ustawić preset w pliku `pint.json` Twojego projektu:

```json
{
    "preset": "psr12"
}
```

Obecnie obsługiwane presety Pinta to: `laravel`, `per`, `psr12`, `symfony` i `empty`.

<a name="rules"></a>
### Reguły

Reguły to wytyczne stylistyczne, których Pint będzie używał do naprawiania problemów ze stylem kodu w Twoim kodzie. Jak wspomniano powyżej, presety to predefiniowane grupy reguł, które powinny być idealne dla większości projektów PHP, więc zazwyczaj nie musisz się martwić o poszczególne reguły, które zawierają.

Jednak jeśli chcesz, możesz włączyć lub wyłączyć określone reguły w pliku `pint.json` lub użyć presetu `empty` i zdefiniować reguły od zera:

```json
{
    "preset": "laravel",
    "rules": {
        "simplified_null_return": true,
        "array_indentation": false,
        "new_with_parentheses": {
            "anonymous_class": true,
            "named_class": true
        }
    }
}
```

Pint jest zbudowany na bazie [PHP CS Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer). Dlatego możesz użyć dowolnej z jego reguł do naprawiania problemów ze stylem kodu w Twoim projekcie: [PHP CS Fixer Configurator](https://mlocati.github.io/php-cs-fixer-configurator).

<a name="excluding-files-or-folders"></a>
### Wykluczanie Plików / Folderów

Domyślnie Pint sprawdzi wszystkie pliki `.php` w Twoim projekcie z wyjątkiem tych w katalogu `vendor`. Jeśli chcesz wykluczyć więcej folderów, możesz to zrobić za pomocą opcji konfiguracyjnej `exclude`:

```json
{
    "exclude": [
        "my-specific/folder"
    ]
}
```

Jeśli chcesz wykluczyć wszystkie pliki, które zawierają dany wzór nazwy, możesz to zrobić za pomocą opcji konfiguracyjnej `notName`:

```json
{
    "notName": [
        "*-my-file.php"
    ]
}
```

Jeśli chcesz wykluczyć plik, podając dokładną ścieżkę do pliku, możesz to zrobić za pomocą opcji konfiguracyjnej `notPath`:

```json
{
    "notPath": [
        "path/to/excluded-file.php"
    ]
}
```

<a name="continuous-integration"></a>
## Ciągła Integracja

<a name="running-tests-on-github-actions"></a>
### GitHub Actions

Aby zautomatyzować lintowanie projektu za pomocą Laravel Pint, możesz skonfigurować [GitHub Actions](https://github.com/features/actions), aby uruchamiać Pint za każdym razem, gdy nowy kod jest wypychany do GitHuba. Najpierw upewnij się, że przyznałeś "Read and write permissions" dla workflow w GitHub pod **Settings > Actions > General > Workflow permissions**. Następnie utwórz plik `.github/workflows/lint.yml` z następującą treścią:

```yaml
name: Fix Code Style

on: [push]

jobs:
  lint:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: true
      matrix:
        php: [8.4]

    steps:
      - name: Checkout code
        uses: actions/checkout@v5

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php }}
          tools: pint

      - name: Run Pint
        run: pint

      - name: Commit linted files
        uses: stefanzweifel/git-auto-commit-action@v6
```
