# Product-Focused Cleanup Workflow

After product files (mission, tech-stack, roadmap) are created, this workflow cleans irrelevant content from agent-os files and enhances remaining content with product-specific knowledge.

## Core Responsibilities

1. **Detect Product Scope**: Load and parse product files to understand language, framework, project type, and architecture
2. **Simplify Files**: Remove irrelevant technology examples, workflows, and patterns not matching the product scope
3. **Expand Knowledge**: Enhance remaining content with product-specific examples and web-validated best practices
4. **Generate Report**: Create a transparent report of all changes made

## Workflow

### Step 1: Load Product Scope Detection Sources

Load the product definition files to understand the project scope:

```bash
echo "📋 Loading product scope detection sources..."

# Initialize scope variables
DETECTED_LANGUAGE=""
DETECTED_FRAMEWORKS=""
DETECTED_PROJECT_TYPE=""
DETECTED_ARCHITECTURE=""

# Load tech-stack.md for language and framework detection
if [ -f "agent-os/product/tech-stack.md" ]; then
    TECH_STACK_CONTENT=$(cat agent-os/product/tech-stack.md)
    echo "✅ Loaded tech-stack.md"
else
    echo "⚠️ tech-stack.md not found - scope detection limited"
fi

# Load mission.md for project type indicators
if [ -f "agent-os/product/mission.md" ]; then
    MISSION_CONTENT=$(cat agent-os/product/mission.md)
    echo "✅ Loaded mission.md"
else
    echo "⚠️ mission.md not found - project type detection limited"
fi

# Load roadmap.md for feature scope
if [ -f "agent-os/product/roadmap.md" ]; then
    ROADMAP_CONTENT=$(cat agent-os/product/roadmap.md)
    echo "✅ Loaded roadmap.md"
else
    echo "⚠️ roadmap.md not found - feature scope detection limited"
fi
```

### Step 2: Parse and Categorize Detected Scope

Analyze product files to categorize the project scope:

