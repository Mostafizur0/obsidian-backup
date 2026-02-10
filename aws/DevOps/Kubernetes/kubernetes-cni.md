It allows **each pod** to receive a **unique IP address**, enabling direct communication between pods without the need for network address translation (NAT)

**Static pods** as well as pods running as part of a **DaemonSet** will have IP address of the node they are running on, so they don't rely on CNI in that sense. pods having `hostNetworking: true` in their manifest does not make use of CNI and use host interfaces directly. Think about it: if CNI installation requires use of `kubectl`, it wouldn't be possible without e.g. API server up and running.
https://www.tigera.io/learn/guides/kubernetes-networking/kubernetes-cni/
https://www.plural.sh/blog/kubernetes-cni-guide/
https://serverfault.com/questions/1156373/why-does-kubeproxy-apiserver-and-etcd-not-need-cni-plugins-to-start
[[Overview of Kubernetes CNI Network Models]]

# Calico

# [[Flannel]]
