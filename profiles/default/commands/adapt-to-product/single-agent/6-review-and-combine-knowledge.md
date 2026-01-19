Now that all product documents have been created, review and combine the knowledge to ensure consistency and completeness.

## Step 1: Load All Product Documents

Read and analyze all created product documents:

```bash
# Load all product files
MISSION=$(cat geist/product/mission.md 2>/dev/null)
ROADMAP=$(cat geist/product/roadmap.md 2>/dev/null)
TECH_STACK=$(cat geist/product/tech-stack.md 2>/dev/null)

# Verify all files exist
if [ -z "$MISSION" ] || [ -z "$ROADMAP" ] || [ -z "$TECH_STACK" ]; then
    echo "⚠️  Warning: Some product files are missing"
fi
```

## Step 2: Verify Consistency Between Documents

Analyze the documents together to verify consistency:

1. **Mission-Roadmap Alignment**: Verify roadmap features support the mission goals
2. **Roadmap-Tech Stack Alignment**: Verify tech stack supports roadmap features
3. **Mission-Tech Stack Alignment**: Verify tech choices align with product vision

Check for:
- Contradictions between documents
- Missing connections between mission goals and roadmap items
- Tech stack gaps that would block roadmap features
- Terminology consistency across all documents

## Step 3: Identify Gaps or Issues

Document any issues found:
- Missing information that should be added
- Inconsistencies that need resolution
- Questions that need user clarification

---

### ⚠️ CHECKPOINT - USER INTERACTION REQUIRED (if issues found)

**IF any gaps, inconsistencies, or questions are identified:**
1. Present all issues clearly to the user
2. **STOP and WAIT** for the user to provide clarification or resolution
3. Do NOT proceed until issues are resolved

**IF no issues are found**, you may proceed automatically.

---

## Step 4: Create Product Summary Report

Create a summary document that combines insights from all product files:

```bash
mkdir -p geist/output/adapt-to-product/reports

cat > geist/output/adapt-to-product/reports/product-summary.md << 'EOF'
# Product Documentation Summary

## Documents Created
- `geist/product/mission.md` - Product mission and vision
- `geist/product/roadmap.md` - Development roadmap and phases
- `geist/product/tech-stack.md` - Technical stack documentation

## Consistency Analysis

### Mission-Roadmap Alignment
[Summary of how roadmap features support mission goals]

### Tech Stack Coverage
[Summary of how tech stack supports all planned features]

### Cross-Document Consistency
[Summary of terminology and concept consistency]

## Identified Gaps (if any)
[List any gaps or missing information]

## Recommendations
[Suggestions for improvements or additions]

## Next Steps
Ready to proceed with codebase analysis and basepoints generation.
EOF
```

{{UNLESS compiled_single_command}}
## Display confirmation and next step

Once you've reviewed and combined the knowledge, output the following message:

```
✅ I have reviewed and combined all product knowledge.

**Consistency Check Results:**
- Mission-Roadmap alignment: [status]
- Tech stack coverage: [status]
- Cross-document consistency: [status]

**Summary report created:** `geist/output/adapt-to-product/reports/product-summary.md`

NEXT STEP 👉 Run the command, `7-navigate-to-next-command.md`
```
{{ENDUNLESS compiled_single_command}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

Ensure the review process follows the user's standards and preferences as documented in these files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
