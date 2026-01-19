# Workflow: Question Templates

## Purpose

Provide minimal question templates for information that cannot be automatically detected from the codebase. Maximum 2-3 questions per command.

---

## ⚠️ CRITICAL: USER INTERACTION REQUIRED

**This workflow requires user responses.** You MUST:
1. Present the questions clearly to the user
2. **STOP and WAIT** for the user to respond
3. Do NOT use defaults without asking the user first
4. Do NOT proceed until the user provides answers

---

## Questions That MUST Be Asked

These questions gather information that cannot be detected from code:

### Question 1: Compliance Requirements

```markdown
**1. Compliance Requirements** (can't detect from code)

What compliance standards apply to this project?

   [ ] None (default)
   [ ] SOC 2 - Security/availability controls
   [ ] HIPAA - Healthcare data protection
   [ ] GDPR - EU data privacy
   [ ] PCI-DSS - Payment card data
   [ ] Other (specify)

Enter your choice (e.g., 'none', 'gdpr', 'soc2,hipaa'):
```

**Default:** `none`

### Question 2: Human Review Preference

```markdown
**2. Human Review Preference**

How much human oversight do you want for AI-generated changes?

   [ ] minimal  - Trust AI, review only critical changes
   [ ] moderate - Review architectural decisions (default)
   [ ] high     - Review all significant changes

Enter your choice (minimal/moderate/high):
```

**Default:** `moderate`

---

## Questions Asked Only If Detection Fails

These are only asked when automatic detection confidence is below 70%:

### Question 3: Project Type (if ambiguous)

```markdown
**3. Project Type** (detection was uncertain)

I couldn't confidently determine the project type. Is this a:

   [ ] web_app  - Web application
   [ ] api      - API/Backend service
   [ ] cli      - Command-line tool
   [ ] library  - Reusable library
   [ ] monorepo - Multiple packages
   [ ] other    - Something else

Enter your choice:
```

### Question 4: Module Boundaries (if unclear)

```markdown
**4. Module Boundaries** (structure was unclear)

How should I identify module boundaries for basepoint generation?

   [ ] directory  - Each top-level directory is a module
   [ ] package    - Each package.json/Cargo.toml is a module
   [ ] manual     - Let me specify the modules

Enter your choice:
```

---

## Implementation

### Bash Script for Questions

```bash
# Function to ask compliance question
ask_compliance() {
    echo ""
    echo "1. What compliance requirements apply to this project?"
    echo ""
    echo "   [ ] None (default)"
    echo "   [ ] SOC 2"
    echo "   [ ] HIPAA"
    echo "   [ ] GDPR"
    echo "   [ ] PCI-DSS"
    echo "   [ ] Other"
    echo ""
    echo "   Enter your choice (e.g., 'none', 'gdpr', 'soc2,hipaa'):"
    
    # In non-interactive mode, use default
    USER_COMPLIANCE="${USER_COMPLIANCE:-none}"
    echo "   → Using: $USER_COMPLIANCE"
}

# Function to ask human review preference
ask_human_review() {
    echo ""
    echo "2. How much human oversight do you want for AI changes?"
    echo ""
    echo "   [ ] minimal  - Trust AI, review only critical"
    echo "   [ ] moderate - Review architectural decisions (default)"
    echo "   [ ] high     - Review all significant changes"
    echo ""
    echo "   Enter your choice (minimal/moderate/high):"
    
    # In non-interactive mode, use default
    USER_HUMAN_REVIEW="${USER_HUMAN_REVIEW:-moderate}"
    echo "   → Using: $USER_HUMAN_REVIEW"
}

# Function to ask project type (only if detection failed)
ask_project_type() {
    if [ "$NEEDS_USER_INPUT_TYPE" = "true" ]; then
        echo ""
        echo "3. I couldn't determine the project type. Is this a:"
        echo ""
        echo "   [ ] web_app  - Web application"
        echo "   [ ] api      - API/Backend service"
        echo "   [ ] cli      - Command-line tool"
        echo "   [ ] library  - Reusable library"
        echo "   [ ] monorepo - Multiple packages"
        echo ""
        echo "   Enter your choice:"
        
        # In non-interactive mode, use unknown
        USER_PROJECT_TYPE="${USER_PROJECT_TYPE:-unknown}"
        DETECTED_PROJECT_TYPE="$USER_PROJECT_TYPE"
    fi
}

# Main question flow
ask_minimal_questions() {
    echo ""
    echo "═══════════════════════════════════════════════════════"
    echo "  QUESTIONS REQUIRING YOUR INPUT"
    echo "═══════════════════════════════════════════════════════"
    echo ""
    echo "These items cannot be determined from your codebase:"
    
    ask_compliance
    ask_human_review
    
    # Only ask additional questions if detection was uncertain
    ask_project_type
    
    echo ""
    echo "═══════════════════════════════════════════════════════"
    echo ""
    echo "Press Enter to accept these values, or re-run with"
    echo "environment variables to override:"
    echo ""
    echo "   USER_COMPLIANCE=gdpr USER_HUMAN_REVIEW=high /adapt-to-product"
    echo ""
}

# Execute questions
ask_minimal_questions
```

---

## Storing Answers

After questions are answered, update the project profile:

```bash
# Update project profile with user answers
update_profile_with_answers() {
    PROFILE_FILE="geist/config/project-profile.yml"
    
    if [ -f "$PROFILE_FILE" ]; then
        # Update user_specified section
        # Note: In practice, use yq or proper YAML parsing
        
        echo ""
        echo "Updating profile with your preferences..."
        echo "   Compliance: $USER_COMPLIANCE"
        echo "   Human Review: $USER_HUMAN_REVIEW"
        
        # The profile should already exist from detection
        # This just updates the user_specified section
    fi
}
```

---

## Question Count by Command

| Command | Questions Asked | Conditional Questions |
|---------|-----------------|----------------------|
| `/adapt-to-product` | 2 | +1 if type detection failed |
| `/create-basepoints` | 0-1 | Only if modules unclear |
| `/deploy-agents` | 0-2 | Only if not set in prior command |

**Total across all commands:** Maximum 3-5 questions (vs 20+ traditional approach)

---

## Important Constraints

- Maximum 2-3 questions per command
- All questions have sensible defaults
- User can press Enter to accept defaults
- Questions are skipped if already answered in prior command
- Additional questions only asked when detection confidence is low
