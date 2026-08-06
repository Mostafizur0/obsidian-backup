[[Kubernetes]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Application-Developer-CKAD/Configuration/ConfigMaps/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Application-Developer-CKAD/Configuration/Solution-ConfigMaps-Optional/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Application-Lifecycle-Management/Secrets/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Application-Lifecycle-Management/Demo-Encrypting-Secret-Data-at-Rest/page

In case of mounting Secrets as files within a Pod. When mounted, ==each key in the Secret becomes a file, and its content is the corresponding decoded value==. For example:
The corresponding Secret definition with properly encoded data:

```
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw=
  DB_User: cm9vdA==
  DB_Password: cGFzd3Jk
```

```
volumes:
  - name: app-secret-volume
    secret:
      secretName: app-secret
```

Listing the mounted volume may reveal files like:

```
ls /opt/app-secret-volumes
# Output: DB_Host  DB_Password  DB_User
```

And to view the database password:

```
cat /opt/app-secret-volumes/DB_Password
# Output: paswrd
```
https://notes.kodekloud.com/docs/Certified-Kubernetes-Application-Developer-CKAD/Configuration/Secrets/page#secrets

# Secret Store CSI Driver Tutorial
https://www.youtube.com/watch?v=MTnQW9MxnRI

# Secrets safety best practices
Secrets are not encrypted, so it is not safer in that sense. However, some best practices around using secrets make it safer. As in best practices like:

- Not checking in secret object definition files to source code repositories.
- [Enabling Encryption at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) for Secrets so they are stored encrypted in ETCD.

Also, the way Kubernetes handles secrets. Such as:

- A secret is only sent to a node if a pod on that node requires it.
    
- Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
    
- Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.
Read about the [protections](https://kubernetes.io/docs/concepts/configuration/secret/#protections) and [risks](https://kubernetes.io/docs/concepts/configuration/secret/#risks) of using secrets [here](https://kubernetes.io/docs/concepts/configuration/secret/#risks).

Having said that, there are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets, and [HashiCorp](https://www.vaultproject.io/)

[[Encrypting Confidential Data at Rest in ETCD]]
