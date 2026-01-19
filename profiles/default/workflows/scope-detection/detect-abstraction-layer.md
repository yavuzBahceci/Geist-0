# Detect Abstraction Layer

## Core Responsibilities

1. **Analyze Feature Description**: Parse feature/spec description for layer indicators
2. **Match Against Documented Layers**: Compare against layers from headquarter.md
3. **Detect Primary Layer**: Determine the primary abstraction layer for the scope
4. **Store Detection Result**: Save detected layer for use by other workflows
5. **Provide Default Behavior**: Default to "unknown" if detection fails

## Workflow

### Step 1: Load Feature/Spec Description

```bash
# SPEC_PATH should be set by the calling command
if [ -z "$SPEC_PATH" ]; then
    echo "⚠️ SPEC_PATH not set. Using FEATURE_DESCRIPTION if available."
fi

echo "🎯 Detecting abstraction layer..."

# Gather text to analyze
ANALYSIS_TEXT=""

# Check feature description first
if [ -n "$FEATURE_DESCRIPTION" ]; then
    ANALYSIS_TEXT="$FEATURE_DESCRIPTION"
fi

# Add spec documents if available
if [ -n "$SPEC_PATH" ]; then
    if [ -f "$SPEC_PATH/planning/initialization.md" ]; then
        ANALYSIS_TEXT="${ANALYSIS_TEXT}
$(cat "$SPEC_PATH/planning/initialization.md")"
    fi
    
    if [ -f "$SPEC_PATH/planning/requirements.md" ]; then
        ANALYSIS_TEXT="${ANALYSIS_TEXT}
$(head -100 "$SPEC_PATH/planning/requirements.md")"
    fi
    
    if [ -f "$SPEC_PATH/spec.md" ]; then
        ANALYSIS_TEXT="${ANALYSIS_TEXT}
$(head -100 "$SPEC_PATH/spec.md")"
    fi
fi

if [ -z "$ANALYSIS_TEXT" ]; then
    echo "⚠️ No text to analyze. Defaulting to unknown."
    DETECTED_LAYER="unknown"
else
    echo "   Text to analyze: $(echo "$ANALYSIS_TEXT" | wc -w | tr -d ' ') words"
fi
```

### Step 2: Load Documented Layers from Headquarter

```bash
BASEPOINTS_PATH="geist/basepoints"
DOCUMENTED_LAYERS=""

if [ -f "$BASEPOINTS_PATH/headquarter.md" ]; then
    echo "📖 Loading layers from headquarter.md..."
    
    # Extract layer names from headquarter
    # Look for patterns like "**ROOT**", "**PROFILES**", etc.
    DOCUMENTED_LAYERS=$(grep -oE '\*\*[A-Z_]+\*\*' "$BASEPOINTS_PATH/headquarter.md" | \
        sed 's/\*//g' | sort -u | tr '\n' ' ')
    
    if [ -z "$DOCUMENTED_LAYERS" ]; then
        # Try alternative patterns
        DOCUMENTED_LAYERS=$(grep -E "^### Layer:" "$BASEPOINTS_PATH/headquarter.md" | \
            sed 's/### Layer: //' | tr '\n' ' ')
    fi
fi

# Default layers if none found
if [ -z "$DOCUMENTED_LAYERS" ]; then
    DOCUMENTED_LAYERS="ROOT PROFILES SCRIPTS"
    echo "   Using default layers: $DOCUMENTED_LAYERS"
else
    echo "   Documented layers: $DOCUMENTED_LAYERS"
fi
```

### Step 3: Define Layer Indicator Keywords

```bash
# Define keywords that indicate each layer
# These are specific to Geist/Geist architecture

# ROOT layer indicators
ROOT_KEYWORDS="readme|manifest|changelog|config|license|documentation|root|base|top-level|project|repository"

# PROFILES layer indicators  
PROFILES_KEYWORDS="profile|command|workflow|agent|standard|template|spec|shape|write|create|implement|orchestrate|validation|basepoint|knowledge|extraction|scope|detection"

# SCRIPTS layer indicators
SCRIPTS_KEYWORDS="script|install|update|shell|bash|function|utility|helper|common|project-install|project-update"

echo "   Layer indicators defined"
```

