# Path Reference Guide for profiles/default Templates

## Overview

Files in `profiles/default/` are **templates** that get installed into `geist/` when a project is set up. This guide explains how path references work and how to ensure they resolve correctly in both template and installed contexts.

---

## Reference Types

### 1. Template References (Compile-Time)

These are resolved during compilation when templates are installed:

| Pattern | Purpose | Resolution | Example |
|---------|---------|------------|---------|
| `{{workflows/...}}` | Inject workflow content | Replaced with workflow content during compilation | `{{workflows/prompting/construct-prompt}}` |
| `{{standards/...}}` | Inject standards content | Replaced with standards content during compilation | `{{standards/global/conventions}}` |
| `{{PHASE X: @geist/commands/...}}` | Embed phase file | Replaced with phase file content during compilation | `{{PHASE 1: @geist/commands/create-tasks/1-initialize.md}}` |

**✅ Correct**: These work in both contexts (template and installed)

---

### 2. Runtime Path References

These are file paths that need to resolve at runtime when commands/workflows execute:

| Pattern | Purpose | When Used | Resolution |
|---------|---------|-----------|------------|
| `geist/...` | Reference installed files | Always use installed paths | `geist/basepoints/headquarter.md` |
| `@geist/...` | Reference annotation | Documentation/inline references | `@geist/workflows/validation/orchestrate-validation.md` |
| Context-aware paths | Dynamic path resolution | When file location differs by context | Use `$WORKFLOWS_BASE` pattern |

**✅ Correct**: Always use `geist/...` paths in templates (they will exist in installed context)

---

### 3. Context Detection (Runtime)

Used to determine if running in template or installed context:

| Pattern | Purpose | When to Use |
|---------|---------|-------------|
| `if [ -d "geist/workflows" ]` | Check if installed | Before referencing workflows/commands |
| `if [ -d "profiles/default" ]` | Check if template | When validating template vs installed |
| `$WORKFLOWS_BASE` variable | Dynamic base path | When sourcing workflows that could be in either location |

**✅ Correct**: Use for runtime path resolution

---

## Path Mapping During Installation

```
BEFORE INSTALLATION (Template)          AFTER INSTALLATION (Installed)
─────────────────────────────────      ──────────────────────────────────

profiles/default/                       geist/
├── commands/                           ├── commands/
│   └── shape-spec/                     │   └── shape-spec/
│       └── single-agent/               │       └── 2-shape-spec.md
│           └── 2-shape-spec.md         │
├── workflows/                          ├── workflows/
│   └── prompting/                      │   └── prompting/
│       └── construct-prompt.md         │       └── construct-prompt.md
└── standards/                          └── standards/
    └── global/                             └── global/
        └── conventions.md                    └── conventions.md
```

**Key Points**:
- Templates are in `profiles/default/` (only exists in Geist repository)
- Installed files are in `geist/` (exists in project)
- References in templates should point to `geist/...` paths
- Context detection determines runtime location

---

## Reference Resolution Logic

### During Compilation (`project-install.sh`)

```bash
# Template file
profiles/default/commands/shape-spec/single-agent/2-shape-spec.md
  → Contains: {{workflows/prompting/construct-prompt}}
  → Resolved: Workflow content embedded
  → Output: geist/commands/shape-spec/2-shape-spec.md (with workflow content)
```

### During Runtime (Command Execution)

```bash
# Installed file
geist/commands/shape-spec/2-shape-spec.md
  → Contains: geist/basepoints/headquarter.md
  → Resolved: File exists in project's geist/ directory
  → Works: ✅
```

### Context-Aware Resolution

```bash
# Template file
profiles/default/workflows/validation/validation-registry.md
  → Contains: source "$WORKFLOWS_BASE/validation/..."
  → Runtime check:
     if [ -d "geist/workflows" ]; then
         WORKFLOWS_BASE="geist/workflows"
     else
         WORKFLOWS_BASE="profiles/default/workflows"
     fi
  → Resolved: Correct path based on runtime context
```

---

## Common Patterns

### ✅ Correct Patterns

#### 1. Always Use `geist/` for Installed Paths

