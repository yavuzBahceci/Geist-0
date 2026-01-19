# Accumulate Knowledge Across Commands

## Core Responsibilities

1. **Load Previous Knowledge**: Load accumulated knowledge from previous command executions
2. **Prune Irrelevant Knowledge**: Remove knowledge outside the current command's scope with keyword awareness
3. **Deepen Relevant Knowledge**: Add scope-appropriate knowledge based on command depth
4. **Merge New Knowledge**: Combine new extracted knowledge with existing accumulated knowledge
5. **Deduplicate Knowledge**: Detect and remove duplicate content sections
6. **Enforce Context Limits**: Estimate context size and truncate if necessary
7. **Track Knowledge Sources**: Record which commands contributed to the accumulated knowledge
8. **Store Refined Knowledge**: Cache the refined knowledge for subsequent commands
9. **Provide Knowledge Summary**: Generate a summary of all accumulated knowledge with size reporting

## Inputs

- `SPEC_PATH` (required): Path to the current spec
- `CURRENT_COMMAND` (required): Name of the current command
- `KNOWLEDGE_SCOPE` (optional): Scope for pruning (default: "all")
- `KNOWLEDGE_DEPTH` (optional): Depth for new knowledge (default: "libraries")
- `NEW_BASEPOINTS_KNOWLEDGE`, `NEW_LIBRARY_KNOWLEDGE`, `NEW_PRODUCT_KNOWLEDGE`, `NEW_CODE_KNOWLEDGE` (optional)

## Outputs

- `$CACHE_PATH/refined-knowledge.md`: Pruned + deepened knowledge for next command
- `$CACHE_PATH/accumulated-knowledge.md`: Full accumulated knowledge
- `$CACHE_PATH/knowledge-sources.md`: Track of command contributions
- `$CACHE_PATH/knowledge-summary.md`: Summary with size statistics

## Progressive Refinement Strategy

Knowledge flows through commands with progressive refinement:

```
shape-spec    → Product + Architecture (NO code)
     ↓ (pass refined knowledge)
write-spec    → Keep related + File patterns/standards (no code details)
     ↓ (pass refined knowledge)
create-tasks  → Keep related + Code implementation patterns
     ↓ (pass refined knowledge)
orchestrate   → Keep all related + Library knowledge per task group
```

**At Each Step:**
1. **PRUNE**: Remove knowledge irrelevant to current command scope
2. **DEEPEN**: Add deeper knowledge for narrowed relevant scope
3. **PASS**: Hand refined knowledge to next command

## Workflow

### Step 1: Determine Cache Location and Parameters

Set up the cache path and knowledge parameters:

```bash
# Determine spec path from context
# SPEC_PATH should be set by the calling command
if [ -z "$SPEC_PATH" ]; then
    echo "⚠️ SPEC_PATH not set. Cannot accumulate knowledge."
    return 1
fi

CACHE_PATH="$SPEC_PATH/implementation/cache"
ACCUMULATED_KNOWLEDGE_FILE="$CACHE_PATH/accumulated-knowledge.md"
REFINED_KNOWLEDGE_FILE="$CACHE_PATH/refined-knowledge.md"
KNOWLEDGE_SOURCES_FILE="$CACHE_PATH/knowledge-sources.md"

# Create cache directory if it doesn't exist
mkdir -p "$CACHE_PATH"
echo "📁 Cache location: $CACHE_PATH"

# Knowledge refinement parameters (set by calling command)
# KNOWLEDGE_SCOPE: defines what's relevant for current command
#   - "product" = product-related knowledge only
#   - "architecture" = product + architecture patterns
#   - "files" = above + file patterns/standards
#   - "code" = above + code implementation patterns
#   - "libraries" = above + library-specific knowledge
#   - "all" = no filtering (default)
if [ -z "$KNOWLEDGE_SCOPE" ]; then
    KNOWLEDGE_SCOPE="all"
fi

# KNOWLEDGE_DEPTH: determines what new knowledge to extract
#   - "architecture" = product + top-level architecture only
#   - "files" = + file patterns/standards (no code)
#   - "code" = + code implementation patterns
#   - "libraries" = + library knowledge per task group
if [ -z "$KNOWLEDGE_DEPTH" ]; then
    KNOWLEDGE_DEPTH="libraries"
fi

# Context size limit (configurable)
MAX_CONTEXT_LINES="${MAX_CONTEXT_LINES:-3000}"

# Initialize tracking files
HASH_FILE="$CACHE_PATH/.section-hashes.tmp"
TRUNCATION_LOG="$CACHE_PATH/.truncation-log.tmp"
DEDUP_LOG="$CACHE_PATH/.dedup-log.tmp"
PRUNE_LOG="$CACHE_PATH/.prune-log.tmp"

# Clean up previous tracking files
rm -f "$HASH_FILE" "$TRUNCATION_LOG" "$DEDUP_LOG" "$PRUNE_LOG"

echo "🎯 Knowledge Scope: $KNOWLEDGE_SCOPE"
echo "📊 Knowledge Depth: $KNOWLEDGE_DEPTH"
echo "📏 Max Context Lines: $MAX_CONTEXT_LINES"
```

