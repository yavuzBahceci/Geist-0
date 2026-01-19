# Construct Prompt

Core workflow called at the START of every command to build an optimized prompt.

## When Called

- At the beginning of EVERY command execution
- Before any command logic runs
- Automatically injected into command flow

## Inputs

- `COMMAND_NAME` - Which command is running
- `USER_INPUT` - What the user provided (e.g., feature description)
- `PREVIOUS_OUTPUT` - Output from previous command (if any)

## Process

### Step 1: Load Context Sources

```bash
# Load basepoints
HEADQUARTER=$(cat geist/basepoints/headquarter.md 2>/dev/null || echo "")
PROJECT_PROFILE=$(cat geist/config/project-profile.yml 2>/dev/null || echo "")

# Load session learnings
SUCCESS_PATTERNS=$(cat geist/output/session-feedback/patterns/successful.md 2>/dev/null || echo "")
FAILED_PATTERNS=$(cat geist/output/session-feedback/patterns/failed.md 2>/dev/null || echo "")

# Load previous handoff (if exists)
HANDOFF_CONTEXT=$(cat geist/output/handoff/current.md 2>/dev/null || echo "")
```

### Step 2: Build Context Block

Based on command type, select relevant context:

| Command | Context Needed |
|---------|---------------|
| /shape-spec | Tech stack, architecture patterns, similar features |
| /write-spec | Requirements, architecture, standards |
| /create-tasks | Spec details, complexity, patterns |
| /implement-tasks | Tasks, patterns, validation commands, layer info |
| /orchestrate-tasks | All tasks, specialists, dependencies |

**Example Context Block:**

```markdown
## Context

### Project
[Extract from headquarter.md: tech stack, architecture style, patterns]

### Relevant Patterns
[From session learnings: patterns that apply to this task]

### Constraints
[From standards + anti-patterns: what to avoid]

### Previous Output
[If handoff exists: summary of what was completed]
```

### Step 3: Build Instruction Block

Structure instructions based on command:

```markdown
## Objective
[Clear single-sentence goal based on user input and command type]

## Steps
[Command-specific steps - these are defined in the command template]

## Boundaries

### DO
- Follow project patterns from basepoints
- Use validation commands: [from project profile]
- [Additional positive guidance]

### DO NOT
[Inject anti-patterns from session learnings]
- [Specific things to avoid based on failed patterns]
```

### Step 4: Build Output Specification

Specify expected output format:

| Command | Output Specification |
|---------|---------------------|
| /shape-spec | `requirements.md` in planning folder |
| /write-spec | `spec.md` with technical specification |
| /create-tasks | `tasks.md` with task breakdown |
| /implement-tasks | Implementation files as specified in tasks |
| /orchestrate-tasks | `orchestration.yml` with task groups |

**Example Output Spec:**

```markdown
## Expected Output

### File
- Path: geist/specs/[date]-[name]/planning/requirements.md
- Format: Markdown with structured sections

### Structure
[Template or example of expected output]

### Handoff
- Next command: /write-spec
- Context to pass: Feature scope, constraints, architecture hints
```

### Step 5: Assemble & Return

Combine all sections into final prompt:

```bash
# Assemble sections
CONTEXT_BLOCK=$(build_context_block "$COMMAND_NAME")
INSTRUCTION_BLOCK=$(build_instruction_block "$COMMAND_NAME" "$USER_INPUT")
OUTPUT_SPEC=$(build_output_spec "$COMMAND_NAME")

# Combine
FINAL_PROMPT="$CONTEXT_BLOCK

$INSTRUCTION_BLOCK

$OUTPUT_SPEC"

# Return for command execution
echo "$FINAL_PROMPT"
```

## Output

Returns optimized prompt that includes:

1. **Relevant context** (not everything, just what's needed for this command)
2. **Clear instructions** with project-specific guidance
3. **Output format** with handoff information

## Integration

Every command template starts with:

```markdown
{{workflows/prompting/construct-prompt}}

# [Command Name]

[Rest of command using constructed context]
```

The workflow injects the optimized prompt context at the top of every command execution.

## Notes

- Only loads context that's relevant to the current command
- Automatically injects successful patterns from session learnings
- Warns against anti-patterns that caused failures
- Prepares handoff context for next command in workflow
