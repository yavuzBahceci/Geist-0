Now that we've initiated and planned the details for a new spec, we will now proceed with drafting the specification document.

## Step 1: Extract Basepoints Knowledge (if basepoints exist)

Before writing the specification, extract relevant basepoints knowledge to inform the spec writing:

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
    
    # Load extracted knowledge for use in spec writing
    if [ -f "$SPEC_PATH/implementation/cache/basepoints-knowledge.md" ]; then
        EXTRACTED_KNOWLEDGE=$(cat "$SPEC_PATH/implementation/cache/basepoints-knowledge.md")
    fi
    
    # Load detected layer
    if [ -f "$SPEC_PATH/implementation/cache/detected-layer.txt" ]; then
        DETECTED_LAYER=$(cat "$SPEC_PATH/implementation/cache/detected-layer.txt")
    fi
fi
```

If basepoints exist, the extracted knowledge will be used to:
- Suggest reusable patterns, modules, and code during spec creation
- Reference project-specific standards and flows in spec
- Include relevant testing approaches and strategies in spec
- Avoid unnecessary work by leveraging existing patterns

## Step 2: Write Specification

Follow these instructions for writing the specification:

{{workflows/specification/write-spec}}

## Step 3: Check for Trade-offs (if needed)

Before finalizing the spec, check if trade-offs need to be reviewed:

```bash
{{workflows/human-review/review-trade-offs}}
```

If trade-offs or contradictions are detected:
- Present options with pros/cons from basepoints
- Require explicit human selection before proceeding
- Log decisions to `$SPEC_PATH/implementation/cache/human-decisions.md`

## Step 4: Run Validation

Before completing, run validation to ensure outputs are correct:

```bash
SPEC_PATH="agent-os/specs/[current-spec]"
COMMAND="write-spec"
{{workflows/validation/validate-output-exists}}
{{workflows/validation/validate-knowledge-integration}}
{{workflows/validation/validate-references}}
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
The spec has been created at `agent-os/specs/[this-spec]/spec.md`.

✅ Spec document created
✅ Basepoints knowledge integrated: [Yes / No basepoints found]
✅ Detected layer: [LAYER or unknown]
✅ Trade-off review: [Completed / No trade-offs detected]
✅ Validation: [PASSED / WARNINGS]
✅ Resources consulted: See `implementation/cache/resources-consulted.md`

Review it closely to ensure everything aligns with your vision and requirements.

👉 Run `/create-tasks` to break down the spec into implementable tasks.
```

## Step 6: Save Handoff

{{workflows/prompting/save-handoff}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that the specification document's content is ALIGNED and DOES NOT CONFLICT with the user's preferences and standards as detailed in the following files:

{{standards/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
