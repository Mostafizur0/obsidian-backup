[[DevOps/Kubernetes/Security/Security|Security]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/API-Groups/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Security-Specialist-CKS/Cluster-Setup-and-Hardening/API-Groups/page#summary

The Kubernetes API forms the foundation for all interactions with a cluster. Whether you use the Kubernetes command-line utility (kubectl) or interact directly via REST, every operation communicates with the API server. For example, to check the version of the Kubernetes API server, you can run the command below to access it on the master node’s default port 6443:
```curl
curl https://kube-master:6443/version
response:
{
  "major": "1",
  "minor": "13",
  "gitVersion": "v1.13.0",
  "gitCommit": "ddf47ac13c1a9483ea035a79cd7c1005ff21a6d",
  "gitTreeState": "clean",
  "buildDate": "2018-12-03T20:56:12Z",
  "goVersion": "go1.11.2",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```
![[Pasted image 20261120213447.png]]

The API is divided into multiple groups based on their function. Some groups are dedicated to core resources—such as pods, namespaces, persistent volumes—while others are logically grouped by functionality, such as metrics, health, and logs.
![[Pasted image 20261120213546.png]]

Kubernetes organizes its API resources into two main groups:
1. **Core API Group:** Contains resources integral to cluster operation
2. **Named API Group:** Organizes newer features and additional functionalities into distinct categories. All new features will be available under this group in future.
![[Pasted image 20261120213849.png]]

All Kubernetes resources are organized into API groups. The core API group contains essential resources for cluster operation, while named API groups provide a structured approach to additional functionalities. Each resource supports actions (verbs) like list, get, create, delete, update, and watch.
![[Pasted image 20261120214148.png]]
For further details on these interactions and available operations, refer to the Kubernetes API reference documentation. For example, the documentation for “Pod v1 core” includes group details and information on API actions.
https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/

To access the api server we need to authenticate except for some endpoint like version.
![[Pasted image 20261120214701.png]]

A better alternative is use kubectl proxy which use kubeconfig file to authenticate.
```bash
kubectl proxy
Starting to serve on 127.0.0.1:8001
```
![[Pasted image 20261120215118.png]]

> [!NOTE]
> Both “kube proxy” and “kubectl proxy” might sound similar but serve different functions:
> - “Kube proxy” manages communication between pods and services across nodes.
> - “Kubectl proxy” is an HTTP proxy used to securely access the Kubernetes API server.

