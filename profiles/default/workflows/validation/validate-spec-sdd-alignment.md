# Validate Spec SDD Alignment

## Core Responsibilities

1. **Check Spec Completeness**: Verify spec has clear requirements, acceptance criteria, scope boundaries
2. **Check Spec Structure**: Verify spec follows SDD best practices (user stories format, minimal scope, intentional boundaries)
3. **Check Anti-Patterns**: Detect specification theater, premature comprehensiveness, over-specification
4. **Check Spec-Implementation Alignment**: Validate spec-implementation alignment when implementation exists
5. **Return Status**: Pass/fail with detailed findings

## Workflow

### Step 1: Define Paths

```bash
# SPEC_PATH should be set by the calling workflow
if [ -z "$SPEC_PATH" ]; then
    echo "❌ SPEC_PATH not set"
    exit 1
fi

echo "🔍 Validating spec SDD alignment..."

SPEC_FILE="$SPEC_PATH/spec.md"
REQUIREMENTS_FILE="$SPEC_PATH/planning/requirements.md"
TASKS_FILE="$SPEC_PATH/tasks.md"
IMPLEMENTATION_PATH="$SPEC_PATH/implementation"

CACHE_PATH="$SPEC_PATH/implementation/cache"
VALIDATION_PATH="$CACHE_PATH/validation"
mkdir -p "$VALIDATION_PATH"

# Initialize tracking
ERRORS=0
WARNINGS=0
PASSED=0
```

### Step 2: Check Spec Completeness (SDD-aligned)

```bash
echo ""
echo "📋 Checking spec completeness (SDD requirements)..."
echo ""

if [ ! -f "$SPEC_FILE" ]; then
    echo "  ❌ spec.md not found - cannot validate SDD alignment"
    ERRORS=$((ERRORS + 1))
    exit 1
fi

# Check for user stories format: "As a [user], I want [action], so that [benefit]"
USER_STORIES=$(grep -iE "as a .*i want .*so that|user story" "$SPEC_FILE" | wc -l)
if [ "$USER_STORIES" -gt 0 ]; then
    echo "  ✅ User stories present: $USER_STORIES found"
    PASSED=$((PASSED + 1))
else
    echo "  ❌ User stories missing: Should include format 'As a [user], I want [action], so that [benefit]'"
    ERRORS=$((ERRORS + 1))
fi

# Check for acceptance criteria
ACCEPTANCE_CRITERIA=$(grep -iE "acceptance criteria|Acceptance Criteria" "$SPEC_FILE" | wc -l)
if [ "$ACCEPTANCE_CRITERIA" -gt 0 ]; then
    echo "  ✅ Acceptance criteria present: $ACCEPTANCE_CRITERIA found"
    PASSED=$((PASSED + 1))
else
    echo "  ❌ Acceptance criteria missing: Should include explicit acceptance criteria"
    ERRORS=$((ERRORS + 1))
fi

# Check for scope boundaries (in-scope vs out-of-scope)
SCOPE_BOUNDARIES=$(grep -iE "in scope|out of scope|scope boundary|Scope:" "$SPEC_FILE" | wc -l)
if [ "$SCOPE_BOUNDARIES" -gt 0 ]; then
    echo "  ✅ Scope boundaries present: $SCOPE_BOUNDARIES found"
    PASSED=$((PASSED + 1))
else
    echo "  ⚠️ Scope boundaries missing: Should explicitly define in-scope vs out-of-scope"
    WARNINGS=$((WARNINGS + 1))
fi

# Check for clear requirements
REQUIREMENTS=$(grep -iE "requirement|Requirements|Specific Requirements" "$SPEC_FILE" | wc -l)
if [ "$REQUIREMENTS" -gt 0 ]; then
    echo "  ✅ Clear requirements present: $REQUIREMENTS found"
    PASSED=$((PASSED + 1))
else
    echo "  ⚠️ Requirements section may be missing or unclear"
    WARNINGS=$((WARNINGS + 1))
fi
```

### Step 3: Check Spec Structure (SDD-aligned)

