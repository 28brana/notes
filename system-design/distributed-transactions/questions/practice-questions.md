# Distributed Transactions · Practice Questions

[← Back to Distributed Transactions](../README.md) · [Q&A with answers →](distributed-transactions-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](distributed-transactions-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**15 questions.**

---

## 01 · The Distributed Transactions Problem

- [ ] 🟢 Q1. What is a distributed transaction, and why can't we just use a normal database transaction?
- [ ] 🟡 Q2. A call to the payment service times out. Did the charge happen?
- [ ] 🔴 Q3. Users report being charged for Premium but not getting it. What's likely wrong and how do you fix it?

---

## 02 · Two-Phase Commit (2PC)

- [ ] 🟢 Q4. Explain two-phase commit.
- [ ] 🟡 Q5. Why is 2PC called a blocking protocol?
- [ ] 🔴 Q6. Your team wants 2PC across the Payment, Subscription and Notification microservices. What do you say?

---

## 03 · Three-Phase Commit (3PC)

- [ ] 🟢 Q7. What does 3PC add over 2PC?
- [ ] 🟡 Q8. If 3PC is non-blocking, why isn't it used in practice?

---

## 04 · Saga Pattern

- [ ] 🟢 Q9. What is a saga and what is a compensating action?
- [ ] 🟡 Q10. Choreography vs orchestration — when do you pick which?
- [ ] 🟡 Q11. Sagas don't have isolation. What problems does that cause and how do you handle it?
- [ ] 🔴 Q12. A user got charged twice during checkout after a network glitch. What went wrong in the saga?

---

## 05 · Outbox Pattern

- [ ] 🟢 Q13. What problem does the outbox pattern solve?
- [ ] 🟡 Q14. Polling vs CDC for the outbox relay — trade-offs?
- [ ] 🔴 Q15. Consumers are processing some events twice and sometimes out of order. How do you fix it?
