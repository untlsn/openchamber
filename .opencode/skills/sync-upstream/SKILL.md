---
name: sync-upstream
description: Use when the user asks to sync this fork, fetch or pull from upstream, or rebase local work onto the original OpenChamber repository.
---

# Sync Upstream

Synchronize the current local branch with the original OpenChamber repository by rebasing local commits onto upstream.

## Repository Contract

- Fork remote: `origin`
- Original repository remote: `upstream`
- Original repository URL: `https://github.com/openchamber/openchamber.git`
- Never push as part of this skill unless the user explicitly asks.
- Never discard, reset, skip, or overwrite local work to complete synchronization.

## Workflow

1. Verify this is a Git worktree and inspect the current branch and status.
2. Stop and ask the user if HEAD is detached or another rebase, merge, or cherry-pick is already in progress.
3. Inspect `git remote -v`. If `upstream` is missing, add it with the original repository URL above. If it exists with a different URL, stop and ask before changing it.
4. Run `git fetch upstream`.
5. Use the current branch name as the upstream target when `upstream/<current-branch>` exists. Otherwise use `upstream/main`, and state that fallback before rebasing.
6. Run `git rebase upstream/<target-branch>`. Do not change persistent Git rebase or pull configuration.
7. If Git refuses because of uncommitted changes, report the affected files and ask whether to use `--autostash`; do not stash implicitly.
8. If conflicts occur, inspect the rebased commit and both sides. Preserve the local commit's intent on top of current upstream behavior rather than blindly choosing `ours` or `theirs`.
9. Before editing source conflicts, load every project skill and documentation required by `AGENTS.md`. Resolve only conflicts whose intended result is clear. Ask the user when resolution requires a product or behavior decision.
10. Stage only resolved paths and continue with `GIT_EDITOR=true git rebase --continue`. Repeat until complete. Never use `git rebase --skip` or `git rebase --abort` unless the user explicitly requests it.
11. Verify the final status and run `git rev-list --left-right --count upstream/<target-branch>...HEAD`.

## Completion Report

Report:

- The upstream target used.
- Whether the rebase completed or stopped on a conflict.
- The final behind/ahead counts relative to the target.
- Any conflicts and how they were resolved.
- That no push was performed.

After a successful rebase, mention that updating an already-published fork branch may require a separate explicit `git push --force-with-lease origin <branch>` request. Do not perform that push automatically.
