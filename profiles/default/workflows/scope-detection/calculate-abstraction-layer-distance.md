# Abstraction Layer Distance Calculation

## Core Responsibilities

1. **Determine Abstraction Layer Distance**: Calculate distance from spec context to actual implementation
2. **Calculate Distance Metrics**: Create distance metrics for deep reading decisions
3. **Create Heuristics**: Define heuristics for when deep implementation reading is needed
4. **Base on Project Structure**: Use project structure (after agent-deployment) for calculations
5. **Store Calculation Results**: Cache distance calculation results

## Workflow

### Step 1: Load Scope Detection Results

Load previous scope detection results:

```bash
# Determine spec path and cache path
SPEC_PATH="{{SPEC_PATH}}"
if [ -z "$SPEC_PATH" ]; then
    SPEC_PATH="geist/specs/[current-spec]"
fi

if [ -n "$SPEC_PATH" ]; then
    CACHE_PATH="$SPEC_PATH/implementation/cache"
else
    CACHE_PATH="geist/output/deploy-agents/knowledge"
fi

# Load scope detection results
if [ -f "$CACHE_PATH/scope-detection/semantic-analysis.json" ]; then
    SEMANTIC_RESULTS=$(cat "$CACHE_PATH/scope-detection/semantic-analysis.json")
fi

if [ -f "$CACHE_PATH/scope-detection/keyword-matching.json" ]; then
    KEYWORD_RESULTS=$(cat "$CACHE_PATH/scope-detection/keyword-matching.json")
fi

if [ -f "$CACHE_PATH/scope-detection/cross-layer-detection.json" ]; then
    CROSS_LAYER_RESULTS=$(cat "$CACHE_PATH/scope-detection/cross-layer-detection.json")
fi

if [ -f "$CACHE_PATH/scope-detection/same-layer-detection.json" ]; then
    SAME_LAYER_RESULTS=$(cat "$CACHE_PATH/scope-detection/same-layer-detection.json")
fi
```

### Step 2: Determine Spec Context Abstraction Layer

Identify the abstraction layer of the spec context:

```bash
# Determine spec context layer from scope detection
SPEC_CONTEXT_LAYER=$({{DETERMINE_SPEC_CONTEXT_LAYER}})

# If spec spans multiple layers, identify the highest/primary layer
if {{SPEC_SPANS_MULTIPLE_LAYERS}}; then
    PRIMARY_LAYER=$({{IDENTIFY_PRIMARY_LAYER}})
    SPEC_CONTEXT_LAYER="$PRIMARY_LAYER"
fi

# If spec is at product/architecture level, mark as highest abstraction
if {{IS_PRODUCT_LEVEL}}; then
    SPEC_CONTEXT_LAYER="product"
fi
```

### Step 3: Load Project Structure and Abstraction Layers

Load project structure for distance calculation:

```bash
# Load abstraction layers from basepoints
BASEPOINTS_PATH="{{BASEPOINTS_PATH}}"

# Read headquarter.md for layer structure
if [ -f "$BASEPOINTS_PATH/headquarter.md" ]; then
    HEADQUARTER=$(cat "$BASEPOINTS_PATH/headquarter.md")
    ABSTRACTION_LAYERS=$({{EXTRACT_ABSTRACTION_LAYERS}})
fi

# Create layer hierarchy (ordered from highest to lowest abstraction)
LAYER_HIERARCHY=$({{CREATE_LAYER_HIERARCHY}})

# Example hierarchy: product > architecture > domain > data > infrastructure > implementation
```

### Step 4: Calculate Distance from Spec Context to Implementation

Calculate distance metrics:

```bash
# Calculate distance for each relevant layer
DISTANCE_METRICS=""
echo "$ABSTRACTION_LAYERS" | while read layer; do
    if [ -z "$layer" ]; then
        continue
    fi
    
    # Calculate distance from spec context to this layer
    DISTANCE=$({{CALCULATE_LAYER_DISTANCE}})
    
    # Calculate distance to actual implementation (lowest layer)
    IMPLEMENTATION_DISTANCE=$({{CALCULATE_IMPLEMENTATION_DISTANCE}})
    
    DISTANCE_METRICS="$DISTANCE_METRICS\n${layer}:${DISTANCE}:${IMPLEMENTATION_DISTANCE}"
done

# Calculate overall distance to implementation
OVERALL_DISTANCE=$({{CALCULATE_OVERALL_DISTANCE}})
```

### Step 5: Create Heuristics for Deep Reading Decisions

Define when deep reading is needed:

```bash
# Create heuristics based on distance
DEEP_READING_HEURISTICS=""

# Higher abstraction = less need for implementation reading
if [ "$OVERALL_DISTANCE" -ge 3 ]; then
    DEEP_READING_NEEDED="low"
    DEEP_READING_HEURISTICS="$DEEP_READING_HEURISTICS\nhigh_abstraction:low_need"
fi

# One or two layers above implementation = more need
if [ "$OVERALL_DISTANCE" -le 2 ] && [ "$OVERALL_DISTANCE" -ge 1 ]; then
    DEEP_READING_NEEDED="high"
    DEEP_READING_HEURISTICS="$DEEP_READING_HEURISTICS\nclose_to_implementation:high_need"
fi

# At implementation layer = very high need
if [ "$OVERALL_DISTANCE" -eq 0 ]; then
    DEEP_READING_NEEDED="very_high"
    DEEP_READING_HEURISTICS="$DEEP_READING_HEURISTICS\nat_implementation:very_high_need"
fi

# Create decision rules
DEEP_READING_DECISION=$({{CREATE_DEEP_READING_DECISION}})
```

Heuristics:
- **High Abstraction (3+ layers away)**: Low need for deep reading
- **Medium Abstraction (1-2 layers away)**: High need for deep reading
- **At Implementation Layer**: Very high need for deep reading
- **Cross-Layer Patterns**: May need deep reading for understanding interactions

### Step 6: Base Distance Calculations on Project Structure

Use actual project structure (after agent-deployment):

```bash
# After agent-deployment, use actual project structure
if {{AFTER_AGENT_DEPLOYMENT}}; then
    # Use actual project structure from basepoints
    PROJECT_STRUCTURE=$({{LOAD_PROJECT_STRUCTURE}})
    
    # Calculate distances based on actual structure
    ACTUAL_DISTANCES=$({{CALCULATE_ACTUAL_DISTANCES}})
    
    # Update heuristics based on actual structure
    DEEP_READING_HEURISTICS=$({{UPDATE_HEURISTICS_FOR_STRUCTURE}})
else
    # Before deployment, use generic/placeholder calculations
    GENERIC_DISTANCES=$({{CALCULATE_GENERIC_DISTANCES}})
    DEEP_READING_HEURISTICS=$({{CREATE_GENERIC_HEURISTICS}})
fi
```

### Step 7: Store Calculation Results

Cache distance calculation results:

```bash
mkdir -p "$CACHE_PATH/scope-detection"

# Store distance calculation results
cat > "$CACHE_PATH/scope-detection/abstraction-layer-distance.json" << EOF
{
  "spec_context_layer": "$SPEC_CONTEXT_LAYER",
  "abstraction_layers": $(echo "$ABSTRACTION_LAYERS" | {{JSON_FORMAT}}),
  "layer_hierarchy": $(echo "$LAYER_HIERARCHY" | {{JSON_FORMAT}}),
  "distance_metrics": $(echo "$DISTANCE_METRICS" | {{JSON_FORMAT}}),
  "overall_distance": "$OVERALL_DISTANCE",
  "deep_reading_needed": "$DEEP_READING_NEEDED",
  "deep_reading_heuristics": $(echo "$DEEP_READING_HEURISTICS" | {{JSON_FORMAT}}),
  "deep_reading_decision": "$DEEP_READING_DECISION"
}
EOF

# Also create human-readable summary
cat > "$CACHE_PATH/scope-detection/abstraction-layer-distance-summary.md" << EOF
# Abstraction Layer Distance Calculation Results

## Spec Context Layer
[The abstraction layer where the spec context resides]

## Abstraction Layer Hierarchy
[Ordered list of abstraction layers from highest to lowest]

## Distance Metrics

### By Layer
[Distance from spec context to each layer]

### Overall Distance to Implementation
[Overall distance to actual implementation]

## Deep Reading Decision

### Need Level
[Level of need for deep implementation reading: low/high/very_high]

### Heuristics Applied
[Summary of heuristics used to determine deep reading need]

### Decision
[Final decision on whether deep reading is needed]
EOF
```

## Important Constraints

- Must determine abstraction layer distance from spec context to actual implementation
- Must calculate distance metrics for deep reading decisions
- Must create heuristics for when deep implementation reading is needed
- Must base distance calculations on project structure (after agent-deployment)
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
- Must be technology-agnostic and work with any basepoint structure
- **CRITICAL**: All calculation results must be stored in `geist/specs/[current-spec]/implementation/cache/scope-detection/`, not scattered around the codebase
