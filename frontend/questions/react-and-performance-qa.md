# Q&A · Frontend — React & Performance

[← Back to Frontend](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 14 · React Fundamentals

[Read the notes →](../14-react-fundamentals.md)

> [!question]- 🟢 Q1. What is the difference between props and state?
> Props are inputs passed from the parent and are read-only inside the component. State is data the component owns and changes with its setter. Changing either causes a re-render. Data flows down through props, and children tell parents about changes through callback props.

> [!question]- 🟢 Q2. What is the virtual DOM and what is reconciliation?
> The virtual DOM is a lightweight JS object tree describing the UI. On every render React builds a new tree, diffs it with the previous one — that's reconciliation — and applies only the minimal changes to the real DOM. It's about batching and minimising DOM work, not about being magically faster than the DOM.

> [!question]- 🟡 Q3. Why do keys matter in lists, and why is the index a bad key?
> Keys tell React which item is which between renders. If you use the index and an item is removed or reordered, indexes shift, so React reuses the wrong DOM nodes and component state — typed text or checkboxes appear on the wrong row. Use a stable unique ID from the data.

> [!question]- 🔴 Q4. A teammate pushes an item into the state array and calls setItems, but the UI doesn't update. Why?
> They mutated the existing array, so the reference didn't change and React's `Object.is` check thinks nothing changed, skipping the re-render. State must be treated as immutable: `setItems(prev => [...prev, item])`. Mutation also breaks memoization and React.memo children.

---

## 15 · Hooks

[Read the notes →](../15-hooks.md)

> [!question]- 🟢 Q5. Explain the useEffect dependency array and cleanup function.
> With no array the effect runs after every render; with `[]` it runs once after mount; with `[a, b]` it runs when those values change. The returned cleanup runs before the next effect run and on unmount — used to clear timers, unsubscribe, or abort fetches.

> [!question]- 🟡 Q6. What is a stale closure in hooks and how do you fix it?
> Each render's functions capture that render's props and state. If an effect or interval uses `count` but `count` isn't in the deps, it keeps seeing the old value forever. Fix by adding it to the deps, using the updater form `setCount(c => c + 1)`, or storing the latest value in a ref.

> [!question]- 🟡 Q7. What are the rules of hooks and why do they exist?
> Call hooks only at the top level — not in conditions, loops or nested functions — and only from components or custom hooks. React stores hook state in a list per component and matches it by call order, so a conditional hook shifts every later hook to the wrong slot.

> [!question]- 🟡 Q8. useMemo vs useCallback vs useRef?
> useMemo caches a computed value until its deps change. useCallback caches a function reference — handy when passing it to a memoized child or using it as an effect dependency. useRef holds a mutable value that survives renders without triggering a re-render, and gives access to DOM nodes.

> [!question]- 🔴 Q9. Your component fetches data in useEffect and the network tab shows an infinite loop of requests. What's wrong?
> Likely a dependency that changes every render — an object, array or function created inline — or the effect sets state that's also in its own deps. Fix by depending on primitive values (`userId`), memoizing the object/function, or moving it inside the effect. Better still, use React Query for fetching.

---

## 16 · State Management

[Read the notes →](../16-state-management.md)

> [!question]- 🟢 Q10. What is prop drilling and how do you avoid it?
> Prop drilling is passing data through many layers of components that don't use it, just to reach a deep child. Fixes are component composition (pass `children` or elements), Context for stable app-wide values, or a store like Zustand or Redux for shared changing state.

> [!question]- 🟡 Q11. Why isn't Context a full replacement for Redux or Zustand?
> When a Context value changes, every consumer re-renders, even if it uses an unrelated field, and there are no selectors or devtools. It's great for rarely changing data like theme and auth user. For frequently changing shared state, stores with selectors only re-render the components that read the changed slice.

> [!question]- 🟡 Q12. Why use React Query instead of storing API data in Redux?
> API data is server state: it's owned elsewhere, can go stale, and needs caching, deduping, refetching and retries. React Query does all that out of the box keyed by query; doing it in Redux means hand-writing loading flags, cache invalidation and sync logic. The client store then only holds true client state.

> [!question]- 🔴 Q13. You're starting a large app with 15 frontend devs. How do you decide state management?
> I classify state: local UI state stays in components, server data goes to React Query, filters and pagination go in the URL, theme and auth in Context. For remaining shared client state, I'd pick Redux Toolkit for a big team because its structure and devtools help consistency — Zustand if the team prefers something lighter.

---

## 17 · React Performance

[Read the notes →](../17-react-performance.md)

> [!question]- 🟢 Q14. What causes a React component to re-render?
> Its own state changes, its parent re-renders, or a Context it reads changes. By default a parent re-render re-renders all children, even with identical props, unless the child is wrapped in React.memo.

> [!question]- 🟡 Q15. Why does React.memo sometimes not prevent re-renders?
> React.memo compares props shallowly. If the parent passes a new inline function, object or array each render, the reference changes and memo always misses. Stabilise those props with useCallback and useMemo, or restructure so they don't need to be passed.

> [!question]- 🟡 Q16. How would you render a list of 10,000 items smoothly?
> Virtualize it with react-window or TanStack Virtual so only the ~30 visible rows exist in the DOM. Give rows stable keys, memoize the row component, and paginate or infinite-load from the server instead of fetching all 10,000 at once.

> [!question]- 🔴 Q17. Typing in a search box above a big table lags badly. Walk me through fixing it.
> First profile with React DevTools to confirm the table re-renders on every key. Then move the input state down so only the search box updates per keystroke, debounce the filter, and use useTransition or useDeferredValue so filtering is low priority. Finally memoize rows and virtualize the table.

---

## 18 · Forms

[Read the notes →](../18-forms.md)

> [!question]- 🟢 Q18. Controlled vs uncontrolled inputs — when would you use each?
> Controlled inputs keep the value in React state, good for instant validation, formatting and dependent fields. Uncontrolled inputs leave the value in the DOM and read it via ref or FormData on submit — simpler and fewer re-renders, which is why react-hook-form uses them.

> [!question]- 🟡 Q19. How do you handle validation in a large React form?
> I use react-hook-form with a Zod schema via a resolver, so validation rules and TypeScript types come from one place. Errors show on blur and submit, inputs are linked to errors with aria-describedby, the submit button is disabled while submitting, and the server validates again.

> [!question]- 🔴 Q20. Users report the order form sometimes creates two orders. What do you check?
> Probably double submit: double-clicks or Enter plus click before the first request finishes. Disable the button and ignore submits while `isSubmitting`, and on the backend use an idempotency key so retries don't create duplicates.

---

## 19 · Routing

[Read the notes →](../19-routing.md)

> [!question]- 🟢 Q21. How does client-side routing work?
> The router intercepts link clicks, changes the URL with `history.pushState` without requesting a new page, and renders the component matching the path. It listens to `popstate` so Back and Forward work. The page never fully reloads, so JS state is kept.

> [!question]- 🟡 Q22. How do you implement protected routes, and are they secure?
> Wrap private routes in a guard that checks the auth state and redirects to login with `<Navigate>`, remembering the original location. They are only UX — anyone can bypass client code — so the API must verify the token and permissions on every request.

> [!question]- 🔴 Q23. The SPA works when you click around, but refreshing on /products/42 gives a 404. Why?
> On refresh the browser asks the server for `/products/42`, and the server has no such file. The server or CDN must be configured to fall back to `index.html` for unknown paths so the client router can handle it.

---

## 20 · Data Fetching

[Read the notes →](../20-data-fetching.md)

> [!question]- 🟢 Q24. What states should every data-fetching UI handle?
> Loading, error with a retry option, empty, and success. Also remember `fetch` doesn't throw on 404 or 500, so check `response.ok` and treat those as errors.

> [!question]- 🟡 Q25. What is a race condition in data fetching and how do you prevent it?
> When params change quickly, an older slow request can resolve after a newer one and overwrite fresh data. Prevent it by aborting the previous request with AbortController in the effect cleanup, or ignoring stale responses with a flag. React Query handles it automatically by keying data per query.

> [!question]- 🟡 Q26. What is an optimistic update?
> You update the UI immediately as if the request succeeded — like incrementing a like count — then send the request. If it fails you roll back to the previous value and show an error. It makes the app feel instant on slow networks.

> [!question]- 🔴 Q27. A page shows spinners in sequence and takes 3 seconds even though each API call is fast. What's happening?
> A request waterfall: the parent fetches, renders a child, which then fetches, and so on. Fix by fetching independent data in parallel at the route level, prefetching, or fetching on the server with SSR or Server Components so the client receives data in one go.

---

## 21 · Rendering Strategies

[Read the notes →](../21-rendering-strategies.md)

> [!question]- 🟢 Q28. What's the difference between CSR, SSR and SSG?
> CSR builds the page in the browser after JS loads — slow first paint, weak SEO. SSR renders HTML on the server per request — fast first paint, fresh data, but server cost. SSG builds HTML once at build time and serves it from a CDN — fastest, but data only updates on rebuild.

> [!question]- 🟡 Q29. What is hydration and what causes a hydration mismatch?
> Hydration is React attaching event handlers and state to server-rendered HTML so it becomes interactive. A mismatch happens when the client's first render differs from the server HTML — for example using `Date.now()`, random values, or `window` checks during render.

> [!question]- 🟡 Q30. How are React Server Components different from SSR?
> SSR renders client components to HTML once, then ships their JS to hydrate. Server Components run only on the server, can read databases directly, and send no JS to the browser at all. In Next.js App Router they're the default, with `'use client'` marking interactive parts.

> [!question]- 🔴 Q31. Choose a rendering strategy for an e-commerce site: home, product pages, cart, and admin.
> Home and marketing pages: SSG on a CDN. Product pages: ISR so they're static and SEO-friendly but refresh prices every few minutes. Cart and checkout: SSR or client rendering since it's personal. Admin dashboard behind login: CSR is fine since SEO doesn't matter.

---

## 22 · Web Performance

[Read the notes →](../22-web-performance.md)

> [!question]- 🟢 Q32. What are the Core Web Vitals and their good thresholds?
> LCP measures loading — when the largest visible element appears — good is 2.5 s or less. INP measures responsiveness — the delay from an interaction to the next paint — good is 200 ms or less. CLS measures layout shifts — good is 0.1 or less. They're judged at the 75th percentile of real users.

> [!question]- 🟡 Q33. How do you improve LCP?
> Reduce TTFB with a CDN and SSR or static pages, optimize the hero image with modern formats, correct sizes and `fetchpriority="high"`, preload it and key fonts, and remove render-blocking CSS and JS. Never lazy-load the LCP image.

> [!question]- 🟡 Q34. What caching headers would you set for a React app's files?
> Hashed JS and CSS files get `Cache-Control: public, max-age=31536000, immutable` since a new build means new filenames. `index.html` gets `no-cache` so the browser revalidates each time and picks up the new asset names after a deploy.

> [!question]- 🔴 Q35. Your page has a CLS of 0.35 and poor INP on mobile. What do you do?
> For CLS, find the shifting elements in DevTools — usually images without dimensions, late ads or banners, or font swaps — and reserve space with width/height or aspect-ratio. For INP, profile long tasks: split heavy JS, cut re-renders, defer third-party scripts and move work off the main thread.

---

## 23 · Build Tools

[Read the notes →](../23-build-tools.md)

> [!question]- 🟢 Q36. What is tree shaking and what does it need?
> Tree shaking removes exports that nothing imports, shrinking the bundle. It needs ES modules because static `import`/`export` lets the bundler see what's used at build time; CommonJS `require` is dynamic, so it mostly can't be shaken. Packages should also be marked side-effect free.

> [!question]- 🟡 Q37. Why is Vite's dev server so much faster than webpack's?
> Vite doesn't bundle in development: it serves native ES modules and transforms each file on demand with esbuild, so startup is near-instant and HMR only touches the changed module. webpack bundles the whole app first. For production Vite still bundles and optimizes with Rollup.

> [!question]- 🔴 Q38. The main bundle grew to 2 MB. How do you investigate and fix it?
> Run a bundle analyzer to find the biggest modules. Usual fixes: lazy-load routes and heavy libraries like charts or editors, replace or import only parts of big libraries, check tree shaking isn't broken by CommonJS packages, and add a bundle size budget in CI so it doesn't creep back.

---

## 24 · Frontend Testing

[Read the notes →](../24-frontend-testing.md)

> [!question]- 🟢 Q39. What is the philosophy of React Testing Library?
> Test components the way users use them: find elements by role, label or text, interact with userEvent, and assert on what's visible. Avoid testing internal state or implementation details, so tests survive refactors and catch real bugs.

> [!question]- 🟡 Q40. Unit vs integration vs E2E tests — how do you balance them on the frontend?
> Static checks like TypeScript and ESLint as the base, unit tests for tricky pure logic, most effort on integration tests with RTL and MSW for API mocking, and a few Playwright E2E tests for critical flows like login and checkout, since E2E is slow and more flaky.

> [!question]- 🔴 Q41. After a refactor, half the test suite fails but the app works. What does that tell you?
> The tests are coupled to implementation details — internal state, component names, class names or test IDs — instead of behaviour. I'd rewrite them to query by role and text and assert on user-visible outcomes, so they only fail when behaviour actually breaks.

---

## 25 · Frontend System Design

[Read the notes →](../25-frontend-system-design.md)

> [!question]- 🟢 Q42. What is the RADIO framework?
> A structure for frontend system design: Requirements, Architecture, Data model, Interface or API, and Optimizations. It keeps you from jumping into code and makes sure you cover components, state, API shape, performance and accessibility in 45 minutes.

> [!question]- 🟡 Q43. Why use cursor pagination instead of offset for a news feed?
> In a live feed new posts are added at the top, so offsets shift and the next page returns duplicates or skips items. A cursor points at the last item seen, so the next page always continues from the right place, and it's also faster for the database.

> [!question]- 🔴 Q44. Design an autocomplete search box. What are the key points?
> Debounce input around 300 ms and require a minimum length. Cache results per query, and cancel or ignore stale requests so old responses can't overwrite new ones. Limit results, show loading and empty states, support arrow keys, Enter and Escape, and use ARIA combobox and listbox roles for accessibility.

> [!question]- 🔴 Q45. Design an infinite-scroll news feed that stays fast after 1,000 posts.
> Use cursor pagination with IntersectionObserver to load the next page early, and virtualize the list so only visible posts are in the DOM. Lazy-load images with fixed aspect ratios to avoid layout shift, normalize data in a query cache, use optimistic likes, show a "new posts" pill instead of jumping scroll, and SSR the first page for fast LCP.
