# Process for Orchestrating a Spec's Implementation

Now that we have a spec and tasks list ready for implementation, we will proceed with orchestrating the implementation by creating prompts for each task group.

**Important:** This command generates implementation prompts - it does NOT execute implementation. Implementation happens separately when the generated prompt files are executed.

Follow each of these phases and their individual workflows IN SEQUENCE:

## Multi-Phase Process

### FIRST: Extract Basepoints Knowledge for Orchestration

Before orchestrating tasks, extract basepoints knowledge to inform sub-agent context:

```bash
{{workflows/common/extract-basepoints-with-scope-detection}}
```

If basepoints exist, the extracted knowledge (`$EXTRACTED_KNOWLEDGE` and `$DETECTED_LAYER`) will be used to:
- Match tasks to relevant module basepoints
- Inject basepoint knowledge into sub-agent prompts
- Provide coding patterns and standards context per task group
- Enable sub-agents to leverage project-specific patterns

### NEXT: Get tasks.md for this spec

IF you already know which spec we're working on and IF that spec folder has a `tasks.md` file, then use that and skip to the NEXT phase.

IF you don't already know which spec we're working on and IF that spec folder doesn't yet have a `tasks.md` THEN output the following request to the user:

```
Please point me to a spec's `tasks.md` that you want to orchestrate implementation for.

If you don't have one yet, then run any of these commands first:
/shape-spec
/write-spec
/create-tasks
```

### NEXT: Create orchestration.yml to serve as a roadmap for orchestration of task groups

In this spec's folder, create this file: `agent-os/specs/[this-spec]/orchestration.yml`.

Populate this file with with the names of each task group found in this spec's `tasks.md` and use this EXACT structure for the content of `orchestration.yml`:

```yaml
task_groups:
  - name: [task-group-name]
  - name: [task-group-name]
  - name: [task-group-name]
  # Repeat for each task group found in tasks.md
```

{{IF use_claude_code_subagents}}
### NEXT: Auto-detect layer specialists for each task group

First, load the specialist registry and analyze each task group to suggest appropriate layer specialists:

```bash
# Load specialist registry if it exists
SPECIALIST_REGISTRY="agent-os/agents/specialists/registry.yml"
SPECIALISTS_AVAILABLE="false"

if [ -f "$SPECIALIST_REGISTRY" ]; then
    SPECIALISTS_AVAILABLE="true"
    echo "✅ Layer specialists available"
    
    # Extract available specialists
    AVAILABLE_SPECIALISTS=$(grep -A1 "^  - name:" "$SPECIALIST_REGISTRY" | grep "name:" | sed 's/.*name: //')
    echo "Available specialists:"
    echo "$AVAILABLE_SPECIALISTS"
fi

# Also check for standard agents
STANDARD_AGENTS=$(find agent-os/agents -maxdepth 1 -name "*.md" -type f | xargs -I{} basename {} .md)
echo "Standard agents: $STANDARD_AGENTS"
```

For each task group in `tasks.md`, analyze the task content to detect which abstraction layer it targets:

```bash
# Layer detection keywords (from registry or defaults)
declare -A LAYER_KEYWORDS
LAYER_KEYWORDS[ui]="component view screen button form modal layout style css render widget"
LAYER_KEYWORDS[api]="endpoint route controller handler request response middleware api rest graphql"
LAYER_KEYWORDS[data]="model schema migration query database repository entity storage persistence"
LAYER_KEYWORDS[platform]="ios android native device system config platform infrastructure"
LAYER_KEYWORDS[test]="test spec mock fixture coverage assertion expect describe"

# For each task group, count keyword matches to suggest layer
detect_task_layer() {
    local task_content="$1"
    local task_lower=$(echo "$task_content" | tr '[:upper:]' '[:lower:]')
    
    local best_layer=""
    local best_score=0
    
    for layer in "${!LAYER_KEYWORDS[@]}"; do
        local score=0
        for keyword in ${LAYER_KEYWORDS[$layer]}; do
            if echo "$task_lower" | grep -q "$keyword"; then
                ((score++))
            fi
        done
        
        if [ $score -gt $best_score ]; then
            best_score=$score
            best_layer=$layer
        fi
    done
    
    if [ $best_score -gt 0 ]; then
        echo "${best_layer}-specialist"
    else
        echo "implementer"  # Default to generic implementer
    fi
}

# Generate suggestions for each task group
SUGGESTIONS=""
TASK_NUM=1
while read -r task_group; do
    SUGGESTED_AGENT=$(detect_task_layer "$task_group")
    SUGGESTIONS="${SUGGESTIONS}${TASK_NUM}. ${task_group} → ${SUGGESTED_AGENT}\n"
    ((TASK_NUM++))
done < <(grep -E "^## Task Group|^### " agent-os/specs/[this-spec]/tasks.md | head -20)
```

