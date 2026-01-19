# Project Structure Mirroring

## Core Responsibilities

1. **Create Basepoints Folder**: Ensure `geist/basepoints/` folder exists
2. **Mirror Directory Structure**: Replicate project directory structure within basepoints folder
3. **Exclude Irrelevant Folders**: Skip generated and irrelevant folders during mirroring
4. **Identify Module Folders**: Determine which folders contain actual modules (not config/build)
5. **Include Project Root**: Include the software project's current path layer in the structure

## Workflow

### Step 1: Create Basepoints Folder

Ensure the basepoints folder exists:

```bash
mkdir -p geist/basepoints
```

### Step 2: Define Exclusion Patterns

Set up exclusion patterns for folders that should not be mirrored:

```bash
EXCLUDED_DIRS=(
    "node_modules" ".git" "build" "dist" ".next" "vendor"
    "geist" "basepoints" ".cache" "coverage" ".vscode" ".idea"
)
```

### Step 3: Cleanup Existing Duplicates

Before creating new structure, clean up any existing duplicate folders:

```bash
# Remove duplicate basepoints. folder if it exists
if [ -d "geist/basepoints." ]; then
    echo "⚠️  Removing duplicate basepoints. folder..."
    rm -rf "geist/basepoints."
fi

# Remove any incorrect workflows/basepoints/ folders
if [ -d "geist/workflows/basepoints" ]; then
    echo "⚠️  Removing incorrect workflows/basepoints/ folder..."
    rm -rf "geist/workflows/basepoints"
fi
```

### Step 4: Mirror Project Structure

Create mirrored structure in basepoints folder with proper path normalization:

```bash
find . -type d ! -path "*/\.*" | while read dir; do
    # Normalize directory path: remove leading ./ and handle root (.)
    NORMALIZED_DIR=$(echo "$dir" | sed 's|^\./||' | sed 's|^\.$||')
    
    # Check if should be excluded
    SHOULD_EXCLUDE=false
    for excluded in "${EXCLUDED_DIRS[@]}"; do
        if [[ "$dir" == *"$excluded"* ]] || [[ "$(basename "$dir")" == "$excluded" ]]; then
            SHOULD_EXCLUDE=true
            break
        fi
    done
    
    if [ "$SHOULD_EXCLUDE" = false ]; then
        # Handle root-level path correctly
        if [ -z "$NORMALIZED_DIR" ] || [ "$NORMALIZED_DIR" = "." ]; then
            # Root level - basepoints folder itself (already created in Step 1)
            # Don't create a subdirectory for root
            continue
        else
            # Normal path - create mirrored structure
            basepoints_dir="geist/basepoints/$NORMALIZED_DIR"
            
            # Validate no duplicate (shouldn't happen with normalized paths, but check anyway)
            if [ -d "$basepoints_dir" ] && [ "$basepoints_dir" != "geist/basepoints" ]; then
                echo "⚠️  Warning: Directory already exists: $basepoints_dir"
            fi
            
            mkdir -p "$basepoints_dir"
        fi
    fi
done
```

### Step 5: Identify Module Folders (Deterministic Traversal)

Identify ALL folders that contain actual content files using a deterministic traversal.

**IMPORTANT**: This traversal MUST be comprehensive and include ALL content-containing folders. Do not miss any modules.

```bash
mkdir -p geist/output/create-basepoints/cache

# Clear previous module folders list
> geist/output/create-basepoints/cache/module-folders.txt

# Define content file patterns (comprehensive list)
CONTENT_PATTERNS=(
    "*.ts" "*.tsx" "*.js" "*.jsx"      # JavaScript/TypeScript
    "*.py" "*.pyx"                      # Python
    "*.java" "*.kt" "*.scala"           # JVM languages
    "*.go"                              # Go
    "*.rs"                              # Rust
    "*.rb"                              # Ruby
    "*.php"                             # PHP
    "*.cs" "*.fs"                       # .NET
    "*.swift" "*.m"                     # Apple
    "*.c" "*.cpp" "*.h" "*.hpp"         # C/C++
    "*.sh" "*.bash"                     # Shell scripts
    "*.md"                              # Markdown (documentation/commands)
    "*.yml" "*.yaml"                    # YAML configs
    "*.json"                            # JSON configs
    "*.sql"                             # SQL
    "*.html" "*.css" "*.scss"           # Web files
)

# Build find pattern for content files
FIND_PATTERN=""
for pattern in "${CONTENT_PATTERNS[@]}"; do
    if [ -z "$FIND_PATTERN" ]; then
        FIND_PATTERN="-name \"$pattern\""
    else
        FIND_PATTERN="$FIND_PATTERN -o -name \"$pattern\""
    fi
done

# Deterministic traversal: find ALL directories, sorted for consistency
find . -type d ! -path "*/\.*" 2>/dev/null | sort | while read dir; do
    # Normalize directory path
    NORMALIZED_DIR=$(echo "$dir" | sed 's|^\./||' | sed 's|^\.$||')
    
    # Skip excluded directories
    SHOULD_EXCLUDE=false
    for excluded in "${EXCLUDED_DIRS[@]}"; do
        if [[ "$dir" == *"$excluded"* ]] || [[ "$(basename "$dir")" == "$excluded" ]]; then
            SHOULD_EXCLUDE=true
            break
        fi
    done
    
    if [ "$SHOULD_EXCLUDE" = false ]; then
        # Check if directory contains ANY content files (using expanded patterns)
        HAS_CONTENT=false
        for pattern in "${CONTENT_PATTERNS[@]}"; do
            if find "$dir" -maxdepth 1 -type f -name "$pattern" 2>/dev/null | grep -q .; then
                HAS_CONTENT=true
                break
            fi
        done
        
        if [ "$HAS_CONTENT" = true ]; then
            if [ -z "$NORMALIZED_DIR" ] || [ "$NORMALIZED_DIR" = "." ]; then
                echo "." >> geist/output/create-basepoints/cache/module-folders.txt
            else
                echo "$NORMALIZED_DIR" >> geist/output/create-basepoints/cache/module-folders.txt
            fi
        fi
    fi
done

# Sort and deduplicate for determinism
sort -u geist/output/create-basepoints/cache/module-folders.txt -o geist/output/create-basepoints/cache/module-folders.txt
```

