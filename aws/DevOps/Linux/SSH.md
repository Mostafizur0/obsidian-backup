[[Linux]]

## How ssh works
https://youtu.be/CfpVMMV3kL0?si=PA2S14_GBwEG3LqY
ssh comes to solve the issue with telnet. Telnet sends every thing including credentials in plain text which is prime target of password swiping attack. The goal was
1. A connection that was secure in every way that mattered. No one could read it. 
2. No one could tamper with it without being noticed. 
3. You could be sure the server was really the server and 
4. The server could be sure that you were really you. 
The remarkable part is how SSH pulls off all four without ever trusting the network it runs on.
SSh achieved this by:
- Shared session secret key through Deffi-helman key exchange ![[Pasted image 20260812155129.png]]
- Every server has its own identity, a long-term key called a host key. During the handshake, the server proves it owns that key by signing the conversation with it. and your computer checks the signature. The catch is that the very first time you connect, your computer has never seen this server's key before, so it has no way to know whether it's genuine. That prompt is your computer admitting I can't vouch for this one yet. When you say yes, it saves the server's key in a file called known host. And from then on, every time you reconnect, it silently checks that the server's key still matches. If it ever changes, SSH throws a loud warning because that could mean someone is impersonating the machine you trust. Now the tunnel is built and you trust the server. ![[Pasted image 20260812155653.png]] ![[Pasted image 20260812155809.png]]
- It's finally your turn to prove who you are. There are two ways to do it. 
	- The first is a password. Your password really is sent to the server, but it's sent inside the encrypted tunnel you just built. So no one on the network can read it. The catch is that the server still receives your actual password. 
	- The second way, the way you should be using, never sends a secret at all. It's called ==public key authentication.== It starts with a pair of keys, a private key that lives on your laptop and never ever leaves it. And a public key which you copy onto the server ahead of time. When you log in, the server sends your computer a challenge tied to this exact session. Your computer signs that challenge with your private key, producing a signature, that proves you hold the key without revealing it. The server checks that signature against the public key it already has. If it matches, you're in. The signature only works for this one session. So even if someone captured it, they could never reuse it. Your private key never touched the network. That is the whole reason Keybase login is safer than a password. There's nothing reusable to steal, nothing to fish, and the key itself is so large that guessing it is hopeless. No computer on Earth could try every possibility before the sun burned out. And once that one idea clicks, the rest of SSH falls into place.
     ![[Pasted image 20260812160830.png|263]]

# ssh-copy-id
The **`ssh-copy-id`** command is ==a built-in OpenSSH utility used to **install a local SSH public key onto a remote server's `authorized_keys` file**==. This process enables passwordless authentication, securing your future SSH connections and automating logins.
Quick Reference Commands

| Task                    | Command                                                 |
| ----------------------- | ------------------------------------------------------- |
| **Copy Default Key**    | `ssh-copy-id user@remote_host`                          |
| **Copy a Specific Key** | `ssh-copy-id -i ~/.ssh/id_ed25519.pub user@remote_host` |
| **Use Custom Port**     | `ssh-copy-id -p 2222 user@remote_host`                  |
| **Dry Run (Preview)**   | `ssh-copy-id -n user@remote_host`                       |
`ssh-copy-id` uses the SSH protocol to connect to the target host and upload the SSH user key. The command ==edits the `authorized_keys` file on the server. It creates the `.ssh` directory if it doesn't exist. It creates the authorized keys file if it doesn't exist. Effectively, ssh key copied to server.==

It also checks if the key already exists on the server. Unless the `-f` option is given, each key is only added to the authorized keys file once.

It further ensures that ==the key files have appropriate permissions.== Generally, the user's home directory or any file or directory containing keys files should not be writable by anyone else. Otherwise someone else could add new authorized keys for the user and gain access. Private key files should not be readable by anyone else.

https://www.ssh.com/academy/ssh/copy-id

---
## 🔑 Setup Password-less SSH Access (Jump Host to App Servers)

- Date: 2026-08-12
- Author: DevOps Team
- Source: KodeKloud
- Tags: #ssh #security #devops #linux 

---

## 🎯 Objective

Set up secure, password-less SSH authentication from the `thor` user on the Jump Host to all target application servers through their respective sudo users.

```unset
[ Jump Host (thor) ] ───( SSH Key )───► [ stapp01 (tony) ]
                                      ├──► [ stapp02 (steve) ]
                                      └──► [ stapp03 (banner) ]
```

---

## 🛠️ Step-by-Step Implementation

## Step 1: Switch to the Thor User

Log into your Jump Host and make sure you are operating as the `thor` user.

```bash
sudo su - thor
```

## Step 2: Generate the SSH Key Pair

Create a new digital lock and key (RSA key pair).

> [!IMPORTANT]  
> When prompted to "Enter passphrase", leave it empty and press Enter. This allows the login to happen automatically without a password.

```bash
ssh-keygen -t rsa
```

## Step 3: Copy the Public Key to App Servers

Send your public key to each application server. You will need to type each user's password one last time during this step.

- App Server 1 (`stapp01` via `tony`):
    
    ```bash
    ssh-copy-id tony@stapp01
    ```
    
- App Server 2 (`stapp02` via `steve`):
    
    ```bash
    ssh-copy-id steve@stapp02
    ```
    
- App Server 3 (`stapp03` via `banner`):
    
    ```bash
    ssh-copy-id banner@stapp03
    ```
    

## Step 4: Verify Your Connections

Test each connection to ensure you are logged in automatically without being asked for a password.

```bash
ssh tony@stapp01 "hostname"
ssh steve@stapp02 "hostname"
ssh banner@stapp03 "hostname"
```

_If each command returns the server name without blocking for a password, your setup is complete!_ 🎉

---

## 💡 Troubleshooting Tips

> [!WARNING]  
> If you are still asked for a password, check the following rules on the remote servers:
> 
> 1. The remote user's home folder must be private (`chmod 700 ~`).
> 2. The `.ssh` folder must be private (`chmod 700 ~/.ssh`).
> 3. The `authorized_keys` file must be restricted (`chmod 600 ~/.ssh/authorized_keys`).

