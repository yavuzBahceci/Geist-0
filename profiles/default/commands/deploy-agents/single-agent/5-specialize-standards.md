# Phase 5: Specialize Standards

Now that knowledge has been merged, specialize the standards files based on the project's actual patterns.

## Core Responsibilities

1. **Read extracted knowledge**: Load patterns from basepoints and product knowledge
2. **Update existing standards**: Modify standards to reflect project-specific patterns
3. **Create new standards**: Add standards for project-specific patterns not covered
4. **Remove irrelevant content**: Simplify or remove content that doesn't apply

---

## Step 1: Load Extracted Knowledge

**ACTION REQUIRED:** Read the following files:

1. `geist/output/deploy-agents/knowledge/basepoints-knowledge.json`
2. `geist/output/deploy-agents/knowledge/basepoints-knowledge-summary.md`
3. `geist/output/deploy-agents/knowledge/merged-knowledge.md`
4. `geist/config/project-profile.yml`

Extract from these files:
- **Language/Tech Stack**: What languages and tools does this project use?
- **Patterns**: What coding patterns are used?
- **Standards**: What naming conventions, coding styles are followed?
- **Flows**: What are the key workflows?
- **Project Type**: Is it a webapp, api, cli, mobile, library, framework?

---

## Step 2: Identify Standards to Update

**ACTION REQUIRED:** For each standard file in `geist/standards/`, determine what needs updating:

| Standard File | What to Update |
|---------------|----------------|
| `global/coding-style.md` | Add project-specific language patterns |
| `global/conventions.md` | Add project-specific conventions |
| `global/error-handling.md` | Specialize for project's error patterns |
| `global/validation.md` | Add project-specific validation patterns |
| `global/commenting.md` | Specialize for project's languages |
| `testing/test-writing.md` | Specialize for project's test approach |
| `process/development-workflow.md` | Reflect project's actual workflow |
| `documentation/standards.md` | Match project's documentation style |

---

## Step 3: Update Each Standard

**ACTION REQUIRED:** For each standard file that needs updating:

### 3.1 Read the current standard file

### 3.2 Identify what to change:
- **Replace generic examples** with project-specific examples from basepoints
- **Add project-specific patterns** that were extracted
- **Remove irrelevant content** that doesn't apply to this project's tech stack
- **Update language references** to match the project's actual languages

### 3.3 Write the updated standard

**Example transformation for a Python project:**

Before (generic):
```markdown
## Coding style best practices
- Use consistent naming conventions
- Use automated formatting
```

After (specialized):
```markdown
## Python Coding Style Standards
- Use snake_case for functions and variables
- Use PascalCase for classes
- Run `black` for formatting
- Run `flake8` for linting
```

---

## Step 4: Create New Standards (if needed)

**ACTION REQUIRED:** If the basepoints knowledge contains patterns not covered by existing standards:

1. Identify the pattern category (e.g., "Template Syntax", "API Design")
2. Create a new standard file: `geist/standards/project/[pattern-name].md`
3. Document the pattern with:
   - Overview
   - Guidelines
   - Examples from the codebase

---

## Step 5: Generate Specialization Report

**ACTION REQUIRED:** Create `geist/output/deploy-agents/reports/standards-specialization.md`:

```markdown
# Standards Specialization Report

## Project Profile
- Language: [detected language]
- Framework: [detected framework]
- Project Type: [type]

## Standards Updated
- [list of updated standards with summary of changes]

## New Standards Created
- [list of new standards if any]

## Content Removed
- [list of irrelevant content removed]
```

---

## Display confirmation and next step

Once standards are specialized, output:

```
✅ Standards specialized for project!

**Project Type:** [type]
**Language:** [language]
**Standards Updated:** [count]
**New Standards Created:** [count]

Report: geist/output/deploy-agents/reports/standards-specialization.md

NEXT STEP 👉 Proceeding to phase 6: Specialize Agents
```

---

## User Standards & Preferences Compliance

IMPORTANT: When specializing standards, ensure they align with:

@geist/standards/global/coding-style.md
@geist/standards/global/conventions.md
