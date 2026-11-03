[[Kubernetes]]

# Controllers
Each resource type has a controller which monitors changes in the etcd for that resource and take action if anything changes in the etcd related to that resource. When you create a built-in resource such as a Deployment, Kubernetes persists the object in etcd and a controller reconciles the cluster to match the desired state recorded in that object. You can manage these objects with kubectl.
![[Pasted image 20261103175849.png]]
 Controllers are control-plane background processes that watch objects and reconcile actual cluster state to the desired state. For example, the Deployment controller creates a ReplicaSet for the Deployment, and the ReplicaSet creates Pods. <span style="color: #22C55E;">It contains the logic for the functionality of the resource.</span>

![[Pasted image 20261103175014.png]]

Small excerpt of what a controller looks like in Go (for illustration only):
```go
package deployment

var controllerKind = apps.SchemeGroupVersion.WithKind("Deployment")

// Run begins watching and syncing.
func (dc *DeploymentController) Run(workers int, stopCh <-chan struct{})

// Add ReplicaSet
func (dc *DeploymentController) addReplicaSet(obj interface{})
```

If the cluster does not know about the FlightTicket resource type, kubectl will fail with the following error:
```bash
$ kubectl create -f flightticket.yml
error: unable to recognize "flightticket.yml": no matches for kind "FlightTicket" in version "flights.com/v1"
```
# CustomResourceDefinition (CRD)
A CRD is itself an API object (apiVersion: `apiextensions.k8s.io/v1`, kind: `CustomResourceDefinition`) that instructs the API server to accept a new resource type. The CRD defines:

![[Pasted image 20261103181026.png]]