```bash
echo "🔍 Parsing and categorizing product scope..."

# Language Detection
detect_language() {
    local tech_stack="$1"
    local tech_lower=$(echo "$tech_stack" | tr '[:upper:]' '[:lower:]')
    
    # Check for common languages
    if echo "$tech_lower" | grep -qE "typescript|\.ts\b"; then
        echo "typescript"
    elif echo "$tech_lower" | grep -qE "javascript|\.js\b|node\.js|nodejs"; then
        echo "javascript"
    elif echo "$tech_lower" | grep -qE "python|\.py\b|django|flask|fastapi"; then
        echo "python"
    elif echo "$tech_lower" | grep -qE "rust|cargo|\.rs\b"; then
        echo "rust"
    elif echo "$tech_lower" | grep -qE "golang|go\b|\.go\b"; then
        echo "go"
    elif echo "$tech_lower" | grep -qE "ruby|rails|\.rb\b"; then
        echo "ruby"
    elif echo "$tech_lower" | grep -qE "java\b|spring|\.java\b"; then
        echo "java"
    elif echo "$tech_lower" | grep -qE "c#|csharp|\.net|dotnet"; then
        echo "csharp"
    elif echo "$tech_lower" | grep -qE "php|laravel|symfony"; then
        echo "php"
    elif echo "$tech_lower" | grep -qE "swift|ios|swiftui"; then
        echo "swift"
    elif echo "$tech_lower" | grep -qE "kotlin|android"; then
        echo "kotlin"
    else
        echo "unknown"
    fi
}

# Framework Detection
detect_frameworks() {
    local tech_stack="$1"
    local tech_lower=$(echo "$tech_stack" | tr '[:upper:]' '[:lower:]')
    local frameworks=""
    
    # Frontend frameworks
    echo "$tech_lower" | grep -qE "react|reactjs" && frameworks="$frameworks react"
    echo "$tech_lower" | grep -qE "vue|vuejs" && frameworks="$frameworks vue"
    echo "$tech_lower" | grep -qE "angular" && frameworks="$frameworks angular"
    echo "$tech_lower" | grep -qE "svelte" && frameworks="$frameworks svelte"
    echo "$tech_lower" | grep -qE "next\.js|nextjs" && frameworks="$frameworks nextjs"
    echo "$tech_lower" | grep -qE "nuxt" && frameworks="$frameworks nuxt"
    
    # Backend frameworks
    echo "$tech_lower" | grep -qE "express|expressjs" && frameworks="$frameworks express"
    echo "$tech_lower" | grep -qE "fastify" && frameworks="$frameworks fastify"
    echo "$tech_lower" | grep -qE "nest\.js|nestjs" && frameworks="$frameworks nestjs"
    echo "$tech_lower" | grep -qE "django" && frameworks="$frameworks django"
    echo "$tech_lower" | grep -qE "flask" && frameworks="$frameworks flask"
    echo "$tech_lower" | grep -qE "fastapi" && frameworks="$frameworks fastapi"
    echo "$tech_lower" | grep -qE "rails" && frameworks="$frameworks rails"
    echo "$tech_lower" | grep -qE "spring" && frameworks="$frameworks spring"
    echo "$tech_lower" | grep -qE "laravel" && frameworks="$frameworks laravel"
    
    # Mobile frameworks
    echo "$tech_lower" | grep -qE "react.native" && frameworks="$frameworks react-native"
    echo "$tech_lower" | grep -qE "flutter" && frameworks="$frameworks flutter"
    echo "$tech_lower" | grep -qE "expo" && frameworks="$frameworks expo"
    
    echo "$frameworks" | xargs
}

# Project Type Detection
detect_project_type() {
    local mission="$1"
    local roadmap="$2"
    local combined=$(echo "$mission $roadmap" | tr '[:upper:]' '[:lower:]')
    
    # Check for project type indicators
    if echo "$combined" | grep -qE "cli|command.line|terminal|console.app"; then
        echo "cli"
    elif echo "$combined" | grep -qE "api|rest|graphql|backend|server|microservice"; then
        echo "api"
    elif echo "$combined" | grep -qE "web.app|webapp|frontend|dashboard|portal|spa"; then
        echo "webapp"
    elif echo "$combined" | grep -qE "mobile|ios|android|app.store|play.store"; then
        echo "mobile"
    elif echo "$combined" | grep -qE "library|package|sdk|npm|pip|crate"; then
        echo "library"
    elif echo "$combined" | grep -qE "ai|ml|machine.learning|llm|agent|model"; then
        echo "ai-service"
    elif echo "$combined" | grep -qE "full.stack|fullstack"; then
        echo "fullstack"
    else
        echo "unknown"
    fi
}

# Architecture Detection
detect_architecture() {
    local tech_stack="$1"
    local roadmap="$2"
    local combined=$(echo "$tech_stack $roadmap" | tr '[:upper:]' '[:lower:]')
    
    if echo "$combined" | grep -qE "microservice|micro.service|distributed|kubernetes|k8s"; then
        echo "microservices"
    elif echo "$combined" | grep -qE "serverless|lambda|cloud.function|faas"; then
        echo "serverless"
    elif echo "$combined" | grep -qE "monolith|monolithic|single.app"; then
        echo "monolith"
    elif echo "$combined" | grep -qE "event.driven|event.sourcing|cqrs"; then
        echo "event-driven"
    else
        echo "monolith"  # Default to monolith
    fi
}

# Run detection
DETECTED_LANGUAGE=$(detect_language "$TECH_STACK_CONTENT")
DETECTED_FRAMEWORKS=$(detect_frameworks "$TECH_STACK_CONTENT")
DETECTED_PROJECT_TYPE=$(detect_project_type "$MISSION_CONTENT" "$ROADMAP_CONTENT")
DETECTED_ARCHITECTURE=$(detect_architecture "$TECH_STACK_CONTENT" "$ROADMAP_CONTENT")

echo ""
echo "📊 Detected Product Scope:"
echo "   Language: $DETECTED_LANGUAGE"
echo "   Frameworks: $DETECTED_FRAMEWORKS"
echo "   Project Type: $DETECTED_PROJECT_TYPE"
echo "   Architecture: $DETECTED_ARCHITECTURE"

# Save scope detection results
mkdir -p agent-os/output/product-cleanup
cat > agent-os/output/product-cleanup/detected-scope.yml << EOF
# Product Scope Detection Results
# Generated by product-focused-cleanup workflow

language: $DETECTED_LANGUAGE
frameworks: [$DETECTED_FRAMEWORKS]
project_type: $DETECTED_PROJECT_TYPE
architecture: $DETECTED_ARCHITECTURE

detection_sources:
  tech_stack: agent-os/product/tech-stack.md
  mission: agent-os/product/mission.md
  roadmap: agent-os/product/roadmap.md
EOF

echo "✅ Scope detection complete"
```

