
# 📌 RHACM + ZTP – Checklist completa per il corso Telco

Il cliente telco con ZTP (Zero Touch Provisioning) vuole automatizzare completamente il provisioning e la configurazione dei cluster OpenShift ai siti di rete (core, edge, RAN) senza intervento manuale; RHACM fornisce il controllo centralizzato, GitOps ZTP, policy declarative e integrazione con assisted service per realizzare questa visione in scala

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


### Checklist operativa per l’amministratore dell’Hub
1. **Definire obiettivi e scala**  
   - Numero iniziale e target a regime; tipologia siti (SNO vs small cluster); requisiti disconnected e compliance.  

2. **Provisioning Hub e alta disponibilità**  
   - Deploy dell’**Hub RHACM** ridondato; dimensionamento per il numero di managed clusters previsto; abilitare ArgoCD/GitOps sul Hub.

3. **Abilitare Assisted Service e GitOps ZTP**  
   - Installare e configurare **assisted-service**; abilitare il pipeline GitOps ZTP e i generator (PolicyGenerator/PolicyGenTemplate) per creare SiteConfig e CR di installazione.

4. **Preparare mirror registry e immagini RHCOS/RootFS**  
   - Caricare ISO, rootfs e immagini nel mirror per ambienti disconnected; testare accesso e firma delle immagini.

5. **Strutturare il repository Git ZTP**  
   - Definire branch strategy, template SiteConfig per SNO/small cluster, e manifest per networking, storage e CNF; impostare protezione branch e CI per validazione.

6. **Creare segreti e accessi per host bare‑metal**  
   - Generare managed bare‑metal host secrets, configurare discovery ISO e kernel args per boot automatico dei nodi.

7. **Policy, compliance e configurazioni telco**  
   - Modellare **PolicyGenerator** per profili vRAN/DU; definire policy di sicurezza, network (BGP/VLAN), e firmware management.

8. **Monitoraggio, logging e alerting**  
   - Abilitare monitoraggio installazioni, ArgoCD sync status, e alert per failure di SiteConfig o installazioni fallite.

9. **Test PoC e runbook di rollback**  
   - Eseguire PoC su 2–3 siti; definire runbook per rollback, rimozione site dalla pipeline e pulizia dei contenuti obsoleti.

10. **Sicurezza supply chain e governance**  
    - Firmare immagini/ISO, controllare accessi al repo Git, definire RBAC Hub e policy di approvazione.

11. **Documentazione e formazione**  
    - Documentare playbook ZTP, checklist hardware, e procedure di troubleshooting per operatori sul campo.

---


