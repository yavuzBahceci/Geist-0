# profiles/default - Agent OS Template for Any Project

This is the default template that gets installed into your project. It provides abstract, project-agnostic commands and workflows that become specific to your project after specialization.

---

## What This Is

**In simple terms**: A set of commands you can run in your AI chat (Cursor, Claude Desktop, etc.) that help you build features using spec-driven development. The commands automatically know about your project's patterns, architecture, and conventions—because they're specialized to your codebase.

**What it does**:
- Documents your codebase patterns into "basepoints" (living documentation)
- Transforms abstract commands → project-specific commands
- Chains commands together so each step provides context for the next
- Validates with your actual build/test/lint commands

**What it doesn't do**:
- It's not a runtime system or framework
- It doesn't execute code—it generates prompts and files
- It doesn't replace your build tools—it uses them for validation
- It's not magic—you still need to review and guide the AI

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
  └──────────┬────────────────────────────┘
             │
             │ Creates Files (Required by Step 3)
             ▼
  ┌───────────────────────────────────────┐
  │ 📄 basepoints/headquarter.md          │
  │ 📁 basepoints/[layers]/[modules]/     │
  │    📄 agent-base-*.md                 │
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
  ┃  Step 4: /cleanup-agent-os (Optional)                      ┃
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
  │    │  └─ Used by steps 2, 3, 4        │
  │    └─ detected-layer.txt              │
  │       └─ Used by steps 2, 3, 4        │
  └──────────┬────────────────────────────┘
             │
             │ ═══════════════════════════
             │ DEPENDENCY: Step 2 requires requirements.md
             │ Also: basepoints-knowledge.md used by steps 2,3,4
             ▼
  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
  ┃  Step 2: /write-spec                                       ┃
  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
  
  Inputs:
  ┌───────────────────────────────────────┐
  │ ✅ requirements.md (from Step 1)      │
  │ ✅ basepoints-knowledge.md (Step 1)   │
  │ ✅ detected-layer.txt (Step 1)        │
  │ • basepoints/**/*.md (still available)│
  └──────────┬────────────────────────────┘
             │
             │ Process
             ▼
  ┌───────────────────────────────────────┐
  │ 1. Read requirements + cached knowledge│
  │ 2. Reference your patterns            │
  │ 3. Write specification                │
  └──────────┬────────────────────────────┘
             │
             │ Creates Files (Required by Step 3)
             ▼
  ┌───────────────────────────────────────┐
  │ 📄 specs/[name]/spec.md               │
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
  │ ✅ basepoints-knowledge.md (cached)   │
  └──────────┬────────────────────────────┘
             │
             │ Process
             ▼
  ┌───────────────────────────────────────┐
  │ 1. Read specification                 │
  │ 2. Break into tasks                   │
  │ 3. Add acceptance criteria            │
  └──────────┬────────────────────────────┘
             │
             │ Creates Files (Required by Step 4)
             ▼
  ┌───────────────────────────────────────┐
  │ 📄 specs/[name]/tasks.md              │
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
  │ ✅ basepoints-knowledge.md (cached)   │
  │ ✅ detected-layer.txt (cached)        │
  │ • basepoints/**/*.md (for patterns)   │
  └──────────┬────────────────────────────┘
             │
             │ Process
             ▼
  ┌───────────────────────────────────────┐
  │ 1. Read tasks + cached knowledge      │
  │ 2. Use your patterns                  │
  │ 3. Implement code                     │
  │ 4. Validate with YOUR commands        │
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

## Visual: Complete Command Chain with File Dependencies

### Setup Chain (Files Created and Used)

