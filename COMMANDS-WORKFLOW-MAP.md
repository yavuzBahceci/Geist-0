# Workflow Map: Complete Command & Workflow Reference

This document provides a comprehensive visual map of all commands and their workflows, showing what each workflow does, where it's called, and what it produces.

---

## Table of Contents

- [Command Overview](#command-overview)
- [Workflow Categories](#workflow-categories)
- [Command Flow Diagrams](#command-flow-diagrams)
  - [Specialization Commands](#1-specialization-commands)
  - [Development Commands](#2-development-commands)
  - [Maintenance Commands](#3-maintenance-commands)
- [Workflow Reference](#workflow-reference)
- [Knowledge Flow](#knowledge-flow)
- [Output Locations](#output-locations)

---

## Command Overview

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              GEIST COMMAND ECOSYSTEM                                 │
└─────────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────┐
│  SPECIALIZATION COMMANDS (Run Once)                                                  │
│  ═══════════════════════════════════                                                │
│                                                                                      │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐               │
│  │  adapt-to-product │───▶│ create-basepoints │───▶│  deploy-agents   │               │
│  │  OR plan-product  │    │                  │    │                  │               │
│  └──────────────────┘    └──────────────────┘    └──────────────────┘               │
│         │                        │                        │                          │
│         ▼                        ▼                        ▼                          │
│  geist/product/        geist/basepoints/     geist/commands/               │
│  • mission.md             • headquarter.md         geist/workflows/               │
│  • roadmap.md             • agent-base-*.md        geist/standards/               │
│  • tech-stack.md          • libraries/                                               │
└─────────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────┐
│  DEVELOPMENT COMMANDS (Run Per Feature)                                              │
│  ══════════════════════════════════════                                             │
│                                                                                      │
│  ┌────────────┐   ┌────────────┐   ┌─────────────┐   ┌─────────────────────┐        │
│  │ shape-spec │──▶│ write-spec │──▶│ create-tasks│──▶│ orchestrate-tasks   │        │
│  └────────────┘   └────────────┘   └─────────────┘   │ OR implement-tasks  │        │
│       │                │                 │           └─────────────────────┘        │
│       ▼                ▼                 ▼                     │                     │
│  requirements.md    spec.md          tasks.md                  ▼                     │
│  + accumulated     + accumulated    + accumulated        Code Changes               │
│    knowledge         knowledge        knowledge                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────┐
│  MAINTENANCE & UTILITY COMMANDS                                                      │
│  ══════════════════════════════                                                     │
│                                                                                      │
│  ┌─────────────────────────────┐   ┌─────────────────┐   ┌─────────────────┐        │
│  │ update-basepoints-and-     │   │  cleanup-agent- │   │    fix-bug      │        │
│  │ redeploy                    │   │  os             │   │                 │        │
│  └─────────────────────────────┘   └─────────────────┘   └─────────────────┘        │
│              │                              │                     │                  │
│              ▼                              ▼                     ▼                  │
│  Updated basepoints              Cleaned geist            Bug fixes with            │
│  Re-specialized commands         Verified knowledge       knowledge synthesis       │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Workflow Categories

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              WORKFLOW CATEGORIES                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘

workflows/
├── basepoints/                 # Basepoints extraction and organization
│   ├── extract-basepoints-knowledge-automatic.md
│   ├── extract-basepoints-knowledge-on-demand.md
│   └── organize-and-cache-basepoints-knowledge.md
│
├── codebase-analysis/          # Codebase analysis and basepoint generation
│   ├── analyze-codebase.md
│   ├── detect-abstraction-layers.md
│   ├── generate-headquarter.md
│   ├── generate-module-basepoints.md
│   ├── generate-parent-basepoints.md
│   ├── generate-library-basepoints.md    ◀── NEW
│   └── ...
│
├── common/                     # Shared workflows across commands
│   ├── extract-basepoints-with-scope-detection.md
│   ├── extract-library-basepoints-knowledge.md   ◀── NEW
│   ├── accumulate-knowledge.md                   ◀── NEW
│   ├── validate-prerequisites.md
│   └── ...
│
├── detection/                  # Project detection workflows
│   ├── detect-project-profile.md
│   ├── detect-tech-stack.md
│   ├── detect-architecture.md
│   └── ...
│
├── human-review/               # Human decision workflows
│   ├── review-trade-offs.md
│   ├── create-checkpoint.md
│   └── ...
│
├── implementation/             # Implementation workflows
│   ├── implement-tasks.md
│   ├── create-tasks-list.md
│   ├── compile-implementation-standards.md
│   └── verification/
│       ├── verify-tasks.md
│       ├── run-all-tests.md
│       └── create-verification-report.md
│
├── learning/                   # Session learning workflows
│   ├── capture-session-feedback.md
│   ├── extract-session-patterns.md
│   └── ...
│
├── prompting/                  # Prompt optimization workflows
│   ├── save-handoff.md
│   ├── construct-prompt.md
│   └── ...
│
├── research/                   # Web research workflows
│   ├── research-orchestrator.md
│   ├── research-library.md
│   ├── research-library-documentation.md   ◀── NEW
│   └── ...
│
├── scope-detection/            # Scope and layer detection
│   ├── detect-abstraction-layer.md
│   ├── detect-scope-semantic-analysis.md
│   └── ...
│
├── specification/              # Spec writing workflows
│   ├── research-spec.md
│   ├── write-spec.md
│   └── ...
│
├── validation/                 # Validation workflows
│   ├── validate-implementation.md
│   ├── validate-output-exists.md
│   └── ...
│
└── cleanup/                    # Cleanup workflows
    └── product-focused-cleanup.md
```

---

## Command Flow Diagrams

### 1. Specialization Commands

#### adapt-to-product / plan-product

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                        ADAPT-TO-PRODUCT / PLAN-PRODUCT                               │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 1: Setup & Detection
══════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                                                                      │
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/detection/          │                                                │
│  │   detect-project-profile}}      │                                                │
│  └───────────────┬─────────────────┘                                                │
│                  │                                                                   │
│    ┌─────────────┼─────────────┬─────────────┬─────────────┐                        │
│    ▼             ▼             ▼             ▼             ▼                        │
│ detect-      detect-       detect-      detect-       detect-                       │
│ tech-stack   commands      architecture security      project-                      │
│    │             │             │             │         profile                       │
│    └─────────────┴─────────────┴─────────────┴─────────────┘                        │
│                                │                                                     │
│                                ▼                                                     │
│                  ┌─────────────────────────────┐                                    │
│                  │ geist/config/            │                                    │
│                  │ project-profile.yml         │                                    │
│                  └─────────────────────────────┘                                    │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 2: Web Research
═════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                                                                      │
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/research/           │                                                │
│  │   research-orchestrator}}       │                                                │
│  └───────────────┬─────────────────┘                                                │
│                  │                                                                   │
│    ┌─────────────┼─────────────┬─────────────┬─────────────┐                        │
│    ▼             ▼             ▼             ▼             ▼                        │
│ research-    research-     research-    research-    version-                       │
│ library      stack-        security     domain       analysis                       │
│              patterns                                                               │
│    │             │             │             │             │                        │
│    └─────────────┴─────────────┴─────────────┴─────────────┘                        │
│                                │                                                     │
│                                ▼                                                     │
│                  ┌─────────────────────────────┐                                    │
│                  │ geist/config/            │                                    │
│                  │ enriched-knowledge/         │                                    │
│                  │ • library-research.md       │                                    │
│                  │ • stack-best-practices.md   │                                    │
│                  │ • security-notes.md         │                                    │
│                  └─────────────────────────────┘                                    │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 3-5: Create Product Docs
══════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                                                                      │
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/planning/           │                                                │
│  │   gather-product-info}}         │                                                │
│  └───────────────┬─────────────────┘                                                │
│                  │                                                                   │
│    ┌─────────────┼─────────────┬─────────────┐                                      │
│    ▼             ▼             ▼             ▼                                      │
│ create-      create-       create-                                                  │
│ mission      roadmap       tech-stack                                               │
│    │             │             │                                                     │
│    └─────────────┴─────────────┴─────────────┘                                      │
│                                │                                                     │
│                                ▼                                                     │
│                  ┌─────────────────────────────┐                                    │
│                  │ geist/product/           │                                    │
│                  │ • mission.md                │                                    │
│                  │ • roadmap.md                │                                    │
│                  │ • tech-stack.md             │                                    │
│                  └─────────────────────────────┘                                    │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 7: Product-Focused Cleanup
════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                                                                      │
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/cleanup/            │                                                │
│  │   product-focused-cleanup}}     │                                                │
│  └───────────────┬─────────────────┘                                                │
│                  │                                                                   │
│    ┌─────────────┴─────────────┐                                                    │
│    ▼                           ▼                                                    │
│ Phase A:                   Phase B:                                                 │
│ SIMPLIFY                   EXPAND                                                   │
│ Remove irrelevant          Add tech-specific                                        │
│ tech examples              patterns                                                 │
│    │                           │                                                     │
│    └───────────┬───────────────┘                                                    │
│                ▼                                                                     │
│  ┌─────────────────────────────┐                                                    │
│  │ geist/output/            │                                                    │
│  │ product-cleanup/            │                                                    │
│  │ • detected-scope.yml        │                                                    │
│  │ • cleanup-report.md         │                                                    │
│  └─────────────────────────────┘                                                    │
└─────────────────────────────────────────────────────────────────────────────────────┘

OUTPUT: Ready for create-basepoints
═══════════════════════════════════
```

#### create-basepoints

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              CREATE-BASEPOINTS                                       │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 1: Validate Prerequisites
═══════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/codebase-analysis/  │     Checks:                                    │
│  │   validate-prerequisites}}      │───▶ • product files exist                      │
│  └─────────────────────────────────┘     • project-profile.yml exists               │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 2: Detect Abstraction Layers
══════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/codebase-analysis/  │     Outputs:                                   │
│  │   detect-abstraction-layers}}   │───▶ • Layer hierarchy                          │
│  └─────────────────────────────────┘     • Module boundaries                        │
│                                                                                      │
│  Example layers detected:                                                            │
│  ┌─────────────────────────────────────────────────────────────────────────────┐    │
│  │  ROOT ──▶ PROFILES ──▶ COMMANDS ──▶ WORKFLOWS ──▶ STANDARDS                 │    │
│  └─────────────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 3: Mirror Project Structure
═════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/codebase-analysis/  │     Creates:                                   │
│  │   mirror-project-structure}}    │───▶ geist/basepoints/                       │
│  └─────────────────────────────────┘     └── [mirrors project structure]            │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 4: Analyze Codebase
═════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/codebase-analysis/  │     Extracts:                                  │
│  │   analyze-codebase}}            │───▶ • Patterns                                 │
│  └─────────────────────────────────┘     • Standards                                │
│                                          • Flows                                     │
│                                          • Strategies                                │
│                                          • Testing approaches                        │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 5: Generate Module Basepoints
═══════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/codebase-analysis/  │     Creates:                                   │
│  │   generate-module-basepoints}}  │───▶ geist/basepoints/                       │
│  └─────────────────────────────────┘     └── [layer]/[module]/                      │
│                                              └── agent-base-[module].md             │
│                                                                                      │
│  Each basepoint contains:                                                            │
│  ┌─────────────────────────────────────────────────────────────────────────────┐    │
│  │  ## Patterns        ## Standards       ## Flows                             │    │
│  │  ## Strategies      ## Testing         ## Dependencies                      │    │
│  └─────────────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 6: Generate Parent Basepoints
═══════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/codebase-analysis/  │     Creates:                                   │
│  │   generate-parent-basepoints}}  │───▶ Aggregated parent basepoints               │
│  └─────────────────────────────────┘     (combines child basepoints)                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 7: Generate Headquarter
═════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/codebase-analysis/  │     Creates:                                   │
│  │   generate-headquarter}}        │───▶ geist/basepoints/headquarter.md         │
│  └─────────────────────────────────┘                                                │
│                                          Contains:                                   │
│                                          • Project overview                          │
│                                          • Architecture summary                      │
│                                          • Basepoint index                           │
│                                          • Key patterns                              │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 8: Generate Library Basepoints ◀── NEW
════════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/codebase-analysis/  │                                                │
│  │   generate-library-basepoints}} │                                                │
│  └───────────────┬─────────────────┘                                                │
│                  │                                                                   │
│    ┌─────────────┼─────────────┬─────────────┬─────────────┐                        │
│    ▼             ▼             ▼             ▼             ▼                        │
│ Load          Extract       Analyze      Research      Generate                     │
│ tech-stack    usage from    codebase     official      basepoints                   │
│               basepoints    imports      docs                                       │
│    │             │             │             │             │                        │
│    └─────────────┴─────────────┴─────────────┴─────────────┘                        │
│                                │                                                     │
│                                ▼                                                     │
│                  ┌─────────────────────────────┐                                    │
│                  │ geist/basepoints/        │                                    │
│                  │ libraries/                  │                                    │
│                  │ ├── data/                   │                                    │
│                  │ ├── domain/                 │                                    │
│                  │ ├── util/                   │                                    │
│                  │ ├── infrastructure/         │                                    │
│                  │ ├── framework/              │                                    │
│                  │ └── README.md               │                                    │
│                  └─────────────────────────────┘                                    │
│                                                                                      │
│  Each library basepoint contains:                                                    │
│  ┌─────────────────────────────────────────────────────────────────────────────┐    │
│  │  ## How WE Use It      ## Boundaries (what is/isn't used)                   │    │
│  │  ## Patterns           ## Workflows                                         │    │
│  │  ## Best Practices     ## Troubleshooting                                   │    │
│  └─────────────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────────────┘

