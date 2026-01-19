# Extract Library Basepoints Knowledge

## Core Responsibilities

1. **Check Library Basepoints Availability**: Verify that library basepoints exist before attempting extraction
2. **Traverse Flat Library Structure**: Navigate flat folder structure (geist/basepoints/libraries/*.md)
3. **Extract Library Knowledge**: Extract Overview, Our Usage - Deep Knowledge, Opportunities, Relevant Gotchas from each library basepoint
4. **Organize as Flat List**: Structure extracted knowledge as a flat list of libraries
5. **Cache Library Knowledge**: Store extracted library knowledge for use during command execution

## Inputs

- `SPEC_PATH` (optional): Path to spec for cache location
- `geist/basepoints/libraries/*.md`: Library basepoint files in flat structure

## Outputs

- `$CACHE_PATH/library-basepoints-knowledge.md`: Extracted library knowledge
- `$CACHE_PATH/library-index.md`: Index of all library basepoints

## Workflow

### Step 1: Check if Library Basepoints Exist

Before attempting extraction, verify that library basepoints are available:

```bash
# Define library basepoints path
LIBRARY_BASEPOINTS_PATH="geist/basepoints/libraries"

# Check if library basepoints exist
if [ -d "$LIBRARY_BASEPOINTS_PATH" ]; then
    LIBRARY_BASEPOINTS_AVAILABLE="true"
    echo "✅ Library basepoints folder found"
    
    # Count library basepoint files (flat structure - direct children only)
    LIBRARY_COUNT=$(find "$LIBRARY_BASEPOINTS_PATH" -maxdepth 1 -name "*.md" -type f ! -name "README.md" 2>/dev/null | wc -l | tr -d ' ')
    echo "   Found $LIBRARY_COUNT library basepoint file(s)"
else
    LIBRARY_BASEPOINTS_AVAILABLE="false"
    echo "⚠️ Library basepoints not found at $LIBRARY_BASEPOINTS_PATH"
    echo "   Continuing without library basepoints knowledge."
    # Exit gracefully - library basepoints are optional
    return 0
fi
```

### Step 2: Determine Cache Location

Set up the cache path based on context:

```bash
# Determine spec path from context
# SPEC_PATH should be set by the calling command
if [ -n "$SPEC_PATH" ]; then
    CACHE_PATH="$SPEC_PATH/implementation/cache"
else
    # Fallback for non-spec contexts
    CACHE_PATH="geist/output/library-basepoints-extraction"
fi

# Create cache directory
mkdir -p "$CACHE_PATH"
echo "📁 Cache location: $CACHE_PATH"
```

### Step 3: Extract Knowledge from Flat Library Structure

Traverse the flat libraries folder and extract knowledge from each library basepoint:

```bash
if [ "$LIBRARY_BASEPOINTS_AVAILABLE" = "true" ]; then
    echo "📖 Extracting library basepoints knowledge..."
    
    # Initialize collections for new template sections
    ALL_LIBRARY_OVERVIEW=""
    ALL_LIBRARY_USAGE=""
    ALL_LIBRARY_OPPORTUNITIES=""
    ALL_LIBRARY_GOTCHAS=""
    
    # Process each library file in flat structure
    for library_file in "$LIBRARY_BASEPOINTS_PATH"/*.md; do
        # Skip README.md
        if [ "$(basename "$library_file")" = "README.md" ]; then
            continue
        fi
        
        # Skip if no files found (glob didn't match)
        if [ ! -f "$library_file" ]; then
            continue
        fi
        
        echo "  📄 Reading: $library_file"
        
        # Determine library name from file
        LIBRARY_NAME=$(basename "$library_file" .md)
        
        # Read content
        LIBRARY_CONTENT=$(cat "$library_file")
        
        # Extract Overview section
        OVERVIEW=$(echo "$LIBRARY_CONTENT" | sed -n '/^## Overview/,/^## /p' | head -n -1)
        if [ -n "$OVERVIEW" ]; then
            ALL_LIBRARY_OVERVIEW="${ALL_LIBRARY_OVERVIEW}

### $LIBRARY_NAME
$OVERVIEW"
        fi
        
        # Extract Our Usage - Deep Knowledge section
        USAGE=$(echo "$LIBRARY_CONTENT" | sed -n '/^## Our Usage - Deep Knowledge/,/^## /p' | head -n -1)
        if [ -n "$USAGE" ]; then
            ALL_LIBRARY_USAGE="${ALL_LIBRARY_USAGE}

### $LIBRARY_NAME
$USAGE"
        fi
        
        # Extract Opportunities - Surface Knowledge section
        OPPORTUNITIES=$(echo "$LIBRARY_CONTENT" | sed -n '/^## Opportunities - Surface Knowledge/,/^## /p' | head -n -1)
        if [ -z "$OPPORTUNITIES" ]; then
            # Try alternative header
            OPPORTUNITIES=$(echo "$LIBRARY_CONTENT" | sed -n '/^## Opportunities/,/^## /p' | head -n -1)
        fi
        if [ -n "$OPPORTUNITIES" ]; then
            ALL_LIBRARY_OPPORTUNITIES="${ALL_LIBRARY_OPPORTUNITIES}

### $LIBRARY_NAME
$OPPORTUNITIES"
        fi
        
        # Extract Relevant Gotchas section
        GOTCHAS=$(echo "$LIBRARY_CONTENT" | sed -n '/^## Relevant Gotchas/,/^## /p' | head -n -1)
        if [ -n "$GOTCHAS" ]; then
            ALL_LIBRARY_GOTCHAS="${ALL_LIBRARY_GOTCHAS}

### $LIBRARY_NAME
$GOTCHAS"
        fi
    done
fi
```

### Step 4: Compile and Cache Library Knowledge

Generate the library-basepoints-knowledge.md file:

```bash
echo "📝 Compiling library basepoints knowledge..."

# Create the comprehensive library knowledge document
cat > "$CACHE_PATH/library-basepoints-knowledge.md" << 'LIBRARY_KNOWLEDGE_EOF'
# Extracted Library Basepoints Knowledge

## Extraction Metadata
- **Extracted**: $(date)
- **Source**: geist/basepoints/libraries/
- **Library Basepoints Available**: $LIBRARY_BASEPOINTS_AVAILABLE
- **Library Count**: $LIBRARY_COUNT

---

## Library Overview

Brief descriptions and purposes of each library:

$ALL_LIBRARY_OVERVIEW

---

## Our Usage - Deep Knowledge

How we actually use each library in this project - patterns, configurations, and implementation details:

$ALL_LIBRARY_USAGE

---

## Opportunities - Surface Knowledge

Features and capabilities we're aware of but haven't fully utilized:

$ALL_LIBRARY_OPPORTUNITIES

---

## Relevant Gotchas

Critical issues, version-specific problems, and known limitations:

$ALL_LIBRARY_GOTCHAS

---

*Library knowledge extracted automatically from library basepoints.*
LIBRARY_KNOWLEDGE_EOF

echo "✅ Library knowledge cached to: $CACHE_PATH/library-basepoints-knowledge.md"
```

### Step 5: Generate Library Index

Create a flat index of all library basepoints for quick reference:

```bash
echo "📋 Generating library index..."

cat > "$CACHE_PATH/library-index.md" << 'INDEX_EOF'
# Library Basepoints Index

## Libraries

INDEX_EOF

# Add each library (flat structure)
for library_file in "$LIBRARY_BASEPOINTS_PATH"/*.md; do
    # Skip README.md
    if [ "$(basename "$library_file")" = "README.md" ]; then
        continue
    fi
    
    # Skip if no files found
    if [ ! -f "$library_file" ]; then
        continue
    fi
    
    LIBRARY_NAME=$(basename "$library_file" .md)
    echo "- [$LIBRARY_NAME]($library_file)" >> "$CACHE_PATH/library-index.md"
done

cat >> "$CACHE_PATH/library-index.md" << 'INDEX_EOF'

---

*Generated: $(date)*
INDEX_EOF

echo "✅ Library index generated"
```

### Step 6: Return Status

Provide summary of extraction:

```bash
echo ""
echo "═══════════════════════════════════════════════════════"
echo "  LIBRARY BASEPOINTS KNOWLEDGE EXTRACTION COMPLETE"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Library Basepoints Available: $LIBRARY_BASEPOINTS_AVAILABLE"
echo "  Libraries Found: $LIBRARY_COUNT"
echo ""
echo "  Cache Location: $CACHE_PATH"
echo "  Files Created:"
echo "    - library-basepoints-knowledge.md"
echo "    - library-index.md"
echo ""
echo "═══════════════════════════════════════════════════════"
```

## Graceful Fallback Behavior

When library basepoints don't exist or are incomplete:

1. **No libraries folder**: Set `LIBRARY_BASEPOINTS_AVAILABLE=false`, continue without library knowledge
2. **No library files**: Skip if glob doesn't match any files
3. **Missing sections**: Skip missing sections in library basepoints
4. **Empty extraction**: Generate empty knowledge file with metadata

Commands should check `LIBRARY_BASEPOINTS_AVAILABLE` flag and adjust behavior accordingly.

## Important Constraints

- Must traverse the flat folder structure (`geist/basepoints/libraries/*.md`)
- Must extract: Overview, Our Usage - Deep Knowledge, Opportunities, Relevant Gotchas
- Must organize knowledge as flat library list (no categories)
- Must preserve source information (which library file each piece of knowledge came from)
- Must cache extracted knowledge to `$SPEC_PATH/implementation/cache/`
- Must provide graceful fallback when library basepoints don't exist
- Must be technology-agnostic and work with any library basepoint structure
- **CRITICAL**: All cached documents must be stored in `$SPEC_PATH/implementation/cache/` when running within a spec command
