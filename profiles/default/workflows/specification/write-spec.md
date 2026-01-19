# Spec Writing

## Core Responsibilities

1. **Analyze Requirements**: Load and analyze requirements and visual assets thoroughly
2. **Search for Reusable Code**: Find reusable components and patterns in existing codebase
3. **Create Specification**: Write comprehensive specification document

## Workflow

### Step 1: Analyze Requirements and Context

Read and understand all inputs and THINK HARD:
```bash
# Read the requirements document
cat geist/specs/[current-spec]/planning/requirements.md

# Check for visual assets
ls -la geist/specs/[current-spec]/planning/visuals/ 2>/dev/null | grep -v "^total" | grep -v "^d"
```

Parse and analyze:
- User's feature description and goals
- Requirements gathered by spec-shaper
- Visual mockups or screenshots (if present)
- Any constraints or out-of-scope items mentioned

### Step 2: Load Basepoints Knowledge and Search for Reusable Code

Before creating specifications, load basepoints knowledge and search the codebase for existing patterns and components that can be reused.

```bash
# Determine spec path
SPEC_PATH="geist/specs/[current-spec]"

# Load extracted basepoints knowledge if available
if [ -f "$SPEC_PATH/implementation/cache/basepoints-knowledge.json" ]; then
    EXTRACTED_KNOWLEDGE=$(cat "$SPEC_PATH/implementation/cache/basepoints-knowledge.json")
    SCOPE_DETECTION=$(cat "$SPEC_PATH/implementation/cache/scope-detection/semantic-analysis.json" 2>/dev/null || echo "{}")
    KEYWORD_MATCHING=$(cat "$SPEC_PATH/implementation/cache/scope-detection/keyword-matching.json" 2>/dev/null || echo "{}")
fi
```

Based on the feature requirements and basepoints knowledge, identify relevant patterns:

**From Basepoints Knowledge (if available):**
- Reusable patterns extracted from basepoints
- Common modules that match your needs
- Related modules and dependencies from basepoints
- Standards, flows, and strategies relevant to this spec
- Testing approaches and strategies from basepoints
- Historical decisions and pros/cons that inform this feature

**From Codebase Search:**
- Similar features or functionality
- Existing components or modules that match your needs
- Related logic, services, or classes with similar functionality
- Interface patterns that could be extended
- Data structures or schemas that could be reused

Use appropriate search tools and commands for the project's technology stack to find:
- Components or modules that can be reused or extended
- Patterns to follow from similar features
- Naming conventions used in the codebase (from basepoints standards)
- Architecture patterns already established (from basepoints)

Document your findings for use in the specification, prioritizing basepoints knowledge when available.

### Step 3: Check if Deep Reading is Needed

Before creating the specification, check if deep implementation reading is needed:

```bash
# Check abstraction layer distance
{{workflows/scope-detection/calculate-abstraction-layer-distance}}

# If deep reading is needed, perform deep reading
{{workflows/deep-reading/read-implementation-deep}}

# Load deep reading results if available
if [ -f "$SPEC_PATH/implementation/cache/deep-reading/implementation-analysis.json" ]; then
    DEEP_READING_RESULTS=$(cat "$SPEC_PATH/implementation/cache/deep-reading/implementation-analysis.json")
    
    # Detect reusable code
    {{workflows/deep-reading/detect-reusable-code}}
fi
```

If deep reading was performed, use the results to:
- Inform specification writing with actual implementation patterns
- Suggest reusable code and modules from actual implementation
- Reference similar logic found in implementation
- Include refactoring opportunities in spec

### Step 4: Validate SDD Spec Completeness (SDD-aligned)

Before creating the specification, validate that requirements meet SDD completeness criteria:

```bash
# SDD Spec Completeness Validation
SPEC_PATH="geist/specs/[current-spec]"
REQUIREMENTS_FILE="$SPEC_PATH/planning/requirements.md"

if [ -f "$REQUIREMENTS_FILE" ]; then
    # Check for user stories format: "As a [user], I want [action], so that [benefit]"
    HAS_USER_STORIES=$(grep -iE "as a .*i want .*so that|as a .*i want .*so that|user story" "$REQUIREMENTS_FILE" | wc -l)
    
    # Check for acceptance criteria
    HAS_ACCEPTANCE_CRITERIA=$(grep -iE "acceptance criteria|acceptance criterion" "$REQUIREMENTS_FILE" | wc -l)
    
    # Check for scope boundaries (in-scope vs out-of-scope)
    HAS_SCOPE_BOUNDARIES=$(grep -iE "in scope|out of scope|scope boundary|scope:" "$REQUIREMENTS_FILE" | wc -l)
    
    # Check for clear requirements
    HAS_REQUIREMENTS=$(grep -iE "requirement|functional requirement|requirement:" "$REQUIREMENTS_FILE" | wc -l)
    
    # SDD Validation: Check completeness
    SDD_COMPLETE="true"
    MISSING_ELEMENTS=""
    
    if [ "$HAS_USER_STORIES" -eq 0 ]; then
        SDD_COMPLETE="false"
        MISSING_ELEMENTS="${MISSING_ELEMENTS}user stories format, "
    fi
    
    if [ "$HAS_ACCEPTANCE_CRITERIA" -eq 0 ]; then
        SDD_COMPLETE="false"
        MISSING_ELEMENTS="${MISSING_ELEMENTS}acceptance criteria, "
    fi
    
    if [ "$HAS_SCOPE_BOUNDARIES" -eq 0 ]; then
        SDD_COMPLETE="false"
        MISSING_ELEMENTS="${MISSING_ELEMENTS}scope boundaries, "
    fi
    
    if [ "$HAS_REQUIREMENTS" -eq 0 ]; then
        SDD_COMPLETE="false"
        MISSING_ELEMENTS="${MISSING_ELEMENTS}clear requirements, "
    fi
    
    if [ "$SDD_COMPLETE" = "false" ]; then
        echo "⚠️ SDD Spec Completeness Check: Missing elements detected"
        echo "Missing: ${MISSING_ELEMENTS%??}"
        echo "Please ensure the specification includes:"
        echo "  - User stories in format 'As a [user], I want [action], so that [benefit]'"
        echo "  - Explicit acceptance criteria"
        echo "  - Clear scope boundaries (in-scope vs out-of-scope)"
        echo "  - Clear requirements"
        echo ""
        echo "Should we proceed anyway or enhance requirements first?"
        # In actual execution, wait for user decision
    else
        echo "✅ SDD Spec Completeness Check: All required elements present"
    fi
fi
```

### Step 5: Check for SDD Anti-Patterns (SDD-aligned)

Before creating the specification, check for SDD anti-patterns:

```bash
# SDD Anti-Pattern Detection
if [ -f "$REQUIREMENTS_FILE" ]; then
    # Check for premature technical details (violates SDD "What & Why, not How" principle)
    PREMATURE_TECH=$(grep -iE "implementation details|code structure|database schema|api endpoints|class hierarchy|architecture diagram|tech stack|framework|library|npm package|import|require" "$REQUIREMENTS_FILE" | wc -l)
    
    # Check for over-specification (too many details)
    LINE_COUNT=$(wc -l < "$REQUIREMENTS_FILE")
    OVER_SPECIFIED="false"
    if [ "$LINE_COUNT" -gt 500 ]; then
        OVER_SPECIFIED="true"
    fi
    
    # Check for specification theater (requirements that are too vague)
    VAGUE_REQUIREMENTS=$(grep -iE "should|might|could|maybe|possibly|perhaps|kind of|sort of" "$REQUIREMENTS_FILE" | wc -l)
    
    # Detect anti-patterns
    ANTI_PATTERNS_FOUND="false"
    WARNINGS=""
    
    if [ "$PREMATURE_TECH" -gt 5 ]; then
        ANTI_PATTERNS_FOUND="true"
        WARNINGS="${WARNINGS}⚠️ Premature technical details detected (SDD: focus on What & Why, not How in spec phase). "
    fi
    
    if [ "$OVER_SPECIFIED" = "true" ]; then
        ANTI_PATTERNS_FOUND="true"
        WARNINGS="${WARNINGS}⚠️ Over-specification detected (SDD: minimal, intentionally scoped specs preferred). "
    fi
    
    if [ "$VAGUE_REQUIREMENTS" -gt 10 ]; then
        ANTI_PATTERNS_FOUND="true"
        WARNINGS="${WARNINGS}⚠️ Vague requirements detected (may indicate specification theater). "
    fi
    
    if [ "$ANTI_PATTERNS_FOUND" = "true" ]; then
        echo "🔍 SDD Anti-Pattern Detection: Issues found"
        echo "$WARNINGS"
        echo "Consider reviewing requirements to align with SDD best practices."
        # In actual execution, present to user for review
    fi
fi
```

### Step 6: Check for Trade-offs and Create Checkpoints (if needed)

