# Extract Session Patterns

Analyze session data to identify successful and failed patterns.

## Purpose

Extract patterns from session implementation data to identify:
- Successful patterns (100% success rate, 3+ uses)
- Failed patterns/anti-patterns (any failure)
- Pattern usage statistics

## Inputs

- Session file: `agent-os/output/session-feedback/current-session.md`
- Raw patterns file: `agent-os/output/session-feedback/patterns/raw-patterns.txt`

## Process

### Step 1: Read All Implementations from Session

```bash
SESSION_FILE="agent-os/output/session-feedback/current-session.md"
PATTERNS_DIR="agent-os/output/session-feedback/patterns"

# Read session file
if [ ! -f "$SESSION_FILE" ]; then
    echo "⚠️ No session file found. Run /implement-tasks first."
    exit 0
fi

# Extract implementations table
IMPLEMENTATIONS=$(grep -A 100 "| # | Spec | Tasks | Outcome | Duration | Notes |" "$SESSION_FILE" | grep "^| [0-9]" || echo "")
```

### Step 2: Group by Patterns Used

```bash
# Parse patterns from raw patterns file
RAW_PATTERNS_FILE="$PATTERNS_DIR/raw-patterns.txt"

if [ -f "$RAW_PATTERNS_FILE" ]; then
    # Extract patterns mentioned in implementations
    # Pattern format: "Pattern: [name] (used in implementation #N)"
    
    # Group patterns by name
    declare -A PATTERN_COUNTS
    declare -A PATTERN_SUCCESS_COUNTS
    declare -A PATTERN_FAILURE_COUNTS
    
    while read -r line; do
        if [[ "$line" =~ "Pattern:" ]]; then
            PATTERN_NAME=$(echo "$line" | sed -n 's/.*Pattern: \([^ ]*\).*/\1/p')
            IMPL_NUM=$(echo "$line" | sed -n 's/.*implementation #\([0-9]*\).*/\1/p')
            
            # Check if this implementation was successful
            OUTCOME=$(echo "$IMPLEMENTATIONS" | grep "^| $IMPL_NUM " | awk -F'|' '{print $5}' | tr -d ' ')
            
            if [ -n "$PATTERN_NAME" ]; then
                PATTERN_COUNTS["$PATTERN_NAME"]=$((${PATTERN_COUNTS["$PATTERN_NAME"]:-0} + 1))
                
                if [[ "$OUTCOME" =~ "✅" ]]; then
                    PATTERN_SUCCESS_COUNTS["$PATTERN_NAME"]=$((${PATTERN_SUCCESS_COUNTS["$PATTERN_NAME"]:-0} + 1))
                else
                    PATTERN_FAILURE_COUNTS["$PATTERN_NAME"]=$((${PATTERN_FAILURE_COUNTS["$PATTERN_NAME"]:-0} + 1))
                fi
            fi
        fi
    done < "$RAW_PATTERNS_FILE"
fi
```

### Step 3: Calculate Success Rate per Pattern

```bash
# Calculate success rates
declare -A PATTERN_SUCCESS_RATES

for pattern in "${!PATTERN_COUNTS[@]}"; do
    total=${PATTERN_COUNTS["$pattern"]}
    successes=${PATTERN_SUCCESS_COUNTS["$pattern"]:-0}
    failures=${PATTERN_FAILURE_COUNTS["$pattern"]:-0}
    
    if [ $total -gt 0 ]; then
        success_rate=$(awk "BEGIN {printf \"%.2f\", ($successes / $total)}")
        PATTERN_SUCCESS_RATES["$pattern"]=$success_rate
    fi
done
```

### Step 4: Identify Successful Patterns

