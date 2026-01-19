The FIRST STEP is to validate prerequisites and run comprehensive validation to identify issues by following these instructions:

## Core Responsibilities

1. **Check Prerequisites**: Verify that geist is deployed and basepoints/product files exist
2. **Run Comprehensive Validation**: Run all validation utilities to identify issues
3. **Determine Cleanup Scope**: Identify which issues need to be fixed
4. **Support Dry-Run Mode**: Allow user to preview changes before applying

## Workflow

### Step 1: Validate Prerequisites

Check if geist is deployed and prerequisites exist:

```bash
# Check if geist directory exists
if [ ! -d "geist" ]; then
    echo "❌ geist directory not found. Please run deploy-agents first."
    exit 1
fi

echo "✅ geist directory found"

# Check if basepoints exist (needed for placeholder replacement)
if [ ! -d "geist/basepoints" ] || [ ! -f "geist/basepoints/headquarter.md" ]; then
    echo "⚠️  Warning: Basepoints not found. Placeholder cleaning may be limited."
    BASEPOINTS_AVAILABLE=false
else
    echo "✅ Basepoints found"
    BASEPOINTS_AVAILABLE=true
fi

# Check if product files exist (needed for some replacements)
if [ ! -d "geist/product" ]; then
    echo "⚠️  Warning: Product files not found. Some replacements may be limited."
    PRODUCT_AVAILABLE=false
else
    echo "✅ Product files found"
    PRODUCT_AVAILABLE=true
fi
```

### Step 2: Check for Dry-Run Mode

Check if user wants to preview changes before applying:

```bash
# Check for dry-run flag (can be set via environment variable or user input)
DRY_RUN="${DRY_RUN:-false}"

if [ "$DRY_RUN" = "true" ]; then
    echo "🔍 DRY-RUN MODE: Changes will be previewed but not applied"
else
    echo "🔧 APPLY MODE: Changes will be applied to files"
fi
```

### Step 3: Run Comprehensive Validation

Run all validation utilities to identify issues:

```bash
# Create cleanup cache directory
mkdir -p geist/.cleanup-cache

# Set spec path for validation workflows
SPEC_PATH="geist/.cleanup-cache"
VALIDATION_CACHE="$SPEC_PATH/validation"
mkdir -p "$VALIDATION_CACHE"

# Run comprehensive validation using orchestrator
echo "🔍 Running comprehensive validation to identify issues..."
{{workflows/validation/orchestrate-validation}}

# Load validation results
if [ -f "$VALIDATION_CACHE/comprehensive-validation-report.json" ]; then
    VALIDATION_REPORT=$(cat "$VALIDATION_CACHE/comprehensive-validation-report.json")
    
    # Extract issue counts
    PLACEHOLDER_TOTAL=$(echo "$VALIDATION_REPORT" | grep -oE '"placeholder_detection":\{"total":[0-9]+' | grep -oE '[0-9]+' | head -1 || echo "0")
    UNNECESSARY_LOGIC_TOTAL=$(echo "$VALIDATION_REPORT" | grep -oE '"unnecessary_logic_detection":\{"total":[0-9]+' | grep -oE '[0-9]+' | head -1 || echo "0")
    CYCLE_TOTAL=$(echo "$VALIDATION_REPORT" | grep -oE '"command_cycle_validation":\{"total":[0-9]+' | grep -oE '[0-9]+' | head -1 || echo "0")
    STRUCTURE_TOTAL=$(echo "$VALIDATION_REPORT" | grep -oE '"project_structure_alignment_validation":\{"total":[0-9]+' | grep -oE '[0-9]+' | head -1 || echo "0")
    
    echo "📊 Validation Results:"
    echo "  - Placeholders: $PLACEHOLDER_TOTAL"
    echo "  - Unnecessary Logic: $UNNECESSARY_LOGIC_TOTAL"
    echo "  - Command Cycle Issues: $CYCLE_TOTAL"
    echo "  - Structure Alignment Issues: $STRUCTURE_TOTAL"
    
    # Store issue counts for later phases
    echo "$PLACEHOLDER_TOTAL" > "$VALIDATION_CACHE/placeholder-count.txt"
    echo "$UNNECESSARY_LOGIC_TOTAL" > "$VALIDATION_CACHE/unnecessary-logic-count.txt"
    echo "$CYCLE_TOTAL" > "$VALIDATION_CACHE/cycle-count.txt"
    echo "$STRUCTURE_TOTAL" > "$VALIDATION_CACHE/structure-count.txt"
else
    echo "❌ Validation failed - no results found"
    exit 1
fi
```

### Step 4: Determine Cleanup Scope

Identify which issues need to be fixed:

```bash
# Load issue counts
PLACEHOLDER_COUNT=$(cat "$VALIDATION_CACHE/placeholder-count.txt" 2>/dev/null || echo "0")
UNNECESSARY_LOGIC_COUNT=$(cat "$VALIDATION_CACHE/unnecessary-logic-count.txt" 2>/dev/null || echo "0")
CYCLE_COUNT=$(cat "$VALIDATION_CACHE/cycle-count.txt" 2>/dev/null || echo "0")
STRUCTURE_COUNT=$(cat "$VALIDATION_CACHE/structure-count.txt" 2>/dev/null || echo "0")

TOTAL_ISSUES=$((PLACEHOLDER_COUNT + UNNECESSARY_LOGIC_COUNT + CYCLE_COUNT + STRUCTURE_COUNT))

if [ "$TOTAL_ISSUES" -eq 0 ]; then
    echo "✅ No issues found! geist is already clean."
    echo "You can exit the cleanup command now."
    exit 0
fi

echo "📋 Cleanup Scope:"
echo "  - Placeholders to clean: $PLACEHOLDER_COUNT"
echo "  - Unnecessary logic to remove: $UNNECESSARY_LOGIC_COUNT"
echo "  - Broken references to fix: $CYCLE_COUNT"
echo "  - Structure alignment issues: $STRUCTURE_COUNT"
echo ""
echo "Total issues to fix: $TOTAL_ISSUES"
```

## Important Constraints

- Must verify geist is deployed before proceeding
- Must run comprehensive validation to identify all issues
- Must support dry-run mode for previewing changes
- Must determine cleanup scope based on validation results
- **CRITICAL**: All validation reports must be stored in `geist/.cleanup-cache/validation/` (temporary, cleaned up after cleanup completes)
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
