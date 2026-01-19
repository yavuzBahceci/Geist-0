# Module Basepoint Generation (Bottom-Up)

## Core Responsibilities

1. **Order Modules by Depth**: Sort modules from deepest (leaf nodes) to shallowest
2. **Generate Basepoints Bottom-Up**: Start with deepest modules, then work upward
3. **Track Progress**: Update task list as each basepoint is created
4. **Verify Coverage**: Ensure ALL modules have basepoints (no misses)
5. **Report Completion**: Provide summary of all generated basepoints

## Bottom-Up Generation Strategy

```
IMPORTANT: Generate basepoints starting from the DEEPEST modules first.

This ensures:
1. Leaf modules (no children) are processed first
2. Parent modules can aggregate from their children's already-generated basepoints
3. Knowledge flows correctly from specific (deep) to general (shallow)
4. No missing context when generating parent basepoints

Example order for:
  src/
  ├── features/
  │   ├── auth/
  │   │   └── hooks/
  │   └── dashboard/
  └── shared/
      └── utils/

Generation Order:
1. src/features/auth/hooks/      (depth 4 - deepest)
2. src/features/dashboard/        (depth 3)
3. src/shared/utils/              (depth 3)
4. src/features/auth/             (depth 3 - aggregates from hooks/)
5. src/features/                  (depth 2 - aggregates from auth/, dashboard/)
6. src/shared/                    (depth 2 - aggregates from utils/)
7. src/                           (depth 1 - aggregates from features/, shared/)
```

## Workflow

### Step 1: Load and Sort Modules by Depth (Deepest First)

Load module folders and sort by depth (deepest first for bottom-up generation):

```bash
# Load module folders
if [ ! -f "geist/output/create-basepoints/cache/module-folders.txt" ]; then
    echo "❌ ERROR: Module folders list not found. Run mirror-project-structure first."
    exit 1
fi

# Load raw modules
RAW_MODULES=$(cat geist/output/create-basepoints/cache/module-folders.txt | grep -v "^#" | grep -v "^$")

# Sort by depth (count slashes) - deepest first
# This implements the BOTTOM-UP approach
echo "$RAW_MODULES" | while read dir; do
    depth=$(echo "$dir" | tr -cd '/' | wc -c)
    echo "$depth|$dir"
done | sort -t'|' -k1 -rn | cut -d'|' -f2 > geist/output/create-basepoints/cache/modules-by-depth.txt

MODULE_FOLDERS=$(cat geist/output/create-basepoints/cache/modules-by-depth.txt)
TOTAL_MODULES=$(echo "$MODULE_FOLDERS" | wc -l | tr -d ' ')

echo "📋 Loaded $TOTAL_MODULES modules (sorted deepest-first for bottom-up generation)"

# Display depth order for verification
echo ""
echo "Generation Order (bottom-up):"
cat geist/output/create-basepoints/cache/modules-by-depth.txt | head -10
if [ "$TOTAL_MODULES" -gt 10 ]; then
    echo "... ($((TOTAL_MODULES - 10)) more modules)"
fi
echo ""

# Initialize progress tracking
cat > geist/output/create-basepoints/cache/generation-progress.md << PROGRESS_EOF
# Basepoints Generation Progress (Bottom-Up)

Started: $(date)

## Strategy: Bottom-Up Generation
- Starting with deepest modules (leaf nodes)
- Working upward to parent modules
- Each parent can aggregate from already-generated child basepoints

## Progress

PROGRESS_EOF
```

### Step 2: Generate Basepoints Bottom-Up (Deepest First)

Process each module from deepest to shallowest:

```bash
CURRENT=0
GENERATED=()
FAILED=()

cat geist/output/create-basepoints/cache/modules-by-depth.txt | while read module_dir; do
    if [ -z "$module_dir" ]; then
        continue
    fi
    
    CURRENT=$((CURRENT + 1))
    
    # Calculate depth for display
    DEPTH=$(echo "$module_dir" | tr -cd '/' | wc -c)
    
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "📦 Processing module $CURRENT/$TOTAL_MODULES: $module_dir (depth: $DEPTH)"
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
    
    # Check for existing child basepoints (for aggregation)
    CHILD_BASEPOINTS=""
    if [ -d "$BASEPOINT_DIR" ]; then
        CHILD_BASEPOINTS=$(find "$BASEPOINT_DIR" -mindepth 2 -name "agent-base-*.md" -type f 2>/dev/null | head -20)
    fi
    
    if [ -n "$CHILD_BASEPOINTS" ]; then
        echo "   → Found child basepoints to aggregate:"
        echo "$CHILD_BASEPOINTS" | while read child; do
            echo "      - $(basename "$(dirname "$child")")/$(basename "$child")"
        done
    fi
    
    # Generate basepoint content
    echo "   → Analyzing module: $module_dir"
    echo "   → Creating basepoint: $BASEPOINT_FILE"
    
    # [Analyze module and generate content here - see Step 3]
    
    # Log progress
    echo "- [x] **$CURRENT/$TOTAL_MODULES** (depth $DEPTH): \`$module_dir\` → \`$BASEPOINT_FILE\`" >> geist/output/create-basepoints/cache/generation-progress.md
    
    echo "   ✅ Created: $BASEPOINT_FILE"
done
```

### Step 2.5: Extract Library Imports from Module Files

Before generating basepoint content, extract library imports from module files:

