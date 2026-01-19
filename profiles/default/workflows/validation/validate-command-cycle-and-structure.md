# Command Cycle and Structure Validation

## Core Responsibilities

1. **Comprehensive Command Cycle Validation**: Verify all command flow connections and dependencies
2. **Project Structure Alignment Validation**: Verify specialized commands align with actual project structure
3. **Technology Transition Validation**: Verify proper transformation from technology-agnostic to codebase-specialized
4. **Generate Combined Report**: Create comprehensive report combining all validation findings

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

echo "🔍 Starting command cycle and structure validation for: $VALIDATION_CONTEXT"
```

### Step 2: Run Command Cycle Validation

Use the command cycle validation utility from Task Group 1:

```bash
# Run command cycle validation utility
{{workflows/validation/validate-command-cycle}}

# Load detection results
if [ -f "$CACHE_PATH/command-cycle-validation.json" ]; then
    CYCLE_VALIDATION_DATA=$(cat "$CACHE_PATH/command-cycle-validation.json")
else
    echo "⚠️  Warning: Command cycle validation results not found"
    CYCLE_VALIDATION_DATA="{}"
fi
```

### Step 3: Run Project Structure Alignment Validation

Use the project structure alignment validation utility from Task Group 1:

```bash
# Run project structure alignment validation utility
{{workflows/validation/validate-project-structure-alignment}}

# Load detection results
if [ -f "$CACHE_PATH/project-structure-alignment-validation.json" ]; then
    STRUCTURE_VALIDATION_DATA=$(cat "$CACHE_PATH/project-structure-alignment-validation.json")
else
    echo "⚠️  Warning: Project structure alignment validation results not found"
    STRUCTURE_VALIDATION_DATA="{}"
fi
```

### Step 4: Run Technology Transition Validation

Use the technology-agnostic validation utility from Task Group 1:

```bash
# Run technology-agnostic validation utility
{{workflows/validation/validate-technology-agnostic}}

# Load detection results
if [ -f "$CACHE_PATH/technology-agnostic-validation.json" ]; then
    TECH_VALIDATION_DATA=$(cat "$CACHE_PATH/technology-agnostic-validation.json")
else
    echo "⚠️  Warning: Technology transition validation results not found"
    TECH_VALIDATION_DATA="{}"
fi
```

### Step 5: Implement Command Cycle Flow Validation

Verify all command flow connections:

```bash
# Initialize results
CYCLE_FLOW_ISSUES=""

# Define command files
SHAPE_SPEC_FILE="$SCAN_PATH/commands/shape-spec/single-agent/shape-spec.md"
WRITE_SPEC_FILE="$SCAN_PATH/commands/write-spec/single-agent/write-spec.md"
CREATE_TASKS_FILE="$SCAN_PATH/commands/create-tasks/single-agent/create-tasks.md"
IMPLEMENT_TASKS_FILE="$SCAN_PATH/commands/implement-tasks/single-agent/implement-tasks.md"
ORCHESTRATE_TASKS_FILE="$SCAN_PATH/commands/orchestrate-tasks/orchestrate-tasks.md"

# Verify shape-spec → write-spec flow
if [ -f "$SHAPE_SPEC_FILE" ] && [ -f "$WRITE_SPEC_FILE" ]; then
    WRITE_SPEC_CONTENT=$(cat "$WRITE_SPEC_FILE")
    
    if ! echo "$WRITE_SPEC_CONTENT" | grep -qE "spec\.md|requirements\.md|planning/"; then
        CYCLE_FLOW_ISSUES="${CYCLE_FLOW_ISSUES}\nflow_issue:shape-spec→write-spec:$WRITE_SPEC_FILE:write-spec may not reference shape-spec outputs (spec.md, requirements.md)"
    fi
fi

# Verify write-spec → create-tasks flow
if [ -f "$WRITE_SPEC_FILE" ] && [ -f "$CREATE_TASKS_FILE" ]; then
    CREATE_TASKS_CONTENT=$(cat "$CREATE_TASKS_FILE")
    
    if ! echo "$CREATE_TASKS_CONTENT" | grep -qE "spec\.md|requirements\.md"; then
        CYCLE_FLOW_ISSUES="${CYCLE_FLOW_ISSUES}\nflow_issue:write-spec→create-tasks:$CREATE_TASKS_FILE:create-tasks may not reference write-spec outputs (spec.md, requirements.md)"
    fi
fi

# Verify create-tasks → implement-tasks flow
if [ -f "$CREATE_TASKS_FILE" ] && [ -f "$IMPLEMENT_TASKS_FILE" ]; then
    IMPLEMENT_TASKS_CONTENT=$(cat "$IMPLEMENT_TASKS_FILE")
    
    if ! echo "$IMPLEMENT_TASKS_CONTENT" | grep -qE "tasks\.md"; then
        CYCLE_FLOW_ISSUES="${CYCLE_FLOW_ISSUES}\nflow_issue:create-tasks→implement-tasks:$IMPLEMENT_TASKS_FILE:implement-tasks may not reference create-tasks outputs (tasks.md)"
    fi
