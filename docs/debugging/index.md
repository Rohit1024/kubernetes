---
icon: lucide/bug
---

# Debugging workloads and networking

Diagnostic playbooks for investigating container crashes, network timeouts, failed image pulls, and database connection issues in Kubernetes.

## Diagnostic guides

1. **[Troubleshooting CrashLoopBackOff](0001-debugging-crashloopbackoff.md)**
   Investigate container exit codes, termination signals, and previous application crash logs.

2. **[Troubleshooting DNS and service routing](0002-dns-networking-troubleshooting.md)**
   Verify Service endpoints, resolve CoreDNS service names, test direct Pod IP routing, and inspect NetworkPolicies.

3. **[Troubleshooting ImagePullBackOff](0003-debugging-imagepullbackoff.md)**
   Diagnose container registry authentication failures, image tag typos, and VPC network egress limits.

4. **[Troubleshooting database connectivity on GKE](0004-database-connectivity-gke.md)**
   Diagnose application-to-database connection errors, firewall rules, Workload Identity permissions, and Cloud SQL Auth Proxy sidecars.

---

[← Home](../index.md)
