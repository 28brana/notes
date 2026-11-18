# Q&A · Reliability

[← Back to System Design](../README.md) · [Read the notes →](../04-reliability.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

> [!question]- 🟢 Q1. What is reliability?
> A system is reliable if it keeps doing the right thing — correct results, no lost or duplicated data — even when hardware, software, or people make mistakes. It's about correctness under failure, not just being up.

> [!question]- 🟢 Q2. Reliability vs availability?
> Availability asks "is it answering?"; reliability asks "is the answer correct and is the data safe?". A site that's 99.99% up but double-charges users is available but not reliable. Reliable systems are usually available, but not always the other way round.

> [!question]- 🟢 Q3. What's the difference between a fault and a failure?
> A fault is one part going wrong — a dead disk, a timed-out call, a bug. A failure is when the whole system stops giving users the right result. Fault tolerance means designing so faults don't turn into failures.

> [!question]- 🟢 Q4. What kinds of faults should you design for?
> Hardware faults like dead disks and power cuts, software faults like bugs and memory leaks, and human errors like bad configs or wrong commands. Hardware faults are random, so redundancy handles them. Bugs and human errors are correlated and actually cause most outages, so they need testing, canaries, and guardrails.

> [!question]- 🟢 Q5. What does idempotent mean? Give examples.
> Doing it once or many times gives the same result. GET, PUT with a full value, and DELETE are idempotent. POST to create something, "charge ₹500", or `likes = likes + 1` are not — repeating them changes the result.

> [!question]- 🟡 Q6. Why are retries dangerous, and how do idempotency keys fix that?
> A timeout means "I don't know" — the server may have done the work and only the reply was lost, so a retry does it twice. With an idempotency key, the client sends the same unique ID on every retry; the server stores it with a unique constraint and returns the saved result for repeats. That's how Stripe prevents double charges.

> [!question]- 🟡 Q7. Explain at-most-once, at-least-once, and exactly-once delivery.
> At-most-once acks before processing, so a crash loses the message. At-least-once acks after processing, so a crash causes a redelivery and a duplicate. Exactly-once in practice is at-least-once delivery plus idempotent consumers that skip message IDs they've already processed.

> [!question]- 🟡 Q8. What is the dual-write problem and the outbox pattern?
> When you write to the DB and publish to a queue separately, a crash in between leaves them out of sync — a post with no event, or an event with no post. The outbox pattern writes the event into an outbox table in the same DB transaction, and a relay publishes it to the queue. Consumers stay idempotent because the relay may publish twice.

> [!question]- 🟡 Q9. Is replication enough to protect your data?
> No. Replication protects against a machine or disk dying, but it copies mistakes instantly — a bad `DELETE` disappears from every replica in milliseconds. You also need point-in-time backups, ideally following 3-2-1 (3 copies, 2 media, 1 off-site), and you must regularly test restoring them.

> [!question]- 🟡 Q10. How do you keep data correct when the app might have bugs?
> Defence in depth: validate input at the API, use transactions so multi-step changes are all-or-nothing, and add database constraints like UNIQUE, FOREIGN KEY and `CHECK (balance >= 0)` as a last line of defence. Checksums catch silent corruption, and optimistic locking stops two writers overwriting each other.

> [!question]- 🟡 Q11. How do you reduce human errors in production?
> Make the safe path the easy one: least-privilege access, config changes through code review and canary, soft deletes, confirmation prompts on dangerous actions, runbooks and automation. After incidents, run blameless post-mortems that fix the system rather than blaming a person.

> [!question]- 🟡 Q12. What is a gray failure?
> A component that passes its health checks but returns wrong or partial results — like a cache serving stale data or a service returning empty lists after a bad config. Liveness checks don't catch it, so you need correctness checks: end-to-end probes, correctness SLIs, and reconciliation.

> [!question]- 🔴 Q13. Users report their post appears three times after one tap. What's happening and how do you fix it?
> The client or a proxy timed out and retried a non-idempotent POST, but the first request actually succeeded. I'd generate an idempotency key on the client when the user taps Post, send it on every retry, and store it on the server with a unique constraint so repeats return the original post. I'd also check retries happen at only one layer.

> [!question]- 🔴 Q14. Some users were charged twice for a subscription. How would you design payments to prevent and detect this?
> Prevent: idempotency key per checkout, passed through to the payment provider too; write the order and an outbox event in one transaction; state machine on the order (PENDING → PAID) so a second "paid" update is a no-op. Detect: a nightly reconciliation job comparing our payments table with the provider's report, alerting on and auto-refunding duplicates.

> [!question]- 🔴 Q15. A post's like count shows 120 but only 80 people are in the likes list. Why, and how would you fix it?
> The counter was updated non-idempotently — `count = count + 1` ran again on retried requests or redelivered events, and maybe some unlikes were lost. I'd store likes as a set with a unique (user, post) constraint so a like can only exist once, derive or update the count from that, and run a periodic recount job to repair any drift.

> [!question]- 🔴 Q16. Our notification service sometimes sends the same push 5 times and sometimes not at all. What would you change?
> "Not at all" suggests the worker acks before processing (at-most-once), so crashes lose messages. "5 times" suggests redeliveries with no dedupe. I'd switch to ack-after-processing for at-least-once, store processed message IDs to skip duplicates, and use the outbox pattern so the event is always published when the post is saved.