### Step 3: Phase A - Simplify (Remove Irrelevant Content)

Remove content that doesn't match the detected product scope:

```bash
echo ""
echo "🧹 Phase A: Simplifying agent-os files..."

# Initialize tracking
REMOVED_FILES=()
MODIFIED_FILES=()
REMOVED_CONTENT=()

# Define what to remove based on detected scope
define_removal_rules() {
    local lang="$1"
    local proj_type="$2"
    local arch="$3"
    
    # Language-based removal patterns
    case "$lang" in
        "typescript"|"javascript")
            REMOVE_LANG_PATTERNS="python ruby rust go java csharp php swift kotlin"
            ;;
        "python")
            REMOVE_LANG_PATTERNS="typescript javascript ruby rust go java csharp php swift kotlin"
            ;;
        "rust")
            REMOVE_LANG_PATTERNS="typescript javascript python ruby go java csharp php swift kotlin"
            ;;
        "go")
            REMOVE_LANG_PATTERNS="typescript javascript python ruby rust java csharp php swift kotlin"
            ;;
        *)
            REMOVE_LANG_PATTERNS=""
            ;;
    esac
    
    # Project type-based removal patterns
    case "$proj_type" in
        "cli")
            REMOVE_TYPE_PATTERNS="ui frontend screen design mockup component widget button form modal"
            ;;
        "api")
            REMOVE_TYPE_PATTERNS="ui frontend screen design mockup component widget button form modal mobile ios android"
            ;;
        "webapp")
            REMOVE_TYPE_PATTERNS="mobile ios android cli terminal console"
            ;;
        "mobile")
            REMOVE_TYPE_PATTERNS="cli terminal console"
            ;;
        "library")
            REMOVE_TYPE_PATTERNS="ui frontend screen design mockup mobile ios android"
            ;;
        "ai-service")
            REMOVE_TYPE_PATTERNS="crud traditional-web mobile ios android"
            ;;
        *)
            REMOVE_TYPE_PATTERNS=""
            ;;
    esac
    
    # Architecture-based removal patterns
    case "$arch" in
        "monolith")
            REMOVE_ARCH_PATTERNS="microservice kubernetes k8s service-mesh distributed"
            ;;
        "microservices")
            REMOVE_ARCH_PATTERNS="monolith single-app"
            ;;
        "serverless")
            REMOVE_ARCH_PATTERNS="monolith traditional-server"
            ;;
        *)
            REMOVE_ARCH_PATTERNS=""
            ;;
    esac
}

# Run removal rules definition
define_removal_rules "$DETECTED_LANGUAGE" "$DETECTED_PROJECT_TYPE" "$DETECTED_ARCHITECTURE"

# Process each agent-os directory
process_directory_for_removal() {
    local dir="$1"
    local dir_name=$(basename "$dir")
    
    echo "   Processing $dir_name/..."
    
    # Find all markdown files
    for file in $(find "$dir" -name "*.md" -type f 2>/dev/null); do
        local file_name=$(basename "$file")
        local file_content=$(cat "$file")
        local file_lower=$(echo "$file_content" | tr '[:upper:]' '[:lower:]')
        local should_remove=false
        local removal_reason=""
        
        # Check language patterns
        for pattern in $REMOVE_LANG_PATTERNS; do
            if echo "$file_lower" | grep -qiE "\b${pattern}\b.*example|\b${pattern}\b.*pattern"; then
                # Check if this is a significant match (not just a mention)
                match_count=$(echo "$file_lower" | grep -oiE "\b${pattern}\b" | wc -l)
                if [ "$match_count" -gt 3 ]; then
                    should_remove=true
                    removal_reason="Language not in scope: $pattern"
                    break
                fi
            fi
        done
        
        # Check project type patterns (for workflow files)
        if [ "$should_remove" = false ] && [[ "$dir" == *"workflows"* ]]; then
            for pattern in $REMOVE_TYPE_PATTERNS; do
                if echo "$file_name $file_lower" | grep -qiE "\b${pattern}\b"; then
                    should_remove=true
                    removal_reason="Project type not in scope: $pattern"
                    break
                fi
            done
        fi
        
        # Instead of removing files, mark sections for removal or simplify
        if [ "$should_remove" = true ]; then
            REMOVED_CONTENT+=("$file: $removal_reason")
            # Add marker comment at top of file
            echo "<!-- PRODUCT-CLEANUP: This file contains content outside product scope ($removal_reason). Consider reviewing or removing. -->" | cat - "$file" > temp && mv temp "$file"
            MODIFIED_FILES+=("$file")
        fi
    done
}

# Process directories
for dir in agent-os/commands agent-os/workflows agent-os/standards agent-os/agents; do
    if [ -d "$dir" ]; then
        process_directory_for_removal "$dir"
    fi
done

echo "✅ Phase A complete: ${#MODIFIED_FILES[@]} files marked for review"
```

