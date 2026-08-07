---
icon: lucide/bug
---

# Debugging Workloads & Networking

Welcome to the debugging section. When applications fail to boot, crash, or fail to communicate inside the cluster, use these step-by-step diagnostic playbooks to isolate and fix the root causes.

## Diagnostic Guides

1. **[Troubleshooting CrashLoopBackOff](0001-debugging-crashloopbackoff.md)**
   : What the status means, how to check container exit codes, and retrieving previous logs.

2. **[Troubleshooting DNS & Service Routing](0002-dns-networking-troubleshooting.md)**
   : Checking service endpoints, resolving CoreDNS service names, bypassing service routing, and checking network policies.

3. **[Troubleshooting ImagePullBackOff](0003-debugging-imagepullbackoff.md)**
   : Diagnosing image pull failures, verifying tags, authentication, and network issues.

4. **[Troubleshooting Database Connectivity](0004-database-connectivity-gke.md)**
   : Diagnosing app-to-database connection failures, checking DNS, firewalls, Workload Identity, and the Cloud SQL Auth Proxy.

---

[← Home](../index.md)
