# System Design · Distributed Transactions

[← Back to System Design](../README.md) · [Interview Q&A →](questions/distributed-transactions-qa.md) · [Practice questions 📝](questions/practice-questions.md)

How to keep one business action all-or-nothing when it spans several services or databases. Interviewers love this in any microservices design — "what if payment succeeds but the next step fails?"

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [The Distributed Transactions Problem](01-distributed-transactions-problem.md) | Why one ACID transaction stops working once each service owns its DB | ⚡ Medium |
| 02 | [Two-Phase Commit (2PC)](02-two-phase-commit.md) | Coordinator runs prepare + commit votes; strong but blocking | ⚡ Medium |
| 03 | [Three-Phase Commit (3PC)](03-three-phase-commit.md) | Adds pre-commit to avoid blocking; unsafe on partitions, rarely used | 💤 Low |
| 04 | [Saga Pattern](04-saga-pattern.md) | Chain of local transactions with compensating actions; choreography vs orchestration | 🔥 High |
| 05 | [Outbox Pattern](05-outbox-pattern.md) | Save data + event in one transaction; polling vs CDC, ordering, inbox | ⚡ Medium |
