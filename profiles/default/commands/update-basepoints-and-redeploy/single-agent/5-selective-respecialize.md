The FIFTH STEP is to update Standards and Agents based on the updated knowledge.

**IMPORTANT:** Commands and Workflows are STATIC and are NEVER modified after installation.
Only Standards and Agents are updated based on code changes.

## Key Principle

```
┌─────────────────────────────────────────────────────────────────┐
│  STATIC (Never change after installation)                        │
│  ─────────────────────────────────────────                       │
│  Commands   - Reference @geist/standards, @geist/agents          │
│  Workflows  - Reusable building blocks                           │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  DYNAMIC (Updated based on code changes)                         │
│  ──────────────────────────────────────                          │
│  Basepoints - Patterns, flows, Libraries Used from code          │
│  Standards  - Coding rules derived from basepoints               │
│  Agents     - Expertise from basepoints + standards              │
│              (can be created, updated, or removed)               │
└─────────────────────────────────────────────────────────────────┘
```

## Phase 5 Actions

### 5.1 Load Knowledge Changes

Load the merged knowledge and identify what changed:

```bash
CACHE_DIR="geist/output/update-basepoints-and-redeploy/cache"

# Load merged knowledge
if [ ! -f "$CACHE_DIR/merged-knowledge.md" ]; then
    echo "❌ ERROR: Merged knowledge not found."
    echo "   Run Phase 4 (re-extract-knowledge) first."
    exit 1
fi

# Load change flags from previous phases
PATTERNS_CHANGED=$(cat "$CACHE_DIR/patterns-changed.txt" 2>/dev/null || echo "false")
STANDARDS_CHANGED=$(cat "$CACHE_DIR/standards-changed.txt" 2>/dev/null || echo "false")
FLOWS_CHANGED=$(cat "$CACHE_DIR/flows-changed.txt" 2>/dev/null || echo "false")
STRATEGIES_CHANGED=$(cat "$CACHE_DIR/strategies-changed.txt" 2>/dev/null || echo "false")
LIBRARY_USAGE_CHANGED=$(cat "$CACHE_DIR/library-usage-changed.txt" 2>/dev/null || echo "false")

# Record phase start time for tracking updates
date +%s > "$CACHE_DIR/phase-5-start-time.txt"

echo "📋 Loaded knowledge change flags"
echo "   Patterns changed: $PATTERNS_CHANGED"
echo "   Standards changed: $STANDARDS_CHANGED"
echo "   Flows changed: $FLOWS_CHANGED"
echo "   Strategies changed: $STRATEGIES_CHANGED"
echo "   Library usage changed: $LIBRARY_USAGE_CHANGED"
```

### 5.2 Update Standards from Basepoints

Update Standards based on basepoint pattern changes:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📋 UPDATING STANDARDS FROM BASEPOINTS"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

STANDARDS_UPDATED=0
STANDARDS_CREATED=0
STANDARDS_REMOVED=0

if [ "$PATTERNS_CHANGED" = "true" ] || [ "$STANDARDS_CHANGED" = "true" ] || [ "$LIBRARY_USAGE_CHANGED" = "true" ]; then
    echo "   Changes detected - updating standards..."
    
    STANDARDS_DIR="geist/standards"
    
    # Update existing standards with new patterns from basepoints
    if [ -d "$STANDARDS_DIR" ]; then
        for std_file in $(find "$STANDARDS_DIR" -name "*.md" -type f); do
            # Backup before updating
            cp "$std_file" "${std_file}.backup"
            
            # Update standard with new patterns
            # - coding-style.md: Update with new coding patterns from basepoints
            # - conventions.md: Update with new naming conventions
            # - Add library-specific patterns if library usage changed
            
            STANDARDS_UPDATED=$((STANDARDS_UPDATED + 1))
            echo "      Updated: $std_file"
        done
    fi
    
    # Create new standards if needed (e.g., new library patterns)
    if [ "$LIBRARY_USAGE_CHANGED" = "true" ]; then
        # Check if new library-specific standards are needed
        # Create standards for new libraries if patterns are significant
        echo "      Checking for new library-specific standards..."
    fi
    
    # Remove standards if patterns no longer exist
    # (Flag for manual review rather than auto-delete)
    
    echo "   ✅ Standards updated from basepoints"
else
    echo "   ℹ️ No standard updates needed (no relevant changes detected)"
