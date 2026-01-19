# Phase 4: Code Analysis

Deep-dive into specific files and modules where the issue occurs.

## Step 1: Load Previous Analysis

Load all previous analysis:

```bash
CACHE_PATH="geist/output/fix-bug/cache"

# Load issue analysis
ISSUE_ANALYSIS=$(cat "$CACHE_PATH/issue-analysis.md" 2>/dev/null)

# Load library research
LIBRARY_RESEARCH=$(cat "$CACHE_PATH/library-research.md" 2>/dev/null)

# Load basepoints integration
BASEPOINTS_INTEGRATION=$(cat "$CACHE_PATH/basepoints-integration.md" 2>/dev/null)

# Extract affected modules and file references
AFFECTED_MODULES=$(grep -A 10 "### Modules" "$CACHE_PATH/issue-analysis.md" | grep -v "^#" | head -10)
FILE_REFERENCES=$(grep "File References" "$CACHE_PATH/issue-analysis.md" -A 5 | grep -oE "[a-zA-Z0-9_/.-]+\.(ts|js|py|go|rs|java|md|sh)(:[0-9]+)?")

echo "✅ Previous analysis loaded"
echo "   Affected modules: $(echo "$AFFECTED_MODULES" | wc -w | tr -d ' ')"
echo "   File references: $(echo "$FILE_REFERENCES" | wc -w | tr -d ' ')"
```

## Step 2: Identify Exact File/Module Locations

Identify exact locations from error logs:

```bash
echo "🔍 Identifying exact file/module locations..."

CODE_LOCATIONS=""

# Parse file references with line numbers
for ref in $FILE_REFERENCES; do
    FILE_PATH=$(echo "$ref" | cut -d':' -f1)
    LINE_NUMBER=$(echo "$ref" | cut -d':' -f2)
    
    if [ -f "$FILE_PATH" ]; then
        echo "   Found: $FILE_PATH (line $LINE_NUMBER)"
        CODE_LOCATIONS="${CODE_LOCATIONS}
- $FILE_PATH:$LINE_NUMBER"
    fi
done

# Also check affected modules
for module in $AFFECTED_MODULES; do
    if [ -f "$module" ]; then
        echo "   Found module: $module"
        CODE_LOCATIONS="${CODE_LOCATIONS}
- $module"
    fi
done

echo "   Total locations identified: $(echo "$CODE_LOCATIONS" | grep -c "^-")"
```

## Step 3: Deep-Dive into Relevant Code Files

Read and analyze the relevant code files:

```bash
echo "📖 Reading relevant code files..."

CODE_ANALYSIS=""

for location in $(echo "$CODE_LOCATIONS" | grep "^-" | sed 's/^- //'); do
    FILE_PATH=$(echo "$location" | cut -d':' -f1)
    LINE_NUMBER=$(echo "$location" | cut -d':' -f2)
    
    if [ ! -f "$FILE_PATH" ]; then
        continue
    fi
    
    echo "   Analyzing: $FILE_PATH"
    
    # Read file content
    FILE_CONTENT=$(cat "$FILE_PATH")
    
    # If line number specified, focus on that area
    if [ -n "$LINE_NUMBER" ] && [ "$LINE_NUMBER" != "$FILE_PATH" ]; then
        # Extract context around the line (20 lines before and after)
        START_LINE=$((LINE_NUMBER - 20))
        [ "$START_LINE" -lt 1 ] && START_LINE=1
        END_LINE=$((LINE_NUMBER + 20))
        
        CONTEXT=$(sed -n "${START_LINE},${END_LINE}p" "$FILE_PATH")
        
        CODE_ANALYSIS="${CODE_ANALYSIS}

## $FILE_PATH (around line $LINE_NUMBER)

### Code Context
\`\`\`
$CONTEXT
\`\`\`

### Analysis
[Analysis of this code section]
"
    else
        # Analyze entire file structure
        CODE_ANALYSIS="${CODE_ANALYSIS}

## $FILE_PATH

### File Structure
[Overview of file structure and key components]

### Key Functions/Methods
[List key functions/methods in this file]

### Analysis
[Analysis of this file]
"
    fi
done
```

