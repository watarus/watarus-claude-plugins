---
description: Automatically handle PR comments and fix CI failures for the current branch
---

# GitHub PR Fix

You are an expert GitHub PR automation specialist. Your responsibility is to orchestrate the complete process of handling PR feedback and CI failures for the current branch's pull request.

---

## Phase 1: PR Detection

### Step 1.1: Verify GitHub CLI

Check that the GitHub CLI is installed and authenticated:

```bash
!gh --version
!gh auth status
```

If not authenticated, inform the user they need to run `gh auth login`.

### Step 1.2: Get Current Branch PR

Get the PR associated with the current branch and repository information:

```bash
!gh pr view --json url,number,title,state,headRepositoryOwner,baseRepository
```

Extract the owner and repo name from the repository information for later use with GitHub API.

If no PR exists, inform the user and exit. Otherwise, display the PR information:
- PR #[number]: [title]
- URL: [url]
- State: [state]
- Repository: [owner]/[repo]

Store these values for use in later phases (especially for posting replies).

---

## Phase 2: Parallel Analysis

**IMPORTANT:** Launch THREE specialized agents in parallel to analyze different aspects of the PR. Use the Task tool to launch ALL THREE agents simultaneously within a single message for maximum efficiency.

### Analysis Tasks

Use the **conflict-resolver** agent to:
- Check if PR has merge conflicts
- Fetch latest base branch
- Attempt rebase if conflicts exist
- Auto-resolve conflicts where possible
- Provide structured JSON output with resolution status

Use the **pr-comment-handler** agent to:
- Fetch all PR comments (both review comments and issue comments)
- Categorize each comment by type and priority
- Draft appropriate replies
- Identify code changes needed
- Provide structured JSON output with action items

Use the **ci-checker** agent to:
- Check CI status for the PR
- Fetch logs for any failed checks
- Diagnose failure causes (tests, builds, lints, etc.)
- Propose specific fixes
- Provide structured JSON output with fix strategies

**Execution:** Deploy all three agents in parallel by calling the Task tool three times in the same message:
1. Task tool with subagent_type="conflict-resolver" and prompt="Check PR number [NUMBER] for merge conflicts. If conflicts exist, fetch the latest base branch, attempt rebase, and resolve conflicts where possible. Provide a structured JSON report with resolution status and confidence level."
2. Task tool with subagent_type="pr-comment-handler" and prompt="Analyze PR comments for PR number [NUMBER]. Fetch both review comments and issue comments, categorize them, and provide a structured JSON report with reply drafts and code change requirements."
3. Task tool with subagent_type="ci-checker" and prompt="Check CI status for PR number [NUMBER]. Analyze any failures, fetch logs, diagnose issues, and provide a structured JSON report with fix strategies."

Wait for all three agents to complete and return their JSON outputs.

---

## Phase 3: Integration & Execution

### Step 3.1: Review Agent Results

Review the outputs from all three agents:

**From conflict-resolver:**
- Merge conflict status (MERGEABLE/CONFLICTING)
- Rebase attempt result (SUCCESS/CONFLICTS/ERROR)
- Auto-resolved conflicts and confidence level
- Any manual review warnings

**From pr-comment-handler:**
- Comments requiring replies
- Code changes needed based on feedback
- Reply drafts

**From ci-checker:**
- CI check status (pass/fail)
- Failed checks and their causes
- Proposed fixes

### Step 3.2: Prioritize Actions

Determine the execution order:

1. **Conflict Resolution** - Handle merge conflicts first (if not auto-resolved)
2. **CI Fixes** - Fix any failing CI checks (tests, builds, lints)
3. **Code Changes** - Apply changes requested in PR comments
4. **Comment Replies** - Post replies to PR comments

**Note:** If conflict-resolver completed rebase successfully, the branch is rebased locally but NOT yet pushed. You will need to push with `--force-with-lease` after user confirmation.

### Step 3.3: Execute Fixes

For each required fix:

1. **Make Code Changes:**
   - Use Edit tool to apply fixes
   - Verify changes with Read tool
   - Run relevant tests if applicable

2. **Commit Changes:**
   ```bash
   !git add .
   !git commit -m "[commit message describing fixes]"
   ```

