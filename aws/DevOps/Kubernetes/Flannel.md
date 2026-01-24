![[Pasted image 20260124125015.png]]
![[Pasted image 20260124125049.png]]![[Pasted image 20260124125125.png]]
https://medium.com/@extio/understanding-kubernetes-node-to-node-communication-a-deep-dive-e1d6a5ff87f3
Kubernetes clusters require to allocate non-overlapping **IP addresses for Pods, Services and Nodes**, from a range of available addresses configured in the following components:

- The network plugin is configured to assign IP addresses to Pods.
- The kube-apiserver is configured to assign IP addresses to Services.
- The kubelet or the cloud-controller-manager is configured to assign IP addresses to Nodes.
![[Pasted image 20260124173533.png]]

