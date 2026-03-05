This is a great "what-if" scenario that often confuses people. The short answer: **The Pod keeps running.**

In Kubernetes, the Control Plane is the "Brain," but the Worker Node (via the **Kubelet**) is the "Muscle." If the connection to the brain is severed, the muscle carries out its last known orders.

Here is the step-by-step breakdown of what happens:

https://stackoverflow.com/a/74789627
https://www.howtogeek.com/devops/what-happens-during-a-kubernetes-control-plane-failure/
---

### 1. The Pod’s Immediate State

The container itself doesn't "talk" to the API server for its basic survival. As long as the **Kubelet** (the agent on the node) is healthy and the container process hasn't crashed, the Pod continues to work exactly as it was.

- **Traffic:** If a Service is sending traffic to that Pod, the traffic will continue to flow because the `kube-proxy` rules (IPTables/IPVS) are already programmed into the node's kernel.
    
- **Logic:** The application inside the Pod stays active.
    

### 2. The Control Plane’s Perspective

After a certain timeout (default is **40 seconds**), the Control Plane notices the Node is no longer sending "heartbeats" (NodeLease).

- The Node status changes to **`NotReady`**.
    
- The Controller Manager waits for a grace period (default **5 minutes**, governed by the `--pod-eviction-timeout` flag).
    

### 3. The "Eviction" Phase

If the connection isn't restored within that 5-minute window:

- The Control Plane marks the Pods on that disconnected node for **Termination**.
    
- **Crucial Point:** Since the Control Plane can't talk to the node, it **cannot actually kill** the Pods.
    
- Instead, the Control Plane starts "Replacement" Pods on other **Healthy** nodes to maintain your desired replica count.
    

---

### 4. What happens when the connection returns?

When the "lost" node finally reconnects to the API server:

1. The Kubelet checks in and sees that its Pods have been marked for deletion in the API database.
    
2. The Kubelet kills the local containers.
    
3. The "Replacement" Pods created earlier on other nodes continue to run.
    

---

### Summary Table: Impact of Connection Loss

|**Component**|**Status during disconnect**|
|---|---|
|**Existing Pods**|**Running.** They stay up until the connection returns or the node is wiped.|
|**New Pods**|**Pending.** No new pods can be scheduled to that specific node.|
|**Service Traffic**|**Active.** Internal node rules keep routing traffic until the API server updates the Endpoint list.|
|**`kubectl` commands**|**Failing.** You won't be able to `get logs` or `exec` into pods on that node.|

### The "Split Brain" Risk

The biggest danger here is a **StatefulSet**. If a Pod is writing to a database and loses connection, and K8s starts a "replacement" Pod elsewhere, you could have two pods writing to the same storage simultaneously. This is why Kubernetes is very conservative about "force deleting" pods on unresponsive nodes.

![[Pasted image 20260305235850.png]]
![[Pasted image 20260306000044.png]]
