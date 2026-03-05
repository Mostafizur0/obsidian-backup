https://github.com/Osomudeya/DevOps-Home-Lab-2026-2027

By the end of this setup, you'll have:

- **Docker** running for containerization
- **Kubernetes tools** (kubectl, k3d) for cluster management
- **Development tools** (Node.js, jq) for application building
- **Package managers** (Helm) for Kubernetes applications
```bash
brew install docker docker-compose kubectl k3d helm nodejs jq
```

```bash
docker --version
kubectl version --client
k3d version
helm version
node --version
npm --version
jq --version
docker info
```

#### Test in Docker
```bash
git clone https://github.com/Osomudeya/DevOps-Home-Lab-2025.git
cd DevOps-Home-Lab-2025
```

```bash
docker-compose build
docker-compose up -d
curl http://localhost:3001/health
curl http://localhost:3000/
```

#### Test in K8s
Create a local Docker image registry (needed for pushing images used by Kubernetes (k3d) cluster)
```bash
k3d registry create k3d-registry --port 5001
```
Create a local 3-node Kubernetes cluster
```bash
k3d cluster create dev-cluster \
  --servers 1 \
  --agents 2 \
  --port "8080:80@loadbalancer" \
  --port "8443:443@loadbalancer"
```

```bash
kubectl get nodes
kubectl get nodes -o wide
kubectl cluster-info
```

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/frontend-config.yaml
kubectl apply -f k8s/secrets.yaml
kubectl get configmap -n humor-game
kubectl get secrets -n humor-game
```

```bash
kubectl apply -f k8s/postgres.yaml
kubectl apply -f k8s/redis.yaml
kubectl get pods -n humor-game
```

### Build and Deploy Application Services

```bash
docker build -t humor-game-frontend:latest ./frontend
docker build -t humor-game-backend:latest ./backend
k3d image import humor-game-frontend:latest -c dev-cluster
k3d image import humor-game-backend:latest -c dev-cluster
kubectl apply -f k8s/backend.yaml
kubectl apply -f k8s/frontend.yaml
kubectl get pods -n humor-game
kubectl port-forward service/backend 3001:3001 -n humor-game &
curl http://localhost:3001/health
```

Ingress

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/baremetal/deploy.yaml

kubectl patch deployment ingress-nginx-controller -n ingress-nginx -p '{"spec":{"template":{"spec":{"containers":[{"name":"controller","ports":[{"containerPort":80,"hostPort":80,"name":"http","protocol":"TCP"},{"containerPort":443,"hostPort":443,"name":"https","protocol":"TCP"},{"containerPort":8443,"name":"webhook","protocol":"TCP"}]}]}}}}'

kubectl rollout status deployment/ingress-nginx-controller -n ingress-nginx

kubectl create namespace argocd

kubectl apply -f k8s/ingress.yaml
kubectl get ingress -n humor-game

echo "127.0.0.1 gameapp.local" | sudo tee -a /etc/hosts
ping gameapp.local
curl -H "Host: gameapp.local" -I http://localhost:8080/
curl -H "Host: gameapp.local" http://localhost:8080/api/health
```
