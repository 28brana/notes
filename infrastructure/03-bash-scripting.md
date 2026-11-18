# 03 · Bash Scripting

> [!info] Topic info
> **Subject:** Infrastructure · **Part:** Linux, Containers & Kubernetes · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Infrastructure](README.md) · [Interview questions →](questions/linux-containers-k8s-qa.md)

> [!abstract] In one line
> **Bash scripts** glue commands together for deploys, backups and cron jobs — write them **safely** with `set -euo pipefail`, quoted variables and clear exit codes.

---

## 1. The problem

Every deploy you run the same 6 commands by hand: pull, install, migrate, restart, check health. One day you forget the migration and prod breaks.

A script makes it repeatable — but a careless script can also `rm -rf` the wrong folder.

---

## 2. The building blocks

```bash
#!/usr/bin/env bash
set -euo pipefail                 # fail fast (see below)

APP_DIR="/srv/myapp"
ENV="${1:-staging}"               # first arg, default staging

if [[ "$ENV" == "prod" ]]; then echo "Deploying to PROD"; fi

for svc in api worker; do
  systemctl restart "$svc"
done

curl -fsS localhost:3000/health || { echo "health check failed"; exit 1; }
```

- `$1, $2` = arguments, `$#` = count, `$?` = last exit code, `$(cmd)` = command output.
- **Exit code 0 = success**, anything else = failure. `&&` runs next only on success, `||` only on failure.
- `> file` overwrites, `>> file` appends, `2>&1` sends errors to the same place.

---

## 3. Safe-script rules

| Setting | What it fixes |
|---|---|
| **set -e** | Stop on the first failing command instead of carrying on |
| **set -u** | Error on unset variables — no silent `rm -rf "$DIR/"` with empty `DIR` |
| **set -o pipefail** | A pipe fails if any part fails, not just the last |
| **Quote "$VAR"** | Spaces in paths don't split into two args |

> [!tip] 🎤 Say it like this
> "Every script I write starts with `set -euo pipefail` so it stops on the first error, fails on typos in variable names, and catches failures inside pipes. I quote all variables and make scripts idempotent so running them twice is safe."

---

## 4. Cron — running on a schedule

`crontab -e` → `min hour day month weekday command`:

```bash
0 2 * * *  /opt/scripts/backup.sh >> /var/log/backup.log 2>&1   # daily 02:00
*/5 * * * * /opt/scripts/healthcheck.sh                           # every 5 min
```

Cron runs with a **minimal PATH** and no `.bashrc` — use absolute paths and log output, or failures vanish silently.

> [!failure] Interview traps
> - No `set -e` — the script "succeeds" after a failed step.
> - Unquoted variables breaking on spaces or turning into empty strings.
> - Cron jobs that work by hand but fail in cron because of PATH / env.
> - Writing 500-line Bash — at that size, switch to Python/Node or a real tool (Ansible, CI pipeline).

---

## Summary

- Shebang + `set -euo pipefail` + quoted vars = safe default.
- Exit codes drive logic: `&&`, `||`, `if`.
- Cron: absolute paths, redirect output to a log.

> [!success] Key takeaway
> Bash is for short glue scripts — make them fail loudly and run safely twice.
