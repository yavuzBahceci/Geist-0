# Present Human Decision

---

## ⚠️ CRITICAL: USER INTERACTION REQUIRED

**This workflow presents decisions to the user and requires their response.** You MUST:
1. Display all detected issues clearly
2. Present the AI recommendation
3. Show all decision options
4. **STOP and WAIT** for the user to make a decision
5. Do NOT proceed until the user provides their decision
6. Do NOT assume the user accepts the recommendation

---

## Core Responsibilities

1. **Load Trade-offs and Contradictions**: Read detected issues from cache
2. **Format Trade-off Presentation**: Create human-readable decision format
3. **Present Options with Pros/Cons**: Include information from basepoints
4. **Provide AI Recommendation**: Suggest best option with reasoning
5. **Log Human Decision**: Record decision for future reference

## Workflow

### Step 1: Load Detected Issues

```bash
# SPEC_PATH should be set by the calling command
if [ -z "$SPEC_PATH" ]; then
    echo "⚠️ SPEC_PATH not set. Cannot present decision."
    exit 1
fi

echo "📋 Loading detected issues..."

CACHE_PATH="$SPEC_PATH/implementation/cache"
REVIEW_PATH="$CACHE_PATH/human-review"

# Load trade-offs
TRADE_OFFS=""
if [ -f "$REVIEW_PATH/trade-offs.md" ]; then
    TRADE_OFFS=$(cat "$REVIEW_PATH/trade-offs.md")
    echo "✅ Loaded trade-offs"
fi

# Load contradictions
CONTRADICTIONS=""
if [ -f "$REVIEW_PATH/contradictions.md" ]; then
    CONTRADICTIONS=$(cat "$REVIEW_PATH/contradictions.md")
    echo "✅ Loaded contradictions"
fi

# Check if there's anything to present
if [ -z "$TRADE_OFFS" ] && [ -z "$CONTRADICTIONS" ]; then
    echo "ℹ️ No issues to present for human review"
    exit 0
fi
```

### Step 2: Load Basepoints Knowledge for Context

```bash
echo "📖 Loading context from basepoints..."

KNOWLEDGE=""
if [ -f "$CACHE_PATH/basepoints-knowledge.md" ]; then
    KNOWLEDGE=$(cat "$CACHE_PATH/basepoints-knowledge.md")
fi

# Extract relevant patterns and their pros/cons
RELEVANT_PATTERNS=$(echo "$KNOWLEDGE" | sed -n '/## Patterns/,/^## [^P]/p' | head -50)
```

### Step 3: Format Trade-off Presentation

```bash
echo "📝 Formatting decision presentation..."

# Create the presentation document
mkdir -p "$REVIEW_PATH"

cat > "$REVIEW_PATH/human-decision-request.md" << 'DECISION_HEADER'
# 🔍 Human Decision Required

A review checkpoint has been triggered. Please review the following trade-offs and/or contradictions and provide your decision.

---

DECISION_HEADER

# Add trade-offs section if present
if [ -n "$TRADE_OFFS" ]; then
    TRADE_OFF_COUNT=$(grep -c "TRADE-OFF-" "$REVIEW_PATH/trade-offs.md" 2>/dev/null || echo "0")
    
    cat >> "$REVIEW_PATH/human-decision-request.md" << TRADEOFFS_SECTION
## ⚖️ Trade-offs Detected ($TRADE_OFF_COUNT)

The following trade-offs were identified during analysis:

TRADEOFFS_SECTION

    # Extract and format each trade-off
    grep -A 3 "TRADE-OFF-" "$REVIEW_PATH/trade-offs.md" 2>/dev/null | \
        sed 's/^TRADE-OFF-/\n### Trade-off /' | \
        sed 's/Severity: /**Severity**: /' | \
        sed 's/Description: /\n**Description**: /' | \
        sed 's/Action: /\n**Recommended Action**: /' \
        >> "$REVIEW_PATH/human-decision-request.md"
fi

# Add contradictions section if present
if [ -n "$CONTRADICTIONS" ]; then
    CRITICAL_COUNT=$(grep -c "⛔" "$REVIEW_PATH/contradictions.md" 2>/dev/null || echo "0")
    WARNING_COUNT=$(grep -c "⚠️" "$REVIEW_PATH/contradictions.md" 2>/dev/null || echo "0")
    
    cat >> "$REVIEW_PATH/human-decision-request.md" << CONTRADICTIONS_SECTION

---

## ⚠️ Contradictions Detected (Critical: $CRITICAL_COUNT, Warnings: $WARNING_COUNT)

The following contradictions with documented standards were identified:

CONTRADICTIONS_SECTION

    # Extract critical contradictions
    if [ "$CRITICAL_COUNT" -gt 0 ]; then
        echo "### Critical (Must Resolve)" >> "$REVIEW_PATH/human-decision-request.md"
        grep "⛔" "$REVIEW_PATH/contradictions.md" >> "$REVIEW_PATH/human-decision-request.md"
    fi
    
    # Extract warning contradictions
    if [ "$WARNING_COUNT" -gt 0 ]; then
        echo "" >> "$REVIEW_PATH/human-decision-request.md"
        echo "### Warnings (Should Review)" >> "$REVIEW_PATH/human-decision-request.md"
        grep "⚠️" "$REVIEW_PATH/contradictions.md" >> "$REVIEW_PATH/human-decision-request.md"
    fi
fi
```

