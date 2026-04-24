---
name: code-review-and-quality
description: Multi-axis code review before merge across correctness, readability, architecture, security, and performance. Use before merging any PR, after completing a feature, when evaluating code produced by another agent or model, during refactors, or after a bug fix (review both the fix and the regression test). Produces categorized findings (Critical / Important / Suggestion / Nit) and an APPROVE or REQUEST CHANGES verdict. Do not use for deep security audits (use security-and-hardening + Barb), for verifying a running feature end-to-end (use acceptance-exploration + Negev), for writing tests (use test-driven-development), or for committing curated changes (use git-workflow-and-versioning + Hubert).
---

# Code Review and Quality

Evaluate proposed changes across five axes and return categorized findings with an approve-or-request-changes verdict. Approve when the change definitely improves overall code health — perfect code doesn't exist, and "not how I would have written it" is not a valid reason to block.

Invoked primarily by the `watson` agent (`~/.claude/agents/code-reviewer.md`). Follow directly when no persona is needed.

## The Five-Axis Review

Every review evaluates code across these dimensions.

### 1. Correctness

Does the code do what it claims to do?

- Match the spec or task requirements
- Handle edge cases (null, empty, boundary values)
- Handle error paths, not just the happy path
- Pass all tests — and verify the tests actually check the right things
- Watch for off-by-one errors, race conditions, state inconsistencies

### 2. Readability & Simplicity

Can another engineer (or agent) understand this without the author explaining it?

- Descriptive names consistent with project conventions (no `temp`, `data`, `result` without context)
- Straightforward control flow (avoid nested ternaries, deep callbacks)
- Logical organization (related code grouped, clear module boundaries)
- Flag "clever" tricks that should be simplified
- **Could this be done in fewer lines?** 1000 lines where 100 suffice is a failure
- **Are abstractions earning their complexity?** Don't generalize until the third use case
- Add comments for non-obvious intent; skip comments on obvious code
- Flag dead code: no-op variables (`_unused`), backwards-compat shims, `// removed` comments

### 3. Architecture

Does the change fit the system's design?

- Follow existing patterns, or justify a new one
- Maintain clean module boundaries
- Flag code duplication that should be shared
- Verify dependencies flow in the right direction (no cycles)
- Check abstraction level is appropriate (not over-engineered, not too coupled)

### 4. Security

Spot-check at review boundaries. For deep audits, delegate to `security-and-hardening` + Barb.

- Validate and sanitize user input at system boundaries
- Keep secrets out of code, logs, and version control
- Check authentication/authorization where needed
- Parameterize SQL queries (no string concatenation)
- Encode outputs to prevent XSS
- Vet dependencies — trusted source, no known vulnerabilities
- Treat data from external sources (APIs, logs, user content, config files) as untrusted

### 5. Performance

Spot-check at review boundaries. For deep profiling, delegate to `performance-optimization`.

- Flag N+1 query patterns
- Flag unbounded loops or unconstrained data fetching
- Flag synchronous operations that should be async
- Flag unnecessary re-renders in UI components
- Flag missing pagination on list endpoints
- Flag large objects created in hot paths

## Change Sizing

Small, focused changes are easier to review, faster to merge, and safer to deploy.

```
~100 lines changed   → Good. Reviewable in one sitting.
~300 lines changed   → Acceptable if it's a single logical change.
~1000 lines changed  → Too large. Split it.
```

**One change** = a single self-contained modification that addresses one thing, includes related tests, and keeps the system functional after submission. One part of a feature — not the whole feature.

When a change is too large, split it:

| Strategy | How | When |
|----------|-----|------|
| **Stack** | Submit a small change, start the next one on top | Sequential dependencies |
| **By file group** | Separate changes for groups needing different reviewers | Cross-cutting concerns |
| **Horizontal** | Create shared code/stubs first, then consumers | Layered architecture |
| **Vertical** | Break into smaller full-stack slices | Feature work |