## Step 4: Analyze Code Patterns and Flows

Analyze code patterns and execution flows in the error context:

```bash
echo "🔍 Analyzing code patterns and flows..."

PATTERN_ANALYSIS=""

# Look for common patterns in the code
# - Error handling patterns
# - Data flow patterns
# - Control flow patterns

for location in $(echo "$CODE_LOCATIONS" | grep "^-" | sed 's/^- //' | cut -d':' -f1 | sort -u); do
    if [ ! -f "$location" ]; then
        continue
    fi
    
    echo "   Analyzing patterns in: $location"
    
    # Check for error handling
    ERROR_HANDLING=$(grep -n "try\|catch\|except\|error\|Error\|throw\|raise" "$location" 2>/dev/null | head -10)
    
    # Check for async patterns
    ASYNC_PATTERNS=$(grep -n "async\|await\|Promise\|Future\|callback" "$location" 2>/dev/null | head -10)
    
    # Check for data validation
    VALIDATION=$(grep -n "validate\|check\|assert\|if.*null\|if.*undefined" "$location" 2>/dev/null | head -10)
    
    PATTERN_ANALYSIS="${PATTERN_ANALYSIS}

### $location

#### Error Handling
$ERROR_HANDLING

#### Async Patterns
$ASYNC_PATTERNS

#### Validation
$VALIDATION
"
done
```

## Step 5: Trace Execution Paths

Trace execution paths leading to the error:

```bash
echo "🔍 Tracing execution paths..."

EXECUTION_TRACE=""

# Based on stack trace or error location, trace the execution path
# This involves:
# 1. Identifying entry points
# 2. Following function calls
# 3. Identifying where the error occurs

# Extract function/method names from error context
FUNCTION_NAMES=$(echo "$ISSUE_ANALYSIS" | grep -oE "[a-zA-Z_][a-zA-Z0-9_]*\(" | sed 's/($//' | sort -u)

for func in $FUNCTION_NAMES; do
    if [ -z "$func" ]; then
        continue
    fi
    
    # Search for function definitions
    DEFINITIONS=$(grep -rn "function $func\|def $func\|fn $func\|func $func" . --include="*.ts" --include="*.js" --include="*.py" --include="*.go" --include="*.rs" 2>/dev/null | head -5)
    
    if [ -n "$DEFINITIONS" ]; then
        EXECUTION_TRACE="${EXECUTION_TRACE}

### $func
$DEFINITIONS
"
    fi
done
```

## Step 6: Create Code Analysis Document

Generate the code analysis document:

```bash
echo "📝 Creating code analysis document..."

cat > "$CACHE_PATH/code-analysis.md" << 'CODE_EOF'
# Code Analysis

## Analysis Summary
- **Files Analyzed:** [count]
- **Locations Identified:** [count]
- **Analysis Date:** $(date)

---

## Code Locations

$CODE_LOCATIONS

---

## Code Analysis

$CODE_ANALYSIS

---

## Pattern Analysis

$PATTERN_ANALYSIS

---

## Execution Trace

$EXECUTION_TRACE

---

## Key Findings

### Root Cause Hypothesis
[Based on code analysis, hypothesis about root cause]

### Problematic Code Sections
[Specific code sections that appear problematic]

### Missing Error Handling
[Areas where error handling is missing or insufficient]

### Pattern Violations
[Any violations of expected patterns]

---

*Generated by fix-bug command - Phase 4*
CODE_EOF

echo "✅ Code analysis complete"
echo "   Analysis saved to: $CACHE_PATH/code-analysis.md"
```

## Display Progress and Next Step

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔬 PHASE 4: CODE ANALYSIS COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Files analyzed: [count]
✅ Code patterns identified
✅ Execution paths traced
✅ Root cause hypothesis formed

Analysis saved to: geist/output/fix-bug/cache/code-analysis.md

NEXT STEP 👉 Proceeding to Phase 5: Knowledge Synthesis
```
