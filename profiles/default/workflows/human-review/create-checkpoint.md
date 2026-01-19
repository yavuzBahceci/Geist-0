# Create Checkpoint for Big Changes

## Core Responsibilities

1. **Detect Big Changes**: Identify big changes or abstraction layer transitions in decision making
2. **Create Optional Checkpoint**: Create optional checkpoint following default profile structure
3. **Present Recommendations**: Present recommendations with context before proceeding
4. **Allow User Review**: Allow user to review and confirm before continuing
5. **Store Checkpoint Results**: Cache checkpoint decisions

## Workflow

### Step 1: Detect Big Changes or Abstraction Layer Transitions

Check if a big change or abstraction layer transition is detected:

```bash
# Determine spec path
SPEC_PATH="{{SPEC_PATH}}"
if [ -z "$SPEC_PATH" ]; then
    SPEC_PATH="geist/specs/[current-spec]"
fi

# Load scope detection results
if [ -f "$SPEC_PATH/implementation/cache/scope-detection/abstraction-layer-distance.json" ]; then
    DISTANCE_RESULTS=$(cat "$SPEC_PATH/implementation/cache/scope-detection/abstraction-layer-distance.json")
fi

# Detect abstraction layer transitions
LAYER_TRANSITION=$({{DETECT_LAYER_TRANSITION}})

# Detect big changes
BIG_CHANGE=$({{DETECT_BIG_CHANGE}})

# Determine if checkpoint is needed
if [ "$LAYER_TRANSITION" = "true" ] || [ "$BIG_CHANGE" = "true" ]; then
    CHECKPOINT_NEEDED="true"
else
    CHECKPOINT_NEEDED="false"
fi
```

### Step 2: Prepare Checkpoint Presentation

If checkpoint is needed, prepare the checkpoint presentation:

```bash
if [ "$CHECKPOINT_NEEDED" = "true" ]; then
    # Extract recommendations with context
    RECOMMENDATIONS=$({{EXTRACT_RECOMMENDATIONS}})
    
    # Extract context from basepoints
    RECOMMENDATION_CONTEXT=$({{EXTRACT_RECOMMENDATION_CONTEXT}})
    
    # Prepare checkpoint presentation
    CHECKPOINT_PRESENTATION=$({{PREPARE_CHECKPOINT_PRESENTATION}})
fi
```

### Step 3: Present Checkpoint to User

Present the checkpoint following default profile human-in-the-loop structure:

```bash
if [ "$CHECKPOINT_NEEDED" = "true" ]; then
    # Output checkpoint presentation to user
    echo "$CHECKPOINT_PRESENTATION"
    
    # Wait for user confirmation
fi
```

**Presentation Format:**

**For Standard Checkpoints:**
```
⚠️ Checkpoint: Big Change or Abstraction Layer Transition Detected

Before proceeding, I'd like to confirm the following significant decision:

**Change Type:** [Abstraction Layer Transition / Big Change]

**Current Context:**
[Current abstraction layer or state]

**Proposed Change:**
[Description of the change]

**Recommendation:**
Based on basepoints knowledge and project structure, I recommend:
- [Recommendation 1]
- [Recommendation 2]

**Context from Basepoints:**
[Relevant context from basepoints that informs this decision]

**Impact:**
- [Impact on architecture]
- [Impact on existing patterns]
- [Impact on related modules]

**Your Confirmation:**
Please confirm to proceed, or provide modifications:
1. Confirm: Proceed with recommendation
2. Modify: [Your modification]
3. Cancel: Do not proceed with this change
```

**For SDD Checkpoints:**

**SDD Checkpoint: Spec Completeness Before Task Creation**
```
🔍 SDD Checkpoint: Spec Completeness Validation

Before proceeding to task creation (SDD phase order: Specify → Tasks → Implement), let's ensure the specification is complete:

**SDD Principle:** "Specify" phase should be complete before "Tasks" phase

**Current Spec Status:**
- User stories: [Present / Missing]
- Acceptance criteria: [Present / Missing]
- Scope boundaries: [Present / Missing]

**Recommendation:**
Based on SDD best practices, ensure the spec includes:
- Clear user stories in format "As a [user], I want [action], so that [benefit]"
- Explicit acceptance criteria for each requirement
- Defined scope boundaries (in-scope vs out-of-scope)

**Your Decision:**
1. Enhance spec: Review and enhance spec before creating tasks
2. Proceed anyway: Create tasks despite incomplete spec
3. Cancel: Do not proceed with task creation
```

