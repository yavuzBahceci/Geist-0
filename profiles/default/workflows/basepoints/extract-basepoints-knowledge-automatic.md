# Automatic Basepoints Knowledge Extraction

## Core Responsibilities

1. **Read All Basepoint Files**: Traverse basepoints folder structure and read all basepoint files
2. **Extract Same-Layer Patterns**: Extract patterns specific to each abstraction layer
3. **Extract Cross-Layer Patterns**: Identify patterns spanning multiple abstraction layers
4. **Extract All Knowledge Categories**: Extract standards, flows, strategies, testing approaches, pros/cons, historical decisions
5. **Extract Libraries Used**: Extract `## Libraries Used` from module basepoints and `## Libraries Used (Aggregated)` from parent basepoints
6. **Organize Knowledge**: Structure extracted knowledge by category and abstraction layer
7. **Cache Knowledge**: Store extracted knowledge for use during command execution

## Inputs

- `SPEC_PATH` (optional): Path to spec for cache location
- `geist/basepoints/`: Basepoints folder containing headquarter.md and module basepoints

## Outputs

- `$CACHE_PATH/basepoints-knowledge.md`: Extracted basepoints knowledge including Libraries Used
- `$CACHE_PATH/detected-layer.txt`: Detected abstraction layers
- `$CACHE_PATH/resources-consulted.md`: List of resources consulted

## Workflow

### Step 1: Validate Basepoints Existence

Check if basepoints folder exists and contains basepoint files:

```bash
# Define paths
BASEPOINTS_PATH="geist/basepoints"
BASEPOINT_FILE_PATTERN="agent-base-*.md"

# Check if basepoints folder exists
if [ ! -d "$BASEPOINTS_PATH" ]; then
    echo "⚠️ Basepoints folder not found at $BASEPOINTS_PATH"
    echo "Continuing without basepoints knowledge. Run /create-basepoints to generate basepoints."
    # Set flag for graceful fallback
    BASEPOINTS_AVAILABLE="false"
else
    BASEPOINTS_AVAILABLE="true"
    
    # Check for headquarter.md
    if [ ! -f "$BASEPOINTS_PATH/headquarter.md" ]; then
        echo "⚠️ Warning: headquarter.md not found. Continuing with module basepoints only."
        HAS_HEADQUARTER="false"
    else
        HAS_HEADQUARTER="true"
        echo "✅ Found headquarter.md"
    fi
    
    # Count basepoint files
    BASEPOINT_COUNT=$(find "$BASEPOINTS_PATH" -name "$BASEPOINT_FILE_PATTERN" -type f 2>/dev/null | wc -l | tr -d ' ')
    if [ "$BASEPOINT_COUNT" -eq 0 ]; then
        echo "⚠️ No module basepoint files found. Using headquarter.md only."
    else
        echo "✅ Found $BASEPOINT_COUNT basepoint file(s)"
    fi
fi
```

**Graceful Fallback**: If basepoints don't exist, continue without knowledge extraction. Commands should still function, just without basepoints-derived context.

### Step 2: Determine Cache Location

Set up the cache path based on context:

```bash
# Determine spec path from context
# SPEC_PATH should be set by the calling command
if [ -n "$SPEC_PATH" ]; then
    CACHE_PATH="$SPEC_PATH/implementation/cache"
else
    # Fallback for non-spec contexts (e.g., deploy-agents)
    CACHE_PATH="geist/output/basepoints-extraction"
fi

# Create cache directory
mkdir -p "$CACHE_PATH"
echo "📁 Cache location: $CACHE_PATH"
```

### Step 3: Extract Abstraction Layers from Headquarter

If headquarter.md exists, extract the documented abstraction layers:

```bash
if [ "$HAS_HEADQUARTER" = "true" ]; then
    echo "📖 Extracting abstraction layers from headquarter.md..."
    
    # Read headquarter content
    HEADQUARTER_CONTENT=$(cat "$BASEPOINTS_PATH/headquarter.md")
    
    # Extract layers from "Detected Abstraction Layers" or "Architecture Overview" section
    # Look for table rows with layer names or bullet points
    ABSTRACTION_LAYERS=$(echo "$HEADQUARTER_CONTENT" | grep -E "^\| \*\*[A-Z]+\*\*|^- \*\*[A-Z]+\*\*" | sed 's/.*\*\*\([A-Z_]*\)\*\*.*/\1/' | tr '\n' ',' | sed 's/,$//')
    
    if [ -z "$ABSTRACTION_LAYERS" ]; then
        # Fallback: try to find layer mentions in headers
        ABSTRACTION_LAYERS=$(echo "$HEADQUARTER_CONTENT" | grep -E "^### Layer:" | sed 's/### Layer: //' | tr '\n' ',' | sed 's/,$//')
    fi
    
    if [ -z "$ABSTRACTION_LAYERS" ]; then
        # Default layers if none detected
        ABSTRACTION_LAYERS="ROOT,PROFILES,SCRIPTS"
    fi
    
    echo "✅ Detected layers: $ABSTRACTION_LAYERS"
    
    # Save detected layers
    echo "$ABSTRACTION_LAYERS" > "$CACHE_PATH/detected-layer.txt"
fi
```

### Step 4: Read and Extract from Headquarter.md

Extract top-level architecture and patterns from headquarter:

```bash
if [ "$HAS_HEADQUARTER" = "true" ]; then
    echo "📖 Extracting knowledge from headquarter.md..."
    
    # Initialize knowledge sections
    HQ_PATTERNS=""
    HQ_STANDARDS=""
    HQ_FLOWS=""
    HQ_STRATEGIES=""
    
    # Extract Architecture Patterns section
    HQ_PATTERNS=$(echo "$HEADQUARTER_CONTENT" | sed -n '/## Architecture Patterns/,/^## /p' | head -n -1)
    if [ -z "$HQ_PATTERNS" ]; then
        HQ_PATTERNS=$(echo "$HEADQUARTER_CONTENT" | sed -n '/### Core Patterns/,/^### /p' | head -n -1)
    fi
    
    # Extract Standards section
    HQ_STANDARDS=$(echo "$HEADQUARTER_CONTENT" | sed -n '/## Standards/,/^## /p' | head -n -1)
    if [ -z "$HQ_STANDARDS" ]; then
        HQ_STANDARDS=$(echo "$HEADQUARTER_CONTENT" | sed -n '/### Naming Conventions/,/^### /p' | head -n -1)
    fi
    
    # Extract Workflow Patterns / Flows
    HQ_FLOWS=$(echo "$HEADQUARTER_CONTENT" | sed -n '/## Development Workflow/,/^## /p' | head -n -1)
    if [ -z "$HQ_FLOWS" ]; then
        HQ_FLOWS=$(echo "$HEADQUARTER_CONTENT" | sed -n '/### Workflow Patterns/,/^### /p' | head -n -1)
    fi
    
    # Extract Strategies
    HQ_STRATEGIES=$(echo "$HEADQUARTER_CONTENT" | sed -n '/## Testing Strategy/,/^## /p' | head -n -1)
    
    echo "✅ Extracted from headquarter: patterns, standards, flows, strategies"
fi
```

### Step 5: Read All Module Basepoints

Traverse the basepoints folder and extract knowledge from each module:

```bash
if [ "$BASEPOINTS_AVAILABLE" = "true" ]; then
    echo "📖 Reading module basepoints..."
    
    # Initialize collections
    ALL_PATTERNS=""
    ALL_STANDARDS=""
    ALL_FLOWS=""
    ALL_STRATEGIES=""
    ALL_TESTING=""
    ALL_RELATED=""
    ALL_LIBRARIES_USED=""
    
    # Find all basepoint files
    find "$BASEPOINTS_PATH" -name "$BASEPOINT_FILE_PATTERN" -type f | sort | while read basepoint_file; do
        if [ -z "$basepoint_file" ]; then
            continue
        fi
        
        echo "  📄 Reading: $basepoint_file"
        
        # Determine module layer from path
        MODULE_PATH=$(dirname "$basepoint_file" | sed "s|$BASEPOINTS_PATH/||")
        MODULE_NAME=$(basename "$basepoint_file" .md | sed 's/agent-base-//')
        
        # Read content
        MODULE_CONTENT=$(cat "$basepoint_file")
        
        # Extract Patterns section
        PATTERNS=$(echo "$MODULE_CONTENT" | sed -n '/## Patterns/,/^## /p' | head -n -1)
        if [ -n "$PATTERNS" ]; then
            ALL_PATTERNS="${ALL_PATTERNS}

### From: $MODULE_PATH ($MODULE_NAME)
$PATTERNS"
        fi
        
        # Extract Standards section
        STANDARDS=$(echo "$MODULE_CONTENT" | sed -n '/## Standards/,/^## /p' | head -n -1)
        if [ -n "$STANDARDS" ]; then
            ALL_STANDARDS="${ALL_STANDARDS}

### From: $MODULE_PATH ($MODULE_NAME)
$STANDARDS"
        fi
        
        # Extract Flows section
        FLOWS=$(echo "$MODULE_CONTENT" | sed -n '/## Flows/,/^## /p' | head -n -1)
        if [ -n "$FLOWS" ]; then
            ALL_FLOWS="${ALL_FLOWS}

### From: $MODULE_PATH ($MODULE_NAME)
$FLOWS"
        fi
        
        # Extract Strategies section
        STRATEGIES=$(echo "$MODULE_CONTENT" | sed -n '/## Strategies/,/^## /p' | head -n -1)
        if [ -n "$STRATEGIES" ]; then
            ALL_STRATEGIES="${ALL_STRATEGIES}

### From: $MODULE_PATH ($MODULE_NAME)
$STRATEGIES"
        fi
        
        # Extract Testing section
        TESTING=$(echo "$MODULE_CONTENT" | sed -n '/## Testing/,/^## /p' | head -n -1)
        if [ -n "$TESTING" ]; then
            ALL_TESTING="${ALL_TESTING}

### From: $MODULE_PATH ($MODULE_NAME)
$TESTING"
        fi
        
        # Extract Related Modules section
        RELATED=$(echo "$MODULE_CONTENT" | sed -n '/## Related/,/^## /p' | head -n -1)
        if [ -n "$RELATED" ]; then
            ALL_RELATED="${ALL_RELATED}

### From: $MODULE_PATH ($MODULE_NAME)
$RELATED"
        fi
        
        # Extract Libraries Used section (from module basepoints)
        LIBRARIES_USED=$(echo "$MODULE_CONTENT" | sed -n '/^## Libraries Used$/,/^## /p' | head -n -1)
        if [ -n "$LIBRARIES_USED" ]; then
            ALL_LIBRARIES_USED="${ALL_LIBRARIES_USED}

### From: $MODULE_PATH ($MODULE_NAME)
$LIBRARIES_USED"
        fi
        
        # Extract Libraries Used (Aggregated) section (from parent basepoints)
        LIBRARIES_AGGREGATED=$(echo "$MODULE_CONTENT" | sed -n '/^## Libraries Used (Aggregated)/,/^## /p' | head -n -1)
        if [ -n "$LIBRARIES_AGGREGATED" ]; then
            ALL_LIBRARIES_USED="${ALL_LIBRARIES_USED}

### From: $MODULE_PATH ($MODULE_NAME) [Aggregated]
$LIBRARIES_AGGREGATED"
        fi
    done
fi
```

### Step 6: Compile and Cache Knowledge

Generate the basepoints-knowledge.md file:

```bash
echo "📝 Compiling extracted knowledge..."

# Create the comprehensive knowledge document
cat > "$CACHE_PATH/basepoints-knowledge.md" << 'KNOWLEDGE_EOF'
# Extracted Basepoints Knowledge

## Extraction Metadata
- **Extracted**: $(date)
- **Source**: geist/basepoints/
- **Basepoints Available**: $BASEPOINTS_AVAILABLE

---

## Detected Abstraction Layers

$ABSTRACTION_LAYERS

---

## Patterns

### From Headquarter (Cross-Layer)
$HQ_PATTERNS

### From Module Basepoints (Same-Layer)
$ALL_PATTERNS

---

## Standards

### Global Standards (from Headquarter)
$HQ_STANDARDS

### Module-Specific Standards
$ALL_STANDARDS

---

## Flows

### Project-Level Flows (from Headquarter)
$HQ_FLOWS

### Module-Level Flows
$ALL_FLOWS

---

## Strategies

### Global Strategies
$HQ_STRATEGIES

### Module-Specific Strategies
$ALL_STRATEGIES

---

## Testing Approaches

$ALL_TESTING

---

## Related Modules and Dependencies

$ALL_RELATED

---

## Libraries Used

Library usage patterns extracted from module and parent basepoints:

$ALL_LIBRARIES_USED

---

*Knowledge extracted automatically from basepoints.*
KNOWLEDGE_EOF

echo "✅ Knowledge cached to: $CACHE_PATH/basepoints-knowledge.md"
```

