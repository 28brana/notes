# 22 · Web Performance & Core Web Vitals

> [!info] Topic info
> **Subject:** Frontend · **Part:** React & Performance · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Frontend](README.md) · [Interview questions →](questions/react-and-performance-qa.md)

> [!abstract] In one line
> Web performance is measured by Google's **Core Web Vitals** — **LCP** (loading), **INP** (responsiveness) and **CLS** (visual stability) — and improved by **shipping less**, **loading smarter** (lazy loading, image optimization, code splitting) and **caching well**.

---

## 1. The problem

A store's product page takes 4.5 s to show the main image on a mid-range phone over 4G. Buttons feel sticky, and the "Buy" button jumps down when an ad loads. Users leave, and Google ranks the page lower.

Rule of thumb: every extra second of load time costs a noticeable share of conversions.

---

## 2. Core Web Vitals

| Metric | Measures | Good | Usual culprits |
|---|---|---|---|
| **LCP** — Largest Contentful Paint | When the **biggest visible element** (hero image, headline) appears | ≤ **2.5 s** | Slow server, big images, render-blocking CSS/JS, CSR |
| **INP** — Interaction to Next Paint | Delay from click/tap/key to the **next frame** (worst-ish interaction) | ≤ **200 ms** | Long JS tasks, heavy re-renders, big hydration |
| **CLS** — Cumulative Layout Shift | How much content **jumps** while loading | ≤ **0.1** | Images without size, late ads/banners, web fonts |

INP replaced **FID** in March 2024. Supporting metrics: **TTFB** (time to first byte) and **FCP** (first contentful paint).

> [!note] Lab vs field data
> **Lab** = Lighthouse / DevTools on your machine (good for debugging). **Field** = real users (CrUX, `web-vitals` library, RUM tools) — this is what Google ranks on. Judge by the **75th percentile** of real users.

---

## 3. Fixing each one

### LCP — get the main content on screen fast
- Fast **TTFB**: CDN, caching, SSR/SSG instead of CSR ([rendering strategies](21-rendering-strategies.md)).
- **Optimize the hero image**: AVIF/WebP, right size via `srcset`, `fetchpriority="high"`, and **never lazy-load it**.
- `<link rel="preload">` for the LCP image and key fonts; inline critical CSS; `defer` scripts.

### INP — keep the main thread free
- Break **long tasks** (> 50 ms) into chunks; yield to the browser.
- Ship less JS: code splitting, drop heavy libraries, move work to a **Web Worker**.
- In React: fewer re-renders, `useTransition` for heavy updates ([React performance](17-react-performance.md)).

### CLS — reserve space
- Always set `width`/`height` or `aspect-ratio` on images, videos, iframes.
- Reserve slots for ads/embeds; insert banners **below** the fold or push via overlay.
- Fonts: `font-display: swap` + preload, or `size-adjust` fallback fonts.

---

## 4. Loading smarter

```html
<img src="hero.avif" width="1200" height="600" fetchpriority="high" alt="Sale">
<img src="card.webp" loading="lazy" width="300" height="300" alt="Shoes">
<script src="analytics.js" defer></script>
```

- **Lazy load** below-the-fold images (`loading="lazy"`) and components (`React.lazy`).
- **Code split** by route; **tree shake** unused code ([build tools](23-build-tools.md)).
- **Compress** with Brotli/gzip; serve over HTTP/2 or HTTP/3.
- `preconnect` to critical third-party origins; audit third-party scripts — they are often the worst offenders.

### Caching headers

```mermaid
flowchart LR
    B[🌐 Browser] --> HTML[📄 index.html no-cache]
    HTML --> JS[📦 app.3f9a1c.js]
    JS -- immutable 1 year --> C[⚡ Browser and CDN cache]
```

- **Hashed assets** (`app.3f9a1c.js`): `Cache-Control: public, max-age=31536000, immutable` — new deploy = new filename.
- **HTML**: `no-cache` (revalidate each time with ETag) so users get the new asset names.
- See [CDN](../system-design/caching/05-cdn.md) and [HTTP](../computer-networks/02-http-and-https.md).

> [!tip] 🎤 Say it like this
> "I measure with field data first — Core Web Vitals at the 75th percentile. For LCP I speed up TTFB with a CDN and SSR or static pages and prioritize the hero image. For INP I cut long JavaScript tasks and unnecessary re-renders. For CLS I reserve space for images, ads and fonts. Then I add code splitting, lazy loading and long-lived caching for hashed assets."

> [!failure] Interview traps
> - Lazy-loading the hero (LCP) image — makes LCP worse.
> - Trusting a Lighthouse score of 100 on a MacBook while real users on cheap Android phones suffer.
> - Long cache on `index.html` — users stuck on an old version after deploy.
> - Ignoring third-party scripts (chat widgets, tag managers).

---

## Summary

- **LCP ≤ 2.5 s**, **INP ≤ 200 ms**, **CLS ≤ 0.1** at p75 of real users.
- LCP → fast server + prioritized hero image. INP → less main-thread JS. CLS → reserve space.
- Lazy load, code split, compress, cache hashed assets forever.

> [!success] Key takeaway
> Measure real users, fix the metric that's failing with its specific fixes, and above all ship less JavaScript.
