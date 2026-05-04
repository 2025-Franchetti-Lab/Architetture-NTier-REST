Informatica · Classe 5ª · Reti e Architetture

# Proxy, Gateway *& Load Balancer*

Architetture per servizi REST
//
Anno scolastico 2025–26

[Introduzione](#intro)
[Proxy](#proxy)
[API Gateway](#gateway)
[Load Balancer](#lb)
[Confronto](#confronto)
[Scenari reali](#scenari)
[Sommario](#sommario)

// 00 — Contesto

## Perché esistono questi componenti?

Quando un'applicazione client vuole comunicare con uno o più server REST, raramente lo fa in modo diretto. Tra il client e il server si interpone spesso uno o più componenti di **intermediazione** che svolgono funzioni cruciali: sicurezza, ottimizzazione, bilanciamento del carico, instradamento intelligente.

In questa unità didattica analizziamo tre componenti fondamentali di questo ecosistema: il **Proxy**, l'**API Gateway** e il **Load Balancer**. Pur avendo funzioni simili in superficie, rispondono a esigenze molto diverse e spesso convivono nella stessa architettura.

SENZA INTERMEDIARI

CLIENT

GET /api/prodotti — nessun controllo, nessuna ottimizzazione

SERVER REST

CON ARCHITETTURA STRATIFICATA

CLIENT

PROXY

API GATEWAY

LOAD BALANCER

SERVER REST

cache · SSL · filtro
authn · rate limit · routing
distribuzione · HA · health check

risposta HTTP percorre il tragitto inverso

**Nota metodologica:** nella pratica non è sempre necessario avere tutti e tre i componenti. La scelta dipende dalle esigenze del progetto: dimensione del traffico, sicurezza richiesta, numero di servizi backend.

// 01 — Primo componente

Proxy

## Il Proxy

Il termine *proxy* viene dall'inglese e significa "delegato" o "intermediario". In ambito informatico è un server che si frappone tra il client e il server di destinazione, **agendo per conto del client**. Il client non conosce (e spesso non si preoccupa di) chi è il destinatario finale della richiesta.

### Come funziona un Proxy in un contesto REST

Il client invia una richiesta HTTP/HTTPS al proxy. Il proxy la riceve, può ispezionarla, modificarla, filtrarla, e poi la inoltra al server REST. La risposta percorre il tragitto inverso.

BROWSER
/ App client

GET /api/prodotti

PROXY SERVER

① Cache hit?
→ sì: risposta immediata al client
→ no: procede al passo successivo

② Filtra header / IP
③ Comprime risposta (gzip)
④ Logga la richiesta
⑤ Inoltra al server →

GET /api/prodotti (inoltrata)

SERVER REST

200 OK { data: [...] }

PROXY (risposta di ritorno)
salva in cache · decomprime · aggiunge header
→ risposta al client

BROWSER
(riceve risposta)

richiesta →

risposta ←

### Tipologie principali

#### Proxy Forward (Forward Proxy)

È il tipo più comune nelle reti aziendali. Il client lo configura esplicitamente: ogni richiesta verso l'esterno passa attraverso di esso. Nasconde l'identità del client verso il server.

#### Proxy Reverse (Reverse Proxy)

È il più usato nelle architetture REST. Il client non sa dell'esistenza del proxy: vede solo un indirizzo IP pubblico. Il reverse proxy smista le richieste verso uno o più server interni. **È il tipo più rilevante per le applicazioni web moderne.**

### Caratteristiche principali

* **Caching:** memorizza le risposte per richieste identiche, riducendo il carico sui server REST e migliorando i tempi di risposta.
* **Anonimizzazione:** nasconde l'indirizzo IP del client (forward proxy) o del server (reverse proxy).
* **Filtraggio dei contenuti:** può bloccare richieste verso domini non autorizzati o con payload sospetti.
* **Compressione:** può comprimere le risposte (gzip, br) prima di inviarle al client, riducendo il traffico di rete.
* **SSL Termination:** gestisce la crittografia HTTPS, sollevando i server backend da questo compito computazionalmente costoso.
* **Logging e monitoraggio:** registra tutto il traffico in transito per analisi e audit.

#### Quando usarlo ✓

* Serve caching aggressivo per API con dati che cambiano poco
* Si vuole nascondere la topologia interna della rete
* È necessario terminare SSL centralmente
* Si vuole loggare tutto il traffico HTTP
* Accesso controllato a risorse esterne dalla rete aziendale

#### Quando evitarlo ✗

* Si ha bisogno di logica di routing complessa tra microservizi
* Serve autenticazione e autorizzazione granulare per API
* Il traffico è già end-to-end crittografato con mutual TLS
* Si gestiscono richieste con stato (WebSocket, SSE) e il proxy non le supporta

**Attenzione:** un proxy mal configurato può diventare un collo di bottiglia o un single point of failure. È fondamentale dimensionarlo correttamente e prevedere ridondanza.

Strumenti comuni: `Nginx`, `HAProxy`, `Squid`, `Varnish` (orientato al caching).

// 02 — Secondo componente

API Gateway

## L'API Gateway

L'API Gateway è un componente specializzato nella gestione del traffico verso le API. A differenza del proxy generico, **conosce la semantica delle API** e può prendere decisioni sofisticate basate sul contenuto delle richieste: l'endpoint chiamato, il token JWT allegato, la versione dell'API, la quota residua del client.

### Ruolo nelle architetture a microservizi

In un'architettura a microservizi, ogni servizio espone la propria API REST. Senza un gateway, il client dovrebbe conoscere l'indirizzo e la porta di ogni singolo servizio. Il gateway funge da **punto di ingresso unico** (*single entry point*): il client parla solo col gateway, che instrada la richiesta al microservizio corretto.

CLIENT MOBILE
iOS / Android

CLIENT WEB
Browser SPA

CLIENT IoT
Dispositivo Edge

API GATEWAY

① Autenticazione
JWT / OAuth2 / API Key

② Rate Limiting
100 req/min per API key

③ Routing semantico
/utenti → svc-utenti
/ordini → svc-ordini

④ Versioning
/v1/... → backend v1

⑤ Circuit Breaker
stop se svc non risponde

/utenti/\*
/ordini/\*
/pagamenti/\*

svc-utenti
:8081 REST

svc-ordini
:8082 REST

svc-pagamenti
:8083 REST

### Caratteristiche principali

* **Autenticazione e Autorizzazione:** verifica token JWT, chiavi API, sessioni OAuth2. Il client viene autenticato una sola volta al gateway; i servizi interni non devono occuparsene.
* **Rate Limiting e Throttling:** limita il numero di richieste per client/API key in un intervallo di tempo, proteggendo i backend dall'abuso.
* **Routing semantico:** instrada le richieste in base all'URL, al metodo HTTP, agli header o al payload verso il servizio corretto.
* **Trasformazione delle richieste:** può modificare header, convertire formati (XML → JSON), aggregare risposte da più servizi.
* **Versioning delle API:** gestisce più versioni della stessa API (`/v1/`, `/v2/`) in modo trasparente per il client.
* **Circuit Breaker:** interrompe il traffico verso un servizio che sta fallendo, evitando cascate di errori nell'architettura.
* **Documentazione e Developer Portal:** spesso integra Swagger/OpenAPI per esporre la documentazione ai sviluppatori.

#### Quando usarlo ✓

* Architettura a microservizi con molte API da esporre
* Necessità di autenticazione centralizzata
* Si vuole applicare rate limiting per cliente
* API pubbliche consumate da sviluppatori terzi
* Gestione di più versioni della stessa API
* Monitoraggio dettagliato dell'utilizzo delle API

#### Quando evitarlo ✗

* Applicazione monolitica con una sola API semplice
* Il gateway introduce latenza insostenibile per requisiti real-time stretti
* Risorse limitate: un gateway enterprise è costoso da gestire
* Comunicazione interna tra microservizi (meglio una service mesh)

Strumenti comuni: `Kong`, `AWS API Gateway`, `Azure API Management`, `Apigee`, `Traefik`, `NGINX Plus`.

// 03 — Terzo componente

Load Balancer

## Il Load Balancer

Il Load Balancer (bilanciatore di carico) risolve un problema di **scalabilità e disponibilità**: distribuisce il traffico in ingresso tra più istanze dello stesso server REST, in modo che nessuna istanza sia sovraccaricata e che il servizio rimanga attivo anche se una o più istanze vanno offline.

### Il problema che risolve

Un singolo server REST ha limiti fisici: CPU, RAM, connessioni concorrenti. Quando il traffico supera questi limiti, il server rallenta o si blocca. La soluzione è avere più istanze dello stesso server e distribuire il carico tra loro.

TRAFFICO IN ARRIVO
migliaia di richieste HTTP

LOAD BALANCER

Algoritmo
Round Robin
Least Conn.
IP Hash

Health Check
GET /health
ogni 5 secondi
timeout: 2s

Sessioni
Sticky: opz.
SSL offload
L4 / L7

req. 1
req. 2
req. 3
esclusa

REST istanza 1
:8080 ● attiva
✓ health OK

REST istanza 2
:8081 ● attiva
✓ health OK

REST istanza 3
:8082 ● attiva
✓ health OK

REST istanza 4
:8083 ✗ DOWN
esclusa dal pool

health check

### Algoritmi di bilanciamento

#### Round Robin

Le richieste vengono distribuite in sequenza ciclica tra le istanze: 1→2→3→4→1→2→... È il metodo più semplice e funziona bene quando tutte le istanze hanno le stesse capacità.

#### Least Connections

La richiesta viene inviata all'istanza con il minor numero di connessioni attive. Utile quando le richieste hanno durate molto diverse (es. alcune chiamate REST restituiscono subito, altre impiegano secondi).

#### IP Hash

L'istanza viene scelta in base all'hash dell'IP del client. Lo stesso client va sempre sulla stessa istanza (*sticky session*). Utile se il server REST mantiene stato locale legato al client.

#### Weighted Round Robin

Come il Round Robin, ma le istanze più potenti ricevono un peso maggiore e dunque più richieste proporzionalmente.

### Caratteristiche principali

* **Alta disponibilità (HA):** se un'istanza si guasta, il LB la esclude automaticamente dopo i health check falliti. Il servizio rimane attivo.
* **Scalabilità orizzontale:** per aumentare la capacità si aggiungono nuove istanze senza downtime (scale-out).
* **Health Check:** il LB interroga periodicamente ogni istanza (es. `GET /health`) per verificarne la disponibilità.
* **SSL Termination:** come il proxy, può gestire HTTPS centralmente.
* **Session Persistence:** garantisce che un client venga sempre diretto alla stessa istanza durante una sessione (quando necessario).
* **Funziona a diversi livelli OSI:** L4 (TCP/UDP) o L7 (HTTP) — quest'ultimo permette routing basato sugli URL.

#### Layer 4 vs Layer 7

Un LB di **Livello 4** lavora a livello di trasporto (TCP/UDP): è velocissimo ma non vede il contenuto HTTP. Un LB di **Livello 7** lavora a livello applicativo: può ispezionare header HTTP, cookie, URL e prendere decisioni di routing più intelligenti — avvicinandosi per certi versi a un proxy o gateway.

#### Quando usarlo ✓

* Traffico elevato che una singola istanza non riesce a gestire
* Requisiti di alta disponibilità: nessun singolo punto di fallimento
* Deploy di più istanze dello stesso servizio REST
* Necessità di aggiornamenti senza downtime (rolling update)
* Ambienti cloud con autoscaling automatico

#### Quando evitarlo ✗

* Una sola istanza del servizio (il LB è inutile e aggiunge latenza)
* Servizi con forte stato locale non replicabile tra istanze
* Applicazioni di sviluppo/test locali (overhead non giustificato)
* Il servizio usa WebSocket e il LB L4 non supporta connessioni persistenti

Strumenti comuni: `HAProxy`, `Nginx`, `AWS Elastic Load Balancing`, `Google Cloud Load Balancing`, `Traefik`, `MetalLB` (Kubernetes).

// 04 — Analisi comparativa

## Confronto tra i tre componenti

Nonostante le sovrapposizioni funzionali, i tre componenti hanno responsabilità primarie distinte. La tabella seguente riassume le differenze chiave.

| Caratteristica | Proxy | API Gateway | Load Balancer |
| --- | --- | --- | --- |
| Scopo primario | Intermediazione e filtraggio del traffico | Gestione intelligente delle API | Distribuzione del carico tra istanze |
| Conoscenza delle API | Bassa (generico HTTP) | Alta (conosce endpoint, versioni, authn) | Bassa (lavora su connessioni, non contenuto) |
| Autenticazione | No (o molto basica) | Sì (JWT, OAuth2, API Key) | No |
| Caching | Sì (punto di forza) | Sì (parziale) | No |
| Rate Limiting | No / limitato | Sì (punto di forza) | No |
| Alta disponibilità | Parzialmente | Parzialmente | Sì (punto di forza) |
| Scalabilità orizzontale | No | No (instrada, non scala) | Sì (punto di forza) |
| Health check | No / limitato | Sì (circuit breaker) | Sì (attivo e continuo) |
| Complessità di configurazione | Bassa | Alta | Media |
| Overhead di latenza | Basso | Medio-alto | Molto basso |
| Livello OSI | L7 (HTTP) | L7 (HTTP/API) | L4 (TCP) o L7 (HTTP) |

### Possono coesistere?

Assolutamente sì — e nelle architetture enterprise lo fanno spesso. Una configurazione tipica in produzione potrebbe essere:

INTERNET / CLIENT
richiesta HTTPS

HTTPS

REVERSE PROXY / CDN
SSL termination · caching statico · filtraggio IP · compressione

HTTP interno

API GATEWAY
autenticazione JWT · rate limiting · routing semantico · versioning

richiesta autenticata

LOAD BALANCER
distribuzione Round Robin · health check · failover automatico

SERVER REST
istanza 1 :8080

SERVER REST
istanza 2 :8081

SERVER REST
istanza 3 :8082

risposta HTTP ←

**Principio chiave:** ogni componente rispetta la *separation of concerns*. Il proxy non si preoccupa di quante istanze ci sono; il gateway non sa come distribuire il carico; il load balancer non sa nulla di autenticazione. Questa separazione rende il sistema modulare e mantenibile.

// 05 — Applicazione pratica

## Scenari reali di utilizzo

Vediamo come scegliere il componente giusto in base al contesto applicativo.

🏫

#### Piccolo progetto scolastico

Un'API REST su un singolo server, pochi utenti. **Nessun intermediario necessario.** Al limite, un reverse proxy Nginx per la SSL termination.

🛒

#### E-commerce medio

Traffico variabile, picchi nel weekend. **Load Balancer** con 2–4 istanze + **Reverse Proxy** con caching per le pagine prodotto statiche.

📱

#### App con API pubblica

Sviluppatori terzi usano le API, serve controllo delle quote. **API Gateway** con rate limiting, API key, documentazione OpenAPI.

🏢

#### Architettura enterprise

Decine di microservizi, milioni di richieste/giorno. Tutti e tre i componenti in combinazione: **CDN + Gateway + LB**.

☁️

#### Cloud con autoscaling

Le istanze si aggiungono e rimuovono automaticamente in base al traffico. Il **Load Balancer** cloud-native (es. AWS ALB) si integra con l'autoscaler.

🔒

#### Rete aziendale chiusa

I dipendenti accedono a servizi REST interni. Un **Forward Proxy** controlla e logga il traffico in uscita verso API esterne.

// 06 — Riepilogo

## Sommario e punti chiave

### Proxy

Intermediario generico tra client e server. Il suo punto di forza è il **caching** e l'**anonimizzazione**. Nella variante *reverse proxy* è quasi sempre presente nelle architetture web per gestire SSL e nascondere la rete interna. È semplice da configurare e introduce poca latenza.

### API Gateway

Il "controllore di frontiera" delle API. Conosce profondamente il dominio applicativo e gestisce **autenticazione**, **rate limiting**, **routing semantico** e **versioning**. È il componente giusto quando si espongono API a client diversi o a sviluppatori terzi. Ha costo di configurazione e latenza superiori rispetto al proxy.

### Load Balancer

Il garante della **scalabilità** e dell'**alta disponibilità**. Non sa nulla di autenticazione o caching: il suo unico scopo è distribuire le richieste in modo equo tra le istanze e rilevare quelle non disponibili. È indispensabile in qualsiasi sistema che debba reggere traffico significativo o garantire continuità di servizio.

**Regola pratica:** quando si progetta un'architettura, si inizia sempre dal Load Balancer se si hanno più istanze, si aggiunge il Gateway se le API sono esposte all'esterno o a client eterogenei, e si considera il Proxy se serve caching o controllo del traffico. Non bisogna aggiungere complessità prima che sia necessaria — il principio YAGNI (*You Aren't Gonna Need It*) vale anche in architettura di rete.

### Domande di verifica

* Qual è la differenza fondamentale tra un forward proxy e un reverse proxy?
* Perché un API Gateway è preferibile a un semplice proxy in un'architettura a microservizi?
* Spiega il concetto di health check e perché è fondamentale per un load balancer.
* In quale scenario useresti un LB di Livello 4 invece di uno di Livello 7?
* Progetta un'architettura per un'applicazione con 3 microservizi REST, traffico di 10.000 richieste/minuto e API pubblica con rate limiting. Quali componenti sceglieresti e perché?

Informatica · Classe 5ª · Architetture per servizi REST  ·  Documento didattico
