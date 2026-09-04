# CodeQL, workflow i administracja GHAS

> Zakres: sposób działania CodeQL i QL, konfiguracja code scanning, CodeQL CLI oraz zarządzanie GitHub Advanced Security, zasadami, regułami i audytem.

## 1. CodeQL — trzy fazy analizy

Analiza CodeQL zawsze przebiega w trzech etapach:

1. **Utworzenie bazy danych CodeQL** z kodu źródłowego.
2. **Uruchomienie zapytań QL** względem tej bazy.
3. **Interpretacja wyników** i przedstawienie ich w kontekście kodu.

### Tworzenie bazy danych

- Dla **języków kompilowanych** CodeQL monitoruje zwykły proces kompilacji. Zbiera m.in. AST, wiązanie nazw i informacje o typach.
- Dla **języków interpretowanych** ekstraktor działa bezpośrednio na kodzie źródłowym i rozpoznaje zależności.
- Każdy obsługiwany język ma własny ekstraktor i własną bazę danych.
- Baza zawiera dane relacyjne, kopię plików źródłowych oraz schemat specyficzny dla języka.

### Uruchamianie i interpretowanie zapytań

- Zapytania są pisane w obiektowym, deklaratywnym języku **QL**.
- Można je uruchamiać przez rozszerzenie CodeQL dla VS Code albo CodeQL CLI.
- Metadane zapytania mówią, jak zinterpretować wynik:
  - prosty komunikat w jednej lokalizacji kodu;
  - ścieżka przepływu danych lub sterowania, czyli kilka lokalizacji i opis.
- Zapytanie **bez metadanych** zwraca tabelę danych, ale nie jest wyświetlane jako problem w kodzie.

## 2. QL — czym jest i czym nie jest

**QL** to deklaratywny, zorientowany obiektowo język zapytań zoptymalizowany do analizy hierarchicznych danych — zwłaszcza baz reprezentujących kod.

| Właściwość | Znaczenie |
|---|---|
| Deklaratywność | Opisujesz, **jakie warunki** ma spełniać wynik, a nie procedurę jego wyznaczania. |
| Logiczna semantyka | QL jest oparty na Datalogu; operacje są logiczne. |
| Rekursja i agregacja | Umożliwiają zwięzłe pytania o relacje, np. wszystkich potomków i ich liczbę. |
| Orientacja obiektowa | Klasy są predykatami opisującymi zbiory istniejących wartości, a dziedziczenie jest implikacją. |
| Praca na zbiorach | QL działa na zestawach krotek, nie na pojedynczych obiektach w pamięci. |

### QL a języki ogólnego przeznaczenia

- QL nie ma imperatywnych przypisań zmiennych ani operacji systemu plików.
- Nie tworzysz instancji przez alokowanie pamięci; klasa opisuje logiczną właściwość zbioru już istniejących wartości.
- Język przypomina składnią SQL, ale jego semantyka jest oparta na Datalogu.

## 3. Skanowanie kodu z CodeQL — trzy konfiguracje

| Podejście | Kiedy wybrać | Charakterystyka |
|---|---|---|
| **Domyślna konfiguracja** | Chcesz szybko zacząć | GitHub automatycznie wybiera języki, zestaw zapytań i wyzwalacze; można je też zmienić ręcznie. |
| **Zaawansowana konfiguracja** | Potrzebujesz kontroli | W repozytorium powstaje edytowalny workflow `codeql.yml` korzystający z `github/codeql-action`. |
| **CodeQL CLI w zewnętrznym CI** | Używasz własnego systemu CI | CLI analizuje kod poza GitHub Actions, a wyniki SARIF są przekazywane do GitHub. |

W zaawansowanej konfiguracji domyślny workflow zwykle skanuje przy `push` do gałęzi domyślnej/chronionych i przy `pull_request` względem gałęzi domyślnej.

### Masowe włączanie

Skanowanie CodeQL można wdrożyć w wielu repozytoriach skryptem, który tworzy pull requesty dodające workflow GitHub Actions. To podejście pomaga ujednolicić konfigurację organizacji.

