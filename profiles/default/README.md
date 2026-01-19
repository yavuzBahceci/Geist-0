# profiles/default - Geist Template for Any Project

This is the default template that gets installed into your project. It provides abstract, project-agnostic commands and workflows that become specific to your project after specialization.

---

## What This Is

**In simple terms**: A cognitive architecture that gives your AI assistant persistent memory, structured workflows, and deep codebase understanding. When you run commands like `/shape-spec` or `/implement-tasks`, the AI gets your project's patterns, architecture, and conventions as context—automatically.

**What it does**:
- Documents your codebase patterns into "basepoints" (living documentation)
- Transforms abstract commands → project-specific commands
- Chains commands together so each step provides context for the next
- Validates with your actual build/test/lint commands
- Accumulates knowledge across commands—nothing is lost

**What it doesn't do**:
- It's not a runtime system or framework
- It doesn't execute code—it generates prompts and files
- It doesn't replace your build tools—it uses them for validation
- It's not magic—you still need to review and guide the AI

---

## Why Geist Over Standard AI Tools?

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   STANDARD AI TOOLS                        GEIST                            │
│   ──────────────────                       ─────                            │
│                                                                             │
│   ┌─────────────┐                          ┌─────────────┐                  │
│   │   Rules     │  Static text files       │  Basepoints │  Living docs    │
│   │   File      │  You write manually      │             │  Auto-generated │
│   └─────────────┘                          └─────────────┘                  │
│         ↓                                        ↓                          │
│   "Follow these rules..."                  "Here's how this codebase        │
│   (generic, disconnected)                   actually works..."              │
│                                            (specific, interconnected)       │
│                                                                             │
│   ┌─────────────┐                          ┌─────────────┐                  │
│   │  Context    │  Copy-paste files        │  Knowledge  │  Accumulated    │
│   │  Window     │  Limited tokens          │  System     │  across commands│
│   └─────────────┘                          └─────────────┘                  │
│         ↓                                        ↓                          │
│   Context lost between                     Context flows between            │
│   conversations                            commands automatically           │
│                                                                             │
│   ┌─────────────┐                          ┌─────────────┐                  │
│   │  Prompts    │  Ad-hoc, inconsistent    │  Workflows  │  Structured,    │
│   │             │  "Add auth somehow"      │             │  spec-driven    │
│   └─────────────┘                          └─────────────┘                  │
│         ↓                                        ↓                          │
│   Results vary wildly                      Consistent, validated            │
│   based on prompt quality                  results every time               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

| Standard Approach | Problem | Geist Solution |
|-------------------|---------|----------------|
| **Rules files** | Static, generic, manually written | Basepoints: auto-generated, living, specific to your code |
| **Context window** | Limited tokens, lost between sessions | Knowledge system: accumulated, persistent, interconnected |
| **Ad-hoc prompts** | Inconsistent results, no structure | Workflows: structured, validated, repeatable |
| **Memory features** | Shallow, conversation-scoped | Deep, project-scoped, flows between commands |

---

## The Core Idea

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Abstract Templates  →  Specialize  →  Project-Specific   │
│  (this folder)             (deploy)      Commands          │
│                                                             │
│  {{PLACEHOLDERS}}    →   Replace     →   npm run build    │
│  Generic patterns    →   Inject      →   Your patterns    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Key insight**: Commands chain together by passing knowledge forward. Each command reads context from previous commands and adds to it.

---

## How Commands Chain Together: Complete Workflow in Order

### Setup Chain (Run Once - Sequential Order)

