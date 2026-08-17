# Lesson 21: Flux CD (Flux v2) GitOps Engine & Image Automation

## 1. What is Flux CD?

**Flux** is a CNCF Graduated GitOps toolkit built from the ground up to leverage Kubernetes' core extension mechanisms (Custom Resource Definitions and specialized controllers).

Unlike Argo CD, which employs a centralized application server with an integrated web UI, **Flux v2 is a decentralized suite of composable Kubernetes controllers**. Each controller is responsible for a single, focused GitOps lifecycle task.

```mermaid
graph TD
    subgraph Sources["Artifact & Code Sources"]
        Git["GitRepository / OCIRepository / HelmRepository"]
    end
    subgraph Controllers["Flux Specialized Controllers"]
        SourceCtrl["source-controller\n(Pulls Git / Helm / OCI / Buckets)"]
        KustCtrl["kustomize-controller\n(Reconciles YAML & Kustomize)"]
        HelmCtrl["helm-controller\n(Reconciles Helm Releases)"]
        NotifyCtrl["notification-controller\n(Inbound Webhooks & Outbound Alerts)"]
        ImageReflect["image-reflector-controller\n(Scans Registry Tags)"]
        ImageAuto["image-automation-controller\n(Commits Tags to Git)"]
    end
    Git --> SourceCtrl
    SourceCtrl -->|Produces Artifact| KustCtrl
    SourceCtrl -->|Produces Artifact| HelmCtrl
    ImageReflect --> ImageAuto
    ImageAuto -->|Pushes Commit| Git
```

---

## 2. Core Flux Custom Resources

Flux models the entire delivery workflow using declarative Kubernetes custom resources:

| Resource Kind | Controller | Purpose |
| :--- | :--- | :--- |
| **`GitRepository`** | `source-controller` | Defines the source Git repository, branch, tag, and polling interval. |
| **`OCIRepository`** | `source-controller` | Pulls Kubernetes manifests packaged as OCI artifacts directly from container registries. |
| **`Kustomization`** | `kustomize-controller` | Defines what path inside the source repository to build and apply, with health checks and dependencies. |
| **`HelmRepository`** | `source-controller` | Registers an HTTP or OCI-based Helm chart repository. |
| **`HelmRelease`** | `helm-controller` | Declaratively manages the installation, upgrades, and values of a Helm chart. |

---

## 3. Declarative Flux Manifests

### A. Defining a Git Source & Kustomization
Flux separates **where the source code lives** (`GitRepository`) from **what path gets applied** (`Kustomization`):

```yaml
# 1. Source Definition
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: fleet-infra
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/my-org/fleet-infra
  ref:
    branch: main
---
# 2. Reconciler Definition
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps-prod
  namespace: flux-system
spec:
  interval: 5m
  targetNamespace: production
  sourceRef:
    kind: GitRepository
    name: fleet-infra
  path: ./apps/production
  prune: true
  wait: true
  timeout: 3m
```

### B. Declarative Helm Chart Deployment
```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: bitnami
  namespace: flux-system
spec:
  interval: 1h
  url: https://charts.bitnami.com/bitnami
---
apiVersion: helm.toolkit.fluxcd.io/v2beta2
kind: HelmRelease
metadata:
  name: redis-cache
  namespace: caching
spec:
  interval: 30m
  chart:
    spec:
      chart: redis
      version: '18.x'
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: flux-system
  values:
    architecture: standalone
    auth:
      enabled: false
```

---

## 4. Flux Automated Image Updates

Flux provides automated Git write-back through two cooperating controllers:
1. **`ImageRepository` / `ImagePolicy`**: Scans the container registry and selects the latest tag according to SemVer or regex rules.
2. **`ImageUpdateAutomation`**: Parses your Git manifests, locates markers (e.g., `{"$imagepolicy": "flux-system:my-app"}`), updates the YAML, and pushes a Git commit.

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: backend-api
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: backend-api
  policy:
    semver:
      range: '>=1.2.0 <2.0.0'
---
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 1m
  sourceRef:
    kind: GitRepository
    name: fleet-infra
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        email: fluxcdbot@users.noreply.github.com
        name: fluxcdbot
      messageTemplate: 'chore(image): update {{range .Updated.Images}}{{..}}{{end}}'
    push:
      branch: main
  update:
    path: ./apps/production
    strategy: Setters
```

---

## 5. Multi-Tenancy & Security Isolation

Flux provides native tenant isolation by leveraging Kubernetes **Service Account Impersonation**. You can restrict a tenant's `Kustomization` or `HelmRelease` so it can only create resources permitted by their specific ServiceAccount RBAC:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: team-billing
  namespace: team-billing
spec:
  serviceAccountName: billing-deployer # Impersonates this SA during reconciliation
  sourceRef:
    kind: GitRepository
    name: billing-repo
  path: ./manifests
  prune: true
```

---

## Test Your Knowledge

1. In Flux v2, which controller is responsible for fetching artifacts from Git, Helm registries, and S3 buckets?
   - [ ] A) The source-controller component
   - [ ] B) The kustomize-controller component
   
   *Answer:* A) The source-controller component - Correct! `source-controller` acts as the universal artifact acquisition engine in the Flux ecosystem.

2. How does Flux ensure multi-tenant security when reconciling manifests for different development teams?
   - [ ] A) By impersonating tenant service accounts
   - [ ] B) By restarting cluster kubelet services
   
   *Answer:* A) By impersonating tenant service accounts - Correct! Flux enforces RBAC boundaries by impersonating a team-specific ServiceAccount when applying resources.

---

## Interactive Win: Inspecting Flux Controller Resources

Let's explore the essential `flux` CLI commands used to inspect sources and trigger instantaneous reconciliations.

### Step 1: Flux CLI Inspection Commands
```bash
# Check overall status of all Flux components
flux check

# List all tracked Git repositories and their current commit SHAs
flux get sources git

# List all Helm releases and their status
flux get helmreleases -A

# List all Kustomizations and their health
flux get kustomizations -A
```

### Step 2: Forcing an Instant Reconciliation (Bypassing Polling Interval)
```bash
# Trigger immediate Git repository fetch
flux reconcile source git fleet-infra

# Trigger immediate Kustomization sync with source
flux reconcile kustomization apps-prod --with-source
```

---

## Recommended Primary Resource
- [Flux CD Official Documentation](https://fluxcd.io/flux/)
- [Flux Multi-Tenancy Best Practices](https://fluxcd.io/flux/guides/multi-tenancy/)

---
**Setting up Flux webhook receiver or S3 bucket sources?** Ask in chat, and we can configure your alerting rules together!

[← Lesson 20: Pipelines with Argo Workflows & Events](./0020-argo-workflows-and-argo-events.md) | [Lesson 22: Argo CD vs. Flux CD Deep Comparison →](./0022-argocd-vs-fluxcd-comparison.md)
