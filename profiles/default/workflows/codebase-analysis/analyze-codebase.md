# Codebase Analysis

## Core Responsibilities

1. **Analyze Code Files**: Examine all code files in identified module folders
2. **Analyze Configuration Files**: Review configuration files for project settings and patterns
3. **Analyze Test Files**: Extract testing approaches and patterns from test files
4. **Analyze Documentation**: Review documentation files for architectural insights
5. **Extract Patterns**: Identify patterns, standards, flows, strategies, and testing approaches
6. **Process Layer by Layer**: Extract patterns at every abstraction layer and across layers

## Workflow

### Step 1: Load Module Folders

Load the list of module folders identified in previous phase:

```bash
if [ -f "geist/output/create-basepoints/cache/module-folders.txt" ]; then
    MODULE_FOLDERS=$(cat geist/output/create-basepoints/cache/module-folders.txt | grep -v "^#")
fi
```

### Step 2: Analyze Code Files

Process each module folder and analyze code files:

```bash
mkdir -p geist/output/create-basepoints/analysis

echo "$MODULE_FOLDERS" | while read module_dir; do
    if [ -z "$module_dir" ]; then
        continue
    fi
    
    # Find code files in this module
    find "$module_dir" -type f \( -name "*.ts" -o -name "*.js" -o -name "*.py" -o -name "*.java" -o -name "*.go" -o -name "*.rs" -o -name "*.rb" -o -name "*.php" -o -name "*.cs" \) | while read code_file; do
        # Analyze file for patterns, standards, flows, strategies, testing
        # Store analysis in cache
    done
done
```

### Step 3: Extract Patterns

For each file, extract:
- **Patterns**: Design patterns, coding patterns, architectural patterns
- **Standards**: Coding standards, naming conventions, formatting rules
- **Flows**: Data flows, control flows, dependency flows
- **Strategies**: Implementation strategies, architectural strategies
- **Testing Approaches**: Test patterns, test strategies, test organization

### Step 4: Analyze Configuration Files

Analyze configuration files:

```bash
find . -type f \( -name "package.json" -o -name "tsconfig.json" -o -name "pyproject.toml" -o -name "go.mod" -o -name "*.config.*" \) ! -path "*/node_modules/*" | while read config_file; do
    # Extract configuration patterns and standards
done
```

### Step 5: Analyze Test Files

Analyze test files:

```bash
find . -type f \( -name "*test*" -o -name "*spec*" \) ! -path "*/node_modules/*" | while read test_file; do
    # Extract testing patterns and strategies
done
```

### Step 6: Analyze Documentation

Review documentation files:

```bash
find . -type f \( -name "README.md" -o -name "ARCHITECTURE.md" -o -name "DESIGN.md" \) ! -path "*/node_modules/*" ! -path "*/geist/*" | while read doc_file; do
    # Extract architectural insights and patterns
done
```

### Step 7: Aggregate Patterns by Layer

Organize extracted patterns by abstraction layer using detected layers from cache.

### Step 8: Save Analysis Results

Save comprehensive analysis results to cache for use in basepoint generation.

## Important Constraints

- Must process files one by one for comprehensive extraction
- Must analyze at every abstraction layer individually
- Must also extract patterns across abstraction layers
- Must extract patterns, standards, flows, strategies, and testing approaches comprehensively
- Must save analysis results for use in basepoint generation phases
