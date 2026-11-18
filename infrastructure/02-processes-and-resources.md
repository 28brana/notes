# 02 · Processes & Resources

> [!info] Topic info
> **Subject:** Infrastructure · **Part:** Linux, Containers & Kubernetes · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Infrastructure](README.md) · [Interview questions →](questions/linux-containers-k8s-qa.md)

> [!abstract] In one line
> A server has four resources — **CPU, memory, disk, network** — and debugging "the server is slow" means checking each with tools like **top, free, df, ss**, and understanding **signals**, **load average**, the **OOM killer** and **file descriptors**.

---

## 1. The problem

At 2 a.m. the alert fires: API p99 latency went from 80 ms to 4 s. You SSH in. What do you type first?

You need a quick mental checklist and the meaning of what you see.

---

## 2. Processes and signals

Every running program is a **process** with a **PID** and a parent. `ps aux` lists them; `top` / `htop` show them live sorted by CPU or memory.

A **signal** is a message the kernel delivers to a process:

| Signal | Sent by | Can the app catch it? |
|---|---|---|
| **SIGTERM (15)** | `kill PID`, Docker/K8s stop | ✅ Yes — clean up and exit |
| **SIGINT (2)** | Ctrl+C | ✅ Yes |
| **SIGKILL (9)** | `kill -9`, OOM killer, K8s after grace period | ❌ No — instant death |
| **SIGHUP (1)** | Terminal closed / "reload config" | ✅ Yes (Nginx reloads on it) |

```js
process.on('SIGTERM', async () => {
  server.close();          // stop new connections, finish in-flight
  await db.end();
  process.exit(0);
});
```

> [!tip] 🎤 Say it like this
> "SIGTERM is a polite request to stop — the app can finish in-flight requests and close DB connections. SIGKILL can't be caught; the kernel just kills it. Kubernetes sends SIGTERM, waits 30 seconds, then SIGKILL — so graceful shutdown must finish inside that window."

> [!note] Zombies
> A **zombie** is a dead child whose parent never read its exit status. Harmless alone, but thousands mean a buggy parent. In containers, PID 1 must reap children — that's why `tini` / `--init` exists.

---

## 3. CPU and load average

`uptime` → `load average: 3.2, 2.8, 1.5` = average runnable (+ waiting-on-disk) processes over **1, 5, 15 min**.

Compare with **core count** (`nproc`): load 3.2 on 4 cores = fine; load 12 on 4 cores = things are queuing. Rising 1-min vs 15-min = getting worse.

In `top`, check **%us** (your code), **%sy** (kernel), **%wa** (waiting on disk — high means I/O bottleneck, not CPU).

---

## 4. Memory and the OOM killer

```bash
free -h
#        total  used  free  buff/cache  available
# Mem:    7.7G  5.1G  200M     2.4G       2.3G
```

Look at **available**, not **free** — Linux uses spare RAM as disk cache and gives it back when needed.

When memory truly runs out, the kernel's **OOM killer** picks a big process and SIGKILLs it. Check with `dmesg -T | grep -i oom`. In containers, hitting the memory limit gets you `OOMKilled` (exit code **137** = 128 + 9).

---

## 5. Disk and inodes

```bash
df -h            # free space per filesystem
du -sh /var/log/* | sort -h   # what is eating space
df -i            # inode usage
```

An **inode** is the metadata record for one file. Millions of tiny files (sessions, cache) can use up all inodes — the disk says "No space left" while `df -h` shows 40% free.

> [!warning] Deleted but still open
> Deleting a huge log that a process still has open does **not** free space. Find it with `lsof | grep deleted`, then restart the process or truncate the file (`> app.log`).

---

## 6. File descriptors and ulimit

A **file descriptor (FD)** is a number for anything open — files, **sockets**, pipes. Every TCP connection uses one.

The default limit is often **1024 per process** (`ulimit -n`). A busy Node or Nginx server with 5,000 connections hits `EMFILE: too many open files`. Raise it with `LimitNOFILE=65535` in the systemd unit.

---

## 7. "Server is slow" checklist

```mermaid
flowchart LR
    A[🐢 Slow alert] --> B[📈 uptime load vs cores]
    B --> C[⚙️ top CPU and wa]
    C --> D[🧠 free -h and dmesg OOM]
    D --> E[💾 df -h and df -i]
    E --> F[🌐 ss -s connections]
    F --> G[📜 app logs and journalctl]
```

Also check: what changed (deploy? traffic spike?), and downstream deps — often the DB is slow, not the box.

> [!failure] Interview traps
> - Using `kill -9` first — skips cleanup, can corrupt state. Try SIGTERM first.
> - Panicking about low "free" memory — check "available".
> - Thinking load average is a CPU percentage.
> - Forgetting disk can be "full" via inodes or deleted-but-open files.

---

## Summary

| Resource | Tool | Red flag |
|---|---|---|
| **CPU** | `top`, `uptime` | load ≫ cores, high `%wa` |
| **Memory** | `free -h`, `dmesg` | low available, OOM kills, exit 137 |
| **Disk** | `df -h`, `df -i`, `du` | 100% space or inodes |
| **FDs / network** | `ulimit -n`, `ss -s` | EMFILE, thousands of TIME_WAIT |

> [!success] Key takeaway
> Walk the four resources in order, and always ask "what changed?" — most slowness is one of them running out.