### Step 2: Load Previous Accumulated Knowledge

Load any existing accumulated knowledge from previous commands:

```bash
# Prefer refined knowledge if available (from previous command)
if [ -f "$REFINED_KNOWLEDGE_FILE" ]; then
    PREVIOUS_KNOWLEDGE=$(cat "$REFINED_KNOWLEDGE_FILE")
    echo "✅ Loaded refined knowledge from previous command"
elif [ -f "$ACCUMULATED_KNOWLEDGE_FILE" ]; then
    PREVIOUS_KNOWLEDGE=$(cat "$ACCUMULATED_KNOWLEDGE_FILE")
    echo "✅ Loaded accumulated knowledge"
else
    PREVIOUS_KNOWLEDGE=""
    echo "📝 No previous knowledge found (starting fresh)"
fi

# Load knowledge sources
if [ -f "$KNOWLEDGE_SOURCES_FILE" ]; then
    PREVIOUS_SOURCES=$(cat "$KNOWLEDGE_SOURCES_FILE")
else
    PREVIOUS_SOURCES=""
fi
```

### Step 2.5: Define Context Size and Deduplication Functions

Define helper functions for context management:

```bash
# Function to estimate context size (lines and characters)
estimate_context_size() {
    local knowledge="$1"
    local line_count=$(echo "$knowledge" | wc -l | tr -d ' ')
    local char_count=$(echo "$knowledge" | wc -c | tr -d ' ')
    echo "$line_count|$char_count"
}

# Function to compute content hash (ignoring whitespace variations)
compute_section_hash() {
    local content="$1"
    # Normalize whitespace and compute hash
    echo "$content" | tr -s '[:space:]' ' ' | tr '[:upper:]' '[:lower:]' | md5 2>/dev/null || echo "$content" | tr -s '[:space:]' ' ' | tr '[:upper:]' '[:lower:]' | md5sum | cut -d' ' -f1
}

# Function to check if section is duplicate
is_duplicate_section() {
    local content="$1"
    local hash=$(compute_section_hash "$content")
    
    if grep -q "^$hash$" "$HASH_FILE" 2>/dev/null; then
        return 0  # Is duplicate
    else
        echo "$hash" >> "$HASH_FILE"
        return 1  # Not duplicate
    fi
}

# Function to extract keywords from spec for relevance matching
extract_spec_keywords() {
    local spec_file="$SPEC_PATH/spec.md"
    local keywords=""
    
    if [ -f "$spec_file" ]; then
        # Extract key terms from spec: headers, bold text, code references
        keywords=$(cat "$spec_file" | \
            grep -oE '##+ [A-Za-z ]+|`[^`]+`|\*\*[^*]+\*\*' | \
            sed 's/##\+ //g; s/`//g; s/\*\*//g' | \
            tr '[:upper:]' '[:lower:]' | \
            tr ' ' '\n' | \
            grep -E '^[a-z]{3,}' | \
            sort -u | \
            head -50)
    fi
    
    echo "$keywords"
}