OUTPUT: Ready for deploy-agents
═══════════════════════════════
```

#### deploy-agents

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                DEPLOY-AGENTS                                         │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 1: Validate Prerequisites
═══════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐     Loads:                                     │
│  │ Load project-profile.yml        │───▶ • Tech stack                               │
│  │ Load enriched-knowledge/        │     • Commands                                 │
│  └─────────────────────────────────┘     • Security level                           │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 2: Extract Basepoints Knowledge
═════════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/basepoints/         │     Extracts:                                  │
│  │   extract-basepoints-knowledge- │───▶ • Patterns from all basepoints             │
│  │   automatic}}                   │     • Standards                                │
│  └─────────────────────────────────┘     • Flows                                    │
│                                                                                      │
│  Output: geist/output/deploy-agents/knowledge/basepoints-knowledge.json          │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 3-4: Extract & Merge Knowledge
════════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ Extract product knowledge       │     Combines:                                  │
│  │ Merge all knowledge sources     │───▶ • Basepoints knowledge                     │
│  └─────────────────────────────────┘     • Product knowledge                        │
│                                          • Enriched knowledge                        │
│                                                                                      │
│  Output: geist/output/deploy-agents/knowledge/merged-knowledge.json              │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 5-7: Specialize Commands
══════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  Specializes:                                                                        │
│  • shape-spec                                                                        │
│  • write-spec                                                                        │
│  • create-tasks                                                                      │
│  • implement-tasks                                                                   │
│  • orchestrate-tasks                                                                 │
│                                                                                      │
│  Actions:                                                                            │
│  • Replace {{PLACEHOLDER}} with actual values                                        │
│  • Inject project-specific patterns                                                  │
│  • Configure validation commands                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 8: Specialize Standards ◀── ENHANCED
══════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ Classify patterns:              │                                                │
│  │ • Project-wide vs Module-specific│                                               │
│  └───────────────┬─────────────────┘                                                │
│                  │                                                                   │
│    ┌─────────────┴─────────────┐                                                    │
│    ▼                           ▼                                                    │
│ Project-Wide                Module-Specific                                         │
│ (→ standards/)              (→ stays in basepoints)                                 │
│                                                                                      │
│  Classification criteria:                                                            │
│  • Cross-cutting concerns (testing, lint, naming) → Project-wide                    │
│  • Patterns in 3+ modules → Project-wide                                            │
│  • Single-module patterns → Module-specific                                         │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 9-11: Specialize Agents, Workflows, Structure
═══════════════════════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  • Specialize agent behaviors                                                        │
│  • Adapt workflow complexity                                                         │
│  • Run comprehensive validation                                                      │
│  • Generate deployment report                                                        │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 12-13: Optimize Prompts
═════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/prompting/          │                                                │
│  │   generate-optimizations}}      │                                                │
│  │ {{workflows/prompting/          │                                                │
│  │   apply-prompt-changes}}        │                                                │
│  └─────────────────────────────────┘                                                │
└─────────────────────────────────────────────────────────────────────────────────────┘

OUTPUT: Specialized geist/ ready for development
═══════════════════════════════════════════════════
```

