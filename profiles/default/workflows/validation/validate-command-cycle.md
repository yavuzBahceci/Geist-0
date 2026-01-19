# Command Cycle Validation Utility

## Core Responsibilities

1. **Verify Command Flow**: Verify each command correctly references outputs from previous commands
2. **Check Independence**: Verify implement-tasks and orchestrate-tasks are independent (both depend on create-tasks, not each other)
3. **Detect Broken References**: Check for missing connections or broken references between commands
4. **Validate Cycle Structure**: Verify specialized commands maintain same cycle structure as profiles/default versions

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

### Step 2: Verify shape-spec → write-spec Flow

Check that write-spec references shape-spec outputs:

```bash
# Initialize results
CYCLE_ISSUES=""

# Check shape-spec → write-spec flow
SHAPE_SPEC_FILE="$SCAN_PATH/commands/shape-spec/single-agent/shape-spec.md"
WRITE_SPEC_FILE="$SCAN_PATH/commands/write-spec/single-agent/write-spec.md"

if [ -f "$SHAPE_SPEC_FILE" ] && [ -f "$WRITE_SPEC_FILE" ]; then
    WRITE_SPEC_CONTENT=$(cat "$WRITE_SPEC_FILE")
    
    # Check if write-spec references shape-spec outputs
    if ! echo "$WRITE_SPEC_CONTENT" | grep -qE "spec\.md|requirements\.md|planning/"; then
        CYCLE_ISSUES="${CYCLE_ISSUES}\nflow_issue:shape-spec→write-spec:$WRITE_SPEC_FILE:write-spec may not reference shape-spec outputs (spec.md, requirements.md)"
    fi
fi
```

### Step 3: Verify write-spec → create-tasks Flow

Check that create-tasks references write-spec outputs:

```bash
# Check write-spec → create-tasks flow
CREATE_TASKS_FILE="$SCAN_PATH/commands/create-tasks/single-agent/create-tasks.md"

if [ -f "$WRITE_SPEC_FILE" ] && [ -f "$CREATE_TASKS_FILE" ]; then
    CREATE_TASKS_CONTENT=$(cat "$CREATE_TASKS_FILE")
    
    # Check if create-tasks references write-spec outputs
    if ! echo "$CREATE_TASKS_CONTENT" | grep -qE "spec\.md|requirements\.md"; then
        CYCLE_ISSUES="${CYCLE_ISSUES}\nflow_issue:write-spec→create-tasks:$CREATE_TASKS_FILE:create-tasks may not reference write-spec outputs (spec.md, requirements.md)"
    fi
fi
```

### Step 4: Verify create-tasks → implement-tasks Flow

Check that implement-tasks references create-tasks outputs:

```bash
# Check create-tasks → implement-tasks flow
IMPLEMENT_TASKS_FILE="$SCAN_PATH/commands/implement-tasks/single-agent/implement-tasks.md"

if [ -f "$CREATE_TASKS_FILE" ] && [ -f "$IMPLEMENT_TASKS_FILE" ]; then
    IMPLEMENT_TASKS_CONTENT=$(cat "$IMPLEMENT_TASKS_FILE")
    
    # Check if implement-tasks references create-tasks outputs
    if ! echo "$IMPLEMENT_TASKS_CONTENT" | grep -qE "tasks\.md"; then
        CYCLE_ISSUES="${CYCLE_ISSUES}\nflow_issue:create-tasks→implement-tasks:$IMPLEMENT_TASKS_FILE:implement-tasks may not reference create-tasks outputs (tasks.md)"
    fi
fi
```

### Step 5: Verify create-tasks → orchestrate-tasks Flow

Check that orchestrate-tasks references create-tasks outputs:

```bash
# Check create-tasks → orchestrate-tasks flow
ORCHESTRATE_TASKS_FILE="$SCAN_PATH/commands/orchestrate-tasks/orchestrate-tasks.md"

if [ -f "$CREATE_TASKS_FILE" ] && [ -f "$ORCHESTRATE_TASKS_FILE" ]; then
    ORCHESTRATE_TASKS_CONTENT=$(cat "$ORCHESTRATE_TASKS_FILE")
    
    # Check if orchestrate-tasks references create-tasks outputs
    if ! echo "$ORCHESTRATE_TASKS_CONTENT" | grep -qE "tasks\.md"; then
        CYCLE_ISSUES="${CYCLE_ISSUES}\nflow_issue:create-tasks→orchestrate-tasks:$ORCHESTRATE_TASKS_FILE:orchestrate-tasks may not reference create-tasks outputs (tasks.md)"
    fi
fi
```

