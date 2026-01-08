# Red Hat Quay – One Day Training Deck

> Questo deck combina **slide esistenti dal workshop** + **slide mancanti create ex-novo** per coprire l’intero programma del corso di un giorno.

---

## 1. Identifying the Components and Features of Red Hat Quay

### Slide 1 – Course Introduction

* Obiettivi del corso
* Agenda della giornata
* Target audience

### Slide 2 – What Is Red Hat Quay (rif. workshop slide 5)

* Enterprise container registry
* Supported deployment models
* SaaS vs Self-managed

### Slide 3 – Benefits of an Enterprise Registry (rif. workshop slide 5, 27)

* Scalabilità
* Sicurezza
* Controllo degli accessi
* Distribuzione globale delle immagini

### Slide 4 – Red Hat Quay Core Features (rif. workshop slide 6, 85, 121–122)

* Repository mirroring
* Geo-replication
* Image scanning (Clair)
* RBAC e tenancy
* Build integrati
* Notifiche
* Metriche

### Slide 5 – Quay.io vs Self‑Managed Quay (rif. workshop slide 7–9)

| Feature                  | Quay    | Quay.io |
| ------------------------ | ------- | ------- |
| Gestione infrastruttura  | Cliente | Red Hat |
| Configurazione sicurezza | Sì      | No      |
| Mirroring                | Sì      | No      |
| Geo‑replication          | Sì      | Sì      |

---

## 2. Quay Architecture

### Slide 6 – Quay Architecture Overview (rif. workshop slide 13–14)

* Quay application
* Database (PostgreSQL)
* Object storage
* Redis
* Clair

### Slide 7 – Stateful vs Stateless Components

* Componenti stateful
* Impatti su HA

### Slide 8 – Geo‑replication Architecture (rif. workshop slide 46)

* Multiple region deployment
* Storage locality
* Single logical registry

---

## 3. Quay on OpenShift

### Slide 9 – Why Quay on OpenShift

* Operator lifecycle management
* Autoscaling
* Integrated monitoring

### Slide 10 – Quay Operators Overview (rif. workshop slide 86, 92–93)

* Quay Operator
* Quay Container Security Operator
* Quay Bridge Operator

### Slide 11 – Quay Operator Managed Components (rif. workshop slide 99, 103)

* PostgreSQL
* Redis
* Object storage
* Clair
* Routes & TLS

---

## 4. Deploying and Configuring Red Hat Quay

### Slide 12 – Installation Requirements

* OpenShift 4.5+
* CPU / RAM
* Permissions

### Slide 13 – Object Storage Options

* S3 compatible
* ODF Multicloud Gateway
* HA ODF

### Slide 14 – QuayRegistry CRD

```yaml
apiVersion: quay.redhat.com/v1
kind: QuayRegistry
metadata:
  name: central
  namespace: registry
spec:
  configBundleSecret: init-config-bundle-secret
```

### Slide 15 – Configuration Bundle

* Secret con config.yaml
* Certificati
* Lifecycle della configurazione

### Slide 16 – Modifying Quay Configuration

* New secret
* Update QuayRegistry
* Rolling update

---

## 5. Repository Model and RBAC

### Slide 17 – Quay Tenancy Model

* Organization
* Repository
* Team
* User

### Slide 18 – RBAC Model

* Read / Write / Admin
* Team roles

### Slide 19 – Organizations via Web UI

* Create org
* Manage teams
* Assign permissions

### Slide 20 – Quay API & Tokens

* OAuth applications
* Token scopes
* Security best practices

### Slide 21 – Tag Management & Time Machine

* Tag expiration
* Image GC
* Recovery

---

## 6. Multicluster and Multicloud Use Cases

### Slide 22 – Deployment Patterns (rif. workshop slide 31–32)

* Central registry
* Multiple registries
* Hybrid

### Slide 23 – Benefits of Central Registry

* Cost
* Security
* Storage efficiency

### Slide 24 – Mirroring & Disconnected Clusters

* Upstream mirroring
* Offline clusters

### Slide 25 – Quay Bridge Operator (rif. workshop slide 68–74)

* Namespace ↔ organization
* Robot accounts
* ImageStreams sync

---

## 7. Restricting Image Registry Sources

### Slide 26 – Why Restrict Registries

* Supply chain security
* Compliance

### Slide 27 – OpenShift Image Controller

```yaml
apiVersion: config.openshift.io/v1
kind: Image
metadata:
  name: cluster
spec:
  registrySources:
    allowedRegistries:
      - quay.io
      - registry.redhat.io
```

### Slide 28 – Impact on Cluster Nodes

* MCO drain
* Rolling updates

### Slide 29 – RHACM Governance Policy

```yaml
kind: Policy
spec:
  remediationAction: enforce
```

---

## 8. Deploying from Quay to the Cluster Fleet

### Slide 30 – Robot Accounts

* Org‑scoped
* Token‑based
* Least privilege

### Slide 31 – Creating Robot Accounts (API)

```json
{"name":"finance+deployer","token":"****"}
```

### Slide 32 – Using Robot Accounts in OpenShift

```yaml
imagePullSecrets:
- name: quay-pull-secret
```

### Slide 33 – Fleet‑wide Deployments

* RHACM
* Bridge Operator
* Global pull secret

---

## 9. Operations and Day‑2

### Slide 34 – Monitoring and Metrics

* Prometheus endpoint
* Grafana dashboard

### Slide 35 – Backup and Disaster Recovery

* DB
* Object storage
* Config bundle

### Slide 36 – Troubleshooting

* Logs
* Health checks
* Common issues

---

## 10. Course Summary

### Slide 37 – Key Takeaways

* Quay as enterprise registry
* Operator‑based lifecycle
* Secure multicluster patterns

### Slide 38 – Next Steps

* Advanced Quay topics
* RHACS integration
* Automation