---

### 2. Development Commands

#### shape-spec

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                  SHAPE-SPEC                                          │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 1: Initialize Spec
════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  Creates:                                                                            │
│  geist/specs/[spec-name]/                                                         │
│  ├── planning/                                                                       │
│  │   ├── initialization.md                                                           │
│  │   └── visuals/                                                                    │
│  └── implementation/                                                                 │
│      └── cache/                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 2: Extract Knowledge ◀── ENHANCED
═══════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/common/             │                                                │
│  │   extract-basepoints-with-      │                                                │
│  │   scope-detection}}             │                                                │
│  └───────────────┬─────────────────┘                                                │
│                  │                                                                   │
│    ┌─────────────┼─────────────┬─────────────┬─────────────┐                        │
│    ▼             ▼             ▼             ▼             ▼                        │
│ Extract       Extract       Detect       Scope          Load                        │
│ basepoints    library       layer        detection      results                     │
│ knowledge     basepoints                                                            │
│    │             │             │             │             │                        │
│    └─────────────┴─────────────┴─────────────┴─────────────┘                        │
│                                │                                                     │
│                                ▼                                                     │
│  Variables set:                                                                      │
│  • $EXTRACTED_KNOWLEDGE                                                              │
│  • $LIBRARY_KNOWLEDGE ◀── NEW                                                        │
│  • $DETECTED_LAYER                                                                   │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 2.5: Narrow Focus + Expand Knowledge ◀── NEW
═══════════════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                                                                      │
│  NARROW FOCUS                          EXPAND KNOWLEDGE                             │
│  ════════════                          ════════════════                             │
│  • Scope to spec requirements          • Basepoints knowledge                       │
│  • Filter relevant knowledge           • Library basepoints knowledge               │
│                                        • Product documentation                      │
│                                                                                      │
│  Combined into: $ENRICHED_CONTEXT                                                    │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 3: Research Spec
══════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/specification/      │     Uses enriched context to:                  │
│  │   research-spec}}               │───▶ • Inform clarifying questions              │
│  └─────────────────────────────────┘     • Suggest reusable patterns                │
│                                          • Reference historical decisions           │
│                                                                                      │
│  Output: geist/specs/[spec]/planning/requirements.md                             │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 4-5: Validate & Accumulate ◀── NEW
════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/validation/         │                                                │
│  │   validate-output-exists}}      │                                                │
│  │ {{workflows/validation/         │                                                │
│  │   validate-knowledge-           │                                                │
│  │   integration}}                 │                                                │
│  └─────────────────────────────────┘                                                │
│                                                                                      │
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/common/             │     Stores accumulated knowledge               │
│  │   accumulate-knowledge}}        │───▶ for write-spec to use                      │
│  └─────────────────────────────────┘                                                │
│                                                                                      │
│  Output: geist/specs/[spec]/implementation/cache/accumulated-knowledge.md        │
└─────────────────────────────────────────────────────────────────────────────────────┘

