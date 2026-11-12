# 📚 Study Notes

My personal study notes, organized by subject. Each subject has its own folder with an index (`README.md`), numbered topic notes, and an `assets/` folder for diagrams and images.

## Subjects

| Subject | Description | Notes |
|---|---|---|
| [System Design](system-design/README.md) | Designing scalable, reliable distributed systems | 1 |
| [Computer Networks](computer-networks/README.md) | Networking for full stack developers: HTTP, browser, APIs, auth, deployment, debugging | 9 + Q&A |

## Repository structure

```
notes/
├── README.md                  ← you are here (master index)
├── templates/
│   └── topic-template.md      ← copy this to start a new note
├── system-design/
│   ├── README.md              ← subject index + progress
│   ├── 01-fundamentals.md
│   └── assets/                ← images/diagrams for this subject
└── computer-networks/
    ├── README.md
    ├── 01-how-the-internet-works.md … 09-troubleshooting-and-performance.md
    ├── questions/                 ← interview Q&A, one file per topic
    ├── deep-dives/                ← optional theory (OSI, subnetting)
    └── assets/
```

## Conventions

- **Folders:** one per subject, `kebab-case` (e.g. `operating-systems/`).
- **Files:** `NN-topic-name.md` — the number sets the reading order (`01-`, `02-`, …).
- **New note:** copy [`templates/topic-template.md`](templates/topic-template.md), then add a row to the subject's `README.md`.
- **New subject:** create the folder with its own `README.md` and `assets/`, then add a row to the table above.
- **Images:** save into the subject's `assets/` folder and link relatively: `![diagram](assets/my-diagram.png)`.
- **Status tags** used in subject indexes: ✅ done · 🚧 in progress · 📝 planned.

## Commit messages

```
notes(system-design): add caching topic
fix(computer-networks): correct TCP handshake diagram
```
