# Common Prerequisites Validation

## Core Responsibilities

1. **Check Agent-OS Deployment**: Verify that geist directory exists
2. **Check Basepoints Availability**: Verify basepoints folder and files exist (optional check)
3. **Check Product Files Availability**: Verify product files exist (optional check)
4. **Set Availability Flags**: Set flags indicating which resources are available
5. **Provide Clear Feedback**: Output status messages for each check

## Workflow

### Step 1: Check Agent-OS Deployment

Verify that the geist directory exists:

```bash
# Check if geist directory exists
if [ ! -d "geist" ]; then
    echo "❌ geist directory not found. Please run deploy-agents first."
    GEIST_AVAILABLE="false"
    # Note: Commands may choose to exit or continue based on their needs
else
    echo "✅ geist directory found"
    GEIST_AVAILABLE="true"
fi
```

### Step 2: Check Basepoints Availability (Optional)

Check if basepoints are available (non-blocking check):

```bash
# Check if basepoints exist (optional - some commands don't require basepoints)
if [ ! -d "geist/basepoints" ] || [ ! -f "geist/basepoints/headquarter.md" ]; then
    echo "⚠️  Warning: Basepoints not found."
    BASEPOINTS_AVAILABLE="false"
else
    echo "✅ Basepoints found"
    BASEPOINTS_AVAILABLE="true"
fi
```

### Step 3: Check Product Files Availability (Optional)

Check if product files are available (non-blocking check):

```bash
# Check if product files exist (optional - some commands don't require product files)
if [ ! -d "geist/product" ] || [ ! -f "geist/product/mission.md" ]; then
    echo "⚠️  Warning: Product files not found."
    PRODUCT_AVAILABLE="false"
else
    echo "✅ Product files found"
    PRODUCT_AVAILABLE="true"
fi
```

### Step 4: Summary

Output a summary of available resources:

```bash
echo ""
echo "Prerequisites Status:"
echo "  Agent-OS: $GEIST_AVAILABLE"
echo "  Basepoints: ${BASEPOINTS_AVAILABLE:-not checked}"
echo "  Product Files: ${PRODUCT_AVAILABLE:-not checked}"
```

## Usage Notes

This workflow provides a standardized, non-blocking prerequisite check that sets flags for resource availability. Commands can use these flags to decide how to proceed.

**Key Features:**
- **Non-blocking**: Checks don't exit - they set flags
- **Optional Checks**: Basepoints and product checks are optional based on command needs
- **Standardized Flags**: Uses consistent variable names across commands

**Typical Usage:**

Commands can use this workflow early in their execution to check prerequisites:

```bash
{{workflows/common/validate-prerequisites}}
```

Commands can then use the flags to customize their behavior:

```bash
if [ "$BASEPOINTS_AVAILABLE" = "true" ]; then
    # Use basepoints knowledge
fi
```

## Important Constraints

- **Non-Blocking Design**: This workflow does not exit - it only sets flags. Commands must decide how to handle missing prerequisites.
- **Optional Checks**: Commands may skip basepoints or product checks if not needed.
- **Flag-Based**: Uses boolean flags for state tracking, allowing commands to make decisions based on availability.
