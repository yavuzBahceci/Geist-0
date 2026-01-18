---
name: Refactoring Plan - Reduce Redundancy in Commands and Workflows
overview: Identify and eliminate repetitive patterns in commands and workflows, leveraging the existing workflow injection system to reduce file length and maintenance burden.
version: 1.0
date: 2026-01
todos: []
---

# Refactoring Plan: Reduce Redundancy in Commands and Workflows

## Problem Statement

Commands and workflows are too long with significant repetition:
1. **Entire workflows copied inline** instead of using `{{workflows/...}}` references
2. **Basepoints extraction code duplicated** across multiple command phases
3. **Repetitive boilerplate** for checking basepoints existence
4. **Verbose explanations** that could be shortened or referenced
5. **Same patterns repeated** with slight variations

## Key Findings

### Critical Issue: Inline Workflow Copies

**Problem:** `shape-spec/2-shape-spec.md` (3126 lines) contains the entire basepoints extraction workflow copied inline.

**Current (Bad):**
```markdown
## Step 1: Extract Basepoints Knowledge (if basepoints exist)

Before researching requirements, extract relevant basepoints knowledge:

```bash
# Check if basepoints exist
if [ -d "agent-os/basepoints" ] && [ -f "agent-os/basepoints/headquarter.md" ]; then
    # [3000+ lines of workflow code copied here]
fi
```

**Should Be (Good):**
```markdown
## Step 1: Extract Basepoints Knowledge

Extract relevant basepoints knowledge to inform the research process:

{{workflows/basepoints/extract-basepoints-knowledge-automatic}}
```

### Repetitive Basepoints Checks

**Found in:** All command phases that need basepoints knowledge:
- `shape-spec/2-shape-spec.md`
- `write-spec/single-agent/write-spec.md`
- `create-tasks/single-agent/2-create-tasks-list.md`
- `implement-tasks/single-agent/2-implement-tasks.md`
- `orchestrate-tasks/orchestrate-tasks.md`

**Current Pattern:**
```markdown
## Step 1: Extract Basepoints Knowledge (if basepoints exist)

Before [action], extract basepoints knowledge:

```bash
# Check if basepoints exist
if [ -d "agent-os/basepoints" ] && [ -f "agent-os/basepoints/headquarter.md" ]; then
    SPEC_PATH="agent-os/specs/[current-spec]"
    # Extract basepoints knowledge
    {{workflows/basepoints/extract-basepoints-knowledge-automatic}}
    
    # Load extracted knowledge
    if [ -f "$SPEC_PATH/implementation/cache/basepoints-knowledge.md" ]; then
        EXTRACTED_KNOWLEDGE=$(cat "$SPEC_PATH/implementation/cache/basepoints-knowledge.md")
    fi
fi
```
```

**Should Be:**
```markdown
## Step 1: Extract Basepoints Knowledge

Extract basepoints knowledge to inform this phase:

{{workflows/basepoints/extract-basepoints-knowledge-automatic}}

The workflow handles graceful fallback if basepoints don't exist.
```

## Refactoring Strategy

### 1. Replace Inline Workflows with References

**Principle:** Never copy workflow content inline. Always use `{{workflows/...}}` references.

**Actions:**
1. Find all inline workflow copies
2. Replace with workflow references
3. Remove duplicate code

**Files to Fix:**
- `profiles/default/commands/shape-spec/single-agent/2-shape-spec.md` (3126 lines → ~200 lines)
- Any other commands with inline workflow copies

### 2. Create Standard Workflow Wrappers

**Principle:** Create reusable wrapper workflows for common patterns.

**New Workflows to Create:**
- `workflows/common/extract-basepoints-and-load.md` - Combines extraction + loading
- `workflows/common/check-prerequisites.md` - Standard prerequisite checking
- `workflows/common/initialize-context.md` - Standard context initialization

**Example:**
```markdown
# workflows/common/extract-basepoints-and-load.md

Extract basepoints knowledge and load it into context:

{{workflows/basepoints/extract-basepoints-knowledge-automatic}}

Load extracted knowledge:
```bash
if [ -f "$SPEC_PATH/implementation/cache/basepoints-knowledge.md" ]; then
    EXTRACTED_KNOWLEDGE=$(cat "$SPEC_PATH/implementation/cache/basepoints-knowledge.md")
fi

if [ -f "$SPEC_PATH/implementation/cache/detected-layer.txt" ]; then
    DETECTED_LAYER=$(cat "$SPEC_PATH/implementation/cache/detected-layer.txt")
fi
```

This wrapper can then be used in all commands:
```markdown
{{workflows/common/extract-basepoints-and-load}}
```

### 3. Shorten Verbose Explanations

**Principle:** Move detailed explanations to workflow files or standards. Keep commands concise.

