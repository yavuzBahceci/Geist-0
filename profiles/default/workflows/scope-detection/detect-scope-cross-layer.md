# Cross-Layer Scope Detection

## Core Responsibilities

1. **Detect Architectural Patterns**: Identify patterns that span multiple abstraction layers
2. **Identify Cross-Layer Dependencies**: Find dependencies and interactions between layers
3. **Map to Headquarter and Parent Basepoints**: Map detected scope to headquarter.md and parent basepoints
4. **Extract Cross-Layer Patterns**: Extract cross-layer patterns relevant to spec context
5. **Store Detection Results**: Cache cross-layer detection results

## Workflow

### Step 1: Load Semantic and Keyword Detection Results

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

# Load semantic analysis results
if [ -f "$CACHE_PATH/scope-detection/semantic-analysis.json" ]; then
    SEMANTIC_RESULTS=$(cat "$CACHE_PATH/scope-detection/semantic-analysis.json")
fi

# Load keyword matching results
if [ -f "$CACHE_PATH/scope-detection/keyword-matching.json" ]; then
    KEYWORD_RESULTS=$(cat "$CACHE_PATH/scope-detection/keyword-matching.json")
fi
```

### Step 2: Detect Architectural Patterns Spanning Multiple Layers

Identify patterns that span multiple abstraction layers:

```bash
# Load abstraction layers
ABSTRACTION_LAYERS=$({{LOAD_ABSTRACTION_LAYERS}})

# Detect cross-layer architectural patterns
CROSS_LAYER_PATTERNS=""

# Check for layered architecture patterns
if {{DETECT_LAYERED_ARCHITECTURE}}; then
    CROSS_LAYER_PATTERNS="$CROSS_LAYER_PATTERNS\nlayered_architecture"
fi

# Check for hexagonal architecture patterns
if {{DETECT_HEXAGONAL_ARCHITECTURE}}; then
    CROSS_LAYER_PATTERNS="$CROSS_LAYER_PATTERNS\nhexagonal_architecture"
fi

# Check for microservices patterns
if {{DETECT_MICROSERVICES}}; then
    CROSS_LAYER_PATTERNS="$CROSS_LAYER_PATTERNS\nmicroservices"
fi

# Check for event-driven patterns
if {{DETECT_EVENT_DRIVEN}}; then
    CROSS_LAYER_PATTERNS="$CROSS_LAYER_PATTERNS\nevent_driven"
fi

# Check for other cross-layer patterns
CUSTOM_PATTERNS=$({{DETECT_CUSTOM_CROSS_LAYER_PATTERNS}})
CROSS_LAYER_PATTERNS="$CROSS_LAYER_PATTERNS\n$CUSTOM_PATTERNS"
```

Detect patterns:
- **Layered Architecture**: Clear separation between data, domain, presentation layers
- **Hexagonal Architecture**: Ports and adapters pattern
- **Microservices**: Service boundaries spanning layers
- **Event-Driven**: Event flows across layers
- **Custom Patterns**: Project-specific cross-layer patterns

### Step 3: Identify Cross-Layer Dependencies and Interactions

Find dependencies and interactions between layers:

```bash
# Load basepoint files
BASEPOINTS_PATH="{{BASEPOINTS_PATH}}"

# Read headquarter.md for cross-layer architecture
if [ -f "$BASEPOINTS_PATH/headquarter.md" ]; then
    HEADQUARTER=$(cat "$BASEPOINTS_PATH/headquarter.md")
    
    # Extract cross-layer dependencies
    CROSS_LAYER_DEPENDENCIES=$({{EXTRACT_CROSS_LAYER_DEPENDENCIES}})
    
    # Extract cross-layer interactions
    CROSS_LAYER_INTERACTIONS=$({{EXTRACT_CROSS_LAYER_INTERACTIONS}})
fi

# Analyze parent basepoints for aggregation patterns
PARENT_BASEPOINTS=$(find "$BASEPOINTS_PATH" -type d -exec sh -c 'find "$1" -maxdepth 1 -name "{{BASEPOINT_FILE_PATTERN}}" | grep -q . && echo "$1"' _ {} \;)

echo "$PARENT_BASEPOINTS" | while read parent_dir; do
    PARENT_BASEPOINT=$(find "$parent_dir" -maxdepth 1 -name "{{BASEPOINT_FILE_PATTERN}}" | head -1)
    if [ -n "$PARENT_BASEPOINT" ]; then
        PARENT_CONTENT=$(cat "$PARENT_BASEPOINT")
        
        # Extract parent-level cross-layer patterns
        PARENT_PATTERNS=$({{EXTRACT_PARENT_CROSS_LAYER_PATTERNS}})
        CROSS_LAYER_PATTERNS="$CROSS_LAYER_PATTERNS\n$PARENT_PATTERNS"
    fi
