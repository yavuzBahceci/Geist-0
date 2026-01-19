# Phase 1: Issue Analysis

Parse the input (bug or feedback) and work through abstraction levels to identify the root cause.

## Step 0: Abstraction-Level Debugging Checklist

**IMPORTANT**: Before investigating the codebase, work through these abstraction levels from highest to lowest. Most issues are found in the higher levels.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 ABSTRACTION-LEVEL DEBUG CHECKLIST
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Before investigating the codebase, work through these levels:

## Level 1: User's System Configuration (Highest Priority)
- [ ] How is the user using our structure?
- [ ] Is their Geist installation correct?
- [ ] Are their configuration files valid?
- [ ] Are they following the expected workflow?
- [ ] Did they run the correct command?
- [ ] Are they in the right directory?

Example issues:
- User ran command from wrong directory
- Config file has incorrect paths
- Missing required setup steps

## Level 2: Their System/Environment Issues
- [ ] What can go wrong in their environment?
- [ ] OS/shell compatibility issues?
- [ ] File permissions?
- [ ] Missing dependencies (node, python, etc)?
- [ ] Environment variables not set?
- [ ] Disk space or memory issues?

Example issues:
- User's bash version doesn't support certain syntax
- Permission denied on file operations
- Required tool not installed

## Level 3: Our System Issues (Geist Framework)
- [ ] What can go wrong in Geist itself?
- [ ] Are our templates correct?
- [ ] Are our workflows complete?
- [ ] Are our references valid ({{}} syntax)?
- [ ] Is our documentation accurate?
- [ ] Are there version mismatches?

Example issues:
- Workflow file has incorrect template syntax
- Missing workflow file referenced
- Outdated documentation

## Level 4: Connection/Integration Issues
- [ ] Integration points between systems?
- [ ] Path resolution issues (relative vs absolute)?
- [ ] File reference issues?
- [ ] Command execution issues?
- [ ] Data format mismatches?
- [ ] API/interface changes?

Example issues:
- Path reference uses wrong relative path format
- Output format changed breaking downstream
- API response structure changed

## Level 5: Codebase Investigation (Lowest Priority)
⚠️ ONLY proceed here after ruling out Levels 1-4

This is where we investigate actual code bugs:
- Logic errors in implementation
- Algorithm issues
- Race conditions
- Edge cases not handled
- Resource leaks

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Step 0.1: Present Checklist and Gather Initial Information

Before proceeding, gather information about the higher abstraction levels:

```bash
echo ""
echo "Before investigating the codebase, let's check the higher abstraction levels."
echo ""
echo "📋 Quick Questions:"
echo "1. What command were you running?"
echo "2. What directory were you in?"
echo "3. What was the expected behavior?"
echo "4. What actually happened?"
echo "5. Any error messages or logs?"
echo ""
echo "⚠️ **STOP and WAIT** for user response before proceeding."
```

### Step 0.2: Abstraction Level Analysis

Based on user's response, assess which level the issue likely falls into:

```bash
echo "🔍 Analyzing which abstraction level this issue falls into..."

# Store user's responses
USER_COMMAND="[from user response]"
USER_DIRECTORY="[from user response]"
EXPECTED_BEHAVIOR="[from user response]"
ACTUAL_BEHAVIOR="[from user response]"
ERROR_MESSAGE="[from user response]"

# Initialize level assessment
LIKELY_LEVEL="unknown"
LEVEL_REASONING=""

# Level 1 indicators (User Configuration)
if echo "$ERROR_MESSAGE" | grep -iE "not found|command not found|no such file|permission denied" > /dev/null; then
    LIKELY_LEVEL="1"
    LEVEL_REASONING="Error suggests configuration or path issue"
fi

# Level 2 indicators (Their System)
if echo "$ERROR_MESSAGE" | grep -iE "syntax error|bash|shell|version|dependency|module not found" > /dev/null; then
    LIKELY_LEVEL="2"
    LEVEL_REASONING="Error suggests environment or dependency issue"
fi

# Level 3 indicators (Our System)
if echo "$ERROR_MESSAGE" | grep -iE "template|workflow|geist|{{|reference" > /dev/null; then
    LIKELY_LEVEL="3"
    LEVEL_REASONING="Error suggests Geist framework issue"
fi

# Level 4 indicators (Connection)
if echo "$ERROR_MESSAGE" | grep -iE "path|relative|integration|format|parse" > /dev/null; then
    LIKELY_LEVEL="4"
    LEVEL_REASONING="Error suggests integration or path issue"
fi

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📊 INITIAL ASSESSMENT"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "Likely abstraction level: Level $LIKELY_LEVEL"
echo "Reasoning: $LEVEL_REASONING"
echo ""
echo "Recommended next steps for Level $LIKELY_LEVEL:"
case "$LIKELY_LEVEL" in
    "1") echo "- Verify user's directory and command"
         echo "- Check config file validity"
         echo "- Verify Geist installation"
         ;;
    "2") echo "- Check system dependencies"
         echo "- Verify shell/OS compatibility"
         echo "- Check permissions"
         ;;
    "3") echo "- Review Geist templates and workflows"
         echo "- Check for missing files"
         echo "- Verify reference syntax"
         ;;
    "4") echo "- Check path resolution"
         echo "- Verify data formats"
         echo "- Check integration points"
         ;;
    *)   echo "- Need more information to assess"
         ;;
esac
echo ""
```

### Step 0.3: Checkpoint Before Codebase Investigation

```bash
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "⚠️ CHECKPOINT: CONFIRM BEFORE CODEBASE INVESTIGATION"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "Have you ruled out issues at Levels 1-4?"
echo ""
echo "- Level 1 (User Config): Verified ✓/✗"
echo "- Level 2 (Their System): Verified ✓/✗"
echo "- Level 3 (Our System): Verified ✓/✗"
echo "- Level 4 (Connections): Verified ✓/✗"
echo ""
echo "To proceed to codebase investigation, confirm that you've checked"
echo "the higher levels, or indicate which level needs more investigation."
echo ""
echo "Options:"
echo "- Type \"confirmed\" to proceed to codebase investigation"
echo "- Type \"level 1\", \"level 2\", etc. to investigate that level further"
echo ""
echo "⚠️ **STOP and WAIT** for user response."
```

---

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

CACHE_PATH="geist/output/fix-bug/cache"
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

Analysis saved to: geist/output/fix-bug/cache/issue-analysis.md

NEXT STEP 👉 Proceeding to Phase 2: Library Research
```
