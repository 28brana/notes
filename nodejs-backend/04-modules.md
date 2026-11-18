# 04 · Modules: CommonJS vs ES Modules

> [!info] Topic info
> **Subject:** Node.js Backend · **Part:** Node.js Core · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Node.js Backend](README.md) · [Interview questions →](questions/nodejs-core-qa.md)

> [!abstract] In one line
> Node has two module systems: **CommonJS** (`require`, sync, Node's original) and **ES Modules** (`import`, static, the JS standard) — chosen per file by **extension** or `"type"` in package.json.

---

## 1. The problem

A real app has hundreds of files. Without modules everything would share global variables and collide. Modules give each file its **own scope** and a clear list of what it exports.

---

## 2. The two systems

```js
// CommonJS (math.js)
module.exports = { add: (a, b) => a + b };
const { add } = require('./math');

// ESM (math.mjs)
export const add = (a, b) => a + b;
import { add } from './math.mjs';
```

| | **CommonJS** | **ES Modules** |
|---|---|---|
| **Syntax** | `require` / `module.exports` | `import` / `export` |
| **Loading** | Synchronous, at runtime | Async, parsed **before** running |
| **Imports** | Dynamic — `require` anywhere, even in `if` | Static at top; `await import()` for dynamic |
| **Exports** | Copy of the value at export time | **Live bindings** — see updates |
| **Tree-shaking** | Hard | Easy (static) |
| **Top-level await** | ❌ | ✅ |
| **`__dirname`, `require`** | ✅ available | ❌ use `import.meta.dirname` / `import.meta.url` |
| **Extensions in path** | Optional | Required (`./math.js`) |

**How Node picks:** `.cjs` → CommonJS, `.mjs` → ESM, `.js` → whatever `"type"` in the nearest `package.json` says (`"module"` or `"commonjs"`, default commonjs).

**Mixing:** ESM can `import` a CJS file. CJS couldn't `require()` ESM for years (had to use `await import()`); newer Node versions (22+) allow `require()` of ESM without top-level await.

---

## 3. How `require` works

Resolve path → wrap the file in a function `(exports, require, module, __filename, __dirname)` → run it → **cache** the result.

The second `require('./db')` returns the **same object** from cache — that's why a module is a natural **singleton** (one DB pool for the whole app).

> [!warning] Circular imports
> `a.js` requires `b.js`, which requires `a.js`. In CJS, `b` gets a **half-finished** `a` (whatever was exported so far) — often `undefined` functions. Fix by moving shared code to a third module.

> [!tip] 🎤 Say it like this
> "CommonJS is Node's original system — require is synchronous and dynamic, and modules are cached so they behave like singletons. ES Modules are the standard: imports are static so tools can tree-shake, they support top-level await and live bindings. For new projects I use ESM with type module, but I still meet CommonJS in older code and many packages."

> [!failure] Interview traps
> - Using `__dirname` in ESM — it's not defined.
> - Thinking `exports = {...}` works — it breaks the link; use `module.exports = {...}`.
> - Forgetting the file extension in ESM imports.

---

## Summary

- **CJS:** `require`, sync, dynamic, cached.
- **ESM:** `import`, static, async, live bindings, top-level await.
- Mode decided by `.cjs` / `.mjs` / `"type"` in package.json.

> [!success] Key takeaway
> New code → ESM. Know CJS well because the module cache, singletons and circular-import bugs come up in interviews.
