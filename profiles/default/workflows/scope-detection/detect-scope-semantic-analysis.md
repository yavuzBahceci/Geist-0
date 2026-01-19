# Semantic Analysis for Scope Detection

## Core Responsibilities

1. **Analyze Spec Requirements/Description**: Extract semantic meaning from spec text
2. **Extract Key Concepts**: Identify entities, relationships, and domain concepts
3. **Map to Abstraction Layers**: Infer relevant abstraction layers from semantic concepts
4. **Identify Relevant Basepoints**: Map semantic concepts to basepoint files
5. **Store Detection Results**: Cache scope detection results for use in knowledge extraction

## Workflow

### Step 1: Load Spec Requirements/Description

Read the spec requirements or description to analyze:

```bash
# Determine spec path from context
# SPEC_PATH should be set by the calling command
if [ -z "$SPEC_PATH" ]; then
    echo "⚠️ SPEC_PATH not set. Cannot perform semantic analysis."
    exit 1
fi

echo "📖 Loading spec from: $SPEC_PATH"

# Read spec requirements or description (try multiple sources)
SPEC_TEXT=""

if [ -f "$SPEC_PATH/planning/requirements.md" ]; then
    SPEC_TEXT=$(cat "$SPEC_PATH/planning/requirements.md")
    echo "✅ Loaded requirements.md"
elif [ -f "$SPEC_PATH/planning/initialization.md" ]; then
    SPEC_TEXT=$(cat "$SPEC_PATH/planning/initialization.md")
    echo "✅ Loaded initialization.md"
elif [ -f "$SPEC_PATH/spec.md" ]; then
    SPEC_TEXT=$(cat "$SPEC_PATH/spec.md")
    echo "✅ Loaded spec.md"
else
    echo "⚠️ No spec documents found. Using feature description if provided."
    SPEC_TEXT="${FEATURE_DESCRIPTION:-}"
fi

if [ -z "$SPEC_TEXT" ]; then
    echo "❌ No text to analyze"
    exit 1
fi
```

### Step 2: Extract Key Concepts, Entities, and Relationships

Analyze the spec text to extract semantic concepts:

```bash
echo "🔍 Extracting semantic concepts..."

# Define layer indicator keywords for this project type (Geist/Geist)
# These map concepts to abstraction layers
declare -A LAYER_INDICATORS
LAYER_INDICATORS["ROOT"]="readme|manifest|changelog|config|documentation|license"
LAYER_INDICATORS["PROFILES"]="command|workflow|agent|standard|template|profile"
LAYER_INDICATORS["SCRIPTS"]="script|install|update|shell|bash|function"

# Extract entities (nouns that represent modules/components)
# Convert to lowercase and extract significant words
ENTITIES=$(echo "$SPEC_TEXT" | tr '[:upper:]' '[:lower:]' | \
    grep -oE '\b[a-z]{4,}\b' | \
    grep -vE '^(this|that|with|from|have|will|when|what|which|there|their|these|those|would|should|could|about|after|before|being|between|during|through|where|while|until|because|since|without|within|above|below|under|over|such|only|some|also|than|then|into|other|more|most|very|just|even|back|down|well|much|same|still|each|both|many|make|made|like|take|find|give|tell|know|want|look|work|seem|come|good|long|great|little|own|old|right|high|different|small|large|next|early|young|important|public|able|last|sure|better|best|certain|likely|true|full|whole|free|clear|real|less|main|late|hard|past|present|possible|local|major|must|else|here|away|always|never|often|already|together|however|another|maybe|sometimes|instead|along|nothing|everything|everyone|anything|anyone|actually|something|someone|probably|especially|although|whether|either|neither|among|against|several|including|regarding|according|following)$' | \
    sort | uniq -c | sort -rn | head -20 | awk '{print $2}')

echo "   Entities found: $(echo "$ENTITIES" | wc -w | tr -d ' ')"

# Extract technical concepts (patterns, standards, workflows, etc.)
TECHNICAL_CONCEPTS=$(echo "$SPEC_TEXT" | tr '[:upper:]' '[:lower:]' | \
    grep -oE '(basepoint|knowledge|extraction|validation|pattern|standard|workflow|command|spec|implementation|cache|layer|abstraction|module|scope|detection)s?' | \
    sort | uniq)

echo "   Technical concepts: $(echo "$TECHNICAL_CONCEPTS" | wc -w | tr -d ' ')"

# Extract architectural concepts (layers, components, relationships)
ARCHITECTURAL_CONCEPTS=$(echo "$SPEC_TEXT" | tr '[:upper:]' '[:lower:]' | \
    grep -oE '(layer|component|module|service|interface|architecture|structure|hierarchy|pattern|design|integration|dependency)' | \
    sort | uniq)

echo "   Architectural concepts: $(echo "$ARCHITECTURAL_CONCEPTS" | wc -w | tr -d ' ')"
```

