# profiles/default - Project-Agnostic Agent-OS Profile (Default)

## Overview

`profiles/default` is a **project-agnostic template** that provides a complete, reusable Agent OS structure for any software project. It contains abstract commands, workflows, standards, and agents that can be installed into any project and then specialized to match that project's specific structure, patterns, and requirements.

**Important**: This is the **default profile** in Geist that gets used automatically when Geist is installed in a project.

---

## Table of Contents

- [What is Geist vs Agent OS?](#what-is-geist-vs-agent-os)
- [Architecture](#architecture)
- [Installation Flow](#installation-flow)
- [Command Reference](#command-reference)
- [Workflow Reference](#workflow-reference)
- [Configuration Files](#configuration-files)
- [Specialization Process](#specialization-process)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

---

## What is Geist vs Agent OS?

### Geist (This Repository)

**Geist** is the tool/framework that builds Agent OS instances:

- **Project-Agnostic**: Works with any software project
- **Template-Based**: Provides reusable, technology-agnostic profile templates
- **Builder Tool**: Creates specialized Agent OS instances in projects
- **Abstract**: Contains abstract, project-agnostic templates
- **Location**: This repository (`geist/` or similar)

### Agent OS (In Your Project)

**Agent OS** is the specialized instance created by Geist:

- **Project-Specific**: Tailored to your codebase after specialization
- **Specialized**: Uses your project's patterns and structure
- **Ready to Use**: After deployment, ready for spec-driven development
- **Location**: Lives in your project's `agent-os/` folder
- **Created By**: Geist installs and specializes it

---

## Architecture

```
profiles/default/
├── commands/              # Abstract, project-agnostic commands
│   ├── adapt-to-product/     # Extract product info (with auto-detection)
│   ├── create-basepoints/    # Create codebase documentation
│   ├── deploy-agents/        # Specialize templates (uses all knowledge)
│   ├── shape-spec/           # Research and shape features
│   ├── write-spec/           # Write detailed specifications
│   ├── create-tasks/         # Break specs into tasks
│   ├── implement-tasks/      # Implement tasks
│   ├── orchestrate-tasks/    # Multi-agent task coordination
│   ├── cleanup-agent-os/     # Clean and verify
│   └── update-basepoints-and-redeploy/  # Incremental updates
│
├── workflows/             # Reusable workflow templates
│   ├── basepoints/           # Knowledge extraction from codebase
│   ├── codebase-analysis/    # Codebase analysis and change detection
│   ├── detection/            # 🆕 Project profile detection
│   ├── research/             # 🆕 Web research for libraries/patterns
│   ├── deep-reading/         # Deep code reading
│   ├── human-review/         # Trade-off and contradiction detection
│   ├── implementation/       # Implementation workflows
│   ├── learning/             # 🆕 Session feedback and learning
│   ├── planning/             # Product planning
│   ├── prompting/            # 🆕 Prompt construction and optimization
│   ├── scope-detection/      # Scope and layer detection
│   ├── specification/        # Specification workflows
│   └── validation/           # Deterministic validation
│
├── standards/             # Global coding standards and conventions
│   └── global/
│       ├── conventions.md
│       ├── tech-stack.md
│       ├── project-profile-schema.md      # 🆕 Profile structure
│       ├── enriched-knowledge-templates.md # 🆕 Research templates
│       └── validation-commands.md          # 🆕 Validation commands
│
├── agents/                # Agent definitions and behaviors
│
├── docs/                  # 🆕 Detailed documentation
│   ├── COMMAND-FLOWS.md      # Detailed command documentation
│   └── INSTALLATION-GUIDE.md # Step-by-step installation
│
└── README.md              # This file
```

---

## Installation Flow

### Complete Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         COMPLETE GEIST FLOW                                  │
└─────────────────────────────────────────────────────────────────────────────┘

INSTALLATION PHASE
┌─────────────────────────────────────────────────────────────────────────────┐
│  1. Clone Geist (one-time)                                                   │
│     git clone <repo-url> ~/geist                                            │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│  2. Project Installation                                                     │
│     ~/geist/scripts/project-install.sh --profile default                    │
│     → Creates agent-os/ in your project                                     │
│     → Installs project-agnostic templates                                   │
└─────────────────────────────────────────────────────────────────────────────┘

SPECIALIZATION PHASE
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│  3. Product Definition                                                       │
│     /adapt-to-product  OR  /plan-product                                    │
│                                                                              │
│     🆕 AUTOMATIC DETECTION:                                                  │
│     ┌─────────────────────────────────────────────────────────────────────┐ │
│     │ • Scans: package.json, Cargo.toml, go.mod, requirements.txt        │ │
│     │ • Detects: tech stack, frameworks, databases                        │ │
│     │ • Extracts: build/test/lint commands                                │ │
│     │ • Analyzes: security level, project complexity                      │ │
│     │ • Infers: architecture patterns, module boundaries                  │ │
│     └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│     🆕 WEB RESEARCH:                                                         │
│     ┌─────────────────────────────────────────────────────────────────────┐ │
│     │ • Library best practices                                            │ │
│     │ • Known issues and CVEs                                             │ │
│     │ • Latest versions (version analysis)                                │ │
│     │ • Stack architecture patterns                                       │ │
│     └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│     🆕 MINIMAL QUESTIONS (only 2-3):                                         │
│     ┌─────────────────────────────────────────────────────────────────────┐ │
│     │ 1. Compliance requirements? [None/SOC2/HIPAA/GDPR]                 │ │
│     │ 2. Human review preference? [Minimal/Moderate/High]                │ │
│     └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│     → Creates: agent-os/product/{mission,roadmap,tech-stack}.md             │
│     → Creates: agent-os/config/project-profile.yml (NEW)                    │
│     → Creates: agent-os/config/enriched-knowledge/ (NEW)                    │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│  4. Basepoints Creation                                                      │
│     /create-basepoints                                                       │
│                                                                              │
│     🆕 LOADS EXISTING PROFILE (no re-detection)                             │
│     🆕 ARCHITECTURE RESEARCH (adds domain knowledge)                        │
│     🆕 ASKS ONLY IF MODULE DETECTION UNCLEAR                               │
│                                                                              │
│     → Creates: agent-os/basepoints/{headquarter,modules}.md                 │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│  5. Specialization                                                           │
│     /deploy-agents                                                           │
│                                                                              │
│     🆕 USES ALL GATHERED KNOWLEDGE:                                         │
│     ┌─────────────────────────────────────────────────────────────────────┐ │
│     │ • project-profile.yml → Validation commands, tech decisions        │ │
│     │ • enriched-knowledge/ → Workflow complexity, patterns              │ │
│     │ • basepoints/ → Patterns, standards, strategies                    │ │
│     │ • product/ → Mission alignment, tech stack                         │ │
│     └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│     → Transforms: Abstract templates → Project-specific                     │
│     → Configures: Validation commands automatically                         │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│  6. Cleanup & Verification                                                   │
│     /cleanup-agent-os                                                        │
│                                                                              │
│     → Cleans: placeholders, unnecessary logic                               │
│     → Verifies: knowledge completeness                                       │
│     → Reports: recommendations                                               │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
                              ✅ READY TO USE
                              Specialized Agent OS instance

MAINTENANCE PHASE (after codebase changes)
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│  7. Incremental Update                                                       │
│     /update-basepoints-and-redeploy                                         │
│                                                                              │
│     → Detects changes (git or timestamps)                                   │
│     → Updates only affected basepoints                                      │
│     → Re-specializes all commands                                           │
│     → Validates and reports                                                  │
│                                                                              │
│     🆕 PHASE 7: Review Session Learnings                                    │
│     ┌─────────────────────────────────────────────────────────────────────┐ │
│     │ • Extract patterns from session data                                 │ │
│     │ • Analyze prompt effectiveness                                       │ │
│     │ • Generate adaptation suggestions                                    │ │
│     │ • Present for human review (approve/select/skip)                    │ │
│     └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│     🆕 PHASE 8: Adapt Commands (if approved)                                │
│     ┌─────────────────────────────────────────────────────────────────────┐ │
│     │ • Apply user-approved adaptations to commands                        │ │
│     │ • Create backup before changes                                       │ │
│     │ • Log all changes to applied.md                                      │ │
│     │ • Archive session to history                                         │ │
│     └─────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Command Reference

### Specialization Commands

#### 1. adapt-to-product

**Purpose**: Extract product information from existing codebase

**Flow**:
```
┌─────────────────────────────────────────────────────────────────┐
│  adapt-to-product Flow (Enhanced)                                │
└─────────────────────────────────────────────────────────────────┘

Phase 1: Setup & Information Gathering (Enhanced)
├─ 🆕 Call detect-project-profile workflow
│   ├─ detect-tech-stack.md → Languages, frameworks, deps
│   ├─ detect-commands.md → Build, test, lint commands
│   ├─ detect-architecture.md → Module structure, patterns
│   └─ detect-security-level.md → Auth, secrets, SSL
├─ 🆕 Call research-orchestrator workflow
│   ├─ research-library.md → Best practices, known issues
│   ├─ research-stack-patterns.md → Architecture patterns
│   └─ research-security.md → CVE vulnerabilities
├─ 🆕 Call present-and-confirm workflow
│   └─ Show detected values, allow overrides
├─ 🆕 Ask ONLY compliance + human review
└─ 🆕 Store to project-profile.yml + enriched-knowledge/
    ↓
Phase 2-6: (Existing phases)
├─ Analyze codebase
├─ Create mission.md
├─ Create roadmap.md
├─ Create tech-stack.md
└─ Review and combine knowledge
```

**Outputs**:
- `agent-os/product/mission.md`
- `agent-os/product/roadmap.md`
- `agent-os/product/tech-stack.md`
- `agent-os/config/project-profile.yml` (NEW)
- `agent-os/config/enriched-knowledge/` (NEW)

---

#### 2. create-basepoints

**Purpose**: Create comprehensive codebase documentation

**Flow**:
```
┌─────────────────────────────────────────────────────────────────┐
│  create-basepoints Flow (Enhanced)                               │
└─────────────────────────────────────────────────────────────────┘

Phase 1: Validate Prerequisites (Enhanced)
├─ 🆕 Load existing project-profile.yml
├─ 🆕 Call detection ONLY for missing values
├─ 🆕 Call architecture-specific research
└─ 🆕 Store updated profile
    ↓
Phase 2-7: (Existing phases)
├─ Detect abstraction layers
├─ Mirror project structure
├─ Analyze codebase
├─ Generate module basepoints
├─ Generate parent basepoints
└─ Generate headquarter.md
```

**Outputs**:
- `agent-os/basepoints/headquarter.md`
- `agent-os/basepoints/[layer]/[module]/agent-base-[module].md`

---

#### 3. deploy-agents

**Purpose**: Transform abstract templates into project-specific implementations

**Flow**:
```
┌─────────────────────────────────────────────────────────────────┐
│  deploy-agents Flow (Enhanced)                                   │
└─────────────────────────────────────────────────────────────────┘

Phase 1: Validate Prerequisites (Enhanced)
├─ 🆕 Load project-profile.yml
│   ├─ Extract: language, framework, security level
│   ├─ Extract: build, test, lint commands
│   └─ Extract: human review preference
├─ 🆕 Load enriched-knowledge/
│   ├─ Check: library-research.md
│   ├─ Check: security-notes.md (flag critical issues)
│   └─ Check: version-analysis.md (flag outdated)
├─ 🆕 Ask ONLY if preferences not set
└─ 🆕 Determine specialization hints
    ↓
Phase 2-7: (Existing phases)
├─ Extract basepoints knowledge
├─ Extract product knowledge
├─ Merge knowledge and resolve conflicts
├─ Specialize shape-spec and write-spec
├─ Specialize task commands
└─ Update supporting structures
    ↓
Phase 8: Specialize Standards (Enhanced)
├─ 🆕 Detect validation commands from project
├─ 🆕 Replace {{PROJECT_BUILD_COMMAND}} etc.
└─ 🆕 Configure validate-implementation.md
    ↓
Phase 9: Specialize Agents (Enhanced)
├─ 🆕 Generate layer-specialist agents
│   ├─ Read detected abstraction layers from headquarter.md
│   ├─ Create: ui-specialist, api-specialist, data-specialist, etc.
│   ├─ Each specialist knows layer-specific basepoints
│   └─ Generate: agent-os/agents/specialists/registry.yml
├─ Evaluate agent relevance based on complexity
└─ Create project-specific agents
    ↓
Phase 10: Specialize Workflows (Enhanced)
├─ 🆕 Apply complexity-based simplification
│   ├─ Simple: specification + implementation only
│   ├─ Moderate: + planning + detection + research
│   └─ Complex: all workflows + layer validations
├─ Create workflow-config.yml
└─ Configure layer validation triggers
    ↓
Phase 11: Adapt structure and finalize
    ↓
Phase 12: Optimize Prompts (NEW)
├─ 🆕 Analyze codebase patterns and conventions
├─ 🆕 Generate project-specific prompt improvements
└─ 🆕 Present suggestions for user approval
    ↓
Phase 13: Apply Prompt Optimizations (NEW)
├─ 🆕 Apply user-approved improvements
├─ 🆕 Create backup before changes
└─ 🆕 Log all changes for audit
```

**Outputs**:
- Specialized commands in `agent-os/commands/`
- Specialized workflows in `agent-os/workflows/`
- Configured validation commands
- 🆕 Layer specialists in `agent-os/agents/specialists/`
- 🆕 Specialist registry in `agent-os/agents/specialists/registry.yml`
- 🆕 Workflow config in `agent-os/workflows/workflow-config.yml`

---

### Development Commands

#### 4. shape-spec

**Purpose**: Research and shape a new feature specification

**Flow**:
```
┌─────────────────────────────────────────────────────────────────┐
│  shape-spec Flow (Enhanced)                                      │
└─────────────────────────────────────────────────────────────────┘

Phase 1: Initialize
├─ Create spec folder structure
└─ Store feature description
    ↓
Phase 2: Shape Spec (Enhanced)
├─ 🆕 Step 0: Construct Optimal Prompt
│   └─ {{workflows/prompting/construct-prompt}}
├─ 🆕 Extract basepoints knowledge
│   ├─ Detect abstraction layer
│   ├─ Find relevant patterns
│   └─ Cache to implementation/cache/
├─ 🆕 Inject knowledge into clarifying questions
├─ 🆕 Suggest reusable modules from basepoints
├─ Gather requirements through Q&A
├─ 🆕 Run validation before completing
└─ 🆕 Save Handoff
    └─ {{workflows/prompting/save-handoff}}
```

**Outputs**:
- `agent-os/specs/[spec-name]/planning/requirements.md`
- `agent-os/specs/[spec-name]/planning/initialization.md`
- `agent-os/specs/[spec-name]/implementation/cache/basepoints-knowledge.md` (NEW)
- `agent-os/specs/[spec-name]/implementation/cache/detected-layer.txt` (NEW)

---

#### 5. write-spec

**Purpose**: Write detailed specification from requirements

**Flow**:
```
┌─────────────────────────────────────────────────────────────────┐
│  write-spec Flow (Enhanced)                                      │
└─────────────────────────────────────────────────────────────────┘

├─ 🆕 Step 0: Construct Optimal Prompt
│   └─ {{workflows/prompting/construct-prompt}}
├─ 🆕 Load extracted knowledge from cache
├─ 🆕 Reference applicable standards in spec
├─ 🆕 Suggest existing code from basepoints
├─ Write detailed specification
├─ 🆕 Detect trade-offs for human review
├─ 🆕 Run validation before completing
└─ 🆕 Save Handoff
    └─ {{workflows/prompting/save-handoff}}
```

**Outputs**:
- `agent-os/specs/[spec-name]/spec.md`
- `agent-os/specs/[spec-name]/implementation/cache/resources-consulted.md` (NEW)

---

#### 6. create-tasks

**Purpose**: Break specification into actionable tasks

**Flow**:
```
├─ 🆕 Step 0: Construct Optimal Prompt
│   └─ {{workflows/prompting/construct-prompt}}
├─ 🆕 Load extracted knowledge from cache
├─ 🆕 Include implementation hints from patterns
├─ 🆕 Reference basepoints strategies in descriptions
├─ Create task groups
├─ Create tasks with acceptance criteria
├─ 🆕 Run validation before completing
└─ 🆕 Save Handoff
    └─ {{workflows/prompting/save-handoff}}
```

**Outputs**:
- `agent-os/specs/[spec-name]/tasks.md`

---

#### 7. implement-tasks / orchestrate-tasks

**Purpose**: Implement tasks with full knowledge context

**Flow**:
```
┌─────────────────────────────────────────────────────────────────┐
│  implement-tasks / orchestrate-tasks Flow (Enhanced)             │
└─────────────────────────────────────────────────────────────────┘

├─ 🆕 Step 0: Construct Optimal Prompt
│   └─ {{workflows/prompting/construct-prompt}}
│      ├─ Load basepoints context
│      ├─ Load session learnings (patterns, anti-patterns)
│      └─ Build optimized prompt
├─ 🆕 Load module-specific patterns from cache
├─ 🆕 Provide coding patterns in context
├─ 🆕 Reference standards for code style
├─ Implement/orchestrate tasks
├─ 🆕 Run project-specific validation
│   ├─ {{PROJECT_BUILD_COMMAND}}
│   ├─ {{PROJECT_TEST_COMMAND}}
│   ├─ {{PROJECT_LINT_COMMAND}}
│   └─ {{PROJECT_TYPECHECK_COMMAND}}
├─ 🆕 Capture Session Feedback (NEW)
│   └─ {{workflows/learning/capture-session-feedback}}
│      ├─ Record implementation outcome
│      ├─ Track patterns used
│      └─ Record prompt effectiveness
├─ 🆕 Check for human review (trade-offs)
└─ 🆕 Save Handoff
    └─ {{workflows/prompting/save-handoff}}
```

**Outputs**:
- Code changes
- `orchestration.yml` (orchestrate-tasks)
- `implementation/prompts/` (orchestrate-tasks)
- `implementation/cache/validation-report.md`

---

## Workflow Reference

### Detection Workflows (NEW)

| Workflow | Purpose | Output |
|----------|---------|--------|
| `detect-project-profile.md` | Orchestrate all detection | Unified profile |
| `detect-tech-stack.md` | Parse config files for tech | Languages, frameworks, deps |
| `detect-commands.md` | Extract build/test/lint | Command strings |
| `detect-architecture.md` | Analyze directory structure | Architecture patterns |
| `detect-security-level.md` | Check auth/secrets | Security level (low/moderate/high) |
| `present-and-confirm.md` | Show detected values | Confirmed profile |
| `question-templates.md` | Minimal question templates | User preferences |

### Research Workflows (NEW)

| Workflow | Purpose | Output |
|----------|---------|--------|
| `research-orchestrator.md` | Coordinate research | Enriched knowledge |
| `research-library.md` | Research library best practices | Best practices, issues |
| `research-stack-patterns.md` | Research stack architecture | Architecture patterns |
| `research-domain.md` | Research domain patterns | Domain knowledge |
| `research-security.md` | Research CVE vulnerabilities | Security notes |
| `synthesize-knowledge.md` | Combine research outputs | Unified knowledge |
| `version-analysis.md` | Compare versions | Outdated deps |

### Validation Workflows (NEW)

| Workflow | Purpose | Output |
|----------|---------|--------|
| `validate-output-exists.md` | Check required files exist | Pass/fail |
| `validate-knowledge-integration.md` | Check knowledge was used | Pass/fail |
| `validate-references.md` | Check @agent-os/ refs resolve | Broken refs |
| `generate-validation-report.md` | Generate markdown report | Report |
| `validation-registry.md` | Core + project validators | Validator list |
| `validate-implementation.md` | Run build/test/lint | Pass/fail |
| `validate-detection-accuracy.md` | Validate detection results | Accuracy report |
| `detection-tests.md` | Integration tests for detection | Test results |
| `validate-ui-patterns.md` | 🆕 Validate UI layer patterns | UI issues |
| `validate-api-patterns.md` | 🆕 Validate API layer patterns | API issues |
| `validate-data-patterns.md` | 🆕 Validate data layer patterns | Data issues |
| `orchestrate-validation.md` | 🆕 Run all validators incl. layer | Comprehensive report |

### Learning Workflows (NEW)

| Workflow | Purpose | Output |
|----------|---------|--------|
| `capture-session-feedback.md` | Capture implementation outcomes | Session feedback data |
| `extract-session-patterns.md` | Extract successful/failed patterns | Pattern files |
| `present-learnings-for-review.md` | Present learnings to user | User approval choice |
| `apply-command-adaptations.md` | Apply approved adaptations | Updated commands |

### Prompting Workflows (NEW)

| Workflow | Purpose | Output |
|----------|---------|--------|
| `construct-prompt.md` | **CORE** - Construct optimal prompt for commands | Optimized prompt |
| `save-handoff.md` | Save context for next command | Handoff file |
| `prompting-best-practices.md` | Reference guide for prompt construction | Best practices |
| `generate-optimizations.md` | Generate project-specific prompt improvements | Optimization suggestions |
| `apply-prompt-changes.md` | Apply approved prompt optimizations | Updated commands |
| `analyze-prompt-effectiveness.md` | Analyze prompt effectiveness from session | Effectiveness analysis |

### Human Review Workflows (Enhanced)

| Workflow | Purpose | Output |
|----------|---------|--------|
| `detect-trade-offs.md` | Detect pattern conflicts | Trade-off list (NEW) |
| `detect-contradictions.md` | Detect standard violations | Contradiction list (NEW) |
| `present-human-decision.md` | Format for human review | Decision log (NEW) |
| `review-trade-offs.md` | Orchestrate review | Review result (Enhanced) |
| `create-checkpoint.md` | Create review checkpoint | Checkpoint |

### Basepoints Workflows (Enhanced)

| Workflow | Purpose | Output |
|----------|---------|--------|
| `extract-basepoints-knowledge-automatic.md` | Auto-extract on command start | Knowledge cache (Enhanced) |
| `extract-basepoints-knowledge-on-demand.md` | Targeted extraction | Filtered knowledge (Enhanced) |
| `organize-and-cache-basepoints-knowledge.md` | Per-spec caching | Cached knowledge (Enhanced) |

### Scope Detection Workflows (Enhanced)

| Workflow | Purpose | Output |
|----------|---------|--------|
| `detect-scope-semantic-analysis.md` | Semantic concept extraction | Layer mapping (Enhanced) |
| `detect-scope-keyword-matching.md` | Keyword matching | Matched modules (Enhanced) |
| `detect-abstraction-layer.md` | Detect feature layer | detected-layer.txt (NEW) |

---

## Configuration Files

### project-profile.yml (NEW)

```yaml
# agent-os/config/project-profile.yml
# Generated from automatic detection + user confirmation

gathered:
  # Auto-detected (high confidence)
  project_type: web_application
  tech_stack:
    language: typescript
    framework: react
    backend: nodejs
    database: postgresql
  size:
    lines: 15234
    files: 120
    modules: 8
  commands:
    build: "npm run build"
    test: "npm test"
    lint: "npm run lint"
  
  # Inferred (medium confidence)
  security_level: high
  complexity: moderate

user_confirmed:
  # User validated these inferences
  security_level: high
  
user_specified:
  # Only things user had to manually specify
  compliance: [gdpr]
  human_review_level: moderate

_meta:
  detected_at: 2026-01-16T12:00:00Z
  detection_confidence: 0.92
  questions_asked: 2
  questions_auto_answered: 24
```

### enriched-knowledge/ (NEW)

```
agent-os/config/enriched-knowledge/
├── library-research.md       # Per-library best practices
├── stack-best-practices.md   # Tech stack patterns
├── domain-knowledge.md       # Domain-specific info
├── version-analysis.md       # Outdated deps analysis
└── security-notes.md         # CVE vulnerabilities
```

### validation-commands.md (NEW)

Defines project-specific validation commands used by `validate-implementation.md`:

- `{{PROJECT_BUILD_COMMAND}}` → e.g., `npm run build`
- `{{PROJECT_TEST_COMMAND}}` → e.g., `npm test`
- `{{PROJECT_LINT_COMMAND}}` → e.g., `npm run lint`
- `{{PROJECT_TYPECHECK_COMMAND}}` → e.g., `tsc --noEmit`

---

## Specialization Process

### Knowledge Flow

```
┌───────────────────────────────────────────────────────────────────────────┐
│                         KNOWLEDGE AGGREGATION                              │
└───────────────────────────────────────────────────────────────────────────┘

        ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
        │   DETECTION     │     │    RESEARCH     │     │   BASEPOINTS    │
        │                 │     │                 │     │                 │
        │ project-profile │     │ enriched-       │     │ headquarter.md  │
        │ .yml            │     │ knowledge/      │     │ module-*.md     │
        └────────┬────────┘     └────────┬────────┘     └────────┬────────┘
                 │                       │                       │
                 └───────────────────────┼───────────────────────┘
                                         │
                                         ▼
        ┌───────────────────────────────────────────────────────────────┐
        │                    UNIFIED KNOWLEDGE                          │
        │                                                               │
        │  • Tech stack decisions (from detection + product)           │
        │  • Validation commands (from detection)                       │
        │  • Best practices (from research)                            │
        │  • Patterns & standards (from basepoints)                    │
        │  • Strategies & flows (from basepoints)                      │
        │  • Security considerations (from research + detection)       │
        └───────────────────────────────────────────────────────────────┘
                                         │
                                         ▼
        ┌───────────────────────────────────────────────────────────────┐
        │                    SPECIALIZATION                             │
        │                                                               │
        │  • Replace {{PLACEHOLDERS}} with actual values               │
        │  • Configure validation commands                              │
        │  • Inject patterns into commands                             │
        │  • Adapt complexity based on project                         │
        │  • Set human review triggers                                 │
        └───────────────────────────────────────────────────────────────┘
```

### Transformation Example

**Before (Abstract Template)**:
```markdown
## Validate Implementation

```bash
BUILD_COMMAND="{{PROJECT_BUILD_COMMAND}}"
TEST_COMMAND="{{PROJECT_TEST_COMMAND}}"
...
```
```

**After (Specialized)**:
```markdown
## Validate Implementation

```bash
BUILD_COMMAND="npm run build"
TEST_COMMAND="npm test"
...
```
```

---

## Layer Specialists (NEW)

During `/deploy-agents`, Geist generates layer-specialist agents based on detected abstraction layers:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      LAYER SPECIALIST GENERATION                             │
└─────────────────────────────────────────────────────────────────────────────┘

headquarter.md                    Generated Specialists
┌──────────────────────┐         ┌──────────────────────────────────────────┐
│ Detected Layers:     │         │ agent-os/agents/specialists/             │
│ • UI/Frontend        │ ──────► │ ├─ ui-specialist.md                      │
│ • API/Backend        │         │ ├─ api-specialist.md                     │
│ • Data/Persistence   │         │ ├─ data-specialist.md                    │
│ • Platform           │         │ ├─ platform-specialist.md                │
│ • Testing            │         │ └─ registry.yml                          │
└──────────────────────┘         └──────────────────────────────────────────┘
```

### How Specialists Work

Each specialist knows:
- **Layer-specific patterns** from basepoints
- **Relevant standards** for that layer
- **How to stay within layer boundaries**

### Auto-Assignment in Orchestration

During `/orchestrate-tasks`, task groups are analyzed and matched to specialists:

```
Task: "Create user profile component"
  ↓ Keyword analysis: "component", "profile"
  ↓ Detected layer: UI
  ↓ Suggested: ui-specialist
```

### Specialist Registry

The registry enables automatic specialist selection:

```yaml
# agent-os/agents/specialists/registry.yml
specialists:
  - name: ui-specialist
    layer: ui
  - name: api-specialist
    layer: api
  - name: data-specialist
    layer: data

layer_keywords:
  ui: [component, view, screen, button, form, modal]
  api: [endpoint, route, controller, handler, request]
  data: [model, schema, migration, query, database]
```

---

## Complexity-Based Simplification (NEW)

Geist automatically adjusts workflow complexity based on project nature:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    COMPLEXITY-BASED WORKFLOW TIERS                           │
└─────────────────────────────────────────────────────────────────────────────┘

PROJECT NATURE        ACTIVE WORKFLOWS                    SKIPPED
──────────────────────────────────────────────────────────────────────────────
SIMPLE                specification                       validation
                      implementation                      deep-reading
                      basepoints                          human-review
                                                         scope-detection
                                                         layer validations

MODERATE              specification                       deep-reading (partial)
                      implementation
                      basepoints
                      planning
                      detection
                      research

COMPLEX               ALL WORKFLOWS                       none
                      + layer validations
                      + comprehensive research
                      + human review checkpoints
```

### Workflow Configuration

Generated during `/deploy-agents`:

```yaml
# agent-os/workflows/workflow-config.yml

project_nature: moderate

active_categories:
  - specification
  - implementation
  - basepoints
  - planning
  - detection
  - research

simplification:
  max_research_iterations: 3
  
layer_validations:
  enabled: false  # Only for complex projects
```

### Simple Projects

For simple projects, a guide is generated:

```
agent-os/workflows/simplified-workflows.md

Recommended flow:
/adapt-to-product → /create-basepoints → /deploy-agents
       ↓
/shape-spec → /write-spec → /create-tasks → /implement-tasks

Skip: /orchestrate-tasks (use /implement-tasks directly)
```

---

## Layer Validation (NEW)

For complex projects, layer-specific validations run automatically:

| Layer | Validator | What It Checks |
|-------|-----------|----------------|
| UI | `validate-ui-patterns.md` | Component structure, naming, styling, accessibility |
| API | `validate-api-patterns.md` | Endpoint naming, HTTP methods, error handling, auth |
| Data | `validate-data-patterns.md` | Model naming, query patterns, migrations, validation |

### Validation Triggers

Layer validations run:
- After `/implement-tasks` completion
- During `/orchestrate-tasks` verification
- When running `/cleanup-agent-os`

### Comprehensive Validation Report

```
agent-os/output/validation/
├── ui-validation-results.md
├── api-validation-results.md
├── data-validation-results.md
└── comprehensive-validation-report.json
```

---

## Prompting Specialist & Learning System (NEW)

Geist includes a **Prompting Specialist** that optimizes prompts across all commands and a **Learning System** that captures feedback to continuously improve.

### Prompting Specialist

The Prompting Specialist is integrated into **ALL command flows** to construct optimal prompts:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PROMPTING SPECIALIST - CORE FLOW                          │
└─────────────────────────────────────────────────────────────────────────────┘

Every Command Execution:
├─ Step 0: Construct Optimal Prompt
│  └─ {{workflows/prompting/construct-prompt}}
│     ├─ Load basepoints context
│     ├─ Load session learnings (successful/failed patterns)
│     ├─ Load previous handoff context
│     ├─ Build context block (project info, patterns, constraints)
│     ├─ Build instruction block (clear objective, steps, boundaries)
│     └─ Build output specification (file format, handoff info)
│
└─ Final Step: Save Handoff
   └─ {{workflows/prompting/save-handoff}}
      ├─ Capture what was completed
      ├─ Highlight key decisions
      └─ Prepare context for next command
```

### Prompt Construction Process

The Prompting Specialist constructs prompts with:

1. **Context Block**
   - Project identity (tech stack, architecture from `headquarter.md`)
   - Relevant patterns (successful patterns from session learnings)
   - Constraints (anti-patterns to avoid from failed patterns)
   - Previous output (handoff context from last command)

2. **Instruction Block**
   - Clear objective (single-sentence goal)
   - Numbered steps (with expected outputs)
   - Boundaries (DO/DO NOT sections)

3. **Output Specification**
   - File path and format
   - Handoff context for next command
   - Validation criteria

### First Installation Optimization

During `/deploy-agents`, two new phases optimize prompts for the project:

**Phase 12: Optimize Prompts**
- Analyzes codebase patterns and conventions
- Generates project-specific prompt improvements
- Presents suggestions for user approval

**Phase 13: Apply Prompt Optimizations**
- Applies user-approved improvements
- Creates backup before changes
- Logs all changes for audit

### Session Feedback & Learning

After each `/implement-tasks` run, the system captures feedback:

```
agent-os/output/session-feedback/
├── current-session.md           # Active session
├── patterns/
│   ├── successful.md            # Patterns with 100% success, 3+ uses
│   └── failed.md                # Anti-patterns that caused failures
└── prompts/
    ├── effective.md             # Prompts that worked well
    └── needs-improvement.md     # Prompts that needed clarification
```

### Learning & Adaptation

During `/update-basepoints-and-redeploy`, the system learns from session feedback:

**Phase 7: Review Session Learnings**
- Extracts patterns from session data (successful/failed)
- Analyzes prompt effectiveness
- Generates adaptation suggestions
- **Human review required** - presents options:
  - [a] Apply all adaptations
  - [s] Select which to apply
  - [n] Skip (save for later)

**Phase 8: Adapt Commands**
- Applies user-approved adaptations to command templates
- Creates backup before changes
- Updates successful patterns to command context
- Adds anti-pattern warnings to constraints
- Logs all changes to `applied.md`

### Safety & Rollback

All adaptations include safety measures:

- ✅ **Human Review**: Always asks user before applying changes
- ✅ **Backup**: Creates timestamped backup before any changes
- ✅ **Rollback**: Restore from backup if issues occur
- ✅ **Change Tracking**: All changes logged in `applied.md`

### Learning Configuration

Configure learning behavior in `workflow-config.yml`:

```yaml
learning:
  enabled: true
  capture:
    implementation_outcomes: true
    validation_results: true
    patterns_used: true
    prompt_effectiveness: true
  review:
    always_ask: true              # Never skip review
    allow_selective: true         # Let user choose
    save_rejected: true           # Keep for later
  patterns:
    min_uses_for_recommendation: 3
    success_rate_threshold: 1.0   # 100% for recommendations
  safety:
    backup_before_changes: true
    require_explicit_approval: true
```

---

## Best Practices

### 1. Run Commands in Order

```bash
# Initial setup
/adapt-to-product  # or /plan-product
/create-basepoints
/deploy-agents
/cleanup-agent-os

# Then use specialized commands
/shape-spec "My feature"
/write-spec
/create-tasks
/implement-tasks  # or /orchestrate-tasks
```

### 2. Trust Automatic Detection

The system is designed to detect everything possible automatically:

- ✅ Let detection run first
- ✅ Review the confirmation prompt
- ✅ Override only if detection was incorrect
- ❌ Don't manually specify what can be detected

### 3. Use Incremental Updates

After codebase changes:

```bash
/update-basepoints-and-redeploy  # Fast, incremental
```

Not:

```bash
/create-basepoints  # Slower, full regeneration
/deploy-agents
```

### 4. Review Enriched Knowledge

Before deployment, review:

- `enriched-knowledge/security-notes.md` for CVEs
- `enriched-knowledge/version-analysis.md` for outdated deps

### 5. Use Validation Reports

Check `implementation/cache/validation-report.md` after each command.

### 6. Review Session Learnings (NEW)

Periodically run `/update-basepoints-and-redeploy` to:
- Review successful patterns discovered
- Apply adaptations to improve commands
- Learn from implementation feedback

The system will:
- Extract patterns from all `/implement-tasks` runs
- Analyze prompt effectiveness
- Suggest improvements (requires your approval)

### 7. Trust the Prompting Specialist

The Prompting Specialist automatically:
- Loads relevant context for each command
- Injects successful patterns from session learnings
- Warns against anti-patterns that caused failures
- Prepares handoff context for next command

No manual prompt construction needed.

---

## Troubleshooting

### Detection Issues

**Problem**: Detection failed or returned incorrect values

**Solution**:
1. Override in confirmation prompt
2. Check if config files exist (package.json, etc.)
3. Manually update `project-profile.yml`

### Research Issues

**Problem**: Web research failed or returned empty

**Solution**:
1. Check network connectivity
2. Research depth setting in profile
3. Add libraries to `skip_research_for` if internal

### Validation Failures

**Problem**: `validate-implementation` fails

**Solution**:
1. Check detected validation commands
2. Verify commands work manually
3. Update `validation-commands.md` if incorrect

### Knowledge Gaps

**Problem**: Basepoints missing for some modules

**Solution**:
1. Run `/cleanup-agent-os` for knowledge verification
2. Review recommendations
3. Manually create missing basepoints or re-run `/create-basepoints`

---

## File Organization

### Commands Structure

```
commands/
├── [command-name]/
│   ├── [command-name].md          # Main command file
│   ├── single-agent/              # Single-agent version
│   │   ├── [command-name].md
│   │   └── [N]-[phase-name].md    # Numbered phases
│   └── multi-agent/               # Multi-agent version (if applicable)
│       └── [command-name].md
```

#### Multi-Agent Mode Availability

Not all commands have multi-agent mode—this is **intentional**:

| Command | Single-Agent | Multi-Agent | Rationale |
|---------|-------------|-------------|-----------|
| `shape-spec` | ✅ | ✅ | Complex research benefits from delegation |
| `write-spec` | ✅ | ✅ | Spec writing can delegate sections |
| `create-tasks` | ✅ | ✅ | Task breakdown can be parallelized |
| `implement-tasks` | ✅ | ✅ | Multi-agent orchestration for complex implementations |
| `orchestrate-tasks` | ✅ | N/A | Already multi-agent by design |
| `plan-product` | ✅ | ✅ | Product planning benefits from delegation |
| `create-basepoints` | ✅ | ✅ | Module analysis can be parallelized |
| `adapt-to-product` | ✅ | ❌ | Sequential detection, no benefit from delegation |
| `deploy-agents` | ✅ | ❌ | Sequential specialization, order matters |
| `cleanup-agent-os` | ✅ | ❌ | Validation requires sequential checks |
| `update-basepoints-and-redeploy` | ✅ | ❌ | Sequential update process |

**Rule of thumb**: Setup/specialization commands are single-agent only. Development commands support multi-agent for complex work.

### Workflows Structure

```
workflows/
├── [category]/
│   └── [workflow-name].md
```

### Standards Structure

```
standards/
└── global/
    ├── conventions.md
    ├── tech-stack.md
    ├── project-profile-schema.md       # NEW
    ├── enriched-knowledge-templates.md # NEW
    └── validation-commands.md          # NEW
```

---

## License

[Add your license information here]

---

**Last Updated**: 2026-01-16

**New Features Added**:
- Adaptive Questionnaire System (automatic detection, minimal questions)
- Web Research & Knowledge Enrichment
- Basepoints Knowledge Integration
- Deterministic Validation with project-specific commands
- Human Alignment on trade-offs and contradictions
