# Lesson 0024: Workload scaling with external metric triggers

## 1. What are KEDA scalers?

A **KEDA Scaler** is an integration module that queries external metrics (message counts in a queue, query statistics in a database, request rates in Prometheus) and translates those values into scaling targets for Kubernetes workloads.

KEDA provides **over 60 built-in scalers** across cloud services, databases, messaging brokers, and monitoring platforms:

```mermaid
graph TD
    subgraph Monitoring["Monitoring & Telemetry"]
        Prom["Prometheus PromQL"]
        Datadog["Datadog"]
        NewRelic["New Relic"]
    end

    subgraph Messaging["Message Queues & Streaming"]
        Kafka["Apache Kafka (Lag)"]
        RabbitMQ["RabbitMQ (Queue Length)"]
        SQS["AWS SQS"]
        PubSub["Google Cloud Pub/Sub"]
        Redis["Redis (Streams & Lists)"]
    end

    subgraph Databases["Databases & Storage"]
        Postgres["PostgreSQL Query"]
        MySQL["MySQL"]
        Elastic["Elasticsearch"]
    end

    Monitoring --> Scalers["KEDA Scalers Engine"]
    Messaging --> Scalers
    Databases --> Scalers
    Scalers --> K8s["Kubernetes Workload Scaler (0 ↔ N)"]
```

---

## 2. Scaling with Prometheus PromQL

The **Prometheus scaler** allows evaluating arbitrary PromQL queries to drive workload scale.

### Example: Scaling on HTTP request rate per Pod
Suppose an API service targets an average of **100 requests per second (RPS)** per pod. When total traffic reaches 500 RPS, KEDA scales the deployment to 5 Pods:

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: api-service-scaler
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-service
  minReplicaCount: 1                  # Keep at least 1 pod active
  maxReplicaCount: 20                 # Burst up to 20 pods
  pollingInterval: 15                 # Poll Prometheus every 15s
  cooldownPeriod: 120                 # Wait 2 minutes before scaling down
  triggers:
    - type: prometheus
      metadata:
        serverAddress: http://prometheus-operated.monitoring.svc:9090
        metricName: http_requests_per_second
        # PromQL query returning global requests/second
        query: sum(rate(http_requests_total{app="api-service"}[2m]))
        # Target threshold: 1 pod per 100 req/sec
        threshold: '100'
        # Activation threshold: scale from 0 to 1 if RPS exceeds 5
        activationThreshold: '5'
```

### Formula for replica calculation
The HPA calculates desired replicas using this formula:
$$\text{Desired Replicas} = \left\lceil \frac{\text{Current Metric Value}}{\text{Threshold}} \right\rceil$$

If `sum(rate(...))` equals `450`, Desired Replicas = $\lceil 450 / 100 \rceil = 5$ pods.

---

## 3. Scaling with message queues (RabbitMQ, Kafka, AWS SQS)

Message-driven workloads scale based on **queue backlog or consumer lag** rather than CPU utilization. When messages accumulate, CPU usage may remain steady while processing latency increases.

### Example: RabbitMQ queue scaler
Scales worker pods based on the count of unacknowledged messages:

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: rabbitmq-worker-scaler
  namespace: workers
spec:
  scaleTargetRef:
    name: order-consumer
  minReplicaCount: 0                  # Scale to zero when queue is empty
  maxReplicaCount: 50                 # Scale up to 50 workers under heavy load
  pollingInterval: 10                 # Poll every 10 seconds
  cooldownPeriod: 180                 # 3-minute cooldown before scaling to zero
  triggers:
    - type: rabbitmq
      metadata:
        queueName: orders-queue
        host: http://guest:guest@rabbitmq.messaging.svc:15672
        queueLength: '20'             # 1 worker pod for every 20 messages in queue
        activationQueueLength: '1'    # 1 message activates the first worker (0 → 1)
```

### Example: Kafka consumer group lag scaler
Scales workers based on partition consumer lag:

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: kafka-consumer-scaler
  namespace: streaming
spec:
  scaleTargetRef:
    name: event-stream-processor
  minReplicaCount: 2                  # Maintain 2 warm pods for instant ingestion
  maxReplicaCount: 24                 # Match the number of Kafka topic partitions
  triggers:
    - type: kafka
      metadata:
        bootstrapServers: kafka-broker.streaming.svc:9092
        consumerGroup: event-processor-group
        topic: transaction-events
        lagThreshold: '50'            # Scale 1 pod per 50 uncommitted partition messages
        activationLagThreshold: '5'
```

---

## 4. Production resilience: Fallback and stabilization windows

Metric services can experience network blips or restarts. KEDA includes controls to keep autoscaling stable during metric disruptions.

### 1. Fallback configuration
If KEDA fails to query the metric endpoint, `fallback` maintains a safe baseline replica count:

```yaml
spec:
  fallback:
    failureThreshold: 3               # If metric fetch fails 3 times consecutively
    replicas: 5                       # Force replicas to 5 until metric source recovers
```

### 2. Stabilization windows (Preventing flapping)
To prevent rapid scaling oscillation, configure custom HPA behavior inside `advanced.horizontalPodAutoscalerConfig`:

```yaml
spec:
  advanced:
    horizontalPodAutoscalerConfig:
      behavior:
        scaleDown:
          stabilizationWindowSeconds: 300  # Wait 5 min of sustained low traffic before reducing pods
          policies:
            - type: Percent
              value: 20                    # Scale down at most 20% of pods per minute
              periodSeconds: 60
        scaleUp:
          stabilizationWindowSeconds: 0    # Scale up immediately without delay
          policies:
            - type: Percent
              value: 100                   # Double capacity if needed
              periodSeconds: 15
```

---

## Test your knowledge

1. What is the difference between `threshold` and `activationThreshold` in a KEDA trigger?
   - [ ] A) Activation triggers 0 to 1 scaling while threshold calculates target pod counts
   - [ ] B) Activation triggers pod deletion while threshold initiates rolling deployments
   
   Answer: A. `activationThreshold` is used by the KEDA Operator for `0 → 1` activation, while `threshold` is used by HPA for `1 → N` target calculations.

2. If your Prometheus server becomes temporarily unreachable, what KEDA feature prevents your application from scaling down to zero?
   - [ ] A) The fallback configuration block
   - [ ] B) The cluster daemonset controller
   
   Answer: A. The `fallback` configuration specifies a safe default replica count when consecutive metric queries fail.

---

## Hands-on practice: Triggering and validating an event scale-up

### Step 1: Check ScaledObject metrics and status
```bash
# Get real-time status and active replica count
kubectl get scaledobject rabbitmq-worker-scaler -n workers
```

### Step 2: Query the synthetic KEDA metrics API
```bash
# Query the raw External Metrics endpoint
kubectl get --raw "/apis/external.metrics.k8s.io/v1beta1" | jq .

# Inspect the exact metric value currently reported
kubectl get --raw "/apis/external.metrics.k8s.io/v1beta1/namespaces/workers/s0-rabbitmq-orders-queue" | jq .
```

---

## Recommended primary resources
- [KEDA external scalers directory](https://keda.sh/docs/latest/scalers/)
- [KEDA Prometheus scaler](https://keda.sh/docs/latest/scalers/prometheus/)

---

[← Lesson 23: KEDA fundamentals and autoscaling architecture](./0023-keda-fundamentals-and-architecture.md) | [Lesson 25: Scheduled autoscaling with Cron scalers and multi-trigger composition →](./0025-keda-cron-and-scheduled-scaling.md)
