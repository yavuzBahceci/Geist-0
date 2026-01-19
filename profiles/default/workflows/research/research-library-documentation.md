# Workflow: Research Library Documentation

## Purpose

Research comprehensive documentation for project libraries including internal architecture, workflows, best practices, and troubleshooting guidance. Creates library basepoints with deep technical understanding.

## Inputs

- `LIBRARY_NAME` - Name of the library to research
- `LIBRARY_CATEGORY` - Category (data, domain, util, infrastructure, framework)
- `LIBRARY_IMPORTANCE` - Importance level (critical, important, supporting)
- `PROJECT_USAGE` - How the library is used in this project
- `TECH_STACK_FILE` - Path to tech-stack.md for context

## Outputs

- Library basepoint file at `geist/basepoints/libraries/[library-name].md` (flat structure)

---

## Workflow

### Step 1: Determine Research Depth

Based on library importance, determine research depth:

```bash
echo "📚 Researching library: $LIBRARY_NAME"
echo "   Category: $LIBRARY_CATEGORY"
echo "   Importance: $LIBRARY_IMPORTANCE"

# Determine research depth based on importance
case "$LIBRARY_IMPORTANCE" in
    "critical")
        RESEARCH_DEPTH="deep"
        echo "   Research depth: DEEP (critical library)"
        ;;
    "important")
        RESEARCH_DEPTH="moderate"
        echo "   Research depth: MODERATE (important library)"
        ;;
    "supporting")
        RESEARCH_DEPTH="basic"
        echo "   Research depth: BASIC (supporting library)"
        ;;
    *)
        RESEARCH_DEPTH="moderate"
        echo "   Research depth: MODERATE (default)"
        ;;
esac
```

### Step 2: Research Official Documentation

Search for official documentation and best practices:

```bash
CURRENT_YEAR=$(date +%Y)

echo "🔍 Searching official documentation..."

# Web search queries for official documentation
# Query 1: Official documentation
# web_search("$LIBRARY_NAME official documentation")

# Query 2: Best practices from official sources
# web_search("$LIBRARY_NAME best practices official $CURRENT_YEAR")

# Query 3: Getting started guide
# web_search("$LIBRARY_NAME getting started guide official")
```

### Step 3: Research Internal Architecture (Critical/Important Only)

For critical and important libraries, research internal architecture:

```bash
if [ "$RESEARCH_DEPTH" = "deep" ] || [ "$RESEARCH_DEPTH" = "moderate" ]; then
    echo "🔍 Researching internal architecture..."
    
    # Query 4: Internal architecture
    # web_search("$LIBRARY_NAME architecture internals how it works")
    
    # Query 5: Core concepts
    # web_search("$LIBRARY_NAME core concepts design")
    
    # Query 6: Component interactions
    # web_search("$LIBRARY_NAME components modules interaction")
fi
```

### Step 4: Research Workflows and Patterns (Critical/Important Only)

For critical and important libraries, research workflows:

```bash
if [ "$RESEARCH_DEPTH" = "deep" ] || [ "$RESEARCH_DEPTH" = "moderate" ]; then
    echo "🔍 Researching workflows and patterns..."
    
    # Query 7: Common workflows
    # web_search("$LIBRARY_NAME workflow patterns")
    
    # Query 8: Execution flow
    # web_search("$LIBRARY_NAME execution flow lifecycle")
    
    # Query 9: Event handling (if applicable)
    # web_search("$LIBRARY_NAME events handlers callbacks")
fi
```

### Step 5: Research Troubleshooting (Critical Only)

For critical libraries, research troubleshooting and debugging:

```bash
if [ "$RESEARCH_DEPTH" = "deep" ]; then
    echo "🔍 Researching troubleshooting and debugging..."
    
    # Query 10: Common issues
    # web_search("$LIBRARY_NAME common issues problems")
    
    # Query 11: Debugging strategies
    # web_search("$LIBRARY_NAME debugging troubleshooting")
    
    # Query 12: Known bugs and gotchas
    # web_search("$LIBRARY_NAME gotchas pitfalls bugs")
    
    # Query 13: Error patterns
    # web_search("$LIBRARY_NAME error handling common errors")
fi
```

### Step 6: Create Library Basepoint File

Generate the library basepoint file:

```bash
# Determine output path (FLAT structure - no category subfolder)
LIBRARY_SLUG=$(echo "$LIBRARY_NAME" | tr '[:upper:]' '[:lower:]' | tr ' ' '-')
OUTPUT_DIR="geist/basepoints/libraries"
OUTPUT_FILE="$OUTPUT_DIR/$LIBRARY_SLUG.md"

# Create directory if needed
mkdir -p "$OUTPUT_DIR"

echo "📝 Creating library basepoint: $OUTPUT_FILE"

# Generate basepoint content
cat > "$OUTPUT_FILE" << 'BASEPOINT_EOF'
# Library Basepoint: $LIBRARY_NAME

## Overview

| Attribute | Value |
|-----------|-------|
| **Library** | $LIBRARY_NAME |
| **Category** | $LIBRARY_CATEGORY |
| **Importance** | $LIBRARY_IMPORTANCE |
| **Research Depth** | $RESEARCH_DEPTH |
| **Generated** | $(date) |

---

## Project Usage

### How We Use This Library

$PROJECT_USAGE

### Boundaries

#### What We Use
<!-- Document which parts/features of the library are used in this project -->

- [List features and components that are actively used]
- [Include specific modules, APIs, or patterns]

#### What We Don't Use
<!-- Document which parts are NOT used - important for understanding scope -->

- [List features and components that are NOT used]
- [Include reasons if relevant]

---

## Patterns

### Usage Patterns

<!-- Patterns for using this library in our project -->

[Document common usage patterns observed in the codebase]

### Integration Patterns

<!-- How this library integrates with other libraries -->

[Document integration patterns with other project libraries]

---

## Workflows

### Internal Workflows

<!-- How the library works internally -->

[Document internal execution flows and component interactions]

### Common Workflows

<!-- Common workflows when using this library -->

[Document typical workflows for common use cases]

---

## Best Practices

### Official Guidelines

<!-- Best practices from official documentation -->

[Document official best practices and recommendations]

### Project-Specific Practices

<!-- Best practices specific to how we use this library -->

[Document project-specific conventions and practices]

---

## Troubleshooting

### Common Issues

<!-- Common problems and their solutions -->

[Document common issues encountered and how to resolve them]

### Debugging Strategies

<!-- How to debug issues with this library -->

[Document debugging approaches and tools]

### Known Gotchas

<!-- Edge cases and pitfalls to avoid -->

[Document known gotchas and edge cases]

---

## Resources

### Official Documentation
- [Official Docs](URL)

### Community Resources
- [Community guides and tutorials]

---

*Generated by Geist Library Research Workflow*
*Research depth: $RESEARCH_DEPTH*
BASEPOINT_EOF

echo "✅ Library basepoint created: $OUTPUT_FILE"
```

### Step 7: Return Status

```bash
echo ""
echo "═══════════════════════════════════════════════════════"
echo "  LIBRARY DOCUMENTATION RESEARCH COMPLETE"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Library: $LIBRARY_NAME"
echo "  Category: $LIBRARY_CATEGORY"
echo "  Importance: $LIBRARY_IMPORTANCE"
echo "  Research Depth: $RESEARCH_DEPTH"
echo "  Output: $OUTPUT_FILE"
echo ""
echo "═══════════════════════════════════════════════════════"
```

---

## Web Search Integration

When executing this workflow, the AI agent should use the `web_search` tool to gather current information:

### Example Web Search Calls

For a critical library like "asyncio" (Python):
```
- web_search("asyncio official documentation python")
- web_search("asyncio best practices official 2026")
- web_search("asyncio architecture internals how it works")
- web_search("asyncio event loop design")
- web_search("asyncio common issues problems")
- web_search("asyncio debugging troubleshooting")
- web_search("asyncio gotchas pitfalls")
```

For an important library like "requests" (Python):
```
- web_search("requests library official documentation")
- web_search("requests best practices 2026")
- web_search("requests architecture design")
- web_search("requests common patterns")
```

For a supporting library like "python-dotenv":
```
- web_search("python-dotenv official documentation")
- web_search("python-dotenv best practices")
```

---

## Important Constraints

- Must prioritize official documentation over community resources
- Must include publication dates for time-sensitive information
- Must handle missing information gracefully
- Must document both what IS used and what is NOT used (boundaries)
- Research depth must match library importance classification
- Results should be actionable and project-specific
- Must be technology-agnostic (works for any library/language)
