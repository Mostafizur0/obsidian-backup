[[Docker]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Storage/Storage-in-Docker/page
Docker images are constructed from immutable, read-only layers. Each instruction in a `Dockerfile` typically produces <span style="color: #22C55E;">a new image layer that stores only the differences introduced by that instruction.</span>
When the lower layers match, Docker reuses cached layers and only creates new layers for the differing instructions. This saves time and disk space.
![[Screenshot 2026-11-04 at 1.07.48 am.png]]
# Image layers are immutable; containers add a writable layer
<span style="color: #22C55E;">All image layers are read-only.</span> When you run a container (for example `docker run mmumshad/my-custom-app`), Docker creates a thin writable layer on top of the image layers. The container stores runtime changes—logs, temporary files, files created or modified by processes—inside that writable layer.
- The writable layer is ephemeral: when you remove the container, that layer (and its changes) are removed.
- The underlying image remains unchanged and reusable by other containers.
![[Pasted image 20261104011613.png|377]]
#### Copy-on-write model
If a container modifies a file that originally came from a read-only image layer, Docker uses copy-on-write: it first copies the file into the container’s writable layer, then applies the modification. This preserves the original image artifact.
![[Pasted image 20261104011841.png|377]]
Do not rely on a container’s writable layer for persistent data. Removing the container deletes that writable layer and any data it contains.
