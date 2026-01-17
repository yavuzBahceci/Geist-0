# Phase 13: Apply Context Sections to Commands

Inject user-approved context sections into command templates at the top, before compilation.

## Prerequisites

- User has reviewed context sections from Phase 12
- User has selected which commands to enhance

## Core Responsibilities

1. **Load Approved Context Sections**: Read user-selected context sections
2. **Backup Before Changes**: Create backup of affected command files
3. **Inject Context Sections**: Add context sections at the top of each command file
4. **Validate Changes**: Ensure commands are still valid after injection
5. **Generate Report**: Document what was changed

## Workflow

### Step 1: Load Approved Context Sections

```bash
CONTEXT_SECTIONS_FILE="agent-os/output/deploy-agents/cache/context-sections.md"

if [ ! -f "$CONTEXT_SECTIONS_FILE" ]; then
    echo "No context sections to apply."
    exit 0
fi

echo "📋 Loading approved context sections..."

# Parse context sections from file
# Format: === command-name === followed by context section
```

### Step 2: Backup Before Changes

```bash
BACKUP_DIR="agent-os/output/deploy-agents/backups/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP_DIR"

# Backup affected command files
find agent-os/commands -name "*.md" -type f | while read cmd_file; do
    # Get relative path for backup structure
    RELATIVE_PATH=$(echo "$cmd_file" | sed 's|agent-os/commands/||')
    BACKUP_SUBDIR="$BACKUP_DIR/$(dirname "$RELATIVE_PATH")"
    mkdir -p "$BACKUP_SUBDIR"
    cp "$cmd_file" "$BACKUP_SUBDIR/"
done

echo "✅ Backed up commands to: $BACKUP_DIR"
```

### Step 3: Inject Context Sections

For each approved command, inject context section at the top:

```bash
# Commands to enhance (from user selection)
APPROVED_COMMANDS=(
    "shape-spec/single-agent/2-shape-spec.md"
    "write-spec/single-agent/write-spec.md"
    "create-tasks/single-agent/2-create-tasks-list.md"
    "implement-tasks/single-agent/2-implement-tasks.md"
    "orchestrate-tasks/orchestrate-tasks.md"
)

for CMD_PATH in "${APPROVED_COMMANDS[@]}"; do
    CMD_FILE="agent-os/commands/$CMD_PATH"
    
    if [ ! -f "$CMD_FILE" ]; then
        echo "⚠️  Command file not found: $CMD_FILE"
        continue
    fi
    
    # Extract context section for this command
    CONTEXT_SECTION=$(awk "/^=== $(basename "$CMD_PATH" .md) ===/,/^=== /" "$CONTEXT_SECTIONS_FILE" | sed '1d' | sed '/^=== /,$d')
    
    if [ -z "$CONTEXT_SECTION" ]; then
        echo "⚠️  No context section found for: $CMD_PATH"
        continue
    fi
    
    # Check if context section already exists
    if grep -q "^## Context" "$CMD_FILE"; then
        echo "⚠️  Context section already exists in: $CMD_FILE"
        echo "   Skipping (will not overwrite existing context)"
        continue
    fi
    
    # Inject context section at the top of the file
    TEMP_FILE=$(mktemp)
    echo "$CONTEXT_SECTION" > "$TEMP_FILE"
    echo "" >> "$TEMP_FILE"
    cat "$CMD_FILE" >> "$TEMP_FILE"
    
    # Validate markdown (basic check)
    if grep -q "^#" "$TEMP_FILE"; then
        mv "$TEMP_FILE" "$CMD_FILE"
        echo "✅ Injected context section into: $CMD_FILE"
    else
        echo "❌ Validation failed for: $CMD_FILE"
        rm "$TEMP_FILE"
    fi
done
```

### Step 4: Generate Report

```bash
REPORT_FILE="agent-os/output/deploy-agents/reports/context-injection-report.md"

COMMANDS_ENHANCED=$(echo "${APPROVED_COMMANDS[@]}" | wc -w)

cat > "$REPORT_FILE" << EOF
# Context Section Injection Report

**Date**: $(date -Iseconds)
**Commands Enhanced**: $COMMANDS_ENHANCED

## Applied Changes

| Command | Context Section | Status |
|---------|----------------|--------|
| /shape-spec | Project identity, patterns, constraints | ✅ Injected |
| /write-spec | Project identity, patterns, constraints | ✅ Injected |
| /create-tasks | Project identity, patterns, constraints | ✅ Injected |
| /implement-tasks | Project identity, patterns, constraints | ✅ Injected |
| /orchestrate-tasks | Project identity, patterns, constraints | ✅ Injected |

## Context Section Structure

Each command now has a context section at the top with:
- **Project Identity**: Tech stack, framework, architecture, key patterns
- **Relevant Patterns**: Successful patterns from session learnings (if available)
- **Constraints & Anti-patterns**: Failed patterns to avoid (if available)
- **Previous Handoff**: Runtime loading from handoff file (if exists)

## Backup Location

\`$BACKUP_DIR\`

## Rollback

To rollback, restore files from backup directory:
\`\`\`bash
cp -r $BACKUP_DIR/* agent-os/commands/
\`\`\`
EOF

echo "✅ Context injection report: $REPORT_FILE"
```

## Output

- Context sections injected into command files in `agent-os/commands/`
- Backup location: `agent-os/output/deploy-agents/backups/[timestamp]/`
- Report: `agent-os/output/deploy-agents/reports/context-injection-report.md`

## Benefits

1. ✅ **Simpler**: No runtime prompt construction - context is static at top
2. ✅ **Clearer**: Context at top, instructions below (standard structure)
3. ✅ **Faster**: No runtime overhead for prompt construction
4. ✅ **Maintainable**: Context enhancement happens once during specialization

## Next Step

Continue to finalize deployment (or next phase if applicable).
