# System Design · Deployment Patterns

[← Back to System Design](../README.md) · [Interview Q&A →](questions/deployment-qa.md) · [Practice questions 📝](questions/practice-questions.md)

Most outages are caused by **our own changes**, not hardware. This chapter is about shipping new code many times a day **without** taking the site down — and getting back fast when a release goes wrong.

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [Deployment Strategies](01-deployment-strategies.md) | Overview + decision table: recreate, rolling, blue-green, canary, shadow | 🔥 High |
| 02 | [CI/CD](02-ci-cd.md) | Automated pipeline from commit → build → test → deploy | 🔥 High |
| 03 | [Rolling Deployments](03-rolling-deployments.md) | Replace servers a few at a time, no extra fleet | 🔥 High |
| 04 | [Blue-Green Deployments](04-blue-green.md) | Two full environments, flip traffic in one switch | 🔥 High |
| 05 | [Canary Releases](05-canary-releases.md) | Send a small % of traffic to the new version, watch metrics, then grow | 🔥 High |
| 06 | [Feature Flags](06-feature-flags.md) | Ship code turned off, turn it on at runtime — deploy ≠ release | 🔥 High |
| 07 | [A/B Testing](07-ab-testing.md) | Experiment: split users, measure a metric, check statistical significance | ⚡ Medium |
| 08 | [Schema Migrations](08-schema-migrations.md) | Change the database without downtime: expand → migrate → contract | 🔥 High |
| 09 | [Rollbacks & Immutable Infrastructure](09-rollbacks-immutable-infra.md) | Build once, replace not patch, rollback = redeploy the previous image | ⚡ Medium |
