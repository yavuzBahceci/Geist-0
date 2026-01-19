The FOURTH STEP is to re-extract and merge knowledge from the updated basepoints:

## Phase 4 Actions

### 4.1 Load Existing Knowledge Cache

Load the existing merged knowledge from the previous deploy-agents run:

```bash
CACHE_DIR="geist/output/update-basepoints-and-redeploy/cache"
DEPLOY_CACHE="geist/output/deploy-agents/cache"

# Check for existing knowledge cache
if [ -f "$DEPLOY_CACHE/merged-knowledge.md" ]; then
    echo "📂 Loading existing knowledge cache..."
    EXISTING_KNOWLEDGE=$(cat "$DEPLOY_CACHE/merged-knowledge.md")
    echo "   ✅ Existing knowledge loaded"
else
    echo "⚠️  No existing knowledge cache found"
    echo "   Will perform full knowledge extraction"
    EXISTING_KNOWLEDGE=""
fi

# Load list of updated basepoints
UPDATED_BASEPOINTS=$(cat "$CACHE_DIR/updated-basepoints.txt" 2>/dev/null)
UPDATED_COUNT=$(echo "$UPDATED_BASEPOINTS" | grep -v "^$" | wc -l | tr -d ' ')

echo "📋 Updated basepoints to extract from: $UPDATED_COUNT"
```

### 4.2 Extract Knowledge from Updated Basepoints

Re-extract knowledge from only the updated basepoints:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "🔍 EXTRACTING KNOWLEDGE FROM UPDATED BASEPOINTS"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Initialize extraction output
> "$CACHE_DIR/extracted-knowledge.md"

cat >> "$CACHE_DIR/extracted-knowledge.md" << EOF
# Extracted Knowledge from Updated Basepoints

**Extraction Time:** $(date -u +%Y-%m-%dT%H:%M:%SZ)
**Source Basepoints:** $UPDATED_COUNT

EOF

# Extract from each updated basepoint
echo "$UPDATED_BASEPOINTS" | while read basepoint_path; do
    if [ -z "$basepoint_path" ] || [ ! -f "$basepoint_path" ]; then
        continue
    fi
    
    echo "   Extracting from: $basepoint_path"
    
    CONTENT=$(cat "$basepoint_path")
    MODULE_NAME=$(basename "$basepoint_path" .md | sed 's/agent-base-//')
    
    # Extract Patterns
    PATTERNS=$(echo "$CONTENT" | grep -A 100 "^## Patterns" | grep -B 100 "^## " | head -n -1 || echo "")
    
    # Extract Standards
    STANDARDS=$(echo "$CONTENT" | grep -A 100 "^## Standards" | grep -B 100 "^## " | head -n -1 || echo "")
    
    # Extract Flows
    FLOWS=$(echo "$CONTENT" | grep -A 100 "^## Flows" | grep -B 100 "^## " | head -n -1 || echo "")
    
    # Extract Strategies
    STRATEGIES=$(echo "$CONTENT" | grep -A 100 "^## Strategies" | grep -B 100 "^## " | head -n -1 || echo "")
    
    # Extract Testing
    TESTING=$(echo "$CONTENT" | grep -A 100 "^## Testing" | grep -B 100 "^## " | head -n -1 || echo "")
    
    # Append to extraction file
    cat >> "$CACHE_DIR/extracted-knowledge.md" << EOF

## From: $MODULE_NAME

### Patterns
$PATTERNS

### Standards
$STANDARDS

### Flows
$FLOWS

### Strategies
$STRATEGIES

### Testing
$TESTING

---
EOF
done

echo "   ✅ Knowledge extracted from $UPDATED_COUNT basepoints"
```

### 4.3 Check for Product Knowledge Updates

If product files changed, re-extract product knowledge:

```bash
PRODUCT_CHANGED=$(cat "$CACHE_DIR/product-files-changed.txt" 2>/dev/null || echo "false")

if [ "$PRODUCT_CHANGED" = "true" ]; then
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "📦 EXTRACTING UPDATED PRODUCT KNOWLEDGE"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    
    # Extract from product files
    if [ -f "geist/product/mission.md" ]; then
        echo "   Extracting from: mission.md"
        MISSION=$(cat "geist/product/mission.md")
    fi
    
    if [ -f "geist/product/roadmap.md" ]; then
        echo "   Extracting from: roadmap.md"
        ROADMAP=$(cat "geist/product/roadmap.md")
    fi
    
    if [ -f "geist/product/tech-stack.md" ]; then
        echo "   Extracting from: tech-stack.md"
        TECH_STACK=$(cat "geist/product/tech-stack.md")
    fi
    
    # Append product knowledge to extraction
    cat >> "$CACHE_DIR/extracted-knowledge.md" << EOF

## Product Knowledge (Updated)

### Mission
$MISSION

### Roadmap
$ROADMAP

### Tech Stack
$TECH_STACK

---
EOF
    
    echo "   ✅ Product knowledge extracted"
else
    echo ""
    echo "📦 Product files unchanged - using existing product knowledge"
