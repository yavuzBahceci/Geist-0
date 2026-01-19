# Feedback Checkpoint for Command Outputs

## Core Responsibilities

1. **Display Output Summary**: Show the command's output for user review
2. **Prompt for Feedback**: Ask user if they want to update or continue
3. **Handle User Response**: Process feedback or proceed based on user choice
4. **Enable Iteration**: Allow multiple feedback rounds before proceeding

## Workflow

### Step 1: Initialize Checkpoint Variables

Set up checkpoint variables from calling command:

```bash
# OUTPUT_SUMMARY should be set by the calling command
# NEXT_STEP should be set by the calling command
# ALLOW_UPDATE should be set by the calling command (default: true)
# CURRENT_COMMAND should be set by the calling command

if [ -z "$OUTPUT_SUMMARY" ]; then
    OUTPUT_SUMMARY="Command output not provided."
fi

if [ -z "$NEXT_STEP" ]; then
    NEXT_STEP="next phase"
fi

if [ -z "$ALLOW_UPDATE" ]; then
    ALLOW_UPDATE="true"
fi

if [ -z "$CURRENT_COMMAND" ]; then
    CURRENT_COMMAND="unknown-command"
fi

# Track feedback iterations
FEEDBACK_ITERATION=0
MAX_ITERATIONS=10
```

### Step 2: Display Output and Prompt

Present the output and ask for user feedback:

```bash
display_feedback_checkpoint() {
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "✅ $CURRENT_COMMAND COMPLETE"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo ""
    echo "$OUTPUT_SUMMARY"
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    
    if [ "$ALLOW_UPDATE" = "true" ]; then
        echo ""
        echo "**Do you want to update this output, or continue with $NEXT_STEP?**"
        echo ""
        echo "- To update: provide your feedback"
        echo "- To continue: type \"continue\" or just proceed"
        echo ""
        echo "⚠️ **STOP and WAIT** for user response."
        echo ""
    else
        echo ""
        echo "➡️ Proceeding to $NEXT_STEP..."
        echo ""
    fi
}

# Display the checkpoint
display_feedback_checkpoint
```

**Output Format:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [COMMAND] COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[OUTPUT_SUMMARY]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Do you want to update this output, or continue with [NEXT_STEP]?**

- To update: provide your feedback
- To continue: type "continue" or just proceed

⚠️ **STOP and WAIT** for user response.
```

### Step 3: Process User Response

Handle the user's response:

```bash
process_user_response() {
    local user_response="$1"
    
    # Normalize response
    local normalized_response=$(echo "$user_response" | tr '[:upper:]' '[:lower:]' | xargs)
    
    # Check for continue signals
    if [ "$normalized_response" = "continue" ] || \
       [ "$normalized_response" = "proceed" ] || \
       [ "$normalized_response" = "yes" ] || \
       [ "$normalized_response" = "ok" ] || \
       [ "$normalized_response" = "next" ] || \
       [ -z "$normalized_response" ]; then
        FEEDBACK_ACTION="continue"
        USER_FEEDBACK=""
        return 0
    fi
    
    # Anything else is feedback
    FEEDBACK_ACTION="update"
    USER_FEEDBACK="$user_response"
    return 0
}

# Process the user's response
# USER_RESPONSE should be captured from user input
process_user_response "$USER_RESPONSE"
```

### Step 4: Handle Feedback Loop

If user provides feedback, process it and re-display:

```bash
handle_feedback_loop() {
    while [ "$FEEDBACK_ACTION" = "update" ] && [ "$FEEDBACK_ITERATION" -lt "$MAX_ITERATIONS" ]; do
        FEEDBACK_ITERATION=$((FEEDBACK_ITERATION + 1))
        
        echo ""
        echo "📝 Processing feedback (iteration $FEEDBACK_ITERATION)..."
        echo ""
        echo "User feedback:"
        echo "$USER_FEEDBACK"
        echo ""
        
        # The calling command should update its output based on feedback
        # Then set OUTPUT_SUMMARY to the new output
        # FEEDBACK_PROCESSED should be set to "true" by calling command
        
        if [ "$FEEDBACK_PROCESSED" = "true" ]; then
            echo "✅ Output updated based on feedback."
            echo ""
            
            # Re-display checkpoint with updated output
            display_feedback_checkpoint
            
            # Wait for next user response
            # USER_RESPONSE should be captured again
            process_user_response "$USER_RESPONSE"
        else
            echo "⚠️ Feedback processing not implemented for this command."
            FEEDBACK_ACTION="continue"
        fi
    done
    
    if [ "$FEEDBACK_ITERATION" -ge "$MAX_ITERATIONS" ]; then
        echo "⚠️ Maximum feedback iterations reached. Proceeding..."
        FEEDBACK_ACTION="continue"
    fi
}

