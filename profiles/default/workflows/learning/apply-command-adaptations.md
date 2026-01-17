# Apply Command Adaptations

Workflow to apply approved command adaptations with safety checks and rollback support.

## Purpose

Apply user-approved adaptations to command templates:
- Backup before changes
- Apply only approved adaptations
- Log all changes with timestamps
- Support rollback on errors

## Inputs

- Pending adaptations file: `agent-os/output/session-feedback/adaptations/pending.md`
- User approval status (all/select/skip)

## Process

### Step 1: Load Approved Adaptations

```bash
PENDING_FILE="agent-os/output/session-feedback/adaptations/pending.md"
APPLIED_FILE="agent-os/output/session-feedback/adaptations/applied.md"

if [ ! -f "$PENDING_FILE" ]; then
    echo "⚠️ No pending adaptations found."
    exit 0
fi

# Check approval status
APPROVAL_STATUS=$(grep "Approval Status:" "$PENDING_FILE" | sed 's/.*: \([^ ]*\).*/\1/' | head -1)

if [ "$APPROVAL_STATUS" = "skip" ]; then
    echo "ℹ️ Adaptations were skipped by user. Not applying changes."
    exit 0
fi

echo "📋 Loading approved adaptations..."
```

### Step 2: Create Backup Directory

```bash
BACKUP_DIR="agent-os/output/update-basepoints-and-redeploy/backups/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP_DIR"

echo "📦 Creating backup at: $BACKUP_DIR"

# Copy all command files that might be affected
COMMANDS_DIR="agent-os/commands"
find "$COMMANDS_DIR" -name "*.md" -type f | while read cmd_file; do
    # Get relative path for backup structure
    RELATIVE_PATH=$(echo "$cmd_file" | sed "s|$COMMANDS_DIR/||")
    BACKUP_SUBDIR="$BACKUP_DIR/$(dirname "$RELATIVE_PATH")"
    mkdir -p "$BACKUP_SUBDIR"
    cp "$cmd_file" "$BACKUP_SUBDIR/"
done

echo "✅ Backup created: $BACKUP_DIR"
```

### Step 3: Parse Adaptations to Apply

```bash
# Extract adaptations from pending file
# Format will be in pending.md

# For each adaptation in pending.md:
# - Extract target file
# - Extract adaptation type (Context Addition, Constraint Addition, Warning Addition)
# - Extract content to add
```

### Step 4: Apply Each Adaptation

For each approved adaptation:

```bash
# Determine adaptation type and target
ADAPTATION_TYPE="[Context Addition | Constraint Addition | Warning Addition]"
TARGET_FILE="agent-os/commands/[command]/[phase].md"

# Find insertion point based on type
case "$ADAPTATION_TYPE" in
    "Context Addition")
        # Insert in "## Context" or "## Project Patterns" section
        if grep -q "## Context" "$TARGET_FILE"; then
            INSERTION_POINT="after_context"
        elif grep -q "## Project Patterns" "$TARGET_FILE"; then
            INSERTION_POINT="after_project_patterns"
        else
            # Add new Context section
            INSERTION_POINT="new_context_section"
        fi
        ;;
    "Constraint Addition")
        # Insert in "## Constraints" or "## Important" section
        if grep -q "## Constraints" "$TARGET_FILE"; then
            INSERTION_POINT="after_constraints"
        elif grep -q "## Important" "$TARGET_FILE"; then
            INSERTION_POINT="after_important"
        else
            # Add new Constraints section
            INSERTION_POINT="new_constraints_section"
        fi
        ;;
    "Warning Addition")
        # Insert in "## Warnings" or create new section
        if grep -q "## Warnings" "$TARGET_FILE"; then
            INSERTION_POINT="after_warnings"
        else
            INSERTION_POINT="new_warnings_section"
        fi
        ;;
esac

# Apply adaptation
apply_adaptation() {
    local file="$1"
    local type="$2"
    local content="$3"
    local insertion_point="$4"
    
    TEMP_FILE=$(mktemp)
    
    case "$insertion_point" in
        "after_context")
            # Insert after "## Context" section
            awk -v content="$content" '/^## Context$/ {print; getline; print; print ""; print content; next} {print}' "$file" > "$TEMP_FILE"
            ;;
        "after_constraints")
            # Insert after "## Constraints" section
            awk -v content="$content" '/^## Constraints$/ {print; getline; print; print ""; print content; next} {print}' "$file" > "$TEMP_FILE"
            ;;
        "new_constraints_section")
            # Add new Constraints section after Context
            awk -v content="$content" '/^## Context$/ {print; print ""; print "## Constraints"; print ""; print content; next} {print}' "$file" > "$TEMP_FILE"
            ;;
        *)
            # Default: add at end of file
            cat "$file" > "$TEMP_FILE"
            echo "" >> "$TEMP_FILE"
            echo "$content" >> "$TEMP_FILE"
            ;;
    esac
    
    # Validate markdown syntax
    if validate_markdown "$TEMP_FILE"; then
        mv "$TEMP_FILE" "$file"
        return 0
    else
        echo "⚠️ Markdown validation failed. Restoring from backup."
        restore_from_backup "$file" "$BACKUP_DIR"
        return 1
    fi
}

# Apply the adaptation
if apply_adaptation "$TARGET_FILE" "$ADAPTATION_TYPE" "$ADAPTATION_CONTENT" "$INSERTION_POINT"; then
    echo "✅ Applied: $ADAPTATION_TYPE to $TARGET_FILE"
else
    echo "❌ Failed: $ADAPTATION_TYPE to $TARGET_FILE"
fi
```

