# Workflow: Present and Confirm

## Purpose

Format detected values for display, present a confirmation prompt to the user, handle user overrides, and output the final confirmed profile.

---

## ⚠️ CRITICAL: USER INTERACTION REQUIRED

**This workflow requires user confirmation.** You MUST:
1. Display the detected configuration clearly
2. **STOP and WAIT** for the user to confirm or request changes
3. Do NOT proceed until the user explicitly confirms
4. Do NOT assume the user accepts the defaults

## Inputs

Expects these variables to be set (from prior detection workflows):
- `DETECTED_PROJECT_TYPE`
- `DETECTED_LANGUAGE`
- `DETECTED_FRAMEWORK`
- `DETECTED_DATABASE`
- `DETECTED_BUILD_CMD`
- `DETECTED_TEST_CMD`
- `DETECTED_LINT_CMD`
- `DETECTED_SECURITY_LEVEL`
- `DETECTED_FILE_COUNT`
- `DETECTED_LINE_COUNT`
- `DETECTION_CONFIDENCE`

## Outputs

- Updates `geist/config/project-profile.yml` with confirmed values
- Sets `USER_CONFIRMED=true` when user accepts

---

## Workflow

### Step 1: Display Detected Configuration

```bash
echo ""
echo "═══════════════════════════════════════════════════════════════════════════"
echo "                      DETECTED PROJECT CONFIGURATION                         "
echo "═══════════════════════════════════════════════════════════════════════════"
echo ""

# Project Profile Section
echo "📦 PROJECT PROFILE"
echo "─────────────────────────────────────────────────────────────────────────────"
printf "   %-20s %s\n" "Type:" "${DETECTED_PROJECT_TYPE:-unknown} ✓"
printf "   %-20s %s\n" "Size:" "${DETECTED_FILE_COUNT:-?} files, ~${DETECTED_LINE_COUNT:-?} lines ✓"
printf "   %-20s %s\n" "Maturity:" "$([ -d '.git' ] && echo 'Version controlled' || echo 'Unknown') ✓"
echo ""

# Tech Stack Section
echo "🔧 TECH STACK"
echo "─────────────────────────────────────────────────────────────────────────────"
printf "   %-20s %s\n" "Language:" "${DETECTED_LANGUAGE:-unknown} ✓"
[ -n "$DETECTED_FRAMEWORK" ] && printf "   %-20s %s\n" "Framework:" "$DETECTED_FRAMEWORK ✓"
[ -n "$DETECTED_BACKEND" ] && printf "   %-20s %s\n" "Backend:" "$DETECTED_BACKEND ✓"
[ -n "$DETECTED_DATABASE" ] && printf "   %-20s %s\n" "Database:" "$DETECTED_DATABASE ✓"
echo ""

# Commands Section
echo "⚙️  DETECTED COMMANDS"
echo "─────────────────────────────────────────────────────────────────────────────"
printf "   %-20s %s\n" "Build:" "${DETECTED_BUILD_CMD:-(not detected)} ✓"
printf "   %-20s %s\n" "Test:" "${DETECTED_TEST_CMD:-(not detected)} ✓"
printf "   %-20s %s\n" "Lint:" "${DETECTED_LINT_CMD:-(not detected)} ✓"
echo ""

# Inferred Settings Section
echo "🔒 INFERRED SETTINGS"
echo "─────────────────────────────────────────────────────────────────────────────"
printf "   %-20s %s\n" "Security Level:" "${DETECTED_SECURITY_LEVEL:-moderate} ✓"
printf "   %-20s %s\n" "Complexity:" "${INFERRED_COMPLEXITY:-moderate} ✓"
echo ""

# Confidence
echo "📊 DETECTION CONFIDENCE: ${DETECTION_CONFIDENCE:-0.80}"
echo ""
echo "═══════════════════════════════════════════════════════════════════════════"
```

### Step 2: Present Minimal Questions

Only ask questions for things that cannot be detected:

```bash
echo ""
echo "⚠️  QUESTIONS REQUIRING YOUR INPUT"
echo "─────────────────────────────────────────────────────────────────────────────"
echo ""
echo "These items cannot be determined automatically from your codebase:"
echo ""

# Question 1: Compliance Requirements
echo "1. What compliance requirements apply to this project?"
echo ""
echo "   [ ] None (default)"
echo "   [ ] SOC 2"
echo "   [ ] HIPAA"
echo "   [ ] GDPR"
echo "   [ ] PCI-DSS"
echo "   [ ] Other"
echo ""
echo "   Enter your choice (e.g., 'none', 'gdpr', 'soc2,hipaa'): "

# In non-interactive mode or if user presses Enter, use default
USER_COMPLIANCE="${USER_COMPLIANCE:-none}"

echo ""

# Question 2: Human Review Level
echo "2. How much human oversight do you want for AI-generated changes?"
echo ""
echo "   [ ] minimal  - Trust AI, review only critical changes"
echo "   [ ] moderate - Review architectural decisions (default)"
echo "   [ ] high     - Review all significant changes"
echo ""
echo "   Enter your choice (minimal/moderate/high): "

# Default to moderate
USER_HUMAN_REVIEW="${USER_HUMAN_REVIEW:-moderate}"

echo ""
echo "═══════════════════════════════════════════════════════════════════════════"
```

