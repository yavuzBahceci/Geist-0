# Command Reference: fix-bug

## Overview

The `fix-bug` command is a comprehensive error/issue analysis and fixing command that accepts bugs or feedbacks via direct prompt input and systematically resolves them through knowledge extraction and iterative fixing.

## Location

`commands/fix-bug/single-agent/`

## Prerequisites

- Deployed Agent OS (for basepoints integration)
- Bug report, error log, or feedback description

## Input Formats

### For Bugs
- Error logs and stack traces
- Error codes and messages
- Written bug descriptions
- Bug reports (structured or unstructured)
- Code snippets that produce errors

### For Feedbacks
- Feature requests
- Enhancement suggestions
- User feedback about behavior
- Natural language improvement descriptions

## Outputs

### On Success
- `agent-os/output/fix-bug/cache/issue-analysis.md`
- `agent-os/output/fix-bug/cache/library-research.md`
- `agent-os/output/fix-bug/cache/basepoints-integration.md`
- `agent-os/output/fix-bug/cache/code-analysis.md`
- `agent-os/output/fix-bug/cache/knowledge-synthesis.md`
- `agent-os/output/fix-bug/cache/fix-report.md`
- Code changes that resolve the issue

### On Stop Condition (3 Worsening Results)
- All analysis files above
- `agent-os/output/fix-bug/cache/guidance-request.md`

## Phases

### Phase 1: Issue Analysis
- Parse input (bug/feedback)
- Extract details from error logs, codes, descriptions
- Identify affected libraries and modules
- Support multiple input formats

### Phase 2: Library Research
- Deep-dive research on related libraries
- Research internal architecture and workflows
- Research known issues and bug patterns
- Understand library component interactions in error scenarios

### Phase 3: Basepoints Integration
- Extract relevant basepoints knowledge
- Find basepoints describing error location
- Extract patterns and standards related to error context
- Identify similar issues in basepoints

### Phase 4: Code Analysis
- Identify exact file/module locations from error logs
- Deep-dive into relevant code files
- Analyze code patterns and flows in error context
- Trace execution paths leading to error

### Phase 5: Knowledge Synthesis
- Combine all knowledge sources into comprehensive analysis
- Create unified view of issue context
- Prepare knowledge for fix implementation

### Phase 6: Iterative Fix Implementation
- Use synthesized knowledge to implement initial fix
- Apply library-specific patterns and best practices
- Follow basepoints patterns and standards
- Iterative refinement loop:
  - If getting closer: continue iterating
  - If worsening: track counter
  - Stop after 3 worsening results
- Present knowledge summary and request guidance when stop condition met

## Usage

```bash
# Direct command with bug description
/fix-bug "Error: Cannot read property 'x' of undefined at line 42 in src/utils.ts"

# With stack trace
/fix-bug "
Traceback (most recent call last):
  File 'main.py', line 10, in <module>
    result = process_data(data)
  File 'processor.py', line 25, in process_data
    return data['key']
KeyError: 'key'
"

# With feedback
/fix-bug "The search feature should support fuzzy matching for better user experience"
```

## Iterative Fix Behavior

### Continue Iterating When
- Error count decreases (getting closer)
- Partial success achieved
- Different but related errors appear

### Stop and Request Guidance When
- 3 consecutive worsening results
- Error count increases or stays same 3 times
- No progress after multiple attempts

### Guidance Request Format
When stop condition is met, the command outputs:
1. Knowledge summary from all phases
2. What was tried and what happened
3. Current state of errors
4. Specific questions for user guidance

## Integration with Other Commands

| Integration | Description |
|-------------|-------------|
| Basepoints | Uses basepoints knowledge for context |
| Library Basepoints | Uses library patterns and troubleshooting |
| Standards | Follows project standards during fix |
| Validation | Runs validation after each fix attempt |

## Best Practices

1. **Provide Complete Error Information**: Include full stack traces and error messages
2. **Describe Context**: Explain what you were doing when the error occurred
3. **Include Reproduction Steps**: If known, include steps to reproduce
4. **Be Specific**: The more specific the input, the better the analysis

## Related Commands

- `/implement-tasks` - For implementing new features
- `/update-basepoints-and-redeploy` - After fixing, update basepoints if patterns changed

---

*Last Updated: 2026-01-18*
