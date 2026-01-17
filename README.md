# Geist

### Agent OS, for any project.

---

## Why Use Geist?

**Geist gives your AI coding tools (Cursor, Claude Desktop, etc.) the "memory" they don't have.**

When you use AI to build features, it doesn't know your project's patterns, architecture, or conventions. You repeat yourself. It suggests generic solutions. Things break because it doesn't understand your codebase.

**Geist solves this by creating "basepoints"**—living documentation of your project's patterns, architecture, and decisions. Every AI command automatically includes relevant context, so suggestions match your actual codebase.

**How it works with Cursor and Claude:**
- Geist analyzes your codebase and documents your patterns
- You run commands like `/shape-spec` or `/implement-tasks` in your AI chat
- Geist automatically injects relevant project context into the prompts
- AI suggestions align with your existing code, architecture, and standards
- No more context-switching or explaining your project structure repeatedly

**The result:** AI that understands your project and suggests solutions that fit, not generic code you'll have to adapt.

---

## Built On Agent OS

**Geist is built on [Agent OS](https://buildermethods.com/agent-os) by Brian Casel @ Builder Methods.**

Agent OS introduced spec-driven agentic development—a structured way to work with AI that actually works. The core concepts (commands, workflows, standards, knowledge extraction) all come from Brian's work.

**Why Geist exists**: I tried Agent OS on a mobile project. The ideas were perfect, but the implementation assumed full-stack web development. Adapting it took more time than building. Geist makes Agent OS work for any project type—mobile, CLI, embedded, whatever you're building.

If you're building full-stack web apps, check out the [original Agent OS](https://buildermethods.com/agent-os) first. If you need something more flexible, that's what Geist is for.

---

## What Geist Does

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   ANY PROJECT          →    GEIST           →    SPECIALIZED            │
│                                                  AGENT OS               │
│   • Mobile (iOS/Android)     Detects your                               │
│   • Web (any framework)      tech stack         Commands that           │
│   • CLI tools                                   understand YOUR         │
│   • Backend services         Learns your        patterns, not           │
│   • Embedded systems         patterns           assumed patterns        │
│   • Anything else                                                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**Automatically detects** your project—tech stack, build commands, architecture  
**Documents your patterns** into basepoints (the "memory" AI doesn't have)  
**Specializes workflows** so they match how you actually build  
**Validates with your tools**—your build, your tests, your linter

---

## Geist vs Agent OS

| Feature | Agent OS | Geist |
|---------|----------|-------|
| **Project Type** | Full-stack web | Any project (mobile, CLI, embedded, web, etc.) |
| **Tech Detection** | Manual configuration | Auto-detects from config files |
| **Questioning** | Full questionnaire | Adaptive—only asks what it can't infer |
| **Codebase Knowledge** | Manual documentation | Auto-generates basepoints from code |
| **Context Enrichment** | — | Injects relevant basepoints into every command |
| **Web Research** | — | Researches best practices, CVEs, known issues |
| **Profile Inheritance** | — | Profiles can inherit and override |
| **Validation** | — | Shell-script validation with exit codes |
| **Multi-Agent Mode** | — | Coordinated sub-agents for complex tasks |
| **Layer Specialists** | — | Auto-generated specialists per abstraction layer |
| **Auto Agent Assignment** | — | Detects task layer, suggests appropriate specialist |
| **Layer Validation** | — | UI/API/Data pattern validation against basepoints |
| **Complexity Simplification** | — | Auto-adjust workflows based on project complexity |
| **Claude Code Skills** | — | Standards as auto-activated skills |

### What Geist Adds

```
┌────────────────────────────────────────────────────────────────┐
│                        GEIST LAYER                             │
├────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  Detection   │  │  Basepoints  │  │  Context Enrichment  │  │
│  │  System      │  │  Generation  │  │  on Every Command    │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  Adaptive    │  │  Web         │  │  Profile             │  │
│  │  Questions   │  │  Research    │  │  Inheritance         │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  Template    │  │  Conditional │  │  Shell               │  │
│  │  Compilation │  │  Compilation │  │  Validation          │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  Layer       │  │  Auto Agent  │  │  Multi-Agent         │  │
│  │  Specialists │  │  Assignment  │  │  Orchestration       │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
├────────────────────────────────────────────────────────────────┤
│                      AGENT OS CORE                             │
│         (Commands, Workflows, Standards, Specs)                │
└────────────────────────────────────────────────────────────────┘
```

### Layer Specialists

Geist automatically generates specialist agents for each detected abstraction layer:

```
┌─────────────────────────────────────────────────────────────────┐
│  LAYER SPECIALIST GENERATION                                    │
│                                                                 │
│  /create-basepoints → detects abstraction layers                │
│  /deploy-agents     → generates layer specialists               │
│                                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  Detected Layers           Generated Specialists           │ │
│  ├────────────────────────────────────────────────────────────┤ │
│  │  UI/Frontend         →    ui-specialist                    │ │
│  │  API/Backend         →    api-specialist                   │ │
│  │  Data/Persistence    →    data-specialist                  │ │
│  │  Platform/Infra      →    platform-specialist              │ │
│  │  Test/Quality        →    test-specialist                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│  Each specialist knows:                                         │
│  • Layer-specific patterns from basepoints                      │
│  • Relevant standards for that layer                            │
│  • How to stay within layer boundaries                          │
└─────────────────────────────────────────────────────────────────┘
```

**Auto-assignment during orchestration:**

```
/orchestrate-tasks
    │
    ├─ Analyzes task group content for layer keywords
    │   "Create user profile component" → ui-specialist
    │   "Add REST endpoint for users" → api-specialist
    │   "Create User model with migration" → data-specialist
    │
    ├─ Suggests specialists for each task group
    │
    └─ User confirms or overrides suggestions
```

---

## Quick Start

```bash
# 1. Clone Geist
git clone <repository-url> ~/geist

# 2. Install in your project
cd /path/to/your/project
~/geist/scripts/project-install.sh --profile default

# 3. Specialize for your project
/adapt-to-product     # Detects everything, asks 2-3 questions max
/create-basepoints    # Documents your codebase
/deploy-agents        # Configures for your specific project

# 4. Build
/shape-spec "Add payment processing"
# → AI now knows YOUR patterns
```

---

## How It Works

### The Problem It Solves

Every AI conversation starts from zero. The AI doesn't know your codebase, your patterns, your past decisions. You re-explain everything, every time.

### The Solution

```
              YOUR PROJECT
                   │
                   ▼
         ┌─────────────────┐
         │   DETECTION     │  ← Scans config files, analyzes structure
         └────────┬────────┘
                  │
                  ▼
         ┌─────────────────┐
         │   BASEPOINTS    │  ← Documents patterns, decisions, flows
         └────────┬────────┘
                  │
                  ▼
         ┌─────────────────┐
         │ SPECIALIZATION  │  ← Configures commands for YOUR project
         └────────┬────────┘
                  │
                  ▼
         ┌─────────────────┐
         │    AI AGENT     │  ← Now informed, not guessing
         └─────────────────┘
```

Basepoints are the "collective memory" your AI doesn't have. When you run commands like `/shape-spec` or `/implement-tasks`, Geist extracts relevant knowledge and gives it to the AI as context.

### Context Enrichment Flow

Every command automatically enriches AI context:

```
┌─────────────────────────────────────────────────────────────────┐
│  COMMAND EXECUTION                                              │
│                                                                 │
│  /shape-spec "Add payment processing"                           │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  CONTEXT ENRICHMENT                                     │    │
│  │                                                         │    │
│  │  1. Read @agent-os/basepoints/headquarter.md            │    │
│  │     → Project architecture, patterns, standards         │    │
│  │                                                         │    │
│  │  2. Detect relevant modules from feature scope          │    │
│  │     → @agent-os/basepoints/modules/payments/...         │    │
│  │                                                         │    │
│  │  3. Load applicable standards                           │    │
│  │     → @agent-os/standards/global/conventions.md         │    │
│  │     → @agent-os/standards/quality/assurance.md          │    │
│  │                                                         │    │
│  │  4. Inject into AI prompt                               │    │
│  └─────────────────────────────────────────────────────────┘    │
│       │                                                         │
│       ▼                                                         │
│  AI now operates with full project context                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## Shell Scripts

Geist uses shell scripts for installation, compilation, and updates. These scripts handle template processing, profile inheritance, and conditional compilation.

### Scripts Overview

| Script | Purpose |
|--------|---------|
| `project-install.sh` | Install Geist into a project |
| `project-update.sh` | Update existing installation |
| `create-profile.sh` | Create custom profiles |
| `common-functions.sh` | Shared utilities |

### Installation Flow

```
┌─────────────────────────────────────────────────────────────────┐
│  project-install.sh --profile default                           │
│                                                                 │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  1. DETECT GEIST REPOSITORY                             │    │
│  │     detect_base_dir() → finds profiles/default/         │    │
│  └─────────────────────────────────────────────────────────┘    │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  2. LOAD CONFIGURATION                                  │    │
│  │     config.yml → profile, flags, version                │    │
│  └─────────────────────────────────────────────────────────┘    │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  3. COMPILE TEMPLATES                                   │    │
│  │                                                         │    │
│  │     For each command/workflow/standard:                 │    │
│  │     ├─ process_conditionals()  → {{IF}}/{{UNLESS}}      │    │
│  │     ├─ process_workflows()     → {{workflows/...}}      │    │
│  │     ├─ process_standards()     → {{standards/...}}      │    │
│  │     └─ process_phase_tags()    → {{PHASE X: ...}}       │    │
│  └─────────────────────────────────────────────────────────┘    │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  4. INSTALL TO PROJECT                                  │    │
│  │                                                         │    │
│  │     your-project/                                       │    │
│  │     ├─ agent-os/                                        │    │
│  │     │  ├─ commands/      ← Compiled commands            │    │
│  │     │  ├─ workflows/     ← Workflow templates           │    │
│  │     │  ├─ standards/     ← Standards files              │    │
│  │     │  └─ config.yml     ← Installation config          │    │
│  │     └─ .claude/          ← Claude Code integration      │    │
│  │        ├─ commands/agent-os/                            │    │
│  │        ├─ agents/agent-os/                              │    │
│  │        └─ skills/                                       │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### Template Compilation System

Geist compiles abstract templates into project-specific implementations:

```
┌─────────────────────────────────────────────────────────────────┐
│  TEMPLATE COMPILATION PIPELINE                                  │
│                                                                 │
│  Source: profiles/default/commands/shape-spec/single-agent/     │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  1. CONDITIONAL COMPILATION                             │    │
│  │                                                         │    │
│  │     {{IF use_claude_code_subagents}}                    │    │
│  │       ... multi-agent content ...                       │    │
│  │     {{ENDIF use_claude_code_subagents}}                 │    │
│  │                                                         │    │
│  │     {{UNLESS compiled_single_command}}                  │    │
│  │       ... only in non-compiled mode ...                 │    │
│  │     {{ENDUNLESS compiled_single_command}}               │    │
│  └─────────────────────────────────────────────────────────┘    │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  2. WORKFLOW INJECTION                                  │    │
│  │                                                         │    │
│  │     {{workflows/detection/tech-stack-detection}}        │    │
│  │           ↓                                             │    │
│  │     [Full workflow content inserted]                    │    │
│  │                                                         │    │
│  │     Recursive: workflows can reference other workflows  │    │
│  └─────────────────────────────────────────────────────────┘    │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  3. STANDARDS INJECTION                                 │    │
│  │                                                         │    │
│  │     {{standards/global/*}}                              │    │
│  │           ↓                                             │    │
│  │     @agent-os/standards/global/conventions.md           │    │
│  │     @agent-os/standards/global/codebase-analysis.md     │    │
│  └─────────────────────────────────────────────────────────┘    │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  4. PHASE EMBEDDING (single-agent mode)                 │    │
│  │                                                         │    │
│  │     {{PHASE 1: @agent-os/commands/.../1-detect.md}}     │    │
│  │           ↓                                             │    │
│  │     # PHASE 1: Detect                                   │    │
│  │     [Full phase content embedded]                       │    │
│  └─────────────────────────────────────────────────────────┘    │
│       │                                                         │
│       ▼                                                         │
│  Output: your-project/agent-os/commands/shape-spec.md           │
│          (Fully compiled, project-ready)                        │
└─────────────────────────────────────────────────────────────────┘
```

### Profile Inheritance

Profiles can inherit from other profiles, allowing customization:

```
┌─────────────────────────────────────────────────────────────────┐
│  PROFILE INHERITANCE                                            │
│                                                                 │
│  profiles/                                                      │
│  ├─ default/              ← Base profile (all templates)        │
│  │  ├─ commands/                                                │
│  │  ├─ workflows/                                               │
│  │  └─ standards/                                               │
│  │                                                              │
│  └─ rails/                ← Custom profile                      │
│     ├─ profile-config.yml                                       │
│     │    inherits_from: default                                 │
│     │    exclude_inherited_files:                               │
│     │      - standards/frontend/*                               │
│     │                                                           │
│     └─ standards/         ← Overrides default standards         │
│        └─ backend/                                              │
│           └─ api.md       ← Rails-specific API standards        │
│                                                                 │
│  Resolution: get_profile_file()                                 │
│  1. Check custom profile first                                  │
│  2. Fall back to inherited profile                              │
│  3. Apply exclusion patterns                                    │
└─────────────────────────────────────────────────────────────────┘
```

### Key Functions

| Function | File | Purpose |
|----------|------|---------|
| `detect_base_dir()` | common-functions.sh | Finds Geist repository |
| `get_profile_file()` | common-functions.sh | Resolves file with inheritance |
| `get_profile_files()` | common-functions.sh | Lists all files for profile |
| `process_conditionals()` | common-functions.sh | Handles {{IF}}/{{UNLESS}} |
| `process_workflows()` | common-functions.sh | Injects workflow content |
| `process_standards()` | common-functions.sh | Resolves standards references |
| `process_phase_tags()` | common-functions.sh | Embeds phase files |
| `compile_command()` | common-functions.sh | Full compilation pipeline |
| `compile_agent()` | common-functions.sh | Compiles agent definitions |

### Installation Options

```bash
# Basic installation
~/geist/scripts/project-install.sh --profile default

# With Claude Code integration
~/geist/scripts/project-install.sh \
  --profile default \
  --claude-code-commands true \
  --use-claude-code-subagents true \
  --standards-as-claude-code-skills true

# Agent-os commands only (for other AI tools)
~/geist/scripts/project-install.sh \
  --profile default \
  --agent-os-commands true \
  --claude-code-commands false

# Dry run (preview changes)
~/geist/scripts/project-install.sh --profile default --dry-run

# Re-install (clean slate)
~/geist/scripts/project-install.sh --profile default --re-install
```

### Update Options

```bash
# Update to latest
~/geist/scripts/project-update.sh

# Overwrite specific components
~/geist/scripts/project-update.sh --overwrite-commands
~/geist/scripts/project-update.sh --overwrite-standards
~/geist/scripts/project-update.sh --overwrite-agents
~/geist/scripts/project-update.sh --overwrite-all
```

---

## Commands

### Setup (run once)

| Command | What it does |
|---------|--------------|
| `/adapt-to-product` | Detects your project, creates product docs |
| `/create-basepoints` | Documents your codebase patterns |
| `/deploy-agents` | Specializes everything for your project |

### Development (ongoing)

| Command | What it does |
|---------|--------------|
| `/shape-spec` | Research and define a feature |
| `/write-spec` | Write detailed specification |
| `/create-tasks` | Break spec into tasks |
| `/implement-tasks` | Implement the tasks |
| `/orchestrate-tasks` | Multi-agent coordination |

### Maintenance

| Command | What it does |
|---------|--------------|
| `/update-basepoints-and-redeploy` | Sync after codebase changes |
| `/cleanup-agent-os` | Verify and clean installation |

### Command Flow

```
┌─────────────────────────────────────────────────────────────────┐
│  FEATURE DEVELOPMENT CYCLE                                      │
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐       │
│  │ /shape-spec  │ →  │ /write-spec  │ →  │/create-tasks │       │
│  │              │    │              │    │              │       │
│  │ Research &   │    │ Detailed     │    │ Break into   │       │
│  │ requirements │    │ spec doc     │    │ tasks.md     │       │
│  └──────────────┘    └──────────────┘    └──────────────┘       │
│                                                 │                │
│                                                 ▼                │
│                                          ┌──────────────┐       │
│                                          │/implement or │       │
│                                          │/orchestrate  │       │
│                                          │              │       │
│                                          │ Execute      │       │
│                                          │ tasks        │       │
│                                          └──────────────┘       │
│                                                                 │
│  Each command:                                                  │
│  • Loads relevant basepoints as context                         │
│  • Applies applicable standards                                 │
│  • Validates output against project conventions                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Features

### Smart Detection

Geist scans your project and figures out:
- Tech stack (from package.json, Cargo.toml, go.mod, etc.)
- Build/test/lint commands
- Project architecture
- Security indicators

It only asks questions it can't answer from your code.

### Basepoints

Documentation that AI can actually use:
- **Patterns**: How you do things
- **Standards**: Your conventions
- **Flows**: How data moves
- **Decisions**: Why you made choices

### Project-Agnostic

Works with any project type, any language, any framework. The same commands work whether you're building iOS apps or Kubernetes operators.

### Validation System

```
┌─────────────────────────────────────────────────────────────────┐
│  VALIDATION FLOW                                                │
│                                                                 │
│  Implementation                                                 │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  1. Run YOUR validation commands:                       │    │
│  │                                                         │    │
│  │  $ npm run build        → exit 0 ✓                      │    │
│  │  $ npm run test         → exit 0 ✓                      │    │
│  │  $ npm run lint         → exit 0 ✓                      │    │
│  │                                                         │    │
│  │  Commands detected during /adapt-to-product             │    │
│  └─────────────────────────────────────────────────────────┘    │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  2. Run LAYER validations (complex projects):           │    │
│  │                                                         │    │
│  │  validate-ui-patterns.md   → Component conventions ✓    │    │
│  │  validate-api-patterns.md  → Endpoint conventions ✓     │    │
│  │  validate-data-patterns.md → Data model patterns ✓      │    │
│  │                                                         │    │
│  │  Validates against patterns in basepoints               │    │
│  └─────────────────────────────────────────────────────────┘    │
│       │                                                         │
│       ▼                                                         │
│  Pass all → Implementation complete                             │
│  Fail any → AI fixes issues automatically                       │
└─────────────────────────────────────────────────────────────────┘
```

### Complexity-Based Simplification

Geist automatically adjusts based on project complexity:

```
┌─────────────────────────────────────────────────────────────────┐
│  PROJECT COMPLEXITY → WORKFLOW ADJUSTMENT                       │
│                                                                 │
│  SIMPLE PROJECT                                                 │
│  ├─ Active: specification, implementation, basepoints           │
│  ├─ Skipped: validation, deep-reading, research, human-review   │
│  └─ Use /implement-tasks directly (skip orchestration)          │
│                                                                 │
│  MODERATE PROJECT                                               │
│  ├─ Active: + planning, detection, research                     │
│  ├─ Skipped: deep-reading (partial)                             │
│  └─ Optional: layer validations                                 │
│                                                                 │
│  COMPLEX PROJECT                                                │
│  ├─ Active: ALL workflows                                       │
│  ├─ Enabled: layer validations, human review checkpoints        │
│  └─ Full: research iterations, comprehensive validation         │
└─────────────────────────────────────────────────────────────────┘
```

---

## Project Structure

```
geist/
├── profiles/default/          # Project-agnostic templates
│   ├── commands/              # Abstract commands
│   │   ├── shape-spec/
│   │   │   ├── single-agent/  # All phases in one prompt
│   │   │   └── multi-agent/   # Phases delegated to sub-agents
│   │   └── ...
│   ├── workflows/             # Reusable workflow templates
│   │   ├── detection/         # Tech stack detection
│   │   ├── research/          # Web research workflows
│   │   ├── validation/        # Validation workflows
│   │   ├── basepoints/        # Knowledge extraction
│   │   └── human-review/      # Human checkpoint flows
│   ├── standards/             # Global standards
│   └── agents/                # Agent definitions
├── scripts/                   # Installation scripts
│   ├── project-install.sh     # Install to project
│   ├── project-update.sh      # Update installation
│   ├── create-profile.sh      # Create custom profile
│   └── common-functions.sh    # Shared utilities
└── config.yml                 # Default configuration

your-project/agent-os/         # After installation
├── basepoints/                # Your codebase knowledge
│   ├── headquarter.md         # Architecture overview
│   └── modules/               # Per-module documentation
├── product/                   # Your product docs
│   ├── mission.md
│   ├── roadmap.md
│   └── tech-stack.md
├── commands/                  # Specialized for YOU
├── workflows/                 # Workflow templates
├── standards/                 # Standards files
├── specs/                     # Feature specifications
└── config.yml                 # Installation config
```

---

## Documentation

| Doc | What it covers |
|-----|----------------|
| [MANIFEST.md](MANIFEST.md) | Why Geist exists, the philosophy |
| [profiles/default/README.md](profiles/default/README.md) | Comprehensive usage guide |
| [profiles/default/docs/COMMAND-FLOWS.md](profiles/default/docs/COMMAND-FLOWS.md) | Detailed command documentation |
| [profiles/default/docs/INSTALLATION-GUIDE.md](profiles/default/docs/INSTALLATION-GUIDE.md) | Step-by-step setup |

---

## Credits

**Geist is built on [Agent OS](https://buildermethods.com/agent-os) by Brian Casel @ Builder Methods.**

The spec-driven workflow, the command structure, the concept of extracting codebase knowledge—these ideas come from Brian's work. Geist extends them to work for any project type.

---

## License

MIT

---

*Agent OS made universal.*

**January 2026**
