# Task List Creation

## Core Responsibilities

1. **Analyze spec and requirements**: Read and analyze the spec.md and/or requirements.md to inform the tasks list you will create.
2. **Plan task execution order**: Break the requirements into a list of tasks in an order that takes their dependencies into account.
3. **Group tasks by specialization**: Group tasks that require the same skill or domain specialization together (data layer, interface layer, business logic, etc.)
4. **Create Tasks list**: Create the markdown tasks list broken into groups with sub-tasks.

## Workflow

### Step 1: Analyze Spec & Requirements

Read each of these files (whichever are available) and analyze them to understand the requirements for this feature implementation:
- `geist/specs/[this-spec]/spec.md`
- `geist/specs/[this-spec]/planning/requirements.md`

```bash
# Determine spec path
SPEC_PATH="geist/specs/[this-spec]"

# Load extracted basepoints knowledge if available
if [ -f "$SPEC_PATH/implementation/cache/basepoints-knowledge.json" ]; then
    EXTRACTED_KNOWLEDGE=$(cat "$SPEC_PATH/implementation/cache/basepoints-knowledge.json")
    SCOPE_DETECTION=$(cat "$SPEC_PATH/implementation/cache/scope-detection/semantic-analysis.json" 2>/dev/null || echo "{}")
    KEYWORD_MATCHING=$(cat "$SPEC_PATH/implementation/cache/scope-detection/keyword-matching.json" 2>/dev/null || echo "{}")
fi
```

Use your learnings from spec/requirements AND basepoints knowledge (if available) to inform the tasks list and groupings you will create in the next step.

**From Basepoints Knowledge (if available):**
- Use extracted patterns to inform task breakdown
- Suggest existing patterns and checkpoints from basepoints
- Reference project-specific implementation strategies
- Include relevant testing approaches in task creation


### Step 2: Check if Deep Reading is Needed

Before creating tasks, check if deep implementation reading is needed:

```bash
# Check abstraction layer distance
{{workflows/scope-detection/calculate-abstraction-layer-distance}}

# If deep reading is needed, perform deep reading
{{workflows/deep-reading/read-implementation-deep}}

# Load deep reading results if available
if [ -f "$SPEC_PATH/implementation/cache/deep-reading/implementation-analysis.json" ]; then
    DEEP_READING_RESULTS=$(cat "$SPEC_PATH/implementation/cache/deep-reading/implementation-analysis.json")
    
    # Detect reusable code
    {{workflows/deep-reading/detect-reusable-code}}
fi
```

If deep reading was performed, use the results to:
- Inform task breakdown with actual implementation patterns
- Suggest existing patterns and checkpoints from actual implementation
- Include tasks for reusing existing code
- Include tasks for refactoring opportunities

### Step 2.5: Check for Trade-offs (if needed)

Before creating tasks, check if trade-offs need to be reviewed:

```bash
{{workflows/human-review/review-trade-offs}}
```

If trade-offs are detected, present them to the user and wait for their decision before proceeding.

### Step 3: Check for Checkpoints (if needed)

Before creating tasks, check if checkpoints are needed for big changes:

```bash
{{workflows/human-review/create-checkpoint}}
```

If a checkpoint is needed, present it to the user and wait for their confirmation before proceeding.

### Step 4: Create Tasks Breakdown (SDD-aligned)

Generate `geist/specs/[current-spec]/tasks.md`.

**SDD Task Decomposition Best Practices:**
- Respect SDD phase order: Specify → Tasks → Implement (spec should be complete before tasks)
- Ensure each task can be validated against spec acceptance criteria
- Break work into small, testable, isolated tasks
- Order tasks by dependency (respecting SDD phase order)
- Reference spec acceptance criteria when creating task validation

**INVEST Criteria for Task Quality:**
When creating tasks, ensure they are:
- **Independent**: Can be done in any order where dependencies allow
- **Valuable**: Deliver standalone value (not just technical subtasks)
- **Small**: Manageable size that can be completed efficiently
- **Estimable**: Can be estimated with reasonable accuracy
- **Testable**: Have clear acceptance criteria from spec

