# Project Structure Mirroring

## Core Responsibilities

1. **Detect Project Complexity**: Assess project size and structure
2. **Prompt for Coverage Strategy**: For complex projects, ask user to select a coverage strategy
3. **Create Basepoints Folder**: Ensure `geist/basepoints/` folder exists
4. **Mirror Directory Structure**: Replicate project directory structure within basepoints folder
5. **Exclude Irrelevant Folders**: Skip generated and irrelevant folders during mirroring
6. **Identify Module Folders**: Determine which folders contain actual modules (not config/build)
7. **Include Project Root**: Include the software project's current path layer in the structure

## Workflow

### Step 1: Detect Project Complexity and Prompt for Strategy

First, assess the project size to determine if strategy selection is needed:

```bash
echo "🔍 Assessing project complexity..."

# Count total directories (excluding common ignores)
TOTAL_DIRS=$(find . -type d ! -path "*/\.*" ! -path "*/node_modules/*" ! -path "*/vendor/*" ! -path "*/build/*" ! -path "*/dist/*" ! -path "*/geist/*" 2>/dev/null | wc -l | tr -d ' ')

# Count source files
TOTAL_FILES=$(find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.java" -o -name "*.kt" -o -name "*.go" -o -name "*.rs" -o -name "*.swift" -o -name "*.cs" \) ! -path "*/node_modules/*" ! -path "*/vendor/*" ! -path "*/build/*" ! -path "*/dist/*" 2>/dev/null | wc -l | tr -d ' ')

echo "   Total directories: $TOTAL_DIRS"
echo "   Total source files: $TOTAL_FILES"

# Determine complexity
# Simple: < 20 directories and < 100 files
# Complex: >= 20 directories or >= 100 files
if [ "$TOTAL_DIRS" -ge 20 ] || [ "$TOTAL_FILES" -ge 100 ]; then
    PROJECT_COMPLEXITY="complex"
    echo "   📊 Project complexity: COMPLEX"
else
    PROJECT_COMPLEXITY="simple"
    echo "   📊 Project complexity: SIMPLE"
fi
```

### Step 1.5: Coverage Strategy Selection (Complex Projects Only)

For complex projects, prompt the user to select a coverage strategy:

```bash
if [ "$PROJECT_COMPLEXITY" = "complex" ]; then
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "📋 COVERAGE STRATEGY SELECTION"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo ""
    echo "Your project is complex ($TOTAL_DIRS directories, $TOTAL_FILES source files)."
    echo "Please select a basepoint coverage strategy:"
    echo ""
    echo "  1. **ALL** - Generate basepoints for ALL folders (default, most thorough)"
    echo "     Best for: Complete documentation, new team members"
    echo ""
    echo "  2. **SELECTIVE** - Select specific folders to include"
    echo "     Best for: Focus on key areas, faster generation"
    echo ""
    echo "  3. **LAYER-BASED** - Select by abstraction layer"
    echo "     Options: presentation, business, data, infrastructure, shared"
    echo "     Best for: Layer-focused development"
    echo ""
    echo "  4. **DOMAIN-BASED** - Select by domain/feature area"
    echo "     Will list detected domains for selection"
    echo "     Best for: Feature-focused development"
    echo ""
    echo "  5. **TOP-N-LEVELS** - Only top N levels of depth"
    echo "     Best for: High-level overview without deep details"
    echo ""
    echo "Enter your choice (1-5) or press Enter for ALL:"
    echo ""
    echo "⚠️ **STOP and WAIT** for user response."
fi
```

### Step 1.6: Process Coverage Strategy Selection

Based on user's choice, set up folder filtering:

