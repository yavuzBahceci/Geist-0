# Phase 4: Enrich with Library Knowledge

Load relevant library basepoints and check for library misuse or anti-patterns.

## Step 1: Identify Libraries Used in Implementation

```bash
echo "🔍 Identifying libraries used in implementation..."

REVIEW_CACHE="$SPEC_PATH/implementation/cache/review"

# Get modified files
MODIFIED_FILES=$(cat "$REVIEW_CACHE/review-context.md" 2>/dev/null | sed -n '/## Modified Codebase Files/,/## /p' | grep -v "^##" | grep -v "^$")

# Extract imports/requires from modified files
LIBRARIES_USED=""

echo "$MODIFIED_FILES" | while read file; do
    if [ -f "$file" ]; then
        # Extract library imports based on file type
        case "$file" in
            *.ts|*.tsx|*.js|*.jsx)
                # JavaScript/TypeScript imports
                imports=$(grep -E "^import .* from ['\"]" "$file" 2>/dev/null | grep -v "^\." | sed "s/.*from ['\"]\\([^'\"]*\\)['\"].*/\\1/" | cut -d'/' -f1)
                ;;
            *.py)
                # Python imports
                imports=$(grep -E "^import |^from " "$file" 2>/dev/null | sed 's/import //;s/from //;s/ .*//;s/\..*//') 
                ;;
            *.go)
                # Go imports
                imports=$(grep -E "^\t\"" "$file" 2>/dev/null | tr -d '\t"' | cut -d'/' -f1)
                ;;
            *.java|*.kt)
                # Java/Kotlin imports
                imports=$(grep -E "^import " "$file" 2>/dev/null | sed 's/import //;s/\..*//') 
                ;;
            *)
                imports=""
                ;;
        esac
        
        if [ -n "$imports" ]; then
            LIBRARIES_USED="${LIBRARIES_USED}\n$imports"
        fi
    fi
done

# Deduplicate
UNIQUE_LIBRARIES=$(echo -e "$LIBRARIES_USED" | sort -u | grep -v "^$")
LIB_COUNT=$(echo "$UNIQUE_LIBRARIES" | grep -c "." || echo "0")

echo "   Found $LIB_COUNT unique libraries used"
```

## Step 2: Load Relevant Library Basepoints

```bash
echo "📖 Loading relevant library basepoints..."

LIBRARIES_DIR="geist/basepoints/libraries"
LIBRARY_FINDINGS=""

if [ -d "$LIBRARIES_DIR" ]; then
    echo "$UNIQUE_LIBRARIES" | while read lib; do
        if [ -n "$lib" ]; then
            # Normalize library name for file lookup
            LIB_SLUG=$(echo "$lib" | tr '[:upper:]' '[:lower:]' | tr '/@' '-')
            
            # Search for library basepoint
            LIB_BASEPOINT=$(find "$LIBRARIES_DIR" -name "*$LIB_SLUG*.md" -type f | head -1)
            
            if [ -f "$LIB_BASEPOINT" ]; then
                echo "   ✅ Found basepoint for: $lib"
                
                # Load library basepoint content
                LIB_CONTENT=$(cat "$LIB_BASEPOINT")
                
                # Extract key sections
                INTERNAL_ARCH=$(echo "$LIB_CONTENT" | sed -n '/## Internal Architecture/,/^## /p')
                BEST_PRACTICES=$(echo "$LIB_CONTENT" | sed -n '/## Best Practices/,/^## /p')
                PITFALLS=$(echo "$LIB_CONTENT" | sed -n '/## Pitfalls/,/^## /p')
                ANTI_PATTERNS=$(echo "$LIB_CONTENT" | sed -n '/## Anti-Patterns/,/^## /p')
                
            else
                echo "   ⚠️ No basepoint for: $lib"
            fi
        fi
    done
else
    echo "   ⚠️ Libraries directory not found"
fi
```

## Step 3: Check for Library Best Practices

```bash
echo "🔍 Checking for library best practices compliance..."

echo "$UNIQUE_LIBRARIES" | while read lib; do
    if [ -n "$lib" ]; then
        echo "   Checking best practices for: $lib"
        
        # Find library basepoint
        LIB_SLUG=$(echo "$lib" | tr '[:upper:]' '[:lower:]' | tr '/@' '-')
        LIB_BASEPOINT=$(find "$LIBRARIES_DIR" -name "*$LIB_SLUG*.md" -type f 2>/dev/null | head -1)
        
        if [ -f "$LIB_BASEPOINT" ]; then
            # Extract best practices
            BEST_PRACTICES=$(cat "$LIB_BASEPOINT" | sed -n '/## Best Practices/,/^## /p')
            
            if [ -n "$BEST_PRACTICES" ]; then
                # [AI agent checks if implementation follows best practices]
                echo "      Checking against documented best practices..."
            fi
        fi
    fi
done
```

