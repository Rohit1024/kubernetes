---
icon: lucide/cpu
---

# Parameterized Kubernetes Manifest Generator

A comprehensive collection of production-ready, battle-tested Kubernetes YAML manifests. Each manifest includes **parameterized placeholder tokens** and **interactive code annotations** with best-practice guidance.

---

## 🚀 Workloads & Core Deployables

=== "🚀 Stateless Microservice (Deployment + Service + HPA)"

    ```yaml
    # ============================================================================
    # 1. APPLICATION DEPLOYMENT
    # ============================================================================
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: payment-api # (1)!
      namespace: production # (2)!
      labels:
        app.kubernetes.io/name: payment-api
        app.kubernetes.io/tier: backend
    spec:
      replicas: 3 # (3)!
      strategy:
        type: RollingUpdate # (4)!
        rollingUpdate:
          maxSurge: 25%
          maxUnavailable: 0
      selector:
        matchLabels:
          app: payment-api
      template:
        metadata:
          labels:
            app: payment-api
        spec:
          securityContext: # (5)!
            runAsNonRoot: true
            runAsUser: 10001
            fsGroup: 10001
            seccompProfile:
              type: RuntimeDefault
          affinity: # (6)!
            podAntiAffinity:
              preferredDuringSchedulingIgnoredDuringExecution:
                - weight: 100
                  podAffinityTerm:
                    labelSelector:
                      matchLabels:
                        app: payment-api
                    topologyKey: kubernetes.io/hostname
          topologySpreadConstraints: # (7)!
            - maxSkew: 1
              topologyKey: topology.kubernetes.io/zone
              whenUnsatisfiable: ScheduleAnyway
              labelSelector:
                matchLabels:
                  app: payment-api
          terminationGracePeriodSeconds: 45
          containers:
            - name: app
              image: ghcr.io/my-org/payment-api:v2.1.0 # (8)!
              imagePullPolicy: IfNotPresent
              ports:
                - name: http
                  containerPort: 8080 # (9)!
                  protocol: TCP
              resources: # (10)!
                requests:
                  cpu: "250m"
                  memory: "512Mi"
                limits:
                  cpu: "1"
                  memory: "1Gi"
              startupProbe: # (11)!
                httpGet:
                  path: /healthz
                  port: 8080
                initialDelaySeconds: 5
                periodSeconds: 3
                failureThreshold: 20
              livenessProbe: # (12)!
                httpGet:
                  path: /healthz
                  port: 8080
                periodSeconds: 10
                failureThreshold: 3
              readinessProbe: # (13)!
                httpGet:
                  path: /ready
                  port: 8080
                periodSeconds: 5
                failureThreshold: 2
              lifecycle: # (14)!
                preStop:
                  exec:
                    command: ["/bin/sh", "-c", "sleep 10"]
              securityContext:
                allowPrivilegeEscalation: false
                readOnlyRootFilesystem: true
                capabilities:
                  drop: ["ALL"]
    ---
    # ============================================================================
    # 2. COMPANION CLUSTERIP SERVICE
    # ============================================================================
    apiVersion: v1
    kind: Service
    metadata:
      name: payment-api-service # (15)!
      namespace: production
    spec:
      type: ClusterIP
      selector:
        app: payment-api
      ports:
        - name: http
          port: 80 # (16)!
          targetPort: 8080
    ---
    # ============================================================================
    # 3. HORIZONTAL POD AUTOSCALER (HPA)
    # ============================================================================
    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    metadata:
      name: payment-api-hpa
      namespace: production
    spec:
      scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: payment-api
      minReplicas: 3 # (17)!
      maxReplicas: 15 # (18)!
      metrics:
        - type: Resource
          resource:
            name: cpu
            target:
              type: Utilization
              averageUtilization: 70 # (19)!
    ```

    1. **`{{ APP_NAME }}`**: Unique identifier for your workload across the cluster.
    2. **`{{ NAMESPACE }}`**: Target logical namespace for team isolation.
    3. **`{{ REPLICAS }}`**: Initial baseline replicas. Keep $\ge 2$ for High Availability.
    4. **`RollingUpdate`**: Ensures zero downtime with `maxUnavailable: 0` (no dropped traffic).
    5. **`securityContext`**: Adheres to CIS benchmarks (non-root UID `10001`).
    6. **`podAntiAffinity`**: Soft anti-affinity prevents placing two replicas on the same worker node.
    7. **`topologySpreadConstraints`**: Evenly distributes replicas across cloud availability zones (`maxSkew: 1`).
    8. **`{{ CONTAINER_IMAGE }}`**: Immutable image tag (e.g. SemVer or Git commit SHA; avoid `latest`).
    9. **`{{ CONTAINER_PORT }}`**: Port your application listens on inside the container.
    10. **`resources`**: Requests guarantee node scheduling allocation; Limits prevent node starvation (QoS: *Burstable*).
    11. **`startupProbe`**: Gives legacy apps up to 60s ($20 \times 3\text{s}$) to boot before liveness checks kick in.
    12. **`livenessProbe`**: Restarts the container if deadlocked or unresponsive.
    13. **`readinessProbe`**: Controls whether the Pod IP receives traffic from the Service.
    14. **`preStop` hook**: A 10s sleep gives `kube-proxy` time to remove the terminating Pod IP from active routing tables.
    15. **`{{ SERVICE_NAME }}`**: Internal DNS name (`payment-api-service.production.svc.cluster.local`).
    16. **`{{ SERVICE_PORT }}`**: Port exposed internally to other microservices.
    17. **`{{ MIN_REPLICAS }}`**: Minimum capacity maintained during low traffic.
    18. **`{{ MAX_REPLICAS }}`**: Maximum capacity ceiling during extreme traffic surges.
    19. **`{{ CPU_TARGET }}`**: Target CPU threshold (70%) to initiate horizontal scaling.

