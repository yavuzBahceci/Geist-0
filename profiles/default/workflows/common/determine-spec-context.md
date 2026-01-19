# Determine Spec Context

## Core Responsibilities

1. **Establish Spec Path**: Determine the spec path for the current operation
2. **Validate Spec Path**: Ensure the spec path exists and is valid
3. **Set Context Variables**: Make spec path available for use in subsequent workflow steps

## Workflow

### Step 1: Determine Spec Path

Establish the spec path for the current operation:

```bash
# Determine spec path
SPEC_PATH="geist/specs/[current-spec]"
```

**Note:** Replace `[current-spec]` with the actual spec name (e.g., "2026-01-18-refactor-reduce-redundancy").

### Step 2: Validate Spec Path (Optional)

If needed, verify that the spec path exists:

```bash
# Optional: Validate spec path exists
if [ ! -d "$SPEC_PATH" ]; then
    echo "⚠️ Warning: Spec path does not exist: $SPEC_PATH"
    echo "Continuing anyway - path may be created later"
fi
```

### Step 3: Create Cache Directory Structure

Ensure cache directories exist for storing intermediate results:

```bash
# Create cache directories if they don't exist
mkdir -p "$SPEC_PATH/implementation/cache"
mkdir -p "$SPEC_PATH/implementation/cache/validation" 2>/dev/null
mkdir -p "$SPEC_PATH/implementation/cache/scope-detection" 2>/dev/null
```

## Usage Notes

This workflow standardizes the pattern for determining and setting the spec path across commands. It's a simple utility workflow that ensures consistency.

**Typical Usage:**

Commands should use this workflow at the beginning when they need to establish spec context:

```bash
{{workflows/common/determine-spec-context}}
```

This replaces repeated instances of `SPEC_PATH="geist/specs/[current-spec]"` throughout command files.

## Important Constraints

- **Placeholder Replacement**: The `[current-spec]` placeholder must be replaced with the actual spec name before execution.
- **Cache Creation**: This workflow creates cache directories to ensure subsequent workflows can store their outputs.
- **Path Validation**: Validation is optional and non-blocking, allowing for workflows that create the spec structure.
