# Q&A · Infrastructure — Cloud & Operations

[← Back to Infrastructure](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 13 · Cloud Fundamentals

[Read the notes →](../13-cloud-fundamentals.md)

> [!question]- 🟢 Q1. What is the difference between IaaS, PaaS and SaaS?
> It's about how much of the stack you manage. With IaaS, like EC2, you get VMs and manage the OS, runtime and app. With PaaS, like Heroku or Elastic Beanstalk, you just push code and the platform handles servers and scaling. With SaaS, like Gmail or Slack, you just use the finished software.

> [!question]- 🟢 Q2. What is a region vs an Availability Zone?
> A region is a geographic area like Mumbai or N. Virginia. An Availability Zone is one or more isolated data centers inside that region with their own power and network, a few milliseconds apart. Spreading across AZs protects you from a single data-center failure; spreading across regions protects you from a regional outage.

> [!question]- 🟡 Q3. Explain the shared responsibility model with an example.
> The provider secures the cloud itself — buildings, hardware, network, hypervisor. You secure what you put in it — IAM, data, encryption, security groups and, on EC2, OS patches. On RDS AWS patches the database engine, but if you leave the DB publicly accessible with a weak password, that's still your fault.

---

## 14 · AWS Core Services

[Read the notes →](../14-aws-core-services.md)

> [!question]- 🟢 Q4. When would you use S3 vs EBS vs RDS?
> S3 is object storage for files like images, backups and static sites, accessed over HTTP. EBS is a block disk attached to one EC2 instance, like a hard drive. RDS is a managed relational database for structured data with transactions and queries.

> [!question]- 🟡 Q5. What's the difference between SQS and SNS?
> SQS is a queue: messages wait until a consumer pulls and deletes them, so each message is handled by one worker. SNS is pub/sub: it pushes one message to all subscribers at once. A common pattern is SNS fanning out to several SQS queues so each service processes the event independently.

> [!question]- 🟡 Q6. RDS Multi-AZ vs read replicas — what's the difference?
> Multi-AZ keeps a synchronous standby in another AZ for automatic failover — it's for availability, and you don't read from it. Read replicas are asynchronous copies you can send read queries to — they're for scaling reads, and may lag a bit. Many production setups use both.

> [!question]- 🔴 Q7. Your Lambda-based API is fast most of the time but some requests take 2 seconds, and the DB starts refusing connections at peak. What's happening?
> The slow requests are likely cold starts — a new Lambda instance has to boot after being idle or during scale-out. The DB issue is because each concurrent Lambda opens its own connection, so 1,000 parallel invocations means 1,000 connections. I'd use RDS Proxy to pool connections, provisioned concurrency for latency-critical functions, and keep the bundle small to speed up cold starts.

---

## 15 · VPC Networking

[Read the notes →](../15-vpc-networking.md)

> [!question]- 🟢 Q8. What makes a subnet public or private?
> Only its route table. A public subnet has a route `0.0.0.0/0` to an Internet Gateway, so resources with public IPs can be reached from the internet. A private subnet has no such route — usually it sends outbound traffic through a NAT Gateway instead.

> [!question]- 🟡 Q9. Security groups vs NACLs?
> Security groups attach to instances, are stateful so replies are automatically allowed, and only support allow rules. NACLs attach to subnets, are stateless so you must allow both directions, support deny rules, and are evaluated in number order. Security groups do most of the work; NACLs are an extra subnet-level guardrail, e.g. to block an IP range.

> [!question]- 🟡 Q10. Design a VPC for a typical 3-tier web app.
> One VPC, say `10.0.0.0/16`, with subnets in at least two AZs per tier. Public subnets hold the ALB and NAT gateways; private app subnets hold the servers; private DB subnets hold RDS. Security groups chain the tiers: ALB allows 443 from the internet, app allows traffic only from the ALB's SG, DB allows 5432 only from the app's SG.

> [!question]- 🔴 Q11. Servers in a private subnet can't download packages from the internet. What do you check?
> First the route table of the private subnet: it needs `0.0.0.0/0` pointing to a NAT Gateway, and the NAT must sit in a public subnet whose route goes to the Internet Gateway. Then check the security group allows outbound traffic and that NACLs allow both the outbound request and the returning ephemeral ports. For AWS services like S3, a VPC endpoint is a cheaper alternative to NAT.

---

## 16 · IAM

[Read the notes →](../16-iam.md)

> [!question]- 🟢 Q12. What's the difference between an IAM user and an IAM role?
> A user has long-term credentials — a password or access keys — and represents a person or app. A role has no permanent credentials; it's assumed by a service, user or external identity, which gets temporary keys that expire. Roles are preferred because a leaked temporary key stops working soon.

> [!question]- 🟡 Q13. How does AWS decide whether a request is allowed?
> Everything is denied by default. If any applicable policy explicitly allows the action on that resource, it's allowed — unless some policy has an explicit deny, which always wins. Organization SCPs and permission boundaries can further limit what's possible.

> [!question]- 🔴 Q14. An AWS access key was pushed to a public GitHub repo. What do you do?
> Immediately deactivate and delete the key, then check CloudTrail for what it was used for and look for unknown resources like new EC2 instances or IAM users. Clean up anything created, rotate any other secrets it could reach, and remove it from Git history. Long term, replace access keys with roles — instance roles for servers and OIDC for CI — and add secret scanning to the repo.

---

## 17 · Terraform

[Read the notes →](../17-terraform.md)

> [!question]- 🟢 Q15. What do `terraform plan` and `terraform apply` do?
> `plan` compares your code with the state file and real infrastructure and shows what will be created, changed or destroyed, without changing anything. `apply` executes those changes and updates the state. In a team, the plan is usually posted on the PR for review before apply runs.

> [!question]- 🟡 Q16. What is Terraform state and why store it remotely with locking?
> State maps each resource in your code to the real resource ID in the cloud, so Terraform knows what it manages. If it's on a laptop, nobody else can safely run Terraform, and it can be lost. Remote state in S3 gives one shared source of truth, and locking stops two people from applying at the same time and corrupting it.

> [!question]- 🟡 Q17. What is drift and how do you handle it?
> Drift is when real infrastructure no longer matches the code — usually because someone changed something in the console. `terraform plan` shows it as a diff. You either apply to put it back, or update the code if the change was intended, and reduce drift by removing manual write access in prod.

> [!question]- 🔴 Q18. A teammate's PR renames a resource and the plan says the production database will be destroyed and recreated. What now?
> Don't apply it. Renaming a resource address makes Terraform think it's a new resource. Use a `moved` block or `terraform state mv` so state maps the old resource to the new name, and re-run plan until it shows no destroy. Add `prevent_destroy` lifecycle rules on critical resources like databases so this can't slip through again.

---

## 18 · Ansible

[Read the notes →](../18-ansible.md)

> [!question]- 🟢 Q19. What is Ansible and how does it differ from Terraform?
> Ansible is agentless configuration management: it SSHes into servers from an inventory and runs idempotent YAML tasks, like installing Nginx or copying config. Terraform provisions infrastructure such as VPCs, VMs and databases and tracks it in state. A common combo is Terraform to create servers and Ansible to configure them.

---

## 19 · CI/CD Pipelines

[Read the notes →](../19-ci-cd-pipelines.md)

> [!question]- 🟢 Q20. Walk me through a CI/CD pipeline you would set up.
> On every PR: lint, type check, unit tests and a build. On merge to main: build one Docker image tagged with the commit SHA, scan it, push it to the registry, deploy to staging and run smoke tests. Then promote the same image to prod with a rolling or canary deploy and automatic rollback on failed health checks.

> [!question]- 🟡 Q21. Why build the artifact once and promote it, instead of building per environment?
> If you rebuild for prod, dependencies or base images might differ, so prod runs something staging never tested. Building once and tagging with the commit SHA means the exact tested artifact reaches prod. It also makes rollback simple — just redeploy the previous SHA.

> [!question]- 🔴 Q22. Your pipeline takes 40 minutes and developers are bypassing it. How do you fix it?
> Measure which steps are slow first. Then cache dependencies and Docker layers, run independent jobs in parallel, split or shard the test suite, and only run expensive end-to-end tests on main rather than every PR. Fix or quarantine flaky tests, since retries waste a lot of time. Aim for PR checks under about 10 minutes.

---

## 20 · Prometheus & Grafana

[Read the notes →](../20-prometheus-and-grafana.md)

> [!question]- 🟢 Q23. How does Prometheus collect metrics?
> It uses a pull model: each service exposes a `/metrics` endpoint and Prometheus scrapes it every few seconds, storing time series with labels. For things you don't control, like Linux hosts or Postgres, you run exporters. Grafana then queries Prometheus with PromQL to draw dashboards, and Alertmanager routes alerts.

> [!question]- 🔴 Q24. Prometheus memory usage keeps growing after a new release. What's a likely cause?
> Most likely a high-cardinality label — for example someone added `user_id` or the full URL as a label, so every unique value creates a new time series. Check the top series counts per metric to find it. Fix it by using bounded labels like the route template and status code, and put per-user detail in logs or traces instead.

---

## 21 · Logging Stack

[Read the notes →](../21-logging-stack.md)

> [!question]- 🟢 Q25. How do logs get from a container to a central place where you can search them?
> The app writes structured JSON to stdout. An agent like Fluent Bit runs on each node as a DaemonSet, tails container logs, adds metadata like pod and namespace, and ships them to a store such as Elasticsearch, Loki or CloudWatch. You search them in Kibana or Grafana.

> [!question]- 🟡 Q26. ELK vs Loki — when would you choose each?
> Elasticsearch indexes every field, so full-text search and analytics are powerful, but it's heavy and expensive at high volume. Loki only indexes labels and compresses the log text, so it's much cheaper and fits well with Prometheus and Grafana, but free-text searches are slower. I'd pick Loki for cost-sensitive general logging and ELK when rich search is a real need.

---

## 22 · Autoscaling & Cost

[Read the notes →](../22-autoscaling-and-cost.md)

> [!question]- 🟢 Q27. What is an Auto Scaling Group?
> It's a group of EC2 instances that AWS keeps between a min and max size, launched from a template. Scaling policies like target tracking add or remove instances based on a metric such as CPU. It also replaces unhealthy instances automatically, so it gives self-healing as well as scaling.

> [!question]- 🟡 Q28. When would you use spot, reserved and on-demand instances?
> Reserved instances or savings plans for the steady baseline that's always running, since they're 30–70% cheaper with a commitment. Spot for interruptible, stateless work like batch jobs, CI runners or queue workers — up to 90% cheaper but can be reclaimed with two minutes' notice. On-demand for spikes and anything unpredictable.

> [!question]- 🔴 Q29. You added autoscaling but the site is still slow at peak even with 20 servers. Why?
> The bottleneck probably isn't the app servers — often it's the database, a downstream API, or a connection limit, and more servers just add more pressure on it. Check DB CPU, connections and slow queries. Also check scaling speed: if instances take minutes to boot, they arrive after the spike, so use scheduled or predictive scaling and faster-booting images.

---

## 23 · Backups & Disaster Recovery

[Read the notes →](../23-backups-and-disaster-recovery.md)

> [!question]- 🟢 Q30. What are RPO and RTO?
> RPO is the maximum data loss you can accept, measured in time — RPO 5 minutes means losing at most the last 5 minutes of writes. RTO is the maximum downtime — how fast the service must be back. Lower values for both cost more, so they come from business requirements.

> [!question]- 🟡 Q31. Compare backup-and-restore, pilot light, warm standby and active-active.
> Backup-and-restore only keeps backups in the DR region — cheapest, but hours to recover. Pilot light replicates core data with servers switched off, so recovery takes tens of minutes. Warm standby runs a small full copy you scale up, recovering in minutes. Active-active serves traffic from both regions, so near-zero downtime, but it's the most expensive and complex.

> [!question]- 🔴 Q32. Someone ran a DELETE without a WHERE clause on production. You have replicas. How do you recover?
> Replicas don't help — they copied the delete within milliseconds. I'd use point-in-time recovery to restore the database to a new instance just before the delete, then either copy the missing rows back into prod or switch traffic to the restored instance, depending on how many writes happened since. Afterwards: restrict direct prod write access and add review for manual queries.

---

## 24 · Incident Response

[Read the notes →](../24-incident-response.md)

> [!question]- 🟢 Q33. What are the main steps of incident response?
> Detect through alerts or reports, triage and set a severity, mitigate to stop user impact, resolve the root cause, then write a post-mortem. Communication runs through all of it — internal updates and the status page.

> [!question]- 🟡 Q34. Why "mitigate first, debug later"?
> Users care about the service working again, not about the root cause. A rollback or turning off a feature flag can restore service in minutes, while debugging can take hours. Once things are stable, you investigate calmly with logs and metrics.

> [!question]- 🟡 Q35. What makes a good post-mortem?
> A clear timeline, the impact in users and minutes, the root cause and contributing factors, and concrete action items with owners and dates. It must be blameless — it asks why the system allowed the mistake, not who made it — otherwise people hide problems next time.

> [!question]- 🔴 Q36. You're on call and errors jump to 30% right after a deploy at 2 am. What do you do?
> Acknowledge the page and declare a SEV1 or SEV2. Since it lines up with a deploy, the first move is to roll back to the previous version, then watch the error rate drop. I'd post updates in the incident channel and status page, and only after recovery dig into what in the release caused it, followed by a post-mortem with action items like a canary step that would have caught it.
