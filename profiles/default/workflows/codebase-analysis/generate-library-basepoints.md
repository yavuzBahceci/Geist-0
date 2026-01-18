# Workflow: Generate Library Basepoints

## Purpose

Create comprehensive library basepoints by combining:
1. **Project basepoints knowledge** - How we use the libraries in our codebase
2. **Product knowledge** - Tech stack, detected libraries
3. **Official documentation research** - Best practices, architecture, troubleshooting
4. **Our implementation patterns** - Actual usage patterns from codebase analysis

This workflow is called by `create-basepoints` Phase 8 after module basepoints are generated.

## Prerequisites

- Module basepoints must be generated (Phase 5-7 complete)
- `agent-os/basepoints/headquarter.md` must exist
- `agent-os/product/tech-stack.md` must exist

## Inputs

- `TECH_STACK_FILE` - Path to tech-stack.md
- `HEADQUARTER_FILE` - Path to headquarter.md
- `BASEPOINTS_DIR` - Path to basepoints directory

## Outputs

- Library basepoints in `agent-os/basepoints/libraries/[category]/[library].md`
- Library index at `agent-os/basepoints/libraries/README.md`

---

## Workflow

### Step 1: Load Product Knowledge

Load the tech stack to identify libraries:

```bash
echo "📖 Loading product knowledge..."

TECH_STACK_FILE="agent-os/product/tech-stack.md"
HEADQUARTER_FILE="agent-os/basepoints/headquarter.md"
BASEPOINTS_DIR="agent-os/basepoints"
LIBRARIES_DIR="$BASEPOINTS_DIR/libraries"

# Load tech stack
if [ -f "$TECH_STACK_FILE" ]; then
    TECH_STACK_CONTENT=$(cat "$TECH_STACK_FILE")
    echo "✅ Loaded tech stack"
else
    echo "❌ Tech stack not found at $TECH_STACK_FILE"
    exit 1
fi

# Load headquarter for project context
if [ -f "$HEADQUARTER_FILE" ]; then
    HEADQUARTER_CONTENT=$(cat "$HEADQUARTER_FILE")
    echo "✅ Loaded headquarter"
else
    echo "⚠️ Headquarter not found, continuing without project overview"
    HEADQUARTER_CONTENT=""
fi
```

### Step 2: Extract Libraries from Tech Stack

Parse the tech stack to identify all libraries:

```bash
echo "🔍 Extracting libraries from tech stack..."

# The AI agent should analyze the tech stack content and extract:
# - Primary language and runtime
# - Framework(s)
# - Database libraries
# - Utility libraries
# - Infrastructure libraries
# - Testing libraries

# Create a list of libraries with metadata
LIBRARIES_LIST=""

# Example extraction (AI agent implements based on tech stack format):
# For each library found in tech-stack.md:
# - Library name
# - Version (if specified)
# - Category (data, domain, util, infrastructure, framework)
# - Importance (critical, important, supporting)

echo "   Libraries identified from tech stack"
```

### Step 3: Extract Project Usage Patterns from Basepoints

Analyze existing module basepoints to understand how libraries are used:

```bash
echo "🔍 Extracting library usage patterns from basepoints..."

# For each library identified:
for library in $LIBRARIES_LIST; do
    LIBRARY_NAME=$(echo "$library" | cut -d'|' -f1)
    
    echo "   Analyzing usage of: $LIBRARY_NAME"
    
    # Search all module basepoints for mentions of this library
    USAGE_PATTERNS=""
    
    # Find all basepoint files
    find "$BASEPOINTS_DIR" -name "agent-base-*.md" -type f | while read basepoint_file; do
        # Check if this basepoint mentions the library
        if grep -qi "$LIBRARY_NAME" "$basepoint_file" 2>/dev/null; then
            # Extract relevant sections
            PATTERNS=$(grep -A 5 -i "$LIBRARY_NAME" "$basepoint_file" | head -20)
            USAGE_PATTERNS="${USAGE_PATTERNS}

### From $(basename "$basepoint_file")
$PATTERNS
"
        fi
    done
    
    # Store usage patterns for this library
    echo "$USAGE_PATTERNS" > "/tmp/library-usage-$LIBRARY_NAME.txt"
done

echo "✅ Library usage patterns extracted from basepoints"
```