**Atomic Task Principles:**
- Tasks should be independently valuable (not just technical subtasks)
- Tasks should be independently testable (have clear acceptance criteria)
- Task decomposition should follow SDD principles (small, testable, isolated)

**Important**: The exact tasks, task groups, and organization will vary based on the feature's specific requirements. The following is an example format - adapt the content of the tasks list to match what THIS feature actually needs.

```markdown
# Task Breakdown: [Feature Name]

## Overview
Total Tasks: [count]

## Task List

### Core Functionality Layer

#### Task Group 1: Core Logic and Data Structures
**Dependencies:** None

- [ ] 1.0 Complete core functionality layer
  - [ ] 1.1 Write 2-8 focused tests for core functionality
    - Limit to 2-8 highly focused tests maximum
    - Test only critical behaviors (e.g., primary validation, key relationships, core methods)
    - Skip exhaustive coverage of all methods and edge cases
  - [ ] 1.2 Create core data structures or classes
    - Fields/Properties: [list]
    - Validations/Constraints: [list]
    - Reuse pattern from: [existing structure if applicable]
  - [ ] 1.3 Implement data persistence (if applicable)
    - Schema or structure: [description]
    - Relationships: [if applicable]
  - [ ] 1.4 Set up relationships or dependencies
    - [Structure] relates to [related structure]
    - [Structure] depends on [dependency]
  - [ ] 1.5 Ensure core layer tests pass
    - Run ONLY the 2-8 tests written in 1.1
    - Verify data structures work correctly
    - Do NOT run the entire test suite at this stage

**Acceptance Criteria:**
- The 2-8 tests written in 1.1 pass
- Core structures pass validation tests
- Relationships work correctly

### Interface Layer (if applicable)

#### Task Group 2: Interface Implementation
**Dependencies:** Task Group 1

- [ ] 2.0 Complete interface layer
  - [ ] 2.1 Write 2-8 focused tests for interface
    - Limit to 2-8 highly focused tests maximum
    - Test only critical interface behaviors (e.g., primary operations, key error cases)
    - Skip exhaustive testing of all operations and scenarios
  - [ ] 2.2 Create interface implementation
    - Operations: [list of key operations]
    - Follow pattern from: [existing interface]
  - [ ] 2.3 Implement authentication/authorization (if applicable)
    - Use existing auth pattern
    - Add permission checks
  - [ ] 2.4 Add response formatting and error handling
    - Response format: [description]
    - Error handling: [approach]
    - Status indicators: [how success/failure is communicated]
  - [ ] 2.5 Ensure interface layer tests pass
    - Run ONLY the 2-8 tests written in 2.1
    - Verify critical operations work
    - Do NOT run the entire test suite at this stage

**Acceptance Criteria:**
- The 2-8 tests written in 2.1 pass
- All critical operations work
- Proper authorization enforced (if applicable)
- Consistent response format

### User Interface Layer (if applicable)

#### Task Group 3: User Interface Design
**Dependencies:** Task Group 2 (if interface layer exists) or Task Group 1

- [ ] 3.0 Complete user interface
  - [ ] 3.1 Write 2-8 focused tests for UI components
    - Limit to 2-8 highly focused tests maximum
    - Test only critical component behaviors (e.g., primary user interaction, key form submission, main rendering case)
    - Skip exhaustive testing of all component states and interactions
  - [ ] 3.2 Create UI components or views
    - Reuse: [existing component] as base
    - Properties/Props: [list]
    - State: [list]
  - [ ] 3.3 Implement user input handling
    - Fields/Inputs: [list]
    - Validation: [approach]
    - Submit/Process handling
  - [ ] 3.4 Build main view or screen
    - Layout: [description]
    - Components: [list]
    - Match mockup: `planning/visuals/[file]` (if applicable)
  - [ ] 3.5 Apply styling and design
    - Follow existing design system
    - Use design tokens from: [style system]
  - [ ] 3.6 Implement responsive or adaptive design (if applicable)
    - Breakpoints or contexts: [description]
  - [ ] 3.7 Add interactions and feedback
    - User feedback mechanisms
    - Loading states
    - Error states
  - [ ] 3.8 Ensure UI component tests pass
    - Run ONLY the 2-8 tests written in 3.1
    - Verify critical component behaviors work
    - Do NOT run the entire test suite at this stage

**Acceptance Criteria:**
- The 2-8 tests written in 3.1 pass
- Components render correctly
- Forms/inputs validate and process correctly
- Matches visual design (if provided)

### Testing

#### Task Group 4: Test Review & Gap Analysis
**Dependencies:** Task Groups 1-3

- [ ] 4.0 Review existing tests and fill critical gaps only
  - [ ] 4.1 Review tests from Task Groups 1-3
    - Review the 2-8 tests written in Task Group 1
    - Review the 2-8 tests written in Task Group 2 (if applicable)
    - Review the 2-8 tests written in Task Group 3 (if applicable)
    - Total existing tests: approximately 6-24 tests
  - [ ] 4.2 Analyze test coverage gaps for THIS feature only
    - Identify critical user workflows that lack test coverage
    - Focus ONLY on gaps related to this spec's feature requirements
    - Do NOT assess entire application test coverage
    - Prioritize end-to-end workflows over unit test gaps
  - [ ] 4.3 Write up to 10 additional strategic tests maximum
    - Add maximum of 10 new tests to fill identified critical gaps
    - Focus on integration points and end-to-end workflows
    - Do NOT write comprehensive coverage for all scenarios
    - Skip edge cases, performance tests, and accessibility tests unless business-critical
  - [ ] 4.4 Run feature-specific tests only
    - Run ONLY tests related to this spec's feature (tests from 1.1, 2.1, 3.1, and 4.3)
    - Expected total: approximately 16-34 tests maximum
    - Do NOT run the entire application test suite
    - Verify critical workflows pass

**Acceptance Criteria:**
- All feature-specific tests pass (approximately 16-34 tests total)
- Critical user workflows for this feature are covered
- No more than 10 additional tests added when filling in testing gaps
- Testing focused exclusively on this spec's feature requirements

## Execution Order

Recommended implementation sequence:
1. Core Functionality Layer (Task Group 1)
2. Interface Layer (Task Group 2) - if applicable
3. User Interface Layer (Task Group 3) - if applicable
4. Test Review & Gap Analysis (Task Group 4)
```

