https://chatgpt.com/share/69a891f1-d844-8005-af4e-f2c227c3eace

Here is a **clear, exact, step-by-step instruction set** based _only_ on what the four Medium articles describe (Parts I–IV by Enrique Torres). I’ve **not added assumptions outside those docs and referenced what each step comes from** so you can follow precisely. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))

---

# 📌 **Prerequisites**

Before you begin, ensure:
✔ You are starting from **Windows 10 Pro (or Enterprise/Education)** with virtualization available. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))  
✔ You have a **Dell laptop** with enough CPU/RAM/disk — recommended **≥8 GiB RAM per Ubuntu VM**, **20 GiB disk each**. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))  
✔ You have downloaded **Ubuntu 22.04 Server ISO** from ubuntu.com (manual install). ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))  
✔ You will use **Hyper-V** as the VM provider. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))

---

# 🧱 **STEP 1 — Enable Hyper-V and Networking on Windows**

(This is about preparing Windows to host VMs.)

1. Go to **Settings → Apps → Optional features → More Windows features**.
    
2. Enable:
    
    - **Hyper-V Management Tools**
        
    - **Hyper-V Platform**  
        Click **OK** and reboot. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))
        
3. **Enable IP routing** on the Windows host (PowerShell _as Administrator_):
    
    ```powershell
    Set-ItemProperty -Path HKLM:\system\CurrentControlSet\services\Tcpip\Parameters -Name IpEnableRouter -Value 1
    ```
    
    Reboot. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))
    
4. In **Hyper-V Manager → Virtual Switch Manager**, create an **Internal virtual switch** (e.g., named `vS_k8sCluster`). ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))
    
5. Assign a static IP on the newly created virtual adapter in **Network Connections → Properties → IPv4**.  
    Example chosen in the article:
    
    ```
    192.168.5.1/24
    ```
    
    This will act as the **gateway** for your Kubernetes VMs. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))
    
6. Add a NAT entry so VMs can reach the internet:
    
    ```powershell
    New-NetNAT -Name "natK8sCluster" -InternalIPInterfaceAddressPrefix 192.168.5.0/24
    ```
    
    This enables routing from your VMs to outside. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))

	If an error that it is already exist pops up.
		If you get an error  because a NAT network with the same name or IP prefix already exists in Windows.
		
		This step comes from **Part I** of the guide (the Hyper-V + NAT setup). The command used there is:
		
		```powershell
		New-NetNAT -Name "natK8sCluster" -InternalIPInterfaceAddressPrefix 192.168.5.0/24
		```
		
		If PowerShell says it already exists, do the following.
		
		---
		
		# ✅ Step 1 — Check Existing NAT Entries
		
		Open **PowerShell as Administrator** and run:
		
		```powershell
		Get-NetNAT
		```
		
		You’ll see something like:
		
		```
		Name           : natK8sCluster
		InternalIPInterfaceAddressPrefix : 192.168.5.0/24
		```
		
		---
		
		# ✅ Step 2 — Decide What to Do
		
		There are **two safe solutions**.
		
		---
		
		## 🔹 Option A (Recommended): Use the Existing NAT
		
		If the NAT entry already shows:
		
		```
		192.168.5.0/24
		```
		
		and that matches the subnet you configured for:
		
		```
		vS_k8sCluster
		IP: 192.168.5.1
		Mask: 255.255.255.0
		```
		
		👉 Then **do nothing**.  
		You already have NAT configured correctly. Continue to the next step in the guide.
		
		---
		
		## 🔹 Option B: Remove and Recreate It (Clean Fix)
		
		If:
		
		- The prefix is wrong
		    
		- The name conflicts
		    
		- Or it was created incorrectly
		    
		
		Then remove it:
		
		```powershell
		Remove-NetNAT -Name "natK8sCluster"
		```
		
		Now recreate it:
		
		```powershell
		New-NetNAT -Name "natK8sCluster" -InternalIPInterfaceAddressPrefix 192.168.5.0/24
		```
		
		---
		
		# 🚨 If You Get “Prefix Already Exists” Error
		
		Sometimes Windows auto-creates a NAT for Hyper-V (like `Default Switch`).
		
		Check:
		
		```powershell
		Get-NetNAT | Format-List
		```
		
		If another NAT is already using:
		
		```
		192.168.5.0/24
		```
		
		You must either:
		
		### Option 1 — Use That Existing Prefix
		
		or
		
		### Option 2 — Change Your Cluster Subnet
		
		For example, choose a new subnet:
		
		```
		192.168.100.0/24
		```
		
		Then:
		
		1. Change the virtual adapter IP to:
		    
		    ```
		    192.168.100.1
		    ```
		    
		2. Create NAT:
		    
		
		```powershell
		New-NetNAT -Name "natK8sCluster" -InternalIPInterfaceAddressPrefix 192.168.100.0/24
		```
		
		⚠ If you change the subnet, you must also change:
		
		- Control plane static IP
		    
		- Worker node IPs
		    
		- kubeadm config (if already created)
		    
		
		---
		
		# 🎯 What I Recommend For You
		
		Run:
		
		```powershell
		Get-NetNAT
		```
		
		If you see `natK8sCluster` with `192.168.5.0/24`,  
		👉 **You are already correctly configured. Continue with VM setup.**
		
		If not, paste the output here and I’ll tell you exactly what to remove or modify.
		
		---
		
		Since you're building this for Kubernetes practice (and preparing for AWS SAA), getting this networking layer clean is important — this NAT is what allows:
		
		- `apt update`
		    
		- pulling container images
		    
		- Helm downloads
		    
		- Cilium install
		    
		
		So let’s make sure it’s correct before moving forward.

