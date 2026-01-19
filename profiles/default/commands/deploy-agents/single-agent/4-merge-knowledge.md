# Phase 4: Merge Knowledge

Now that both basepoints and product knowledge are extracted, merge them into a unified knowledge document.

## Core Responsibilities

1. **Combine knowledge sources**: Merge basepoints and product knowledge
2. **Detect conflicts**: Identify contradictions between sources
3. **Resolve conflicts**: Present conflicts to user if needed
4. **Create merged knowledge**: Produce unified knowledge document

---

## Step 1: Load Extracted Knowledge

**ACTION REQUIRED:** Read the following files:

1. `geist/output/deploy-agents/knowledge/basepoints-knowledge.json`
2. `geist/output/deploy-agents/knowledge/product-knowledge.json`
3. `geist/config/project-profile.yml`

---

## Step 2: Identify Potential Conflicts

**ACTION REQUIRED:** Compare knowledge sources for conflicts:

### Check for conflicts in:
- **Tech stack**: Does product tech-stack match basepoints patterns?
- **Coding style**: Do basepoints patterns match product conventions?
- **Architecture**: Does product architecture match basepoints structure?

### Common conflict types:
- Different naming conventions mentioned
- Conflicting coding patterns
- Inconsistent architecture descriptions

---

## Step 3: Resolve Conflicts (if any)

**If conflicts are found:**

⚠️ CHECKPOINT - USER INTERACTION REQUIRED (only if conflicts exist)

Present conflicts to user:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️ KNOWLEDGE CONFLICTS DETECTED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Conflict 1: [Description]
- Basepoints says: [X]
- Product docs say: [Y]

Which should take precedence? (basepoints/product/custom)
```

**STOP and WAIT** for user response before proceeding.

**If no conflicts:** Continue to Step 4.

---

## Step 4: Create Merged Knowledge Document

**ACTION REQUIRED:** Create `geist/output/deploy-agents/knowledge/merged-knowledge.md` with:

```markdown
# Merged Project Knowledge

## Project Identity
**Name**: [from product]
**Type**: [from product/basepoints]
**Primary Language**: [from tech-stack/basepoints]
**Architecture**: [from basepoints]

## Core Patterns (from Basepoints)
[List key patterns that will guide specialization]

## Product Context (from Product Files)
### Mission
[Summary of mission]

### Key Features
[List key features]

### Target Users
[Who will use this]

## Specialization Guidelines

### For Standards
- [What patterns to add to standards]
- [What conventions to enforce]

### For Agents
- [What abstraction layers need specialists]
- [What expertise each specialist needs]

## Conflict Resolution
[Document any conflicts and how they were resolved]
```

---

## Display confirmation and next step

Once merging is complete, output:

```
✅ Knowledge merged successfully!

**Conflicts detected**: [count] (resolved: [count])

Merged knowledge saved to:
- geist/output/deploy-agents/knowledge/merged-knowledge.md

NEXT STEP 👉 Proceeding to phase 5: Specialize Standards
```

---

## User Standards & Preferences Compliance

IMPORTANT: Ensure knowledge merging aligns with:

@geist/standards/global/conventions.md
