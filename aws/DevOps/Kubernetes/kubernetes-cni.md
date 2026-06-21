**Best source (read later)**
https://itnext.io/container-network-interface-cni-in-kubernetes-an-introduction-6cd453b622bd
https://www.linkedin.com/pulse/demystifying-kubernetes-networking-deep-dive-cni-services-dilhan-trhwc
https://www.srodi.com/posts/kubernetes-networking-series-part-2
https://ronaknathani.com/blog/2020/08/how-a-kubernetes-pod-gets-an-ip-address/
![[Pasted image 20260417201749.png]]


It allows **each pod** to receive a **unique IP address**, enabling direct communication between pods without the need for network address translation (NAT)

**Static pods** as well as pods running as part of a **DaemonSet** will have IP address of the node they are running on, so they don't rely on CNI in that sense. pods having `hostNetworking: true` in their manifest does not make use of CNI and use host interfaces directly. Think about it: if CNI installation requires use of `kubectl`, it wouldn't be possible without e.g. API server up and running.
https://www.tigera.io/learn/guides/kubernetes-networking/kubernetes-cni/
https://www.plural.sh/blog/kubernetes-cni-guide/
https://serverfault.com/questions/1156373/why-does-kubeproxy-apiserver-and-etcd-not-need-cni-plugins-to-start
[[Overview of Kubernetes CNI Network Models]]

**How CNI Plugins Work**
In Kubernetes, the **kubelet** (the agent on each node) is responsible for managing the lifecycle of pods. However, the kubelet does not know how to configure networking; it delegates this to a CNI plugin (like Calico, Flannel, or Cilium). 

The Execution Flow:

1. **Pod Scheduled:** A pod is assigned to a node.
2. **Kubelet Call:** The kubelet looks at the local CNI configuration file (usually in `/etc/cni/net.d/`).
3. **Plugin Execution:** The kubelet calls the CNI binary (found in `/opt/cni/bin/`) with the command `ADD`.
4. **Network Setup:** The plugin performs several tasks:
    - Creates a **veth pair** (virtual ethernet cable).
    - Moves one end of the veth pair into the Pod’s network namespace (the "container side").
    - Attaches the other end to a bridge (like `cbr0`) or a routing table on the Host (the "host side").
    - **IPAM (IP Address Management):** The plugin assigns an IP address to the pod from a pre-allocated range (PodCIDR) assigned to that specific node.
5. **Result:** The plugin returns a JSON result to the kubelet confirming the IP address and interface details. 

---

**How Cross-Node Pod Communication Happens**
By default, every pod in a Kubernetes cluster can talk to every other pod without NAT. This is achieved via **East-West networking**. When Pod A on Node 1 wants to talk to Pod B on Node 2, the traffic follows this path: 

Step A: Leaving the Node

1. **Routing Table:** Pod A sends a packet. The Pod's internal routing table sees that the destination IP (Pod B) is outside its local network, so it sends the packet through its `eth0` (the veth pair) to the Host.
2. **Host Decision:** The Host receives the packet. It looks at its own routing table. It sees that the destination IP range belongs to Node 2. 

Step B: Traveling the Physical Network

To get the packet from Node 1 to Node 2, CNI plugins generally use one of two methods: **Encapsulation (Overlay)** or **Direct Routing (L3)**. 

**Method 1: Overlay Networks (VXLAN / UDP)**

- The CNI (like Flannel or Calico in VXLAN mode) wraps the original Pod-to-Pod packet inside a standard UDP/IP packet.
- **Source IP:** Node 1 IP | **Destination IP:** Node 2 IP.
- The physical network (routers/switches) only sees traffic between Node 1 and Node 2.
- When it arrives at Node 2, the CNI "unwraps" the packet and delivers the original inner packet to Pod B. 

**Method 2: Direct Routing (BGP / Host-GW)**

- The CNI (like Calico or Cilium) acts as a router. It tells the physical network or the other nodes: _"To reach the IPs on Node 2, send the traffic to Node 2's physical IP."_
- There is no "wrapping" (no encapsulation overhead). The packet stays exactly as it is. This is faster but requires the underlying network to be "aware" of the pod routes (often via BGP). 

Step C: Arriving at the Destination

1. **Node 2 Receipt:** The packet arrives at Node 2's physical interface.
2. **Local Delivery:** Node 2 sees that the destination IP belongs to Pod B. It forwards the packet through the local bridge or routing table to Pod B’s specific `veth` interface.
3. **Completion:** The packet enters Pod B's namespace. 

---

Summary Table

| Component             | Responsibility                                                                |
| --------------------- | ----------------------------------------------------------------------------- |
| **Kubelet**           | Triggers the CNI when a pod starts/dies.                                      |
| **CNI Binary**        | Configures the interfaces and IP addresses.                                   |
| **IPAM**              | Manages the pool of available IP addresses.                                   |
| **Overlay (VXLAN)**   | Packages pod traffic to "tunnel" through a network it doesn't control.        |
| **Underlay (L3/BGP)** | Routes pod traffic directly across the physical network for high performance. |

# Calico

# [[Flannel]]
