[[Kubernetes]]

https://notes.kodekloud.com/docs/Kubernetes-and-Cloud-Native-Associate-KCNA/Scheduling/Static-Pods/page
Location of static pod folder config: /var/lib/kubelet/config.yaml (kubelet config file path)
![[Pasted image 20260801000833.png]]

Get static pods
![[Pasted image 20260801001602.png]]
Or on master node run (kubelet creates a read only mirror object in etcd)
![[Pasted image 20260801001829.png]]

If any of the static pod crashes it is restarted automatically by the kubelet.

![[Pasted image 20260801002304.png]]
