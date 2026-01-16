# Detect Task Layer Workflow

This workflow analyzes task content to determine which abstraction layer it targets, enabling automatic specialist agent assignment.

## Purpose

- Analyze task group content for layer-specific keywords
- Match tasks to appropriate layer specialists
- Enable intelligent agent assignment in orchestration

## Layer Detection Logic

```bash
#!/bin/bash

# Layer keyword mappings
# Each layer has associated keywords that indicate task relevance

declare -A LAYER_KEYWORDS=(
    # UI/Frontend Layer
    ["ui"]="component view screen button form modal layout style css render widget template jsx tsx html svg icon animation transition theme responsive mobile desktop tablet"
    
    # API/Backend Layer  
    ["api"]="endpoint route controller handler request response middleware api rest graphql mutation query resolver service webhook authentication authorization token jwt session"
    
    # Data/Database Layer
    ["data"]="model schema migration query database repository entity storage persistence table column index relation foreign key primary cache redis mongo postgres mysql sqlite orm"
    
    # Platform/Infrastructure Layer
    ["platform"]="ios android native device system config platform infrastructure docker kubernetes deployment ci cd pipeline environment variable secret certificate ssl tls nginx apache"
    
    # Testing Layer
    ["test"]="test spec mock fixture coverage assertion expect describe it beforeEach afterEach jest mocha cypress playwright snapshot integration unit e2e"
    
    # Logic/Domain Layer
    ["logic"]="service usecase interactor domain business rule validation transform calculate process workflow state machine event handler"
)

# Detect layer from task content
detect_layer() {
    local task_content="$1"
    local task_lower=$(echo "$task_content" | tr '[:upper:]' '[:lower:]')
    
    local best_layer="logic"  # Default layer
    local best_score=0
    
    for layer in "${!LAYER_KEYWORDS[@]}"; do
        local score=0
        for keyword in ${LAYER_KEYWORDS[$layer]}; do
            # Count occurrences of keyword
            local count=$(echo "$task_lower" | grep -o "\b$keyword\b" | wc -l)
            score=$((score + count))
        done
        
        if [ $score -gt $best_score ]; then
            best_score=$score
            best_layer=$layer
        fi
    done
    
    # Return layer and confidence
    if [ $best_score -ge 3 ]; then
        echo "${best_layer}:high"
    elif [ $best_score -ge 1 ]; then
        echo "${best_layer}:medium"
    else
        echo "mixed:low"
    fi
}

# Map layer to specialist agent
layer_to_specialist() {
    local layer="$1"
    
    case "$layer" in
        ui|frontend|presentation|view)
            echo "ui-specialist"
            ;;
        api|backend|service)
            echo "api-specialist"
            ;;
        data|database|persistence|storage)
            echo "data-specialist"
            ;;
        platform|infrastructure|system)
            echo "platform-specialist"
            ;;
        test|testing|quality)
            echo "test-specialist"
            ;;
        logic|domain|business)
            echo "logic-specialist"
            ;;
        *)
            echo "implementer"
            ;;
    esac
}

# Check if specialist exists
specialist_exists() {
    local specialist="$1"
    local specialist_file="agent-os/agents/specialists/${specialist}.md"
    
    if [ -f "$specialist_file" ]; then
        return 0
    else
        return 1
    fi
}

# Get best available agent for layer
get_agent_for_layer() {
    local layer="$1"
    local specialist=$(layer_to_specialist "$layer")
    
    if specialist_exists "$specialist"; then
        echo "$specialist"
    else
        # Fall back to generic implementer
        echo "implementer"
    fi
}
```

## Usage in Orchestration

When analyzing tasks for agent assignment:

```bash
# Read task group content
TASK_CONTENT=$(get_task_group_content "$TASK_GROUP_NAME")

# Detect layer
LAYER_RESULT=$(detect_layer "$TASK_CONTENT")
DETECTED_LAYER=$(echo "$LAYER_RESULT" | cut -d':' -f1)
CONFIDENCE=$(echo "$LAYER_RESULT" | cut -d':' -f2)

# Get appropriate agent
SUGGESTED_AGENT=$(get_agent_for_layer "$DETECTED_LAYER")

echo "Task: $TASK_GROUP_NAME"
echo "Detected Layer: $DETECTED_LAYER (confidence: $CONFIDENCE)"
echo "Suggested Agent: $SUGGESTED_AGENT"
```

## Layer Hierarchy

```
┌─────────────────────────────────────────────────────────────┐
│                    ABSTRACTION LAYERS                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐                                            │
│  │      UI      │  ← ui-specialist                           │
│  │  Components  │    (views, components, styles)             │
│  └──────┬───────┘                                            │
│         │                                                    │
│  ┌──────▼───────┐                                            │
│  │    Logic     │  ← logic-specialist                        │
│  │   Domain     │    (business rules, use cases)             │
│  └──────┬───────┘                                            │
│         │                                                    │
│  ┌──────▼───────┐                                            │
│  │     API      │  ← api-specialist                          │
│  │   Services   │    (endpoints, handlers)                   │
│  └──────┬───────┘                                            │
│         │                                                    │
│  ┌──────▼───────┐                                            │
│  │    Data      │  ← data-specialist                         │
│  │  Persistence │    (models, queries, migrations)           │
│  └──────┬───────┘                                            │
│         │                                                    │
│  ┌──────▼───────┐                                            │
│  │   Platform   │  ← platform-specialist                     │
│  │Infrastructure│    (config, deployment, native)            │
│  └──────────────┘                                            │
│                                                              │
│  ┌──────────────┐                                            │
│  │    Test      │  ← test-specialist                         │
│  │   Quality    │    (all layers, cross-cutting)             │
│  └──────────────┘                                            │
│                                                              │
│  ┌──────────────┐                                            │
│  │   Mixed/     │  ← implementer                             │
│  │   General    │    (cross-layer, integration)              │
│  └──────────────┘                                            │
└─────────────────────────────────────────────────────────────┘
```

## Output Format

The workflow outputs a JSON-like structure for each task group:

```json
{
  "task_group": "user-profile-component",
  "detected_layer": "ui",
  "confidence": "high",
  "suggested_agent": "ui-specialist",
  "fallback_agent": "implementer",
  "keywords_matched": ["component", "view", "style", "render"]
}
```
