# Analyze Prompt Effectiveness

Determine which prompts led to good/bad outcomes.

## Purpose

Analyze prompt effectiveness from session data to identify:
- Prompts that worked well (effective)
- Prompts that needed clarification (ineffective)
- Improvement opportunities for prompt construction

## Inputs

- Session file: `geist/output/session-feedback/current-session.md`
- Raw prompt data: `geist/output/session-feedback/prompts/effective-raw.txt`
- Raw prompt issues: `geist/output/session-feedback/prompts/needs-improvement-raw.txt`

## Indicators of Effective Prompts

Prompts that lead to:
- ✅ Implementation completed without retries
- ✅ No clarification questions needed
- ✅ Validation passed first time
- ✅ Correct files created/modified
- ✅ Clear understanding of requirements

## Indicators of Ineffective Prompts

Prompts that lead to:
- ❌ Multiple retries needed
- ❌ User had to clarify requirements
- ❌ Validation failed due to misunderstanding
- ❌ Wrong files created/modified
- ❌ Missing context or constraints

## Process

### Step 1: Load Session Data

```bash
SESSION_FILE="geist/output/session-feedback/current-session.md"
PROMPTS_DIR="geist/output/session-feedback/prompts"

# Create prompts directory if needed
mkdir -p "$PROMPTS_DIR"

# Read session file
if [ ! -f "$SESSION_FILE" ]; then
    echo "⚠️ No session file found. Run /implement-tasks first."
    exit 0
fi

# Extract implementations
IMPLEMENTATIONS=$(grep -A 100 "| # | Spec | Tasks | Outcome | Duration | Notes |" "$SESSION_FILE" | grep "^| [0-9]" || echo "")
```

### Step 2: Analyze Effective Prompts

```bash
EFFECTIVE_FILE="$PROMPTS_DIR/effective.md"
EFFECTIVE_RAW="$PROMPTS_DIR/effective-raw.txt"

cat > "$EFFECTIVE_FILE" << EOF
# Effective Prompts

Prompts that led to successful implementations without issues.

EOF

# Count effective prompts
EFFECTIVE_COUNT=0

# Read raw effective data
if [ -f "$EFFECTIVE_RAW" ]; then
    while read -r line; do
        if [[ "$line" =~ "Implementation #([0-9]+): Passed without issues" ]]; then
            IMPL_NUM="${BASH_REMATCH[1]}"
            EFFECTIVE_COUNT=$((EFFECTIVE_COUNT + 1))
            
            # Extract implementation details
            IMPL_DETAILS=$(echo "$IMPLEMENTATIONS" | grep "^| $IMPL_NUM " || echo "")
            
            if [ -n "$IMPL_DETAILS" ]; then
                SPEC=$(echo "$IMPL_DETAILS" | awk -F'|' '{print $3}' | tr -d ' ')
                TASKS=$(echo "$IMPL_DETAILS" | awk -F'|' '{print $4}' | tr -d ' ')
                
                cat >> "$EFFECTIVE_FILE" << EOF
## Implementation #$IMPL_NUM: $SPEC

**Spec**: $SPEC
**Tasks**: $TASKS
**Outcome**: ✅ Passed without issues

**What Worked Well**:
- No retries needed
- Validation passed first time
- Clear understanding of requirements

**Prompt Characteristics**:
- Sufficient context provided
- Clear constraints defined
- Output format specified

---

EOF
            fi
        fi
    done < "$EFFECTIVE_RAW"
fi

echo "✅ Effective prompts: $EFFECTIVE_COUNT identified"
```

### Step 3: Analyze Ineffective Prompts