### Step 6: Create Basepoints Task List

Create a comprehensive task list of all basepoints that will be generated:

```bash
# Create task list from module folders
echo "# Basepoints Generation Task List" > geist/output/create-basepoints/cache/basepoints-task-list.md
echo "" >> geist/output/create-basepoints/cache/basepoints-task-list.md
echo "Generated: $(date)" >> geist/output/create-basepoints/cache/basepoints-task-list.md
echo "" >> geist/output/create-basepoints/cache/basepoints-task-list.md
echo "## Modules to Process" >> geist/output/create-basepoints/cache/basepoints-task-list.md
echo "" >> geist/output/create-basepoints/cache/basepoints-task-list.md

TASK_COUNT=0
while read module_dir; do
    if [ -n "$module_dir" ]; then
        TASK_COUNT=$((TASK_COUNT + 1))
        
        # Determine basepoint path
        NORMALIZED_DIR=$(echo "$module_dir" | sed 's|^\./||' | sed 's|^\.$||')
        if [ -z "$NORMALIZED_DIR" ] || [ "$NORMALIZED_DIR" = "." ]; then
            BASEPOINT_PATH="geist/basepoints/agent-base-[project-root].md"
            MODULE_NAME="[project-root]"
        else
            MODULE_NAME=$(basename "$NORMALIZED_DIR")
            BASEPOINT_PATH="geist/basepoints/$NORMALIZED_DIR/agent-base-$MODULE_NAME.md"
        fi
        
        echo "- [ ] **Task $TASK_COUNT**: \`$module_dir\` → \`$BASEPOINT_PATH\`" >> geist/output/create-basepoints/cache/basepoints-task-list.md
    fi
done < geist/output/create-basepoints/cache/module-folders.txt

echo "" >> geist/output/create-basepoints/cache/basepoints-task-list.md
echo "## Summary" >> geist/output/create-basepoints/cache/basepoints-task-list.md
echo "" >> geist/output/create-basepoints/cache/basepoints-task-list.md
echo "**Total modules to process**: $TASK_COUNT" >> geist/output/create-basepoints/cache/basepoints-task-list.md
echo "" >> geist/output/create-basepoints/cache/basepoints-task-list.md
echo "## Parent Basepoints (will be generated after modules)" >> geist/output/create-basepoints/cache/basepoints-task-list.md
echo "" >> geist/output/create-basepoints/cache/basepoints-task-list.md
echo "Parent basepoints will be auto-generated for all intermediate directories." >> geist/output/create-basepoints/cache/basepoints-task-list.md

echo "✅ Task list created: geist/output/create-basepoints/cache/basepoints-task-list.md"
echo "📋 Total modules to process: $TASK_COUNT"
```

### Step 7: Present Task List for Approval

Present the task list to the user for review before proceeding:

```
📋 BASEPOINTS TASK LIST

The following modules will have basepoints generated:

[Show content of basepoints-task-list.md]

Total: [N] module basepoints + parent basepoints

Do you want to proceed with generating these basepoints? (y/n)
```

**IMPORTANT**: Wait for user approval before proceeding to basepoint generation.

## Important Constraints

- Must use deterministic traversal (sorted paths)
- Must include ALL content file types (not just code files)
- Must exclude standard generated/irrelevant folders
- Must create comprehensive task list before generation
- Must present task list for user approval
- Must maintain relative path structure in basepoints folder
- Must identify which folders are module folders for later analysis
- **MUST NOT miss any modules - complete coverage is required**
