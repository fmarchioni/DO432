# Observability in Red Hat Advanced Cluster Management (RHACM)

Questo documento fornisce una **spiegazione chiara e dettagliata** di come funziona l’**observability in RHACM**, con particolare attenzione a:

* architettura dei componenti
* flusso dei dati
* differenze rispetto al monitoring **single-cluster OpenShift**
* cosa cambia a livello **operativo e concettuale** passando a una gestione multicluster

Il contenuto integra il **manuale DO432** con considerazioni architetturali e best practice.

---

## 1️⃣ Cos’è la Multicluster Observability in RHACM

L’**observability di RHACM** è un servizio centralizzato che consente di:

* raccogliere metriche da **più cluster OpenShift**
* correlare stato e utilizzo di cluster e workload
* conservare metriche a lungo termine
* interrogare i dati in modo globale

👉 L’obiettivo non è sostituire il monitoring nativo di OpenShift, ma **estenderlo a livello fleet**.

---

## 2️⃣ Monitoring OpenShift single-cluster (baseline)

### Architettura standard RHOCP

Ogni cluster OpenShift include:

* **Prometheus** (per cluster)
* **Alertmanager** (per cluster)
* **Grafana** (per cluster)

Caratteristiche:

* metriche locali al cluster
* retention limitata
* visibilità solo intra-cluster
* focus su **platform health**

A partire da OCP 4.6:

* possibile monitorare anche **user-defined projects**

---

## 3️⃣ Perché il single-cluster non basta

In ambienti enterprise:

* decine o centinaia di cluster
* ambienti ibridi / edge
* necessità di:

  * capacity planning
  * analisi storica
  * correlazione eventi

👉 Il monitoring single-cluster **non scala concettualmente**.

---

## 4️⃣ Architettura Observability RHACM (big picture)

```
Managed Cluster (N)
└─ OpenShift Prometheus
   └─ Observability Endpoint (addon)
        ↓
        ↓  (metrics)
        ↓
Hub Cluster (RHACM)
└─ Thanos Receive
└─ Thanos Store
└─ Thanos Compactor
└─ Alertmanager
└─ Grafana
└─ Object Storage (S3)
```

👉 I cluster **non condividono Prometheus**: inviano metriche al hub.

---

## 5️⃣ Componenti principali RHACM Observability

### Prometheus (managed cluster)

* continua a raccogliere metriche localmente
* nessuna modifica distruttiva
* espone metriche all’addon observability

---

### Observability Endpoint Operator

* deployato automaticamente su ogni managed cluster
* legge metriche da Prometheus locale
* invia i dati al hub cluster

👉 È il **bridge tra managed cluster e hub**.

---

### Multicluster Observability Operator (hub)

* deployato dal `multiclusterhub-operator`
* orchestra tutta l’infrastruttura observability
* crea componenti Thanos, Grafana, Alertmanager

---

### Thanos (cuore del multicluster)

RHACM usa **Thanos** per:

* aggregare metriche da più Prometheus
* effettuare query globali
* archiviare dati storici

Componenti chiave:

* **Receive** → riceve metriche dai cluster
* **Store** → espone metriche storiche
* **Compactor** → ottimizza e applica retention

---

### Object Storage (S3)

Elemento **obbligatorio** in RHACM:

* storage persistente
* retention a lungo termine
* indipendente dal lifecycle dei cluster

Esempi:

* OpenShift Data Foundation (NooBaa)
* S3 compatibile esterno

---

### Grafana (hub)

* dashboard **fleet-wide**
* viste aggregate per cluster, namespace, workload
* dashboard preconfigurate RHACM

📌 Non sostituisce la Grafana di ogni cluster.

---

### Alertmanager (hub)

* riceve alert aggregati
* deduplica alert simili da cluster diversi
* routing centralizzato (email, Slack, PagerDuty)

---

## 6️⃣ Differenze chiave: Single-cluster vs RHACM

| Aspetto    | OpenShift Single Cluster | RHACM Observability |
| ---------- | ------------------------ | ------------------- |
| Scope      | 1 cluster                | Fleet di cluster    |
| Prometheus | Locale                   | Distribuito         |
| Storage    | Locale                   | S3 centralizzato    |
| Retention  | Limitata                 | Lungo termine       |
| Query      | Per cluster              | Globale             |
| Alerting   | Per cluster              | Centralizzato       |
| Use case   | Ops locali               | Ops & governance    |

---

## 7️⃣ Cosa cambia concettualmente

### Da cluster-centric a fleet-centric

* prima: “il cluster è il mondo”
* dopo: “il cluster è un nodo della fleet”

---

### Da troubleshooting a analisi

* single cluster → incident response
* RHACM → trend, capacity, postmortem

---

## 8️⃣ Abilitazione Observability in RHACM (logica)

### Passaggi chiave

1. Namespace dedicato
2. Pull secret
3. Object storage
4. Secret Thanos
5. `MultiClusterObservability` CR

👉 L’operatore fa il resto.

---

## 9️⃣ Il Custom Resource MultiClusterObservability

È il **punto di controllo** dell’intero stack.

Permette di:

* abilitare metriche
* configurare downsampling
* definire storage
* scegliere nodi (infra)

👉 È qui che si governa l’osservabilità multicluster.

---

## 🔟 Best practice architetturali

* usare nodi **infra** per lo stack
* dimensionare correttamente lo storage
* usare downsampling per retention lunga
* separare monitoring locale e fleet

---

## 1️⃣1️⃣ Relazione con governance e policy

Observability RHACM:

* **non applica policy**
* **fornisce visibilità**
* supporta decisioni di governance

👉 Governance senza observability è cieca.

---

## 1️⃣2️⃣ Frase chiave per la docenza

> **OpenShift monitora i cluster. RHACM osserva la fleet.**

Oppure:

> **La differenza non è tecnica, è di scala.**

---

## 1️⃣3️⃣ Sintesi finale

* RHACM Observability è **Prometheus + Thanos + Grafana + Alertmanager**
* il hub cluster è il centro di raccolta
* i managed cluster restano autonomi
* l’object storage è il fattore abilitante

---

*Documento pensato per studio approfondito e docenza DO432.*
