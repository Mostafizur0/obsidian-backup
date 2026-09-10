---
title: "Production Incident #07 — OOM Killer Playbook"
source: https://www.instagram.com/p/Dc3Yd6_joUJ/
author: "@linux_dev_ops"
type: tutorial
tags:
  - linux
  - devops
  - oom-killer
  - memory
  - troubleshooting
  - systemd
---

# Production Incident #07
---
## Step 1 — Prove What Killed It

Your API disappeared at **02:14 AM.**
No application logs.
No stack trace.
But the process keeps dying and restarting...

> [!warning] App Down Again? Why?
> No logs? What killed my app?

**Before making any changes, ask Linux what happened.**

```bash
$ dmesg -T | grep -Ei 'out of memory|oom|killed process'
Oct 02 02:14:31 ip-10-0-1-23 kernel: Out of memory: Killed process 4092 (myapp)
Oct 02 02:14:31 ip-10-0-1-23 kernel: oom-kill: constraint=CONSTRAINT_NONE,
task_memcg=/, pid=4092, uid=1001

$ journalctl -k --since "30 min ago" | grep -i oom
Oct 02 02:14:31 kernel: [ 1234.567890] oom_reaper: reaped process 4092 (myapp)
```

### What do these commands show?

- **dmesg** → Kernel ring buffer (recent kernel messages)
- **journalctl -k** → Kernel logs via systemd journal (persistent)

> [!tip] Key Takeaway
> Your application logs can be empty because **the kernel killed the process**, not the application itself.

> [!success] Next Step →
> We now know **WHO** killed it. Next, let's check if the server is actually out of memory and how bad the memory pressure was.

---

## Step 2 — Check Memory Pressure

We know the process was killed by the OOM Killer. Now let's check if the server is actually running out of memory.

> [!note] High memory usage? Swap used? Let's check!
> Is the server out of memory or just normal usage?

### 1. Check overall memory usage

```bash
$ free -h
              total   used   free   available
Mem:           16G    15G   300M         420M
Swap:          2G   2.0G     0B
```

- → used close to total and low available = **memory pressure**.
- → Swap fully used = a bad sign.
- → Use **"available"** instead of "free" for real picture.

### 2. Watch memory activity in real-time

```bash
$ vmstat 1 5
procs -----------memory---------- ---swap--
 r  b   swpd   free   buff  cache    si    so
 0  0 2048000 312000 102400 500000    42    87
 1  0 2050000 280000 102400 490000    38    76
 0  0 2049000 310000 102400 480000    45    90
```

- → **si** (swap in) and **so** (swap out) show swapping activity.
- → Consistently high values = memory pressure.

### 3. Check specific memory details

```bash
$ cat /proc/meminfo | grep -E 'MemTotal|MemAvailable'
MemTotal:       16384256 kB
MemAvailable:    428312 kB
SwapTotal:       2097148 kB
SwapFree:             0 kB
```

- → **MemAvailable** gives a better idea of real free memory.
- → If **SwapFree** is 0, the system is under heavy pressure.

### 4. See memory usage by process (top consumers)

```bash
$ ps aux --sort=-%mem | head
USER       PID %CPU %MEM    RSS COMMAND
app       4092 82.1 47.8  7.8G myapp
redis     1832  5.2 14.3  2.3G redis-server
root       721  1.1  3.4  556M dockerd
root       908  0.6  2.1  342M containerd
```

- → Quickly shows which process is consuming the most RAM.
- → We'll investigate the top process in the next step.

> [!note] Important Note
> - Linux tries to use available RAM for cache, which is normal.
> - But if available memory is very low and swap is in use, the system is under **real memory pressure**.
> - Check over time, not just a single snapshot.

> [!danger] Key Indicators of Memory Pressure
> - [ ] Low MemAvailable (e.g. < 500MB)
> - [ ] High swap usage (SwapFree = 0)
> - [ ] High si/so values in vmstat
> - [ ] Important processes using large %MEM
> - [ ] System becoming slow or unresponsive

> [!success] Next Step →
> Memory is under pressure. Now let's find which process is eating all the RAM →

---

## Step 3 — Find Who Is Eating RAM

Now that we know the server is under memory pressure, let's find **which process** is consuming all the memory.

> [!note] Which process is using so much memory?

### 1. List processes sorted by memory

```bash
$ ps aux --sort=-%mem | head
USER       PID %CPU %MEM    RSS COMMAND
app       4092 82.1 47.8  7.8G myapp
redis     1832  5.2 14.3  2.3G redis-server
root       721  1.1  3.4  556M dockerd
root       908  0.6  2.1  342M containerd
```

- → Shows all processes sorted by memory usage.
- → Here, **myapp (PID 4092)** is using **47.8%** of RAM.

### 2. Use top for live view

