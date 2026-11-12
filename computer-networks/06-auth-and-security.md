# 06 · Authentication and Network Security

> **Subject:** Computer Networks (for full stack developers)
> **Status:** ✅ done
> **Last reviewed:** 2026-11-13

[← Back to Computer Networks](README.md) · [Questions for this topic →](questions/06-auth-and-security-qa.md)

## In one line

Authentication proves **who you are**, authorization decides **what you can do**, and network security makes sure nobody can **read, change or fake** the traffic in between.

---

## 1. Authentication vs authorization

> 🎤 **Say it like this:** "Authentication is verifying identity — login with password, OTP, Google. Authorization happens after that and checks permissions — can this user delete this order? Authentication failure is 401, authorization failure is 403."

| | Authentication (AuthN) | Authorization (AuthZ) |
|---|---|---|
| Question | Who are you? | What are you allowed to do? |
| Happens | First | After authentication |
| Example | Login, JWT verification | Role check: `admin` only |
| Failure code | **401 Unauthorized** | **403 Forbidden** |

Common authorization styles: **RBAC** (role-based: admin, editor) and **ABAC** / ownership checks (user can edit only *their own* post).

---

## 2. Session-based (cookie-based) authentication

> 🎤 **Say it like this:** "On login, the server creates a session in a store like Redis and sends back only a random session ID in an HttpOnly cookie. The browser sends that cookie automatically on every request, and the server looks up the session to know the user. It's stateful — the server remembers the user."

```
1. POST /login (email, password)
2. Server: verify → save session {id: "abc", userId: 42} in Redis
3. Response: Set-Cookie: sid=abc; HttpOnly; Secure; SameSite=Lax
4. Next requests: Cookie: sid=abc → server looks up Redis → user 42
5. Logout: delete session from Redis → instantly invalid
```

> 💡 "Cookie-based auth" just means the credential travels in a cookie. The cookie can hold a **session ID** (most common) or a **JWT**.

---

## 3. JWT (JSON Web Token)

> 🎤 **Say it like this:** "A JWT is a self-contained token with three base64url parts — header, payload and signature. The server signs it with a secret, so it can verify it without any DB lookup. It's signed, not encrypted — anyone can decode the payload, so never put secrets in it. Its weakness is revocation: once issued, it's valid until it expires."

```
eyJhbGciOiJIUzI1NiJ9 . eyJzdWIiOiI0MiIsInJvbGUiOiJhZG1pbiIsImV4cCI6MTcwMH0 . SflKxwRJSMeKKF2QT4fwpM
└────── header ─────┘   └──────────────────── payload ──────────────────────┘   └──── signature ────┘
 {"alg":"HS256"}          {"sub":"42","role":"admin","exp":1700...}            HMAC(header.payload, secret)
```

- **Header:** algorithm (`HS256` = shared secret, `RS256` = private/public key pair).
- **Payload (claims):** `sub` (user id), `exp` (expiry), `iat`, `role`… readable by anyone.
- **Signature:** proves the token wasn't changed. Change one character → verification fails.

**Rules:** always verify signature + `exp`, reject `alg: none`, keep tokens short-lived, no sensitive data in payload.

### The revocation problem

Server doesn't store JWTs, so it can't "delete" one. Logout / ban / password change won't kill an already-issued token. Fixes:
- Short expiry for access tokens (5–15 min) + refresh tokens.
- A **denylist** in Redis (by `jti`) — but then you're back to a lookup.
- A `tokenVersion` on the user; bump it to invalidate all old tokens.

---

## 4. Session vs JWT — which one?

| | Session (server-side) | JWT (stateless) |
|---|---|---|
| Where state lives | Server (Redis/DB) | Inside the token |
| Lookup per request | Yes (fast with Redis) | No, just verify signature |
| Logout / revoke | Instant (delete session) | Hard (wait for expiry) |
| Scaling | Needs shared session store | Easy, any server can verify |
| Size | Tiny cookie | Bigger (every request) |
| Best for | Classic web apps, same domain | Mobile apps, microservices, third-party APIs |

