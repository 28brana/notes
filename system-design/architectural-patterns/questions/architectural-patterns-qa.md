# Q&A · Architectural Patterns

[← Back to Architectural Patterns](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · Client-Server

[Read the notes →](../01-client-server.md)

> [!question]- 🟢 Q1. What is client-server architecture?
> Clients like browsers and mobile apps send requests, and a central server owns the data and business rules and sends responses. The client initiates; the server answers. It's the default model for almost every web app.

> [!question]- 🟢 Q2. What is a 3-tier architecture and why is it the standard?
> Client, app server, and database, each on its own layer. The client never touches the DB directly — the app server enforces rules and security in between. Each tier can be scaled and secured on its own.

> [!question]- 🟡 Q3. Why should the server never trust the client?
> Anyone can modify a client — edit JavaScript, replay requests, or call the API with curl. So prices, permissions, and input must be validated on the server. Client-side checks are only for user experience.

> [!question]- 🔴 Q4. The single app server is maxed out at 100% CPU during peaks. What do you do?
> First make the server stateless — sessions in Redis or tokens — then run several copies behind a load balancer and auto-scale. Add caching and a CDN to cut load, and read replicas if the DB is the next bottleneck. This also removes the server as a single point of failure.

---

## 02 · Monolith

[Read the notes →](../02-monolith.md)

> [!question]- 🟢 Q5. What is a monolith, and what are its main advantages?
> One codebase, built and deployed as one unit, usually with one database. It's simple to build, test, and debug, calls between modules are fast function calls, and you get normal ACID transactions across everything.

> [!question]- 🟡 Q6. What is a modular monolith and why do people recommend starting with one?
> It's a single deployable with strict internal module boundaries — each module owns its tables and exposes a clear interface. You keep monolith simplicity now, and if one module later needs its own scaling or team, it's easy to extract as a service.

> [!question]- 🔴 Q7. Your monolith takes 45 minutes to deploy and teams keep blocking each other. Do you go straight to microservices?
> Not all at once. First fix boundaries inside the code — a modular monolith — and speed up the build and tests. Then extract the module causing the most pain using the strangler fig pattern, one service at a time. A big-bang rewrite usually fails.

---

## 03 · Microservices

[Read the notes →](../03-microservices.md)

> [!question]- 🟢 Q8. What are microservices?
> An app split into small services, each owning one business capability and its own database, deployed independently and owned by one team. They talk over the network via APIs or events.

> [!question]- 🟡 Q9. Monolith vs microservices — how do you choose?
> It's mostly about team size and scaling needs. Small team or new product: modular monolith, because it's simpler and faster. Many teams, parts with very different load or reliability needs: microservices, accepting network failures, eventual consistency, and heavy ops cost.

> [!question]- 🟡 Q10. Why should each microservice have its own database?
> If services share tables, a schema change in one breaks the others, and they can't deploy or scale independently. Owning its data makes the service truly independent. Others get the data through its API or its events.

> [!question]- 🔴 Q11. After splitting into microservices, the app got slower and outages got more frequent. Why?
> Likely a distributed monolith: long chains of sync calls, each adding latency, and each one a failure point — 5 services at 99.9% each give about 99.5% overall. Fix by cutting sync chains, using events for side effects, adding timeouts, retries, and circuit breakers, and merging services that always change together.

---

## 04 · Serverless

[Read the notes →](../04-serverless.md)

> [!question]- 🟢 Q12. What is serverless, and when is it a good fit?
> You deploy functions, the cloud runs them per event, scales automatically including to zero, and you pay per invocation. Great for spiky or low traffic and short event-driven jobs — image resizing, webhooks, cron jobs.

> [!question]- 🟡 Q13. What is a cold start and how do you reduce it?
> The first request to a new function instance waits for the runtime to boot, adding from ~100 ms to seconds. Reduce it with provisioned or pre-warmed instances, smaller packages, lighter runtimes, and keeping heavy init out of the hot path.

> [!question]- 🔴 Q14. During a traffic spike your Lambda functions start failing with "too many connections" from Postgres. What happened?
> Each function instance opened its own DB connection, and thousands of instances exceeded Postgres's connection limit. Fix with a connection pooler or proxy like RDS Proxy or PgBouncer, cap function concurrency, or use a DB with an HTTP-based or serverless-friendly API.

---

## 05 · Event-Driven Architecture

[Read the notes →](../05-event-driven.md)

> [!question]- 🟢 Q15. What is event-driven architecture?
> Services publish events about things that happened to a broker, and other services subscribe and react. The producer doesn't know or wait for the consumers. This decouples services and keeps the producer fast.

> [!question]- 🟡 Q16. What's the difference between an event and a command?
> An event is a past-tense fact, like PostCreated — any number of services can react, and it can't be rejected. A command is a request to one specific handler, like SendEmail — it expects to be handled and can fail.

> [!question]- 🟡 Q17. Choreography vs orchestration — when do you use each?
> Choreography: services react to each other's events with no central boss — very decoupled, good for simple flows. Orchestration: one orchestrator tells each step what to do — easier to see, change, and handle rollbacks, so better for complex multi-step flows like order checkout sagas.

> [!question]- 🔴 Q18. Users sometimes get the same notification twice, and some post-created events seem to vanish. What's wrong?
> Duplicates mean the broker is at-least-once and the consumer isn't idempotent — dedupe using an event ID. Vanishing events usually mean a dual write: the DB commit succeeded but publishing failed. Fix with the outbox pattern, and send events that keep failing to a dead letter queue instead of dropping them.

---

## 06 · Peer-to-Peer

[Read the notes →](../06-peer-to-peer.md)

> [!question]- 🟢 Q19. What is peer-to-peer and how is it different from client-server?
> In P2P every node is both client and server, sharing data directly. In client-server, more users means more load; in P2P, more users means more capacity. Examples are BitTorrent and blockchains.

> [!question]- 🟡 Q20. What are the main challenges of P2P systems?
> Peers are untrusted, so data must be verified with hashes. Peers join and leave constantly, NAT and firewalls block direct connections, and finding who has what needs a tracker or a DHT. There's also no central control for updates or moderation.

---

## 07 · Hexagonal Architecture

[Read the notes →](../07-hexagonal.md)

> [!question]- 🟢 Q21. What is hexagonal architecture?
> Business logic sits in the core with no framework or DB code. The core defines ports — interfaces — and adapters plug real tech into them, like a REST controller or a Postgres repository. Dependencies always point inward.

> [!question]- 🟡 Q22. What benefit does hexagonal architecture give you in practice?
> You can unit test business rules with in-memory fakes instead of a real DB, swap tech like MySQL to Postgres without touching the rules, and add new entry points like a queue consumer that reuse the same logic. It's overkill for simple CRUD services.

---

## 08 · CQRS

[Read the notes →](../08-cqrs.md)

> [!question]- 🟢 Q23. What is CQRS?
> Command Query Responsibility Segregation — separate models for writes and reads. Writes go to a normalized store that enforces rules; reads come from denormalized views shaped for each screen, synced by events.

> [!question]- 🟡 Q24. When is CQRS worth it, and when is it overkill?
> Worth it when reads and writes differ a lot — very read-heavy, many different read views, complex write rules. Overkill for simple CRUD with balanced traffic, where it only adds sync code and eventual consistency.

> [!question]- 🔴 Q25. With CQRS in place, users complain their new post doesn't show up right after they publish it. How do you fix it?
> The read model is eventually consistent and hasn't caught up yet. Options: show the post optimistically in the UI, read the user's own recent writes from the write side, or have the client wait for the projection to reach the write's version. Also monitor projector lag.

---

## 09 · Event Sourcing

[Read the notes →](../09-event-sourcing.md)

> [!question]- 🟢 Q26. What is event sourcing?
> Instead of storing current state, you store every change as an immutable event in an append-only log. Current state is rebuilt by replaying events. Mistakes are fixed by adding a correcting event, never by editing old ones.

> [!question]- 🟡 Q27. Event sourcing vs event-driven architecture — what's the difference?
> Event sourcing is how one service stores its data — as a log of events. Event-driven architecture is how services communicate — by publishing events to each other. You can use either without the other.

> [!question]- 🔴 Q28. Your event-sourced account service is getting slower every month. Why, and what do you do?
> Each load replays the full event history, which keeps growing. Add snapshots — save state every N events and replay only events after the latest snapshot. For queries, serve them from projections instead of replaying.