```bash
# Default to ALL strategy
COVERAGE_STRATEGY="all"
SELECTED_FOLDERS=""
SELECTED_LAYERS=""
MAX_DEPTH=999

# Process user's selection
process_coverage_selection() {
    local selection="$1"
    
    case "$selection" in
        "1"|"all"|"")
            COVERAGE_STRATEGY="all"
            echo "✅ Coverage strategy: ALL folders"
            ;;
        "2"|"selective")
            COVERAGE_STRATEGY="selective"
            echo ""
            echo "Enter folder paths to include (comma-separated):"
            echo "Example: src/features, src/core, lib"
            echo ""
            # User provides: SELECTED_FOLDERS="src/features,src/core,lib"
            ;;
        "3"|"layer"|"layer-based")
            COVERAGE_STRATEGY="layer"
            echo ""
            echo "Select layers to include (comma-separated):"
            echo "Available: presentation, business, data, infrastructure, shared, all"
            echo ""
            # User provides: SELECTED_LAYERS="business,data"
            ;;
        "4"|"domain"|"domain-based")
            COVERAGE_STRATEGY="domain"
            echo ""
            echo "Detecting domains in your project..."
            
            # Detect potential domains from folder structure
            DETECTED_DOMAINS=$(find . -type d -maxdepth 3 ! -path "*/\.*" ! -path "*/node_modules/*" | \
                grep -E "(features|modules|domains|apps)/[^/]+$" | \
                sed 's|.*/||' | sort -u)
            
            if [ -n "$DETECTED_DOMAINS" ]; then
                echo "Detected domains:"
                echo "$DETECTED_DOMAINS" | nl
                echo ""
                echo "Enter domain numbers to include (comma-separated) or 'all':"
            else
                echo "No standard domain structure detected."
                echo "Enter folder paths containing your domains:"
            fi
            ;;
        "5"|"top"|"top-n"|"levels")
            COVERAGE_STRATEGY="top-n"
            echo ""
            echo "Enter maximum folder depth (1-5):"
            echo "  1 = Top-level only"
            echo "  2 = Top-level + one level down"
            echo "  3 = Recommended for overview"
            echo ""
            # User provides: MAX_DEPTH=3
            ;;
        *)
            echo "Invalid selection. Using ALL strategy."
            COVERAGE_STRATEGY="all"
            ;;
    esac
}

# Store strategy for later use
mkdir -p geist/output/create-basepoints/cache
cat > geist/output/create-basepoints/cache/coverage-strategy.txt << STRATEGY_EOF
COVERAGE_STRATEGY=$COVERAGE_STRATEGY
SELECTED_FOLDERS=$SELECTED_FOLDERS
SELECTED_LAYERS=$SELECTED_LAYERS
MAX_DEPTH=$MAX_DEPTH
PROJECT_COMPLEXITY=$PROJECT_COMPLEXITY
TOTAL_DIRS=$TOTAL_DIRS
TOTAL_FILES=$TOTAL_FILES
STRATEGY_EOF

echo "📁 Coverage strategy saved to: geist/output/create-basepoints/cache/coverage-strategy.txt"
```

### Step 2: Create Basepoints Folder

Ensure the basepoints folder exists:

```bash
mkdir -p geist/basepoints
```

### Step 3: Define Exclusion Patterns

Set up exclusion patterns for folders that should not be mirrored:

```bash
EXCLUDED_DIRS=(
    "node_modules" ".git" "build" "dist" ".next" "vendor"
    "geist" "basepoints" ".cache" "coverage" ".vscode" ".idea"
)
```

### Step 4: Cleanup Existing Duplicates

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

### Step 5: Mirror Project Structure

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

### Step 6: Identify Module Folders (With Strategy Filtering)

Identify folders that contain actual content files, filtered by the selected coverage strategy.

**IMPORTANT**: This traversal MUST respect the coverage strategy while being comprehensive within selected scope.

