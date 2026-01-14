psql -h 172.19.0.7 -U postgres

helm repo add kong https://charts.konghq.com
helm repo update
helm search repo kong/ingress
helm fetch kong/ingress
tar xvf ingress-0.21.0.tgz
helm install kong ./ingress -n kong
curl https://192.168.139.2:31603/ -k
check gateway proxy working

kubectl port-forward --address 0.0.0.0 service/kong-gateway-admin -n kong 8444:8444
kubectl port-forward --address 0.0.0.0 service/kong-gateway-manager -n kong 8445:8445
https://shantanudeyanik.medium.com/install-kong-with-a-database-and-custom-values-using-helm-by-fetching-chart-locally-2a1f72e881fb

kubectl apply -R -f ./secret/ -n order
