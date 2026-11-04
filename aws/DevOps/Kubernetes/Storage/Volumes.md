[[Storage]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Storage/Lab-Solution-Persistent-Volumes-and-Persistent-Volume-Claims-optional/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Storage/Volumes/page
![[Pasted image 20261104221616.png]]
Note that not all Kubernetes volume types are persistent—<span style="color: #22C55E;">`emptyDir`, for example, is ephemeral and deleted when the pod is removed.</span> To retain data beyond a pod’s lifetime, use a backing store that persists independently of the pod: <span style="color: #22C55E;">e.g., `hostPath` (node filesystem), network-backed storage (NFS, GlusterFS, CephFS), cloud block/file storage (AWS EBS, Azure Disk/File, GCP PD), or Kubernetes PersistentVolumes (PV) provisioned via a CSI driver.</span>

The example below uses a host directory (`/data`) on the node as the backing storage and mounts it at `/opt` inside the container. Anything written to `/opt` is stored on the node under `/data`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  restartPolicy: Never
  containers:
  - name: alpine
    image: alpine
    command: ["/bin/sh", "-c"]
    args:
      - shuf -i 1-100 -n 1 >> /opt/number.out
    volumeMounts:
    - name: data-volume
      mountPath: /opt
  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: DirectoryOrCreate
```
When this pod runs it appends a random number to `/opt/number.out`. Because `/opt` is backed by the node path `/data`, the file persists on the node at `/data/number.out`. Deleting the pod does not delete that host file.

> [!NOTE]
> Using `hostPath` directly ties storage to a specific node’s filesystem. This is acceptable for single-node clusters or development/testing.
> Do not use `hostPath` in multi-node or production clusters—data will not be portable between nodes and can be lost if the Pod is rescheduled to a different node. Use a networked volume or a CSI driver for durability and portability.

Choose the backend based on <span style="color: #22C55E;">durability, access mode (ReadWriteOnce, ReadOnlyMany, ReadWriteMany)[Explained below], performance, and whether the storage must be accessible across nodes.</span> CSI drivers are the recommended plugin mechanism for most cloud and on-premises storage solutions.

| Storage category            | When to use                             | Example / note                                                              |
| --------------------------- | --------------------------------------- | --------------------------------------------------------------------------- |
| Node-local                  | Fast local disk; data tied to one node  | `hostPath` (not for production multi-node)                                  |
| Network file systems        | Share files across many pods/nodes      | NFS, GlusterFS, CephFS — good for ReadWriteMany                             |
| Block storage               | High performance, often single-AZ       | AWS EBS, GCE Persistent Disk, Azure Disk — typically ReadWriteOnce per zone |
| Managed cloud file services | Shared file storage with cloud provider | Azure File, AWS EFS, Google Filestore — provide ReadWriteMany in many cases |
| CSI-backed volumes          | Modern, extensible driver model         | Any storage supported by a CSI driver (recommended)                         |
Best practices:
- Prefer dynamic provisioning via PersistentVolume and PersistentVolumeClaim (PV/PVC) backed by a CSI driver for production workloads.
- Select storage based on access mode requirements: if multiple nodes need RW access, use shared network or file storage (ReadWriteMany).
- Avoid `hostPath` for production multi-node clusters.
- Use Kubernetes StorageClasses to define performance tiers and dynamic provisioning policies.

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Storage/Persistent-Volumes/page
# Persistent Volumes
When you declare volumes directly inside a Pod spec, the storage configuration (volume type, IDs, filesystem, etc.) is embedded in each Pod manifest. In small examples this works, but in larger clusters this approach is hard to manage: every Pod author must know and repeat the storage configuration, and any change to the storage backend requires updating many manifests.
PersistentVolumes (PVs) solve this by enabling central storage management at the cluster level. An administrator provisions a pool of storage as PVs (or enables dynamic provisioning via a StorageClass). <span style="color: #22C55E;">Users request storage by creating PersistentVolumeClaims (PVCs). The Kubernetes control plane will bind PVCs to matching PVs based on capacity, access modes, and other constraints.</span>

Key concepts at a glance:
- PersistentVolume (PV): a cluster-scoped resource representing a piece of storage.
- PersistentVolumeClaim (PVC): a user request for storage that gets bound to a PV.
- StorageClass: optional resource to enable dynamic provisioning of PVs.
- Binding: the process that matches PVCs to available PVs based on size, access mode, and selectors. (Described below)

<span style="color: #22C55E;">Access modes determine how a volume can be mounted by Pods. Common modes include `ReadWriteOnce`, `ReadOnlyMany`, and `ReadWriteMany`.</span> See the table below for concise descriptions and typical use cases.

| Access Mode     | Description                            | Typical use case                                                          |
| --------------- | -------------------------------------- | ------------------------------------------------------------------------- |
| `ReadWriteOnce` | Mounted as read-write by a single node | Block storage (AWS EBS, GCE PD) used by one Pod at a time per node        |
| `ReadOnlyMany`  | Mounted read-only by many nodes        | Shared read-only content (e.g., shared config or static data)             |
| `ReadWriteMany` | Mounted read-write by many nodes       | Networked file systems (NFS, some CSI plugins) for shared writable access |
Common PV provisioning approaches:

| Provisioning type    | Description                                           | Typical plugins/examples                                                          |
| -------------------- | ----------------------------------------------------- | --------------------------------------------------------------------------------- |
| Static provisioning  | Admin creates PV objects that map to existing storage | `hostPath`, pre-provisioned NFS, cloud volumes (manually created EBS/GCE PD)      |
| Dynamic provisioning | Kubernetes creates PVs on demand via a `StorageClass` | CSI drivers, `kubernetes.io/aws-ebs`, `kubernetes.io/gce-pd` (prefer CSI drivers) |

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  awsElasticBlockStore:
    volumeID: "<volume-id>"
    fsType: ext4
```

```bash
kubectl create -f pv-definition.yaml
kubectl get persistentvolumes
# or the shorthand
kubectl get pv
```

> [!NOTE]
> `persistentVolumeReclaimPolicy` is set on the PV (not the PVC) and controls what happens to the PV after its bound PVC is deleted.

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Storage/Persistent-Volume-Claims/page
# PVC
### How binding works
A PVC is bound to a single PV. During binding, Kubernetes searches for a PV that:
- Has sufficient capacity for the requested storage.
- Supports the requested `accessModes`.
- Matches other requested properties such as `volumeMode` and `storageClass`.

<span style="color: #22C55E;">If multiple PVs match, Kubernetes picks one automatically. If you must bind a PVC to a specific PV, use labels on the PV and a selector on the PVC. </span>

Kubernetes will bind a smaller claim to a larger PV if all other criteria match and there are no better-fitting PVs. <span style="color: #22C55E;">There is a strict one-to-one relationship between a bound PVC and a PV — remaining capacity on a PV is not shared with other claims.</span>

If no suitable PV exists, the PVC remains in the `Pending` state until a matching PV is created or dynamically provisioned.

PV (metadata.labels) example:
```yaml
metadata:
  labels:
    name: my-pv
```

PVC (spec.selector) example:
```yaml
spec:
  selector:
    matchLabels:
      name: my-pv
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

```bash
kubectl create -f pvc-definition.yaml
kubectl get pvc
# or target the claim directly:
kubectl get pvc myclaim

kubectl delete pvc myclaim
```

### Deleting a PVC and reclaim policy behavior
What happens to the underlying PV depends on its `persistentVolumeReclaimPolicy`. Common policies:

| Reclaim Policy | What happens                                                                                      | Notes                                                                                                                                                           |
| -------------- | ------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Retain`       | PV object and the underlying storage remain; PV is not automatically available for new claims.    | Manual administrator intervention is required to reuse or delete the volume. <span style="color: #22C55E;">This is the safe default for many static PVs.</span> |
| `Delete`       | The underlying storage (cloud volume or provisioned resource) is deleted when the PVC is deleted. | Supported when the <span style="color: #22C55E;">PV was dynamically provisioned and the provisioner supports deletion (e.g., via CSI).</span>                   |
| `Recycle`      | Attempted simple file-level scrub (deprecated).                                                   | Deprecated — did not provide secure/portable cleanup across volume types.                                                                                       |
> [!NOTE]
> Recycle is deprecated. Use dynamic provisioning with StorageClasses and CSI drivers for provider-supported lifecycle and cleanup operations instead of relying on the old recycle behavior.
### Why `recycle` was deprecated
- The old recycler performed a best-effort file delete and could not guarantee secure erasure.
- It did not handle snapshots, provider metadata, or encrypted volumes correctly.
- It worked only for some in-tree volume plugins.
- Proper cleanup typically requires unmount/detach, filesystem reformat, snapshot handling, or provider-level deletes — operations the recycler did not perform reliably.
## Recommended modern approach
Use StorageClasses and CSI drivers for dynamic provisioning and lifecycle management:
- StorageClasses let Kubernetes dynamically create PVs with specified parameters.
- CSI drivers (or cloud provisioners) handle creation, deletion, and advanced lifecycle operations such as snapshots, encryption, and resizing.
- Dynamic provisioning avoids manual PV creation and reduces administrative overhead.
## Quick reference & next steps
- Use a selector if you must bind a PVC to a specific PV by label.
- Prefer dynamic provisioning via StorageClass + CSI for cloud or production workloads.
- Inspect `kubectl get pv,pvc` and `kubectl describe pv <name>` / `kubectl describe pvc <name>` when troubleshooting binding problems.

[[Storage Class]]
