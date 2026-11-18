# Networking · Practice Questions

[← Back to Networking](../README.md) · [Q&A with answers →](networking-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](networking-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**24 questions.**

---

## 01 · OSI Model

- [ ] 🟢 Q1. What is the OSI model and which layers matter most in system design?
- [ ] 🟡 Q2. What's the difference between an L4 and an L7 load balancer?
- [ ] 🔴 Q3. You need to send /api to one service and /images to another. Can an L4 load balancer do it?

---

## 02 · IP Address

- [ ] 🟢 Q4. What's the difference between a public and a private IP?
- [ ] 🟡 Q5. What is NAT and why do private servers need it?
- [ ] 🔴 Q6. You rate limit by IP and a whole office of users gets blocked. Why?

---

## 03 · TCP vs UDP

- [ ] 🟢 Q7. What's the difference between TCP and UDP?
- [ ] 🟡 Q8. Which would you use for a video call, a payment API and DNS — and why?
- [ ] 🟡 Q9. What is head-of-line blocking and how does HTTP/3 fix it?
- [ ] 🔴 Q10. Your service opens a new database connection per request and latency is high. What's wrong?

---

## 04 · HTTP & HTTPS

- [ ] 🟢 Q11. What does it mean that HTTP is stateless, and why does it help scaling?
- [ ] 🟡 Q12. What changed from HTTP/1.1 to HTTP/2 to HTTP/3?
- [ ] 🟡 Q13. Which failed requests should a client retry?
- [ ] 🔴 Q14. Where would you terminate TLS in a large system, and what's the trade-off?

---

## 05 · DNS

- [ ] 🟢 Q15. What happens during a DNS lookup?
- [ ] 🟡 Q16. What is TTL and how do you choose it?
- [ ] 🟡 Q17. How can DNS help with global load balancing?
- [ ] 🔴 Q18. You switched DNS to the backup region 10 minutes ago, but many users still hit the dead one. Why?

---

## 06 · Proxy vs Reverse Proxy

- [ ] 🟢 Q19. What's the difference between a forward proxy and a reverse proxy?
- [ ] 🟡 Q20. What jobs does a reverse proxy do in a typical system?
- [ ] 🟡 Q21. How is a reverse proxy different from a load balancer and an API gateway?
- [ ] 🔴 Q22. After adding a reverse proxy, all your logs show the same client IP. What happened and how do you fix it?

---

## 07 · Checksums

- [ ] 🟢 Q23. What is a checksum and where is it used?
- [ ] 🟡 Q24. Does a checksum protect data from attackers?
