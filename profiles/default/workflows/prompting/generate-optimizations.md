# Generate Optimizations

Workflow to generate project-specific prompt optimization suggestions.

## Purpose

Analyze project context and generate specific suggestions for optimizing command prompts based on:
- Detected tech stack
- Architecture patterns
- Codebase conventions
- Project-specific patterns from basepoints

## Inputs

- `HEADQUARTER` - Project headquarter.md content
- `PROJECT_PROFILE` - Project profile YAML content
- Command files to analyze

## Process

### Step 1: Extract Project Context

```bash
# Extract tech stack
TECH_STACK=$(echo "$PROJECT_PROFILE" | grep -E "language:|primary_language:" | cut -d: -f2 | tr -d ' ' | head -1)

# Extract architecture style
ARCHITECTURE=$(echo "$PROJECT_PROFILE" | grep -E "architecture_style:|architecture:" | cut -d: -f2 | tr -d ' ' | head -1)

# Extract key patterns from headquarter
KEY_PATTERNS=$(extract_patterns_from_headquarter "$HEADQUARTER")
```

### Step 2: Analyze Each Command

For each command file in `agent-os/commands/`:

```bash
# Check context placement
if ! grep -q "## Context\|### Project" "$CMD_FILE"; then
    SUGGEST_ADD_CONTEXT=true
fi

# Check project relevance
if ! grep -qi "$TECH_STACK\|$ARCHITECTURE" "$CMD_FILE"; then
    SUGGEST_ADD_PROJECT_INFO=true
fi

# Check pattern references
if ! grep -q "basepoints\|patterns" "$CMD_FILE"; then
    SUGGEST_ADD_PATTERNS=true
fi
```

### Step 3: Generate Suggestions

For each command needing optimization, generate a suggestion:

```markdown
## Optimization: [Command Name]

**Issue**: [What's suboptimal]
- Missing project context
- No pattern references
- Generic instructions

**Impact**: [Why it matters]
- AI won't use project-specific patterns
- May create code inconsistent with codebase
- Missing validation commands

**Suggestion**: [Specific change]

**Before**:
[Current text example]

**After**:
[Improved text with project context]
```

### Step 4: Format Suggestions

Organize suggestions by command and priority:

```markdown
# Prompt Optimization Suggestions

**Generated**: [timestamp]
**Project**: [project name]
**Tech Stack**: [detected]
**Architecture**: [detected]

## High Priority

### [Command Name]
[Optimization suggestion]

## Medium Priority

### [Command Name]
[Optimization suggestion]

## Low Priority

### [Command Name]
[Optimization suggestion]
```

## Output

Saves suggestions to: `agent-os/output/deploy-agents/cache/prompt-optimizations.md`

## Usage

Called by Phase 11 of `/deploy-agents` during first installation to suggest project-specific prompt improvements.