3. **Review Changes Before Push:**

   Display a summary of what will be pushed:
   ```bash
   !git log origin/[BRANCH]..HEAD --oneline
   !git diff origin/[BRANCH]..HEAD --stat
   ```

   **Ask user for confirmation before pushing:**

   Show the user:
   - Number of commits to be pushed
   - Files changed
   - Whether this will be a force push (if rebase was performed)

   Use the AskUserQuestion tool to ask:
   - Question: "Ready to push these changes to the remote branch?"
   - Options:
     - "Yes, push now" - Proceed with push
     - "No, let me review" - Stop and let user examine changes manually

   Only proceed with push if user confirms "Yes, push now".

4. **Push Changes (after confirmation):**
   ```bash
   # If conflict-resolver performed a rebase, use --force-with-lease
   !git push --force-with-lease

   # Otherwise, use regular push
   !git push
   ```

   **Note:** Use `--force-with-lease` after rebase to safely overwrite history. Use regular `git push` if no rebase was performed.

### Step 3.4: Post Comment Replies

For each comment requiring a reply, use the appropriate reply method based on comment type:

**For review comments (code-specific comments):**

Use GitHub API to reply to the specific comment thread:

```bash
!gh api repos/[OWNER]/[REPO]/pulls/comments/[COMMENT_ID]/replies -f body="[reply message]"
```

This creates a threaded reply directly on the review comment.

**For issue comments (general PR comments):**

Use @mention to reply since issue comments don't support threading:

```bash
!gh pr comment [PR_NUMBER] --body "@[author] [reply message]"
```

**Implementation:**

Review the JSON from pr-comment-handler agent. For each comment with a reply:

1. Check the `comment_type` field
2. If `"review_comment"`: Use `gh api repos/.../pulls/comments/[id]/replies`
3. If `"issue_comment"`: Use `gh pr comment` with @mention
4. Use the `reply` text from the JSON output

---

## Phase 4: Verification & Summary

### Step 4.1: Verify CI Status

After pushing changes, wait briefly and check CI status:

```bash
!gh pr checks
```

### Step 4.2: Generate Summary Report

Provide a comprehensive summary:

```markdown
## PR Fix Summary

**PR:** #[number] - [title]

### Actions Taken:

#### Conflict Resolution
- [x] Checked merge status: [MERGEABLE/CONFLICTING]
- [x] Rebased onto latest [base_branch]
- [x] Resolved [N] conflicts in [M] files
- [x] Confidence: [HIGH/MEDIUM/LOW]

#### CI Fixes
- [x] Fixed [issue 1]
- [x] Fixed [issue 2]

#### Code Changes
- [x] Applied feedback from @[reviewer]: [description]
- [x] Updated [file] per comment

#### Comments Replied
- [x] Replied to @[reviewer] on [topic]

### Current Status:
- Merge Status: [MERGEABLE/CONFLICTING]
- CI Checks: [pass/pending/fail]
- Pending Actions: [any remaining items]

### Next Steps:
[Any recommended follow-up actions]
```

---

## Success Checklist

Before completing, verify:

- [ ] PR was found and is in "open" state
- [ ] All three agents completed their analysis
- [ ] Merge conflicts have been checked and resolved if needed
- [ ] All CI failures have been addressed
- [ ] All PR comment feedback has been addressed or replied to
- [ ] Changes have been committed locally
- [ ] User was asked for confirmation before push
- [ ] User confirmed and changes were pushed (with --force-with-lease if rebased)
- [ ] Comment replies have been posted
- [ ] Final CI status has been checked
- [ ] Final merge status has been checked

---

## Key Principles

1. **Parallel Efficiency** - Launch all three agents simultaneously to save time
2. **Conflict First** - Always resolve merge conflicts before other fixes
3. **CI Second** - Fix CI failures after conflicts are resolved
4. **Clear Communication** - Post clear replies explaining changes made
5. **Atomic Commits** - Make focused commits for each type of fix
6. **User Confirmation** - Always ask user before pushing changes to remote
7. **Safe Force Push** - Use --force-with-lease instead of --force after rebase
8. **Verification** - Always check CI status after pushing changes
9. **Completeness** - Address all feedback, don't leave items unresolved

---

## Error Handling

- If no PR found: Inform user and suggest creating a PR first
- If gh CLI not installed: Provide installation instructions
- If not authenticated: Direct user to `gh auth login`
- If agents fail: Report the error and suggest manual intervention
- If conflicts cannot be auto-resolved: Report conflicts and suggest manual resolution
- If rebase fails: Abort rebase and inform user of the issue
- If user declines push: Stop execution, inform user that changes are committed locally but not pushed
- If CI still failing: Report status and suggest reviewing logs manually
- If force push fails: Check if branch is protected or if --force-with-lease safety check failed
