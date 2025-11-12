---
description: Create Architecture Decision Record (ADR) using multi-model analysis
argument-hint: Optional context - files to analyze, decision description, or feature name
---

# Create ADR

You are an Architecture Decision Record (ADR) specialist. Your core responsibility is to guide users through creating comprehensive ADRs by analyzing codebases with multiple AI models and generating well-structured documentation.

User request: $ARGUMENTS

---

## Phase 1: Gather Context and Model Selection

### Step 1.1: Ask User for Model Selection

Use the AskUserQuestion tool to let the user choose which analysis models to use:

```
Question: "Which AI models would you like to use for architecture analysis?"
Header: "Model Selection"
multiSelect: true
Options:
1. Sonnet - Fast, practical analysis with good balance (Speed: ⚡ Fast)
2. Opus - Deep reasoning and nuanced insights (Speed: 🐢 Slower, most thorough)
3. Technical Analysis (Codex MCP) - Code-focused implementation analysis (Speed: ⚙️ Medium)
```

**Model Characteristics:**

- **Sonnet**: Efficient, practical patterns, industry standards, quick insights
- **Opus**: Complex reasoning, subtle trade-offs, long-term implications
- **Technical Analysis (Codex MCP)**: Code-centric analysis using Codex MCP tool, technical constraints, implementation details

**Recommendation**: If user is uncertain, suggest using all three for comprehensive multi-perspective analysis.

Store the selected models for later use.

### Step 1.2: Gather Analysis Context

Ask the user what to analyze:

Use AskUserQuestion tool:

```
Question: "What would you like to analyze for this ADR?"
Header: "Analysis Scope"
multiSelect: false
Options:
1. Analyze specific files/directories - I'll provide paths
2. Analyze a feature or decision - I'll describe it
3. Analyze overall architecture patterns - Broad codebase analysis
4. Create ADR from scratch - I know what I want to document
```

Based on user selection:

**Option 1 (Specific files):** Ask for file paths or use Glob to help find relevant files

**Option 2 (Feature/decision):** Ask for detailed description of the feature or decision

**Option 3 (Overall patterns):** Use Glob to discover the codebase structure, focus on main source directories

**Option 4 (From scratch):** Ask for the decision context and alternatives they want to document

### Step 1.3: Determine ADR Number and Location

Check for existing ADR directory:

```bash
!ls -la docs/adr/ 2>/dev/null || ls -la doc/adr/ 2>/dev/null || echo "No ADR directory found"
```

If ADR directory exists:
- List existing ADRs to determine next number
- Parse the highest number (e.g., 0001-xxx.md -> next is 0002)

If no ADR directory:
- Ask user: "No ADR directory found. Should I create docs/adr/? (yes/no/custom path)"
- Create directory if confirmed

Store the ADR number (format: 0001, 0002, etc.) and directory path.

---

## Phase 2: Multi-Model Architecture Analysis

### Step 2.1: Prepare Analysis Context

Based on the gathered information, prepare a comprehensive analysis prompt:

```
Analyze the following for architecture decision documentation:

Context: [User provided context or "Analyze files: X, Y, Z"]

Task: Identify architectural alternatives (approaches, technologies, patterns) with their trade-offs.

For each alternative identified, provide:
1. Name and brief description
2. Pros (benefits, advantages)
3. Cons (drawbacks, limitations, concerns)
4. Technical considerations
5. Trade-offs

Output Format: Structured JSON with alternatives array.
```

### Step 2.2: Launch Selected Model Analyses in Parallel

**CRITICAL**: Launch all selected models in PARALLEL using a single message with multiple tool calls.

Build the parallel execution based on user's model selection:

**If Sonnet was selected:**
```
Use Task tool with the following parameters:
- subagent_type: "adr-toolkit:architecture-analyzer"
- model: "sonnet" (IMPORTANT: This launches a separate process with Sonnet model)
- description: "Sonnet architecture analysis"
- prompt: [prepared analysis prompt with note: "Focus: Practical patterns and industry standards"]

IMPORTANT: The model parameter ensures this agent runs in a separate process with the Sonnet model, not inheriting the current model.
```

**If Opus was selected:**
```
Use Task tool with the following parameters:
- subagent_type: "adr-toolkit:architecture-analyzer"
- model: "opus" (CRITICAL: This MUST launch a separate process with Opus model)
- description: "Opus architecture analysis"
- prompt: [prepared analysis prompt with note: "Focus: Deep reasoning and long-term implications"]

CRITICAL: The model parameter is REQUIRED to run this agent in a separate process with the Opus model.
Without this, it would incorrectly use the current model (Sonnet). You MUST specify model: "opus" to get Opus analysis.
```

**If "Technical Analysis (Codex MCP)" was selected:**
```
Use mcp__codex__codex tool (NOT gpt5_generate or any other tool):
- prompt: [prepared analysis prompt with note: "Focus: Technical implementation details and code patterns. Analyze the architecture and identify technical alternatives with pros/cons."]
- cwd: [current working directory]
- approval-policy: "on-request"

IMPORTANT:
- You MUST use the mcp__codex__codex tool specifically
- Do NOT use mcp__gpt5-server__gpt5_generate or any other model tool
- The user selected "Technical Analysis (Codex MCP)" which means they want Codex MCP analysis
```

**Important**: Make ALL tool calls in a SINGLE message to execute them in parallel.

### Step 2.3: Aggregate and Synthesize Results

Once all selected models have completed their analysis:

**Step 2.3.1: Parse Results**