### Step 3: Map Semantic Concepts to Abstraction Layers

Map extracted concepts to abstraction layers:

```bash
echo "🎯 Mapping concepts to abstraction layers..."

# Load abstraction layers from headquarter.md if available
BASEPOINTS_PATH="geist/basepoints"
DETECTED_LAYERS=""

if [ -f "$BASEPOINTS_PATH/headquarter.md" ]; then
    # Extract layers from headquarter
    DETECTED_LAYERS=$(grep -E "^\| \*\*[A-Z]+\*\*|^- \*\*[A-Z]+\*\*" "$BASEPOINTS_PATH/headquarter.md" | \
        sed 's/.*\*\*\([A-Z_]*\)\*\*.*/\1/' | sort -u)
fi

if [ -z "$DETECTED_LAYERS" ]; then
    # Default layers for Geist-type projects
    DETECTED_LAYERS="ROOT
PROFILES
SCRIPTS"
fi

echo "   Known layers: $(echo "$DETECTED_LAYERS" | tr '\n' ', ' | sed 's/,$//')"

# Score each layer based on concept matches
LAYER_SCORES=""
PRIMARY_LAYER=""
HIGHEST_SCORE=0

for layer in $DETECTED_LAYERS; do
    SCORE=0
    LAYER_LOWER=$(echo "$layer" | tr '[:upper:]' '[:lower:]')
    
    # Get indicator pattern for this layer
    INDICATOR_PATTERN="${LAYER_INDICATORS[$layer]:-$LAYER_LOWER}"
    
    # Count matches in spec text
    MATCHES=$(echo "$SPEC_TEXT" | tr '[:upper:]' '[:lower:]' | grep -oE "$INDICATOR_PATTERN" | wc -l | tr -d ' ')
    SCORE=$((SCORE + MATCHES))
    
    # Check if layer is explicitly mentioned
    if echo "$SPEC_TEXT" | grep -qi "$layer"; then
        SCORE=$((SCORE + 5))
    fi
    
    LAYER_SCORES="${LAYER_SCORES}$layer:$SCORE
"
    
    if [ "$SCORE" -gt "$HIGHEST_SCORE" ]; then
        HIGHEST_SCORE=$SCORE
        PRIMARY_LAYER=$layer
    fi
done

# Default to PROFILES if no clear winner
if [ -z "$PRIMARY_LAYER" ] || [ "$HIGHEST_SCORE" -eq 0 ]; then
    PRIMARY_LAYER="PROFILES"
fi

echo "   Primary layer detected: $PRIMARY_LAYER (score: $HIGHEST_SCORE)"
```

### Step 4: Identify Relevant Basepoint Files

Map semantic concepts to relevant basepoint files:

```bash
echo "📁 Identifying relevant basepoints..."

RELEVANT_BASEPOINTS=""
BASEPOINT_FILE_PATTERN="agent-base-*.md"

# Always include headquarter.md for cross-layer context
if [ -f "$BASEPOINTS_PATH/headquarter.md" ]; then
    RELEVANT_BASEPOINTS="$BASEPOINTS_PATH/headquarter.md"
    echo "   + headquarter.md (cross-layer context)"
fi

# Find basepoints matching detected layer
if [ -d "$BASEPOINTS_PATH" ]; then
    PRIMARY_LOWER=$(echo "$PRIMARY_LAYER" | tr '[:upper:]' '[:lower:]')
    
    # Look for basepoints in the layer's directory
    LAYER_BASEPOINTS=$(find "$BASEPOINTS_PATH" -path "*/$PRIMARY_LOWER/*" -name "$BASEPOINT_FILE_PATTERN" -type f 2>/dev/null)
    
    if [ -n "$LAYER_BASEPOINTS" ]; then
        echo "$LAYER_BASEPOINTS" | while read bp; do
            echo "   + $(basename "$bp") (primary layer match)"
        done
        RELEVANT_BASEPOINTS="${RELEVANT_BASEPOINTS}
$LAYER_BASEPOINTS"
    fi
    
    # Also search by entity/concept name matches
    for entity in $ENTITIES; do
        MATCHED=$(find "$BASEPOINTS_PATH" -name "*$entity*" -type f 2>/dev/null | head -3)
        if [ -n "$MATCHED" ]; then
            RELEVANT_BASEPOINTS="${RELEVANT_BASEPOINTS}
$MATCHED"
        fi
    done
fi

# Remove duplicates
RELEVANT_BASEPOINTS=$(echo "$RELEVANT_BASEPOINTS" | grep -v '^$' | sort -u)
BASEPOINT_COUNT=$(echo "$RELEVANT_BASEPOINTS" | grep -c '.' || echo "0")
echo "   Total relevant basepoints: $BASEPOINT_COUNT"
```

### Step 5: Store Detection Results

Cache scope detection results:

```bash
echo "💾 Storing detection results..."

# Determine cache path
CACHE_PATH="$SPEC_PATH/implementation/cache"
mkdir -p "$CACHE_PATH/scope-detection"

# Store primary detected layer
echo "$PRIMARY_LAYER" > "$CACHE_PATH/detected-layer.txt"

# Store semantic analysis results as markdown (human-readable)
cat > "$CACHE_PATH/scope-detection/semantic-analysis.md" << SEMANTIC_EOF
# Semantic Analysis Scope Detection Results

**Analyzed**: $(date)
**Spec Path**: $SPEC_PATH

## Primary Abstraction Layer

**Detected**: $PRIMARY_LAYER

### Layer Scores
$(echo "$LAYER_SCORES" | grep -v '^$' | sed 's/^/- /')

## Extracted Concepts

### Entities (Top 20)
$(echo "$ENTITIES" | head -20 | sed 's/^/- /')

### Technical Concepts
$(echo "$TECHNICAL_CONCEPTS" | sed 's/^/- /')

### Architectural Concepts
$(echo "$ARCHITECTURAL_CONCEPTS" | sed 's/^/- /')

## Relevant Basepoints Identified

$(echo "$RELEVANT_BASEPOINTS" | sed 's/^/- /')

---

*Use this analysis to guide knowledge extraction and scope filtering.*
SEMANTIC_EOF

echo "✅ Semantic analysis stored at: $CACHE_PATH/scope-detection/semantic-analysis.md"
```

### Step 6: Return Results

```bash
echo ""
echo "═══════════════════════════════════════════════════════"
echo "  SEMANTIC SCOPE DETECTION COMPLETE"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Primary Layer: $PRIMARY_LAYER"
echo "  Entities Found: $(echo "$ENTITIES" | wc -w | tr -d ' ')"
echo "  Relevant Basepoints: $BASEPOINT_COUNT"
echo ""
echo "  Results at: $CACHE_PATH/scope-detection/semantic-analysis.md"
echo ""
echo "═══════════════════════════════════════════════════════"

# Export for use by other workflows
export DETECTED_LAYER="$PRIMARY_LAYER"
export RELEVANT_BASEPOINTS="$RELEVANT_BASEPOINTS"
```

## Important Constraints

- Must analyze spec requirements/description semantically to infer relevant modules and abstraction layers
- Must extract key concepts, entities, and relationships from spec text
- Must map semantic concepts to abstraction layers (ROOT, PROFILES, SCRIPTS for Geist)
- Must identify relevant basepoint files based on semantic analysis
- Must provide graceful fallback when basepoints don't exist
- Must be technology-agnostic but include sensible defaults for Geist projects
- **CRITICAL**: All detection results must be stored in `$SPEC_PATH/implementation/cache/scope-detection/`
