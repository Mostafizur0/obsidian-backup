[[DevOps/Kubernetes/Security/Security|Security]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Security-Specialist-CKS/Cluster-Setup-and-Hardening/What-are-CIS-Benchmarks/page
CIS, or the Center for Internet Security, is a nonprofit organization focused on enhancing cybersecurity through community-driven best practices. Their mission is to create a safer connected world by developing, validating, and promoting actionable security recommendations. Users can register on the CIS website to download the benchmarks that suit their specific environment.

Each CIS benchmark includes detailed security recommendations:
- An explanation of the risks associated with non-compliant configurations.
- Step-by-step instructions to verify if a security risk exists, including the necessary commands.
- Procedures to resolve identified issues.
# Configuration Assessment Tool
CIS not only provides these best practices but also offers tools for automated assessments. The [CIS CAT](https://www.cisecurity.org/cis-cat-pro/) (Configuration Assessment Tool) automates the process of comparing your server’s configuration against CIS benchmarks and generates a comprehensive HTML report.

https://www.cisecurity.org/cis-benchmarks#kubernetes
https://www.cisecurity.org/benchmark/kubernetes
https://www.cisecurity.org/cybersecurity-tools/cis-cat-pro/cis-benchmarks-supported-by-cis-cat-pro
https://learn.cisecurity.org/cis-cat-lite
# CIS Benchmark Assessment Tool
https://notes.kodekloud.com/docs/Certified-Kubernetes-Security-Specialist-CKS/Cluster-Setup-and-Hardening/CIS-benchmark-for-Kubernetes/page
# Kube bench
Developed by Aqua Security, Kube-Bench is an open-source tool that automatically verifies whether your Kubernetes deployment follows the best practices outlined in the CIS Benchmarks. Kube-Bench aligns its assessment with the list of recommended CIS best practices for Kubernetes. For each CIS recommendation, there’s a corresponding check performed by the tool. This systematic approach helps you quickly identify potential security issues in your cluster configurations.
https://github.com/aquasecurity/kube-bench
![[Pasted image 20261117230848.png]]

There are several methods available to deploy and run Kube-Bench:
- **As a Docker Container:** Run Kube-Bench in isolation using Docker.
- **As a Kubernetes Job:** Deploy Kube-Bench as a job within your Kubernetes cluster to regularly monitor compliance.
- **Using Binaries or Compiling from Source:** Install the tool directly on a master node or compile it from source for customized use.
https://notes.kodekloud.com/docs/Certified-Kubernetes-Security-Specialist-CKS/Cluster-Setup-and-Hardening/Kube-bench/page
