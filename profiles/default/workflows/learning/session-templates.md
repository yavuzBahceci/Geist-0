# Session Feedback Templates

Templates for session feedback data structures created at runtime in `geist/output/session-feedback/`.

## Current Session Template

**File**: `geist/output/session-feedback/current-session.md`

```markdown
# Session: YYYY-MM-DD

## Metadata
- Started: YYYY-MM-DDTHH:MM:SS
- Last Updated: YYYY-MM-DDTHH:MM:SS
- Implementations: 0
- Success Rate: 0%

## Implementations

| # | Spec | Tasks | Outcome | Duration | Notes |
|---|------|-------|---------|----------|-------|

## Patterns Observed

### Successful
(Patterns will be listed here as they are detected)

### Failed
(Failed patterns will be listed here as they are detected)

## Prompt Effectiveness

### Worked Well
(Prompts that worked well will be listed here)

### Needed Clarification
(Prompts that needed clarification will be listed here)
```

## Successful Patterns Template

**File**: `geist/output/session-feedback/patterns/successful.md`

```markdown
# Successful Patterns

## Pattern: [Pattern Name]

**Description**: [Brief description of the pattern]

**First Observed**: YYYY-MM-DD
**Times Used**: 0
**Success Rate**: 100%

**Example**:
```[language]
[Code example showing the pattern]
```

**Applicable To**: [Use cases, layers, contexts where this pattern applies]

---

(Additional patterns will be added here)
```

## Failed Patterns Template

**File**: `geist/output/session-feedback/patterns/failed.md`

```markdown
# Failed Patterns (Anti-patterns)

## Pattern: [Anti-pattern Name]

**Description**: [Description of what was attempted and why it failed]

**First Observed**: YYYY-MM-DD
**Times Used**: 0
**Failure Rate**: 100%

**Error Details**:
- Implementation: [spec/task name where it failed]
- Error: [Error message or issue description]
- Fix Applied: [How it was resolved]

**Example (What NOT to do)**:
```[language]
[Code example showing the anti-pattern]
```

**Recommended Alternative**:
```[language]
[Code example showing the correct approach]
```

**Applicable To**: [Contexts where this anti-pattern should be avoided]

---

(Additional anti-patterns will be added here)
```

## Usage Notes

These templates are used by workflows to create session feedback files at runtime:

1. **current-session.md**: Created/updated after each `/implement-tasks` run
2. **patterns/successful.md**: Populated by `extract-session-patterns` workflow
3. **patterns/failed.md**: Populated by `extract-session-patterns` workflow

These files are created dynamically in target projects' `geist/output/session-feedback/` directory when commands run.