# Handle feedback loop if needed
if [ "$FEEDBACK_ACTION" = "update" ]; then
    handle_feedback_loop
fi
```

### Step 5: Record Checkpoint Result

Store the checkpoint result for tracking:

```bash
# Determine cache path
if [ -n "$SPEC_PATH" ]; then
    CACHE_PATH="$SPEC_PATH/implementation/cache"
else
    CACHE_PATH="geist/output/feedback-checkpoints"
fi

mkdir -p "$CACHE_PATH/feedback-checkpoints"

# Store checkpoint result
CHECKPOINT_TIMESTAMP=$(date +%Y%m%d_%H%M%S)
CHECKPOINT_FILE="$CACHE_PATH/feedback-checkpoints/${CURRENT_COMMAND}_${CHECKPOINT_TIMESTAMP}.md"

cat > "$CHECKPOINT_FILE" << CHECKPOINT_EOF
# Feedback Checkpoint Result

## Command
$CURRENT_COMMAND

## Timestamp
$(date)

## Output Summary
$OUTPUT_SUMMARY

## User Response
$FEEDBACK_ACTION

## Feedback Iterations
$FEEDBACK_ITERATION

## User Feedback (if any)
$USER_FEEDBACK

## Next Step
$NEXT_STEP

## Proceeded
$(if [ "$FEEDBACK_ACTION" = "continue" ]; then echo "Yes"; else echo "No"; fi)
CHECKPOINT_EOF

echo "📋 Checkpoint recorded: $CHECKPOINT_FILE"
```

### Step 6: Return Status

Indicate whether to proceed:

```bash
if [ "$FEEDBACK_ACTION" = "continue" ]; then
    echo ""
    echo "➡️ Proceeding to $NEXT_STEP..."
    echo ""
    CHECKPOINT_PROCEED="true"
else
    echo ""
    echo "⏸️ Paused at checkpoint. Awaiting user decision."
    echo ""
    CHECKPOINT_PROCEED="false"
fi

# Export for calling command
export CHECKPOINT_PROCEED
export USER_FEEDBACK
export FEEDBACK_ACTION
```

## Usage Notes

This workflow is designed to be called after major command outputs to allow user feedback before proceeding.

**Typical Usage Pattern:**

```bash
# At the end of a command phase, after generating output:
CURRENT_COMMAND="shape-spec"
OUTPUT_SUMMARY="
Requirements gathered:
- 5 user stories documented
- 8 functional requirements captured
- Scope boundaries defined

Files created:
- geist/specs/[spec]/planning/requirements.md
"
NEXT_STEP="/write-spec to create the spec.md document"
ALLOW_UPDATE=true

{{workflows/human-review/feedback-checkpoint}}

# Check if we should proceed
if [ "$CHECKPOINT_PROCEED" = "true" ]; then
    # Continue to next step
    echo "Proceeding..."
else
    # Handle feedback (this branch typically handled by AI agent)
    echo "Processing feedback: $USER_FEEDBACK"
fi
```

**Required Variables:**
- `$CURRENT_COMMAND` - Name of the current command
- `$OUTPUT_SUMMARY` - Summary of what was created/done
- `$NEXT_STEP` - Description of what happens next

**Optional Variables:**
- `$ALLOW_UPDATE` - Whether updates are allowed (default: true)
- `$SPEC_PATH` - Path to current spec (for checkpoint storage)

**Output Variables:**
- `$CHECKPOINT_PROCEED` - Whether to proceed (true/false)
- `$USER_FEEDBACK` - User's feedback text (if any)
- `$FEEDBACK_ACTION` - User's action (continue/update)

## Integration with Commands

Commands should integrate this workflow at key output points:

| Command | Checkpoint After | NEXT_STEP |
|---------|------------------|-----------|
| shape-spec | Requirements gathered | /write-spec |
| write-spec | spec.md created | /create-tasks |
| create-tasks | tasks.md created | /implement-tasks or /orchestrate-tasks |
| implement-tasks | Each task group | Next task group or completion |

## Important Constraints

- **STOP and WAIT**: The workflow must stop and wait for user response
- **Allow Multiple Iterations**: User can provide feedback multiple times
- **Explicit Proceed**: Only proceed when user explicitly confirms or says "continue"
- **Record All Checkpoints**: Store checkpoint results for audit trail
- **Respect ALLOW_UPDATE**: If false, proceed without asking
- **Max Iterations**: Prevent infinite feedback loops (default: 10)
