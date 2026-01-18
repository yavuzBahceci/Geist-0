# Extract Library Basepoints Knowledge

## Core Responsibilities

1. **Check Library Basepoints Availability**: Verify that library basepoints exist before attempting extraction
2. **Traverse Library Categories**: Navigate category-based folder structure (data, domain, util, infrastructure, framework)
3. **Extract Library Knowledge**: Extract patterns, workflows, best practices, troubleshooting guidance from each library basepoint
4. **Organize by Category**: Structure extracted knowledge by library category and importance
5. **Cache Library Knowledge**: Store extracted library knowledge for use during command execution

## Workflow

### Step 1: Check if Library Basepoints Exist

Before attempting extraction, verify that library basepoints are available:

```bash
# Define library basepoints path
LIBRARY_BASEPOINTS_PATH="agent-os/basepoints/libraries"

# Check if library basepoints exist
if [ -d "$LIBRARY_BASEPOINTS_PATH" ]; then
    LIBRARY_BASEPOINTS_AVAILABLE="true"
    echo "✅ Library basepoints folder found"
    
    # Count library basepoint files
    LIBRARY_COUNT=$(find "$LIBRARY_BASEPOINTS_PATH" -name "*.md" -type f 2>/dev/null | wc -l | tr -d ' ')
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
    CACHE_PATH="agent-os/output/library-basepoints-extraction"
fi

# Create cache directory
mkdir -p "$CACHE_PATH"
echo "📁 Cache location: $CACHE_PATH"
```

### Step 3: Define Library Categories

Define the supported library categories:

```bash
# Define library categories
# These match the folder structure under agent-os/basepoints/libraries/
LIBRARY_CATEGORIES="data domain util infrastructure framework"

echo "📚 Library categories: $LIBRARY_CATEGORIES"
```

### Step 4: Extract Knowledge from Each Category

Traverse each category and extract knowledge from library basepoints:

```bash
if [ "$LIBRARY_BASEPOINTS_AVAILABLE" = "true" ]; then
    echo "📖 Extracting library basepoints knowledge..."
    
    # Initialize collections by category
    ALL_LIBRARY_PATTERNS=""
    ALL_LIBRARY_WORKFLOWS=""
    ALL_LIBRARY_BEST_PRACTICES=""
    ALL_LIBRARY_TROUBLESHOOTING=""
    ALL_LIBRARY_BOUNDARIES=""
    
    # Process each category
    for category in $LIBRARY_CATEGORIES; do
        CATEGORY_PATH="$LIBRARY_BASEPOINTS_PATH/$category"
        
        if [ -d "$CATEGORY_PATH" ]; then
            echo "  📂 Processing category: $category"
            
            # Find all library basepoint files in this category (including nested folders)
            find "$CATEGORY_PATH" -name "*.md" -type f | sort | while read library_file; do
                if [ -z "$library_file" ]; then
                    continue
                fi
                
                echo "    📄 Reading: $library_file"
                
                # Determine library name from file path
                LIBRARY_NAME=$(basename "$library_file" .md)
                RELATIVE_PATH=$(echo "$library_file" | sed "s|$LIBRARY_BASEPOINTS_PATH/||")
                
                # Read content
                LIBRARY_CONTENT=$(cat "$library_file")
                
                # Extract Patterns section
                PATTERNS=$(echo "$LIBRARY_CONTENT" | sed -n '/## Patterns/,/^## /p' | head -n -1)
                if [ -z "$PATTERNS" ]; then
                    PATTERNS=$(echo "$LIBRARY_CONTENT" | sed -n '/### Usage Patterns/,/^### /p' | head -n -1)
                fi
                if [ -n "$PATTERNS" ]; then
                    ALL_LIBRARY_PATTERNS="${ALL_LIBRARY_PATTERNS}

### $category/$LIBRARY_NAME
$PATTERNS"
                fi
                
                # Extract Workflows section
                WORKFLOWS=$(echo "$LIBRARY_CONTENT" | sed -n '/## Workflows/,/^## /p' | head -n -1)
                if [ -z "$WORKFLOWS" ]; then
                    WORKFLOWS=$(echo "$LIBRARY_CONTENT" | sed -n '/### Internal Workflows/,/^### /p' | head -n -1)
                fi
                if [ -n "$WORKFLOWS" ]; then
                    ALL_LIBRARY_WORKFLOWS="${ALL_LIBRARY_WORKFLOWS}

### $category/$LIBRARY_NAME
$WORKFLOWS"
                fi
                
                # Extract Best Practices section
                BEST_PRACTICES=$(echo "$LIBRARY_CONTENT" | sed -n '/## Best Practices/,/^## /p' | head -n -1)
                if [ -z "$BEST_PRACTICES" ]; then
                    BEST_PRACTICES=$(echo "$LIBRARY_CONTENT" | sed -n '/### Official Guidelines/,/^### /p' | head -n -1)
                fi
                if [ -n "$BEST_PRACTICES" ]; then
                    ALL_LIBRARY_BEST_PRACTICES="${ALL_LIBRARY_BEST_PRACTICES}

### $category/$LIBRARY_NAME
$BEST_PRACTICES"
                fi
                
                # Extract Troubleshooting section
                TROUBLESHOOTING=$(echo "$LIBRARY_CONTENT" | sed -n '/## Troubleshooting/,/^## /p' | head -n -1)
                if [ -z "$TROUBLESHOOTING" ]; then
                    TROUBLESHOOTING=$(echo "$LIBRARY_CONTENT" | sed -n '/### Common Issues/,/^### /p' | head -n -1)
                fi
                if [ -z "$TROUBLESHOOTING" ]; then
                    TROUBLESHOOTING=$(echo "$LIBRARY_CONTENT" | sed -n '/### Debugging/,/^### /p' | head -n -1)
                fi
                if [ -n "$TROUBLESHOOTING" ]; then
                    ALL_LIBRARY_TROUBLESHOOTING="${ALL_LIBRARY_TROUBLESHOOTING}

### $category/$LIBRARY_NAME
$TROUBLESHOOTING"
                fi
                
                # Extract Boundaries section (what is/isn't used)
                BOUNDARIES=$(echo "$LIBRARY_CONTENT" | sed -n '/## Boundaries/,/^## /p' | head -n -1)
                if [ -z "$BOUNDARIES" ]; then
                    BOUNDARIES=$(echo "$LIBRARY_CONTENT" | sed -n '/### Scope/,/^### /p' | head -n -1)
                fi
                if [ -z "$BOUNDARIES" ]; then
                    BOUNDARIES=$(echo "$LIBRARY_CONTENT" | sed -n '/### What We Use/,/^### /p' | head -n -1)
                fi
                if [ -n "$BOUNDARIES" ]; then
                    ALL_LIBRARY_BOUNDARIES="${ALL_LIBRARY_BOUNDARIES}

### $category/$LIBRARY_NAME
$BOUNDARIES"
                fi
            done
        fi
    done
fi
```

