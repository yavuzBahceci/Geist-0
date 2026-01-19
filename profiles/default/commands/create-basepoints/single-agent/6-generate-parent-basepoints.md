Now that module basepoints are generated, proceed with generating basepoint files for parent folders.

{{workflows/codebase-analysis/generate-parent-basepoints}}

## Display confirmation and next step

Once parent basepoints are generated, output:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 PARENT BASEPOINTS GENERATION COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Parent basepoints created: [number] files
✅ Hierarchy levels: [number] levels
✅ Root-level basepoint: geist/basepoints/agent-base-[root-name].md

Parent basepoints:
[List all parent basepoint files]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 COMPLETE BASEPOINTS SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Total basepoint files:
- Module basepoints: [number]
- Parent basepoints: [number]
- Total: [number]

All basepoint files (modules + parents) are now complete.

NEXT STEP 👉 Run the command, `7-generate-headquarter.md`
```

## User Standards & Preferences Compliance

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your parent basepoint generation aligns with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
