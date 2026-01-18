# Geist

### Agent OS, for any project.

Geist extends [Agent OS](https://buildermethods.com/agent-os) to work with any project—not just full-stack web apps. It detects your tech stack automatically, documents your codebase patterns, and specializes commands to match how you actually build.

---

## What This Is

**In simple terms**: A tool that installs a set of AI commands into your project. These commands know about your codebase because they're specialized to it. When you run commands like `/shape-spec` or `/implement-tasks`, the AI gets your project's patterns, architecture, and conventions as context—automatically.

**What it does**:
- Installs abstract command templates into your project
- Detects your tech stack, build commands, and architecture
- Documents your codebase patterns into "basepoints"
- Specializes commands to use your patterns and conventions
- Works with any project type (web, mobile, CLI, embedded, etc.)

**What it doesn't do**:
- It's not a framework or runtime—just command templates
- It doesn't execute code—generates prompts and files
- It doesn't replace your tools—uses them for validation
- It's not magic—you still review and guide the AI

---

## The Core Concept

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. Install Templates     2. Specialize     3. Use Commands    │
│     ┌──────────────┐         ┌─────────┐      ┌───────────┐   │
│     │   Abstract   │    →    │ Project │  →   │   AI      │   │
│     │  Templates   │         │ Specific│      │ Commands  │   │
│     │              │         │         │      │ with YOUR │   │
│     │ Any project  │         │ YOUR    │      │ patterns  │   │
│     │ type         │         │ project │      │           │   │
│     └──────────────┘         └─────────┘      └───────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Key idea**: Templates are project-agnostic. Specialization makes them project-specific. You get commands that understand YOUR codebase, not generic ones that assume web development.

---

## How It Works: The Complete Flow

### Step 1: Install (One-Time)

```bash
# Clone Geist
git clone <repo-url> ~/geist

# Install into your project
cd /path/to/your/project
~/geist/scripts/project-install.sh --profile default
```

**What happens**:
```
┌─────────────────────────────────────────────────────────────────┐
│  INSTALLATION                                                  │
│                                                                 │
│  ~/geist/profiles/default/                                     │
│  ├── commands/         (abstract templates)                    │
│  ├── workflows/        (reusable patterns)                     │
│  └── standards/        (generic standards)                     │
│       │                                                         │
│       ▼ compilation                                             │
│  your-project/agent-os/                                        │
│  ├── commands/         (installed templates)                   │
│  ├── workflows/        (installed templates)                   │
│  └── standards/        (installed templates)                   │
│                                                                 │
│  At this point: Still abstract, not specialized                │
└─────────────────────────────────────────────────────────────────┘
```

---

### Step 2: Specialize (One-Time Setup)

You run three commands to specialize the templates for your project:

```
┌─────────────────────────────────────────────────────────────────┐
│                    SPECIALIZATION PHASE                         │
└─────────────────────────────────────────────────────────────────┘

  /adapt-to-product
  │
  ├─ Detects: Tech stack from config files (package.json, etc.)
  ├─ Researches: Best practices, CVEs, known issues
  ├─ Asks: Only 2-3 questions (compliance, review preference)
  │
  └─ Creates:
      ├─ product/mission.md
      ├─ product/roadmap.md
      ├─ product/tech-stack.md
      ├─ config/project-profile.yml
      └─ config/enriched-knowledge/
       │
       ▼
  /create-basepoints
  │
  ├─ Reads: Product files (from step 1)
  ├─ Analyzes: Your codebase structure
  ├─ Documents: Patterns per module
  │
  └─ Creates:
      ├─ basepoints/headquarter.md (project overview)
      └─ basepoints/[layers]/[modules]/agent-base-*.md
       │
       ▼
  /deploy-agents
  │
  ├─ Reads: Product files + basepoints (from steps 1 & 2)
  ├─ Transforms: Abstract templates → Project-specific
  ├─ Replaces: {{PLACEHOLDERS}} → actual values
  │   • {{PROJECT_BUILD_COMMAND}} → npm run build
  │   • Generic patterns → YOUR patterns
  │
  └─ Outputs:
      └─ Specialized commands/ (ready to use)
       │
       ▼
  ✅ READY - Commands now know your project
```

**Time**: ~20-30 minutes total (most of it is automatic)

---

### Step 3: Use Commands (Per Feature)

After specialization, you use commands to build features. **These must run in order**—each command depends on outputs from the previous one:

```
┌─────────────────────────────────────────────────────────────────┐
│              COMMAND CHAIN (Run in Order - Required)            │
└─────────────────────────────────────────────────────────────────┘

  Step 1: /shape-spec "Feature description"
  ┌─────────────────────────────────────────────────────────────┐
  │  INPUTS:                                                    │
  │  • Your feature description (from you)                      │
  │  • basepoints/ (from specialization)                        │
  │                                                             │
  │  PROCESS:                                                   │
  │  1. Extract relevant basepoints for this feature           │
  │  2. Detect abstraction layer (UI/API/data)                 │
  │  3. Ask clarifying questions informed by your patterns     │
  │                                                             │
  │  OUTPUTS:                                                   │
  │  📄 specs/[name]/planning/requirements.md                  │
  │  📄 specs/[name]/implementation/cache/                      │
  │      ├─ basepoints-knowledge.md                            │
  │      └─ detected-layer.txt                                 │
  └─────────────────────────────────────────────────────────────┘
                           │
                           │ (next command reads these files)
                           ▼
  Step 2: /write-spec
  ┌─────────────────────────────────────────────────────────────┐
  │  INPUTS:                                                    │
  │  • requirements.md (from step 1)                           │
  │  • basepoints-knowledge.md (from step 1)                   │
  │  • detected-layer.txt (from step 1)                        │
  │                                                             │
  │  PROCESS:                                                   │
  │  1. Read requirements and cached basepoints                │
  │  2. Reference your standards and patterns                  │
  │  3. Suggest reusable code from basepoints                  │
  │  4. Write detailed specification                           │
  │                                                             │
  │  OUTPUTS:                                                   │
  │  📄 specs/[name]/spec.md                                   │
  └─────────────────────────────────────────────────────────────┘
                           │
                           │ (next command reads this file)
                           ▼
  Step 3: /create-tasks
  ┌─────────────────────────────────────────────────────────────┐
  │  INPUTS:                                                    │
  │  • spec.md (from step 2)                                   │
  │  • basepoints-knowledge.md (from step 1, still available)  │
  │                                                             │
  │  PROCESS:                                                   │
  │  1. Read specification                                     │
  │  2. Break into actionable tasks                            │
  │  3. Group related tasks                                    │
  │  4. Add acceptance criteria                                │
  │                                                             │
  │  OUTPUTS:                                                   │
  │  📄 specs/[name]/tasks.md                                  │
  └─────────────────────────────────────────────────────────────┘
                           │
                           │ (next command reads this file)
                           ▼
  Step 4: /implement-tasks  OR  /orchestrate-tasks
  ┌─────────────────────────────────────────────────────────────┐
  │  INPUTS:                                                    │
  │  • tasks.md (from step 3)                                  │
  │  • basepoints-knowledge.md (from step 1, still available)  │
  │  • detected-layer.txt (from step 1)                        │
  │                                                             │
  │  PROCESS:                                                   │
  │  1. Read tasks and cached basepoints                       │
  │  2. Use your coding patterns and standards                 │
  │  3. Implement code changes                                 │
  │  4. Validate with your build/test/lint commands            │
  │                                                             │
  │  OUTPUTS:                                                   │
  │  📝 Code changes (you review these)                        │
  │  📄 specs/[name]/implementation/cache/validation-report.md │
  └─────────────────────────────────────────────────────────────┘

⚠️  ORDER MATTERS: Each command reads outputs from the previous command.
   Skipping steps will fail—commands depend on files created earlier.
```

**Key Rules**:
1. **Run in order**: shape-spec → write-spec → create-tasks → implement-tasks
2. **Each command creates files the next command needs**
3. **Cache files persist**: `basepoints-knowledge.md` from step 1 is used by steps 2, 3, and 4
4. **Cannot skip steps**: Each command requires files from previous commands

---

## Visual: Complete Command Chain with File Dependencies

### Setup Chain (One-Time)

```
┌──────────────────────────────────────────────────────────────────┐
│              SETUP COMMAND CHAIN (Must Run in Order)             │
└──────────────────────────────────────────────────────────────────┘

  Command 1: /adapt-to-product
  ┌─────────────────────────────┐
  │  Reads:                     │
  │  • package.json             │
  │  • Cargo.toml               │
  │  • go.mod                   │
  │  • Your codebase            │
  └───────────┬─────────────────┘
              │
              │ Creates:
              ▼
  ┌───────────────────────────────────────────┐
  │  OUTPUT FILES                             │
  │  ├─ product/mission.md                    │
  │  ├─ product/roadmap.md                    │
  │  ├─ product/tech-stack.md                 │
  │  ├─ config/project-profile.yml            │
  │  └─ config/enriched-knowledge/            │
  └───────────┬───────────────────────────────┘
              │
              │ Command 2 reads these files
              ▼
  Command 2: /create-basepoints
  ┌─────────────────────────────┐
  │  Reads:                     │
  │  • product/*.md (from #1)   │
  │  • config/project-profile   │
  │    .yml (from #1)           │
  │  • Your codebase structure  │
  └───────────┬─────────────────┘
              │
              │ Creates:
              ▼
  ┌───────────────────────────────────────────┐
  │  OUTPUT FILES                             │
  │  ├─ basepoints/headquarter.md             │
  │  └─ basepoints/[layers]/[modules]/        │
  │     └─ agent-base-*.md                    │
  └───────────┬───────────────────────────────┘
              │
              │ Command 3 reads these files
              ▼
  Command 3: /deploy-agents
  ┌─────────────────────────────┐
  │  Reads:                     │
  │  • product/*.md (from #1)   │
  │  • basepoints/**/*.md       │
  │    (from #2)                │
  │  • config/*.yml (from #1)   │
  └───────────┬─────────────────┘
              │
              │ Transforms:
              ▼
  ┌───────────────────────────────────────────┐
  │  OUTPUT FILES                             │
  │  └─ commands/ (specialized)               │
  │     └─ Ready to use                       │
  └───────────────────────────────────────────┘

  ⚠️  DEPENDENCY: Each command requires outputs from previous command.
     Cannot skip steps—order is enforced by file dependencies.
```

### Development Chain (Per Feature)

```
┌──────────────────────────────────────────────────────────────────┐
│         DEVELOPMENT COMMAND CHAIN (Run Sequentially)             │
└──────────────────────────────────────────────────────────────────┘

  Command 1: /shape-spec "Add user authentication"
  ┌─────────────────────────────────────────────────────────────┐
  │  READS:                                                     │
  │  ┌─────────────────────────────────────────────────────┐   │
  │  │ • basepoints/headquarter.md (from setup)            │   │
  │  │ • basepoints/**/agent-base-*.md (from setup)        │   │
  │  │ • Your feature description (your input)             │   │
  │  └─────────────────────────────────────────────────────┘   │
  │                                                             │
  │  PROCESS:                                                   │
  │  ┌─────────────────────────────────────────────────────┐   │
  │  │ 1. Extract relevant patterns                        │   │
  │  │ 2. Detect abstraction layer                         │   │
  │  │ 3. Ask clarifying questions                          │   │
  │  └─────────────────────────────────────────────────────┘   │
  │                                                             │
  │  WRITES:                                                    │
  │  ┌─────────────────────────────────────────────────────┐   │
  │  │ 📄 specs/auth/planning/requirements.md             │   │
  │  │ 📄 specs/auth/implementation/cache/                 │   │
  │  │    ├─ basepoints-knowledge.md                       │   │
  │  │    └─ detected-layer.txt                            │   │
  │  └─────────────────────────────────────────────────────┘   │
  └─────────────────────────────────────────────────────────────┘
                           │
                           │ ════════════════════════════════════
                           │ DEPENDENCY: write-spec needs these files
                           ▼
  Command 2: /write-spec
  ┌─────────────────────────────────────────────────────────────┐
  │  READS:                                                     │
  │  ┌─────────────────────────────────────────────────────┐   │
  │  │ ✅ requirements.md (from command 1)                 │   │
  │  │ ✅ basepoints-knowledge.md (from command 1)         │   │
  │  │ ✅ detected-layer.txt (from command 1)              │   │
  │  │ ✅ basepoints/**/*.md (still available)             │   │
  │  └─────────────────────────────────────────────────────┘   │
  │                                                             │
  │  PROCESS:                                                   │
  │  ┌─────────────────────────────────────────────────────┐   │
  │  │ 1. Read requirements and cached knowledge           │   │
  │  │ 2. Reference your patterns from basepoints          │   │
  │  │ 3. Write detailed specification                     │   │
  │  └─────────────────────────────────────────────────────┘   │
  │                                                             │
  │  WRITES:                                                    │
  │  ┌─────────────────────────────────────────────────────┐   │
  │  │ 📄 specs/auth/spec.md                              │   │
  │  └─────────────────────────────────────────────────────┘   │
  └─────────────────────────────────────────────────────────────┘
                           │
                           │ ════════════════════════════════════
                           │ DEPENDENCY: create-tasks needs this file
                           ▼
  Command 3: /create-tasks
  ┌─────────────────────────────────────────────────────────────┐
  │  READS:                                                     │
  │  ┌─────────────────────────────────────────────────────┐   │
  │  │ ✅ spec.md (from command 2)                         │   │
  │  │ ✅ basepoints-knowledge.md (still cached)           │   │
  │  └─────────────────────────────────────────────────────┘   │
  │                                                             │
  │  PROCESS:                                                   │
  │  ┌─────────────────────────────────────────────────────┐   │
  │  │ 1. Read specification                               │   │
  │  │ 2. Break into actionable tasks                      │   │
  │  │ 3. Group related tasks                              │   │
  │  │ 4. Add acceptance criteria                          │   │
  │  └─────────────────────────────────────────────────────┘   │
  │                                                             │
  │  WRITES:                                                    │
  │  ┌─────────────────────────────────────────────────────┐   │
  │  │ 📄 specs/auth/tasks.md                             │   │
  │  └─────────────────────────────────────────────────────┘   │
  └─────────────────────────────────────────────────────────────┘
                           │
                           │ ════════════════════════════════════
                           │ DEPENDENCY: implement-tasks needs this file
                           ▼
  Command 4: /implement-tasks  OR  /orchestrate-tasks
  ┌─────────────────────────────────────────────────────────────┐
  │  READS:                                                     │
  │  ┌─────────────────────────────────────────────────────┐   │
  │  │ ✅ tasks.md (from command 3)                        │   │
  │  │ ✅ basepoints-knowledge.md (still cached)           │   │
  │  │ ✅ detected-layer.txt (still cached)                │   │
  │  │ ✅ basepoints/**/*.md (for pattern reference)       │   │
  │  └─────────────────────────────────────────────────────┘   │
  │                                                             │
  │  PROCESS:                                                   │
  │  ┌─────────────────────────────────────────────────────┐   │
  │  │ 1. Read tasks and cached knowledge                  │   │
  │  │ 2. Use your coding patterns                         │   │
  │  │ 3. Implement code changes                           │   │
  │  │ 4. Validate with YOUR build/test/lint commands      │   │
  │  └─────────────────────────────────────────────────────┘   │
  │                                                             │
  │  WRITES:                                                    │
  │  ┌─────────────────────────────────────────────────────┐   │
  │  │ 📝 Code changes (you review)                        │   │
  │  │ 📄 specs/auth/implementation/cache/                 │   │
  │  │    └─ validation-report.md                          │   │
  │  └─────────────────────────────────────────────────────┘   │
  └─────────────────────────────────────────────────────────────┘

  🔗 FILE CHAIN: Each command creates files the next command requires.
     Breaking the chain (skipping a command) = missing required files = failure.
```

---

## What Gets Detected

When you run `/adapt-to-product`, Geist automatically detects:

**From Config Files**:
- **Tech stack**: Language, framework, database (from package.json, Cargo.toml, go.mod, etc.)
- **Build commands**: `npm run build`, `cargo build`, etc. (from scripts/Makefile/CI)
- **Test commands**: `npm test`, `cargo test`, etc.
- **Lint commands**: `npm run lint`, `cargo clippy`, etc.

**From Codebase Analysis**:
- **Architecture**: Module boundaries, layer structure
- **Patterns**: How you organize code
- **Standards**: Conventions you follow

**From Web Research** (optional):
- **Best practices**: For your tech stack
- **Known issues**: CVEs, common problems
- **Version info**: Outdated dependencies

**What You Provide** (2-3 questions):
- Compliance requirements? (None/SOC2/HIPAA/GDPR)
- Human review preference? (Minimal/Moderate/High)

Everything else is detected automatically.

---

## How Specialization Transforms Templates

### Before Specialization (Abstract Template)

```markdown
# Validate Implementation

Run these commands:

```bash
BUILD_CMD="{{PROJECT_BUILD_COMMAND}}"
TEST_CMD="{{PROJECT_TEST_COMMAND}}"
LINT_CMD="{{PROJECT_LINT_COMMAND}}"

$BUILD_CMD && $TEST_CMD && $LINT_CMD
```
```

### After Specialization (Your Project)

```markdown
# Validate Implementation

Run these commands:

```bash
BUILD_CMD="npm run build"
TEST_CMD="npm test"
LINT_CMD="npm run lint"

npm run build && npm test && npm run lint
```
```

**What changed**:
- `{{PROJECT_BUILD_COMMAND}}` → `npm run build` (from detection)
- Placeholders replaced with actual values from your project
- Commands now use YOUR tools, not generic placeholders

**This happens during** `/deploy-agents` - it reads all detected knowledge and replaces placeholders.

---

## File Structure

```
geist/                           (this repository)
├── profiles/default/            # Abstract templates
│   ├── commands/                # Command templates
│   │   ├── shape-spec/
│   │   ├── write-spec/
│   │   ├── create-tasks/
│   │   └── implement-tasks/
│   ├── workflows/               # Reusable workflow templates
│   │   ├── detection/           # Auto-detection
│   │   ├── basepoints/          # Knowledge extraction
│   │   └── validation/          # Validation utilities
│   ├── standards/               # Generic standards
│   └── docs/                    # Documentation
│
├── scripts/                     # Installation scripts
│   ├── project-install.sh       # Install templates
│   ├── project-update.sh        # Update installation
│   └── common-functions.sh      # Compilation utilities
│
└── config.yml                   # Default configuration

───────────────────────────────────────────────────────────────────

your-project/                    (after installation)
└── agent-os/                    # Installed templates
    ├── commands/                # (abstract, not specialized yet)
    ├── workflows/
    └── standards/

───────────────────────────────────────────────────────────────────

your-project/agent-os/           (after specialization)
├── basepoints/                  # Your codebase documentation
│   ├── headquarter.md           # Project overview
│   └── [layers]/[modules]/      # Per-module patterns
│
├── product/                     # Product documentation
│   ├── mission.md
│   ├── roadmap.md
│   └── tech-stack.md
│
├── config/                      # Project configuration
│   ├── project-profile.yml      # Detected profile
│   └── enriched-knowledge/      # Research results
│
├── commands/                    # Specialized commands (YOUR project)
├── workflows/                   # Specialized workflows
│
└── specs/                       # Your feature specs
    └── [feature-name]/
        ├── planning/
        ├── spec.md
        └── tasks.md
```

---

## Complete Command Chain: Visual Flow

Here's the complete chain showing exactly how files flow between commands:

```
┌─────────────────────────────────────────────────────────────────┐
│              COMPLETE COMMAND CHAIN (Visual Flow)               │
└─────────────────────────────────────────────────────────────────┘

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃  SETUP CHAIN (One-Time)                                        ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

  /adapt-to-product
  ┌─────────────┐
  │ Reads:      │  package.json, codebase
  │ Creates:    │  product/*.md
  │             │  config/project-profile.yml
  └──────┬──────┘
         │
         │ File dependency
         ├─────────────────┐
         │                 │
         ▼                 ▼
  ┌─────────────┐  ┌──────────────────┐
  │ product/    │  │ config/          │
  │ mission.md  │  │ project-profile  │
  │ roadmap.md  │  │ .yml             │
  │ tech-stack  │  └──────────────────┘
  │ .md         │
  └──────┬──────┘
         │
         │ Required by next command
         ▼
  /create-basepoints
  ┌─────────────┐
  │ Reads:      │  product/*.md (from above)
  │             │  config/*.yml (from above)
  │ Creates:    │  basepoints/**/*.md
  └──────┬──────┘
         │
         │ File dependency
         ▼
  ┌──────────────────────┐
  │ basepoints/          │
  │ headquarter.md       │
  │ [layers]/[modules]/  │
  │   agent-base-*.md    │
  └──────┬───────────────┘
         │
         │ Required by next command
         ▼
  /deploy-agents
  ┌─────────────┐
  │ Reads:      │  product/*.md
  │             │  basepoints/**/*.md
  │             │  config/*.yml
  │ Creates:    │  Specialized commands/
  └──────┬──────┘
         │
         ▼
  ✅ Ready to use specialized commands

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃  DEVELOPMENT CHAIN (Per Feature - Must Run in Order)           ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

  /shape-spec "Feature"
  ┌─────────────┐
  │ Reads:      │  basepoints/**/*.md (from setup)
  │ Creates:    │  specs/[name]/planning/requirements.md
  │             │  specs/[name]/implementation/cache/
  │             │    basepoints-knowledge.md ← Used by steps 2,3,4
  │             │    detected-layer.txt ← Used by steps 2,3,4
  └──────┬──────┘
         │
         │ ═══════════════════════════════════════════════════════
         │ Files required by next command
         ├─────────────────────────────┬──────────────────────────┐
         ▼                             ▼                          ▼
  ┌──────────────┐          ┌──────────────────┐       ┌──────────────────┐
  │ requirements │          │ basepoints-      │       │ detected-        │
  │ .md          │          │ knowledge.md     │       │ layer.txt        │
  └──────┬───────┘          └──────────────────┘       └──────────────────┘
         │                                                    │
         │ Required by write-spec                            │
         │ (knowledge files also used by create-tasks &      │
         │  implement-tasks)                                 │
         ▼                                                    │
  /write-spec                                                 │
  ┌─────────────┐                                            │
  │ Reads:      │  requirements.md ←─────────────────────────┘
  │             │  basepoints-knowledge.md ←──────────────────┐
  │             │  detected-layer.txt ←───────────────────────┘
  │ Creates:    │  specs/[name]/spec.md
  └──────┬──────┘
         │
         │ File dependency
         ▼
  ┌──────────────┐
  │ spec.md      │
  └──────┬───────┘
         │
         │ Required by next command
         ▼
  /create-tasks
  ┌─────────────┐
  │ Reads:      │  spec.md
  │             │  basepoints-knowledge.md (still cached)
  │ Creates:    │  specs/[name]/tasks.md
  └──────┬──────┘
         │
         │ File dependency
         ▼
  ┌──────────────┐
  │ tasks.md     │
  └──────┬───────┘
         │
         │ Required by next command
         ▼
  /implement-tasks  OR  /orchestrate-tasks
  ┌─────────────┐
  │ Reads:      │  tasks.md
  │             │  basepoints-knowledge.md (still cached)
  │             │  detected-layer.txt (still cached)
  │ Creates:    │  Code changes (you review)
  │             │  validation-report.md
  └─────────────┘

  🔗 CHAIN RULE: Each command creates files that the next command requires.
     Breaking the chain = missing files = command fails.
```

---

## How Templates Get Compiled

During installation, abstract templates are compiled into project-ready commands:

```
┌─────────────────────────────────────────────────────────────────┐
│              TEMPLATE COMPILATION PROCESS                       │
└─────────────────────────────────────────────────────────────────┘

  Source: profiles/default/commands/shape-spec/single-agent/
  │
  ├─ Step 1: Process conditionals
  │  {{IF use_claude_code_subagents}}
  │    ... content ...
  │  {{ENDIF}}
  │
  ├─ Step 2: Inject workflow references
  │  {{workflows/common/extract-basepoints-with-scope-detection}}
  │        ↓
  │  [Full workflow content inserted here]
  │
  ├─ Step 3: Inject standards references
  │  {{standards/global/*}}
  │        ↓
  │  @agent-os/standards/global/conventions.md
  │  @agent-os/standards/global/codebase-analysis.md
  │  ...
  │
  └─ Step 4: Embed phase files
     {{PHASE 1: @agent-os/commands/.../1-detect.md}}
           ↓
     # PHASE 1: Detect
     [Full phase content embedded]

  Output: your-project/agent-os/commands/shape-spec.md
          (Compiled, but still has {{PLACEHOLDERS}})
```

**Then during specialization** (`/deploy-agents`):

```
Compiled template
  │
  ├─ Replace: {{PROJECT_BUILD_COMMAND}} → npm run build
  ├─ Replace: {{BASEPOINTS_PATH}} → agent-os/basepoints
  ├─ Inject: Your patterns into command context
  │
  └─ Output: Fully specialized command (no placeholders)
```

---

## Key Features (Honestly Explained)

### Auto-Detection
**What it does**: Scans config files (package.json, Cargo.toml, etc.) to detect tech stack, build commands, architecture.  
**What it doesn't do**: It can't detect everything—unusual project structures might need manual correction.

### Basepoints
**What they are**: Documentation files that describe your codebase patterns, architecture, and decisions.  
**Why they matter**: Commands read these to inject your patterns into AI prompts.  
**Maintenance**: You update them when codebase changes significantly (run `/update-basepoints-and-redeploy`).

### Project-Agnostic Templates
**What it means**: Same commands work for web, mobile, CLI, embedded—any project type.  
**How**: Templates use placeholders and abstractions, not concrete technology assumptions.  
**Trade-off**: Sometimes more generic means less specific—you may need to adapt for very unusual projects.

### Validation System
**How it works**: After implementation, runs YOUR build/test/lint commands and checks exit codes.  
**What it validates**: Build succeeds, tests pass, linter passes.  
**What it doesn't validate**: Business logic correctness, edge cases, or things your tests don't cover.

---

## Installation Options

### Basic Installation

```bash
~/geist/scripts/project-install.sh --profile default
```

Installs templates only. Use with any AI tool (Cursor, Claude Desktop, etc.).

### With Claude Code Integration

```bash
~/geist/scripts/project-install.sh \
  --profile default \
  --claude-code-commands true \
  --use-claude-code-subagents true \
  --standards-as-claude-code-skills true
```

Creates `/` commands in Claude Code and integrates standards as skills.

### Update Existing Installation

```bash
~/geist/scripts/project-update.sh
```

Updates templates without losing your specialized knowledge.

---

## Limitations & Trade-offs

**This isn't magic**:
- You still review AI output—it's not perfect
- Commands can fail if your codebase is very unusual
- Basepoints need maintenance as your project evolves

**Detection isn't perfect**:
- Some tech stacks are harder to detect
- You may need to manually correct detection results
- Research can fail for internal/obscure libraries

**Requires structure**:
- Works best with organized codebases
- Benefits from clear module boundaries
- Struggles with very small (<100 lines) or very large (>100K lines) projects

**Validation depends on you**:
- Uses YOUR build/test/lint—if they're broken, validation is broken
- Only validates what you configure—won't catch everything
- Exit code validation is basic—doesn't verify quality

**Time investment**:
- Initial setup: ~20-30 minutes (automatic, but takes time)
- Per feature: ~5-10 minutes for command chain (plus your review time)
- Maintenance: Run `/update-basepoints-and-redeploy` when codebase changes significantly

---

## Quick Start

```bash
# 1. Clone Geist (one-time)
git clone <repo-url> ~/geist

# 2. Install in your project
cd /path/to/your/project
~/geist/scripts/project-install.sh --profile default

# 3. Specialize (one-time setup)
/adapt-to-product     # Detects everything, asks 2-3 questions
/create-basepoints    # Documents your codebase (~10-20 min)
/deploy-agents        # Specializes commands (~2-5 min)

# 4. Build a feature
/shape-spec "Add user authentication"
/write-spec
/create-tasks
/implement-tasks

# 5. Review code and iterate
```

---

## Differences from Agent OS

| Aspect | Agent OS | Geist |
|--------|----------|-------|
| **Project types** | Full-stack web only | Any project type |
| **Setup** | Manual questionnaire | Auto-detection + 2-3 questions |
| **Tech detection** | Manual | Automatic from config files |
| **Patterns** | Manual documentation | Auto-generated basepoints |
| **Validation** | Basic | Uses your actual build/test/lint |
| **Multi-agent** | — | Coordinated sub-agents supported |

**When to use Agent OS**: If you're building full-stack web apps and want the original implementation.  
**When to use Geist**: If you need flexibility for mobile, CLI, embedded, or want auto-detection.

---

## Documentation

- **[profiles/default/README.md](profiles/default/README.md)** - Detailed usage guide
- **[profiles/default/docs/COMMAND-FLOWS.md](profiles/default/docs/COMMAND-FLOWS.md)** - Command documentation
- **[profiles/default/docs/INSTALLATION-GUIDE.md](profiles/default/docs/INSTALLATION-GUIDE.md)** - Step-by-step setup
- **[MANIFEST.md](MANIFEST.md)** - Philosophy and motivation

---

## Credits

**Geist is built on [Agent OS](https://buildermethods.com/agent-os) by Brian Casel @ Builder Methods.**

The core concepts (spec-driven workflow, commands, workflows, knowledge extraction) come from Agent OS. Geist extends them to work for any project type and adds auto-detection, basepoints generation, and project-agnostic templates.

---

## License

MIT

---

**Last Updated**: 2026-01-18
