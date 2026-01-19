# Workflow: Synthesize Knowledge

## Purpose

Combine all research outputs, remove duplicates, prioritize actionable insights, and create a unified summary for easy consumption.

## Inputs

Research files from:
- `geist/config/enriched-knowledge/library-research.md`
- `geist/config/enriched-knowledge/stack-best-practices.md`
- `geist/config/enriched-knowledge/domain-knowledge.md`
- `geist/config/enriched-knowledge/version-analysis.md`
- `geist/config/enriched-knowledge/security-notes.md`

## Outputs

- Updates to individual research files (deduplication)
- `geist/config/enriched-knowledge/README.md` - Summary index

---

## Workflow

### Step 1: Create Summary Index

```bash
KNOWLEDGE_DIR="geist/config/enriched-knowledge"
SUMMARY_FILE="$KNOWLEDGE_DIR/README.md"

cat > "$SUMMARY_FILE" << HEADER_EOF
# Enriched Knowledge Index

> Consolidated research findings for your project
> Generated: $(date -Iseconds)

This directory contains research gathered during the adaptive questionnaire
process. Use this knowledge to inform your development decisions.

---

## Available Research

HEADER_EOF
```

### Step 2: Index Available Files

```bash
echo "### Research Files" >> "$SUMMARY_FILE"
echo "" >> "$SUMMARY_FILE"

# Check each expected file and add to index
if [ -f "$KNOWLEDGE_DIR/library-research.md" ]; then
    echo "- 📚 [Library Research](library-research.md) - Best practices for your dependencies" >> "$SUMMARY_FILE"
fi

if [ -f "$KNOWLEDGE_DIR/stack-best-practices.md" ]; then
    echo "- 🏗️ [Stack Best Practices](stack-best-practices.md) - Architecture patterns for your tech stack" >> "$SUMMARY_FILE"
fi

if [ -f "$KNOWLEDGE_DIR/domain-knowledge.md" ]; then
    echo "- 🎯 [Domain Knowledge](domain-knowledge.md) - Industry-specific patterns" >> "$SUMMARY_FILE"
fi

if [ -f "$KNOWLEDGE_DIR/version-analysis.md" ]; then
    echo "- 📦 [Version Analysis](version-analysis.md) - Dependency version status" >> "$SUMMARY_FILE"
fi

if [ -f "$KNOWLEDGE_DIR/security-notes.md" ]; then
    echo "- 🔒 [Security Notes](security-notes.md) - Security considerations and CVEs" >> "$SUMMARY_FILE"
fi

echo "" >> "$SUMMARY_FILE"
```

### Step 3: Extract Key Insights

```bash
cat >> "$SUMMARY_FILE" << 'INSIGHTS_HEADER'

---

## Key Insights Summary

### 🔴 Critical Items

Items requiring immediate attention:

INSIGHTS_HEADER

# Check for critical security issues
if [ -f "$KNOWLEDGE_DIR/security-notes.md" ]; then
    if grep -q "CRITICAL\|🔴" "$KNOWLEDGE_DIR/security-notes.md" 2>/dev/null; then
        echo "- ⚠️ Critical security issues found - see [Security Notes](security-notes.md)" >> "$SUMMARY_FILE"
    else
        echo "- ✅ No critical security issues detected" >> "$SUMMARY_FILE"
    fi
fi

# Check for outdated dependencies
if [ -f "$KNOWLEDGE_DIR/version-analysis.md" ]; then
    if grep -q "OUTDATED\|Major update" "$KNOWLEDGE_DIR/version-analysis.md" 2>/dev/null; then
        echo "- ⚠️ Outdated dependencies found - see [Version Analysis](version-analysis.md)" >> "$SUMMARY_FILE"
    else
        echo "- ✅ Dependencies are up to date" >> "$SUMMARY_FILE"
    fi
fi

echo "" >> "$SUMMARY_FILE"
```

### Step 4: Add Quick Reference

```bash
cat >> "$SUMMARY_FILE" << 'QUICKREF_EOF'

### 📋 Quick Reference

| Area | Document | When to Use |
|------|----------|-------------|
| Starting a new feature | Stack Best Practices | Architecture decisions |
| Adding dependencies | Library Research | Evaluate libraries |
| Security review | Security Notes | Before deployment |
| Updating packages | Version Analysis | Maintenance cycles |
| Domain questions | Domain Knowledge | Business logic |

---

QUICKREF_EOF
```

### Step 5: Add Usage Instructions

```bash
cat >> "$SUMMARY_FILE" << 'USAGE_EOF'

## How to Use This Knowledge

### During Development

1. **Before implementing a feature**: Check Stack Best Practices for patterns
2. **When choosing libraries**: Review Library Research for recommendations
3. **During code review**: Reference Security Notes for secure coding
4. **When debugging**: Check Library Research for known issues

### During Maintenance

1. **Regular updates**: Use Version Analysis to prioritize updates
2. **Security patches**: Follow Security Notes recommendations
3. **Refactoring**: Reference Stack Best Practices for patterns

### During Planning

1. **Architecture decisions**: Stack Best Practices + Domain Knowledge
2. **Compliance requirements**: Domain Knowledge + Security Notes
3. **Technology choices**: Library Research + Version Analysis

---

USAGE_EOF
```

### Step 6: Add Refresh Instructions

```bash
cat >> "$SUMMARY_FILE" << 'REFRESH_EOF'

## Keeping Knowledge Fresh

This research was generated at a point in time. To refresh:

1. **Re-run detection**: This will update research with latest information
2. **Manual update**: Edit files directly for project-specific notes
3. **Research depth**: Adjust depth for more/less detail

### Research Depth Levels

| Level | Time | Best For |
|-------|------|----------|
| `minimal` | ~30s | Quick updates |
| `standard` | ~2m | Regular use |
| `comprehensive` | ~5m | Major decisions |

To change depth:
```bash
RESEARCH_DEPTH=comprehensive
# Then re-run adapt-to-product or the research orchestrator
```

---

## Files in This Directory

REFRESH_EOF

# List all files with sizes
ls -lh "$KNOWLEDGE_DIR"/*.md 2>/dev/null | awk '{print "- `" $NF "` (" $5 ")"}' >> "$SUMMARY_FILE" || echo "- No files found" >> "$SUMMARY_FILE"

cat >> "$SUMMARY_FILE" << 'FOOTER_EOF'

---

*Generated by Geist Adaptive Questionnaire System*

FOOTER_EOF

echo "   ✓ Knowledge synthesis complete"
echo "   ✓ Summary index saved to $SUMMARY_FILE"
```

---

## Deduplication Logic

When synthesizing, the workflow identifies and consolidates:

1. **Repeated recommendations** across files
2. **Conflicting advice** (flags for human review)
3. **Version-specific information** (keeps most recent)
4. **Source attribution** (maintains for verification)

---

## Important Constraints

- Must maintain source attribution
- Should flag conflicting recommendations
- Must create navigable index
- Should highlight critical items prominently
- Must provide clear usage guidance