# Function to score section relevance by keyword overlap
score_section_relevance() {
    local section="$1"
    local keywords="$2"
    local score=0
    
    # Normalize section text
    local normalized=$(echo "$section" | tr '[:upper:]' '[:lower:]')
    
    # Count keyword matches
    for keyword in $keywords; do
        if echo "$normalized" | grep -q "$keyword"; then
            score=$((score + 1))
        fi
    done
    
    echo "$score"
}

# Function to truncate knowledge if exceeding limit
truncate_knowledge() {
    local knowledge="$1"
    local max_lines="$2"
    local current_lines=$(echo "$knowledge" | wc -l | tr -d ' ')
    
    if [ "$current_lines" -le "$max_lines" ]; then
        echo "$knowledge"
        return
    fi
    
    echo "⚠️ Context size ($current_lines lines) exceeds limit ($max_lines). Truncating..." >&2
    
    # Truncate from the end, preserving section headers
    local truncated=$(echo "$knowledge" | head -n "$max_lines")
    
    # Log truncation
    local truncated_lines=$((current_lines - max_lines))
    echo "Truncated $truncated_lines lines (from $current_lines to $max_lines)" >> "$TRUNCATION_LOG"
    
    echo "$truncated"
}

echo "✅ Context management functions defined"
```

### Step 3: Prune Irrelevant Knowledge

Remove knowledge outside the current command's scope:

```bash
echo "✂️ Pruning knowledge to scope: $KNOWLEDGE_SCOPE"

# Extract spec keywords for relevance scoring
SPEC_KEYWORDS=$(extract_spec_keywords)
KEYWORD_COUNT=$(echo "$SPEC_KEYWORDS" | wc -w | tr -d ' ')
echo "   Extracted $KEYWORD_COUNT keywords from spec for relevance matching"

# Minimum relevance score to keep a section (sections below this are pruned)
MIN_RELEVANCE_SCORE=2

prune_knowledge_to_scope() {
    local knowledge="$1"
    local scope="$2"
    local keywords="$3"
    local pruned_output=""
    local current_section=""
    local current_header=""
    local sections_kept=0
    local sections_pruned=0
    
    # First apply scope-based pruning
    case "$scope" in
        "product")
            # Keep only product-related sections
            knowledge=$(echo "$knowledge" | grep -A 1000 "## Product" | grep -B 1000 -m 1 "^## [^P]" | head -n -1)
            ;;
        "architecture")
            # Keep product + architecture sections
            knowledge=$(echo "$knowledge" | grep -E "(## Product|## Architecture|## Basepoints.*architecture)" -A 50 | head -200)
            ;;
        "files")
            # Keep product + architecture + file patterns (no code details)
            knowledge=$(echo "$knowledge" | grep -v "## Code Knowledge" | grep -v "implementation patterns")
            ;;
        "code")
            # Keep everything except library-specific details
            knowledge=$(echo "$knowledge" | grep -v "## Library Knowledge")
            ;;
        "libraries"|"all"|*)
            # Keep everything for scope-based pruning
            ;;
    esac
    
    # Now apply keyword-based relevance pruning
    # Only if we have keywords and scope is not "all"
    if [ -n "$keywords" ] && [ "$scope" != "all" ]; then
        # Process line by line, tracking sections
        while IFS= read -r line; do
            # Check if this is a section header
            if echo "$line" | grep -qE "^##+ "; then
                # If we have a previous section, evaluate and potentially add it
                if [ -n "$current_section" ]; then
                    local score=$(score_section_relevance "$current_section" "$keywords")
                    if [ "$score" -ge "$MIN_RELEVANCE_SCORE" ]; then
                        pruned_output="${pruned_output}${current_section}"
                        sections_kept=$((sections_kept + 1))
                    else
                        echo "Pruned section '$current_header' (score: $score < $MIN_RELEVANCE_SCORE)" >> "$PRUNE_LOG"
                        sections_pruned=$((sections_pruned + 1))
                    fi
                fi
                # Start new section
                current_header="$line"
                current_section="$line
