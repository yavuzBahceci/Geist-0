Now that agents are specialized, proceed with specializing workflows based on project needs by following these instructions:

## Core Responsibilities

1. **Analyze Workflow Requirements**: Determine which workflows are needed based on project complexity
2. **Evaluate Existing Workflows**: Check which workflows from templates are relevant to this project
3. **Specialize Workflows**: Update workflows with project-specific patterns and context
4. **Remove Unnecessary Workflows**: For simple projects, remove workflows that add unnecessary complexity
5. **Create Project-Specific Workflows**: Add new workflows based on project-specific patterns

## Workflow

### Step 1: Load Knowledge and Complexity Assessment

Load knowledge and complexity assessment:

```bash
# Load merged knowledge
if [ -f "agent-os/output/deploy-agents/knowledge/merged-knowledge.json" ]; then
    MERGED_KNOWLEDGE=$(cat agent-os/output/deploy-agents/knowledge/merged-knowledge.json)
    echo "✅ Loaded merged knowledge"
fi

# Load basepoints knowledge
if [ -f "agent-os/output/deploy-agents/knowledge/basepoints-knowledge.json" ]; then
    BASEPOINTS_KNOWLEDGE=$(cat agent-os/output/deploy-agents/knowledge/basepoints-knowledge.json)
fi

# Load complexity assessment
if [ -f "agent-os/output/deploy-agents/reports/complexity-assessment.json" ]; then
    COMPLEXITY=$(cat agent-os/output/deploy-agents/reports/complexity-assessment.json)
    PROJECT_NATURE=$(echo "$COMPLEXITY" | grep -o '"nature": *"[^"]*"' | cut -d'"' -f4)
else
    PROJECT_NATURE="moderate"
fi

echo "Project Nature: $PROJECT_NATURE"
```

### Step 2: Evaluate Workflow Relevance

Determine which workflows are needed for this project:

```bash
# List existing workflows
EXISTING_WORKFLOWS=$(find agent-os/workflows -name "*.md" -type f 2>/dev/null)

# Core workflow categories
WORKFLOW_CATEGORIES=(
    "specification"    # Always needed
    "implementation"   # Always needed
    "planning"         # Needed for moderate/complex
    "validation"       # Needed for complex
    "codebase-analysis" # Needed for complex
)

# For each workflow, evaluate relevance based on complexity
WORKFLOWS_EVALUATION=""
for workflow_file in $EXISTING_WORKFLOWS; do
    WORKFLOW_NAME=$(basename "$workflow_file" .md)
    WORKFLOW_DIR=$(dirname "$workflow_file" | sed 's|agent-os/workflows/||')
    
    # Evaluate based on complexity and category
    case "$PROJECT_NATURE" in
        "simple")
            # Simple projects need only core workflows
            if [[ "$WORKFLOW_DIR" =~ ^(specification|implementation)$ ]]; then
                RELEVANCE="required"
            else
                RELEVANCE="optional"
            fi
            ;;
        "moderate")
            # Moderate projects need most workflows
            if [[ "$WORKFLOW_DIR" =~ ^(validation|scope-detection)$ ]]; then
                RELEVANCE="optional"
            else
                RELEVANCE="required"
            fi
            ;;
        "complex")
            # Complex projects need all workflows
            RELEVANCE="required"
            ;;
    esac
    
    WORKFLOWS_EVALUATION="${WORKFLOWS_EVALUATION}${WORKFLOW_DIR}/${WORKFLOW_NAME}:${RELEVANCE}\n"
done
```

### Step 3: Specialize Workflows with Project Patterns

Update workflows with project-specific patterns:

```bash
# Extract workflow-relevant patterns from basepoints
WORKFLOW_PATTERNS=$(extract_workflow_patterns "$BASEPOINTS_KNOWLEDGE")

# For each existing workflow
for workflow_file in $EXISTING_WORKFLOWS; do
    WORKFLOW_NAME=$(basename "$workflow_file" .md)
    WORKFLOW_CONTENT=$(cat "$workflow_file")
    
    # Extract project-specific patterns relevant to this workflow
    RELEVANT_PATTERNS=$(extract_patterns_for_workflow "$WORKFLOW_PATTERNS" "$WORKFLOW_NAME")
    
    if [ -n "$RELEVANT_PATTERNS" ]; then
        # Inject project patterns into workflow
        UPDATED_CONTENT=$(inject_patterns_into_workflow "$WORKFLOW_CONTENT" "$RELEVANT_PATTERNS")
        echo "$UPDATED_CONTENT" > "$workflow_file"
        echo "✅ Updated workflow with project patterns: $WORKFLOW_NAME"
    fi
done
```

