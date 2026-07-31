[[Kubernetes]]
https://github.com/kubernetes/community/blob/main/contributors/devel/sig-scheduling/scheduling_code_hierarchy_overview.md
https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/
https://jvns.ca/blog/2017/07/27/how-does-the-kubernetes-scheduler-work/
https://stackoverflow.com/questions/28857993/how-does-kubernetes-scheduler-work
https://alibaba-cloud.medium.com/getting-started-with-kubernetes-scheduling-process-and-scheduler-algorithms-847e660533f1
![[Pasted image 20260801014929.png]]
https://helayoty.substack.com/p/kubernetes-scheduler-queue-management
![[Pasted image 20260801015406.png]]
# Multiple schedulers
https://notes.kodekloud.com/docs/Kubernetes-and-Cloud-Native-Associate-KCNA/Scheduling/Multiple-Schedulers/page
![[Pasted image 20260801022615.png]]

Deploy additional scheduler as a pod
![[Pasted image 20260801022927.png]]
https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/
![[Pasted image 20260801023548.png|481]]
![[Pasted image 20260801023701.png|338]]

Configure custom scheduler
![[Pasted image 20260801023932.png]]

View which scheduler is used to schedule which pod from events
![[Pasted image 20260801024135.png]]

Debug scheduling issue from pod logs
![[Pasted image 20260801024254.png]]
# Scheduling plugins
https://notes.kodekloud.com/docs/Kubernetes-and-Cloud-Native-Associate-KCNA/Scheduling/Configuring-Kubernetes-Scheduler-Profiles/page
![[Pasted image 20260801030644.png]]
# Extension points
https://notes.kodekloud.com/docs/Kubernetes-and-Cloud-Native-Associate-KCNA/Scheduling/Configuring-Kubernetes-Scheduler-Profiles/page
Places where we can customise and add code in scheduling steps to change the default behaviour
![[Pasted image 20260801031123.png]]
# Scheduler profiles
In multiple scheduler scenario there can create a race condition among schedulers to schedule same pod in different ways. To prevent that K8s brings scheduler profile ???
![[Pasted image 20260801032124.png]]
