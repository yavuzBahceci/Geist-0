# Review Implementation

Review an implementation against standards, basepoints, and library knowledge without auto-fixing issues.

## Command Overview

This command performs a post-implementation review, analyzing the work from highest abstraction layer down, checking against standards and basepoints, and generating a findings report.

**Key Principle**: This command REPORTS findings only - it does NOT auto-fix issues.

## Prerequisites

- Implementation must exist (completed `/implement-tasks` or manual implementation)
- Basepoints should be generated for comprehensive review
- Spec should exist to validate against requirements

## Phases

### Phase 1: Load Context
**File:** `1-load-context.md`

Load implementation files and accumulated knowledge to understand what was implemented.

### Phase 2: Review Abstraction Layers
**File:** `2-review-layers.md`

Review from highest abstraction layer to lowest, checking for issues at each layer.

### Phase 3: Enrich with Basepoints Knowledge
**File:** `3-enrich-basepoints.md`

Load relevant basepoints and compare implementation against documented patterns.

### Phase 4: Enrich with Library Knowledge
**File:** `4-enrich-libraries.md`

Load relevant library basepoints and check for library misuse or anti-patterns.

### Phase 5: Generate Report
**File:** `5-generate-report.md`

Generate findings report with Layer → Issues → Severity → Recommendation structure.

---

## Execution Flow

```
1-load-context.md
       ↓
2-review-layers.md
       ↓
3-enrich-basepoints.md
       ↓
4-enrich-libraries.md
       ↓
5-generate-report.md
       ↓
   REPORT GENERATED (no auto-fix)
```

## Usage

```bash
# Review implementation for a spec
/review-implementation "geist/specs/2026-01-19-my-feature"

# Review implementation in current context
/review-implementation
```

## Output

The command generates a review report at:
`[spec-path]/implementation/review-report.md`

The report includes:
- Issues organized by abstraction layer
- Severity levels (critical, high, medium, low)
- Specific recommendations for each issue
- Standards/basepoints references for each finding

**IMPORTANT**: This command does NOT fix issues. It provides findings for manual review.