=== "🐘 Stateful Database (StatefulSet + Headless + PVC)"

    ```yaml
    # ============================================================================
    # 1. DATABASE CREDENTIALS SECRET
    # ============================================================================
    apiVersion: v1
    kind: Secret
    metadata:
      name: postgres-credentials # (1)!
      namespace: database
    type: Opaque
    stringData:
      POSTGRES_USER: "postgres_admin" # (2)!
      POSTGRES_PASSWORD: "ChangeThisSecurePassword123!" # (3)!
      POSTGRES_DB: "orders_db"
    ---
    # ============================================================================
    # 2. HEADLESS SERVICE FOR DETERMINISTIC DNS
    # ============================================================================
    apiVersion: v1
    kind: Service
    metadata:
      name: postgres-headless # (4)!
      namespace: database
    spec:
      clusterIP: None # (5)!
      selector:
        app: postgres-cluster
      ports:
        - name: postgres
          port: 5432 # (6)!
    ---
    # ============================================================================
    # 3. STATEFULSET CONTROLLER
    # ============================================================================
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: postgres-cluster # (7)!
      namespace: database
    spec:
      serviceName: "postgres-headless" # (8)!
      replicas: 3 # (9)!
      podManagementPolicy: OrderedReady # (10)!
      selector:
        matchLabels:
          app: postgres-cluster
      template:
        metadata:
          labels:
            app: postgres-cluster
        spec:
          terminationGracePeriodSeconds: 60
          containers:
            - name: postgres
              image: postgres:15-alpine
              ports:
                - name: postgres
                  containerPort: 5432
              env:
                - name: POSTGRES_USER
                  valueFrom:
                    secretKeyRef:
                      name: postgres-credentials
                      key: POSTGRES_USER
                - name: POSTGRES_PASSWORD
                  valueFrom:
                    secretKeyRef:
                      name: postgres-credentials
                      key: POSTGRES_PASSWORD
                - name: POSTGRES_DB
                  valueFrom:
                    secretKeyRef:
                      name: postgres-credentials
                      key: POSTGRES_DB
              resources:
                requests:
                  cpu: "500m"
                  memory: "1Gi"
                limits:
                  cpu: "2"
                  memory: "4Gi"
              volumeMounts:
                - name: data # (11)!
                  mountPath: /var/lib/postgresql/data
      volumeClaimTemplates: # (12)!
        - metadata:
            name: data
          spec:
            accessModes: ["ReadWriteOnce"] # (13)!
            storageClassName: standard-rwo # (14)!
            resources:
              requests:
                storage: 50Gi # (15)!
    ```

    1. **`{{ SECRET_NAME }}`**: Name of the database credentials Secret.
    2. **`{{ DB_USER }}`**: Administrator database username.
    3. **`{{ DB_PASS }}`**: Strong password (or injected via External Secrets Operator from GCP Secret Manager).
    4. **`{{ HEADLESS_SVC }}`**: Generates DNS A-records: `postgres-cluster-0.postgres-headless.database.svc.cluster.local`.
    5. **`clusterIP: None`**: Explicitly disables virtual IP allocation for direct Pod-to-Pod clustering.
    6. **`{{ DB_PORT }}`**: Default PostgreSQL port (5432).
    7. **`{{ STATEFULSET_NAME }}`**: Generates ordinal Pods: `postgres-cluster-0`, `postgres-cluster-1`, `postgres-cluster-2`.
    8. **`serviceName`**: Links the StatefulSet to the Headless Service.
    9. **`{{ REPLICAS }}`**: Number of database nodes in the cluster.
    10. **`OrderedReady`**: Starts pods sequentially ($0 \to 1 \to 2$) and shuts down in reverse ($2 \to 1 \to 0$).
    11. **`volumeMounts`**: Target directory where the database engine writes state files.
    12. **`volumeClaimTemplates`**: Automatically synthesizes a dedicated PVC for each replica: `data-postgres-cluster-0`.
    13. **`ReadWriteOnce (RWO)`**: Mounted as read-write by a single worker node at a time.
    14. **`{{ STORAGE_CLASS }}`**: StorageClass configured with `volumeBindingMode: WaitForFirstConsumer`.
    15. **`{{ STORAGE_SIZE }}`**: Dedicated disk capacity allocated per replica.

