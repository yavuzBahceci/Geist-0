# Spec Research

---

## ⚠️ CRITICAL: USER INTERACTION REQUIRED

**This workflow requires multiple rounds of user interaction.** You MUST:
1. Ask clarifying questions and **STOP and WAIT** for user responses
2. Process answers and check for visual assets
3. Ask follow-up questions if needed and **STOP and WAIT** for responses
4. Do NOT proceed to the next step until you receive user input
5. Do NOT assume answers or skip questions

---

## Core Responsibilities

1. **Read Initial Idea**: Load the raw idea from initialization.md
2. **Analyze Context**: Understand the codebase and existing patterns to inform questions
3. **Ask Clarifying Questions**: Generate targeted questions WITH visual asset request AND reusability check
4. **Process Answers**: Analyze responses and any provided visuals
5. **Ask Follow-ups**: Based on answers and visual analysis if needed
6. **Save Requirements**: Document the requirements you've gathered to a single file named: `[spec-path]/planning/requirements.md`

## Workflow

### Step 1: Read Initial Idea

Read the raw idea from `[spec-path]/planning/initialization.md` to understand what the user wants to build.

### Step 2: Analyze Context

Before generating questions, understand the codebase context:

1. **Load Basepoints Knowledge (if available)**: If basepoints exist and were extracted, load the extracted knowledge:
   ```bash
   # Determine spec path
   SPEC_PATH="[spec-path]"
   
   # Load extracted basepoints knowledge if available
   if [ -f "$SPEC_PATH/implementation/cache/basepoints-knowledge.json" ]; then
       EXTRACTED_KNOWLEDGE=$(cat "$SPEC_PATH/implementation/cache/basepoints-knowledge.json")
       SCOPE_DETECTION=$(cat "$SPEC_PATH/implementation/cache/scope-detection/semantic-analysis.json" 2>/dev/null || echo "{}")
   fi
   ```

2. **Explore Existing Codebase**: Look for existing patterns, similar features, or related functionality that might inform this spec
3. **Identify Reusable Patterns**: Use extracted basepoints knowledge to identify reusable patterns, modules, and components
4. **Understand System Architecture**: Consider how this feature fits into the existing system architecture using basepoints knowledge

This context will help you:
- Ask more relevant and contextual questions informed by basepoints knowledge
- Identify existing features that might be reused or referenced from basepoints
- Ensure the feature aligns with existing patterns documented in basepoints
- Understand technical constraints and capabilities from basepoints
- Reference historical decisions and pros/cons from basepoints when relevant

### Step 3: Check for Trade-offs and Create Checkpoints (if needed)

Before generating questions, check if trade-offs need to be reviewed:

```bash
# Check for multiple valid patterns or conflicts
{{workflows/human-review/review-trade-offs}}
```

If trade-offs are detected, present them to the user and wait for their decision before proceeding.

### Step 4: Generate First Round of Questions WITH Visual Request AND Reusability Check

Based on the initial idea, generate 4-8 targeted, NUMBERED questions that explore requirements while suggesting reasonable defaults.

**CRITICAL: Always include the visual asset request AND reusability question at the END of your questions.**

**Question generation guidelines (SDD-aligned):**
- Start each question with a number
- Propose sensible assumptions based on best practices and SDD principles
- Frame questions as "I'm assuming X, is that correct?"
- Make it easy for users to confirm or provide alternatives
- Include specific suggestions they can say yes/no to
- Always end with an open question about exclusions

