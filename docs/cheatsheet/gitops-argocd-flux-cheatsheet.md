# GitOps, Argo CD, and Flux CD cheatsheet

Practical command reference and manifest blueprints for **Argo CD**, **Argo Rollouts**, **Argo Workflows**, **Argo Events**, and **Flux CD**.

---

## 1. Argo CD CLI (argocd) quick reference

### Authentication and cluster management
```bash
# Login to Argo CD server
argocd login <ARGOCD_SERVER_IP>:443 --username admin --password <PASSWORD> --insecure

# Update admin password
argocd account update-password

# Add an external target Kubernetes cluster
argocd cluster add <CONTEXT_NAME> --name prod-us-east

# List connected clusters
argocd cluster list
```

### Application lifecycle operations
```bash
# Create an application declaratively from CLI
argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default

# List all applications and health status
argocd app list

# Get detailed info and live resource tree
argocd app get guestbook

# Trigger a manual sync
argocd app sync guestbook

# Sync with force and pruning enabled
argocd app sync guestbook --force --prune

# Roll back an application to a previous revision
argocd app rollback guestbook <REVISION_ID>

# Delete an application
argocd app delete guestbook --cascade
```

### Manifest diff and troubleshooting
```bash
# View live diff between Git desired state and live cluster
argocd app diff guestbook

# Stream live application pod logs
argocd app logs guestbook --follow

# Set parameter overrides dynamically
argocd app set guestbook -p image.tag=v2.1.0
```

---

## 2. Argo Rollouts CLI (kubectl argo rollouts)

```bash
# Get live visual status of a rollout
kubectl argo rollouts get rollout <ROLLOUT_NAME> --watch

# Update image (triggers canary or blue-green rollout)
kubectl argo rollouts set image <ROLLOUT_NAME> <CONTAINER_NAME>=<IMAGE>:<TAG>

# Promote a paused canary rollout to next step
kubectl argo rollouts promote <ROLLOUT_NAME>

# Fully promote a rollout immediately (skips remaining steps)
kubectl argo rollouts promote <ROLLOUT_NAME> --full

# Abort a running rollout and rollback to stable
kubectl argo rollouts abort <ROLLOUT_NAME>

# Retry an aborted rollout
kubectl argo rollouts retry <ROLLOUT_NAME>

# Launch the local Web UI
kubectl argo rollouts dashboard
```

---

## 3. Flux CD CLI (flux) quick reference

### Cluster verification and bootstrap
```bash
# Pre-flight cluster checks
flux check --pre

# Bootstrap Flux v2 onto a GitHub repository
flux bootstrap github \
  --owner=<GITHUB_USER_OR_ORG> \
  --repository=<REPO_NAME> \
  --branch=main \
  --path=clusters/production \
  --personal
```

### Sources and reconciliations
```bash
# List all Git, Helm, and OCI sources
flux get sources all

# List all Kustomizations
flux get kustomizations

# List all HelmReleases
flux get helmreleases -A

# Force an immediate reconciliation (bypass polling interval)
flux reconcile source git <SOURCE_NAME>
flux reconcile kustomization <APP_NAME> --with-source
flux reconcile helmrelease <RELEASE_NAME> --with-source

# Suspend or resume reconciliation
flux suspend kustomization <APP_NAME>
flux resume kustomization <APP_NAME>
```

---

## 4. Key custom resource YAML snippets

### A. Argo CD Application (with auto-sync and self-heal)
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: microservice
  namespace: argocd
  finalizers: [resources-finalizer.argocd.argoproj.io]
spec:
  project: default
  source:
    repoURL: 'https://github.com/org/repo.git'
    targetRevision: main
    path: overlays/prod
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### B. Argo Rollout (Canary with analysis)
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: web-app
spec:
  replicas: 5
  strategy:
    canary:
      analysis:
        templates:
          - templateName: error-rate-check
      steps:
        - setWeight: 20
        - pause: { duration: 5m }
        - setWeight: 50
        - pause: { duration: 10m }
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web
        image: app:v2.0.0
```

### C. Flux v2 GitRepository and Kustomization
```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: podinfo
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/stefanprodan/podinfo
  ref:
    branch: master
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: podinfo
  namespace: flux-system
spec:
  interval: 5m
  sourceRef:
    kind: GitRepository
    name: podinfo
  path: ./kustomize
  prune: true
```

---

## 5. Common troubleshooting and debugging

| Symptom / issue | Potential cause | Fix / remediation |
| :--- | :--- | :--- |
| **Argo CD `OutOfSync` and will not self-heal** | Live cluster mutation or invalid CRD schema | Check `argocd app diff <app>` and examine mutating webhooks or ignored differences. |
| **Sync waves blocked** | Earlier wave resource never reached `Healthy` | Run `kubectl describe` on pods in the earlier wave; resolve failing readiness probes. |
| **Rollout stuck in `Paused`** | Waiting for step duration or manual promotion | Run `kubectl argo rollouts get rollout <name>` to inspect paused timer or metric evaluation. |
| **Flux `ArtifactFailed`** | Invalid Git credentials or unreachable repo | Run `flux get sources git` and verify Secret reference or SSH deploy key permissions. |
| **Argo CD Vault Plugin missing values** | Misconfigured CMP sidecar or missing Secret path | Check `kubectl logs -n argocd -l app.kubernetes.io/name=argocd-repo-server -c avp` and verify Vault token. |

---

[← Cheatsheets overview](./index.md) | [KEDA autoscaling cheatsheet →](./keda-autoscaling-cheatsheet.md)
