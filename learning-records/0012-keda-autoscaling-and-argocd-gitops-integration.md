# Event-Driven Autoscaling with KEDA & GitOps Synchronization Module

We designed and delivered an advanced autoscaling and GitOps integration curriculum across 5 dedicated lessons (Lessons 23 through 27):

- **Lesson 23 (KEDA Fundamentals & Architecture)**:
  - Limitations of native Kubernetes HPA (inability to scale to/from 0, lack of native event awareness).
  - Internal architecture: `keda-operator` (CRD reconciler and 0 ↔ 1 activations), `keda-operator-metrics-apiserver` (External Metrics API adapter for 1 ↔ N HPA scaling), and admission webhooks.
  - Core CRDs: `ScaledObject`, `ScaledJob`, `TriggerAuthentication`, and `ClusterTriggerAuthentication`.

- **Lesson 24 (Scaling Workloads with KEDA External Metric Triggers)**:
  - 60+ built-in scalers for cloud brokers, messaging systems, and databases.
  - Prometheus PromQL triggers with global rate calculations and `threshold` vs. `activationThreshold` separation.
  - Message-driven scaling for RabbitMQ (`queueLength`), Kafka (`lagThreshold`), and AWS SQS.
  - Production resilience: `fallback` safe replica counts during metrics API outages and custom HPA stabilization windows (`scaleDown.stabilizationWindowSeconds`).

- **Lesson 25 (Time-Based Autoscaling with KEDA Cron Scaler & Multi-Trigger Composition)**:
  - Proactive pre-warming vs. reactive autoscaling for predictable enterprise traffic cycles.
  - IANA timezone awareness (`timezone: America/New_York`), start/end cron definitions, and active window evaluation.
  - Multi-trigger composition rule: KEDA calculates desired replicas for all active triggers and enforces $\max(\text{Trigger}_1, \dots, \text{Trigger}_n)$.

- **Lesson 26 (Solving the GitOps Tug-of-War: KEDA + Argo CD Drift Resolution)**:
  - The Replicas Tug-of-War: Argo CD `selfHeal: true` treating KEDA dynamic replica changes as unauthorized drift, causing destructive pod termination flapping loops.
  - Root cause analysis and production resolution patterns:
    1. Configuring `spec.ignoreDifferences` on `/spec/replicas` at Application and system (`argocd-cm`) levels.
    2. Omitting `spec.replicas` in Git manifests to rely on 3-way strategic merge patch defaults.
    3. Managing auto-generated HPAs (`keda-hpa-*`) and preventing extraneous pruning.
    4. Scaling Argo Rollouts (`argoproj.io/v1alpha1`) with canary releases and KEDA capacity management.

- **Lesson 27 (Event-Driven Batch Processing with KEDA ScaledJobs & Secure Authentication)**:
  - `ScaledObject` (long-running servers) vs. `ScaledJob` (discrete, run-to-completion batch tasks like video transcoding, LLM inference, and DLQ replays).
  - ScaledJob properties: `activeDeadlineSeconds`, `scalingStrategy.targetWorkload`, history limits.
  - Keyless Cloud Workload Identity (`TriggerAuthentication` with GCP WIF and AWS IRSA) vs. Kubernetes Secret references.

- **Cheatsheet & References**:
  - Authored `docs/cheatsheet/keda-autoscaling-cheatsheet.md` with CLI inspection commands, CRD templates, Argo CD drift configurations, and troubleshooting matrices.
  - Updated `docs/references/resources.md` with official KEDA documentation and CNCF Slack community links.
