# 05 · JavaScript Fundamentals — Types, Scope & Hoisting

> [!info] Topic info
> **Subject:** Frontend · **Part:** JavaScript, HTML/CSS & the Browser · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Frontend](README.md) · [Interview questions →](questions/javascript-and-browser-qa.md)

> [!abstract] In one line
> JS has **7 primitive types + objects**, **loose (`==`) vs strict (`===`) equality**, and three ways to declare variables — **`var` (function-scoped, hoisted as undefined)** vs **`let`/`const` (block-scoped, in the TDZ until declared)**.

---

## 1. The problem

These look like trick questions, but they're the #1 opener in frontend rounds:

```js
console.log(a); // undefined
var a = 5;
console.log(b); // ReferenceError
let b = 5;
[] == false;    // true 🤯
```

You need to explain *why*, not just memorise outputs.

---

## 2. Types

**Primitives (7):** `string`, `number`, `bigint`, `boolean`, `undefined`, `null`, `symbol`. Stored **by value**, immutable.
**Everything else is an object** — arrays, functions, dates. Stored **by reference**.

```js
const a = { n: 1 }; const b = a;
b.n = 2;            // a.n is now 2 — same object
typeof null;        // "object"  (old bug, kept forever)
typeof [];          // "object"  → use Array.isArray([])
typeof function(){} // "function"
```

- **`undefined`** = declared but no value yet. **`null`** = "intentionally empty", set by you.
- **Falsy values (8):** `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`. Everything else is truthy — including `[]`, `{}`, `"0"`.
- `NaN !== NaN` → use `Number.isNaN(x)`.

---

## 3. `==` vs `===` and coercion

- **`===` (strict)** — same type **and** same value. No conversion.
- **`==` (loose)** — converts types first (**coercion**), then compares.

```js
1 == "1"            // true  (string → number)
0 == false          // true
null == undefined   // true  (special rule)
null == 0           // false
[] == false         // true  ([] → "" → 0, false → 0)
"5" + 1             // "51"  (+ with a string → concatenation)
"5" - 1             // 4     (- always converts to number)
```

> [!important] Golden rule
> Always use `===`. The one accepted exception: `x == null` to check for **both** null and undefined.

---

## 4. var vs let vs const, hoisting & TDZ

| | **var** | **let** | **const** |
|---|---|---|---|
| **Scope** | Function | Block `{}` | Block `{}` |
| **Hoisted?** | ✅ set to `undefined` | ✅ but in TDZ | ✅ but in TDZ |
| **Re-assign** | ✅ | ✅ | ❌ |
| **Re-declare** | ✅ | ❌ | ❌ |
| **On `window`** (global) | ✅ | ❌ | ❌ |

**Hoisting** = before running code, JS registers all declarations in their scope. `var` is pre-filled with `undefined`; **function declarations** are hoisted **fully** (you can call them before the line). Function *expressions* and arrow functions follow their `var`/`let` rules.

**TDZ (Temporal Dead Zone)** = the time between entering the block and reaching the `let`/`const` line. Touching the variable there throws `ReferenceError`.

> [!note] `const` is not "immutable"
> `const user = {}` stops you re-assigning `user`, but `user.name = "A"` still works. Use `Object.freeze` for shallow immutability.

**Scope:** global → function → block. Lookups walk **outwards** through parent scopes (the **scope chain**). Scope is decided by where code is **written** (lexical scope), not where it's called — this is the base for [closures](06-closures-this-prototypes.md).

> [!tip] 🎤 Say it like this
> "`var` is function-scoped and hoisted as `undefined`, so you can read it early without an error. `let` and `const` are block-scoped and hoisted too, but sit in the temporal dead zone until their line runs, so early access throws. I default to `const`, use `let` when I must re-assign, and avoid `var`. For comparisons I always use `===` to skip type coercion."

> [!failure] Interview traps
> - Saying `let`/`const` are "not hoisted" — they are, they're just in the TDZ.
> - Thinking `const` makes objects immutable.
> - `typeof null === "object"` and `typeof NaN === "number"`.
> - `[]` and `{}` are truthy.

---

## Summary

- 7 primitives (by value) + objects (by reference).
- Use `===`; `==` coerces types.
- `var` = function scope + `undefined` hoisting; `let`/`const` = block scope + TDZ.
- Function declarations are fully hoisted.

> [!success] Key takeaway
> Default to `const` and `===` — most JS "weirdness" comes from `var`, `==`, and coercion.
