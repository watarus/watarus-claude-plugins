# Pre-CI Check Plugin

Analyze CI configuration and run lint/typecheck/test/build checks locally in parallel before pushing to verify CI will pass.

## Features

- Auto-detect CI platforms (GitHub Actions, CircleCI, GitLab CI, etc.)
- Parallel execution via 4 specialized sub-agents
- Aggregated results with fix suggestions

## Installation

```bash
claude plugin install pre-ci-check
```

## Usage

```bash
/pre-ci-check
```

## How It Works

```
/pre-ci-check (Command)
    │
    ├── Phase 1: CI Config Detection
    │   ├── Detect CI platform
    │   ├── Parse check commands
    │   └── Identify project stack
    │
    ├── Phase 2: Parallel Execution
    │   ├── ci-lint-checker (lint/format)
    │   ├── ci-type-checker (typecheck)
    │   ├── ci-test-runner (test)
    │   └── ci-build-checker (build)
    │
    └── Phase 3: Results & Fixes
        ├── Summary report
        ├── Fix suggestions by priority
        └── Auto-fix commands
```

## Sub-agents

| Agent | Role |
|-------|------|
| `ci-lint-checker` | ESLint, Prettier, golangci-lint, etc. |
| `ci-type-checker` | TypeScript, mypy, etc. |
| `ci-test-runner` | Jest, pytest, go test, etc. |
| `ci-build-checker` | npm build, go build, etc. |

## Supported CI Platforms

- GitHub Actions (`.github/workflows/*.yml`)
- CircleCI (`.circleci/config.yml`)
- GitLab CI (`.gitlab-ci.yml`)
- Jenkins (`Jenkinsfile`)
- Travis CI (`.travis.yml`)
- Azure DevOps (`azure-pipelines.yml`)
- Bitbucket Pipelines (`bitbucket-pipelines.yml`)

## Output Example

```markdown
## Pre-CI Check Results

### Summary

| Check | Status | Issues |
|-------|--------|--------|
| Lint | PASSED | 0 issues |
| TypeCheck | PASSED | 0 errors |
| Test | FAILED | 3 failures |
| Build | PASSED | 0 errors |

**Overall:** CI will likely FAIL

### Suggested Fixes
[Detailed fix suggestions for each failure]
```

## License

MIT
