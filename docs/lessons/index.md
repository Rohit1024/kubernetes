---
icon: lucide/graduation-cap
---

# Lessons

Welcome to the lessons section. Here, you will learn the core concepts, architecture, and production workflows of Kubernetes and modern GitOps step by step.

---

## Module 1: Core Kubernetes Architecture & Workloads

1. **[Lesson 1: Introduction to Kubernetes & Prerequisites](0001-what-is-kubernetes-and-prerequisites.md)**
    - Understanding container orchestration and high-level control plane vs worker node architecture.
    - Prerequisites: Docker, terminal commands, container network concepts.
2. **[Lesson 2: Pod Anatomy & Configuration](0002-pod-anatomy.md)**
    - How containers share network namespaces (localhost) and storage volumes inside a Pod.
    - Implementing the sidecar design pattern and understanding Pod lifecycle phases.
3. **[Lesson 3: Node Scheduling, Deployment Strategies & Autoscaling](0003-node-scheduling-deployment-strategies-autoscaling.md)**
    - Labels, selectors, taints, and scheduling decisions.
    - Deployments (RollingUpdate vs Recreate) and horizontal pod auto-scaling.
4. **[Lesson 4: Service-to-Service Communication & DNS](0004-service-communication.md)**
    - Internal service discovery, Service types (ClusterIP, NodePort, LoadBalancer), CoreDNS resolution.
5. **[Lesson 5: Stateless/Stateful App Configuration & Secrets](0005-stateless-stateful-secrets-gcp.md)**
    - Mounting configurations using ConfigMaps, managing sensitive Secrets, GKE credential sync.
6. **[Lesson 6: Ingress & GKE Load Balancing](0006-ingress-gke-load-balancing.md)**
    - Using Ingress controllers to route external HTTP/HTTPS traffic to backend cluster services.
7. **[Lesson 7: Persistent Volumes, PVCs & StorageClasses](0007-pv-pvc-storageclasses.md)**
    - Requesting storage dynamically with StorageClasses and mounting Persistent Volumes.
8. **[Lesson 8: GKE Gateway API](0008-gke-gateway-api.md)**
    - Advanced traffic routing, path-based matching, and multi-tenant Gateway configuration.
9. **[Lesson 9: Pod Lifecycle, Resource Allocation, and Health Probes](0009-resources-probes-graceful-shutdown.md)**
    - Tuning CPU/Memory requests/limits, configuring liveness/readiness/startup probes, implementing preStop graceful shutdown.
10. **[Lesson 10: Capstone Project](0010-capstone-project.md)**
    - Deploying a complete, highly-available multi-tier web application.

---

## Module 2: Packaging, CI/CD & Operations

11. **[Lesson 11: Helm Package Manager](0011-helm-package-manager.md)**
    - Packaging, parameterizing, and versioning repeatable Kubernetes manifest sets.
12. **[Lesson 12: CI/CD with GitHub Actions & GKE](0012-github-actions-cicd-gke.md)**
    - Automating GKE deployments using GitHub Actions, Workload Identity Federation (WIF), and DNS-based kubeconfig credentials.
13. **[Lesson 13: Zero-Downtime Cluster Upgrades](0013-zero-downtime-cluster-upgrades.md)**
    - Upgrading control plane and worker nodes without downtime using Surge Upgrades, PDBs, and graceful termination.

---

## Module 3: GitOps & Progressive Delivery (Argo CD & Flux CD)

14. **[Lesson 14: GitOps Core Principles & Argo CD Fundamentals](0014-gitops-principles-and-argocd-fundamentals.md)**
    - OpenGitOps 4 principles, pull-based vs push-based CI/CD, Argo CD architecture, declarative `Application` CRD, self-healing, and the App of Apps pattern.
15. **[Lesson 15: Argo CD with Helm, Kustomize, Sync Waves & Hooks](0015-argo-helm-kustomize-sync-waves.md)**
    - Native Helm/Kustomize rendering, deterministic sync waves, PreSync/PostSync database migration hooks, and health checks.
16. **[Lesson 16: Multi-Cluster & Multi-Tenant Scalability with ApplicationSets](0016-argo-applicationsets.md)**
    - Eliminating boilerplate across clusters using ApplicationSet Generators (Git Directory, Cluster, Matrix, Merge) and Progressive Syncs.
17. **[Lesson 17: Secret Management (Argo CD Vault Plugin) & Automated Deployments (Argo CD Image Updater)](0017-argocd-image-updater-and-vault-plugin.md)**
    - Solving GitOps secrets with Argo CD Vault Plugin (AVP) and automating SemVer container upgrades with Argo CD Image Updater.
18. **[Lesson 18: Production GitOps Architecture & Bootstrapping with Argo CD Autopilot](0018-argocd-autopilot-repo-structure.md)**
    - Monorepo vs Polyrepo setups, multi-environment promotion strategies, team AppProjects, and cluster bootstrapping with Autopilot.
19. **[Lesson 19: Progressive Delivery with Argo Rollouts](0019-argo-rollouts-progressive-delivery.md)**
    - Canary and Blue/Green strategies with traffic routing (Ingress NGINX/Gateway API) and automated Prometheus metric analysis & rollback.
20. **[Lesson 20: Event-Driven Automation & Pipelines with Argo Workflows & Argo Events](0020-argo-workflows-and-argo-events.md)**
    - Kubernetes-native DAG pipelines with Argo Workflows and event-driven automation (EventSource, EventBus, Sensor) with Argo Events.
21. **[Lesson 21: Flux CD (Flux v2) GitOps Engine & Image Automation](0021-fluxcd-fundamentals-and-architecture.md)**
    - Decentralized microservices controllers (source, kustomize, helm, notify), OCI artifacts, automated Git commit writes, and multi-tenancy.
22. **[Lesson 22: Argo CD vs. Flux CD Deep Comparison & Production Selection Guide](0022-argocd-vs-fluxcd-comparison.md)**
    - Feature-by-feature architectural comparison, security models, developer experience, and production decision matrix.

---

[← Home](../index.md)
