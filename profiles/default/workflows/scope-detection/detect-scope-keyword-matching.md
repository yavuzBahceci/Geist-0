# Keyword/Pattern Matching for Scope Detection

## Core Responsibilities

1. **Extract Keywords and Patterns**: Extract keywords and patterns from spec requirements/description
2. **Match Against Basepoint Modules**: Match keywords against basepoint module names and content
3. **Identify Relevant Modules**: Identify relevant modules within abstraction layers
4. **Create Affected Modules List**: Generate list of modules affected by this spec
5. **Store Detection Results**: Cache keyword matching results for use in knowledge extraction

## Workflow

### Step 1: Extract Keywords and Patterns from Spec

Extract keywords and patterns from spec text:

```bash
# Determine spec path from context
if [ -z "$SPEC_PATH" ]; then
    echo "⚠️ SPEC_PATH not set. Cannot perform keyword matching."
    exit 1
fi

echo "📖 Loading spec from: $SPEC_PATH"

# Read spec text from available sources
SPEC_TEXT=""

if [ -f "$SPEC_PATH/planning/requirements.md" ]; then
    SPEC_TEXT=$(cat "$SPEC_PATH/planning/requirements.md")
elif [ -f "$SPEC_PATH/planning/initialization.md" ]; then
    SPEC_TEXT=$(cat "$SPEC_PATH/planning/initialization.md")
elif [ -f "$SPEC_PATH/spec.md" ]; then
    SPEC_TEXT=$(cat "$SPEC_PATH/spec.md")
else
    SPEC_TEXT="${FEATURE_DESCRIPTION:-}"
fi

if [ -z "$SPEC_TEXT" ]; then
    echo "❌ No text to analyze"
    exit 1
fi

echo "🔍 Extracting keywords and patterns..."

# Extract keywords (significant terms, 4+ characters)
KEYWORDS=$(echo "$SPEC_TEXT" | tr '[:upper:]' '[:lower:]' | \
    tr -cs '[:alnum:]' '\n' | \
    grep -E '^[a-z]{4,}$' | \
    grep -vE '^(this|that|with|from|have|will|when|what|which|there|their|these|those|would|should|could|about|after|before|being|between|during|through|where|while|until|because|since|without|within|must|also|than|then|into|other|more|most|very|just|even|make|like|each|some|only)$' | \
    sort | uniq -c | sort -rn | awk '{print $2}')

echo "   Keywords extracted: $(echo "$KEYWORDS" | wc -w | tr -d ' ')"

# Extract module-like names (kebab-case or camelCase patterns)
MODULE_PATTERNS=$(echo "$SPEC_TEXT" | \
    grep -oE '[a-z]+-[a-z]+(-[a-z]+)*|[a-z]+[A-Z][a-zA-Z]*' | \
    tr '[:upper:]' '[:lower:]' | \
    sort | uniq)

echo "   Module patterns: $(echo "$MODULE_PATTERNS" | wc -w | tr -d ' ')"

# Extract file path patterns
FILE_PATTERNS=$(echo "$SPEC_TEXT" | \
    grep -oE '[a-z]+/[a-z-]+(/[a-z-]+)*|[a-z-]+\.md' | \
    sort | uniq)

echo "   File patterns: $(echo "$FILE_PATTERNS" | wc -w | tr -d ' ')"

# Extract layer keywords (specific to Geist)
LAYER_KEYWORDS=$(echo "$SPEC_TEXT" | tr '[:upper:]' '[:lower:]' | \
    grep -oE '(profile|command|workflow|standard|agent|script|template|basepoint)s?' | \
    sort | uniq)

echo "   Layer keywords: $(echo "$LAYER_KEYWORDS" | wc -w | tr -d ' ')"
```

### Step 2: Load Basepoint Module Names and Content

Load basepoint information for matching:

```bash
echo "📁 Loading basepoint modules..."

BASEPOINTS_PATH="geist/basepoints"
BASEPOINT_FILE_PATTERN="agent-base-*.md"

# Check if basepoints exist
if [ ! -d "$BASEPOINTS_PATH" ]; then
    echo "⚠️ Basepoints not found. Using default module list."
    BASEPOINTS_AVAILABLE="false"
else
    BASEPOINTS_AVAILABLE="true"
fi

# Build module info list
MODULE_INFO=""

if [ "$BASEPOINTS_AVAILABLE" = "true" ]; then
    # Process each basepoint file
    find "$BASEPOINTS_PATH" -name "$BASEPOINT_FILE_PATTERN" -type f | while read basepoint_file; do
        if [ -z "$basepoint_file" ]; then
            continue
        fi
        
        # Extract module name from filename
        MODULE_NAME=$(basename "$basepoint_file" .md | sed 's/agent-base-//')
        
        # Extract path relative to basepoints
        MODULE_PATH=$(dirname "$basepoint_file" | sed "s|$BASEPOINTS_PATH/||")
        
        # Determine layer from path
        if echo "$MODULE_PATH" | grep -qi "profile"; then
            MODULE_LAYER="PROFILES"
        elif echo "$MODULE_PATH" | grep -qi "script"; then
            MODULE_LAYER="SCRIPTS"
        else
            MODULE_LAYER="ROOT"
        fi
        
        # Extract keywords from basepoint content (first 500 lines)
        MODULE_KEYWORDS=$(head -500 "$basepoint_file" | tr '[:upper:]' '[:lower:]' | \
            tr -cs '[:alnum:]' '\n' | \
            grep -E '^[a-z]{4,}$' | sort | uniq | head -30 | tr '\n' ',')
        
        echo "$MODULE_NAME|$MODULE_LAYER|$MODULE_PATH|$basepoint_file|$MODULE_KEYWORDS"
    done > /tmp/module_info_$$
    
    MODULE_INFO=$(cat /tmp/module_info_$$ 2>/dev/null)
    rm -f /tmp/module_info_$$
fi

MODULE_COUNT=$(echo "$MODULE_INFO" | grep -c '|' || echo "0")
echo "   Modules loaded: $MODULE_COUNT"
```

### Step 3: Match Keywords Against Basepoint Modules

Match extracted keywords against basepoint modules:

```bash
echo "🔗 Matching keywords to modules..."

MATCHED_MODULES=""
MATCHED_BASEPOINTS=""
MATCH_SCORES=""

if [ -n "$MODULE_INFO" ]; then
    echo "$MODULE_INFO" | while IFS='|' read -r module_name module_layer module_path basepoint_file module_keywords; do
        if [ -z "$module_name" ]; then
            continue
        fi
        
        SCORE=0
        
        # Check if keywords match module name
        for keyword in $KEYWORDS; do
            if echo "$module_name" | grep -qi "$keyword"; then
                SCORE=$((SCORE + 3))
            fi
        done
        
        # Check if module patterns match
        for pattern in $MODULE_PATTERNS; do
            if echo "$module_name" | grep -qi "$pattern"; then
                SCORE=$((SCORE + 2))
            fi
            if echo "$module_path" | grep -qi "$pattern"; then
                SCORE=$((SCORE + 2))
            fi
        done
        
        # Check if keywords match module content keywords
        for keyword in $KEYWORDS; do
            if echo "$module_keywords" | grep -qi "$keyword"; then
                SCORE=$((SCORE + 1))
            fi
        done
        
        # Check layer keyword matches
        for layer_kw in $LAYER_KEYWORDS; do
            if echo "$module_layer" | grep -qi "$layer_kw"; then
                SCORE=$((SCORE + 2))
            fi
            if echo "$module_path" | grep -qi "$layer_kw"; then
                SCORE=$((SCORE + 1))
            fi
        done
        
        # If score > 0, this module is relevant
        if [ "$SCORE" -gt 0 ]; then
            echo "$module_name:$SCORE:$basepoint_file"
        fi
    done > /tmp/match_scores_$$
    
    MATCH_SCORES=$(cat /tmp/match_scores_$$ 2>/dev/null | sort -t: -k2 -rn)
    rm -f /tmp/match_scores_$$
fi

# Also always check headquarter.md
if [ -f "$BASEPOINTS_PATH/headquarter.md" ]; then
    HQ_CONTENT=$(cat "$BASEPOINTS_PATH/headquarter.md" | tr '[:upper:]' '[:lower:]')
    HQ_SCORE=0
    
    for keyword in $KEYWORDS; do
        if echo "$HQ_CONTENT" | grep -q "$keyword"; then
            HQ_SCORE=$((HQ_SCORE + 1))
        fi
    done
    
    if [ "$HQ_SCORE" -gt 0 ]; then
        MATCH_SCORES="headquarter:$HQ_SCORE:$BASEPOINTS_PATH/headquarter.md
$MATCH_SCORES"
    fi
fi

echo "   Matched modules: $(echo "$MATCH_SCORES" | grep -c ':' || echo "0")"
```

