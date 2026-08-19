# Debugging database connectivity from GKE

When an application running in a Google Kubernetes Engine (GKE) cluster fails to connect to a database (such as Cloud SQL, AlloyDB, or a database on Compute Engine), the error usually stems from configuration, network routing, or IAM authentication.

Use this guide to diagnose and resolve database connection failures.

---

## Diagnostic flowchart

```mermaid
flowchart TD
    Start([App Fails to Connect]) --> Logs{Check Pod Logs}
    
    Logs -- "Authentication Failed" --> IAM[Check IAM & Workload Identity]
    Logs -- "Connection Refused / Timeout" --> Config{Check Connection String}
    Logs -- "Unknown Host" --> DNS[Check DNS Resolution]
    
    Config -- Incorrect --> FixConfig[Update ConfigMap / Secret]
    Config -- Correct --> Ping{Network Connectivity Test}
    
    Ping -- "Port Open" --> AuthProxy[Check Cloud SQL Auth Proxy]
    Ping -- "Port Closed / Timeout" --> FW{Check Firewall Rules}
    
    FW -- "Blocked" --> FixFW[Add VPC Firewall Rule or Authorized Network]
    FW -- "Allowed" --> Route[Check VPC Peering & Routes]
    
    DNS --> Exec[Run `nslookup` inside Pod]
```

---

## Step 1: Verify the application error

Inspect the stdout/stderr logs from the application pod:

```bash
kubectl logs <pod-name> -n <namespace>
```
- **"Unknown host" or "Name or service not known":** DNS resolution failure.
- **"Connection timed out":** Network routing issue, missing VPC firewall rule, or unrouted subnet.
- **"Connection refused":** The destination host was reached, but no process is listening on that port or it is bound only to localhost.
- **"Access denied" or "Authentication failed":** Network connectivity works, but database credentials or IAM permissions are invalid.

---

## Step 2: Validate the connection string

Verify that the Pod receives the correct connection string, username, and password from ConfigMaps and Secrets:

```bash
kubectl get secret db-credentials -o yaml
# Decode the base64 value
echo "<base64string>" | base64 --decode
```
Verify the IP address, port, and credentials match the target database instance.

---

## Step 3: Network diagnostics from the pod

Exec into the pod (or an ephemeral debug container) to test connectivity directly from the Pod network namespace:

```bash
kubectl exec -it <pod-name> -- /bin/sh
```

**1. Test DNS resolution:**
```bash
nslookup <database-hostname>
```
If this fails, check CoreDNS pods in `kube-system`, or verify Cloud DNS private zone bindings.

**2. Test port connectivity:**
```bash
# Example for PostgreSQL (5432) or MySQL (3306)
nc -zv <database-ip> 5432
```
If the connection times out:
- Check GCP VPC Firewall Rules (verify ingress is permitted from the GKE Pod CIDR).
- If using public IP on Cloud SQL, verify the GKE egress NAT IP is listed in "Authorized Networks".
- If using Private Services Access (VPC Peering), verify that custom routes and Pod ranges are exported and imported across the peering connection.

---

## Step 4: Authentication and Cloud SQL Auth Proxy

When connecting to Cloud SQL, the standard setup runs the **Cloud SQL Auth Proxy** as a sidecar container:

1. **Check proxy logs:**
   If the proxy fails to authenticate or connect, the application receives `Connection Refused` when connecting to `127.0.0.1`:
   ```bash
   kubectl logs <pod-name> -c cloud-sql-proxy
   ```
2. **Workload Identity:**
   The proxy requires the `roles/cloudsql.client` IAM role:
   - Does the Kubernetes Service Account (KSA) have the `iam.gke.io/gcp-service-account` annotation?
   - Does the GCP Service Account (GSA) have the `roles/iam.workloadIdentityUser` binding granting access to the KSA?

---

## Step 5: Network policies

Check if any Kubernetes NetworkPolicies in the namespace block egress to external IP ranges:

```bash
kubectl get networkpolicies -n <namespace>
```

---

[← Troubleshooting ImagePullBackOff](./0003-debugging-imagepullbackoff.md) | [Debugging overview](./index.md)
