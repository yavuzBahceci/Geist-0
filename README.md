# Geist

<p align="center">
  <strong>A Mind for Agentic Development</strong>
</p>

<p align="center">
  <em>Give your AI assistant persistent memory, structured workflows, and deep codebase understanding.</em>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License: MIT"></a>
  <a href="#quick-start"><img src="https://img.shields.io/badge/setup-~20min-green.svg" alt="Setup: ~20min"></a>
  <a href="profiles/default/docs/COMMAND-FLOWS.md"><img src="https://img.shields.io/badge/commands-13-orange.svg" alt="Commands: 13"></a>
  <a href="CONTRIBUTING.md"><img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" alt="PRs Welcome"></a>
</p>

---

## The Problem with AI Coding Today

Modern AI coding assistants are powerful, but they're **stateless**. Every conversation starts from zero. You explain your patterns, conventions, and architecture—again and again.

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

---

## What is Geist?

Geist is a **cognitive architecture for AI coding assistants**. It gives your AI:

- **Persistent Memory**: Basepoints document your codebase patterns, architecture, and decisions
- **Structured Workflows**: Commands chain together, each building on the previous
- **Deep Understanding**: Auto-detection learns your tech stack, conventions, and standards
- **Accumulated Knowledge**: Context flows between commands—nothing is lost

Think of it as giving your AI assistant a **mind** instead of just a **prompt**.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         THE GEIST ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                         KNOWLEDGE LAYER                             │   │
│   │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │   │
│   │  │  Basepoints  │  │   Product    │  │   Library    │              │   │
│   │  │  (codebase)  │  │  (mission)   │  │  (tech stack)│              │   │
│   │  └──────────────┘  └──────────────┘  └──────────────┘              │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                    ↓                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                         WORKFLOW LAYER                              │   │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐           │   │
│   │  │ shape-   │→ │ write-   │→ │ create-  │→ │implement-│           │   │
│   │  │ spec     │  │ spec     │  │ tasks    │  │ tasks    │           │   │
│   │  └──────────┘  └──────────┘  └──────────┘  └──────────┘           │   │
│   │       ↓              ↓             ↓             ↓                 │   │
│   │  requirements    spec.md      tasks.md      code changes           │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                    ↓                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                        VALIDATION LAYER                             │   │
│   │  Uses YOUR build/test/lint commands to verify every change         │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Why Not Just Use Rules Files?

| Approach | Limitation | Geist Solution |
|----------|------------|----------------|
| **Rules files** | Static, generic, you write them manually | Basepoints are auto-generated from your actual codebase |
| **Context files** | Limited by token window, lost between sessions | Knowledge accumulates across commands, persists in files |
| **Custom prompts** | Inconsistent results, no structure | Structured workflows with validation at each step |
| **Memory features** | Shallow, conversation-scoped | Deep, project-scoped, interconnected knowledge |

**The key difference**: Standard tools give AI *instructions*. Geist gives AI *understanding*.

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

**Key idea**: Templates are project-agnostic. Specialization makes them project-specific. You get commands that understand YOUR codebase, not generic ones.

---

## How It Works

### Step 1: Install (One-Time)

```bash
# Clone Geist
git clone https://github.com/yavuzBahceci/geist.git ~/geist

# Install into your project
cd /path/to/your/project
~/geist/scripts/project-install.sh --profile default
```

### Step 2: Specialize (One-Time Setup, ~20 min)

```
/adapt-to-product     # Auto-detects tech stack, asks 2-3 questions
       ↓
/create-basepoints    # Documents your codebase patterns
       ↓
/deploy-agents        # Specializes commands for YOUR project
       ↓
/cleanup-geist        # Validates everything is ready
```

### Step 3: Build Features (Per Feature)

```
/shape-spec "Add user authentication"
       ↓
/write-spec           # Detailed specification
       ↓
/create-tasks         # Actionable task breakdown
       ↓
/implement-tasks      # Code with validation
```

**Each command reads context from the previous and adds to it.** Nothing is lost.

---

## What Gets Auto-Detected

When you run `/adapt-to-product`, Geist automatically detects:

| Category | Detection Method |
|----------|-----------------|
| **Language** | package.json, Cargo.toml, go.mod, requirements.txt |
| **Framework** | Dependencies analysis |
| **Build command** | package.json scripts, Makefile, CI configs |
| **Test command** | Test configs, CI pipelines |
| **Lint command** | .eslintrc, Makefile, package.json |
| **Architecture** | Directory structure analysis |

