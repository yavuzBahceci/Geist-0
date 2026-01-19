# Validate References

## Core Responsibilities

1. **Find @geist/ References**: Scan files for internal references
2. **Resolve References to Paths**: Convert references to actual file paths
3. **Check Path Existence**: Verify each referenced path exists
4. **Output Broken References**: List any references that don't resolve
5. **Return Status**: Fail if critical references are broken

## Workflow

### Step 1: Define Scope

```bash
# SPEC_PATH should be set by the calling workflow
if [ -z "$SPEC_PATH" ]; then
    echo "❌ SPEC_PATH not set"
    exit 1
fi

echo "🔍 Validating references..."

# Files to check for references
FILES_TO_CHECK=""

# Add spec files
[ -f "$SPEC_PATH/spec.md" ] && FILES_TO_CHECK="$FILES_TO_CHECK $SPEC_PATH/spec.md"
[ -f "$SPEC_PATH/tasks.md" ] && FILES_TO_CHECK="$FILES_TO_CHECK $SPEC_PATH/tasks.md"
[ -f "$SPEC_PATH/planning/requirements.md" ] && FILES_TO_CHECK="$FILES_TO_CHECK $SPEC_PATH/planning/requirements.md"
[ -f "$SPEC_PATH/planning/architecture.md" ] && FILES_TO_CHECK="$FILES_TO_CHECK $SPEC_PATH/planning/architecture.md"

FILE_COUNT=$(echo "$FILES_TO_CHECK" | wc -w | tr -d ' ')
echo "   Files to check: $FILE_COUNT"
```

### Step 2: Find @geist/ References

```bash
echo ""
echo "📋 Finding @geist/ references..."
echo ""

ALL_REFERENCES=""
REFERENCE_COUNT=0

for file in $FILES_TO_CHECK; do
    if [ -f "$file" ]; then
        # Find @geist/ references
        REFS=$(grep -oE '@geist/[^[:space:]]+' "$file" 2>/dev/null | sort -u)
        
        if [ -n "$REFS" ]; then
            REF_COUNT=$(echo "$REFS" | wc -l | tr -d ' ')
            echo "  Found $REF_COUNT references in $(basename "$file")"
            ALL_REFERENCES="$ALL_REFERENCES
$REFS"
            REFERENCE_COUNT=$((REFERENCE_COUNT + REF_COUNT))
        fi
    fi
done

# Also check for standard markdown links
for file in $FILES_TO_CHECK; do
    if [ -f "$file" ]; then
        # Find [text](path) style links to internal files
        LINK_REFS=$(grep -oE '\]\([^)]+\.md\)' "$file" 2>/dev/null | sed 's/\](\(.*\))/\1/' | sort -u)
        
        if [ -n "$LINK_REFS" ]; then
            LINK_COUNT=$(echo "$LINK_REFS" | wc -l | tr -d ' ')
            echo "  Found $LINK_COUNT markdown links in $(basename "$file")"
        fi
    fi
done

ALL_REFERENCES=$(echo "$ALL_REFERENCES" | grep -v '^$' | sort -u)
echo ""
echo "   Total unique references: $REFERENCE_COUNT"
```

### Step 3: Resolve and Check References

```bash
echo ""
echo "🔗 Checking reference resolution..."
echo ""

BROKEN_REFS=""
BROKEN_COUNT=0
VALID_COUNT=0

for ref in $ALL_REFERENCES; do
    # Skip empty lines
    [ -z "$ref" ] && continue
    
    # Convert @geist/ to actual path
    ACTUAL_PATH=$(echo "$ref" | sed 's/@//')
    
    # Check if path exists
    if [ -f "$ACTUAL_PATH" ] || [ -d "$ACTUAL_PATH" ]; then
        echo "  ✅ $ref"
        VALID_COUNT=$((VALID_COUNT + 1))
    else
        echo "  ❌ $ref (not found: $ACTUAL_PATH)"
        BROKEN_REFS="$BROKEN_REFS
$ref"
        BROKEN_COUNT=$((BROKEN_COUNT + 1))
    fi
done
```

### Step 4: Classify Broken References

```bash
echo ""
echo "📊 Classifying broken references..."
echo ""

CRITICAL_BROKEN=0
WARNING_BROKEN=0

for ref in $BROKEN_REFS; do
    [ -z "$ref" ] && continue
    
    # Critical: references to required files (spec, tasks, workflows)
    if echo "$ref" | grep -qE 'spec\.md|tasks\.md|workflows/|commands/'; then
        echo "  ⛔ CRITICAL: $ref"
        CRITICAL_BROKEN=$((CRITICAL_BROKEN + 1))
    else
        echo "  ⚠️ WARNING: $ref"
        WARNING_BROKEN=$((WARNING_BROKEN + 1))
    fi
done

if [ "$BROKEN_COUNT" -eq 0 ]; then
    echo "  No broken references found"
fi
```

### Step 5: Generate Output and Exit Code

```bash
echo ""
echo "═══════════════════════════════════════════════════════"
echo "  REFERENCE VALIDATION RESULTS"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Spec Path: $SPEC_PATH"
echo ""
echo "  Total References: $REFERENCE_COUNT"
echo "  Valid: $VALID_COUNT"
echo "  Broken: $BROKEN_COUNT"
echo "    - Critical: $CRITICAL_BROKEN"
echo "    - Warnings: $WARNING_BROKEN"
echo ""

# Fail only on critical broken references
if [ "$CRITICAL_BROKEN" -gt 0 ]; then
    echo "  Status: ❌ FAILED ($CRITICAL_BROKEN critical broken references)"
    VALIDATION_STATUS="FAILED"
    EXIT_CODE=1
elif [ "$WARNING_BROKEN" -gt 0 ]; then
    echo "  Status: ⚠️ PASSED WITH WARNINGS ($WARNING_BROKEN broken references)"
    VALIDATION_STATUS="PASSED_WITH_WARNINGS"
    EXIT_CODE=0
else
    echo "  Status: ✅ PASSED"
    VALIDATION_STATUS="PASSED"
    EXIT_CODE=0
fi

echo ""
echo "═══════════════════════════════════════════════════════"

# Store result for validation report
mkdir -p "$SPEC_PATH/implementation/cache"
cat > "$SPEC_PATH/implementation/cache/references-validation.txt" << EOF
VALIDATOR=references
TIMESTAMP=$(date +%s)
TOTAL_REFS=$REFERENCE_COUNT
VALID=$VALID_COUNT
BROKEN=$BROKEN_COUNT
CRITICAL_BROKEN=$CRITICAL_BROKEN
WARNING_BROKEN=$WARNING_BROKEN
STATUS=$VALIDATION_STATUS
EOF

# Export and exit
export REFERENCES_STATUS="$VALIDATION_STATUS"
export REFERENCES_BROKEN="$BROKEN_COUNT"
exit $EXIT_CODE
```

## Important Constraints

- Must find all @geist/ references
- Must resolve references to actual paths
- Must classify broken references by severity
- Must fail only on critical broken references
- Works for any project (project-agnostic)
- **CRITICAL**: This is a core validator that runs for ALL projects
