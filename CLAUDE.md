# CLAUDE.md — How to write notes in this repo

Personal study notes for **interview prep**, viewed in **Obsidian** (also readable on GitHub).
The user gives a topic → you create the note (+ Q&A file) and update the indexes.
Add useful things the user didn't ask for if an interviewer would expect them.

**Gold-standard examples — copy their style:**
- `system-design/02-scalability.md` (topic note)
- `system-design/questions/02-scalability-qa.md` (Q&A)

---

## Language

- **Simple, spoken English** — like explaining to an interviewer, not a textbook.
- Short paragraphs: **2–3 lines max**, one idea each.
- Use concrete numbers and real examples (users, req/s, ms, GB).
- Explain jargon the first time it appears.
- Never copy text from reference sites — learn the structure, write it in our own words.

## Structure of a topic note

Tell it as a **story**: one running example (e.g. a social media app) that grows, and each section fixes the next problem.

```
# NN · Topic Name

> [!info] Topic info
> **Subject:** … · **Priority:** 🔥 High · **Status:** ✅ done · **Last reviewed:** YYYY-MM-DD
>
> [← Back to <Subject>](README.md) · [Interview questions →](questions/NN-topic-qa.md)

> [!abstract] In one line
> One-sentence definition with the key words in **bold**.

---
## 1. The problem          ← start with a relatable scenario, not a definition
## 2. … numbered sections  ← each solves the next problem in the story
---
## Summary                 ← table (step → what it fixes) + [!success] key takeaway
## References
```

- Separate every `##` section with `---`.
- Numbered `##` sections; `###` for sub-parts.

## Visual elements (Obsidian)

| Use | For |
|---|---|
| ` ```mermaid ` flowchart LR | **Every architecture / flow.** Never ASCII-art diagrams. Use emojis in nodes (👥 🖥️ 🗄️ ⚡ ⚖️ 📬 ⚙️ 📦). Keep edge labels simple — no quotes or parentheses. |
| `> [!tip] 🎤 Say it like this` | 2–4 sentence answer to say out loud in an interview. One per major concept. |
| `> [!warning]` | Traps, downsides, things that get harder |
| `> [!note]` | Important side point / clarification |
| `> [!important]` | Golden rules |
| `> [!example]` | Analogies, cheat sheets, quick worked examples |
| `> [!question]` | Questions to ask the interviewer |
| `> [!failure]` | Mistakes to avoid |
| `> [!success]` | Key takeaways / "in real life" advice |
| **✅ Pros / ❌ Cons** | Bold label + short bullet list (not giant tables) |
| Tables | Comparisons (A vs B) and summaries only. Bold the first column. |

Don't overdo it: roughly a callout or diagram every section, not every paragraph.

## Q&A file (`<subject>/questions/NN-topic-qa.md`)

```
# Q&A · Topic Name

[← Back to <Subject>](../README.md) · [Read the notes →](../NN-topic.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

> [!question]- 🟢 Q1. Question?
> Short answer you can say out loud (2–4 sentences).
```

- `[!question]-` (with the dash) = folded by default.
- ~12–15 questions, ordered 🟢 → 🟡 → 🔴. Include scenario questions ("site is slow after adding servers — why?").

## After creating a note — always

1. Add/update the row in the subject's `README.md` (topic table, status ✅ / 🚧 / 📝).
2. Add/update the row in `<subject>/questions/README.md`.
3. Update the note count and folder tree in the root `README.md`.
4. Link the new note from related notes (e.g. the core-concepts table in `system-design/01-what-is-system-design.md`).
5. Don't commit unless asked. Commit format: `notes(<subject>): add <topic>`.

## Conventions

- Folders: `kebab-case` per subject, each with `README.md`, `questions/`, `assets/`.
- Files: `NN-topic-name.md` — number = reading order.
- Images go in the subject's `assets/` and are linked relatively.
