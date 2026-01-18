Now that commands are specialized, proceed with specializing standards based on project needs by following these instructions:

## Core Responsibilities

1. **Analyze Project Complexity**: Determine which standards are needed based on project complexity
2. **Evaluate Existing Standards**: Check which standards from templates are relevant to this project
3. **Specialize Standards**: Update, simplify, or extend standards based on project patterns
4. **Remove Unnecessary Standards**: For simple projects, remove standards that add unnecessary complexity
5. **Create Project-Specific Standards**: Add new standards based on project-specific patterns from basepoints

## Workflow

### Step 1: Load Knowledge and Assess Complexity

Load knowledge and assess project complexity:

```bash
# Load merged knowledge
if [ -f "agent-os/output/deploy-agents/knowledge/merged-knowledge.json" ]; then
    MERGED_KNOWLEDGE=$(cat agent-os/output/deploy-agents/knowledge/merged-knowledge.json)
    echo "✅ Loaded merged knowledge"
fi

# Load basepoints knowledge for patterns
if [ -f "agent-os/output/deploy-agents/knowledge/basepoints-knowledge.json" ]; then
    BASEPOINTS_KNOWLEDGE=$(cat agent-os/output/deploy-agents/knowledge/basepoints-knowledge.json)
fi

# Load complexity assessment
if [ -f "agent-os/output/deploy-agents/reports/complexity-assessment.json" ]; then
    COMPLEXITY=$(cat agent-os/output/deploy-agents/reports/complexity-assessment.json)
    PROJECT_NATURE=$(echo "$COMPLEXITY" | grep -o '"nature": *"[^"]*"' | cut -d'"' -f4)
else
    PROJECT_NATURE="moderate"
fi

echo "Project Nature: $PROJECT_NATURE"
```

### Step 2: Evaluate Standards Relevance

Determine which standards are needed for this project:

```bash
# List existing standards
EXISTING_STANDARDS=$(find agent-os/standards -name "*.md" -type f 2>/dev/null)

# For each standard, evaluate relevance based on:
# 1. Project patterns from basepoints
# 2. Project complexity
# 3. Tech stack requirements

STANDARDS_EVALUATION=""
for standard_file in $EXISTING_STANDARDS; do
    STANDARD_NAME=$(basename "$standard_file" .md)
    
    # Evaluate based on complexity
    case "$PROJECT_NATURE" in
        "simple")
            # Simple projects need only core standards
            if [[ "$STANDARD_NAME" =~ ^(coding-style|conventions|error-handling)$ ]]; then
                RELEVANCE="required"
            else
                RELEVANCE="optional"
            fi
            ;;
        "moderate")
            # Moderate projects need most standards
            RELEVANCE="required"
            ;;
        "complex")
            # Complex projects need all standards plus project-specific
            RELEVANCE="required"
            ;;
    esac
    
    STANDARDS_EVALUATION="${STANDARDS_EVALUATION}${STANDARD_NAME}:${RELEVANCE}\n"
done
```

### Step 3: Classify Standards as Project-Wide vs Module-Specific

Before specializing standards, classify extracted patterns to include only project-wide patterns:

```bash
echo "📊 Classifying standards patterns..."

# Extract standards-related patterns from basepoints
BASEPOINT_STANDARDS=$(extract_standards_from_basepoints "$BASEPOINTS_KNOWLEDGE")

# Define cross-cutting concern keywords (always project-wide)
CROSS_CUTTING_KEYWORDS="testing|test|lint|linting|naming|convention|error|handling|sdd|documentation|logging|security|authentication|validation|formatting|style"

# Initialize classification collections
PROJECT_WIDE_PATTERNS=""
MODULE_SPECIFIC_PATTERNS=""

# Classify each pattern
classify_pattern() {
    local pattern_name="$1"
    local pattern_content="$2"
    local pattern_sources="$3"  # Which modules this pattern appears in
    
    # Count how many modules this pattern appears in
    local module_count=$(echo "$pattern_sources" | tr ',' '\n' | wc -l | tr -d ' ')
    
    # Check if pattern is a cross-cutting concern
    local is_cross_cutting=$(echo "$pattern_name" | grep -iE "$CROSS_CUTTING_KEYWORDS" && echo "true" || echo "false")
    
    # Classification logic:
    # 1. Cross-cutting concerns are always project-wide
    # 2. Patterns appearing in 3+ modules are project-wide
    # 3. Patterns appearing in 1-2 modules are module-specific
    
    if [ "$is_cross_cutting" = "true" ]; then
        echo "project-wide:cross-cutting"
    elif [ "$module_count" -ge 3 ]; then
        echo "project-wide:multi-module"
    else
        echo "module-specific"
    fi
}

# Process each pattern from basepoints
echo "$BASEPOINT_STANDARDS" | while IFS='|' read -r pattern_name pattern_content pattern_sources; do
    if [ -z "$pattern_name" ]; then
        continue
    fi
    
    CLASSIFICATION=$(classify_pattern "$pattern_name" "$pattern_content" "$pattern_sources")
    
    case "$CLASSIFICATION" in
        project-wide:*)
            echo "   ✅ Project-wide: $pattern_name (${CLASSIFICATION#project-wide:})"
            PROJECT_WIDE_PATTERNS="${PROJECT_WIDE_PATTERNS}${pattern_name}|${pattern_content}\n"
            ;;
        module-specific)
            echo "   📦 Module-specific (kept in basepoints): $pattern_name"
            MODULE_SPECIFIC_PATTERNS="${MODULE_SPECIFIC_PATTERNS}${pattern_name}|${pattern_content}\n"
            ;;
    esac
done

echo ""
echo "📋 Classification Summary:"
echo "   Project-wide patterns: $(echo -e "$PROJECT_WIDE_PATTERNS" | grep -c '|')"
echo "   Module-specific patterns (kept in basepoints): $(echo -e "$MODULE_SPECIFIC_PATTERNS" | grep -c '|')"
```