**SDD Checkpoint: Spec Alignment Before Implementation**
```
🔍 SDD Checkpoint: Spec Alignment Validation

Before proceeding to implementation (SDD: spec as source of truth), let's validate that tasks align with the spec:

**SDD Principle:** Spec is the source of truth - implementation should validate against spec

**Current Alignment:**
- Spec acceptance criteria: [Count]
- Task acceptance criteria: [Count]
- Alignment: [Aligned / Misaligned]

**Recommendation:**
Based on SDD best practices, ensure tasks can be validated against spec acceptance criteria:
- Each task should reference spec acceptance criteria
- Tasks should align with spec scope and requirements
- Implementation should validate against spec as source of truth

**Your Decision:**
1. Align tasks: Review and align tasks with spec before implementation
2. Proceed anyway: Begin implementation despite misalignment
3. Cancel: Do not proceed with implementation
```

### Step 4: Process User Confirmation

Process the user's confirmation:

```bash
# Wait for user response
USER_CONFIRMATION=$({{GET_USER_CONFIRMATION}})

# Process confirmation
if [ "$USER_CONFIRMATION" = "confirm" ]; then
    PROCEED="true"
    MODIFICATIONS=""
elif [ "$USER_CONFIRMATION" = "modify" ]; then
    PROCEED="true"
    MODIFICATIONS=$({{GET_USER_MODIFICATIONS}})
elif [ "$USER_CONFIRMATION" = "cancel" ]; then
    PROCEED="false"
    MODIFICATIONS=""
fi

# Store checkpoint decision
CHECKPOINT_RESULT="{
  \"checkpoint_type\": \"big_change_or_layer_transition\",
  \"change_type\": \"$LAYER_TRANSITION_OR_BIG_CHANGE\",
  \"recommendations\": $(echo "$RECOMMENDATIONS" | {{JSON_FORMAT}}),
  \"user_confirmation\": \"$USER_CONFIRMATION\",
  \"proceed\": \"$PROCEED\",
  \"modifications\": \"$MODIFICATIONS\"
}"
```

### Step 2.5: Check for SDD Checkpoints (SDD-aligned)

Before presenting checkpoints, check if SDD-specific checkpoints are needed:

```bash
# SDD Checkpoint Detection
SDD_CHECKPOINT_NEEDED="false"
SDD_CHECKPOINT_TYPE=""

SPEC_FILE="$SPEC_PATH/spec.md"
TASKS_FILE="$SPEC_PATH/tasks.md"
REQUIREMENTS_FILE="$SPEC_PATH/planning/requirements.md"
IMPLEMENTATION_PATH="$SPEC_PATH/implementation"

# Checkpoint 1: Before task creation - ensure spec is complete (SDD: "Specify" phase complete before "Tasks" phase)
# This checkpoint should trigger when transitioning from spec creation to task creation
if [ -f "$SPEC_FILE" ] && [ ! -f "$TASKS_FILE" ]; then
    # Spec exists but tasks don't - this is the transition point
    
    # Check if spec is complete (has user stories, acceptance criteria, scope boundaries)
    HAS_USER_STORIES=$(grep -iE "as a .*i want .*so that|user story" "$SPEC_FILE" | wc -l)
    HAS_ACCEPTANCE_CRITERIA=$(grep -iE "acceptance criteria|Acceptance Criteria" "$SPEC_FILE" | wc -l)
    HAS_SCOPE=$(grep -iE "in scope|out of scope|scope boundary|Scope:" "$SPEC_FILE" | wc -l)
    
    # Only trigger checkpoint if spec appears incomplete
    if [ "$HAS_USER_STORIES" -eq 0 ] || [ "$HAS_ACCEPTANCE_CRITERIA" -eq 0 ] || [ "$HAS_SCOPE" -eq 0 ]; then
        SDD_CHECKPOINT_NEEDED="true"
        SDD_CHECKPOINT_TYPE="spec_completeness_before_tasks"
        echo "🔍 SDD Checkpoint: Spec completeness validation needed before task creation"
    fi
fi

# Checkpoint 2: Before implementation - validate spec alignment (SDD: spec as source of truth validation)
# This checkpoint should trigger when transitioning from task creation to implementation
if [ -f "$TASKS_FILE" ] && [ ! -d "$IMPLEMENTATION_PATH" ] || [ -z "$(find "$IMPLEMENTATION_PATH" -name "*.md" -o -name "*.js" -o -name "*.py" -o -name "*.ts" 2>/dev/null | head -1)" ]; then
    # Tasks exist but implementation doesn't - this is the transition point
    
    # Check if tasks can be validated against spec acceptance criteria
    if [ -f "$SPEC_FILE" ]; then
        SPEC_AC_COUNT=$(grep -c "Acceptance Criteria:" "$SPEC_FILE" 2>/dev/null || echo "0")
        TASKS_AC_COUNT=$(grep -c "Acceptance Criteria:" "$TASKS_FILE" 2>/dev/null || echo "0")
        
        # Only trigger checkpoint if tasks don't align with spec
        if [ "$SPEC_AC_COUNT" -gt 0 ] && [ "$TASKS_AC_COUNT" -eq 0 ]; then
            SDD_CHECKPOINT_NEEDED="true"
            SDD_CHECKPOINT_TYPE="spec_alignment_before_implementation"
            echo "🔍 SDD Checkpoint: Spec alignment validation needed before implementation"
        fi
    fi
fi

# If SDD checkpoint needed, add to checkpoint list
if [ "$SDD_CHECKPOINT_NEEDED" = "true" ]; then
    # Merge with existing checkpoint detection
    if [ "$CHECKPOINT_NEEDED" = "false" ]; then
        CHECKPOINT_NEEDED="true"
        echo "   SDD checkpoint added: $SDD_CHECKPOINT_TYPE"
    else
        echo "   Additional SDD checkpoint: $SDD_CHECKPOINT_TYPE"
    fi
    
    # Store SDD checkpoint info for presentation
    SDD_CHECKPOINT_INFO="{
      \"type\": \"$SDD_CHECKPOINT_TYPE\",
      \"sdd_aligned\": true,
      \"principle\": \"$(if [ "$SDD_CHECKPOINT_TYPE" = "spec_completeness_before_tasks" ]; then echo "Specify phase complete before Tasks phase"; else echo "Spec as source of truth validation"; fi)\"
    }"
fi
```