### Step 4: Simplify Workflows Based on Complexity

Apply complexity-based simplification to reduce overhead for simpler projects:

```bash
echo "📋 Applying complexity-based workflow configuration..."

# Define workflow tiers by complexity
declare -A WORKFLOW_TIERS
WORKFLOW_TIERS[simple]="specification implementation basepoints"
WORKFLOW_TIERS[moderate]="specification implementation basepoints planning detection research"
WORKFLOW_TIERS[complex]="specification implementation basepoints planning detection research validation scope-detection deep-reading human-review codebase-analysis"

# Get active workflow categories for this complexity
ACTIVE_CATEGORIES="${WORKFLOW_TIERS[$PROJECT_NATURE]}"

# Create workflow configuration
cat > "agent-os/workflows/workflow-config.yml" << EOF
# Workflow Configuration
# Auto-generated based on project complexity: $PROJECT_NATURE

project_nature: $PROJECT_NATURE

# Active workflow categories for this project
active_categories:
$(echo "$ACTIVE_CATEGORIES" | tr ' ' '\n' | while read cat; do
    echo "  - $cat"
done)

# Workflow simplification rules
simplification:
  simple:
    # Skip these for simple projects
    skip_workflows:
      - deep-reading/*
      - human-review/*
      - scope-detection/*
      - validation/validate-*-patterns.md  # Layer validations optional
    # Combine these into lighter alternatives
    combine:
      - specification/* → specification/write-spec.md only
    # Max iterations for research
    max_research_iterations: 1
    
  moderate:
    skip_workflows:
      - deep-reading/read-implementation-deep.md
    max_research_iterations: 3
    
  complex:
    skip_workflows: []
    max_research_iterations: 5
    # Enable all layer validations
    enable_layer_validations: true

# Layer validation configuration
layer_validations:
  enabled: $([ "$PROJECT_NATURE" = "complex" ] && echo "true" || echo "false")
  validators:
    - validate-ui-patterns.md
    - validate-api-patterns.md
    - validate-data-patterns.md
EOF

echo "✅ Created workflow configuration: agent-os/workflows/workflow-config.yml"

# For simple projects, create a simplified workflow manifest
if [ "$PROJECT_NATURE" = "simple" ]; then
    echo "📋 Simplifying workflows for simple project..."
    
    # Create a simplified-workflows.md guide
    cat > "agent-os/workflows/simplified-workflows.md" << EOF
# Simplified Workflows for Simple Project

This project is classified as **simple**, so workflows have been streamlined.

## Active Workflows

| Category | Status | Notes |
|----------|--------|-------|
| specification | ✅ Active | Full spec workflow |
| implementation | ✅ Active | Full implementation workflow |
| basepoints | ✅ Active | Basic knowledge extraction |
| planning | ⚠️ Minimal | Use /adapt-to-product only |
| detection | ⚠️ Minimal | Tech stack detection only |
| validation | ❌ Skipped | Use project's own tests |
| research | ❌ Skipped | Manual research if needed |
| deep-reading | ❌ Skipped | Direct file reading |
| human-review | ❌ Skipped | Direct user interaction |
| scope-detection | ❌ Skipped | Use simple layer detection |

## Recommended Workflow

For simple projects:

\`\`\`
/adapt-to-product → /create-basepoints → /deploy-agents
    ↓
/shape-spec → /write-spec → /create-tasks → /implement-tasks
\`\`\`

Skip /orchestrate-tasks for simple projects - use /implement-tasks directly.

## Re-enabling Workflows

If your project grows in complexity, re-run:
\`\`\`
/deploy-agents
\`\`\`

This will reassess complexity and enable appropriate workflows.
EOF
    
    echo "✅ Created simplified workflow guide: agent-os/workflows/simplified-workflows.md"
fi

# For complex projects, ensure all layer validations are enabled
if [ "$PROJECT_NATURE" = "complex" ]; then
    echo "📋 Enabling comprehensive workflows for complex project..."
    
    # Ensure layer validation workflows are properly configured
    if [ -d "agent-os/agents/specialists" ]; then
        cat > "agent-os/workflows/layer-validation-config.md" << EOF
# Layer Validation Configuration

This project is **complex** with detected abstraction layers.

## Enabled Layer Validations

| Layer | Validator | Specialist |
|-------|-----------|------------|
$(find agent-os/agents/specialists -name "*.md" -type f 2>/dev/null | while read f; do
    SPECIALIST=$(basename "$f" .md)
    LAYER=$(echo "$SPECIALIST" | sed 's/-specialist//')
    echo "| $LAYER | validate-${LAYER}-patterns.md | $SPECIALIST |"
done)

## Validation Trigger Points

Layer validations run automatically:
- After each /implement-tasks completion
- During /orchestrate-tasks verification phase
- When running /cleanup-agent-os

## Disabling Layer Validations

To skip layer validations (not recommended), add to your command:
\`\`\`
Skip layer validations for this implementation.
\`\`\`
EOF
        
        echo "✅ Configured layer validations for complex project"
    fi
fi
```

