Questa è una domanda cruciale per evitare confusione negli studenti. La risposta breve è: **Non sono due modalità alternative, ma una è l'evoluzione "industriale" dell'altra.**

Ecco come puoi spiegare il passaggio in modo chiaro: **`SiteConfig` è il "Motore", ZTP GitOps è la "Catena di Montaggio".**

### 1. La Relazione tra i due concetti

Non sono due fasi diverse dell'installazione di RHACM, né modalità completamente distinte. Sono legati da una relazione di **contenuto** vs **metodo di consegna**.

* **SiteConfig / ClusterInstance (Il *Cosa*):** È la tecnologia di templating che hai appena mostrato. È l'operatore che prende un file YAML "semplice" e lo trasforma in tutti i complessi manifest necessari a OpenShift (AgentClusterInstall, BareMetalHost, ecc.).
* **ZTP GitOps (Il *Come*):** È l'architettura che usa ArgoCD per "sparare" quei file `SiteConfig` o `ClusterInstance` sul cluster Hub automaticamente, leggendoli da Git, invece di farli applicare a mano da un umano con `oc apply`.

### 2. Come spiegarlo agli studenti (L'Analogia)

Puoi usare questa metafora per rendere il concetto immediato:

> *"Fino ad ora abbiamo imparato a scrivere una lettera (il `ClusterInstance`) e a imbucarla a mano nella cassetta della posta (l'Hub) usando `oc apply`. L'Hub l'ha letta e ha creato il cluster.
> Adesso immaginiamo di dover spedire 5.000 lettere a 5.000 destinatari diversi. Non possiamo imbucarle a mano una alla volta.
> **ZTP GitOps** è il servizio postale automatizzato. Noi mettiamo tutte le lettere in un archivio centrale (Git), e un sistema automatico (ArgoCD) le prende e le consegna all'Hub per noi. Ma il contenuto della lettera (il `SiteConfig/ClusterInstance`) è esattamente lo stesso che avete appena studiato."*

### 3. Differenze Chiave: Manuale vs GitOps

Puoi presentare questa tabella per visualizzare il cambio di paradigma:

| Caratteristica | Quello che abbiamo appena fatto (SiteConfig Manuale) | Quello che faremo ora (ZTP GitOps) |
| --- | --- | --- |
| **Trigger** | Comando `oc apply -f clusterinstance.yaml` | `git commit` & `git push` |
| **Chi applica?** | L'Amministratore (Sysadmin) | ArgoCD (OpenShift GitOps) |
| **Scala** | Adatto per 1-10 cluster (Lab/PoC) | Adatto per 100-1000+ cluster (Edge/Telco) |
| **Motore sottostante** | **SiteConfig Operator** | **SiteConfig Operator** |

### 4. Il "Gancio" Tecnico

Per collegare la lezione precedente a quella nuova, mostra loro che il file YAML non cambia drasticamente.

1. **Lezione Precedente:** Avete creato un file `clusterinstance.yaml` e lanciato `oc apply`. L'operatore SiteConfig ha rilevato la risorsa e avviato il provisioning.
2. **Lezione ZTP:** Prenderemo quello stesso tipo di definizione, la metteremo in una cartella dentro un repo Git, e configureremo ArgoCD per "guardare" quella cartella. ArgoCD vedrà il file e farà `oc apply` al posto vostro. L'operatore SiteConfig si sveglierà e farà il lavoro esattamente come prima.

### In sintesi per la classe

*"Ragazzi, `SiteConfig` è lo strumento che semplifica la configurazione rendendola un template. ZTP GitOps è l'automazione che ci permette di usare quello strumento su scala massiva senza impazzire. ZTP **usa** SiteConfig."*

---


### Introduzione alla Demo: OpenShift GitOps ZTP

Buongiorno a tutti. Prima di avviare la simulazione del terminale, è fondamentale capire il **contesto** di ciò che stiamo per vedere.

Oggi parliamo di come gestire l'infrastruttura su scala massiva, uno scenario tipico delle Telco (5G) o della grande distribuzione (Edge Computing).

#### 1. Il Problema: La Scala "Edge"

Immaginate di dover installare e gestire OpenShift non su 10 server nel vostro data center, ma su **5.000 piccole postazioni** sparse per il paese (es. antenne telefoniche o supermercati).

* Non potete mandare un tecnico con una chiavetta USB in ogni sito.
* Non potete configurare ogni cluster manualmente. Sarebbe troppo lento e propenso ad errori.

#### 2. La Soluzione: GitOps ZTP (Zero Touch Provisioning)

Per risolvere questo problema, Red Hat utilizza un approccio chiamato **ZTP**.

