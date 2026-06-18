# Stateless vs. Stateful Workloads & Secrets Management

We designed and delivered Lesson 4 covering:
- **Stateless vs Stateful workloads**: Deployments vs. StatefulSets, stable identities (`mysql-0`), ordinal scaling, and Headless Services for peer discovery.
- **Stateful gotchas**: Persistent Volume Claims (PVCs) auto-provisioning via `volumeClaimTemplates` and manual PVC deletion safety mechanisms.
- **CI/CD Secrets Integration**: Using base64 encoded Kubernetes native Secrets, mounting secrets as environment variables (`secretKeyRef`) or filesystem volumes.
- **GCP Secret Manager integration**: Implementing the Secrets Store CSI Driver (`SecretProviderClass`) with Workload Identity annotations for automated secret retrieval.

The curriculum was updated in `NOTES.md` and a premium interactive guide was created at `lessons/0004-stateless-stateful-secrets-gcp.html`.
