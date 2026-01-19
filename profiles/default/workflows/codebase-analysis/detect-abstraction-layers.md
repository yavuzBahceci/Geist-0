# Abstraction Layer Detection

## Core Responsibilities

1. **Analyze Folder Structure**: Examine the project's directory structure to identify potential abstraction layers
2. **Detect Common Patterns**: Identify common architecture patterns (mobile, web, etc.)
3. **Analyze File Naming**: Review file naming conventions to infer layer boundaries
4. **Prompt User Confirmation**: Present detected layers to user for confirmation or refinement
5. **Document Detected Layers**: Save the detected abstraction layers for use in subsequent phases

## Workflow

### Step 1: Analyze Folder Structure

Scan the project root to identify top-level directories:

```bash
find . -maxdepth 1 -type d ! -name ".*" ! -name "node_modules" ! -name "vendor" ! -name "build" ! -name "dist" ! -name ".next" ! -name "geist" ! -name "basepoints" | sort
```

Analyze for common patterns:
- **Mobile**: `data/`, `domain/`, `presentation/`, `core/`, `common/`, `util/`
- **Web**: `backend/`, `frontend/`, `api/`, `client/`, `server/`
- **General**: `src/`, `lib/`, `modules/`, `components/`, `services/`, `models/`

### Step 2: Detect Common Patterns

Identify architecture patterns:

```bash
# Detect mobile app structure
if [ -d "data" ] && [ -d "domain" ] && [ -d "presentation" ]; then
    DETECTED_ARCH="mobile"
fi

# Detect web app structure
if [ -d "backend" ] || [ -d "frontend" ]; then
    DETECTED_ARCH="web"
fi

# Detect generic layered structure
if [ -d "src" ] && [ -d "lib" ]; then
    DETECTED_ARCH="layered"
fi
```

### Step 3: Analyze File Naming Patterns

Examine file naming conventions to infer layer information:

```bash
find . -type f -name "*.ts" -o -name "*.js" -o -name "*.py" | head -50 | while read file; do
    # Analyze file path and name to infer layer
    echo "$file"
done
```

### Step 4: Compile Detected Layers

Create summary of detected abstraction layers:

```bash
mkdir -p geist/output/create-basepoints/cache
cat > geist/output/create-basepoints/cache/detected-layers.md << EOF
# Detected Abstraction Layers

## Architecture Type
$DETECTED_ARCH

## Identified Layers
[Layer names with paths and purposes]

EOF
```

### Step 5: Prompt User for Confirmation

OUTPUT detected layers to user and ask for confirmation. WAIT for user response.

### Step 6: Process User Feedback

IF user provides corrections, update detected layers accordingly.

### Step 7: Save Final Layers

Save final detected abstraction layers to cache for use in subsequent phases.

## Important Constraints

- Must analyze actual folder structure, not make assumptions
- Must present detected layers to user for confirmation
- Must allow user to override or refine layer boundaries
- Must document final layers for use in subsequent phases
- Handle unknown or custom architectures gracefully
