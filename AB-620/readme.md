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

