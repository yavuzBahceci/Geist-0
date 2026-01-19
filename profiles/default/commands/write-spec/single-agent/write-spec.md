Now that we've initiated and planned the details for a new spec, we will now proceed with drafting the specification document.

## Step 1: Load Accumulated Knowledge from shape-spec

First, load any accumulated knowledge from the previous shape-spec command:

```bash
SPEC_PATH="geist/specs/[current-spec]"
CACHE_PATH="$SPEC_PATH/implementation/cache"

# Load accumulated knowledge from shape-spec
if [ -f "$CACHE_PATH/accumulated-knowledge.md" ]; then
    PREVIOUS_KNOWLEDGE=$(cat "$CACHE_PATH/accumulated-knowledge.md")
    echo "✅ Loaded accumulated knowledge from shape-spec"
else
    PREVIOUS_KNOWLEDGE=""
    echo "ℹ️ No accumulated knowledge found (shape-spec may not have been run)"
fi
```

## Step 2: Extract Basepoints Knowledge (if basepoints exist)

Extract relevant basepoints knowledge to inform the spec writing:

```bash
{{workflows/common/extract-basepoints-with-scope-detection}}
```

If basepoints exist, the extracted knowledge (`$EXTRACTED_KNOWLEDGE`, `$LIBRARY_KNOWLEDGE`, and `$DETECTED_LAYER`) will be used to:
- Suggest reusable patterns, modules, and code during spec creation
- Reference project-specific standards and flows in spec
- Include relevant testing approaches and strategies in spec
- Avoid unnecessary work by leveraging existing patterns
- **Include library-specific patterns and best practices** in spec documentation
- **Document library workflows and constraints** relevant to the spec

## Step 2.5: Apply Narrow Focus + Expand Knowledge Strategy

Apply the context enrichment strategy for write-spec, building upon shape-spec's knowledge:

```bash
echo "🎯 Applying narrow focus + expand knowledge strategy..."

# NARROW FOCUS: Scope to spec writing
# - Focus on creating the specification document
# - Filter knowledge to what's relevant for documenting the spec

# EXPAND KNOWLEDGE: Build upon previous knowledge
# 1. Previous accumulated knowledge (from shape-spec)
# 2. Fresh basepoints knowledge extraction
# 3. Fresh library basepoints knowledge
# 4. Product documentation

# Load product knowledge
PRODUCT_KNOWLEDGE=""
if [ -f "geist/product/mission.md" ]; then
    PRODUCT_KNOWLEDGE="${PRODUCT_KNOWLEDGE}\n## Mission\n$(cat geist/product/mission.md)"
fi
if [ -f "geist/product/tech-stack.md" ]; then
    PRODUCT_KNOWLEDGE="${PRODUCT_KNOWLEDGE}\n## Tech Stack\n$(cat geist/product/tech-stack.md)"
fi

# Combine all knowledge sources (building upon previous)
ENRICHED_CONTEXT="
# Enriched Context for Write-Spec

## Previous Knowledge (from shape-spec)
$PREVIOUS_KNOWLEDGE

## Current Basepoints Knowledge
$EXTRACTED_KNOWLEDGE

## Library Patterns and Best Practices
$LIBRARY_KNOWLEDGE

## Product Context
$PRODUCT_KNOWLEDGE

## Detected Abstraction Layer
$DETECTED_LAYER
"

echo "✅ Context enriched (building upon shape-spec):"
echo "   - Previous accumulated knowledge"
echo "   - Current basepoints patterns and standards"
echo "   - Library patterns, best practices, and workflows"
echo "   - Product mission and tech stack"
echo "   - Detected abstraction layer: $DETECTED_LAYER"
```

**Use the enriched context to:**
- Include library-specific patterns and best practices in the spec
- Document library constraints and capabilities relevant to the spec
- Reference existing patterns from basepoints in the spec
- Ensure spec aligns with product mission and tech stack

## Step 3: Write Specification

Follow these instructions for writing the specification:

{{workflows/specification/write-spec}}

## Step 4: Check for Trade-offs (if needed)

