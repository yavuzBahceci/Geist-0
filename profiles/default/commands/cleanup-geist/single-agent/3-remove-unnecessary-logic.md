Now that placeholders are cleaned, proceed with removing unnecessary logic by following these instructions:

## Core Responsibilities

1. **Load Unnecessary Logic Detection Results**: Load unnecessary logic validation results
2. **Remove Project-Agnostic Conditionals**: Remove conditional logic that checks for project-agnostic scenarios ({{IF}}/{{UNLESS}} blocks that are no longer needed)
3. **Convert Generic Patterns to Project-Specific**: Replace generic examples with project-specific ones from basepoints where possible
4. **Abstract Redundant Patterns**: Simplify patterns that can be abstracted without losing functionality
5. **Remove Unnecessary Complexity**: Remove abstraction layers that are no longer needed after specialization
6. **Remove profiles/default References**: Remove all references to "profiles/default" from specialized commands
7. **Support Dry-Run Mode**: Preview changes before applying
8. **Track Changes**: Track what was converted, abstracted, or removed for cleanup report

## Workflow

### Step 1: Load Unnecessary Logic Detection Results

Load unnecessary logic validation results:

```bash
# Load validation cache
VALIDATION_CACHE="geist/.cleanup-cache/validation"
CLEANUP_CACHE="geist/.cleanup-cache"
DRY_RUN="${DRY_RUN:-false}"

# Load unnecessary logic detection results
if [ -f "$VALIDATION_CACHE/unnecessary-logic-removal-validation.json" ]; then
    UNNECESSARY_LOGIC_DATA=$(cat "$VALIDATION_CACHE/unnecessary-logic-removal-validation.json")
    UNNECESSARY_LOGIC_TOTAL=$(echo "$UNNECESSARY_LOGIC_DATA" | grep -o '"total_issues_found":[0-9]*' | cut -d: -f2)
    
    if [ "$UNNECESSARY_LOGIC_TOTAL" -eq 0 ]; then
        echo "✅ No unnecessary logic to remove"
        echo "0" > "$CLEANUP_CACHE/unnecessary-logic-removed.txt"
        exit 0
    fi
    
    echo "🔧 Found $UNNECESSARY_LOGIC_TOTAL unnecessary logic issue(s) to remove"
else
    echo "❌ Unnecessary logic validation results not found"
    exit 1
fi
```

### Step 2: Remove Project-Agnostic Conditionals

Remove conditional logic that checks for project-agnostic scenarios:

```bash
# Initialize cleanup tracking
UNNECESSARY_LOGIC_REMOVED=0
FILES_MODIFIED=0

# Extract files with unnecessary logic
echo "$UNNECESSARY_LOGIC_DATA" | grep -oE '"file": "[^"]+"' | cut -d'"' -f4 | sort -u > "$CLEANUP_CACHE/files-with-unnecessary-logic.txt"

# Process each file
while read file_path; do
    if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
        continue
    fi
    
    FILE_CONTENT=$(cat "$file_path")
    ORIGINAL_CONTENT="$FILE_CONTENT"
    MODIFIED=false
    
    # Remove technology-agnostic fallback logic
    if echo "$FILE_CONTENT" | grep -qE "if.*technology.*agnostic|if.*project.*agnostic|fallback.*generic|default.*generic"; then
        # Remove lines with technology-agnostic checks (simplified - in practice would need more sophisticated parsing)
        FILE_CONTENT=$(echo "$FILE_CONTENT" | grep -vE "if.*technology.*agnostic|if.*project.*agnostic|fallback.*generic|default.*generic")
        MODIFIED=true
        UNNECESSARY_LOGIC_REMOVED=$((UNNECESSARY_LOGIC_REMOVED + 1))
    fi
    
    # Remove conditional logic checking for template state
    if echo "$FILE_CONTENT" | grep -qE "if.*template|if.*abstract|if.*generic.*pattern"; then
        FILE_CONTENT=$(echo "$FILE_CONTENT" | grep -vE "if.*template|if.*abstract|if.*generic.*pattern")
        MODIFIED=true
        UNNECESSARY_LOGIC_REMOVED=$((UNNECESSARY_LOGIC_REMOVED + 1))
    fi
    
    # Apply changes if modified
    if [ "$MODIFIED" = "true" ]; then
        if [ "$DRY_RUN" = "true" ]; then
            echo "🔍 [DRY-RUN] Would modify: $file_path"
            echo "   Changes preview available in: $CLEANUP_CACHE/dry-run-preview/$file_path"
            mkdir -p "$(dirname "$CLEANUP_CACHE/dry-run-preview/$file_path")"
            echo "$FILE_CONTENT" > "$CLEANUP_CACHE/dry-run-preview/$file_path"
        else
            echo "$FILE_CONTENT" > "$file_path"
            echo "✅ Modified: $file_path"
            FILES_MODIFIED=$((FILES_MODIFIED + 1))
        fi
    fi
done < "$CLEANUP_CACHE/files-with-unnecessary-logic.txt"
```

