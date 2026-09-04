# GitHub Advanced Security

> Zakres: GHAS, zarządzanie zależnościami, Dependabot, secret scanning, code scanning i CodeQL.

## 1. Mapa tematu

GitHub Advanced Security (GHAS) wbudowuje zabezpieczenia w codzienny cykl tworzenia oprogramowania. Celem jest wykrywanie problemów możliwie wcześnie — podczas pracy z kodem, pull requestów i CI/CD.

| Obszar | Chronione ryzyko | Najważniejsze funkcje |
|---|---|---|
| GitHub Code Security | Luki i błędy w kodzie aplikacji | Code scanning, CodeQL, Copilot Autofix, dependency review |
| GitHub Secret Protection | Ujawnione tokeny, klucze i poświadczenia | Secret scanning, push protection, custom patterns |
| Supply Chain Security | Podatne lub nieaktualne zależności | Dependency graph, Dependabot alerts, security updates, dependency review |

**Skrót:** kod własny → CodeQL / code scanning; sekrety → secret scanning / push protection; zewnętrzne biblioteki → dependency graph i Dependabot.

## 2. GHAS i alerty bezpieczeństwa

### Dlaczego bezpieczeństwo „shift left”?

- Wcześniejsze wykrycie zmniejsza koszt i czas naprawy.
- Zmniejsza ryzyko, że podatność trafi na produkcję.
- Bezpieczeństwo staje się wspólną odpowiedzialnością programistów i zespołu bezpieczeństwa.
- Ignorowanie alertów grozi wyciekiem danych, przestojem, niezgodnością regulacyjną i stratami finansowymi.

| Typ alertu | Co wykrywa | Typowa reakcja |
|---|---|---|
| Code scanning / CodeQL | Niebezpieczne wzorce kodu, np. SQL injection lub XSS | Ocenić wpływ, poprawić kod, przetestować, zamknąć alert. |
| Secret scanning | Tokeny, API keys, klucze prywatne, poświadczenia | Unieważnić/obrócić sekret, zaktualizować usługi, udokumentować. |
| Dependabot | Zależność ze znaną luką | Przejrzeć szczegóły, zaktualizować, przetestować i scalić. |
| Dependency review | Ryzykowną zmianę zależności w PR | Zatrzymać ryzyko przed mergem. |

W materiałach: alerty **code scanning** i **Dependabot** mogą być obsługiwane przez użytkowników z rolą `Write`, a alerty **secret scanning** przez `Admin`. Dodatkowy dostęp do alertów można nadać osobie lub zespołowi niezależnie od roli w repozytorium.

## 3. Zależności i supply chain security

### Dependency graph

Dependency graph to inwentarz pakietów projektu. GitHub buduje go, analizując obsługiwane pliki manifestów i lockfile’y.

- Pokazuje zależności **bezpośrednie** i **przechodnie**.
- Aktualizuje się po zmianie manifestu/lockfile’a w gałęzi domyślnej.
- Stanowi źródło danych dla Dependabot alerts, security updates i dependency review.
- Można go wyeksportować jako **SBOM** zgodny ze SPDX.

### SBOM i GitHub Advisory Database

- **SBOM** (*Software Bill of Materials*) to lista składników oprogramowania; wspiera przejrzystość, szybsze wykrywanie podatności i zgodność.
- **GitHub Advisory Database** dostarcza informacji o znanych lukach m.in. z GitHub, NVD i baz ekosystemów pakietów.

### Dependabot — porównanie funkcji

| Funkcja | Kiedy działa | Rezultat |
|---|---|---|
| Dependabot alerts | Pojawia się nowa porada lub zmienia się dependency graph | Alert z metadanymi: m.in. CVSS, CWE, CVE, GHSA. |
| Security updates | Dla alertu istnieje dostępna poprawka | Pull request z aktualizacją podatnej zależności. |
| Version updates | Zgodnie z `dependabot.yml`, niezależnie od luki | PR-y utrzymujące pakiety przy aktualnych wersjach. |
| Grouped security updates | Po włączeniu w repozytorium/organizacji | Łączenie kompatybilnych aktualizacji w mniej PR-ów. |

