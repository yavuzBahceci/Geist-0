Now that unnecessary logic is removed, proceed with fixing broken references by following these instructions:

## Core Responsibilities

1. **Load Command Cycle Validation Results**: Load command cycle validation results
2. **Fix Broken Command Cycle References**: Fix broken references between commands
3. **Align Commands with Project Structure**: Update commands to align with current project structure
4. **Support Dry-Run Mode**: Preview changes before applying
5. **Track Changes**: Track what was fixed for cleanup report

## Workflow

### Step 1: Load Command Cycle Validation Results

Load command cycle validation results:

```bash
# Load validation cache
VALIDATION_CACHE="geist/.cleanup-cache/validation"
CLEANUP_CACHE="geist/.cleanup-cache"
DRY_RUN="${DRY_RUN:-false}"

# Load command cycle validation results
if [ -f "$VALIDATION_CACHE/command-cycle-validation.json" ]; then
    CYCLE_DATA=$(cat "$VALIDATION_CACHE/command-cycle-validation.json")
    CYCLE_TOTAL=$(echo "$CYCLE_DATA" | grep -o '"total_issues_found":[0-9]*' | cut -d: -f2)
    
    if [ "$CYCLE_TOTAL" -eq 0 ]; then
        echo "✅ No broken references to fix"
        echo "0" > "$CLEANUP_CACHE/broken-references-fixed.txt"
        exit 0
    fi
    
    echo "🔧 Found $CYCLE_TOTAL broken reference(s) to fix"
else
    echo "❌ Command cycle validation results not found"
    exit 1
fi
```

### Step 2: Fix Broken Command Cycle References

Fix broken references between commands:

