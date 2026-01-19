# Workflow: Version Analysis

## Purpose

Compare detected dependency versions against latest stable versions, flag outdated dependencies, and note breaking changes in newer versions.

## Inputs

- `package.json` (Node.js)
- `Cargo.toml` (Rust)
- `go.mod` (Go)
- `requirements.txt` / `pyproject.toml` (Python)

## Outputs

- `geist/config/enriched-knowledge/version-analysis.md`

---

## Web Search Queries

### Query Templates

```
1. "[package] latest version"
2. "[package] changelog breaking changes"
3. "[package] migration guide v[old] to v[new]"
```

---

## Workflow

### Step 1: Initialize Output File

```bash
CURRENT_YEAR=$(date +%Y)
OUTPUT_FILE="geist/config/enriched-knowledge/version-analysis.md"

cat > "$OUTPUT_FILE" << HEADER_EOF
# Version Analysis

> Dependency version status and update recommendations
> Generated: $(date -Iseconds)

This analysis compares your current dependency versions against the latest
stable releases and flags potential updates.

---

## Summary

HEADER_EOF
```

### Step 2: Analyze Node.js Dependencies

```bash
if [ -f "package.json" ]; then
    echo "" >> "$OUTPUT_FILE"
    echo "## Node.js Dependencies" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "<!-- Run \`npm outdated\` for real-time version check -->" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "| Package | Current | Status | Notes |" >> "$OUTPUT_FILE"
    echo "|---------|---------|--------|-------|" >> "$OUTPUT_FILE"
    
    # Extract key dependencies (simplified - in practice, use npm outdated)
    # This is a template showing the expected output format
    
    # Check for common frameworks and their typical update status
    if grep -q '"react"' package.json 2>/dev/null; then
        REACT_VER=$(grep '"react"' package.json | grep -oE '[0-9]+\.[0-9]+' | head -1)
        if [ "${REACT_VER%%.*}" -lt "18" ] 2>/dev/null; then
            echo "| react | ${REACT_VER:-unknown} | ⚠️ OUTDATED | Consider upgrading to React 18 |" >> "$OUTPUT_FILE"
        else
            echo "| react | ${REACT_VER:-unknown} | ✅ Current | |" >> "$OUTPUT_FILE"
        fi
    fi
    
    if grep -q '"next"' package.json 2>/dev/null; then
        NEXT_VER=$(grep '"next"' package.json | grep -oE '[0-9]+\.[0-9]+' | head -1)
        echo "| next | ${NEXT_VER:-unknown} | ℹ️ Check | Major versions may have breaking changes |" >> "$OUTPUT_FILE"
    fi
    
    if grep -q '"typescript"' package.json 2>/dev/null; then
        TS_VER=$(grep '"typescript"' package.json | grep -oE '[0-9]+\.[0-9]+' | head -1)
        echo "| typescript | ${TS_VER:-unknown} | ℹ️ Check | TypeScript 5.x has new features |" >> "$OUTPUT_FILE"
    fi
    
    echo "" >> "$OUTPUT_FILE"
    echo "### Recommended Actions" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "\`\`\`bash" >> "$OUTPUT_FILE"
    echo "# Check all outdated packages" >> "$OUTPUT_FILE"
    echo "npm outdated" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "# Update all packages (minor/patch)" >> "$OUTPUT_FILE"
    echo "npm update" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "# Interactive update (recommended)" >> "$OUTPUT_FILE"
    echo "npx npm-check-updates -i" >> "$OUTPUT_FILE"
    echo "\`\`\`" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
fi
```

### Step 3: Analyze Rust Dependencies

```bash
if [ -f "Cargo.toml" ]; then
    echo "" >> "$OUTPUT_FILE"
    echo "## Rust Dependencies" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "<!-- Run \`cargo outdated\` for real-time version check -->" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "| Crate | Current | Status | Notes |" >> "$OUTPUT_FILE"
    echo "|-------|---------|--------|-------|" >> "$OUTPUT_FILE"
    
    # Check for common crates
    if grep -q 'tokio' Cargo.toml 2>/dev/null; then
        TOKIO_VER=$(grep 'tokio' Cargo.toml | grep -oE '[0-9]+\.[0-9]+' | head -1)
        echo "| tokio | ${TOKIO_VER:-unknown} | ℹ️ Check | Async runtime |" >> "$OUTPUT_FILE"
    fi
    
    if grep -q 'serde' Cargo.toml 2>/dev/null; then
        SERDE_VER=$(grep 'serde' Cargo.toml | grep -oE '[0-9]+\.[0-9]+' | head -1)
        echo "| serde | ${SERDE_VER:-unknown} | ℹ️ Check | Serialization |" >> "$OUTPUT_FILE"
    fi
    
    echo "" >> "$OUTPUT_FILE"
    echo "### Recommended Actions" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "\`\`\`bash" >> "$OUTPUT_FILE"
    echo "# Install cargo-outdated" >> "$OUTPUT_FILE"
    echo "cargo install cargo-outdated" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "# Check outdated dependencies" >> "$OUTPUT_FILE"
    echo "cargo outdated" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "# Update dependencies" >> "$OUTPUT_FILE"
    echo "cargo update" >> "$OUTPUT_FILE"
    echo "\`\`\`" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
fi
```

### Step 4: Analyze Python Dependencies

