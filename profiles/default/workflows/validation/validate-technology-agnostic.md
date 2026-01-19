# Technology-Agnostic Validation Utility

## Core Responsibilities

1. **Verify profiles/default Technology-Agnostic**: Verify profiles/default commands are truly technology-agnostic
2. **Verify Specialized Commands Reference Project Structure**: Verify specialized commands in installed geist reference actual project structure
3. **Check for Technology Leaks**: Check for technology-specific assumptions leaking into profiles/default template
4. **Validate Transition**: Validate transition preserves functionality while adding specialization

## Workflow

### Step 1: Determine Context and Paths

Determine whether we're validating profiles/default (template) or installed geist (specialized):

```bash
# Determine spec path
SPEC_PATH="{{SPEC_PATH}}"

# Determine validation context
if [ -d "profiles/default" ] && [ "$(pwd)" = *"/profiles/default"* ]; then
    VALIDATION_CONTEXT="profiles/default"
    SCAN_PATH="profiles/default"
    CACHE_PATH="$SPEC_PATH/implementation/cache/validation"
else
    VALIDATION_CONTEXT="installed-geist"
    SCAN_PATH="geist"
    CACHE_PATH="$SPEC_PATH/implementation/cache/validation"
fi

# Create cache directory
mkdir -p "$CACHE_PATH"
```

### Step 2: Verify profiles/default Technology-Agnostic (if validating template)

If validating profiles/default template, check for technology-specific assumptions:

```bash
# Initialize results
TECHNOLOGY_LEAKS=""

if [ "$VALIDATION_CONTEXT" = "profiles/default" ]; then
    # Find files to scan
    FILES_TO_SCAN=$(find "$SCAN_PATH" -type f \( -name "*.md" \) ! -path "*/node_modules/*" ! -path "*/.git/*" ! -path "*/build/*" ! -path "*/dist/*" ! -path "*/cache/*")
    
    # Common technology-specific patterns to detect
    TECH_SPECIFIC_PATTERNS=(
        "React|Vue|Angular"
        "Express|FastAPI|Django|Rails"
        "TypeScript|JavaScript"
        "Python|Java|Go|Rust"
        "PostgreSQL|MySQL|MongoDB"
        "npm|yarn|pip|maven"
        "\.jsx|\.tsx|\.vue"
        "package\.json|requirements\.txt|pom\.xml"
    )
    
    # Scan for technology-specific assumptions
    echo "$FILES_TO_SCAN" | while read file_path; do
        if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
            continue
        fi
        
        FILE_CONTENT=$(cat "$file_path")
        
        for pattern in "${TECH_SPECIFIC_PATTERNS[@]}"; do
            if echo "$FILE_CONTENT" | grep -qiE "$pattern"; then
                LINE_NUMBERS=$(grep -niE "$pattern" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
                TECHNOLOGY_LEAKS="${TECHNOLOGY_LEAKS}\nleak:$file_path:$LINE_NUMBERS:Technology-specific assumption: $pattern"
            fi
        done
    done
fi
```

### Step 3: Verify Specialized Commands Reference Project Structure (if validating installed geist)

If validating installed geist, verify commands reference actual project structure:

```bash
# Initialize results
PROJECT_STRUCTURE_ISSUES=""

if [ "$VALIDATION_CONTEXT" = "installed-geist" ]; then
    # Check if basepoints exist (to verify project structure references)
    if [ -d "geist/basepoints" ] && [ -f "geist/basepoints/headquarter.md" ]; then
        # Extract actual project structure from basepoints
        ACTUAL_LAYERS=$(find geist/basepoints -type d -mindepth 1 -maxdepth 2 | sed 's|geist/basepoints/||' | cut -d'/' -f1 | sort -u)
        ACTUAL_MODULES=$(find geist/basepoints -name "agent-base-*.md" -type f | sed 's|geist/basepoints/||' | sed 's|/agent-base-.*\.md||')
        
        # Find files to scan
        FILES_TO_SCAN=$(find "$SCAN_PATH/commands" -type f \( -name "*.md" \) ! -path "*/node_modules/*" ! -path "*/.git/*" 2>/dev/null)
        
        # Check if commands reference actual project structure
        echo "$FILES_TO_SCAN" | while read file_path; do
            if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
                continue
            fi
            
            FILE_CONTENT=$(cat "$file_path")
            
            # Check for template path references (should be replaced with actual paths)
            if echo "$FILE_CONTENT" | grep -qE "profiles/default|template.*path|abstract.*path"; then
                LINE_NUMBERS=$(grep -nE "profiles/default|template.*path|abstract.*path" "$file_path" | cut -d: -f1 | tr '\n' ',' | sed 's/,$//')
                PROJECT_STRUCTURE_ISSUES="${PROJECT_STRUCTURE_ISSUES}\nissue:$file_path:$LINE_NUMBERS:Template path reference (should be actual project path)"
            fi
            
            # Check if commands reference actual project structure from basepoints
            # (This is a positive check - commands should reference actual structure)
            REFERENCED_LAYERS=""
            for layer in $ACTUAL_LAYERS; do
                if echo "$FILE_CONTENT" | grep -qi "$layer"; then
                    REFERENCED_LAYERS="${REFERENCED_LAYERS} $layer"
                fi
            done
            
            # If no actual layers are referenced, flag as potential issue
            if [ -z "$REFERENCED_LAYERS" ] && echo "$FILE_CONTENT" | grep -qE "layer|abstraction"; then
                LINE_NUMBERS=$(grep -nE "layer|abstraction" "$file_path" | head -1 | cut -d: -f1)
                PROJECT_STRUCTURE_ISSUES="${PROJECT_STRUCTURE_ISSUES}\nissue:$file_path:$LINE_NUMBERS:May not reference actual project structure from basepoints"
            fi
        done
    fi
fi
```

### Step 4: Validate Transition Preserves Functionality

Check that transition from template to specialized maintains functionality:

```bash
# Initialize results
FUNCTIONALITY_ISSUES=""

if [ "$VALIDATION_CONTEXT" = "installed-geist" ]; then
    # Check that specialized commands maintain same structure as template
    TEMPLATE_COMMANDS=$(find profiles/default/commands -name "*.md" -type f 2>/dev/null | wc -l | tr -d ' ')
    SPECIALIZED_COMMANDS=$(find geist/commands -name "*.md" -type f 2>/dev/null | wc -l | tr -d ' ')
    
    if [ "$TEMPLATE_COMMANDS" -gt 0 ] && [ "$SPECIALIZED_COMMANDS" -eq 0 ]; then
        FUNCTIONALITY_ISSUES="${FUNCTIONALITY_ISSUES}\nissue:No specialized commands found (expected $TEMPLATE_COMMANDS commands)"
    fi
    
    # Check that command structure is preserved
    TEMPLATE_STRUCTURE=$(find profiles/default/commands -type d | sort)
    SPECIALIZED_STRUCTURE=$(find geist/commands -type d 2>/dev/null | sort)
    
    # Compare structure (simplified check)
    if [ -n "$TEMPLATE_STRUCTURE" ] && [ -z "$SPECIALIZED_STRUCTURE" ]; then
        FUNCTIONALITY_ISSUES="${FUNCTIONALITY_ISSUES}\nissue:Command structure not preserved during specialization"
    fi
fi
```

### Step 5: Generate Validation Report

Generate comprehensive technology-agnostic validation report:

```bash
# Count findings
LEAKS_COUNT=$(echo "$TECHNOLOGY_LEAKS" | grep -c . || echo "0")
STRUCTURE_ISSUES_COUNT=$(echo "$PROJECT_STRUCTURE_ISSUES" | grep -c . || echo "0")
FUNCTIONALITY_ISSUES_COUNT=$(echo "$FUNCTIONALITY_ISSUES" | grep -c . || echo "0")
TOTAL_COUNT=$((LEAKS_COUNT + STRUCTURE_ISSUES_COUNT + FUNCTIONALITY_ISSUES_COUNT))

# Create JSON report
cat > "$CACHE_PATH/technology-agnostic-validation.json" << EOF
{
  "validation_context": "$VALIDATION_CONTEXT",
  "scan_path": "$SCAN_PATH",
  "scan_timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "total_issues_found": $TOTAL_COUNT,
  "categories": {
    "technology_leaks": {
      "count": $LEAKS_COUNT,
      "issues": [
$(echo "$TECHNOLOGY_LEAKS" | grep -v "^$" | while IFS=':' read -r type file_path line_numbers description; do
    echo "        {\"type\": \"$type\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ]
    },
    "project_structure_issues": {
      "count": $STRUCTURE_ISSUES_COUNT,
      "issues": [
$(echo "$PROJECT_STRUCTURE_ISSUES" | grep -v "^$" | while IFS=':' read -r type file_path line_numbers description; do
    echo "        {\"type\": \"$type\", \"file\": \"$file_path\", \"lines\": \"$line_numbers\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ]
    },
    "functionality_issues": {
      "count": $FUNCTIONALITY_ISSUES_COUNT,
      "issues": [
$(echo "$FUNCTIONALITY_ISSUES" | grep -v "^$" | while IFS=':' read -r type description; do
    echo "        {\"type\": \"$type\", \"description\": \"$description\"},"
done | sed '$ s/,$//')
      ]
    }
  }
}
EOF

# Create markdown summary report
cat > "$CACHE_PATH/technology-agnostic-validation-summary.md" << EOF
# Technology-Agnostic Validation Report

**Validation Context:** $VALIDATION_CONTEXT  
**Scan Path:** $SCAN_PATH  
**Scan Timestamp:** $(date -u +%Y-%m-%dT%H:%M:%SZ)

## Summary

- **Total Issues Found:** $TOTAL_COUNT
- **Technology Leaks (in template):** $LEAKS_COUNT
- **Project Structure Issues (in specialized):** $STRUCTURE_ISSUES_COUNT
- **Functionality Issues:** $FUNCTIONALITY_ISSUES_COUNT

## Technology Leaks (profiles/default template)

$(echo "$TECHNOLOGY_LEAKS" | grep -v "^$" | while IFS=':' read -r type file_path line_numbers description; do
    echo "- **$description**"
    echo "  - File: \`$file_path\`"
    echo "  - Lines: $line_numbers"
    echo "  - Recommendation: Remove technology-specific assumptions from template"
    echo ""
done)

## Project Structure Issues (installed geist)

$(echo "$PROJECT_STRUCTURE_ISSUES" | grep -v "^$" | while IFS=':' read -r type file_path line_numbers description; do
    echo "- **$description**"
    echo "  - File: \`$file_path\`"
    echo "  - Lines: $line_numbers"
    echo ""
done)

## Functionality Issues

$(echo "$FUNCTIONALITY_ISSUES" | grep -v "^$" | while IFS=':' read -r type description; do
    echo "- **$description**"
    echo ""
done)
EOF

echo "✅ Technology-agnostic validation complete. Report stored in $CACHE_PATH/"
```

## Important Constraints

- Must verify profiles/default commands are technology-agnostic (no tech stack assumptions)
- Must verify specialized commands reference actual project structure
- Must check for technology-specific assumptions leaking into profiles/default template
- Must validate transition preserves functionality while adding specialization
- **CRITICAL**: All reports must be stored in `geist/specs/[current-spec]/implementation/cache/validation/` when running within a spec command, not scattered around the codebase
- Must use placeholder syntax ({{PLACEHOLDER}}) for project-specific parts that will be replaced during deploy-agents
