
# Building a Kubernetes Cluster on Windows 10 Using Hyper-V (With Real Troubleshooting Scenarios)

This guide documents a full Kubernetes lab setup using:

- Windows 10 (Dell laptop)
    
- Hyper-V
    
- Ubuntu Server 22.04 VMs
    
- kubeadm
    
- containerd
    
- Calico (instead of Cilium)
    
- SSH access from Mac
    
- Real-world debugging scenarios
    

It includes every issue encountered and how to fix them.

---

# 1️⃣ Windows 10 Preparation

## Enable Hyper-V

Go to:

```
Settings → Apps → Optional Features → More Windows Features
```

Enable:

- Hyper-V Management Tools
    
- Hyper-V Platform
    

Click **OK** and Reboot.

---

## Enable IP Routing

Open PowerShell as Administrator:

```powershell
Set-ItemProperty -Path HKLM:\system\CurrentControlSet\services\Tcpip\Parameters -Name IpEnableRouter -Value 1
```

Reboot.

---

# 2️⃣ Create Internal Virtual Switch

In Hyper-V Manager:

```
Virtual Switch Manager → New Virtual Switch → Internal
```

Name it:

```
vS_k8sCluster
```

After creation, Windows creates a new adapter:

```
vEthernet (vS_k8sCluster)
```

Assign it in **Network Connections → Properties → IPv4**:
```
IP: 192.168.5.1
Mask: 255.255.255.0
```
This will act as the **gateway** for your Kubernetes VMs.

---

# 3️⃣ Create NAT Entry

Run in PowerShell (Admin):

```powershell
New-NetNAT -Name "natK8sCluster" -InternalIPInterfaceAddressPrefix 192.168.5.0/24
```
This enables routing from your VMs to outside.

---

## ❗ Error: NAT Already Exists

If you get:

```
Instance already exists
```

Check existing NAT:

```powershell
Get-NetNAT
```
You’ll see something like:  
`Name : natK8sCluster InternalIPInterfaceAddressPrefix : 192.168.5.0/24`

### Two solutions:

### ✅ Option A — Use Existing NAT

If the NAT entry already shows:  
`natK8sCluster` with `192.168.5.0/24`
  
and that matches the subnet you configured for:  
`vS_k8sCluster IP: 192.168.5.1 Mask: 255.255.255.0`
continue.

### ✅ Option B — Remove and Recreate

```powershell
Remove-NetNAT -Name "natK8sCluster"
New-NetNAT -Name "natK8sCluster" -InternalIPInterfaceAddressPrefix 192.168.5.0/24
```

---

# 4️⃣ Create Ubuntu Control Plane VM

VM settings:

- Generation 1
- **Static memory** (disable dynamic)
    
- 2 CPUs
    
- 4GB RAM (minimum)
    
- 20GB disk
    
- Network: `vS_k8sCluster`
    

Install Ubuntu Server 22.04 with static ip.

---

# 5️⃣ Configure Static IP in Ubuntu (Not working for internal switch)

Find interface:

```bash
ip a
```

Edit netplan:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Example config:

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.5.10/24
      routes:
        - to: default
          via: 192.168.5.1
      nameservers:
        addresses:
          - 8.8.8.8
