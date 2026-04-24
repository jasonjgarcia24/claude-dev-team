# Debugging with git

Use git history to find when bugs were introduced and who changed what.

```bash
# Find which commit introduced a bug (binary search)
git bisect start
git bisect bad HEAD
git bisect good <known-good-commit>
# Git checks out midpoints; run your test at each to narrow down
git bisect reset            # when done

# View what changed recently
git log --oneline -20
git diff HEAD~5..HEAD -- src/

# Find who last changed a specific line
git blame src/services/task.ts

# Search commit messages for a keyword
git log --grep="validation" --oneline

# Show all files changed across a range
git log --name-only HEAD~10..HEAD

# Find the commit that added or removed a specific string
git log -S"oldFunctionName" --oneline

# Show the full history of a single file, including renames
git log --follow --oneline -- path/to/file.ts
```
