# Surface: agent or skill

Probing tool: Agent tool (for subagents) or Skill tool (for skills).

## How to probe

Invoke the target in representative scenarios drawn from its `description` field — one scenario per documented trigger or use case.

**For agents:**
- Construct a caller prompt that matches a real invocation pattern
- Invoke via the Agent tool with the target's `subagent_type`
- Capture the full response and any result line the agent specifies

**For skills:**
- Construct a caller prompt that matches a real trigger from the skill's description
- Invoke via the Skill tool with the skill name
- Capture the response and any output the skill produces

## Evidence to capture

- Full agent/skill response per scenario — save to evidence dir
- Result line verbatim (for agents with a result-line contract) — every variant (✓ / ✗ / ⚠)
- Side effects: files written, commits made, external API calls, state changes
- Tool-use trace if available — confirms the agent stayed within its declared tool scope

## Stage-specific additions

### MVP
- Happy-path invocation produces a coherent response matching the description
- Result line (if specified) is present and parseable

### Beta
- Refusal paths trigger where the spec says they should (e.g., destructive ops, out-of-scope requests)
- Ambiguous inputs surface clarifying questions instead of silent guesses
- Tool scope honored — the agent does not use tools outside its declared `tools:` frontmatter

### GA
- Result line format matches the documented contract across all verdict states (✓ / ✗ / ⚠)
- Persona discipline holds — introduction, tone, scope limits all consistent
- Failure modes surface cleanly — no silent drops, no malformed result lines, no leaked internal reasoning
- Cross-references to sibling personas/skills are accurate (e.g., "see Watson for code review" — confirm Watson exists and still matches)
- The agent/skill degrades cleanly when a required input is missing (asks for it) or when an upstream dependency is absent (reports deferred)
