# Deep Implementation Reading

## Core Responsibilities

1. **Determine if Deep Reading is Needed**: Check abstraction layer distance to decide if deep reading is required
2. **Read Implementation Files**: Read actual implementation files from modules referenced in basepoints
3. **Extract Patterns and Logic**: Extract patterns, similar logic, and reusable code from implementation
4. **Identify Reusability Opportunities**: Identify opportunities for making code reusable
5. **Analyze Implementation**: Analyze implementation to understand logic and patterns
6. **Store Reading Results**: Cache deep reading results for use in workflows

## Workflow

### Step 1: Check if Deep Reading is Needed

Check abstraction layer distance to determine if deep reading is needed:

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

# Load abstraction layer distance calculation
if [ -f "$CACHE_PATH/scope-detection/abstraction-layer-distance.json" ]; then
    DISTANCE_RESULTS=$(cat "$CACHE_PATH/scope-detection/abstraction-layer-distance.json")
    DEEP_READING_NEEDED=$({{EXTRACT_DEEP_READING_NEEDED}})
    OVERALL_DISTANCE=$({{EXTRACT_OVERALL_DISTANCE}})
else
    # If distance calculation not available, skip deep reading
    DEEP_READING_NEEDED="unknown"
    OVERALL_DISTANCE="unknown"
fi

# Determine if deep reading should proceed
if [ "$DEEP_READING_NEEDED" = "low" ] || [ "$DEEP_READING_NEEDED" = "unknown" ]; then
    echo "⚠️  Deep reading not needed (abstraction layer distance: $OVERALL_DISTANCE, need level: $DEEP_READING_NEEDED)"
    exit 0
fi

echo "✅ Deep reading needed (abstraction layer distance: $OVERALL_DISTANCE, need level: $DEEP_READING_NEEDED)"
```

### Step 2: Identify Modules Referenced in Basepoints

Find modules that are referenced in relevant basepoints:

```bash
# Load scope detection results
if [ -f "$CACHE_PATH/scope-detection/same-layer-detection.json" ]; then
    SAME_LAYER_RESULTS=$(cat "$CACHE_PATH/scope-detection/same-layer-detection.json")
    RELEVANT_MODULES=$({{EXTRACT_RELEVANT_MODULES}})
fi

if [ -f "$CACHE_PATH/scope-detection/cross-layer-detection.json" ]; then
    CROSS_LAYER_RESULTS=$(cat "$CACHE_PATH/scope-detection/cross-layer-detection.json")
    CROSS_LAYER_MODULES=$({{EXTRACT_CROSS_LAYER_MODULES}})
fi

# Combine relevant modules
ALL_RELEVANT_MODULES=$({{COMBINE_MODULES}})

# Extract module paths from basepoints
MODULE_PATHS=""
echo "$ALL_RELEVANT_MODULES" | while read module_name; do
    if [ -z "$module_name" ]; then
        continue
    fi
    
    # Find actual module path in project
    MODULE_PATH=$({{FIND_MODULE_PATH}})
    
    if [ -n "$MODULE_PATH" ]; then
        MODULE_PATHS="$MODULE_PATHS\n$MODULE_PATH"
    fi
done
```

### Step 3: Read Implementation Files from Modules

Read actual implementation files from identified modules:

```bash
# Read implementation files
IMPLEMENTATION_FILES=""
echo "$MODULE_PATHS" | while read module_path; do
    if [ -z "$module_path" ]; then
        continue
    fi
    
    # Find code files in this module
    # Use {{CODE_FILE_PATTERNS}} placeholder for project-specific file extensions
    CODE_FILES=$(find "$module_path" -type f \( {{CODE_FILE_PATTERNS}} \) ! -path "*/node_modules/*" ! -path "*/.git/*" ! -path "*/build/*" ! -path "*/dist/*")
    
    echo "$CODE_FILES" | while read code_file; do
        if [ -z "$code_file" ]; then
            continue
        fi
        
        # Read file content
        FILE_CONTENT=$(cat "$code_file")
        
        # Store file info
        IMPLEMENTATION_FILES="$IMPLEMENTATION_FILES\n${code_file}|${FILE_CONTENT}"
    done
done
```

### Step 4: Extract Patterns, Similar Logic, and Reusable Code

Analyze implementation files to extract patterns and reusable code:

```bash
# Extract patterns from implementation
EXTRACTED_PATTERNS=""
EXTRACTED_LOGIC=""
REUSABLE_CODE=""

echo "$IMPLEMENTATION_FILES" | while IFS='|' read -r code_file file_content; do
    if [ -z "$code_file" ] || [ -z "$file_content" ]; then
        continue
    fi
    
    # Extract design patterns
    DESIGN_PATTERNS=$({{EXTRACT_DESIGN_PATTERNS}})
    
    # Extract coding patterns
    CODING_PATTERNS=$({{EXTRACT_CODING_PATTERNS}})
    
    # Extract similar logic
    SIMILAR_LOGIC=$({{EXTRACT_SIMILAR_LOGIC}})
    
    # Extract reusable code blocks
    REUSABLE_BLOCKS=$({{EXTRACT_REUSABLE_BLOCKS}})
    
    # Extract functions/methods that could be reused
    REUSABLE_FUNCTIONS=$({{EXTRACT_REUSABLE_FUNCTIONS}})
    
    # Extract classes/modules that could be reused
    REUSABLE_CLASSES=$({{EXTRACT_REUSABLE_CLASSES}})
    
    EXTRACTED_PATTERNS="$EXTRACTED_PATTERNS\n${code_file}:${DESIGN_PATTERNS}:${CODING_PATTERNS}"
    EXTRACTED_LOGIC="$EXTRACTED_LOGIC\n${code_file}:${SIMILAR_LOGIC}"
    REUSABLE_CODE="$REUSABLE_CODE\n${code_file}:${REUSABLE_BLOCKS}:${REUSABLE_FUNCTIONS}:${REUSABLE_CLASSES}"
