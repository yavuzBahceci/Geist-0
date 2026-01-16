# Validate UI Patterns Workflow

This workflow validates that UI/frontend implementations follow established patterns from basepoints.

## Purpose

- Ensure UI components follow project conventions
- Validate styling patterns and naming
- Check accessibility and responsiveness patterns
- Verify component structure matches basepoints

## Validation Steps

```bash
#!/bin/bash

echo "🎨 Validating UI layer patterns..."

VALIDATION_PASSED=true
ISSUES=()

# Load UI patterns from basepoints
UI_BASEPOINTS=""
if [ -d "agent-os/basepoints/modules" ]; then
    UI_BASEPOINTS=$(find agent-os/basepoints/modules -name "*.md" -exec grep -l -i "ui\|component\|view\|frontend" {} \; 2>/dev/null)
fi

# Extract expected patterns
COMPONENT_PATTERN=""
STYLING_PATTERN=""
NAMING_PATTERN=""

if [ -n "$UI_BASEPOINTS" ]; then
    for bp in $UI_BASEPOINTS; do
        # Extract component patterns
        if grep -q "## Component" "$bp"; then
            COMPONENT_PATTERN=$(grep -A5 "## Component" "$bp" | head -5)
        fi
        # Extract styling patterns
        if grep -q "## Styling\|## Style" "$bp"; then
            STYLING_PATTERN=$(grep -A5 "## Styl" "$bp" | head -5)
        fi
    done
fi
```

## Validation Checks

### 1. Component Structure Validation

```bash
validate_component_structure() {
    local file="$1"
    local issues=""
    
    # Check for common UI patterns based on tech stack
    TECH_STACK=$(cat agent-os/product/tech-stack.md 2>/dev/null || echo "")
    
    if echo "$TECH_STACK" | grep -qi "react"; then
        # React-specific checks
        if grep -q "class.*extends.*Component" "$file"; then
            # Check for proper lifecycle methods
            if ! grep -q "componentDidMount\|useEffect" "$file"; then
                issues="${issues}Missing lifecycle/effect hooks in $file\n"
            fi
        fi
    fi
    
    if echo "$TECH_STACK" | grep -qi "vue"; then
        # Vue-specific checks
        if grep -q "<template>" "$file"; then
            if ! grep -q "<script" "$file"; then
                issues="${issues}Vue component missing script section: $file\n"
            fi
        fi
    fi
    
    if echo "$TECH_STACK" | grep -qi "swift\|ios"; then
        # SwiftUI/UIKit checks
        if grep -q "struct.*View" "$file"; then
            if ! grep -q "var body" "$file"; then
                issues="${issues}SwiftUI view missing body: $file\n"
            fi
        fi
    fi
    
    echo "$issues"
}
```

### 2. Naming Convention Validation

```bash
validate_ui_naming() {
    local dir="$1"
    local issues=""
    
    # Check component naming based on basepoints
    if [ -n "$NAMING_PATTERN" ]; then
        # Extract naming convention from basepoints
        EXPECTED_PATTERN=$(echo "$NAMING_PATTERN" | grep -o "PascalCase\|camelCase\|kebab-case\|snake_case" | head -1)
        
        case "$EXPECTED_PATTERN" in
            "PascalCase")
                # Check for non-PascalCase component names
                find "$dir" -name "*.tsx" -o -name "*.jsx" -o -name "*.vue" | while read f; do
                    FILENAME=$(basename "$f" | sed 's/\.[^.]*$//')
                    if ! echo "$FILENAME" | grep -qE "^[A-Z][a-zA-Z0-9]*$"; then
                        echo "Component should be PascalCase: $f"
                    fi
                done
                ;;
        esac
    fi
    
    echo "$issues"
}
```

### 3. Styling Pattern Validation