**SDD-informed question patterns:**
- Ensure questions capture clear user stories in format: "As a [user], I want [action], so that [benefit]"
- Validate that acceptance criteria will be explicitly documented (not implied)
- Check for explicit scope boundaries (what's in-scope vs out-of-scope)
- Avoid questions that lead to premature technical details (SDD: focus on What & Why, not How in spec phase)
- Encourage minimal, intentionally scoped specs (prevent feature bloat)
- Help avoid SDD anti-patterns:
  - Specification theater: Ask questions that ensure specs will be actionable and referenced
  - Premature comprehensiveness: Ask questions that encourage incremental, focused specs
  - Over-engineering: Avoid questions that push toward excessive technical detail too early

**Required output format (SDD-aligned):**
```
Based on your idea for [spec name], I have some clarifying questions:

1. I assume [specific assumption]. Is that correct, or [alternative]?
2. I'm thinking [specific approach]. Should we [alternative]?
3. [Continue with numbered questions...]

**SDD Requirements Check:**
To ensure we create a well-structured specification (following spec-driven development best practices), I want to confirm:
- Will we capture user stories in the format "As a [user], I want [action], so that [benefit]"?
- Will we define clear acceptance criteria for each requirement?
- Should we explicitly define what's in-scope vs out-of-scope for this feature?

[Last numbered question about exclusions]

**Existing Code Reuse:**
Are there existing features in your codebase with similar patterns we should reference? For example:
- Similar interface elements or components to re-use
- Comparable patterns or structures
- Related logic or service objects
- Existing modules or classes with similar functionality

{{#if basepoints_knowledge_available}}
Based on basepoints analysis, I've identified these potentially reusable patterns:
- [Reusable patterns from basepoints]
- [Common modules that might be relevant]
- [Historical decisions that inform this feature]

Please provide file/folder paths or names of these features if they exist, or confirm if the basepoints suggestions are relevant.
{{/if}}

Please provide file/folder paths or names of these features if they exist.

**Visual Assets Request:**
Do you have any design mockups, wireframes, or screenshots that could help guide the development?

If yes, please place them in: `[spec-path]/planning/visuals/`

Use descriptive file names like:
- homepage-mockup.png
- dashboard-wireframe.jpg
- lofi-form-layout.png
- mobile-view.png
- existing-ui-screenshot.png

Please answer the questions above and let me know if you've added any visual files or can point to similar existing features.
```

**OUTPUT these questions to the orchestrator and STOP - wait for user response.**

### Step 5: Process Answers and MANDATORY Visual Check

After receiving user's answers from the orchestrator:

1. Store the user's answers for later documentation

2. **MANDATORY: Check for visual assets regardless of user's response:**

**CRITICAL**: You MUST run the following bash command even if the user says "no visuals" or doesn't mention visuals (Users often add files without mentioning them):

```bash
# List all files in visuals folder - THIS IS MANDATORY
ls -la [spec-path]/planning/visuals/ 2>/dev/null | grep -E '\.(png|jpg|jpeg|gif|svg|pdf)$' || echo "No visual files found"
```

3. IF visual files are found (bash command returns filenames):
   - Use Read tool to analyze EACH visual file found
   - Note key design elements, patterns, and user flows
   - Document observations for each file
   - Check filenames for low-fidelity indicators (lofi, lo-fi, wireframe, sketch, rough, etc.)

4. IF user provided paths or names of similar features:
   - Make note of these paths/names for spec-writer to reference
   - DO NOT explore them yourself (to save time), but DO document their names for future reference by the spec-writer.

### Step 6: Check for Checkpoints Before Big Changes (SDD-aligned)

After processing answers, check if any big changes or abstraction layer transitions are detected:

```bash
# Check for big changes or layer transitions
{{workflows/human-review/create-checkpoint}}
```

If a checkpoint is needed, present it to the user and wait for their confirmation before proceeding.

**SDD Checkpoint: Spec Completeness Before Task Decomposition (Conditional)**

As part of SDD best practices, validate spec completeness before proceeding to task decomposition:

```bash
# Conditionally check if spec completeness validation is needed
# Only trigger if it would add value and doesn't create unnecessary friction
SPEC_COMPLETE_CHECK_NEEDED="false"

# Check if spec has clear requirements, acceptance criteria, and scope boundaries
if [ -f "$SPEC_PATH/planning/requirements.md" ]; then
    # Check for user stories format
    HAS_USER_STORIES=$(grep -i "as a.*i want.*so that" "$SPEC_PATH/planning/requirements.md" | wc -l)
    
    # Check for acceptance criteria
    HAS_ACCEPTANCE_CRITERIA=$(grep -i "acceptance criteria\|acceptance criterion" "$SPEC_PATH/planning/requirements.md" | wc -l)
    
    # Check for scope boundaries
    HAS_SCOPE_BOUNDARIES=$(grep -i "in scope\|out of scope\|scope boundary" "$SPEC_PATH/planning/requirements.md" | wc -l)
    
    # Only trigger checkpoint if key SDD elements are missing AND it would be useful
    if [ "$HAS_USER_STORIES" -eq 0 ] || [ "$HAS_ACCEPTANCE_CRITERIA" -eq 0 ] || [ "$HAS_SCOPE_BOUNDARIES" -eq 0 ]; then
        SPEC_COMPLETE_CHECK_NEEDED="true"
    fi
fi

if [ "$SPEC_COMPLETE_CHECK_NEEDED" = "true" ]; then
    echo "🔍 SDD Checkpoint: Spec Completeness Validation"
    echo ""
    echo "Before proceeding to task decomposition (SDD phase order: Specify → Tasks),"
    echo "let's ensure the specification is complete with:"
    echo "  - Clear user stories"
    echo "  - Explicit acceptance criteria"
    echo "  - Defined scope boundaries (in-scope vs out-of-scope)"
    echo ""
    echo "Should we review and enhance the requirements before creating tasks?"
    echo "This ensures the 'Specify' phase is complete before the 'Tasks' phase (SDD best practice)."
    echo ""
    echo "Reply: [Yes/No/Proceed anyway]"
    
    # Wait for user confirmation (implementation will handle this in actual execution)
fi
```

**Note:** This checkpoint is conditional and only triggers when it would add meaningful value. It follows SDD principle: "Specify" phase should be complete before "Tasks" phase, ensuring spec is the source of truth.

### Step 7: Generate Follow-up Questions (if needed)

Determine if follow-up questions are needed based on:

**Visual-triggered follow-ups:**
- If visuals were found but user didn't mention them: "I found [filename(s)] in the visuals folder. Let me analyze these for the specification."
- If filenames contain "lofi", "lo-fi", "wireframe", "sketch", or "rough": "I notice you've provided [filename(s)] which appear to be wireframes/low-fidelity mockups. Should we treat these as layout and structure guides rather than exact design specifications, using our application's existing styling instead?"
- If visuals show features not discussed in answers
- If there are discrepancies between answers and visuals

**Reusability follow-ups:**
   - If user didn't provide similar features but the spec seems common: "This seems like it might share patterns with existing features. Could you point me to any similar functionality or logic in your codebase?"
- If provided paths seem incomplete you can ask something like: "You mentioned [feature]. Are there any related modules or logic we should also reference?"

**User's Answers-triggered follow-ups:**
- Vague requirements need clarification
- Missing technical details
- Unclear scope boundaries

**If follow-ups needed, OUTPUT to orchestrator:**
```
Based on your answers [and the visual files I found], I have a few follow-up questions:

1. [Specific follow-up question]
2. [Another follow-up if needed]

Please provide these additional details.
```

**Then STOP and wait for responses.**

### Step 8: Save Complete Requirements

After all questions are answered, record ALL gathered information to ONE FILE at this location with this name: `[spec-path]/planning/requirements.md`

Use the following structure and do not deviate from this structure when writing your gathered information to `requirements.md`.  Include ONLY the items specified in the following structure:

```markdown
# Spec Requirements: [Spec Name]

## Initial Description
[User's original spec description from initialization.md]

## Requirements Discussion

### First Round Questions

**Q1:** [First question asked]
**Answer:** [User's answer]

**Q2:** [Second question asked]
**Answer:** [User's answer]

[Continue for all questions]

### Existing Code to Reference
[Based on user's response about similar features]

**Similar Features Identified:**
- Feature: [Name] - Path: `[path provided by user]`
- Components or modules to potentially reuse: [user's description]
- Related logic to reference: [user's description]

[If user provided no similar features]
No similar existing features identified for reference.

### Follow-up Questions
[If any were asked]

**Follow-up 1:** [Question]
**Answer:** [User's answer]

## Visual Assets

### Files Provided:
[Based on actual bash check, not user statement]
- `filename.png`: [Description of what it shows from your analysis]
- `filename2.jpg`: [Key elements observed from your analysis]

### Visual Insights:
- [Design patterns identified]
- [User flow implications]
- [UI components shown]
- [Fidelity level: high-fidelity mockup / low-fidelity wireframe]

[If bash check found no files]
No visual assets provided.

## Requirements Summary

### Functional Requirements
- [Core functionality based on answers]
- [User actions enabled]
- [Data to be managed]

### Reusability Opportunities
- [Components or modules that might exist already based on user's input]
- [Code patterns to investigate]
- [Similar features to model after]

### Scope Boundaries
**In Scope:**
- [What will be built]

**Out of Scope:**
- [What won't be built]
- [Future enhancements mentioned]

### Technical Considerations
- [Integration points mentioned]
- [Existing system constraints]
- [Technical preferences or constraints stated]
- [Similar code patterns to follow]
```

### Step 9: Output Completion

Return to orchestrator:

```
Requirements research complete!

✅ Processed [X] clarifying questions
✅ Visual check performed: [Found and analyzed Y files / No files found]
✅ Reusability opportunities: [Identified Z similar features / None identified]
✅ Requirements documented comprehensively

Requirements saved to: `[spec-path]/planning/requirements.md`

Ready for specification creation.
```

## Important Constraints

- **MANDATORY**: Always run bash command to check visuals folder after receiving user answers
- DO NOT write technical specifications for development. Just record your findings from information gathering to this single file: `[spec-path]/planning/requirements.md`.
- Visual check is based on actual file(s) found via bash, NOT user statements
- Check filenames for low-fidelity indicators and clarify design intent if found
- Ask about existing similar features to promote code reuse
- Keep follow-ups minimal (1-3 questions max)
- Save user's exact answers, not interpretations
- Document all visual findings including fidelity level
- Document paths to similar features for spec-writer to reference
- OUTPUT questions and STOP to wait for orchestrator to relay responses

## SDD Integration Notes

This workflow has been enhanced with Spec-Driven Development (SDD) best practices:

**SDD Principles Integrated:**
- **Specification as Source of Truth**: Questions ensure specs are actionable and will be referenced
- **Minimal, Clear Specs**: Questions encourage intentional scoping and avoid feature bloat
- **SDD Phase Order**: Conditional checkpoint validates "Specify" phase is complete before "Tasks" phase

**SDD-Aware Question Generation:**
- Questions ensure user stories are captured in standard format
- Questions validate acceptance criteria will be explicit
- Questions check for explicit scope boundaries
- Questions avoid leading to premature technical details (What & Why, not How in spec phase)

**SDD Anti-Pattern Prevention:**
- Questions help avoid specification theater (specs that are written but never referenced)
- Questions prevent premature comprehensiveness (trying to spec everything upfront)
- Questions discourage over-engineering (excessive technical detail too early)

**Technology-Agnostic Approach (Default Profile Templates Only):**
- All SDD framework references are abstract (e.g., "task decomposition frameworks" not technology-specific tools)
- No hardcoded technology-specific SDD tool references in default templates
- Questions maintain technology-agnostic state throughout **in default profile templates**
- **After Specialization:** When templates are compiled to `geist/workflows/`, workflows can and should become technology-specific based on the project's actual stack
- **Command Outputs:** Specs, tasks, and implementations should reflect the project's actual technology stack

**Conditional SDD Checkpoints:**
- Spec completeness checkpoint only triggers when it would add value
- Checkpoints follow existing human-review workflow patterns
- Checkpoints don't create unnecessary friction