"
            else
                # Add line to current section
                current_section="${current_section}${line}
"
            fi
        done <<< "$knowledge"
        
        # Don't forget the last section
        if [ -n "$current_section" ]; then
            local score=$(score_section_relevance "$current_section" "$keywords")
            if [ "$score" -ge "$MIN_RELEVANCE_SCORE" ]; then
                pruned_output="${pruned_output}${current_section}"
                sections_kept=$((sections_kept + 1))
            else
                echo "Pruned section '$current_header' (score: $score < $MIN_RELEVANCE_SCORE)" >> "$PRUNE_LOG"
                sections_pruned=$((sections_pruned + 1))
            fi
        fi
        
        echo "   Keyword pruning: kept $sections_kept sections, pruned $sections_pruned sections" >&2
        echo "$pruned_output"
    else
        # No keyword pruning, return scope-pruned knowledge
        echo "$knowledge"
    fi
}

PRUNED_KNOWLEDGE=$(prune_knowledge_to_scope "$PREVIOUS_KNOWLEDGE" "$KNOWLEDGE_SCOPE" "$SPEC_KEYWORDS")

if [ "$KNOWLEDGE_SCOPE" != "all" ]; then
    echo "✅ Knowledge pruned to scope: $KNOWLEDGE_SCOPE (with keyword awareness)"
else
    echo "ℹ️ No pruning applied (scope: all)"
fi
```

### Step 4: Gather New Knowledge Based on Depth

Gather new knowledge appropriate for the current command's depth:

```bash
# NEW_BASEPOINTS_KNOWLEDGE should be set by the calling command
# NEW_LIBRARY_KNOWLEDGE should be set by the calling command
# NEW_PRODUCT_KNOWLEDGE should be set by the calling command (optional)
# NEW_CODE_KNOWLEDGE should be set by the calling command (optional)
# CURRENT_COMMAND should be set by the calling command

if [ -z "$CURRENT_COMMAND" ]; then
    CURRENT_COMMAND="unknown-command"
fi

echo "📖 Gathering new knowledge from: $CURRENT_COMMAND (depth: $KNOWLEDGE_DEPTH)"

# Filter new knowledge based on depth
NEW_KNOWLEDGE=""

# Product knowledge always included
if [ -n "$NEW_PRODUCT_KNOWLEDGE" ]; then
    NEW_KNOWLEDGE="${NEW_KNOWLEDGE}

## Product Knowledge (from $CURRENT_COMMAND)

$NEW_PRODUCT_KNOWLEDGE"
fi

# Architecture/basepoints knowledge for depth >= architecture
if [ "$KNOWLEDGE_DEPTH" != "product" ] && [ -n "$NEW_BASEPOINTS_KNOWLEDGE" ]; then
    # For shape-spec (architecture depth), only include high-level patterns
    if [ "$KNOWLEDGE_DEPTH" = "architecture" ]; then
        FILTERED_BASEPOINTS=$(echo "$NEW_BASEPOINTS_KNOWLEDGE" | grep -v "implementation" | grep -v "code pattern" | head -100)
        NEW_KNOWLEDGE="${NEW_KNOWLEDGE}

## Architecture Knowledge (from $CURRENT_COMMAND)

$FILTERED_BASEPOINTS"
    else
        NEW_KNOWLEDGE="${NEW_KNOWLEDGE}

## Basepoints Knowledge (from $CURRENT_COMMAND)

$NEW_BASEPOINTS_KNOWLEDGE"
    fi
fi

