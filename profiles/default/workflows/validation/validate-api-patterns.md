# Validate API Patterns Workflow

This workflow validates that API/backend implementations follow established patterns from basepoints.

## Purpose

- Ensure API endpoints follow project conventions
- Validate request/response patterns
- Check error handling consistency
- Verify authentication/authorization patterns

## Validation Steps

```bash
#!/bin/bash

echo "🔌 Validating API layer patterns..."

VALIDATION_PASSED=true
ISSUES=()

# Load API patterns from basepoints
API_BASEPOINTS=""
if [ -d "agent-os/basepoints/modules" ]; then
    API_BASEPOINTS=$(find agent-os/basepoints/modules -name "*.md" -exec grep -l -i "api\|endpoint\|route\|controller\|handler" {} \; 2>/dev/null)
fi
```

## Validation Checks

### 1. Endpoint Naming Convention

```bash
validate_endpoint_naming() {
    local dir="$1"
    local issues=""
    
    # Find route/endpoint definitions
    find "$dir" -name "*.ts" -o -name "*.js" -o -name "*.py" -o -name "*.rb" -o -name "*.go" 2>/dev/null | while read f; do
        # REST convention: lowercase, hyphenated
        ENDPOINTS=$(grep -oE "['\"]/[a-zA-Z0-9/_-]+['\"]" "$f" 2>/dev/null | tr -d "'\"/")
        
        for endpoint in $ENDPOINTS; do
            # Check for camelCase in URLs (anti-pattern)
            if echo "$endpoint" | grep -qE "[a-z][A-Z]"; then
                echo "Endpoint uses camelCase (use kebab-case): $endpoint in $f"
            fi
            
            # Check for underscores in URLs (anti-pattern for REST)
            if echo "$endpoint" | grep -q "_"; then
                echo "Endpoint uses underscores (use hyphens): $endpoint in $f"
            fi
        done
    done
    
    echo "$issues"
}
```

### 2. HTTP Method Validation

```bash
validate_http_methods() {
    local dir="$1"
    local issues=""
    
    # Check for proper HTTP method usage
    find "$dir" -name "*.ts" -o -name "*.js" -o -name "*.py" 2>/dev/null | while read f; do
        # Check for GET with body (anti-pattern)
        if grep -qE "\.get\(.*body:" "$f" || grep -qE "GET.*body" "$f"; then
            echo "GET request with body (anti-pattern): $f"
        fi
        
        # Check for POST used for fetching (should be GET)
        if grep -qE "\.post\(.*\/get|\.post\(.*\/fetch|\.post\(.*\/list" "$f"; then
            echo "POST used for fetching data (use GET): $f"
        fi
        
        # Check for PUT vs PATCH usage
        if grep -qE "\.put\(" "$f" && grep -qE "\.patch\(" "$f"; then
            # Both used - check if intentional
            :
        fi
    done
    
    echo "$issues"
}
```

### 3. Error Handling Validation

```bash
validate_error_handling() {
    local dir="$1"
    local issues=""
    
    # Check for proper error handling
    find "$dir" -name "*.ts" -o -name "*.js" 2>/dev/null | while read f; do
        # Check for unhandled async/await
        if grep -q "async" "$f"; then
            if ! grep -qE "try\s*{|\.catch\(" "$f"; then
                echo "Async function without error handling: $f"
            fi
        fi
        
        # Check for generic error messages
        if grep -qE "catch.*throw.*Error\(['\"]Error['\"]|throw.*['\"]Something went wrong" "$f"; then
            echo "Generic error message (be specific): $f"
        fi
    done
    
    # Check for consistent error response format
    ERROR_FORMATS=$(find "$dir" -name "*.ts" -o -name "*.js" | xargs grep -ohE "res\.(status|json).*error" 2>/dev/null | sort -u | wc -l)
    if [ "$ERROR_FORMATS" -gt 3 ]; then
        echo "Multiple error response formats detected - consider standardizing"
    fi
    
    echo "$issues"
}
```

### 4. Authentication Pattern Validation

```bash
validate_auth_patterns() {
    local dir="$1"
    local issues=""
    
    # Check for auth middleware usage
    find "$dir" -name "*.ts" -o -name "*.js" 2>/dev/null | while read f; do
        # Check for routes without auth (potential security issue)
        if grep -qE "router\.(post|put|patch|delete)" "$f"; then
            if ! grep -qE "auth|authenticate|authorize|protect|guard" "$f"; then
                echo "⚠️ Modifying route without visible auth middleware: $f"
            fi
        fi
        
        # Check for hardcoded tokens/secrets
        if grep -qE "Bearer [a-zA-Z0-9]+|api[_-]?key.*=.*['\"][a-zA-Z0-9]+" "$f"; then
            echo "🚨 Possible hardcoded token/secret: $f"
        fi
    done
    
    echo "$issues"
}
```

