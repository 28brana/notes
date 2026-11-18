# Q&A · Big Data Processing

[← Back to Big Data Processing](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · Batch vs Stream

[Read the notes →](../01-batch-vs-stream.md)

> [!question]- 🟢 Q1. What is the difference between batch and stream processing?
> Batch processes a bounded chunk of data — like yesterday's logs — all at once on a schedule. Stream processes an unbounded flow of events one by one as they arrive. Batch gives high throughput and simple re-runs but results are hours old; streaming gives results in seconds but is harder to build.

> [!question]- 🟡 Q2. Why does every aggregation in a stream need a window?
> A stream never ends, so "total views" has no final answer. A window cuts the stream into time buckets, like the last 5 minutes, so the engine can produce a result per bucket and then free the state.

> [!question]- 🟡 Q3. What is micro-batching and where does it fit?
> Micro-batching runs tiny batch jobs every second or so, like classic Spark Structured Streaming. It feels near real-time to users while reusing batch machinery. It's fine when ~1 second latency is OK, but not for millisecond needs.

> [!question]- 🔴 Q4. Product wants a "trending posts" list and a monthly creator payout report. How do you build them?
> Trending needs freshness, so I'd stream view events from Kafka into Flink with a sliding 5-minute window and write the top posts to Redis. Payouts need exact numbers and can wait, so I'd run a nightly or monthly batch job over the full event history in the warehouse. Different freshness needs, different pipelines.

---

## 02 · Change Data Capture

[Read the notes →](../02-change-data-capture.md)

> [!question]- 🟢 Q5. What is CDC and how does log-based CDC work?
> CDC turns every insert, update and delete in a database into an event. Log-based CDC tools like Debezium read the database's own change log — the Postgres WAL or MySQL binlog — and publish each row change to Kafka in order, with almost no extra load on the database.

> [!question]- 🟡 Q6. Why is log-based CDC better than polling an updated_at column?
> Polling misses deletes because the row is gone, can miss a row that changed twice between polls, and adds query load and delay. Reading the log catches every change, including deletes, in commit order.

> [!question]- 🔴 Q7. Search results sometimes show posts that were already deleted. The app writes to Postgres then Elasticsearch. What's wrong and how do you fix it?
> That's the dual-write problem: if the Elasticsearch write fails or races with another write, the two drift apart. I'd make the app write only to Postgres and use CDC to stream changes into Elasticsearch. The consumer upserts and deletes by post ID, so it's idempotent and follows the DB in order.

---

## 03 · MapReduce

[Read the notes →](../03-mapreduce.md)

> [!question]- 🟢 Q8. Explain MapReduce with an example.
> To count views per post across 10 TB of logs, mappers read chunks in parallel and emit (postId, 1). The shuffle groups all pairs with the same postId onto one reducer, and reducers sum them. Failed tasks are simply re-run because the input files never change.

> [!question]- 🔴 Q9. One reducer takes 10x longer than the others. Why, and what do you do?
> That's data skew — a hot key, like a viral post, sends far more data to one reducer. Fixes: use a combiner to pre-sum on mappers, or salt the hot key into several sub-keys, aggregate them, then combine the partial results.

---

## 04 · ETL Pipelines

[Read the notes →](../04-etl-pipelines.md)

> [!question]- 🟢 Q10. What's the difference between ETL and ELT?
> ETL transforms data in a separate engine before loading it into the warehouse. ELT loads raw data first, then transforms it with SQL inside the warehouse. ELT is the modern default because cloud warehouses are powerful and you keep raw data to re-transform later.

> [!question]- 🟡 Q11. What makes a data pipeline idempotent and why does it matter?
> Running it twice for the same day gives the same result — for example, overwrite the day's partition instead of appending. It matters because pipelines get retried and backfilled all the time, and appending on retry creates duplicate rows.

> [!question]- 🔴 Q12. The daily revenue dashboard showed the same number for three days and nobody noticed. What went wrong and how do you prevent it?
> The pipeline probably failed or loaded nothing, and there was no monitoring. I'd add freshness checks (alert if the latest partition is older than expected), row-count and null checks, and alerts on job failures from the orchestrator like Airflow.

---

## 05 · Data Lakes

[Read the notes →](../05-data-lakes.md)

> [!question]- 🟢 Q13. What is a data lake and what does schema-on-read mean?
> A data lake is raw data of any format stored in cheap object storage like S3. Schema-on-read means files are stored as-is and the structure is applied by whoever reads them, at query time, instead of being enforced when writing.

> [!question]- 🟡 Q14. Queries on our S3 data lake are very slow. What would you check?
> First, file format and size — millions of small JSON files are slow, so compact into large Parquet files. Second, partitioning — partition by date so queries skip days they don't need. Columnar Parquet also lets queries read only the columns they use.

---

## 06 · Data Warehousing

[Read the notes →](../06-data-warehousing.md)

> [!question]- 🟢 Q15. What's the difference between OLTP and OLAP?
> OLTP is the app's database: many small, fast reads and writes on a few rows, row-oriented. OLAP is analytics: few big queries scanning and aggregating millions of rows, usually on a columnar warehouse. They're tuned in opposite ways, so we keep them separate.

> [!question]- 🟡 Q16. Why is columnar storage faster for analytics?
> An analytics query like SUM(views) needs only a few columns. Columnar storage reads just those columns instead of whole rows, and similar values in a column compress very well, so far less data is read from disk.

> [!question]- 🟡 Q17. What is a star schema?
> A central fact table holds events with numbers and IDs, like each post view with watch time. Small dimension tables around it describe the IDs — users, posts, dates, countries. Analysts join the fact to dimensions, which is simple and fast.

---

## 07 · Data Lakehouse

[Read the notes →](../07-data-lakehouse.md)

> [!question]- 🟢 Q18. What is a lakehouse?
> It's a data lake with a table format like Delta Lake or Iceberg on top. The format keeps a transaction log of which files make up each table version, giving ACID writes, updates, deletes and time travel on cheap object storage, so BI and ML can share one copy of data.

> [!question]- 🟡 Q19. Compare a data lake, a warehouse and a lakehouse.
> A lake is cheapest and holds any data, but has no transactions and is slower for BI. A warehouse is structured, fast and ACID, but costlier and proprietary. A lakehouse keeps open files on cheap storage and adds warehouse-like ACID and schemas through a table format.

---

## 08 · Lambda Architecture

[Read the notes →](../08-lambda-architecture.md)

> [!question]- 🟢 Q20. What is Lambda architecture?
> It runs a batch layer that recomputes accurate results over all history and a speed layer that covers recent data with low latency. A serving layer merges them at query time, and each batch run overwrites the speed layer's approximate numbers.

> [!question]- 🟡 Q21. What's the biggest downside of Lambda?
> You write and maintain the same logic twice, in a batch system and a streaming system. The two drift apart over time and produce different numbers, and you run double the infrastructure.

> [!question]- 🔴 Q22. The live view count and the next-day report show different numbers for the same post. Why might that be in a Lambda setup?
> The speed layer is approximate — it may double-count on retries or drop late events — while batch recomputes exactly. The two codebases may also have drifted. That's expected until the batch overwrites the real-time view; if it persists, the logic in the two layers differs.

---

## 09 · Kappa Architecture

[Read the notes →](../09-kappa-architecture.md)

> [!question]- 🟢 Q23. What is Kappa architecture and how do you reprocess data in it?
> Kappa uses only a streaming pipeline, with the event log as the source of truth. To fix a bug or recompute, you start a new version of the job reading from the beginning of the log into a new table, then switch reads to it once it catches up.

> [!question]- 🟡 Q24. When would you still choose Lambda over Kappa?
> When you need heavy historical batch work like ML training or huge joins that are slow to replay through a stream, when you can't keep long log retention, or when your stream engine can't give exactly-once results.

---

## 10 · Streaming Engines

[Read the notes →](../10-streaming-engines.md)

> [!question]- 🟢 Q25. Explain tumbling, sliding and session windows.
> Tumbling windows are fixed and don't overlap, like every 5 minutes. Sliding windows are fixed-size but overlap, like the last 5 minutes refreshed every minute. Session windows group a user's activity and close after a gap of inactivity, like 30 minutes.

> [!question]- 🟡 Q26. What is the difference between event time and processing time, and what is a watermark?
> Event time is when something happened; processing time is when the engine sees it. They differ due to delays and offline devices, so analytics should use event time. A watermark is the engine's signal that all events up to time T have probably arrived, so it can close windows up to T.

> [!question]- 🔴 Q27. A Flink job crashes and restarts. How does it avoid double-counting, and what can still go wrong?
> It restores state from the last checkpoint and rewinds Kafka to the offsets saved in that checkpoint, so state reflects each event exactly once. But events after the checkpoint are re-processed, so if the sink isn't transactional or idempotent, outputs written since the checkpoint can be duplicated.
