# Placeholder Cleaning Validation

## Core Responsibilities

1. **Comprehensive Placeholder Detection**: Detect all placeholder types that should be cleaned from specialized commands
2. **Categorize by Type**: Group placeholders by category (basepoints, scope detection, deep reading, workflows, standards)
3. **Generate Cleaning Report**: Create detailed report with file locations and recommendations for cleaning
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

echo "🔍 Starting placeholder cleaning validation for: $VALIDATION_CONTEXT"
```

### Step 2: Use Placeholder Detection Utility

Use the placeholder detection utility from Task Group 1:

```bash
# Run placeholder detection utility
{{workflows/validation/detect-placeholders}}

# Load detection results
if [ -f "$CACHE_PATH/placeholder-detection.json" ]; then
    PLACEHOLDER_DATA=$(cat "$CACHE_PATH/placeholder-detection.json")
else
    echo "❌ Placeholder detection failed - no results found"
    exit 1
fi
```

### Step 3: Implement Basepoints-Related Placeholder Detection

Extract and categorize basepoints-related placeholders:

```bash
# Extract basepoints-related placeholders from detection results
BASEPOINTS_PLACEHOLDERS_FOUND=""

# Key basepoints placeholders to check
BASEPOINTS_KEY_PLACEHOLDERS=(
    "{{BASEPOINTS_PATH}}"
    "{{BASEPOINT_FILE_PATTERN}}"
    "{{EXTRACT_PATTERNS_SECTION}}"
    "{{EXTRACT_DESIGN_PATTERNS}}"
    "{{EXTRACT_CODING_PATTERNS}}"
    "{{EXTRACT_ARCHITECTURAL_PATTERNS}}"
    "{{EXTRACT_STANDARDS_SECTION}}"
    "{{EXTRACT_FLOWS_SECTION}}"
    "{{EXTRACT_STRATEGIES_SECTION}}"
    "{{EXTRACT_TESTING_SECTION}}"
    "{{EXTRACT_PROS_CONS}}"
    "{{EXTRACT_HISTORICAL_DECISIONS}}"
    "{{EXTRACT_RELATED_MODULES}}"
    "{{EXTRACT_PERFORMANCE_CONSIDERATIONS}}"
    "{{EXTRACT_REUSABLE_PATTERNS}}"
    "{{ABSTRACTION_LAYER_NAMES}}"
    "{{LAYER_DETECTION_PATTERN}}"
    "{{EXTRACT_LAYER_FROM_PATH}}"
    "{{EXTRACT_LAYERS_FROM_HEADQUARTER}}"
    "{{EXTRACT_MODULE_NAME}}"
    "{{EXTRACT_MODULE_LAYER}}"
    "{{EXTRACT_CROSS_LAYER_PATTERNS}}"
    "{{EXTRACT_DEPENDENCY_FLOWS}}"
    "{{EXTRACT_INTERACTION_PATTERNS}}"
    "{{EXTRACT_NAMING_CONVENTIONS}}"
    "{{EXTRACT_CODING_STYLE}}"
    "{{EXTRACT_STRUCTURE_STANDARDS}}"
    "{{EXTRACT_DATA_FLOWS}}"
    "{{EXTRACT_CONTROL_FLOWS}}"
    "{{EXTRACT_IMPL_STRATEGIES}}"
    "{{EXTRACT_ARCH_STRATEGIES}}"
    "{{EXTRACT_TEST_PATTERNS}}"
    "{{EXTRACT_TEST_STRATEGIES}}"
    "{{EXTRACT_TEST_ORGANIZATION}}"
    "{{GENERATE_CACHE_KEY}}"
    "{{CHECK_CACHE}}"
    "{{FIND_MODULE_BASEPOINTS}}"
    "{{FIND_LAYER_BASEPOINTS}}"
    "{{EXTRACT_SAME_LAYER_PATTERNS}}"
    "{{QUERY_CATEGORY_FROM_CACHE}}"
    "{{QUERY_LAYER_FROM_CACHE}}"
    "{{QUERY_MODULE_FROM_CACHE}}"
    "{{QUERY_PATTERNS_FROM_CACHE}}"
    "{{QUERY_STANDARDS_FROM_CACHE}}"
    "{{QUERY_FLOWS_FROM_CACHE}}"
    "{{QUERY_STRATEGIES_FROM_CACHE}}"
    "{{QUERY_TESTING_FROM_CACHE}}"
    "{{QUERY_PROS_CONS_FROM_CACHE}}"
    "{{QUERY_HISTORICAL_DECISIONS_FROM_CACHE}}"
    "{{QUERY_RELATED_MODULES_FROM_CACHE}}"
    "{{QUERY_PERFORMANCE_FROM_CACHE}}"
    "{{QUERY_REUSABLE_PATTERNS_FROM_CACHE}}"
    "{{GET_CACHE_TIMESTAMP}}"
    "{{GET_CACHE_AGE}}"
    "{{GET_CACHE_SIZE}}"
)

