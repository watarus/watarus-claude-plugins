# Design Doc Toolkit

A Claude Code plugin for generating comprehensive system design documents with multi-model AI analysis.

## Overview

This plugin helps you create professional system design documents by:

1. Analyzing your existing codebase with multiple AI models
2. Synthesizing insights from different model perspectives
3. Generating well-structured design documents following industry standards

## Features

- **Multi-Model Analysis**: Leverage Sonnet, Opus, and Codex MCP for comprehensive insights
- **Multiple Document Types**: New features, refactoring, API design, data models
- **Codebase-Aware**: Analyzes existing patterns and conventions
- **Standard Format**: Generates documents with all essential sections
- **Interactive Workflow**: Guide you through design decisions

## Installation

Add to your Claude Code plugins:

```bash
claude plugins:add watarus/watarus-claude-plugins/design-doc-toolkit
```

## Usage

### Create a Design Document

```bash
/design-doc-toolkit:create-design-doc user authentication system
```

Or simply:

```bash
/create-design-doc payment processing feature
```

### Workflow

1. **Select Document Type**
   - New Feature
   - System Refactoring
   - API Design
   - Data Model Design

2. **Choose Analysis Models**
   - Sonnet: Fast, practical analysis
   - Opus: Deep reasoning, complex trade-offs
   - Codex MCP: Technical implementation details

3. **Define Scope**
   - Analyze existing code
   - Describe requirements
   - Both
   - Greenfield design

4. **Review Analysis**
   - Synthesized insights from all models
   - Architecture patterns
   - Integration points
   - Security considerations

5. **Generate Document**
   - Comprehensive design document
   - Industry-standard format
   - All relevant sections

## Document Structure

Generated documents include:

- **Overview**: Summary, background, goals, non-goals
- **Requirements**: Functional, non-functional, constraints
- **System Architecture**: High-level design, components
- **Detailed Design**: Component design, data models, APIs
- **Security**: Authentication, authorization, data protection
- **Performance**: Targets, scalability, caching
- **Reliability**: Error handling, failure modes, monitoring
- **Testing Strategy**: Unit, integration, E2E, load testing
- **Migration & Rollout**: Plan, rollback, feature flags
- **Alternatives Considered**: Other approaches and trade-offs

## Agents

### system-analyzer

Analyzes codebases to understand existing architecture, patterns, data flows, and dependencies.

**Capabilities:**
- Architecture pattern identification
- Component dependency mapping
- Data model analysis
- API pattern recognition
- Security observation
- Integration point discovery

### design-doc-writer

Generates well-structured design documents following best practices.

**Capabilities:**
- Standard document formatting
- Section-by-section generation
- Code example inclusion
- Diagram descriptions
- TBD marking for incomplete sections

## Example Output

```markdown
# User Authentication System Design Document

**Author**: Development Team
**Date**: 2024-01-15
**Status**: Draft

## 1. Overview

### 1.1 Summary
Design for implementing user authentication with OAuth2/JWT support...

### 1.2 Goals
- Secure user authentication with industry standards
- Support for multiple authentication providers
- Session management with refresh tokens
...
```

## Configuration

Documents are saved to:
- `docs/design/` (default)
- Custom path if specified

## Requirements

- Claude Code CLI
- Optional: Codex MCP for technical analysis

## License

MIT
