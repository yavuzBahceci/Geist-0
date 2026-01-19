# Unnecessary Logic Removal Validation

## Core Responsibilities

1. **Comprehensive Unnecessary Logic Detection**: Detect all types of unnecessary logic that should be removed from specialized commands
2. **Categorize by Type**: Group findings by category (conditionals, examples, patterns, references)
3. **Generate Removal Report**: Create detailed report with file locations and recommendations for removal
4. **Support Validation Context**: Support validation of both profiles/default (template) and installed geist (specialized)

## Workflow

### Step 1: Determine Context and Initialize

Determine validation context and initialize paths:

```bash
# Determine spec path
SPEC_PATH="{{SPEC_PATH}}"

# Determine validation context
if [ -d "profiles/default" ] && [ "$(pwd)" = *"/profiles/default"* ]; then
    VALIDATION_CONTEXT="profiles/default"
    SCAN_PATH="profiles/default"
    CACHE_PATH="$SPEC_PATH/implementation/cache/validation"
else
    VALIDATION_CONTEXT="installed-geist"
    SCAN_PATH="geist"
    CACHE_PATH="$SPEC_PATH/implementation/cache/validation"
fi

# Create cache directory
mkdir -p "$CACHE_PATH"

echo "🔍 Starting unnecessary logic removal validation for: $VALIDATION_CONTEXT"
```

### Step 2: Use Unnecessary Logic Detection Utility

Use the unnecessary logic detection utility from Task Group 1:

```bash
# Run unnecessary logic detection utility
{{workflows/validation/detect-unnecessary-logic}}

# Load detection results
if [ -f "$CACHE_PATH/unnecessary-logic-detection.json" ]; then
    UNNECESSARY_LOGIC_DATA=$(cat "$CACHE_PATH/unnecessary-logic-detection.json")
else
    echo "❌ Unnecessary logic detection failed - no results found"
    exit 1
fi
```

### Step 3: Detect Project-Agnostic Conditional Logic

Extract and categorize project-agnostic conditional logic:

```bash
# Initialize results
PROJECT_AGNOSTIC_CONDITIONALS=""

# Find files to scan
FILES_TO_SCAN=$(find "$SCAN_PATH" -type f \( -name "*.md" \) ! -path "*/node_modules/*" ! -path "*/.git/*" ! -path "*/build/*" ! -path "*/dist/*" ! -path "*/cache/*" ! -path "*/output/deploy-agents/*" ! -path "*/.cleanup-cache/*")

# Scan for project-agnostic conditionals
echo "$FILES_TO_SCAN" | while read file_path; do
    if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
        continue
    fi
    
    FILE_CONTENT=$(cat "$file_path")
    
    # Detect technology-agnostic fallback logic
    if echo "$FILE_CONTENT" | grep -qE "if.*technology.*agnostic|if.*project.*agnostic|fallback.*generic|default.*generic"; then
        LINE_NUMBERS=$(grep -nE "if.*technology.*agnostic|if.*project.*agnostic|fallback.*generic|default.*generic" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
        PROJECT_AGNOSTIC_CONDITIONALS="${PROJECT_AGNOSTIC_CONDITIONALS}\nconditional:$file_path:$LINE_NUMBERS:Technology-agnostic fallback logic"
    fi
    
    # Detect conditional logic checking for template state
    if echo "$FILE_CONTENT" | grep -qE "if.*template|if.*abstract|if.*generic.*pattern"; then
        LINE_NUMBERS=$(grep -nE "if.*template|if.*abstract|if.*generic.*pattern" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
        PROJECT_AGNOSTIC_CONDITIONALS="${PROJECT_AGNOSTIC_CONDITIONALS}\nconditional:$file_path:$LINE_NUMBERS:Template/abstract pattern check"
    fi
    
    # Detect conditional logic that checks for project-agnostic scenarios
    if echo "$FILE_CONTENT" | grep -qE "if.*not.*specialized|if.*not.*deployed|if.*before.*deployment|if.*after.*deployment.*not"; then
        LINE_NUMBERS=$(grep -nE "if.*not.*specialized|if.*not.*deployed|if.*before.*deployment|if.*after.*deployment.*not" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
        PROJECT_AGNOSTIC_CONDITIONALS="${PROJECT_AGNOSTIC_CONDITIONALS}\nconditional:$file_path:$LINE_NUMBERS:Project-agnostic scenario check"
    fi
    
    # Detect fallback logic that should be removed after specialization
    if echo "$FILE_CONTENT" | grep -qE "else.*generic|else.*default|else.*fallback|else.*template"; then
        LINE_NUMBERS=$(grep -nE "else.*generic|else.*default|else.*fallback|else.*template" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
        PROJECT_AGNOSTIC_CONDITIONALS="${PROJECT_AGNOSTIC_CONDITIONALS}\nconditional:$file_path:$LINE_NUMBERS:Fallback logic that should be removed"
    fi
done

# Count project-agnostic conditionals
CONDITIONALS_COUNT=$(echo "$PROJECT_AGNOSTIC_CONDITIONALS" | grep -c . || echo "0")
```

