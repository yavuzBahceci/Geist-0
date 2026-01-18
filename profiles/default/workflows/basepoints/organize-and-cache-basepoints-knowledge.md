# Organize and Cache Basepoints Knowledge

## Core Responsibilities

1. **Structure Extracted Knowledge**: Organize knowledge by abstraction layer, category, and module
2. **Implement Per-Spec Caching**: Store extracted knowledge in the spec's implementation folder
3. **Provide Query Utilities**: Enable querying cached knowledge by category, layer, or module
4. **Manage Cache Lifecycle**: Handle cache creation, validation, and reuse across commands

## Workflow

### Step 1: Determine Cache Location (Per-Spec)

Caching strategy: **Per-spec caching** (not global, not per-command)

```bash
# SPEC_PATH should be set by the calling command
# This ensures knowledge is cached per-spec and reused across commands in the same spec

if [ -n "$SPEC_PATH" ]; then
    CACHE_PATH="$SPEC_PATH/implementation/cache"
    echo "📁 Per-spec cache location: $CACHE_PATH"
else
    # Fallback for non-spec contexts
    CACHE_PATH="agent-os/output/basepoints-extraction"
    echo "⚠️ No SPEC_PATH set. Using fallback cache: $CACHE_PATH"
fi

# Create cache directory structure
mkdir -p "$CACHE_PATH"
mkdir -p "$CACHE_PATH/on-demand"
```

### Step 2: Check Cache Validity

Determine if existing cache can be reused:

```bash
check_cache_validity() {
    CACHE_FILE="$CACHE_PATH/basepoints-knowledge.md"
    
    if [ ! -f "$CACHE_FILE" ]; then
        echo "CACHE_STATUS=missing"
        return 1
    fi
    
    # Check cache age (optional - cache is valid for the spec's lifetime)
    CACHE_MTIME=$(stat -f %m "$CACHE_FILE" 2>/dev/null || stat -c %Y "$CACHE_FILE" 2>/dev/null)
    CURRENT_TIME=$(date +%s)
    CACHE_AGE=$((CURRENT_TIME - CACHE_MTIME))
    
    # Cache is valid if it exists (per-spec caching means it's always valid within the spec)
    echo "✅ Cache found (age: ${CACHE_AGE}s)"
    echo "CACHE_STATUS=valid"
    return 0
}

# Run check
check_cache_validity
```

### Step 3: Check Cache Invalidation Conditions

Cache should be invalidated if:

```bash
should_invalidate_cache() {
    CACHE_FILE="$CACHE_PATH/basepoints-knowledge.md"
    BASEPOINTS_PATH="agent-os/basepoints"
    
    # Condition 1: No cache exists
    if [ ! -f "$CACHE_FILE" ]; then
        echo "invalidate:no_cache"
        return 0
    fi
    
    # Condition 2: Basepoints modified after cache creation
    if [ -d "$BASEPOINTS_PATH" ]; then
        CACHE_MTIME=$(stat -f %m "$CACHE_FILE" 2>/dev/null || stat -c %Y "$CACHE_FILE" 2>/dev/null)
        
        # Find most recent basepoint modification
        LATEST_BP_MTIME=$(find "$BASEPOINTS_PATH" -name "*.md" -type f -exec stat -f %m {} \; 2>/dev/null | sort -rn | head -1)
        if [ -z "$LATEST_BP_MTIME" ]; then
            LATEST_BP_MTIME=$(find "$BASEPOINTS_PATH" -name "*.md" -type f -exec stat -c %Y {} \; 2>/dev/null | sort -rn | head -1)
        fi
        
        if [ -n "$LATEST_BP_MTIME" ] && [ "$LATEST_BP_MTIME" -gt "$CACHE_MTIME" ]; then
            echo "invalidate:basepoints_updated"
            return 0
        fi
    fi
    
    # Condition 3: Force refresh requested
    if [ "$FORCE_REFRESH" = "true" ]; then
        echo "invalidate:force_refresh"
        return 0
    fi
    
    # Cache is valid
    echo "valid"
    return 1
}

# Check invalidation
INVALIDATION_REASON=$(should_invalidate_cache)
if [ "$INVALIDATION_REASON" != "valid" ]; then
    echo "🔄 Cache invalidation: $INVALIDATION_REASON"
    NEED_EXTRACTION="true"
else
    echo "✅ Cache is valid, reusing existing knowledge"
    NEED_EXTRACTION="false"
fi
```