```

| Line             | Meaning                |
| ---------------- | ---------------------- |
| dhcp4: no        | Disable automatic IP   |
| 192.168.5.10/24  | Static IP for VM       |
| via: 192.168.5.1 | Windows host = gateway |
| 8.8.8.8          | Google DNS             |

Apply:

```bash
sudo netplan apply
```

Verify IP Address

Run:

```bash
ip a
```

You should now see:

```
inet 192.168.5.10/24
```

### Test Internet access (IP only):

```bash
ping 8.8.8.8
```

If it replies → NAT works.

---

### Test DNS resolution:

```bash
ping google.com
```

If this works → DNS is working.


---

# 6️⃣ Debugging: Cannot Ping Gateway

If:

```bash
ping 192.168.5.1
```

does not work:

Check:

### Windows:

```powershell
ipconfig
```

Verify:

```
vEthernet (vS_k8sCluster) = 192.168.5.1
```

### Ubuntu:

```bash
ip route
```

Should show:

```
default via 192.168.5.1
```

### Common causes:

- VM attached to wrong switch
    
- Windows firewall blocking ICMP
	Sometimes Windows blocks ping.
	
	On Windows:
	
	Open **Windows Defender Firewall**  
	→ Advanced Settings  
	→ Inbound Rules  
	→ Enable:
	
	```
	File and Printer Sharing (Echo Request - ICMPv4-In)
	```
	
	OR temporarily test by disabling firewall (just for testing):
	
	```powershell
	Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
	```
	
	Test ping again.
	
	If it works → firewall was blocking it.
	
	Re-enable firewall after testing:
	
	```powershell
	Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True
	```
- Incorrect subnet
    

---

# 7️⃣ DNS Shows 127.0.0.53

When checking:

```bash
cat /etc/resolv.conf
```

You see:

```
nameserver 127.0.0.53
```

This is normal in Ubuntu 22.04.

Ubuntu uses a local DNS stub resolver (`systemd-resolved`).

Real DNS server is set in netplan.

Verify:

```bash
resolvectl status
```

If DNS fails:

- Ensure `nameservers:` is defined in netplan. (nameserver 8.8.8.8)
    
- Do NOT manually edit `/etc/resolv.conf`.
    

---
# **Shutdown the VM** and give it **2 virtual CPUs** in Hyper-V settings.


# 1️⃣3️⃣ Setup SSH From Mac to VM

Install SSH in Ubuntu:

```bash
sudo apt update
sudo apt install openssh-server -y
```
Check it is running:

```bash
sudo systemctl status ssh
```

You should see:

```
Active: active (running)
```

If not running:

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

Allow SSH Through Ubuntu Firewall (If UFW Enabled)

Check:

```bash
sudo ufw status
```

If active:

```bash
sudo ufw allow ssh
```

Since using Internal switch, Mac cannot see VM directly.

Use port forwarding:

Confirm Ubuntu VM IP

Inside Ubuntu:

```bash
ip a
```

You should see:

```
192.168.5.10
```

Remember this.

On Windows:

```powershell
netsh interface portproxy add v4tov4 listenport=2222 listenaddress=0.0.0.0 connectport=22 connectaddress=192.168.5.10
```
Now check:

```powershell
netsh interface portproxy show all
```

You should see:

```
0.0.0.0    2222    192.168.5.10    22
```

Allow firewall:

```powershell
New-NetFirewallRule -DisplayName "SSH to Ubuntu VM" -Direction Inbound -LocalPort 2222 -Protocol TCP -Action Allow
```

Get Your Windows Laptop IP (Visible to Mac)

On Windows (PowerShell):

```powershell
ipconfig
```

Look for your WiFi adapter:

Example:

```
IPv4 Address: 192.168.1.45
```

This is the IP your Mac can see.

From Mac:

```bash
ssh username@WINDOWS_IP -p 2222
```
Example:

```bash
ssh ubuntu@192.168.1.45 -p 2222
```

---



# 8️⃣ Install Kubernetes Components

Disable swap:

```bash
swapoff -a
```
Then edit `/etc/fstab` and **comment** the swap line.

Update system:

```bash
apt-get update && apt-get upgrade -y
```

Install required packages:

```bash
apt-get install -y apt-transport-https ca-certificates curl gpg
```

Add Kubernetes repo and key.
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo \
  "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
  https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
Install:

```bash
apt-get update
apt-get install -y kubeadm kubelet kubectl
apt-mark hold kubeadm kubelet kubectl
```

### Load Kernel Modules & Sysctl Config

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

Check:

```bash
sysctl net.bridge.bridge-nf-call-iptables net.ipv4.ip_forward
```
and confirm values are `1`.

---

# 9️⃣ Install containerd

(_Also inside the same VM_)

1. Add Docker GPG key:
    
    ```bash
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
      | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    ```
    
    ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-ii-bd7e4e0c9555 "Setting up a K8s cluster using Ubuntu and Hyper-V P2 | Medium"))
    
2. Add Docker repo:
    
    ```bash
    echo \
      "deb [signed-by=/etc/apt/keyrings/docker.gpg arch=amd64] \
      https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
      | sudo tee /etc/apt/sources.list.d/containerd.list
    ```
    
    ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-ii-bd7e4e0c9555 "Setting up a K8s cluster using Ubuntu and Hyper-V P2 | Medium"))
    
3. Update & install containerd:
    
    ```bash
    apt-get update
    apt-get install -y containerd
    ```
    
    ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-ii-bd7e4e0c9555 "Setting up a K8s cluster using Ubuntu and Hyper-V P2 | Medium"))
    
4. Generate default config and adjust:
    
    ```bash
    mkdir -p /etc/containerd
    containerd config default | sudo tee /etc/containerd/config.toml
    ```
    
    ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-ii-bd7e4e0c9555 "Setting up a K8s cluster using Ubuntu and Hyper-V P2 | Medium"))
    
    In `/etc/containerd/config.toml`:
    
    - Change `sandbox_image` to `pause:3.9`  
        
    - Find the section `[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]`
    - Set `SystemdCgroup = true`  
        ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-ii-bd7e4e0c9555 "Setting up a K8s cluster using Ubuntu and Hyper-V P2 | Medium"))  
        
5. Restart containerd:
    
    ```bash
    sudo systemctl restart containerd
    ```
    
    ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-ii-bd7e4e0c9555 "Setting up a K8s cluster using Ubuntu and Hyper-V P2 | Medium"))
    
6. Update crictl endpoint:
    
    ```bash
    sudo crictl config --set runtime-endpoint=unix:////var/run/containerd/containerd.sock
    ```
    
    ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-ii-bd7e4e0c9555 "Setting up a K8s cluster using Ubuntu and Hyper-V P2 | Medium"))
---

# 🔟 Initialize Control Plane

Optionally, add DNS alias in `/etc/hosts`:
```
192.168.5.10 ckak8scp
```

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

If use Calico as CNI then the command would be
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

After init:

```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Verify:
```bash
kubectl get nodes -o wide
kubectl get pods -A
```
Initially `coredns` will be `PENDING`.

