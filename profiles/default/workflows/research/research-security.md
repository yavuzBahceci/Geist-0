# Workflow: Research Security

## Purpose

Research security vulnerabilities (CVEs), security advisories, and security best practices for the detected dependencies.

## Inputs

- `DETECTED_LANGUAGE` - Primary language
- `DETECTED_FRAMEWORK` - Web framework
- `DETECTED_DATABASE` - Database
- Dependencies from package.json, Cargo.toml, etc.

## Outputs

- `geist/config/enriched-knowledge/security-notes.md`

---

## Web Search Queries

### Query Templates

```
1. "[dependency] CVE vulnerabilities [year]"
2. "[dependency] security advisory"
3. "[framework] security best practices"
4. "[language] OWASP top 10 prevention"
5. "[dependency] security patch"
```

---

## Workflow

### Step 1: Initialize Output File

```bash
CURRENT_YEAR=$(date +%Y)
OUTPUT_FILE="geist/config/enriched-knowledge/security-notes.md"

cat > "$OUTPUT_FILE" << HEADER_EOF
# Security Notes

> Security vulnerabilities and recommendations for your dependencies
> Generated: $(date -Iseconds)

⚠️ **Important**: Always verify security information with official sources.
Check npm audit, cargo audit, or pip-audit for real-time vulnerability scanning.

---

HEADER_EOF
```

### Step 2: Research Language Security

```bash
if [ -n "$DETECTED_LANGUAGE" ] && [ "$DETECTED_LANGUAGE" != "unknown" ]; then
    cat >> "$OUTPUT_FILE" << LANG_SEC_EOF

## $DETECTED_LANGUAGE Security

<!-- Web search: "$DETECTED_LANGUAGE security best practices $CURRENT_YEAR" -->

### Common Vulnerabilities

| Vulnerability | Risk | Prevention |
|---------------|------|------------|
| Injection attacks | High | Input validation, parameterized queries |
| XSS (if web) | High | Output encoding, CSP headers |
| Insecure dependencies | Medium | Regular audits, version pinning |
| Secrets exposure | Critical | Environment variables, secret managers |

### Security Tools

LANG_SEC_EOF

    case "$DETECTED_LANGUAGE" in
        "javascript"|"typescript")
            cat >> "$OUTPUT_FILE" << 'JSTOOLS_EOF'
- **npm audit** - Check for vulnerable dependencies
- **Snyk** - Continuous vulnerability monitoring
- **ESLint security plugins** - Static code analysis
- **Helmet** - Security headers for Express

```bash
# Run security audit
npm audit
npm audit fix

# Use Snyk
npx snyk test
```

JSTOOLS_EOF
            ;;
        "rust")
            cat >> "$OUTPUT_FILE" << 'RSTOOLS_EOF'
- **cargo audit** - Check for vulnerable dependencies
- **cargo deny** - Lint dependencies
- **clippy** - Catch common mistakes

```bash
# Run security audit
cargo audit

# Install if needed
cargo install cargo-audit
```

RSTOOLS_EOF
            ;;
        "python")
            cat >> "$OUTPUT_FILE" << 'PYTOOLS_EOF'
- **pip-audit** - Check for vulnerable dependencies
- **safety** - Check dependencies against safety db
- **bandit** - Security linter

```bash
# Run security audit
pip-audit

# Or use safety
safety check
```

PYTOOLS_EOF
            ;;
        "go")
            cat >> "$OUTPUT_FILE" << 'GOTOOLS_EOF'
- **govulncheck** - Official Go vulnerability scanner
- **gosec** - Security linter

```bash
# Run vulnerability check
govulncheck ./...

# Install if needed
go install golang.org/x/vuln/cmd/govulncheck@latest
```

GOTOOLS_EOF
            ;;
    esac
    
    echo "---" >> "$OUTPUT_FILE"
fi
```

### Step 3: Research Framework Security

