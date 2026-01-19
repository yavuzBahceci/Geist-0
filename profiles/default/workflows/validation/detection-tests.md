# Workflow: Detection Tests

## Purpose

Integration test scenarios for the project detection workflows. These tests verify that detection works correctly for different project types.

---

## Test Fixtures

### Fixture 1: Node.js/TypeScript Project

**Setup:**
```bash
mkdir -p test-fixtures/nodejs-project
cd test-fixtures/nodejs-project

# Create package.json
cat > package.json << 'EOF'
{
  "name": "test-nodejs-project",
  "version": "1.0.0",
  "scripts": {
    "build": "tsc",
    "test": "jest",
    "lint": "eslint ."
  },
  "dependencies": {
    "react": "^18.2.0",
    "express": "^4.18.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "jest": "^29.0.0",
    "eslint": "^8.0.0"
  }
}
EOF

# Create tsconfig.json
cat > tsconfig.json << 'EOF'
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs"
  }
}
EOF

# Create directory structure
mkdir -p src/components
touch src/index.ts
touch src/components/App.tsx
```

**Expected Detection:**
```yaml
gathered:
  project_type: web_app
  tech_stack:
    language: typescript
    framework: react
    backend: nodejs-express
  commands:
    build: "npm run build"
    test: "npm test"
    lint: "npm run lint"
```

---

### Fixture 2: Rust Project

**Setup:**
```bash
mkdir -p test-fixtures/rust-project
cd test-fixtures/rust-project

# Create Cargo.toml
cat > Cargo.toml << 'EOF'
[package]
name = "test-rust-project"
version = "0.1.0"
edition = "2021"

[dependencies]
actix-web = "4"
tokio = { version = "1", features = ["full"] }
sqlx = { version = "0.7", features = ["postgres"] }
EOF

mkdir -p src
touch src/main.rs
touch src/lib.rs
```

**Expected Detection:**
```yaml
gathered:
  project_type: api
  tech_stack:
    language: rust
    framework: actix-web
    backend: rust
    database: postgresql
  commands:
    build: "cargo build"
    test: "cargo test"
    lint: "cargo clippy"
```

---

### Fixture 3: Python Project

**Setup:**
```bash
mkdir -p test-fixtures/python-project
cd test-fixtures/python-project

# Create requirements.txt
cat > requirements.txt << 'EOF'
fastapi==0.100.0
uvicorn==0.22.0
sqlalchemy==2.0.0
psycopg2-binary==2.9.0
pytest==7.4.0
flake8==6.0.0
EOF

# Create pyproject.toml
cat > pyproject.toml << 'EOF'
[tool.pytest.ini_options]
testpaths = ["tests"]

[tool.mypy]
python_version = "3.11"
EOF

mkdir -p src tests
touch src/main.py
touch tests/test_main.py
```

**Expected Detection:**
```yaml
gathered:
  project_type: api
  tech_stack:
    language: python
    framework: fastapi
    backend: python
    database: postgresql
  commands:
    test: "pytest"
    lint: "flake8"
```

---

### Fixture 4: Go Project

**Setup:**
```bash
mkdir -p test-fixtures/go-project
cd test-fixtures/go-project

# Create go.mod
cat > go.mod << 'EOF'
module github.com/test/go-project

go 1.21

require (
    github.com/gin-gonic/gin v1.9.0
    github.com/jackc/pgx/v5 v5.4.0
)
EOF

mkdir -p cmd pkg
touch cmd/main.go
touch pkg/handlers.go
```

**Expected Detection:**
```yaml
gathered:
  project_type: api
  tech_stack:
    language: go
    framework: gin
    backend: go
    database: postgresql
  commands:
    build: "go build ./..."
    test: "go test ./..."
    lint: "golangci-lint run ./..."
```

---

### Fixture 5: Unknown/Empty Project

**Setup:**
```bash
mkdir -p test-fixtures/unknown-project
cd test-fixtures/unknown-project

# Create minimal files
touch README.md
mkdir -p docs
```

**Expected Detection:**
```yaml
gathered:
  project_type: unknown
  tech_stack:
    language: unknown
  commands: {}

_meta:
  detection_confidence: 0.50
  needs_user_input:
    language: true
    project_type: true
```

---

## Test Scenarios

### Scenario 1: Full Detection Flow

```bash
# Test complete detection workflow
run_test() {
    cd test-fixtures/nodejs-project
    
    # Run detection
    source profiles/default/workflows/detection/detect-project-profile.md
    
    # Verify results
    assert_equals "$DETECTED_LANGUAGE" "typescript"
    assert_equals "$DETECTED_FRAMEWORK" "react"
    assert_equals "$DETECTED_BUILD_CMD" "npm run build"
    assert_gte "$DETECTION_CONFIDENCE" "0.90"
}
```

### Scenario 2: Confidence Threshold

```bash
# Test that low confidence triggers questions
test_low_confidence() {
    cd test-fixtures/unknown-project
    
    source profiles/default/workflows/detection/detect-project-profile.md
    
    assert_equals "$NEEDS_USER_INPUT_TYPE" "true"
    assert_equals "$NEEDS_USER_INPUT_LANGUAGE" "true"
    assert_lt "$DETECTION_CONFIDENCE" "0.70"
}
```

### Scenario 3: Confirmation Override

```bash
# Test user override capability
test_override() {
    cd test-fixtures/nodejs-project
    
    # Set override
    DETECTED_FRAMEWORK="vue"
    USER_CHOICE="type"
    USER_PROJECT_TYPE="library"
    
    source profiles/default/workflows/detection/present-and-confirm.md
    
    # Verify override applied
    assert_equals "$DETECTED_PROJECT_TYPE" "library"
}
```

### Scenario 4: Progressive Knowledge

```bash
# Test that profile persists across commands
test_progressive() {
    cd test-fixtures/nodejs-project
    
    # Run adapt-to-product detection
    source profiles/default/workflows/detection/detect-project-profile.md
    
    # Verify profile created
    assert_file_exists "geist/config/project-profile.yml"
    
    # Simulate create-basepoints loading profile
    PROFILE_EXISTS=$([ -f "geist/config/project-profile.yml" ] && echo "true")
    assert_equals "$PROFILE_EXISTS" "true"
}
```

---

## Running Tests

```bash
# Run all detection tests
./scripts/run-detection-tests.sh

# Run specific fixture test
./scripts/run-detection-tests.sh --fixture=nodejs-project

# Run with verbose output
./scripts/run-detection-tests.sh --verbose

# Generate test report
./scripts/run-detection-tests.sh --report > detection-test-report.md
```

---

## Expected Test Output

```
Detection Tests
===============

Fixture: nodejs-project
  ✓ Language detection: typescript
  ✓ Framework detection: react
  ✓ Commands detection: npm scripts
  ✓ Confidence: 0.95 (high)

Fixture: rust-project
  ✓ Language detection: rust
  ✓ Framework detection: actix-web
  ✓ Database detection: postgresql
  ✓ Confidence: 0.95 (high)

Fixture: python-project
  ✓ Language detection: python
  ✓ Framework detection: fastapi
  ✓ Confidence: 0.90 (high)

Fixture: go-project
  ✓ Language detection: go
  ✓ Framework detection: gin
  ✓ Confidence: 0.95 (high)

Fixture: unknown-project
  ✓ Fallback: unknown
  ✓ User input flags: true
  ✓ Confidence: 0.50 (low)

All tests passed: 20/20
```

---

## Important Constraints

- Tests should be idempotent (can run multiple times)
- Tests should clean up after themselves
- Tests should work offline (no web searches during tests)
- Tests should verify both positive and negative cases
