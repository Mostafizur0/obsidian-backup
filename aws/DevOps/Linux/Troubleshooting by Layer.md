Tools That Map to the Architecture
- Processes/Scheduling: top, htop, ps, chrt, taskset
- Memory: free, vmstat, smem, /proc/meminfo, perf mem
- Filesystems/I/O: iostat, iotop, df, du, lsof, mount
- Networking: ss, ip, ethtool, tcpdump, nft, tc
- Kernel logs and modules: dmesg, journalctl -k, lsmod, modprobe
- Security: getenforce/setenforce (SELinux), aa-status (AppArmor), capsh

Diagnose issues by moving from user space to kernel space: check application logs, then service state, then kernel messages and resource limits.