### Najważniejsza różnica: Dependency review vs Dependabot

- **Dependency review** jest **prewencyjny**: sprawdza zmiany zależności w pull requeście, zanim zostaną scalone.
- **Dependabot** reaguje na **znane podatności** w istniejących zależnościach i pomaga je aktualizować.

### Reagowanie na alert Dependabot

1. Otwórz alert i poznaj lukę oraz jej wpływ na aplikację.
2. Jeśli istnieje security update PR — przejrzyj go, uruchom testy i scal.
3. Jeśli automatyczna aktualizacja nie istnieje — zaktualizuj zależność ręcznie.
4. Jeśli aktualizacja nie jest możliwa — oceń wykorzystywalność luki i zastosuj środek zaradczy.
5. Jeśli odrzucasz alert — udokumentuj decyzję i dalej monitoruj ryzyko.

`dependency-review-action` może blokować merge PR po przekroczeniu ustalonego progu ważności. Opcje `allow-licenses` i `deny-licenses` są wzajemnie wykluczające.

## 4. Secret scanning i push protection

**Sekret** to poświadczenie uwierzytelniające: token, klucz API, klucz prywatny itp. Umieszczenie go w repozytorium może pozwolić innym osobom działać wobec zewnętrznej usługi z uprawnieniami właściciela.

### Zakres secret scanning

- cała historia Git we wszystkich gałęziach;
- opisy i komentarze w issues;
- tytuły, opisy i komentarze w issues i pull requestach;
- tytuły, opisy i komentarze w GitHub Discussions.

| Mechanizm | Moment działania | Znaczenie |
|---|---|---|
| Secret scanning | Po wykryciu sekretu w repozytorium | Tworzy alert dla administratorów; w wybranych przypadkach powiadamia też dostawcę sekretu. |
| Push protection | Przed wysłaniem commita | Blokuje push z rozpoznawalnym sekretem. |
| Validity checks | Po wykryciu, gdy dostępne | Pomagają ustalić, czy token nadal jest lub był aktywny. |

W repozytoriach publicznych secret scanning i push protection są domyślnie włączone. Dla repozytoriów prywatnych/wewnętrznych potrzeba odpowiednich możliwości GHAS i konfiguracji.

### Reakcja na ujawniony sekret

> **Zasada egzaminacyjna:** sekret zapisany w repozytorium uznaj za skompromitowany. Usunięcie go z pliku nie wystarcza.

| Przypadek | Działania |
|---|---|
| Personal access token GitHub | Unieważnij token → utwórz nowy → zaktualizuj wszystkie korzystające usługi. |
| Inny sekret | Sprawdź poprawność → utwórz nowy → zaktualizuj usługi → usuń stary sekret. |
| Zamknięcie alertu | Dopiero po naprawie wybierz poprawny powód `Close as` i zachowaj dokumentację. |

Przed odrzuceniem alertu potwierdź, że poświadczenie unieważniono/obrócono, działania incident response zakończono, a powód odrzucenia udokumentowano. Jest to ważne dla audytu i standardów takich jak SOC 2, ISO 27001 czy PCI DSS.

### Custom patterns

Custom patterns wykrywają sekrety specyficzne dla organizacji, których GitHub nie rozpoznaje domyślnie.

- Wzorzec zawiera nazwę, regex Hyperscan, opcjonalne warunki i przykładowy ciąg testowy.
- Najpierw użyj **Save and dry run**, przejrzyj maksymalnie 1000 dopasowań i usuń false positives.
- Dopiero potem opublikuj wzorzec.
- Limity z materiału: do 500 wzorców na organizację/przedsiębiorstwo i do 100 na prywatne repozytorium.

