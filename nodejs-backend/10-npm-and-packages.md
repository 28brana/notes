# 10 · npm and Packages

> [!info] Topic info
> **Subject:** Node.js Backend · **Part:** Node.js Core · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Node.js Backend](README.md) · [Interview questions →](questions/nodejs-core-qa.md)

> [!abstract] In one line
> **package.json** declares dependency **ranges** (`^`, `~`), **package-lock.json** pins the **exact** tree, and **`npm ci`** installs exactly that lock for reproducible builds.

---

## 1. The problem

"Works on my machine." Your laptop installed `lodash 4.17.20`, CI installed `4.17.21` a week later, and a subtle change breaks production. We need **the same versions everywhere**.

---

## 2. Semantic versioning

`MAJOR.MINOR.PATCH` → `2.4.1`. **Major** = breaking change, **minor** = new feature (compatible), **patch** = bug fix.

| Range | Means | Example matches |
|---|---|---|
| **`^1.2.3`** (npm default) | Same **major** | `1.2.3` → `1.9.9`, not `2.0.0` |
| **`~1.2.3`** | Same **minor** | `1.2.3` → `1.2.9`, not `1.3.0` |
| **`1.2.3`** | Exact | only `1.2.3` |

> [!note] Zero major
> For `0.x`, caret is stricter: `^0.2.3` allows only `0.2.x`, because pre-1.0 minors can break things.

---

## 3. Lockfile and `npm ci`

- **package.json** — what you *allow* (ranges).
- **package-lock.json** — what was *actually installed*: exact version + integrity hash for every package, including dependencies of dependencies. **Commit it.**

| | **`npm install`** | **`npm ci`** |
|---|---|---|
| **Reads** | package.json, updates lock if needed | lock only |
| **Modifies lock** | Yes | Never — fails if lock and package.json disagree |
| **node_modules** | Updates in place | Deletes and reinstalls clean |
| **Use in** | Development | **CI and Docker builds** |

---

## 4. Dependency types and safety

- **dependencies** — needed at runtime (`express`).
- **devDependencies** — only for build/test (`jest`, `typescript`). Skip in prod with `npm ci --omit=dev`.
- **peerDependencies** — "I need the host app to provide this" (a React plugin needs `react`).
- **npx** — run a package's binary without installing it globally.

**Supply-chain risk:** an average app pulls in hundreds of transitive packages. Run `npm audit`, keep the lockfile, review new deps, and use tools like Dependabot/Renovate for updates.

> [!tip] 🎤 Say it like this
> "package.json holds version ranges — caret allows minor and patch updates, tilde only patches. The lockfile pins the exact version of every package in the tree, so I always commit it. In CI and Docker I use npm ci, which installs exactly what's in the lock and fails if it's out of sync, so every build is reproducible."

> [!failure] Interview traps
> - Not committing `package-lock.json` (or adding it to `.gitignore`).
> - Using `npm install` in CI — can silently change versions.
> - Putting build tools in `dependencies` → bloated production images.

---

## Summary

- Semver: `^` = same major, `~` = same minor.
- Lockfile = exact tree; commit it.
- `npm ci` for CI/Docker; `--omit=dev` for prod.
- Audit dependencies — they're your attack surface.

> [!success] Key takeaway
> Ranges in package.json, exact truth in the lockfile, `npm ci` in pipelines — that's how you get identical builds everywhere.
