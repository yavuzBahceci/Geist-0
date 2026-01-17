You are helping to clean up and fix issues in an already-deployed agent-os instance. This includes:

- **Validation**: Run comprehensive validation to identify issues (placeholders, unnecessary logic, broken references)
- **Knowledge Verification**: Verify that sufficient knowledge has been extracted and no important information is missing
- **Placeholder Cleaning**: Replace remaining placeholders with project-specific content
- **Unnecessary Logic Removal**: Remove project-agnostic conditionals, generic examples, and profiles/default references
  - **Conditional Cleanup**: Remove resolved {{IF}}/{{UNLESS}} blocks that are no longer needed after specialization
  - **Pattern Conversion**: Convert generic patterns to project-specific ones from basepoints where possible
  - **Abstraction Simplification**: Abstract redundant patterns and remove unnecessary complexity
  - **Technology-Agnostic Cleanup**: Remove abstraction layers that were needed for templates but not for specialized commands
- **Broken Reference Fixing**: Fix broken command cycle references and align commands with current project structure
- **Reporting**: Generate comprehensive cleanup report showing what was converted, abstracted, or removed

Carefully read and execute the instructions in the following files IN SEQUENCE, following their numbered file names. Only proceed to the next numbered instruction file once the previous numbered instruction has been executed.

Instructions to follow in sequence:

{{PHASE 1: @agent-os/commands/cleanup-agent-os/1-validate-prerequisites-and-run-validation.md}}

{{PHASE 2: @agent-os/commands/cleanup-agent-os/2-clean-placeholders.md}}

{{PHASE 3: @agent-os/commands/cleanup-agent-os/3-remove-unnecessary-logic.md}}

{{PHASE 4: @agent-os/commands/cleanup-agent-os/4-fix-broken-references.md}}

{{PHASE 5: @agent-os/commands/cleanup-agent-os/5-verify-knowledge-completeness.md}}

{{PHASE 6: @agent-os/commands/cleanup-agent-os/6-generate-cleanup-report.md}}