```bash
if [ -f "requirements.txt" ] || [ -f "pyproject.toml" ]; then
    echo "" >> "$OUTPUT_FILE"
    echo "## Python Dependencies" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "<!-- Run \`pip list --outdated\` for real-time version check -->" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "| Package | Current | Status | Notes |" >> "$OUTPUT_FILE"
    echo "|---------|---------|--------|-------|" >> "$OUTPUT_FILE"
    
    # Check for common packages
    PYDEPS=""
    [ -f "requirements.txt" ] && PYDEPS=$(cat requirements.txt)
    
    if echo "$PYDEPS" | grep -qi 'django'; then
        DJANGO_VER=$(echo "$PYDEPS" | grep -i 'django' | grep -oE '[0-9]+\.[0-9]+' | head -1)
        echo "| django | ${DJANGO_VER:-unknown} | ℹ️ Check | Web framework |" >> "$OUTPUT_FILE"
    fi
    
    if echo "$PYDEPS" | grep -qi 'fastapi'; then
        FASTAPI_VER=$(echo "$PYDEPS" | grep -i 'fastapi' | grep -oE '[0-9]+\.[0-9]+' | head -1)
        echo "| fastapi | ${FASTAPI_VER:-unknown} | ℹ️ Check | API framework |" >> "$OUTPUT_FILE"
    fi
    
    echo "" >> "$OUTPUT_FILE"
    echo "### Recommended Actions" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "\`\`\`bash" >> "$OUTPUT_FILE"
    echo "# Check outdated packages" >> "$OUTPUT_FILE"
    echo "pip list --outdated" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "# Update all packages" >> "$OUTPUT_FILE"
    echo "pip install --upgrade -r requirements.txt" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "# Use pip-tools for better dependency management" >> "$OUTPUT_FILE"
    echo "pip install pip-tools" >> "$OUTPUT_FILE"
    echo "pip-compile --upgrade" >> "$OUTPUT_FILE"
    echo "\`\`\`" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
fi
```

### Step 5: Analyze Go Dependencies

```bash
if [ -f "go.mod" ]; then
    echo "" >> "$OUTPUT_FILE"
    echo "## Go Dependencies" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "<!-- Run \`go list -m -u all\` for real-time version check -->" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "| Module | Current | Status | Notes |" >> "$OUTPUT_FILE"
    echo "|--------|---------|--------|-------|" >> "$OUTPUT_FILE"
    
    # Check Go version
    GO_VER=$(grep "^go " go.mod | awk '{print $2}')
    echo "| go (runtime) | ${GO_VER:-unknown} | ℹ️ Check | Go version |" >> "$OUTPUT_FILE"
    
    echo "" >> "$OUTPUT_FILE"
    echo "### Recommended Actions" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "\`\`\`bash" >> "$OUTPUT_FILE"
    echo "# Check for available updates" >> "$OUTPUT_FILE"
    echo "go list -m -u all" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
    echo "# Update all dependencies" >> "$OUTPUT_FILE"
    echo "go get -u ./..." >> "$OUTPUT_FILE"
    echo "go mod tidy" >> "$OUTPUT_FILE"
    echo "\`\`\`" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"
fi
```

### Step 6: Add Update Strategy

```bash
cat >> "$OUTPUT_FILE" << 'STRATEGY_EOF'

---

## Update Strategy

### Version Status Legend

| Status | Meaning | Action |
|--------|---------|--------|
| ✅ Current | Latest stable version | No action needed |
| ℹ️ Check | Unknown/needs verification | Run version check tool |
| ⚠️ OUTDATED | Behind latest stable | Plan update |
| 🔴 CRITICAL | Security patch available | Update immediately |

### Safe Update Process

1. **Check current versions**
   ```bash
   # Use appropriate tool for your language
   npm outdated / cargo outdated / pip list --outdated
   ```

2. **Review changelogs**
   - Check for breaking changes
   - Review migration guides
   - Note deprecated features

3. **Update in stages**
   - Patch versions first (x.x.PATCH)
   - Then minor versions (x.MINOR.x)
   - Major versions last (MAJOR.x.x)

4. **Test thoroughly**
   - Run test suite after each update
   - Check critical paths manually
   - Monitor for regressions

### When to Update

| Situation | Recommended Action |
|-----------|-------------------|
| Security vulnerability | Update immediately |
| Major version behind | Plan migration sprint |
| Minor version behind | Update during maintenance |
| Patch version behind | Update with next release |

---

STRATEGY_EOF
```

### Step 7: Add Footer

```bash
cat >> "$OUTPUT_FILE" << 'FOOTER_EOF'

## Automated Version Checking

For real-time, accurate version information, use these tools:

| Language | Tool | Command |
|----------|------|---------|
| Node.js | npm | `npm outdated` |
| Node.js | npx | `npx npm-check-updates` |
| Rust | cargo | `cargo outdated` |
| Python | pip | `pip list --outdated` |
| Go | go | `go list -m -u all` |

### CI Integration

Consider adding version checking to your CI pipeline:

```yaml
# Example GitHub Action
- name: Check for outdated dependencies
  run: npm outdated || true  # Don't fail build
```

---

*Generated by Geist Adaptive Questionnaire System*
*Run the appropriate version check tool for accurate, real-time information*

FOOTER_EOF

echo "   ✓ Version analysis saved to $OUTPUT_FILE"
```

---

## Important Constraints

- Must recommend using real-time tools for accuracy
- Should not make definitive version claims without verification
- Must provide safe update strategy
- Should distinguish between critical and non-critical updates
- Must include commands for each language ecosystem
