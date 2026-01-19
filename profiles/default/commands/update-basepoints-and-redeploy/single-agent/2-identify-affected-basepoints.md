The SECOND STEP is to identify which basepoints are affected by the detected changes:

## Phase 2 Actions

### 2.1 Load Changed Files

Load the changed files list from Phase 1:

```bash
CACHE_DIR="geist/output/update-basepoints-and-redeploy/cache"

if [ ! -f "$CACHE_DIR/changed-files.txt" ]; then
    echo "❌ ERROR: Changed files list not found."
    echo "   Run Phase 1 (detect-changes) first."
    exit 1
fi

CHANGED_FILES=$(cat "$CACHE_DIR/changed-files.txt")
TOTAL_CHANGES=$(echo "$CHANGED_FILES" | grep -v "^$" | wc -l | tr -d ' ')

echo "📋 Loaded $TOTAL_CHANGES changed files"
```

### 2.2 Map Files to Basepoints

Apply mapping rules to determine which basepoints are affected:

**Mapping Rules:**

| Changed File Pattern | Affected Basepoint |
|---------------------|-------------------|
| `scripts/*.sh` | `basepoints/scripts/agent-base-scripts.md` |
| `profiles/default/commands/*` | `basepoints/profiles/default/commands/agent-base-commands.md` |
| `profiles/default/workflows/*` | `basepoints/profiles/default/workflows/agent-base-workflows.md` |
| `profiles/default/standards/*` | `basepoints/profiles/default/standards/agent-base-standards.md` |
| `profiles/default/agents/*` | `basepoints/profiles/default/agents/agent-base-agents.md` |
| `profiles/default/*` | `basepoints/profiles/default/agent-base-default.md` |
| `profiles/*` | `basepoints/profiles/agent-base-profiles.md` |
| `geist/product/*` | _(flag for knowledge re-extraction only)_ |
| `*` (root files) | `basepoints/agent-base-self.md` |

**Mapping Logic:**

```bash
# Initialize affected basepoints list
> "$CACHE_DIR/affected-basepoints.txt"
> "$CACHE_DIR/affected-basepoints-unique.txt"

# Process each changed file
echo "$CHANGED_FILES" | while read changed_file; do
    if [ -z "$changed_file" ]; then
        continue
    fi
    
    # Normalize path (remove leading ./)
    changed_file=$(echo "$changed_file" | sed 's|^\./||')
    
    # Determine affected basepoint based on path
    case "$changed_file" in
        scripts/*)
            echo "geist/basepoints/scripts/agent-base-scripts.md" >> "$CACHE_DIR/affected-basepoints.txt"
            ;;
        profiles/default/commands/*)
            echo "geist/basepoints/profiles/default/commands/agent-base-commands.md" >> "$CACHE_DIR/affected-basepoints.txt"
            ;;
        profiles/default/workflows/*)
            echo "geist/basepoints/profiles/default/workflows/agent-base-workflows.md" >> "$CACHE_DIR/affected-basepoints.txt"
            ;;
        profiles/default/standards/*)
            echo "geist/basepoints/profiles/default/standards/agent-base-standards.md" >> "$CACHE_DIR/affected-basepoints.txt"
            ;;
        profiles/default/agents/*)
            echo "geist/basepoints/profiles/default/agents/agent-base-agents.md" >> "$CACHE_DIR/affected-basepoints.txt"
            ;;
        profiles/default/*)
            echo "geist/basepoints/profiles/default/agent-base-default.md" >> "$CACHE_DIR/affected-basepoints.txt"
            ;;
        profiles/*)
            echo "geist/basepoints/profiles/agent-base-profiles.md" >> "$CACHE_DIR/affected-basepoints.txt"
            ;;
        geist/product/*)
            # Product files don't have basepoints, but flag for knowledge re-extraction
            echo "PRODUCT_CHANGE:$changed_file" >> "$CACHE_DIR/product-changes-detail.txt"
            ;;
        geist/*)
            # Ignore other geist files (output, specs, etc.)
            ;;
        *)
            # Root-level files affect root basepoint
            echo "geist/basepoints/agent-base-self.md" >> "$CACHE_DIR/affected-basepoints.txt"
            ;;
    esac
done

# Remove duplicates
sort -u "$CACHE_DIR/affected-basepoints.txt" > "$CACHE_DIR/affected-basepoints-unique.txt"
mv "$CACHE_DIR/affected-basepoints-unique.txt" "$CACHE_DIR/affected-basepoints.txt"
```

### 2.2.5 Flag Modules for Libraries Used Refresh

For source files that changed, flag their module basepoints for `## Libraries Used` section updates:

```bash
# Initialize modules for library refresh list
> "$CACHE_DIR/modules-for-library-refresh.txt"

# Process each changed file - flag source files for Libraries Used refresh
echo "$CHANGED_FILES" | while read changed_file; do
    if [ -z "$changed_file" ]; then
        continue
    fi
    
    # Normalize path
    changed_file=$(echo "$changed_file" | sed 's|^\./||')
    
    # Check if this is a source file that might have import changes
    case "$changed_file" in
        *.ts|*.js|*.tsx|*.jsx|*.py|*.go|*.rs|*.swift|*.dart|*.kt|*.java|*.cs|*.rb|*.php|*.vue|*.svelte)
            # This is a source file - flag its module basepoint for Libraries Used refresh
            
            # Determine the module basepoint for this file
            # Extract the module path from the file path
            FILE_DIR=$(dirname "$changed_file")
            
            # Find the corresponding basepoint
            # Map source path to basepoint path
            case "$changed_file" in
                profiles/default/commands/*)
                    BASEPOINT="geist/basepoints/profiles/default/commands/agent-base-commands.md"
                    ;;
                profiles/default/workflows/*)
                    BASEPOINT="geist/basepoints/profiles/default/workflows/agent-base-workflows.md"
                    ;;
                profiles/default/*)
                    BASEPOINT="geist/basepoints/profiles/default/agent-base-default.md"
                    ;;
                scripts/*)
                    BASEPOINT="geist/basepoints/scripts/agent-base-scripts.md"
                    ;;
                *)
                    # For other source files, try to find matching basepoint
                    BASEPOINT="geist/basepoints/agent-base-self.md"
                    ;;
            esac
            
            echo "$BASEPOINT" >> "$CACHE_DIR/modules-for-library-refresh.txt"
            ;;
    esac
done

# Remove duplicates from modules for library refresh
if [ -f "$CACHE_DIR/modules-for-library-refresh.txt" ]; then
    sort -u "$CACHE_DIR/modules-for-library-refresh.txt" > "$CACHE_DIR/modules-for-library-refresh-unique.txt"
    mv "$CACHE_DIR/modules-for-library-refresh-unique.txt" "$CACHE_DIR/modules-for-library-refresh.txt"
fi

MODULES_FOR_REFRESH=$(wc -l < "$CACHE_DIR/modules-for-library-refresh.txt" 2>/dev/null | tr -d ' ')
echo "📚 Modules flagged for Libraries Used refresh: $MODULES_FOR_REFRESH"
```

### 2.3 Calculate Parent Chain

For each affected basepoint, identify parent basepoints that also need updating:

```bash
# Parent chain propagation
# If a child basepoint changes, its parent must also be updated

> "$CACHE_DIR/parent-basepoints.txt"

while read basepoint; do
    if [ -z "$basepoint" ]; then
        continue
    fi
    
    # Extract parent path
    PARENT_DIR=$(dirname "$basepoint")
    
    # Walk up the tree to find parent basepoints
    while [ "$PARENT_DIR" != "geist/basepoints" ] && [ "$PARENT_DIR" != "." ]; do
        PARENT_NAME=$(basename "$PARENT_DIR")
        PARENT_BASEPOINT="$PARENT_DIR/agent-base-$PARENT_NAME.md"
        
        if [ -f "$PARENT_BASEPOINT" ]; then
            echo "$PARENT_BASEPOINT" >> "$CACHE_DIR/parent-basepoints.txt"
        fi
        
        PARENT_DIR=$(dirname "$PARENT_DIR")
    done
done < "$CACHE_DIR/affected-basepoints.txt"

# Add parents to affected list (unique)
cat "$CACHE_DIR/affected-basepoints.txt" "$CACHE_DIR/parent-basepoints.txt" 2>/dev/null | sort -u > "$CACHE_DIR/all-affected-basepoints.txt"
mv "$CACHE_DIR/all-affected-basepoints.txt" "$CACHE_DIR/affected-basepoints.txt"

# Always add headquarter if any basepoint is affected
if [ -s "$CACHE_DIR/affected-basepoints.txt" ]; then
    echo "geist/basepoints/headquarter.md" >> "$CACHE_DIR/affected-basepoints.txt"
fi
```

### 2.4 Check Product File Changes

Determine if product files changed (requires knowledge re-extraction):

```bash
PRODUCT_CHANGED=$(cat "$CACHE_DIR/product-files-changed.txt" 2>/dev/null || echo "false")

if [ "$PRODUCT_CHANGED" = "true" ]; then
    echo "📦 Product files changed - knowledge re-extraction required"
    echo "true" > "$CACHE_DIR/requires-knowledge-reextraction.txt"
else
    echo "false" > "$CACHE_DIR/requires-knowledge-reextraction.txt"
fi
```

### 2.5 Generate Affected Basepoints Summary

Create a summary of affected basepoints:

```bash
AFFECTED_COUNT=$(wc -l < "$CACHE_DIR/affected-basepoints.txt" | tr -d ' ')

cat > "$CACHE_DIR/affected-basepoints-summary.md" << EOF
# Affected Basepoints Summary

**Analysis Time:** $(date -u +%Y-%m-%dT%H:%M:%SZ)
**Total Changed Files:** $TOTAL_CHANGES
**Affected Basepoints:** $AFFECTED_COUNT
**Product Changes:** $PRODUCT_CHANGED

## Affected Basepoints

$(cat "$CACHE_DIR/affected-basepoints.txt" | sed 's/^/- /')

## File-to-Basepoint Mapping

| Changed File | Affected Basepoint |
|-------------|-------------------|
$(# Generate mapping table from logs)

## Update Order

Basepoints will be updated in this order (children first, parents last):

1. Module-level basepoints (deepest first)
2. Parent basepoints (bottom-up)
3. Headquarter (last)
EOF
```

## Expected Outputs

After this phase, the following files should exist:

| File | Description |
|------|-------------|
| `affected-basepoints.txt` | List of basepoints to update |
| `affected-basepoints-summary.md` | Human-readable summary |
| `requires-knowledge-reextraction.txt` | Flag for knowledge re-extraction |
| `modules-for-library-refresh.txt` | Modules needing `## Libraries Used` refresh |

## Display confirmation and next step

Once basepoint identification is complete, output the following message:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PHASE 2 COMPLETE: Identify Affected Basepoints
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 Analysis Results:
   Changed files:       [N]
   Affected basepoints: [N]
   Product changes:     [Yes/No]
   Modules for Libraries Used refresh: [N]

📋 Affected Basepoints:
   [List of affected basepoint paths]

📦 Knowledge Re-extraction: [Required/Not required]

📋 Summary: geist/output/update-basepoints-and-redeploy/cache/affected-basepoints-summary.md

NEXT STEP 👉 Run Phase 3: `3-update-basepoints.md`
```

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your basepoint identification process aligns with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