```
┌──────────────────────────────────────────────────────────────────┐
│            SETUP CHAIN - FILE DEPENDENCY FLOW                    │
└──────────────────────────────────────────────────────────────────┘

  /adapt-to-product
  ┌─────────────────────────────────────────┐
  │  INPUT:  Your codebase                 │
  │  OUTPUT: Creates files below           │
  └────────────┬────────────────────────────┘
               │
               │ Creates 5 outputs:
               ├───► 📄 product/mission.md
               ├───► 📄 product/roadmap.md
               ├───► 📄 product/tech-stack.md
               ├───► 📄 config/project-profile.yml
               └───► 📁 config/enriched-knowledge/
                     │
                     │ All required by next command
                     ▼
  /create-basepoints
  ┌─────────────────────────────────────────┐
  │  INPUT:  Reads files from above        │
  │          ✅ product/*.md               │
  │          ✅ config/project-profile.yml │
  │  OUTPUT: Creates files below           │
  └────────────┬────────────────────────────┘
               │
               │ Creates basepoints files:
               ├───► 📄 basepoints/headquarter.md
               └───► 📁 basepoints/[layers]/[modules]/
                     └───► 📄 agent-base-*.md (per module)
                           │
                           │ Required by next command
                           ▼
  /deploy-agents
  ┌─────────────────────────────────────────┐
  │  INPUT:  Reads ALL files from above    │
  │          ✅ product/*.md               │
  │          ✅ basepoints/**/*.md         │
  │          ✅ config/*.yml               │
  │  OUTPUT: Specialized commands/         │
  └─────────────────────────────────────────┘
```

### Development Chain (File Flow Per Feature)

```
┌──────────────────────────────────────────────────────────────────┐
│       DEVELOPMENT CHAIN - FILE DEPENDENCY FLOW                   │
└──────────────────────────────────────────────────────────────────┘

  /shape-spec "Add payment"
  ┌─────────────────────────────────────────┐
  │  INPUT:                                 │
  │  • Feature description (your text)      │
  │  • basepoints/**/*.md (from setup)      │
  │                                         │
  │  OUTPUT: Creates 3 files                │
  └────────────┬────────────────────────────┘
               │
               │ Creates files:
               ├───► 📄 specs/payment/planning/
               │           requirements.md
               │           │
               │           └─► Required by write-spec
               │
               ├───► 📄 specs/payment/implementation/
               │           cache/basepoints-knowledge.md
               │           │
               │           └─► Used by write-spec, create-tasks,
               │               implement-tasks (cached)
               │
               └───► 📄 specs/payment/implementation/
                         cache/detected-layer.txt
                         │
                         └─► Used by write-spec, create-tasks,
                             implement-tasks (cached)
               │
               │ ═══════════════════════════════════════════════
               │ Next command reads these files
               ▼
  /write-spec
  ┌─────────────────────────────────────────┐
  │  INPUT:  Reads files from shape-spec    │
  │          ✅ requirements.md             │
  │          ✅ basepoints-knowledge.md     │
  │          ✅ detected-layer.txt          │
  │                                         │
  │  OUTPUT: Creates 1 file                 │
  └────────────┬────────────────────────────┘
               │
               │ Creates file:
               └───► 📄 specs/payment/spec.md
                     │
                     └─► Required by create-tasks
               │
               │ ═══════════════════════════════════════════════
               │ Next command reads this file
               ▼
  /create-tasks
  ┌─────────────────────────────────────────┐
  │  INPUT:  Reads files from previous      │
  │          ✅ spec.md (from write-spec)   │
  │          ✅ basepoints-knowledge.md     │
  │             (still cached from shape)   │
  │                                         │
  │  OUTPUT: Creates 1 file                 │
  └────────────┬────────────────────────────┘
               │
               │ Creates file:
               └───► 📄 specs/payment/tasks.md
                     │
                     └─► Required by implement-tasks
               │
               │ ═══════════════════════════════════════════════
               │ Next command reads this file
               ▼
  /implement-tasks  OR  /orchestrate-tasks
  ┌─────────────────────────────────────────┐
  │  INPUT:  Reads files from previous      │
  │          ✅ tasks.md (from create-tasks)│
  │          ✅ basepoints-knowledge.md     │
  │             (still cached from shape)   │
  │          ✅ detected-layer.txt          │
  │             (still cached from shape)   │
  │                                         │
  │  OUTPUT: Creates code + report          │
  └────────────┬────────────────────────────┘
               │
               │ Creates:
               ├───► 📝 Code changes (you review)
               └───► 📄 specs/payment/implementation/
                         cache/validation-report.md
```

