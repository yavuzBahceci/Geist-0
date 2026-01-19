# Phase 3: Extract Product Knowledge

Now that basepoints knowledge is extracted, extract knowledge from product files.

## Core Responsibilities

1. **Read product files**: Read mission.md, roadmap.md, tech-stack.md
2. **Extract mission**: Product purpose, users, differentiators, features
3. **Extract roadmap**: Completed phases, planned features, priorities
4. **Extract tech stack**: Languages, frameworks, tools, dependencies
5. **Store knowledge**: Save extracted knowledge for merging

---

## Step 1: Read Mission File

**ACTION REQUIRED:** Read `geist/product/mission.md` and extract:

- **Pitch**: One-sentence product description
- **Users**: Primary customers and personas
- **Problem**: What problem does this solve?
- **Solution**: How does it solve the problem?
- **Differentiators**: What makes this unique?
- **Key Features**: Core, workflow, extensibility features

---

## Step 2: Read Roadmap File

**ACTION REQUIRED:** Read `geist/product/roadmap.md` and extract:

- **Completed Phases**: What's already built?
- **Current Phase**: What's being worked on?
- **Planned Features**: What's coming next?
- **Priorities**: What's most important?

---

## Step 3: Read Tech Stack File

**ACTION REQUIRED:** Read `geist/product/tech-stack.md` and extract:

- **Core Technologies**: Primary languages and tools
- **Project Structure**: How files are organized
- **Supported Targets**: What platforms/languages are supported
- **Dependencies**: What external tools are required
- **AI Integration**: How does it integrate with AI assistants?

---

## Step 4: Store Product Knowledge

**ACTION REQUIRED:** Create `geist/output/deploy-agents/knowledge/product-knowledge.json` with:

```json
{
  "mission": {
    "pitch": "...",
    "users": {
      "primary": ["..."],
      "personas": ["..."]
    },
    "problem": "...",
    "solution": "...",
    "differentiators": ["..."],
    "key_features": {
      "core": ["..."],
      "workflow": ["..."],
      "extensibility": ["..."]
    }
  },
  "roadmap": {
    "completed_phases": ["..."],
    "current_phase": "...",
    "planned_features": ["..."]
  },
  "tech_stack": {
    "core": {
      "language": "...",
      "framework": "...",
      "tools": ["..."]
    },
    "structure": {
      "commands": "...",
      "workflows": "...",
      "standards": "..."
    },
    "dependencies": ["..."]
  }
}
```

---

## Step 5: Create Product Knowledge Summary

**ACTION REQUIRED:** Create `geist/output/deploy-agents/knowledge/product-knowledge-summary.md` with:

```markdown
# Extracted Product Knowledge

## Mission
**Pitch**: [one sentence]
**Problem Solved**: [description]
**Solution**: [description]

## Target Users
1. [User type 1]
2. [User type 2]

## Key Differentiators
1. [Differentiator 1]
2. [Differentiator 2]

## Key Features
### Core
- [Feature]

### Workflow
- [Feature]

## Roadmap Status
### Completed
- [Phase/feature]

### Planned
- [Phase/feature]

## Tech Stack
- **Language**: [language]
- **Framework**: [framework]
- **Dependencies**: [list]
```

---

## Display confirmation and next step

Once extraction is complete, output:

```
✅ Product knowledge extraction complete!

- Mission extracted: pitch, users, problem, solution, differentiators
- Roadmap extracted: [count] completed phases, [count] planned features
- Tech stack extracted: [language], [framework], [count] dependencies

Knowledge saved to:
- geist/output/deploy-agents/knowledge/product-knowledge.json
- geist/output/deploy-agents/knowledge/product-knowledge-summary.md

NEXT STEP 👉 Proceeding to phase 4: Merge Knowledge and Resolve Conflicts
```

---

## User Standards & Preferences Compliance

IMPORTANT: Ensure knowledge extraction aligns with:

@geist/standards/global/conventions.md
