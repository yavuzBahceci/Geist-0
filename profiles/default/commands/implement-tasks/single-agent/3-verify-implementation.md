Now that we've implemented all tasks in tasks.md, we must run final verifications and produce a verification report using the following MULTI-PHASE workflow:

## Workflow

### Step 1: Ensure tasks.md has been updated

{{workflows/implementation/verification/verify-tasks}}

### Step 2: Update roadmap (if applicable)

{{workflows/implementation/verification/update-roadmap}}

### Step 3: Run entire tests suite

{{workflows/implementation/verification/run-all-tests}}

### Step 4: Comprehensive Final Verification

Perform comprehensive verification to check for problems, gaps, and issues:

```bash
echo "🔍 Running comprehensive final verification..."

SPEC_PATH="geist/specs/[current-spec]"
VERIFICATION_ISSUES=""

# 4.1: Check for Problems and Gaps
echo "📋 Checking for problems and gaps..."

# Check for incomplete implementations
INCOMPLETE_TASKS=$(grep -c "^\- \[ \]" "$SPEC_PATH/tasks.md" 2>/dev/null || echo "0")
if [ "$INCOMPLETE_TASKS" -gt 0 ]; then
    VERIFICATION_ISSUES="${VERIFICATION_ISSUES}\n⚠️ INCOMPLETE: $INCOMPLETE_TASKS tasks still marked as incomplete"
fi

# Check for TODO/FIXME comments in implementation
TODO_COUNT=$(grep -rn "TODO\|FIXME\|XXX\|HACK" . --include="*.md" --include="*.sh" 2>/dev/null | wc -l | tr -d ' ')
if [ "$TODO_COUNT" -gt 0 ]; then
    VERIFICATION_ISSUES="${VERIFICATION_ISSUES}\n⚠️ TODO/FIXME: Found $TODO_COUNT TODO/FIXME comments that may need attention"
fi

# 4.2: Verify References
echo "📋 Checking references that need updating..."

# Check for broken references ({{...}} placeholders that weren't replaced)
BROKEN_REFS=$(grep -rn "{{[^}]*}}" . --include="*.md" 2>/dev/null | grep -v "profiles/default" | wc -l | tr -d ' ')
if [ "$BROKEN_REFS" -gt 0 ]; then
    VERIFICATION_ISSUES="${VERIFICATION_ISSUES}\n⚠️ BROKEN REFS: Found $BROKEN_REFS unresolved placeholder references"
fi

# Check for missing imports/dependencies
# (This is technology-specific, so we check for common patterns)
MISSING_IMPORTS=$(grep -rn "import.*from\|require(" . --include="*.ts" --include="*.js" --include="*.py" 2>/dev/null | grep -i "undefined\|not found" | wc -l | tr -d ' ')
if [ "$MISSING_IMPORTS" -gt 0 ]; then
    VERIFICATION_ISSUES="${VERIFICATION_ISSUES}\n⚠️ IMPORTS: Potential missing imports detected"
fi

# 4.3: Check Documentation
echo "📋 Checking documentation that needs updating..."

# Check if README needs updating
if [ -f "README.md" ]; then
    README_UPDATED=$(git diff --name-only 2>/dev/null | grep -c "README.md" || echo "0")
    if [ "$README_UPDATED" -eq 0 ]; then
        VERIFICATION_ISSUES="${VERIFICATION_ISSUES}\nℹ️ DOCS: README.md may need updating to reflect new changes"
    fi
fi

# 4.4: Check Code Quality
echo "📋 Checking code quality issues..."

# Check for missing dependencies in package files
if [ -f "package.json" ]; then
    # Check for packages used but not in dependencies
    echo "   Checking package.json dependencies..."
fi

if [ -f "requirements.txt" ]; then
    # Check for Python packages used but not in requirements
    echo "   Checking requirements.txt dependencies..."
fi

# 4.5: Verify Pattern Consistency
echo "📋 Checking pattern consistency with existing standards..."

# Load basepoints patterns for comparison
if [ -f "$SPEC_PATH/implementation/cache/basepoints-knowledge.md" ]; then
    echo "   Comparing implementation against basepoints patterns..."
fi

# 4.6: Ensure Implementation Completeness
echo "📋 Checking implementation completeness..."

# Verify all spec requirements are addressed
if [ -f "$SPEC_PATH/spec.md" ]; then
    SPEC_REQUIREMENTS=$(grep -c "^- " "$SPEC_PATH/spec.md" 2>/dev/null || echo "0")
    echo "   Spec requirements: $SPEC_REQUIREMENTS"
fi

# Generate verification issues report
if [ -n "$VERIFICATION_ISSUES" ]; then
    echo ""
    echo "═══════════════════════════════════════════════════════"
    echo "  VERIFICATION ISSUES FOUND"
    echo "═══════════════════════════════════════════════════════"
    echo -e "$VERIFICATION_ISSUES"
    echo "═══════════════════════════════════════════════════════"
    
    # Save issues to file
    echo -e "$VERIFICATION_ISSUES" > "$SPEC_PATH/implementation/cache/verification-issues.md"
else
    echo ""
    echo "✅ No verification issues found"
fi
```

### Step 5: Create final verification report

{{workflows/implementation/verification/create-verification-report}}

### Step 6: Capture Session Feedback

After verification completes, capture the implementation outcome for session learning:

{{workflows/learning/capture-session-feedback}}

### Step 7: Final Verification Summary

Output the final verification summary:

```
═══════════════════════════════════════════════════════
  IMPLEMENTATION VERIFICATION COMPLETE
═══════════════════════════════════════════════════════

## Verification Results

✅ Tasks Verified: [X of Y tasks complete]
✅ Tests: [PASSED / FAILED / SKIPPED]
✅ Code Quality: [PASSED / WARNINGS]
✅ Pattern Consistency: [PASSED / WARNINGS]
✅ Documentation: [Updated / Needs Update]
✅ References: [Valid / Broken refs found]

## Issues Found

[List any issues from comprehensive verification]

## Recommendations

[List any recommendations for follow-up]

## Reports

- Verification Report: `geist/specs/[this-spec]/implementation/cache/verification-report.md`
- Verification Issues: `geist/specs/[this-spec]/implementation/cache/verification-issues.md`
- Implementation Decision: `geist/specs/[this-spec]/implementation/cache/implementation-decision.md`

═══════════════════════════════════════════════════════
```