## 5. Code scanning

Code scanning analizuje kod w repozytorium, znajduje potencjalne luki i błędy, a wyniki pokazuje jako alerty na karcie **Security**.

| Konfiguracja | Kiedy wybrać | Cechy |
|---|---|---|
| Domyślna | Szybkie uruchomienie CodeQL | Wybór języków, zestawu zapytań i zdarzeń; używa GitHub Actions. |
| Zaawansowana | Potrzebna pełna kontrola | Workflow w repozytorium, własny build, zapytania i harmonogram. |
| Narzędzie zewnętrzne | Używany jest inny silnik analizy | Wyniki są importowane do code scanning jako SARIF. |

Skanowanie można wyzwalać np. przez `push`, `pull_request` lub harmonogram.

### Czytanie i zamykanie alertu

- Alert zawiera m.in. nazwę problemu/narzędzia, linię kodu, klasyfikację i poziom zagrożenia.
- Poziomy wyniku: `Error`, `Warning`, `Note`.
- Poziomy zagrożenia: `Critical`, `High`, `Medium`, `Low`.
- Alert zamykamy przez poprawę kodu albo odrzucenie/usunięcie.
- Odrzucenie zapisuje przyczynę i zapobiega wygenerowaniu alertu dla identycznego kodu przy następnym skanowaniu.
- **Copilot Autofix** może przygotować propozycję poprawki, ale nadal trzeba ją zweryfikować testami i code review.

## 6. CodeQL

> **CodeQL** to silnik analizy statycznej. Traktuje kod jak dane: z kodu powstaje baza danych, a luki opisuje się zapytaniami uruchamianymi względem tej bazy.

### Analiza wariantów

**Variant analysis** wykorzystuje znaną lukę jako punkt startowy do wyszukania podobnych błędów w jednej lub wielu bazach kodu.

### Baza danych CodeQL

- Jest migawką kodu dla **jednego języka** w określonym momencie.
- Każdy język ma osobny ekstraktor, schemat i bazę.
- Zawiera m.in. AST, graf przepływu danych i graf przepływu sterowania.
- Dla Javy przykładowe tabele to `expressions` i `statements`; biblioteki udostępniają klasy `Expr` i `Stmt`.
- Bazy nie aktualizują się przyrostowo — zmiana kodu, builda lub konfiguracji zapytań wymaga nowej bazy.
- Dla języków kompilowanych CodeQL obserwuje udaną kompilację; dla interpretowanych działa bezpośrednio na źródłach.

### Query suites i pakiety

| Pojęcie | Co zapamiętać |
|---|---|
| Query suite (`.qls`) | YAML wybierający wiele zapytań według nazwy, metadanych lub lokalizacji. |
| `default` | Domyślny zestaw dla code scanning; wysoka precyzja i mniej false positives. |
| `security-extended` | Zawiera `default` i dodatkowe reguły; szersze pokrycie, ale możliwe więcej false positives. |
| Query package | Pakiet przeznaczony do uruchamiania zapytań. |
| Library package | Biblioteka dla zapytań/bibliotek; nie zawiera samych zapytań. |
| Model package | Rozszerza analizę o zależności niewspierane domyślnie. |
| `qlpack.yml` | Główny plik pakietu: nazwa, wersja, zależności, extractor i zestawy zapytań. |

## 7. Tworzenie bazy i uruchamianie zapytań

1. Zainstaluj CodeQL CLI i sprawdź obsługiwane języki.
2. Utwórz bazę z katalogu głównego projektu.
3. Dla języków kompilowanych zainstaluj zależności i wykonaj poprawną kompilację.
4. Dla wielu języków użyj `--db-cluster` albo macierzy języków w GitHub Actions.
5. Uruchom zapytania i opublikuj wyniki SARIF w GitHub.

