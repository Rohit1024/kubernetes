# Kubernetes complete cheatsheet

> **Purpose:** Reference guide for `kubectl` commands, flags, resource types, and grep patterns for day-to-day operations and interviews.

---

## Table of contents

1. [kubectl global flags](#kubectl-global-flags)
2. [Cluster and context management](#cluster-and-context-management)
3. [Viewing resources (Get / Describe / Explain)](#viewing-resources)
4. [Creating resources](#creating-resources)
5. [Editing and patching resources](#editing-and-patching-resources)
6. [Deleting resources](#deleting-resources)
7. [Pod operations](#pod-operations)
8. [Deployments and rollouts](#deployments-and-rollouts)
9. [Services and networking](#services-and-networking)
10. [ConfigMaps and Secrets](#configmaps-and-secrets)
11. [Namespaces](#namespaces)
12. [Nodes](#nodes)
13. [Persistent storage (PV / PVC / StorageClass)](#persistent-storage)
14. [Jobs and CronJobs](#jobs-and-cronjobs)
15. [DaemonSets and StatefulSets](#daemonsets-and-statefulsets)
16. [Horizontal Pod Autoscaler (HPA)](#horizontal-pod-autoscaler)
17. [RBAC (Roles, ClusterRoles, Bindings)](#rbac)
18. [Service Accounts](#service-accounts)
19. [Network Policies](#network-policies)
20. [Ingress](#ingress)
21. [Resource quotas and limit ranges](#resource-quotas-and-limit-ranges)
22. [Taints, tolerations, and affinity](#taints-tolerations-and-affinity)
23. [Labels, selectors, and annotations](#labels-selectors-and-annotations)
24. [Debugging and troubleshooting](#debugging-and-troubleshooting)
25. [Output formatting and JSONPath](#output-formatting-and-jsonpath)
26. [kubectl plugins and Krew](#kubectl-plugins-and-krew)
27. [Helm package manager](#helm-package-manager)
28. [All resource types quick reference](#all-resource-types-quick-reference)
29. [Grep combos with kubectl](#grep-combos-with-kubectl)
30. [One-liners and power tricks](#one-liners-and-power-tricks)
31. [Interview quick-fire questions](#interview-quick-fire-questions)

---

## kubectl global flags

These flags can be used with any `kubectl` command.

| Flag | Short | Description |
|------|-------|-------------|
| `--kubeconfig` | | Path to the kubeconfig file (default `~/.kube/config`) |
| `--context` | | Name of the kubeconfig context to use |
| `--cluster` | | Name of the kubeconfig cluster to use |
| `--user` | | Name of the kubeconfig user to use |
| `--namespace` | `-n` | Kubernetes namespace scope for this request |
| `--all-namespaces` | `-A` | Scope to all namespaces |
| `--server` | `-s` | Address of the Kubernetes API server |
| `--certificate-authority` | | Path to CA cert file |
| `--client-certificate` | | Path to client certificate file |
| `--client-key` | | Path to client key file |
| `--token` | | Bearer token for authentication |
| `--as` | | Username to impersonate |
| `--as-group` | | Group to impersonate |
| `--as-uid` | | UID to impersonate |
| `--output` | `-o` | Output format: `json`, `yaml`, `wide`, `name`, `jsonpath=...`, `custom-columns=...` |
| `--dry-run` | | `none`, `client`, `server`: simulate without persisting |
| `--v` | | Verbosity level (0 to 9) |
| `--warnings-as-errors` | | Treat warnings as errors |
| `--request-timeout` | | Timeout for a single request (default `0` = no timeout) |
| `--cache-dir` | | Default cache directory |
| `--insecure-skip-tls-verify` | | Skip server certificate verification |
| `--tls-server-name` | | Server name for TLS certificate validation |
| `--field-manager` | | Name of the manager used for server-side apply |

```bash
# Examples
kubectl -n production get pods                    # Specific namespace
kubectl -A get pods                               # All namespaces
kubectl --context=staging get svc                 # Specific context
kubectl --kubeconfig=/path/to/config get nodes    # Custom kubeconfig
kubectl -v=6 get pods                             # Verbose (shows HTTP requests)
kubectl -v=9 get pods                             # Maximum verbosity (request/response bodies)
```

---

## Cluster and context management

### `kubectl config`: Manage kubeconfig

| Command | Description |
|---------|-------------|
| `kubectl config view` | Display merged kubeconfig |
| `kubectl config current-context` | Display current context |
| `kubectl config use-context NAME` | Switch context |
| `kubectl config set-context NAME` | Set a context entry |
| `kubectl config delete-context NAME` | Delete a context |
| `kubectl config get-contexts` | List all contexts |
| `kubectl config get-clusters` | List all clusters |
| `kubectl config get-users` | List all users |
| `kubectl config set-cluster NAME` | Set a cluster entry |
| `kubectl config set-credentials NAME` | Set a user entry |
| `kubectl config rename-context OLD NEW` | Rename a context |
| `kubectl config unset PROPERTY` | Unset a kubeconfig property |

```bash
kubectl config view                                    # Show full kubeconfig
kubectl config view --minify                           # Show only current context config
kubectl config view -o jsonpath='{.users[*].name}'     # List user names
kubectl config current-context                          # Current context name
kubectl config use-context production                   # Switch context
kubectl config get-contexts                             # List all contexts
kubectl config set-context --current --namespace=dev    # Set default namespace for current context
kubectl config set-context myctx --cluster=mycluster --user=myuser --namespace=default
kubectl config delete-context old-context
```

### `kubectl cluster-info`

```bash
kubectl cluster-info                       # Display cluster master and services URLs
kubectl cluster-info dump                  # Dump cluster state for debugging
kubectl cluster-info dump --output-directory=/tmp/cluster-dump
```

### `kubectl api-resources` and `kubectl api-versions`

```bash
kubectl api-resources                                # All resource types
kubectl api-resources --namespaced=true              # Namespaced resources
kubectl api-resources --namespaced=false             # Cluster-scoped resources
kubectl api-resources --verbs=list,get               # Resources that support list and get
kubectl api-resources -o wide                        # Extra info (verbs, short names)
kubectl api-resources --api-group=apps               # Apps API group resources
kubectl api-versions                                 # Registered API versions
```

---

## Viewing resources

### `kubectl get`: List resources

| Flag | Short | Description |
|------|-------|-------------|
| `--output` | `-o` | Output format: `wide`, `yaml`, `json`, `name`, `jsonpath=...`, `custom-columns=...` |
| `--selector` | `-l` | Filter by label selector |
| `--field-selector` | | Filter by field (e.g., `status.phase=Running`) |
| `--all-namespaces` | `-A` | All namespaces |
| `--watch` | `-w` | Watch for changes |
| `--show-labels` | | Show labels column |
| `--sort-by` | | Sort by JSONPath expression |
| `--no-headers` | | Omit table headers |
| `--label-columns` | `-L` | Show specific label values as columns |

```bash
kubectl get pods                                          # Current namespace
kubectl get pods -A                                       # All namespaces
kubectl get pods -o wide                                  # Extra columns (Node, IP)
kubectl get pods -o yaml                                  # YAML output
kubectl get pods --show-labels                            # Show labels
kubectl get pods -l app=web                               # Filter by label
kubectl get pods -l 'app in (web,api)'                    # Set selector
kubectl get pods --field-selector status.phase=Running    # Field selector
kubectl get pods --sort-by=.metadata.creationTimestamp    # Sort by creation
kubectl get pods -w                                       # Watch
kubectl get pods -L app,version                           # Label columns
kubectl get all -A                                        # Common resources across all namespaces
```

### `kubectl describe`: Detailed resource info

```bash
kubectl describe pod mypod
kubectl describe node mynode
kubectl describe svc myservice
kubectl describe deploy mydeployment
kubectl describe pv mypv
kubectl describe ingress myingress
```

### `kubectl explain`: API documentation

```bash
kubectl explain pod                          # Top-level Pod spec
kubectl explain pod.spec                     # Pod spec fields
kubectl explain pod.spec.containers          # Container spec
kubectl explain pod.spec.containers.resources
kubectl explain deployment.spec.strategy     # Deployment strategy fields
kubectl explain pod --recursive              # Full recursive field tree
```

---

## Creating resources

### `kubectl apply`: Declarative management

```bash
kubectl apply -f pod.yaml
kubectl apply -f ./manifests/                        # All files in directory
kubectl apply -f ./manifests/ -R                     # Recursive directory
kubectl apply -f pod.yaml --dry-run=client           # Client dry run
kubectl apply -f pod.yaml --dry-run=server           # Server dry run
kubectl apply -k ./kustomize/                        # Kustomize overlay
kubectl diff -f pod.yaml                             # Preview differences
```

### `kubectl create`: Imperative creation

```bash
kubectl create namespace staging
kubectl create deployment web --image=nginx --replicas=3 --port=80
kubectl create service clusterip web --tcp=80:80
kubectl create configmap myconfig --from-literal=key1=val1 --from-file=config.yaml
kubectl create secret generic mysecret --from-literal=password=s3cr3t
kubectl create secret tls tls-secret --cert=cert.pem --key=key.pem
kubectl create job myjob --image=busybox -- echo "hello"
kubectl create cronjob mycron --image=busybox --schedule="*/5 * * * *" -- echo "tick"
kubectl create serviceaccount mysa
kubectl create role pod-reader --verb=get,list,watch --resource=pods
kubectl create rolebinding pod-reader-binding --role=pod-reader --serviceaccount=default:mysa
kubectl create ingress myingress --rule="example.com/=web:80" --class=nginx

# Generate YAML without creating
kubectl create deployment web --image=nginx --dry-run=client -o yaml > deployment.yaml
kubectl create service clusterip web --tcp=80:80 --dry-run=client -o yaml > service.yaml
```

### `kubectl run`: Quick Pod creation

```bash
kubectl run nginx --image=nginx                        # Quick pod
kubectl run nginx --image=nginx --port=80              # With port
kubectl run busybox --image=busybox -it --rm -- sh     # Ephemeral shell
kubectl run test --image=curlimages/curl -it --rm -- curl http://myservice
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

---

## Editing and patching resources

```bash
# In-place edit
kubectl edit deployment mydeployment
kubectl edit svc myservice

# Strategic merge patch
kubectl patch deployment web -p '{"spec":{"replicas":5}}'

# Merge patch
kubectl patch deployment web --type=merge -p '{"spec":{"replicas":5}}'

# JSON patch
kubectl patch deployment web --type=json -p '[{"op":"replace","path":"/spec/replicas","value":5}]'

# Imperative field setters
kubectl set image deployment/web nginx=nginx:1.25
kubectl set env deployment/web DB_HOST=newdb
kubectl set resources deployment/web -c=nginx --requests=cpu=100m,memory=256Mi --limits=cpu=500m,memory=512Mi
```

---

## Deleting resources

```bash
kubectl delete pod mypod
kubectl delete pod mypod --now                         # Immediate termination (grace period = 1)
kubectl delete pod mypod --force --grace-period=0      # Force kill stuck pod
kubectl delete -f deployment.yaml                      # Delete from manifest
kubectl delete pods -l app=web                         # Delete by label
kubectl delete pods --all                              # Delete all pods in namespace
kubectl delete pods --field-selector status.phase=Failed
```

---

## Pod operations

### Logs

```bash
kubectl logs mypod
kubectl logs mypod -f                              # Follow stream
kubectl logs mypod --tail=100                      # Last 100 lines
kubectl logs mypod --since=1h                      # Last hour
kubectl logs mypod -c mycontainer                  # Specific container
kubectl logs mypod --all-containers                # All containers
kubectl logs mypod -p                              # Previous (crashed) instance
kubectl logs mypod --timestamps                    # Timestamps
kubectl logs -l app=web                            # By label selector
```

### Exec and port-forward

```bash
# Exec
kubectl exec -it mypod -- bash                     # Interactive bash
kubectl exec -it mypod -- sh                       # Interactive sh
kubectl exec mypod -- env                           # Run command

# Port Forward
kubectl port-forward pod/mypod 8080:80                # Pod port
kubectl port-forward svc/myservice 8080:80             # Service port
kubectl port-forward deploy/mydeployment 8080:80       # Deployment port

# Copy files
kubectl cp mypod:/app/log.txt ./log.txt            # Pod to host
kubectl cp ./config.yml mypod:/app/config.yml       # Host to pod
```

---

## Deployments and rollouts

```bash
# Rollout status & history
kubectl rollout status deployment/web
kubectl rollout history deployment/web
kubectl rollout history deployment/web --revision=3

# Rollback
kubectl rollout undo deployment/web                        # Previous revision
kubectl rollout undo deployment/web --to-revision=2        # Specific revision

# Pause & Resume
kubectl rollout pause deployment/web
kubectl rollout resume deployment/web

# Rolling restart
kubectl rollout restart deployment/web
kubectl rollout restart daemonset/myds

# Scale
kubectl scale deployment web --replicas=5
kubectl scale deployment web --replicas=0
```

---

## Services and networking

| Type | Description |
|------|-------------|
| `ClusterIP` | Internal cluster IP (default). |
| `NodePort` | Exposes on each node IP at a static port (30000 to 32767). |
| `LoadBalancer` | Provisions cloud provider load balancer. |
| `ExternalName` | Maps service to external DNS CNAME. |
| `Headless` | `ClusterIP: None`. Direct DNS resolution to individual Pod IPs. |

```bash
kubectl get svc
kubectl get endpoints myservice
kubectl describe svc myservice
```

---

## ConfigMaps and Secrets

```bash
# ConfigMaps
kubectl create configmap myconfig --from-literal=key1=val1 --from-file=config.properties
kubectl get configmap myconfig -o yaml

# Secrets
kubectl create secret generic mysecret --from-literal=pass=s3cr3t
kubectl create secret tls tls-secret --cert=tls.crt --key=tls.key
kubectl get secret mysecret -o jsonpath='{.data.pass}' | base64 -d
```

---

## Namespaces

```bash
kubectl get ns
kubectl create namespace staging
kubectl delete namespace staging
kubectl config set-context --current --namespace=staging
```

---

## Nodes

```bash
kubectl get nodes -o wide
kubectl describe node mynode
kubectl top node

# Cordon / Drain
kubectl cordon mynode
kubectl uncordon mynode
kubectl drain mynode --ignore-daemonsets --delete-emptydir-data
```

---

## Persistent storage

```bash
kubectl get pv
kubectl get pvc
kubectl get sc
kubectl describe pvc mypvc
```

| Access mode | Abbreviation | Description |
| :--- | :--- | :--- |
| `ReadWriteOnce` | RWO | Single node read-write |
| `ReadOnlyMany` | ROX | Multiple nodes read-only |
| `ReadWriteMany` | RWX | Multiple nodes read-write |
| `ReadWriteOncePod` | RWOP | Single Pod read-write |

---

## Jobs and CronJobs

```bash
# One-off Job
kubectl create job myjob --image=busybox -- echo "completed"
kubectl get jobs
kubectl logs job/myjob

# CronJob
kubectl create cronjob mycron --image=busybox --schedule="0 * * * *" -- echo "hourly"
kubectl get cronjobs
```

---

## DaemonSets and StatefulSets

```bash
# DaemonSet
kubectl get ds
kubectl rollout restart ds/fluentd

# StatefulSet
kubectl get sts
kubectl scale sts/mysql --replicas=3
```

---

## Horizontal Pod Autoscaler

```bash
kubectl autoscale deployment web --min=2 --max=10 --cpu-percent=80
kubectl get hpa
kubectl describe hpa web
```

---

## RBAC

```bash
# Roles and Bindings
kubectl create role pod-reader --verb=get,list,watch --resource=pods
kubectl create rolebinding pod-reader-binding --role=pod-reader --serviceaccount=default:mysa

# ClusterRoles and ClusterRoleBindings
kubectl create clusterrole node-reader --verb=get,list --resource=nodes
kubectl create clusterrolebinding cluster-view --clusterrole=view --group=developers

# Check authorization
kubectl auth can-i create pods
kubectl auth can-i create pods --as=jane
kubectl auth can-i list secrets --namespace=production
```

---

## Service Accounts

```bash
kubectl create serviceaccount mysa
kubectl get sa
kubectl create token mysa --duration=24h
kubectl set serviceaccount deployment/web mysa
```

---

## Network Policies

```bash
kubectl get netpol -A
kubectl describe netpol mypolicy
```

---

## Ingress

```bash
kubectl get ing -A
kubectl describe ingress myingress
kubectl create ingress myingress --rule="example.com/=web:80" --class=nginx
```

---

## Resource quotas and limit ranges

```bash
kubectl create quota myquota --hard=cpu=4,memory=8Gi,pods=20
kubectl get quota
kubectl get limitranges
```

---

## Taints, tolerations, and affinity

```bash
# Add taint
kubectl taint nodes mynode key=value:NoSchedule
kubectl taint nodes mynode key=value:NoExecute

# Remove taint
kubectl taint nodes mynode key:NoSchedule-
```

---

## Labels, selectors, and annotations

```bash
# Labels
kubectl label pod mypod app=web --overwrite
kubectl label pod mypod app-
kubectl get pods -l app=web

# Annotations
kubectl annotate pod mypod description="Production service" --overwrite
kubectl annotate pod mypod description-
```

---

## Debugging and troubleshooting

```bash
# Events
kubectl get events --sort-by='.lastTimestamp'
kubectl get events --field-selector type=Warning

# Resource usage
kubectl top pods
kubectl top nodes

# Ephemeral debug container
kubectl debug mypod -it --image=busybox
```

---

## Output formatting and JSONPath

```bash
# Extract pod IPs
kubectl get pods -o jsonpath='{.items[*].status.podIP}'

# Pod names with restart counts
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.containerStatuses[0].restartCount}{"\n"}{end}'

# Custom columns
kubectl get pods -o custom-columns='NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName,IP:.status.podIP'
```

---

## kubectl plugins and Krew

```bash
kubectl krew search
kubectl krew install ctx
kubectl krew install ns
kubectl krew install neat
```

---

## Helm package manager

```bash
# Repositories
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Install, Upgrade, Rollback
helm install myrelease bitnami/nginx -f values.yaml
helm upgrade myrelease bitnami/nginx --set replicaCount=5
helm rollback myrelease 1

# List & Status
helm list -A
helm status myrelease
helm uninstall myrelease
```

---

## All resource types quick reference

| Resource | Short name | API group | Namespaced |
| :--- | :--- | :--- | :--- |
| Pod | `po` | `v1` | Yes |
| Deployment | `deploy` | `apps/v1` | Yes |
| StatefulSet | `sts` | `apps/v1` | Yes |
| DaemonSet | `ds` | `apps/v1` | Yes |
| Service | `svc` | `v1` | Yes |
| Ingress | `ing` | `networking.k8s.io/v1` | Yes |
| ConfigMap | `cm` | `v1` | Yes |
| Secret | `secret` | `v1` | Yes |
| PersistentVolumeClaim | `pvc` | `v1` | Yes |
| PersistentVolume | `pv` | `v1` | No |
| Node | `no` | `v1` | No |
| Namespace | `ns` | `v1` | No |

---

## Grep combos with kubectl

```bash
# Find pods by status
kubectl get pods | grep "CrashLoopBackOff"
kubectl get pods -A | grep "ImagePullBackOff"

# Count pods by status
kubectl get pods | grep -c "Running"

# Filter logs for errors
kubectl logs mypod | grep -i "error"
```

---

## One-liners and power tricks

```bash
# Print all pod IPs
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.podIP}{"\n"}{end}'

# Delete all failed or evicted pods
kubectl delete pods --field-selector status.phase=Failed -A

# Test DNS resolution
kubectl run dns --image=busybox -it --rm -- nslookup kubernetes.default.svc.cluster.local
```

---

## Interview quick-fire questions

| # | Question | Answer |
|---|----------|--------|
| 1 | **Pod vs Container?** | A Pod is the atomic scheduling unit in Kubernetes. It encapsulates one or more containers sharing network namespaces and storage volumes. |
| 2 | **ReplicaSet vs Deployment?** | ReplicaSet maintains pod counts. Deployments manage ReplicaSets, orchestrating declarative rollouts and rollbacks. |
| 3 | **StatefulSet vs Deployment?** | Deployments manage stateless, interchangeable pods. StatefulSets maintain stable network identities (`pod-0`, `pod-1`) and dedicated persistent storage per index. |
| 4 | **DaemonSet use case?** | Runs one pod per matching node for cluster-level background services: logging agents, node metric collectors, or CNI network plugins. |
| 5 | **Service types?** | `ClusterIP` (internal default), `NodePort` (node port exposure), `LoadBalancer` (cloud LB integration), `ExternalName` (CNAME redirect), `Headless` (returns pod IPs directly). |
| 6 | **Ingress vs Service?** | Services operate at Layer 4 (TCP/UDP). Ingress manages Layer 7 HTTP/HTTPS host and path routing and TLS termination through an Ingress controller. |
| 7 | **ConfigMap vs Secret?** | Both store key-value configuration. Secrets are base64-encoded and can integrate with RBAC and encryption at rest. |
| 8 | **Liveness vs Readiness vs Startup probes?** | Liveness restarts crashed containers. Readiness controls whether the pod receives Service traffic. Startup protects slow-initializing workloads from premature restarts. |
| 9 | **Resource Requests vs Limits?** | Requests determine node scheduling guarantees. Limits cap resource consumption: memory overages trigger OOMKills; CPU overages result in throttling. |
| 10 | **`kubectl apply` vs `kubectl create`?** | `create` is an imperative action that fails if the resource exists. `apply` computes declarative strategic merges, creating or updating resources as needed. |

---

[← Docker cheatsheet](./docker-cheatsheet.md) | [GitOps cheatsheet →](./gitops-argocd-flux-cheatsheet.md)
