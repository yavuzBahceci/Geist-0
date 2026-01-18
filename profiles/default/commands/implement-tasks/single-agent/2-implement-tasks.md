Now that you have the task group(s) to be implemented, proceed with implementation.

## Step 1: Load Accumulated Knowledge

First, load any accumulated knowledge from previous commands (shape-spec, write-spec, create-tasks):

```bash
SPEC_PATH="agent-os/specs/[current-spec]"
CACHE_PATH="$SPEC_PATH/implementation/cache"

# Load accumulated knowledge from previous commands
if [ -f "$CACHE_PATH/accumulated-knowledge.md" ]; then
    PREVIOUS_KNOWLEDGE=$(cat "$CACHE_PATH/accumulated-knowledge.md")
    echo "✅ Loaded accumulated knowledge from previous commands"
else
    PREVIOUS_KNOWLEDGE=""
    echo "ℹ️ No accumulated knowledge found"
fi
```

## Step 2: Extract Comprehensive Knowledge

Extract knowledge from all sources to inform implementation:

```bash
{{workflows/common/extract-basepoints-with-scope-detection}}
```

If basepoints exist, the extracted knowledge (`$EXTRACTED_KNOWLEDGE`, `$LIBRARY_KNOWLEDGE`, and `$DETECTED_LAYER`) will be used to:
- Guide implementation with extracted patterns
- Suggest reusable code and modules during implementation
- Reference project-specific standards and coding patterns
- Use extracted testing approaches for test writing
- **Apply library-specific patterns and best practices**
- **Leverage library capabilities and avoid constraints**
- **Use library troubleshooting guidance when issues arise**

## Step 2.5: Create Comprehensive Implementation Prompt

Create a comprehensive implementation prompt with all extracted knowledge:

```bash
echo "📝 Creating comprehensive implementation prompt..."

# Load product knowledge
PRODUCT_KNOWLEDGE=""
if [ -f "agent-os/product/mission.md" ]; then
    PRODUCT_KNOWLEDGE="${PRODUCT_KNOWLEDGE}\n## Mission\n$(cat agent-os/product/mission.md)"
fi
if [ -f "agent-os/product/tech-stack.md" ]; then
    PRODUCT_KNOWLEDGE="${PRODUCT_KNOWLEDGE}\n## Tech Stack\n$(cat agent-os/product/tech-stack.md)"
fi

# Create comprehensive implementation prompt
IMPLEMENTATION_PROMPT="
# Comprehensive Implementation Context

## Task Requirements
[Task group requirements from tasks.md]

## Previous Accumulated Knowledge
$PREVIOUS_KNOWLEDGE

## Basepoints Patterns and Standards
$EXTRACTED_KNOWLEDGE

## Library Knowledge
### Patterns and Workflows
$LIBRARY_KNOWLEDGE

### Library Capabilities
[Extract relevant library capabilities for this task]

### Library Constraints
[Extract relevant library constraints to consider]

### Best Practices
[Extract library-specific best practices]

### Troubleshooting Guidance
[Extract relevant troubleshooting guidance]

## Product Context
$PRODUCT_KNOWLEDGE

## Detected Abstraction Layer
$DETECTED_LAYER

## Implementation Approach Decision

Based on the above knowledge, the recommended implementation approach is:

### Approach Rationale
[Document why this approach was chosen based on:]
- Available patterns from basepoints
- Product requirements and constraints
- Library capabilities and workflows
- Codebase structure and existing patterns

### Implementation Steps
[Outline the implementation steps based on the chosen approach]

### Patterns to Apply
[List specific patterns from basepoints and library basepoints to apply]

### Potential Issues to Watch For
[List potential issues based on library troubleshooting guidance]
"

echo "✅ Comprehensive implementation prompt created"
echo "   - Task requirements included"
echo "   - Basepoints patterns included"
echo "   - Library knowledge included"
echo "   - Product context included"
echo "   - Implementation approach decided"
```

## Step 2.6: Decide on Best Implementation Approach

Decide on the best implementation approach based on available knowledge:

```bash
echo "🎯 Deciding on best implementation approach..."

# Analyze available patterns
# 1. Check basepoints for relevant patterns
# 2. Check library basepoints for library-specific patterns
# 3. Check product constraints
# 4. Check codebase structure

# Decision criteria:
# - Pattern availability: Use existing patterns when available
# - Library capabilities: Leverage library features appropriately
# - Product alignment: Ensure approach aligns with product goals
# - Codebase consistency: Maintain consistency with existing code

# Document decision rationale
DECISION_RATIONALE="
## Implementation Approach Decision

### Selected Approach
[Describe the chosen approach]

### Rationale
- **Pattern Match**: [Which patterns from basepoints apply]
- **Library Fit**: [How library capabilities support this approach]
- **Product Alignment**: [How this aligns with product goals]
- **Codebase Consistency**: [How this maintains consistency]

### Alternative Approaches Considered
[List alternatives and why they were not chosen]
"

echo "$DECISION_RATIONALE" > "$CACHE_PATH/implementation-decision.md"
echo "✅ Implementation approach decided and documented"
```

## Step 3: Check for Trade-offs (if needed)

Before implementing, check if trade-offs need to be reviewed:

```bash
{{workflows/human-review/review-trade-offs}}
```

## Step 4: Check for Checkpoints (if needed)

Before implementing, check if checkpoints are needed for big changes:

```bash
{{workflows/human-review/create-checkpoint}}
```

## Step 5: Implement Tasks Using Enriched Prompt

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

## Step 6: Run Validation After Implementation

After each task group is implemented, run validation:

```bash
SPEC_PATH="agent-os/specs/[current-spec]"
COMMAND="implement-tasks"
{{workflows/validation/validate-output-exists}}
{{workflows/validation/validate-knowledge-integration}}
{{workflows/validation/validate-references}}
{{workflows/validation/generate-validation-report}}
```

## Step 7: Generate Resource Checklist

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

## Step 8: Save Handoff

{{workflows/prompting/save-handoff}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that the tasks list is ALIGNED and DOES NOT CONFLICT with the user's preferences and standards as detailed in the following files:

{{standards/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
