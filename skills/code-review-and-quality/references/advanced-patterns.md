# Advanced Review Patterns

Optional workflow variants that apply to specific review contexts — not every review.

## Dead Code Hygiene

After refactoring or implementation, check for orphaned code:

1. Identify code that is now unreachable or unused.
2. List it explicitly.
3. **Ask before deleting:** "Should I remove these now-unused elements: [list]?"

Do not leave dead code around — it confuses future readers and agents. Do not silently delete things you're unsure about.

```
DEAD CODE IDENTIFIED:
- formatLegacyDate() in src/utils/date.ts — replaced by formatDate()
- OldTaskCard component in src/components/ — replaced by TaskCard
- LEGACY_API_URL constant in src/config.ts — no remaining references
→ Safe to remove these?
```

## Dependency Discipline

Before adding any dependency:

1. Does the existing stack solve this? Often yes.
2. How large is the dependency? Check bundle impact.
3. Is it actively maintained? Check last commit, open issues.
4. Does it have known vulnerabilities? Run `npm audit` or equivalent.
5. What's the license? Must be compatible with the project.

**Rule:** Prefer standard library and existing utilities over new dependencies. Every dependency is a liability.

## Multi-Model Review Pattern

Use different models for different review perspectives to catch blind spots:

```
Model A writes the code
    ↓
Model B reviews for correctness and architecture
    ↓
Model A addresses the feedback
    ↓
Human makes the final call
```

Example prompt for a review agent:
```
Review this change for correctness, security, and project conventions.
Spec: [X]. Change should [Y].
Flag issues as Critical, Important, or Suggestion.
```
