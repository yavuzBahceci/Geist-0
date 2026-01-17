Now that you have the task group(s) to be implemented, proceed with implementation.

## Step 1: Extract Basepoints Knowledge (if basepoints exist)

Before implementing tasks, extract relevant basepoints knowledge to inform implementation:

```bash
# Check if basepoints exist
if [ -d "agent-os/basepoints" ] && [ -f "agent-os/basepoints/headquarter.md" ]; then
    # Determine spec path
    SPEC_PATH="agent-os/specs/[current-spec]"
    
    # Extract basepoints knowledge using scope detection
    {{workflows/basepoints/extract-basepoints-knowledge-automatic}}
    {{workflows/scope-detection/detect-abstraction-layer}}
    {{workflows/scope-detection/detect-scope-semantic-analysis}}
    {{workflows/scope-detection/detect-scope-keyword-matching}}
    
    # Load extracted knowledge for use in implementation
    if [ -f "$SPEC_PATH/implementation/cache/basepoints-knowledge.md" ]; then
        EXTRACTED_KNOWLEDGE=$(cat "$SPEC_PATH/implementation/cache/basepoints-knowledge.md")
    fi
    
    # Load detected layer for module-specific patterns
    if [ -f "$SPEC_PATH/implementation/cache/detected-layer.txt" ]; then
        DETECTED_LAYER=$(cat "$SPEC_PATH/implementation/cache/detected-layer.txt")
    fi
fi
```

If basepoints exist, the extracted knowledge will be used to:
- Guide implementation with extracted patterns
- Suggest reusable code and modules during implementation
- Reference project-specific standards and coding patterns
- Use extracted testing approaches for test writing

## Step 2: Check for Trade-offs (if needed)

Before implementing, check if trade-offs need to be reviewed:

```bash
{{workflows/human-review/review-trade-offs}}
```

## Step 3: Check for Checkpoints (if needed)

Before implementing, check if checkpoints are needed for big changes:

```bash
{{workflows/human-review/create-checkpoint}}
```

## Step 4: Implement Tasks

Proceed with implementation by following these instructions:

{{workflows/implementation/implement-tasks}}

## Display confirmation and next step

Display a summary of what was implemented.

IF all tasks are now marked as done (with `- [x]`) in tasks.md, display this message to user:

```
All tasks have been implemented: `agent-os/specs/[this-spec]/tasks.md`.

NEXT STEP 👉 Run `3-verify-implementation.md` to verify the implementation.
```

IF there are still tasks in tasks.md that have yet to be implemented (marked unfinished with `- [ ]`) then display this message to user:

```
Would you like to proceed with implementation of the remaining tasks in tasks.md?

If not, please specify which task group(s) to implement next.
```

## Step 5: Run Validation After Implementation

After each task group is implemented, run validation:

```bash
SPEC_PATH="agent-os/specs/[current-spec]"
COMMAND="implement-tasks"
{{workflows/validation/validate-output-exists}}
{{workflows/validation/validate-knowledge-integration}}
{{workflows/validation/validate-references}}
{{workflows/validation/generate-validation-report}}
```

## Step 6: Generate Resource Checklist

Generate a checklist of all resources consulted during implementation:

```bash
{{workflows/basepoints/organize-and-cache-basepoints-knowledge}}
```

Include in the implementation summary:
- ✅ Tasks implemented: [X of Y]
- ✅ Coding patterns applied: [List from basepoints]
- ✅ Standards referenced: [List from detected layer]
- ✅ Validation: [PASSED / WARNINGS]
- ✅ Resources consulted: See `implementation/cache/resources-consulted.md`

## Step 7: Save Handoff

{{workflows/prompting/save-handoff}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that the tasks list is ALIGNED and DOES NOT CONFLICT with the user's preferences and standards as detailed in the following files:

{{standards/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
