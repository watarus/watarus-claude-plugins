# GitHub PR Fix

Automate GitHub PR comment responses and CI fixes with intelligent parallel analysis.

## Overview

This plugin streamlines the pull request review workflow by automatically:

1. **Resolving merge conflicts** - Detecting conflicts, rebasing with latest base branch, and auto-resolving where possible
2. **Analyzing PR comments** - Understanding reviewer feedback and determining appropriate responses
3. **Checking CI status** - Identifying and diagnosing test failures, build errors, and other CI issues
4. **Applying fixes** - Making code changes to address feedback and resolve CI failures
5. **Posting replies** - Responding to reviewer comments with clear explanations

All analysis happens in parallel for maximum efficiency.

## Prerequisites

- [GitHub CLI](https://cli.github.com/) installed and authenticated
- An active pull request for your current branch
- Claude Code with plugin support

## Installation

### Option 1: Install from this repository

```bash
# Add this repository as a marketplace
/plugin marketplace add watarus/watarus-claude-plugins

# Install the plugin
/plugin install github-pr-fix
```

### Option 2: Local installation

```bash
# Clone this repository
git clone https://github.com/watarus/watarus-claude-plugins.git

# Link the plugin
/plugin add /path/to/watarus-claude-plugins/github-pr-fix
```

## Features

### Commands

#### `/github-pr-fix`

Automatically handle all aspects of PR maintenance for your current branch:

- Fetches PR information
- Launches parallel analysis of conflicts, comments, and CI status
- Resolves merge conflicts by rebasing with latest base branch
- Applies code fixes for both review feedback and CI failures
- Posts comment replies
- Verifies fixes with updated CI status

**Usage:**

```bash
# Fix everything for the current branch's PR
/github-pr-fix
```

The command will:
1. Detect your PR automatically
2. Run three specialized agents in parallel
3. Prioritize and execute all necessary fixes
4. Provide a comprehensive summary

### Agents

The plugin uses three specialized agents that work in parallel:

#### `conflict-resolver`

Detects and resolves merge conflicts:
- Checks PR mergeable status
- Fetches latest base branch
- Attempts automatic rebase
- Resolves simple conflicts intelligently
- Flags complex conflicts for manual review

**Output:** Structured JSON with resolution status and confidence

#### `pr-comment-handler`

Analyzes all PR comments and determines:
- Which comments need replies
- Which require code changes
- Priority and complexity of each action
- Draft replies for reviewer questions

**Output:** Structured JSON with actionable items

#### `ci-checker`

Examines CI check status and failures:
- Categorizes failure types (tests, build, lint, etc.)
- Extracts error messages and locations
- Proposes specific fixes
- Estimates complexity and confidence

**Output:** Structured JSON with fix strategies

## Usage Examples

### Basic workflow

```bash
# You've received PR feedback, have merge conflicts, and CI is failing
git checkout feature-branch

# Run the automated fix
/github-pr-fix
```

The plugin will:
1. Find PR #123 for your branch
2. Check and resolve merge conflicts by rebasing
3. Analyze 5 review comments (3 need replies, 2 need code changes)
4. Check CI status (2 tests failing, 1 lint error)
5. Fix all issues in priority order
6. Commit and push changes (with --force-with-lease if rebased)
7. Post comment replies
8. Show summary report

### What gets fixed automatically

**Merge Conflicts:**
- ✅ Simple overlapping changes
- ✅ Non-semantic conflicts
- ✅ Automatic rebase with latest base branch
- ⚠️  Complex semantic conflicts (flagged for manual review)

**CI Failures:**
- ✅ Test failures with clear error messages
- ✅ Build/compilation errors
- ✅ Lint and formatting issues
- ✅ Type checking errors
- ⚠️  Complex test failures (will propose fix for review)

**PR Comments:**
- ✅ Questions (drafts replies)
- ✅ Simple change requests
- ✅ Code suggestions
- ⚠️  Architectural discussions (drafts reply for review)

## How It Works

### Parallel Processing

The plugin maximizes efficiency by running analyses simultaneously:

```
/github-pr-fix
     ↓
Get PR Info (#123)
     ↓
┌────────────────────┬──────────────────────┬──────────────────────┐
│ conflict-resolver  │ pr-comment-handler   │ ci-checker           │
│ • Check mergeable  │ • Fetch comments     │ • Check CI status    │
│ • Fetch base       │ • Categorize         │ • Get failure logs   │
│ • Attempt rebase   │ • Draft replies      │ • Diagnose issues    │
│ • Resolve conflicts│ • Plan fixes         │ • Propose solutions  │
└────────────────────┴──────────────────────┴──────────────────────┘
     ↓                        ↓                        ↓
     └────────────────────────┬────────────────────────┘
                              ↓
                     Integrate Results
                              ↓
                     Priority Order:
                     1. Conflict resolution
                     2. CI fixes
                     3. Code changes
                     4. Comment replies
                              ↓
                     Execute & Push
                     (with --force-with-lease if rebased)
                              ↓
                     Summary Report
```

### Priority System

Fixes are applied in this order:

1. **Merge Conflicts** - Resolve conflicts and rebase first
2. **CI Failures** - Tests and builds must pass
3. **Code Changes** - Implement reviewer feedback
4. **Comment Replies** - Explain changes and answer questions

## Configuration

No configuration needed! The plugin works out of the box with:
- GitHub CLI defaults
- Current git branch detection
- Standard PR and CI workflows

## Troubleshooting

### "No pull request found"

Make sure:
- You're on a branch with an open PR
- GitHub CLI is authenticated: `gh auth status`
- You're in a git repository with a GitHub remote

### "gh: command not found"

Install GitHub CLI:
```bash
# macOS
brew install gh

# Linux
# See https://github.com/cli/cli/blob/trunk/docs/install_linux.md

# Windows
# See https://github.com/cli/cli#windows
```

Then authenticate:
```bash
gh auth login
```

### Agent errors

If an agent fails:
- Check your internet connection
- Verify GitHub API access with `gh api user`
- Review the error message for specific issues
- Try running the command again

## Limitations

- Requires GitHub CLI and authentication
- Only works with GitHub PRs (not GitLab, Bitbucket, etc.)
- Complex semantic conflicts require manual resolution
- Complex architectural changes may need manual review
- Flaky or intermittent CI failures are flagged but not auto-fixed
- Force push uses --force-with-lease for safety (may fail if branch was updated remotely)

## Contributing

Found a bug or want to improve the plugin?

1. Open an issue in this repository
2. Submit a pull request with improvements
3. Share feedback on usage and edge cases

## License

MIT License - See LICENSE file for details

## Author

Created by watarus

## Resources

- [GitHub CLI Documentation](https://cli.github.com/manual/)
- [Claude Code Plugin Docs](https://docs.claude.com/docs/claude-code)
- [Plugin Development Guide](https://docs.claude.com/docs/claude-code/plugins)