---

# 🐧 **STEP 2 — Create Control Plane VM and Install Ubuntu**

(For the Kubernetes Control Plane node.)

1. In **Hyper-V Manager → New Virtual Machine**:
    
    - **Generation 1**
        
    - **Static memory** (disable dynamic)
        
    - **2 CPUs**
        
    - **4 GiB RAM** (minimum), recommended more
        
    - **20 GiB VHDX disk**
        
    - Connect to `vS_k8sCluster`
        
    - Mount downloaded Ubuntu 22.04 ISO  
        ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))
        
2. Install **Ubuntu server** normally.
    
3. In Ubuntu:
    
    - Assign a **static IP** for this VM (e.g., `192.168.5.10/24`), Gateway → `192.168.5.1` (your Windows host virtual switch), DNS → `8.8.8.8`
			# 🧠 Why We Must Do This

			Your Hyper-V Internal Switch has:
			
			- Windows host IP → `192.168.5.1`
			    
			- NAT network → `192.168.5.0/24`
			    
			
			So your Ubuntu VM must:
			
			- Be in the same subnet
			    
			- Use Windows host as gateway
			    
			- Have DNS so it can download packages
						
			# ✅ STEP 1 — Find Your Network Interface Name
			
			Inside Ubuntu VM, run:
			
			```bash
			ip a
			```
			
			You will see something like:
			
			```
			2: eth0:
			```
			
			or
			
			```
			2: ens33:
			```
			
			or
			
			```
			2: enp0s3:
			```
			
			👉 **Remember this interface name** (example: `eth0`)
			
			We will use it in the next step.
			
			---
			
			# ✅ STEP 2 — Edit Netplan Configuration
			
			Ubuntu Server uses **Netplan** for networking.
			
			Check existing config:
			
			```bash
			ls /etc/netplan/
			```
			
			You will see something like:
			
			```
			00-installer-config.yaml
			```
			
			Open it:
			
			```bash
			sudo nano /etc/netplan/00-installer-config.yaml
			```
			
			---
			
			# ✅ STEP 3 — Replace With Static IP Configuration
			
			Now modify the file to look like this:
			
			(Replace `eth0` with YOUR interface name)
			
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
			
			### Explanation:
			
			|Line|Meaning|
			|---|---|
			|dhcp4: no|Disable automatic IP|
			|192.168.5.10/24|Static IP for VM|
			|via: 192.168.5.1|Windows host = gateway|
			|8.8.8.8|Google DNS|
			
			---
			
			# ✅ STEP 4 — Apply the Configuration
			
			Save the file (CTRL + X → Y → Enter)
			
			Then run:
			
			```bash
			sudo netplan apply
			```
			
			If no errors appear, it worked.
			
			---
			
			# ✅ STEP 5 — Verify IP Address
			
			Run:
			
			```bash
			ip a
			```
			
			You should now see:
			
			```
			inet 192.168.5.10/24
			```
			
			---
			
			# ✅ STEP 6 — Test Network Connectivity
			
			### Test connection to Windows host (gateway):
			
			```bash
			ping 192.168.5.1
			```
			
			If it replies → local network works.
			ping 192.168.5.1 is not working. no response returned. what to do?
			Good — this tells us something is wrong in the **Windows ↔ Hyper-V ↔ Ubuntu networking layer**.
			
			If `ping 192.168.5.1` does not work, the Ubuntu VM cannot reach the Windows host virtual adapter.
			
			We will debug this **step by step**.
			
			Do not skip steps.
			
			---
			
			# ✅ STEP 1 — Verify Windows Virtual Switch IP
			
			On **Windows (not inside Ubuntu)**:
			
			Open **PowerShell as Administrator** and run:
			
			```powershell
			ipconfig
			```
			
			Look for something like:
			
			```
			Ethernet adapter vEthernet (vS_k8sCluster):
			
			   IPv4 Address . . . . . . . . . . : 192.168.5.1
			   Subnet Mask  . . . . . . . . . . : 255.255.255.0
			```
			
			### ❓ Check:
			
			- Does `vEthernet (vS_k8sCluster)` exist?
			    
			- Is the IP really `192.168.5.1`?
			    
			
			---
			
			### 🔴 If NO virtual adapter exists
			
			You did not create the **Internal Virtual Switch** correctly.
			
			Go to:
			
			```
			Hyper-V Manager → Virtual Switch Manager
			```
			
			Make sure:
			
			- Type = **Internal**
			    
			- Name = `vS_k8sCluster`
			    
			
			After creating it, Windows automatically creates the `vEthernet` adapter.
			
			Then manually assign:
			
			```
			192.168.5.1
			Subnet: 255.255.255.0
			```
			
			---
			
			# ✅ STEP 2 — Verify Ubuntu Static IP Configuration
			
			Inside Ubuntu VM:
			
			Run:
			
			```bash
			ip a
			```
			
			You should see:
			
			```
			inet 192.168.5.10/24
			```
			
			If you DO NOT see that, your netplan config is wrong.
			
			If correct, continue.
			
			---
			
			# ✅ STEP 3 — Verify Ubuntu Routing Table
			
			Inside Ubuntu:
			
			```bash
			ip route
			```
			
			You must see:
			
			```
			default via 192.168.5.1 dev eth0
			192.168.5.0/24 dev eth0 proto kernel scope link src 192.168.5.10
			```
			
			### ❓ If default route is missing
			
			Your netplan config is wrong.
			
			Fix it to:
			
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
			        addresses: [8.8.8.8]
			```
			
			Then:
			
			```bash
			sudo netplan apply
			```
			
			---
			
			# ✅ STEP 4 — Verify VM Is Connected to Correct Switch
			
			This is VERY COMMON mistake.
			
			In **Hyper-V Manager**:
			
			1. Select your Ubuntu VM
			    
			2. Click **Settings**
			    
			3. Click **Network Adapter**
			    
			4. Confirm:
			    
			
			```
			Virtual Switch: vS_k8sCluster
			```
			
			If it says:
			
			- `Default Switch`
			    
			- or something else
			    
			
			👉 Change it to `vS_k8sCluster`  
			👉 Apply  
			👉 Restart VM
			
			---
			
			# ✅ STEP 5 — Check Windows Firewall (Important)
			
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
			
			---
			
			# ✅ STEP 6 — Check NAT Exists (Optional but Good)
			
			On Windows:
			
			```powershell
			Get-NetNAT
			```
			
			You should see:
			
			```
			natK8sCluster
			InternalIPInterfaceAddressPrefix : 192.168.5.0/24
			```
			
			---
			
			# 🚨 Most Common Causes (From Experience)
			
			If ping fails, it is usually:
			
			1. ❌ VM connected to wrong switch
			    
			2. ❌ Windows virtual adapter has no IP
			    
			3. ❌ Wrong subnet (example: Ubuntu is 192.168.0.x but Windows is 192.168.5.x)
			    
			4. ❌ Firewall blocking ICMP
			    
			5. ❌ Netplan default route missing
			    
			
			---
			
			# 🎯 What I Need From You
			
			Please paste:
			
			From Ubuntu:
			
			```
			ip a
			ip route
			```
			
			From Windows PowerShell:
			
			```
			ipconfig
			Get-NetNAT
			```
			
			Then I will tell you exactly what is wrong in your setup.
			
			We’ll fix it step by step.
			
			---
			
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
			
			
			        
	- Set **DNS server IP** in `/etc/resolv.conf` if needed (e.g., `8.8.8.8`) so it can resolve external names for package downloads. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))
			
			
			    
			
						---
			# ❗ About `/etc/resolv.conf`
			
			In the Medium guide, they mention setting DNS in `/etc/resolv.conf`.
			
			⚠ IMPORTANT:  
			In modern Ubuntu (22.04), **you should NOT manually edit `/etc/resolv.conf`** because:
			
			- It is automatically managed
			    
			- It will be overwritten on reboot
			    
			
			Instead, setting DNS in the **netplan file (as shown above)** is the correct method.
			
			Only if DNS fails, check:
			
			```bash
			cat /etc/resolv.conf
			```
			
			It should contain:
			
			```
			nameserver 8.8.8.8
			```
			
			If not, your netplan config may be wrong.
			
			---
			
			# 🧠 Final Result Should Be:
			
			Your VM network should be:
			
			|Component|Value|
			|---|---|
			|Control Plane IP|192.168.5.10|
			|Worker Node IP|192.168.5.20 (later)|
			|Gateway|192.168.5.1|
			|Subnet|192.168.5.0/24|
			|DNS|8.8.8.8|
			
			---
			
			# 🚀 After This Works
			
			You can safely continue with:
			
			- `apt update`
			    
			- Installing Kubernetes
			    
			- Installing containerd
			    
			- Running kubeadm init
			    
			
			---
			
			If you want, next I can:
			
			- Draw a simple diagram of your Windows + Hyper-V + Kubernetes network  
			    OR
			    
			- Help you configure the Worker node static IP correctly
			    
			
			Tell me where you are now.
        
4. **Shutdown the VM** and give it **2 virtual CPUs** in Hyper-V settings. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))
    
# set up **SSH from your Mac → Ubuntu VM
Perfect 👍 — now we will set up **SSH from your Mac → Ubuntu VM (Control Plane VM running in Hyper-V on your Windows laptop).**

Since your VM is on an **Internal Hyper-V switch (192.168.5.0/24)**, your Mac cannot directly see that network.

So we must expose SSH properly.

I will give you the **correct working method** step-by-step.

---

# 🧠 First Understand the Network Situation

Right now:

```
Mac  ❌ cannot see 192.168.5.10
Windows Host 192.168.5.1
Ubuntu VM 192.168.5.10
```

Because:

- You created an **Internal Virtual Switch**
    
- That network exists only inside Windows
    

So we must either:

1. Use **External Virtual Switch** (simpler)  
    OR
    
2. Use **Port forwarding on Windows** (works with Internal switch)
    

Since the guide uses Internal switch, we will use **port forwarding**.

---

# ✅ STEP 1 — Install SSH Server on Ubuntu VM

Inside your Ubuntu VM:

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

---

# ✅ STEP 2 — Allow SSH Through Ubuntu Firewall (If UFW Enabled)

Check:

```bash
sudo ufw status
```

If active:

```bash
sudo ufw allow ssh
```

---

# ✅ STEP 3 — Confirm Ubuntu VM IP

Inside Ubuntu:

```bash
ip a
```

You should see:

```
192.168.5.10
```

Remember this.

---

# ✅ STEP 4 — Get Your Windows Laptop IP (Visible to Mac)

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

---

# ✅ STEP 5 — Create Port Forwarding on Windows

We now tell Windows:

👉 When someone connects to port 2222 on Windows  
👉 Forward it to 192.168.5.10 port 22

Open **PowerShell as Administrator**:

```powershell
netsh interface portproxy add v4tov4 ^
listenport=2222 ^
listenaddress=0.0.0.0 ^
connectport=22 ^
connectaddress=192.168.5.10
```

Now check:

```powershell
netsh interface portproxy show all
```

You should see:

```
0.0.0.0    2222    192.168.5.10    22
```

---

# ✅ STEP 6 — Allow Port 2222 in Windows Firewall

Run:

```powershell
New-NetFirewallRule -DisplayName "SSH to Ubuntu VM" `
-Direction Inbound `
-LocalPort 2222 `
-Protocol TCP `
-Action Allow
```

---

# ✅ STEP 7 — Connect From Your Mac

On your Mac terminal:

```bash
ssh username@WINDOWS_IP -p 2222
```

Example:

```bash
ssh ubuntu@192.168.1.45 -p 2222
```

Replace:

- `ubuntu` → your Ubuntu username
    
- `192.168.1.45` → your Windows WiFi IP
    

First time you’ll see:

```
Are you sure you want to continue connecting?
```

Type:

```
yes
```

Enter password.

You are now inside your VM 🎉

---

# 🔍 If It Fails

On Windows check:

```powershell
netsh interface portproxy show all
```

On Ubuntu check:

```bash
sudo systemctl status ssh
```

---

# 🎯 Alternative (Easier Method)

If you want simpler networking:

Instead of Internal switch, create an **External Virtual Switch** in:

```
Hyper-V Manager → Virtual Switch Manager → External
```

Attach VM to it.

Then your VM will get IP like:

```
192.168.1.x
```

Same network as your Mac.

Then you can directly:

```bash
ssh ubuntu@192.168.1.x
```

No port forwarding needed.

---

# 🚀 My Recommendation For You

Since you're building a Kubernetes lab and preparing seriously:

👉 Use **External switch** for easier SSH and testing NodePort services.

It will save you many networking headaches later.

---

If you tell me:

- Your Windows WiFi IP
    
- Your Ubuntu VM IP
    

I’ll confirm the exact SSH command you should run.
---

# 🛠 **STEP 3 — Install Kubernetes Dependencies on the Control Plane**

(_Inside the Ubuntu VM, as root_ — control plane node)

### Disable Swap

Kubernetes doesn’t allow swap:

```bash
swapoff -a
```

Then edit `/etc/fstab` and **comment** the swap line. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))

---

### System Update

```bash
apt-get update && apt-get upgrade -y
```

([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))

---

### Install prereq packages for Kubernetes repo

```bash
apt-get install -y apt-transport-https ca-certificates curl gpg
```

([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))

---

### Add Kubernetes Signing Key and Repo

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo \
  "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
  https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))

---

### Install Kubernetes Components

```bash
apt-get update
apt-get install -y kubeadm kubelet kubectl
apt-mark hold kubeadm kubelet kubectl
```

([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))

---

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

and confirm values are `1`. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-i-8719f79d045e?utm_source=chatgpt.com "Setting up a K8s cluster using Ubuntu and Hyper-V P1 | Medium"))

---

# 🐳 **STEP 4 — Install Containerd (Container Runtime)**

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
    
    In `/etc/containerd/config.toml`:
    
    - Change `sandbox_image` to `pause:3.9`
        
    - Set `SystemdCgroup = true`  
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

# 🧠 **STEP 5 — Bootstrap the Control Plane**

(_Inside the same Ubuntu VM_)

1. Optionally, add DNS alias in `/etc/hosts`:
    
    ```
    192.168.5.10 ckak8scp
    ```
    
    ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iii-f2d2bb804d09 "Setting up a K8s cluster using Ubuntu and Hyper-V P3 | Medium"))
    
2. Create **kubeadm config file** (you can copy example YAML from Part III and set values such as `controlPlaneEndpoint` and `podSubnet`). ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iii-f2d2bb804d09 "Setting up a K8s cluster using Ubuntu and Hyper-V P3 | Medium"))
    
3. Initialize control plane:
    
    ```bash
    sudo kubeadm init \
      --config=kubeadm-config.yaml \
      --upload-certs \
      --ignore-preflight-errors=all | tee kubeadm-init.out
    ```
    
    ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iii-f2d2bb804d09 "Setting up a K8s cluster using Ubuntu and Hyper-V P3 | Medium"))
    
4. Set up kubectl for a normal user (e.g., `ubuntu`):
    
    ```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
    
    ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iii-f2d2bb804d09 "Setting up a K8s cluster using Ubuntu and Hyper-V P3 | Medium"))
    
