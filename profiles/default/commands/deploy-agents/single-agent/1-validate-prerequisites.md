The FIRST STEP is to validate prerequisites and load project knowledge for specialization.

## Step 0: Load Project Profile and Enriched Knowledge (NEW)

Before validating prerequisites, load all gathered knowledge for use in specialization:

```bash
echo "═══════════════════════════════════════════════════════"
echo "  LOADING PROJECT KNOWLEDGE"
echo "═══════════════════════════════════════════════════════"
echo ""

# Initialize knowledge variables
PROJECT_PROFILE_LOADED=false
ENRICHED_KNOWLEDGE_LOADED=false
USER_PREFERENCES_SET=false
```

### 0.1: Load Project Profile

```bash
if [ -f "geist/config/project-profile.yml" ]; then
    echo "✅ Loading project profile..."
    
    # Extract key values for specialization hints
    DETECTED_LANGUAGE=$(grep "language:" geist/config/project-profile.yml | head -1 | awk '{print $2}')
    DETECTED_FRAMEWORK=$(grep "framework:" geist/config/project-profile.yml | head -1 | awk '{print $2}')
    PROJECT_TYPE=$(grep "project_type:" geist/config/project-profile.yml | head -1 | awk '{print $2}')
    SECURITY_LEVEL=$(grep "security_level:" geist/config/project-profile.yml | head -1 | awk '{print $2}')
    COMPLEXITY=$(grep "complexity:" geist/config/project-profile.yml | head -1 | awk '{print $2}')
    
    # Extract commands
    BUILD_CMD=$(grep "build:" geist/config/project-profile.yml | head -1 | cut -d'"' -f2)
    TEST_CMD=$(grep "test:" geist/config/project-profile.yml | head -1 | cut -d'"' -f2)
    LINT_CMD=$(grep "lint:" geist/config/project-profile.yml | head -1 | cut -d'"' -f2)
    
    # Extract user preferences
    COMPLIANCE=$(grep -A5 "user_specified:" geist/config/project-profile.yml | grep "compliance:" | head -1)
    HUMAN_REVIEW=$(grep "human_review_level:" geist/config/project-profile.yml | head -1 | awk '{print $2}')
    
    echo "   Language: $DETECTED_LANGUAGE"
    echo "   Framework: ${DETECTED_FRAMEWORK:-(none)}"
    echo "   Security Level: $SECURITY_LEVEL"
    echo "   Complexity: $COMPLEXITY"
    echo ""
    echo "   Build: ${BUILD_CMD:-(not set)}"
    echo "   Test: ${TEST_CMD:-(not set)}"
    echo "   Lint: ${LINT_CMD:-(not set)}"
    echo ""
    
    PROJECT_PROFILE_LOADED=true
    
    # Check if user preferences are set
    if [ -n "$HUMAN_REVIEW" ] && [ "$HUMAN_REVIEW" != "" ]; then
        USER_PREFERENCES_SET=true
        echo "   Human Review: $HUMAN_REVIEW"
    fi
else
    echo "⚠️  No project profile found"
    echo "   Running detection to gather project information..."
    echo ""
    
    {{workflows/detection/detect-project-profile}}
    
    PROJECT_PROFILE_LOADED=true
fi
echo ""
```

### 0.2: Load Enriched Knowledge

```bash
if [ -d "geist/config/enriched-knowledge" ]; then
    echo "✅ Loading enriched knowledge..."
    
    # List available knowledge files
    for knowledge_file in geist/config/enriched-knowledge/*.md; do
        if [ -f "$knowledge_file" ]; then
            filename=$(basename "$knowledge_file")
            echo "   • $filename"
        fi
    done
    
    ENRICHED_KNOWLEDGE_LOADED=true
    echo ""
    
    # Check for critical security issues
    if [ -f "geist/config/enriched-knowledge/security-notes.md" ]; then
        if grep -q "CRITICAL\|🔴" geist/config/enriched-knowledge/security-notes.md 2>/dev/null; then
            echo "⚠️  CRITICAL security issues found - see security-notes.md"
            echo ""
        fi
    fi
    
    # Check for outdated dependencies
    if [ -f "geist/config/enriched-knowledge/version-analysis.md" ]; then
        if grep -q "OUTDATED" geist/config/enriched-knowledge/version-analysis.md 2>/dev/null; then
            echo "ℹ️  Outdated dependencies detected - see version-analysis.md"
            echo ""
        fi
    fi
else
    echo "ℹ️  No enriched knowledge found"
    echo "   Consider running /adapt-to-product first for better specialization"
    echo ""
fi
```

### 0.3: Ask Missing User Preferences

Only ask if preferences were not set in a prior command:

```bash
if [ "$USER_PREFERENCES_SET" != "true" ]; then
    echo "═══════════════════════════════════════════════════════"
    echo "  QUESTIONS REQUIRING YOUR INPUT"
    echo "═══════════════════════════════════════════════════════"
    echo ""
    echo "These preferences were not set in a prior command:"
    echo ""
    
    {{workflows/detection/question-templates}}
    
    echo ""
fi
```

### 0.4: Specialization Hints

Use loaded knowledge to inform specialization:

```bash
echo "═══════════════════════════════════════════════════════"
echo "  SPECIALIZATION HINTS"
echo "═══════════════════════════════════════════════════════"
echo ""

# Determine workflow complexity based on profile
if [ "$COMPLEXITY" = "complex" ] || [ "$SECURITY_LEVEL" = "high" ]; then
    echo "ℹ️  Recommending comprehensive workflows due to:"
    [ "$COMPLEXITY" = "complex" ] && echo "   • Project complexity: $COMPLEXITY"
    [ "$SECURITY_LEVEL" = "high" ] && echo "   • Security level: $SECURITY_LEVEL"
    RECOMMENDED_WORKFLOW_COMPLEXITY="comprehensive"
else
    echo "ℹ️  Recommending standard workflows"
    RECOMMENDED_WORKFLOW_COMPLEXITY="standard"
fi

# Note validation commands to use
if [ -n "$BUILD_CMD" ] || [ -n "$TEST_CMD" ] || [ -n "$LINT_CMD" ]; then
    echo ""
    echo "ℹ️  Validation commands will be configured:"
    [ -n "$BUILD_CMD" ] && echo "   • Build: $BUILD_CMD"
    [ -n "$TEST_CMD" ] && echo "   • Test: $TEST_CMD"
    [ -n "$LINT_CMD" ] && echo "   • Lint: $LINT_CMD"
fi

echo ""
```

---

## Step 1: Validate Prerequisites

The core validation step checks for required basepoints and product files:

## Core Responsibilities

1. **Check Basepoints Folder and Files**: Verify that basepoints folder exists with required files
2. **Check Product Files Existence**: Verify that required product files exist
3. **Handle Missing Prerequisites**: Prompt user to create missing prerequisites if validation fails
4. **Stop Process Gracefully**: Stop process if prerequisites are missing with clear instructions

## Workflow

### Step 1: Validate Basepoints Folder and Files

Check if the basepoints folder and required files exist:

```bash
# Check if basepoints folder exists
if [ ! -d "geist/basepoints" ]; then
    echo "❌ Basepoints folder not found: geist/basepoints/"
    MISSING_BASEPOINTS=true
else
    echo "✅ Basepoints folder found: geist/basepoints/"
    
    # Check for headquarter.md
    if [ ! -f "geist/basepoints/headquarter.md" ]; then
        echo "❌ Missing: geist/basepoints/headquarter.md"
        MISSING_BASEPOINTS=true
    else
        echo "✅ Found: geist/basepoints/headquarter.md"
    fi
    
    # Check for at least one module-specific basepoint file
    BASEPOINT_COUNT=$(find geist/basepoints -name "agent-base-*.md" -type f | wc -l | tr -d ' ')
    if [ "$BASEPOINT_COUNT" -eq 0 ]; then
        echo "❌ No module-specific basepoint files found (looking for agent-base-*.md)"
        MISSING_BASEPOINTS=true
    else
        echo "✅ Found $BASEPOINT_COUNT module-specific basepoint file(s)"
    fi
    
    # Check if basepoint files contain extractable content
    if [ -f "geist/basepoints/headquarter.md" ]; then
        if ! grep -q -E "(Pattern|Standard|Flow|Strategy|Testing|Test)" geist/basepoints/headquarter.md 2>/dev/null; then
            echo "⚠️  Warning: headquarter.md may not contain extractable patterns, standards, flows, or strategies"
        fi
    fi
fi
```

### Step 2: Validate Product Folder and Files

Check if the product folder and required files exist:

```bash
# Check if product folder exists
if [ ! -d "geist/product" ]; then
    echo "❌ Product folder not found: geist/product/"
    MISSING_PRODUCT=true
else
    echo "✅ Product folder found: geist/product/"
    
    # Check for required product files
    if [ ! -f "geist/product/mission.md" ]; then
        echo "❌ Missing: geist/product/mission.md"
        MISSING_PRODUCT=true
    else
        echo "✅ Found: geist/product/mission.md"
    fi
    
    if [ ! -f "geist/product/roadmap.md" ]; then
        echo "❌ Missing: geist/product/roadmap.md"
        MISSING_PRODUCT=true
    else
        echo "✅ Found: geist/product/roadmap.md"
    fi
    
    if [ ! -f "geist/product/tech-stack.md" ]; then
        echo "❌ Missing: geist/product/tech-stack.md"
        MISSING_PRODUCT=true
    else
        echo "✅ Found: geist/product/tech-stack.md"
    fi
fi
```