```bash
echo ""
echo "📐 Checking spec structure (SDD best practices)..."
echo ""

# Check for What & Why focus (not How - implementation details belong in task phase)
PREMATURE_TECH=$(grep -iE "implementation details|code structure|database schema|api endpoints|class hierarchy|architecture diagram|tech stack|framework|library|npm package|import|require" "$SPEC_FILE" | wc -l)
if [ "$PREMATURE_TECH" -gt 5 ]; then
    echo "  ❌ Premature technical details detected: $PREMATURE_TECH instances (SDD: spec should focus on What & Why, not How)"
    ERRORS=$((ERRORS + 1))
else
    echo "  ✅ Spec focuses on What & Why (not premature technical details)"
    PASSED=$((PASSED + 1))
fi

# Check for minimal scope (spec should not be over-specified)
SPEC_LINE_COUNT=$(wc -l < "$SPEC_FILE" 2>/dev/null || echo "0")
SPEC_SECTION_COUNT=$(grep -c "^##" "$SPEC_FILE" 2>/dev/null || echo "0")

if [ "$SPEC_LINE_COUNT" -gt 1000 ] || [ "$SPEC_SECTION_COUNT" -gt 20 ]; then
    echo "  ⚠️ Spec may be over-specified: $SPEC_LINE_COUNT lines, $SPEC_SECTION_COUNT sections (SDD: minimal, intentional scope preferred)"
    WARNINGS=$((WARNINGS + 1))
elif [ "$SPEC_LINE_COUNT" -gt 500 ] || [ "$SPEC_SECTION_COUNT" -gt 15 ]; then
    echo "  ⚠️ Spec is moderately large: $SPEC_LINE_COUNT lines, $SPEC_SECTION_COUNT sections (consider breaking into smaller, focused specs)"
    WARNINGS=$((WARNINGS + 1))
else
    echo "  ✅ Spec has minimal, intentional scope: $SPEC_LINE_COUNT lines, $SPEC_SECTION_COUNT sections"
    PASSED=$((PASSED + 1))
fi

# Check for intentional boundaries (explicit in-scope vs out-of-scope)
# Already checked in Step 2, but verify it's explicit
EXPLICIT_OUT_OF_SCOPE=$(grep -iE "out of scope|Out of Scope|out-of-scope" "$SPEC_FILE" | wc -l)
if [ "$EXPLICIT_OUT_OF_SCOPE" -gt 0 ]; then
    echo "  ✅ Explicit out-of-scope boundaries defined"
    PASSED=$((PASSED + 1))
else
    echo "  ⚠️ Out-of-scope section may be missing (SDD: explicit boundaries preferred)"
    WARNINGS=$((WARNINGS + 1))
fi
```

### Step 4: Check Anti-Patterns (SDD-aligned)

```bash
echo ""
echo "🚫 Checking for SDD anti-patterns..."
echo ""

# Check for specification theater (specs that are written but never referenced)
# This is harder to detect automatically, but we can check if spec is very vague
VAGUE_SPEC=$(grep -iE "should|could|might|maybe|perhaps|kind of|sort of" "$SPEC_FILE" | wc -l)
if [ "$VAGUE_SPEC" -gt 20 ]; then
    echo "  ⚠️ Spec may contain vague requirements: $VAGUE_SPEC instances (may indicate specification theater)"
    WARNINGS=$((WARNINGS + 1))
else
    echo "  ✅ Spec appears actionable (not vague specification theater)"
    PASSED=$((PASSED + 1))
fi

# Check for premature comprehensiveness (trying to spec everything upfront)
# Already checked in Step 3 (over-specification), but add specific check
if [ "$SPEC_LINE_COUNT" -gt 1000 ] || [ "$SPEC_SECTION_COUNT" -gt 20 ]; then
    echo "  ⚠️ Spec may be prematurely comprehensive: Too many details upfront (SDD: incremental, focused specs preferred)"
    WARNINGS=$((WARNINGS + 1))
fi

# Check for over-specification (excessive detail)
# Already checked in Step 3, but consolidate here
ANTI_PATTERN_DETECTED="false"
if [ "$PREMATURE_TECH" -gt 5 ] || [ "$SPEC_LINE_COUNT" -gt 1000 ] || [ "$SPEC_SECTION_COUNT" -gt 20 ]; then
    ANTI_PATTERN_DETECTED="true"
fi

if [ "$ANTI_PATTERN_DETECTED" = "false" ]; then
    echo "  ✅ No significant SDD anti-patterns detected"
    PASSED=$((PASSED + 1))
fi
```

### Step 5: Check Spec-Implementation Alignment (SDD-aligned)

