# Phase 6: Iterative Fix Implementation

Use synthesized knowledge to implement the fix with iterative refinement.

## Step 1: Load Synthesized Knowledge

Load the knowledge synthesis from Phase 5:

```bash
CACHE_PATH="geist/output/fix-bug/cache"

if [ -f "$CACHE_PATH/knowledge-synthesis.md" ]; then
    KNOWLEDGE_SYNTHESIS=$(cat "$CACHE_PATH/knowledge-synthesis.md")
    echo "✅ Loaded knowledge synthesis"
else
    echo "❌ Knowledge synthesis not found. Run Phase 5 first."
    exit 1
fi

# Extract fix approaches
FIX_APPROACHES=$(grep -A 50 "## Prioritized Fix Approaches" "$CACHE_PATH/knowledge-synthesis.md")

# Extract implementation guidance
IMPLEMENTATION_GUIDANCE=$(grep -A 30 "## Implementation Guidance" "$CACHE_PATH/knowledge-synthesis.md")

# Initialize iteration tracking
ITERATION_COUNT=0
WORSENING_COUNT=0
MAX_WORSENING=3
ITERATION_HISTORY=""
```

## Step 2: Implement Initial Fix

Apply the first (highest confidence) fix approach:

```bash
echo "🔧 Implementing initial fix..."

ITERATION_COUNT=$((ITERATION_COUNT + 1))

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "  ITERATION $ITERATION_COUNT: Initial Fix Attempt"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Apply library-specific patterns and best practices
echo "   Applying library-specific patterns..."

# Follow basepoints patterns and standards
echo "   Following basepoints patterns and standards..."

# Implement the fix based on Approach 1
echo "   Implementing fix..."

# [AI agent implements the fix here based on synthesized knowledge]

# Record what was done
ITERATION_HISTORY="${ITERATION_HISTORY}

### Iteration $ITERATION_COUNT
- **Approach:** Initial fix (Approach 1)
- **Changes Made:** [List changes]
- **Files Modified:** [List files]
- **Rationale:** [Why this approach]
"
```

## Step 3: Test the Fix

Run validation to test if the fix works:

```bash
echo "🧪 Testing the fix..."

# Run project validation commands
{{workflows/validation/validate-implementation}}

# Capture test results
TEST_RESULT="unknown"
ERROR_COUNT_BEFORE="[from issue analysis]"
ERROR_COUNT_AFTER="[from test results]"

# Determine if fix is working
if [ "$ERROR_COUNT_AFTER" -eq 0 ]; then
    TEST_RESULT="success"
    echo "✅ Fix successful! Error resolved."
elif [ "$ERROR_COUNT_AFTER" -lt "$ERROR_COUNT_BEFORE" ]; then
    TEST_RESULT="improving"
    echo "📈 Getting closer! Errors reduced from $ERROR_COUNT_BEFORE to $ERROR_COUNT_AFTER"
elif [ "$ERROR_COUNT_AFTER" -eq "$ERROR_COUNT_BEFORE" ]; then
    TEST_RESULT="same"
    echo "➡️ Same errors. No improvement."
    WORSENING_COUNT=$((WORSENING_COUNT + 1))
else
    TEST_RESULT="worsening"
    echo "📉 More errors! Increased from $ERROR_COUNT_BEFORE to $ERROR_COUNT_AFTER"
    WORSENING_COUNT=$((WORSENING_COUNT + 1))
fi

# Update iteration history
ITERATION_HISTORY="${ITERATION_HISTORY}
- **Result:** $TEST_RESULT
- **Errors Before:** $ERROR_COUNT_BEFORE
- **Errors After:** $ERROR_COUNT_AFTER
"
```

## Step 4: Iterative Refinement Loop

Continue iterating based on results:

```bash
echo "🔄 Entering iterative refinement loop..."

while [ "$TEST_RESULT" != "success" ] && [ "$WORSENING_COUNT" -lt "$MAX_WORSENING" ]; do
    ITERATION_COUNT=$((ITERATION_COUNT + 1))
    
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "  ITERATION $ITERATION_COUNT: Refinement Attempt"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "  Worsening count: $WORSENING_COUNT / $MAX_WORSENING"
    
    # Determine next approach based on previous result
    if [ "$TEST_RESULT" = "improving" ]; then
        echo "   Continuing with current approach (making progress)..."
        # Refine current approach
    elif [ "$TEST_RESULT" = "same" ] || [ "$TEST_RESULT" = "worsening" ]; then
        echo "   Trying alternative approach..."
        # Try next approach from prioritized list
    fi
    
    # Apply fix refinement
    echo "   Implementing refinement..."
    
    # [AI agent implements refinement here]
    
    # Record iteration
    ITERATION_HISTORY="${ITERATION_HISTORY}

### Iteration $ITERATION_COUNT
- **Approach:** [Describe approach]
- **Changes Made:** [List changes]
- **Files Modified:** [List files]
- **Rationale:** [Why this approach]
"
    
    # Test again
    echo "🧪 Testing refinement..."
    
    # [Run tests and update TEST_RESULT, ERROR_COUNT_AFTER, WORSENING_COUNT]
    
    # Update iteration history with results
    ITERATION_HISTORY="${ITERATION_HISTORY}
- **Result:** $TEST_RESULT
- **Errors Before:** $ERROR_COUNT_BEFORE
- **Errors After:** $ERROR_COUNT_AFTER
"
done
```

