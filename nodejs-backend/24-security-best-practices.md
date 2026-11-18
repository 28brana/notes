# 24 · Security Best Practices

> [!info] Topic info
> **Subject:** Node.js Backend · **Part:** Building Production Backends · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Node.js Backend](README.md) · [Interview questions →](questions/building-backends-qa.md)

> [!abstract] In one line
> Secure a Node API in layers: **helmet** headers, strict **CORS**, **rate limiting**, **input validation**, **parameterized queries**, proper **auth/authz**, safe **secrets**, and **dependency audits** — covering the **OWASP Top 10**.

---

## 1. The problem

Your API works fine until someone sends `' OR 1=1 --` in the login form, scrapes all users by changing `/users/1` to `/users/2`, and brute-forces `/login` 10,000 times a minute.

Most breaches are boring mistakes, not genius hacks. Security is a **checklist**, applied in layers.

---

## 2. The Express security baseline

```js
app.use(helmet());                                     // security headers
app.use(cors({ origin: ['https://app.example.com'], credentials: true }));
app.use(express.json({ limit: '100kb' }));             // stop huge bodies
app.use('/auth', rateLimit({ windowMs: 60_000, max: 5 }));   // express-rate-limit
app.disable('x-powered-by');                           // helmet does this too
```

- **helmet** — sets headers like `Content-Security-Policy`, `Strict-Transport-Security` (HTTPS only), `X-Content-Type-Options`.
- **CORS** — a **browser** rule that controls which origins can call your API from JS. Allowlist origins; never `origin: '*'` with credentials. It does **not** stop curl or servers.
- **Rate limiting** — per IP/user, strict on login/OTP; store counters in Redis when you have many instances. See [rate limiting](../system-design/api-fundamentals/08-rate-limiting.md).
- **Body size limits** — prevents memory exhaustion.

---

## 3. OWASP top issues — the Node version

| Risk | Example | Fix in Node |
|---|---|---|
| **Injection (SQL / NoSQL)** | `` `SELECT * FROM users WHERE id = ${id}` `` or `{ "password": { "$ne": null } }` | Parameterized queries / ORM; validate types; `express-mongo-sanitize` |
| **Broken access control** | `/orders/43` returns someone else's order (IDOR) | Check ownership in every service |
| **Broken authentication** | No rate limit, weak hashing | bcrypt, rate limits, short JWTs ([auth](16-authentication.md)) |
| **XSS** | Stored `<script>` rendered by the frontend | Escape output, CSP via helmet, httpOnly cookies |
| **Security misconfiguration** | Stack traces in responses, debug mode on | `NODE_ENV=production`, generic 500 messages |
| **Vulnerable dependencies** | Old `lodash` with a known CVE | `npm audit`, Dependabot / Snyk, lockfile |
| **SSRF** | `?url=http://169.254.169.254/` hits cloud metadata | Allowlist outbound hosts |

```js
// ❌ injectable
db.query(`SELECT * FROM users WHERE email = '${email}'`);
// ✅ parameterized — driver sends data separately from SQL
db.query('SELECT * FROM users WHERE email = $1', [email]);
```

---

## 4. Node-specific traps

- **ReDoS** — a badly written regex on user input can take seconds and **block the event loop** for everyone. Limit input length, use safe regex.
- **Prototype pollution** — merging user JSON with `__proto__` keys can change every object. Validate input with a schema and avoid unsafe deep-merge.
- **`eval` / `child_process.exec` with user input** — remote code execution. Use `execFile` with argument arrays.
- **Supply chain** — typosquatted packages, malicious `postinstall` scripts. Pin with lockfile, use `npm ci`, review new deps.
- Run as a **non-root** user in Docker.

> [!tip] 🎤 Say it like this
> "I layer it: helmet for headers, a strict CORS allowlist, body size limits and rate limiting at the edge; zod validation on every input; parameterized queries so injection isn't possible; auth plus ownership checks for access control. Secrets live in a secrets manager, and npm audit plus Dependabot run in CI. Node-specific, I watch for ReDoS and prototype pollution since they hit the single event loop."

> [!failure] Interview traps
> - Thinking CORS protects the API from attackers — it only restricts browsers.
> - Thinking an ORM makes injection impossible — raw queries and Mongo operators still can.
> - Returning stack traces or DB errors to clients.
> - Checking authentication but not authorization on every resource.

---

## Summary

- Headers (**helmet**), **CORS** allowlist, **rate limits**, body limits.
- **Validate** input, **parameterize** queries.
- **AuthN + AuthZ** (ownership) on every route.
- **Audit dependencies**, secrets out of code, non-root containers.

> [!success] Key takeaway
> No single middleware makes you secure — defense in depth, with input validation and access checks doing most of the work.
