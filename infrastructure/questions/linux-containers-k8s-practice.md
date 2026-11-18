# Infrastructure — Linux, Containers & Kubernetes · Practice Questions

[← Back to Infrastructure](../README.md) · [Q&A with answers →](linux-containers-k8s-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](linux-containers-k8s-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**42 questions.**

---

## 01 · Linux Essentials

- [ ] 🟢 Q1. What does `chmod 755` mean, and when would you use 644 or 600?
- [ ] 🟢 Q2. Where would you look for config files and logs on a Linux server?
- [ ] 🟡 Q3. How do you keep a Node app running after a crash or reboot on a plain VM?
- [ ] 🔴 Q4. Your app fails with `EACCES: permission denied` writing to `/var/log/myapp`. How do you fix it properly?

---

## 02 · Processes & Resources

- [ ] 🟢 Q5. What is the difference between SIGTERM and SIGKILL?
- [ ] 🟡 Q6. Load average is 6.0 — is that bad?
- [ ] 🟡 Q7. A container keeps restarting with exit code 137. What happened?
- [ ] 🔴 Q8. Users say the server is slow. Walk me through what you check.
- [ ] 🔴 Q9. `df -h` shows 40% free, but writes fail with "No space left on device". Why?

---

## 03 · Bash Scripting

- [ ] 🟢 Q10. What does `set -euo pipefail` do and why use it?
- [ ] 🟡 Q11. A script works when you run it by hand but fails from cron. Why?

---

## 04 · SSH & Server Hardening

- [ ] 🟢 Q12. How does SSH key authentication work?
- [ ] 🟡 Q13. You just launched a VM with a public IP. What do you harden first?

---

## 05 · Nginx

- [ ] 🟢 Q14. Why put Nginx in front of a Node.js app?
- [ ] 🟡 Q15. What is TLS termination and what's the trade-off?
- [ ] 🟡 Q16. Why does every request in your app logs show the same IP after adding Nginx?
- [ ] 🔴 Q17. Users get 502 and 504 errors from Nginx. How do you tell them apart and debug?

---

## 06 · Docker Basics

- [ ] 🟢 Q18. What is the difference between an image and a container?
- [ ] 🟡 Q19. How is a container different from a VM?
- [ ] 🟡 Q20. Your Postgres container lost all its data after a restart. Why?
- [ ] 🔴 Q21. `docker stop` always takes 10 seconds and the app never runs its shutdown code. Why?

---

## 07 · Dockerfile Best Practices

- [ ] 🟢 Q22. What is a multi-stage build and why use it?
- [ ] 🟡 Q23. Every small code change makes `npm install` run again in the build. How do you fix it?
- [ ] 🟡 Q24. Why run containers as non-root, and how?
- [ ] 🔴 Q25. A security scan says your image has 300 CVEs and is 1.4 GB. What do you change?

---

## 08 · Docker Compose

- [ ] 🟢 Q26. What is Docker Compose used for?
- [ ] 🟡 Q27. The API starts before Postgres is ready and crashes. How do you fix it in Compose?
- [ ] 🟡 Q28. Would you run production on Docker Compose?

---

## 09 · Kubernetes Architecture

- [ ] 🟢 Q29. What are the main components of a Kubernetes cluster?
- [ ] 🟡 Q30. What does "declarative" and "reconciliation" mean in Kubernetes?
- [ ] 🟡 Q31. Walk me through what happens when you run `kubectl apply` on a Deployment.
- [ ] 🔴 Q32. A pod is stuck in Pending. How do you debug it?

---

## 10 · Kubernetes Workloads

- [ ] 🟢 Q33. What is a pod, and why not just call a pod's IP directly?
- [ ] 🟡 Q34. Deployment vs StatefulSet vs DaemonSet — when do you use each?
- [ ] 🟡 Q35. Explain the Service types ClusterIP, NodePort and LoadBalancer, and where Ingress fits.
- [ ] 🔴 Q36. During every rolling deploy, users see a burst of 502 errors. What's wrong?

---

## 11 · Kubernetes Config & Scaling

- [ ] 🟢 Q37. ConfigMap vs Secret?
- [ ] 🟡 Q38. What's the difference between liveness, readiness and startup probes?
- [ ] 🟡 Q39. What are resource requests and limits, and what happens when you exceed them?
- [ ] 🔴 Q40. The database had a 20-second blip and every API pod in the cluster restarted. Why, and how do you prevent it?
- [ ] 🔴 Q41. HPA is configured at 70% CPU but never scales up, even under heavy load. What do you check?

---

## 12 · Helm

- [ ] 🟢 Q42. What are a Helm chart, values and a release?
