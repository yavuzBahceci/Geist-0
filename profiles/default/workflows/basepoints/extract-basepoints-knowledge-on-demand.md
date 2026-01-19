# On-Demand Basepoints Knowledge Extraction

## Core Responsibilities

1. **Receive Extraction Request**: Accept specific knowledge categories, modules, or abstraction layers to extract
2. **Targeted Extraction**: Extract only the requested knowledge categories from relevant basepoints
3. **Scope Filtering**: Filter extraction to specific modules or abstraction layers when requested
4. **Merge with Cache**: Merge newly extracted knowledge with existing cache
5. **Return Extracted Knowledge**: Provide extracted knowledge in organized format

## Workflow

### Step 1: Receive Extraction Request

Accept parameters for on-demand extraction:

```bash
# Parameters that may be provided by the calling command:
# - KNOWLEDGE_CATEGORIES: Specific categories to extract
#   Options: patterns, standards, flows, strategies, testing, related_modules, all
# - TARGET_MODULES: Specific modules to extract from (comma-separated)
#   Example: "commands,workflows,standards"
# - TARGET_LAYER: Specific abstraction layer to focus on
#   Example: "PROFILES" or "SCRIPTS"
# - FEATURE_DESCRIPTION: Feature text for relevance matching
# - MERGE_WITH_CACHE: Whether to merge with existing cache (true/false)

# Set defaults
KNOWLEDGE_CATEGORIES="${KNOWLEDGE_CATEGORIES:-all}"
TARGET_MODULES="${TARGET_MODULES:-}"
TARGET_LAYER="${TARGET_LAYER:-}"
FEATURE_DESCRIPTION="${FEATURE_DESCRIPTION:-}"
MERGE_WITH_CACHE="${MERGE_WITH_CACHE:-true}"

echo "📋 On-Demand Extraction Request:"
echo "   Categories: $KNOWLEDGE_CATEGORIES"
echo "   Target Modules: ${TARGET_MODULES:-all}"
echo "   Target Layer: ${TARGET_LAYER:-all}"
echo "   Merge with cache: $MERGE_WITH_CACHE"
```

### Step 2: Determine Cache and Basepoints Paths

```bash
# Define paths
BASEPOINTS_PATH="geist/basepoints"
BASEPOINT_FILE_PATTERN="agent-base-*.md"

# Determine cache path from context
if [ -n "$SPEC_PATH" ]; then
    CACHE_PATH="$SPEC_PATH/implementation/cache"
else
    CACHE_PATH="geist/output/basepoints-extraction"
fi

# Create cache directory if needed
mkdir -p "$CACHE_PATH/on-demand"

# Check basepoints availability
if [ ! -d "$BASEPOINTS_PATH" ]; then
    echo "⚠️ Basepoints not available. Returning empty result."
    BASEPOINTS_AVAILABLE="false"
else
    BASEPOINTS_AVAILABLE="true"
fi
```

### Step 3: Check Existing Cache

Check if requested knowledge is already cached:

```bash
if [ "$MERGE_WITH_CACHE" = "true" ] && [ -f "$CACHE_PATH/basepoints-knowledge.md" ]; then
    echo "✅ Found existing cache at $CACHE_PATH/basepoints-knowledge.md"
    EXISTING_CACHE=$(cat "$CACHE_PATH/basepoints-knowledge.md")
    HAS_EXISTING_CACHE="true"
else
    HAS_EXISTING_CACHE="false"
fi
```

### Step 4: Determine Target Basepoint Files

Identify which basepoint files to read based on request:

```bash
TARGET_FILES=""

if [ "$BASEPOINTS_AVAILABLE" = "true" ]; then
    # Always include headquarter.md for cross-layer context
    if [ -f "$BASEPOINTS_PATH/headquarter.md" ]; then
        TARGET_FILES="$BASEPOINTS_PATH/headquarter.md"
    fi
    
    # If specific modules requested
    if [ -n "$TARGET_MODULES" ]; then
        echo "🎯 Filtering to specific modules: $TARGET_MODULES"
        
        # Convert comma-separated to newline for processing
        echo "$TARGET_MODULES" | tr ',' '\n' | while read module; do
            module=$(echo "$module" | tr -d ' ')
            # Find basepoint files matching the module name
            MATCHING=$(find "$BASEPOINTS_PATH" -name "*$module*" -o -path "*/$module/*" -name "$BASEPOINT_FILE_PATTERN" 2>/dev/null)
            if [ -n "$MATCHING" ]; then
                TARGET_FILES="${TARGET_FILES}
$MATCHING"
            fi
        done
    # If specific layer requested
    elif [ -n "$TARGET_LAYER" ]; then
        echo "🎯 Filtering to layer: $TARGET_LAYER"
        
        # Find basepoint files in the target layer's directory
        # Layer names typically map to directory names
        LAYER_DIR=$(echo "$TARGET_LAYER" | tr '[:upper:]' '[:lower:]')
        MATCHING=$(find "$BASEPOINTS_PATH" -path "*/$LAYER_DIR/*" -name "$BASEPOINT_FILE_PATTERN" 2>/dev/null)
        if [ -n "$MATCHING" ]; then
            TARGET_FILES="${TARGET_FILES}
$MATCHING"
        fi
    else
        # Get all basepoint files
        TARGET_FILES="${TARGET_FILES}
$(find "$BASEPOINTS_PATH" -name "$BASEPOINT_FILE_PATTERN" -type f 2>/dev/null)"
    fi
    
    # Remove empty lines and duplicates
    TARGET_FILES=$(echo "$TARGET_FILES" | grep -v '^$' | sort -u)
    
    FILE_COUNT=$(echo "$TARGET_FILES" | grep -c '.' || echo "0")
    echo "📁 Found $FILE_COUNT target file(s)"
fi
```