### Step 4: Phase B - Expand (Enhance with Product-Specific Knowledge)

Enhance remaining content with product-specific examples and patterns:

```bash
echo ""
echo "📚 Phase B: Expanding with product-specific knowledge..."

# Initialize tracking
ENHANCED_FILES=()
ADDED_CONTENT=()

# Define enhancement rules based on detected scope
define_enhancement_rules() {
    local lang="$1"
    local frameworks="$2"
    local proj_type="$3"
    
    # Language-specific enhancements
    case "$lang" in
        "typescript")
            LANG_PATTERNS="TypeScript type safety, interfaces, generics, strict mode"
            LANG_EXAMPLES="Use TypeScript interfaces for data structures, leverage type inference"
            ;;
        "python")
            LANG_PATTERNS="Python type hints, dataclasses, async/await patterns"
            LANG_EXAMPLES="Use type hints for function signatures, leverage dataclasses for DTOs"
            ;;
        "rust")
            LANG_PATTERNS="Rust ownership, borrowing, lifetimes, Result/Option patterns"
            LANG_EXAMPLES="Use Result for error handling, leverage pattern matching"
            ;;
        "go")
            LANG_PATTERNS="Go interfaces, goroutines, channels, error handling"
            LANG_EXAMPLES="Use interfaces for abstraction, leverage goroutines for concurrency"
            ;;
        *)
            LANG_PATTERNS=""
            LANG_EXAMPLES=""
            ;;
    esac
    
    # Framework-specific enhancements
    FRAMEWORK_PATTERNS=""
    for fw in $frameworks; do
        case "$fw" in
            "react")
                FRAMEWORK_PATTERNS="$FRAMEWORK_PATTERNS React hooks, functional components, state management"
                ;;
            "express")
                FRAMEWORK_PATTERNS="$FRAMEWORK_PATTERNS Express middleware, routing, error handling"
                ;;
            "fastapi")
                FRAMEWORK_PATTERNS="$FRAMEWORK_PATTERNS FastAPI dependency injection, Pydantic models, async endpoints"
                ;;
            "nextjs")
                FRAMEWORK_PATTERNS="$FRAMEWORK_PATTERNS Next.js App Router, Server Components, API routes"
                ;;
        esac
    done
    
    # Project type enhancements
    case "$proj_type" in
        "cli")
            TYPE_PATTERNS="CLI argument parsing, interactive prompts, progress indicators, exit codes"
            ;;
        "api")
            TYPE_PATTERNS="REST conventions, API versioning, authentication, rate limiting, documentation"
            ;;
        "webapp")
            TYPE_PATTERNS="Component architecture, state management, routing, responsive design"
            ;;
        "ai-service")
            TYPE_PATTERNS="Prompt engineering, model integration, context management, token optimization"
            ;;
        *)
            TYPE_PATTERNS=""
            ;;
    esac
}

# Run enhancement rules definition
define_enhancement_rules "$DETECTED_LANGUAGE" "$DETECTED_FRAMEWORKS" "$DETECTED_PROJECT_TYPE"

# Inject product terminology from mission/roadmap
inject_product_terminology() {
    local file="$1"
    local mission="$2"
    local roadmap="$3"
    
    # Extract key terms from mission and roadmap
    # This is a simplified version - in practice, use NLP or keyword extraction
    local key_terms=$(echo "$mission $roadmap" | grep -oE '\b[A-Z][a-z]+[A-Z][a-zA-Z]*\b' | sort -u | head -10)
    
    if [ -n "$key_terms" ]; then
        echo "   Injecting product terminology into $file"
        ENHANCED_FILES+=("$file")
    fi
}

# Create product-specific enhancement section
create_enhancement_section() {
    cat << EOF

## Product-Specific Patterns

Based on detected product scope ($DETECTED_LANGUAGE / $DETECTED_PROJECT_TYPE):

### Language Patterns
$LANG_PATTERNS

### Framework Patterns
$FRAMEWORK_PATTERNS

### Project Type Patterns
$TYPE_PATTERNS

EOF
}

echo "✅ Phase B complete: Enhancement rules defined"
echo "   Language patterns: $LANG_PATTERNS"
echo "   Framework patterns: $FRAMEWORK_PATTERNS"
echo "   Project type patterns: $TYPE_PATTERNS"
```

