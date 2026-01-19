# Phase 5: Generate Review Report

Consolidate all findings into a comprehensive review report.

**IMPORTANT**: This phase generates a REPORT ONLY. It does NOT auto-fix any issues.

## Step 1: Load All Findings

```bash
echo "📖 Loading all review findings..."

REVIEW_CACHE="$SPEC_PATH/implementation/cache/review"

# Load layer findings
LAYER_FINDINGS=""
if [ -f "$REVIEW_CACHE/layer-findings.md" ]; then
    LAYER_FINDINGS=$(cat "$REVIEW_CACHE/layer-findings.md")
    echo "   ✅ Loaded layer findings"
fi

# Load basepoint findings
BASEPOINT_FINDINGS=""
if [ -f "$REVIEW_CACHE/basepoint-findings.md" ]; then
    BASEPOINT_FINDINGS=$(cat "$REVIEW_CACHE/basepoint-findings.md")
    echo "   ✅ Loaded basepoint findings"
fi

# Load library findings
LIBRARY_FINDINGS=""
if [ -f "$REVIEW_CACHE/library-findings.md" ]; then
    LIBRARY_FINDINGS=$(cat "$REVIEW_CACHE/library-findings.md")
    echo "   ✅ Loaded library findings"
fi
```

## Step 2: Categorize Issues by Severity

```bash
echo "📊 Categorizing issues by severity..."

# Severity Levels:
# - CRITICAL: Blocks functionality, security issues, data loss risk
# - HIGH: Major functionality issues, significant performance problems
# - MEDIUM: Code quality issues, minor functionality issues
# - LOW: Style issues, minor improvements, suggestions

CRITICAL_ISSUES=""
HIGH_ISSUES=""
MEDIUM_ISSUES=""
LOW_ISSUES=""

# [AI agent categorizes all collected issues by severity]

# Count issues
CRITICAL_COUNT=0
HIGH_COUNT=0
MEDIUM_COUNT=0
LOW_COUNT=0

echo "   Critical issues: $CRITICAL_COUNT"
echo "   High issues: $HIGH_COUNT"
echo "   Medium issues: $MEDIUM_COUNT"
echo "   Low issues: $LOW_COUNT"
```

## Step 3: Generate Executive Summary

```bash
echo "📝 Generating executive summary..."

# Calculate overall health score
TOTAL_ISSUES=$((CRITICAL_COUNT + HIGH_COUNT + MEDIUM_COUNT + LOW_COUNT))

if [ "$CRITICAL_COUNT" -gt 0 ]; then
    HEALTH_STATUS="🔴 CRITICAL - Immediate attention required"
elif [ "$HIGH_COUNT" -gt 0 ]; then
    HEALTH_STATUS="🟠 NEEDS ATTENTION - High priority issues found"
elif [ "$MEDIUM_COUNT" -gt 0 ]; then
    HEALTH_STATUS="🟡 FAIR - Some improvements recommended"
else
    HEALTH_STATUS="🟢 GOOD - Minor or no issues found"
fi

echo "   Overall status: $HEALTH_STATUS"
```

## Step 4: Generate Final Report

