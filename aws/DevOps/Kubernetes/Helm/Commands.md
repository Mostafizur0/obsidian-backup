[[Helm]]

```bash
# Snap (Ubuntu / distributions with snapd)
sudo snap install helm --classic

# Debian/Ubuntu (recommended: signed keyring + repository)
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates gnupg

# Download and add Helm signing key, then add the repo
curl https://baltocdn.com/helm/signing.asc | sudo gpg --dearmor -o /usr/share/keyrings/helm.gpg
echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list > /dev/null

sudo apt-get update
sudo apt-get install -y helm

# pkg (e.g., FreeBSD)
sudo pkg install helm

helm version

# Confirm Helm can list releases in the cluster
helm list --all-namespaces 

# 3. If you added a repo, update and list repos:
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm repo list
```

```bash
helm search hub wordpress
# Download the chart archive
helm pull bitnami/wordpress

# Or download and extract in one step
helm pull bitnami/wordpress --untar

# You should now have a 'wordpress' directory
ls wordpress

helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-release bitnami/wordpress

helm dependency update ./hello-world-chart

# Install using the local chart directory
helm install my-release ./wordpress

helm install my-site bitnami/wordpress
helm install my-SECOND-site bitnami/wordpress

helm install my-release bitnami/wordpress \
  --set wordpressBlogName="Helm Tutorials" \
  --set wordpressEmail="john@example.com"

helm install my-release bitnami/wordpress --values custom-values.yaml

helm install nginx-release bitnami/nginx --version 7.1.0

helm upgrade nginx-release bitnami/nginx

helm list

helm history nginx-release

helm rollback nginx-release 1

helm uninstall my-release

$ helm install wordpress ./wordpress-chart      # creates revision 1
$ helm upgrade wordpress ./wordpress-chart-v2  # creates revision 2
$ helm rollback wordpress 1                    # attempts to roll back to revision 1

$ kubectl set image deployment/wordpress wordpress=wordpress:5.8-apache
```

```bash
$ helm search hub wordpress

URL                                             CHART VERSION   APP VERSION   DESCRIPTION
https://artifacthub.io/packages/helm/riftbit/...   12.1.16        5.8.1         Web publishing platform for building blogs
https://artifacthub.io/packages/helm/bitnami-ak...   12.1.18        5.8.1         Web publishing platform for building blogs
https://artifacthub.io/packages/helm/bitnami/wo...   12.1.27        5.8.1         Web publishing platform for building blogs
```

```bash
$ helm upgrade nginx-release bitnami/nginx

Release "nginx-release" has been upgraded. Happy Helming!
NAME: nginx-release
LAST DEPLOYED: Mon Nov 15 19:25:55 2021
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 9.5.13
APP VERSION: 1.21.4
```

```bash
$ helm list

NAME            NAMESPACE   REVISION    STATUS     CHART           APP VERSION
nginx-release   default     2           deployed   nginx-9.5.13    1.21.4
```

```bash
$ helm history nginx-release

REVISION  UPDATED                  STATUS       CHART           APP VERSION   DESCRIPTION
1         Mon Nov 15 19:20:51 2021  superseded   nginx-7.1.0     1.19.2        Install complete
2         Mon Nov 15 19:25:55 2021  deployed     nginx-9.5.13    1.21.4        Upgrade complete

$ helm history nginx-release

REVISION  UPDATED                  STATUS        CHART           APP VERSION   DESCRIPTION
1         Mon Nov 15 19:20:51 2021  superseded    nginx-7.1.0     1.19.2        Install complete
2         Mon Nov 15 19:25:55 2021  superseded    nginx-9.5.13    1.21.4        Upgrade complete
3         Mon Nov 15 19:30:00 2021  deployed      nginx-7.1.0     1.19.2        Rollback to 1
```
