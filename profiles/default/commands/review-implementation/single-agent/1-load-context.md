# Phase 1: Load Context

Load implementation files and accumulated knowledge to understand what was implemented.

## Step 1: Determine Spec Path

```bash
echo "🔍 Determining spec path..."

# SPEC_PATH should be provided by user or determined from context
# If not provided, try to find the most recent spec
if [ -z "$SPEC_PATH" ]; then
    # Find most recent spec folder
    SPEC_PATH=$(find geist/specs -maxdepth 1 -type d -name "20*" | sort -r | head -1)
    
    if [ -z "$SPEC_PATH" ]; then
        echo "❌ ERROR: No spec path provided and no specs found."
        echo "   Usage: /review-implementation \"geist/specs/YYYY-MM-DD-spec-name\""
        exit 1
    fi
    
    echo "   Using most recent spec: $SPEC_PATH"
else
    echo "   Spec path: $SPEC_PATH"
fi

# Validate spec exists
if [ ! -d "$SPEC_PATH" ]; then
    echo "❌ ERROR: Spec path does not exist: $SPEC_PATH"
    exit 1
fi
```

## Step 2: Load Spec and Requirements

```bash
echo "📖 Loading spec and requirements..."

SPEC_FILE="$SPEC_PATH/spec.md"
REQUIREMENTS_FILE="$SPEC_PATH/planning/requirements.md"
TASKS_FILE="$SPEC_PATH/tasks.md"

# Load spec
if [ -f "$SPEC_FILE" ]; then
    SPEC_CONTENT=$(cat "$SPEC_FILE")
    echo "   ✅ Loaded spec.md"
else
    echo "   ⚠️ spec.md not found"
    SPEC_CONTENT=""
fi

# Load requirements
if [ -f "$REQUIREMENTS_FILE" ]; then
    REQUIREMENTS_CONTENT=$(cat "$REQUIREMENTS_FILE")
    echo "   ✅ Loaded requirements.md"
else
    echo "   ⚠️ requirements.md not found"
    REQUIREMENTS_CONTENT=""
fi

# Load tasks
if [ -f "$TASKS_FILE" ]; then
    TASKS_CONTENT=$(cat "$TASKS_FILE")
    echo "   ✅ Loaded tasks.md"
else
    echo "   ⚠️ tasks.md not found"
    TASKS_CONTENT=""
fi
```

## Step 3: Identify Implementation Files

```bash
echo "📁 Identifying implementation files..."

IMPL_PATH="$SPEC_PATH/implementation"
IMPL_FILES=""

if [ -d "$IMPL_PATH" ]; then
    # Find all implementation-related files
    IMPL_FILES=$(find "$IMPL_PATH" -type f -name "*.md" -o -name "*.ts" -o -name "*.js" -o -name "*.py" | sort)
    IMPL_COUNT=$(echo "$IMPL_FILES" | grep -c "." || echo "0")
    echo "   ✅ Found $IMPL_COUNT implementation files"
else
    echo "   ⚠️ No implementation directory found"
fi

# Also check if files were modified in the codebase
# (Look for files mentioned in tasks.md)
MODIFIED_FILES=""
if [ -n "$TASKS_CONTENT" ]; then
    # Extract file paths from tasks
    MODIFIED_FILES=$(echo "$TASKS_CONTENT" | grep -oE "\`[a-zA-Z0-9_/.-]+\.(ts|tsx|js|jsx|py|go|rs|java|kt|swift|cs|md)\`" | tr -d '`' | sort -u)
    
    if [ -n "$MODIFIED_FILES" ]; then
        MOD_COUNT=$(echo "$MODIFIED_FILES" | wc -l | tr -d ' ')
        echo "   ✅ Found $MOD_COUNT files mentioned in tasks"
    fi
fi
```

## Step 4: Load Accumulated Knowledge

```bash
echo "📚 Loading accumulated knowledge..."

CACHE_PATH="$SPEC_PATH/implementation/cache"
KNOWLEDGE_FILE="$CACHE_PATH/accumulated-knowledge.md"
REFINED_FILE="$CACHE_PATH/refined-knowledge.md"

# Prefer refined knowledge if available
if [ -f "$REFINED_FILE" ]; then
    ACCUMULATED_KNOWLEDGE=$(cat "$REFINED_FILE")
    echo "   ✅ Loaded refined-knowledge.md"
elif [ -f "$KNOWLEDGE_FILE" ]; then
    ACCUMULATED_KNOWLEDGE=$(cat "$KNOWLEDGE_FILE")
    echo "   ✅ Loaded accumulated-knowledge.md"
else
    echo "   ⚠️ No accumulated knowledge found"
    ACCUMULATED_KNOWLEDGE=""
fi
```

## Step 5: Extract Acceptance Criteria

```bash
echo "📋 Extracting acceptance criteria..."

# Extract from spec
SPEC_ACCEPTANCE_CRITERIA=""
if [ -n "$SPEC_CONTENT" ]; then
    SPEC_ACCEPTANCE_CRITERIA=$(echo "$SPEC_CONTENT" | sed -n '/## Acceptance Criteria/,/^## /p' | head -n -1)
fi

# Extract from tasks
TASKS_ACCEPTANCE_CRITERIA=""
if [ -n "$TASKS_CONTENT" ]; then
    TASKS_ACCEPTANCE_CRITERIA=$(echo "$TASKS_CONTENT" | grep -A 10 "Acceptance Criteria:" | head -10)
fi

TOTAL_CRITERIA=0
if [ -n "$SPEC_ACCEPTANCE_CRITERIA" ]; then
    SPEC_CRITERIA_COUNT=$(echo "$SPEC_ACCEPTANCE_CRITERIA" | grep -c "^\- " || echo "0")
    TOTAL_CRITERIA=$((TOTAL_CRITERIA + SPEC_CRITERIA_COUNT))
fi

echo "   ✅ Extracted $TOTAL_CRITERIA acceptance criteria"
```

## Step 6: Create Review Context Document

```bash
echo "📝 Creating review context document..."

REVIEW_CACHE="$CACHE_PATH/review"
mkdir -p "$REVIEW_CACHE"

cat > "$REVIEW_CACHE/review-context.md" << CONTEXT_EOF
# Review Context

## Spec Information
- **Spec Path**: $SPEC_PATH
- **Spec File**: $([ -f "$SPEC_FILE" ] && echo "Found" || echo "Not found")
- **Requirements File**: $([ -f "$REQUIREMENTS_FILE" ] && echo "Found" || echo "Not found")
- **Tasks File**: $([ -f "$TASKS_FILE" ] && echo "Found" || echo "Not found")

## Implementation Files
$IMPL_FILES

## Modified Codebase Files
$MODIFIED_FILES

## Acceptance Criteria to Verify

### From Spec
$SPEC_ACCEPTANCE_CRITERIA

### From Tasks
$TASKS_ACCEPTANCE_CRITERIA

## Accumulated Knowledge Summary
$(echo "$ACCUMULATED_KNOWLEDGE" | head -50)
...

---

*Generated: $(date)*
CONTEXT_EOF

echo "✅ Review context saved to: $REVIEW_CACHE/review-context.md"
```

## Display Progress and Next Step

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 PHASE 1: LOAD CONTEXT COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Spec path: [SPEC_PATH]
✅ Implementation files: [count]
✅ Modified codebase files: [count]
✅ Acceptance criteria: [count]
✅ Accumulated knowledge: [loaded/not found]

Context saved to: [REVIEW_CACHE]/review-context.md

NEXT STEP 👉 Proceeding to Phase 2: Review Abstraction Layers
```