fi

# Verify create-tasks → orchestrate-tasks flow
if [ -f "$CREATE_TASKS_FILE" ] && [ -f "$ORCHESTRATE_TASKS_FILE" ]; then
    ORCHESTRATE_TASKS_CONTENT=$(cat "$ORCHESTRATE_TASKS_FILE")
    
    if ! echo "$ORCHESTRATE_TASKS_CONTENT" | grep -qE "tasks\.md"; then
        CYCLE_FLOW_ISSUES="${CYCLE_FLOW_ISSUES}\nflow_issue:create-tasks→orchestrate-tasks:$ORCHESTRATE_TASKS_FILE:orchestrate-tasks may not reference create-tasks outputs (tasks.md)"
    fi
fi

# Verify implement-tasks and orchestrate-tasks are independent
if [ -f "$IMPLEMENT_TASKS_FILE" ] && [ -f "$ORCHESTRATE_TASKS_FILE" ]; then
    IMPLEMENT_TASKS_CONTENT=$(cat "$IMPLEMENT_TASKS_FILE")
    ORCHESTRATE_TASKS_CONTENT=$(cat "$ORCHESTRATE_TASKS_FILE")
    
    # Check if implement-tasks references orchestrate-tasks (should not)
    if echo "$IMPLEMENT_TASKS_CONTENT" | grep -qE "orchestrate-tasks|orchestration"; then
        CYCLE_FLOW_ISSUES="${CYCLE_FLOW_ISSUES}\ndependency_issue:implement-tasks→orchestrate-tasks:$IMPLEMENT_TASKS_FILE:implement-tasks should not depend on orchestrate-tasks (they are independent)"
    fi
    
    # Check if orchestrate-tasks references implement-tasks outputs (should not directly)
    if echo "$ORCHESTRATE_TASKS_CONTENT" | grep -qE "implement-tasks.*output|implement-tasks.*result"; then
        CYCLE_FLOW_ISSUES="${CYCLE_FLOW_ISSUES}\ndependency_issue:orchestrate-tasks→implement-tasks:$ORCHESTRATE_TASKS_FILE:orchestrate-tasks should not depend on implement-tasks outputs (they are independent)"
    fi
    
    # Both should reference create-tasks outputs (tasks.md)
    if ! echo "$IMPLEMENT_TASKS_CONTENT" | grep -qE "tasks\.md"; then
        CYCLE_FLOW_ISSUES="${CYCLE_FLOW_ISSUES}\nmissing_dependency:implement-tasks→create-tasks:$IMPLEMENT_TASKS_FILE:implement-tasks should reference create-tasks outputs (tasks.md)"
    fi
    
    if ! echo "$ORCHESTRATE_TASKS_CONTENT" | grep -qE "tasks\.md"; then
        CYCLE_FLOW_ISSUES="${CYCLE_FLOW_ISSUES}\nmissing_dependency:orchestrate-tasks→create-tasks:$ORCHESTRATE_TASKS_FILE:orchestrate-tasks should reference create-tasks outputs (tasks.md)"
    fi
fi

# Count cycle flow issues
CYCLE_FLOW_COUNT=$(echo "$CYCLE_FLOW_ISSUES" | grep -c . || echo "0")
```

### Step 6: Implement Project Structure Alignment Validation

Verify specialized commands align with actual project structure:

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
        
        # Check if commands reference actual project layers
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
            
            # Check checkpoint count matches complexity
            CHECKPOINT_COUNT=$(echo "$FILE_CONTENT" | grep -cE "checkpoint|Checkpoint|CHECKPOINT" || echo "0")
            
            if [ "$PROJECT_COMPLEXITY" = "simple" ] && [ "$CHECKPOINT_COUNT" -gt 5 ]; then
                STRUCTURE_ALIGNMENT_ISSUES="${STRUCTURE_ALIGNMENT_ISSUES}\ncomplexity_issue:$file_path:Command has too many checkpoints ($CHECKPOINT_COUNT) for simple project (expected ≤5)"
            elif [ "$PROJECT_COMPLEXITY" = "complex" ] && [ "$CHECKPOINT_COUNT" -lt 3 ]; then
                STRUCTURE_ALIGNMENT_ISSUES="${STRUCTURE_ALIGNMENT_ISSUES}\ncomplexity_issue:$file_path:Command has too few checkpoints ($CHECKPOINT_COUNT) for complex project (expected ≥3)"
            fi
        done
    else
        STRUCTURE_ALIGNMENT_ISSUES="${STRUCTURE_ALIGNMENT_ISSUES}\nissue:Basepoints not found - cannot validate project structure alignment"
    fi
fi

# Count structure alignment issues
STRUCTURE_ALIGNMENT_COUNT=$(echo "$STRUCTURE_ALIGNMENT_ISSUES" | grep -c . || echo "0")
```

