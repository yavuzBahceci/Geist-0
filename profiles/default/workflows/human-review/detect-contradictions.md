# Detect Contradictions

## Core Responsibilities

1. **Load Standards and Basepoints Knowledge**: Read documented standards from multiple sources
2. **Compare Against Proposed Implementation**: Detect conflicts with documented standards
3. **Identify Standard Violations**: Flag when proposed approach violates standards
4. **Classify Severity**: Distinguish critical vs warning level contradictions
5. **Output Contradictions List**: Generate list for human review

## Workflow

### Step 1: Load Standards from Multiple Sources

```bash
# SPEC_PATH should be set by the calling command
if [ -z "$SPEC_PATH" ]; then
    echo "⚠️ SPEC_PATH not set. Cannot detect contradictions."
    exit 1
fi

echo "🔍 Detecting contradictions..."

CACHE_PATH="$SPEC_PATH/implementation/cache"
STANDARDS_PATH="geist/standards"
BASEPOINTS_PATH="geist/basepoints"

# Initialize standards collection
ALL_STANDARDS=""

# Load from basepoints knowledge cache
if [ -f "$CACHE_PATH/basepoints-knowledge.md" ]; then
    BASEPOINT_STANDARDS=$(cat "$CACHE_PATH/basepoints-knowledge.md" | \
        sed -n '/## Standards/,/^## [^S]/p' | head -n -1)
    ALL_STANDARDS="${ALL_STANDARDS}

## From Basepoints
${BASEPOINT_STANDARDS}"
    echo "✅ Loaded standards from basepoints knowledge"
fi

# Load from standards directory
if [ -d "$STANDARDS_PATH" ]; then
    for std_file in $(find "$STANDARDS_PATH" -name "*.md" -type f 2>/dev/null); do
        STD_CONTENT=$(cat "$std_file" 2>/dev/null)
        STD_NAME=$(basename "$std_file" .md)
        ALL_STANDARDS="${ALL_STANDARDS}

## From $STD_NAME
${STD_CONTENT}"
    done
    echo "✅ Loaded standards from standards directory"
fi

# Load from headquarter.md
if [ -f "$BASEPOINTS_PATH/headquarter.md" ]; then
    HQ_STANDARDS=$(sed -n '/## Standards/,/^## [^S]/p' "$BASEPOINTS_PATH/headquarter.md" | head -n -1)
    ALL_STANDARDS="${ALL_STANDARDS}

## From Headquarter
${HQ_STANDARDS}"
    echo "✅ Loaded standards from headquarter"
fi

STANDARDS_LOADED=$([ -n "$ALL_STANDARDS" ] && echo "true" || echo "false")
echo "   Standards loaded: $STANDARDS_LOADED"
```

### Step 2: Load Proposed Implementation

```bash
echo "📋 Loading proposed implementation..."

PROPOSED=""

# Load from spec documents
if [ -f "$SPEC_PATH/spec.md" ]; then
    PROPOSED=$(cat "$SPEC_PATH/spec.md")
    echo "   Loaded from spec.md"
elif [ -f "$SPEC_PATH/planning/requirements.md" ]; then
    PROPOSED=$(cat "$SPEC_PATH/planning/requirements.md")
    echo "   Loaded from requirements.md"
fi

# Also check for any implementation hints
if [ -f "$SPEC_PATH/tasks.md" ]; then
    PROPOSED="${PROPOSED}
$(cat "$SPEC_PATH/tasks.md")"
fi

if [ -z "$PROPOSED" ]; then
    echo "⚠️ No proposed implementation found"
    PROPOSED="${FEATURE_DESCRIPTION:-}"
fi
```

### Step 3: Extract Standard Rules

```bash
echo "📏 Extracting standard rules..."

# Extract explicit rules (must, should, always, never patterns)
MUST_RULES=$(echo "$ALL_STANDARDS" | grep -iE 'must|required|always|never|mandatory' | head -30)
SHOULD_RULES=$(echo "$ALL_STANDARDS" | grep -iE 'should|recommended|prefer|avoid' | head -30)

# Extract naming conventions
NAMING_RULES=$(echo "$ALL_STANDARDS" | grep -iE 'naming|name|convention|format|case' | head -20)

# Extract structural rules
STRUCTURE_RULES=$(echo "$ALL_STANDARDS" | grep -iE 'structure|folder|directory|path|location|file' | head -20)

RULE_COUNT=$(($(echo "$MUST_RULES" | grep -c '.' || echo "0") + \
              $(echo "$SHOULD_RULES" | grep -c '.' || echo "0")))
echo "   Extracted $RULE_COUNT rules"
```

### Step 4: Check for Contradictions

