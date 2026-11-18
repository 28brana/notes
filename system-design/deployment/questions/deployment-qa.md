# Q&A · Deployment Patterns

[← Back to Deployment Patterns](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · Deployment Strategies

[Read the notes →](../01-deployment-strategies.md)

> [!question]- 🟢 Q1. What deployment strategies do you know, and how do they differ?
> Recreate stops everything and starts the new version — simple but has downtime. Rolling replaces servers in batches with no extra cost. Blue-green runs two full environments and flips traffic for instant rollback. Canary sends a small % of traffic first, and shadow copies traffic to the new version without users seeing its replies.

> [!question]- 🟡 Q2. What is a shadow (dark) deployment, and what's the main risk?
> Live traffic is copied to the new version, but its responses are thrown away, so users never see it. It's great for testing a rewrite under real load. The risk is side effects — the shadow must not charge cards, send emails or write to the real DB, or every action happens twice.

> [!question]- 🔴 Q3. You're designing deploys for a payments service with 50k req/s. Which strategy would you pick?
> I'd use a canary in front of a rolling or blue-green rollout: 1% of traffic first, with automated checks on error rate, p99 latency and payment success rate, then step up. High traffic means the canary gets a meaningful signal fast. I'd add feature flags for new payment flows and make sure all DB changes are backward compatible so rollback is always possible.

---

## 02 · CI/CD

[Read the notes →](../02-ci-cd.md)

> [!question]- 🟢 Q4. What's the difference between CI, continuous delivery and continuous deployment?
> CI means every commit is merged often and automatically built and tested. Continuous delivery means every green build is ready for prod, but a human presses deploy. Continuous deployment removes that click — every green build goes to prod automatically.

> [!question]- 🟡 Q5. Why "build once, deploy many"?
> If you rebuild for each environment, dependencies or build tools can change, so prod runs something you never tested. Building one versioned artifact and promoting it through staging to prod guarantees you ship exactly what you tested, and makes rollback easy.

> [!question]- 🟡 Q6. What does a good CI/CD pipeline look like?
> Commit → build → unit and integration tests → versioned artifact in a registry → deploy to staging → end-to-end smoke tests → progressive deploy to prod with monitoring and automatic rollback. It should be fast (CI under ~10 minutes), use secrets from a vault, and work with small, frequent changes.

> [!question]- 🔴 Q7. The team adopted continuous deployment and bugs reach prod more often. What's wrong?
> CD only ships faster — it doesn't make code safer. Likely the test suite is weak or flaky, and prod rollout is all-at-once with no canary or auto-rollback. I'd strengthen tests on critical paths, add a canary stage with metric-based promotion, and put risky features behind flags.

---

## 03 · Rolling Deployments

[Read the notes →](../03-rolling-deployments.md)

> [!question]- 🟢 Q8. How does a rolling deployment work?
> Servers are updated in small batches. For each batch: take it out of the load balancer, drain in-flight requests, install the new version, wait for health checks, and add it back. No downtime and no extra fleet needed.

> [!question]- 🟡 Q9. What problems come from having two versions live at once?
> The same user may hit v1 and v2 on consecutive requests, so APIs must be backward compatible. Both versions share the database, so schema changes must work for both. Queue messages written by one version may be read by the other, so message formats must stay compatible too.

> [!question]- 🟡 Q10. What do maxUnavailable and maxSurge control?
> maxUnavailable is how many servers can be out of service at once — it trades rollout speed for lost capacity. maxSurge is how many extra servers can be added temporarily, so capacity stays full during the rollout. Kubernetes Deployments use both.

> [!question]- 🔴 Q11. During a rolling deploy, users keep getting logged out. Why?
> Sessions are probably stored in each server's memory, so when a server is replaced its sessions vanish. The fix is to make servers stateless — store sessions in a shared store like Redis or use tokens. Connection draining also helps avoid cutting off in-flight requests.

---

## 04 · Blue-Green Deployments

[Read the notes →](../04-blue-green.md)

> [!question]- 🟢 Q12. What is a blue-green deployment?
> Two identical environments: blue serves users, green gets the new version. After testing green, the router switches all traffic to it at once. If something breaks, we switch back to blue in seconds.

> [!question]- 🟡 Q13. Why is a load balancer switch better than a DNS switch for blue-green?
> A load balancer change takes effect immediately. DNS records are cached by clients and resolvers for the TTL, sometimes longer, so some users keep hitting the old environment for minutes. That makes both the cutover and the rollback slow and unpredictable.

> [!question]- 🔴 Q14. You flipped back to blue after a bad release, but blue is also throwing errors. What happened?
> Most likely the database. Blue and green share it, and the new release ran a migration that blue's code can't handle — for example, it renamed or dropped a column. Rollback only works if schema changes are backward compatible, so I'd use expand → migrate → contract.

> [!question]- 🔴 Q15. After a blue-green deploy, some emails are sent twice. Why?
> Background workers or cron jobs are probably running in both blue and green, both consuming the same queue or schedule. Only the live color should run jobs, or jobs need a lock or idempotency keys so duplicates are harmless.

---

## 05 · Canary Releases

[Read the notes →](../05-canary-releases.md)

> [!question]- 🟢 Q16. What is a canary release?
> We deploy the new version to a small slice of traffic, say 1–5%, and compare its error rate, latency and business metrics against the stable version. If it's healthy we step up gradually to 100%; if not we route traffic away in seconds.

> [!question]- 🟡 Q17. How do you decide whether the canary is healthy?
> Compare the canary with the stable fleet at the same time, not with last week. Watch error rate, p99 latency, saturation and business metrics like posts created or checkouts. Ideally automated: if the canary's error rate is worse than stable by a threshold, abort.

> [!question]- 🟡 Q18. Canary vs A/B test — what's the difference?
> A canary asks "is this build safe?" and is judged on technical health, then rolled to everyone. An A/B test asks "which version is better?" and is judged on a product metric with statistical significance. The canary is temporary; the losing A/B variant is deleted.

> [!question]- 🔴 Q19. Your service gets 10 req/s. Does a 1% canary make sense?
> Not really — 1% is one request every 10 seconds, too little to see a real difference in error rate. I'd use a bigger percentage, run it longer, route internal users first, or rely more on pre-prod tests and feature flags. Canaries shine on high-traffic services.

---

## 06 · Feature Flags

[Read the notes →](../06-feature-flags.md)

> [!question]- 🟢 Q20. What is a feature flag, and what does "deploy ≠ release" mean?
> A feature flag is a runtime switch around code, with its value stored outside the code. We can deploy code with the feature turned off and release it later — to employees, then a small %, then everyone. Turning it off is instant and needs no redeploy.

> [!question]- 🟡 Q21. What are the downsides of feature flags?
> Flag debt — old flags pile up and fill the code with dead branches. More on/off combinations to test. And the flag service becomes a dependency, so apps must cache values and fall back to a safe default if it's down.

> [!question]- 🔴 Q22. A new feature is overloading the database at peak. It's behind a flag. What do you do?
> Flip the flag off — that's the kill switch, effective in seconds without a deploy. Then investigate: maybe a missing index or a query without caching. Re-enable gradually for a small % while watching DB load.

---

## 07 · A/B Testing

[Read the notes →](../07-ab-testing.md)

> [!question]- 🟢 Q23. How is an A/B test different from a feature flag?
> A feature flag controls who gets a feature. An A/B test is an experiment: random, sticky groups, one main metric, and a statistical check to decide which version is better. A/B tests are usually built on top of flags.

> [!question]- 🟡 Q24. Why must A/B assignment be sticky, and how do you do it?
> If a user flips between A and B, their behaviour mixes both versions and the data is useless — plus it's a confusing experience. Hash the user ID into a bucket so the same user always gets the same variant.

> [!question]- 🔴 Q25. After 2 days, variant B shows +8% engagement. The PM wants to ship it. What do you say?
> Don't stop early — that's peeking. Early results swing a lot and often shrink or vanish. We should run until the sample size we planned upfront and at least a full week to cover weekday patterns, then check significance on the main metric.

---

## 08 · Schema Migrations

[Read the notes →](../08-schema-migrations.md)

> [!question]- 🟢 Q26. Why can't you deploy new code and a breaking schema change together?
> During a rollout old and new code run together, and the old code would break against the new schema. And if you need to roll back, the old code can't read the changed schema. App code rolls back in seconds; data changes don't.

> [!question]- 🟡 Q27. Walk me through renaming a column with zero downtime.
> Expand: add the new column as nullable. Deploy code that writes both columns. Backfill old rows in small batches. Deploy code that reads the new column. Finally, once nothing uses the old column, stop writing it and drop it. Each step is backward compatible and can be rolled back.

> [!question]- 🟡 Q28. How do tools like gh-ost or pt-online-schema-change work?
> They create an empty copy of the table with the new schema, copy rows over in small throttled chunks, and keep the copy in sync with live writes via triggers or the binlog. When it's caught up they swap the tables with a quick atomic rename, so the original table is never locked for long.

> [!question]- 🔴 Q29. A simple ALTER TABLE on the posts table took the site down. Why?
> On a huge table, the ALTER can take a lock that blocks writes while it rewrites the table — for minutes or hours. Every request that writes posts piles up, connections run out, and the app goes down. Use an online migration tool, or the database's online DDL options, and test on a production-sized copy first.

---

## 09 · Rollbacks & Immutable Infrastructure

[Read the notes →](../09-rollbacks-immutable-infra.md)

> [!question]- 🟢 Q30. What is immutable infrastructure?
> We never change running servers. Each change produces a new versioned image, and we replace servers with it. This removes configuration drift, and rollback is just redeploying the previous image.

> [!question]- 🟡 Q31. What are your options to undo a bad release, from fastest to slowest?
> Flip a feature flag off — seconds. Route traffic back to the old version in blue-green or canary — seconds. Redeploy the previous image — minutes. Roll forward with a fix when rollback isn't possible, for example after data changes.

> [!question]- 🔴 Q32. You rolled back to the previous image but users still see corrupted profiles. Why?
> Rollback restores code, not data. The bad release wrote bad rows or ran a migration, and those changes are still there. We need a data fix — a repair script or restore from backup — and in future, backward-compatible migrations and validation to limit damage.
