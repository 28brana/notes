# Deployment Patterns · Practice Questions

[← Back to Deployment Patterns](../README.md) · [Q&A with answers →](deployment-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](deployment-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**32 questions.**

---

## 01 · Deployment Strategies

- [ ] 🟢 Q1. What deployment strategies do you know, and how do they differ?
- [ ] 🟡 Q2. What is a shadow (dark) deployment, and what's the main risk?
- [ ] 🔴 Q3. You're designing deploys for a payments service with 50k req/s. Which strategy would you pick?

---

## 02 · CI/CD

- [ ] 🟢 Q4. What's the difference between CI, continuous delivery and continuous deployment?
- [ ] 🟡 Q5. Why "build once, deploy many"?
- [ ] 🟡 Q6. What does a good CI/CD pipeline look like?
- [ ] 🔴 Q7. The team adopted continuous deployment and bugs reach prod more often. What's wrong?

---

## 03 · Rolling Deployments

- [ ] 🟢 Q8. How does a rolling deployment work?
- [ ] 🟡 Q9. What problems come from having two versions live at once?
- [ ] 🟡 Q10. What do maxUnavailable and maxSurge control?
- [ ] 🔴 Q11. During a rolling deploy, users keep getting logged out. Why?

---

## 04 · Blue-Green Deployments

- [ ] 🟢 Q12. What is a blue-green deployment?
- [ ] 🟡 Q13. Why is a load balancer switch better than a DNS switch for blue-green?
- [ ] 🔴 Q14. You flipped back to blue after a bad release, but blue is also throwing errors. What happened?
- [ ] 🔴 Q15. After a blue-green deploy, some emails are sent twice. Why?

---

## 05 · Canary Releases

- [ ] 🟢 Q16. What is a canary release?
- [ ] 🟡 Q17. How do you decide whether the canary is healthy?
- [ ] 🟡 Q18. Canary vs A/B test — what's the difference?
- [ ] 🔴 Q19. Your service gets 10 req/s. Does a 1% canary make sense?

---

## 06 · Feature Flags

- [ ] 🟢 Q20. What is a feature flag, and what does "deploy ≠ release" mean?
- [ ] 🟡 Q21. What are the downsides of feature flags?
- [ ] 🔴 Q22. A new feature is overloading the database at peak. It's behind a flag. What do you do?

---

## 07 · A/B Testing

- [ ] 🟢 Q23. How is an A/B test different from a feature flag?
- [ ] 🟡 Q24. Why must A/B assignment be sticky, and how do you do it?
- [ ] 🔴 Q25. After 2 days, variant B shows +8% engagement. The PM wants to ship it. What do you say?

---

## 08 · Schema Migrations

- [ ] 🟢 Q26. Why can't you deploy new code and a breaking schema change together?
- [ ] 🟡 Q27. Walk me through renaming a column with zero downtime.
- [ ] 🟡 Q28. How do tools like gh-ost or pt-online-schema-change work?
- [ ] 🔴 Q29. A simple ALTER TABLE on the posts table took the site down. Why?

---

## 09 · Rollbacks & Immutable Infrastructure

- [ ] 🟢 Q30. What is immutable infrastructure?
- [ ] 🟡 Q31. What are your options to undo a bad release, from fastest to slowest?
- [ ] 🔴 Q32. You rolled back to the previous image but users still see corrupted profiles. Why?
