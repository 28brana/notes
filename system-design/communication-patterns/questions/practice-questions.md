# Communication Patterns · Practice Questions

[← Back to Communication Patterns](../README.md) · [Q&A with answers →](communication-patterns-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](communication-patterns-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**34 questions.**

---

## 01 · Long Polling

- [ ] 🟢 Q1. What is long polling and how is it different from short polling?
- [ ] 🟡 Q2. When would you still choose long polling today?
- [ ] 🔴 Q3. Your long-polling clients keep getting 504 errors after 60 seconds. What's wrong?

---

## 02 · Server-Sent Events

- [ ] 🟢 Q4. What is SSE and when would you use it over WebSockets?
- [ ] 🟡 Q5. How does SSE handle a dropped connection without losing events?
- [ ] 🔴 Q6. Your SSE events arrive in big bursts every few seconds instead of instantly. Why?

---

## 03 · WebSockets

- [ ] 🟢 Q7. How does a WebSocket connection get established?
- [ ] 🟡 Q8. Why are WebSockets harder to scale than a REST API?
- [ ] 🔴 Q9. Alice is connected to server 1 and Bob to server 3. How does Alice's chat message reach Bob?
- [ ] 🔴 Q10. After every deploy, your chat servers get overloaded for a few minutes. What's happening?

---

## 04 · Webhooks

- [ ] 🟢 Q11. What is a webhook?
- [ ] 🟡 Q12. How do you receive webhooks safely?
- [ ] 🔴 Q13. Customers report being charged once but their order is marked paid twice and gets two receipts. Why?
- [ ] 🔴 Q14. You're the provider. How do you design reliable webhook delivery to thousands of customers?

---

## 05 · WebRTC

- [ ] 🟢 Q15. What are signaling, STUN and TURN in WebRTC?
- [ ] 🟡 Q16. Why does a 20-person video call need an SFU instead of pure peer-to-peer?

---

## 06 · Message Queues

- [ ] 🟢 Q17. Why would you put a message queue between two services?
- [ ] 🟡 Q18. What happens if a worker crashes halfway through a message?
- [ ] 🔴 Q19. Your queue depth keeps growing all day and never goes down. What do you check?
- [ ] 🔴 Q20. The same video gets transcoded twice at the same time. Why?

---

## 07 · Pub/Sub

- [ ] 🟢 Q21. What's the difference between a message queue and pub/sub?
- [ ] 🟡 Q22. When a user creates a post, five services must react. Would you call them directly or use pub/sub?
- [ ] 🔴 Q23. You used Redis Pub/Sub for order events and some orders were never shipped. Why?

---

## 08 · Message Brokers

- [ ] 🟢 Q24. What's the main difference between Kafka and RabbitMQ?
- [ ] 🟡 Q25. How does Kafka scale consumers and keep ordering?
- [ ] 🔴 Q26. You added 10 more consumers to your Kafka group but throughput didn't change. Why?
- [ ] 🔴 Q27. Kafka, RabbitMQ or SQS for a notification system on AWS? Justify.

---

## 09 · Delivery Semantics

- [ ] 🟢 Q28. Explain at-most-once, at-least-once and exactly-once.
- [ ] 🟡 Q29. How does offset commit timing affect delivery in Kafka?
- [ ] 🔴 Q30. Kafka says it supports exactly-once. Can your consumer that writes to Postgres and sends emails rely on that?
- [ ] 🔴 Q31. Order events arrive out of order — "paid" after "shipped". How do you handle it?

---

## 10 · Dead Letter Queues

- [ ] 🟢 Q32. What is a dead letter queue and why do you need one?
- [ ] 🟡 Q33. Should every failed message be retried before going to the DLQ?
- [ ] 🔴 Q34. You fixed a bug and redrove 50k messages from the DLQ, and now some customers got duplicate emails. What went wrong?