### Step 4: Generate Affected Modules List

Create list of modules affected by this spec:

```bash
echo "📋 Generating affected modules list..."

# Extract top matched modules (score > 2)
AFFECTED_MODULES=$(echo "$MATCH_SCORES" | awk -F: '$2 > 2 {print $1}' | head -10)

# Extract corresponding basepoint files
AFFECTED_BASEPOINTS=$(echo "$MATCH_SCORES" | awk -F: '$2 > 2 {print $3}' | head -10)

echo "   High-relevance modules:"
echo "$MATCH_SCORES" | head -5 | while IFS=: read -r mod score bp; do
    echo "     - $mod (score: $score)"
done
```

### Step 5: Store Detection Results

Cache keyword matching results:

```bash
echo "💾 Storing detection results..."

# Determine cache path
CACHE_PATH="$SPEC_PATH/implementation/cache"
mkdir -p "$CACHE_PATH/scope-detection"

# Store keyword matching results as markdown
cat > "$CACHE_PATH/scope-detection/keyword-matching.md" << KEYWORD_EOF
# Keyword Matching Scope Detection Results

**Analyzed**: $(date)
**Spec Path**: $SPEC_PATH

## Extracted Keywords

### Top Keywords
$(echo "$KEYWORDS" | head -20 | sed 's/^/- /')

### Module Patterns
$(echo "$MODULE_PATTERNS" | sed 's/^/- /')

### File Patterns
$(echo "$FILE_PATTERNS" | sed 's/^/- /')

### Layer Keywords
$(echo "$LAYER_KEYWORDS" | sed 's/^/- /')

## Module Match Scores

| Module | Score | Basepoint |
|--------|-------|-----------|
$(echo "$MATCH_SCORES" | head -15 | while IFS=: read -r mod score bp; do
    echo "| $mod | $score | $bp |"
done)

## Affected Modules (High Relevance)

$(echo "$AFFECTED_MODULES" | sed 's/^/- /')

## Affected Basepoints

$(echo "$AFFECTED_BASEPOINTS" | sed 's/^/- /')

---

*Use this matching to guide targeted knowledge extraction.*
KEYWORD_EOF

# Also create simple affected modules file
echo "$AFFECTED_MODULES" > "$CACHE_PATH/scope-detection/affected-modules.txt"

echo "✅ Keyword matching stored at: $CACHE_PATH/scope-detection/keyword-matching.md"
```

### Step 6: Return Results

```bash
echo ""
echo "═══════════════════════════════════════════════════════"
echo "  KEYWORD MATCHING SCOPE DETECTION COMPLETE"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Keywords Extracted: $(echo "$KEYWORDS" | wc -w | tr -d ' ')"
echo "  Modules Matched: $(echo "$MATCH_SCORES" | grep -c ':' || echo "0")"
echo "  High-Relevance Modules: $(echo "$AFFECTED_MODULES" | grep -c '.' || echo "0")"
echo ""
echo "  Results at: $CACHE_PATH/scope-detection/keyword-matching.md"
echo ""
echo "═══════════════════════════════════════════════════════"

# Export for use by other workflows
export AFFECTED_MODULES="$AFFECTED_MODULES"
export AFFECTED_BASEPOINTS="$AFFECTED_BASEPOINTS"
```

## Important Constraints

- Must extract keywords and patterns from spec requirements/description
- Must match keywords against basepoint module names and content
- Must identify relevant modules within abstraction layers using scoring
- Must create mapping from spec context to relevant basepoints
- Must provide graceful fallback when basepoints don't exist
- Must be technology-agnostic and work with any basepoint structure
- **CRITICAL**: All detection results must be stored in `$SPEC_PATH/implementation/cache/scope-detection/`
