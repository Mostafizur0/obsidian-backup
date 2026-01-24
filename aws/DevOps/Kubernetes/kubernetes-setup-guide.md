# Kubernetes Cluster Setup Guide

This guide walks through setting up a production-ready Kubernetes cluster on Ubuntu VMs with one master node and two worker nodes.

## Prerequisites

- 3 Ubuntu VMs (1 master, 2 workers)
- Root or sudo access
- Network connectivity between all nodes

## Architecture Overview

- **Master Node**: Controls the Kubernetes cluster
- **Worker Node 1**: Runs application workloads
- **Worker Node 2**: Runs application workloads

---

## Step 1: Prepare the Ubuntu VM

### Update System Packages

Run on all nodes:

```bash
sudo apt update && sudo apt upgrade -y
```

### Configure Hostnames

#### Update /etc/hosts (All Nodes)

Edit the hosts file to enable hostname resolution:

```bash
sudo nano /etc/hosts
```

Add the following entries (replace with your actual IP addresses):

```
<MASTER_IP>    k8s-master-node
<WORKER1_IP>   k8s-worker-node-1
<WORKER2_IP>   k8s-worker-node-2
```

Example:
```
192.168.1.10   k8s-master-node
192.168.1.11   k8s-worker-node-1
192.168.1.12   k8s-worker-node-2
```

#### Set Hostnames

**On Master Node:**
```bash
sudo hostnamectl set-hostname "k8s-master-node"
```

**On Worker Node 1:**
```bash
sudo hostnamectl set-hostname "k8s-worker-node-1"
```

**On Worker Node 2:**
```bash
sudo hostnamectl set-hostname "k8s-worker-node-2"
```

### Disable Swap (Required by Kubernetes)

Run on all nodes:

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

Verify swap is disabled:
```bash
free -h
```

---

## Step 2: Enable Kernel Modules and Sysctl Settings

Run on all nodes:

```bash
sudo modprobe overlay
sudo modprobe br_netfilter

sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

---

## Step 3: Install Container Runtime

Choose either Docker or Containerd (Containerd is recommended for production).

### Option A: Install Docker

```bash
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
```

Verify installation:
```bash
docker --version
```

### Option B: Install Containerd (Recommended)

```bash
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

#### Enable systemd cgroup

```bash
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```

#### Update sandbox image

```bash
sudo nano /etc/containerd/config.toml
```

Search for:
```toml
[plugins."io.containerd.grpc.v1.cri"]
```

Update the sandbox_image to:
```toml
sandbox_image = "registry.k8s.io/pause:3.10"
```

#### Restart Containerd

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

Verify:
```bash
sudo systemctl status containerd
```

---

## Step 4: Add Kubernetes APT Repository

Run on all nodes:

```bash
sudo apt install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /" | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
```

---

## Step 5: Install Kubernetes Tools

Run on all nodes:

```bash
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Verify installation:
```bash
kubeadm version
kubectl version --client
kubelet --version
```

---

## Step 6: Install Helm (Master Node Only)

### Method 1: Using Script

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Method 2: Manual Installation

```bash
curl -LO https://get.helm.sh/helm-v3.17.3-linux-amd64.tar.gz
tar -zxvf helm-v3.17.3-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
rm -rf helm-v3.17.3-linux-amd64.tar.gz linux-amd64
```

Verify:
```bash
helm version
```

### Add Helm Repositories

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

## Step 7: Initialize Kubernetes Control Plane (Master Node Only)

Initialize the cluster:

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

**Important:** Save the `kubeadm join` command from the output. You'll need it to join worker nodes.

Example output:
```
kubeadm join 172.31.19.36:6443 --token 922x9d.v0jn4c8he0s286js \
  --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxx
```

---

## Step 8: Configure kubectl (Master Node Only)

Set up kubeconfig for the current user:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## Step 9: Check Node Status

```bash
kubectl get nodes
```

Expected output (NotReady is normal at this stage):
```
NAME              STATUS     ROLES           AGE   VERSION
k8s-master-node   NotReady   control-plane   1m    v1.33.x
```

---

## Step 10: Install Pod Network (CNI)

### Install Flannel CNI

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

### Verify Installation

```bash
kubectl get pods --all-namespaces
```

Wait until all pods are in `Running` state:

```bash
kubectl get nodes
```

Expected output:
```
NAME              STATUS   ROLES           AGE   VERSION
k8s-master-node   Ready    control-plane   5m    v1.33.x
```

---

## Step 11: Join Worker Nodes to the Cluster

On each worker node, run the `kubeadm join` command you saved from Step 7:

```bash
sudo kubeadm join <MASTER_IP>:6443 --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH>
```

Example:
```bash
sudo kubeadm join 172.31.19.36:6443 --token 922x9d.v0jn4c8he0s286js \
  --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxx
```

### Verify Cluster Status (From Master Node)

```bash
kubectl get nodes
```

Expected output:
```
NAME                STATUS   ROLES           AGE   VERSION
k8s-master-node     Ready    control-plane   10m   v1.33.x
k8s-worker-node-1   Ready    <none>          2m    v1.33.x
k8s-worker-node-2   Ready    <none>          1m    v1.33.x
```

Check all system pods:
```bash
kubectl get pods --all-namespaces
```

---

## Step 12: Install and Configure MetalLB (Load Balancer)

MetalLB is a load balancer implementation for bare metal Kubernetes clusters, enabling LoadBalancer-type services to work in on-premise environments.

### Why MetalLB?

In cloud environments (AWS, GCP, Azure), LoadBalancer services are automatically provisioned. For on-premise/bare metal clusters, MetalLB provides this functionality by:
- Assigning external IPs to LoadBalancer services
- Announcing these IPs to the network using Layer 2 (ARP) or BGP

### Install MetalLB using Manifest

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
```

### Verify MetalLB Installation

Wait for MetalLB pods to be running:

```bash
kubectl get pods -n metallb-system
```

Expected output:
```
NAME                          READY   STATUS    RESTARTS   AGE
controller-xxxxxxxxxx-xxxxx   1/1     Running   0          1m
speaker-xxxxx                 1/1     Running   0          1m
speaker-xxxxx                 1/1     Running   0          1m
speaker-xxxxx                 1/1     Running   0          1m
```

### Configure MetalLB IP Address Pool

Create a configuration file for MetalLB with an IP address pool from your network range.

#### Create IPAddressPool Configuration

```bash
cat <<EOF | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.200-192.168.1.250
EOF
```

**Important:** Replace `192.168.1.200-192.168.1.250` with an IP range from your network that:
- Is in the same subnet as your nodes
- Is NOT used by DHCP
- Is available for MetalLB to assign

Examples:
- Single IP: `192.168.1.240/32`
- IP Range: `192.168.1.200-192.168.1.250`
- CIDR: `192.168.1.0/24`

#### Create L2Advertisement Configuration

This tells MetalLB to advertise the IPs using Layer 2 (ARP):

```bash
cat <<EOF | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default-l2-advert
  namespace: metallb-system
spec:
  ipAddressPools:
  - default-pool
EOF
```

### Verify MetalLB Configuration

```bash
kubectl get ipaddresspool -n metallb-system
kubectl get l2advertisement -n metallb-system
```

### Test MetalLB with Sample Application

Deploy a test application to verify MetalLB is working:

```bash
# Create a deployment
kubectl create deployment nginx-test --image=nginx

# Expose it as LoadBalancer service
kubectl expose deployment nginx-test --port=80 --type=LoadBalancer
```

Check the service to see if an external IP was assigned:

```bash
kubectl get svc nginx-test
```

Expected output:
```
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
nginx-test   LoadBalancer   10.96.123.45    192.168.1.200   80:31234/TCP   1m
```

The `EXTERNAL-IP` should show an IP from your MetalLB pool (not `<pending>`).

### Test Access

From any machine on the same network:

```bash
curl http://192.168.1.200
```

You should see the nginx welcome page.

### Clean Up Test Resources

```bash
kubectl delete svc nginx-test
kubectl delete deployment nginx-test
```

### MetalLB Configuration Examples

#### Example 1: Multiple IP Pools for Different Purposes

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: production-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.200-192.168.1.220
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: development-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.221-192.168.1.250
```

To use a specific pool, add annotation to your service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  annotations:
    metallb.universe.tf/address-pool: production-pool
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: my-app
```

#### Example 2: Assign Specific IP to Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  annotations:
    metallb.universe.tf/loadBalancerIPs: 192.168.1.200
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: my-app
```

### MetalLB Troubleshooting

#### Check MetalLB Logs

```bash
# Controller logs
kubectl logs -n metallb-system -l component=controller

