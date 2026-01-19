Now that all cleanup operations are complete, proceed with generating a comprehensive cleanup report by following these instructions:

## Core Responsibilities

1. **Collect Cleanup Statistics**: Gather statistics from all cleanup phases
2. **Generate Comprehensive Report**: Create detailed cleanup report
3. **Store Report Appropriately**: Store report in appropriate location based on context
4. **Clean Up Temporary Files**: Clean up temporary cache files after report generation

## Workflow

### Step 1: Collect Cleanup Statistics

Gather statistics from all cleanup phases:

```bash
# Load cleanup cache
CLEANUP_CACHE="geist/.cleanup-cache"
VALIDATION_CACHE="$CLEANUP_CACHE/validation"
KNOWLEDGE_VERIFICATION_CACHE="$CLEANUP_CACHE/knowledge-verification"
DRY_RUN="${DRY_RUN:-false}"

# Load cleanup statistics
PLACEHOLDERS_CLEANED=$(cat "$CLEANUP_CACHE/placeholders-cleaned.txt" 2>/dev/null || echo "0")
UNNECESSARY_LOGIC_REMOVED=$(cat "$CLEANUP_CACHE/unnecessary-logic-removed.txt" 2>/dev/null || echo "0")
BROKEN_REFERENCES_FIXED=$(cat "$CLEANUP_CACHE/broken-references-fixed.txt" 2>/dev/null || echo "0")

FILES_MODIFIED_PLACEHOLDERS=$(cat "$CLEANUP_CACHE/files-modified-placeholders.txt" 2>/dev/null || echo "0")
FILES_MODIFIED_UNNECESSARY=$(cat "$CLEANUP_CACHE/files-modified-unnecessary-logic.txt" 2>/dev/null || echo "0")
FILES_MODIFIED_REFERENCES=$(cat "$CLEANUP_CACHE/files-modified-references.txt" 2>/dev/null || echo "0")

# Load knowledge verification status
KNOWLEDGE_STATUS=$(cat "$KNOWLEDGE_VERIFICATION_CACHE/verification-status.txt" 2>/dev/null || echo "UNKNOWN")

TOTAL_FIXES=$((PLACEHOLDERS_CLEANED + UNNECESSARY_LOGIC_REMOVED + BROKEN_REFERENCES_FIXED))
TOTAL_FILES_MODIFIED=$((FILES_MODIFIED_PLACEHOLDERS + FILES_MODIFIED_UNNECESSARY + FILES_MODIFIED_REFERENCES))
```

### Step 2: Determine Report Storage Location

Determine where to store the cleanup report:

```bash
# Check if running within spec context
if [ -d "geist/specs" ] && [ -n "$(find geist/specs -mindepth 1 -maxdepth 1 -type d | head -1)" ]; then
    # Find most recent spec directory
    LATEST_SPEC=$(find geist/specs -mindepth 1 -maxdepth 1 -type d | sort -r | head -1)
    REPORT_PATH="$LATEST_SPEC/implementation/cache/cleanup"
    mkdir -p "$REPORT_PATH"
    echo "📁 Storing cleanup report in spec context: $REPORT_PATH"
else
    # Store in cleanup cache (will be cleaned up)
    REPORT_PATH="$CLEANUP_CACHE/report"
    mkdir -p "$REPORT_PATH"
    echo "📁 Storing cleanup report in cleanup cache: $REPORT_PATH"
fi
```

### Step 3: Generate Comprehensive Cleanup Report

Create detailed cleanup report:

```bash
# Create JSON report
cat > "$REPORT_PATH/cleanup-report.json" << EOF
{
  "cleanup_timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "dry_run": $DRY_RUN,
  "summary": {
    "total_fixes": $TOTAL_FIXES,
    "total_files_modified": $TOTAL_FILES_MODIFIED,
    "placeholders_cleaned": $PLACEHOLDERS_CLEANED,
    "unnecessary_logic_removed": $UNNECESSARY_LOGIC_REMOVED,
    "broken_references_fixed": $BROKEN_REFERENCES_FIXED
  },
  "files_modified": {
    "by_placeholders": $FILES_MODIFIED_PLACEHOLDERS,
    "by_unnecessary_logic": $FILES_MODIFIED_UNNECESSARY,
    "by_references": $FILES_MODIFIED_REFERENCES,
    "total": $TOTAL_FILES_MODIFIED
  },
  "validation_reports": {
    "placeholder_validation": "$VALIDATION_CACHE/placeholder-cleaning-validation.json",
    "unnecessary_logic_validation": "$VALIDATION_CACHE/unnecessary-logic-removal-validation.json",
    "command_cycle_validation": "$VALIDATION_CACHE/command-cycle-validation.json",
    "structure_alignment_validation": "$VALIDATION_CACHE/project-structure-alignment-validation.json"
  },
  "knowledge_verification": {
    "status": "$KNOWLEDGE_STATUS",
    "report": "$KNOWLEDGE_VERIFICATION_CACHE/knowledge-gap-report.json",
    "summary": "$KNOWLEDGE_VERIFICATION_CACHE/knowledge-gap-report.md"
  }
}
EOF

# Create markdown summary report
cat > "$REPORT_PATH/cleanup-report-summary.md" << EOF
# Cleanup Agent-OS Report

**Cleanup Timestamp:** $(date -u +%Y-%m-%dT%H:%M:%SZ)  
**Mode:** $(if [ "$DRY_RUN" = "true" ]; then echo "DRY-RUN (Preview Only)"; else echo "APPLY (Changes Applied)"; fi)

## Summary

- **Total Fixes Applied:** $TOTAL_FIXES
- **Total Files Modified:** $TOTAL_FILES_MODIFIED
- **Placeholders Cleaned:** $PLACEHOLDERS_CLEANED
- **Unnecessary Logic Removed:** $UNNECESSARY_LOGIC_REMOVED
- **Broken References Fixed:** $BROKEN_REFERENCES_FIXED

## Details by Category

### Placeholder Cleaning
- **Placeholders Cleaned:** $PLACEHOLDERS_CLEANED
- **Files Modified:** $FILES_MODIFIED_PLACEHOLDERS

### Unnecessary Logic Removal
- **Issues Removed:** $UNNECESSARY_LOGIC_REMOVED
- **Files Modified:** $FILES_MODIFIED_UNNECESSARY

### Broken Reference Fixing
- **References Fixed:** $BROKEN_REFERENCES_FIXED
- **Files Modified:** $FILES_MODIFIED_REFERENCES

## Validation Reports

All validation reports are available in:
- \`$VALIDATION_CACHE/\`

## Knowledge Verification

**Knowledge Status:** $KNOWLEDGE_STATUS

$(if [ -f "$KNOWLEDGE_VERIFICATION_CACHE/knowledge-gap-report.md" ]; then
    echo "Knowledge gap report available: \`$KNOWLEDGE_VERIFICATION_CACHE/knowledge-gap-report.md\`"
fi)

$(if [ "$KNOWLEDGE_STATUS" = "CRITICAL" ]; then
    echo ""
    echo "⚠️  **CRITICAL**: Missing required knowledge files detected. Please create missing prerequisites before using specialized commands."
elif [ "$KNOWLEDGE_STATUS" = "WARNING" ]; then
    echo ""
    echo "⚠️  **WARNING**: Some knowledge may be incomplete. Review the knowledge gap report for details."
fi)

## Next Steps

$(if [ "$DRY_RUN" = "true" ]; then
    echo "This was a DRY-RUN. To apply these changes, run the cleanup command again without dry-run mode."
else
    echo "✅ Cleanup complete! Your geist has been cleaned and fixed."
    echo ""
    echo "You can now:"
    echo "1. Run validation again to verify all issues are resolved"
    echo "2. Use your cleaned geist commands"
fi)
EOF

# Display summary
cat "$REPORT_PATH/cleanup-report-summary.md"
```

### Step 4: Clean Up Temporary Files (if not dry-run)

Clean up temporary cache files after report generation:

```bash
if [ "$DRY_RUN" != "true" ]; then
    # Keep validation reports and final cleanup report, but clean up intermediate files
    echo "🧹 Cleaning up temporary files..."
    
    # Remove intermediate tracking files (keep reports)
    rm -f "$CLEANUP_CACHE"/*.txt 2>/dev/null
    rm -rf "$CLEANUP_CACHE/dry-run-preview" 2>/dev/null
    
    echo "✅ Temporary files cleaned up"
    echo "📁 Cleanup report stored in: $REPORT_PATH/"
else
    echo "🔍 DRY-RUN mode: Keeping all preview files in $CLEANUP_CACHE/"
fi
```

## Important Constraints

- Must collect statistics from all cleanup phases
- Must generate comprehensive cleanup report with all details
- Must store report in appropriate location (spec context or cleanup cache)
- Must clean up temporary files after report generation (unless dry-run)
- **CRITICAL**: 
  - If run within spec context: Store report in `geist/specs/[current-spec]/implementation/cache/cleanup/`
  - If run standalone: Store report in `geist/.cleanup-cache/report/` (temporary, can be cleaned up)
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
