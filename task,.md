# Azure + Docker
## Cel zadania

Należy wykonać pełny proces konteneryzacji prostej aplikacji Python oraz wdrożyć ją do Azure:

1. Create Docker image with application
2. Test containerized application locally
3. Create Azure Container Registry (ACR)
4. Upload Docker image into ACR
5. Create App Service Plan
6. Create Web App
7. Launch application on Web App

---

## 1. Prosta aplikacja Python — Hello World

Struktura katalogu projektu:

```text
projekt/
├── app.py
├── requirements.txt
└── Dockerfile
```

### `app.py`

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

> Ważne: aplikacja w kontenerze musi nasłuchiwać na `0.0.0.0`, a nie na `127.0.0.1`.

### `requirements.txt`

```text
Flask==3.0.3
```

### `Dockerfile`

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8080

CMD ["python", "app.py"]
```

---

## 2. Zbudowanie obrazu Docker

W katalogu projektu:

```bash
docker build -t hello-python:v1 .
```

Sprawdzenie obrazu:

```bash
docker images
```

Powinien być widoczny obraz:

```text
hello-python   v1
```

---

## 3. Test aplikacji lokalnie

Uruchomienie kontenera:

```bash
docker run --name hello-python -p 8080:8080 hello-python:v1
```

Test w drugim terminalu:

```bash
curl http://127.0.0.1:8080
```

Oczekiwany wynik:

```text
Hello World!
```

Sprawdzenie działających kontenerów:

```bash
docker ps
```

Jeżeli port `8080` jest już zajęty:

```bash
docker ps -a
```

Usunięcie starego kontenera:

```bash
docker rm -f hello-python
```

Można też użyć innego portu hosta:

```bash
docker run --name hello-python -p 8081:8080 hello-python:v1
```

Wtedy test:

```bash
curl http://127.0.0.1:8081
```

---

## 4. Azure Container Registry — ACR

W zadaniu używana jest:

```text
Resource Group: organization-interview
ACR: organizationacr12345
```

Jeżeli ACR nie istnieje, można utworzyć go poleceniem:

```bash
az acr create   --resource-group organization-interview   --name organizationacr12345   --sku Basic   --location swedencentral
```

Sprawdzenie:

```bash
az acr show   --resource-group organization-interview   --name organizationacr12345   --output table
```

---

## 5. Logowanie do ACR

```bash
az acr login --name organizationacr12345
```

Oczekiwany wynik:

```text
Login Succeeded
```

---

## 6. Otagowanie obrazu dla ACR

Lokalny obraz:

```text
hello-python:v1
```

Adres obrazu w ACR:

```text
organizationacr12345.azurecr.io/hello-python:v1
```

Tagowanie:

```bash
docker tag hello-python:v1 organizationacr12345.azurecr.io/hello-python:v1
```

Sprawdzenie:

```bash
docker images
```

---

## 7. Upload obrazu do ACR

```bash
docker push organizationacr12345.azurecr.io/hello-python:v1
```

Po poprawnym wysłaniu powinien pojawić się digest, np.:

```text
v1: digest: sha256:...
```

Sprawdzenie repozytorium:

```bash
az acr repository list   --name organizationacr12345   --output table
```

Oczekiwany wynik:

```text
hello-python
```

Sprawdzenie tagów:

```bash
az acr repository show-tags   --name organizationacr12345   --repository hello-python   --output table
```

Oczekiwany tag:

```text
v1
```

W Azure Portal obraz znajduje się w:

```text
Azure Container Registry
→ Usługi
→ Repozytoria
→ hello-python
→ v1
```

---

## 8. Utworzenie App Service Plan

Przykładowa nazwa planu:

```text
organization-app-plan
```

Polecenie:

```bash
az appservice plan create   --name organization-app-plan   --resource-group organization-interview   --location swedencentral   --is-linux   --sku B1
```

Sprawdzenie:

```bash
az appservice plan show   --name organization-app-plan   --resource-group organization-interview   --output table
```

---

## 9. Utworzenie Web App

Przykładowa nazwa:

```text
organization-hello-python12345
```

> Nazwa Web App musi być globalnie unikalna.

Utworzenie aplikacji:

```bash
az webapp create   --resource-group organization-interview   --plan organization-app-plan   --name organization-hello-python12345   --container-image-name organizationacr12345.azurecr.io/hello-python:v1
```

---

## 10. Włączenie Managed Identity

```bash
az webapp identity assign   --resource-group organization-interview   --name organization-hello-python12345
```

Z wyniku należy zapisać wartość:

```text
principalId
```

---

## 11. Pobranie ID Azure Container Registry

```bash
az acr show   --resource-group organization-interview   --name organizationacr12345   --query id   --output tsv
```

Wynik będzie miał format podobny do:

```text
/subscriptions/.../resourceGroups/organization-interview/providers/Microsoft.ContainerRegistry/registries/organizationacr12345
```

---

## 12. Nadanie Web App uprawnienia `AcrPull`

Podstaw wartości:

- `<PRINCIPAL_ID>` — `principalId` z Web App
- `<ACR_ID>` — ID ACR z poprzedniego kroku

```bash
az role assignment create   --assignee-object-id <PRINCIPAL_ID>   --assignee-principal-type ServicePrincipal   --scope <ACR_ID>   --role AcrPull
```

To pozwala Web App pobierać prywatny obraz z ACR.

---

## 13. Włączenie pobierania obrazu przez Managed Identity

```bash
az webapp config set   --resource-group organization-interview   --name organization-hello-python12345   --generic-configurations '{"acrUseManagedIdentityCreds": true}'
```

---

## 14. Konfiguracja portu aplikacji

Aplikacja Flask działa na porcie `8080`, dlatego należy ustawić:

```bash
az webapp config appsettings set   --resource-group organization-interview   --name organization-hello-python12345   --settings WEBSITES_PORT=8080
```

W `app.py` musi pozostać:

```python
app.run(host="0.0.0.0", port=8080)
```

---

## 15. Restart Web App

```bash
az webapp restart   --resource-group organization-interview   --name organization-hello-python12345
```

---

## 16. Uruchomienie aplikacji na Web App

Pobranie adresu:

```bash
az webapp show   --resource-group organization-interview   --name organization-hello-python12345   --query defaultHostName   --output tsv
```

Adres będzie podobny do:

```text
https://organization-hello-python12345.azurewebsites.net
```

Można także spróbować otworzyć aplikację poleceniem:

```bash
az webapp browse   --resource-group organization-interview   --name organization-hello-python12345
```

Oczekiwany rezultat w przeglądarce:

```text
Hello World!
```

---

## 17. Diagnostyka

Jeżeli aplikacja nie działa, włącz logowanie kontenera:

```bash
az webapp log config   --resource-group organization-interview   --name organization-hello-python12345   --docker-container-logging filesystem
```

Podgląd logów:

```bash
az webapp log tail   --resource-group organization-interview   --name organization-hello-python12345
```

Sprawdzenie konfiguracji kontenera:

```bash
az webapp config container show   --resource-group organization-interview   --name organization-hello-python12345
```

---

## Najczęstsze problemy

### `socket hang up` / `ECONNRESET`

Aplikacja nie powinna nasłuchiwać na:

```python
host="127.0.0.1"
```

Poprawnie:

```python
host="0.0.0.0"
```

### Port already allocated

Błąd:

```text
Bind for 0.0.0.0:8080 failed: port is already allocated
```

Sprawdzenie:

```bash
docker ps -a
```

Usunięcie starego kontenera:

```bash
docker rm -f hello-python
```

### Niewłaściwe mapowanie portu

Jeżeli Flask działa na `8080`, poprawne uruchomienie to:

```bash
docker run -p 8080:8080 hello-python:v1
```

Nie:

```bash
docker run -p 8080:80 hello-python:v1
```

---

# Rezultat końcowy

Po wykonaniu zadania w grupie zasobów `organization-interview` powinny istnieć:

```text
Azure Container Registry: organizationacr12345
Repository: hello-python
Tag: v1

App Service Plan: organization-app-plan

Web App: organization-hello-python12345
Image: organizationacr12345.azurecr.io/hello-python:v1
Port: 8080
```

Aplikacja dostępna przez Azure Web App powinna wyświetlać:

```text
Hello World!
```