**Key Visual Rule**:
```
Command N creates files → Command N+1 reads those files → Command N+1 creates new files → Command N+2 reads those files...

Breaking the chain = missing files = next command fails
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
- `agent-os/product/mission.md`
- `agent-os/product/roadmap.md`
- `agent-os/product/tech-stack.md`
- `agent-os/config/project-profile.yml`
- `agent-os/config/enriched-knowledge/`

**Time**: ~5-10 minutes (mostly waiting for research)

---

#### `/create-basepoints`
**Input**: Product files (from step 1) + your codebase  
**Process**:
1. Analyzes your directory structure
2. Detects abstraction layers (UI, API, data, etc.)
3. Documents patterns per module
4. Creates comprehensive basepoints files

**Output Files**:
- `agent-os/basepoints/headquarter.md` (project overview)
- `agent-os/basepoints/[layer]/[module]/agent-base-*.md` (one per module)

**Time**: ~10-20 minutes (depends on codebase size)

---

#### `/deploy-agents`
**Input**: Everything from steps 1 & 2  
**Process**:
1. Reads all knowledge (product, basepoints, profile)
2. Transforms abstract templates → project-specific
3. Replaces `{{PROJECT_BUILD_COMMAND}}` → `npm run build`
4. Injects your patterns into commands
5. Configures validation commands

**Output**: Specialized `agent-os/commands/` and `agent-os/workflows/`

**Time**: ~2-5 minutes

---

#### `/cleanup-agent-os`
**Input**: Specialized commands (from step 3)  
**Process**:
1. Removes any remaining placeholders
2. Cleans unused logic
3. Verifies knowledge completeness

**Output**: Cleanup report

**Time**: ~1-2 minutes

---

### Development Commands (Run Per Feature)

#### `/shape-spec "Feature description"`
**Input**: Feature description (your text)  
**Process**:
1. Extracts relevant basepoints for this feature
2. Detects abstraction layer (UI/API/data)
3. Asks clarifying questions informed by your patterns
4. Caches extracted knowledge for next commands

**Output Files**:
- `agent-os/specs/[name]/planning/requirements.md`
- `agent-os/specs/[name]/implementation/cache/basepoints-knowledge.md`

**What it does NOT do**: It doesn't write code yet—just shapes requirements.

---

#### `/write-spec`
**Input**: `requirements.md` (from shape-spec) + cached basepoints  
**Process**:
1. Reads requirements and basepoints knowledge
2. References your standards and patterns
3. Suggests reusable code from basepoints
4. Writes detailed specification

**Output Files**:
- `agent-os/specs/[name]/spec.md`

**What it does NOT do**: Still no code—just a specification document.

---

#### `/create-tasks`
**Input**: `spec.md` (from write-spec)  
**Process**:
1. Reads specification
2. Breaks it into actionable tasks
3. Groups related tasks
4. Adds acceptance criteria

**Output Files**:
- `agent-os/specs/[name]/tasks.md`

**What it does NOT do**: Still no code—just a task breakdown.

---

#### `/implement-tasks` OR `/orchestrate-tasks`
**Input**: `tasks.md` (from create-tasks) + cached basepoints  
**Process**:
1. Reads tasks and basepoints knowledge
2. Uses your coding patterns and standards
3. Implements code changes
4. Validates with your build/test/lint commands
5. Reports results

**Output**: Code changes (files you review) + validation report

**This is where code actually gets written.**

---

## Visual: Command Dependencies

```
                  adapt-to-product
                         │
                         ├──► product/*.md
                         ├──► config/project-profile.yml
                         └──► config/enriched-knowledge/
                                  │
                                  │ depends on
                                  ▼
                         create-basepoints
                                  │
                                  ├──► basepoints/headquarter.md
                                  └──► basepoints/[layers]/[modules]/agent-base-*.md
                                           │
                                           │ depends on
                                           ▼
                                  deploy-agents
                                           │
                                           └──► Specialized commands/
                                                └──► (ready to use)
                                                     │
                                                     │ used by
                                                     ▼
                                            shape-spec
                                                     │
                                                     ├──► specs/[name]/planning/requirements.md
                                                     └──► specs/[name]/implementation/cache/basepoints-knowledge.md
                                                              │
                                                              │ depends on
                                                              ▼
                                                     write-spec
                                                              │
                                                              └──► specs/[name]/spec.md
                                                                   │
                                                                   │ depends on
                                                                   ▼
                                                            create-tasks
                                                                   │
                                                                   └──► specs/[name]/tasks.md
                                                                        │
                                                                        │ depends on
                                                                        ▼
                                                               implement-tasks
                                                                        │
                                                                        └──► Code changes
```

**Red arrows** = You must run these in order  
**Green arrows** = Commands read outputs from previous steps

---

## How Workflows Compose Commands: Internal Structure

Commands don't execute code directly—they chain workflows together in sequence. Each command is a series of workflow calls that happen in order.

### Visual: Command Internal Workflow Chain

```
┌─────────────────────────────────────────────────────────────────┐
│         COMMAND = SEQUENCE OF WORKFLOW CALLS (IN ORDER)          │
└─────────────────────────────────────────────────────────────────┘

  User runs: /shape-spec "Add payment"
       │
       ▼
  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
  ┃  Command: shape-spec                                        ┃
  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
       │
       │ ═════════════════════════════════════════════════════
       │ Step 1: Extract Basepoints Knowledge
       ▼
┌─────────────────────────────────────────────────────────────┐
  │  Workflow: extract-basepoints-with-scope-detection         │
  │                                                             │
  │  This workflow internally chains 4 sub-workflows:          │
  │                                                             │
  │    ┌─────────────────────────────────────────────────┐     │
  │    │ 1. extract-basepoints-knowledge-automatic      │     │
  │    │    │                                            │     │
  │    │    └─► Reads basepoints/**/*.md                │     │
  │    │    └─► Extracts patterns                       │     │
  │    └─────────────────────────────────────────────────┘     │
  │           │                                                │
  │           ▼                                                │
  │    ┌─────────────────────────────────────────────────┐     │
  │    │ 2. detect-abstraction-layer                     │     │
  │    │    │                                            │     │
  │    │    └─► Detects: UI/API/data/platform          │     │
  │    └─────────────────────────────────────────────────┘     │
  │           │                                                │
  │           ▼                                                │
  │    ┌─────────────────────────────────────────────────┐     │
  │    │ 3. detect-scope-semantic-analysis               │     │
  │    │    │                                            │     │
  │    │    └─► Finds relevant modules semantically     │     │
  │    └─────────────────────────────────────────────────┘     │
  │           │                                                │
  │           ▼                                                │
  │    ┌─────────────────────────────────────────────────┐     │
  │    │ 4. detect-scope-keyword-matching                │     │
  │    │    │                                            │     │
  │    │    └─► Matches keywords to modules             │     │
  │    └─────────────────────────────────────────────────┘     │
  │                                                             │
  │  OUTPUT: Caches knowledge                                   │
  │  ┌───────────────────────────────────────────────┐         │
  │  │ specs/[name]/implementation/cache/             │         │
  │  │   ├─ basepoints-knowledge.md                  │         │
  │  │   └─ detected-layer.txt                       │         │
  │  └───────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────┘
       │
       │ ═════════════════════════════════════════════════════
       │ Step 2: Research Requirements
       │        (Uses files from Step 1)
       ▼
