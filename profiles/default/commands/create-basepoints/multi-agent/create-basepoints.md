# Codebase Basepoints Generation Process

You are helping me analyze an existing codebase and generate comprehensive basepoint documentation. This process will analyze the codebase, detect abstraction layers, and create basepoint files for each module and parent folder, extracting patterns, standards, flows, strategies, and testing approaches at every abstraction layer.

This process will follow 7 main phases, each with their own workflow steps:

**Process Overview:**
- PHASE 1. Validate prerequisites (product files existence)
- PHASE 2. Detect abstraction layers from codebase structure
- PHASE 3. Mirror project structure in basepoints folder
- PHASE 4. Analyze codebase comprehensively for patterns
- PHASE 5. Generate module basepoint files
- PHASE 6. Generate parent folder basepoint files
- PHASE 7. Generate headquarter.md from product files

Follow each of these phases and their individual workflows IN SEQUENCE:

## Multi-Phase Process:

### PHASE 1: Validate Prerequisites

Use the **codebase-analyzer** subagent to validate prerequisites:
- Check for product files existence (`geist/product/mission.md`, `roadmap.md`, `tech-stack.md`)
- Prompt user to create product files if missing (direct to `plan-product` or `adapt-to-product`)
- Validate basepoints folder doesn't already exist or handle existing folder scenario
- Set up initial structure for basepoints folder

### PHASE 2: Detect Abstraction Layers

Use the **abstraction-detector** subagent to detect abstraction layers:
- Analyze folder structure to detect abstraction layers
- Detect common patterns (mobile: `data/`, `domain/`, `presentation/`; web: `backend/`, `frontend/`)
- Analyze file naming patterns and code organization
- Prompt user to confirm or clarify detected layers
- Allow user to override or refine layer boundaries
- Document detected abstraction layers

### PHASE 3: Mirror Project Structure

Use the **codebase-analyzer** subagent to mirror project structure:
- Create `geist/basepoints/` folder
- Mirror project directory structure within basepoints folder
- Exclude generated/irrelevant folders (`node_modules/`, `.git/`, `build/`, `dist/`, `.next/`, `vendor/`, `geist/`, `basepoints/`)
- Identify folders containing actual modules (exclude configuration/build folders)
- Include software project's current path layer in structure

### PHASE 4: Analyze Codebase

Use the **codebase-analyzer** subagent to analyze codebase comprehensively:
- Analyze all code files in identified module folders
- Analyze configuration files, test files, and documentation
- Extract patterns, standards, flows, strategies, and testing approaches
- Process files one by one for comprehensive extraction
- Extract patterns at every abstraction layer and across abstraction layers

### PHASE 5: Generate Module Basepoints

Use the **basepoint-generator** subagent to generate module basepoint files:
- Generate `agent-base-[module-name].md` files for each folder containing modules
- Place files in mirrored structure (e.g., `geist/basepoints/src/data/models/agent-base-models.md`)
- Name files based on actual module names found in folder structure
- Document patterns, standards, flows, strategies, and testing for each module
- Use extracted patterns from codebase analysis

### PHASE 6: Generate Parent Basepoints

Use the **basepoint-generator** subagent to generate parent folder basepoint files:
- Generate basepoint files for parent folders containing modules
- Continue creating basepoints up to top layer of software project
- Use child basepoint files as source for creating parent basepoint files
- Aggregate patterns and knowledge from child modules into parent basepoint documentation
- Maintain hierarchical structure with parent basepoints referencing children

### PHASE 7: Generate Headquarter

Use the **headquarter-writer** subagent to generate headquarter.md:
- Create `geist/basepoints/headquarter.md` at root of basepoints folder
- Generate content from product files (`mission.md`, `roadmap.md`, `tech-stack.md`)
- Bridge product-level abstraction with software project-level abstraction
- Document overall architecture, abstraction layers, and module relationships
- Include detected abstraction layers and how modules relate to each other

### PHASE 8: Inform the user

After all steps complete, inform the user:

```
Basepoints generation complete!

✅ Basepoints folder structure created: `geist/basepoints/`
✅ Headquarter file generated: `geist/basepoints/headquarter.md`
✅ Module basepoint files generated for all modules
✅ Parent folder basepoint files generated

All basepoint documentation is ready for use.
```