### Step 4: Detect Generic Examples and Abstract Patterns

Extract and categorize generic examples and abstract patterns:

```bash
# Initialize results
GENERIC_EXAMPLES=""
ABSTRACT_PATTERNS=""

# Scan for generic examples
echo "$FILES_TO_SCAN" | while read file_path; do
    if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
        continue
    fi
    
    FILE_CONTENT=$(cat "$file_path")
    
    # Detect generic example patterns
    if echo "$FILE_CONTENT" | grep -qE "Example:.*[Uu]ser.*model|Example:.*generic|Example:.*sample|mock.*data|placeholder.*example|example.*user.*model|example.*generic.*module"; then
        LINE_NUMBERS=$(grep -nE "Example:.*[Uu]ser.*model|Example:.*generic|Example:.*sample|mock.*data|placeholder.*example|example.*user.*model|example.*generic.*module" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
        GENERIC_EXAMPLES="${GENERIC_EXAMPLES}\nexample:$file_path:$LINE_NUMBERS:Generic example pattern"
    fi
    
    # Detect placeholder examples or mock data
    if echo "$FILE_CONTENT" | grep -qE "placeholder.*data|mock.*example|sample.*data|dummy.*data|test.*data.*example"; then
        LINE_NUMBERS=$(grep -nE "placeholder.*data|mock.*example|sample.*data|dummy.*data|test.*data.*example" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
        GENERIC_EXAMPLES="${GENERIC_EXAMPLES}\nexample:$file_path:$LINE_NUMBERS:Placeholder example or mock data"
    fi
    
    # Detect abstract pattern references
    if echo "$FILE_CONTENT" | grep -qE "abstract.*pattern|generic.*pattern|template.*pattern|placeholder.*pattern"; then
        LINE_NUMBERS=$(grep -nE "abstract.*pattern|generic.*pattern|template.*pattern|placeholder.*pattern" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
        ABSTRACT_PATTERNS="${ABSTRACT_PATTERNS}\npattern:$file_path:$LINE_NUMBERS:Abstract pattern reference"
    fi
    
    # Detect patterns that should be replaced with concrete project patterns
    if echo "$FILE_CONTENT" | grep -qE "use.*generic.*pattern|apply.*abstract.*pattern|follow.*template.*pattern"; then
        LINE_NUMBERS=$(grep -nE "use.*generic.*pattern|apply.*abstract.*pattern|follow.*template.*pattern" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
        ABSTRACT_PATTERNS="${ABSTRACT_PATTERNS}\npattern:$file_path:$LINE_NUMBERS:Should use concrete project pattern instead"
    fi
done

# Count generic examples and abstract patterns
EXAMPLES_COUNT=$(echo "$GENERIC_EXAMPLES" | grep -c . || echo "0")
PATTERNS_COUNT=$(echo "$ABSTRACT_PATTERNS" | grep -c . || echo "0")
```

### Step 5: Detect profiles/default References in Specialized Commands

Find any references to "profiles/default" in specialized commands (should only exist in template):

