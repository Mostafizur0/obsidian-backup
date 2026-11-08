[[Kubernetes]]

https://github.com/kodekloudhub/certified-kubernetes-administrator-course
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
https://kubernetes.io/docs/reference/kubectl/conventions/
https://kubernetes.io/docs/reference/kubectl/generated/
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/upgrading-linux-nodes/
https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/
https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster
https://kubernetes.io/docs/tasks/administer-cluster/certificates/
https://kubernetes.io/docs/tasks/tls/certificate-issue-client-csr/
https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#launch-a-pod-using-service-account-token-projection
https://kubernetes.io/docs/tutorials/configuration/configure-persistent-volume-storage/
https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/
https://kubernetes.io/docs/tasks/manage-kubernetes-objects/update-api-object-kubectl-patch/

```
export KUBECONFIG="${KUBECONFIG}:config-demo:config-demo-2"

kubectl config view
kubectl config --kubeconfig=config-demo view --minify // view current context
kubectl config --kubeconfig=config-demo use-context research

alias k=kubectl
export d="--dry-run=client -o yaml"
export now="--force --grace-period 0"

kubectl run nginx --image=nginx

kubectl run nginx --image=nginx --dry-run=client -o yaml

kubectl create deployment --image=nginx nginx

kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml

kubectl create -f nginx-deployment.yaml

kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml

kubectl create deployment nginx --image=nginx --replicas=4

kubectl scale

kubectl scale deployment nginx--replicas=4

kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml

kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml // only use it for nodeport service

kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml

kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml // only use it for nodeport service

kubectl api-resources

kubectl explain pods
kubectl explain pods.spec
kubectl explain pods --recursive

kubectl exec app -- vi /log/app.log -n elastic-stack

openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text -noout

crictl ps -a

crictl logs <contaier-id>

k get rolebindings -n kube-system -o wide | grep <role-name>

ps -aux | grep authorization // linux processes with startup commands

kubectl get pods -A --no-headers | wc -l // no. of pods in all ns

kubectl exec <pod> -- <bash-command>
```

`--dry-run` By default as soon as the command is run, the resource will be created. If you simply want to test your command, use the `--dry-run=client` option. This will not create the resource; instead, it tells you whether the resource can be created and if your command is right.

`-o yaml` This will output the resource definition in YAML format on the screen.