### Step 4: Analyze Codebase for Implementation Details

Deep-dive into the codebase to understand actual library usage:

```bash
echo "🔍 Analyzing codebase for library implementation details..."

for library in $LIBRARIES_LIST; do
    LIBRARY_NAME=$(echo "$library" | cut -d'|' -f1)
    
    echo "   Analyzing codebase usage of: $LIBRARY_NAME"
    
    # Search codebase for import/require statements
    IMPORT_PATTERNS=$(grep -rn "import.*$LIBRARY_NAME\|require.*$LIBRARY_NAME\|from $LIBRARY_NAME\|use $LIBRARY_NAME" . \
        --include="*.ts" --include="*.js" --include="*.py" --include="*.go" --include="*.rs" \
        2>/dev/null | head -50)
    
    # Identify which files use this library
    FILES_USING=$(echo "$IMPORT_PATTERNS" | cut -d':' -f1 | sort -u)
    
    # Extract usage patterns from those files
    IMPLEMENTATION_PATTERNS=""
    for file in $FILES_USING; do
        if [ -f "$file" ]; then
            # Extract function calls and patterns
            PATTERNS=$(grep -n "$LIBRARY_NAME" "$file" | head -20)
            IMPLEMENTATION_PATTERNS="${IMPLEMENTATION_PATTERNS}

### $file
$PATTERNS
"
        fi
    done
    
    # Store implementation patterns
    echo "$IMPLEMENTATION_PATTERNS" > "/tmp/library-impl-$LIBRARY_NAME.txt"
done

echo "✅ Codebase implementation patterns analyzed"
```

### Step 5: Research Official Documentation

For each library, research official documentation:

```bash
echo "🔍 Researching official documentation..."

for library in $LIBRARIES_LIST; do
    LIBRARY_NAME=$(echo "$library" | cut -d'|' -f1)
    LIBRARY_IMPORTANCE=$(echo "$library" | cut -d'|' -f4)
    
    echo "   Researching: $LIBRARY_NAME (importance: $LIBRARY_IMPORTANCE)"
    
    # Determine research depth based on importance
    case "$LIBRARY_IMPORTANCE" in
        "critical")
            # Deep research for critical libraries
            # web_search("$LIBRARY_NAME official documentation")
            # web_search("$LIBRARY_NAME architecture internals")
            # web_search("$LIBRARY_NAME best practices official")
            # web_search("$LIBRARY_NAME troubleshooting debugging")
            # web_search("$LIBRARY_NAME common issues gotchas")
            RESEARCH_DEPTH="deep"
            ;;
        "important")
            # Moderate research for important libraries
            # web_search("$LIBRARY_NAME official documentation")
            # web_search("$LIBRARY_NAME best practices")
            # web_search("$LIBRARY_NAME common issues")
            RESEARCH_DEPTH="moderate"
            ;;
        "supporting")
            # Basic research for supporting libraries
            # web_search("$LIBRARY_NAME documentation")
            RESEARCH_DEPTH="basic"
            ;;
    esac
    
    # Store research results
    # OFFICIAL_DOCS, BEST_PRACTICES, ARCHITECTURE, TROUBLESHOOTING
done

echo "✅ Official documentation researched"
```

### Step 5.5: Extract Security Information from Enriched Knowledge

If security notes exist from the initial `adapt-to-product` research, extract relevant information:

```bash
echo "🔒 Extracting security information from enriched knowledge..."

AGENT_OS_PATH="agent-os"

for library in $LIBRARIES_LIST; do
    LIBRARY_NAME=$(echo "$library" | cut -d'|' -f1)
    
    SECURITY_NOTES=""
    if [ -f "$AGENT_OS_PATH/config/enriched-knowledge/security-notes.md" ]; then
        echo "   Checking security notes for: $LIBRARY_NAME"
        # Search for library-specific security notes (CVEs, vulnerabilities, security considerations)
        SECURITY_NOTES=$(grep -A 20 -i "$LIBRARY_NAME" "$AGENT_OS_PATH/config/enriched-knowledge/security-notes.md" 2>/dev/null | head -25)
        
        if [ -n "$SECURITY_NOTES" ]; then
            echo "   ⚠️ Security notes found for $LIBRARY_NAME"
            echo "$SECURITY_NOTES" > "/tmp/library-security-$LIBRARY_NAME.txt"
        else
            echo "   ✅ No security issues found for $LIBRARY_NAME"
            echo "No known security issues documented." > "/tmp/library-security-$LIBRARY_NAME.txt"
        fi
    else
        echo "   ℹ️ No enriched-knowledge/security-notes.md found"
        echo "Security notes not available. Run /adapt-to-product to generate." > "/tmp/library-security-$LIBRARY_NAME.txt"
    fi
done

echo "✅ Security information extracted"
```

### Step 5.6: Extract Version Information from Enriched Knowledge

If version analysis exists from the initial research, extract relevant information:

```bash
echo "📦 Extracting version information from enriched knowledge..."

for library in $LIBRARIES_LIST; do
    LIBRARY_NAME=$(echo "$library" | cut -d'|' -f1)
    
    VERSION_STATUS=""
    if [ -f "$AGENT_OS_PATH/config/enriched-knowledge/version-analysis.md" ]; then
        echo "   Checking version status for: $LIBRARY_NAME"
        # Search for library-specific version info (current version, latest, upgrade recommendations)
        VERSION_STATUS=$(grep -A 10 -i "$LIBRARY_NAME" "$AGENT_OS_PATH/config/enriched-knowledge/version-analysis.md" 2>/dev/null | head -15)
        
        if [ -n "$VERSION_STATUS" ]; then
            echo "   📋 Version status found for $LIBRARY_NAME"
            echo "$VERSION_STATUS" > "/tmp/library-version-$LIBRARY_NAME.txt"
        else
            echo "   ℹ️ No version analysis for $LIBRARY_NAME"
            echo "Version analysis not available for this library." > "/tmp/library-version-$LIBRARY_NAME.txt"
        fi
    else
        echo "   ℹ️ No enriched-knowledge/version-analysis.md found"
        echo "Version analysis not available. Run /adapt-to-product to generate." > "/tmp/library-version-$LIBRARY_NAME.txt"
    fi
done

echo "✅ Version information extracted"
```

### Step 6: Classify Library Importance

Classify each library by importance based on usage analysis:

```bash
echo "📊 Classifying library importance..."

classify_importance() {
    local library_name="$1"
    local usage_count="$2"
    local is_framework="$3"
    local is_core_infrastructure="$4"
    
    # Critical: Framework, core infrastructure, or used in 10+ files
    if [ "$is_framework" = "true" ] || [ "$is_core_infrastructure" = "true" ] || [ "$usage_count" -ge 10 ]; then
        echo "critical"
    # Important: Used in 3-9 files
    elif [ "$usage_count" -ge 3 ]; then
        echo "important"
    # Supporting: Used in 1-2 files
    else
        echo "supporting"
    fi
}

# Classify each library
for library in $LIBRARIES_LIST; do
    LIBRARY_NAME=$(echo "$library" | cut -d'|' -f1)
    
    # Count files using this library
    USAGE_COUNT=$(grep -rl "$LIBRARY_NAME" . --include="*.ts" --include="*.js" --include="*.py" 2>/dev/null | wc -l | tr -d ' ')
    
    # Check if it's a framework or core infrastructure
    IS_FRAMEWORK=$(echo "$TECH_STACK_CONTENT" | grep -i "framework.*$LIBRARY_NAME" && echo "true" || echo "false")
    IS_CORE=$(echo "$TECH_STACK_CONTENT" | grep -i "core.*$LIBRARY_NAME\|infrastructure.*$LIBRARY_NAME" && echo "true" || echo "false")
    
    IMPORTANCE=$(classify_importance "$LIBRARY_NAME" "$USAGE_COUNT" "$IS_FRAMEWORK" "$IS_CORE")
    
    echo "   $LIBRARY_NAME: $IMPORTANCE (used in $USAGE_COUNT files)"
done
```

