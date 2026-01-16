# Validate Data Patterns Workflow

This workflow validates that data/persistence layer implementations follow established patterns from basepoints.

## Purpose

- Ensure data models follow project conventions
- Validate query patterns and performance
- Check migration conventions
- Verify data validation and sanitization

## Validation Steps

```bash
#!/bin/bash

echo "💾 Validating data layer patterns..."

VALIDATION_PASSED=true
ISSUES=()

# Load data patterns from basepoints
DATA_BASEPOINTS=""
if [ -d "agent-os/basepoints/modules" ]; then
    DATA_BASEPOINTS=$(find agent-os/basepoints/modules -name "*.md" -exec grep -l -i "model\|schema\|database\|entity\|repository" {} \; 2>/dev/null)
fi
```

## Validation Checks

### 1. Model Naming Convention

```bash
validate_model_naming() {
    local dir="$1"
    local issues=""
    
    # Find model files
    find "$dir" -name "*model*.ts" -o -name "*entity*.ts" -o -name "*schema*.ts" \
         -o -name "*model*.py" -o -name "*model*.rb" 2>/dev/null | while read f; do
        FILENAME=$(basename "$f" | sed 's/\.[^.]*$//')
        
        # Models should be PascalCase singular
        if echo "$FILENAME" | grep -qE "s$|Model$|Entity$"; then
            echo "Model name should be singular without suffix: $f"
        fi
    done
    
    # Check class/type names inside files
    find "$dir" -name "*.ts" -o -name "*.py" 2>/dev/null | while read f; do
        # TypeScript/JavaScript
        if grep -qE "class\s+[a-z]" "$f" 2>/dev/null; then
            echo "Model class should be PascalCase: $f"
        fi
        
        # Check for plural table names (convention varies)
        if grep -qE "tableName.*=.*['\"][A-Z]" "$f" 2>/dev/null; then
            echo "Check table name convention (usually lowercase plural): $f"
        fi
    done
    
    echo "$issues"
}
```

### 2. Query Pattern Validation

```bash
validate_query_patterns() {
    local dir="$1"
    local issues=""
    
    find "$dir" -name "*.ts" -o -name "*.js" -o -name "*.py" 2>/dev/null | while read f; do
        # Check for N+1 query patterns
        if grep -qE "\.find\(|\.findOne\(" "$f"; then
            # Look for loops around queries
            if grep -B5 "\.find\(" "$f" | grep -qE "for\s*\(|\.forEach\(|\.map\("; then
                echo "⚠️ Potential N+1 query pattern: $f"
            fi
        fi
        
        # Check for missing select/projection (fetching all fields)
        if grep -qE "\.find\(\s*\)" "$f" || grep -qE "\.findAll\(\s*\)" "$f"; then
            if ! grep -qE "select:|attributes:|projection:" "$f"; then
                echo "Query without field selection (consider selecting only needed fields): $f"
            fi
        fi
        
        # Check for raw queries without parameterization
        if grep -qE "query\(.*\\\$\{|execute\(.*\\\$\{" "$f"; then
            echo "🚨 Possible SQL injection risk (use parameterized queries): $f"
        fi
    done
    
    echo "$issues"
}
```

### 3. Migration Pattern Validation

```bash
validate_migration_patterns() {
    local dir="$1"
    local issues=""
    
    # Find migration files
    MIGRATION_DIRS=$(find "$dir" -type d -name "migrations" -o -name "migrate" 2>/dev/null)
    
    for mig_dir in $MIGRATION_DIRS; do
        # Check migration naming convention (timestamp prefix)
        find "$mig_dir" -name "*.ts" -o -name "*.js" -o -name "*.sql" 2>/dev/null | while read f; do
            FILENAME=$(basename "$f")
            
            # Should start with timestamp or version number
            if ! echo "$FILENAME" | grep -qE "^[0-9]{8,}|^V[0-9]+|^[0-9]+_"; then
                echo "Migration missing timestamp/version prefix: $f"
            fi
        done
        
        # Check for down/rollback migrations
        MIGRATIONS=$(find "$mig_dir" -name "*.ts" -o -name "*.js" 2>/dev/null)
        for m in $MIGRATIONS; do
            if grep -q "async up\|def up\|exports.up" "$m"; then
                if ! grep -q "async down\|def down\|exports.down" "$m"; then
                    echo "Migration missing down/rollback method: $m"
                fi
            fi
        done
    done
    
    echo "$issues"
}
```

### 4. Data Validation Patterns