fi
```

### 4.4 Merge New Knowledge with Existing

Merge the newly extracted knowledge with the existing cache:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "🔄 MERGING KNOWLEDGE"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Initialize merged knowledge file
cat > "$CACHE_DIR/merged-knowledge.md" << EOF
# Merged Project Knowledge

**Merge Time:** $(date -u +%Y-%m-%dT%H:%M:%SZ)
**Update Type:** Incremental
**Basepoints Updated:** $UPDATED_COUNT
**Product Updated:** $PRODUCT_CHANGED

---

EOF

# Merge strategy:
# 1. Start with existing knowledge
# 2. Replace sections that correspond to updated basepoints
# 3. Add new knowledge from updated basepoints
# 4. Preserve knowledge from unchanged basepoints

# For each knowledge category, merge intelligently
KNOWLEDGE_CATEGORIES=("Patterns" "Standards" "Flows" "Strategies" "Testing")

for category in "${KNOWLEDGE_CATEGORIES[@]}"; do
    echo "   Merging: $category"
    
    # Extract category from existing knowledge
    EXISTING_SECTION=$(echo "$EXISTING_KNOWLEDGE" | grep -A 500 "^## $category" | grep -B 500 "^## " | head -n -1 || echo "")
    
    # Extract category from new extraction
    NEW_SECTION=$(cat "$CACHE_DIR/extracted-knowledge.md" | grep -A 500 "^### $category" || echo "")
    
    # Merge (new takes precedence for updated modules)
    cat >> "$CACHE_DIR/merged-knowledge.md" << EOF

## $category

### From Updated Basepoints
$NEW_SECTION

### From Unchanged Basepoints (preserved)
[Preserved from existing knowledge]

EOF
done

echo "   ✅ Knowledge merged"
```

### 4.5 Detect and Log Conflicts

Check for any conflicts between new and existing knowledge:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "🔍 CHECKING FOR CONFLICTS"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

CONFLICTS_FOUND=0

# Initialize conflicts log
cat > "$CACHE_DIR/knowledge-conflicts.md" << EOF
# Knowledge Merge Conflicts

**Check Time:** $(date -u +%Y-%m-%dT%H:%M:%SZ)

EOF

# Check for potential conflicts
# (e.g., contradicting patterns, inconsistent standards)

if [ "$CONFLICTS_FOUND" -eq 0 ]; then
    echo "   ✅ No conflicts detected"
    echo "No conflicts detected during merge." >> "$CACHE_DIR/knowledge-conflicts.md"
else
    echo "   ⚠️  $CONFLICTS_FOUND conflict(s) detected"
    echo "   See: $CACHE_DIR/knowledge-conflicts.md"
fi
```

### 4.6 Generate Knowledge Diff

Create a diff showing what changed:

```bash
cat > "$CACHE_DIR/knowledge-diff.md" << EOF
# Knowledge Changes

**Diff Generated:** $(date -u +%Y-%m-%dT%H:%M:%SZ)

## Summary

- **Basepoints updated:** $UPDATED_COUNT
- **Product knowledge updated:** $PRODUCT_CHANGED
- **Conflicts found:** $CONFLICTS_FOUND

## Changed Knowledge Categories

$(echo "$UPDATED_BASEPOINTS" | while read bp; do
    if [ -n "$bp" ]; then
        MODULE=$(basename "$bp" .md | sed 's/agent-base-//')
        echo "### $MODULE"
        echo "- Patterns: [updated/unchanged]"
        echo "- Standards: [updated/unchanged]"
        echo "- Flows: [updated/unchanged]"
        echo "- Strategies: [updated/unchanged]"
        echo "- Testing: [updated/unchanged]"
        echo ""
    fi
done)

## Impact on Commands

The following commands will need re-specialization:
- shape-spec
- write-spec
- create-tasks
- implement-tasks
- orchestrate-tasks
EOF

echo "📋 Knowledge diff saved to: $CACHE_DIR/knowledge-diff.md"
```

### 4.7 Update Deploy-Agents Cache

Copy merged knowledge to the deploy-agents cache location:

```bash
# Ensure deploy-agents cache directory exists
mkdir -p "$DEPLOY_CACHE"

# Update the merged knowledge cache
cp "$CACHE_DIR/merged-knowledge.md" "$DEPLOY_CACHE/merged-knowledge.md"

echo "💾 Updated deploy-agents knowledge cache"
```

## Expected Outputs

After this phase, the following files should exist:

| File | Description |
|------|-------------|
| `cache/extracted-knowledge.md` | Newly extracted knowledge |
| `cache/merged-knowledge.md` | Merged knowledge (new + existing) |
| `cache/knowledge-diff.md` | What changed in this update |
| `cache/knowledge-conflicts.md` | Any conflicts detected |
| `deploy-agents/cache/merged-knowledge.md` | Updated cache for commands |

## Display confirmation and next step

Once knowledge re-extraction is complete, output the following message:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PHASE 4 COMPLETE: Re-extract Knowledge
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 Extraction Results:
   Basepoints extracted:   [N]
   Product updated:        [Yes/No]
   Conflicts found:        [N]

📋 Knowledge Categories Updated:
   - Patterns
   - Standards
   - Flows
   - Strategies
   - Testing

📋 Diff: geist/output/update-basepoints-and-redeploy/cache/knowledge-diff.md

NEXT STEP 👉 Run Phase 5: `5-selective-respecialize.md`
```

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your knowledge extraction process aligns with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}

## Important Constraints

- **MUST load existing knowledge** before extraction
- **MUST extract from all updated basepoints** listed in Phase 3
- **MUST include product knowledge** if product files changed
- **MUST merge intelligently** - new knowledge updates corresponding sections
- **MUST preserve knowledge** from unchanged basepoints
- **MUST detect and log conflicts** for review
- **MUST update deploy-agents cache** for command specialization
- Must create knowledge diff for traceability
