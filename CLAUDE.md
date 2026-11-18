# CLAUDE.md — How to write notes in this repo

Personal study notes for **interview prep**, viewed in **Obsidian** (also readable on GitHub).
The user gives a topic → you create the note (+ Q&A file) and update the indexes.
Add useful things the user didn't ask for if an interviewer would expect them.

**Gold-standard examples — copy their style:**
- `system-design/core-concepts/01-scalability.md` (topic note)
- `system-design/core-concepts/questions/01-scalability-qa.md` (Q&A)

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
4. Link the new note from related notes (e.g. the core-concepts table in `system-design/01-what-is-system-design.md`). If it lives in a sub-folder, also update that sub-folder's `README.md` and `questions/README.md`.
5. Don't commit unless asked. Commit format: `notes(<subject>): add <topic>`.

## Conventions

- Folders: `kebab-case` per subject, each with `README.md`, `questions/`, `assets/`.
- A subject can group related notes in a **chapter sub-folder** (e.g. `system-design/caching/`) with its own `README.md` and `questions/`; numbering restarts at `01` inside it.

## System Design chapters (short-note format)

`system-design/` follows a chapter roadmap (see its `README.md`). Except `core-concepts/` (long notes, one Q&A per topic), chapters use:

- **Short, interview-focused notes** — High priority ~90–150 lines, Medium ~70–110, Low ~45–80. No padding, no trivia.
- Info line adds `**Chapter:** <name>` and priority 🔥 High / ⚡ Medium / 💤 Low.
- **One Q&A file per chapter:** `<chapter>/questions/<chapter>-qa.md`, a `## NN · Topic` heading per topic, questions numbered continuously.
- **Practice sheet** `<chapter>/questions/practice-questions.md` (questions only, checkboxes) — generated from the Q&A file; regenerate after changing questions.
- Don't re-explain a topic another note covers — recap in 1–2 lines and link it.
- Files: `NN-topic-name.md` — number = reading order.
- Images go in the subject's `assets/` and are linked relatively.