## 4. Dostosowanie workflow CodeQL — dodatkowe zapytania

Analiza uruchamia standardowy zestaw zapytań, ale można dodać własne.

| Opcja w `init` | Znaczenie |
|---|---|
| `packs` | Pobiera i uruchamia pakiety zapytań CodeQL. |
| `queries` | Wskazuje pojedynczy plik `.ql`, katalog zapytań, zestaw `.qls` albo kombinację. |
| `config-file` | Wczytuje osobny YAML z konfiguracją analizy. |

Przykład:

```yaml
- uses: github/codeql-action/init@v3
  with:
    queries: security-extended
    external-repository-token: ${{ secrets.ACCESS_TOKEN }}
```

### Pakiety i zestawy zapytań

- Pakiety CodeQL opisane w materiale są funkcją beta.
- Gdy nie podasz wersji pakietu, pobierana jest najnowsza wersja.
- Dla prywatnego pakietu potrzebny jest token z dostępem do pakietu.
- Nie należy odwoływać się bezpośrednio do zestawów z repozytorium `github/codeql` na gałęzi `main`, ponieważ wersje mogą nie być zgodne z użytym silnikiem CodeQL.

| Zestaw | Zawartość i kompromis |
|---|---|
| `security-extended` | Domyślne zapytania + zapytania o niższej precyzji/ważności; szersze wykrywanie, możliwie więcej false positives. |
| `security-and-quality` | `security-extended` + zapytania dotyczące jakości, konserwacji i niezawodności. |

### Workflow i `config-file` razem

Jeśli workflow i plik konfiguracji określają dodatkowe `packs` lub `queries`, wartości z workflow **zastępują** wartości z pliku. Aby je **połączyć**, poprzedź wartość znakiem `+`.

```yaml
- uses: github/codeql-action/init@v3
  with:
    config-file: ./.github/codeql/codeql-config.yml
    queries: +security-and-quality,octo-org/python-qlpack/show_ifs.ql@main
```

> Dla workflow budujących bazy dla wielu języków pakiety zapytań należy określać w pliku konfiguracji, a nie bezpośrednio w workflow.

## 5. Niestandardowy plik konfiguracji CodeQL

`config-file` może znajdować się w analizowanym repozytorium albo w zewnętrznym repozytorium. Centralny plik zewnętrzny pozwala utrzymywać jedną konfigurację dla wielu repozytoriów.

```yaml
- uses: github/codeql-action/init@v3
  with:
    config-file: ./.github/codeql/codeql-config.yml
```

Dla prywatnego repozytorium z konfiguracją lub zapytaniami użyj `external-repository-token`.

### Co może zawierać konfiguracja YAML?

| Klucz | Zastosowanie |
|---|---|
| `packs` | Lista pakietów lub mapa pakietów przypisanych do języków. |
| `queries` | Zapytania, katalogi, pakiety lub pliki `.qls`. |
| `disable-default-queries: true` | Wyłącza domyślne zapytania — użyteczne, gdy uruchamiasz wyłącznie własne. |
| `query-filters` | Filtry `include` i `exclude` dla reguł. |
| `paths` | Katalogi do skanowania w Pythonie, Ruby i JavaScript/TypeScript. |
| `paths-ignore` | Katalogi/pliki wyłączone ze skanowania. |

Przykład ograniczenia zakresu:

```yaml
paths:
  - src
paths-ignore:
  - src/node_modules
  - '**/*.test.js'
```

### Filtry zapytań — pułapka

- Identyfikator zapytania znajdziesz w szczegółach alertu jako **Rule ID**.
- Kolejność filtrów ma znaczenie.
- Pierwszy filtr po definicjach zapytań/pakietów ustala, czy zapytania są początkowo włączone czy wyłączone.
- Kolejne filtry są wykonywane po kolei; późniejsze mają pierwszeństwo przed wcześniejszymi.

## 6. CodeQL CLI

CLI pozwala tworzyć bazy, analizować je i przekazywać wyniki do GitHub.

