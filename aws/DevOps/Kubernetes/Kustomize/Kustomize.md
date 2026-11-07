[[Kubernetes]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/Kustomize-Problem-Statement-and-Ideology/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/Kustomize-vs-Helm/page
Install - https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/InstallationSetup/page
```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash

curl -O "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"
# inspect the downloaded file, then run:
bash install_kustomize.sh
```
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/The-kustomizationyaml-File/page
```bash
kustomize build k8s/ | kubectl apply -f -
```
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/Kustomize-Output/page
```bash
$ kustomize build k8s/
$ kustomize build k8s/ | kubectl apply -f -
service/nginx-loadbalancer-service created
deployment.apps/nginx-deployment created

kubectl apply -k k8s/
```
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/Kustomize-ApiVersion-and-Kind/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/Managing-Directories/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/Managing-Directories-Demo/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/Image-Transformers/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/Transformers-Demo/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/Patches-Intro/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/Different-Types-of-Patches/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/Patches-Dictionary/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/Patches-List/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/Overlays/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Kustomize-Basics-2025-Updates/Components/page
