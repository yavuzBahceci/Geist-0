# Incremental Basepoint Update

## Core Responsibilities

1. **Load Existing Basepoint Content**: Read current basepoint files that need updating
2. **Re-analyze Specific Modules**: Analyze only modules affected by changes (not entire codebase)
3. **Merge Updates with Existing Content**: Intelligently merge new patterns/standards with existing
4. **Handle Parent Chain Updates**: Propagate changes up the basepoint hierarchy (bottom-up)
5. **Preserve Unchanged Sections**: Keep sections that weren't affected by changes

## Workflow

### Step 1: Load Affected Basepoints List

Load the list of basepoints that need updating from Phase 2:

```bash
CACHE_DIR="geist/output/update-basepoints-and-redeploy/cache"
AFFECTED_BASEPOINTS_FILE="$CACHE_DIR/affected-basepoints.txt"

if [ ! -f "$AFFECTED_BASEPOINTS_FILE" ]; then
    echo "❌ ERROR: Affected basepoints list not found."
    echo "   Expected: $AFFECTED_BASEPOINTS_FILE"
    echo "   Run Phase 2 (identify-affected-basepoints) first."
    exit 1
fi

# Load and categorize basepoints
AFFECTED_BASEPOINTS=$(cat "$AFFECTED_BASEPOINTS_FILE" | grep -v "^#" | grep -v "^$")
TOTAL_BASEPOINTS=$(echo "$AFFECTED_BASEPOINTS" | wc -l | tr -d ' ')

echo "📋 Loaded $TOTAL_BASEPOINTS basepoints to update"

# Separate module-level and parent-level basepoints for ordered processing
echo "$AFFECTED_BASEPOINTS" | grep -v "headquarter" | grep "agent-base-" > "$CACHE_DIR/module-basepoints-to-update.txt" || true
echo "$AFFECTED_BASEPOINTS" | grep "headquarter" > "$CACHE_DIR/headquarter-to-update.txt" || true

MODULE_COUNT=$(wc -l < "$CACHE_DIR/module-basepoints-to-update.txt" 2>/dev/null | tr -d ' ' || echo "0")
HEADQUARTER_UPDATE=$([ -s "$CACHE_DIR/headquarter-to-update.txt" ] && echo "Yes" || echo "No")

echo "   Module basepoints: $MODULE_COUNT"
echo "   Headquarter update: $HEADQUARTER_UPDATE"
```

### Step 2: Sort Basepoints by Depth (Children First)

Sort basepoints so children are processed before parents:

```bash
# Calculate depth for each basepoint (deeper = more slashes in path)
while read basepoint_path; do
    if [ -z "$basepoint_path" ]; then
        continue
    fi
    
    DEPTH=$(echo "$basepoint_path" | tr -cd '/' | wc -c)
    echo "$DEPTH:$basepoint_path"
done < "$CACHE_DIR/module-basepoints-to-update.txt" | sort -t: -k1 -nr | cut -d: -f2 > "$CACHE_DIR/sorted-basepoints-to-update.txt"

echo "📊 Sorted basepoints by depth (deepest first for bottom-up processing)"
```

### Step 3: Update Each Module Basepoint

Process each module basepoint, preserving unchanged content:

```bash
CURRENT=0
UPDATED=()
FAILED=()

while read basepoint_path; do
    if [ -z "$basepoint_path" ]; then
        continue
    fi
    
    CURRENT=$((CURRENT + 1))
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "📝 Updating basepoint $CURRENT/$MODULE_COUNT: $basepoint_path"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    
    # Extract module path from basepoint path
    # basepoints/profiles/default/agent-base-default.md -> profiles/default
    MODULE_PATH=$(echo "$basepoint_path" | sed 's|^geist/basepoints/||' | sed 's|/agent-base-.*\.md$||')
    
    if [ -z "$MODULE_PATH" ] || [ "$MODULE_PATH" = "agent-base-"* ]; then
        # Root-level basepoint
        MODULE_PATH="."
    fi
    
    echo "   Module path: $MODULE_PATH"
    
    # Check if basepoint file exists
    if [ -f "$basepoint_path" ]; then
        echo "   📂 Loading existing basepoint content..."
        EXISTING_CONTENT=$(cat "$basepoint_path")
        
        # Backup existing content
        cp "$basepoint_path" "${basepoint_path}.backup"
    else
        echo "   ⚠️  Basepoint file not found - will create new"
        EXISTING_CONTENT=""
    fi
    
    # Get list of changed files for this module
    CHANGED_FILES_FOR_MODULE=$(grep "^$MODULE_PATH" "$CACHE_DIR/changed-files.txt" 2>/dev/null || true)
    CHANGED_COUNT=$(echo "$CHANGED_FILES_FOR_MODULE" | grep -v "^$" | wc -l | tr -d ' ')
    
    echo "   📋 Changed files in module: $CHANGED_COUNT"
    
    # [Re-analyze module and generate updated content]
    # This step should:
    # 1. Read the changed files
    # 2. Extract new patterns, standards, flows, strategies
    # 3. Merge with existing content (preserve unchanged sections)
    # 4. Write updated basepoint
    
    echo "   ✅ Updated: $basepoint_path"
    
    # Log progress
    echo "- [x] \`$basepoint_path\` (module: $MODULE_PATH, changes: $CHANGED_COUNT files)" >> "$CACHE_DIR/update-progress.md"
    
done < "$CACHE_DIR/sorted-basepoints-to-update.txt"
```

