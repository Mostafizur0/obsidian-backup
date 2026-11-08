[[Troubleshooting]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Other-Topics/JSON-Path-in-Kubernetes/page
```json
{
  "apiVersion": "v1",
  "kind": "List",
  "items": [
    {
      "apiVersion": "v1",
      "kind": "Pod",
      "metadata": {
        "name": "nginx-5557945897-gznjp"
      },
      "spec": {
        "containers": [
          {
            "image": "nginx:alpine",
            "name": "nginx"
          }
        ],
        "nodeName": "node01"
      }
    }
  ]
}
```

```bash
$ kubectl get pods -o json

$ kubectl get pods -o=jsonpath='{.items[0].spec.containers[0].image}'
nginx:alpine

$ kubectl get nodes -o=jsonpath='{.items[*].metadata.name}'
master node01

$ kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.architecture}'
amd64 amd64

$ kubectl get nodes -o=jsonpath='{.items[*].status.capacity.cpu}'
4 4

$ kubectl get nodes -o=jsonpath='{.items[*].metadata.name} {.items[*].status.capacity.cpu}'
master node01 4 4

$ kubectl get nodes -o=jsonpath='{.items[*].metadata.name} {"\n"} {.items[*].status.capacity.cpu}'
master node01
4 4

$ kubectl get nodes -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'
master  4
node01  4

$ kubectl get nodes -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu
NODE     CPU
master   4
node01   4

$ kubectl get nodes --sort-by=.metadata.name -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu
NODE    CPU
master  4
node01  4
```
