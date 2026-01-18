# Path Reference Guide for profiles/default Templates

## Overview

Files in `profiles/default/` are **templates** that get installed into `agent-os/` when a project is set up. This guide explains how path references work and how to ensure they resolve correctly in both template and installed contexts.

---

## Reference Types

### 1. Template References (Compile-Time)

These are resolved during compilation when templates are installed:

| Pattern | Purpose | Resolution | Example |
|---------|---------|------------|---------|
| `{{workflows/...}}` | Inject workflow content | Replaced with workflow content during compilation | `{{workflows/prompting/construct-prompt}}` |
| `{{standards/...}}` | Inject standards content | Replaced with standards content during compilation | `{{standards/global/conventions}}` |
| `{{PHASE X: @agent-os/commands/...}}` | Embed phase file | Replaced with phase file content during compilation | `{{PHASE 1: @agent-os/commands/create-tasks/1-initialize.md}}` |

**✅ Correct**: These work in both contexts (template and installed)

---

### 2. Runtime Path References

These are file paths that need to resolve at runtime when commands/workflows execute:

| Pattern | Purpose | When Used | Resolution |
|---------|---------|-----------|------------|
| `agent-os/...` | Reference installed files | Always use installed paths | `agent-os/basepoints/headquarter.md` |
| `@agent-os/...` | Reference annotation | Documentation/inline references | `@agent-os/workflows/validation/orchestrate-validation.md` |
| Context-aware paths | Dynamic path resolution | When file location differs by context | Use `$WORKFLOWS_BASE` pattern |

**✅ Correct**: Always use `agent-os/...` paths in templates (they will exist in installed context)

---

### 3. Context Detection (Runtime)

Used to determine if running in template or installed context:

| Pattern | Purpose | When to Use |
|---------|---------|-------------|
| `if [ -d "agent-os/workflows" ]` | Check if installed | Before referencing workflows/commands |
| `if [ -d "profiles/default" ]` | Check if template | When validating template vs installed |
| `$WORKFLOWS_BASE` variable | Dynamic base path | When sourcing workflows that could be in either location |

**✅ Correct**: Use for runtime path resolution

---

## Path Mapping During Installation

```
BEFORE INSTALLATION (Template)          AFTER INSTALLATION (Installed)
─────────────────────────────────      ──────────────────────────────────

profiles/default/                       agent-os/
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
- Installed files are in `agent-os/` (exists in project)
- References in templates should point to `agent-os/...` paths
- Context detection determines runtime location

---

## Reference Resolution Logic

### During Compilation (`project-install.sh`)

```bash
# Template file
profiles/default/commands/shape-spec/single-agent/2-shape-spec.md
  → Contains: {{workflows/prompting/construct-prompt}}
  → Resolved: Workflow content embedded
  → Output: agent-os/commands/shape-spec/2-shape-spec.md (with workflow content)
```

### During Runtime (Command Execution)

```bash
# Installed file
agent-os/commands/shape-spec/2-shape-spec.md
  → Contains: agent-os/basepoints/headquarter.md
  → Resolved: File exists in project's agent-os/ directory
  → Works: ✅
```

### Context-Aware Resolution

```bash
# Template file
profiles/default/workflows/validation/validation-registry.md
  → Contains: source "$WORKFLOWS_BASE/validation/..."
  → Runtime check:
     if [ -d "agent-os/workflows" ]; then
         WORKFLOWS_BASE="agent-os/workflows"
     else
         WORKFLOWS_BASE="profiles/default/workflows"
     fi
  → Resolved: Correct path based on runtime context
```

---

## Common Patterns

### ✅ Correct Patterns

#### 1. Always Use `agent-os/` for Installed Paths

```bash
# ✅ Correct - References installed path
HEADQUARTER=$(cat agent-os/basepoints/headquarter.md 2>/dev/null || echo "")
PROJECT_PROFILE=$(cat agent-os/config/project-profile.yml 2>/dev/null || echo "")
```

#### 2. Use Context Detection for Workflows/Commands

```bash
# ✅ Correct - Context-aware workflow sourcing
if [ -d "agent-os/workflows" ]; then
    WORKFLOWS_BASE="agent-os/workflows"
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
    VALIDATION_CONTEXT="installed-agent-os"
    SCAN_PATH="agent-os"
