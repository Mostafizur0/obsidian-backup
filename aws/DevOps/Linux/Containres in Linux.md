## **From Embedded to Cloud: Why Linux Scales**

- Modular kernel: add or remove features via modules and configuration
- Broad hardware support: ARM, x86_64, RISC-V, and more
- Efficient networking stack with eBPF/XDP for high throughput
- **Container-native**: namespaces, cgroups, and overlay filesystems underpin Docker and Kubernetes

That flexibility makes [Linux the de facto choice for servers](https://www.youstable.com/blog/configure-ci-cd-on-linux/), appliances, smartphones, and supercomputers alike.

**Containers are VMs**: Containers share the host kernel; VMs emulate hardware and run their own kernel.

### **How do containers work on Linux?**

Containers use namespaces to isolate resources (PID, NET, MNT, etc.) and cgroups to control resource usage. They share the host kernel but have separate views of filesystems, processes, and networks, enabling lightweight isolation compared to VMs.