```bash
echo "⚔️ Checking for contradictions..."

CONTRADICTIONS=""
CONTRADICTION_COUNT=0

# Convert to lowercase for comparison
PROPOSED_LOWER=$(echo "$PROPOSED" | tr '[:upper:]' '[:lower:]')
STANDARDS_LOWER=$(echo "$ALL_STANDARDS" | tr '[:upper:]' '[:lower:]')

# Check placeholder violations
if echo "$STANDARDS_LOWER" | grep -q "placeholder"; then
    if echo "$PROPOSED_LOWER" | grep -q "hardcode\|hard-code\|literal value"; then
        CONTRADICTIONS="${CONTRADICTIONS}
CONTRADICTION-PLACEHOLDER: Proposal may hardcode values that should use placeholders
Severity: warning
"
        CONTRADICTION_COUNT=$((CONTRADICTION_COUNT + 1))
    fi
fi

# Check cache location violations
if echo "$STANDARDS_LOWER" | grep -q "cache.*implementation/cache\|must be stored"; then
    if echo "$PROPOSED_LOWER" | grep -q "scattered\|different location"; then
        CONTRADICTIONS="${CONTRADICTIONS}
CONTRADICTION-CACHE: Proposal may not follow cache location standards
Severity: warning
"
        CONTRADICTION_COUNT=$((CONTRADICTION_COUNT + 1))
    fi
fi

# Check naming convention violations (kebab-case for Geist)
if echo "$STANDARDS_LOWER" | grep -qi "kebab-case"; then
    if echo "$PROPOSED_LOWER" | grep -oE '[a-z]+_[a-z]+' | head -1 | grep -q '.'; then
        CONTRADICTIONS="${CONTRADICTIONS}
CONTRADICTION-NAMING: Proposal may use snake_case instead of kebab-case
Severity: info
"
        CONTRADICTION_COUNT=$((CONTRADICTION_COUNT + 1))
    fi
fi

echo "   Found $CONTRADICTION_COUNT potential contradictions"
```

### Step 5: Classify Severity

```bash
echo "⚠️ Classifying contradiction severity..."

CRITICAL_CONTRADICTIONS=""
WARNING_CONTRADICTIONS=""
INFO_CONTRADICTIONS=""

# Parse contradictions and classify
CRITICAL_COUNT=0
WARNING_COUNT=0
INFO_COUNT=0

# Count by severity
CRITICAL_COUNT=$(echo "$CONTRADICTIONS" | grep -c "Severity: critical" || echo "0")
WARNING_COUNT=$(echo "$CONTRADICTIONS" | grep -c "Severity: warning" || echo "0")
INFO_COUNT=$(echo "$CONTRADICTIONS" | grep -c "Severity: info" || echo "0")

echo "   Critical: $CRITICAL_COUNT"
echo "   Warnings: $WARNING_COUNT"
echo "   Info: $INFO_COUNT"
```

### Step 6: Generate Contradictions Report

```bash
echo "📝 Generating contradictions report..."

# Determine if human review is required
if [ "$CRITICAL_COUNT" -gt 0 ]; then
    NEEDS_HUMAN_REVIEW="true"
    REVIEW_URGENCY="REQUIRED"
elif [ "$WARNING_COUNT" -gt 0 ]; then
    NEEDS_HUMAN_REVIEW="true"
    REVIEW_URGENCY="RECOMMENDED"
else
    NEEDS_HUMAN_REVIEW="false"
    REVIEW_URGENCY="OPTIONAL"
fi

# Store contradictions report
mkdir -p "$CACHE_PATH/human-review"

cat > "$CACHE_PATH/human-review/contradictions.md" << CONTRADICTIONS_EOF
# Detected Contradictions

**Analyzed**: $(date)
**Spec Path**: $SPEC_PATH
**Total Contradictions**: $CONTRADICTION_COUNT
**Critical**: $CRITICAL_COUNT
**Warnings**: $WARNING_COUNT
**Informational**: $INFO_COUNT
**Human Review**: $REVIEW_URGENCY

---

## Contradictions Detected

$CONTRADICTIONS

---

## Standards Sources Checked

- Basepoints knowledge cache
- Standards directory (geist/standards/)
- Headquarter.md

## Resolution Actions

$(if [ "$CRITICAL_COUNT" -gt 0 ]; then
    echo "1. ⛔ STOP: Critical contradictions must be resolved before proceeding"
    echo "2. Review each critical contradiction against source standard"
    echo "3. Either modify proposal or request standard exception"
elif [ "$WARNING_COUNT" -gt 0 ]; then
    echo "1. Review warning-level contradictions"
    echo "2. Decide if deviations are acceptable for this feature"
    echo "3. Document any intentional deviations"
else
    echo "No blocking contradictions. Proceed with caution."
fi)

---

*Generated by detect-contradictions workflow*
CONTRADICTIONS_EOF

echo "✅ Contradictions stored at: $CACHE_PATH/human-review/contradictions.md"
```

### Step 7: Return Results

```bash
echo ""
echo "═══════════════════════════════════════════════════════"
echo "  CONTRADICTION DETECTION COMPLETE"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Contradictions Found: $CONTRADICTION_COUNT"
echo "  Critical: $CRITICAL_COUNT"
echo "  Warnings: $WARNING_COUNT"
echo "  Human Review: $REVIEW_URGENCY"
echo ""
echo "═══════════════════════════════════════════════════════"

# Export for use by other workflows
export CONTRADICTION_COUNT="$CONTRADICTION_COUNT"
export CRITICAL_COUNT="$CRITICAL_COUNT"
export NEEDS_HUMAN_REVIEW="$NEEDS_HUMAN_REVIEW"
```

## Important Constraints

- Must load standards from basepoints, standards directory, and headquarter
- Must detect conflicts between proposed implementation and documented standards
- Must identify and classify standard violations by severity
- Must flag critical contradictions as blocking
- **CRITICAL**: Contradictions stored in `$SPEC_PATH/implementation/cache/human-review/contradictions.md`