fi
```

### ❌ Incorrect Patterns

#### 1. Direct `profiles/default/` Paths in Runtime Code

```bash
# ❌ Wrong - Will fail in installed context
source "profiles/default/workflows/validation/validate.md"

# ✅ Correct - Context-aware
if [ -d "agent-os/workflows" ]; then
    WORKFLOWS_BASE="agent-os/workflows"
else
    WORKFLOWS_BASE="profiles/default/workflows"
fi
source "$WORKFLOWS_BASE/validation/validate.md"
```

#### 2. Mixed Context Paths

```bash
# ❌ Wrong - Mixed paths
find "profiles/default/commands" -name "*.md"
find "agent-os/workflows" -name "*.md"

# ✅ Correct - Use consistent context detection
if [ -d "agent-os/commands" ]; then
    COMMANDS_DIR="agent-os/commands"
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
   ├─ Command file: agent-os/commands/shape-spec/2-shape-spec.md
   └─ Already compiled (workflows/standards injected)

2. Context Detection (if needed)
   ├─ Check: if [ -d "agent-os/workflows" ]
   ├─ Set: WORKFLOWS_BASE="agent-os/workflows"
   └─ Use: source "$WORKFLOWS_BASE/..."

3. Path References
   ├─ agent-os/basepoints/headquarter.md ✅ (installed path)
   ├─ agent-os/config/project-profile.yml ✅ (installed path)
   └─ agent-os/output/handoff/current.md ✅ (installed path)

4. Workflow Execution
   ├─ Source workflows from: $WORKFLOWS_BASE/... ✅ (context-aware)
   └─ Workflows use: agent-os/... paths ✅ (installed paths)

5. Command Completion
   └─ Save handoff to: agent-os/output/handoff/current.md ✅
```

### Workflow Reference Example: `review-trade-offs.md`

```bash
# Runtime resolution flow:

1. Workflow starts
   ├─ Location: agent-os/workflows/human-review/review-trade-offs.md
   └─ (or: profiles/default/workflows/human-review/review-trade-offs.md during template testing)

2. Context detection
   ├─ Check: [ -d "agent-os/workflows" ]
   ├─ Result: WORKFLOWS_BASE="agent-os/workflows"
   └─ (or: "profiles/default/workflows" if in template)

3. Source sub-workflows
   ├─ source "$WORKFLOWS_BASE/human-review/detect-trade-offs.md"
   ├─ Resolves to: agent-os/workflows/human-review/detect-trade-offs.md ✅
   └─ (or: profiles/default/workflows/... during template testing ✅)