=== "🔀 Gateway API & Canary (HTTPRoute)"

    ```yaml
    # ============================================================================
    # 1. CLUSTER GATEWAY (Platform / Infrastructure Team)
    # ============================================================================
    apiVersion: gateway.networking.k8s.io/v1
    kind: Gateway
    metadata:
      name: external-gateway # (1)!
      namespace: infra # (2)!
    spec:
      gatewayClassName: gke-l7-gxlb # (3)!
      listeners:
        - name: https-listener
          protocol: HTTPS
          port: 443
          allowedRoutes:
            namespaces:
              from: All # (4)!
    ---
    # ============================================================================
    # 2. HTTPROUTE WITH CANARY TRAFFIC SPLIT (Application Developer)
    # ============================================================================
    apiVersion: gateway.networking.k8s.io/v1
    kind: HTTPRoute
    metadata:
      name: checkout-api-route # (5)!
      namespace: payments
    spec:
      parentRefs:
        - name: external-gateway # (6)!
          namespace: infra
          sectionName: https-listener
      hostnames:
        - "api.mycompany.com" # (7)!
      rules:
        # Rule A: Beta Testers Header Match
        - matches:
            - headers:
                - name: "X-Beta-Tester" # (8)!
                  value: "true"
          backendRefs:
            - name: checkout-service-v2
              port: 8080
        # Rule B: Canary Traffic Split (90% Stable, 10% Canary)
        - matches:
            - path:
                type: PathPrefix
                value: /api/checkout # (9)!
          backendRefs:
            - name: checkout-service-v1
              port: 8080
              weight: 90 # (10)!
            - name: checkout-service-v2
              port: 8080
              weight: 10 # (11)!
    ```

    1. **`{{ GATEWAY_NAME }}`**: Cloud Load Balancer instance name.
    2. **`{{ INFRA_NS }}`**: Central infrastructure namespace.
    3. **`gke-l7-gxlb`**: Google Cloud Global External Application Load Balancer GatewayClass.
    4. **`allowedRoutes`**: Permits applications in all namespaces to bind routes to this shared Gateway.
    5. **`{{ ROUTE_NAME }}`**: Application HTTPRoute definition.
    6. **`parentRefs`**: Binds the route directly to the parent Gateway in the `infra` namespace.
    7. **`{{ DOMAIN }}`**: Public DNS hostname routed to this service.
    8. **`Header Match`**: Routes users with `X-Beta-Tester: true` 100% to the v2 canary.
    9. **`{{ PATH_PREFIX }}`**: URL prefix path to match.
    10. **`Stable Weight`**: 90% of production traffic directed to stable `v1`.
    11. **`Canary Weight`**: 10% of production traffic directed to experimental `v2`.