Present auto-detected suggestions to user for confirmation:

```
I've analyzed the task groups and detected these layer-specialist suggestions:

$SUGGESTIONS

Available specialists:
- ui-specialist (UI/frontend layer)
- api-specialist (API/backend layer)  
- data-specialist (Data/database layer)
- platform-specialist (Platform/infrastructure layer)
- test-specialist (Testing layer)
- implementer (Generic, cross-layer tasks)

Would you like to:
1. Accept these suggestions
2. Modify some assignments (specify which)
3. Assign manually for all task groups

Reply with your choice (1/2/3) or specify modifications like "2: change api-specialist, 4: change ui-specialist"
```

Using the user's responses, update `orchestration.yml` to specify the agent names. The file should look like:

```yaml
task_groups:
  - name: [task-group-name]
    claude_code_subagent: [specialist-name]
    detected_layer: [layer]  # For reference
  - name: [task-group-name]
    claude_code_subagent: [specialist-name]
    detected_layer: [layer]
  # Repeat for each task group
```

For example, after this step with layer detection:

```yaml
task_groups:
  - name: user-profile-ui
    claude_code_subagent: ui-specialist
    detected_layer: ui
  - name: profile-api-endpoints
    claude_code_subagent: api-specialist
    detected_layer: api
  - name: user-data-model
    claude_code_subagent: data-specialist
    detected_layer: data
  - name: cross-layer-integration
    claude_code_subagent: implementer
    detected_layer: mixed
```

### FALLBACK: Manual assignment if no specialists

If layer specialists don't exist (user hasn't run `/deploy-agents` or `/create-basepoints`), fall back to manual assignment:

```
Please specify the name of each subagent to be assigned to each task group:

1. [task-group-name]
2. [task-group-name]
3. [task-group-name]
[repeat for each task-group you've added to orchestration.yml]

Available agents:
[list agents from agent-os/agents/]

Simply respond with the subagent names and corresponding task group number.
```
{{ENDIF use_claude_code_subagents}}

{{UNLESS standards_as_claude_code_skills}}
### NEXT: Ask user to assign standards to each task group

Next we must determine which standards should guide the implementation of each task group.  Ask the user to provide this info using the following request to user and WAIT for user's response:

```
Please specify the standard(s) that should be used to guide the implementation of each task group:

1. [task-group-name]
2. [task-group-name]
3. [task-group-name]
[repeat for each task-group you've added to orchestration.yml]

For each task group number, you can specify any combination of the following:

"all" to include all of your standards
"global/*" to include all of the files inside of standards/global
"global/coding-style.md" to include the coding-style.md standard file
"none" to include no standards for this task group.
```

Using the user's responses, update `orchestration.yml` to specify those standards for each task group.  `orchestration.yml` should end up having AT LEAST the following information added to it:

```yaml
task_groups:
  - name: [task-group-name]
    standards:
      - [users' 1st response for this task group]
      - [users' 2nd response for this task group]
      - [users' 3rd response for this task group]
      # Repeat for all standards that the user specified for this task group
  - name: [task-group-name]
    standards:
      - [users' 1st response for this task group]
      - [users' 2nd response for this task group]
      # Repeat for all standards that the user specified for this task group
  # Repeat for each task group found in tasks.md
```

For example, after this step, the `orchestration.yml` file might look like this (exact names will vary):

