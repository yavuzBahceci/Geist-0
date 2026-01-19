The FIRST STEP is to detect codebase changes since the last update by following these instructions:

{{workflows/codebase-analysis/detect-codebase-changes}}

## Phase 1 Actions

### 1.1 Initialize Output Structure

Create the output cache directory structure:

```bash
mkdir -p geist/output/update-basepoints-and-redeploy/cache
mkdir -p geist/output/update-basepoints-and-redeploy/reports
```

### 1.2 Detect Changes

Use the workflow to detect changes. The workflow will:

1. **Check for git availability** - Prefer git-based detection
2. **Load previous update reference** - From `cache/last-update-commit.txt` or `cache/last-update-timestamp.txt`
3. **Detect and categorize changes** - Added, modified, deleted files
4. **Filter irrelevant files** - Exclude node_modules, .git, geist/output, etc.
5. **Identify product file changes** - Flag if `geist/product/*` files changed

### 1.3 Handle First-Run Scenario

If this is the first run (no previous update reference exists):
- Treat ALL relevant files as "added"
- This will trigger a full basepoint update (similar to initial creation)
- Display appropriate message to user

### 1.4 Handle No-Changes Scenario

If no changes are detected:
- Display "No changes detected" message
- Offer options: Exit or Force refresh
- If user chooses to exit, stop the command gracefully

## Expected Outputs

After this phase, the following files should exist in `geist/output/update-basepoints-and-redeploy/cache/`:

| File | Description |
|------|-------------|
| `changed-files.txt` | Combined list of all changed files |
| `added-files.txt` | Files that were added |
| `modified-files.txt` | Files that were modified |
| `deleted-files.txt` | Files that were deleted |
| `change-summary.md` | Human-readable summary report |
| `product-files-changed.txt` | Boolean flag for product changes |

## Display confirmation and next step

Once change detection is complete, output the following message:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PHASE 1 COMPLETE: Change Detection
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 Changes Detected:
   Added:    [N] files
   Modified: [N] files
   Deleted:  [N] files
   Total:    [N] files

📦 Product Files Changed: [Yes/No]

📋 Summary: geist/output/update-basepoints-and-redeploy/cache/change-summary.md

NEXT STEP 👉 Run Phase 2: `2-identify-affected-basepoints.md`
```

If no changes detected:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ NO CHANGES DETECTED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

No codebase changes found since last update.

Options:
1. Exit (no update needed)
2. Force full refresh

👉 What would you like to do?
```

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your change detection process aligns with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
