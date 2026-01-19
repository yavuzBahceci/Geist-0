# Gather Product Information

Collect comprehensive product information from the user for planning a new product.

---

## ⚠️ CRITICAL: USER INTERACTION REQUIRED

**This workflow requires user input.** You MUST:
1. Present the questions below to the user
2. **STOP and WAIT** for the user to respond
3. Do NOT proceed with assumptions or defaults
4. Do NOT skip questions - all information is required

---

## Step 1: Create Documentation Folders

Create folders for user-provided documents and resources:

```bash
mkdir -p geist/product/inheritance
mkdir -p geist/product/docs
```

- `geist/product/inheritance/` - For files that inform product documentation
- `geist/product/docs/` - For external resources, links, and research materials

## Step 2: Check Existing Product Documentation

```bash
if [ -d "geist/product" ]; then
    echo "Product documentation already exists."
    ls -la geist/product/
fi
```

If product files already exist, ask the user: "Product documentation already exists. Should I use the existing files or start fresh?"

**WAIT for user response before proceeding.**

---

## Step 3: Gather Required Information

### ⚠️ CHECKPOINT - ASK THESE QUESTIONS AND WAIT FOR RESPONSES

Present these questions to the user:

```
═══════════════════════════════════════════════════════════════════════════
                    PRODUCT INFORMATION GATHERING
═══════════════════════════════════════════════════════════════════════════

I need some information about your product to create accurate documentation.
Please answer the following questions:

1. **Product Idea / Core Concept** (required)
   What is the main idea for this product? What problem does it solve?

2. **Key Features** (minimum 3 required)
   What are the main features or capabilities of this product?
   Please list at least 3 features with brief descriptions.

3. **Target Users** (minimum 1 required)
   Who are the primary users of this product?
   Please describe at least 1 user segment with their use cases.

4. **Tech Stack** (required)
   Will this product use your usual tech stack choices or deviate in any way?
   Please confirm or specify any tech stack preferences.

5. **Documentation & Resources** (optional but recommended)
   
   📎 **Links you can provide:**
   - Reference products or competitors to learn from
   - Design inspiration or UI examples
   - Technical documentation or tutorials
   - Industry articles or best practices
   
   📁 **Files you can place in `geist/product/docs/`:**
   - PRDs or feature specs
   - Wireframes or mockups
   - User research or personas
   - Competitor analysis
   - Any planning documents
   
   📁 **Files you can place in `geist/product/inheritance/`:**
   - Existing documentation to build upon
   - Brand guidelines
   - Technical constraints or requirements

6. **Web Research Suggestions** (optional)
   Would you like me to research any of the following?
   
   - Similar products in the market
   - Best practices for your product category
   - Technical patterns for your tech stack
   - Industry trends
   
   If yes, provide keywords or topics to search.

═══════════════════════════════════════════════════════════════════════════
```

**IMPORTANT:** After presenting these questions:
- **STOP** and wait for the user to respond
- Do NOT assume answers or use defaults
- Do NOT proceed until the user provides all required information

---

## Step 4: Validate Responses

After receiving user responses, verify you have:

- [ ] Product idea/concept clearly defined
- [ ] At least 3 key features listed
- [ ] At least 1 target user segment identified
- [ ] Tech stack confirmed or specified

If any required information is missing, prompt the user:

```
Please provide the following missing information:
1. [List missing items]
```

**WAIT for user to provide missing information before proceeding.**

---

## Step 5: Process Documentation Folders

Check for user-provided files:

```bash
# Check docs folder
if [ -d "geist/product/docs" ] && [ "$(ls -A geist/product/docs 2>/dev/null)" ]; then
    echo "📁 Found files in geist/product/docs/:"
    ls -la geist/product/docs/
fi

# Check inheritance folder
if [ -d "geist/product/inheritance" ] && [ "$(ls -A geist/product/inheritance 2>/dev/null)" ]; then
    echo "📁 Found files in geist/product/inheritance/:"
    ls -la geist/product/inheritance/
fi
```

For each file found, read and extract relevant information.

---

## Step 6: Execute Web Research (if requested)

If the user provided web research suggestions:

1. Search for each keyword/topic
2. Research competitors if specified
3. Gather technical best practices
4. Save results to `geist/product/docs/research-notes.md`

---

## Step 7: Confirm Information

Once all information is gathered, present a summary to the user:

```
═══════════════════════════════════════════════════════════════════════════
                    PRODUCT INFORMATION SUMMARY
═══════════════════════════════════════════════════════════════════════════

📦 Product: [product name/concept]
✨ Features: [list features]
👥 Target Users: [list user segments]
🔧 Tech Stack: [tech stack details]

📚 Sources Used:
   - User responses: ✅
   - Files in docs/: [count] files
   - Files in inheritance/: [count] files
   - Links processed: [count]
   - Web research: [completed/skipped]

Is this information correct? (yes/no)
═══════════════════════════════════════════════════════════════════════════
```

**WAIT for user confirmation before proceeding to the next phase.**