# Find files to scan
FILES_TO_SCAN=$(find "$SCAN_PATH" -type f \( -name "*.md" \) ! -path "*/node_modules/*" ! -path "*/.git/*" ! -path "*/build/*" ! -path "*/dist/*" ! -path "*/cache/*" ! -path "*/output/deploy-agents/*" ! -path "*/.cleanup-cache/*")

# Scan for basepoints placeholders
echo "$FILES_TO_SCAN" | while read file_path; do
    if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
        continue
    fi
    
    FILE_CONTENT=$(cat "$file_path")
    
    for placeholder in "${BASEPOINTS_KEY_PLACEHOLDERS[@]}"; do
        if echo "$FILE_CONTENT" | grep -q "$placeholder"; then
            LINE_NUMBERS=$(grep -n "$placeholder" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
            BASEPOINTS_PLACEHOLDERS_FOUND="${BASEPOINTS_PLACEHOLDERS_FOUND}\nbasepoints:$placeholder:$file_path:$LINE_NUMBERS"
        fi
    done
done

# Count basepoints placeholders
BASEPOINTS_COUNT=$(echo "$BASEPOINTS_PLACEHOLDERS_FOUND" | grep -c . || echo "0")
```

### Step 4: Implement Scope Detection Placeholder Detection

Extract and categorize scope detection placeholders:

```bash
# Extract scope detection placeholders
SCOPE_DETECTION_PLACEHOLDERS_FOUND=""

# Key scope detection placeholders to check
SCOPE_DETECTION_KEY_PLACEHOLDERS=(
    "{{KEYWORD_EXTRACTION_PATTERN}}"
    "{{EXTRACT_KEYWORDS}}"
    "{{EXTRACT_PATTERNS}}"
    "{{EXTRACT_MODULE_NAMES}}"
    "{{EXTRACT_LAYER_NAMES}}"
    "{{EXTRACT_TECHNICAL_TERMS}}"
    "{{SEMANTIC_ANALYSIS_PATTERN}}"
    "{{EXTRACT_ENTITIES}}"
    "{{EXTRACT_RELATIONSHIPS}}"
    "{{EXTRACT_DOMAIN_CONCEPTS}}"
    "{{EXTRACT_TECHNICAL_CONCEPTS}}"
    "{{EXTRACT_ARCHITECTURAL_CONCEPTS}}"
    "{{DETERMINE_SPEC_CONTEXT_LAYER}}"
    "{{SPEC_SPANS_MULTIPLE_LAYERS}}"
    "{{IDENTIFY_PRIMARY_LAYER}}"
    "{{IS_PRODUCT_LEVEL}}"
    "{{CALCULATE_LAYER_DISTANCE}}"
    "{{CALCULATE_IMPLEMENTATION_DISTANCE}}"
    "{{CALCULATE_OVERALL_DISTANCE}}"
    "{{FIND_MODULE_LAYER}}"
    "{{MATCH_KEYWORDS_TO_MODULE_NAME}}"
    "{{MATCH_KEYWORDS_TO_MODULE_CONTENT}}"
    "{{MATCH_LAYER_NAMES}}"
    "{{MATCH_KEYWORDS_TO_HEADQUARTER}}"
    "{{GROUP_BY_LAYER}}"
    "{{LOAD_ABSTRACTION_LAYERS}}"
    "{{DETECT_LAYERED_ARCHITECTURE}}"
    "{{DETECT_HEXAGONAL_ARCHITECTURE}}"
    "{{DETECT_MICROSERVICES}}"
    "{{DETECT_EVENT_DRIVEN}}"
    "{{DETECT_CUSTOM_CROSS_LAYER_PATTERNS}}"
    "{{EXTRACT_CROSS_LAYER_DEPENDENCIES}}"
    "{{EXTRACT_CROSS_LAYER_INTERACTIONS}}"
    "{{EXTRACT_PARENT_CROSS_LAYER_PATTERNS}}"
    "{{CHECK_PARENT_RELEVANCE}}"
    "{{CHECK_PATTERN_RELEVANCE}}"
    "{{EXTRACT_DETECTED_LAYERS}}"
    "{{EXTRACT_MATCHED_LAYERS}}"
    "{{COMBINE_LAYERS}}"
    "{{CHECK_MODULE_RELEVANCE}}"
    "{{MAP_ENTITIES_TO_LAYERS}}"
    "{{MAP_RELATIONSHIPS_TO_LAYERS}}"
    "{{MAP_DOMAIN_TO_LAYERS}}"
    "{{MAP_TECHNICAL_TO_LAYERS}}"
    "{{MAP_ARCHITECTURAL_TO_LAYERS}}"
    "{{CHECK_HEADQUARTER_RELEVANCE}}"
    "{{AFTER_AGENT_DEPLOYMENT}}"
    "{{LOAD_PROJECT_STRUCTURE}}"
    "{{CALCULATE_ACTUAL_DISTANCES}}"
    "{{UPDATE_HEURISTICS_FOR_STRUCTURE}}"
    "{{CALCULATE_GENERIC_DISTANCES}}"
    "{{CREATE_GENERIC_HEURISTICS}}"
    "{{CREATE_DEEP_READING_DECISION}}"
)

# Scan for scope detection placeholders
echo "$FILES_TO_SCAN" | while read file_path; do
    if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
        continue
    fi
    
    FILE_CONTENT=$(cat "$file_path")
    
    for placeholder in "${SCOPE_DETECTION_KEY_PLACEHOLDERS[@]}"; do
        if echo "$FILE_CONTENT" | grep -q "$placeholder"; then
            LINE_NUMBERS=$(grep -n "$placeholder" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
            SCOPE_DETECTION_PLACEHOLDERS_FOUND="${SCOPE_DETECTION_PLACEHOLDERS_FOUND}\nscope_detection:$placeholder:$file_path:$LINE_NUMBERS"
        fi
    done
done

# Count scope detection placeholders
SCOPE_DETECTION_COUNT=$(echo "$SCOPE_DETECTION_PLACEHOLDERS_FOUND" | grep -c . || echo "0")
```

### Step 5: Implement Deep Reading Placeholder Detection

Extract and categorize deep reading placeholders:

```bash
# Extract deep reading placeholders
DEEP_READING_PLACEHOLDERS_FOUND=""

# Key deep reading placeholders to check
DEEP_READING_KEY_PLACEHOLDERS=(
    "{{CODE_FILE_PATTERNS}}"
    "{{FIND_MODULE_PATH}}"
    "{{EXTRACT_DESIGN_PATTERNS}}"
    "{{EXTRACT_CODING_PATTERNS}}"
    "{{EXTRACT_SIMILAR_LOGIC}}"
    "{{EXTRACT_REUSABLE_BLOCKS}}"
    "{{EXTRACT_REUSABLE_FUNCTIONS}}"
    "{{EXTRACT_REUSABLE_CLASSES}}"
    "{{DETECT_SIMILAR_CODE}}"
    "{{IDENTIFY_CORE_PATTERNS}}"
    "{{IDENTIFY_COMMON_MODULES}}"
    "{{IDENTIFY_SHARED_LOCATION_OPPORTUNITIES}}"
    "{{ANALYZE_LOGIC_FLOW}}"
    "{{ANALYZE_DATA_FLOW}}"
    "{{ANALYZE_CONTROL_FLOW}}"
    "{{ANALYZE_DEPENDENCIES}}"
    "{{ANALYZE_PATTERNS_USED}}"
    "{{EXTRACT_PATTERNS}}"
    "{{EXTRACT_LOGIC}}"
    "{{EXTRACT_REUSABLE_CODE}}"
    "{{EXTRACT_REUSABILITY_OPPORTUNITIES}}"
    "{{FIND_SIMILAR_LOGIC}}"
    "{{FIND_EXISTING_MODULES}}"
    "{{CAN_REUSE_MODULE}}"
    "{{FIND_EXISTING_CODE}}"
    "{{CAN_REUSE_CODE}}"
    "{{DETECT_DUPLICATE_CODE}}"
    "{{IDENTIFY_SHARED_LOCATION_CANDIDATES}}"
    "{{IDENTIFY_COMMON_MODULE_CANDIDATES}}"
    "{{IDENTIFY_CORE_MODULE_CANDIDATES}}"
    "{{GET_BASEPOINT_CONTEXT}}"
    "{{GET_BASEPOINT_PROS_CONS}}"
    "{{EXTRACT_RELEVANT_MODULES}}"
    "{{EXTRACT_CROSS_LAYER_MODULES}}"
    "{{COMBINE_MODULES}}"
    "{{EXTRACT_DEEP_READING_NEEDED}}"
    "{{EXTRACT_OVERALL_DISTANCE}}"
)

# Scan for deep reading placeholders
echo "$FILES_TO_SCAN" | while read file_path; do
    if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
        continue
    fi
    
    FILE_CONTENT=$(cat "$file_path")
    
    for placeholder in "${DEEP_READING_KEY_PLACEHOLDERS[@]}"; do
        if echo "$FILE_CONTENT" | grep -q "$placeholder"; then
            LINE_NUMBERS=$(grep -n "$placeholder" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
            DEEP_READING_PLACEHOLDERS_FOUND="${DEEP_READING_PLACEHOLDERS_FOUND}\ndeep_reading:$placeholder:$file_path:$LINE_NUMBERS"
        fi
    done
done

# Count deep reading placeholders
DEEP_READING_COUNT=$(echo "$DEEP_READING_PLACEHOLDERS_FOUND" | grep -c . || echo "0")
```

### Step 6: Implement Workflow and Standards Placeholder Detection

Check for unresolved workflow and standards placeholders:

```bash
# Extract workflow and standards placeholders
WORKFLOWS_STANDARDS_PLACEHOLDERS_FOUND=""

# Scan for unresolved workflow/standards placeholders
echo "$FILES_TO_SCAN" | while read file_path; do
    if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
        continue
    fi
    
    FILE_CONTENT=$(cat "$file_path")
    
    # Check for unresolved {{workflows/...}} placeholders
    if echo "$FILE_CONTENT" | grep -qE '\{\{workflows/[^}]+\}\}'; then
        WORKFLOW_MATCHES=$(grep -nE '\{\{workflows/[^}]+\}\}' "$file_path")
        echo "$WORKFLOW_MATCHES" | while IFS=':' read -r line_number rest; do
            PLACEHOLDER=$(echo "$rest" | grep -oE '\{\{workflows/[^}]+\}\}')
            # Check if placeholder references an actual workflow file
            WORKFLOW_PATH=$(echo "$PLACEHOLDER" | sed 's/{{workflows\///' | sed 's/}}//')
            if [ "$VALIDATION_CONTEXT" = "installed-geist" ]; then
                # In specialized context, check if workflow exists
                if [ ! -f "$SCAN_PATH/workflows/$WORKFLOW_PATH.md" ] && [ ! -f "profiles/default/workflows/$WORKFLOW_PATH.md" ]; then
                    WORKFLOWS_STANDARDS_PLACEHOLDERS_FOUND="${WORKFLOWS_STANDARDS_PLACEHOLDERS_FOUND}\nworkflows:$PLACEHOLDER:$file_path:$line_number:Unresolved workflow reference"
                fi
            fi
        done
    fi
    
    # Check for unresolved {{standards/...}} placeholders
    if echo "$FILE_CONTENT" | grep -qE '\{\{standards/[^}]+\}\}'; then
        STANDARDS_MATCHES=$(grep -nE '\{\{standards/[^}]+\}\}' "$file_path")
        echo "$STANDARDS_MATCHES" | while IFS=':' read -r line_number rest; do
            PLACEHOLDER=$(echo "$rest" | grep -oE '\{\{standards/[^}]+\}\}')
            # Check if placeholder references an actual standards file
            STANDARDS_PATH=$(echo "$PLACEHOLDER" | sed 's/{{standards\///' | sed 's/}}//')
            if [ "$VALIDATION_CONTEXT" = "installed-geist" ]; then
                # In specialized context, check if standards file exists
                if [ ! -f "$SCAN_PATH/standards/$STANDARDS_PATH.md" ] && [ ! -f "profiles/default/standards/$STANDARDS_PATH.md" ]; then
                    WORKFLOWS_STANDARDS_PLACEHOLDERS_FOUND="${WORKFLOWS_STANDARDS_PLACEHOLDERS_FOUND}\nstandards:$PLACEHOLDER:$file_path:$line_number:Unresolved standards reference"
                fi
            fi
        done
    fi
done

# Count workflow/standards placeholders
WORKFLOWS_STANDARDS_COUNT=$(echo "$WORKFLOWS_STANDARDS_PLACEHOLDERS_FOUND" | grep -c . || echo "0")
```

### Step 7: Generate Placeholder Validation Report

Generate comprehensive placeholder cleaning validation report:

```bash
# Calculate totals
TOTAL_PLACEHOLDERS=$((BASEPOINTS_COUNT + SCOPE_DETECTION_COUNT + DEEP_READING_COUNT + WORKFLOWS_STANDARDS_COUNT))

# Create JSON report
cat > "$CACHE_PATH/placeholder-cleaning-validation.json" << EOF
{
  "validation_context": "$VALIDATION_CONTEXT",
  "scan_path": "$SCAN_PATH",
  "validation_timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "total_placeholders_found": $TOTAL_PLACEHOLDERS,
  "categories": {
    "basepoints": {
      "count": $BASEPOINTS_COUNT,
      "placeholders": [
$(echo "$BASEPOINTS_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS=':' read -r category placeholder file_path line_numbers; do
    echo "        {\"placeholder\": \"$placeholder\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\"},"
done | sed '$ s/,$//')
      ],
      "recommendation": "Replace with project-specific basepoints paths and extraction logic from deploy-agents specialization"
    },
    "scope_detection": {
      "count": $SCOPE_DETECTION_COUNT,
      "placeholders": [
$(echo "$SCOPE_DETECTION_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS=':' read -r category placeholder file_path line_numbers; do
    echo "        {\"placeholder\": \"$placeholder\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\"},"
done | sed '$ s/,$//')
      ],
      "recommendation": "Replace with project-specific scope detection patterns from deploy-agents specialization"
    },
    "deep_reading": {
      "count": $DEEP_READING_COUNT,
      "placeholders": [
$(echo "$DEEP_READING_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS=':' read -r category placeholder file_path line_numbers; do
    echo "        {\"placeholder\": \"$placeholder\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\"},"
done | sed '$ s/,$//')
      ],
      "recommendation": "Replace with project-specific deep reading patterns from deploy-agents specialization"
    },
    "workflows_standards": {
      "count": $WORKFLOWS_STANDARDS_COUNT,
      "placeholders": [
$(echo "$WORKFLOWS_STANDARDS_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS=':' read -r category placeholder file_path line_numbers description; do
    echo "        {\"placeholder\": \"$placeholder\", \"file\": \"$file_path\", \"line\": \"$line_numbers\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ],
      "recommendation": "Verify all workflow and standards references resolve to actual files"
    }
  },
  "cleaning_status": "$(if [ "$TOTAL_PLACEHOLDERS" -eq 0 ]; then echo "clean"; else echo "needs_cleaning"; fi)"
}
EOF

# Create markdown summary report
cat > "$CACHE_PATH/placeholder-cleaning-validation-summary.md" << EOF
# Placeholder Cleaning Validation Report

**Validation Context:** $VALIDATION_CONTEXT  
**Scan Path:** $SCAN_PATH  
**Validation Timestamp:** $(date -u +%Y-%m-%dT%H:%M:%SZ)

## Summary

- **Total Placeholders Found:** $TOTAL_PLACEHOLDERS
- **Basepoints Placeholders:** $BASEPOINTS_COUNT
- **Scope Detection Placeholders:** $SCOPE_DETECTION_COUNT
- **Deep Reading Placeholders:** $DEEP_READING_COUNT
- **Workflows/Standards Placeholders:** $WORKFLOWS_STANDARDS_COUNT
- **Cleaning Status:** $(if [ "$TOTAL_PLACEHOLDERS" -eq 0 ]; then echo "✅ Clean - No placeholders found"; else echo "⚠️  Needs Cleaning - $TOTAL_PLACEHOLDERS placeholder(s) need to be replaced"; fi)

## Basepoints-Related Placeholders

$(if [ "$BASEPOINTS_COUNT" -gt 0 ]; then
    echo "$BASEPOINTS_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS=':' read -r category placeholder file_path line_numbers; do
        echo "- **$placeholder**"
        echo "  - File: \`$file_path\`"
        echo "  - Lines: $line_numbers"
        echo "  - Recommendation: Replace with project-specific basepoints paths and extraction logic"
        echo ""
    done
else
    echo "✅ No basepoints-related placeholders found"
fi)

## Scope Detection Placeholders

$(if [ "$SCOPE_DETECTION_COUNT" -gt 0 ]; then
    echo "$SCOPE_DETECTION_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS=':' read -r category placeholder file_path line_numbers; do
        echo "- **$placeholder**"
        echo "  - File: \`$file_path\`"
        echo "  - Lines: $line_numbers"
        echo "  - Recommendation: Replace with project-specific scope detection patterns"
        echo ""
    done
else
    echo "✅ No scope detection placeholders found"
fi)

## Deep Reading Placeholders

$(if [ "$DEEP_READING_COUNT" -gt 0 ]; then
    echo "$DEEP_READING_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS=':' read -r category placeholder file_path line_numbers; do
        echo "- **$placeholder**"
        echo "  - File: \`$file_path\`"
        echo "  - Lines: $line_numbers"
        echo "  - Recommendation: Replace with project-specific deep reading patterns"
        echo ""
    done
else
    echo "✅ No deep reading placeholders found"
fi)

## Workflows and Standards Placeholders

$(if [ "$WORKFLOWS_STANDARDS_COUNT" -gt 0 ]; then
    echo "$WORKFLOWS_STANDARDS_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS=':' read -r category placeholder file_path line_numbers description; do
        echo "- **$placeholder**"
        echo "  - File: \`$file_path\`"
        echo "  - Line: $line_numbers"
        echo "  - Issue: $description"
        echo "  - Recommendation: Verify workflow/standards file exists or fix reference"
        echo ""
    done
else
    echo "✅ No unresolved workflow/standards placeholders found"
fi)

## Recommendations

$(if [ "$TOTAL_PLACEHOLDERS" -gt 0 ]; then
    echo "### Action Required"
    echo ""
    echo "The following actions are recommended to clean remaining placeholders:"
    echo ""
    echo "1. **Run deploy-agents command** to replace placeholders with project-specific content"
    echo "2. **Review deploy-agents specialization functions** to ensure all placeholder types are handled"
    echo "3. **Run cleanup command** (if available) to fix remaining placeholders in already-deployed geist"
    echo "4. **Manually review** files listed above and replace placeholders with project-specific content"
else
    echo "✅ **No Action Required** - All placeholders have been cleaned!"
fi)
EOF

echo "✅ Placeholder cleaning validation complete!"
echo "📊 Total placeholders found: $TOTAL_PLACEHOLDERS"
echo "📁 Report stored in: $CACHE_PATH/placeholder-cleaning-validation-summary.md"
```

## Important Constraints

- Must detect all placeholder types (basepoints, scope detection, deep reading, workflows, standards)
- Must categorize placeholders by type for easy analysis
- Must generate comprehensive reports with file locations
- Must provide recommendations for cleaning each placeholder type
- Must support validation of both profiles/default (template) and installed geist (specialized)
- **CRITICAL**: All reports must be stored in `geist/specs/[current-spec]/implementation/cache/validation/` when running within a spec command, not scattered around the codebase
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