```bash
# ✅ Correct - References installed path
HEADQUARTER=$(cat geist/basepoints/headquarter.md 2>/dev/null || echo "")
PROJECT_PROFILE=$(cat geist/config/project-profile.yml 2>/dev/null || echo "")
```

#### 2. Use Context Detection for Workflows/Commands

```bash
# ✅ Correct - Context-aware workflow sourcing
if [ -d "geist/workflows" ]; then
    WORKFLOWS_BASE="geist/workflows"
else
    WORKFLOWS_BASE="profiles/default/workflows"
fi

source "$WORKFLOWS_BASE/human-review/detect-trade-offs.md"
```

#### 3. Use Template References for Compile-Time Injection

```bash
# ✅ Correct - Compile-time reference
{{workflows/prompting/save-handoff}}
{{standards/global/conventions}}
```

#### 4. Context Detection for Validation

```bash
# ✅ Correct - Context detection for validation
if [ -d "profiles/default" ] && [ "$(pwd)" = *"/profiles/default"* ]; then
    VALIDATION_CONTEXT="profiles/default"
    SCAN_PATH="profiles/default"
else
    VALIDATION_CONTEXT="installed-geist"
    SCAN_PATH="geist"
fi
```

### ❌ Incorrect Patterns

#### 1. Direct `profiles/default/` Paths in Runtime Code

```bash
# ❌ Wrong - Will fail in installed context
source "profiles/default/workflows/validation/validate.md"

# ✅ Correct - Context-aware
if [ -d "geist/workflows" ]; then
    WORKFLOWS_BASE="geist/workflows"
else
    WORKFLOWS_BASE="profiles/default/workflows"
fi
source "$WORKFLOWS_BASE/validation/validate.md"
```

#### 2. Mixed Context Paths

```bash
# ❌ Wrong - Mixed paths
find "profiles/default/commands" -name "*.md"
find "geist/workflows" -name "*.md"

# ✅ Correct - Use consistent context detection
if [ -d "geist/commands" ]; then
    COMMANDS_DIR="geist/commands"
else
    COMMANDS_DIR="profiles/default/commands"
fi
find "$COMMANDS_DIR" -name "*.md"
```

---

## Reference Handling During Commands

### Command Flow Example: `/shape-spec`

```
┌─────────────────────────────────────────────────────────────────┐
│                     RUNTIME REFERENCE FLOW                       │
└─────────────────────────────────────────────────────────────────┘

1. Command Starts
   ├─ Command file: geist/commands/shape-spec/2-shape-spec.md
   └─ Already compiled (workflows/standards injected)

2. Context Detection (if needed)
   ├─ Check: if [ -d "geist/workflows" ]
   ├─ Set: WORKFLOWS_BASE="geist/workflows"
   └─ Use: source "$WORKFLOWS_BASE/..."

3. Path References
   ├─ geist/basepoints/headquarter.md ✅ (installed path)
   ├─ geist/config/project-profile.yml ✅ (installed path)
   └─ geist/output/handoff/current.md ✅ (installed path)

4. Workflow Execution
   ├─ Source workflows from: $WORKFLOWS_BASE/... ✅ (context-aware)
   └─ Workflows use: geist/... paths ✅ (installed paths)

5. Command Completion
   └─ Save handoff to: geist/output/handoff/current.md ✅
```

### Workflow Reference Example: `review-trade-offs.md`

```bash
# Runtime resolution flow:

1. Workflow starts
   ├─ Location: geist/workflows/human-review/review-trade-offs.md
   └─ (or: profiles/default/workflows/human-review/review-trade-offs.md during template testing)

2. Context detection
   ├─ Check: [ -d "geist/workflows" ]
   ├─ Result: WORKFLOWS_BASE="geist/workflows"
   └─ (or: "profiles/default/workflows" if in template)

3. Source sub-workflows
   ├─ source "$WORKFLOWS_BASE/human-review/detect-trade-offs.md"
   ├─ Resolves to: geist/workflows/human-review/detect-trade-offs.md ✅
   └─ (or: profiles/default/workflows/... during template testing ✅)

4. Sub-workflow uses paths
   ├─ geist/basepoints/... ✅ (installed paths)
   └─ geist/output/... ✅ (installed paths)
```

---

## Special Cases