> 🎤 **Senior answer:** "For a normal web app on one domain I'd prefer sessions in an HttpOnly cookie — simple and revocable. JWTs make sense when many services need to verify identity without calling an auth service, or for mobile/API clients. Often the best is a mix: short-lived JWT access token + revocable refresh token stored server-side."

---

## 5. Access and refresh tokens

> 🎤 **Say it like this:** "The access token is short-lived, like 15 minutes, and is sent with every API call. The refresh token is long-lived, like 7–30 days, and is used only to get a new access token. If an access token leaks, damage is limited. With refresh token rotation, every refresh returns a new refresh token and the old one is invalidated — if an old one is reused, we know it was stolen and kill the whole session."

```
Login ──▶ access (15m) + refresh (30d, HttpOnly cookie, stored/hashed in DB)

API call with access ──▶ 200 OK
API call with expired access ──▶ 401
      │
      ▼
POST /auth/refresh (sends refresh cookie)
      ──▶ new access + NEW refresh (old refresh marked used)

Old refresh used again? ──▶ reuse detected ──▶ revoke all tokens of that user/family
```

Frontend tip: on 401, call refresh **once**, queue other failing requests while refreshing, then retry them.

---

## 6. Where to store tokens in the browser

| Storage | Readable by JS? | Sent automatically? | Main risk |
|---|---|---|---|
| **localStorage / sessionStorage** | ✅ Yes | ❌ No (you add header) | **XSS** can steal it |
| **HttpOnly cookie** | ❌ No | ✅ Yes | **CSRF** (fix with SameSite + CSRF token) |
| **In-memory (JS variable)** | Only by your code | ❌ No | Lost on refresh (pair with refresh cookie) |

> 🎤 **Say it like this:** "I avoid localStorage for auth tokens because any XSS can read it. I prefer HttpOnly + Secure + SameSite cookies — JS can't read them, and SameSite plus a CSRF token handles CSRF. A common SPA pattern is access token in memory and refresh token in an HttpOnly cookie."

---

## 7. Cookie flags (Secure, HttpOnly, SameSite…)

```
Set-Cookie: sid=abc123; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=86400; Domain=example.com
```

| Flag | What it does |
|---|---|
| **HttpOnly** | JS (`document.cookie`) can't read it → protects from XSS theft |
| **Secure** | Sent only over HTTPS |
| **SameSite=Strict** | Never sent on cross-site requests (even clicking a link from another site) |
| **SameSite=Lax** | Sent on top-level navigation (GET link clicks), not on cross-site POST/fetch. **Browser default** |
| **SameSite=None** | Sent on all cross-site requests — **must** also be `Secure`. Needed for 3rd-party/embedded use |
| **Domain** | Which hosts get it. `Domain=example.com` → also sent to `api.example.com` |
| **Path** | Only sent for URLs under this path |
| **Max-Age / Expires** | Lifetime. Without it → session cookie (deleted when browser closes) |

---

## 8. CSRF (Cross-Site Request Forgery)

> 🎤 **Say it like this:** "CSRF tricks a logged-in user's browser into sending a request to your site from an evil site. Because the browser attaches cookies automatically, the server thinks it's the real user. Defenses are SameSite cookies, CSRF tokens, and checking Origin header. If you use Authorization header tokens instead of cookies, CSRF mostly isn't an issue."

```
User logged in to bank.com (cookie set)
     │
Visits evil.com which has:
<form action="https://bank.com/transfer" method="POST"> <input name="to" value="hacker">…</form>
<script>document.forms[0].submit()</script>
     │
Browser sends POST to bank.com WITH the bank cookie → money transferred 😱
```