```
┌─────────────────────────────────────────────────────────────────┐
│              SETUP COMMAND CHAIN (Run in Order)                 │
└─────────────────────────────────────────────────────────────────┘

  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
  ┃  Step 1: /adapt-to-product                                 ┃
  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
  
  Inputs:
  ┌──────────────────────┐
  │ • package.json       │
  │ • Cargo.toml         │
  │ • go.mod             │
  │ • Your codebase      │
  └──────────┬───────────┘
             │
             │ Process
             ▼
  ┌───────────────────────────────────────┐
  │ 1. Detect tech stack                  │
  │ 2. Research best practices/CVEs       │
  │ 3. Ask 2-3 questions (compliance)     │
  └──────────┬────────────────────────────┘
             │
             │ Creates Files (Required by Step 2)
             ▼
  ┌───────────────────────────────────────┐
  │ 📄 product/mission.md                 │
  │ 📄 product/roadmap.md                 │
  │ 📄 product/tech-stack.md              │
  │ 📄 config/project-profile.yml         │
  │ 📁 config/enriched-knowledge/         │
  └──────────┬────────────────────────────┘
             │
             │ ═══════════════════════════
             │ DEPENDENCY: Step 2 requires these files
             ▼
  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
  ┃  Step 2: /create-basepoints                                ┃
  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
  
  Inputs:
  ┌───────────────────────────────────────┐
  │ ✅ product/*.md (from Step 1)         │
  │ ✅ config/project-profile.yml         │
  │    (from Step 1)                      │
  │ • Your codebase structure             │
  └──────────┬────────────────────────────┘
             │
             │ Process
             ▼
  ┌───────────────────────────────────────┐
  │ 1. Read product files (from Step 1)   │
  │ 2. Analyze codebase structure         │
  │ 3. Document patterns per module       │
  │ 4. Generate library basepoints        │
  └──────────┬────────────────────────────┘
             │
             │ Creates Files (Required by Step 3)
             ▼
  ┌───────────────────────────────────────┐
  │ 📄 basepoints/headquarter.md          │
  │ 📁 basepoints/[layers]/[modules]/     │
  │    📄 agent-base-*.md                 │
  │ 📁 basepoints/libraries/              │
  │    📄 [library]-basepoint.md          │
  └──────────┬────────────────────────────┘
             │
             │ ═══════════════════════════
             │ DEPENDENCY: Step 3 requires these files
             ▼
  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
  ┃  Step 3: /deploy-agents                                    ┃
  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
  
  Inputs:
  ┌───────────────────────────────────────┐
  │ ✅ product/*.md (from Step 1)         │
  │ ✅ basepoints/**/*.md (from Step 2)   │
  │ ✅ config/*.yml (from Step 1)         │
  └──────────┬────────────────────────────┘
             │
             │ Process
             ▼
  ┌───────────────────────────────────────┐
  │ 1. Read all knowledge (Steps 1 & 2)   │
  │ 2. Transform templates                │
  │ 3. Replace {{PLACEHOLDERS}}           │
  │ 4. Configure validation commands      │
  └──────────┬────────────────────────────┘
             │
             │ Creates Files
             ▼
  ┌───────────────────────────────────────┐
  │ 📁 commands/ (specialized)            │
  │ 📁 workflows/ (specialized)           │
  │ ✅ Ready to use                       │
  └───────────────────────────────────────┘
             │
             │ Optional
             ▼
  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
  ┃  Step 4: /cleanup-geist (Optional)                      ┃
  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
  
  • Cleans remaining placeholders
  • Verifies knowledge completeness
  • Creates verification report
```

### Development Chain (Per Feature - Sequential Order)

