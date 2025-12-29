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



# 1️⃣ Cos’è Gatekeeper (prima ancora di RHACM)

**Gatekeeper** è:

* un **policy engine** per Kubernetes
* basato su **OPA (Open Policy Agent)**
* che lavora **in fase di ammissione** (Admission Controller)

👉 Gatekeeper decide:

> “Questo oggetto Kubernetes può essere creato / modificato oppure no?”

📌 Differenza chiave rispetto al Compliance Operator:

| Compliance Operator | Gatekeeper                  |
| ------------------- | --------------------------- |
| Audit / scan        | Enforcement                 |
| Post-evento         | Pre-evento                  |
| Periodico           | Sincrono                    |
| Stato del sistema   | Validazione delle richieste |

---

# 2️⃣ Gatekeeper in RHACM: il ruolo del governance layer

RHACM **non sostituisce Gatekeeper**, ma:

* lo **installa**
* lo **configura**
* lo **governa** su **più cluster**

Schema mentale:

```
RHACM Policy
   ↓
ConstraintTemplate + Constraint
   ↓
Gatekeeper (managed cluster)
   ↓
Admission Webhook Kubernetes
```

---

# 3️⃣ I due oggetti fondamentali: ConstraintTemplate e Constraint

Questo è **IL punto cruciale**.

## ConstraintTemplate → il “contratto”

## Constraint → l’“istanza concreta”

Un’analogia che funziona benissimo in aula:

> **ConstraintTemplate = classe**
>
> **Constraint = oggetto**

---

## 3.1 ConstraintTemplate: cosa definisce

Il `ConstraintTemplate`:

* definisce **la regola**
* definisce **lo schema dei parametri**
* contiene il **codice Rego**

Nel tuo esempio:

```yaml
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
```

👉 Stai creando **un nuovo tipo di constraint** chiamato:

```
K8sRequiredLabels
```

📌 Gatekeeper estende l’API Kubernetes:

* dopo questo template
* puoi creare risorse `kind: K8sRequiredLabels`

---

## 3.2 La sezione CRD: schema dei parametri

```yaml
crd:
  spec:
    names:
      kind: K8sRequiredLabels
    validation:
      openAPIV3Schema:
        properties:
          labels:
            type: array
            items: string
```

Questa parte dice:

> “Ogni constraint di questo tipo **DEVE** avere un campo `labels`, che è un array di stringhe.”

📌 Importantissimo:

* Gatekeeper **valida il constraint**
* prima ancora di eseguire Rego

---

# 4️⃣ Rego: il linguaggio di Gatekeeper

Rego **non è YAML**
Rego **non è JSON**
Rego **non è Kubernetes**

👉 Rego è:

* un linguaggio **dichiarativo**
* basato su **logica**
* orientato a **valutare condizioni**

---

## 4.1 Struttura base del Rego

Dal tuo esempio:

```rego
package k8srequiredlabels
```

📌 Il package **deve** corrispondere al nome del ConstraintTemplate.

---

## 4.2 Il concetto di violation

```rego
violation[{"msg": msg, "details": {"missing_labels": missing}}] {
```

Traduzione mentale:

> “Esiste una violazione se la condizione seguente è vera”

Gatekeeper:

* NON chiede “true / false”
* chiede:

> “Esistono violazioni?”

---

## 4.3 Accesso all’oggetto Kubernetes

```rego
input.review.object.metadata.labels
```

Questa è una cosa da spiegare bene agli studenti.

`input` contiene:

* la **richiesta di ammissione**
* l’oggetto Kubernetes **così come arriva all’API server**

Quindi:

* Namespace
* Pod
* Deployment
* ecc.

---

## 4.4 Logica della regola

```rego
provided := {label | input.review.object.metadata.labels[label]}
required := {label | label := input.parameters.labels[_]}
missing := required - provided
count(missing) > 0
```

Spiegazione **linea per linea**:

1. `provided`
   → insieme delle label presenti sull’oggetto

2. `required`
   → insieme delle label richieste (parametri del constraint)

3. `missing`
   → differenza tra richieste e presenti

4. `count(missing) > 0`
   → se manca almeno una label → violazione

📌 Questo è **set-based logic**, non imperativa.

---

# 5️⃣ Il Constraint: applicare la regola

Ora arriva il `Constraint`.

```yaml
kind: K8sRequiredLabels
metadata:
  name: ns-must-have-gk
```

👉 Qui stai dicendo:

> “Applica questa regola concreta”

---

## 5.1 enforcementAction

```yaml
enforcementAction: dryrun
```

Valori tipici:

* `deny` → blocca l’oggetto
* `dryrun` → segnala ma non blocca

📌 In aula:

> *Gatekeeper può essere introdotto in modalità osservativa prima di diventare bloccante.*

---

