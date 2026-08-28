---
name: cleanup
description: Safely audit and remove the local Git branch and linked worktree for a completed task. Use when the user asks to clean up, tear down, or remove local repository state after work finishes or a pull request merges.
---

# Cleanup

Remove only the completed task identified by the current conversation and repository context. Preserve unrelated repository state.

## Scope the target

1. Derive the exact repository, worktree path, branch, and task from the current directory and conversation. Ask the user when ownership is ambiguous.
2. Read applicable repository instructions before inspecting or changing state.
3. List registered worktrees only to find the exact target, identify a surviving checkout, and detect branch conflicts. Do not inspect unrelated worktree contents.
4. Expand inspection only when the initial checks reveal ambiguous ownership, unique work, stale metadata, or conflicting copies.
5. Never inventory or modify shared stashes, unrelated caches, remote branches, or unrelated worktrees.

## Require approval before mutation

1. Begin with read-only inspection, even when the user has already asked to clean up.
2. Present the evidence and classify the exact target as `remove`, `preserve`, or `needs user decision`.
3. Get explicit approval for the named operations before deleting or untracking a branch, removing a worktree, pruning metadata, or clearing files.
4. Treat approval as scoped. Reconfirm if the target or consequences change.

## Audit the exact task

Resolve and inspect:

1. The repository root, common Git directory, exact worktree path, current branch, upstream, HEAD, default branch, and registered worktrees.
2. Tracked, staged, conflicted, and untracked files in the target worktree.
3. Commits unique relative to the upstream and default branch. Do not assume a clean status means all work is recoverable.
4. Pull-request state and merge evidence when the repository uses pull requests.
5. Graphite parent, children, tracking state, and pull-request state when Graphite is installed and the branch is tracked. Otherwise, rely on Git and pull-request evidence.
6. Worktree disk usage only when the user asks about recovered space or removal size affects the decision.

Do not run mutation-capable convenience commands such as `gt sync`, `git fetch --prune`, or `git worktree prune` during the audit. A normal `git fetch` is also unnecessary unless the user asks for refreshed remote evidence or existing local evidence is insufficient; fetching changes repository state and requires approval.

## Classify and propose

Use these outcomes:

1. `remove`: the target worktree is clean, completion or merge is conclusive, no local-only files or commits would be lost, and the branch has no active dependents in a Graphite stack.
2. `preserve`: the worktree is dirty, work is still open, the branch has active dependents, or local-only files or commits exist.
3. `needs user decision`: ownership, recoverability, completion, or conflicting copies remain unclear after scoped inspection.

For `remove`, show a short evidence summary and the exact commands or operations proposed. State whether the local worktree, local branch, and Graphite metadata will be removed. State that remote branches and pull requests will remain unchanged unless the user separately requests their removal. Ask for one explicit approval covering only those operations.

For `preserve` or `needs user decision`, stop before mutation and explain what prevents safe removal.

## Execute approved cleanup

Immediately before mutation, recheck the target path, branch, HEAD, and worktree status. Stop if any relevant state changed.

For an approved, clean linked worktree:

1. Identify a surviving checkout of the same repository before removing the target. Never remove the only usable checkout or assume a directory is a disposable linked worktree.
2. From the surviving checkout, run `git worktree remove <exact-absolute-worktree-path>`. Do not use `--force` unless newly discovered changes have been audited and the user separately approves discarding them.
3. If the repository uses Graphite and the branch is tracked, use Graphite's local branch deletion command to remove the exact merged branch and its Graphite metadata. Otherwise, run `git branch -d <exact-branch>` from the surviving checkout.
4. Do not close pull requests, delete remote branches, remove stashes, or clear unrelated caches unless the user separately requests those actions.

If the target is the repository's primary checkout rather than a removable linked worktree, do not delete its directory. Propose only the safe branch operation appropriate to that checkout and get approval for it.

Never use broad `rm`, unresolved variables, globs, repository roots, home directories, `git reset --hard`, or forced branch deletion as shortcuts.

## Verify and report

After cleanup, verify from the surviving checkout that:

1. A removed worktree path no longer exists and is absent from `git worktree list --porcelain`.
2. The exact local branch and, when applicable, Graphite entry are gone.
3. The pull request, remote branch, merged commit, or default branch still provides the stated recovery path.

Report what was removed, what remote or shared state was preserved, and how the code can be recovered. Report approximate disk recovery only when measured.