# Speaker logs
kubectl logs -n metallb-system -l component=speaker
```

#### Service Stuck in Pending

If your LoadBalancer service shows `<pending>` for EXTERNAL-IP:

```bash
# Check if IPAddressPool exists
kubectl get ipaddresspool -n metallb-system

# Check if L2Advertisement exists
kubectl get l2advertisement -n metallb-system

# Check MetalLB controller logs
kubectl logs -n metallb-system -l component=controller -f
```

#### IP Not Reachable

1. Verify IP is in correct subnet
2. Check firewall rules
3. Verify L2Advertisement is configured
4. Check speaker logs on all nodes

---

## Step 13: Install and Configure NGINX Ingress Controller

NGINX Ingress Controller manages external access to services in your cluster, typically HTTP/HTTPS. It works together with MetalLB to provide a complete traffic routing solution.

### Traffic Flow Architecture

```
Internet/External Network
         ↓
    MetalLB (External IP)
         ↓
  NGINX Ingress Controller (LoadBalancer Service)
         ↓
    Ingress Rules (Routing)
         ↓
  Backend Services (ClusterIP)
         ↓
       Pods
```

### Install NGINX Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0/deploy/static/provider/cloud/deploy.yaml
```

### Verify Installation

Wait for the ingress controller to be ready:

```bash
kubectl get pods -n ingress-nginx
```

Expected output:
```
NAME                                       READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-xxxxxxxxx-xxxxx   1/1     Running   0          2m
```

### Check Ingress Service (MetalLB Integration)

The ingress controller creates a LoadBalancer service that MetalLB will assign an external IP:

```bash
kubectl get svc -n ingress-nginx
```

Expected output:
```
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)
ingress-nginx-controller             LoadBalancer   10.96.45.123    192.168.1.200   80:31234/TCP,443:31567/TCP
ingress-nginx-controller-admission   ClusterIP      10.96.78.234    <none>          443/TCP
```

The `EXTERNAL-IP` (e.g., 192.168.1.200) is assigned by MetalLB. This is the IP you'll use to access your applications.

### Verify Ingress Controller is Working

```bash
# Check ingress class
kubectl get ingressclass

# Test basic connectivity
curl http://192.168.1.200
```

You should get a default 404 response from NGINX (this is normal - no ingress rules configured yet).

---

## Step 14: Complete Deployment Example - Echo Server Application

This example demonstrates the complete traffic flow: **MetalLB → Ingress → Service → Pods**

### Architecture Overview

```
External Client (Browser/curl)
         ↓
MetalLB External IP: 192.168.1.200
         ↓
NGINX Ingress Controller
         ↓ (routes based on hostname/path)
         ↓
Echo Server Service (ClusterIP)
         ↓
Echo Server Pods (3 replicas)
```

### Step 14.1: Create Namespace

```bash
kubectl create namespace demo-app
```

### Step 14.2: Deploy Echo Server Application

Create the deployment manifest:

```bash
cat <<EOF > echo-server-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-server
  namespace: demo-app
  labels:
    app: echo-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: echo-server
  template:
    metadata:
      labels:
        app: echo-server
    spec:
      containers:
      - name: echo-server
        image: ealen/echo-server:latest
        ports:
        - containerPort: 80
        env:
        - name: PORT
          value: "80"
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "100m"
EOF
```

Apply the deployment:

```bash
kubectl apply -f echo-server-deployment.yaml
```

Verify pods are running:

```bash
kubectl get pods -n demo-app
```

Expected output:
```
NAME                           READY   STATUS    RESTARTS   AGE
echo-server-xxxxxxxxxx-xxxxx   1/1     Running   0          30s
echo-server-xxxxxxxxxx-xxxxx   1/1     Running   0          30s
echo-server-xxxxxxxxxx-xxxxx   1/1     Running   0          30s
```

### Step 14.3: Create ClusterIP Service

Create the service manifest:

```bash
cat <<EOF > echo-server-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: echo-server-service
  namespace: demo-app
  labels:
    app: echo-server
spec:
  type: ClusterIP
  selector:
    app: echo-server
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
EOF
```

Apply the service:

```bash
kubectl apply -f echo-server-service.yaml
```

Verify service:

```bash
kubectl get svc -n demo-app
```

Expected output:
```
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
echo-server-service   ClusterIP   10.96.123.45    <none>        80/TCP    30s
```

### Step 14.4: Create Ingress Resource

Create the ingress manifest:

```bash
cat <<EOF > echo-server-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo-server-ingress
  namespace: demo-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: echo.local.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: echo-server-service
            port:
              number: 80
EOF
```

Apply the ingress:

```bash
kubectl apply -f echo-server-ingress.yaml
```

Verify ingress:

```bash
kubectl get ingress -n demo-app
```

Expected output:
```
NAME                  CLASS   HOSTS             ADDRESS         PORTS   AGE
echo-server-ingress   nginx   echo.local.com    192.168.1.200   80      30s
```

### Step 14.5: Configure DNS/Hosts File

For testing, add entry to your local machine's hosts file:

**On Linux/Mac:**
```bash
sudo nano /etc/hosts
```

**On Windows:**
```
C:\Windows\System32\drivers\etc\hosts
```

Add this line (replace with your MetalLB IP):
```
192.168.1.200  echo.local.com
```

### Step 14.6: Test the Complete Setup

```bash
# Test with curl
curl http://echo.local.com

# Test with specific headers
curl -H "Host: echo.local.com" http://192.168.1.200

# Get detailed response
curl -v http://echo.local.com
```

You should see a JSON response from the echo server showing request details:
```json
{
  "host": {
    "hostname": "echo.local.com",
    "ip": "::ffff:10.244.1.5",
    "ips": []
  },
  "http": {
    "method": "GET",
    "baseUrl": "",
    "originalUrl": "/",
    "protocol": "http"
  },
  "request": {
    "params": {
      "0": "/"
    },
    "query": {},
    "cookies": {},
    "body": {},
    "headers": {
      "host": "echo.local.com",
      "user-agent": "curl/7.68.0"
    }
  },
  "environment": {
    "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
    "HOSTNAME": "echo-server-xxxxxxxxxx-xxxxx",
    "PORT": "80"
  }
}
```

### Step 14.7: Advanced Ingress Configuration

#### Example 1: Multiple Paths

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-path-ingress
  namespace: demo-app
spec:
  ingressClassName: nginx
  rules:
  - host: echo.local.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

#### Example 2: Multiple Hosts

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-host-ingress
  namespace: demo-app
spec:
  ingressClassName: nginx
  rules:
  - host: api.local.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
  - host: app.local.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

#### Example 3: TLS/HTTPS Configuration

First, create a TLS secret:

```bash
# Create self-signed certificate (for testing)
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=echo.local.com/O=demo-app"

# Create Kubernetes secret
kubectl create secret tls echo-tls-secret \
  --key tls.key \
  --cert tls.crt \
  -n demo-app
```

Then create HTTPS ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo-server-ingress-tls
  namespace: demo-app
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - echo.local.com
    secretName: echo-tls-secret
  rules:
  - host: echo.local.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: echo-server-service
            port:
              number: 80
```

Test HTTPS:
```bash
curl -k https://echo.local.com
```

### Step 14.8: Verify Complete Traffic Flow

```bash
# 1. Check MetalLB assigned IP to Ingress Controller
kubectl get svc -n ingress-nginx ingress-nginx-controller

# 2. Check Ingress rules
kubectl get ingress -n demo-app

# 3. Check Service endpoints
kubectl get endpoints -n demo-app echo-server-service

# 4. Check Pod status
kubectl get pods -n demo-app -o wide

# 5. View Ingress Controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller -f

# 6. Describe ingress for details
kubectl describe ingress echo-server-ingress -n demo-app
```

### Step 14.9: Monitoring and Debugging

#### Check Ingress Controller Metrics

```bash
# Port-forward to access metrics
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller-metrics 10254:10254

# In another terminal
curl http://localhost:10254/metrics
```

#### Test from Within Cluster

```bash
# Create a test pod
kubectl run test-pod --image=curlimages/curl -it --rm -- sh