### Step 4: Add Options and AI Recommendation

```bash
# Add decision options
cat >> "$REVIEW_PATH/human-decision-request.md" << 'OPTIONS_SECTION'

---

## 🤖 AI Recommendation

Based on the analysis:

OPTIONS_SECTION

# Generate recommendation based on severity
if grep -q "Critical:" "$REVIEW_PATH/contradictions.md" 2>/dev/null; then
    CRIT_COUNT=$(grep "Critical:" "$REVIEW_PATH/contradictions.md" | grep -oE '[0-9]+' | head -1)
    if [ "$CRIT_COUNT" -gt 0 ]; then
        cat >> "$REVIEW_PATH/human-decision-request.md" << 'CRITICAL_REC'
**⛔ RECOMMENDATION: STOP AND RESOLVE**

Critical contradictions were detected. I recommend:
1. Review each critical contradiction carefully
2. Either modify the proposal to comply with standards, OR
3. Request a formal exception to the standard with justification

Do not proceed until critical contradictions are resolved.

CRITICAL_REC
    fi
else
    cat >> "$REVIEW_PATH/human-decision-request.md" << 'PROCEED_REC'
**✅ RECOMMENDATION: PROCEED WITH AWARENESS**

No critical issues were detected. I recommend proceeding with the following considerations:
- Review the warning-level trade-offs
- Document any intentional deviations from patterns
- Consider the trade-offs when making implementation decisions

PROCEED_REC
fi

# Add decision input section
cat >> "$REVIEW_PATH/human-decision-request.md" << 'DECISION_INPUT'

---

## 👉 Your Decision

Please respond with one of the following:

### For Trade-offs:
1. **Accept recommendation** - Proceed with the AI-recommended approach
2. **Choose alternative** - Specify which alternative approach you prefer
3. **Custom resolution** - Describe your preferred resolution

### For Contradictions:
1. **Will fix** - I will modify the proposal to comply
2. **Request exception** - I request an exception (provide justification)
3. **Not applicable** - This contradiction doesn't apply (explain why)

### General:
- **Proceed** - Continue with implementation as-is
- **Stop** - Halt and reconsider the approach
- **Modify** - Make specific changes (describe them)

---

*Please provide your decision below:*

DECISION_INPUT

echo "✅ Decision request created"
```

### Step 5: Display to User

```bash
echo ""
echo "═══════════════════════════════════════════════════════════════════"
cat "$REVIEW_PATH/human-decision-request.md"
echo "═══════════════════════════════════════════════════════════════════"
echo ""
echo "📄 Decision request saved to: $REVIEW_PATH/human-decision-request.md"
echo ""
echo "👉 Please provide your decision..."
```

### Step 6: Log Human Decision (Template)

When the human provides their decision, log it:

```bash
# This section is executed after receiving user input
# USER_DECISION should contain the user's response

log_human_decision() {
    USER_DECISION="$1"
    DECISION_REASON="${2:-No reason provided}"
    
    cat > "$REVIEW_PATH/human-decisions.md" << DECISION_LOG
# Human Decisions Log

**Decision Made**: $(date)
**Spec Path**: $SPEC_PATH

---

## Decision

**User Choice**: $USER_DECISION

**Reasoning**: $DECISION_REASON

---

## Context at Decision Time

### Trade-offs Considered
$(grep "TRADE-OFF-" "$REVIEW_PATH/trade-offs.md" 2>/dev/null | head -10 || echo "None")

### Contradictions Considered  
$(grep -E "⛔|⚠️" "$REVIEW_PATH/contradictions.md" 2>/dev/null | head -10 || echo "None")

---

## Resolution

Based on the human decision:
- Proceed with: $USER_DECISION
- Documented for future reference
- Can be referenced for similar future decisions

---

*Logged by present-human-decision workflow*
DECISION_LOG

    echo "✅ Decision logged to: $REVIEW_PATH/human-decisions.md"
}

# Export the function for use by the calling command
export -f log_human_decision 2>/dev/null || true
```

### Step 7: Cache Resolution for Future

```bash
# After decision is logged, cache it for similar future conflicts
cache_decision_resolution() {
    DECISION_TYPE="$1"  # trade-off or contradiction
    DECISION_KEY="$2"   # identifier for the specific issue
    RESOLUTION="$3"     # how it was resolved
    
    RESOLUTION_CACHE="$CACHE_PATH/decision-resolutions.md"
    
    cat >> "$RESOLUTION_CACHE" << RESOLUTION_ENTRY

---

## Resolution: $DECISION_KEY

**Type**: $DECISION_TYPE
**Date**: $(date)
**Resolution**: $RESOLUTION

RESOLUTION_ENTRY
    
    echo "✅ Resolution cached for future reference"
}

export -f cache_decision_resolution 2>/dev/null || true
```

## Important Constraints

- Must load trade-offs and contradictions from cache
- Must format presentation clearly for human review
- Must include pros/cons from basepoints when available
- Must provide AI recommendation with reasoning
- Must log human decision for future reference
- Must cache resolution for similar future conflicts
- **CRITICAL**: All decision logs stored in `$SPEC_PATH/implementation/cache/human-review/`
