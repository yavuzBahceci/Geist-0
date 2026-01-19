# Workflow: Research Domain

## Purpose

Research domain-specific patterns, compliance requirements, and industry best practices based on the detected project type.

## Inputs

- `PROJECT_TYPE` - web_app, cli, api, library, monorepo
- `DETECTED_FRAMEWORK` - For context
- `SECURITY_LEVEL` - Informs compliance research depth

## Outputs

- `geist/config/enriched-knowledge/domain-knowledge.md`

---

## Web Search Queries

### Query Templates

```
1. "[project_type] software architecture patterns"
2. "[project_type] industry best practices"
3. "[project_type] compliance requirements"
4. "[project_type] common challenges"
5. "[project_type] scalability patterns"
```

---

## Workflow

### Step 1: Initialize Output File

```bash
CURRENT_YEAR=$(date +%Y)
OUTPUT_FILE="geist/config/enriched-knowledge/domain-knowledge.md"

cat > "$OUTPUT_FILE" << HEADER_EOF
# Domain Knowledge

> Industry-specific patterns and considerations
> Generated: $(date -Iseconds)

**Project Type:** ${PROJECT_TYPE:-unknown}

---

HEADER_EOF
```

### Step 2: Research Based on Project Type

```bash
case "${PROJECT_TYPE:-unknown}" in
    "web_app")
        cat >> "$OUTPUT_FILE" << 'WEBAPP_EOF'

## Web Application Patterns

<!-- Web search: "web application architecture patterns [year]" -->

### Common Architectures

1. **Single Page Application (SPA)**
   - Rich client-side interactivity
   - API-driven data fetching
   - Client-side routing
   
2. **Server-Side Rendered (SSR)**
   - Better SEO
   - Faster initial load
   - Server load considerations

3. **Hybrid (SSR + SPA)**
   - Best of both worlds
   - Complexity trade-off
   - Popular in modern frameworks

### User Experience Considerations

- **Performance**: First Contentful Paint < 1.8s
- **Accessibility**: WCAG 2.1 compliance
- **Responsive Design**: Mobile-first approach
- **Progressive Enhancement**: Works without JS

### Security Requirements

- **Authentication**: Secure login flows
- **Authorization**: Role-based access control
- **Data Protection**: HTTPS, CSP headers
- **Input Validation**: Client and server-side

---

WEBAPP_EOF
        ;;
        
    "api")
        cat >> "$OUTPUT_FILE" << 'API_EOF'

## API Service Patterns

<!-- Web search: "API design best practices [year]" -->

### API Design Principles

1. **RESTful Design**
   - Resource-oriented URLs
   - HTTP methods for operations
   - Stateless communication
   
2. **Versioning Strategy**
   - URL versioning: `/api/v1/`
   - Header versioning
   - Query parameter versioning

3. **Error Handling**
   - Consistent error format
   - Meaningful status codes
   - Detailed error messages (dev only)

### Performance Patterns

- **Pagination**: Cursor-based for large datasets
- **Caching**: ETags, Cache-Control headers
- **Rate Limiting**: Protect against abuse
- **Compression**: gzip/brotli responses

### Documentation

- OpenAPI/Swagger specification
- Interactive documentation
- Code examples
- Versioned docs

---

API_EOF
        ;;
        
    "cli")
        cat >> "$OUTPUT_FILE" << 'CLI_EOF'

## CLI Tool Patterns

<!-- Web search: "CLI application best practices" -->

### CLI Design Principles

1. **User Experience**
   - Clear help text (`--help`)
   - Intuitive command structure
   - Meaningful error messages
   - Progress indicators for long operations

2. **Command Structure**
   ```
   tool <command> [subcommand] [options] [arguments]
   ```

3. **Configuration**
   - Config file support
   - Environment variables
   - Command-line flags (highest priority)

### Best Practices

- Follow POSIX conventions
- Support piping and redirection
- Provide quiet (`-q`) and verbose (`-v`) modes
- Exit with appropriate codes
- Support both short (`-h`) and long (`--help`) flags

### Distribution

- Single binary if possible
- Package manager support (npm, cargo, brew)
- Auto-update mechanism
- Cross-platform builds

---

CLI_EOF
        ;;
        
    "library")
        cat >> "$OUTPUT_FILE" << 'LIB_EOF'

## Library Design Patterns

<!-- Web search: "library design best practices" -->

### API Design

1. **Minimal Surface Area**
   - Expose only what's needed
   - Internal vs. public APIs
   - Deprecation strategy

2. **Consistency**
   - Naming conventions
   - Error handling patterns
   - Return value patterns

3. **Extensibility**
   - Plugin/middleware support
   - Configuration options
   - Hooks for customization

### Documentation

- Comprehensive README
- API documentation
- Usage examples
- Migration guides
- Changelog

### Versioning

- Semantic versioning (SemVer)
- Clear breaking change policy
- Deprecation warnings
- LTS versions for stability

---

LIB_EOF
        ;;
        
    *)
        cat >> "$OUTPUT_FILE" << 'DEFAULT_EOF'

## General Software Patterns

### Universal Best Practices

1. **Code Quality**
   - Consistent formatting
   - Meaningful naming
   - Documentation
   - Testing

2. **Architecture**
   - Separation of concerns
   - Dependency management
   - Configuration management
   - Error handling

3. **Operations**
   - Logging and monitoring
   - Health checks
   - Graceful shutdown
   - Configuration via environment

---

DEFAULT_EOF
        ;;
esac
```

### Step 3: Research Compliance Requirements

```bash
if [ "$SECURITY_LEVEL" = "high" ]; then
    cat >> "$OUTPUT_FILE" << 'COMPLIANCE_EOF'

## Compliance Considerations

<!-- Web search: "software compliance requirements [year]" -->

### Common Compliance Frameworks

| Framework | Focus | Key Requirements |
|-----------|-------|------------------|
| **GDPR** | Data Privacy (EU) | Consent, data rights, breach notification |
| **SOC 2** | Security Controls | Access control, encryption, monitoring |
| **HIPAA** | Healthcare Data | PHI protection, access logs, encryption |
| **PCI-DSS** | Payment Data | Cardholder data protection, network security |

### General Compliance Checklist

- [ ] Data encryption at rest and in transit
- [ ] Access control and authentication
- [ ] Audit logging
- [ ] Data retention policies
- [ ] Incident response plan
- [ ] Regular security assessments
- [ ] Privacy policy and terms of service

### Security Controls

1. **Authentication**
   - Multi-factor authentication
   - Session management
   - Password policies

2. **Authorization**
   - Principle of least privilege
   - Role-based access control
   - Resource-level permissions

3. **Data Protection**
   - Encryption standards (AES-256)
   - Key management
   - Secure deletion

---

COMPLIANCE_EOF
fi
```

### Step 4: Add Footer

```bash
cat >> "$OUTPUT_FILE" << 'FOOTER_EOF'

## Implementation Priority

Based on your project type, prioritize:

1. **Core Functionality** - Get the basics right first
2. **Security** - Build security in from the start
3. **Performance** - Optimize critical paths
4. **Scalability** - Design for growth
5. **Maintainability** - Think long-term

## Sources

Domain knowledge compiled from:
- Industry standards and frameworks
- Community best practices
- Regulatory requirements
- Real-world case studies

---

*Generated by Geist Adaptive Questionnaire System*

FOOTER_EOF

echo "   ✓ Domain knowledge saved to $OUTPUT_FILE"
```

---

## Important Constraints

- Research should be specific to detected project type
- Compliance info should match security level
- Should provide actionable recommendations
- Must include implementation priorities
