# System Design · Observability

[← Back to System Design](../README.md) · [Interview Q&A →](questions/observability-qa.md) · [Practice questions 📝](questions/practice-questions.md)

How you see what's happening inside a running system — logs, metrics, traces, alerts — and find out *why* it broke. Interviewers expect "how would you monitor this?" at the end of almost every design.

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [The Three Pillars](01-three-pillars.md) | Monitoring vs observability; metrics say *what*, traces *where*, logs *why* | 🔥 High |
| 02 | [Logging](02-logging.md) | Structured JSON logs, levels, trace IDs, no secrets/PII, cost control | 🔥 High |
| 03 | [Log Aggregation](03-log-aggregation.md) | Agents → buffer → store (ELK / Loki) with hot/cold retention | ⚡ Medium |
| 04 | [Metrics](04-metrics.md) | Counter/gauge/histogram, RED & USE, Prometheus pull, cardinality trap | ⚡ Medium |
| 05 | [Distributed Tracing](05-distributed-tracing.md) | Traces & spans, `traceparent` propagation, OpenTelemetry, head vs tail sampling | 🔥 High |
| 06 | [Alerts and Monitoring](06-alerts-and-monitoring.md) | Symptom-based alerts, SLO burn rate, page vs ticket, alert fatigue | ⚡ Medium |
| 07 | [Dashboards and Runbooks](07-dashboards-and-runbooks.md) | Health at a glance + step-by-step guide for each alert | 💤 Low |