## Step 4: Check for Library Anti-Patterns

```bash
echo "⚠️ Checking for library anti-patterns..."

echo "$UNIQUE_LIBRARIES" | while read lib; do
    if [ -n "$lib" ]; then
        echo "   Checking anti-patterns for: $lib"
        
        # Find library basepoint
        LIB_SLUG=$(echo "$lib" | tr '[:upper:]' '[:lower:]' | tr '/@' '-')
        LIB_BASEPOINT=$(find "$LIBRARIES_DIR" -name "*$LIB_SLUG*.md" -type f 2>/dev/null | head -1)
        
        if [ -f "$LIB_BASEPOINT" ]; then
            # Extract anti-patterns
            ANTI_PATTERNS=$(cat "$LIB_BASEPOINT" | sed -n '/## Pitfalls\|## Anti-Patterns/,/^## /p')
            
            if [ -n "$ANTI_PATTERNS" ]; then
                # [AI agent checks if implementation has anti-patterns]
                echo "      Checking against documented anti-patterns..."
            fi
        fi
    fi
done
```

## Step 5: Check for Library Limitations Usage

```bash
echo "🚫 Checking for usage of library limitations..."

echo "$UNIQUE_LIBRARIES" | while read lib; do
    if [ -n "$lib" ]; then
        echo "   Checking limitations for: $lib"
        
        # Find library basepoint
        LIB_SLUG=$(echo "$lib" | tr '[:upper:]' '[:lower:]' | tr '/@' '-')
        LIB_BASEPOINT=$(find "$LIBRARIES_DIR" -name "*$LIB_SLUG*.md" -type f 2>/dev/null | head -1)
        
        if [ -f "$LIB_BASEPOINT" ]; then
            # Extract limitations
            LIMITATIONS=$(cat "$LIB_BASEPOINT" | sed -n "/## What's Possible/,/^## /p")
            
            if [ -n "$LIMITATIONS" ]; then
                # [AI agent checks if implementation tries to do something not possible]
                echo "      Checking against documented limitations..."
            fi
        fi
    fi
done
```

## Step 6: Store Library Review Findings

```bash
echo "📝 Storing library review findings..."

cat > "$REVIEW_CACHE/library-findings.md" << LIB_FINDINGS_EOF
# Library Review Findings

## Review Date
$(date)

---

## Libraries Used in Implementation

$UNIQUE_LIBRARIES

---

## Library-Specific Findings

[For each library with a basepoint:]

### [Library Name]

**Basepoint**: [path or "Not found"]
**Version in Use**: [version if known]

#### Best Practices Compliance
- ✅/❌ [Practice 1]: [Status]
- ✅/❌ [Practice 2]: [Status]

#### Anti-Patterns Detected
- ⚠️ [Anti-pattern 1]: [Where found]
- ⚠️ [Anti-pattern 2]: [Where found]

#### Limitations Violated
- 🚫 [Limitation 1]: [Where attempted]

#### Issues Found
| Issue | File | Severity | Recommendation |
|-------|------|----------|----------------|
| [Issue] | [file] | [severity] | [recommendation] |

---

## Libraries Without Basepoints

[List of libraries used that don't have basepoints - potential knowledge gaps]

- [library 1]
- [library 2]

---

## Summary of Library-Related Issues

| Library | Issue | Severity | Recommendation |
|---------|-------|----------|----------------|
| [lib] | [issue] | [severity] | [recommendation] |

---

*Generated: $(date)*
LIB_FINDINGS_EOF

echo "✅ Library findings saved to: $REVIEW_CACHE/library-findings.md"
```

## Display Progress and Next Step

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 PHASE 4: LIBRARY KNOWLEDGE ENRICHMENT COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Libraries identified: [count]
✅ Library basepoints loaded: [count]
✅ Best practices checked
✅ Anti-patterns checked
✅ Limitations checked

Findings saved to: [REVIEW_CACHE]/library-findings.md

NEXT STEP 👉 Proceeding to Phase 5: Generate Report
```