OUTPUT: requirements.md + accumulated knowledge
═══════════════════════════════════════════════
```

#### write-spec

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                   WRITE-SPEC                                         │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 1: Load Accumulated Knowledge ◀── NEW
══════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  Loads: $SPEC_PATH/implementation/cache/accumulated-knowledge.md                     │
│  (Contains knowledge from shape-spec)                                                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 2: Extract Fresh Knowledge
═══════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/common/             │                                                │
│  │   extract-basepoints-with-      │                                                │
│  │   scope-detection}}             │                                                │
│  └─────────────────────────────────┘                                                │
│                                                                                      │
│  Extracts: $EXTRACTED_KNOWLEDGE, $LIBRARY_KNOWLEDGE, $DETECTED_LAYER                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 2.5: Narrow Focus + Expand Knowledge ◀── NEW
═══════════════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  Combines:                                                                           │
│  • Previous accumulated knowledge (from shape-spec)                                  │
│  • Fresh basepoints knowledge                                                        │
│  • Fresh library basepoints knowledge                                                │
│  • Product documentation                                                             │
│                                                                                      │
│  Into: $ENRICHED_CONTEXT (builds upon shape-spec)                                    │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 3: Write Specification
═══════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/specification/      │     Uses enriched context to:                  │
│  │   write-spec}}                  │───▶ • Include library patterns                 │
│  └─────────────────────────────────┘     • Reference basepoints standards           │
│                                          • Suggest existing code                     │
│                                                                                      │
│  Output: geist/specs/[spec]/spec.md                                              │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 4: Review Trade-offs
═════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/human-review/       │     If trade-offs detected:                    │
│  │   review-trade-offs}}           │───▶ • Present options                          │
│  └─────────────────────────────────┘     • Wait for human decision                  │
│                                          • Log to human-decisions.md                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 5-7: Validate & Accumulate
═══════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/common/             │     Stores accumulated knowledge               │
│  │   accumulate-knowledge}}        │───▶ for create-tasks to use                    │
│  └─────────────────────────────────┘                                                │
└─────────────────────────────────────────────────────────────────────────────────────┘

OUTPUT: spec.md + accumulated knowledge (builds on shape-spec)
══════════════════════════════════════════════════════════════
```

