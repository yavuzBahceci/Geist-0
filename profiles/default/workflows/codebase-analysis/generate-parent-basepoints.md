# Parent Basepoint Generation

## Core Responsibilities

1. **Identify Parent Folders**: Find parent folders that contain modules
2. **Load Child Basepoints**: Retrieve child module basepoint files as source
3. **Generate Parent Basepoints**: Create basepoint files for parent folders
4. **Aggregate Knowledge**: Combine patterns and knowledge from child modules
5. **Maintain Hierarchy**: Ensure parent basepoints reference their children

## Workflow

### Step 1: Load Module Folders

Load the list of module folders from cache:

```bash
if [ -f "geist/output/create-basepoints/cache/module-folders.txt" ]; then
    MODULE_FOLDERS=$(cat geist/output/create-basepoints/cache/module-folders.txt | grep -v "^#" | grep -v "^$")
else
    echo "❌ Module folders list not found. Run previous phases first."
    exit 1
fi
```

### Step 2: Create Basepoints for All Parent Folders in Path

For each module folder, create basepoints for ALL parent directories up to root:

```bash
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

# Remove duplicates and sort (deepest first, then root)
sort -u "$TEMP_PARENT_DIRS" | sort -r > geist/output/create-basepoints/cache/parent-folders.txt

# Clean up temp file
rm -f "$TEMP_PARENT_DIRS"
```

### Step 3: Generate Basepoints for All Parent Folders

Generate basepoints for all identified parent directories, starting from deepest and moving up:

```bash
cat geist/output/create-basepoints/cache/parent-folders.txt | while read parent_dir; do
    if [ -z "$parent_dir" ]; then
        continue
    fi
    
    # Determine basepoint directory and name
    if [ -z "$parent_dir" ] || [ "$parent_dir" = "." ]; then
        BASEPOINT_DIR="geist/basepoints"
        PARENT_NAME="root"
    else
        BASEPOINT_DIR="geist/basepoints/$parent_dir"
        PARENT_NAME=$(basename "$parent_dir")
    fi
    
    PARENT_BASEPOINT="$BASEPOINT_DIR/agent-base-$PARENT_NAME.md"
    
    # Skip if basepoint already exists
    if [ -f "$PARENT_BASEPOINT" ]; then
        echo "ℹ️  Basepoint already exists: $PARENT_BASEPOINT"
        continue
    fi
    
    # Ensure directory exists
    mkdir -p "$BASEPOINT_DIR"
    
    # Generate parent basepoint by aggregating from children
    # (Implementation details in Step 5)
done
```

### Step 4: Generate Root-Level Basepoint

After all parent basepoints are created, generate root-level basepoint that aggregates from all top-level modules:

```bash
# Get project root name (from current directory or config)
PROJECT_ROOT_NAME=$(basename "$(pwd)" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9-]/-/g')
ROOT_BASEPOINT="geist/basepoints/agent-base-$PROJECT_ROOT_NAME.md"

# Skip if root basepoint already exists
if [ -f "$ROOT_BASEPOINT" ]; then
    echo "ℹ️  Root basepoint already exists: $ROOT_BASEPOINT"
else
    # Find all top-level modules (direct children of basepoints folder)
    TOP_LEVEL_MODULES=$(find geist/basepoints -maxdepth 2 -name "agent-base-*.md" -type f | grep -v "/.*/" | sed 's|geist/basepoints/||' | sed 's|/agent-base-.*\.md||' | sort -u)
    
    # Generate root basepoint by aggregating from all top-level modules
    # This follows the same pattern as parent basepoints but aggregates from all top-level children
    # (Implementation details: aggregate patterns, standards, flows, strategies, testing from all top-level modules)
    
    echo "✅ Root-level basepoint created: $ROOT_BASEPOINT"
fi
```

### Step 5: Generate Parent Basepoint Content

For each parent basepoint, create content by aggregating from children:

Create parent basepoint by aggregating from children:
- Modules list with links
- Sub-folders list with links
- Aggregated Patterns from children
- Aggregated Standards from children
- Aggregated Flows from children
- Aggregated Strategies from children
- Testing Overview from children
- Architecture at this level
- Notes

```bash
# Function to aggregate from children
aggregate_from_children() {
    local parent_dir="$1"
    local parent_name="$2"
    local basepoint_file="$3"
    
    # Find all child basepoints (direct children in this directory)
    if [ -z "$parent_dir" ] || [ "$parent_dir" = "." ]; then
        CHILD_BASEPOINTS=$(find geist/basepoints -maxdepth 1 -name "agent-base-*.md" -type f | grep -v "headquarter.md")
    else
        CHILD_BASEPOINTS=$(find "geist/basepoints/$parent_dir" -maxdepth 1 -name "agent-base-*.md" -type f)
    fi
    
    # Aggregate patterns, standards, flows, strategies, testing from each child
    # Extract and combine information from child basepoints:
    # - Read child basepoint files
    # - Extract patterns, standards, flows, strategies from each child
    # - Aggregate similar patterns together
    # - Identify common themes across children
}
```

### Step 6: Verify Complete Coverage

Verify all basepoints were generated and hierarchy is maintained:

```bash
# Check that root-level basepoint exists
if [ ! -f "geist/basepoints/agent-base-$PROJECT_ROOT_NAME.md" ]; then
    echo "⚠️  Warning: Root-level basepoint not found"
fi

# Verify all parent folders have basepoints
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
        echo "⚠️  Warning: Missing basepoint for parent: $parent_dir"
    fi
done

echo "✅ Basepoint coverage verification complete"
```

## Important Constraints

- Must generate basepoints for all parent folders containing modules
- Must continue up to the top layer of the software project
- Must use child basepoint files as source for creating parent basepoints
- Must aggregate patterns and knowledge from child modules
- Must maintain hierarchical structure with parent basepoints referencing children
- Must preserve relationships between parent and child modules
