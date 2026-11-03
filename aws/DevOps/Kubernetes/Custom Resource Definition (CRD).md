[[Kubernetes]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/2025-Updates-Custom-Resource-Definition-CRD/page#what-is-a-customresourcedefinition-crd-
# Controllers
Each resource type has a controller which monitors changes in the etcd for that resource and take action if anything changes in the etcd related to that resource. When you create a built-in resource such as a Deployment, Kubernetes persists the object in etcd and a controller reconciles the cluster to match the desired state recorded in that object. You can manage these objects with kubectl.
![[Pasted image 20261103175849.png]]
 Controllers are control-plane background processes that watch objects and reconcile actual cluster state to the desired state. For example, the Deployment controller creates a ReplicaSet for the Deployment, and the ReplicaSet creates Pods. <span style="color: #22C55E;">It contains the logic for the functionality of the resource.</span>

> [!NOTE]
> Creating a CRD registers a new resource type with the Kubernetes API and enables persisted objects of that type. A CRD provides validation and API wiring but does not implement business logic — to make the resource trigger actions (for example, call an external booking API), you must deploy a controller that watches those custom objects and reconciles desired state to actual state.

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

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/2025-Updates-Custom-Controllers/page
# Create custom controller
https://github.com/kubernetes/sample-controller
Example commands and build/run flow:
```bash
$ git clone https://github.com/kubernetes/sample-controller.git
Cloning into 'sample-controller'...
Resolving deltas: 100% (15787/15787), done.

$ cd sample-controller

# Customize controller.go with your custom logic
$ go build -o sample-controller .
go: downloading k8s.io/client-go v0.0.0-20211001003700-dbfa30b9d908
go: downloading golang.org/x/text v0.3.6

$ ./sample-controller -kubeconfig=$HOME/.kube/config
I1013 02:11:07.489479   40117 controller.go:115] Setting up event handlers
I1013 02:11:07.489701   40117 controller.go:156] Starting FlightTicket controller
```
Inside a Go controller you will typically see declarations and function signatures.
When your controller works, package it as a container image and deploy it to the cluster (commonly as a Deployment or DaemonSet). Running controllers inside the cluster simplifies authentication (in-cluster config) and benefits from Kubernetes’ scheduling and lifecycle management.
# CustomResourceDefinition (CRD)
A CRD is itself an API object (apiVersion: `apiextensions.k8s.io/v1`, kind: `CustomResourceDefinition`) that instructs the API server to accept a new resource type. The CRD defines:
![[Pasted image 20261103181633.png]]

![[Pasted image 20261103181026.png]]

Example CRD (key parts):
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flighttickets.flights.com
spec:
  group: flights.com
  scope: Namespaced
  names:
    kind: FlightTicket
    singular: flightticket
    plural: flighttickets
    shortNames:
      - ft
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                from:
                  type: string
                to:
                  type: string
                number:
                  type: integer
                  minimum: 1
                  maximum: 10
```
The `openAPIV3Schema` field lets the API server validate incoming objects. In this example, `from` and `to` are strings and `number` must be an integer between 1 and 10. If a user submits invalid data, the API server will reject the object according to this schema.

Create the CRD and then the custom resource:
```bash
$ kubectl create -f flightticket-crd.yml
customresourcedefinition.apiextensions.k8s.io/flighttickets.flights.com created

$ kubectl create -f flightticket.yml
flightticket "my-flight-ticket" created

$ kubectl get flighttickets
NAME               AGE
my-flight-ticket   24m

$ kubectl delete -f flightticket.yml
flightticket "my-flight-ticket" deleted
```
# Operator Framework
Operators effectively bundle CRDs and controllers into a single, deployable unit so you can deliver application-specific operations in a repeatable, automated way.

An operator packages Kubernetes application knowledge — the human operational procedures required to install, upgrade, back up, and recover an application — into code that runs inside the cluster. Operators combine:
- A CustomResourceDefinition (CRD) that declares a new Kubernetes API kind.
- One or more controllers that watch and reconcile instances of that CRD, implementing domain-specific logic.

Common operator responsibilities:
- Provisioning and configuring application instances (create/update).
- Automated backups and restores.
- Upgrades and schema migrations.
- Failure detection and remediation.
- Exposing operational metrics and status.

Popular operators (available on OperatorHub) include etcd, MySQL, Prometheus, Grafana, Argo CD, and Istio.
![[Pasted image 20261103193116.png]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/2025-Updates-Operator-Framework/page
# Deploying operators from OperatorHub
https://operatorhub.io/
Installment example
```bash
# Install Operator Lifecycle Manager (example version)
$ curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.32.0/install.sh | bash -s v0.32.0

# Install the etcd operator from OperatorHub manifest
$ kubectl create -f https://operatorhub.io/install/etcd.yaml

# Verify ClusterServiceVersions in the target namespace (example namespace)
$ kubectl get csv -n my-etcd
```
