Informatica 5ª — Infrastrutture Cloud

# Virtual Machines, *Containers* & Serverless Computing

03 — paradigmi di deployment
confronto · pro/contro · casi d'uso

[Introduzione](#intro)
[Virtual Machines](#vm)
[Containers](#containers)
[Serverless](#serverless)
[Confronto](#confronto)
[Quando usare cosa](#casi-uso)

00 — panoramica

## Tre modi di eseguire il software

Quando si distribuisce un'applicazione nel mondo reale, la prima domanda non è *quale linguaggio* usare, ma *dove e come* far girare il codice. Tre paradigmi dominano il panorama attuale: le **Virtual Machines**, i **Containers** e il **Serverless Computing**. Ognuno risolve problemi diversi e comporta compromessi precisi su isolamento, velocità, costo e complessità operativa.

Questa pagina illustra i tre paradigmi con diagrammi architetturali, analisi pro/contro e linee guida pratiche per scegliere quello giusto in base al contesto.

VIRTUAL MACHINE
CONTAINER
SERVERLESS

Hardware fisico

Hypervisor (VMware / KVM)

Guest OS

Libs / Deps

App A
VM 1

Guest OS

Libs / Deps

App B
VM 2

~GB per VM · avvio in minuti

Hardware fisico

Host OS (kernel condiviso)

Container Engine (Docker / containerd)

Libs / Deps

App A
Container 1

Libs / Deps

App B
Container 2
~MB per container · avvio in secondi

Cloud Provider (gestito)

OS · Runtime · Scaling (nascosti)

Event Bus / Trigger

fn handler()

Logica A
Function 1

fn handler()

Logica B
Function 2
nessun server da gestire · pay-per-call

Confronto degli stack architetturali — VM vs Container vs Serverless

01 — paradigma

## Virtual Machines

Una **Virtual Machine** (VM) è un'emulazione software completa di un computer fisico. Grazie a un livello software chiamato *hypervisor*, è possibile eseguire più sistemi operativi guest su un singolo host, con ciascuno che crede di operare su hardware dedicato.

## Come funziona l'Hypervisor

Virtual Machine Monitor

L'hypervisor si frappone tra il hardware fisico e i sistemi operativi guest. Esistono due tipi principali:

TYPE 1 — BARE METAL

Hardware fisico

Hypervisor Type 1 (VMware ESXi, Hyper-V, KVM)

Guest OS 1

App A
VM 1

Guest OS 2

App B
VM 2

TYPE 2 — HOSTED

Hardware fisico

Host OS (Windows / macOS / Linux)

Hypervisor Type 2 (VirtualBox, VMware Workstation)

Guest OS + App A
VM 1

Guest OS + App B
VM 2

#### Caratteristiche principali

* **Isolamento completo** — ogni VM ha il proprio kernel, memoria, storage e stack di rete separati dal host e dalle altre VM.
* **OS guest indipendente** — si può eseguire Windows su un host Linux e viceversa senza alcuna modifica.
* **Snapshot e live migration** — lo stato di una VM può essere salvato e ripristinato in qualsiasi momento, oppure spostato su un altro host a caldo.
* **Overhead elevato** — ogni VM replica un intero OS (tipicamente 1–20 GB di disco, 512 MB–8 GB di RAM solo per il sistema).
* **Avvio lento** — il boot di un sistema operativo completo richiede da 30 secondi a diversi minuti.
* **Consolidazione server** — più VM sullo stesso hardware riducono il numero di macchine fisiche necessarie.

#### Vantaggi

* Isolamento di sicurezza massimo (kernel separato)
* Compatibilità con qualsiasi OS guest
* Snapshot, rollback e live migration
* Ideale per workload legacy
* Facilmente integrabile con sistemi di monitoring esistenti
* Supporta ambienti multi-tenant ad alto isolamento

#### Svantaggi

* Overhead elevato (CPU, RAM, disco per ogni OS)
* Boot time lento (minuti, non secondi)
* Immagini di grandi dimensioni (GB)
* Scalabilità orizzontale lenta e costosa
* Provisioning manuale complesso
* Consumo energetico maggiore rispetto a soluzioni più leggere

02 — paradigma

## Containers

Un **container** è un'unità di deployment leggera che impacchetta il codice dell'applicazione insieme alle sue dipendenze (librerie, runtime, variabili d'ambiente), isolandola dal sistema host ma *condividendo il kernel del sistema operativo*. Docker è lo strumento più diffuso; Kubernetes è l'orchestratore di riferimento per ambienti di produzione.

## Architettura dei Container

Docker · Kubernetes · OCI

I container utilizzano due funzionalità del kernel Linux — **namespaces** (isolamento di risorse) e **cgroups** (limitazione di risorse) — per creare ambienti di esecuzione separati senza la necessità di emulare hardware.

Hardware fisico

Host OS — kernel Linux (namespaces + cgroups)

Container Runtime (Docker Engine / containerd / CRI-O)

CONTAINER 1

Libs / Deps (layer)

Runtime (Node 20)

App A

CONTAINER 2

Libs / Deps (layer)

Runtime (Python 3.12)

App B

CONTAINER 3

PostgreSQL 16

Volume persistente

Database

KUBERNETES
Scheduling · Scaling
Service Discovery
Load Balancing
Rolling Updates

kernel Linux condiviso — overhead minimo — avvio in millisecondi

#### Caratteristiche principali

* **Kernel condiviso** — i container condividono il kernel del host OS, eliminando la necessità di emulare hardware o eseguire un OS completo per ogni istanza.
* **Image immutabili a layer** — le immagini Docker sono costruite a strati (Union File System); i layer comuni sono condivisi, riducendo drasticamente spazio e banda.
* **Portabilità** — "build once, run anywhere": un container eseguito in sviluppo funziona identicamente in staging e produzione.
* **Avvio in millisecondi** — non essendoci boot di un OS, i container si avviano in pochi centesimi di secondo.
* **Orchestrazione** — Kubernetes gestisce deployment, scaling automatico, service discovery e rolling updates su cluster di migliaia di nodi.
* **Isolamento parziale** — l'isolamento è meno forte rispetto alle VM; vulnerabilità del kernel host possono potenzialmente impattare tutti i container.

#### Vantaggi

* Leggerissimi (MB vs GB delle VM)
* Avvio quasi istantaneo
* Alta densità — decine di container per host
* Portabilità totale tra ambienti
* Pipeline CI/CD rapide e riproducibili
* Orchestrazione avanzata con Kubernetes

#### Svantaggi

* Isolamento di sicurezza inferiore alle VM
* Solo Linux natively (Windows containers con limitazioni)
* Gestione dello stato (storage persistente) complessa
* Kubernetes ha una curva di apprendimento ripida
* Debug di problemi di rete inter-container non banale
* Container image mal configurate sono un rischio di sicurezza

03 — paradigma

## Serverless Computing

Il termine **Serverless** non significa "nessun server", ma che lo sviluppatore non deve *gestire* i server. Il cloud provider si occupa di provisioning, scaling, patching e disponibilità dell'infrastruttura. Il modello più diffuso è il **Function as a Service (FaaS)**: il codice viene eseguito in risposta a eventi e fatturato al millisecondo di esecuzione.

## Architettura Event-Driven

FaaS · AWS Lambda · Azure Functions · Google Cloud Run

In un'architettura serverless, le funzioni sono stateless, effimere e si attivano in risposta a *trigger*: richieste HTTP, messaggi su code, eventi da database, timer schedulati, ecc.

SORGENTI EVENTO

HTTP Request (API GW)

Message Queue (SQS)

DB Trigger (DynamoDB)

Cron / Schedule

CLOUD PROVIDER

Provisioning automatico

Scaling 0→∞ istantaneo

Patching OS & Runtime

Alta disponibilità (multi-AZ)

FUNZIONI

processOrder(event)
⏱ max 15 min · stateless

sendNotification(event)
⏱ max 15 min · stateless

generateReport(event)
⏱ max 15 min · stateless

fatturazione al millisecondo · zero costo a zero richieste

#### Caratteristiche principali

* **Zero gestione server** — nessun provisioning, patching, monitoraggio dell'infrastruttura. Il provider gestisce tutto.
* **Scaling automatico 0→∞** — da zero istanze (nessun costo a riposo) a migliaia di istanze parallele in millisecondi, senza configurazione.
* **Modello pay-per-use** — si paga solo il tempo CPU effettivo (al millisecondo), non per risorse allocate ma inutilizzate.
* **Stateless per design** — ogni invocazione è indipendente; lo stato deve essere esternalizzato su database, cache o object storage.
* **Cold start** — la prima invocazione dopo un periodo di inattività può richiedere qualche centinaio di ms–secondi per inizializzare l'ambiente.
* **Limiti di esecuzione** — AWS Lambda ha un timeout massimo di 15 minuti; non adatto a processi long-running.

#### Vantaggi

* Zero overhead operativo (no DevOps per l'infra)
* Costo zero a zero traffico
* Scaling automatico illimitato
* Time-to-market rapidissimo
* Fault tolerance integrata dal provider
* Focus totale sulla logica di business

#### Svantaggi

* Cold start latency (specialmente con Java/.NET)
* Timeout massimo (es. 15 min su Lambda)
* Vendor lock-in elevato
* Debug e testing locale più complessi
* Costo alto ad alto volume continuativo
* Gestione dello stato complessa (necessita servizi esterni)

**Cold Start:** quando una funzione non viene invocata per un certo periodo, il provider dealloca le risorse. La successiva invocazione ("cold start") deve caricare il runtime e il codice ex novo. Linguaggi compilati come Java o .NET possono avere cold start di 1–3 secondi; Go e linguaggi interpretati leggeri come Python e Node.js partono in 100–300 ms. Le "provisioned concurrency" (Lambda) e i container pre-riscaldati riducono questo problema a scapito del costo.

04 — analisi comparativa

## Confronto diretto

La tabella seguente riassume le differenze chiave tra i tre paradigmi su dimensioni tecniche, operative ed economiche.

| Dimensione | Virtual Machine | Container | Serverless |
| --- | --- | --- | --- |
| Unità di deploy | Immagine VM (GB) | Immagine Docker (MB) | Funzione / Handler |
| Tempo di avvio | Minuti | Secondi / ms | ms (warm) / 1–3 s (cold) |
| Isolamento | Massimo (kernel separato) | Medio (kernel condiviso) | Gestito dal provider |
| Overhead risorse | Elevato (OS per VM) | Basso (solo app + libs) | Nessuno lato utente |
| Scalabilità | Manuale / lenta | Automatica (con K8s) | Automatica 0→∞ |
| Portabilità | Bassa (dipende dall'hypervisor) | Alta (OCI standard) | Bassa (vendor lock-in) |
| Gestione OS | A carico dell'utente | A carico dell'utente | A carico del provider |
| Stato applicativo | Stateful nativo | Volume persistente | Stateless (esterno) |
| Modello di costo | Pay per VM allocata | Pay per nodo/cluster | Pay per invocazione/ms |
| Curva di apprendimento | Media | Alta (K8s) | Bassa (ma debugging hard) |
| Sicurezza | Forte isolamento HW | Dipende dalla configurazione | Dipende dal provider |
| Esempi cloud | AWS EC2, Azure VM, GCE | EKS, AKS, GKE | Lambda, Azure Fn, Cloud Run |

POSIZIONAMENTO — Velocità vs Isolamento vs Costo operativo

Velocità di avvio →
Isolamento →

lento
veloce

basso
alto

VM
~minuti

costo: fisso / allocato

Container
~secondi
costo: per nodo

Serverless
~ms
costo: pay-per-call

Virtual Machine

Container

Serverless
raggio = overhead operativo

05 — linee guida

## Quando usare cosa

La scelta tra VM, container e serverless non è assoluta: molte architetture moderne le combinano. Tuttavia, per ogni scenario esiste un paradigma naturalmente più adatto.

### Virtual Machines — scegli quando…

VM

#### Applicazioni legacy

Software che dipende da un OS specifico, da librerie di sistema particolari o che non è stato progettato per il deployment containerizzato.

VM

#### Sicurezza e compliance rigorosa

Ambienti regolamentati (healthcare, finance, governo) dove l'isolamento a livello hardware è un requisito obbligatorio.

VM

#### Workload stateful intensivi

Database ad alte prestazioni, sistemi che richiedono accesso diretto all'hardware (GPU, FPGA) o configurazioni di rete avanzate.

VM

#### Ambienti di sviluppo isolati

Team che necessitano di replicare ambienti OS completi per test, sviluppo o demo senza impattare il sistema host.

### Containers — scegli quando…

Container

#### Microservizi

Architetture a microservizi dove ogni servizio ha le proprie dipendenze, scala indipendentemente e viene rilasciato autonomamente.

Container

#### Pipeline CI/CD

Ambienti di build e test riproducibili al 100%: "funziona sul mio PC" diventa un problema del passato.

Container

#### Portabilità multi-cloud

Applicazioni che devono girare su AWS, Azure, GCP e on-premise senza modifiche, sfruttando Kubernetes come strato di astrazione.

Container

#### Scaling rapido e denso

Servizi web con traffico variabile che beneficiano dell'alta densità (centinaia di container per host) e dell'avvio istantaneo.

### Serverless — scegli quando…

Serverless

#### Event processing

Elaborazione di file caricati su S3, messaggi su code SQS/Kafka, webhook, notifiche push: ogni evento attiva una funzione.

Serverless

#### Backend per applicazioni mobile

API semplici con traffico imprevedibile. Costo zero nelle ore di basso utilizzo, scaling automatico nei picchi.

Serverless

#### Automazioni e job schedulati

Report notturni, cleanup del database, invio di email periodiche: task brevi che non giustificano un server sempre attivo.

Serverless

#### Startup e MVP

Progetti in fase iniziale dove il team è piccolo, il budget limitato e si vuole validare il prodotto senza gestire infrastruttura.

### Albero decisionale

Nuovo deployment

Hai bisogno di un OS specifico
o di isolamento hardware?

Sì

VM

No

Il processo è short-lived (<15 min)
e attivato da eventi?

Sì

Il traffico è molto variabile
o hai budget limitato?

Sì

Serverless

No

Container (+ K8s)

No

nota: le architetture reali spesso combinano tutti e tre i paradigmi

**Nota architetturale:** nei sistemi moderni è normale trovare i tre paradigmi co-esistenti. Ad esempio: un database PostgreSQL su VM dedicata, i microservizi applicativi su container orchestrati da Kubernetes, e le elaborazioni asincrone (ridimensionamento immagini, invio email, report) su Lambda. Scegliere il paradigma giusto per ogni componente è un'abilità fondamentale dell'ingegnere cloud.

Informatica 5ª — Architetture Cloud & Infrastrutture · 2025–2026

[← Torna all'indice](index.html)
