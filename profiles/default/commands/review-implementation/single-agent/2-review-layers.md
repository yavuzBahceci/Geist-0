# Phase 2: Review Abstraction Layers

Review implementation from highest abstraction layer to lowest, checking for issues at each layer.

## Abstraction Layer Review Strategy

```
Review Order (Top to Bottom):
1. Product Layer     - Does it meet business requirements?
2. Architecture Layer - Does it follow architectural patterns?
3. Module Layer      - Are modules organized correctly?
4. Code Layer        - Is code implemented correctly?
5. Test Layer        - Are tests adequate?
```

## Step 1: Review Product Layer

Check if implementation meets business requirements from spec:

```bash
echo "🏢 Reviewing Product Layer..."

FINDINGS=""
LAYER="Product"

# Load review context
REVIEW_CACHE="$SPEC_PATH/implementation/cache/review"
source_context() {
    if [ -f "$REVIEW_CACHE/review-context.md" ]; then
        return 0
    fi
    return 1
}

# Check against spec acceptance criteria
echo "   Checking against spec acceptance criteria..."

# Extract spec criteria
SPEC_CRITERIA=$(cat "$SPEC_PATH/spec.md" 2>/dev/null | sed -n '/## Acceptance Criteria/,/^## /p' | grep "^\- " || echo "")

if [ -n "$SPEC_CRITERIA" ]; then
    echo "$SPEC_CRITERIA" | while read criteria; do
        echo "   - Checking: $criteria"
        # [AI agent checks if criteria is met based on implementation]
    done
else
    echo "   ⚠️ No acceptance criteria found in spec"
    FINDINGS="${FINDINGS}\n- [Product] No acceptance criteria defined in spec"
fi

# Check if all user stories are addressed
echo "   Checking user stories coverage..."
USER_STORIES=$(cat "$SPEC_PATH/spec.md" 2>/dev/null | grep -E "As a .*, I want .*, so that" || echo "")

if [ -n "$USER_STORIES" ]; then
    STORY_COUNT=$(echo "$USER_STORIES" | wc -l | tr -d ' ')
    echo "   Found $STORY_COUNT user stories to verify"
else
    echo "   ⚠️ No user stories found"
fi

echo "   ✅ Product layer review complete"
```

## Step 2: Review Architecture Layer

Check if implementation follows architectural patterns:

```bash
echo "🏛️ Reviewing Architecture Layer..."

LAYER="Architecture"

# Check if implementation follows headquarter patterns
echo "   Checking against headquarter patterns..."

if [ -f "geist/basepoints/headquarter.md" ]; then
    HEADQUARTER=$(cat "geist/basepoints/headquarter.md")
    
    # Extract architecture patterns from headquarter
    ARCH_PATTERNS=$(echo "$HEADQUARTER" | sed -n '/## Architecture/,/^## /p')
    
    if [ -n "$ARCH_PATTERNS" ]; then
        echo "   Found architecture patterns to verify against"
    fi
fi

# Check folder structure alignment
echo "   Checking folder structure alignment..."

# Verify implementation files are in expected locations
# [AI agent checks if files follow expected structure]

# Check for separation of concerns
echo "   Checking separation of concerns..."

# Look for common architectural anti-patterns
ANTIPATTERNS=(
    "God class"
    "Circular dependencies"
    "Mixed concerns"
    "Hardcoded configuration"
)

for pattern in "${ANTIPATTERNS[@]}"; do
    echo "   - Checking for: $pattern"
done

echo "   ✅ Architecture layer review complete"
```

## Step 3: Review Module Layer

Check if modules are organized correctly:

```bash
echo "📦 Reviewing Module Layer..."

LAYER="Module"

# Get list of modified files from context
MODIFIED_FILES=$(cat "$REVIEW_CACHE/review-context.md" 2>/dev/null | sed -n '/## Modified Codebase Files/,/## /p' | grep -v "^##" | grep -v "^$")

if [ -n "$MODIFIED_FILES" ]; then
    echo "   Checking $( echo "$MODIFIED_FILES" | wc -l | tr -d ' ') modified files..."
    
    echo "$MODIFIED_FILES" | while read file; do
        if [ -f "$file" ]; then
            echo "   - Reviewing: $file"
            
            # Check module organization
            # - Single responsibility
            # - Proper imports/exports
            # - Module boundaries
        fi
    done
fi

# Check for module-level issues
echo "   Checking for module-level issues..."

# Look for:
# - Files in wrong directories
# - Missing index files
# - Inconsistent naming
# - Missing exports

echo "   ✅ Module layer review complete"
```

