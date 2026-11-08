[[Troubleshooting]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Troubleshooting/Lab-Solution-Control-Plane-Failure/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Troubleshooting/Control-Plane-Failure/page
Guide to troubleshooting Kubernetes control plane failures by checking node health, control plane components, logs, etcd, certificates, networking, and services to identify root causes.

In this lesson we cover systematic ways to troubleshoot Kubernetes control plane failures. Start troubleshooting at the node level, then inspect control plane components (either pod-managed or host-managed), and finally review logs and cluster dependencies like etcd, certificates, and networking.

Quick checklist

| Check | Pod-managed (kubeadm-style) | Host-managed (systemd/service) |
| - | - | - |
| Node health | `kubectl get nodes` | N/A |
| Control plane components | `kubectl get pods -n kube-system` | `sudo systemctl status kube-apiserver` (or `service kube-apiserver status`) |
| Component logs | `kubectl logs <pod> -n kube-system` | `sudo journalctl -u kube-apiserver` |
| Describe failing pods | `kubectl describe pod <pod> -n kube-system` | N/A |
| Additional checks | `kubectl get componentstatuses` (deprecated — prefer direct checks) | Verify unit files, flags, and environment |

Step-by-step troubleshooting

1. Validate cluster nodes

* Verify nodes are Ready before focusing on control plane services:

```bash theme={null}
$ kubectl get nodes
NAME      STATUS   ROLES    AGE   VERSION
master-1  Ready    master   7d    v1.11.3
worker-1  Ready    <none>   7d    v1.11.3
worker-2  Ready    <none>   7d    v1.11.3
```

* If nodes are NotReady, check `kubelet` and node-level networking/firewall first.

2. Inspect control plane components (pod-managed)

* If your control plane runs as pods (typical with kubeadm), list system pods:

```bash theme={null}
$ kubectl get pods -n kube-system
NAME                             READY   STATUS    RESTARTS   AGE
coredns-78fcdf6894-5dntv         1/1     Running   0          1h
coredns-78fcdf6894-knpzl         1/1     Running   0          1h
etcd-master                      1/1     Running   0          1h
kube-apiserver-master            1/1     Running   0          1h
kube-controller-manager-master   1/1     Running   0          1h
kube-proxy-fvbpj                 1/1     Running   0          1h
kube-proxy-v5r2t                 1/1     Running   0          1h
kube-scheduler-master            1/1     Running   0          1h
weave-net-7kd52                  2/2     Running   1          1h
weave-net-jtl5m                  2/2     Running   1          1h
```

If you see control plane pods in `CrashLoopBackOff`, `ImagePullBackOff`, or `NotReady` status, run `kubectl describe pod <pod-name> -n kube-system` to inspect events and `kubectl logs <pod-name> -n kube-system` for container output. These commands often show configuration, permission, or startup errors.

3. Inspect control plane components (host-managed)

* On clusters where control plane binaries run as host services (not as pods), check the service status on the master node. Many distributions provide both a service wrapper and systemd unit; prefer systemd where available.

Using service wrapper (works on many distros):

```bash theme={null}
$ service kube-apiserver status
● kube-apiserver.service - Kubernetes API Server
  Loaded: loaded (/etc/systemd/system/kube-apiserver.service; enabled; vendor preset: enabled)
  Active: active (running) since Wed 2019-03-20 07:57:25 UTC; 1 weeks 1 days ago
  Docs: https://github.com/kubernetes/kubernetes
  Main PID: 15767 (kube-apiserver)
  Tasks: 13 (limit: 2362)
```

Using systemd directly:

```bash theme={null}
$ sudo systemctl status kube-apiserver
● kube-apiserver.service - Kubernetes API Server
   Loaded: loaded (/etc/systemd/system/kube-apiserver.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2019-03-20 07:57:25 UTC; 1 weeks 1 days ago
   Main PID: 15767 (kube-apiserver)
```

Also verify `kubelet` and `kube-proxy` on worker nodes:

```bash theme={null}
$ service kubelet status
● kubelet.service - Kubernetes Kubelet
   Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2019-03-20 14:22:06 UTC; 1 weeks 1 days ago
   Main PID: 1281 (kubelet)
```

```bash theme={null}
$ service kube-proxy status
● kube-proxy.service - Kubernetes Kube Proxy
   Loaded: loaded (/etc/systemd/system/kube-proxy.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2019-03-20 14:21:54 UTC; 1 weeks 1 days ago
```

4. Inspect logs for control plane components

* Pod-managed components: fetch logs with `kubectl logs`:

```bash theme={null}
$ kubectl logs kube-apiserver-master -n kube-system
I0401 13:45:38.190735       1 server.go:703] external host was not specified, using 172.17.0.117
I0401 13:45:38.194290       1 server.go:145] Version: v1.11.3
I0401 13:45:38.819705       1 plugins.go:158] Loaded 8 mutating admission controller(s) successfully: NamespaceLifecycle,LimitRanger,ServiceAccount,NodeRestriction,Priority,DefaultTolerationSeconds,DefaultStorageClass,MutatingAdmissionWebhook.
I0401 13:45:38.821410       1 plugins.go:161] Loaded 6 validating admission controller(s) successfully: LimitRanger,ServiceAccount,Priority,PersistentVolumeClaimResize,ValidatingAdmissionWebhook,ResourceQuota.
I0401 13:45:38.985453       1 master.go:234] Using reconciler: lease
W0401 13:45:40.900380       1 genericapiserver.go:319] Skipping API batch/v2alpha1 because it has no resources.
W0401 13:45:41.370677       1 genericapiserver.go:319] Skipping API rbac.authorization.k8s.io/v1alpha1 because it has no resources.
W0401 13:45:41.381736       1 genericapiserver.go:319] Skipping API scheduling.k8s.io/v1alpha1 because it has no resources.
```

* Host-managed components: use the host journal (`journalctl`) to view unit logs:

```bash theme={null}
$ sudo journalctl -u kube-apiserver
Mar 20 07:57:25 master-1 systemd[1]: Started Kubernetes API Server.
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.553377 15767 flags.go:33] FLAG: --address="127.0.0.1"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558273 15767 flags.go:33] FLAG: --admission-control="[]"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558333 15767 flags.go:33] FLAG: --advertise-address="192.168.5.11"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558365 15767 flags.go:33] FLAG: --allow-privileged="true"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558587 15767 flags.go:33] FLAG: --api-audiences="[]"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558645 15767 flags.go:33] FLAG: --apiserver-count="3"
```

Common follow-up checks

* etcd health: run `etcdctl` (or check the etcd pod's readiness) and confirm cluster quorum.
* Network: verify connectivity between masters and workers (iptables, firewall, routing, CNI plugin status).
* Certificates and clock: check certificate expiry and NTP / clock skew between nodes.
* Recent changes: review config changes, upgrades, or package installs that preceded the failure.
* Resource pressure: CPU, memory, or disk pressure can cause control plane instability.

If the API server is down, kubelets and kube-proxy might still appear healthy locally, but cluster-level operations will fail. Prioritize restoring the API server and etcd (if applicable) before investigating higher-level application issues.