#### create-tasks

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                  CREATE-TASKS                                        │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 1: Load Accumulated Knowledge ◀── NEW
══════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  Loads: accumulated-knowledge.md (from shape-spec + write-spec)                      │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 2: Extract Fresh Knowledge
═══════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/common/             │                                                │
│  │   extract-basepoints-with-      │                                                │
│  │   scope-detection}}             │                                                │
│  └─────────────────────────────────┘                                                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 2.5: Narrow Focus + Expand Knowledge ◀── NEW
═══════════════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  Combines:                                                                           │
│  • Previous accumulated knowledge (from shape-spec + write-spec)                     │
│  • Fresh basepoints knowledge                                                        │
│  • Fresh library basepoints knowledge                                                │
│  • Tech stack for implementation context                                             │
│                                                                                      │
│  Into: $ENRICHED_CONTEXT (builds upon write-spec)                                    │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 4: Create Tasks List
═════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/implementation/     │     Uses enriched context to:                  │
│  │   create-tasks-list}}           │───▶ • Structure tasks by library workflows     │
│  └─────────────────────────────────┘     • Include implementation hints             │
│                                          • Reference patterns                        │
│                                                                                      │
│  Output: geist/specs/[spec]/tasks.md                                             │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 7: Accumulate Knowledge
════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/common/             │     Stores accumulated knowledge               │
│  │   accumulate-knowledge}}        │───▶ for orchestrate/implement to use           │
│  └─────────────────────────────────┘                                                │
└─────────────────────────────────────────────────────────────────────────────────────┘