### Step 4: Specialize Standards with Project-Wide Patterns Only

Update standards with ONLY project-wide patterns from basepoints:

```bash
echo "📝 Specializing standards with project-wide patterns only..."

# For each existing standard
for standard_file in $EXISTING_STANDARDS; do
    STANDARD_NAME=$(basename "$standard_file" .md)
    STANDARD_CONTENT=$(cat "$standard_file")
    
    # Extract ONLY project-wide patterns for this standard type
    # Filter out module-specific patterns
    PROJECT_PATTERNS=$(echo -e "$PROJECT_WIDE_PATTERNS" | grep -i "$STANDARD_NAME" | cut -d'|' -f2)
    
    if [ -n "$PROJECT_PATTERNS" ]; then
        # Merge project-wide patterns into standard
        UPDATED_CONTENT=$(merge_patterns_into_standard "$STANDARD_CONTENT" "$PROJECT_PATTERNS")
        echo "$UPDATED_CONTENT" > "$standard_file"
        echo "✅ Updated standard with project-wide patterns: $STANDARD_NAME"
    fi
done

# Add note about module-specific patterns
echo ""
echo "ℹ️  Module-specific patterns were NOT added to standards files."
echo "   These patterns remain accessible in basepoints for when needed."
echo "   Reference: agent-os/basepoints/ for module-specific patterns."
```

### Step 5: Simplify Standards for Simple Projects

For simple projects, remove or combine unnecessary standards:

```bash
if [ "$PROJECT_NATURE" = "simple" ]; then
    echo "📋 Simplifying standards for simple project..."
    
    # List of standards that can be removed for simple projects
    OPTIONAL_STANDARDS=(
        "codebase-analysis.md"
        "validation.md"
    )
    
    for optional in "${OPTIONAL_STANDARDS[@]}"; do
        OPTIONAL_FILE="agent-os/standards/global/$optional"
        if [ -f "$OPTIONAL_FILE" ]; then
            # Instead of removing, mark as optional or reduce content
            echo "ℹ️  Standard marked as optional for simple project: $optional"
        fi
    done
    
    echo "✅ Standards simplified for simple project"
fi
```

### Step 6: Specialize Validation Commands

Load from project profile (if available) or detect tech stack and specialize validation commands:

```bash
echo "🔧 Specializing validation commands..."

VALIDATION_FILE="agent-os/standards/global/validation-commands.md"

# FIRST: Try to load from project profile (generated by Adaptive Questionnaire System)
if [ -f "agent-os/config/project-profile.yml" ]; then
    echo "   Loading commands from project profile..."
    
    BUILD_CMD=$(grep "build:" agent-os/config/project-profile.yml | head -1 | cut -d'"' -f2)
    TEST_CMD=$(grep "test:" agent-os/config/project-profile.yml | head -1 | cut -d'"' -f2)
    LINT_CMD=$(grep "lint:" agent-os/config/project-profile.yml | head -1 | cut -d'"' -f2)
    
    # Also load security level for review trigger configuration
    SECURITY_LEVEL=$(grep "security_level:" agent-os/config/project-profile.yml | head -1 | awk '{print $2}')
    COMPLEXITY=$(grep "complexity:" agent-os/config/project-profile.yml | head -1 | awk '{print $2}')
    
    if [ -n "$BUILD_CMD" ] || [ -n "$TEST_CMD" ] || [ -n "$LINT_CMD" ]; then
        echo "   ✓ Found commands in project profile"
        PROFILE_COMMANDS_LOADED=true
    fi
fi

# SECOND: Load enriched knowledge for additional context
if [ -d "agent-os/config/enriched-knowledge" ]; then
    echo "   Loading enriched knowledge for specialization hints..."
    
    # Check for security notes to adjust review triggers
    if [ -f "agent-os/config/enriched-knowledge/security-notes.md" ]; then
        if grep -q "CRITICAL\|🔴" agent-os/config/enriched-knowledge/security-notes.md 2>/dev/null; then
            echo "   ⚠️ Critical security issues found - enabling strict review"
            STRICT_REVIEW=true
        fi
    fi
    
    # Check version analysis for outdated deps
    if [ -f "agent-os/config/enriched-knowledge/version-analysis.md" ]; then
        if grep -q "OUTDATED" agent-os/config/enriched-knowledge/version-analysis.md 2>/dev/null; then
            echo "   ℹ️ Outdated dependencies - consider updating"
        fi
    fi
fi

# THIRD: Fallback to auto-detection if profile not loaded
if [ "$PROFILE_COMMANDS_LOADED" != "true" ]; then
    echo "   No profile found, detecting from project files..."
fi

# Detect tech stack from project files (fallback or supplement)
# NOTE: This function contains technology-specific detection patterns (Node.js, Rust, Go, Python, etc.)
# These are common examples used for detecting and specializing validation commands during deployment.
# This detection logic is functional code required for the specialization process to work.
# The patterns shown here are illustrative examples that get specialized based on the actual project's tech stack.
detect_validation_commands() {
    local BUILD_CMD=""
    local TEST_CMD=""
    local LINT_CMD=""
    local TYPECHECK_CMD=""
    local FORMAT_CMD=""
    
    # Node.js / JavaScript / TypeScript - Example detection pattern
    if [ -f "package.json" ]; then
        echo "   Detected: Node.js project"
        
        # Check for common scripts in package.json
        if grep -q '"build"' package.json 2>/dev/null; then
            BUILD_CMD="npm run build"
        fi
        if grep -q '"test"' package.json 2>/dev/null; then
            TEST_CMD="npm test"
        fi
        if grep -q '"lint"' package.json 2>/dev/null; then
            LINT_CMD="npm run lint"
        fi
        if grep -q '"typecheck"' package.json 2>/dev/null; then
            TYPECHECK_CMD="npm run typecheck"
        elif [ -f "tsconfig.json" ]; then
            TYPECHECK_CMD="npx tsc --noEmit"
        fi
        if grep -q '"format:check"' package.json 2>/dev/null; then
            FORMAT_CMD="npm run format:check"
        elif grep -q '"prettier"' package.json 2>/dev/null; then
            FORMAT_CMD="npx prettier --check ."
        fi
    fi
    
    # Rust - Example detection pattern
    if [ -f "Cargo.toml" ]; then
        echo "   Detected: Rust project"
        BUILD_CMD="cargo build"
        TEST_CMD="cargo test"
        LINT_CMD="cargo clippy -- -D warnings"
        TYPECHECK_CMD="cargo check"
        FORMAT_CMD="cargo fmt --check"
    fi
    
    # Go - Example detection pattern
    if [ -f "go.mod" ]; then
        echo "   Detected: Go project"
        BUILD_CMD="go build ./..."
        TEST_CMD="go test ./..."
        LINT_CMD="golangci-lint run"
        TYPECHECK_CMD="go vet ./..."
        FORMAT_CMD="gofmt -l ."
    fi
    
    # Python - Example detection pattern
    if [ -f "requirements.txt" ] || [ -f "pyproject.toml" ] || [ -f "setup.py" ]; then
        echo "   Detected: Python project"
        if [ -f "pyproject.toml" ] && grep -q "pytest" pyproject.toml 2>/dev/null; then
            TEST_CMD="pytest"
        elif [ -f "requirements.txt" ] && grep -q "pytest" requirements.txt 2>/dev/null; then
            TEST_CMD="pytest"
        fi
        if grep -q "flake8" requirements.txt 2>/dev/null || grep -q "flake8" pyproject.toml 2>/dev/null; then
            LINT_CMD="flake8 ."
        elif grep -q "pylint" requirements.txt 2>/dev/null; then
            LINT_CMD="pylint ."
        fi
        if grep -q "mypy" requirements.txt 2>/dev/null || grep -q "mypy" pyproject.toml 2>/dev/null; then
            TYPECHECK_CMD="mypy ."
        fi
        if grep -q "black" requirements.txt 2>/dev/null || grep -q "black" pyproject.toml 2>/dev/null; then
            FORMAT_CMD="black --check ."
        fi
    fi
    
    # Makefile (generic)
    if [ -f "Makefile" ]; then
        echo "   Detected: Makefile"
        if grep -q "^build:" Makefile 2>/dev/null && [ -z "$BUILD_CMD" ]; then
            BUILD_CMD="make build"
        fi
        if grep -q "^test:" Makefile 2>/dev/null && [ -z "$TEST_CMD" ]; then
            TEST_CMD="make test"
        fi
        if grep -q "^lint:" Makefile 2>/dev/null && [ -z "$LINT_CMD" ]; then
            LINT_CMD="make lint"
        fi
    fi
    
    # Shell scripts (for shell-based projects like Geist)
    if ls scripts/*.sh 1>/dev/null 2>&1; then
        echo "   Detected: Shell scripts"
        if [ -z "$LINT_CMD" ]; then
            LINT_CMD="shellcheck scripts/*.sh"
        fi
    fi
    
    # Output detected commands
    echo "BUILD_CMD=$BUILD_CMD"
    echo "TEST_CMD=$TEST_CMD"
    echo "LINT_CMD=$LINT_CMD"
    echo "TYPECHECK_CMD=$TYPECHECK_CMD"
    echo "FORMAT_CMD=$FORMAT_CMD"
}

# Run detection
DETECTED_COMMANDS=$(detect_validation_commands)

# Extract individual commands
BUILD_CMD=$(echo "$DETECTED_COMMANDS" | grep "BUILD_CMD=" | cut -d'=' -f2)
TEST_CMD=$(echo "$DETECTED_COMMANDS" | grep "TEST_CMD=" | cut -d'=' -f2)
LINT_CMD=$(echo "$DETECTED_COMMANDS" | grep "LINT_CMD=" | cut -d'=' -f2)
TYPECHECK_CMD=$(echo "$DETECTED_COMMANDS" | grep "TYPECHECK_CMD=" | cut -d'=' -f2)
FORMAT_CMD=$(echo "$DETECTED_COMMANDS" | grep "FORMAT_CMD=" | cut -d'=' -f2)

# Update validation-commands.md with detected commands
if [ -f "$VALIDATION_FILE" ]; then
    # Replace placeholders with detected commands
    sed -i.bak "s|{{PROJECT_BUILD_COMMAND}}|${BUILD_CMD:-echo \"No build command configured\"}|g" "$VALIDATION_FILE"
    sed -i.bak "s|{{PROJECT_TEST_COMMAND}}|${TEST_CMD:-echo \"No test command configured\"}|g" "$VALIDATION_FILE"
    sed -i.bak "s|{{PROJECT_LINT_COMMAND}}|${LINT_CMD:-echo \"No lint command configured\"}|g" "$VALIDATION_FILE"
    sed -i.bak "s|{{PROJECT_TYPECHECK_COMMAND}}|${TYPECHECK_CMD:-echo \"No type check configured\"}|g" "$VALIDATION_FILE"
    sed -i.bak "s|{{PROJECT_FORMAT_CHECK_COMMAND}}|${FORMAT_CMD:-echo \"No format check configured\"}|g" "$VALIDATION_FILE"
    rm -f "${VALIDATION_FILE}.bak"
    
    echo "✅ Specialized validation commands"
    echo "   Build: ${BUILD_CMD:-Not configured}"
    echo "   Test: ${TEST_CMD:-Not configured}"
    echo "   Lint: ${LINT_CMD:-Not configured}"
    echo "   Type Check: ${TYPECHECK_CMD:-Not configured}"
    echo "   Format: ${FORMAT_CMD:-Not configured}"
fi

# Also update the validate-implementation.md workflow
IMPL_VALIDATION_FILE="agent-os/workflows/validation/validate-implementation.md"
if [ -f "$IMPL_VALIDATION_FILE" ]; then
    sed -i.bak "s|{{PROJECT_BUILD_COMMAND}}|${BUILD_CMD:-echo \"No build command configured\"}|g" "$IMPL_VALIDATION_FILE"
    sed -i.bak "s|{{PROJECT_TEST_COMMAND}}|${TEST_CMD:-echo \"No test command configured\"}|g" "$IMPL_VALIDATION_FILE"
    sed -i.bak "s|{{PROJECT_LINT_COMMAND}}|${LINT_CMD:-echo \"No lint command configured\"}|g" "$IMPL_VALIDATION_FILE"
    sed -i.bak "s|{{PROJECT_TYPECHECK_COMMAND}}|${TYPECHECK_CMD:-echo \"No type check configured\"}|g" "$IMPL_VALIDATION_FILE"
    sed -i.bak "s|{{PROJECT_CUSTOM_VALIDATORS}}|echo \"No custom validators configured\"|g" "$IMPL_VALIDATION_FILE"
    rm -f "${IMPL_VALIDATION_FILE}.bak"
    
    echo "✅ Updated validate-implementation workflow"
fi
```

