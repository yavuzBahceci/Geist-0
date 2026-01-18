# Refactoring Guidelines for profiles/default Templates

This document provides guidelines for maintaining and refactoring command and workflow files in `profiles/default/` to ensure they remain concise, maintainable, and technology-agnostic.

---

## Table of Contents

- [Core Principles](#core-principles)
- [Using Workflow References](#using-workflow-references)
- [Creating Common Workflows](#creating-common-workflows)
- [File Length Guidelines](#file-length-guidelines)
- [Before/After Examples](#beforeafter-examples)
- [Technology-Agnostic Patterns](#technology-agnostic-patterns)

---

## Core Principles

1. **Single Source of Truth**: Extract repeated patterns into reusable workflows
2. **Workflow References First**: Always prefer `{{workflows/...}}` references over inline code
3. **Conciseness**: Keep command files focused and concise (target: under 500 lines, ideally under 200)
4. **Technology-Agnostic**: Use abstract patterns and placeholders, not concrete technology examples

---

## Using Workflow References

### When to Use `{{workflows/...}}` References

**Always use workflow references when:**
- The same pattern appears in 3+ command files
- The pattern is longer than ~15 lines
- The pattern combines multiple workflow steps
- You need to reference existing workflow functionality

**Example - Bad (Inline Code):**
```bash
# Check if basepoints exist
if [ -d "agent-os/basepoints" ] && [ -f "agent-os/basepoints/headquarter.md" ]; then
    SPEC_PATH="agent-os/specs/[current-spec]"
    {{workflows/basepoints/extract-basepoints-knowledge-automatic}}
    {{workflows/scope-detection/detect-abstraction-layer}}
    {{workflows/scope-detection/detect-scope-semantic-analysis}}
    {{workflows/scope-detection/detect-scope-keyword-matching}}
    # Load extracted knowledge...
    # Load detected layer...
fi
```

**Example - Good (Workflow Reference):**
```bash
{{workflows/common/extract-basepoints-with-scope-detection}}
```

### How Workflow References Work

Workflow references like `{{workflows/common/extract-basepoints-with-scope-detection}}` are expanded inline during the compilation process when commands are specialized for a project. This means:

- **Source templates** remain concise and maintainable
- **Compiled commands** contain the full expanded content (expected behavior)
- **Single source of truth** - update the workflow, all commands benefit

---

## Creating Common Workflows

### When to Create New Common Workflows

Create a new workflow in `workflows/common/` when:

1. **Pattern Repetition**: The same sequence appears in 3+ command files
2. **Logical Grouping**: Multiple workflow references are commonly used together
3. **Abstraction Opportunity**: A complex pattern can be simplified with a wrapper

### Workflow Creation Checklist

- [ ] Pattern appears in 3+ files
- [ ] Workflow follows standard structure (Core Responsibilities + Workflow steps)
- [ ] Workflow uses `{{workflows/...}}` references for dependencies
- [ ] Workflow is abstract and technology-agnostic
- [ ] Workflow includes usage notes and examples

### Example: Common Workflow Structure

```markdown
# Workflow Name

## Core Responsibilities

1. **Responsibility 1**: Description
2. **Responsibility 2**: Description

## Workflow

### Step 1: Step Name

[Instructions and bash code]

## Usage Notes

[How to use this workflow]

## Important Constraints

[Any important constraints or requirements]
```

---

## File Length Guidelines

### Target File Lengths

- **Command Files**: Under 500 lines, ideally under 200 lines
- **Workflow Files**: No strict limit, but keep focused and cohesive
- **Common Workflows**: Typically 50-150 lines

### When Files Get Too Long

If a command file exceeds 500 lines:

1. **Extract Repeated Patterns**: Look for duplicated blocks (15+ lines)
2. **Use Common Workflows**: Replace inline code with `{{workflows/common/...}}` references
3. **Simplify Explanations**: Keep descriptions concise; details belong in workflows
4. **Break into Sub-Commands**: If functionality is distinct enough, consider splitting

### Before/After Example: File Length Reduction

**Before** (92 lines with inline basepoints extraction):
```bash
## Step 1: Extract Basepoints Knowledge

[... 22 lines of inline extraction code ...]

If basepoints exist, the extracted knowledge will be used to:
- Inform clarifying questions...
```

**After** (72 lines with workflow reference):
```bash
## Step 1: Extract Basepoints Knowledge

```bash
{{workflows/common/extract-basepoints-with-scope-detection}}
```

If basepoints exist, the extracted knowledge (`$EXTRACTED_KNOWLEDGE` and `$DETECTED_LAYER`) will be used to:
- Inform clarifying questions...
```

**Result**: 20 lines removed (22% reduction), single source of truth established

---

## Before/After Examples

### Example 1: Basepoints Extraction Pattern

**❌ Before - Inline Pattern (Repeated in 5 Files):**
```bash
# Check if basepoints exist
if [ -d "agent-os/basepoints" ] && [ -f "agent-os/basepoints/headquarter.md" ]; then
    SPEC_PATH="agent-os/specs/[current-spec]"
    {{workflows/basepoints/extract-basepoints-knowledge-automatic}}
    {{workflows/scope-detection/detect-abstraction-layer}}
    {{workflows/scope-detection/detect-scope-semantic-analysis}}
    {{workflows/scope-detection/detect-scope-keyword-matching}}
    
    if [ -f "$SPEC_PATH/implementation/cache/basepoints-knowledge.md" ]; then
        EXTRACTED_KNOWLEDGE=$(cat "$SPEC_PATH/implementation/cache/basepoints-knowledge.md")
    fi
    
    if [ -f "$SPEC_PATH/implementation/cache/detected-layer.txt" ]; then
        DETECTED_LAYER=$(cat "$SPEC_PATH/implementation/cache/detected-layer.txt")
    fi
fi
```

**✅ After - Workflow Reference:**
```bash
{{workflows/common/extract-basepoints-with-scope-detection}}
```

**Benefits:**
- Reduced from 22 lines to 1 line per file
- Single source of truth in `workflows/common/extract-basepoints-with-scope-detection.md`
- Updates propagate automatically to all commands
- Clearer intent and easier maintenance

### Example 2: Spec Path Determination

**❌ Before - Repeated Pattern:**
```bash
# Determine spec path
SPEC_PATH="agent-os/specs/[current-spec]"

# Create cache directories
mkdir -p "$SPEC_PATH/implementation/cache"
```

**✅ After - Use Common Workflow (if pattern repeats):**
```bash
{{workflows/common/determine-spec-context}}
```

---

## Technology-Agnostic Patterns

### Using Placeholders vs. Concrete Examples

**❌ Bad - Concrete Technology Names:**
```bash
# Check for Node.js project
if [ -f "package.json" ]; then
    BUILD_CMD="npm run build"
fi
```

**✅ Good - Abstract Detection (when functional code is required):**
```bash
# NOTE: This function contains technology-specific detection patterns
# These are common examples used for detecting validation commands during specialization.
# This detection logic is functional code required for the specialization process to work.
detect_validation_commands() {
    # Node.js / JavaScript / TypeScript - Example detection pattern
    if [ -f "package.json" ]; then
        BUILD_CMD="npm run build"
    fi
    # ... other patterns ...
}
```

**✅ Good - Abstract Description (in documentation):**
```markdown
This will detect build/test/lint commands from project configuration files (e.g., package.json, Makefile, etc.).
```

### Key Principles

1. **Functional Code**: When detection logic is required for functionality, keep it but document with clarifying comments
2. **Documentation Examples**: Use abstract descriptions or minimal concrete examples for illustration
3. **Placeholders**: Use `{{PROJECT_BUILD_COMMAND}}` syntax when values are replaced during specialization
4. **Comments**: Clarify when examples are illustrative vs. functional requirements

---

## Summary

- **Prefer workflow references** over inline code for repeated patterns
- **Create common workflows** when patterns appear in 3+ files
- **Keep files concise** (under 500 lines, ideally under 200)
- **Use abstract patterns** and document when concrete examples are necessary
- **Maintain single source of truth** for all repeated patterns

For technology-agnostic guidelines, see [TECHNOLOGY-AGNOSTIC-BEST-PRACTICES.md](./TECHNOLOGY-AGNOSTIC-BEST-PRACTICES.md).
