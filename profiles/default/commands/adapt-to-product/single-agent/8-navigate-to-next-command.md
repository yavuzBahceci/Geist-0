The adapt-to-product process is now complete. This final phase provides a summary of what was accomplished and guidance on next steps.

## Summary of Accomplishments

Display a comprehensive summary of the adapt-to-product command results:

### Files Created

The following product documentation files have been created:

| File | Location | Purpose |
|------|----------|---------|
| Mission | `geist/product/mission.md` | Product vision, goals, and strategic direction |
| Roadmap | `geist/product/roadmap.md` | Phased development plan with prioritized features |
| Tech Stack | `geist/product/tech-stack.md` | Technical stack and architecture decisions |
| Summary | `geist/output/adapt-to-product/reports/product-summary.md` | Combined knowledge and consistency analysis |

### Process Completed

1. ✅ Setup and Information Gathering
2. ✅ Codebase Analysis
3. ✅ Mission Document Creation
4. ✅ Roadmap Creation
5. ✅ Tech Stack Documentation
6. ✅ Knowledge Review and Combination

## Next Steps

Now that product documentation is complete, the recommended next step is to analyze your codebase structure and create basepoints that document your code patterns.

### Recommended Command

Run the **create-basepoints** command to:
- Mirror your project structure
- Detect abstraction layers
- Analyze code patterns
- Generate module and parent basepoints
- Create a headquarter file that bridges product and code knowledge

## Output

Display the following completion message:

```
🎉 adapt-to-product Complete!

**Product Documentation Created:**
├── geist/product/
│   ├── mission.md       - Product vision and goals
│   ├── roadmap.md       - Development roadmap
│   └── tech-stack.md    - Technical stack
└── geist/output/adapt-to-product/reports/
    └── product-summary.md - Knowledge summary

**What's Next?**

Your product documentation is ready. To continue setting up your Geist:

👉 Run `/create-basepoints` to analyze your codebase and generate pattern documentation.

This will create basepoints that document your code patterns, making it easier for AI agents to understand and work with your codebase.
```

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

Ensure the navigation guidance follows the user's standards and preferences as documented in these files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
