# Knowledge Systems Reference

This document explains the two knowledge systems in Geist that contain library and technology information, when each is created, and how they work together.

---

## Table of Contents

- [Overview](#overview)
- [System 1: Enriched Knowledge](#system-1-enriched-knowledge)
- [System 2: Library Basepoints](#system-2-library-basepoints)
- [Comparison](#comparison)
- [Timeline: When Each Is Created](#timeline-when-each-is-created)
- [Usage: When Each Is Used](#usage-when-each-is-used)
- [Merge Strategy](#merge-strategy)
- [Visual Data Flow](#visual-data-flow)

---

## Overview

Geist maintains two complementary knowledge systems for library information:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         TWO KNOWLEDGE SYSTEMS                                │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────┐     ┌─────────────────────────────────────┐
│  config/enriched-knowledge/     │     │  basepoints/libraries/              │
│  ═══════════════════════════    │     │  ═══════════════════════════        │
│                                 │     │                                     │
│  Created: EARLY (adapt-to-     │     │  Created: LATER (create-basepoints) │
│           product)              │     │                                     │
│                                 │     │                                     │
│  Purpose: Generic research      │     │  Purpose: Project-specific          │
│           about libraries       │     │           library knowledge         │
│                                 │     │                                     │
│  Content:                       │     │  Content:                           │
│  • Security vulnerabilities     │     │  • How YOUR project uses it         │
│  • Version analysis             │     │  • YOUR patterns and boundaries     │
│  • Generic best practices       │     │  • YOUR troubleshooting context     │
│  • Known issues from web        │     │  • Combined with official docs      │
│                                 │     │                                     │
│  Used by: deploy-agents         │     │  Used by: spec/implementation       │
│           (specialization)      │     │           commands                  │
└─────────────────────────────────┘     └─────────────────────────────────────┘
```

**Key Insight**: These systems are complementary, not duplicative. Enriched knowledge provides early generic research; library basepoints provide deep project-specific knowledge.

---

## System 1: Enriched Knowledge

### Location

```
geist/config/enriched-knowledge/
├── library-research.md      # Generic best practices for detected libraries
├── stack-best-practices.md  # Architecture patterns for your stack
├── security-notes.md        # Security vulnerabilities (CVEs)
├── version-analysis.md      # Dependency version status
└── domain-knowledge.md      # Industry-specific patterns
```

### When Created

**Command**: `/adapt-to-product` (Phase 2: Web Research)

**Timing**: Very early in setup, BEFORE basepoints exist

### What It Contains

| File | Content | Source |
|------|---------|--------|
| `library-research.md` | Generic best practices, common patterns | Web search |
| `stack-best-practices.md` | Architecture patterns for your stack | Web search |
| `security-notes.md` | CVEs, vulnerabilities, security issues | Web search |
| `version-analysis.md` | Current vs latest versions, upgrade notes | Web search |
| `domain-knowledge.md` | Industry-specific patterns | Web search |

### Characteristics

- ❌ **NOT project-specific** - doesn't know how YOU use the library
- ❌ **No boundaries** - doesn't know what features you use/don't use
- ✅ **Security info** - CVEs and vulnerabilities
- ✅ **Version info** - upgrade recommendations
- ✅ **Generic best practices** - from official sources

### Where Used

1. **`deploy-agents/1-validate-prerequisites.md`** - Loads for specialization hints
2. **`deploy-agents/8-specialize-standards.md`** - Checks for security issues, outdated deps
3. **Referenced in documentation** - General guidance

---

## System 2: Library Basepoints

### Location

```
geist/basepoints/libraries/
├── README.md                    # Index of all library basepoints
├── data/                        # Data access libraries
│   └── [library].md
├── domain/                      # Domain logic libraries
│   └── [library].md
├── util/                        # Utility libraries
│   └── [library].md
├── infrastructure/              # Infrastructure libraries
│   └── [library].md
└── framework/                   # Framework libraries
    └── [library].md
```

### When Created

**Command**: `/create-basepoints` (Phase 8: Generate Library Basepoints)

**Timing**: After module basepoints are generated, requires codebase analysis

### What It Contains

Each library basepoint includes:

| Section | Content | Source |
|---------|---------|--------|
| **Project Usage** | How YOUR project uses this library | Basepoints + codebase analysis |
| **Boundaries** | What you use / don't use | Codebase analysis |
| **Patterns** | YOUR usage and integration patterns | Basepoints + codebase |
| **Workflows** | Internal and project-specific workflows | Official docs + codebase |
| **Best Practices** | Official + project-specific | Web research + codebase |
| **Security Notes** | CVEs and vulnerabilities | Merged from enriched-knowledge |
| **Version Status** | Version info and upgrade notes | Merged from enriched-knowledge |
| **Troubleshooting** | Common issues and debugging | Official docs + experience |

### Characteristics

- ✅ **Project-specific** - knows how YOU use the library
- ✅ **Has boundaries** - documents what you use/don't use
- ✅ **Has troubleshooting** - in YOUR context
- ✅ **Security info** - merged from enriched-knowledge
- ✅ **Version info** - merged from enriched-knowledge
- ✅ **Best practices** - both generic AND project-specific

### Where Used

1. **`extract-library-basepoints-knowledge.md`** - Extracts for spec/implementation commands
2. **`shape-spec`, `write-spec`, `create-tasks`, `implement-tasks`** - Context enrichment
3. **`orchestrate-tasks`** - Injected into prompts
4. **`fix-bug`** - Library-specific troubleshooting

---

## Comparison

| Aspect | `config/enriched-knowledge/` | `basepoints/libraries/` |
|--------|------------------------------|-------------------------|
| **When Created** | adapt-to-product (early) | create-basepoints (after codebase analysis) |
| **Project-Specific** | ❌ No | ✅ Yes |
| **Knows Your Usage** | ❌ No | ✅ Yes |
| **Has Boundaries** | ❌ No | ✅ Yes (what you use/don't use) |
| **Has Troubleshooting** | ❌ Limited | ✅ Yes (in your context) |
| **Security Info** | ✅ Yes (CVEs) | ✅ Yes (merged from enriched-knowledge) |
| **Version Info** | ✅ Yes | ✅ Yes (merged from enriched-knowledge) |
| **Best Practices** | ✅ Generic | ✅ Generic + Project-specific |
| **Primary Users** | deploy-agents | spec/implementation commands |

---

## Timeline: When Each Is Created

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         KNOWLEDGE CREATION TIMELINE                          │
└─────────────────────────────────────────────────────────────────────────────┘

    /adapt-to-product                    /create-basepoints
    ═════════════════                    ══════════════════
           │                                    │
           │                                    │
    ┌──────▼──────┐                      ┌──────▼──────┐
    │   Phase 2   │                      │   Phase 8   │
    │ Web Research│                      │  Generate   │
    │             │                      │  Library    │
    │ • Search    │                      │  Basepoints │
    │   libraries │                      │             │
    │ • Get CVEs  │                      │ • Analyze   │
    │ • Get       │                      │   codebase  │
    │   versions  │                      │ • Extract   │
    └──────┬──────┘                      │   patterns  │
           │                             │ • Merge     │
           ▼                             │   security  │
    ┌─────────────────┐                  │   & version │
    │ config/enriched-│                  │   info      │
    │ knowledge/      │                  └──────┬──────┘
    │                 │                         │
    │ • library-      │                         ▼
    │   research.md   │                  ┌─────────────────┐
    │ • security-     │───────────────▶  │ basepoints/     │
    │   notes.md      │   (merged into)  │ libraries/      │
    │ • version-      │                  │                 │
    │   analysis.md   │                  │ • [library].md  │
    └─────────────────┘                  │   with security │
                                         │   & version     │
                                         │   sections      │
                                         └─────────────────┘

    ◀────────────────────────────────────────────────────────────────────────▶
    EARLY (before basepoints)              LATER (after module basepoints)
```

---

## Usage: When Each Is Used

### Enriched Knowledge Usage

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ENRICHED KNOWLEDGE USAGE                                  │
└─────────────────────────────────────────────────────────────────────────────┘

    /deploy-agents
    ═══════════════
           │
    ┌──────▼──────────────────────────────────────────────────────────────────┐
    │  Phase 1: Validate Prerequisites                                         │
    │  ────────────────────────────────                                        │
    │  • Loads enriched-knowledge for specialization hints                     │
    │  • Uses security-notes.md to flag vulnerable dependencies                │
    │  • Uses version-analysis.md to identify outdated packages                │
    └─────────────────────────────────────────────────────────────────────────┘
           │
    ┌──────▼──────────────────────────────────────────────────────────────────┐
    │  Phase 8: Specialize Standards                                           │
    │  ────────────────────────────────                                        │
    │  • Checks security-notes.md for security-related standards               │
    │  • Uses stack-best-practices.md for pattern guidance                     │
    └─────────────────────────────────────────────────────────────────────────┘
```

### Library Basepoints Usage

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    LIBRARY BASEPOINTS USAGE                                  │
└─────────────────────────────────────────────────────────────────────────────┘

    /shape-spec, /write-spec, /create-tasks, /implement-tasks
    ═══════════════════════════════════════════════════════════
           │
    ┌──────▼──────────────────────────────────────────────────────────────────┐
    │  extract-library-basepoints-knowledge.md                                 │
    │  ────────────────────────────────────────                                │
    │  • Traverses basepoints/libraries/                                       │
    │  • Extracts patterns, workflows, best practices, troubleshooting         │
    │  • Organizes by category (data, domain, util, infrastructure, framework) │
    │  • Caches to implementation/cache/library-basepoints-knowledge.md        │
    └─────────────────────────────────────────────────────────────────────────┘
           │
    ┌──────▼──────────────────────────────────────────────────────────────────┐
    │  Context Enrichment                                                      │
    │  ──────────────────                                                      │
    │  • Library knowledge injected into $ENRICHED_CONTEXT                     │
    │  • Used to inform spec writing, task creation, implementation            │
    │  • Provides boundaries (what to use / not use)                           │
    └─────────────────────────────────────────────────────────────────────────┘

    /orchestrate-tasks
    ═══════════════════
           │
    ┌──────▼──────────────────────────────────────────────────────────────────┐
    │  Prompt Generation                                                       │
    │  ─────────────────                                                       │
    │  • Library basepoints knowledge injected into each prompt file           │
    │  • Provides context for implementation agents                            │
    └─────────────────────────────────────────────────────────────────────────┘

    /fix-bug
    ════════
           │
    ┌──────▼──────────────────────────────────────────────────────────────────┐
    │  Phase 2: Research Libraries                                             │
    │  ───────────────────────────                                             │
    │  • Deep-dives into library basepoints for affected libraries             │
    │  • Uses troubleshooting sections for debugging guidance                  │
    │  • Uses patterns sections to understand expected behavior                │
    └─────────────────────────────────────────────────────────────────────────┘
```

### Knowledge Usage by Command

| Command | Enriched Knowledge | Library Basepoints |
|---------|-------------------|-------------------|
| adapt-to-product | ✅ Creates | - |
| create-basepoints | ✅ Reads (for merge) | ✅ Creates |
| deploy-agents | ✅ Uses | ✅ Uses |
| shape-spec | - | ✅ Extracts |
| write-spec | - | ✅ Uses |
| create-tasks | - | ✅ Uses |
| implement-tasks | - | ✅ Uses |
| orchestrate-tasks | - | ✅ Injects |
| fix-bug | - | ✅ Uses |

---

## Merge Strategy

Library basepoints merge information from enriched-knowledge to provide a complete picture:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         MERGE STRATEGY                                       │
└─────────────────────────────────────────────────────────────────────────────┘

    config/enriched-knowledge/              basepoints/libraries/[lib].md
    ══════════════════════════              ═════════════════════════════

    security-notes.md ─────────────────────▶ ## Security Notes
    │                                        [CVEs and vulnerabilities]
    │ Extracted for each library
    │
    version-analysis.md ───────────────────▶ ## Version Status
    │                                        [Version info and upgrades]
    │ Extracted for each library
    │
    library-research.md ───────────────────▶ ## Best Practices
                                             ### Official Guidelines
                                             [Generic best practices]

    + Codebase Analysis ───────────────────▶ ## Project Usage
                                             ## Boundaries
                                             ## Patterns
                                             ## Best Practices
                                             ### Project-Specific Practices
```

### How Merge Works

1. **During `create-basepoints` Phase 8**:
   - For each library being processed
   - Check if `config/enriched-knowledge/security-notes.md` exists
   - Extract security notes specific to that library
   - Check if `config/enriched-knowledge/version-analysis.md` exists
   - Extract version info specific to that library
   - Include in generated library basepoint

2. **Result**:
   - Library basepoints contain BOTH project-specific knowledge AND security/version info
   - Single source of truth for spec/implementation commands
   - No need to check multiple locations

---

## Visual Data Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    COMPLETE KNOWLEDGE DATA FLOW                              │
└─────────────────────────────────────────────────────────────────────────────┘

                              /adapt-to-product
                              ═════════════════
                                     │
                                     │ Web Research
                                     ▼
                    ┌─────────────────────────────────┐
                    │   config/enriched-knowledge/    │
                    │   • security-notes.md           │
                    │   • version-analysis.md         │
                    │   • library-research.md         │
                    └────────────────┬────────────────┘
                                     │
                                     │ Used by deploy-agents
                                     │ for specialization
                                     │
                                     │ AND
                                     │
                                     │ Merged into library
                                     │ basepoints
                                     ▼
                              /create-basepoints
                              ══════════════════
                                     │
                                     │ Phase 8: Generate Library Basepoints
                                     │ • Codebase analysis
                                     │ • Basepoints patterns
                                     │ • Official docs research
                                     │ • MERGE security/version from enriched-knowledge
                                     ▼
                    ┌─────────────────────────────────┐
                    │   basepoints/libraries/         │
                    │   • [category]/[library].md     │
                    │     - Project Usage             │
                    │     - Boundaries                │
                    │     - Patterns                  │
                    │     - Best Practices            │
                    │     - Security Notes ◀──────── merged
                    │     - Version Status ◀──────── merged
                    │     - Troubleshooting           │
                    └────────────────┬────────────────┘
                                     │
                                     │ Extracted by
                                     │ extract-library-basepoints-knowledge.md
                                     ▼
                    ┌─────────────────────────────────┐
                    │   $LIBRARY_KNOWLEDGE            │
                    │   (context enrichment)          │
                    └────────────────┬────────────────┘
                                     │
                    ┌────────────────┼────────────────┐
                    │                │                │
                    ▼                ▼                ▼
             /shape-spec      /write-spec      /implement-tasks
             /create-tasks    /orchestrate-tasks    /fix-bug
```

---

## Best Practices

### When to Use Enriched Knowledge

- During `deploy-agents` for specialization decisions
- When you need security vulnerability information BEFORE basepoints exist
- For generic best practices during early setup

### When to Use Library Basepoints

- During spec/implementation commands (always)
- When you need project-specific library guidance
- When you need to know what features you use/don't use
- When troubleshooting library issues

### Regenerating Knowledge

**To regenerate enriched-knowledge:**
```bash
# Re-run adapt-to-product
/adapt-to-product
```

**To regenerate library basepoints:**
```bash
# Re-run create-basepoints (or update)
/create-basepoints
# OR
/update-basepoints-and-redeploy
```

---

*Last Updated: 2026-01-18*
