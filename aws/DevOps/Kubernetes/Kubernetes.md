https://github.com/hashicorp/memberlist
## Local Kubernetes cluster
https://felipetrindade.com/kubernetes-ingress-load-balancer/
https://github.com/chipmk/docker-mac-net-connect
```
brew install chipmk/tap/docker-mac-net-connect
sudo brew services start chipmk/tap/docker-mac-net-connect
```
## Connect From MAC 
https://www.youtube.com/watch?v=KBqfsfRDv8E
## Master Node Creation
```
sudo apt update && sudo apt upgrade -y
```

How LVM works?

Disable swap as kubernitis can not work with swap on
```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
> [!NOTE]
> sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab - not working, if swap line is not commented out in /etc/fstab file; comment out swap line manually otherwise next reboot will revert the changes

Check swap using one of these commands
```
swapon --show
free -m
htop
lsblk
```

Load necessary kernel modules
> [!NOTE]
> Kernel modules are pieces of code that can be loaded into the Linux kernel to extend its functionality
> **overlayfs** - An overlay kernel module allows you to create a writable overlay filesystem on top of an existing filesystem, enabling modifications without altering the original data.
> https://influentcoder.com/posts/overlayfs/
> **overlay network technology**—often implemented via kernel modules or user-space agents—is commonly used in Kubernetes to enable communication between pods across different nodes. These solutions (e.g., VXLAN in Flannel, OVN-Kubernetes) encapsulate packet traffic to create a virtual layer over the physical network. **Different node pod-to-pod communication**
> Do not confuse **Overlay networking** (used for CNI traffic) with **OverlayFS**, which is a storage driver used for container file systems.
> While overlay networks are a very common choice for Kubernetes networking, they are not the _only_ way nodes communicate; direct **routing or cloud-native VPC networking** (e.g., Terway) are also used.
> https://medium.com/@extio/understanding-kubernetes-node-to-node-communication-a-deep-dive-e1d6a5ff87f3
> **br_netfilter** - br_netfilter plays a critical role in this network management by enabling packet filtering and network policy enforcement on bridged networks. br_netfilter ensures network policies can be enforced on all traffic passing through bridge devices.
> https://architecture-evolution.blogspot.com/2024/07/understanding-brnetfilter-in-kubernetes.html
> https://medium.com/@rifewang/overview-of-kubernetes-cni-network-models-veth-bridge-overlay-bgp-ea9bfa621d32
> https://medium.com/@rifewang/kubernetes-how-kube-proxy-and-cni-work-together-1255d273f291

```
sudo modprobe overlay
sudo modprobe br_netfilter
```

Ensure bridged IPv4/IPv6 traffic is visible to `iptables`:
> [!NOTE]
> Manually enable IPv4 packet forwarding as by default, the Linux kernel does not allow IPv4 packets to be routed between interfaces.
> https://kubernetes.io/docs/setup/production-environment/container-runtimes/#network-configuration
```
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

```
sudo sysctl --system
```