### Step 3: Confirmation Prompt

```bash
echo ""
echo "📋 CONFIRMATION"
echo "─────────────────────────────────────────────────────────────────────────────"
echo ""
echo "Press Enter to accept these detected values, or type a section name to modify:"
echo ""
echo "   • 'type'      - Change project type"
echo "   • 'language'  - Change language/framework"
echo "   • 'commands'  - Change build/test/lint commands"
echo "   • 'security'  - Change security level"
echo "   • 'all'       - Review all settings interactively"
echo ""
echo "Your choice (Enter to accept): "

# For non-interactive execution, assume acceptance
USER_CHOICE="${USER_CHOICE:-accept}"

if [ "$USER_CHOICE" = "" ] || [ "$USER_CHOICE" = "accept" ]; then
    echo ""
    echo "✅ Configuration confirmed!"
    USER_CONFIRMED="true"
fi
```

### Step 4: Handle Overrides (if requested)

```bash
# Handle override requests
case "$USER_CHOICE" in
    "type")
        echo "Enter new project type (web_app, cli, api, library, monorepo):"
        read -r NEW_TYPE
        [ -n "$NEW_TYPE" ] && DETECTED_PROJECT_TYPE="$NEW_TYPE"
        ;;
    "language")
        echo "Enter primary language (typescript, javascript, rust, python, go):"
        read -r NEW_LANG
        [ -n "$NEW_LANG" ] && DETECTED_LANGUAGE="$NEW_LANG"
        echo "Enter framework (or leave empty):"
        read -r NEW_FRAMEWORK
        DETECTED_FRAMEWORK="$NEW_FRAMEWORK"
        ;;
    "commands")
        echo "Enter build command (or leave empty):"
        read -r NEW_BUILD
        DETECTED_BUILD_CMD="$NEW_BUILD"
        echo "Enter test command (or leave empty):"
        read -r NEW_TEST
        DETECTED_TEST_CMD="$NEW_TEST"
        echo "Enter lint command (or leave empty):"
        read -r NEW_LINT
        DETECTED_LINT_CMD="$NEW_LINT"
        ;;
    "security")
        echo "Enter security level (low, moderate, high):"
        read -r NEW_SECURITY
        [ -n "$NEW_SECURITY" ] && DETECTED_SECURITY_LEVEL="$NEW_SECURITY"
        ;;
esac
```

### Step 5: Update Profile with Confirmed Values

```bash
# Update the profile with user-specified values
cat > geist/config/project-profile.yml << CONFIRMED_EOF
# Project Profile
# Auto-detected and confirmed by user
# Generated: $(date -Iseconds)

gathered:
  project_type: ${DETECTED_PROJECT_TYPE:-unknown}
  tech_stack:
    language: ${DETECTED_LANGUAGE:-unknown}
    framework: ${DETECTED_FRAMEWORK:-}
    backend: ${DETECTED_BACKEND:-}
    database: ${DETECTED_DATABASE:-}
  size:
    lines: ${DETECTED_LINE_COUNT:-0}
    files: ${DETECTED_FILE_COUNT:-0}
    modules: ${DETECTED_MODULE_COUNT:-0}
  commands:
    build: "${DETECTED_BUILD_CMD:-}"
    test: "${DETECTED_TEST_CMD:-}"
    lint: "${DETECTED_LINT_CMD:-}"

inferred:
  security_level: ${DETECTED_SECURITY_LEVEL:-moderate}
  complexity: ${INFERRED_COMPLEXITY:-moderate}

user_specified:
  compliance:
$(echo "$USER_COMPLIANCE" | tr ',' '\n' | sed 's/^/    - /' | grep -v "^    - none$" || echo "    []")
  human_review_level: ${USER_HUMAN_REVIEW:-moderate}

_meta:
  detected_at: $(date -Iseconds)
  confirmed_at: $(date -Iseconds)
  detection_confidence: ${DETECTION_CONFIDENCE:-0.80}
  user_confirmed: ${USER_CONFIRMED:-true}
  questions_asked: 2
  questions_auto_answered: ${DETECTIONS_SUCCESS:-0}

CONFIRMED_EOF

echo ""
echo "✅ Profile saved to: geist/config/project-profile.yml"
echo ""
```

---

## Output Summary

After confirmation, display:

```bash
echo "═══════════════════════════════════════════════════════════════════════════"
echo "                        CONFIGURATION COMPLETE                               "
echo "═══════════════════════════════════════════════════════════════════════════"
echo ""
echo "Your project profile has been saved and will be used for:"
echo ""
echo "  • Basepoint generation (create-basepoints)"
echo "  • Agent specialization (deploy-agents)"
echo "  • Validation command configuration"
echo "  • Workflow complexity selection"
echo ""
echo "You can modify the profile at any time by editing:"
echo "  geist/config/project-profile.yml"
echo ""
echo "═══════════════════════════════════════════════════════════════════════════"
```

---

## Important Constraints

- Default to accepting detected values (user presses Enter)
- Maximum 2-3 questions that can't be auto-detected
- Provide sensible defaults for all questions
- Allow overrides but don't require them
- Save confirmed profile for use by subsequent commands
