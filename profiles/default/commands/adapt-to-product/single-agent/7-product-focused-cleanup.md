Now that product files (mission, tech-stack, roadmap) have been created, run the product-focused cleanup workflow to simplify and enhance geist files based on the detected product scope.

## Core Responsibilities

1. **Run Cleanup Workflow**: Execute the product-focused cleanup workflow to process geist files
2. **Review Results**: Present the cleanup report to the user for review
3. **Proceed to Next Phase**: Guide user to the final phase

## Workflow

Execute the product-focused cleanup workflow:

{{workflows/cleanup/product-focused-cleanup}}

## Display Confirmation and Next Step

Once the cleanup workflow is complete, output the following message:

```
✅ Product-Focused Cleanup Complete!

📊 Detected Scope:
   Language: [detected language]
   Frameworks: [detected frameworks]
   Project Type: [detected project type]
   Architecture: [detected architecture]

🧹 Phase A (Simplify):
   Files reviewed: [count]
   Content flagged: [count]

📚 Phase B (Expand):
   Enhancement rules defined
   Web search queries generated

📁 Reports Generated:
   - geist/output/product-cleanup/detected-scope.yml
   - geist/output/product-cleanup/search-queries.md
   - geist/output/product-cleanup/cleanup-report.md

NEXT STEP 👉 Proceeding to Phase 8: Navigate to Next Command
```

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

Ensure cleanup follows the user's preferences:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