Install Docker or Containerd (Container Runtime)
> [!NOTE]
> On Linux, [control groups](https://kubernetes.io/docs/reference/glossary/?all=true#term-cgroup) are used to constrain resources that are allocated to processes. Both the [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet) and the underlying container runtime need to interface with control groups to enforce [resource management for pods and containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) and set resources such as cpu/memory requests and limits. To interface with control groups, the kubelet and the container runtime need to use a _cgroup driver_. It's critical that the kubelet and the container runtime use the same cgroup driver and are configured the same.
> https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cgroup-drivers

```
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```

> [!NOTE]
> https://kubernetes.io/docs/setup/production-environment/container-runtimes/#override-pause-image-containerd
```
sudo nano /etc/containerd/config.toml
press `Ctrl + W` for search and type [plugins."io.containerd.grpc.v1.cri"]
update below image
sandbox_image = "registry.k8s.io/pause:3.10"
Then restart  the process
sudo systemctl restart containerd
sudo systemctl enable containerd
Check with
sudo systemctl status containerd
```

Install Kubernetes with `kubeadm`
```
sudo apt install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | \
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /" | \
sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
> [!NOTE]
> The following additional packages will be installed:
 > cri-tools kubernetes-cni
 > The following NEW packages will be installed:
> cri-tools kubeadm kubectl kubelet kubernetes-cni
> [[cri-tools]] aims to provide a series of debugging and validation tools for Kubelet CRI
> https://github.com/kubernetes-sigs/cri-tools/blob/master/README.md
> https://kubernetes.io/docs/tasks/debug/debug-cluster/crictl/
> [[kubernetes-cni]] Container Network Interface (CNI) is a framework for dynamically configuring networking resources.
> **Kubeadm** is a tool built to provide `kubeadm init` and `kubeadm join` as best-practice "fast paths" for creating Kubernetes clusters. kubeadm performs the actions necessary to get a minimum viable cluster up and running.
> https://kubernetes.io/docs/reference/setup-tools/kubeadm/
> **kubectl** is a client for the Kubernetes API. The Kubernetes API is an HTTP REST API. This API is the real Kubernetes user interface. Kubernetes is fully controlled through this API. This means that every Kubernetes operation is exposed as an API endpoint and can be executed by an HTTP request to this endpoint. Consequently, the main job of kubectl is to carry out HTTP requests to the Kubernetes API
> https://dockerlabs.collabnix.com/kubernetes/beginners/what-is-kubect.html
> **kubelet** https://www.windriver.com/solutions/learning/what-is-a-kubelet

Initialise the control plane which makes a node **master** (This cidr is required for Flannel)
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

This returns an error for low memory, so need to set static memory more than 1700MB
```
I0114 12:44:08.369706   11192 version.go:261] remote version is much newer: v1.35.0; falling back to: stable-1.33
[init] Using Kubernetes version: v1.33.7
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR Mem]: the system RAM (621 MB) is less than the minimum 1700 MB
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

After changing this rerun the command and it will succedd with this log
```
I0114 13:09:44.584851    1364 version.go:261] remote version is much newer: v1.35.0; falling back to: stable-1.33
[init] Using Kubernetes version: v1.33.7
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.0.17]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s localhost] and IPs [192.168.0.17 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s localhost] and IPs [192.168.0.17 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "super-admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests"
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 501.621905ms
[control-plane-check] Waiting for healthy control plane components. This can take up to 4m0s
[control-plane-check] Checking kube-apiserver at https://192.168.0.17:6443/livez
[control-plane-check] Checking kube-controller-manager at https://127.0.0.1:10257/healthz
[control-plane-check] Checking kube-scheduler at https://127.0.0.1:10259/livez
[control-plane-check] kube-controller-manager is healthy after 1.61258158s
[control-plane-check] kube-scheduler is healthy after 2.646129494s
[control-plane-check] kube-apiserver is healthy after 4.503130757s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node k8s as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: b6360d.2p4p533i7t2lm608
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.17:6443 --token b6360d.2p4p533i7t2lm608 \
        --discovery-token-ca-cert-hash sha256:3d0c5b27a8f8461767287753de29ce4455db5a21d96a70f8f28f012a91be0c9f
```

Connect to the cluster as root user
```
export KUBECONFIG=/etc/kubernetes/admin.conf
```

Install [[Flannel]] network plugin (run only on master node):
```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

```
namespace/kube-flannel created
serviceaccount/flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```

```
kubectl get ns
NAME              STATUS   AGE
default           Active   16m
kube-flannel      Active   46s
kube-node-lease   Active   16m
kube-public       Active   16m
kube-system       Active   16m
```

```
kubectl get pod -A -o wide
NAMESPACE      NAME                          READY   STATUS              RESTARTS      AGE   IP             NODE   NOMINATED NODE   READINESS GATES
kube-flannel   kube-flannel-ds-4jt7d         0/1     Error               3 (40s ago)   91s   192.168.0.17   k8s    <none>           <none>
kube-system    coredns-674b8bbfcf-hltbx      0/1     ContainerCreating   0             17m   <none>         k8s    <none>           <none>
kube-system    coredns-674b8bbfcf-wm5cz      0/1     ContainerCreating   0             17m   <none>         k8s    <none>           <none>
kube-system    etcd-k8s                      1/1     Running             0             17m   192.168.0.17   k8s    <none>           <none>
kube-system    kube-apiserver-k8s            1/1     Running             0             17m   192.168.0.17   k8s    <none>           <none>
kube-system    kube-controller-manager-k8s   1/1     Running             0             17m   192.168.0.17   k8s    <none>           <none>
kube-system    kube-proxy-p2w7j              1/1     Running             0             17m   192.168.0.17   k8s    <none>           <none>
kube-system    kube-scheduler-k8s            1/1     Running             0             17m   192.168.0.17   k8s    <none>           <none>
```
This kube-flannel-ds-4jt7d (Demon set) will get error until at least one worker node is up
```
kubectl get nodes
NAME   STATUS   ROLES           AGE   VERSION
k8s    Ready    control-plane   23m   v1.33.7
```

Add worker node to master
```
kubeadm join 192.168.0.17:6443 --token b6360d.2p4p533i7t2lm608 \
        --discovery-token-ca-cert-hash sha256:3d0c5b27a8f8461767287753de29ce4455db5a21d96a70f8f28f012a91be0c9f