| Polecenie | Rola |
|---|---|
| `codeql database create` | Tworzy bazę CodeQL z kodu. |
| `codeql database analyze` | Uruchamia zapytania i generuje wynik, np. SARIF. |
| `codeql github upload-results` | Przesyła wyniki SARIF do GitHub. |

### Bezpieczne uwierzytelnianie

Najlepsza praktyka z materiału: używać magazynu sekretów i opcji `--github-auth-stdin`. Nie wpisuj tokenu bezpośrednio w argumentach polecenia ani w kodzie.

### Projekty wielojęzyczne

Twórz bazę dla każdego języka; w praktyce można używać bazy-klastra. Rezultatem są osobne podkatalogi baz dla języków.

## 7. Języki i kompilacja

- Dla JavaScript/TypeScript, Pythona i Ruby analiza opiera się na źródłach.
- Dla języków kompilowanych trzeba dodać poprawne, własne kroki builda do workflow.
- `autobuild` nie jest niezawodny dla każdego projektu; jawna kompilacja daje większą kontrolę.
- W repozytoriach wielojęzycznych macierz języków pozwala uruchamiać analizy równolegle.

## 8. GHAS — produkty i funkcje

W materiałach funkcje GHAS są oferowane przez dwa główne produkty:

| Produkt | Cel |
|---|---|
| **GitHub Code Security** | Wykrywanie i usuwanie podatności w kodzie. |
| **GitHub Secret Protection** | Wykrywanie i zapobieganie wyciekom sekretów. |

Najważniejsze możliwości:

- Code scanning / CodeQL i poprawki wspomagane przez Copilot Autofix;
- Secret scanning, push protection i custom patterns;
- Dependency review przed scaleniem;
- Security overview na poziomie organizacji;
- Security campaigns do grupowania i systematycznego usuwania wielu alertów.

Repozytoria publiczne zachowują szeroki zestaw bezpłatnych mechanizmów. Funkcje zaawansowane kładą nacisk na prewencję, automatyzację i szybsze usuwanie problemów.

## 9. Włączanie i dostęp do GHAS

### Włączanie

- W Enterprise Cloud GHAS można włączyć na poziomie organizacji/repozytorium, jeśli dostępne są licencje.
- Brak dostępnych licencji oznacza, że GHAS pozostaje wyłączone.
- W Enterprise Server należy najpierw spełnić wymagania wstępne funkcji; włączenie jest możliwe przez UI lub powłokę administracyjną.

### Dostęp i odpowiedzialność

Uprawnienia do alertów muszą być nadawane zgodnie z zasadą najmniejszych uprawnień. Role powinny odzwierciedlać obowiązki:

| Alert | Przykładowe działanie zespołu |
|---|---|
| Code scanning | Poprawa kodu, odrzucenie false positive lub usunięcie alertu. |
| Secret scanning | Usunięcie ujawnionego sekretu, utworzenie nowego tokenu, aktualizacja usług. |
| Dependabot | Aktualizacja podatnej zależności lub uzasadnione odrzucenie alertu. |

Zasady można ustawiać na poziomie organizacji i repozytorium. Ustawienia przedsiębiorstwa mogą zastępować ustawienia organizacji; zablokowanych ustawień nie zmieni właściciel organizacji.

## 10. Security overview, alerty i `GITHUB_TOKEN`

### Security overview

| Poziom | Co daje |
|---|---|
| Organizacja | Zagregowany obraz bezpieczeństwa i dane per repozytorium. |
| Zespół | Dane dla repozytoriów, do których zespół ma uprawnienia. |
| Repozytorium | Stan włączonych funkcji i opcje ich konfiguracji. |

Można dzięki niemu śledzić wdrożenie funkcji, filtrować alerty według ważności/repozytorium/typu/czasu/właściciela/stanu, mierzyć postęp napraw i monitorować obejścia zasad.

Trzy perspektywy przeglądu bezpieczeństwa:

