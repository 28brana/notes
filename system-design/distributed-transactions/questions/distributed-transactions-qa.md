# Q&A · Distributed Transactions

[← Back to Distributed Transactions](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · The Distributed Transactions Problem

[Read the notes →](../01-distributed-transactions-problem.md)

> [!question]- 🟢 Q1. What is a distributed transaction, and why can't we just use a normal database transaction?
> It's one business action — like buying Premium — that changes data in several services, each with its own database. A normal ACID transaction only works inside one database. With database-per-service, nobody can commit all the parts together, so we need 2PC or a saga.

> [!question]- 🟡 Q2. A call to the payment service times out. Did the charge happen?
> We don't know — a timeout means "no answer", not "failed". The charge may have succeeded and the reply was lost. So we retry with the same idempotency key, or query the payment status, and never blindly charge again.

> [!question]- 🔴 Q3. Users report being charged for Premium but not getting it. What's likely wrong and how do you fix it?
> Payment committed but the subscription step failed or never ran — a partial failure, probably a crash between the two or a lost event. Fix it with a saga that compensates (refund) or retries the activation, and an outbox so the "payment done" event can't be lost. Also run a reconciliation job to find and fix existing mismatches.

---

## 02 · Two-Phase Commit (2PC)

[Read the notes →](../02-two-phase-commit.md)

> [!question]- 🟢 Q4. Explain two-phase commit.
> A coordinator asks every participant to prepare — do the work, lock, and vote yes or no. If all vote yes, it tells everyone to commit; if any says no or times out, everyone aborts. This gives all-or-nothing across databases.

> [!question]- 🟡 Q5. Why is 2PC called a blocking protocol?
> After a participant votes yes, it must wait for the decision and keep its locks. If the coordinator crashes then, the participant can't commit or abort on its own, so it blocks — and other transactions touching those rows block too — until the coordinator recovers.

> [!question]- 🔴 Q6. Your team wants 2PC across the Payment, Subscription and Notification microservices. What do you say?
> I'd push back. It ties the availability of all three together, holds locks across network calls, blocks if the coordinator fails, and external providers like Stripe don't support a prepare phase anyway. I'd use a saga with compensations and an outbox instead, and keep 2PC for inside a single distributed database.

---

## 03 · Three-Phase Commit (3PC)

[Read the notes →](../03-three-phase-commit.md)

> [!question]- 🟢 Q7. What does 3PC add over 2PC?
> A pre-commit phase between voting and commit. Once a participant gets pre-commit, it knows everyone voted yes, so if the coordinator dies it can commit after a timeout instead of blocking.

> [!question]- 🟡 Q8. If 3PC is non-blocking, why isn't it used in practice?
> It assumes there are no network partitions and that timeouts reliably detect crashes. During a partition, one side can commit while the other aborts, breaking consistency. It's also slower, so real systems use 2PC with a consensus-replicated coordinator, or sagas.

---

## 04 · Saga Pattern

[Read the notes →](../04-saga-pattern.md)

> [!question]- 🟢 Q9. What is a saga and what is a compensating action?
> A saga is a sequence of local transactions, one per service, linked by events or an orchestrator. If a step fails, we run compensating actions for the completed steps in reverse — like refunding a payment. A compensation is a new action, not a true rollback.

> [!question]- 🟡 Q10. Choreography vs orchestration — when do you pick which?
> Choreography has services react to each other's events with no central controller — loose coupling, good for 2–4 simple steps. Orchestration has one orchestrator call each step and track state — easier to see, debug and add timeouts, better for long or branching flows. Tools like Temporal or Step Functions help with orchestration.

> [!question]- 🟡 Q11. Sagas don't have isolation. What problems does that cause and how do you handle it?
> Other requests can see half-finished state — e.g. a user briefly has Premium before a later step fails and it's removed. I use pending states as semantic locks (status PENDING until the saga finishes), order steps so user-visible ones come last, and use commutative updates or version checks.

> [!question]- 🔴 Q12. A user got charged twice during checkout after a network glitch. What went wrong in the saga?
> The charge step was retried — at-least-once delivery or a client retry — and it wasn't idempotent. Every step and compensation should take an idempotency key like the saga ID, so a repeat call returns the earlier result. Refund the duplicate and add the key check.

---

## 05 · Outbox Pattern

[Read the notes →](../05-outbox-pattern.md)

> [!question]- 🟢 Q13. What problem does the outbox pattern solve?
> The dual-write problem: saving to the database and publishing to a queue are separate, so a crash in between leaves them out of sync. With an outbox, the event is written to a table in the same transaction as the data, and a relay publishes it later.

> [!question]- 🟡 Q14. Polling vs CDC for the outbox relay — trade-offs?
> Polling just queries unpublished rows every few hundred ms — simple, but adds DB load and latency. CDC, like Debezium, reads the database's change log and streams to Kafka — near real time and low load, but more infrastructure to run. Start with polling, move to CDC at scale.

> [!question]- 🔴 Q15. Consumers are processing some events twice and sometimes out of order. How do you fix it?
> Duplicates are expected — the outbox is at-least-once — so add an inbox table of processed event IDs, written in the same transaction as the consumer's change. For ordering, publish with the entity ID as the partition key and in sequence order, and make sure only one relay handles each partition.