### Step 7: Create Project-Specific Standards (Project-Wide Only)

Create new standards based on unique project patterns:

```bash
# Extract project-specific patterns that don't fit existing standards
# IMPORTANT: Only include patterns that are project-wide (appear in multiple modules or are cross-cutting)
PROJECT_SPECIFIC=$(extract_unique_patterns "$BASEPOINTS_KNOWLEDGE" | filter_project_wide_only)

if [ -n "$PROJECT_SPECIFIC" ]; then
    # Create project-specific standards directory if needed
    mkdir -p agent-os/standards/project
    
    # Generate project-specific standards
    for pattern_type in $PROJECT_SPECIFIC; do
        PATTERN_NAME=$(echo "$pattern_type" | cut -d':' -f1)
        PATTERN_CONTENT=$(echo "$pattern_type" | cut -d':' -f2-)
        
        cat > "agent-os/standards/project/${PATTERN_NAME}.md" << EOF
# Project-Specific Standard: ${PATTERN_NAME}

## Overview

This standard was generated based on patterns detected in the project's codebase.

## Guidelines

${PATTERN_CONTENT}

## Source

Extracted from basepoints analysis during deploy-agents specialization.
EOF
        
        echo "✅ Created project-specific standard: ${PATTERN_NAME}"
    done
fi
```

