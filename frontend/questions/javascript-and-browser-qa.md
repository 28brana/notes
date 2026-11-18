# Q&A · Frontend — JavaScript, HTML/CSS & the Browser

[← Back to Frontend](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · How Browsers Render a Page

[Read the notes →](../01-how-browsers-render.md)

> [!question]- 🟢 Q1. Walk me through the critical rendering path.
> The browser parses HTML into the DOM and CSS into the CSSOM, then combines them into a render tree of visible elements. Layout calculates every box's size and position, paint fills in the pixels, and composite has the GPU stack the layers into the final frame.

> [!question]- 🟢 Q2. What is the difference between async and defer?
> Both download the script in parallel without blocking parsing. `async` runs the script as soon as it arrives, in any order — good for analytics. `defer` waits until the HTML is fully parsed and keeps the order — the right default for app code.

> [!question]- 🟡 Q3. What's the difference between reflow and repaint?
> Reflow (layout) recalculates sizes and positions — triggered by changing width, margin, font size or adding nodes — and is the most expensive. Repaint only redraws pixels, like changing colour or background. Changing `transform` or `opacity` can skip both and only composite, which is why animations should use them.

> [!question]- 🔴 Q4. A page shows a blank white screen for 3 seconds on mobile. How do you debug and fix it?
> I'd open the Performance and Network panels and look for render-blocking resources. Usual fixes: move scripts to `defer`, inline the small critical CSS and load the rest later, preload the main font and hero image, and split the big JS bundle. Then I'd re-check First Contentful Paint in Lighthouse.

---

## 02 · Semantic HTML & Accessibility

[Read the notes →](../02-html-and-accessibility.md)

> [!question]- 🟢 Q5. What is semantic HTML and why does it matter?
> It means using tags that describe meaning — `header`, `nav`, `main`, `button` — instead of generic `div`s. Screen readers use them to navigate, search engines understand the page better, and native elements like `button` give keyboard and focus behaviour for free.

> [!question]- 🟡 Q6. When should you use ARIA?
> Only when no native HTML element does the job — for custom widgets like tabs, modals or comboboxes. ARIA describes role, name and state, like `aria-expanded` or `aria-label`. It doesn't add behaviour, so you still handle keyboard yourself, and wrong ARIA is worse than none.

> [!question]- 🔴 Q7. A designer's custom dropdown is built from divs. What accessibility problems do you check for?
> Can I reach and open it with the keyboard (Tab, Enter, arrows, Escape)? Does it have a visible focus state, the right roles (`combobox`/`listbox`/`option`) and `aria-expanded`? Is the selected value announced and does it have a label? If that's a lot of work, a styled native `select` is often the better answer.

---

## 03 · CSS Layout — Box Model, Flexbox & Grid

[Read the notes →](../03-css-layout.md)

> [!question]- 🟢 Q8. Explain the CSS box model and `box-sizing`.
> Each element is content, then padding, border and margin. With the default `content-box`, width only covers content, so padding and border are added on top. With `border-box`, width includes padding and border, which is far more predictable — so most projects set it globally.

> [!question]- 🟢 Q9. Flexbox vs Grid — when do you use which?
> Flexbox is one-dimensional — a row or a column — and great for navbars, toolbars and centering. Grid is two-dimensional, rows and columns together, and suits page layouts and galleries where columns must line up. I often use Grid for the page and Flexbox inside components.

> [!question]- 🟡 Q10. How do you center a div horizontally and vertically?
> On the parent: `display: flex; justify-content: center; align-items: center;`. With Grid it's even shorter: `display: grid; place-items: center;`. For a positioned element, `position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%)` also works.

> [!question]- 🔴 Q11. Cards in a 3-column row keep wrapping to 2 columns. What's likely wrong?
> Most likely `box-sizing` is `content-box`, so padding and border push each card's real width over a third. Other causes are margins adding up, or a fixed width that doesn't fit the container. Fix with `border-box` and `gap` instead of margins — or just use Grid with `repeat(3, 1fr)`.

---

## 04 · CSS Specificity, Positioning & z-index

[Read the notes →](../04-css-specificity-and-positioning.md)

> [!question]- 🟢 Q12. How is CSS specificity calculated?
> It's scored as IDs, then classes/attributes/pseudo-classes, then elements, compared left to right. One ID beats any number of classes, inline styles beat selectors, and `!important` beats everything. If scores tie, the rule written later wins.

> [!question]- 🟡 Q13. What's the difference between relative, absolute, fixed and sticky?
> `relative` stays in flow and shifts from its normal spot. `absolute` leaves the flow and positions against the nearest positioned ancestor. `fixed` positions against the viewport, and `sticky` behaves normally until a scroll threshold, then sticks.

> [!question]- 🔴 Q14. Your modal has `z-index: 9999` but appears behind the header. Why?
> The modal is inside a parent that creates its own stacking context — for example with `transform`, `opacity` or a positioned `z-index` lower than the header's. The 9999 only competes inside that parent. The fix is to render the modal at the body level, like a React portal.

---

## 05 · JavaScript Fundamentals

[Read the notes →](../05-javascript-fundamentals.md)

> [!question]- 🟢 Q15. What's the difference between `var`, `let` and `const`?
> `var` is function-scoped, can be re-declared, and is hoisted as `undefined`. `let` and `const` are block-scoped and stay in the temporal dead zone until their line runs. `const` can't be re-assigned, but the object it points to can still change.

> [!question]- 🟢 Q16. What's the difference between `==` and `===`?
> `===` checks type and value with no conversion. `==` converts types first, which gives surprises like `0 == ""` and `[] == false` being true. I always use `===`, except `x == null` to check for both null and undefined.

> [!question]- 🟡 Q17. What is hoisting and the temporal dead zone?
> Before running code, JS registers all declarations in their scope. `var` is set to `undefined`, function declarations are fully available, and `let`/`const` exist but can't be touched — that gap until the declaration line is the TDZ, and accessing it throws a ReferenceError.

> [!question]- 🔴 Q18. What do `typeof null`, `typeof NaN` and `[] + {}` return, and why?
> `typeof null` is `"object"` — an old bug kept for compatibility. `typeof NaN` is `"number"` because NaN is a special number value. `[] + {}` is `"[object Object]"` because `+` converts both to strings: `""` plus `"[object Object]"`.

---

## 06 · Closures, `this` & Prototypes

[Read the notes →](../06-closures-this-prototypes.md)

> [!question]- 🟢 Q19. What is a closure? Give an example.
> A closure is a function that remembers the variables from the scope where it was created, even after that scope has finished. A counter factory is the classic example: `makeCounter` has a private `count`, and the returned function can still increment it. It's used for private state, debounce and memoize.

> [!question]- 🟡 Q20. Why does a `for` loop with `var` and `setTimeout` print `3 3 3`, and how do you fix it?
> `var` creates one shared `i` for the whole function; by the time the timeouts run, the loop has finished and `i` is 3. Switching to `let` creates a fresh `i` for each iteration, so each callback captures its own value and prints `0 1 2`.

> [!question]- 🟡 Q21. How is `this` decided in JavaScript?
> By how the function is called, in priority order: with `new` it's the new object; with `call`/`apply`/`bind` it's what you pass; called as `obj.fn()` it's `obj`; a plain `fn()` gives `undefined` in strict mode. Arrow functions don't have their own `this` — they take it from the surrounding code.

> [!question]- 🟡 Q22. What's the difference between call, apply and bind?
> All three set `this`. `call` runs the function now with arguments listed one by one, `apply` runs it now with arguments as an array, and `bind` returns a new function with `this` fixed, to call later.

> [!question]- 🔴 Q23. Explain the prototype chain. How does `class` relate to it?
> Every object links to a prototype object. When a property isn't found on the object, JS looks at its prototype, then that prototype's prototype, up to `Object.prototype` and then `null`. `class` is sugar over this: methods go on `Class.prototype` and every instance links to it, so they share one copy.

---

## 07 · Async JavaScript — Event Loop & Promises

[Read the notes →](../07-async-javascript.md)

> [!question]- 🟢 Q24. What are the states of a promise?
> Pending, then either fulfilled with a value or rejected with an error. Once settled it never changes. `.then` handles success, `.catch` handles failure, and `.finally` runs either way.

> [!question]- 🟡 Q25. Explain the event loop, microtasks and macrotasks.
> JS runs synchronous code on the call stack. When the stack is empty, the event loop runs all microtasks — promise callbacks and code after `await` — then the browser may render, then it takes one macrotask like a timer or click event, and repeats. So promises always run before a `setTimeout` of 0.

> [!question]- 🟡 Q26. What does this print? `console.log(1); setTimeout(() => console.log(2)); Promise.resolve().then(() => console.log(3)); console.log(4);`
> `1 4 3 2`. 1 and 4 are synchronous. The promise callback is a microtask, so 3 runs as soon as the stack is empty. The `setTimeout` callback is a macrotask, so 2 comes last.

> [!question]- 🔴 Q27. A dashboard loads user, orders and notifications one after another and takes 900 ms. How do you speed it up, and what if one call fails?
> They're independent, so run them in parallel with `Promise.all` — total time drops to the slowest call, around 300 ms. But `Promise.all` rejects if any one fails, so if notifications are optional I'd use `Promise.allSettled` and render whatever succeeded.

---

## 08 · DOM & Events

[Read the notes →](../08-dom-and-events.md)

> [!question]- 🟢 Q28. What are event bubbling and capturing?
> An event first travels down from the window to the target (capturing), then back up from the target to the window (bubbling). Listeners run in the bubbling phase by default; pass `true` as the third argument to listen during capture.

> [!question]- 🟡 Q29. What is event delegation and why use it?
> Instead of adding a listener to every child, you add one to the parent and use `e.target.closest()` to see which child was clicked. It saves memory with long lists and automatically works for items added later.

> [!question]- 🟡 Q30. Debounce vs throttle — what's the difference and when do you use each?
> Debounce waits until events stop for X ms, then runs once — ideal for a search box or autosave. Throttle runs at most once every X ms while events keep coming — ideal for scroll or mousemove handlers.

> [!question]- 🔴 Q31. Search-as-you-type sends a request per keystroke and sometimes shows old results. How do you fix it?
> Debounce the input by about 300 ms so we only call the API when the user pauses. Old results appear because an earlier slow response arrives after a newer one — so cancel the previous request with `AbortController`, or ignore any response that isn't for the latest query.

---

## 09 · Modern JavaScript (ES6+)

[Read the notes →](../09-modern-javascript.md)

> [!question]- 🟢 Q32. What's the difference between spread and rest?
> Same `...` syntax, opposite jobs. Spread expands an array or object into individual items — for copying or merging. Rest collects remaining items into an array, like in function parameters `(...args)` or destructuring.

> [!question]- 🟡 Q33. What's the difference between `??` and `||`?
> `||` falls back on any falsy value, so `0 || 10` gives 10 — often a bug. `??` only falls back on `null` or `undefined`, so `0 ?? 10` keeps 0. Use `??` for defaults.

> [!question]- 🟡 Q34. ES modules vs CommonJS — what's the difference?
> ESM uses `import`/`export` and is static, so bundlers can analyse it and tree-shake unused code. CommonJS uses `require`/`module.exports` and loads dynamically at runtime. ESM is the standard for browsers and modern Node; dynamic `import()` gives code splitting.

---

## 10 · TypeScript Basics

[Read the notes →](../10-typescript.md)

> [!question]- 🟢 Q35. What's the difference between `type` and `interface`?
> Both describe object shapes. Only `type` can describe unions, primitives and tuples, while `interface` supports declaration merging and `extends`. I use interfaces for object shapes and types for unions — and keep it consistent across the team.

> [!question]- 🟡 Q36. What's the difference between `any` and `unknown`?
> `any` switches type checking off — you can do anything with it, so bugs slip through. `unknown` also accepts any value, but you must narrow it with a check like `typeof` before using it. So `unknown` is the safe choice for API data or caught errors.

> [!question]- 🟡 Q37. What are generics, and name a few utility types?
> Generics are type parameters that let one function or type work with many types while keeping type info — like `ApiResponse<T>`. Common utility types: `Partial` makes fields optional, `Pick` and `Omit` select or drop keys, `Record` builds a key-value map type, `Readonly` blocks re-assignment.

---

## 11 · Browser Storage

[Read the notes →](../11-browser-storage.md)

> [!question]- 🟢 Q38. Compare cookies, localStorage and sessionStorage.
> Cookies are about 4 KB and are sent to the server with every request, so they're used for sessions. localStorage holds a few MB, stays until cleared, and never goes to the server. sessionStorage is similar but only lives for that one tab.

> [!question]- 🔴 Q39. Where would you store a JWT in the browser, and why?
> Preferably in an HttpOnly, Secure, SameSite cookie: JavaScript can't read it, so an XSS bug can't steal it, and SameSite blocks most CSRF. localStorage is easy, but any injected script can read it. The trade-off is that cookies need CSRF protection and a backend that sets them.

---

## 12 · Web Security

[Read the notes →](../12-web-security.md)

> [!question]- 🟢 Q40. What is XSS and how do you prevent it?
> Cross-site scripting is when an attacker's JavaScript runs on your page, usually from unescaped user input. Prevent it by escaping output (React does by default), sanitizing any HTML you must render, avoiding `innerHTML`, using HttpOnly cookies, and adding a Content Security Policy.

> [!question]- 🟡 Q41. What is CSRF and how is it different from XSS?
> CSRF tricks a logged-in user's browser into sending a request to your site from another site, using cookies the browser attaches automatically. XSS runs attacker code inside your site. CSRF is fixed with SameSite cookies and CSRF tokens; XSS with escaping and CSP.

> [!question]- 🟡 Q42. Does CORS protect my API?
> No. CORS only controls whether a browser lets JavaScript from another origin *read* the response. The request can still be sent, and tools like curl ignore CORS completely. Real protection is authentication, authorization and CSRF defences on the server.

> [!question]- 🔴 Q43. Users report their accounts posted spam they never wrote, right after viewing a certain comment. What happened and what do you fix?
> Classic stored XSS: the comment contains a script that runs for every viewer and calls the API as them. Fix the rendering path — escape or sanitize that content, remove any `innerHTML` or `dangerouslySetInnerHTML` use — clean up the stored payloads, add a strict CSP, and make sure session cookies are HttpOnly.

---

## 13 · JS Coding Questions

[Read the notes →](../13-js-coding-questions.md)

> [!question]- 🟡 Q44. Implement debounce.
> Keep a timer in a closure; on every call clear it and start a new one.
> ```js
> function debounce(fn, ms) {
>   let t;
>   return function (...args) {
>     clearTimeout(t);
>     t = setTimeout(() => fn.apply(this, args), ms);
>   };
> }
> ```

> [!question]- 🟡 Q45. What are the key points when writing a `Promise.all` polyfill?
> Return a new promise; store each result at its own index so order matches input; count completions and resolve when the count hits the length; reject on the first failure. Wrap items in `Promise.resolve` so plain values work, and resolve immediately for an empty array.

> [!question]- 🟡 Q46. How would you implement `Function.prototype.bind`?
> Save the original function (`this`), and return a new function that calls it with `apply`, passing the bound context and the preset arguments followed by any later arguments. A full version also handles being called with `new`.

> [!question]- 🔴 Q47. Write a deep clone. What edge cases matter?
> Recursively copy objects and arrays, returning primitives as they are. Edge cases: circular references (track seen objects in a `WeakMap`), Dates, Maps, Sets and class instances. `JSON.parse(JSON.stringify())` drops functions, `undefined` and dates and crashes on cycles — in modern code `structuredClone` handles most cases.
