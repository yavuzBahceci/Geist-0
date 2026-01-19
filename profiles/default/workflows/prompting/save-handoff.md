# Save Handoff Context

Called at the END of every command to save context for the next command.

## Purpose

- Capture what was completed in the current command
- Prepare context for the next command in the workflow
- Enable seamless continuation between commands

## When Called

- At the END of every command execution
- After command logic completes successfully
- Before command returns/finishes

## Process

### Step 1: Determine Handoff Target

Identify which command typically follows the current command:

| Current Command | Next Command | Context to Pass |
|----------------|--------------|-----------------|
| /shape-spec | /write-spec | Requirements summary, features, constraints |
| /write-spec | /create-tasks | Spec summary, components, complexity |
| /create-tasks | /implement-tasks or /orchestrate-tasks | Task list, dependencies, sequence |
| /orchestrate-tasks | /implement-tasks | Task groups, assignments, standards |
| /implement-tasks | (none or /verify-implementation) | Implementation summary, outcomes |

### Step 2: Extract Key Information

From the current command's output, extract:

- **What was completed**: Files created/modified, outcomes achieved
- **Key decisions**: Important choices made, constraints identified
- **Context for next**: What the next command needs to know

### Step 3: Build Handoff Document

Create handoff markdown file:

```markdown
# Handoff: [from-command] → [to-command]

## Completed
[Summary of what was accomplished in current command]

## Key Decisions
[Important choices, constraints, patterns identified]

## For [to-command]
[Specific context needed for next command]
- Focus areas: [list]
- Architecture hints: [from patterns]
- Complexity: [assessment]
- Dependencies: [list]

## Timestamp
[ISO 8601 timestamp]
```

### Step 4: Save to Handoff Location

```bash
# Create handoff directory if it doesn't exist
mkdir -p geist/output/handoff

# Save handoff context
cat > geist/output/handoff/current.md << EOF
[Handoff document content]
EOF

echo "✅ Handoff context saved for $TO_COMMAND"
```

## Usage in Commands

At the end of each command template:

```markdown
## Step N: Save Handoff

{{workflows/prompting/save-handoff}}
```

The workflow automatically:
1. Detects current command name
2. Determines next command in sequence
3. Extracts relevant context from current output
4. Saves handoff document

## Handoff Templates

### shape-spec → write-spec

```markdown
# Handoff: shape-spec → write-spec

## Completed
- Requirements documented in `geist/specs/[date]-[name]/planning/requirements.md`
- Key features identified: [feature list]
- Constraints established: [constraint list]

## Key Decisions
- Architecture approach: [choice]
- Technology selection: [if relevant]
- Complexity level: [simple/moderate/complex]

## For write-spec
- Focus areas: [main areas from requirements]
- Architecture hints: [patterns from basepoints that apply]
- Complexity assessment: [simple/moderate/complex]
- Standards to reference: [list]
```

### write-spec → create-tasks

```markdown
# Handoff: write-spec → create-tasks

## Completed
- Detailed specification in `geist/specs/[date]-[name]/spec.md`
- Architecture defined: [components, layers]
- Interfaces documented: [APIs, contracts]

## Key Decisions
- Implementation approach: [choice]
- Layer assignments: [if relevant]
- Dependency structure: [if relevant]

## For create-tasks
- Task granularity: [based on complexity]
- Suggested groupings: [by feature/layer/domain]
- Dependencies identified: [list of task dependencies]
- Estimated complexity: [overall assessment]
```

### create-tasks → implement-tasks

```markdown
# Handoff: create-tasks → implement-tasks

## Completed
- Tasks listed in `geist/specs/[date]-[name]/tasks.md`
- Dependencies mapped: [dependency graph]
- Complexity estimated: [per task group]

## Key Decisions
- Task grouping strategy: [approach]
- Implementation order: [sequence]

## For implement-tasks
- Implementation order: [recommended sequence]
- Layer specialists: [suggested assignments if available]
- Validation commands: [from project profile]
- Critical tasks: [tasks requiring special attention]
```

## Integration

Handoff context is automatically loaded by `construct-prompt.md` at the start of the next command, ensuring seamless context flow between commands.
