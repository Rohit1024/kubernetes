---
icon: lucide/award
---

# Lesson 0010: Capstone project: Deploying a multi-tier production stack

## Fast interview summary and architecture blueprint

| Tier | Workload type | Storage and networking | Availability and resilience |
| :--- | :--- | :--- | :--- |
| **Ingress / Gateway** | `Gateway` + `HTTPRoute` | Layer 7 Cloud Load Balancer (Port 80/443) | Managed SSL, URL Path Routing (`/`, `/api`) |
| **Frontend tier** | Stateless `Deployment` | `ClusterIP` Service + Container-Native NEG | `PodAntiAffinity`, `maxUnavailable: 0`, `preStop` hook |
| **Backend API tier** | Stateless `Deployment` | `ClusterIP` Service | `HPA` (CPU Autoscaling), Secret Injection, Probes |
| **Database tier** | Stateful `StatefulSet` | `Headless Service` (`clusterIP: None`) | `volumeClaimTemplates` (SSD RWO), `WaitForFirstConsumer` |
| **Cluster defense** | `PodDisruptionBudget` + `NetworkPolicy` | Namespace Zero-Trust Isolation | Prevents node drains from causing downtime; isolates DB access |

```mermaid
graph TD
    Client["External Client / Browser"] -->|HTTPS Traffic| GW["Google Cloud HTTP(S) Gateway / LB"]
    
    subgraph FrontendTier ["Tier 1: Web Frontend (Stateless)"]
        GW -->|Route: /| FeSvc["frontend-service (ClusterIP + NEG)"]
        FeSvc --> FePod1["frontend-pod-1 (Zone A)"]
        FeSvc --> FePod2["frontend-pod-2 (Zone B)"]
    end

    subgraph BackendTier ["Tier 2: REST API Backend (Stateless & Autoscaled)"]
        GW -->|Route: /api| BeSvc["api-service (ClusterIP)"]
        BeSvc --> BePod1["api-pod-1"]
        BeSvc --> BePod2["api-pod-2"]
        HPA["HPA (Target 70% CPU)"] -->|Autoscales 2-10 Pods| BackendTier
        Sec[("Secret: db-credentials")] -.->|Injected as Env| BackendTier
    end

    subgraph DatabaseTier ["Tier 3: PostgreSQL Database (Stateful)"]
        BePod1 & BePod2 -->|DNS: postgres-0.db-headless| Headless["db-headless (clusterIP: None)"]
        Headless --> DB["StatefulSet: postgres-0"]
        DB --> PVC[("PVC: data-postgres-0")]
        PVC --> PV[("GCP Persistent Disk SSD (WaitForFirstConsumer)")]
    end

    subgraph SecurityPolicy ["Zero-Trust Security & Governance"]
        NetPol["NetworkPolicy: Block direct frontend-to-database access"]
        PDB["PodDisruptionBudget: minAvailable=1"]
    end
```

---

## 1. Complete production manifest stack

Below is the complete, integrated production manifest stack demonstrating the core workload patterns:

```yaml
# ============================================================================
# 1. DATABASE TIER: Secret, Headless Service & StatefulSet
# ============================================================================
apiVersion: v1
kind: Secret
metadata:
  name: postgres-credentials
type: Opaque
data:
  POSTGRES_USER: cG9zdGdyZXM=       # postgres
  POSTGRES_PASSWORD: bXlwYXNzd29yZA== # mypassword
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-headless
spec:
  clusterIP: None                     # Headless Service for StatefulSet
  selector:
    app: postgres
  ports:
    - port: 5432
      name: postgres
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: "postgres-headless"
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:15-alpine
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
          resources:
            requests:
              cpu: "250m"
              memory: "512Mi"
            limits:
              cpu: "1"
              memory: "1Gi"
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: standard-rwo # GKE SSD with WaitForFirstConsumer
        resources:
          requests:
            storage: 20Gi
---
# ============================================================================
# 2. BACKEND API TIER: Deployment, Probes, HPA & Service
# ============================================================================
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-backend
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 0              # Zero downtime guarantee
  selector:
    matchLabels:
      app: api-backend
  template:
    metadata:
      labels:
        app: api-backend
    spec:
      containers:
        - name: api
          image: hashicorp/http-echo:latest
          args: ["-text", "API Gateway Response from Backend"]
          ports:
            - containerPort: 5678
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
          startupProbe:
            httpGet:
              path: /
              port: 5678
            failureThreshold: 10
            periodSeconds: 2
          readinessProbe:
            httpGet:
              path: /
              port: 5678
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /
              port: 5678
            periodSeconds: 10
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "sleep 10"] # In-flight connection drain
---
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  type: ClusterIP
  selector:
    app: api-backend
  ports:
    - port: 8080
      targetPort: 5678
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-backend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-backend
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
---
# ============================================================================
# 3. RELIABILITY: Pod Disruption Budget (PDB)
# ============================================================================
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: api-backend-pdb
spec:
  minAvailable: 1                     # Always guarantees at least 1 pod during node drains
  selector:
    matchLabels:
      app: api-backend
```

---

## Interview deep-dives and scenarios