┌─────────────────────────────────────────────────────────────┐
  │  Workflow: research-spec                                    │
  │                                                             │
  │  INPUTS:                                                    │
  │  ✅ basepoints-knowledge.md (from Step 1)                  │
  │  ✅ detected-layer.txt (from Step 1)                        │
  │  • Your feature description                                │
  │                                                             │
  │  PROCESS:                                                   │
  │  1. Load cached knowledge                                  │
  │  2. Ask clarifying questions (informed by patterns)        │
  │  3. Gather requirements                                    │
  │                                                             │
  │  OUTPUT:                                                    │
  │  📄 specs/[name]/planning/requirements.md                  │
  └─────────────────────────────────────────────────────────────┘
       │
       │ ═════════════════════════════════════════════════════
       │ Step 3: Validate Outputs
       ▼
  ┌─────────────────────────────────────────────────────────────┐
  │  Workflow: validate-output-exists                           │
  │                                                             │
  │  CHECKS:                                                    │
  │  ✅ requirements.md exists?                                │
  │  ✅ basepoints-knowledge.md exists?                        │
  │                                                             │
  │  If all checks pass → Command succeeds                      │
└─────────────────────────────────────────────────────────────┘
```

### Workflow Reuse: How Same Workflows Are Used Across Commands

The same workflows are reused across multiple commands for consistency:

```
┌─────────────────────────────────────────────────────────────────┐
│              WORKFLOW REUSE VISUAL                               │
└─────────────────────────────────────────────────────────────────┘

  Shared Workflow: extract-basepoints-with-scope-detection
