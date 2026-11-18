# 10 · TypeScript Basics

> [!info] Topic info
> **Subject:** Frontend · **Part:** JavaScript, HTML/CSS & the Browser · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Frontend](README.md) · [Interview questions →](questions/javascript-and-browser-qa.md)

> [!abstract] In one line
> **TypeScript** is JavaScript plus **static types** checked at **compile time** — it catches bugs like `user.nmae` before the code runs, then compiles away to plain JS.

---

## 1. The problem

An API changes `price` from number to string. In plain JS, `price * qty` quietly becomes `NaN` in production. With TypeScript, the editor marks every broken line **before you deploy**, and refactors across 200 files become safe.

> [!note]
> Types exist **only at compile time**. At runtime it's plain JS — so data from APIs still needs runtime validation (e.g. Zod).

---

## 2. Types vs interfaces

```ts
interface User { id: number; name: string; email?: string }   // ? = optional
type Status = "active" | "banned";                              // union — only type can do this
type Admin = User & { permissions: string[] };                  // intersection
```

| | **interface** | **type** |
|---|---|---|
| **Object shapes** | ✅ | ✅ |
| **Unions, primitives, tuples** | ❌ | ✅ |
| **Extend** | `extends` | `&` |
| **Declaration merging** | ✅ Same name merges | ❌ Error |

Rule of thumb: **interface** for object shapes / public APIs, **type** for unions and everything else. Teams just pick one and stay consistent.

---

## 3. Generics

A **generic** is a type parameter — "fill in the type later" — so one function works for many types without losing type info.

```ts
function first<T>(arr: T[]): T | undefined { return arr[0]; }
first([1, 2]);        // number
first(["a"]);         // string

interface ApiResponse<T> { data: T; error?: string }
const res: ApiResponse<User[]> = await getUsers();
```

Add limits with `extends`: `function getId<T extends { id: number }>(x: T) { return x.id; }`.

---

## 4. any vs unknown vs never

| Type | Meaning | Can use it directly? |
|---|---|---|
| **any** | Turn off type checking | ✅ Anything goes (unsafe) |
| **unknown** | "Could be anything — check first" | ❌ Must narrow first |
| **never** | A value that can't exist | Used for exhaustive checks |

```ts
function handle(input: unknown) {
  if (typeof input === "string") return input.toUpperCase(); // narrowed ✅
}
```

**Narrowing** = TS learns the exact type from checks: `typeof`, `instanceof`, `in`, `===`, or a discriminant field (`if (shape.kind === "circle")`).

---

## 5. Utility types

| Utility | Does | Example |
|---|---|---|
| **Partial\<T\>** | All fields optional | Update/patch payloads |
| **Required\<T\>** | All fields required | |
| **Pick\<T, K\>** | Keep some keys | `Pick<User, "id" \| "name">` |
| **Omit\<T, K\>** | Drop some keys | `Omit<User, "password">` |
| **Readonly\<T\>** | No re-assigning fields | Config objects |
| **Record\<K, V\>** | Object with keys K, values V | `Record<string, number>` |
| **ReturnType\<F\>** | Type a function returns | |

> [!tip] 🎤 Say it like this
> "TypeScript adds compile-time types to JavaScript, so mistakes show up in the editor instead of production. I use interfaces for object shapes, type aliases for unions, generics for reusable functions like API wrappers, and `unknown` instead of `any` for untrusted data so I'm forced to check it. Utility types like `Partial`, `Pick` and `Omit` let me derive types instead of duplicating them."

> [!failure] Interview traps
> - Using `any` everywhere — you lose all the benefits.
> - Assuming TS validates API responses at runtime — it doesn't.
> - `as User` casts silence errors without checking anything.

---

## Summary

- TS = JS + compile-time types; zero runtime cost, zero runtime checks.
- interface for shapes; type for unions/intersections.
- Generics = reusable, type-safe code.
- Prefer `unknown` over `any`; narrow before use.

> [!success] Key takeaway
> TypeScript is a safety net for refactoring and teamwork — but validate data at the boundaries, because types disappear at runtime.
