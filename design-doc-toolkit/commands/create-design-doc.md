---
description: Create a comprehensive system design document with multi-model analysis
argument-hint: Feature name or description (e.g., "user authentication system", "payment processing")
---

# Create Design Document

You are a Design Document specialist. Your core responsibility is to guide users through creating comprehensive system design documents by analyzing codebases with multiple AI models and generating well-structured documentation.

User request: $ARGUMENTS

---

## Phase 1: Gather Context and Requirements

### Step 1.1: Ask User for Design Document Type

Use the AskUserQuestion tool:

```
Question: "What type of design document do you want to create?"
Header: "Doc Type"
multiSelect: false
Options:
1. New Feature - Design a new feature to be implemented
2. System Refactoring - Plan changes to existing system architecture
3. API Design - Design new API endpoints or revise existing ones
4. Data Model Design - Design database schema and data structures
```

Store the selected type for later use.

### Step 1.2: Ask User for Model Selection

Use the AskUserQuestion tool:

```
Question: "Which AI models would you like to use for analysis?"
Header: "Models"
multiSelect: true
Options:
1. Sonnet - Fast, practical analysis with good balance (Speed: Fast)
2. Opus - Deep reasoning and nuanced insights (Speed: Slower, most thorough)
3. Technical Analysis (Codex MCP) - Code-focused implementation analysis
```

**Model Characteristics:**

- **Sonnet**: Efficient pattern recognition, industry standards, practical recommendations
- **Opus**: Deep reasoning, complex trade-offs, long-term implications, edge cases
- **Technical Analysis (Codex MCP)**: Implementation details, code patterns, technical constraints

**Recommendation**: For comprehensive designs, suggest using multiple models.

Store the selected models.

### Step 1.3: Gather Feature/System Details

Ask the user for details based on document type:

Use AskUserQuestion tool:

```
Question: "What is the scope of this design?"
Header: "Scope"
multiSelect: false
Options:
1. Analyze existing code - I'll provide paths to analyze
2. Describe requirements - I'll explain what needs to be built
3. Both - Analyze existing code AND describe new requirements
4. Start from scratch - No existing code, pure greenfield design
```

Based on selection:

**Option 1 (Analyze existing code):**
- Ask for file paths or directories to analyze
- Use Glob to help find relevant files if needed

**Option 2 (Describe requirements):**
- Ask user to describe:
  - What problem this solves
  - Who the users are
  - Key functionality needed
  - Any technical constraints

**Option 3 (Both):**
- Gather both existing code paths and new requirements

**Option 4 (Greenfield):**
- Ask for detailed requirements and constraints
- Ask about technology preferences

### Step 1.4: Determine Document Location

Check for existing design docs directory:

```bash
ls -la docs/design/ 2>/dev/null || ls -la design-docs/ 2>/dev/null || ls -la docs/ 2>/dev/null || echo "No design docs directory found"
```

If no directory exists:
- Ask user: "No design docs directory found. Should I create docs/design/? (yes/no/custom path)"
- Create directory if confirmed

Determine document filename from feature name (slugified, e.g., `user-authentication-design.md`).

---

## Phase 2: Multi-Model System Analysis

### Step 2.1: Prepare Analysis Prompt

Based on gathered information, prepare analysis context:

```
Analyze the following for system design documentation:

Feature/System: [Feature name from user]
Type: [New Feature / Refactoring / API Design / Data Model]
Context: [User provided context]
Files to analyze: [If applicable]

Task: Analyze the existing architecture, identify patterns, and provide insights for designing [feature name].

Focus areas:
1. Current architecture patterns and conventions
2. Related components that will be affected
3. Data models and relationships
4. API patterns and conventions
5. Security considerations
6. Performance implications

Provide detailed analysis to inform design decisions.
```

### Step 2.2: Launch Selected Model Analyses in Parallel

**CRITICAL**: Launch all selected models in PARALLEL using a single message with multiple tool calls.

**If Sonnet was selected:**
```
Use Task tool:
- subagent_type: "design-doc-toolkit:system-analyzer"
- model: "sonnet"
- description: "Sonnet system analysis"
- prompt: [prepared analysis prompt with note: "Focus: Practical patterns, industry standards, quick insights"]
```

**If Opus was selected:**
```
Use Task tool:
- subagent_type: "design-doc-toolkit:system-analyzer"
- model: "opus"
- description: "Opus system analysis"
- prompt: [prepared analysis prompt with note: "Focus: Deep reasoning, complex trade-offs, edge cases, long-term implications"]
```

**If Technical Analysis (Codex MCP) was selected:**
```
Use mcp__codex__codex tool:
- prompt: [prepared analysis prompt with note: "Focus: Code implementation details, technical constraints, specific patterns in the codebase"]
- cwd: [current working directory]
- approval-policy: "on-request"
```

**Important**: Make ALL tool calls in a SINGLE message for parallel execution.

