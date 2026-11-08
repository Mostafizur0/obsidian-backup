[[Troubleshooting]]

https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Troubleshooting/Lab-Solutions-Worker-Node-Failure/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Troubleshooting/Worker-Node-Failure/page

Practical steps to troubleshoot Kubernetes worker node failures, inspect node status, kubelet logs and certificates, diagnose resource or connectivity issues, and follow a checklist to restore node health.

In this lesson we cover practical steps to troubleshoot worker node failures in a Kubernetes cluster. You will learn how to identify node status with `kubectl`, interpret node conditions, inspect the node locally (if reachable), examine kubelet logs and certificates, and follow a concise checklist to restore node health.
## 1. Check cluster node status

Start by checking which nodes are Ready, NotReady, or have other conditions:

```bash theme={null}
kubectl get nodes
```

Example output:

```bash theme={null}
$ kubectl get nodes
NAME           STATUS                     ROLES    AGE   VERSION
worker-1       Ready                      <none>   8d    v1.13.0
worker-2       NotReady                   <none>   8d    v1.13.0
worker-3       Ready,SchedulingDisabled   <none>   8d    v1.13.0
```

If a node is NotReady (or has other problems), inspect detailed node information:

```bash theme={null}
kubectl describe node <node-name>
```

Example excerpt:

```bash theme={null}
$ kubectl describe node worker-1
...
Conditions:
 Type           Status  LastHeartbeatTime              Reason                     Message
 ----           ------  ------------------             ------                     -------
 OutOfDisk      False   Mon, 01 Apr 2019 14:30:33 +0000 KubeletHasSufficientDisk   kubelet has sufficient disk space available
 MemoryPressure False   Mon, 01 Apr 2019 14:30:33 +0000 KubeletHasSufficientMemory kubelet has sufficient memory available
 DiskPressure   False   Mon, 01 Apr 2019 14:30:33 +0000 KubeletHasNoDiskPressure   kubelet has no disk pressure
 PIDPressure    False   Mon, 01 Apr 2019 14:30:33 +0000 KubeletHasSufficientPID    kubelet has sufficient PID available
 Ready          True    Mon, 01 Apr 2019 14:30:33 +0000 KubeletReady               kubelet is posting ready status. AppArmor enabled
```

## 2. Understand node conditions

Node conditions indicate node health. Below is a quick reference for common condition types and recommended actions.

| Node condition | What it means | Recommended first action |
| - | - | - |
| OutOfDisk | Node has insufficient disk space | Run `df -h`, clean logs or remove unused images/containers |
| MemoryPressure | Node is experiencing memory pressure | Check `top`, identify memory-hungry processes or increase resources |
| DiskPressure | Disk IO/space constraints affecting the node | Investigate disk usage, pending writes, or filesystem issues |
| PIDPressure | Running out of process IDs | Inspect processes and restart services if needed |
| Ready | Kubelet is healthy and reporting status | If `False` or `Unknown`, check kubelet logs and node reachability |

If the kubelet stops reporting, conditions can switch to `Unknown` and the `Reason` may show `NodeStatusUnknown`. Check `LastHeartbeatTime` to see when the node stopped reporting.

```bash theme={null}
$ kubectl describe node worker-1

...
Conditions:
  Type             Status    LastHeartbeatTime                 Reason                      Message
  ----             ------    -----------------                 ------                      -------
  OutOfDisk        Unknown   Mon, 01 Apr 2019 14:20:20 +0000   NodeStatusUnknown           Kubelet stopped posting node status.
  MemoryPressure   Unknown   Mon, 01 Apr 2019 14:20:20 +0000   NodeStatusUnknown           Kubelet stopped posting node status.
  DiskPressure     Unknown   Mon, 01 Apr 2019 14:20:20 +0000   NodeStatusUnknown           Kubelet stopped posting node status.
  PIDPressure      False     Mon, 01 Apr 2019 14:20:20 +0000   KubeletHasSufficientPID    kubelet has sufficient PID available
  Ready            Unknown   Mon, 01 Apr 2019 14:20:20 +0000   NodeStatusUnknown           Kubelet stopped posting node status.
```

