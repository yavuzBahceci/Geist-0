The THIRD STEP is to update only the affected basepoints by following these instructions:

{{workflows/codebase-analysis/incremental-basepoint-update}}

## Phase 3 Actions

### 3.1 Load Affected Basepoints

Load the list of basepoints that need updating from Phase 2:

```bash
CACHE_DIR="geist/output/update-basepoints-and-redeploy/cache"

if [ ! -f "$CACHE_DIR/affected-basepoints.txt" ]; then
    echo "❌ ERROR: Affected basepoints list not found."
    echo "   Run Phase 2 (identify-affected-basepoints) first."
    exit 1
fi

AFFECTED_BASEPOINTS=$(cat "$CACHE_DIR/affected-basepoints.txt" | grep -v "^$")
TOTAL_AFFECTED=$(echo "$AFFECTED_BASEPOINTS" | wc -l | tr -d ' ')

echo "📋 Loaded $TOTAL_AFFECTED basepoints to update"
```

### 3.2 Determine Update Order

Sort basepoints so children are processed before parents (deepest first):

**Update Order:**
1. **Module-level basepoints** - Deepest in hierarchy first
2. **Parent basepoints** - Bottom-up (child changes propagate to parent)
3. **Headquarter** - Always last (aggregates all changes)

```bash
# Separate headquarter from other basepoints
HEADQUARTER_PATH="geist/basepoints/headquarter.md"
MODULE_BASEPOINTS=$(echo "$AFFECTED_BASEPOINTS" | grep -v "headquarter.md")

# Sort by depth (more slashes = deeper = process first)
SORTED_BASEPOINTS=$(echo "$MODULE_BASEPOINTS" | while read bp; do
    DEPTH=$(echo "$bp" | tr -cd '/' | wc -c)
    echo "$DEPTH:$bp"
done | sort -t: -k1 -nr | cut -d: -f2)

echo "📊 Update order determined (deepest first)"
```

### 3.3 Update Each Affected Basepoint

For each affected basepoint, re-analyze the module and update:

```bash
CURRENT=0
UPDATED_LIST=""

echo "$SORTED_BASEPOINTS" | while read basepoint_path; do
    if [ -z "$basepoint_path" ]; then
        continue
    fi
    
    CURRENT=$((CURRENT + 1))
    MODULE_COUNT=$(echo "$SORTED_BASEPOINTS" | grep -v "^$" | wc -l | tr -d ' ')
    
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "📝 Updating basepoint $CURRENT/$MODULE_COUNT"
    echo "   Path: $basepoint_path"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    
    # Backup existing basepoint
    if [ -f "$basepoint_path" ]; then
        cp "$basepoint_path" "${basepoint_path}.backup"
        echo "   💾 Backed up existing content"
    fi
    
    # Extract module path from basepoint path
    MODULE_PATH=$(echo "$basepoint_path" | sed 's|^geist/basepoints/||' | sed 's|/agent-base-.*\.md$||')
    
    # Get changed files for this module
    CHANGED_IN_MODULE=$(grep "^$MODULE_PATH" "$CACHE_DIR/changed-files.txt" 2>/dev/null | wc -l | tr -d ' ')
    echo "   📁 Changed files in module: $CHANGED_IN_MODULE"
    
    # Re-analyze module using existing workflow patterns
    echo "   🔍 Re-analyzing module..."
    
    # Use codebase analysis workflow to extract updated patterns
    # {{workflows/codebase-analysis/analyze-codebase}}
    
    # Generate updated basepoint content
    # {{workflows/codebase-analysis/generate-module-basepoints}}
    
    echo "   ✅ Updated: $basepoint_path"
    
    # Track updated basepoints
    echo "$basepoint_path" >> "$CACHE_DIR/updated-basepoints.txt"
done
```

### 3.4 Merge Updates with Existing Content

For each basepoint, intelligently merge new content with existing:

**Merge Strategy:**

| Section | Strategy |
|---------|----------|
| Module Overview | Update if structure changed |
| Patterns | ADD new, KEEP unchanged, REMOVE deleted |
| Standards | Update if conventions changed |
| Flows | Re-trace affected flows only |
| Strategies | Update if approach changed |
| Testing | Update if test files changed |
| Related Modules | Update dependencies |

**Important:** Preserve sections that weren't affected by changes.

