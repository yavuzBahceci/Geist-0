# Command References

> Visual guides for all Geist commands with step-by-step workflows

---

## Quick Navigation

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         GEIST COMMAND OVERVIEW                               │
└─────────────────────────────────────────────────────────────────────────────┘

                           SETUP COMMANDS
                    (Run once to set up Geist)
                                 │
    ┌────────────────────────────┼────────────────────────────┐
    │                            │                            │
    ▼                            ▼                            ▼
┌─────────────┐          ┌─────────────┐          ┌─────────────┐
│ /adapt-to-  │─────────▶│ /create-    │─────────▶│ /deploy-    │
│ product     │          │ basepoints  │          │ agents      │
└─────────────┘          └─────────────┘          └─────────────┘
     │                        │                        │
     ▼                        ▼                        ▼
 Extract product         Create codebase         Specialize
 info from code          documentation           commands


                         DEVELOPMENT COMMANDS
                    (Run for each feature/spec)
                                 │
    ┌────────────────────────────┼────────────────────────────┐
    │                            │                            │
    ▼                            ▼                            ▼
┌─────────────┐          ┌─────────────┐          ┌─────────────┐
│ /shape-spec │─────────▶│ /write-spec │─────────▶│ /create-    │
│             │          │             │          │ tasks       │
└─────────────┘          └─────────────┘          └─────────────┘
     │                        │                        │
     ▼                        ▼                        ▼
 Research &              Write detailed           Break into
 gather reqs             specification           tasks
                                                      │
                              ┌────────────────────────┤
                              │                        │
                              ▼                        ▼
                       ┌─────────────┐          ┌─────────────┐
                       │ /implement- │          │ /orchestrate│
                       │ tasks       │          │ -tasks      │
                       └─────────────┘          └─────────────┘
                            │                        │
                            ▼                        ▼
                       Single agent             Multi-agent
                       implementation           orchestration


                        MAINTENANCE & UTILITY COMMANDS
                    (Run to keep Geist updated or fix issues)
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
              ▼                  ▼                  ▼
       ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
       │ /update-    │    │ /cleanup-   │    │ /fix-bug    │
       │ basepoints  │    │ geist    │    │             │
       │ -and-redeploy    │             │    │             │
       └─────────────┘    └─────────────┘    └─────────────┘
            │                  │                  │
            ▼                  ▼                  ▼
       Sync after         Verify &          Analyze &
       code changes       clean up          fix bugs
```

---

## Setup Commands

These commands are run once (or occasionally) to set up and configure Geist for your project.

| Command | Purpose | When to Run |
|---------|---------|-------------|
| [/adapt-to-product](./adapt-to-product.md) | Extract product info from existing codebase | First setup (existing project) |
| [/plan-product](./plan-product.md) | Plan product from scratch (no codebase) | First setup (new project) |
| [/create-basepoints](./create-basepoints.md) | Create codebase documentation | After adapt/plan-product |
| [/deploy-agents](./deploy-agents.md) | Specialize commands for your project | After create-basepoints |

---

## Development Commands

These commands are run for each feature you develop.

| Command | Purpose | When to Run |
|---------|---------|-------------|
| [/shape-spec](./shape-spec.md) | Research and gather requirements | Start of new feature |
| [/write-spec](./write-spec.md) | Write detailed specification | After shape-spec |
| [/create-tasks](./create-tasks.md) | Break spec into tasks | After write-spec |
| [/implement-tasks](./implement-tasks.md) | Implement tasks (single agent) | After create-tasks |
| [/orchestrate-tasks](./orchestrate-tasks.md) | Generate prompts for multi-agent | After create-tasks |

---

## Maintenance & Utility Commands

These commands keep your Geist synchronized with your codebase and help fix issues.

| Command | Purpose | When to Run |
|---------|---------|-------------|
| [/update-basepoints-and-redeploy](./update-basepoints-and-redeploy.md) | Sync after code changes | After significant changes |
| [/cleanup-geist](./cleanup-geist.md) | Verify and clean deployment | After deploy or periodically |
| [/fix-bug](./fix-bug.md) | Analyze and fix bugs/feedback | When encountering bugs or implementing feedback |

---

## Typical Workflows

### First-Time Setup

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        FIRST-TIME SETUP WORKFLOW                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Option A: Existing Codebase                                                 │
│  ───────────────────────────                                                 │
│  1. /adapt-to-product                                                        │
│     └── Extracts: mission.md, roadmap.md, tech-stack.md                      │
│     └── Runs: Product-Focused Cleanup (Phase 7)                              │
│                                                                              │
│  Option B: New Project                                                       │
│  ────────────────────                                                        │
│  1. /plan-product                                                            │
│     └── Creates: mission.md, roadmap.md, tech-stack.md                       │
│     └── Runs: Product-Focused Cleanup (Phase 5)                              │
│                                                                              │
│  Then continue with:                                                         │
│  ──────────────────                                                          │
│  2. /create-basepoints                                                       │
│     └── Creates: headquarter.md, module basepoints                           │
│                                                                              │
│  3. /deploy-agents                                                           │
│     └── Specializes: all commands for your project                           │
│                                                                              │
│  4. /cleanup-geist (optional)                                             │
│     └── Verifies: deployment completeness                                    │
│                                                                              │
│  ✅ Geist is now ready for development!                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Feature Development

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       FEATURE DEVELOPMENT WORKFLOW                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. /shape-spec "Add user authentication"                                    │
│     └── Creates: requirements.md, basepoints-knowledge.md                    │
│                                                                              │
│  2. /write-spec                                                              │
│     └── Creates: spec.md                                                     │
│                                                                              │
│  3. /create-tasks                                                            │
│     └── Creates: tasks.md                                                    │
│                                                                              │
│  4. /implement-tasks OR /orchestrate-tasks                                   │
│     └── Implements: code changes                                             │
│                                                                              │
│  5. /update-basepoints-and-redeploy (after feature complete)                 │
│     └── Syncs: basepoints with new code                                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Command Comparison

### implement-tasks vs orchestrate-tasks

| Aspect | /implement-tasks | /orchestrate-tasks |
|--------|------------------|-------------------|
| **Agent Mode** | Single agent | Multi-agent |
| **Execution** | Immediate | Generates prompts |
| **Best For** | Small features | Large features |
| **Task Groups** | Sequential | Can be parallel |
| **Output** | Code changes | Prompt files |

---

## Quick Reference

### Setup Sequence

```
/adapt-to-product → /create-basepoints → /deploy-agents → /cleanup-geist
```

### Development Sequence

```
/shape-spec → /write-spec → /create-tasks → /implement-tasks (or /orchestrate-tasks)
```

### Maintenance & Utility

```
After code changes: /update-basepoints-and-redeploy
Periodic cleanup:   /cleanup-geist
Bug fixing:         /fix-bug "error description or feedback"
```

---

## See Also

- [COMMAND-FLOWS.md](../COMMAND-FLOWS.md) - Detailed technical documentation
- [INSTALLATION-GUIDE.md](../INSTALLATION-GUIDE.md) - How to install Geist
- [PATH-REFERENCE-GUIDE.md](../PATH-REFERENCE-GUIDE.md) - File path conventions
