# Validation Registry

## Core Responsibilities

1. **Define Core Validators**: List validators that run for ALL projects
2. **Define Project Validators Placeholder**: Allow specialization injection
3. **Provide run_all_validators Function**: Orchestrate validator execution
4. **Provide Validator Composition Logic**: Allow combining validators

## Core Validators Array

```bash
# Core validators that run for ALL projects
# These are project-agnostic and work without specialization

CORE_VALIDATORS=(
    "validate-output-exists"      # Check required files exist
    "validate-knowledge-integration"  # Check knowledge cache
    "validate-references"         # Check @geist/ references resolve
)

# Description of each core validator
declare -A VALIDATOR_DESCRIPTIONS
VALIDATOR_DESCRIPTIONS["validate-output-exists"]="Checks that all required output files exist and have content"
VALIDATOR_DESCRIPTIONS["validate-knowledge-integration"]="Verifies basepoints knowledge was extracted and used"
VALIDATOR_DESCRIPTIONS["validate-references"]="Ensures all @geist/ references resolve to existing paths"
```

## Project Validators Placeholder

```bash
# Project-specific validators added during deploy-agents
# This array is populated by specialization

PROJECT_VALIDATORS=(
    {{PROJECT_VALIDATORS}}
)

# Example after specialization for a React project:
# PROJECT_VALIDATORS=(
#     "validate-component-structure"
#     "validate-hooks-usage"
#     "validate-typescript-types"
# )
```

## Run All Validators Function

```bash
# Run all validators (core + project)
run_all_validators() {
    SPEC_PATH="$1"
    COMMAND="$2"
    
    if [ -z "$SPEC_PATH" ]; then
        echo "❌ SPEC_PATH required"
        return 1
    fi
    
    echo "═══════════════════════════════════════════════════════"
    echo "  RUNNING VALIDATION PIPELINE"
    echo "═══════════════════════════════════════════════════════"
    echo ""
    echo "  Spec Path: $SPEC_PATH"
    echo "  Command: ${COMMAND:-unknown}"
    echo ""
    
    VALIDATION_FAILED="false"
    VALIDATORS_RUN=0
    VALIDATORS_PASSED=0
    VALIDATORS_FAILED=0
    
    # Run core validators
    echo "📋 Running Core Validators..."
    echo ""
    
    # Determine workflow base path (geist when installed, profiles/default for template)
    if [ -d "geist/workflows" ]; then
        WORKFLOWS_BASE="geist/workflows"
    else
        WORKFLOWS_BASE="profiles/default/workflows"
    fi
    
    for validator in "${CORE_VALIDATORS[@]}"; do
        echo "  Running: $validator"
        
        # Execute validator
        SPEC_PATH="$SPEC_PATH" COMMAND="$COMMAND" \
            source "$WORKFLOWS_BASE/validation/$validator.md"
        
        VALIDATORS_RUN=$((VALIDATORS_RUN + 1))
        
        if [ $? -eq 0 ]; then
            echo "    ✅ Passed"
            VALIDATORS_PASSED=$((VALIDATORS_PASSED + 1))
        else
            echo "    ❌ Failed"
            VALIDATORS_FAILED=$((VALIDATORS_FAILED + 1))
            VALIDATION_FAILED="true"
        fi
        echo ""
    done
    
    # Run project validators (if any)
    if [ ${#PROJECT_VALIDATORS[@]} -gt 0 ] && [ "${PROJECT_VALIDATORS[0]}" != "{{PROJECT_VALIDATORS}}" ]; then
        echo "📋 Running Project Validators..."
        echo ""
        
        for validator in "${PROJECT_VALIDATORS[@]}"; do
            [ -z "$validator" ] && continue
            
            echo "  Running: $validator"
            
            # Execute validator
            SPEC_PATH="$SPEC_PATH" COMMAND="$COMMAND" \
                source "$WORKFLOWS_BASE/validation/$validator.md" 2>/dev/null
            
            VALIDATORS_RUN=$((VALIDATORS_RUN + 1))
            
            if [ $? -eq 0 ]; then
                echo "    ✅ Passed"
                VALIDATORS_PASSED=$((VALIDATORS_PASSED + 1))
            else
                echo "    ❌ Failed"
                VALIDATORS_FAILED=$((VALIDATORS_FAILED + 1))
                VALIDATION_FAILED="true"
            fi
            echo ""
        done
    fi
    
    # Generate validation report
    echo "📊 Generating Validation Report..."
    SPEC_PATH="$SPEC_PATH" COMMAND="$COMMAND" \
        source "$WORKFLOWS_BASE/validation/generate-validation-report.md"
    
    echo ""
    echo "═══════════════════════════════════════════════════════"
    echo "  VALIDATION PIPELINE COMPLETE"
    echo "═══════════════════════════════════════════════════════"
    echo ""
    echo "  Validators Run: $VALIDATORS_RUN"
    echo "  Passed: $VALIDATORS_PASSED"
    echo "  Failed: $VALIDATORS_FAILED"
    echo ""
    
    if [ "$VALIDATION_FAILED" = "true" ]; then
        echo "  Overall Status: ❌ FAILED"
        return 1
    else
        echo "  Overall Status: ✅ PASSED"
        return 0
    fi
}
```

