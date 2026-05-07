# Architettura N-Tier su Cloud

Confronto implementativo su **AWS**, **Microsoft Azure** e **Google Cloud Platform**
con servizi consigliati, stima dei costi mensili e analisi punti di forza/debolezza

## 1. Architettura di riferimento

L'architettura analizzata è un'architettura **N-Tier** per un servizio online composta da sei livelli distinti che separano le responsabilità e garantiscono scalabilità, sicurezza e resilienza.

REVERSE PROXY
reverse proxy

API GATEWAY
api gateway

LOAD BALANCER
load balancer

APP SERVER
generic service

CACHE
redis

DATABASE
relazionale

| Livello | Ruolo |
| --- | --- |
| **Reverse Proxy** | Punto d'ingresso pubblico; gestisce TLS termination, CDN, WAF e DDoS protection |
| **API Gateway** | Routing, autenticazione, rate limiting, trasformazione richieste e logging centralizzato |
| **Load Balancer** | Distribuzione del traffico tra le istanze del servizio applicativo (L4/L7) |
| **App Server** | Logica di business del microservizio/REST API (stateless) |
| **Database (Oracle / SQL Server / MySQL)** | Persistenza dati relazionali con replica e failover automatico (Oracle su AWS, SQL Server su Azure, MySQL su GCP) |
| **Cache (Redis)** | Cache in-memory per ridurre la latenza e il carico sul DB |

## 2. Amazon Web Services (AWS)

AWS VPC

CloudFront + WAF
Reverse Proxy
CDN / DDoS / TLS

Amazon API Gateway
API Gateway
REST / HTTP / WebSocket

ALB (L7)
Load Balancer
Application LB / NLB

ECS Fargate / EC2
App Server
Container / Lambda

ElastiCache for Redis
Cache Server
Cluster Mode / Multi-AZ

Amazon RDS for Oracle
Database Oracle
Multi-AZ / SE2 / EE / BYOL

☁ Amazon Web Services

| Livello N-Tier | Servizio AWS | Note |
| --- | --- | --- |
| Reverse Proxy | **Amazon CloudFront** + AWS WAF + AWS Shield | CDN globale, protezione DDoS, TLS termination, caching edge |
| API Gateway | **Amazon API Gateway** (HTTP API o REST API) | Rate limiting, JWT/Cognito auth, Lambda authorizer, usage plans |
| Load Balancer | **Application Load Balancer (ALB)** | Routing path/host-based, sticky sessions, target groups, health check |
| App Server | **Amazon ECS Fargate** / EC2 Auto Scaling / Lambda | Container serverless con auto-scaling; alternativa: EKS (Kubernetes) |
| Database | **Amazon RDS for Oracle** | Supporto Oracle SE2 (License Included o BYOL) e Oracle EE (solo BYOL); Multi-AZ con standby sincrono, automated backups, read replica, Oracle Data Guard integrato; compatibilità nativa PL/SQL, stored procedure e Oracle-specific features |
| Cache | **Amazon ElastiCache for Redis** | Cluster Mode, Multi-AZ, encryption at rest/in transit |

📌 Servizi complementari consigliati: **AWS Secrets Manager** (gestione credenziali), **Amazon Route 53** (DNS), **AWS Certificate Manager** (certificati TLS gratuiti), **Amazon CloudWatch** (monitoring), **AWS X-Ray** (tracing distribuito).

### Stima costi mensili (carico medio: ~1000 req/s, 2 AZ)

| Servizio | Configurazione | Costo stimato/mese |
| --- | --- | --- |
| CloudFront + WAF | 1 TB transfer out, 10M req | ~$60–90 |
| API Gateway (HTTP API) | 30M richieste | ~$30–50 |
| ALB | 2 istanze, LCU medio | ~$30–45 |
| ECS Fargate | 4 task × 1vCPU / 2GB RAM | ~$120–180 |
| RDS Oracle SE2 – License Included (db.m5.large) | Multi-AZ, 100 GB SSD | ~$350–500 |
| RDS Oracle SE2 – BYOL (db.m5.large) | Multi-AZ, 100 GB SSD | ~$150–200 *(+ costo lic. Oracle)* |
| ElastiCache Redis (cache.t3.medium) | 2 nodi cluster | ~$70–100 |
| Data transfer, monitoring, misc | – | ~$40–60 |

📌 **Licensing Oracle su RDS:** la modalità *License Included* (SE2) include la licenza Oracle nel prezzo orario AWS e non richiede contratti Oracle separati; la modalità *BYOL* (Bring Your Own License) consente di usare licenze Oracle già possedute e riduce il costo cloud, ma richiede la gestione del contratto di supporto Oracle separatamente.

