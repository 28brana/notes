# Q&A · Communication Patterns

[← Back to Communication Patterns](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · Long Polling

[Read the notes →](../01-long-polling.md)

> [!question]- 🟢 Q1. What is long polling and how is it different from short polling?
> Short polling asks the server every few seconds and usually gets an empty answer. With long polling the server holds the request open until there's new data or a timeout, then the client immediately reconnects. So you get near real-time updates with far fewer wasted requests.

> [!question]- 🟡 Q2. When would you still choose long polling today?
> As a fallback when WebSockets or SSE are blocked by proxies or old infrastructure, or for rare updates like "your export is ready". It works over plain HTTP everywhere. For frequent updates it's too costly, because every message needs a new request.

> [!question]- 🔴 Q3. Your long-polling clients keep getting 504 errors after 60 seconds. What's wrong?
> A proxy or load balancer between client and server has an idle timeout around 60 seconds and kills the held request. Set the server's hold time below that, like 25–30 seconds, and return an empty response so the client reconnects cleanly. Also make sure the client sends a cursor so no events are lost between requests.

---

## 02 · Server-Sent Events

[Read the notes →](../02-server-sent-events.md)

> [!question]- 🟢 Q4. What is SSE and when would you use it over WebSockets?
> SSE is a one-way stream from server to client over a normal HTTP response with the text/event-stream type. I use it when only the server pushes — notifications, live scores, streaming AI responses. It's plain HTTP, so it's simpler and works with existing auth and load balancers.

> [!question]- 🟡 Q5. How does SSE handle a dropped connection without losing events?
> The browser's EventSource reconnects automatically. Each event has an id, and on reconnect the browser sends Last-Event-ID. The server replays events after that id, as long as it keeps recent events in something like Redis or a log.

> [!question]- 🔴 Q6. Your SSE events arrive in big bursts every few seconds instead of instantly. Why?
> Something in between is buffering the response — often a reverse proxy like Nginx with buffering on, or compression middleware. Disable buffering for the stream endpoint and flush after each event. Also send a heartbeat comment every 15–30 seconds so idle connections aren't closed.

---

## 03 · WebSockets

[Read the notes →](../03-websockets.md)

> [!question]- 🟢 Q7. How does a WebSocket connection get established?
> The client sends a normal HTTP request with an Upgrade: websocket header. The server replies 101 Switching Protocols, and from then on the same TCP connection carries WebSocket frames in both directions. There's no HTTP overhead per message after that.

> [!question]- 🟡 Q8. Why are WebSockets harder to scale than a REST API?
> WebSocket servers are stateful — each one holds live connections, roughly tens of thousands per server. You need load balancers that support upgrades and long idle timeouts, heartbeats to find dead connections, and graceful draining on deploys. And users on different servers can't talk without something connecting the servers.

> [!question]- 🔴 Q9. Alice is connected to server 1 and Bob to server 3. How does Alice's chat message reach Bob?
> Server 1 first saves the message to the database, then publishes it to a pub/sub channel — Redis Pub/Sub or Kafka. Server 3 is subscribed to channels for its connected users, receives it, and pushes it down Bob's socket. If Bob is offline, he fetches it from the database when he reconnects.

> [!question]- 🔴 Q10. After every deploy, your chat servers get overloaded for a few minutes. What's happening?
> It's a reconnect storm. Restarting a server drops all its connections at once, and every client reconnects at the same moment, also re-syncing missed messages. Fix it with jittered exponential backoff on clients, draining servers gradually during rolling deploys, and rate-limiting new connections.

---

## 04 · Webhooks

[Read the notes →](../04-webhooks.md)

> [!question]- 🟢 Q11. What is a webhook?
> It's a reverse API — instead of me polling a provider, the provider sends an HTTP POST to a URL I registered when an event happens. For example, a payment provider calls me when a payment succeeds. It's server-to-server and event-driven.

> [!question]- 🟡 Q12. How do you receive webhooks safely?
> Verify the HMAC signature on the raw body and check the timestamp to block replays. Store the event, return 200 quickly, and process it from a queue. Make the handler idempotent using the event ID, because providers retry and send duplicates.

> [!question]- 🔴 Q13. Customers report being charged once but their order is marked paid twice and gets two receipts. Why?
> The webhook handler is probably slow — it does all the work inline, the provider times out and retries, and the event is processed twice. Respond 2xx fast and do the work async, and dedupe on the event ID with a unique constraint in the same transaction as the update.

> [!question]- 🔴 Q14. You're the provider. How do you design reliable webhook delivery to thousands of customers?
> Put events in a queue and have delivery workers POST them, retrying with exponential backoff for up to a few days. After that, move them to a dead letter queue and alert the customer. Isolate customers so one slow endpoint doesn't block others, sign every payload, and block internal IPs to prevent SSRF.

---

## 05 · WebRTC

[Read the notes →](../05-webrtc.md)

> [!question]- 🟢 Q15. What are signaling, STUN and TURN in WebRTC?
> Signaling exchanges connection details between peers — you build it, usually with WebSockets. STUN tells a device its public IP and port so peers can connect directly. TURN relays the media when a direct path is impossible, like behind strict firewalls.

> [!question]- 🟡 Q16. Why does a 20-person video call need an SFU instead of pure peer-to-peer?
> In a mesh, each person uploads their video to all 19 others, which kills upload bandwidth and CPU. With an SFU, each person uploads once to a server that forwards streams to everyone. It scales much better and lets the server pick the quality for each receiver.

---

## 06 · Message Queues

[Read the notes →](../06-message-queues.md)

> [!question]- 🟢 Q17. Why would you put a message queue between two services?
> To make the work async and decoupled. The API enqueues a job and responds immediately, while workers process it at their own pace. Spikes wait in the queue instead of crashing workers, and if a worker is down, messages just wait.

> [!question]- 🟡 Q18. What happens if a worker crashes halfway through a message?
> The message was never acked, so after the visibility timeout it becomes visible again and another worker picks it up. That means it can be processed twice, so the consumer must be idempotent.

> [!question]- 🔴 Q19. Your queue depth keeps growing all day and never goes down. What do you check?
> Compare produce rate and consume rate — if producers add more per second than consumers finish, the backlog grows forever. Check for slow downstream calls, a poison message being retried, or too few workers. Fix by autoscaling workers on queue depth, fixing the slow dependency, or adding a DLQ for failing messages.

> [!question]- 🔴 Q20. The same video gets transcoded twice at the same time. Why?
> The visibility timeout is shorter than the processing time. The job takes 5 minutes but the timeout is 30 seconds, so the message reappears and a second worker grabs it. Raise the timeout above the worst-case time or extend it while working, and make the job idempotent.

---

## 07 · Pub/Sub

[Read the notes →](../07-pub-sub.md)

> [!question]- 🟢 Q21. What's the difference between a message queue and pub/sub?
> In a queue each message goes to one consumer — it's a task to do. In pub/sub every subscriber gets its own copy — it's an event that happened. Real systems often combine them: each subscriber group gets every event, and workers inside the group share the load.

> [!question]- 🟡 Q22. When a user creates a post, five services must react. Would you call them directly or use pub/sub?
> Pub/sub. The post service publishes one PostCreated event and feed, notifications, search and analytics each subscribe. The post service stays fast and doesn't need code changes when a new consumer appears. The trade-off is eventual consistency and harder tracing.

> [!question]- 🔴 Q23. You used Redis Pub/Sub for order events and some orders were never shipped. Why?
> Redis Pub/Sub is fire-and-forget — it doesn't store messages. If the shipping service was restarting or disconnected, it simply missed those events. For business events that must not be lost, use a durable system like Kafka or SNS with SQS, where messages wait until consumed.

---

## 08 · Message Brokers

[Read the notes →](../08-message-brokers.md)

> [!question]- 🟢 Q24. What's the main difference between Kafka and RabbitMQ?
> RabbitMQ is a queue broker — messages are delivered, acked and deleted, with flexible routing. Kafka is a distributed log — messages are kept for a retention period, and each consumer group tracks its own offset. So Kafka supports replay and many independent readers at very high throughput.

> [!question]- 🟡 Q25. How does Kafka scale consumers and keep ordering?
> A topic is split into partitions, and order is guaranteed only within a partition. Messages with the same key go to the same partition, so all events for one user stay in order. In a consumer group each partition goes to one consumer, so the max parallelism equals the number of partitions.

> [!question]- 🔴 Q26. You added 10 more consumers to your Kafka group but throughput didn't change. Why?
> The topic probably has fewer partitions than consumers — extra consumers sit idle because each partition is read by only one consumer in a group. Increase the partitions. Also check for a hot key sending most traffic to one partition.

> [!question]- 🔴 Q27. Kafka, RabbitMQ or SQS for a notification system on AWS? Justify.
> If I just need reliable background jobs, SQS with SNS for fan-out is simplest — fully managed, no ops. If many teams need the same event stream with replay and analytics, Kafka. RabbitMQ fits when I need complex routing or priorities and I'm fine running it myself.

---

## 09 · Delivery Semantics

[Read the notes →](../09-delivery-semantics.md)

> [!question]- 🟢 Q28. Explain at-most-once, at-least-once and exactly-once.
> At-most-once acks before processing, so a crash loses the message. At-least-once acks after processing, so a crash causes a redelivery and a duplicate. Exactly-once means each message has its effect once — in practice that's at-least-once plus idempotent consumers.

> [!question]- 🟡 Q29. How does offset commit timing affect delivery in Kafka?
> Committing before processing gives at-most-once — a crash skips messages. Committing after processing gives at-least-once — a crash reprocesses them. Auto-commit runs on a timer regardless of processing, so for important data I commit manually after the work is done.

> [!question]- 🔴 Q30. Kafka says it supports exactly-once. Can your consumer that writes to Postgres and sends emails rely on that?
> No. Kafka's exactly-once covers Kafka-to-Kafka pipelines using idempotent producers and transactions. Once you write to an external database or send an email, you need your own idempotency — a dedupe table or unique key in the same DB transaction, or storing the offset with the result.

> [!question]- 🔴 Q31. Order events arrive out of order — "paid" after "shipped". How do you handle it?
> Use the order ID as the partition key so one order's events stay in one partition and in order. Also add a version or sequence number to each event, and have the consumer ignore events older than the current state. Retries can still reorder things, so the version check is the real safety net.

---

## 10 · Dead Letter Queues

[Read the notes →](../10-dead-letter-queues.md)

> [!question]- 🟢 Q32. What is a dead letter queue and why do you need one?
> It's a side queue where messages go after failing a set number of times. Without it, a poison message — like broken JSON — gets retried forever, wasting resources and in ordered systems blocking everything behind it.

> [!question]- 🟡 Q33. Should every failed message be retried before going to the DLQ?
> No. Transient errors like timeouts or 503s should be retried with backoff. Permanent errors like invalid data will never succeed, so send them to the DLQ right away with the error reason attached.

> [!question]- 🔴 Q34. You fixed a bug and redrove 50k messages from the DLQ, and now some customers got duplicate emails. What went wrong?
> Some of those messages had partly succeeded before failing — for example the email was sent but a later step crashed. Redriving reran the whole thing. Consumers must be idempotent, tracking which side effects already happened per message ID, before you redrive.
