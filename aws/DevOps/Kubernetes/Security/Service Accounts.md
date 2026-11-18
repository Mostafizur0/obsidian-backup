[[DevOps/Kubernetes/Security/Security|Security]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/Service-Accounts/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/Lab-Solution-Service-Accounts/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Security-Specialist-CKS/Cluster-Setup-and-Hardening/Service-Accounts/page

There are two main types of accounts in Kubernetes:
- **User Accounts:** Designed for human users like administrators or developers.
- **Service Accounts:** Intended for machine-to-machine interactions or application-specific tasks. For instance, monitoring tools like Prometheus use a service account to query the Kubernetes API for performance metrics, while Jenkins uses one for deploying applications.

```bash
kubectl create serviceaccount dashboard-sa
kubectl get serviceaccount
kubectl describe serviceaccount dashboard-sa
kubectl describe secret dashboard-sa-token-kbbdm // to view the token
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  namespace: default
```

When a service account is created, Kubernetes automatically generates a token and stores it as a Secret object. This token is then used by your application as bearer for requesting to the k8s api (API authentication).
```curl
curl https://192.168.56.70:6443/api -k \
--header "Authorization: Bearer eyJhbgG…"
```

![[Pasted image 20261118030741.png]]
### Automatic Mounting of Service Account Tokens
When deploying third-party applications (such as a custom dashboard or Prometheus) on a Kubernetes cluster, you can have Kubernetes automatically mount the service account token as a volume into the Pod. This token is typically available at the path: `/var/run/secrets/kubernetes.io/serviceaccount`.

![[Pasted image 20261118031734.png|204]]

==Every process running inside a pod needs an identity to communicate with the Kubernetes API server. This ensures pods have immediate, out-of-the-box infrastructure to authenticate their requests.== Every namespace includes a default service account that is automatically injected into Pods. For example, consider the following simple Pod manifest using a custom dashboard image:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  serviceAccountName: dashboard-sa
  containers:
    - name: my-kubernetes-dashboard
      image: my-kubernetes-dashboard
```

```bash
kubectl describe pod my-kubernetes-dashboard
```
Output
```yaml
Name:           my-kubernetes-dashboard
Namespace:      default
Status:         Running
IP:             10.244.0.15
Containers:
  nginx:
    Image:        my-kubernetes-dashboard
    Mounts:       /var/run/secrets/kubernetes.io/serviceaccount from default-token-j4hkv (ro)
Volumes:
  default-token-j4hkv:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-j4hkv
    Optional:    false
```
Inside the Pod, listing the contents of the service account directory shows files such as the `token` file containing the bearer token:
```bash
kubectl exec -it my-kubernetes-dashboard -- ls /var/run/secrets/kubernetes.io/serviceaccount
kubectl exec -it my-kubernetes-dashboard -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
```
If you wish to disable the automatic mounting of the service account token, set `automountServiceAccountToken` to `false` in the Pod specification:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  automountServiceAccountToken: false
  containers:
  - name: my-kubernetes-dashboard
    image: my-kubernetes-dashboard
```

> [!NOTE]
> Prior to Kubernetes v1.22, service account tokens were automatically mounted from Secrets without an expiration date. Starting with v1.22, the TokenRequest API (KEP-1205) was introduced to generate tokens that are audience-bound, time-bound, and object-bound—enhancing security significantly.

Starting with Kubernetes v1.24, Kubernetes no longer automatically creates non-expiring service account tokens stored as Secrets. Instead, after creating a new service account, you must generate a token explicitly with:
```bash
kubectl create token dashboard-sa
kubectl create token dashboard-sa --duration 2h
```

By default all tokens are valid for 1 hour.
If you decide to revert to the old method of manually creating a Secret, you can do so by defining a Secret with the type `kubernetes.io/service-account-token` and adding the appropriate annotation:
```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: dashboard-sa
```
Ensure that the service account exists before creating the Secret to guarantee proper association. However, the recommended practice is to utilize the token request API for better security and token management.

![[Pasted image 20261118050624.png]]

When a pod is created now, Kubernetes communicates with the token controller API to produce a token with a defined lifetime. The following example demonstrates a pod definition using a projected volume:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: default
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          name: kube-api-access-6mtg8
          readOnly: true
  volumes:
    - name: kube-api-access-6mtg8
      projected:
        defaultMode: 420
        sources:
          - serviceAccountToken:
              expirationSeconds: 3607
              path: token
          - configMap:
              name: kube-root-ca.crt
              items:
                - key: ca.crt
                  path: ca.crt
          - downwardAPI:
              items:
                - fieldRef:
                    apiVersion: v1
                    fieldPath: metadata.namespace
```

Prior to these enhancements, the service account token was mounted as a traditional Secret. Today, it is provided as a projected volume that interacts dynamically with the token controller API.

This command outputs a token with an expiry (default is one hour unless configured otherwise). To inspect token details, you can decode it using tools such as JWT.io or by running:
```bash
jq -R 'split(".") | select(length > 0) | .[0] | @base64 | fromjson' <<< eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdC1kYXNob2FyZC1zYSIsImF1ZCI6WyJodHRwczovL2t1YmVybmV0ZXMuZGVmYXVsdC5zdmMuY2x1c3Rlci5sb2NhbCJdLCJleHBpcmF0aW9uIjoxNjY0MDM3NzYzLCJpc3MiOiJodHRwczovL2t1YmVybmV0ZXMuZGVmYXVsdC5zdmMuY2x1c3Rlci5sb2NhbCJ9.k5Y3R-
```