💰 Totale stimato: $650 – $975 / mese (License Included)  |  $450 – $635 / mese (BYOL)

#### ✅ Punti di Forza

* Ecosistema più maturo e ampio del mercato (200+ servizi)
* CloudFront ha la rete edge più capillare (600+ PoP)
* RDS for Oracle supporta Oracle SE2 ed EE con compatibilità nativa PL/SQL, procedure e trigger Oracle
* Modalità BYOL permette di riutilizzare licenze Oracle esistenti, abbattendo il costo cloud
* Oracle Data Guard Multi-AZ garantisce failover automatico e sincrono
* Fargate elimina la gestione del cluster EC2
* Documentazione eccellente e community vastissima

#### ⚠️ Punti di Debolezza

* Costo elevato in modalità License Included (SE2 su db.m5.large Multi-AZ ~$350–500/mese)
* Licenze Oracle EE supportate solo in modalità BYOL, con complessità contrattuale
* Curva di apprendimento ripida: molte opzioni e servizi
* API Gateway REST ha latenza aggiuntiva percepibile
* ElastiCache non ha tier serverless (provisioning sempre richiesto)

## 3. Microsoft Azure

Azure Virtual Network (VNet)

Azure Front Door
Reverse Proxy
CDN + WAF + DDoS Std

Azure API Management
API Gateway
Policies / OAuth2 / Portal

Application Gateway v2
Load Balancer L7
WAF integrato / Autoscale

Container Apps / AKS
App Server
Serverless containers / K8s

Azure Cache for Redis
Cache Server
Standard/Premium / Geo-rep

Azure SQL Database
Database SQL Server
General Purpose / Bus. Crit.

☁ Microsoft Azure

| Livello N-Tier | Servizio Azure | Note |
| --- | --- | --- |
| Reverse Proxy | **Azure Front Door** (Standard/Premium) + Azure WAF | Anycast globale, SSL offload, routing basato su latenza, DDoS Protection Standard |
| API Gateway | **Azure API Management (APIM)** | Developer Portal integrato, policy engine potente, subscription keys, OAuth2/OIDC |
| Load Balancer | **Azure Application Gateway v2** | WAF integrato (OWASP), autoscale, SSL termination, Cookie-based affinity |
| App Server | **Azure Container Apps** / AKS / App Service | Container Apps per serverless (KEDA scaling); AKS per K8s gestito |
| Database | **Azure SQL Database** (SQL Server managed) | HA automatico, PITR, elastic pool, opzione serverless; supporto T-SQL completo; Azure Hybrid Benefit riduce i costi con licenze SQL Server esistenti |
| Cache | **Azure Cache for Redis** | Tier Basic/Standard/Premium; geo-replica e clustering nel tier Premium |

📌 Servizi complementari consigliati: **Azure Key Vault** (segreti/certificati), **Azure DNS**, **Azure Monitor + Log Analytics**, **Application Insights** (APM), **Microsoft Entra ID** (identità), **Azure Defender** (sicurezza).

### Stima costi mensili (carico medio: ~1000 req/s, 2 zone)

| Servizio | Configurazione | Costo stimato/mese |
| --- | --- | --- |
| Azure Front Door Standard + WAF | 1 TB transfer, 10M req | ~$65–100 |
| APIM (Developer tier) | 1 unit — solo dev/test | ~$50 |
| APIM (Standard tier) | 1 unit — produzione | ~$700 |
| Application Gateway v2 | Small, 2 capacity unit | ~$45–65 |
| Container Apps | 4 replica, 1 vCPU / 2GB | ~$110–160 |
| Azure SQL Database (General Purpose, 2 vCore) | HA, 100 GB SSD | ~$125–160 |
| Azure Cache for Redis (C1 Standard) | 2 nodi | ~$75–100 |
| Monitoring, networking, misc | – | ~$40–60 |

💰 Totale stimato: $510 – $645 / mese (Developer APIM) oppure $1.155 – $1.345 / mese (Standard APIM)

#### ✅ Punti di Forza

* Integrazione nativa con ecosistema Microsoft (Active Directory, Office 365)
* APIM tra i più completi: developer portal, mock, test integrato
* Azure Container Apps: autoscaling KEDA su eventi (code, HTTP, cron)
* Azure SQL Database: motore SQL Server completamente gestito, con T-SQL nativo e compatibilità piena con applicazioni .NET/enterprise
* Ottimo per aziende già nel mondo Microsoft/Enterprise
* Azure Hybrid Benefit riduce sensibilmente i costi per chi ha già licenze SQL Server

