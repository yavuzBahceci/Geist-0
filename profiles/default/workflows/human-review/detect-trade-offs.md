# Detect Trade-offs

## Core Responsibilities

1. **Load Extracted Knowledge**: Read cached basepoints knowledge and proposed approach
2. **Compare Against Documented Patterns**: Identify when multiple valid patterns could apply
3. **Identify Conflicting Approaches**: Detect when basepoints suggest different approaches
4. **Check Mission/Roadmap Alignment**: Flag when priorities conflict with technical approach
5. **Output Trade-off List**: Generate list of trade-offs with severity ratings

## Workflow

### Step 1: Load Extracted Knowledge and Proposed Approach

```bash
# SPEC_PATH should be set by the calling command
if [ -z "$SPEC_PATH" ]; then
    echo "⚠️ SPEC_PATH not set. Cannot detect trade-offs."
    exit 1
fi

echo "🔍 Detecting trade-offs..."

CACHE_PATH="$SPEC_PATH/implementation/cache"

# Load extracted basepoints knowledge
KNOWLEDGE_FILE="$CACHE_PATH/basepoints-knowledge.md"
if [ -f "$KNOWLEDGE_FILE" ]; then
    EXTRACTED_KNOWLEDGE=$(cat "$KNOWLEDGE_FILE")
    HAS_KNOWLEDGE="true"
    echo "✅ Loaded basepoints knowledge"
else
    HAS_KNOWLEDGE="false"
    echo "⚠️ No basepoints knowledge found"
fi

# Load proposed approach from spec
PROPOSED_APPROACH=""
if [ -f "$SPEC_PATH/planning/requirements.md" ]; then
    PROPOSED_APPROACH=$(cat "$SPEC_PATH/planning/requirements.md")
elif [ -f "$SPEC_PATH/spec.md" ]; then
    PROPOSED_APPROACH=$(cat "$SPEC_PATH/spec.md")
fi

if [ -z "$PROPOSED_APPROACH" ]; then
    echo "⚠️ No proposed approach found. Using feature description."
    PROPOSED_APPROACH="${FEATURE_DESCRIPTION:-}"
fi
```

### Step 2: Extract Patterns from Knowledge

```bash
echo "📋 Extracting patterns from knowledge..."

PATTERNS_FROM_KNOWLEDGE=""

if [ "$HAS_KNOWLEDGE" = "true" ]; then
    # Extract pattern names and descriptions from knowledge
    PATTERNS_FROM_KNOWLEDGE=$(echo "$EXTRACTED_KNOWLEDGE" | \
        grep -E "^\*\*[A-Za-z]+ Pattern\*\*|^- \*\*[A-Za-z]+\*\*:" | \
        sed 's/\*//g' | head -20)
    
    # Count unique patterns mentioned
    PATTERN_COUNT=$(echo "$PATTERNS_FROM_KNOWLEDGE" | grep -c '.' || echo "0")
    echo "   Found $PATTERN_COUNT patterns in knowledge"
fi
```

### Step 3: Compare Proposed Approach Against Patterns

