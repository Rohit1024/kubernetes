# Helm Package Manager & Core Open Source Charts

We designed and delivered Lesson 10 covering:
- **Helm Fundamentals**: Moving away from static YAML duplication to dynamic template rendering with Go template engines using Charts, Releases, and customizable `values.yaml` files.
- **Repository and Release Commands**: Managing remote registries (`repo add`/`update`), configuring installs and upgrades (`install`, `upgrade --install`, `-f values.yaml`), rollback options (`rollback`), and auditing deployment histories (`history`, `list`).
- **Core Open Source Infrastructure Charts**: In-depth review of industry-standard Helm charts used to bootstrap production environments, including:
  - `ingress-nginx/ingress-nginx` (routing/load balancing)
  - `prometheus-community/kube-prometheus-stack` (observability/metrics)
  - `jetstack/cert-manager` (HTTPS certificate provisioning)
  - `bitnami/postgresql` / `bitnami/redis` (stateful services setup)
- **Knowledge Validation & Lab**: Implementing a custom standalone Redis deployment configured with persistence overrides using values file injections to practice package customization.

The curriculum was updated in `NOTES.md` and a premium interactive guide was created at `lessons/0010-helm-package-manager.html` (mirrored at `docs/lessons/0010-helm-package-manager.md`).