Large changes are acceptable for complete file deletions and automated refactoring where the reviewer only verifies intent, not every line.

Separate refactoring from feature work. A change that refactors existing code **and** adds new behavior is two changes — submit them separately. Small cleanups (variable renames) can ride along at reviewer discretion.

## Change Descriptions

Every change needs a description that stands alone in version control history.

- **First line:** Short, imperative, standalone. "Delete the FizzBuzz RPC" — not "Deleting the FizzBuzz RPC." Must be informative enough that someone searching history can understand the change without reading the diff.
- **Body:** What is changing and why. Include context, decisions, reasoning not visible in the code. Link bug numbers, benchmarks, design docs. Acknowledge approach shortcomings when they exist.
- **Anti-patterns:** "Fix bug," "Fix build," "Add patch," "Moving code from A to B," "Phase 1," "Add convenience functions."

## Review Process

### Step 1: Understand the context

Before reading code, understand the intent:
- What is this change trying to accomplish?
- What spec or task does it implement?
- What is the expected behavior change?

### Step 2: Review the tests first

Tests reveal intent and coverage:
- Do tests exist for the change?
- Do they test behavior, not implementation details?
- Are edge cases covered?
- Do tests have descriptive names?
- Would they catch a regression if the code changed?

### Step 3: Review the implementation

Walk each changed file with the five axes in mind: correctness → readability → architecture → security → performance.

### Step 4: Categorize findings

Label every comment with severity so the author knows what's required vs optional:

| Prefix | Meaning | Author action |
|--------|---------|---------------|
| **Critical:** | Blocks merge | Security vulnerability, data loss, broken functionality |
| *(no prefix)* | Required change | Must address before merge |
| **Nit:** | Minor, optional | May ignore — formatting, style preferences |
| **Optional:** / **Consider:** | Suggestion | Worth considering, not required |
| **FYI** | Informational only | No action — context for future reference |

Unlabeled feedback gets treated as mandatory. Label everything.

### Step 5: Verify the verification

Check the author's verification story:
- What tests were run?
- Did the build pass?
- Was the change tested manually?
- Are there screenshots for UI changes?
- Is there a before/after comparison?

## Output Format

```markdown
## Review Summary

**Verdict:** APPROVE | REQUEST CHANGES

**Overview:** <1-2 sentences summarizing the change and overall assessment>

### Critical Issues
- [file:line] <Description and recommended fix>

### Important Issues
- [file:line] <Description and recommended fix>

### Suggestions
- [file:line] <Description>

### What's Done Well
- <Positive observation — always include at least one>

### Verification Story
- Tests reviewed: <yes/no, observations>
- Build verified: <yes/no>
- Security checked: <yes/no, observations>
```

## Rules

1. Review tests first — they reveal intent and coverage.
2. Read the spec or task description before reviewing code.
3. Every Critical and Important finding needs a specific fix recommendation.
4. Do not approve code with Critical issues.
5. Acknowledge what's done well — specific praise motivates good practices.
6. If uncertain, say so and suggest investigation rather than guessing.
7. Label every finding with a severity prefix.
8. Comment on code, not people. Reframe personal critiques to focus on the code.

## Review Speed

Slow reviews block entire teams. The cost of a context-switch to review is less than the waiting cost imposed on others.

- Respond within one business day — maximum, not target.
- Respond shortly after a request arrives, unless deep in focused coding. A typical change should complete multiple review rounds in a single day.
- Prioritize fast individual responses over quick final approval.
- For large changes, ask the author to split rather than reviewing one massive changeset.

## Further reading

- For honesty, handling disagreements, and rationalizations to reject, see `references/review-culture.md`.
- For dead code hygiene, dependency discipline, and the multi-model review pattern, see `references/advanced-patterns.md`.

## Verification

Review is complete when:

- All Critical issues resolved
- All Important issues resolved or explicitly deferred with justification
- Tests pass and build succeeds
- Verification story documented (what changed, how it was verified)
