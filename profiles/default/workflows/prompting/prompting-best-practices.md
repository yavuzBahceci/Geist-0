# Prompting Best Practices

Reference guide for constructing effective prompts in command templates.

## Structure Template

```markdown
# Command: [Name]

## Context
[Relevant basepoints, project info]

## Objective
[Clear statement of what to achieve]

## Constraints
- DO: [positive guidance]
- DO NOT: [explicit restrictions]

## Steps
1. [First action]
2. [Second action]
3. [...]

## Output
- File: [path]
- Format: [description]

## Success Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]
```

## Context Types by Command

| Command | Essential Context |
|---------|-------------------|
| /shape-spec | Tech stack, architecture patterns, similar features |
| /write-spec | Existing specs, standards, requirements |
| /create-tasks | Spec details, complexity level, dependencies |
| /implement-tasks | Basepoints, validation commands, layer specialists |
| /orchestrate-tasks | Task dependencies, specialist availability |

## Context Loading Guidelines

### Always Include

1. **Project Identity** (from headquarter.md)
   - Tech stack
   - Architecture style
   - Key patterns

2. **Relevant Patterns** (from session learnings)
   - Successful patterns that apply
   - Anti-patterns to avoid

3. **Standards** (from project profile)
   - Coding conventions
   - Validation commands
   - Testing requirements

### Command-Specific Context

**For /shape-spec:**
- Similar features already implemented
- Architecture patterns used
- Tech stack constraints

**For /write-spec:**
- Existing specifications (for consistency)
- Standards to follow
- Component patterns

**For /implement-tasks:**
- Basepoints for relevant layers
- Validation commands
- Layer specialist assignments
- Session patterns (what worked/failed)

## Common Issues to Fix

| Issue | Problem | Solution |
|-------|---------|----------|
| Vague scope | AI does too much/little | Add explicit boundaries with DO/DO NOT |
| Missing context | AI makes wrong assumptions | Add basepoints reference and project info |
| No constraints | AI makes avoidable mistakes | Add "DO NOT" section with anti-patterns |
| Unclear output | AI creates wrong artifacts | Specify exact file paths and formats |
| No handoff | Next command lacks context | Always specify handoff information |

## Prompt Optimization Checklist

Before finalizing a command prompt:

- [ ] **Context loaded** from basepoints + learnings
- [ ] **Relevant patterns** injected (not everything, just what applies)
- [ ] **Anti-patterns** warned against (from session failures)
- [ ] **Clear objective** stated (single sentence goal)
- [ ] **Steps are numbered** and specific (with expected outputs)
- [ ] **Boundaries explicitly** defined (DO/DO NOT sections)
- [ ] **Output format** specified (exact paths, formats)
- [ ] **Handoff context** prepared (what next command needs)
- [ ] **Validation criteria** included (how to verify success)

## Pattern Injection Guidelines

### Successful Patterns

When a pattern has:
- 100% success rate
- 3+ uses

Inject it as positive guidance:

```markdown
### DO
- Use [pattern] for [use case]
  (Pattern has 100% success rate across 5 implementations)
```

### Failed Patterns (Anti-patterns)

When a pattern has:
- Any failure
- Known issues

Warn against it explicitly:

```markdown
### DO NOT
- Avoid [anti-pattern]
  (This caused failures in: [examples])
  Use [alternative pattern] instead.
```

## Command Flow Optimization

### shape-spec

**Context Needed:**
- Tech stack from headquarter.md
- Similar features from codebase
- Architecture patterns

**Patterns to Inject:**
- How similar features were structured
- Common constraints for this project type

### write-spec

**Context Needed:**
- Requirements from previous command
- Existing spec patterns
- Standards to follow

**Patterns to Inject:**
- Spec structure used in project
- Component documentation style

### implement-tasks

**Context Needed:**
- Basepoints for relevant layers
- Validation commands
- Session learnings (successful/failed patterns)

**Patterns to Inject:**
- Patterns that worked for this layer
- Anti-patterns that caused failures
- Validation requirements

## Handoff Best Practices

Always prepare handoff context for next command:

1. **Summarize** what was completed
2. **Highlight** key decisions/constraints
3. **Provide** specific context needed
4. **Suggest** next steps if helpful

Example:

```markdown
## Handoff: shape-spec → write-spec

### Completed
- Requirements in requirements.md
- Features: [list]
- Constraints: [list]

### For write-spec
- Focus: [main areas]
- Architecture: [hints from patterns]
- Complexity: [assessment]
```

## Continuous Improvement

Prompts are optimized based on:

1. **First Installation** (`/deploy-agents` Phase 11)
   - Project-specific patterns
   - Tech stack guidance
   - Architecture constraints

2. **Session Learning** (`/update-basepoints-and-redeploy` Phase 7)
   - What worked well
   - What caused failures
   - Pattern effectiveness

The prompting specialist uses this feedback to refine prompts over time.
