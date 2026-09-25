---
name: git-commit
description: Stage and commit git changes with a single-line conventional-commit message inferred from the diff. Use when the user says "commit", "commit this/these changes", or similar.
---

# Git Commit

1. `git status --porcelain` - if empty, say there's nothing to commit and stop.
2. `git add -A`
3. `git diff --staged` - infer type (feat/fix/refactor/chore/docs/test/style/perf/build/ci) and write one line: `type: short imperative description`. No body, no period, no Claude/AI mention, no "Generated with"/"Co-Authored-By" trailers.
4. `git commit -m "<message>"` - single `-m`, no HEREDOC.
5. Report the message and short hash (`git log -1 --oneline`).

If commit fails, show the actual git error instead of retrying blindly.
