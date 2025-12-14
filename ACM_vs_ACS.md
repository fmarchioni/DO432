# Confronto tra Policy RHACM (ACM) e Policy di Sicurezza ACS (DO430)

Questo documento confronta **le Policy di Red Hat Advanced Cluster Management (RHACM / ACM)** con **le Policy di Red Hat Advanced Cluster Security (ACS)**, come studiate nel corso **DO430**.

L’obiettivo è chiarire:

* **scopo**
* **modello concettuale**
* **quando usare l’una o l’altra**

---

## Visione di insieme

| Aspetto     | RHACM – Governance Policy       | ACS – Security Policy                      |
| ----------- | ------------------------------- | ------------------------------------------ |
| Dominio     | **Governance / Configurazione** | **Security / Threat detection**            |
| Obiettivo   | Stato desiderato dei cluster    | Riduzione del rischio e protezione runtime |
| Filosofia   | *Desired State*                 | *Risk-based security*                      |
| Ambito      | Multi-cluster                   | Cluster + workload                         |
| Enforcement | Opzionale                       | Spesso automatico                          |

---

## 1️⃣ Policy RHACM (Governance)

### Scopo

Le policy ACM servono a:

* garantire **coerenza** tra cluster
* applicare **standard operativi**
* imporre configurazioni Kubernetes

Sono **configurationali**, non reattive.

---

### Come funzionano

* Basate su **ConfigurationPolicy**
* Valutano se una risorsa **esiste / non esiste / è conforme**
* Possono:

  * segnalare (`inform`)
  * correggere (`enforce`)

---

### Esempi tipici

* Namespace obbligatori
* Operator installati
* SecurityContext definiti
* NetworkPolicy presenti
* Compliance (CIS, NIST)

---

### Modello mentale

> “**Come deve essere configurato il cluster?**”

---

## 2️⃣ Policy ACS (Security – DO430)

### Scopo

Le policy ACS servono a:

* **identificare rischi di sicurezza**
* prevenire deploy pericolosi
* rilevare comportamenti anomali

Sono **reattive e preventive**.

---

### Come funzionano

* Basate su **regole di sicurezza**
* Valutano:

  * immagini
  * deploy
  * runtime
* Agiscono in più fasi:

  * Build
  * Deploy
  * Runtime

---

### Esempi tipici

* Container privilegiati
* Immagini con CVE critiche
* Exec in container
* Network flow sospetti
* Accesso a secret

---

### Modello mentale

> “**Questo workload è sicuro? Sta facendo qualcosa di pericoloso?**”

---

## 3️⃣ Differenze chiave (concettuali)

### Statico vs Dinamico

| RHACM              | ACS                     |
| ------------------ | ----------------------- |
| Stato dichiarativo | Analisi comportamentale |
| Configurazione     | Sicurezza runtime       |
| Cluster-wide       | Workload-centric        |

---

### Enforcement

| RHACM                | ACS                 |
| -------------------- | ------------------- |
| `inform` / `enforce` | Block / Fail / Kill |
| Idempotente          | Evento-driven       |

---

### Livello di intervento

| RHACM              | ACS                         |
| ------------------ | --------------------------- |
| Kubernetes objects | Processi, syscalls, network |
| API Server         | Kernel / runtime            |

---

## 4️⃣ Relazione tra ACM e ACS

Le due tecnologie **non si sovrappongono**, ma si **completano**.

### Insieme coprono:

* **Prevenzione configurazionale** (ACM)
* **Prevenzione e rilevamento security** (ACS)

---

### Esempio reale

| Fase    | Strumento | Azione                        |
| ------- | --------- | ----------------------------- |
| Design  | ACM       | Impone NetworkPolicy          |
| Deploy  | ACS       | Blocca container privilegiato |
| Runtime | ACS       | Rileva exec sospetto          |
| Drift   | ACM       | Ripristina config corretta    |

---

## 5️⃣ Confronto diretto DO432 vs DO430

| Tema          | DO432 – ACM | DO430 – ACS |
| ------------- | ----------- | ----------- |
| Tipo policy   | Governance  | Security    |
| Focus         | Coerenza    | Rischio     |
| Multi-cluster | Sì          | Limitato    |
| Runtime       | No          | Sì          |
| GitOps        | Centrale    | Secondario  |

---

## 6️⃣ Frase chiave da ricordare (docenza)

> **ACM governa *come* i cluster devono essere configurati.**
> **ACS protegge *da cosa* i workload devono essere difesi.**

Oppure:

> **ACM impone ordine. ACS difende dal caos.**

---

## 7️⃣ Quando usare cosa

* **Solo ACM** → standardizzazione, compliance, multi-cluster
* **Solo ACS** → sicurezza runtime
* **ACM + ACS** → piattaforma enterprise completa

---

## 8️⃣ Errore comune da evitare

❌ Pensare che le policy ACS sostituiscano ACM
❌ Pensare che le policy ACM facciano sicurezza runtime

✔️ Sono strumenti **complementari**, non alternativi

---

## Sintesi finale

| ACM             | ACS              |
| --------------- | ---------------- |
| Governance      | Security         |
| Desired State   | Risk Detection   |
| GitOps-first    | Event-driven     |
| Cluster-centric | Workload-centric |

---

*Documento pensato per studio, ripasso e docenza (DO432 / DO430).*