### Step 7: Implement Technology Transition Validation

Verify proper transformation from technology-agnostic to codebase-specialized:

```bash
# Initialize results
TECH_TRANSITION_ISSUES=""

# Check for technology leaks in profiles/default (if validating template)
if [ "$VALIDATION_CONTEXT" = "profiles/default" ]; then
    FILES_TO_SCAN=$(find "$SCAN_PATH" -type f \( -name "*.md" \) ! -path "*/node_modules/*" ! -path "*/.git/*" ! -path "*/build/*" ! -path "*/dist/*" ! -path "*/cache/*")
    
    TECH_SPECIFIC_PATTERNS=(
        "React|Vue|Angular"
        "Express|FastAPI|Django|Rails"
        "TypeScript|JavaScript"
        "Python|Java|Go|Rust"
        "PostgreSQL|MySQL|MongoDB"
        "npm|yarn|pip|maven"
    )
    
    echo "$FILES_TO_SCAN" | while read file_path; do
        if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
            continue
        fi
        
        FILE_CONTENT=$(cat "$file_path")
        
        for pattern in "${TECH_SPECIFIC_PATTERNS[@]}"; do
            if echo "$FILE_CONTENT" | grep -qiE "$pattern"; then
                LINE_NUMBERS=$(grep -niE "$pattern" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
                TECH_TRANSITION_ISSUES="${TECH_TRANSITION_ISSUES}\nleak:$file_path:$LINE_NUMBERS:Technology-specific assumption: $pattern"
            fi
        done
    done
fi

# Check specialized commands reference actual project structure (if validating installed geist)
if [ "$VALIDATION_CONTEXT" = "installed-geist" ]; then
    COMMAND_FILES=$(find "$SCAN_PATH/commands" -name "*.md" -type f 2>/dev/null)
    
    echo "$COMMAND_FILES" | while read file_path; do
        if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
            continue
        fi
        
        FILE_CONTENT=$(cat "$file_path")
        
        # Check for template path references (should be replaced with actual paths)
        if echo "$FILE_CONTENT" | grep -qE "profiles/default|template.*path|abstract.*path"; then
            LINE_NUMBERS=$(grep -nE "profiles/default|template.*path|abstract.*path" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
            TECH_TRANSITION_ISSUES="${TECH_TRANSITION_ISSUES}\nissue:$file_path:$LINE_NUMBERS:Template path reference (should be actual project path)"
        fi
    done
fi

# Count technology transition issues
TECH_TRANSITION_COUNT=$(echo "$TECH_TRANSITION_ISSUES" | grep -c . || echo "0")
```

### Step 8: Generate Comprehensive Validation Report

Generate comprehensive command cycle and structure validation report:

```bash
# Calculate totals
TOTAL_ISSUES=$((CYCLE_FLOW_COUNT + STRUCTURE_ALIGNMENT_COUNT + TECH_TRANSITION_COUNT))

# Create JSON report
cat > "$CACHE_PATH/command-cycle-and-structure-validation.json" << EOF
{
  "validation_context": "$VALIDATION_CONTEXT",
  "scan_path": "$SCAN_PATH",
  "validation_timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "project_complexity": "${PROJECT_COMPLEXITY:-unknown}",
  "total_issues_found": $TOTAL_ISSUES,
  "categories": {
    "command_cycle_flow": {
      "count": $CYCLE_FLOW_COUNT,
      "issues": [
$(echo "$CYCLE_FLOW_ISSUES" | grep -v "^$" | while IFS=':' read -r type flow file_path description; do
    echo "        {\"type\": \"$type\", \"flow\": \"$flow\", \"file\": \"$file_path\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ]
    },
    "project_structure_alignment": {
      "count": $STRUCTURE_ALIGNMENT_COUNT,
      "issues": [
$(echo "$STRUCTURE_ALIGNMENT_ISSUES" | grep -v "^$" | while IFS=':' read -r type file_path line_number description; do
    if [ -n "$line_number" ] && [ "$line_number" != "$file_path" ]; then
        echo "        {\"type\": \"$type\", \"file\": \"$file_path\", \"line\": \"$line_number\", \"description\": \"$description\"},"
    else
        echo "        {\"type\": \"$type\", \"file\": \"$file_path\", \"description\": \"$description\"},"
    fi
done | sed '$ s/,$//')
      ]
    },
    "technology_transition": {
      "count": $TECH_TRANSITION_COUNT,
      "issues": [
$(echo "$TECH_TRANSITION_ISSUES" | grep -v "^$" | while IFS=':' read -r type file_path line_numbers description; do
    echo "        {\"type\": \"$type\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ]
    }
  }
}
EOF

# Create markdown summary report
cat > "$CACHE_PATH/command-cycle-and-structure-validation-summary.md" << EOF
# Command Cycle and Structure Validation Report

**Validation Context:** $VALIDATION_CONTEXT  
**Scan Path:** $SCAN_PATH  
**Validation Timestamp:** $(date -u +%Y-%m-%dT%H:%M:%SZ)  
**Project Complexity:** ${PROJECT_COMPLEXITY:-unknown}

## Summary

- **Total Issues Found:** $TOTAL_ISSUES
- **Command Cycle Flow Issues:** $CYCLE_FLOW_COUNT
- **Project Structure Alignment Issues:** $STRUCTURE_ALIGNMENT_COUNT
- **Technology Transition Issues:** $TECH_TRANSITION_COUNT

## Command Cycle Flow Validation

$(if [ "$CYCLE_FLOW_COUNT" -gt 0 ]; then
    echo "$CYCLE_FLOW_ISSUES" | grep -v "^$" | while IFS=':' read -r type flow file_path description; do
        echo "- **$flow**"
        echo "  - File: \`$file_path\`"
        echo "  - Issue: $description"
        echo ""
    done
else
    echo "✅ All command cycle flows verified correctly"
    echo ""
    echo "- shape-spec → write-spec: ✅ Verified"
    echo "- write-spec → create-tasks: ✅ Verified"
    echo "- create-tasks → implement-tasks: ✅ Verified"
    echo "- create-tasks → orchestrate-tasks: ✅ Verified"
    echo "- implement-tasks and orchestrate-tasks independence: ✅ Verified"
fi)

## Project Structure Alignment Validation

$(if [ "$STRUCTURE_ALIGNMENT_COUNT" -gt 0 ]; then
    echo "$STRUCTURE_ALIGNMENT_ISSUES" | grep -v "^$" | while IFS=':' read -r type file_path line_number description; do
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
    done
else
    echo "✅ All specialized commands align with project structure"
fi)

## Technology Transition Validation

$(if [ "$TECH_TRANSITION_COUNT" -gt 0 ]; then
    echo "$TECH_TRANSITION_ISSUES" | grep -v "^$" | while IFS=':' read -r type file_path line_numbers description; do
        echo "- **$description**"
        echo "  - File: \`$file_path\`"
        echo "  - Lines: $line_numbers"
        echo ""
    done
else
    if [ "$VALIDATION_CONTEXT" = "profiles/default" ]; then
        echo "✅ profiles/default commands are technology-agnostic"
    else
        echo "✅ Specialized commands properly reference actual project structure"
    fi
fi)

## Recommendations

$(if [ "$TOTAL_ISSUES" -gt 0 ]; then
    echo "### Action Required"
    echo ""
    echo "The following actions are recommended:"
    echo ""
    if [ "$CYCLE_FLOW_COUNT" -gt 0 ]; then
        echo "1. **Fix Command Cycle Flow Issues**: Ensure all commands correctly reference outputs from previous commands"
    fi
    if [ "$STRUCTURE_ALIGNMENT_COUNT" -gt 0 ]; then
        echo "2. **Align with Project Structure**: Update commands to reference actual project layers and modules from basepoints"
        echo "3. **Adjust Checkpoints**: Adapt command checkpoints to match project complexity"
    fi
    if [ "$TECH_TRANSITION_COUNT" -gt 0 ]; then
        if [ "$VALIDATION_CONTEXT" = "profiles/default" ]; then
            echo "4. **Remove Technology Leaks**: Remove technology-specific assumptions from profiles/default template"
        else
            echo "4. **Fix Template References**: Replace template path references with actual project paths"
        fi
    fi
else
    echo "✅ **No Action Required** - All validations passed!"
fi)
EOF

echo "✅ Command cycle and structure validation complete!"
echo "📊 Total issues found: $TOTAL_ISSUES"
echo "📁 Report stored in: $CACHE_PATH/command-cycle-and-structure-validation-summary.md"
```

## Important Constraints

- Must verify all command flow connections (shape-spec → write-spec → create-tasks → implement-tasks/orchestrate-tasks)
- Must verify implement-tasks and orchestrate-tasks are independent (both depend on create-tasks, not each other)
- Must verify specialized commands align with actual project structure from basepoints
- Must verify project complexity assessment correctly adapts command structure
- Must verify technology-agnostic → codebase-specialized transition
- Must check for technology leaks in profiles/default template
- Must verify specialized commands reference actual project paths, not template paths
- **CRITICAL**: All reports must be stored in `geist/specs/[current-spec]/implementation/cache/validation/` when running within a spec command, not scattered around the codebase
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