=== "⚡ KEDA Event-Driven Autoscaler (ScaledObject)"

    ```yaml
    apiVersion: keda.sh/v1alpha1
    kind: ScaledObject
    metadata:
      name: order-processor-scaler # (1)!
      namespace: processing
    spec:
      scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: order-processor-worker # (2)!
      minReplicaCount: 0 # (3)!
      maxReplicaCount: 40 # (4)!
      pollingInterval: 15 # (5)!
      cooldownPeriod: 180 # (6)!
      fallback: # (7)!
        failureThreshold: 3
        replicas: 4
      triggers:
        # Trigger 1: Working Hours Pre-Warming (Cron)
        - type: cron
          name: business-hours-prewarm
          metadata:
            timezone: America/New_York # (8)!
            start: 0 8 * * 1-5 # (9)!
            end: 0 18 * * 1-5 # (10)!
            desiredReplicas: "12" # (11)!
        # Trigger 2: Reactive Traffic Bursting (Prometheus)
        - type: prometheus
          name: rps-surge
          metadata:
            serverAddress: http://prometheus.monitoring.svc:9090
            metricName: orders_per_second
            query: sum(rate(orders_processed_total[2m])) # (12)!
            threshold: "50" # (13)!
            activationThreshold: "1" # (14)!
    ```

    1. **`{{ SCALER_NAME }}`**: Name of the KEDA ScaledObject.
    2. **`{{ TARGET_WORKLOAD }}`**: Target Deployment or Argo Rollout to autoscale.
    3. **`minReplicaCount: 0`**: Enables **Scale-to-Zero** when no messages or traffic exist.
    4. **`{{ MAX_REPLICAS }}`**: Maximum burst scaling ceiling (40 Pods).
    5. **`pollingInterval`**: KEDA checks the external event source every 15 seconds.
    6. **`cooldownPeriod`**: Waits 3 minutes of 0 events before scaling down to 0 replicas.
    7. **`fallback`**: Maintains 4 replicas if Prometheus fails or times out.
    8. **`{{ TIMEZONE }}`**: Explicit IANA timezone string for business hours.
    9. **`start`**: 8:00 AM Monday through Friday.
    10. **`end`**: 6:00 PM Monday through Friday.
    11. **`desiredReplicas`**: Number of pre-warmed replicas during business hours.
    12. **`query`**: PromQL rate calculation for incoming workload events.
    13. **`threshold`**: 1 worker Pod added for every 50 orders per second.
    14. **`activationThreshold`**: Scales from $0 \to 1$ when at least 1 event is detected.

=== "⏱️ Scheduled Batch Task (CronJob)"

    ```yaml
    apiVersion: batch/v1
    kind: CronJob
    metadata:
      name: nightly-database-backup # (1)!
      namespace: maintenance
    spec:
      schedule: "0 3 * * *" # (2)!
      timeZone: "Etc/UTC" # (3)!
      concurrencyPolicy: Forbid # (4)!
      successfulJobsHistoryLimit: 3 # (5)!
      failedJobsHistoryLimit: 5 # (6)!
      jobTemplate:
        spec:
          activeDeadlineSeconds: 3600 # (7)!
          backoffLimit: 2 # (8)!
          template:
            spec:
              restartPolicy: OnFailure # (9)!
              containers:
                - name: backup-worker
                  image: gcr.io/my-org/db-backup:v1.2.0
                  command: ["/bin/sh", "-c", "python run_backup.py"]
                  resources:
                    requests:
                      cpu: "500m"
                      memory: "1Gi"
                    limits:
                      cpu: "2"
                      memory: "4Gi"
    ```

    1. **`{{ JOB_NAME }}`**: Name of the batch CronJob.
    2. **`{{ CRON_SCHEDULE }}`**: Standard 5-field cron syntax (`0 3 * * *` = 3:00 AM daily).
    3. **`timeZone`**: Explicit timezone for execution scheduling (K8s 1.27+).
    4. **`concurrencyPolicy: Forbid`**: Prevents new job runs if the previous execution is still running.
    5. **`successfulJobsHistoryLimit`**: Keeps the last 3 successful jobs in `etcd` for audit trails.
    6. **`failedJobsHistoryLimit`**: Keeps the last 5 failed jobs for debugging.
    7. **`activeDeadlineSeconds`**: Forcibly kills the Job if it runs longer than 1 hour (3600s).
    8. **`backoffLimit`**: Retries a failed job at most 2 times before marking it failed.
    9. **`restartPolicy: OnFailure`**: Container restarts inside the same Pod if it exits with a non-zero code.