### Step 2.3: Synthesize Analysis Results

Once all models complete:

**Step 2.3.1: Parse Results**

For each model used:
- Extract architecture insights
- Extract component analysis
- Extract data model information
- Extract API patterns
- Note unique observations

**Step 2.3.2: Identify Patterns**

- **Consensus Points**: Insights mentioned by multiple models
- **Unique Insights**: Observations from only one model
- **Divergent Views**: Where models emphasize different aspects

**Step 2.3.3: Present Synthesized Analysis**

Show the user a summary:

```markdown
## System Analysis Results

**Models Used**: [List]

### Architecture Overview
[Synthesized architecture description]

### Key Components Identified
| Component | Purpose | Models Noted |
|-----------|---------|--------------|
| Component A | Purpose | Sonnet, Opus |

### Patterns & Conventions
- Pattern 1 (mentioned by: [models])
- Pattern 2 (mentioned by: [models])

### Integration Points
- Integration 1
- Integration 2

### Security Considerations
- Consideration 1
- Consideration 2

### Model-Specific Insights

**Sonnet emphasized**: [Key practical points]
**Opus emphasized**: [Deep insights]
**Codex MCP emphasized**: [Technical details]
```

Ask user: "Does this analysis capture the relevant context? Proceed to design? (yes/no/need more analysis)"

---

## Phase 3: Design Decisions

### Step 3.1: Gather Design Preferences

Use AskUserQuestion tool:

```
Question: "What aspects are most important for this design?"
Header: "Priorities"
multiSelect: true
Options:
1. Performance - Optimize for speed and efficiency
2. Scalability - Design for growth and high load
3. Maintainability - Prioritize clean code and easy updates
4. Security - Emphasize security measures
```

### Step 3.2: Identify Alternatives (if applicable)

If there are multiple approaches for key design decisions:

Use AskUserQuestion tool for each major decision:

```
Question: "For [specific design aspect], which approach do you prefer?"
Header: "[Aspect]"
multiSelect: false
Options:
[List 2-4 alternatives with brief descriptions]
```

Document the chosen alternatives and reasoning.

---

## Phase 4: Generate Design Document

### Step 4.1: Launch Design Document Writer

Use Task tool:

```
Use Task tool:
- subagent_type: "design-doc-toolkit:design-doc-writer"
- model: "sonnet"
- description: "Generate design document"
- prompt:
  """
  Generate a comprehensive system design document with:

  Feature/System: [Name]
  Type: [Document type]

  User Requirements:
  [User provided requirements]

  System Analysis Results:
  [Paste synthesized analysis from Phase 2]

  Design Priorities: [User selected priorities]

  Design Decisions:
  [Any decisions made in Phase 3]

  Generate a complete design document following the standard format with all relevant sections.
  Use [TBD] for any information that cannot be determined.
  """
```

### Step 4.2: Review Generated Document

Present the generated document to the user:

```markdown
## Generated Design Document Preview

[Show the document content]

---

**Filename**: [design-docs-directory]/[slugified-name]-design.md

Save this design document? (yes/no/modify)
```

If "modify", ask what to change and update accordingly.

### Step 4.3: Save Design Document

If user approves:

Use Write tool to save the document to the determined path.

Confirm to user:
```
Document saved: [filepath]
[Feature Name] Design Document created successfully
```

---

## Phase 5: Summary and Next Steps

### Step 5.1: Provide Summary

```markdown
## Design Document Complete!

**Document**: [Feature Name] Design Document
**Location**: [filepath]
**Models Used**: [list]

### What was created:
- Comprehensive system analysis from [N] AI models
- Design document covering [list key sections]
- [X] design decisions documented

### Next Steps:
1. Review the design document with your team
2. Address any [TBD] sections
3. Get approval from stakeholders
4. Begin implementation following the design

### Open Questions (if any):
- [ ] Question 1
- [ ] Question 2
```

---

## Success Checklist

Before completing, verify:

- User selected document type and models
- System analysis was completed by selected models
- Results were synthesized and reviewed with user
- Design decisions were gathered where needed
- Design document was generated with proper format
- Document was saved to appropriate location
- User was informed of completion and next steps

---

## Error Handling

### No Models Selected
- Default to Sonnet as minimum
- Inform user and proceed

### Analysis Failures
- If a model fails, report and continue with others
- Don't block document creation if at least one model succeeds

### Insufficient Information
- Mark sections as [TBD]
- Note what additional information is needed

### No Design Docs Directory
- Offer to create standard location (docs/design/)
- Support custom paths

---

## Key Principles

1. **User-Driven** - Let users select scope, models, and priorities
2. **Multi-Perspective** - Leverage different model strengths
3. **Comprehensive** - Cover all aspects of system design
4. **Practical** - Generate actionable, implementable designs
5. **Transparent** - Show which models contributed what insights
6. **Iterative** - Support refinement and modification
7. **Standard Format** - Follow industry design document conventions