**Current Pattern:**
```markdown
## Step 1: Extract Basepoints Knowledge (if basepoints exist)

Before researching requirements, extract relevant basepoints knowledge to inform the research process. This will help identify existing patterns, standards, and approaches used in the codebase that should be considered when researching requirements.

```bash
# Check if basepoints exist
# [verbose explanation of what this does]
# [more explanation]
# [even more explanation]
```

**Should Be:**
```markdown
## Step 1: Extract Basepoints Knowledge

Extract basepoints knowledge to inform requirements research:

{{workflows/common/extract-basepoints-and-load}}

See `{{workflows/basepoints/extract-basepoints-knowledge-automatic}}` for details.
```

### 4. Consolidate Repetitive Patterns

**Pattern:** "Get spec path" or "Determine spec" appears in many commands.

**Solution:** Create standard workflow:
```markdown
# workflows/common/determine-spec-context.md

Determine spec path and context:
```bash
# Get current spec from context or prompt
if [ -z "$SPEC_PATH" ]; then
    # Logic to determine spec path
fi
```
```

**Usage:**
```markdown
{{workflows/common/determine-spec-context}}
```

### 5. Extract Common Command Patterns

**Pattern:** Many commands start with similar steps:
1. Get/validate input
2. Extract basepoints
3. Load context
4. Execute main logic

**Solution:** Create command templates or standard phase sequences:

```markdown
# workflows/commands/standard-command-start.md

Standard command initialization:

{{workflows/common/determine-spec-context}}
{{workflows/common/extract-basepoints-and-load}}
{{workflows/common/validate-prerequisites}}
```

### 6. Simplify Workflow Descriptions

**Current:** Workflows have verbose "Core Responsibilities" sections that repeat in command files.

**Solution:** 
- Keep detailed descriptions in workflow files only
- Commands reference workflows with brief, purpose-specific context
- Use `{{workflows/...}}` to inject full workflow content

## Implementation Plan

### Phase 1: Audit and Document Redundancy (Week 1)

**Tasks:**
1. Scan all command files for inline workflow copies
2. Identify repetitive patterns across commands
3. Create inventory of redundant code blocks
4. Document common patterns

**Deliverables:**
- Redundancy audit report
- Pattern inventory
- Files requiring refactoring (priority ordered)

### Phase 2: Create Common Workflows (Week 1)

**Tasks:**
1. Create `workflows/common/` directory
2. Extract common patterns into reusable workflows:
   - `extract-basepoints-and-load.md`
   - `determine-spec-context.md`
   - `validate-prerequisites.md`
   - `initialize-context.md`
   - `cache-knowledge.md`
   - `load-cached-knowledge.md`

**Files to Create:**
- `profiles/default/workflows/common/extract-basepoints-and-load.md`
- `profiles/default/workflows/common/determine-spec-context.md`
- `profiles/default/workflows/common/validate-prerequisites.md`
- `profiles/default/workflows/common/initialize-context.md`
- `profiles/default/workflows/common/cache-knowledge.md`
- `profiles/default/workflows/common/load-cached-knowledge.md`

### Phase 3: Refactor High-Impact Files (Week 2)

**Priority Files (largest impact):**
1. `profiles/default/commands/shape-spec/single-agent/2-shape-spec.md` (3126 → ~200 lines)
2. `profiles/default/commands/write-spec/single-agent/write-spec.md`
3. `profiles/default/commands/create-tasks/single-agent/2-create-tasks-list.md`
4. `profiles/default/commands/implement-tasks/single-agent/2-implement-tasks.md`
5. `profiles/default/commands/orchestrate-tasks/orchestrate-tasks.md`

**Refactoring Steps:**
1. Replace inline workflow copies with `{{workflows/...}}` references
2. Replace repetitive basepoints code with `{{workflows/common/extract-basepoints-and-load}}`
3. Shorten verbose explanations
4. Use common workflows for standard patterns

### Phase 4: Refactor Remaining Commands (Week 2)

**Tasks:**
1. Apply common workflow patterns to all commands
2. Remove redundant explanations
3. Consolidate similar patterns
4. Update command documentation

### Phase 5: Validation and Testing (Week 3)

**Tasks:**
1. Verify workflow compilation works correctly
2. Test that all references resolve
3. Ensure functionality is preserved
4. Validate file length reductions

**Success Metrics:**
- Average command file length reduced by 60-80%
- No inline workflow copies remain
- All basepoints extraction uses standard workflows
- Compilation system works correctly

## Specific File Refactoring Examples

### Example 1: shape-spec/2-shape-spec.md

**Current:** 3126 lines (entire basepoints workflow inline)

**Target:** ~200 lines

**Refactoring:**
```markdown
# Current structure (simplified):
## Step 1: Extract Basepoints Knowledge (if basepoints exist)

[3000 lines of inline workflow code]

## Step 2: Research Requirements
[actual research logic]

# Refactored structure:
## Step 1: Extract Basepoints Knowledge

Extract basepoints knowledge to inform requirements research:

{{workflows/common/extract-basepoints-and-load}}

## Step 2: Research Requirements

Research feature requirements:

{{workflows/specification/research-spec}}