### 1. Test Files (`detection-tests.md`)

Test files explicitly test templates, so they use `profiles/default/` paths:

```bash
# ✅ Correct - Test file explicitly tests templates
source profiles/default/workflows/detection/detect-project-profile.md
```

### 2. Documentation References

Documentation can reference both locations:

```markdown
**File**: `geist/workflows/prompting/construct-prompt.md` (when installed)  
**Template**: `profiles/default/workflows/prompting/construct-prompt.md`
```

### 3. Cleanup Commands

Commands that clean up installed files should detect and reference `geist/`:

```bash
# ✅ Correct - Cleanup command references installed paths
FILES_WITH_PROFILES_DEFAULT=$(find geist/commands -name "*.md" ...)
```

---

## Verification Checklist

Before committing changes to `profiles/default/`, verify:

- [ ] No direct `profiles/default/` paths in runtime code (only in context detection)
- [ ] All runtime paths use `geist/...` 
- [ ] Workflow sourcing uses context detection (`$WORKFLOWS_BASE`)
- [ ] Template references use `{{workflows/...}}` or `{{standards/...}}`
- [ ] Test files explicitly testing templates can use `profiles/default/`
- [ ] Documentation references both template and installed locations where relevant

---

## Quick Reference Table

| Reference Type | Template | Installed | Pattern |
|---------------|----------|-----------|---------|
| Compile-time injection | ✅ | ✅ | `{{workflows/...}}` |
| Installed file paths | ❌ | ✅ | `geist/...` |
| Context-aware workflows | ✅ | ✅ | `$WORKFLOWS_BASE/...` |
| Context detection | ✅ | ✅ | `if [ -d "geist/workflows" ]` |
| Test files | ✅ | ❌ | `profiles/default/...` |
| Documentation | ✅ | ✅ | Both locations |

---

## Reference Audit Results

### ✅ Verified Correct References

#### 1. `/deploy-agents` Phases
**Status**: ✅ **Correct** - These run during installation and need to read templates

```bash
# profiles/default/commands/deploy-agents/single-agent/4-merge-knowledge-and-resolve-conflicts.md
# Reading templates to specialize them (runs BEFORE installation)
if [ -f "profiles/default/commands/$cmd/single-agent/$cmd.md" ]; then
    ABSTRACT_COMMANDS="${ABSTRACT_COMMANDS}\n$(cat profiles/default/commands/$cmd/single-agent/$cmd.md)"
fi
```

**Why OK**: These phases run during `/deploy-agents` before templates are installed. They need to read from `profiles/default/` to get the abstract templates for specialization.

#### 2. Validation Workflows
**Status**: ✅ **Correct** - Validation workflows check both template and installed contexts

```bash
# profiles/default/workflows/validation/validate-technology-agnostic.md
# Checking template files during validation (explicit template validation)
TEMPLATE_COMMANDS=$(find profiles/default/commands -name "*.md" -type f ...)
```

**Why OK**: These workflows explicitly validate templates, so referencing `profiles/default/` is intentional.

#### 3. Context Detection
**Status**: ✅ **Correct** - Context detection checks both locations

```bash
# All validation workflows
if [ -d "profiles/default" ] && [ "$(pwd)" = *"/profiles/default"* ]; then
    VALIDATION_CONTEXT="profiles/default"
    SCAN_PATH="profiles/default"
else
    VALIDATION_CONTEXT="installed-geist"
    SCAN_PATH="geist"
fi
```

**Why OK**: This is context detection logic that determines which path to use.

#### 4. Test Files
**Status**: ✅ **Correct** - Test files explicitly test templates

```bash
# profiles/default/workflows/validation/detection-tests.md
# Explicitly testing template workflows
source profiles/default/workflows/detection/detect-project-profile.md
```

**Why OK**: Test files explicitly test template functionality.

#### 5. Fallback Checks
**Status**: ✅ **Correct** - Fallback to template when installed version missing

```bash
# profiles/default/workflows/validation/validate-placeholder-cleaning.md
# Checking both installed and template as fallback
if [ ! -f "$SCAN_PATH/workflows/$WORKFLOW_PATH.md" ] && [ ! -f "profiles/default/workflows/$WORKFLOW_PATH.md" ]; then
    # Report error
fi
```

