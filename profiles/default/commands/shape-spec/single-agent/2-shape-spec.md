Now that you've initialized the folder for this new spec, proceed with the research phase.

## Step 1: Extract Basepoints Knowledge (if basepoints exist)

Before researching requirements, extract relevant basepoints knowledge to inform the research process:

```bash
{{workflows/common/extract-basepoints-with-scope-detection}}
```

If basepoints exist, the extracted knowledge (`$EXTRACTED_KNOWLEDGE`, `$LIBRARY_KNOWLEDGE`, and `$DETECTED_LAYER`) will be used to:
- Inform clarifying questions with existing patterns
- Suggest reusable patterns and modules for the detected abstraction layer
- Reference historical decisions and pros/cons
- Present common/reusable patterns to avoid unnecessary work
- **Include library-specific capabilities and constraints** from library basepoints
- **Identify possible solutions** that libraries provide for the requirements

## Step 1.5: Apply Narrow Focus + Expand Knowledge Strategy

Apply the context enrichment strategy for shape-spec:

```bash
echo "🎯 Applying narrow focus + expand knowledge strategy..."

# NARROW FOCUS: Scope to specific spec requirements
# - Focus on the feature/requirements being shaped
# - Filter knowledge to what's relevant to this spec

# EXPAND KNOWLEDGE: Gather from multiple sources
# 1. Basepoints knowledge (already extracted)
# 2. Library basepoints knowledge (already extracted)
# 3. Product documentation

# Load product knowledge
PRODUCT_KNOWLEDGE=""
if [ -f "agent-os/product/mission.md" ]; then
    PRODUCT_KNOWLEDGE="${PRODUCT_KNOWLEDGE}\n## Mission\n$(cat agent-os/product/mission.md)"
fi
if [ -f "agent-os/product/roadmap.md" ]; then
    PRODUCT_KNOWLEDGE="${PRODUCT_KNOWLEDGE}\n## Roadmap\n$(cat agent-os/product/roadmap.md)"
fi
if [ -f "agent-os/product/tech-stack.md" ]; then
    PRODUCT_KNOWLEDGE="${PRODUCT_KNOWLEDGE}\n## Tech Stack\n$(cat agent-os/product/tech-stack.md)"
fi

# Combine all knowledge sources for context enrichment
ENRICHED_CONTEXT="
# Enriched Context for Shape-Spec

## Basepoints Knowledge
$EXTRACTED_KNOWLEDGE

## Library Capabilities and Constraints
$LIBRARY_KNOWLEDGE

## Product Context
$PRODUCT_KNOWLEDGE

## Detected Abstraction Layer
$DETECTED_LAYER
"

echo "✅ Context enriched with:"
echo "   - Basepoints patterns and standards"
echo "   - Library capabilities, constraints, and possible solutions"
echo "   - Product mission, roadmap, and tech stack"
echo "   - Detected abstraction layer: $DETECTED_LAYER"
```

**Use the enriched context to:**
- Identify library-specific solutions that could address the requirements
- Consider library constraints when shaping requirements
- Align requirements with product mission and roadmap
- Leverage existing patterns from basepoints

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

## Step 5: Accumulate Knowledge for Subsequent Commands

Accumulate the enriched knowledge for subsequent commands (write-spec, create-tasks, etc.):

```bash
# Set variables for knowledge accumulation
CURRENT_COMMAND="shape-spec"
NEW_BASEPOINTS_KNOWLEDGE="$EXTRACTED_KNOWLEDGE"
NEW_LIBRARY_KNOWLEDGE="$LIBRARY_KNOWLEDGE"
NEW_PRODUCT_KNOWLEDGE="$PRODUCT_KNOWLEDGE"

# Accumulate knowledge
{{workflows/common/accumulate-knowledge}}

echo "✅ Knowledge accumulated for subsequent commands"
echo "   Next command (write-spec) will build upon this enriched context"
```

## Step 6: Save Handoff

{{workflows/prompting/save-handoff}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your research questions and insights are ALIGNED and DOES NOT CONFLICT with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
