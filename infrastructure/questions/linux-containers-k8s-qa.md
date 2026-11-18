# Q&A · Infrastructure — Linux, Containers & Kubernetes

[← Back to Infrastructure](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · Linux Essentials

[Read the notes →](../01-linux-essentials.md)

> [!question]- 🟢 Q1. What does `chmod 755` mean, and when would you use 644 or 600?
> Permissions are r=4, w=2, x=1 for owner, group and others. 755 means the owner can read/write/execute and everyone else can read/execute — typical for scripts and directories. 644 is for normal files and configs, and 600 (owner only) is for private keys and `.env` files.

> [!question]- 🟢 Q2. Where would you look for config files and logs on a Linux server?
> Config lives under `/etc` (for example `/etc/nginx/`), logs under `/var/log` or in journald via `journalctl`. Variable data like databases sits in `/var/lib`, and `/proc` exposes live info about running processes.

> [!question]- 🟡 Q3. How do you keep a Node app running after a crash or reboot on a plain VM?
> Create a systemd unit with `ExecStart=/usr/bin/node server.js`, `User=deploy` and `Restart=always`, then `systemctl enable --now myapp`. systemd starts it at boot, restarts it on crash, and `journalctl -u myapp` shows its logs. `nohup` or a tmux session is not production-grade.

> [!question]- 🔴 Q4. Your app fails with `EACCES: permission denied` writing to `/var/log/myapp`. How do you fix it properly?
> Check who the app runs as (`ps -o user`) and who owns the folder (`ls -ld`). Then fix ownership, e.g. `chown -R deploy:deploy /var/log/myapp` with 755 on the dir — don't `chmod 777` and don't run the app as root. Better still, log to stdout and let journald or the container platform collect it.

---

## 02 · Processes & Resources

[Read the notes →](../02-processes-and-resources.md)

> [!question]- 🟢 Q5. What is the difference between SIGTERM and SIGKILL?
> SIGTERM asks the process to stop; the app can catch it, finish in-flight requests, close DB connections and exit. SIGKILL can't be caught — the kernel kills the process instantly. Docker and Kubernetes send SIGTERM first and SIGKILL after a grace period (10 s / 30 s).

> [!question]- 🟡 Q6. Load average is 6.0 — is that bad?
> It depends on the core count. Load is the average number of processes running or waiting (including on disk I/O); on 8 cores 6.0 is fine, on 2 cores it means heavy queuing. Compare the 1, 5 and 15 minute values to see whether it's rising, and check `%wa` in top to tell CPU pressure from I/O wait.

> [!question]- 🟡 Q7. A container keeps restarting with exit code 137. What happened?
> 137 = 128 + 9, meaning it was killed by SIGKILL — almost always the OOM killer because it went over its memory limit. Confirm with `kubectl describe pod` (OOMKilled) or `dmesg`. Fix the memory leak or raise the limit, and for Node set `--max-old-space-size` below the container limit.

> [!question]- 🔴 Q8. Users say the server is slow. Walk me through what you check.
> First, what changed — a deploy or a traffic spike? Then go resource by resource: `uptime` load vs `nproc`, `top` for CPU hogs and high I/O wait, `free -h` and `dmesg` for memory and OOM kills, `df -h` and `df -i` for disk and inodes, `ss -s` for connection counts and FD limits. Finally app logs and downstream dependencies — often the database is the slow part, not the box.

> [!question]- 🔴 Q9. `df -h` shows 40% free, but writes fail with "No space left on device". Why?
> Either the filesystem ran out of **inodes** (millions of tiny files — check `df -i`), or you're looking at a different mount than the one being written to. A related case: space not freed after deleting a big log because a process still holds it open — `lsof | grep deleted` finds it.

---

## 03 · Bash Scripting

[Read the notes →](../03-bash-scripting.md)

> [!question]- 🟢 Q10. What does `set -euo pipefail` do and why use it?
> `-e` stops the script on the first failing command, `-u` errors on unset variables, and `pipefail` makes a pipeline fail if any command in it fails. Together they stop a deploy script from silently carrying on after something broke.

> [!question]- 🟡 Q11. A script works when you run it by hand but fails from cron. Why?
> Cron runs with a minimal environment — a short PATH, no `.bashrc`, a different working directory. Use absolute paths to binaries and files, set needed env vars in the script, and redirect output to a log (`>> /var/log/job.log 2>&1`) so failures are visible.

---

## 04 · SSH & Server Hardening

[Read the notes →](../04-ssh-and-server-hardening.md)

> [!question]- 🟢 Q12. How does SSH key authentication work?
> You keep the private key on your machine and put the public key in the server's `~/.ssh/authorized_keys`. The server sends a challenge that only the private key can sign, and verifies it with the public key — the private key never travels over the network.

> [!question]- 🟡 Q13. You just launched a VM with a public IP. What do you harden first?
> Disable password and root login over SSH, allow SSH only from a VPN or bastion, and set a firewall or security group that exposes just 80 and 443. Turn on automatic security updates, run the app as a non-root user, and keep database ports private.

---

## 05 · Nginx

[Read the notes →](../05-nginx.md)

> [!question]- 🟢 Q14. Why put Nginx in front of a Node.js app?
> As a reverse proxy it gives one public entry point and handles TLS termination, gzip, static files, buffering slow clients and load balancing across several Node processes. The app only has to deal with business logic over plain HTTP.

> [!question]- 🟡 Q15. What is TLS termination and what's the trade-off?
> Nginx (or a load balancer) decrypts HTTPS and forwards plain HTTP to the app on a private network. It centralizes certificates and saves CPU on app servers, but traffic behind the proxy is unencrypted — fine inside a trusted VPC, otherwise re-encrypt or use mTLS.

> [!question]- 🟡 Q16. Why does every request in your app logs show the same IP after adding Nginx?
> The app sees Nginx as the client. Nginx must pass the real IP with `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for` (and `X-Real-IP`), and the app must trust the proxy — in Express, `app.set('trust proxy', 1)`.

> [!question]- 🔴 Q17. Users get 502 and 504 errors from Nginx. How do you tell them apart and debug?
> 502 means Nginx couldn't get a valid response — the app is down, crashed, or on the wrong port; check `systemctl status`, the upstream address and the Nginx error log. 504 means the app took longer than `proxy_read_timeout` (60 s default) — look for slow queries or blocked event loops rather than just raising the timeout.

---

## 06 · Docker Basics

[Read the notes →](../06-docker-basics.md)

> [!question]- 🟢 Q18. What is the difference between an image and a container?
> An image is an immutable, layered template with the app and its dependencies. A container is a running instance of an image with a thin writable layer on top. One image can run as many containers.

> [!question]- 🟡 Q19. How is a container different from a VM?
> A VM virtualizes hardware and runs its own full OS kernel, so it's heavy and slow to boot. A container is just a host process isolated with namespaces (what it can see — PIDs, network, filesystem) and limited with cgroups (CPU, memory). It shares the host kernel, so it starts in about a second but has weaker isolation.

> [!question]- 🟡 Q20. Your Postgres container lost all its data after a restart. Why?
> The data was in the container's writable layer, which is deleted when the container is removed or recreated. Mount a named volume or host path at `/var/lib/postgresql/data` so data lives outside the container's lifecycle.

> [!question]- 🔴 Q21. `docker stop` always takes 10 seconds and the app never runs its shutdown code. Why?
> The app isn't receiving SIGTERM — usually because CMD is in shell form or started via `npm start`, so `sh` or npm is PID 1 and doesn't forward the signal. Docker waits 10 s then SIGKILLs. Use exec-form `CMD ["node", "server.js"]`, add `--init`/tini if you spawn children, and handle SIGTERM in code.

---

## 07 · Dockerfile Best Practices

[Read the notes →](../07-dockerfile-best-practices.md)

> [!question]- 🟢 Q22. What is a multi-stage build and why use it?
> You build in one stage with compilers and dev dependencies, then copy only the built output and production deps into a clean slim final stage. The final image is much smaller and has fewer packages to attack — often 1 GB down to 150–200 MB.

> [!question]- 🟡 Q23. Every small code change makes `npm install` run again in the build. How do you fix it?
> Docker reuses a layer only if it and every layer before it is unchanged. Copy `package*.json` first, run `npm ci`, and only then `COPY . .` — now the dependency layer stays cached until package.json changes. Add a `.dockerignore` so `node_modules` and `.git` don't bust the cache either.

> [!question]- 🟡 Q24. Why run containers as non-root, and how?
> If an attacker breaks out of the app or the container, root inside can more easily become root on the host. Add `USER node` (or create a user) in the Dockerfile and `chown` the files the app needs; Kubernetes can enforce it with `runAsNonRoot`.

> [!question]- 🔴 Q25. A security scan says your image has 300 CVEs and is 1.4 GB. What do you change?
> Switch to a slim or distroless base, use a multi-stage build so build tools and dev deps don't ship, and pin and regularly rebuild the base to pick up patches. Remove package-manager caches in the same RUN layer and make sure no secrets or `.git` are copied in. Then add image scanning to CI so it doesn't regress.

---

## 08 · Docker Compose

[Read the notes →](../08-docker-compose.md)

> [!question]- 🟢 Q26. What is Docker Compose used for?
> It defines a multi-container app — API, database, Redis — in one YAML file and runs it with `docker compose up`. Services share a network and reach each other by service name. It's mainly for local development and CI.

> [!question]- 🟡 Q27. The API starts before Postgres is ready and crashes. How do you fix it in Compose?
> Plain `depends_on` only waits for the container to start. Add a healthcheck on the DB (`pg_isready`) and use `depends_on: db: condition: service_healthy`. The app should also retry connections on startup, because in production dependencies can restart any time.

> [!question]- 🟡 Q28. Would you run production on Docker Compose?
> For a small single-server app, it can be fine. But Compose is one host with no rescheduling, autoscaling or proper rolling updates, so for multi-node production I'd use Kubernetes or a managed service like ECS or Cloud Run.

---

## 09 · Kubernetes Architecture

[Read the notes →](../09-kubernetes-architecture.md)

> [!question]- 🟢 Q29. What are the main components of a Kubernetes cluster?
> The control plane has the api-server (front door for all operations), etcd (stores cluster state), the scheduler (picks nodes for pods) and the controller-manager (runs reconcile loops). Each worker node runs the kubelet (starts and monitors pods), kube-proxy (Service routing) and a container runtime like containerd.

> [!question]- 🟡 Q30. What does "declarative" and "reconciliation" mean in Kubernetes?
> You declare desired state — "3 replicas of api:1.4" — instead of issuing commands. Controllers continuously compare desired and actual state and act to close the gap, so if a pod or node dies, a replacement is created automatically.

> [!question]- 🟡 Q31. Walk me through what happens when you run `kubectl apply` on a Deployment.
> The api-server validates it and stores it in etcd. The Deployment controller creates a ReplicaSet, which creates Pod objects. The scheduler assigns each pod to a node, the kubelet on that node pulls the image and starts the container, and kube-proxy updates Service routing once pods are ready.

> [!question]- 🔴 Q32. A pod is stuck in Pending. How do you debug it?
> Pending means it hasn't been scheduled. `kubectl describe pod` shows the scheduler's events — usually not enough CPU or memory for the requests, a node selector or taint that no node matches, or an unbound PersistentVolumeClaim. Fix the requests, add nodes (or let the cluster autoscaler), or fix the volume.

---

## 10 · Kubernetes Workloads

[Read the notes →](../10-kubernetes-workloads.md)

> [!question]- 🟢 Q33. What is a pod, and why not just call a pod's IP directly?
> A pod is the smallest unit in Kubernetes — one or a few containers sharing an IP and volumes. Pods are disposable and get a new IP when recreated, so you call them through a Service, which gives a stable DNS name and load-balances across matching pods.

> [!question]- 🟡 Q34. Deployment vs StatefulSet vs DaemonSet — when do you use each?
> Deployment for stateless apps like APIs, where any replica is interchangeable. StatefulSet for databases and brokers that need stable names (`db-0`) and their own persistent disk. DaemonSet when you need exactly one pod per node, like a log shipper or node exporter.

> [!question]- 🟡 Q35. Explain the Service types ClusterIP, NodePort and LoadBalancer, and where Ingress fits.
> ClusterIP is internal-only, for service-to-service calls. NodePort opens a port on every node; LoadBalancer provisions a cloud load balancer for one service. Ingress is an HTTP layer on top — one load balancer routing many hosts and paths to different ClusterIP Services, with TLS, via an Ingress controller.

> [!question]- 🔴 Q36. During every rolling deploy, users see a burst of 502 errors. What's wrong?
> New pods get traffic before they're ready, or old pods are killed while still serving requests. Add a readiness probe so pods join only when ready, handle SIGTERM gracefully (stop accepting, finish in-flight), and add a short `preStop` sleep so the load balancer removes the pod before it shuts down. Check `maxUnavailable` isn't dropping too much capacity.

---

## 11 · Kubernetes Config & Scaling

[Read the notes →](../11-kubernetes-config-and-scaling.md)

> [!question]- 🟢 Q37. ConfigMap vs Secret?
> Both inject config into pods as env vars or files so the image stays the same across environments. ConfigMaps are for non-sensitive values; Secrets are for passwords and keys. Secrets are only base64-encoded by default, so enable encryption at rest and lock them down with RBAC or sync them from a vault.

> [!question]- 🟡 Q38. What's the difference between liveness, readiness and startup probes?
> Readiness decides whether the pod receives traffic — failing just removes it from the Service. Liveness decides whether the container is stuck — failing restarts it. Startup gives slow apps time to boot by delaying liveness checks until it passes.

> [!question]- 🟡 Q39. What are resource requests and limits, and what happens when you exceed them?
> Requests are what the scheduler reserves for the pod and what HPA percentages are measured against. Limits are a hard cap: going over the CPU limit gets you throttled, going over the memory limit gets the container OOMKilled and restarted.

> [!question]- 🔴 Q40. The database had a 20-second blip and every API pod in the cluster restarted. Why, and how do you prevent it?
> The liveness probe was checking the database, so all pods failed liveness at once and were restarted — making a short outage much worse. Liveness should only check that the process itself responds; put dependency checks in readiness so pods are just taken out of rotation until the DB returns.

> [!question]- 🔴 Q41. HPA is configured at 70% CPU but never scales up, even under heavy load. What do you check?
> Check the pods have CPU **requests** — HPA computes utilization against them. Make sure metrics-server is running (`kubectl top pods` works) and look at `kubectl describe hpa` for errors. If pods scale but stay Pending, the cluster needs more nodes via the cluster autoscaler; and if the app is I/O-bound, scale on req/s or queue length instead of CPU.

---

## 12 · Helm

[Read the notes →](../12-helm.md)

> [!question]- 🟢 Q42. What are a Helm chart, values and a release?
> A chart is a package of templated Kubernetes manifests with default values. Values files override settings per environment, like replicas or image tag. A release is one installed instance of a chart with a revision history, so `helm rollback` can return to an earlier revision.