```bash
# Initialize results
PROFILES_SELF_REFERENCES=""

# Only check if validating installed geist (specialized commands shouldn't reference profiles/default)
if [ "$VALIDATION_CONTEXT" = "installed-geist" ]; then
    echo "$FILES_TO_SCAN" | while read file_path; do
        if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
            continue
        fi
        
        FILE_CONTENT=$(cat "$file_path")
        
        # Detect profiles/default references
        if echo "$FILE_CONTENT" | grep -qE "profiles/default|profiles\.self|@profiles/default|profiles/default/"; then
            LINE_NUMBERS=$(grep -nE "profiles/default|profiles\.self|@profiles/default|profiles/default/" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
            PROFILES_SELF_REFERENCES="${PROFILES_SELF_REFERENCES}\nreference:$file_path:$LINE_NUMBERS:profiles/default reference in specialized command"
        fi
        
        # Detect template path references
        if echo "$FILE_CONTENT" | grep -qE "template.*path|abstract.*path|generic.*path"; then
            LINE_NUMBERS=$(grep -nE "template.*path|abstract.*path|generic.*path" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
            PROFILES_SELF_REFERENCES="${PROFILES_SELF_REFERENCES}\nreference:$file_path:$LINE_NUMBERS:Template path reference (should be actual project path)"
        fi
        
        # Detect logic that checks for "profiles/default" after deployment
        if echo "$FILE_CONTENT" | grep -qE "if.*profiles/default|check.*profiles/default|detect.*profiles/default"; then
            LINE_NUMBERS=$(grep -nE "if.*profiles/default|check.*profiles/default|detect.*profiles/default" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
            PROFILES_SELF_REFERENCES="${PROFILES_SELF_REFERENCES}\nreference:$file_path:$LINE_NUMBERS:Logic checking for profiles/default after deployment"
        fi
    done
fi

# Count profiles/default references
REFERENCES_COUNT=$(echo "$PROFILES_SELF_REFERENCES" | grep -c . || echo "0")
```

### Step 6: Generate Unnecessary Logic Validation Report

Generate comprehensive unnecessary logic removal validation report:

```bash
# Calculate totals
TOTAL_ISSUES=$((CONDITIONALS_COUNT + EXAMPLES_COUNT + PATTERNS_COUNT + REFERENCES_COUNT))

# Create JSON report
cat > "$CACHE_PATH/unnecessary-logic-removal-validation.json" << EOF
{
  "validation_context": "$VALIDATION_CONTEXT",
  "scan_path": "$SCAN_PATH",
  "validation_timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "total_issues_found": $TOTAL_ISSUES,
  "categories": {
    "project_agnostic_conditionals": {
      "count": $CONDITIONALS_COUNT,
      "issues": [
$(echo "$PROJECT_AGNOSTIC_CONDITIONALS" | grep -v "^$" | while IFS=':' read -r type file_path line_numbers description; do
    echo "        {\"type\": \"$type\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ],
      "recommendation": "Remove conditional logic that checks for project-agnostic scenarios. After specialization, these checks are no longer needed."
    },
    "generic_examples": {
      "count": $EXAMPLES_COUNT,
      "issues": [
$(echo "$GENERIC_EXAMPLES" | grep -v "^$" | while IFS=':' read -r type file_path line_numbers description; do
    echo "        {\"type\": \"$type\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ],
      "recommendation": "Replace generic examples with project-specific ones from basepoints or actual project structure."
    },
    "abstract_patterns": {
      "count": $PATTERNS_COUNT,
      "issues": [
$(echo "$ABSTRACT_PATTERNS" | grep -v "^$" | while IFS=':' read -r type file_path line_numbers description; do
    echo "        {\"type\": \"$type\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ],
      "recommendation": "Replace abstract pattern references with concrete project patterns from basepoints."
    },
    "profiles_self_references": {
      "count": $REFERENCES_COUNT,
      "issues": [
$(echo "$PROFILES_SELF_REFERENCES" | grep -v "^$" | while IFS=':' read -r type file_path line_numbers description; do
    echo "        {\"type\": \"$type\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ],
      "recommendation": "Remove all references to 'profiles/default' from specialized commands. Specialized commands should not reference the template."
    }
  },
  "removal_status": "$(if [ "$TOTAL_ISSUES" -eq 0 ]; then echo "clean"; else echo "needs_removal"; fi)"
}
EOF

# Create markdown summary report
cat > "$CACHE_PATH/unnecessary-logic-removal-validation-summary.md" << EOF
# Unnecessary Logic Removal Validation Report

**Validation Context:** $VALIDATION_CONTEXT  
**Scan Path:** $SCAN_PATH  
**Validation Timestamp:** $(date -u +%Y-%m-%dT%H:%M:%SZ)

## Summary

- **Total Issues Found:** $TOTAL_ISSUES
- **Project-Agnostic Conditionals:** $CONDITIONALS_COUNT
- **Generic Examples:** $EXAMPLES_COUNT
- **Abstract Patterns:** $PATTERNS_COUNT
- **profiles/default References:** $REFERENCES_COUNT
- **Removal Status:** $(if [ "$TOTAL_ISSUES" -eq 0 ]; then echo "✅ Clean - No unnecessary logic found"; else echo "⚠️  Needs Removal - $TOTAL_ISSUES issue(s) need to be removed"; fi)

## Project-Agnostic Conditional Logic

$(if [ "$CONDITIONALS_COUNT" -gt 0 ]; then
    echo "$PROJECT_AGNOSTIC_CONDITIONALS" | grep -v "^$" | while IFS=':' read -r type file_path line_numbers description; do
        echo "- **$description**"
        echo "  - File: \`$file_path\`"
        echo "  - Lines: $line_numbers"
        echo "  - Recommendation: Remove conditional logic that checks for project-agnostic scenarios"
        echo ""
    done
else
    echo "✅ No project-agnostic conditional logic found"
fi)

## Generic Examples

$(if [ "$EXAMPLES_COUNT" -gt 0 ]; then
    echo "$GENERIC_EXAMPLES" | grep -v "^$" | while IFS=':' read -r type file_path line_numbers description; do
        echo "- **$description**"
        echo "  - File: \`$file_path\`"
        echo "  - Lines: $line_numbers"
        echo "  - Recommendation: Replace with project-specific examples from basepoints"
        echo ""
    done
else
    echo "✅ No generic examples found"
fi)

## Abstract Patterns

$(if [ "$PATTERNS_COUNT" -gt 0 ]; then
    echo "$ABSTRACT_PATTERNS" | grep -v "^$" | while IFS=':' read -r type file_path line_numbers description; do
        echo "- **$description**"
        echo "  - File: \`$file_path\`"
        echo "  - Lines: $line_numbers"
        echo "  - Recommendation: Replace with concrete project patterns from basepoints"
        echo ""
    done
else
    echo "✅ No abstract patterns found"
fi)

## profiles/default References

$(if [ "$REFERENCES_COUNT" -gt 0 ]; then
    echo "$PROFILES_SELF_REFERENCES" | grep -v "^$" | while IFS=':' read -r type file_path line_numbers description; do
        echo "- **$description**"
        echo "  - File: \`$file_path\`"
        echo "  - Lines: $line_numbers"
        echo "  - Recommendation: Remove all references to 'profiles/default' from specialized commands"
        echo ""
    done
else
    if [ "$VALIDATION_CONTEXT" = "installed-geist" ]; then
        echo "✅ No profiles/default references found in specialized commands"
    else
        echo "ℹ️  profiles/default references are expected in template (profiles/default context)"
    fi
fi)

## Recommendations

$(if [ "$TOTAL_ISSUES" -gt 0 ]; then
    echo "### Action Required"
    echo ""
    echo "The following actions are recommended to remove unnecessary logic:"
    echo ""
    echo "1. **Remove Project-Agnostic Conditionals**: Delete conditional logic that checks for project-agnostic scenarios"
    echo "2. **Replace Generic Examples**: Replace generic examples with project-specific ones from basepoints"
    echo "3. **Replace Abstract Patterns**: Replace abstract pattern references with concrete project patterns"
    echo "4. **Remove profiles/default References**: Remove all references to 'profiles/default' from specialized commands"
    echo "5. **Run cleanup command** (if available) to automatically fix these issues in already-deployed geist"
    echo "6. **Manually review** files listed above and remove unnecessary logic"
else
    echo "✅ **No Action Required** - All unnecessary logic has been removed!"
fi)
EOF

echo "✅ Unnecessary logic removal validation complete!"
echo "📊 Total issues found: $TOTAL_ISSUES"
echo "📁 Report stored in: $CACHE_PATH/unnecessary-logic-removal-validation-summary.md"
```

## Important Constraints

- Must detect all types of unnecessary logic (conditionals, examples, patterns, references)
- Must categorize findings by type for easy analysis
- Must generate comprehensive reports with file locations and recommendations
- Must only check for profiles/default references when validating installed geist (not in template)
- Must provide specific recommendations for removal of each type
- Must support validation of both profiles/default (template) and installed geist (specialized)
- **CRITICAL**: All reports must be stored in `geist/specs/[current-spec]/implementation/cache/validation/` when running within a spec command, not scattered around the codebase
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
