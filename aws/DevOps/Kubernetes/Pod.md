[[Kubernetes]]

# Self-healing applications
Kubernetes supports self-healing applications through ==ReplicaSets and Replication Controllers==. The replication controller helps ensure that a POD is re-created automatically when the application within the POD crashes. It helps in ensuring enough replicas of the application are running at all times.

Kubernetes provides additional support to check the health of applications running within PODs and take necessary actions through Liveness and Readiness Probes.
https://notes.kodekloud.com/docs/Docker-Certified-Associate-Exam-Course/Kubernetes/Demo-Replica-Sets/page#3-replicaset-self-healing
https://notes.kodekloud.com/docs/Docker-Certified-Associate-Exam-Course/Kubernetes/Replication-Controllers-and-ReplicaSets/page
