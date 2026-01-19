# Phase 8: Generate Headquarter

Now that all module basepoints, parent basepoints, AND library basepoints are generated, proceed with generating the headquarter.md file.

The headquarter is the **final aggregation point** that merges:
- **Codebase knowledge** from parent basepoints
- **Library knowledge** from library basepoints

## Prerequisites

- Module basepoints should be generated (Phase 5)
- Parent basepoints should be generated (Phase 6)
- Library basepoints should be generated (Phase 7)

## Step 1: Validate Prerequisites

```bash
BASEPOINTS_DIR="geist/basepoints"
LIBRARIES_DIR="geist/basepoints/libraries"

# Check module basepoints exist
MODULE_COUNT=$(find "$BASEPOINTS_DIR" -name "agent-base-*.md" 2>/dev/null | wc -l | tr -d ' ')
if [ "$MODULE_COUNT" -eq 0 ]; then
    echo "⚠️ No module basepoints found"
    exit 1
fi

# Check library basepoints exist (optional but recommended)
LIBRARY_COUNT=$(find "$LIBRARIES_DIR" -maxdepth 1 -name "*.md" ! -name "README.md" 2>/dev/null | wc -l | tr -d ' ')
if [ "$LIBRARY_COUNT" -eq 0 ]; then
    echo "ℹ️ No library basepoints found - headquarter will only include codebase knowledge"
else
    echo "✅ Library basepoints found: $LIBRARY_COUNT"
fi

echo "✅ Prerequisites validated"
echo "   - Module basepoints: $MODULE_COUNT"
echo "   - Library basepoints: $LIBRARY_COUNT"
```

## Step 2: Generate Headquarter

{{workflows/codebase-analysis/generate-headquarter}}

## Display confirmation and next step

Once headquarter is generated, output:

```
✅ Basepoints generation complete!

**Headquarter file**: geist/basepoints/headquarter.md
**Total basepoints**: [number] files (modules + parents + libraries)
**Structure**: Complete hierarchy documented
**Abstraction layers**: [number] layers identified
**Library basepoints**: [number] libraries documented

All basepoint documentation is now complete and ready for use!

The headquarter merges:
  ✅ Codebase knowledge (from parent basepoints)
  ✅ Library knowledge (from library basepoints)

**Navigation**:
- Start here: geist/basepoints/headquarter.md
- Browse by module: geist/basepoints/
- Browse by layer: [links organized by abstraction layer]
- Browse libraries: geist/basepoints/libraries/
```

## User Standards & Preferences Compliance

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your headquarter generation aligns with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
