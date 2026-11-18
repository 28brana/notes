# 11 · Kubernetes Config & Scaling

> [!info] Topic info
> **Subject:** Infrastructure · **Part:** Linux, Containers & Kubernetes · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Infrastructure](README.md) · [Interview questions →](questions/linux-containers-k8s-qa.md)

> [!abstract] In one line
> Production pods need **config** (ConfigMaps, Secrets), **health checks** (liveness, readiness, startup probes), **resource requests/limits**, and **autoscaling** (HPA) — together these make the cluster self-healing and right-sized.

---

## 1. The problem

Your Deployment runs, but: the DB URL is baked into the image, a pod stuck in a deadlock keeps receiving traffic, one pod eats all the node's memory, and Black Friday traffic is 5× normal.

Each section fixes one of these.

---

## 2. ConfigMaps and Secrets

Keep config **out of the image** so the same image runs in staging and prod.

- **ConfigMap** — non-sensitive config: `LOG_LEVEL`, feature flags, config files.
- **Secret** — passwords, API keys, certs. Injected as env vars or mounted files.

```yaml
envFrom:
  - configMapRef: { name: api-config }
  - secretRef:    { name: api-secrets }
```

> [!warning] Secrets are not encrypted by default
> They're only **base64-encoded** in etcd. Turn on encryption at rest, restrict access with RBAC, and ideally sync from a real vault (AWS Secrets Manager, Vault) — see [Secrets Management](../system-design/security/03-secrets-management.md).

---

## 3. Probes — is it alive, is it ready?

| Probe | Question | On failure |
|---|---|---|
| **Liveness** | Is the process stuck? | **Restart** the container |
| **Readiness** | Can it take traffic right now? | **Remove from Service** endpoints (no restart) |
| **Startup** | Has it finished booting? | Holds off liveness until it passes — for slow-starting apps |

```yaml
readinessProbe: { httpGet: { path: /ready, port: 3000 }, periodSeconds: 5 }
livenessProbe:  { httpGet: { path: /live,  port: 3000 }, periodSeconds: 10, failureThreshold: 3 }
```

> [!tip] 🎤 Say it like this
> "Readiness decides whether a pod gets traffic — it fails during startup, warm-up or when a critical dependency is down, and the pod is just taken out of rotation. Liveness decides whether to restart it, so it should only check that the process itself is responsive. A startup probe gives slow apps time to boot before liveness kicks in."

> [!failure] Liveness that checks the database
> If the DB blips, **every** pod fails liveness and restarts at once — turning a small outage into a big one. Check dependencies in **readiness**, not liveness.

---

## 4. Requests and limits

```yaml
resources:
  requests: { cpu: 250m, memory: 256Mi }   # guaranteed, used for scheduling
  limits:   { cpu: "1",  memory: 512Mi }   # hard ceiling
```

- **Requests** — what the scheduler reserves. `250m` = a quarter of a core.
- **Limits** — the cgroup cap. Over the **CPU** limit → **throttled** (slow). Over the **memory** limit → **OOMKilled** (exit 137) and restarted.
- No requests → scheduler packs pods blindly; noisy neighbours starve each other.

---

## 5. Autoscaling

```mermaid
flowchart LR
    M[📊 metrics-server] --> H[⚙️ HPA]
    H -- target 70 percent CPU --> D[📦 Deployment 3 to 10 pods]
    D -- no room on nodes --> CA[🖥️ Cluster Autoscaler adds node]
```

- **HPA** (Horizontal Pod Autoscaler) — adds/removes pods based on CPU %, memory, or custom metrics like req/s or queue length. CPU % is measured **against requests**, so requests must be set.
- **VPA** — adjusts requests/limits of pods (resizes rather than adds).
- **Cluster Autoscaler / Karpenter** — adds nodes when pods are **Pending** for lack of room, removes idle nodes.

```bash
kubectl autoscale deployment api --cpu-percent=70 --min=3 --max=10
```

> [!failure] Interview traps
> - HPA with no CPU **requests** — it can't compute a percentage.
> - Memory limit too close to normal usage — random OOMKills under load.
> - Same endpoint for liveness and readiness that also checks the DB.
> - Treating Secrets as encrypted just because they're base64.

---

## Summary

| Need | Tool |
|---|---|
| **Config out of image** | ConfigMap (plain), Secret (sensitive) |
| **Health** | Startup → Readiness (traffic) → Liveness (restart) |
| **Fair sharing** | Requests (scheduling) + limits (cap) |
| **Scale** | HPA for pods, Cluster Autoscaler for nodes |

> [!success] Key takeaway
> Set requests, separate readiness from liveness, and let HPA + cluster autoscaler handle traffic swings.