```bash
echo ""
echo "🔗 Checking spec-implementation alignment (SDD: spec as source of truth)..."
echo ""

# Check if implementation exists
IMPLEMENTATION_EXISTS="false"
if [ -d "$IMPLEMENTATION_PATH" ]; then
    if find "$IMPLEMENTATION_PATH" -name "*.md" -o -name "*.js" -o -name "*.py" -o -name "*.ts" 2>/dev/null | head -1 | grep -q .; then
        IMPLEMENTATION_EXISTS="true"
        echo "  ℹ️ Implementation exists: Checking alignment with spec"
    fi
fi

if [ "$IMPLEMENTATION_EXISTS" = "true" ]; then
    # Compare spec acceptance criteria to task acceptance criteria (if tasks exist)
    if [ -f "$TASKS_FILE" ]; then
        SPEC_AC_COUNT=$(grep -c "Acceptance Criteria:" "$SPEC_FILE" 2>/dev/null || echo "0")
        TASKS_AC_COUNT=$(grep -c "Acceptance Criteria:" "$TASKS_FILE" 2>/dev/null || echo "0")
        
        if [ "$SPEC_AC_COUNT" -gt 0 ] && [ "$TASKS_AC_COUNT" -gt 0 ]; then
            # Check alignment (allowing some flexibility - tasks may have more granular AC)
            if [ "$TASKS_AC_COUNT" -ge "$SPEC_AC_COUNT" ]; then
                echo "  ✅ Tasks align with spec: Spec has $SPEC_AC_COUNT acceptance criteria, tasks have $TASKS_AC_COUNT"
                PASSED=$((PASSED + 1))
            else
                echo "  ⚠️ Tasks may not fully align with spec: Spec has $SPEC_AC_COUNT acceptance criteria, tasks have $TASKS_AC_COUNT (SDD: tasks should validate against spec)"
                WARNINGS=$((WARNINGS + 1))
            fi
        elif [ "$SPEC_AC_COUNT" -eq 0 ]; then
            echo "  ⚠️ Spec lacks acceptance criteria: Cannot validate alignment (SDD: spec should have acceptance criteria)"
            WARNINGS=$((WARNINGS + 1))
        else
            echo "  ⚠️ Tasks lack acceptance criteria: Cannot validate alignment (SDD: tasks should reference spec acceptance criteria)"
            WARNINGS=$((WARNINGS + 1))
        fi
    else
        echo "  ⚠️ Tasks file not found: Cannot validate spec-implementation alignment"
        WARNINGS=$((WARNINGS + 1))
    fi
    
    # Check if spec and implementation are in sync (basic check)
    # This is a simplified check - actual drift detection would compare requirements to implementation
    echo "  ℹ️ Basic alignment check: Implementation exists and appears to be in progress"
    echo "     (Full drift detection would require comparing requirements to actual implementation)"
else
    echo "  ℹ️ No implementation found: Skipping alignment check (SDD: validate when implementation exists)"
fi
```

### Step 6: Generate Output and Status

