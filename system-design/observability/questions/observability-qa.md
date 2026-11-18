# Q&A · Observability

[← Back to Observability](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · The Three Pillars

[Read the notes →](../01-three-pillars.md)

> [!question]- 🟢 Q1. What is the difference between monitoring and observability?
> Monitoring checks things you predicted could break — like error rate above 1% — and tells you *that* something is wrong. Observability is being able to ask new questions of your system's data and find *why*, even for problems you never saw before. Monitoring is one part of observability.

> [!question]- 🟢 Q2. What are the three pillars of observability, and what is each good for?
> Metrics are cheap numbers over time, great for dashboards and alerts — they tell you *something is wrong*. Traces follow one request across services and show *where* time is spent. Logs record individual events in detail and explain *why* it failed.

> [!question]- 🟡 Q3. How do logs, metrics and traces work together during an incident?
> A metric alert fires — say feed errors jump to 5%. I open a slow or failed trace for that endpoint and see most of the time goes into one Redis call. Then I filter logs by that trace ID and see "connection pool exhausted". The shared trace ID is what lets me jump between them.

> [!question]- 🔴 Q4. You're designing a new service. How would you make it observable from day one?
> I'd add OpenTelemetry for traces and metrics, track RED metrics with latency histograms, and write structured JSON logs with the trace ID on every line. I'd define an SLO, alert on burn rate, and ship a dashboard and runbook with the service. All of it goes to central backends so it survives pods dying.

---

## 02 · Logging

[Read the notes →](../02-logging.md)

> [!question]- 🟢 Q5. Why use structured (JSON) logs instead of plain text?
> JSON logs have named fields — level, service, trace ID, user ID, event — so the log system can filter and group them like a database. With plain text you need fragile regex, and every developer formats lines differently.

> [!question]- 🟡 Q6. What is a correlation or request ID, and how does it work across services?
> It's a unique ID created at the edge, like the API gateway, for each request. Every service passes it on in a header and adds it to every log line. Then one search for that ID shows the whole story of one request across all services.

> [!question]- 🟡 Q7. What should you never log, and how do you enforce it?
> Never log passwords, tokens, API keys, card numbers or personal data like emails and phone numbers unless masked. Logs get copied to many systems and read by many people, so a leak is a security and legal problem. I enforce it with automatic redaction in the logging library or pipeline, not just code review.

> [!question]- 🔴 Q8. Your logging bill doubled this quarter. How do you cut it without losing debugging power?
> First find which services and log lines produce the most volume. Then turn off DEBUG in prod, stop logging inside loops, keep all errors but sample successful-request logs at 1–10%, and move anything counted from logs into metrics. Finally, shorten hot retention to 1–2 weeks and archive the rest to cheap object storage.

---

## 03 · Log Aggregation

[Read the notes →](../03-log-aggregation.md)

> [!question]- 🟢 Q9. Why do we need log aggregation?
> With hundreds of servers and autoscaling, you can't SSH into each box to grep, and logs on a deleted server are gone. Aggregation ships all logs to one central store so you can search across every service in seconds.

> [!question]- 🟡 Q10. Why put Kafka between the log agents and Elasticsearch?
> It's a buffer. During a traffic spike or when the store is slow, logs wait in Kafka instead of being dropped or slowing down the app. It also lets several consumers read the same logs, like the search store and a security system.

> [!question]- 🟡 Q11. ELK vs Loki — what's the trade-off?
> Elasticsearch indexes every word, so search is very fast and flexible but storage and indexing are expensive. Loki only indexes labels like service and environment and keeps compressed raw logs in object storage, so it's much cheaper but text search is slower — you filter by labels first.

---

## 04 · Metrics

[Read the notes →](../04-metrics.md)

> [!question]- 🟢 Q12. What's the difference between a counter, a gauge and a histogram?
> A counter only goes up, like total requests — you look at its rate. A gauge goes up and down, like memory used or queue length. A histogram puts values into buckets, like request durations, so you can calculate percentiles such as p99.

> [!question]- 🟡 Q13. What are the RED and USE methods?
> RED is for services: Rate of requests, Errors, and Duration (latency). USE is for resources like CPU, disks or a connection pool: Utilization, Saturation, and Errors. Together they cover most of what you need to watch.

> [!question]- 🔴 Q14. A developer added `user_id` as a label on a metric and Prometheus crashed. Why?
> Every unique label combination is its own time series kept in memory. With millions of users, that one label creates millions of series — a cardinality explosion — and Prometheus runs out of memory. Labels must have a small, fixed set of values; per-user detail belongs in logs or traces.

---

## 05 · Distributed Tracing

[Read the notes →](../05-distributed-tracing.md)

> [!question]- 🟢 Q15. What are a trace and a span?
> A trace is the full journey of one request through the system, with one trace ID. A span is one step inside it — an HTTP call, DB query or cache lookup — with its own timing and a link to its parent span. Together the spans form a tree you can see as a waterfall.

> [!question]- 🟡 Q16. How does the trace ID travel from one service to another?
> Through context propagation: every outgoing call carries the trace context in a header, usually the W3C `traceparent` header with the trace ID, parent span ID and a sampled flag. Each service reads it, creates a child span, and passes it on. For queues, the context goes in the message headers.

> [!question]- 🟡 Q17. Head-based vs tail-based sampling?
> Head-based decides at the start of the request, for example keep 1% at random — cheap, but you keep only 1% of your errors. Tail-based decides after the trace finishes, so you can keep all errors and slow requests, but the collector must buffer every span for a while. Many teams combine a low head rate with tail rules for errors.

> [!question]- 🔴 Q18. Your traces stop at the Kafka queue and the worker's spans show up as a separate trace. What's wrong?
> The trace context isn't being propagated through the message. The producer must inject the `traceparent` into the Kafka message headers, and the consumer must extract it and start its span as a child. Same fix for any proxy or library that drops the header.

---

## 06 · Alerts and Monitoring

[Read the notes →](../06-alerts-and-monitoring.md)

> [!question]- 🟢 Q19. Should you alert on symptoms or on causes?
> Page on symptoms users feel — error rate and latency against the SLO. High CPU might be totally harmless, and cause-based alerts miss problems you didn't predict. Causes like CPU and disk go on dashboards to help debug.

> [!question]- 🟡 Q20. What is an SLO burn-rate alert?
> Burn rate is how fast you're using your error budget; 1× means you'd use it exactly by the end of the 30-day window. You page when the burn is fast — like 14× over an hour, which empties the budget in about two days — and open a ticket for slow burns. It ties alerts to real user impact instead of random thresholds.

> [!question]- 🔴 Q21. Your on-call gets 150 alerts a night and missed a real outage. How do you fix it?
> That's alert fatigue. I'd review every alert: if it needed no action, delete it or make it a ticket. Move paging to SLO burn-rate alerts on user symptoms, group related alerts, add short "for" durations to skip blips, and make sure every remaining page has a runbook.

---

## 07 · Dashboards and Runbooks

[Read the notes →](../07-dashboards-and-runbooks.md)

> [!question]- 🟢 Q22. What goes into a good runbook?
> What the alert means and the user impact, links to the dashboard, logs and traces, quick checks like "was there a recent deploy", concrete mitigation steps such as rollback or failover, and who to escalate to. It should let any on-call engineer stop the bleeding without knowing the service deeply.