```bash
# Initialize cleanup tracking
BROKEN_REFERENCES_FIXED=0
FILES_MODIFIED=0

# Fix shape-spec → write-spec flow
SHAPE_SPEC_FILE="geist/commands/shape-spec/single-agent/shape-spec.md"
WRITE_SPEC_FILE="geist/commands/write-spec/single-agent/write-spec.md"

if [ -f "$WRITE_SPEC_FILE" ]; then
    WRITE_SPEC_CONTENT=$(cat "$WRITE_SPEC_FILE")
    ORIGINAL_CONTENT="$WRITE_SPEC_CONTENT"
    
    # Ensure write-spec references shape-spec outputs
    if ! echo "$WRITE_SPEC_CONTENT" | grep -qE "spec\.md|requirements\.md|planning/"; then
        # Add reference to spec.md (simplified - in practice would need more sophisticated insertion)
        WRITE_SPEC_CONTENT="${WRITE_SPEC_CONTENT}\n\n# References shape-spec outputs: spec.md, requirements.md"
        MODIFIED=true
        BROKEN_REFERENCES_FIXED=$((BROKEN_REFERENCES_FIXED + 1))
    fi
    
    if [ "$MODIFIED" = "true" ] && [ "$WRITE_SPEC_CONTENT" != "$ORIGINAL_CONTENT" ]; then
        if [ "$DRY_RUN" = "true" ]; then
            echo "🔍 [DRY-RUN] Would fix write-spec → shape-spec reference"
            mkdir -p "$(dirname "$CLEANUP_CACHE/dry-run-preview/$WRITE_SPEC_FILE")"
            echo "$WRITE_SPEC_CONTENT" > "$CLEANUP_CACHE/dry-run-preview/$WRITE_SPEC_FILE"
        else
            echo "$WRITE_SPEC_CONTENT" > "$WRITE_SPEC_FILE"
            echo "✅ Fixed write-spec → shape-spec reference"
            FILES_MODIFIED=$((FILES_MODIFIED + 1))
        fi
    fi
fi

# Fix write-spec → create-tasks flow
CREATE_TASKS_FILE="geist/commands/create-tasks/single-agent/create-tasks.md"

if [ -f "$CREATE_TASKS_FILE" ]; then
    CREATE_TASKS_CONTENT=$(cat "$CREATE_TASKS_FILE")
    ORIGINAL_CONTENT="$CREATE_TASKS_CONTENT"
    MODIFIED=false
    
    # Ensure create-tasks references write-spec outputs
    if ! echo "$CREATE_TASKS_CONTENT" | grep -qE "spec\.md|requirements\.md"; then
        CREATE_TASKS_CONTENT="${CREATE_TASKS_CONTENT}\n\n# References write-spec outputs: spec.md, requirements.md"
        MODIFIED=true
        BROKEN_REFERENCES_FIXED=$((BROKEN_REFERENCES_FIXED + 1))
    fi
    
    if [ "$MODIFIED" = "true" ] && [ "$CREATE_TASKS_CONTENT" != "$ORIGINAL_CONTENT" ]; then
        if [ "$DRY_RUN" = "true" ]; then
            echo "🔍 [DRY-RUN] Would fix create-tasks → write-spec reference"
            mkdir -p "$(dirname "$CLEANUP_CACHE/dry-run-preview/$CREATE_TASKS_FILE")"
            echo "$CREATE_TASKS_CONTENT" > "$CLEANUP_CACHE/dry-run-preview/$CREATE_TASKS_FILE"
        else
            echo "$CREATE_TASKS_CONTENT" > "$CREATE_TASKS_FILE"
            echo "✅ Fixed create-tasks → write-spec reference"
            FILES_MODIFIED=$((FILES_MODIFIED + 1))
        fi
    fi
fi

# Fix create-tasks → implement-tasks and orchestrate-tasks flows
IMPLEMENT_TASKS_FILE="geist/commands/implement-tasks/single-agent/implement-tasks.md"
ORCHESTRATE_TASKS_FILE="geist/commands/orchestrate-tasks/orchestrate-tasks.md"

for file_path in "$IMPLEMENT_TASKS_FILE" "$ORCHESTRATE_TASKS_FILE"; do
    if [ -f "$file_path" ]; then
        FILE_CONTENT=$(cat "$file_path")
        ORIGINAL_CONTENT="$FILE_CONTENT"
        MODIFIED=false
        
        # Ensure command references create-tasks outputs
        if ! echo "$FILE_CONTENT" | grep -qE "tasks\.md"; then
            FILE_CONTENT="${FILE_CONTENT}\n\n# References create-tasks outputs: tasks.md"
            MODIFIED=true
            BROKEN_REFERENCES_FIXED=$((BROKEN_REFERENCES_FIXED + 1))
        fi
        
        if [ "$MODIFIED" = "true" ] && [ "$FILE_CONTENT" != "$ORIGINAL_CONTENT" ]; then
            if [ "$DRY_RUN" = "true" ]; then
                echo "🔍 [DRY-RUN] Would fix reference in: $file_path"
                mkdir -p "$(dirname "$CLEANUP_CACHE/dry-run-preview/$file_path")"
                echo "$FILE_CONTENT" > "$CLEANUP_CACHE/dry-run-preview/$file_path"
            else
                echo "$FILE_CONTENT" > "$file_path"
                echo "✅ Fixed reference in: $file_path"
                FILES_MODIFIED=$((FILES_MODIFIED + 1))
            fi
        fi
    fi
done

# Store cleanup results
echo "$BROKEN_REFERENCES_FIXED" > "$CLEANUP_CACHE/broken-references-fixed.txt"
echo "$FILES_MODIFIED" > "$CLEANUP_CACHE/files-modified-references.txt"

if [ "$DRY_RUN" = "true" ]; then
    echo "🔍 [DRY-RUN] Would fix $BROKEN_REFERENCES_FIXED broken reference(s) in files"
else
    echo "✅ Fixed $BROKEN_REFERENCES_FIXED broken reference(s) in $FILES_MODIFIED file(s)"
fi
```

## Important Constraints

- Must use command cycle validation results from validation phase
- Must fix broken references between commands
- Must align commands with current project structure
- Must support dry-run mode to preview changes
- Must track what was fixed for cleanup report
- **CRITICAL**: All cleanup cache files must be stored in `geist/.cleanup-cache/` (temporary, cleaned up after cleanup completes)
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
