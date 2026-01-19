Now that cleanup operations are complete, verify that sufficient knowledge has been extracted and no important information is missing by following these instructions:

## Core Responsibilities

1. **Verify Basepoints Completeness**: Check that all important modules and abstraction layers are covered
2. **Verify Product Knowledge Completeness**: Ensure all required product information is present and complete
3. **Detect Missing Information**: Identify gaps in extracted knowledge
4. **Coverage Analysis**: Verify knowledge covers the project structure adequately
5. **Generate Knowledge Gap Report**: Document any missing information or incomplete knowledge

## Workflow

### Step 1: Verify Basepoints Completeness

Check that basepoints cover all important aspects of the codebase:

```bash
# Create knowledge verification cache
CLEANUP_CACHE="geist/.cleanup-cache"
KNOWLEDGE_VERIFICATION_CACHE="$CLEANUP_CACHE/knowledge-verification"
mkdir -p "$KNOWLEDGE_VERIFICATION_CACHE"

echo "🔍 Verifying basepoints completeness..."

# Check basepoints structure
if [ ! -d "geist/basepoints" ]; then
    echo "❌ Basepoints directory not found"
    MISSING_BASEPOINTS=true
else
    echo "✅ Basepoints directory found"
    MISSING_BASEPOINTS=false
    
    # Count basepoint files
    BASEPOINT_FILES=$(find geist/basepoints -name "agent-base-*.md" -type f | wc -l | tr -d ' ')
    echo "  - Module basepoints found: $BASEPOINT_FILES"
    
    # Check for headquarter.md
    if [ -f "geist/basepoints/headquarter.md" ]; then
        echo "  ✅ Headquarter basepoint found"
        
        # Verify headquarter contains required sections
        REQUIRED_SECTIONS=("Pattern" "Standard" "Flow" "Strategy" "Architecture" "Abstraction" "Testing")
        MISSING_SECTIONS=()
        
        for section in "${REQUIRED_SECTIONS[@]}"; do
            if ! grep -qi "$section" geist/basepoints/headquarter.md 2>/dev/null; then
                MISSING_SECTIONS+=("$section")
            fi
        done
        
        if [ ${#MISSING_SECTIONS[@]} -gt 0 ]; then
            echo "  ⚠️  Warning: Headquarter missing sections: ${MISSING_SECTIONS[*]}"
        else
            echo "  ✅ Headquarter contains all required sections"
        fi
    else
        echo "  ❌ Headquarter basepoint missing"
        MISSING_BASEPOINTS=true
    fi
    
    # Analyze basepoint coverage
    if [ "$BASEPOINT_FILES" -lt 3 ]; then
        echo "  ⚠️  Warning: Only $BASEPOINT_FILES basepoint file(s) found. Consider if all important modules are covered."
    fi
    
    # Check for basepoints in different abstraction layers
    LAYER_COUNT=$(find geist/basepoints -type d -mindepth 1 -maxdepth 1 | wc -l | tr -d ' ')
    if [ "$LAYER_COUNT" -eq 0 ] && [ "$BASEPOINT_FILES" -gt 0 ]; then
        echo "  ⚠️  Warning: Basepoints may not be organized by abstraction layers"
    else
        echo "  ✅ Basepoints organized across $LAYER_COUNT abstraction layer(s)"
    fi
fi
```

### Step 2: Verify Product Knowledge Completeness

Check that product files contain all required information:

```bash
echo ""
echo "🔍 Verifying product knowledge completeness..."

if [ ! -d "geist/product" ]; then
    echo "❌ Product directory not found"
    MISSING_PRODUCT=true
else
    echo "✅ Product directory found"
    MISSING_PRODUCT=false
    
    # Check each required product file
    PRODUCT_FILES=("mission.md" "roadmap.md" "tech-stack.md")
    MISSING_PRODUCT_FILES=()
    INCOMPLETE_PRODUCT_FILES=()
    
    for file in "${PRODUCT_FILES[@]}"; do
        if [ ! -f "geist/product/$file" ]; then
            MISSING_PRODUCT_FILES+=("$file")
            echo "  ❌ Missing: geist/product/$file"
        else
            # Check if file has substantial content (more than just headers)
            CONTENT_LINES=$(grep -v '^#' geist/product/$file | grep -v '^$' | wc -l | tr -d ' ')
            if [ "$CONTENT_LINES" -lt 5 ]; then
                INCOMPLETE_PRODUCT_FILES+=("$file")
                echo "  ⚠️  Warning: geist/product/$file exists but may be incomplete ($CONTENT_LINES content lines)"
            else
                echo "  ✅ geist/product/$file (complete, $CONTENT_LINES content lines)"
            fi
        fi
    done
    
    # Verify mission.md contains key elements
    if [ -f "geist/product/mission.md" ]; then
        MISSION_KEYWORDS=("purpose" "vision" "goal" "objective" "target" "user")
        FOUND_KEYWORDS=0
        for keyword in "${MISSION_KEYWORDS[@]}"; do
            if grep -qi "$keyword" geist/product/mission.md 2>/dev/null; then
                ((FOUND_KEYWORDS++))
            fi
        done
        if [ $FOUND_KEYWORDS -lt 2 ]; then
            echo "  ⚠️  Warning: mission.md may be missing key elements (purpose, vision, goals, target users)"
        fi
    fi
    
    # Verify roadmap.md contains phases or features
    if [ -f "geist/product/roadmap.md" ]; then
        if ! grep -qiE "(phase|feature|milestone|roadmap|plan)" geist/product/roadmap.md 2>/dev/null; then
            echo "  ⚠️  Warning: roadmap.md may not contain development phases or features"
        fi
    fi
    
    # Verify tech-stack.md contains technology information
    if [ -f "geist/product/tech-stack.md" ]; then
        TECH_KEYWORDS=("language" "framework" "library" "database" "tool" "stack")
        FOUND_TECH=0
        for keyword in "${TECH_KEYWORDS[@]}"; do
            if grep -qi "$keyword" geist/product/tech-stack.md 2>/dev/null; then
                ((FOUND_TECH++))
            fi
        done
        if [ $FOUND_TECH -lt 2 ]; then
            echo "  ⚠️  Warning: tech-stack.md may be missing technology details"
        fi
    fi
fi
```

### Step 3: Detect Missing Information

Analyze the codebase structure to identify potential gaps:

```bash
echo ""
echo "🔍 Detecting missing information..."

# Analyze project structure to identify potential missing basepoints
if [ -d "geist/basepoints" ] && [ -d "." ]; then
    # Get project source directories (common patterns)
    SOURCE_DIRS=()
    [ -d "src" ] && SOURCE_DIRS+=("src")
    [ -d "lib" ] && SOURCE_DIRS+=("lib")
    [ -d "app" ] && SOURCE_DIRS+=("app")
    [ -d "components" ] && SOURCE_DIRS+=("components")
    [ -d "services" ] && SOURCE_DIRS+=("services")
    [ -d "models" ] && SOURCE_DIRS+=("models")
    [ -d "controllers" ] && SOURCE_DIRS+=("controllers")
    
    if [ ${#SOURCE_DIRS[@]} -gt 0 ]; then
        echo "  📁 Project structure analysis:"
        for dir in "${SOURCE_DIRS[@]}"; do
            # Check if there's a corresponding basepoint
            BASEPOINT_FOUND=false
            if find geist/basepoints -path "*$dir*" -name "agent-base-*.md" | grep -q .; then
                BASEPOINT_FOUND=true
            fi
            
            if [ "$BASEPOINT_FOUND" = false ]; then
                echo "    ⚠️  Warning: No basepoint found for $dir/ directory"
                echo "$dir" >> "$KNOWLEDGE_VERIFICATION_CACHE/potentially-missing-basepoints.txt"
            else
                echo "    ✅ Basepoint found for $dir/"
            fi
        done
    fi
fi

# Check for common patterns that should be documented
COMMON_PATTERNS=("api" "database" "auth" "test" "config" "util")
MISSING_PATTERNS=()

for pattern in "${COMMON_PATTERNS[@]}"; do
    # Check if pattern exists in codebase
    if find . -type f -name "*$pattern*" -not -path "./geist/*" -not -path "./.git/*" | head -1 | grep -q .; then
        # Check if pattern is documented in basepoints
        if ! find geist/basepoints -name "*.md" -exec grep -l -i "$pattern" {} \; | grep -q .; then
            MISSING_PATTERNS+=("$pattern")
        fi
    fi
done

if [ ${#MISSING_PATTERNS[@]} -gt 0 ]; then
    echo "  ⚠️  Warning: These patterns exist in codebase but may not be documented: ${MISSING_PATTERNS[*]}"
fi
```

