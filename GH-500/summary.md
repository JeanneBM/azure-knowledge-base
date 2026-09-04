# GitHub Advanced Security (GHAS) 

---

## 1. Co to jest GHAS?

**GitHub Advanced Security (GHAS)** – rozwiązanie zabezpieczeń aplikacji wbudowane bezpośrednio w przepływ pracy deweloperskiej. Ułatwia zabezpieczanie kodu i zarządzanie ryzykiem **bez zakłócania** i spowalniania programowania.

### Dwa główne produkty GHAS
- **GitHub Code Security** – wykrywanie i usuwanie luk w kodzie oraz zależnościach
- **GitHub Secret Protection** – wykrywanie i zapobieganie wyciekom sekretów

### Trzy kluczowe obszary rozwiązań

| Obszar rozwiązania       | Główny fokus                              | Kluczowe możliwości                                      | Typowe przypadki użycia                          |
|--------------------------|-------------------------------------------|----------------------------------------------------------|--------------------------------------------------|
| **Code Security**        | Luki w kodzie źródłowym i zależnościach   | Code scanning, Copilot Autofix, Security campaigns, Dependency review | Wykrywanie i naprawianie niezabezpieczonych wzorców kodu |
| **Secret Protection**    | Ujawnione poświadczenia i tokeny          | Secret scanning, Push protection, Custom patterns, Security campaigns | Zapobieganie przypadkowym wyciekom sekretów     |
| **Supply Chain Security**| Ryzyka w zależnościach zewnętrznych       | Dependabot alerts, Security updates, Dependency graph, Dependency review | Zarządzanie podatnymi pakietami open source     |

**Dlaczego to ważne?**
- Gartner przewiduje, że do 2025 r. **45%** organizacji globalnych będzie miało wpływ ataku łańcucha dostaw.
- Verizon Data Breach Investigation 2022: aplikacje są głównym wektorem ataku i znajdują się w centrum **>40%** wszystkich naruszeń danych.

---

## 2. Trzy integralne funkcje GHAS

### A. Skanowanie sekretów (Secret Scanning)

- Identyfikuje i pomaga zapobiegać przypadkowemu narażeniu poufnych informacji (klucze API, tokeny, hasła, klucze prywatne).
- Działa na:
  - całej historii Git we wszystkich gałęziach
  - opisach i komentarzach w issues
  - tytułach, opisach i komentarzach w PR-ach (otwartych i zamkniętych)
  - dyskusjach GitHub
- **Push protection** – proaktywnie skanuje kod podczas push i **blokuje** commit zawierający znane sekrety.
- Dostępne **bezpłatnie** dla wszystkich publicznych repozytoriów.
- Dla prywatnych wymaga licencji GHAS.
- Można tworzyć **własne wzorce (custom patterns)**.
- Alerty przegląda się w: **Security → Secret scanning**.

### B. Skanowanie kodu (Code Scanning)

- Analizuje kod źródłowy pod kątem luk w zabezpieczeniach i błędów kodowania (SQL injection, XSS, buffer overflow itd.).
- Wykorzystuje technikę **analizy statycznej**.
- Główne narzędzie: **CodeQL**.
- Dostępne dla publicznych repo + prywatnych z GHAS.
- Alerty pojawiają się na karcie **Security** i zamykają się automatycznie po naprawie kodu.
- Można uruchamiać:
  - według harmonogramu (schedule)
  - przy push
  - przy pull_request
- Wspiera też narzędzia trzecie poprzez upload plików **SARIF 2.1.0**.

### C. Dependabot

Automatyczne narzędzie do zarządzania zależnościami.

**Składa się z:**
- **Alerty Dependabot** – informują, że kod zależy od niezabezpieczonego pakietu
- **Aktualizacje zabezpieczeń (Security updates)** – automatyczne PR-y aktualizujące zależności z znanymi lukami
- **Aktualizacje wersji (Version updates)** – automatyczne PR-y aktualizujące zależności nawet bez luk (na podstawie semver)

Dependabot ściśle współpracuje z **Dependency Graph** i **GitHub Advisory Database**.

---

## 3. Dependency Graph (Wykres zależności)

- Identyfikuje **wszystkie zależności nadrzędne i podrzędne** (bezpośrednie + pośrednie/transitive).
- Generowany automatycznie na podstawie:
  - plików manifestu
  - plików lockfile
  - danych przesłanych przez Dependency Submission API (beta)
- Automatycznie aktualizowany przy push do default branch.
- Podstawa działania:
  - Dependabot alerts
  - Security updates
  - Dependency review
  - SBOM

**Jak wyświetlić?**  
Repozytorium → **Insights → Dependency graph**

**Eksport:** można wyeksportować jako **SBOM** (format SPDX).

**Włączanie dla prywatnych repo:**  
Settings → Code security and analysis → Dependency graph → Enable

> **Ważne:** Lockfile’y dają najdokładniejszy graf (dokładne wersje bezpośrednich i pośrednich zależności).

---

## 4. Dependabot – szczegóły

### Kiedy generowane są alerty?
1. Nowy advisory zostaje dodany do **GitHub Advisory Database**
2. Zmienia się **Dependency Graph** repozytorium
3. Próba merge zależności z luką do default branch

### Włączanie alertów
- Nie są włączone domyślnie.
- Można włączyć na poziomie: repozytorium / organizacji / enterprise.
- Wymaga włączonego Dependency Graph.

### Dostęp do alertów
| Typ alertu              | Minimalna rola          |
|-------------------------|-------------------------|
| Code scanning + Dependabot | **Write**              |
| Secret scanning         | **Admin**               |
| Dodatkowy dostęp        | Można nadać indywidualnie/zespołom niezależnie od roli (Settings → Access to alerts) |

