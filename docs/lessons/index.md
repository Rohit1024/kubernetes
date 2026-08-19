---
icon: lucide/graduation-cap
---

# Lessons

Practical guides on Kubernetes architecture, production operations, GitOps workflows, and event-driven autoscaling.

---

## Module 1: Core Kubernetes architecture and workloads

1. **[Lesson 1: Introduction to Kubernetes architecture and prerequisites](0001-what-is-kubernetes-and-prerequisites.md)**
    - Container orchestration mechanics, control plane components, and worker node internals.
    - Prerequisites: Linux primitives, container runtime basics, and networking fundamentals.
2. **[Lesson 2: Pod anatomy, multi-container patterns, and lifecycle](0002-pod-anatomy.md)**
    - Shared network namespaces, the pause container, IPC, and storage volumes inside a Pod.
    - Native sidecars, container restart policies, and pod phase state transitions.
3. **[Lesson 3: Node scheduling, deployment strategies, and autoscaling](0003-node-scheduling-deployment-strategies-autoscaling.md)**
    - Node affinity, taints, tolerations, and scheduler scoring.
    - Rolling updates, recreate strategies, and horizontal pod autoscaling.
4. **[Lesson 4: Service-to-service communication and DNS](0004-service-communication.md)**
    - ClusterIP, NodePort, LoadBalancer routing, kube-proxy modes, and CoreDNS lookups.
5. **[Lesson 5: Stateless and stateful configuration with Secrets and ConfigMaps](0005-stateless-stateful-secrets-gcp.md)**
    - ConfigMap mounts, secret security, and cloud credential synchronization.
6. **[Lesson 6: Ingress and GKE load balancing](0006-ingress-gke-load-balancing.md)**
    - Ingress controllers, Google Cloud load balancers, and network endpoint groups (NEGs).
7. **[Lesson 7: Persistent volumes, PVCs, and StorageClasses](0007-pv-pvc-storageclasses.md)**
    - Dynamic storage provisioning, reclaim policies, and CSI volume drivers.
8. **[Lesson 8: GKE Gateway API](0008-gke-gateway-api.md)**
    - GatewayClass, Gateway, HTTPRoute, traffic splitting, and multi-tenant ingress routing.
9. **[Lesson 9: Pod lifecycle, resource allocation, and health probes](0009-resources-probes-graceful-shutdown.md)**
    - CPU and memory requests vs limits, liveness and readiness probes, and preStop hooks for graceful shutdown.
10. **[Lesson 10: Capstone project](0010-capstone-project.md)**
    - End-to-end production deployment of a multi-tier web application.

---

## Module 2: Packaging, CI/CD, and operations

11. **[Lesson 11: Helm package manager](0011-helm-package-manager.md)**
    - Chart structure, Go templating, values files, and release lifecycle management.
12. **[Lesson 12: CI/CD with GitHub Actions and GKE](0012-github-actions-cicd-gke.md)**
    - Automated deployment pipelines, Workload Identity Federation (WIF), and keyless cluster authentication.
13. **[Lesson 13: Zero-downtime cluster upgrades](0013-zero-downtime-cluster-upgrades.md)**
    - Control plane and worker node upgrades, surge upgrades, PodDisruptionBudgets, and node draining.

---

## Module 3: GitOps and progressive delivery (Argo CD and Flux CD)

14. **[Lesson 14: GitOps principles and Argo CD fundamentals](0014-gitops-principles-and-argocd-fundamentals.md)**
    - OpenGitOps core tenets, pull-based reconciliation, Application CRDs, and the app-of-apps pattern.
15. **[Lesson 15: Argo CD with Helm, Kustomize, sync waves, and hooks](0015-argo-helm-kustomize-sync-waves.md)**
    - Template rendering, deterministic sync waves, pre-sync database migration hooks, and health assessments.
16. **[Lesson 16: Multi-cluster and multi-tenant management with ApplicationSets](0016-argo-applicationsets.md)**
    - ApplicationSet generators (Git directory, cluster, matrix, merge) and progressive rollouts.
17. **[Lesson 17: Secret management with Vault plugin and automated image updates](0017-argocd-image-updater-and-vault-plugin.md)**
    - Argo CD Vault Plugin secret injection and SemVer container upgrades with Image Updater.
18. **[Lesson 18: Production repository architecture and Argo CD Autopilot](0018-argocd-autopilot-repo-structure.md)**
    - Repository layout strategies, multi-environment promotions, team AppProjects, and cluster bootstrapping.
19. **[Lesson 19: Progressive delivery with Argo Rollouts](0019-argo-rollouts-progressive-delivery.md)**
    - Canary and blue-green deployments, traffic routing, and automated metric analysis rollbacks.
20. **[Lesson 20: Pipelines and event-driven automation with Argo Workflows and Argo Events](0020-argo-workflows-and-argo-events.md)**
    - Kubernetes-native DAG pipelines with Argo Workflows, and event triggers with Argo Events.
21. **[Lesson 21: Flux CD architecture and automated Git write-backs](0021-fluxcd-fundamentals-and-architecture.md)**
    - Source, Kustomize, Helm, and Notification controllers, OCI repositories, and automated image updates.
22. **[Lesson 22: Argo CD and Flux CD comparison](0022-argocd-vs-fluxcd-comparison.md)**
    - Architectural differences, security models, developer workflows, and selection trade-offs.

---

## Module 4: Event-driven autoscaling with KEDA and GitOps synchronization

23. **[Lesson 23: KEDA fundamentals and autoscaling architecture](0023-keda-fundamentals-and-architecture.md)**
    - Custom metrics scaling, scale-to-zero mechanics, and KEDA operator architecture.
24. **[Lesson 24: Workload scaling with external metric triggers](0024-keda-external-metrics-scalers.md)**
    - Prometheus queries, message queue depths (RabbitMQ, Kafka, AWS SQS), and fallback triggers.
25. **[Lesson 25: Scheduled autoscaling with Cron scalers and multi-trigger composition](0025-keda-cron-and-scheduled-scaling.md)**
    - Timezone-aware scheduling, proactive workload pre-warming, and composite trigger evaluations.
26. **[Lesson 26: KEDA and Argo CD replica drift resolution](0026-keda-argocd-gitops-integration-and-drift.md)**
    - Preventing GitOps replica fighting with ignoreDifferences and headless deployment manifests.
27. **[Lesson 27: Batch processing with ScaledJobs and workload identity](0027-keda-scaledjobs-and-batch-processing.md)**
    - Queue-driven run-to-completion Jobs with ScaledJob and keyless Cloud IAM authentication.

---

[← Home](../index.md)