```bash
echo ""
echo "═══════════════════════════════════════════════════════"
echo "  SDD ALIGNMENT VALIDATION RESULTS"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Spec Path: $SPEC_PATH"
echo ""
echo "  Completeness Checks: Passed: $(($PASSED - $ERRORS)), Errors: $ERRORS"
echo "  Structure Checks: Validated"
echo "  Anti-Pattern Checks: Validated"
echo "  Alignment Checks: Validated"
echo ""
echo "  Total Passed: $PASSED"
echo "  Total Errors: $ERRORS"
echo "  Total Warnings: $WARNINGS"
echo ""

# Determine overall status
if [ "$ERRORS" -eq 0 ] && [ "$WARNINGS" -eq 0 ]; then
    echo "  Status: ✅ PASSED - Spec fully aligned with SDD best practices"
    VALIDATION_STATUS="PASSED"
    EXIT_CODE=0
elif [ "$ERRORS" -eq 0 ]; then
    echo "  Status: ⚠️ PASSED WITH WARNINGS - Spec mostly aligned, but has some SDD concerns"
    VALIDATION_STATUS="PASSED_WITH_WARNINGS"
    EXIT_CODE=0  # Warnings don't fail, but should be reviewed
else
    echo "  Status: ❌ FAILED - Spec has SDD alignment issues that should be addressed"
    VALIDATION_STATUS="FAILED"
    EXIT_CODE=1
fi

echo ""
echo "═══════════════════════════════════════════════════════"

# Store result for validation report
mkdir -p "$VALIDATION_PATH"
cat > "$VALIDATION_PATH/sdd-alignment-validation.json" << EOF
{
  "validator": "validate-spec-sdd-alignment",
  "timestamp": $(date +%s),
  "spec_path": "$SPEC_PATH",
  "passed": $PASSED,
  "errors": $ERRORS,
  "warnings": $WARNINGS,
  "status": "$VALIDATION_STATUS",
  "checks": {
    "completeness": {
      "user_stories": $USER_STORIES,
      "acceptance_criteria": $ACCEPTANCE_CRITERIA,
      "scope_boundaries": $SCOPE_BOUNDARIES,
      "requirements": $REQUIREMENTS
    },
    "structure": {
      "premature_tech": $PREMATURE_TECH,
      "line_count": $SPEC_LINE_COUNT,
      "section_count": $SPEC_SECTION_COUNT
    },
    "anti_patterns": {
      "vague_spec": $VAGUE_SPEC,
      "over_specified": $(if [ "$SPEC_LINE_COUNT" -gt 1000 ] || [ "$SPEC_SECTION_COUNT" -gt 20 ]; then echo "true"; else echo "false"; fi)
    },
    "alignment": {
      "implementation_exists": $IMPLEMENTATION_EXISTS,
      "spec_ac_count": $SPEC_AC_COUNT,
      "tasks_ac_count": $TASKS_AC_COUNT
    }
  }
}
EOF

cat > "$VALIDATION_PATH/sdd-alignment-validation.txt" << EOF
VALIDATOR=validate-spec-sdd-alignment
TIMESTAMP=$(date +%s)
PASSED=$PASSED
ERRORS=$ERRORS
WARNINGS=$WARNINGS
STATUS=$VALIDATION_STATUS
EOF

# Export and exit
export SDD_ALIGNMENT_STATUS="$VALIDATION_STATUS"
export SDD_ALIGNMENT_ERRORS="$ERRORS"
export SDD_ALIGNMENT_WARNINGS="$WARNINGS"
exit $EXIT_CODE
```

## Important Constraints

- Must validate spec completeness (user stories, acceptance criteria, scope boundaries, clear requirements)
- Must validate spec structure (What & Why focus, minimal scope, intentional boundaries)
- Must detect SDD anti-patterns (specification theater, premature comprehensiveness, over-specification)
- Must validate spec-implementation alignment when implementation exists
- Must return exit code 0 for pass/warnings, 1 for failure
- Works for any project (project-agnostic)
- **CRITICAL**: This is a core validator that validates SDD alignment for ALL projects

## When to Use This Workflow

This workflow should be called:
1. **After spec creation** (write-spec command) - Validate spec follows SDD best practices
2. **Before task creation** - Ensure spec is complete before creating tasks
3. **Before implementation** - Validate spec alignment before implementation
4. **After implementation** - Check for spec-implementation drift
5. **As part of validation suite** - Regular SDD compliance checks

## Integration with Other Validation Workflows

This workflow integrates with:
- `{{workflows/validation/validate-output-exists}}` - Validates spec file exists
- `{{workflows/validation/validate-knowledge-integration}}` - Can check if spec references basepoints knowledge
- Other validation workflows in the validation ecosystem

## SDD Integration Notes

This workflow validates specs against Spec-Driven Development (SDD) best practices:

**SDD Principles Validated:**
- **Specification as Source of Truth**: Validates that specs are actionable and can be validated
- **SDD Phase Order**: Validates that spec is complete before tasks (Specify → Tasks → Implement)
- **What & Why, not How**: Validates that specs focus on requirements, not implementation details

**SDD Compliance Checks:**
- **Spec Completeness**: User stories format, acceptance criteria, scope boundaries, clear requirements
- **Spec Structure**: What & Why focus, minimal scope, intentional boundaries
- **Anti-Pattern Detection**: Specification theater, premature comprehensiveness, over-specification
- **Spec-Implementation Alignment**: Drift detection when implementation exists

**Technology-Agnostic Approach (Default Profile Templates Only):**
- All SDD validation checks are structure-based, not technology-specific
- No hardcoded technology-specific references in default templates
- Validation maintains technology-agnostic state throughout **in default profile templates**
- **After Specialization:** When templates are compiled to `geist/workflows/`, workflows can and should become technology-specific based on the project's actual stack
- **Command Outputs:** Specs, tasks, and implementations should reflect the project's actual technology stack
