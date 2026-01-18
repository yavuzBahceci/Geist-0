# Phase 5: Knowledge Synthesis

Combine all knowledge sources into comprehensive analysis.

## Step 1: Load All Previous Analysis

Load all analysis from previous phases:

```bash
CACHE_PATH="agent-os/output/fix-bug/cache"

echo "📖 Loading all previous analysis..."

# Phase 1: Issue Analysis
ISSUE_ANALYSIS=$(cat "$CACHE_PATH/issue-analysis.md" 2>/dev/null)
echo "   ✅ Issue analysis loaded"

# Phase 2: Library Research
LIBRARY_RESEARCH=$(cat "$CACHE_PATH/library-research.md" 2>/dev/null)
echo "   ✅ Library research loaded"

# Phase 3: Basepoints Integration
BASEPOINTS_INTEGRATION=$(cat "$CACHE_PATH/basepoints-integration.md" 2>/dev/null)
echo "   ✅ Basepoints integration loaded"

# Phase 4: Code Analysis
CODE_ANALYSIS=$(cat "$CACHE_PATH/code-analysis.md" 2>/dev/null)
echo "   ✅ Code analysis loaded"
```

## Step 2: Extract Key Findings from Each Source

Extract the most important findings from each knowledge source:

```bash
echo "🔍 Extracting key findings..."

# From Issue Analysis
ISSUE_TYPE=$(grep "Input Type" "$CACHE_PATH/issue-analysis.md" | head -1)
AFFECTED_COMPONENTS=$(grep -A 20 "## Affected Components" "$CACHE_PATH/issue-analysis.md" | head -20)

# From Library Research
LIBRARY_FINDINGS=$(grep -A 10 "## Key Findings" "$CACHE_PATH/library-research.md" | head -10)
POTENTIAL_CAUSES=$(grep -A 10 "### Potential Root Causes" "$CACHE_PATH/library-research.md" | head -10)

# From Basepoints Integration
PATTERNS_TO_APPLY=$(grep -A 10 "### Patterns to Apply" "$CACHE_PATH/basepoints-integration.md" | head -10)
STANDARDS_TO_FOLLOW=$(grep -A 10 "### Standards to Follow" "$CACHE_PATH/basepoints-integration.md" | head -10)

# From Code Analysis
ROOT_CAUSE_HYPOTHESIS=$(grep -A 10 "### Root Cause Hypothesis" "$CACHE_PATH/code-analysis.md" | head -10)
PROBLEMATIC_CODE=$(grep -A 10 "### Problematic Code Sections" "$CACHE_PATH/code-analysis.md" | head -10)

echo "   Key findings extracted from all sources"
```

## Step 3: Create Unified View of Issue Context

Combine findings into a unified view:

```bash
echo "🔄 Creating unified view of issue context..."

UNIFIED_CONTEXT="
# Unified Issue Context

## Issue Overview
$ISSUE_TYPE

### Affected Components
$AFFECTED_COMPONENTS

## Root Cause Analysis

### From Library Research
$POTENTIAL_CAUSES

### From Code Analysis
$ROOT_CAUSE_HYPOTHESIS

### Problematic Areas
$PROBLEMATIC_CODE

## Solution Guidance

### Patterns to Apply
$PATTERNS_TO_APPLY

### Standards to Follow
$STANDARDS_TO_FOLLOW

### Library-Specific Guidance
$LIBRARY_FINDINGS
"
```

## Step 4: Prioritize Fix Approaches

Based on synthesized knowledge, prioritize fix approaches:

```bash
echo "📊 Prioritizing fix approaches..."

FIX_APPROACHES="
## Prioritized Fix Approaches

### Approach 1: [Most Likely Fix]
- **Confidence:** High/Medium/Low
- **Based on:** [Which knowledge sources support this]
- **Steps:**
  1. [Step 1]
  2. [Step 2]
  3. [Step 3]
- **Risks:** [Potential risks]

### Approach 2: [Alternative Fix]
- **Confidence:** High/Medium/Low
- **Based on:** [Which knowledge sources support this]
- **Steps:**
  1. [Step 1]
  2. [Step 2]
- **Risks:** [Potential risks]

### Approach 3: [Fallback Fix]
- **Confidence:** High/Medium/Low
- **Based on:** [Which knowledge sources support this]
- **Steps:**
  1. [Step 1]
  2. [Step 2]
- **Risks:** [Potential risks]
"
```

## Step 5: Prepare Knowledge for Fix Implementation

Prepare the synthesized knowledge for use in fix implementation:

```bash
echo "📝 Preparing knowledge for fix implementation..."

FIX_CONTEXT="
# Fix Implementation Context

## Quick Reference

### Issue Type
$ISSUE_TYPE

### Root Cause
$ROOT_CAUSE_HYPOTHESIS

### Files to Modify
$PROBLEMATIC_CODE

## Implementation Guidance

### Patterns to Apply
$PATTERNS_TO_APPLY

### Standards to Follow
$STANDARDS_TO_FOLLOW

### Library Best Practices
$LIBRARY_FINDINGS

## Validation Criteria

### Success Indicators
- [ ] Error no longer occurs
- [ ] Tests pass
- [ ] No new errors introduced
- [ ] Follows project patterns
- [ ] Follows library best practices

### Regression Checks
- [ ] Existing functionality preserved
- [ ] Performance not degraded
- [ ] No security issues introduced
"
```

## Step 6: Create Knowledge Synthesis Document

Generate the comprehensive knowledge synthesis document:

```bash
echo "📝 Creating knowledge synthesis document..."

cat > "$CACHE_PATH/knowledge-synthesis.md" << 'SYNTHESIS_EOF'
# Knowledge Synthesis

## Synthesis Summary
- **Knowledge Sources:** 4 (Issue Analysis, Library Research, Basepoints, Code Analysis)
- **Synthesis Date:** $(date)

---

$UNIFIED_CONTEXT

---

$FIX_APPROACHES

---

$FIX_CONTEXT

---

## Knowledge Summary (for guidance request if needed)

### What We Know
1. **Issue Type:** [bug/feedback]
2. **Affected Components:** [list]
3. **Root Cause Hypothesis:** [summary]
4. **Library Insights:** [key library findings]
5. **Basepoints Patterns:** [relevant patterns]
6. **Code Analysis:** [key code findings]

### What We've Tried
[Will be updated during fix implementation]

### Current State
[Will be updated during fix implementation]

---

*Generated by fix-bug command - Phase 5*
SYNTHESIS_EOF

echo "✅ Knowledge synthesis complete"
echo "   Synthesis saved to: $CACHE_PATH/knowledge-synthesis.md"
```

## Display Progress and Next Step

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧠 PHASE 5: KNOWLEDGE SYNTHESIS COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ All knowledge sources combined
✅ Unified issue context created
✅ Fix approaches prioritized
✅ Implementation guidance prepared

Synthesis saved to: agent-os/output/fix-bug/cache/knowledge-synthesis.md

NEXT STEP 👉 Proceeding to Phase 6: Implement Fix
```
