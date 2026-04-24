---
description: First-run setup for claude-dev-team — creates user-level skill symlinks so the personified agents (Hubert / Watson / Barb / Pepper / Negev) can read their paired skills via stable `~/.claude/skills/` paths. Idempotent.
---

# claude-dev-team:init

Set up the user-level skill symlinks that bridge between this plugin's namespaced skill install location and the agent files' hardcoded `Read ~/.claude/skills/<skill>/SKILL.md` paths.

## Why this exists

The five personified agents (Hubert, Watson, Barb, Pepper, Negev) read their paired skill at the start of every operation via `Read ~/.claude/skills/<skill-name>/SKILL.md`. The plugin install puts skills at namespaced paths under `~/.claude/plugins/marketplaces/<marketplace>/skills/...`. Without a user-level symlink at the expected path, each agent falls back to memory-based discipline and notes the missing skill in its output. This command creates the bridge so each agent reads from the live, plugin-installed source.

## When to run

- **Once after installing the plugin** (`/plugin install claude-dev-team@jason-claude-dev-team`)
- **After updating the plugin** (`/plugin marketplace update jason-claude-dev-team` followed by reinstall) — re-running ensures symlinks point at the newest installed version
- **Any time the symlinks are missing or stale**

Idempotent — safe to re-run. Each invocation overwrites any existing symlink at the destination.

## What it does

Creates these six symlinks (one per skill bundled in the plugin):

| Symlink at | Points to |
|---|---|
| `~/.claude/skills/git-workflow-and-versioning` | `<plugin-root>/skills/git-workflow-and-versioning` |
| `~/.claude/skills/code-review-and-quality` | `<plugin-root>/skills/code-review-and-quality` |
| `~/.claude/skills/security-and-hardening` | `<plugin-root>/skills/security-and-hardening` |
| `~/.claude/skills/test-driven-development` | `<plugin-root>/skills/test-driven-development` |
| `~/.claude/skills/acceptance-exploration` | `<plugin-root>/skills/acceptance-exploration` |
| `~/.claude/skills/browser-testing-with-devtools` | `<plugin-root>/skills/browser-testing-with-devtools` |

`<plugin-root>` is discovered dynamically by scanning `~/.claude/plugins/marketplaces/*/.claude-plugin/plugin.json` for the entry whose `name` is `claude-dev-team`. Works regardless of which marketplace name the user installed under.

## Implementation

Run the following Bash sequence. It discovers the plugin install path, then creates the six symlinks.

```bash
set -e

# Discover plugin install root by scanning marketplace dirs for plugin.json with name=claude-dev-team
PLUGIN_ROOT=""
for d in ~/.claude/plugins/marketplaces/*/; do
  manifest="$d/.claude-plugin/plugin.json"
  if [ -f "$manifest" ]; then
    name=$(python3 -c "import json,sys; print(json.load(open(sys.argv[1])).get('name',''))" "$manifest" 2>/dev/null || \
           grep -E '"name"\s*:\s*"' "$manifest" | head -1 | sed -E 's/.*"name"\s*:\s*"([^"]+)".*/\1/')
    if [ "$name" = "claude-dev-team" ]; then
      PLUGIN_ROOT="${d%/}"
      break
    fi
  fi
done

if [ -z "$PLUGIN_ROOT" ]; then
  echo "✗ claude-dev-team plugin not found in ~/.claude/plugins/marketplaces/." >&2
  echo "  Run: /plugin marketplace add jasonjgarcia24/claude-dev-team" >&2
  echo "       /plugin install claude-dev-team@jason-claude-dev-team" >&2
  echo "       /reload-plugins" >&2
  echo "  Then re-run /claude-dev-team:init." >&2
  exit 1
fi

echo "Plugin root: $PLUGIN_ROOT"
echo ""
mkdir -p ~/.claude/skills

created=0
skipped=0
for s in git-workflow-and-versioning code-review-and-quality security-and-hardening \
         test-driven-development acceptance-exploration browser-testing-with-devtools; do
  src="$PLUGIN_ROOT/skills/$s"
  dst="$HOME/.claude/skills/$s"
  if [ ! -d "$src" ]; then
    echo "  ⚠ $s — source missing at $src (skipping)"
    skipped=$((skipped+1))
    continue
  fi
  ln -sfn "$src" "$dst"
  echo "  ✓ $s"
  created=$((created+1))
done

echo ""
echo "claude-dev-team:init complete — $created symlinks created, $skipped skipped."
echo ""
echo "The personified agents (Hubert, Watson, Barb, Pepper, Negev) can now read their paired skills."
echo "Test with: ask Hubert to commit the current diff, or ask Watson to review the changes on this branch."
```

Report the command output as-is. If `created < 6`, surface the warning lines verbatim — they indicate the plugin install is incomplete.

## Cleanup (optional)

To remove the symlinks created by this command (e.g., before uninstalling the plugin):

```bash
for s in git-workflow-and-versioning code-review-and-quality security-and-hardening \
         test-driven-development acceptance-exploration browser-testing-with-devtools; do
  rm -f "$HOME/.claude/skills/$s"
done
```

## What this command does NOT do

- Does NOT create user-level symlinks for agents or commands. Personas are invoked via the namespaced form (`claude-dev-team:hubert`, `/claude-dev-team:build`, etc.). If you want short-name invocation (`hubert`, `/build`), create those symlinks manually following the manual install path in the README.
- Does NOT modify `~/.claude/settings.json` or any other Claude Code config.
- Does NOT install the plugin — that's `/plugin install`'s job.
