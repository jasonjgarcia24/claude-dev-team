---
name: test-driven-development
description: Drives development with tests. Write a failing test before writing code that makes it pass; for bugs, reproduce with a test before attempting a fix (the Prove-It Pattern). Use when implementing any new logic or behavior, fixing any bug, modifying existing functionality, adding edge case handling, or making any change that could break existing behavior. Do NOT use for pure configuration changes, documentation updates, or static content changes with no behavioral impact. Covers the RED/GREEN/REFACTOR cycle, the test pyramid and test-size model, writing-good-tests patterns (state over interactions, DAMP over DRY, real over mocks, Arrange-Act-Assert), and common anti-patterns. For browser runtime verification, combine with the `browser-testing-with-devtools` skill.
---

# Test-Driven Development

Write a failing test before writing the code that makes it pass. For bug fixes, reproduce the bug with a test before attempting a fix. Tests are proof — "seems right" is not done. A codebase with good tests is an AI agent's superpower; a codebase without tests is a liability.

Invoked primarily by the `pepper` agent (`~/.claude/agents/test-engineer.md`). Follow directly when no persona is needed.

## The TDD Cycle

```
    RED                GREEN              REFACTOR
 Write a test    Write minimal code    Clean up the
 that fails  ──→  to make it pass  ──→  implementation  ──→  (repeat)
      │                  │                    │
      ▼                  ▼                    ▼
   Test FAILS        Test PASSES         Tests still PASS
```

### Step 1: RED — Write a Failing Test

Write the test first. It must fail. A test that passes immediately proves nothing.

```typescript
// RED — createTask does not exist yet, so this fails
describe('TaskService', () => {
  it('creates a task with title and default status', async () => {
    const task = await taskService.createTask({ title: 'Buy groceries' });

    expect(task.id).toBeDefined();
    expect(task.title).toBe('Buy groceries');
    expect(task.status).toBe('pending');
    expect(task.createdAt).toBeInstanceOf(Date);
  });
});
```

### Step 2: GREEN — Make It Pass

Write the minimum code to make the test pass. Do not over-engineer.

```typescript
// GREEN — minimal implementation
export async function createTask(input: { title: string }): Promise<Task> {
  const task = {
    id: generateId(),
    title: input.title,
    status: 'pending' as const,
    createdAt: new Date(),
  };
  await db.tasks.insert(task);
  return task;
}
```

### Step 3: REFACTOR — Clean Up

With tests green, improve the code without changing behavior:

- Extract shared logic
- Improve naming
- Remove duplication
- Optimize if necessary

Run tests after every refactor step to confirm nothing broke.

## The Prove-It Pattern (Bug Fixes)

When a bug is reported, do not start by trying to fix it. Start by writing a test that reproduces it.

```
Bug report arrives
       │
       ▼
  Write a test that demonstrates the bug
       │
       ▼
  Test FAILS (confirming the bug exists)
       │
       ▼
  Implement the fix
       │
       ▼
  Test PASSES (proving the fix works)
       │
       ▼
  Run full test suite (no regressions)
```

Example:

```typescript
// Bug: "Completing a task doesn't update the completedAt timestamp"

// Step 1: Write the reproduction test — it should FAIL
it('sets completedAt when task is completed', async () => {
  const task = await taskService.createTask({ title: 'Test' });
  const completed = await taskService.completeTask(task.id);

  expect(completed.status).toBe('completed');
  expect(completed.completedAt).toBeInstanceOf(Date);  // fails → bug confirmed
});

// Step 2: Fix the bug
export async function completeTask(id: string): Promise<Task> {
  return db.tasks.update(id, {
    status: 'completed',
    completedAt: new Date(),  // was missing
  });
}

// Step 3: Test passes → bug fixed, regression guarded
```

## The Test Pyramid and Writing Good Tests

For the test pyramid, test-size (resource) model, state-vs-interactions, DAMP over DRY, mocks preference, Arrange-Act-Assert, one-assertion-per-concept, and descriptive naming, see `references/writing-good-tests.md`.

## Anti-Patterns, Rationalizations, and Red Flags

For anti-patterns to avoid, common rationalizations, and red flags, see `references/anti-patterns.md`.

## Browser Runtime Verification

For browser-based changes, combine TDD with runtime verification — see the `browser-testing-with-devtools` skill.

## Subagent Delegation

For when and how to spawn a subagent to write reproduction tests, see `references/subagent-delegation.md`.

## Verification

After completing any implementation:

- [ ] Every new behavior has a corresponding test
- [ ] All tests pass: `npm test`
- [ ] Bug fixes include a reproduction test that failed before the fix
- [ ] Test names describe the behavior being verified
- [ ] No tests were skipped or disabled
- [ ] Coverage hasn't decreased (if tracked)
