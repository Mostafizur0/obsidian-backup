https://www.wiz.io/academy/container-security/kubernetes-control-plane
The Kubernetes control plane is the cluster’s management layer that exposes the API, stores cluster state, and continuously reconciles desired configuration—scheduling, scaling, and replacing pods as needed—to keep applications healthy and consistent across nodes.
## Core components of the Kubernetes control plane

The control plane isn't just one piece of software—it's made up of several components that work together. Each component has a specific job, and they all communicate with each other to keep your cluster running smoothly.

### kube-apiserver

The API server is like the front desk of a hotel—everything goes through it. When you use kubectl commands or when other parts of Kubernetes need to do something, they all talk to the API server first. It checks if you're allowed to do what you're asking for, then either approves or rejects your request.

The API server also validates everything you send to make sure it makes sense. If you try to create a pod with invalid settings, the API server catches this before it causes problems in your cluster.
![[Pasted image 20260306021256.png]]

![[Pasted image 20260306021633.png]]

### etcd

etcd is your cluster's memory bank where all important information gets stored. It remembers every configuration you've made, all your secrets, and the current state of everything in your cluster. Think of it like a super-reliable filing cabinet that never loses anything.

This database uses special technology called the Raft consensus algorithm to stay consistent even when multiple copies exist. If one copy gets damaged, the others can take over without losing any data.

![[Pasted image 20260306014911.png]]

![[Pasted image 20260306022130.png]]

### kube-scheduler

The scheduler is like a smart assistant that decides where to place your applications. When you create a new pod, the scheduler looks at all your worker nodes and picks the best one based on available resources, special requirements, and other rules you've set up.

It considers things like:

- How much CPU and memory each node has available
    
- Whether your pod needs to run on specific types of nodes
    
- If your pod should be close to certain data or away from other pods
    

### kube-controller-manager

The controller manager runs several smaller programs called controllers that each watch for specific problems and fix them. It's like having multiple specialized maintenance workers who each focus on different parts of your building.

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
    
- **Kube-proxy:** Implements Service routing on each node using iptables or IPVS rules (note: some modern CNIs like Cilium replace kube-proxy with eBPF-based data paths for better performance)
    

The control plane and data plane communicate through the API server. Worker nodes regularly check in to get new instructions and report back on what's happening. This design means existing workloads continue running if the control plane becomes temporarily unavailable, but new pods won't be scheduled and configuration changes won't apply until the control plane recovers.