# Spec Implementation Process

Now that we have a spec and tasks list ready for implementation, we will proceed with implementation of this spec by following this multi-phase process:

PHASE 1: Determine which task group(s) from tasks.md should be implemented
PHASE 2: Select appropriate specialist agent for the task layer
PHASE 3: Delegate implementation to the selected specialist
PHASE 4: After ALL task groups have been implemented, delegate to implementation-verifier to produce the final verification report.

Follow each of these phases and their individual workflows IN SEQUENCE:

## Multi-Phase Process

### PHASE 1: Determine which task group(s) to implement

First, check if the user has already provided instructions about which task group(s) to implement.

**If the user HAS provided instructions:** Proceed to PHASE 2 to select the appropriate specialist.

**If the user has NOT provided instructions:**

Read `agent-os/specs/[this-spec]/tasks.md` to review the available task groups, then output the following message to the user and WAIT for their response:

```
Should we proceed with implementation of all task groups in tasks.md?

If not, then please specify which task(s) to implement.
```

### PHASE 2: Select appropriate specialist agent

For each task group to implement, determine the best specialist agent:

**Option A: Use orchestration.yml if available**

```bash
ORCHESTRATION_FILE="agent-os/specs/[this-spec]/orchestration.yml"

if [ -f "$ORCHESTRATION_FILE" ]; then
    # Extract assigned specialist for this task group
    ASSIGNED_AGENT=$(grep -A2 "name: $TASK_GROUP_NAME" "$ORCHESTRATION_FILE" | \
        grep "claude_code_subagent:" | \
        sed 's/.*claude_code_subagent: //')
    
    if [ -n "$ASSIGNED_AGENT" ]; then
        SELECTED_AGENT="$ASSIGNED_AGENT"
        echo "✅ Using pre-assigned specialist: $SELECTED_AGENT"
    fi
fi
```

**Option B: Auto-detect layer if no assignment exists**

```bash
if [ -z "$SELECTED_AGENT" ]; then
    # Use layer detection workflow
    {{workflows/scope-detection/detect-task-layer}}
    
    # Get task group content
    TASK_CONTENT=$(get_task_group_content "$TASK_GROUP_NAME")
    
    # Detect layer
    LAYER_RESULT=$(detect_layer "$TASK_CONTENT")
    DETECTED_LAYER=$(echo "$LAYER_RESULT" | cut -d':' -f1)
    
    # Get specialist for layer
    SELECTED_AGENT=$(get_agent_for_layer "$DETECTED_LAYER")
    
    echo "Auto-detected layer: $DETECTED_LAYER"
    echo "Selected specialist: $SELECTED_AGENT"
fi
```

**Option C: Fall back to generic implementer**

```bash
if [ -z "$SELECTED_AGENT" ] || [ ! -f "agent-os/agents/specialists/${SELECTED_AGENT}.md" ]; then
    SELECTED_AGENT="implementer"
    echo "Using generic implementer agent"
fi
```

### PHASE 3: Delegate implementation to the selected specialist

Delegate to the **$SELECTED_AGENT** subagent to implement the specified task group(s):

Provide to the subagent:
- The specific task group(s) from `agent-os/specs/[this-spec]/tasks.md` including the parent task, all sub-tasks, and any sub-bullet points
- The path to this spec's documentation: `agent-os/specs/[this-spec]/spec.md`
- The path to this spec's requirements: `agent-os/specs/[this-spec]/planning/requirements.md`
- The path to this spec's visuals (if any): `agent-os/specs/[this-spec]/planning/visuals`
- The detected layer context (if layer specialist): "This task targets the **$DETECTED_LAYER** layer"

Instruct the subagent to:
1. Analyze the provided spec.md, requirements.md, and visuals (if any)
2. Analyze patterns in the codebase according to its built-in workflow
3. **If layer specialist**: Focus on layer-specific patterns from basepoints
4. Implement the assigned task group according to requirements and standards
5. Update `agent-os/specs/[this-spec]/tasks.md` to mark completed tasks with `- [x]`

**Layer-specific instructions** (include when using layer specialists):

| Specialist | Additional Instructions |
|------------|------------------------|
| ui-specialist | Focus on component patterns, styling conventions, accessibility |
| api-specialist | Follow API conventions, error handling patterns, auth flows |
| data-specialist | Use data model patterns, migration conventions, query patterns |
| platform-specialist | Follow platform-specific guidelines, native patterns |
| test-specialist | Use testing conventions, coverage requirements, mock patterns |

### PHASE 3: Produce the final verification report

IF ALL task groups in tasks.md are marked complete with `- [x]`, then proceed with this step.  Otherwise, return to PHASE 1.

Assuming all tasks are marked complete, then delegate to the **implementation-verifier** subagent to do its implementation verification and produce its final verification report.

Provide to the subagent the following:
- The path to this spec: `agent-os/specs/[this-spec]`
Instruct the subagent to do the following:
  1. Run all of its final verifications according to its built-in workflow
  2. Produce the final verification report in `agent-os/specs/[this-spec]/verifications/final-verification.md`.
