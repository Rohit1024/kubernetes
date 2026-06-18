# Persistent Volumes & StorageClasses on GKE

We designed and delivered Lesson 6 covering:
- **Storage Primitives**: Core concepts of Persistent Volumes (PV), Persistent Volume Claims (PVC), and StorageClasses (SC).
- **Dynamic Provisioning**: How StorageClasses dynamically interact with cloud provider APIs to automate disk provisioning.
- **Access Modes & Reclaim Policies**: Deep-dive into access modes (`ReadWriteOnce`, `ReadOnlyMany`, `ReadWriteMany`) and reclaim policies (`Delete`, `Retain`).
- **GKE Implementations**: Best practices like using `volumeBindingMode: WaitForFirstConsumer` to avoid cross-zone scheduling deadlocks.
- **Troubleshooting**: Diagnosing pending PVC states (`kubectl describe pvc`) and resolving volume multi-attach conflicts during rolling deployments.

The curriculum was updated in `NOTES.md` and a premium interactive guide was created at `lessons/0006-pv-pvc-storageclasses.html`.