### Step 4: Merge New Content with Existing

For each section of a basepoint, intelligently merge updates:

```markdown
# Merge Strategy per Section

## Module Overview
- Update if module structure changed (files added/removed)
- Preserve purpose and layer if unchanged

## Patterns Section
### Design Patterns
- ADD new patterns found in changed files
- KEEP existing patterns if still present in code
- REMOVE patterns no longer present (if files deleted)

### Coding Patterns
- Same merge strategy as design patterns

## Standards Section
- Update naming conventions if new patterns found
- Preserve existing standards unless contradicted

## Flows Section
### Data Flow
- Re-trace data flow if interface files changed
- Merge new flow paths with existing

### Control Flow
- Re-trace control flow if logic files changed
- Update entry/exit points if changed

## Strategies Section
- Update if implementation approach changed
- Preserve architectural strategies unless contradicted

## Testing Section
- Update if test files changed
- Add new test patterns found
- Keep existing test documentation

## Related Modules Section
- Update dependencies if imports changed
- Update relationships if module structure changed
```

### Step 5: Update Parent Basepoints (Bottom-Up)

After updating module basepoints, propagate changes to parents:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📤 PROPAGATING CHANGES TO PARENT BASEPOINTS"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Find parent basepoints that need updating
# A parent needs updating if any of its children were updated
find geist/basepoints -name "agent-base-*.md" -type f | while read parent_basepoint; do
    PARENT_DIR=$(dirname "$parent_basepoint")
    
    # Check if any child basepoints were updated
    CHILD_UPDATED=false
    while read updated_basepoint; do
        UPDATED_DIR=$(dirname "$updated_basepoint")
        
        # Check if updated_basepoint is a child of parent_basepoint
        if [[ "$UPDATED_DIR" == "$PARENT_DIR/"* ]]; then
            CHILD_UPDATED=true
            break
        fi
    done < "$CACHE_DIR/sorted-basepoints-to-update.txt"
    
    if [ "$CHILD_UPDATED" = "true" ]; then
        echo "   📝 Updating parent: $parent_basepoint"
        
        # Re-aggregate knowledge from child basepoints
        # Update "Related Modules" section
        # Update aggregated patterns/standards/flows
        
        echo "   ✅ Parent updated: $parent_basepoint"
    fi
done
```

### Step 6: Update Headquarter (Last)

If any basepoints changed, update the headquarter file:

```bash
if [ -s "$CACHE_DIR/headquarter-to-update.txt" ]; then
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "🏢 UPDATING HEADQUARTER"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    
    HEADQUARTER_PATH="geist/basepoints/headquarter.md"
    
    if [ -f "$HEADQUARTER_PATH" ]; then
        echo "   📂 Loading existing headquarter content..."
        cp "$HEADQUARTER_PATH" "${HEADQUARTER_PATH}.backup"
        
        # Re-aggregate high-level knowledge from all basepoints
        # Update architecture overview if structure changed
        # Update navigation sections
        # Update cross-cutting patterns
        
        echo "   ✅ Headquarter updated"
    else
        echo "   ⚠️  Headquarter not found - will create new"
        # Generate new headquarter using generate-headquarter workflow
    fi
fi
```

### Step 7: Generate Update Log

Create detailed log of all updates made:

```bash
# Generate update log
cat > "$CACHE_DIR/update-log.md" << EOF
# Incremental Basepoint Update Log

**Update Time:** $(date -u +%Y-%m-%dT%H:%M:%SZ)

## Summary

- **Module basepoints updated:** $MODULE_COUNT
- **Parent basepoints updated:** [N]
- **Headquarter updated:** $HEADQUARTER_UPDATE

## Updated Basepoints

$(cat "$CACHE_DIR/update-progress.md" 2>/dev/null || echo "_No updates logged_")

## Backup Files

Backup files created with \`.backup\` extension:
$(find geist/basepoints -name "*.backup" -type f 2>/dev/null | sed 's/^/- /' || echo "_No backups created_")

## Merge Decisions

[Log of merge decisions made during update]

EOF

echo "📋 Update log saved to: $CACHE_DIR/update-log.md"
```

### Step 8: Output Update Results

Display update results for user:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ INCREMENTAL BASEPOINT UPDATE COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 Update Summary:
   Module basepoints: [N] updated
   Parent basepoints: [N] updated
   Headquarter: [Yes/No]

📋 Update log: geist/output/update-basepoints-and-redeploy/cache/update-log.md
💾 Backups: *.backup files created for rollback if needed

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Next: Re-extract and merge knowledge from updated basepoints.
```

## Important Constraints

- **MUST process children before parents** (bottom-up order)
- **MUST update headquarter last** after all other basepoints
- **MUST preserve unchanged content** - only update affected sections
- **MUST create backups** before modifying existing basepoints
- **MUST log all changes** for traceability and potential rollback
- Must use existing basepoint structure and format
- Must maintain consistency with unaffected basepoints
- Must handle missing basepoints gracefully (create new if needed)
- Must not modify basepoints that weren't in the affected list