=== "🛡️ Reliability & Governance (PDB + NetworkPolicy)"

    ```yaml
    # ============================================================================
    # 1. POD DISRUPTION BUDGET (PDB)
    # ============================================================================
    apiVersion: policy/v1
    kind: PodDisruptionBudget
    metadata:
      name: payment-api-pdb # (1)!
      namespace: production
    spec:
      minAvailable: 1 # (2)!
      selector:
        matchLabels:
          app: payment-api
    ---
    # ============================================================================
    # 2. ZERO-TRUST NETWORKPOLICY (DEFAULT DENY + SELECTIVE ALLOW)
    # ============================================================================
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: payment-api-netpol # (3)!
      namespace: production
    spec:
      podSelector:
        matchLabels:
          app: payment-api # (4)!
      policyTypes:
        - Ingress
        - Egress
      # Ingress: Allow ONLY frontend microservices
      ingress:
        - from:
            - podSelector:
                matchLabels:
                  app.kubernetes.io/tier: frontend # (5)!
          ports:
            - protocol: TCP
              port: 8080 # (6)!
      # Egress: Allow ONLY PostgreSQL database + CoreDNS
      egress:
        - to:
            - podSelector:
                matchLabels:
                  app: postgres-cluster # (7)!
          ports:
            - protocol: TCP
              port: 5432
        - to:
            - namespaceSelector: {}
              podSelector:
                matchLabels:
                  k8s-app: kube-dns # (8)!
          ports:
            - protocol: UDP
              port: 53
    ```

    1. **`{{ PDB_NAME }}`**: Name of the Pod Disruption Budget.
    2. **`minAvailable: 1`**: Prevents node maintenance or GKE cluster upgrades from dropping below 1 healthy replica.
    3. **`{{ NETPOL_NAME }}`**: Name of the Zero-Trust NetworkPolicy.
    4. **`podSelector`**: Targets the `payment-api` microservice pods.
    5. **`Ingress Allow`**: Restricts incoming traffic exclusively to Pods bearing `tier: frontend`.
    6. **`{{ APP_PORT }}`**: Port 8080. All other ingress ports are silently dropped.
    7. **`Egress Allow`**: Restricts outgoing traffic exclusively to the database port (5432).
    8. **`CoreDNS Egress`**: Permits DNS resolution on UDP port 53.

---

## 📋 Parameter Reference Matrix

| Parameter Token | Default / Example | Target Resource | Production Recommendation |
| :--- | :--- | :--- | :--- |
| **`{{ APP_NAME }}`** | `payment-api` | Deployment, Service, HPA, PDB | Use kebab-case conforming to DNS-1123 label standards. |
| **`{{ NAMESPACE }}`** | `production` | All Resources | Separate environments (`dev`, `staging`, `prod`) and team tenants. |
| **`{{ REPLICAS }}`** | `3` | Deployment, StatefulSet | Maintain $\ge 2$ replicas for SLA protection during node drains. |
| **`{{ CONTAINER_IMAGE }}`** | `ghcr.io/org/api:v1.0.0` | Deployment, Pod, Job | Always use immutable image tags; avoid `latest`. |
| **`{{ CONTAINER_PORT }}`** | `8080` | Deployment, Service | Port exposed by application process inside container. |
| **`{{ CPU_REQ }}` / `{{ CPU_LIM }}`** | `250m` / `1` | Deployment, Pod | Set requests for scheduler; limits enforce CFS throttling. |
| **`{{ MEM_REQ }}` / `{{ MEM_LIM }}`** | `512Mi` / `1Gi` | Deployment, Pod | Memory limit breach triggers immediate `OOMKilled` (Exit 137). |
| **`{{ STORAGE_SIZE }}`** | `50Gi` | PersistentVolumeClaim | Ensure StorageClass supports `allowVolumeExpansion: true`. |
| **`{{ CRON_SCHEDULE }}`** | `0 2 * * *` | CronJob, KEDA ScaledObject | Include explicit IANA `timeZone` in manifest. |

---

## ⚡ Instant One-Line Deployment

Pipe any manifest directly into `kubectl` via standard input:

```bash
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-api
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payment-api
  template:
    metadata:
      labels:
        app: payment-api
    spec:
      containers:
        - name: app
          image: nginx:alpine
          ports:
            - containerPort: 80
EOF
```

---

[← Return to Home](../index.md) | [Explore Step-by-Step Lessons →](../lessons/index.md)
