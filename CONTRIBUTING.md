# Contributing to Geist

Thank you for your interest in contributing to Geist! The primary way to contribute is by **creating new profiles** for specific project types or **improving existing profiles** to work better for different use cases.

---

## Table of Contents

- [Why Profiles?](#why-profiles)
- [Creating a New Profile](#creating-a-new-profile)
- [Improving Existing Profiles](#improving-existing-profiles)
- [Profile Structure](#profile-structure)
- [Testing Your Profile](#testing-your-profile)
- [PR Process](#pr-process)

---

## Why Profiles?

Geist uses **profiles** to adapt to different project types. The `default` profile is technology-agnostic and works with any project, but specialized profiles can provide:

- **Better detection** for specific tech stacks
- **Optimized workflows** for particular architectures
- **Tailored standards** for domain-specific requirements
- **Pre-configured patterns** that match how teams actually work

```
profiles/
├── default/              # Technology-agnostic (works with anything)
├── react-nextjs/         # (example) Optimized for React + Next.js
├── mobile-native/        # (example) iOS/Android native development
├── rust-systems/         # (example) Systems programming with Rust
└── your-profile/         # Your contribution!
```

---

## Creating a New Profile

### Step 1: Identify the Need

Good candidates for new profiles:

| Project Type | Why a Profile Helps |
|--------------|---------------------|
| **Framework-specific** | React, Vue, Rails, Django have distinct patterns |
| **Platform-specific** | iOS, Android, embedded systems have unique workflows |
| **Domain-specific** | ML/AI, game dev, fintech have specialized standards |
| **Architecture-specific** | Microservices, monoliths, serverless differ significantly |

### Step 2: Create Profile Directory

```bash
# Create your profile
mkdir -p profiles/your-profile-name

# Copy default as starting point
cp -r profiles/default/* profiles/your-profile-name/
```

### Step 3: Configure Inheritance

Create `profiles/your-profile-name/profile-config.yml`:

```yaml
# Profile Configuration
name: your-profile-name
description: "Optimized for [your use case]"
version: 1.0.0

# Inherit from default (recommended)
inherits_from: default

# Override specific files from parent
# Files not listed here are inherited automatically
exclude_inherited_files:
  - workflows/detection/detect-project-profile.md  # Replace with your own

# Profile-specific settings
settings:
  primary_language: typescript  # or python, rust, swift, etc.
  framework: nextjs             # or django, rails, flutter, etc.
  architecture: monolith        # or microservices, serverless, etc.
```

### Step 4: Customize for Your Use Case

Focus on these areas:

#### Detection (`workflows/detection/`)

Improve auto-detection for your tech stack:

```markdown
# In detect-project-profile.md

## Framework Detection

```bash
# Detect Next.js specifically
if [ -f "next.config.js" ] || [ -f "next.config.mjs" ]; then
    FRAMEWORK="nextjs"
    FRAMEWORK_VERSION=$(grep '"next":' package.json | cut -d'"' -f4)
fi
```
```

#### Standards (`standards/`)

Add domain-specific standards:

```
standards/
├── global/                    # Inherited from default
├── nextjs/                    # Your additions
│   ├── app-router.md          # App Router patterns
│   ├── server-components.md   # RSC best practices
│   └── data-fetching.md       # Fetching patterns
```

#### Workflows (`workflows/`)

Optimize workflows for your patterns:

```markdown
# In workflows/implementation/implement-tasks.md

## Framework-Specific Implementation

For Next.js projects, follow these patterns:
- Server Components by default
- Client Components only when needed ('use client')
- API routes in app/api/
```

### Step 5: Document Your Profile

Create `profiles/your-profile-name/README.md`:

```markdown
# Profile: your-profile-name

## Overview

This profile is optimized for [description].

## When to Use

Use this profile if your project:
- Uses [framework/language]
- Follows [architecture pattern]
- Has [specific requirements]

## Key Differences from Default

| Area | Default | This Profile |
|------|---------|--------------|
| Detection | Generic | [Your improvements] |
| Standards | Technology-agnostic | [Your additions] |
| Workflows | Generic | [Your optimizations] |

## Installation

```bash
~/geist/scripts/project-install.sh --profile your-profile-name
```

## Credits

Created by [your name/handle]
```

---

## Improving Existing Profiles

### Types of Improvements

1. **Better Detection**
   - More accurate tech stack detection
   - Handling edge cases
   - Supporting newer framework versions

2. **Enhanced Workflows**
   - More efficient processes
   - Better error handling
   - Clearer instructions

3. **Updated Standards**
   - Current best practices
   - New patterns from the community
   - Bug fixes in existing standards

4. **Documentation**
   - Clearer explanations
   - More examples
   - Better troubleshooting guides

### Making Improvements

```bash
# Fork and clone
git clone https://github.com/YOUR-USERNAME/Geist-v1.git
cd Geist-v1

# Create feature branch
git checkout -b improve/detection-nextjs-15

# Make your changes
# Edit files in profiles/default/ or profiles/[specific-profile]/

# Test thoroughly (see Testing section)

# Submit PR
```

---

## Profile Structure

Every profile follows this structure:

```
profiles/[profile-name]/
├── profile-config.yml        # Profile configuration
├── README.md                 # Profile documentation
│
├── commands/                 # Command templates
│   ├── adapt-to-product/
│   ├── create-basepoints/
│   ├── deploy-agents/
│   ├── shape-spec/
│   ├── write-spec/
│   ├── create-tasks/
│   ├── implement-tasks/
│   ├── orchestrate-tasks/
│   ├── fix-bug/
│   ├── cleanup-geist/
│   └── update-basepoints-and-redeploy/
│
├── workflows/                # Reusable workflows
│   ├── basepoints/
│   ├── codebase-analysis/
│   ├── common/
│   ├── detection/
│   ├── human-review/
│   ├── implementation/
│   ├── learning/
│   ├── planning/
│   ├── prompting/
│   ├── research/
│   ├── scope-detection/
│   ├── specification/
│   └── validation/
│
├── standards/                # Quality standards
│   └── global/
│
├── agents/                   # Agent definitions
│
└── docs/                     # Documentation
    ├── COMMAND-FLOWS.md
    ├── INSTALLATION-GUIDE.md
    ├── TROUBLESHOOTING.md
    └── command-references/
```

### Key Files to Customize

| File | Purpose | Customization Priority |
|------|---------|----------------------|
| `workflows/detection/detect-project-profile.md` | Auto-detection | **High** - Make detection accurate |
| `standards/global/conventions.md` | Coding conventions | **High** - Match your patterns |
| `workflows/codebase-analysis/generate-module-basepoints.md` | Basepoint generation | **Medium** - Optimize for your structure |
| `commands/*/single-agent/*.md` | Command phases | **Medium** - Adjust for your workflow |
| `docs/TROUBLESHOOTING.md` | Common issues | **Low** - Add profile-specific issues |

---

## Testing Your Profile

### 1. Install to Test Project

```bash
# Create or use a test project matching your profile's target
cd /path/to/test-project

# Install with your profile
~/geist/scripts/project-install.sh --profile your-profile-name --geist-commands true
```

### 2. Run the Full Command Chain

```bash
# Setup commands
/adapt-to-product     # Should detect your tech stack correctly
/create-basepoints    # Should generate relevant basepoints
/deploy-agents        # Should specialize appropriately
/cleanup-geist        # Should validate successfully

# Development commands
/shape-spec "Add a test feature"
/write-spec
/create-tasks
/implement-tasks
```

### 3. Verify Key Behaviors

| Check | How to Verify |
|-------|---------------|
| Detection accuracy | Review `geist/config/project-profile.yml` |
| Basepoint quality | Read generated basepoints in `geist/basepoints/` |
| Standard relevance | Check `geist/standards/` for your patterns |
| Command flow | Run full spec cycle, check outputs |

### 4. Test Edge Cases

- Projects with unusual structure
- Mixed tech stacks
- Monorepos
- Minimal projects

---

## PR Process

### 1. Before Submitting

- [ ] Profile tested with real project
- [ ] README.md documents the profile
- [ ] profile-config.yml is complete
- [ ] No broken workflow references
- [ ] Standards follow existing format

### 2. PR Description

```markdown
## New Profile: [profile-name]

### Target Use Case
[Who should use this profile and why]

### Key Features
- [Detection improvements]
- [Workflow optimizations]
- [Standards additions]

### Testing
- Tested with [describe test project]
- Full command chain verified
- Edge cases checked: [list]

### Screenshots/Examples
[If applicable, show detection results or generated outputs]
```

### 3. For Profile Improvements

```markdown
## Improvement: [brief description]

### Problem
[What wasn't working well]

### Solution
[What you changed and why]

### Testing
- [ ] Tested with existing projects
- [ ] No regressions in other areas
- [ ] Documentation updated

### Files Changed
- `profiles/default/workflows/...`
- `profiles/default/standards/...`
```

---

## Questions?

- **Check existing profiles** for patterns and examples
- **Open an issue** to discuss new profile ideas before starting
- **Review `profiles/default/`** as the reference implementation

Thank you for contributing to Geist!

---

*Last Updated: 2026-01-18*
