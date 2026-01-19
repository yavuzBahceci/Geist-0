# Headquarter File Generation

## Core Responsibilities

1. **Load Product Files**: Read product files (mission.md, roadmap.md, tech-stack.md)
2. **Load Detected Layers**: Retrieve detected abstraction layers from phase 2
3. **Load Library Basepoints**: Aggregate library knowledge from library basepoints
4. **Generate Headquarter**: Create headquarter.md at root of basepoints folder
5. **Bridge Abstractions**: Connect product-level abstraction with software project-level abstraction
6. **Document Architecture**: Document overall architecture, abstraction layers, module relationships, and library usage

## Workflow

### Step 1: Load Product Files

Load the required product files:

```bash
if [ ! -f "geist/product/mission.md" ] || [ ! -f "geist/product/roadmap.md" ] || [ ! -f "geist/product/tech-stack.md" ]; then
    echo "❌ Product files not found. Cannot generate headquarter."
    exit 1
fi

MISSION_CONTENT=$(cat geist/product/mission.md)
ROADMAP_CONTENT=$(cat geist/product/roadmap.md)
TECH_STACK_CONTENT=$(cat geist/product/tech-stack.md)
```

### Step 2: Load Detected Layers

Load the detected abstraction layers from cache.

### Step 3: Analyze Basepoint Structure

Analyze the generated basepoint structure to understand module relationships:

```bash
TOP_LEVEL_BASEPOINTS=$(find geist/basepoints -mindepth 1 -maxdepth 1 -name "agent-base-*.md" | sort)
MODULE_COUNT=$(find geist/basepoints -name "agent-base-*.md" | wc -l)
```

### Step 3.5: Load Library Basepoints Knowledge

Aggregate library knowledge from library basepoints:

```bash
LIBRARIES_DIR="geist/basepoints/libraries"
LIBRARY_COUNT=$(find "$LIBRARIES_DIR" -maxdepth 1 -name "*.md" ! -name "README.md" 2>/dev/null | wc -l | tr -d ' ')

# Extract library summaries for headquarter
LIBRARY_SUMMARIES=""
if [ "$LIBRARY_COUNT" -gt 0 ]; then
    for lib_file in "$LIBRARIES_DIR"/*.md; do
        if [ "$(basename "$lib_file")" != "README.md" ] && [ -f "$lib_file" ]; then
            LIB_NAME=$(basename "$lib_file" .md)
            # Extract Overview section (first paragraph after ## Overview)
            LIB_OVERVIEW=$(sed -n '/^## Overview/,/^## /p' "$lib_file" | head -n 10 | tail -n +2)
            LIBRARY_SUMMARIES="${LIBRARY_SUMMARIES}\n### $LIB_NAME\n$LIB_OVERVIEW\n"
        fi
    done
fi
```

### Step 4: Generate Headquarter Content

Create the headquarter.md file with comprehensive content including:
- Product-Level Abstraction (Mission, Roadmap, Tech Stack)
- Software Project-Level Abstraction (Architecture Overview, Detected Layers, Module Structure, Module Relationships)
- Abstraction Bridge (Product → Software Project Mapping, Technology Decisions, Feature → Module Mapping)
- Architecture Patterns
- Standards and Conventions
- **Library Knowledge** (aggregated from library basepoints)
- Development Workflow
- Testing Strategy
- Key Insights
- Navigation (By Abstraction Layer, By Module, By Library)
- References

### Library Knowledge Section Template

Include this section in the headquarter:

```markdown
## Library Knowledge

This section aggregates knowledge from library basepoints.

**Total Libraries Documented:** [LIBRARY_COUNT]

[LIBRARY_SUMMARIES]

For detailed library documentation, see: `geist/basepoints/libraries/`
```

### Step 5: Populate Headquarter Content

Fill in the headquarter file with actual extracted content from product files and basepoint analysis.

### Step 6: Verify Headquarter File

Verify the headquarter file was generated correctly and contains all required sections.

## Important Constraints

- Must use product files (mission.md, roadmap.md, tech-stack.md) as source
- Must bridge product-level abstraction with software project-level abstraction
- Must include detected abstraction layers
- Must document overall architecture and module relationships
- **Must include Library Knowledge section** aggregated from library basepoints
- Must provide navigation to all basepoint files (including library basepoints)
- Must be placed at root of basepoints folder (`geist/basepoints/headquarter.md`)
- Headquarter is the **final aggregation point** merging codebase + library knowledge
