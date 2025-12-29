Perfetto, questo è **un ottimo quiz “da architetto”**, non da operatore 👍
Lo imposto in modo che **ogni scenario abbia UNA risposta chiaramente corretta**, così eviti ambiguità in aula (fondamentale).

Userò sempre le **stesse 3 opzioni**:

* **A)** Gatekeeper (OPA)
* **B)** Compliance Operator
* **C)** RHACM Governance Policy

Alla fine trovi **soluzioni + spiegazione breve**, stile “takeaway”.

---

# 🧠 Quiz – Sicurezza e Governance in OpenShift / RHACM

---

## **Domanda 1**

**Scenario:**
Voglio impedire che vengano deployate immagini container che usano il tag `:latest`.

**Quale strumento uso?**
A) Gatekeeper
B) Compliance Operator
C) RHACM Governance Policy

---

## **Domanda 2**

**Scenario:**
Voglio verificare che i nodi del cluster rispettino lo standard **Essential 8 (E8)** e ottenere un report di compliance.

**Quale strumento uso?**
A) Gatekeeper
B) Compliance Operator
C) RHACM Governance Policy

---

## **Domanda 3**

**Scenario:**
Voglio assicurarmi che **tutti i cluster della mia fleet** abbiano installato il Compliance Operator.

**Quale strumento uso?**
A) Gatekeeper
B) Compliance Operator
C) RHACM Governance Policy

---

## **Domanda 4**

**Scenario:**
Voglio bloccare la creazione di namespace che non hanno una label obbligatoria `environment`.

**Quale strumento uso?**
A) Gatekeeper
B) Compliance Operator
C) RHACM Governance Policy

---

## **Domanda 5**

**Scenario:**
Voglio controllare se su un cluster sono presenti **configurazioni non conformi** già esistenti (es. sysctl errati sui nodi).

**Quale strumento uso?**
A) Gatekeeper
B) Compliance Operator
C) RHACM Governance Policy

---

## **Domanda 6**

**Scenario:**
Voglio applicare la **stessa regola di sicurezza** (es. niente privilegi `privileged: true`) su **decine di cluster** da un punto centrale.

**Quale strumento uso?**
A) Gatekeeper
B) Compliance Operator
C) RHACM Governance Policy

---

## **Domanda 7**

**Scenario:**
Voglio impedire che un Pod venga creato se richiede più di 2 CPU, **prima** che venga salvato in etcd.

**Quale strumento uso?**
A) Gatekeeper
B) Compliance Operator
C) RHACM Governance Policy

---

## **Domanda 8**

**Scenario:**
Voglio dimostrare a un auditor che i miei cluster OpenShift rispettano uno standard di sicurezza certificabile.

**Quale strumento uso?**
A) Gatekeeper
B) Compliance Operator
C) RHACM Governance Policy

---

## **Domanda 9**

**Scenario:**
Voglio applicare una policy che **verifica e (se necessario) crea** una specifica configurazione Kubernetes (es. Namespace, LimitRange, ResourceQuota).

**Quale strumento uso?**
A) Gatekeeper
B) Compliance Operator
C) RHACM Governance Policy

---

## **Domanda 10**

**Scenario:**
Voglio bloccare una configurazione **in tempo reale**, senza aspettare scansioni o riconciliazioni.

**Quale strumento uso?**
A) Gatekeeper
B) Compliance Operator
C) RHACM Governance Policy

---

# ✅ Soluzioni e spiegazioni

| #  | Risposta corretta               | Spiegazione                                                           |
| -- | ------------------------------- | --------------------------------------------------------------------- |
| 1  | **A – Gatekeeper**              | È una validazione **admission-time**, perfetta per policy su manifest |
| 2  | **B – Compliance Operator**     | E8 è uno **standard di compliance** basato su scan e report           |
| 3  | **C – RHACM Governance Policy** | RHACM governa **la distribuzione degli operatori**                    |
| 4  | **A – Gatekeeper**              | Validazione strutturale degli oggetti Kubernetes                      |
| 5  | **B – Compliance Operator**     | Analizza **stato esistente** dei nodi e del cluster                   |
| 6  | **C – RHACM Governance Policy** | Centralizza e distribuisce policy su più cluster                      |
| 7  | **A – Gatekeeper**              | Blocca l’oggetto **prima del commit**                                 |
| 8  | **B – Compliance Operator**     | Produce **evidenze e report** per audit                               |
| 9  | **C – RHACM Governance Policy** | Può **verificare e riconciliare** risorse                             |
| 10 | **A – Gatekeeper**              | Unico strumento che lavora **in real-time**                           |

---

 

## **Domanda 11 (Avanzata)**

**Scenario:**
Voglio applicare una policy che **crea risorse in cluster specifici solo se hanno un certo label** (`environment=prod`), e voglio che la policy sia gestita **centralmente** su tutti i cluster.

**Quale strumento uso?**
A) Gatekeeper
B) Compliance Operator
C) RHACM Governance Policy

---

## **Domanda 12 (Avanzata)**

**Scenario:**
Voglio controllare la compliance di un cluster rispetto a un **profilo personalizzato di sicurezza**, ma voglio **modificare alcune regole del profilo** per adattarle alle mie esigenze aziendali.

**Quale strumento uso?**
A) Gatekeeper
B) Compliance Operator
C) RHACM Governance Policy

---

## **Domanda 13 (Avanzata)**

**Scenario:**
Voglio che le policy **segnalino una violazione** se una configurazione non è conforme, ma **non blocchino automaticamente** la creazione delle risorse, lasciando agli sviluppatori il tempo di correggere.

**Quale strumento uso?**
A) Gatekeeper
B) Compliance Operator
C) RHACM Governance Policy

---

# ✅ Soluzioni avanzate

| #  | Risposta corretta               | Spiegazione                                                                             |
| -- | ------------------------------- | --------------------------------------------------------------------------------------- |
| 11 | **C – RHACM Governance Policy** | RHACM permette di usare **Placement e PlacementBinding** per selezionare cluster target |
| 12 | **B – Compliance Operator**     | TailoredProfile permette di **customizzare regole** di profili predefiniti              |
| 13 | **A – Gatekeeper**              | Con `enforcementAction: dryrun` Gatekeeper segnala violazioni senza bloccare le risorse |

 

