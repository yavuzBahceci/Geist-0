# Placeholder Detection Utility

## Core Responsibilities

1. **Scan Files for Placeholders**: Scan commands, workflows, and standards files for placeholder syntax
2. **Categorize Placeholders**: Group placeholders by type (basepoints, scope detection, deep reading, workflows, standards)
3. **Generate Report**: Create comprehensive report with file locations and placeholder types
4. **Support Multiple Contexts**: Support validation of both profiles/default (template) and installed geist (specialized)

## Workflow

### Step 1: Determine Context and Paths

Determine whether we're validating profiles/default (template) or installed geist (specialized):

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
```

### Step 2: Define Placeholder Patterns

Define all placeholder patterns to detect:

```bash
# Basepoints-related placeholders
BASEPOINTS_PLACEHOLDERS=(
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

# Scope detection placeholders
SCOPE_DETECTION_PLACEHOLDERS=(
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
    "{{EXTRACT_DESIGN_PATTERNS}}"
    "{{EXTRACT_CODING_PATTERNS}}"
    "{{EXTRACT_ARCHITECTURAL_PATTERNS}}"
    "{{EXTRACT_STANDARDS}}"
    "{{EXTRACT_FLOWS}}"
    "{{CHECK_MODULE_RELEVANCE}}"
    "{{MAP_ENTITIES_TO_LAYERS}}"
    "{{MAP_RELATIONSHIPS_TO_LAYERS}}"
    "{{MAP_DOMAIN_TO_LAYERS}}"
    "{{MAP_TECHNICAL_TO_LAYERS}}"
    "{{MAP_ARCHITECTURAL_TO_LAYERS}}"
    "{{CHECK_HEADQUARTER_RELEVANCE}}"
    "{{CHECK_MODULE_RELEVANCE}}"
    "{{AFTER_AGENT_DEPLOYMENT}}"
    "{{LOAD_PROJECT_STRUCTURE}}"
    "{{CALCULATE_ACTUAL_DISTANCES}}"
    "{{UPDATE_HEURISTICS_FOR_STRUCTURE}}"
    "{{CALCULATE_GENERIC_DISTANCES}}"
    "{{CREATE_GENERIC_HEURISTICS}}"
    "{{CREATE_DEEP_READING_DECISION}}"
    "{{JSON_FORMAT}}"
)

# Deep reading placeholders
DEEP_READING_PLACEHOLDERS=(
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

# Workflow and standards placeholders
WORKFLOW_STANDARDS_PLACEHOLDERS=(
    "{{workflows/"
    "{{standards/"
    "{{agents/"
    "{{PHASE"
    "{{IF"
    "{{UNLESS"
    "{{ENDIF"
    "{{ENDUNLESS"
)
```

### Step 3: Scan Files for Placeholders

Scan all relevant files in the target path:

```bash
# Find all files to scan
FILES_TO_SCAN=$(find "$SCAN_PATH" -type f \( -name "*.md" \) ! -path "*/node_modules/*" ! -path "*/.git/*" ! -path "*/build/*" ! -path "*/dist/*" ! -path "*/cache/*" ! -path "*/output/deploy-agents/*" ! -path "*/.cleanup-cache/*")

# Initialize placeholder detection results
PLACEHOLDER_RESULTS=""

# Scan each file
echo "$FILES_TO_SCAN" | while read file_path; do
    if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
        continue
    fi
    
    FILE_CONTENT=$(cat "$file_path")
    FILE_PLACEHOLDERS=""
    
    # Check for basepoints placeholders
    for placeholder in "${BASEPOINTS_PLACEHOLDERS[@]}"; do
        if echo "$FILE_CONTENT" | grep -q "$placeholder"; then
            LINE_NUMBERS=$(grep -n "$placeholder" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
            FILE_PLACEHOLDERS="${FILE_PLACEHOLDERS}\nbasepoints:$placeholder:$file_path:$LINE_NUMBERS"
        fi
    done
    
    # Check for scope detection placeholders
    for placeholder in "${SCOPE_DETECTION_PLACEHOLDERS[@]}"; do
        if echo "$FILE_CONTENT" | grep -q "$placeholder"; then
            LINE_NUMBERS=$(grep -n "$placeholder" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
            FILE_PLACEHOLDERS="${FILE_PLACEHOLDERS}\nscope_detection:$placeholder:$file_path:$LINE_NUMBERS"
        fi
    done
    
    # Check for deep reading placeholders
    for placeholder in "${DEEP_READING_PLACEHOLDERS[@]}"; do
        if echo "$FILE_CONTENT" | grep -q "$placeholder"; then
            LINE_NUMBERS=$(grep -n "$placeholder" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
            FILE_PLACEHOLDERS="${FILE_PLACEHOLDERS}\ndeep_reading:$placeholder:$file_path:$LINE_NUMBERS"
        fi
    done
    
    # Check for workflow/standards placeholders (pattern matching)
    if echo "$FILE_CONTENT" | grep -qE '\{\{workflows/[^}]+\}\}|\{\{standards/[^}]+\}\}|\{\{agents/[^}]+\}\}'; then
        MATCHES=$(grep -nE '\{\{workflows/[^}]+\}\}|\{\{standards/[^}]+\}\}|\{\{agents/[^}]+\}\}' "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
        PLACEHOLDER_PATTERN=$(grep -oE '\{\{workflows/[^}]+\}\}|\{\{standards/[^}]+\}\}|\{\{agents/[^}]+\}\}' "$file_path" | head -1)
        FILE_PLACEHOLDERS="${FILE_PLACEHOLDERS}\nworkflows_standards:$PLACEHOLDER_PATTERN:$file_path:$MATCHES"
    fi
    
    # Check for conditional compilation placeholders
    if echo "$FILE_CONTENT" | grep -qE '\{\{IF|\{\{UNLESS|\{\{ENDIF|\{\{ENDUNLESS'; then
        MATCHES=$(grep -nE '\{\{IF|\{\{UNLESS|\{\{ENDIF|\{\{ENDUNLESS' "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
        FILE_PLACEHOLDERS="${FILE_PLACEHOLDERS}\nconditionals:{{IF/UNLESS}}:$file_path:$MATCHES"
    fi
    
    # Check for PHASE placeholders
    if echo "$FILE_CONTENT" | grep -qE '\{\{PHASE'; then
        MATCHES=$(grep -nE '\{\{PHASE' "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
        FILE_PLACEHOLDERS="${FILE_PLACEHOLDERS}\nphases:{{PHASE}}:$file_path:$MATCHES"
    fi
    
    # Add to results if placeholders found
    if [ -n "$FILE_PLACEHOLDERS" ]; then
        PLACEHOLDER_RESULTS="${PLACEHOLDER_RESULTS}${FILE_PLACEHOLDERS}"
    fi
done
```

### Step 4: Categorize and Organize Results

Organize detected placeholders by category:

```bash
# Initialize categorized results
BASEPOINTS_PLACEHOLDERS_FOUND=""
SCOPE_DETECTION_PLACEHOLDERS_FOUND=""
DEEP_READING_PLACEHOLDERS_FOUND=""
WORKFLOWS_STANDARDS_PLACEHOLDERS_FOUND=""
CONDITIONALS_PLACEHOLDERS_FOUND=""
PHASE_PLACEHOLDERS_FOUND=""

# Categorize results
echo "$PLACEHOLDER_RESULTS" | while IFS=':' read -r category placeholder file_path line_numbers; do
    if [ -z "$category" ]; then
        continue
    fi
    
    case "$category" in
        basepoints)
            BASEPOINTS_PLACEHOLDERS_FOUND="${BASEPOINTS_PLACEHOLDERS_FOUND}\n$placeholder|$file_path|$line_numbers"
            ;;
        scope_detection)
            SCOPE_DETECTION_PLACEHOLDERS_FOUND="${SCOPE_DETECTION_PLACEHOLDERS_FOUND}\n$placeholder|$file_path|$line_numbers"
            ;;
        deep_reading)
            DEEP_READING_PLACEHOLDERS_FOUND="${DEEP_READING_PLACEHOLDERS_FOUND}\n$placeholder|$file_path|$line_numbers"
            ;;
        workflows_standards)
            WORKFLOWS_STANDARDS_PLACEHOLDERS_FOUND="${WORKFLOWS_STANDARDS_PLACEHOLDERS_FOUND}\n$placeholder|$file_path|$line_numbers"
            ;;
        conditionals)
            CONDITIONALS_PLACEHOLDERS_FOUND="${CONDITIONALS_PLACEHOLDERS_FOUND}\n$placeholder|$file_path|$line_numbers"
            ;;
        phases)
            PHASE_PLACEHOLDERS_FOUND="${PHASE_PLACEHOLDERS_FOUND}\n$placeholder|$file_path|$line_numbers"
            ;;
    esac
done

# Count placeholders by category
BASEPOINTS_COUNT=$(echo "$BASEPOINTS_PLACEHOLDERS_FOUND" | grep -c . || echo "0")
SCOPE_DETECTION_COUNT=$(echo "$SCOPE_DETECTION_PLACEHOLDERS_FOUND" | grep -c . || echo "0")
DEEP_READING_COUNT=$(echo "$DEEP_READING_PLACEHOLDERS_FOUND" | grep -c . || echo "0")
WORKFLOWS_STANDARDS_COUNT=$(echo "$WORKFLOWS_STANDARDS_PLACEHOLDERS_FOUND" | grep -c . || echo "0")
CONDITIONALS_COUNT=$(echo "$CONDITIONALS_PLACEHOLDERS_FOUND" | grep -c . || echo "0")
PHASE_COUNT=$(echo "$PHASE_PLACEHOLDERS_FOUND" | grep -c . || echo "0")
TOTAL_COUNT=$((BASEPOINTS_COUNT + SCOPE_DETECTION_COUNT + DEEP_READING_COUNT + WORKFLOWS_STANDARDS_COUNT + CONDITIONALS_COUNT + PHASE_COUNT))
```

### Step 5: Generate Validation Report

Generate comprehensive placeholder detection report:

```bash
# Create JSON report
cat > "$CACHE_PATH/placeholder-detection.json" << EOF
{
  "validation_context": "$VALIDATION_CONTEXT",
  "scan_path": "$SCAN_PATH",
  "scan_timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "total_placeholders_found": $TOTAL_COUNT,
  "categories": {
    "basepoints": {
      "count": $BASEPOINTS_COUNT,
      "placeholders": [
$(echo "$BASEPOINTS_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS='|' read -r placeholder file_path line_numbers; do
    echo "        {\"placeholder\": \"$placeholder\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\"},"
done | sed '$ s/,$//')
      ]
    },
    "scope_detection": {
      "count": $SCOPE_DETECTION_COUNT,
      "placeholders": [
$(echo "$SCOPE_DETECTION_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS='|' read -r placeholder file_path line_numbers; do
    echo "        {\"placeholder\": \"$placeholder\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\"},"
done | sed '$ s/,$//')
      ]
    },
    "deep_reading": {
      "count": $DEEP_READING_COUNT,
      "placeholders": [
$(echo "$DEEP_READING_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS='|' read -r placeholder file_path line_numbers; do
    echo "        {\"placeholder\": \"$placeholder\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\"},"
done | sed '$ s/,$//')
      ]
    },
    "workflows_standards": {
      "count": $WORKFLOWS_STANDARDS_COUNT,
      "placeholders": [
$(echo "$WORKFLOWS_STANDARDS_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS='|' read -r placeholder file_path line_numbers; do
    echo "        {\"placeholder\": \"$placeholder\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\"},"
done | sed '$ s/,$//')
      ]
    },
    "conditionals": {
      "count": $CONDITIONALS_COUNT,
      "placeholders": [
$(echo "$CONDITIONALS_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS='|' read -r placeholder file_path line_numbers; do
    echo "        {\"placeholder\": \"$placeholder\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\"},"
done | sed '$ s/,$//')
      ]
    },
    "phases": {
      "count": $PHASE_COUNT,
      "placeholders": [
$(echo "$PHASE_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS='|' read -r placeholder file_path line_numbers; do
    echo "        {\"placeholder\": \"$placeholder\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\"},"
done | sed '$ s/,$//')
      ]
    }
  }
}
EOF

# Create markdown summary report
cat > "$CACHE_PATH/placeholder-detection-summary.md" << EOF
# Placeholder Detection Report

**Validation Context:** $VALIDATION_CONTEXT  
**Scan Path:** $SCAN_PATH  
**Scan Timestamp:** $(date -u +%Y-%m-%dT%H:%M:%SZ)

## Summary

- **Total Placeholders Found:** $TOTAL_COUNT
- **Basepoints Placeholders:** $BASEPOINTS_COUNT
- **Scope Detection Placeholders:** $SCOPE_DETECTION_COUNT
- **Deep Reading Placeholders:** $DEEP_READING_COUNT
- **Workflows/Standards Placeholders:** $WORKFLOWS_STANDARDS_COUNT
- **Conditional Compilation Placeholders:** $CONDITIONALS_COUNT
- **Phase Placeholders:** $PHASE_COUNT

## Basepoints Placeholders

$(echo "$BASEPOINTS_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS='|' read -r placeholder file_path line_numbers; do
    echo "- **$placeholder**"
    echo "  - File: \`$file_path\`"
    echo "  - Lines: $line_numbers"
    echo ""
done)

## Scope Detection Placeholders

$(echo "$SCOPE_DETECTION_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS='|' read -r placeholder file_path line_numbers; do
    echo "- **$placeholder**"
    echo "  - File: \`$file_path\`"
    echo "  - Lines: $line_numbers"
    echo ""
done)

## Deep Reading Placeholders

$(echo "$DEEP_READING_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS='|' read -r placeholder file_path line_numbers; do
    echo "- **$placeholder**"
    echo "  - File: \`$file_path\`"
    echo "  - Lines: $line_numbers"
    echo ""
done)

## Workflows/Standards Placeholders

$(echo "$WORKFLOWS_STANDARDS_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS='|' read -r placeholder file_path line_numbers; do
    echo "- **$placeholder**"
    echo "  - File: \`$file_path\`"
    echo "  - Lines: $line_numbers"
    echo ""
done)

## Conditional Compilation Placeholders

$(echo "$CONDITIONALS_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS='|' read -r placeholder file_path line_numbers; do
    echo "- **$placeholder**"
    echo "  - File: \`$file_path\`"
    echo "  - Lines: $line_numbers"
    echo ""
done)

## Phase Placeholders

$(echo "$PHASE_PLACEHOLDERS_FOUND" | grep -v "^$" | while IFS='|' read -r placeholder file_path line_numbers; do
    echo "- **$placeholder**"
    echo "  - File: \`$file_path\`"
    echo "  - Lines: $line_numbers"
    echo ""
done)
EOF

echo "✅ Placeholder detection complete. Report stored in $CACHE_PATH/"
```

## Important Constraints

- Must detect all placeholder syntax patterns across all categories
- Must scan commands, workflows, and standards files
- Must support both profiles/default (template) and installed geist (specialized) validation
- Must generate comprehensive reports with file locations and placeholder types
- Must categorize placeholders by type for easy analysis
- **CRITICAL**: All reports must be stored in `geist/specs/[current-spec]/implementation/cache/validation/` when running within a spec command, not scattered around the codebase
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
