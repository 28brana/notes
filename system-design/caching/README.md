# System Design · Caching

[← Back to System Design](../README.md) · [Interview Q&A →](questions/caching-qa.md) · [Practice questions 📝](questions/practice-questions.md)

Caching is the first tool for making reads fast and cheap. Almost every design interview has a "where do you cache, and how do you keep it fresh?" moment.

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [What Is Caching](01-what-is-caching.md) | Fast copy of hot data; hit ratio and cache levels | 🔥 High |
| 02 | [Caching Strategies](02-caching-strategies.md) | Cache-aside, read-through, write-through, write-behind, write-around | 🔥 High |
| 03 | [Cache Eviction Policies](03-cache-eviction-policies.md) | What to drop when full — LRU, LFU, FIFO, TTL | 🔥 High |
| 04 | [Cache Invalidation](04-cache-invalidation.md) | Keeping the cache fresh — TTL, delete on write, versioned keys | 🔥 High |
| 05 | [CDN](05-cdn.md) | Edge servers that cache content close to users | 🔥 High |
| 06 | [Distributed Cache](06-distributed-cache.md) | Sharded, replicated shared cache — Redis vs Memcached, hot keys | 🔥 High |
| 07 | [Cache Stampede](07-cache-stampede.md) | Many misses hitting the DB at once, and how to stop it | ⚡ Medium |
| 08 | [Cache Warming](08-cache-warming.md) | Pre-loading hot data to avoid a cold cache | 💤 Low |
