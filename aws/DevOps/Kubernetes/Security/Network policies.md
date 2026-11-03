[[DevOps/Kubernetes/Security/Security|Security]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/Network-Policies/page
# Ingress and Egress
Important: there are two traffic directions for any pod:
- Ingress — traffic arriving at the pod (for example, a user request to the web server).
- Egress — traffic that originates from the pod (for example, the web server calling the API).

Always determine ingress/egress based on where the traffic originated. Response packets do not change the original direction classification.
![[Pasted image 20261103124832.png|244]]    ![[Pasted image 20261103125146.png|430]]

To find out Ingress/ Egress follow the actual request flow. Is the request going in / goin out from a component in the order of request flow?

Kubernetes cluster networking basics:
- Each node, pod, and Service has an IP address.
- The CNI (Container Network Interface) must enable <span style="color: #22C55E;">pod-to-pod connectivity across nodes without manual routing.</span>
- By default, Kubernetes permits all pod-to-pod and pod-to-Service traffic (an “all-allow” baseline). NetworkPolicy resources allow you to restrict that default.

Why use NetworkPolicy?
- Enforce least privilege between workloads (for example, prevent frontend pods from talking directly to DB pods).
- Reduce attack surface by limiting which peers and ports can access a pod.
- Split trust boundaries between namespaces and services.
# Network policies
- If a NetworkPolicy selects a pod and declares `Ingress` rules, any ingress not explicitly allowed by that policy is denied (default deny for ingress). Egress remains unaffected unless `Egress` rules exist or another policy controls egress for that pod.
- Policies are additive: multiple policies selecting the same pod <span style="color: #22C55E;">combine to form the union of allowed traffic.</span>
- NetworkPolicy resources only express allowed traffic — they are <span style="color: #22C55E;">allow-lists, not deny-lists.</span>

![[Pasted image 20261103132336.png|461]]

- Use pod labels to target pods (for example `role: db` for database pods and `name: api-pod` for API pods).
- Use `policyTypes` to indicate which directions the policy governs.
Complete `NetworkPolicy` YAML that allows only pods labeled `name: api-pod` to connect to pods labeled `role: db` on TCP port 3306:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
```

>[!NOTE]
>NetworkPolicies are enforced by the cluster CNI plugin. Creating a NetworkPolicy resource only defines desired rules — your CNI must implement enforcement for the policy to take effect.
>
>Not every CNI supports NetworkPolicy enforcement. CNIs commonly supporting NetworkPolicy include <span style="color: #22C55E;">Calico, Cilium, Kube-router, Romana, and Weave Net</span>. A basic Flannel setup, for example, may not enforce NetworkPolicies. Always consult your CNI provider documentation for current support details.
>
> <span style="color: #22C55E;">NetworkPolicy objects are namespaced and affect only pods in their namespace unless you use `namespaceSelector`.</span>
> 
> You can create and apply a `NetworkPolicy` YAML file on any cluster, even if the underlying CNI does not support it. <span style="color: #22C55E;">The resource will be accepted without error, resulting in a silent failure where traffic remains unblocked.</span>
> 
> Because policies are additive and depend on the CNI's data plane translation, you must verify behavior using active commands (such as `kubectl exec` with `curl` or `nc` tests) rather than assuming the manifest took effect. 

CNI enforcement examples

| CNI             | Typical NetworkPolicy support          |
| --------------- | -------------------------------------- |
| Calico          | Full support, advanced policy features |
| Cilium          | Full support, BPF-based enforcement    |
| Weave Net       | Supports NetworkPolicy                 |
| Kube-router     | Supports NetworkPolicy                 |
| Flannel (basic) | May not enforce NetworkPolicy          |
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/Developing-Network-Policies/page
# Edge Cases

### Ingress rules

<span style="color: #22C55E;">All rules in from section are combined with logical or. So here pods with matching selector or all pods in prod ns or request from the cider block are allowed.</span>

![[Pasted image 20261103141247.png]]

Ingress-only example
- DB Pods labeled `role: db` are allowed incoming TCP traffic on port 3306 only from:
    - API Pods labeled `role: api` in the `prod` namespace, OR
    - the external IP `192.168.5.10`
<span style="color: #22C55E;">On the other hand rules defined in each -from array element forms an and relationship. So for the 1st rule the pod must have `api-prod` label and must be in prod ns to request to the db.</span>

![[Pasted image 20261103141624.png]]

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    # Entry 1: both podSelector AND namespaceSelector must match (AND)
    - podSelector:
        matchLabels:
          role: api
      namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: prod
    # Entry 2: allow from a specific external IP (OR)
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 3306
```

### Egress rules
Combined ingress + egress example
- Allow API Pods to connect to DB on 3306/TCP.
- Allow DB Pod to reach an external backup server at `192.168.5.10:80`.
![[Pasted image 20261103140546.png|286]]

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: api
      namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: prod
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 80
```