```bash
echo "📄 Generating final review report..."

REPORT_FILE="$SPEC_PATH/implementation/review-report.md"

cat > "$REPORT_FILE" << REPORT_EOF
# Implementation Review Report

## Report Information
- **Spec**: $SPEC_PATH
- **Review Date**: $(date)
- **Report Type**: Review Only (No Auto-Fix)

---

## Executive Summary

### Overall Health Status
$HEALTH_STATUS

### Issue Summary
| Severity | Count |
|----------|-------|
| 🔴 Critical | $CRITICAL_COUNT |
| 🟠 High | $HIGH_COUNT |
| 🟡 Medium | $MEDIUM_COUNT |
| 🔵 Low | $LOW_COUNT |
| **Total** | **$TOTAL_ISSUES** |

### Key Findings
[AI-generated summary of the most important findings]

---

## Critical Issues 🔴

These issues require immediate attention:

[For each critical issue:]

### [Issue Title]
- **Layer**: [Product/Architecture/Module/Code/Test]
- **Location**: [File/Module]
- **Description**: [What's wrong]
- **Impact**: [Why it matters]
- **Recommendation**: [How to fix]
- **Reference**: [Basepoint/Standard that applies]

---

## High Priority Issues 🟠

These issues should be addressed soon:

[For each high priority issue:]

### [Issue Title]
- **Layer**: [Product/Architecture/Module/Code/Test]
- **Location**: [File/Module]
- **Description**: [What's wrong]
- **Impact**: [Why it matters]
- **Recommendation**: [How to fix]
- **Reference**: [Basepoint/Standard that applies]

---

## Medium Priority Issues 🟡

These issues should be addressed when possible:

[For each medium priority issue:]

### [Issue Title]
- **Layer**: [Product/Architecture/Module/Code/Test]
- **Location**: [File/Module]
- **Description**: [What's wrong]
- **Recommendation**: [How to fix]

---

## Low Priority Issues 🔵

These are suggestions and minor improvements:

[For each low priority issue:]

- **[Issue]**: [Description] → [Recommendation]

---

## Findings by Abstraction Layer

### Product Layer
[Summary of product-level findings]

### Architecture Layer
[Summary of architecture-level findings]

### Module Layer
[Summary of module-level findings]

### Code Layer
[Summary of code-level findings]

### Test Layer
[Summary of test-level findings]

---

## Basepoint Compliance Summary

### Headquarter Alignment
[How well implementation aligns with headquarter patterns]

### Module Basepoint Compliance
| Module | Compliance | Issues |
|--------|------------|--------|
| [module] | [%] | [count] |

### Standards Compliance
| Standard | Status |
|----------|--------|
| Coding Style | [✅/⚠️/❌] |
| Conventions | [✅/⚠️/❌] |
| Quality Assurance | [✅/⚠️/❌] |
| Error Handling | [✅/⚠️/❌] |

---

## Library Usage Summary

### Libraries Reviewed
| Library | Basepoint | Best Practices | Anti-Patterns |
|---------|-----------|----------------|---------------|
| [lib] | [✅/❌] | [status] | [count found] |

### Library-Specific Recommendations
[Specific recommendations for library usage]

---

## Acceptance Criteria Verification

| Criteria | Status | Notes |
|----------|--------|-------|
| [Criteria 1] | [✅/❌/⚠️] | [Notes] |
| [Criteria 2] | [✅/❌/⚠️] | [Notes] |

---

## Recommended Actions

### Immediate Actions (Critical/High)
1. [Action 1]
2. [Action 2]

### Short-term Actions (Medium)
1. [Action 1]
2. [Action 2]

### Future Improvements (Low)
1. [Action 1]
2. [Action 2]

---

## Notes

⚠️ **This is a review report only. No changes were made to the implementation.**

To address the issues found:
1. Review each issue in detail
2. Prioritize based on severity and impact
3. Make changes manually or create a new spec for fixes
4. Re-run \`/review-implementation\` to verify fixes

---

*Generated by Geist Review Implementation Command*
*This report does not auto-fix issues - all changes must be made manually*
REPORT_EOF

echo "✅ Review report generated: $REPORT_FILE"
```

## Step 5: Display Report Summary

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📊 IMPLEMENTATION REVIEW COMPLETE"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "Overall Status: $HEALTH_STATUS"
echo ""
echo "Issues Found:"
echo "   🔴 Critical: $CRITICAL_COUNT"
echo "   🟠 High: $HIGH_COUNT"
echo "   🟡 Medium: $MEDIUM_COUNT"
echo "   🔵 Low: $LOW_COUNT"
echo "   ────────────"
echo "   Total: $TOTAL_ISSUES"
echo ""
echo "📄 Full report: $REPORT_FILE"
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "⚠️  IMPORTANT: This command DOES NOT auto-fix issues."
echo "    Review the report and make changes manually."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

## Important Notes

- This command generates a REPORT ONLY
- NO changes are made to the implementation
- All issues must be addressed manually
- Re-run the command after fixes to verify resolution

## Output Files

| File | Purpose |
|------|---------|
| `implementation/review-report.md` | Final consolidated report |
| `implementation/cache/review/review-context.md` | Review context |
| `implementation/cache/review/layer-findings.md` | Layer review findings |
| `implementation/cache/review/basepoint-findings.md` | Basepoint comparison findings |
| `implementation/cache/review/library-findings.md` | Library review findings |