# Inside the pod, test the service directly
curl http://echo-server-service.demo-app.svc.cluster.local
```

#### Common Ingress Annotations

```yaml
metadata:
  annotations:
    # Rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "10"

    # Connection timeout
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "600"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "600"

    # Client body size limit
    nginx.ingress.kubernetes.io/proxy-body-size: "100m"

    # CORS
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "*"

    # SSL redirect
    nginx.ingress.kubernetes.io/ssl-redirect: "true"

    # Authentication
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"
```

### Step 14.10: Cleanup (Optional)

To remove the demo application:

```bash
kubectl delete -f echo-server-ingress.yaml
kubectl delete -f echo-server-service.yaml
kubectl delete -f echo-server-deployment.yaml
kubectl delete namespace demo-app
```

---

## Production Best Practices

### 1. Resource Management

Always set resource requests and limits:

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

### 2. High Availability

- Run multiple replicas of your application
- Configure Pod Disruption Budgets (PDB)
- Use anti-affinity rules to spread pods across nodes

```yaml
spec:
  replicas: 3
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - echo-server
          topologyKey: kubernetes.io/hostname
```

### 3. Health Checks

Always configure liveness and readiness probes:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 80
  initialDelaySeconds: 30
  periodSeconds: 10
readinessProbe:
  httpGet:
    path: /ready
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
```

### 4. Security

- Use NetworkPolicies to restrict traffic
- Enable Pod Security Standards
- Use RBAC for access control
- Scan images for vulnerabilities
- Use TLS for ingress

### 5. Monitoring and Logging

- Deploy Prometheus for metrics
- Use Grafana for visualization
- Implement centralized logging (EFK/ELK stack)
- Set up alerts for critical issues

---

## Troubleshooting

### If Token Expires

Generate a new token on the master node:

```bash
kubeadm token create --print-join-command
```

### Check Cluster Health

```bash
kubectl cluster-info
kubectl get componentstatuses
```

### View Logs

```bash
# Kubelet logs
sudo journalctl -u kubelet -f

# Container runtime logs
sudo journalctl -u containerd -f
```

### Reset Kubernetes (If Needed)

```bash
sudo kubeadm reset
sudo rm -rf /etc/cni/net.d
sudo rm -rf $HOME/.kube/config
```

---

## Next Steps

1. **Deploy Applications**: Use `kubectl apply` or Helm charts
2. **Set Up Monitoring**: Install Prometheus and Grafana using Helm
3. **Configure Ingress**: Set up NGINX Ingress Controller for HTTP/HTTPS routing
4. **Enable Security**: Configure RBAC, Network Policies, and Pod Security Standards
5. **Storage Solution**: Install and configure persistent storage (NFS, Longhorn, or Rook-Ceph)
6. **Backup & Disaster Recovery**: Set up Velero for cluster backups

---

## Useful Commands

```bash
# View all resources
kubectl get all --all-namespaces

# Describe a node
kubectl describe node <node-name>

# View cluster info
kubectl cluster-info

# Get cluster events
kubectl get events --all-namespaces

# Check pod logs
kubectl logs <pod-name> -n <namespace>

# Execute commands in pod
kubectl exec -it <pod-name> -- /bin/bash

# Apply configuration
kubectl apply -f <file.yaml>

# Delete resources
kubectl delete -f <file.yaml>
```

---

## Summary

You now have a fully functional production-ready Kubernetes cluster with:
- ✅ 1 Master Node (Control Plane)
- ✅ 2 Worker Nodes
- ✅ Flannel Pod Network (CNI)
- ✅ Helm Package Manager
- ✅ MetalLB Load Balancer (External IP assignment) [Advanced L2 configuration :: MetalLB, bare metal load-balancer for Kubernetes](https://metallb.io/configuration/_advanced_l2_configuration/)
- ✅ NGINX Ingress Controller (HTTP/HTTPS routing)
- ✅ kubectl configured
- ✅ Complete traffic flow: **External Network → MetalLB → Ingress → Service → Pods**

Your cluster is ready for deploying production applications with external load balancing and ingress capabilities!

---

## References

- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [Kubeadm Documentation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)
- [Flannel CNI](https://github.com/flannel-io/flannel)
- [Helm Documentation](https://helm.sh/docs/)
- [MetalLB Documentation](https://metallb.universe.tf/)
- [MetalLB Configuration Guide](https://metallb.universe.tf/configuration/)
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [Ingress Documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [NGINX Ingress Annotations](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)
- [Advanced L2 configuration :: MetalLB, bare metal load-balancer for Kubernetes](https://metallb.io/configuration/_advanced_l2_configuration/)
- 
