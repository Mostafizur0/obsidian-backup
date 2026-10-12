[[Authorization]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/Role-Based-Access-Controls/page

View the kubeconfig
```
kubectl config view
```

```bash
kubectl get roles
kubectl get rolebindings
kubectl describe role developer
kubectl describe rolebinding devuser-developer-binding
```

### Testing Permissions with kubectl auth
```bash
kubectl auth can-i create deployments
# Output: yes
kubectl auth can-i delete nodes
# Output: no
kubectl auth can-i create deployments --as dev-user
# Output: no
kubectl auth can-i create pods --as dev-user
# Output: yes
```

### Limiting Access to Specific Resources
```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "create", "update"]
  resourceNames: ["blue", "orange"]
```

# Cluster Roles
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/Cluster-Roles/page
![[Pasted image 20261013011353.png]]
To list namespaced and non-namespaced resources, you can use these commands:
```bash
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
```

> [!NOTE]
> It’s important to note that while cluster roles and role bindings are primarily used for cluster-scoped resources, they can also manage access to namespace-scoped resources. When you bind a cluster role that grants permissions on pods, for instance, the user will have access to pods in every namespace—unlike a namespaced role which restricts access to a single namespace.


