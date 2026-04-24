# Review Culture

How to conduct yourself as a reviewer — honesty, disagreements, and rationalizations to reject.

## Honesty in Review

- Do not rubber-stamp. "LGTM" without evidence helps no one.
- Do not soften real issues. "This might be a minor concern" when it's a production bug is dishonest.
- Quantify problems when possible. "This N+1 query adds ~50ms per item" beats "this could be slow."
- Push back on approaches with clear problems. Sycophancy is a failure mode in reviews.
- Accept override gracefully. If the author has full context and disagrees, defer to their judgment.

## Handling Disagreements

Apply this hierarchy to resolve disputes:

1. Technical facts and data override opinions and preferences.
2. Style guides are the absolute authority on style matters.
3. Software design is evaluated on engineering principles, not personal preference.
4. Codebase consistency is acceptable if it doesn't degrade overall health.

Do not accept "I'll clean it up later" — deferred cleanup rarely happens. Require cleanup before submission unless it's a genuine emergency. If surrounding issues can't be addressed in this change, require a filed bug with self-assignment.

## Common Rationalizations and Red Flags

Reject these rationalizations:

- *"It works, that's good enough"* — unreadable, insecure, or architecturally wrong code creates compounding debt.
- *"I wrote it, so I know it's correct"* — authors are blind to their own assumptions.
- *"We'll clean it up later"* — later never comes. The review is the quality gate.
- *"AI-generated code is probably fine"* — AI code needs more scrutiny, not less.
- *"The tests pass, so it's good"* — tests miss architecture, security, and readability.

Watch for these red flags:

- PRs merged without review, or "LGTM" without evidence
- Reviews that only check if tests pass
- Security-sensitive changes without security-focused review
- Large PRs "too big to review properly" — split them
- Bug-fix PRs with no regression test
- Review comments without severity labels
- Accepting "I'll fix it later"
