# 04 · CSS Specificity, Positioning & z-index

> [!info] Topic info
> **Subject:** Frontend · **Part:** JavaScript, HTML/CSS & the Browser · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Frontend](README.md) · [Interview questions →](questions/javascript-and-browser-qa.md)

> [!abstract] In one line
> **Specificity** decides which CSS rule wins when several match; **position** decides how an element is placed; **z-index** only orders elements within the same **stacking context**.

---

## 1. The problem

You write `.btn { color: red }` but the button stays blue. Your modal has `z-index: 9999` but still hides behind the header. Both are "CSS is broken" moments — really they're specificity and stacking contexts.

---

## 2. Specificity — who wins

The browser scores each selector as **(IDs, classes, elements)** and compares left to right:

| Selector | Score |
|---|---|
| **`p`** | 0-0-1 |
| **`.btn`**, `[type=text]`, `:hover` | 0-1-0 |
| **`nav .btn:hover`** | 0-2-1 |
| **`#cart`** | 1-0-0 |
| **inline `style=""`** | beats all selectors |
| **`!important`** | beats everything (avoid) |

- **One ID beats any number of classes** — `#a` (1-0-0) beats `.a.b.c.d` (0-4-0).
- Same score → the rule **written later** wins.
- `*` and combinators (`>`, `+`) add nothing. `:where()` adds zero; `:is()` / `:not()` take their strongest argument.

**Cascade order:** importance (`!important`) → specificity → source order. Inherited values (like `color` from a parent) lose to any rule that targets the element directly.

> [!tip] 🎤 Say it like this
> "Specificity is a score of IDs, then classes, then elements. One ID beats any number of classes, inline styles beat selectors, and `!important` beats everything. If scores tie, the later rule wins. I keep specificity low and flat — mostly single classes — so I never need `!important`."

---

## 3. Position values

| Value | In normal flow? | Positioned relative to | Typical use |
|---|---|---|---|
| **static** (default) | ✅ | — (top/left ignored) | Everything |
| **relative** | ✅ (space kept) | Its own normal spot | Small nudges; anchor for absolute children |
| **absolute** | ❌ | Nearest **positioned** ancestor (not static) | Badges, dropdowns, tooltips |
| **fixed** | ❌ | The viewport | Sticky headers, chat buttons |
| **sticky** | ✅ until threshold | Scroll container | Table headers, section titles |

> [!warning] Gotchas
> - `absolute` with no positioned ancestor → positions relative to the page. Fix: `position: relative` on the parent.
> - `fixed` breaks if an ancestor has `transform` — it then behaves like absolute to that ancestor.
> - `sticky` fails if a parent has `overflow: hidden` or you forgot `top: 0`.

---

## 4. z-index & stacking contexts

`z-index` only works on positioned elements (or flex/grid children). Higher = on top — **but only compared with siblings in the same stacking context**.

A **stacking context** is a sealed layer group. Created by: `position` + `z-index` (not auto), `opacity < 1`, `transform`, `filter`, `position: fixed/sticky`, `isolation: isolate`.

```html
<header style="position:relative; z-index:2">…</header>
<main style="position:relative; z-index:1">
  <div class="modal" style="position:fixed; z-index:9999">…</div>
</main>
```

The modal's 9999 only counts **inside `main`**, and `main` (1) is below `header` (2). So the modal sits under the header. Fix: render the modal at the end of `<body>` (a **portal** in React).

> [!failure] Interview traps
> - Thinking `z-index: 9999` always wins.
> - Using `z-index` on a `position: static` element.
> - Fighting specificity with `!important` instead of fixing selectors.

---

## Summary

- Specificity: **inline > ID > class/attr/pseudo-class > element**; tie → later wins.
- `absolute` looks for the nearest positioned ancestor.
- `opacity`, `transform`, and positioned + z-index create new stacking contexts.
- Modals and tooltips go in a portal at the body level.

> [!success] Key takeaway
> When CSS "doesn't apply", check specificity; when z-index "doesn't work", find the parent that created a stacking context.