### Step 5: Web Search for Trusted Information

Search for current best practices and validate with multiple sources:

```bash
echo ""
echo "🌐 Searching for trusted information (2+ source validation)..."

# Define search categories
SEARCH_CATEGORIES=(
    "best practices"
    "common patterns"
    "anti-patterns"
    "performance optimization"
    "security considerations"
    "testing strategies"
)

# Generate search queries based on detected scope
generate_search_queries() {
    local lang="$1"
    local frameworks="$2"
    local proj_type="$3"
    
    local queries=()
    
    # Language-specific queries
    [ -n "$lang" ] && [ "$lang" != "unknown" ] && queries+=("$lang best practices 2024")
    
    # Framework-specific queries
    for fw in $frameworks; do
        queries+=("$fw best practices patterns")
        queries+=("$fw common mistakes anti-patterns")
    done
    
    # Project type queries
    case "$proj_type" in
        "cli")
            queries+=("CLI application best practices")
            queries+=("command line tool UX patterns")
            ;;
        "api")
            queries+=("REST API design best practices")
            queries+=("API security best practices")
            ;;
        "webapp")
            queries+=("web application architecture patterns")
            queries+=("frontend performance optimization")
            ;;
        "ai-service")
            queries+=("LLM application best practices")
            queries+=("AI agent design patterns")
            ;;
    esac
    
    printf '%s\n' "${queries[@]}"
}

# Generate queries
SEARCH_QUERIES=$(generate_search_queries "$DETECTED_LANGUAGE" "$DETECTED_FRAMEWORKS" "$DETECTED_PROJECT_TYPE")

echo "   Generated search queries:"
echo "$SEARCH_QUERIES" | while read query; do
    [ -n "$query" ] && echo "   - $query"
done

# Note: Actual web search would be performed here
# The workflow instructs the AI agent to:
# 1. Perform web searches for each query
# 2. Validate information appears in 2+ independent sources
# 3. Prioritize: Official docs > Reputable blogs > Stack Overflow
# 4. Add validated information to enhancement section

echo ""
echo "📝 Web Search Instructions for AI Agent:"
echo "   1. Search for each query listed above"
echo "   2. Validate information from 2+ independent sources"
echo "   3. Prioritize sources: Official docs > Reputable tech blogs > Stack Overflow (high-voted)"
echo "   4. Include version-specific information when available"
echo "   5. Add validated patterns to agent-os files"

# Save search queries for reference
cat > agent-os/output/product-cleanup/search-queries.md << EOF
# Web Search Queries for Product Enhancement

Generated based on detected scope:
- Language: $DETECTED_LANGUAGE
- Frameworks: $DETECTED_FRAMEWORKS
- Project Type: $DETECTED_PROJECT_TYPE

## Queries to Execute

$(echo "$SEARCH_QUERIES" | while read query; do
    [ -n "$query" ] && echo "- [ ] $query"
done)

## Validation Requirements

- Information must appear in 2+ independent sources
- Prioritize: Official documentation > Reputable tech blogs > Stack Overflow
- Include version numbers when applicable
- Note the sources for each piece of information added

## Categories to Cover

$(for cat in "${SEARCH_CATEGORIES[@]}"; do
    echo "- [ ] $cat"
done)
EOF

echo "✅ Search queries saved to agent-os/output/product-cleanup/search-queries.md"
```

