---
description: Setup or cleanup for claude-dev-team. Default creates user-level skill symlinks so the personified agents (Hubert / Watson / Barb / Pepper / Negev) can read their paired skills. Use `--remove` to undo: removes those symlinks and prints `/plugin uninstall` guidance for full removal. Idempotent.
---

# claude-dev-team:init [--remove]

Two modes, dispatched by flag.

| Flag | Mode | What it does |
|------|------|--------------|
| (none) | **Setup** | Discover plugin install path, create 6 skill symlinks at `~/.claude/skills/<skill>/`. |
| `--remove` | **Cleanup** | Remove the 6 skill symlinks created by setup mode. Print `/plugin uninstall` and `/plugin marketplace remove` commands for the user to run if they want to fully uninstall. |

Both modes are idempotent — safe to re-run.

## Step 0 — Parse flag and dispatch

Inspect `$ARGUMENTS` (or the user's invocation text):

| Flag | Section to follow |
|------|-------------------|
| (none) | **Setup mode** below — skip the Cleanup mode section. |
| `--remove` | **Cleanup mode** below — skip the Setup mode section. |

If both flags appear, prefer `--remove` (cleanup wins).

---

# Setup mode (default)

Create the user-level skill symlinks that bridge between the plugin's namespaced skill install location and the agent files' hardcoded `Read ~/.claude/skills/<skill>/SKILL.md` paths.

## Why this exists

The five personified agents (Hubert, Watson, Barb, Pepper, Negev) read their paired skill at the start of every operation via `Read ~/.claude/skills/<skill-name>/SKILL.md`. The plugin install puts skills at namespaced paths under `~/.claude/plugins/marketplaces/<marketplace>/skills/...`. Without a user-level symlink at the expected path, each agent falls back to memory-based discipline and notes the missing skill in its output.

## When to run

- **Once after installing the plugin** (`/plugin install claude-dev-team@jason-claude-dev-team`)
- **After updating the plugin** (re-run to point symlinks at the newest installed version)
- **Any time the symlinks are missing or stale**

## What it creates

| Symlink at | Points to |
|---|---|
| `~/.claude/skills/git-workflow-and-versioning` | `<plugin-root>/skills/git-workflow-and-versioning` |
| `~/.claude/skills/code-review-and-quality` | `<plugin-root>/skills/code-review-and-quality` |
| `~/.claude/skills/security-and-hardening` | `<plugin-root>/skills/security-and-hardening` |
| `~/.claude/skills/test-driven-development` | `<plugin-root>/skills/test-driven-development` |
| `~/.claude/skills/acceptance-exploration` | `<plugin-root>/skills/acceptance-exploration` |
| `~/.claude/skills/browser-testing-with-devtools` | `<plugin-root>/skills/browser-testing-with-devtools` |

`<plugin-root>` is discovered dynamically by scanning `~/.claude/plugins/marketplaces/*/.claude-plugin/plugin.json` for the entry whose `name` is `claude-dev-team`. Works regardless of which marketplace the user installed under.

## Setup implementation

Run the following Bash sequence:

```bash
set -e

# Discover plugin install root
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
echo "Test: ask Hubert to commit the current diff, or ask Watson to review the changes on this branch."
```

Report the command output as-is. If `created < 6`, surface the warning lines verbatim — they indicate the plugin install is incomplete.

---

# Cleanup mode (`--remove`)

Remove the 6 skill symlinks that setup mode creates, and print guidance for fully uninstalling the plugin.

## What it does

1. Removes the 6 skill symlinks at `~/.claude/skills/<skill>/` — but **only if** they point at a `claude-dev-team` location. Symlinks pointing elsewhere (e.g., user-managed or from another bundle) are left alone.
2. Prints `/plugin uninstall` and `/plugin marketplace remove` commands for the user to run if they want to fully remove the plugin.

This command does NOT actually invoke `/plugin uninstall` itself — that's a Claude Code slash command, not a Bash command, and it must be invoked by the user. Cleanup mode only handles the file-system symlink removal; full plugin teardown requires the two slash commands at the end.

## Cleanup implementation

Run the following Bash sequence:

```bash
removed=0
foreign=0
not_symlink=0
absent=0

for s in git-workflow-and-versioning code-review-and-quality security-and-hardening \
         test-driven-development acceptance-exploration browser-testing-with-devtools; do
  dst="$HOME/.claude/skills/$s"
  if [ -L "$dst" ]; then
    target=$(readlink "$dst")
    if echo "$target" | grep -q "claude-dev-team"; then
      rm "$dst"
      echo "  ✓ removed: $s"
      removed=$((removed+1))
    else
      echo "  ⚠ kept: $s — symlink points elsewhere ($target); not touching"
      foreign=$((foreign+1))
    fi
  elif [ -e "$dst" ]; then
    echo "  ⚠ kept: $s — exists but is not a symlink (real dir/file); not touching"
    not_symlink=$((not_symlink+1))
  else
    absent=$((absent+1))
  fi
done

echo ""
echo "claude-dev-team:init --remove complete — $removed removed, $absent already absent, $foreign kept (foreign), $not_symlink kept (not a symlink)."
echo ""
echo "The personified agents will now fall back to memory-mode for skill discipline."
echo ""
echo "To also uninstall the plugin itself, run these in your Claude Code session:"
echo ""
echo "  /plugin uninstall claude-dev-team@jason-claude-dev-team"
echo "  /plugin marketplace remove jason-claude-dev-team"
echo ""
echo "(Skipping those keeps the plugin installed — symlinks can be recreated with /claude-dev-team:init)"
```

Report the command output verbatim.

## Cleanup safety rules

- Never remove a symlink that points outside `claude-dev-team` paths — the user might be using the same skill name from another bundle.
- Never remove a real directory at `~/.claude/skills/<skill>/` — it's user content, not something this command created.
- Never modify `~/.claude/settings.json` or any other Claude Code config.
- Never auto-invoke `/plugin uninstall` — print the command and let the user decide.

## What this command does NOT do (either mode)

- Does NOT create or remove user-level symlinks for agents or commands. Personas are invoked via the namespaced form (`claude-dev-team:hubert`, `/claude-dev-team:build`, etc.). For short-name access, follow the manual install instructions in the README.
- Does NOT modify `~/.claude/settings.json` or any other Claude Code config.
- Does NOT install or uninstall the plugin — those are handled by `/plugin install` and `/plugin uninstall`. This command only manages the skill-symlink shim.
