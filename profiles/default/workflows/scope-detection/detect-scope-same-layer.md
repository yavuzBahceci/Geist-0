# Same-Layer Scope Detection

## Core Responsibilities

1. **Detect Modules Within Same Abstraction Layer**: Identify modules within the same layer relevant to spec
2. **Identify Module-Specific Patterns**: Find module-specific patterns within detected layers
3. **Map to Module-Specific Basepoints**: Map detected scope to module-specific basepoint files
4. **Extract Same-Layer Patterns**: Extract same-layer patterns relevant to spec context
5. **Store Detection Results**: Cache same-layer detection results

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
    DETECTED_LAYERS=$({{EXTRACT_DETECTED_LAYERS}})
fi

# Load keyword matching results
if [ -f "$CACHE_PATH/scope-detection/keyword-matching.json" ]; then
    KEYWORD_RESULTS=$(cat "$CACHE_PATH/scope-detection/keyword-matching.json")
    MATCHED_LAYERS=$({{EXTRACT_MATCHED_LAYERS}})
fi

# Combine detected layers
RELEVANT_LAYERS=$({{COMBINE_LAYERS}})
```

### Step 2: Detect Modules Within Same Abstraction Layer

Identify modules within each relevant layer:

```bash
# Load basepoint files
BASEPOINTS_PATH="{{BASEPOINTS_PATH}}"
ALL_BASEPOINTS=$(find "$BASEPOINTS_PATH" -name "{{BASEPOINT_FILE_PATTERN}}" -type f)

# Group modules by layer
LAYER_MODULES=""
echo "$RELEVANT_LAYERS" | while read layer; do
    if [ -z "$layer" ]; then
        continue
    fi
    
    # Find modules in this layer
    LAYER_MODULE_LIST=""
    echo "$ALL_BASEPOINTS" | while read basepoint_file; do
        if [ -z "$basepoint_file" ]; then
            continue
        fi
        
        MODULE_LAYER=$({{EXTRACT_MODULE_LAYER}})
        if [ "$MODULE_LAYER" = "$layer" ]; then
            MODULE_NAME=$({{EXTRACT_MODULE_NAME}})
            LAYER_MODULE_LIST="$LAYER_MODULE_LIST\n$MODULE_NAME|$basepoint_file"
        fi
    done
    
    LAYER_MODULES="$LAYER_MODULES\n${layer}:${LAYER_MODULE_LIST}"
done
```

### Step 3: Identify Module-Specific Patterns Within Detected Layers

Extract patterns specific to each module:

```bash
# Extract module-specific patterns
MODULE_PATTERNS=""
echo "$LAYER_MODULES" | while IFS=':' read -r layer module_list; do
    if [ -z "$layer" ] || [ -z "$module_list" ]; then
        continue
    fi
    
    echo "$module_list" | while IFS='|' read -r module_name basepoint_file; do
        if [ -z "$module_name" ] || [ -z "$basepoint_file" ]; then
            continue
        fi
        
        MODULE_CONTENT=$(cat "$basepoint_file")
        
        # Extract design patterns
        DESIGN_PATTERNS=$({{EXTRACT_DESIGN_PATTERNS}})
        
        # Extract coding patterns
        CODING_PATTERNS=$({{EXTRACT_CODING_PATTERNS}})
        
        # Extract architectural patterns
        ARCH_PATTERNS=$({{EXTRACT_ARCHITECTURAL_PATTERNS}})
        
        # Extract standards
        STANDARDS=$({{EXTRACT_STANDARDS}})
        
        # Extract flows
        FLOWS=$({{EXTRACT_FLOWS}})
        
        MODULE_PATTERNS="$MODULE_PATTERNS\n${layer}:${module_name}:${DESIGN_PATTERNS}:${CODING_PATTERNS}:${ARCH_PATTERNS}:${STANDARDS}:${FLOWS}"
    done
done
```

### Step 4: Map Detected Scope to Module-Specific Basepoint Files

Create mapping from spec context to module basepoints:

```bash
# Create module basepoint mapping
MODULE_BASEPOINT_MAPPING=""
echo "$LAYER_MODULES" | while IFS=':' read -r layer module_list; do
    if [ -z "$layer" ] || [ -z "$module_list" ]; then
        continue
    fi
    
    echo "$module_list" | while IFS='|' read -r module_name basepoint_file; do
        if [ -z "$module_name" ] || [ -z "$basepoint_file" ]; then
            continue
        fi
        
        # Check if module is relevant to spec context
        if {{CHECK_MODULE_RELEVANCE}}; then
            MODULE_BASEPOINT_MAPPING="$MODULE_BASEPOINT_MAPPING\n${layer}:${module_name}:${basepoint_file}"
        fi
    done
done
```

### Step 5: Extract Same-Layer Patterns Relevant to Spec Context

Extract patterns relevant to the current spec:

```bash
# Filter patterns based on spec context
RELEVANT_SAME_LAYER_PATTERNS=""
echo "$MODULE_PATTERNS" | while IFS=':' read -r layer module_name design coding arch standards flows; do
    if [ -z "$layer" ] || [ -z "$module_name" ]; then
        continue
    fi
    
    # Check if patterns are relevant to spec
    if {{CHECK_PATTERN_RELEVANCE}}; then
        RELEVANT_SAME_LAYER_PATTERNS="$RELEVANT_SAME_LAYER_PATTERNS\n${layer}:${module_name}:${design}:${coding}:${arch}:${standards}:${flows}"
    fi
done
```

### Step 6: Store Detection Results

Cache same-layer detection results:

```bash
mkdir -p "$CACHE_PATH/scope-detection"

# Store same-layer detection results
cat > "$CACHE_PATH/scope-detection/same-layer-detection.json" << EOF
{
  "relevant_layers": $(echo "$RELEVANT_LAYERS" | {{JSON_FORMAT}}),
  "layer_modules": $(echo "$LAYER_MODULES" | {{JSON_FORMAT}}),
  "module_patterns": $(echo "$MODULE_PATTERNS" | {{JSON_FORMAT}}),
  "module_basepoint_mapping": $(echo "$MODULE_BASEPOINT_MAPPING" | {{JSON_FORMAT}}),
  "relevant_patterns": $(echo "$RELEVANT_SAME_LAYER_PATTERNS" | {{JSON_FORMAT}})
}
EOF

# Also create human-readable summary
cat > "$CACHE_PATH/scope-detection/same-layer-detection-summary.md" << EOF
# Same-Layer Scope Detection Results

## Relevant Abstraction Layers
[Summary of relevant layers detected]

## Modules by Layer

### Layer: [Layer Name]
[List of modules in this layer]

## Module-Specific Patterns

### [Module Name]
- Design Patterns: [patterns]
- Coding Patterns: [patterns]
- Architectural Patterns: [patterns]
- Standards: [standards]
- Flows: [flows]

## Relevant Basepoints
[List of relevant module-specific basepoint files]
EOF
```

## Important Constraints

- Must detect modules within same abstraction layer relevant to spec
- Must identify module-specific patterns within detected layers
- Must map detected scope to module-specific basepoint files
- Must extract same-layer patterns relevant to spec context
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
- Must be technology-agnostic and work with any basepoint structure
- **CRITICAL**: All detection results must be stored in `geist/specs/[current-spec]/implementation/cache/scope-detection/`, not scattered around the codebase
