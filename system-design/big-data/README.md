# System Design · Big Data Processing

[← Back to System Design](../README.md) · [Interview Q&A →](questions/big-data-qa.md) · [Practice questions 📝](questions/practice-questions.md)

How to move, store and crunch huge amounts of data for analytics — reports, trending, ML. Interviewers ask this whenever a design has "analytics", "metrics", "real-time counts" or "dashboards".

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [Batch vs Stream](01-batch-vs-stream.md) | Process big chunks on a schedule vs each event as it arrives | 🔥 High |
| 02 | [Change Data Capture](02-change-data-capture.md) | Turn DB changes into an event stream by reading the WAL / binlog | ⚡ Medium |
| 03 | [MapReduce](03-mapreduce.md) | Map → shuffle → reduce, the model behind big batch jobs | 💤 Low |
| 04 | [ETL Pipelines](04-etl-pipelines.md) | Extract, transform, load — and why ELT is the modern default | ⚡ Medium |
| 05 | [Data Lakes](05-data-lakes.md) | Raw data of any type in cheap object storage, schema-on-read | ⚡ Medium |
| 06 | [Data Warehousing](06-data-warehousing.md) | Columnar OLAP database for fast analytics SQL | ⚡ Medium |
| 07 | [Data Lakehouse](07-data-lakehouse.md) | Lake files + table format = ACID and SQL on cheap storage | 💤 Low |
| 08 | [Lambda Architecture](08-lambda-architecture.md) | Batch layer + speed layer, merged at query time | ⚡ Medium |
| 09 | [Kappa Architecture](09-kappa-architecture.md) | Stream only; reprocess by replaying the log | 💤 Low |
| 10 | [Streaming Engines](10-streaming-engines.md) | Flink & co: windows, event time, watermarks, checkpoints | ⚡ Medium |
