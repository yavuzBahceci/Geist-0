# Phase 8: Generate Library Basepoints

Now that module basepoints, parent basepoints, and headquarter have been generated, proceed with creating library basepoints for the project's tech stack.

## Prerequisites

- Tech stack must be documented in `geist/product/tech-stack.md`
- Module basepoints should be generated (Phase 5)
- Parent basepoints should be generated (Phase 6)
- Headquarter should be generated (Phase 7)

## Step 1: Validate Prerequisites

```bash
TECH_STACK_FILE="geist/product/tech-stack.md"
HEADQUARTER_FILE="geist/basepoints/headquarter.md"
BASEPOINTS_DIR="geist/basepoints"

if [ ! -f "$TECH_STACK_FILE" ]; then
    echo "⚠️ Tech stack file not found at $TECH_STACK_FILE"
    echo "   Skipping library basepoints generation."
    echo "   Run /adapt-to-product first to create tech-stack.md"
    exit 0
fi

if [ ! -f "$HEADQUARTER_FILE" ]; then
    echo "⚠️ Headquarter not found at $HEADQUARTER_FILE"
    echo "   Module basepoints must be generated first (Phases 5-7)"
    exit 1
fi

echo "✅ Prerequisites validated"
echo "   - Tech stack: $TECH_STACK_FILE"
echo "   - Headquarter: $HEADQUARTER_FILE"
echo "   - Basepoints: $BASEPOINTS_DIR"
```

## Step 2: Generate Library Basepoints

Run the comprehensive library basepoints generation workflow that combines multiple knowledge sources:

```bash
echo "📚 Generating library basepoints..."

# Run the comprehensive library basepoints generation workflow
{{workflows/codebase-analysis/generate-library-basepoints}}
```

### What This Workflow Does

The `generate-library-basepoints` workflow creates comprehensive library basepoints by combining:

1. **Product Knowledge** (tech-stack.md)
   - Identifies all libraries in the project
   - Extracts versions and categories
   
2. **Project Basepoints Knowledge** (headquarter.md, agent-base-*.md)
   - How we use each library in our codebase
   - Usage patterns from module basepoints
   - Integration points with other modules

3. **Codebase Analysis**
   - Actual import/require statements
   - Implementation patterns
   - Which files use which libraries
   - Usage frequency for importance classification

4. **Official Documentation Research** (web search)
   - Best practices from official sources
   - Internal architecture and workflows
   - Troubleshooting guidance
   - Known issues and gotchas
   - Research depth based on library importance:
     - Critical: Deep research
     - Important: Moderate research
     - Supporting: Basic research

### Knowledge Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    LIBRARY BASEPOINT GENERATION FLOW                         │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  tech-stack.md  │     │  headquarter.md │     │   Codebase      │
│                 │     │  agent-base-*   │     │                 │
│ • Library list  │     │ • How we use    │     │ • Imports       │
│ • Versions      │     │   libraries     │     │ • Actual usage  │
│ • Categories    │     │ • Patterns      │     │ • File counts   │
└────────┬────────┘     └────────┬────────┘     └────────┬────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                                 ▼
                    ┌─────────────────────┐
                    │  Official Research  │
                    │  (web search)       │
                    │                     │
                    │ • Documentation     │
                    │ • Best practices    │
                    │ • Architecture      │
                    │ • Troubleshooting   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │  Library Basepoint  │
                    │                     │
                    │ • How WE use it     │
                    │ • What WE use       │
                    │ • What WE don't use │
                    │ • OUR patterns      │
                    │ • Official guidance │
                    │ • Troubleshooting   │
                    └─────────────────────┘
```

## Step 3: Verify Library Basepoints

After generation, verify the library basepoints:

```bash
LIBRARIES_PATH="geist/basepoints/libraries"

echo "🔍 Verifying library basepoints..."

# Count libraries from tech stack
TECH_STACK_LIBRARIES=$(grep -cE "^\s*-\s+[a-zA-Z]" "$TECH_STACK_FILE" 2>/dev/null || echo "0")

# Count generated basepoints
GENERATED_BASEPOINTS=$(find "$LIBRARIES_PATH" -name "*.md" -type f ! -name "README.md" 2>/dev/null | wc -l | tr -d ' ')

echo "   Libraries in tech stack: $TECH_STACK_LIBRARIES"
echo "   Basepoints generated: $GENERATED_BASEPOINTS"

# Verify each category has content
for category in data domain util infrastructure framework; do
    COUNT=$(ls -1 "$LIBRARIES_PATH/$category" 2>/dev/null | wc -l | tr -d ' ')
    echo "   - $category/: $COUNT basepoints"
done
```

## Display Confirmation and Next Step

Once ALL library basepoints are generated, output:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📚 LIBRARY BASEPOINTS GENERATION COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Libraries identified: [number]
✅ Library basepoints created: [number]
✅ Solution-specific basepoints: [number]

📁 Library basepoints location: geist/basepoints/libraries/
📋 Library index: geist/basepoints/libraries/README.md

Knowledge Sources Combined:
  ✅ Product knowledge (tech-stack.md)
  ✅ Project basepoints (headquarter.md, agent-base-*.md)
  ✅ Codebase analysis (imports, usage patterns)
  ✅ Official documentation research

Categories:
  - data/: [count] basepoints
  - domain/: [count] basepoints
  - util/: [count] basepoints
  - infrastructure/: [count] basepoints
  - framework/: [count] basepoints

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 VERIFICATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Libraries from tech stack: [N]
Library basepoints created: [N]
Status: ✅ COMPLETE (or ❌ INCOMPLETE if mismatch)

✅ Create-basepoints command complete!
   All basepoints (module, parent, headquarter, library) have been generated.
```

## User Standards & Preferences Compliance

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your library basepoint generation aligns with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
