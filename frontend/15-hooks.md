# 15 · React Hooks

> [!info] Topic info
> **Subject:** Frontend · **Part:** React & Performance · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Frontend](README.md) · [Interview questions →](questions/react-and-performance-qa.md)

> [!abstract] In one line
> **Hooks** are functions like `useState` and `useEffect` that let function components **hold state, run side effects and reuse logic** — they must be called in the **same order on every render**.

---

## 1. The problem

Before hooks (2019), state and lifecycle needed **class components**: logic for one feature was split across `componentDidMount`, `componentDidUpdate` and `componentWillUnmount`, and sharing logic needed awkward HOCs (higher-order components) and render props.

Hooks put related logic together and let you share it as a plain function — a **custom hook**.

---

## 2. The core hooks

| Hook | Use it for | Re-renders on change? |
|---|---|---|
| **useState** | Simple local state | ✅ |
| **useReducer** | State with many related transitions (`dispatch({type})`) | ✅ |
| **useEffect** | Syncing with outside world: fetch, subscriptions, timers | — |
| **useRef** | Mutable box that survives renders; DOM node access | ❌ |
| **useMemo** | Cache an expensive **value** | — |
| **useCallback** | Cache a **function** reference | — |
| **useContext** | Read a value from a Context provider | ✅ when value changes |

Newer: `useTransition` / `useDeferredValue` (React 18, mark updates as low priority), `useId` (stable IDs for a11y), and in React 19 `use()`, `useActionState` and `useOptimistic`.

---

## 3. useEffect — the one everyone gets wrong

```jsx
useEffect(() => {
  const id = setInterval(() => setSeconds(s => s + 1), 1000);
  return () => clearInterval(id);   // cleanup: before next run + on unmount
}, []);                              // dependency array
```

- **No array** → runs after every render.
- **`[]`** → runs once after mount.
- **`[userId]`** → runs when `userId` changes. Cleanup of the old run happens first.

> [!warning] Stale closures
> Each render's effect "sees" the props and state **of that render**. If `count` is missing from deps, the interval keeps reading `count = 0` forever. Fix: add it to deps, or use the **updater form** `setCount(c => c + 1)`, or keep the latest value in a `ref`.

> [!note] You might not need an effect
> Don't use effects to compute values from props/state — just compute them during render. Effects are for **talking to things outside React**. In dev, Strict Mode runs effects **twice** on purpose to expose missing cleanups.

---

## 4. useRef, useMemo, useCallback

- **useRef** — `inputRef.current.focus()`, or store a timer ID / previous value without causing re-renders.
- **useMemo** — `const sorted = useMemo(() => sort(list), [list])` — skip heavy work when inputs didn't change.
- **useCallback** — `useCallback(fn, deps)` = `useMemo(() => fn, deps)`. Only useful when passing the function to a `React.memo` child or as an effect dependency.

> [!note] React Compiler
> The React Compiler (stable since 2025) auto-memoizes components and values, so hand-written `useMemo`/`useCallback` is becoming less needed in new codebases.

---

## 5. Custom hooks and rules of hooks

A **custom hook** is a function starting with `use` that calls other hooks — e.g. `useDebounce(value, 300)`, `useFetch(url)`. Each component calling it gets **its own state**.

> [!important] Rules of hooks
> 1. Call hooks only at the **top level** — never inside `if`, loops or nested functions.
> 2. Call them only from **components or custom hooks**.
>
> Why: React stores hook state in a **list per component, matched by call order**. Skip one in an `if` and every hook after it reads the wrong slot.

> [!tip] 🎤 Say it like this
> "Hooks give function components state and side effects. useEffect syncs with the outside world — its dependency array controls when it re-runs, and the cleanup function prevents leaks. Most effect bugs are stale closures from missing dependencies. Hooks must be called in the same order every render because React tracks them by position."

> [!failure] Interview traps
> - Making the effect callback `async` — it must return a cleanup or nothing, not a Promise. Define an async function inside instead.
> - Objects/functions in deps created every render → effect runs every render (infinite fetch loop).
> - Thinking `setState` updates the variable immediately — it schedules a re-render; the value changes next render.
> - Wrapping everything in `useMemo`/`useCallback` — memoization has a cost too.

---

## Summary

- `useState`/`useReducer` for state, `useEffect` for side effects, `useRef` for mutable non-render values.
- Effects: **deps array** + **cleanup**; beware **stale closures**.
- `useMemo` caches values, `useCallback` caches functions — use when it measurably helps.
- **Custom hooks** share logic, not state.

> [!success] Key takeaway
> Know useEffect deeply — deps, cleanup, stale closures and "you might not need an effect" — that's where interviewers dig.
