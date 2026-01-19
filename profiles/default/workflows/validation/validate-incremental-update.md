# Validate Incremental Update

## Core Responsibilities

1. **Validate Updated Basepoint Files**: Check markdown structure and required sections
2. **Validate Command Files**: Ensure specialized command files exist and are valid
3. **Check for Broken References**: Detect broken `{{workflows/...}}` and `{{standards/...}}` references
4. **Verify Knowledge Cache Consistency**: Ensure cache files are consistent with basepoints
5. **Generate Validation Report**: Create comprehensive validation report with issues found

## Workflow

### Step 1: Initialize Validation Environment

Set up validation environment and load update context:

```bash
CACHE_DIR="geist/output/update-basepoints-and-redeploy/cache"
REPORTS_DIR="geist/output/update-basepoints-and-redeploy/reports"

mkdir -p "$REPORTS_DIR"

# Load update context
if [ ! -f "$CACHE_DIR/update-log.md" ]; then
    echo "⚠️  Warning: Update log not found. Validating all basepoints."
    VALIDATION_SCOPE="all"
else
    VALIDATION_SCOPE="incremental"
    UPDATED_BASEPOINTS=$(cat "$CACHE_DIR/update-progress.md" 2>/dev/null | grep "\[x\]" | sed 's/.*`\([^`]*\)`.*/\1/')
fi

# Initialize validation results
VALIDATION_ISSUES=()
VALIDATION_WARNINGS=()
VALIDATION_PASSED=0
VALIDATION_FAILED=0

echo "🔍 Starting incremental update validation..."
echo "   Scope: $VALIDATION_SCOPE"
```

### Step 2: Validate Updated Basepoint Files

Check each updated basepoint for proper markdown structure:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📄 VALIDATING BASEPOINT FILES"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Required sections for basepoint files
REQUIRED_SECTIONS=(
    "Module Overview"
    "Patterns"
    "Standards"
    "Flows"
    "Strategies"
)

# Determine files to validate
if [ "$VALIDATION_SCOPE" = "incremental" ]; then
    FILES_TO_VALIDATE="$UPDATED_BASEPOINTS"
else
    FILES_TO_VALIDATE=$(find geist/basepoints -name "agent-base-*.md" -type f)
fi

echo "$FILES_TO_VALIDATE" | while read basepoint_file; do
    if [ -z "$basepoint_file" ] || [ ! -f "$basepoint_file" ]; then
        continue
    fi
    
    echo "   Validating: $basepoint_file"
    FILE_VALID=true
    
    # Check file is not empty
    if [ ! -s "$basepoint_file" ]; then
        VALIDATION_ISSUES+=("EMPTY_FILE:$basepoint_file:Basepoint file is empty")
        FILE_VALID=false
        continue
    fi
    
    # Check for required sections
    FILE_CONTENT=$(cat "$basepoint_file")
    
    for section in "${REQUIRED_SECTIONS[@]}"; do
        if ! echo "$FILE_CONTENT" | grep -q "## $section\|# $section"; then
            VALIDATION_WARNINGS+=("MISSING_SECTION:$basepoint_file:Missing section: $section")
        fi
    done
    
    # Check for valid markdown (no broken headers, no unclosed code blocks)
    # Count opening and closing code blocks
    OPEN_BLOCKS=$(echo "$FILE_CONTENT" | grep -c '```' || echo "0")
    if [ $((OPEN_BLOCKS % 2)) -ne 0 ]; then
        VALIDATION_ISSUES+=("UNCLOSED_CODE_BLOCK:$basepoint_file:Unclosed code block detected")
        FILE_VALID=false
    fi
    
    # Check for placeholder remnants that shouldn't be there
    if echo "$FILE_CONTENT" | grep -qE '\{\{[A-Z_]+\}\}'; then
        # Exclude intentional documentation placeholders
        PLACEHOLDERS=$(echo "$FILE_CONTENT" | grep -oE '\{\{[A-Z_]+\}\}' | sort -u)
        for placeholder in $PLACEHOLDERS; do
            if [[ "$placeholder" != "{{PLACEHOLDER}}" ]] && [[ "$placeholder" != "{{UNLESS}}" ]]; then
                VALIDATION_WARNINGS+=("UNRESOLVED_PLACEHOLDER:$basepoint_file:Unresolved placeholder: $placeholder")
            fi
        done
    fi
    
    if [ "$FILE_VALID" = "true" ]; then
        VALIDATION_PASSED=$((VALIDATION_PASSED + 1))
        echo "      ✅ Valid"
    else
        VALIDATION_FAILED=$((VALIDATION_FAILED + 1))
        echo "      ❌ Issues found"
    fi
done
```

### Step 3: Validate Command Files

Check that specialized command files are valid:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📜 VALIDATING COMMAND FILES"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Find all command files
COMMAND_FILES=$(find geist/commands -name "*.md" -type f 2>/dev/null)

if [ -z "$COMMAND_FILES" ]; then
    echo "   ⚠️  No command files found in geist/commands/"
    VALIDATION_WARNINGS+=("NO_COMMANDS:geist/commands/:No command files found")
else
    echo "$COMMAND_FILES" | while read command_file; do
        if [ -z "$command_file" ]; then
            continue
        fi
        
        echo "   Validating: $command_file"
        
        # Check file is not empty
        if [ ! -s "$command_file" ]; then
            VALIDATION_ISSUES+=("EMPTY_COMMAND:$command_file:Command file is empty")
            echo "      ❌ Empty file"
            continue
        fi
        
        # Check for valid phase references
        FILE_CONTENT=$(cat "$command_file")
        
        # Check for broken phase references
        PHASE_REFS=$(echo "$FILE_CONTENT" | grep -oE '\{\{PHASE [0-9]+: @geist/commands/[^}]+\}\}' || true)
        
        if [ -n "$PHASE_REFS" ]; then
            echo "$PHASE_REFS" | while read phase_ref; do
                # Extract the referenced file path
                REF_PATH=$(echo "$phase_ref" | sed 's/.*@geist\/commands\/\([^}]*\)\.md.*/\1.md/')
                FULL_PATH="geist/commands/$REF_PATH"
                
                if [ ! -f "$FULL_PATH" ]; then
                    VALIDATION_ISSUES+=("BROKEN_PHASE_REF:$command_file:Broken phase reference: $REF_PATH")
                    echo "      ❌ Broken phase ref: $REF_PATH"
                fi
            done
        fi
        
        echo "      ✅ Valid"
    done
fi
```

### Step 4: Check for Broken Workflow and Standard References

Scan for broken `{{workflows/...}}` and `{{standards/...}}` references:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "🔗 CHECKING REFERENCES"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Find all markdown files in geist
ALL_FILES=$(find geist -name "*.md" -type f ! -path "*/output/*" ! -path "*/specs/*")

echo "$ALL_FILES" | while read file; do
    if [ -z "$file" ] || [ ! -f "$file" ]; then
        continue
    fi
    
    FILE_CONTENT=$(cat "$file")
    
    # Check workflow references
    WORKFLOW_REFS=$(echo "$FILE_CONTENT" | grep -oE '\{\{workflows/[^}]+\}\}' || true)
    
    if [ -n "$WORKFLOW_REFS" ]; then
        echo "$WORKFLOW_REFS" | while read ref; do
            # Extract workflow path
            WORKFLOW_PATH=$(echo "$ref" | sed 's/{{workflows\/\([^}]*\)}}/\1/')
            FULL_PATH="geist/workflows/${WORKFLOW_PATH}.md"
            
            if [ ! -f "$FULL_PATH" ]; then
                VALIDATION_ISSUES+=("BROKEN_WORKFLOW_REF:$file:Broken workflow reference: $WORKFLOW_PATH")
                echo "   ❌ Broken workflow ref in $file: $WORKFLOW_PATH"
            fi
        done
    fi
    
    # Check standard references
    STANDARD_REFS=$(echo "$FILE_CONTENT" | grep -oE '\{\{standards/[^}]+\}\}' || true)
    
    if [ -n "$STANDARD_REFS" ]; then
        echo "$STANDARD_REFS" | while read ref; do
            # Extract standard path
            STANDARD_PATH=$(echo "$ref" | sed 's/{{standards\/\([^}]*\)}}/\1/')
            FULL_PATH="geist/standards/${STANDARD_PATH}.md"
            
            if [ ! -f "$FULL_PATH" ]; then
                VALIDATION_ISSUES+=("BROKEN_STANDARD_REF:$file:Broken standard reference: $STANDARD_PATH")
                echo "   ❌ Broken standard ref in $file: $STANDARD_PATH"
            fi
        done
    fi
done

echo "   ✅ Reference check complete"
```

### Step 5: Verify Knowledge Cache Consistency

Check that knowledge cache is consistent with basepoints:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "💾 VERIFYING KNOWLEDGE CACHE"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

KNOWLEDGE_CACHE="geist/output/deploy-agents/cache/merged-knowledge.md"

if [ -f "$KNOWLEDGE_CACHE" ]; then
    echo "   📂 Knowledge cache found: $KNOWLEDGE_CACHE"
    
    # Check cache is not empty
    if [ ! -s "$KNOWLEDGE_CACHE" ]; then
        VALIDATION_ISSUES+=("EMPTY_CACHE:$KNOWLEDGE_CACHE:Knowledge cache is empty")
        echo "   ❌ Knowledge cache is empty"
    else
        # Check cache timestamp vs basepoints
        CACHE_MTIME=$(stat -f %m "$KNOWLEDGE_CACHE" 2>/dev/null || stat -c %Y "$KNOWLEDGE_CACHE" 2>/dev/null)
        
        # Find newest basepoint
        NEWEST_BASEPOINT=$(find geist/basepoints -name "*.md" -type f -exec stat -f '%m %N' {} \; 2>/dev/null | sort -rn | head -1 | cut -d' ' -f2)
        NEWEST_MTIME=$(stat -f %m "$NEWEST_BASEPOINT" 2>/dev/null || stat -c %Y "$NEWEST_BASEPOINT" 2>/dev/null)
        
        if [ "$CACHE_MTIME" -lt "$NEWEST_MTIME" ]; then
            VALIDATION_WARNINGS+=("STALE_CACHE:$KNOWLEDGE_CACHE:Knowledge cache is older than newest basepoint")
            echo "   ⚠️  Knowledge cache may be stale"
        else
            echo "   ✅ Knowledge cache is up to date"
        fi
    fi
else
    VALIDATION_WARNINGS+=("NO_CACHE:$KNOWLEDGE_CACHE:Knowledge cache not found")
    echo "   ⚠️  Knowledge cache not found (expected after deploy-agents)"
fi
```

### Step 6: Generate Validation Report

Create comprehensive validation report:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📋 GENERATING VALIDATION REPORT"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

ISSUES_COUNT=${#VALIDATION_ISSUES[@]}
WARNINGS_COUNT=${#VALIDATION_WARNINGS[@]}

# Determine overall status
if [ "$ISSUES_COUNT" -eq 0 ]; then
    OVERALL_STATUS="PASSED"
    STATUS_EMOJI="✅"
else
    OVERALL_STATUS="FAILED"
    STATUS_EMOJI="❌"
fi

# Generate report
cat > "$REPORTS_DIR/validation-report.md" << EOF
# Incremental Update Validation Report

**Validation Time:** $(date -u +%Y-%m-%dT%H:%M:%SZ)
**Validation Scope:** $VALIDATION_SCOPE
**Overall Status:** $STATUS_EMOJI $OVERALL_STATUS

## Summary

| Metric | Count |
|--------|-------|
| Issues (blocking) | $ISSUES_COUNT |
| Warnings (non-blocking) | $WARNINGS_COUNT |
| Files passed | $VALIDATION_PASSED |
| Files failed | $VALIDATION_FAILED |

## Issues (Must Fix)

$(if [ "$ISSUES_COUNT" -gt 0 ]; then
    for issue in "${VALIDATION_ISSUES[@]}"; do
        TYPE=$(echo "$issue" | cut -d: -f1)
        FILE=$(echo "$issue" | cut -d: -f2)
        DESC=$(echo "$issue" | cut -d: -f3-)
        echo "### $TYPE"
        echo "- **File:** \`$FILE\`"
        echo "- **Description:** $DESC"
        echo ""
    done
else
    echo "_No blocking issues found_"
fi)

## Warnings (Should Review)

$(if [ "$WARNINGS_COUNT" -gt 0 ]; then
    for warning in "${VALIDATION_WARNINGS[@]}"; do
        TYPE=$(echo "$warning" | cut -d: -f1)
        FILE=$(echo "$warning" | cut -d: -f2)
        DESC=$(echo "$warning" | cut -d: -f3-)
        echo "- **$TYPE** in \`$FILE\`: $DESC"
    done
else
    echo "_No warnings_"
fi)

## Validation Details

### Basepoint Files Checked
$(if [ "$VALIDATION_SCOPE" = "incremental" ]; then
    echo "$UPDATED_BASEPOINTS" | sed 's/^/- /'
else
    find geist/basepoints -name "agent-base-*.md" -type f | sed 's/^/- /'
fi)

### Command Files Checked
$(find geist/commands -name "*.md" -type f 2>/dev/null | sed 's/^/- /' || echo "_None found_")

### Reference Checks
- Workflow references: Checked
- Standard references: Checked
- Phase references: Checked

### Cache Validation
- Knowledge cache: $([ -f "$KNOWLEDGE_CACHE" ] && echo "Present" || echo "Not found")
- Cache freshness: $([ "$CACHE_MTIME" -ge "$NEWEST_MTIME" ] && echo "Up to date" || echo "May be stale")

## Recommendations

$(if [ "$ISSUES_COUNT" -gt 0 ]; then
    echo "1. Fix all blocking issues before proceeding"
    echo "2. Re-run validation after fixes"
else
    echo "1. Review any warnings listed above"
    echo "2. Incremental update is ready for use"
fi)
EOF

echo "📋 Validation report saved to: $REPORTS_DIR/validation-report.md"
```

### Step 7: Output Validation Results

Display validation results for user:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[✅/❌] VALIDATION [PASSED/FAILED]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 Results:
   ❌ Issues (blocking):     [N]
   ⚠️  Warnings (review):    [N]
   ✅ Files passed:          [N]
   ❌ Files failed:          [N]

[If issues found:]
❌ BLOCKING ISSUES:
   - [Issue type]: [file] - [description]
   ...

[If warnings found:]
⚠️  WARNINGS:
   - [Warning type]: [file] - [description]
   ...

📋 Full report: geist/output/update-basepoints-and-redeploy/reports/validation-report.md

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[If passed:]
✅ Incremental update validated successfully!
   Update tracking files have been saved.

[If failed:]
❌ Please fix the issues above and re-run validation.
```

## Important Constraints

- **MUST check all updated basepoints** for valid markdown structure
- **MUST check command files** for valid phase references
- **MUST detect broken references** to workflows and standards
- **MUST verify knowledge cache** consistency
- **MUST generate comprehensive report** with all findings
- **MUST distinguish between blocking issues and warnings**
- Must not modify any files during validation (read-only)
- Must validate in incremental scope when update log exists
- Must provide actionable recommendations in report
