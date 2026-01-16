Now that standards are specialized, proceed with specializing agents based on project needs by following these instructions:

## Core Responsibilities

1. **Analyze Agent Requirements**: Determine which agents are needed based on project complexity
2. **Evaluate Existing Agents**: Check which agents from templates are relevant to this project
3. **Specialize Agents**: Update agents with project-specific context and patterns
4. **Remove Unnecessary Agents**: For simple projects, remove agents that add unnecessary complexity
5. **Create Project-Specific Agents**: Add new agents based on project-specific needs

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

### Step 2: Evaluate Agent Relevance

Determine which agents are needed for this project:

```bash
# List existing agents
EXISTING_AGENTS=$(find agent-os/agents -name "*.md" -type f 2>/dev/null)

# For each agent, evaluate relevance based on:
# 1. Project patterns from basepoints
# 2. Project complexity
# 3. Project tech stack and requirements

AGENTS_EVALUATION=""
for agent_file in $EXISTING_AGENTS; do
    AGENT_NAME=$(basename "$agent_file" .md)
    
    # Evaluate based on complexity
    case "$PROJECT_NATURE" in
        "simple")
            # Simple projects need only core agents
            if [[ "$AGENT_NAME" =~ ^(implementation|specification)$ ]]; then
                RELEVANCE="required"
            else
                RELEVANCE="optional"
            fi
            ;;
        "moderate")
            # Moderate projects need most agents
            RELEVANCE="required"
            ;;
        "complex")
            # Complex projects need all agents plus specialized
            RELEVANCE="required"
            ;;
    esac
    
    AGENTS_EVALUATION="${AGENTS_EVALUATION}${AGENT_NAME}:${RELEVANCE}\n"
done
```

### Step 3: Specialize Agents with Project Context

Update agents with project-specific context:

```bash
# Extract project context from basepoints
PROJECT_CONTEXT=$(extract_project_context "$BASEPOINTS_KNOWLEDGE" "$MERGED_KNOWLEDGE")

# For each existing agent
for agent_file in $EXISTING_AGENTS; do
    AGENT_NAME=$(basename "$agent_file" .md)
    AGENT_CONTENT=$(cat "$agent_file")
    
    # Add project-specific context to agent
    # - Project patterns and standards
    # - Project-specific terminology
    # - Project architecture context
    
    UPDATED_CONTENT=$(inject_project_context "$AGENT_CONTENT" "$PROJECT_CONTEXT")
    echo "$UPDATED_CONTENT" > "$agent_file"
    echo "✅ Updated agent with project context: $AGENT_NAME"
done
```

### Step 4: Simplify Agents for Simple Projects

For simple projects, remove or combine unnecessary agents:

```bash
if [ "$PROJECT_NATURE" = "simple" ]; then
    echo "📋 Simplifying agents for simple project..."
    
    # For simple projects, some specialized agents may not be needed
    # Instead of removing, we can mark them as optional or combine functionality
    
    # List of agents that can be combined for simple projects
    COMBINABLE_AGENTS=(
        "researcher"
        "reviewer"
    )
    
    for optional in "${COMBINABLE_AGENTS[@]}"; do
        OPTIONAL_FILE="agent-os/agents/${optional}.md"
        if [ -f "$OPTIONAL_FILE" ]; then
            echo "ℹ️  Agent marked as optional for simple project: $optional"
        fi
    done
    
    echo "✅ Agents simplified for simple project"
fi
```

### Step 5: Generate Layer-Specialist Agents

Generate specialized agents for each detected abstraction layer from basepoints:

```bash
echo "📋 Generating layer-specialist agents..."

# Extract abstraction layers from headquarter.md
if [ -f "agent-os/basepoints/headquarter.md" ]; then
    # Parse detected layers from headquarter
    DETECTED_LAYERS=$(grep -A 50 "Detected Abstraction Layers" agent-os/basepoints/headquarter.md | \
        grep -E "^\| \*\*[A-Z]+" | \
        sed 's/.*\*\*\([A-Z_]*\)\*\*.*/\1/' | \
        tr '[:upper:]' '[:lower:]')
    
    echo "Detected layers: $DETECTED_LAYERS"
    
    # Create layer-specialist registry
    mkdir -p agent-os/agents/specialists
    
    SPECIALISTS_CREATED=0
    
    for layer in $DETECTED_LAYERS; do
        # Skip meta layers that don't need specialists
        if [[ "$layer" =~ ^(root|documentation|config)$ ]]; then
            continue
        fi
        
        # Generate specialist name
        SPECIALIST_NAME="${layer}-specialist"
        SPECIALIST_FILE="agent-os/agents/specialists/${SPECIALIST_NAME}.md"
        
        # Extract layer-specific patterns from basepoints
        LAYER_PATTERNS=""
        LAYER_STANDARDS=""
        LAYER_CONTEXT=""
        
        # Find module basepoints for this layer
        if [ -d "agent-os/basepoints/modules" ]; then
            LAYER_MODULES=$(find agent-os/basepoints/modules -name "*.md" -exec grep -l -i "$layer" {} \; 2>/dev/null | head -5)
            if [ -n "$LAYER_MODULES" ]; then
                LAYER_CONTEXT="Reference these basepoints for ${layer} layer patterns:\n"
                for module in $LAYER_MODULES; do
                    LAYER_CONTEXT="${LAYER_CONTEXT}- @agent-os/$(echo $module | sed 's|^agent-os/||')\n"
                done
            fi
        fi
        
        # Map layer to relevant standards
        case "$layer" in
            ui|frontend|presentation|view)
                LAYER_STANDARDS="@agent-os/standards/global/conventions.md"
                LAYER_FOCUS="UI components, user interactions, visual presentation, accessibility"
                ;;
            api|backend|service|logic)
                LAYER_STANDARDS="@agent-os/standards/global/conventions.md\n@agent-os/standards/quality/assurance.md"
                LAYER_FOCUS="Business logic, API endpoints, service orchestration, data validation"
                ;;
            data|database|persistence|storage)
                LAYER_STANDARDS="@agent-os/standards/global/conventions.md"
                LAYER_FOCUS="Data models, database operations, migrations, caching strategies"
                ;;
            platform|infrastructure|system)
                LAYER_STANDARDS="@agent-os/standards/global/conventions.md\n@agent-os/standards/quality/assurance.md"
                LAYER_FOCUS="Platform-specific code, system integration, deployment configuration"
                ;;
            test|testing|quality)
                LAYER_STANDARDS="@agent-os/standards/testing/test-writing.md\n@agent-os/standards/quality/assurance.md"
                LAYER_FOCUS="Test implementation, coverage, quality validation, test patterns"
                ;;
            *)
                LAYER_STANDARDS="@agent-os/standards/global/conventions.md"
                LAYER_FOCUS="Domain-specific implementation for ${layer} layer"
                ;;
        esac
        
        # Generate the specialist agent file
        cat > "$SPECIALIST_FILE" << EOF
---
name: ${SPECIALIST_NAME}
description: Specialist for ${layer} layer implementation. Use for tasks targeting the ${layer} abstraction layer.
tools: Write, Read, Bash, WebFetch, Playwright
color: blue
model: inherit
---

You are a specialist developer with deep expertise in the **${layer}** abstraction layer. Your role is to implement tasks that specifically target this layer while maintaining consistency with the project's established patterns.

## Layer Focus

${LAYER_FOCUS}

## Layer-Specific Context

$(echo -e "$LAYER_CONTEXT")

## Implementation Workflow

{{workflows/implementation/implement-tasks}}

## Layer-Aware Guidelines

1. **Stay in your layer**: Focus on ${layer}-layer concerns. If a task requires crossing layers, flag it for coordination.

2. **Use layer patterns**: Reference the basepoints for this layer to maintain consistency:
$(echo -e "$LAYER_CONTEXT")

3. **Layer boundaries**: Understand how this layer interfaces with adjacent layers. Don't bleed concerns across boundaries.

4. **Layer-specific validation**: After implementation, verify:
   - Code follows ${layer} layer patterns from basepoints
   - No unintended layer boundary violations
   - Standards compliance for this layer

## Standards

$(echo -e "$LAYER_STANDARDS")

{{UNLESS standards_as_claude_code_skills}}
## Additional Standards

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
EOF
        
        echo "✅ Created layer specialist: ${SPECIALIST_NAME}"
        ((SPECIALISTS_CREATED++))
    done
    
    # Create specialist registry for orchestration
    cat > "agent-os/agents/specialists/registry.yml" << EOF
# Layer Specialist Registry
# Auto-generated during deploy-agents
# Used by orchestrate-tasks to suggest specialists for task groups

specialists:
$(for layer in $DETECTED_LAYERS; do
    if [[ ! "$layer" =~ ^(root|documentation|config)$ ]]; then
        echo "  - name: ${layer}-specialist"
        echo "    layer: ${layer}"
        echo "    file: agent-os/agents/specialists/${layer}-specialist.md"
    fi
done)

# Layer keyword mapping for auto-detection
layer_keywords:
  ui:
    - component
    - view
    - screen
    - button
    - form
    - modal
    - layout
    - style
    - css
    - render
  api:
    - endpoint
    - route
    - controller
    - handler
    - request
    - response
    - middleware
  data:
    - model
    - schema
    - migration
    - query
    - database
    - repository
    - entity
  platform:
    - ios
    - android
    - native
    - device
    - system
    - config
  test:
    - test
    - spec
    - mock
    - fixture
    - coverage
EOF
    
    echo "✅ Created specialist registry: agent-os/agents/specialists/registry.yml"
    echo "📊 Total specialists created: $SPECIALISTS_CREATED"
else
    echo "⚠️  No headquarter.md found - skipping layer specialist generation"
    echo "   Run /create-basepoints first to detect abstraction layers"
fi
```

### Step 5b: Create Project-Specific Agents

Create additional agents based on unique project needs:

```bash
# Analyze if project needs specialized agents beyond layer specialists
# - Unique domain patterns (e.g., payment-specialist, auth-specialist)
# - Specific technology requirements (e.g., graphql-specialist)
# - Complex domain patterns

PROJECT_AGENT_NEEDS=$(analyze_agent_needs "$BASEPOINTS_KNOWLEDGE")

if [ -n "$PROJECT_AGENT_NEEDS" ]; then
    for agent_need in $PROJECT_AGENT_NEEDS; do
        AGENT_NAME=$(echo "$agent_need" | cut -d':' -f1)
        AGENT_PURPOSE=$(echo "$agent_need" | cut -d':' -f2-)
        
        cat > "agent-os/agents/${AGENT_NAME}.md" << EOF
---
name: ${AGENT_NAME}
description: ${AGENT_PURPOSE}
tools: Write, Read, Bash, WebFetch, Playwright
color: purple
model: inherit
---

You are a specialist with expertise in ${AGENT_NAME//-/ }. Your role is to handle tasks specifically related to this domain.

## Purpose

${AGENT_PURPOSE}

## Responsibilities

1. Implement features related to this domain
2. Maintain consistency with established patterns
3. Follow project standards and conventions

## Context

This agent was created based on patterns detected in the project's codebase during deploy-agents specialization.

## Implementation Workflow

{{workflows/implementation/implement-tasks}}

## Guidelines

- Follow project standards and conventions
- Reference basepoints knowledge for patterns
- Maintain consistency with project architecture

{{UNLESS standards_as_claude_code_skills}}
{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
EOF
        
        echo "✅ Created project-specific agent: ${AGENT_NAME}"
    done
fi
```

### Step 6: Verify Agents Consistency

Verify all agents are consistent and properly configured:

```bash
# Count agents
CORE_AGENTS_COUNT=$(find agent-os/agents -maxdepth 1 -name "*.md" -type f | wc -l | tr -d ' ')
SPECIALIST_COUNT=$(find agent-os/agents/specialists -name "*.md" -type f 2>/dev/null | wc -l | tr -d ' ')
TOTAL_AGENTS=$((CORE_AGENTS_COUNT + SPECIALIST_COUNT))

# Log to reports
mkdir -p agent-os/output/deploy-agents/reports
cat > agent-os/output/deploy-agents/reports/agents-specialization.md << EOF
# Agents Specialization Report

## Summary
- Project Nature: $PROJECT_NATURE
- Core Agents: $CORE_AGENTS_COUNT
- Layer Specialists: $SPECIALIST_COUNT
- Total Agents: $TOTAL_AGENTS

## Core Agents
$(find agent-os/agents -maxdepth 1 -name "*.md" -type f | while read f; do
    echo "- $(basename "$f" .md)"
done)

## Layer Specialists
$(if [ -d "agent-os/agents/specialists" ]; then
    find agent-os/agents/specialists -name "*.md" -type f | while read f; do
        echo "- $(basename "$f" .md)"
    done
else
    echo "- (none generated - run /create-basepoints first)"
fi)

## Specialist Registry
$(if [ -f "agent-os/agents/specialists/registry.yml" ]; then
    echo "✅ Registry created at: agent-os/agents/specialists/registry.yml"
    echo ""
    echo "Layer mappings:"
    grep -A1 "^  - name:" agent-os/agents/specialists/registry.yml | \
        grep -E "name:|layer:" | \
        sed 's/^  /    /'
else
    echo "⚠️ No registry - specialists will use default layer detection"
fi)

## Usage in Workflows

Layer specialists are automatically used by:
- \`/orchestrate-tasks\` - Suggests specialists based on task layer
- \`/implement-tasks\` - Delegates to appropriate specialist

### How it works:
1. Task group content is analyzed for layer keywords
2. Matching specialist is selected (or falls back to implementer)
3. Specialist has layer-specific basepoints context
4. Implementation stays consistent with layer patterns

## Actions Taken
- Updated core agents with project context from basepoints
- Generated layer specialists from detected abstraction layers
- Created specialist registry for orchestration
- Simplified agents for project complexity level

## Next Steps
Proceed to specialize workflows in phase 10.
EOF

echo "✅ Agents specialization complete"
echo "📊 Core agents: $CORE_AGENTS_COUNT | Layer specialists: $SPECIALIST_COUNT"
echo "📁 Report saved to: agent-os/output/deploy-agents/reports/agents-specialization.md"
```

{{UNLESS compiled_single_command}}
## Display confirmation and next step

Once you've specialized agents, output the following message:

```
✅ Agents specialized for project!

**Project Nature:** [simple/moderate/complex]
**Agents Updated:** [count]
**Project-Specific Agents Created:** [count]

Report: `agent-os/output/deploy-agents/reports/agents-specialization.md`

NEXT STEP 👉 Run the command, `10-specialize-workflows.md`
```
{{ENDUNLESS compiled_single_command}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

Ensure agents specialization follows the user's preferences:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