### Step 5: Extract Requested Knowledge Categories

Extract only the requested categories from target files:

```bash
# Initialize extraction results
EXTRACTED_PATTERNS=""
EXTRACTED_STANDARDS=""
EXTRACTED_FLOWS=""
EXTRACTED_STRATEGIES=""
EXTRACTED_TESTING=""
EXTRACTED_RELATED=""

if [ "$BASEPOINTS_AVAILABLE" = "true" ] && [ -n "$TARGET_FILES" ]; then
    echo "$TARGET_FILES" | while read basepoint_file; do
        if [ -z "$basepoint_file" ] || [ ! -f "$basepoint_file" ]; then
            continue
        fi
        
        echo "  📄 Extracting from: $basepoint_file"
        
        # Read content
        CONTENT=$(cat "$basepoint_file")
        MODULE_NAME=$(basename "$basepoint_file" .md)
        
        # Extract Patterns if requested
        if [[ "$KNOWLEDGE_CATEGORIES" == *"patterns"* ]] || [[ "$KNOWLEDGE_CATEGORIES" == "all" ]]; then
            SECTION=$(echo "$CONTENT" | sed -n '/## Patterns/,/^## /p' | head -n -1)
            if [ -z "$SECTION" ]; then
                SECTION=$(echo "$CONTENT" | sed -n '/### Core Patterns/,/^### /p' | head -n -1)
            fi
            if [ -z "$SECTION" ]; then
                SECTION=$(echo "$CONTENT" | sed -n '/## Architecture Patterns/,/^## /p' | head -n -1)
            fi
            if [ -n "$SECTION" ]; then
                EXTRACTED_PATTERNS="${EXTRACTED_PATTERNS}

### From: $MODULE_NAME
$SECTION"
            fi
        fi
        
        # Extract Standards if requested
        if [[ "$KNOWLEDGE_CATEGORIES" == *"standards"* ]] || [[ "$KNOWLEDGE_CATEGORIES" == "all" ]]; then
            SECTION=$(echo "$CONTENT" | sed -n '/## Standards/,/^## /p' | head -n -1)
            if [ -n "$SECTION" ]; then
                EXTRACTED_STANDARDS="${EXTRACTED_STANDARDS}

### From: $MODULE_NAME
$SECTION"
            fi
        fi
        
        # Extract Flows if requested
        if [[ "$KNOWLEDGE_CATEGORIES" == *"flows"* ]] || [[ "$KNOWLEDGE_CATEGORIES" == "all" ]]; then
            SECTION=$(echo "$CONTENT" | sed -n '/## Flows/,/^## /p' | head -n -1)
            if [ -z "$SECTION" ]; then
                SECTION=$(echo "$CONTENT" | sed -n '/## Development Workflow/,/^## /p' | head -n -1)
            fi
            if [ -n "$SECTION" ]; then
                EXTRACTED_FLOWS="${EXTRACTED_FLOWS}

### From: $MODULE_NAME
$SECTION"
            fi
        fi
        
        # Extract Strategies if requested
        if [[ "$KNOWLEDGE_CATEGORIES" == *"strategies"* ]] || [[ "$KNOWLEDGE_CATEGORIES" == "all" ]]; then
            SECTION=$(echo "$CONTENT" | sed -n '/## Strategies/,/^## /p' | head -n -1)
            if [ -n "$SECTION" ]; then
                EXTRACTED_STRATEGIES="${EXTRACTED_STRATEGIES}

### From: $MODULE_NAME
$SECTION"
            fi
        fi
        
        # Extract Testing if requested
        if [[ "$KNOWLEDGE_CATEGORIES" == *"testing"* ]] || [[ "$KNOWLEDGE_CATEGORIES" == "all" ]]; then
            SECTION=$(echo "$CONTENT" | sed -n '/## Testing/,/^## /p' | head -n -1)
            if [ -n "$SECTION" ]; then
                EXTRACTED_TESTING="${EXTRACTED_TESTING}

### From: $MODULE_NAME
$SECTION"
            fi
        fi
        
        # Extract Related Modules if requested
        if [[ "$KNOWLEDGE_CATEGORIES" == *"related"* ]] || [[ "$KNOWLEDGE_CATEGORIES" == "all" ]]; then
            SECTION=$(echo "$CONTENT" | sed -n '/## Related/,/^## /p' | head -n -1)
            if [ -n "$SECTION" ]; then
                EXTRACTED_RELATED="${EXTRACTED_RELATED}

### From: $MODULE_NAME
$SECTION"
            fi
        fi
    done
fi
```