```bash
$ top
  PID USER      PR  NI  VIRT   RES %MEM COMMAND
 4092 app       20   0  8.1g  7.8g 47.8 myapp
 1832 redis     20   0  2.5g  2.3g 14.3 redis-server
  721 root      20   0  1.2g  556m  3.4 dockerd
```

- → Real-time view of process resource usage.
- → Press **Shift + M** to sort by memory.

### 3. Get detailed memory info of a process

```bash
$ cat /proc/4092/status | grep -E 'VmRSS|VmSize|VmSwap'
VmSize:   10242000 kB
VmRSS:     7982144 kB
VmSwap:     512000 kB
```

- → **VmRSS** = actual physical memory used.
- → **VmSize** = total virtual memory (can be much larger).
- → **VmSwap** = memory moved to swap.

### 4. See memory map (optional)

```bash
$ pmap -x 4092 | tail -5
Address    Kbytes      RSS   Dirty  Mode   Mapping
00007f...   524288   520000  520000 rw--- [heap]
00007f...    10240    10240   10240 rw--- [anon]
00007f...     2048        0       0 r---- [anon]
total      10242000 7982144 7800000
```

- → Shows detailed memory mapping of the process.
- → Useful to identify large allocations (heap, mmap, shared libs).

> [!check] Quick Tips
> - High %MEM with large RSS is a red flag.
> - Check for multiple instances of the same process.
> - Also check child processes (e.g. workers, threads).
> - If using containers, check inside the container as well:
>
> ```bash
> $ docker stats    # for containers
> ```

> [!danger] Common Culprits
> - Memory leaks in the application
> - Unbounded cache or in-memory data
> - Large file uploads / request payloads
> - Too many worker processes
> - Memory-intensive libraries (e.g. ML, image processing)

> [!warning] Don't just kill the process. Understand **why** it's using so much memory.

> [!success] Next Step →
> We found the memory hog. Now let's check if the process has any memory limits (systemd/cgroups).

---

## Step 4 — Check systemd & cgroup Limits

Your process is using a lot of memory, but was it allowed to use this much? Let's check the systemd & cgroup limits.

> [!note] Maybe there is a memory limit set? Let's check!
> Hmm... Is myapp hitting a cgroup limit, or the whole server is out of memory?

### 1. Check service status

```bash
$ systemctl status myapp
● myapp.service - My Application
   Loaded: loaded (/etc/systemd/system/myapp.service)
   Active: failed (Result: OOM-Killed)
   Process: 4092 (code=killed, signal=9)
   ...
```

- Shows the current status of the service and why it stopped.

### 2. Check memory limits for the service

```bash
$ systemctl show myapp -p MemoryCurrent -p MemoryMax -p MemoryPeak
MemoryCurrent=3.8G
MemoryPeak=4.1G
MemoryMax=4G
```

- → **MemoryCurrent**: current memory usage
- → **MemoryPeak**: highest memory usage
- → **MemoryMax**: memory limit set for this service

> [!info]
> If MemoryCurrent reaches MemoryMax, the kernel will kill the process (**cgroup OOM**).

### 3. View cgroup usage (all services)

```bash
$ systemd-cgtop
Control Group                     Tasks %CPU   Memory
/                                   412  12.3   12.4G
system.slice                        210   5.1    8.1G
└─ myapp.service                     15   1.2    3.8G
└─ other.service                      8   0.6    1.1G
```

- Shows real-time resource usage for all services using cgroups.

### 4. Check cgroup memory limit (manual)

```bash
$ cat /sys/fs/cgroup/system.slice/myapp.service/memory.max
4294967296                    # 4G (in bytes)
$ cat /sys/fs/cgroup/system.slice/myapp.service/memory.current
4043309056                    # ~3.8G (in bytes)
```

- Directly check the cgroup limit and current usage. (Useful for containers and systemd services)

> [!check] Important: Two Types of OOM

| 1. Host OOM | 2. Cgroup OOM (service/container) |
|---|---|
| Whole server runs out of memory | Specific service/container hits its memory limit |
| Affects all processes | Only that process is killed |
| Seen in dmesg / kernel logs | Seen in service logs (Result: OOM-Killed) |
| Often caused by overall high usage | Common in systemd, Docker, Kubernetes |

> [!tip] Pro Tip
> Set a reasonable memory limit to protect the server, but also **fix the underlying memory issue** in your application.

> [!success] Next Step →
> Let's monitor memory usage over time to see if this is a memory leak.

---

## Step 5 — Is It a Memory Leak?

Sometimes a process is killed because it slowly keeps consuming more and more memory. Let's confirm if it's a **memory leak**.

> [!note] Let's watch the memory usage over time.
> Is this a memory leak or just a traffic spike?

### 1. Watch memory usage over time

```bash
$ pidstat -r -p 4092 5
Time        minflt/s majflt/s    RSS   %MEM
10:00:01        120        0   1.2G    8.1
10:00:06        135        0   1.8G   11.5
10:00:11        142        0   2.6G   16.2
10:00:16        158        0   3.5G   21.9
```