#### ⚠️ Punti di Debolezza

* APIM Standard tier molto costoso (~$700/mese per unità)
* Naming e navigazione del portale spesso confusi
* Documentazione a volte frammentata o datata
* Disponibilità regionale inferiore a AWS in alcune aree
* Application Gateway non è un CDN (dipende da Front Door per edge)

## 4. Google Cloud Platform (GCP)

Google Cloud VPC

Cloud CDN + Cloud Armor
Reverse Proxy
Edge caching / WAF / DDoS

Apigee API Management
API Gateway
Analytics / Rate limit / OAuth

Cloud Load Balancing
Load Balancer
Global HTTP(S) LB / Anycast

Cloud Run / GKE
App Server
Serverless containers / K8s

Memorystore for Redis
Cache Server
Basic / Standard / HA

Cloud SQL for MySQL
Database MySQL
HA / Read Replica / PITR

☁ Google Cloud Platform

| Livello N-Tier | Servizio GCP | Note |
| --- | --- | --- |
| Reverse Proxy | **Cloud CDN** + **Cloud Armor** + Google Cloud Load Balancing | Edge globale sulla rete Google, WAF managed rules, protezione DDoS adattiva |
| API Gateway | **Apigee API Management** (o Cloud Endpoints per API leggere) | Analytics avanzate, monetizzazione API, developer portal, hybrid/multi-cloud |
| Load Balancer | **Cloud Load Balancing HTTP(S)** (Global) | Anycast nativo, autoscale backend, SSL policy, Cloud Armor integrato |
| App Server | **Cloud Run** / GKE Autopilot / Compute Engine MIG | Cloud Run: serverless container con cold start <1s; GKE Autopilot: K8s gestito |
| Database | **Cloud SQL for MySQL** | HA automatico, Read Replica, PITR, supporto MySQL 8.0+; VPC nativo, backup gestiti, manutenzione automatica |
| Cache | **Memorystore for Redis** | Tier Basic e Standard (con replica); supporto Redis 6/7, TLS, VPC nativo |

📌 Servizi complementari consigliati: **Secret Manager**, **Cloud DNS**, **Cloud Monitoring + Cloud Logging**, **Cloud Trace** (tracing), **Identity Platform** (autenticazione), **Cloud IAP** (accesso sicuro).

### Stima costi mensili (carico medio: ~1000 req/s, 2 zone)

| Servizio | Configurazione | Costo stimato/mese |
| --- | --- | --- |
| Cloud CDN + Cloud Armor | 1 TB, 10M req, policy base | ~$50–80 |
| Apigee X (Pay-as-you-go) | 30M API call/mese | ~$60–120 |
| Cloud Load Balancing | Global HTTPS, 1 TB | ~$25–45 |
| Cloud Run | 4 istanze, 1 vCPU / 2GB | ~$90–140 |
| Cloud SQL MySQL (db-standard-2) | HA, 100 GB SSD | ~$110–145 |
| Memorystore Redis (Standard 5 GB) | 2 nodi replica | ~$60–90 |
| Networking, monitoring, misc | – | ~$35–55 |

💰 Totale stimato: $430 – $675 / mese

#### ✅ Punti di Forza

* Rete privata globale Google: latenza e affidabilità eccellenti
* Cloud Run: il serverless container più maturo del mercato
* Cloud SQL for MySQL: pienamente gestito, compatibile MySQL 8.0, integrazione nativa con VPC e IAM
* GKE è il Kubernetes gestito di riferimento (Google ha inventato K8s)
* Cloud Load Balancing è globale e anycast per default
* Prezzi generalmente più bassi di AWS su compute e storage

#### ⚠️ Punti di Debolezza

* Apigee molto costoso rispetto ad alternative (richiede progetto separato)
* Ecosistema enterprise meno maturo rispetto ad AWS/Azure
* Supporto enterprise percepito come meno reattivo
* Memorystore non supporta tutti i moduli Redis enterprise
* Meno servizi managed "niche" rispetto all'ecosistema AWS

## 5. Confronto riepilogativo

### Tabella comparativa dei servizi per livello

