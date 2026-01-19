# Parent Basepoint Generation (Bottom-Up Completion)

## Core Responsibilities

1. **Identify Parent Folders**: Find parent folders that contain modules
2. **Order Parents by Depth**: Sort parents from deepest to shallowest (bottom-up)
3. **Load Child Basepoints**: Retrieve child module basepoint files as source
4. **Generate Parent Basepoints**: Create basepoint files for parent folders
5. **Aggregate Knowledge**: Combine patterns and knowledge from child modules
6. **Complete to Headquarter**: Ensure hierarchy reaches up to headquarter.md

## Bottom-Up Parent Generation Strategy

```
IMPORTANT: Generate parent basepoints starting from DEEPEST parents first.

This ensures:
1. Deepest parents aggregate from their already-generated children
2. Mid-level parents can aggregate from their children (which include lower-level aggregations)
3. Knowledge flows correctly up the hierarchy
4. Headquarter gets complete aggregated knowledge from all levels

Example for:
  src/
  ├── features/
  │   ├── auth/
  │   │   └── hooks/
  │   └── dashboard/
  └── shared/
      └── utils/

Parent Generation Order (after modules are done):
1. src/features/auth/      (depth 3 - aggregates hooks/)
2. src/features/           (depth 2 - aggregates auth/, dashboard/)
3. src/shared/             (depth 2 - aggregates utils/)
4. src/                    (depth 1 - aggregates features/, shared/)
5. headquarter.md          (aggregates all top-level)
```

## Workflow

### Step 1: Load Module Folders and Identify Parents

Load the list of module folders from cache and identify all parent directories:

```bash
if [ -f "geist/output/create-basepoints/cache/module-folders.txt" ]; then
    MODULE_FOLDERS=$(cat geist/output/create-basepoints/cache/module-folders.txt | grep -v "^#" | grep -v "^$")
else
    echo "❌ Module folders list not found. Run previous phases first."
    exit 1
fi

# Create temporary file to collect all parent directories
TEMP_PARENT_DIRS="geist/output/create-basepoints/cache/temp-parent-dirs.txt"
> "$TEMP_PARENT_DIRS"  # Clear file

echo "$MODULE_FOLDERS" | while read module_dir; do
    if [ -z "$module_dir" ]; then
        continue
    fi
    
    # Normalize module directory path
    NORMALIZED_DIR=$(echo "$module_dir" | sed 's|^\./||' | sed 's|^\.$||')
    
    # Skip root-level modules (handled separately)
    if [ -z "$NORMALIZED_DIR" ] || [ "$NORMALIZED_DIR" = "." ]; then
        continue
    fi
    
    # Traverse from module to root, collecting all parent directories
    current_path="$NORMALIZED_DIR"
    while [ "$current_path" != "." ] && [ -n "$current_path" ]; do
        parent_path=$(dirname "$current_path")
        
        # Normalize parent path
        if [ "$parent_path" = "." ] || [ "$parent_path" = "./" ]; then
            parent_path=""
        else
            parent_path=$(echo "$parent_path" | sed 's|^\./||')
        fi
        
        # Add parent directory to temp file if not empty
        if [ -n "$parent_path" ]; then
            echo "$parent_path" >> "$TEMP_PARENT_DIRS"
        fi
        
        # Move up one level
        current_path="$parent_path"
    done
done

# Remove duplicates and sort by depth (deepest first for bottom-up)
sort -u "$TEMP_PARENT_DIRS" | while read dir; do
    depth=$(echo "$dir" | tr -cd '/' | wc -c)
    echo "$depth|$dir"
done | sort -t'|' -k1 -rn | cut -d'|' -f2 > geist/output/create-basepoints/cache/parent-folders.txt

# Clean up temp file
rm -f "$TEMP_PARENT_DIRS"

TOTAL_PARENTS=$(cat geist/output/create-basepoints/cache/parent-folders.txt | wc -l | tr -d ' ')
echo "📋 Identified $TOTAL_PARENTS parent folders (sorted deepest-first)"
```