done
```

### Step 5: Identify Opportunities for Making Code Reusable

Identify opportunities to refactor code for reusability:

```bash
# Identify reusable opportunities
REUSABILITY_OPPORTUNITIES=""

# Detect similar code across modules
SIMILAR_CODE_DETECTED=$({{DETECT_SIMILAR_CODE}})

# Identify core/common patterns
CORE_PATTERNS=$({{IDENTIFY_CORE_PATTERNS}})

# Identify common modules
COMMON_MODULES=$({{IDENTIFY_COMMON_MODULES}})

# Identify opportunities to move code to shared locations
SHARED_LOCATION_OPPORTUNITIES=$({{IDENTIFY_SHARED_LOCATION_OPPORTUNITIES}})

REUSABILITY_OPPORTUNITIES="$REUSABILITY_OPPORTUNITIES\nSimilar Code: ${SIMILAR_CODE_DETECTED}\nCore Patterns: ${CORE_PATTERNS}\nCommon Modules: ${COMMON_MODULES}\nShared Locations: ${SHARED_LOCATION_OPPORTUNITIES}"
```

### Step 6: Analyze Implementation to Understand Logic and Patterns

Analyze implementation files to understand logic and patterns:

```bash
# Analyze implementation logic
IMPLEMENTATION_ANALYSIS=""

echo "$IMPLEMENTATION_FILES" | while IFS='|' read -r code_file file_content; do
    if [ -z "$code_file" ] || [ -z "$file_content" ]; then
        continue
    fi
    
    # Analyze logic flow
    LOGIC_FLOW=$({{ANALYZE_LOGIC_FLOW}})
    
    # Analyze data flow
    DATA_FLOW=$({{ANALYZE_DATA_FLOW}})
    
    # Analyze control flow
    CONTROL_FLOW=$({{ANALYZE_CONTROL_FLOW}})
    
    # Analyze dependencies
    DEPENDENCIES=$({{ANALYZE_DEPENDENCIES}})
    
    # Analyze patterns used
    PATTERNS_USED=$({{ANALYZE_PATTERNS_USED}})
    
    IMPLEMENTATION_ANALYSIS="$IMPLEMENTATION_ANALYSIS\n${code_file}:\n  Logic Flow: ${LOGIC_FLOW}\n  Data Flow: ${DATA_FLOW}\n  Control Flow: ${CONTROL_FLOW}\n  Dependencies: ${DEPENDENCIES}\n  Patterns: ${PATTERNS_USED}"
done
```

### Step 7: Store Deep Reading Results

Cache deep reading results:

```bash
mkdir -p "$CACHE_PATH/deep-reading"

# Store deep reading results
cat > "$CACHE_PATH/deep-reading/implementation-analysis.json" << EOF
{
  "deep_reading_triggered": true,
  "abstraction_layer_distance": "$OVERALL_DISTANCE",
  "need_level": "$DEEP_READING_NEEDED",
  "modules_analyzed": $(echo "$ALL_RELEVANT_MODULES" | {{JSON_FORMAT}}),
  "files_read": $(echo "$IMPLEMENTATION_FILES" | {{JSON_FORMAT}}),
  "extracted_patterns": $(echo "$EXTRACTED_PATTERNS" | {{JSON_FORMAT}}),
  "extracted_logic": $(echo "$EXTRACTED_LOGIC" | {{JSON_FORMAT}}),
  "reusable_code": $(echo "$REUSABLE_CODE" | {{JSON_FORMAT}}),
  "reusability_opportunities": $(echo "$REUSABILITY_OPPORTUNITIES" | {{JSON_FORMAT}}),
  "implementation_analysis": $(echo "$IMPLEMENTATION_ANALYSIS" | {{JSON_FORMAT}})
}
EOF

# Also create human-readable summary
cat > "$CACHE_PATH/deep-reading/implementation-analysis-summary.md" << EOF
# Deep Implementation Reading Results

## Trigger Information
- Abstraction Layer Distance: $OVERALL_DISTANCE
- Need Level: $DEEP_READING_NEEDED
- Deep Reading Triggered: Yes

## Modules Analyzed
[List of modules that were analyzed]

## Files Read
[List of implementation files that were read]

## Extracted Patterns
[Summary of patterns extracted from implementation]

## Extracted Logic
[Summary of similar logic found]

## Reusable Code Identified
[Summary of reusable code blocks, functions, and classes]

## Reusability Opportunities
[Summary of opportunities to make code reusable]

## Implementation Analysis
[Summary of logic flow, data flow, control flow, dependencies, and patterns]
EOF
```

## Important Constraints

- Must determine if deep reading is needed based on abstraction layer distance
- Must read actual implementation files from modules referenced in basepoints
- Must extract patterns, similar logic, and reusable code from implementation
- Must identify opportunities for making code reusable (moving core/common/similar modules)
- Must analyze implementation to understand logic and patterns
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
- Must be technology-agnostic and work with any basepoint structure
- **CRITICAL**: All deep reading results must be stored in `geist/specs/[current-spec]/implementation/cache/deep-reading/`, not scattered around the codebase
- Must cache results to avoid redundant reads
