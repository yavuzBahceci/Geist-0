# Phase 3: Enrich with Basepoints Knowledge

Load relevant basepoints and compare implementation against documented patterns.

## Step 1: Identify Relevant Basepoints

```bash
echo "🔍 Identifying relevant basepoints..."

REVIEW_CACHE="$SPEC_PATH/implementation/cache/review"

# Get modified files from context
MODIFIED_FILES=$(cat "$REVIEW_CACHE/review-context.md" 2>/dev/null | sed -n '/## Modified Codebase Files/,/## /p' | grep -v "^##" | grep -v "^$")

# Find basepoints for modified modules
RELEVANT_BASEPOINTS=""

echo "$MODIFIED_FILES" | while read file; do
    if [ -n "$file" ]; then
        # Extract module path
        MODULE_DIR=$(dirname "$file")
        
        # Look for corresponding basepoint
        BASEPOINT_FILE="geist/basepoints/$MODULE_DIR/agent-base-$(basename "$MODULE_DIR").md"
        
        if [ -f "$BASEPOINT_FILE" ]; then
            echo "   Found basepoint for: $MODULE_DIR"
            RELEVANT_BASEPOINTS="${RELEVANT_BASEPOINTS}\n$BASEPOINT_FILE"
        fi
    fi
done

# Also include parent basepoints
echo "   Checking parent basepoints..."

# Find all basepoints that might be relevant
ALL_BASEPOINTS=$(find geist/basepoints -name "agent-base-*.md" -type f 2>/dev/null | head -20)
BASEPOINT_COUNT=$(echo "$ALL_BASEPOINTS" | grep -c "." || echo "0")

echo "   Found $BASEPOINT_COUNT total basepoints"
```

## Step 2: Load Headquarter Knowledge

```bash
echo "📖 Loading headquarter knowledge..."

HEADQUARTER_FILE="geist/basepoints/headquarter.md"

if [ -f "$HEADQUARTER_FILE" ]; then
    HEADQUARTER_CONTENT=$(cat "$HEADQUARTER_FILE")
    echo "   ✅ Loaded headquarter.md"
    
    # Extract key sections
    ARCH_OVERVIEW=$(echo "$HEADQUARTER_CONTENT" | sed -n '/## Architecture/,/^## /p')
    PATTERNS=$(echo "$HEADQUARTER_CONTENT" | sed -n '/## Patterns/,/^## /p')
    STANDARDS=$(echo "$HEADQUARTER_CONTENT" | sed -n '/## Standards/,/^## /p')
else
    echo "   ⚠️ Headquarter not found"
    HEADQUARTER_CONTENT=""
fi
```

## Step 3: Compare Against Basepoint Patterns

```bash
echo "🔍 Comparing implementation against basepoint patterns..."

BASEPOINT_FINDINGS=""

# Load accumulated knowledge for pattern context
KNOWLEDGE_FILE="$SPEC_PATH/implementation/cache/accumulated-knowledge.md"
if [ -f "$KNOWLEDGE_FILE" ]; then
    ACCUMULATED_KNOWLEDGE=$(cat "$KNOWLEDGE_FILE")
fi

# For each relevant basepoint, compare implementation
echo "$RELEVANT_BASEPOINTS" | while read basepoint_file; do
    if [ -f "$basepoint_file" ]; then
        echo "   Comparing against: $(basename "$basepoint_file")"
        
        BASEPOINT_CONTENT=$(cat "$basepoint_file")
        
        # Extract patterns from basepoint
        BP_PATTERNS=$(echo "$BASEPOINT_CONTENT" | sed -n '/## Patterns/,/^## /p')
        BP_STANDARDS=$(echo "$BASEPOINT_CONTENT" | sed -n '/## Standards/,/^## /p')
        BP_FLOWS=$(echo "$BASEPOINT_CONTENT" | sed -n '/## Flows/,/^## /p')
        
        # Check for pattern compliance
        # [AI agent compares implementation against patterns]
        
        # Check for standards compliance
        # [AI agent compares implementation against standards]
        
        # Check for flow compliance
        # [AI agent compares implementation against documented flows]
    fi
done

echo "   ✅ Basepoint comparison complete"
```

## Step 4: Check Against Standards Files

```bash
echo "📋 Checking against standards files..."

STANDARDS_DIR="geist/standards"

if [ -d "$STANDARDS_DIR" ]; then
    # Find all standards files
    STANDARDS_FILES=$(find "$STANDARDS_DIR" -name "*.md" -type f)
    
    echo "   Found $(echo "$STANDARDS_FILES" | wc -l | tr -d ' ') standards files"
    
    # Key standards to check
    KEY_STANDARDS=(
        "coding-style.md"
        "conventions.md"
        "quality-assurance.md"
        "error-handling.md"
    )
    
    for standard in "${KEY_STANDARDS[@]}"; do
        STANDARD_FILE=$(find "$STANDARDS_DIR" -name "$standard" -type f | head -1)
        
        if [ -f "$STANDARD_FILE" ]; then
            echo "   Checking against: $standard"
            STANDARD_CONTENT=$(cat "$STANDARD_FILE")
            
            # [AI agent compares implementation against standards]
        fi
    done
else
    echo "   ⚠️ Standards directory not found"
fi
```

## Step 5: Store Basepoint Comparison Findings

```bash
echo "📝 Storing basepoint comparison findings..."

cat > "$REVIEW_CACHE/basepoint-findings.md" << BP_FINDINGS_EOF
# Basepoint Comparison Findings

## Review Date
$(date)

---

## Headquarter Compliance

### Architecture Alignment
[How well implementation aligns with documented architecture]

### Pattern Compliance
[How well implementation follows documented patterns]

---

## Module Basepoint Compliance

### Relevant Basepoints Reviewed
$RELEVANT_BASEPOINTS

### Findings by Module

[For each module basepoint reviewed:]

#### [Module Name]
- **Basepoint**: [path]
- **Pattern Compliance**: [✓/✗]
- **Standards Compliance**: [✓/✗]
- **Flow Compliance**: [✓/✗]
- **Issues Found**:
  - [List of issues]

---

## Standards Compliance

### Coding Style
[Compliance status]

### Conventions
[Compliance status]

### Quality Assurance
[Compliance status]

### Error Handling
[Compliance status]

---

## Summary of Basepoint-Related Issues

| Issue | Basepoint | Severity | Recommendation |
|-------|-----------|----------|----------------|
| [Issue 1] | [basepoint] | [severity] | [recommendation] |
| [Issue 2] | [basepoint] | [severity] | [recommendation] |

---

*Generated: $(date)*
BP_FINDINGS_EOF

echo "✅ Basepoint findings saved to: $REVIEW_CACHE/basepoint-findings.md"
```

## Display Progress and Next Step

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 PHASE 3: BASEPOINTS KNOWLEDGE ENRICHMENT COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Headquarter knowledge loaded
✅ Module basepoints compared: [count]
✅ Standards files checked: [count]
✅ Basepoint findings recorded

Findings saved to: [REVIEW_CACHE]/basepoint-findings.md

NEXT STEP 👉 Proceeding to Phase 4: Enrich with Library Knowledge
```