done
```

Identify:
- **Dependencies**: Which layers depend on which other layers
- **Interactions**: How layers communicate and interact
- **Data Flows**: How data flows across layers
- **Control Flows**: How control flows across layers

### Step 4: Map Detected Scope to Headquarter and Parent Basepoints

Map cross-layer scope to relevant basepoints:

```bash
# Initialize relevant basepoints
RELEVANT_BASEPOINTS=""

# Always include headquarter.md for cross-layer patterns
if [ -f "$BASEPOINTS_PATH/headquarter.md" ]; then
    RELEVANT_BASEPOINTS="$RELEVANT_BASEPOINTS\n$BASEPOINTS_PATH/headquarter.md"
fi

# Include parent basepoints that contain cross-layer patterns
echo "$PARENT_BASEPOINTS" | while read parent_dir; do
    PARENT_BASEPOINT=$(find "$parent_dir" -maxdepth 1 -name "{{BASEPOINT_FILE_PATTERN}}" | head -1)
    if [ -n "$PARENT_BASEPOINT" ]; then
        if {{CHECK_PARENT_RELEVANCE}}; then
            RELEVANT_BASEPOINTS="$RELEVANT_BASEPOINTS\n$PARENT_BASEPOINT"
        fi
    fi
done
```

### Step 5: Extract Cross-Layer Patterns Relevant to Spec Context

Extract patterns relevant to the current spec:

```bash
# Filter cross-layer patterns based on spec context
RELEVANT_CROSS_LAYER_PATTERNS=""

# Check each pattern against spec context
echo "$CROSS_LAYER_PATTERNS" | while read pattern; do
    if [ -z "$pattern" ]; then
        continue
    fi
    
    if {{CHECK_PATTERN_RELEVANCE}}; then
        RELEVANT_CROSS_LAYER_PATTERNS="$RELEVANT_CROSS_LAYER_PATTERNS\n$pattern"
    fi
done

# Extract specific cross-layer patterns from relevant basepoints
EXTRACTED_PATTERNS=""
echo "$RELEVANT_BASEPOINTS" | while read basepoint_file; do
    if [ -z "$basepoint_file" ]; then
        continue
    fi
    
    CONTENT=$(cat "$basepoint_file")
    
    # Extract architectural patterns
    ARCH_PATTERNS=$({{EXTRACT_ARCHITECTURAL_PATTERNS}})
    
    # Extract dependency flows
    DEPENDENCY_PATTERNS=$({{EXTRACT_DEPENDENCY_PATTERNS}})
    
    # Extract interaction patterns
    INTERACTION_PATTERNS=$({{EXTRACT_INTERACTION_PATTERNS}})
    
    EXTRACTED_PATTERNS="$EXTRACTED_PATTERNS\nFrom: $basepoint_file\nArchitectural: $ARCH_PATTERNS\nDependencies: $DEPENDENCY_PATTERNS\nInteractions: $INTERACTION_PATTERNS"
done
```

### Step 6: Store Detection Results

Cache cross-layer detection results:

```bash
mkdir -p "$CACHE_PATH/scope-detection"

# Store cross-layer detection results
cat > "$CACHE_PATH/scope-detection/cross-layer-detection.json" << EOF
{
  "cross_layer_patterns": $(echo "$CROSS_LAYER_PATTERNS" | {{JSON_FORMAT}}),
  "cross_layer_dependencies": $(echo "$CROSS_LAYER_DEPENDENCIES" | {{JSON_FORMAT}}),
  "cross_layer_interactions": $(echo "$CROSS_LAYER_INTERACTIONS" | {{JSON_FORMAT}}),
  "relevant_basepoints": $(echo "$RELEVANT_BASEPOINTS" | {{JSON_FORMAT}}),
  "relevant_patterns": $(echo "$RELEVANT_CROSS_LAYER_PATTERNS" | {{JSON_FORMAT}}),
  "extracted_patterns": $(echo "$EXTRACTED_PATTERNS" | {{JSON_FORMAT}})
}
EOF

# Also create human-readable summary
cat > "$CACHE_PATH/scope-detection/cross-layer-detection-summary.md" << EOF
# Cross-Layer Scope Detection Results

## Detected Cross-Layer Patterns
[Summary of cross-layer architectural patterns]

## Cross-Layer Dependencies
[Summary of dependencies between layers]

## Cross-Layer Interactions
[Summary of interactions between layers]

## Relevant Basepoints
[List of relevant basepoint files (headquarter.md and parent basepoints)]

## Extracted Cross-Layer Patterns
[Summary of extracted patterns relevant to spec context]
EOF
```

## Important Constraints

- Must detect architectural patterns that span multiple abstraction layers
- Must identify cross-layer dependencies and interactions
- Must map detected scope to headquarter.md and parent basepoints
- Must extract cross-layer patterns relevant to spec context
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
- Must be technology-agnostic and work with any basepoint structure
- **CRITICAL**: All detection results must be stored in `geist/specs/[current-spec]/implementation/cache/scope-detection/`, not scattered around the codebase
