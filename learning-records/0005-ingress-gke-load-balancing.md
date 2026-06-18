# Ingress & GKE Load Balancing

We designed and delivered Lesson 5 covering:
- **External Traffic Exposure Options**: Comparing NodePort, LoadBalancer, and Ingress routing patterns and architectural trade-offs.
- **Ingress Controller vs Ingress Resource**: Highlighting the declarative configuration (resource) vs routing daemon (controller) separation, specifically on GKE (GCE Ingress Controller / HTTP(S) Load Balancer).
- **GKE Ingress Routing Manifests**: Writing host and path-based ingress rules targeting backend services.
- **Advanced GKE Configs**: Integrating `FrontendConfig` for automatic HTTP-to-HTTPS redirect, and `BackendConfig` for custom GKE health checks, Cloud Armor security policies, and Cloud CDN.
- **Ingress Troubleshooting**: Deep-diving into "502 Bad Gateway" errors, readiness probe failures, and using `kubectl describe ingress` to monitor ingress synchronization events.

The curriculum was updated in `NOTES.md` and a premium interactive guide was created at `lessons/0005-ingress-gke-load-balancing.html`.
