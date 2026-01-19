# Reusable Code Detection and Suggestion

## Core Responsibilities

1. **Detect Similar Logic**: Identify similar logic and reusable code patterns from deep reading
2. **Suggest Existing Modules**: Suggest existing modules and code that can be reused
3. **Identify Refactoring Opportunities**: Identify opportunities to refactor code for reusability
4. **Present Reusable Options**: Present reusable options to user with context and pros/cons
5. **Store Detection Results**: Cache reusable code detection results

## Workflow

### Step 1: Load Deep Reading Results

Load previous deep reading results:

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

# Load deep reading results
if [ -f "$CACHE_PATH/deep-reading/implementation-analysis.json" ]; then
    DEEP_READING_RESULTS=$(cat "$CACHE_PATH/deep-reading/implementation-analysis.json")
    EXTRACTED_PATTERNS=$({{EXTRACT_PATTERNS}})
    EXTRACTED_LOGIC=$({{EXTRACT_LOGIC}})
    REUSABLE_CODE=$({{EXTRACT_REUSABLE_CODE}})
    REUSABILITY_OPPORTUNITIES=$({{EXTRACT_REUSABILITY_OPPORTUNITIES}})
else
    echo "⚠️  No deep reading results found. Run deep reading first."
    exit 1
fi
```

### Step 2: Detect Similar Logic and Reusable Code Patterns

Analyze extracted code to detect similar patterns:

```bash
# Detect similar logic patterns
SIMILAR_LOGIC_PATTERNS=""
echo "$EXTRACTED_LOGIC" | while IFS=':' read -r file logic; do
    if [ -z "$file" ] || [ -z "$logic" ]; then
        continue
    fi
    
    # Compare logic with other files
    SIMILAR_FILES=$({{FIND_SIMILAR_LOGIC}})
    
    if [ -n "$SIMILAR_FILES" ]; then
        SIMILAR_LOGIC_PATTERNS="$SIMILAR_LOGIC_PATTERNS\n${file}:${SIMILAR_FILES}"
    fi
done

# Detect reusable code patterns
REUSABLE_CODE_PATTERNS=""
echo "$REUSABLE_CODE" | while IFS=':' read -r file blocks functions classes; do
    if [ -z "$file" ]; then
        continue
    fi
    
    # Identify reusable patterns
    if [ -n "$blocks" ] || [ -n "$functions" ] || [ -n "$classes" ]; then
        REUSABLE_CODE_PATTERNS="$REUSABLE_CODE_PATTERNS\n${file}:blocks=${blocks}:functions=${functions}:classes=${classes}"
    fi
done
```

### Step 3: Suggest Existing Modules and Code That Can Be Reused

Create suggestions for reusable code:

```bash
# Create reuse suggestions
REUSE_SUGGESTIONS=""

# Suggest existing modules
EXISTING_MODULES=$({{FIND_EXISTING_MODULES}})
echo "$EXISTING_MODULES" | while read module; do
    if [ -z "$module" ]; then
        continue
    fi
    
    # Check if module can be reused
    if {{CAN_REUSE_MODULE}}; then
        REUSE_SUGGESTIONS="$REUSE_SUGGESTIONS\nModule: ${module}\n  Can be reused for: [use case]\n  Pros: [advantages]\n  Cons: [disadvantages]"
    fi
done

# Suggest existing code
EXISTING_CODE=$({{FIND_EXISTING_CODE}})
echo "$EXISTING_CODE" | while read code_item; do
    if [ -z "$code_item" ]; then
        continue
    fi
    
    # Check if code can be reused
    if {{CAN_REUSE_CODE}}; then
        REUSE_SUGGESTIONS="$REUSE_SUGGESTIONS\nCode: ${code_item}\n  Can be reused for: [use case]\n  Pros: [advantages]\n  Cons: [disadvantages]"
    fi
done
```

### Step 4: Identify Opportunities to Refactor Code for Reusability

Identify refactoring opportunities:

```bash
# Identify refactoring opportunities
REFACTORING_OPPORTUNITIES=""

# Detect duplicate code
DUPLICATE_CODE=$({{DETECT_DUPLICATE_CODE}})

# Identify code that should be moved to shared locations
SHARED_LOCATION_CANDIDATES=$({{IDENTIFY_SHARED_LOCATION_CANDIDATES}})

# Identify code that should be extracted to common modules
COMMON_MODULE_CANDIDATES=$({{IDENTIFY_COMMON_MODULE_CANDIDATES}})

# Identify code that should be extracted to core modules
CORE_MODULE_CANDIDATES=$({{IDENTIFY_CORE_MODULE_CANDIDATES}})

