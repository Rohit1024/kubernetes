# GitOps with Argo CD & Flux CD Complete Ecosystem Module

We designed and delivered an enterprise-grade GitOps curriculum across 9 dedicated lessons (Lessons 14 through 22):

- **Lesson 14 (GitOps Principles & Argo CD Fundamentals)**:
  - OpenGitOps 4 Core Principles (Declarative, Versioned, Pulled, Reconciled).
  - Pull-based vs. Push-based CI/CD security architecture (preventing API server exposure).
  - Argo CD internal architecture (`argocd-server`, `argocd-repo-server`, `argocd-application-controller`, `argocd-notifications`).
  - Declarative `Application` CRD, self-healing, automatic pruning, drift detection, and the root "App of Apps" pattern.

- **Lesson 15 (Argo CD with Helm, Kustomize, Sync Waves & Hooks)**:
  - In-repo server manifest rendering of Helm charts and Kustomize overlays.
  - Deterministic deployment ordering using Sync Waves (`argocd.argoproj.io/sync-wave`).
  - PreSync, Sync, PostSync, and SyncFail resource hooks for automated database migrations and verification jobs.

- **Lesson 16 (Multi-Cluster Scalability with ApplicationSets)**:
  - Solving multi-tenant and multi-cluster boilerplate via the ApplicationSet Controller.
  - In-depth generator mechanics: Git Directory, Git File, Cluster, List, Matrix, and Merge generators.
  - Progressive Syncs for canary cluster rollouts across staging and production tiers.

- **Lesson 17 (Secret Management & Automated Deployments)**:
  - The GitOps Secrets Dilemma: Comparison of SealedSecrets, ESO, and Argo CD Vault Plugin (AVP).
  - Injecting dynamic secrets via ConfigManagementPlugin (CMP) placeholders without storing plaintext in Git.
  - Argo CD Image Updater: Tracking container registries (GHCR, ECR, GCR) with SemVer constraints and Git commit write-backs.

- **Lesson 18 (Production GitOps Architecture & Argo CD Autopilot)**:
  - Repository topologies (App Code Repo vs. Config Repo, Monorepo vs. Polyrepo).
  - Disaster recovery protocols and rapid cluster re-creation from Git.
  - Argo CD Autopilot structure (`bootstrap/`, `projects/`, `apps/`) and team `AppProject` RBAC isolation.

- **Lesson 19 (Progressive Delivery with Argo Rollouts)**:
  - Replacing Kubernetes `Deployment` with the `Rollout` CRD.
  - Blue/Green deployments with active/preview services and Canary deployments with traffic routing (NGINX, Gateway API, Istio).
  - Automated metric verification via `AnalysisTemplate` querying Prometheus with instant zero-downtime automated rollbacks.

- **Lesson 20 (Pipelines with Argo Workflows & Argo Events)**:
  - Kubernetes-native CI and DAG workflows with isolated container steps, artifact repositories (S3/GCS), and `WorkflowTemplate`.
  - Event-driven automation architecture: `EventSource` (Webhooks, Kafka), `EventBus` (NATS JetStream), and `Sensor` (Workflow/Sync triggers).

- **Lesson 21 (Flux CD Architecture & Image Automation)**:
  - Decentralized microservices controllers: `source-controller`, `kustomize-controller`, `helm-controller`, `notification-controller`, `image-reflector`, and `image-automation`.
  - Core CRDs: `GitRepository`, `OCIRepository`, `Kustomization`, `HelmRelease`, `ImagePolicy`.
  - Native Kubernetes ServiceAccount impersonation for strict multi-tenant RBAC boundaries.

- **Lesson 22 (Argo CD vs. Flux CD Deep Comparison & Selection Matrix)**:
  - Multi-dimensional head-to-head comparison (UI/Dashboards, Multi-cluster topologies, Multi-tenancy, Helm/OCI support, Progressive Delivery).
  - Production decision flowchart for engineering leadership and platform teams.

- **Cheatsheet & References**:
  - Authored `docs/cheatsheet/gitops-argocd-flux-cheatsheet.md` with CLI commands (`argocd`, `kubectl-argo-rollouts`, `flux`), CRD blueprints, and troubleshooting tables.
  - Expanded `docs/references/resources.md` with official documentation and community channels for the Argo and Flux ecosystems.