### Step 4: Score Each Layer

```bash
echo "📊 Scoring layers..."

# Convert text to lowercase for matching
ANALYSIS_LOWER=$(echo "$ANALYSIS_TEXT" | tr '[:upper:]' '[:lower:]')

# Score ROOT layer
ROOT_SCORE=0
for keyword in $(echo "$ROOT_KEYWORDS" | tr '|' ' '); do
    MATCHES=$(echo "$ANALYSIS_LOWER" | grep -oE "$keyword" | wc -l | tr -d ' ')
    ROOT_SCORE=$((ROOT_SCORE + MATCHES))
done

# Score PROFILES layer
PROFILES_SCORE=0
for keyword in $(echo "$PROFILES_KEYWORDS" | tr '|' ' '); do
    MATCHES=$(echo "$ANALYSIS_LOWER" | grep -oE "$keyword" | wc -l | tr -d ' ')
    PROFILES_SCORE=$((PROFILES_SCORE + MATCHES))
done

# Score SCRIPTS layer
SCRIPTS_SCORE=0
for keyword in $(echo "$SCRIPTS_KEYWORDS" | tr '|' ' '); do
    MATCHES=$(echo "$ANALYSIS_LOWER" | grep -oE "$keyword" | wc -l | tr -d ' ')
    SCRIPTS_SCORE=$((SCRIPTS_SCORE + MATCHES))
done

# Add bonus for explicit layer mentions
if echo "$ANALYSIS_TEXT" | grep -qi "ROOT"; then
    ROOT_SCORE=$((ROOT_SCORE + 10))
fi
if echo "$ANALYSIS_TEXT" | grep -qi "PROFILES\|profiles/default"; then
    PROFILES_SCORE=$((PROFILES_SCORE + 10))
fi
if echo "$ANALYSIS_TEXT" | grep -qi "SCRIPTS\|scripts/"; then
    SCRIPTS_SCORE=$((SCRIPTS_SCORE + 10))
fi

echo "   ROOT score: $ROOT_SCORE"
echo "   PROFILES score: $PROFILES_SCORE"  
echo "   SCRIPTS score: $SCRIPTS_SCORE"
```

### Step 5: Determine Primary Layer

```bash
echo "🎯 Determining primary layer..."

# Find highest scoring layer
DETECTED_LAYER="unknown"
HIGHEST_SCORE=0

if [ "$ROOT_SCORE" -gt "$HIGHEST_SCORE" ]; then
    HIGHEST_SCORE=$ROOT_SCORE
    DETECTED_LAYER="ROOT"
fi

if [ "$PROFILES_SCORE" -gt "$HIGHEST_SCORE" ]; then
    HIGHEST_SCORE=$PROFILES_SCORE
    DETECTED_LAYER="PROFILES"
fi

if [ "$SCRIPTS_SCORE" -gt "$HIGHEST_SCORE" ]; then
    HIGHEST_SCORE=$SCRIPTS_SCORE
    DETECTED_LAYER="SCRIPTS"
fi

# If no clear winner (all scores = 0), default behavior
if [ "$HIGHEST_SCORE" -eq 0 ]; then
    echo "   No clear layer detected. Defaulting to 'unknown'."
    DETECTED_LAYER="unknown"
    NEEDS_HUMAN_REVIEW="true"
else
    echo "   Primary layer: $DETECTED_LAYER (score: $HIGHEST_SCORE)"
    NEEDS_HUMAN_REVIEW="false"
fi

# Determine secondary layers (scores > 0 but not highest)
SECONDARY_LAYERS=""
if [ "$ROOT_SCORE" -gt 0 ] && [ "$DETECTED_LAYER" != "ROOT" ]; then
    SECONDARY_LAYERS="${SECONDARY_LAYERS} ROOT"
fi
if [ "$PROFILES_SCORE" -gt 0 ] && [ "$DETECTED_LAYER" != "PROFILES" ]; then
    SECONDARY_LAYERS="${SECONDARY_LAYERS} PROFILES"
fi
if [ "$SCRIPTS_SCORE" -gt 0 ] && [ "$DETECTED_LAYER" != "SCRIPTS" ]; then
    SECONDARY_LAYERS="${SECONDARY_LAYERS} SCRIPTS"
fi

if [ -n "$SECONDARY_LAYERS" ]; then
    echo "   Secondary layers:$SECONDARY_LAYERS"
fi
```

