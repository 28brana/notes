# 08 · Docker Compose

> [!info] Topic info
> **Subject:** Infrastructure · **Part:** Linux, Containers & Kubernetes · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Infrastructure](README.md) · [Interview questions →](questions/linux-containers-k8s-qa.md)

> [!abstract] In one line
> **Docker Compose** describes a **multi-container app** (API + DB + Redis) in one **YAML file** and starts it with one command — great for **local dev and CI**, limited for production at scale.

---

## 1. The problem

Your app needs Postgres, Redis and a worker. A new teammate must run 4 long `docker run` commands with the right ports, networks, env vars and volumes — in the right order.

Compose puts all of that in `compose.yaml` and makes it `docker compose up`.

---

## 2. How it works

```yaml
services:
  api:
    build: .
    ports: ["3000:3000"]
    env_file: .env
    depends_on:
      db: { condition: service_healthy }
  db:
    image: postgres:16
    environment: { POSTGRES_PASSWORD: dev }
    volumes: [pgdata:/var/lib/postgresql/data]
    healthcheck: { test: ["CMD", "pg_isready"], interval: 5s }
  redis:
    image: redis:7
volumes:
  pgdata:
```

- Compose creates a **shared network** — the API connects to `db:5432` and `redis:6379` by **service name**.
- **Named volumes** keep DB data between `down` / `up`.
- `depends_on` with `service_healthy` waits for the DB to be **ready**, not just started.

```bash
docker compose up -d --build     # build and start everything
docker compose logs -f api
docker compose down              # stop + remove containers (add -v to wipe volumes)
```

> [!tip] 🎤 Say it like this
> "Compose defines all the services an app needs in one YAML file — images, ports, env, volumes and dependencies — and runs them on a shared network where they find each other by name. I use it for local dev and integration tests in CI; for production across many machines I'd use Kubernetes or a managed container service."

---

## 3. Compose vs Kubernetes

| | Docker Compose | Kubernetes |
|---|---|---|
| **Scope** | One host | A cluster of many nodes |
| **Self-healing** | Basic restart policy | Reschedules pods onto healthy nodes |
| **Scaling** | Manual `--scale` | Autoscaling (HPA) |
| **Rolling updates** | No real support | Built in |
| **Best for** | Local dev, CI, small single-server apps | Production at scale |

> [!failure] Interview traps
> - Assuming `depends_on` alone waits for the DB to accept connections — it only waits for start unless you add a healthcheck condition.
> - `docker compose down -v` in the wrong place — deletes your DB volume.
> - Committing `.env` with real secrets next to `compose.yaml`.

---

## Summary

- One YAML file → whole stack with `up` / `down`.
- Service names = hostnames; named volumes = persistent data.
- Perfect for dev and CI; Kubernetes for multi-node production.

> [!success] Key takeaway
> Compose makes "clone and run" a one-liner — reach for Kubernetes when you need many machines, self-healing and rolling deploys.
