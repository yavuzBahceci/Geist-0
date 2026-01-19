# Installation Guide

This guide walks you through installing and setting up Geist in your project, from initial installation to fully specialized Geist.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Detailed Installation](#detailed-installation)
- [Post-Installation Setup](#post-installation-setup)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)

---

## Prerequisites

### System Requirements

- **Shell**: Bash or Zsh
- **Git**: For cloning and change detection (optional but recommended)
- **Editor**: Cursor IDE (recommended for `/` commands) or any editor

### Supported Project Types

Geist auto-detects and works with:

| Project Type | Config Files | Commands Detected |
|--------------|--------------|-------------------|
| Node.js/TypeScript | `package.json`, `tsconfig.json` | npm scripts |
| Rust | `Cargo.toml` | cargo commands |
| Go | `go.mod` | go commands |
| Python | `requirements.txt`, `pyproject.toml` | pytest, flake8, etc. |
| Generic | `Makefile`, CI configs | make targets |

---

## Quick Start

```bash
# 1. Clone Geist (one-time)
git clone <repository-url> ~/geist

# 2. Navigate to your project
cd /path/to/your/project

# 3. Install Geist
~/geist/scripts/project-install.sh --profile default --geist-commands true

# 4. Run specialization commands (in Cursor IDE or terminal)
/adapt-to-product   # Detects and extracts product info
/create-basepoints  # Creates codebase documentation
/deploy-agents      # Specializes templates
/cleanup-geist   # Verifies completeness

# 5. Ready! Start using development commands
/shape-spec "My new feature"
```

---

## Detailed Installation

### Step 1: Clone Geist Repository

Clone Geist to a permanent location (e.g., your home directory):

```bash
git clone <repository-url> ~/geist
```

**Location choice**:
- `~/geist` - Recommended for easy access
- `~/tools/geist` - If you organize tools
- Any location - Just remember the path

### Step 2: Navigate to Your Project

```bash
cd /path/to/your/project
```

**Important**: Run the installation script from your project's root directory.

### Step 3: Run Installation Script

```bash
~/geist/scripts/project-install.sh --profile default --geist-commands true
```

**Options**:

| Option | Description | Default |
|--------|-------------|---------|
| `--profile` | Profile to use | `default` |
| `--geist-commands` | Install as `/` commands | `false` |
| `--dry-run` | Preview without installing | `false` |

**What gets installed**:

```
your-project/
└── geist/
    ├── commands/      # Abstract commands (will be specialized)
    ├── workflows/     # Abstract workflows (will be specialized)
    ├── standards/     # Global standards
    ├── agents/        # Agent definitions
    ├── config.yml     # Project configuration
    └── output/        # Command outputs (created as needed)
```

### Step 4: Verify Installation

Check that the installation completed:

```bash
ls -la geist/
```

You should see:
- `commands/` directory
- `workflows/` directory
- `standards/` directory
- `agents/` directory
- `config.yml` file

---

## Post-Installation Setup

After installation, you need to **specialize** your Geist to match your project.

### Phase 1: Product Definition

Run one of these commands (use `adapt-to-product` for existing projects):

```bash
/adapt-to-product
```

**What happens**:

1. **Automatic Detection** (no questions asked):
   ```
   ✅ Detected: Node.js project
   ✅ Tech Stack: React 18, TypeScript, Express
   ✅ Database: PostgreSQL (from deps)
   ✅ Build: npm run build
   ✅ Test: npm test
   ✅ Lint: npm run lint
   ✅ Security: High (auth dependencies detected)
   ✅ Complexity: Medium (15K lines, 120 files)
   ```

2. **Web Research** (automatic):
   ```
   🔍 Researching: React 18 best practices...
   🔍 Researching: Express security considerations...
   🔍 Checking: CVE vulnerabilities for dependencies...
   ```

3. **Confirmation** (review, don't re-enter):
   ```
   Based on your project, I detected:

   Project Type: Web Application (React + Node.js) ✓
   Size: Medium (~15K lines, 120 files) ✓
   Security: High (auth dependencies detected) ✓

   Press Enter to confirm, or type to modify.
   ```

4. **Minimal Questions** (only 2-3):
   ```
   ⚠️ Questions requiring your input:

   1. Compliance requirements? [None/SOC2/HIPAA/GDPR]
   2. Human review preference? [Minimal/Moderate/High]
   ```

**Outputs created**:
- `geist/product/mission.md`
- `geist/product/roadmap.md`
- `geist/product/tech-stack.md`
- `geist/config/project-profile.yml`
- `geist/config/enriched-knowledge/`

### Phase 2: Basepoints Creation

```bash
/create-basepoints
```

**What happens**:

1. Loads existing profile (no re-detection)
2. Detects abstraction layers in your codebase
3. Mirrors your project structure
4. Analyzes patterns, standards, flows
5. Creates module-specific basepoints
6. Creates project overview (headquarter.md)

**Outputs created**:
- `geist/basepoints/headquarter.md`
- `geist/basepoints/[layer]/[module]/agent-base-[module].md`

### Phase 3: Specialization

```bash
/deploy-agents
```

**What happens**:

1. Validates prerequisites (basepoints + product)
2. Loads all gathered knowledge:
   - `project-profile.yml` → validation commands
   - `enriched-knowledge/` → workflow decisions
   - `basepoints/` → patterns and standards
3. Extracts and merges knowledge
4. Specializes all commands with project-specific content
5. Configures validation commands automatically
6. Generates deployment report

**What gets specialized**:
- Commands: Placeholders replaced with actual values
- Workflows: Adapted to project complexity
- Standards: Project conventions applied
- Validation: Build/test/lint commands configured

### Phase 4: Cleanup & Verification

```bash
/cleanup-geist
```

**What happens**:

1. Cleans any remaining placeholders
2. Removes unnecessary generic logic
3. Fixes broken references
4. Verifies knowledge completeness
5. Generates recommendations

**Verification checklist**:
- ✅ No `{{PLACEHOLDER}}` remaining
- ✅ No project-agnostic conditionals
- ✅ All references resolve
- ✅ Knowledge coverage adequate
- ✅ All product files complete

---

## Verification

### Verify Installation Success

```bash
# Check structure
ls -la geist/

# Check product files
ls -la geist/product/

# Check basepoints
ls -la geist/basepoints/

# Check config
cat geist/config/project-profile.yml
```

### Verify Specialization Success

```bash
# Check for remaining placeholders
grep -r "{{" geist/commands/ | head -20

# Should return empty or only valid workflow references
```

### Run Cleanup Verification

```bash
/cleanup-geist
```

Expected output:
```
📊 Knowledge Completeness Verification Report

✅ Basepoints Analysis
   - Basepoints Directory: ✅ Found
   - Headquarter Basepoint: ✅ Found
   - Module Basepoints: 12 file(s) found
   - Required Sections: ✅ All present

✅ Product Knowledge Analysis
   - Product Directory: ✅ Found
   - All Files Present: ✅
   - Content Complete: ✅

✅ Coverage Analysis
   - Coverage Ratio: 5.3 (adequate)

✅ Status: READY TO USE
```

---

## Troubleshooting

### Installation Issues

**Problem**: `project-install.sh: command not found`

**Solution**: Use full path to script:
```bash
~/geist/scripts/project-install.sh --profile default
```

**Problem**: `Cannot install Geist in base installation directory`

**Solution**: You're in the Geist repo, not your project:
```bash
cd /path/to/your/project
~/geist/scripts/project-install.sh --profile default
```

### Detection Issues

**Problem**: Detection failed or returned incorrect values

**Solution**:
1. Check if config files exist (package.json, Cargo.toml, etc.)
2. Override in confirmation prompt
3. Manually update `geist/config/project-profile.yml`

**Problem**: Commands not detected

**Solution**:
1. Ensure build/test/lint scripts exist in package.json
2. Check Makefile for targets
3. Manually add to `project-profile.yml`

### Specialization Issues

**Problem**: Placeholders remain after deploy-agents

**Solution**:
1. Run `/cleanup-geist`
2. Check validation report for issues
3. Re-run `/deploy-agents`

**Problem**: Validation commands not working

**Solution**:
1. Check `geist/standards/global/validation-commands.md`
2. Verify commands work manually
3. Update with correct commands

### Knowledge Issues

**Problem**: Missing basepoints for some modules

**Solution**:
1. Run `/cleanup-geist` for verification
2. Review knowledge gap report
3. Re-run `/create-basepoints` if needed

**Problem**: Enriched knowledge empty

**Solution**:
1. Check network connectivity (web research)
2. Review research settings in profile
3. Add libraries to research manually

---

## FAQ

### General

**Q: Do I need to re-run installation after updating my project?**

A: No. Use `/update-basepoints-and-redeploy` for incremental updates.

**Q: Can I install Geist in multiple projects?**

A: Yes! Clone Geist once, install in each project separately.

**Q: Does Geist modify my source code?**

A: No. Geist creates and modifies only files in `geist/` folder.

### Detection

**Q: Why doesn't Geist ask many questions?**

A: Geist auto-detects everything possible. It only asks what can't be determined from code (compliance, user preferences).

**Q: Can I override detected values?**

A: Yes. Either in the confirmation prompt or by editing `project-profile.yml`.

**Q: What if detection is wrong?**

A: Override during confirmation, or edit `project-profile.yml` and re-run `/deploy-agents`.

### Specialization

**Q: What are placeholders?**

A: `{{PLACEHOLDER}}` syntax marks project-specific content that gets replaced during specialization.

**Q: Why run cleanup-geist?**

A: To verify knowledge completeness and clean any remaining issues.

**Q: How do I know specialization worked?**

A: Run `/cleanup-geist` and check the verification report.

### Commands

**Q: What's the difference between implement-tasks and orchestrate-tasks?**

A: 
- `implement-tasks`: Single agent implements all tasks
- `orchestrate-tasks`: Creates prompts for multi-agent implementation

**Q: Do I need to run commands in order?**

A: Yes. Follow: `adapt-to-product` → `create-basepoints` → `deploy-agents` → `cleanup-geist`

**Q: Can I skip commands?**

A: No. Each command depends on outputs from previous commands.

---

## Next Steps

After successful installation and verification:

1. **Start developing features**:
   ```bash
   /shape-spec "Add user authentication"
   ```

2. **Read documentation**:
   - [Command Flows](COMMAND-FLOWS.md) - Detailed command documentation
   - [Profile README](../README.md) - Complete profile documentation

3. **Keep Geist updated**:
   ```bash
   # After making codebase changes
   /update-basepoints-and-redeploy
   ```

---

*Last Updated: 2026-01-16*
