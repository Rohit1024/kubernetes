# Learn Kubernetes Portal

A structured, self-paced learning portal and hands-on laboratory environment designed to master Kubernetes concepts, workload deployments on Google Kubernetes Engine (GKE), and rapid diagnostic troubleshooting.

This project is built and served using **Zensical**, a modern Markdown-based static site generator.

---

## 📂 Repository Structure

* **`docs/`**: Main documentation and portal source files.
  * **`docs/index.md`**: Portal homepage.
  * **`docs/mission.md`**: Current learning mission, goals, constraints, and out-of-scope boundaries.
  * **`docs/lessons/`**: Step-by-step markdown lessons covering core Kubernetes components and debug practices.
  * **`docs/cheatsheet/`**: Command reference cheat sheets (e.g., `kubectl` debugging).
  * **`docs/references/`**: Compiled reference links, community directories, and resources (`docs/references/resources.md`).
* **`learning-records/`**: Sequence of learning record documents tracking established prior knowledge, non-trivial achievements, and decision logs.
* **`overrides/`**: Directory containing layout and style overrides for the Zensical theme.
* **`zensical.toml`**: The Zensical project configuration file (metadata, navigation links, and theme palettes).
* **`.agents/`**: Workspace-scoped custom agent instructions (the `teach` skill).

---

## 🚀 Running Locally

This project uses Python and `uv` for package management.

### 1. Install Dependencies
Ensure you have `uv` installed, then synchronize dependencies:
```bash
uv sync
```

