# Lesson 0003: Service-to-Service Communication & DNS Troubleshooting

## 1. How Cluster Services Communicate

In Kubernetes, Pods are ephemeral—they die and get recreated with new IP addresses. You cannot hardcode Pod IPs for communication.

Instead, we use  **Services** . A Service provides a single, stable IP address (the  **ClusterIP** ) and a DNS name that automatically load-balances traffic across matching healthy Pods.

### DNS Resolution Mechanics

Every Kubernetes cluster runs an internal DNS service (usually  **CoreDNS** ). When a Pod attempts to resolve a domain, CoreDNS answers with the ClusterIP of the target Service. The naming convention is:

`<service-name>.<namespace>.svc.cluster.local`

!!! note "Namespace Shortcut"
    If both Pods are in the **same namespace**, you can omit the suffix and simply connect using `http://<service-name>:<port>`.

### DNS & Service Routing Workflow

```mermaid
graph TD
    Client["Client Pod"] -->|1. DNS Query: backend-service| DNS["CoreDNS"]
    DNS -->|2. Returns ClusterIP: 10.96.0.45| Client
    Client -->|3. TCP request to 10.96.0.45| KP["kube-proxy / iptables"]
    KP -->|4. Load Balances (IP Translation)| Pod1["Backend Pod 1: 10.244.1.3"]
    KP -->|or 4.| Pod2["Backend Pod 2: 10.244.2.4"]
```

## 2. Code Sample: Frontend connecting to Backend

Here is how a frontend application connects to a backend database/API service internally.

### Backend Deployment & Service Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-api
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-server
  template:
    metadata:
      labels:
        app: api-server
    spec:
      containers:
      - name: api
        image: hashicorp/http-echo:latest
        args: ["-text", "Hello from the Backend API!"]
        ports:
        - containerPort: 5678

---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: default
spec:
  type: ClusterIP
  selector:
    app: api-server # Must match the Pod labels in the deployment
  ports:
  - port: 80           # The port the service exposes inside the cluster
    targetPort: 5678    # The port the container actually listens on
```

### Frontend Application Code Configuration

The frontend application container configuration would point to the backend service via environment variables:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-ui
  template:
    metadata:
      labels:
        app: web-ui
    spec:
      containers:
      - name: web
        image: curlimages/curl:latest
        command: ["/bin/sh", "-c"]
        args:
        - "while true; do curl -s http://backend-service:80; sleep 5; done"
```

## 3. Common Pitfalls to Avoid

!!! warning "1. Port vs. TargetPort Confusion"
    * `port`: The port numbers clients use to call the Service.
    * `targetPort`: The port the container application listens on (matches `containerPort`).

    If they do not align correctly, requests will result in connection timeouts or refused connections.

!!! warning "2. Incorrect Selectors"
    If the Service's `spec.selector` does not perfectly match the labels in the Pod's `template.metadata.labels`, the Service will have no target endpoints.

!!! warning "3. Pod Readiness"
    If a Pod has a `readinessProbe` configured and it fails, Kubernetes excludes that Pod from the Service's Endpoints pool. No traffic will reach it.

## 4. Step-by-Step Connectivity & DNS Troubleshooting Flow

When you get a timeout or "Host not found" error, execute these diagnostics in order:

### Step A: Verify Service Endpoints

Does the service actually point to any running Pods? Run:

```bash
kubectl get endpoints backend-service
```

If the `ENDPOINTS` column is empty or says `<none>`, check your selectors or Pod health statuses.

### Step B: Test DNS Resolution

Run a temporary container inside the cluster to check if the DNS system can resolve the service name:

```bash
kubectl run dns-test -it --image=radial/busyboxplus:curl --rm --restart=Never
```

Once inside the prompt, run:

```bash
nslookup backend-service
# Or if in a different namespace:
nslookup backend-service.default.svc.cluster.local
```

If DNS fails, CoreDNS is likely failing or overwhelmed. Check CoreDNS logs:

```bash
kubectl logs -n kube-system -l k8s-app=kube-dns
```

### Step C: Bypass Service and Test Direct Pod IP

Find one of the pod IPs directly:

```bash
kubectl get pods -o wide
```

Then, from your test container, try to curl that Pod IP directly: `curl http://<pod-ip>:5678`.
    If this works but the Service name does not, the issue lies in the Service configuration (e.g. port mismatch) or Kube-Proxy routing.

### Step D: Check Network Policies

If you get a connection timeout but DNS resolved successfully, check if there is an active `NetworkPolicy` blocking ingress traffic to your backend pods:

```bash
kubectl get networkpolicies -A
```

## Test Your Knowledge

### 1. If you configure a Service with `port: 8080` and `targetPort: 3000`, which port should client pods use when sending HTTP requests to the Service?

- [ ] **A.** Port 3000
- [ ] **B.** Port 8080

<details>
<summary><b>Answer & Explanation</b></summary>

**Correct Answer:** B

Correct! Clients talk to the service using the "port" value. The service then forwards traffic to the pods on "targetPort".
</details>

### 2. If 'kubectl get endpoints my-service' returns '', what is the most likely cause?

- [ ] **A.** The CoreDNS server is down.
- [ ] **B.** The service selector does not match the labels on any running pods.
- [ ] **C.** The client Pod is in a different namespace.

<details>
<summary><b>Answer & Explanation</b></summary>

**Correct Answer:** B

Correct! If there are no endpoints, it means the service has failed to locate any pods matching its selector labels.
</details>

## Hands-on Troubleshooting Challenge

Let's simulate a broken connection scenario in your cluster and fix it.

**Step 1:**  Apply the broken setup:

```yaml
# Save as broken-service.yaml and apply:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-backend
  template:
    metadata:
      labels:
        app: app-backend
    spec:
      containers:
      - name: echo
        image: hashicorp/http-echo
        args: ["-text", "Hello World"]
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: app-backend-wrong-label # Look closely!
  ports:
  - port: 80
    targetPort: 5678
```

**Step 2:**  Troubleshoot the deployment by checking endpoints:

`kubectl get endpoints app-service`

Observe that it has no targets. Fix the selector in the YAML file and re-apply it, then verify that the endpoint registers.

---

[← Lesson 2](./0002-node-scheduling-deployment-strategies-autoscaling.md) | [Lesson 4: Stateless vs. Stateful Workloads →](./0004-stateless-stateful-secrets-gcp.md)