## 5.2 match: dove si applica

```yaml
match:
  kinds:
    - apiGroups: [""]
      kinds: ["Namespace"]
  namespaces:
    - test-labels-ns
```

Questo è **fondamentale**:

* Gatekeeper **non si applica ovunque**
* si applica **solo dove matcha**

📌 Nota didattica:

> Match è l’equivalente di “Placement” per Gatekeeper.

---

## 5.3 parameters: collegamento col template

```yaml
parameters:
  labels: ["gatekeeper"]
```

Questo valore:

* viene passato a Rego come:

```rego
input.parameters.labels
```

👉 **Template = logica**
👉 **Constraint = dati**

---

# 6️⃣ Come RHACM integra Gatekeeper

RHACM:

* NON interpreta Rego
* NON valuta i constraint

Fa solo:

1. Distribuzione
2. Osservazione dello stato
3. Raccolta violazioni

Schema:

```
RHACM Policy
   ↓
ConstraintTemplate + Constraint
   ↓
Gatekeeper (cluster)
   ↓
Admission Controller
   ↓
Violation Event
   ↓
RHACM Dashboard
```

---

# 7️⃣ Gatekeeper vs ConfigurationPolicy (messaggio chiave)

Questa tabella ti salva da molte domande:

| Feature     | ConfigurationPolicy | Gatekeeper  |
| ----------- | ------------------- | ----------- |
| Quando      | Dopo                | Prima       |
| Modalità    | Stato desiderato    | Validazione |
| Linguaggio  | YAML                | Rego        |
| Complessità | Bassa               | Alta        |
| Bloccante   | Sì (enforce)        | Sì (deny)   |

📌 Frase che funziona sempre:

> “ConfigurationPolicy corregge, Gatekeeper impedisce.”

---
Sì — e questa è una **domanda molto intelligente**, perché Gatekeeper/OPA e **Rego** sono potenti ma possono risultare ostici da scrivere “a mano”.
Esistono **strumenti e workflow** pensati per **semplificare la scrittura, la validazione e il debugging delle regole Rego**.

Questa risposta ti dà:

* gli **strumenti ufficiali e non ufficiali**
* come usarli concretamente
* collegamenti a RHACM / Gatekeeper
* consigli pratici per l’aula o scenari enterprise

---

# 🧠 Primer: perché serve un tool per Rego

Rego è un **linguaggio dichiarativo logico**, non imperativo.
Non si programma come in Bash/Python, quindi:

* le **regole si esprimono come logica predicativa**
* non ci sono IDE standard con evidenziazione/auto-completion
* gli errori sintattici o semantici non sono sempre immediati

🛠️ Per questo nascono strumenti che aiutano a:

* **scrivere** regole
* **validare** sintassi e semantica
* **testare** con input di esempio
* **debuggare** con output strutturato

---

# 🛠️ Principali strumenti per Rego / Gatekeeper

## OPA CLI (ufficiale)

📦 *Il tool principale e fondamentale.*

### Cosa fa

* analizza policy Rego
* valida sintassi
* esegue query locali
* consente test automatizzati

### Installazione

```bash
brew install opa        # macOS
sudo yum install opa    # RHEL/CentOS
```
 

---

## VS Code + Estensioni Rego

📌 IDE consigliato per scrivere Rego

### Estensioni utili

* **OPA Rego** (editor support)
* **Syntax highlighting**
* **Diagnostics**
* **Auto-completion su input e data**

👉 Molto utile per chi impara

---

## Playground Online (rapid prototyping)

🔗 **OPA Playground** (browser):
[https://play.openpolicyagent.org/](https://play.openpolicyagent.org/)

 
 

## ✅ Policy completa (OPA Playground – Rego v1)

```rego
package kubernetes.admission

violation contains result if {
  input.review.kind.kind == "Pod"

  container := input.review.object.spec.containers[_]

  endswith(container.image, ":latest")

  not allowed_namespace

  result := {
    "msg": sprintf(
      "Image %q uses the forbidden 'latest' tag",
      [container.image]
    ),
    "details": {
      "container": container.name,
      "namespace": input.review.object.metadata.namespace
    }
  }
}

allowed_namespace if {
  input.review.object.metadata.namespace == "dev"
}

allowed_namespace if {
  input.review.object.metadata.namespace == "test"
}
```
 

INPUT:
```yaml
{
  "review": {
    "kind": {
      "kind": "Pod"
    },
    "object": {
      "metadata": {
        "name": "nginx-pod",
        "namespace": "production"
      },
      "spec": {
        "containers": [
          {
            "name": "nginx",
            "image": "nginx:latest"
          }
        ]
      }
    }
  }
}

```

* Fai click su **Evaluate** e verifica la violazione
* Rimuovi il tag latest. Dovrebbe scomparire la violazione
