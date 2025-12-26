
# 📌 RHACM + ZTP – Checklist completa per il corso Telco

## 🧠 Mental model (prima di tutto)

> **ACM non è il protagonista.
> ZTP + GitOps sono il protagonista.
> ACM è il motore di controllo.**

Se questo passa, il corso funziona.

---

## 1️⃣ Componenti **fondamentali** per ZTP (non negoziabili)

### 🔹 MultiClusterHub (CR principale)

```yaml
apiVersion: operator.open-cluster-management.io/v1
kind: MultiClusterHub
```

**Da conoscere bene:**

* `disableUpdateClusterImageSets` ✅ (fondamentale in telco)
* `disableHubSelfManagement`
* `availabilityConfig: High`
* lifecycle del MCH

👉 Domanda tipica:

> *“Perché in ZTP il MCH deve essere il più statico possibile?”*

---

### 🔹 Hive (cluster provisioning)

CR **chiave per ZTP**:

* `ClusterDeployment`
* `AgentClusterInstall`
* `InfraEnv`
* `BareMetalHost`
* `NMStateConfig`

👉 Anche se ZTP “nasconde” Hive, **le domande arrivano sempre qui**.

---

### 🔹 SiteConfig (legacy ZTP)

> **Telecom è ancora in modalità legacy** → devi conoscerla.

CR / concetti:

* `SiteConfig`
* plugin ZTP che genera:

  * ClusterDeployment
  * BMH
  * NMState
* limite: **scarsa flessibilità day-2**

👉 Punto perfetto per spiegare:

* *perché è macchinoso sostituire un nodo*
* *perché Red Hat spinge verso GitOps + Policy Generator*

---

## 2️⃣ GitOps: Argo CD + struttura repo ZTP

### 🔹 OpenShift GitOps (Argo CD)

CR da conoscere:

* `ArgoCD`
* `Application`
* `ApplicationSet`

**In ZTP:**

* Argo **non deploya app**
* Argo **deploya cluster**

👉 Frase chiave da corso:

> *“In ZTP, il cluster è l’applicazione.”*

---

### 🔹 Repository ZTP (struttura standard)

Devi saperla **disegnare a mano**:

```
ztp-repo/
├── siteconfigs/          # legacy
├── clusters/             # new model
├── policies/
│   ├── common/
│   ├── group/
│   └── site/
├── clusterImageSets/
├── kustomization.yaml
```

👉 Andrea F. ha detto che:

* fanno **kustomize local only**
* debug prima del push

Perfetto: è *best practice*.

---

## 3️⃣ Policy: il cuore vero di ACM in ZTP

### 🔹 Policy Generator (NON legacy)

CR:

```yaml
kind: PolicyGenerator
```

Devi conoscere **molto bene**:

* `policyDefaults`
* `placement`
* `remediationAction: enforce`
* `orderManifests`

👉 Punto chiave:

> Telecom usa **misto legacy + policy generator**
> → **preparati a spiegare la migrazione**

---

### 🔹 Policy (ACM)

CR:

* `Policy`
* `ConfigurationPolicy`
* `OperatorPolicy`

Casi ZTP:

* installazione operatori
* configurazioni day-2
* certificati
* utenti (Must)

---

### 🔹 Placement & ClusterSet

CR fondamentali:

* `ManagedClusterSet`
* `ManagedClusterSetBinding`
* `Placement`

👉 Domande tipiche:

> *“Perché devo usare un namespace per agire sul cluster?”*

(che hai già centrato perfettamente nelle discussioni precedenti)

---

## 4️⃣ CGU & TALM (che Andrea F. ha citato esplicitamente)

### 🔹 TALM – Topology Aware Lifecycle Manager

CR **importantissimi**:

* `ClusterGroupUpgrade`
* `ManagedClusterView`

👉 In telco:

* upgrade **scaglionati**
* finestre
* canary
* rollback

Perfetto per:

* spiegare *day-2*
* rispondere a “operazioni macchinose”

---

## 5️⃣ ClusterImageSet & ambiente disconnesso 🔥

### 🔹 ClusterImageSet

CR:

```yaml
kind: ClusterImageSet
```

**Devi saper spiegare:**

* perché va **versionato**
* perché va **bloccato**
* perché `disableUpdateClusterImageSets=true`

---

### 🔹 Image mirroring (disconnected)

Concetti chiave:

* `ImageContentSourcePolicy` (legacy)
* `ImageDigestMirrorSet`
* `ImageTagMirrorSet`

👉 Andrea F. lo ha citato → **ci saranno domande**

---

## 6️⃣ Backup & DR (ODP / Velero)

Componenti:

* OADP operator
* Velero
* backup di:

  * policy
  * BMH
  * ACM resources

👉 Tipica domanda:

> *“Se perdo l’hub, perdo tutto?”*

---

## 7️⃣ Day-2 operations ZTP (pain point reale)

Preparati a spiegare:

* sostituzione nodo
* cambio NIC
* certificati
* utenti
* scaling cluster

E soprattutto:

> **Perché ZTP è eccellente in day-0
> ma complesso in day-2**

---

## 8️⃣ Tool interni (Must)

Qui la cosa importante **non è il tool** ma il pattern:

* ZTP crea:

  * utenti
  * cert
  * RBAC
* Tool esterno usa il cluster **indirettamente**

👉 Pattern:

> *ZTP come “bootstrap identity & access”*

---

## 9️⃣ Argomenti “futuri” che potrebbero saltare fuori

Andrea F. li ha citati:

* KubeVirt (ACM + virtualization)
* monitoring aggregato
* global hub (non ancora centrale per Telecom)

👉 Basta **sapere cosa sono**, non deep dive.

---

## 🧩 Come ti suggerisco di impostare il corso

### Ordine ideale (telco-friendly):

1. **ZTP workflow end-to-end**
2. Git repo → Argo → ACM → Hive
3. Policy Generator
4. ClusterImageSet + disconnected
5. CGU & day-2
6. Limiti attuali + roadmap

---

## 🧠 Frase chiave di chiusura (golden)

> *“In Telecom non usiamo ACM per ‘gestire cluster’, ma per rendere ZTP riproducibile, governabile e auditabile.”*