- **wykrywanie** — co zostało znalezione;
- **naprawa** — co zostało poprawione, np. w PR lub przez Autofix;
- **zapobieganie** — co zostało zatrzymane, np. przez push protection i dependency review.

### `GITHUB_TOKEN`

- Ustaw uprawnienia tokenu na możliwie najmniejsze.
- Gdy domyślne uprawnienia są zbyt restrykcyjne, zwiększ je tylko dla konkretnego workflow.
- Gdy są zbyt szerokie, usuń zbędne uprawnienia.

## 11. Zasady bezpieczeństwa i dokumentacja

Zasady bezpieczeństwa wspierają bezpieczne, powtarzalne procesy, przejrzyste zgłaszanie luk oraz kontrolę dostępu zgodną z zasadą najmniejszych uprawnień.

### Ważne pliki repozytorium

| Plik | Znaczenie |
|---|---|
| `SECURITY.md` | Jak zgłaszać podatności i które wersje są wspierane. |
| `CODE_OF_CONDUCT.md` | Zasady zachowania społeczności. |
| `CONTRIBUTING.md` | Wytyczne wkładu. |
| `GOVERNANCE.md` | Sposób podejmowania decyzji. |
| `SUPPORT.md` | Kanały wsparcia. |
| Szablony issue/PR | Standaryzują zgłoszenia i wkład. |

### Usuwanie poufnych danych z historii

1. Najpierw **unieważnij/obróć sekret** — samo przepisanie historii nie cofa wycieku.
2. Usuń dane z historii (materiał wskazuje m.in. BFG Repo-Cleaner jako zalecane narzędzie).
3. Wyczyść reflog/Git GC i wykonaj wymuszony push po uzgodnieniu z zespołem.
4. Dla danych publicznych lub krytycznych poproś GitHub Support o usunięcie danych z pamięci podręcznych po przepisaniu historii.
5. Zapobiegaj kolejnym wyciekom: `.gitignore`, GitHub Secrets/menedżer sekretów i regularne skanowanie.

> Przepisywanie historii i force push są operacjami wysokiego ryzyka — wymagają koordynacji z zespołem.

## 12. Repository rulesets

**Ruleset** to nazwana kolekcja reguł stosowana do wybranych gałęzi lub tagów.

| Element | Przykłady |
|---|---|
| Elementy docelowe | `feature/*`, `release/*`, tagi `v*.*` |
| Reguły | Wymagane status checks, podpisane commity, blokada force push/usuwania, ograniczenie merge/push. |
| Bypass permissions | Uprawnione osoby, zespoły lub aplikacje mogą obejść określoną regułę. |

### Rulesets vs branch protection

| Cecha | Klasyczna ochrona gałęzi/tagów | Rulesets |
|---|---|---|
| Wiele grup reguł współistnieje | Nie | Tak |
| Włączenie/wyłączenie bez usuwania | Nie | Tak |
| Widoczne dla osób z `Read` | Nie, zwykle tylko administrator | Tak |
| Tryb oceny/testowania | Nie | Tak |

**Warstwowanie:** GitHub zbiera wszystkie mające zastosowanie branch protections, tag protections i rulesets, a następnie wymusza **najbardziej restrykcyjne** ustawienie.

Przykład: ruleset wymaga 3 review i podpisanych commitów, a branch protection — historii liniowej i 2 review. Wynik: wymagane są 3 review, podpisy i historia liniowa.

### Typowe wymagania

- pozytywne status checks, np. CodeQL i Dependency Review;
- podpisane commity;
- blokada force push oraz usuwania;
- ograniczenie, kto może wprowadzać i scalać zmiany;
- kluczowe workflow CI/CD jako wymagane kontrole statusu przed merge.

**Kompromis:** więcej kontroli zwiększa bezpieczeństwo i zgodność, ale może spowalniać wdrożenia i frustrować zespoły. Dobieraj zasady do krytyczności gałęzi i wymogów organizacji.

## 13. Raportowanie i audit log

Audit log pozwala ustalić **kto, co, kiedy i gdzie** zrobił. Służy do incident response, zgodności i rozwiązywania problemów.

