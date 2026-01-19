Now that all module and parent basepoints are generated, proceed with generating the headquarter.md file.

{{workflows/codebase-analysis/generate-headquarter}}

## Display confirmation and next step

Once headquarter is generated, output:

```
✅ Basepoints generation complete!

**Headquarter file**: geist/basepoints/headquarter.md
**Total basepoints**: [number] files (modules + parents)
**Structure**: Complete hierarchy documented
**Abstraction layers**: [number] layers identified

All basepoint documentation is now complete and ready for use!

**Navigation**:
- Start here: geist/basepoints/headquarter.md
- Browse by module: geist/basepoints/
- Browse by layer: [links organized by abstraction layer]
```

## User Standards & Preferences Compliance

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your headquarter generation aligns with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
