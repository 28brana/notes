# 15 · Validation

> [!info] Topic info
> **Subject:** Node.js Backend · **Part:** Building Production Backends · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Node.js Backend](README.md) · [Interview questions →](questions/building-backends-qa.md)

> [!abstract] In one line
> **Validate every input at the edge** (body, params, query) with a **schema library** like **zod** or **joi**, reject bad data with **400**, and pass only clean, typed data into your services.

---

## 1. The problem

A client sends `{ "age": "abc", "role": "admin" }` to `POST /users`. Without checks, `"abc"` reaches the DB, and the user just made themselves an admin (**mass assignment**).

**Never trust the client** — the frontend's form validation is for UX, not security. Anyone can call your API with curl.

---

## 2. Schema validation with zod

Describe the shape once; the library checks it and gives you typed, cleaned data.

```js
const { z } = require('zod');
const createUser = z.object({
  email: z.string().email(),
  name:  z.string().min(1).max(100).trim(),
  age:   z.coerce.number().int().min(13).optional(),
}).strict();                              // unknown keys like "role" → error

const validate = (schema) => (req, res, next) => {
  const result = schema.safeParse(req.body);
  if (!result.success) return res.status(400).json({ error: { code: 'VALIDATION_FAILED', details: result.error.issues } });
  req.body = result.data;                 // only the parsed, whitelisted fields
  next();
};
router.post('/users', validate(createUser), userCtrl.create);
```

---

## 3. Which library?

| Library | Strength |
|---|---|
| **zod** | TypeScript-first — `z.infer<typeof schema>` gives you the type for free |
| **joi** | Mature, very expressive, popular in older Express apps |
| **express-validator** | Chain-style checks per field, built on validator.js |
| **Fastify / AJV** | JSON Schema, compiled — very fast; built into Fastify |
| **NestJS** | `class-validator` decorators on DTO classes + `ValidationPipe` |

> [!tip] 🎤 Say it like this
> "I validate body, params and query in a middleware using a zod schema, right after the router. Unknown fields are stripped or rejected to stop mass assignment, and errors return 400 with field-level details. Services then receive data that's already typed and safe, so they don't re-check shapes — only business rules."

> [!failure] Interview traps
> - Relying on frontend validation only.
> - Passing `req.body` straight into `User.create(req.body)` → mass assignment.
> - Forgetting to validate **params and query** (`?limit=1000000`).
> - Confusing **validation** (shape: is it an email?) with **business rules** (is this email already taken? → 409, done in the service).

---

## Summary

- Validate at the edge, in middleware, with a schema.
- Whitelist fields (`.strict()` or strip unknown keys).
- 400 + field details on failure.
- Shape checks in middleware, business checks in services.

> [!success] Key takeaway
> One schema per endpoint turns "any JSON from the internet" into "data I can trust" before your code touches it.
