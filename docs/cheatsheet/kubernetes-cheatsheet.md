# ☸️ Kubernetes Complete Cheatsheet

> **Purpose:** Quick-reference for all `kubectl` commands, flags, resource types, and `grep` combos — built for interview prep and daily use.

---

## Table of Contents

1. [kubectl Global Flags](#kubectl-global-flags)
2. [Cluster & Context Management](#cluster-context-management)
3. [Viewing Resources (Get / Describe / Explain)](#viewing-resources)
4. [Creating Resources](#creating-resources)
5. [Editing & Patching Resources](#editing-patching-resources)
6. [Deleting Resources](#deleting-resources)
7. [Pod Operations](#pod-operations)
8. [Deployments & Rollouts](#deployments-rollouts)
9. [Services & Networking](#services-networking)
10. [ConfigMaps & Secrets](#configmaps-secrets)
11. [Namespaces](#namespaces)
12. [Nodes](#nodes)
13. [Persistent Storage (PV / PVC / StorageClass)](#persistent-storage)
14. [Jobs & CronJobs](#jobs-cronjobs)
15. [DaemonSets & StatefulSets](#daemonsets-statefulsets)
16. [Horizontal Pod Autoscaler (HPA)](#horizontal-pod-autoscaler)
17. [RBAC (Roles, ClusterRoles, Bindings)](#rbac)
18. [Service Accounts](#service-accounts)
19. [Network Policies](#network-policies)
20. [Ingress](#ingress)
21. [Resource Quotas & Limit Ranges](#resource-quotas-limit-ranges)
22. [Taints, Tolerations & Affinity](#taints-tolerations-affinity)
23. [Labels, Selectors & Annotations](#labels-selectors-annotations)
24. [Debugging & Troubleshooting](#debugging-troubleshooting)
25. [Output Formatting & JSONPath](#output-formatting-jsonpath)
26. [kubectl Plugins & Krew](#kubectl-plugins-krew)
27. [Helm (Package Manager)](#helm-package-manager)
28. [All Resource Types Quick Reference](#all-resource-types-quick-reference)
29. [Grep Combos with kubectl](#grep-combos-with-kubectl)
30. [One-Liners & Power Tricks](#one-liners-power-tricks)
31. [Interview Quick-Fire Q&A](#interview-quick-fire-qa)

---

## kubectl Global Flags

These flags can be used with **any** `kubectl` command.

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
| `--output` | `-o` | Output format: `json`, `yaml`, `wide`, `name`, `jsonpath=...`, `go-template=...`, `custom-columns=...` |
| `--dry-run` | | `none`, `client`, `server` — simulate without persisting |
| `--v` | | Verbosity level (0–9) |
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

## Cluster & Context Management

### `kubectl config` — Manage kubeconfig

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
kubectl cluster-info                       # Display cluster master & services URLs
kubectl cluster-info dump                  # Dump cluster state for debugging
kubectl cluster-info dump --output-directory=/tmp/cluster-dump   # Dump to directory
```

### `kubectl version`

```bash
kubectl version                   # Client and server version
kubectl version --client          # Client version only
kubectl version -o json           # JSON output
kubectl version -o yaml           # YAML output
```

### `kubectl api-resources` & `kubectl api-versions`

| Command | Key Flags | Description |
|---------|-----------|-------------|
| `kubectl api-resources` | `--namespaced`, `--verbs`, `-o wide`, `--api-group` | List all resource types |
| `kubectl api-versions` | *(none)* | List all API versions |

```bash
kubectl api-resources                                # All resource types
kubectl api-resources --namespaced=true              # Only namespaced resources
kubectl api-resources --namespaced=false             # Only cluster-scoped resources
kubectl api-resources --verbs=list,get               # Resources that support list and get
kubectl api-resources -o wide                        # Show extra info (verbs, etc.)
kubectl api-resources --api-group=apps               # Only resources in apps group
kubectl api-versions                                 # All API group versions
```

---

## Viewing Resources

### `kubectl get` — List resources

| Flag | Short | Description |
|------|-------|-------------|
| `--output` | `-o` | Output format: `wide`, `yaml`, `json`, `name`, `jsonpath=...`, `custom-columns=...` |
| `--selector` | `-l` | Filter by label selector |
| `--field-selector` | | Filter by field (e.g., `status.phase=Running`) |
| `--all-namespaces` | `-A` | All namespaces |
| `--watch` | `-w` | Watch for changes |
| `--show-labels` | | Show labels column |
| `--sort-by` | | Sort by JSONPath expression (e.g., `.metadata.creationTimestamp`) |
| `--no-headers` | | Don't print headers |
| `--chunk-size` | | Return large lists in chunks |
| `--label-columns` | `-L` | Show specific label values as columns |
| `--show-kind` | | Show resource kind in NAME column |
| `--show-managed-fields` | | Show managed fields in YAML/JSON |

```bash
kubectl get pods                                          # Pods in current namespace
kubectl get pods -A                                       # All namespaces
kubectl get pods -o wide                                  # Extra info (Node, IP)
kubectl get pods -o yaml                                  # Full YAML
kubectl get pods -o json                                  # Full JSON
kubectl get pods -o name                                  # Just resource names
kubectl get pods --show-labels                            # Show all labels
kubectl get pods -l app=web                               # Filter by label
kubectl get pods -l 'app in (web,api)'                    # Set-based selector
kubectl get pods -l 'app!=db'                             # Negative selector
kubectl get pods --field-selector status.phase=Running    # Field selector
kubectl get pods --field-selector spec.nodeName=node1     # By node
kubectl get pods --sort-by=.metadata.creationTimestamp    # Sort by creation
kubectl get pods --sort-by=.status.startTime             # Sort by start time
kubectl get pods -w                                       # Watch for changes
kubectl get pods --no-headers                             # No headers (good for scripting)
kubectl get pods -L app,version                           # Show specific labels as columns
kubectl get all                                           # All common resources
kubectl get all -A                                        # Everything, all namespaces

# Custom columns
kubectl get pods -o custom-columns='NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName'

# Multiple resource types
kubectl get pods,svc,deploy

# Specific resource
kubectl get pod mypod
kubectl get pod mypod -o yaml
```

### `kubectl describe` — Detailed resource info

```bash
kubectl describe pod mypod
kubectl describe node mynode
kubectl describe svc myservice
kubectl describe deploy mydeployment
kubectl describe pv mypv
kubectl describe ns production
kubectl describe ingress myingress

# Describe all pods matching a label
kubectl describe pods -l app=web
```

> [!TIP]
> `describe` shows **Events** at the bottom — this is often the most useful part for debugging.

### `kubectl explain` — API documentation

| Flag | Description |
|------|-------------|
| `--recursive` | Print the fields tree |
| `--api-version` | API version (e.g., `apps/v1`) |

```bash
kubectl explain pod                          # Top-level Pod spec
kubectl explain pod.spec                     # Pod spec fields
kubectl explain pod.spec.containers          # Container spec
kubectl explain pod.spec.containers.resources
kubectl explain deployment.spec.strategy     # Deployment strategy fields
kubectl explain pod --recursive              # Full recursive tree
kubectl explain pod --api-version=v1
```

---

## Creating Resources

### `kubectl apply` — Declarative management (recommended)

| Flag | Short | Description |
|------|-------|-------------|
| `--filename` | `-f` | File, directory, or URL |
| `--recursive` | `-R` | Process directory recursively |
| `--selector` | `-l` | Filter by label selector |
| `--prune` | | Remove resources not in the config |
| `--dry-run` | | `client` or `server` |
| `--force` | | Force apply (delete + re-create if needed) |
| `--server-side` | | Use server-side apply |
| `--field-manager` | | Manager name for server-side apply |
| `--force-conflicts` | | Force ownership of shared fields (server-side apply) |
| `--validate` | | Validate input: `true`, `false`, `strict`, `warn` |
| `--record` | | Record command in annotation (**deprecated**) |

```bash
kubectl apply -f pod.yaml
kubectl apply -f deployment.yaml -f service.yaml
kubectl apply -f ./manifests/                        # All files in directory
kubectl apply -f ./manifests/ -R                     # Recursive
kubectl apply -f https://raw.githubusercontent.com/...  # From URL
kubectl apply -f pod.yaml --dry-run=client           # Client-side dry run
kubectl apply -f pod.yaml --dry-run=server           # Server-side dry run (validates against API)
kubectl apply -f pod.yaml --server-side              # Server-side apply
kubectl apply -k ./kustomize/                        # Kustomize directory
kubectl diff -f pod.yaml                             # Preview changes before apply
```

### `kubectl create` — Imperative creation

| Resource | Command | Key Flags |
|----------|---------|-----------|
| Namespace | `kubectl create namespace NAME` | — |
| Deployment | `kubectl create deployment NAME --image=IMG` | `--replicas`, `--port` |
| Service (ClusterIP) | `kubectl create service clusterip NAME --tcp=PORT:TARGET` | — |
| Service (NodePort) | `kubectl create service nodeport NAME --tcp=PORT:TARGET` | `--node-port` |
| Service (LoadBalancer) | `kubectl create service loadbalancer NAME --tcp=PORT:TARGET` | — |
| ConfigMap | `kubectl create configmap NAME` | `--from-literal`, `--from-file`, `--from-env-file` |
| Secret (generic) | `kubectl create secret generic NAME` | `--from-literal`, `--from-file` |
| Secret (tls) | `kubectl create secret tls NAME` | `--cert`, `--key` |
| Secret (docker-registry) | `kubectl create secret docker-registry NAME` | `--docker-server`, `--docker-username`, `--docker-password` |
| Job | `kubectl create job NAME --image=IMG` | `--from=cronjob/NAME` |
| CronJob | `kubectl create cronjob NAME --image=IMG --schedule=CRON` | — |
| ServiceAccount | `kubectl create serviceaccount NAME` | — |
| Role | `kubectl create role NAME --verb=V --resource=R` | `--resource-name` |
| ClusterRole | `kubectl create clusterrole NAME --verb=V --resource=R` | `--non-resource-url` |
| RoleBinding | `kubectl create rolebinding NAME --role=R --user=U` | `--serviceaccount`, `--group` |
| ClusterRoleBinding | `kubectl create clusterrolebinding NAME --clusterrole=R --user=U` | `--serviceaccount`, `--group` |
| Ingress | `kubectl create ingress NAME --rule=HOST/PATH=SVC:PORT` | `--annotation`, `--class` |
| PriorityClass | `kubectl create priorityclass NAME --value=V` | `--global-default`, `--preemption-policy` |
| Quota | `kubectl create quota NAME --hard=cpu=1,memory=1G,pods=10` | `--scopes` |
| Token | `kubectl create token SA_NAME` | `--duration`, `--audience` |

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

# Generate YAML without creating (combine with dry-run)
kubectl create deployment web --image=nginx --dry-run=client -o yaml > deployment.yaml
kubectl create service clusterip web --tcp=80:80 --dry-run=client -o yaml > service.yaml
```

### `kubectl run` — Quick pod creation

| Flag | Short | Description |
|------|-------|-------------|
| `--image` | | Container image |
| `--port` | | Container port to expose |
| `--env` | | Set environment variables |
| `--labels` | `-l` | Set labels |
| `--command` | | Use command mode (not args mode) |
| `--restart` | | Restart policy: `Always`, `OnFailure`, `Never` |
| `--rm` | | Delete pod after it exits (requires `-it`) |
| `--overrides` | | Inline JSON override of pod spec |
| `--serviceaccount` | | Service account name |
| `--attach` | | Attach to the pod |
| `--stdin` | `-i` | Keep stdin open |
| `--tty` | `-t` | Allocate TTY |
| `--dry-run` | | `client` or `server` |
| `--image-pull-policy` | | `Always`, `IfNotPresent`, `Never` |
| `--privileged` | | Run in privileged mode |

```bash
kubectl run nginx --image=nginx                        # Quick pod
kubectl run nginx --image=nginx --port=80              # With port
kubectl run busybox --image=busybox -it --rm -- sh     # Temp interactive pod
kubectl run test --image=curlimages/curl -it --rm -- curl http://myservice
kubectl run nginx --image=nginx --dry-run=client -o yaml   # Generate YAML
kubectl run nginx --image=nginx --env="DB_HOST=db" --labels="app=web,env=dev"
kubectl run nginx --image=nginx --restart=Never        # Pod (no restart)
kubectl run nginx --image=nginx --overrides='{"spec":{"nodeSelector":{"disk":"ssd"}}}'
```

---

## Editing & Patching Resources

### `kubectl edit` — Edit in your editor

```bash
kubectl edit deployment mydeployment
kubectl edit svc myservice
kubectl edit pod mypod                            # Limited fields editable
KUBE_EDITOR="nano" kubectl edit deployment web    # Use nano instead of vi
kubectl edit deployment web -o json               # Edit as JSON
```

### `kubectl patch` — Partial update

| Flag | Description |
|------|-------------|
| `--type` | `strategic` (default), `merge`, `json` |
| `-p` | Patch data (JSON or YAML) |

```bash
# Strategic merge patch (default)
kubectl patch deployment web -p '{"spec":{"replicas":5}}'

# Merge patch
kubectl patch deployment web --type=merge -p '{"spec":{"replicas":5}}'

# JSON patch (add, remove, replace, move, copy, test)
kubectl patch deployment web --type=json -p '[{"op":"replace","path":"/spec/replicas","value":5}]'

# Patch a node (e.g., add label)
kubectl patch node worker1 -p '{"metadata":{"labels":{"disk":"ssd"}}}'

# Patch a service
kubectl patch svc web -p '{"spec":{"type":"LoadBalancer"}}'

# Patch with YAML file
kubectl patch deployment web --patch-file patch.yaml
```

### `kubectl replace` — Full replacement

```bash
kubectl replace -f pod.yaml                   # Replace entire resource
kubectl replace --force -f pod.yaml           # Force (delete + re-create)
```

### `kubectl set` — Update specific fields

| Command | Description |
|---------|-------------|
| `kubectl set image` | Update container image |
| `kubectl set env` | Update environment variables |
| `kubectl set resources` | Update resource requests/limits |
| `kubectl set serviceaccount` | Update service account |
| `kubectl set selector` | Update selector |
| `kubectl set subject` | Update subjects in role bindings |

```bash
kubectl set image deployment/web nginx=nginx:1.25
kubectl set image deployment/web *=nginx:1.25           # All containers
kubectl set env deployment/web DB_HOST=newdb
kubectl set env deployment/web DB_HOST-                  # Remove env var
kubectl set env deployment/web --list                    # List env vars
kubectl set resources deployment/web -c=nginx --requests=cpu=100m,memory=256Mi --limits=cpu=500m,memory=512Mi
kubectl set serviceaccount deployment/web mysa
```

---

## Deleting Resources

### `kubectl delete`

| Flag | Short | Description |
|------|-------|-------------|
| `--filename` | `-f` | Delete from file/directory |
| `--selector` | `-l` | Delete by label selector |
| `--all` | | Delete all resources of a type |
| `--all-namespaces` | `-A` | All namespaces |
| `--now` | | Immediate shutdown (grace period = 1) |
| `--force` | | Force deletion (grace period = 0) |
| `--grace-period` | | Override grace period (seconds) |
| `--cascade` | | `background` (default), `orphan`, `foreground` |
| `--wait` | | Wait for resources to be gone (default `true`) |
| `--dry-run` | | `client` or `server` |
| `--ignore-not-found` | | Don't error if resource is missing |
| `--timeout` | | Timeout for delete |
| `--field-selector` | | Delete by field selector |

```bash
kubectl delete pod mypod
kubectl delete pod mypod --now                         # Immediate (grace = 1)
kubectl delete pod mypod --force --grace-period=0      # Force delete (stuck pods)
kubectl delete -f deployment.yaml                      # Delete from manifest
kubectl delete pods -l app=web                         # By label
kubectl delete pods --all                              # All pods in namespace
kubectl delete pods --all -A                           # All pods, all namespaces
kubectl delete deployment,svc web                      # Multiple resource types
kubectl delete ns staging                              # Delete namespace (and everything in it)
kubectl delete pods --field-selector status.phase=Failed   # By field
kubectl delete pod mypod --cascade=orphan              # Delete pod but keep dependents
kubectl delete pod mypod --dry-run=client              # Dry run
kubectl delete pod mypod --ignore-not-found            # No error if missing
```

> [!WARNING]
> `kubectl delete ns NAME` deletes the namespace **and all resources inside it**. This is irreversible.

---

## Pod Operations

### Logs

| Flag | Short | Description |
|------|-------|-------------|
| `--follow` | `-f` | Stream logs |
| `--tail` | | Lines from end (default `-1` = all) |
| `--since` | | Duration (e.g., `1h`, `30m`) |
| `--since-time` | | RFC3339 timestamp |
| `--timestamps` | | Include timestamps |
| `--container` | `-c` | Specific container in multi-container pod |
| `--all-containers` | | Logs from all containers |
| `--previous` | `-p` | Logs from previous (crashed) container instance |
| `--prefix` | | Prefix each log line with pod/container name |
| `--max-log-requests` | | Max concurrent log requests (when using selectors) |
| `--selector` | `-l` | Label selector |
| `--ignore-errors` | | Ignore errors when fetching logs from multiple pods |

```bash
kubectl logs mypod
kubectl logs mypod -f                              # Stream/follow
kubectl logs mypod --tail=100                      # Last 100 lines
kubectl logs mypod --since=1h                      # Last hour
kubectl logs mypod -c mycontainer                  # Specific container
kubectl logs mypod --all-containers                # All containers
kubectl logs mypod -p                              # Previous (crashed) instance
kubectl logs mypod --timestamps                    # With timestamps
kubectl logs -l app=web                            # All pods with label
kubectl logs -l app=web --all-containers --prefix  # All pods, all containers, prefixed
kubectl logs -f deployment/web                     # Follow deployment logs
kubectl logs job/myjob                             # Job logs
```

### Exec

```bash
kubectl exec mypod -- ls /app                      # Run command
kubectl exec -it mypod -- bash                     # Interactive shell
kubectl exec -it mypod -- sh                       # sh (if bash unavailable)
kubectl exec -it mypod -c mycontainer -- bash      # Specific container
kubectl exec mypod -- env                           # List env vars
kubectl exec mypod -- cat /etc/resolv.conf          # Check DNS config
kubectl exec mypod -- printenv                      # All environment variables
```

### Port Forward

| Flag | Description |
|------|-------------|
| `--address` | Addresses to listen on (default `localhost`) |

```bash
kubectl port-forward pod/mypod 8080:80                # Pod port forward
kubectl port-forward svc/myservice 8080:80             # Service port forward
kubectl port-forward deploy/mydeployment 8080:80       # Deployment port forward
kubectl port-forward pod/mypod 8080:80 --address=0.0.0.0   # Listen on all interfaces
kubectl port-forward pod/mypod 8080:80 9090:9090       # Multiple ports
```

### Copy Files

```bash
kubectl cp mypod:/path/to/file ./local-file            # Pod → Local
kubectl cp ./local-file mypod:/path/to/file            # Local → Pod
kubectl cp mypod:/path/to/dir ./local-dir              # Copy directory
kubectl cp ns/mypod:/file ./file                       # With namespace
kubectl cp mypod:/file ./file -c mycontainer           # Specific container
```

### Attach

```bash
kubectl attach mypod -it                  # Attach to running process
kubectl attach mypod -c mycontainer -it   # Specific container
```

### Debug (Ephemeral Containers)

```bash
kubectl debug mypod -it --image=busybox                     # Debug with sidecar
kubectl debug mypod -it --image=busybox --target=mycontainer  # Share PID namespace
kubectl debug mypod -it --copy-to=mypod-debug --share-processes  # Copy pod for debug
kubectl debug node/mynode -it --image=ubuntu                 # Debug a node
```

---

## Deployments & Rollouts

### `kubectl rollout` — Manage rollouts

| Command | Description |
|---------|-------------|
| `kubectl rollout status` | Watch rollout progress |
| `kubectl rollout history` | View rollout history |
| `kubectl rollout undo` | Rollback to previous revision |
| `kubectl rollout pause` | Pause a rollout |
| `kubectl rollout resume` | Resume a paused rollout |
| `kubectl rollout restart` | Trigger a rolling restart |

```bash
# Status
kubectl rollout status deployment/web
kubectl rollout status deployment/web --timeout=5m

# History
kubectl rollout history deployment/web
kubectl rollout history deployment/web --revision=3

# Undo / Rollback
kubectl rollout undo deployment/web                        # Previous revision
kubectl rollout undo deployment/web --to-revision=2        # Specific revision

# Pause & Resume (for batch changes)
kubectl rollout pause deployment/web
kubectl set image deployment/web nginx=nginx:1.25
kubectl set resources deployment/web -c=nginx --limits=cpu=500m
kubectl rollout resume deployment/web

# Rolling restart
kubectl rollout restart deployment/web
kubectl rollout restart daemonset/myds
```

### Scaling

```bash
kubectl scale deployment web --replicas=5
kubectl scale deployment web --replicas=0                  # Scale to zero
kubectl scale deployment web --replicas=3 --current-replicas=5   # Precondition
kubectl scale statefulset myss --replicas=3
kubectl scale replicaset myrs --replicas=2
```

### `kubectl expose` — Create a Service for a resource

| Flag | Description |
|------|-------------|
| `--port` | Service port |
| `--target-port` | Container port |
| `--type` | `ClusterIP` (default), `NodePort`, `LoadBalancer`, `ExternalName` |
| `--name` | Service name |
| `--protocol` | `TCP` (default), `UDP`, `SCTP` |
| `--selector` | Label selector |
| `--external-ip` | External IPs |
| `--cluster-ip` | ClusterIP address |

```bash
kubectl expose deployment web --port=80 --target-port=8080 --type=LoadBalancer
kubectl expose pod mypod --port=80 --name=mypod-svc
kubectl expose deployment web --port=80 --type=NodePort
kubectl expose deployment web --port=80 --dry-run=client -o yaml
```

---

## Services & Networking

### Service Types Summary

| Type | Description |
|------|-------------|
| `ClusterIP` | Internal-only (default). Accessible within the cluster. |
| `NodePort` | Exposes on each Node's IP at a static port (30000–32767). |
| `LoadBalancer` | Provisions external load balancer (cloud provider). |
| `ExternalName` | Maps to a DNS CNAME record. |
| `Headless` | `ClusterIP: None`. Returns Pod IPs directly. Used for StatefulSets. |

```bash
kubectl get svc
kubectl get svc -A
kubectl get svc myservice -o yaml
kubectl get endpoints myservice               # View endpoint IPs
kubectl get endpointslices -l kubernetes.io/service-name=myservice
kubectl describe svc myservice
```

### DNS Resolution Inside Cluster

```bash
# Service DNS: <service>.<namespace>.svc.cluster.local
kubectl run tmp --image=busybox -it --rm -- nslookup myservice.default.svc.cluster.local
kubectl run tmp --image=busybox -it --rm -- wget -qO- http://myservice:80
```

---

## ConfigMaps & Secrets

### ConfigMaps

```bash
# Create
kubectl create configmap myconfig --from-literal=key1=val1 --from-literal=key2=val2
kubectl create configmap myconfig --from-file=config.properties
kubectl create configmap myconfig --from-file=mykey=config.properties   # Custom key
kubectl create configmap myconfig --from-env-file=.env

# View
kubectl get configmap myconfig -o yaml
kubectl describe configmap myconfig

# Edit
kubectl edit configmap myconfig

# Delete
kubectl delete configmap myconfig
```

### Secrets

```bash
# Create
kubectl create secret generic mysecret --from-literal=user=admin --from-literal=pass=s3cr3t
kubectl create secret generic mysecret --from-file=ssh-key=~/.ssh/id_rsa
kubectl create secret tls tls-secret --cert=tls.crt --key=tls.key
kubectl create secret docker-registry regcred \
  --docker-server=registry.io \
  --docker-username=user \
  --docker-password=pass

# View
kubectl get secret mysecret -o yaml
kubectl get secret mysecret -o jsonpath='{.data.pass}' | base64 -d    # Decode value

# Delete
kubectl delete secret mysecret
```

> [!IMPORTANT]
> Secrets are **base64 encoded, not encrypted** by default. Enable encryption at rest in your cluster for real security.

---

## Namespaces

```bash
kubectl get namespaces                         # List all
kubectl get ns                                 # Short alias
kubectl create namespace staging
kubectl delete namespace staging               # Deletes ALL resources in it

# Set default namespace for current context
kubectl config set-context --current --namespace=staging

# Get resources across all namespaces
kubectl get pods -A
kubectl get all -A
```

---

## Nodes

```bash
kubectl get nodes                                    # List nodes
kubectl get nodes -o wide                            # Show IPs, OS, kernel
kubectl describe node mynode                         # Detailed info + allocatable resources
kubectl top node                                     # CPU/memory usage (requires metrics-server)
kubectl top node mynode

# Cordon / Uncordon (prevent/allow scheduling)
kubectl cordon mynode                                # Mark unschedulable
kubectl uncordon mynode                              # Mark schedulable

# Drain (safely evict pods before maintenance)
kubectl drain mynode --ignore-daemonsets --delete-emptydir-data
kubectl drain mynode --ignore-daemonsets --force      # Force (ignores PDBs)
kubectl drain mynode --ignore-daemonsets --grace-period=30

# Labels
kubectl label node mynode disk=ssd
kubectl label node mynode disk-                      # Remove label
```

---

## Persistent Storage

### PersistentVolume (PV)

```bash
kubectl get pv                           # List all PVs (cluster-scoped)
kubectl get pv -o wide
kubectl describe pv mypv
kubectl delete pv mypv
```

### PersistentVolumeClaim (PVC)

```bash
kubectl get pvc                          # List PVCs in namespace
kubectl get pvc -A                       # All namespaces
kubectl describe pvc mypvc
kubectl delete pvc mypvc
```

### StorageClass

```bash
kubectl get storageclass                 # or kubectl get sc
kubectl describe sc standard
kubectl get sc -o yaml
```

### Access Modes Reference

| Mode | Abbreviation | Description |
|------|-------------|-------------|
| `ReadWriteOnce` | RWO | Single node read-write |
| `ReadOnlyMany` | ROX | Multiple nodes read-only |
| `ReadWriteMany` | RWX | Multiple nodes read-write |
| `ReadWriteOncePod` | RWOP | Single pod read-write (1.22+) |

### Reclaim Policies

| Policy | Description |
|--------|-------------|
| `Retain` | Keep PV after PVC is deleted (manual cleanup) |
| `Delete` | Delete PV and underlying storage when PVC is deleted |
| `Recycle` | Basic scrub (`rm -rf /thevolume/*`) — **deprecated** |

---

## Jobs & CronJobs

### Jobs

```bash
kubectl create job myjob --image=busybox -- echo "hello"
kubectl create job myjob --from=cronjob/mycron     # One-off from CronJob
kubectl get jobs
kubectl describe job myjob
kubectl logs job/myjob
kubectl delete job myjob
```

### CronJobs

```bash
kubectl create cronjob mycron --image=busybox --schedule="0 * * * *" -- echo "hourly"
kubectl get cronjobs                                # or kubectl get cj
kubectl describe cronjob mycron
kubectl delete cronjob mycron
```

### Key Spec Fields

| Field | Description |
|-------|-------------|
| `spec.completions` | Number of successful completions needed |
| `spec.parallelism` | Number of pods running in parallel |
| `spec.backoffLimit` | Number of retries before marking failed |
| `spec.activeDeadlineSeconds` | Maximum runtime |
| `spec.ttlSecondsAfterFinished` | Auto-cleanup after completion |
| `spec.suspend` | Suspend/resume a CronJob |
| `spec.concurrencyPolicy` | `Allow`, `Forbid`, `Replace` (CronJob) |
| `spec.successfulJobsHistoryLimit` | How many completed jobs to keep (CronJob) |
| `spec.failedJobsHistoryLimit` | How many failed jobs to keep (CronJob) |

---

## DaemonSets & StatefulSets

### DaemonSets

```bash
kubectl get daemonsets                   # or kubectl get ds
kubectl get ds -A
kubectl describe ds myds
kubectl rollout status ds/myds
kubectl rollout restart ds/myds
kubectl delete ds myds
```

> [!NOTE]
> DaemonSets run **one pod per node** (or per matching node). Common use cases: log collectors (Fluentd), monitoring agents (Prometheus node-exporter), network plugins.

### StatefulSets

```bash
kubectl get statefulsets                 # or kubectl get sts
kubectl get sts -A
kubectl describe sts myss
kubectl scale sts myss --replicas=5
kubectl rollout status sts/myss
kubectl rollout restart sts/myss
kubectl delete sts myss
kubectl delete sts myss --cascade=orphan   # Delete StatefulSet but keep pods
```

> [!NOTE]
> StatefulSet guarantees: **stable network identity** (`pod-0`, `pod-1`, ...), **stable persistent storage** (per-pod PVC), **ordered deployment and scaling**. Use for databases, distributed systems.

---

## Horizontal Pod Autoscaler

```bash
# Create HPA
kubectl autoscale deployment web --min=2 --max=10 --cpu-percent=80

# View
kubectl get hpa
kubectl describe hpa web

# Edit
kubectl edit hpa web

# Delete
kubectl delete hpa web

# YAML generation
kubectl autoscale deployment web --min=2 --max=10 --cpu-percent=80 --dry-run=client -o yaml
```

> [!IMPORTANT]
> HPA requires the **metrics-server** to be installed in the cluster.

---

## RBAC

### Roles & ClusterRoles

```bash
# Create
kubectl create role pod-reader --verb=get,list,watch --resource=pods
kubectl create role pod-reader --verb=get --resource=pods --resource-name=mypod
kubectl create clusterrole node-reader --verb=get,list --resource=nodes
kubectl create clusterrole secret-admin --verb='*' --resource=secrets

# View
kubectl get roles
kubectl get clusterroles
kubectl describe role pod-reader
kubectl describe clusterrole cluster-admin

# Delete
kubectl delete role pod-reader
```

### RoleBindings & ClusterRoleBindings

```bash
# Bind role to user
kubectl create rolebinding rb1 --role=pod-reader --user=jane

# Bind role to service account
kubectl create rolebinding rb2 --role=pod-reader --serviceaccount=default:mysa

# Bind clusterrole to user (namespace-scoped)
kubectl create rolebinding rb3 --clusterrole=view --user=jane

# Bind clusterrole to user (cluster-scoped)
kubectl create clusterrolebinding crb1 --clusterrole=cluster-admin --user=admin

# Bind to group
kubectl create clusterrolebinding crb2 --clusterrole=view --group=developers

# View
kubectl get rolebindings
kubectl get clusterrolebindings
kubectl describe rolebinding rb1
```

### `kubectl auth can-i` — Check permissions

```bash
kubectl auth can-i create pods                              # Current user
kubectl auth can-i create pods --as=jane                    # Impersonate user
kubectl auth can-i create pods --as=system:serviceaccount:default:mysa   # As SA
kubectl auth can-i list secrets --namespace=production      # In specific namespace
kubectl auth can-i '*' '*'                                  # Check for admin
kubectl auth can-i --list                                   # List all permissions
kubectl auth can-i --list --as=jane                         # Jane's permissions
kubectl auth can-i --list --namespace=production
```

---

## Service Accounts

```bash
kubectl create serviceaccount mysa
kubectl get serviceaccounts                     # or kubectl get sa
kubectl describe sa mysa
kubectl delete sa mysa

# Create a token for a service account
kubectl create token mysa
kubectl create token mysa --duration=24h

# Assign SA to a deployment
kubectl set serviceaccount deployment/web mysa
```

---

## Network Policies

```bash
kubectl get networkpolicies                      # or kubectl get netpol
kubectl get netpol -A
kubectl describe netpol mypolicy
kubectl delete netpol mypolicy
kubectl apply -f network-policy.yaml
```

### Example: Deny all ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}          # Applies to all pods
  policyTypes:
    - Ingress              # No ingress rules = deny all
```

### Example: Allow from specific namespace

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: frontend
      ports:
        - port: 8080
```

> [!NOTE]
> Network policies require a **CNI plugin** that supports them (e.g., Calico, Cilium, Weave Net). The default kubenet does NOT support network policies.

---

## Ingress

```bash
kubectl get ingress                              # or kubectl get ing
kubectl get ing -A
kubectl describe ingress myingress
kubectl delete ingress myingress
kubectl create ingress myingress --rule="example.com/=web:80" --class=nginx
kubectl create ingress myingress \
  --rule="example.com/api=api-svc:8080" \
  --rule="example.com/=web-svc:80" \
  --annotation="nginx.ingress.kubernetes.io/rewrite-target=/"
```

### Ingress Controllers

| Controller | Description |
|------------|-------------|
| NGINX Ingress | Most popular, community & F5 versions |
| Traefik | Cloud-native, auto-discovery |
| HAProxy | High-performance |
| Istio Gateway | Service mesh integrated |
| AWS ALB | AWS-specific |
| GCE (GKE) | Google Cloud native |
| Contour | Envoy-based |

---

## Resource Quotas & Limit Ranges

### Resource Quotas

```bash
kubectl create quota myquota --hard=cpu=4,memory=8Gi,pods=20
kubectl get resourcequotas                       # or kubectl get quota
kubectl describe quota myquota
kubectl delete quota myquota
```

### Limit Ranges

```bash
kubectl get limitranges                          # or kubectl get limits
kubectl describe limitrange mylimit
kubectl apply -f limitrange.yaml
```

### Example LimitRange

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
spec:
  limits:
    - default:                    # Default limits
        cpu: "500m"
        memory: "256Mi"
      defaultRequest:             # Default requests
        cpu: "100m"
        memory: "128Mi"
      max:
        cpu: "2"
        memory: "1Gi"
      min:
        cpu: "50m"
        memory: "64Mi"
      type: Container
```

---

## Taints, Tolerations & Affinity

### Taints (on Nodes)

```bash
# Add taint
kubectl taint nodes mynode key=value:NoSchedule
kubectl taint nodes mynode key=value:NoExecute
kubectl taint nodes mynode key=value:PreferNoSchedule

# Remove taint (trailing -)
kubectl taint nodes mynode key:NoSchedule-
kubectl taint nodes mynode key-                     # Remove all taints with key

# View taints
kubectl describe node mynode | grep -A 5 Taints
```

### Taint Effects

| Effect | Description |
|--------|-------------|
| `NoSchedule` | Don't schedule new pods without matching toleration |
| `PreferNoSchedule` | Try to avoid scheduling (soft) |
| `NoExecute` | Evict existing pods without matching toleration |

### Node Affinity Types

| Type | Description |
|------|-------------|
| `requiredDuringSchedulingIgnoredDuringExecution` | Hard requirement (must match) |
| `preferredDuringSchedulingIgnoredDuringExecution` | Soft preference (try to match) |

### Pod Affinity / Anti-Affinity

Used to co-locate or spread pods relative to other pods.

```yaml
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values: ["web"]
          topologyKey: kubernetes.io/hostname    # One per node
```

---

## Labels, Selectors & Annotations

### Labels

```bash
# Add/update label
kubectl label pod mypod app=web
kubectl label pod mypod app=web --overwrite          # Update existing

# Add label to all pods
kubectl label pods --all env=dev

# Remove label
kubectl label pod mypod app-

# Show labels
kubectl get pods --show-labels
kubectl get pods -L app,env                          # As columns
```

### Label Selectors

```bash
kubectl get pods -l app=web                          # Equality
kubectl get pods -l 'app!=db'                        # Inequality
kubectl get pods -l 'app in (web,api)'               # Set-based
kubectl get pods -l 'app notin (db)'                 # Not in set
kubectl get pods -l 'environment'                    # Label exists
kubectl get pods -l '!environment'                   # Label doesn't exist
kubectl get pods -l 'app=web,env=prod'               # AND (multiple selectors)
```

### Annotations

```bash
# Add/update annotation
kubectl annotate pod mypod description="My web server"
kubectl annotate pod mypod description="Updated" --overwrite

# Remove annotation
kubectl annotate pod mypod description-

# View annotations
kubectl get pod mypod -o jsonpath='{.metadata.annotations}'
```

---

## Debugging & Troubleshooting

### Quick Diagnosis Flow

```
Pod not starting?
├── kubectl get pods                    → Check STATUS
├── kubectl describe pod NAME           → Check Events section
├── kubectl logs NAME                   → Check application logs
├── kubectl logs NAME -p                → Previous container logs (crash loops)
├── kubectl get events --sort-by='.lastTimestamp'    → Cluster events
└── kubectl debug NAME -it --image=busybox           → Debug container
```

### Events

```bash
kubectl get events                                          # Current namespace
kubectl get events -A                                       # All namespaces
kubectl get events --sort-by='.lastTimestamp'               # Sorted by time
kubectl get events --field-selector type=Warning            # Only warnings
kubectl get events --field-selector involvedObject.name=mypod   # For specific pod
kubectl get events --field-selector reason=FailedScheduling
kubectl get events -w                                       # Watch
```

### Resource Usage (requires metrics-server)

```bash
kubectl top pods                         # Pod CPU/memory usage
kubectl top pods -A                      # All namespaces
kubectl top pods --sort-by=memory        # Sort by memory
kubectl top pods --sort-by=cpu           # Sort by CPU
kubectl top pods -l app=web              # By label
kubectl top pods --containers            # Per-container breakdown
kubectl top node                         # Node usage
```

### Common Pod Statuses & Causes

| Status | Likely Cause |
|--------|-------------|
| `Pending` | No node available, insufficient resources, unbound PVC |
| `ContainerCreating` | Pulling image, mounting volumes |
| `ImagePullBackOff` | Wrong image name/tag, private registry auth, rate limits |
| `CrashLoopBackOff` | App crashing on start — check `kubectl logs -p` |
| `OOMKilled` | Container exceeded memory limit |
| `Error` | Container exited with error code |
| `Evicted` | Node under resource pressure |
| `Terminating` | Being deleted (stuck = check finalizers) |
| `Init:Error` | Init container failed |
| `RunContainerError` | Configuration error (bad command, missing mount) |

---

## Output Formatting & JSONPath

### Output Formats

| Format | Flag | Description |
|--------|------|-------------|
| Wide | `-o wide` | Extra columns |
| YAML | `-o yaml` | Full YAML spec |
| JSON | `-o json` | Full JSON spec |
| Name | `-o name` | Just `type/name` |
| JSONPath | `-o jsonpath='{...}'` | Extract specific fields |
| Go Template | `-o go-template='{{...}}'` | Go template formatting |
| Custom Columns | `-o custom-columns=...` | Table with custom columns |

### JSONPath Examples

```bash
# Pod IP
kubectl get pod mypod -o jsonpath='{.status.podIP}'

# All pod names
kubectl get pods -o jsonpath='{.items[*].metadata.name}'

# All pod names (one per line)
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'

# Pod name and IP in table
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.podIP}{"\n"}{end}'

# Node IPs
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

# All container images
kubectl get pods -o jsonpath='{.items[*].spec.containers[*].image}'

# Specific annotation
kubectl get pod mypod -o jsonpath='{.metadata.annotations.description}'

# Conditions
kubectl get pod mypod -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}'

# All secrets keys
kubectl get secret mysecret -o jsonpath='{.data}' | jq

# Sort-like: Pods with restart counts
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.containerStatuses[0].restartCount}{"\n"}{end}'
```

### Custom Columns

```bash
kubectl get pods -o custom-columns=\
'NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName,IP:.status.podIP'

kubectl get nodes -o custom-columns=\
'NAME:.metadata.name,CPU:.status.capacity.cpu,MEM:.status.capacity.memory'

kubectl get pods -o custom-columns=\
'POD:.metadata.name,CONTAINERS:.spec.containers[*].name,IMAGES:.spec.containers[*].image'
```

---

## kubectl Plugins & Krew

### Krew (Plugin Manager)

```bash
# Install Krew
kubectl krew install                     # Follow https://krew.sigs.k8s.io/docs/user-guide/setup/install/

# Search & install plugins
kubectl krew search
kubectl krew install ctx                 # kubectx
kubectl krew install ns                  # kubens
kubectl krew install neat                # Clean YAML output
kubectl krew install tree                # Resource hierarchy
kubectl krew install whoami              # Show current user
kubectl krew install images              # Show container images
kubectl krew install resource-capacity   # Show cluster capacity
kubectl krew install sniff               # Packet capture
kubectl krew install stern               # Multi-pod log tailing

# List installed
kubectl krew list

# Update
kubectl krew update
kubectl krew upgrade

# Uninstall
kubectl krew uninstall ctx
```

### Popular Standalone Tools

| Tool | Description |
|------|-------------|
| `kubectx` / `kubens` | Fast context and namespace switching |
| `k9s` | Terminal UI for Kubernetes |
| `stern` | Multi-pod log tailing with color |
| `lens` | GUI dashboard |
| `kustomize` | Template-free YAML customization |
| `kubeseal` | Encrypt secrets for Git |
| `kubeshark` | API traffic viewer (like Wireshark for K8s) |
| `popeye` | Cluster resource sanitizer |
| `kube-bench` | CIS benchmark checks |

---

## Helm (Package Manager)

### Repository Commands

```bash
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm repo list
helm repo remove stable
helm search repo nginx                        # Search repos
helm search hub nginx                         # Search Artifact Hub
```

### Chart Commands

```bash
helm show chart bitnami/nginx                 # Chart metadata
helm show values bitnami/nginx                # Default values
helm show readme bitnami/nginx                # README
helm show all bitnami/nginx                   # Everything
helm pull bitnami/nginx                       # Download chart archive
helm pull bitnami/nginx --untar               # Download and extract
helm template myrelease bitnami/nginx         # Render templates locally
```

### Install / Upgrade / Rollback

```bash
# Install
helm install myrelease bitnami/nginx
helm install myrelease bitnami/nginx -n production --create-namespace
helm install myrelease bitnami/nginx -f values.yaml
helm install myrelease bitnami/nginx --set replicaCount=3
helm install myrelease bitnami/nginx --set-string image.tag="1.25"
helm install myrelease bitnami/nginx --dry-run --debug    # Preview
helm install myrelease bitnami/nginx --wait --timeout=5m  # Wait for ready
helm install myrelease bitnami/nginx --atomic             # Rollback on failure
helm install myrelease ./mychart/                         # From local chart

# Upgrade
helm upgrade myrelease bitnami/nginx --set replicaCount=5
helm upgrade myrelease bitnami/nginx -f new-values.yaml
helm upgrade --install myrelease bitnami/nginx            # Install if not exists
helm upgrade myrelease bitnami/nginx --reuse-values       # Keep existing values
helm upgrade myrelease bitnami/nginx --reset-values       # Use default values

# Rollback
helm rollback myrelease                       # Previous revision
helm rollback myrelease 2                     # Specific revision
```

### Release Management

```bash
helm list                                     # List releases
helm list -A                                  # All namespaces
helm list --filter 'web.*'                    # Filter by regex
helm list --deployed                          # Only deployed
helm list --failed                            # Only failed
helm status myrelease                         # Release status
helm history myrelease                        # Release history
helm get values myrelease                     # Current values
helm get manifest myrelease                   # Rendered manifests
helm get all myrelease                        # Everything
helm uninstall myrelease                      # Delete release
helm uninstall myrelease --keep-history       # Keep history
```

### Create Charts

```bash
helm create mychart                           # Scaffold new chart
helm lint mychart/                            # Validate chart
helm package mychart/                         # Package as .tgz
helm dependency update mychart/               # Update chart dependencies
helm dependency build mychart/                # Rebuild dependencies
```

---

## All Resource Types Quick Reference

### Workloads

| Resource | Short | API Group | Namespaced |
|----------|-------|-----------|------------|
| Pod | `po` | `v1` | ✅ |
| ReplicaSet | `rs` | `apps/v1` | ✅ |
| Deployment | `deploy` | `apps/v1` | ✅ |
| StatefulSet | `sts` | `apps/v1` | ✅ |
| DaemonSet | `ds` | `apps/v1` | ✅ |
| Job | `job` | `batch/v1` | ✅ |
| CronJob | `cj` | `batch/v1` | ✅ |
| ReplicationController | `rc` | `v1` | ✅ |

### Networking

| Resource | Short | API Group | Namespaced |
|----------|-------|-----------|------------|
| Service | `svc` | `v1` | ✅ |
| Endpoints | `ep` | `v1` | ✅ |
| EndpointSlice | | `discovery.k8s.io/v1` | ✅ |
| Ingress | `ing` | `networking.k8s.io/v1` | ✅ |
| IngressClass | | `networking.k8s.io/v1` | ❌ |
| NetworkPolicy | `netpol` | `networking.k8s.io/v1` | ✅ |

### Configuration

| Resource | Short | API Group | Namespaced |
|----------|-------|-----------|------------|
| ConfigMap | `cm` | `v1` | ✅ |
| Secret | | `v1` | ✅ |
| ResourceQuota | `quota` | `v1` | ✅ |
| LimitRange | `limits` | `v1` | ✅ |
| HorizontalPodAutoscaler | `hpa` | `autoscaling/v2` | ✅ |
| PodDisruptionBudget | `pdb` | `policy/v1` | ✅ |
| PriorityClass | `pc` | `scheduling.k8s.io/v1` | ❌ |

### Storage

| Resource | Short | API Group | Namespaced |
|----------|-------|-----------|------------|
| PersistentVolume | `pv` | `v1` | ❌ |
| PersistentVolumeClaim | `pvc` | `v1` | ✅ |
| StorageClass | `sc` | `storage.k8s.io/v1` | ❌ |
| CSIDriver | | `storage.k8s.io/v1` | ❌ |
| CSINode | | `storage.k8s.io/v1` | ❌ |
| VolumeAttachment | | `storage.k8s.io/v1` | ❌ |

### RBAC

| Resource | Short | API Group | Namespaced |
|----------|-------|-----------|------------|
| ServiceAccount | `sa` | `v1` | ✅ |
| Role | | `rbac.authorization.k8s.io/v1` | ✅ |
| ClusterRole | | `rbac.authorization.k8s.io/v1` | ❌ |
| RoleBinding | | `rbac.authorization.k8s.io/v1` | ✅ |
| ClusterRoleBinding | | `rbac.authorization.k8s.io/v1` | ❌ |

### Cluster

| Resource | Short | API Group | Namespaced |
|----------|-------|-----------|------------|
| Node | `no` | `v1` | ❌ |
| Namespace | `ns` | `v1` | ❌ |
| Event | `ev` | `v1` | ✅ |
| ComponentStatus | `cs` | `v1` | ❌ |
| Lease | | `coordination.k8s.io/v1` | ✅ |

### Custom Resources

```bash
kubectl get crd                                    # List all CRDs
kubectl get crd -o name                            # Just names
kubectl describe crd mycrd.example.com
kubectl get mycustomresource                       # Use once CRD is installed
```

---

## Grep Combos with kubectl

### Pod Grep Patterns

```bash
# Find pods by name pattern
kubectl get pods | grep "web"

# Find pods in a specific state
kubectl get pods | grep "Running"
kubectl get pods | grep "CrashLoopBackOff"
kubectl get pods | grep "Pending"
kubectl get pods | grep "Error"
kubectl get pods | grep "Evicted"
kubectl get pods -A | grep "ImagePullBackOff"

# Count pods by status
kubectl get pods | grep -c "Running"
kubectl get pods -A | grep -c "CrashLoopBackOff"

# Find pods NOT running
kubectl get pods | grep -v "Running" | grep -v "NAME"

# Find pods with high restart counts
kubectl get pods | grep -v "0         "    # Depends on column alignment

# Case-insensitive search
kubectl get pods -A | grep -i "nginx"

# Find pods on a specific node (wide output)
kubectl get pods -o wide | grep "worker-node-1"

# Get pod names matching pattern
kubectl get pods --no-headers | grep "web" | awk '{print $1}'
```

### Node Grep Patterns

```bash
# Find nodes by status
kubectl get nodes | grep "Ready"
kubectl get nodes | grep "NotReady"
kubectl get nodes | grep "SchedulingDisabled"

# Find node with specific role
kubectl get nodes | grep "control-plane"
kubectl get nodes | grep "worker"

# Node resource info
kubectl describe nodes | grep -A 5 "Allocated resources"
kubectl describe nodes | grep -A 3 "Conditions"
kubectl describe nodes | grep -E "cpu|memory" | grep -i "allocat"
```

### Service & Networking Grep Patterns

```bash
# Find services by type
kubectl get svc -A | grep "LoadBalancer"
kubectl get svc -A | grep "NodePort"
kubectl get svc -A | grep "ClusterIP"

# Find services by port
kubectl get svc | grep "8080"

# Find ingress with specific host
kubectl get ingress -A | grep "example.com"

# Network policies
kubectl get netpol -A | grep "deny"
```

### Log Grep Patterns

```bash
# Search for errors in pod logs
kubectl logs mypod | grep -i "error"

# Multiple patterns
kubectl logs mypod | grep -iE "error|warn|fatal|exception"

# Errors with context
kubectl logs mypod | grep -A 5 -B 2 "Exception"

# Follow logs with grep filter
kubectl logs -f mypod | grep --line-buffered "error"

# Count errors
kubectl logs mypod | grep -c "ERROR"

# Previous container logs (crash debugging)
kubectl logs mypod -p | grep -i "error"

# Exclude noise
kubectl logs mypod | grep -v "healthcheck" | grep -v "readiness"

# Extract IPs from logs
kubectl logs mypod | grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+'

# HTTP status codes
kubectl logs mypod | grep -E 'HTTP/[12][.][01]" [45][0-9]{2}'

# Timestamps + errors
kubectl logs mypod --timestamps | grep "error"

# All containers in a pod
kubectl logs mypod --all-containers | grep -i "error"

# Logs from all pods with a label
kubectl logs -l app=web --all-containers | grep -i "error"
```

### Event Grep Patterns

```bash
# Find warnings
kubectl get events | grep "Warning"

# Scheduling failures
kubectl get events | grep "FailedScheduling"

# Image pull issues
kubectl get events | grep "Failed"
kubectl get events | grep -i "pull"

# OOM kills
kubectl get events | grep "OOMKill"

# Specific pod events
kubectl get events | grep "mypod"

# Volume issues
kubectl get events | grep -i "volume"
kubectl get events | grep -i "mount"
```

### Describe Grep Patterns

```bash
# Find resource requests/limits
kubectl describe pod mypod | grep -A 2 "Limits"
kubectl describe pod mypod | grep -A 2 "Requests"

# Find environment variables
kubectl describe pod mypod | grep -A 20 "Environment"

# Find mount paths
kubectl describe pod mypod | grep -A 5 "Mounts"

# Find node selectors
kubectl describe pod mypod | grep -A 3 "Node-Selectors"

# Find tolerations
kubectl describe pod mypod | grep -A 5 "Tolerations"

# Find conditions
kubectl describe pod mypod | grep -A 10 "Conditions"

# Check taints on nodes
kubectl describe nodes | grep -A 3 "Taints"

# Resource allocation on nodes
kubectl describe nodes | grep -A 8 "Allocated resources"

# Find QoS class
kubectl describe pod mypod | grep "QoS"
```

### YAML/JSON Grep Patterns

```bash
# Find specific fields in YAML output
kubectl get deployment web -o yaml | grep "replicas"
kubectl get pod mypod -o yaml | grep "image:"
kubectl get pod mypod -o yaml | grep -A 3 "resources"

# Find all images across all pods
kubectl get pods -A -o yaml | grep "image:" | sort -u

# Find all used service accounts
kubectl get pods -A -o yaml | grep "serviceAccount:" | sort -u

# Find pods with no resource limits
kubectl get pods -o yaml | grep -B 5 -A 5 "resources: {}"
```

### Multi-Resource Grep

```bash
# Find all resources with a specific label
kubectl get all -l app=web

# Find resources containing "web" across types
kubectl get deploy,svc,ing -A | grep "web"

# Find all resources in a namespace
kubectl get all -n production | grep -v "NAME"

# RBAC: Find all bindings for a user
kubectl get rolebindings,clusterrolebindings -A -o yaml | grep -B 10 "name: jane"

# Find secrets of specific type
kubectl get secrets -A | grep "kubernetes.io/tls"
kubectl get secrets -A | grep "Opaque"
```

### Grep Flag Quick Reference for kubectl

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Case-insensitive | `kubectl get pods \| grep -i "nginx"` |
| `-v` | Invert (exclude) | `kubectl get pods \| grep -v "Running"` |
| `-c` | Count matches | `kubectl get pods \| grep -c "Error"` |
| `-E` | Extended regex (OR) | `kubectl logs pod \| grep -E "error\|warn"` |
| `-A N` | N lines after match | `kubectl describe pod x \| grep -A 5 "Events"` |
| `-B N` | N lines before match | `kubectl describe pod x \| grep -B 3 "Error"` |
| `-C N` | N lines context | `kubectl logs pod \| grep -C 5 "Exception"` |
| `-o` | Only matched part | `kubectl logs pod \| grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+'` |
| `-w` | Whole word | `kubectl get pods \| grep -w "web"` |
| `--line-buffered` | For piped streams | `kubectl logs -f pod \| grep --line-buffered "err"` |
| `--color` | Highlight matches | `kubectl get pods -A \| grep --color "Error"` |

---

## One-Liners & Power Tricks

```bash
# Get all pod IPs
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.podIP}{"\n"}{end}'

# Get all node IPs
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.addresses[?(@.type=="InternalIP")].address}{"\n"}{end}'

# Get all container images running in the cluster
kubectl get pods -A -o jsonpath='{range .items[*]}{range .spec.containers[*]}{.image}{"\n"}{end}{end}' | sort -u

# Find pods without resource limits
kubectl get pods -A -o json | jq -r '.items[] | select(.spec.containers[].resources.limits == null) | .metadata.namespace + "/" + .metadata.name'

# Delete all evicted pods
kubectl get pods -A --field-selector status.phase=Failed | grep Evicted | awk '{print $2 " -n " $1}' | xargs -L1 kubectl delete pod

# Delete all completed/errored pods
kubectl delete pods --field-selector status.phase=Failed -A
kubectl delete pods --field-selector status.phase=Succeeded -A

# Restart all pods in a deployment
kubectl rollout restart deployment/web

# Force delete a stuck pod
kubectl delete pod mypod --force --grace-period=0

# Watch all events (sorted)
kubectl get events -w --sort-by='.lastTimestamp'

# Get top resource-consuming pods
kubectl top pods -A --sort-by=cpu | head -20
kubectl top pods -A --sort-by=memory | head -20

# Copy a secret between namespaces
kubectl get secret mysecret -n source -o yaml | sed 's/namespace: source/namespace: target/' | kubectl apply -n target -f -

# Generate YAML for existing resource (strip managed fields)
kubectl get deployment web -o yaml | kubectl neat    # Requires krew neat plugin

# Quick test pod with curl
kubectl run curl --image=curlimages/curl -it --rm -- curl -s http://myservice.default.svc.cluster.local

# Quick DNS test
kubectl run dns --image=busybox -it --rm -- nslookup kubernetes.default.svc.cluster.local

# List all resources in a namespace
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get --show-kind --ignore-not-found -n production

# Check which pods are running on each node
kubectl get pods -A -o wide --no-headers | awk '{print $8}' | sort | uniq -c | sort -rn

# Get pods sorted by restart count
kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

# Find pods older than 7 days
kubectl get pods -o json | jq -r '.items[] | select(now - (.metadata.creationTimestamp | fromdateiso8601) > 604800) | .metadata.name'

# Decode all secret values
kubectl get secret mysecret -o json | jq -r '.data | to_entries[] | "\(.key): \(.value | @base64d)"'

# Watch pod count per namespace
watch "kubectl get pods -A --no-headers | awk '{print \$1}' | sort | uniq -c | sort -rn"

# Dry-run everything before applying
kubectl diff -f ./manifests/

# Quick port-forward to a service
kubectl port-forward svc/grafana 3000:3000 -n monitoring

# Check PVC status
kubectl get pvc -A -o custom-columns='NAMESPACE:.metadata.namespace,NAME:.metadata.name,STATUS:.status.phase,CAPACITY:.status.capacity.storage,STORAGECLASS:.spec.storageClassName'

# Export all resources in a namespace
kubectl get all -n production -o yaml > production-backup.yaml
```

---

## Interview Quick-Fire Q&A

| # | Question | Answer |
|---|----------|--------|
| 1 | **Pod vs Container?** | A Pod is the smallest K8s unit. It wraps 1+ containers that share network (localhost) and storage. Containers in a pod are always co-scheduled. |
| 2 | **ReplicaSet vs Deployment?** | ReplicaSet maintains N pod replicas. Deployment manages ReplicaSets and adds rollouts, rollbacks, and update strategies. **Never create RS directly.** |
| 3 | **StatefulSet vs Deployment?** | Deployment = stateless, interchangeable pods. StatefulSet = ordered pod names (`pod-0`, `pod-1`), stable network identity, per-pod PVCs. Use for databases. |
| 4 | **DaemonSet use case?** | Runs exactly one pod per node (or per matching node). Used for log collectors, monitoring agents, network plugins, storage daemons. |
| 5 | **Service types?** | `ClusterIP` (internal default), `NodePort` (external via node port), `LoadBalancer` (cloud LB), `ExternalName` (DNS CNAME), `Headless` (ClusterIP=None). |
| 6 | **Ingress vs Service?** | Service exposes pods at L4 (TCP/UDP). Ingress provides L7 HTTP routing (host/path-based, TLS termination). Requires an Ingress Controller. |
| 7 | **ConfigMap vs Secret?** | Both store key-value config. Secrets are base64-encoded (not encrypted by default) and have RBAC restrictions. Use Secrets for sensitive data. |
| 8 | **Liveness vs Readiness vs Startup probes?** | **Liveness**: Is the app alive? (restart if fails). **Readiness**: Can it serve traffic? (remove from Service if fails). **Startup**: Has it started? (protect slow-starting apps). |
| 9 | **What is a Sidecar?** | A secondary container in a pod that supplements the main container. E.g., log shipper, proxy (Envoy in Istio), config reloader. K8s 1.28+ has native sidecar support. |
| 10 | **Init Containers?** | Run **before** app containers, sequentially. Must complete successfully. Used for setup tasks: DB migrations, config generation, waiting for dependencies. |
| 11 | **What is a PodDisruptionBudget (PDB)?** | Limits how many pods can be unavailable during voluntary disruptions (drains, upgrades). Ensures minimum availability. |
| 12 | **Taint vs Toleration vs Affinity?** | **Taint** (on node): repels pods. **Toleration** (on pod): allows scheduling on tainted node. **Affinity** (on pod): attracts pod to specific nodes/pods. |
| 13 | **What are Resource Requests vs Limits?** | **Request** = guaranteed minimum (used for scheduling). **Limit** = maximum allowed (OOMKilled if exceeded for memory, throttled for CPU). |
| 14 | **QoS Classes?** | **Guaranteed** (requests == limits for all containers), **Burstable** (at least one request set), **BestEffort** (no requests/limits). Eviction order: BestEffort first. |
| 15 | **What is a Namespace?** | Virtual cluster within a physical cluster. Provides scope for names, resource quotas, RBAC. Default namespaces: `default`, `kube-system`, `kube-public`, `kube-node-lease`. |
| 16 | **How does DNS work in K8s?** | CoreDNS creates records: `<svc>.<ns>.svc.cluster.local`. Pods get `/etc/resolv.conf` configured automatically. Headless services return pod IPs. |
| 17 | **Rolling Update vs Recreate strategy?** | **RollingUpdate** (default): gradually replaces old pods with new. Configurable via `maxSurge` and `maxUnavailable`. **Recreate**: kills all old pods first, then creates new (downtime). |
| 18 | **What is ETCD?** | Distributed key-value store. Stores **all** cluster state (resources, configs, secrets). Single source of truth. Must be backed up. |
| 19 | **Control Plane components?** | `kube-apiserver` (API gateway), `etcd` (state store), `kube-scheduler` (pod placement), `kube-controller-manager` (reconciliation loops), `cloud-controller-manager` (cloud provider). |
| 20 | **Worker Node components?** | `kubelet` (node agent, runs pods), `kube-proxy` (network rules, Service routing), **container runtime** (containerd, CRI-O). |
| 21 | **What is a CRD?** | Custom Resource Definition. Extends K8s API with custom resource types. Used by operators (e.g., Prometheus Operator creates `ServiceMonitor` CRD). |
| 22 | **What is an Operator?** | Pattern that uses CRDs + custom controllers to automate complex application lifecycle (install, upgrade, backup, failover). E.g., database operators. |
| 23 | **NetworkPolicy?** | Firewall rules for pod-to-pod traffic. Whitelist-based (default: allow all). Requires a CNI that supports it (Calico, Cilium). |
| 24 | **What is Helm?** | Package manager for K8s. Charts = templated YAML bundles. Supports versioning, rollbacks, value overrides. `helm install`, `helm upgrade`, `helm rollback`. |
| 25 | **`kubectl apply` vs `kubectl create`?** | `create` = imperative, fails if exists. `apply` = declarative, creates or updates. **Always prefer `apply`** for production (GitOps-friendly). |

---

> [!TIP]
> **Interview pro tips:**
> - Understand the **reconciliation loop** pattern (desired state vs actual state)
> - Know the **pod scheduling flow**: API Server → Scheduler → kubelet → container runtime
> - Be able to explain **how a Service routes traffic** (kube-proxy, iptables/IPVS, Endpoints)
> - Practice **imperative commands with dry-run** for CKA/CKAD exams:
>   ```bash
>   kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
>   kubectl create deployment web --image=nginx --dry-run=client -o yaml > deploy.yaml
>   kubectl expose deployment web --port=80 --dry-run=client -o yaml > svc.yaml
>   ```
> - Know the **difference between `kubectl apply`, `patch`, `edit`, `replace`, and `set`**
> - For CKA: Practice `etcd` backup/restore, cluster upgrades with `kubeadm`, and certificate management

---

*Last updated: August 2026 | Kubernetes v1.31+ / kubectl*
