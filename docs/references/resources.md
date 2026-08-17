# Kubernetes & GitOps Resources

## Knowledge

### Core Kubernetes
- [Official Kubernetes Documentation](https://kubernetes.io/docs/)
  The ultimate reference source for all API components, YAML schemas, and core concepts. Use for: standard manifest templates, troubleshooting steps, and kubectl CLI references.
- [Google Kubernetes Engine (GKE) Documentation](https://cloud.google.com/kubernetes-engine/docs)
  Comprehensive guide for running K8s workload components specifically on Google Cloud. Use for: GKE load balancing, storage classes, and node scaling.
- [Kubernetes Academy by VMware](https://kubernetes.academy/)
  Free, structured video courses explaining containers, Pods, Deployments, and networking fundamentals. Use for: architectural visual explanations.
- [Killercoda Kubernetes Playgrounds](https://killercoda.com/)
  Interactive web-based terminal environments for practicing K8s. Use for: testing commands quickly when away from a real cluster.

### GitOps & Argo Project Ecosystem
- [OpenGitOps Specification](https://opengitops.dev/)
  The definitive vendor-neutral definition and 4 principles of GitOps managed by the CNCF.
- [Argo CD Official Documentation](https://argo-cd.readthedocs.io/)
  Comprehensive documentation for Argo CD, ApplicationSets, Sync Waves, RBAC, and Web UI configuration.
- [Argo Rollouts Documentation](https://argoproj.github.io/argo-rollouts/)
  Official guide for progressive delivery, canary releases, Blue/Green deployments, and automated Prometheus analysis.
- [Argo Workflows & Events Documentation](https://argoproj.github.io/)
  Kubernetes-native DAG workflow engine and event-driven automation framework.
- [Argo CD Vault Plugin (AVP)](https://argocd-vault-plugin.readthedocs.io/)
  Secret injection tool for Argo CD without storing plaintext secrets in Git.
- [Argo CD Image Updater](https://argocd-image-updater.readthedocs.io/)
  Automated container registry polling and Git write-back tool.
- [Argo CD Autopilot](https://argocd-autopilot.readthedocs.io/)
  Opinionated GitOps repository bootstrapping and multi-project structure automation.

### Flux CD Ecosystem
- [Flux CD Documentation](https://fluxcd.io/)
  Official documentation for Flux v2, Source Controller, Kustomize Controller, Helm Controller, and Image Automation.
- [Flux Multi-Tenancy Guide](https://fluxcd.io/flux/guides/multi-tenancy/)
  Enterprise security patterns and ServiceAccount impersonation for multi-team clusters.

---

## Wisdom (Communities)

- [r/kubernetes Subreddit](https://reddit.com/r/kubernetes)
  High-signal community of platform engineers and developers sharing real-world setups, troubleshooting tips, and post-mortems. Use for: debugging obscure issues and comparing deployment patterns.
- [CNCF Slack Workspace](https://slack.cncf.io/)
  Official CNCF Slack containing `#argo-cd`, `#argo-rollouts`, `#argo-workflows`, `#flux`, `#kubernetes-users`, and `#gke` channels where maintainers and practitioners collaborate in real-time. Use for: asking specific setup questions and getting architectural reviews.
- [Argo Project GitHub Discussions](https://github.com/argoproj/argo-cd/discussions)
  Community forum for architectural RFCs, plugin discussions, and design patterns.