### Step 3: Remove Unnecessary {{IF}}/{{UNLESS}} Conditional Blocks

After specialization, many conditional blocks are no longer needed. Remove resolved conditionals:

```bash
# Initialize tracking
CONDITIONALS_REMOVED=0

# Scan all geist command files for {{IF}}/{{UNLESS}} blocks
FILES_TO_SCAN=$(find geist/commands geist/workflows -name "*.md" -type f 2>/dev/null)

echo "$FILES_TO_SCAN" | while read file_path; do
    if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
        continue
    fi
    
    FILE_CONTENT=$(cat "$file_path")
    ORIGINAL_CONTENT="$FILE_CONTENT"
    MODIFIED=false
    
    # Detect {{IF}}/{{UNLESS}} blocks that are always true/false after specialization
    # Pattern: {{IF use_claude_code_subagents}}...{{ENDIF use_claude_code_subagents}}
    # After specialization, these flags are resolved - remove the conditionals, keep the content if true
    
    # For {{IF}} blocks: If condition is always true after specialization, remove tags, keep content
    if echo "$FILE_CONTENT" | grep -qE '\{\{IF[[:space:]]+[a-z_]+\}\}'; then
        # Extract project profile to determine resolved flags
        PROJECT_PROFILE=$(cat geist/config/project-profile.yml 2>/dev/null || echo "")
        
        # Check each conditional flag
        for flag in use_claude_code_subagents standards_as_claude_code_skills compiled_single_command; do
            # Determine if flag is true in project profile
            FLAG_VALUE="false"
            if echo "$PROJECT_PROFILE" | grep -qE "^[[:space:]]*${flag}:[[:space:]]*true"; then
                FLAG_VALUE="true"
            fi
            
            # If {{IF flag}} is always true, remove tags and keep content
            if [ "$FLAG_VALUE" = "true" ]; then
                # Remove {{IF flag}} and {{ENDIF flag}} tags, keep content
                FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "s/{{IF ${flag}}}/ /g" | sed "s/{{ENDIF ${flag}}}/ /g")
                MODIFIED=true
                CONDITIONALS_REMOVED=$((CONDITIONALS_REMOVED + 1))
            else
                # If {{IF flag}} is always false, remove entire block including content
                # This requires more sophisticated parsing (simplified here)
                FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "/{{IF ${flag}}}/,/{{ENDIF ${flag}}}/d")
                MODIFIED=true
                CONDITIONALS_REMOVED=$((CONDITIONALS_REMOVED + 1))
            fi
            
            # Handle {{UNLESS flag}} (opposite logic)
            if [ "$FLAG_VALUE" = "false" ]; then
                FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "s/{{UNLESS ${flag}}}/ /g" | sed "s/{{ENDUNLESS ${flag}}}/ /g")
                MODIFIED=true
                CONDITIONALS_REMOVED=$((CONDITIONALS_REMOVED + 1))
            else
                FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "/{{UNLESS ${flag}}}/,/{{ENDUNLESS ${flag}}}/d")
                MODIFIED=true
                CONDITIONALS_REMOVED=$((CONDITIONALS_REMOVED + 1))
            fi
        done
    fi
    
    # Apply changes if modified
    if [ "$MODIFIED" = "true" ]; then
        if [ "$DRY_RUN" = "true" ]; then
            echo "🔍 [DRY-RUN] Would remove resolved conditionals from: $file_path"
            mkdir -p "$(dirname "$CLEANUP_CACHE/dry-run-preview/$file_path")"
            echo "$FILE_CONTENT" > "$CLEANUP_CACHE/dry-run-preview/$file_path"
        else
            echo "$FILE_CONTENT" > "$file_path"
            echo "✅ Removed resolved conditionals from: $file_path"
            UNNECESSARY_LOGIC_REMOVED=$((UNNECESSARY_LOGIC_REMOVED + CONDITIONALS_REMOVED))
            FILES_MODIFIED=$((FILES_MODIFIED + 1))
        fi
    fi
done
```

