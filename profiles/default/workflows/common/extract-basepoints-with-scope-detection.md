# Extract Basepoints Knowledge with Scope Detection

## Core Responsibilities

1. **Check Basepoints Availability**: Verify that basepoints exist before attempting extraction
2. **Determine Spec Context**: Establish the spec path for caching extracted knowledge
3. **Extract Basepoints Knowledge**: Automatically extract knowledge from all basepoint files
4. **Detect Abstraction Layer**: Identify which abstraction layer the spec targets
5. **Perform Scope Detection**: Run semantic analysis and keyword matching to narrow scope
6. **Load Extracted Knowledge**: Load cached knowledge files for use in command execution
7. **Load Detected Layer**: Load detected abstraction layer information

## Workflow

### Step 1: Check if Basepoints Exist

Before attempting extraction, verify that basepoints are available:

```bash
# Check if basepoints exist
if [ -d "agent-os/basepoints" ] && [ -f "agent-os/basepoints/headquarter.md" ]; then
    BASEPOINTS_AVAILABLE="true"
else
    BASEPOINTS_AVAILABLE="false"
    echo "⚠️ Basepoints not found. Continuing without basepoints knowledge."
    # Exit early - this workflow requires basepoints
    return 0
fi
```

### Step 2: Determine Spec Path

Establish the spec path for caching extracted knowledge:

```bash
# Determine spec path
SPEC_PATH="agent-os/specs/[current-spec]"
```

### Step 3: Extract Basepoints Knowledge Automatically

Extract knowledge from all basepoint files using the automatic extraction workflow:

```bash
# Extract basepoints knowledge using scope detection
{{workflows/basepoints/extract-basepoints-knowledge-automatic}}
```

### Step 4: Detect Abstraction Layer

Identify which abstraction layer this spec targets:

```bash
# Detect abstraction layer
{{workflows/scope-detection/detect-abstraction-layer}}
```

### Step 5: Perform Scope Detection

Run scope detection workflows to narrow the context:

```bash
# Perform scope semantic analysis
{{workflows/scope-detection/detect-scope-semantic-analysis}}

# Perform scope keyword matching
{{workflows/scope-detection/detect-scope-keyword-matching}}
```

### Step 6: Load Extracted Knowledge

Load the cached knowledge file for use in command execution:

```bash
# Load extracted knowledge for use in command
if [ -f "$SPEC_PATH/implementation/cache/basepoints-knowledge.md" ]; then
    EXTRACTED_KNOWLEDGE=$(cat "$SPEC_PATH/implementation/cache/basepoints-knowledge.md")
    echo "✅ Loaded basepoints knowledge"
else
    EXTRACTED_KNOWLEDGE=""
    echo "⚠️ Basepoints knowledge file not found after extraction"
fi
```

### Step 7: Load Detected Layer

Load the detected abstraction layer information:

```bash
# Load detected layer
if [ -f "$SPEC_PATH/implementation/cache/detected-layer.txt" ]; then
    DETECTED_LAYER=$(cat "$SPEC_PATH/implementation/cache/detected-layer.txt")
    echo "✅ Detected layer: $DETECTED_LAYER"
else
    DETECTED_LAYER="unknown"
    echo "⚠️ Layer detection file not found"
fi
```

## Usage Notes

This workflow is designed to be used in commands that need to extract and load basepoints knowledge with scope detection. It wraps the common pattern of:
1. Checking basepoints availability
2. Extracting knowledge
3. Running scope detection
4. Loading cached results

**Typical Usage Pattern:**

Commands can replace their inline basepoints extraction + scope detection blocks with:

```bash
{{workflows/common/extract-basepoints-with-scope-detection}}
```

This eliminates redundancy and provides a single source of truth for this common pattern.

## Important Constraints

- **Requires Basepoints**: This workflow requires basepoints to exist. If they don't exist, it will exit gracefully.
- **Spec Path Required**: The `[current-spec]` placeholder must be replaced with the actual spec name.
- **Cache Dependencies**: This workflow depends on the cache directory structure created by the referenced workflows.