```bash
INEFFECTIVE_FILE="$PROMPTS_DIR/needs-improvement.md"
INEFFECTIVE_RAW="$PROMPTS_DIR/needs-improvement-raw.txt"

cat > "$INEFFECTIVE_FILE" << EOF
# Prompts Needing Improvement

Prompts that led to issues or required clarification.

EOF

# Count ineffective prompts
INEFFECTIVE_COUNT=0

# Read raw ineffective data
if [ -f "$INEFFECTIVE_RAW" ]; then
    while read -r line; do
        if [[ "$line" =~ "Implementation #([0-9]+): (.+)" ]]; then
            IMPL_NUM="${BASH_REMATCH[1]}"
            ISSUE="${BASH_REMATCH[2]}"
            INEFFECTIVE_COUNT=$((INEFFECTIVE_COUNT + 1))
            
            # Extract implementation details
            IMPL_DETAILS=$(echo "$IMPLEMENTATIONS" | grep "^| $IMPL_NUM " || echo "")
            
            if [ -n "$IMPL_DETAILS" ]; then
                SPEC=$(echo "$IMPL_DETAILS" | awk -F'|' '{print $3}' | tr -d ' ')
                TASKS=$(echo "$IMPL_DETAILS" | awk -F'|' '{print $4}' | tr -d ' ')
                OUTCOME=$(echo "$IMPL_DETAILS" | awk -F'|' '{print $5}' | tr -d ' ')
                
                cat >> "$INEFFECTIVE_FILE" << EOF
## Implementation #$IMPL_NUM: $SPEC

**Spec**: $SPEC
**Tasks**: $TASKS
**Outcome**: $OUTCOME
**Issue**: $ISSUE

**What Went Wrong**:
- [Issue details]

**Prompt Issues**:
- Missing context: [specific context needed]
- Unclear instructions: [what was unclear]
- Missing constraints: [what constraints were missing]

**Recommended Improvements**:
- Add: [specific improvement]
- Clarify: [what to clarify]
- Include: [missing information]

---

EOF
            fi
        fi
    done < "$INEFFECTIVE_RAW"
fi

echo "✅ Prompts needing improvement: $INEFFECTIVE_COUNT identified"
```

### Step 4: Generate Prompt Effectiveness Summary

```bash
# Update session file with prompt effectiveness summary
sed -i '' "/## Prompt Effectiveness/,/^#/c\\
## Prompt Effectiveness\\
\\
### Worked Well\\
$EFFECTIVE_COUNT prompt(s) led to successful implementations without issues\\
\\
### Needed Clarification\\
$INEFFECTIVE_COUNT prompt(s) required clarification or led to issues\\
" "$SESSION_FILE"
```

### Step 5: Generate Improvement Suggestions

```bash
SUGGESTIONS_FILE="$PROMPTS_DIR/improvement-suggestions.md"

cat > "$SUGGESTIONS_FILE" << EOF
# Prompt Improvement Suggestions

Based on analysis of $INEFFECTIVE_COUNT problematic implementations.

## Common Issues

EOF

# Extract common issues from ineffective prompts
if [ -f "$INEFFECTIVE_FILE" ]; then
    # Count common issues
    MISSING_CONTEXT=$(grep -c "Missing context:" "$INEFFECTIVE_FILE" || echo "0")
    UNCLEAR_INSTRUCTIONS=$(grep -c "Unclear instructions:" "$INEFFECTIVE_FILE" || echo "0")
    MISSING_CONSTRAINTS=$(grep -c "Missing constraints:" "$INEFFECTIVE_FILE" || echo "0")
    
    cat >> "$SUGGESTIONS_FILE" << EOF
1. **Missing Context**: $MISSING_CONTEXT occurrences
   - Add project patterns from basepoints
   - Include relevant tech stack info
   - Reference similar previous implementations

2. **Unclear Instructions**: $UNCLEAR_INSTRUCTIONS occurrences
   - Use numbered steps
   - Specify expected outputs
   - Define clear boundaries

3. **Missing Constraints**: $MISSING_CONSTRAINTS occurrences
   - Add "DO NOT" sections
   - Include anti-patterns to avoid
   - Specify validation requirements

## Recommendations

- Always include basepoints context
- Use clear, numbered steps
- Explicitly state constraints
- Specify output format
- Include validation criteria

EOF
fi

echo "✅ Improvement suggestions: $SUGGESTIONS_FILE"
```

## Output

- Effective prompts: `geist/output/session-feedback/prompts/effective.md`
- Prompts needing improvement: `geist/output/session-feedback/prompts/needs-improvement.md`
- Improvement suggestions: `geist/output/session-feedback/prompts/improvement-suggestions.md`
- Updated session file with prompt effectiveness summary

## Usage

Called by `/update-basepoints-and-redeploy` Phase 7 to analyze prompt effectiveness before human review.

Can also be called manually to analyze current session data.

## Notes

- Effectiveness is measured by implementation success without retries
- Patterns in ineffective prompts are used to improve prompt construction
- Analysis improves over time as more data is collected
