# 03 · Async Patterns: Callbacks → Promises → async/await

> [!info] Topic info
> **Subject:** Node.js Backend · **Part:** Node.js Core · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Node.js Backend](README.md) · [Interview questions →](questions/nodejs-core-qa.md)

> [!abstract] In one line
> Node async code evolved from **error-first callbacks** to **Promises** to **async/await**; the combinators **Promise.all / allSettled / race / any** let you run work **in parallel** safely.

---

## 1. The problem

Load a user, then their orders, then the products — each step needs the previous result. With callbacks this nests deeper and deeper ("**callback hell**"), and every level needs its own error check.

---

## 2. Three generations

**Callbacks** — Node's original style is **error-first**: `(err, result)`.

```js
fs.readFile('a.txt', 'utf8', (err, data) => {
  if (err) return handle(err);
  console.log(data);
});
```

**Promises** — an object for a future value: *pending → fulfilled or rejected*. Chain with `.then`, one `.catch` for all errors.

**async/await** — syntax on top of Promises. Reads like sync code, errors via `try/catch`.

```js
async function getOrders(userId) {
  try {
    const user = await db.users.find(userId);
    return await db.orders.findBy(user.id);
  } catch (err) {
    logger.error(err); throw err;
  }
}
```

Convert old callback APIs with `util.promisify(fn)`, or use promise versions directly: `require('node:fs/promises')`.

> [!note] `return await` inside try
> Inside a `try` block, `return await p` matters — without `await`, a rejection skips your `catch`.

---

## 3. Sequential vs parallel

```js
// ❌ sequential: 100 ms + 100 ms = 200 ms
const a = await getUser(); const b = await getStats();
// ✅ parallel: ~100 ms
const [a, b] = await Promise.all([getUser(), getStats()]);
```

| Combinator | Resolves when | Rejects when | Use for |
|---|---|---|---|
| **Promise.all** | all succeed → array | **first** rejection | Need every result |
| **Promise.allSettled** | all finish → `{status, value/reason}` | never | Partial failure OK (dashboard widgets) |
| **Promise.race** | first to **settle** (win or fail) | first settles with error | Timeouts |
| **Promise.any** | first to **succeed** | all fail → `AggregateError` | Fastest of several mirrors |

> [!warning] Unbounded parallelism
> `Promise.all(ids.map(fetch))` with 10,000 ids fires 10,000 requests at once — you'll exhaust DB connections or get rate-limited. Process in **batches** or use a concurrency limiter like `p-limit` (e.g. 10 at a time).

> [!tip] 🎤 Say it like this
> "Callbacks led to deep nesting and scattered error handling, so we moved to Promises, and async/await is syntax on top of them. I watch out for accidental sequential awaits — independent calls go into Promise.all. If partial failure is fine I use allSettled, race is good for timeouts, and any gives the first success."

> [!failure] Interview traps
> - `await` inside `forEach` — it doesn't wait. Use `for...of` (sequential) or `Promise.all(map)` (parallel).
> - Thinking `Promise.all` cancels the others on failure — they keep running; only the result is ignored. Use `AbortController` to really cancel.
> - Forgetting `await` → you get a Promise object, and errors become unhandled rejections.

---

## Summary

- **Callbacks** → error-first, nesting pain.
- **Promises** → chainable, one `.catch`.
- **async/await** → readable, `try/catch`.
- Run independent work with **Promise.all**; limit concurrency for big lists.

For browser-side async (event loop in the browser, fetch), see [Async JavaScript](../frontend/07-async-javascript.md).

> [!success] Key takeaway
> Use async/await for readability, Promise.all for speed, and always know where each rejection gets caught.
