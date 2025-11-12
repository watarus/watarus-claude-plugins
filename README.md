# Watarus Claude Plugins & Skills

Personal collection of Claude Code plugins and skills for development workflow automation.

## Available Plugins

### github-pr-fix

Automate GitHub PR maintenance with intelligent parallel analysis.

**Version:** 1.1.3

**Features:**
- ✅ Automatic merge conflict resolution with intelligent rebase
- ✅ PR comment analysis and threaded replies
- ✅ CI failure diagnosis and automatic fixes
- ✅ User confirmation before pushing changes
- ✅ Parallel processing for maximum efficiency

**[View Documentation →](./github-pr-fix/README.md)**

---

### atomic-commit

Analyze changes and create atomic, compilable commits.

**Version:** 1.0.0

**Features:**
- ✅ Intelligent change analysis and grouping
- ✅ Atomic commits that pass build checks
- ✅ Commit order based on dependency analysis
- ✅ Safety backup branches
- ✅ Build verification for each commit

**[View Documentation →](./atomic-commit/README.md)**

---

### adr-toolkit

Analyze codebase architecture and generate Architecture Decision Records (ADRs) using multi-model analysis.

**Version:** 1.0.0

**Features:**
- ✅ Multi-model AI analysis (Sonnet, Opus, Codex)
- ✅ User-selectable model combinations
- ✅ Parallel model execution for efficiency
- ✅ Comprehensive trade-off analysis
- ✅ Standard ADR format with attribution
- ✅ Automatic ADR numbering and organization

**[View Documentation →](./adr-toolkit/README.md)**

---

## Available Skills

### go-table-test

Generate table-driven tests for Golang functions and methods using Given-When-Then structure.

**Version:** 1.0.0

**Features:**
- ✅ Table-driven test generation for any Go function
- ✅ Given-When-Then structure for clarity
- ✅ Mock setup with testify
- ✅ Database setup patterns
- ✅ Complex assertion support
- ✅ Automatic *_test.go file creation/updates
- ✅ Comprehensive pattern library

**[View Documentation →](./skills/go-table-test/SKILL.md)**

---

## Installation

### Add Marketplace

```bash
/plugin marketplace add watarus/watarus-claude-plugins
```

### Install Plugins

```bash
# Install github-pr-fix
/plugin install github-pr-fix

# Install atomic-commit
/plugin install atomic-commit

# Install adr-toolkit
/plugin install adr-toolkit
```

### Install Skills

```bash
# Install go-table-test
/skill install go-table-test
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

### atomic-commit

The `atomic-commit` plugin helps you create clean, atomic commits that are independently compilable and revertable:

1. **Analyze changes** - Intelligently groups related changes
2. **Determine order** - Respects dependencies between changes
3. **Verify builds** - Each commit must pass compilation
4. **Safety first** - Creates backup branches before committing
5. **Clear messages** - Generates meaningful commit messages

**Usage:**

```bash
/atomic-commit
```

The plugin will analyze your staged and unstaged changes, create a commit plan, and execute it with build verification.

---

### adr-toolkit

The `adr-toolkit` plugin revolutionizes architecture documentation by leveraging multiple AI models:

1. **Model selection** - Choose Sonnet (fast), Opus (deep), or Codex (technical)
2. **Parallel analysis** - All selected models run simultaneously
3. **Alternative identification** - Discovers architectural options with trade-offs
4. **Consensus building** - Shows where models agree and disagree
5. **ADR generation** - Creates comprehensive, well-structured ADRs

**Usage:**

```bash
# Create ADR with model selection
/create-adr Analyze authentication approach in src/auth/

# Compare database options
/create-adr Compare PostgreSQL vs MongoDB for user service
```

The plugin will:
1. Ask which models to use (Sonnet, Opus, Codex)
2. Gather analysis context
3. Run parallel multi-model analysis
4. Synthesize results with attribution
5. Generate ADR with automatic numbering

---

## Skill Overview

### go-table-test

The `go-table-test` skill generates comprehensive table-driven tests for Golang functions:

1. **Function analysis** - Identifies inputs, outputs, dependencies, and error cases
2. **Mock generation** - Creates testify/mock implementations for interfaces
3. **Table structure** - Generates test cases with Given-When-Then pattern
4. **Pattern support** - Handles pure functions, mocks, DB setup, context, complex assertions
5. **Best practices** - Uses testify assertions, proper error handling, cleanup patterns

**Usage:**

```bash
# Write tests for a specific function
Write tests for the CreateUser function

# Write tests for an entire file
Add tests to service.go

# Write tests for a package
Write tests for the entire auth package
```

The skill will:
1. Read the function/file implementation
2. Identify all test cases (success, error, edge cases)
3. Generate comprehensive table-driven tests
4. Create or update *_test.go files
5. Include mock definitions if needed

**Example generated test:**

```go
func TestCreateUser(t *testing.T) {
    tests := []struct {
        name      string
        input     *User
        setupMock func(*MockRepository)
        want      *User
        wantErr   bool
    }{
        {
            name: "success: creates new user",
            input: &User{Name: "John", Email: "john@example.com"},
            setupMock: func(m *MockRepository) {
                m.On("Create", mock.Anything).Return(&User{ID: 1}, nil)
            },
            want: &User{ID: 1, Name: "John"},
            wantErr: false,
        },
        {
            name: "error: duplicate email",
            input: &User{Name: "Jane", Email: "duplicate@example.com"},
            setupMock: func(m *MockRepository) {
                m.On("Create", mock.Anything).Return(nil, errors.New("duplicate"))
            },
            want: nil,
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Given: Setup mock
            mockRepo := new(MockRepository)
            if tt.setupMock != nil {
                tt.setupMock(mockRepo)
            }
            defer mockRepo.AssertExpectations(t)

            svc := NewService(mockRepo)

            // When: Execute
            got, err := svc.CreateUser(tt.input)

            // Then: Assert
            if tt.wantErr {
                assert.Error(t, err)
                return
            }
            assert.NoError(t, err)
            assert.Equal(t, tt.want, got)
        })
    }
}
```

---

## License

MIT License - See individual plugin directories for details

## Author

Created by watarus

## Resources

- [Claude Code Documentation](https://docs.claude.com/docs/claude-code)
- [Plugin Development Guide](https://docs.claude.com/docs/claude-code/plugins)
- [GitHub CLI Manual](https://cli.github.com/manual/)