### Step 6: Generate Cleanup/Enhancement Report

Create a comprehensive report of all changes:

```bash
echo ""
echo "📊 Generating cleanup/enhancement report..."

# Create report directory
mkdir -p agent-os/output/product-cleanup

# Generate the report
cat > agent-os/output/product-cleanup/cleanup-report.md << EOF
# Product-Focused Cleanup Report

Generated: $(date -u +"%Y-%m-%d %H:%M:%S UTC")

## Detected Product Scope

| Attribute | Detected Value |
|-----------|----------------|
| Language | $DETECTED_LANGUAGE |
| Frameworks | $DETECTED_FRAMEWORKS |
| Project Type | $DETECTED_PROJECT_TYPE |
| Architecture | $DETECTED_ARCHITECTURE |

## Phase A: Simplification Summary

### Files Marked for Review
$(if [ ${#MODIFIED_FILES[@]} -gt 0 ]; then
    for file in "${MODIFIED_FILES[@]}"; do
        echo "- $file"
    done
else
    echo "No files marked for review"
fi)

### Content Flagged as Out-of-Scope
$(if [ ${#REMOVED_CONTENT[@]} -gt 0 ]; then
    for content in "${REMOVED_CONTENT[@]}"; do
        echo "- $content"
    done
else
    echo "No content flagged"
fi)

### Removal Rules Applied

**Language-based removals:** $REMOVE_LANG_PATTERNS
**Project type-based removals:** $REMOVE_TYPE_PATTERNS
**Architecture-based removals:** $REMOVE_ARCH_PATTERNS

## Phase B: Enhancement Summary

### Enhancement Rules Applied

**Language Patterns:** $LANG_PATTERNS
**Framework Patterns:** $FRAMEWORK_PATTERNS
**Project Type Patterns:** $TYPE_PATTERNS

### Files Enhanced
$(if [ ${#ENHANCED_FILES[@]} -gt 0 ]; then
    for file in "${ENHANCED_FILES[@]}"; do
        echo "- $file"
    done
else
    echo "Enhancement pending web search validation"
fi)

## Web Search Status

See \`search-queries.md\` for queries to execute and validate.

## Next Steps

1. Review files marked with PRODUCT-CLEANUP comments
2. Execute web searches and validate information
3. Add validated patterns to relevant agent-os files
4. Re-run this workflow to verify completeness

---

*This report was generated by the product-focused-cleanup workflow.*
*Location: agent-os/output/product-cleanup/cleanup-report.md*
EOF

echo "✅ Report generated: agent-os/output/product-cleanup/cleanup-report.md"
```

## Display Confirmation

Once cleanup is complete, output the following:

```
✅ Product-Focused Cleanup Complete!

📊 Detected Scope:
   Language: [detected language]
   Frameworks: [detected frameworks]
   Project Type: [detected project type]
   Architecture: [detected architecture]

🧹 Phase A (Simplify):
   Files reviewed: [count]
   Content flagged: [count]

📚 Phase B (Expand):
   Enhancement rules defined
   Web search queries generated

📁 Reports Generated:
   - agent-os/output/product-cleanup/detected-scope.yml
   - agent-os/output/product-cleanup/search-queries.md
   - agent-os/output/product-cleanup/cleanup-report.md

NEXT STEP 👉 Review the cleanup report and execute web searches for validated enhancements.
```

{{UNLESS compiled_single_command}}
## Display confirmation and next step

Output the confirmation message above after completing the cleanup workflow.
{{ENDUNLESS compiled_single_command}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

Ensure cleanup follows the user's preferences:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