```bash
extract_library_imports() {
    local module_dir="$1"
    local output_file="$2"
    
    echo "   → Extracting library imports from module files..."
    
    # Initialize imports file
    > "$output_file"
    
    # Define file extensions based on detected project type
    # (PROJECT_TYPE should be set from tech-stack detection)
    case "$PROJECT_TYPE" in
        nodejs|react-native)
            FILE_PATTERN="*.ts *.tsx *.js *.jsx *.mjs *.cjs"
            ;;
        python)
            FILE_PATTERN="*.py"
            ;;
        rust)
            FILE_PATTERN="*.rs"
            ;;
        go)
            FILE_PATTERN="*.go"
            ;;
        ruby)
            FILE_PATTERN="*.rb"
            ;;
        php)
            FILE_PATTERN="*.php"
            ;;
        java-maven|java-gradle|android)
            FILE_PATTERN="*.java *.kt"
            ;;
        ios-cocoapods|ios-spm|ios-xcode)
            FILE_PATTERN="*.swift *.m *.mm"
            ;;
        flutter)
            FILE_PATTERN="*.dart"
            ;;
        dotnet)
            FILE_PATTERN="*.cs"
            ;;
        *)
            FILE_PATTERN="*.ts *.js *.py *.go *.rs *.java *.kt *.swift *.dart *.cs *.rb *.php"
            ;;
    esac
    
    # Find all source files in module
    for pattern in $FILE_PATTERN; do
        find "$module_dir" -maxdepth 1 -name "$pattern" -type f 2>/dev/null | while read source_file; do
            if [ -f "$source_file" ]; then
                # Extract imports based on language
                case "$source_file" in
                    *.ts|*.tsx|*.js|*.jsx|*.mjs|*.cjs)
                        # JavaScript/TypeScript: import X from 'lib' or require('lib')
                        grep -E "^import .* from ['\"]|require\(['\"]" "$source_file" 2>/dev/null | \
                            sed -E "s/.*from ['\"]([^'\"]+)['\"].*/\1/" | \
                            sed -E "s/.*require\(['\"]([^'\"]+)['\"].*/\1/" | \
                            grep -v "^\." | grep -v "^@/" | \
                            while read lib; do
                                # Extract base library name (handle scoped packages)
                                BASE_LIB=$(echo "$lib" | sed -E 's|^(@[^/]+/[^/]+).*|\1|' | sed -E 's|^([^/]+).*|\1|')
                                echo "$BASE_LIB|$(basename "$source_file")" >> "$output_file"
                            done
                        ;;
                    *.py)
                        # Python: import lib or from lib import X
                        grep -E "^import |^from " "$source_file" 2>/dev/null | \
                            sed -E "s/^import ([a-zA-Z0-9_]+).*/\1/" | \
                            sed -E "s/^from ([a-zA-Z0-9_]+).*/\1/" | \
                            grep -v "^\." | \
                            while read lib; do
                                echo "$lib|$(basename "$source_file")" >> "$output_file"
                            done
                        ;;
                    *.go)
                        # Go: import "lib" or import ( "lib" )
                        grep -E '^\s*"[^"]+"|^\s*[a-z]+ "[^"]+"' "$source_file" 2>/dev/null | \
                            grep -oE '"[^"]+"' | tr -d '"' | \
                            while read lib; do
                                BASE_LIB=$(echo "$lib" | rev | cut -d'/' -f1 | rev)
                                echo "$BASE_LIB|$(basename "$source_file")" >> "$output_file"
                            done
                        ;;
                    *.rs)
                        # Rust: use lib::X or extern crate lib
                        grep -E "^use [a-z_]+::|^extern crate " "$source_file" 2>/dev/null | \
                            sed -E "s/^use ([a-z_]+)::.*/\1/" | \
                            sed -E "s/^extern crate ([a-z_]+).*/\1/" | \
                            while read lib; do
                                echo "$lib|$(basename "$source_file")" >> "$output_file"
                            done
                        ;;
                    *.java|*.kt)
                        # Java/Kotlin: import com.lib.X
                        grep -E "^import " "$source_file" 2>/dev/null | \
                            sed -E "s/^import (static )?([a-z]+\.[a-z]+).*/\2/" | \
                            grep -v "^java\." | grep -v "^javax\." | grep -v "^android\." | \
                            while read lib; do
                                echo "$lib|$(basename "$source_file")" >> "$output_file"
                            done
                        ;;
                    *.swift)
                        # Swift: import Lib
                        grep -E "^import " "$source_file" 2>/dev/null | \
                            sed -E "s/^import ([A-Za-z_]+).*/\1/" | \
                            grep -v "^Foundation$" | grep -v "^UIKit$" | grep -v "^SwiftUI$" | \
                            while read lib; do
                                echo "$lib|$(basename "$source_file")" >> "$output_file"
                            done
                        ;;
                    *.dart)
                        # Dart: import 'package:lib/X.dart'
                        grep -E "^import ['\"]package:" "$source_file" 2>/dev/null | \
                            sed -E "s/.*package:([^/]+).*/\1/" | \
                            while read lib; do
                                echo "$lib|$(basename "$source_file")" >> "$output_file"
                            done
                        ;;
                    *.cs)
                        # C#: using Lib.X
                        grep -E "^using " "$source_file" 2>/dev/null | \
                            sed -E "s/^using (static )?([A-Za-z]+).*/\2/" | \
                            grep -v "^System$" | \
                            while read lib; do
                                echo "$lib|$(basename "$source_file")" >> "$output_file"
                            done
                        ;;
                    *.rb)
                        # Ruby: require 'lib' or gem 'lib'
                        grep -E "^require ['\"]|^gem ['\"]" "$source_file" 2>/dev/null | \
                            sed -E "s/.*['\"]([^'\"]+)['\"].*/\1/" | \
                            while read lib; do
                                echo "$lib|$(basename "$source_file")" >> "$output_file"
                            done
                        ;;
                    *.php)
                        # PHP: use Lib\X
                        grep -E "^use " "$source_file" 2>/dev/null | \
                            sed -E "s/^use ([A-Za-z]+).*/\1/" | \
                            while read lib; do
                                echo "$lib|$(basename "$source_file")" >> "$output_file"
                            done
                        ;;
                esac
            fi
        done
    done
    
    # Deduplicate and count
    if [ -f "$output_file" ] && [ -s "$output_file" ]; then
        sort "$output_file" | uniq > "${output_file}.tmp"
        mv "${output_file}.tmp" "$output_file"
        IMPORT_COUNT=$(wc -l < "$output_file" | tr -d ' ')
        echo "   → Found $IMPORT_COUNT library imports"
    else
        echo "   → No library imports found"
    fi
}

# Call during module processing (after BASEPOINT_DIR is set)
IMPORTS_FILE="/tmp/module-imports-$(echo "$MODULE_NAME" | tr '/' '-').txt"
extract_library_imports "$module_dir" "$IMPORTS_FILE"
```

### Step 3: Generate Basepoint Content (With Child Aggregation)

For each module, create comprehensive basepoint content that includes aggregated child knowledge:

```markdown
# Basepoint: [Module Name]

## Module Overview
- **Location**: `/path/to/module/`
- **Purpose**: [Extracted from analysis]
- **Layer**: [Abstraction layer]
- **Depth**: [Depth in hierarchy]

## Child Modules
[List of child modules with links to their basepoints - enables drill-down]

| Child Module | Basepoint | Summary |
|--------------|-----------|---------|
| [child1/] | [link] | [Brief summary from child basepoint] |
| [child2/] | [link] | [Brief summary from child basepoint] |

## Files in Module
| File | Purpose |
|------|---------|
| [files...] | [purposes...] |

## Patterns

### Patterns from This Module
[Directly extracted patterns from this module's code]

### Aggregated Patterns from Children
[Patterns identified across child modules - common themes]

## Standards
[Extracted standards]

## Flows

### Data Flow
[Extracted flows]

### Control Flow
[Extracted flows]

### Cross-Module Flows
[Flows that span multiple child modules]

## Libraries Used

[For each library imported in this module, document usage]

### [Library Name]
- **Imports**: [List of specific imports: functions, classes, modules]
- **Usage**: [How this module uses the library]
- **Module Pattern**: [Module-specific usage pattern]

[Repeat for each library used in this module]

## Strategies
[Extracted strategies]

## Testing

### Testing in This Module
[Testing approaches for this module]

### Testing Patterns from Children
[Common testing patterns across child modules]

## Related Modules
[Dependencies and relationships]

## Notes
- Generated using bottom-up approach
- Aggregates knowledge from [N] child basepoints
```

### Step 3.5: Generate Libraries Used Section

Generate the `## Libraries Used` section from extracted imports:

```bash
generate_libraries_used_section() {
    local imports_file="$1"
    local output_var="$2"
    
    if [ ! -f "$imports_file" ] || [ ! -s "$imports_file" ]; then
        echo "## Libraries Used

*No external library imports detected in this module.*
"
        return
    fi
    
    echo "## Libraries Used"
    echo ""
    
    # Group imports by library
    cut -d'|' -f1 "$imports_file" | sort -u | while read library; do
        if [ -n "$library" ]; then
            # Get files that use this library
            FILES_USING=$(grep "^$library|" "$imports_file" | cut -d'|' -f2 | sort -u | tr '\n' ', ' | sed 's/, $//')
            
            echo "### $library"
            echo "- **Imports**: \`$library\` (and related exports)"
            echo "- **Used in**: $FILES_USING"
            echo "- **Usage**: [Analyze actual usage patterns in code]"
            echo "- **Module Pattern**: [Identify module-specific patterns]"
            echo ""
        fi
    done
}

# Call when generating basepoint content
LIBRARIES_SECTION=$(generate_libraries_used_section "$IMPORTS_FILE")
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

# List all generated basepoints by depth
echo "" >> geist/output/create-basepoints/cache/generation-progress.md
echo "## Generated Basepoints (by depth)" >> geist/output/create-basepoints/cache/generation-progress.md
echo "" >> geist/output/create-basepoints/cache/generation-progress.md

# Group by depth
for depth in $(seq 10 -1 0); do
    BASEPOINTS_AT_DEPTH=$(find geist/basepoints -name "agent-base-*.md" -type f | while read bp; do
        bp_depth=$(echo "$bp" | tr -cd '/' | wc -c)
        if [ "$bp_depth" -eq "$((depth + 2))" ]; then  # +2 for geist/basepoints/
            echo "$bp"
        fi
    done)
    
    if [ -n "$BASEPOINTS_AT_DEPTH" ]; then
        echo "### Depth $depth" >> geist/output/create-basepoints/cache/generation-progress.md
        echo "$BASEPOINTS_AT_DEPTH" | while read bp; do
            echo "- \`$bp\`" >> geist/output/create-basepoints/cache/generation-progress.md
        done
        echo "" >> geist/output/create-basepoints/cache/generation-progress.md
    fi
done

# Check for missing basepoints
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
📊 MODULE BASEPOINTS GENERATION COMPLETE (Bottom-Up)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Total modules processed: [N]
✅ Basepoints created: [N]
❌ Failed: [N] (if any)

📈 Generation Strategy: Bottom-Up (deepest modules first)
📁 Basepoints location: geist/basepoints/
📋 Progress report: geist/output/create-basepoints/cache/generation-progress.md

Next: Parent basepoints will complete the hierarchy up to headquarter.
```

## Important Constraints

- **MUST generate in bottom-up order** - deepest modules first
- **MUST process ALL modules from task list** - no exceptions
- **MUST track progress** for each module processed
- **MUST aggregate from child basepoints** when they exist
- **MUST verify coverage** after completion
- **MUST report any missing basepoints** as errors
- Must generate one basepoint file per module folder
- Must place files in mirrored structure within basepoints folder
- Must name files based on actual module names from folder structure
- Must include all extracted patterns, standards, flows, strategies, and testing
- Must reference the module's abstraction layer and depth
- Must save basepoint files for use in parent basepoint generation
