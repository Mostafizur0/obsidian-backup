[[Docker]]

# CMD
It is the first process / command that will run after the container starts.
![[Pasted image 20260806031109.png]]
To overwrite it provide the command with argument after docker run command
![[Pasted image 20260806031220.png]]
# Entrypoint
It is the first command that will run after the container starts without the argument. It expects an argument, without it the container will through error.
![[Pasted image 20260806031719.png]]
Provide default argument of entrypoint through CMD command after it. It append the entrypoint and cmd values after the run command.
Can override the CMD value in docker run command. To override entrypoint command use --entrypoint flag in docker run command.
![[Pasted image 20260806032118.png]]
# Override them in Kubernetes
We can override the image Entrypoint and cmd through commands and args keys in the k8s manifest.
![[Pasted image 20260806032855.png]]

![[Pasted image 20260806033001.png]]
https://notes.kodekloud.com/docs/Certified-Kubernetes-Application-Developer-CKAD/Configuration/Commands-and-Arguments-in-Docker/page#using-entrypoint-to-combine-commands-and-arguments
https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Application-Lifecycle-Management/Commands-and-Arguments-in-Kubernetes/page#overriding-default-behavior-with-arguments
