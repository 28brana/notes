# Big Data Processing · Practice Questions

[← Back to Big Data Processing](../README.md) · [Q&A with answers →](big-data-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](big-data-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**27 questions.**

---

## 01 · Batch vs Stream

- [ ] 🟢 Q1. What is the difference between batch and stream processing?
- [ ] 🟡 Q2. Why does every aggregation in a stream need a window?
- [ ] 🟡 Q3. What is micro-batching and where does it fit?
- [ ] 🔴 Q4. Product wants a "trending posts" list and a monthly creator payout report. How do you build them?

---

## 02 · Change Data Capture

- [ ] 🟢 Q5. What is CDC and how does log-based CDC work?
- [ ] 🟡 Q6. Why is log-based CDC better than polling an updated_at column?
- [ ] 🔴 Q7. Search results sometimes show posts that were already deleted. The app writes to Postgres then Elasticsearch. What's wrong and how do you fix it?

---

## 03 · MapReduce

- [ ] 🟢 Q8. Explain MapReduce with an example.
- [ ] 🔴 Q9. One reducer takes 10x longer than the others. Why, and what do you do?

---

## 04 · ETL Pipelines

- [ ] 🟢 Q10. What's the difference between ETL and ELT?
- [ ] 🟡 Q11. What makes a data pipeline idempotent and why does it matter?
- [ ] 🔴 Q12. The daily revenue dashboard showed the same number for three days and nobody noticed. What went wrong and how do you prevent it?

---

## 05 · Data Lakes

- [ ] 🟢 Q13. What is a data lake and what does schema-on-read mean?
- [ ] 🟡 Q14. Queries on our S3 data lake are very slow. What would you check?

---

## 06 · Data Warehousing

- [ ] 🟢 Q15. What's the difference between OLTP and OLAP?
- [ ] 🟡 Q16. Why is columnar storage faster for analytics?
- [ ] 🟡 Q17. What is a star schema?

---

## 07 · Data Lakehouse

- [ ] 🟢 Q18. What is a lakehouse?
- [ ] 🟡 Q19. Compare a data lake, a warehouse and a lakehouse.

---

## 08 · Lambda Architecture

- [ ] 🟢 Q20. What is Lambda architecture?
- [ ] 🟡 Q21. What's the biggest downside of Lambda?
- [ ] 🔴 Q22. The live view count and the next-day report show different numbers for the same post. Why might that be in a Lambda setup?

---

## 09 · Kappa Architecture

- [ ] 🟢 Q23. What is Kappa architecture and how do you reprocess data in it?
- [ ] 🟡 Q24. When would you still choose Lambda over Kappa?

---

## 10 · Streaming Engines

- [ ] 🟢 Q25. Explain tumbling, sliding and session windows.
- [ ] 🟡 Q26. What is the difference between event time and processing time, and what is a watermark?
- [ ] 🔴 Q27. A Flink job crashes and restarts. How does it avoid double-counting, and what can still go wrong?
