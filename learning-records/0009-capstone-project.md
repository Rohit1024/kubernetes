# Multi-Tier Application Workflow (Capstone)

We designed and delivered Lesson 9 covering:
- **Capstone Scope**: Synthesizing all prior lessons into a single, comprehensive three-tier production architecture deployment on GKE.
- **Architectural Components**:
  - Stateful PostgreSQL Database (Secrets integration, StatefulSet, PVC templates, standard-rwo StorageClass, Headless service discovery).
  - Stateless REST API Backend (readiness/liveness probes, resource requests/limits bounds, and connection configurations).
  - Front-end proxy (graceful preStop shutdown configuration).
  - GKE Gateway Routing (Global HTTP Load Balancer setup, path routing rules mapping `/` and `/api` paths).
- **Knowledge Validation**: Implementing a verification quiz confirming database state mechanics and debugging strategies.

The curriculum was updated in `NOTES.md` and a premium interactive guide was created at `lessons/0009-capstone-project.html`.
