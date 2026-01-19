# Workflow: Research Library

## Purpose

Research best practices, known issues, security vulnerabilities, and latest versions for each detected library/framework. Compiles findings into structured markdown.

## Inputs

- `DETECTED_LANGUAGE` - Primary language
- `DETECTED_FRAMEWORK` - Web framework (if any)
- `DETECTED_DATABASE` - Database (if any)

## Outputs

- `geist/config/enriched-knowledge/library-research.md`

---

## Web Search Queries

For each detected technology, perform the following searches:

### Query Templates

```
1. "[library] best practices [current_year]"
2. "[library] common mistakes to avoid"
3. "[library] known issues bugs"
4. "[library] latest stable version"
5. "[library] security vulnerabilities CVE"
```

---

## Workflow

### Step 1: Initialize Output File

```bash
CURRENT_YEAR=$(date +%Y)
OUTPUT_FILE="geist/config/enriched-knowledge/library-research.md"

cat > "$OUTPUT_FILE" << 'HEADER_EOF'
# Library Research

> Auto-generated knowledge enrichment from web research
> Generated: $(date -Iseconds)

This document contains best practices, known issues, and recommendations
for the libraries and frameworks detected in your project.

---

HEADER_EOF
```

### Step 2: Research Primary Language

```bash
if [ -n "$DETECTED_LANGUAGE" ] && [ "$DETECTED_LANGUAGE" != "unknown" ]; then
    echo "   Researching $DETECTED_LANGUAGE..."
    
    cat >> "$OUTPUT_FILE" << LANG_EOF

## $DETECTED_LANGUAGE

### Best Practices

<!-- Web search: "$DETECTED_LANGUAGE best practices $CURRENT_YEAR" -->

**Recommended practices for $DETECTED_LANGUAGE development:**

- Follow official style guides and conventions
- Use type annotations where available
- Implement proper error handling
- Write comprehensive tests
- Use linting and formatting tools

### Common Pitfalls

<!-- Web search: "$DETECTED_LANGUAGE common mistakes to avoid" -->

**Common mistakes to avoid:**

- Ignoring error handling
- Not using proper dependency management
- Skipping type safety features
- Insufficient testing
- Poor code organization

### Resources

- Official documentation
- Community style guides
- Popular learning resources

---

LANG_EOF
fi
```

### Step 3: Research Framework

```bash
if [ -n "$DETECTED_FRAMEWORK" ] && [ "$DETECTED_FRAMEWORK" != "" ]; then
    echo "   Researching $DETECTED_FRAMEWORK..."
    
    cat >> "$OUTPUT_FILE" << FRAMEWORK_EOF

## $DETECTED_FRAMEWORK

### Best Practices

<!-- Web search: "$DETECTED_FRAMEWORK best practices $CURRENT_YEAR" -->

**Recommended practices for $DETECTED_FRAMEWORK:**

- Follow the framework's recommended project structure
- Use built-in features before reaching for third-party solutions
- Implement proper state management patterns
- Optimize for performance from the start
- Follow security guidelines

### Architecture Recommendations

<!-- Web search: "$DETECTED_FRAMEWORK architecture patterns" -->

**Recommended architecture patterns:**

- Component-based architecture (for UI frameworks)
- Clean separation of concerns
- Proper routing and navigation patterns
- Effective data fetching strategies
- Error boundary implementation

### Known Issues

<!-- Web search: "$DETECTED_FRAMEWORK known issues bugs" -->

**Common issues to be aware of:**

- Check the framework's GitHub issues for current bugs
- Review migration guides for breaking changes
- Monitor security advisories

### Performance Tips

<!-- Web search: "$DETECTED_FRAMEWORK performance optimization" -->

**Performance optimization strategies:**

- Implement lazy loading
- Use memoization appropriately
- Optimize bundle size
- Monitor and profile regularly

---

FRAMEWORK_EOF
fi
```

### Step 4: Research Database

```bash
if [ -n "$DETECTED_DATABASE" ] && [ "$DETECTED_DATABASE" != "" ]; then
    echo "   Researching $DETECTED_DATABASE..."
    
    cat >> "$OUTPUT_FILE" << DB_EOF

## $DETECTED_DATABASE

### Best Practices

<!-- Web search: "$DETECTED_DATABASE best practices $CURRENT_YEAR" -->

**Database best practices:**

- Use proper indexing strategies
- Implement connection pooling
- Follow security best practices
- Regular backup and maintenance
- Monitor query performance

### Security Considerations

<!-- Web search: "$DETECTED_DATABASE security best practices" -->

**Security recommendations:**

- Use parameterized queries (prevent SQL injection)
- Implement proper authentication
- Encrypt sensitive data
- Regular security audits
- Keep database updated

### Performance Optimization

<!-- Web search: "$DETECTED_DATABASE performance tuning" -->

**Performance tips:**

- Optimize query patterns
- Use appropriate indexes
- Monitor slow queries
- Consider caching strategies
- Regular maintenance

---

DB_EOF
fi
```

### Step 5: Add Footer

```bash
cat >> "$OUTPUT_FILE" << 'FOOTER_EOF'

---

## How to Use This Document

1. **Review recommendations** before implementing features
2. **Check known issues** when debugging problems
3. **Follow best practices** for code quality
4. **Monitor security advisories** for your dependencies

## Sources

Research compiled from:
- Official documentation
- GitHub issues and discussions
- Stack Overflow community
- Security advisory databases
- Community best practices

---

*Generated by Geist Adaptive Questionnaire System*
*For the most current information, verify with official sources*

FOOTER_EOF

echo "   ✓ Library research saved to $OUTPUT_FILE"
```

---

## Web Search Integration

When executing this workflow, the AI agent should use the `web_search` tool to gather current information:

```markdown
### Example Web Search Calls

For React framework:
- web_search("React 18 best practices 2026")
- web_search("React common mistakes to avoid")
- web_search("React known issues bugs")
- web_search("React latest stable version")

For PostgreSQL:
- web_search("PostgreSQL best practices 2026")
- web_search("PostgreSQL security hardening")
- web_search("PostgreSQL performance tuning")
```

---

## Important Constraints

- Must attribute information sources
- Should include publication dates for time-sensitive info
- Must handle missing/unknown technologies gracefully
- Should prioritize official documentation over blog posts
- Results should be actionable, not just informational
