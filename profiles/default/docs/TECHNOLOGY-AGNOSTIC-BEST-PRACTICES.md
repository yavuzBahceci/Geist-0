# Technology-Agnostic Best Practices

This document outlines best practices for maintaining technology-agnostic templates in `profiles/default/`. Templates should work with any technology stack and be specialized during deployment.

---

## Table of Contents

- [Core Principle](#core-principle)
- [Acceptable vs. Unacceptable Patterns](#acceptable-vs-unacceptable-patterns)
- [Using Placeholders](#using-placeholders)
- [Functional Detection Code](#functional-detection-code)
- [Examples in Documentation](#examples-in-documentation)
- [Validation](#validation)

---

## Core Principle

**`profiles/default/` templates must be abstract and technology-agnostic.**

Technology-specific patterns should only exist in specialized agent-os instances (created during deployment), not in the base templates.

---

## Acceptable vs. Unacceptable Patterns

### ✅ Acceptable: Minimal Examples for Illustration

**In Documentation:**
```markdown
This will detect build/test/lint commands from project configuration files 
(e.g., package.json, Makefile, requirements.txt, etc.).
```

**In Comments:**
```bash
# Example: Product tech-stack suggests "use TypeScript" patterns but 
# abstract command uses generic patterns
```

**Why Acceptable:** These are minimal examples used to illustrate concepts, not functional code.

### ✅ Acceptable: Functional Detection Code (with Documentation)

**In Specialization Logic:**
```bash
# NOTE: This function contains technology-specific detection patterns
# These are common examples used for detecting and specializing validation commands during deployment.
# This detection logic is functional code required for the specialization process to work.
detect_validation_commands() {
    # Node.js / JavaScript / TypeScript - Example detection pattern
    if [ -f "package.json" ]; then
        BUILD_CMD="npm run build"
    fi
    # ... other patterns ...
}
```

**Why Acceptable:** This is functional code required for specialization to work. It's documented with clarifying comments explaining its purpose.

### ❌ Unacceptable: Hardcoded Technology Assumptions

**Bad:**
```bash
# Check if this is a React project
if [ -f "src/App.jsx" ]; then
    echo "This is a React application"
fi
```

**Better:**
```bash
# Detect project type from structure
# (Use abstract patterns, not technology-specific file checks)
```

**Why Unacceptable:** Assumes specific technology without abstraction or documentation.

---

## Using Placeholders

### Project-Specific Commands

**❌ Bad:**
```bash
npm run build
npm test
npm run lint
```

**✅ Good:**
```bash
{{PROJECT_BUILD_COMMAND}}
{{PROJECT_TEST_COMMAND}}
{{PROJECT_LINT_COMMAND}}
```

### Path References

**❌ Bad:**
```bash
agent-os/basepoints
agent-os/product
```

**✅ Good (when values vary):**
```bash
{{BASEPOINTS_PATH}}
{{PRODUCT_PATH}}
```

**Note:** Hardcoded paths like `agent-os/basepoints` are acceptable when they're the standard Agent OS structure (not project-specific).

---

## Functional Detection Code

### When Detection Code is Necessary

Some templates require technology detection as part of their functionality:

- **Specialization commands**: Must detect tech stack to specialize validation commands
- **Configuration generation**: Need to identify project type to generate appropriate configs

### How to Handle Functional Detection Code

1. **Document Clearly**: Add comments explaining this is functional code required for specialization
2. **Explain Purpose**: Clarify that examples are illustrative patterns, not hardcoded assumptions
3. **Use Comments**: Add "Example detection pattern" comments to each technology section
4. **Keep Abstract**: Use abstract descriptions where possible; use concrete examples only when necessary

**Example - Well-Documented Detection Code:**
```bash
# Detect tech stack from project files (fallback or supplement)
# NOTE: This function contains technology-specific detection patterns (Node.js, Rust, Go, Python, etc.)
# These are common examples used for detecting and specializing validation commands during deployment.
# This detection logic is functional code required for the specialization process to work.
# The patterns shown here are illustrative examples that get specialized based on the actual project's tech stack.
detect_validation_commands() {
    # Node.js / JavaScript / TypeScript - Example detection pattern
    if [ -f "package.json" ]; then
        echo "   Detected: Node.js project"
        # ... detection logic ...
    fi
    # ... other patterns ...
}
```

---

## Examples in Documentation

### Guidelines for Documentation Examples

1. **Minimal and Necessary**: Only include examples that help illustrate the concept
2. **Abstract When Possible**: Use generic descriptions instead of specific technologies
3. **Use "e.g." for Examples**: Clarify that examples are illustrative, not exhaustive

**✅ Good Documentation Examples:**
```markdown
Extract build/test/lint commands from project configuration files 
(e.g., package.json, Makefile, requirements.txt, etc.).
```

```markdown
Detects language, framework, and database from config files.
```

**❌ Bad Documentation Examples:**
```markdown
Extract commands from package.json (Node.js projects only).
```

```markdown
This works for TypeScript React applications.
```

---

## Validation

### Running Technology-Agnostic Validation

Use the validation workflow to check for technology leaks:

```bash
{{workflows/validation/validate-technology-agnostic}}
```

This will:
- Scan all files in `profiles/default/`
- Detect technology-specific assumptions
- Report findings (distinguishing functional code vs. examples)

### What Validation Checks For

- Technology-specific framework names (React, Vue, Angular, etc.)
- Package manager references (npm, yarn, pip, etc.)
- Language-specific file patterns (.jsx, .tsx, etc.)
- Technology-specific commands (npm run build, cargo test, etc.)

### Validation Results

Validation distinguishes between:

1. **Functional Detection Code**: Code required for specialization (acceptable with documentation)
2. **Documentation Examples**: Minimal examples in docs/comments (acceptable)
3. **Hardcoded Assumptions**: Technology-specific logic without abstraction (unacceptable)

---

## Examples: Abstract vs. Concrete Patterns

### Pattern 1: Command Execution

**❌ Concrete:**
```bash
npm run build
npm test
```

**✅ Abstract (Placeholder):**
```bash
{{PROJECT_BUILD_COMMAND}}
{{PROJECT_TEST_COMMAND}}
```

**✅ Abstract (Detection):**
```bash
# Detect build command from project configuration
BUILD_CMD=$(detect_build_command)
```

### Pattern 2: File Detection

**❌ Concrete:**
```bash
if [ -f "package.json" ]; then
    echo "Node.js project"
fi
```

**✅ Abstract (Documentation):**
```markdown
Detects project type from configuration files (e.g., package.json, Cargo.toml, go.mod, etc.).
```

**✅ Abstract (Functional Detection with Documentation):**
```bash
# NOTE: This detection logic contains technology-specific patterns
# These are illustrative examples for detecting project type during specialization.
# Node.js / JavaScript / TypeScript - Example detection pattern
if [ -f "package.json" ]; then
    # ... detection logic ...
fi
```

### Pattern 3: Path References

**❌ Concrete (Project-Specific):**
```bash
src/components/Button.tsx
```

**✅ Abstract:**
```bash
{{PROJECT_SOURCE_PATH}}/{{COMPONENT_NAME}}
```

**✅ Abstract (Standard Structure):**
```bash
agent-os/basepoints  # Standard Agent OS structure (acceptable)
```

---

## Summary

- **Use placeholders** for project-specific values (`{{PROJECT_BUILD_COMMAND}}`)
- **Use abstract descriptions** in documentation instead of concrete technology names
- **Document functional detection code** with clarifying comments explaining its purpose
- **Keep examples minimal** and only when necessary for illustration
- **Run validation** regularly to catch technology leaks
- **Distinguish between**: Functional code (acceptable with docs), Examples (acceptable when minimal), Hardcoded assumptions (unacceptable)

---

## Related Documentation

- [REFACTORING-GUIDELINES.md](./REFACTORING-GUIDELINES.md) - Guidelines for refactoring templates
- [PATH-REFERENCE-GUIDE.md](./PATH-REFERENCE-GUIDE.md) - Reference guide for path placeholders