### Step 5: Create Project-Specific Workflows

Create new workflows based on project-specific patterns:

```bash
# Analyze if project needs specialized workflows based on:
# - Unique data flows from basepoints
# - Specific control flows from basepoints
# - Complex dependency flows

PROJECT_WORKFLOW_NEEDS=$(analyze_workflow_needs "$BASEPOINTS_KNOWLEDGE")

if [ -n "$PROJECT_WORKFLOW_NEEDS" ]; then
    # Create project-specific workflows directory
    mkdir -p agent-os/workflows/project
    
    for workflow_need in $PROJECT_WORKFLOW_NEEDS; do
        WORKFLOW_NAME=$(echo "$workflow_need" | cut -d':' -f1)
        WORKFLOW_PURPOSE=$(echo "$workflow_need" | cut -d':' -f2-)
        
        cat > "agent-os/workflows/project/${WORKFLOW_NAME}.md" << EOF
# Workflow: ${WORKFLOW_NAME}

## Purpose

${WORKFLOW_PURPOSE}

## Core Responsibilities

1. [Responsibility based on project patterns]
2. [Responsibility based on project flows]

## Workflow Steps

### Step 1: [Step Name]

[Step description based on project patterns]

### Step 2: [Step Name]

[Step description based on project patterns]

## Context

This workflow was created based on patterns detected in the project's codebase during deploy-agents specialization.

## Important Constraints

- Follow project standards and conventions
- Reference basepoints knowledge for patterns
- Maintain consistency with project architecture
EOF
        
        echo "✅ Created project-specific workflow: ${WORKFLOW_NAME}"
    done
fi
```

### Step 6: Verify Workflows Consistency

Verify all workflows are consistent and properly configured:

```bash
# Count workflows
WORKFLOWS_COUNT=$(find agent-os/workflows -name "*.md" -type f | wc -l | tr -d ' ')

# Count workflow categories
CATEGORY_COUNT=$(find agent-os/workflows -type d | wc -l | tr -d ' ')

# Log to reports
mkdir -p agent-os/output/deploy-agents/reports
cat > agent-os/output/deploy-agents/reports/workflows-specialization.md << EOF
# Workflows Specialization Report

## Summary
- Project Nature: $PROJECT_NATURE
- Total Workflows: $WORKFLOWS_COUNT
- Workflow Categories: $CATEGORY_COUNT

## Workflow Categories
$(find agent-os/workflows -type d | sed 's|agent-os/workflows/||' | while read d; do
    [ -n "$d" ] && echo "- $d: $(find "agent-os/workflows/$d" -name "*.md" -type f 2>/dev/null | wc -l | tr -d ' ') workflows"
done)

## Actions Taken
- Updated workflows with project patterns from basepoints
- Simplified workflows for project complexity level
- Created project-specific workflows where needed

## Next Steps
Proceed to finalize deployment in phase 11.
EOF

echo "✅ Workflows specialization complete"
echo "📁 Report saved to: agent-os/output/deploy-agents/reports/workflows-specialization.md"
```

{{UNLESS compiled_single_command}}
## Display confirmation and next step

Once you've specialized workflows, output the following message:

```
✅ Workflows specialized for project!

**Project Nature:** [simple/moderate/complex]
**Workflows Updated:** [count]
**Project-Specific Workflows Created:** [count]

Report: `agent-os/output/deploy-agents/reports/workflows-specialization.md`

NEXT STEP 👉 Run the command, `11-adapt-structure-and-finalize.md`
```
{{ENDUNLESS compiled_single_command}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

Ensure workflows specialization follows the user's preferences:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