### Step 4: Convert Generic Patterns to Project-Specific

Replace generic examples and patterns with project-specific ones from basepoints:

```bash
# Initialize tracking
PATTERNS_CONVERTED=0

# Load basepoints for project-specific patterns
HEADQUARTER=$(cat geist/basepoints/headquarter.md 2>/dev/null || echo "")
PROJECT_PROFILE=$(cat geist/config/project-profile.yml 2>/dev/null || echo "")

# Extract project-specific patterns from basepoints
if [ -n "$HEADQUARTER" ]; then
    # Extract module names, component patterns, model names from headquarter
    MODULE_NAMES=$(echo "$HEADQUARTER" | grep -oE "module|Module|module_name" | head -5 | tr '\n' ',' | sed 's/,$//')
    COMPONENT_PATTERNS=$(echo "$HEADQUARTER" | grep -oE "Component|component|component_name" | head -5 | tr '\n' ',' | sed 's/,$//')
    
    # Extract tech stack info
    TECH_STACK=$(echo "$PROJECT_PROFILE" | grep -E "^[[:space:]]*language:" | cut -d: -f2 | tr -d ' ' | head -1)
    FRAMEWORK=$(echo "$PROJECT_PROFILE" | grep -E "^[[:space:]]*framework:" | cut -d: -f2 | tr -d ' ' | head -1)
fi

# Scan for generic patterns in commands
FILES_TO_SCAN=$(find geist/commands geist/workflows -name "*.md" -type f 2>/dev/null)

echo "$FILES_TO_SCAN" | while read file_path; do
    if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
        continue
    fi
    
    FILE_CONTENT=$(cat "$file_path")
    ORIGINAL_CONTENT="$FILE_CONTENT"
    MODIFIED=false
    
    # Replace generic examples with project-specific ones
    # Example: "Example: User model" -> "Example: [actual model from basepoints]"
    if echo "$FILE_CONTENT" | grep -qE "Example:.*[Uu]ser.*model|Example:.*generic|Example:.*sample"; then
        # Replace generic examples (if we have project-specific alternatives)
        if [ -n "$MODULE_NAMES" ]; then
            FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "s/Example: User model/Example: Project model/g" | \
                sed "s/Example: generic/Example: $TECH_STACK/g" | \
                sed "s/Example: sample/Example: project-specific/g")
            MODIFIED=true
            PATTERNS_CONVERTED=$((PATTERNS_CONVERTED + 1))
        fi
    fi
    
    # Replace "generic implementer" or "default" references with project-specific agents
    if echo "$FILE_CONTENT" | grep -qE "generic implementer|default implementer|fallback.*generic"; then
        # Replace with project-specific agent names from specialist registry
        if [ -f "geist/agents/specialists/registry.yml" ]; then
            # Extract first specialist as default
            FIRST_SPECIALIST=$(grep -E "^[[:space:]]*[a-z]+:" geist/agents/specialists/registry.yml 2>/dev/null | head -1 | cut -d: -f1 | tr -d ' ')
            if [ -n "$FIRST_SPECIALIST" ]; then
                FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "s/generic implementer/${FIRST_SPECIALIST}-specialist/g" | \
                    sed "s/default implementer/${FIRST_SPECIALIST}-specialist/g")
                MODIFIED=true
                PATTERNS_CONVERTED=$((PATTERNS_CONVERTED + 1))
            fi
        fi
    fi
    
    # Remove "mock data" or "placeholder example" references
    if echo "$FILE_CONTENT" | grep -qE "mock.*data|placeholder.*example|sample.*data"; then
        # Replace with instruction to use actual project patterns
        FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "s/mock data/actual project data from basepoints/g" | \
            sed "s/placeholder example/project-specific example from basepoints/g" | \
            sed "s/sample data/actual project data/g")
        MODIFIED=true
        PATTERNS_CONVERTED=$((PATTERNS_CONVERTED + 1))
    fi
    
    # Apply changes if modified
    if [ "$MODIFIED" = "true" ]; then
        if [ "$DRY_RUN" = "true" ]; then
            echo "🔍 [DRY-RUN] Would convert generic patterns in: $file_path"
            mkdir -p "$(dirname "$CLEANUP_CACHE/dry-run-preview/$file_path")"
            echo "$FILE_CONTENT" > "$CLEANUP_CACHE/dry-run-preview/$file_path"
        else
            echo "$FILE_CONTENT" > "$file_path"
            echo "✅ Converted generic patterns in: $file_path"
            UNNECESSARY_LOGIC_REMOVED=$((UNNECESSARY_LOGIC_REMOVED + PATTERNS_CONVERTED))
            FILES_MODIFIED=$((FILES_MODIFIED + 1))
        fi
    fi
done
```

