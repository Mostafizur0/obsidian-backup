[[Security]]

A **Kubernetes TLS Secret** is a specific type of Secret (`kubernetes.io/tls`) used to **store a public/private key pair (X.509 certificate and private key)**. It is primarily used to terminate HTTPS traffic for Kubernetes Ingress controllers or to secure internal pod-to-pod communication.
```bash
kubectl create secret tls webhook-server-tls \
  --cert="/root/keys/webhook-server-tls.crt" \
  --key="/root/keys/webhook-server-tls.key" \
  --namespace=webhook-demo
```
https://support.virtru.com/hc/en-us/articles/39253944595351-Kubernetes-TLS-Secret
