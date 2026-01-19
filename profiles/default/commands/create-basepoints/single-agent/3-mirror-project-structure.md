Now that abstraction layers are detected, proceed with mirroring the project structure in the basepoints folder.

{{workflows/codebase-analysis/mirror-project-structure}}

{{UNLESS compiled_single_command}}
## Display Task List for Approval
{{ENDUNLESS compiled_single_command}}

After structure is mirrored and task list is created, **PRESENT THE TASK LIST TO THE USER FOR APPROVAL**:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 BASEPOINTS TASK LIST - REVIEW REQUIRED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

The following modules were detected and will have basepoints generated:

[Display full content of geist/output/create-basepoints/cache/basepoints-task-list.md]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  VERIFICATION CHECKPOINT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Please review the task list above and verify:
1. All important modules are included
2. No modules are missing
3. The paths look correct

**If any modules are MISSING**, please identify them now.

Proceed with basepoint generation? (y/n)
```

**IMPORTANT**: 
- **WAIT for user approval** before proceeding
- If user identifies missing modules, add them to the task list
- Only proceed to next step after user confirms the list is complete

{{UNLESS compiled_single_command}}
## Display confirmation and next step
{{ENDUNLESS compiled_single_command}}

Once structure is mirrored AND user approves the task list, output:

```
✅ Project structure mirrored to basepoints folder!
✅ Task list approved by user!

**Created structure**: geist/basepoints/
**Module folders identified**: [number] folders
**Excluded folders**: [list of excluded patterns]
**Task list**: geist/output/create-basepoints/cache/basepoints-task-list.md

Structure is ready for codebase analysis.

NEXT STEP 👉 Run the command, `4-analyze-codebase.md`
```

## User Standards & Preferences Compliance

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your structure mirroring aligns with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
