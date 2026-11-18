# 14 · React Fundamentals

> [!info] Topic info
> **Subject:** Frontend · **Part:** React & Performance · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Frontend](README.md) · [Interview questions →](questions/react-and-performance-qa.md)

> [!abstract] In one line
> React builds UIs from **components** — functions that turn **props and state into UI** — and **re-renders** them when data changes, using a **virtual DOM** and **reconciliation** to update only what changed.

---

## 1. The problem

A shop page shows a cart count in the header, the sidebar and the checkout button. With plain JS you must remember to update all three DOM nodes every time the cart changes — miss one and the UI lies.

React flips it: you describe **what the UI should look like for this data**, and React figures out the DOM changes. `UI = f(state)`.

---

## 2. Components, JSX, props and state

A **component** is a function that returns **JSX** (HTML-like syntax that compiles to `React.createElement` / `jsx()` calls).

```jsx
function CartButton({ count }) {          // props come in from the parent
  const [open, setOpen] = useState(false); // state lives inside
  return <button onClick={() => setOpen(!open)}>Cart ({count})</button>;
}
```

| | **Props** | **State** |
|---|---|---|
| **Owned by** | Parent | The component itself |
| **Changeable here?** | ❌ Read-only | ✅ Via setter (`setOpen`) |
| **Triggers re-render?** | Yes, when parent passes new ones | Yes, when set to a new value |

**One-way data flow:** data goes **down** via props; changes go **up** via callbacks (`onAdd={...}`). This makes bugs easy to trace — you always know who owns the data.

---

## 3. Virtual DOM and reconciliation

The real DOM is slow to touch. On each render React builds a cheap **JS object tree** (the virtual DOM), **diffs** it with the previous one, and applies only the minimal real DOM changes. That diff is called **reconciliation**.

```mermaid
flowchart LR
    S[🔄 State changes] --> R[⚙️ Component re-runs]
    R --> V[🌳 New virtual tree]
    V -- diff with old tree --> D[📋 List of changes]
    D -- commit --> DOM[🖥️ Real DOM updated]
```

The diff uses two shortcuts to stay fast (O(n), not O(n³)):
- **Different element type** (`<div>` → `<section>`, `<A/>` → `<B/>`) → throw away the subtree and rebuild it.
- **Lists** are matched by **`key`**.

### Why keys matter

`key` tells React "this is the same item as last time". With `key={index}`, deleting the first row shifts every index — React reuses the wrong DOM nodes, so input text and checkbox state jump to the wrong row.

> [!important] Golden rule
> Use a **stable, unique ID** from the data as `key` (`key={todo.id}`). Index is only OK for static lists that never reorder.

> [!note] Render vs commit
> **Render phase** = calling your components and diffing (pure, can be paused in React 18). **Commit phase** = touching the DOM and running effects. "Re-render" does not mean the DOM changed.

---

## 4. Controlled vs uncontrolled components

- **Controlled:** React state is the source of truth — `<input value={name} onChange={e => setName(e.target.value)} />`. Easy to validate and format on every keystroke.
- **Uncontrolled:** the DOM keeps the value; you read it with a `ref` when needed. Less code, fewer re-renders (used by react-hook-form).

> [!tip] 🎤 Say it like this
> "React components are functions of props and state. When state changes, React re-runs the component, builds a new virtual tree, diffs it against the old one, and commits only the changes to the real DOM. Keys let it match list items correctly, so I always use stable IDs, not indexes."

> [!failure] Interview traps
> - Saying "virtual DOM is faster than the DOM" — it's a way to **batch and minimise** DOM work, not magic speed.
> - Mutating state (`items.push(x); setItems(items)`) — same reference, so React skips the re-render. Always create a new array/object.
> - Using `key={Math.random()}` — remounts every item on every render.
> - Forgetting React 18 **batches** multiple `setState` calls into one render, even inside timeouts and promises.

---

## Summary

- Components = functions of **props** (from parent, read-only) and **state** (own, changeable).
- **One-way data flow**: data down, events up.
- **Virtual DOM + reconciliation** = diff and apply minimal DOM changes.
- **Keys** must be stable and unique.
- **Controlled** = React owns the value; **uncontrolled** = DOM owns it.

> [!success] Key takeaway
> Describe the UI for the current data and let React do the DOM work — just never mutate state and always give lists stable keys.
