Now that you've initialized the folder for this new spec, proceed with the research phase.

## Step 1: Extract Basepoints Knowledge (if basepoints exist)

Before researching requirements, extract relevant basepoints knowledge to inform the research process:

```bash
{{workflows/common/extract-basepoints-with-scope-detection}}
```

If basepoints exist, the extracted knowledge (`$EXTRACTED_KNOWLEDGE` and `$DETECTED_LAYER`) will be used to:
- Inform clarifying questions with existing patterns
- Suggest reusable patterns and modules for the detected abstraction layer
- Reference historical decisions and pros/cons
- Present common/reusable patterns to avoid unnecessary work

## Step 2: Research Spec Requirements

Follow these instructions for researching this spec's requirements:

{{workflows/specification/research-spec}}

## Display intermediate confirmation

Once you've completed your research and documented it, proceed to validation:

## Step 3: Run Validation

Before completing, run validation to ensure outputs are correct:

```bash
SPEC_PATH="agent-os/specs/[current-spec]"
COMMAND="shape-spec"
{{workflows/validation/validate-output-exists}}
{{workflows/validation/validate-knowledge-integration}}
{{workflows/validation/generate-validation-report}}
```

## Step 4: Generate Resource Checklist

Generate a checklist of all resources consulted:

```bash
{{workflows/basepoints/organize-and-cache-basepoints-knowledge}}
```

After all steps complete, inform the user:

```
Spec initialized successfully!

✅ Spec folder created: `[spec-path]`
✅ Requirements gathered
✅ Basepoints knowledge extracted: [Yes / No basepoints found]
✅ Detected layer: [LAYER or unknown]
✅ Visual assets: [Found X files / No files provided]
✅ Validation: [PASSED / WARNINGS]

👉 Run `/write-spec` to create the spec.md document.
```

## Step 5: Save Handoff

{{workflows/prompting/save-handoff}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your research questions and insights are ALIGNED and DOES NOT CONFLICT with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