* **ZTP (Zero Touch Provisioning):** L'obiettivo è collegare il cavo di rete e l'alimentazione al server "nudo" (Bare Metal) e... basta. La macchina si accende, scarica il sistema operativo e si configura da sola.
* **GitOps:** Ma *chi* dice alla macchina come configurarsi? **Git**. Tutta la configurazione (indirizzi IP, dischi, VLAN) è scritta in file YAML dentro un repository Git. Git è l'unica "fonte di verità".

#### 3. L'Architettura: Hub & Spoke

In questo scenario abbiamo due attori principali:

1. **L'Hub Cluster (La Torre di Controllo):** È il cluster centrale dove gira *Red Hat Advanced Cluster Management (RHACM)*. È il "cervello".
2. **I Managed Clusters (Gli Spoke):** Sono le migliaia di piccoli cluster remoti che dobbiamo installare.

#### 4. Cosa vedremo nella Demo?

La demo che sta per partire mostra il **"Giorno Zero"**.
Non stiamo ancora installando le antenne. Stiamo preparando la **Torre di Controllo (l'Hub)** affinché sia capace di leggere le istruzioni da Git e trasmetterle ai server remoti.

Nello specifico, vedrete l'amministratore eseguire queste operazioni:

1. **Estrarre gli strumenti:** Recuperare dal container ufficiale Red Hat i template necessari.
2. **Patchare ArgoCD:** Configurare il motore GitOps (ArgoCD) che vive sull'Hub.
3. **Collegare Git:** Dire all'Hub: *"Guarda, le istruzioni per costruire i nuovi siti le trovi in QUESTO repository Git, su QUESTO branch"*.
4. **Verificare la sincronizzazione:** Assicurarsi che l'Hub abbia "capito" e sia pronto a lavorare.

Una volta terminata questa procedura, l'Hub sarà pronto. Da quel momento in poi, per installare un nuovo sito, basterà fare un *Commit* su Git, e l'Hub farà tutto il resto automaticamente.

Avviamo ora la sessione sul terminale dell'amministratore...

---

Questa introduzione dovrebbe dare agli studenti la "mappa mentale" necessaria per capire perché quei comandi `sed` e `oc apply` sono così importanti.



[sysadmin@bastion-host ~]$ # 1. VERIFICA PRELIMINARE DEGLI OPERATORI
[sysadmin@bastion-host ~]$ # Verifichiamo che RHACM e GitOps siano installati e in stato 'Succeeded'
[sysadmin@bastion-host ~]$ oc get csv -A | grep -E "advanced-cluster-management|openshift-gitops"
open-cluster-management    advanced-cluster-management.v2.11.0   Advanced Cluster Management   2.11.0    Succeeded
openshift-gitops           openshift-gitops-operator.v1.14.0     Red Hat OpenShift GitOps      1.14.0    Succeeded

[sysadmin@bastion-host ~]$ # Tutto ok. Procediamo con la preparazione ZTP.
[sysadmin@bastion-host ~]$ 
[sysadmin@bastion-host ~]$ # 2. ESTRAZIONE DEI TOOL ZTP
[sysadmin@bastion-host ~]$ # Creiamo una directory di lavoro ed estraiamo i template dall'immagine ufficiale
[sysadmin@bastion-host ~]$ mkdir -p ztp-project
[sysadmin@bastion-host ~]$ cd ztp-project
[sysadmin@bastion-host ztp-project]$ export ZTP_IMG=registry.redhat.io/openshift4/ztp-site-generate-rhel8:v4.18
[sysadmin@bastion-host ztp-project]$ podman run --log-driver=none --rm $ZTP_IMG extract /home/ztp --tar | tar x -C .
[sysadmin@bastion-host ztp-project]$ ls -F
out/

[sysadmin@bastion-host ztp-project]$ # Vediamo cosa è stato estratto
[sysadmin@bastion-host ztp-project]$ tree out/argocd/ -L 2
out/argocd/
├── deployment
│   ├── argocd-openshift-gitops-patch.json
│   ├── clusters-app.yaml
│   ├── kustomization.yaml
│   └── policies-app.yaml
└── example
    ├── policygentemplates
    └── siteconfig

3 directories, 4 files

[sysadmin@bastion-host ztp-project]$ # 3. CONFIGURAZIONE DELL'HUB (ARGO CD PATCH)
[sysadmin@bastion-host ztp-project]$ # Applichiamo la patch ad ArgoCD per abilitare il plugin PolicyGenerator
[sysadmin@bastion-host ztp-project]$ oc patch argocd openshift-gitops \
> -n openshift-gitops --type=merge \
> --patch-file out/argocd/deployment/argocd-openshift-gitops-patch.json
argocd.argoproj.io/openshift-gitops patched

[sysadmin@bastion-host ztp-project]$ # Verifica rapida che lo stato di ArgoCD sia 'Phase: Available' dopo la patch
[sysadmin@bastion-host ztp-project]$ oc get argocd openshift-gitops -n openshift-gitops -o jsonpath='{.status.phase}'
Available
[sysadmin@bastion-host ztp-project]$ 
[sysadmin@bastion-host ztp-project]$ # -----------------------------------------------------------
[sysadmin@bastion-host ztp-project]$ # FASE 4: CONFIGURAZIONE DEI PUNTATORI AL REPOSITORY GITOPS
[sysadmin@bastion-host ztp-project]$ # -----------------------------------------------------------
[sysadmin@bastion-host ztp-project]$ 
[sysadmin@bastion-host ztp-project]$ # Definiamo le variabili del nostro repository Git
[sysadmin@bastion-host ztp-project]$ export GIT_URL="https://github.com/acme-telco/ztp-site-configs.git"
[sysadmin@bastion-host ztp-project]$ export GIT_BRANCH="main"
[sysadmin@bastion-host ztp-project]$ 
[sysadmin@bastion-host ztp-project]$ # Sostituiamo i valori di default nei template YAML
[sysadmin@bastion-host ztp-project]$ sed -i "s|https://github.com/openshift-kni/cnf-features-deploy.git|$GIT_URL|g" out/argocd/deployment/clusters-app.yaml
[sysadmin@bastion-host ztp-project]$ sed -i "s|targetRevision: master|targetRevision: $GIT_BRANCH|g" out/argocd/deployment/clusters-app.yaml
[sysadmin@bastion-host ztp-project]$ 
[sysadmin@bastion-host ztp-project]$ sed -i "s|https://github.com/openshift-kni/cnf-features-deploy.git|$GIT_URL|g" out/argocd/deployment/policies-app.yaml
[sysadmin@bastion-host ztp-project]$ sed -i "s|targetRevision: master|targetRevision: $GIT_BRANCH|g" out/argocd/deployment/policies-app.yaml
[sysadmin@bastion-host ztp-project]$ 
[sysadmin@bastion-host ztp-project]$ # -----------------------------------------------------------
[sysadmin@bastion-host ztp-project]$ # VERIFICA DELLE MODIFICHE
[sysadmin@bastion-host ztp-project]$ # Controlliamo il contenuto di 'clusters-app.yaml' prima di applicarlo
[sysadmin@bastion-host ztp-project]$ # -----------------------------------------------------------
[sysadmin@bastion-host ztp-project]$ cat out/argocd/deployment/clusters-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: clusters
  namespace: openshift-gitops
spec:
  destination:
    server: https://kubernetes.default.svc
    namespace: openshift-gitops
  project: default
  source:
    path: siteconfigs
    repoURL: https://github.com/acme-telco/ztp-site-configs.git
    targetRevision: main
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

[sysadmin@bastion-host ztp-project]$ # Il file punta correttamente al repo 'acme-telco' sul branch 'main'.
[sysadmin@bastion-host ztp-project]$ 
[sysadmin@bastion-host ztp-project]$ # -----------------------------------------------------------
[sysadmin@bastion-host ztp-project]$ # FASE 5: ATTIVAZIONE DELLA PIPELINE ZTP
[sysadmin@bastion-host ztp-project]$ # -----------------------------------------------------------
[sysadmin@bastion-host ztp-project]$ oc apply -f out/argocd/deployment/clusters-app.yaml
application.argoproj.io/clusters created

[sysadmin@bastion-host ztp-project]$ oc apply -f out/argocd/deployment/policies-app.yaml
application.argoproj.io/policies created

[sysadmin@bastion-host ztp-project]$ # -----------------------------------------------------------
[sysadmin@bastion-host ztp-project]$ # FASE 6: VERIFICA FINALE DELLO STATO DI SYNC
[sysadmin@bastion-host ztp-project]$ # -----------------------------------------------------------
[sysadmin@bastion-host ztp-project]$ echo "Waiting for ArgoCD to sync applications..."
Waiting for ArgoCD to sync applications...
[sysadmin@bastion-host ztp-project]$ oc get app -n openshift-gitops
NAME       SYNC STATUS   HEALTH STATUS
clusters   Synced        Healthy
policies   Synced        Healthy

[sysadmin@bastion-host ztp-project]$ # Setup completato. Hub pronto.
[sysadmin@bastion-host ztp-project]$ exit
logout