```bash
# Thresholds from config
MIN_USES_FOR_RECOMMENDATION=3
SUCCESS_RATE_THRESHOLD=1.0  # 100%

SUCCESSFUL_PATTERNS_FILE="$PATTERNS_DIR/successful.md"

cat > "$SUCCESSFUL_PATTERNS_FILE" << EOF
# Successful Patterns

EOF

for pattern in "${!PATTERN_COUNTS[@]}"; do
    total=${PATTERN_COUNTS["$pattern"]}
    success_rate=${PATTERN_SUCCESS_RATES["$pattern"]}
    
    # Pattern is successful if:
    # - Used 3+ times
    # - 100% success rate
    if [ $total -ge $MIN_USES_FOR_RECOMMENDATION ] && [ "$(echo "$success_rate >= $SUCCESS_RATE_THRESHOLD" | bc -l)" = "1" ]; then
        cat >> "$SUCCESSFUL_PATTERNS_FILE" << EOF
## Pattern: $pattern

**Description**: Pattern extracted from successful implementations

**First Observed**: $(date +%Y-%m-%d)
**Times Used**: $total
**Success Rate**: $(awk "BEGIN {printf \"%.0f\", $success_rate * 100}")%

**Applicable To**: [Context where this pattern applies]

---

EOF
    fi
done

echo "✅ Successful patterns written to: $SUCCESSFUL_PATTERNS_FILE"
```

### Step 5: Identify Failed Patterns (Anti-patterns)

```bash
FAILED_PATTERNS_FILE="$PATTERNS_DIR/failed.md"

cat > "$FAILED_PATTERNS_FILE" << EOF
# Failed Patterns (Anti-patterns)

EOF

for pattern in "${!PATTERN_COUNTS[@]}"; do
    failures=${PATTERN_FAILURE_COUNTS["$pattern"]:-0}
    
    # Pattern is an anti-pattern if it caused any failures
    if [ $failures -gt 0 ]; then
        total=${PATTERN_COUNTS["$pattern"]}
        failure_rate=$(awk "BEGIN {printf \"%.2f\", ($failures / $total)}")
        
        cat >> "$FAILED_PATTERNS_FILE" << EOF
## Pattern: $pattern

**Description**: This pattern caused implementation failures

**First Observed**: $(date +%Y-%m-%d)
**Times Used**: $total
**Failure Rate**: $(awk "BEGIN {printf \"%.0f\", $failure_rate * 100}")%

**Error Details**:
- Implementations affected: $failures of $total
- Common errors: [Extract from session data]

**Recommended Alternative**: [Suggest alternative approach]

---

EOF
    fi
done

echo "✅ Failed patterns written to: $FAILED_PATTERNS_FILE"
```

### Step 6: Write Pattern Summary to Session File

```bash
# Update session file with pattern summary
SUCCESSFUL_COUNT=$(grep -c "^## Pattern:" "$SUCCESSFUL_PATTERNS_FILE" 2>/dev/null || echo "0")
FAILED_COUNT=$(grep -c "^## Pattern:" "$FAILED_PATTERNS_FILE" 2>/dev/null || echo "0")

# Update session file patterns section
sed -i '' "/## Patterns Observed/,/## Prompt Effectiveness/c\\
## Patterns Observed\\
\\
### Successful\\
$SUCCESSFUL_COUNT pattern(s) identified with 100% success rate and 3+ uses\\
\\
### Failed\\
$FAILED_COUNT anti-pattern(s) identified\\
\\
## Prompt Effectiveness" "$SESSION_FILE"
```

## Output

- Successful patterns: `agent-os/output/session-feedback/patterns/successful.md`
- Failed patterns: `agent-os/output/session-feedback/patterns/failed.md`
- Updated session file with pattern summary

## Thresholds

From `workflow-config.yml`:
- `min_uses_for_recommendation: 3` - Minimum uses before recommending
- `success_rate_threshold: 1.0` - 100% success rate required

## Usage

Called by `/update-basepoints-and-redeploy` Phase 7 to extract patterns before human review.

Can also be called manually to analyze current session data.

## Notes

- Patterns with 100% success and 3+ uses are marked as successful
- Patterns with any failures are marked as anti-patterns
- Pattern extraction improves over time as more data is collected