```bash
echo "⚖️ Comparing proposed approach against patterns..."

TRADE_OFFS=""
TRADE_OFF_COUNT=0

if [ "$HAS_KNOWLEDGE" = "true" ] && [ -n "$PROPOSED_APPROACH" ]; then
    # Convert both to lowercase for comparison
    APPROACH_LOWER=$(echo "$PROPOSED_APPROACH" | tr '[:upper:]' '[:lower:]')
    KNOWLEDGE_LOWER=$(echo "$EXTRACTED_KNOWLEDGE" | tr '[:upper:]' '[:lower:]')
    
    # Check for multiple approaches mentioned in knowledge for same problem
    MULTIPLE_APPROACHES=$(echo "$KNOWLEDGE_LOWER" | \
        grep -oE '(alternatively|or instead|another approach|option [0-9])' | wc -l | tr -d ' ')
    
    if [ "$MULTIPLE_APPROACHES" -gt 0 ]; then
        TRADE_OFFS="${TRADE_OFFS}
TRADE-OFF-1: Multiple Valid Approaches
Severity: warning
Description: Basepoints document multiple valid approaches that could apply
Action: Review alternatives and select most appropriate
"
        TRADE_OFF_COUNT=$((TRADE_OFF_COUNT + 1))
    fi
    
    # Check for "should" vs "must" language conflicts
    MUSTS_IN_KNOWLEDGE=$(echo "$KNOWLEDGE_LOWER" | grep -c 'must\|required\|always' || echo "0")
    SHOULDS_IN_PROPOSAL=$(echo "$APPROACH_LOWER" | grep -c 'should\|might\|could\|optionally' || echo "0")
    
    if [ "$MUSTS_IN_KNOWLEDGE" -gt 0 ] && [ "$SHOULDS_IN_PROPOSAL" -gt 0 ]; then
        TRADE_OFFS="${TRADE_OFFS}
TRADE-OFF-2: Requirement Strength Mismatch
Severity: warning
Description: Knowledge has strict requirements (must) but proposal uses softer language (should)
Action: Verify if requirements can be relaxed or if proposal should be strengthened
"
        TRADE_OFF_COUNT=$((TRADE_OFF_COUNT + 1))
    fi
    
    # Check for complexity trade-offs
    if echo "$APPROACH_LOWER" | grep -q 'simple\|minimal\|basic'; then
        if echo "$KNOWLEDGE_LOWER" | grep -q 'comprehensive\|complete\|full\|robust'; then
            TRADE_OFFS="${TRADE_OFFS}
TRADE-OFF-3: Simplicity vs Completeness
Severity: info
Description: Proposal favors simplicity but knowledge suggests more comprehensive approach
Action: Decide if simplicity or completeness is priority for this feature
"
            TRADE_OFF_COUNT=$((TRADE_OFF_COUNT + 1))
        fi
    fi
    
    # Check for performance vs maintainability
    if echo "$APPROACH_LOWER" | grep -q 'performance\|fast\|efficient\|optimize'; then
        if echo "$KNOWLEDGE_LOWER" | grep -q 'maintainable\|readable\|clean\|simple'; then
            TRADE_OFFS="${TRADE_OFFS}
TRADE-OFF-4: Performance vs Maintainability
Severity: warning
Description: Proposal emphasizes performance but knowledge emphasizes maintainability
Action: Decide which is more important for this specific feature
"
            TRADE_OFF_COUNT=$((TRADE_OFF_COUNT + 1))
        fi
    fi
fi

echo "   Detected $TRADE_OFF_COUNT potential trade-offs"
```

### Step 4: Check Mission and Roadmap Alignment

```bash
echo "🎯 Checking mission/roadmap alignment..."

PRODUCT_PATH="geist/product"

# Load mission if available
if [ -f "$PRODUCT_PATH/mission.md" ]; then
    MISSION=$(cat "$PRODUCT_PATH/mission.md" | tr '[:upper:]' '[:lower:]')
    
    # Check if proposal aligns with mission goals
    MISSION_KEYWORDS=$(echo "$MISSION" | grep -oE '(trust|reliability|human|agent|collaboration|knowledge|abstraction)' | sort -u)
    
    # Count how many mission keywords are in proposal
    ALIGNMENT_SCORE=0
    for keyword in $MISSION_KEYWORDS; do
        if echo "$APPROACH_LOWER" | grep -q "$keyword"; then
            ALIGNMENT_SCORE=$((ALIGNMENT_SCORE + 1))
        fi
    done
    
    MISSION_KEYWORD_COUNT=$(echo "$MISSION_KEYWORDS" | wc -w | tr -d ' ')
    
    if [ "$ALIGNMENT_SCORE" -lt 2 ] && [ "$MISSION_KEYWORD_COUNT" -gt 3 ]; then
        TRADE_OFFS="${TRADE_OFFS}
TRADE-OFF-5: Mission Alignment
Severity: info
Description: Proposal may not strongly align with mission goals
Action: Review if feature supports core mission (alignment score: $ALIGNMENT_SCORE/$MISSION_KEYWORD_COUNT)
"
        TRADE_OFF_COUNT=$((TRADE_OFF_COUNT + 1))
    fi
fi

# Load roadmap if available
if [ -f "$PRODUCT_PATH/roadmap.md" ]; then
    ROADMAP=$(cat "$PRODUCT_PATH/roadmap.md")
    
    # Check if this feature is on roadmap
    FEATURE_NAME=$(basename "$SPEC_PATH")
    if ! echo "$ROADMAP" | grep -qi "$FEATURE_NAME"; then
        TRADE_OFFS="${TRADE_OFFS}
TRADE-OFF-6: Roadmap Priority
Severity: info
Description: This feature may not be explicitly on the current roadmap
Action: Verify feature priority and timing
"
        TRADE_OFF_COUNT=$((TRADE_OFF_COUNT + 1))
    fi
fi
```