Node will show:

```
NotReady
```

This is expected (no CNI yet).

Install bash completion and kubectl autocompletion (optional but recommended):
```bash
sudo apt-get install bash-completion -y
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

---
# **Install Cilium CNI Plugin**

(To allow pod networking so Control Plane becomes `Ready`)

1. Install Helm:
    
    ```bash
    wget https://get.helm.sh/helm-v3.12.0-linux-amd64.tar.gz
    tar -xvf helm-v3.12.0-linux-amd64.tar.gz
    sudo cp linux-amd64/helm /usr/local/bin/helm
    ```
    
    ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iii-f2d2bb804d09 "Setting up a K8s cluster using Ubuntu and Hyper-V P3 | Medium"))
    
2. Add and update Cilium Helm repo:
    
    ```bash
    helm repo add cilium https://helm.cilium.io
    helm repo update
    ```
    
    ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iii-f2d2bb804d09 "Setting up a K8s cluster using Ubuntu and Hyper-V P3 | Medium"))
    
3. Generate Cilium manifest matching your pod CIDR:
    
    ```bash
    helm template cilium cilium/cilium --version 1.15.5 --namespace kube-system > cilium.yaml
    ```
    
    Edit `cilium.yaml` and set `cluster-pool-ipv4-cidr` to match `podSubnet` you chose earlier. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iii-f2d2bb804d09 "Setting up a K8s cluster using Ubuntu and Hyper-V P3 | Medium"))
    
4. Deploy:
    
    ```bash
    kubectl apply -f cilium.yaml
    ```
    
    ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iii-f2d2bb804d09 "Setting up a K8s cluster using Ubuntu and Hyper-V P3 | Medium"))
    
5. Check status:
    
    ```bash
    kubectl get nodes -o wide
    kubectl get pods -A
    ```
    
    Control plane node should now be `Ready`.
# 1️⃣1️⃣ Install Calico Instead of Cilium

Apply Calico:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml
```

---

## ❗ Important: CIDR Must Match

If kubeadm used:

```
10.244.0.0/16
```

Download and edit:

```bash
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml
nano calico.yaml
```

Find:

```
CALICO_IPV4POOL_CIDR
```

Change to:

```
10.244.0.0/16
```

Apply:

```bash
kubectl apply -f calico.yaml
```

---
# Verify

Wait 1–2 minutes, then:

```bash
kubectl get pods -A
```

You should see:

```
calico-node-xxxxx             Running
calico-kube-controllers       Running
coredns-xxxxx                 Running
```

If calico-kube-controllers is showing ImagePullBackOff error then manually download the image using

```
sudo ctr images pull docker.io/calico/kube-controllers:v3.27.2
```

Then check node:

```bash
kubectl get nodes
```

You should see:

```
STATUS: Ready
```
# 🚨 If You Already Applied Wrong Calico

If you applied Calico with the wrong CIDR and pods are stuck:

### Remove it first:

```bash
kubectl delete -f calico.yaml
```

Wait until calico pods disappear:

```bash
kubectl get pods -A
```

Then edit and reapply.

---

# 🧠 Why This Is Important

Kubernetes assigns pod IPs from:

```
--pod-network-cidr
```

Calico creates its IP pool from:

```
CALICO_IPV4POOL_CIDR
```

If they don’t match:

- Nodes stay `NotReady`  
    
- Pods get stuck  
    
- Networking fails completely  
    
---
# 🎯 Quick Example Scenario

If you used:

```bash
kubeadm init --pod-network-cidr=172.16.0.0/16
```

Then set:

```yaml
value: "172.16.0.0/16"
```

That’s it.

