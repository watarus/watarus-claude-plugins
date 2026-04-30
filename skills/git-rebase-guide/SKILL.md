---
name: git-rebase-guide
description: Guide for git rebase operations including interactive rebase, fixup, squash, and conflict resolution. Use this skill when performing rebase operations, cleaning up commit history, squashing commits, or recovering from rebase conflicts.
---

# Git Rebase Guide

## Overview

Safe and correct git rebase operations including interactive rebase, fixup/squash, and conflict recovery. This skill ensures proper workflow and prevents common mistakes.

## When to Use This Skill

Use this skill when:
- Rebasing a branch onto main/master
- Cleaning up commit history with interactive rebase
- Squashing or fixing up commits
- Resolving rebase conflicts
- Recovering from a failed rebase

## MANDATORY: Backup Before Rebase

**This is NOT optional. ALWAYS create a backup branch before ANY rebase operation.**

```bash
# REQUIRED: Create backup branch BEFORE rebase
git branch backup-$(git branch --show-current)-$(date +%Y%m%d%H%M%S)
```

This allows instant recovery if anything goes wrong:
```bash
# If rebase fails or produces unexpected results:
git checkout <original-branch>
git reset --hard backup-<branch>-<timestamp>
```

## Pre-Rebase Checklist

Before starting rebase:

1. **Create backup branch** (MANDATORY - see above)
2. **Check current state:**
   ```bash
   git status
   git log --oneline -10
   ```
3. **Ensure working directory is clean:**
   ```bash
   git stash  # if there are uncommitted changes
   ```
4. **Count commits** — if many (≈5+), squash first (see next section)
   ```bash
   git log --oneline main..HEAD | wc -l
   ```

## Squash First When There Are Many Commits

**When rebasing onto an updated base branch with many commits, squash first, then rebase.**

Why: rebasing N commits onto an updated main forces you to resolve conflicts up to N times — once per commit. Squashing into a single (or few) logical commits collapses conflict resolution into one round, and the final history stays clean.

**Threshold:** if `git log --oneline main..HEAD` shows roughly 5+ commits — especially WIP/fix-typo style commits — squash first.

**Workflow:**

```bash
# 1. Backup (MANDATORY)
git branch backup-$(git branch --show-current)-$(date +%Y%m%d%H%M%S)

# 2. Squash locally first (do NOT pull main yet — squash on the current base)
git rebase -i $(git merge-base HEAD main)
# In editor: keep first commit as 'pick', change rest to 'fixup' (or 'squash')
# Save and close

# 3. Now update main and rebase the squashed commit
git fetch origin
git rebase origin/main
# Conflicts (if any) resolved once, not N times
```

**When NOT to squash first:** commits are already logically separated and worth preserving in history (e.g., reviewable atomic commits in a long-lived PR).

## Core Operations

### 1. Basic Rebase onto Main

**Update local main first:**
```bash
git fetch origin
git checkout main
git pull origin main
git checkout <feature-branch>
```

**Rebase onto main:**
```bash
git rebase main
```

### 2. Interactive Rebase

**Start interactive rebase for last N commits:**
```bash
git rebase -i HEAD~N
```

**Rebase from a specific commit:**
```bash
git rebase -i <commit-hash>^
```

**Rebase onto main interactively:**
```bash
git rebase -i main
```

### 3. Interactive Rebase Commands

Editor will show commits oldest-to-newest:

```
pick abc1234 First commit message
pick def5678 Second commit message
pick ghi9012 Third commit message
```

**Available commands:**
| Command | Short | Description |
|---------|-------|-------------|
| `pick` | `p` | Use commit as-is |
| `reword` | `r` | Use commit but edit message |
| `edit` | `e` | Stop for amending |
| `squash` | `s` | Meld into previous commit, keep both messages |
| `fixup` | `f` | Meld into previous commit, discard this message |
| `drop` | `d` | Remove commit entirely |

### 4. Squash vs Fixup

**Squash** - Combine commits, edit combined message:
```
pick abc1234 Add user authentication
squash def5678 Fix typo in auth
squash ghi9012 Add tests for auth
```
Result: Opens editor to combine all three commit messages.

**Fixup** - Combine commits, keep only first message:
```
pick abc1234 Add user authentication
fixup def5678 Fix typo in auth
fixup ghi9012 Add tests for auth
```
Result: Single commit with message "Add user authentication".

### 5. Using --fixup and --autosquash

**Create a fixup commit for a previous commit:**
```bash
# Find the commit to fix
git log --oneline -10

# Create fixup commit (automatically names it "fixup! <original message>")
git commit --fixup=<commit-hash>
```

**Create a squash commit:**
```bash
git commit --squash=<commit-hash>
```

**Auto-arrange and apply fixups:**
```bash
git rebase -i --autosquash main
```
This automatically reorders fixup/squash commits next to their targets.

### 6. Reordering Commits

In interactive rebase editor, simply change the order:

Before:
```
pick abc1234 Add feature A
pick def5678 Add feature B
pick ghi9012 Fix feature A bug
```

After (move fix next to its feature):
```
pick abc1234 Add feature A
pick ghi9012 Fix feature A bug
pick def5678 Add feature B
```

## Conflict Resolution

### When Conflicts Occur

**Check status:**
```bash
git status
```

Shows files with conflicts marked as "both modified".

### Resolution Workflow

**Step 1: Identify conflicted files**
```bash
git diff --name-only --diff-filter=U
```

**Step 2: Open and resolve each file**

Conflict markers look like:
```
<<<<<<< HEAD
Current branch content
=======
Incoming branch content
>>>>>>> branch-name
```

Edit to keep desired content, remove markers.

**Step 3: Mark as resolved**
```bash
git add <resolved-file>
```

**Step 4: Continue rebase**
```bash
git rebase --continue
```

**Step 5: Repeat if more conflicts**

### Conflict Resolution Options

**Continue after resolving:**
```bash
git rebase --continue
```

**Skip this commit (use with caution):**
```bash
git rebase --skip
```

**Abort and return to original state:**
```bash
git rebase --abort
```

## Recovery Operations

### Abort Current Rebase

If something goes wrong during rebase:
```bash
git rebase --abort
```
Returns to the exact state before rebase started.

### Recover from Completed Bad Rebase

**Using backup branch:**
```bash
git checkout <feature-branch>
git reset --hard backup-<feature-branch>-<timestamp>
```

**Using reflog (if no backup):**
```bash
# Find the commit before rebase
git reflog

# Output shows:
# abc1234 HEAD@{0}: rebase (finish): ...
# def5678 HEAD@{1}: rebase (start): ...
# ghi9012 HEAD@{2}: commit: Your last commit before rebase  <-- this one

# Reset to that point
git reset --hard HEAD@{2}
```

### Undo a Pushed Rebase (DANGEROUS)

Only if you're the only one using the branch:
```bash
git push --force-with-lease origin <branch>
```

**NEVER force push to shared branches like main/master.**

## Common Scenarios

### Scenario 1: Clean Up Before PR

You have messy commits and want to squash them:

```bash
# Create backup
git branch backup-$(git branch --show-current)-$(date +%Y%m%d%H%M%S)

# Count commits since branching from main
git log --oneline main..HEAD

# Interactive rebase
git rebase -i main

# In editor: change 'pick' to 'squash' or 'fixup' for commits to combine
# Save and close editor
# Edit combined commit message if prompted
```

### Scenario 2: Rebase Feature Branch onto Updated Main

```bash
# Update main
git fetch origin
git checkout main
git pull origin main

# Backup and rebase
git checkout feature-branch
git branch backup-feature-branch-$(date +%Y%m%d%H%M%S)
git rebase main

# If conflicts:
# 1. Resolve conflicts in files
# 2. git add <files>
# 3. git rebase --continue

# Force push (only if branch is yours alone)
git push --force-with-lease origin feature-branch
```

### Scenario 3: Fix a Typo in Old Commit

```bash
# Make the fix
vim file.txt

# Create fixup commit
git add file.txt
git commit --fixup=<commit-hash-to-fix>

# Autosquash
git rebase -i --autosquash main
# Editor opens with fixup already in correct position
# Save and close
```

### Scenario 4: Split a Commit

```bash
git rebase -i HEAD~3

# Change 'pick' to 'edit' for the commit to split
# Save and close

# Reset the commit but keep changes
git reset HEAD^

# Create multiple commits
git add file1.txt
git commit -m "First part"
git add file2.txt
git commit -m "Second part"

# Continue
git rebase --continue
```

### Scenario 5: Rebase Aborted Due to Conflicts - Start Over

```bash
# Abort current mess
git rebase --abort

# Check you're back to original state
git log --oneline -5
git status

# Try again more carefully
git rebase main
```

## Best Practices

1. **Always backup before rebase** - Create a backup branch
2. **Squash before rebasing many commits** - Resolve conflicts once instead of N times
3. **Never rebase shared branches** - Only rebase your own feature branches
4. **Use --force-with-lease** - Safer than --force when pushing
5. **Rebase often** - Smaller rebases are easier than big ones
6. **Clean up before PR** - Squash WIP commits, keep logical commits
7. **Test after rebase** - Run tests to ensure nothing broke

## Quick Reference

| Task | Command |
|------|---------|
| Rebase onto main | `git rebase main` |
| Interactive rebase last 5 | `git rebase -i HEAD~5` |
| Create fixup commit | `git commit --fixup=<hash>` |
| Auto-squash fixups | `git rebase -i --autosquash main` |
| Continue after conflict | `git rebase --continue` |
| Abort rebase | `git rebase --abort` |
| Skip conflicted commit | `git rebase --skip` |
| Check reflog | `git reflog` |
| Force push safely | `git push --force-with-lease` |
