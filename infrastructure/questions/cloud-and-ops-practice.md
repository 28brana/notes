# Infrastructure — Cloud & Operations · Practice Questions

[← Back to Infrastructure](../README.md) · [Q&A with answers →](cloud-and-ops-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](cloud-and-ops-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**36 questions.**

---

## 13 · Cloud Fundamentals

- [ ] 🟢 Q1. What is the difference between IaaS, PaaS and SaaS?
- [ ] 🟢 Q2. What is a region vs an Availability Zone?
- [ ] 🟡 Q3. Explain the shared responsibility model with an example.

---

## 14 · AWS Core Services

- [ ] 🟢 Q4. When would you use S3 vs EBS vs RDS?
- [ ] 🟡 Q5. What's the difference between SQS and SNS?
- [ ] 🟡 Q6. RDS Multi-AZ vs read replicas — what's the difference?
- [ ] 🔴 Q7. Your Lambda-based API is fast most of the time but some requests take 2 seconds, and the DB starts refusing connections at peak. What's happening?

---

## 15 · VPC Networking

- [ ] 🟢 Q8. What makes a subnet public or private?
- [ ] 🟡 Q9. Security groups vs NACLs?
- [ ] 🟡 Q10. Design a VPC for a typical 3-tier web app.
- [ ] 🔴 Q11. Servers in a private subnet can't download packages from the internet. What do you check?

---

## 16 · IAM

- [ ] 🟢 Q12. What's the difference between an IAM user and an IAM role?
- [ ] 🟡 Q13. How does AWS decide whether a request is allowed?
- [ ] 🔴 Q14. An AWS access key was pushed to a public GitHub repo. What do you do?

---

## 17 · Terraform

- [ ] 🟢 Q15. What do `terraform plan` and `terraform apply` do?
- [ ] 🟡 Q16. What is Terraform state and why store it remotely with locking?
- [ ] 🟡 Q17. What is drift and how do you handle it?
- [ ] 🔴 Q18. A teammate's PR renames a resource and the plan says the production database will be destroyed and recreated. What now?

---

## 18 · Ansible

- [ ] 🟢 Q19. What is Ansible and how does it differ from Terraform?

---

## 19 · CI/CD Pipelines

- [ ] 🟢 Q20. Walk me through a CI/CD pipeline you would set up.
- [ ] 🟡 Q21. Why build the artifact once and promote it, instead of building per environment?
- [ ] 🔴 Q22. Your pipeline takes 40 minutes and developers are bypassing it. How do you fix it?

---

## 20 · Prometheus & Grafana

- [ ] 🟢 Q23. How does Prometheus collect metrics?
- [ ] 🔴 Q24. Prometheus memory usage keeps growing after a new release. What's a likely cause?

---

## 21 · Logging Stack

- [ ] 🟢 Q25. How do logs get from a container to a central place where you can search them?
- [ ] 🟡 Q26. ELK vs Loki — when would you choose each?

---

## 22 · Autoscaling & Cost

- [ ] 🟢 Q27. What is an Auto Scaling Group?
- [ ] 🟡 Q28. When would you use spot, reserved and on-demand instances?
- [ ] 🔴 Q29. You added autoscaling but the site is still slow at peak even with 20 servers. Why?

---

## 23 · Backups & Disaster Recovery

- [ ] 🟢 Q30. What are RPO and RTO?
- [ ] 🟡 Q31. Compare backup-and-restore, pilot light, warm standby and active-active.
- [ ] 🔴 Q32. Someone ran a DELETE without a WHERE clause on production. You have replicas. How do you recover?

---

## 24 · Incident Response

- [ ] 🟢 Q33. What are the main steps of incident response?
- [ ] 🟡 Q34. Why "mitigate first, debug later"?
- [ ] 🟡 Q35. What makes a good post-mortem?
- [ ] 🔴 Q36. You're on call and errors jump to 30% right after a deploy at 2 am. What do you do?
