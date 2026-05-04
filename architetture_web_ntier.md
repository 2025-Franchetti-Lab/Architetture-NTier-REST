Informatica · Classe 5ª · Reti e Architetture

# Architetture Web *N-Tier*

Modelli a livelli per applicazioni web
//
Anno scolastico 2025–26

[Introduzione](#intro)
[1-Tier](#tier1)
[2-Tier](#tier2)
[3-Tier](#tier3)
[N-Tier](#ntier)
[Macchine server](#server)
[Scalabilità](#scalabilita)
[Sommario](#sommario)

// 00 — Contesto

## Le architetture software a più livelli

Le architetture per il web si basano su una **modellizzazione a livelli**, esattamente come il modello ISO/OSI e l'architettura TCP/IP che abbiamo già studiato. Nel contesto dei sistemi web questi livelli prendono il nome di **tier** (strati) e le architetture si chiamano **N-tier** in base al numero N di strati progettati.

Sul lato server, i tier fondamentali sono tre. Sul lato client è sufficiente un unico tier — il *Presentation o Client Tier* — rappresentato tipicamente dal browser che effettua richieste HTTP.

LATO CLIENT

BROWSER
Presentation Tier
(Client Tier)

HTTP request

HTTP response

LATO SERVER

PRESENTATION
TIER
Web Server · Front-end

BUSINESS
TIER
App Server · Logica

DATA
TIER
DBMS · Back-end

I tier vengono fisicamente realizzati da macchine server specializzate. Le architetture N-tier prevedono
strati aggiuntivi oltre i 3 fondamentali (Security, Networking, Data Access, Data Storage).

1-Tier

2-Tier

3-Tier

N-Tier
← Evoluzione storica delle architetture

I tier vengono fisicamente realizzati mediante macchine server in grado di svolgere i compiti richiesti da ogni strato. Vediamo come queste architetture si sono evolute nel tempo.

// 01 — Prima architettura

Architettura 1-Tier

## Informatica centralizzata

Nell'architettura **1-tier** tutte le componenti — Presentation, Business e Data — fanno parte di un unico tier fisico. È lo scenario tipico dell'informatica centralizzata, dove i client sono semplici terminali di I/O che si collegano a un mainframe che elabora tutto.

### Come funziona

I client (terminali "stupidi", privi di capacità di elaborazione propria) inviano input al mainframe. Il mainframe elabora la logica applicativa, accede ai dati e restituisce l'output ai terminali. Tutto avviene in un unico sistema centralizzato. Questa architettura è stata dominante fino agli anni Novanta del secolo scorso.

MAINFRAME

PRESENTATION TIER
Gestione interfaccia e rendering

BUSINESS TIER
Logica applicativa ed elaborazione

DATA TIER
Archiviazione e accesso ai dati

TERMINALE
I/O

TERMINALE
I/O

TERMINALE
I/O

TERMINALE
I/O

TERMINALE
I/O

TERMINALE
I/O

↑ tutte le componenti in un unico tier fisico

### Problematiche dell'architettura 1-tier

* **Scalabilità:** difficoltà a modificare il quantitativo o il tipo di dati trattati — bisogna intervenire sull'unico sistema centrale.
* **Portabilità:** difficoltà a spostare il sistema in un altro ambiente hardware o software.
* **Aggiornamento:** difficoltà a migrare verso release aggiornate; ogni modifica impatta l'intero sistema.
* **Flessibilità:** impossibile modificare una singola componente senza ripercussioni sulle altre — tutto è strettamente accoppiato.

**Contesto storico:** questa architettura è stata il paradigma dominante fino agli anni Novanta. Oggi sopravvive solo in contesti molto specifici (legacy bancari, sistemi di controllo industriale) dove la migrazione è troppo costosa o rischiosa.

// 02 — Seconda architettura

Architettura 2-Tier

## Il modello Client-Server

L'architettura **2-tier** è il modello *Client-Server*, sviluppatosi a partire dagli anni Novanta diventando il paradigma di Internet. È lo scenario tipico dell'**informatica distribuita**, dove le applicazioni e i dati possono risiedere in remoto su più macchine, anche in luoghi fisicamente diversi.

### Come funziona

Il client non è più un terminale stupido ma un **host** in grado di elaborare autonomamente i dati. Quando ha necessità, richiede servizi in rete al server. I server in ascolto elaborano le richieste e forniscono i risultati. L'architettura separa il Data Tier dagli strati Presentation e Business, che invece risiedono entrambi sul client.

CLIENT HOST

PRESENTATION TIER
Browser / GUI / Interfaccia utente

BUSINESS TIER
Logica applicativa sul client

Il client elabora autonomamente la logica
e richiede i dati al server quando necessario

richiesta dati
SQL / query / API

risultati / dati

SERVER

DATA TIER
DBMS · Database · Archiviazione dati

DATABASE

Presentation e Business sul client · Data Tier sul server
Tipico: applicazioni desktop anni '90, client SQL thick

### Vantaggi rispetto alla 1-tier

Il Data Tier diventa **indipendente** e può essere su una macchina dedicata. I client possono distribuirsi geograficamente (CED aziendali, data center con server farm). Il server è in grado di gestire più client contemporaneamente.

**Limitazione:** Presentation e Business risiedono ancora insieme sul client. Questo mantiene un forte accoppiamento e problemi di scalabilità, portabilità e aggiornamento quasi invariati rispetto all'architettura 1-tier.

// 03 — Terza architettura

Architettura 3-Tier

## La separazione completa dei livelli

Progettando un **Middle Tier** intermedio, sempre sul modello Client-Server, si ottiene un'architettura **3-tier** che rende *indipendenti* i tre strati fondamentali. È l'architettura tipica degli anni Duemila, tuttora largamente utilizzata.

### Come funziona

Il tier intermedio — tipicamente il Business Tier realizzato da un *Application Server* — agisce **da server** nei confronti del Presentation Tier e **da client** nei confronti del Data Tier. Ogni strato è realizzato su macchine server differenti e separate.

CLIENT

PRESENTATION TIER
Browser / Thin client
Invia richiesta HTTP
Riceve risposta HTML/JSON
Ruolo: CLIENT
verso il Web Server

HTTP req

HTTP resp

WEB SERVER

PRESENTATION TIER
Front-end · Rendering
Gestisce req. HTTP
Serve pagine statiche
Inoltra req. all'App Server
SERVER vs client
CLIENT vs App Server

req. interna

risultato

APP SERVER

BUSINESS TIER
Logica applicativa
Elabora req. dinamiche
Accede al DBMS
Risponde al Web Server
SERVER vs Web Server
CLIENT vs DBMS

DBMS

DATA TIER
Database

DATABASE

Ogni tier è su una macchina fisica separata · Separazione completa delle responsabilità
Il tier intermedio (Middle Tier) funge alternativamente da client e da server

### Vantaggi della separazione in tre tier

#### Problemi risolti ✓

* Ogni tier è modificabile indipendentemente
* Possibile aggiornare l'App Server senza toccare DB o client
* Distribuzione su macchine hardware diverse e ottimizzate
* Scalabilità migliorata per ogni tier separatamente
* Portabilità: ogni livello può migrare autonomamente

#### Esempi di implementazione

* Web Server: Apache, Nginx
* App Server: Tomcat, Node.js, .NET
* DBMS Server: MySQL, PostgreSQL, Oracle
* Tipico sito web anni 2000 (LAMP stack)

// 04 — Architetture moderne

Architettura N-Tier

## Multi-Tier Architecture

Le architetture **N-tier** (o Multi-Tier) sono le più attuali e prevedono diversi strati intermedi, uno per ogni funzione sviluppata. Ogni tier agisce sia da client sia da server a seconda della direzione della comunicazione.

### I tier aggiuntivi

Oltre ai tre tier fondamentali (Presentation, Business, Data), le architetture aziendali moderne prevedono strati specializzati:

Tier 0

CLIENT TIER — Browser, App mobile, dispositivo IoT

Tier 1

NETWORKING TIER — Garantisce connettività e privacy del client (Proxy, CDN, VPN)

Tier 2

SECURITY TIER — Monitoraggio e controllo degli accessi alla rete (Firewall, AAA Server)

Tier 3

PRESENTATION TIER — Front-end, rendering, gestione interfaccia (Web Server)

Tier 4

BUSINESS TIER — Logica applicativa, elaborazione, funzionalità (Application Server)

Tier 5

DATA ACCESS TIER — Autorizzazioni e accesso ai dati sensibili (DAL, ORM)

Tier 6

DATA STORAGE TIER — Operazioni fisiche sul database (DBMS Server, replica)

### Schema di un'architettura 4-tier in contesto sicurezza

INTERNET / CLIENT
Browser · App mobile · IoT

HTTPS

TIER NETWORKING · Proxy Server / CDN
SSL termination · caching contenuti statici · compressione gzip · anonimizzazione IP

TIER SECURITY · AAA Server / Firewall
Authentication · Authorization · Accounting · filtraggio pacchetti · IDS/IPS

TIER PRESENTATION · Web Server
gestione richieste HTTP · routing · serving pagine statiche · interfaccia grafica

TIER BUSINESS · Application Server
logica applicativa · elaborazione dati · regole di business · accesso al Data Tier

TIER DATA · DBMS Server

◀ risposta percorre il percorso inverso

**Principio fondamentale:** in ogni architettura N-tier, il tier intermedio agisce *da server* verso il tier precedente e *da client* verso il tier successivo. Questo disaccoppiamento è la chiave che rende ogni strato modificabile indipendentemente.

// 05 — Infrastruttura fisica

## Le macchine server

I tier vengono fisicamente realizzati mediante macchine server specializzate. Ciascuna risponde a requisiti funzionali precisi e può ospitare uno o più tier dell'architettura.

| Tipo di server | Tier realizzato | Funzione principale | Esempi software |
| --- | --- | --- | --- |
| Web Server | Presentation | Riceve, gestisce e risponde alle richieste HTTP/HTTPS provenienti dalla rete (servizi esterni) | Nginx, Apache |
| Application Server | Business | Ricerca informazioni ed elabora dati; offre servizi interni all'architettura | Tomcat, Node.js, .NET, Django |
| DBMS Server | Data | Gestisce il database; può sdoppiarsi in Data Access Tier e Data Storage Tier | MySQL, PostgreSQL, Oracle, MongoDB |
| Proxy Server | Presentation + Security/Networking | Intermediario tra client e server; caching, filtraggio, SSL termination, anonimizzazione | Nginx, Squid, HAProxy, Varnish |
| AAA Server | Security | Authentication, Authorization, Accounting: gestisce identità, permessi e tracciamento accessi | RADIUS, LDAP, Active Directory, Keycloak |

**Nota:** un unico server fisico può ospitare più tier (es. Web Server + App Server sulla stessa macchina), ma questa scelta riduce l'indipendenza e la scalabilità. Nelle architetture enterprise ogni tier ha la propria infrastruttura dedicata.

// 06 — Crescita del sistema

Scalabilità

## Scalabilità orizzontale e verticale

La **scalabilità** di un'applicazione è la capacità di aumentarne il throughput in proporzione all'hardware impiegato per ospitarla. Si distingue in due strategie fondamentali: *scale-out* (orizzontale) e *scale-up* (verticale).

### Scalabilità orizzontale — Scale-out

Si ottiene aumentando il **numero di nodi** che ospitano l'applicazione, nodi del tutto simili tra loro in termini di CPU e memoria. In questo modo è possibile gestire in parallelo il carico di lavoro. Raddoppiando il numero di server si raddoppia la capacità di gestire utenti contemporanei. Il principale vantaggio è la maggior **fault-tolerance**: un guasto in un nodo non pregiudica il funzionamento dell'intero servizio.

SCALE-OUT — Scalabilità orizzontale

LOAD BALANCER
distribuisce il traffico

APP SERVER
istanza 1
CPU: 4 core · RAM: 8 GB
● attivo

APP SERVER
istanza 2
CPU: 4 core · RAM: 8 GB
● attivo

APP SERVER
istanza 3
CPU: 4 core · RAM: 8 GB
● attivo

APP SERVER
istanza 4 (aggiunta)
CPU: 4 core · RAM: 8 GB
+ scale-out

Tutti i nodi sono identici · Un guasto non blocca il servizio · Tipico: Web Server, App Server
Vantaggio: fault-tolerance · Svantaggio: il servizio deve supportare la distribuzione

### Scalabilità verticale — Scale-up

Si ottiene aumentando le **risorse di un singolo nodo**: CPU con frequenza maggiore, più memoria RAM, dischi più veloci (SSD NVMe), GPU dedicata. L'obiettivo è incrementare le prestazioni dell'intero sistema su una sola macchina. Lo scale-up presenta costi maggiori rispetto allo scale-out perché acquistare macchine server più performanti è più costoso che aggiungere macchine equivalenti.

SCALE-UP — Scalabilità verticale

PRIMA — Server base

CPU: 4 core
2.4 GHz

RAM: 16 GB
DDR4

Disco: HDD
1 TB · 100 MB/s

Rete: 1 Gbps
singola NIC
capacità: ~200 req/s

→
UPGRADE

DOPO — Server potenziato

CPU: 16 core
3.8 GHz

RAM: 128 GB
DDR5 ECC

Disco: NVMe
2 TB · 7 GB/s

Rete: 25 Gbps
bonding 2 NIC
capacità: ~2000 req/s

### Confronto tra le due strategie

| Caratteristica | Scale-out | Scale-up |
| --- | --- | --- |
| Metodo | Aggiungere nodi identici | Potenziare l'hardware del singolo nodo |
| Costo | Inferiore (hardware commodity) | Superiore (hardware enterprise) |
| Fault-tolerance | Alta — un nodo giù, gli altri continuano | Bassa — SPOF, se cade il server il servizio è fermo |
| Limiti fisici | Quasi illimitato (aggiunta nodi) | Limitato dall'hardware disponibile |
| Complessità progettuale | Alta — il servizio deve supportare la distribuzione | Bassa — architettura invariata |
| Tipico uso | Web Server, App Server, siti statici | Database Server (DBMS), sistemi real-time |
| Downtime per scalare | Zero — nodi aggiunti a caldo | Sì — richiede solitamente riavvio |

**Nota sul tier del database:** la scalabilità verticale è tipicamente richiesta nel tier del database, perché scalare orizzontalmente un DBMS relazionale (con transazioni ACID e consistenza forte) è architetturalmente complesso. I database NoSQL sono invece progettati per lo scale-out.

### Ambienti cloud e CED locale

Naturalmente i servizi possono essere proposti sia in ambito **CED locale** (data center proprietario) sia in **cloud** appoggiandosi ai data center di provider come AWS, Azure o Google Cloud. In cloud, lo scale-out è semplificato dall'*autoscaling* automatico: il sistema aggiunge o rimuove istanze in base al traffico rilevato.

// 07 — Riepilogo

## Sommario e punti chiave

1-Tier

#### Informatica centralizzata

Tutti i tier su un unico sistema (mainframe). Terminali stupidi per I/O. Problemi di scalabilità, portabilità, flessibilità. Paradigma pre-1990.

2-Tier

#### Client-Server

Presentation e Business sul client (host), Data sul server. Informatica distribuita, paradigma Internet anni '90. Separazione parziale.

3-Tier

#### Middle Tier

Tre livelli fisicamente separati: Web Server, App Server, DBMS. Ogni tier modificabile indipendentemente. Paradigma anni 2000.

N-Tier

#### Multi-Tier moderno

Strati aggiuntivi per sicurezza, networking, data access. Architetture enterprise con microservizi, cloud, container.

Scale-out

#### Scalabilità orizzontale

Più nodi identici + load balancer. Alta fault-tolerance, costo contenuto. Tipico per Web Server e App Server.

Scale-up

#### Scalabilità verticale

Hardware più potente per il singolo nodo. Più semplice ma costoso e con SPOF. Tipico per DBMS Server.

### Domande di verifica

* Qual è la differenza fondamentale tra architettura 1-tier e 2-tier? Quali problemi restano irrisolti nel modello 2-tier?
* Perché il Middle Tier nell'architettura 3-tier si dice che "agisce sia da client sia da server"? Spiega con un esempio.
* Elenca quattro tier presenti in un'architettura N-tier moderna, indicando per ciascuno la funzione svolta e il tipo di server che lo realizza.
* Confronta scale-out e scale-up: in quale tier applicheresti ciascuna strategia e perché?
* Un sito di e-commerce subisce picchi di traffico durante le promozioni. Quale tipo di scalabilità consiglieresti e come struttureresti l'architettura N-tier?

Informatica · Classe 5ª · Architetture Web N-Tier  ·  Documento didattico  ·  A.S. 2025–26
