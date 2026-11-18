# Observability · Practice Questions

[← Back to Observability](../README.md) · [Q&A with answers →](observability-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](observability-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**22 questions.**

---

## 01 · The Three Pillars

- [ ] 🟢 Q1. What is the difference between monitoring and observability?
- [ ] 🟢 Q2. What are the three pillars of observability, and what is each good for?
- [ ] 🟡 Q3. How do logs, metrics and traces work together during an incident?
- [ ] 🔴 Q4. You're designing a new service. How would you make it observable from day one?

---

## 02 · Logging

- [ ] 🟢 Q5. Why use structured (JSON) logs instead of plain text?
- [ ] 🟡 Q6. What is a correlation or request ID, and how does it work across services?
- [ ] 🟡 Q7. What should you never log, and how do you enforce it?
- [ ] 🔴 Q8. Your logging bill doubled this quarter. How do you cut it without losing debugging power?

---

## 03 · Log Aggregation

- [ ] 🟢 Q9. Why do we need log aggregation?
- [ ] 🟡 Q10. Why put Kafka between the log agents and Elasticsearch?
- [ ] 🟡 Q11. ELK vs Loki — what's the trade-off?

---

## 04 · Metrics

- [ ] 🟢 Q12. What's the difference between a counter, a gauge and a histogram?
- [ ] 🟡 Q13. What are the RED and USE methods?
- [ ] 🔴 Q14. A developer added `user_id` as a label on a metric and Prometheus crashed. Why?

---

## 05 · Distributed Tracing

- [ ] 🟢 Q15. What are a trace and a span?
- [ ] 🟡 Q16. How does the trace ID travel from one service to another?
- [ ] 🟡 Q17. Head-based vs tail-based sampling?
- [ ] 🔴 Q18. Your traces stop at the Kafka queue and the worker's spans show up as a separate trace. What's wrong?

---

## 06 · Alerts and Monitoring

- [ ] 🟢 Q19. Should you alert on symptoms or on causes?
- [ ] 🟡 Q20. What is an SLO burn-rate alert?
- [ ] 🔴 Q21. Your on-call gets 150 alerts a night and missed a real outage. How do you fix it?

---

## 07 · Dashboards and Runbooks

- [ ] 🟢 Q22. What goes into a good runbook?