### Step 2: Generate Parent Basepoints Bottom-Up

Generate basepoints for all identified parent directories, starting from deepest and moving up:

```bash
CURRENT=0

cat geist/output/create-basepoints/cache/parent-folders.txt | while read parent_dir; do
    if [ -z "$parent_dir" ]; then
        continue
    fi
    
    CURRENT=$((CURRENT + 1))
    DEPTH=$(echo "$parent_dir" | tr -cd '/' | wc -c)
    
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "📁 Processing parent $CURRENT/$TOTAL_PARENTS: $parent_dir (depth: $DEPTH)"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    
    # Determine basepoint directory and name
    if [ -z "$parent_dir" ] || [ "$parent_dir" = "." ]; then
        BASEPOINT_DIR="geist/basepoints"
        PARENT_NAME="root"
    else
        BASEPOINT_DIR="geist/basepoints/$parent_dir"
        PARENT_NAME=$(basename "$parent_dir")
    fi
    
    PARENT_BASEPOINT="$BASEPOINT_DIR/agent-base-$PARENT_NAME.md"
    
    # Skip if basepoint already exists (may have been created as a module)
    if [ -f "$PARENT_BASEPOINT" ]; then
        echo "   ℹ️ Basepoint already exists, updating with child aggregation: $PARENT_BASEPOINT"
        # May need to enhance existing basepoint with child aggregation
    fi
    
    # Ensure directory exists
    mkdir -p "$BASEPOINT_DIR"
    
    # Find all direct child basepoints
    echo "   → Finding child basepoints to aggregate..."
    CHILD_BASEPOINTS=$(find "$BASEPOINT_DIR" -mindepth 2 -maxdepth 2 -name "agent-base-*.md" -type f 2>/dev/null)
    
    CHILD_COUNT=0
    if [ -n "$CHILD_BASEPOINTS" ]; then
        CHILD_COUNT=$(echo "$CHILD_BASEPOINTS" | wc -l | tr -d ' ')
        echo "   → Found $CHILD_COUNT child basepoints:"
        echo "$CHILD_BASEPOINTS" | while read child; do
            echo "      - $(basename "$(dirname "$child")")/$(basename "$child")"
        done
    else
        echo "   → No child basepoints found (leaf parent)"
    fi
    
    # Generate parent basepoint by aggregating from children
    echo "   → Creating parent basepoint: $PARENT_BASEPOINT"
    
    # [Generate content using aggregate_from_children - see Step 3]
    
    echo "   ✅ Created: $PARENT_BASEPOINT (aggregated from $CHILD_COUNT children)"
done
```

### Step 3: Aggregate Knowledge from Children

For each parent basepoint, aggregate knowledge from child basepoints:

```bash
aggregate_from_children() {
    local parent_dir="$1"
    local parent_name="$2"
    local basepoint_file="$3"
    
    # Find all child basepoints (direct children in this directory)
    if [ -z "$parent_dir" ] || [ "$parent_dir" = "." ]; then
        CHILD_BASEPOINTS=$(find geist/basepoints -maxdepth 2 -mindepth 2 -name "agent-base-*.md" -type f 2>/dev/null | grep -v "headquarter.md")
    else
        CHILD_BASEPOINTS=$(find "geist/basepoints/$parent_dir" -maxdepth 2 -mindepth 2 -name "agent-base-*.md" -type f 2>/dev/null)
    fi
    
    # Initialize aggregated content
    AGGREGATED_PATTERNS=""
    AGGREGATED_STANDARDS=""
    AGGREGATED_FLOWS=""
    AGGREGATED_STRATEGIES=""
    AGGREGATED_TESTING=""
    AGGREGATED_LIBRARIES=""
    CHILD_LIST=""
    
    # Temporary file for library aggregation
    TEMP_LIBRARIES_FILE="/tmp/aggregated-libraries-$parent_name.txt"
    > "$TEMP_LIBRARIES_FILE"
    
    # Read each child basepoint and extract knowledge
    echo "$CHILD_BASEPOINTS" | while read child_file; do
        if [ -z "$child_file" ] || [ ! -f "$child_file" ]; then
            continue
        fi
        
        CHILD_NAME=$(basename "$child_file" | sed 's/agent-base-//' | sed 's/.md//')
        CHILD_DIR=$(dirname "$child_file" | sed "s|geist/basepoints/$parent_dir/||" | sed 's|geist/basepoints/||')
        
        # Add to child list
        CHILD_LIST="${CHILD_LIST}\n| $CHILD_NAME | [$CHILD_NAME](./$CHILD_DIR/agent-base-$CHILD_NAME.md) |"
        
        # Extract patterns section
        PATTERNS=$(sed -n '/^## Patterns/,/^## [^P]/p' "$child_file" | head -n -1)
        if [ -n "$PATTERNS" ]; then
            AGGREGATED_PATTERNS="${AGGREGATED_PATTERNS}\n\n### From $CHILD_NAME\n$PATTERNS"
        fi
        
        # Extract standards section
        STANDARDS=$(sed -n '/^## Standards/,/^## [^S]/p' "$child_file" | head -n -1)
        if [ -n "$STANDARDS" ]; then
            AGGREGATED_STANDARDS="${AGGREGATED_STANDARDS}\n\n### From $CHILD_NAME\n$STANDARDS"
        fi
        
        # Extract flows section
        FLOWS=$(sed -n '/^## Flows/,/^## [^F]/p' "$child_file" | head -n -1)
        if [ -n "$FLOWS" ]; then
            AGGREGATED_FLOWS="${AGGREGATED_FLOWS}\n\n### From $CHILD_NAME\n$FLOWS"
        fi
        
        # Extract strategies section
        STRATEGIES=$(sed -n '/^## Strategies/,/^## [^S]/p' "$child_file" | head -n -1)
        if [ -n "$STRATEGIES" ]; then
            AGGREGATED_STRATEGIES="${AGGREGATED_STRATEGIES}\n\n### From $CHILD_NAME\n$STRATEGIES"
        fi
        
        # Extract testing section
        TESTING=$(sed -n '/^## Testing/,/^## [^T]/p' "$child_file" | head -n -1)
        if [ -n "$TESTING" ]; then
            AGGREGATED_TESTING="${AGGREGATED_TESTING}\n\n### From $CHILD_NAME\n$TESTING"
        fi
        
        # Extract Libraries Used section (NEW)
        LIBRARIES=$(sed -n '/^## Libraries Used/,/^## [^L]/p' "$child_file" | head -n -1)
        if [ -n "$LIBRARIES" ]; then
            # Parse each library entry and store for aggregation
            # Format: library_name|child_name|imports|usage
            echo "$LIBRARIES" | grep -E "^### " | sed 's/^### //' | while read lib_name; do
                if [ -n "$lib_name" ]; then
                    echo "$lib_name|$CHILD_NAME" >> "$TEMP_LIBRARIES_FILE"
                fi
            done
        fi
    done
    
    # Export for use in basepoint generation
    export AGGREGATED_PATTERNS AGGREGATED_STANDARDS AGGREGATED_FLOWS AGGREGATED_STRATEGIES AGGREGATED_TESTING CHILD_LIST TEMP_LIBRARIES_FILE
}
```

### Step 3.5: Generate Aggregated Libraries Section

After aggregating from children, generate the `## Libraries Used (Aggregated)` section:

```bash
generate_aggregated_libraries_section() {
    local libraries_file="$1"
    local parent_name="$2"
    
    if [ ! -f "$libraries_file" ] || [ ! -s "$libraries_file" ]; then
        echo "## Libraries Used (Aggregated)

*No library usage detected in child modules.*
"
        return
    fi
    
    echo "## Libraries Used (Aggregated)"
    echo ""
    
    # Group by library name and aggregate children
    cut -d'|' -f1 "$libraries_file" | sort -u | while read library; do
        if [ -n "$library" ]; then
            # Get all children that use this library
            CHILDREN_USING=$(grep "^$library|" "$libraries_file" | cut -d'|' -f2 | sort -u | tr '\n' ', ' | sed 's/, $//')
            CHILD_COUNT=$(grep "^$library|" "$libraries_file" | cut -d'|' -f2 | sort -u | wc -l | tr -d ' ')
            
            echo "### $library"
            echo "- **Used in**: $CHILDREN_USING"
            echo "- **Child modules using**: $CHILD_COUNT"
            echo "- **Aggregated Imports**: [Union of imports from all children]"
            echo "- **Usage Summary**: [Summarize how children use this library]"
            echo ""
        fi
    done
    
    # Cleanup temp file
    rm -f "$libraries_file"
}

# Call after aggregate_from_children
AGGREGATED_LIBRARIES_SECTION=$(generate_aggregated_libraries_section "$TEMP_LIBRARIES_FILE" "$PARENT_NAME")
```

### Step 4: Generate Root-Level Basepoint

After all parent basepoints are created, generate the root-level project basepoint:

```bash
# Get project root name (from current directory or config)
PROJECT_ROOT_NAME=$(basename "$(pwd)" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9-]/-/g')
ROOT_BASEPOINT="geist/basepoints/agent-base-$PROJECT_ROOT_NAME.md"

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "🏠 Generating root-level basepoint: $ROOT_BASEPOINT"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Find all top-level child basepoints (direct children of basepoints folder)
TOP_LEVEL_CHILDREN=$(find geist/basepoints -maxdepth 2 -mindepth 2 -name "agent-base-*.md" -type f 2>/dev/null | grep -v "headquarter.md")
TOP_LEVEL_COUNT=$(echo "$TOP_LEVEL_CHILDREN" | grep -v "^$" | wc -l | tr -d ' ')

echo "   → Aggregating from $TOP_LEVEL_COUNT top-level modules"

# Aggregate from all top-level modules
aggregate_from_children "" "$PROJECT_ROOT_NAME" "$ROOT_BASEPOINT"

# Generate root basepoint with complete aggregated knowledge
# This is the final step before headquarter.md

echo "   ✅ Root-level basepoint created: $ROOT_BASEPOINT"
```

### Step 5: Complete to Headquarter

Ensure the headquarter.md is updated to reference the root basepoint:

```bash
HEADQUARTER_FILE="geist/basepoints/headquarter.md"

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "🏛️ Completing hierarchy to headquarter.md"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

if [ -f "$HEADQUARTER_FILE" ]; then
    echo "   → Headquarter exists, verifying linkage to root basepoint..."
    
    # Check if headquarter references the root basepoint
    if grep -q "agent-base-$PROJECT_ROOT_NAME" "$HEADQUARTER_FILE" 2>/dev/null; then
        echo "   ✅ Headquarter already linked to root basepoint"
    else
        echo "   → Updating headquarter to reference root basepoint..."
        # Add reference to root basepoint in headquarter
    fi
else
    echo "   ⚠️ Headquarter not found - will be generated by generate-headquarter workflow"
fi

echo ""
echo "📊 Bottom-up basepoint generation complete!"
echo "   - All parent basepoints aggregate from their children"
echo "   - Knowledge flows correctly up the hierarchy"
echo "   - Root basepoint contains complete project knowledge"
```

### Step 6: Verify Complete Coverage

