# 07 · Clock Synchronization

> [!info] Topic info
> **Subject:** System Design · **Chapter:** Distributed Systems · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Distributed Systems](README.md) · [Interview questions →](questions/distributed-systems-qa.md)

> [!abstract] In one line
> Every machine's clock **drifts**; protocols like **NTP** keep them roughly in sync (ms off), but never perfectly — so **don't use wall-clock time to order events** across machines.

---

## 1. The problem

Two servers in our app both receive edits to the same post. Server A stamps its write `10:00:00.120`. Server B stamps its write `10:00:00.100` — but B's clock is **50 ms behind**, so B's write actually happened **later**.

With last-write-wins by timestamp, the **newer** edit is thrown away. Silent data loss. 😬

---

## 2. Why clocks disagree

- **Drift** — quartz crystals tick at slightly different speeds with temperature. Can be ~**17 seconds per day** in the worst case unsynced (Google's spec assumes 200 ppm).
- **Sync jumps** — when NTP corrects a clock, time can **jump forward or even backward**.
- **Leap seconds, VM pauses, bad config** — all cause surprises.

### Two kinds of clocks on every machine
| Clock | Use it for | Don't use it for |
|---|---|---|
| **Wall clock** (time-of-day) | Showing time to users, logs | Measuring durations — it can jump back |
| **Monotonic clock** | Timeouts, measuring how long something took | Comparing across machines — meaningless |

---

## 3. How sync works — NTP

```mermaid
flowchart LR
    A[⚛️ Atomic clock or GPS] --> S1[🖥️ NTP stratum 1]
    S1 --> S2[🖥️ NTP stratum 2]
    S2 --> M[🖥️ Our servers]
```

**NTP (Network Time Protocol)**: your server asks a time server for the time, measures the **round trip**, and assumes the reply took half of it. Then it slowly adjusts the local clock.

- Typical accuracy: **a few ms** in a data center, **tens of ms** over the internet.
- **PTP (Precision Time Protocol)** — hardware-assisted, gets to **microseconds**. Used in finance and modern clouds.

### Google TrueTime (Spanner)
GPS + atomic clocks in every data center. Instead of one time, it returns an **interval** `[earliest, latest]` (about 7 ms wide). Spanner **waits out the uncertainty** before committing, so timestamp order matches real order.

> [!tip] 🎤 Say it like this
> "Clocks on different machines drift and NTP only gets them within a few milliseconds, and sometimes it jumps them backward. So I never order events across machines by wall-clock time. For durations and timeouts I use the monotonic clock; for ordering I use logical clocks or a single sequencer. Spanner is the exception — it uses GPS and atomic clocks and waits out the uncertainty."

> [!failure] Interview traps
> - Using wall-clock timestamps for **last-write-wins** across nodes — silently drops newer writes.
> - Measuring timeouts with the wall clock — an NTP jump can fire them early or never.
> - Assuming "we run NTP, so clocks are in sync" — sync is **approximate**, and a broken NTP setup fails silently.

---

## Summary

- Clocks **drift**; NTP keeps them within ms, PTP within µs — never exact.
- **Monotonic** clock for durations, **wall** clock only for display.
- To order events → [Logical Clocks](08-logical-clocks.md), [Lamport](09-lamport-timestamps.md), [Vector clocks](10-vector-clocks.md).

> [!success] Key takeaway
> Physical time across machines is a **guess with error bars**. Use it for humans, not for deciding which write wins.
