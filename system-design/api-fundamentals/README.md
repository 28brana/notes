# System Design · API Fundamentals

[← Back to System Design](../README.md) · [Interview Q&A →](questions/api-fundamentals-qa.md) · [Practice questions 📝](questions/practice-questions.md)

APIs are the front door of every system you design. Interviewers expect you to define clean endpoints, pick the right style (REST / GraphQL / gRPC), make writes safe to retry, protect the API with rate limits, and secure it with the right auth.

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [What is an API](01-what-is-an-api.md) | A **contract** that lets one program use another without knowing its insides | 🔥 High |
| 02 | [API Styles](02-api-styles.md) | REST, GraphQL, gRPC, WebSockets, webhooks — which one fits which job | ⚡ Medium |
| 03 | [Idempotency](03-idempotency.md) | Same request many times = same result; **Idempotency-Key** makes POST safe to retry | 🔥 High |
| 04 | [Data Formats](04-data-formats.md) | JSON vs XML vs Protobuf — readability vs size and speed | ⚡ Medium |
| 05 | [REST](05-rest.md) | Resources + HTTP methods + status codes; pagination, versioning, filtering | 🔥 High |
| 06 | [GraphQL](06-graphql.md) | One endpoint, client asks for **exactly** the fields it needs | ⚡ Medium |
| 07 | [gRPC](07-grpc.md) | Fast, typed service-to-service calls with Protobuf over HTTP/2 | ⚡ Medium |
| 08 | [Rate Limiting](08-rate-limiting.md) | Cap requests per client — token bucket, sliding window, Redis, 429 | 🔥 High |
| 09 | [Authentication vs Authorization](09-authentication-vs-authorization.md) | **Who are you** (401) vs **what can you do** (403) | 🔥 High |
| 10 | [Session vs Token Auth](10-session-vs-token-auth.md) | Server remembers you (session) vs you carry proof (token) | 🔥 High |
| 11 | [JWT](11-jwt.md) | Signed, self-contained token — no DB lookup, but hard to revoke | 🔥 High |
| 12 | [OAuth 2.0](12-oauth2.md) | Let an app act on your behalf **without your password** | ⚡ Medium |
| 13 | [SSO](13-sso.md) | Log in once, use many apps — via an identity provider | ⚡ Medium |
