# Gather Product Information from Existing Codebase

Collect comprehensive product information from multiple sources for adapting an existing codebase into product documentation.

---

## ⚠️ CRITICAL: USER INTERACTION REQUIRED

**This workflow requires user input.** You MUST:
1. Ask the questions listed below
2. **STOP and WAIT** for user responses
3. Do NOT proceed with assumptions or defaults without user confirmation
4. Do NOT skip to creating product files until you have gathered information from the user

---

## Step 1: Create Documentation Folders

Create folders for user-provided documents and resources:

```bash
mkdir -p geist/product/inheritance
mkdir -p geist/product/docs
```

- `geist/product/inheritance/` - For files that inform product documentation (PRDs, specs, designs)
- `geist/product/docs/` - For external resources, links, and research materials

## Step 2: Check Existing Product Documentation

Check if product folder already exists:

```bash
if [ -d "geist/product" ]; then
    ls -la geist/product/
fi
```

If product files already exist, ask the user: "Product documentation already exists. Should I use the existing files or start fresh?"

---

## Step 3: Gather Information from User

### ⚠️ CHECKPOINT - ASK THESE QUESTIONS AND WAIT FOR RESPONSES

Present these questions to the user and **WAIT for their responses**:

```
═══════════════════════════════════════════════════════════════════════════
                    PRODUCT INFORMATION GATHERING
═══════════════════════════════════════════════════════════════════════════

I need some information about your product to create accurate documentation.
Please answer the following questions:

1. **Product Name & Core Concept**
   What is the name of your product and what does it do in one sentence?
   (Example: "TaskFlow is a project management tool that helps remote teams collaborate")

2. **Target Users**
   Who are the primary users of this product?
   (Example: "Software development teams, project managers, remote workers")

3. **Key Features** (list at least 3)
   What are the main features or capabilities?
   (Example: "Task boards, time tracking, team chat, file sharing")

4. **Main Problem Solved**
   What problem does this product solve for users?
   (Example: "Helps distributed teams stay organized and communicate effectively")

5. **Public Resources & Documentation** (optional but recommended)
   Do you have any of the following to share?
   
   📎 **Links to provide:**
   - Product website or landing page URL
   - GitHub repository (if public)
   - API documentation
   - Blog posts or articles about the product
   - Demo videos or tutorials
   
   📁 **Files to place in `geist/product/docs/`:**
   - PRDs (Product Requirements Documents)
   - Design documents or specs
   - Architecture diagrams
   - User research or personas
   - Competitor analysis
   - Any other relevant documentation
   
   📁 **Files to place in `geist/product/inheritance/`:**
   - Existing README or documentation
   - Marketing copy or pitch decks
   - Feature lists or roadmaps

6. **Web Research Suggestions** (optional)
   Would you like me to research any of the following?
   
   - Best practices for [your product category]
   - Competitor products and their features
   - Industry trends related to your product
   - Technical patterns for your tech stack
   
   If yes, please provide:
   - Keywords to search (e.g., "AI coding assistant", "developer tools")
   - Specific competitors to research
   - Technical topics to explore

═══════════════════════════════════════════════════════════════════════════
```

**IMPORTANT:** After presenting these questions:
- **STOP** and wait for the user to respond
- Do NOT assume answers or use defaults
- Do NOT proceed until the user provides information

---

## Step 4: Process User Responses

After receiving user responses:

### A. Record User Input

Store the user's responses for use in creating product documentation:
- Product name and concept
- Target users
- Key features
- Problem statement
- Any links or documents provided
- Web research requests

### B. Process Documentation Folders

Check both folders for user-provided files:

**geist/product/docs/ folder:**
```bash
if [ -d "geist/product/docs" ] && [ "$(ls -A geist/product/docs 2>/dev/null)" ]; then
    echo "📁 Found files in geist/product/docs/:"
    ls -la geist/product/docs/
fi
```

**geist/product/inheritance/ folder:**
```bash
if [ -d "geist/product/inheritance" ] && [ "$(ls -A geist/product/inheritance 2>/dev/null)" ]; then
    echo "📁 Found files in geist/product/inheritance/:"
    ls -la geist/product/inheritance/
fi
```

For each file found:
1. **Read and process each file:**
   - Markdown files (.md) - Extract text content
   - Text files (.txt) - Read plain text
   - PDF files (.pdf) - Extract text content
   - JSON/YAML files - Parse structured data
   - Image files - Note for reference (diagrams, mockups)
   - Other formats - Attempt to read as text

2. **Extract information:**
   - Product description and purpose
   - Features and capabilities
   - User information
   - Technical details
   - Roadmap items

### C. Process Links Provided

For any URLs the user provided:

1. **Validate the URL** - Check format and accessibility
2. **Fetch content** - Use web search or browser tools to access
3. **Extract information** - Parse product-relevant content
4. **Save reference** - Store link and summary in `geist/product/docs/links.md`
5. **Handle errors gracefully** - Continue if a link fails

### D. Execute Web Research (if requested)

If the user provided web research suggestions:

1. **Search for each keyword/topic:**
   - Use web search tool with provided keywords
   - Focus on recent, authoritative sources
   
2. **Research competitors (if specified):**
   - Search for competitor product information
   - Note key features and differentiators
   
3. **Gather technical best practices:**
   - Search for patterns related to tech stack
   - Find industry standards and conventions

4. **Save research results:**
   - Create `geist/product/docs/research-notes.md`
   - Include sources and key findings
   - Organize by topic

---

## Step 5: Merge All Information Sources

Combine information from all sources with this priority order:

1. **User-provided information** (highest priority)
2. **Files in geist/product/docs/**
3. **Files in geist/product/inheritance/**
4. **Scraped web content from user-provided links**
5. **Codebase analysis findings**
6. **Web research results** (lowest priority)

### Conflict Resolution

When the same information appears from multiple sources:
- Use the highest priority source
- Merge complementary information
- Flag conflicts for user review if significant

### Create Unified Knowledge Base

Organize gathered information by category:
- Mission and vision
- Target users and personas
- Features and capabilities
- Technical stack
- Roadmap items
- Competitive landscape (if researched)

---

## Output

After gathering and merging all information, confirm with the user:

```
═══════════════════════════════════════════════════════════════════════════
                    PRODUCT INFORMATION SUMMARY
═══════════════════════════════════════════════════════════════════════════

I have collected the following information:

📦 Product: [name]
👥 Users: [target users]
✨ Features: [key features]
🎯 Problem: [problem solved]

📚 Sources Used:
   - User responses: ✅
   - Files in docs/: [count] files
   - Files in inheritance/: [count] files
   - Links processed: [count]
   - Web research: [completed/skipped]

📁 Documentation Created:
   - geist/product/docs/links.md (if links provided)
   - geist/product/docs/research-notes.md (if research done)

Is this information correct? Should I proceed with creating the product documentation?
═══════════════════════════════════════════════════════════════════════════
```

**WAIT for user confirmation before proceeding to the next phase.**