4. Sub-workflow uses paths
   ├─ agent-os/basepoints/... ✅ (installed paths)
   └─ agent-os/output/... ✅ (installed paths)
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
**File**: `agent-os/workflows/prompting/construct-prompt.md` (when installed)  
**Template**: `profiles/default/workflows/prompting/construct-prompt.md`
```

### 3. Cleanup Commands

Commands that clean up installed files should detect and reference `agent-os/`:

```bash
# ✅ Correct - Cleanup command references installed paths
FILES_WITH_PROFILES_DEFAULT=$(find agent-os/commands -name "*.md" ...)
```

---

## Verification Checklist

Before committing changes to `profiles/default/`, verify:

- [ ] No direct `profiles/default/` paths in runtime code (only in context detection)
- [ ] All runtime paths use `agent-os/...` 
- [ ] Workflow sourcing uses context detection (`$WORKFLOWS_BASE`)
- [ ] Template references use `{{workflows/...}}` or `{{standards/...}}`
- [ ] Test files explicitly testing templates can use `profiles/default/`
- [ ] Documentation references both template and installed locations where relevant

---

## Quick Reference Table

| Reference Type | Template | Installed | Pattern |
|---------------|----------|-----------|---------|
| Compile-time injection | ✅ | ✅ | `{{workflows/...}}` |
| Installed file paths | ❌ | ✅ | `agent-os/...` |
| Context-aware workflows | ✅ | ✅ | `$WORKFLOWS_BASE/...` |
| Context detection | ✅ | ✅ | `if [ -d "agent-os/workflows" ]` |
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
    VALIDATION_CONTEXT="installed-agent-os"
    SCAN_PATH="agent-os"
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
   └─ References: Use agent-os/ paths (for runtime) or {{...}} (for compile-time)

2. project-install.sh Execution
   ├─ Reads templates from: profiles/default/
   ├─ Resolves: {{workflows/...}} → workflow content
   ├─ Resolves: {{standards/...}} → standards content
   └─ Outputs: Compiled files to agent-os/

3. Installed Files (agent-os/)
   ├─ Contains: Specialized, compiled commands with embedded content
   ├─ Location: Project directory
   └─ References: All use agent-os/ paths (runtime resolution)
```

### Runtime Execution Flow: Command → Workflows

```
┌─────────────────────────────────────────────────────────────────┐
│                    RUNTIME EXECUTION FLOW                         │
└─────────────────────────────────────────────────────────────────┘

Command Execution (agent-os/commands/shape-spec/2-shape-spec.md)
   │
   ├─ 1. Context Detection
   │   ├─ Check: [ -d "agent-os/workflows" ]
   │   ├─ Set: WORKFLOWS_BASE="agent-os/workflows"
   │   └─ (or: "profiles/default/workflows" if testing templates)
   │
   ├─ 2. Load Context
   │   ├─ Read: agent-os/basepoints/headquarter.md ✅
   │   ├─ Read: agent-os/config/project-profile.yml ✅
   │   └─ Read: agent-os/output/handoff/current.md ✅ (if exists)
   │
   ├─ 3. Execute Workflows
   │   ├─ Source: $WORKFLOWS_BASE/prompting/construct-prompt.md ✅
   │   ├─ Source: $WORKFLOWS_BASE/basepoints/extract-basepoints-knowledge-automatic.md ✅
   │   └─ Workflows use: agent-os/... paths ✅
   │
   └─ 4. Save Results
       ├─ Write: agent-os/specs/[spec]/planning/requirements.md ✅
       └─ Write: agent-os/output/handoff/current.md ✅
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

2. Load Project Context (agent-os/)
   ├─ Read: agent-os/basepoints/headquarter.md
   ├─ Read: agent-os/config/project-profile.yml
   └─ Read: agent-os/product/mission.md

3. Specialize Templates
   ├─ Inject: Project patterns from basepoints
   ├─ Replace: {{workflows/...}} with project-specific workflows
   ├─ Replace: {{standards/...}} with project-specific standards
   └─ Output: Specialized files to agent-os/commands/

4. Result (agent-os/)
   ├─ agent-os/commands/shape-spec/2-shape-spec.md (specialized)
   ├─ agent-os/workflows/specification/research-spec.md (specialized)
   └─ All references point to: agent-os/... ✅
```

---

## Summary

**Golden Rule**: Templates should reference `agent-os/...` paths because they will be installed into projects where `agent-os/` exists. Use context detection only when the file location itself needs to be dynamic (e.g., workflow sourcing that works in both template testing and installed contexts).

**Verified Correct Exceptions**:
- ✅ `/deploy-agents` phases read from `profiles/default/` to load templates for specialization
- ✅ Validation workflows check `profiles/default/` when validating templates
- ✅ Context detection checks `profiles/default/` to determine runtime context
- ✅ Test files explicitly test templates using `profiles/default/` paths
- ✅ Documentation references both locations for clarity

**All path references have been verified and are correct!** ✅
