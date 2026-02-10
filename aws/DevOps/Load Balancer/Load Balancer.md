# MetalLB
MetalLB is a load-balancer implementation for bare metal Kubernetes clusters, using standard routing protocols. 
It works in hub and spoke model. controller deployment works as hub and speaker demon set work as spoke. It is a software based LB not a physical LB. It needs an Ip pool to get the IP. 
L2Advertisement.
When an ingress resource is created then only a load balancer resource is created. Until that it remains in listening state.
The implementations of network load balancers that Kubernetes does ship with are all glue code that calls out to various **IaaS platforms (GCP, AWS, Azure…)**. If you’re not running on a supported IaaS platform (GCP, AWS, Azure…), LoadBalancers will remain in the “pending” state indefinitely when created.
Bare-metal cluster operators are left with two lesser tools to bring user traffic into their clusters, “NodePort” and “externalIPs” services. Both of these options have significant downsides for production use.
MetalLB aims to redress this imbalance by offering a **[[Network Load Balancer]] implementation** that integrates with standard network equipment, so that external services on bare-metal clusters also “just work” as much as possible.
https://metallb.io/installation/network-addons/
## Load-balancing behavior
In layer 2 mode, all traffic for a service IP goes to one node. From there, `kube-proxy` spreads the traffic to all the service’s pods.
In that sense, **layer 2 does not implement a load balancer**. Rather, it implements a **failover mechanism** so that a different node can take over should the current leader node fail for some reason.
If the leader node fails for some reason, failover is automatic: the failed node is detected using [memberlist](https://github.com/hashicorp/memberlist), at which point new nodes take over ownership of the IP addresses from the failed node.
https://metallb.io/concepts/layer2/#load-balancing-behavior
https://metallb.io/concepts/layer2/#limitations
https://metallb.io/installation/

## Cilium