Before creating the specification, check if trade-offs need to be reviewed:

```bash
# Check for multiple valid patterns or conflicts
{{workflows/human-review/review-trade-offs}}
```

If trade-offs are detected, present them to the user and wait for their decision before proceeding.

Also check for big changes or abstraction layer transitions:

```bash
# Check for big changes or layer transitions
{{workflows/human-review/create-checkpoint}}
```

If a checkpoint is needed, present it to the user and wait for their confirmation before proceeding.

### Step 7: Create Core Specification (SDD-aligned)

Write the main specification to `geist/specs/[current-spec]/spec.md`.

DO NOT write actual code in the spec.md document. Just describe the requirements clearly and concisely.

**SDD Best Practices for Spec Content:**
- Focus on **What & Why**, not **How** (implementation details belong in task creation phase)
- Ensure user stories follow format: "As a [user], I want [action], so that [benefit]"
- Include explicit acceptance criteria for each requirement
- Define clear scope boundaries (in-scope vs out-of-scope)
- Keep specs minimal and intentionally scoped (avoid feature bloat)
- Avoid premature technical details (no implementation specifics, database schemas, API endpoints, etc.)

Keep it short and include only essential information for each section.

Follow this structure exactly when creating the content of `spec.md`:

```markdown
# Specification: [Feature Name]

## Goal
[1-2 sentences describing the core objective]

## User Stories
- As a [user type], I want to [action] so that [benefit]
- [repeat for up to 2 max additional user stories]

## Specific Requirements

**Specific requirement name**
- [Up to 8 CONCISE sub-bullet points to clarify specific sub-requirements, design or architectual decisions that go into this requirement, or the technical approach to take when implementing this requirement]

[repeat for up to a max of 10 specific requirements]

## Visual Design
[If mockups provided]

**`planning/visuals/[filename]`**
- [up to 8 CONCISE bullets describing specific UI elements found in this visual to address when building]

[repeat for each file in the `planning/visuals` folder]

## Existing Code to Leverage

**Code, component, or existing logic found**
- [up to 5 bullets that describe what this existing code does and how it should be re-used or replicated when building this spec]

[repeat for up to 5 existing code areas]

**Basepoints Knowledge to Leverage (if available)**
- [Reusable patterns from basepoints that should be used]
- [Common modules from basepoints that can be referenced]
- [Standards and flows from basepoints that apply to this spec]
- [Testing approaches from basepoints that should be followed]
- [Historical decisions from basepoints that inform this feature]

## Out of Scope
- [up to 10 concise descriptions of specific features that are out of scope and MUST NOT be built in this spec]
```

## Important Constraints

1. **Always search for reusable code** before specifying new components
2. **Reference visual assets** when available
3. **Do NOT write actual code** in the spec
4. **Keep each section short**, with clear, direct, skimmable specifications
5. **Do NOT deviate from the template above** and do not add additional sections

## SDD Integration Notes

This workflow has been enhanced with Spec-Driven Development (SDD) best practices:

**SDD Principles Integrated:**
- **Specification as Source of Truth**: Spec completeness validation ensures specs are actionable
- **SDD Phase Order**: Spec validation occurs before task creation (Specify → Tasks → Implement)
- **What & Why, not How**: Spec content focuses on requirements, not implementation details

**SDD Validation Methods:**
- **Spec Completeness Checks**: Validates user stories format, acceptance criteria, scope boundaries, clear requirements
- **SDD Structure Validation**: Ensures specs follow SDD best practices (What & Why, minimal scope)
- **Anti-Pattern Detection**: Detects and warns about:
  - Specification theater (vague requirements that are written but never referenced)
  - Premature comprehensiveness (over-specification, trying to spec everything upfront)
  - Over-engineering (premature technical details in spec phase)

**Technology-Agnostic Approach (Default Profile Templates Only):**
- All SDD validation checks are structure-based, not technology-specific in default templates
- No hardcoded technology-specific SDD tool references in default templates
- Validation maintains technology-agnostic state throughout **in default profile templates**
- **After Specialization:** When templates are compiled to `geist/workflows/`, workflows can and should become technology-specific based on the project's actual stack
- **Command Outputs:** Specs, tasks, and implementations should reflect the project's actual technology stack

**Workflow Steps Enhanced:**
- Step 4: Added SDD spec completeness validation
- Step 5: Added SDD anti-pattern detection
- Step 7: Enhanced spec creation guidance with SDD best practices (What & Why focus)
