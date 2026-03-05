Taking an **ETCD backup** is one of the most common tasks in the CKA exam. It is a high-stakes task because if you get the paths or the certificates wrong, the backup fails, and you lose those points.

Since `etcd` is a TLS-secured key-value store, you must provide the correct certificates to talk to it.

---

### **Step 1: Identify the Certificate Paths**

Before running the command, check your static pod manifest to find the exact file paths for the certificates. This ensures you don't guess incorrectly.

Bash

```
cat /etc/kubernetes/manifests/etcd.yaml | grep -E "file|cert|key"
```

You are looking for:

- `--trusted-ca-file` (usually `/etc/kubernetes/pki/etcd/ca.crt`)
    
- `--cert-file` (usually `/etc/kubernetes/pki/etcd/server.crt`)
    
- `--key-file` (usually `/etc/kubernetes/pki/etcd/server.key`)
    

---

### **Step 2: Run the Snapshot Command**

In the exam, they will give you a specific path (e.g., `/opt/cluster_backup.db`). Use the `etcdctl snapshot save` command.

**The "Golden" Command:**

Bash

```
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/cluster_backup.db
```

---

### **Step 3: Verify the Backup**

Never assume the backup worked just because the command finished. Check the status of the file you just created:

Bash

```
ETCDCTL_API=3 etcdctl --write-out=table snapshot status /opt/cluster_backup.db
```

_If you see a table with the **Total Size** and **Hash**, your backup is valid._

---

### **Critical CKA Tips for ETCD**

- **ETCDCTL_API=3**: Always prefix your command with this. By default, `etcdctl` might try to use API v2, which will fail or return empty results in modern Kubernetes.
    
- **The Endpoint**: Use `https://127.0.0.1:2379`. If you are on the control plane, this is the most reliable address.
    
- **Permissions**: If you get a "permission denied" error, run the command with `sudo`.
    
- **Shorthand**: In the exam environment, `etcdctl` might already be installed, but if the command isn't found, you can sometimes run it from inside the etcd pod using `kubectl exec`, though running it directly on the host is the standard CKA method.
    

---

### **The "Hard Part": How to Restore**

The backup is easy; the restore is what breaks clusters. To restore, you usually have to:

1. Stop the API Server and ETCD (by moving files out of `/etc/kubernetes/manifests`).
    
2. Run `etcdctl snapshot restore --data-dir=/var/lib/etcd-new ...`
    
3. Update the `etcd.yaml` manifest to point to the **new** data directory.
    

Restoring an ETCD backup is the "High-Boss" level of the CKA exam. If you miss one step, the API server won't come back online, and the cluster will stay "dead."

Here is the safest way to do it, step-by-step.

---

### **Step 1: Stop the Control Plane**

Kubernetes will try to restart ETCD if you don't stop it first. The cleanest way is to move the static pod manifests out of their folder.

Bash

```
mkdir /tmp/k8s-backup
mv /etc/kubernetes/manifests/*.yaml /tmp/k8s-backup/
```

_Wait 30 seconds. Run `docker ps` or `crictl ps` to ensure the `etcd` and `kube-apiserver` containers have stopped._

---

### **Step 2: Run the Restore Command**

You must specify a **new data directory**. Do not overwrite the existing one yet—it’s safer to point to a new location.

**The Command:**

Bash

```
ETCDCTL_API=3 etcdctl snapshot restore /opt/cluster_backup.db \
  --data-dir=/var/lib/etcd-from-backup
```

- `--data-dir`: This is where the restored database will live.
    

---

### **Step 3: Update the ETCD Manifest**

Now you need to tell the ETCD pod to use the **new** directory (`/var/lib/etcd-from-backup`) instead of the old one (`/var/lib/etcd`).

1. Edit the manifest file you moved: `nano /tmp/k8s-backup/etcd.yaml`
    
2. Find the **Volumes** section (at the bottom) and the **VolumeMounts** section.
    
3. Change the `hostPath` for the `etcd-data` volume:
    
    YAML
    
    ```
    # Look for this section and update the path:
    volumes:
    - name: etcd-data
      hostPath:
        path: /var/lib/etcd-from-backup  # Change this from /var/lib/etcd
        type: DirectoryOrCreate
    ```
    
4. Save and exit.
    

---

### **Step 4: Restart the Control Plane**

Move the manifests back into the magic folder. Kubelet will see them and start the pods automatically.

Bash

```
mv /tmp/k8s-backup/*.yaml /etc/kubernetes/manifests/
```

---

### **Step 5: Verification (The "Breath of Relief")**

Wait about 1 minute for the API server to initialize, then check if your nodes and pods are back:

Bash

```
kubectl get nodes
kubectl get pods -A
```

---

### **Common Trap: Permission Errors**

After restoring to a new directory, sometimes the `etcd` container (which runs as a specific user, not always root) can't read the new folder. If the pod keeps crashing, try:

Bash

```
chown -R 2345:2345 /var/lib/etcd-from-backup
# (2345 is the standard UID for the etcd user in many K8s distros)
```

### **Summary Table: Backup vs. Restore**

|**Action**|**Key Tool**|**Critical Flag**|
|---|---|---|
|**Backup**|`etcdctl snapshot save`|Needs `--cert`, `--key`, `--cacert`|
|**Restore**|`etcdctl snapshot restore`|Needs `--data-dir`|
|**Reboot**|`mv /etc/kubernetes/manifests`|Essential to stop/start the "brain"|
