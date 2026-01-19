# Workflow: Research Stack Patterns

## Purpose

Research architecture patterns, project structure conventions, and testing strategies for the detected tech stack combination.

## Inputs

- `DETECTED_LANGUAGE` - Primary language
- `DETECTED_FRAMEWORK` - Web framework
- `DETECTED_BACKEND` - Backend technology
- `DETECTED_DATABASE` - Database

## Outputs

- `geist/config/enriched-knowledge/stack-best-practices.md`

---

## Web Search Queries

### Query Templates

```
1. "[frontend] [backend] architecture patterns"
2. "[stack] project structure best practices"
3. "[stack] folder organization"
4. "[stack] testing strategy"
5. "[frontend] [backend] full stack patterns"
```

---

## Workflow

### Step 1: Build Stack String

```bash
CURRENT_YEAR=$(date +%Y)
OUTPUT_FILE="geist/config/enriched-knowledge/stack-best-practices.md"

# Build a stack description string
STACK_PARTS=""
[ -n "$DETECTED_FRAMEWORK" ] && STACK_PARTS="$DETECTED_FRAMEWORK"
[ -n "$DETECTED_BACKEND" ] && STACK_PARTS="$STACK_PARTS $DETECTED_BACKEND"
[ -n "$DETECTED_DATABASE" ] && STACK_PARTS="$STACK_PARTS $DETECTED_DATABASE"
STACK_STRING=$(echo "$STACK_PARTS" | xargs)  # Trim whitespace

echo "   Researching patterns for: $STACK_STRING"
```

### Step 2: Initialize Output File

```bash
cat > "$OUTPUT_FILE" << HEADER_EOF
# Tech Stack Best Practices

> Architecture patterns and recommendations for your tech stack
> Generated: $(date -Iseconds)

**Detected Stack:** $STACK_STRING

---

HEADER_EOF
```

### Step 3: Research Architecture Patterns

```bash
cat >> "$OUTPUT_FILE" << 'ARCH_EOF'

## Recommended Architecture

<!-- Web search: "[stack] architecture patterns [year]" -->

### Overview

Based on your tech stack, consider these architectural approaches:

### Layer Architecture

```
┌─────────────────────────────────────────┐
│           Presentation Layer            │
│    (UI Components, Views, Templates)    │
├─────────────────────────────────────────┤
│           Application Layer             │
│   (Controllers, Services, Use Cases)    │
├─────────────────────────────────────────┤
│             Domain Layer                │
│    (Business Logic, Entities, Rules)    │
├─────────────────────────────────────────┤
│          Infrastructure Layer           │
│  (Database, External APIs, File System) │
└─────────────────────────────────────────┘
```

### Key Principles

1. **Separation of Concerns** - Each layer has a distinct responsibility
2. **Dependency Inversion** - High-level modules don't depend on low-level modules
3. **Single Responsibility** - Each component does one thing well
4. **Open/Closed** - Open for extension, closed for modification

---

ARCH_EOF
```

### Step 4: Research Project Structure

```bash
cat >> "$OUTPUT_FILE" << 'STRUCTURE_EOF'

## Recommended Project Structure

<!-- Web search: "[stack] project structure best practices" -->

### General Structure

```
project-root/
├── src/
│   ├── components/     # UI components (if applicable)
│   ├── services/       # Business logic services
│   ├── models/         # Data models/entities
│   ├── utils/          # Utility functions
│   ├── config/         # Configuration files
│   └── index.ts        # Entry point
├── tests/
│   ├── unit/           # Unit tests
│   ├── integration/    # Integration tests
│   └── e2e/            # End-to-end tests
├── docs/               # Documentation
├── scripts/            # Build/deploy scripts
└── config files        # package.json, tsconfig, etc.
```

### Naming Conventions

- **Files**: `kebab-case.ts` or `PascalCase.tsx` for components
- **Directories**: `kebab-case/`
- **Tests**: `*.test.ts` or `*.spec.ts`
- **Types/Interfaces**: `PascalCase`

### Module Organization

- Group by feature, not by type (feature-first)
- Keep related files close together
- Limit folder nesting (max 3-4 levels)
- Use barrel exports (index.ts) for clean imports

---

STRUCTURE_EOF
```

### Step 5: Research Testing Strategy

```bash
cat >> "$OUTPUT_FILE" << 'TESTING_EOF'

## Testing Strategy

<!-- Web search: "[stack] testing strategy" -->

### Test Pyramid

```
         /\
        /  \
       / E2E\        <- Few, slow, expensive
      /──────\
     /  Int.  \      <- Some, medium speed
    /──────────\
   /    Unit    \    <- Many, fast, cheap
  /──────────────\
```

### Recommended Coverage

| Layer | Coverage Target | Focus |
|-------|----------------|-------|
| Unit | 80%+ | Business logic, utilities |
| Integration | 60%+ | API endpoints, services |
| E2E | Critical paths | User journeys |

### Testing Best Practices

1. **Write tests first** (TDD) for complex logic
2. **Test behavior, not implementation**
3. **Use meaningful test names** that describe the scenario
4. **Keep tests independent** - no shared state
5. **Mock external dependencies** in unit tests
6. **Use factories** for test data creation

### Recommended Tools

- **Unit Testing**: Jest, Vitest, pytest, cargo test
- **Integration Testing**: Supertest, pytest, integration frameworks
- **E2E Testing**: Playwright, Cypress, Selenium
- **Mocking**: MSW, unittest.mock, mockall

---

TESTING_EOF
```

### Step 6: Research Data Flow Patterns

```bash
cat >> "$OUTPUT_FILE" << 'DATAFLOW_EOF'

## Data Flow Patterns

<!-- Web search: "[stack] state management patterns" -->

### Frontend State Management

For complex applications, consider:

1. **Local State** - Component-level state for UI
2. **Server State** - Data from API (use React Query, SWR, etc.)
3. **Global State** - Shared across components (Context, Redux, Zustand)

### API Design Patterns

- **REST** - Resource-based, stateless, cacheable
- **GraphQL** - Query language, single endpoint, typed
- **tRPC** - End-to-end type safety for TypeScript

### Data Fetching Strategies

1. **Server-Side Rendering (SSR)** - SEO, initial load
2. **Static Site Generation (SSG)** - Performance, caching
3. **Client-Side Rendering (CSR)** - Interactivity
4. **Incremental Static Regeneration (ISR)** - Best of both

---

DATAFLOW_EOF
```

### Step 7: Add Footer

```bash
cat >> "$OUTPUT_FILE" << 'FOOTER_EOF'

## Implementation Checklist

Before implementing, verify:

- [ ] Architecture aligns with team expertise
- [ ] Project structure supports scalability
- [ ] Testing strategy covers critical paths
- [ ] Data flow patterns match requirements
- [ ] Security considerations addressed

## Sources

Patterns compiled from:
- Official framework documentation
- Community best practices
- Industry standards (Clean Architecture, DDD)
- Real-world project examples

---

*Generated by Geist Adaptive Questionnaire System*

FOOTER_EOF

echo "   ✓ Stack patterns saved to $OUTPUT_FILE"
```

---

## Important Constraints

- Patterns should be practical, not theoretical
- Should adapt recommendations to detected stack
- Must provide concrete examples where possible
- Should highlight trade-offs for different approaches