| Dane wpisu audytowego | Przykłady |
|---|---|
| Zasób | Repozytorium, którego dotyczy akcja. |
| Aktor | Użytkownik wykonujący akcję. |
| Akcja | Np. utworzenie, usunięcie, zmiana uprawnienia. |
| Lokalizacja | Kraj/region i źródłowy adres IP. |
| Czas | Data i godzina zdarzenia. |

### Retencja i API

| Kanał | Zakres z materiału | Ważne ograniczenie |
|---|---|---|
| Enterprise Cloud / REST | Do 90 dni; zdarzenia Git przez 7 dni | REST obejmuje m.in. zmiany ustawień, uprawnień, członkostw i zdarzenia Git. |
| Enterprise Server / GraphQL | Do 120 dni | GraphQL nie obejmuje zdarzeń Git, np. push/pull. |

Filtry w interfejsie obejmują m.in. `action`, `actor`, `repo`, `org` i `created`. Logi można eksportować jako CSV lub JSON.

### Badanie brakującego zasobu

1. Określ zdarzenie, np. `repository.deleted`.
2. Wyszukaj je w logu przez REST (`phrase=repository.deleted`) albo GraphQL.
3. Sprawdź metadane: aktora, czas i nazwę zasobu.
4. Podejmij korygowanie: odzyskaj kopię zapasową lub przywróć dostęp.

### Streamowanie do SIEM

Audit log można przesyłać w czasie rzeczywistym do SIEM, np. Splunk albo Datadog, z miejscami docelowymi takimi jak AWS S3 czy Azure Event Hubs. Daje to dłuższą retencję i lepsze wykrywanie zdarzeń.

## 14. Powtórka w 60 sekund

- CodeQL: **baza danych → zapytania QL → interpretacja wyniku**.
- QL jest deklaratywny, logiczny, obiektowy i pracuje na zbiorach krotek.
- Code scanning: default setup, advanced workflow albo CLI w zewnętrznym CI.
- `packs` pobiera pakiety zapytań; `queries` wskazuje `.ql`, katalog lub `.qls`.
- `+` przy `packs`/`queries` łączy workflow z `config-file`; bez `+` workflow zastępuje konfigurację.
- `disable-default-queries: true` wyłącza standardowe zapytania.
- Dla języków kompilowanych preferuj jawne kroki builda, gdy autobuild nie wystarcza.
- GHAS obejmuje Code Security i Secret Protection.
- Security overview pokazuje wykrywanie, naprawę i zapobieganie.
- Najmniejsze uprawnienia `GITHUB_TOKEN` i RBAC to podstawowa praktyka.
- Rulesets warstwują się z ochroną gałęzi; obowiązuje najbardziej restrykcyjna reguła.
- Audit log odpowiada na pytania: kto, co, kiedy i skąd.

## 15. Pytania kontrolne

1. Jakie są trzy fazy analizy CodeQL?
   - Utworzenie bazy, wykonanie zapytań i interpretacja wyników.
2. Czym różni się QL od języka imperatywnego?
   - Opisuje logiczne warunki wyniku i pracuje na zbiorach; nie ma przypisań ani operacji plikowych.
3. Co zrobi `queries` w kroku `init`?
   - Uruchomi wskazane zapytanie, katalog zapytań lub zestaw `.qls` dodatkowo do domyślnego zestawu, o ile konfiguracja go nie wyłącza.
4. Jak łączyć dodatkowe zapytania z workflow i `config-file`?
   - Dodać prefiks `+` do wartości `queries` lub `packs` w workflow.
5. Kiedy potrzebujesz `external-repository-token`?
   - Gdy konfiguracja, zapytania lub pakiety są w prywatnym zewnętrznym repozytorium.
6. Dlaczego rulesets są bardziej elastyczne od pojedynczej branch protection?
   - Umożliwiają wiele współistniejących grup reguł, tryb oceny oraz włączanie/wyłączanie bez usuwania.
7. Które API nie obejmuje zdarzeń Git według materiału?
   - GraphQL audit log na Enterprise Server.


