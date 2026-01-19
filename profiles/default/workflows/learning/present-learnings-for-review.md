# Present Learnings for Review

Workflow to present session learnings to user for review and approval.

## Purpose

Display session learnings summary with clear options for user approval:
- Successful patterns discovered
- Anti-patterns identified
- Prompt effectiveness analysis
- Proposed adaptations

## Inputs

- Session file: `geist/output/session-feedback/current-session.md`
- Successful patterns: `geist/output/session-feedback/patterns/successful.md`
- Failed patterns: `geist/output/session-feedback/patterns/failed.md`
- Prompt analysis: `geist/output/session-feedback/prompts/effective.md` and `needs-improvement.md`

## Process

### Step 1: Load Session Learnings

```bash
SESSION_FILE="geist/output/session-feedback/current-session.md"
PATTERNS_SUCCESS="geist/output/session-feedback/patterns/successful.md"
PATTERNS_FAILED="geist/output/session-feedback/patterns/failed.md"
PROMPTS_EFFECTIVE="geist/output/session-feedback/prompts/effective.md"
PROMPTS_IMPROVEMENT="geist/output/session-feedback/prompts/needs-improvement.md"

if [ ! -f "$SESSION_FILE" ]; then
    echo "ℹ️ No session feedback found. Skipping learning phase."
    echo "   Run /implement-tasks to generate session feedback."
    exit 0
fi

echo "📋 Loading session learnings..."
```

### Step 2: Extract Session Summary

```bash
# Extract metadata from session
TOTAL_IMPL=$(grep "Implementations:" "$SESSION_FILE" | sed 's/.*: \([0-9]*\).*/\1/' | head -1)
SUCCESS_RATE=$(grep "Success Rate:" "$SESSION_FILE" | sed 's/.*: \([0-9]*\)%.*/\1/' | head -1)

# Count successful implementations
SUCCESSFUL=$(grep -c "✅ Pass" "$SESSION_FILE" || echo "0")
FAILED=$(grep -c "❌ Fail" "$SESSION_FILE" || echo "0")

# Count patterns
SUCCESSFUL_PATTERNS_COUNT=$(grep -c "^## Pattern:" "$PATTERNS_SUCCESS" 2>/dev/null || echo "0")
FAILED_PATTERNS_COUNT=$(grep -c "^## Pattern:" "$PATTERNS_FAILED" 2>/dev/null || echo "0")
```

### Step 3: Extract Pattern Details

```bash
# Extract successful patterns
SUCCESSFUL_PATTERNS=""
if [ -f "$PATTERNS_SUCCESS" ]; then
    while IFS= read -r line; do
        if [[ "$line" =~ "^## Pattern: (.*)" ]]; then
            PATTERN_NAME="${BASH_REMATCH[1]}"
            # Get times used
            TIMES_USED=$(grep -A 5 "^## Pattern: $PATTERN_NAME" "$PATTERNS_SUCCESS" | grep "Times Used:" | sed 's/.*: \([0-9]*\).*/\1/')
            SUCCESSFUL_PATTERNS="${SUCCESSFUL_PATTERNS}  - $PATTERN_NAME (used $TIMES_USED times)\n"
        fi
    done < "$PATTERNS_SUCCESS"
fi

# Extract failed patterns
FAILED_PATTERNS=""
if [ -f "$PATTERNS_FAILED" ]; then
    while IFS= read -r line; do
        if [[ "$line" =~ "^## Pattern: (.*)" ]]; then
            PATTERN_NAME="${BASH_REMATCH[1]}"
            # Get error details
            ERROR=$(grep -A 10 "^## Pattern: $PATTERN_NAME" "$PATTERNS_FAILED" | grep "Error Details:" -A 3 | head -1)
            FAILED_PATTERNS="${FAILED_PATTERNS}  - $PATTERN_NAME: $ERROR\n"
        fi
    done < "$PATTERNS_FAILED"
fi
```

### Step 4: Extract Prompt Effectiveness

```bash
# Count effective prompts
EFFECTIVE_COUNT=$(grep -c "^## Implementation #" "$PROMPTS_EFFECTIVE" 2>/dev/null || echo "0")
IMPROVEMENT_COUNT=$(grep -c "^## Implementation #" "$PROMPTS_IMPROVEMENT" 2>/dev/null || echo "0")
```

### Step 5: Generate Adaptation Suggestions

```bash
# Generate suggestions based on patterns
SUGGESTIONS=""

# For successful patterns - suggest adding to commands
if [ $SUCCESSFUL_PATTERNS_COUNT -gt 0 ]; then
    SUGGESTIONS="${SUGGESTIONS}✅ Add successful patterns to command context\n"
fi

# For failed patterns - suggest adding warnings
if [ $FAILED_PATTERNS_COUNT -gt 0 ]; then
    SUGGESTIONS="${SUGGESTIONS}⚠️ Add anti-pattern warnings to commands\n"
fi

# For prompt improvements
if [ $IMPROVEMENT_COUNT -gt 0 ]; then
    SUGGESTIONS="${SUGGESTIONS}📝 Improve prompts based on effectiveness analysis\n"
fi
```