**Note**: When creating tasks, leverage basepoints knowledge (if available) to:
- Suggest existing patterns and checkpoints from basepoints
- Reference project-specific implementation strategies
- Include relevant testing approaches from basepoints
- Avoid unnecessary work by leveraging existing patterns

**Note**: Adapt this structure based on the actual feature requirements. Some features may need:
- Different task groups (e.g., notification systems, data processing, integration work)
- Different execution order based on dependencies
- More or fewer sub-tasks per group
- Task groups for specific domains (e.g., algorithms, data processing, system integration)

### Step 5: Validate Tasks Against SDD Principles (SDD-aligned)

After creating tasks, validate them against SDD best practices and INVEST criteria:

```bash
# SDD Task Validation
TASKS_FILE="$SPEC_PATH/tasks.md"
SPEC_FILE="$SPEC_PATH/spec.md"

if [ -f "$TASKS_FILE" ] && [ -f "$SPEC_FILE" ]; then
    echo "🔍 Validating tasks against SDD principles..."
    
    # INVEST Criteria Validation
    INVEST_ISSUES=""
    
    # Check for Independent tasks (can be done in any order where dependencies allow)
    # Validation: tasks should not have circular dependencies
    # This is checked by ensuring task dependencies follow a DAG structure
    
    # Check for Valuable tasks (deliver standalone value)
    # Look for tasks that are too granular (technical subtasks without standalone value)
    GRANULAR_TASKS=$(grep -iE "TODO|FIXME|refactor|cleanup|optimize" "$TASKS_FILE" | grep -v "Write.*tests" | wc -l)
    if [ "$GRANULAR_TASKS" -gt 5 ]; then
        INVEST_ISSUES="${INVEST_ISSUES}⚠️ Too many technical subtasks detected (may violate 'Valuable' principle). Consider grouping technical subtasks into independently valuable tasks. "
    fi
    
    # Check for Small tasks (manageable size)
    # Validation: tasks should have clear scope that can be completed efficiently
    # Check if tasks are too large (exceeding reasonable completion time)
    LARGE_TASKS=$(grep -iE "complete.*feature|implement.*system|build.*application" "$TASKS_FILE" | wc -l)
    if [ "$LARGE_TASKS" -gt 3 ]; then
        INVEST_ISSUES="${INVEST_ISSUES}⚠️ Large tasks detected (may violate 'Small' principle). Consider breaking down into smaller, manageable tasks. "
    fi
    
    # Check for Estimable tasks (can be estimated with reasonable accuracy)
    # Validation: tasks should have clear scope and acceptance criteria
    TASKS_WITHOUT_AC=$(grep -c "Acceptance Criteria:" "$TASKS_FILE" 2>/dev/null || echo "0")
    TOTAL_TASK_GROUPS=$(grep -c "^#### Task Group" "$TASKS_FILE" 2>/dev/null || echo "0")
    if [ "$TOTAL_TASK_GROUPS" -gt 0 ] && [ "$TASKS_WITHOUT_AC" -lt "$TOTAL_TASK_GROUPS" ]; then
        INVEST_ISSUES="${INVEST_ISSUES}⚠️ Some task groups lack acceptance criteria (may violate 'Estimable' and 'Testable' principles). Ensure all task groups have clear acceptance criteria from spec. "
    fi
    
    # Check for Testable tasks (have clear acceptance criteria from spec)
    # Validate that tasks reference spec acceptance criteria
    TASKS_REFERENCING_SPEC=$(grep -iE "spec|acceptance criteria|requirement" "$TASKS_FILE" | wc -l)
    if [ "$TASKS_REFERENCING_SPEC" -eq 0 ]; then
        INVEST_ISSUES="${INVEST_ISSUES}⚠️ Tasks may not be validating against spec acceptance criteria (may violate 'Testable' principle). Ensure tasks can be validated against spec. "
    fi
    
    # Atomic Task Principles Validation
    ATOMIC_ISSUES=""
    
    # Check for independently valuable tasks
    # Validation: tasks should deliver standalone value, not just be technical subtasks
    SUBTASK_INDICATORS=$(grep -iE "setup|configure|initialize|prepare|helper|utility" "$TASKS_FILE" | grep -v "Write.*tests" | wc -l)
    if [ "$SUBTASK_INDICATORS" -gt 3 ] && [ "$TOTAL_TASK_GROUPS" -lt 5 ]; then
        ATOMIC_ISSUES="${ATOMIC_ISSUES}⚠️ Many technical subtasks detected. Consider ensuring tasks are independently valuable, not just setup/preparation steps. "
    fi
    
    # Check for independently testable tasks
    # Validation: tasks should have clear acceptance criteria
    # Already checked above in INVEST validation
    
    # SDD Phase Order Validation
    SDD_PHASE_ISSUES=""
    
    # Check that tasks respect SDD phase order (Specify → Tasks → Implement)
    # Validation: spec should be complete before tasks are created
    # This is implicit - if we're creating tasks, spec should already exist
    
    # Check that tasks can be validated against spec acceptance criteria
    if [ -f "$SPEC_FILE" ]; then
        SPEC_AC=$(grep -iE "acceptance criteria|Acceptance Criteria" "$SPEC_FILE" | wc -l)
        if [ "$SPEC_AC" -eq 0 ]; then
            SDD_PHASE_ISSUES="${SDD_PHASE_ISSUES}⚠️ Spec may lack acceptance criteria. Tasks should be validated against spec acceptance criteria (SDD best practice). "
        fi
    fi
    
    # Report validation results
    if [ -n "$INVEST_ISSUES$ATOMIC_ISSUES$SDD_PHASE_ISSUES" ]; then
        echo "📋 SDD Task Validation: Issues detected"
        echo ""
        if [ -n "$INVEST_ISSUES" ]; then
            echo "INVEST Criteria Issues:"
            echo "$INVEST_ISSUES"
            echo ""
        fi
        if [ -n "$ATOMIC_ISSUES" ]; then
            echo "Atomic Task Principles Issues:"
            echo "$ATOMIC_ISSUES"
            echo ""
        fi
        if [ -n "$SDD_PHASE_ISSUES" ]; then
            echo "SDD Phase Order Issues:"
            echo "$SDD_PHASE_ISSUES"
            echo ""
        fi
        echo "Consider reviewing tasks to align with SDD best practices."
        echo "Proceed anyway? [Yes/No]"
        # In actual execution, wait for user decision
    else
        echo "✅ SDD Task Validation: All checks passed"
        echo "Tasks align with INVEST criteria, atomic task principles, and SDD best practices."
    fi
fi
```

