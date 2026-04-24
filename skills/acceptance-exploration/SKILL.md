---
name: acceptance-exploration
description: Verify a finished feature works end-to-end before declaring done. Use after implementation and unit tests pass, when you need a pass/fail verdict on whether the feature actually works from a user's or caller's perspective — not just that tests are green. Works across browser apps, CLI tools, HTTP APIs, and agent/skill bundles at stage-appropriate depth (prototype / MVP / beta / GA). Produces a verdict with evidence (screenshots, transcripts, request logs). Do not use for writing tests (use test-driven-development), code review (use code-review-and-quality), or security audits (use security-and-hardening).
---

# Acceptance Exploration

Run the finished feature end-to-end and return a stage-appropriate pass/fail verdict with evidence. Tests verify code; acceptance exploration verifies the *feature*.

Invoked primarily by the `negev` agent (`~/.claude/agents/acceptance-explorer.md`). Follow directly when no persona is needed.

## Two required inputs

Every exploration needs both:

1. **Stage** — how deep to probe: `prototype` / `MVP` / `beta` / `GA`
2. **Surface** — how to drive and observe: `browser` / `cli` / `http-api` / `agent-skill` / `other`

Stop and ask if either is missing. Stage is semantic (what to check). Surface is mechanical (how to check).

Also confirm: the spec (what the feature should do) and launch instructions (how to start it).

## Stages

Deeper stages include everything from lighter stages. Checklists are semantic — apply them across all surfaces.

### Prototype — "the core idea demonstrates"
- Happy path runs start-to-finish without crashing
- Core value proposition is observable in the output
- No uncaught errors on the primary flow

### MVP — "the feature is usable"
All prototype checks, plus:
- Top 3 consumer flows complete successfully
- Error states produce useful output (no blank screens, silent exits, or raw stack traces leaked to end users)
- Transport layer clean (no 5xx, no non-zero exit on valid input)

### Beta — "it handles real use"
All MVP checks, plus:
- Edge cases: empty input, very long input, invalid input, duplicate/rapid invocations
- Error recovery: retries, degraded-dependency handling, timeouts
- Input-variant coverage across realistic consumer patterns

### GA — "production-ready"
All beta checks, plus:
- Performance meets any stated budget
- Observability: failures leave useful traces; non-generic error messages
- Graceful degradation (no silent failures, no infinite hangs)
- Surface-specific hardening (see surface reference)

## Surfaces

Pick the matching reference file and load only that one:

- **Browser app** → `references/surface-browser.md`
- **CLI tool** → `references/surface-cli.md`
- **HTTP API** → `references/surface-http-api.md`
- **Agent or skill** → `references/surface-agent-skill.md`
- **Other** (daemon, extension, MCP server, library without a public endpoint) — adapt method; document the surface description in the report

Each reference covers: probing tool, how to drive the surface, evidence to capture, and surface-specific hardening items per stage.

If a feature spans multiple surfaces (e.g., CLI + API), run acceptance against each separately and consolidate in the verdict.

## Method

1. **Load the spec.** Read the task, PRD, issue, or spec file.
2. **Confirm inputs.** Stage + surface + launch instructions. Stop and ask if any are missing.
3. **Load the surface reference.** Read only the one that matches.
4. **Launch.** Start the system. Verify baseline reachability before probing.
5. **Drive the flows.** Walk the stage checklist using the surface's probing tool. One flow at a time.
6. **Capture evidence as you go.** Save to `./acceptance-evidence/<YYYY-MM-DD-HHMM>/<flow-name>/` if running in a project directory.
7. **Score.** For each checklist item: PASS, FAIL, or BLOCKED. BLOCKED ≠ FAIL (BLOCKED means verification was prevented).
8. **Report.** Overall verdict + flow-by-flow results + evidence paths.

## Output format

```markdown
## Acceptance Report — <feature>, stage: <stage>, surface: <surface>

**Verdict:** PASS | FAIL | PARTIAL

**Overview:** <1-2 sentences>

### Flows verified
- [PASS] <flow name> — <what was verified>
- [FAIL] <flow name> — <what broke + evidence path + reproduction steps>
- [BLOCKED] <flow name> — <what prevented verification>

### Evidence
- <surface-appropriate paths>

### Observations beyond scope
- <things noticed but outside the stage checklist — optional>

### Blockers
- <what prevented full coverage, and what's needed to unblock — include only if PARTIAL>
```

## Rules

1. Report PASS only if every checklist item completes successfully.
2. Capture evidence as you go — no evidence, no verdict.
3. Report DEFERRED if the system won't launch; do not guess at behavior.
4. Don't upgrade stage or swap surfaces without the caller asking.
5. Don't modify the system under test. Observe and report.
6. Surface breaks with reproduction steps. Do not fix them.
7. For multi-surface features, run each surface separately and consolidate.
