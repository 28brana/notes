# 01 · Linux Essentials

> [!info] Topic info
> **Subject:** Infrastructure · **Part:** Linux, Containers & Kubernetes · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Infrastructure](README.md) · [Interview questions →](questions/linux-containers-k8s-qa.md)

> [!abstract] In one line
> Almost every server runs **Linux** — you need to know the **filesystem layout**, **permissions** (`chmod`, `chown`, `755`), **users/groups** and **systemd services** to deploy and debug your own code.

---

## 1. The problem

You SSH into a fresh Ubuntu server to deploy your Node API. Where do configs live? Why does the app say `EACCES: permission denied` on `/var/log/myapp`? How do you keep it running after a reboot?

That's all Linux basics — and interviewers expect a backend dev to answer them without googling.

---

## 2. Filesystem layout

Everything is a file under one root `/`. The folders you'll actually touch:

| Path | What lives there |
|---|---|
| **/etc** | Config files — `/etc/nginx/nginx.conf`, `/etc/hosts` |
| **/var** | Changing data — logs in `/var/log`, DB files in `/var/lib` |
| **/home** | User home dirs (`/home/deploy`); root's is `/root` |
| **/opt**, **/srv** | Third-party apps / your app code |
| **/usr/bin**, **/bin** | Installed programs |
| **/tmp** | Temp files, cleared on reboot |
| **/proc** | Virtual files about running processes (`/proc/<pid>/`) |
| **/dev** | Devices — disks (`/dev/sda`), `/dev/null` |

---

## 3. Permissions

`ls -l` shows `-rwxr-xr--  deploy  www-data  app.sh` → type, then 3 groups of `rwx` for **owner**, **group**, **others**.

Each letter is a number: **r = 4, w = 2, x = 1**. Add them per group:

| Mode | Meaning | Typical use |
|---|---|---|
| **755** | owner rwx, others r-x | Scripts, directories |
| **644** | owner rw, others read | Normal files, configs |
| **600** | owner only rw | SSH private keys, `.env` |
| **777** | everyone everything | ❌ Almost never — security hole |

```bash
chmod 755 deploy.sh             # set mode
chmod +x deploy.sh              # just add execute
chown deploy:www-data app.log   # change owner and group
chown -R deploy /srv/myapp      # recursive
```

> [!note] `x` on a directory
> For directories, `x` means "can enter / traverse it". A folder with `r` but no `x` lets you list names but not open files inside.

---

## 4. Users, groups and sudo

- Each user has a **UID**; **root** is UID 0 and can do anything.
- **Groups** share access — add `deploy` to `docker` group so it can run Docker without sudo.
- `sudo` runs one command as root, logged and permission-checked. Never run your app as root.

```bash
sudo useradd -m -s /bin/bash deploy
sudo usermod -aG docker deploy   # -a = append, without it you REPLACE groups
id deploy                        # show uid, gid, groups
```

---

## 5. systemd services

**systemd** is the first process (PID 1). It starts services at boot, restarts them if they crash, and collects their logs.

```ini
# /etc/systemd/system/myapp.service
[Service]
User=deploy
WorkingDirectory=/srv/myapp
ExecStart=/usr/bin/node server.js
Restart=always
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload && sudo systemctl enable --now myapp
systemctl status myapp       # running? last log lines
journalctl -u myapp -f       # follow logs
```

> [!tip] 🎤 Say it like this
> "On a VM I'd run the app as a non-root user under systemd with `Restart=always`, so it starts at boot and comes back after a crash. Configs go in `/etc`, logs in `/var/log` or journald, and secrets files get `600` permissions."

---

## 6. Commands you should know cold

| Task | Command |
|---|---|
| **Find text in files** | `grep -rn "ERROR" /var/log/myapp` |
| **Find files** | `find / -name "*.log" -size +100M` |
| **Watch a log** | `tail -f app.log`, `less +F app.log` |
| **Who uses a port** | `ss -tlnp \| grep 3000` (or `lsof -i :3000`) |
| **Test an endpoint** | `curl -v localhost:3000/health` |
| **Pipe and count** | `grep 500 access.log \| wc -l` |
| **Environment** | `env`, `export PORT=3000`, `echo $PATH` |

> [!failure] Interview traps
> - "Just `chmod 777` it" — fixes nothing properly and opens a hole. Fix the **owner** instead.
> - `usermod -G` without `-a` wipes the user's other groups.
> - Running the app as root "because port 80 needs it" — put Nginx in front, or use `CAP_NET_BIND_SERVICE`.
> - Starting the app with `nohup node app &` in production — no restart, no boot start. Use systemd or a container.

---

## Summary

- `/etc` config, `/var/log` logs, `/proc` process info.
- Permissions = owner/group/others × r4 w2 x1 → **755**, **644**, **600**.
- Least privilege: dedicated user, groups for shared access, `sudo` only when needed.
- **systemd** keeps services alive; **journalctl** reads their logs.

> [!success] Key takeaway
> Know where things live, who owns them, and how a service is kept running — that covers most "it works locally but not on the server" bugs.
