##### Type-1: 
Also known as bare-metal hypervisors, these run directly on the host's hardware. They provide better performance and security.
Microsoft Hyper-V, VMware ESXi, KVM
##### Type-2:
Known as hosted hypervisors, these run on top of a conventional operating system. They are easier to set up but may have lower performance.
VMware Workstation, Oracle VirtualBox, WSL
> [!NOTE]
> To share the host network in WSL need to set mirror network
> https://learn.microsoft.com/en-us/windows/wsl/networking#mirrored-mode-networking

![[Pasted image 20260116110154.png | right | 300 ]]
### Hyper-v (Windows)
1. Enable Hyper-V from Windows Features
2. Resters the pc
3. Open Hyper-V Manager
4. Create VM from this window
5. Create External Virtual Switch
6. Create a new virtual machine from new dropdown and create a generation 1 VM with created external switch (can not use default switch)

#### Virtual Network Type
Internal: Connect inside the PC
External: Connect with internet
Private: Connect with no other 

#### Dynamic IP vs Static IP
Hyper-v first gets dynamic ip then convert them to static for k8s
Setup static ip in ubuntu server netplan file. It manages network config for ubuntu 24.04.
![[Pasted image 20260124115459.png]]
#### Logical Volume Management(LVM)
LVM (Logical Volume Manager) in Linux is a flexible storage management system that adds a layer of abstraction between physical disks/partitions and the file system, allowing you to pool storage into Volume Groups (VGs) and carve out resizable Logical Volumes (LVs), making storage easier to manage, resize, and migrate than traditional partitions.
Keep only root and boot volume for K8s. Boot partition (carnal) always will be raw disk (not logical) (can be 200/300 mb). If we put root in lvm we can extend the volume if necessary, lvm can add raw disk as logical also without any partition.
![[Pasted image 20260124103428.png | right | 300 ]]  ![[Pasted image 20260124103701.png | left | 300 ]]
A Logical Unit Number (LUN) is a unique identifier used in SCSI-based storage systems, such as Storage Area Networks (SANs), to designate specific, addressable logical storage volumes. It allows a host server to treat a partitioned portion of a physical disk, or a group of disks, as a distinct, usable storage device for data I/O operations. AWS provides LUN.

No swap volume should be added for k8s, as it can not work with swap volume (disk converted into temp ram)

Tic mark install openssh server to enable ssh to the server

#### DNS server
/etc/hosts - works as local dns in linux
Node first search in this file for dns resolution, if not found then it goes to router gateway