### Step 4: Coverage Analysis

Verify knowledge covers the project structure adequately:

```bash
echo ""
echo "🔍 Analyzing knowledge coverage..."

# Calculate coverage metrics
TOTAL_BASEPOINTS=$BASEPOINT_FILES
if [ -f "geist/basepoints/headquarter.md" ]; then
    TOTAL_BASEPOINTS=$((TOTAL_BASEPOINTS + 1))
fi

# Estimate project complexity (rough heuristic)
PROJECT_FILES=$(find . -type f -name "*.js" -o -name "*.ts" -o -name "*.py" -o -name "*.rb" -o -name "*.java" -o -name "*.go" | grep -v node_modules | grep -v ".git" | grep -v "geist" | wc -l | tr -d ' ')

if [ "$PROJECT_FILES" -gt 0 ]; then
    COVERAGE_RATIO=$(echo "scale=2; $TOTAL_BASEPOINTS * 100 / ($PROJECT_FILES / 10 + 1)" | bc 2>/dev/null || echo "0")
    
    echo "  📊 Coverage Analysis:"
    echo "    - Project files: $PROJECT_FILES"
    echo "    - Basepoint files: $TOTAL_BASEPOINTS"
    
    if [ "$(echo "$COVERAGE_RATIO" | cut -d. -f1)" -lt 5 ]; then
        echo "    ⚠️  Warning: Basepoint coverage may be low for project size"
        echo "    💡 Consider running /create-basepoints again to ensure all modules are covered"
    else
        echo "    ✅ Basepoint coverage appears adequate"
    fi
fi
```

### Step 5: Generate Knowledge Gap Report

Document any missing information or incomplete knowledge:

