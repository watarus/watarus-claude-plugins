# ADR Toolkit

Analyze codebase architecture and generate Architecture Decision Records (ADRs) using multi-model AI analysis.

## Overview

ADR Toolkit is a Claude Code plugin that helps you create comprehensive Architecture Decision Records by leveraging multiple AI models (Sonnet, Opus, and Codex) to analyze your codebase from different perspectives. This multi-model approach provides:

- **Diverse perspectives** on architectural decisions
- **Comprehensive trade-off analysis** with pros and cons
- **Balanced recommendations** based on consensus and unique insights
- **Well-structured ADR documents** following industry best practices

## Features

### Commands

- **`/create-adr [context]`** - Create a new ADR by analyzing code or documenting decisions
  - Optional context: files to analyze, feature description, or decision details
  - Interactive model selection (Sonnet, Opus, Codex)
  - Parallel multi-model analysis for efficiency
  - Automatic ADR numbering and organization

### Agents

- **architecture-analyzer** - Deeply analyze codebase architecture patterns
  - Identifies architectural alternatives and patterns
  - Evaluates trade-offs across multiple dimensions
  - Provides specific pros/cons with examples
  - Can run as Sonnet (practical) or Opus (deep) for different perspectives

- **adr-writer** - Generate well-structured ADR documents
  - Transforms analysis into standard ADR format
  - Attributes insights to specific models
  - Follows ADR best practices
  - Includes comprehensive documentation

## Installation

### Prerequisites

- Claude Code CLI installed
- Access to the plugin marketplace

### Install from Marketplace

```bash
# Add this plugin marketplace
/plugin marketplace add watarus/watarus-claude-plugins

# Install the plugin
/plugin install adr-toolkit
```

### Manual Installation

```bash
# Clone the repository
git clone https://github.com/watarus/watarus-claude-plugins.git

# Add as local marketplace
/plugin marketplace add /path/to/watarus-claude-plugins

# Install the plugin
/plugin install adr-toolkit
```

## Usage

### Basic Usage

```bash
# Create ADR with model selection prompt
/create-adr Analyze authentication approach in src/auth/

# Create ADR with specific context
/create-adr Compare database options for user service

# Create ADR from broad analysis
/create-adr Analyze overall architecture patterns
```

### Model Selection

When you run `/create-adr`, you'll be prompted to select which AI models to use:

- **Sonnet** ⚡ - Fast, practical analysis
  - Focus: Industry standards, common patterns
  - Best for: Quick decisions, established patterns

- **Opus** 🐢 - Deep, nuanced analysis
  - Focus: Complex trade-offs, long-term implications
  - Best for: Critical decisions, complex systems

- **Codex** ⚙️ - Technical, code-focused analysis
  - Focus: Implementation details, technical constraints
  - Best for: Technical architecture decisions

**Recommendation**: Use all three models for comprehensive analysis, or select based on:
- Time constraints (Sonnet for speed)
- Decision complexity (Opus for depth)
- Technical focus (Codex for implementation)

### Workflow Example

1. **Run the command**:
   ```bash
   /create-adr Compare REST vs GraphQL for API layer
   ```

2. **Select models**:
   - Choose Sonnet, Opus, and Codex for comprehensive analysis
   - Or select just Sonnet + Codex for faster technical analysis

3. **Provide context**:
   - Select analysis scope (specific files, feature, or broad patterns)
   - Provide relevant file paths or descriptions

4. **Review analysis**:
   - Multi-model analysis runs in parallel
   - Results are synthesized showing consensus and unique insights

5. **Generate ADR**:
   - Approve the analysis
   - ADR is generated with all perspectives documented
   - File saved to `docs/adr/NNNN-title.md`

## ADR Format

The plugin generates ADRs in the following format:

```markdown
# 0001. [Title]

Date: YYYY-MM-DD
Analysis Models: Sonnet, Opus, Codex

## Status
[proposed | accepted | deprecated | superseded]

## Context
[Problem description and background]

## Analysis Methodology
[Which models were used and their focus areas]

## Alternatives Considered

### Alternative 1: [Name]
**Description**: [Brief description]
**Pros**:
- [Benefit] *(Consensus: All models)*
- [Benefit] *(Opus insight)*

**Cons**:
- [Drawback] *(Codex: technical constraint)*

**Technical Considerations**: [Details]
**Trade-offs**: [Performance, Maintainability, etc.]

### Alternative 2: [Name]
[...]

## Decision
[Which alternative was chosen and why]

## Consequences
### Positive
- [Consequence]

### Negative
- [Consequence]

## Model Perspectives Summary
- **Sonnet**: [Key practical points]
- **Opus**: [Deep insights]
- **Codex**: [Technical details]

## Related Decisions
[Links to related ADRs]
```

## Configuration

### ADR Directory

By default, ADRs are saved to:
- `docs/adr/` (preferred)
- `doc/adr/` (alternative)

If no ADR directory exists, the plugin will offer to create one.

### ADR Numbering

ADRs are automatically numbered sequentially:
- `0001-first-decision.md`
- `0002-second-decision.md`
- etc.

## Best Practices

### When to Use Multiple Models

- **All three models**: For critical architectural decisions
- **Sonnet + Opus**: For balanced practical + deep analysis
- **Sonnet + Codex**: For fast technical analysis
- **Single model**: For documenting existing decisions

### Model Strengths

- **Sonnet**: Quick iterations, standard patterns
- **Opus**: Complex systems, subtle trade-offs
- **Codex**: Implementation details, code patterns

### ADR Writing Tips

1. **Clear Context**: Explain the problem, not the solution
2. **Fair Alternatives**: Present all options objectively
3. **Specific Trade-offs**: Use concrete examples and metrics
4. **Honest Consequences**: Include negative impacts
5. **Attribution**: Show which models contributed insights

## Examples

### Database Selection

```bash
/create-adr Analyze src/database/ for database choice decision

# Select: Sonnet, Opus, Codex
# Scope: Analyze specific files in src/database/
# Result: ADR comparing PostgreSQL, MongoDB, DynamoDB
```

### Authentication Strategy

```bash
/create-adr Document authentication approach in auth module

# Select: Sonnet, Opus
# Scope: Analyze src/auth/ directory
# Result: ADR comparing JWT, Session, OAuth2
```

### Microservices vs Monolith

```bash
/create-adr Evaluate overall architecture: microservices vs monolith

# Select: All three models
# Scope: Broad codebase analysis
# Result: Comprehensive ADR with deep trade-off analysis
```

## Troubleshooting

### No ADR directory found

The plugin will prompt to create `docs/adr/`. You can:
- Accept the default location
- Provide a custom path
- Create the directory manually first

### Model analysis fails

If a model fails to complete analysis:
- The plugin continues with successful analyses
- At least one model must succeed to generate the ADR
- Check model availability and quotas

### ADR number conflicts

If an ADR number already exists:
- The plugin automatically increments to the next number
- Existing ADRs are never overwritten

## Contributing

Contributions are welcome! Please feel free to:
- Report issues
- Suggest improvements
- Submit pull requests

Repository: https://github.com/watarus/watarus-claude-plugins

## License

MIT License - See LICENSE file for details

## Author

**watarus**
- GitHub: https://github.com/watarus
- Repository: https://github.com/watarus/watarus-claude-plugins

## Related Resources

- [ADR Best Practices](https://adr.github.io/)
- [Claude Code Documentation](https://docs.claude.com/en/docs/claude-code)
- [Plugin Development Guide](https://docs.claude.com/en/docs/claude-code/plugins)

---

*Built with Claude Code Plugin Builder*