┌─────────────────────────────────────────────────────────────┐
  │                                                             │
  │  (Defined once, used everywhere)                           │
  │                                                             │
  └────────────┬────────────────────────────────────────────────┘
               │
               │ Used by 5 commands:
               │
               ├──────────┬──────────┬──────────┬──────────┬──────────┐
               │          │          │          │          │          │
               ▼          ▼          ▼          ▼          ▼          ▼
      ┌──────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌──────────────┐
      │shape-spec│  │write-   │  │create-  │  │implement│  │orchestrate  │
      │          │  │spec     │  │tasks    │  │-tasks   │  │-tasks       │
      │          │  │         │  │         │  │         │  │             │
      │ Step 1   │  │ Step 1  │  │ Step 1  │  │ Step 1  │  │ Step 1      │
      └──────────┘  └─────────┘  └─────────┘  └─────────┘  └──────────────┘
      
      All 5 commands use the same workflow → Consistent behavior
      
      Update workflow once → All 5 commands benefit automatically
```

**Why this matters**: 
- **Single source of truth**: Update the workflow once, all commands benefit
- **Consistency**: All commands extract basepoints the same way
- **Maintainability**: Changes propagate automatically

---

## How Specialization Works

Specialization transforms abstract templates into project-specific commands by replacing placeholders with actual values.

### Before Specialization (Abstract Template)

```markdown
# Validate Implementation

Run validation commands:

```bash
BUILD_CMD="{{PROJECT_BUILD_COMMAND}}"
TEST_CMD="{{PROJECT_TEST_COMMAND}}"
LINT_CMD="{{PROJECT_LINT_COMMAND}}"

$BUILD_CMD
$TEST_CMD
$LINT_CMD
```

See {{workflows/validation/validate-implementation}} for details.
```

### After Specialization (Project-Specific)

```markdown
# Validate Implementation

Run validation commands:

```bash
BUILD_CMD="npm run build"
TEST_CMD="npm test"
LINT_CMD="npm run lint"

npm run build
npm test
npm run lint
```

See workflows/validation/validate-implementation.md for details.
```

**What changed**:
- `{{PROJECT_BUILD_COMMAND}}` → `npm run build` (from detection)
- `{{workflows/...}}` → actual file path (compiled)
- Template → specialized command

**When this happens**: During `/deploy-agents`

---

## How Knowledge Gets Injected

Basepoints knowledge is automatically extracted and cached when commands run:

```
Command starts
     │
     ▼