If node conditions are `Unknown`, first verify the node itself (network, power, or hypervisor state). Use the `LastHeartbeatTime` to determine when it stopped reporting, then try to reach the node via SSH or your cloud provider console.

## 3. Troubleshoot on the node (when reachable)

If you can SSH into the node (or access it via cloud console), follow these steps in order:

1. Verify basic reachability and host health
   * Ping the node, check cloud provider instance status, or SSH into it.
   * If the node is powered off or unreachable at hypervisor level, bring it online.

2. Inspect system resource usage
   * Check CPU, memory and load with `top` or `htop`.
   * Inspect disk usage with `df -h` and inode usage with `df -i`.

Example `top` output:

```bash theme={null}
$ top
top - 14:43:56 up 3 days, 19:02, 1 user, load average: 0.35, 0.29, 0.21
Tasks: 112 total,   1 running,  72 sleeping,   0 stopped,   0 zombie
%Cpu(s): 3.9 us,  1.7 sy,  0.1 ni, 94.3 id,  0.0 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem :  1009112 total,   74144 free,  736608 used,  198360 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   129244 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
   34 root      20   0      0      0      0 S  5.9  0.0   0:13.14 kswapd0
28826 999       20   0 1361320  383208  3596 S  5.9 38.0   0:46.95 mysqld
    1 root     20   0  78260   5924   3192 S  0.0  0.6   0:21.88 systemd
```

3. Check kubelet service and inspect logs
   * Verify kubelet is running:

```bash theme={null}
systemctl status kubelet
```

Example:

```bash theme={null}
$ systemctl status kubelet
● kubelet.service - Kubernetes Kubelet
   Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2019-03-20 14:22:06 UTC; 1 week 1 day ago
   Docs: https://github.com/kubernetes/kubernetes
   Main PID: 1281 (kubelet)
    Tasks: 24 (limit: 1152)
```

* Inspect recent kubelet logs for authentication, certificate, network, or filesystem errors:

```bash theme={null}
journalctl -u kubelet -n 200 --no-pager
```

4. Validate kubelet certificates and credentials
   * If kubelet cannot authenticate to the API server, certificate issues (expired, wrong subject/SAN, wrong CA) are common culprits.
   * Check the node certificate:

```bash theme={null}
openssl x509 -in /var/lib/kubelet/worker-1.crt -text -noout
```

Sample certificate excerpt:

```text theme={null}
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            ff:e0:23:9d:fc:78:03:35
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = KUBERNETES-CA
        Validity
            Not Before: Mar 20 08:09:29 2019 GMT
            Not After : Apr 19 08:09:29 2019 GMT
        Subject: CN = system:node:worker-1, O = system:nodes
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
    X509v3 extensions:
        X509v3 Basic Constraints:
            CA:FALSE
        X509v3 Key Usage:
            Digital Signature, Non Repudiation, Key Encipherment
        X509v3 Subject Alternative Name:
            DNS:worker-1, IP Address:192.168.5.11
```

If the certificate is expired, or the `Subject` / SANs do not match the node identity, kubelet registration and authentication will fail and the control plane will mark the node accordingly.

## 4. Quick troubleshooting checklist

* Run `kubectl get nodes` to spot non-Ready nodes.
* Use `kubectl describe node <node>` to examine Conditions and `LastHeartbeatTime`.
* If Conditions are `Unknown`, verify node reachability via SSH or cloud console.
* On the node: check `top`, `df -h`, `free -m` for resource exhaustion.
* Verify kubelet status: `systemctl status kubelet` and `journalctl -u kubelet`.
* Confirm kubelet certificates are valid: `openssl x509 -in /var/lib/kubelet/<node>.crt -text -noout`.
* If kubelet config or certs are broken, re-provision kubelet credentials or rotate certificates as needed.
