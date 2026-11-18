# Infrastructure

[← Back to all subjects](../README.md)

How your code actually runs in production — Linux servers, containers, Kubernetes, the cloud, and keeping it all healthy.
Interviewers expect a backend developer to deploy, scale and debug their own services, not just write them.

## Part 1 · Linux, Containers & Kubernetes
[Interview Q&A →](questions/linux-containers-k8s-qa.md) · [Practice questions 📝](questions/linux-containers-k8s-practice.md)

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [Linux Essentials](01-linux-essentials.md) | Filesystem layout, permissions (755/644/600), users/groups, systemd, key commands | 🔥 High |
| 02 | [Processes & Resources](02-processes-and-resources.md) | Signals, load average, memory/OOM, disk/inodes, FDs — the "server is slow" checklist | 🔥 High |
| 03 | [Bash Scripting](03-bash-scripting.md) | Safe scripts with `set -euo pipefail`, exit codes, cron | ⚡ Medium |
| 04 | [SSH & Server Hardening](04-ssh-and-server-hardening.md) | Key-based SSH, bastions, firewall, no root/password login | ⚡ Medium |
| 05 | [Nginx](05-nginx.md) | Reverse proxy: TLS termination, load balancing, gzip, static files, 502 vs 504 | 🔥 High |
| 06 | [Docker Basics](06-docker-basics.md) | Image vs container, layers, namespaces + cgroups, volumes, networks | 🔥 High |
| 07 | [Dockerfile Best Practices](07-dockerfile-best-practices.md) | Multi-stage, slim base, layer caching, .dockerignore, non-root, signals | 🔥 High |
| 08 | [Docker Compose](08-docker-compose.md) | Multi-container apps in one YAML for local dev and CI | ⚡ Medium |
| 09 | [Kubernetes Architecture](09-kubernetes-architecture.md) | Control plane vs nodes, desired state and reconcile loops | 🔥 High |
| 10 | [Kubernetes Workloads](10-kubernetes-workloads.md) | Pods, Deployments, Services, Ingress, StatefulSets, DaemonSets, Jobs | 🔥 High |
| 11 | [Kubernetes Config & Scaling](11-kubernetes-config-and-scaling.md) | ConfigMaps/Secrets, probes, requests/limits, HPA | 🔥 High |
| 12 | [Helm](12-helm.md) | Charts, values and releases — package manager for Kubernetes | 💤 Low |

## Part 2 · Cloud & Operations
[Interview Q&A →](questions/cloud-and-ops-qa.md) · [Practice questions 📝](questions/cloud-and-ops-practice.md)

| # | Topic | In one line | Priority |
|---|---|---|---|
| 13 | [Cloud Fundamentals](13-cloud-fundamentals.md) | IaaS/PaaS/SaaS, regions and availability zones, shared responsibility | 🔥 High |
| 14 | [AWS Core Services](14-aws-core-services.md) | EC2, S3, RDS, Lambda, ECS/EKS and when to pick each | 🔥 High |
| 15 | [VPC Networking](15-vpc-networking.md) | Subnets, route tables, NAT, security groups vs NACLs | 🔥 High |
| 16 | [IAM](16-iam.md) | Who can do what on which resource — policies, roles, least privilege | 🔥 High |
| 17 | [Terraform](17-terraform.md) | Infrastructure as code: plan, apply and the state file | 🔥 High |
| 18 | [Ansible](18-ansible.md) | Agentless configuration management with idempotent YAML playbooks | 💤 Low |
| 19 | [CI/CD Pipelines](19-ci-cd-pipelines.md) | Build, test, scan and deploy automatically on every push | 🔥 High |
| 20 | [Prometheus & Grafana](20-prometheus-and-grafana.md) | Pull-based metrics, dashboards and alerting | ⚡ Medium |
| 21 | [Logging Stack](21-logging-stack.md) | Structured logs shipped to a central store (ELK / Loki) | ⚡ Medium |
| 22 | [Autoscaling & Cost](22-autoscaling-and-cost.md) | Scale with load; rightsizing, reserved and spot to cut the bill | ⚡ Medium |
| 23 | [Backups & Disaster Recovery](23-backups-and-disaster-recovery.md) | RPO, RTO, tested backups and standby strategies | ⚡ Medium |
| 24 | [Incident Response](24-incident-response.md) | Detect, mitigate first, communicate, blameless post-mortem | 🔥 High |

> [!tip] Related
> System-level theory lives in [System Design](../system-design/README.md) — especially deployment, observability and security chapters.