## Validator Composition Logic

```bash
# Compose validators for specific contexts
compose_validators() {
    CONTEXT="$1"  # "shape-spec", "write-spec", "create-tasks", etc.
    
    case "$CONTEXT" in
        "shape-spec")
            # Minimal validation for shape-spec
            COMPOSED_VALIDATORS=("validate-output-exists")
            ;;
        "write-spec")
            # Full validation for write-spec
            COMPOSED_VALIDATORS=("validate-output-exists" "validate-knowledge-integration")
            ;;
        "create-tasks")
            # Full validation for create-tasks
            COMPOSED_VALIDATORS=("validate-output-exists" "validate-knowledge-integration" "validate-references")
            ;;
        "implement-tasks")
            # All validators for implement-tasks
            COMPOSED_VALIDATORS=("${CORE_VALIDATORS[@]}" "${PROJECT_VALIDATORS[@]}")
            ;;
        *)
            # Default: all core validators
            COMPOSED_VALIDATORS=("${CORE_VALIDATORS[@]}")
            ;;
    esac
    
    echo "${COMPOSED_VALIDATORS[@]}"
}

# Run composed validators for a context
run_validators_for_context() {
    SPEC_PATH="$1"
    CONTEXT="$2"
    
    # Determine workflow base path (geist when installed, profiles/default for template)
    if [ -d "geist/workflows" ]; then
        WORKFLOWS_BASE="geist/workflows"
    else
        WORKFLOWS_BASE="profiles/default/workflows"
    fi
    
    VALIDATORS_TO_RUN=$(compose_validators "$CONTEXT")
    
    echo "Running validators for $CONTEXT: $VALIDATORS_TO_RUN"
    
    for validator in $VALIDATORS_TO_RUN; do
        [ -z "$validator" ] && continue
        [ "$validator" = "{{PROJECT_VALIDATORS}}" ] && continue
        
        SPEC_PATH="$SPEC_PATH" COMMAND="$CONTEXT" \
            source "$WORKFLOWS_BASE/validation/$validator.md"
    done
}
```

## Export Functions

```bash
# Export functions for use by commands
export -f run_all_validators 2>/dev/null || true
export -f compose_validators 2>/dev/null || true
export -f run_validators_for_context 2>/dev/null || true

# Export arrays
export CORE_VALIDATORS
export PROJECT_VALIDATORS
```

## Important Constraints

- Core validators run for ALL projects without specialization
- Project validators are added during `deploy-agents` via placeholder
- `{{PROJECT_VALIDATORS}}` must be preserved for specialization
- Validator composition allows context-specific validation
- Exit codes: 0 = success, 1 = failure (for CI/CD integration)
- **CRITICAL**: This registry orchestrates ALL validation for the system
