# Detect Codebase Changes

## Core Responsibilities

1. **Detect Changes Using Git**: Use git diff to identify files that changed since last update
2. **Fallback to Timestamps**: If git is unavailable, use file modification timestamps
3. **Categorize Changes**: Group changes as added, modified, or deleted
4. **Filter Relevant Files**: Exclude irrelevant paths (node_modules, .git, geist/output, etc.)
5. **Generate Change Summary**: Create comprehensive summary with counts and file lists

## Workflow

### Step 1: Initialize Change Detection Environment

Set up the detection environment and determine detection method:

```bash
# Create output directories
mkdir -p geist/output/update-basepoints-and-redeploy/cache
mkdir -p geist/output/update-basepoints-and-redeploy/reports

# Initialize change detection output files
CACHE_DIR="geist/output/update-basepoints-and-redeploy/cache"
CHANGED_FILES="$CACHE_DIR/changed-files.txt"
CHANGE_SUMMARY="$CACHE_DIR/change-summary.md"

# Determine detection method
if [ -d ".git" ]; then
    DETECTION_METHOD="git"
    echo "📍 Using git-based change detection"
else
    DETECTION_METHOD="timestamp"
    echo "📍 Using timestamp-based change detection (git not available)"
fi
```

### Step 2: Detect Changes Using Git (Preferred Method)

If git is available, use git diff to detect changes:

```bash
if [ "$DETECTION_METHOD" = "git" ]; then
    # Check for previous update commit reference
    LAST_UPDATE_COMMIT_FILE="$CACHE_DIR/last-update-commit.txt"
    
    if [ -f "$LAST_UPDATE_COMMIT_FILE" ]; then
        LAST_UPDATE_COMMIT=$(cat "$LAST_UPDATE_COMMIT_FILE")
        CURRENT_COMMIT=$(git rev-parse HEAD)
        
        echo "🔍 Comparing changes from $LAST_UPDATE_COMMIT to $CURRENT_COMMIT"
        
        # Get changed files with status
        git diff --name-status "$LAST_UPDATE_COMMIT" HEAD > "$CACHE_DIR/raw-changes.txt"
        
        # Parse into categorized lists
        grep "^A" "$CACHE_DIR/raw-changes.txt" | cut -f2 > "$CACHE_DIR/added-files.txt"
        grep "^M" "$CACHE_DIR/raw-changes.txt" | cut -f2 > "$CACHE_DIR/modified-files.txt"
        grep "^D" "$CACHE_DIR/raw-changes.txt" | cut -f2 > "$CACHE_DIR/deleted-files.txt"
        grep "^R" "$CACHE_DIR/raw-changes.txt" | cut -f2,3 > "$CACHE_DIR/renamed-files.txt"
        
    else
        echo "⚠️  First run detected - no previous update commit found"
        echo "   Treating all tracked files as 'added' for initial basepoint creation"
        
        # First run - list all tracked files as "added"
        git ls-files > "$CACHE_DIR/added-files.txt"
        touch "$CACHE_DIR/modified-files.txt"
        touch "$CACHE_DIR/deleted-files.txt"
        touch "$CACHE_DIR/renamed-files.txt"
    fi
fi
```

### Step 3: Detect Changes Using Timestamps (Fallback Method)

If git is not available, use file modification timestamps:

```bash
if [ "$DETECTION_METHOD" = "timestamp" ]; then
    LAST_UPDATE_TIMESTAMP_FILE="$CACHE_DIR/last-update-timestamp.txt"
    
    if [ -f "$LAST_UPDATE_TIMESTAMP_FILE" ]; then
        LAST_UPDATE_TIMESTAMP=$(cat "$LAST_UPDATE_TIMESTAMP_FILE")
        
        echo "🔍 Finding files modified since $LAST_UPDATE_TIMESTAMP"
        
        # Find files modified since last update
        find . -type f -newer "$LAST_UPDATE_TIMESTAMP_FILE" \
            ! -path "./.git/*" \
            ! -path "./node_modules/*" \
            ! -path "./geist/output/*" \
            ! -path "./**/dist/*" \
            ! -path "./**/build/*" \
            ! -path "./**/.cache/*" \
            -print > "$CACHE_DIR/modified-files.txt"
        
        touch "$CACHE_DIR/added-files.txt"
        touch "$CACHE_DIR/deleted-files.txt"
        touch "$CACHE_DIR/renamed-files.txt"
        
        echo "⚠️  Note: Timestamp-based detection cannot distinguish added/deleted files"
        
    else
        echo "⚠️  First run detected - no previous timestamp found"
        echo "   Treating all files as 'added' for initial basepoint creation"
        
        # First run - list all files as "added"
        find . -type f \
            ! -path "./.git/*" \
            ! -path "./node_modules/*" \
            ! -path "./geist/output/*" \
            ! -path "./**/dist/*" \
            ! -path "./**/build/*" \
            ! -path "./**/.cache/*" \
            -print > "$CACHE_DIR/added-files.txt"
        
        touch "$CACHE_DIR/modified-files.txt"
        touch "$CACHE_DIR/deleted-files.txt"
        touch "$CACHE_DIR/renamed-files.txt"
    fi
fi
```

### Step 4: Filter Relevant Files

Filter out files that should not trigger basepoint updates:

```bash
# Define exclusion patterns for files that don't affect basepoints
EXCLUDE_PATTERNS=(
    "^\.git/"
    "^node_modules/"
    "^geist/output/"
    "^geist/specs/"
    "^\.env"
    "^\.DS_Store"
    "^.*\.log$"
    "^.*\.tmp$"
    "^.*\.lock$"
    "^package-lock\.json$"
    "^yarn\.lock$"
    "^pnpm-lock\.yaml$"
    "^Pipfile\.lock$"
    "^poetry\.lock$"
    "^dist/"
    "^build/"
    "^\.cache/"
    "^coverage/"
    "^\.nyc_output/"
    "^__pycache__/"
    "^\.pytest_cache/"
    "^\.venv/"
    "^venv/"
)

# Build grep exclusion pattern
EXCLUDE_REGEX=$(printf "%s\n" "${EXCLUDE_PATTERNS[@]}" | paste -sd'|' -)

# Filter each file list
for file_type in added modified deleted; do
    FILE_LIST="$CACHE_DIR/${file_type}-files.txt"
    if [ -f "$FILE_LIST" ]; then
        grep -vE "$EXCLUDE_REGEX" "$FILE_LIST" > "$CACHE_DIR/${file_type}-files-filtered.txt" 2>/dev/null || touch "$CACHE_DIR/${file_type}-files-filtered.txt"
        mv "$CACHE_DIR/${file_type}-files-filtered.txt" "$FILE_LIST"
    fi
done

# Combine all changes into single file
cat "$CACHE_DIR/added-files.txt" "$CACHE_DIR/modified-files.txt" "$CACHE_DIR/deleted-files.txt" 2>/dev/null | sort -u > "$CHANGED_FILES"
```

### Step 5: Identify Product File Changes

Specifically check if product files changed (for knowledge re-extraction):

```bash
# Check for product file changes
PRODUCT_FILES_CHANGED=false
PRODUCT_CHANGES=""

if grep -q "^geist/product/" "$CHANGED_FILES" 2>/dev/null; then
    PRODUCT_FILES_CHANGED=true
    PRODUCT_CHANGES=$(grep "^geist/product/" "$CHANGED_FILES")
    echo "📦 Product file changes detected:"
    echo "$PRODUCT_CHANGES" | sed 's/^/   /'
fi

# Save product change flag
echo "$PRODUCT_FILES_CHANGED" > "$CACHE_DIR/product-files-changed.txt"
if [ "$PRODUCT_FILES_CHANGED" = "true" ]; then
    echo "$PRODUCT_CHANGES" > "$CACHE_DIR/product-changes.txt"
fi
```

### Step 6: Generate Change Summary

Create comprehensive change summary report:

```bash
# Count changes by category
ADDED_COUNT=$(wc -l < "$CACHE_DIR/added-files.txt" 2>/dev/null | tr -d ' ' || echo "0")
MODIFIED_COUNT=$(wc -l < "$CACHE_DIR/modified-files.txt" 2>/dev/null | tr -d ' ' || echo "0")
DELETED_COUNT=$(wc -l < "$CACHE_DIR/deleted-files.txt" 2>/dev/null | tr -d ' ' || echo "0")
TOTAL_CHANGES=$((ADDED_COUNT + MODIFIED_COUNT + DELETED_COUNT))

# Generate summary report
cat > "$CHANGE_SUMMARY" << EOF
# Codebase Change Summary

**Detection Method:** $DETECTION_METHOD
**Detection Time:** $(date -u +%Y-%m-%dT%H:%M:%SZ)
**Total Changes:** $TOTAL_CHANGES files

## Summary by Category

| Category | Count |
|----------|-------|
| Added    | $ADDED_COUNT |
| Modified | $MODIFIED_COUNT |
| Deleted  | $DELETED_COUNT |
| **Total** | **$TOTAL_CHANGES** |

## Product Files Changed

$(if [ "$PRODUCT_FILES_CHANGED" = "true" ]; then
    echo "**Yes** - Knowledge re-extraction required"
    echo ""
    echo "Changed product files:"
    cat "$CACHE_DIR/product-changes.txt" 2>/dev/null | sed 's/^/- /'
else
    echo "**No** - No product file changes detected"
fi)

## Added Files ($ADDED_COUNT)

$(if [ -s "$CACHE_DIR/added-files.txt" ]; then
    cat "$CACHE_DIR/added-files.txt" | head -50 | sed 's/^/- /'
    if [ "$ADDED_COUNT" -gt 50 ]; then
        echo "- ... and $((ADDED_COUNT - 50)) more"
    fi
else
    echo "_No files added_"
fi)

## Modified Files ($MODIFIED_COUNT)

$(if [ -s "$CACHE_DIR/modified-files.txt" ]; then
    cat "$CACHE_DIR/modified-files.txt" | head -50 | sed 's/^/- /'
    if [ "$MODIFIED_COUNT" -gt 50 ]; then
        echo "- ... and $((MODIFIED_COUNT - 50)) more"
    fi
else
    echo "_No files modified_"
fi)

## Deleted Files ($DELETED_COUNT)

$(if [ -s "$CACHE_DIR/deleted-files.txt" ]; then
    cat "$CACHE_DIR/deleted-files.txt" | head -50 | sed 's/^/- /'
    if [ "$DELETED_COUNT" -gt 50 ]; then
        echo "- ... and $((DELETED_COUNT - 50)) more"
    fi
else
    echo "_No files deleted_"
fi)
EOF

echo "📋 Change summary saved to: $CHANGE_SUMMARY"
```

### Step 7: Output Detection Results

Display detection results for user:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 CHANGE DETECTION COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Detection Method: [git/timestamp]

📊 Changes Detected:
   ✅ Added:    [N] files
   ✏️  Modified: [N] files
   ❌ Deleted:  [N] files
   ─────────────────────
   📁 Total:    [N] files

📦 Product Files Changed: [Yes/No]

📋 Full report: geist/output/update-basepoints-and-redeploy/cache/change-summary.md

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Next: Identify which basepoints are affected by these changes.
```

### Step 8: Handle No Changes Scenario

If no changes detected, provide early exit option:

```bash
if [ "$TOTAL_CHANGES" -eq 0 ]; then
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "✅ NO CHANGES DETECTED"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo ""
    echo "No codebase changes found since last update."
    echo ""
    echo "Options:"
    echo "1. Exit (no update needed)"
    echo "2. Force full refresh (regenerate all basepoints)"
    echo ""
    
    # Save no-changes flag
    echo "true" > "$CACHE_DIR/no-changes-detected.txt"
fi
```

## Important Constraints

- **MUST check git availability** before attempting git-based detection
- **MUST handle first-run scenario** where no previous update exists
- **MUST filter out irrelevant files** (build artifacts, locks, caches)
- **MUST identify product file changes** separately for knowledge re-extraction
- **MUST provide clear summary** of all detected changes
- **MUST handle no-changes scenario** gracefully with early exit option
- Must save all intermediate files to cache for subsequent phases
- Must use deterministic file ordering (sort) for reproducibility
