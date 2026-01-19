This begins a multi-step process for adapting an existing codebase into product documentation.

## Step 0: Automatic Project Detection & Research

Before gathering product information, automatically detect project characteristics and enrich with web research.

### 0.1: Run Project Detection

Automatically analyze the codebase to detect tech stack, commands, architecture, and security level.

**Read and follow the workflow instructions in:** `@geist/workflows/detection/detect-project-profile.md`

This will:
- Detect language, framework, and database from config files
- Extract build/test/lint commands from package.json, Makefile, etc.
- Analyze directory structure for project type and architecture
- Check for security indicators (auth, secrets management)
- Calculate overall detection confidence

### 0.2: Enrich with Web Research (Optional)

Research best practices and patterns for the detected tech stack. Set `RESEARCH_DEPTH` to control depth (minimal, standard, comprehensive).

**Read and follow the workflow instructions in:** `@geist/workflows/research/research-orchestrator.md`

This will:
- Research best practices for detected libraries
- Gather architecture patterns for your tech stack
- Check for security vulnerabilities and updates
- Store findings in `geist/config/enriched-knowledge/`

### 0.3: Present Findings and Get User Confirmation

**⚠️ CHECKPOINT - USER INTERACTION REQUIRED**

Display the detected configuration to the user and ask for confirmation before proceeding.

**Read and follow the workflow instructions in:** `@geist/workflows/detection/present-and-confirm.md`

**IMPORTANT:** After presenting the detected configuration, you MUST:
1. Display all detected values clearly formatted
2. **STOP and WAIT** for the user to confirm or request changes
3. Do NOT proceed until the user explicitly confirms or provides corrections

### 0.4: Ask Minimal Questions

**⚠️ CHECKPOINT - USER INTERACTION REQUIRED**

Ask the user questions that cannot be detected from the codebase.

**Read and follow the workflow instructions in:** `@geist/workflows/detection/question-templates.md`

**IMPORTANT:** You MUST ask these questions and **WAIT for user responses**:

1. **Compliance requirements** - GDPR, HIPAA, SOC 2, etc. (can't detect from code)
2. **Human review preference** - How much AI oversight do you want?

Do NOT assume defaults without asking. Present the questions and wait for the user to respond.

The profile is saved to `geist/config/project-profile.yml` for use by subsequent commands.

---

## Step 1: Gather Product Information

**⚠️ CHECKPOINT - USER INTERACTION REQUIRED**

Set up documentation folders and gather comprehensive product information from multiple sources.

This step creates:
- `geist/product/docs/` - For external resources, research, and reference materials
- `geist/product/inheritance/` - For existing documentation to build upon

**Read and follow the workflow instructions in:** `@geist/workflows/planning/gather-product-info-from-codebase.md`

**IMPORTANT:** This workflow requires user interaction to gather:
- Product name and core concept (if not clear from codebase)
- Target users and key features
- Public resources: website links, documentation URLs, competitor references
- Files to place in `geist/product/docs/` (PRDs, specs, research, diagrams)
- Files to place in `geist/product/inheritance/` (existing docs, brand guidelines)
- Web research suggestions: keywords, competitors, topics to research

You MUST ask the user for this information and **WAIT for their responses** before proceeding.

{{UNLESS compiled_single_command}}
## Display confirmation and next step

Once you've completed setup and information gathering (including receiving all user responses), output the following message:

```
I have gathered product information from all available sources.

📊 Project Profile: Saved to geist/config/project-profile.yml
📚 Enriched Knowledge: Saved to geist/config/enriched-knowledge/

NEXT STEP 👉 Run the command, `2-analyze-codebase.md`
```
{{ENDUNLESS compiled_single_command}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

When gathering product information, use the user's standards and preferences for context and baseline assumptions, as documented in these files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
