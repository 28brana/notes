# Node.js Backend — Building Production Backends · Practice Questions

[← Back to Node.js Backend](../README.md) · [Q&A with answers →](building-backends-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](building-backends-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**51 questions.**

---

## 12 · Express Basics

- [ ] 🟢 Q1. What is Express and what does it add on top of Node's http module?
- [ ] 🟡 Q2. How do you structure a medium-size Express app?
- [ ] 🟡 Q3. Express vs Fastify vs NestJS — when would you pick each?
- [ ] 🔴 Q4. You see "Cannot set headers after they are sent to the client" in logs. What's going on?

---

## 13 · Middleware

- [ ] 🟢 Q5. What is middleware in Express?
- [ ] 🟡 Q6. In what order would you register middleware, and why?
- [ ] 🟡 Q7. How does error-handling middleware work, and how do you catch errors from async handlers?
- [ ] 🔴 Q8. Some requests hang forever with no response and no error. What do you check?

---

## 14 · REST API Design

- [ ] 🟢 Q9. What's the difference between 401 and 403?
- [ ] 🟡 Q10. Offset vs cursor pagination — trade-offs?
- [ ] 🟡 Q11. How do you version an API, and when do you bump the version?
- [ ] 🔴 Q12. A mobile client retried POST /payments on a timeout and the user was charged twice. How do you fix it?

---

## 15 · Validation

- [ ] 🟢 Q13. Why validate on the server if the frontend already validates?
- [ ] 🟡 Q14. How do you implement validation in Express, and what is mass assignment?
- [ ] 🔴 Q15. Where does "email already taken" belong — validation middleware or service? What status code?

---

## 16 · Authentication

- [ ] 🟢 Q16. How do you store passwords in a Node app?
- [ ] 🟡 Q17. Why use both an access token and a refresh token?
- [ ] 🟡 Q18. Where should tokens be stored in the browser?
- [ ] 🔴 Q19. A user changed their password but their stolen session still works for a week. What's wrong with the design and how do you fix it?

---

## 17 · Databases and ORMs

- [ ] 🟢 Q20. What is the N+1 query problem and how do you fix it?
- [ ] 🟡 Q21. Why connection pooling, and how do you size the pool?
- [ ] 🟡 Q22. How do you do a transaction with an ORM, and what should you avoid inside it?
- [ ] 🔴 Q23. After scaling from 5 to 30 pods, the API starts failing with "too many connections". What happened?

---

## 18 · Caching with Redis

- [ ] 🟢 Q24. Explain the cache-aside pattern.
- [ ] 🟡 Q25. Why Redis instead of an in-memory JS object?
- [ ] 🟡 Q26. Why delete the key on write instead of updating it?
- [ ] 🔴 Q27. A popular product's cache key expires and the DB CPU spikes to 100%. Why, and how do you prevent it?

---

## 19 · Background Jobs

- [ ] 🟢 Q28. Why move work to a background queue?
- [ ] 🟡 Q29. How does BullMQ handle failures?
- [ ] 🟡 Q30. Why must job handlers be idempotent?
- [ ] 🔴 Q31. A nightly report cron is running 6 times a night. Why?

---

## 20 · File Uploads

- [ ] 🟢 Q32. How do you handle file uploads in Express?
- [ ] 🟡 Q33. What is a presigned URL and why use it for uploads?
- [ ] 🔴 Q34. Users uploading large videos make the API run out of memory. What's wrong?

---

## 21 · Realtime with Socket.IO

- [ ] 🟢 Q35. What does Socket.IO give you over plain WebSockets?
- [ ] 🔴 Q36. Chat works with one server, but after adding a second server some users stop receiving messages. Why?

---

## 22 · Testing

- [ ] 🟢 Q37. What's the difference between unit and integration tests in a Node API?
- [ ] 🟡 Q38. How do you test Express routes with Supertest?
- [ ] 🟡 Q39. How do you manage the database in tests?
- [ ] 🔴 Q40. All tests pass, but production breaks on a SQL constraint error. What's wrong with the test strategy?

---

## 23 · Config and Logging

- [ ] 🟢 Q41. How should a Node app handle configuration and secrets?
- [ ] 🟡 Q42. Why use pino instead of console.log?
- [ ] 🔴 Q43. A customer reports one failed checkout and you have 2 million log lines from 10 pods. How do you find what happened?

---

## 24 · Security Best Practices

- [ ] 🟢 Q44. What does helmet do?
- [ ] 🟡 Q45. Does CORS protect your API from attackers?
- [ ] 🟡 Q46. How do you prevent SQL and NoSQL injection in Node?
- [ ] 🔴 Q47. One API request with a long input string makes the entire server unresponsive for seconds. What could it be?

---

## 25 · Production Deployment

- [ ] 🟢 Q48. How do you use all CPU cores with Node?
- [ ] 🟡 Q49. What is graceful shutdown and how do you implement it?
- [ ] 🟡 Q50. Liveness vs readiness probes?
- [ ] 🔴 Q51. Every deploy causes a few seconds of 502 errors. What do you check?