## Step 5: Handle Stop Condition

If stop condition is met (3 worsening results), present knowledge summary and request guidance:

```bash
if [ "$WORSENING_COUNT" -ge "$MAX_WORSENING" ]; then
    echo ""
    echo "═══════════════════════════════════════════════════════════════"
    echo "  ⚠️ STOP CONDITION MET: 3 Worsening Results"
    echo "═══════════════════════════════════════════════════════════════"
    
    # Generate knowledge summary
    cat > "$CACHE_PATH/guidance-request.md" << 'GUIDANCE_EOF'
# Guidance Request

## Stop Condition
After $ITERATION_COUNT iterations, we've encountered $WORSENING_COUNT worsening results.
The fix attempts are not converging on a solution.

## Knowledge Summary

### What We Know
1. **Issue Type:** [From issue analysis]
2. **Affected Components:** [List]
3. **Root Cause Hypothesis:** [From synthesis]
4. **Library Insights:** [Key findings]
5. **Basepoints Patterns:** [Relevant patterns]
6. **Code Analysis:** [Key findings]

### What We've Tried

$ITERATION_HISTORY

### Current State
- **Current Error Count:** $ERROR_COUNT_AFTER
- **Original Error Count:** $ERROR_COUNT_BEFORE
- **Iterations Completed:** $ITERATION_COUNT
- **Worsening Results:** $WORSENING_COUNT

## Guidance Needed

We need your guidance to proceed. Please consider:

1. **Is the root cause hypothesis correct?**
   - Our hypothesis: [summary]
   - Alternative possibilities: [list]

2. **Are there constraints we're missing?**
   - Known constraints: [list]
   - Possible unknown constraints: [list]

3. **Should we try a different approach entirely?**
   - Approaches tried: [list]
   - Approaches not yet tried: [list]

4. **Is there additional context we need?**
   - Information we have: [list]
   - Information that might help: [list]

---

*Please provide guidance to continue fixing this issue.*
GUIDANCE_EOF

    echo ""
    echo "📋 Knowledge summary and guidance request saved to:"
    echo "   $CACHE_PATH/guidance-request.md"
    echo ""
    echo "⏳ Waiting for user guidance..."
    
    # Output to user
    cat "$CACHE_PATH/guidance-request.md"
    
    exit 0
fi
```

## Step 6: Generate Success Output

If fix is successful, generate success output:

```bash
if [ "$TEST_RESULT" = "success" ]; then
    echo ""
    echo "═══════════════════════════════════════════════════════════════"
    echo "  🎉 FIX SUCCESSFUL!"
    echo "═══════════════════════════════════════════════════════════════"
    
    # Generate success report
    cat > "$CACHE_PATH/fix-report.md" << 'REPORT_EOF'
# Fix Report

## Summary
- **Issue Type:** [bug/feedback]
- **Status:** ✅ RESOLVED
- **Iterations:** $ITERATION_COUNT
- **Date:** $(date)

## Issue Analysis
[Summary from Phase 1]

## Root Cause
[Identified root cause]

## Fix Applied

### Changes Made
$ITERATION_HISTORY

### Files Modified
[List all modified files]

## Knowledge Applied

### Library Patterns Used
[List library patterns that guided the fix]

### Basepoints Standards Followed
[List standards that were followed]

## Validation
- ✅ Error resolved
- ✅ Tests pass
- ✅ No new errors introduced
- ✅ Follows project patterns

## Recommendations

### Prevent Similar Issues
[Recommendations to prevent similar issues]

### Documentation Updates
[Any documentation that should be updated]

---

*Generated by fix-bug command*
REPORT_EOF

    echo ""
    echo "📋 Fix report saved to: $CACHE_PATH/fix-report.md"
    
    # Output summary to user
    echo ""
    echo "✅ Issue resolved in $ITERATION_COUNT iteration(s)"
    echo ""
    echo "## Changes Made"
    echo "$ITERATION_HISTORY" | grep -A 2 "Changes Made"
    echo ""
    echo "## Files Modified"
    echo "[List modified files]"
fi
```

## Display Final Status

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔧 PHASE 6: FIX IMPLEMENTATION COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Status: [SUCCESS / GUIDANCE NEEDED]
Iterations: [count]
Worsening Results: [count]

[If SUCCESS:]
✅ Issue resolved!
📋 Fix report: geist/output/fix-bug/cache/fix-report.md

[If GUIDANCE NEEDED:]
⚠️ Stop condition met after [count] worsening results
📋 Guidance request: geist/output/fix-bug/cache/guidance-request.md
⏳ Awaiting user guidance to continue
```