REFACTORING_OPPORTUNITIES="$REFACTORING_OPPORTUNITIES\nDuplicate Code: ${DUPLICATE_CODE}\nShared Location Candidates: ${SHARED_LOCATION_CANDIDATES}\nCommon Module Candidates: ${COMMON_MODULE_CANDIDATES}\nCore Module Candidates: ${CORE_MODULE_CANDIDATES}"
```

### Step 5: Present Reusable Options to User with Context and Pros/Cons

Prepare presentation of reusable options:

```bash
# Prepare presentation
REUSABLE_OPTIONS_PRESENTATION=""

# Format reuse suggestions with context
echo "$REUSE_SUGGESTIONS" | while IFS='|' read -r type item use_case pros cons; do
    if [ -z "$type" ] || [ -z "$item" ]; then
        continue
    fi
    
    # Add context from basepoints
    CONTEXT=$({{GET_BASEPOINT_CONTEXT}})
    
    # Add pros/cons from basepoints
    PROS_CONS=$({{GET_BASEPOINT_PROS_CONS}})
    
    REUSABLE_OPTIONS_PRESENTATION="$REUSABLE_OPTIONS_PRESENTATION\n**${type}: ${item}**\n  Use Case: ${use_case}\n  Context: ${CONTEXT}\n  Pros: ${pros}\n  Cons: ${cons}\n  Basepoints Info: ${PROS_CONS}"
done

# Format refactoring opportunities
echo "$REFACTORING_OPPORTUNITIES" | while IFS='|' read -r category candidates; do
    if [ -z "$category" ] || [ -z "$candidates" ]; then
        continue
    fi
    
    REUSABLE_OPTIONS_PRESENTATION="$REUSABLE_OPTIONS_PRESENTATION\n**Refactoring Opportunity: ${category}**\n  Candidates: ${candidates}\n  Recommendation: [suggestion]"
done
```

**Presentation Format:**

```
🔍 Reusable Code Detection Results

## Existing Code That Can Be Reused

**Module: [Module Name]**
- Can be reused for: [Use case]
- Context: [Context from basepoints]
- Pros: [Advantages]
- Cons: [Disadvantages]
- Basepoints Info: [Additional info from basepoints]

**Code: [Code Item]**
- Can be reused for: [Use case]
- Context: [Context from basepoints]
- Pros: [Advantages]
- Cons: [Disadvantages]
- Basepoints Info: [Additional info from basepoints]

## Refactoring Opportunities

**Duplicate Code**
- Candidates: [List of duplicate code locations]
- Recommendation: Extract to shared module

**Shared Location Candidates**
- Candidates: [Code that should be moved to shared locations]
- Recommendation: Move to [suggested location]

**Common Module Candidates**
- Candidates: [Code that should be extracted to common modules]
- Recommendation: Create common module at [suggested location]

**Core Module Candidates**
- Candidates: [Code that should be extracted to core modules]
- Recommendation: Create core module at [suggested location]
```

### Step 6: Store Detection Results

Cache reusable code detection results:

```bash
mkdir -p "$CACHE_PATH/deep-reading"

# Store detection results
cat > "$CACHE_PATH/deep-reading/reusable-code-detection.json" << EOF
{
  "similar_logic_patterns": $(echo "$SIMILAR_LOGIC_PATTERNS" | {{JSON_FORMAT}}),
  "reusable_code_patterns": $(echo "$REUSABLE_CODE_PATTERNS" | {{JSON_FORMAT}}),
  "reuse_suggestions": $(echo "$REUSE_SUGGESTIONS" | {{JSON_FORMAT}}),
  "refactoring_opportunities": $(echo "$REFACTORING_OPPORTUNITIES" | {{JSON_FORMAT}}),
  "reusable_options_presentation": $(echo "$REUSABLE_OPTIONS_PRESENTATION" | {{JSON_FORMAT}})
}
EOF

# Also create human-readable summary
cat > "$CACHE_PATH/deep-reading/reusable-code-detection-summary.md" << EOF
# Reusable Code Detection Results

## Similar Logic Patterns
[Summary of similar logic patterns detected]

## Reusable Code Patterns
[Summary of reusable code patterns detected]

## Reuse Suggestions
[Summary of existing modules and code that can be reused]

## Refactoring Opportunities
[Summary of opportunities to refactor code for reusability]

## Reusable Options Presentation
[Formatted presentation of reusable options with context and pros/cons]
EOF
```

## Important Constraints

- Must detect similar logic and reusable code patterns from deep reading
- Must suggest existing modules and code that can be reused
- Must identify opportunities to refactor code for reusability
- Must present reusable options to user with context and pros/cons
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
- Must be technology-agnostic and work with any basepoint structure
- **CRITICAL**: All detection results must be stored in `geist/specs/[current-spec]/implementation/cache/deep-reading/`, not scattered around the codebase