### Step 5: Abstract Redundant Patterns

Simplify patterns that can be abstracted without losing functionality:

```bash
# Initialize tracking
PATTERNS_ABSTRACTED=0

# Scan for redundant patterns that can be simplified
FILES_TO_SCAN=$(find geist/commands geist/workflows -name "*.md" -type f 2>/dev/null)

echo "$FILES_TO_SCAN" | while read file_path; do
    if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
        continue
    fi
    
    FILE_CONTENT=$(cat "$file_path")
    ORIGINAL_CONTENT="$FILE_CONTENT"
    MODIFIED=false
    
    # Remove redundant abstraction layers
    # Example: "if basepoints exist, load basepoints" -> "load basepoints" (after specialization, basepoints always exist)
    if echo "$FILE_CONTENT" | grep -qE "if.*basepoints.*exist|if.*basepoints.*available"; then
        # After specialization, basepoints always exist - remove conditional checks
        FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "s/if.*basepoints.*exist/ /g" | \
            sed "s/if.*basepoints.*available/ /g" | \
            sed "s/Check if basepoints exist/Load basepoints/g")
        MODIFIED=true
        PATTERNS_ABSTRACTED=$((PATTERNS_ABSTRACTED + 1))
    fi
    
    # Remove "project-agnostic fallback" logic (no longer needed after specialization)
    if echo "$FILE_CONTENT" | grep -qE "project-agnostic.*fallback|technology-agnostic.*fallback|generic.*fallback"; then
        # Remove fallback logic blocks
        FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "/project-agnostic.*fallback/d" | \
            sed "/technology-agnostic.*fallback/d" | \
            sed "/generic.*fallback/d")
        MODIFIED=true
        PATTERNS_ABSTRACTED=$((PATTERNS_ABSTRACTED + 1))
    fi
    
    # Apply changes if modified
    if [ "$MODIFIED" = "true" ]; then
        if [ "$DRY_RUN" = "true" ]; then
            echo "🔍 [DRY-RUN] Would abstract redundant patterns in: $file_path"
            mkdir -p "$(dirname "$CLEANUP_CACHE/dry-run-preview/$file_path")"
            echo "$FILE_CONTENT" > "$CLEANUP_CACHE/dry-run-preview/$file_path"
        else
            echo "$FILE_CONTENT" > "$file_path"
            echo "✅ Abstracted redundant patterns in: $file_path"
            UNNECESSARY_LOGIC_REMOVED=$((UNNECESSARY_LOGIC_REMOVED + PATTERNS_ABSTRACTED))
            FILES_MODIFIED=$((FILES_MODIFIED + 1))
        fi
    fi
done
```

