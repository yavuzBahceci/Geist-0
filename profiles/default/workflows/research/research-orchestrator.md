# Workflow: Research Orchestrator

## Purpose

Main orchestrator for web research. Loads detected tech stack, determines research depth, calls appropriate research workflows, and aggregates results into the enriched-knowledge directory.

## Inputs

- `geist/config/project-profile.yml` - Detected project profile
- `RESEARCH_DEPTH` - minimal, standard, or comprehensive (default: standard)

## Outputs

- `geist/config/enriched-knowledge/` directory with research results

---

## Workflow

### Step 1: Load Project Profile

```bash
echo "🔬 Starting knowledge enrichment research..."
echo ""

# Create enriched-knowledge directory
mkdir -p geist/config/enriched-knowledge

# Load project profile
if [ -f "geist/config/project-profile.yml" ]; then
    echo "📂 Loading project profile..."
    
    # Extract key values (simplified parsing)
    DETECTED_LANGUAGE=$(grep "language:" geist/config/project-profile.yml | head -1 | awk '{print $2}')
    DETECTED_FRAMEWORK=$(grep "framework:" geist/config/project-profile.yml | head -1 | awk '{print $2}')
    DETECTED_DATABASE=$(grep "database:" geist/config/project-profile.yml | head -1 | awk '{print $2}')
    PROJECT_TYPE=$(grep "project_type:" geist/config/project-profile.yml | head -1 | awk '{print $2}')
    SECURITY_LEVEL=$(grep "security_level:" geist/config/project-profile.yml | head -1 | awk '{print $2}')
    
    echo "   Language: $DETECTED_LANGUAGE"
    echo "   Framework: ${DETECTED_FRAMEWORK:-(none)}"
    echo "   Database: ${DETECTED_DATABASE:-(none)}"
    echo "   Project Type: $PROJECT_TYPE"
else
    echo "⚠️ No project profile found. Run detection first."
    echo "   Using defaults..."
    DETECTED_LANGUAGE="unknown"
fi
```

### Step 2: Determine Research Depth

```bash
# Set research depth (can be overridden)
RESEARCH_DEPTH="${RESEARCH_DEPTH:-standard}"

echo ""
echo "📊 Research depth: $RESEARCH_DEPTH"
echo ""

# Define what each depth includes
case "$RESEARCH_DEPTH" in
    "minimal")
        echo "   • Latest versions"
        echo "   • Critical security issues"
        echo "   Estimated time: ~30 seconds"
        DO_LIBRARY_RESEARCH=true
        DO_SECURITY_RESEARCH=true
        DO_STACK_PATTERNS=false
        DO_DOMAIN_RESEARCH=false
        ;;
    "standard")
        echo "   • Latest versions"
        echo "   • Security issues"
        echo "   • Best practices"
        echo "   • Common pitfalls"
        echo "   Estimated time: ~2 minutes"
        DO_LIBRARY_RESEARCH=true
        DO_SECURITY_RESEARCH=true
        DO_STACK_PATTERNS=true
        DO_DOMAIN_RESEARCH=false
        ;;
    "comprehensive")
        echo "   • All standard research"
        echo "   • Architecture patterns"
        echo "   • Domain knowledge"
        echo "   • Migration guides"
        echo "   Estimated time: ~5 minutes"
        DO_LIBRARY_RESEARCH=true
        DO_SECURITY_RESEARCH=true
        DO_STACK_PATTERNS=true
        DO_DOMAIN_RESEARCH=true
        ;;
esac

echo ""
```

### Step 3: Execute Research Workflows

```bash
echo "═══════════════════════════════════════════════════════"
echo "  EXECUTING RESEARCH WORKFLOWS"
echo "═══════════════════════════════════════════════════════"
echo ""

# Track what was researched
RESEARCH_COMPLETED=""

# 1. Library Research (always in minimal+)
if [ "$DO_LIBRARY_RESEARCH" = "true" ]; then
    echo "📚 Researching libraries and frameworks..."
    {{workflows/research/research-library}}
    RESEARCH_COMPLETED="${RESEARCH_COMPLETED}library,"
fi

# 2. Security Research (always in minimal+)
if [ "$DO_SECURITY_RESEARCH" = "true" ]; then
    echo ""
    echo "🔒 Researching security vulnerabilities..."
    {{workflows/research/research-security}}
    RESEARCH_COMPLETED="${RESEARCH_COMPLETED}security,"
fi

# 3. Stack Patterns (standard+)
if [ "$DO_STACK_PATTERNS" = "true" ]; then
    echo ""
    echo "🏗️ Researching architecture patterns..."
    {{workflows/research/research-stack-patterns}}
    RESEARCH_COMPLETED="${RESEARCH_COMPLETED}patterns,"
fi

# 4. Domain Research (comprehensive only)
if [ "$DO_DOMAIN_RESEARCH" = "true" ]; then
    echo ""
    echo "🎯 Researching domain-specific knowledge..."
    {{workflows/research/research-domain}}
    RESEARCH_COMPLETED="${RESEARCH_COMPLETED}domain,"
fi

# 5. Version Analysis (always)
echo ""
echo "📦 Analyzing dependency versions..."
{{workflows/research/version-analysis}}
RESEARCH_COMPLETED="${RESEARCH_COMPLETED}versions,"
```

### Step 4: Synthesize Knowledge

```bash
echo ""
echo "🔄 Synthesizing research findings..."
{{workflows/research/synthesize-knowledge}}
```

### Step 5: Generate Research Summary

```bash
echo ""
echo "═══════════════════════════════════════════════════════"
echo "  RESEARCH COMPLETE"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "📁 Enriched knowledge saved to: geist/config/enriched-knowledge/"
echo ""
echo "Files generated:"
ls -la geist/config/enriched-knowledge/ 2>/dev/null | grep ".md" | awk '{print "   • " $NF}'
echo ""
echo "Research areas completed: $(echo $RESEARCH_COMPLETED | sed 's/,$//' | tr ',' ', ')"
echo ""
```

---

## Integration

This workflow is called by:
- `adapt-to-product/1-setup-and-information-gathering.md` (after detection)
- `create-basepoints/1-validate-prerequisites.md` (for architecture research)

The enriched knowledge is used by:
- `deploy-agents` for specialization
- Validation workflows for security checks
- Human review for flagging issues

---

## Research Depth Guidelines

| Depth | Time | Use Case |
|-------|------|----------|
| `minimal` | ~30s | Quick setup, simple projects |
| `standard` | ~2m | Most projects (default) |
| `comprehensive` | ~5m | Enterprise, complex projects |

---

## Important Constraints

- Must handle web search failures gracefully
- Should cache results to avoid redundant searches
- Must attribute sources in output
- Should prioritize actionable insights over raw data
