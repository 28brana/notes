# 📚 Study Notes

My personal study notes, organized by subject. Each subject has its own folder with an index (`README.md`), numbered topic notes, and an `assets/` folder for diagrams and images.

## Subjects

| Subject | Description | Notes |
|---|---|---|
| [System Design](system-design/README.md) | Designing scalable, reliable distributed systems — 18 chapters | 159 + Q&A |
| [Computer Networks](computer-networks/README.md) | Networking for full stack developers: HTTP, browser, APIs, auth, deployment, debugging | 9 + Q&A |
| [DSA](dsa/README.md) | LeetCode practice list (questions only), from [LeetCode Patterns](https://seanprashad.com/leetcode-patterns/) | 179 problems |
| [DBMS](dbms/README.md) | SQL, data modeling, normalization, transactions, locking, indexing, SQL interview problems | 23 + Q&A |
| [Infrastructure](infrastructure/README.md) | Linux, Nginx, Docker, Kubernetes, AWS, VPC, Terraform, CI/CD, monitoring, incidents | 24 + Q&A |
| [Node.js Backend](nodejs-backend/README.md) | Event loop, async, streams, workers, memory + Express, auth, DBs, Redis, jobs, testing, production | 25 + Q&A |
| [Frontend](frontend/README.md) | Browser rendering, HTML/CSS, JavaScript, TypeScript, web security, React, performance, frontend system design | 25 + Q&A |

## Repository structure

```
notes/
├── README.md                  ← you are here (master index)
├── templates/
│   └── topic-template.md      ← copy this to start a new note
├── system-design/
│   ├── README.md              ← chapter index (start here)
│   ├── 01-what-is-system-design.md
│   ├── 02-must-know-concepts.md   ← one-page map of the whole course
│   ├── core-concepts/         ← scalability, availability, reliability, SPOF, latency, hashing, CAP
│   ├── networking/  load-balancing/  api-fundamentals/  communication-patterns/
│   ├── caching/  databases/  database-scaling/  storage-systems/
│   ├── architectural-patterns/  microservices-patterns/  distributed-systems/
│   ├── distributed-transactions/  data-structures/  big-data/
│   ├── deployment/  observability/  security/
│   │   └── (each chapter: README.md, NN-topic.md notes, questions/ with Q&A + practice sheet)
│   ├── questions/             ← Q&A index for all chapters
│   └── assets/                ← images/diagrams for this subject
├── dbms/  infrastructure/  nodejs-backend/  frontend/
│   └── (each: README.md, NN-topic.md notes, questions/ with Q&A + practice sheet per part)
├── dsa/
│   └── README.md          ← 179 LeetCode problems with links + checkboxes
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
