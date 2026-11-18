# Microservices Patterns · Practice Questions

[← Back to Microservices Patterns](../README.md) · [Q&A with answers →](microservices-patterns-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](microservices-patterns-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**23 questions.**

---

## 01 · Service Discovery

- [ ] 🟢 Q1. What is service discovery and why do microservices need it?
- [ ] 🟡 Q2. Client-side vs server-side discovery — what's the difference?
- [ ] 🟡 Q3. How does Kubernetes handle service discovery?
- [ ] 🔴 Q4. After a deploy, some requests still go to instances that were shut down. Why, and how do you fix it?

---

## 02 · API Gateway

- [ ] 🟢 Q5. What is an API gateway and what does it do?
- [ ] 🟡 Q6. API gateway vs load balancer vs reverse proxy?
- [ ] 🟡 Q7. What are the downsides of an API gateway?
- [ ] 🔴 Q8. Your gateway does all auth checks. A developer calls the Posts service directly from inside the network and edits another user's post. What went wrong?

---

## 03 · Backend for Frontend

- [ ] 🟢 Q9. What is the Backend for Frontend pattern?
- [ ] 🟡 Q10. When would you use a BFF instead of one API gateway — and when not?
- [ ] 🔴 Q11. Your mobile and web BFFs now contain the same "who can see this post" logic, and they've started to disagree. How do you fix it?

---

## 04 · Sidecar

- [ ] 🟢 Q12. What is the sidecar pattern?
- [ ] 🟡 Q13. Why use a sidecar instead of a shared library?

---

## 05 · Circuit Breaker

- [ ] 🟢 Q14. What is a circuit breaker and why do we need it?
- [ ] 🟡 Q15. Explain the three states of a circuit breaker.
- [ ] 🟡 Q16. How do you choose the thresholds, and what counts as a failure?
- [ ] 🔴 Q17. Recommendations is down and your feed page now takes 30 seconds to load, even with retries configured. What's happening and what do you do?

---

## 06 · Bulkhead

- [ ] 🟢 Q18. What is the bulkhead pattern?
- [ ] 🔴 Q19. One slow, non-critical service takes down your whole API even though you have a circuit breaker. Why?

---

## 07 · Strangler Fig

- [ ] 🟢 Q20. What is the strangler fig pattern?
- [ ] 🔴 Q21. You're asked to move a 10-year-old monolith to microservices. How do you start?

---

## 08 · Service Mesh

- [ ] 🟢 Q22. What is a service mesh?
- [ ] 🔴 Q23. Your team of 6 engineers with 8 services wants to adopt Istio. What's your advice?
