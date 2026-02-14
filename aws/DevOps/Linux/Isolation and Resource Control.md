## Users, Namespaces, and Cgroups
Linux enforces permissions through users, groups, and ACLs. Capabilities allow granting targeted privileges (e.g., CAP_NET_ADMIN) instead of full root. Namespaces isolate resources for containers, and cgroups v2 apply CPU, memory, and I/O limits.

- **Namespaces**: isolate PID trees, networking (veth, bridges), mounts, users, hostnames.
- **Cgroups**: throttle or guarantee resources; enable quotas and accounting.
- SELinux/AppArmor: restrict process capabilities and [file access](https://www.youstable.com/blog/access-file-manager-in-cpanel/) paths.