### 2. Run the Development Server
Serve the documentation locally on [http://localhost:8000](http://localhost:8000):
```bash
uv run zensical serve
```

### 3. Build the Static Site
Generate the compiled HTML files inside the `site/` folder:
```bash
uv run zensical build
```

---

## 📖 Curriculum Outline

The portal is organized into 4 sequential modules containing 27 structured lessons:

### Module 1: Core Kubernetes Architecture & Workloads
1. **[Lesson 1: Introduction to Kubernetes & Prerequisites](docs/lessons/0001-what-is-kubernetes-and-prerequisites.md)** — Learn container orchestration fundamentals, Control Plane/Worker Node architectures, and required pre-requisites.
2. **[Lesson 2: Pod Anatomy & Configuration](docs/lessons/0002-pod-anatomy.md)** — Understand Pod resource isolation, container network sharing (`localhost`), shared volumes, and the sidecar design pattern.
3. **[Lesson 3: Node Scheduling, Deployment Strategies & Autoscaling](docs/lessons/0003-node-scheduling-deployment-strategies-autoscaling.md)** — Master selectors, rolling updates, and horizontal auto-scaling.
4. **[Lesson 4: Service-to-Service Communication & DNS](docs/lessons/0004-service-communication.md)** — Establish internal cluster routing with ClusterIP, NodePort, LoadBalancer service types and CoreDNS namespace rules.
5. **[Lesson 5: Stateless/Stateful App Configuration & Secrets](docs/lessons/0005-stateless-stateful-secrets-gcp.md)** — Inject settings and credentials using ConfigMaps, native Secrets, and GCP Secret Manager sync.
6. **[Lesson 6: Ingress & GKE Load Balancing](docs/lessons/0006-ingress-gke-load-balancing.md)** — Route external HTTP/HTTPS traffic to services.
7. **[Lesson 7: Persistent Volumes, PVCs & StorageClasses](docs/lessons/0007-pv-pvc-storageclasses.md)** — Request and mount dynamic persistent cloud storage volumes.
8. **[Lesson 8: GKE Gateway API](docs/lessons/0008-gke-gateway-api.md)** — Implement advanced traffic routing, path-based matching, and multi-tenant Gateway configuration.
9. **[Lesson 9: Pod Lifecycle, Resource Allocation, and Health Probes](docs/lessons/0009-resources-probes-graceful-shutdown.md)** — Configure CPU/Memory limits, tune Startup/Liveness/Readiness probes, and script graceful terminations.
10. **[Lesson 10: Capstone Project](docs/lessons/0010-capstone-project.md)** — Deploy a highly-available, multi-tier web application stack.

### Module 2: Packaging, CI/CD & Operations
11. **[Lesson 11: Helm Package Manager](docs/lessons/0011-helm-package-manager.md)** — Template, parameterize, and version repeatable Kubernetes manifest sets.
12. **[Lesson 12: CI/CD with GitHub Actions & GKE](docs/lessons/0012-github-actions-cicd-gke.md)** — Build automated delivery pipelines using GitHub Actions, Workload Identity Federation, and DNS-based credential endpoints.
13. **[Lesson 13: Zero-Downtime Cluster Upgrades](docs/lessons/0013-zero-downtime-cluster-upgrades.md)** — Upgrade control plane and worker nodes without downtime using Surge Upgrades, PDBs, and graceful termination.

### Module 3: GitOps & Progressive Delivery (Argo CD & Flux CD)
14. **[Lesson 14: GitOps Core Principles & Argo CD Fundamentals](docs/lessons/0014-gitops-principles-and-argocd-fundamentals.md)** — OpenGitOps 4 principles, pull-based vs push-based CI/CD, Argo CD architecture, declarative `Application` CRD, self-healing, and App of Apps pattern.
15. **[Lesson 15: Argo CD with Helm, Kustomize, Sync Waves & Hooks](docs/lessons/0015-argo-helm-kustomize-sync-waves.md)** — Native Helm/Kustomize rendering, deterministic sync waves, PreSync/PostSync database migration hooks, and health checks.
16. **[Lesson 16: Multi-Cluster Scalability with ApplicationSets](docs/lessons/0016-argo-applicationsets.md)** — Eliminate boilerplate across clusters using ApplicationSet Generators and Progressive Syncs.
17. **[Lesson 17: Secret Management (Argo CD Vault Plugin) & Automated Deployments (Argo CD Image Updater)](docs/lessons/0017-argocd-image-updater-and-vault-plugin.md)** — GitOps secrets with Argo CD Vault Plugin (AVP) and automated container upgrades with Image Updater.
18. **[Lesson 18: Production GitOps Architecture & Argo CD Autopilot](docs/lessons/0018-argocd-autopilot-repo-structure.md)** — Monorepo vs Polyrepo setups, multi-environment promotion strategies, team AppProjects, and cluster bootstrapping with Autopilot.
19. **[Lesson 19: Progressive Delivery with Argo Rollouts](docs/lessons/0019-argo-rollouts-progressive-delivery.md)** — Canary and Blue/Green strategies with traffic routing (Ingress NGINX/Gateway API) and automated Prometheus metric analysis & rollback.
20. **[Lesson 20: Event-Driven Automation & Pipelines with Argo Workflows & Argo Events](docs/lessons/0020-argo-workflows-and-argo-events.md)** — Kubernetes-native DAG pipelines with Argo Workflows and event-driven automation (EventSource, EventBus, Sensor) with Argo Events.
21. **[Lesson 21: Flux CD (Flux v2) GitOps Engine & Image Automation](docs/lessons/0021-fluxcd-fundamentals-and-architecture.md)** — Decentralized microservices controllers (source, kustomize, helm, notify), OCI artifacts, automated Git commit writes, and multi-tenancy.
22. **[Lesson 22: Argo CD vs. Flux CD Deep Comparison & Production Selection Guide](docs/lessons/0022-argocd-vs-fluxcd-comparison.md)** — Architectural comparison, security models, developer experience, and production decision matrix.

### Module 4: Event-Driven Autoscaling with KEDA & GitOps Synchronization
23. **[Lesson 23: KEDA Fundamentals & Event-Driven Autoscaling Architecture](docs/lessons/0023-keda-fundamentals-and-architecture.md)** — Understanding KEDA Operator, Metrics Server, scale-to-zero (`0 ↔ N`) lifecycle, and CRD models (`ScaledObject`, `TriggerAuthentication`).
24. **[Lesson 24: Scaling Workloads with KEDA External Metric Triggers](docs/lessons/0024-keda-external-metrics-scalers.md)** — Prometheus PromQL triggers, message queue depth (RabbitMQ, Kafka lag, AWS SQS), fallback safety, and HPA stabilization behavior.
25. **[Lesson 25: Time-Based Autoscaling with KEDA Cron Scaler & Multi-Trigger Composition](docs/lessons/0025-keda-cron-and-scheduled-scaling.md)** — Proactive pre-warming for business hours, IANA timezone schedules, and multi-trigger MAX evaluation rules.
26. **[Lesson 26: Solving the GitOps Tug-of-War: KEDA + Argo CD Drift Resolution](docs/lessons/0026-keda-argocd-gitops-integration-and-drift.md)** — Resolving the Replicas Tug-of-War, configuring Argo CD `ignoreDifferences` on `/spec/replicas`, omitting replica keys in Git, and scaling Argo Rollouts.
27. **[Lesson 27: Event-Driven Batch Processing with KEDA ScaledJobs & Secure Authentication](docs/lessons/0027-keda-scaledjobs-and-batch-processing.md)** — Spawning discrete run-to-completion batch Jobs with `ScaledJob`, keyless Cloud Workload Identity with `TriggerAuthentication`, and DLQ resilience.

## Create your own lessons with Agent Teach