```bash
if [ -n "$DETECTED_FRAMEWORK" ]; then
    cat >> "$OUTPUT_FILE" << FRAMEWORK_SEC_EOF

## $DETECTED_FRAMEWORK Security

<!-- Web search: "$DETECTED_FRAMEWORK security vulnerabilities CVE" -->

### Security Checklist

- [ ] Keep framework updated to latest stable version
- [ ] Review security advisories regularly
- [ ] Follow framework's security guidelines
- [ ] Implement recommended security middleware
- [ ] Use built-in security features

### Known Security Considerations

FRAMEWORK_SEC_EOF

    case "$DETECTED_FRAMEWORK" in
        "react"|"vue"|"angular")
            cat >> "$OUTPUT_FILE" << 'FRONTEND_SEC_EOF'
**Frontend Framework Security:**

1. **XSS Prevention**
   - Use framework's built-in escaping
   - Avoid `dangerouslySetInnerHTML` (React) or `v-html` (Vue)
   - Sanitize user input before display

2. **CSRF Protection**
   - Use anti-CSRF tokens
   - SameSite cookie attribute
   - Verify origin headers

3. **Secure Dependencies**
   - Regular `npm audit`
   - Lock file integrity
   - Review new dependencies

FRONTEND_SEC_EOF
            ;;
        "express"|"fastify"|"koa")
            cat >> "$OUTPUT_FILE" << 'BACKEND_SEC_EOF'
**Backend Framework Security:**

1. **Request Validation**
   - Validate all input
   - Sanitize data
   - Use schema validation (Zod, Joi)

2. **Authentication**
   - Secure session management
   - Rate limiting on auth endpoints
   - Brute force protection

3. **Headers & HTTPS**
   - Use Helmet.js for security headers
   - Force HTTPS in production
   - Set secure cookie flags

BACKEND_SEC_EOF
            ;;
    esac
    
    echo "---" >> "$OUTPUT_FILE"
fi
```

### Step 4: Research Database Security

```bash
if [ -n "$DETECTED_DATABASE" ]; then
    cat >> "$OUTPUT_FILE" << DB_SEC_EOF

## $DETECTED_DATABASE Security

<!-- Web search: "$DETECTED_DATABASE security best practices" -->

### Security Checklist

- [ ] Use strong, unique passwords
- [ ] Enable authentication
- [ ] Encrypt connections (TLS/SSL)
- [ ] Regular security patches
- [ ] Principle of least privilege for users

### Common Vulnerabilities

| Risk | Prevention |
|------|------------|
| SQL Injection | Parameterized queries, ORM |
| Unauthorized access | Authentication, firewall |
| Data exposure | Encryption, access control |
| Backup theft | Encrypted backups |

### Secure Configuration

- Disable default/public access
- Use connection pooling with limits
- Enable query logging (audit)
- Regular backup testing

---

DB_SEC_EOF
fi
```

### Step 5: Add OWASP Reference

```bash
cat >> "$OUTPUT_FILE" << 'OWASP_EOF'

## OWASP Top 10 Reference

The most critical web application security risks:

| # | Risk | Key Prevention |
|---|------|----------------|
| 1 | Broken Access Control | Deny by default, validate permissions |
| 2 | Cryptographic Failures | Encrypt sensitive data, strong algorithms |
| 3 | Injection | Input validation, parameterized queries |
| 4 | Insecure Design | Threat modeling, secure patterns |
| 5 | Security Misconfiguration | Hardened configs, remove defaults |
| 6 | Vulnerable Components | Regular updates, dependency scanning |
| 7 | Auth Failures | MFA, rate limiting, secure sessions |
| 8 | Data Integrity Failures | Digital signatures, CI/CD security |
| 9 | Logging Failures | Comprehensive logging, monitoring |
| 10 | SSRF | Validate URLs, network segmentation |

---

OWASP_EOF
```

### Step 6: Add Severity Legend and Footer

```bash
cat >> "$OUTPUT_FILE" << 'FOOTER_EOF'

## Severity Levels

| Level | Description | Action |
|-------|-------------|--------|
| 🔴 **CRITICAL** | Actively exploited, patch immediately | Stop and fix now |
| 🟠 **HIGH** | Significant risk, patch soon | Fix within 24-48 hours |
| 🟡 **MEDIUM** | Moderate risk | Fix within 1 week |
| 🟢 **LOW** | Minor risk | Fix in next release |

## Recommended Actions

1. **Immediate**: Run security audit tool for your language
2. **Weekly**: Review security advisories for your dependencies
3. **Monthly**: Update dependencies to latest stable versions
4. **Quarterly**: Perform security review/penetration testing

## Resources

- [OWASP](https://owasp.org)
- [CVE Database](https://cve.mitre.org)
- [GitHub Security Advisories](https://github.com/advisories)
- [Snyk Vulnerability Database](https://snyk.io/vuln/)

---

*Generated by Geist Adaptive Questionnaire System*
*Always verify security information with official sources*

FOOTER_EOF

echo "   ✓ Security notes saved to $OUTPUT_FILE"
```

---

## Important Constraints

- Must clearly indicate severity of issues
- Should provide actionable remediation steps
- Must recommend official security scanning tools
- Should link to authoritative sources
- Must emphasize verification with real-time tools