```yaml
task_groups:
  - name: core-functionality
    standards:
      - all
  - name: user-interface
    standards:
      - global/*
      - quality/assurance.md
  - name: task-group-with-no-standards
  - name: interface-implementation
    standards:
      - global/*
      - global/error-handling.md
```

Note: If the `use_claude_code_subagents` flag is enabled, the final `orchestration.yml` would include BOTH `claude_code_subagent` assignments AND `standards` for each task group.
{{ENDUNLESS standards_as_claude_code_skills}}

{{IF use_claude_code_subagents}}
### NEXT: Delegate task groups implementations to assigned subagents

Loop through each task group in `agent-os/specs/[this-spec]/tasks.md` and delegate its implementation to the assigned subagent specified in `orchestration.yml`.

For each delegation, provide the subagent with:
- The task group (including the parent task and all sub-tasks)
- The spec file: `agent-os/specs/[this-spec]/spec.md`
- Instruct subagent to:
  - Perform their implementation
  - Check off the task and sub-task(s) in `agent-os/specs/[this-spec]/tasks.md`
{{UNLESS standards_as_claude_code_skills}}

In addition to the above items, also instruct the subagent to closely adhere to the user's standards & preferences as specified in the following files.  To build the list of file references to give to the subagent, follow these instructions:

{{workflows/implementation/compile-implementation-standards}}

Provide all of the above to the subagent when delegating tasks for it to implement.
{{ENDUNLESS standards_as_claude_code_skills}}
{{ENDIF use_claude_code_subagents}}

{{UNLESS use_claude_code_subagents}}
### NEXT: Generate prompts

Now we must generate an ordered series of prompt texts, which will be used to direct the implementation of each task group listed in `orchestration.yml`.

Follow these steps to generate this spec's ordered series of prompts texts, each in its own .md file located in `agent-os/specs/[this-spec]/implementation/prompts/`.

LOOP through EACH task group in `agent-os/specs/[this-spec]/tasks.md` and for each, use the following workflow to generate a markdown file with prompt text for each task group:

#### Step 1. Create the prompt markdown file

Create the prompt markdown file using this naming convention:
`agent-os/specs/[this-spec]/implementation/prompts/[task-group-number]-[task-group-title].md`.

For example, if the 3rd task group in tasks.md is named "Comment System" then create `3-comment-system.md`.

#### Step 2. Populate the prompt file

Populate the prompt markdown file using the following Prompt file content template.

##### Bracket content replacements

In the content template below, replace "[spec-title]" and "[this-spec]" with the current spec's title, and "[task-group-number]" with the current task group's number.

{{UNLESS standards_as_claude_code_skills}}
To replace "[orchestrated-standards]", use the following workflow:

{{workflows/implementation/compile-implementation-standards}}
{{ENDUNLESS standards_as_claude_code_skills}}

#### Prompt file content template:

```markdown
# Task Group [task-group-number]: [task-group-name]

**Spec:** [spec-title]
**Scope:** This prompt covers ONLY Task Group [task-group-number]. Do NOT implement other task groups.

---

## Task to Implement

[paste entire task group including parent task, all of its' sub-tasks, and sub-bullet points]

---

## Context

Read these files to understand the context:
- @agent-os/specs/[this-spec]/spec.md
- @agent-os/specs/[this-spec]/planning/requirements.md
- @agent-os/specs/[this-spec]/planning/visuals (if exists)

## Basepoints Knowledge Context

[IF basepoints-knowledge.md exists for this spec, include relevant patterns and standards for this task group:]

The following patterns and standards from basepoints are relevant to this task group:

[Extract and include relevant sections from $SPEC_PATH/implementation/cache/basepoints-knowledge.md that match this task group's domain]

Use these patterns to guide your implementation and ensure consistency with existing codebase conventions.

## Library Basepoints Knowledge

[IF library-basepoints-knowledge.md exists for this spec, include relevant library knowledge for this task group:]

The following library patterns, workflows, and best practices are relevant to this task group:

[Extract and include relevant sections from $SPEC_PATH/implementation/cache/library-basepoints-knowledge.md that match this task group's domain]

**Library Capabilities:** [List relevant library capabilities for this task group]
**Library Constraints:** [List relevant library constraints to consider]
**Best Practices:** [Include library-specific best practices]
**Troubleshooting:** [Include relevant troubleshooting guidance]

Use this library knowledge to leverage library capabilities and avoid common pitfalls.

---

## Implementation Instructions

1. Implement ONLY the tasks listed above (Task Group [task-group-number])
2. Mark each completed task with `[x]` in `tasks.md`
3. Follow the workflow: {{workflows/implementation/implement-tasks}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your implementation work is ALIGNED and DOES NOT CONFLICT with the user's preferences and standards as detailed in:

[orchestrated-standards]
{{ENDUNLESS standards_as_claude_code_skills}}

---

## ✅ Completion and Validation

When you have completed ALL tasks in Task Group [task-group-number]:

### Step 1: Run Implementation Validation

Before marking tasks complete, run deterministic validation:

```bash
SPEC_PATH="agent-os/specs/[this-spec]"
{{workflows/validation/validate-implementation}}
```

This runs project-specific validators (build, test, lint, type-check) that were configured during deployment.

**IF validation FAILED:**
- Present failures to user
- Wait for fixes or user override
- Do NOT proceed until resolved

**IF validation PASSED or WARNINGS:**
- Proceed to Step 2

### Step 2: Mark Completion
1. ✅ Mark all tasks as complete `[x]` in `tasks.md`
2. ✅ Add ✅ marker to this task group header in `tasks.md`
3. ✅ Provide a brief summary of what was implemented

### Step 4: Check If Human Review Needed

Run the review check workflow:

```bash
SPEC_PATH="agent-os/specs/[this-spec]"
{{workflows/human-review/review-trade-offs}}
```

This will automatically:
- Detect any trade-offs or contradictions in your implementation
- If issues found → Present them and WAIT for user decision
- If no issues → Output "✅ No human review needed. Proceeding automatically."

### Step 5: Continue Based on Review Result

**IF review detected issues (REVIEW_TRIGGERED=true):**
- Present the trade-offs/contradictions to user
- Wait for user decision before proceeding
- Once user approves, continue to Step 6

**IF no review needed (REVIEW_TRIGGERED=false):**
- Proceed directly to Step 6

### Step 6: Proceed to Next Prompt

👉 Read and execute the next prompt: `[next-prompt-number]-[next-prompt-name].md`
   (Located in: `agent-os/specs/[this-spec]/implementation/prompts/`)

**Output format:**

```
✅ Task Group [task-group-number] Complete: [task-group-name]

Implemented: [brief list]
Files modified: [list files]

🔍 Review check: [No issues found / X trade-offs, Y contradictions detected]
[IF review needed: "⏳ Waiting for user decision..." ELSE: "✅ Auto-proceeding"]

➡️ Proceeding to Task Group [next-number]...
```

⚠️ **IMPORTANT**: Do NOT implement the next task group directly. You MUST read the next prompt file first and follow its instructions.

[IF this is the final task group, output instead:]
```
🎉 ALL TASK GROUPS COMPLETE!

The implementation is finished. See validation report at:
`agent-os/specs/[this-spec]/implementation/cache/validation-report.md`
```
```

### Step 3: Match Tasks to Relevant Module Basepoints

For each task group, analyze the task description and match it to the most relevant module basepoints:

```bash
# For each task group in orchestration.yml
for task_group in $(yq e '.task_groups[].name' "$SPEC_PATH/orchestration.yml"); do
    # Find relevant basepoints based on task keywords
    # This enables targeted knowledge injection per task group
    {{workflows/scope-detection/detect-scope-keyword-matching}}
done
```

When generating prompts, include the relevant basepoints knowledge for that specific task group.

### Step 4: Output the list of created prompt files

Output to user the following:

```
✅ Prompts generated for [spec-title]!

✅ Basepoints knowledge extracted: [Yes / No basepoints found]
✅ Library basepoints knowledge extracted: [Yes / No library basepoints found]
✅ Detected layer: [LAYER or unknown]
✅ Task groups matched to basepoints: [X of Y]
✅ Implementation prompts generated: [X] prompt files created

## Implementation Prompts Generated

The following prompt files have been created in `agent-os/specs/[this-spec]/implementation/prompts/`:

[list prompt files in order, numbered with full paths]
```

