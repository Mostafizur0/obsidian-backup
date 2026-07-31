System pods always get the highest priority
![[Pasted image 20260801012359.png]]

List priority classes
![[Pasted image 20260801012559.png]]
# Create and use priority class
![[Pasted image 20260801012942.png]]

A [Kubernetes PriorityClass](https://kubernetes.io/docs/reference/kubernetes-api/scheduling/priority-class-v1/) with `globalDefault: true` sets the fallback priority value for any pod created without an explicit `priorityClassName`. ==Without a global default, these pods receive a default priority of== `0`. Only one object can hold this status.

![[Pasted image 20260801013448.png]]

https://www.plural.sh/blog/pod-priority-and-preemption/
This value will cause lower priority pods to terminate to give place for higher priority pods if needed.
![[Pasted image 20260801013637.png|391]]
The never value will not cause the lower priority pods to be terminated, instead it will wait in the scheduling queue. However, this pod will get priority during new pod scheduling time if enough resource available.
![[Pasted image 20260801013837.png|397]]
