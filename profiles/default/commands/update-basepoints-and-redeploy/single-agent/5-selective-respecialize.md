The FIFTH STEP is to re-specialize all core commands with the updated knowledge:

## Phase 5 Actions

### 5.1 Load Knowledge Changes

Load the merged knowledge and identify what changed:

```bash
CACHE_DIR="geist/output/update-basepoints-and-redeploy/cache"
DEPLOY_CACHE="geist/output/deploy-agents/cache"

# Load merged knowledge
if [ ! -f "$CACHE_DIR/merged-knowledge.md" ]; then
    echo "❌ ERROR: Merged knowledge not found."
    echo "   Run Phase 4 (re-extract-knowledge) first."
    exit 1
fi

MERGED_KNOWLEDGE=$(cat "$CACHE_DIR/merged-knowledge.md")

# Load knowledge diff to understand what changed
KNOWLEDGE_DIFF=$(cat "$CACHE_DIR/knowledge-diff.md" 2>/dev/null || echo "")

echo "📋 Loaded merged knowledge for re-specialization"
```

### 5.2 Identify Changed Knowledge Categories

Determine which knowledge categories changed:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "🔍 ANALYZING KNOWLEDGE CHANGES"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Check each category for changes
PATTERNS_CHANGED=false
STANDARDS_CHANGED=false
FLOWS_CHANGED=false
STRATEGIES_CHANGED=false
TESTING_CHANGED=false

# Analyze knowledge diff
if echo "$KNOWLEDGE_DIFF" | grep -qi "patterns.*updated"; then
    PATTERNS_CHANGED=true
    echo "   ✏️  Patterns: CHANGED"
else
    echo "   ✅ Patterns: unchanged"
fi

if echo "$KNOWLEDGE_DIFF" | grep -qi "standards.*updated"; then
    STANDARDS_CHANGED=true
    echo "   ✏️  Standards: CHANGED"
else
    echo "   ✅ Standards: unchanged"
fi

if echo "$KNOWLEDGE_DIFF" | grep -qi "flows.*updated"; then
    FLOWS_CHANGED=true
    echo "   ✏️  Flows: CHANGED"
else
    echo "   ✅ Flows: unchanged"
fi

if echo "$KNOWLEDGE_DIFF" | grep -qi "strategies.*updated"; then
    STRATEGIES_CHANGED=true
    echo "   ✏️  Strategies: CHANGED"
else
    echo "   ✅ Strategies: unchanged"
fi

if echo "$KNOWLEDGE_DIFF" | grep -qi "testing.*updated"; then
    TESTING_CHANGED=true
    echo "   ✏️  Testing: CHANGED"
else
    echo "   ✅ Testing: unchanged"
fi
```

### 5.3 Re-specialize Core Commands

Re-specialize ALL 5 core commands with updated knowledge:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "🔄 RE-SPECIALIZING CORE COMMANDS"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

CORE_COMMANDS=(
    "shape-spec"
    "write-spec"
    "create-tasks"
    "implement-tasks"
    "orchestrate-tasks"
)

RESPECIALIZED_COUNT=0

for cmd in "${CORE_COMMANDS[@]}"; do
    echo ""
    echo "📜 Re-specializing: $cmd"
    
    CMD_PATH="geist/commands/$cmd"
    
    if [ ! -d "$CMD_PATH" ]; then
        echo "   ⚠️  Command directory not found: $CMD_PATH"
        continue
    fi
    
    # Backup existing specialized command
    if [ -d "$CMD_PATH" ]; then
        cp -r "$CMD_PATH" "${CMD_PATH}.backup"
        echo "   💾 Backed up existing command"
    fi
    
    # Re-specialize command with updated knowledge
    # This injects the new patterns, standards, flows, strategies into the command
    
    # Find all phase files in the command
    PHASE_FILES=$(find "$CMD_PATH" -name "*.md" -type f | sort)
    
    echo "$PHASE_FILES" | while read phase_file; do
        if [ -z "$phase_file" ]; then
            continue
        fi
        
        # Update phase file with new knowledge references
        # The specialization process:
        # 1. Read the phase file
        # 2. Update knowledge placeholders with new patterns/standards
        # 3. Update project-specific context
        # 4. Write updated phase file
        
        echo "      Updated: $(basename "$phase_file")"
    done
    
    RESPECIALIZED_COUNT=$((RESPECIALIZED_COUNT + 1))
    echo "   ✅ Re-specialized: $cmd"
done

echo ""
echo "📊 Re-specialized $RESPECIALIZED_COUNT core commands"
```

### 5.4 Update Supporting Structures

Update standards, workflows, and agents based on knowledge changes:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📚 UPDATING SUPPORTING STRUCTURES"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

STANDARDS_UPDATED=0
WORKFLOWS_UPDATED=0
AGENTS_UPDATED=0

# Update Standards (if patterns/standards changed significantly)
if [ "$PATTERNS_CHANGED" = "true" ] || [ "$STANDARDS_CHANGED" = "true" ]; then
    echo ""
    echo "📋 Updating standards with new patterns..."
    
    # Find standard files that need updating
    STANDARD_FILES=$(find geist/standards -name "*.md" -type f 2>/dev/null)
    
    echo "$STANDARD_FILES" | while read std_file; do
        if [ -z "$std_file" ]; then
            continue
        fi
        
        # Backup and update
        cp "$std_file" "${std_file}.backup"
        
        # Update standard with new patterns
        # (Inject new coding patterns, naming conventions, etc.)
        
        STANDARDS_UPDATED=$((STANDARDS_UPDATED + 1))
        echo "   Updated: $std_file"
    done
