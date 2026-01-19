# Validate Implementation

## Purpose

Run project-specific deterministic validation before marking tasks as complete. This workflow contains abstract validation steps that are specialized during `deploy-agents` based on the project's tech stack.

## Workflow

### Step 1: Determine Available Validators

```bash
SPEC_PATH="${SPEC_PATH:-geist/specs/current-spec}"
VALIDATION_CACHE="$SPEC_PATH/implementation/cache/validation"
mkdir -p "$VALIDATION_CACHE"

echo "═══════════════════════════════════════════════════════"
echo "  IMPLEMENTATION VALIDATION"
echo "═══════════════════════════════════════════════════════"
echo ""
```

### Step 2: Run Project-Specific Build (if applicable)

```bash
# {{PROJECT_BUILD_COMMAND}}
# This placeholder is replaced during deploy-agents with project-specific build command
# Examples after specialization:
#   - npm run build
#   - cargo build
#   - go build ./...
#   - make build
#   - python setup.py build

echo "🔨 Running build validation..."

BUILD_RESULT=0
{{PROJECT_BUILD_COMMAND}}
BUILD_RESULT=$?

if [ $BUILD_RESULT -eq 0 ]; then
    echo "   ✅ Build passed"
    echo "BUILD: PASSED" >> "$VALIDATION_CACHE/validation-results.txt"
else
    echo "   ❌ Build failed (exit code: $BUILD_RESULT)"
    echo "BUILD: FAILED ($BUILD_RESULT)" >> "$VALIDATION_CACHE/validation-results.txt"
fi
```

### Step 3: Run Project-Specific Tests (if applicable)

```bash
# {{PROJECT_TEST_COMMAND}}
# This placeholder is replaced during deploy-agents with project-specific test command
# Examples after specialization:
#   - npm test
#   - cargo test
#   - go test ./...
#   - pytest
#   - make test

echo "🧪 Running test validation..."

TEST_RESULT=0
{{PROJECT_TEST_COMMAND}}
TEST_RESULT=$?

if [ $TEST_RESULT -eq 0 ]; then
    echo "   ✅ Tests passed"
    echo "TESTS: PASSED" >> "$VALIDATION_CACHE/validation-results.txt"
else
    echo "   ❌ Tests failed (exit code: $TEST_RESULT)"
    echo "TESTS: FAILED ($TEST_RESULT)" >> "$VALIDATION_CACHE/validation-results.txt"
fi
```

### Step 4: Run Project-Specific Lint (if applicable)

```bash
# {{PROJECT_LINT_COMMAND}}
# This placeholder is replaced during deploy-agents with project-specific lint command
# Examples after specialization:
#   - npm run lint
#   - cargo clippy
#   - golangci-lint run
#   - flake8 / pylint
#   - shellcheck

echo "📏 Running lint validation..."

LINT_RESULT=0
{{PROJECT_LINT_COMMAND}}
LINT_RESULT=$?

if [ $LINT_RESULT -eq 0 ]; then
    echo "   ✅ Lint passed"
    echo "LINT: PASSED" >> "$VALIDATION_CACHE/validation-results.txt"
else
    echo "   ⚠️ Lint warnings/errors (exit code: $LINT_RESULT)"
    echo "LINT: WARNINGS ($LINT_RESULT)" >> "$VALIDATION_CACHE/validation-results.txt"
fi
```

### Step 5: Run Project-Specific Type Check (if applicable)

```bash
# {{PROJECT_TYPECHECK_COMMAND}}
# This placeholder is replaced during deploy-agents with project-specific type check
# Examples after specialization:
#   - npx tsc --noEmit
#   - mypy .
#   - cargo check

echo "🔍 Running type check validation..."

TYPECHECK_RESULT=0
{{PROJECT_TYPECHECK_COMMAND}}
TYPECHECK_RESULT=$?

if [ $TYPECHECK_RESULT -eq 0 ]; then
    echo "   ✅ Type check passed"
    echo "TYPECHECK: PASSED" >> "$VALIDATION_CACHE/validation-results.txt"
else
    echo "   ❌ Type check failed (exit code: $TYPECHECK_RESULT)"
    echo "TYPECHECK: FAILED ($TYPECHECK_RESULT)" >> "$VALIDATION_CACHE/validation-results.txt"
fi
```

