# Standard: Project Profile Schema

## Overview

This document defines the schema for `geist/config/project-profile.yml`, which stores auto-detected and user-confirmed project configuration. This profile is generated during `/adapt-to-product` and used by subsequent commands.

---

## Schema Definition

```yaml
# Project Profile Schema
# Location: geist/config/project-profile.yml

gathered:
  # Auto-detected from codebase analysis
  
  project_type: string
    # Values: web_app | api | cli | library | monorepo | unknown
    # Source: Directory structure, config files, entry points
    # Example: "web_app"
  
  tech_stack:
    language: string
      # Values: typescript | javascript | rust | python | go | unknown
      # Source: Config files (package.json, Cargo.toml, etc.)
      # Example: "typescript"
    
    framework: string
      # Values: react | vue | angular | nextjs | express | fastapi | etc.
      # Source: Dependencies in config files
      # Example: "nextjs"
    
    backend: string
      # Values: nodejs | rust | python | go | etc.
      # Source: Runtime detection from configs
      # Example: "nodejs"
    
    database: string
      # Values: postgresql | mongodb | mysql | redis | etc.
      # Source: Database dependencies or config
      # Example: "postgresql"
  
  size:
    lines: number
      # Approximate total lines of code (excluding deps)
      # Source: File analysis
      # Example: 15234
    
    files: number
      # Total source files count
      # Source: File enumeration
      # Example: 120
    
    modules: number
      # Detected module count
      # Source: Directory analysis
      # Example: 8
  
  commands:
    build: string
      # Build/compile command
      # Source: package.json scripts, Makefile, CI config
      # Example: "npm run build"
    
    test: string
      # Test command
      # Source: package.json scripts, test config files
      # Example: "npm test"
    
    lint: string
      # Lint/static analysis command
      # Source: package.json scripts, lint configs
      # Example: "npm run lint"

inferred:
  # Values derived from gathered data
  
  security_level: string
    # Values: low | moderate | high
    # Inferred from: Auth deps, secrets management, encryption
    # Example: "moderate"
  
  complexity: string
    # Values: simple | moderate | complex
    # Inferred from: File count, module count, architecture
    # Example: "moderate"

user_specified:
  # Values that must be provided by user (can't detect)
  
  compliance: array
    # Values: [] | [gdpr] | [hipaa] | [soc2] | [pci-dss] | combinations
    # Source: User input (can't detect from code)
    # Example: ["gdpr", "soc2"]
  
  human_review_level: string
    # Values: minimal | moderate | high
    # Source: User preference
    # Example: "moderate"

_meta:
  # Metadata about the detection process
  
  detected_at: datetime
    # ISO 8601 timestamp of detection
    # Example: "2026-01-16T12:00:00Z"
  
  confirmed_at: datetime
    # ISO 8601 timestamp of user confirmation
    # Example: "2026-01-16T12:01:00Z"
  
  detection_confidence: number
    # Overall confidence score (0.0 - 1.0)
    # Example: 0.92
  
  user_confirmed: boolean
    # Whether user confirmed the detected values
    # Example: true
  
  questions_asked: number
    # Number of questions asked to user
    # Example: 2
  
  questions_auto_answered: number
    # Number of values auto-detected (not asked)
    # Example: 24
  
  needs_user_input:
    # Flags for values that need user clarification
    language: boolean
    project_type: boolean
```

---

## Example Profile

```yaml
# Example: React + Node.js Web Application

gathered:
  project_type: web_app
  tech_stack:
    language: typescript
    framework: nextjs
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

inferred:
  security_level: high
  complexity: moderate

user_specified:
  compliance:
    - gdpr
  human_review_level: moderate

_meta:
  detected_at: "2026-01-16T12:00:00Z"
  confirmed_at: "2026-01-16T12:01:00Z"
  detection_confidence: 0.92
  user_confirmed: true
  questions_asked: 2
  questions_auto_answered: 24
  needs_user_input:
    language: false
    project_type: false
```

---

## Validation Rules

| Field | Required | Validation |
|-------|----------|------------|
| `gathered.project_type` | Yes | Must be valid enum value or "unknown" |
| `gathered.tech_stack.language` | Yes | Must be valid enum value or "unknown" |
| `gathered.size.files` | Yes | Must be >= 0 |
| `inferred.security_level` | Yes | Must be low/moderate/high |
| `user_specified.human_review_level` | Yes | Must be minimal/moderate/high |
| `_meta.detection_confidence` | Yes | Must be 0.0 - 1.0 |

---

## Usage by Commands

| Command | Reads | Writes | Purpose |
|---------|-------|--------|---------|
| `/adapt-to-product` | - | All | Initial detection and creation |
| `/create-basepoints` | All | Partial | Load profile, add architecture info |
| `/deploy-agents` | All | - | Use for specialization decisions |

---

## Confidence Thresholds

| Confidence | Meaning | Action |
|------------|---------|--------|
| >= 0.90 | High confidence | Proceed without questions |
| 0.70 - 0.89 | Medium confidence | Proceed, flag for confirmation |
| < 0.70 | Low confidence | Ask user to clarify |

---

*Schema Version: 1.0*
*Generated by Geist Adaptive Questionnaire System*