| Livello N-Tier | AWS | Microsoft Azure | Google Cloud |
| --- | --- | --- | --- |
| **Reverse Proxy / CDN** | CloudFront + WAF + Shield | Azure Front Door + WAF | Cloud CDN + Cloud Armor |
| **API Gateway** | Amazon API Gateway | Azure API Management | Apigee / Cloud Endpoints |
| **Load Balancer** | Application Load Balancer | Application Gateway v2 | Cloud Load Balancing (Global) |
| **App Server (container)** | ECS Fargate / EKS | Container Apps / AKS | Cloud Run / GKE Autopilot |
| **Database relazionale** | RDS for Oracle (SE2 / EE) | Azure SQL Database (SQL Server) | Cloud SQL for MySQL |
| **Cache Redis** | ElastiCache for Redis | Azure Cache for Redis | Memorystore for Redis |
| **Secrets / Config** | Secrets Manager / SSM | Azure Key Vault | Secret Manager |
| **Monitoring / APM** | CloudWatch + X-Ray | Azure Monitor + App Insights | Cloud Monitoring + Trace |

### Stima costi mensili – confronto (carico medio)

| Voce di costo | AWS | Azure | GCP |
| --- | --- | --- | --- |
| CDN / Reverse Proxy | $60–90 | $65–100 | $50–80 |
| API Gateway | $30–50 | $50–700\* | $60–120 |
| Load Balancer | $30–45 | $45–65 | $25–45 |
| App Server (containers) | $120–180 | $110–160 | $90–140 |
| Database (Oracle / SQL Server / MySQL) | $350–500 (Lic. Incl.) | $125–160 | $110–145 |
| Cache Redis | $70–100 | $75–100 | $60–90 |
| Networking / Misc | $40–60 | $40–60 | $35–55 |
| TOTALE STIMATO / mese | $650 – $975 (Lic. Incl.) $450–$635 con BYOL | $510 – $1.345\* | $430 – $675 |

\* Azure APIM: Developer tier ~$50/mese (solo sviluppo/test, SLA assente); Standard tier ~$700/mese (produzione). Per produzione la voce APIM sale notevolmente.
\*\* AWS Oracle: in modalità BYOL il costo RDS scende a ~$150–200/mese, ma richiede licenze Oracle preesistenti e contratto di supporto Oracle separato.

### Grafico radar comparativo

Radar: AWS vs Azure vs GCP
(scala 1–10 per dimensione)

Ecosistema
servizi disponibili
Sicurezza
WAF/Compliance
Supporto
Enterprise SLA
Costo
prezzo/performance
Performance
latenza/velocità
Facilità
setup/gestione

AWS

Azure

GCP

| Dimensione | AWS | Azure | GCP |
| --- | --- | --- | --- |
| Ecosistema servizi | 9.5/10 | 8.5/10 | 8/10 |
| Sicurezza & Compliance | 9/10 | 9.5/10 | 8.5/10 |
| Supporto Enterprise | 8.5/10 | 9/10 | 7/10 |
| Costo/Performance | 6.5/10 | 6/10 | 8.5/10 |
| Performance Network | 8.5/10 | 8/10 | 9/10 |
| Facilità di setup | 7/10 | 7.5/10 | 8/10 |

Valutazione soggettiva basata su best practice di settore e feedback della community al 2025.

## 6. Raccomandazioni d'uso

✨ Scegli AWS se...

* Il team ha già esperienza AWS o vuole la piattaforma più matura
* Il progetto richiede Oracle come database (PL/SQL, stored procedure, Oracle-specific features)
* Hai già licenze Oracle e vuoi sfruttarle in cloud con la modalità BYOL
* Vuoi il CDN edge più capillare al mondo (CloudFront)
* Il progetto richiede molti servizi managed "niche" (ML, IoT, analytics)
* Prevedi uso intensivo di Lambda (serverless event-driven)

✨ Scegli Azure se...

* L'azienda usa già Microsoft 365, Teams, Active Directory
* Hai licenze SQL Server esistenti da sfruttare con Azure Hybrid Benefit (risparmio fino al 40%)
* Hai bisogno del developer portal APIM più completo del mercato
* Lavori in contesti enterprise con requisiti di compliance europei
* Il team di sviluppo è .NET / C# oriented

✨ Scegli GCP se...

* Vuoi il miglior serverless container (Cloud Run) con cold start minimo
* Hai carichi con database MySQL e vuoi un servizio completamente gestito su infrastruttura Google
* Vuoi Kubernetes gestito al massimo livello (GKE Autopilot)
* Vuoi ridurre i costi mantenendo alte performance di rete
* Il progetto include AI/ML (integrazione nativa con Vertex AI)

Documento elaborato con dati di riferimento aggiornati al 2025 · Architettura N-Tier per servizi online · AWS, Azure e GCP sono marchi registrati dei rispettivi proprietari