- → If RSS keeps growing over time, it's a strong signal of a memory leak.
- → Run this for a few minutes and look for a steady increase.

### 2. Quick graph (what to look for)

Memory (RSS) **consistently increasing = Suspicious!**

- ✅ **Normal:** Goes up and down based on traffic
- ❌ **Leak:** Keeps increasing even when traffic is steady

### 3. Get detailed memory info

```bash
$ cat /proc/4092/status | grep -E 'VmRSS|VmSize|VmSwap'
VmSize:   10242000 kB
VmRSS:     7982144 kB
VmSwap:     512000 kB
```

- → **VmRSS** = actual physical memory used
- → **VmSize** = total virtual memory (can be much larger)
- → **VmSwap** = memory moved to swap

### 4. Dig deeper (optional tools)

```bash
$ pmap -x 4092 | tail -5
Address    Kbytes      RSS   Dirty  Mode   Mapping
00007f...   524288   520000  520000 rw--- [heap]
00007f...    10240    10240   10240 rw--- [anon]
00007f...     2048        0       0 r---- [anon]
total      10242000 7982144 7800000
```

- → Shows detailed memory mapping of the process.
- → Helpful to identify large allocations (heap, mmap, shared libs).

### Language Specific Profiling

| Go | Java | C/C++ (Native) |
|---|---|---|
| Use **pprof** | Use **jcmd** or **jmap** | Use **valgrind** |
| `curl http://localhost:6060/debug/pprof/heap` | `jcmd <pid> GC.heap_info` | Use **AddressSanitizer (ASan)** |
| `go tool pprof heap.out` | `jmap -dump:live,format=b <pid> heap.hprof` | Check for memory management issues |

> [!tip] Key Takeaway
> Increasing memory over time is a **signal, not proof**. Use application profiling tools to confirm the root cause.

> [!success] Next Step →
> Now that we know if it's a leak, let's fix it and prevent it from happening again.

---

## Step 6 — Fix the Root Cause & Prevent It

We found the issue, now it's time to fix it and make sure it doesn't happen again. A restart brings the service back, but the real goal is to fix the **underlying problem**.

> [!warning] Solve the problem. Don't just restart it.
> Stable systems are built with observability and limits!

### 1. Fix the application

- [ ] Check for memory leaks (heap growth over time)
- [ ] Remove unbounded caches or large in-memory data
- [ ] Optimize large payloads / file processing
- [ ] Reduce unnecessary concurrency / goroutines
- [ ] Fix third-party library leaks
- [ ] Load test with production-like traffic

> [!tip]
> Use profiling tools (pprof, jmap, valgrind, etc.) to find the exact cause.

### 2. Set resource limits (systemd)

```bash
# Create an override file
$ sudo systemctl edit myapp

[Service]
MemoryHigh=3G
MemoryMax=4G
Restart=on-failure
RestartSec=10
```

- **MemoryHigh** → soft limit (throttling starts)
- **MemoryMax** → hard limit (process killed if exceeded)
- **Restart** → auto restart for recovery

### 3. Add monitoring & alerting

- [ ] Monitor memory usage (process & system)
- [ ] Alert on high memory usage / low available memory
- [ ] Alert on OOM kills (from dmesg / logs)
- [ ] Track swap usage
- [ ] Visualize trends in Grafana (RSS, MemoryAvailable, etc.)

**Useful metrics to monitor:**

- `process_resident_memory_bytes`
- `node_memory_MemAvailable_bytes`
- `node_memory_SwapFree_bytes`
- `rate(node_oom_kill_total[5m])`

> [!warning] Alert before it breaks!

### 4. Validate the fix

```bash
$ systemctl restart myapp
$ systemctl status myapp | grep Active
   Active: active (running)
```

- [ ] Monitor for a few hours / days.
- [ ] Verify memory usage is stable.
- [ ] Ensure no more OOM kills in dmesg.
- [ ] Check application logs for improvements.

> [!success] Stable service = Happy SRE 🙂

> [!check] Final Checklist
> - [ ] Root cause identified and fixed
> - [ ] Resource limits configured
> - [ ] Monitoring and alerting in place
> - [ ] Load tested (if possible)
> - [ ] Runbook updated for future incidents

> [!important] Restart is a **recovery step, not a solution**. Fix the cause, add guardrails, monitor, and sleep better! 🙂

### Production Incident Flow Recap

1. Prove what killed it (dmesg, journalctl)
2. Check memory pressure (free, vmstat)
3. Find the memory hog (ps, top, /proc)
4. Check systemd & cgroup limits.
5. Analyze for memory leak (pidstat, profiling)
6. Fix, set limits, monitor, prevent

---

> [!quote] Save for later · Share with a friend · Comment "OOM" if you found this useful
> Follow **@linux_dev_ops** for more Linux & DevOps content — Keep Learning and Building!