### Step 7: Categorize Libraries

Categorize each library by its domain:

```bash
echo "📁 Categorizing libraries..."

categorize_library() {
    local library_name="$1"
    local library_description="$2"
    
    # Data: databases, ORM, data access
    if echo "$library_description" | grep -iE "database|sql|orm|mongo|redis|postgres|mysql|data access|persistence" > /dev/null; then
        echo "data"
    # Infrastructure: networking, HTTP, threading, system
    elif echo "$library_description" | grep -iE "http|network|async|thread|socket|server|client|request|api" > /dev/null; then
        echo "infrastructure"
    # Framework: web frameworks, UI frameworks
    elif echo "$library_description" | grep -iE "framework|react|vue|angular|express|fastapi|django|rails" > /dev/null; then
        echo "framework"
    # Domain: business logic, domain-specific
    elif echo "$library_description" | grep -iE "business|domain|model|entity|service" > /dev/null; then
        echo "domain"
    # Util: everything else
    else
        echo "util"
    fi
}

# Create category directories
mkdir -p "$LIBRARIES_DIR/data"
mkdir -p "$LIBRARIES_DIR/domain"
mkdir -p "$LIBRARIES_DIR/util"
mkdir -p "$LIBRARIES_DIR/infrastructure"
mkdir -p "$LIBRARIES_DIR/framework"

echo "✅ Library categories created"
```

### Step 8: Generate Library Basepoint Files

For each library, generate a comprehensive basepoint file:

```bash
echo "📝 Generating library basepoint files..."

for library in $LIBRARIES_LIST; do
    LIBRARY_NAME=$(echo "$library" | cut -d'|' -f1)
    LIBRARY_VERSION=$(echo "$library" | cut -d'|' -f2)
    LIBRARY_CATEGORY=$(echo "$library" | cut -d'|' -f3)
    LIBRARY_IMPORTANCE=$(echo "$library" | cut -d'|' -f4)
    
    # Load gathered knowledge
    USAGE_PATTERNS=$(cat "/tmp/library-usage-$LIBRARY_NAME.txt" 2>/dev/null)
    IMPL_PATTERNS=$(cat "/tmp/library-impl-$LIBRARY_NAME.txt" 2>/dev/null)
    SECURITY_NOTES=$(cat "/tmp/library-security-$LIBRARY_NAME.txt" 2>/dev/null)
    VERSION_STATUS=$(cat "/tmp/library-version-$LIBRARY_NAME.txt" 2>/dev/null)
    
    # Determine output path
    LIBRARY_SLUG=$(echo "$LIBRARY_NAME" | tr '[:upper:]' '[:lower:]' | tr ' ' '-')
    OUTPUT_FILE="$LIBRARIES_DIR/$LIBRARY_CATEGORY/$LIBRARY_SLUG.md"
    
    echo "   Creating: $OUTPUT_FILE"
    
    # Generate basepoint content
    cat > "$OUTPUT_FILE" << BASEPOINT_EOF
# Library Basepoint: $LIBRARY_NAME

## Overview

| Attribute | Value |
|-----------|-------|
| **Library** | $LIBRARY_NAME |
| **Version** | $LIBRARY_VERSION |
| **Category** | $LIBRARY_CATEGORY |
| **Importance** | $LIBRARY_IMPORTANCE |
| **Generated** | $(date) |

---

## Project Usage

### How We Use This Library

[Summary of how this library is used in our project, based on basepoints and codebase analysis]

### Usage Patterns from Basepoints

$USAGE_PATTERNS

### Implementation Patterns from Codebase

$IMPL_PATTERNS

---

## Boundaries

### What We Use

[List the specific features, APIs, and patterns from this library that we actively use]

- [Feature 1]
- [Feature 2]
- [API/Pattern 3]

### What We Don't Use

[List features and APIs that are available but NOT used in our project - important for understanding scope]

- [Unused Feature 1]
- [Unused Feature 2]

---

## Patterns

### Usage Patterns

[Document common usage patterns observed in our codebase]

### Integration Patterns

[How this library integrates with other libraries in our project]

---

## Workflows

### Internal Workflows

[Document internal execution flows and component interactions - from official documentation research]

### Common Workflows in Our Project

[Document typical workflows when using this library in our specific context]

---

## Best Practices

### Official Guidelines

[Best practices from official documentation]

### Project-Specific Practices

[Best practices specific to how we use this library in our project]

---

## Security Notes

[Security vulnerabilities, CVEs, and security considerations for this library]

$SECURITY_NOTES

---

## Version Status

[Current version, latest available, upgrade recommendations]

$VERSION_STATUS

---

## Troubleshooting

### Common Issues

[Common problems and their solutions - from official docs and our experience]

### Debugging Strategies

[How to debug issues with this library]

### Known Gotchas

[Edge cases and pitfalls to avoid]

---

## Resources

### Official Documentation
- [Official Docs](URL)

### Internal References
- [List relevant module basepoints that use this library]

---

*Generated by Geist Library Basepoints Workflow*
*Importance: $LIBRARY_IMPORTANCE | Category: $LIBRARY_CATEGORY*
BASEPOINT_EOF

done

echo "✅ Library basepoint files generated"
```

