# Extract Basepoints Knowledge and Load

## Core Responsibilities

1. **Check Basepoints Availability**: Verify that basepoints exist before attempting extraction
2. **Determine Spec Context**: Establish the spec path for caching extracted knowledge
3. **Extract Basepoints Knowledge**: Automatically extract knowledge from all basepoint files
4. **Load Extracted Knowledge**: Load cached knowledge files for use in command execution
5. **Load Detected Layer**: Load detected abstraction layer information (if available)

## Workflow

### Step 1: Check if Basepoints Exist

Before attempting extraction, verify that basepoints are available:

```bash
# Check if basepoints exist
if [ -d "geist/basepoints" ] && [ -f "geist/basepoints/headquarter.md" ]; then
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
SPEC_PATH="geist/specs/[current-spec]"
```

### Step 3: Extract Basepoints Knowledge Automatically

Extract knowledge from all basepoint files using the automatic extraction workflow:

```bash
# Extract basepoints knowledge
{{workflows/basepoints/extract-basepoints-knowledge-automatic}}
```

### Step 4: Load Extracted Knowledge

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

### Step 5: Load Detected Layer (Optional)

Load the detected abstraction layer information if it exists:

```bash
# Load detected layer (if available)
if [ -f "$SPEC_PATH/implementation/cache/detected-layer.txt" ]; then
    DETECTED_LAYER=$(cat "$SPEC_PATH/implementation/cache/detected-layer.txt")
    echo "✅ Detected layer: $DETECTED_LAYER"
else
    DETECTED_LAYER=""
fi
```

## Usage Notes

This workflow is a simpler version that extracts and loads basepoints knowledge without running scope detection. Use this when you only need the extracted knowledge without scope narrowing.

**Use `extract-basepoints-with-scope-detection.md` instead if you need:**
- Abstraction layer detection
- Semantic analysis
- Keyword matching for scope narrowing

**Use this workflow when you need:**
- Simple basepoints extraction and loading
- No scope detection required
- Just the extracted knowledge file

## Important Constraints

- **Requires Basepoints**: This workflow requires basepoints to exist. If they don't exist, it will exit gracefully.
- **Spec Path Required**: The `[current-spec]` placeholder must be replaced with the actual spec name.
- **Cache Dependencies**: This workflow depends on the cache directory structure created by the referenced workflows.
