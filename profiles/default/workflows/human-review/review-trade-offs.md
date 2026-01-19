# Human Review for Trade-offs

---

## ⚠️ NOTE: This workflow may require user interaction

If trade-offs or contradictions are detected, this workflow will:
1. Present all issues to the user
2. **STOP and WAIT** for the user to make decisions
3. Log the user's decisions for future reference

If no issues are detected, the workflow proceeds automatically.

---

## Core Responsibilities

1. **Orchestrate Trade-off Detection**: Trigger detection workflows for trade-offs and contradictions
2. **Present Trade-offs**: Format and present detected issues for human review
3. **Capture Human Decision**: Wait for and record user decision
4. **Store Review Results**: Cache decisions for use in subsequent workflow steps

## Workflow

### Step 1: Determine If Review Is Needed

```bash
# SPEC_PATH should be set by the calling command
if [ -z "$SPEC_PATH" ]; then
    echo "⚠️ SPEC_PATH not set. Cannot perform human review."
    exit 1
fi

echo "🔍 Checking if human review is needed..."

CACHE_PATH="$SPEC_PATH/implementation/cache"
REVIEW_PATH="$CACHE_PATH/human-review"
mkdir -p "$REVIEW_PATH"

# Initialize review flags
NEEDS_TRADE_OFF_REVIEW="false"
NEEDS_CONTRADICTION_REVIEW="false"
REVIEW_TRIGGERED="false"
```

### Step 2: Run Trade-off Detection

```bash
# Determine workflow base path (geist when installed, profiles/default for template)
if [ -d "geist/workflows" ]; then
    WORKFLOWS_BASE="geist/workflows"
else
    WORKFLOWS_BASE="profiles/default/workflows"
fi

echo "📊 Running trade-off detection..."

# Execute detect-trade-offs workflow
# This workflow compares proposed approach against basepoints patterns
source "$WORKFLOWS_BASE/human-review/detect-trade-offs.md"

# Check results
if [ -f "$REVIEW_PATH/trade-offs.md" ]; then
    TRADE_OFF_COUNT=$(grep -c "TRADE-OFF-" "$REVIEW_PATH/trade-offs.md" 2>/dev/null || echo "0")
    
    if [ "$TRADE_OFF_COUNT" -gt 0 ]; then
        NEEDS_TRADE_OFF_REVIEW="true"
        echo "   Found $TRADE_OFF_COUNT trade-offs"
    else
        echo "   No significant trade-offs found"
    fi
fi
```

### Step 2.5: Run SDD Trade-off Detection (SDD-aligned)

After running standard trade-off detection, check for SDD-specific trade-offs:

```bash
echo "📊 Running SDD trade-off detection..."

SPEC_FILE="$SPEC_PATH/spec.md"
REQUIREMENTS_FILE="$SPEC_PATH/planning/requirements.md"
TASKS_FILE="$SPEC_PATH/tasks.md"
IMPLEMENTATION_PATH="$SPEC_PATH/implementation"

# Initialize SDD trade-off tracking
SDD_TRADE_OFF_COUNT=0
SDD_TRADE_OFFS=""

# Check for spec-implementation drift (when implementation exists and diverges from spec)
if [ -f "$SPEC_FILE" ] && [ -d "$IMPLEMENTATION_PATH" ]; then
    # Check if implementation exists
    if find "$IMPLEMENTATION_PATH" -name "*.md" -o -name "*.js" -o -name "*.py" -o -name "*.ts" 2>/dev/null | head -1 | grep -q .; then
        # Implementation exists - check for drift
        # This is a simplified check - actual drift detection would compare spec requirements to implementation
        # For now, we check if spec and implementation align structurally
        SPEC_AC_COUNT=$(grep -c "Acceptance Criteria:" "$SPEC_FILE" 2>/dev/null || echo "0")
        TASKS_AC_COUNT=$(grep -c "Acceptance Criteria:" "$TASKS_FILE" 2>/dev/null || echo "0")
        
        if [ "$SPEC_AC_COUNT" -gt 0 ] && [ "$TASKS_AC_COUNT" -gt 0 ] && [ "$SPEC_AC_COUNT" -ne "$TASKS_AC_COUNT" ]; then
            SDD_TRADE_OFF_COUNT=$((SDD_TRADE_OFF_COUNT + 1))
            SDD_TRADE_OFFS="${SDD_TRADE_OFFS}\nTRADE-OFF-SDD-001: Spec-implementation drift detected. Spec has $SPEC_AC_COUNT acceptance criteria, but tasks have $TASKS_AC_COUNT. Implementation may be diverging from spec (SDD principle: spec as source of truth)."
        fi
    fi
fi

# Check for premature technical decisions in spec phase (violates SDD "What & Why" principle)
if [ -f "$SPEC_FILE" ] || [ -f "$REQUIREMENTS_FILE" ]; then
    # Check spec file for premature technical details
    if [ -f "$SPEC_FILE" ]; then
        PREMATURE_TECH=$(grep -iE "implementation details|code structure|database schema|api endpoints|class hierarchy|architecture diagram|tech stack|framework|library|npm package|import|require" "$SPEC_FILE" | wc -l)
        
        if [ "$PREMATURE_TECH" -gt 5 ]; then
            SDD_TRADE_OFF_COUNT=$((SDD_TRADE_OFF_COUNT + 1))
            SDD_TRADE_OFFS="${SDD_TRADE_OFFS}\nTRADE-OFF-SDD-002: Premature technical decisions detected in spec ($PREMATURE_TECH instances). Spec should focus on What & Why, not How (SDD principle). Technical details belong in task creation/implementation phase."
        fi
    fi
    
    # Check requirements file for premature technical details
    if [ -f "$REQUIREMENTS_FILE" ]; then
        PREMATURE_TECH_REQ=$(grep -iE "implementation details|code structure|database schema|api endpoints|class hierarchy|architecture diagram|tech stack|framework|library|npm package|import|require" "$REQUIREMENTS_FILE" | wc -l)
        
        if [ "$PREMATURE_TECH_REQ" -gt 5 ]; then
            SDD_TRADE_OFF_COUNT=$((SDD_TRADE_OFF_COUNT + 1))
            SDD_TRADE_OFFS="${SDD_TRADE_OFFS}\nTRADE-OFF-SDD-003: Premature technical decisions detected in requirements ($PREMATURE_TECH_REQ instances). Requirements should focus on What & Why, not How (SDD principle)."
        fi
    fi
fi

# Check for over-specification or feature bloat (violates SDD "minimal, intentional scope" principle)
if [ -f "$SPEC_FILE" ]; then
    # Check spec file size (over-specification indicator)
    SPEC_LINE_COUNT=$(wc -l < "$SPEC_FILE" 2>/dev/null || echo "0")
    SPEC_SECTION_COUNT=$(grep -c "^##" "$SPEC_FILE" 2>/dev/null || echo "0")
    
    # Heuristic: If spec has more than 500 lines or more than 15 sections, it might be over-specified
    if [ "$SPEC_LINE_COUNT" -gt 500 ] || [ "$SPEC_SECTION_COUNT" -gt 15 ]; then
        SDD_TRADE_OFF_COUNT=$((SDD_TRADE_OFF_COUNT + 1))
        SDD_TRADE_OFFS="${SDD_TRADE_OFFS}\nTRADE-OFF-SDD-004: Over-specification detected. Spec has $SPEC_LINE_COUNT lines and $SPEC_SECTION_COUNT sections. May violate SDD 'minimal, intentional scope' principle. Consider breaking into smaller, focused specs."
    fi
fi

# If SDD trade-offs found, add to trade-offs file
if [ "$SDD_TRADE_OFF_COUNT" -gt 0 ]; then
    echo "   Found $SDD_TRADE_OFF_COUNT SDD-specific trade-offs"
    
    # Append SDD trade-offs to trade-offs file
    if [ -f "$REVIEW_PATH/trade-offs.md" ]; then
        echo "" >> "$REVIEW_PATH/trade-offs.md"
        echo "## SDD-Specific Trade-offs" >> "$REVIEW_PATH/trade-offs.md"
        echo -e "$SDD_TRADE_OFFS" >> "$REVIEW_PATH/trade-offs.md"
    else
        # Create new trade-offs file with SDD trade-offs
        cat > "$REVIEW_PATH/trade-offs.md" << EOF
# Trade-offs Detected

## SDD-Specific Trade-offs
$(echo -e "$SDD_TRADE_OFFS")

EOF
    fi
    
    NEEDS_TRADE_OFF_REVIEW="true"
else
    echo "   No SDD-specific trade-offs found"
fi
```

