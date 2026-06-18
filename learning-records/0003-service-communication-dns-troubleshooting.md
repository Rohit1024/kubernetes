# Service-to-Service Communication & DNS Troubleshooting

We designed and delivered Lesson 3 covering:
- **DNS Service Discovery**: How CoreDNS resolves `<service-name>.<namespace>.svc.cluster.local` to ClusterIP.
- **Port vs TargetPort**: Mapping outer traffic ports to internal container ports correctly.
- **Debugging & Troubleshooting Workflow**:
  - Checking endpoints with `kubectl get endpoints`.
  - Simulating busybox/curl pods to run `nslookup` inside the cluster.
  - Verifying direct Pod IP connectivity.
  - Reviewing CoreDNS system logs and NetworkPolicies.

The curriculum was updated in `NOTES.md` and a premium interactive guide was created at `lessons/0003-service-communication-dns-troubleshooting.html`.