5. Install bash completion and kubectl autocompletion (optional but recommended):
    
    ```bash
    sudo apt-get install bash-completion -y
    source <(kubectl completion bash)
    echo "source <(kubectl completion bash)" >> ~/.bashrc
    ```
    
    ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iii-f2d2bb804d09 "Setting up a K8s cluster using Ubuntu and Hyper-V P3 | Medium"))
    
6. Verify:
    
    ```bash
    kubectl get nodes -o wide
    kubectl get pods -A
    ```
    
    Initially `coredns` will be `PENDING`. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iii-f2d2bb804d09 "Setting up a K8s cluster using Ubuntu and Hyper-V P3 | Medium"))
    

---

# 🌐 **STEP 6 — Install Cilium CNI Plugin**

(To allow pod networking so Control Plane becomes `Ready`)

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
    
    Edit `cilium.yaml` and set `cluster-pool-ipv4-cidr` to match `podSubnet` you chose earlier. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iii-f2d2bb804d09 "Setting up a K8s cluster using Ubuntu and Hyper-V P3 | Medium"))
    
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
    
    Control plane node should now be `Ready`. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iii-f2d2bb804d09 "Setting up a K8s cluster using Ubuntu and Hyper-V P3 | Medium"))
    

---

# Install Calico Instead of Cilium
Perfect 👍 — we will replace **Cilium (STEP 6)** with **Calico CNI**, but everything else from the guide remains the same.

