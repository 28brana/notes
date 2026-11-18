# API Fundamentals · Practice Questions

[← Back to API Fundamentals](../README.md) · [Q&A with answers →](api-fundamentals-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](api-fundamentals-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**42 questions.**

---

## 01 · What is an API

- [ ] 🟢 Q1. What is an API, in simple words?
- [ ] 🟡 Q2. What makes a good API?
- [ ] 🔴 Q3. In a system design interview, when and how do you define the API?

---

## 02 · API Styles

- [ ] 🟢 Q4. Name the main API styles and what each is good for.
- [ ] 🟡 Q5. Compare REST, GraphQL and gRPC.
- [ ] 🔴 Q6. You're designing a social app with web, mobile, 30 microservices and chat. Which API styles do you use where?

---

## 03 · Idempotency

- [ ] 🟢 Q7. What does idempotent mean, and which HTTP methods are idempotent?
- [ ] 🟡 Q8. How does an Idempotency-Key work?
- [ ] 🔴 Q9. Two identical payment requests with the same key arrive at the same time on two servers. How do you avoid a double charge?
- [ ] 🔴 Q10. Users report double charges even though you added idempotency keys. What could be wrong?

---

## 04 · Data Formats

- [ ] 🟢 Q11. JSON vs Protobuf — when would you use each?
- [ ] 🟡 Q12. How do you change a Protobuf or Avro schema without breaking running services?

---

## 05 · REST

- [ ] 🟢 Q13. What are the core principles of REST?
- [ ] 🟡 Q14. Offset vs cursor pagination — which would you use for a news feed and why?
- [ ] 🟡 Q15. How do you version a REST API?
- [ ] 🔴 Q16. The mobile feed screen makes 21 REST calls and feels slow. What do you do?

---

## 06 · GraphQL

- [ ] 🟢 Q17. What problem does GraphQL solve?
- [ ] 🟡 Q18. What is the N+1 problem in GraphQL and how do you fix it?
- [ ] 🔴 Q19. Your public GraphQL API is being hit with deeply nested queries that overload the database. What do you do?

---

## 07 · gRPC

- [ ] 🟢 Q20. What is gRPC and why is it fast?
- [ ] 🟡 Q21. Why don't we usually expose gRPC directly to browsers?
- [ ] 🔴 Q22. After moving to gRPC, one backend server gets most of the traffic while others sit idle. Why?

---

## 08 · Rate Limiting

- [ ] 🟢 Q23. What is rate limiting and why do we need it?
- [ ] 🟡 Q24. Explain token bucket vs fixed window.
- [ ] 🟡 Q25. How does the sliding window counter work?
- [ ] 🔴 Q26. You have 10 API gateway nodes, each with an in-memory limit of 100/min, but users are getting 1,000/min. Fix it.

---

## 09 · Authentication vs Authorization

- [ ] 🟢 Q27. What's the difference between authentication and authorization?
- [ ] 🟡 Q28. In a microservices system, where do you do authentication and where authorization?
- [ ] 🔴 Q29. A user changes `/orders/101` to `/orders/102` in the URL and sees someone else's order. What went wrong?

---

## 10 · Session vs Token Auth

- [ ] 🟢 Q30. What's the difference between session-based and token-based auth?
- [ ] 🟡 Q31. How do you scale session-based auth across many servers?
- [ ] 🟡 Q32. Why use both access and refresh tokens?
- [ ] 🔴 Q33. After adding a second server, users randomly get logged out. Why, and how do you fix it?

---

## 11 · JWT

- [ ] 🟢 Q34. What is a JWT and what are its three parts?
- [ ] 🟡 Q35. Why use RS256 instead of HS256 in microservices?
- [ ] 🔴 Q36. A user's phone is stolen. How do you invalidate their JWT immediately?
- [ ] 🔴 Q37. An admin removed a user's moderator role, but the user can still delete posts for a while. Why?

---

## 12 · OAuth 2.0

- [ ] 🟢 Q38. What is OAuth 2.0 used for?
- [ ] 🟡 Q39. Explain the Authorization Code flow with PKCE.
- [ ] 🟡 Q40. What's the difference between OAuth 2.0 and OpenID Connect?

---

## 13 · SSO

- [ ] 🟢 Q41. How does SSO let a user log in only once?
- [ ] 🔴 Q42. What are the risks of SSO and how do you handle them?