```
If run as non admin user gives an error
```
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR IsPrivilegedUser]: user is not running as root
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```
So run join command as root / admin user
```
[preflight] Running pre-flight checks
[preflight] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[preflight] Use 'kubeadm init phase upload-config --config your-config-file' to re-upload it.
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 501.887312ms
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

No kubectl command runs from worker as it does not have admin cluster config. it has kubelet.conf. So login to the master node and run kubelet commands
```
kubectl get pod -A -o wide
NAMESPACE      NAME                          READY   STATUS              RESTARTS         AGE     IP             NODE           NOMINATED NODE   READINESS GATES
kube-flannel   kube-flannel-ds-4jt7d         0/1     CrashLoopBackOff    30 (3m52s ago)   6h5m    192.168.0.17   k8s            <none>           <none>
kube-flannel   kube-flannel-ds-qlht5         1/1     Running             0                22m     192.168.0.16   k8s-worker-2   <none>           <none>
kube-system    coredns-674b8bbfcf-hltbx      0/1     ContainerCreating   0                6h21m   <none>         k8s            <none>           <none>
kube-system    coredns-674b8bbfcf-wm5cz      0/1     ContainerCreating   0                6h21m   <none>         k8s            <none>           <none>
kube-system    etcd-k8s                      1/1     Running             1 (50m ago)      6h21m   192.168.0.17   k8s            <none>           <none>
kube-system    kube-apiserver-k8s            1/1     Running             1 (50m ago)      6h21m   192.168.0.17   k8s            <none>           <none>
kube-system    kube-controller-manager-k8s   1/1     Running             1 (50m ago)      6h21m   192.168.0.17   k8s            <none>           <none>
kube-system    kube-proxy-p2w7j              1/1     Running             1 (50m ago)      6h21m   192.168.0.17   k8s            <none>           <none>
kube-system    kube-proxy-qb42k              1/1     Running             0                22m     192.168.0.16   k8s-worker-2   <none>           <none>
kube-system    kube-scheduler-k8s            1/1     Running             1 (50m ago)      6h21m   192.168.0.17   k8s            <none>           <none>
```
Only kube-proxy and flannel pods are scheduled in worker node

> [!NOTE]
> KUBE-PROXY
> `kube-proxy` is a **DaemonSet** and is installed when using `kubeadm`. Just to understand kube-proxy, Here’s how Kubernetes services work! A service is a collection of pods, which each have their own IP address (like 10.1.0.3, 10.2.3.5, 10.3.5.6)
> Every Kubernetes service gets an IP address (like 10.23.1.2)
> kube-dns resolves Kubernetes service DNS names to IP addresses (so my-svc.my-namespace.svc.cluster.local might map to 10.23.1.2)
> kube-proxy sets up iptables rules in order to do random load balancing between them.
> So when you make a request to my-svc.my-namespace.svc.cluster.local, it resolves to 10.23.1.2, and then iptables rules on your local host (generated by kube-proxy) redirect it to one of 10.1.0.3 or 10.2.3.5 or 10.3.5.6 at random.
> https://stackoverflow.com/questions/53534553/kubernetes-cni-vs-kube-proxy

Menifest saved in /etc/kubernetis/menifest
kubeadm command to create token

