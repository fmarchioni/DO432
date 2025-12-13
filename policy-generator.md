# RHACM PolicyGenerator – Guida dettagliata

Questa guida spiega in modo **chiaro e progressivo** come funziona il **PolicyGenerator** in Red Hat Advanced Cluster Management (RHACM), partendo dai concetti fino al flusso operativo completo.

---

## 1. Cos’è il PolicyGenerator

Il **PolicyGenerator** non è una risorsa Kubernetes applicata al cluster, ma **uno strumento di generazione**.

Il suo scopo è:

* Partire da **manifest Kubernetes “normali”** (Namespace, ConfigMap, Role, ecc.)
* Generare automaticamente:

  * una **Policy**
  * un **Placement**
  * un **PlacementBinding**

In pratica:

> **PolicyGenerator = compilatore di policy RHACM**

Tu scrivi *cosa deve esistere*, lui produce *come RHACM lo governa sui cluster*.

---

## 2. Due modalità di utilizzo

RHACM supporta due modi per usare il PolicyGenerator.

### 2.1 PolicyGenerator + Kustomize

* Usa **Kustomize** come motore
* Il PolicyGenerator è un **plugin esterno**
* Serve:

  * `kustomization.yaml`
  * `policy-generator-config.yaml`

Prerequisito:

```
~/.config/kustomize/plugin/
└── policy.open-cluster-management.io/v1/policygenerator
    └── PolicyGenerator
```

Esecuzione:

```
kubectl kustomize --enable-alpha-plugins <dir>
```

📌 **Nota importante**: il flag `--enable-alpha-plugins` è obbligatorio.

---

### 2.2 PolicyGenerator CLI (consigliato per i lab)

* Tool standalone fornito con RHACM
* Non usa Kustomize
* Più semplice e diretto

Flusso:

```
PolicyGenerator policy-generator-config.yaml
```

Produce direttamente YAML pronti da applicare con `oc apply`.

📌 **È questa la modalità usata nel corso/lab**.

---

## 3. Struttura del file PolicyGenerator

Il file `PolicyGenerator` ha tre sezioni chiave:

### 3.1 policyDefaults (obbligatoria)

Definisce i **valori di default** per tutte le policy generate.

Esempio:

```yaml
policyDefaults:
  namespace: policies
  severity: low
  remediationAction: inform
  complianceType: musthave
```

Significato:

* **namespace**: dove verranno create le Policy (non sovrascrivibile)
* **severity**: livello di gravità
* **remediationAction**:

  * `inform` → solo segnalazione
  * `enforce` → applica le modifiche
* **complianceType**:

  * `musthave` → l’oggetto deve esistere
  * `mustonlyhave` → deve esistere *esattamente così*
  * `mustnothave` → non deve esistere

---

### 3.2 policies (obbligatoria)

Definisce **le policy da creare**, una per una.

Esempio:

```yaml
policies:
- name: namespace-existence
  manifests:
  - path: /manifest-files/manifest.yaml
```

Qui:

* `name`: nome della Policy RHACM
* `manifests`: file o directory di manifest Kubernetes da includere

📌 Ogni file diventa un **object-template** dentro una `ConfigurationPolicy`.

---

### 3.3 policySets (opzionale)

Serve per:

* Raggruppare policy
* Applicare Placement comuni
* Usare PolicySet (governance avanzata)

Non usata nel lab, ma molto comune in produzione.

---

## 4. I manifest “di input”

I file referenziati nella sezione `manifests` sono **manifest Kubernetes standard**.

Esempio:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: testing-policies
```

Il PolicyGenerator:

* **non li applica direttamente**
* li **incapsula dentro una Policy**

---

## 5. Struttura delle directory

### 5.1 Con Kustomize

```
base/
├── kustomization.yaml
├── policy-generator-config.yaml
└── manifest-files/
    └── manifest.yaml
