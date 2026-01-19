Now that you have the spec.md AND/OR requirements.md, proceed with creating the tasks list.

## Step 1: Load Accumulated Knowledge from write-spec

First, load any accumulated knowledge from the previous write-spec command:

```bash
SPEC_PATH="geist/specs/[current-spec]"
CACHE_PATH="$SPEC_PATH/implementation/cache"

# Load accumulated knowledge from write-spec (which includes shape-spec knowledge)
if [ -f "$CACHE_PATH/accumulated-knowledge.md" ]; then
    PREVIOUS_KNOWLEDGE=$(cat "$CACHE_PATH/accumulated-knowledge.md")
    echo "✅ Loaded accumulated knowledge from previous commands"
else
    PREVIOUS_KNOWLEDGE=""
    echo "ℹ️ No accumulated knowledge found (shape-spec/write-spec may not have been run)"
fi
```

## Step 2: Extract Basepoints Knowledge (if basepoints exist)

Extract relevant basepoints knowledge to inform task creation:

```bash
{{workflows/common/extract-basepoints-with-scope-detection}}
```

If basepoints exist, the extracted knowledge (`$EXTRACTED_KNOWLEDGE`, `$LIBRARY_KNOWLEDGE`, and `$DETECTED_LAYER`) will be used to:
- Inform task breakdown with existing patterns
- Suggest existing patterns and checkpoints from basepoints
- Reference project-specific implementation strategies
- Include relevant testing approaches in task creation
- **Use library workflows and patterns to inform task structure**
- **Include library-specific implementation guidance in tasks**

## Step 2.5: Apply Narrow Focus + Expand Knowledge Strategy

Apply the context enrichment strategy for create-tasks, building upon previous knowledge:

```bash
echo "🎯 Applying narrow focus + expand knowledge strategy..."

# NARROW FOCUS: Scope to task breakdown
# - Focus on breaking down spec into implementable tasks
# - Filter knowledge to what's relevant for task structure

# EXPAND KNOWLEDGE: Build upon previous knowledge
# 1. Previous accumulated knowledge (from shape-spec + write-spec)
# 2. Fresh basepoints knowledge extraction
# 3. Fresh library basepoints knowledge
# 4. Product documentation (tech stack for implementation context)

# Load tech stack for implementation context
TECH_STACK=""
if [ -f "geist/product/tech-stack.md" ]; then
    TECH_STACK=$(cat geist/product/tech-stack.md)
fi

# Combine all knowledge sources (building upon previous)
ENRICHED_CONTEXT="
# Enriched Context for Create-Tasks

## Previous Knowledge (from shape-spec + write-spec)
$PREVIOUS_KNOWLEDGE

## Current Basepoints Knowledge
$EXTRACTED_KNOWLEDGE

## Library Workflows and Patterns
$LIBRARY_KNOWLEDGE

## Tech Stack (for implementation context)
$TECH_STACK

## Detected Abstraction Layer
$DETECTED_LAYER
"

echo "✅ Context enriched (building upon shape-spec + write-spec):"
echo "   - Previous accumulated knowledge"
echo "   - Current basepoints patterns and standards"
echo "   - Library workflows and implementation patterns"
echo "   - Tech stack for implementation context"
echo "   - Detected abstraction layer: $DETECTED_LAYER"
```

**Use the enriched context to:**
- Structure tasks based on library workflows and patterns
- Include library-specific implementation guidance in task descriptions
- Reference existing patterns from basepoints in task hints
- Ensure tasks align with tech stack capabilities

## Step 3: Check for Trade-offs (if needed)

Before creating tasks, check if trade-offs need to be reviewed:

```bash
{{workflows/human-review/review-trade-offs}}
```

## Step 4: Create Tasks List

Break down the spec and requirements into an actionable tasks list with strategic grouping and ordering, by following these instructions:

{{workflows/implementation/create-tasks-list}}

## Step 5: Run Validation

Before completing, run validation to ensure outputs are correct:

```bash
SPEC_PATH="geist/specs/[current-spec]"
COMMAND="create-tasks"
{{workflows/validation/validate-output-exists}}
{{workflows/validation/validate-knowledge-integration}}
{{workflows/validation/generate-validation-report}}
```

## Step 6: Generate Resource Checklist

Generate a checklist of all resources consulted:

```bash
{{workflows/basepoints/organize-and-cache-basepoints-knowledge}}
```

## Step 6.5: Feedback Checkpoint

After tasks.md is created, pause for user feedback before proceeding:

```bash
# Prepare feedback checkpoint
CURRENT_COMMAND="create-tasks"
OUTPUT_SUMMARY="
Tasks list created!

✅ Tasks breakdown complete at: \`$SPEC_PATH/tasks.md\`
✅ Basepoints knowledge used: [Yes / No basepoints found]
✅ Implementation hints included: [Yes / No patterns found]
✅ Validation: [PASSED / WARNINGS]

Tasks Summary:
[AI: List task group names and counts]

Total Tasks: [count]
Estimated Complexity: [Low/Medium/High]

Files Created:
- $SPEC_PATH/tasks.md
- $SPEC_PATH/implementation/cache/resources-consulted.md
"
NEXT_STEP="/implement-tasks (simple) or /orchestrate-tasks (advanced) to start building"
ALLOW_UPDATE=true

# Invoke feedback checkpoint
{{workflows/human-review/feedback-checkpoint}}
```

**IMPORTANT**: 
- **STOP and WAIT** for user response
- If user provides feedback, update the tasks.md and re-display
- Only proceed to next step when user confirms

---

After user confirms to continue:

```
The tasks list has been created at `geist/specs/[this-spec]/tasks.md`.

✅ Tasks breakdown complete
✅ Basepoints knowledge used: [Yes / No basepoints found]
✅ Implementation hints included: [Yes / No patterns found]
✅ Validation: [PASSED / WARNINGS]
✅ Resources consulted: See `implementation/cache/resources-consulted.md`

Review it closely to make sure it all looks good.

👉 Run `/implement-tasks` (simple, effective) or `/orchestrate-tasks` (advanced, powerful) to start building!
```

## Step 7: Accumulate Knowledge for Subsequent Commands

Accumulate the enriched knowledge for subsequent commands (orchestrate-tasks, implement-tasks):

```bash
# Set variables for knowledge accumulation
CURRENT_COMMAND="create-tasks"
KNOWLEDGE_SCOPE="files"    # Keep file-level knowledge from write-spec
KNOWLEDGE_DEPTH="code"     # Add code implementation patterns

NEW_BASEPOINTS_KNOWLEDGE="$EXTRACTED_KNOWLEDGE"
NEW_LIBRARY_KNOWLEDGE="$LIBRARY_KNOWLEDGE"
NEW_PRODUCT_KNOWLEDGE="$TECH_STACK"
NEW_CODE_KNOWLEDGE="$CODE_PATTERNS"  # Code patterns identified during task breakdown

# Accumulate knowledge with progressive refinement (builds upon shape-spec + write-spec knowledge)
{{workflows/common/accumulate-knowledge}}

echo "✅ Knowledge accumulated for subsequent commands"
echo "   Scope: $KNOWLEDGE_SCOPE | Depth: $KNOWLEDGE_DEPTH"
echo "   Next command (orchestrate-tasks or implement-tasks) will build upon this enriched context"
```

## Step 8: Save Handoff

{{workflows/prompting/save-handoff}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that the tasks list is ALIGNED and DOES NOT CONFLICT with the user's preferences and standards as detailed in the following files:

{{standards/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
