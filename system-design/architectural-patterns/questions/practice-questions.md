# Architectural Patterns · Practice Questions

[← Back to Architectural Patterns](../README.md) · [Q&A with answers →](architectural-patterns-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](architectural-patterns-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**28 questions.**

---

## 01 · Client-Server

- [ ] 🟢 Q1. What is client-server architecture?
- [ ] 🟢 Q2. What is a 3-tier architecture and why is it the standard?
- [ ] 🟡 Q3. Why should the server never trust the client?
- [ ] 🔴 Q4. The single app server is maxed out at 100% CPU during peaks. What do you do?

---

## 02 · Monolith

- [ ] 🟢 Q5. What is a monolith, and what are its main advantages?
- [ ] 🟡 Q6. What is a modular monolith and why do people recommend starting with one?
- [ ] 🔴 Q7. Your monolith takes 45 minutes to deploy and teams keep blocking each other. Do you go straight to microservices?

---

## 03 · Microservices

- [ ] 🟢 Q8. What are microservices?
- [ ] 🟡 Q9. Monolith vs microservices — how do you choose?
- [ ] 🟡 Q10. Why should each microservice have its own database?
- [ ] 🔴 Q11. After splitting into microservices, the app got slower and outages got more frequent. Why?

---

## 04 · Serverless

- [ ] 🟢 Q12. What is serverless, and when is it a good fit?
- [ ] 🟡 Q13. What is a cold start and how do you reduce it?
- [ ] 🔴 Q14. During a traffic spike your Lambda functions start failing with "too many connections" from Postgres. What happened?

---

## 05 · Event-Driven Architecture

- [ ] 🟢 Q15. What is event-driven architecture?
- [ ] 🟡 Q16. What's the difference between an event and a command?
- [ ] 🟡 Q17. Choreography vs orchestration — when do you use each?
- [ ] 🔴 Q18. Users sometimes get the same notification twice, and some post-created events seem to vanish. What's wrong?

---

## 06 · Peer-to-Peer

- [ ] 🟢 Q19. What is peer-to-peer and how is it different from client-server?
- [ ] 🟡 Q20. What are the main challenges of P2P systems?

---

## 07 · Hexagonal Architecture

- [ ] 🟢 Q21. What is hexagonal architecture?
- [ ] 🟡 Q22. What benefit does hexagonal architecture give you in practice?

---

## 08 · CQRS

- [ ] 🟢 Q23. What is CQRS?
- [ ] 🟡 Q24. When is CQRS worth it, and when is it overkill?
- [ ] 🔴 Q25. With CQRS in place, users complain their new post doesn't show up right after they publish it. How do you fix it?

---

## 09 · Event Sourcing

- [ ] 🟢 Q26. What is event sourcing?
- [ ] 🟡 Q27. Event sourcing vs event-driven architecture — what's the difference?
- [ ] 🔴 Q28. Your event-sourced account service is getting slower every month. Why, and what do you do?
