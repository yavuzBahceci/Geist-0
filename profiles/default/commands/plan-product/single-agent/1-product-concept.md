This begins a multi-step process for planning and documenting the mission and roadmap for the current product.

---

## ⚠️ CHECKPOINT - USER INTERACTION REQUIRED

The FIRST STEP is to gather product information from the user.

**Read and follow the workflow instructions in:** `@geist/workflows/planning/gather-product-info.md`

**IMPORTANT:** This step requires user input. You MUST:
1. Ask the user for product information (idea, features, target users, tech stack)
2. **STOP and WAIT** for the user to respond
3. Do NOT proceed until you have gathered the required information
4. Do NOT make assumptions about the product without asking

---

After gathering product information, WAIT for the user to give you specific instructions on how to use the information you've gathered to create the mission and roadmap.

{{UNLESS compiled_single_command}}
## Display confirmation and next step

Once you've gathered all of the necessary information from the user, output the following message:

```
I have all the info I need to help you plan this product.

NEXT STEP 👉 Run the command, `2-create-mission.md`
```
{{ENDUNLESS compiled_single_command}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

When planning the product's tech stack, mission statement and roadmap, use the user's standards and preferences for context and baseline assumptions, as documented in these files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
