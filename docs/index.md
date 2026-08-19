---
icon: lucide/book-open
---

# Learn Kubernetes

Welcome to the **Kubernetes Learning Portal**! This documentation is structured to help you understand Kubernetes concepts from the ground up, master CLI workflows, deploy production-grade workloads on Google Kubernetes Engine (GKE), and build end-to-end GitOps delivery pipelines.

Whether you're a developer, DevOps engineer, or system administrator, you'll find step-by-step modular lessons, hands-on debugging scenarios, real-world interview challenges, and quick reference cheatsheets.

---

## :mortar_board: Curriculum Modules & Lessons

The curriculum is organized into **4 sequential modules** containing **27 comprehensive lessons**:

### Module 1: Core Kubernetes Architecture & Workloads

Foundational primitives, cluster architecture, workload controllers, storage, and networking on Google Kubernetes Engine.

1. **[Lesson 1: Introduction to Kubernetes & Prerequisites](lessons/0001-what-is-kubernetes-and-prerequisites.md)**
    - Understanding container orchestration, control plane vs. worker node components, and CLI prerequisites.
2. **[Lesson 2: Pod Anatomy & Configuration](lessons/0002-pod-anatomy.md)**
    - Container network namespaces (`localhost`), shared volumes, sidecar patterns, and Pod lifecycles.
3. **[Lesson 3: Node Scheduling, Deployment Strategies & Autoscaling](lessons/0003-node-scheduling-deployment-strategies-autoscaling.md)**
    - Labels, node selectors, taints/tolerations, RollingUpdate vs. Recreate strategies, and Horizontal Pod Autoscalers (HPA).
4. **[Lesson 4: Service-to-Service Communication & DNS](lessons/0004-service-communication.md)**
    - Internal service discovery, Service types (ClusterIP, NodePort, LoadBalancer), kube-proxy, and CoreDNS.
5. **[Lesson 5: Stateless/Stateful App Configuration & Secrets](lessons/0005-stateless-stateful-secrets-gcp.md)**
    - Decoupling configuration with ConfigMaps, managing sensitive Secrets, and GCP Secret Manager synchronization.
6. **[Lesson 6: Ingress & GKE Load Balancing](lessons/0006-ingress-gke-load-balancing.md)**
    - Ingress controllers, path/host routing rules, TLS termination, and Google Cloud Load Balancing integration.
7. **[Lesson 7: Persistent Volumes, PVCs & StorageClasses](lessons/0007-pv-pvc-storageclasses.md)**
    - Dynamic volume provisioning with StorageClasses, PersistentVolumeClaims, and access modes.
8. **[Lesson 8: GKE Gateway API](lessons/0008-gke-gateway-api.md)**
    - Advanced traffic management with GatewayClass, Gateway, HTTPRoute, traffic splitting, and multi-tenant routing.
9. **[Lesson 9: Pod Lifecycle, Resource Allocation, and Health Probes](lessons/0009-resources-probes-graceful-shutdown.md)**
    - Tuning CPU/Memory requests & limits, Startup/Liveness/Readiness probes, and graceful zero-downtime termination.
10. **[Lesson 10: Capstone Project](lessons/0010-capstone-project.md)**
    - Deploying a production-ready, highly available multi-tier web application stack from scratch.

---

### Module 2: Packaging, CI/CD & Operations

Packaging applications for repeatability, building automated deployment pipelines, and managing cluster lifecycle operations.

11. **[Lesson 11: Helm Package Manager](lessons/0011-helm-package-manager.md)**
    - Chart structure, templating engine, `values.yaml` customization, dependency management, and release versioning.
12. **[Lesson 12: CI/CD with GitHub Actions & GKE](lessons/0012-github-actions-cicd-gke.md)**
    - Automated delivery pipelines with GitHub Actions, keyless authentication via Workload Identity Federation (WIF), and GKE deployment.
13. **[Lesson 13: Zero-Downtime Cluster Upgrades](lessons/0013-zero-downtime-cluster-upgrades.md)**
    - Control plane and worker node upgrade workflows, Surge Upgrades, Pod Disruption Budgets (PDBs), and graceful node draining.

---

### Module 3: GitOps & Progressive Delivery (Argo CD & Flux CD)

Enterprise GitOps automation, multi-cluster fleet management, progressive canary rollouts, event-driven pipelines, and GitOps engine comparisons.

14. **[Lesson 14: GitOps Core Principles & Argo CD Fundamentals](lessons/0014-gitops-principles-and-argocd-fundamentals.md)**
    - OpenGitOps 4 principles, pull-based vs. push-based architectures, Argo CD components, `Application` CRDs, and the App of Apps pattern.
15. **[Lesson 15: Argo CD with Helm, Kustomize, Sync Waves & Hooks](lessons/0015-argo-helm-kustomize-sync-waves.md)**
    - Native manifest templating, deterministic sync waves, PreSync/PostSync database migration hooks, and health assessments.
16. **[Lesson 16: Multi-Cluster & Multi-Tenant Scalability with ApplicationSets](lessons/0016-argo-applicationsets.md)**
    - Scaling deployments across multi-region clusters using Git Directory, List, Matrix, and Merge generators with Progressive Syncs.
