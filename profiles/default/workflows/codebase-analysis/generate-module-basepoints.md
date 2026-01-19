# Module Basepoint Generation

## Core Responsibilities

1. **Load Task List**: Retrieve the basepoints task list from previous phase
2. **Generate Basepoints One-by-One**: Create each basepoint with progress tracking
3. **Track Progress**: Update task list as each basepoint is created
4. **Verify Coverage**: Ensure ALL modules have basepoints (no misses)
5. **Report Completion**: Provide summary of all generated basepoints

## Workflow

### Step 1: Load Module Folders and Task List

Load module folders and their task list from cache:

```bash
# Load module folders
if [ ! -f "geist/output/create-basepoints/cache/module-folders.txt" ]; then
    echo "❌ ERROR: Module folders list not found. Run mirror-project-structure first."
    exit 1
fi

MODULE_FOLDERS=$(cat geist/output/create-basepoints/cache/module-folders.txt | grep -v "^#" | grep -v "^$")
TOTAL_MODULES=$(echo "$MODULE_FOLDERS" | wc -l | tr -d ' ')

echo "📋 Loaded $TOTAL_MODULES modules from task list"

# Initialize progress tracking
echo "# Basepoints Generation Progress" > geist/output/create-basepoints/cache/generation-progress.md
echo "" >> geist/output/create-basepoints/cache/generation-progress.md
echo "Started: $(date)" >> geist/output/create-basepoints/cache/generation-progress.md
echo "" >> geist/output/create-basepoints/cache/generation-progress.md
```

### Step 2: Generate Basepoint for Each Module (One-by-One)

Process each module deterministically, tracking progress:

```bash
CURRENT=0
GENERATED=()
FAILED=()

echo "$MODULE_FOLDERS" | while read module_dir; do
    if [ -z "$module_dir" ]; then
        continue
    fi
    
    CURRENT=$((CURRENT + 1))
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "📦 Processing module $CURRENT/$TOTAL_MODULES: $module_dir"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    
    # Normalize module directory path
    NORMALIZED_DIR=$(echo "$module_dir" | sed 's|^\./||' | sed 's|^\.$||')
    
    # Handle root-level modules
    if [ -z "$NORMALIZED_DIR" ] || [ "$NORMALIZED_DIR" = "." ]; then
        PROJECT_ROOT_NAME=$(basename "$(pwd)" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9-]/-/g')
        BASEPOINT_DIR="geist/basepoints"
        MODULE_NAME="$PROJECT_ROOT_NAME"
    else
        BASEPOINT_DIR="geist/basepoints/$NORMALIZED_DIR"
        MODULE_NAME=$(basename "$NORMALIZED_DIR")
    fi
    
    BASEPOINT_FILE="$BASEPOINT_DIR/agent-base-$MODULE_NAME.md"
    
    # Create directory if needed
    mkdir -p "$BASEPOINT_DIR"
    
    # Generate basepoint content
    echo "   → Analyzing module: $module_dir"
    echo "   → Creating basepoint: $BASEPOINT_FILE"
    
    # [Analyze module and generate content here]
    
    # Log progress
    echo "- [x] **$CURRENT/$TOTAL_MODULES**: \`$module_dir\` → \`$BASEPOINT_FILE\`" >> geist/output/create-basepoints/cache/generation-progress.md
    
    echo "   ✅ Created: $BASEPOINT_FILE"
done
```

### Step 3: Generate Basepoint Content

For each module, create comprehensive basepoint content including:

```markdown
# Basepoint: [Module Name]

## Module Overview
- **Location**: `/path/to/module/`
- **Purpose**: [Extracted from analysis]
- **Layer**: [Abstraction layer]

## Files in Module
| File | Purpose |
|------|---------|
| [files...] | [purposes...] |

## Patterns
### Design Patterns
[Extracted patterns]

### Coding Patterns
[Extracted patterns]

## Standards
[Extracted standards]

## Flows
### Data Flow
[Extracted flows]

### Control Flow
[Extracted flows]

## Strategies
[Extracted strategies]

## Testing
[Extracted testing approaches]

## Related Modules
[Dependencies and relationships]
```

