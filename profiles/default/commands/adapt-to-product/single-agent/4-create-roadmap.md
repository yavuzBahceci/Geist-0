Now that you've created this product's mission.md, use that to guide your creation of the roadmap in `geist/product/roadmap.md` by following these instructions:

**Read and follow the workflow instructions in:** `@geist/workflows/planning/create-product-roadmap.md`

---

### ⚠️ CHECKPOINT - USER REVIEW RECOMMENDED

After creating the roadmap document:
1. Present a summary of the roadmap phases to the user
2. Ask: "Does this roadmap align with your development priorities?"
3. **WAIT** for user confirmation or feedback
4. If the user requests changes, update the document accordingly

---

{{UNLESS compiled_single_command}}
## Display confirmation and next step

Once you've created roadmap.md, output the following message:

```
✅ I have documented the product roadmap at `geist/product/roadmap.md`.

Review it to ensure it aligns with how you see this product roadmap going forward.

NEXT STEP 👉 Run the command, `5-create-tech-stack.md`
```
{{ENDUNLESS compiled_single_command}}

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure the product roadmap is ALIGNED and DOES NOT CONFLICT with the user's preferences and standards as detailed in the following files:

{{standards/global/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
