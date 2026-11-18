# 02 · Semantic HTML & Accessibility

> [!info] Topic info
> **Subject:** Frontend · **Part:** JavaScript, HTML/CSS & the Browser · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Frontend](README.md) · [Interview questions →](questions/javascript-and-browser-qa.md)

> [!abstract] In one line
> **Semantic HTML** uses tags that describe meaning (`<nav>`, `<button>`, `<main>`) so browsers, **screen readers** and search engines understand the page; **accessibility (a11y)** means everyone — including keyboard and screen-reader users — can use it.

---

## 1. The problem

A dev builds a "button" with `<div onclick="buy()">`. It looks fine. But a blind user's screen reader says nothing useful, and a keyboard user can't Tab to it or press Enter.

Roughly **1 in 6 people** has some disability. Many countries also make a11y a legal requirement.

---

## 2. Semantic tags

```html
<header>…logo, top nav…</header>
<nav>…links…</nav>
<main>
  <article><h1>Post title</h1>…</article>
  <aside>…related posts…</aside>
</main>
<footer>…</footer>
```

**Why it matters:**
- **Screen readers** let users jump between landmarks (`nav`, `main`) and headings.
- **SEO** — search engines understand structure.
- **Free behaviour** — `<button>` is focusable, works with Enter/Space, and submits forms. A `<div>` gives you none of that.

> [!important] Golden rule
> Use the **native element** first (`<button>`, `<a href>`, `<input>`, `<select>`). Only reach for ARIA when no native element fits.

---

## 3. Accessibility basics checklist

- **`alt` text on images** — `alt="Red running shoes, side view"`. Decorative images get `alt=""` so readers skip them.
- **Labels on inputs** — `<label for="email">Email</label><input id="email">`. A placeholder is **not** a label.
- **Headings in order** — one `h1`, then `h2`, `h3`… don't skip levels for styling.
- **Keyboard** — everything clickable must be reachable with Tab and have a visible **focus ring**. Never `outline: none` without a replacement.
- **Colour contrast** — at least **4.5:1** for normal text (WCAG AA). Don't use colour alone for errors.
- **Links vs buttons** — `<a>` navigates somewhere; `<button>` does an action.

---

## 4. ARIA (when native HTML isn't enough)

**ARIA** (Accessible Rich Internet Applications) = attributes that tell assistive tech what a custom widget is and what state it's in.

```html
<button aria-expanded="false" aria-controls="menu">Menu</button>
<ul id="menu" hidden>…</ul>

<button aria-label="Close dialog">✕</button>
<div role="alert">Payment failed</div>  <!-- read out immediately -->
```

- **`role`** — what it is (`dialog`, `tab`, `alert`).
- **`aria-label` / `aria-labelledby`** — the accessible name when there's no visible text.
- **State** — `aria-expanded`, `aria-checked`, `aria-hidden="true"` (hide from readers).
- **`aria-live`** — announce dynamic updates (toasts, form errors).

> [!tip] 🎤 Say it like this
> "I start with semantic HTML — real buttons, labels, landmarks and alt text — because it gives keyboard and screen-reader support for free. ARIA is only for custom widgets like tabs or modals, to describe role and state. And I test by tabbing through the page and running Lighthouse or axe."

> [!failure] Interview traps
> - Using `<div>`/`<span>` with click handlers instead of `<button>`.
> - Adding `role="button"` to a div but forgetting `tabindex="0"` and Enter/Space handling.
> - Sprinkling ARIA everywhere — wrong ARIA misleads screen readers; no ARIA is better than bad ARIA.
> - Treating placeholder as a label.

---

## Summary

- Semantic tags give meaning → better a11y and SEO.
- Native elements first; ARIA only for custom widgets.
- Checklist: **alt, labels, headings, keyboard + focus, contrast**.
- Test with keyboard only, Lighthouse/axe, and a screen reader.

> [!success] Key takeaway
> If it can't be used with just a keyboard and a screen reader, it's not done — and semantic HTML gets you 80% of the way for free.
