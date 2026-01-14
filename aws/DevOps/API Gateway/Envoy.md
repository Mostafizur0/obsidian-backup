[[API Gateway]]
```
helm install eg oci://docker.io/envoyproxy/gateway-helm --version v1.6.1 -n envoy --create-namespace
```

kubectl wait --timeout=5m -n envoy deployment/envoy-gateway --for=condition=Available


kubectl apply -f https://github.com/envoyproxy/gateway/releases/download/v1.6.1/quickstart.yaml -n default

```
kubectl apply -f https://github.com/envoyproxy/gateway/releases/download/v1.6.1/quickstart.yaml -n default
```
```
export ENVOY_SERVICE=$(kubectl get svc -n envoy --selector=gateway.envoyproxy.io/owning-gateway-namespace=default,gateway.envoyproxy.io/owning-gateway-name=eg -o jsonpath='{.items[0].metadata.name}')
kubectl -n envoy port-forward service/${ENVOY_SERVICE} 8888:80 &
```

export ENVOY_SERVICE=$(kubectl get svc -n envoy --selector=gateway.envoyproxy.io/owning-gateway-namespace=default,gateway.envoyproxy.io/owning-gateway-name=eg -o jsonpath='{.items[0].metadata.name}')

kubectl -n envoy port-forward service/${ENVOY_SERVICE} 8887:80 &

curl --verbose --header "Host: www.example.com" http://localhost:8887/get