# Code knowledge for depth >= code
if [ "$KNOWLEDGE_DEPTH" = "code" ] || [ "$KNOWLEDGE_DEPTH" = "libraries" ]; then
    if [ -n "$NEW_CODE_KNOWLEDGE" ]; then
        NEW_KNOWLEDGE="${NEW_KNOWLEDGE}

## Code Knowledge (from $CURRENT_COMMAND)

$NEW_CODE_KNOWLEDGE"
    fi
fi

# Library knowledge only for depth = libraries
if [ "$KNOWLEDGE_DEPTH" = "libraries" ]; then
    if [ -n "$NEW_LIBRARY_KNOWLEDGE" ]; then
        NEW_KNOWLEDGE="${NEW_KNOWLEDGE}

## Library Knowledge (from $CURRENT_COMMAND)

$NEW_LIBRARY_KNOWLEDGE"
    fi
fi

echo "✅ New knowledge gathered for depth: $KNOWLEDGE_DEPTH"
```

### Step 5: Merge and Refine Knowledge (with Deduplication)

Combine pruned previous knowledge with new depth-appropriate knowledge, removing duplicates:

```bash
echo "🔄 Merging and refining knowledge..."

# Initialize deduplication tracking
UNIQUE_SECTIONS=0
DUPLICATE_SECTIONS=0

# Function to deduplicate sections in knowledge
deduplicate_knowledge() {
    local knowledge="$1"
    local deduped_output=""
    local current_section=""
    local current_header=""
    
    # Process line by line, tracking sections
    while IFS= read -r line; do
        if echo "$line" | grep -qE "^##+ "; then
            # If we have a previous section, check for duplicate
            if [ -n "$current_section" ]; then
                if ! is_duplicate_section "$current_section"; then
                    deduped_output="${deduped_output}${current_section}"
                    UNIQUE_SECTIONS=$((UNIQUE_SECTIONS + 1))
                else
                    echo "Removed duplicate section: '$current_header'" >> "$DEDUP_LOG"
                    DUPLICATE_SECTIONS=$((DUPLICATE_SECTIONS + 1))
                fi
            fi
            current_header="$line"
            current_section="$line
"
        else
            current_section="${current_section}${line}
"
        fi
    done <<< "$knowledge"
    
    # Don't forget the last section
    if [ -n "$current_section" ]; then
        if ! is_duplicate_section "$current_section"; then
            deduped_output="${deduped_output}${current_section}"
            UNIQUE_SECTIONS=$((UNIQUE_SECTIONS + 1))
        else
            echo "Removed duplicate section: '$current_header'" >> "$DEDUP_LOG"
            DUPLICATE_SECTIONS=$((DUPLICATE_SECTIONS + 1))
        fi
    fi
    
    echo "$deduped_output"
}

# Combine pruned and new knowledge, then deduplicate
COMBINED_KNOWLEDGE="${PRUNED_KNOWLEDGE}

${NEW_KNOWLEDGE}"

# Apply deduplication
DEDUPED_KNOWLEDGE=$(deduplicate_knowledge "$COMBINED_KNOWLEDGE")

echo "   Deduplication: $UNIQUE_SECTIONS unique sections, $DUPLICATE_SECTIONS duplicates removed"

# Apply context size limit
FINAL_KNOWLEDGE=$(truncate_knowledge "$DEDUPED_KNOWLEDGE" "$MAX_CONTEXT_LINES")

# Report final size
FINAL_SIZE=$(estimate_context_size "$FINAL_KNOWLEDGE")
FINAL_LINES=$(echo "$FINAL_SIZE" | cut -d'|' -f1)
FINAL_CHARS=$(echo "$FINAL_SIZE" | cut -d'|' -f2)
echo "   Final context size: $FINAL_LINES lines, $FINAL_CHARS characters"

# Check if truncation occurred
if [ -f "$TRUNCATION_LOG" ]; then
    TRUNCATION_OCCURRED="true"
    TRUNCATION_INFO=$(cat "$TRUNCATION_LOG")
    echo "   ⚠️ Truncation applied: $TRUNCATION_INFO"