[Only include command-specific logic here]
```

### Example 2: Multiple Commands with Same Pattern

**Current Pattern (repeated 5+ times):**
```markdown
## Step 1: Extract Basepoints Knowledge (if basepoints exist)

Before [action], extract basepoints knowledge to inform the [action] process. This will help identify existing patterns and standards.

```bash
# Check if basepoints exist
if [ -d "agent-os/basepoints" ] && [ -f "agent-os/basepoints/headquarter.md" ]; then
    SPEC_PATH="agent-os/specs/[current-spec]"
    
    # Extract basepoints knowledge using scope detection
    {{workflows/basepoints/extract-basepoints-knowledge-automatic}}
    {{workflows/scope-detection/detect-abstraction-layer}}
    {{workflows/scope-detection/detect-scope-semantic-analysis}}
    {{workflows/scope-detection/detect-scope-keyword-matching}}
    
    # Load extracted knowledge for context
    if [ -f "$SPEC_PATH/implementation/cache/basepoints-knowledge.md" ]; then
        EXTRACTED_KNOWLEDGE=$(cat "$SPEC_PATH/implementation/cache/basepoints-knowledge.md")
    fi
    
    # Load detected layer for module matching
    if [ -f "$SPEC_PATH/implementation/cache/detected-layer.txt" ]; then
        DETECTED_LAYER=$(cat "$SPEC_PATH/implementation/cache/detected-layer.txt")
    fi
fi
```
```

**Refactored (Single Line):**
```markdown
## Step 1: Extract Basepoints Knowledge

Extract basepoints knowledge with scope detection:

{{workflows/common/extract-basepoints-with-scope-detection}}
```

**New Workflow:**
```markdown
# workflows/common/extract-basepoints-with-scope-detection.md

Extract basepoints knowledge with scope detection:

{{workflows/basepoints/extract-basepoints-knowledge-automatic}}
{{workflows/scope-detection/detect-abstraction-layer}}
{{workflows/scope-detection/detect-scope-semantic-analysis}}
{{workflows/scope-detection/detect-scope-keyword-matching}}

Load extracted knowledge:
```bash
if [ -f "$SPEC_PATH/implementation/cache/basepoints-knowledge.md" ]; then
    EXTRACTED_KNOWLEDGE=$(cat "$SPEC_PATH/implementation/cache/basepoints-knowledge.md")
fi

if [ -f "$SPEC_PATH/implementation/cache/detected-layer.txt" ]; then
    DETECTED_LAYER=$(cat "$SPEC_PATH/implementation/cache/detected-layer.txt")
fi
```
```

## Guidelines for Future Commands

### DO:
- ✅ Use `{{workflows/...}}` references instead of copying workflow content
- ✅ Use common workflows for standard patterns
- ✅ Keep command files focused on command-specific logic
- ✅ Reference workflows/standards for detailed explanations
- ✅ Keep commands concise and scannable

### DON'T:
- ❌ Copy workflow code inline
- ❌ Repeat basepoints extraction code
- ❌ Include verbose explanations already in workflows
- ❌ Duplicate common patterns across commands
- ❌ Write commands longer than 500 lines (should be much shorter)

## Expected Outcomes

### File Length Reductions

**Before:**
- `shape-spec/2-shape-spec.md`: 3126 lines
- Average command phase: ~500-800 lines
- Total command code: ~50,000+ lines

**After:**
- `shape-spec/2-shape-spec.md`: ~200 lines (94% reduction)
- Average command phase: ~100-200 lines (60-75% reduction)
- Total command code: ~15,000-20,000 lines (60-70% reduction)

### Maintenance Benefits

1. **Single Source of Truth:** Workflow changes update everywhere automatically
2. **Easier Testing:** Test workflows once, not in every command
3. **Better Readability:** Commands focus on their unique logic
4. **Faster Updates:** Update common patterns in one place
5. **Less Error-Prone:** No risk of workflow code getting out of sync

### Compilation Benefits

The existing compilation system already handles `{{workflows/...}}` injection. By using it more extensively:
- Commands become more maintainable
- Workflows are reusable
- System stays DRY (Don't Repeat Yourself)

## Risk Mitigation

**Risk:** Breaking existing functionality during refactoring

**Mitigation:**
1. Test compilation system works with all references
2. Validate each refactored command maintains functionality
3. Keep original files as backup during refactoring
4. Refactor incrementally, one command at a time
5. Test after each refactoring phase

**Risk:** Workflow references might not work in all contexts

**Mitigation:**
1. Test workflow injection in all contexts
2. Document any limitations
3. Create context-specific workflow wrappers if needed
4. Ensure compilation system handles nested workflows correctly

## Next Steps

1. **Review this plan** - Confirm approach and priorities
2. **Start Phase 1** - Audit current redundancy
3. **Create common workflows** - Build reusable patterns
4. **Refactor high-impact files** - Start with largest files
5. **Iterate** - Refine based on learnings

---

*This refactoring will significantly reduce code duplication, improve maintainability, and leverage the existing workflow compilation system effectively.*
