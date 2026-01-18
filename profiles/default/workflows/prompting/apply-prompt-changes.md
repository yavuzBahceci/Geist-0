# Apply Prompt Changes

Workflow to apply approved prompt optimizations to command files.

## Purpose

Apply user-approved prompt optimizations to command templates with safety checks:
- Backup before changes
- Validate markdown syntax
- Track applied changes
- Support rollback

## Inputs

- `OPTIMIZATIONS_FILE` - Path to approved optimizations file
- `BACKUP_DIR` - Backup directory location

## Process

### Step 1: Parse Approved Optimizations

```bash
# Read optimizations file
OPTIMIZATIONS=$(cat "$OPTIMIZATIONS_FILE")

# Extract list of commands to update
COMMANDS_TO_UPDATE=$(echo "$OPTIMIZATIONS" | grep "## Optimization:" | sed 's/## Optimization: //')
```

### Step 2: Validate Each Optimization

For each optimization:

```bash
# Check if target file exists
if [ ! -f "$TARGET_FILE" ]; then
    echo "⚠️ Warning: Target file not found: $TARGET_FILE"
    SKIP_THIS=true
fi

# Check if suggested change is valid
if ! validate_markdown_syntax "$SUGGESTED_CHANGE"; then
    echo "⚠️ Warning: Invalid markdown in suggested change"
    SKIP_THIS=true
fi
```

### Step 3: Apply Each Optimization

For each approved optimization:

```bash
# Find insertion point
if grep -q "## Context" "$TARGET_FILE"; then
    INSERTION_POINT="after_context"
elif grep -q "## Step 0" "$TARGET_FILE"; then
    INSERTION_POINT="after_step0"
else
    INSERTION_POINT="at_top"
fi

# Apply change based on insertion point
case "$INSERTION_POINT" in
    "after_context")
        # Insert after "## Context" section
        insert_after_section "$TARGET_FILE" "## Context" "$OPTIMIZATION_TEXT"
        ;;
    "after_step0")
        # Insert after Step 0
        insert_after_section "$TARGET_FILE" "## Step 0" "$OPTIMIZATION_TEXT"
        ;;
    "at_top")
        # Insert at top of file
        insert_at_top "$TARGET_FILE" "$OPTIMIZATION_TEXT"
        ;;
esac

# Validate file after change
if ! validate_markdown_syntax "$TARGET_FILE"; then
    echo "❌ Error: Markdown validation failed after applying change"
    # Rollback this specific file
    restore_from_backup "$TARGET_FILE" "$BACKUP_DIR"
    SKIP_THIS=true
fi
```

### Step 4: Track Applied Changes

```bash
# Log applied changes
APPLIED_CHANGES=""

for cmd in $COMMANDS_TO_UPDATE; do
    if [ "$SKIP_THIS" != "true" ]; then
        APPLIED_CHANGES="${APPLIED_CHANGES}✅ $cmd: Applied\n"
    else
        APPLIED_CHANGES="${APPLIED_CHANGES}⏭️ $cmd: Skipped\n"
    fi
done

# Save applied changes log
echo "$APPLIED_CHANGES" > "$BACKUP_DIR/applied-changes.log"
```

## Safety Checks

1. **Backup Required**: Workflow requires backup directory to be created first
2. **Markdown Validation**: Validate syntax after each change
3. **Rollback Support**: Restore from backup if validation fails
4. **Change Tracking**: Log all applied changes for audit

## Output

- Updated command files in `agent-os/commands/`
- Applied changes log: `$BACKUP_DIR/applied-changes.log`
- Report file with summary (created by calling phase)

## Usage

Called by Phase 12 of `/deploy-agents` to apply approved prompt optimizations.

## Error Handling

If any optimization fails:
1. Log the error
2. Restore file from backup
3. Continue with remaining optimizations
4. Report failures in final report
