# System Design · Architectural Patterns

[← Back to System Design](../README.md) · [Interview Q&A →](questions/architectural-patterns-qa.md) · [Practice questions 📝](questions/practice-questions.md)

How to shape a whole system: one big app or many small ones, request/response or events, servers or no servers.
Interviewers love these because every answer is a **trade-off** — "it depends" is fine, as long as you say *on what*.

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [Client-Server](01-client-server.md) | Clients ask, a central server answers — the base of almost every web app | 🔥 High |
| 02 | [Monolith](02-monolith.md) | One codebase, one deploy — simple to start, painful at huge team size | ⚡ Medium |
| 03 | [Microservices](03-microservices.md) | Many small services, each owns its data and deploys alone | 🔥 High |
| 04 | [Serverless](04-serverless.md) | Run functions on demand, pay per call, no servers to manage | ⚡ Medium |
| 05 | [Event-Driven Architecture](05-event-driven.md) | Services react to events instead of calling each other directly | 🔥 High |
| 06 | [Peer-to-Peer](06-peer-to-peer.md) | Every node is both client and server — no central boss | 💤 Low |
| 07 | [Hexagonal Architecture](07-hexagonal.md) | Keep business logic in the middle, plug tech in through ports and adapters | 💤 Low |
| 08 | [CQRS](08-cqrs.md) | Separate models for writes and reads | ⚡ Medium |
| 09 | [Event Sourcing](09-event-sourcing.md) | Store every change as an event, rebuild state by replaying them | ⚡ Medium |
