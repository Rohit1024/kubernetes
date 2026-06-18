# GKE Gateway API & GatewayClass

We designed and delivered Lesson 7 covering:
- **Evolution of Ingress**: Explaining why Gateway API is replacing Ingress (monolithic resource vs. role-oriented decoupling, standardization of annotations).
- **Role-Oriented Primitives**: GatewayClass (Infra), Gateway (Operator), and HTTPRoute (Developer) separation of duties.
- **GKE GatewayClasses**: Understanding GKE L7 Load Balancer bindings like `gke-l7-gxlb` (external) and `gke-l7-rilb` (internal).
- **Practical Manifests**: Instantiating a Gateway resource and writing HTTPRoute manifests demonstrating native Canary traffic splits and HTTP-to-HTTPS redirects.
- **Advanced Integrations**: Attaching policies (`GCPGatewayPolicy`, `GCPBackendPolicy`) for Cloud Armor, SSL, and CDN configs.
- **Troubleshooting**: Validating status conditions (e.g. `Accepted`, `Programmed`) on Gateways and matching route attachments.

The curriculum was updated in `NOTES.md` and a premium interactive guide was created at `lessons/0007-gke-gateway-api.html`.