### Step 5: Generate Trade-off Output

```bash
echo "📝 Generating trade-off report..."

# Determine if human review is needed
if [ "$TRADE_OFF_COUNT" -gt 0 ]; then
    NEEDS_HUMAN_REVIEW="true"
    CRITICAL_COUNT=$(echo "$TRADE_OFFS" | grep -c "Severity: critical" || echo "0")
    WARNING_COUNT=$(echo "$TRADE_OFFS" | grep -c "Severity: warning" || echo "0")
else
    NEEDS_HUMAN_REVIEW="false"
    CRITICAL_COUNT=0
    WARNING_COUNT=0
fi

# Store trade-offs
mkdir -p "$CACHE_PATH/human-review"

cat > "$CACHE_PATH/human-review/trade-offs.md" << TRADEOFFS_EOF
# Detected Trade-offs

**Analyzed**: $(date)
**Spec Path**: $SPEC_PATH
**Total Trade-offs**: $TRADE_OFF_COUNT
**Critical**: $CRITICAL_COUNT
**Warnings**: $WARNING_COUNT
**Needs Human Review**: $NEEDS_HUMAN_REVIEW

---

## Trade-offs Detected

$TRADE_OFFS

---

## Recommendations

$(if [ "$TRADE_OFF_COUNT" -eq 0 ]; then
    echo "No significant trade-offs detected. Proceed with implementation."
elif [ "$CRITICAL_COUNT" -gt 0 ]; then
    echo "⛔ CRITICAL trade-offs detected. Human review REQUIRED before proceeding."
elif [ "$WARNING_COUNT" -gt 0 ]; then
    echo "⚠️ Warning-level trade-offs detected. Human review RECOMMENDED."
else
    echo "ℹ️ Informational trade-offs detected. Review at your discretion."
fi)

---

*Use /present-human-decision if human review is needed.*
TRADEOFFS_EOF

echo "✅ Trade-offs stored at: $CACHE_PATH/human-review/trade-offs.md"
```

### Step 6: Return Results

```bash
echo ""
echo "═══════════════════════════════════════════════════════"
echo "  TRADE-OFF DETECTION COMPLETE"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Trade-offs Found: $TRADE_OFF_COUNT"
echo "  Critical: $CRITICAL_COUNT"
echo "  Warnings: $WARNING_COUNT"
echo "  Needs Human Review: $NEEDS_HUMAN_REVIEW"
echo ""
echo "═══════════════════════════════════════════════════════"

# Export for use by other workflows
export TRADE_OFF_COUNT="$TRADE_OFF_COUNT"
export NEEDS_HUMAN_REVIEW="$NEEDS_HUMAN_REVIEW"
export TRADE_OFFS="$TRADE_OFFS"
```

## Important Constraints

- Must load extracted knowledge from cache
- Must compare proposed approach against documented patterns
- Must identify when basepoints suggest different approaches
- Must check mission/roadmap alignment
- Must output trade-off list with severity ratings
- Must flag for human review when significant trade-offs detected
- **CRITICAL**: Trade-off results stored in `$SPEC_PATH/implementation/cache/human-review/trade-offs.md`
