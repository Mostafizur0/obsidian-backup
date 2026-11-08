[[Kubernetes]]

https://kubernetes.io/docs/tasks/debug/ - official k8s guide
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Troubleshooting/Troubleshooting-Section-Introduction/page
## Quick reference: common kubectl commands

Use these commands frequently while troubleshooting:

- Inspect resources:
    - `kubectl get pods --all-namespaces`
    - `kubectl get events --all-namespaces`
    - `kubectl get nodes -o wide`
- Describe and inspect:
    - `kubectl describe pod <pod-name> -n <namespace>`
    - `kubectl logs <pod-name> -c <container> -n <namespace>`
    - `kubectl exec -it <pod-name> -n <namespace> -- /bin/sh`
- Rollouts and debugging:
    - `kubectl rollout status deployment/<name> -n <namespace>`
    - `kubectl rollout undo deployment/<name> -n <namespace>`
- Cluster and control plane checks:
    - `kubectl get componentstatuses` (deprecated in newer versions — check API server health endpoints)
    - `kubectl get apiservices`
- Node-level inspection:
    - `kubectl describe node <node-name>`
    - `journalctl -u kubelet -n 200` (on the node)

Tip: When capturing logs, include timestamps and context (namespace, node, pod name) so you can correlate events across components and timelines.

## Troubleshooting checklist (step-by-step)

1. Confirm the scope
    - Is the issue isolated to a single pod, deployment, node, or cluster-wide?
2. Gather evidence
    - `kubectl get pods -A`, `kubectl get events -A`, `kubectl top pods` (metrics-server)
3. Inspect resource status and events
    - `kubectl describe` to reveal scheduling, image pull, readiness, or liveness failures
4. Collect logs
    - Application logs: `kubectl logs`
    - Node and system logs: `journalctl`, container runtime logs
5. Test connectivity
    - Use `kubectl exec` and networking tools (`curl`, `nslookup`, `ping`) to test DNS and service reachability
6. Apply mitigations
    - Scale replicas, restart affected pods, adjust resource requests/limits, or rollback deployments
7. Validate and document
    - Re-run status checks and events; capture the root cause and remediation steps for postmortem

## Recommended tools and integrations
- kubectl (core CLI)
- kubectl plugins (kubectl-trace, konnectivity tools)
- Logging: EFK/ELK, Fluentd/Fluent Bit
- Monitoring: Prometheus & Grafana
- Tracing: Jaeger or Zipkin
- Diagnostics: `tcpdump`, `strace` (node debugging), `etcdctl` (etcd cluster checks)
