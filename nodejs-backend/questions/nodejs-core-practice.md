# Node.js Backend — Node.js Core · Practice Questions

[← Back to Node.js Backend](../README.md) · [Q&A with answers →](nodejs-core-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](nodejs-core-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**38 questions.**

---

## 01 · What is Node.js

- [ ] 🟢 Q1. What is Node.js, and what are V8 and libuv?
- [ ] 🟡 Q2. Node is "single-threaded" — so how does it handle 10,000 connections, and which operations use the thread pool?
- [ ] 🔴 Q3. Under load, file reads and outgoing HTTP calls get slow, but CPU is only 40%. The app hashes passwords with pbkdf2 on login. What's happening?

---

## 02 · The Event Loop

- [ ] 🟢 Q4. What are the phases of the Node.js event loop?
- [ ] 🟡 Q5. What does this print, and why?
- [ ] 🟡 Q6. What's the difference between process.nextTick, setImmediate and setTimeout(fn, 0)?
- [ ] 🔴 Q7. API latency spikes to 2 seconds for everyone whenever one admin downloads a report. What do you check?

---

## 03 · Async Patterns

- [ ] 🟢 Q8. What is callback hell and how did Node move past it?
- [ ] 🟡 Q9. Explain Promise.all, allSettled, race and any.
- [ ] 🟡 Q10. Why is using await inside forEach a bug?
- [ ] 🔴 Q11. An endpoint makes 3 independent DB calls of ~100 ms each and takes 300 ms. Then someone "fixes" a batch job with Promise.all over 20,000 IDs and the DB falls over. Explain both.

---

## 04 · Modules

- [ ] 🟢 Q12. What are the main differences between CommonJS and ES Modules?
- [ ] 🟡 Q13. Why does requiring the same module twice give you the same object?
- [ ] 🔴 Q14. After a refactor, a function imported from another file is suddenly undefined at runtime, but it's clearly exported. What could cause it?

---

## 05 · Streams and Buffers

- [ ] 🟢 Q15. What are streams and what are the four types?
- [ ] 🟡 Q16. What is backpressure and how does Node handle it?
- [ ] 🟡 Q17. Why prefer stream.pipeline over .pipe, and what is a Buffer?
- [ ] 🔴 Q18. Your CSV export endpoint crashes the server with out-of-memory when several users export at once. How do you fix it?

---

## 06 · EventEmitter

- [ ] 🟢 Q19. What is EventEmitter, and are listeners called synchronously?
- [ ] 🟡 Q20. What happens if you emit 'error' with no listener, and what does MaxListenersExceededWarning mean?

---

## 07 · Error Handling

- [ ] 🟢 Q21. What's the difference between operational and programmer errors?
- [ ] 🟡 Q22. How do you handle errors from async route handlers in Express?
- [ ] 🟡 Q23. What should you do in unhandledRejection and uncaughtException handlers?
- [ ] 🔴 Q24. Some requests hang forever with no error in the logs. What do you suspect?

---

## 08 · Worker Threads, Cluster and Child Processes

- [ ] 🟢 Q25. How do you use all CPU cores with Node?
- [ ] 🟡 Q26. Compare cluster, worker_threads and child_process.
- [ ] 🟡 Q27. Should you use worker threads to speed up DB queries or HTTP calls?
- [ ] 🔴 Q28. After moving to PM2 cluster mode with 4 instances, users get randomly logged out. Why?

---

## 09 · Memory and Garbage Collection

- [ ] 🟢 Q29. How does garbage collection work in V8, at a high level?
- [ ] 🟡 Q30. What are common causes of memory leaks in Node?
- [ ] 🟡 Q31. What does --max-old-space-size do, and when do you set it?
- [ ] 🔴 Q32. A Node service's memory grows steadily for hours and then it crashes with "heap out of memory". How do you investigate?

---

## 10 · npm and Packages

- [ ] 🟢 Q33. What's the difference between ^1.2.3 and ~1.2.3?
- [ ] 🟡 Q34. Why commit package-lock.json, and when do you use npm ci instead of npm install?
- [ ] 🔴 Q35. A build that passed yesterday fails today with no code changes. What happened, and how do you prevent it?

---

## 11 · Performance and Profiling

- [ ] 🟢 Q36. What is event loop lag and why monitor it?
- [ ] 🟡 Q37. How do you profile a slow Node.js app?
- [ ] 🔴 Q38. After a release, p99 latency went from 80 ms to 900 ms but p50 barely moved. Where do you look?
