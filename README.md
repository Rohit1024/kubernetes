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

The portal is organized into 10 structured lessons:

1. **[Lesson 1: Pod Anatomy & CrashLoopBackOff](docs/lessons/0001-pod-anatomy-and-crashloopbackoff.md)** — Understand Pod boundaries and debug crashing containers using logs and events.
2. **[Lesson 2: Node Scheduling, Deployment Strategies & Autoscaling](docs/lessons/0002-node-scheduling-deployment-strategies-autoscaling.md)** — Master selectors, rolling updates, and horizontal auto-scaling.
3. **[Lesson 3: Service Communication, DNS & Troubleshooting](docs/lessons/0003-service-communication-dns-troubleshooting.md)** — Dive into internal cluster networking and DNS resolving.
4. **[Lesson 4: Stateless/Stateful App Configuration & Secrets](docs/lessons/0004-stateless-stateful-secrets-gcp.md)** — Configure apps using ConfigMaps, Secrets, and environment variables.
5. **[Lesson 5: Ingress & GKE Load Balancing](docs/lessons/0005-ingress-gke-load-balancing.md)** — Route external HTTP/HTTPS traffic to GKE cluster workloads.
6. **[Lesson 6: Persistent Volumes, PVCs & StorageClasses](docs/lessons/0006-pv-pvc-storageclasses.md)** — Request and mount dynamic persistent storage.
7. **[Lesson 7: GKE Gateway API](docs/lessons/0007-gke-gateway-api.md)** — Work with advanced routing and multi-tenant traffic rules.
8. **[Lesson 8: Resource Limits, Probes & Graceful Shutdown](docs/lessons/0008-resources-probes-graceful-shutdown.md)** — Optimize resource requests, configure probes, and handle terminations gracefully.
9. **[Lesson 9: Capstone Project](docs/lessons/0009-capstone-project.md)** — Deploy a multi-tier, highly-available application with external ingress.
10. **[Lesson 10: Helm Package Manager](docs/lessons/0010-helm-package-manager.md)** — Package, parameterize, and deploy repeatable Kubernetes workloads.

## Create your own lessons with Agent Teach

