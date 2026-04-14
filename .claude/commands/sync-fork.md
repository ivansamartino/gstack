# Sync Fork with Upstream

Merge upstream changes into this fork while preserving our modifications.

## Step 0 — Validate state

Check current branch and working tree:

```bash
git status --porcelain
git branch --show-current
```

If the working tree is dirty, **stop and ask the user** whether to stash, commit,
or abort. Do not proceed with uncommitted changes.

## Step 1 — Detect upstream remote

```bash
git remote -v
```

Look for a remote named `upstream`. If it doesn't exist:

1. Detect the parent repo:
   ```bash
   gh api repos/{owner}/{repo} --jq '.parent.full_name'
   ```
2. If a parent is found, add it:
   ```bash
   git remote add upstream https://github.com/<parent_full_name>.git
   ```
3. If no parent is found, ask the user for the upstream URL.

## Step 2 — Detect the default branch

Determine the upstream default branch (usually `main` or `master`):

```bash
git remote show upstream | grep 'HEAD branch'
```

Use whatever branch name is reported. Call it `<upstream-branch>` in the steps below.

Also detect the local default branch:

```bash
git remote show origin | grep 'HEAD branch'
```

Call it `<local-branch>`.

## Step 3 — Fetch upstream

```bash
git fetch upstream
```

## Step 4 — Check for new changes

```bash
git log --oneline <local-branch>..upstream/<upstream-branch> | head -20
```

If there are no new commits, report "Already up to date" and stop.

Show the user a summary of incoming changes (count + notable commits).

## Step 5 — Merge with our changes winning

Switch to the local default branch and merge:

```bash
git checkout <local-branch>
git merge upstream/<upstream-branch> --strategy-option ours --no-edit
```

The `--strategy-option ours` flag means: when there's a conflict, **our version wins**.
Non-conflicting upstream changes (new files, changes to files we haven't touched) merge
in normally.

## Step 6 — Verify and push

```bash
git log --oneline -5
git diff --stat HEAD~1
```

Show the user what changed. Then push:

```bash
git push origin <local-branch>
```

## Step 7 — Summary

Report:
- How many upstream commits were merged
- Whether any conflicts were auto-resolved in our favor
- The new HEAD commit

If any active feature branches exist, suggest rebasing them:
```bash
git branch --no-merged <local-branch>
```