??? question "Interview question: What is a Pod Disruption Budget (PDB), and why is it needed in production?"
    - **Voluntary disruptions:** Occur when an administrator or automated system drains a node (such as GKE node upgrades, Cluster Autoscaler scale-downs, or `kubectl drain`).
    - **Without a PDB:** A node drain can terminate all replicas of a service simultaneously, causing an unexpected outage.
    - **With a PDB (`minAvailable: 1` or `maxUnavailable: 1`):** The Kubernetes Eviction API checks the PDB before allowing the node to terminate a Pod. If evicting the Pod would violate the budget, the eviction is blocked until new replicas are healthy on other nodes.

??? question "Interview scenario: How do you prevent the frontend tier from communicating directly with the database tier?"
    Deploy a Kubernetes `NetworkPolicy` that enforces default-deny and allows ingress to the database only from Pods with label `app: api-backend`:
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: allow-only-api-to-database
    spec:
      podSelector:
        matchLabels:
          app: postgres
      policyTypes:
        - Ingress
      ingress:
        - from:
            - podSelector:
                matchLabels:
                  app: api-backend     # Only API pods allowed
          ports:
            - protocol: TCP
              port: 5432
    ```

---

## Common production pitfalls and interview traps

??? warning "Production trap: Setting `minAvailable: 100%` on a PDB"
    If you set `minAvailable: 100%` (or `minAvailable: 2` on a 2-replica deployment), `kubectl drain` and automated GKE node pool upgrades will hang indefinitely. Because all replicas must stay active, the eviction API refuses to evict any pod. Always leave headroom (such as `minAvailable: 1` or `maxUnavailable: 25%`).

---

## Module 1 review checklist

Key architecture concepts covered in Module 1:

- [x] **Control plane and node architecture:** `kube-apiserver`, `etcd` Raft quorum, `kubelet` PLEG, and CRI/CNI/CSI ([Lesson 1](0001-what-is-kubernetes-and-prerequisites.md)).
- [x] **Pod anatomy and lifecycles:** Pause containers, native sidecars, exit codes (137 OOMKilled vs 143 SIGTERM) ([Lesson 2](0002-pod-anatomy.md)).
- [x] **Scheduling and rollouts:** Node affinity, topology spread constraints, taints (`NoExecute`), and zero-downtime rollouts ([Lesson 3](0003-node-scheduling-deployment-strategies-autoscaling.md)).
- [x] **Service discovery and networking:** Virtual ClusterIPs, `IPVS` vs `iptables`, EndpointSlices, Headless Services, and CoreDNS `ndots` ([Lesson 4](0004-service-communication.md)).
- [x] **StatefulSets and secrets:** Ordinal identities, ConfigMap volume live updates, and GCP Secret Manager via ESO ([Lesson 5](0005-stateless-stateful-secrets-gcp.md)).
- [x] **Ingress and GKE load balancing:** Container-Native NEGs, edge TLS termination, `FrontendConfig`, and `BackendConfig` ([Lesson 6](0006-ingress-gke-load-balancing.md)).
- [x] **Storage architecture:** `StorageClasses`, `WaitForFirstConsumer` multi-zone binding, access modes, and volume expansion ([Lesson 7](0007-pv-pvc-storageclasses.md)).
- [x] **Gateway API:** Role-oriented design, canary traffic weighting, header routing, and `ReferenceGrant` multi-tenancy ([Lesson 8](0008-gke-gateway-api.md)).
- [x] **Probes and graceful shutdown:** QoS eviction classes (`Guaranteed`, `Burstable`, `BestEffort`), probe design, and `preStop` race conditions ([Lesson 9](0009-resources-probes-graceful-shutdown.md)).
- [x] **Multi-tier capstone integration:** PDBs, NetworkPolicies, and end-to-end production readiness ([Lesson 10](0010-capstone-project.md)).

---

## Test your knowledge

1. Why is setting `minAvailable: 1` in a `PodDisruptionBudget` recommended for production Deployments?
   - [ ] A) It ensures node drain and upgrade operations never terminate all replicas simultaneously
   - [ ] B) It forces the cluster autoscaler to maintain at least one node running in every zone
   
   Answer: A. PDBs instruct the Kubernetes eviction API to maintain service availability during voluntary disruptions like node upgrades.

2. In a multi-tier architecture, what Kubernetes resource prevents a compromised frontend container from executing queries directly against the backend PostgreSQL database?
   - [ ] A) A NetworkPolicy declaring ingress rules restricted to API backend pod selectors
   - [ ] B) A StorageClass configured with WaitForFirstConsumer volume binding mode
   
   Answer: A. NetworkPolicies implement network segmentation, enforcing that only authorized backend pods can connect to the database port.

---

## Recommended primary resources
- [Kubernetes production best practices](https://kubernetes.io/docs/setup/best-practices/)
- [Kubernetes Pod Disruption Budgets](https://kubernetes.io/docs/tasks/run-application/configure-pdb/)

---

[← Lesson 9: Resources, probes, and graceful shutdown](./0009-resources-probes-graceful-shutdown.md) | [Lesson 11: Helm package manager →](./0011-helm-package-manager.md)
