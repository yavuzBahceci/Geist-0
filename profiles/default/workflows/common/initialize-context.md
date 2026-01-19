# Initialize Context

## Core Responsibilities

1. **Set Common Variables**: Initialize common context variables used across command workflows
2. **Determine Paths**: Establish standard paths for geist, basepoints, product, and spec operations
3. **Set Flags**: Initialize boolean flags for feature detection and state tracking
4. **Prepare Environment**: Set up environment for command execution

## Workflow

### Step 1: Set Base Paths

Establish standard paths used throughout command execution:

```bash
# Set base paths
GEIST_PATH="geist"
BASEPOINTS_PATH="$GEIST_PATH/basepoints"
PRODUCT_PATH="$GEIST_PATH/product"
CONFIG_PATH="$GEIST_PATH/config"
```

### Step 2: Determine Spec Path (if applicable)

If working with a spec, determine and set the spec path:

```bash
# Determine spec path (if applicable)
if [ -n "[current-spec]" ]; then
    SPEC_PATH="$GEIST_PATH/specs/[current-spec]"
    CACHE_PATH="$SPEC_PATH/implementation/cache"
    mkdir -p "$CACHE_PATH"
else
    SPEC_PATH=""
    CACHE_PATH=""
fi
```

**Note:** Replace `[current-spec]` with the actual spec name, or leave empty if not applicable.

### Step 3: Initialize Flags

Set common boolean flags for state tracking:

```bash
# Initialize common flags
BASEPOINTS_AVAILABLE="false"
PRODUCT_AVAILABLE="false"
PROJECT_PROFILE_LOADED="false"
ENRICHED_KNOWLEDGE_LOADED="false"
```

### Step 4: Detect Available Resources

Check for available resources and update flags:

```bash
# Check for basepoints
if [ -d "$BASEPOINTS_PATH" ] && [ -f "$BASEPOINTS_PATH/headquarter.md" ]; then
    BASEPOINTS_AVAILABLE="true"
fi

# Check for product files
if [ -d "$PRODUCT_PATH" ] && [ -f "$PRODUCT_PATH/mission.md" ]; then
    PRODUCT_AVAILABLE="true"
fi

# Check for project profile
if [ -f "$CONFIG_PATH/project-profile.yml" ]; then
    PROJECT_PROFILE_LOADED="true"
fi

# Check for enriched knowledge
if [ -d "$CONFIG_PATH/enriched-knowledge" ]; then
    ENRICHED_KNOWLEDGE_LOADED="true"
fi
```

## Usage Notes

This workflow provides a standard initialization pattern for commands. It sets up common paths, flags, and detects available resources.

**Typical Usage:**

Commands can use this workflow at the beginning to initialize their context:

```bash
{{workflows/common/initialize-context}}
```

This ensures consistent variable naming and resource detection across all commands.

## Important Constraints

- **Placeholder Replacement**: The `[current-spec]` placeholder should be replaced with the actual spec name, or removed if not applicable.
- **Non-Blocking**: Resource detection does not fail if resources are missing - it only sets flags.
- **Path Consistency**: Uses standardized path structure that matches geist architecture.