OUTPUT: tasks.md + accumulated knowledge (builds on write-spec)
═══════════════════════════════════════════════════════════════
```

#### orchestrate-tasks

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                               ORCHESTRATE-TASKS                                      │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 1: Extract Knowledge
═════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/common/             │                                                │
│  │   extract-basepoints-with-      │                                                │
│  │   scope-detection}}             │                                                │
│  └─────────────────────────────────┘                                                │
│                                                                                      │
│  Extracts: $EXTRACTED_KNOWLEDGE, $LIBRARY_KNOWLEDGE, $DETECTED_LAYER                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 2: Match Tasks to Basepoints
═════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/scope-detection/    │     For each task group:                       │
│  │   detect-scope-keyword-         │───▶ Find relevant basepoints                   │
│  │   matching}}                    │                                                │
│  └─────────────────────────────────┘                                                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 3: Generate Prompts
════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  For each task group, creates:                                                       │
│  geist/specs/[spec]/implementation/prompts/[N]-[task-group].md                   │
│                                                                                      │
│  Each prompt includes: ◀── ENHANCED                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────────┐    │
│  │  ## Task to Implement                                                        │    │
│  │  ## Context (spec.md, requirements.md)                                       │    │
│  │  ## Basepoints Knowledge Context                                             │    │
│  │  ## Library Basepoints Knowledge ◀── NEW                                     │    │
│  │  ## Implementation Instructions                                              │    │
│  │  ## Standards Compliance                                                     │    │
│  │  ## Completion and Validation                                                │    │
│  └─────────────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 5: Execute Prompts Iteratively ◀── NEW
═══════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────────────────────────────────────────────────┐    │
│  │                                                                              │    │
│  │  for each prompt in prompts/:                                                │    │
│  │      ┌─────────────────────────────────────────────────────────────────┐    │    │
│  │      │  1. Read prompt file                                             │    │    │
│  │      │  2. Execute implementation                                       │    │    │
│  │      │  3. Run validation                                               │    │    │
│  │      │  4. Mark tasks complete                                          │    │    │
│  │      │  5. Update orchestration-status.md                               │    │    │
│  │      │  6. Proceed to next prompt                                       │    │    │
│  │      └─────────────────────────────────────────────────────────────────┘    │    │
│  │                                                                              │    │
│  │  ⚠️ CRITICAL: Always use the specific prompt - never implement directly     │    │
│  │                                                                              │    │
│  └─────────────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────────────┘

OUTPUT: Prompts + Iterative execution + Code changes
════════════════════════════════════════════════════
```

#### implement-tasks

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                IMPLEMENT-TASKS                                       │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 1: Load Accumulated Knowledge ◀── NEW
══════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  Loads: accumulated-knowledge.md (from shape-spec + write-spec + create-tasks)       │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 2: Extract Comprehensive Knowledge ◀── ENHANCED
════════════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/common/             │                                                │
│  │   extract-basepoints-with-      │                                                │
│  │   scope-detection}}             │                                                │
│  └─────────────────────────────────┘                                                │
│                                                                                      │
│  Extracts: $EXTRACTED_KNOWLEDGE, $LIBRARY_KNOWLEDGE, $DETECTED_LAYER                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 2.5: Create Comprehensive Implementation Prompt ◀── NEW
════════════════════════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  Creates $IMPLEMENTATION_PROMPT with:                                                │
│  ┌─────────────────────────────────────────────────────────────────────────────┐    │
│  │  ## Task Requirements                                                        │    │
│  │  ## Previous Accumulated Knowledge                                           │    │
│  │  ## Basepoints Patterns and Standards                                        │    │
│  │  ## Library Knowledge                                                        │    │
│  │     • Patterns and Workflows                                                 │    │
│  │     • Library Capabilities                                                   │    │
│  │     • Library Constraints                                                    │    │
│  │     • Best Practices                                                         │    │
│  │     • Troubleshooting Guidance                                               │    │
│  │  ## Product Context                                                          │    │
│  │  ## Implementation Approach Decision                                         │    │
│  └─────────────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 2.6: Decide on Best Implementation Approach ◀── NEW
════════════════════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  Decision criteria:                                                                  │
│  • Pattern availability from basepoints                                              │
│  • Library capabilities                                                              │
│  • Product alignment                                                                 │
│  • Codebase consistency                                                              │
│                                                                                      │
│  Output: $SPEC_PATH/implementation/cache/implementation-decision.md                  │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 5: Implement Tasks
═══════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/implementation/     │                                                │
│  │   implement-tasks}}             │                                                │
│  └─────────────────────────────────┘                                                │
│                                                                                      │
│  Uses enriched context to implement code changes                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘

Step 6: Validate Implementation
═══════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/validation/         │     Runs:                                      │
│  │   validate-implementation}}     │───▶ • {{PROJECT_BUILD_COMMAND}}                │
│  └─────────────────────────────────┘     • {{PROJECT_TEST_COMMAND}}                 │
│                                          • {{PROJECT_LINT_COMMAND}}                  │
│                                          • {{PROJECT_TYPECHECK_COMMAND}}             │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 3: Verify Implementation ◀── ENHANCED
═══════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ Comprehensive Final Verification│                                                │
│  └───────────────┬─────────────────┘                                                │
│                  │                                                                   │
│    ┌─────────────┼─────────────┬─────────────┬─────────────┬─────────────┐          │
│    ▼             ▼             ▼             ▼             ▼             ▼          │
│ Check         Verify        Check         Check         Verify        Ensure       │
│ problems      references    docs          code          pattern       complete     │
│ & gaps        (imports)     updates       quality       consistency   -ness        │
│                                                                                      │
│  Output: $SPEC_PATH/implementation/cache/verification-issues.md                      │
└─────────────────────────────────────────────────────────────────────────────────────┘