### Step 6: Remove profiles/default References

Remove all references to "profiles/default" from specialized commands:

```bash
# Find all files in geist/commands that reference profiles/default
FILES_WITH_PROFILES_DEFAULT=$(find geist/commands -name "*.md" -type f -exec grep -l "profiles/default\|profiles\.default\|@profiles/default" {} \; 2>/dev/null)

if [ -n "$FILES_WITH_PROFILES_DEFAULT" ]; then
    echo "$FILES_WITH_PROFILES_DEFAULT" | while read file_path; do
        if [ -z "$file_path" ] || [ ! -f "$file_path" ]; then
            continue
        fi
        
        FILE_CONTENT=$(cat "$file_path")
        ORIGINAL_CONTENT="$FILE_CONTENT"
        
        # Remove profiles/default references
        FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "s|profiles/default|geist|g")
        FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "s|profiles\.self|geist|g")
        FILE_CONTENT=$(echo "$FILE_CONTENT" | sed "s|@profiles/default|@geist|g")
        
        if [ "$FILE_CONTENT" != "$ORIGINAL_CONTENT" ]; then
            if [ "$DRY_RUN" = "true" ]; then
                echo "🔍 [DRY-RUN] Would remove profiles/default references from: $file_path"
                mkdir -p "$(dirname "$CLEANUP_CACHE/dry-run-preview/$file_path")"
                echo "$FILE_CONTENT" > "$CLEANUP_CACHE/dry-run-preview/$file_path"
            else
                echo "$FILE_CONTENT" > "$file_path"
                echo "✅ Removed profiles/default references from: $file_path"
                UNNECESSARY_LOGIC_REMOVED=$((UNNECESSARY_LOGIC_REMOVED + 1))
                FILES_MODIFIED=$((FILES_MODIFIED + 1))
            fi
        fi
    done
else
    echo "✅ No profiles/default references found"
fi

# Store cleanup results
echo "$UNNECESSARY_LOGIC_REMOVED" > "$CLEANUP_CACHE/unnecessary-logic-removed.txt"
echo "$FILES_MODIFIED" > "$CLEANUP_CACHE/files-modified-unnecessary-logic.txt"

if [ "$DRY_RUN" = "true" ]; then
    echo "🔍 [DRY-RUN] Would remove $UNNECESSARY_LOGIC_REMOVED unnecessary logic issue(s) in files"
else
    echo "✅ Removed $UNNECESSARY_LOGIC_REMOVED unnecessary logic issue(s) in $FILES_MODIFIED file(s)"
fi
```

## Important Constraints

- Must use unnecessary logic detection results from validation phase
- Must remove project-agnostic conditional logic (including {{IF}}/{{UNLESS}} blocks that are resolved)
- Must convert generic patterns to project-specific ones from basepoints where possible
- Must abstract redundant patterns that can be simplified without losing functionality
- Must remove unnecessary complexity (abstraction layers no longer needed after specialization)
- Must remove profiles/default references from specialized commands
- Must support dry-run mode to preview changes
- Must track what was converted, abstracted, or removed for cleanup report
- **CRITICAL**: All cleanup cache files must be stored in `geist/.cleanup-cache/` (temporary, cleaned up after cleanup completes)
- Must preserve functionality while simplifying - only remove what's truly unnecessary after specialization
