[[Kubernetes]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Helm-Basics-2025-Updates/Helm-Introduction/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Helm-Basics-2025-Updates/Installation-and-Configuration/page
Helm 3’s three-way merge - https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Helm-Basics-2025-Updates/A-quick-note-about-Helm2-vs-Helm3/page
Helm components - https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Helm-Basics-2025-Updates/Helm-Components/page
![[Pasted image 20261107165040.png]]
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Helm-Basics-2025-Updates/Helm-charts/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Helm-Basics-2025-Updates/Customizing-chart-parameters/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Helm-Basics-2025-Updates/Working-with-Helm-basics/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Helm-Basics-2025-Updates/Lifecycle-management-with-Helm/page

Use hooks or external backup tooling For workloads that store critical data, use pre-upgrade hooks for snapshots or external backups. Hooks can run jobs to snapshot databases or copy data to external storage and can also be used to validate or prepare the system before applying new manifests.

https://medium.com/@DynamoDevOps/the-complete-practical-guide-to-helm-for-kubernetes-from-basics-to-production-best-practices-b7c058b0589e?sk=06bd3eeee28cd14290016129cce5d1f1
### Best Practice Tips
- Make sure to execute **helm lint** and **helm template** prior to deployment.

To fix releases that are stuck:
- Use `helm history` and `helm ls --all` to review the state of the release.
- Rollback or remove any stuck releases as necessary.
- If desired, remove Helm’s secret entries to resolve any inconsistencies in the state:
```bash
kubectl delete secret sh.helm.release.v1.<release>.v<revision>
```
- For issues related to the OCI registry, make sure to use`helm registry login` for authentication.
- Regularly update chart repositories to avoid encountering “not a valid chart repository” errors.
- Ensure that your`.helmignore` file remains concise to prevent reaching payload limitations.
