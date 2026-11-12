# Q&A · Authentication and Network Security

[← Back to Computer Networks](../README.md) · [Read the notes →](../06-auth-and-security.md)

> Short answers you can say out loud. 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

### 🟢 Q1. Authentication vs authorization?
Authentication checks who you are (login). Authorization checks what you're allowed to do (roles/permissions). Failing AuthN → 401, failing AuthZ → 403.

### 🟢 Q2. How does session-based authentication work?
On login the server stores a session (e.g. in Redis) and sends a random session ID in an HttpOnly cookie. The browser sends the cookie on each request; the server looks up the session. Logout = delete the session.

### 🟢 Q3. What is a JWT and what are its parts?
A signed token: `header.payload.signature`, each base64url-encoded. Header = algorithm, payload = claims like `sub` and `exp`, signature = proof it wasn't modified.

### 🟡 Q4. Is a JWT encrypted?
No, it's only signed. Anyone can decode and read the payload, so never put passwords or sensitive data in it. The signature only guarantees it wasn't tampered with.

### 🟡 Q5. Session vs JWT — when would you use each?
Sessions: classic web apps on one domain, need instant logout/revocation. JWT: mobile apps, microservices, or third-party APIs where services verify tokens without a central lookup. Common mix: short JWT access token + revocable refresh token.

### 🟡 Q6. How do you log out / revoke a JWT?
You can't delete it, it's valid until `exp`. Use short expiry, revoke the refresh token, and if needed keep a denylist (by `jti`) in Redis or a `tokenVersion` on the user.

### 🟡 Q7. Why use access and refresh tokens?
Access token is short-lived (minutes) so a leak does limited damage. Refresh token is long-lived and used only to get new access tokens, so the user doesn't log in again.

### 🔴 Q8. What is refresh token rotation?
Every refresh returns a new refresh token and invalidates the old one. If an old token is reused, it means it was stolen — revoke the whole token family and force login.

### 🟡 Q9. Where should you store tokens in the browser?
Prefer HttpOnly + Secure + SameSite cookies — JS can't read them, so XSS can't steal them. localStorage is readable by any script (XSS risk). A common SPA pattern: access token in memory, refresh token in HttpOnly cookie.

### 🟢 Q10. What do HttpOnly, Secure and SameSite do?
HttpOnly: JS can't read the cookie. Secure: only sent over HTTPS. SameSite: controls sending the cookie on cross-site requests — Strict (never), Lax (only top-level GET navigation, default), None (always, must be Secure).

### 🟡 Q11. What is CSRF and how do you prevent it?
An evil site makes the user's browser send a request to your site, and the browser attaches cookies automatically. Prevent with SameSite cookies, CSRF tokens (or double-submit cookie), Origin header checks, and no state changes on GET.

### 🟡 Q12. What is XSS? How do you prevent it?
Attacker's script runs on your page (stored, reflected or DOM-based), and can read localStorage and call APIs as the user. Prevent by escaping output, sanitizing HTML (DOMPurify), avoiding `innerHTML`/`dangerouslySetInnerHTML`, CSP header and HttpOnly cookies.

### 🟡 Q13. Does CORS protect my API?
No. CORS only controls whether browser JS from another origin can read the response. curl, Postman and servers ignore it. Protect APIs with authentication, authorization and rate limiting.

### 🟡 Q14. OAuth 2.0 vs OpenID Connect?
OAuth 2.0 is for authorization — giving an app access to resources on your behalf (access token). OIDC adds authentication on top — an ID token that says who the user is. "Login with Google" uses OIDC.

### 🔴 Q15. Explain the Authorization Code flow with PKCE.
App creates a random `code_verifier` and sends its hash (`code_challenge`) when redirecting to the auth server. User logs in, auth server redirects back with a `code`. App exchanges `code + code_verifier` for tokens. A stolen code is useless without the verifier.

### 🟢 Q16. What is SSO?
Single Sign-On — log in once with an identity provider (Okta, Google, Azure AD) and access many apps without logging in again. Built on OIDC or SAML.

### 🟡 Q17. What is a MITM attack and how does HTTPS/HSTS help?
Someone between client and server reads or changes traffic (e.g. public Wi-Fi). TLS encrypts the traffic and the certificate proves server identity. HSTS forces the browser to always use HTTPS, so it can't be downgraded to HTTP.

### 🟡 Q18. How would you implement rate limiting?
Key by user/API key/IP, keep counters in Redis so all servers share them, use token bucket or sliding window, and return 429 with `Retry-After`. Also add limits at the edge (API Gateway, Nginx, Cloudflare).

### 🟢 Q19. API key vs access token?
API key identifies an application, is long-lived and used server-to-server. Access token represents a user and their permissions and expires quickly.

### 🟡 Q20. Hashing vs encryption vs encoding?
Encoding (base64) is reversible by anyone — just a format. Encryption (AES) is reversible with a key. Hashing is one-way. Passwords must be hashed with a slow salted algorithm like bcrypt or argon2.

### 🟡 Q21. Why add a salt? Why not SHA-256 for passwords?
Salt makes identical passwords produce different hashes and kills rainbow tables. SHA-256 is too fast — GPUs can try billions per second; bcrypt/argon2 are deliberately slow.

### 🟢 Q22. How do you prevent SQL injection?
Use parameterized queries or an ORM, never string-concatenate user input into SQL. Also validate input and use a least-privilege DB user.

### 🔴 Q23. A developer accidentally pushed the `.env` with the JWT secret to GitHub. What do you do?
Rotate the secret immediately (all existing tokens become invalid → users re-login), rotate any other leaked keys, check logs for misuse, then remove it from git history. Move secrets to a secret manager and add secret scanning in CI.

### 🔴 Q24. Your SPA on `app.example.com` calls `api.example.com` with cookies, but the cookie isn't sent. Why?
Check: fetch needs `credentials: 'include'`; server needs `Access-Control-Allow-Credentials: true` and an exact origin (not `*`); cookie `Domain=example.com` so the subdomain gets it; `Secure` on HTTPS. Same-site subdomains work with `SameSite=Lax`; truly cross-site needs `SameSite=None; Secure`.
