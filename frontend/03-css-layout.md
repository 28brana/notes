# 03 · CSS Layout — Box Model, Flexbox & Grid

> [!info] Topic info
> **Subject:** Frontend · **Part:** JavaScript, HTML/CSS & the Browser · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Frontend](README.md) · [Interview questions →](questions/javascript-and-browser-qa.md)

> [!abstract] In one line
> Every element is a **box** (content + padding + border + margin); **Flexbox** arranges boxes in **one dimension** (a row or a column), **Grid** in **two** (rows and columns together).

---

## 1. The problem

You set a card to `width: 300px`, add `padding: 20px` and a `2px` border — and it renders **344px** wide, breaking your 3-column row. To lay things out with confidence you need the box model first, then the right layout tool.

---

## 2. The box model

From inside out: **content → padding → border → margin**.

```css
.card { width: 300px; padding: 20px; border: 2px solid; }
/* content-box (default): real width = 300 + 40 + 4 = 344px */

*, *::before, *::after { box-sizing: border-box; }
/* border-box: real width = 300px; content shrinks to 256px */
```

Almost every project sets `border-box` globally — widths then mean what you expect.

> [!note] Margin collapse
> Vertical margins of block elements **collapse**: a `20px` bottom margin above a `30px` top margin gives a **30px** gap, not 50. It doesn't happen inside flex or grid containers.

**Display types:** `block` (full width, new line), `inline` (flows in text, ignores width/height), `inline-block` (in-line but accepts width/height), `none` (removed).

---

## 3. Flexbox — one direction

```css
.nav { display: flex; justify-content: space-between; align-items: center; gap: 16px; }
.nav .search { flex: 1; }   /* take the remaining space */
```

- **Main axis** = `flex-direction` (row by default). `justify-content` aligns along it.
- **Cross axis** = the other one. `align-items` aligns along it.
- `flex: 1` = `flex-grow: 1; flex-shrink: 1; flex-basis: 0` — share leftover space.
- `flex-wrap: wrap` lets items go onto new lines.

**Classic: center anything** → `display: flex; justify-content: center; align-items: center;`

---

## 4. Grid — two directions

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 16px;
}
```

That one rule makes a **responsive gallery** with no media queries: as many 200px+ columns as fit. `fr` = a fraction of the free space. `grid-template-areas` lets you name regions (header, sidebar, main) for page layouts.

| | **Flexbox** | **Grid** |
|---|---|---|
| **Dimensions** | 1D — row *or* column | 2D — rows *and* columns |
| **Driven by** | Content size (content-out) | The layout you define (layout-in) |
| **Best for** | Navbars, toolbars, centering, button groups | Page layouts, galleries, dashboards |
| **Alignment across rows** | ❌ Each line is independent | ✅ Columns line up |

> [!tip] 🎤 Say it like this
> "I use Flexbox for one-dimensional things like a navbar or centering a button row, and Grid for two-dimensional layouts like a page shell or a card gallery where columns must line up. They work together — a Grid page with Flexbox inside each card. And I always set `box-sizing: border-box` so padding doesn't break widths."

**Responsive basics:** mobile-first media queries (`@media (min-width: 768px)`), relative units (`rem`, `%`, `vw`), and `<meta name="viewport" content="width=device-width, initial-scale=1">`.

> [!failure] Interview traps
> - Forgetting padding + border add to width in `content-box`.
> - Saying "Grid replaces Flexbox" — they solve different problems.
> - Setting width/height on an `inline` element and wondering why nothing happens.
> - Using `px` for font sizes everywhere — `rem` respects user zoom settings.

---

## Summary

- Box = content + padding + border + margin; use `border-box`.
- Vertical margins collapse (not in flex/grid).
- **Flexbox** = 1D, content-driven. **Grid** = 2D, layout-driven.
- `repeat(auto-fill, minmax(200px, 1fr))` = responsive grid without media queries.

> [!success] Key takeaway
> Box model for sizing, Flexbox for lines of things, Grid for the whole page — and combine them freely.