### Step 6: Verify implement-tasks and orchestrate-tasks Independence

Check that implement-tasks and orchestrate-tasks are independent (both depend on create-tasks, not each other):

```bash
# Check independence
if [ -f "$IMPLEMENT_TASKS_FILE" ] && [ -f "$ORCHESTRATE_TASKS_FILE" ]; then
    IMPLEMENT_TASKS_CONTENT=$(cat "$IMPLEMENT_TASKS_FILE")
    ORCHESTRATE_TASKS_CONTENT=$(cat "$ORCHESTRATE_TASKS_FILE")
    
    # Check if implement-tasks references orchestrate-tasks (should not)
    if echo "$IMPLEMENT_TASKS_CONTENT" | grep -qE "orchestrate-tasks|orchestration"; then
        CYCLE_ISSUES="${CYCLE_ISSUES}\ndependency_issue:implement-tasks→orchestrate-tasks:$IMPLEMENT_TASKS_FILE:implement-tasks should not depend on orchestrate-tasks (they are independent)"
    fi
    
    # Check if orchestrate-tasks references implement-tasks outputs (should not directly)
    # Note: orchestrate-tasks may reference implement-tasks workflow, but should not depend on implement-tasks command outputs
    if echo "$ORCHESTRATE_TASKS_CONTENT" | grep -qE "implement-tasks.*output|implement-tasks.*result"; then
        CYCLE_ISSUES="${CYCLE_ISSUES}\ndependency_issue:orchestrate-tasks→implement-tasks:$ORCHESTRATE_TASKS_FILE:orchestrate-tasks should not depend on implement-tasks outputs (they are independent)"
    fi
    
    # Both should reference create-tasks outputs (tasks.md)
    if ! echo "$IMPLEMENT_TASKS_CONTENT" | grep -qE "tasks\.md"; then
        CYCLE_ISSUES="${CYCLE_ISSUES}\nmissing_dependency:implement-tasks→create-tasks:$IMPLEMENT_TASKS_FILE:implement-tasks should reference create-tasks outputs (tasks.md)"
    fi
    
    if ! echo "$ORCHESTRATE_TASKS_CONTENT" | grep -qE "tasks\.md"; then
        CYCLE_ISSUES="${CYCLE_ISSUES}\nmissing_dependency:orchestrate-tasks→create-tasks:$ORCHESTRATE_TASKS_FILE:orchestrate-tasks should reference create-tasks outputs (tasks.md)"
    fi
fi
```

### Step 7: Check for Broken References

Check for missing connections or broken references:

```bash
# Check for broken file references
COMMAND_FILES=$(find "$SCAN_PATH/commands" -name "*.md" -type f 2>/dev/null)

echo "$COMMAND_FILES" | while read file_path; do
    if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
        continue
    fi
    
    FILE_CONTENT=$(cat "$file_path")
    
    # Check for references to non-existent files
    if echo "$FILE_CONTENT" | grep -qE "@geist/commands/[^/]+/[^/]+\.md|@geist/workflows/[^/]+/[^/]+\.md"; then
        REFERENCED_FILES=$(grep -oE "@geist/commands/[^/]+/[^/]+\.md|@geist/workflows/[^/]+/[^/]+\.md" "$file_path" | sed 's|@geist/||')
        
        for ref_file in $REFERENCED_FILES; do
            if [ ! -f "$SCAN_PATH/$ref_file" ]; then
                LINE_NUMBER=$(grep -n "$ref_file" "$file_path" | head -1 | cut -d: -f1)
                CYCLE_ISSUES="${CYCLE_ISSUES}\nbroken_reference:$file_path:$LINE_NUMBER:Reference to non-existent file: $ref_file"
            fi
        done
    fi
done
```

### Step 8: Generate Validation Report

Generate comprehensive command cycle validation report:

```bash
# Count findings
FLOW_ISSUES_COUNT=$(echo "$CYCLE_ISSUES" | grep -c "flow_issue" || echo "0")
DEPENDENCY_ISSUES_COUNT=$(echo "$CYCLE_ISSUES" | grep -c "dependency_issue" || echo "0")
MISSING_DEPENDENCY_COUNT=$(echo "$CYCLE_ISSUES" | grep -c "missing_dependency" || echo "0")
BROKEN_REFERENCE_COUNT=$(echo "$CYCLE_ISSUES" | grep -c "broken_reference" || echo "0")
TOTAL_COUNT=$((FLOW_ISSUES_COUNT + DEPENDENCY_ISSUES_COUNT + MISSING_DEPENDENCY_COUNT + BROKEN_REFERENCE_COUNT))

# Create JSON report
cat > "$CACHE_PATH/command-cycle-validation.json" << EOF
{
  "validation_context": "$VALIDATION_CONTEXT",
  "scan_path": "$SCAN_PATH",
  "scan_timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "total_issues_found": $TOTAL_COUNT,
  "categories": {
    "flow_issues": {
      "count": $FLOW_ISSUES_COUNT,
      "issues": [
$(echo "$CYCLE_ISSUES" | grep "flow_issue" | while IFS=':' read -r type flow file_path description; do
    echo "        {\"type\": \"$type\", \"flow\": \"$flow\", \"file\": \"$file_path\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ]
    },
    "dependency_issues": {
      "count": $DEPENDENCY_ISSUES_COUNT,
      "issues": [
$(echo "$CYCLE_ISSUES" | grep "dependency_issue" | while IFS=':' read -r type flow file_path description; do
    echo "        {\"type\": \"$type\", \"flow\": \"$flow\", \"file\": \"$file_path\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ]
    },
    "missing_dependencies": {
      "count": $MISSING_DEPENDENCY_COUNT,
      "issues": [
$(echo "$CYCLE_ISSUES" | grep "missing_dependency" | while IFS=':' read -r type flow file_path description; do
    echo "        {\"type\": \"$type\", \"flow\": \"$flow\", \"file\": \"$file_path\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ]
    },
    "broken_references": {
      "count": $BROKEN_REFERENCE_COUNT,
      "issues": [
$(echo "$CYCLE_ISSUES" | grep "broken_reference" | while IFS=':' read -r type file_path line_number description; do
    echo "        {\"type\": \"$type\", \"file\": \"$file_path\", \"line\": \"$line_number\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ]
    }
  }
}
EOF

# Create markdown summary report
cat > "$CACHE_PATH/command-cycle-validation-summary.md" << EOF
# Command Cycle Validation Report

**Validation Context:** $VALIDATION_CONTEXT  
**Scan Path:** $SCAN_PATH  
**Scan Timestamp:** $(date -u +%Y-%m-%dT%H:%M:%SZ)

## Summary

- **Total Issues Found:** $TOTAL_COUNT
- **Flow Issues:** $FLOW_ISSUES_COUNT
- **Dependency Issues:** $DEPENDENCY_ISSUES_COUNT
- **Missing Dependencies:** $MISSING_DEPENDENCY_COUNT
- **Broken References:** $BROKEN_REFERENCE_COUNT

## Flow Issues

$(echo "$CYCLE_ISSUES" | grep "flow_issue" | while IFS=':' read -r type flow file_path description; do
    echo "- **$flow**"
    echo "  - File: \`$file_path\`"
    echo "  - Issue: $description"
    echo ""
done)

## Dependency Issues

$(echo "$CYCLE_ISSUES" | grep "dependency_issue" | while IFS=':' read -r type flow file_path description; do
    echo "- **$flow**"
    echo "  - File: \`$file_path\`"
    echo "  - Issue: $description"
    echo ""
done)

## Missing Dependencies

$(echo "$CYCLE_ISSUES" | grep "missing_dependency" | while IFS=':' read -r type flow file_path description; do
    echo "- **$flow**"
    echo "  - File: \`$file_path\`"
    echo "  - Issue: $description"
    echo ""
done)

## Broken References

$(echo "$CYCLE_ISSUES" | grep "broken_reference" | while IFS=':' read -r type file_path line_number description; do
    echo "- **$description**"
    echo "  - File: \`$file_path\`"
    echo "  - Line: $line_number"
    echo ""
done)
EOF

echo "✅ Command cycle validation complete. Report stored in $CACHE_PATH/"
```

## Important Constraints

- Must verify all command flow connections (shape-spec → write-spec → create-tasks → implement-tasks/orchestrate-tasks)
- Must verify implement-tasks and orchestrate-tasks are independent (both depend on create-tasks, not each other)
- Must check for missing connections or broken references
- Must verify specialized commands maintain same cycle structure as profiles/default versions
- **CRITICAL**: All reports must be stored in `geist/specs/[current-spec]/implementation/cache/validation/` when running within a spec command, not scattered around the codebase
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
