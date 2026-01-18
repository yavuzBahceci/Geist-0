# Accumulate Knowledge Across Commands

## Core Responsibilities

1. **Load Previous Knowledge**: Load accumulated knowledge from previous command executions
2. **Merge New Knowledge**: Combine new extracted knowledge with existing accumulated knowledge
3. **Track Knowledge Sources**: Record which commands contributed to the accumulated knowledge
4. **Store Accumulated Knowledge**: Cache the combined knowledge for subsequent commands
5. **Provide Knowledge Summary**: Generate a summary of all accumulated knowledge

## Workflow

### Step 1: Determine Cache Location

Set up the cache path based on spec context:

```bash
# Determine spec path from context
# SPEC_PATH should be set by the calling command
if [ -z "$SPEC_PATH" ]; then
    echo "⚠️ SPEC_PATH not set. Cannot accumulate knowledge."
    return 1
fi

CACHE_PATH="$SPEC_PATH/implementation/cache"
ACCUMULATED_KNOWLEDGE_FILE="$CACHE_PATH/accumulated-knowledge.md"
KNOWLEDGE_SOURCES_FILE="$CACHE_PATH/knowledge-sources.md"

# Create cache directory if it doesn't exist
mkdir -p "$CACHE_PATH"
echo "📁 Cache location: $CACHE_PATH"
```

### Step 2: Load Previous Accumulated Knowledge

Load any existing accumulated knowledge from previous commands:

```bash
# Load previous accumulated knowledge
if [ -f "$ACCUMULATED_KNOWLEDGE_FILE" ]; then
    PREVIOUS_KNOWLEDGE=$(cat "$ACCUMULATED_KNOWLEDGE_FILE")
    echo "✅ Loaded previous accumulated knowledge"
else
    PREVIOUS_KNOWLEDGE=""
    echo "📝 No previous accumulated knowledge found (starting fresh)"
fi

# Load knowledge sources
if [ -f "$KNOWLEDGE_SOURCES_FILE" ]; then
    PREVIOUS_SOURCES=$(cat "$KNOWLEDGE_SOURCES_FILE")
else
    PREVIOUS_SOURCES=""
fi
```

### Step 3: Gather New Knowledge

Gather new knowledge from current command execution:

```bash
# NEW_BASEPOINTS_KNOWLEDGE should be set by the calling command
# NEW_LIBRARY_KNOWLEDGE should be set by the calling command
# NEW_PRODUCT_KNOWLEDGE should be set by the calling command (optional)
# NEW_CODE_KNOWLEDGE should be set by the calling command (optional)
# CURRENT_COMMAND should be set by the calling command

if [ -z "$CURRENT_COMMAND" ]; then
    CURRENT_COMMAND="unknown-command"
fi

echo "📖 Gathering new knowledge from: $CURRENT_COMMAND"

# Combine new knowledge sections
NEW_KNOWLEDGE=""

if [ -n "$NEW_BASEPOINTS_KNOWLEDGE" ]; then
    NEW_KNOWLEDGE="${NEW_KNOWLEDGE}

## Basepoints Knowledge (from $CURRENT_COMMAND)

$NEW_BASEPOINTS_KNOWLEDGE"
fi

if [ -n "$NEW_LIBRARY_KNOWLEDGE" ]; then
    NEW_KNOWLEDGE="${NEW_KNOWLEDGE}

## Library Knowledge (from $CURRENT_COMMAND)

$NEW_LIBRARY_KNOWLEDGE"
fi

if [ -n "$NEW_PRODUCT_KNOWLEDGE" ]; then
    NEW_KNOWLEDGE="${NEW_KNOWLEDGE}

## Product Knowledge (from $CURRENT_COMMAND)

$NEW_PRODUCT_KNOWLEDGE"
fi

if [ -n "$NEW_CODE_KNOWLEDGE" ]; then
    NEW_KNOWLEDGE="${NEW_KNOWLEDGE}

## Code Knowledge (from $CURRENT_COMMAND)

$NEW_CODE_KNOWLEDGE"
fi
```

### Step 4: Merge Knowledge

Combine previous and new knowledge:

```bash
echo "🔄 Merging knowledge..."

# Create merged knowledge document
MERGED_KNOWLEDGE="# Accumulated Knowledge

## Metadata
- **Last Updated**: $(date)
- **Spec Path**: $SPEC_PATH

---

## Previous Knowledge

$PREVIOUS_KNOWLEDGE

---

## New Knowledge (from $CURRENT_COMMAND)

$NEW_KNOWLEDGE

---"
```

### Step 5: Update Knowledge Sources

Track which commands contributed knowledge:

```bash
echo "📋 Updating knowledge sources..."

# Add current command to sources
UPDATED_SOURCES="${PREVIOUS_SOURCES}
- **$CURRENT_COMMAND**: $(date)"

# Create sources document
cat > "$KNOWLEDGE_SOURCES_FILE" << SOURCES_EOF
# Knowledge Sources

## Commands That Contributed Knowledge

$UPDATED_SOURCES

---

## Knowledge Flow

The following commands have contributed to the accumulated knowledge in order:

$UPDATED_SOURCES

---

*Updated: $(date)*
SOURCES_EOF

echo "✅ Knowledge sources updated"
```

### Step 6: Store Accumulated Knowledge

Save the merged knowledge for subsequent commands:

```bash
echo "💾 Storing accumulated knowledge..."

# Write accumulated knowledge
cat > "$ACCUMULATED_KNOWLEDGE_FILE" << ACCUMULATED_EOF
$MERGED_KNOWLEDGE
ACCUMULATED_EOF

echo "✅ Accumulated knowledge stored to: $ACCUMULATED_KNOWLEDGE_FILE"
```

### Step 7: Generate Knowledge Summary

Create a summary of all accumulated knowledge:

```bash
echo "📊 Generating knowledge summary..."

# Count knowledge sections
BASEPOINTS_SECTIONS=$(echo "$MERGED_KNOWLEDGE" | grep -c "## Basepoints Knowledge" || echo "0")
LIBRARY_SECTIONS=$(echo "$MERGED_KNOWLEDGE" | grep -c "## Library Knowledge" || echo "0")
PRODUCT_SECTIONS=$(echo "$MERGED_KNOWLEDGE" | grep -c "## Product Knowledge" || echo "0")
CODE_SECTIONS=$(echo "$MERGED_KNOWLEDGE" | grep -c "## Code Knowledge" || echo "0")

# Create summary
cat > "$CACHE_PATH/knowledge-summary.md" << SUMMARY_EOF
# Knowledge Summary

## Statistics

| Knowledge Type | Sections |
|----------------|----------|
| Basepoints | $BASEPOINTS_SECTIONS |
| Library | $LIBRARY_SECTIONS |
| Product | $PRODUCT_SECTIONS |
| Code | $CODE_SECTIONS |

## Sources

$(cat "$KNOWLEDGE_SOURCES_FILE" 2>/dev/null || echo "No sources recorded")

---

*Generated: $(date)*
SUMMARY_EOF

echo "✅ Knowledge summary generated"
```

### Step 8: Return Status

Provide summary of accumulation:

```bash
echo ""
echo "═══════════════════════════════════════════════════════"
echo "  KNOWLEDGE ACCUMULATION COMPLETE"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Current Command: $CURRENT_COMMAND"
echo "  Accumulated Knowledge: $ACCUMULATED_KNOWLEDGE_FILE"
echo "  Knowledge Sources: $KNOWLEDGE_SOURCES_FILE"
echo "  Knowledge Summary: $CACHE_PATH/knowledge-summary.md"
echo ""
echo "  Knowledge Sections:"
echo "    - Basepoints: $BASEPOINTS_SECTIONS"
echo "    - Library: $LIBRARY_SECTIONS"
echo "    - Product: $PRODUCT_SECTIONS"
echo "    - Code: $CODE_SECTIONS"
echo ""
echo "═══════════════════════════════════════════════════════"

# Export accumulated knowledge for use by calling command
export ACCUMULATED_KNOWLEDGE="$MERGED_KNOWLEDGE"
```

## Usage Notes

This workflow is designed to be called at the end of each spec/implementation command to accumulate knowledge for subsequent commands.

**Typical Usage Pattern:**

```bash
# At the end of a command, after extracting knowledge:
CURRENT_COMMAND="shape-spec"
NEW_BASEPOINTS_KNOWLEDGE="$EXTRACTED_KNOWLEDGE"
NEW_LIBRARY_KNOWLEDGE="$LIBRARY_KNOWLEDGE"
{{workflows/common/accumulate-knowledge}}
```

**Required Variables:**
- `$SPEC_PATH` - Path to the current spec
- `$CURRENT_COMMAND` - Name of the current command

**Optional Variables:**
- `$NEW_BASEPOINTS_KNOWLEDGE` - New basepoints knowledge to add
- `$NEW_LIBRARY_KNOWLEDGE` - New library knowledge to add
- `$NEW_PRODUCT_KNOWLEDGE` - New product knowledge to add
- `$NEW_CODE_KNOWLEDGE` - New code knowledge to add

**Output Variables:**
- `$ACCUMULATED_KNOWLEDGE` - The merged accumulated knowledge

## Important Constraints

- **Spec Path Required**: The `$SPEC_PATH` variable must be set by the calling command.
- **Command Name Required**: The `$CURRENT_COMMAND` variable should be set to track knowledge sources.
- **Idempotent**: Running this workflow multiple times with the same knowledge will add duplicate sections.
- **Order Matters**: Knowledge is accumulated in the order commands are executed.
- **Cache Location**: All accumulated knowledge is stored in `$SPEC_PATH/implementation/cache/`.