### Step 3: Handle Missing Prerequisites

IF any prerequisites are missing, OUTPUT the following to the user and STOP:

```
❌ Prerequisites required before proceeding with deploy-agents.

Missing prerequisites detected:

[MISSING_BASEPOINTS_MESSAGE]
[MISSING_PRODUCT_MESSAGE]

Please create the missing prerequisites first:

[MISSING_BASEPOINTS_INSTRUCTIONS]
[MISSING_PRODUCT_INSTRUCTIONS]

Once all prerequisites exist, you can run `/deploy-agents` again.
```

**Specific Messages:**

- If basepoints are missing:
```
👉 Run `/create-basepoints` to generate basepoint documentation from your codebase

Required basepoints:
- geist/basepoints/headquarter.md
- At least one module-specific basepoint file (agent-base-[module-name].md)
```

- If product files are missing:
```
👉 Run `/plan-product` to create new product documentation
👉 Run `/adapt-to-product` to generate product documentation from existing codebase

Required product files:
- geist/product/mission.md
- geist/product/roadmap.md
- geist/product/tech-stack.md
```

**WAIT for user to create prerequisites before proceeding.**

### Step 4: Validate Prerequisites Content

IF all files exist, verify they contain extractable content:

```bash
# Verify basepoint files contain patterns, standards, flows, strategies
echo "Validating basepoint files contain extractable knowledge..."

if [ -f "geist/basepoints/headquarter.md" ]; then
    REQUIRED_SECTIONS=("Pattern" "Standard" "Flow" "Strategy" "Testing" "Test" "Architecture" "Abstraction")
    FOUND_SECTIONS=0
    
    for section in "${REQUIRED_SECTIONS[@]}"; do
        if grep -qi "$section" geist/basepoints/headquarter.md 2>/dev/null; then
            ((FOUND_SECTIONS++))
        fi
    done
    
    if [ $FOUND_SECTIONS -lt 2 ]; then
        echo "⚠️  Warning: headquarter.md may not contain sufficient extractable knowledge"
        echo "   Expected sections: Patterns, Standards, Flows, Strategies, Testing"
    else
        echo "✅ Basepoint files contain extractable knowledge"
    fi
fi

# Verify product files are not empty
for file in geist/product/mission.md geist/product/roadmap.md geist/product/tech-stack.md; do
    if [ -f "$file" ] && [ ! -s "$file" ]; then
        echo "⚠️  Warning: $file exists but is empty"
    fi
done
```

### Step 5: Confirm Validation Success

IF all prerequisites are validated successfully, proceed:

```bash
echo "✅ All prerequisites validated successfully!"
echo ""
echo "Basepoints:"
echo "  - Folder: geist/basepoints/"
echo "  - Headquarter: geist/basepoints/headquarter.md"
echo "  - Module basepoints: [count] found"
echo ""
echo "Product files:"
echo "  - Mission: geist/product/mission.md"
echo "  - Roadmap: geist/product/roadmap.md"
echo "  - Tech Stack: geist/product/tech-stack.md"
echo ""
echo "Project Knowledge:"
echo "  - Profile: ${PROJECT_PROFILE_LOADED:-false}"
echo "  - Enriched Knowledge: ${ENRICHED_KNOWLEDGE_LOADED:-false}"
echo ""
echo "Ready to proceed with knowledge extraction and command specialization."
```

{{UNLESS compiled_single_command}}
## Display confirmation and next step

Once prerequisites are validated, output the following message:

```
✅ Prerequisites validated!

- Basepoints folder: Found with headquarter.md and [count] module basepoint file(s)
- Product files: Found (mission.md, roadmap.md, tech-stack.md)
- Project Profile: [Loaded / Generated]
- Enriched Knowledge: [Loaded / Not found]
- Specialization: [comprehensive / standard] workflows recommended

NEXT STEP 👉 Run the command, `2-extract-basepoints-knowledge.md`
```
{{ENDUNLESS compiled_single_command}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your validation process aligns with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}

## Important Constraints

- **MANDATORY**: Both basepoints and product files must exist before proceeding
- If basepoints are missing, workflow must stop and direct user to run `/create-basepoints`
- If product files are missing, workflow must stop and direct user to run `/plan-product` or `/adapt-to-product`
- Process must stop gracefully if any prerequisites are missing
- Validation should verify files contain extractable content (patterns, standards, flows, strategies)
- Always provide clear instructions for creating missing prerequisites
- **NEW**: Load and use project profile for specialization hints
- **NEW**: Use enriched knowledge for workflow complexity decisions