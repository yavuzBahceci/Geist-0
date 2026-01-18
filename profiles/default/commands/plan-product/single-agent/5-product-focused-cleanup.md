Now that product files (mission, tech-stack, roadmap) have been created, run the product-focused cleanup workflow to simplify and enhance agent-os files based on the detected product scope.

## Core Responsibilities

1. **Run Cleanup Workflow**: Execute the product-focused cleanup workflow to process agent-os files
2. **Review Results**: Present the cleanup report to the user for review
3. **Complete Plan-Product**: Finalize the plan-product command

## Workflow

Execute the product-focused cleanup workflow:

{{workflows/cleanup/product-focused-cleanup}}

## Display Confirmation and Completion

Once the cleanup workflow is complete, output the following message:

```
🎉 plan-product Complete!

**Product Documentation Created:**
├── agent-os/product/
│   ├── mission.md       - Product vision and goals
│   ├── roadmap.md       - Development roadmap
│   └── tech-stack.md    - Technical stack

**Product-Focused Cleanup Applied:**
├── agent-os/output/product-cleanup/
│   ├── detected-scope.yml    - Detected product scope
│   ├── search-queries.md     - Web search queries for enhancement
│   └── cleanup-report.md     - Cleanup/enhancement report

📊 Detected Scope:
   Language: [detected language]
   Frameworks: [detected frameworks]
   Project Type: [detected project type]
   Architecture: [detected architecture]

**What's Next?**

Your product documentation is ready and agent-os files have been cleaned/enhanced for your specific product scope.

👉 Run `/create-basepoints` to analyze your codebase and generate pattern documentation.

This will create basepoints that document your code patterns, making it easier for AI agents to understand and work with your codebase.
```

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

Ensure cleanup follows the user's preferences:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