Na wydajność wpływają rozmiar repozytorium, liczba języków, zasoby CI i zakres analizy. Pomagają większe zasoby runnera, macierz języków, ograniczenie testów/wygenerowanego kodu oraz rozsądny harmonogram.

### Budowa zapytania CodeQL

| Element | Rola |
|---|---|
| `import` | Ładuje model językowy lub bibliotekę. |
| `from` | Definiuje analizowane dane. |
| `where` | Filtruje wzorce. |
| `select` | Określa zwracany wynik. |
| Metadane | `@id`, `@name`, `@description`, `@kind`; pozwalają sklasyfikować wynik jako alert. |

- **Alert queries** wskazują problem w konkretnej lokalizacji kodu.
- **Path queries** pokazują przepływ informacji między źródłem a ujściem.
- Bez metadanych zapytanie może działać, ale wynik pozostanie surową tabelą i nie stanie się alertem GitHub.

`CodeQL → baza danych → zapytania → SARIF → GitHub → alert code scanning`

## 8. Troubleshooting i pułapki

| Problem | Najlepsza odpowiedź |
|---|---|
| Analiza trwa zbyt długo | Dodaj CPU/RAM, użyj macierzy języków, ogranicz analizowany kod lub uruchamiaj według harmonogramu. |
| Własne zapytanie jest wolne | Ogranicz kosztowne, duże predykaty i pamiętaj o kosztach operacji relacyjnych. |
| Potrzebny debugging workflow | W kroku `init` ustaw `debug: true`; otrzymasz artefakt `debug-artifacts` z logami, bazami i SARIF. |
| `Server error` | Ponów workflow; jeśli błąd trwa, skontaktuj się z GitHub Support. |
| Brak dysku lub pamięci | Zwiększ zasoby runnera. |
| 403 przy Dependabot | Dependabot ma uprawnienia read-only; dla jego zmian uruchamiaj analizę przez `pull_request`, nie `push`. |
| SARIF odrzucony przez default setup | Nie przesyłaj równolegle CodeQL SARIF przy włączonej domyślnej konfiguracji CodeQL. |

## 9. Powtórka w 60 sekund

- Dependency graph = inwentarz zależności.
- Advisory Database + dependency graph = Dependabot alerts.
- Security updates = PR z naprawą podatnej zależności.
- Version updates = aktualność zależności, niekoniecznie bezpieczeństwo.
- Dependency review = prewencja w pull requeście.
- Secret scanning = wykrywa wycieki; push protection = blokuje je przed pushem.
- Ujawniony sekret = rotacja/unieważnienie + aktualizacja usług.
- Code scanning = funkcja GitHub; CodeQL = silnik analizy.
- CodeQL: kod → baza → zapytanie → SARIF → alert.
- `default` = precyzja; `security-extended` = szersze pokrycie i możliwe dodatkowe false positives.

## 10. Pytania kontrolne

1. Czym różni się dependency review od Dependabot?
   - Dependency review blokuje ryzykowne zmiany zależności w PR; Dependabot wykrywa znane podatności i pomaga aktualizować istniejące pakiety.
2. Co robisz po znalezieniu tokenu w repozytorium?
   - Uznajesz go za skompromitowany, unieważniasz/obracasz, tworzysz nowy, aktualizujesz usługi i dokumentujesz zamknięcie alertu.
3. Co znajduje się w bazie CodeQL?
   - Dane wyodrębnione z kodu jednego języka, m.in. AST oraz grafy przepływu danych i sterowania.
4. Po co metadane w zapytaniu CodeQL?
   - Aby GitHub mógł sklasyfikować i pokazać wynik jako alert.
5. Kiedy używać push protection?
   - Gdy chcesz zatrzymać rozpoznawalny sekret przed wysłaniem go do repozytorium.
6. Jak usunąć błąd 403 dla CodeQL uruchomionego przez Dependabot?
   - Uruchomić analizę dla zdarzenia `pull_request`, nie `push`.

