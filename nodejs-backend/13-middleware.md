# 13 · Middleware

> [!info] Topic info
> **Subject:** Node.js Backend · **Part:** Building Production Backends · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Node.js Backend](README.md) · [Interview questions →](questions/building-backends-qa.md)

> [!abstract] In one line
> **Middleware** is a function `(req, res, next)` that runs **in order** for each request — it can read/modify the request, **end** the response, or call **`next()`** to pass control on; **error middleware** has **4 arguments** `(err, req, res, next)`.

---

## 1. The problem

Every endpoint needs logging, JSON parsing, auth checks and error handling. Copy-pasting that into 50 handlers is a bug factory.

Middleware lets you write it **once** and plug it into the pipeline — globally, per router, or per route.

---

## 2. How it works

```js
function requestTimer(req, res, next) {
  const start = Date.now();
  res.on('finish', () => console.log(req.method, req.url, Date.now() - start, 'ms'));
  next();                                 // forget this → request hangs forever
}
app.use(requestTimer);                    // global
app.get('/admin', requireAuth, requireRole('admin'), adminCtrl);   // per route
```

Each middleware must do **one** of: call `next()`, call `next(err)`, or **send a response**. Otherwise the client waits until timeout.

---

## 3. Order matters

Express runs middleware **top to bottom** in registration order. A typical order:

```mermaid
flowchart LR
    A[🛡️ helmet, cors] --> B[📝 request logger]
    B --> C[🚦 rate limit]
    C --> D[📦 body parser]
    D --> E[🔐 auth]
    E --> F[🛣️ routes]
    F --> G[❓ 404 handler]
    G --> H[🚨 error handler]
```

- Security headers and rate limiting **early** — reject bad traffic cheaply.
- Body parser **before** routes that read `req.body`.
- **404 handler after all routes**, **error handler last**.

---

## 4. Error middleware + async errors

Express recognises error middleware by its **4 parameters**. Anything passed to `next(err)` jumps straight to it, skipping normal middleware.

```js
app.use((err, req, res, next) => {
  req.log?.error(err);
  const status = err.status || 500;
  res.status(status).json({
    error: { code: err.code || 'INTERNAL', message: status < 500 ? err.message : 'Something went wrong' },
  });
});
```

**Async trap:** in **Express 4**, a rejected promise inside an `async` handler is **not** caught — the request hangs and you get an unhandled rejection. Fixes:
- Wrap handlers: `const ah = fn => (req, res, next) => fn(req, res, next).catch(next);`
- Or use the `express-async-errors` package.
- **Express 5** forwards rejected promises to `next(err)` automatically.

> [!tip] 🎤 Say it like this
> "Middleware is a chain of `(req, res, next)` functions that run in registration order. I put security and rate limiting first, then body parsing and auth, then routes, then a 404 handler and finally one central error handler with four arguments. In Express 4 I wrap async handlers so rejected promises reach that error handler."

> [!failure] Interview traps
> - Error handler with 3 args — Express treats it as normal middleware and skips it.
> - Registering the error handler **before** routes.
> - Leaking stack traces to clients in production.
> - Forgetting `next()` → hanging requests.

---

## Summary

| Kind | Signature | Use |
|---|---|---|
| **App-level** | `app.use(fn)` | Logging, parsing, helmet |
| **Router / route-level** | `router.use(fn)`, `app.get(path, fn, handler)` | Auth, role checks, validation |
| **Error** | `(err, req, res, next)` | One central error format |

> [!success] Key takeaway
> Middleware = ordered pipeline. Get the **order** right and have **one** error handler at the end that every error (sync or async) reaches.
