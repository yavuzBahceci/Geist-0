# Prerequisites Validation

## Core Responsibilities

1. **Check Product Files Existence**: Verify that required product files exist
2. **Handle Missing Product Files**: Prompt user to create product files if missing
3. **Validate Basepoints Folder**: Check if basepoints folder already exists and handle accordingly
4. **Set Up Initial Structure**: Prepare basepoints folder structure for the process

## Workflow

### Step 1: Check Product Files Existence

Check if the required product files exist before proceeding:

```bash
# Check if product files exist
if [ ! -f "geist/product/mission.md" ] || [ ! -f "geist/product/roadmap.md" ] || [ ! -f "geist/product/tech-stack.md" ]; then
    echo "ERROR: Required product files are missing."
    echo "Missing files:"
    [ ! -f "geist/product/mission.md" ] && echo "  - geist/product/mission.md"
    [ ! -f "geist/product/roadmap.md" ] && echo "  - geist/product/roadmap.md"
    [ ! -f "geist/product/tech-stack.md" ] && echo "  - geist/product/tech-stack.md"
    exit 1
else
    echo "✅ All required product files found."
fi
```

### Step 2: Handle Missing Product Files

IF any product files are missing, OUTPUT the following to the user and STOP:

```
❌ Product files required before proceeding.

Missing product files detected. Please create them first using one of these commands:

👉 Run `/plan-product` to create new product documentation
👉 Run `/adapt-to-product` to generate product documentation from existing codebase

Required files:
- geist/product/mission.md
- geist/product/roadmap.md
- geist/product/tech-stack.md

Once these files exist, you can run `/create-basepoints` again.
```

**WAIT for user to create product files before proceeding.**

### Step 3: Validate Basepoints Folder

Check if basepoints folder already exists:

```bash
# Check if basepoints folder already exists
if [ -d "geist/basepoints" ]; then
    if [ "$(ls -A geist/basepoints 2>/dev/null)" ]; then
        echo "⚠️  Basepoints folder already exists and contains files."
        echo "Options:"
        echo "1. Use existing basepoints folder (will overwrite existing files)"
        echo "2. Backup existing basepoints and create fresh"
        echo "3. Cancel operation"
    else
        echo "✅ Basepoints folder exists but is empty. Ready to proceed."
    fi
else
    echo "✅ Basepoints folder will be created."
fi
```

IF basepoints folder exists and contains files, PROMPT user with options and WAIT for response.

### Step 4: Set Up Initial Structure

IF proceeding, prepare the structure:

```bash
# If user chose backup, create backup first
if [ "$USER_CHOICE" = "2" ]; then
    BACKUP_NAME="geist/basepoints-backup-$(date +%Y%m%d-%H%M%S)"
    mv geist/basepoints "$BACKUP_NAME"
    echo "✅ Backed up existing basepoints to: $BACKUP_NAME"
fi

# Create basepoints folder if it doesn't exist
mkdir -p geist/basepoints

echo "✅ Basepoints folder structure ready."
```

## Important Constraints

- **MANDATORY**: Product files must exist before proceeding
- If product files are missing, workflow must stop and direct user to create them
- Handle existing basepoints folder gracefully (backup or overwrite options)
- Always create basepoints folder structure before proceeding to next phase