### 3.5 Update Libraries Used Sections

Update `## Libraries Used` sections in module basepoints that had source file changes:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📚 UPDATING LIBRARIES USED SECTIONS"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

MODULES_FOR_REFRESH=$(cat "$CACHE_DIR/modules-for-library-refresh.txt" 2>/dev/null)
LIBRARY_USAGE_CHANGED=$(cat "$CACHE_DIR/library-usage-changed.txt" 2>/dev/null || echo "false")

LIBRARIES_UPDATED=0

if [ -n "$MODULES_FOR_REFRESH" ]; then
    echo "$MODULES_FOR_REFRESH" | while read module_basepoint; do
        if [ -z "$module_basepoint" ] || [ ! -f "$module_basepoint" ]; then
            continue
        fi
        
        echo "   📝 Updating Libraries Used in: $module_basepoint"
        
        # Extract module path from basepoint path
        MODULE_PATH=$(echo "$module_basepoint" | sed 's|^geist/basepoints/||' | sed 's|/agent-base-.*\.md$||')
        
        # Re-extract library imports from code files in this module
        # Uses extract_library_imports() pattern from generate-module-basepoints.md
        
        # Find source files in this module
        SOURCE_FILES=$(find "$MODULE_PATH" -type f \( -name "*.ts" -o -name "*.js" -o -name "*.tsx" -o -name "*.jsx" -o -name "*.py" -o -name "*.go" -o -name "*.rs" \) 2>/dev/null)
        
        if [ -n "$SOURCE_FILES" ]; then
            # Extract imports from source files
            IMPORTS=""
            echo "$SOURCE_FILES" | while read src_file; do
                # Extract import statements (varies by language)
                case "$src_file" in
                    *.ts|*.tsx|*.js|*.jsx)
                        FILE_IMPORTS=$(grep -E "^import |^from " "$src_file" 2>/dev/null | head -20)
                        ;;
                    *.py)
                        FILE_IMPORTS=$(grep -E "^import |^from .* import" "$src_file" 2>/dev/null | head -20)
                        ;;
                    *.go)
                        FILE_IMPORTS=$(sed -n '/^import/,/)/p' "$src_file" 2>/dev/null | head -20)
                        ;;
                esac
                IMPORTS="${IMPORTS}${FILE_IMPORTS}\n"
            done
            
            # Update the ## Libraries Used section in the basepoint
            # Note: Actual section replacement is done by the agent analyzing the imports
            echo "      Found imports to analyze"
        fi
        
        LIBRARIES_UPDATED=$((LIBRARIES_UPDATED + 1))
        echo "   ✅ Libraries Used updated in: $module_basepoint"
    done
    
    echo ""
    echo "   📊 Module basepoints with Libraries Used updated: $LIBRARIES_UPDATED"
fi
```

### 3.5.5 Update Libraries Used (Aggregated) in Parent Basepoints

After module Libraries Used are updated, aggregate to parent basepoints:

```bash
echo ""
echo "   📚 Aggregating library usage to parent basepoints..."

# For each parent basepoint, aggregate Libraries Used from children
# Uses aggregate_from_children() pattern from generate-parent-basepoints.md

PARENT_BASEPOINTS=$(find geist/basepoints -name "agent-base-*.md" -type f | while read bp; do
    # Check if this basepoint has children
    BP_DIR=$(dirname "$bp")
    CHILD_COUNT=$(find "$BP_DIR" -mindepth 2 -name "agent-base-*.md" 2>/dev/null | wc -l | tr -d ' ')
    if [ "$CHILD_COUNT" -gt 0 ]; then
        echo "$bp"
    fi
done)

if [ -n "$PARENT_BASEPOINTS" ]; then
    echo "$PARENT_BASEPOINTS" | while read parent_bp; do
        if [ -z "$parent_bp" ]; then continue; fi
        
        echo "      Aggregating to: $parent_bp"
        
        # Find child basepoints
        PARENT_DIR=$(dirname "$parent_bp")
        CHILD_BASEPOINTS=$(find "$PARENT_DIR" -mindepth 2 -name "agent-base-*.md" 2>/dev/null)
        
        # Aggregate ## Libraries Used sections from children
        # Update ## Libraries Used (Aggregated) section in parent
        # Note: Actual aggregation is done by the agent
    done