```bash
echo ""
echo "📝 Generating knowledge gap report..."

# Create knowledge gap report
cat > "$KNOWLEDGE_VERIFICATION_CACHE/knowledge-gap-report.json" << EOF
{
  "verification_timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "basepoints": {
    "exists": $([ "$MISSING_BASEPOINTS" = "false" ] && echo "true" || echo "false"),
    "file_count": $BASEPOINT_FILES,
    "headquarter_exists": $([ -f "geist/basepoints/headquarter.md" ] && echo "true" || echo "false"),
    "missing_sections": $(if [ ${#MISSING_SECTIONS[@]} -gt 0 ]; then echo "\"${MISSING_SECTIONS[*]}\""; else echo "[]"; fi),
    "layer_count": $LAYER_COUNT
  },
  "product": {
    "exists": $([ "$MISSING_PRODUCT" = "false" ] && echo "true" || echo "false"),
    "missing_files": $(if [ ${#MISSING_PRODUCT_FILES[@]} -gt 0 ]; then echo "\"${MISSING_PRODUCT_FILES[*]}\""; else echo "[]"; fi),
    "incomplete_files": $(if [ ${#INCOMPLETE_PRODUCT_FILES[@]} -gt 0 ]; then echo "\"${INCOMPLETE_PRODUCT_FILES[*]}\""; else echo "[]"; fi)
  },
  "coverage": {
    "total_basepoints": $TOTAL_BASEPOINTS,
    "project_files": $PROJECT_FILES,
    "coverage_ratio": "$COVERAGE_RATIO"
  },
  "recommendations": []
}
EOF

# Generate markdown summary
cat > "$KNOWLEDGE_VERIFICATION_CACHE/knowledge-gap-report.md" << EOF
# Knowledge Completeness Verification Report

**Verification Timestamp:** $(date -u +%Y-%m-%dT%H:%M:%SZ)

## Summary

$(if [ "$MISSING_BASEPOINTS" = "true" ] || [ "$MISSING_PRODUCT" = "true" ]; then
    echo "❌ **CRITICAL**: Missing required knowledge files"
else
    echo "✅ All required knowledge files present"
fi)

## Basepoints Analysis

- **Basepoints Directory:** $([ "$MISSING_BASEPOINTS" = "false" ] && echo "✅ Found" || echo "❌ Missing")
- **Headquarter Basepoint:** $([ -f "geist/basepoints/headquarter.md" ] && echo "✅ Found" || echo "❌ Missing")
- **Module Basepoints:** $BASEPOINT_FILES file(s) found
- **Abstraction Layers:** $LAYER_COUNT layer(s) covered
$(if [ ${#MISSING_SECTIONS[@]} -gt 0 ]; then
    echo "- **Missing Sections:** ⚠️ ${MISSING_SECTIONS[*]}"
fi)

## Product Knowledge Analysis

- **Product Directory:** $([ "$MISSING_PRODUCT" = "false" ] && echo "✅ Found" || echo "❌ Missing")
$(if [ ${#MISSING_PRODUCT_FILES[@]} -gt 0 ]; then
    echo "- **Missing Files:** ❌ ${MISSING_PRODUCT_FILES[*]}"
fi)
$(if [ ${#INCOMPLETE_PRODUCT_FILES[@]} -gt 0 ]; then
    echo "- **Incomplete Files:** ⚠️ ${INCOMPLETE_PRODUCT_FILES[*]}"
fi)

## Coverage Analysis

- **Total Basepoints:** $TOTAL_BASEPOINTS
- **Project Files:** $PROJECT_FILES
- **Coverage Ratio:** $COVERAGE_RATIO

## Recommendations

$(if [ "$MISSING_BASEPOINTS" = "true" ]; then
    echo "1. ❌ **CRITICAL**: Run \`/create-basepoints\` to generate basepoint documentation"
fi)
$(if [ "$MISSING_PRODUCT" = "true" ]; then
    echo "2. ❌ **CRITICAL**: Run \`/plan-product\` or \`/adapt-to-product\` to create product documentation"
fi)
$(if [ ${#MISSING_SECTIONS[@]} -gt 0 ]; then
    echo "3. ⚠️  Consider updating headquarter.md to include: ${MISSING_SECTIONS[*]}"
fi)
$(if [ "$BASEPOINT_FILES" -lt 3 ]; then
    echo "4. ⚠️  Consider running \`/create-basepoints\` again to ensure all important modules are covered"
fi)
$(if [ "$(echo "$COVERAGE_RATIO" | cut -d. -f1)" -lt 5 ]; then
    echo "5. ⚠️  Basepoint coverage may be low. Review project structure and ensure all abstraction layers are documented"
fi)

## Next Steps

$(if [ "$MISSING_BASEPOINTS" = "true" ] || [ "$MISSING_PRODUCT" = "true" ]; then
    echo "⚠️  **Action Required**: Missing knowledge files detected. Please create missing prerequisites before using specialized commands."
    echo ""
    echo "To fix:"
    [ "$MISSING_BASEPOINTS" = "true" ] && echo "  - Run \`/create-basepoints\`"
    [ "$MISSING_PRODUCT" = "true" ] && echo "  - Run \`/plan-product\` or \`/adapt-to-product\`"
else
    echo "✅ Knowledge extraction appears complete. You can proceed with using specialized commands."
fi)
EOF

# Display summary
cat "$KNOWLEDGE_VERIFICATION_CACHE/knowledge-gap-report.md"
```

### Step 6: Store Verification Results

Store verification results for the cleanup report:

```bash
# Store verification status
if [ "$MISSING_BASEPOINTS" = "true" ] || [ "$MISSING_PRODUCT" = "true" ]; then
    echo "CRITICAL" > "$KNOWLEDGE_VERIFICATION_CACHE/verification-status.txt"
    echo "⚠️  CRITICAL: Missing required knowledge files detected"
elif [ ${#MISSING_SECTIONS[@]} -gt 0 ] || [ ${#INCOMPLETE_PRODUCT_FILES[@]} -gt 0 ]; then
    echo "WARNING" > "$KNOWLEDGE_VERIFICATION_CACHE/verification-status.txt"
    echo "⚠️  WARNING: Some knowledge may be incomplete"
else
    echo "OK" > "$KNOWLEDGE_VERIFICATION_CACHE/verification-status.txt"
    echo "✅ Knowledge verification passed"
fi
```

## Important Constraints

- Must verify basepoints completeness (headquarter + module basepoints)
- Must verify product knowledge completeness (mission, roadmap, tech-stack)
- Must detect missing information by analyzing project structure
- Must perform coverage analysis to ensure adequate knowledge extraction
- Must generate comprehensive knowledge gap report
- **CRITICAL**: All verification reports must be stored in `geist/.cleanup-cache/knowledge-verification/`
- Must provide clear recommendations for fixing missing or incomplete knowledge
- Must identify if critical knowledge is missing (basepoints or product files) vs. incomplete knowledge (missing sections)
