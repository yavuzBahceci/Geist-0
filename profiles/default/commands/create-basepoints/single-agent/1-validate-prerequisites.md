The FIRST STEP is to validate prerequisites and load/update project profile.

## Step 0: Load or Update Project Profile (NEW)

Before validating prerequisites, load existing project profile or detect missing information:

```bash
echo "═══════════════════════════════════════════════════════"
echo "  PROJECT PROFILE CHECK"
echo "═══════════════════════════════════════════════════════"
echo ""

# Check for existing project profile
if [ -f "geist/config/project-profile.yml" ]; then
    echo "✅ Found existing project profile: geist/config/project-profile.yml"
    
    # Load key values
    DETECTED_LANGUAGE=$(grep "language:" geist/config/project-profile.yml | head -1 | awk '{print $2}')
    DETECTED_FRAMEWORK=$(grep "framework:" geist/config/project-profile.yml | head -1 | awk '{print $2}')
    PROJECT_TYPE=$(grep "project_type:" geist/config/project-profile.yml | head -1 | awk '{print $2}')
    
    echo "   Language: $DETECTED_LANGUAGE"
    echo "   Framework: ${DETECTED_FRAMEWORK:-(none)}"
    echo "   Project Type: $PROJECT_TYPE"
    echo ""
    
    PROFILE_EXISTS=true
else
    echo "ℹ️  No project profile found. Running detection..."
    echo ""
    
    # Run detection for missing profile
    {{workflows/detection/detect-project-profile}}
    
    PROFILE_EXISTS=false
fi
```

### 0.1: Architecture-Specific Research (if profile exists)

If profile exists, run additional architecture-focused research:

```bash
if [ "$PROFILE_EXISTS" = "true" ]; then
    # Check if enriched knowledge already exists
    if [ -d "geist/config/enriched-knowledge" ]; then
        echo "✅ Enriched knowledge found"
        
        # Run additional architecture research for basepoints context
        if [ ! -f "geist/config/enriched-knowledge/stack-best-practices.md" ]; then
            echo "   Adding architecture patterns research..."
            RESEARCH_DEPTH="standard"
            DO_STACK_PATTERNS=true
            DO_LIBRARY_RESEARCH=false
            DO_SECURITY_RESEARCH=false
            DO_DOMAIN_RESEARCH=false
            {{workflows/research/research-stack-patterns}}
        fi
    else
        echo "ℹ️  Running knowledge enrichment..."
        RESEARCH_DEPTH="standard"
        {{workflows/research/research-orchestrator}}
    fi
fi
echo ""
```

---

## Step 1: Validate Prerequisites

{{workflows/codebase-analysis/validate-prerequisites}}

## Display confirmation and next step

Once prerequisites are validated, output the following message:

```
✅ Prerequisites validated!

- Product files: Found (mission.md, roadmap.md, tech-stack.md)
- Basepoints folder: [Ready / Created / Backed up]
- Project Profile: [Found / Generated]
- Enriched Knowledge: [Found / Generated]

NEXT STEP 👉 Run the command, `2-detect-abstraction-layers.md`
```

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that your validation process aligns with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
