[[Troubleshooting]]

https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Troubleshooting/Application-Failure/page
A concise troubleshooting workflow to diagnose Kubernetes application failures by checking user endpoints, Services, pods, logs, and datastore, with commands and common pitfalls.
Before you run commands, draw a small topology diagram showing how components connect (user → web Service → web Pod → DB Service → DB Pod). This reduces guesswork and helps decide whether to begin at the frontend or backend based on symptoms.
## Troubleshooting checklist (ordered)
1) Test the user-facing endpoint
2) Inspect the Service
3) Check the backing Pod(s)
4) Read application logs
5) Move to the database layer (if web layer looks healthy)
6) Use describe for deeper diagnostics
## Quick command reference

| Task                                 | Command                                       |
| ------------------------------------ | --------------------------------------------- |
| Test external access (NodePort)      | `curl http://<node-ip>:<node-port>`           |
| Describe a Service                   | `kubectl describe service web-service`        |
| List pods                            | `kubectl get pods`                            |
| Describe a Pod                       | `kubectl describe pod webapp-mysql`           |
| Follow pod logs                      | `kubectl logs -f webapp-mysql`                |
| Previous pod logs                    | `kubectl logs -p webapp-mysql`                |
| multi-container pods                 | kubectl logs webapp-mysql -c <container-name> |
| Exec into a pod for in-cluster tests | `kubectl exec -it <pod> -- /bin/sh`           |
Common pitfalls: Service selectors that don’t match pod labels, container ports misconfigured relative to Service targetPort, NetworkPolicies or firewall rules blocking traffic, and kube-proxy issues. Confirm labels, ports, and network rules before assuming application-level failures.
## Additional diagnostic tips
- If Services report endpoints but traffic still fails, check:
    - NetworkPolicies (restricting Pod-to-Pod traffic),
    - Node firewall rules and security groups,
    - kube-proxy status and mode on nodes.
- To verify connectivity from within the cluster, create a temporary debug pod (busybox, curl images) and exec into it to curl Service IPs and pod IPs.
- For troubleshooting liveness/readiness probe failures, inspect probe definitions in the Pod spec and pod events for probe failure messages.