### Step 4: Verify Complete Coverage

After generating all module basepoints, verify complete coverage:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "🔍 VERIFICATION: Checking basepoint coverage"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Count expected vs actual
EXPECTED_COUNT=$(cat geist/output/create-basepoints/cache/module-folders.txt | grep -v "^#" | grep -v "^$" | wc -l | tr -d ' ')
ACTUAL_COUNT=$(find geist/basepoints -name "agent-base-*.md" -type f | wc -l | tr -d ' ')

echo "Expected module basepoints: $EXPECTED_COUNT"
echo "Actual module basepoints: $ACTUAL_COUNT"

# List all generated basepoints
echo "" >> geist/output/create-basepoints/cache/generation-progress.md
echo "## Generated Basepoints" >> geist/output/create-basepoints/cache/generation-progress.md
echo "" >> geist/output/create-basepoints/cache/generation-progress.md
find geist/basepoints -name "agent-base-*.md" -type f | sort | while read bp; do
    echo "- \`$bp\`" >> geist/output/create-basepoints/cache/generation-progress.md
done

# Check for missing basepoints
echo "" >> geist/output/create-basepoints/cache/generation-progress.md
echo "## Coverage Check" >> geist/output/create-basepoints/cache/generation-progress.md
echo "" >> geist/output/create-basepoints/cache/generation-progress.md

MISSING_COUNT=0
while read module_dir; do
    if [ -z "$module_dir" ]; then
        continue
    fi
    
    NORMALIZED_DIR=$(echo "$module_dir" | sed 's|^\./||' | sed 's|^\.$||')
    
    if [ -z "$NORMALIZED_DIR" ] || [ "$NORMALIZED_DIR" = "." ]; then
        PROJECT_ROOT_NAME=$(basename "$(pwd)" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9-]/-/g')
        EXPECTED_FILE="geist/basepoints/agent-base-$PROJECT_ROOT_NAME.md"
    else
        MODULE_NAME=$(basename "$NORMALIZED_DIR")
        EXPECTED_FILE="geist/basepoints/$NORMALIZED_DIR/agent-base-$MODULE_NAME.md"
    fi
    
    if [ ! -f "$EXPECTED_FILE" ]; then
        echo "❌ MISSING: $module_dir → $EXPECTED_FILE" >> geist/output/create-basepoints/cache/generation-progress.md
        MISSING_COUNT=$((MISSING_COUNT + 1))
    fi
done < geist/output/create-basepoints/cache/module-folders.txt

if [ "$MISSING_COUNT" -gt 0 ]; then
    echo "❌ VERIFICATION FAILED: $MISSING_COUNT basepoints missing!"
    echo "   Check: geist/output/create-basepoints/cache/generation-progress.md"
else
    echo "✅ VERIFICATION PASSED: All $EXPECTED_COUNT module basepoints created"
    echo "✅ All modules have basepoints" >> geist/output/create-basepoints/cache/generation-progress.md
fi
```

### Step 5: Output Generation Summary

Display final summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 MODULE BASEPOINTS GENERATION COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Total modules processed: [N]
✅ Basepoints created: [N]
❌ Failed: [N] (if any)

📁 Basepoints location: geist/basepoints/
📋 Progress report: geist/output/create-basepoints/cache/generation-progress.md

Next: Parent basepoints will be generated to complete the hierarchy.
```

## Important Constraints

- **MUST process ALL modules from task list** - no exceptions
- **MUST track progress** for each module processed
- **MUST verify coverage** after completion
- **MUST report any missing basepoints** as errors
- Must generate one basepoint file per module folder
- Must place files in mirrored structure within basepoints folder
- Must name files based on actual module names from folder structure
- Must include all extracted patterns, standards, flows, strategies, and testing
- Must reference the module's abstraction layer
- Must save basepoint files for use in parent basepoint generation