```
┌─────────────────────────────────────────────────────────────────┐
│         DEVELOPMENT COMMAND CHAIN (Must Run in Order)           │
└─────────────────────────────────────────────────────────────────┘

  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
  ┃  Step 1: /shape-spec "Feature description"                 ┃
  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
  
  Inputs:
  ┌───────────────────────────────────────┐
  │ • Your feature description            │
  │ • basepoints/**/*.md (from setup)     │
  └──────────┬────────────────────────────┘
             │
             │ Process
             ▼
  ┌───────────────────────────────────────┐
  │ 1. Extract relevant basepoints        │
  │ 2. Detect abstraction layer           │
  │ 3. Ask clarifying questions           │
  │ 4. Accumulate knowledge               │
  └──────────┬────────────────────────────┘
             │
             │ Creates Files (Required by Step 2, 3, 4)
             ▼
  ┌───────────────────────────────────────┐
  │ 📄 specs/[name]/planning/             │
  │    requirements.md                    │
  │ 📄 specs/[name]/implementation/       │
  │    cache/                             │
  │    ├─ basepoints-knowledge.md         │
  │    ├─ library-basepoints-knowledge.md │
  │    ├─ accumulated-knowledge.md        │
  │    └─ detected-layer.txt              │
  └──────────┬────────────────────────────┘
             │
             │ ═══════════════════════════
             │ DEPENDENCY: Step 2 requires requirements.md
             │ Also: knowledge files used by steps 2,3,4
             ▼
  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
  ┃  Step 2: /write-spec                                       ┃
  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
  
  Inputs:
  ┌───────────────────────────────────────┐
  │ ✅ requirements.md (from Step 1)      │
  │ ✅ accumulated-knowledge.md (Step 1)  │
  │ ✅ detected-layer.txt (Step 1)        │
  │ • basepoints/**/*.md (still available)│
  └──────────┬────────────────────────────┘
             │
             │ Process
             ▼
  ┌───────────────────────────────────────┐
  │ 1. Load accumulated knowledge         │
  │ 2. Read requirements + cached knowledge│
  │ 3. Reference your patterns            │
  │ 4. Write specification                │
  │ 5. Accumulate more knowledge          │
  └──────────┬────────────────────────────┘
             │
             │ Creates Files (Required by Step 3)
             ▼
  ┌───────────────────────────────────────┐
  │ 📄 specs/[name]/spec.md               │
  │ 📄 accumulated-knowledge.md (updated) │
  └──────────┬────────────────────────────┘
             │
             │ ═══════════════════════════
             │ DEPENDENCY: Step 3 requires spec.md
             ▼
  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
  ┃  Step 3: /create-tasks                                    ┃
  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
  
  Inputs:
  ┌───────────────────────────────────────┐
  │ ✅ spec.md (from Step 2)              │
  │ ✅ accumulated-knowledge.md (cached)  │
  └──────────┬────────────────────────────┘
             │
             │ Process
             ▼
  ┌───────────────────────────────────────┐
  │ 1. Load accumulated knowledge         │
  │ 2. Read specification                 │
  │ 3. Break into tasks                   │
  │ 4. Add acceptance criteria            │
  │ 5. Accumulate more knowledge          │
  └──────────┬────────────────────────────┘
             │
             │ Creates Files (Required by Step 4)
             ▼
  ┌───────────────────────────────────────┐
  │ 📄 specs/[name]/tasks.md              │
  │ 📄 accumulated-knowledge.md (updated) │
  └──────────┬────────────────────────────┘
             │
             │ ═══════════════════════════
             │ DEPENDENCY: Step 4 requires tasks.md
             ▼
  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
  ┃  Step 4: /implement-tasks OR /orchestrate-tasks            ┃
  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
  
  Inputs:
  ┌───────────────────────────────────────┐
  │ ✅ tasks.md (from Step 3)             │
  │ ✅ accumulated-knowledge.md (cached)  │
  │ ✅ detected-layer.txt (cached)        │
  │ • basepoints/**/*.md (for patterns)   │
  └──────────┬────────────────────────────┘
             │
             │ Process
             ▼
  ┌───────────────────────────────────────┐
  │ 1. Load full accumulated context      │
  │ 2. Read tasks + cached knowledge      │
  │ 3. Use your patterns                  │
  │ 4. Implement code                     │
  │ 5. Validate with YOUR commands        │
  └──────────┬────────────────────────────┘
             │
             │ Creates Files
             ▼
  ┌───────────────────────────────────────┐
  │ 📝 Code changes (you review)          │
  │ 📄 validation-report.md               │
  └───────────────────────────────────────┘

  ⚠️  ORDER ENFORCEMENT: Each step creates files the next step requires.
     Cannot skip steps—commands will fail if required files are missing.
```

---

## Knowledge System

### Basepoints: Your Codebase's Memory

Basepoints are living documentation of your codebase—auto-generated, not manually written:

```
basepoints/
├── headquarter.md           # Project overview
├── ui/
│   └── components/
│       └── agent-base-components.md  # Component patterns
├── api/
│   └── routes/
│       └── agent-base-routes.md      # API patterns
└── libraries/
    ├── react/
    │   └── react-basepoint.md        # React usage patterns
    └── prisma/
        └── prisma-basepoint.md       # Database patterns
```

Each basepoint contains:
- **Patterns**: How code is organized in this module
- **Standards**: Conventions followed
- **Flows**: How data/control moves
- **Strategies**: Decision patterns

### Knowledge Accumulation

Context flows between commands—nothing is lost:

```
shape-spec
  └─► Extracts relevant basepoints
       └─► accumulated-knowledge.md
            │
write-spec  │
  └─► Loads previous + adds own knowledge
       └─► accumulated-knowledge.md (updated)
            │
create-tasks │
  └─► Loads previous + adds own knowledge
       └─► accumulated-knowledge.md (updated)
            │
implement-tasks
  └─► Uses full accumulated context
```

### Context Enrichment Strategy

All spec/implementation commands use "narrow focus + expand knowledge":

