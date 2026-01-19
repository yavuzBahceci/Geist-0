# Project Structure Alignment Validation Utility

## Core Responsibilities

1. **Verify Specialized Commands Align with Project Structure**: Verify specialized commands align with actual project folder structure
2. **Verify Abstraction Layers Usage**: Verify abstraction layers from basepoints are correctly used in specialized commands
3. **Verify Complexity Assessment**: Check that project complexity assessment correctly adapts command structure
4. **Validate Checkpoints Match Complexity**: Validate command checkpoints match project complexity (simple/moderate/complex)

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

### Step 2: Extract Actual Project Structure (if validating installed geist)

If validating installed geist, extract actual project structure from basepoints:

```bash
# Initialize results
STRUCTURE_ALIGNMENT_ISSUES=""

if [ "$VALIDATION_CONTEXT" = "installed-geist" ]; then
    # Extract actual project structure from basepoints
    if [ -d "geist/basepoints" ] && [ -f "geist/basepoints/headquarter.md" ]; then
        # Extract abstraction layers
        ACTUAL_LAYERS=$(find geist/basepoints -type d -mindepth 1 -maxdepth 2 | sed 's|geist/basepoints/||' | cut -d'/' -f1 | sort -u)
        
        # Extract module hierarchy
        ACTUAL_MODULES=$(find geist/basepoints -name "agent-base-*.md" -type f | sed 's|geist/basepoints/||' | sed 's|/agent-base-.*\.md||')
        
        # Extract project folder structure (from headquarter.md if available)
        if [ -f "geist/basepoints/headquarter.md" ]; then
            HEADQUARTER_CONTENT=$(cat geist/basepoints/headquarter.md)
            # Extract project structure section if present
            PROJECT_STRUCTURE_SECTION=$(echo "$HEADQUARTER_CONTENT" | grep -A 50 -i "project.*structure\|folder.*structure" || echo "")
        fi
        
        # Determine project complexity
        MODULE_COUNT=$(find geist/basepoints -name "agent-base-*.md" -type f | wc -l | tr -d ' ')
        LAYER_COUNT=$(echo "$ACTUAL_LAYERS" | wc -l | tr -d ' ')
        
        # Calculate complexity
        COMPLEXITY_SCORE=0
        if [ "$MODULE_COUNT" -gt 20 ]; then
            COMPLEXITY_SCORE=$((COMPLEXITY_SCORE + 2))
        elif [ "$MODULE_COUNT" -gt 10 ]; then
            COMPLEXITY_SCORE=$((COMPLEXITY_SCORE + 1))
        fi
        
        if [ "$LAYER_COUNT" -gt 4 ]; then
            COMPLEXITY_SCORE=$((COMPLEXITY_SCORE + 2))
        elif [ "$LAYER_COUNT" -gt 2 ]; then
            COMPLEXITY_SCORE=$((COMPLEXITY_SCORE + 1))
        fi
        
        if [ "$COMPLEXITY_SCORE" -ge 3 ]; then
            PROJECT_COMPLEXITY="complex"
        elif [ "$COMPLEXITY_SCORE" -eq 0 ]; then
            PROJECT_COMPLEXITY="simple"
        else
            PROJECT_COMPLEXITY="moderate"
        fi
    else
        STRUCTURE_ALIGNMENT_ISSUES="${STRUCTURE_ALIGNMENT_ISSUES}\nissue:Basepoints not found - cannot validate project structure alignment"
    fi
fi
```

### Step 3: Verify Specialized Commands Align with Project Structure

Check that specialized commands reference actual project structure:

```bash
if [ "$VALIDATION_CONTEXT" = "installed-geist" ] && [ -n "$ACTUAL_LAYERS" ]; then
    # Find command files to check
    COMMAND_FILES=$(find "$SCAN_PATH/commands" -name "*.md" -type f 2>/dev/null)
    
    echo "$COMMAND_FILES" | while read file_path; do
        if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
            continue
        fi
        
        FILE_CONTENT=$(cat "$file_path")
        
        # Check if command references actual project layers
        REFERENCED_LAYERS=""
        for layer in $ACTUAL_LAYERS; do
            if echo "$FILE_CONTENT" | grep -qi "$layer"; then
                REFERENCED_LAYERS="${REFERENCED_LAYERS} $layer"
            fi
        done
        
        # If command mentions layers but doesn't reference actual layers, flag as issue
        if echo "$FILE_CONTENT" | grep -qiE "layer|abstraction" && [ -z "$REFERENCED_LAYERS" ]; then
            LINE_NUMBER=$(grep -niE "layer|abstraction" "$file_path" | head -1 | cut -d: -f1)
            STRUCTURE_ALIGNMENT_ISSUES="${STRUCTURE_ALIGNMENT_ISSUES}\nalignment_issue:$file_path:$LINE_NUMBER:Command mentions layers but doesn't reference actual project layers from basepoints"
        fi
        
        # Check if command references template paths instead of actual project paths
        if echo "$FILE_CONTENT" | grep -qE "profiles/default|template.*path"; then
            LINE_NUMBER=$(grep -nE "profiles/default|template.*path" "$file_path" | head -1 | cut -d: -f1)
            STRUCTURE_ALIGNMENT_ISSUES="${STRUCTURE_ALIGNMENT_ISSUES}\nalignment_issue:$file_path:$LINE_NUMBER:Command references template paths instead of actual project paths"
        fi
    done
fi
```