**Defenses:**
1. **SameSite=Lax/Strict** cookies — cookie not sent on cross-site POST.
2. **CSRF token (synchronizer token)** — server puts a random token in the page/form; must come back in a header/body. Evil site can't read it.
3. **Double-submit cookie** — token in a cookie *and* in a header; server checks they match (attacker can't read your cookie to copy it).
4. Check **Origin / Referer** header.
5. Never change state with GET.

---

## 9. XSS (Cross-Site Scripting) and browser storage

> 🎤 **Say it like this:** "XSS is when an attacker gets their JavaScript to run on your site, usually through unescaped user input. That script runs with your page's privileges, so it can read localStorage, call your APIs, and change the page. That's why tokens in localStorage are risky. Defenses: escape output, sanitize HTML, use a Content-Security-Policy, and HttpOnly cookies."

| Type | How |
|---|---|
| **Stored** | Malicious script saved in DB (e.g. comment), shown to every visitor |
| **Reflected** | Script in URL param reflected back in the response |
| **DOM-based** | Frontend JS puts untrusted data into the DOM (`innerHTML = location.hash`) |

**Defenses:**
- Escape output — React/Vue escape by default; danger zones are `dangerouslySetInnerHTML`, `v-html`, `innerHTML`.
- Sanitize user HTML (DOMPurify).
- **CSP header:** `Content-Security-Policy: script-src 'self'` blocks inline/unknown scripts.
- HttpOnly cookies so stolen JS can't read session.

> 💡 Note: even with HttpOnly, XSS can still *make requests* as the user. HttpOnly limits damage; it doesn't fix XSS.

---

## 10. CORS is NOT a security/auth mechanism

> 🎤 **Say it like this:** "CORS is a browser rule that relaxes the same-origin policy — it decides whether browser JS on another origin may *read* the response. It doesn't protect your API: curl, Postman or a server can call it regardless. Real protection is authentication, authorization and CSRF defense."

Details on CORS headers & preflight → [03 · Browser networking](03-browser-networking.md)

---

## 11. OAuth 2.0, OpenID Connect and SSO

> 🎤 **Say it like this:** "OAuth 2.0 is for **authorization** — letting an app access something on your behalf without sharing your password, like 'allow this app to read your Google Calendar'. OpenID Connect sits on top of OAuth and adds **authentication** — it gives an ID token saying who the user is. 'Login with Google' is OIDC. SSO means one login works across many apps, usually using OIDC or SAML."

**Authorization Code flow with PKCE** (the recommended flow for web & mobile):

```
 User        Your App (client)                Auth Server (Google)          Your API
  │  click "Login with Google"                       │
  │──────────▶│ create code_verifier (random),        │
  │           │ code_challenge = SHA256(verifier)     │
  │◀── redirect to Google /authorize?client_id&redirect_uri&code_challenge&state
  │──── user logs in & consents ───────────────────▶│
  │◀── redirect back: /callback?code=XYZ&state ──────│
  │──────────▶│ POST /token (code + code_verifier) ─▶│ verifies challenge
  │           │◀── access_token (+ id_token, refresh_token)
  │           │──── call API with access_token ─────────────────────────────▶│
```

- **state** → protects against CSRF on the callback.
- **PKCE** → even if someone steals the `code`, they don't have the `code_verifier`.
- Never use the old **Implicit flow** (token in URL).

| Term | Meaning |
|---|---|
| Resource owner | The user |
| Client | Your app |
| Authorization server | Google / Auth0 / Keycloak / Cognito |
| Access token | Used to call APIs |
| ID token (OIDC) | JWT with user identity (`sub`, `email`) |

---

## 12. TLS, encryption basics, MITM and HSTS

> 🎤 **Say it like this:** "TLS encrypts traffic between client and server, so attackers on the network can't read or modify it, and the certificate proves the server is really who it claims. Without HTTPS, a man-in-the-middle — like on public Wi-Fi — can read passwords and cookies. HSTS tells the browser to always use HTTPS for my domain, so it can't be downgraded to HTTP."

- **Symmetric encryption** (AES): same key to encrypt & decrypt — fast, used for actual data.
- **Asymmetric encryption** (RSA/ECDHE): public/private key pair — used in the handshake to safely agree on the symmetric key.
- **MITM:** attacker sits between you and server, reads/changes traffic. TLS + certificate validation stops it.
- **HSTS header:** `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`

Handshake details → [02 · HTTP and HTTPS](02-http-and-https.md)

---

## 13. Rate limiting

> 🎤 **Say it like this:** "Rate limiting caps how many requests a client can make in a time window — to protect against abuse, brute-force login and accidental overload. I usually key it by user ID, API key or IP, store counters in Redis so it works across servers, and return 429 Too Many Requests with a Retry-After header."

| Algorithm | Idea | Note |
|---|---|---|
| **Fixed window** | Max 100 req per clock minute | Simple; burst at window edges (100 at 0:59 + 100 at 1:00) |
| **Sliding window** | Count over the *last* 60s | Smoother, a bit more memory |
| **Token bucket** | Bucket refills N tokens/sec, each request takes one | Allows short bursts, smooth average. Very common |
| **Leaky bucket** | Requests leave at a fixed rate | Smooths output |

```
HTTP/1.1 429 Too Many Requests
Retry-After: 30
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
```

Where: API Gateway / Nginx / Cloudflare (edge) and app-level (e.g. `express-rate-limit` + Redis). Login and OTP endpoints need stricter limits.

---

## 14. API keys vs tokens

| | API key | Token (JWT / OAuth) |
|---|---|---|
| Identifies | An app / project / server | A user (and their permissions) |
| Lifetime | Long-lived until rotated | Short-lived, expires |
| Typical use | Server-to-server, public APIs (Stripe, Maps) | User sessions, delegated access |
| Sent as | `x-api-key` header / `Authorization: Bearer` | `Authorization: Bearer <token>` |

API key rules: never in frontend code or git, store only a **hash** in DB, allow rotation, scope permissions, rate limit per key.

---

## 15. Password hashing vs encryption vs encoding

> 🎤 **Say it like this:** "Encoding is just a format change, anyone can reverse it — like base64. Encryption is reversible with a key — used when you need the original back. Hashing is one-way — used for passwords. For passwords I use a slow, salted hash like bcrypt or argon2, never encryption and never plain SHA-256."

| | Encoding | Encryption | Hashing |
|---|---|---|---|
| Reversible? | Yes, by anyone | Yes, with key | ❌ No (one-way) |
| Purpose | Data format | Confidentiality | Integrity / password check |
| Example | Base64, URL encoding | AES, RSA | bcrypt, argon2, SHA-256 |
| Use for passwords? | ❌ | ❌ | ✅ (slow + salted) |

- **Salt:** random value per user added before hashing → same passwords get different hashes, rainbow tables don't work. bcrypt/argon2 add it automatically.
- **Why slow hashes:** SHA-256 can do billions/sec on a GPU; bcrypt (cost factor) / argon2 make brute force very expensive.

```js
const hash = await bcrypt.hash(password, 12);        // on signup
const ok   = await bcrypt.compare(password, hash);   // on login
```

---

## 16. SQL injection (briefly)

User input becomes part of the SQL command.

```js
// ❌ vulnerable:  email = "' OR 1=1 --"
db.query(`SELECT * FROM users WHERE email = '${email}'`);

// ✅ parameterized query
db.query('SELECT * FROM users WHERE email = $1', [email]);
```

Fix: parameterized queries / ORM, validate input, least-privilege DB user. Same idea for NoSQL injection (`{ "$gt": "" }` in Mongo).

---

## 17. Secrets in environment variables

- Never commit secrets (DB passwords, JWT secret, API keys) to git. `.env` goes in `.gitignore`.
- Load from env vars locally; in production use a **secret manager** (AWS Secrets Manager, SSM Parameter Store, Vault).
- Never expose backend secrets in frontend bundles — anything in `NEXT_PUBLIC_*` / `VITE_*` is public.
- Rotate secrets; leaked key = rotate immediately, not just delete the commit.

---

## ⚡ Quick revision

- AuthN = who (401), AuthZ = what (403).
- Session = state on server, easy revoke. JWT = signed (not encrypted), stateless, hard to revoke.
- Short access token + rotating refresh token in HttpOnly cookie.
- localStorage → XSS risk. Cookies → CSRF risk (fix with SameSite + CSRF token).
- Cookie flags: HttpOnly, Secure, SameSite (Lax default, None needs Secure).
- XSS: escape output, sanitize, CSP. CORS ≠ security.
- OAuth = authorization, OIDC = authentication on top, use Auth Code + PKCE.
- HTTPS stops MITM; HSTS forces HTTPS.
- Rate limit with Redis, token bucket, return 429 + Retry-After.
- Passwords → bcrypt/argon2 with salt. Never encrypt or base64 them.
- Parameterized queries. Secrets in a secret manager, never in git or frontend.
