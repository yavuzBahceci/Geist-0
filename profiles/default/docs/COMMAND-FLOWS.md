# Command Flows Reference

This document provides detailed documentation of all commands in the Geist system, including their phases, inputs, outputs, and integration points.

---

## Table of Contents

- [Specialization Commands](#specialization-commands)
  - [adapt-to-product](#1-adapt-to-product)
  - [plan-product](#15-plan-product)
  - [create-basepoints](#2-create-basepoints)
  - [deploy-agents](#3-deploy-agents)
  - [cleanup-geist](#4-cleanup-geist)
  - [update-basepoints-and-redeploy](#5-update-basepoints-and-redeploy)
- [Development Commands](#development-commands)
  - [shape-spec](#6-shape-spec)
  - [write-spec](#7-write-spec)
  - [create-tasks](#8-create-tasks)
  - [implement-tasks](#9-implement-tasks)
  - [orchestrate-tasks](#10-orchestrate-tasks)
  - [fix-bug](#11-fix-bug)
- [Workflows](#workflows)
  - [Product-Focused Cleanup](#product-focused-cleanup-workflow)
- [Command Cycle Flow](#command-cycle-flow)
- [Knowledge Integration](#knowledge-integration)

---

## Specialization Commands

These commands are used during initial setup to create and specialize your Geist instance.

---

### 1. adapt-to-product

**Purpose**: Extract product information from an existing codebase with automatic detection.

**Location**: `commands/adapt-to-product/`

**Prerequisites**: None

**Outputs**:
- `geist/product/mission.md`
- `geist/product/roadmap.md`
- `geist/product/tech-stack.md`
- `geist/config/project-profile.yml`
- `geist/config/enriched-knowledge/`
- `geist/output/product-cleanup/detected-scope.yml` (NEW)
- `geist/output/product-cleanup/search-queries.md` (NEW)
- `geist/output/product-cleanup/cleanup-report.md` (NEW)

#### Phases

```
Phase 1: Setup & Information Gathering (Enhanced)
├── Step 0.1: Automatic Detection
│   └── {{workflows/detection/detect-project-profile}}
│       ├── detect-tech-stack.md
│       │   └── Parse: package.json, Cargo.toml, go.mod, requirements.txt
│       ├── detect-commands.md
│       │   └── Extract: build, test, lint from scripts/Makefile/CI
│       ├── detect-architecture.md
│       │   └── Analyze: directory structure, module boundaries
│       └── detect-security-level.md
│           └── Check: auth deps, secrets management, SSL
│
├── Step 0.2: Web Research
│   └── {{workflows/research/research-orchestrator}}
│       ├── research-library.md
│       │   └── Search: "[library] best practices", "known issues", "CVE"
│       ├── research-stack-patterns.md
│       │   └── Search: "[stack] architecture patterns"
│       ├── research-security.md
│       │   └── Search: "[dep] vulnerabilities"
│       └── version-analysis.md
│           └── Compare: detected vs latest versions
│
├── Step 0.3: Present & Confirm
│   └── {{workflows/detection/present-and-confirm}}
│       └── Show: detected profile, allow overrides
│
├── Step 0.4: Minimal Questions
│   └── {{workflows/detection/question-templates}}
│       └── Ask ONLY:
│           1. Compliance requirements? [None/SOC2/HIPAA/GDPR]
│           2. Human review preference? [Minimal/Moderate/High]
│
└── Step 0.5: Store Profile
    └── Save to: geist/config/project-profile.yml
                 geist/config/enriched-knowledge/

Phase 2: Analyze Codebase
└── Analyze README, structure, patterns

Phase 3: Create Mission
└── Generate: geist/product/mission.md

Phase 4: Create Roadmap
└── Generate: geist/product/roadmap.md

Phase 5: Create Tech Stack
└── Generate: geist/product/tech-stack.md

Phase 6: Review & Combine Knowledge
└── Verify consistency, combine into unified knowledge

Phase 7: Product-Focused Cleanup (NEW)
├── Step 1: Load product scope detection sources
│   └── Read: tech-stack.md, mission.md, roadmap.md
├── Step 2: Parse and categorize detected scope
│   └── Detect: language, framework, project type, architecture
├── Step 3: Phase A - Simplify
│   └── Remove: irrelevant tech examples, unused workflows
├── Step 4: Phase B - Expand
│   └── Add: tech-specific patterns, web-validated best practices
├── Step 5: Web search for trusted information
│   └── Validate: 2+ sources required
└── Step 6: Generate cleanup report
    └── Output: geist/output/product-cleanup/

Phase 8: Navigate to Next Command
└── Display: "NEXT STEP: Run /create-basepoints"
```

#### Detection Details

**What Gets Detected Automatically**:

| Category | Detection Method | Confidence |
|----------|-----------------|------------|
| Language | Config files (package.json, Cargo.toml, etc.) | High |
| Framework | Dependencies analysis | High |
| Build command | package.json scripts, Makefile, CI configs | High |
| Test command | Test configs, CI pipelines | High |
| Lint command | .eslintrc, Makefile, package.json | High |
| Security level | Auth deps, secrets management | Medium |
| Complexity | File count, module count | High |
| Architecture | Directory structure analysis | Medium |

**Questions Asked (Max 2-3)**:

| Question | Why Asked | Can't Detect From |
|----------|-----------|-------------------|
| Compliance requirements | Regulatory, not in code | Code analysis |
| Human review preference | Subjective user choice | Code analysis |
| (Only if ambiguous) | Detection confidence < 80% | Varies |

---

### 1.5. plan-product

**Purpose**: Plan and document the mission, roadmap, and tech stack for a new product (no existing codebase).

**Location**: `commands/plan-product/`

**Prerequisites**: None (for new projects without existing codebase)

**Outputs**:
- `geist/product/mission.md`
- `geist/product/roadmap.md`
- `geist/product/tech-stack.md`
- `geist/output/product-cleanup/detected-scope.yml`
- `geist/output/product-cleanup/search-queries.md`
- `geist/output/product-cleanup/cleanup-report.md`

#### Phases

```
Phase 1: Product Concept
├── Gather product vision
├── Define target users
├── Identify problems to solve
└── List key features

Phase 2: Create Mission
└── Generate: geist/product/mission.md

Phase 3: Create Roadmap
└── Generate: geist/product/roadmap.md

Phase 4: Create Tech Stack
└── Generate: geist/product/tech-stack.md

Phase 5: Product-Focused Cleanup (NEW)
├── Step 1: Load product scope detection sources
│   └── Read: tech-stack.md, mission.md, roadmap.md
├── Step 2: Parse and categorize detected scope
│   └── Detect: language, framework, project type, architecture
├── Step 3: Phase A - Simplify
│   └── Remove: irrelevant tech examples, unused workflows
├── Step 4: Phase B - Expand
│   └── Add: tech-specific patterns, web-validated best practices
├── Step 5: Web search for trusted information
│   └── Validate: 2+ sources required
└── Step 6: Generate cleanup report
    └── Output: geist/output/product-cleanup/
```

#### When to Use

| Scenario | Command |
|----------|---------|
| Starting new project from scratch | `/plan-product` |
| Have existing codebase | `/adapt-to-product` |

---

### 2. create-basepoints

**Purpose**: Create comprehensive codebase documentation at all abstraction layers.

**Location**: `commands/create-basepoints/`

**Prerequisites**: Product files (`mission.md`, `roadmap.md`, `tech-stack.md`)

**Outputs**:
- `geist/basepoints/headquarter.md`
- `geist/basepoints/[layer]/[module]/agent-base-[module].md`

#### Phases

```
Phase 1: Validate Prerequisites (Enhanced)
├── Step 0.1: Load Project Profile
│   └── if [ -f "geist/config/project-profile.yml" ]; then
│       Load existing profile (no re-detection)
│   └── else
│       Run detection workflow
│
├── Step 0.2: Architecture Research (if not done)
│   └── {{workflows/research/research-orchestrator}}
│       └── Research: architecture patterns for detected stack
│
├── Step 0.3: Ask Only if Unclear
│   └── Only if module detection confidence < 80%
│
└── Validate: product files exist

Phase 2: Detect Abstraction Layers
└── Analyze: directory structure, identify layers

Phase 3: Mirror Project Structure
└── Create: basepoints directory structure

Phase 4: Analyze Codebase
└── Deep analysis: patterns, standards, flows

Phase 5: Generate Module Basepoints
└── For each module:
    └── Create: agent-base-[module].md
        ├── Patterns section
        ├── Standards section
        ├── Flows section
        ├── Strategies section
        └── Testing section

Phase 6: Generate Parent Basepoints
└── Aggregate: child basepoints into parent

Phase 7: Generate Headquarter
└── Create: headquarter.md (project overview)

Phase 8: Generate Library Basepoints (NEW)
├── Validate prerequisites (tech-stack.md, headquarter.md)
└── {{workflows/codebase-analysis/generate-library-basepoints}}
    ├── Load product knowledge (tech-stack.md)
    ├── Extract library usage from module basepoints
    ├── Analyze codebase for implementation patterns
    ├── Research official documentation (depth by importance)
    ├── Classify importance (critical, important, supporting)
    ├── Categorize (data, domain, util, infrastructure, framework)
    ├── Generate library basepoint files
    ├── Detect solution-specific patterns
    └── Create library index
```

#### Knowledge Extracted

| Section | Contents | Usage |
|---------|----------|-------|
| Patterns | Design, coding, architectural patterns | Reused in implementations |
| Standards | Naming, coding style, structure | Enforced in code reviews |
| Flows | Data, control, dependency flows | Understood for changes |
| Strategies | Implementation, architectural strategies | Guide decisions |
| Testing | Testing approaches, coverage | Guide test writing |

---

### 3. deploy-agents

**Purpose**: Specialize the installed Geist for this specific project by extracting knowledge and applying it to standards and agents.

**Location**: `commands/deploy-agents/`

**Prerequisites**: 
- Basepoints (`headquarter.md`, module basepoints)
- Product files (`mission.md`, `roadmap.md`, `tech-stack.md`)

**Outputs**:
- Specialized standards in `geist/standards/` (project-specific patterns)
- Specialist agents in `geist/agents/specialists/` (layer-specific agents)
- Specialist registry in `geist/agents/specialists/registry.yml`
- Knowledge files in `geist/output/deploy-agents/knowledge/`
- Reports in `geist/output/deploy-agents/reports/`
- Navigation guidance to `/cleanup-geist`

#### Phases (Simplified - 7 phases)

```
Phase 1: Validate Prerequisites
├── Check: geist directory exists
├── Check: basepoints exist (headquarter.md + module basepoints)
├── Check: product files exist (mission.md, roadmap.md, tech-stack.md)
└── Load: project-profile.yml

Phase 2: Extract Basepoints Knowledge
├── Read: headquarter.md
├── Read: all agent-base-*.md files
├── Extract: patterns, standards, flows, strategies
└── Output: basepoints-knowledge.json, basepoints-knowledge-summary.md

Phase 3: Extract Product Knowledge
├── Read: mission.md, roadmap.md, tech-stack.md
├── Extract: mission, features, tech stack
└── Output: product-knowledge.json, product-knowledge-summary.md

Phase 4: Merge Knowledge
├── Combine: basepoints + product knowledge
├── Detect: conflicts between sources
├── Resolve: conflicts (user interaction if needed)
└── Output: merged-knowledge.md

Phase 5: Specialize Standards
├── Read: extracted knowledge
├── Update: coding-style.md, conventions.md, error-handling.md, etc.
├── Add: project-specific patterns and examples
├── Create: new standards if needed (e.g., template-syntax.md)
└── Output: standards-specialization.md report

Phase 6: Specialize Agents
├── Read: abstraction layers from basepoints
├── Create: specialist agent per layer (e.g., ui-specialist, api-specialist)
├── Create: registry.yml with collaboration hints
└── Output: agents-specialization.md report

Phase 7: Navigate to Cleanup
├── Display: summary of accomplishments
├── List: specialized files
└── Recommend: /cleanup-geist
```

#### Key Design Decisions

1. **No command specialization**: Commands use `@geist/` references that resolve at runtime. No need to rewrite command files.

2. **No workflow specialization**: Workflows are reusable building blocks that stay generic. Specialization happens at the standards level.

3. **Standards are the specialization point**: Project patterns go into standards, which commands/workflows reference.

4. **Specialist agents**: Created based on abstraction layers detected in basepoints. Support multiple specialists per task for cross-layer work.

#### What's Next

After `/deploy-agents` completes, run `/cleanup-geist` to:
- Verify all placeholders are properly replaced
- Check for broken file references
- Ensure knowledge completeness
- Generate a cleanup report

#### Specialization Details

**Placeholder Replacement**:

| Placeholder | Replaced With | Source |
|------------|--------------|--------|
| `{{BASEPOINTS_PATH}}` | `geist/basepoints` | Convention |
| `{{PROJECT_BUILD_COMMAND}}` | `npm run build` | Detection |
| `{{PROJECT_TEST_COMMAND}}` | `npm test` | Detection |
| `{{PROJECT_LINT_COMMAND}}` | `npm run lint` | Detection |
| `{{workflows/validation/...}}` | Actual workflow path | Convention |

---

### 4. cleanup-geist

**Purpose**: Clean up and verify knowledge completeness in deployed Geist.

**Location**: `commands/cleanup-geist/`

**Prerequisites**: Deployed Geist

**Outputs**:
- Cleaned commands (no placeholders)
- Knowledge verification report
- Recommendations

#### Phases

```
Phase 1: Validate & Run Validation
├── Check: geist deployment
└── Run: comprehensive validation

Phase 2: Clean Placeholders
└── Replace: remaining {{PLACEHOLDER}} with values

Phase 3: Remove Unnecessary Logic
└── Remove: project-agnostic conditionals

Phase 4: Fix Broken References
└── Fix: @geist/ references

Phase 5: Verify Knowledge Completeness
├── Verify: basepoints completeness
│   ├── headquarter.md exists
│   ├── module basepoints cover all layers
│   └── required sections present
├── Verify: product knowledge
│   ├── mission.md, roadmap.md, tech-stack.md
│   └── content completeness
├── Detect: missing information
└── Coverage: analysis

Phase 6: Generate Report
├── Statistics
├── Recommendations
└── Knowledge gaps
```

---

### 5. update-basepoints-and-redeploy

**Purpose**: Incrementally update basepoints after codebase changes.

**Location**: `commands/update-basepoints-and-redeploy/`

**Prerequisites**: Deployed Geist

**Outputs**:
- Updated basepoints
- Re-specialized commands
- Update report

#### Phases

```
Phase 1: Detect Changes
├── Use: git diff (preferred) or timestamps
├── Categorize: added, modified, deleted
└── Filter: irrelevant files

Phase 2: Identify Affected Basepoints
├── Map: changed files to basepoints
└── Calculate: parent propagation

Phase 3: Update Basepoints
├── Process: children before parents
├── Create: backups
└── Merge: new content

Phase 4: Re-extract Knowledge
├── Load: existing cache
└── Extract: from updated only

Phase 5: Re-specialize Commands
├── Re-specialize: all 5 core commands
└── Update: standards, workflows, agents

Phase 6: Validate & Report
├── Validate: all updates
└── Generate: report
```

---

## Development Commands

These commands are used during feature development after Geist is specialized.

---

### 6. shape-spec

**Purpose**: Research and shape a new feature specification.

**Location**: `commands/shape-spec/`

**Prerequisites**: Specialized Geist

**Outputs**:
- `geist/specs/[spec-name]/planning/requirements.md`
- `geist/specs/[spec-name]/planning/initialization.md`
- `geist/specs/[spec-name]/implementation/cache/basepoints-knowledge.md`
- `geist/specs/[spec-name]/implementation/cache/detected-layer.txt`

#### Phases

```
Phase 1: Initialize Spec
├── Create: spec folder structure
│   ├── planning/
│   ├── planning/visuals/
│   └── implementation/cache/
└── Store: feature description in initialization.md

Phase 2: Shape Spec (Enhanced)
├── Step 1: Extract Basepoints Knowledge
│   └── {{workflows/basepoints/extract-basepoints-knowledge-automatic}}
│       ├── Read: headquarter.md
│       ├── Detect: abstraction layer
│       ├── Find: relevant module basepoints
│       └── Extract: Patterns, Standards, Flows, Strategies
│
├── Step 2: Detect Abstraction Layer
│   └── {{workflows/scope-detection/detect-abstraction-layer}}
│       └── Store: detected-layer.txt
│
├── Step 3: Inject Knowledge into Questions
│   └── Use: extracted patterns to suggest approaches
│
├── Step 4: Gather Requirements
│   └── Q&A: with user
│
├── Step 5: Suggest Reusable Modules
│   └── Based on: basepoints patterns
│
├── Step 6: Validate
│   └── {{workflows/validation/validate-output-exists}}
│
└── Step 7: Generate Report
    └── {{workflows/validation/generate-validation-report}}
```

#### Knowledge Integration

```
┌─────────────────────────────────────────────────────────────────┐
│                    SHAPE-SPEC KNOWLEDGE FLOW                     │
└─────────────────────────────────────────────────────────────────┘

Feature Description
        │
        ▼
┌─────────────────┐
│ Detect Layer    │ → detected-layer.txt (e.g., "PROFILES")
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Extract         │ → basepoints-knowledge.md
│ Knowledge       │   • Relevant patterns
└────────┬────────┘   • Applicable standards
         │            • Related flows
         ▼            • Suggested strategies
┌─────────────────┐
│ Generate        │ → requirements.md
│ Requirements    │   (informed by patterns)
└─────────────────┘
```

---

### 7. write-spec

**Purpose**: Write detailed specification from requirements.

**Location**: `commands/write-spec/`

**Prerequisites**: `requirements.md` (from shape-spec)

**Outputs**:
- `geist/specs/[spec-name]/spec.md`
- `geist/specs/[spec-name]/implementation/cache/resources-consulted.md`

#### Flow

```
├── Step 1: Load Knowledge Cache
│   └── Read: basepoints-knowledge.md, detected-layer.txt
│
├── Step 2: Reference Standards
│   └── Include: applicable standards in spec
│
├── Step 3: Suggest Existing Code
│   └── Based on: patterns in basepoints
│
├── Step 4: Write Specification
│   └── Generate: spec.md
│
├── Step 5: Detect Trade-offs
│   └── {{workflows/human-review/review-trade-offs}}
│
├── Step 6: Validate
│   └── {{workflows/validation/validate-output-exists}}
│
└── Step 7: Generate Resources List
    └── Create: resources-consulted.md
```

---

### 8. create-tasks

**Purpose**: Break specification into actionable tasks.

**Location**: `commands/create-tasks/`

**Prerequisites**: `spec.md` (from write-spec)

**Outputs**:
- `geist/specs/[spec-name]/tasks.md`

#### Flow

```
├── Step 1: Load Knowledge
│   └── Read: basepoints-knowledge.md
│
├── Step 2: Analyze Spec
│   └── Parse: spec.md for requirements
│
├── Step 3: Create Task Groups
│   └── Organize: by feature area or component
│
├── Step 4: Include Implementation Hints
│   └── From: patterns in basepoints
│
├── Step 5: Reference Strategies
│   └── In: task descriptions
│
├── Step 6: Add Acceptance Criteria
│   └── For: each task
│
├── Step 7: Validate
│   └── {{workflows/validation/validate-output-exists}}
│
└── Step 8: Update Report
    └── Update: validation-report.md
```

---

### 9. implement-tasks

**Purpose**: Implement tasks from task list.

**Location**: `commands/implement-tasks/`

**Prerequisites**: `tasks.md` (from create-tasks)

**Outputs**:
- Code changes
- `implementation/cache/validation-report.md`

#### Flow

```
Phase 1: Determine Tasks
└── Parse: tasks.md for uncompleted tasks

Phase 2: Implement Tasks (Enhanced)
├── Step 1: Load Patterns
│   └── Read: module-specific patterns from cache
│
├── Step 2: Provide Context
│   └── Inject: coding patterns into implementation
│
├── Step 3: Reference Standards
│   └── Apply: code style from standards
│
├── Step 4: Implement
│   └── Write: code changes
│
├── Step 5: Validate Implementation
│   └── {{workflows/validation/validate-implementation}}
│       ├── Run: {{PROJECT_BUILD_COMMAND}}
│       ├── Run: {{PROJECT_TEST_COMMAND}}
│       ├── Run: {{PROJECT_LINT_COMMAND}}
│       └── Run: {{PROJECT_TYPECHECK_COMMAND}}
│
└── Step 6: Mark Complete
    └── Update: tasks.md with [x]

Phase 3: Verify Implementation
└── Final validation and testing
```

---

### 10. orchestrate-tasks

**Purpose**: Coordinate task implementation with specialist assignment.

**Location**: `commands/orchestrate-tasks/`

**Prerequisites**: `tasks.md` (from create-tasks)

**Outputs**:
- `geist/specs/[spec-name]/orchestration.yml` (with specialist assignments)
- `geist/specs/[spec-name]/implementation/prompts/[N]-[task-group].md`

#### Flow

```
├── Step 1: Extract Basepoints Knowledge
│   └── Match: tasks to relevant module basepoints
│
├── Step 2: Create Task Groups
│   └── Parse: tasks.md for task groups
│
├── Step 3: Auto-detect Layer Specialists
│   └── For each task group:
│       ├── Analyze: task keywords
│       ├── Detect: abstraction layer(s)
│       └── Suggest: specialist(s)
│
├── Step 4: User Confirmation
│   └── Present: suggested specialists
│       ├── Accept all
│       ├── Modify some
│       └── Assign manually
│
├── Step 5: Generate Orchestration
│   └── Create: orchestration.yml with specialists
│
├── Step 6: Generate Prompts (single-agent mode)
│   └── For each group:
│       Create: [N]-[task-group].md
│       ├── Task description
│       ├── Context references
│       ├── Basepoints knowledge injection
│       ├── Specialist context (if assigned)
│       ├── Standards compliance
│       ├── Validation step
│       └── Auto-proceed to next prompt
│
└── Step 7: Validate
    └── Update: validation-report.md
```

#### Multi-Specialist Support

Task groups can have **multiple specialists** assigned for cross-layer work:

```yaml
# orchestration.yml
task_groups:
  - name: user-profile-ui
    specialists: [ui-specialist]
    detected_layers: [ui]
    
  - name: user-dashboard-feature
    specialists: [ui-specialist, api-specialist]  # Cross-layer
    detected_layers: [ui, api]
    
  - name: database-migration
    specialists: [data-specialist]
    detected_layers: [data]
```

When multiple specialists are assigned:
- Context from ALL specialists is loaded
- Combined expertise guides implementation
- Works in both Cursor (single-agent) and Claude Code (multi-agent)

#### Prompt Template

```markdown
# Task Group [N]: [Title]

## Task to Implement
[Task description and subtasks]

## Context
- @geist/specs/[spec]/spec.md
- @geist/specs/[spec]/planning/requirements.md

## Specialist Knowledge (if assigned)
[Context from assigned specialist(s)]

## Basepoints Knowledge Context
[Extracted patterns and standards relevant to this task group]

## Implementation Instructions
@geist/workflows/implementation/implement-tasks.md

## Standards Compliance
@geist/standards/global/*

## ✅ Completion and Validation

### Step 1: Run Implementation Validation
SPEC_PATH="geist/specs/[spec]"
@geist/workflows/validation/validate-implementation.md

### Step 2: Mark Completion
- Mark tasks complete [x] in tasks.md
- Add ✅ marker to Task Group [N]

### Step 3: Check Human Review
@geist/workflows/human-review/review-trade-offs.md

### Step 4: Proceed to Next Prompt
- If validation passes and no review needed → auto-proceed
- **Automatically proceeding to Prompt [N+1]**
```

---

### 11. fix-bug

**Purpose**: Analyze and fix bugs or implement feedback through systematic knowledge extraction and iterative fixing.

**Location**: `commands/fix-bug/`

**Prerequisites**: Deployed Geist (for basepoints integration)

**Inputs**:
- Bug reports (error logs, stack traces, error codes)
- Feedback (feature requests, enhancement suggestions)

**Outputs**:
- `geist/output/fix-bug/cache/issue-analysis.md`
- `geist/output/fix-bug/cache/library-research.md`
- `geist/output/fix-bug/cache/basepoints-integration.md`
- `geist/output/fix-bug/cache/code-analysis.md`
- `geist/output/fix-bug/cache/knowledge-synthesis.md`
- `geist/output/fix-bug/cache/fix-report.md` (on success)
- `geist/output/fix-bug/cache/guidance-request.md` (on stop condition)

#### Phases

```
Phase 1: Issue Analysis
├── Identify input type (bug/feedback)
├── Extract details from error logs, codes, descriptions
├── Identify affected libraries and modules
└── Create issue analysis document

Phase 2: Library Research
├── Research internal architecture of affected libraries
├── Research known issues and bug patterns
├── Research error scenarios and recovery strategies
└── Create library research document

Phase 3: Basepoints Integration
├── Extract basepoints knowledge
│   └── {{workflows/common/extract-basepoints-with-scope-detection}}
├── Find basepoints describing error location
├── Extract patterns and standards related to error context
├── Identify similar issues in basepoints
└── Create basepoints integration document

Phase 4: Code Analysis
├── Identify exact file/module locations from error logs
├── Deep-dive into relevant code files
├── Analyze code patterns and flows in error context
├── Trace execution paths leading to error
└── Create code analysis document

Phase 5: Knowledge Synthesis
├── Combine all knowledge sources
├── Create unified view of issue context
├── Prioritize fix approaches
├── Prepare knowledge for fix implementation
└── Create knowledge synthesis document

Phase 6: Iterative Fix Implementation
├── Implement initial fix using synthesized knowledge
├── Apply library-specific patterns and best practices
├── Follow basepoints patterns and standards
├── Run validation after each fix attempt
├── Iterative refinement loop:
│   ├── If getting closer → continue iterating
│   ├── If worsening → track counter
│   └── Stop after 3 worsening results
└── Output:
    ├── On success → fix-report.md
    └── On stop condition → guidance-request.md
```

#### Iterative Fix Behavior

| Condition | Action |
|-----------|--------|
| Error count decreases | Continue iterating |
| Error count same/increases | Increment worsening counter |
| 3 worsening results | Stop and request guidance |
| Error resolved | Generate success report |

---

## Workflows

### Product-Focused Cleanup Workflow

**Purpose**: Clean irrelevant content and enhance geist files based on detected product scope.

**Location**: `workflows/cleanup/product-focused-cleanup.md`

**Called By**:
- `/adapt-to-product` (Phase 7)
- `/plan-product` (Phase 5)

**Inputs**:
- `geist/product/tech-stack.md` - Language, framework detection
- `geist/product/mission.md` - Project type detection
- `geist/product/roadmap.md` - Architecture detection

**Outputs**:
- `geist/output/product-cleanup/detected-scope.yml`
- `geist/output/product-cleanup/search-queries.md`
- `geist/output/product-cleanup/cleanup-report.md`

#### Workflow Steps

```
Step 1: Load Product Scope Detection Sources
├── Read tech-stack.md for language/framework
├── Read mission.md for project type
└── Read roadmap.md for architecture

Step 2: Parse and Categorize Detected Scope
├── detect_language() → typescript, python, rust, go, etc.
├── detect_frameworks() → react, express, fastapi, etc.
├── detect_project_type() → cli, api, webapp, mobile, library, ai-service
└── detect_architecture() → monolith, microservices, serverless

Step 3: Phase A - Simplify
├── Define removal rules based on scope
├── Process geist/commands/
├── Process geist/workflows/
├── Process geist/standards/
└── Process geist/agents/

Step 4: Phase B - Expand
├── Define enhancement rules based on scope
├── Add technology-specific examples
├── Generate patterns based on best practices
└── Inject project-specific terminology

Step 5: Web Search for Trusted Information
├── Generate targeted queries for detected stack
├── Validate: require 2+ independent sources
├── Prioritize: Official docs > Blogs > Stack Overflow
└── Include version-specific guidance

Step 6: Generate Cleanup Report
├── List files modified
├── List content removed (with reasons)
├── List content added (with sources)
└── Save to geist/output/product-cleanup/
```

#### Scope Detection Criteria

| Criterion | Detection Source | Example Values |
|-----------|------------------|----------------|
| Language | tech-stack.md | typescript, python, rust, go |
| Framework | tech-stack.md | react, express, fastapi, rails |
| Project Type | mission.md | cli, api, webapp, mobile, library |
| Architecture | tech-stack.md, roadmap.md | monolith, microservices, serverless |

#### Example Cleanup Scenarios

| Project Type | Remove | Enhance |
|--------------|--------|---------|
| TypeScript CLI | UI workflows, frontend patterns | CLI patterns, Node.js best practices |
| Python REST API | Frontend workflows, TypeScript examples | Python patterns, FastAPI examples |
| Full-Stack React + Node | Mobile patterns | React + Node.js patterns |

---

## Command Cycle Flow

The complete development cycle from spec to implementation:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         COMMAND CYCLE FLOW                                   │
└─────────────────────────────────────────────────────────────────────────────┘

    ┌──────────────┐
    │  shape-spec  │  "Add user authentication"
    │              │
    │  Extracts:   │  • Basepoints knowledge
    │              │  • Abstraction layer
    │              │  • Relevant patterns
    └──────┬───────┘
           │
           │ Outputs: requirements.md, initialization.md
           │          cache/basepoints-knowledge.md
           │          cache/detected-layer.txt
           │
    ┌──────▼───────┐
    │  write-spec  │
    │              │
    │  Uses:       │  • Extracted knowledge
    │              │  • Standards reference
    │              │  • Trade-off detection
    └──────┬───────┘
           │
           │ Outputs: spec.md
           │          cache/resources-consulted.md
           │
    ┌──────▼───────┐
    │ create-tasks │
    │              │
    │  Includes:   │  • Implementation hints
    │              │  • Strategy references
    │              │  • Acceptance criteria
    └──────┬───────┘
           │
           │ Outputs: tasks.md
           │
           ├─────────────────────────────────┐
           │                                 │
    ┌──────▼───────┐                  ┌──────▼────────────┐
    │ implement-   │                  │ orchestrate-      │
    │ tasks        │                  │ tasks             │
    │              │                  │                   │
    │  Validates:  │                  │  Generates:       │
    │  • Build     │                  │  • orchestration  │
    │  • Test      │                  │  • prompts/       │
    │  • Lint      │                  │  • with knowledge │
    └──────────────┘                  └───────────────────┘
```

---

## Knowledge Integration

### Context Enrichment Strategy

All spec/implementation commands follow a consistent "narrow focus + expand knowledge" strategy:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    CONTEXT ENRICHMENT STRATEGY                               │
└─────────────────────────────────────────────────────────────────────────────┘

For each command:
1. NARROW FOCUS: Scope to specific task (spec, task group, feature)
2. EXPAND KNOWLEDGE: Extract from multiple sources:
   ├── Basepoints knowledge
   ├── Library basepoints knowledge (NEW)
   ├── Product documentation
   └── Codebase navigation

3. ACCUMULATE: Each command builds upon previous command's knowledge
   shape-spec → write-spec → create-tasks → orchestrate-tasks → implement-tasks
```

### Library Basepoints Knowledge Extraction

A new workflow extracts knowledge from library basepoints:

```
{{workflows/common/extract-library-basepoints-knowledge}}

Extracts from: geist/basepoints/libraries/
├── data/           - Data access, databases, ORM
├── domain/         - Domain logic, business rules
├── util/           - Utilities, helpers
├── infrastructure/ - Networking, HTTP, threading
└── framework/      - Framework components, UI

Knowledge extracted:
├── Library patterns and workflows
├── Best practices from official documentation
├── Troubleshooting guidance
└── Library boundaries (what is/isn't used)
```

### Knowledge Accumulation

Knowledge accumulates across commands using:

```
{{workflows/common/accumulate-knowledge}}

Stored in: $SPEC_PATH/implementation/cache/accumulated-knowledge.md

Each command:
1. Loads previous accumulated knowledge
2. Extracts fresh knowledge
3. Combines and stores for next command
```

### Knowledge Sources

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         KNOWLEDGE SOURCES                                    │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   DETECTION     │     │    RESEARCH     │     │   BASEPOINTS    │
│                 │     │                 │     │                 │
│ project-profile │     │ library-research│     │ headquarter.md  │
│ .yml            │     │ .md             │     │                 │
│                 │     │                 │     │ agent-base-     │
│ • tech stack    │     │ stack-best-     │     │ *.md            │
│ • commands      │     │ practices.md    │     │                 │
│ • security      │     │                 │     │ • Patterns      │
│ • complexity    │     │ security-notes  │     │ • Standards     │
│                 │     │ .md             │     │ • Flows         │
│                 │     │                 │     │ • Strategies    │
│                 │     │ version-analysis│     │                 │
│                 │     │ .md             │     │                 │
└────────┬────────┘     └────────┬────────┘     └────────┬────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                                 ▼
                    ┌─────────────────────┐
                    │   UNIFIED KNOWLEDGE │
                    │                     │
                    │ Used by:            │
                    │ • shape-spec        │
                    │ • write-spec        │
                    │ • create-tasks      │
                    │ • implement-tasks   │
                    │ • orchestrate-tasks │
                    └─────────────────────┘
```

### Knowledge Usage by Command

| Command | Detection | Research | Basepoints | Library Basepoints | Product | Accumulates |
|---------|-----------|----------|------------|-------------------|---------|-------------|
| adapt-to-product | ✅ Creates | ✅ Creates | - | - | ✅ Creates | - |
| create-basepoints | ✅ Loads | ✅ Adds | ✅ Creates | ✅ Creates | ✅ Uses | - |
| deploy-agents | ✅ Uses | ✅ Uses | ✅ Uses | ✅ Uses | ✅ Uses | - |
| shape-spec | - | - | ✅ Extracts | ✅ Extracts | ✅ Uses | ✅ Yes |
| write-spec | - | - | ✅ Uses | ✅ Uses | ✅ Uses | ✅ Yes |
| create-tasks | - | - | ✅ Uses | ✅ Uses | ✅ Uses | ✅ Yes |
| implement-tasks | - | - | ✅ Uses | ✅ Uses | ✅ Uses | ✅ Yes |
| orchestrate-tasks | - | - | ✅ Injects | ✅ Injects | - | - |
| fix-bug | - | ✅ Creates | ✅ Uses | ✅ Uses | - | - |

---

## Validation Integration

### Validation at Each Stage

| Command | Validation Run | Exit on Failure |
|---------|---------------|-----------------|
| shape-spec | `validate-output-exists` | Warning |
| write-spec | `validate-output-exists`, `review-trade-offs` | Warning |
| create-tasks | `validate-output-exists` | Warning |
| implement-tasks | `validate-implementation` | Error |
| orchestrate-tasks | `validate-output-exists` | Warning |

### validate-implementation Details

```bash
# Runs project-specific validation
{{PROJECT_BUILD_COMMAND}}    # e.g., npm run build
{{PROJECT_TEST_COMMAND}}     # e.g., npm test
{{PROJECT_LINT_COMMAND}}     # e.g., npm run lint
{{PROJECT_TYPECHECK_COMMAND}}# e.g., tsc --noEmit

# Returns exit code 0 (pass) or 1 (fail)
# Generates: implementation-validation-report.md
```

---

*Last Updated: 2026-01-16*