**Why OK**: This is a validation check that uses both paths as fallback verification.

---

## Reference Flow Diagrams

### Installation Flow: Template → Installed

```
┌─────────────────────────────────────────────────────────────────┐
│                    INSTALLATION PROCESS                          │
└─────────────────────────────────────────────────────────────────┘

1. Template Files (profiles/default/)
   ├─ Contains: Abstract templates with {{workflows/...}} references
   ├─ Location: Geist repository only
   └─ References: Use geist/ paths (for runtime) or {{...}} (for compile-time)

2. project-install.sh Execution
   ├─ Reads templates from: profiles/default/
   ├─ Resolves: {{workflows/...}} → workflow content
   ├─ Resolves: {{standards/...}} → standards content
   └─ Outputs: Compiled files to geist/

3. Installed Files (geist/)
   ├─ Contains: Specialized, compiled commands with embedded content
   ├─ Location: Project directory
   └─ References: All use geist/ paths (runtime resolution)
```

### Runtime Execution Flow: Command → Workflows

```
┌─────────────────────────────────────────────────────────────────┐
│                    RUNTIME EXECUTION FLOW                         │
└─────────────────────────────────────────────────────────────────┘

Command Execution (geist/commands/shape-spec/2-shape-spec.md)
   │
   ├─ 1. Context Detection
   │   ├─ Check: [ -d "geist/workflows" ]
   │   ├─ Set: WORKFLOWS_BASE="geist/workflows"
   │   └─ (or: "profiles/default/workflows" if testing templates)
   │
   ├─ 2. Load Context
   │   ├─ Read: geist/basepoints/headquarter.md ✅
   │   ├─ Read: geist/config/project-profile.yml ✅
   │   └─ Read: geist/output/handoff/current.md ✅ (if exists)
   │
   ├─ 3. Execute Workflows
   │   ├─ Source: $WORKFLOWS_BASE/prompting/construct-prompt.md ✅
   │   ├─ Source: $WORKFLOWS_BASE/basepoints/extract-basepoints-knowledge-automatic.md ✅
   │   └─ Workflows use: geist/... paths ✅
   │
   └─ 4. Save Results
       ├─ Write: geist/specs/[spec]/planning/requirements.md ✅
       └─ Write: geist/output/handoff/current.md ✅
```

### Specialization Flow: Abstract → Project-Specific

```
┌─────────────────────────────────────────────────────────────────┐
│                  SPECIALIZATION PROCESS                          │
└─────────────────────────────────────────────────────────────────┘

/deploy-agents Execution (runs during installation)

1. Read Templates (profiles/default/)
   ├─ Load: profiles/default/commands/shape-spec/single-agent/2-shape-spec.md
   ├─ Load: profiles/default/workflows/specification/research-spec.md
   └─ Load: profiles/default/standards/global/conventions.md

2. Load Project Context (geist/)
   ├─ Read: geist/basepoints/headquarter.md
   ├─ Read: geist/config/project-profile.yml
   └─ Read: geist/product/mission.md

3. Specialize Templates
   ├─ Inject: Project patterns from basepoints
   ├─ Replace: {{workflows/...}} with project-specific workflows
   ├─ Replace: {{standards/...}} with project-specific standards
   └─ Output: Specialized files to geist/commands/

4. Result (geist/)
   ├─ geist/commands/shape-spec/2-shape-spec.md (specialized)
   ├─ geist/workflows/specification/research-spec.md (specialized)
   └─ All references point to: geist/... ✅
```

---

## Summary

**Golden Rule**: Templates should reference `geist/...` paths because they will be installed into projects where `geist/` exists. Use context detection only when the file location itself needs to be dynamic (e.g., workflow sourcing that works in both template testing and installed contexts).

**Verified Correct Exceptions**:
- ✅ `/deploy-agents` phases read from `profiles/default/` to load templates for specialization
- ✅ Validation workflows check `profiles/default/` when validating templates
- ✅ Context detection checks `profiles/default/` to determine runtime context
- ✅ Test files explicitly test templates using `profiles/default/` paths
- ✅ Documentation references both locations for clarity

**All path references have been verified and are correct!** ✅
