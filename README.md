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

The portal is organized into 12 structured lessons:

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
11. **[Lesson 11: Helm Package Manager](docs/lessons/0011-helm-package-manager.md)** — Template, parameterize, and version repeatable Kubernetes manifest sets.
12. **[Lesson 12: CI/CD with GitHub Actions & GKE](docs/lessons/0012-github-actions-cicd-gke.md)** — Build automated delivery pipelines using GitHub Actions, Workload Identity Federation, and DNS-based credential endpoints.

## Create your own lessons with Agent Teach

