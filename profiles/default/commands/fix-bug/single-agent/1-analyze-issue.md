# Phase 1: Issue Analysis

Parse the input (bug or feedback) and extract relevant details.

## Step 1: Identify Input Type

Determine whether the input is a bug report or feedback:

```bash
echo "🔍 Analyzing input type..."

# Check for bug indicators
BUG_INDICATORS="error|exception|crash|fail|bug|broken|not working|stack trace|traceback"
FEEDBACK_INDICATORS="feature|enhancement|improve|suggest|request|would be nice|should|could"

INPUT_TEXT="[USER_INPUT]"

if echo "$INPUT_TEXT" | grep -iE "$BUG_INDICATORS" > /dev/null; then
    INPUT_TYPE="bug"
    echo "   Input type: BUG"
elif echo "$INPUT_TEXT" | grep -iE "$FEEDBACK_INDICATORS" > /dev/null; then
    INPUT_TYPE="feedback"
    echo "   Input type: FEEDBACK"
else
    INPUT_TYPE="unknown"
    echo "   Input type: UNKNOWN (treating as bug)"
    INPUT_TYPE="bug"
fi
```

## Step 2: Extract Details from Input

Extract relevant details based on input type:

### For Bugs:

```bash
if [ "$INPUT_TYPE" = "bug" ]; then
    echo "📋 Extracting bug details..."
    
    # Extract error message
    ERROR_MESSAGE=$(echo "$INPUT_TEXT" | grep -iE "error:|exception:|Error|Exception" | head -5)
    
    # Extract stack trace (if present)
    STACK_TRACE=$(echo "$INPUT_TEXT" | grep -A 20 "Traceback\|at \|Stack trace\|Error:")
    
    # Extract error code (if present)
    ERROR_CODE=$(echo "$INPUT_TEXT" | grep -oE "[A-Z_]+_ERROR|E[0-9]+|error [0-9]+")
    
    # Extract file/line references
    FILE_REFERENCES=$(echo "$INPUT_TEXT" | grep -oE "[a-zA-Z0-9_/.-]+\.(ts|js|py|go|rs|java|md|sh):[0-9]+")
    
    echo "   Error message: ${ERROR_MESSAGE:-Not found}"
    echo "   Stack trace: ${STACK_TRACE:+Found}"
    echo "   Error code: ${ERROR_CODE:-Not found}"
    echo "   File references: ${FILE_REFERENCES:-Not found}"
fi
```

### For Feedbacks:

```bash
if [ "$INPUT_TYPE" = "feedback" ]; then
    echo "📋 Extracting feedback details..."
    
    # Extract feature description
    FEATURE_DESCRIPTION="$INPUT_TEXT"
    
    # Extract affected area (if mentioned)
    AFFECTED_AREA=$(echo "$INPUT_TEXT" | grep -oE "in (the )?[a-zA-Z0-9_-]+ (module|component|feature|page|section)")
    
    # Extract desired behavior
    DESIRED_BEHAVIOR=$(echo "$INPUT_TEXT" | grep -iE "should|could|would|want|need|like to")
    
    echo "   Feature description: Found"
    echo "   Affected area: ${AFFECTED_AREA:-Not specified}"
    echo "   Desired behavior: ${DESIRED_BEHAVIOR:+Found}"
fi
```

## Step 3: Identify Affected Libraries and Modules

Analyze the input to identify which libraries and modules are affected:

```bash
echo "🔍 Identifying affected libraries and modules..."

# Extract library names from error messages or stack traces
AFFECTED_LIBRARIES=""

# Check for common library patterns in the input
# (Technology-agnostic patterns)
if echo "$INPUT_TEXT" | grep -iE "import|require|from|use " > /dev/null; then
    LIBRARY_MENTIONS=$(echo "$INPUT_TEXT" | grep -oE "(import|require|from|use) [a-zA-Z0-9_.-]+")
    AFFECTED_LIBRARIES="$LIBRARY_MENTIONS"
fi

# Extract module/file paths
AFFECTED_MODULES=$(echo "$INPUT_TEXT" | grep -oE "[a-zA-Z0-9_/.-]+\.(ts|js|py|go|rs|java|md|sh)" | sort -u)

echo "   Affected libraries: ${AFFECTED_LIBRARIES:-None identified}"
echo "   Affected modules: ${AFFECTED_MODULES:-None identified}"
```

## Step 4: Create Issue Analysis Document

Generate the issue analysis document:

```bash
echo "📝 Creating issue analysis document..."

CACHE_PATH="agent-os/output/fix-bug/cache"
mkdir -p "$CACHE_PATH"

cat > "$CACHE_PATH/issue-analysis.md" << 'ANALYSIS_EOF'
# Issue Analysis

## Input Type
$INPUT_TYPE

## Issue Summary
[Brief summary of the bug/feedback]

## Details Extracted

### For Bugs:
- **Error Message:** $ERROR_MESSAGE
- **Stack Trace:** $STACK_TRACE
- **Error Code:** $ERROR_CODE
- **File References:** $FILE_REFERENCES

### For Feedbacks:
- **Feature Description:** $FEATURE_DESCRIPTION
- **Affected Area:** $AFFECTED_AREA
- **Desired Behavior:** $DESIRED_BEHAVIOR

## Affected Components

### Libraries
$AFFECTED_LIBRARIES

### Modules
$AFFECTED_MODULES

## Initial Assessment
[Initial assessment of the issue based on extracted details]

---

*Generated: $(date)*
ANALYSIS_EOF

echo "✅ Issue analysis complete"
echo "   Analysis saved to: $CACHE_PATH/issue-analysis.md"
```

## Display Progress and Next Step

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 PHASE 1: ISSUE ANALYSIS COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Input type identified: [bug/feedback]
✅ Details extracted
✅ Affected libraries identified: [count]
✅ Affected modules identified: [count]

Analysis saved to: agent-os/output/fix-bug/cache/issue-analysis.md

NEXT STEP 👉 Proceeding to Phase 2: Library Research
```
