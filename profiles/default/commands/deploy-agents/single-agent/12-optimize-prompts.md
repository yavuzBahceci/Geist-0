# Phase 12: Enhance Commands with Context Sections

Now that the project is analyzed and specialized, enhance all command templates with context sections at the top. This replaces runtime prompt construction with compile-time context injection.

## Core Responsibilities

1. **Load Project Context**: Tech stack, patterns, conventions from basepoints and project profile
2. **Load Session Learnings**: Successful patterns and anti-patterns from session feedback (if available)
3. **Generate Context Sections**: Create context sections for each command based on project-specific information
4. **Present for Review**: Show context sections to user for approval before injection

## Workflow

### Step 1: Load Project Context

```bash
echo "📋 Loading project context for context section generation..."

# Load from basepoints
HEADQUARTER=$(cat agent-os/basepoints/headquarter.md 2>/dev/null || echo "")
PROJECT_PROFILE=$(cat agent-os/config/project-profile.yml 2>/dev/null || echo "")

# Extract key information
TECH_STACK=$(echo "$PROJECT_PROFILE" | grep -E "language:|primary_language:" | cut -d: -f2 | tr -d ' ' | head -1)
FRAMEWORK=$(echo "$PROJECT_PROFILE" | grep -E "framework:|primary_framework:" | cut -d: -f2 | tr -d ' ' | head -1)
ARCHITECTURE=$(echo "$PROJECT_PROFILE" | grep -E "architecture_style:|architecture:" | cut -d: -f2 | tr -d ' ' | head -1)

# Extract patterns from headquarter
KEY_PATTERNS=$(echo "$HEADQUARTER" | grep -E "Pattern|pattern|Convention" | head -5 | sed 's/^/  - /')

echo "   Tech Stack: $TECH_STACK"
echo "   Framework: $FRAMEWORK"
echo "   Architecture: $ARCHITECTURE"
```

### Step 2: Load Session Learnings (if available)

```bash
# Load session learnings for patterns and anti-patterns
PATTERNS_SUCCESS=""
PATTERNS_FAILED=""

if [ -f "agent-os/output/session-feedback/patterns/successful.md" ]; then
    PATTERNS_SUCCESS=$(cat agent-os/output/session-feedback/patterns/successful.md | grep -E "^\s*-\s*✅" | head -3 | sed 's/^/  /')
fi

if [ -f "agent-os/output/session-feedback/patterns/failed.md" ]; then
    PATTERNS_FAILED=$(cat agent-os/output/session-feedback/patterns/failed.md | grep -E "^\s*-\s*❌" | head -3 | sed 's/^/  /')
fi

echo "   Loaded session learnings (if available)"
```

### Step 3: Generate Context Sections for Each Command

For each command that needs context enhancement, generate a context section:

```bash
# Commands to enhance
COMMANDS_TO_ENHANCE=(
    "shape-spec/single-agent/2-shape-spec.md"
    "write-spec/single-agent/write-spec.md"
    "create-tasks/single-agent/2-create-tasks-list.md"
    "implement-tasks/single-agent/2-implement-tasks.md"
    "orchestrate-tasks/orchestrate-tasks.md"
)

CONTEXT_SECTIONS=""

for CMD_PATH in "${COMMANDS_TO_ENHANCE[@]}"; do
    CMD_FILE="agent-os/commands/$CMD_PATH"
    
    if [ ! -f "$CMD_FILE" ]; then
        continue
    fi
    
    CMD_NAME=$(basename "$CMD_PATH" .md)
    
    # Generate context section for this command
    CONTEXT_SECTION=$(cat <<EOF
## Context

### Project Identity
- Tech Stack: $TECH_STACK
- Framework: $FRAMEWORK
- Architecture: $ARCHITECTURE
- Key Patterns:
$KEY_PATTERNS

### Relevant Patterns
$PATTERNS_SUCCESS

### Constraints & Anti-patterns
$PATTERNS_FAILED

### Previous Handoff
[Will be loaded from agent-os/output/handoff/current.md at runtime if exists]

---

EOF
)
    
    CONTEXT_SECTIONS="${CONTEXT_SECTIONS}\n\n=== $CMD_NAME ===\n$CONTEXT_SECTION"
done
```

### Step 4: Present for Human Review

Display generated context sections:

```
┌─────────────────────────────────────────────────────────────────┐
│  📋 CONTEXT SECTIONS TO INJECT                                  │
│                                                                 │
│  Based on project analysis:                                     │
│  • Tech Stack: $TECH_STACK                                         │
│  • Architecture: $ARCHITECTURE                                   │
│  • Key Patterns: [from basepoints]                              │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  [1] /shape-spec                                                │
│      Context section preview:                                    │
│      - Project Identity (tech stack, architecture)              │
│      - Relevant Patterns (from session learnings)               │
│      - Constraints (anti-patterns to avoid)                    │
│                                                                 │
│  [2] /write-spec                                                │
│      [Same structure]                                           │
│                                                                 │
│  [3] /create-tasks                                              │
│      [Same structure]                                           │
│                                                                 │
│  [4] /implement-tasks                                           │
│      [Same structure]                                           │
│                                                                 │
│  [5] /orchestrate-tasks                                         │
│      [Same structure]                                           │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  Options:                                                       │
│  [a] Apply all                                                  │
│  [s] Select which to apply                                      │
│  [n] Skip (save for later)                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

Wait for user response before proceeding.

## Output

Save context sections to: `agent-os/output/deploy-agents/cache/context-sections.md`

If user selects "Skip", save context sections for later review in `agent-os/output/deploy-agents/cache/context-sections-pending.md`.

## Next Step

If user approved context sections (selected "Apply all" or specific commands), proceed to Phase 13: Apply Context Sections.

If user skipped, continue to finalize deployment (skip Phase 13).
