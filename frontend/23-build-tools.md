# 23 · Build Tools (Vite, webpack & bundling)

> [!info] Topic info
> **Subject:** Frontend · **Part:** React & Performance · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Frontend](README.md) · [Interview questions →](questions/react-and-performance-qa.md)

> [!abstract] In one line
> Build tools turn our **JSX/TypeScript modules** into **small, optimized files** browsers can load — **bundling**, **transpiling**, **tree shaking**, **minifying**, **code splitting** and adding **content hashes** — with **Vite** and **webpack** as the main options.

---

## 1. The problem

Our app has 800 source files in TypeScript and JSX, plus npm packages. Browsers don't understand JSX or TS, 800 separate requests would be slow, and we don't want to ship the 90% of lodash we never use.

---

## 2. What a production build does

```mermaid
flowchart LR
    S[📝 TS and JSX source] --> T[⚙️ Transpile]
    T --> G[🌳 Build module graph]
    G --> TS[✂️ Tree shake]
    TS --> SP[📦 Split into chunks]
    SP --> M[🗜️ Minify + hash names]
    M --> O[🚀 dist folder for CDN]
```

- **Transpile** — JSX/TS → plain JS for target browsers (esbuild, SWC, Babel).
- **Bundle** — follow `import`s from the entry file and combine modules into a few files.
- **Tree shaking** — drop exports nobody imports. Needs **ES modules** (`import/export`, statically analysable) — not CommonJS `require`.
- **Code splitting** — dynamic `import()` creates separate chunks loaded on demand.
- **Minification** — remove whitespace, shorten names (Terser, esbuild): often 60–70% smaller before gzip.
- **Content hashing** — `app.3f9a1c.js`, so files can be cached forever ([web performance](22-web-performance.md)).
- **Source maps** — map minified code back to the original lines for debugging and error tracking (Sentry). Upload them privately rather than exposing them publicly.

---

## 3. Vite vs webpack

| | **Vite** | **webpack** |
|---|---|---|
| **Dev server** | Serves **native ES modules**, transforms files on demand → starts in ~1 s | Bundles the whole app first → slow start on big apps |
| **Hot reload (HMR)** | Very fast, only the changed module | Slower as the app grows |
| **Production build** | Rollup (Rolldown in newer versions) | webpack itself |
| **Config** | Minimal, sensible defaults | Powerful but verbose (loaders, plugins) |
| **Use when** | New projects (default today) | Large legacy apps, Module Federation |

Others to name: **esbuild** and **SWC** (very fast transpilers written in Go/Rust), **Turbopack** (Next.js), **Rspack** (webpack-compatible, in Rust).

> [!tip] 🎤 Say it like this
> "A build tool transpiles TS and JSX, bundles modules, tree-shakes unused exports, splits code into lazy chunks, minifies and adds content hashes for caching. Vite is the modern default because its dev server serves native ES modules and starts almost instantly, while production builds still bundle and optimize. webpack is still common in older large apps."

---

## 4. Keeping bundles small

- Analyse with a **bundle analyzer** (`rollup-plugin-visualizer`, `webpack-bundle-analyzer`).
- Import only what you need (`import debounce from 'lodash-es/debounce'`), prefer ESM packages.
- Mark packages side-effect free (`"sideEffects": false`) so tree shaking works.
- Split vendor code and lazy-load heavy libs (charts, editors) only where used.
- Set a **bundle size budget** in CI.

> [!failure] Interview traps
> - Thinking tree shaking works with CommonJS — it mostly doesn't.
> - Saying Vite doesn't bundle — it **does** for production.
> - Exposing source maps publicly by accident (leaks source code).
> - Confusing **transpiling** (syntax conversion) with **polyfilling** (adding missing APIs at runtime).

---

## Summary

- Build = transpile → bundle → tree shake → split → minify → hash.
- **Tree shaking needs ES modules.**
- **Vite** = fast dev via native ESM + Rollup builds; **webpack** = mature, configurable.
- Source maps for debugging; hashes for caching.

> [!success] Key takeaway
> The build step is where bundle size is won or lost — know what each stage does and measure with a bundle analyzer.
