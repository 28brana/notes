# Q&A · API Fundamentals

[← Back to API Fundamentals](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · What is an API

[Read the notes →](../01-what-is-an-api.md)

> [!question]- 🟢 Q1. What is an API, in simple words?
> An API is a contract between two programs: which requests you can send, what you need to include, and what you get back. It hides the implementation, so the server can change its database or language without breaking clients. Like a restaurant menu — you order from it without entering the kitchen.

> [!question]- 🟡 Q2. What makes a good API?
> It's consistent in naming and errors, versioned so old clients don't break, and paginated for lists. Writes are idempotent so retries are safe, and it's protected with authentication and rate limits. It's also documented, for example with an OpenAPI spec.

> [!question]- 🔴 Q3. In a system design interview, when and how do you define the API?
> Right after clarifying requirements and before the high-level design. I list the 3–5 core endpoints that map to the functional requirements — method, path, key params, response, and auth. For example `POST /v1/posts` and `GET /v1/feed?cursor=&limit=`. The user ID comes from the auth token, not the request body.

---

## 02 · API Styles

[Read the notes →](../02-api-styles.md)

> [!question]- 🟢 Q4. Name the main API styles and what each is good for.
> REST for public and CRUD APIs, GraphQL when clients need flexible nested data, and gRPC for fast internal service-to-service calls. WebSockets handle two-way real-time like chat, and webhooks push events to another system's URL.

> [!question]- 🟡 Q5. Compare REST, GraphQL and gRPC.
> REST uses many resource URLs with JSON and is easy to cache with HTTP. GraphQL has one endpoint where the client picks fields, which fixes over- and under-fetching but makes caching harder. gRPC uses Protobuf over HTTP/2, so it's the fastest and strictly typed, but browsers need gRPC-Web to call it.

> [!question]- 🔴 Q6. You're designing a social app with web, mobile, 30 microservices and chat. Which API styles do you use where?
> REST for the public and partner API, maybe GraphQL as a gateway layer for the mobile app so it gets exactly the fields it needs in one call. gRPC between internal microservices for speed and typed contracts. WebSockets for chat, and webhooks if partners need event notifications.

---

## 03 · Idempotency

[Read the notes →](../03-idempotency.md)

> [!question]- 🟢 Q7. What does idempotent mean, and which HTTP methods are idempotent?
> An operation is idempotent if doing it once or many times leaves the same server state. GET, PUT and DELETE are idempotent; POST is not, and PATCH depends on the operation. A second DELETE returning 404 is still idempotent because the state didn't change.

> [!question]- 🟡 Q8. How does an Idempotency-Key work?
> The client generates a UUID once per user action and sends it in a header on every retry. The server stores the key with the request hash and the response. If the key shows up again, it returns the saved response instead of running the action again. Keys are kept for around 24 hours.

> [!question]- 🔴 Q9. Two identical payment requests with the same key arrive at the same time on two servers. How do you avoid a double charge?
> The key reservation must be atomic — insert the key with a unique constraint in the DB, or use Redis `SET NX`. Only one request wins; the other gets a 409 or waits for the saved result. Ideally the key and the business change are saved in the same transaction, and an in-progress lock has a timeout in case the winner crashes.

> [!question]- 🔴 Q10. Users report double charges even though you added idempotency keys. What could be wrong?
> The client might be generating a new key on every retry instead of once per action. The server might check-then-insert without a unique constraint, so concurrent requests race. Or the TTL is too short and late retries look new, or the key is stored separately from the payment and a crash left them out of sync.

---

## 04 · Data Formats

[Read the notes →](../04-data-formats.md)

> [!question]- 🟢 Q11. JSON vs Protobuf — when would you use each?
> JSON for public and browser-facing APIs, because it's readable, universal, and easy to debug. Protobuf for high-volume internal traffic, because it's binary, several times smaller, faster to parse, and has a strict schema.

> [!question]- 🟡 Q12. How do you change a Protobuf or Avro schema without breaking running services?
> Only add new optional fields, and never reuse or change existing field numbers or types. Old readers ignore unknown fields and new readers use defaults for missing ones. That lets old and new versions talk during a rolling deploy.

---

## 05 · REST

[Read the notes →](../05-rest.md)

> [!question]- 🟢 Q13. What are the core principles of REST?
> Everything is a resource with a URL, and HTTP methods are the actions. Requests are stateless, so each one carries everything needed, like the auth token. It uses standard status codes, and GET responses can be cached.

> [!question]- 🟡 Q14. Offset vs cursor pagination — which would you use for a news feed and why?
> Cursor. Offset makes the database scan and skip rows, so deep pages get slow, and new posts arriving shift the pages causing duplicates or skipped items. A cursor says "give me items after this one", uses an index, and stays stable as new data comes in.

> [!question]- 🟡 Q15. How do you version a REST API?
> Most commonly in the URL, like `/v1/posts`, because it's simple and visible; headers are another option. Adding fields isn't breaking, so I only create a new version for breaking changes like removing or renaming fields, and keep the old one running for a while.

> [!question]- 🔴 Q16. The mobile feed screen makes 21 REST calls and feels slow. What do you do?
> That's under-fetching — one call for posts and one per author. I'd add a feed endpoint that returns posts with embedded author info, or a BFF built for the mobile screen. If many screens have this problem, GraphQL is a good fit. I'd also check payloads for over-fetching and use sparse fields.

---

## 06 · GraphQL

[Read the notes →](../06-graphql.md)

> [!question]- 🟢 Q17. What problem does GraphQL solve?
> Over-fetching and under-fetching. The client sends one query to one endpoint, listing exactly the fields and related objects it needs, and gets all of it in one round trip.

> [!question]- 🟡 Q18. What is the N+1 problem in GraphQL and how do you fix it?
> Fetching 20 posts, then the author resolver runs once per post — 20 extra queries. The fix is DataLoader, which collects all the author IDs requested in one tick and loads them in a single batch query.

> [!question]- 🔴 Q19. Your public GraphQL API is being hit with deeply nested queries that overload the database. What do you do?
> Add a query depth limit and a cost analysis that rejects expensive queries. Rate limit by query cost rather than request count. For our own apps, use persisted queries so only pre-approved queries are allowed.

---

## 07 · gRPC

[Read the notes →](../07-grpc.md)

> [!question]- 🟢 Q20. What is gRPC and why is it fast?
> gRPC lets one service call another like a local function, using code generated from a proto file. It's fast because messages are binary Protobuf and calls are multiplexed over one HTTP/2 connection.

> [!question]- 🟡 Q21. Why don't we usually expose gRPC directly to browsers?
> Browsers don't give JavaScript the low-level HTTP/2 control gRPC needs, so you need gRPC-Web plus a proxy. Binary payloads are also harder to debug. So we keep REST or GraphQL at the edge and use gRPC internally.

> [!question]- 🔴 Q22. After moving to gRPC, one backend server gets most of the traffic while others sit idle. Why?
> gRPC uses long-lived HTTP/2 connections, and an L4 load balancer balances connections, not requests. So each client stays stuck to one server. The fix is an L7 balancer that spreads individual requests, or client-side load balancing across all backends.

---

## 08 · Rate Limiting

[Read the notes →](../08-rate-limiting.md)

> [!question]- 🟢 Q23. What is rate limiting and why do we need it?
> It caps how many requests a client can make in a time window, like 100 per minute per user. It protects against abuse, brute-force logins and buggy clients, so one bad actor can't slow down everyone. Over-limit requests get 429 with a Retry-After header.

> [!question]- 🟡 Q24. Explain token bucket vs fixed window.
> Token bucket refills tokens at a fixed rate up to a max, and each request uses one — it allows short bursts while keeping the average. Fixed window counts requests per clock minute; it's simple but allows twice the limit around window edges, like 100 at 12:05:59 and 100 at 12:06:00.

> [!question]- 🟡 Q25. How does the sliding window counter work?
> It keeps counters for the current and previous window and estimates the last 60 seconds as current count plus previous count times the fraction of the previous window still in range. It's almost as accurate as a full log but only needs two counters per client.

> [!question]- 🔴 Q26. You have 10 API gateway nodes, each with an in-memory limit of 100/min, but users are getting 1,000/min. Fix it.
> Each node counts separately, so the real limit is 10×. Move the counters to a shared store like Redis and do the check-and-update atomically with INCR or a Lua script. Decide what happens if Redis is down — usually fail open — and for huge scale, use local counters synced to Redis periodically.

---

## 09 · Authentication vs Authorization

[Read the notes →](../09-authentication-vs-authorization.md)

> [!question]- 🟢 Q27. What's the difference between authentication and authorization?
> Authentication proves who you are — login, then verifying a token or session. Authorization decides what you're allowed to do — roles and ownership checks. A failed authentication is 401; authenticated but not allowed is 403.

> [!question]- 🟡 Q28. In a microservices system, where do you do authentication and where authorization?
> Authenticate once at the API gateway by verifying the token, then pass the user ID to services. Coarse checks like "is admin" can be at the gateway, but fine-grained authorization like "does this user own post 7" belongs in the service that owns the data.

> [!question]- 🔴 Q29. A user changes `/orders/101` to `/orders/102` in the URL and sees someone else's order. What went wrong?
> That's IDOR — the API checked that the user is logged in but not that they own the resource. The fix is an ownership check on every request, using the user ID from the verified token, never from the request. Non-guessable IDs help but are not a fix by themselves.

---

## 10 · Session vs Token Auth

[Read the notes →](../10-session-vs-token-auth.md)

> [!question]- 🟢 Q30. What's the difference between session-based and token-based auth?
> With sessions, the server stores the login state, for example in Redis, and the client holds only a session ID. With tokens, the client holds a signed token with the user info, and any server verifies it without a lookup. Stateful versus stateless.

> [!question]- 🟡 Q31. How do you scale session-based auth across many servers?
> Move sessions out of server memory into a shared store like Redis, so any server can look up any session. Sticky sessions also work but cause uneven load, and losing a server logs its users out.

> [!question]- 🟡 Q32. Why use both access and refresh tokens?
> The access token is short-lived, around 15 minutes, and verified without a lookup on every request. The refresh token is long-lived but stored server-side, so it can be revoked. You get stateless speed on normal calls and a way to log users out within minutes.

> [!question]- 🔴 Q33. After adding a second server, users randomly get logged out. Why, and how do you fix it?
> Sessions are stored in each server's memory, so when the load balancer sends a user to the other server, their session isn't there. Move sessions to a shared store like Redis, or switch to stateless tokens. Sticky sessions are a quick but weaker fix.

---

## 11 · JWT

[Read the notes →](../11-jwt.md)

> [!question]- 🟢 Q34. What is a JWT and what are its three parts?
> A JWT is a signed, self-contained token. The header says the algorithm, the payload holds claims like user ID and expiry, and the signature proves nobody changed them. It's base64-encoded, not encrypted, so never put secrets in it.

> [!question]- 🟡 Q35. Why use RS256 instead of HS256 in microservices?
> With HS256, every service that verifies also has the secret that can create tokens. With RS256, only the auth service holds the private key, and other services verify using the public key, usually fetched from a JWKS endpoint, which also makes key rotation easier.

> [!question]- 🔴 Q36. A user's phone is stolen. How do you invalidate their JWT immediately?
> A JWT stays valid until expiry, so first, access tokens should be short-lived and the refresh token is revoked server-side. For immediate cut-off, add the token's `jti` to a denylist in Redis until it expires, or bump a per-user token version that services check. Both add a small lookup — that's the trade-off.

> [!question]- 🔴 Q37. An admin removed a user's moderator role, but the user can still delete posts for a while. Why?
> The role is baked into the JWT, so the old token still says moderator until it expires. Fixes: keep access tokens short, force a token refresh on role change, or check sensitive permissions against the database instead of trusting claims.

---

## 12 · OAuth 2.0

[Read the notes →](../12-oauth2.md)

> [!question]- 🟢 Q38. What is OAuth 2.0 used for?
> Delegated authorization — letting an app access your data on another service with limited scopes, without sharing your password. For example, letting our app read your Google contacts, which you can revoke any time.

> [!question]- 🟡 Q39. Explain the Authorization Code flow with PKCE.
> The app redirects the user to the provider with the requested scopes and a hashed random secret. The user logs in and consents, and the provider redirects back with a short-lived code. The app exchanges the code plus the original secret for an access token in a back-channel call, so a stolen code is useless.

> [!question]- 🟡 Q40. What's the difference between OAuth 2.0 and OpenID Connect?
> OAuth gives an access token to call APIs — it's authorization. OIDC sits on top and adds an ID token, a JWT that says who the user is — it's authentication. "Sign in with Google" is OIDC.

---

## 13 · SSO

[Read the notes →](../13-sso.md)

> [!question]- 🟢 Q41. How does SSO let a user log in only once?
> A central identity provider handles login and keeps its own session cookie. When a second app redirects the user there, the IdP sees the existing session and immediately returns a signed token without asking for a password. Each app then creates its own local session.

> [!question]- 🔴 Q42. What are the risks of SSO and how do you handle them?
> The IdP becomes a single point of failure, so it must be highly available. One stolen login opens every app, so strong MFA is required. And single logout is hard because each app keeps a local session — use short app sessions or back-channel logout.
