Now that you have the spec.md AND/OR requirements.md, proceed with creating the tasks list.

## Step 1: Extract Basepoints Knowledge (if basepoints exist)

Before creating the tasks list, extract relevant basepoints knowledge to inform task creation:

```bash
{{workflows/common/extract-basepoints-with-scope-detection}}
```

If basepoints exist, the extracted knowledge (`$EXTRACTED_KNOWLEDGE` and `$DETECTED_LAYER`) will be used to:
- Inform task breakdown with existing patterns
- Suggest existing patterns and checkpoints from basepoints
- Reference project-specific implementation strategies
- Include relevant testing approaches in task creation

## Step 2: Check for Trade-offs (if needed)

Before creating tasks, check if trade-offs need to be reviewed:

```bash
{{workflows/human-review/review-trade-offs}}
```

## Step 3: Create Tasks List

Break down the spec and requirements into an actionable tasks list with strategic grouping and ordering, by following these instructions:

{{workflows/implementation/create-tasks-list}}

## Step 4: Run Validation

Before completing, run validation to ensure outputs are correct:

```bash
SPEC_PATH="agent-os/specs/[current-spec]"
COMMAND="create-tasks"
{{workflows/validation/validate-output-exists}}
{{workflows/validation/validate-knowledge-integration}}
{{workflows/validation/generate-validation-report}}
```

## Step 5: Generate Resource Checklist

Generate a checklist of all resources consulted:

```bash
{{workflows/basepoints/organize-and-cache-basepoints-knowledge}}
```

## Display confirmation and next step

Display the following message to the user:

```
The tasks list has been created at `agent-os/specs/[this-spec]/tasks.md`.

✅ Tasks breakdown complete
✅ Basepoints knowledge used: [Yes / No basepoints found]
✅ Implementation hints included: [Yes / No patterns found]
✅ Validation: [PASSED / WARNINGS]
✅ Resources consulted: See `implementation/cache/resources-consulted.md`

Review it closely to make sure it all looks good.

👉 Run `/implement-tasks` (simple, effective) or `/orchestrate-tasks` (advanced, powerful) to start building!
```

## Step 6: Save Handoff

{{workflows/prompting/save-handoff}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that the tasks list is ALIGNED and DOES NOT CONFLICT with the user's preferences and standards as detailed in the following files:

{{standards/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
