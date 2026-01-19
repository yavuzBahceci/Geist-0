# Phase 7: Navigate to Cleanup

The deploy-agents process is now complete. This final phase provides a summary of what was accomplished and guidance on next steps.

## Summary of Accomplishments

Display a comprehensive summary of the deploy-agents command results:

### Phases Completed

1. ✅ Validate Prerequisites
2. ✅ Extract Basepoints Knowledge
3. ✅ Extract Product Knowledge
4. ✅ Merge Knowledge
5. ✅ Specialize Standards
6. ✅ Specialize Agents

### What Was Specialized

| Category | Location | Description |
|----------|----------|-------------|
| Standards | `geist/standards/` | Updated with project-specific patterns |
| Agents | `geist/agents/specialists/` | Specialist agents for abstraction layers |

### Reports Generated

Reports are available in `geist/output/deploy-agents/`:
- `knowledge/basepoints-knowledge.json` - Extracted patterns
- `knowledge/product-knowledge.json` - Product context
- `knowledge/merged-knowledge.md` - Combined knowledge
- `reports/standards-specialization.md` - Standards changes
- `reports/agents-specialization.md` - Agents created

## Next Steps

Now that your Geist is specialized, the recommended next step is to validate the deployment and clean up any remaining issues.

### Recommended Command

Run the **cleanup-geist** command to:
- Verify all placeholders are properly replaced
- Check for broken file references
- Ensure knowledge completeness
- Generate a cleanup report

## Output

Display the following completion message:

```
═══════════════════════════════════════════════════════════
  DEPLOY-AGENTS COMPLETE
═══════════════════════════════════════════════════════════

🎉 Your Geist has been specialized for this project!

**What was specialized:**
├── geist/standards/           (project-specific patterns)
│   ├── coding-style.md
│   ├── conventions.md
│   └── [other standards...]
└── geist/agents/specialists/  (layer-specific agents)
    ├── [layer]-specialist.md
    └── registry.yml

**Knowledge extracted:**
└── geist/output/deploy-agents/
    ├── knowledge/
    │   ├── basepoints-knowledge.json
    │   ├── product-knowledge.json
    │   └── merged-knowledge.md
    └── reports/
        ├── standards-specialization.md
        └── agents-specialization.md

═══════════════════════════════════════════════════════════
  NEXT STEP
═══════════════════════════════════════════════════════════

👉 Run `/cleanup-geist` to validate your deployment.

This will:
• Verify all placeholders are properly replaced
• Check for broken file references
• Ensure knowledge completeness
• Generate a cleanup report

After cleanup, your Geist is ready to use!
```

---

## User Standards & Preferences Compliance

Ensure the navigation guidance follows the user's standards:

@geist/standards/global/conventions.md
