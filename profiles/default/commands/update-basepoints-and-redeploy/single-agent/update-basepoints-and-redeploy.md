You are helping to incrementally update basepoints after codebase changes and re-specialize agent-os with updated knowledge. This is more efficient than full regeneration when only some files have changed. The process includes:

- **Change Detection**: Detect codebase changes since last update using git or timestamps
- **Basepoint Identification**: Map changed files to their affected basepoints
- **Incremental Update**: Update only affected basepoints (not entire codebase)
- **Knowledge Re-extraction**: Re-extract and merge knowledge from updated basepoints
- **Re-specialization**: Re-specialize ALL core commands with updated knowledge
- **Validation**: Validate all updates and generate comprehensive report

This command is designed to be run after making changes to your codebase, ensuring your agent-os stays synchronized with your project's current state without the overhead of full regeneration.

Carefully read and execute the instructions in the following files IN SEQUENCE, following their numbered file names. Only proceed to the next numbered instruction file once the previous numbered instruction has been executed.

Instructions to follow in sequence:

{{PHASE 1: @agent-os/commands/update-basepoints-and-redeploy/1-detect-changes.md}}

{{PHASE 2: @agent-os/commands/update-basepoints-and-redeploy/2-identify-affected-basepoints.md}}

{{PHASE 3: @agent-os/commands/update-basepoints-and-redeploy/3-update-basepoints.md}}

{{PHASE 4: @agent-os/commands/update-basepoints-and-redeploy/4-re-extract-knowledge.md}}

{{PHASE 5: @agent-os/commands/update-basepoints-and-redeploy/5-selective-respecialize.md}}

{{PHASE 6: @agent-os/commands/update-basepoints-and-redeploy/6-validate-and-report.md}}

{{PHASE 7: @agent-os/commands/update-basepoints-and-redeploy/7-review-session-learnings.md}}

{{PHASE 8: @agent-os/commands/update-basepoints-and-redeploy/8-adapt-commands.md}}