```bash
validate_data_validation() {
    local dir="$1"
    local issues=""
    
    find "$dir" -name "*.ts" -o -name "*.js" 2>/dev/null | while read f; do
        # Check for validation on models/schemas
        if grep -qE "class.*Model|Schema\(|@Entity" "$f"; then
            # Should have some form of validation
            if ! grep -qE "@IsString|@IsNumber|validate|validator|required:|type:" "$f"; then
                echo "Model/schema without visible validation: $f"
            fi
        fi
        
        # Check for sanitization on user input
        if grep -qE "req\.body|req\.params|req\.query" "$f"; then
            if ! grep -qE "sanitize|escape|trim|validate" "$f"; then
                echo "⚠️ User input without visible sanitization: $f"
            fi
        fi
    done
    
    echo "$issues"
}
```

### 5. Index and Performance Patterns

```bash
validate_performance_patterns() {
    local dir="$1"
    local issues=""
    
    find "$dir" -name "*.ts" -o -name "*.js" 2>/dev/null | while read f; do
        # Check for indexed fields on frequently queried columns
        if grep -qE "\.find\(\s*\{.*:" "$f"; then
            # Extract query fields
            QUERY_FIELDS=$(grep -oE "\.find\(\s*\{[^}]+" "$f" | grep -oE "[a-zA-Z_]+:" | tr -d ':')
            
            # Check if these fields are indexed in schema
            for field in $QUERY_FIELDS; do
                if ! grep -qE "index.*$field|$field.*index" "$f"; then
                    echo "ℹ️ Frequently queried field may need index: $field in $f"
                fi
            done
        fi
        
        # Check for pagination on list queries
        if grep -qE "\.find\(\s*\{\s*\}\s*\)" "$f"; then
            if ! grep -qE "limit|skip|offset|page" "$f"; then
                echo "⚠️ Query without pagination (could return too many results): $f"
            fi
        fi
    done
    
    echo "$issues"
}
```

## Main Validation Orchestration

```bash
run_data_validation() {
    local target_dir="${1:-.}"
    local results_file="agent-os/output/validation/data-validation-results.md"
    
    mkdir -p agent-os/output/validation
    
    echo "# Data Pattern Validation Results" > "$results_file"
    echo "" >> "$results_file"
    echo "**Timestamp:** $(date)" >> "$results_file"
    echo "**Target:** $target_dir" >> "$results_file"
    echo "" >> "$results_file"
    
    # Run all data validations
    echo "## Model Naming" >> "$results_file"
    NAMING_ISSUES=$(validate_model_naming "$target_dir")
    if [ -n "$NAMING_ISSUES" ]; then
        echo "$NAMING_ISSUES" >> "$results_file"
        VALIDATION_PASSED=false
    else
        echo "✅ Model naming follows conventions" >> "$results_file"
    fi
    
    echo "" >> "$results_file"
    echo "## Query Patterns" >> "$results_file"
    QUERY_ISSUES=$(validate_query_patterns "$target_dir")
    if [ -n "$QUERY_ISSUES" ]; then
        echo "$QUERY_ISSUES" >> "$results_file"
    else
        echo "✅ Query patterns look good" >> "$results_file"
    fi
    
    echo "" >> "$results_file"
    echo "## Migration Patterns" >> "$results_file"
    MIGRATION_ISSUES=$(validate_migration_patterns "$target_dir")
    if [ -n "$MIGRATION_ISSUES" ]; then
        echo "$MIGRATION_ISSUES" >> "$results_file"
        VALIDATION_PASSED=false
    else
        echo "✅ Migrations follow conventions" >> "$results_file"
    fi
    
    echo "" >> "$results_file"
    echo "## Data Validation" >> "$results_file"
    VALIDATION_ISSUES=$(validate_data_validation "$target_dir")
    if [ -n "$VALIDATION_ISSUES" ]; then
        echo "$VALIDATION_ISSUES" >> "$results_file"
    else
        echo "✅ Data validation in place" >> "$results_file"
    fi
    
    echo "" >> "$results_file"
    echo "## Performance Patterns" >> "$results_file"
    PERF_ISSUES=$(validate_performance_patterns "$target_dir")
    if [ -n "$PERF_ISSUES" ]; then
        echo "$PERF_ISSUES" >> "$results_file"
        echo "ℹ️ Performance suggestions (review recommended)" >> "$results_file"
    else
        echo "✅ Performance patterns look good" >> "$results_file"
    fi
    
    # Summary
    echo "" >> "$results_file"
    echo "---" >> "$results_file"
    if [ "$VALIDATION_PASSED" = true ]; then
        echo "## ✅ Data Validation Passed" >> "$results_file"
        exit 0
    else
        echo "## ❌ Data Validation Failed" >> "$results_file"
        echo "See issues above for details." >> "$results_file"
        exit 1
    fi
}

# Run validation
run_data_validation "$TARGET_DIR"
```

## Integration with Layer Specialists

When `data-specialist` completes implementation, this validation runs automatically. The validation loads patterns from:
- `agent-os/basepoints/modules/` (data-related modules)
- `agent-os/product/tech-stack.md` (ORM/database framework rules)
- `agent-os/standards/global/conventions.md`