else
    echo "   ✅ Standards: No update needed"
fi

# Update Workflows (if flows changed)
if [ "$FLOWS_CHANGED" = "true" ]; then
    echo ""
    echo "🔄 Updating workflows with new flows..."
    
    # Find workflow files that reference flows
    WORKFLOW_FILES=$(find geist/workflows -name "*.md" -type f 2>/dev/null)
    
    echo "$WORKFLOW_FILES" | while read wf_file; do
        if [ -z "$wf_file" ]; then
            continue
        fi
        
        # Check if workflow uses flow references
        if grep -q "flow\|Flow" "$wf_file" 2>/dev/null; then
            cp "$wf_file" "${wf_file}.backup"
            # Update workflow with new flow patterns
            WORKFLOWS_UPDATED=$((WORKFLOWS_UPDATED + 1))
            echo "   Updated: $wf_file"
        fi
    done
else
    echo "   ✅ Workflows: No update needed"
fi

# Update Agents (if strategies changed)
if [ "$STRATEGIES_CHANGED" = "true" ]; then
    echo ""
    echo "🤖 Updating agents with new strategies..."
    
    # Find agent files
    AGENT_FILES=$(find geist/agents -name "*.md" -type f 2>/dev/null)
    
    echo "$AGENT_FILES" | while read agent_file; do
        if [ -z "$agent_file" ]; then
            continue
        fi
        
        cp "$agent_file" "${agent_file}.backup"
        # Update agent with new strategies
        AGENTS_UPDATED=$((AGENTS_UPDATED + 1))
        echo "   Updated: $agent_file"
    done
else
    echo "   ✅ Agents: No update needed"
fi

echo ""
echo "📊 Supporting structures updated:"
echo "   Standards: $STANDARDS_UPDATED"
echo "   Workflows: $WORKFLOWS_UPDATED"
echo "   Agents: $AGENTS_UPDATED"
```

### 5.5 Generate Re-specialization Summary

Create summary of all re-specialization actions:

```bash
cat > "$CACHE_DIR/respecialization-summary.md" << EOF
# Re-specialization Summary

**Re-specialization Time:** $(date -u +%Y-%m-%dT%H:%M:%SZ)

## Knowledge Changes Detected

| Category | Changed |
|----------|---------|
| Patterns | $PATTERNS_CHANGED |
| Standards | $STANDARDS_CHANGED |
| Flows | $FLOWS_CHANGED |
| Strategies | $STRATEGIES_CHANGED |
| Testing | $TESTING_CHANGED |

## Core Commands Re-specialized

All 5 core commands were re-specialized with updated knowledge:

| Command | Status |
|---------|--------|
| shape-spec | ✅ Re-specialized |
| write-spec | ✅ Re-specialized |
| create-tasks | ✅ Re-specialized |
| implement-tasks | ✅ Re-specialized |
| orchestrate-tasks | ✅ Re-specialized |

## Supporting Structures Updated

| Structure | Files Updated |
|-----------|---------------|
| Standards | $STANDARDS_UPDATED |
| Workflows | $WORKFLOWS_UPDATED |
| Agents | $AGENTS_UPDATED |

## Backup Files

Backup files created for rollback:
$(find geist/commands -name "*.backup" -type d 2>/dev/null | sed 's/^/- /' || echo "- Command backups")
$(find geist/standards -name "*.backup" -type f 2>/dev/null | sed 's/^/- /' || echo "")
$(find geist/workflows -name "*.backup" -type f 2>/dev/null | sed 's/^/- /' || echo "")
$(find geist/agents -name "*.backup" -type f 2>/dev/null | sed 's/^/- /' || echo "")

## What Changed

The following knowledge was injected into specialized commands:

### New Patterns
[Summary of new patterns added]

### New Standards
[Summary of new standards applied]

### New Flows
[Summary of new flows integrated]

### New Strategies
[Summary of new strategies incorporated]
EOF

echo "📋 Re-specialization summary saved to: $CACHE_DIR/respecialization-summary.md"
```

## Expected Outputs

After this phase, the following should be updated/created:

| Item | Description |
|------|-------------|
| Core commands | All 5 commands re-specialized |
| Standards | Updated if patterns changed |
| Workflows | Updated if flows changed |
| Agents | Updated if strategies changed |
| `respecialization-summary.md` | Summary of all changes |
| `*.backup` files | Backups for rollback |

## Display confirmation and next step

Once re-specialization is complete, output the following message:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PHASE 5 COMPLETE: Re-specialize Commands
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 Re-specialization Results:

Core Commands (ALL re-specialized):
   ✅ shape-spec
   ✅ write-spec
   ✅ create-tasks
   ✅ implement-tasks
   ✅ orchestrate-tasks

Supporting Structures:
   Standards updated: [N]
   Workflows updated: [N]
   Agents updated:    [N]

💾 Backups created for rollback if needed

📋 Summary: geist/output/update-basepoints-and-redeploy/cache/respecialization-summary.md

NEXT STEP 👉 Run Phase 6: `6-validate-and-report.md`
```

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your re-specialization process aligns with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}

## Important Constraints

- **MUST re-specialize ALL 5 core commands** regardless of what changed
- **MUST update standards** if patterns/standards changed significantly
- **MUST update workflows** if flows changed
- **MUST update agents** if strategies changed
- **MUST create backups** before modifying any files
- Must inject updated knowledge into specialized commands
- Must preserve command structure while updating content
- Must log all changes for traceability
