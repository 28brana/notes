# 05 · CDN (Content Delivery Network)

> [!info] Topic info
> **Subject:** System Design · **Chapter:** Caching · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Caching](README.md) · [Interview questions →](questions/caching-qa.md)

> [!abstract] In one line
> A **CDN** is a network of **edge servers** around the world that cache your content **close to users**, cutting **latency** and taking load off your **origin** servers.

---

## 1. The problem

Our social media app runs in one region: **Mumbai**. A user in New York opens a profile with 20 images.

Each round trip to Mumbai is ~**250 ms**. Images and JS load slowly. And every image download for **50 million users** hits our own servers and bandwidth bill.

> [!note] Recap
> [Scalability](../core-concepts/01-scalability.md) (section 13) introduced CDNs briefly. Here's how they work and what interviewers probe.

---

## 2. How it works

The CDN has servers (**PoPs** — points of presence) in hundreds of cities. DNS or [anycast](../load-balancing/04-anycast-routing.md) sends each user to the **nearest** one.

```mermaid
flowchart LR
    U[👥 User in New York] --> E[🌍 CDN edge New York]
    E -- hit ~20 ms --> U
    E -- miss --> O[🖥️ Origin in Mumbai]
    O -- file + cache headers --> E
```

- **Hit**: the edge has the file → served in ~20 ms. Origin never sees the request.
- **Miss**: the edge fetches from the **origin** (your server or S3 bucket), caches it, then serves it.
- How long the edge keeps it is set by HTTP headers like `Cache-Control: max-age=86400`.

> [!example] The win
> 1 million image views, 95% hit ratio → only **50,000** reach your origin. Users get the file from their own city.

---

## 3. Push vs pull CDN

| Type | How content gets to the edge | Good for |
|---|---|---|
| **Pull** | Edge fetches from origin on the **first miss** | Most websites — zero effort, the default |
| **Push** | You **upload** content to the CDN ahead of time | Big files you know will be hot (video releases, game patches) |

---

## 4. What to put on a CDN

- **Static files**: images, videos, JS, CSS, fonts — the main use.
- **Public dynamic pages** with short TTLs (a public post page cached for 30 s).
- **Not** personalized or private data (your home feed, your DMs) — unless the cache key includes the user, which kills the hit ratio.

Modern CDNs also give **DDoS protection**, **TLS termination** near the user, and **edge compute** (small functions running at the edge).

---

## 5. Invalidating CDN content

Edges are spread across the world, so clearing them is slow and awkward.

- **TTL** — content expires on its own.
- **Purge API** — tell the CDN to drop a URL. Takes seconds to minutes to reach every edge.
- **Versioned URLs** (best) — `logo.v2.png` or `app.3f9a.js`. Cache them for a year; a new deploy just uses a new URL. See [Cache Invalidation](04-cache-invalidation.md).

> [!tip] 🎤 Say it like this
> "A CDN caches static content on edge servers close to users, so a user in New York gets images in 20 ms instead of 250 ms from our origin, and most requests never reach us. I'd use a pull CDN with long TTLs and versioned file names, so a deploy never needs a purge. Personalized data stays off the CDN."

**✅ Pros**
- Much lower latency for global users.
- Huge cut in origin load and bandwidth cost.
- Absorbs traffic spikes and DDoS attacks.

**❌ Cons**
- Stale content until TTL or purge.
- Cost per GB served; harder debugging (which edge served what?).
- Little help for personalized or write-heavy data.

> [!failure] Interview traps
> - Caching **private** pages on a shared CDN — one user sees another's data. Use `Cache-Control: private` for those.
> - Relying on **purges** for every deploy instead of versioned URLs.
> - Forgetting that the first user in each region still pays the **miss** (cold edge).

---

## Summary

- CDN = cache at the network edge, close to users.
- **Pull** is the default; **push** for large, known-hot content.
- Cache static files with long TTLs + versioned URLs; keep private data off it.

> [!success] Key takeaway
> Any design that serves images, video or static files to users worldwide should put a CDN in front — say it early in the interview.