#### Static pod
_Static Pods_ are managed directly by the **kubelet daemon** on a specific node, without the API server observing them. Managed directly by the kubelet, they are continuously monitored based on pod definition files placed in a designated directory. **etcd and kube-apiserver** are examples of static pods.
DaemonSets ensure that a specific pod runs on **all nodes in a cluster, managed by a DaemonSet controller via the API server**. In contrast, static pods are created directly by the kubelet without intervention from the API server or other control plane components. Both methods bypass the kube scheduler.
**They get IP from the node they are scheduled not from [[kubernetes-cni]]**
```
cat /var/lib/kubelet/config.yaml
```

![[Pasted image 20260125184127.png]]

https://notes.kodekloud.com/docs/Kubernetes-and-Cloud-Native-Associate-KCNA/Scheduling/Static-Pods

#### Folder Structure
https://yuminlee2.medium.com/kubernetes-folder-structure-and-functionality-overview-5b4ec10c32bf
![[Pasted image 20260124191013.png]]
#### Etcd
It stores the configuration data, state data, and metadata in Kubernetes. Etcd is a distributed key-value store that Kubernetes uses to store ***all the data related to the state of the cluster. This includes the cluster’s configuration, node information, pod states, service details, and secrets***. In simple terms, etcd is the brain of your Kubernetes cluster, keeping track of everything.
We know that Kubernetes is an orchestration tool whose tasks involve managing application container workloads, their configuration, deployments, service discovery, load balancing, scheduling, scaling, and monitoring, and many more tasks which might spread across multiple machines across many locations. Kubernetes needs to maintain coordination between all the components involved.
But to achieve that reliable coordination, k8s needs a data source that can help with the information about all the components, their required configuration, state data, etc. That data store must provide a **consistent, single source of truth at any given point in time. In Kubernetes, that job is done by etcd**. etcd is the data store used to create and maintain the version of the truth.
https://thesecretlivesofdata.com/raft/
https://medium.com/@__karnati/understanding-etcd-in-kubernetes-a-beginners-guide-743ecf17c361

#### Backing up Your Etcd Cluster
The etcd server is the **only stateful component of the Kubernetes cluster**. Kuberenetes stores all API objects and settings on the etcd server.  
Backing up this storage is **enough to restore the Kubernetes cluster’s state completely.**
If etcd data becomes corrupted or is lost, the entire Kubernetes cluster could be rendered inoperable. Therefore, it’s critical to back up etcd regularly. Backups allow you to restore the cluster to a previous healthy state if something goes wrong, such as:
1. **Prevent Data Loss:** Regular backups to avoid accidental data deletion, data corruption or misconfigurations.
2. **Disaster Recovery:** Off-site backups for recovering from catastrophic etcd cluster failures. Hardware failures, Network issues that cause split-brain situations
3. **Cluster Migration:** Backup before migrating, restore on the new cluster for a seamless transition.
4. **Rollback to Stable State:** Use backups to revert the cluster to a stable state after faulty changes.
5. **Testing & Development:** Snapshot before testing changes; restore if issues arise.

https://learn.k21academy.com/kubernetes/etcd-backup-restore-in-k8s-step-by-step/
https://itgix.com/blog/etcd-cluster-backup-and-restore-for-kubernetes-2/
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \ --cacert=/etc/kubernetes/pki/etcd/ca.crt \ --cert=/etc/kubernetes/pki/etcd/server.crt \ --key=/etc/kubernetes/pki/etcd/server.key \ snapshot save <backup-file-location>
```

Before restoring from a backup, ensure that you have a good understanding of the etcd cluster's state. It's important to **stop the etcd service, perform the restore, and then restart etcd.** Additionally, verify the integrity of your backup files.
```
ETCDCTL_API=3 etcdctl --data-dir="/var/lib/etcd-backup" \ --endpoints=https://127.0.0.1:2379 \ --cacert=/etc/kubernetes/pki/etcd/ca.crt \ --cert=/etc/kubernetes/pki/etcd/server.crt \ --key=/etc/kubernetes/pki/etcd/server.key \ snapshot restore etcd-backup.db
```

The etcd backup process can be automated using scripts and scheduling tools. You can create a script that runs the etcdctl snapshot save command and use a tool like cron (on Linux) or Task Scheduler (on Windows) to schedule regular backups.

### [[Load Balancer]]

### [[Longhorn]]

### [[Ingress]]