You are modifying only the networking plugin step after:

```
kubeadm init
```

and after configuring kubectl.

---

# 🧠 Where We Are in the Process

You already ran:

```bash
sudo kubeadm init ...
```

When you run:

```bash
kubectl get nodes
```

You will see:

```
STATUS: NotReady
```

This is normal because **no CNI plugin is installed yet**.

Instead of installing Cilium (as the article does), we will install:

👉 **Calico**

---

# ✅ STEP 6 — Install Calico CNI Plugin

⚠ IMPORTANT:  
Calico must match the **pod subnet** you used in `kubeadm init`.

If you followed the guide, the pod subnet was likely:

```
192.168.0.0/16
```

If unsure, check:

```bash
cat kubeadm-config.yaml
```

Look for:

```yaml
podSubnet: 192.168.0.0/16
```

---

## ✅ STEP 6.1 — Apply Official Calico Manifest

On the **control plane node**, run:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml
```

This will:

- Create calico-system namespace
    
- Install calico-node daemonset
    
- Install calico-kube-controllers
    
- Configure networking
    

---

## 🧠 What This Does

Calico will:

- Create a virtual overlay network between nodes
    
- Allow Pods to communicate
    
- Enable routing between control plane and worker nodes
    

Without this, Kubernetes cannot schedule pods.

---

# ✅ STEP 6.2 — Verify Installation

Immediately after applying, check:

```bash
kubectl get pods -A
```

You should see something like:

```
calico-node-xxxxx         Running
calico-kube-controllers   Running
coredns                   Running
```

Wait 1–2 minutes if needed.

---

# ✅ STEP 6.3 — Check Node Status

Now run:

```bash
kubectl get nodes -o wide
```

Your control plane should now show:

```
STATUS: Ready
```

If it still says `NotReady`, wait a bit and recheck.

---

# 🔍 If Calico Pods Stay in CrashLoopBackOff

Most common cause:

👉 Pod CIDR mismatch.

If your kubeadm init used:

```
--pod-network-cidr=10.244.0.0/16
```

Then Calico default `192.168.0.0/16` won't match.

In that case we must edit calico.yaml before applying.

Perfect 👌 — I’ll show you **exactly how to modify the Calico manifest** if your Pod CIDR is different from Calico’s default `192.168.0.0/16`.

This is necessary when:

- You initialized Kubernetes with something like:
    

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16
```