```

📌 I manifest devono stare **in una directory separata**, altrimenti il PolicyGenerator finirebbe per includere se stesso.

---

### 5.2 Con PolicyGenerator CLI

```
policy-gen/
├── policy-generator-config.yaml
└── manifest-files/
    └── manifest.yaml
```

---

## 6. Output generato

Eseguendo il tool, vengono generati **tre oggetti**.

### 6.1 Policy

* Contiene una `ConfigurationPolicy`
* Dentro ci sono gli `object-templates`
* È la **regola di compliance**

---

### 6.2 Placement

* Decide **su quali cluster** applicare la policy
* Di default:

  * tutti i cluster disponibili
  * cluster unreachable/unavailable tollerati

---

### 6.3 PlacementBinding

* Collega **Policy ↔ Placement**
* Senza di lui, la policy non viene applicata

---

## 7. Naming automatico

Il PolicyGenerator aggiunge automaticamente i suffissi:

* `placement-<policy-name>`
* `binding-<policy-name>`

Questo:

* evita collisioni
* standardizza la governance

---

## 8. Applicazione al cluster

Una volta generati i manifest:

```
oc apply -f policy-definition.yaml
```

Risultato:

* Policy creata
* Placement creato
* PlacementBinding creato

📌 Da questo momento RHACM **valuta e applica la compliance**.

---

## 9. Aggiornare una policy

Il PolicyGenerator **non modifica direttamente** le risorse già presenti.

Flusso corretto:

1. Modifichi `policy-generator-config.yaml`
2. Rigeneri i manifest
3. Riapplichi con `oc apply`

Esempio patch:

```yaml
policyDefaults:
  severity: medium
```

Risultato:

* Policy aggiornata
* Compliance ricalcolata

---

## 10. Avvertenza importante

⚠️ **Kustomize + PolicyGenerator insieme possono causare problemi in produzione**.

Motivo:

* due strumenti di generazione
* difficile tracciabilità del risultato finale

📌 In produzione:

* GitOps (ArgoCD)
* manifest generati e versionati

---

## 11. Riassunto concettuale

* PolicyGenerator = strumento di generazione
* Input = manifest Kubernetes normali
* Output = Policy + Placement + Binding
* Governance dichiarativa e ripetibile

---

# RHACM PolicyGenerator – Ripasso veloce

## Cos’è

**PolicyGenerator** è uno strumento che:

* prende manifest Kubernetes standard
* genera automaticamente:

  * **Policy**
  * **Placement**
  * **PlacementBinding**

---

## Modalità d’uso

### 1️⃣ Con Kustomize

* PolicyGenerator come plugin
* Serve `kustomization.yaml`
* Comando:

```
kubectl kustomize --enable-alpha-plugins
```

---

### 2️⃣ CLI PolicyGenerator (lab)

* Tool standalone RHACM
* Nessun kustomization

```
PolicyGenerator policy-generator.yaml
```

---

## Struttura PolicyGenerator

### policyDefaults (obbligatoria)

* namespace (non sovrascrivibile)
* severity
* remediationAction (`inform` | `enforce`)
* complianceType (`musthave`, `mustonlyhave`, `mustnothave`)

---

### policies (obbligatoria)

* nome policy
* file o directory di manifest

Ogni manifest → object-template

---

### policySets (opzionale)

* raggruppa policy
* governance avanzata

---

## Output generato

Sempre:

* **Policy** → regola di compliance
* **Placement** → target cluster
* **PlacementBinding** → collega policy e placement

Naming automatico:

* `placement-<policy>`
* `binding-<policy>`

---

## Flusso tipico

1. Scrivi manifest Kubernetes
2. Scrivi PolicyGenerator YAML
3. Generi i manifest
4. `oc apply -f`

---

## Update policy

1. Modifica PolicyGenerator YAML
2. Rigenera
3. Riapplica

---

## Nota importante

⚠️ Evita Kustomize + PolicyGenerator insieme in produzione.

---

## Frase chiave da ricordare

> **PolicyGenerator non applica risorse, genera governance**