fi
```

### 5.3 Update Agents from Basepoints + Standards

Update Agents based on basepoint and standards changes:

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "🤖 UPDATING AGENTS FROM BASEPOINTS + STANDARDS"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

AGENTS_UPDATED=0
AGENTS_CREATED=0
AGENTS_REMOVED=0

if [ "$STRATEGIES_CHANGED" = "true" ] || [ "$FLOWS_CHANGED" = "true" ] || [ "$LIBRARY_USAGE_CHANGED" = "true" ]; then
    echo "   Changes detected - updating agents..."
    
    AGENTS_DIR="geist/agents"
    
    # Update existing specialist agents with new expertise
    if [ -d "$AGENTS_DIR" ]; then
        for agent_file in $(find "$AGENTS_DIR" -name "*.md" -type f); do
            # Backup before updating
            cp "$agent_file" "${agent_file}.backup"
            
            # Update agent with new knowledge:
            # - Inject new patterns, flows, library knowledge
            # - Update agent contexts
            # - Include library-specific expertise
            
            AGENTS_UPDATED=$((AGENTS_UPDATED + 1))
            echo "      Updated: $agent_file"
        done
    fi
    
    # Create new agents if new abstraction layers detected
    # (e.g., new module types that need specialist agents)
    
    # Remove agents if layers no longer exist
    # (Flag for manual review rather than auto-delete)
    
    # Update registry.yml if agent capabilities changed
    if [ -f "geist/agents/registry.yml" ]; then
        cp "geist/agents/registry.yml" "geist/agents/registry.yml.backup"
        echo "      Updated: registry.yml"
    fi
    
    echo "   ✅ Agents updated from basepoints + standards"
else
    echo "   ℹ️ No agent updates needed (no relevant changes detected)"
fi
```

### 5.4 Generate Update Summary

Create summary of Standards and Agents updates:

```bash
# Calculate totals
PHASE_START=$(cat "$CACHE_DIR/phase-5-start-time.txt" 2>/dev/null || echo "0")

# Count files modified after phase start
if [ -d "geist/standards" ]; then
    STANDARDS_MODIFIED=$(find geist/standards -name "*.md" -newer "$CACHE_DIR/phase-5-start-time.txt" 2>/dev/null | wc -l | tr -d ' ')
else
    STANDARDS_MODIFIED=0
fi

if [ -d "geist/agents" ]; then
    AGENTS_MODIFIED=$(find geist/agents -name "*.md" -newer "$CACHE_DIR/phase-5-start-time.txt" 2>/dev/null | wc -l | tr -d ' ')
else
    AGENTS_MODIFIED=0
fi

cat > "$CACHE_DIR/specialization-summary.md" << EOF
# Specialization Summary

**Update Time:** $(date -u +%Y-%m-%dT%H:%M:%SZ)

## Key Principle

\`\`\`
STATIC (Never change after installation):
  - Commands ✅ (unchanged)
  - Workflows ✅ (unchanged)

DYNAMIC (Updated based on code changes):
  - Basepoints ✅ (updated in Phase 3)
  - Standards ✅ (updated in this phase)
  - Agents ✅ (updated in this phase)
\`\`\`

## Knowledge Changes Detected

| Category | Changed |
|----------|---------|
| Patterns | $PATTERNS_CHANGED |
| Standards | $STANDARDS_CHANGED |
| Flows | $FLOWS_CHANGED |
| Strategies | $STRATEGIES_CHANGED |
| Library Usage | $LIBRARY_USAGE_CHANGED |

## Updates Applied

### Standards
- Updated: $STANDARDS_UPDATED
- Created: $STANDARDS_CREATED
- Removed: $STANDARDS_REMOVED

### Agents
- Updated: $AGENTS_UPDATED
- Created: $AGENTS_CREATED
- Removed: $AGENTS_REMOVED

## NOT Modified (Static)

| Item | Status |
|------|--------|
| Commands | ✅ Unchanged (static) |
| Workflows | ✅ Unchanged (static) |

## Backup Files

Backup files created for rollback:
$(find geist/standards -name "*.backup" -type f 2>/dev/null | sed 's/^/- /' || echo "- None")
$(find geist/agents -name "*.backup" -type f 2>/dev/null | sed 's/^/- /' || echo "")
EOF

echo ""
echo "📋 Specialization summary saved to: $CACHE_DIR/specialization-summary.md"
```

## Expected Outputs

After this phase, the following should be updated/created:

| Item | Description |
|------|-------------|
| Standards | Updated if patterns/library usage changed |
| Agents | Updated if strategies/flows/library usage changed |
| `specialization-summary.md` | Summary of all changes |
| `*.backup` files | Backups for rollback |

**NOT Modified:**
- Commands (static)
- Workflows (static)

## Display confirmation and next step

Once specialization is complete, output the following message:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PHASE 5 COMPLETE: Update Standards and Agents
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 Specialization Results:

Standards:
   Updated: [N]
   Created: [N]
   Removed: [N]

Agents:
   Updated: [N]
   Created: [N]
   Removed: [N]

Static (unchanged):
   ℹ️ Commands: unchanged (static)
   ℹ️ Workflows: unchanged (static)

💾 Backups created for rollback if needed

📋 Summary: geist/output/update-basepoints-and-redeploy/cache/specialization-summary.md

NEXT STEP 👉 Run Phase 6: `6-validate-and-report.md`
```

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your specialization process aligns with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}

## Important Constraints

- **NEVER modify Commands** - Commands are STATIC after installation
- **NEVER modify Workflows** - Workflows are STATIC after installation
- **ONLY update Standards** if patterns/standards/library usage changed
- **ONLY update Agents** if strategies/flows/library usage changed
- **MUST create backups** before modifying any files
- Must inject updated knowledge into Standards and Agents
- Must include library knowledge in Standards and Agents
- Must log all changes for traceability
