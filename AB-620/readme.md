# ALM

**ALM** (*Application Lifecycle Management*) to zarządzanie cyklem życia aplikacji lub agenta - od tworzenia, przez testowanie, po wdrożenie na produkcję.

W Copilot Studio pomaga bezpiecznie przenosić agenta między środowiskami:

1. **Dev** - tworzenie i edycja agenta.
2. **Test** - sprawdzanie działania.
3. **Prod** - gotowa wersja dla użytkowników.

## Elementy ALM

- **Custom solution** - paczka z agentem, topicami, flow i innymi komponentami; można ją eksportować i importować.
- **Environment variables** - wartości zależne od środowiska, np. osobne adresy API dla testu i produkcji.
- **Connection references** - konfiguracja połączeń bez wpisywania danych dostępowych w agencie.
- **Pipelines** - kontrolowane wdrażanie: `Dev -> Test -> Prod`.
- **Unmanaged solution** - rozwiązanie deweloperskie, które można edytować.
- **Managed solution** - rozwiązanie produkcyjne, chronione przed przypadkową edycją.

> W skrócie: ALM umożliwia tworzenie agenta w środowisku deweloperskim, testowanie go, a następnie uporządkowane wdrożenie na produkcję.

## 1. Topics

| Topic| Tak | Nie |
|---|---|---|
| PDF, SharePoint, polityki, FAQ | **Knowledge source** | ręcznych topiców Q&A |
| Indeks, cytaty, enterprise search | **Azure AI Search** | wrzucenia dokumentów zamiast indeksu |
| Dane live, tylko odczyt, bez indeksu | **Real-time knowledge** | knowledge connectora indeksującego |
| Zmiana danych / transakcja / create ticket | **Tool / connector** | knowledge source |
| Wewnętrzne REST API bez gotowego konektora | **Custom connector** | HTTP jako domyślnej odpowiedzi |
| Proste publiczne REST API + API key | **HTTP request / REST tool** | Dataverse lub knowledge |
| Rebooking, retry, cache, logika wielokrotnego użytku | **Agent flow** | kopiowania logiki po topicach |
| Metryki / semantic model w Fabric | **Fabric Data Agent** | indeksowania tabel Fabric |
| Partner-agent / standardowy endpoint agenta | **A2A endpoint** | MCP lub knowledge |
| Agent z Microsoft Foundry | **External agent + Foundry project endpoint** | dodawania go jako knowledge |
| Standaryzowane wywoływalne API/tools | **MCP server + tools** | samego promptu lub publish |
| Stary program desktopowy bez API | **Computer use** | custom connectora |

##
1. **Czy dane są dokumentem, czy działaniem?**
   - dokument / wiedza -> knowledge
   - działanie / zapis / pobranie live -> tool, connector, HTTP lub flow
2. **Czy to dane osobowe albo dane pracownika?**
   - PII -> user sign-in
   - pracownik -> Microsoft Entra ID
   - ogólne FAQ -> anonymous może być OK
3. **Czy rozwiązanie ma być wspólne i powtarzalne?**
   - logika -> agent flow
   - elementy wspólne -> component collection
   - migracja środowisk -> custom solution + ALM
##
`DLP -> content moderation -> prompt modifications`

- **DLP** chroni przepływ danych i konektory.
- **Moderation** filtruje niebezpieczne wejście i wyjście.
- **Prompt modifications** ustawiają ton, odmowę i disclaimer dla odpowiedzi generatywnych.

##

**FAQ = anon; moje dane = logowanie; firma = Entra.**

##
| Potrzeba | Wybór |
|---|---|
| Partner lub niezależny agent przez standard protokołu | A2A + endpoint |
| Inny agent Copilot Studio | publish connected agent + configure connection + add agent |
| Agent w Microsoft Foundry | external agent + project endpoint URL |
| Narzędzia MCP | add MCP server to tool configuration |

Przy produkcyjnym problemie: **disable flow -> resubmit run -> review activity history**.

### MCP w 4 hasłach

`server URL + authentication on connection + allowed tools + test`

- Tylko dozwolone operacje: wybierz konkretne tools.
- API key/sekret należy do connection, nie do topica.
- Sama dokumentacja serwera, wiedza o serwerze lub publish **nie** uruchamia MCP.

### Kolejność AI Search

`Add knowledge -> create AI Search connection -> auth/details -> choose index -> associate with agent`

Jeśli wymagane są odpowiedzi **tylko na podstawie indeksu**: wybierz grounding z AI Search i wymaganie znalezienia relewantnej treści; nie zastępuj tego uploadem dokumentów.

## 8. Computer use

Skrót: **brak API + aplikacja desktopowa = Computer use**.

## 9. ALM

`Dev (unmanaged) -> Test -> Prod (managed)`

- **Custom solution** jest kontenerem do export/import i transportu agenta.
- Kanał publikacji udostępnia agenta użytkownikom - **nie** przenosi go między środowiskami.
- **Environment variables**: endpointy, ID i ustawienia różne dla Dev/Test/Prod.
- **Connection references**: połączenia zależne od środowiska.
- Non-secret env var po zmianie zwykle wymaga republish; sekrety obsługuj przez właściwą konfigurację środowiska.

### Pipelines - kolejność

`enable tenant pipelines -> assign environments to stages -> run pipeline -> review result`

Hasła hotspotów:

- Dev/Test/Prod po kolei -> **sequential stage progression**
- brakujące connection refs/env vars przed deployem -> **deployment input validation**
- centralne zarządzanie etapami i security -> **centralized pipeline configuration**

## 10. Evaluation

### Test set

`high-value intents -> representative prompts -> expected outcomes -> baseline`

- Powtarzalne testy -> curated prompt list.
- Uruchamiane przez zespół -> on-demand evaluation.
- Kryterium sukcesu -> compare against defined expectations.
- Odpowiedź nie musi być identycznym stringiem: **semantic match** może być poprawny.

### Jak czytać wynik

- zgodne / niezgodne z oczekiwaniem -> pass/fail
- te same błędy w wielu runach -> consistent, powtarzalny problem
- expected i generated response -> porównanie, nie automatyczna root cause

## 11. Procesy

| Sytuacja | Kolejność |
|---|---|
| RAI | `DLP -> moderation -> prompt modifications` |
| Flow sypie się w prod | `disable -> resubmit -> activity history` |
| Nowy agent flow | `create flow -> invocation trigger -> I/O + logika -> publish/connect` |
| REST tool | `add tool -> endpoint -> auth -> test` |
| AI Search | `knowledge -> connection -> auth -> index -> agent` |
| Pipeline | `enable -> assign stages -> run -> review` |
| Evaluation | `intents -> prompts -> expectations -> baseline` |
| App Insights | `destination -> configure agent -> enable -> publish` |

> **Dokument = knowledge. Dane live = connector/HTTP. Akcja = tool. Logika = flow. Drugi agent = A2A/external agent. Migracja = solution.**