### Step 3: Run Contradiction Detection

```bash
echo "📏 Running contradiction detection..."

# Execute detect-contradictions workflow
# This workflow compares proposed approach against standards
source "$WORKFLOWS_BASE/human-review/detect-contradictions.md"

# Check results
if [ -f "$REVIEW_PATH/contradictions.md" ]; then
    CRITICAL_COUNT=$(grep "Critical:" "$REVIEW_PATH/contradictions.md" 2>/dev/null | grep -oE '[0-9]+' | head -1 || echo "0")
    WARNING_COUNT=$(grep "Warnings:" "$REVIEW_PATH/contradictions.md" 2>/dev/null | grep -oE '[0-9]+' | head -1 || echo "0")
    
    if [ "$CRITICAL_COUNT" -gt 0 ]; then
        NEEDS_CONTRADICTION_REVIEW="true"
        REVIEW_URGENCY="REQUIRED"
        echo "   ⛔ Found $CRITICAL_COUNT critical contradictions - Review REQUIRED"
    elif [ "$WARNING_COUNT" -gt 0 ]; then
        NEEDS_CONTRADICTION_REVIEW="true"
        REVIEW_URGENCY="RECOMMENDED"
        echo "   ⚠️ Found $WARNING_COUNT warning contradictions - Review RECOMMENDED"
    else
        echo "   No contradictions found"
    fi
fi
```

### Step 4: Determine Review Necessity

```bash
# Determine if any review is needed
if [ "$NEEDS_TRADE_OFF_REVIEW" = "true" ] || [ "$NEEDS_CONTRADICTION_REVIEW" = "true" ]; then
    REVIEW_TRIGGERED="true"
fi

echo ""
echo "═══════════════════════════════════════════════════════"
echo "  REVIEW NECESSITY CHECK"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Trade-off Review Needed: $NEEDS_TRADE_OFF_REVIEW"
echo "  Contradiction Review Needed: $NEEDS_CONTRADICTION_REVIEW"
echo "  Overall Review Triggered: $REVIEW_TRIGGERED"
if [ -n "$REVIEW_URGENCY" ]; then
    echo "  Review Urgency: $REVIEW_URGENCY"
fi
echo ""
echo "═══════════════════════════════════════════════════════"
```

### Step 5: Present For Human Review (If Needed)

```bash
if [ "$REVIEW_TRIGGERED" = "true" ]; then
    echo ""
    echo "👤 Presenting for human review..."
    echo ""
    
    # Execute present-human-decision workflow
    source "$WORKFLOWS_BASE/human-review/present-human-decision.md"
    
    # The presentation workflow will:
    # 1. Format all detected issues
    # 2. Provide AI recommendation
    # 3. Present decision options
    # 4. Wait for user input
else
    echo ""
    echo "✅ No human review needed. Proceeding automatically."
    echo ""
    
    # Create a "no review needed" log
    cat > "$REVIEW_PATH/review-result.md" << NO_REVIEW_EOF
# Trade-off Review Result

**Date**: $(date)
**Spec Path**: $SPEC_PATH
**Review Triggered**: No

## Summary

No significant trade-offs or contradictions were detected that require human review.

The analysis checked:
- Multiple valid patterns from basepoints
- Conflicts between proposal and documented patterns
- Mission/roadmap alignment
- Standard compliance

**Result**: Proceed with implementation.

NO_REVIEW_EOF
fi
```

### Step 6: Process User Decision (When Review Is Triggered)