```
┌─────────────────────────────────────────────────────────────┐
│                  CONTEXT ENRICHMENT                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. NARROW FOCUS                                            │
│     └─► Detect relevant modules for this feature            │
│     └─► Extract only applicable basepoints                  │
│                                                             │
│  2. EXPAND KNOWLEDGE                                        │
│     └─► Add library capabilities/constraints                │
│     └─► Add product context                                 │
│     └─► Add accumulated knowledge from previous commands    │
│                                                             │
│  3. ENRICHED CONTEXT                                        │
│     └─► Precise, comprehensive context for AI               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## What Each Command Actually Does

### Setup Commands (Run Once)

#### `/adapt-to-product`
**Input**: Your codebase (reads package.json, Cargo.toml, etc.)  
**Process**:
1. Detects tech stack from config files
2. Researches web for best practices/CVEs
3. Asks you 2-3 questions (compliance, review preference)
4. Creates product documentation

**Output Files**:
- `geist/product/mission.md`
- `geist/product/roadmap.md`
- `geist/product/tech-stack.md`
- `geist/config/project-profile.yml`
- `geist/config/enriched-knowledge/`

---

#### `/create-basepoints`
**Input**: Product files (from step 1) + your codebase  
**Process**:
1. Analyzes your directory structure
2. Detects abstraction layers (UI, API, data, etc.)
3. Documents patterns per module
4. Creates library basepoints for your tech stack
5. Creates comprehensive basepoints files

**Output Files**:
- `geist/basepoints/headquarter.md` (project overview)
- `geist/basepoints/[layer]/[module]/agent-base-*.md` (one per module)
- `geist/basepoints/libraries/[category]/[library]-basepoint.md`

---

#### `/deploy-agents`
**Input**: Everything from steps 1 & 2  
**Process**:
1. Reads all knowledge (product, basepoints, profile)
2. Transforms abstract templates → project-specific
3. Replaces `{{PROJECT_BUILD_COMMAND}}` → `npm run build`
4. Injects your patterns into commands
5. Configures validation commands

**Output**: Specialized `geist/commands/` and `geist/workflows/`

---

### Development Commands (Run Per Feature)

#### `/shape-spec "Feature description"`
**Input**: Feature description (your text)  
**Process**:
1. Extracts relevant basepoints for this feature
2. Extracts library basepoints knowledge
3. Detects abstraction layer (UI/API/data)
4. Asks clarifying questions informed by your patterns
5. Caches extracted knowledge for next commands
6. Accumulates knowledge

**Output Files**:
- `geist/specs/[name]/planning/requirements.md`
- `geist/specs/[name]/implementation/cache/basepoints-knowledge.md`
- `geist/specs/[name]/implementation/cache/library-basepoints-knowledge.md`
- `geist/specs/[name]/implementation/cache/accumulated-knowledge.md`

---

#### `/write-spec`
**Input**: `requirements.md` (from shape-spec) + accumulated knowledge  
**Process**:
1. Loads accumulated knowledge from shape-spec
2. Reads requirements and basepoints knowledge
3. References your standards and patterns
4. Suggests reusable code from basepoints
5. Writes detailed specification
6. Accumulates more knowledge

**Output Files**:
- `geist/specs/[name]/spec.md`
- `accumulated-knowledge.md` (updated)

---

#### `/create-tasks`
**Input**: `spec.md` (from write-spec) + accumulated knowledge  
**Process**:
1. Loads accumulated knowledge
2. Reads specification
3. Breaks it into actionable tasks
4. Groups related tasks
5. Adds acceptance criteria
6. Accumulates more knowledge

**Output Files**:
- `geist/specs/[name]/tasks.md`
- `accumulated-knowledge.md` (updated)

---

#### `/implement-tasks` OR `/orchestrate-tasks`
**Input**: `tasks.md` (from create-tasks) + full accumulated context  
**Process**:
1. Loads full accumulated knowledge
2. Reads tasks and basepoints knowledge
3. Uses your coding patterns and standards
4. Implements code changes
5. Validates with your build/test/lint commands
6. Reports results

**Output**: Code changes (files you review) + validation report

---

#### `/fix-bug`
**Input**: Bug description or error message  
**Process**:
1. Analyzes issue (parses error, extracts details)
2. Researches affected libraries
3. Integrates basepoints knowledge
4. Analyzes code (traces execution, finds root cause)
5. Synthesizes all knowledge
6. Implements fix iteratively with validation

**Output**: Fix + `fix-report.md` or `guidance-request.md` if stuck

---

## File Structure

```
profiles/default/
├── commands/                    # Abstract commands (templates)
│   ├── adapt-to-product/        # Setup: Extract product info
│   ├── plan-product/            # Setup: Plan new product
│   ├── create-basepoints/       # Setup: Create codebase docs
│   ├── deploy-agents/           # Setup: Specialize commands
│   ├── cleanup-geist/        # Maintenance: Verify deployment
│   ├── update-basepoints-and-redeploy/  # Maintenance: Sync changes
│   ├── shape-spec/              # Development: Research requirements
│   ├── write-spec/              # Development: Write specification
│   ├── create-tasks/            # Development: Break into tasks
│   ├── implement-tasks/         # Development: Single-agent impl
│   ├── orchestrate-tasks/       # Development: Multi-agent impl
│   └── fix-bug/                 # Utility: Analyze & fix bugs
│
├── workflows/                   # Reusable workflow templates
│   ├── basepoints/              # Knowledge extraction
│   ├── codebase-analysis/       # Codebase analysis & basepoints
│   ├── common/                  # Shared workflows
│   ├── detection/               # Auto-detection
│   ├── research/                # Web research
│   ├── validation/              # Validation utilities
│   ├── specification/           # Spec writing
│   ├── implementation/          # Task implementation
│   ├── learning/                # Session learning
│   └── cleanup/                 # Cleanup workflows
│
├── standards/                   # Global standards (abstract)
│   ├── global/                  # Cross-cutting standards
│   ├── documentation/           # Doc standards
│   ├── process/                 # Process standards
│   ├── quality/                 # Quality standards
│   └── testing/                 # Testing standards
│
├── agents/                      # Agent definitions
│
└── docs/                        # Documentation
    ├── COMMAND-FLOWS.md         # Detailed command flows
    ├── KNOWLEDGE-SYSTEMS.md     # Knowledge integration
    ├── INSTALLATION-GUIDE.md    # Installation guide
    ├── PATH-REFERENCE-GUIDE.md  # Path conventions
    ├── TROUBLESHOOTING.md       # Common issues
    ├── REFACTORING-GUIDELINES.md
    ├── TECHNOLOGY-AGNOSTIC-BEST-PRACTICES.md
    └── command-references/      # Per-command visual guides
