# Lesson 16: Multi-Cluster & Multi-Tenant Scalability with ApplicationSets

## 1. The Multi-Cluster & Multi-App Challenge

In an enterprise environment, you may have **50 microservices** running across **10 Kubernetes clusters** (Development, Staging, QA, Production across multiple regions). Writing and maintaining 500 individual `Application` CRD manifests is error-prone and unmaintainable.

**ApplicationSets** solve this problem by introducing automated templating and generation for Argo CD `Application` resources.

```mermaid
graph TD
    AppSet["ApplicationSet Controller\n(Single ApplicationSet CRD)"] -->|Evaluates Generators| Gen["Generators\n(Git Repos, Cluster Secrets, Lists)"]
    Gen -->|Generates N Application CRDs| App1["Application: auth (dev-us-east)"]
    Gen -->|Generates N Application CRDs| App2["Application: auth (prod-us-east)"]
    Gen -->|Generates N Application CRDs| App3["Application: auth (prod-eu-west)"]
    Gen -->|Generates N Application CRDs| App4["Application: payment (prod-us-east)"]
```

---

## 2. Anatomy of an ApplicationSet

An `ApplicationSet` CRD consists of two main components:
1. **`generators`**: Determine *what parameters* to extract (e.g., directory names, cluster URLs, environment metadata).
2. **`template`**: An Argo CD `Application` manifest blueprint containing `{{ parameter }}` placeholders that get rendered dynamically.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: cluster-addons
  namespace: argocd
spec:
  generators:
    - list:
        elements:
          - env: development
            clusterName: dev-cluster
            server: https://10.0.0.1
          - env: staging
            clusterName: staging-cluster
            server: https://10.0.0.2
          - env: production
            clusterName: prod-cluster
            server: https://10.0.0.3
  template:
    metadata:
      name: '{{env}}-monitoring'
    spec:
      project: default
      source:
        repoURL: https://github.com/my-org/gitops-addons.git
        targetRevision: main
        path: 'addons/monitoring/{{env}}'
      destination:
        server: '{{server}}'
        namespace: monitoring
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

---

## 3. Core Generators Overview

Argo CD provides several powerful generators to fit different operational topologies:

| Generator | How It Works | Primary Use Case |
| :--- | :--- | :--- |
| **List Generator** | Uses a static list of key-value maps inside the manifest | Small clusters or fixed environments |
| **Cluster Generator** | Queries Argo CD cluster secrets using Kubernetes label selectors | Dynamically targeting all clusters labeled `env: prod` |
| **Git Directory Generator** | Scans folders in a Git repository matching a path pattern | "One folder per microservice" repo designs |
| **Git File Generator** | Reads JSON/YAML configuration files from a Git repo | Centralized cluster metadata registries |
| **Matrix Generator** | Computes the Cartesian product of two child generators | Deploying *all apps in Git* to *all target clusters* |
| **Merge Generator** | Combines two generators, using one to override another | Default configs with cluster-specific overrides |

---

## 4. The Matrix Generator: Git Folders × Multi-Clusters

The **Matrix Generator** is the standard enterprise pattern for deploying a suite of microservices across multiple target clusters simultaneously.

```mermaid
graph LR
    subgraph Matrix["Matrix Generator"]
        GitGen["Git Directory Generator\n(3 Microservices)"]
        ClusterGen["Cluster Generator\n(3 Target Clusters)"]
    end
    Matrix -->|3 x 3 Matrix Product| Apps["9 Dynamic Argo CD Applications Generated"]
```

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: enterprise-workloads
  namespace: argocd
spec:
  generators:
    - matrix:
        generators:
          # Generator 1: Find all microservice directories
          - git:
              repoURL: https://github.com/my-org/services.git
              revision: main
              directories:
                - path: apps/*
          # Generator 2: Find all production clusters
          - clusters:
              selector:
                matchLabels:
                  tier: production
  template:
    metadata:
      name: '{{name}}-{{path.basename}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/my-org/services.git
        targetRevision: main
        path: '{{path}}/overlays/{{metadata.labels.environment}}'
      destination:
        server: '{{server}}'
        namespace: '{{path.basename}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

---

## 5. Progressive Syncs with ApplicationSets

When upgrading a shared infrastructure service across dozens of clusters, you do not want to update all clusters simultaneously. **Progressive Syncs** enable rolling updates across stages.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: rolling-cluster-upgrade
  namespace: argocd
spec:
  strategy:
    type: RollingSync
    rollingSync:
      steps:
        - matchExpressions:
            - key: environment
              operator: In
              values:
                - dev
          maxUpdate: 100%
        - matchExpressions:
            - key: environment
              operator: In
              values:
                - staging
          maxUpdate: 100%
        - matchExpressions:
            - key: environment
              operator: In
              values:
                - prod
          maxUpdate: 25% # Upgrade production clusters gradually
  generators:
    - clusters: {}
  template:
    metadata:
      name: '{{name}}-ingress'
    spec:
      project: default
      source:
        repoURL: https://charts.bitnami.com/bitnami
        chart: nginx-ingress-controller
        targetRevision: 9.9.0
      destination:
        server: '{{server}}'
        namespace: ingress
```

---

## Test Your Knowledge

1. Which ApplicationSet generator computes the Cartesian product of two different generators?
   - [ ] A) The Matrix Generator
   - [ ] B) The Merge Generator
   
   *Answer:* A) The Matrix Generator - Correct! The Matrix generator combines two child generators to generate applications matching all combinatorial pairs.

2. In an ApplicationSet Progressive Sync strategy, what prevents production clusters from updating if staging fails?
   - [ ] A) Subsequent steps pause until current steps succeed
   - [ ] B) Rollback triggers immediately upon cluster drain
   
   *Answer:* A) Subsequent steps pause until current steps succeed - Correct! Progressive sync steps execute sequentially; a failure in an earlier stage blocks promotion to the next stage.

---

## Interactive Win: Creating a Directory-Based ApplicationSet

Let's configure an ApplicationSet that watches a Git directory containing multiple sub-services and creates individual applications automatically.

### Step 1: Write the ApplicationSet Manifest
Save this as `appset-git-dirs.yaml`:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: guestbook-services
  namespace: argocd
spec:
  generators:
    - git:
        repoURL: https://github.com/argoproj/argocd-example-apps.git
        revision: HEAD
        directories:
          - path: 'guestbook*'
  template:
    metadata:
      name: '{{path.basename}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/argoproj/argocd-example-apps.git
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{path.basename}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
```

### Step 2: Apply the ApplicationSet
```bash
kubectl apply -f appset-git-dirs.yaml
```

### Step 3: Inspect Generated Applications
```bash
# View all generated Application resources
kubectl get applications -n argocd

# Inspect the ApplicationSet controller status
kubectl get applicationset guestbook-services -n argocd -o yaml
```

---

## Recommended Primary Resource
- [Argo CD ApplicationSet Official Documentation](https://argo-cd.readthedocs.io/en/stable/user-guide/application-set/)
- [Progressive Syncs Specification](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Progressive-Syncs/)

---
**Working on multi-cluster setup or custom cluster generators?** Ask in chat, and we can configure your RBAC secret mappings together!

[← Lesson 15: Argo CD with Helm, Kustomize & Sync Waves](./0015-argo-helm-kustomize-sync-waves.md) | [Lesson 17: Secret Management & Image Updater →](./0017-argocd-image-updater-and-vault-plugin.md)
