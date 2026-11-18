# Frontend — JavaScript, HTML/CSS & the Browser · Practice Questions

[← Back to Frontend](../README.md) · [Q&A with answers →](javascript-and-browser-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](javascript-and-browser-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**47 questions.**

---

## 01 · How Browsers Render a Page

- [ ] 🟢 Q1. Walk me through the critical rendering path.
- [ ] 🟢 Q2. What is the difference between async and defer?
- [ ] 🟡 Q3. What's the difference between reflow and repaint?
- [ ] 🔴 Q4. A page shows a blank white screen for 3 seconds on mobile. How do you debug and fix it?

---

## 02 · Semantic HTML & Accessibility

- [ ] 🟢 Q5. What is semantic HTML and why does it matter?
- [ ] 🟡 Q6. When should you use ARIA?
- [ ] 🔴 Q7. A designer's custom dropdown is built from divs. What accessibility problems do you check for?

---

## 03 · CSS Layout — Box Model, Flexbox & Grid

- [ ] 🟢 Q8. Explain the CSS box model and `box-sizing`.
- [ ] 🟢 Q9. Flexbox vs Grid — when do you use which?
- [ ] 🟡 Q10. How do you center a div horizontally and vertically?
- [ ] 🔴 Q11. Cards in a 3-column row keep wrapping to 2 columns. What's likely wrong?

---

## 04 · CSS Specificity, Positioning & z-index

- [ ] 🟢 Q12. How is CSS specificity calculated?
- [ ] 🟡 Q13. What's the difference between relative, absolute, fixed and sticky?
- [ ] 🔴 Q14. Your modal has `z-index: 9999` but appears behind the header. Why?

---

## 05 · JavaScript Fundamentals

- [ ] 🟢 Q15. What's the difference between `var`, `let` and `const`?
- [ ] 🟢 Q16. What's the difference between `==` and `===`?
- [ ] 🟡 Q17. What is hoisting and the temporal dead zone?
- [ ] 🔴 Q18. What do `typeof null`, `typeof NaN` and `[] + {}` return, and why?

---

## 06 · Closures, `this` & Prototypes

- [ ] 🟢 Q19. What is a closure? Give an example.
- [ ] 🟡 Q20. Why does a `for` loop with `var` and `setTimeout` print `3 3 3`, and how do you fix it?
- [ ] 🟡 Q21. How is `this` decided in JavaScript?
- [ ] 🟡 Q22. What's the difference between call, apply and bind?
- [ ] 🔴 Q23. Explain the prototype chain. How does `class` relate to it?

---

## 07 · Async JavaScript — Event Loop & Promises

- [ ] 🟢 Q24. What are the states of a promise?
- [ ] 🟡 Q25. Explain the event loop, microtasks and macrotasks.
- [ ] 🟡 Q26. What does this print? `console.log(1); setTimeout(() => console.log(2)); Promise.resolve().then(() => console.log(3)); console.log(4);`
- [ ] 🔴 Q27. A dashboard loads user, orders and notifications one after another and takes 900 ms. How do you speed it up, and what if one call fails?

---

## 08 · DOM & Events

- [ ] 🟢 Q28. What are event bubbling and capturing?
- [ ] 🟡 Q29. What is event delegation and why use it?
- [ ] 🟡 Q30. Debounce vs throttle — what's the difference and when do you use each?
- [ ] 🔴 Q31. Search-as-you-type sends a request per keystroke and sometimes shows old results. How do you fix it?

---

## 09 · Modern JavaScript (ES6+)

- [ ] 🟢 Q32. What's the difference between spread and rest?
- [ ] 🟡 Q33. What's the difference between `??` and `||`?
- [ ] 🟡 Q34. ES modules vs CommonJS — what's the difference?

---

## 10 · TypeScript Basics

- [ ] 🟢 Q35. What's the difference between `type` and `interface`?
- [ ] 🟡 Q36. What's the difference between `any` and `unknown`?
- [ ] 🟡 Q37. What are generics, and name a few utility types?

---

## 11 · Browser Storage

- [ ] 🟢 Q38. Compare cookies, localStorage and sessionStorage.
- [ ] 🔴 Q39. Where would you store a JWT in the browser, and why?

---

## 12 · Web Security

- [ ] 🟢 Q40. What is XSS and how do you prevent it?
- [ ] 🟡 Q41. What is CSRF and how is it different from XSS?
- [ ] 🟡 Q42. Does CORS protect my API?
- [ ] 🔴 Q43. Users report their accounts posted spam they never wrote, right after viewing a certain comment. What happened and what do you fix?

---

## 13 · JS Coding Questions

- [ ] 🟡 Q44. Implement debounce.
- [ ] 🟡 Q45. What are the key points when writing a `Promise.all` polyfill?
- [ ] 🟡 Q46. How would you implement `Function.prototype.bind`?
- [ ] 🔴 Q47. Write a deep clone. What edge cases matter?
