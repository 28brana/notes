# 13 · JS Coding Questions (Polyfills & Utilities)

> [!info] Topic info
> **Subject:** Frontend · **Part:** JavaScript, HTML/CSS & the Browser · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Frontend](README.md) · [Interview questions →](questions/javascript-and-browser-qa.md)

> [!abstract] In one line
> Frontend rounds love **"write it yourself"** questions — **debounce, throttle, Promise.all, bind, deep clone, flatten, memoize, curry, event emitter** — which test **closures, `this`, promises and recursion** in 10 lines each.

---

## 1. How to approach them

1. **Restate** the behaviour and ask about edge cases (empty input, errors, `this`).
2. Write the **simple version** first, then improve.
3. **Talk through** what the closure stores and why.

Each solution below uses ideas from [06 · Closures & this](06-closures-this-prototypes.md) and [07 · Async JS](07-async-javascript.md).

---

## 2. Timing: debounce & throttle

```js
function debounce(fn, ms) {
  let timer;
  return function (...args) {
    clearTimeout(timer);                              // reset on every call
    timer = setTimeout(() => fn.apply(this, args), ms);
  };
}
function throttle(fn, ms) {
  let last = 0;
  return function (...args) {
    const now = Date.now();
    if (now - last >= ms) { last = now; fn.apply(this, args); }
  };
}
```

When to use which: [08 · DOM & events](08-dom-and-events.md).

---

## 3. Promise.all polyfill

```js
function promiseAll(items) {
  return new Promise((resolve, reject) => {
    const results = []; let done = 0;
    if (items.length === 0) return resolve(results);
    items.forEach((p, i) => {
      Promise.resolve(p).then((val) => {
        results[i] = val;                              // keep original order
        if (++done === items.length) resolve(results);
      }, reject);                                      // first failure rejects
    });
  });
}
```

---

## 4. Function.prototype.bind polyfill

```js
Function.prototype.myBind = function (ctx, ...preset) {
  const fn = this;                                     // the function being bound
  return function (...later) {
    return fn.apply(ctx, [...preset, ...later]);
  };
};
```

(The full spec also handles `new` on the bound function — mention it, rarely needed.)

---

## 5. Deep clone & flatten

```js
function deepClone(obj, seen = new WeakMap()) {
  if (obj === null || typeof obj !== "object") return obj;
  if (seen.has(obj)) return seen.get(obj);             // handles circular refs
  const copy = Array.isArray(obj) ? [] : {};
  seen.set(obj, copy);
  for (const key of Object.keys(obj)) copy[key] = deepClone(obj[key], seen);
  return copy;
}

function flatten(arr, depth = Infinity) {
  return arr.reduce((acc, x) =>
    Array.isArray(x) && depth > 0 ? acc.concat(flatten(x, depth - 1)) : acc.concat([x]), []);
}
```

In real code: `structuredClone(obj)` and `arr.flat(Infinity)`. Mention that `JSON.parse(JSON.stringify(x))` loses dates, `undefined`, functions and fails on cycles.

---

## 6. Memoize & curry

```js
function memoize(fn) {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (!cache.has(key)) cache.set(key, fn(...args));
    return cache.get(key);
  };
}

function curry(fn) {
  return function curried(...args) {
    return args.length >= fn.length
      ? fn(...args)
      : (...more) => curried(...args, ...more);
  };
}
// curry((a, b, c) => a + b + c)(1)(2)(3) === 6
```

---

## 7. Event emitter

```js
class Emitter {
  events = {};
  on(name, fn) { (this.events[name] ||= []).push(fn); return () => this.off(name, fn); }
  off(name, fn) { this.events[name] = (this.events[name] || []).filter((f) => f !== fn); }
  emit(name, ...args) { (this.events[name] || []).slice().forEach((fn) => fn(...args)); }
  once(name, fn) {
    const off = this.on(name, (...a) => { off(); fn(...a); });
  }
}
```

`slice()` copies the list so a listener that unsubscribes during `emit` doesn't skip others. Node's built-in version: [Node event emitter](../nodejs-backend/06-event-emitter.md).

> [!tip] 🎤 Say it like this
> "Most of these are closures in disguise: debounce keeps a timer, memoize keeps a cache, curry keeps the arguments so far. For `Promise.all` the key points are keeping results in the original order, resolving when the counter hits the length, and rejecting on the first failure. I'd mention edge cases like empty input, `this` binding, and circular references."

> [!failure] Interview traps
> - Debounce/throttle with arrow + `fn(...args)` only — loses `this` for methods. Use `function` + `apply`.
> - `Promise.all` pushing results with `push` — order depends on finish time, which is wrong.
> - Forgetting the empty-array case (should resolve immediately with `[]`).
> - Deep clone without cycle handling → stack overflow.

---

## Summary

| Question | Core idea |
|---|---|
| **Debounce / throttle** | Closure over timer / last time |
| **Promise.all** | Counter + index-ordered results + first reject |
| **bind** | Save `this` as fn, `apply` with preset args |
| **Deep clone / flatten** | Recursion (+ WeakMap for cycles) |
| **Memoize / curry** | Closure over cache / collected args |
| **Event emitter** | Map of name → listener arrays |

> [!success] Key takeaway
> Practise writing each one from memory in under 5 minutes — and always say the edge cases out loud.
