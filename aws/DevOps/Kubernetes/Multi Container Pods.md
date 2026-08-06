[[Pod]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Application-Developer-CKAD/Multi-Container-Pods/Multi-Container-Pods/page
## Multi-Container Pod Design Patterns (Dev perspective)

There are three common patterns for designing multi-container pods:
1. **Sidecar Pattern**: The sidecar pattern involves deploying a supplemental container, such as a logging agent, alongside your primary container. This design pattern enables services to extend or enhance the capabilities of the main application without altering its code. It is particularly useful when the application produces logs in various formats across different services.![[Pasted image 20260806175734.png]]
2. **Adapter Pattern**: The adapter pattern is useful when you need to standardize data formats. For example, when logs from multiple sources need to be unified before they are processed by a central logging service.
3. **Ambassador Pattern**: The ambassador pattern is applied when an application needs to communicate with different database environments. For example, the application might require a local database for development, another for testing, and a production database in live deployment. Instead of incorporating logic to handle multiple environments in the application code, an ambassador container acts as a proxy. The application always sends requests to localhost, while the ambassador routes traffic to the appropriate database backend.
   ![[Pasted image 20260806175655.png]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Application-Lifecycle-Management/Multi-Container-Pods/page
## Multi-Container Pod Design Patterns (Ops perspective)
![[Pasted image 20260806174125.png]]

![[Pasted image 20260806174305.png]]

![[Pasted image 20260806174423.png]]
For sidecar the init container starts before the main container, runs as long as the main container is alive and exits after the main container is dead.
![[Pasted image 20260806174527.png]]
Example of sidecar pattern
![[Pasted image 20260806174843.png]]
