Now that validation is complete, proceed with cleaning remaining placeholders by following these instructions:

## Core Responsibilities

1. **Load Placeholder Detection Results**: Load placeholder validation results
2. **Replace Placeholders**: Replace placeholders with project-specific content using deploy-agents logic
3. **Support Dry-Run Mode**: Preview changes before applying
4. **Track Changes**: Track what was fixed for cleanup report

## Workflow

### Step 1: Load Placeholder Detection Results

Load placeholder validation results:

```bash
# Load validation cache
VALIDATION_CACHE="geist/.cleanup-cache/validation"
CLEANUP_CACHE="geist/.cleanup-cache"
DRY_RUN="${DRY_RUN:-false}"

# Load placeholder detection results
if [ -f "$VALIDATION_CACHE/placeholder-cleaning-validation.json" ]; then
    PLACEHOLDER_DATA=$(cat "$VALIDATION_CACHE/placeholder-cleaning-validation.json")
    PLACEHOLDER_TOTAL=$(echo "$PLACEHOLDER_DATA" | grep -o '"total_placeholders_found":[0-9]*' | cut -d: -f2)
    
    if [ "$PLACEHOLDER_TOTAL" -eq 0 ]; then
        echo "✅ No placeholders to clean"
        echo "0" > "$CLEANUP_CACHE/placeholders-cleaned.txt"
        exit 0
    fi
    
    echo "🔧 Found $PLACEHOLDER_TOTAL placeholder(s) to clean"
else
    echo "❌ Placeholder validation results not found"
    exit 1
fi
```

### Step 2: Load Project-Specific Knowledge

Load project-specific knowledge needed for placeholder replacement:

```bash
# Load basepoints knowledge if available
if [ "$BASEPOINTS_AVAILABLE" = "true" ] && [ -f "geist/basepoints/headquarter.md" ]; then
    # Extract basepoints path pattern
    BASEPOINT_FILE_PATTERN=$(find geist/basepoints -name "agent-base-*.md" -type f | head -1 | xargs basename | sed 's/agent-base-//' | sed 's/\.md//')
    if [ -z "$BASEPOINT_FILE_PATTERN" ]; then
        BASEPOINT_FILE_PATTERN="agent-base-*.md"
    fi
    
    # Extract abstraction layers
    ABSTRACTION_LAYERS=$(find geist/basepoints -type d -mindepth 1 -maxdepth 2 | sed 's|geist/basepoints/||' | cut -d'/' -f1 | sort -u | tr '\n' ',' | sed 's/,$//')
    
    echo "✅ Loaded basepoints knowledge"
else
    echo "⚠️  Warning: Basepoints not available - using generic replacements"
    BASEPOINT_FILE_PATTERN="agent-base-*.md"
    ABSTRACTION_LAYERS=""
fi

# Load product knowledge if available
if [ "$PRODUCT_AVAILABLE" = "true" ] && [ -f "geist/product/tech-stack.md" ]; then
    TECH_STACK=$(cat geist/product/tech-stack.md)
    echo "✅ Loaded product knowledge"
else
    TECH_STACK=""
fi
```

### Step 3: Replace Placeholders

Replace placeholders with project-specific content:

```bash
# Initialize cleanup tracking
PLACEHOLDERS_CLEANED=0
FILES_MODIFIED=0

# Extract placeholder locations from validation results
echo "$PLACEHOLDER_DATA" | grep -oE '"file": "[^"]+"' | cut -d'"' -f4 | sort -u > "$CLEANUP_CACHE/files-with-placeholders.txt"

# Process each file with placeholders
while read file_path; do
    if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
        continue
    fi
    
    FILE_CONTENT=$(cat "$file_path")
    ORIGINAL_CONTENT="$FILE_CONTENT"
    MODIFIED=false
    
    # Replace common basepoints placeholders
    if echo "$FILE_CONTENT" | grep -q "{{BASEPOINTS_PATH}}"; then
        FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "s|{{BASEPOINTS_PATH}}|geist/basepoints|g")
        MODIFIED=true
        PLACEHOLDERS_CLEANED=$((PLACEHOLDERS_CLEANED + 1))
    fi
    
    if echo "$FILE_CONTENT" | grep -q "{{BASEPOINT_FILE_PATTERN}}"; then
        FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "s|{{BASEPOINT_FILE_PATTERN}}|$BASEPOINT_FILE_PATTERN|g")
        MODIFIED=true
        PLACEHOLDERS_CLEANED=$((PLACEHOLDERS_CLEANED + 1))
    fi
    
    if echo "$FILE_CONTENT" | grep -q "{{ABSTRACTION_LAYER_NAMES}}"; then
        FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "s|{{ABSTRACTION_LAYER_NAMES}}|$ABSTRACTION_LAYERS|g")
        MODIFIED=true
        PLACEHOLDERS_CLEANED=$((PLACEHOLDERS_CLEANED + 1))
    fi
    
    # Replace scope detection placeholders with project-specific patterns
    # (These would need more sophisticated replacement based on actual project structure)
    if echo "$FILE_CONTENT" | grep -qE "{{KEYWORD_EXTRACTION_PATTERN}}|{{SEMANTIC_ANALYSIS_PATTERN}}"; then
        # Use project-specific patterns from basepoints if available
        FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "s|{{KEYWORD_EXTRACTION_PATTERN}}|project-specific-keyword-pattern|g")
        FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "s|{{SEMANTIC_ANALYSIS_PATTERN}}|project-specific-semantic-pattern|g")
        MODIFIED=true
        PLACEHOLDERS_CLEANED=$((PLACEHOLDERS_CLEANED + 2))
    fi
    
    # Apply changes if modified
    if [ "$MODIFIED" = "true" ]; then
        if [ "$DRY_RUN" = "true" ]; then
            echo "🔍 [DRY-RUN] Would modify: $file_path"
            echo "   Changes preview available in: $CLEANUP_CACHE/dry-run-preview/$file_path"
            mkdir -p "$(dirname "$CLEANUP_CACHE/dry-run-preview/$file_path")"
            echo "$FILE_CONTENT" > "$CLEANUP_CACHE/dry-run-preview/$file_path"
        else
            echo "$FILE_CONTENT" > "$file_path"
            echo "✅ Modified: $file_path"
            FILES_MODIFIED=$((FILES_MODIFIED + 1))
        fi
    fi
done < "$CLEANUP_CACHE/files-with-placeholders.txt"

# Store cleanup results
echo "$PLACEHOLDERS_CLEANED" > "$CLEANUP_CACHE/placeholders-cleaned.txt"
echo "$FILES_MODIFIED" > "$CLEANUP_CACHE/files-modified-placeholders.txt"

if [ "$DRY_RUN" = "true" ]; then
    echo "🔍 [DRY-RUN] Would clean $PLACEHOLDERS_CLEANED placeholder(s) in files"
else
    echo "✅ Cleaned $PLACEHOLDERS_CLEANED placeholder(s) in $FILES_MODIFIED file(s)"
fi
```

## Important Constraints

- Must use placeholder detection results from validation phase
- Must replace placeholders with project-specific content using deploy-agents logic
- Must support dry-run mode to preview changes
- Must track what was fixed for cleanup report
- **CRITICAL**: All cleanup cache files must be stored in `geist/.cleanup-cache/` (temporary, cleaned up after cleanup completes)
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
