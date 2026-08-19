# KEDA autoscaling cheatsheet

Command reference and manifest blueprints for managing event-driven autoscaling in Kubernetes using **KEDA**, external metrics, scheduled cron triggers, and resolving GitOps drift with **Argo CD**.

---

## 1. Essential kubectl commands for KEDA

### Inspecting ScaledObjects and status
```bash
# List all ScaledObjects across all namespaces
kubectl get scaledobject -A

# Get detailed status, trigger evaluations, and recent events
kubectl describe scaledobject <SCALER_NAME> -n <NAMESPACE>

# Check generated HorizontalPodAutoscalers managed by KEDA
kubectl get hpa -A -l app.kubernetes.io/managed-by=keda-operator

# Check ScaledJobs and their active batch execution count
kubectl get scaledjob -A
kubectl describe scaledjob <JOB_SCALER_NAME> -n <NAMESPACE>

# Inspect TriggerAuthentication resources
kubectl get triggerauthentication,clustertriggerauthentication -A
```

### Checking KEDA controller health and metrics
```bash
# Verify KEDA operator and metrics API server pods
kubectl get pods -n keda

# View KEDA Operator logs in real-time
kubectl logs -n keda -l app.kubernetes.io/name=keda-operator -f

# View Metrics API Adapter logs
kubectl logs -n keda -l app.kubernetes.io/name=keda-operator-metrics-apiserver -f

# Query raw external metrics endpoints exposed to Kubernetes
kubectl get --raw "/apis/external.metrics.k8s.io/v1beta1" | jq .
```

---

## 2. Declarative CRD manifest blueprints

### A. Prometheus PromQL ScaledObject
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: http-traffic-scaler
  namespace: prod
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-api
  minReplicaCount: 1
  maxReplicaCount: 30
  pollingInterval: 15
  cooldownPeriod: 120
  triggers:
    - type: prometheus
      metadata:
        serverAddress: http://prometheus.monitoring.svc:9090
        metricName: http_requests_per_second
        query: sum(rate(http_requests_total{app="web-api"}[2m]))
        threshold: '100'              # 1 pod per 100 req/sec
        activationThreshold: '10'     # Activation threshold
```

### B. Timezone-aware Cron scaler (Pre-warming)
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: business-hours-scaler
  namespace: prod
spec:
  scaleTargetRef:
    name: backend-service
  minReplicaCount: 1
  maxReplicaCount: 20
  triggers:
    - type: cron
      metadata:
        timezone: America/New_York    # IANA timezone name
        start: 0 8 * * 1-5            # 8:00 AM Mon-Fri
        end: 0 18 * * 1-5             # 6:00 PM Mon-Fri
        desiredReplicas: '10'
```

### C. Multi-trigger composition (Cron, metrics, and fallback)
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: robust-multi-scaler
  namespace: prod
spec:
  scaleTargetRef:
    name: order-service
  minReplicaCount: 1
  maxReplicaCount: 40
  pollingInterval: 15
  cooldownPeriod: 180
  fallback:
    failureThreshold: 3
    replicas: 4                       # Safe default if metrics source fails
  triggers:
    - type: cron
      name: peak-prewarm
      metadata:
        timezone: UTC
        start: 0 6 * * *
        end: 0 20 * * *
        desiredReplicas: '8'
    - type: rabbitmq
      name: queue-surge
      metadata:
        queueName: orders
        host: http://guest:guest@rabbitmq.messaging.svc:15672
        queueLength: '25'
        activationQueueLength: '1'
```

### D. Discrete batch ScaledJob
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledJob
metadata:
  name: transcoder-job-scaler
  namespace: processing
spec:
  jobTargetRef:
    template:
      spec:
        restartPolicy: Never
        containers:
          - name: worker
            image: my-repo/transcoder:v1.0
  pollingInterval: 10
  maxReplicaCount: 50
  successfulJobsHistoryLimit: 5
  failedJobsHistoryLimit: 10
  scalingStrategy:
    strategy: accurate
    targetWorkload: '1'               # 1 Job per 1 message
  triggers:
    - type: aws-sqs-queue
      metadata:
        queueURL: https://sqs.us-east-1.amazonaws.com/123456789012/tasks
        queueLength: '1'
        awsRegion: us-east-1
      authenticationRef:
        name: keda-aws-auth
```

---

## 3. Argo CD and KEDA drift solutions

### Prevent replicas flapping in Argo CD Application
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: payments-app
  namespace: argocd
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
    - group: argoproj.io
      kind: Rollout
      jsonPointers:
        - /spec/replicas
```

### Global system configuration (argocd-cm ConfigMap)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  resource.customizations.ignoreDifferences.apps_Deployment: |
    jsonPointers:
      - /spec/replicas
```

---

## 4. Troubleshooting matrix

| Symptom | Root cause | Solution |
| :--- | :--- | :--- |
| **Workload stuck at 0 replicas** | `activationThreshold` not reached or trigger metric returning 0. | Run `kubectl describe scaledobject` and inspect trigger query. |
| **Argo CD constantly reverting replicas** | Argo CD `selfHeal: true` is overwriting dynamic scaling without `ignoreDifferences`. | Add `jsonPointers: [/spec/replicas]` to the Argo CD Application. |
| **Duplicate HPAs fighting over workload** | Manual `HorizontalPodAutoscaler` committed in Git alongside `ScaledObject`. | Remove manual HPA manifest from Git; let KEDA manage the HPA. |
| **`ScaledObject` shows `Ready: False`** | Authentication failure or unreachable metric server address. | Verify `TriggerAuthentication` credentials and network connectivity. |
| **Cron scaler triggering at wrong time** | Missing or incorrect IANA timezone string in metadata. | Set explicit `timezone: <IANA_NAME>` (e.g. `America/New_York`). |

---

[← Cheatsheets overview](./index.md)
