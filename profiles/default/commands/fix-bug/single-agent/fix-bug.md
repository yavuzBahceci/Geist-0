# Fix Bug / Resolve Issue Command

You are helping me analyze and fix a bug or implement feedback. This command accepts bugs or feedbacks via direct prompt input and systematically resolves them through knowledge extraction and iterative fixing.

## Command Interface

This command accepts input in the following formats:

**For Bugs:**
- Error logs and stack traces
- Error codes and messages
- Written bug descriptions
- Bug reports (structured or unstructured)
- Code snippets that produce errors

**For Feedbacks:**
- Feature requests
- Enhancement suggestions
- User feedback about behavior
- Natural language improvement descriptions

## Multi-Phase Process

Follow each of these phases IN SEQUENCE:

{{PHASE 1: @agent-os/commands/fix-bug/1-analyze-issue.md}}

{{PHASE 2: @agent-os/commands/fix-bug/2-research-libraries.md}}

{{PHASE 3: @agent-os/commands/fix-bug/3-integrate-basepoints.md}}

{{PHASE 4: @agent-os/commands/fix-bug/4-analyze-code.md}}

{{PHASE 5: @agent-os/commands/fix-bug/5-synthesize-knowledge.md}}

{{PHASE 6: @agent-os/commands/fix-bug/6-implement-fix.md}}

## Important Notes

- This command combines analysis and fix implementation in one workflow
- The iterative fix implementation continues if getting closer to solution
- After 3 worsening results, the command stops and requests guidance
- All knowledge is synthesized before attempting fixes
- Library-specific patterns and basepoints standards guide the fix implementation