Extract basepoints knowledge
     │
     ├─► Reads: basepoints/headquarter.md
     ├─► Reads: basepoints/[relevant-modules]/*.md
     ├─► Detects: Abstraction layer (UI/API/data)
     └─► Caches: To implementation/cache/basepoints-knowledge.md
          │
          │ (this file gets read by next commands)
          ▼
Command uses knowledge in prompts
     │
     ├─► "Here are patterns from your codebase: ..."
     ├─► "Your standards require: ..."
     └─► "Reusable code you already have: ..."
```

**Example flow**:

```
shape-spec extracts basepoints
  └─► Caches to: specs/my-feature/implementation/cache/basepoints-knowledge.md

write-spec reads cached knowledge
  └─► Uses patterns in spec writing: "Your codebase uses React patterns X, Y, Z"

create-tasks reads cached knowledge
  └─► Uses standards in task breakdown: "Follow your existing test patterns"

implement-tasks reads cached knowledge
  └─► Uses patterns in code generation: "Use your existing Button component pattern"
```

---

## Common Patterns

### Pattern 1: Basepoints Extraction (Used in 5 Commands)

These commands all start by extracting basepoints knowledge:

- `shape-spec`
- `write-spec`
- `create-tasks`
- `implement-tasks`
- `orchestrate-tasks`

**How it works**:

```bash
# Every command does this first:
{{workflows/common/extract-basepoints-with-scope-detection}}
```

This single line expands to:
1. Check if basepoints exist
2. Extract relevant patterns
3. Detect abstraction layer
4. Cache knowledge for next commands

**Why**: Ensures consistent knowledge extraction across all commands.

---

### Pattern 2: Command Chaining via Cache Files

Commands pass knowledge through cache files:

```
shape-spec
  └─► Writes: implementation/cache/basepoints-knowledge.md
       │
       │ (next command reads this)
       ▼
write-spec
  └─► Reads: basepoints-knowledge.md
  └─► Writes: spec.md
       │
       │ (next command reads this)
       ▼
create-tasks
  └─► Reads: spec.md
  └─► Writes: tasks.md
```

**Cache directory structure**:

```
specs/my-feature/
└── implementation/
    └── cache/
        ├── basepoints-knowledge.md  (from shape-spec)
        ├── detected-layer.txt       (from shape-spec)
        └── validation-report.md     (from implement-tasks)
```

---

### Pattern 3: Validation at Each Step

Every command validates its outputs:

```
shape-spec
  └─► Validates: requirements.md exists, basepoints extracted

write-spec
  └─► Validates: spec.md exists, knowledge was used

create-tasks
  └─► Validates: tasks.md exists, all tasks have acceptance criteria

implement-tasks
  └─► Validates: Code builds, tests pass, linter passes
```

**Why**: Catches errors early, before they compound.

---

## What Gets Specialized

During `/deploy-agents`, these get transformed:

| What | Before (Abstract) | After (Specialized) |
|------|-------------------|---------------------|
| **Validation commands** | `{{PROJECT_BUILD_COMMAND}}` | `npm run build` |
| **File paths** | `{{BASEPOINTS_PATH}}` | `agent-os/basepoints` |
| **Patterns** | Generic descriptions | Your actual patterns |
| **Standards** | Abstract standards | Your project standards |
| **Workflow references** | `{{workflows/...}}` | Actual file paths |

**What doesn't get specialized**: The structure—commands, workflows, and file organization stay the same.

---

## File Structure

```
profiles/default/
├── commands/                    # Abstract commands (templates)
│   ├── adapt-to-product/        # Setup: Extract product info
│   ├── plan-product/            # Setup: Plan new product
│   ├── create-basepoints/       # Setup: Create codebase docs
│   ├── deploy-agents/           # Setup: Specialize commands
│   ├── cleanup-agent-os/        # Maintenance: Verify deployment
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
    ├── WORKFLOW-MAP.md          # Visual workflow map
    ├── INSTALLATION-GUIDE.md    # Installation guide
    ├── PATH-REFERENCE-GUIDE.md  # Path conventions
    ├── REFACTORING-GUIDELINES.md
    ├── TECHNOLOGY-AGNOSTIC-BEST-PRACTICES.md
    └── command-references/      # Per-command visual guides
```

**After specialization** (in your project's `agent-os/` folder):

```
agent-os/
├── commands/              # Specialized commands (project-specific)
├── workflows/             # Specialized workflows
├── basepoints/            # Your codebase documentation
├── product/               # Your product files
├── config/                # Project profile + enriched knowledge
└── specs/                 # Your feature specifications
```

---

## Important Concepts

### Basepoints
**What**: Living documentation of your codebase patterns, architecture, and decisions.  
**Where**: `agent-os/basepoints/`  
**Used by**: All development commands to inject your patterns into AI prompts.

### Workflow References
**What**: `{{workflows/...}}` syntax that compiles to actual file content.  
**How**: During specialization, references expand to full file content.  
**Why**: Keeps templates DRY—define once, use everywhere.

### Placeholders
**What**: `{{PROJECT_BUILD_COMMAND}}` syntax for project-specific values.  
**Replaced by**: Actual values detected during `adapt-to-product`.  
**Example**: `{{PROJECT_BUILD_COMMAND}}` → `npm run build`

### Command Chaining
**What**: Commands read outputs from previous commands.  
**Why**: Each step provides context for the next.  
**Rule**: Must run commands in order—they depend on each other.

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
- [WORKFLOW-MAP.md](docs/WORKFLOW-MAP.md) - Visual workflow reference
- [command-references/](docs/command-references/) - Per-command visual guides
- [REFACTORING-GUIDELINES.md](docs/REFACTORING-GUIDELINES.md) - How to maintain templates

---

**Last Updated**: 2026-01-18