### Step 6: Run Custom Project Validators

```bash
# {{PROJECT_CUSTOM_VALIDATORS}}
# This placeholder is replaced during deploy-agents with any project-specific validators
# Examples:
#   - Database migration check
#   - API contract validation
#   - Security scan
#   - Documentation generation check

echo "🔧 Running custom validators..."

{{PROJECT_CUSTOM_VALIDATORS}}
```

### Step 7: Generate Validation Summary

```bash
echo ""
echo "═══════════════════════════════════════════════════════"
echo "  VALIDATION SUMMARY"
echo "═══════════════════════════════════════════════════════"
echo ""

# Count results
PASSED_COUNT=$(grep -c "PASSED" "$VALIDATION_CACHE/validation-results.txt" 2>/dev/null || echo "0")
FAILED_COUNT=$(grep -c "FAILED" "$VALIDATION_CACHE/validation-results.txt" 2>/dev/null || echo "0")
WARNING_COUNT=$(grep -c "WARNINGS" "$VALIDATION_CACHE/validation-results.txt" 2>/dev/null || echo "0")

echo "  ✅ Passed: $PASSED_COUNT"
echo "  ❌ Failed: $FAILED_COUNT"
echo "  ⚠️ Warnings: $WARNING_COUNT"
echo ""

# Determine overall status
if [ "$FAILED_COUNT" -gt 0 ]; then
    VALIDATION_STATUS="FAILED"
    echo "  Overall Status: ❌ VALIDATION FAILED"
    echo ""
    echo "  ⛔ Cannot proceed until failures are fixed."
    echo ""
elif [ "$WARNING_COUNT" -gt 0 ]; then
    VALIDATION_STATUS="WARNINGS"
    echo "  Overall Status: ⚠️ PASSED WITH WARNINGS"
    echo ""
    echo "  Proceeding, but consider addressing warnings."
    echo ""
else
    VALIDATION_STATUS="PASSED"
    echo "  Overall Status: ✅ ALL VALIDATIONS PASSED"
    echo ""
fi

echo "═══════════════════════════════════════════════════════"

# Export for use by calling workflow
export VALIDATION_STATUS
export VALIDATION_FAILED_COUNT=$FAILED_COUNT
```

### Step 8: Handle Validation Failure

```bash
# If validation failed, the calling workflow should:
# 1. Present the failures to the user
# 2. Wait for fixes or user override
# 3. Only proceed after validation passes or user explicitly approves

if [ "$VALIDATION_STATUS" = "FAILED" ]; then
    echo ""
    echo "👤 Validation failed. Options:"
    echo ""
    echo "   1. Fix the issues and re-run validation"
    echo "   2. Override and proceed anyway (not recommended)"
    echo ""
    echo "Waiting for user decision..."
    
    # The calling workflow should handle this pause
    # by presenting options and waiting for user input
fi
```

## Specialization Points

During `deploy-agents`, these placeholders are replaced:

| Placeholder | Description | Example Values |
|-------------|-------------|----------------|
| `{{PROJECT_BUILD_COMMAND}}` | Build command | `npm run build`, `cargo build` |
| `{{PROJECT_TEST_COMMAND}}` | Test command | `npm test`, `pytest` |
| `{{PROJECT_LINT_COMMAND}}` | Lint command | `npm run lint`, `flake8` |
| `{{PROJECT_TYPECHECK_COMMAND}}` | Type check | `npx tsc --noEmit`, `mypy .` |
| `{{PROJECT_CUSTOM_VALIDATORS}}` | Custom checks | Project-specific scripts |

## Default Behavior (Before Specialization)

If placeholders are not replaced (project has no build/test/lint), the workflow:
- Skips that validation step
- Logs "No [validator] configured for this project"
- Proceeds to next step

## Integration with Commands

This workflow is called:
1. **After implementing each task group** (before marking complete)
2. **Before running human review check** (ensures code is valid first)
3. **At end of implement-tasks** (final validation)

Commands should check `$VALIDATION_STATUS`:
- `PASSED` → Proceed normally
- `WARNINGS` → Proceed with note
- `FAILED` → Stop and request user action