17. **[Lesson 17: Secret Management (Argo CD Vault Plugin) & Automated Deployments (Argo CD Image Updater)](lessons/0017-argocd-image-updater-and-vault-plugin.md)**
    - Secure in-repo secret injection with Argo CD Vault Plugin (AVP) and automated GitOps container tag updates via Image Updater.
18. **[Lesson 18: Production GitOps Architecture & Bootstrapping with Argo CD Autopilot](lessons/0018-argocd-autopilot-repo-structure.md)**
    - Monorepo vs. Polyrepo repository structures, environment promotion patterns, team AppProjects, and cluster bootstrapping with Autopilot.
19. **[Lesson 19: Progressive Delivery with Argo Rollouts](lessons/0019-argo-rollouts-progressive-delivery.md)**
    - Canary and Blue/Green release strategies, active traffic shaping (NGINX/Gateway API), and automated Prometheus analysis with auto-rollback.
20. **[Lesson 20: Event-Driven Automation & Pipelines with Argo Workflows & Argo Events](lessons/0020-argo-workflows-and-argo-events.md)**
    - Kubernetes-native DAG pipelines with Argo Workflows and event-driven triggers via EventSource, EventBus, and Sensor.
21. **[Lesson 21: Flux CD (Flux v2) GitOps Engine & Image Automation](lessons/0021-fluxcd-fundamentals-and-architecture.md)**
    - Decentralized Kubernetes-native controllers (source, kustomize, helm, notification), OCI registry artifacts, and automated commit writes.
22. **[Lesson 22: Argo CD vs. Flux CD Deep Comparison & Production Selection Guide](lessons/0022-argocd-vs-fluxcd-comparison.md)**
    - Architectural comparison, security postures, developer experience, and a comprehensive production decision matrix.

---

### Module 4: Event-Driven Autoscaling with KEDA & GitOps Synchronization

Dynamic event-driven scaling, scale-to-zero workloads, timezone-aware cron schedules, and solving GitOps state drift with Argo CD.

23. **[Lesson 23: KEDA Fundamentals & Event-Driven Autoscaling Architecture](lessons/0023-keda-fundamentals-and-architecture.md)**
    - Understanding KEDA Operator, Metrics Server, scale-to-zero (`0 ↔ N`) lifecycle, and CRD models (`ScaledObject`, `TriggerAuthentication`).
24. **[Lesson 24: Scaling Workloads with KEDA External Metric Triggers](lessons/0024-keda-external-metrics-scalers.md)**
    - Prometheus PromQL triggers, message queue depth (RabbitMQ, Kafka lag, AWS SQS), fallback safety, and HPA stabilization behavior.
25. **[Lesson 25: Time-Based Autoscaling with KEDA Cron Scaler & Multi-Trigger Composition](lessons/0025-keda-cron-and-scheduled-scaling.md)**
    - Proactive pre-warming for business hours, IANA timezone schedules, and multi-trigger MAX evaluation rules.
26. **[Lesson 26: Solving the GitOps Tug-of-War: KEDA + Argo CD Drift Resolution](lessons/0026-keda-argocd-gitops-integration-and-drift.md)**
    - Resolving the Replicas Tug-of-War, configuring Argo CD `ignoreDifferences` on `/spec/replicas`, omitting replica keys in Git, and scaling Argo Rollouts.
27. **[Lesson 27: Event-Driven Batch Processing with KEDA ScaledJobs & Secure Authentication](lessons/0027-keda-scaledjobs-and-batch-processing.md)**
    - Spawning discrete run-to-completion batch Jobs with `ScaledJob`, keyless Cloud Workload Identity with `TriggerAuthentication`, and DLQ resilience.

---

## :compass: Additional Learning Resources

Explore diagnostic guides, interview preparation, and quick command references:

- **:hammer_and_wrench: [Interactive Manifest Generator](generator/index.md)**: Visually customize and generate production-ready Kubernetes YAML manifests (Deployments, StatefulSets, Gateway API, KEDA scalers, NetworkPolicies, PDBs) with one-click copy and presets.
- **:beetle: [Debugging Workflows](debugging/index.md)**: Real-world diagnostic recipes for `CrashLoopBackOff`, `ImagePullBackOff`, CoreDNS lookup failures, and GKE database connectivity issues.
- **:question: [Interview Questions](interview/index.md)**: Challenging real-world troubleshooting scenarios covering tricky Pod restarts, silent networking blackholes, and multi-zone storage scheduling.
- **:zap: [Cheatsheets](cheatsheet/index.md)**: Quick syntax references for `kubectl`, Docker CLI, Argo CD, Flux CLI, and KEDA autoscaling.
- **:link: [References & Tools](references/index.md)**: Curated official documentation, local cluster development tools (Kind, Minikube, k3d), and ecosystem resources.

---

!!! tip "Getting Started"
    If you are new to Kubernetes, start with **[Lesson 1: Introduction to Kubernetes & Prerequisites](lessons/0001-what-is-kubernetes-and-prerequisites.md)** to build a strong mental model of cluster architecture before deploying workloads.