## Step 4: Review Code Layer

Check if code is implemented correctly:

```bash
echo "💻 Reviewing Code Layer..."

LAYER="Code"

# Check implementation against tasks
echo "   Checking implementation against tasks..."

# Load tasks
TASKS=$(cat "$SPEC_PATH/tasks.md" 2>/dev/null | grep "\- \[x\]" || echo "")
COMPLETED_COUNT=$(echo "$TASKS" | grep -c "\- \[x\]" || echo "0")
TOTAL_COUNT=$(cat "$SPEC_PATH/tasks.md" 2>/dev/null | grep -c "\- \[" || echo "0")

echo "   Tasks completed: $COMPLETED_COUNT / $TOTAL_COUNT"

if [ "$COMPLETED_COUNT" -lt "$TOTAL_COUNT" ]; then
    echo "   ⚠️ Some tasks are not marked complete"
    FINDINGS="${FINDINGS}\n- [Code] $((TOTAL_COUNT - COMPLETED_COUNT)) tasks not marked complete"
fi

# Check for code quality issues
echo "   Checking code quality..."

# Look for:
# - TODO comments left behind
# - console.log / print statements
# - Commented out code
# - Magic numbers
# - Missing error handling

echo "   ✅ Code layer review complete"
```

## Step 5: Review Test Layer

Check if tests are adequate:

```bash
echo "🧪 Reviewing Test Layer..."

LAYER="Test"

# Check if tests exist
echo "   Checking for test files..."

# Look for test files related to implementation
TEST_FILES=""

# Common test file patterns
for pattern in "*.test.ts" "*.test.js" "*.spec.ts" "*.spec.js" "*_test.py" "*_test.go" "*Test.java"; do
    found=$(find . -name "$pattern" 2>/dev/null | head -10)
    if [ -n "$found" ]; then
        TEST_FILES="${TEST_FILES}\n$found"
    fi
done

if [ -n "$TEST_FILES" ]; then
    TEST_COUNT=$(echo -e "$TEST_FILES" | grep -c "." || echo "0")
    echo "   Found $TEST_COUNT test files"
else
    echo "   ⚠️ No test files found"
    FINDINGS="${FINDINGS}\n- [Test] No test files found for implementation"
fi

# Check test coverage concerns
echo "   Checking test coverage concerns..."

# Look for:
# - Untested critical paths
# - Missing edge case tests
# - Integration test coverage

echo "   ✅ Test layer review complete"
```

## Step 6: Store Layer Review Findings

```bash
echo "📝 Storing layer review findings..."

cat > "$REVIEW_CACHE/layer-findings.md" << FINDINGS_EOF
# Abstraction Layer Review Findings

## Review Date
$(date)

## Spec Path
$SPEC_PATH

---

## Product Layer

### Acceptance Criteria Status
[Status of each acceptance criteria]

### User Stories Coverage
[Status of user story implementation]

### Issues Found
[List of product-level issues]

---

## Architecture Layer

### Pattern Compliance
[How well implementation follows architectural patterns]

### Structure Alignment
[How well files are organized]

### Issues Found
[List of architecture-level issues]

---

## Module Layer

### Module Organization
[Assessment of module organization]

### Issues Found
[List of module-level issues]

---

## Code Layer

### Task Completion
Completed: $COMPLETED_COUNT / $TOTAL_COUNT

### Code Quality
[Assessment of code quality]

### Issues Found
[List of code-level issues]

---

## Test Layer

### Test Coverage
[Assessment of test coverage]

### Issues Found
[List of test-level issues]

---

*Generated: $(date)*
FINDINGS_EOF

echo "✅ Layer findings saved to: $REVIEW_CACHE/layer-findings.md"
```

## Display Progress and Next Step

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 PHASE 2: REVIEW ABSTRACTION LAYERS COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Layers reviewed (top to bottom):
✅ Product Layer - [issues found]
✅ Architecture Layer - [issues found]
✅ Module Layer - [issues found]
✅ Code Layer - [issues found]
✅ Test Layer - [issues found]

Findings saved to: [REVIEW_CACHE]/layer-findings.md

NEXT STEP 👉 Proceeding to Phase 3: Enrich with Basepoints Knowledge
```
