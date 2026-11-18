# 06 · Docker Basics

> [!info] Topic info
> **Subject:** Infrastructure · **Part:** Linux, Containers & Kubernetes · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Infrastructure](README.md) · [Interview questions →](questions/linux-containers-k8s-qa.md)

> [!abstract] In one line
> **Docker** packages an app with everything it needs into an **image**; running an image gives a **container** — an isolated process that shares the host kernel, built on **namespaces** (what it can see) and **cgroups** (what it can use).

---

## 1. The problem

"Works on my machine." Your laptop has Node 20 and libvips 8.15; the server has Node 18 and no libvips. Deploys break on small differences.

Docker ships the app **and its environment** as one artifact, so dev, CI and prod run the exact same thing.

---

## 2. Image vs container

| | Image | Container |
|---|---|---|
| **What** | Read-only template (app + deps + OS libs) | A running instance of an image |
| **Analogy** | A class | An object |
| **State** | Immutable, versioned by tag/digest | Has a thin writable layer, lost on delete |
| **Count** | One `myapi:1.4.2` | Run 10 containers from it |

An image is a stack of **layers** — each Dockerfile instruction adds one. Layers are cached and shared: 20 images on `node:20-slim` store the base only once.

```mermaid
flowchart LR
    D[📄 Dockerfile] -- docker build --> I[📦 Image]
    I -- docker push --> R[🗄️ Registry]
    R -- docker pull --> H[🖥️ Server]
    H -- docker run --> C[⚙️ Containers]
```

---

## 3. Container vs VM — namespaces and cgroups

A **VM** runs a full guest OS on a hypervisor — GBs, boots in ~a minute. A **container** is just a normal Linux process with walls around it — MBs, starts in ~1 second.

The walls come from two kernel features:

- **Namespaces** = what it can **see**. Own PID list (app is PID 1), own network/IP, own filesystem view, own hostname, own users.
- **cgroups** (control groups) = what it can **use**. Caps on CPU, memory, I/O — e.g. "max 512 MB RAM".

> [!tip] 🎤 Say it like this
> "An image is the immutable build artifact; a container is a running instance of it. Unlike a VM, a container doesn't have its own kernel — it's a host process isolated by namespaces, which limit what it can see, and cgroups, which limit how much CPU and memory it can use. That's why containers are light and start in a second."

> [!warning] Weaker isolation than VMs
> All containers share the host kernel — a kernel exploit can escape. That's why you run as non-root and why cloud providers wrap containers in micro-VMs (e.g. Firecracker) for multi-tenant isolation.

---

## 4. Everyday commands

```bash
docker build -t myapi:1.0 .
docker run -d --name api -p 8080:3000 -e NODE_ENV=production myapi:1.0
docker ps                       # running containers
docker logs -f api              # stdout/stderr
docker exec -it api sh          # shell inside
docker stop api                 # SIGTERM, then SIGKILL after 10s
```

- `-p 8080:3000` = host port 8080 → container port 3000.
- **Volumes** (`-v pgdata:/var/lib/postgresql/data`) keep data after the container is deleted.
- **Networks**: containers on the same user-defined network reach each other by **name** (`http://db:5432`).

> [!failure] Interview traps
> - Saying containers are "lightweight VMs" — no own kernel.
> - Storing data inside the container's writable layer — it's gone on `docker rm`.
> - Using `:latest` in prod — you can't tell what's running or roll back cleanly.
> - Logging to files inside the container — log to **stdout**, let the platform collect it.

---

## Summary

- **Image** = immutable layered template; **container** = running instance.
- **Namespaces** isolate the view, **cgroups** limit resources, kernel is shared.
- Registry → pull → run; volumes for data, networks for service names.

> [!success] Key takeaway
> Docker makes the environment part of the artifact — build once, run the same thing everywhere.