### 5. Response Format Validation

```bash
validate_response_format() {
    local dir="$1"
    local issues=""
    
    # Check for consistent response structure
    # Common patterns: { data, error, meta } or { success, data, message }
    
    find "$dir" -name "*.ts" -o -name "*.js" 2>/dev/null | while read f; do
        # Count different response patterns
        DATA_PATTERN=$(grep -c "res.json({ data:" "$f" 2>/dev/null || echo 0)
        SUCCESS_PATTERN=$(grep -c "res.json({ success:" "$f" 2>/dev/null || echo 0)
        DIRECT_PATTERN=$(grep -c "res.json([^{]" "$f" 2>/dev/null || echo 0)
        
        # If multiple patterns in same file
        TOTAL=$((DATA_PATTERN + SUCCESS_PATTERN + DIRECT_PATTERN))
        if [ "$TOTAL" -gt 1 ] && [ "$DATA_PATTERN" -gt 0 ] && [ "$SUCCESS_PATTERN" -gt 0 ]; then
            echo "Inconsistent response format in: $f"
        fi
    done
    
    echo "$issues"
}
```

## Main Validation Orchestration

```bash
run_api_validation() {
    local target_dir="${1:-.}"
    local results_file="agent-os/output/validation/api-validation-results.md"
    
    mkdir -p agent-os/output/validation
    
    echo "# API Pattern Validation Results" > "$results_file"
    echo "" >> "$results_file"
    echo "**Timestamp:** $(date)" >> "$results_file"
    echo "**Target:** $target_dir" >> "$results_file"
    echo "" >> "$results_file"
    
    # Run all API validations
    echo "## Endpoint Naming" >> "$results_file"
    NAMING_ISSUES=$(validate_endpoint_naming "$target_dir")
    if [ -n "$NAMING_ISSUES" ]; then
        echo "$NAMING_ISSUES" >> "$results_file"
        VALIDATION_PASSED=false
    else
        echo "✅ Endpoint naming follows conventions" >> "$results_file"
    fi
    
    echo "" >> "$results_file"
    echo "## HTTP Methods" >> "$results_file"
    METHOD_ISSUES=$(validate_http_methods "$target_dir")
    if [ -n "$METHOD_ISSUES" ]; then
        echo "$METHOD_ISSUES" >> "$results_file"
        VALIDATION_PASSED=false
    else
        echo "✅ HTTP methods used correctly" >> "$results_file"
    fi
    
    echo "" >> "$results_file"
    echo "## Error Handling" >> "$results_file"
    ERROR_ISSUES=$(validate_error_handling "$target_dir")
    if [ -n "$ERROR_ISSUES" ]; then
        echo "$ERROR_ISSUES" >> "$results_file"
        VALIDATION_PASSED=false
    else
        echo "✅ Error handling is consistent" >> "$results_file"
    fi
    
    echo "" >> "$results_file"
    echo "## Authentication" >> "$results_file"
    AUTH_ISSUES=$(validate_auth_patterns "$target_dir")
    if [ -n "$AUTH_ISSUES" ]; then
        echo "$AUTH_ISSUES" >> "$results_file"
        echo "⚠️ Review authentication patterns" >> "$results_file"
    else
        echo "✅ Authentication patterns look good" >> "$results_file"
    fi
    
    echo "" >> "$results_file"
    echo "## Response Format" >> "$results_file"
    RESPONSE_ISSUES=$(validate_response_format "$target_dir")
    if [ -n "$RESPONSE_ISSUES" ]; then
        echo "$RESPONSE_ISSUES" >> "$results_file"
    else
        echo "✅ Response format is consistent" >> "$results_file"
    fi
    
    # Summary
    echo "" >> "$results_file"
    echo "---" >> "$results_file"
    if [ "$VALIDATION_PASSED" = true ]; then
        echo "## ✅ API Validation Passed" >> "$results_file"
        exit 0
    else
        echo "## ❌ API Validation Failed" >> "$results_file"
        echo "See issues above for details." >> "$results_file"
        exit 1
    fi
}

# Run validation
run_api_validation "$TARGET_DIR"
```

## Integration with Layer Specialists

When `api-specialist` completes implementation, this validation runs automatically. The validation loads patterns from:
- `agent-os/basepoints/modules/` (API-related modules)
- `agent-os/product/tech-stack.md` (framework-specific rules)
- `agent-os/standards/global/error-handling.md`