**You only answer 2-3 questions**: Compliance requirements? Human review preference?

Everything else is detected automatically.

---

## Knowledge System

### Basepoints: Your Codebase's Memory

Basepoints are living documentation of your codebase:

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

Context flows between commands:

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

**Nothing is lost between commands.**

---

## Comparison: Before and After

### Without Geist

```
You: "Add authentication. We use React with TypeScript, 
     Zustand for state, our auth patterns are in 
     src/features/auth, we follow this naming convention,
     use this folder structure, our API is REST with
     these endpoints..."

AI: *generates generic code that doesn't match your patterns*

You: "No, we do it this way..." *explains again*
```

### With Geist

```
You: /shape-spec "Add authentication"

AI: *already knows*
    - Your tech stack (React, TypeScript, Zustand)
    - Your patterns (from basepoints)
    - Your conventions (from standards)
    - Your architecture (from detection)
    
    "Based on your existing auth patterns in src/features/auth,
     I'll follow your established Zustand store pattern..."
```

---

## Commands Overview

| Category | Commands | Purpose |
|----------|----------|---------|
| **Setup** | `adapt-to-product`, `plan-product`, `create-basepoints`, `deploy-agents` | One-time project specialization |
| **Development** | `shape-spec`, `write-spec`, `create-tasks`, `implement-tasks`, `orchestrate-tasks` | Feature development workflow |
| **Maintenance** | `cleanup-geist`, `update-basepoints-and-redeploy` | Keep knowledge in sync |
| **Utility** | `fix-bug` | Systematic bug analysis and fixing |

---

## Quick Start

```bash
# 1. Clone Geist
git clone https://github.com/yavuzBahceci/geist.git ~/geist

# 2. Install in your project
cd /path/to/your/project
~/geist/scripts/project-install.sh --profile default

# 3. Specialize (one-time, ~20 min)
/adapt-to-product
/create-basepoints
/deploy-agents
/cleanup-geist

# 4. Build a feature
/shape-spec "Add user authentication"
/write-spec
/create-tasks
/implement-tasks

# 5. Review and iterate
```

---

## File Structure

```
your-project/geist/              (after specialization)
├── basepoints/                  # Your codebase documentation
│   ├── headquarter.md           # Project overview
│   ├── [layers]/[modules]/      # Per-module patterns
│   └── libraries/               # Tech stack documentation
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
├── commands/                    # Specialized commands
├── workflows/                   # Specialized workflows
│
└── specs/                       # Your feature specs
    └── [feature-name]/
        ├── planning/
        ├── spec.md
        └── tasks.md
```

---

## Limitations (Honestly)

**This isn't magic**:
- You still review AI output—it's not perfect
- Commands can fail if your codebase is very unusual
- Basepoints need maintenance as your project evolves

**Detection isn't perfect**:
- Some tech stacks are harder to detect
- You may need to manually correct detection results

**Requires structure**:
- Works best with organized codebases
- Benefits from clear module boundaries

**Time investment**:
- Initial setup: ~20-30 minutes
- Per feature: ~5-10 minutes for command chain
- Maintenance: Run `/update-basepoints-and-redeploy` when codebase changes

---

## Documentation

- **[profiles/default/README.md](profiles/default/README.md)** - Detailed usage guide
- **[profiles/default/docs/COMMAND-FLOWS.md](profiles/default/docs/COMMAND-FLOWS.md)** - Command documentation
- **[profiles/default/docs/INSTALLATION-GUIDE.md](profiles/default/docs/INSTALLATION-GUIDE.md)** - Step-by-step setup
- **[profiles/default/docs/TROUBLESHOOTING.md](profiles/default/docs/TROUBLESHOOTING.md)** - Common issues
- **[MANIFEST.md](MANIFEST.md)** - Philosophy and motivation
- **[CONTRIBUTING.md](CONTRIBUTING.md)** - How to contribute

---

## Credits

Geist builds on the foundational concepts from [Agent OS](https://buildermethods.com/agent-os) by Brian Casel @ Builder Methods—the spec-driven workflow, command structure, and knowledge extraction patterns. Geist extends these ideas into a complete cognitive architecture for agentic development, adding auto-detection, basepoints generation, knowledge accumulation, and support for any project type.

---

## License

MIT

---

**Last Updated**: 2026-01-18
