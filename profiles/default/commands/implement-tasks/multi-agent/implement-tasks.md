# Spec Implementation Process

Now that we have a spec and tasks list ready for implementation, we will proceed with implementation of this spec by following this multi-phase process:

PHASE 1: Determine which task group(s) from tasks.md should be implemented
PHASE 2: Select appropriate specialist agent(s) for the task layer(s)
PHASE 3: Delegate implementation with combined specialist context
PHASE 4: After ALL task groups have been implemented, delegate to implementation-verifier to produce the final verification report.

Follow each of these phases and their individual workflows IN SEQUENCE:

## Multi-Phase Process

### PHASE 1: Determine which task group(s) to implement

First, check if the user has already provided instructions about which task group(s) to implement.

**If the user HAS provided instructions:** Proceed to PHASE 2 to select the appropriate specialist.

**If the user has NOT provided instructions:**

Read `geist/specs/[this-spec]/tasks.md` to review the available task groups, then output the following message to the user and WAIT for their response:

```
Should we proceed with implementation of all task groups in tasks.md?

If not, then please specify which task(s) to implement.
```

### PHASE 2: Select appropriate specialist agent(s)

For each task group to implement, determine the best specialist agent(s):

**Option A: Use orchestration.yml if available**

```bash
ORCHESTRATION_FILE="geist/specs/[this-spec]/orchestration.yml"

if [ -f "$ORCHESTRATION_FILE" ]; then
    # Extract assigned specialists for this task group (supports multiple)
    ASSIGNED_SPECIALISTS=$(grep -A3 "name: $TASK_GROUP_NAME" "$ORCHESTRATION_FILE" | \
        grep "specialists:" | \
        sed 's/.*specialists: \[//' | sed 's/\]//')
    
    if [ -n "$ASSIGNED_SPECIALISTS" ]; then
        # Parse comma-separated list into array
        IFS=',' read -ra SPECIALISTS_ARRAY <<< "$ASSIGNED_SPECIALISTS"
        echo "✅ Using assigned specialists: ${SPECIALISTS_ARRAY[*]}"
    fi
fi
```

**Option B: Auto-detect layer if no assignment exists**

```bash
if [ -z "$ASSIGNED_SPECIALISTS" ]; then
    # Use layer detection workflow
    {{workflows/scope-detection/detect-task-layer}}
    
    # Get task group content
    TASK_CONTENT=$(get_task_group_content "$TASK_GROUP_NAME")
    
    # Detect layer(s) - may return multiple for cross-layer tasks
    LAYER_RESULT=$(detect_layers "$TASK_CONTENT")
    DETECTED_LAYERS=$(echo "$LAYER_RESULT" | tr ',' ' ')
    
    # Get specialist(s) for detected layers
    SPECIALISTS_ARRAY=()
    for layer in $DETECTED_LAYERS; do
        SPECIALIST=$(get_agent_for_layer "$layer")
        SPECIALISTS_ARRAY+=("$SPECIALIST")
    done
    
    echo "Auto-detected layers: $DETECTED_LAYERS"
    echo "Selected specialists: ${SPECIALISTS_ARRAY[*]}"
fi
```

**Option C: Fall back to generic implementer**

```bash
if [ ${#SPECIALISTS_ARRAY[@]} -eq 0 ]; then
    SPECIALISTS_ARRAY=("implementer")
    echo "Using generic implementer agent"
fi
```

### PHASE 3: Delegate implementation with combined specialist context

**For single specialist:** Delegate directly to that specialist.

**For multiple specialists:** Inject context from ALL specialists into the primary implementer.

```bash
# Collect context from all assigned specialists
COMBINED_CONTEXT=""

for specialist in "${SPECIALISTS_ARRAY[@]}"; do
    SPECIALIST_FILE="geist/agents/specialists/${specialist}.md"
    
    if [ -f "$SPECIALIST_FILE" ]; then
        # Extract the specialist's expertise and patterns
        SPECIALIST_CONTEXT=$(cat "$SPECIALIST_FILE")
        COMBINED_CONTEXT="${COMBINED_CONTEXT}\n\n## Context from ${specialist}\n${SPECIALIST_CONTEXT}"
    fi
done
```

Delegate to the **primary specialist** (first in list) or **implementer** with combined context:

Provide to the subagent:
- The specific task group(s) from `geist/specs/[this-spec]/tasks.md` including the parent task, all sub-tasks, and any sub-bullet points
- The path to this spec's documentation: `geist/specs/[this-spec]/spec.md`
- The path to this spec's requirements: `geist/specs/[this-spec]/planning/requirements.md`
- The path to this spec's visuals (if any): `geist/specs/[this-spec]/planning/visuals`
- **Combined specialist context** (if multiple specialists assigned):

```
## Specialist Knowledge for This Task

This task spans multiple layers. Use the combined expertise below:

$COMBINED_CONTEXT

When implementing, ensure your solution:
- Follows patterns from ALL relevant specialists
- Maintains consistency across layer boundaries
- Uses appropriate conventions for each layer touched
```

Instruct the subagent to:
1. Analyze the provided spec.md, requirements.md, and visuals (if any)
2. Analyze patterns in the codebase according to its built-in workflow
3. **If multiple specialists**: Consider patterns from ALL assigned specialists
4. Implement the assigned task group according to requirements and standards
5. Update `geist/specs/[this-spec]/tasks.md` to mark completed tasks with `- [x]`

**Layer-specific instructions** (include relevant ones based on assigned specialists):

| Specialist | Focus Areas |
|------------|-------------|
| ui-specialist | Component patterns, styling conventions, accessibility |
| api-specialist | API conventions, error handling patterns, auth flows |
| data-specialist | Data model patterns, migration conventions, query patterns |
| platform-specialist | Platform-specific guidelines, native patterns |
| test-specialist | Testing conventions, coverage requirements, mock patterns |

### PHASE 4: Produce the final verification report

IF ALL task groups in tasks.md are marked complete with `- [x]`, then proceed with this step. Otherwise, return to PHASE 1.

Assuming all tasks are marked complete, then delegate to the **implementation-verifier** subagent to do its implementation verification and produce its final verification report.

Provide to the subagent the following:
- The path to this spec: `geist/specs/[this-spec]`
Instruct the subagent to do the following:
  1. Run all of its final verifications according to its built-in workflow
  2. Produce the final verification report in `geist/specs/[this-spec]/verifications/final-verification.md`.