### Step 5: Compile and Cache Library Knowledge

Generate the library-basepoints-knowledge.md file:

```bash
echo "📝 Compiling library basepoints knowledge..."

# Create the comprehensive library knowledge document
cat > "$CACHE_PATH/library-basepoints-knowledge.md" << 'LIBRARY_KNOWLEDGE_EOF'
# Extracted Library Basepoints Knowledge

## Extraction Metadata
- **Extracted**: $(date)
- **Source**: agent-os/basepoints/libraries/
- **Library Basepoints Available**: $LIBRARY_BASEPOINTS_AVAILABLE
- **Library Count**: $LIBRARY_COUNT

---

## Library Patterns

Patterns extracted from library basepoints, organized by category:

$ALL_LIBRARY_PATTERNS

---

## Library Workflows

Internal workflows and execution paths from library basepoints:

$ALL_LIBRARY_WORKFLOWS

---

## Best Practices

Official guidelines and best practices from library documentation:

$ALL_LIBRARY_BEST_PRACTICES

---

## Troubleshooting Guide

Common issues, debugging strategies, and known gotchas:

$ALL_LIBRARY_TROUBLESHOOTING

---

## Library Boundaries

What parts of each library are used and not used in this project:

$ALL_LIBRARY_BOUNDARIES

---

*Library knowledge extracted automatically from library basepoints.*
LIBRARY_KNOWLEDGE_EOF

echo "✅ Library knowledge cached to: $CACHE_PATH/library-basepoints-knowledge.md"
```

### Step 6: Generate Library Index

Create an index of all library basepoints for quick reference:

```bash
echo "📋 Generating library index..."

cat > "$CACHE_PATH/library-index.md" << 'INDEX_EOF'
# Library Basepoints Index

## Categories

INDEX_EOF

# Add each category and its libraries
for category in $LIBRARY_CATEGORIES; do
    CATEGORY_PATH="$LIBRARY_BASEPOINTS_PATH/$category"
    
    if [ -d "$CATEGORY_PATH" ]; then
        echo "### $category" >> "$CACHE_PATH/library-index.md"
        echo "" >> "$CACHE_PATH/library-index.md"
        
        find "$CATEGORY_PATH" -name "*.md" -type f | sort | while read library_file; do
            LIBRARY_NAME=$(basename "$library_file" .md)
            RELATIVE_PATH=$(echo "$library_file" | sed "s|$LIBRARY_BASEPOINTS_PATH/||")
            echo "- [$LIBRARY_NAME]($library_file)" >> "$CACHE_PATH/library-index.md"
        done
        
        echo "" >> "$CACHE_PATH/library-index.md"
    fi
done

cat >> "$CACHE_PATH/library-index.md" << 'INDEX_EOF'

---

*Generated: $(date)*
INDEX_EOF

echo "✅ Library index generated"
```

### Step 7: Return Status

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
2. **Empty categories**: Skip empty categories
3. **Missing sections**: Skip missing sections in library basepoints
4. **Empty extraction**: Generate empty knowledge file with metadata

Commands should check `LIBRARY_BASEPOINTS_AVAILABLE` flag and adjust behavior accordingly.

## Important Constraints

- Must traverse the category-based folder structure (data, domain, util, infrastructure, framework)
- Must extract patterns, workflows, best practices, troubleshooting from each library basepoint
- Must organize knowledge by category
- Must preserve source information (which library file each piece of knowledge came from)
- Must cache extracted knowledge to `$SPEC_PATH/implementation/cache/`
- Must provide graceful fallback when library basepoints don't exist
- Must be technology-agnostic and work with any library basepoint structure
- **CRITICAL**: All cached documents must be stored in `$SPEC_PATH/implementation/cache/` when running within a spec command