else
    TRUNCATION_OCCURRED="false"
fi

# Create refined knowledge document
REFINED_KNOWLEDGE="# Refined Knowledge for $CURRENT_COMMAND

## Metadata
- **Last Updated**: $(date)
- **Spec Path**: $SPEC_PATH
- **Knowledge Scope**: $KNOWLEDGE_SCOPE
- **Knowledge Depth**: $KNOWLEDGE_DEPTH
- **Command**: $CURRENT_COMMAND
- **Context Size**: $FINAL_LINES lines / $FINAL_CHARS chars
- **Max Context**: $MAX_CONTEXT_LINES lines
- **Unique Sections**: $UNIQUE_SECTIONS
- **Duplicates Removed**: $DUPLICATE_SECTIONS
- **Truncation Applied**: $TRUNCATION_OCCURRED

---

## Refined Knowledge (deduplicated and size-limited)

$FINAL_KNOWLEDGE

---"

# Also create full accumulated knowledge for reference
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

### Step 6: Update Knowledge Sources

Track which commands contributed knowledge:

```bash
echo "📋 Updating knowledge sources..."

# Add current command to sources with depth info
UPDATED_SOURCES="${PREVIOUS_SOURCES}
- **$CURRENT_COMMAND** (scope: $KNOWLEDGE_SCOPE, depth: $KNOWLEDGE_DEPTH): $(date)"

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

## Progressive Refinement Applied

| Command | Scope | Depth |
|---------|-------|-------|
$(echo "$UPDATED_SOURCES" | grep -E "^\- \*\*" | sed 's/- \*\*\([^*]*\)\*\* (scope: \([^,]*\), depth: \([^)]*\)).*/| \1 | \2 | \3 |/')

---

*Updated: $(date)*
SOURCES_EOF

echo "✅ Knowledge sources updated"
```

### Step 7: Store Refined Knowledge

Save both refined and accumulated knowledge:

```bash
echo "💾 Storing knowledge..."

# Write refined knowledge (for next command to use)
cat > "$REFINED_KNOWLEDGE_FILE" << REFINED_EOF
$REFINED_KNOWLEDGE
REFINED_EOF

echo "✅ Refined knowledge stored to: $REFINED_KNOWLEDGE_FILE"

# Write full accumulated knowledge (for reference)
cat > "$ACCUMULATED_KNOWLEDGE_FILE" << ACCUMULATED_EOF
$MERGED_KNOWLEDGE
ACCUMULATED_EOF

echo "✅ Accumulated knowledge stored to: $ACCUMULATED_KNOWLEDGE_FILE"
```

### Step 8: Generate Knowledge Summary

Create a summary of all accumulated knowledge:

```bash
echo "📊 Generating knowledge summary..."

# Count knowledge sections
BASEPOINTS_SECTIONS=$(echo "$MERGED_KNOWLEDGE" | grep -c "## Basepoints Knowledge\|## Architecture Knowledge" || echo "0")
LIBRARY_SECTIONS=$(echo "$MERGED_KNOWLEDGE" | grep -c "## Library Knowledge" || echo "0")
PRODUCT_SECTIONS=$(echo "$MERGED_KNOWLEDGE" | grep -c "## Product Knowledge" || echo "0")
CODE_SECTIONS=$(echo "$MERGED_KNOWLEDGE" | grep -c "## Code Knowledge" || echo "0")

# Create summary with size reporting
cat > "$CACHE_PATH/knowledge-summary.md" << SUMMARY_EOF
# Knowledge Summary

## Current State
- **Command**: $CURRENT_COMMAND
- **Scope**: $KNOWLEDGE_SCOPE
- **Depth**: $KNOWLEDGE_DEPTH

## Context Size

| Metric | Value |
|--------|-------|
| Current Lines | $FINAL_LINES |
| Current Characters | $FINAL_CHARS |
| Max Lines Limit | $MAX_CONTEXT_LINES |
| Truncation Applied | $TRUNCATION_OCCURRED |

## Deduplication

| Metric | Count |
|--------|-------|
| Unique Sections | $UNIQUE_SECTIONS |
| Duplicates Removed | $DUPLICATE_SECTIONS |

## Knowledge Statistics

| Knowledge Type | Sections |
|----------------|----------|
| Product | $PRODUCT_SECTIONS |
| Architecture/Basepoints | $BASEPOINTS_SECTIONS |
| Code | $CODE_SECTIONS |
| Library | $LIBRARY_SECTIONS |

## Progressive Refinement Status

**Refinement applied:** Pruned to scope ($KNOWLEDGE_SCOPE), deepened to depth ($KNOWLEDGE_DEPTH)

**Context optimization:** Deduplicated ($DUPLICATE_SECTIONS removed), size-limited ($FINAL_LINES / $MAX_CONTEXT_LINES lines)

**Next command should use:** \`$REFINED_KNOWLEDGE_FILE\`

## Pruning Log

$(cat "$PRUNE_LOG" 2>/dev/null || echo "No sections pruned by keyword relevance")

## Deduplication Log

$(cat "$DEDUP_LOG" 2>/dev/null || echo "No duplicates found")

## Truncation Log

$(cat "$TRUNCATION_LOG" 2>/dev/null || echo "No truncation needed")

## Sources

$(cat "$KNOWLEDGE_SOURCES_FILE" 2>/dev/null || echo "No sources recorded")

---

*Generated: $(date)*
SUMMARY_EOF

echo "✅ Knowledge summary generated with size reporting"
```

### Step 9: Return Status

Provide summary of accumulation:

```bash
# Clean up temp files
rm -f "$HASH_FILE" "$TRUNCATION_LOG" "$DEDUP_LOG" "$PRUNE_LOG"

echo ""
echo "═══════════════════════════════════════════════════════"
echo "  KNOWLEDGE ACCUMULATION COMPLETE"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Current Command: $CURRENT_COMMAND"
echo "  Knowledge Scope: $KNOWLEDGE_SCOPE"
echo "  Knowledge Depth: $KNOWLEDGE_DEPTH"
echo ""
echo "  Context Size:"
echo "    - Lines: $FINAL_LINES / $MAX_CONTEXT_LINES (limit)"
echo "    - Characters: $FINAL_CHARS"
echo "    - Truncation: $TRUNCATION_OCCURRED"
echo ""
echo "  Deduplication:"
echo "    - Unique Sections: $UNIQUE_SECTIONS"
echo "    - Duplicates Removed: $DUPLICATE_SECTIONS"
echo ""
echo "  Knowledge Sections:"
echo "    - Product: $PRODUCT_SECTIONS"
echo "    - Architecture/Basepoints: $BASEPOINTS_SECTIONS"
echo "    - Code: $CODE_SECTIONS"
echo "    - Library: $LIBRARY_SECTIONS"
echo ""
echo "  Files:"
echo "    - Refined Knowledge: $REFINED_KNOWLEDGE_FILE"
echo "    - Accumulated Knowledge: $ACCUMULATED_KNOWLEDGE_FILE"
echo "    - Knowledge Sources: $KNOWLEDGE_SOURCES_FILE"
echo "    - Knowledge Summary: $CACHE_PATH/knowledge-summary.md"
echo ""
echo "  ➡️ Next command should load: $REFINED_KNOWLEDGE_FILE"
echo ""
echo "═══════════════════════════════════════════════════════"

# Export refined knowledge for use by calling command
export REFINED_KNOWLEDGE="$REFINED_KNOWLEDGE"
export ACCUMULATED_KNOWLEDGE="$MERGED_KNOWLEDGE"
export CONTEXT_SIZE_LINES="$FINAL_LINES"
export CONTEXT_SIZE_CHARS="$FINAL_CHARS"
export TRUNCATION_OCCURRED="$TRUNCATION_OCCURRED"
```

## Usage Notes

