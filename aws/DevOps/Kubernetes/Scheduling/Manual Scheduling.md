[[Scheduling]]

![[shedulers-1.gif]]

In summary, you have two options for manually scheduling a pod in Kubernetes:

- **During Pod Creation:** Set the `nodeName` field in your pod’s manifest to assign it directly to a node.
- **For Existing Pods:** Create a Binding object and use a POST request to assign the pod to your desired node.

This approach provides flexibility in environments where automated scheduling may not fit specific use cases.

During pod creation
![[Pasted image 20260730195818.png]]
https://yuminlee2.medium.com/kubernetes-scheduler-f3df777a0df4
https://notes.kodekloud.com/docs/Kubernetes-and-Cloud-Native-Associate-KCNA/Scheduling/Manual-Scheduling/page

After pod creation
![[Pasted image 20260730202053.png]]

![[binding.gif]]
