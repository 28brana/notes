# Q&A · Latency vs Throughput vs Bandwidth

[← Back to Core Concepts](../README.md) · [Read the notes →](../05-latency-throughput-bandwidth.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

> [!question]- 🟢 Q1. What's the difference between latency, throughput, and bandwidth?
> Latency is how long one request takes, measured in ms. Throughput is how many requests or bytes we actually handle per second. Bandwidth is the maximum data the link could carry per second. On a highway: latency is one car's travel time, bandwidth is the number of lanes, throughput is cars actually passing per hour.

> [!question]- 🟢 Q2. Can a link have high bandwidth and high latency at the same time?
> Yes — they're independent. Geostationary satellite internet can give 100 Mbps but has ~600 ms round trips because the signal travels to space and back. A truck full of hard drives is the extreme case: petabytes of bandwidth, days of latency.

> [!question]- 🟢 Q3. Why is throughput always lower than bandwidth?
> Bandwidth is the theoretical ceiling. Real throughput loses some to protocol headers, packet loss and retransmits, TCP window limits, and slow servers at either end. A 1 Gbps link typically gives a few hundred to ~900 Mbps of real throughput.

> [!question]- 🟢 Q4. My internet is 100 Mbps. How long does a 1 GB file take to download?
> Networks are sold in bits, files in bytes, and 1 byte is 8 bits. 100 Mbps is 12.5 MB/s, so 1 GB takes about 80 seconds, not 10 — and a bit more in practice due to overhead.

> [!question]- 🟢 Q5. What's the difference between latency, processing time, and response time?
> Processing time is the time the server spends actually working. Latency is time spent waiting — on the network or in queues. Response time is what the user feels end to end: network there, queue wait, processing, and network back. If processing is 20 ms but response time is 900 ms, the problem isn't your code.

> [!question]- 🟢 Q6. Why use percentiles like p99 instead of average latency?
> Averages hide the slow requests. If 95 requests take 50 ms and 5 take 1.5 s, the average is ~125 ms and looks fine, but 1 in 20 users waits 1.5 s. p50 shows the typical user, p95 and p99 show the tail — often your heaviest, most valuable users.

> [!question]- 🟡 Q7. Where does latency come from on a network request?
> Four parts: propagation (distance, limited by the speed of light — ~200 km per ms in fiber), transmission (pushing the bits onto the wire), processing (routers, LBs, servers), and queuing (waiting behind other work). On a new HTTPS connection you also pay DNS, TCP, and TLS handshakes — several round trips before the first byte.

> [!question]- 🟡 Q8. Give some rough latency numbers you keep in your head.
> L1 cache about 1 ns, RAM about 100 ns, SSD random read about 100 µs, a round trip inside one data center about 0.5 ms, a Redis call about 1 ms, an HDD seek about 10 ms, and a cross-continent round trip 100–200 ms. The point is the orders of magnitude: memory beats disk by ~1,000x, and cross-region calls are hundreds of times slower than local ones.

> [!question]- 🟡 Q9. What is Little's Law and how do you use it?
> Concurrency = throughput × latency. At 2,000 req/s and 200 ms per request, there are 400 requests in flight; if a server handles 50 at once, I need 8 servers plus headroom. It also shows that if latency jumps to 1 s, the same traffic needs 2,000 concurrent slots — 5x the capacity — which is how a slow DB exhausts thread and connection pools.

> [!question]- 🟡 Q10. Why does latency shoot up as utilization approaches 100%?
> Requests arrive randomly, so a busy server builds a queue. Response time grows roughly like work time ÷ (1 − utilization): 2x at 50% busy, 10x at 90%, 100x at 99%. It's a hockey-stick curve, so I keep servers around 60–70% at peak, auto-scale before the knee, and shed load instead of letting queues grow unbounded.

> [!question]- 🟡 Q11. How do you improve latency vs throughput vs bandwidth?
> Latency: CDN, caching, closer regions, fewer and parallel round trips, connection reuse, indexes, and moving side work async. Throughput: horizontal scaling, fixing the bottleneck stage, batching, more parallelism, and queues. Bandwidth: compression, modern image formats, right-sized media, adaptive video bitrate, pagination, smaller payloads, and CDN offload.

> [!question]- 🟡 Q12. Give an example where improving throughput hurts latency.
> Batching. If a producer waits 20 ms to collect 500 messages into one write, throughput goes way up because there's much less per-message overhead, but each message now waits up to 20 ms. Kafka's `linger.ms` is exactly this knob. Running servers at 90% CPU is another — more work per server, but much worse latency.

> [!question]- 🔴 Q13. What is tail amplification, and how do you deal with it?
> When a request fans out to many services, it's as slow as the slowest one. If a page calls 100 services and each is slow 1% of the time, 1 − 0.99¹⁰⁰ ≈ 63% of page loads hit a slow call — so one service's rare p99 becomes the common case. I'd use per-call timeouts with partial results, hedged requests to a second replica after the p95, and cut the fan-out by pre-computing or batching.

> [!question]- 🔴 Q14. Users in India say the feed takes 3 seconds, but US users say it's instant. Server time is 50 ms. What's going on?
> It's network latency, not code. Mumbai to US East is ~200 ms per round trip, and a new HTTPS connection needs DNS, TCP, and TLS — 4–5 round trips ≈ 1 s — then the app makes several sequential API calls. I'd put static and media on a CDN, terminate TLS at an edge near India, reuse connections with HTTP/2 or HTTP/3, combine the calls into one feed endpoint, and eventually add an India region with a feed cache.

> [!question]- 🔴 Q15. Our dashboard shows average latency of 120 ms, but users keep complaining the app is slow. How do you investigate?
> First I'd look at p95, p99 and p99.9 instead of the average, and split by endpoint, region, and success vs error — fast errors can make averages look good. Then I'd check real user monitoring, since server-side numbers miss DNS, TLS and mobile networks. Finally I'd trace slow requests to find whether it's a slow dependency, fan-out tail, GC pauses, or queuing at peak.

> [!question]- 🔴 Q16. Every evening at 9 PM the app gets slow, but nothing is failing and CPU is at 92%. What's happening and what do you do?
> The servers are near saturation, so requests are queuing — at 92% busy, response time can be ~10x the idle time even though no errors show up. Short term I'd scale out, ideally pre-scaling before the known peak, and add bounded queues with load shedding. Longer term I'd set auto-scaling to trigger around 60–70% CPU and reduce work per request with caching.

> [!question]- 🔴 Q17. Video uploads take 15 minutes for a 1 GB file. How would you make them faster?
> A 10 Mbps home uplink moves only 1.25 MB/s, so 1 GB is ~13 minutes no matter what the server does. I'd compress or transcode on the device where possible, upload in 5–10 MB chunks in parallel directly to object storage with presigned URLs, and make it resumable so a dropped connection doesn't restart from zero. Parallel streams also help far-away users where the bandwidth-delay product limits a single TCP stream.