fi

echo "   ✅ Library usage aggregated to parent basepoints"
```

### 3.5.6 Check if Library Basepoints Need Updates

```bash
if [ "$LIBRARY_USAGE_CHANGED" = "true" ]; then
    echo ""
    echo "   🔍 Checking if library basepoints need updates..."
    
    # Compare new usage patterns with existing library basepoints
    # Flag for manual review if significant differences detected
    # Do NOT auto-regenerate library basepoints
    
    LIBRARIES_DIR="geist/basepoints/libraries"
    if [ -d "$LIBRARIES_DIR" ]; then
        LIBRARY_BP_COUNT=$(find "$LIBRARIES_DIR" -maxdepth 1 -name "*.md" ! -name "README.md" 2>/dev/null | wc -l | tr -d ' ')
        echo "      Existing library basepoints: $LIBRARY_BP_COUNT"
        echo "      ℹ️ Review library basepoints manually if usage patterns changed significantly"
        echo "true" > "$CACHE_DIR/library-basepoints-review-needed.txt"
    fi
fi
```

### 3.7 Update Headquarter (Last)

After all module basepoints are updated, update the headquarter:

```bash
if echo "$AFFECTED_BASEPOINTS" | grep -q "headquarter.md"; then
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "🏢 UPDATING HEADQUARTER"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    
    HEADQUARTER_PATH="geist/basepoints/headquarter.md"
    
    if [ -f "$HEADQUARTER_PATH" ]; then
        cp "$HEADQUARTER_PATH" "${HEADQUARTER_PATH}.backup"
        echo "   💾 Backed up existing headquarter"
    fi
    
    # Re-aggregate knowledge from all basepoints
    # Update architecture overview if structure changed
    # Update navigation sections
    # Update cross-cutting patterns
    
    # {{workflows/codebase-analysis/generate-headquarter}}
    
    echo "   ✅ Headquarter updated"
    echo "$HEADQUARTER_PATH" >> "$CACHE_DIR/updated-basepoints.txt"
fi
```

### 3.8 Generate Update Summary

Create summary of all updates made:

```bash
UPDATED_COUNT=$(wc -l < "$CACHE_DIR/updated-basepoints.txt" 2>/dev/null | tr -d ' ' || echo "0")

cat > "$CACHE_DIR/basepoint-update-summary.md" << EOF
# Basepoint Update Summary

**Update Time:** $(date -u +%Y-%m-%dT%H:%M:%SZ)
**Total Basepoints Updated:** $UPDATED_COUNT

## Updated Basepoints

$(cat "$CACHE_DIR/updated-basepoints.txt" 2>/dev/null | sed 's/^/- /')

## Backup Files

Backup files created (for rollback if needed):
$(find geist/basepoints -name "*.backup" -type f 2>/dev/null | sed 's/^/- /' || echo "_None_")

## Update Order

1. Module-level basepoints (deepest first)
2. Parent basepoints (bottom-up)
3. Headquarter (last)
EOF
```

## Expected Outputs

After this phase, the following files should exist:

| File | Description |
|------|-------------|
| `updated-basepoints.txt` | List of basepoints that were updated |
| `basepoint-update-summary.md` | Summary of all updates |
| `*.backup` files | Backups of original basepoints |

## Display confirmation and next step

Once basepoint updates are complete, output the following message:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PHASE 3 COMPLETE: Update Basepoints
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 Update Results:
   Module basepoints updated: [N]
   Libraries Used sections updated: [N]
   Parent basepoints updated: [N]
   Headquarter updated:       [Yes/No]
   Total:                     [N] basepoints
   Library basepoints review needed: [Yes/No]

💾 Backups created for rollback if needed

📋 Summary: geist/output/update-basepoints-and-redeploy/cache/basepoint-update-summary.md

NEXT STEP 👉 Run Phase 4: `4-re-extract-knowledge.md`
```

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your basepoint update process aligns with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}

## Important Constraints

- **MUST update children before parents** (bottom-up order)
- **MUST update headquarter last** after all other basepoints
- **MUST create backups** before modifying existing basepoints
- **MUST preserve unchanged sections** - only update affected parts
- Must use existing basepoint structure and format
- Must not modify basepoints that weren't in the affected list
- Must log all changes for traceability