## Important Constraints

- **Create tasks that are specific and verifiable**
- **Group related tasks:** For example, group core logic tasks together, interface tasks together, and UI tasks together (if applicable).
- **Limit test writing during development**:
  - Each task group (1-3) should write 2-8 focused tests maximum
  - Tests should cover only critical behaviors, not exhaustive coverage
  - Test verification should run ONLY the newly written tests, not the entire suite
  - If there is a dedicated test coverage group for filling in gaps in test coverage, this group should add only a maximum of 10 additional tests IF NECESSARY to fill critical gaps
- **Use a focused test-driven approach** where each task group starts with writing 2-8 tests (x.1 sub-task) and ends with running ONLY those tests (final sub-task)
- **Include acceptance criteria** for each task group
- **Reference visual assets** if visuals are available

## SDD Integration Notes

This workflow has been enhanced with Spec-Driven Development (SDD) best practices:

**SDD Principles Integrated:**
- **SDD Phase Order**: Tasks respect SDD phase order (Specify → Tasks → Implement)
- **Spec as Source of Truth**: Tasks can be validated against spec acceptance criteria
- **Task Decomposition Best Practices**: Tasks are broken into small, testable, isolated units

**INVEST Criteria Integration:**
- **Independent**: Tasks can be done in any order where dependencies allow
- **Valuable**: Tasks deliver standalone value (not just technical subtasks)
- **Small**: Tasks are manageable size that can be completed efficiently
- **Estimable**: Tasks can be estimated with reasonable accuracy
- **Testable**: Tasks have clear acceptance criteria from spec

**Atomic Task Principles:**
- Tasks are independently valuable (not just technical subtasks)
- Tasks are independently testable (have clear acceptance criteria)
- Task decomposition follows SDD principles (small, testable, isolated)

**Technology-Agnostic Approach (Default Profile Templates Only):**
- All SDD framework references are abstract (e.g., "task decomposition frameworks" not technology-specific tools)
- No hardcoded technology-specific task management tool references in default templates
- Task validation maintains technology-agnostic state throughout **in default profile templates**
- **After Specialization:** When templates are compiled to `geist/workflows/`, workflows can and should become technology-specific based on the project's actual stack
- **Command Outputs:** Specs, tasks, and implementations should reflect the project's actual technology stack

**Workflow Steps Enhanced:**
- Step 4: Enhanced task creation guidance with SDD best practices and INVEST criteria
- Step 5: Added SDD task validation against INVEST criteria, atomic task principles, and SDD phase order
