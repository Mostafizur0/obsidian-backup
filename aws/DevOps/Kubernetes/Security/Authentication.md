[[DevOps/Kubernetes/Security/Security|Security]]

Access to the API server is regulated by robust authentication mechanisms. Kubernetes supports multiple methods, including:
- Static files with user IDs and passwords
- Tokens
- Certificates
- Integrations with external providers such as LDAP or Kerberos
- Service accounts for machine-to-machine communications [[Service Accounts]]

While you cannot list or create user accounts directly in Kubernetes, the system handles service accounts natively via the Kubernetes API. All requests—whether initiated by the kubectl tool or directly sent to the API—are processed by the kube-apiserver.
https://notes.kodekloud.com/docs/Certified-Kubernetes-Security-Specialist-CKS/Cluster-Setup-and-Hardening/Authentication/page