---

If you paste your actual pod CIDR here, I will tell you exactly what value to put in calico.yaml.

---

# 🚀 After Calico Is Working

You continue exactly as the guide says:

- Create worker node  
    
- Run `kubeadm join`  
    
- Verify both nodes  
    
- Deploy sample application  
    

No other changes required.

---

# 🎯 Why Calico Instead of Cilium?

|Calico|Cilium|
|---|---|
|Simpler|More advanced|
|Widely used|eBPF-based|
|Good for labs|Production-grade observability|

For learning Kubernetes fundamentals, Calico is perfectly fine.

---
# 1️⃣4️⃣ Worker Node Setup

Create second VM.

Repeat:

- Disable swap
    
- Install Kubernetes
    
- Install containerd
	
- Configure network modules  
    This is essentially a cloned version of the control plane setup steps.

Do NOT run `kubeadm init`.

---

# ❗ Mistake: Ran `kubeadm init` On Worker

Fix:

```bash
sudo kubeadm reset -f
sudo rm -rf /etc/kubernetes/
sudo rm -rf /etc/cni/net.d
sudo iptables -F
sudo reboot
```

Then from control plane:

```bash
kubeadm token create --print-join-command
```

# 🤝 **STEP 8 — Join Worker Node to Cluster**

(On the _worker node_ VM)

1. On the _control plane node_, Kubernetes will output a join command after `kubeadm init`. It looks like:
    
    ```
    kubeadm join --token <token> <control-plane-host>:6443 \
      --discovery-token-ca-cert-hash sha256:<hash>
    ```

```bash
kubeadm token create --print-join-command
```
    
    Copy that text. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iv-31ad24098090 "Setting up a K8s cluster using Ubuntu and Hyper-V P4 | Medium"))
    
2. On the _worker VM_:
    
    ```bash
    sudo kubeadm join --token <token> ckak8scp:6443 \
      --discovery-token-ca-cert-hash sha256:<hash>
    ```
    
    Replace `<token>` and `<hash>` with actual values from the control plane output. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iv-31ad24098090 "Setting up a K8s cluster using Ubuntu and Hyper-V P4 | Medium"))
    
3. Verify on control plane:
    
    ```bash
    kubectl get nodes -o wide
    ```
    
    Both nodes should appear.

---

# 1️⃣5️⃣ Final Verification

On control plane:

```bash
kubectl get nodes
kubectl get pods -A
```

Expected:

```
control-plane   Ready
worker-node     Ready
```

Calico pods running.

CoreDNS running.

# 🚀 **STEP 9 — Deploy Example App (podinfo)**

(This verifies cluster functionality.) ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iv-31ad24098090 "Setting up a K8s cluster using Ubuntu and Hyper-V P4 | Medium"))

1. On control plane node:
    
    ```bash
    helm repo add podinfo https://stefanprodan.github.io/podinfo
    helm upgrade -i podinfo-deploy podinfo/podinfo
    ```
    
    ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iv-31ad24098090 "Setting up a K8s cluster using Ubuntu and Hyper-V P4 | Medium"))
    
2. Check application pods:
    
    ```bash
    kubectl get pods -o wide
    ```
    
    ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iv-31ad24098090 "Setting up a K8s cluster using Ubuntu and Hyper-V P4 | Medium"))
    
3. Change service type from `ClusterIP` to `NodePort`:
    
    ```bash
    kubectl edit svc podinfo-deploy
    ```
    
    Change `type: ClusterIP` → `type: NodePort`. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iv-31ad24098090 "Setting up a K8s cluster using Ubuntu and Hyper-V P4 | Medium"))
    
4. Expose app externally:
    
    ```bash
    kubectl get svc podinfo-deploy
    ```
    
    Note the NodePort (e.g., above 30000).  
    From your local machine, access:
    
    ```
    http://192.168.5.20:<NodePort>
    ```
    
    If it loads, your cluster is working with a deployed application.
---

# Key Lessons From This Lab

- `127.0.0.53` in resolv.conf is normal.
    
- `ImagePullBackOff` is almost always networking.
    
- NAT conflicts must be checked with `Get-NetNAT`.
    
- Only ONE node runs `kubeadm init`.
    
- Calico CIDR must match `--pod-network-cidr`.
    
- External Virtual Switch simplifies everything.
    
- Internal switch requires NAT + routing + firewall handling.
    

---

# Final Architecture

Windows 10 (Hyper-V)  
→ Ubuntu Control Plane  
→ Ubuntu Worker Node  
→ containerd  
→ kubeadm  
→ Calico CNI  
→ SSH access from Mac  
→ Working Kubernetes cluster

---
