---
description: Setup or cleanup for claude-dev-team. Default creates user-level symlinks so the personified agents (Hubert / Watson / Barb / Pepper / Negev) and their commands are accessible by short name in addition to the plugin-namespaced form. Use `--remove` to undo: removes those symlinks and prints `/plugin uninstall` guidance for full removal. Idempotent.
---

# claude-dev-team:init [--remove]

Two modes, dispatched by flag.

| Flag | Mode | What it does |
|------|------|--------------|
| (none) | **Setup** | Discover plugin install path, create user-level symlinks for 5 agents, 6 skills, and 2 commands. |
| `--remove` | **Cleanup** | Remove the 13 symlinks created by setup mode. Print `/plugin uninstall` and `/plugin marketplace remove` commands for the user to run if they want to fully uninstall. |

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

Create the user-level symlinks that expose claude-dev-team's agents, skills, and commands at short, un-namespaced paths in `~/.claude/`. Two purposes:

1. **Bridge for the agents' skill Reads.** The personified agents Read their paired skill at the start of every operation via `Read ~/.claude/skills/<skill-name>/SKILL.md`. The plugin install puts skills at namespaced paths under `~/.claude/plugins/marketplaces/<marketplace>/skills/...` — without a user-level symlink at the expected path, each agent falls back to memory-based discipline.
2. **Short-name invocation.** With agent symlinks at `~/.claude/agents/<persona>.md`, the personas are addressable as `hubert` (short) in addition to `claude-dev-team:hubert` (namespaced). Same for the `/build` and `/test` commands.

## When to run

- **Once after installing the plugin** (`/plugin install claude-dev-team@jason-claude-dev-team`)
- **After updating the plugin** (re-run to point symlinks at the newest installed version)
- **Any time the symlinks are missing or stale**

## What it creates

13 symlinks total — 5 agents + 6 skills + 2 commands, all pointing into `<plugin-root>`:

| Category | Symlink at | Points to |
|---|---|---|
| Agent | `~/.claude/agents/git-workflow.md` (Hubert) | `<plugin-root>/agents/git-workflow.md` |
| Agent | `~/.claude/agents/code-reviewer.md` (Watson) | `<plugin-root>/agents/code-reviewer.md` |
| Agent | `~/.claude/agents/security-auditor.md` (Barb) | `<plugin-root>/agents/security-auditor.md` |
| Agent | `~/.claude/agents/test-engineer.md` (Pepper) | `<plugin-root>/agents/test-engineer.md` |
| Agent | `~/.claude/agents/acceptance-explorer.md` (Negev) | `<plugin-root>/agents/acceptance-explorer.md` |
| Skill | `~/.claude/skills/git-workflow-and-versioning` | `<plugin-root>/skills/git-workflow-and-versioning` |
| Skill | `~/.claude/skills/code-review-and-quality` | `<plugin-root>/skills/code-review-and-quality` |
| Skill | `~/.claude/skills/security-and-hardening` | `<plugin-root>/skills/security-and-hardening` |
| Skill | `~/.claude/skills/test-driven-development` | `<plugin-root>/skills/test-driven-development` |
| Skill | `~/.claude/skills/acceptance-exploration` | `<plugin-root>/skills/acceptance-exploration` |
| Skill | `~/.claude/skills/browser-testing-with-devtools` | `<plugin-root>/skills/browser-testing-with-devtools` |
| Command | `~/.claude/commands/build.md` | `<plugin-root>/commands/build.md` |
| Command | `~/.claude/commands/test.md` | `<plugin-root>/commands/test.md` |

`<plugin-root>` is discovered dynamically by scanning `~/.claude/plugins/marketplaces/*/.claude-plugin/plugin.json` for the entry whose `name` is `claude-dev-team`. Works regardless of which marketplace the user installed under.

> Note: setup does NOT create a user-level symlink for `/init` — that would collide with the Claude Code built-in `/init` command (which initializes CLAUDE.md). The init command stays namespaced as `/claude-dev-team:init`.

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
mkdir -p ~/.claude/agents ~/.claude/skills ~/.claude/commands

created=0
skipped=0

echo "Agents:"
for a in git-workflow code-reviewer security-auditor test-engineer acceptance-explorer; do
  src="$PLUGIN_ROOT/agents/$a.md"
  dst="$HOME/.claude/agents/$a.md"
  if [ ! -f "$src" ]; then
    echo "  ⚠ $a.md — source missing at $src (skipping)"
    skipped=$((skipped+1))
    continue
  fi
  ln -sfn "$src" "$dst"
  echo "  ✓ $a.md"
  created=$((created+1))
