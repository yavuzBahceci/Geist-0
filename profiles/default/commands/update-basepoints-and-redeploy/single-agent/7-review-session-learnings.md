# Phase 7: Review Session Learnings

Present session learnings and prompt effectiveness analysis to user for review.

## Core Responsibilities

1. **Load Session Feedback**: Read current session data
2. **Extract Patterns**: Use extraction workflow to identify successful/failed patterns
3. **Analyze Prompts**: Use analysis workflow to determine prompt effectiveness
4. **Generate Adaptations**: Create specific suggestions based on learnings
5. **Present for Review**: Human approval required - NEVER auto-apply

## Prerequisites

- Session feedback exists from previous `/implement-tasks` runs
- Patterns have been extracted (use extraction workflow)
- Prompt effectiveness has been analyzed (use analysis workflow)

## Workflow

### Step 1: Load Session Feedback

```bash
SESSION_FILE="geist/output/session-feedback/current-session.md"

if [ ! -f "$SESSION_FILE" ]; then
    echo "ℹ️ No session feedback found. Skipping learning phase."
    echo "   Run /implement-tasks to generate session feedback."
    exit 0
fi

echo "📋 Loading session feedback..."
```

### Step 2: Extract Patterns from Session

Extract successful and failed patterns from session data:

```bash
{{workflows/learning/extract-session-patterns}}
```

This workflow will:
- Read all implementations from session
- Group by patterns used
- Calculate success rate per pattern
- Write patterns to `patterns/successful.md` and `patterns/failed.md`

### Step 3: Analyze Prompt Effectiveness

Analyze which prompts led to good/bad outcomes:

```bash
{{workflows/prompting/analyze-prompt-effectiveness}}
```

This workflow will:
- Identify effective prompts (no retries, passed first time)
- Identify ineffective prompts (needed clarification, failed)
- Generate improvement suggestions

### Step 4: Generate Adaptation Suggestions

Based on analysis, generate specific adaptation suggestions:

**For Successful Patterns** (used 3+ times with 100% success):
- Add to `/shape-spec` context section
- Add to `/implement-tasks` guidance section
- Include in relevant command patterns

**For Failed Patterns** (caused issues):
- Add warning to `/implement-tasks` constraints
- Add "DO NOT" section to relevant commands
- Include in anti-patterns documentation

**For Prompt Issues** (needed clarification):
- Add missing context to command prompts
- Clarify ambiguous instructions
- Add explicit constraints and boundaries

### Step 5: Present Human Review Checkpoint

Present learnings to user for approval:

```bash
{{workflows/learning/present-learnings-for-review}}
```

This workflow will:
- Display session summary
- Show successful patterns discovered
- Show anti-patterns identified
- Show prompt effectiveness analysis
- Present clear options (Approve all / Select / Skip)
- **WAIT for user response**

### Step 6: Handle User Response

Based on user choice:

```bash
# User choice is saved by present-learnings-for-review workflow
PENDING_FILE="geist/output/session-feedback/adaptations/pending.md"

if [ -f "$PENDING_FILE" ]; then
    APPROVAL_STATUS=$(grep "Approval Status:" "$PENDING_FILE" | sed 's/.*: \([^ ]*\).*/\1/')
    
    case "$APPROVAL_STATUS" in
        "all")
            echo "✅ All adaptations approved. Proceed to Phase 8: Adapt Commands."
            ;;
        "select")
            echo "✅ Selected adaptations approved. Proceed to Phase 8: Adapt Commands."
            ;;
        "skip")
            echo "ℹ️ Adaptations saved for later review. Skipping Phase 8."
            echo "   Adaptations saved to: $PENDING_FILE"
            exit 0
            ;;
    esac
else
    echo "⚠️ No adaptations file found. Skipping Phase 8."
    exit 0
fi
```

## Output

- Adaptations file: `geist/output/session-feedback/adaptations/pending.md`
- User approval status (all/select/skip)

## Critical Safety Requirements

- ⚠️ **ALWAYS ask user** before applying any changes
- ⚠️ **NEVER auto-apply** adaptations
- ⚠️ **Clear presentation** of what will change
- ⚠️ **Option to reject** and save for later

## Next Step

If user approved adaptations (all or select):
- Proceed to Phase 8: Adapt Commands

If user skipped:
- Exit (adaptations saved for later)
- Can review later by running Phase 7 again

## Notes

- This phase is CRITICAL for safety - human review is mandatory
- All adaptations require explicit user approval
- Rejected learnings are saved to history for future reference
- Session data is preserved for later analysis