### Step 8: Verify Standards Consistency

Verify all standards are consistent and properly referenced:

```bash
# Count standards
STANDARDS_COUNT=$(find agent-os/standards -name "*.md" -type f | wc -l | tr -d ' ')

# Log to reports
mkdir -p agent-os/output/deploy-agents/reports
cat > agent-os/output/deploy-agents/reports/standards-specialization.md << EOF
# Standards Specialization Report

## Summary
- Project Nature: $PROJECT_NATURE
- Total Standards: $STANDARDS_COUNT

## Standards Processed
$(find agent-os/standards -name "*.md" -type f | while read f; do
    echo "- $(basename "$f" .md): $(dirname "$f" | sed 's|agent-os/standards/||')"
done)

## Validation Commands Specialized
- **Build Command:** ${BUILD_CMD:-Not configured}
- **Test Command:** ${TEST_CMD:-Not configured}
- **Lint Command:** ${LINT_CMD:-Not configured}
- **Type Check:** ${TYPECHECK_CMD:-Not configured}
- **Format Check:** ${FORMAT_CMD:-Not configured}

## Actions Taken
- Classified patterns as project-wide vs module-specific
- Updated standards with project-wide patterns ONLY from basepoints
- Excluded module-specific patterns (kept in basepoints for reference)
- Specialized validation commands based on detected tech stack
- Simplified standards for project complexity level
- Created project-specific standards where needed (project-wide only)

## Standards Filtering Applied
- Cross-cutting concerns (testing, lint, naming, error handling, SDD) → Project-wide
- Patterns appearing in 3+ modules → Project-wide
- Patterns appearing in 1-2 modules → Module-specific (kept in basepoints only)

## Module-Specific Patterns Location
Module-specific patterns are NOT included in standards files.
They remain accessible in basepoints: \`agent-os/basepoints/\`

## Next Steps
Proceed to specialize agents in phase 9.
EOF

echo "✅ Standards specialization complete"
echo "📁 Report saved to: agent-os/output/deploy-agents/reports/standards-specialization.md"
```

{{UNLESS compiled_single_command}}
## Display confirmation and next step

Once you've specialized standards, output the following message:

```
✅ Standards specialized for project!

**Project Nature:** [simple/moderate/complex]
**Standards Updated:** [count]
**Project-Specific Standards Created:** [count]

Report: `agent-os/output/deploy-agents/reports/standards-specialization.md`

NEXT STEP 👉 Run the command, `9-specialize-agents.md`
```
{{ENDUNLESS compiled_single_command}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

Ensure standards specialization follows the user's preferences:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
