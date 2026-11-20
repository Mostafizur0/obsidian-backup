[[TLS Certificates And Secrets]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Security-Specialist-CKS/Cluster-Setup-and-Hardening/TLS-in-Kubernetes-Certificate-Creation/page
To generate certificates for a Kubernetes cluster we can use tools like EasyRSA, CFSSL and OpenSSL.
# Generating the CA Certificate
Common name must be “KUBERNETES-CA”
```bash
openssl genrsa -out ca.key 2048
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```
# Generating Client Certificates
### Admin User Certificate
Common name can be “kube-admin” or other related name. This naming is essential since it is used within audit logs and other system functions. So CN must be relevant.
```bash
openssl genrsa -out admin.key 2048
openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```
To differentiate admin users from basic users, you can include group details in the CSR by specifying the Organizational Unit (OU). For example, adding the group `system:masters` grants administrative privileges. The k8s has already this group created with admin privileges. So adding to this group will grant admin privileges.
```bash
openssl genrsa -out admin.key 2048
openssl req -new -key admin.key -subj "/CN=kube-admin/OU=system:masters" -out admin.csr
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```
These certificates enable secure authentication with the kube API server, allowing REST API calls that use the key, client certificate, and CA certificate. For example:
```
curl https://kube-apiserver:6443/api/v1/pods \
  --key admin.key --cert admin.crt --cacert ca.crt
```
Most Kubernetes clients consolidate these parameters into a configuration file called KubeConfig, which details the API server endpoint and corresponding certificates.

Both clients and servers must use a shared CA root certificate for secure communication. This mutual trust ensures that the certificates presented by each party are signed by a trusted authority, much like how browsers validate a website’s certificate.
### Kube Scheduler Certificate
Same process as Admin user but the CN=system:kube-scheduler as it is a system component
```bash
openssl genrsa -out scheduler.key 2048
openssl req -new -key scheduler.key -subj "/CN=system:kube-scheduler" -out scheduler.csr
openssl x509 -req -in scheduler.csr -CA ca.crt -CAkey ca.key -out scheduler.crt
```
### Kube Controller Manager Certificate
Same process as Admin user but the CN=system:kube-controller-manager as it is a system component
```bash
openssl genrsa -out controller-manager.key 2048
openssl req -new -key controller-manager.key -subj "/CN=system:kube-controller-manager" -out controller-manager.csr
openssl x509 -req -in controller-manager.csr -CA ca.crt -CAkey ca.key -out controller-manager.crt
```
### Kube Proxy Certificate
Same process as Admin user but the CN=system:kube-proxy as it is a system component
```bash
openssl genrsa -out kube-proxy.key 2048
openssl req -new -key kube-proxy.key -subj "/CN=system:kube-proxy" -out kube-proxy.csr
openssl x509 -req -in kube-proxy.csr -CA ca.crt -CAkey ca.key -out kube-proxy.crt
```
# Generating Server-Side Certificates
### ETCD Server Certificate
For the etcd server, which is critical in high-availability deployments, the certificate generation process is analogous to that for clients. The etcd server may also require additional peer certificates for secure inter-cluster communication.
### Kube API Server Certificate
```bash
openssl genrsa -out apiserver.key 2048
openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr -config openssl.cnf # for alternative names
```
Create an OpenSSL configuration file (e.g., `openssl.cnf`) with the following content to define Subject Alternative Names (SAN):
```cnf
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name

[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation
subjectAltName = @alt_names

[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1
IP.2 = 172.17.0.87
```

```bash
openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out apiserver.crt -extensions v3_req -extfile openssl.cnf -days 1000
```
### Kubelet Server and Client Certificates
Each Kubernetes node runs a kubelet, which serves as an HTTPS API server to manage node operations. Every node must possess its own key and certificate pair, typically named after the node (e.g., node-01, node-02, node-03). The naming convention uses the prefix “system:node:” followed by the node name, ensuring that the API server assigns the correct permissions.
![[Pasted image 20261120040828.png]]

The node-specific certificates are then referenced inside the kubelet configuration file. For example:
```yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
```

![[Pasted image 20261120041314.png]]