### Step 4: Verify Complexity Assessment Adapts Command Structure

Check that project complexity assessment correctly adapts command structure:

```bash
if [ "$VALIDATION_CONTEXT" = "installed-geist" ] && [ -n "$PROJECT_COMPLEXITY" ]; then
    # Check command files for complexity-appropriate structure
    COMMAND_FILES=$(find "$SCAN_PATH/commands" -name "*.md" -type f 2>/dev/null)
    
    echo "$COMMAND_FILES" | while read file_path; do
        if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
            continue
        fi
        
        FILE_CONTENT=$(cat "$file_path")
        
        # Count checkpoints in command
        CHECKPOINT_COUNT=$(echo "$FILE_CONTENT" | grep -cE "checkpoint|Checkpoint|CHECKPOINT" || echo "0")
        
        # Verify checkpoint count matches complexity
        if [ "$PROJECT_COMPLEXITY" = "simple" ] && [ "$CHECKPOINT_COUNT" -gt 5 ]; then
            STRUCTURE_ALIGNMENT_ISSUES="${STRUCTURE_ALIGNMENT_ISSUES}\ncomplexity_issue:$file_path:Command has too many checkpoints ($CHECKPOINT_COUNT) for simple project (expected ≤5)"
        elif [ "$PROJECT_COMPLEXITY" = "complex" ] && [ "$CHECKPOINT_COUNT" -lt 3 ]; then
            STRUCTURE_ALIGNMENT_ISSUES="${STRUCTURE_ALIGNMENT_ISSUES}\ncomplexity_issue:$file_path:Command has too few checkpoints ($CHECKPOINT_COUNT) for complex project (expected ≥3)"
        fi
        
        # Check for complexity-specific adaptations
        if [ "$PROJECT_COMPLEXITY" = "simple" ] && echo "$FILE_CONTENT" | grep -qE "granular|detailed.*validation|multiple.*phases"; then
            LINE_NUMBER=$(grep -nE "granular|detailed.*validation|multiple.*phases" "$file_path" | head -1 | cut -d: -f1)
            STRUCTURE_ALIGNMENT_ISSUES="${STRUCTURE_ALIGNMENT_ISSUES}\ncomplexity_issue:$file_path:$LINE_NUMBER:Command may have unnecessary complexity for simple project"
        fi
    done
fi
```

### Step 5: Verify Module Hierarchy References

Check that module hierarchy from basepoints is correctly referenced:

```bash
if [ "$VALIDATION_CONTEXT" = "installed-geist" ] && [ -n "$ACTUAL_MODULES" ]; then
    # Check if commands reference actual module hierarchy
    COMMAND_FILES=$(find "$SCAN_PATH/commands" -name "*.md" -type f 2>/dev/null)
    
    echo "$COMMAND_FILES" | while read file_path; do
        if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
            continue
        fi
        
        FILE_CONTENT=$(cat "$file_path")
        
        # Check if command references modules
        if echo "$FILE_CONTENT" | grep -qiE "module|component|service"; then
            # Check if it references actual modules from basepoints
            REFERENCED_MODULES=""
            for module_path in $ACTUAL_MODULES; do
                MODULE_NAME=$(basename "$module_path")
                if echo "$FILE_CONTENT" | grep -qi "$MODULE_NAME"; then
                    REFERENCED_MODULES="${REFERENCED_MODULES} $MODULE_NAME"
                fi
            done
            
            # If command mentions modules but doesn't reference actual modules, note it (not necessarily an issue)
            if [ -z "$REFERENCED_MODULES" ] && echo "$FILE_CONTENT" | grep -qiE "module.*pattern|module.*structure"; then
                # This is informational, not necessarily an issue
                STRUCTURE_ALIGNMENT_ISSUES="${STRUCTURE_ALIGNMENT_ISSUES}\ninfo:$file_path:Command mentions modules but may not reference specific modules from basepoints"
            fi
        fi
    done
fi
```

### Step 6: Generate Validation Report

Generate comprehensive project structure alignment validation report:

```bash
# Count findings
ALIGNMENT_ISSUES_COUNT=$(echo "$STRUCTURE_ALIGNMENT_ISSUES" | grep -c "alignment_issue" || echo "0")
COMPLEXITY_ISSUES_COUNT=$(echo "$STRUCTURE_ALIGNMENT_ISSUES" | grep -c "complexity_issue" || echo "0")
INFO_COUNT=$(echo "$STRUCTURE_ALIGNMENT_ISSUES" | grep -c "info" || echo "0")
OTHER_ISSUES_COUNT=$(echo "$STRUCTURE_ALIGNMENT_ISSUES" | grep -c "issue" || echo "0")
TOTAL_COUNT=$((ALIGNMENT_ISSUES_COUNT + COMPLEXITY_ISSUES_COUNT + INFO_COUNT + OTHER_ISSUES_COUNT))

# Create JSON report
cat > "$CACHE_PATH/project-structure-alignment-validation.json" << EOF
{
  "validation_context": "$VALIDATION_CONTEXT",
  "scan_path": "$SCAN_PATH",
  "scan_timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "project_complexity": "${PROJECT_COMPLEXITY:-unknown}",
  "module_count": ${MODULE_COUNT:-0},
  "layer_count": ${LAYER_COUNT:-0},
  "total_issues_found": $TOTAL_COUNT,
  "categories": {
    "alignment_issues": {
      "count": $ALIGNMENT_ISSUES_COUNT,
      "issues": [
$(echo "$STRUCTURE_ALIGNMENT_ISSUES" | grep "alignment_issue" | while IFS=':' read -r type file_path line_number description; do
    echo "        {\"type\": \"$type\", \"file\": \"$file_path\", \"line\": \"$line_number\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ]
    },
    "complexity_issues": {
      "count": $COMPLEXITY_ISSUES_COUNT,
      "issues": [
$(echo "$STRUCTURE_ALIGNMENT_ISSUES" | grep "complexity_issue" | while IFS=':' read -r type file_path line_number description; do
    if [ -n "$line_number" ] && [ "$line_number" != "$file_path" ]; then
        echo "        {\"type\": \"$type\", \"file\": \"$file_path\", \"line\": \"$line_number\", \"description\": \"$description\"},"
    else
        echo "        {\"type\": \"$type\", \"file\": \"$file_path\", \"description\": \"$description\"},"
    fi
done | sed '$ s/,$//')
      ]
    },
    "informational": {
      "count": $INFO_COUNT,
      "items": [
$(echo "$STRUCTURE_ALIGNMENT_ISSUES" | grep "info" | while IFS=':' read -r type file_path description; do
    echo "        {\"type\": \"$type\", \"file\": \"$file_path\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ]
    },
    "other_issues": {
      "count": $OTHER_ISSUES_COUNT,
      "issues": [
$(echo "$STRUCTURE_ALIGNMENT_ISSUES" | grep "^issue:" | while IFS=':' read -r type description; do
    echo "        {\"type\": \"$type\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ]
    }
  }
}
EOF

# Create markdown summary report
cat > "$CACHE_PATH/project-structure-alignment-validation-summary.md" << EOF
# Project Structure Alignment Validation Report

**Validation Context:** $VALIDATION_CONTEXT  
**Scan Path:** $SCAN_PATH  
**Scan Timestamp:** $(date -u +%Y-%m-%dT%H:%M:%SZ)

## Project Information

- **Project Complexity:** ${PROJECT_COMPLEXITY:-unknown}
- **Module Count:** ${MODULE_COUNT:-0}
- **Layer Count:** ${LAYER_COUNT:-0}

## Summary

- **Total Issues Found:** $TOTAL_COUNT
- **Alignment Issues:** $ALIGNMENT_ISSUES_COUNT
- **Complexity Issues:** $COMPLEXITY_ISSUES_COUNT
- **Informational Items:** $INFO_COUNT
- **Other Issues:** $OTHER_ISSUES_COUNT

## Alignment Issues

$(echo "$STRUCTURE_ALIGNMENT_ISSUES" | grep "alignment_issue" | while IFS=':' read -r type file_path line_number description; do
    echo "- **$description**"
    echo "  - File: \`$file_path\`"
    echo "  - Line: $line_number"
    echo ""
done)

## Complexity Issues

$(echo "$STRUCTURE_ALIGNMENT_ISSUES" | grep "complexity_issue" | while IFS=':' read -r type file_path line_number description; do
    if [ -n "$line_number" ] && [ "$line_number" != "$file_path" ]; then
        echo "- **$description**"
        echo "  - File: \`$file_path\`"
        echo "  - Line: $line_number"
        echo ""
    else
        echo "- **$description**"
        echo "  - File: \`$file_path\`"
        echo ""
    fi
done)

## Informational Items

$(echo "$STRUCTURE_ALIGNMENT_ISSUES" | grep "info" | while IFS=':' read -r type file_path description; do
    echo "- **$description**"
    echo "  - File: \`$file_path\`"
    echo ""
done)

## Other Issues

$(echo "$STRUCTURE_ALIGNMENT_ISSUES" | grep "^issue:" | while IFS=':' read -r type description; do
    echo "- **$description**"
    echo ""
done)
EOF

echo "✅ Project structure alignment validation complete. Report stored in $CACHE_PATH/"
```

## Important Constraints

- Must verify specialized commands align with actual project folder structure
- Must verify abstraction layers from basepoints are correctly used in specialized commands
- Must check that project complexity assessment correctly adapts command structure
- Must validate command checkpoints match project complexity (simple/moderate/complex)
- Must verify module hierarchy from basepoints is correctly referenced
- **CRITICAL**: All reports must be stored in `geist/specs/[current-spec]/implementation/cache/validation/` when running within a spec command, not scattered around the codebase
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
