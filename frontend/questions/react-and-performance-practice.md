# Frontend — React & Performance · Practice Questions

[← Back to Frontend](../README.md) · [Q&A with answers →](react-and-performance-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](react-and-performance-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**45 questions.**

---

## 14 · React Fundamentals

- [ ] 🟢 Q1. What is the difference between props and state?
- [ ] 🟢 Q2. What is the virtual DOM and what is reconciliation?
- [ ] 🟡 Q3. Why do keys matter in lists, and why is the index a bad key?
- [ ] 🔴 Q4. A teammate pushes an item into the state array and calls setItems, but the UI doesn't update. Why?

---

## 15 · Hooks

- [ ] 🟢 Q5. Explain the useEffect dependency array and cleanup function.
- [ ] 🟡 Q6. What is a stale closure in hooks and how do you fix it?
- [ ] 🟡 Q7. What are the rules of hooks and why do they exist?
- [ ] 🟡 Q8. useMemo vs useCallback vs useRef?
- [ ] 🔴 Q9. Your component fetches data in useEffect and the network tab shows an infinite loop of requests. What's wrong?

---

## 16 · State Management

- [ ] 🟢 Q10. What is prop drilling and how do you avoid it?
- [ ] 🟡 Q11. Why isn't Context a full replacement for Redux or Zustand?
- [ ] 🟡 Q12. Why use React Query instead of storing API data in Redux?
- [ ] 🔴 Q13. You're starting a large app with 15 frontend devs. How do you decide state management?

---

## 17 · React Performance

- [ ] 🟢 Q14. What causes a React component to re-render?
- [ ] 🟡 Q15. Why does React.memo sometimes not prevent re-renders?
- [ ] 🟡 Q16. How would you render a list of 10,000 items smoothly?
- [ ] 🔴 Q17. Typing in a search box above a big table lags badly. Walk me through fixing it.

---

## 18 · Forms

- [ ] 🟢 Q18. Controlled vs uncontrolled inputs — when would you use each?
- [ ] 🟡 Q19. How do you handle validation in a large React form?
- [ ] 🔴 Q20. Users report the order form sometimes creates two orders. What do you check?

---

## 19 · Routing

- [ ] 🟢 Q21. How does client-side routing work?
- [ ] 🟡 Q22. How do you implement protected routes, and are they secure?
- [ ] 🔴 Q23. The SPA works when you click around, but refreshing on /products/42 gives a 404. Why?

---

## 20 · Data Fetching

- [ ] 🟢 Q24. What states should every data-fetching UI handle?
- [ ] 🟡 Q25. What is a race condition in data fetching and how do you prevent it?
- [ ] 🟡 Q26. What is an optimistic update?
- [ ] 🔴 Q27. A page shows spinners in sequence and takes 3 seconds even though each API call is fast. What's happening?

---

## 21 · Rendering Strategies

- [ ] 🟢 Q28. What's the difference between CSR, SSR and SSG?
- [ ] 🟡 Q29. What is hydration and what causes a hydration mismatch?
- [ ] 🟡 Q30. How are React Server Components different from SSR?
- [ ] 🔴 Q31. Choose a rendering strategy for an e-commerce site: home, product pages, cart, and admin.

---

## 22 · Web Performance

- [ ] 🟢 Q32. What are the Core Web Vitals and their good thresholds?
- [ ] 🟡 Q33. How do you improve LCP?
- [ ] 🟡 Q34. What caching headers would you set for a React app's files?
- [ ] 🔴 Q35. Your page has a CLS of 0.35 and poor INP on mobile. What do you do?

---

## 23 · Build Tools

- [ ] 🟢 Q36. What is tree shaking and what does it need?
- [ ] 🟡 Q37. Why is Vite's dev server so much faster than webpack's?
- [ ] 🔴 Q38. The main bundle grew to 2 MB. How do you investigate and fix it?

---

## 24 · Frontend Testing

- [ ] 🟢 Q39. What is the philosophy of React Testing Library?
- [ ] 🟡 Q40. Unit vs integration vs E2E tests — how do you balance them on the frontend?
- [ ] 🔴 Q41. After a refactor, half the test suite fails but the app works. What does that tell you?

---

## 25 · Frontend System Design

- [ ] 🟢 Q42. What is the RADIO framework?
- [ ] 🟡 Q43. Why use cursor pagination instead of offset for a news feed?
- [ ] 🔴 Q44. Design an autocomplete search box. What are the key points?
- [ ] 🔴 Q45. Design an infinite-scroll news feed that stays fast after 1,000 posts.
