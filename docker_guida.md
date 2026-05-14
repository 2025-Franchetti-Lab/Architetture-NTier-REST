# Guida alla Containerizzazione con Docker

ServerRest Calcolatrice — Java 24 + GSON

Java 24
Docker
Windows
REST API

## Indice

1. [Concetti fondamentali](#concetti)
2. [Installazione Docker Desktop su Windows](#strumenti)
3. [Struttura del progetto](#struttura)
4. [Creazione del Dockerfile](#dockerfile)
5. [File .dockerignore](#dockerignore)
6. [Build dell'immagine](#build)
7. [Avvio del container](#run)
8. [Verifica del funzionamento](#verifica)
9. [Docker Compose (opzionale)](#compose)
10. [Comandi Docker essenziali](#comandi)
11. [Risoluzione problemi comuni](#troubleshooting)

## 1 Concetti fondamentali

Docker permette di eseguire un'applicazione in un **container**: un ambiente isolato e riproducibile che contiene tutto il necessario (JDK, librerie, configurazione) indipendentemente dal sistema operativo dell'host.

### Termini chiave

| Termine | Descrizione |
| --- | --- |
| **Image** | Snapshot read-only del filesystem e della configurazione dell'applicazione. È il "modello" da cui si crea il container. |
| **Container** | Istanza in esecuzione di un'image. Può essere avviato, fermato, eliminato. |
| **Dockerfile** | File di testo con le istruzioni per costruire l'image passo per passo. |
| **Registry** | Repository di image (es. Docker Hub). Da qui si scaricano le image di base (`openjdk`, `ubuntu`, ecc.). |
| **Volume** | Meccanismo per persistere i dati al di fuori del filesystem del container. |
| **Port mapping** | Collega una porta del container a una porta dell'host (`-p HOST:CONTAINER`). |

### Flusso di lavoro

Workflow Docker

`Dockerfile` → `docker build` → **Image** → `docker run` → **Container in esecuzione**

## 2 Installazione Docker Desktop su Windows

**Docker Desktop** è l'ambiente ufficiale per eseguire Docker su Windows. Integra il Docker Engine, la CLI, Docker Compose e un'interfaccia grafica per gestire immagini e container.

### Requisiti di sistema

| Requisito | Dettaglio |
| --- | --- |
| Sistema operativo | Windows 10 64-bit (versione 21H2 o superiore) oppure Windows 11 |
| Architettura | x86-64 (amd64) |
| RAM | Minimo 4 GB (consigliati 8 GB) |
| Virtualizzazione hardware | Abilitata nel BIOS/UEFI (Intel VT-x o AMD-V) |
| Backend consigliato | WSL 2 (Windows Subsystem for Linux 2) |

### Passo 1 — Abilitare WSL 2

Docker Desktop su Windows utilizza **WSL 2** come backend predefinito. Aprire **PowerShell come Amministratore** ed eseguire:

```
# Abilita WSL e installa la distribuzione Ubuntu predefinita
wsl --install

# Imposta WSL 2 come versione predefinita
wsl --set-default-version 2

# Verifica la versione WSL installata
wsl --version
```

Riavvio necessario

Dopo aver abilitato WSL è richiesto il riavvio del computer prima di procedere con l'installazione di Docker Desktop.

### Passo 2 — Scaricare Docker Desktop

1. Aprire il browser e andare su **https://www.docker.com/products/docker-desktop/**
2. Cliccare su **"Download for Windows"** per scaricare il file `Docker Desktop Installer.exe`
3. Attendere il completamento del download (il file pesa circa 500 MB)

### Passo 3 — Eseguire il programma di installazione

1. Avviare `Docker Desktop Installer.exe` con doppio clic (potrebbe richiedere i permessi di amministratore)
2. Nella schermata di configurazione, assicurarsi che l'opzione **"Use WSL 2 instead of Hyper-V"** sia selezionata
3. Cliccare **OK** per avviare l'installazione e attendere il completamento
4. Al termine, cliccare **Close and restart** per riavviare il computer

### Passo 4 — Avviare Docker Desktop

1. Dopo il riavvio, aprire **Docker Desktop** dal menu Start o dal collegamento sul Desktop
2. Accettare il *Docker Subscription Service Agreement* (gratuito per uso personale e didattico)
3. Attendere che Docker Desktop raggiunga lo stato **"Engine running"** (icona verde nella barra delle applicazioni)
4. Facoltativamente, creare un account Docker Hub gratuito per accedere ai registry pubblici

Docker Desktop è pronto

Quando l'icona della balena nella barra delle applicazioni di Windows è di colore verde, il Docker Engine è in esecuzione e i comandi `docker` sono disponibili dal terminale.

### Passo 5 — Verificare l'installazione

Aprire il **Prompt dei comandi** (cmd) o **PowerShell** e verificare che Docker risponda correttamente:

```
# Verifica la versione installata
docker --version
# Output atteso (esempio):
# Docker version 27.x.x, build ...

# Verifica che il daemon sia in esecuzione
docker info
# Deve mostrare "Server: Docker Engine" senza errori

# Test rapido: scarica ed esegue un container di prova
docker run hello-world
# Deve stampare "Hello from Docker!" se tutto funziona
```

Attenzione — "Cannot connect to the Docker daemon"

Se compare questo errore, assicurarsi che Docker Desktop sia avviato e che l'icona nella barra delle applicazioni mostri lo stato *"Engine running"*. In caso di problemi, provare a riavviare Docker Desktop dal menu contestuale dell'icona nella system tray.

### Configurazione consigliata per uso didattico

In Docker Desktop, aprire **Settings → Resources** e impostare:

| Risorsa | Valore consigliato |
| --- | --- |
| CPU | 2–4 core |
| Memory (RAM) | 2–4 GB |
| Disk image size | 20–40 GB |

## 3 Struttura del progetto

Il progetto **ServerRest-Calcolatrice** è un'applicazione Java 24 costruita con **Apache Ant** tramite NetBeans. Espone un'API REST sulla porta `8080`.

```
ServerRest-Calcolatrice/
├── src/serverrest/
│   ├── App.java                # Entry point (main class)
│   ├── ServerRest.java         # Avvio HttpServer
│   ├── CalcolatriceService.java
│   ├── GetHandler.java
│   ├── PostHandler.java
│   ├── OperazioneRequest.java
│   └── OperazioneResponse.java
├── lib/
│   └── gson-2.13.2.jar         # Unica dipendenza esterna
├── build.xml                   # Script Ant
├── manifest.mf
└── nbproject/
```

Endpoint esposti dal server:

* `GET /api/calcola/get?operando1=X&operando2=Y&operatore=OP`
* `POST /api/calcola/post` — body JSON con operando1, operando2, operatore
* `GET /` — info sull'API in formato JSON

## 4 Creazione del Dockerfile

Creare il file `Dockerfile` **senza estensione** nella cartella radice del progetto. Si usa un **multi-stage build**: la prima fase compila il progetto, la seconda crea l'immagine finale leggera.

Attenzione — creazione del Dockerfile su Windows

Windows Explorer e il Blocco Note aggiungono automaticamente l'estensione `.txt`, rendendo il file inutilizzabile da Docker. Per creare il file correttamente usare uno dei metodi seguenti:

* **VS Code** (consigliato): *File → New File*, salvare con nome `Dockerfile` (senza estensione) nella cartella del progetto.
* **PowerShell**: eseguire `New-Item Dockerfile` dalla cartella del progetto — crea il file vuoto senza estensione.
* **Notepad++**: *Salva con nome*, selezionare tipo *"All files (\*.\*)"* e digitare `Dockerfile`.

Lo stesso vale per il file `.dockerignore` (sezione 5).

```
Dockerfile# ── Stage 1: Build ──────────────────────────────────────────
FROM eclipse-temurin:24-jdk-alpine AS builder

# Installa Apache Ant
RUN apk add --no-cache ant

# Crea la directory di lavoro
WORKDIR /app

# Copia i file necessari per la build
COPY src/        ./src/
COPY lib/        ./lib/
COPY build.xml   ./
COPY manifest.mf ./
COPY nbproject/  ./nbproject/

# Compila e crea il JAR tramite Ant
RUN ant jar

# ── Stage 2: Runtime ─────────────────────────────────────────
FROM eclipse-temurin:24-jre-alpine

WORKDIR /app

# Copia solo il JAR prodotto e la libreria GSON
COPY --from=builder /app/dist/ServerRest.jar ./ServerRest.jar
COPY --from=builder /app/lib/              ./lib/

# Espone la porta del server REST
EXPOSE 8080

# Avvia il server; il numero di porta può essere sovrascritto
# passando un argomento al container: docker run ... serverrest 9090
ENTRYPOINT ["java", "-cp", "ServerRest.jar:lib/gson-2.13.2.jar", "serverrest.App"]
CMD ["8080"]
```

### Spiegazione delle istruzioni principali

| Istruzione | Significato |
| --- | --- |
| `FROM` | Specifica l'image di base. `eclipse-temurin:24-jdk-alpine` contiene OpenJDK 24 su Alpine Linux (immagine piccola). |
| `WORKDIR` | Imposta la directory di lavoro all'interno del container. |
| `COPY` | Copia file/cartelle dall'host al filesystem del container. |
| `RUN` | Esegue un comando durante la build (ogni `RUN` crea un layer). |
| `EXPOSE` | Documenta la porta usata dall'applicazione (non la pubblica automaticamente). |
| `ENTRYPOINT` | Comando principale eseguito all'avvio del container. Non sovrascrivibile senza `--entrypoint`. |
| `CMD` | Argomenti di default per `ENTRYPOINT`. Sovrascrivibile passando argomenti a `docker run`. |

Perché multi-stage build?

La prima stage installa JDK + Ant (∼400 MB). La seconda usa solo il JRE (∼180 MB) e contiene esclusivamente il JAR compilato. L'immagine finale è molto più piccola e sicura (nessun compilatore, nessun Ant).

## 5 File .dockerignore

Il file `.dockerignore` esclude dalla build i file non necessari, accelerando il trasferimento del contesto a Docker e riducendo la dimensione dell'immagine.

Creare il file `.dockerignore` nella cartella radice del progetto. Su Windows usare VS Code oppure il comando PowerShell `New-Item .dockerignore`, poiché Windows Explorer non consente di creare file il cui nome inizia con un punto.

```
.dockerignore# Output di build (verranno rigenerati)
build/
dist/

# File di sistema Windows
Thumbs.db
desktop.ini
*.lnk

# Configurazione privata di NetBeans
nbproject/private/

# Git
.git/
.gitattributes

# File HTML di test (non servono nel container)
src/serverrest/test.html
*.html
```

Verifica dell'esclusione

Per verificare quali file vengono inclusi nel contesto di build, eseguire in PowerShell dalla cartella del progetto:
`docker build --no-cache --progress=plain -t test . 2>&1 | Select-String "COPY"`

## 6 Build dell'immagine

Aprire **PowerShell** (consigliato) oppure il **Prompt dei comandi (cmd)**, posizionarsi nella cartella del progetto e lanciare il comando di build:

PowerShell o Prompt dei comandi?

I comandi `docker` funzionano su entrambi. PowerShell è preferibile perché supporta la sintassi con backtick (`` ` ``) per spezzare i comandi su più righe, ed è disponibile di default su Windows 10 e 11.

```
# Apri PowerShell dalla cartella del progetto:
# tasto destro sulla cartella in Explorer → "Apri in Terminal" (Win 11)
# oppure cerca "PowerShell" nel menu Start e naviga manualmente

# Entra nella cartella del progetto — adatta il percorso alla tua situazione
# In PowerShell:
cd C:\Users\nomeutente\Documents\ServerRest-Calcolatrice

# In alternativa, con il percorso relativo dalla home utente:
cd $env:USERPROFILE\Documents\ServerRest-Calcolatrice

# Costruisce l'immagine con il tag "serverrest-calcolatrice:1.0"
docker build -t serverrest-calcolatrice:1.0 .

# Oppure con il tag "latest" (più comune)
docker build -t serverrest-calcolatrice .
```

### Output atteso durante la build

```
[+] Building 45.2s (14/14) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load .dockerignore
 => [builder 1/7] FROM eclipse-temurin:24-jdk-alpine
 => [builder 2/7] RUN apk add --no-cache ant
 => [builder 3/7] WORKDIR /app
 => [builder 4/7] COPY src/ ./src/
 => [builder 5/7] COPY lib/ ./lib/
 ...
 => [builder 7/7] RUN ant jar
 => [stage-1 2/3] COPY --from=builder /app/dist/ServerRest.jar ...
 => exporting to image
 => naming to docker.io/library/serverrest-calcolatrice:1.0
```

### Verificare le immagini presenti

```
docker images

# Output:
# REPOSITORY               TAG    IMAGE ID       SIZE
# serverrest-calcolatrice  1.0    a1b2c3d4e5f6   ~200MB
```

Build riuscita

Se il comando termina senza errori, l'immagine è pronta. La build successiva sarà molto più rapida grazie alla cache dei layer Docker.

## 7 Avvio del container

Continuazione riga su Windows

Su Windows il carattere per spezzare un comando su più righe è diverso dal terminale Unix (`\`):
• **PowerShell**: backtick `` ` `` (il carattere sotto il tasto `Esc`)
• **Prompt dei comandi (cmd)**: `^`
In alternativa si può scrivere il comando tutto su una riga: Docker accetta entrambe le forme.

### Avvio base (foreground)

```
docker run -p 8080:8080 serverrest-calcolatrice
```

### Avvio in background (detached)

```
# PowerShell — continuazione con backtick `
docker run -d `
  -p 8080:8080 `
  --name calcolatrice-server `
  serverrest-calcolatrice

# Prompt dei comandi (cmd) — continuazione con ^
docker run -d ^
  -p 8080:8080 ^
  --name calcolatrice-server ^
  serverrest-calcolatrice

# Oppure su una riga sola (funziona in entrambi i terminali)
docker run -d -p 8080:8080 --name calcolatrice-server serverrest-calcolatrice
```

### Avvio su porta personalizzata (es. 9090)

```
# PowerShell
docker run -d `
  -p 9090:9090 `
  --name calcolatrice-server `
  serverrest-calcolatrice 9090

# Su una riga sola (cmd e PowerShell)
docker run -d -p 9090:9090 --name calcolatrice-server serverrest-calcolatrice 9090
```

### Significato delle opzioni principali

| Opzione | Significato |
| --- | --- |
| `-d` | Detached: il container gira in background |
| `-p 8080:8080` | Mappa la porta 8080 dell'host alla porta 8080 del container |
| `--name calcolatrice-server` | Assegna un nome al container (più comodo dei hash) |
| `--rm` | Elimina automaticamente il container quando si ferma |
| `-e VARIABILE=valore` | Imposta una variabile d'ambiente nel container |
| `-v "C:\percorso\host":/container` | Monta un volume (cartella condivisa host ↔ container). Su Windows i percorsi host usano il backslash e vanno racchiusi tra virgolette doppie. |

### Output del server all'avvio

```
==============================================
  Server REST con GSON avviato!
==============================================
Porta: 8080

Endpoint disponibili:
  - POST: http://localhost:8080/api/calcola/post
  - GET:  http://localhost:8080/api/calcola/get
  - Info: http://localhost:8080/

Operatori supportati:
  SOMMA, SOTTRAZIONE, MOLTIPLICAZIONE, DIVISIONE

Premi Ctrl+C per fermare il server
==============================================
```

## 8 Verifica del funzionamento

### Dall'interfaccia grafica (Docker Desktop)

* **Docker Desktop**: aprire l'app → sezione *Containers* → cliccare sul container → tab *Logs*

### Dalla riga di comando

curl su Windows

`curl.exe` è incluso in Windows 10 (build 17063+) e Windows 11. In **PowerShell**, il comando `curl` è un alias di `Invoke-WebRequest`: per usare il vero curl digitare `curl.exe`, oppure usare direttamente `Invoke-WebRequest` come mostrato di seguito. Nel **Prompt dei comandi (cmd)** il comando `curl` chiama sempre curl.exe.

```
# Test endpoint info — funziona in cmd e PowerShell
curl.exe http://localhost:8080/

# Test GET – somma di 15 + 7 — funziona in cmd e PowerShell
curl.exe "http://localhost:8080/api/calcola/get?operando1=15&operando2=7&operatore=SOMMA"

# Test POST – moltiplicazione 6 × 9
# Prompt dei comandi (cmd): JSON tra virgolette doppie, virgolette interne precedute da \
curl.exe -X POST http://localhost:8080/api/calcola/post -H "Content-Type: application/json" -d "{\"operando1\":6,\"operando2\":9,\"operatore\":\"MOLTIPLICAZIONE\"}"

# PowerShell: alternativa con Invoke-WebRequest (virgolette singole per il body JSON)
Invoke-WebRequest -Method POST `
  -Uri "http://localhost:8080/api/calcola/post" `
  -ContentType "application/json" `
  -Body '{"operando1":6,"operando2":9,"operatore":"MOLTIPLICAZIONE"}'
```

### Risposta JSON attesa (POST)

```
{
  "risultato": 54.0,
  "operazione": "6.0 MOLTIPLICAZIONE 9.0",
  "errore": null
}
```

### Visualizzare i log del container

```
# Log in tempo reale (seguire)
docker logs -f calcolatrice-server

# Ultime 50 righe
docker logs --tail 50 calcolatrice-server
```

## 9 Docker Compose (opzionale)

Docker Compose semplifica l'avvio definendo la configurazione in un file YAML. Utile quando si vogliono aggiungere servizi aggiuntivi (database, proxy, ecc.).

Creare il file `compose.yaml` nella radice del progetto:

```
compose.yamlservices:
  calcolatrice:
    build: .
    image: serverrest-calcolatrice:1.0
    container_name: calcolatrice-server
    ports:
      - "8080:8080"
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8080/"]
      interval: 30s
      timeout: 5s
      retries: 3
```

### Comandi Docker Compose

```
# Avvia i servizi (build inclusa se necessario)
docker compose up -d

# Ferma i servizi
docker compose down

# Ricostruisce l'immagine e riavvia
docker compose up --build -d

# Visualizza lo stato dei servizi
docker compose ps

# Log di tutti i servizi
docker compose logs -f
```

## 10 Comandi Docker essenziali

### Gestione immagini

```
# Elenco immagini
docker images

# Elimina un'immagine
docker rmi serverrest-calcolatrice:1.0

# Elimina tutte le immagini non usate
docker image prune

# Scarica un'immagine da Docker Hub
docker pull eclipse-temurin:24-jre-alpine
```

### Gestione container

```
# Elenco container in esecuzione
docker ps

# Elenco di tutti i container (inclusi quelli fermi)
docker ps -a

# Ferma un container
docker stop calcolatrice-server

# Avvia un container esistente
docker start calcolatrice-server

# Elimina un container (deve essere fermo)
docker rm calcolatrice-server

# Ferma e rimuove in un colpo solo
docker rm -f calcolatrice-server

# Apre una shell all'interno del container in esecuzione
docker exec -it calcolatrice-server sh

# Ispeziona dettagli del container (JSON)
docker inspect calcolatrice-server
```

### Pulizia generale

```
# Rimuove tutto ciò che non è in uso (immagini, container, network, cache)
docker system prune

# Con conferma automatica (attenzione: elimina tutto!)
docker system prune -f
```

## 11 Risoluzione problemi comuni

Problema: "port is already allocated"

La porta 8080 è già usata da un altro processo o container.
**Soluzione 1:** usare una porta diversa — `docker run -p 8081:8080 ...`
**Soluzione 2:** identificare e terminare il processo che occupa la porta. In PowerShell o cmd:

```
# Trova il PID del processo che usa la porta 8080
netstat -ano | findstr :8080
# Termina il processo (sostituire 1234 con il PID trovato)
taskkill /PID 1234 /F
```

Problema: build fallisce su "ant jar"

Ant non riesce a trovare la struttura del progetto NetBeans.
**Soluzione:** verificare che il file `build.xml` e la cartella `nbproject/` siano nella radice del progetto e non siano esclusi dal `.dockerignore`.

Problema: "ClassNotFoundException: serverrest.App"

Il JAR non include le classi o il classpath è errato.
**Soluzione:** verificare che `manifest.mf` contenga `Main-Class: serverrest.App` e che il `ENTRYPOINT` nel Dockerfile includa `lib/gson-2.13.2.jar` nel classpath.

Problema: il container si avvia e si chiude subito

L'applicazione è andata in crash all'avvio.
**Soluzione:** `docker logs calcolatrice-server` per vedere il messaggio di errore.

Comandi diagnostici utili

`docker logs calcolatrice-server` — log completi
`docker inspect calcolatrice-server` — configurazione e stato
`docker exec -it calcolatrice-server sh` — shell nel container

### Riepilogo del workflow completo

1. Assicurarsi che Docker Desktop sia in esecuzione (icona verde nella barra delle applicazioni)
2. Creare `Dockerfile` e `.dockerignore` nella radice del progetto
3. Eseguire `docker build -t serverrest-calcolatrice .`
4. Eseguire `docker run -d -p 8080:8080 --name calcolatrice-server serverrest-calcolatrice`
5. Testare con `curl http://localhost:8080/`
6. Consultare i log con `docker logs -f calcolatrice-server` in caso di problemi

ServerRest-Calcolatrice • TEPSIT Classe 5 • Guida Docker • 2026
