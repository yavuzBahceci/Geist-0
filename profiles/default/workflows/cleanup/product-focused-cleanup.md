# Product-Focused Cleanup Workflow

After product files (mission, tech-stack, roadmap) are created, this workflow cleans irrelevant content from geist files and enhances remaining content with product-specific knowledge.

## Core Responsibilities

1. **Detect Product Scope**: Read product files to understand language, framework, project type
2. **Simplify Files**: Remove or flag irrelevant technology patterns not matching the product scope
3. **Expand Knowledge**: Add product-specific patterns and execute web research
4. **Generate Report**: Create a transparent report of all changes made

---

## Step 1: Detect Product Scope

**ACTION REQUIRED:** Read the following files and extract the detected scope:

1. Read `geist/product/tech-stack.md` and identify:
   - Primary language (e.g., TypeScript, Python, Rust, Go, Swift, etc.)
   - Frameworks used (e.g., React, Next.js, FastAPI, Express, etc.)
   - Database (if any)

2. Read `geist/product/mission.md` and identify:
   - Project type (webapp, api, cli, mobile, library, framework, ai-service)
   - Target users and use cases

3. Read `geist/product/roadmap.md` and identify:
   - Key features that indicate architecture patterns
   - Technical capabilities needed

**OUTPUT:** Create `geist/output/product-cleanup/detected-scope.yml` with:

```yaml
language: [detected primary language]
frameworks: [list of frameworks]
project_type: [webapp|api|cli|mobile|library|framework|ai-service]
architecture: [monolith|microservices|serverless]
has_frontend: [true|false]
has_backend: [true|false]
has_database: [true|false]
has_api: [true|false]
```

---

## Step 2: Phase A - Simplify (Remove Irrelevant Content)

**ACTION REQUIRED:** Based on the detected scope, identify and remove/simplify irrelevant content in `geist/` files.

### Removal Rules by Project Type

**If project is CLI or Library (no UI):**
- Remove or simplify: UI component examples, frontend patterns, screen/view references, responsive design patterns
- Keep: API patterns (if applicable), data processing, error handling

**If project is API-only (no frontend):**
- Remove or simplify: Frontend component examples, UI patterns, client-side state management
- Keep: REST/GraphQL patterns, middleware, authentication, database patterns

**If project is Frontend-only (no backend):**
- Remove or simplify: Server-side patterns, database examples, API implementation details
- Keep: Component patterns, state management, routing, styling

**If project has no database:**
- Remove or simplify: ORM examples, migration patterns, database connection examples
- Keep: In-memory data patterns, file-based storage (if applicable)

### How to Simplify

For each file in `geist/commands/`, `geist/workflows/`, `geist/standards/`:

1. **Read the file** and identify sections with technology-specific examples
2. **If the technology doesn't match the detected scope:**
   - Option A: Remove the irrelevant example entirely
   - Option B: Replace with a relevant example for the detected tech stack
   - Option C: Add a comment noting it's a generic example
3. **Track changes** for the report

**DO NOT remove entire files** - only simplify content within files.

---

## Step 3: Phase B - Expand (Add Product-Specific Knowledge)

**ACTION REQUIRED:** Enhance geist files with product-specific patterns.

### Enhancement Actions

1. **Add language-specific patterns** to `geist/standards/global/coding-style.md`:
   - If TypeScript: Add type safety patterns, interface conventions
   - If Python: Add type hints, dataclass patterns
   - If Rust: Add ownership patterns, Result/Option usage
   - If Go: Add interface patterns, error handling conventions

2. **Add framework-specific patterns** to relevant workflow files:
   - React: Component patterns, hooks usage, state management
   - Next.js: App Router patterns, Server Components
   - FastAPI: Dependency injection, Pydantic models
   - Express: Middleware patterns, routing conventions

3. **Add project-type patterns** to `geist/standards/`:
   - CLI: Argument parsing, progress indicators, exit codes
   - API: REST conventions, versioning, rate limiting
   - Webapp: Component architecture, routing, responsive design

---

## Step 4: Execute Web Research (Optional but Recommended)

**ACTION REQUIRED:** Search for best practices to validate and enhance patterns.

### Search Queries to Execute

Based on detected scope, search for:

1. `[language] best practices 2026`
2. `[framework] common patterns and anti-patterns`
3. `[project_type] architecture best practices`
4. `[language] [framework] production tips`

### Validation Requirements

- Information must appear in **2+ independent sources**
- Prioritize: Official docs > Reputable tech blogs > Stack Overflow (high-voted)
- Include version-specific information when available

### Save Research Results

Create `geist/product/docs/research-notes.md` with validated findings organized by category:
- Best practices
- Common patterns
- Anti-patterns to avoid
- Performance tips
- Security considerations

---

## Step 5: Generate Cleanup Report

**ACTION REQUIRED:** Create `geist/output/product-cleanup/cleanup-report.md` with:

```markdown
# Product-Focused Cleanup Report

## Detected Scope
- Language: [language]
- Frameworks: [frameworks]
- Project Type: [type]
- Architecture: [architecture]

## Phase A: Simplification

### Files Modified
- [file path]: [what was changed]
- [file path]: [what was changed]

### Content Removed/Simplified
- [description of irrelevant content removed]

## Phase B: Enhancement

### Patterns Added
- [file path]: Added [pattern type] for [technology]

### Web Research Completed
- [query]: [key findings]

## Summary
- Files reviewed: [count]
- Files modified: [count]
- Patterns added: [count]
- Research queries executed: [count]
```

---

## Output Confirmation

After completing all steps, display:

```
✅ Product-Focused Cleanup Complete!

📊 Detected Scope:
   Language: [language]
   Frameworks: [frameworks]
   Project Type: [type]

🧹 Phase A (Simplify):
   Files modified: [count]
   Irrelevant content removed: [count] sections

📚 Phase B (Expand):
   Patterns added: [count]
   Research completed: [yes/no]

📁 Reports Generated:
   - geist/output/product-cleanup/detected-scope.yml
   - geist/output/product-cleanup/cleanup-report.md
   - geist/product/docs/research-notes.md (if research done)
```

---

## Important Notes

- **Do NOT delete entire workflow/command files** - they may be needed for future features
- **Do simplify examples** within files to match the detected tech stack
- **Do add product-specific patterns** that will help AI understand this specific project
- **Track all changes** transparently in the cleanup report
