# Q&A · Microservices Patterns

[← Back to Microservices Patterns](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · Service Discovery

[Read the notes →](../01-service-discovery.md)

> [!question]- 🟢 Q1. What is service discovery and why do microservices need it?
> It's how a service finds the current address of another service. In microservices, instances scale up and down and get new IPs on every deploy, so hard-coded addresses break. A service registry keeps a live list of healthy instances that callers look up.

> [!question]- 🟡 Q2. Client-side vs server-side discovery — what's the difference?
> In client-side discovery, the caller queries the registry and picks an instance itself — no extra hop, but every client needs a discovery library. In server-side discovery, the caller hits a load balancer, which queries the registry and picks the instance — simple clients, one extra hop. Kubernetes Services and AWS ALB are server-side; Eureka with Ribbon is client-side.

> [!question]- 🟡 Q3. How does Kubernetes handle service discovery?
> Every Service gets a stable DNS name like `posts.default.svc.cluster.local` that points to a virtual IP. kube-proxy forwards traffic from that IP to healthy pods, using readiness checks. The app just calls `http://posts` and never sees pod IPs.

> [!question]- 🔴 Q4. After a deploy, some requests still go to instances that were shut down. Why, and how do you fix it?
> The callers probably have stale addresses — from DNS caching with long TTLs, or a registry that removes instances only after a slow heartbeat timeout. Fix it with graceful shutdown (deregister first, then drain in-flight requests, then stop), lower TTLs and cache times, and faster health checks. Retries on idempotent calls cover the small gap that remains.

---

## 02 · API Gateway

[Read the notes →](../02-api-gateway.md)

> [!question]- 🟢 Q5. What is an API gateway and what does it do?
> It's the single entry point for all clients into a microservices system. It routes each request to the right service and handles shared concerns — authentication, rate limiting, SSL termination, logging, and sometimes aggregating several service calls into one response.

> [!question]- 🟡 Q6. API gateway vs load balancer vs reverse proxy?
> A reverse proxy is the general idea — a server in front of your servers that forwards requests. A load balancer is a reverse proxy that spreads traffic across copies of the same service. An API gateway routes between different services and adds API concerns like auth, rate limits, and versioning. In practice you often have an LB in front of a gateway cluster.

> [!question]- 🟡 Q7. What are the downsides of an API gateway?
> It adds an extra network hop and is a potential single point of failure, so it must be clustered. It can become a bottleneck if one team owns all its config. And if business logic creeps in, it turns into a new monolith — keep it thin.

> [!question]- 🔴 Q8. Your gateway does all auth checks. A developer calls the Posts service directly from inside the network and edits another user's post. What went wrong?
> We trusted the network — services assumed the gateway already checked everything. The gateway should handle authentication, but each service must still enforce authorization ("can this user edit this post?"). Internal traffic should also be authenticated, e.g. with mTLS via a service mesh, so not every internal caller is trusted.

---

## 03 · Backend for Frontend

[Read the notes →](../03-backend-for-frontend.md)

> [!question]- 🟢 Q9. What is the Backend for Frontend pattern?
> It means each client type — web, mobile, TV — gets its own thin backend that aggregates and shapes data for that client. Mobile can get small, trimmed responses while web gets rich ones. Each BFF is usually owned by the team that builds that frontend.

> [!question]- 🟡 Q10. When would you use a BFF instead of one API gateway — and when not?
> Use BFFs when clients need very different data shapes and separate teams own them, and the shared gateway is filling with client-specific `if` code. Skip it if you have one client, or all clients need similar data. GraphQL is a common alternative, since each client asks for exactly the fields it needs.

> [!question]- 🔴 Q11. Your mobile and web BFFs now contain the same "who can see this post" logic, and they've started to disagree. How do you fix it?
> That rule is business logic and doesn't belong in a BFF. Move it into the owning service (Posts or a Privacy service) so there is one source of truth, and let the BFFs only aggregate and format. Any shared formatting code can go in a library.

---

## 04 · Sidecar

[Read the notes →](../04-sidecar.md)

> [!question]- 🟢 Q12. What is the sidecar pattern?
> A sidecar is a helper container deployed next to each service instance, sharing its network and lifecycle. It handles plumbing like proxying, TLS, log shipping, or secret fetching, so app code stays focused on business logic. It works the same for any language — that's why service meshes are built on sidecars.

> [!question]- 🟡 Q13. Why use a sidecar instead of a shared library?
> A library has to be written and upgraded in every language you use, and changing it means redeploying every service. A sidecar is one implementation for all languages and can be updated on its own. The cost is extra CPU and memory per pod plus a small local hop.

---

## 05 · Circuit Breaker

[Read the notes →](../05-circuit-breaker.md)

> [!question]- 🟢 Q14. What is a circuit breaker and why do we need it?
> It wraps calls to a dependency and tracks failures. When failures cross a threshold, it opens and fails fast instead of calling the broken service. That stops threads piling up waiting on a slow service, stops cascading failures, and gives the sick service time to recover.

> [!question]- 🟡 Q15. Explain the three states of a circuit breaker.
> Closed is normal — calls go through and failures are counted. Open means the threshold was crossed — calls fail instantly and go to a fallback. After a cool-down, half-open lets a few test calls through: success closes the breaker, any failure opens it again.

> [!question]- 🟡 Q16. How do you choose the thresholds, and what counts as a failure?
> Use a failure rate (e.g. 50%) over a sliding window (last 100 calls or 10 s), with a minimum number of calls so low traffic doesn't trip it on noise. Count 5xx errors, timeouts, and very slow calls as failures — not 4xx, which are the caller's fault. Open duration is typically tens of seconds, with a handful of half-open trial calls.

> [!question]- 🔴 Q17. Recommendations is down and your feed page now takes 30 seconds to load, even with retries configured. What's happening and what do you do?
> Each request is probably waiting for a long timeout and then retrying several times, so threads pile up and latency explodes. Add a short timeout, wrap the call in a circuit breaker so it opens and fails fast after the failures, and serve a fallback like cached or trending posts. Make sure retries are inside the breaker and use backoff, so they stop once it opens.

---

## 06 · Bulkhead

[Read the notes →](../06-bulkhead.md)

> [!question]- 🟢 Q18. What is the bulkhead pattern?
> It isolates resources into separate compartments so one failure can't use them all — like watertight sections in a ship. For example, each downstream dependency gets its own small thread or connection pool, so if one hangs only its pool fills up and the rest of the service keeps working.

> [!question]- 🔴 Q19. One slow, non-critical service takes down your whole API even though you have a circuit breaker. Why?
> Before the breaker trips, it needs enough failures — and during that window, slow calls can grab every thread in a shared pool. A bulkhead caps how many concurrent calls go to that dependency, so the rest of the API keeps its threads. Use bulkhead, timeout, and circuit breaker together.

---

## 07 · Strangler Fig

[Read the notes →](../07-strangler-fig.md)

> [!question]- 🟢 Q20. What is the strangler fig pattern?
> It's a way to migrate a legacy system gradually instead of a big-bang rewrite. You put a router or gateway in front, move one feature at a time to a new service, shift traffic over, and delete the old code — until the old system can be switched off. Rollback is just changing the route back.

> [!question]- 🔴 Q21. You're asked to move a 10-year-old monolith to microservices. How do you start?
> I wouldn't rewrite it all at once. I'd put a gateway in front, pick a small, well-bounded feature with few dependencies — say Notifications — extract it, and move traffic gradually with a canary. The hardest part is data, so I'd give the new service its own data and keep it in sync with events or change data capture during the transition.

---

## 08 · Service Mesh

[Read the notes →](../08-service-mesh.md)

> [!question]- 🟢 Q22. What is a service mesh?
> It's an infrastructure layer for service-to-service traffic. A sidecar proxy like Envoy sits next to each service (the data plane), and a control plane like Istio configures them all. It gives every call mTLS, retries, timeouts, traffic splitting, and metrics without changing app code.

> [!question]- 🔴 Q23. Your team of 6 engineers with 8 services wants to adopt Istio. What's your advice?
> Probably not yet. A mesh adds real complexity — new components to run and debug, one proxy per pod, and a few ms per hop. With 8 services, a good resilience library plus an API gateway and TLS is enough. I'd reconsider when we have dozens of services, many languages, or strict mTLS and zero-trust requirements.
