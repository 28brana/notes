# 11 · Browser Storage — Cookies, localStorage, sessionStorage, IndexedDB

> [!info] Topic info
> **Subject:** Frontend · **Part:** JavaScript, HTML/CSS & the Browser · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Frontend](README.md) · [Interview questions →](questions/javascript-and-browser-qa.md)

> [!abstract] In one line
> **Cookies** are small and **sent to the server** with every request; **localStorage** (forever) and **sessionStorage** (per tab) are simple string stores the server never sees; **IndexedDB** is a real async database for large data.

---

## 1. The problem

Our app needs to remember four things: the login session, the user's dark-mode choice, a half-filled checkout form, and 50 MB of offline messages. Each one fits a different storage. Picking wrong means security holes or a frozen page.

---

## 2. The comparison

| | **Cookies** | **localStorage** | **sessionStorage** | **IndexedDB** |
|---|---|---|---|---|
| **Size** | ~4 KB each | ~5–10 MB | ~5 MB | Hundreds of MB+ (quota-based) |
| **Lifetime** | Until `Expires`/`Max-Age` (or browser close) | Until cleared | Until the **tab** closes | Until cleared |
| **Sent to server?** | ✅ Every request to that domain | ❌ | ❌ | ❌ |
| **Shared across tabs?** | ✅ | ✅ Same origin | ❌ Per tab | ✅ Same origin |
| **API** | `document.cookie` / `Set-Cookie` header | Sync, strings only | Sync, strings only | Async, stores objects, indexes |
| **JS can read?** | Not if `HttpOnly` | ✅ | ✅ | ✅ |
| **Best for** | Session / auth tokens | Theme, preferences | One-tab wizard / form draft | Offline data, big caches |

```js
localStorage.setItem("prefs", JSON.stringify({ theme: "dark" })); // must stringify
const prefs = JSON.parse(localStorage.getItem("prefs") ?? "{}");
```

---

## 3. Where should auth tokens go?

Anything in localStorage can be read by **any** JS on the page — so one [XSS](12-web-security.md) bug leaks every user's token.

A cookie with **`HttpOnly; Secure; SameSite=Lax`** can't be read by JS at all, only travels over HTTPS, and isn't sent on most cross-site requests (helps against CSRF). Full trade-offs are in [Auth & security → where to store tokens](../computer-networks/06-auth-and-security.md) and cookie basics in [Browser networking](../computer-networks/03-browser-networking.md).

> [!tip] 🎤 Say it like this
> "Cookies are tiny and go to the server with every request, so they're for session or auth — set as HttpOnly, Secure and SameSite so JavaScript can't steal them. localStorage is for non-sensitive preferences that should survive restarts, sessionStorage for per-tab state, and IndexedDB when I need lots of structured data or offline support, since it's async and doesn't block the main thread."

> [!warning] localStorage is synchronous
> Every read/write blocks the main thread. Fine for a few KB of settings; parsing a 4 MB JSON blob on every load is not — use IndexedDB.

> [!failure] Interview traps
> - Storing JWTs or passwords in localStorage "because it's easy".
> - Putting big data in cookies — every request carries it (and requests to static assets on that domain too).
> - Forgetting `JSON.stringify` — you store `"[object Object]"`.
> - Thinking sessionStorage is shared between tabs.

---

## Summary

- Cookies → server needs it (auth), keep tiny, use `HttpOnly; Secure; SameSite`.
- localStorage → small, non-sensitive, persistent.
- sessionStorage → per tab, dies with the tab.
- IndexedDB → large, structured, async, offline.

> [!success] Key takeaway
> Ask two questions: *does the server need it?* (cookie) and *is it sensitive?* (never in JS-readable storage).