But Calico’s default IP pool is:

```
192.168.0.0/16
```

Those MUST match.

---

# ✅ STEP 1 — Confirm Your Pod CIDR

On the control plane node, check:

```bash
cat kubeadm-config.yaml
```

Look for:

```yaml
podSubnet: 10.244.0.0/16
```

OR if you used CLI:

```bash
kubectl cluster-info dump | grep -i cluster-cidr
```

👉 Remember this value.  
Example we’ll use below:

```
10.244.0.0/16
```

---

# ✅ STEP 2 — Download Calico Manifest (Do NOT Apply Yet)

Instead of applying directly from URL, download it first:

```bash
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml
```

Now you have:

```
calico.yaml
```

---

# ✅ STEP 3 — Edit calico.yaml

Open it:

```bash
nano calico.yaml
```

---

# ✅ STEP 4 — Find Default Calico IP Pool

Search inside nano:

Press:

```
CTRL + W
```

Search for:

```
CALICO_IPV4POOL_CIDR
```

You will find something like:

```yaml
- name: CALICO_IPV4POOL_CIDR
  value: "192.168.0.0/16"
```

---

# ✅ STEP 5 — Change It to Match Your Pod CIDR

If your pod CIDR was:

```
10.244.0.0/16
```

Change the value to:

```yaml
- name: CALICO_IPV4POOL_CIDR
  value: "10.244.0.0/16"
```

⚠ Only change this value. Do not modify anything else.

---

# ✅ STEP 6 — Save and Apply

Save:

```
CTRL + X
Y
Enter
```

Now apply:

```bash
kubectl apply -f calico.yaml
```

---

# ✅ STEP 7 — Verify

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

---

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

- Nodes stay `NotReady`
    
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
    
- Run `kubeadm join`
    
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

If you tell me:

```
kubectl get nodes
kubectl get pods -A
```

I’ll verify whether your Calico installation is correct.


# 🤖 **STEP 7 — Create Worker Node VM**

(Repeat steps 2–4 inside a _second Ubuntu VM_.)

1. New VM in Hyper-V with the same network `vS_k8sCluster`, static IP (e.g., `192.168.5.20`).
    
2. Install Ubuntu and repeat:
    
    - Disable swap
        
    - Install Kubernetes packages
        
    - Install containerd
        
    - Configure network modules  
        This is essentially a cloned version of the control plane setup steps. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iv-31ad24098090 "Setting up a K8s cluster using Ubuntu and Hyper-V P4 | Medium"))
        

---

# 🤝 **STEP 8 — Join Worker Node to Cluster**

(On the _worker node_ VM)

