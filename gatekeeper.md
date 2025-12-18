# Gatekeeper in Red Hat Advanced Cluster Management (RHACM)

Questo documento spiega **perché Gatekeeper viene introdotto nel corso DO432**, qual è il suo ruolo **nel contesto di RHACM**, e **in cosa si differenzia dagli altri policy engine** già visti con le Policy native di ACM.

---

## 1️⃣ Perché serve Gatekeeper in ACM

Le **policy native di RHACM** (ConfigurationPolicy) sono ottime per:

* garantire la **presenza o assenza di risorse**
* imporre uno **stato desiderato** (desired state)
* correggere configurazioni (`inform` / `enforce`)

👉 Ma **non sono progettate per validare ogni richiesta API in tempo reale**.

Qui entra in gioco **Gatekeeper**.

---

## 2️⃣ Open Policy Agent (OPA): il motore

**OPA** è:

* un **policy engine generico**
* indipendente da Kubernetes
* basato sul linguaggio dichiarativo **Rego**

### Cosa fa OPA

* prende una richiesta (input)
* valuta regole (Rego)
* restituisce una decisione (allow / deny / warn)

OPA **non applica policy da solo**: deve essere integrato in un sistema.

---

## 3️⃣ Gatekeeper: OPA per Kubernetes

**OPA Gatekeeper** è:

* una **specializzazione di OPA per Kubernetes**
* implementata come **admission controller webhook**

### Posizione nel flusso Kubernetes

```
Client → API Server
        → AuthN / AuthZ
        → Admission Controllers
            → Gatekeeper
        → Oggetto creato
```

👉 Gatekeeper **intercetta le richieste prima che l’oggetto venga creato**.

---

## 4️⃣ Ruolo di Gatekeeper in RHACM

RHACM **non sostituisce Gatekeeper**, ma lo **orchestra su più cluster**.

### Cosa fa RHACM

* installa Gatekeeper su N cluster
* gestisce ConstraintTemplate e Constraint
* distribuisce le policy in modo GitOps
* centralizza la governance

👉 **ACM governa Gatekeeper**, non il contrario.

---

## 5️⃣ Integrazione Gatekeeper ↔ RHACM (2 step)

### Step 1 – Installazione Gatekeeper

* RHACM usa una **Policy di governance**
* installa il Gatekeeper Operator
* crea la custom resource `Gatekeeper`

📌 Questa parte usa le **ConfigurationPolicy ACM**.

---

### Step 2 – Definizione delle regole

* ConstraintTemplate
* Constraint

Questi oggetti:

* contengono **Rego**
* definiscono **regole di ammissione e audit**

📌 Qui entra in gioco **Gatekeeper come policy engine**.

---

## 6️⃣ ConstraintTemplate vs Constraint

### ConstraintTemplate

* definisce:

  * schema
  * parametri
  * codice Rego

👉 È la **classe** della policy.

---

### Constraint

* istanzia il template
* passa valori concreti
* definisce il match

👉 È l’**istanza** della policy.

---

## 7️⃣ Esempio del corso (spiegato)

### Cosa controlla la policy

> “Ogni Namespace `test-labels-ns` deve avere la label `gatekeeper`.”

---

### Rego (concetto)

* legge le label presenti
* confronta con quelle richieste
* se mancano → violazione

---

### enforcementAction: dryrun

* **dryrun** → segnala ma non blocca
* **deny** → blocca la richiesta

📌 Utile per rollout progressivo.

---

## 8️⃣ Differenza tra Gatekeeper e Policy native ACM

| Aspetto     | ACM ConfigurationPolicy | Gatekeeper            |
| ----------- | ----------------------- | --------------------- |
| Momento     | Dopo il deploy          | Prima del deploy      |
| Tipo        | Stato dichiarativo      | Validazione API       |
| Linguaggio  | YAML                    | Rego                  |
| Ambito      | Oggetti esistenti       | Richieste in ingresso |
| Enforcement | Idempotente             | Blocco immediato      |

---

## 9️⃣ Gatekeeper vs altri Policy Engine in ACM

### Confronto concettuale

| Engine              | Scopo                | Tipo          |
| ------------------- | -------------------- | ------------- |
| ConfigurationPolicy | Governance           | Desired state |
| Gatekeeper          | Admission control    | Preventivo    |
| Compliance Operator | Compliance normativa | Audit         |
| ACS (DO430)         | Runtime security     | Detection     |

---

## 🔟 Perché ACM include Gatekeeper

RHACM **non vuole reinventare un admission controller**.

👉 Usa Gatekeeper perché:

* è standard CNCF
* è estendibile
* usa Rego
* è già diffuso

ACM aggiunge:

* **multi-cluster governance**
* **placement**
* **RBAC e tenancy**

---

## 1️⃣1️⃣ Frase chiave per la docenza

> **Le policy ACM garantiscono *come deve essere* il cluster.**
> **Gatekeeper decide *cosa può entrare* nel cluster.**

Oppure:

> **ACM corregge. Gatekeeper previene.**

---

## 1️⃣2️⃣ Sintesi finale

* Gatekeeper è un **admission controller**
* Usa **OPA + Rego**
* ACM lo distribuisce e governa
* Insieme coprono:

  * prevenzione (Gatekeeper)
  * correzione (ACM)

---

*Documento pensato per studio, ripasso ed esposizione orale nel corso DO432.*