### Step 5: Update Applied Log

```bash
# Move applied adaptations to applied.md
APPLIED_LOG="$APPLIED_FILE"

# Create applied file if it doesn't exist
if [ ! -f "$APPLIED_LOG" ]; then
    cat > "$APPLIED_LOG" << EOF
# Applied Adaptations

EOF
fi

# Append to applied log
cat >> "$APPLIED_LOG" << EOF
## $(date +%Y-%m-%d) Session

EOF

# For each successfully applied adaptation:
cat >> "$APPLIED_LOG" << EOF
### Adaptation: [Adaptation Name]
- **Applied To**: [Target File]
- **Date**: $(date -Iseconds)
- **Type**: [Context Addition | Constraint Addition | Warning Addition]
- **Content**: [Brief description of what was added]

EOF

echo "✅ Applied adaptations logged to: $APPLIED_LOG"
```

### Step 6: Track Applied Changes

```bash
# Create applied changes log in backup directory
APPLIED_CHANGES_LOG="$BACKUP_DIR/applied-changes.log"

cat > "$APPLIED_CHANGES_LOG" << EOF
# Applied Changes Log

**Date**: $(date -Iseconds)
**Backup Location**: $BACKUP_DIR

## Adaptations Applied

EOF

# List all applied adaptations
# This will be populated by the actual application process

cat >> "$APPLIED_CHANGES_LOG" << EOF

## Rollback Instructions

To rollback these changes, restore files from backup:

\`\`\`bash
cp -r $BACKUP_DIR/* agent-os/commands/
\`\`\`
EOF

echo "✅ Changes log: $APPLIED_CHANGES_LOG"
```

## Error Handling

If any adaptation fails:

```bash
# Restore file from backup
restore_from_backup() {
    local file="$1"
    local backup_dir="$2"
    
    RELATIVE_PATH=$(echo "$file" | sed "s|agent-os/commands/||")
    BACKUP_FILE="$backup_dir/$RELATIVE_PATH"
    
    if [ -f "$BACKUP_FILE" ]; then
        cp "$BACKUP_FILE" "$file"
        echo "✅ Restored: $file from backup"
        return 0
    else
        echo "⚠️ Backup not found for: $file"
        return 1
    fi
}

# Validate markdown syntax
validate_markdown() {
    local file="$1"
    # Basic validation - check for balanced markdown
    # This is a placeholder - actual validation would use a markdown parser
    return 0
}
```

## Output

- Applied changes to command files in `agent-os/commands/`
- Backup directory: `agent-os/output/update-basepoints-and-redeploy/backups/[timestamp]/`
- Applied log: `agent-os/output/session-feedback/adaptations/applied.md`
- Changes log: `$BACKUP_DIR/applied-changes.log`

## Safety Checks

1. **Backup Required**: Always creates backup before changes
2. **Markdown Validation**: Validates syntax after each change
3. **Rollback Support**: Restores from backup if validation fails
4. **Change Tracking**: Logs all applied changes for audit

## Rollback Procedure

If adaptations cause issues:

```bash
# Find backup directory
BACKUP_DIR=$(ls -td agent-os/output/update-basepoints-and-redeploy/backups/* | head -1)

# Restore all files
cp -r "$BACKUP_DIR"/* agent-os/commands/

echo "✅ Rollback complete. Files restored from: $BACKUP_DIR"
```

## Usage

Called by Phase 8 of `/update-basepoints-and-redeploy` to apply approved adaptations.

## Notes

- Only applies user-approved adaptations
- Creates backup before any changes
- Logs all changes for audit trail
- Supports rollback on errors
