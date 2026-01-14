https://github.com/hashicorp/memberlist
## Local Kubernetes cluster
https://felipetrindade.com/kubernetes-ingress-load-balancer/
https://github.com/chipmk/docker-mac-net-connect
```
brew install chipmk/tap/docker-mac-net-connect
sudo brew services start chipmk/tap/docker-mac-net-connect
```
## Connect From MAC 
https://www.youtube.com/watch?v=KBqfsfRDv8E
## Master Node Creation
```
sudo apt update && sudo apt upgrade -y
```

Disable swap
```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
> [!NOTE]
> sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab - not working, if swap line is not commented out in /etc/fstab file; comment out swap line manually otherwise next reboot will revert the changes

Load necessary kernel modules
> [!NOTE]
> Kernel modules are pieces of code that can be loaded into the Linux kernel to extend its functionality
> overlay - An overlay kernel module allows you to create a writable overlay filesystem on top of an existing filesystem, enabling modifications without altering the original data.
> https://influentcoder.com/posts/overlayfs/
> br_netfilter - br_netfilter plays a critical role in this network management by enabling packet filtering and network policy enforcement on bridged networks. br_netfilter ensures network policies can be enforced on all traffic passing through bridge devices.

```
sudo modprobe overlay
sudo modprobe br_netfilter
```

```
```