OUTPUT: Code changes + verification report
══════════════════════════════════════════
```

#### fix-bug ◀── NEW COMMAND

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                    FIX-BUG                                           │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 1: Analyze Issue
══════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  Input: Bug report / Error log / Feedback                                            │
│                                                                                      │
│  Extracts:                                                                           │
│  • Input type (bug/feedback)                                                         │
│  • Error message, stack trace, error code                                            │
│  • Affected libraries and modules                                                    │
│                                                                                      │
│  Output: geist/output/fix-bug/cache/issue-analysis.md                            │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 2: Research Libraries
═══════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  For each affected library:                                                          │
│  • Research internal architecture                                                    │
│  • Research known issues and bug patterns                                            │
│  • Research error scenarios                                                          │
│                                                                                      │
│  Output: geist/output/fix-bug/cache/library-research.md                          │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 3: Integrate Basepoints
═════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/common/             │                                                │
│  │   extract-basepoints-with-      │                                                │
│  │   scope-detection}}             │                                                │
│  └─────────────────────────────────┘                                                │
│                                                                                      │
│  • Find basepoints describing error location                                         │
│  • Extract relevant patterns and standards                                           │
│  • Identify similar issues                                                           │
│                                                                                      │
│  Output: geist/output/fix-bug/cache/basepoints-integration.md                    │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 4: Analyze Code
═════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  • Identify exact file/module locations                                              │
│  • Deep-dive into relevant code files                                                │
│  • Analyze code patterns and flows                                                   │
│  • Trace execution paths                                                             │
│                                                                                      │
│  Output: geist/output/fix-bug/cache/code-analysis.md                             │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 5: Synthesize Knowledge
═════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  Combines:                                                                           │
│  • Issue analysis                                                                    │
│  • Library research                                                                  │
│  • Basepoints integration                                                            │
│  • Code analysis                                                                     │
│                                                                                      │
│  Creates:                                                                            │
│  • Unified issue context                                                             │
│  • Prioritized fix approaches                                                        │
│  • Fix implementation context                                                        │
│                                                                                      │
│  Output: geist/output/fix-bug/cache/knowledge-synthesis.md                       │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 6: Implement Fix (Iterative)
══════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────────────────────────────────────────────────┐    │
│  │                                                                              │    │
│  │  while (not success && worsening_count < 3):                                 │    │
│  │      ┌─────────────────────────────────────────────────────────────────┐    │    │
│  │      │  1. Implement fix using synthesized knowledge                    │    │    │
│  │      │  2. Apply library-specific patterns                              │    │    │
│  │      │  3. Follow basepoints standards                                  │    │    │
│  │      │  4. Run validation                                               │    │    │
│  │      │  5. Check result:                                                │    │    │
│  │      │     • Success → Generate fix-report.md                           │    │    │
│  │      │     • Getting closer → Continue                                  │    │    │
│  │      │     • Worsening → Increment counter                              │    │    │
│  │      └─────────────────────────────────────────────────────────────────┘    │    │
│  │                                                                              │    │
│  │  if (worsening_count >= 3):                                                  │    │
│  │      Generate guidance-request.md                                            │    │
│  │      Present knowledge summary                                               │    │
│  │      Request user guidance                                                   │    │
│  │                                                                              │    │
│  └─────────────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────────────┘

OUTPUT: fix-report.md (success) OR guidance-request.md (needs help)
═══════════════════════════════════════════════════════════════════
```

---

### 3. Maintenance Commands

#### cleanup-geist

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                               CLEANUP-AGENT-OS                                       │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 1: Validate & Run Validation
══════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/validation/         │                                                │
│  │   orchestrate-validation}}      │                                                │
│  └─────────────────────────────────┘                                                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 2: Clean Placeholders
═══════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/validation/         │     Replaces remaining                         │
│  │   detect-placeholders}}         │───▶ {{PLACEHOLDER}} with values                │
│  └─────────────────────────────────┘                                                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 3: Remove Unnecessary Logic
═════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/validation/         │     Removes project-agnostic                   │
│  │   detect-unnecessary-logic}}    │───▶ conditionals                               │
│  └─────────────────────────────────┘                                                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 4: Fix Broken References
══════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/validation/         │     Fixes @geist/                           │
│  │   validate-references}}         │───▶ references                                 │
│  └─────────────────────────────────┘                                                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 5: Verify Knowledge Completeness
══════════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/validation/         │     Verifies:                                  │
│  │   validate-knowledge-           │───▶ • Basepoints completeness                  │
│  │   integration}}                 │     • Product knowledge                        │
│  └─────────────────────────────────┘     • Coverage analysis                        │
└─────────────────────────────────────────────────────────────────────────────────────┘

