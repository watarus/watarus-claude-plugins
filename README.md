# Watarus Claude Plugins

Personal collection of Claude Code plugins for development workflow automation.

## Available Plugins

### github-pr-fix

Automate GitHub PR maintenance with intelligent parallel analysis.

**Version:** 1.1.2

**Features:**
- ✅ Automatic merge conflict resolution with intelligent rebase
- ✅ PR comment analysis and threaded replies
- ✅ CI failure diagnosis and automatic fixes
- ✅ User confirmation before pushing changes
- ✅ Parallel processing for maximum efficiency

**[View Documentation →](./github-pr-fix/README.md)**

## Installation

### Add Marketplace

```bash
/plugin marketplace add watarus/watarus-claude-plugins
```

### Install Plugins

```bash
# Install github-pr-fix
/plugin install github-pr-fix
```

## Plugin Overview

### github-pr-fix

The `github-pr-fix` plugin streamlines your pull request workflow by automatically:

1. **Resolving merge conflicts** - Detects conflicts, rebases with latest base branch, and auto-resolves where possible
2. **Analyzing PR comments** - Understanding reviewer feedback and determining appropriate responses
3. **Checking CI status** - Identifying and diagnosing test failures, build errors, and other CI issues
4. **Applying fixes** - Making code changes to address feedback and resolve CI failures
5. **Posting replies** - Responding to reviewer comments with clear explanations

All analysis happens in parallel using three specialized agents:
- **conflict-resolver**: Handles merge conflicts and rebase operations
- **pr-comment-handler**: Analyzes and responds to PR comments
- **ci-checker**: Diagnoses and fixes CI failures

**Usage:**

```bash
# From any branch with an open PR
/github-pr-fix
```

The plugin will:
1. Detect your PR automatically
2. Run three specialized agents in parallel
3. Prioritize and execute all necessary fixes
4. Ask for your confirmation before pushing
5. Provide a comprehensive summary

## Requirements

- [GitHub CLI](https://cli.github.com/) installed and authenticated
- Claude Code with plugin support
- An active GitHub account with repository access

## Contributing

Found a bug or want to suggest improvements?

1. Open an issue in this repository
2. Submit a pull request with your changes
3. Share feedback on usage and edge cases

## Future Plugins

More plugins coming soon! This marketplace will grow with additional development workflow automation tools.

## License

MIT License - See individual plugin directories for details

## Author

Created by watarus

## Resources

- [Claude Code Documentation](https://docs.claude.com/docs/claude-code)
- [Plugin Development Guide](https://docs.claude.com/docs/claude-code/plugins)
- [GitHub CLI Manual](https://cli.github.com/manual/)