### Step 6: Relevance Scoring (if feature description provided)

Score extracted knowledge for relevance to the feature:

```bash
if [ -n "$FEATURE_DESCRIPTION" ]; then
    echo "🎯 Scoring relevance to: $FEATURE_DESCRIPTION"
    
    # Extract keywords from feature description
    KEYWORDS=$(echo "$FEATURE_DESCRIPTION" | tr '[:upper:]' '[:lower:]' | tr -cs '[:alnum:]' '\n' | sort -u | grep -v '^$')
    
    # Score each extracted section by keyword matches
    # Higher score = more relevant
    # This is a simple keyword-based scoring; AI can do semantic scoring
    
    echo "   Keywords extracted: $(echo "$KEYWORDS" | tr '\n' ' ')"
fi
```

### Step 7: Cache On-Demand Extraction

Store extracted knowledge:

```bash
# Generate cache key based on request
CACHE_KEY="on-demand-$(date +%Y%m%d%H%M%S)"
if [ -n "$TARGET_LAYER" ]; then
    CACHE_KEY="${CACHE_KEY}-${TARGET_LAYER}"
fi
if [ -n "$TARGET_MODULES" ]; then
    CACHE_KEY="${CACHE_KEY}-$(echo "$TARGET_MODULES" | tr ',' '-')"
fi

# Create on-demand extraction result
cat > "$CACHE_PATH/on-demand/$CACHE_KEY.md" << ONDEMAND_EOF
# On-Demand Extracted Knowledge

## Request Parameters
- **Categories**: $KNOWLEDGE_CATEGORIES
- **Target Modules**: ${TARGET_MODULES:-all}
- **Target Layer**: ${TARGET_LAYER:-all}
- **Feature Description**: ${FEATURE_DESCRIPTION:-none}
- **Extracted**: $(date)

---

## Extracted Patterns
$EXTRACTED_PATTERNS

---

## Extracted Standards
$EXTRACTED_STANDARDS

---

## Extracted Flows
$EXTRACTED_FLOWS

---

## Extracted Strategies
$EXTRACTED_STRATEGIES

---

## Extracted Testing Approaches
$EXTRACTED_TESTING

---

## Extracted Related Modules
$EXTRACTED_RELATED

ONDEMAND_EOF

echo "✅ On-demand extraction cached to: $CACHE_PATH/on-demand/$CACHE_KEY.md"
```

### Step 8: Merge with Main Cache (if requested)

```bash
if [ "$MERGE_WITH_CACHE" = "true" ] && [ "$HAS_EXISTING_CACHE" = "true" ]; then
    echo "🔄 Merging with existing cache..."
    
    # Append new findings to main cache with timestamp
    cat >> "$CACHE_PATH/basepoints-knowledge.md" << MERGE_EOF

---

## Additional On-Demand Extraction ($CACHE_KEY)

**Request**: Categories=$KNOWLEDGE_CATEGORIES, Layer=$TARGET_LAYER, Modules=$TARGET_MODULES

$EXTRACTED_PATTERNS
$EXTRACTED_STANDARDS
$EXTRACTED_FLOWS
$EXTRACTED_STRATEGIES

MERGE_EOF

    echo "✅ Merged with main cache"
fi
```

### Step 9: Return Results

```bash
echo ""
echo "═══════════════════════════════════════════════════════"
echo "  ON-DEMAND EXTRACTION COMPLETE"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Categories: $KNOWLEDGE_CATEGORIES"
echo "  Files Processed: $FILE_COUNT"
echo "  Cache Key: $CACHE_KEY"
echo ""
echo "  Results at: $CACHE_PATH/on-demand/$CACHE_KEY.md"
echo ""
echo "═══════════════════════════════════════════════════════"

# Return path for calling command to use
echo "$CACHE_PATH/on-demand/$CACHE_KEY.md"
```

## Usage Examples

### Extract only patterns for a specific layer:
```bash
KNOWLEDGE_CATEGORIES="patterns"
TARGET_LAYER="PROFILES"
# Run workflow
```

### Extract all knowledge for specific modules:
```bash
KNOWLEDGE_CATEGORIES="all"
TARGET_MODULES="commands,workflows"
# Run workflow
```

### Extract with feature relevance scoring:
```bash
KNOWLEDGE_CATEGORIES="patterns,standards"
FEATURE_DESCRIPTION="Add validation to spec commands"
# Run workflow
```

## Important Constraints

- Must support extraction of specific knowledge categories on demand
- Must support targeted extraction for specific modules or abstraction layers
- Must cache on-demand extractions to avoid redundant reads
- Must preserve source information for traceability
- Must provide graceful fallback when basepoints don't exist
- Must support merging with existing cache
- Must be technology-agnostic and work with any basepoint structure
- **CRITICAL**: All cached documents must be stored in `$SPEC_PATH/implementation/cache/` when running within a spec command
