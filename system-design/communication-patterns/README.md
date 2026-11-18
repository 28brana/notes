# System Design · Communication Patterns

[← Back to System Design](../README.md) · [Interview Q&A →](questions/communication-patterns-qa.md) · [Practice questions 📝](questions/practice-questions.md)

How services and clients talk beyond simple request/response: real-time push to users, callbacks between servers, and async messaging with queues and brokers. Almost every design interview (chat, feeds, notifications, payments) needs one of these.

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [Long Polling](01-long-polling.md) | Server holds the HTTP request open until there is new data | ⚡ Medium |
| 02 | [Server-Sent Events](02-server-sent-events.md) | One-way server push over one long HTTP response | ⚡ Medium |
| 03 | [WebSockets](03-websockets.md) | Persistent two-way connection; scale with gateways + pub/sub | 🔥 High |
| 04 | [Webhooks](04-webhooks.md) | Provider POSTs events to your URL — verify, ack fast, be idempotent | 🔥 High |
| 05 | [WebRTC](05-webrtc.md) | Peer-to-peer audio/video with signaling, STUN and TURN | 💤 Low |
| 06 | [Message Queues](06-message-queues.md) | Buffer tasks so one worker handles each, later | 🔥 High |
| 07 | [Pub/Sub](07-pub-sub.md) | Publish one event, every subscriber gets a copy | 🔥 High |
| 08 | [Message Brokers](08-message-brokers.md) | Kafka (log) vs RabbitMQ / SQS (queue) — when to pick which | 🔥 High |
| 09 | [Delivery Semantics](09-delivery-semantics.md) | At-most / at-least / exactly-once, offsets, idempotency, ordering | 🔥 High |
| 10 | [Dead Letter Queues](10-dead-letter-queues.md) | Park messages that keep failing so the queue keeps flowing | ⚡ Medium |
