# Phase 2: Extract Basepoints Knowledge

Now that prerequisites are validated, extract knowledge from basepoints for use in specialization.

## Core Responsibilities

1. **Read all basepoint files**: Read headquarter.md and all module basepoints
2. **Extract patterns**: Identify design patterns, coding patterns, architectural patterns
3. **Extract standards**: Identify naming conventions, coding styles, structure standards
4. **Extract flows**: Identify data flows, control flows, dependency flows
5. **Extract strategies**: Identify implementation and architectural strategies
6. **Store knowledge**: Save extracted knowledge for use in later phases

---

## Step 1: Read Headquarter File

**ACTION REQUIRED:** Read `geist/basepoints/headquarter.md` and extract:

- Project overview (type, language, architecture)
- Abstraction layers identified
- Key patterns listed
- Core flows documented

---

## Step 2: Read All Module Basepoints

**ACTION REQUIRED:** Find and read all basepoint files matching `agent-base-*.md` in `geist/basepoints/`.

For each basepoint file, extract:

### Patterns Section
- Design patterns (creational, structural, behavioral)
- Coding patterns (idioms, conventions)
- Architectural patterns (layers, components)

### Standards Section
- Naming conventions
- Coding style rules
- Structure standards

### Flows Section
- Data flows
- Control flows
- Dependency flows

### Strategies Section
- Implementation strategies
- Architectural strategies

### Testing Section
- Test patterns
- Test strategies
- Test organization

---

## Step 3: Organize Extracted Knowledge

**ACTION REQUIRED:** Organize the extracted knowledge into a structured format.

Create `geist/output/deploy-agents/knowledge/basepoints-knowledge.json` with:

```json
{
  "patterns": {
    "same_layer": {
      "[layer_name]": ["pattern1", "pattern2"]
    },
    "cross_layer": ["pattern1", "pattern2"]
  },
  "standards": {
    "naming": ["convention1", "convention2"],
    "coding_style": ["style1", "style2"],
    "structure": ["structure1", "structure2"]
  },
  "flows": {
    "data": ["flow1", "flow2"],
    "control": ["flow1", "flow2"],
    "dependency": ["flow1", "flow2"]
  },
  "strategies": {
    "implementation": ["strategy1", "strategy2"],
    "architectural": ["strategy1", "strategy2"]
  },
  "testing": {
    "patterns": ["pattern1", "pattern2"],
    "strategies": ["strategy1", "strategy2"]
  },
  "project_structure": {
    "abstraction_layers": ["layer1", "layer2"],
    "module_count": 0
  }
}
```

---

## Step 4: Create Knowledge Summary

**ACTION REQUIRED:** Create `geist/output/deploy-agents/knowledge/basepoints-knowledge-summary.md` with:

```markdown
# Extracted Basepoints Knowledge

## Patterns from Same Abstraction Layers

### [Layer Name]
- [Pattern description]

## Patterns Between Abstraction Layers
- [Cross-layer pattern description]

## Standards
### Naming Conventions
- [Convention]

### Coding Style
- [Style rule]

## Flows
- [Flow description]

## Strategies
- [Strategy description]

## Testing Approaches
- [Testing approach]

## Project Structure
- Abstraction Layers: [count]
- Module Basepoints: [count]
```

---

## Step 5: Validate Extraction

**ACTION REQUIRED:** Verify the extraction is complete:

1. Check that `basepoints-knowledge.json` was created
2. Check that all major sections have content
3. Count patterns, standards, flows extracted

---

## Display confirmation and next step

Once extraction is complete, output:

```
✅ Basepoint knowledge extraction complete!

- Basepoint files read: [count] files
- Patterns extracted: [count] same-layer, [count] cross-layer
- Standards extracted: [count]
- Flows extracted: [count]
- Strategies extracted: [count]
- Testing approaches extracted: [count]
- Abstraction layers detected: [list]

Knowledge saved to:
- geist/output/deploy-agents/knowledge/basepoints-knowledge.json
- geist/output/deploy-agents/knowledge/basepoints-knowledge-summary.md

NEXT STEP 👉 Proceeding to phase 3: Extract Product Knowledge
```

---

## User Standards & Preferences Compliance

IMPORTANT: Ensure knowledge extraction aligns with:

@geist/standards/global/codebase-analysis.md
@geist/standards/global/conventions.md