### Step 4: Load or Extract Knowledge

```bash
if [ "$NEED_EXTRACTION" = "true" ]; then
    echo "📖 Extracting fresh knowledge from basepoints..."
    
    # Determine workflow base path (agent-os when installed, profiles/default for template)
    if [ -d "agent-os/workflows" ]; then
        WORKFLOWS_BASE="agent-os/workflows"
    else
        WORKFLOWS_BASE="profiles/default/workflows"
    fi
    
    # Call the automatic extraction workflow
    # This will populate the cache
    source "$WORKFLOWS_BASE/basepoints/extract-basepoints-knowledge-automatic.md"
    
    echo "✅ Fresh extraction complete"
else
    echo "📂 Loading knowledge from cache..."
    
    if [ -f "$CACHE_PATH/basepoints-knowledge.md" ]; then
        CACHED_KNOWLEDGE=$(cat "$CACHE_PATH/basepoints-knowledge.md")
        echo "✅ Loaded $(wc -l < "$CACHE_PATH/basepoints-knowledge.md" | tr -d ' ') lines from cache"
    fi
fi
```

### Step 5: Provide Query Functions

Enable querying cached knowledge:

```bash
# Query patterns from cache
query_patterns() {
    local LAYER="${1:-all}"
    local CACHE_FILE="$CACHE_PATH/basepoints-knowledge.md"
    
    if [ ! -f "$CACHE_FILE" ]; then
        echo ""
        return 1
    fi
    
    if [ "$LAYER" = "all" ]; then
        sed -n '/## Patterns/,/^## [^P]/p' "$CACHE_FILE" | head -n -1
    else
        # Extract patterns for specific layer
        sed -n "/### From:.*$LAYER/,/^### From:/p" "$CACHE_FILE" | head -n -1
    fi
}

# Query standards from cache
query_standards() {
    local CATEGORY="${1:-all}"
    local CACHE_FILE="$CACHE_PATH/basepoints-knowledge.md"
    
    if [ ! -f "$CACHE_FILE" ]; then
        echo ""
        return 1
    fi
    
    sed -n '/## Standards/,/^## [^S]/p' "$CACHE_FILE" | head -n -1
}

# Query flows from cache
query_flows() {
    local CACHE_FILE="$CACHE_PATH/basepoints-knowledge.md"
    
    if [ ! -f "$CACHE_FILE" ]; then
        echo ""
        return 1
    fi
    
    sed -n '/## Flows/,/^## [^F]/p' "$CACHE_FILE" | head -n -1
}

# Query strategies from cache
query_strategies() {
    local CACHE_FILE="$CACHE_PATH/basepoints-knowledge.md"
    
    if [ ! -f "$CACHE_FILE" ]; then
        echo ""
        return 1
    fi
    
    sed -n '/## Strategies/,/^## [^S]/p' "$CACHE_FILE" | head -n -1
}

# Query by module name
query_by_module() {
    local MODULE="$1"
    local CACHE_FILE="$CACHE_PATH/basepoints-knowledge.md"
    
    if [ ! -f "$CACHE_FILE" ]; then
        echo ""
        return 1
    fi
    
    grep -A 50 "### From:.*$MODULE" "$CACHE_FILE" | sed '/^### From:/q' | head -n -1
}

# Get detected abstraction layer
get_detected_layer() {
    local LAYER_FILE="$CACHE_PATH/detected-layer.txt"
    
    if [ -f "$LAYER_FILE" ]; then
        cat "$LAYER_FILE"
    else
        echo "unknown"
    fi
}
```

### Step 6: Cache Statistics

Provide cache information:

```bash
get_cache_stats() {
    echo "═══════════════════════════════════════════════════════"
    echo "  CACHE STATISTICS"
    echo "═══════════════════════════════════════════════════════"
    
    if [ -f "$CACHE_PATH/basepoints-knowledge.md" ]; then
        CACHE_SIZE=$(wc -c < "$CACHE_PATH/basepoints-knowledge.md" | tr -d ' ')
        CACHE_LINES=$(wc -l < "$CACHE_PATH/basepoints-knowledge.md" | tr -d ' ')
        CACHE_MTIME=$(stat -f %Sm -t "%Y-%m-%d %H:%M:%S" "$CACHE_PATH/basepoints-knowledge.md" 2>/dev/null || stat -c %y "$CACHE_PATH/basepoints-knowledge.md" 2>/dev/null | cut -d. -f1)
        
        echo "  Main Cache:"
        echo "    Path: $CACHE_PATH/basepoints-knowledge.md"
        echo "    Size: $CACHE_SIZE bytes"
        echo "    Lines: $CACHE_LINES"
        echo "    Modified: $CACHE_MTIME"
    else
        echo "  Main Cache: Not found"
    fi
    
    if [ -f "$CACHE_PATH/detected-layer.txt" ]; then
        DETECTED=$(cat "$CACHE_PATH/detected-layer.txt")
        echo "  Detected Layer: $DETECTED"
    fi
    
    ON_DEMAND_COUNT=$(find "$CACHE_PATH/on-demand" -name "*.md" 2>/dev/null | wc -l | tr -d ' ')
    echo "  On-Demand Extractions: $ON_DEMAND_COUNT"
    
    echo "═══════════════════════════════════════════════════════"
}

# Display stats
get_cache_stats
```

### Step 7: Cache Reuse Across Commands

The per-spec caching strategy ensures:

```bash
# IMPORTANT: Cache Reuse Strategy
# 
# 1. shape-spec: Extracts knowledge, creates cache
# 2. write-spec: Reuses cache from shape-spec (same SPEC_PATH)
# 3. create-tasks: Reuses cache
# 4. implement-tasks: Reuses cache
# 5. orchestrate-tasks: Reuses cache
#
# Cache is stored at: $SPEC_PATH/implementation/cache/
# This location is shared across all commands for the same spec

ensure_cache_available() {
    local CACHE_FILE="$CACHE_PATH/basepoints-knowledge.md"
    
    if [ -f "$CACHE_FILE" ]; then
        echo "✅ Cache available for reuse"
        return 0
    else
        echo "⚠️ Cache not found. Triggering extraction..."
        # Trigger extraction
        NEED_EXTRACTION="true"
        return 1
    fi
}
```

### Step 8: Export Functions for Commands

Make query functions available to calling commands:

```bash
# Export the cache path for other workflows
export BASEPOINTS_CACHE_PATH="$CACHE_PATH"

# Export query functions (in zsh/bash)
export -f query_patterns 2>/dev/null || true
export -f query_standards 2>/dev/null || true
export -f query_flows 2>/dev/null || true
export -f query_strategies 2>/dev/null || true
export -f query_by_module 2>/dev/null || true
export -f get_detected_layer 2>/dev/null || true

echo "✅ Cache utilities ready"
echo "   Use: query_patterns, query_standards, query_flows, query_strategies"
echo "   Use: query_by_module <module-name>"
echo "   Use: get_detected_layer"
```

## Cache File Structure

After extraction, the cache contains:

```
$SPEC_PATH/implementation/cache/
├── basepoints-knowledge.md       # Main knowledge cache
├── detected-layer.txt            # Detected abstraction layer
├── resources-consulted.md        # List of consulted sources
└── on-demand/                    # On-demand extraction results
    ├── on-demand-*.md            # Timestamped on-demand extractions
    └── ...
```

## Important Constraints

- **Per-spec caching**: All cached documents stored in `$SPEC_PATH/implementation/cache/`
- **Reuse across commands**: Cache is shared across all commands for the same spec
- **Invalidation conditions**: Basepoints updated, force refresh, or cache missing
- **Graceful fallback**: Works even when basepoints don't exist
- **Query utilities**: Functions to query cached knowledge by category/layer/module
- Must preserve source information for traceability
- Must be technology-agnostic and work with any basepoint structure
- **CRITICAL**: Never scatter cache files around the codebase; always use the spec's implementation folder
