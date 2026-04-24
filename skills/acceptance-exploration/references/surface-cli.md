# Surface: CLI tool

Probing tool: Bash.

## How to probe

Run the binary with representative argv and stdin. Capture stdout, stderr, exit code, and side effects.

Pattern:

```bash
<cmd> <args> <<< "<stdin>" > /tmp/stdout.log 2> /tmp/stderr.log
echo "exit: $?"
```

For side-effect observation: snapshot files / dirs / state before and after, then diff.

## Evidence to capture

- stdout transcript (redirect to evidence dir)
- stderr transcript
- Exit code
- List of files changed / created / removed (use `find` with `-newer` or a before/after diff)
- Any network calls made (run under `strace -f -e network` or similar if worth capturing)

## Stage-specific additions

### MVP
- Pipe-friendly output — stdout is data, stderr is chatter and errors
- Common flags documented as supported actually work: `--help`, `--version`, `-h`, `-v`

### Beta
- Invalid argv produces a clear error message, not a stack trace or silent exit
- `SIGINT` (Ctrl+C) exits cleanly — no orphaned state, no corrupted files
- Respects `--` to separate flags from positional args (if relevant)
- Handles missing required args with a specific error, not a generic crash

### GA
- Exit codes follow convention: `0` success, non-zero failure, distinct codes for distinct failure modes where documented
- `--help` / `-h` output is accurate (reflects actual supported args)
- Startup time within budget for the use case
- No leaked debug output on stdout in production paths
- Consistent behavior across shells (bash, zsh) and locales (`LC_ALL=C`)
- Handles stdin EOF cleanly when expected
