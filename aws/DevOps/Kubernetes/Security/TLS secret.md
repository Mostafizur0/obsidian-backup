[[DevOps/Kubernetes/Security/Security]]

A **Kubernetes TLS Secret** is a specific type of Secret (`kubernetes.io/tls`) used to **store a public/private key pair (X.509 certificate and private key)**. It is primarily used to terminate HTTPS traffic for Kubernetes Ingress controllers or to secure internal pod-to-pod communication.
```bash
kubectl create secret tls webhook-server-tls \
  --cert="/root/keys/webhook-server-tls.crt" \
  --key="/root/keys/webhook-server-tls.key" \
  --namespace=webhook-demo
```
https://support.virtru.com/hc/en-us/articles/39253944595351-Kubernetes-TLS-Secret

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/TLS-Basics/page
```yml
Certificate:
Data:
  Serial Number: 420327018966204255
  Signature Algorithm: sha256WithRSAEncryption
  Issuer: CN=kubernetes
  Validity
    Not After : Feb  9 13:41:28 2020 GMT
  Subject: CN=my-bank.com
  X509v3 Subject Alternative Name:
    DNS:mybank.com, DNS:i-bank.com,
    DNS:we-bank.com,
  Subject Public Key Info:
    00:b9:b0:55:24:fb:a4:ef:77:73:7c:9b
```

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/TLS-in-Kubernetes/page

==Every digital certificate contains a public key==. In fact, the primary purpose of an X.509 certificate (like the ones used by `kube-apiserver` and `etcd`) is to securely bind a **public key** to a specific identity (like a server domain name or a client username). It is not possible to forge a certificate as it is signed with a private key from the certificate authority. The private key is only present in the cluster master node and never leaves the master node.

A Kubernetes certificate contains three main pieces of information:
1. **The Identity**: Who owns the certificate (e.g., `CN=kube-apiserver`).
2. **The Public Key**: The actual cryptographic key used to initiate the key exchange.
3. **The Digital Signature**: A stamp from the Certificate Authority (CA) proving the certificate is authentic and has not been tampered with.
The certificate configured inside the `kube-apiserver` manifest and the certificate configured inside the `etcd` configuration file operate as a client-server pair.

Regarding file naming conventions, certificates containing ==public keys typically have extensions such as .crt or .pem (e.g., server.crt, server.pem or client.crt, client.pem), and private key files usually include “key” in the filename or extension (e.g., server.key or server-key.pem).==

Because `etcd` enforces `--client-cert-auth=true`, it will not talk to any client without a valid certificate signed by the ==trusted Certificate Authority (CA)==. When the API server tries to communicate, they perform a handshake where both components validate each other's distinct certificates against the shared **etcd CA** (`ca.crt`).

Certificate Roles
- **The Certificate in `kube-apiserver` (The Client)**: This is an **etcd client certificate** (e.g., `apiserver-etcd-client.crt`). The API server presents this certificate to `etcd` to prove its identity and gain permission to read or write data.
- **The Certificate in `etcd` config (The Server)**: This is the **etcd server certificate** (e.g., `server.crt`). The `etcd` service uses it to encrypt its traffic and prove its identity to the API server.

Summary of the Flow
1. **Verify**: They use their **Certificates** (Asymmetric Crypto) to prove who they are. Then extract the public key from that certificate.
2. **Exchange**: They use **Diffie-Hellman** to agree on a shared secret key.
3. **Encrypt**: They use the **Shared Key** (Symmetric Crypto) / temporary **symmetric session key** to encrypt the actual Kubernetes data traffic. This session key is unique to that specific connection and is discarded when the connection closes.

![[Pasted image 20260811172015.png]]
### Utilizing a Certificate Authority (CA)
![[Pasted image 20260811172304.png]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/TLS-in-Kubernetes-Certificate-Creation/page
### Generating CA Certificates
![[Pasted image 20260811195753.png]]

```bash
openssl genrsa -out ca.key 2048
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/View-Certificate-Details/page
### Admin User Certificate
![[Pasted image 20260811172859.png]]
### Client and Server-Side Certificates
![[Pasted image 20260811173116.png]]

![[Pasted image 20260811175018.png]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/View-Certificate-Details/page
### Etcd Server Certificate
```bash
cat etcd.yaml
- --advertise-client-urls=https://127.0.0.1:2379
- --key-file=/path-to-certs/etcdserver.key
- --cert-file=/path-to-certs/etcdserver.crt
- --client-cert-auth=true
- --data-dir=/var/lib/etcd
- --initial-advertise-peer-urls=https://127.0.0.1:2380
- --initial-cluster=master=https://127.0.0.1:2380
- --listen-client-urls=https://127.0.0.1:2379
- --listen-peer-urls=https://127.0.0.1:2380
- --name=master
- --peer-cert-file=/path-to-certs/etcdpeer1.crt
- --peer-client-cert-auth=true
- --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
- --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
- --snapshot-count=10000
- --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```
### API Server Certificate
```bash
ExecStart=/usr/local/bin/kube-apiserver \
  --advertise-address=${INTERNAL_IP} \
  --allow-privileged=true \
  --apiserver-count=3 \
  --authorization-mode=Node,RBAC \
  --bind-address=0.0.0.0 \
  --enable-swagger-ui=true \
  --etcd-cafile=/var/lib/kubernetes/ca.pem \
  --etcd-certfile=/var/lib/kubernetes/apiserver-etcd-client.crt \
  --etcd-keyfile=/var/lib/kubernetes/apiserver-etcd-client.key \
  --etcd-servers=https://127.0.0.1:2379 \
  --event-ttl=1h \
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \
  --kubelet-client-certificate=/var/lib/kubernetes/apiserver-kubelet-client.crt \
  --kubelet-client-key=/var/lib/kubernetes/apiserver-kubelet-client.key \
  --kubelet-https=true \
  --runtime-config=api/all \
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \
  --service-cluster-ip-range=10.32.0.0/24 \
  --service-node-port-range=30000-32767 \
  --client-ca-file=/var/lib/kubernetes/ca.pem \
  --tls-cert-file=/var/lib/kubernetes/apiserver.crt \
  --tls-private-key-file=/var/lib/kubernetes/apiserver.key \
  --v=2
```
### Inspecting Certificate Details
```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```
### Troubleshooting with Logs
For os logs
```bash
journalctl -u etcd.service -l
```
Below is an example excerpt from etcd logs:
```log
2019-02-13 02:53:28.144631 I | etcdmain: etcd Version: 3.2.18
2019-02-13 02:53:28.144680 I | etcdmain: Git SHA: eddf599c6
2019-02-13 02:53:28.144684 I | etcdmain: Go Version: go1.8.7
2019-02-13 02:53:28.144692 I | etcdmain: Go OS/Arch: linux/amd64
2019-02-13 02:53:28.144696 I | etcdmain: setting maximum number of CPUs to 4, total number of available CPUs is 4
2019-02-13 02:53:28.144734 N | etcdmain: the server is already initialized as member before, starting as etcd member...
2019-02-13 02:53:28.146651 I | etcdserver: name = master
...
WARNING: 2019/02/13 02:53:30 Failed to serve client requests on 127.0.0.1:2379
Failed to dial 127.0.0.1:2379: connection error: desc = "transport: authentication handshake failed: remote error: tls: bad certificate"; please retry.
```
For Clusters Using kubeadm
```shell
kubectl logs <pod-name>
docker ps -a
crictl ps -a
docker logs <container-id>
```
### Kubeadm Certificate path
![[Pasted image 20260811201052.png]]
