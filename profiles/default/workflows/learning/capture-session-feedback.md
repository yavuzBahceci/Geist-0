# Capture Session Feedback

Called automatically after /implement-tasks to record outcome.

## Purpose

Capture implementation outcome data for session learning:
- Success/failure status
- Patterns used during implementation
- Duration and performance metrics
- Errors encountered
- Validation results

## Inputs

- Spec name (from context)
- Task count (tasks implemented)
- Outcome (pass/fail)
- Validation results
- Patterns used (from implementation)
- Errors encountered (if any)
- Duration (time taken)

## Process

### Step 1: Load or Create Current Session File

```bash
SESSION_DIR="agent-os/output/session-feedback"
SESSION_FILE="$SESSION_DIR/current-session.md"

# Create directory structure if it doesn't exist
mkdir -p "$SESSION_DIR/patterns"
mkdir -p "$SESSION_DIR/prompts"
mkdir -p "$SESSION_DIR/history"

# Create session file if it doesn't exist
if [ ! -f "$SESSION_FILE" ]; then
    TODAY=$(date +%Y-%m-%d)
    cat > "$SESSION_FILE" << EOF
# Session: $TODAY

## Metadata
- Started: $(date -Iseconds)
- Last Updated: $(date -Iseconds)
- Implementations: 0
- Success Rate: 0%

## Implementations

| # | Spec | Tasks | Outcome | Duration | Notes |
|---|------|-------|---------|----------|-------|

## Patterns Observed

### Successful
(Patterns will be listed here as they are detected)

### Failed
(Failed patterns will be listed here as they are detected)

## Prompt Effectiveness

### Worked Well
(Prompts that worked well will be listed here)

### Needed Clarification
(Prompts that needed clarification will be listed here)
EOF
fi
```

### Step 2: Extract Implementation Data

```bash
# Get current implementation count
CURRENT_COUNT=$(grep -c "| [0-9]" "$SESSION_FILE" || echo "0")
NEXT_NUMBER=$((CURRENT_COUNT + 1))

# Extract data from context
SPEC_NAME="$SPEC_NAME"  # From context
TASK_COUNT="$TASK_COUNT"  # From context
OUTCOME="$OUTCOME"  # "✅ Pass" or "❌ Fail"
DURATION="$DURATION"  # From timing
NOTES="$NOTES"  # Any additional notes
PATTERNS_USED="$PATTERNS_USED"  # From implementation analysis
ERRORS="$ERRORS"  # Error messages if any
```

### Step 3: Append Implementation Record

```bash
# Append to implementations table
TEMP_FILE=$(mktemp)
awk -v line="| $NEXT_NUMBER | $SPEC_NAME | $TASK_COUNT | $OUTCOME | $DURATION | $NOTES |" \
    '/^\| # \| Spec \| Tasks \| Outcome \| Duration \| Notes \|$/ {print; print line; next} {print}' \
    "$SESSION_FILE" > "$TEMP_FILE" && mv "$TEMP_FILE" "$SESSION_FILE"
```

### Step 4: Update Metadata

```bash
# Update last updated timestamp
sed -i '' "s/- Last Updated:.*/- Last Updated: $(date -Iseconds)/" "$SESSION_FILE"

# Calculate and update success rate
TOTAL_IMPLEMENTATIONS=$NEXT_NUMBER
SUCCESSFUL=$(grep -c "✅ Pass" "$SESSION_FILE" || echo "0")
SUCCESS_RATE=$(awk "BEGIN {printf \"%.0f\", ($SUCCESSFUL / $TOTAL_IMPLEMENTATIONS) * 100}")

# Update metadata
sed -i '' "s/- Implementations:.*/- Implementations: $TOTAL_IMPLEMENTATIONS/" "$SESSION_FILE"
sed -i '' "s/- Success Rate:.*/- Success Rate: ${SUCCESS_RATE}%/" "$SESSION_FILE"
```

### Step 5: Record Patterns Used

```bash
# Extract patterns from implementation
# This is a placeholder - actual pattern extraction happens in extract-session-patterns workflow
# Here we just note which patterns were used
if [ -n "$PATTERNS_USED" ]; then
    echo "Patterns used in implementation #$NEXT_NUMBER: $PATTERNS_USED" >> "$SESSION_DIR/patterns/raw-patterns.txt"
fi
```

### Step 6: Record Errors (if any)

```bash
# If implementation failed, record error details
if [ "$OUTCOME" = "❌ Fail" ] && [ -n "$ERRORS" ]; then
    cat >> "$SESSION_FILE" << EOF

### Failed Implementation #$NEXT_NUMBER
- Spec: $SPEC_NAME
- Error: $ERRORS
- Duration: $DURATION
EOF
fi
```

### Step 7: Record Prompt Effectiveness

```bash
# Track if prompts worked well or needed clarification
if [ "$OUTCOME" = "✅ Pass" ] && [ -z "$ERRORS" ]; then
    # Mark as worked well
    echo "Implementation #$NEXT_NUMBER: Passed without issues" >> "$SESSION_DIR/prompts/effective-raw.txt"
else
    # Mark as needing improvement
    echo "Implementation #$NEXT_NUMBER: $ERRORS" >> "$SESSION_DIR/prompts/needs-improvement-raw.txt"
fi
```

## Output

Updates: `agent-os/output/session-feedback/current-session.md`

The session file is updated with:
- New implementation record in table
- Updated metadata (count, success rate, last updated)
- Error details (if failed)
- Raw pattern and prompt data for later analysis

## Usage

Called automatically at the end of `/implement-tasks` after verification:

```markdown
## Step N: Capture Session Feedback

{{workflows/learning/capture-session-feedback}}
```

## Notes

- Session file is created automatically if it doesn't exist
- Patterns are recorded in raw format for later extraction
- Prompt effectiveness is tracked for learning
- All data is timestamped for analysis
