# Demystifying Kubernetes Networking: A Deep Dive into CNI, Services, and Ingress
https://www.linkedin.com/pulse/demystifying-kubernetes-networking-deep-dive-cni-services-dilhan-trhwc

For many developers and operators, Kubernetes networking is a black box. Pods can talk to each other, services are magically discoverable, and traffic from the internet somehow finds its way to the right container. But when things go wrong, or when you need to design a scalable and secure system, that "magic" can become a source of immense frustration.

The goal of this post is to demystify Kubernetes networking by breaking it down into its three core pillars: the Container Network Interface (CNI), Services, and Ingress. By understanding how these layers work together, you can turn networking from a mystery into a powerful tool.

![Article content](https://media.licdn.com/dms/image/v2/D5612AQE1YmnEbXKH3w/article-inline_image-shrink_1500_2232/B56ZisMoPBHkAU-/0/1755235649097?e=2147483647&v=beta&t=6tqvwRQD5r8Y2cYQei-TBd_qiSHv3R5zQftm4KsZb-8)

## 1. The Foundation: How Pods Get an IP Address (CNI)

Before we can talk about anything else, we have to start with the fundamental rule of Kubernetes networking: every pod gets its own unique IP address, and every pod in a cluster can communicate with every other pod directly, without NAT (Network Address Translation).

This is a simple but powerful concept. It means you don't have to worry about mapping ports between containers and hosts, which greatly simplifies application configuration. But how does a pod actually get its IP and become part of the network?

This is the job of the CNI (Container Network Interface). The CNI isn't a single piece of software; it's a specification or a standard that was created by the Cloud Native Computing Foundation (CNCF). It defines a simple contract between the Kubernetes control plane (specifically, the kubelet on each node) and a network plugin.

Think of the CNI specification as the standard for electrical outlets in a house. The standard dictates the shape of the plug and the voltage, ensuring any compliant appliance can get power. The actual wiring in the walls is done by an electrician, who can choose from different types of wires and techniques. In Kubernetes, the "electricians" are the CNI plugins. When a new pod is scheduled onto a node, the kubelet calls the CNI plugin to perform a few key tasks:

1. Allocate an IP Address: The plugin assigns a unique IP address to the pod from a predefined range.
2. Create a Network Interface: It creates a virtual network interface (like a virtual Ethernet port) for the pod inside its isolated network namespace.
3. Connect to the Node Network: It connects the pod's interface to the node's network, typically using a virtual Ethernet pair (veth) and a network bridge.
4. Configure Routes: It ensures the necessary routes are in place so the pod can reach other pods on the same node and on different nodes.

There are many popular CNI plugins, each with different strengths:

- Flannel: Simple and easy to set up, using a VXLAN overlay network.
- Calico: Known for its high performance and robust network policy features, often using BGP.
- Cilium: A modern, powerful CNI that uses eBPF (extended Berkeley Packet Filter) to provide networking, observability, and security with very low overhead.

The CNI is the foundational layer that makes the Kubernetes networking model possible. It gives every pod an IP and ensures they can all talk to each other.

## 2. Container Network Interface, in practice

CNI plugins wire Pod interfaces, assign IPs, and set up routes. Two dominant models:

### A. Overlay or encapsulation

VXLAN or Geneve encapsulates Pod traffic over the node network.

- Pros: Simple underlay requirements.
- Cons: Extra header bytes, MTU tuning matters. Examples: Flannel (VXLAN), Weave Net, Calico in VXLAN mode, Cilium in VXLAN or Geneve.

### B. Native routing

No encapsulation. Nodes learn Pod CIDRs via BGP or host routing.

- Pros: Lower latency and better throughput.
- Cons: Underlay must carry Pod subnets. Examples: Calico with BGP, Cilium with native routing, AWS VPC CNI in subnet mode, Azure CNI.

Quick checks

# Show Pod interfaces and routes inside a Pod
kubectl exec -it pod/x -- ip addr
kubectl exec -it pod/x -- ip route

# See node routes and Pod CIDRs
ip route | grep -E 'cni|flannel|calico|cilium'        

## 3. Abstracting Pods: The Service

So, all pods can talk to each other. Problem solved, right? Not quite. Pods are ephemeral. They can be created, destroyed, restarted, or moved at any time. Every time a pod is recreated, it gets a new IP address.

This creates a huge problem: if you have a frontend application that needs to talk to a backend application, how can it reliably find the backend, if its IP address is constantly changing?

This is where the Service object comes in. A Service provides a stable endpoint for a group of pods. It gets a single, virtual IP address (called the ClusterIP) and a DNS name that do not change for the lifetime of the Service.

Imagine a Service is like the single, public phone number for a company's customer support department. You don't need to know the direct extension of each individual support agent (the pods). You just call the main number (the Service), and the system automatically routes you to an available agent. If one agent goes on break and another one logs in, you don't notice; you just keep calling the same number.

A Service works by using labels and selectors. You assign a label to your pods (e.g., app: backend), and then you define a Service that selects for that label. Kubernetes automatically keeps track of which pods match the selector and are healthy.

The real magic is performed by kube-proxy, a component that runs on every node in the cluster. kube-proxy watches the Kubernetes API server. When it sees a new Service, it programs networking rules (using technologies like iptables or IPVS) on the node itself. These rules intercept any traffic destined for the Service's ClusterIP and forward it to the real IP address of one of the healthy backend pods, performing load balancing in the process.

There are a few main types of Services:

- ClusterIP: (Default) Exposes the service on an internal-only IP. This is for communication within the cluster.
- NodePort: Exposes the service on a static port on each node's IP address. This allows external traffic to reach the service.
- LoadBalancer: The standard way to expose a service to the internet. It provisions an external load balancer from your cloud provider (like an AWS ELB or GCP Cloud Load Balancer) and automatically configures it to send traffic to the NodePort on your nodes.

### How packets get forwarded

- kube-proxy iptables. Adds DNAT rules per Service and Endpoint. Works, but thousands of rules can slow updates.
- kube-proxy IPVS. Kernel load balancer. Better performance with many Services.
- kube-proxy-less with eBPF. Cilium or similar programs lookup directly in the kernel. Less churn and lower latency.
## 4. Managing External Access: The Ingress

The LoadBalancer Service type is great, but it has a major drawback: it typically provisions one (often expensive) cloud load balancer per service. If you have dozens of microservices, this becomes unmanageable and costly.

This is the problem that Ingress solves. An Ingress is not a Service, but an API object that manages external access to the services in a cluster, typically for HTTP and HTTPS traffic. It lets you define routing rules from the outside world to services within the cluster.

Imagine, if a Service is a company's main phone number, an Ingress is the receptionist sitting at the front desk of a large office building. You don't need a separate entrance for every company in the building. Instead, you go to the main lobby (the Ingress), tell the receptionist who you want to see (e.g., by providing a hostname like api.example.com or a path like /login), and they direct you to the correct office (the Service).

An Ingress works with two components:

1. The Ingress Resource: A YAML file where you define the routing rules. You can create rules based on the requested hostname (e.g., billing.example.com -> billing-service) or the URL path (e.g., example.com/api -> api-service).
2. The Ingress Controller: This is the engine that actually implements the rules. The controller is a pod running a reverse proxy (like NGINX, HAProxy, or Traefik) that watches for Ingress resources. When you create or update an Ingress, the controller reconfigures itself to route traffic according to your new rules. The Ingress Controller itself is typically exposed to the internet via a single LoadBalancer Service, making it the one entry point for all your traffic.

Here's a simple Ingress resource that routes traffic based on hostname:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
spec:
  rules:
  - host: "shop.example.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: shop-service
            port:
              number: 80
  - host: "bar.example.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: bar-service
            port:
              number: 80        

This single Ingress setup allows you to manage traffic for two different services through one load balancer, saving cost and simplifying DNS management.

## 5. Day-2 operations and debugging playbook

When traffic drops or latency spikes, run this sequence. It solves most incidents.

- DNS

kubectl exec -it pod/x -- getent hosts shop.example.com        

If DNS fails, check CoreDNS pods and NodeLocal DNSCache.

- Service and endpoints

kubectl get svc web -n shop -o wide
kubectl get endpointslice -n shop -l kubernetes.io/service-name=web -o wide        

Empty endpoints mean labels do not match or readiness probes fail.

- NetworkPolicy

kubectl get networkpolicy -A        

Temporarily allow traffic from the test Pod to confirm a policy block.

- Node reachability

# From Pod to Pod on another node
kubectl exec -it pod/x -- curl -sSf http://<peer-pod-ip>:8080
# From node to Pod
curl -sSf http://<pod-ip>:8080        

Failure hints at CNI routes or MTU problems.

- kube-proxy or eBPF dataplane

1. iptables mode: iptables-save | grep KUBE-.
2. IPVS mode: ipvsadm -Ln.
3. Cilium: cilium service list, cilium bpf lb list, and Hubble flows.

- Ingress or Gateway: Inspect controller logs and the underlying LoadBalancer health. Confirm externalTrafficPolicy settings and node distribution.
- Packet capture

1. On node: tcpdump -i any host <pod-ip> and port 80.
2. With Cilium Hubble: filter by source and destination labels.

## Final Thoughts

Kubernetes networking might seem complex, but it's built on a logical, layered foundation.

- CNI lays the physical groundwork, giving every pod an IP and connecting them.
- Services provide a stable abstraction over ephemeral pods for reliable internal communication.
- Ingress provides a smart, cost-effective way to manage external traffic routing to those services.

By understanding how these three pillars work together, you can move from being intimidated by the "magic" of Kubernetes networking to confidently designing and managing robust, scalable applications in a cloud-native world.