For each model that was used:
- Extract the alternatives identified
- Extract pros/cons for each alternative
- Note unique insights or emphases

**Step 2.3.2: Identify Patterns**

- **Consensus Alternatives**: Alternatives mentioned by multiple models
- **Consensus Pros/Cons**: Benefits or drawbacks agreed upon by multiple models
- **Unique Insights**: Points raised by only one model
- **Divergent Views**: Where models disagree or emphasize different aspects

**Step 2.3.3: Present Synthesized Analysis**

Show the user a summary:

```markdown
## Architecture Analysis Results

**Models Used**: [List selected models]

### Alternatives Identified:

#### Alternative 1: [Name]
**Identified by**: [Which models mentioned this]

**Consensus Pros** (agreed by multiple models):
- [Pro 1]
- [Pro 2]

**Additional Pros**:
- [Pro 3] *(Sonnet insight)* [if applicable]
- [Pro 4] *(Opus concern)* [if applicable]

**Consensus Cons** (agreed by multiple models):
- [Con 1]
- [Con 2]

**Additional Cons**:
- [Con 3] *(Codex MCP: technical limitation)* [if applicable]

**Technical Considerations** *(Codex MCP)*: [if Codex MCP was used]
- [Detail 1]
- [Detail 2]

---

#### Alternative 2: [Name]
[Similar structure...]

---

### Model-Specific Emphases:

- **Sonnet emphasized**: [Key practical points]
- **Opus emphasized**: [Deep insights or long-term concerns]
- **Codex MCP emphasized**: [Technical implementation details]
```

Ask user: "Does this analysis look good? Should I proceed to create the ADR? (yes/no/modify)"

If "modify", ask what to adjust and potentially re-run specific model analyses.

---

## Phase 3: Generate ADR Document

### Step 3.1: Launch ADR Writer Agent

Use Task tool to launch the adr-writer agent:

```
Use Task tool:
- subagent_type: "adr-toolkit:adr-writer"
- model: "sonnet" (writer doesn't need heavy model)
- description: "Generate ADR document"
- prompt:
  """
  Generate an Architecture Decision Record with the following information:

  ADR Number: [determined number, e.g., 0001]
  Title: [infer from context or ask user]
  Models Used: [list of selected models]

  Context: [user's original context]

  Analysis Results:
  [Paste the synthesized analysis from Phase 2]

  Generate a complete ADR following the standard format with:
  - Title and metadata
  - Status (propose "proposed" for new ADRs)
  - Context section
  - Analysis Methodology (mention which models were used)
  - Alternatives Considered (with pros/cons attributed to models)
  - Decision section (ask user which alternative was chosen, or mark as "To be decided")
  - Consequences
  - Model Perspectives Summary

  Format: Markdown
  """
```

### Step 3.2: Review Generated ADR

Present the generated ADR to the user:

```markdown
## Generated ADR Preview

[Show the ADR content]

---

Filename: [ADR directory]/[number]-[slugified-title].md

Save this ADR? (yes/no/modify)
```

If "modify", ask what to change and use Edit tool to update.

### Step 3.3: Save ADR

If user approves:

```bash
# Save the ADR file
[Write the ADR content to the determined path]

# If docs/adr/README.md exists, update it with the new ADR
```

Confirm to user:
```
✓ ADR saved: [filepath]
✓ ADR-[number]: [title]
```

---

## Phase 4: Summary and Next Steps

### Step 4.1: Provide Summary

```markdown
## ADR Creation Complete!

**ADR**: [number] - [title]
**Location**: [filepath]
**Models Used**: [list]
**Alternatives Analyzed**: [count]

### What was created:
- Comprehensive architecture analysis from [N] AI models
- [X] alternatives evaluated with pros/cons
- Well-structured ADR document

### Next Steps:
1. Review the ADR: `cat [filepath]`
2. Update the status once decision is made
3. Link related ADRs if applicable
4. Share with team for feedback

### Model Insights:
- **Sonnet**: [brief summary if used]
- **Opus**: [brief summary if used]
- **Codex MCP**: [brief summary if used]
```

---

## Success Checklist

Before completing, verify:

- ✓ User selected which models to use
- ✓ Analysis context was gathered
- ✓ All selected models completed their analysis
- ✓ Results were synthesized and presented
- ✓ ADR was generated with proper format
- ✓ ADR was saved to appropriate location
- ✓ User was informed of completion

---

## Key Principles

1. **User Choice** - Let user select which models to use based on their needs
2. **Parallel Execution** - Run all model analyses in parallel for efficiency
3. **Multi-Perspective** - Leverage different model strengths for comprehensive analysis
4. **Transparent Attribution** - Show which models contributed which insights
5. **Flexible Scope** - Support analysis of specific files, features, or broad patterns
6. **Standard Format** - Follow ADR best practices and conventions
7. **Clear Communication** - Keep user informed at each phase

---

## Error Handling

### No Models Selected
- If user doesn't select any models, suggest using Sonnet as minimum

### Analysis Failures
- If a model fails, report it and continue with successful analyses
- Don't block ADR creation if at least one model succeeds

### No ADR Directory
- Offer to create standard location (docs/adr/)
- Support custom paths if user prefers

### File Conflicts
- If ADR number already exists, increment and try next number
- Never overwrite existing ADRs

---

## Notes

- Default to multi-model analysis for best results, but respect user's model selection
- Parallel execution is crucial for efficiency when using multiple models
- Each model brings unique value - don't assume one is always better
- ADR format should clearly attribute insights to specific models
- Support both creation from analysis and documentation of existing decisions
