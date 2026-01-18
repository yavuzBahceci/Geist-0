# Contributing to Geist

Thank you for your interest in contributing to Geist! This guide will help you understand how to add new commands, workflows, and improve the project.

---

## Table of Contents

- [Getting Started](#getting-started)
- [How to Add New Commands](#how-to-add-new-commands)
- [How to Add New Workflows](#how-to-add-new-workflows)
- [How to Add New Standards](#how-to-add-new-standards)
- [Testing Requirements](#testing-requirements)
- [Code Style](#code-style)
- [PR Process](#pr-process)

---

## Getting Started

1. **Fork the repository** on GitHub
2. **Clone your fork**:
   ```bash
   git clone https://github.com/YOUR-USERNAME/Geist-v1.git
   cd Geist-v1
   ```
3. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```

---

## How to Add New Commands

### Directory Structure

Commands live in `profiles/default/commands/`. Each command has this structure:

```
profiles/default/commands/[command-name]/
├── single-agent/                    # Single-agent implementation
│   ├── [command-name].md            # Main entry point
│   ├── 1-[first-phase].md           # Phase 1
│   ├── 2-[second-phase].md          # Phase 2
│   └── ...                          # Additional phases
└── multi-agent/                     # (Optional) Multi-agent implementation
    └── [command-name].md
```

### Required Files

#### 1. Main Command File (`[command-name].md`)

```markdown
# Command: [command-name]

## Purpose

[One sentence describing what this command does]

## Prerequisites

- [List required files/state]
- [e.g., "Specialized Agent OS"]
- [e.g., "`agent-os/product/tech-stack.md` exists"]

## Outputs

- [List files created]
- [e.g., "`agent-os/specs/[name]/spec.md`"]

## Phases

{{PHASE 1: @agent-os/commands/[command-name]/1-[phase-name].md}}

{{PHASE 2: @agent-os/commands/[command-name]/2-[phase-name].md}}

## Standards

{{standards/global/*}}
{{standards/[relevant-category]/*}}
```

#### 2. Phase Files (`N-[phase-name].md`)

```markdown
# Phase N: [Phase Name]

## Purpose

[What this phase accomplishes]

## Steps

### Step 1: [Step Name]

[Instructions or workflow reference]

{{workflows/[category]/[workflow-name]}}

### Step 2: [Step Name]

[More instructions]

## Outputs

- [What this phase produces]

## Next Phase

Proceed to Phase N+1: [Next Phase Name]
```

### Phase Numbering Rules

1. **Start at 1**, increment sequentially
2. **No gaps allowed** (1, 2, 3... not 1, 3, 5)
3. **Last phase** should navigate to next command or completion
4. **Use descriptive names** (e.g., `1-initialize-spec.md`, not `1-phase1.md`)

### Example

See `commands/shape-spec/single-agent/` for a well-structured example:
- `shape-spec.md` - Main entry point
- `1-initialize-spec.md` - Creates folder structure
- `2-shape-spec.md` - Gathers requirements

---

## How to Add New Workflows

### Directory Structure

Workflows live in `profiles/default/workflows/`, organized by category:

```
profiles/default/workflows/
├── basepoints/              # Knowledge extraction
├── codebase-analysis/       # Codebase analysis and basepoint generation
├── common/                  # Shared utilities
├── detection/               # Auto-detection
├── human-review/            # Human review workflows
├── implementation/          # Task implementation
├── learning/                # Session learning
├── planning/                # Product planning
├── prompting/               # Prompt optimization
├── research/                # Web research
├── scope-detection/         # Scope and layer detection
├── specification/           # Spec writing
└── validation/              # Validation utilities
```

### Workflow Template

```markdown
# Workflow: [Workflow Name]

## Purpose

[One sentence describing what this workflow does]

## Prerequisites

- [Required state/files]

## Inputs

- `$VARIABLE_NAME` - [Description]
- [Other inputs]

## Outputs

- [What this workflow produces]
- [Files created/modified]

---

## Workflow

### Step 1: [Step Name]

```bash
echo "📖 [Action description]..."

# Implementation
[code]

echo "✅ [Completion message]"
```

### Step 2: [Step Name]

```bash
# More implementation
```

---

## Important Constraints

- [Any rules or limitations]
- [Technology-agnostic requirements]
```

### Registration

After creating a workflow:

1. **Reference from a command**:
   ```markdown
   {{workflows/[category]/[workflow-name]}}
   ```

2. **Document in WORKFLOW-MAP.md**:
   - Add to appropriate category
   - Include in command flow diagrams if relevant

3. **Update COMMAND-FLOWS.md** if the workflow is part of a command phase

---

## How to Add New Standards

### Directory Structure

Standards live in `profiles/default/standards/`:

```
profiles/default/standards/
├── global/                  # Cross-cutting standards
├── documentation/           # Documentation standards
├── process/                 # Process standards
├── quality/                 # Quality standards
└── testing/                 # Testing standards
```

### Standards Template

```markdown
# Standard: [Standard Name]

## Purpose

[What this standard ensures]

## Rules

### Rule 1: [Rule Name]

[Description of the rule]

**Do**:
- [Good example]

**Don't**:
- [Bad example]

### Rule 2: [Rule Name]

[More rules]

## Validation

[How to check compliance with this standard]
```

---

## Testing Requirements

Before submitting a PR, verify:

### 1. Workflow References Resolve

```bash
# Search for workflow references
grep -r "{{workflows/" profiles/default/commands/

# Verify each referenced workflow exists
ls profiles/default/workflows/[category]/[name].md
```

### 2. Phase Numbering is Sequential

```bash
# List phase files for a command
ls -1 profiles/default/commands/[command]/single-agent/*.md | sort
```

Should show: `1-*.md`, `2-*.md`, `3-*.md`, etc. with no gaps.

### 3. Standards Compliance

- All files use kebab-case naming
- All code blocks have language tags
- No placeholder text (except intentional `{{...}}`)
- Proper markdown structure (single H1, proper hierarchy)

### 4. Test with Sample Project

If possible, test your changes with a sample project:

```bash
# Install to test project
cd /path/to/test-project
~/geist/scripts/project-install.sh --profile default

# Run relevant commands
/adapt-to-product
/create-basepoints
# etc.
```

---

## Code Style

### File Naming

- **Directories**: `kebab-case` (e.g., `single-agent/`, `codebase-analysis/`)
- **Files**: `kebab-case.md` (e.g., `shape-spec.md`, `detect-tech-stack.md`)
- **Exceptions**: `README.md`, `MANIFEST.md`, `CONTRIBUTING.md` (uppercase)

### Code Blocks

Always include language tags:

```markdown
```bash
echo "Hello"
```

```yaml
key: value
```

```json
{"key": "value"}
```
```

### Markdown Structure

- **One H1 heading** per file (the title)
- **Proper hierarchy**: H1 > H2 > H3 (don't skip levels)
- **Horizontal rules** (`---`) to separate major sections

### Placeholders

Use `{{...}}` syntax for:
- Workflow references: `{{workflows/category/name}}`
- Standards references: `{{standards/category/*}}`
- Project-specific values: `{{PROJECT_BUILD_COMMAND}}`

---

## PR Process

### 1. Create Your PR

```bash
# Push your branch
git push origin feature/your-feature-name
```

Then create a PR on GitHub.

### 2. PR Description Template

```markdown
## Summary

[Brief description of changes]

## Changes

- [List of specific changes]
- [Files added/modified]

## Testing

- [ ] Workflow references verified
- [ ] Phase numbering checked
- [ ] Standards compliance verified
- [ ] Tested with sample project (if applicable)

## Documentation

- [ ] COMMAND-FLOWS.md updated (if adding command)
- [ ] WORKFLOW-MAP.md updated (if adding workflow)
- [ ] README updated (if significant change)
```

### 3. Review Process

1. Maintainers will review your PR
2. Address any feedback
3. Once approved, PR will be merged

### 4. After Merge

- Delete your feature branch
- Pull latest changes to your fork

---

## Questions?

- Check existing documentation in `profiles/default/docs/`
- Open an issue for discussion
- Reach out to maintainers

Thank you for contributing!

---

*Last Updated: 2026-01-18*
