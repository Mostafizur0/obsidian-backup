[[DevOps/Kubernetes/Security/Security|Security]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Security-Specialist-CKS/Cluster-Setup-and-Hardening/Kubernetes-Security-Primitives/page
# Securing the Cluster Hosts
Before diving into the intricacies of Kubernetes security, it is essential to secure the underlying infrastructure. Ensure that all hosts in your Kubernetes cluster are protected by:
- Disabling root access and password-based authentication.
- Enabling SSH key-based authentication.
- Implementing additional security measures to protect the physical or virtual infrastructure hosting Kubernetes.
# Kubernetes API Server: The Entry Point
1. [[DevOps/Kubernetes/Security/Authentication|Authentication]]
2. [[Authorization]]
# Securing Intra-Cluster Communications
A critical element of Kubernetes security involves securing communications between various cluster components. All interactions between components—such as the etcd cluster, kube controller manager, scheduler, API server, and worker node components (including kubelet and kube-proxy)—are protected using TLS encryption.
[[TLS Certificates And Secrets]]
[[Certificate Lifecycle and API]]
# Network Policies Within the Cluster
By default, pods within a Kubernetes cluster can communicate freely with one another. To restrict unwanted access and tighten security, network policies can be implemented. These policies enable you to control traffic flow between pods and are an integral part of securing inter-application communications within the cluster.
[[Network policies]]
