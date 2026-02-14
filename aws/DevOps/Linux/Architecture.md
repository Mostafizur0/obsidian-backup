[[Windows Vs Linux]]
https://linuxsimply.com/linux-basics/introduction/architecture-of-linux-operating-system/
![[Pasted image 20260213213107.png]]
https://www.tutorialspoint.com/operating_system/os_linux.htm
![[Pasted image 20260213213732.png]]

https://www.youstable.com/blog/architecture-of-the-linux-operating-system
Core kernel responsibilities include:
- Process management and scheduling (CFS: Completely Fair Scheduler)
- Memory management and virtual memory
- Virtual File System (VFS) layer and filesystem drivers (ext4, XFS, Btrfs)
- Networking stack (TCP/IP, firewalling with netfilter/iptables/nftables)
- Device drivers (storage, network, GPU, USB)
- **Security:** Linux Security Modules (SELinux, AppArmor), capabilities, namespaces, cgroups

### **User Space**
User space runs in ring 3 (unprivileged mode). Applications call into the kernel via system calls, typically through the C standard library (glibc or musl). User space includes **shells** (bash, zsh), daemons (sshd, systemd services), **utilities** (coreutils), and **package managers** (apt, dnf, pacman).

[[Linux Filesystem Hierarchy (FHS)]]

[[Isolation and Resource Control]]

[[Containres in Linux]]

[[Troubleshooting by Layer]]
### **What is the role of systemd in Linux?**

Systemd is the init system and service manager. It starts services in parallel, manages dependencies, handles logging (journald), provides timers and sockets, and centralizes service control via systemctl.

### **Which filesystem should I choose: ext4, XFS, or Btrfs?**

Ext4 is stable and versatile; XFS excels with large files and parallel I/O; Btrfs offers snapshots and checksums with modern features. Choose based on workload: databases often use XFS, general-purpose servers use ext4, and snapshot-heavy systems can benefit from Btrfs.

### **How do system calls work in Linux?**

Applications request kernel services by invoking system calls, typically through the C library. The CPU switches from user mode to kernel mode, executes the syscall handler, and returns a result or [error code](https://www.youstable.com/blog/ssl-error-rx-record-too-long/) to user space.