```bash
# This section handles user response after presentation
# USER_RESPONSE should be provided by the user

process_user_decision() {
    USER_RESPONSE="$1"
    
    echo "📝 Processing user decision: $USER_RESPONSE"
    
    # Parse decision type
    case "$USER_RESPONSE" in
        "proceed"|"Proceed"|"PROCEED")
            DECISION="proceed"
            DECISION_REASON="User approved proceeding as-is"
            ;;
        "stop"|"Stop"|"STOP")
            DECISION="stop"
            DECISION_REASON="User requested halt"
            ;;
        "accept"|"Accept"|"ACCEPT")
            DECISION="accept_recommendation"
            DECISION_REASON="User accepted AI recommendation"
            ;;
        *)
            DECISION="custom"
            DECISION_REASON="$USER_RESPONSE"
            ;;
    esac
    
    # Log the decision
    cat > "$REVIEW_PATH/review-result.md" << REVIEW_RESULT_EOF
# Trade-off Review Result

**Date**: $(date)
**Spec Path**: $SPEC_PATH
**Review Triggered**: Yes

## Human Decision

**Decision**: $DECISION
**Reason**: $DECISION_REASON

## Issues Reviewed

### Trade-offs
$([ -f "$REVIEW_PATH/trade-offs.md" ] && grep "TRADE-OFF-" "$REVIEW_PATH/trade-offs.md" | head -5 || echo "None")

### Contradictions
$([ -f "$REVIEW_PATH/contradictions.md" ] && grep -E "⛔|⚠️" "$REVIEW_PATH/contradictions.md" | head -5 || echo "None")

## Outcome

$(if [ "$DECISION" = "proceed" ] || [ "$DECISION" = "accept_recommendation" ]; then
    echo "✅ Approved to proceed with implementation"
elif [ "$DECISION" = "stop" ]; then
    echo "⛔ Implementation halted by user"
else
    echo "📝 Custom resolution applied"
fi)

---

*Review completed by human-review workflow*
REVIEW_RESULT_EOF

    echo "✅ Decision logged to: $REVIEW_PATH/review-result.md"
    
    # Return decision for calling workflow
    echo "$DECISION"
}

# Export function
export -f process_user_decision 2>/dev/null || true
```

### Step 7: Return Review Status

```bash
# Store final review status
cat > "$REVIEW_PATH/review-status.txt" << STATUS_EOF
REVIEW_TRIGGERED=$REVIEW_TRIGGERED
NEEDS_TRADE_OFF_REVIEW=$NEEDS_TRADE_OFF_REVIEW
NEEDS_CONTRADICTION_REVIEW=$NEEDS_CONTRADICTION_REVIEW
REVIEW_URGENCY=${REVIEW_URGENCY:-NONE}
STATUS_EOF

echo ""
echo "═══════════════════════════════════════════════════════"
echo "  HUMAN REVIEW WORKFLOW COMPLETE"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Review Triggered: $REVIEW_TRIGGERED"
echo "  Status File: $REVIEW_PATH/review-status.txt"
echo ""
echo "═══════════════════════════════════════════════════════"

# Export for use by calling command
export REVIEW_TRIGGERED="$REVIEW_TRIGGERED"
export REVIEW_URGENCY="${REVIEW_URGENCY:-NONE}"
```

## Integration with Commands

Commands should call this workflow at key decision points:

1. **shape-spec**: After gathering requirements, before finalizing
2. **write-spec**: Before completing spec document
3. **create-tasks**: When tasks affect multiple layers
4. **implement-tasks**: Before implementing cross-cutting changes

## Important Constraints

- Must orchestrate both trade-off and contradiction detection
- Must present formatted issues for human review
- Must wait for user confirmation before proceeding on critical issues
- Must log all decisions for future reference
- Must integrate with basepoints knowledge for context
- **CRITICAL**: All review results stored in `$SPEC_PATH/implementation/cache/human-review/`

## SDD Integration Notes

This workflow has been enhanced with Spec-Driven Development (SDD) best practices:

**SDD Trade-off Detection:**
- **Spec-Implementation Drift**: Detects when implementation exists and diverges from spec (violates SDD "spec as source of truth" principle)
- **Premature Technical Decisions**: Identifies technical details in spec phase (violates SDD "What & Why, not How" principle)
- **Over-Specification**: Flags excessive scope or feature bloat (violates SDD "minimal, intentional scope" principle)

**Technology-Agnostic Approach (Default Profile Templates Only):**
- All SDD trade-off detection is structure-based, not technology-specific
- No hardcoded technology-specific references in default templates
- Detection maintains technology-agnostic state throughout **in default profile templates**
- **After Specialization:** When templates are compiled to `geist/workflows/`, workflows can and should become technology-specific based on the project's actual stack
- **Command Outputs:** Specs, tasks, and implementations should reflect the project's actual technology stack
