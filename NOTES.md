# Learning & Teaching Notes

## Learner Focus & Preferences
- **Curriculum**: Comprehensive Kubernetes mastery ranging from core primitives (Pods, Deployments, Storage, Gateway API) to advanced platform engineering, CI/CD, GitOps, and progressive delivery.
- **Tone & Style**: Production-grade, clear architectural breakdowns with Mermaid diagrams, actionable YAML configurations, precise CLI commands, and instant feedback quizzes.
- **GitOps Deep Dive**: Detailed multi-lesson module covering the Argo Project ecosystem (Argo CD, ApplicationSets, Helm/Kustomize, Sync Waves, Vault Plugin, Image Updater, Autopilot, Rollouts, Workflows, Events) and Flux CD (Flux v2, controllers, OCI, image automation, multi-tenancy) with comparative trade-off analysis.

## Completed Modules
1. **Module 1: Core Kubernetes Architecture & Workloads (Lessons 1-10)**
   - Primitives, Pod Anatomy, Scheduling & Autoscaling, Services & DNS, ConfigMaps & Secrets, Ingress, PV/PVC, Gateway API, Probes & Graceful Shutdown, Capstone Project.
2. **Module 2: Packaging, CI/CD & Operations (Lessons 11-13)**
   - Helm Package Manager, GitHub Actions CI/CD with GKE, Zero-Downtime Cluster Upgrades.
3. **Module 3: GitOps & Progressive Delivery (Lessons 14-22)**
   - Lesson 14: GitOps Core Principles & Argo CD Fundamentals
   - Lesson 15: Argo CD with Helm, Kustomize, Sync Waves & Hooks
   - Lesson 16: Multi-Cluster Scalability with ApplicationSets
   - Lesson 17: Secret Management (Argo CD Vault Plugin) & Automated Deployments (Argo CD Image Updater)
   - Lesson 18: Production GitOps Architecture & Argo CD Autopilot
   - Lesson 19: Progressive Delivery with Argo Rollouts
   - Lesson 20: Pipelines with Argo Workflows & Argo Events
   - Lesson 21: Flux CD (Flux v2) GitOps Engine & Image Automation
   - Lesson 22: Argo CD vs. Flux CD Deep Comparison & Production Selection Guide
4. **Module 4: Event-Driven Autoscaling with KEDA & GitOps Synchronization (Lessons 23-27)**
   - Lesson 23: KEDA Fundamentals & Architecture (Scale-to-Zero, Operator, Metrics Adapter)
   - Lesson 24: KEDA External Metric Triggers (Prometheus PromQL, Kafka Lag, RabbitMQ, SQS, Fallbacks)
   - Lesson 25: Time-Based Autoscaling with KEDA Cron Scaler & Multi-Trigger Composition
   - Lesson 26: Solving the GitOps Tug-of-War: KEDA + Argo CD Drift Resolution & Rollouts
   - Lesson 27: Event-Driven Batch Processing with KEDA ScaledJobs & Secure Authentication
