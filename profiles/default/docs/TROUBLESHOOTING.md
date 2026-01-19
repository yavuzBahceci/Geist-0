# Troubleshooting Guide

This guide helps you diagnose and fix common issues when using Geist.

---

## Table of Contents

- [Common Errors](#common-errors)
- [FAQ](#faq)
- [Debug Tips](#debug-tips)
- [Getting Help](#getting-help)

---

## Common Errors

### "Command not found" Errors

**Symptom**: Running `/shape-spec` shows "command not found" or similar

**Cause**: Commands not installed or path incorrect

**Solution**:
1. Verify `geist/commands/` exists in your project
2. Check your AI tool's command configuration points to `geist/commands/`
3. Re-run installation script:
   ```bash
   ~/geist/scripts/project-install.sh --profile default
   ```
4. If using Claude Code, verify commands are registered

---

### "File not found" Errors

**Symptom**: Command fails with "cannot find file X" or "file does not exist"

**Cause**: Missing prerequisite files from previous commands

**Solution**:
1. **Check command order** - Commands must run in sequence:
   - Setup: `adapt-to-product` → `create-basepoints` → `deploy-agents`
   - Development: `shape-spec` → `write-spec` → `create-tasks` → `implement-tasks`

2. **Verify the file exists**:
   ```bash
   ls -la geist/[path-from-error]
   ```

3. **Run prerequisite command**:
   - Missing `product/tech-stack.md`? Run `/adapt-to-product`
   - Missing `basepoints/headquarter.md`? Run `/create-basepoints`
   - Missing `specs/[name]/requirements.md`? Run `/shape-spec`

---

### "Placeholder not replaced" Errors

**Symptom**: Commands contain `{{PLACEHOLDER}}` text or `{{PROJECT_BUILD_COMMAND}}`

**Cause**: Specialization incomplete or not run

**Solution**:
1. Run `/deploy-agents` to specialize commands
2. Run `/cleanup-geist` to verify and clean remaining placeholders
3. If placeholders persist, manually replace:
   - `{{PROJECT_BUILD_COMMAND}}` → your build command (e.g., `npm run build`)
   - `{{PROJECT_TEST_COMMAND}}` → your test command (e.g., `npm test`)
   - `{{PROJECT_LINT_COMMAND}}` → your lint command (e.g., `npm run lint`)

---

### Basepoints Not Loading

**Symptom**: Commands don't use your patterns, generic output

**Cause**: Basepoints not created or not found

**Solution**:
1. **Verify basepoints exist**:
   ```bash
   ls -la geist/basepoints/
   ```
   Should see `headquarter.md` and module folders

2. **Check headquarter.md exists**:
   ```bash
   cat geist/basepoints/headquarter.md
   ```

3. **Re-create basepoints**:
   ```bash
   /create-basepoints
   ```

4. **Check cache files**:
   ```bash
   ls -la geist/specs/[your-spec]/implementation/cache/
   ```
   Should see `basepoints-knowledge.md`

---

### Library Knowledge Not Available

**Symptom**: Commands don't know about your libraries

**Cause**: Library basepoints not generated

**Solution**:
1. **Check library basepoints exist**:
   ```bash
   ls -la geist/basepoints/libraries/
   ```

2. **If missing, re-run create-basepoints**:
   ```bash
   /create-basepoints
   ```
   Phase 8 generates library basepoints

3. **Check tech-stack.md lists your libraries**:
   ```bash
   cat geist/product/tech-stack.md
   ```

---

### Spec Not Using Accumulated Knowledge

**Symptom**: `write-spec` doesn't use context from `shape-spec`

**Cause**: Knowledge accumulation not working

**Solution**:
1. **Check accumulated-knowledge.md exists**:
   ```bash
   cat geist/specs/[your-spec]/implementation/cache/accumulated-knowledge.md
   ```

2. **Verify shape-spec completed**:
   - Check `requirements.md` exists
   - Check `basepoints-knowledge.md` exists in cache

3. **Re-run shape-spec** if files are missing

---

### Validation Failing

**Symptom**: Implementation fails validation (build/test/lint errors)

**Cause**: Code issues or incorrect validation commands

**Solution**:
1. **Check validation commands are correct**:
   ```bash
   grep -r "BUILD_CMD\|TEST_CMD\|LINT_CMD" geist/
   ```

2. **Verify commands work manually**:
   ```bash
   npm run build  # or your build command
   npm test       # or your test command
   npm run lint   # or your lint command
   ```

3. **Fix code issues** reported in validation output

4. **Re-specialize if commands wrong**:
   ```bash
   /deploy-agents
   ```

---

## FAQ

### "How do I update basepoints after code changes?"

Run `/update-basepoints-and-redeploy`. This command:
1. Detects changed files (using git diff or timestamps)
2. Identifies affected basepoints
3. Updates only the affected basepoints
4. Re-extracts knowledge
5. Re-specializes commands

```bash
/update-basepoints-and-redeploy
```

---

### "How do I add a new command?"

See `CONTRIBUTING.md` for a detailed guide. Quick steps:

1. **Create directory structure**:
   ```
   profiles/default/commands/[command-name]/
   └── single-agent/
       ├── [command-name].md      # Main entry point
       ├── 1-[first-phase].md     # Phase 1
       ├── 2-[second-phase].md    # Phase 2
       └── ...
   ```

2. **Create main command file** with:
   - Title and description
   - Prerequisites
   - Phase references

3. **Create phase files** with:
   - Clear phase title
   - Step-by-step instructions
   - Workflow references

4. **Register in documentation**:
   - Add to `COMMAND-FLOWS.md`
   - Add to `WORKFLOW-MAP.md`

---

### "Why is my spec not using my patterns?"

Check these in order:

1. **Basepoints exist and are up-to-date**:
   ```bash
   ls -la geist/basepoints/
   ```

2. **The relevant module has a basepoint**:
   - Look for `agent-base-[module].md` in the appropriate layer folder

3. **The pattern is documented in the basepoint**:
   - Open the basepoint file
   - Check "Patterns" or "Standards" sections

4. **Refresh basepoints**:
   ```bash
   /update-basepoints-and-redeploy
   ```

---

### "How do I debug a failing command?"

1. **Check prerequisites**:
   - Read the command's documentation in `COMMAND-FLOWS.md`
   - Verify all required files exist

2. **Check intermediate files**:
   ```bash
   ls -la geist/specs/[your-spec]/implementation/cache/
   ```

3. **Look for error messages**:
   - Commands log progress with markers like "✅", "❌", "⚠️"
   - Look for "Error:", "Failed:", "Cannot find"

4. **Check validation report**:
   ```bash
   cat geist/specs/[your-spec]/implementation/cache/validation-report.md
   ```

---

### "How do I start fresh?"

To reset and start over:

1. **Remove geist folder**:
   ```bash
   rm -rf geist/
   ```

2. **Re-install**:
   ```bash
   ~/geist/scripts/project-install.sh --profile default
   ```

3. **Re-run setup commands**:
   ```bash
   /adapt-to-product
   /create-basepoints
   /deploy-agents
   ```

---

## Debug Tips

### Check if basepoints are loaded

Look for these indicators in command output:
- "📖 Extracting basepoints knowledge..."
- "✅ Found X relevant basepoints"
- "✅ Loaded headquarter"

Check the cache file:
```bash
cat geist/specs/[your-spec]/implementation/cache/basepoints-knowledge.md
```

Should contain extracted patterns and standards.

---

### Verify specialization worked

1. **Open any command file**:
   ```bash
   cat geist/commands/shape-spec/single-agent/shape-spec.md
   ```

2. **Search for placeholders**:
   ```bash
   grep -r "{{" geist/commands/ | grep -v "workflows/" | grep -v "standards/"
   ```
   Should find nothing (except intentional workflow/standards references)

3. **Check validation commands**:
   ```bash
   grep -r "BUILD_CMD\|TEST_CMD" geist/
   ```
   Should show your actual commands, not placeholders

---

### Trace workflow execution

Commands log their progress. Look for:

1. **Phase markers**:
   ```
   ═══════════════════════════════════════════════════════
     PHASE 1: Initialize Spec
   ═══════════════════════════════════════════════════════
   ```

2. **Step markers**:
   ```
   ### Step 1: Extract Basepoints Knowledge
   ### Step 2: Detect Abstraction Layer
   ```

3. **Success indicators**:
   - ✅ = Success
   - ⚠️ = Warning (continues)
   - ❌ = Error (may stop)

4. **Completion summary**:
   ```
   ═══════════════════════════════════════════════════════
     COMMAND COMPLETE
   ═══════════════════════════════════════════════════════
   ```

---

### Check knowledge flow

Knowledge flows through commands via cache files:

```
shape-spec
  └─► specs/[name]/implementation/cache/
      ├─ basepoints-knowledge.md    ◄── Extracted patterns
      ├─ detected-layer.txt         ◄── UI/API/data layer
      └─ accumulated-knowledge.md   ◄── Combined knowledge
           │
           ▼
write-spec (reads accumulated-knowledge.md)
  └─► specs/[name]/spec.md
           │
           ▼
create-tasks (reads spec.md + accumulated-knowledge.md)
  └─► specs/[name]/tasks.md
```

If a command isn't using knowledge, check if the cache files exist and contain content.

---

## Getting Help

If you're still stuck:

1. **Check documentation**:
   - `COMMAND-FLOWS.md` - Detailed command reference
   - `WORKFLOW-MAP.md` - Visual workflow reference
   - `KNOWLEDGE-SYSTEMS.md` - Knowledge system explanation

2. **Search existing issues** on GitHub

3. **Open a new issue** with:
   - What you tried
   - Error messages
   - Your environment (OS, AI tool, project type)

---

*Last Updated: 2026-01-18*
