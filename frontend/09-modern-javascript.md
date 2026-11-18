# 09 · Modern JavaScript (ES6+)

> [!info] Topic info
> **Subject:** Frontend · **Part:** JavaScript, HTML/CSS & the Browser · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Frontend](README.md) · [Interview questions →](questions/javascript-and-browser-qa.md)

> [!abstract] In one line
> **ES6 (2015) and later** added the features every modern codebase uses: **`let`/`const`, arrow functions, destructuring, spread/rest, template literals, classes, modules, optional chaining `?.` and nullish coalescing `??`**.

---

## 1. Why interviewers ask

Code reviews and live-coding rounds assume you write modern JS fluently. They also probe the subtle bits: `??` vs `||`, shallow copies with spread, and ES modules vs CommonJS.

---

## 2. The features that matter

**Destructuring** — pull values out of objects/arrays:

```js
const { name: fullName, address: { city } = {}, role = "user" } = user; // rename, nested, default
const [first, , third] = [10, 20, 30];                        // skip items
function render({ title, items = [] }) { … }                   // in parameters
```

**Spread / rest** — same `...`, opposite jobs:

```js
const merged = { ...defaults, ...options };   // spread: expand (later keys win)
const copy = [...arr, 4];                      // new array
function sum(...nums) { return nums.reduce((a, b) => a + b, 0); } // rest: collect
```

> [!warning] Spread is a shallow copy
> `{ ...user }` copies top-level keys only. `copy.address.city = "X"` still changes the original. Use `structuredClone(user)` for a deep copy.

**Optional chaining & nullish coalescing:**

```js
const city = user?.address?.city;      // undefined instead of a crash
const n = settings.count ?? 10;        // 10 only if count is null/undefined
const m = settings.count || 10;        // 10 also when count is 0 or "" ❌
```

**Others to know:** template literals (`` `Hi ${name}` ``), arrow functions (no own `this` — see [06](06-closures-this-prototypes.md)), default params, `class`, `Map`/`Set` (any key type, keeps insert order), `for…of`, `Array.prototype.includes/flat/at`, `Object.entries/fromEntries`.

---

## 3. Modules — ES modules vs CommonJS

| | **ES modules (ESM)** | **CommonJS (CJS)** |
|---|---|---|
| **Syntax** | `import` / `export` | `require` / `module.exports` |
| **Loading** | Static — analysed before running | Dynamic — runs when called |
| **Tree-shaking** | ✅ Bundlers drop unused exports | ❌ Hard |
| **Where** | Browsers, modern Node, bundlers | Older Node code |

```js
// math.js
export const add = (a, b) => a + b;
export default function mul(a, b) { return a * b; }
// app.js
import mul, { add } from "./math.js";
const { Chart } = await import("./chart.js");  // dynamic import → code splitting
```

Node's side of this is in [Node.js modules](../nodejs-backend/04-modules.md).

> [!tip] 🎤 Say it like this
> "The ES6+ features I use daily are destructuring, spread and rest, arrow functions, optional chaining and nullish coalescing, and ES modules. Two things I'm careful with: spread only makes a shallow copy, and `??` is better than `||` for defaults because it keeps valid values like 0 and empty string."

> [!failure] Interview traps
> - Using `||` for defaults and losing `0`, `""`, or `false`.
> - Thinking spread deep-copies nested objects.
> - Mixing up rest (collects) and spread (expands).
> - Saying `import` can go inside an `if` — static imports must be top-level; use `import()` instead.

---

## Summary

- Destructuring + defaults for clean parameter handling.
- Spread = shallow copy / merge; rest = collect arguments.
- `?.` avoids crashes; `??` defaults only on null/undefined.
- ESM is static → tree-shaking; `import()` → code splitting.

> [!success] Key takeaway
> Write modern syntax by default, and know its two sharp edges: shallow spread and `||` vs `??`.
