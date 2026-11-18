# 14 · REST API Design

> [!info] Topic info
> **Subject:** Node.js Backend · **Part:** Building Production Backends · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Node.js Backend](README.md) · [Interview questions →](questions/building-backends-qa.md)

> [!abstract] In one line
> A good REST API uses **nouns for resources**, **HTTP methods for actions**, **correct status codes**, **pagination**, **versioning** and **one consistent error format** — so clients can guess how it works.

---

## 1. The problem

The mobile team calls `POST /getUserOrders`, gets `200 OK` with `{ success: false }` on errors, and receives 50,000 orders in one response. Every endpoint behaves differently.

REST principles are covered in [REST](../system-design/api-fundamentals/05-rest.md). Here: the practical rules you apply in an Express API.

---

## 2. URLs and methods

| Action | Request | Success code |
|---|---|---|
| **List** | `GET /api/v1/orders?status=paid&page=2` | 200 |
| **Get one** | `GET /api/v1/orders/42` | 200 / 404 |
| **Create** | `POST /api/v1/orders` | **201** + `Location` header |
| **Replace / update** | `PUT` / `PATCH /api/v1/orders/42` | 200 or 204 |
| **Delete** | `DELETE /api/v1/orders/42` | 204 |
| **Nested** | `GET /api/v1/users/7/orders` | 200 |

Plural nouns, no verbs in URLs. GET, PUT and DELETE are **idempotent** (repeat = same result); POST is not — use an `Idempotency-Key` header for payments (see [idempotency](../system-design/api-fundamentals/03-idempotency.md)).

---

## 3. Status codes you must know

| Code | Meaning | Example |
|---|---|---|
| **400** | Bad input | Missing `email` |
| **401** | Not authenticated | No / expired token |
| **403** | Authenticated but not allowed | User editing another user's order |
| **404** | Not found | Order 999 |
| **409** | Conflict | Email already registered |
| **422** | Valid JSON but fails rules | Quantity = -3 |
| **429** | Rate limited | Too many requests |
| **500 / 503** | Server error / unavailable | DB down |

---

## 4. Pagination, filtering, versioning

**Offset pagination** — `?page=3&limit=20` → `OFFSET 40 LIMIT 20`. Simple, allows jumping to page N, but slow on deep pages (DB still scans 1M rows for page 50,000) and rows shift when new data is inserted.

**Cursor pagination** — `?limit=20&cursor=<last id>` → `WHERE id < :cursor ORDER BY id DESC LIMIT 20`. Fast at any depth and stable for infinite feeds; can't jump to a page number.

Always cap `limit` (e.g. max 100) and return `{ data, nextCursor }` or `{ data, page, total }`.

**Versioning** — URL (`/api/v1`) is most common and easy to route; headers (`Accept: application/vnd.app.v2+json`) are cleaner but harder to test. Only bump the version for **breaking** changes; adding a field is not breaking.

---

## 5. One error format

```json
{ "error": { "code": "VALIDATION_FAILED", "message": "email is required",
             "details": [{ "field": "email", "issue": "required" }], "requestId": "abc-123" } }
```

Produce it from the central error middleware ([middleware](13-middleware.md)) by throwing custom errors like `new NotFoundError('Order')` that carry a status and code.

> [!tip] 🎤 Say it like this
> "I use plural nouns with HTTP verbs, return proper codes — 201 for create, 204 for delete, 401 vs 403, 409 for conflicts — and one error shape with a code and request ID. Lists are always paginated; cursor-based for feeds because offset gets slow on deep pages. I version in the URL and only bump it for breaking changes."

> [!failure] Interview traps
> - Returning `200` with an error body.
> - Mixing up **401** (who are you?) and **403** (I know you, but no).
> - Unbounded list endpoints — one request returns the whole table.
> - Breaking changes (renaming a field) without a new version.

---

## Summary

- Nouns + methods, correct codes, `Location` on 201.
- Paginate everything; **cursor** for big/live data, **offset** for admin tables.
- `/v1` URL versioning, bump only on breaking changes.
- One error format from one error handler.

> [!success] Key takeaway
> Predictability is the goal: a client who has used one of your endpoints should be able to guess how all the others behave.