done

echo ""
echo "Skills:"
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
echo "Commands:"
for c in build test; do
  src="$PLUGIN_ROOT/commands/$c.md"
  dst="$HOME/.claude/commands/$c.md"
  if [ ! -f "$src" ]; then
    echo "  ⚠ $c.md — source missing at $src (skipping)"
    skipped=$((skipped+1))
    continue
  fi
  ln -sfn "$src" "$dst"
  echo "  ✓ $c.md"
  created=$((created+1))
done

echo ""
echo "claude-dev-team:init complete — $created symlinks created, $skipped skipped."
echo ""
echo "Personas accessible as: hubert, watson, barb, pepper, negev (short) OR claude-dev-team:hubert etc. (namespaced)."
echo "Commands accessible as: /build, /test (short) OR /claude-dev-team:build etc. (namespaced)."
echo "Skill Reads at ~/.claude/skills/<name>/SKILL.md now resolve to the plugin install."
echo ""
echo "Test: ask Hubert to commit the current diff, or ask Watson to review the changes on this branch."
```

Report the command output as-is. If `created < 6`, surface the warning lines verbatim — they indicate the plugin install is incomplete.

---

# Cleanup mode (`--remove`)

Remove the 6 skill symlinks that setup mode creates, and print guidance for fully uninstalling the plugin.

## What it does

1. Removes the 13 symlinks at `~/.claude/agents/`, `~/.claude/skills/`, and `~/.claude/commands/` — but **only if** they point at a `claude-dev-team` location. Symlinks pointing elsewhere (e.g., user-managed or from another bundle with the same name) are left alone.
2. Prints `/plugin uninstall` and `/plugin marketplace remove` commands for the user to run if they want to fully remove the plugin.

This command does NOT actually invoke `/plugin uninstall` itself — that's a Claude Code slash command, not a Bash command, and it must be invoked by the user. Cleanup mode only handles the file-system symlink removal; full plugin teardown requires the two slash commands at the end.

## Cleanup implementation

Run the following Bash sequence:

```bash
removed=0
foreign=0
not_symlink=0
absent=0

remove_symlink() {
  local dst="$1"
  local label="$2"
  if [ -L "$dst" ]; then
    local target=$(readlink "$dst")
    if echo "$target" | grep -q "claude-dev-team"; then
      rm "$dst"
      echo "  ✓ removed: $label"
      removed=$((removed+1))
    else
      echo "  ⚠ kept: $label — symlink points elsewhere ($target); not touching"
      foreign=$((foreign+1))
    fi
  elif [ -e "$dst" ]; then
    echo "  ⚠ kept: $label — exists but is not a symlink (real dir/file); not touching"
    not_symlink=$((not_symlink+1))
  else
    absent=$((absent+1))
  fi
}

echo "Agents:"
for a in git-workflow code-reviewer security-auditor test-engineer acceptance-explorer; do
  remove_symlink "$HOME/.claude/agents/$a.md" "agents/$a.md"
done

echo ""
echo "Skills:"
for s in git-workflow-and-versioning code-review-and-quality security-and-hardening \
         test-driven-development acceptance-exploration browser-testing-with-devtools; do
  remove_symlink "$HOME/.claude/skills/$s" "skills/$s"
done

echo ""
echo "Commands:"
for c in build test; do
  remove_symlink "$HOME/.claude/commands/$c.md" "commands/$c.md"
done

echo ""
echo "claude-dev-team:init --remove complete — $removed removed, $absent already absent, $foreign kept (foreign), $not_symlink kept (not a symlink)."
echo ""
echo "The personified agents stay available via the namespaced form (claude-dev-team:hubert etc.)"
echo "but will fall back to memory-mode for skill discipline (no ~/.claude/skills/<name>/ paths)."
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

- Does NOT create or remove a user-level symlink for `init` itself. The init command stays namespaced as `/claude-dev-team:init` to avoid colliding with Claude Code's built-in `/init`.
- Does NOT modify `~/.claude/settings.json` or any other Claude Code config.
- Does NOT install or uninstall the plugin — those are handled by `/plugin install` and `/plugin uninstall`. This command only manages the user-level symlinks.
