---
icon: lucide/graduation-cap
---

# Lessons

Welcome to the lessons section. Here, you will learn the core concepts and architecture of Kubernetes step by step.

## Lesson Outline

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
11. **[Lesson 11: Helm Package Manager](0011-helm-package-manager.md)**
    - Packaging, parameterizing, and versioning repeatable Kubernetes manifest sets.

---

[← Home](../index.md)
