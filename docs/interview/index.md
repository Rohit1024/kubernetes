---
icon: lucide/file-question
---

# Kubernetes interview scenarios and troubleshooting

Production troubleshooting scenarios and architectural questions common in SRE, DevOps, and Platform Engineering interviews.

## Available scenarios

* **[Tricky Pod restarts and silent crashes](./01-tricky-pod-restarts.md)**
  Diagnose OOMKilled vs OOM-killer terminations, zombie process exhaustion under PID 1, and failing liveness probes under heavy load.

* **[Networking blackholes and DNS mysteries](./02-networking-blackholes.md)**
  Investigate connection timeouts, conntrack table exhaustion, CoreDNS 5-second query delays with `ndots:5`, and asymmetric routing.

* **[Scheduling and storage anomalies](./03-scheduling-storage.md)**
  Resolve Pending Pod scheduling deadlocks, multi-attach storage errors with AWS EBS and GKE Persistent Disks, and PodDisruptionBudget deadlocks during cluster drains.

---

[← Home](../index.md)
