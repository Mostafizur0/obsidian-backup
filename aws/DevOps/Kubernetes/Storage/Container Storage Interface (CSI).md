[[Storage]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Storage/Container-Storage-Interface/page
The Container Storage Interface (CSI) standardizes how container orchestrators (like Kubernetes) interact with external storage systems. CSI decouples storage plugins from the core orchestrator code so vendors can write a single driver that works across multiple platforms.

CSI follows the same pluggable pattern for storage. <span style="color: #22C55E;">It defines a vendor-neutral RPC (gRPC) API that orchestrators call and that CSI drivers implement.</span> Because CSI is not Kubernetes-specific, a single driver implementation can support Kubernetes, Cloud Foundry, Mesos, and other orchestrators that implement the CSI spec. <span style="color: #22C55E;">The CSI spec defines the exact RPC signatures, expected responses, error codes, and versioning rules to ensure reliable interoperability between orchestrators and drivers.</span>
![[Pasted image 20261104020323.png]]
Common CSI drivers available from storage vendors include Portworx, Amazon EBS, Azure Disk, Dell EMC, NetApp, Pure Storage, HPE, Nutanix, and many others. Because CSI is an open spec, vendors can expose the same capabilities across multiple orchestration platforms.
# CSI RPCs and the Volume Lifecycle
The CSI specification defines two logical groups of RPCs: <span style="color: #22C55E;">controller-side (cluster-level) and node-side (per-node) calls.</span> The orchestrator (for example, Kubernetes) invokes these RPCs to provision, attach, mount, and delete volumes. Key controller RPCs:
- `CreateVolume` — provision a new backend volume
- `DeleteVolume` — remove a backend volume
- `ControllerPublishVolume` — controller-side attach (if the backend requires it)
- `ControllerUnpublishVolume` — controller-side detach

Key node RPCs:
- `NodeStageVolume` — prepare a volume on the node (optional staging mount)
- `NodePublishVolume` — make the staged volume available to the container (bind-mount, mount)
- `NodeUnpublishVolume` — unmount from container path
- `NodeUnstageVolume` — undo staging on the node (if staged)

Table: Common CSI RPCs and their responsibilities

|RPC|Scope|Purpose|
|---|---|---|
|`CreateVolume`|Controller|Provision storage on the backend (capacity, parameters, volume name)|
|`DeleteVolume`|Controller|Delete the backend volume|
|`ControllerPublishVolume`|Controller|Attach the volume to a specific node (controller-side attach)|
|`ControllerUnpublishVolume`|Controller|Detach the volume from a node|
|`NodeStageVolume`|Node|Perform node-level setup (format, mount to a staging path)|
|`NodePublishVolume`|Node|Bind or mount the staged volume into the Pod’s filesystem|
|`NodeUnpublishVolume`|Node|Unmount from the Pod’s path|
|`NodeUnstageVolume`|Node|Cleanup staging (unmount, remove temp dirs)|

Typical call sequence when a Pod requests a PersistentVolume (simplified):
1. If dynamic provisioning is needed: orchestrator calls `CreateVolume`.
2. If attach is required by the backend: `ControllerPublishVolume`.
3. On the target node: `NodeStageVolume` (optional).
4. On the target node: `NodePublishVolume`.
5. When Pod terminates or PVC deleted: `NodeUnpublishVolume`, `NodeUnstageVolume`, `ControllerUnpublishVolume`, `DeleteVolume` (depending on reclaim policy).

![A dark-themed diagram of the Container Storage Interface (CSI) showing orchestration platforms (Kubernetes, Cloud Foundry, Mesos) on the left, storage providers (Portworx, Amazon EBS, Dell EMC, GlusterFS, etc.) on the right, and RPC controller calls (CreateVolume, DeleteVolume, ControllerPublishVolume) in the center. Checklist items on each side list expected "SHOULD" behaviors for provisioning, deleting, and attaching volumes.](https://mintcdn.com/kodekloud-c4ac6d9a/tjRpKfuaPvPcAVKO/images/Certified-Kubernetes-Administrator-CKA/Storage/Container-Storage-Interface/csi-controller-rpc-orchestrators-storage-providers.jpg?fit=max&auto=format&n=tjRpKfuaPvPcAVKO&q=85&s=025a164119e407d01490b547fde47a5b)