### Step 3: Present Checkpoint to User (Enhanced with SDD Checkpoints)

Present the checkpoint following default profile human-in-the-loop structure, including SDD-specific checkpoints:

```bash
# Determine cache path
if [ -n "$SPEC_PATH" ]; then
    CACHE_PATH="$SPEC_PATH/implementation/cache"
else
    CACHE_PATH="geist/output/deploy-agents/knowledge"
fi

mkdir -p "$CACHE_PATH/human-review"

# Store checkpoint decision
echo "$CHECKPOINT_RESULT" > "$CACHE_PATH/human-review/checkpoint-review.json"

# Also create human-readable summary
cat > "$CACHE_PATH/human-review/checkpoint-review-summary.md" << EOF
# Checkpoint Review Results

## Change Type
[Type of change: abstraction layer transition / big change]

## Recommendations
[Summary of recommendations presented]

## User Confirmation
[User's confirmation: confirm/modify/cancel]

## Proceed
[Whether to proceed: true/false]

## Modifications
[Any modifications requested by user]
EOF
```

## Important Constraints

- Must detect big changes or abstraction layer transitions in decision making
- Must create optional checkpoints following default profile structure
- Must present recommendations with context before proceeding
- Must allow user to review and confirm before continuing
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
- Must be technology-agnostic and work with any basepoint structure
- **CRITICAL**: All checkpoint results must be stored in `geist/specs/[current-spec]/implementation/cache/human-review/`, not scattered around the codebase

## SDD Integration Notes

This workflow has been enhanced with Spec-Driven Development (SDD) best practices:

**SDD Checkpoints:**
- **Before Task Creation**: Validates spec completeness (SDD: "Specify" phase complete before "Tasks" phase)
- **Before Implementation**: Validates spec alignment (SDD: spec as source of truth validation)
- **Conditional Triggering**: SDD checkpoints only trigger when they add value and don't create unnecessary friction
- **Human Review at Every Checkpoint**: Integrates SDD principle to prevent spec drift

**SDD Checkpoint Types:**
- `spec_completeness_before_tasks`: Ensures spec has user stories, acceptance criteria, and scope boundaries before creating tasks
- `spec_alignment_before_implementation`: Validates that tasks can be validated against spec acceptance criteria before implementation

**Technology-Agnostic Approach (Default Profile Templates Only):**
- All SDD checkpoint detection is structure-based, not technology-specific
- No hardcoded technology-specific references in default templates
- Checkpoints maintain technology-agnostic state throughout **in default profile templates**
- **After Specialization:** When templates are compiled to `geist/workflows/`, workflows can and should become technology-specific based on the project's actual stack
- **Command Outputs:** Specs, tasks, and implementations should reflect the project's actual technology stack
