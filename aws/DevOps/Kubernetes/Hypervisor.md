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