### Step 5: Execute Prompts Iteratively

Now execute each prompt one by one, implementing task groups in sequence:

```bash
echo "🚀 Starting iterative prompt execution..."

SPEC_PATH="agent-os/specs/[this-spec]"
PROMPTS_DIR="$SPEC_PATH/implementation/prompts"
TOTAL_PROMPTS=$(ls -1 "$PROMPTS_DIR"/*.md 2>/dev/null | wc -l | tr -d ' ')
CURRENT_PROMPT=1

# Track completion status
COMPLETION_STATUS_FILE="$SPEC_PATH/implementation/cache/orchestration-status.md"

echo "📋 Total prompts to execute: $TOTAL_PROMPTS"

# Iterate through each prompt in order
for prompt_file in $(ls -1 "$PROMPTS_DIR"/*.md | sort -V); do
    PROMPT_NAME=$(basename "$prompt_file" .md)
    
    echo ""
    echo "═══════════════════════════════════════════════════════"
    echo "  EXECUTING PROMPT $CURRENT_PROMPT of $TOTAL_PROMPTS"
    echo "  Prompt: $PROMPT_NAME"
    echo "═══════════════════════════════════════════════════════"
    
    # ⚠️ CRITICAL: Always use the specifically created prompt
    # DO NOT implement directly - MUST read and follow the prompt
    echo "📖 Reading prompt: $prompt_file"
    PROMPT_CONTENT=$(cat "$prompt_file")
    
    # Execute the prompt (follow its instructions)
    echo "🔧 Implementing task group using prompt..."
    # [The AI agent reads and follows the prompt instructions here]
    # This includes:
    # - Reading the task group requirements
    # - Using the basepoints knowledge context
    # - Using the library basepoints knowledge
    # - Following implementation instructions
    # - Running validation
    # - Marking tasks complete
    
    # Update completion status
    echo "- [x] Prompt $CURRENT_PROMPT: $PROMPT_NAME - Completed $(date)" >> "$COMPLETION_STATUS_FILE"
    
    echo "✅ Prompt $CURRENT_PROMPT complete: $PROMPT_NAME"
    
    CURRENT_PROMPT=$((CURRENT_PROMPT + 1))
done

echo ""
echo "═══════════════════════════════════════════════════════"
echo "  ALL PROMPTS EXECUTED"
echo "═══════════════════════════════════════════════════════"
```

**⚠️ CRITICAL: Prompt Usage Enforcement**

- **NEVER** implement a task group without reading its specific prompt first
- **ALWAYS** use the prompt file created for that task group
- **VALIDATE** that implementation follows the prompt's instructions
- **TRACK** completion status in `orchestration-status.md`

### Step 6: Output Final Completion Status

Output to user the following:

```
🎉 Orchestration and Implementation Complete for [spec-title]!

✅ All [X] task groups implemented using their specific prompts
✅ Basepoints knowledge integrated
✅ Library basepoints knowledge integrated
✅ Validation passed

## Completion Summary

[List each task group with completion status]

## Files Modified

[List all files that were modified during implementation]

## Validation Report

See: `agent-os/specs/[this-spec]/implementation/cache/validation-report.md`

## Orchestration Status

See: `agent-os/specs/[this-spec]/implementation/cache/orchestration-status.md`
```

### Step 7: Run Validation

After orchestration setup is complete, run validation:

```bash
SPEC_PATH="agent-os/specs/[current-spec]"
COMMAND="orchestrate-tasks"
{{workflows/validation/validate-output-exists}}
{{workflows/validation/validate-knowledge-integration}}
{{workflows/validation/generate-validation-report}}
```

### Step 8: Generate Resource Checklist

Generate a checklist of all resources consulted:

```bash
{{workflows/basepoints/organize-and-cache-basepoints-knowledge}}
```

### Step 9: Save Handoff

{{workflows/prompting/save-handoff}}

{{ENDUNLESS use_claude_code_subagents}}
