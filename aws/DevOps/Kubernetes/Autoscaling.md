[[Kubernetes]]

![[Pasted image 20260807001033.png]]
# HPA
![[Pasted image 20260807001551.png|306]]

![[Pasted image 20260807002032.png]]
1.23 version HPA is build upon metric server values. So metric server is a prerequisite in 1.23
![[Pasted image 20260807002258.png]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Application-Lifecycle-Management/Introduction-to-Autoscaling-2025-Updates/page
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Application-Lifecycle-Management/Horizontal-Pod-Autoscaler-HPA-2025-Updates/page
# Metrics sources
![[Pasted image 20260807002603.png]]
# In place Resize of Pods 2025 Updates
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Application-Lifecycle-Management/In-place-Resize-of-Pods-2025-Updates/page
# VPA
![[Pasted image 20260807005929.png|352]]
Need to install it from k8s manifest file.
![[Pasted image 20260807010211.png]]

![[Pasted image 20260807010622.png]]
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Application-Lifecycle-Management/Vertical-Pod-Autoscaling-VPA-2025-Updates/page

Kubernetes has a safety feature that prevents removing the last pod of a deployment to avoid service downtime. When you have only 1 replica and VPA tries to evict it, Kubernetes blocks this action with the error message: =="too few replicas".==