This workflow is designed to be called at the end of each spec/implementation command to accumulate and refine knowledge for subsequent commands.

**Typical Usage Pattern:**

```bash
# At the end of a command, after extracting knowledge:
CURRENT_COMMAND="shape-spec"
KNOWLEDGE_SCOPE="architecture"  # What to keep from previous
KNOWLEDGE_DEPTH="architecture"  # What new knowledge to add
NEW_BASEPOINTS_KNOWLEDGE="$EXTRACTED_KNOWLEDGE"
NEW_PRODUCT_KNOWLEDGE="$PRODUCT_KNOWLEDGE"
{{workflows/common/accumulate-knowledge}}
```

**Command-Specific Settings:**

| Command | KNOWLEDGE_SCOPE | KNOWLEDGE_DEPTH | Description |
|---------|-----------------|-----------------|-------------|
| shape-spec | all | architecture | Start fresh, gather product + top-level architecture |
| write-spec | architecture | files | Keep architecture, add file patterns/standards (no code) |
| create-tasks | files | code | Keep files, add code implementation patterns |
| orchestrate-tasks | code | libraries | Keep code, add library knowledge per task group |

**Knowledge Flow Explanation:**

```
shape-spec:     SCOPE=all (start fresh)      DEPTH=architecture (gather high-level)
                           ↓
write-spec:     SCOPE=architecture (prune)   DEPTH=files (add file patterns)
                           ↓
create-tasks:   SCOPE=files (prune)          DEPTH=code (add code patterns)
                           ↓
orchestrate:    SCOPE=code (prune)           DEPTH=libraries (add library knowledge)
```

**Required Variables:**
- `$SPEC_PATH` - Path to the current spec
- `$CURRENT_COMMAND` - Name of the current command

**Optional Variables:**
- `$KNOWLEDGE_SCOPE` - Scope for pruning (default: "all")
- `$KNOWLEDGE_DEPTH` - Depth for new knowledge (default: "libraries")
- `$NEW_BASEPOINTS_KNOWLEDGE` - New basepoints knowledge to add
- `$NEW_LIBRARY_KNOWLEDGE` - New library knowledge to add
- `$NEW_PRODUCT_KNOWLEDGE` - New product knowledge to add
- `$NEW_CODE_KNOWLEDGE` - New code knowledge to add

**Output Variables:**
- `$REFINED_KNOWLEDGE` - The pruned + deepened knowledge for next command
- `$ACCUMULATED_KNOWLEDGE` - The full merged accumulated knowledge
- `$CONTEXT_SIZE_LINES` - Final context size in lines
- `$CONTEXT_SIZE_CHARS` - Final context size in characters
- `$TRUNCATION_OCCURRED` - Whether truncation was applied ("true"/"false")

**Output Files:**
- `refined-knowledge.md` - Pruned + deepened + deduplicated knowledge (next command should load this)
- `accumulated-knowledge.md` - Full accumulated knowledge (for reference)
- `knowledge-sources.md` - Track of which commands contributed
- `knowledge-summary.md` - Summary statistics including size reporting, deduplication, and pruning logs

## Important Constraints

- **Spec Path Required**: The `$SPEC_PATH` variable must be set by the calling command.
- **Command Name Required**: The `$CURRENT_COMMAND` variable should be set to track knowledge sources.
- **Load Refined First**: Next command should load `refined-knowledge.md` instead of `accumulated-knowledge.md`.
- **Scope and Depth Matter**: Set appropriate scope and depth for each command stage.
- **Order Matters**: Knowledge is accumulated and refined in the order commands are executed.
- **Cache Location**: All knowledge files are stored in `$SPEC_PATH/implementation/cache/`.
- **Context Size Limit**: Default 3000 lines. Configure via `MAX_CONTEXT_LINES` variable.
- **Deduplication**: Duplicate sections are automatically detected and removed.
- **Keyword Pruning**: Sections are scored by keyword relevance from spec.md.
- **Truncation**: If context exceeds limit, oldest/least relevant content is truncated.
