# 24 · Frontend Testing

> [!info] Topic info
> **Subject:** Frontend · **Part:** React & Performance · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Frontend](README.md) · [Interview questions →](questions/react-and-performance-qa.md)

> [!abstract] In one line
> Frontend testing mixes **unit tests** (Jest/Vitest), **component/integration tests** (**React Testing Library**) and **end-to-end tests** (**Playwright**/Cypress) — testing **behaviour the user sees**, not implementation details.

---

## 1. The problem

The team refactors the checkout form from `useState` to react-hook-form. Tests that checked internal state all break, even though the form works perfectly. Meanwhile a real bug — the Pay button disabled forever — slipped through.

Good tests break when **behaviour** breaks, not when the code is reorganized.

---

## 2. The layers

| Layer | Tool | Tests | Speed |
|---|---|---|---|
| **Unit** | Jest / **Vitest** | Pure functions, hooks, reducers (`formatPrice`, `cartReducer`) | ms |
| **Component / integration** | **React Testing Library** + Jest/Vitest (jsdom) | A component or small feature as a user uses it | ms–s |
| **End-to-end (E2E)** | **Playwright** / Cypress | Real browser, real app: login → add to cart → pay | seconds |
| **Visual / a11y** | Storybook, Chromatic, axe | Screenshots diff, accessibility rules | — |

> [!example] Testing trophy
> For frontends, the sweet spot is **mostly integration tests** with RTL, a base of static checks (TypeScript, ESLint), some unit tests for tricky logic, and a **few E2E tests for critical flows** (sign-up, checkout).

---

## 3. React Testing Library — test like a user

```jsx
test('shows error for invalid email', async () => {
  const user = userEvent.setup();
  render(<SignupForm />);
  await user.type(screen.getByLabelText(/email/i), 'not-an-email');
  await user.click(screen.getByRole('button', { name: /sign up/i }));
  expect(await screen.findByText(/valid email/i)).toBeInTheDocument();
});
```

- Query by **role, label, text** — what users and screen readers see. `data-testid` only as a last resort.
- `getBy` (must exist), `queryBy` (may not exist — for "not in document"), `findBy` (async, waits).
- Use `userEvent` over `fireEvent` — it simulates real typing and clicks.

**Mocking APIs:** use **MSW** (Mock Service Worker) to intercept `fetch` at the network level, so components use their real data-fetching code.

---

## 4. E2E with Playwright

Runs the real app in Chromium, Firefox and WebKit. Auto-waits for elements, supports parallel runs, traces and screenshots on failure. Keep E2E tests **few and focused** on money paths — they're slow and can be flaky.

> [!tip] 🎤 Say it like this
> "I follow the testing trophy: TypeScript and ESLint as the base, mostly integration tests with React Testing Library that interact like a user — by role and label — with MSW mocking the API, unit tests for complex logic, and a few Playwright E2E tests for critical flows like checkout. I avoid testing implementation details so refactors don't break tests."

> [!failure] Interview traps
> - Testing internal state or which hook was called — brittle.
> - Snapshot tests of whole components — huge diffs nobody reads.
> - Many E2E tests for every edge case — slow, flaky CI.
> - Using `waitFor` with fixed sleeps instead of `findBy` / auto-waiting.

---

## Summary

- **Unit** for logic, **RTL integration** for components, **Playwright** for critical journeys.
- Query by **role/label/text**; use **userEvent** and **MSW**.
- Test **behaviour**, not implementation.

> [!success] Key takeaway
> Write tests that resemble how users use the app — they catch real bugs and survive refactors.
