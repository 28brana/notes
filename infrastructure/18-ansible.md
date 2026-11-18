# 18 · Ansible

> [!info] Topic info
> **Subject:** Infrastructure · **Part:** Cloud & Operations · **Priority:** 💤 Low · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Infrastructure](README.md) · [Interview questions →](questions/cloud-and-ops-qa.md)

> [!abstract] In one line
> **Ansible** is a **configuration management** tool that connects to servers over **SSH** (no agent needed) and runs **idempotent** tasks written in YAML **playbooks**.

---

## 1. The problem

Terraform created 30 EC2 servers. Now each needs Nginx installed, a config file copied, a user created and a service restarted. SSH-ing into 30 machines by hand is slow and error-prone.

---

## 2. How it works

- **Inventory** — list of hosts, grouped (`[web]`, `[db]`).
- **Playbook** — YAML file of **tasks** applied to a group.
- **Modules** — built-in actions (`apt`, `copy`, `template`, `service`, `user`).
- **Roles** — reusable bundles of tasks + templates (like Terraform modules).
- **Agentless** — just needs SSH + Python on the target. Push-based: you run it from your laptop or CI.

```yaml
- hosts: web
  become: true
  tasks:
    - name: Install nginx
      apt: { name: nginx, state: present }
    - name: Copy config
      template: { src: nginx.conf.j2, dest: /etc/nginx/nginx.conf }
      notify: restart nginx
  handlers:
    - name: restart nginx
      service: { name: nginx, state: restarted }
```

**Idempotent** = running it twice gives the same result. `state: present` installs Nginx only if missing; the handler restarts only if the config changed.

---

## 3. Ansible vs Terraform

| | **Terraform** | **Ansible** |
|---|---|---|
| **Main job** | Provision infra (VPC, VM, DB) | Configure what runs inside servers |
| **Style** | Declarative + state file | Mostly procedural tasks, no state file |
| **Typical combo** | Terraform creates the EC2 | Ansible installs and configures software on it |

> [!note] Less common today
> With containers and immutable images, config lives in the Dockerfile or a baked AMI (built with Packer), so Ansible is used less for app servers. It's still common for VMs, on-prem fleets and network devices.

> [!tip] 🎤 Say it like this
> "Ansible is agentless configuration management — it SSHes into hosts from an inventory and runs idempotent YAML tasks. I'd use Terraform to create the servers and Ansible to configure them, though with containers most of that moves into the image instead."

> [!failure] Interview traps
> - Using `shell:` / `command:` for everything — breaks idempotency. Prefer real modules.
> - Saying Ansible needs an agent on each server (that's Chef/Puppet).

---

## Summary

- Agentless, SSH, YAML playbooks, idempotent modules.
- Terraform **builds** the house, Ansible **furnishes** it.

> [!success] Key takeaway
> Know what idempotent means and how Ansible differs from Terraform — that covers most interview questions.