1. On the _control plane node_, Kubernetes will output a join command after `kubeadm init`. It looks like:
    
    ```
    kubeadm join --token <token> <control-plane-host>:6443 \
      --discovery-token-ca-cert-hash sha256:<hash>
    ```
    
    Copy that text. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iv-31ad24098090 "Setting up a K8s cluster using Ubuntu and Hyper-V P4 | Medium"))
    
2. On the _worker VM_:
    
    ```bash
    sudo kubeadm join --token <token> ckak8scp:6443 \
      --discovery-token-ca-cert-hash sha256:<hash>
    ```
    
    Replace `<token>` and `<hash>` with actual values from the control plane output. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iv-31ad24098090 "Setting up a K8s cluster using Ubuntu and Hyper-V P4 | Medium"))
    
3. Verify on control plane:
    
    ```bash
    kubectl get nodes -o wide
    ```
    
    Both nodes should appear. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iv-31ad24098090 "Setting up a K8s cluster using Ubuntu and Hyper-V P4 | Medium"))
    

---

# 🚀 **STEP 9 — Deploy Example App (podinfo)**

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
    
3. Change service type from `ClusterIP` to `NodePort`:
    
    ```bash
    kubectl edit svc podinfo-deploy
    ```
    
    Change `type: ClusterIP` → `type: NodePort`. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iv-31ad24098090 "Setting up a K8s cluster using Ubuntu and Hyper-V P4 | Medium"))
    
4. Expose app externally:
    
    ```bash
    kubectl get svc podinfo-deploy
    ```
    
    Note the NodePort (e.g., above 30000).  
    From your local machine, access:
    
    ```
    http://192.168.5.20:<NodePort>
    ```
    
    If it loads, your cluster is working with a deployed application. ([Medium](https://medium.com/%40enrique.torresds/setting-up-a-kubernetes-cluster-using-ubuntu-virtual-machines-and-hyper-v-part-iv-31ad24098090 "Setting up a K8s cluster using Ubuntu and Hyper-V P4 | Medium"))
    

---

# 🎯 **Summary**

Following these commands and steps **exactly as described in the docs** will give you:

✅ A Kubernetes control plane node running on Ubuntu VM  
✅ A worker node joined to the cluster  
✅ A networking layer (Cilium) making pods communicate  
✅ A sample app (podinfo) exposed via NodePort

If you want, I **can generate actual YAML examples** for the kubeadm config file used in step 5 or paste the recommended Pod CIDRs and network values the docs expect.