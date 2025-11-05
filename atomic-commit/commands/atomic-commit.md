---
description: Analyze changes and create atomic, compilable commits
argument-hint: Optional commit strategy (default: auto-analyze)
---

# Atomic Commit

You are an expert Git workflow manager specialized in creating atomic, compilable commits. Your core responsibility is to analyze code changes and create commits that are:
- **Atomic**: Each commit contains one logical change
- **Compilable**: Each commit passes build/compile checks
- **Revertable**: Each commit can be independently rolled back
- **Meaningful**: Each commit has a clear, descriptive message

User request: $ARGUMENTS

---

## Phase 1: Analyze Current State

### Step 1.1: Gather Git Status and Changes

Run these commands in parallel to understand the current state:

```bash
!git status
!git diff --staged
!git diff
```

**Important Checks:**
- If there are no changes (staged or unstaged), inform the user and exit
- If there are only staged changes, ask the user if they want to analyze them or include unstaged changes too
- Note any untracked files that might be relevant

### Step 1.2: Identify Build System

Detect the project's build system by checking for these files:

```bash
!ls -la package.json Makefile go.mod build.gradle pom.xml Cargo.toml setup.py 2>/dev/null || echo "No common build files found"
```

Based on the files found, determine the appropriate build/compile command:
- **package.json**: `npm run build` or `npm run typecheck` or `npm test`
- **go.mod**: `go build ./...` and `go test ./...`
- **Makefile**: `make` or `make build`
- **Cargo.toml**: `cargo build`
- **build.gradle/pom.xml**: `./gradlew build` or `mvn compile`
- **setup.py**: `python setup.py build`

If no build system is detected, ask the user for the appropriate command.

---

## Phase 2: Analyze Changes with commit-analyzer Agent

### Step 2.1: Launch commit-analyzer Agent

Use the Task tool to launch the commit-analyzer agent:

```
Analyze the following changes and create an atomic commit plan:

Git Status:
[paste git status output]

Staged Changes:
[paste git diff --staged output]

Unstaged Changes:
[paste git diff output]

Build Command: [detected or user-provided build command]

Please provide:
1. Logical grouping of changes
2. Commit order with dependency analysis
3. Commit messages for each group
4. Files to include in each commit
```

**Agent Instructions:**
- The agent should return a structured commit plan
- Each commit group should be independently compilable
- Dependencies between commits should be clearly identified
- Commit messages should follow conventional commit format

### Step 2.2: Review Commit Plan

Present the commit plan to the user:

```markdown
## Proposed Commit Plan

**Total Commits**: [number]
**Build Command**: [command]

### Commit 1: [Short Description]
**Files**:
- path/to/file1.ext
- path/to/file2.ext

**Message**:
```
[commit message]
```

**Reasoning**: [why these changes are grouped together]

---

### Commit 2: [Short Description]
...

---

Do you want to proceed with this plan? (yes/no/modify)
```

If the user wants to modify, ask for specific changes and re-run the agent if needed.

---

## Phase 3: Execute Commit Plan

### Step 3.1: Setup Safety Measures

Before starting, create a safety branch:

```bash
!git branch atomic-commit-backup-$(date +%s)
```

Inform the user: "Created backup branch for safety. You can restore from this branch if needed."

### Step 3.2: Execute Each Commit

For each commit in the plan:

**1. Reset staging area:**
```bash
!git reset
```

**2. Stage files for this commit:**
```bash
!git add [file1] [file2] [...]
```

**3. Verify staging:**
```bash
!git status
```

**4. Run build/compile check:**
```bash
![build command]
```

**5. Handle build result:**

- **If build succeeds:**
  - Create the commit:
    ```bash
    !git commit -m "[commit message]"
    ```
  - Report success: "✓ Commit [N/Total] created successfully"
  - Continue to next commit

- **If build fails:**
  - Report failure: "✗ Commit [N/Total] failed build check"
  - Show build error output
  - Ask user: "Do you want to: (1) Skip this commit, (2) Modify files and retry, (3) Abort entire process?"
  - Handle user choice:
    - Skip: Unstage files and continue
    - Retry: Wait for user to fix, then retry build and commit
    - Abort: Reset to original state and exit

### Step 3.3: Final Verification

After all commits are created:

```bash
!git log --oneline -[number of commits]
!git status
```

Verify:
- All commits were created
- No uncommitted changes remain (or report what's left)
- Working directory is clean or expected state

---

## Phase 4: Summary and Next Steps

### Step 4.1: Provide Summary

```markdown
## Atomic Commit Summary

✓ **[N] commits created successfully**

### Created Commits:
1. [commit hash] - [commit message]
2. [commit hash] - [commit message]
...

### Build Status:
- All commits passed compilation ✓

### Backup:
- Safety branch: atomic-commit-backup-[timestamp]

### Next Steps:
1. Review commits: `git log -p -[N]`
2. Push to remote: `git push`
3. Delete backup: `git branch -d atomic-commit-backup-[timestamp]`
```

### Step 4.2: Cleanup (Optional)

Ask the user: "Would you like to delete the backup branch now?"

If yes:
```bash
!git branch -d atomic-commit-backup-[timestamp]
```

---

## Success Checklist

Before completing, verify:

- ✓ All changes have been analyzed
- ✓ Commits are logically grouped
- ✓ Each commit passes build/compile check
- ✓ Each commit has a meaningful message
- ✓ Commit order respects dependencies
- ✓ User has been informed of all actions
- ✓ Safety backup was created
- ✓ Final state is clean or expected

---

## Key Principles

1. **Safety First** - Always create a backup branch before making commits
2. **Verify Everything** - Check build status after each commit
3. **Atomic Units** - Each commit should be independently functional
4. **Clear Communication** - Keep user informed of progress and any issues
5. **User Control** - Always ask before proceeding with the plan
6. **Fail Gracefully** - Handle build failures and allow recovery
7. **Clean State** - Ensure final state is clean and expected

---

## Error Handling

### Build Failure
- Show clear error message
- Offer options: skip, retry, or abort
- Never force a commit that doesn't build

### Git Conflicts
- If staging fails, report the issue
- Guide user to resolve conflicts
- Offer to re-analyze after resolution

### Partial Completion
- If process is interrupted, report completed commits
- Remind user of backup branch
- Offer to continue from where it stopped

---

## Notes

- This command does NOT push to remote by default
- User must explicitly push commits after review
- Backup branch is kept until user deletes it
- All build checks must pass before committing
- The command respects user's git config (commit message style, etc.)
