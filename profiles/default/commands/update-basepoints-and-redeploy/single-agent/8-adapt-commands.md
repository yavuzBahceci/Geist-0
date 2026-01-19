# Phase 8: Adapt Commands

Apply user-approved adaptations to command templates.

## Prerequisites

- User has reviewed learnings from Phase 7
- User has approved specific adaptations (all or select)
- Adaptations file exists: `geist/output/session-feedback/adaptations/pending.md`

## Core Responsibilities

1. **Load Approved Adaptations**: Read user-approved adaptations from pending.md
2. **Backup Commands**: Create timestamped backup of all command files
3. **Apply Adaptations**: Apply approved changes using workflow
4. **Update Applied Log**: Move adaptations from pending to applied.md
5. **Archive Session**: Move session to history and start fresh
6. **Generate Report**: Document what was changed

## Workflow

### Step 1: Check Prerequisites

```bash
PENDING_FILE="geist/output/session-feedback/adaptations/pending.md"

if [ ! -f "$PENDING_FILE" ]; then
    echo "ℹ️ No pending adaptations found. Skipping adaptation phase."
    exit 0
fi

# Check approval status
APPROVAL_STATUS=$(grep "Approval Status:" "$PENDING_FILE" | sed 's/.*: \([^ ]*\).*/\1/' | head -1)

if [ "$APPROVAL_STATUS" = "skip" ]; then
    echo "ℹ️ Adaptations were skipped by user. Not applying changes."
    exit 0
fi

echo "📋 Applying approved adaptations..."
```

### Step 2: Load Approved Adaptations

```bash
APPLIED_FILE="geist/output/session-feedback/adaptations/applied.md"

# Ensure applied file exists
mkdir -p "$(dirname "$APPLIED_FILE")"

if [ ! -f "$APPLIED_FILE" ]; then
    cat > "$APPLIED_FILE" << EOF
# Applied Adaptations

EOF
fi
```

### Step 3: Apply Adaptations

Use the adaptation workflow to apply approved changes:

```bash
{{workflows/learning/apply-command-adaptations}}
```

This workflow will:
- Create timestamped backup directory
- Copy original command files to backup
- Apply approved adaptations
- Validate markdown syntax after each change
- Log all applied changes

### Step 4: Update Applied Log

Move successfully applied adaptations from `pending.md` to `applied.md`:

```bash
# Append applied adaptations to applied.md
cat >> "$APPLIED_FILE" << EOF
## $(date +%Y-%m-%d) Session

### Applied Adaptations
EOF

# For each successfully applied adaptation, log it
# (This will be done by the apply-command-adaptations workflow)

echo "✅ Applied adaptations logged to: $APPLIED_FILE"
```

### Step 5: Archive Session

After adaptations are applied:

```bash
SESSION_FILE="geist/output/session-feedback/current-session.md"
HISTORY_DIR="geist/output/session-feedback/history"
PATTERNS_DIR="geist/output/session-feedback/patterns"

# Create history directory if it doesn't exist
mkdir -p "$HISTORY_DIR"

# Move current session to history
if [ -f "$SESSION_FILE" ]; then
    TODAY=$(date +%Y-%m-%d)
    mv "$SESSION_FILE" "$HISTORY_DIR/${TODAY}-session.md"
    echo "✅ Session archived to: $HISTORY_DIR/${TODAY}-session.md"
fi

# Clear patterns (start fresh)
if [ -f "$PATTERNS_DIR/successful.md" ]; then
    cat > "$PATTERNS_DIR/successful.md" << EOF
# Successful Patterns

(Patterns will be listed here as they are detected)

EOF
fi

if [ -f "$PATTERNS_DIR/failed.md" ]; then
    cat > "$PATTERNS_DIR/failed.md" << EOF
# Failed Patterns (Anti-patterns)

(Anti-patterns will be listed here as they are detected)

EOF
fi

# Create new empty session
cat > "$SESSION_FILE" << EOF
# Session: $(date +%Y-%m-%d)

## Metadata
- Started: $(date -Iseconds)
- Last Updated: $(date -Iseconds)
- Implementations: 0
- Success Rate: 0%

## Implementations

| # | Spec | Tasks | Outcome | Duration | Notes |
|---|------|-------|---------|----------|-------|

## Patterns Observed

### Successful
(Patterns will be listed here as they are detected)

### Failed
(Failed patterns will be listed here as they are detected)

## Prompt Effectiveness

### Worked Well
(Prompts that worked well will be listed here)

### Needed Clarification
(Prompts that needed clarification will be listed here)
EOF

echo "✅ New session started"
```

### Step 6: Generate Report

```bash
BACKUP_DIR=$(ls -td geist/output/update-basepoints-and-redeploy/backups/* | head -1)
REPORT_FILE="geist/output/update-basepoints-and-redeploy/reports/adaptation-report.md"

mkdir -p "$(dirname "$REPORT_FILE")"

cat > "$REPORT_FILE" << EOF
# Adaptation Report

**Date**: $(date -Iseconds)
**Approval Status**: $APPROVAL_STATUS

## Adaptations Applied

✅ Applied: [count] adaptations
⏭️ Skipped: [count] adaptations

## Backup Location

\`$BACKUP_DIR\`

## Applied Adaptations Log

See: \`geist/output/session-feedback/adaptations/applied.md\`

## Rollback Instructions

To rollback these changes:

\`\`\`bash
cp -r $BACKUP_DIR/* geist/commands/
\`\`\`

## Session Archive

- Previous session archived to history
- New session started
EOF

echo ""
echo "✅ ADAPTATIONS APPLIED"
echo ""
echo "Applied: [N] adaptations"
echo "Backup: $BACKUP_DIR"
echo ""
echo "Session archived to history."
echo "New session started."
echo ""
echo "Report: $REPORT_FILE"
```

## Output

- Applied changes to command files in `geist/commands/`
- Backup directory: `geist/output/update-basepoints-and-redeploy/backups/[timestamp]/`
- Applied log: `geist/output/session-feedback/adaptations/applied.md`
- Adaptation report: `geist/output/update-basepoints-and-redeploy/reports/adaptation-report.md`
- Archived session: `geist/output/session-feedback/history/[date]-session.md`
- New session file: `geist/output/session-feedback/current-session.md`

## Safety Requirements

- ✅ Backup created BEFORE any changes
- ✅ Only user-approved changes applied
- ✅ All changes logged in applied.md
- ✅ Rollback procedure documented
- ✅ Markdown validation after each change

## Error Handling

If any adaptation fails:
- File is restored from backup
- Error is logged
- Remaining adaptations continue
- Final report shows failures

## Next Step

After adaptations are applied:
- Session is archived to history
- New session is started
- Process completes

## Notes

- Only runs if user approved adaptations in Phase 7
- All changes are backed up before application
- Supports rollback if issues occur
- Session is reset after adaptations are applied
