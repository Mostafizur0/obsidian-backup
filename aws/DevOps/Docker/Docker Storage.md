[[Docker]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Storage/Docker-Storage-Introduction/page
Two core concepts define storage for containers:
- Storage drivers — implement how a <span style="color: #22C55E;">container’s writable layer and image layers are managed on the host filesystem (examples: overlay2, aufs, devicemapper).</span> Storage drivers operate at the container runtime level and determine how container filesystem changes are isolated and persisted on the host.
- Volume driver plugins — provide <span style="color: #22C55E;">integration with external or specialized storage backends (examples: NFS, cloud block storage, third-party plugins).</span> Volume drivers enable containers to use persistent storage that outlives the container process and can be shared, mounted, or managed by <span style="color: #22C55E;">external systems.</span>
Key examples:
- Storage drivers: overlay2, aufs, devicemapper
- Volume plugins/backends: NFS, Amazon EBS, Google Persistent Disk, GlusterFS

To see which storage driver Docker is using:
```bash
docker info
docker info --format '{{json .Driver}}'
```

# Storage drivers
On Linux hosts Docker keeps runtime and image data under `/var/lib/docker`. The exact subfolders depend on the storage driver in use. The table below summarizes the most important directories under `/var/lib/docker` and what they contain.

|Path (under `/var/lib/docker`)|Purpose / Contents|
|---|---|
|`containers`|Per-container runtime state, logs and writable layer metadata.|
|`image`|Image metadata and data blobs used by the storage driver.|
|`volumes`|Data for named Docker volumes (when using the `local` driver): `volumes/<name>/_data`.|
|`aufs` / `overlay2` / `devicemapper` / `btrfs`|Storage-driver specific on-disk data structures and layers.|
|`trust` / `network` / `swarm`|Other Docker subsystems (signatures, network state, Swarm metadata).|

Note: when using the default `local` volume driver, a named volume `data_volume` maps to a host directory such as `/var/lib/docker/volumes/data_volume/_data`.
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Storage/Storage-in-Docker/page
<span style="color: #22C55E;">The storage driver implements the layered filesystem, manages how image layers are stored on disk, and provides the container writable layer with copy-on-write semantics.</span> Common drivers include:

| Storage driver | Typical use / notes                                                                             |
| -------------- | ----------------------------------------------------------------------------------------------- |
| `overlay2`     | Modern default on many Linux distributions; efficient and recommended when supported.           |
| `aufs`         | Historically used on some Ubuntu installations; less common today.                              |
| `devicemapper` | Block-device based driver; can be used in direct-lvm mode for production on some OSes.          |
| `btrfs`        | Uses Btrfs filesystem features; useful where the kernel supports it.                            |
| `vfs`          | A fallback (no copy-on-write); easy to understand but has poor performance for many containers. |

<span style="color: #22C55E;">Docker chooses a default driver based on kernel features and distribution.</span> Different drivers offer different performance and stability characteristics—test and choose the one that matches your workload and host OS.
# Data persistence
To persist container data beyond a container’s lifecycle, use Docker volumes or bind mounts:
- Volume: Managed by Docker and stored under `/var/lib/docker/volumes` for the `local` driver. They are preferred for portability and decoupling from host paths.
- Bind mount: Mounts a specific host directory (for example `/data/mysql`) into the container. Use bind mounts when you need direct access to specific host locations.

Volume mount visual
![[Pasted image 20261104012410.png]]

Bind mount visual added
![[Pasted image 20261104013013.png]]

```bash
# Create a named Docker volume
docker volume create data_volume

# Start a MySQL container using the named volume
docker run -v data_volume:/var/lib/mysql mysql

# If the named volume doesn't exist, Docker will create it automatically:
docker run -v data_volume2:/var/lib/mysql mysql

# Bind-mount a host directory into the container:
docker run -v /data/mysql:/var/lib/mysql mysql
```

<span style="color: #22C55E;">Preferred explicit syntax: `--mount` </span>(clearer for complex or production usage): now way of doing mount
```bash
# Bind mount using --mount (preferred syntax)
docker run \
  --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Storage/Volume-Driver-Plugins-in-Docker/page
# Volume Drivers
<span style="color: #22C55E;">The default volume driver is `local`.</span> Volumes created with the `local` driver store their data on the Docker host, typically under:
- `/var/lib/docker/volumes`
Third-party volume plugins let you provision volumes on external providers such as Azure File Storage, DigitalOcean Block Storage, Google Persistent Disks, NetApp, GlusterFS, Portworx, VMware vSphere, and many others. Some plugins support multiple backends; for example, the RexRay plugin can provision storage on AWS EBS, S3, EMC Isilon / ScaleIO, Google PD, or OpenStack Cinder.

![[Pasted image 20261104014453.png]]

You can use a driver-backed volume by either creating the volume first and then attaching it to a container, or by creating and attaching the volume inline when running the container.
1. Create a volume explicitly, then run the container: 
```bash
# Create a volume named ebs-vol using the RexRay EBS driver
docker volume create --driver rexray/ebs --name ebs-vol

# Run MySQL and mount the volume at /var/lib/mysql
docker run -it \
  --name mysql \
  --mount type=volume,source=ebs-vol,target=/var/lib/mysql \
  mysql:latest
```
2. Create and attach the volume inline with `--mount` (specifying the volume driver): 
```bash
docker run -it \
  --name mysql \
  --mount 'type=volume,source=ebs-vol,target=/var/lib/mysql,volume-driver=rexray/ebs' \
  mysql:latest
```

```bash
# List installed Docker plugins (including volume plugins)
docker plugin ls

# List Docker volumes and the plugin/driver they use
docker volume ls

# Inspect a specific volume to see driver and mountpoint details
docker volume inspect ebs-vol
```
