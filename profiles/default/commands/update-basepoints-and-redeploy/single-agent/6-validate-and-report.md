The SIXTH AND FINAL STEP is to validate all updates and generate the comprehensive report:

{{workflows/validation/validate-incremental-update}}

## Phase 6 Actions

### 6.1 Run Validation

Validate all updated basepoints and re-specialized commands:

```bash
CACHE_DIR="geist/output/update-basepoints-and-redeploy/cache"
REPORTS_DIR="geist/output/update-basepoints-and-redeploy/reports"

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "🔍 RUNNING VALIDATION"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# The workflow will:
# 1. Validate updated basepoint markdown files
# 2. Validate re-specialized command files
# 3. Check for broken {{workflows/...}} and {{standards/...}} references
# 4. Verify knowledge cache consistency
# 5. Generate validation report

# Load validation results
VALIDATION_PASSED=true
ISSUES_COUNT=0
WARNINGS_COUNT=0

# Check if validation report was generated
if [ -f "$REPORTS_DIR/validation-report.md" ]; then
    VALIDATION_RESULT=$(grep "Overall Status:" "$REPORTS_DIR/validation-report.md" | head -1)
    
    if echo "$VALIDATION_RESULT" | grep -q "FAILED"; then
        VALIDATION_PASSED=false
        ISSUES_COUNT=$(grep -c "^### " "$REPORTS_DIR/validation-report.md" 2>/dev/null || echo "0")
    fi
fi

echo "   Validation complete"
echo "   Status: $([ "$VALIDATION_PASSED" = "true" ] && echo "✅ PASSED" || echo "❌ FAILED")"
```

### 6.2 Update Tracking Files

Update the tracking files for future incremental updates:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "💾 UPDATING TRACKING FILES"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Update git commit reference (if git available)
if [ -d ".git" ]; then
    CURRENT_COMMIT=$(git rev-parse HEAD)
    echo "$CURRENT_COMMIT" > "$CACHE_DIR/last-update-commit.txt"
    echo "   ✅ Git commit: $CURRENT_COMMIT"
fi

# Update timestamp
CURRENT_TIMESTAMP=$(date -u +%Y-%m-%dT%H:%M:%SZ)
echo "$CURRENT_TIMESTAMP" > "$CACHE_DIR/last-update-timestamp.txt"
echo "   ✅ Timestamp: $CURRENT_TIMESTAMP"

# Save update metadata
cat > "$CACHE_DIR/last-update-metadata.json" << EOF
{
  "commit": "$(git rev-parse HEAD 2>/dev/null || echo "N/A")",
  "timestamp": "$CURRENT_TIMESTAMP",
  "changes_detected": $(wc -l < "$CACHE_DIR/changed-files.txt" 2>/dev/null | tr -d ' ' || echo "0"),
  "basepoints_updated": $(wc -l < "$CACHE_DIR/updated-basepoints.txt" 2>/dev/null | tr -d ' ' || echo "0"),
  "commands_respecialized": 5,
  "validation_passed": $VALIDATION_PASSED
}
EOF
echo "   ✅ Metadata saved"
```

### 6.3 Generate Comprehensive Update Report

Create the final comprehensive report:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📋 GENERATING FINAL REPORT"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Collect statistics
CHANGES_DETECTED=$(wc -l < "$CACHE_DIR/changed-files.txt" 2>/dev/null | tr -d ' ' || echo "0")
ADDED_COUNT=$(wc -l < "$CACHE_DIR/added-files.txt" 2>/dev/null | tr -d ' ' || echo "0")
MODIFIED_COUNT=$(wc -l < "$CACHE_DIR/modified-files.txt" 2>/dev/null | tr -d ' ' || echo "0")
DELETED_COUNT=$(wc -l < "$CACHE_DIR/deleted-files.txt" 2>/dev/null | tr -d ' ' || echo "0")
BASEPOINTS_UPDATED=$(wc -l < "$CACHE_DIR/updated-basepoints.txt" 2>/dev/null | tr -d ' ' || echo "0")
PRODUCT_CHANGED=$(cat "$CACHE_DIR/product-files-changed.txt" 2>/dev/null || echo "false")

# Generate report
cat > "$REPORTS_DIR/update-report.md" << EOF
# Incremental Update Report

**Update Completed:** $(date -u +%Y-%m-%dT%H:%M:%SZ)
**Overall Status:** $([ "$VALIDATION_PASSED" = "true" ] && echo "✅ SUCCESS" || echo "❌ ISSUES FOUND")

---

## Summary

| Metric | Value |
|--------|-------|
| Codebase changes detected | $CHANGES_DETECTED files |
| Basepoints updated | $BASEPOINTS_UPDATED files |
| Commands re-specialized | 5 commands |
| Validation | $([ "$VALIDATION_PASSED" = "true" ] && echo "PASSED" || echo "FAILED") |

---

## Changes Detected

### By Category

| Category | Count |
|----------|-------|
| Added | $ADDED_COUNT |
| Modified | $MODIFIED_COUNT |
| Deleted | $DELETED_COUNT |
| **Total** | **$CHANGES_DETECTED** |

### Changed Files

$(cat "$CACHE_DIR/changed-files.txt" 2>/dev/null | head -30 | sed 's/^/- /')
$([ "$CHANGES_DETECTED" -gt 30 ] && echo "- _... and $((CHANGES_DETECTED - 30)) more_")

---

## Basepoints Updated

$(cat "$CACHE_DIR/updated-basepoints.txt" 2>/dev/null | sed 's/^/- /' || echo "_None_")

---

## Knowledge Changes

**Product files changed:** $PRODUCT_CHANGED

### Categories Updated
$(cat "$CACHE_DIR/knowledge-diff.md" 2>/dev/null | grep -A 20 "## Changed Knowledge" || echo "_See knowledge-diff.md for details_")

---

## Re-specialization

### Core Commands

All 5 core commands were re-specialized with updated knowledge:

| Command | Status |
|---------|--------|
| shape-spec | ✅ Updated |
| write-spec | ✅ Updated |
| create-tasks | ✅ Updated |
| implement-tasks | ✅ Updated |
| orchestrate-tasks | ✅ Updated |

### Supporting Structures

$(cat "$CACHE_DIR/respecialization-summary.md" 2>/dev/null | grep -A 10 "## Supporting Structures" || echo "_See respecialization-summary.md for details_")

---

## Validation Results

$(cat "$REPORTS_DIR/validation-report.md" 2>/dev/null | grep -A 30 "## Summary" || echo "See validation-report.md for details")

---

## Performance

| Metric | Value |
|--------|-------|
| Incremental update | Completed |
| Files processed | $CHANGES_DETECTED |
| Basepoints updated | $BASEPOINTS_UPDATED (vs full: all) |
| Efficiency | Incremental (faster than full regeneration) |

---

## Tracking Information

| Item | Value |
|------|-------|
| Git commit | $(git rev-parse HEAD 2>/dev/null || echo "N/A") |
| Timestamp | $CURRENT_TIMESTAMP |
| Cache location | $CACHE_DIR |

---

## Next Steps

$(if [ "$VALIDATION_PASSED" = "true" ]; then
    echo "✅ **Update completed successfully!**"
    echo ""
    echo "Your geist is now synchronized with your codebase changes."
    echo ""
    echo "You can now use the updated commands:"
    echo "- \`/shape-spec\` - Shape new specifications"
    echo "- \`/write-spec\` - Write detailed specifications"
    echo "- \`/create-tasks\` - Create implementation tasks"
    echo "- \`/implement-tasks\` - Implement tasks"
    echo "- \`/orchestrate-tasks\` - Orchestrate task execution"
else
    echo "⚠️ **Issues were found during validation**"
    echo ""
    echo "Please review the validation report and fix any issues:"
    echo "- Check: $REPORTS_DIR/validation-report.md"
    echo ""
    echo "After fixing issues, you can:"
    echo "1. Re-run \`/update-basepoints-and-redeploy\`"
    echo "2. Or manually fix and run \`/cleanup-geist\` to validate"
fi)

---

## Backup Information

Backup files were created during this update. To rollback:

1. Find backup files: \`find geist -name "*.backup"\`
2. Remove the updated file and rename backup
3. Or delete backups if update is confirmed good

EOF

echo "📋 Final report saved to: $REPORTS_DIR/update-report.md"
```