### Step 9: Detect Solution-Specific Patterns

Detect when different solutions from the same library are used:

```bash
echo "🔍 Detecting solution-specific patterns..."

# For libraries with multiple usage patterns, create separate basepoints
# Examples:
# - asyncio: event loop vs task groups
# - SQLAlchemy: ORM vs Core
# - React: class components vs hooks

for library in $LIBRARIES_LIST; do
    LIBRARY_NAME=$(echo "$library" | cut -d'|' -f1)
    LIBRARY_CATEGORY=$(echo "$library" | cut -d'|' -f3)
    
    # Analyze usage patterns for distinct solutions
    IMPL_PATTERNS=$(cat "/tmp/library-impl-$LIBRARY_NAME.txt" 2>/dev/null)
    
    # Check for distinct solution patterns
    # [AI agent analyzes patterns and identifies distinct solutions]
    
    # If distinct solutions found, create solution-specific basepoints
    # Example: $LIBRARIES_DIR/$LIBRARY_CATEGORY/$LIBRARY_NAME-solution1.md
done

echo "✅ Solution-specific patterns detected"
```

### Step 10: Generate Library Index

Create an index of all library basepoints:

```bash
echo "📋 Generating library index..."

cat > "$LIBRARIES_DIR/README.md" << INDEX_EOF
# Library Basepoints Index

## Overview

This folder contains basepoints for libraries used in the project, organized by category.
Each basepoint combines:
- Project basepoints knowledge (how we use the library)
- Official documentation research (best practices, architecture)
- Codebase analysis (actual implementation patterns)

## Categories

### Data (\`data/\`)
Libraries for data access, databases, ORM, and persistence.

### Domain (\`domain/\`)
Libraries for domain logic, business rules, and core models.

### Util (\`util/\`)
Utility libraries, helpers, and common functions.

### Infrastructure (\`infrastructure/\`)
Libraries for networking, HTTP, threading, and system-level operations.

### Framework (\`framework/\`)
Framework components and UI libraries.

## Library Index

| Library | Category | Importance | Files Using | Basepoint |
|---------|----------|------------|-------------|-----------|
INDEX_EOF

# Add each library to the index
for library in $LIBRARIES_LIST; do
    LIBRARY_NAME=$(echo "$library" | cut -d'|' -f1)
    LIBRARY_CATEGORY=$(echo "$library" | cut -d'|' -f3)
    LIBRARY_IMPORTANCE=$(echo "$library" | cut -d'|' -f4)
    LIBRARY_SLUG=$(echo "$LIBRARY_NAME" | tr '[:upper:]' '[:lower:]' | tr ' ' '-')
    
    USAGE_COUNT=$(grep -rl "$LIBRARY_NAME" . --include="*.ts" --include="*.js" --include="*.py" 2>/dev/null | wc -l | tr -d ' ')
    
    echo "| $LIBRARY_NAME | $LIBRARY_CATEGORY | $LIBRARY_IMPORTANCE | $USAGE_COUNT | [$LIBRARY_SLUG](./$LIBRARY_CATEGORY/$LIBRARY_SLUG.md) |" >> "$LIBRARIES_DIR/README.md"
done

cat >> "$LIBRARIES_DIR/README.md" << FOOTER_EOF

## Usage

These library basepoints are used by:
- \`extract-library-basepoints-knowledge\` workflow for knowledge extraction
- Spec/implementation commands for context enrichment
- \`fix-bug\` command for library-specific troubleshooting

## Regeneration

To regenerate library basepoints after adding new libraries:
1. Update \`agent-os/product/tech-stack.md\`
2. Run \`/update-basepoints-and-redeploy\`

---

*Generated: $(date)*
*By: Geist Library Basepoints Workflow*
FOOTER_EOF

echo "✅ Library index generated"
```

