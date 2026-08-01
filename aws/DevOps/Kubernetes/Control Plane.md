[[Kubernetes]]

https://www.wiz.io/academy/container-security/kubernetes-control-plane
The Kubernetes control plane is the cluster’s management layer that exposes the API, stores cluster state, and continuously reconciles desired configuration—scheduling, scaling, and replacing pods as needed—to keep applications healthy and consistent across nodes.
![[Pasted image 20260728183422.png]]

## Core components of the Kubernetes control plane

The control plane isn't just one piece of software—it's made up of several components that work together. Each component has a specific job, and they all communicate with each other to keep your cluster running smoothly.

### kube-apiserver

The API server is like the front desk of a hotel—everything goes through it. When you use kubectl commands or when other parts of Kubernetes need to do something, they all talk to the API server first. It checks if you're allowed to do what you're asking for, then either approves or rejects your request (<span style="color: green;">authenticate</span>).

The API server also <span style="color: green;">validates everything</span> you send to make sure it makes sense. If you try to create a pod with invalid settings, the API server catches this before it causes problems in your cluster.

![Image](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqp7yd6cdphl0qnp2r8fs.gif)

![[Pasted image 20260728190954.png]]

![[Pasted image 20260728191745.png]]
![[Pasted image 20260306021256.png]]

![[Pasted image 20260306021633.png]]

### etcd
[[Etcd]]
etcd is your cluster's memory bank where all important information gets stored. It remembers every configuration you've made, all your secrets, and the current state of everything in your cluster. Think of it like a super-reliable filing cabinet that never loses anything.

This database uses special technology called the Raft consensus algorithm to stay consistent even when multiple copies exist. If one copy gets damaged, the others can take over without losing any data.

<span style="color: green;">advertise-client-urls</span> is the address on which etcd listens. Normally it is the IP of the server and on port <span style="color: green;">2379</span>, which is the default port on which etcd listens.

![[c1f7ba2320f21a400298fd41e2c7ce4e.svg]]

![[Pasted image 20260306014911.png]]

![[Pasted image 20260306022130.png]]

![[Pasted image 20260728193544.png]]

https://smcgown.com/blog/kubernetes/1-3-etcd-clusters/#managing-etcd-data
The directory structure in ETCD starts with the root as the registry. It contains values for Kubernetes constructs such as minions, pods, replicasets, deployments, roles, and secrets.
/
└── registry
    └── minions
    │   ├── node1
    │   ├── node2
    │   └── node3
    ├── pods
    │   └── namespace1
    │   │   ├── pod1
    │   │   └── pod2
    │   └── namespace2
    │   │   ├── pod1
    │   │   └── pod2
    ├── replicasets
    │   └── namespace1
    │   │   ├── replicaset1
    │   │   └── replicaset2
    │   └── namespace2
    │   │   ├── replicaset1
    │   │   └── replicaset2
    │   ├── namespace1
    │   │   ├── deployment1
    │   │   └── deployment2
    │   └── namespace2
    │       ├── deployment1
    │       └── deployment2
    ...

### kube-scheduler

The scheduler is like a smart assistant that decides where to place your applications. When you create a new pod, the scheduler looks at all your worker nodes and picks the best one based on available resources, special requirements, and other rules you've set up. Scheduler <span style="color: yellow;">ranks</span> the nodes to identify the best fit for the pod. It uses a <span style="color: green;">priority function to assign a score to the nodes on a scale of 0 to 10</span>. For example, the scheduler calculates the amount of resources that would be

It considers things like:

- How much CPU and memory each node has available
- Whether your pod needs to run on specific types of nodes
- If your pod should be close to certain data or away from other pods
- resource requirements, limits, taints and tolerations, node selectors, affinity rules, etc.

### kube-controller-manager

The controller manager runs several smaller programs called controllers that each watch for specific problems and fix them. It's like having multiple specialized maintenance workers who each focus on different parts of your building. In Kubernetes terms, a controller is a process that <span style="color: green;">continuously monitors the state of various components within the system, and works towards bringing the whole system to the desired functioning state.</span>

These controllers include:

- **Node Controller:** Watches for nodes that stop working and marks them as unhealthy
    
- **ReplicaSet Controller:** Ensures the desired number of pod replicas are running at all times (typically managed through Deployments rather than directly)
    
- **Endpoints Controller:** Keeps track of which pods are available to receive traffic
    
- **Service Account Controller:** Creates default accounts for new namespaces
    

### cloud-controller-manager

When you run Kubernetes in the cloud, this component talks to your cloud provider's services. It handles cloud-specific tasks like creating load balancers, setting up storage volumes, and configuring network routes.

This separation keeps Kubernetes flexible—it can work with any cloud provider without needing to know the specific details of each one.

## Control plane vs data plane architecture

Kubernetes splits its work between two main areas: the control plane and the data plane. Understanding this split helps you see how Kubernetes organizes itself and where security matters most.

The control plane makes all the management decisions but doesn't run your actual applications. It's like the management office of a factory—it plans what should happen and gives instructions, but it doesn't operate the machines that make products.

The [data plane](https://www.wiz.io/blog/kubernetes-data-plane) consists of worker nodes that actually run your containers and applications. These nodes contain three main components:

- **Kubelet:** Acts like a local manager on each node, making sure containers start and stay healthy
    
- [**Container runtime**](https://www.wiz.io/academy/container-runtimes)**:** The software that actually runs your containers (like Docker or containerd)
    
- **Kube-proxy:** The service cannot join the pod network because the service is not an actual thing. It is not a container like pods, so it doesn't have any interfaces or an actively listening process. It is a virtual component that only lives in the Kubernetes memory. But then we also said that the service should be accessible across the cluster from any notes. So how is that achieved? That's where Kube Proxy comes in. Implements Service routing on each node using iptables or IPVS rules (note: some modern CNIs like Cilium replace kube-proxy with eBPF-based data paths for better performance). Kube Proxy is a process that runs on each node in the Kubernetes cluster. Its job is to look for new services, and every time a new service is created, it creates the appropriate rules on each node to forward traffic to those services, to the backend pods. One way it does this is using IP tables rules. In this case, it creates an IP tables rule on each node in the cluster to forward traffic heading to the IP of the service, which is 10.96.0.12, to the IP of the actual pod, which is 10.32.0.15. So that's how Kube Proxy configures a service.
    

The control plane and data plane communicate through the API server. Worker nodes regularly check in to get new instructions and report back on what's happening. This design means existing workloads continue running if the control plane becomes temporarily unavailable, but new pods won't be scheduled and configuration changes won't apply until the control plane recovers.
