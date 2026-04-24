---
name: git-workflow-and-versioning
description: Structure git workflow practices — atomic commits, trunk-based branching, descriptive messages, and change summaries. Use when making any code change that gets committed, when splitting a messy working tree, when naming a branch, when writing a commit message, or when cleaning up history. Invoked most often by the `hubert` agent, which layers persona-specific execution (secrets scanning, 70-char subject cap, result-line contract). Do not use for force-pushing, rebasing published history, or PR creation — those are out of scope.
---

# Git Workflow and Versioning

Treat commits as save points, branches as sandboxes, and history as documentation. With agents generating code at high speed, disciplined version control keeps changes reviewable, revertible, and comprehensible.

## Core principles

### Trunk-based development (recommended default)

Keep `main` always deployable. Work in short-lived feature branches that merge back within 1-3 days. Long-lived branches accumulate merge risk and delay integration; DORA research consistently links trunk-based flow to high-performing teams.

```
main ──●──●──●──●──●──●──●──●──●──  (always deployable)
        ╲      ╱  ╲    ╱
         ●──●─╱    ●──╱    ← short-lived feature branches (1-3 days)
```

- Dev branches are costs. Every day a branch lives, it accumulates merge risk.
- Release branches are acceptable for stabilizing a release while main moves forward.
- Prefer feature flags over long branches for incomplete work.

Teams on gitflow or long-lived branches can adapt the principles below. The commit discipline matters more than the specific branching model.

### Commit early, commit often

Commit each successful increment. Do not accumulate large uncommitted changes.

```
Work pattern: Implement slice → Test → Verify → Commit → Next slice
```

Commits are save points. If the next change breaks something, revert to the last known-good state instantly.

### Atomic commits

Each commit does one logical thing. Self-contained, reviewable, revertible.

```
Good: each commit is self-contained
  a1b2c3d feat: add task creation endpoint with validation
  d4e5f6g feat: add task creation form component
  h7i8j9k feat: connect form to API with loading state
  m1n2o3p test: cover task creation (unit + integration)

Bad: everything mixed
  x1y2z3a add task feature, fix sidebar, update deps, refactor utils
```

### Separate concerns

Do not mix formatting changes with behavior changes. Do not mix refactors with features. Each type of change gets its own commit — ideally its own PR.

Small cleanups (renaming a variable, fixing a typo) can ride along with a feature commit at reviewer discretion.

### Size changes

- ~100 lines: easy to review, easy to revert
- ~300 lines: acceptable for a single logical change
- ~1000 lines: split into smaller changes

See `code-review-and-quality` for strategies on splitting large changes.

## Commit message format

```
<type>: <short imperative description>

<optional body explaining why, not what>
```

Subject: imperative, present-tense, lowercase, no trailing period.

**Types:**

- `feat` — new feature
- `fix` — bug fix
- `refactor` — code change that neither fixes a bug nor adds a feature
- `test` — add or update tests
- `docs` — documentation only
- `chore` — tooling, dependencies, config

Body: explain *why*, not *what*. Reference the constraint, prior incident, or spec — not the files touched.

```
Good: explains intent
  feat: add email validation to registration endpoint

  Prevents invalid formats from reaching the database.
  Uses the same Zod pattern as auth.ts for consistency.

Bad: describes what's obvious from the diff
  update auth.ts
```

## Branching

```
main (always deployable)
  ├── feature/task-creation    ← one feature per branch
  ├── feature/user-settings    ← parallel work
  └── fix/duplicate-tasks      ← bug fixes
```

- Branch from `main` (or the team's default branch)
- Keep branches short-lived (merge within 1-3 days)
- Delete branches after merge
- Prefer feature flags over long-lived branches

**Branch naming:** `<prefix>/<kebab-description>`, ≤40 characters.

```
feature/task-creation
fix/duplicate-tasks
chore/update-deps
refactor/auth-module
```

For parallel work across multiple branches without switching, see `references/worktrees.md`.

## The save point pattern

```
Start → Make a change
         ├── Test passes? → Commit → Continue
         └── Test fails?  → Revert to last commit → Investigate
```

Never lose more than one increment. If an agent goes off the rails, `git reset --hard HEAD` (caller-authorized only) returns to the last successful state.

## Change summaries

After any modification, return a structured summary:

```
CHANGES MADE:
- <file>: <what changed and why>

THINGS I DIDN'T TOUCH (intentionally):
- <file or concern>: <reason out of scope>

POTENTIAL CONCERNS:
- <anything surprising — unexpected deletions, new deps, large line counts, hook warnings>
```

The "DIDN'T TOUCH" section is load-bearing. It shows scope discipline and protects against unsolicited renovation.

## Pre-commit hygiene

Before every commit:

```bash
git diff --staged                                                  # review what's staged
git diff --staged | grep -Ei 'password|secret|api_key|token|BEGIN PRIVATE KEY'  # secrets scan
npm test                                                           # or project equivalent
npm run lint                                                       # or project equivalent
npx tsc --noEmit                                                   # or project typecheck
```

Automate via git hooks (husky + lint-staged, pre-commit, etc.) when available. Do not bypass with `--no-verify` — fix the underlying issue.

## Handling generated files

- Commit generated files only if the project expects them (`package-lock.json`, Prisma migrations, etc.)
- Never commit build output (`dist/`, `.next/`), environment files (`.env`), or IDE-local config unless explicitly shared
- Keep a `.gitignore` that covers: `node_modules/`, `dist/`, `.env`, `.env.local`, `*.pem`

## Red flags

- Large uncommitted changes accumulating
- Commit messages like `fix`, `update`, `misc`
- Formatting changes mixed with behavior changes
- Committing `node_modules/`, `.env`, or build artifacts
- Long-lived branches diverging significantly from main
- Force-pushing to shared branches
- `--amend` on a commit already pushed

For debugging with git history (bisect, blame, log tricks), see `references/debugging-with-git.md`.

## Verification

- [ ] Commit does one logical thing
- [ ] Message follows `<type>: <description>`, explains the why
- [ ] Tests pass
- [ ] No secrets in the diff
- [ ] No formatting-only changes mixed with behavior changes
- [ ] `.gitignore` covers standard exclusions