### Security updates vs Version updates
- **Security updates** → tylko zależności z znanymi lukami
- **Version updates** → wszystkie zależności (nawet bez luk), oparte na semantic versioning

Scalenie PR-a z Security update automatycznie zamyka powiązany alert Dependabot.

---

## 5. Dependency Review

- Pokazuje w każdym **Pull Request**:
  - które zależności zostały dodane / usunięte / zaktualizowane
  - które projekty z nich korzystają
  - jakie luki bezpieczeństwa dotyczą nowych wersji
- Umożliwia **shift-left** – przechwytywanie podatnych zależności **przed** trafieniem do produkcji.
- Część GHAS (dla prywatnych repo wymaga licencji).

---

## 6. CodeQL – fundament Code Scanning

### Jak działa analiza CodeQL? (3 kroki)
1. **Przygotowanie bazy danych** – CodeQL wyodrębnia relacyjną reprezentację kodu źródłowego
2. **Uruchomienie zapytań** – standardowe zapytania + własne
3. **Interpretacja wyników** – generowanie alertów (i ścieżek)

### Typy zapytań
- **Alert queries** – podkreślają problemy w konkretnych lokalizacjach kodu
- **Path queries** – opisują przepływ informacji od źródła (source) do ujścia (sink)

### Sposoby konfiguracji
1. **Default setup** – szybkie włączenie, automatyczny wybór języków i zapytań
2. **Advanced setup** – własny workflow YAML z akcją `github/codeql-action`
3. **CodeQL CLI** – lokalnie lub w zewnętrznym CI + upload wyników SARIF

### Obsługiwane języki (główne)
C/C++, C#, Go, Java, JavaScript/TypeScript, Python, Ruby

### Optymalizacja czasów analizy
- Używaj **matrix** dla wielu języków (równoległe uruchomienia)
- Wykluczaj kod testowy
- Zwiększ pamięć/rdzenie na self-hosted runners
- Przy bardzo dużych repo uruchamiaj tylko na `schedule`

### Copilot Autofix
Po wykryciu problemu CodeQL generuje **sugerowane poprawki kodu** wraz z wyjaśnieniem.

---

## 7. Dostępność funkcji

| Funkcja               | Publiczne repo | Prywatne bez GHAS | Prywatne z GHAS |
|-----------------------|----------------|-------------------|-----------------|
| Code scanning         | ✅             | ❌                | ✅              |
| Secret scanning       | ✅             | ❌                | ✅              |
| Dependency review     | ✅             | ❌                | ✅              |
| Dependency graph      | ✅             | ✅ (włączane)     | ✅              |
| Dependabot alerts     | ✅ (włączane)  | ✅ (włączane)     | ✅              |

---

## 8. Administracja i zarządzanie

### Włączanie GHAS
- Na poziomie **organizacji** (Enterprise Cloud) → automatycznie dla wszystkich prywatnych i internal repo.
- Można też włączać indywidualnie na poziomie repozytorium.

### Security Overview
- Dostępny na poziomie organizacji, zespołu i repozytorium.
- Agregowany widok stanu bezpieczeństwa + filtrowanie według funkcji.

### Role i korzyści
- **Developer** – pisze kod, dostaje alerty w PR-ach, naprawia
- **Security Engineer** – monitoruje alerty, priorytetyzuje, tworzy Security Campaigns
- **Administrator** – włącza funkcje, konfiguruje zasady, zarządza licencjami i pokryciem

### Repository Rulesets
Nowoczesna, warstwowa alternatywa dla tradycyjnych Branch Protection Rules:
- Grupowanie wielu reguł pod jedną nazwą
- Targetowanie konkretnych gałęzi/tagów (np. `feature/*`, `v*.*`)
- Możliwość bypass dla adminów/zespołów/aplikacji
- Łatwe włączanie/wyłączanie bez usuwania

### Audit Log
- Rejestruje kto, co i kiedy zrobił (usunięcie repo, uruchomienie workflow itd.)
- Dostępny przez UI, GraphQL API i REST API

---

## 9. Kluczowe zasady i best practices

1. **Bezpieczeństwo wbudowane w cały SDLC** – nie traktuj go jako osobnej bramy na końcu.
2. **Shift-left** – wykrywaj i naprawiaj jak najwcześniej (w PR-ach).
3. **Nie ignoruj alertów** – konsekwencje: naruszenia danych, straty reputacji, kary regulacyjne, większy koszt naprawy później.
4. Utrzymuj aktualne **manifesty i lockfile’y**.
5. Używaj **Security Campaigns** do masowego naprawiania długu bezpieczeństwa.
6. Włącz **Push protection** + custom patterns dla sekretów.
7. Zawsze testuj PR-y Dependabot przed merge (szczególnie Version updates).
8. Korzystaj z **Dependency Review** w każdym PR dotyczącym zależności.

---

## 10. Szybkie pytania egzaminacyjne (must-know)

1. Co to jest GHAS i z jakich dwóch produktów obecnie się składa?
2. Wymień trzy główne obszary rozwiązań GHAS.
3. Czym różnią się Security updates od Version updates w Dependabot?
4. Na czym opierają się alerty Dependabot?
5. Co skanuje Secret scanning poza kodem źródłowym?
6. Jakie są 3 kroki analizy CodeQL?
7. Kto widzi alerty Code scanning/Dependabot, a kto Secret scanning?
8. Co to jest Dependency Review i kiedy się pojawia?
9. Jak włączyć GHAS na poziomie organizacji?
10. Co to są Repository Rulesets i czym przewyższają klasyczne Branch Protection?

---