### 6.4 Cleanup (Optional)

Optionally cleanup backup files after successful validation:

```bash
# Note: Backups are kept by default for safety
# User can manually delete them after confirming the update is good

echo ""
echo "💡 Tip: Backup files have been preserved."
echo "   To cleanup after confirming update is good:"
echo "   find geist -name '*.backup' -delete"
```

## Display Final Completion Summary

Output the final completion message:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[✅/❌] UPDATE [COMPLETE/FAILED]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 FINAL SUMMARY

┌─────────────────────────────────────────────────────────┐
│ Phase 1: Change Detection          ✅ [N] files        │
│ Phase 2: Identify Basepoints       ✅ [N] basepoints   │
│ Phase 3: Update Basepoints         ✅ [N] updated      │
│ Phase 4: Re-extract Knowledge      ✅ Complete         │
│ Phase 5: Re-specialize Commands    ✅ 5 commands       │
│ Phase 6: Validate & Report         ✅ PASSED           │
└─────────────────────────────────────────────────────────┘

📋 Reports:
   • Full report: geist/output/update-basepoints-and-redeploy/reports/update-report.md
   • Validation:  geist/output/update-basepoints-and-redeploy/reports/validation-report.md

💾 Tracking updated for next incremental run

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ INCREMENTAL UPDATE COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Your geist is now synchronized with your latest codebase changes.
All core commands have been re-specialized with updated knowledge.

Next time you make changes, run `/update-basepoints-and-redeploy` again
for fast incremental synchronization.
```

If validation failed:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  UPDATE COMPLETED WITH ISSUES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 Issues Found: [N]

Please review:
   • Validation report: geist/output/update-basepoints-and-redeploy/reports/validation-report.md
   • Full report: geist/output/update-basepoints-and-redeploy/reports/update-report.md

Options:
1. Fix issues and re-run `/update-basepoints-and-redeploy`
2. Rollback using backup files
3. Run `/cleanup-geist` to identify specific issues
```

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your validation and reporting process aligns with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}

## Important Constraints

- **MUST run full validation** using the validation workflow
- **MUST update tracking files** for future incremental updates
- **MUST generate comprehensive report** with all phases summarized
- **MUST preserve backup files** - do not auto-delete
- **MUST clearly indicate success or failure** in final output
- Must provide actionable next steps based on validation results
- Must include performance comparison in report
- Must save all reports to the reports directory