### Step 6: Store Detection Result

```bash
echo "💾 Storing detection result..."

# Determine cache path
if [ -n "$SPEC_PATH" ]; then
    CACHE_PATH="$SPEC_PATH/implementation/cache"
else
    CACHE_PATH="geist/output/layer-detection"
fi

mkdir -p "$CACHE_PATH"

# Store detected layer (simple text file for easy consumption)
echo "$DETECTED_LAYER" > "$CACHE_PATH/detected-layer.txt"

# Store detailed detection report
cat > "$CACHE_PATH/layer-detection-report.md" << LAYER_EOF
# Abstraction Layer Detection Report

**Detected**: $(date)
**Primary Layer**: $DETECTED_LAYER
**Confidence**: $([ "$HIGHEST_SCORE" -gt 5 ] && echo "High" || echo "Low")
**Needs Human Review**: $NEEDS_HUMAN_REVIEW

## Layer Scores

| Layer | Score | Status |
|-------|-------|--------|
| ROOT | $ROOT_SCORE | $([ "$DETECTED_LAYER" = "ROOT" ] && echo "✅ Primary" || echo "-") |
| PROFILES | $PROFILES_SCORE | $([ "$DETECTED_LAYER" = "PROFILES" ] && echo "✅ Primary" || echo "-") |
| SCRIPTS | $SCRIPTS_SCORE | $([ "$DETECTED_LAYER" = "SCRIPTS" ] && echo "✅ Primary" || echo "-") |

## Secondary Layers

$([ -n "$SECONDARY_LAYERS" ] && echo "$SECONDARY_LAYERS" | tr ' ' '\n' | sed 's/^/- /' || echo "None")

## Layer Descriptions

- **ROOT**: Top-level documentation, configuration, and project-wide files
- **PROFILES**: Templates for commands, workflows, agents, and standards
- **SCRIPTS**: Shell scripts for installation, updates, and utilities

## Detection Method

Keywords were extracted from:
- Feature description
- Spec initialization
- Requirements document
- Spec document

Each layer's score is based on keyword frequency matching.

---

*If "Needs Human Review" is true, please verify the detected layer.*
LAYER_EOF

echo "✅ Layer detection stored:"
echo "   - $CACHE_PATH/detected-layer.txt"
echo "   - $CACHE_PATH/layer-detection-report.md"
```

### Step 7: Return Result

```bash
echo ""
echo "═══════════════════════════════════════════════════════"
echo "  ABSTRACTION LAYER DETECTION COMPLETE"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Primary Layer: $DETECTED_LAYER"
echo "  Confidence: $([ "$HIGHEST_SCORE" -gt 5 ] && echo "High ($HIGHEST_SCORE)" || echo "Low ($HIGHEST_SCORE)")"
echo "  Secondary Layers:${SECONDARY_LAYERS:- none}"
echo "  Needs Review: $NEEDS_HUMAN_REVIEW"
echo ""
echo "═══════════════════════════════════════════════════════"

# Export for use by other workflows
export DETECTED_LAYER="$DETECTED_LAYER"
export LAYER_CONFIDENCE="$HIGHEST_SCORE"
export NEEDS_HUMAN_REVIEW="$NEEDS_HUMAN_REVIEW"

# Return the detected layer
echo "$DETECTED_LAYER"
```

## Default Behavior

When detection fails or confidence is low:

1. **Default Layer**: "unknown"
2. **Human Review Flag**: Set to `true`
3. **All Basepoints**: When layer is "unknown", all basepoints are considered relevant
4. **Warning Message**: Inform user that manual layer specification may be needed

## Important Constraints

- Must analyze feature/spec description for layer indicator keywords
- Must match against documented layers from headquarter.md
- Must provide scoring to indicate confidence
- Must default to "unknown" when detection fails
- Must flag for human review when confidence is low
- Must store detection result in `$SPEC_PATH/implementation/cache/detected-layer.txt`
- Must be technology-agnostic but include sensible defaults for Geist projects
- **CRITICAL**: Detection result is used by knowledge extraction to filter relevant basepoints
