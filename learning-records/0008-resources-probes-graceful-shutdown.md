# Resource Management, Health Probes, & Graceful Shutdown

We designed and delivered Lesson 8 covering:
- **Resource Request vs Limits**: Deep-dive into CPU (compressible resource / throttled) and Memory (incompressible resource / `OOMKilled` Exit Code 137).
- **Application Health Checks**: Distinguishing Startup Probes (slow start protection), Liveness Probes (recovery from deadlocks), and Readiness Probes (routing inclusion).
- **Graceful Shutdown**: The step-by-step Pod deletion lifecycle flow.
- **Zero-Downtime Rollouts**: Implementing `preStop` lifecycle hooks (`sleep 15`) and configuring `terminationGracePeriodSeconds` to prevent race conditions with load balancers during rolling updates.

The curriculum was updated in `NOTES.md` and a premium interactive guide was created at `lessons/0008-resources-probes-graceful-shutdown.html`.