```

**After specialization** (in your project's `geist/` folder):

```
geist/
├── commands/              # Specialized commands (project-specific)
├── workflows/             # Specialized workflows
├── basepoints/            # Your codebase documentation
│   ├── headquarter.md
│   ├── [layers]/[modules]/
│   └── libraries/         # Tech stack documentation
├── product/               # Your product files
├── config/                # Project profile + enriched knowledge
└── specs/                 # Your feature specifications
```

---

## Limitations & Honest Notes

**This isn't magic**:
- You still need to review AI output
- Commands can fail if your codebase is unusual
- Basepoints need maintenance as your codebase evolves

**Detection isn't perfect**:
- Some tech stacks are harder to detect than others
- You may need to manually correct detection results
- Research can fail if libraries are internal/obscure

**Requires structure**:
- Works best with organized codebases
- Benefits from clear module boundaries
- Struggles with very small (<100 lines) or very large (>100K lines) projects

**Validation depends on you**:
- Uses your build/test/lint commands—if they're broken, validation is broken
- Only validates what you configure—won't catch everything

---

## Getting Started

1. **Install**: Run the installation script (see INSTALLATION-GUIDE.md)
2. **Setup**: Run setup commands in order (`adapt-to-product` → `create-basepoints` → `deploy-agents`)
3. **Use**: Run development commands per feature (`shape-spec` → `write-spec` → `create-tasks` → `implement-tasks`)

**Detailed guides**:
- [INSTALLATION-GUIDE.md](docs/INSTALLATION-GUIDE.md) - Step-by-step installation
- [COMMAND-FLOWS.md](docs/COMMAND-FLOWS.md) - Detailed command documentation
- [KNOWLEDGE-SYSTEMS.md](docs/KNOWLEDGE-SYSTEMS.md) - How knowledge flows
- [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) - Common issues and solutions
- [command-references/](docs/command-references/) - Per-command visual guides
- [REFACTORING-GUIDELINES.md](docs/REFACTORING-GUIDELINES.md) - How to maintain templates

---

## Credits

Geist builds on the foundational concepts from [Agent OS](https://buildermethods.com/agent-os) by Brian Casel @ Builder Methods—the spec-driven workflow, command structure, and knowledge extraction patterns. Geist extends these ideas into a complete cognitive architecture for agentic development, adding auto-detection, basepoints generation, knowledge accumulation, and support for any project type.

---

**Last Updated**: 2026-01-18