### Step 11: Return Status

```bash
echo ""
echo "═══════════════════════════════════════════════════════"
echo "  LIBRARY BASEPOINTS GENERATION COMPLETE"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Libraries Processed: $(echo "$LIBRARIES_LIST" | wc -l | tr -d ' ')"
echo "  Critical: $(echo "$LIBRARIES_LIST" | grep -c "critical")"
echo "  Important: $(echo "$LIBRARIES_LIST" | grep -c "important")"
echo "  Supporting: $(echo "$LIBRARIES_LIST" | grep -c "supporting")"
echo ""
echo "  Output Location: $LIBRARIES_DIR"
echo "  Index: $LIBRARIES_DIR/README.md"
echo ""
echo "  Categories:"
echo "    - data/: $(ls -1 "$LIBRARIES_DIR/data" 2>/dev/null | wc -l | tr -d ' ') basepoints"
echo "    - domain/: $(ls -1 "$LIBRARIES_DIR/domain" 2>/dev/null | wc -l | tr -d ' ') basepoints"
echo "    - util/: $(ls -1 "$LIBRARIES_DIR/util" 2>/dev/null | wc -l | tr -d ' ') basepoints"
echo "    - infrastructure/: $(ls -1 "$LIBRARIES_DIR/infrastructure" 2>/dev/null | wc -l | tr -d ' ') basepoints"
echo "    - framework/: $(ls -1 "$LIBRARIES_DIR/framework" 2>/dev/null | wc -l | tr -d ' ') basepoints"
echo ""
echo "═══════════════════════════════════════════════════════"
```

---

## Knowledge Sources Combined

This workflow combines knowledge from multiple sources:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    LIBRARY BASEPOINT KNOWLEDGE SOURCES                       │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  PRODUCT DOCS   │     │   BASEPOINTS    │     │    CODEBASE     │
│                 │     │                 │     │                 │
│ tech-stack.md   │     │ headquarter.md  │     │ Source files    │
│ • Libraries     │     │ • Project       │     │ • Imports       │
│ • Versions      │     │   overview      │     │ • Usage         │
│ • Categories    │     │                 │     │   patterns      │
│                 │     │ agent-base-*.md │     │ • Integration   │
│                 │     │ • Module usage  │     │   points        │
│                 │     │ • Patterns      │     │                 │
└────────┬────────┘     └────────┬────────┘     └────────┬────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                                 ▼
                    ┌─────────────────────┐
                    │  OFFICIAL RESEARCH  │
                    │                     │
                    │ • Documentation     │
                    │ • Best practices    │
                    │ • Architecture      │
                    │ • Troubleshooting   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │  LIBRARY BASEPOINT  │
                    │                     │
                    │ • How WE use it     │
                    │ • What WE use       │
                    │ • What WE don't use │
                    │ • OUR patterns      │
                    │ • Official guidance │
                    │ • Troubleshooting   │
                    └─────────────────────┘
```

## Important Constraints

- Must run AFTER module basepoints are generated (Phase 5-7)
- Must combine project-specific knowledge with official documentation
- Must document boundaries (what is/isn't used)
- Must classify importance based on actual usage
- Must categorize by domain (data, domain, util, infrastructure, framework)
- Must be technology-agnostic
- Research depth must match library importance
