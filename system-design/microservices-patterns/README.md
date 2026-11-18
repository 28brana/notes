# System Design · Microservices Patterns

[← Back to System Design](../README.md) · [Interview Q&A →](questions/microservices-patterns-qa.md) · [Practice questions 📝](questions/practice-questions.md)

The patterns that make microservices actually work: finding each other, a front door for clients, and stopping one failing service from taking down the rest. Interviewers expect these the moment you draw more than one service.

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [Service Discovery](01-service-discovery.md) | How services find each other's live addresses via a registry | 🔥 High |
| 02 | [API Gateway](02-api-gateway.md) | Single front door: routing, auth, rate limiting for all clients | 🔥 High |
| 03 | [Backend for Frontend](03-backend-for-frontend.md) | One thin backend per client type — web, mobile, TV | ⚡ Medium |
| 04 | [Sidecar](04-sidecar.md) | Helper container beside each service for shared plumbing | 💤 Low |
| 05 | [Circuit Breaker](05-circuit-breaker.md) | Stop calling a failing service, fail fast, serve a fallback | 🔥 High |
| 06 | [Bulkhead](06-bulkhead.md) | Separate resource pools so one failure can't sink everything | 💤 Low |
| 07 | [Strangler Fig](07-strangler-fig.md) | Replace a monolith piece by piece behind a router | 💤 Low |
| 08 | [Service Mesh](08-service-mesh.md) | Sidecar proxies give mTLS, retries, and tracing to every call | 💤 Low |
