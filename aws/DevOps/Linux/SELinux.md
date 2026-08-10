[[Linux]]
[[DevOps/Kubernetes/Security/Security]]
[[DevSecOps/Security|Security]]

https://medium.com/@damilolavicdavids/100-days-of-devops-day-5-installing-disabling-selinux-for-maintenance-057020564e27
**SELinux (Security-Enhanced Linux)** is a security module built into the Linux kernel that enforces **Mandatory Access Control (MAC)**.  
Instead of relying only on standard Linux permissions (owner, group, others), SELinux uses **policies** to decide which processes can access which files, sockets, and other system resources even if standard permissions would normally allow it.

SELinux ships enabled or strongly encouraged on most mainstream distributions for a reason. In the context of Linux security, <span style="color:green;">it sits below your applications and above the kernel in a way that’s hard to replicate with permissions alone.</span> You don’t interact with it often when things are healthy. When something goes wrong, though, it tends to be very loud and very specific.

It adds mandatory access control on top of traditional permissions, which means <span style="color:green;">the system checks two things every time a process touches something.</span> First, do the Unix permissions allow this? Second, does SELinux policy allow this specific process, in this specific context, to do this specific action? <span style="color:green;">Both have to agree.</span>

You see this most clearly with root. Under classic Linux permissions, root is effectively omnipotent. <span style="color:green;">Under SELinux, root is still constrained by policy.</span> If a domain is not allowed to perform an action, running it as root does not magically fix that.

Without SELinux, a service compromise often explodes outward until something else catches it. With SELinux enforcing, you see repeated denials clustered around one process, one context, one resource. The damage is smaller, the timeline is clearer, and containment is simpler.
# SELinux modes
### Enforcing: 
If an action violates policy, it does not happen. The process gets denied, an AVC is logged, and the system moves on. This applies just as much to admin mistakes as it does to attacker behavior. You feel enforcing mode most when something is misconfigured, half-installed, or when you are making assumptions that the policy does not allow.
### Permissive
It still <span style="color:green;">evaluates policy and still logs denials, but it does not block the action. Nothing breaks. Everything works.</span> That makes it useful for learning how a service behaves under SELinux or for debugging a rollout. It also makes it dangerous to leave in place, because you slowly stop <span style="color:green;">noticing the difference between “allowed” and “would have been blocked.”</span>
### Disabled
Disabled mode removes SELinux from the equation entirely. There are no checks, no labels being enforced, and no denials to review.

# Reading SELinux Denials
An AVC denial is specific by design. It tells you which process was blocked, what it tried to do, which object was involved, and which policy rule said no.

<span style="color:green;">Some denials are genuinely safe to ignore. Volume on its own is a poor signal. Time correlation is better.</span> A spike in new denials right after a deployment almost always points to a configuration mismatch. A sudden cluster during an incident window is more interesting, especially if the process and access type don’t line up with normal behavior.

Treat SELinux as part of your Linux security controls, not a standalone feature. Tune alerts instead of silencing them. Keep denials accessible during investigations. When auditors or incident reviewers ask how access was restricted, SELinux gives you concrete answers instead of hand-waving.

High-exposure servers benefit the most. Internet-facing services, shared environments, anything processing untrusted input all day long. In those cases, SELinux enforcing mode changes how incidents unfold in a measurable way. Compromises are louder. Movement is constrained. Recovery is more predictable.

Containers complicate the picture, but they don’t remove SELinux from it. Used together, they can reinforce each other. Used carelessly, they can double the confusion. The same principle applies either way. Simple, repeatable patterns beat clever configurations that nobody wants to touch six months later.

https://linuxsecurity.com/features/what-is-selinux
https://github.blog/developer-skills/programming-languages-and-frameworks/introduction-to-selinux/
![[Pasted image 20260810173407.png]]
## SELinux architecture—the basics

The SELinux architecture can be split into four main components.

1. Firstly, a **Subject** must request access to take an action. In most cases, the subject is a process that is requesting access to a resource. Access can be controlled via Access Vector Rules, whose details will be presented shortly.
2. Second, an **Object Manager (OM)** that controls the access of the subject. It will query the Security Server in order to allow or deny actions.
3. **Security Server**—the security server makes decisions based on the Security Policy and returns an answer.
4. **Access Vector Cache (AVC)**—this is a cache that stores the decisions of the security server in order to speed up performance.