Before finalizing the spec, check if trade-offs need to be reviewed:

```bash
{{workflows/human-review/review-trade-offs}}
```

If trade-offs or contradictions are detected:
- Present options with pros/cons from basepoints
- Require explicit human selection before proceeding
- Log decisions to `$SPEC_PATH/implementation/cache/human-decisions.md`

## Step 5: Run Validation

Before completing, run validation to ensure outputs are correct:

```bash
SPEC_PATH="geist/specs/[current-spec]"
COMMAND="write-spec"
{{workflows/validation/validate-output-exists}}
{{workflows/validation/validate-knowledge-integration}}
{{workflows/validation/validate-references}}
{{workflows/validation/generate-validation-report}}
```

## Step 6: Generate Resource Checklist

Generate a checklist of all resources consulted:

```bash
{{workflows/basepoints/organize-and-cache-basepoints-knowledge}}
```

## Step 6.5: Feedback Checkpoint

After spec.md is created, pause for user feedback before proceeding:

```bash
# Prepare feedback checkpoint
CURRENT_COMMAND="write-spec"
OUTPUT_SUMMARY="
Spec document created!

✅ Spec document created at: \`$SPEC_PATH/spec.md\`
✅ Basepoints knowledge integrated: [Yes / No basepoints found]
✅ Detected layer: $DETECTED_LAYER
✅ Trade-off review: [Completed / No trade-offs detected]
✅ Validation: [PASSED / WARNINGS]

Spec Summary:
[AI: Provide brief summary of spec goals and scope]

Key Features:
[AI: List 3-5 key features from the spec]

Files Created/Updated:
- $SPEC_PATH/spec.md
- $SPEC_PATH/implementation/cache/resources-consulted.md
"
NEXT_STEP="/create-tasks to break down the spec into implementable tasks"
ALLOW_UPDATE=true

# Invoke feedback checkpoint
{{workflows/human-review/feedback-checkpoint}}
```

**IMPORTANT**: 
- **STOP and WAIT** for user response
- If user provides feedback, update the spec.md and re-display
- Only proceed to next step when user confirms

---

After user confirms to continue:

```
The spec has been created at `geist/specs/[this-spec]/spec.md`.

✅ Spec document created
✅ Basepoints knowledge integrated: [Yes / No basepoints found]
✅ Detected layer: [LAYER or unknown]
✅ Trade-off review: [Completed / No trade-offs detected]
✅ Validation: [PASSED / WARNINGS]
✅ Resources consulted: See `implementation/cache/resources-consulted.md`

Review it closely to ensure everything aligns with your vision and requirements.

👉 Run `/create-tasks` to break down the spec into implementable tasks.
```

## Step 7: Accumulate Knowledge for Subsequent Commands

Accumulate the enriched knowledge for subsequent commands (create-tasks, orchestrate-tasks, etc.):

```bash
# Set variables for knowledge accumulation
CURRENT_COMMAND="write-spec"
KNOWLEDGE_SCOPE="architecture"  # Keep only architecture-related from shape-spec
KNOWLEDGE_DEPTH="files"         # Add file patterns/standards (no code yet)

NEW_BASEPOINTS_KNOWLEDGE="$EXTRACTED_KNOWLEDGE"
NEW_LIBRARY_KNOWLEDGE="$LIBRARY_KNOWLEDGE"
NEW_PRODUCT_KNOWLEDGE="$PRODUCT_KNOWLEDGE"

# Accumulate knowledge with progressive refinement (builds upon shape-spec's knowledge)
{{workflows/common/accumulate-knowledge}}

echo "✅ Knowledge accumulated for subsequent commands"
echo "   Scope: $KNOWLEDGE_SCOPE | Depth: $KNOWLEDGE_DEPTH"
echo "   Next command (create-tasks) will build upon this enriched context"
```

## Step 8: Save Handoff

{{workflows/prompting/save-handoff}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that the specification document's content is ALIGNED and DOES NOT CONFLICT with the user's preferences and standards as detailed in the following files:

{{standards/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