```bash
validate_styling_patterns() {
    local dir="$1"
    local issues=""
    
    # Detect styling approach from tech stack
    if grep -qi "tailwind" agent-os/product/tech-stack.md 2>/dev/null; then
        # Check for inline styles (should use Tailwind classes)
        INLINE_STYLES=$(find "$dir" -name "*.tsx" -o -name "*.jsx" | xargs grep -l "style={{" 2>/dev/null)
        if [ -n "$INLINE_STYLES" ]; then
            issues="${issues}Found inline styles (use Tailwind classes): $INLINE_STYLES\n"
        fi
    fi
    
    if grep -qi "styled-components\|emotion" agent-os/product/tech-stack.md 2>/dev/null; then
        # Check for proper styled component usage
        :
    fi
    
    echo "$issues"
}
```

### 4. Accessibility Validation

```bash
validate_accessibility() {
    local dir="$1"
    local issues=""
    
    # Check for basic accessibility patterns
    find "$dir" -name "*.tsx" -o -name "*.jsx" -o -name "*.vue" 2>/dev/null | while read f; do
        # Check for img without alt
        if grep -q "<img" "$f" && ! grep -q "alt=" "$f"; then
            echo "Missing alt attribute on images: $f"
        fi
        
        # Check for buttons without aria-label or text
        if grep -q "<button" "$f"; then
            if grep -q "<button[^>]*>" "$f" | grep -v "aria-label\|>.*<"; then
                echo "Button may need aria-label: $f"
            fi
        fi
    done
    
    echo "$issues"
}
```

## Main Validation Orchestration

```bash
run_ui_validation() {
    local target_dir="${1:-.}"
    local results_file="agent-os/output/validation/ui-validation-results.md"
    
    mkdir -p agent-os/output/validation
    
    echo "# UI Pattern Validation Results" > "$results_file"
    echo "" >> "$results_file"
    echo "**Timestamp:** $(date)" >> "$results_file"
    echo "**Target:** $target_dir" >> "$results_file"
    echo "" >> "$results_file"
    
    # Run all UI validations
    echo "## Component Structure" >> "$results_file"
    STRUCTURE_ISSUES=$(validate_component_structure "$target_dir")
    if [ -n "$STRUCTURE_ISSUES" ]; then
        echo "$STRUCTURE_ISSUES" >> "$results_file"
        VALIDATION_PASSED=false
    else
        echo "✅ All components follow expected structure" >> "$results_file"
    fi
    
    echo "" >> "$results_file"
    echo "## Naming Conventions" >> "$results_file"
    NAMING_ISSUES=$(validate_ui_naming "$target_dir")
    if [ -n "$NAMING_ISSUES" ]; then
        echo "$NAMING_ISSUES" >> "$results_file"
        VALIDATION_PASSED=false
    else
        echo "✅ All UI files follow naming conventions" >> "$results_file"
    fi
    
    echo "" >> "$results_file"
    echo "## Styling Patterns" >> "$results_file"
    STYLING_ISSUES=$(validate_styling_patterns "$target_dir")
    if [ -n "$STYLING_ISSUES" ]; then
        echo "$STYLING_ISSUES" >> "$results_file"
        VALIDATION_PASSED=false
    else
        echo "✅ Styling follows project patterns" >> "$results_file"
    fi
    
    echo "" >> "$results_file"
    echo "## Accessibility" >> "$results_file"
    A11Y_ISSUES=$(validate_accessibility "$target_dir")
    if [ -n "$A11Y_ISSUES" ]; then
        echo "$A11Y_ISSUES" >> "$results_file"
        echo "⚠️ Accessibility warnings (non-blocking)" >> "$results_file"
    else
        echo "✅ Basic accessibility checks passed" >> "$results_file"
    fi
    
    # Summary
    echo "" >> "$results_file"
    echo "---" >> "$results_file"
    if [ "$VALIDATION_PASSED" = true ]; then
        echo "## ✅ UI Validation Passed" >> "$results_file"
        exit 0
    else
        echo "## ❌ UI Validation Failed" >> "$results_file"
        echo "See issues above for details." >> "$results_file"
        exit 1
    fi
}

# Run validation
run_ui_validation "$TARGET_DIR"
```

## Integration with Layer Specialists

When `ui-specialist` completes implementation, this validation runs automatically to verify patterns were followed. The validation loads patterns from:
- `agent-os/basepoints/modules/` (UI-related modules)
- `agent-os/product/tech-stack.md` (framework-specific rules)
- `agent-os/standards/global/conventions.md` (naming conventions)
