# Lesson 24: Scaling Workloads with KEDA External Metric Triggers

## 1. What are KEDA Scalers?

A **KEDA Scaler** is a dedicated integration module that connects to an external system, queries real-time metrics (e.g., message count in a queue, CPU load on a database, request rate in Prometheus), and translates those values into scaling signals for Kubernetes.

KEDA comes with **over 60+ built-in scalers** supporting all major cloud providers, databases, streaming systems, and monitoring platforms:

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

The **Prometheus scaler** is one of the most flexible scalers because any metric collected in your cluster can be evaluated using PromQL.

### Example: Scaling on HTTP Request Rate per Pod
Suppose you have an API service and you want each Pod to handle an average of **100 requests per second (RPS)**. When the global rate reaches 500 RPS, KEDA should scale the deployment to 5 Pods.

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

### Formula for Replica Calculation
The HPA calculates desired replicas using the standard equation:
$$\text{Desired Replicas} = \left\lceil \frac{\text{Current Metric Value}}{\text{Threshold}} \right\rceil$$

If `sum(rate(...))` = `450`, Desired Replicas = $\lceil 450 / 100 \rceil = 5$ pods.

---

## 3. Scaling with Message Queues (RabbitMQ, Kafka, AWS SQS)

Message-driven workloads need to scale based on **backlog (queue depth or consumer lag)** rather than CPU. If messages are sitting in a queue unprocessed, CPU utilization might remain low while latency spikes.

### Example: RabbitMQ Queue Scaler
Scale worker pods based on the number of messages waiting in a RabbitMQ queue:

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: rabbitmq-worker-scaler
  namespace: workers
spec:
  scaleTargetRef:
    name: order-consumer
  minReplicaCount: 0                  # Scale to ZERO when queue is empty!
  maxReplicaCount: 50                 # Scale up to 50 workers under heavy load
  pollingInterval: 10                 # Frequent polling for rapid responsiveness
  cooldownPeriod: 180                 # 3-minute grace period before scaling to zero
  triggers:
    - type: rabbitmq
      metadata:
        queueName: orders-queue
        host: http://guest:guest@rabbitmq.messaging.svc:15672
        queueLength: '20'             # 1 worker pod for every 20 messages in queue
        activationQueueLength: '1'    # 1 message will activate the first worker (0 → 1)
```

### Example: Kafka Consumer Group Lag Scaler
For Kafka, scaling is based on **consumer lag** across partitions:

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

## 4. Production Resilience: Fallback & Stabilization Windows

In enterprise production environments, metric systems can experience transient network partitions or outages. KEDA provides built-in mechanisms to protect against autoscaling instability.

### 1. Fallback Configuration
If KEDA fails to communicate with the metric source (e.g., Prometheus is restarting or credentials expired), the `fallback` block prevents catastrophic scale-down by maintaining a safe baseline:

```yaml
spec:
  fallback:
    failureThreshold: 3               # If metric fetch fails 3 times consecutively
    replicas: 5                       # Force replicas to 5 until metric source recovers
```

### 2. Stabilization Windows (Preventing Flapping)
To prevent "thrashing" (rapidly scaling up and down in seconds), configure custom HPA behavior inside `advanced.horizontalPodAutoscalerConfig`:

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

## Test Your Knowledge

1. What is the difference between `threshold` and `activationThreshold` in a KEDA trigger?
   - [ ] A) Activation triggers 0 to 1 scaling while threshold calculates target pod counts
   - [ ] B) Activation triggers pod deletion while threshold initiates rolling deployments
   
   *Answer:* A) Activation triggers 0 to 1 scaling while threshold calculates target pod counts - Correct! `activationThreshold` is used by the KEDA Operator for `0 → 1` activation, while `threshold` is passed to the HPA for `1 → N` target calculations.

2. If your Prometheus server becomes temporarily unreachable, what KEDA feature ensures your application does not scale down to zero?
   - [ ] A) The fallback configuration block
   - [ ] B) The cluster daemonset controller
   
   *Answer:* A) The fallback configuration block - Correct! The `fallback` configuration specifies a safe default replica count whenever consecutive metric queries fail.

---

## Interactive Win: Triggering & Validating an Event Scale-Up

Let's simulate inspecting external metric evaluation on a running ScaledObject.

### Step 1: Check ScaledObject Metrics & Status
```bash
# Get real-time status, active state, and current replica count
kubectl get scaledobject rabbitmq-worker-scaler -n workers

# Output should show:
# NAME                     TARGET            MIN   MAX   TRIGGERS   AUTHENTICATION   ACTIVE   READY   AGE
# rabbitmq-worker-scaler   order-consumer    0     50    rabbitmq                    True     True    5m
```

### Step 2: Query the Synthetic KEDA Metrics API
```bash
# Query the raw External Metrics endpoint served by keda-operator-metrics-apiserver
kubectl get --raw "/apis/external.metrics.k8s.io/v1beta1" | jq .

# Inspect the exact metric value currently being reported
kubectl get --raw "/apis/external.metrics.k8s.io/v1beta1/namespaces/workers/s0-rabbitmq-orders-queue" | jq .
```

---

## Recommended Primary Resource
- [KEDA External Scalers Directory](https://keda.sh/docs/latest/scalers/)
- [KEDA Prometheus Scaler Specification](https://keda.sh/docs/latest/scalers/prometheus/)

---
**Need help formulating a PromQL query or tuning Kafka consumer lag thresholds?** Ask in the chat, and we can test the calculations together!

[← Lesson 23: KEDA Fundamentals & Architecture](./0023-keda-fundamentals-and-architecture.md) | [Lesson 25: Time-Based & Cron Scaling →](./0025-keda-cron-and-scheduled-scaling.md)