```bash
mkdir -p geist/output/create-basepoints/cache

# Load coverage strategy
if [ -f "geist/output/create-basepoints/cache/coverage-strategy.txt" ]; then
    source geist/output/create-basepoints/cache/coverage-strategy.txt
else
    COVERAGE_STRATEGY="all"
    MAX_DEPTH=999
fi

echo "📋 Applying coverage strategy: $COVERAGE_STRATEGY"

# Clear previous module folders list
> geist/output/create-basepoints/cache/module-folders.txt

# Strategy filtering function
should_include_folder() {
    local folder="$1"
    local depth=$(echo "$folder" | tr -cd '/' | wc -c)
    
    case "$COVERAGE_STRATEGY" in
        "all")
            return 0  # Include all
            ;;
        "selective")
            # Check if folder matches or is under selected folders
            for selected in $(echo "$SELECTED_FOLDERS" | tr ',' ' '); do
                if [[ "$folder" == "$selected"* ]] || [[ "$selected" == "$folder"* ]]; then
                    return 0
                fi
            done
            return 1  # Not in selected
            ;;
        "layer")
            # Check if folder matches selected layers
            for layer in $(echo "$SELECTED_LAYERS" | tr ',' ' '); do
                case "$layer" in
                    "presentation")
                        if [[ "$folder" == *"ui"* ]] || [[ "$folder" == *"view"* ]] || \
                           [[ "$folder" == *"component"* ]] || [[ "$folder" == *"page"* ]] || \
                           [[ "$folder" == *"screen"* ]]; then
                            return 0
                        fi
                        ;;
                    "business")
                        if [[ "$folder" == *"service"* ]] || [[ "$folder" == *"usecase"* ]] || \
                           [[ "$folder" == *"domain"* ]] || [[ "$folder" == *"business"* ]] || \
                           [[ "$folder" == *"logic"* ]]; then
                            return 0
                        fi
                        ;;
                    "data")
                        if [[ "$folder" == *"data"* ]] || [[ "$folder" == *"repository"* ]] || \
                           [[ "$folder" == *"model"* ]] || [[ "$folder" == *"entity"* ]] || \
                           [[ "$folder" == *"store"* ]]; then
                            return 0
                        fi
                        ;;
                    "infrastructure")
                        if [[ "$folder" == *"infra"* ]] || [[ "$folder" == *"api"* ]] || \
                           [[ "$folder" == *"network"* ]] || [[ "$folder" == *"config"* ]]; then
                            return 0
                        fi
                        ;;
                    "shared"|"common"|"util")
                        if [[ "$folder" == *"shared"* ]] || [[ "$folder" == *"common"* ]] || \
                           [[ "$folder" == *"util"* ]] || [[ "$folder" == *"lib"* ]]; then
                            return 0
                        fi
                        ;;
                    "all")
                        return 0
                        ;;
                esac
            done
            return 1  # Not in selected layers
            ;;
        "domain")
            # Check if folder matches selected domains
            for domain in $(echo "$SELECTED_FOLDERS" | tr ',' ' '); do
                if [[ "$folder" == *"$domain"* ]]; then
                    return 0
                fi
            done
            return 1  # Not in selected domains
            ;;
        "top-n")
            # Check if folder is within max depth
            if [ "$depth" -le "$MAX_DEPTH" ]; then
                return 0
            fi
            return 1  # Too deep
            ;;
        *)
            return 0  # Default: include all
            ;;
    esac
}

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
            # Apply coverage strategy filter
            if should_include_folder "$NORMALIZED_DIR"; then
                if [ -z "$NORMALIZED_DIR" ] || [ "$NORMALIZED_DIR" = "." ]; then
                    echo "." >> geist/output/create-basepoints/cache/module-folders.txt
                else
                    echo "$NORMALIZED_DIR" >> geist/output/create-basepoints/cache/module-folders.txt
                fi
            else
                echo "   ⏭️ Skipped by strategy: $NORMALIZED_DIR"
            fi
        fi
    fi
done

# Report strategy filtering results
TOTAL_INCLUDED=$(cat geist/output/create-basepoints/cache/module-folders.txt | wc -l | tr -d ' ')
echo ""
echo "📊 Coverage strategy: $COVERAGE_STRATEGY"
echo "   Modules included: $TOTAL_INCLUDED"

# Sort and deduplicate for determinism
sort -u geist/output/create-basepoints/cache/module-folders.txt -o geist/output/create-basepoints/cache/module-folders.txt
```

### Step 7: Create Basepoints Task List

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

### Step 8: Present Task List for Approval

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