Verify all basepoints were generated and hierarchy is complete:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "🔍 VERIFICATION: Checking complete hierarchy"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Verify all parent folders have basepoints
MISSING_PARENTS=0
cat geist/output/create-basepoints/cache/parent-folders.txt | while read parent_dir; do
    if [ -z "$parent_dir" ]; then
        continue
    fi
    
    if [ "$parent_dir" = "." ]; then
        PARENT_NAME="$PROJECT_ROOT_NAME"
        BASEPOINT_FILE="geist/basepoints/agent-base-$PARENT_NAME.md"
    else
        PARENT_NAME=$(basename "$parent_dir")
        BASEPOINT_FILE="geist/basepoints/$parent_dir/agent-base-$PARENT_NAME.md"
    fi
    
    if [ ! -f "$BASEPOINT_FILE" ]; then
        echo "   ⚠️ Warning: Missing basepoint for parent: $parent_dir"
        MISSING_PARENTS=$((MISSING_PARENTS + 1))
    fi
done

# Check root basepoint
if [ ! -f "$ROOT_BASEPOINT" ]; then
    echo "   ⚠️ Warning: Root-level basepoint not found"
    MISSING_PARENTS=$((MISSING_PARENTS + 1))
fi

# Summary
TOTAL_MODULE_BPS=$(find geist/basepoints -name "agent-base-*.md" -type f | grep -v "headquarter.md" | wc -l | tr -d ' ')
TOTAL_PARENTS=$(cat geist/output/create-basepoints/cache/parent-folders.txt | wc -l | tr -d ' ')

echo ""
echo "✅ Basepoint hierarchy complete"
echo "   - Module basepoints: $TOTAL_MODULE_BPS"
echo "   - Parent folders processed: $TOTAL_PARENTS"
if [ "$MISSING_PARENTS" -gt 0 ]; then
    echo "   - Missing: $MISSING_PARENTS"
fi
```

## Parent Basepoint Template

Template for parent basepoint with aggregated knowledge:

```markdown
# Parent Basepoint: [Parent Name]

## Overview
- **Location**: `/path/to/parent/`
- **Type**: Parent folder (aggregates child modules)
- **Depth**: [Depth in hierarchy]
- **Children**: [Number of direct child modules]

## Child Modules

| Module | Basepoint |
|--------|-----------|
[CHILD_LIST]

## Architecture at This Level
[High-level architecture of how children work together]

## Aggregated Patterns

### Design Patterns Across Children
[Common design patterns identified across all children]

### Coding Patterns Across Children
[Common coding patterns identified across all children]

### Child-Specific Patterns
[AGGREGATED_PATTERNS]

## Aggregated Standards
[Common standards across children]

[AGGREGATED_STANDARDS]

## Aggregated Flows

### Cross-Module Data Flows
[Data flows that span multiple child modules]

### Cross-Module Control Flows
[Control flows that span multiple child modules]

### Child-Specific Flows
[AGGREGATED_FLOWS]

## Libraries Used (Aggregated)

[Aggregated library usage from all child modules]

### [Library Name]
- **Used in**: [List of child modules using this library]
- **Child modules using**: [Count]
- **Aggregated Imports**: [Union of all imports from children]
- **Usage Summary**: [How this library is used across children]

[Repeat for each library used by children]

[AGGREGATED_LIBRARIES_SECTION]

## Aggregated Strategies
[Common strategies across children]

[AGGREGATED_STRATEGIES]

## Testing Overview
[Testing approaches across all children]

[AGGREGATED_TESTING]

## Notes
- Generated using bottom-up aggregation
- Aggregated from [N] child basepoints
- Knowledge flows from specific children to this parent
- Library usage aggregated from all descendant modules
```

## Important Constraints

- **MUST generate in bottom-up order** - deepest parents first
- **MUST aggregate from child basepoints** - no orphan knowledge
- **MUST complete to root level** - hierarchy must be complete
- Must generate basepoints for all parent folders containing modules
- Must continue up to the top layer of the software project
- Must use child basepoint files as source for creating parent basepoints
- Must aggregate patterns, standards, flows, strategies, testing from children
- Must maintain hierarchical structure with parent basepoints referencing children
- Must preserve relationships between parent and child modules
- Headquarter should reference the root basepoint for complete project knowledge
