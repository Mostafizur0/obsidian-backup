In a Kubernetes cluster, there isn't one single "folder" where everything lives. Instead, data is split between the **Control Plane** (where the brain lives) and the **Nodes** (where the muscle lives).

Here is the map of where your manifests and images are stored.

---

## 1. Where are the Manifests?

There are two types of manifests you'll encounter in the CKA exam:

### **Static Pod Manifests**

These define the core components (API Server, ETCD, Scheduler). If you move or edit a file here, the component restarts automatically.

- **Location:** `/etc/kubernetes/manifests/`
    
- **How to view:** `ls /etc/kubernetes/manifests/`
    

### **Cluster Object Manifests (API Server)**

Objects like Deployments, Services, and Pods created via `kubectl` are stored in the **etcd database**, not as files on the disk.

- **How to view:** You must query the API.
    
    Bash
    
    ```
    kubectl get pod <pod-name> -o yaml
    ```
    

---

## 2. Where are the Images?

Images are stored on the **Worker Nodes** (or Master, if it's a single-node cluster) within the container runtime's storage directory.

- **Location:** Usually `/var/lib/containerd/` (for containerd) or `/var/lib/docker/`.
    
- **How to view them:** Do not browse the folders; use the runtime CLI. Since you are using `containerd`, use `crictl`:
    
    Bash
    
    ```
    crictl images
    ```
    
    _Note: If `crictl` complains about a missing socket, run: `crictl --runtime-endpoint unix:///run/containerd/containerd.sock images`._
    

---

## 3. How to view ETCD Data?

Since `etcd` is a key-value store, you can't just `cat` a file to read it. You need the `etcdctl` tool. In a standard `kubeadm` setup, etcd runs as a Static Pod, so you need to provide certificates to "log in."

### **The CKA "Power Command" to Query ETCD:**

Run this on the control-plane node. It uses the certificates already sitting in `/etc/kubernetes/pki`:

Bash

```
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get / --prefix --keys-only
```

- **`get / --prefix`**: Dumps every single key in the database.
    
- **`--keys-only`**: Prevents the screen from being flooded with the raw binary values of the objects.
    

---

## 4. Summary Table

|**Content Type**|**Location**|**Tool to View**|
|---|---|---|
|**Core Components**|`/etc/kubernetes/manifests`|`cat` or `vi`|
|**Cluster State**|ETCD Database|`etcdctl`|
|**Kubeconfig/Certs**|`/etc/kubernetes/`|`ls -R /etc/kubernetes/`|
|**Container Images**|`/var/lib/containerd`|`crictl images`|
|**Pod Logs**|`/var/log/pods`|`kubectl logs` or `tail`|
