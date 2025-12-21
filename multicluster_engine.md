# RHACM – MultiCluster Engine (MCE)

> **Obiettivo del documento**
> Spiegare in modo chiaro e approfondito cos’è il **MultiCluster Engine (MCE)**, perché esiste, quali componenti include e come si relaziona con **Red Hat Advanced Cluster Management (RHACM)**.
 

---

## 1. Cos’è il MultiCluster Engine (MCE)

Il **MultiCluster Engine (MCE)** è un **Operator** che si può utilizzare come **motore multicluster comune** su cui si basa RHACM.

In altre parole:

> **MCE fornisce le funzionalità fondamentali di gestione multicluster, senza includere la governance avanzata di RHACM.**

È una piattaforma condivisa che implementa le primitive multicluster di base.

---

## 2. Perché Red Hat ha introdotto MCE

Storicamente, tutte le funzionalità multicluster erano incluse direttamente in RHACM.

Con il tempo, Red Hat ha identificato che:

* molte funzionalità erano **riutilizzabili**
* alcuni use case richiedevano **gestione multicluster senza governance completa**
* altri prodotti Red Hat avevano bisogno delle stesse primitive

👉 Da qui nasce **MCE**.

---

## 3. Relazione tra RHACM e MCE

### Relazione concettuale

| Livello                       | Componente      |
| ----------------------------- | --------------- |
| Governance avanzata           | **RHACM**       |
| Gestione multicluster di base | **MCE**         |
| Kubernetes                    | OpenShift / K8s |

> **RHACM = MCE + Governance + UI avanzata + Policy framework**

---

### Cosa succede in pratica

Quando installi RHACM:

* **MCE viene installato automaticamente**
* RHACM usa MCE come fondazione

Puoi anche installare **solo MCE**, senza RHACM.

---

## 4. Cosa fornisce MCE (responsabilità)

MCE fornisce le **capacità multicluster fondamentali**, tra cui:

* import e gestione dei cluster
* lifecycle dei cluster (create / import / detach / destroy)
* gestione delle credenziali
* concetti di base come:

  * `ManagedCluster`
  * `ManagedClusterSet`
  * `Placement`

> Tutto ciò che riguarda il *“sapere che esistono più cluster”* è responsabilità di MCE.

---

## 5. Cosa NON fornisce MCE

MCE **non include**:

* policy framework avanzato
* governance
* compliance
* observability multicluster
* dashboard di governance
* integrazione diretta GitOps applicativa

Queste funzionalità sono **esclusive di RHACM**.

---

## 6. Componenti principali del MultiCluster Engine

### 6.1 Custom Resource fondamentali

MCE introduce CRD centrali:

* `ManagedCluster`
* `ManagedClusterSet`
* `ManagedClusterSetBinding`
* `Placement`

Queste risorse:

* definiscono *chi sono* i cluster
* come vengono raggruppati
* come vengono selezionati

---

### 6.2 Add-on framework

MCE fornisce un **framework di add-on**:

* componenti che vengono installati automaticamente nei managed cluster
* usati per comunicazione e controllo

Esempi:

* cluster-proxy
* registration agent
* work manager

---

## 7. MCE e Cluster Lifecycle Management

MCE è responsabile di:

* import dei cluster (manuale o automatico)
* gestione dello stato (Ready / Unavailable)
* rimozione sicura dei cluster

RHACM **riusa** queste funzionalità e le espone tramite:

* UI più ricca
* workflow guidati

---

## 8. MCE senza RHACM: quando ha senso

Installare **solo MCE** ha senso quando:

* vuoi solo:

  * importare cluster
  * raggrupparli
  * selezionarli
* non hai bisogno di:

  * policy
  * compliance
  * observability

Tipico scenario:

* prodotto Red Hat che richiede gestione multicluster
* piattaforma custom che riusa le primitive MCE

---

## 9. MCE e sicurezza

MCE implementa:

* trust tra hub e managed cluster
* canali di comunicazione sicuri
* gestione dei certificati

Ma:

> **non decide cosa è consentito a livello di governance**

Quella è responsabilità di RHACM.

---

## 10. Differenza chiave da ricordare

| Domanda                              | Risposta |
| ------------------------------------ | -------- |
| Chi conosce i cluster?               | MCE      |
| Chi li governa?                      | RHACM    |
| Chi applica policy?                  | RHACM    |
| Chi fornisce primitive multicluster? | MCE      |

---

## 11. Frase conclusiva (chiave)

> **Il MultiCluster Engine è il motore. RHACM è il cervello.**

Oppure, in modo più tecnico:

> **MCE fornisce le primitive multicluster, RHACM implementa la governance sopra di esse.**

---

## 12. Collegamento con il corso DO432

Nel corso **DO432**:

* MCE è la base implicita di tutti gli esercizi
* RHACM aggiunge:

  * policy
  * observability
  * compliance
  * GitOps

Capire MCE rende RHACM **molto più semplice da assimilare**.
