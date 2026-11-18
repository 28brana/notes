# Caching · Practice Questions

[← Back to Caching](../README.md) · [Q&A with answers →](caching-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](caching-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**27 questions.**

---

## 01 · What Is Caching

- [ ] 🟢 Q1. What is a cache, and why does it make a system faster?
- [ ] 🟢 Q2. What is a cache hit ratio, and why does going from 90% to 99% matter so much?
- [ ] 🟡 Q3. Name the levels where data can be cached between a user and the database.
- [ ] 🔴 Q4. What data would you NOT cache in a payments system, and why?

---

## 02 · Caching Strategies

- [ ] 🟢 Q5. Explain the cache-aside pattern.
- [ ] 🟡 Q6. What's the difference between write-through, write-behind and write-around?
- [ ] 🟡 Q7. When is write-behind a good idea, and when is it dangerous?
- [ ] 🔴 Q8. Users edit their profile but still see the old version right after saving. Your setup is cache-aside with a 1-hour TTL. What's wrong and how do you fix it?

---

## 03 · Cache Eviction Policies

- [ ] 🟢 Q9. What's the difference between eviction and expiration?
- [ ] 🟡 Q10. Compare LRU and LFU. When would LFU beat LRU?
- [ ] 🟡 Q11. How would you implement an LRU cache with O(1) get and put?
- [ ] 🔴 Q12. After a nightly report job runs, the morning cache hit ratio drops from 95% to 60%. Why?

---

## 04 · Cache Invalidation

- [ ] 🟢 Q13. What are the main ways to invalidate a cache?
- [ ] 🟡 Q14. On a write, why delete the cache key instead of updating it with the new value?
- [ ] 🟡 Q15. Should you delete the cache before or after updating the database?
- [ ] 🔴 Q16. Even with DB-then-delete, you occasionally see stale values that stick until TTL. How can that happen, and what do you do?

---

## 05 · CDN

- [ ] 🟢 Q17. What is a CDN and what problem does it solve?
- [ ] 🟡 Q18. Push CDN vs pull CDN — which would you choose?
- [ ] 🟡 Q19. You deploy a new version of your JS bundle, but users keep getting the old one. How do you avoid this?
- [ ] 🔴 Q20. After adding a CDN, a user reports seeing another user's account page. What happened?

---

## 06 · Distributed Cache

- [ ] 🟢 Q21. Why use a distributed cache instead of an in-memory cache on each app server?
- [ ] 🟡 Q22. Redis or Memcached — how do you choose?
- [ ] 🟡 Q23. How do you shard keys across cache nodes, and why not just use hash mod N?
- [ ] 🔴 Q24. One Redis shard is at 100% CPU while the others are at 20%. What's going on and how do you fix it?

---

## 07 · Cache Stampede

- [ ] 🟡 Q25. What is a cache stampede?
- [ ] 🔴 Q26. Every hour on the hour, your DB CPU spikes to 100% for a few seconds. What's the likely cause and fix?

---

## 08 · Cache Warming

- [ ] 🟢 Q27. What is cache warming and when do you need it?
