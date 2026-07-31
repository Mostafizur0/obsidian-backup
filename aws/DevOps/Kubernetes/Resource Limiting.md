[[Kubernetes]]
# Resources
![[Pasted image 20260731143231.png]]
Request == guarantee
Limit == max
# Limit Range
Set request and limit for all pods in a namespace in the pod creation time (will not affect any running pods)
![[Pasted image 20260731144507.png]]
# Resource Quotas
Set limit and request on aggregated resources of a namespace
![[Pasted image 20260731144720.png]]

In Kubernetes, **LimitRange** and **ResourceQuota** are both namespace-scoped tools used to manage compute resources, but they operate at different scales: ==**ResourceQuota** caps aggregate consumption for the entire namespace, while **LimitRange** enforces constraints and default values on individual pods and containers==.