OUTPUT: Cleaned geist/ + cleanup report
══════════════════════════════════════════
```

#### update-basepoints-and-redeploy

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                        UPDATE-BASEPOINTS-AND-REDEPLOY                                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 1: Detect Changes
═══════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/codebase-analysis/  │     Uses git diff or timestamps               │
│  │   detect-codebase-changes}}     │───▶ to find changed files                      │
│  └─────────────────────────────────┘                                                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 2: Identify Affected Basepoints
═════════════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  Maps changed files to affected basepoints                                           │
│  Calculates parent propagation                                                       │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 3: Update Basepoints
══════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/codebase-analysis/  │     Processes children before                  │
│  │   incremental-basepoint-        │───▶ parents, creates backups                   │
│  │   update}}                      │                                                │
│  └─────────────────────────────────┘                                                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 4: Re-extract Knowledge
═════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────┐                                                │
│  │ {{workflows/basepoints/         │                                                │
│  │   extract-basepoints-knowledge- │                                                │
│  │   automatic}}                   │                                                │
│  └─────────────────────────────────┘                                                │
└─────────────────────────────────────────────────────────────────────────────────────┘

Phase 5: Re-specialize Commands
═══════════════════════════════
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  Re-specializes all 5 core commands:                                                 │
│  • shape-spec                                                                        │
│  • write-spec                                                                        │
│  • create-tasks                                                                      │
│  • implement-tasks                                                                   │
│  • orchestrate-tasks                                                                 │
└─────────────────────────────────────────────────────────────────────────────────────┘

OUTPUT: Updated basepoints + re-specialized commands
════════════════════════════════════════════════════
```

---

## Knowledge Flow

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                         KNOWLEDGE ACCUMULATION FLOW                                  │
└─────────────────────────────────────────────────────────────────────────────────────┘

┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────────────┐
│ shape-spec  │────▶│ write-spec  │────▶│create-tasks │────▶│orchestrate/implement│
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────────┬──────────┘
       │                   │                   │                       │
       ▼                   ▼                   ▼                       ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────────────┐
│ Accumulated │     │ Accumulated │     │ Accumulated │     │ Uses all accumulated│
│ Knowledge   │────▶│ Knowledge   │────▶│ Knowledge   │────▶│ knowledge for       │
│ (initial)   │     │ (+ write)   │     │ (+ tasks)   │     │ implementation      │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────────────┘

Each command:
1. Loads previous accumulated knowledge
2. Extracts fresh knowledge (basepoints + library basepoints)
3. Combines into enriched context
4. Stores accumulated knowledge for next command
```

---

## Output Locations

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              OUTPUT LOCATIONS                                        │
└─────────────────────────────────────────────────────────────────────────────────────┘

geist/
├── basepoints/                          # Created by: create-basepoints
│   ├── headquarter.md
│   ├── [layer]/[module]/agent-base-*.md
│   └── libraries/                       # Created by: Phase 8
│       ├── data/
│       ├── domain/
│       ├── util/
│       ├── infrastructure/
│       ├── framework/
│       └── README.md
│
├── commands/                            # Specialized by: deploy-agents
├── workflows/                           # Specialized by: deploy-agents
├── standards/                           # Specialized by: deploy-agents
│
├── config/
│   ├── project-profile.yml              # Created by: adapt-to-product
│   └── enriched-knowledge/              # Created by: adapt-to-product
│
├── product/                             # Created by: adapt-to-product/plan-product
│   ├── mission.md
│   ├── roadmap.md
│   └── tech-stack.md
│
├── specs/[spec-name]/                   # Created by: shape-spec
│   ├── spec.md                          # Created by: write-spec
│   ├── tasks.md                         # Created by: create-tasks
│   ├── orchestration.yml                # Created by: orchestrate-tasks
│   ├── planning/
│   │   ├── initialization.md
│   │   ├── requirements.md
│   │   └── visuals/
│   └── implementation/
│       ├── cache/
│       │   ├── basepoints-knowledge.md
│       │   ├── library-basepoints-knowledge.md
│       │   ├── accumulated-knowledge.md
│       │   ├── detected-layer.txt
│       │   ├── implementation-decision.md
│       │   ├── verification-issues.md
│       │   └── validation-report.md
│       └── prompts/                     # Created by: orchestrate-tasks
│           └── [N]-[task-group].md
│
└── output/
    ├── deploy-agents/                   # Created by: deploy-agents
    │   ├── knowledge/
    │   └── reports/
    ├── product-cleanup/                 # Created by: adapt-to-product
    │   ├── detected-scope.yml
    │   └── cleanup-report.md
    └── fix-bug/                         # Created by: fix-bug
        └── cache/
            ├── issue-analysis.md
            ├── library-research.md
            ├── basepoints-integration.md
            ├── code-analysis.md
            ├── knowledge-synthesis.md
            ├── fix-report.md
            └── guidance-request.md
```

---

*Last Updated: 2026-01-18*
*Version: 2.0 (includes context enrichment and fix-bug command)*
