[[DevOps/Kubernetes/Security/Security|Security]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Security/Certificates-API/page

Kubernetes simplifies certificate management through its built-in Certificates API, automating the CSR lifecycle and certificate rotation. This automation is essential for maintaining secure access as team sizes grow and certificates expire over time.
# Managing Certificate Signing Requests (CSRs)
### Step 1: User Generates Private Key and CSR
```bash
openssl genrsa -out jane.key 2048
```
### Step 2: Administrator Creates a CSR Object
```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  expirationSeconds: 600 # seconds
  usages:
    - digital signature
    - key encipherment
    - server auth
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNUUw0tLS0KTUl1Q1dEQ0NBVUFDQVFBd0V6RVJHQTFVdU0R6VjRkNHTQ0RzU0aU1yY3I0d11qYXl0c1RUVFRlQiVtNS0tLS0tLkRvd25nUIDhUnQ0dXJ0YW50YmlsZWdslNQZHYR0W1nNHh1RVFLdLtJPG0tLkFUTUJQS0w0UlRqS1JlTVUyZUl3bTJaSE44TG5NQ2czTWc9PQ==
```
Check active requests using
```bash
kubectl get csr
```
### Step 3: Approving the CSR
```bash
kubectl certificate approve jane
```
or deny
```bash
kubectl certificate deny jane
```
or delete the request
```bash
kubectl delete csr jane
```
After approval the object becomes
```yml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  creationTimestamp: 2019-02-13T16:36:43Z
  name: new-user
spec:
  groups:
    - system:masters
    - system:authenticated
  expirationSeconds: 600
  usages:
    - digital signature
    - key encipherment
    - server auth
  username: kubernetes-admin
status:
  certificate: L$0tL1CRUdJTiBDRVJUSUZJQ0FURS9tL0t1SURDakNDQWZLZ0F3SUJBZ0lVRmwyQ2wXYXoxalW5M3JNVisreFRYQYouW3dnd0RWpL1pJaHZjTkFRRUwkQkFBd0ZVUnRVMVhQTFRUVF4TUhM1ZpHkdVpMjxkF1RncweE9UQ1NVE14TmpNeU1EQmFgdGl0dY0ZFBl2ajNuSXY3eFd3I1Rm5u440c0t520vXukwTFM5V29ge1hHZdWCMlEZ2FOMVVMRFBXTVhjN09FVnVjSk1k4weRUVtR5tD11zWeHVjS1h6g1dV0pMediMUGbXYFKWVKWMVmBjRVRTY3dod2xiO1ND0kLS0tL1F0kQg0V5VElSGUNBVEUt
  conditions:
    - lastUpdateTime: 2019-02-13T16:37:21Z
      message: This CSR was approved by kubectl certificate approve.
      reason: KubectlApprove
      type: Approved
```
Within the Kubernetes control plane, components such as the API Server, Scheduler, and Controller Manager work together. However, ==all certificate-related operations—such as CSR approval and signing—are managed by the Controller Manager.==
The Controller Manager includes dedicated controllers for CSR approval and CSR signing tasks. Since signing certificates requires access to the CA’s root certificate and private key, its configuration specifies the file paths to these credentials. For example, ==the Controller Manager’s configuration file might include settings like the following:==
```yml
cat /etc/kubernetes/manifests/kube-controller-manager.yaml
spec:
  containers:
  - command:
      - kube-controller-manager
      - --address=127.0.0.1
      - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
      - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
      - --controllers=*,bootstrapsigner,tokencleaner
      - --kubeconfig=/etc/kubernetes/controller-manager.conf
      - --leader-elect=true
      - --root-ca-file=/etc/kubernetes/pki/ca.crt
      - --service-account-private-key-file=/etc/kubernetes/pki/sa.key
      - --use-service-account-credentials=true
```

# Kubelet CSR process
https://kubernetes.io/docs/tasks/tls/certificate-rotation/

https://www.qcecuring.com/blog/kubernetes-certificate-management-guide
## Layer 1: Cluster Infrastructure Certificates

These are the certificates Kubernetes itself uses for internal component authentication. They’re created at cluster initialization and expire after 1 year (kubeadm default).

In Kubernetes, the ==**aggregation layer**== allows you to plug custom or extension API servers (such as `metrics-server`) into the main `kube-apiserver`.

[[TLS secret]]

```
/etc/kubernetes/pki/
├── ca.crt + ca.key                    # Cluster CA (signs everything below)
├── apiserver.crt + apiserver.key      # API server TLS (what kubectl connects to)
├── apiserver-kubelet-client.crt       # API server → kubelet authentication
├── apiserver-etcd-client.crt          # API server → etcd authentication
├── front-proxy-ca.crt + key           # Aggregation layer CA
├── front-proxy-client.crt + key       # Aggregation layer client
├── etcd/
│   ├── ca.crt + ca.key               # etcd CA (separate from cluster CA)
│   ├── server.crt + server.key       # etcd server TLS
│   ├── peer.crt + peer.key           # etcd peer communication
│   └── healthcheck-client.crt + key  # etcd health checks
└── sa.key + sa.pub                    # Service account token signing
```
**kubeadm clusters:** Certificates expire after 1 year. If nobody renews them:

- kubectl returns “certificate has expired”
- New pods can’t be scheduled
- Existing pods continue running but can’t be managed
- The cluster is effectively frozen

**Managed clusters (EKS, GKE, AKS):** The provider handles infrastructure certificate rotation ==automatically. You never see these certificates.==

### How to Manage
```
# Check expiry dates (kubeadm)
kubeadm certs check-expiration

# Renew all certificates
kubeadm certs renew all
systemctl restart kubelet

# Automate with a cron job (run monthly)
0 0 1 * * /usr/bin/kubeadm certs renew all && systemctl restart kubelet
```
## Layer 2: Ingress Certificates (cert-manager)

These are the TLS certificates that terminate HTTPS for your public-facing services. cert-manager is the standard tool for managing them. 
Cert-manager handles: 
key generation → CSR → ACME challenge → certificate issuance → Secret creation → renewal at 2/3 lifetime.
```
# 1. Install cert-manager
# kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml

# 2. Create a ClusterIssuer (Let's Encrypt)
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: platform-team@example.com
    privateKeySecretRef:
      name: letsencrypt-prod-key
    solvers:
    - http01:
        ingress:
          class: nginx
    - dns01:
        cloudDNS:
          project: my-gcp-project
      selector:
        dnsNames:
        - "*.example.com"

# 3. Annotate your Ingress (automatic certificate provisioning)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls  # cert-manager creates and manages this Secret
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app
            port:
              number: 80
```
### For Internal Services (Private CA)
An internal service certificate (or private TLS/SSL certificate) is issued by an organization's own private Certificate Authority (CA) rather than a public CA. It is used ==to secure, authenticate, and encrypt internal communication and operations==.
Key Uses of Internal Service Certificates:
- Securing Internal Web Apps & Intranets
- Service-to-Service & Microservices Communication
- Machine & Device Authentication
- Database & Network Security
- VPN & Wi-Fi Access (802.1X)
- Compliance & Zero-Trust Architecture
```
# Vault issuer for internal certificates
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: vault-internal
spec:
  vault:
    server: https://vault.internal:8200
    path: pki_int/sign/internal-service
    auth:
      kubernetes:
        role: cert-manager
        mountPath: /v1/auth/kubernetes

---
# Internal service certificate
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: payment-api-tls
  namespace: production
spec:
  secretName: payment-api-tls
  issuerRef:
    name: vault-internal
    kind: ClusterIssuer
  dnsNames:
  - payment-api.production.svc.cluster.local
  - payment-api.internal.example.com
  duration: 720h      # 30 days
  renewBefore: 240h   # Renew 10 days before expiry
  privateKey:
    algorithm: ECDSA
    size: 256
```
## Layer 3: Service-to-Service mTLS (Service Mesh)
For encrypting and authenticating all pod-to-pod traffic.
### Istio (Most Common)

```
# Enable strict mTLS mesh-wide
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT

# Istio automatically:
# - Injects Envoy sidecar into every pod
# - Issues per-pod certificates (24h validity, SPIFFE ID)
# - Rotates certificates before expiry
# - Encrypts all pod-to-pod traffic with mTLS
# - No application code changes required
```

### Linkerd (Simpler Alternative)

```
# Install Linkerd (mTLS on by default)
linkerd install | kubectl apply -f -

# Inject into a namespace
kubectl annotate namespace production linkerd.io/inject=enabled

# Verify mTLS is active
linkerd viz edges -n production
# Shows: all connections secured with mTLS
```
## The Complete Monitoring Stack

Monitor all three layers with a unified approach:

```
# Prometheus rules covering all certificate layers
groups:
- name: certificate-monitoring
  rules:
  # Layer 1: Cluster infrastructure
  - alert: ClusterCertExpiring
    expr: (apiserver_client_certificate_expiration_seconds - time()) / 86400 < 30

  # Layer 2: Ingress (cert-manager)
  - alert: IngressCertExpiring
    expr: (certmanager_certificate_expiration_timestamp_seconds - time()) / 86400 < 7

  # Layer 3: Service mesh
  - alert: IstioCertError
    expr: increase(istio_agent_cert_rotation_failure_total[5m]) > 0

  # External probe (what clients actually see)
  - alert: ExternalCertExpiring
    expr: (probe_ssl_earliest_cert_expiry - time()) / 86400 < 14
```