### Step 6: Present Review UI

Display the review UI:

```
┌─────────────────────────────────────────────────────────────────┐
│  🔍 SESSION LEARNINGS REVIEW                                    │
│                                                                 │
│  📊 Session Summary:                                            │
│  • Implementations: $TOTAL_IMPL total ($SUCCESSFUL successful, $FAILED failed) │
│  • Success Rate: $SUCCESS_RATE%                                   │
│  • New Patterns: $SUCCESSFUL_PATTERNS_COUNT                                       │
│  • Anti-patterns: $FAILED_PATTERNS_COUNT                                       │
│                                                                 │
│  ═══════════════════════════════════════════════════════════    │
│                                                                 │
│  ✅ SUCCESSFUL PATTERNS                                         │
│                                                                 │
EOF

# Display successful patterns
if [ $SUCCESSFUL_PATTERNS_COUNT -gt 0 ]; then
    echo "$SUCCESSFUL_PATTERNS" | while read -r pattern; do
        if [ -n "$pattern" ]; then
            echo "│  $pattern"
        fi
    done
else
    echo "│  (No successful patterns identified yet)"
fi

cat << EOF
│                                                                 │
│  ═══════════════════════════════════════════════════════════    │
│                                                                 │
│  ❌ ANTI-PATTERNS (caused failures)                             │
│                                                                 │
EOF

# Display failed patterns
if [ $FAILED_PATTERNS_COUNT -gt 0 ]; then
    echo "$FAILED_PATTERNS" | while read -r pattern; do
        if [ -n "$pattern" ]; then
            echo "│  $pattern"
        fi
    done
else
    echo "│  (No anti-patterns identified)"
fi

cat << EOF
│                                                                 │
│  ═══════════════════════════════════════════════════════════    │
│                                                                 │
│  📝 PROMPT EFFECTIVENESS                                        │
│                                                                 │
│  ✅ Effective: $EFFECTIVE_COUNT prompt(s)                          │
│  ⚠️ Needs Improvement: $IMPROVEMENT_COUNT prompt(s)                            │
│                                                                 │
│  ═══════════════════════════════════════════════════════════    │
│                                                                 │
│  💡 PROPOSED ADAPTATIONS                                        │
│                                                                 │
EOF

echo "$SUGGESTIONS" | while read -r suggestion; do
    if [ -n "$suggestion" ]; then
        echo "│  $suggestion"
    fi
done

cat << EOF
│                                                                 │
│  ═══════════════════════════════════════════════════════════    │
│                                                                 │
│  Options:                                                       │
│  [a] Apply all adaptations                                      │
│  [s] Select which to apply                                      │
│  [n] Skip (save learnings for later)                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Step 7: Wait for User Response

```bash
echo ""
read -p "Your choice [a/s/n]: " USER_CHOICE

case "$USER_CHOICE" in
    "a"|"A"|"all")
        APPROVAL_TYPE="all"
        ;;
    "s"|"S"|"select")
        APPROVAL_TYPE="select"
        ;;
    "n"|"N"|"skip")
        APPROVAL_TYPE="skip"
        ;;
    *)
        echo "⚠️ Invalid choice. Saving for later review."
        APPROVAL_TYPE="skip"
        ;;
esac

echo "User choice: $APPROVAL_TYPE"
```

### Step 8: Save Adaptations to Pending

```bash
ADAPTATIONS_DIR="geist/output/session-feedback/adaptations"
PENDING_FILE="$ADAPTATIONS_DIR/pending.md"

mkdir -p "$ADAPTATIONS_DIR"

cat > "$PENDING_FILE" << EOF
# Pending Adaptations

**Created**: $(date -Iseconds)
**Approval Status**: $APPROVAL_TYPE

## Session Summary
- Total Implementations: $TOTAL_IMPL
- Success Rate: $SUCCESS_RATE%
- Successful Patterns: $SUCCESSFUL_PATTERNS_COUNT
- Anti-patterns: $FAILED_PATTERNS_COUNT

## Proposed Adaptations

EOF

# Add adaptations based on approval type
if [ "$APPROVAL_TYPE" = "all" ] || [ "$APPROVAL_TYPE" = "select" ]; then
    cat >> "$PENDING_FILE" << EOF
### Successful Patterns Adaptations
[Adaptations for successful patterns]

### Anti-patterns Adaptations
[Adaptations for anti-patterns]

### Prompt Improvements
[Adaptations for prompt improvements]
EOF
fi

echo "✅ Adaptations saved to: $PENDING_FILE"
```

## Output

- User approval choice (all/select/skip)
- Pending adaptations file: `geist/output/session-feedback/adaptations/pending.md`

## Critical Safety

- ALWAYS asks user before applying any changes
- NEVER auto-applies adaptations
- Clear presentation of what will change
- Option to reject and save for later

## Usage

Called by Phase 7 of `/update-basepoints-and-redeploy` to present learnings for human review.