### Step 7: Generate Resources Checklist

Track which resources were consulted:

```bash
echo "📋 Generating resources checklist..."

cat > "$CACHE_PATH/resources-consulted.md" << 'RESOURCES_EOF'
# Resources Consulted

## Basepoints
RESOURCES_EOF

if [ "$HAS_HEADQUARTER" = "true" ]; then
    echo "- [x] \`$BASEPOINTS_PATH/headquarter.md\`" >> "$CACHE_PATH/resources-consulted.md"
else
    echo "- [ ] \`$BASEPOINTS_PATH/headquarter.md\` (not found)" >> "$CACHE_PATH/resources-consulted.md"
fi

# List all module basepoints consulted
find "$BASEPOINTS_PATH" -name "$BASEPOINT_FILE_PATTERN" -type f 2>/dev/null | sort | while read f; do
    echo "- [x] \`$f\`" >> "$CACHE_PATH/resources-consulted.md"
done

cat >> "$CACHE_PATH/resources-consulted.md" << 'RESOURCES_EOF'

## Product Files
- [ ] `geist/product/mission.md` (check separately)
- [ ] `geist/product/roadmap.md` (check separately)
- [ ] `geist/product/tech-stack.md` (check separately)

## Standards
- [ ] `geist/standards/global/*.md` (check separately)

---

*Generated: $(date)*
RESOURCES_EOF

echo "✅ Resources checklist generated"
```

### Step 8: Return Status

Provide summary of extraction:

```bash
# Count libraries extracted
LIBRARIES_USED_COUNT=$(echo "$ALL_LIBRARIES_USED" | grep -c "^### From:" || echo "0")

echo ""
echo "═══════════════════════════════════════════════════════"
echo "  BASEPOINTS KNOWLEDGE EXTRACTION COMPLETE"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Basepoints Available: $BASEPOINTS_AVAILABLE"
echo "  Headquarter Found: $HAS_HEADQUARTER"
echo "  Module Basepoints: $BASEPOINT_COUNT"
echo "  Libraries Used Sections: $LIBRARIES_USED_COUNT"
echo "  Detected Layers: $ABSTRACTION_LAYERS"
echo ""
echo "  Cache Location: $CACHE_PATH"
echo "  Files Created:"
echo "    - basepoints-knowledge.md"
echo "    - detected-layer.txt"
echo "    - resources-consulted.md"
echo ""
echo "═══════════════════════════════════════════════════════"
```

## Graceful Fallback Behavior

When basepoints don't exist or are incomplete:

1. **No basepoints folder**: Set `BASEPOINTS_AVAILABLE=false`, continue without knowledge
2. **No headquarter.md**: Use module basepoints only, default layer names
3. **No module basepoints**: Use headquarter.md only
4. **Empty sections**: Skip empty sections in output

Commands should check `BASEPOINTS_AVAILABLE` flag and adjust behavior accordingly.

## Important Constraints

- Must read all basepoint files including headquarter.md and all module-specific files
- Must extract patterns, standards, flows, strategies, testing approaches
- Must extract `## Libraries Used` from module basepoints
- Must extract `## Libraries Used (Aggregated)` from parent basepoints
- Must organize knowledge by abstraction layer and category
- Must preserve source information (which basepoint file each piece of knowledge came from)
- Must cache extracted knowledge to `$SPEC_PATH/implementation/cache/`
- Must provide graceful fallback when basepoints don't exist
- Must be technology-agnostic and work with any basepoint structure
- **CRITICAL**: All cached documents must be stored in `$SPEC_PATH/implementation/cache/` when running within a spec command
