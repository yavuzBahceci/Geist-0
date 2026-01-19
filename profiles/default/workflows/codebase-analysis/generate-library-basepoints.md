# Workflow: Generate Library Basepoints

## Purpose

Create comprehensive library basepoints by combining:
1. **Project basepoints knowledge** - How we use the libraries in our codebase
2. **Product knowledge** - Tech stack, detected libraries
3. **Official documentation research** - Best practices, architecture, troubleshooting
4. **Our implementation patterns** - Actual usage patterns from codebase analysis

This workflow supports **all major technology ecosystems**:
- Web (JavaScript/TypeScript, Python, Ruby, PHP, Go, Rust)
- Mobile (iOS/Swift, Android/Kotlin, Flutter/Dart, React Native)
- Native (Java, Kotlin, C/C++, C#/.NET, Rust)
- Cross-platform (Electron, Tauri, Qt)
- Data/ML (Python, R, Julia)
- Systems (Rust, Go, C/C++)

This workflow is called by `create-basepoints` Phase 8 after module basepoints are generated.

## Prerequisites

- Module basepoints must be generated (Phase 5-7 complete)
- `geist/basepoints/headquarter.md` must exist
- `geist/product/tech-stack.md` must exist

## Inputs

- `TECH_STACK_FILE` - Path to tech-stack.md
- `HEADQUARTER_FILE` - Path to headquarter.md
- `BASEPOINTS_DIR` - Path to basepoints directory

## Outputs

- Library basepoints in `geist/basepoints/libraries/[category]/[library].md`
- Library index at `geist/basepoints/libraries/README.md`

---

## Workflow

### Step 1: Load Product Knowledge

Load the tech stack to identify libraries:

```bash
echo "📖 Loading product knowledge..."

TECH_STACK_FILE="geist/product/tech-stack.md"
HEADQUARTER_FILE="geist/basepoints/headquarter.md"
BASEPOINTS_DIR="geist/basepoints"
LIBRARIES_DIR="$BASEPOINTS_DIR/libraries"

# Load tech stack
if [ -f "$TECH_STACK_FILE" ]; then
    TECH_STACK_CONTENT=$(cat "$TECH_STACK_FILE")
    echo "✅ Loaded tech stack"
else
    echo "❌ Tech stack not found at $TECH_STACK_FILE"
    exit 1
fi

# Load headquarter for project context
if [ -f "$HEADQUARTER_FILE" ]; then
    HEADQUARTER_CONTENT=$(cat "$HEADQUARTER_FILE")
    echo "✅ Loaded headquarter"
else
    echo "⚠️ Headquarter not found, continuing without project overview"
    HEADQUARTER_CONTENT=""
fi
```

### Step 2: Detect Project Type and Extract Dependencies

Automatically detect project type and extract dependencies from all supported ecosystems:

```bash
echo "🔍 Detecting project type and extracting dependencies..."

LIBRARIES_LIST=""
PROJECT_TYPE="unknown"

# ═══════════════════════════════════════════════════════════════════════════════
# JAVASCRIPT / TYPESCRIPT ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "package.json" ]; then
    echo "   📦 Found package.json (Node.js/JavaScript/TypeScript)"
    PROJECT_TYPE="nodejs"
    
    # Extract dependencies
    DEPS=$(cat package.json | grep -A 1000 '"dependencies"' | grep -B 1000 '^\s*}' | head -n -1 | tail -n +2)
    DEV_DEPS=$(cat package.json | grep -A 1000 '"devDependencies"' | grep -B 1000 '^\s*}' | head -n -1 | tail -n +2)
    
    # Parse each dependency
    echo "$DEPS" | grep '"' | while read line; do
        LIB_NAME=$(echo "$line" | cut -d'"' -f2)
        LIB_VERSION=$(echo "$line" | cut -d'"' -f4)
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|$LIB_VERSION|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Node.js dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# PYTHON ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "requirements.txt" ]; then
    echo "   🐍 Found requirements.txt (Python)"
    PROJECT_TYPE="python"
    
    cat requirements.txt | grep -v "^#" | grep -v "^$" | while read line; do
        LIB_NAME=$(echo "$line" | sed 's/[=<>!].*//' | tr -d ' ')
        LIB_VERSION=$(echo "$line" | grep -oE '[0-9]+\.[0-9]+(\.[0-9]+)?' | head -1)
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Python requirements"
fi

if [ -f "pyproject.toml" ]; then
    echo "   🐍 Found pyproject.toml (Python - Modern)"
    PROJECT_TYPE="python"
    
    # Extract from [project.dependencies] or [tool.poetry.dependencies]
    grep -A 100 '\[project.dependencies\]\|\[tool.poetry.dependencies\]' pyproject.toml 2>/dev/null | \
        grep -E '^[a-zA-Z]' | while read line; do
        LIB_NAME=$(echo "$line" | cut -d'=' -f1 | tr -d ' "')
        LIB_VERSION=$(echo "$line" | grep -oE '[0-9]+\.[0-9]+(\.[0-9]+)?' | head -1)
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted pyproject.toml dependencies"
fi

if [ -f "Pipfile" ]; then
    echo "   🐍 Found Pipfile (Python - Pipenv)"
    PROJECT_TYPE="python"
    
    grep -A 100 '\[packages\]' Pipfile 2>/dev/null | grep -E '^[a-zA-Z]' | while read line; do
        LIB_NAME=$(echo "$line" | cut -d'=' -f1 | tr -d ' "')
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|unknown|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Pipfile dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# RUST ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "Cargo.toml" ]; then
    echo "   🦀 Found Cargo.toml (Rust)"
    PROJECT_TYPE="rust"
    
    grep -A 100 '\[dependencies\]' Cargo.toml 2>/dev/null | grep -E '^[a-zA-Z_-]+ =' | while read line; do
        LIB_NAME=$(echo "$line" | cut -d'=' -f1 | tr -d ' ')
        LIB_VERSION=$(echo "$line" | grep -oE '"[0-9]+\.[0-9]+(\.[0-9]+)?"' | tr -d '"' | head -1)
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Cargo dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# GO ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "go.mod" ]; then
    echo "   🐹 Found go.mod (Go)"
    PROJECT_TYPE="go"
    
    grep -E '^\s+[a-zA-Z]' go.mod | grep -v '^module\|^go ' | while read line; do
        LIB_NAME=$(echo "$line" | awk '{print $1}')
        LIB_VERSION=$(echo "$line" | awk '{print $2}')
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Go modules"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# RUBY ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "Gemfile" ]; then
    echo "   💎 Found Gemfile (Ruby)"
    PROJECT_TYPE="ruby"
    
    grep "^gem " Gemfile | while read line; do
        LIB_NAME=$(echo "$line" | cut -d"'" -f2 | cut -d'"' -f2)
        LIB_VERSION=$(echo "$line" | grep -oE "'[0-9]+\.[0-9]+(\.[0-9]+)?'" | tr -d "'" | head -1)
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Ruby gems"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# PHP ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "composer.json" ]; then
    echo "   🐘 Found composer.json (PHP)"
    PROJECT_TYPE="php"
    
    cat composer.json | grep -A 1000 '"require"' | grep -B 1000 '^\s*}' | head -n -1 | tail -n +2 | \
        grep '"' | while read line; do
        LIB_NAME=$(echo "$line" | cut -d'"' -f2)
        LIB_VERSION=$(echo "$line" | cut -d'"' -f4)
        if [ -n "$LIB_NAME" ] && [ "$LIB_NAME" != "php" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Composer packages"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# JAVA ECOSYSTEM (Maven)
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "pom.xml" ]; then
    echo "   ☕ Found pom.xml (Java - Maven)"
    PROJECT_TYPE="java-maven"
    
    # Extract dependencies from pom.xml
    grep -A 3 '<dependency>' pom.xml | grep -E '<groupId>|<artifactId>|<version>' | \
        paste - - - | while read line; do
        GROUP_ID=$(echo "$line" | grep -oP '(?<=<groupId>)[^<]+')
        ARTIFACT_ID=$(echo "$line" | grep -oP '(?<=<artifactId>)[^<]+')
        VERSION=$(echo "$line" | grep -oP '(?<=<version>)[^<]+')
        if [ -n "$ARTIFACT_ID" ]; then
            echo "${GROUP_ID}:${ARTIFACT_ID}|${VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Maven dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# JAVA/KOTLIN ECOSYSTEM (Gradle)
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "build.gradle" ] || [ -f "build.gradle.kts" ]; then
    echo "   ☕ Found build.gradle (Java/Kotlin - Gradle)"
    PROJECT_TYPE="java-gradle"
    
    GRADLE_FILE="build.gradle"
    [ -f "build.gradle.kts" ] && GRADLE_FILE="build.gradle.kts"
    
    # Extract implementation/api dependencies
    grep -E "implementation|api|compile" "$GRADLE_FILE" | grep -oE "'[^']+:[^']+:[^']+'" | tr -d "'" | while read dep; do
        LIB_NAME=$(echo "$dep" | cut -d':' -f1,2)
        LIB_VERSION=$(echo "$dep" | cut -d':' -f3)
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    # Also check for Kotlin DSL format
    grep -E 'implementation\(|api\(' "$GRADLE_FILE" | grep -oE '"[^"]+:[^"]+:[^"]+"' | tr -d '"' | while read dep; do
        LIB_NAME=$(echo "$dep" | cut -d':' -f1,2)
        LIB_VERSION=$(echo "$dep" | cut -d':' -f3)
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Gradle dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# ANDROID ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "app/build.gradle" ] || [ -f "app/build.gradle.kts" ]; then
    echo "   🤖 Found Android project"
    PROJECT_TYPE="android"
    
    GRADLE_FILE="app/build.gradle"
    [ -f "app/build.gradle.kts" ] && GRADLE_FILE="app/build.gradle.kts"
    
    # Extract Android dependencies
    grep -E "implementation|api|kapt|ksp|annotationProcessor" "$GRADLE_FILE" | \
        grep -oE "'[^']+:[^']+:[^']+'" | tr -d "'" | while read dep; do
        LIB_NAME=$(echo "$dep" | cut -d':' -f1,2)
        LIB_VERSION=$(echo "$dep" | cut -d':' -f3)
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    # Kotlin DSL format
    grep -E 'implementation\(|api\(' "$GRADLE_FILE" | grep -oE '"[^"]+:[^"]+:[^"]+"' | tr -d '"' | while read dep; do
        LIB_NAME=$(echo "$dep" | cut -d':' -f1,2)
        LIB_VERSION=$(echo "$dep" | cut -d':' -f3)
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Android dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# IOS / SWIFT ECOSYSTEM (CocoaPods)
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "Podfile" ]; then
    echo "   🍎 Found Podfile (iOS - CocoaPods)"
    PROJECT_TYPE="ios-cocoapods"
    
    grep "^[[:space:]]*pod " Podfile | while read line; do
        LIB_NAME=$(echo "$line" | sed "s/.*pod ['\"]\\([^'\"]*\\).*/\\1/")
        LIB_VERSION=$(echo "$line" | grep -oE "'~> [0-9]+\.[0-9]+(\.[0-9]+)?'" | sed "s/'~> //" | tr -d "'")
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted CocoaPods dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# IOS / SWIFT ECOSYSTEM (Swift Package Manager)
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "Package.swift" ]; then
    echo "   🍎 Found Package.swift (Swift Package Manager)"
    PROJECT_TYPE="ios-spm"
    
    # Extract package dependencies
    grep -E '\.package\(' Package.swift | while read line; do
        # Extract URL and version
        URL=$(echo "$line" | grep -oE 'url: "[^"]+"' | cut -d'"' -f2)
        LIB_NAME=$(echo "$URL" | sed 's|.*/||' | sed 's|\.git$||')
        VERSION=$(echo "$line" | grep -oE 'from: "[^"]+"' | cut -d'"' -f2)
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Swift Package dependencies"
fi

# Check for .xcodeproj (Xcode project without package manager)
if ls *.xcodeproj 1> /dev/null 2>&1; then
    echo "   🍎 Found Xcode project"
    [ "$PROJECT_TYPE" = "unknown" ] && PROJECT_TYPE="ios-xcode"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# FLUTTER / DART ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "pubspec.yaml" ]; then
    echo "   🎯 Found pubspec.yaml (Flutter/Dart)"
    PROJECT_TYPE="flutter"
    
    # Extract dependencies
    grep -A 100 '^dependencies:' pubspec.yaml | grep -B 100 -E '^[a-z_]+:' | head -n -1 | \
        grep -E '^  [a-z_]+:' | while read line; do
        LIB_NAME=$(echo "$line" | cut -d':' -f1 | tr -d ' ')
        LIB_VERSION=$(echo "$line" | cut -d':' -f2 | tr -d ' ^"')
        if [ -n "$LIB_NAME" ] && [ "$LIB_NAME" != "flutter" ] && [ "$LIB_NAME" != "sdk" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Flutter/Dart dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# REACT NATIVE ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "package.json" ] && grep -q "react-native" package.json 2>/dev/null; then
    echo "   📱 Detected React Native project"
    PROJECT_TYPE="react-native"
    # Dependencies already extracted from package.json above
fi

# ═══════════════════════════════════════════════════════════════════════════════
# .NET / C# ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if ls *.csproj 1> /dev/null 2>&1; then
    echo "   🔷 Found .csproj (C#/.NET)"
    PROJECT_TYPE="dotnet"
    
    for csproj in *.csproj; do
        grep '<PackageReference' "$csproj" | while read line; do
            LIB_NAME=$(echo "$line" | grep -oP 'Include="[^"]+"' | cut -d'"' -f2)
            LIB_VERSION=$(echo "$line" | grep -oP 'Version="[^"]+"' | cut -d'"' -f2)
            if [ -n "$LIB_NAME" ]; then
                echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
            fi
        done
    done
    
    echo "   ✅ Extracted NuGet packages"
fi

if [ -f "packages.config" ]; then
    echo "   🔷 Found packages.config (.NET Framework)"
    
    grep '<package ' packages.config | while read line; do
        LIB_NAME=$(echo "$line" | grep -oP 'id="[^"]+"' | cut -d'"' -f2)
        LIB_VERSION=$(echo "$line" | grep -oP 'version="[^"]+"' | cut -d'"' -f2)
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted packages.config dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# ELIXIR ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "mix.exs" ]; then
    echo "   💧 Found mix.exs (Elixir)"
    PROJECT_TYPE="elixir"
    
    grep -A 50 'defp deps' mix.exs | grep -E '^\s+{:' | while read line; do
        LIB_NAME=$(echo "$line" | grep -oE '{:[a-z_]+' | tr -d '{:')
        LIB_VERSION=$(echo "$line" | grep -oE '"~> [0-9]+\.[0-9]+(\.[0-9]+)?"' | sed 's/"~> //' | tr -d '"')
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Hex packages"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# SCALA ECOSYSTEM (SBT)
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "build.sbt" ]; then
    echo "   🔴 Found build.sbt (Scala)"
    PROJECT_TYPE="scala"
    
    grep -E 'libraryDependencies' build.sbt | grep -oE '"[^"]+"\s*%+\s*"[^"]+"\s*%+\s*"[^"]+"' | while read dep; do
        ORG=$(echo "$dep" | cut -d'"' -f2)
        NAME=$(echo "$dep" | cut -d'"' -f4)
        VERSION=$(echo "$dep" | cut -d'"' -f6)
        if [ -n "$NAME" ]; then
            echo "${ORG}:${NAME}|${VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted SBT dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# CLOJURE ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "deps.edn" ]; then
    echo "   🟢 Found deps.edn (Clojure)"
    PROJECT_TYPE="clojure"
    
    # Extract from :deps map
    grep -oE '[a-z]+/[a-z-]+\s+\{:mvn/version "[^"]+"\}' deps.edn | while read line; do
        LIB_NAME=$(echo "$line" | awk '{print $1}')
        LIB_VERSION=$(echo "$line" | grep -oE '"[^"]+"' | tr -d '"')
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Clojure deps"
fi

if [ -f "project.clj" ]; then
    echo "   🟢 Found project.clj (Clojure - Leiningen)"
    PROJECT_TYPE="clojure"
    
    grep -oE '\[[a-z]+/[a-z-]+ "[^"]+"\]' project.clj | while read line; do
        LIB_NAME=$(echo "$line" | awk '{print $1}' | tr -d '[')
        LIB_VERSION=$(echo "$line" | grep -oE '"[^"]+"' | tr -d '"')
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Leiningen dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# HASKELL ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "package.yaml" ] && grep -q "dependencies:" package.yaml 2>/dev/null; then
    echo "   🟣 Found package.yaml (Haskell - Stack)"
    PROJECT_TYPE="haskell"
    
    grep -A 100 '^dependencies:' package.yaml | grep -E '^- ' | while read line; do
        LIB_NAME=$(echo "$line" | sed 's/^- //' | cut -d' ' -f1)
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|unknown|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Haskell dependencies"
fi

if ls *.cabal 1> /dev/null 2>&1; then
    echo "   🟣 Found .cabal file (Haskell)"
    PROJECT_TYPE="haskell"
    
    for cabal in *.cabal; do
        grep -E 'build-depends:' "$cabal" -A 50 | grep -E '^\s+[a-z]' | while read line; do
            LIB_NAME=$(echo "$line" | awk '{print $1}' | tr -d ',')
            if [ -n "$LIB_NAME" ]; then
                echo "$LIB_NAME|unknown|auto|pending" >> /tmp/libraries_raw.txt
            fi
        done
    done
    
    echo "   ✅ Extracted Cabal dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# C/C++ ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "CMakeLists.txt" ]; then
    echo "   🔧 Found CMakeLists.txt (C/C++ - CMake)"
    PROJECT_TYPE="cpp-cmake"
    
    # Extract find_package calls
    grep -E 'find_package\(' CMakeLists.txt | while read line; do
        LIB_NAME=$(echo "$line" | grep -oE 'find_package\([A-Za-z0-9_]+' | sed 's/find_package(//')
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|unknown|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    # Extract FetchContent
    grep -E 'FetchContent_Declare\(' CMakeLists.txt | while read line; do
        LIB_NAME=$(echo "$line" | grep -oE 'FetchContent_Declare\([A-Za-z0-9_]+' | sed 's/FetchContent_Declare(//')
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|unknown|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted CMake dependencies"
fi

if [ -f "conanfile.txt" ] || [ -f "conanfile.py" ]; then
    echo "   🔧 Found Conan file (C/C++ - Conan)"
    PROJECT_TYPE="cpp-conan"
    
    if [ -f "conanfile.txt" ]; then
        grep -A 100 '\[requires\]' conanfile.txt | grep -E '^[a-z]' | while read line; do
            LIB_NAME=$(echo "$line" | cut -d'/' -f1)
            LIB_VERSION=$(echo "$line" | cut -d'/' -f2)
            if [ -n "$LIB_NAME" ]; then
                echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
            fi
        done
    fi
    
    echo "   ✅ Extracted Conan dependencies"
fi

if [ -f "vcpkg.json" ]; then
    echo "   🔧 Found vcpkg.json (C/C++ - vcpkg)"
    PROJECT_TYPE="cpp-vcpkg"
    
    grep -oE '"[a-z0-9-]+"' vcpkg.json | tr -d '"' | while read lib; do
        if [ -n "$lib" ] && [ "$lib" != "name" ] && [ "$lib" != "version" ]; then
            echo "$lib|unknown|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted vcpkg dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# JULIA ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "Project.toml" ]; then
    echo "   🟣 Found Project.toml (Julia)"
    PROJECT_TYPE="julia"
    
    grep -A 100 '\[deps\]' Project.toml | grep -E '^[A-Z]' | while read line; do
        LIB_NAME=$(echo "$line" | cut -d'=' -f1 | tr -d ' ')
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|unknown|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Julia packages"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# R ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "DESCRIPTION" ] && grep -q "^Package:" DESCRIPTION 2>/dev/null; then
    echo "   📊 Found DESCRIPTION (R Package)"
    PROJECT_TYPE="r"
    
    grep -E '^Imports:|^Depends:' DESCRIPTION | sed 's/Imports://;s/Depends://' | \
        tr ',' '\n' | while read lib; do
        LIB_NAME=$(echo "$lib" | tr -d ' ' | cut -d'(' -f1)
        if [ -n "$LIB_NAME" ] && [ "$LIB_NAME" != "R" ]; then
            echo "$LIB_NAME|unknown|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted R dependencies"
fi

if [ -f "renv.lock" ]; then
    echo "   📊 Found renv.lock (R - renv)"
    PROJECT_TYPE="r"
    
    grep -oE '"Package": "[^"]+"' renv.lock | cut -d'"' -f4 | while read lib; do
        if [ -n "$lib" ]; then
            echo "$lib|unknown|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted renv dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# PERL ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "cpanfile" ]; then
    echo "   🐪 Found cpanfile (Perl)"
    PROJECT_TYPE="perl"
    
    grep -E '^requires ' cpanfile | while read line; do
        LIB_NAME=$(echo "$line" | awk '{print $2}' | tr -d "';")
        LIB_VERSION=$(echo "$line" | grep -oE "'[0-9]+\.[0-9]+'" | tr -d "'")
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|${LIB_VERSION:-unknown}|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted CPAN dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# LUA ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "*.rockspec" ] 2>/dev/null || ls *.rockspec 1> /dev/null 2>&1; then
    echo "   🌙 Found .rockspec (Lua)"
    PROJECT_TYPE="lua"
    
    for rockspec in *.rockspec; do
        grep -A 20 'dependencies' "$rockspec" | grep -E '"[a-z]' | while read line; do
            LIB_NAME=$(echo "$line" | grep -oE '"[a-z][^"]*"' | tr -d '"' | head -1)
            if [ -n "$LIB_NAME" ]; then
                echo "$LIB_NAME|unknown|auto|pending" >> /tmp/libraries_raw.txt
            fi
        done
    done
    
    echo "   ✅ Extracted LuaRocks dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# ZIG ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "build.zig.zon" ]; then
    echo "   ⚡ Found build.zig.zon (Zig)"
    PROJECT_TYPE="zig"
    
    grep -E '^\s+\.[a-z]' build.zig.zon | while read line; do
        LIB_NAME=$(echo "$line" | grep -oE '\.[a-z_]+' | tr -d '.')
        if [ -n "$LIB_NAME" ]; then
            echo "$LIB_NAME|unknown|auto|pending" >> /tmp/libraries_raw.txt
        fi
    done
    
    echo "   ✅ Extracted Zig dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# NIM ECOSYSTEM
# ═══════════════════════════════════════════════════════════════════════════════

if [ -f "*.nimble" ] 2>/dev/null || ls *.nimble 1> /dev/null 2>&1; then
    echo "   👑 Found .nimble (Nim)"
    PROJECT_TYPE="nim"
    
    for nimble in *.nimble; do
        grep -E '^requires ' "$nimble" | while read line; do
            echo "$line" | tr ',' '\n' | while read dep; do
                LIB_NAME=$(echo "$dep" | sed 's/requires //' | awk '{print $1}' | tr -d '"')
                if [ -n "$LIB_NAME" ]; then
                    echo "$LIB_NAME|unknown|auto|pending" >> /tmp/libraries_raw.txt
                fi
            done
        done
    done
    
    echo "   ✅ Extracted Nimble dependencies"
fi

# ═══════════════════════════════════════════════════════════════════════════════
# SUMMARY
# ═══════════════════════════════════════════════════════════════════════════════

echo ""
echo "   Project Type: $PROJECT_TYPE"

if [ -f "/tmp/libraries_raw.txt" ]; then
    TOTAL_LIBS=$(cat /tmp/libraries_raw.txt | sort -u | wc -l | tr -d ' ')
    echo "   Total Libraries Found: $TOTAL_LIBS"
    
    # Deduplicate
    sort -u /tmp/libraries_raw.txt > /tmp/libraries_list.txt
    LIBRARIES_LIST=$(cat /tmp/libraries_list.txt)
else
    echo "   ⚠️ No dependencies found or project type not supported"
    echo "   Supported: Node.js, Python, Rust, Go, Ruby, PHP, Java, Kotlin,"
    echo "              Android, iOS, Flutter, React Native, .NET, Elixir,"
    echo "              Scala, Clojure, Haskell, C/C++, Julia, R, Perl, Lua, Zig, Nim"
    LIBRARIES_LIST=""
fi

echo ""
echo "✅ Dependency extraction complete"
```

### Step 3: Extract Project Usage Patterns from Basepoints

Analyze existing module basepoints to understand how libraries are used:

```bash
echo "🔍 Extracting library usage patterns from basepoints..."

# Skip if no libraries found
if [ -z "$LIBRARIES_LIST" ]; then
    echo "   ⚠️ No libraries to analyze"
else
    # For each library identified:
    cat /tmp/libraries_list.txt | while read library_line; do
        LIBRARY_NAME=$(echo "$library_line" | cut -d'|' -f1)
        
        echo "   Analyzing usage of: $LIBRARY_NAME"
        
        # Search all module basepoints for mentions of this library
        USAGE_PATTERNS=""
        
        # Find all basepoint files
        find "$BASEPOINTS_DIR" -name "agent-base-*.md" -type f 2>/dev/null | while read basepoint_file; do
            # Check if this basepoint mentions the library
            if grep -qi "$LIBRARY_NAME" "$basepoint_file" 2>/dev/null; then
                # Extract relevant sections
                PATTERNS=$(grep -A 5 -i "$LIBRARY_NAME" "$basepoint_file" | head -20)
                echo "### From $(basename "$basepoint_file")"
                echo "$PATTERNS"
                echo ""
            fi
        done > "/tmp/library-usage-$LIBRARY_NAME.txt"
    done
fi

echo "✅ Library usage patterns extracted from basepoints"
```

### Step 4: Analyze Codebase for Implementation Details

Deep-dive into the codebase to understand actual library usage:

```bash
echo "🔍 Analyzing codebase for library implementation details..."

# Define file extensions based on project type
get_file_extensions() {
    case "$PROJECT_TYPE" in
        nodejs|react-native)
            echo "*.ts *.tsx *.js *.jsx *.mjs *.cjs"
            ;;
        python)
            echo "*.py *.pyx *.pyi"
            ;;
        rust)
            echo "*.rs"
            ;;
        go)
            echo "*.go"
            ;;
        ruby)
            echo "*.rb *.rake *.gemspec"
            ;;
        php)
            echo "*.php"
            ;;
        java-maven|java-gradle)
            echo "*.java"
            ;;
        android)
            echo "*.kt *.java *.kts"
            ;;
        ios-cocoapods|ios-spm|ios-xcode)
            echo "*.swift *.m *.mm *.h"
            ;;
        flutter)
            echo "*.dart"
            ;;
        dotnet)
            echo "*.cs *.fs *.vb"
            ;;
        elixir)
            echo "*.ex *.exs"
            ;;
        scala)
            echo "*.scala *.sc"
            ;;
        clojure)
            echo "*.clj *.cljs *.cljc *.edn"
            ;;
        haskell)
            echo "*.hs *.lhs"
            ;;
        cpp-cmake|cpp-conan|cpp-vcpkg)
            echo "*.cpp *.cc *.cxx *.c *.h *.hpp *.hxx"
            ;;
        julia)
            echo "*.jl"
            ;;
        r)
            echo "*.R *.r *.Rmd"
            ;;
        perl)
            echo "*.pl *.pm *.t"
            ;;
        lua)
            echo "*.lua"
            ;;
        zig)
            echo "*.zig"
            ;;
        nim)
            echo "*.nim *.nims"
            ;;
        *)
            echo "*.ts *.js *.py *.go *.rs *.java *.kt *.swift *.dart *.cs *.cpp *.c"
            ;;
    esac
}

FILE_EXTENSIONS=$(get_file_extensions)

if [ -n "$LIBRARIES_LIST" ]; then
    cat /tmp/libraries_list.txt | while read library_line; do
        LIBRARY_NAME=$(echo "$library_line" | cut -d'|' -f1)
        
        echo "   Analyzing codebase usage of: $LIBRARY_NAME"
        
        # Build include pattern for grep
        INCLUDE_PATTERN=""
        for ext in $FILE_EXTENSIONS; do
            INCLUDE_PATTERN="$INCLUDE_PATTERN --include=$ext"
        done
        
        # Search codebase for import/require/use statements
        IMPORT_PATTERNS=$(grep -rn $INCLUDE_PATTERN \
            -E "import.*$LIBRARY_NAME|require.*$LIBRARY_NAME|from ['\"]$LIBRARY_NAME|use $LIBRARY_NAME|using $LIBRARY_NAME" \
            . 2>/dev/null | head -50)
        
        # Identify which files use this library
        FILES_USING=$(echo "$IMPORT_PATTERNS" | cut -d':' -f1 | sort -u)
        
        # Extract usage patterns from those files
        IMPLEMENTATION_PATTERNS=""
        for file in $FILES_USING; do
            if [ -f "$file" ]; then
                # Extract function calls and patterns
                PATTERNS=$(grep -n "$LIBRARY_NAME" "$file" 2>/dev/null | head -20)
                if [ -n "$PATTERNS" ]; then
                    IMPLEMENTATION_PATTERNS="${IMPLEMENTATION_PATTERNS}

### $file
$PATTERNS
"
                fi
            fi
        done
        
        # Store implementation patterns
        echo "$IMPLEMENTATION_PATTERNS" > "/tmp/library-impl-$LIBRARY_NAME.txt"
    done
fi

echo "✅ Codebase implementation patterns analyzed"
```

### Step 5: Research Official Documentation (Version-Specific)

For each library, research official documentation **for the specific version we're using**.

**IMPORTANT**: Focus research on OUR version, not generic/latest documentation. Version-specific behavior, breaking changes, and deprecations matter.

```bash
echo "🔍 Researching official documentation (version-specific)..."

if [ -n "$LIBRARIES_LIST" ]; then
    cat /tmp/libraries_list.txt | while read library_line; do
        LIBRARY_NAME=$(echo "$library_line" | cut -d'|' -f1)
        LIBRARY_VERSION=$(echo "$library_line" | cut -d'|' -f2)
        LIBRARY_IMPORTANCE=$(echo "$library_line" | cut -d'|' -f4)
        
        echo "   Researching: $LIBRARY_NAME v$LIBRARY_VERSION (importance: ${LIBRARY_IMPORTANCE:-pending})"
        
        # ═══════════════════════════════════════════════════════════════════════
        # VERSION-SPECIFIC RESEARCH STRATEGY
        # ═══════════════════════════════════════════════════════════════════════
        # 
        # 1. ALWAYS include version in search queries
        # 2. Focus on documentation for OUR major version
        # 3. Look for version-specific migration guides if upgrading
        # 4. Check for deprecations in OUR version
        # 5. Find breaking changes between versions
        #
        # DO NOT research generic "latest" documentation unless version unknown
        # ═══════════════════════════════════════════════════════════════════════
        
        # Extract major version for broader searches
        MAJOR_VERSION=$(echo "$LIBRARY_VERSION" | cut -d'.' -f1)
        MINOR_VERSION=$(echo "$LIBRARY_VERSION" | cut -d'.' -f1,2)
        
        # Determine research depth based on importance
        case "$LIBRARY_IMPORTANCE" in
            "critical")
                # Deep research for critical libraries - VERSION SPECIFIC
                echo "   → Deep research (critical library)"
                
                # Primary: Version-specific documentation
                # web_search("$LIBRARY_NAME $LIBRARY_VERSION documentation")
                # web_search("$LIBRARY_NAME v$MAJOR_VERSION official docs")
                
                # Architecture for OUR version
                # web_search("$LIBRARY_NAME $MAJOR_VERSION architecture internals")
                # web_search("$LIBRARY_NAME $LIBRARY_VERSION how it works under the hood")
                
                # Version-specific best practices
                # web_search("$LIBRARY_NAME $MAJOR_VERSION best practices")
                # web_search("$LIBRARY_NAME v$MINOR_VERSION patterns")
                
                # Version-specific issues and gotchas
                # web_search("$LIBRARY_NAME $LIBRARY_VERSION known issues")
                # web_search("$LIBRARY_NAME $MAJOR_VERSION breaking changes")
                # web_search("$LIBRARY_NAME $LIBRARY_VERSION deprecations")
                
                # Migration info (if relevant)
                # web_search("$LIBRARY_NAME migration guide $MAJOR_VERSION")
                # web_search("$LIBRARY_NAME upgrade to $MAJOR_VERSION")
                
                RESEARCH_DEPTH="deep"
                ;;
            "important")
                # Moderate research for important libraries - VERSION SPECIFIC
                echo "   → Moderate research (important library)"
                
                # Version-specific documentation
                # web_search("$LIBRARY_NAME $LIBRARY_VERSION documentation")
                # web_search("$LIBRARY_NAME v$MAJOR_VERSION guide")
                
                # Version-specific best practices
                # web_search("$LIBRARY_NAME $MAJOR_VERSION best practices")
                
                # Common issues for this version
                # web_search("$LIBRARY_NAME $LIBRARY_VERSION common issues")
                
                RESEARCH_DEPTH="moderate"
                ;;
            *)
                # Basic research for supporting libraries - VERSION AWARE
                echo "   → Basic research (supporting library)"
                
                # Just version-specific docs
                # web_search("$LIBRARY_NAME $LIBRARY_VERSION documentation")
                # If version unknown: web_search("$LIBRARY_NAME documentation")
                
                RESEARCH_DEPTH="basic"
                ;;
        esac
        
        # Store version info for basepoint
        echo "LIBRARY_VERSION=$LIBRARY_VERSION" > "/tmp/library-research-$LIBRARY_NAME.txt"
        echo "MAJOR_VERSION=$MAJOR_VERSION" >> "/tmp/library-research-$LIBRARY_NAME.txt"
        echo "RESEARCH_DEPTH=$RESEARCH_DEPTH" >> "/tmp/library-research-$LIBRARY_NAME.txt"
        
        # ═══════════════════════════════════════════════════════════════════════
        # RESEARCH OUTPUT STRUCTURE (to be filled by AI agent)
        # ═══════════════════════════════════════════════════════════════════════
        # 
        # For each library, document:
        #
        # 1. VERSION-SPECIFIC BEHAVIOR
        #    - How this specific version works
        #    - Key differences from other versions
        #    - Deprecations in this version
        #
        # 2. INTERNALS (for critical libraries)
        #    - Core architecture concepts
        #    - Execution model
        #    - Memory/performance characteristics
        #
        # 3. PATTERNS FOR THIS VERSION
        #    - Recommended patterns for v$MAJOR_VERSION
        #    - Anti-patterns to avoid
        #    - Migration patterns if upgrading
        #
        # 4. KNOWN ISSUES
        #    - Bugs in this specific version
        #    - Workarounds
        #    - Fixed in newer versions (upgrade candidates)
        #
        # ═══════════════════════════════════════════════════════════════════════
    done
fi

echo "✅ Version-specific documentation researched"
```

#### Research Focus Guidelines

When researching libraries, prioritize information relevant to **our specific version**:

| Priority | Focus Area | Why It Matters |
|----------|------------|----------------|
| **1** | Version-specific docs | APIs and behavior differ between versions |
| **2** | Breaking changes | Know what changed from previous versions |
| **3** | Deprecations | Avoid using deprecated APIs |
| **4** | Version-specific bugs | Know workarounds for our version |
| **5** | Upgrade path | Understand what changes if we upgrade |

**DO NOT** waste time on:
- Generic documentation without version context
- Features only available in newer versions we don't use
- Deep internals for supporting/utility libraries
- Historical versions we've never used

### Step 5.5: Extract Security Information from Enriched Knowledge

If security notes exist from the initial `adapt-to-product` research, extract relevant information:

```bash
echo "🔒 Extracting security information from enriched knowledge..."

GEIST_PATH="geist"

if [ -n "$LIBRARIES_LIST" ]; then
    cat /tmp/libraries_list.txt | while read library_line; do
        LIBRARY_NAME=$(echo "$library_line" | cut -d'|' -f1)
        
        SECURITY_NOTES=""
        if [ -f "$GEIST_PATH/config/enriched-knowledge/security-notes.md" ]; then
            echo "   Checking security notes for: $LIBRARY_NAME"
            # Search for library-specific security notes (CVEs, vulnerabilities, security considerations)
            SECURITY_NOTES=$(grep -A 20 -i "$LIBRARY_NAME" "$GEIST_PATH/config/enriched-knowledge/security-notes.md" 2>/dev/null | head -25)
            
            if [ -n "$SECURITY_NOTES" ]; then
                echo "   ⚠️ Security notes found for $LIBRARY_NAME"
                echo "$SECURITY_NOTES" > "/tmp/library-security-$LIBRARY_NAME.txt"
            else
                echo "   ✅ No security issues found for $LIBRARY_NAME"
                echo "No known security issues documented." > "/tmp/library-security-$LIBRARY_NAME.txt"
            fi
        else
            echo "   ℹ️ No enriched-knowledge/security-notes.md found"
            echo "Security notes not available. Run /adapt-to-product to generate." > "/tmp/library-security-$LIBRARY_NAME.txt"
        fi
    done
fi

echo "✅ Security information extracted"
```

### Step 5.6: Extract Version Information from Enriched Knowledge

If version analysis exists from the initial research, extract relevant information:

```bash
echo "📦 Extracting version information from enriched knowledge..."

if [ -n "$LIBRARIES_LIST" ]; then
    cat /tmp/libraries_list.txt | while read library_line; do
        LIBRARY_NAME=$(echo "$library_line" | cut -d'|' -f1)
        
        VERSION_STATUS=""
        if [ -f "$GEIST_PATH/config/enriched-knowledge/version-analysis.md" ]; then
            echo "   Checking version status for: $LIBRARY_NAME"
            # Search for library-specific version info (current version, latest, upgrade recommendations)
            VERSION_STATUS=$(grep -A 10 -i "$LIBRARY_NAME" "$GEIST_PATH/config/enriched-knowledge/version-analysis.md" 2>/dev/null | head -15)
            
            if [ -n "$VERSION_STATUS" ]; then
                echo "   📋 Version status found for $LIBRARY_NAME"
                echo "$VERSION_STATUS" > "/tmp/library-version-$LIBRARY_NAME.txt"
            else
                echo "   ℹ️ No version analysis for $LIBRARY_NAME"
                echo "Version analysis not available for this library." > "/tmp/library-version-$LIBRARY_NAME.txt"
            fi
        else
            echo "   ℹ️ No enriched-knowledge/version-analysis.md found"
            echo "Version analysis not available. Run /adapt-to-product to generate." > "/tmp/library-version-$LIBRARY_NAME.txt"
        fi
    done
fi

echo "✅ Version information extracted"
```

### Step 6: Classify Library Importance

Classify each library by importance based on usage analysis:

```bash
echo "📊 Classifying library importance..."

classify_importance() {
    local library_name="$1"
    local usage_count="$2"
    local is_framework="$3"
    local is_core_infrastructure="$4"
    
    # Critical: Framework, core infrastructure, or used in 10+ files
    if [ "$is_framework" = "true" ] || [ "$is_core_infrastructure" = "true" ] || [ "$usage_count" -ge 10 ]; then
        echo "critical"
    # Important: Used in 3-9 files
    elif [ "$usage_count" -ge 3 ]; then
        echo "important"
    # Supporting: Used in 1-2 files
    else
        echo "supporting"
    fi
}

FILE_EXTENSIONS=$(get_file_extensions)

# Build include pattern
INCLUDE_PATTERN=""
for ext in $FILE_EXTENSIONS; do
    INCLUDE_PATTERN="$INCLUDE_PATTERN --include=$ext"
done

if [ -n "$LIBRARIES_LIST" ]; then
    > /tmp/libraries_classified.txt
    
    cat /tmp/libraries_list.txt | while read library_line; do
        LIBRARY_NAME=$(echo "$library_line" | cut -d'|' -f1)
        LIBRARY_VERSION=$(echo "$library_line" | cut -d'|' -f2)
        
        # Count files using this library
        USAGE_COUNT=$(grep -rl $INCLUDE_PATTERN "$LIBRARY_NAME" . 2>/dev/null | wc -l | tr -d ' ')
        
        # Check if it's a framework or core infrastructure
        IS_FRAMEWORK="false"
        IS_CORE="false"
        
        if echo "$TECH_STACK_CONTENT" | grep -qi "framework.*$LIBRARY_NAME" 2>/dev/null; then
            IS_FRAMEWORK="true"
        fi
        
        if echo "$TECH_STACK_CONTENT" | grep -qiE "core.*$LIBRARY_NAME|infrastructure.*$LIBRARY_NAME" 2>/dev/null; then
            IS_CORE="true"
        fi
        
        IMPORTANCE=$(classify_importance "$LIBRARY_NAME" "$USAGE_COUNT" "$IS_FRAMEWORK" "$IS_CORE")
        
        echo "   $LIBRARY_NAME: $IMPORTANCE (used in $USAGE_COUNT files)"
        echo "$LIBRARY_NAME|$LIBRARY_VERSION|auto|$IMPORTANCE" >> /tmp/libraries_classified.txt
    done
    
    # Update libraries list with classifications
    mv /tmp/libraries_classified.txt /tmp/libraries_list.txt
fi

echo "✅ Library importance classified"
```

### Step 7: Categorize Libraries

Categorize each library by its domain:

```bash
echo "📁 Categorizing libraries..."

categorize_library() {
    local library_name="$1"
    local library_description="$2"
    
    # Convert to lowercase for matching
    local lib_lower=$(echo "$library_name" | tr '[:upper:]' '[:lower:]')
    
    # Data: databases, ORM, data access
    if echo "$lib_lower" | grep -qiE "sql|orm|mongo|redis|postgres|mysql|prisma|sequelize|typeorm|hibernate|room|realm|core-data|sqlite|firebase-database|supabase|dynamo|cassandra|neo4j|elastic"; then
        echo "data"
    # Infrastructure: networking, HTTP, threading, system
    elif echo "$lib_lower" | grep -qiE "http|axios|fetch|request|retrofit|alamofire|okhttp|grpc|graphql|apollo|socket|websocket|async|thread|concurrent|queue|worker|redis|rabbitmq|kafka|aws-sdk|azure|gcp|docker|kubernetes"; then
        echo "infrastructure"
    # Framework: web frameworks, UI frameworks, app frameworks
    elif echo "$lib_lower" | grep -qiE "react|vue|angular|svelte|next|nuxt|remix|express|fastapi|django|flask|rails|spring|laravel|symfony|gin|echo|fiber|actix|axum|rocket|swiftui|uikit|jetpack|compose|flutter|electron|tauri|qt"; then
        echo "framework"
    # UI: UI components, styling, animation
    elif echo "$lib_lower" | grep -qiE "tailwind|styled|emotion|material|chakra|ant-design|bootstrap|bulma|foundation|animate|framer|lottie|rive|css|sass|less|postcss"; then
        echo "ui"
    # State: state management
    elif echo "$lib_lower" | grep -qiE "redux|zustand|mobx|recoil|jotai|valtio|pinia|vuex|ngrx|bloc|provider|riverpod|getx"; then
        echo "state"
    # Testing: testing frameworks and utilities
    elif echo "$lib_lower" | grep -qiE "jest|vitest|mocha|chai|jasmine|pytest|unittest|rspec|junit|testng|xctest|espresso|detox|cypress|playwright|selenium|testing-library"; then
        echo "testing"
    # Auth: authentication and authorization
    elif echo "$lib_lower" | grep -qiE "auth|oauth|jwt|passport|clerk|auth0|firebase-auth|cognito|keycloak|okta|session|cookie"; then
        echo "auth"
    # Validation: validation and schema
    elif echo "$lib_lower" | grep -qiE "zod|yup|joi|validator|class-validator|pydantic|marshmallow|cerberus|json-schema"; then
        echo "validation"
    # Logging/Monitoring: observability
    elif echo "$lib_lower" | grep -qiE "log|winston|pino|bunyan|sentry|datadog|newrelic|prometheus|grafana|opentelemetry|jaeger|zipkin"; then
        echo "observability"
    # Build: build tools and bundlers
    elif echo "$lib_lower" | grep -qiE "webpack|vite|rollup|esbuild|parcel|turbo|nx|lerna|gradle|maven|cmake|cargo|go-build|xcode|fastlane"; then
        echo "build"
    # Domain: business logic, domain-specific
    elif echo "$library_description" | grep -qiE "business|domain|model|entity|service"; then
        echo "domain"
    # Util: everything else
    else
        echo "util"
    fi
}

# Create category directories
mkdir -p "$LIBRARIES_DIR/data"
mkdir -p "$LIBRARIES_DIR/domain"
mkdir -p "$LIBRARIES_DIR/util"
mkdir -p "$LIBRARIES_DIR/infrastructure"
mkdir -p "$LIBRARIES_DIR/framework"
mkdir -p "$LIBRARIES_DIR/ui"
mkdir -p "$LIBRARIES_DIR/state"
mkdir -p "$LIBRARIES_DIR/testing"
mkdir -p "$LIBRARIES_DIR/auth"
mkdir -p "$LIBRARIES_DIR/validation"
mkdir -p "$LIBRARIES_DIR/observability"
mkdir -p "$LIBRARIES_DIR/build"

echo "✅ Library categories created"
```

### Step 8: Generate Library Basepoint Files

For each library, generate a comprehensive basepoint file:

```bash
echo "📝 Generating library basepoint files..."

if [ -n "$LIBRARIES_LIST" ] && [ -f "/tmp/libraries_list.txt" ]; then
    cat /tmp/libraries_list.txt | while read library_line; do
        LIBRARY_NAME=$(echo "$library_line" | cut -d'|' -f1)
        LIBRARY_VERSION=$(echo "$library_line" | cut -d'|' -f2)
        LIBRARY_IMPORTANCE=$(echo "$library_line" | cut -d'|' -f4)
        
        # Categorize the library
        LIBRARY_CATEGORY=$(categorize_library "$LIBRARY_NAME" "")
        
        # Load gathered knowledge
        USAGE_PATTERNS=$(cat "/tmp/library-usage-$LIBRARY_NAME.txt" 2>/dev/null)
        IMPL_PATTERNS=$(cat "/tmp/library-impl-$LIBRARY_NAME.txt" 2>/dev/null)
        
        # Determine output path
        LIBRARY_SLUG=$(echo "$LIBRARY_NAME" | tr '[:upper:]' '[:lower:]' | tr ' /:' '---' | tr -cd 'a-z0-9-')
        OUTPUT_FILE="$LIBRARIES_DIR/$LIBRARY_CATEGORY/$LIBRARY_SLUG.md"
        
        echo "   Creating: $OUTPUT_FILE"
        
        # Extract major version
        MAJOR_VERSION=$(echo "$LIBRARY_VERSION" | cut -d'.' -f1)
        
        # Generate basepoint content
        cat > "$OUTPUT_FILE" << BASEPOINT_EOF
# Library Basepoint: $LIBRARY_NAME

## Overview

| Attribute | Value |
|-----------|-------|
| **Library** | $LIBRARY_NAME |
| **Our Version** | $LIBRARY_VERSION |
| **Major Version** | v$MAJOR_VERSION |
| **Category** | $LIBRARY_CATEGORY |
| **Importance** | $LIBRARY_IMPORTANCE |
| **Project Type** | $PROJECT_TYPE |
| **Generated** | $(date) |

> ⚠️ **Version Notice**: This basepoint documents behavior specific to **v$LIBRARY_VERSION**. 
> APIs, patterns, and internals may differ in other versions.

---

## Version-Specific Information

### Our Version: $LIBRARY_VERSION

[Document version-specific behavior, APIs, and characteristics for v$LIBRARY_VERSION]

### Key Features in v$MAJOR_VERSION

[List the main features available in this major version]

- [Feature 1]
- [Feature 2]

### Deprecations in Our Version

[List any deprecated APIs or patterns in v$LIBRARY_VERSION that we should avoid or migrate away from]

- [Deprecated API 1] - Use [Alternative] instead
- [Deprecated Pattern 2] - Replaced by [New Pattern]

### Breaking Changes from Previous Versions

[Document breaking changes if we upgraded to this version, or changes to be aware of]

- [Change 1] - How it affects our code
- [Change 2] - Migration notes

---

## Project Usage

### How We Use This Library

[Summary of how this library is used in our project, based on basepoints and codebase analysis]

### Usage Patterns from Basepoints

$USAGE_PATTERNS

### Implementation Patterns from Codebase

$IMPL_PATTERNS

---

## How It Works (v$MAJOR_VERSION)

### Core Architecture

[Document how this library works internally - specific to our version]

- Execution model
- Key abstractions
- Data flow

### Key Internals

[For critical libraries: document internal workings relevant to our usage]

- [Internal concept 1]
- [Internal concept 2]

### Performance Characteristics

[Document performance characteristics specific to v$LIBRARY_VERSION]

- Memory usage patterns
- CPU characteristics
- Known bottlenecks

---

## Boundaries

### What We Use

[List the specific features, APIs, and patterns from this library that we actively use]

- [Feature 1]
- [Feature 2]
- [API/Pattern 3]

### What We Don't Use

[List features and APIs that are available but NOT used in our project - important for understanding scope]

- [Unused Feature 1]
- [Unused Feature 2]

---

## Patterns (v$MAJOR_VERSION)

### Usage Patterns

[Document common usage patterns observed in our codebase - specific to v$LIBRARY_VERSION]

### Integration Patterns

[How this library integrates with other libraries in our project]

### Anti-Patterns to Avoid

[Patterns that don't work well in v$MAJOR_VERSION or are deprecated]

---

## Best Practices (v$MAJOR_VERSION)

### Official Guidelines for v$MAJOR_VERSION

[Best practices from official documentation for this version]

### Project-Specific Practices

[Best practices specific to how we use this library in our project]

### Version-Specific Recommendations

[Recommendations that are specific to v$LIBRARY_VERSION]

---

## Troubleshooting (v$LIBRARY_VERSION)

### Known Issues in v$LIBRARY_VERSION

[Bugs and issues specific to our version]

| Issue | Workaround | Fixed In |
|-------|------------|----------|
| [Issue 1] | [Workaround] | v[X.Y.Z] |

### Common Problems

[Common problems and their solutions - from official docs and our experience]

### Debugging Strategies

[How to debug issues with this library in v$MAJOR_VERSION]

### Version-Specific Gotchas

[Edge cases and pitfalls specific to v$LIBRARY_VERSION]

---

## Upgrade Considerations

### Current Version: $LIBRARY_VERSION

### Upgrade Path

[If considering upgrade, document the path and breaking changes]

| From | To | Breaking Changes |
|------|-----|------------------|
| v$LIBRARY_VERSION | v[Next] | [Changes] |

### Why We're on v$LIBRARY_VERSION

[Document why we're using this version - stability, compatibility, features needed]

---

## Resources

### Official Documentation (v$MAJOR_VERSION)
- [Official Docs for v$MAJOR_VERSION](URL)
- [API Reference for v$LIBRARY_VERSION](URL)
- [Migration Guide](URL)

### Internal References
- [List relevant module basepoints that use this library]

---

*Generated by Geist Library Basepoints Workflow*
*Importance: $LIBRARY_IMPORTANCE | Category: $LIBRARY_CATEGORY*
BASEPOINT_EOF

    done
fi

echo "✅ Library basepoint files generated"
```

### Step 9: Detect Solution-Specific Patterns

Detect when different solutions from the same library are used:

```bash
echo "🔍 Detecting solution-specific patterns..."

# For libraries with multiple usage patterns, create separate basepoints
# Examples:
# - asyncio: event loop vs task groups
# - SQLAlchemy: ORM vs Core
# - React: class components vs hooks
# - Jetpack Compose vs XML layouts
# - SwiftUI vs UIKit

if [ -n "$LIBRARIES_LIST" ] && [ -f "/tmp/libraries_list.txt" ]; then
    cat /tmp/libraries_list.txt | while read library_line; do
        LIBRARY_NAME=$(echo "$library_line" | cut -d'|' -f1)
        LIBRARY_CATEGORY=$(categorize_library "$LIBRARY_NAME" "")
        
        # Analyze usage patterns for distinct solutions
        IMPL_PATTERNS=$(cat "/tmp/library-impl-$LIBRARY_NAME.txt" 2>/dev/null)
        
        # Check for distinct solution patterns
        # [AI agent analyzes patterns and identifies distinct solutions]
        
        # If distinct solutions found, create solution-specific basepoints
        # Example: $LIBRARIES_DIR/$LIBRARY_CATEGORY/$LIBRARY_NAME-solution1.md
    done
fi

echo "✅ Solution-specific patterns detected"
```

### Step 10: Generate Library Index

Create an index of all library basepoints:

```bash
echo "📋 Generating library index..."

cat > "$LIBRARIES_DIR/README.md" << INDEX_EOF
# Library Basepoints Index

## Overview

This folder contains basepoints for libraries used in the project, organized by category.
Each basepoint combines:
- Project basepoints knowledge (how we use the library)
- Official documentation research (best practices, architecture)
- Codebase analysis (actual implementation patterns)

## Project Type: $PROJECT_TYPE

## Categories

### Framework (\`framework/\`)
Web frameworks, UI frameworks, and app frameworks.

### UI (\`ui/\`)
UI components, styling, and animation libraries.

### State (\`state/\`)
State management libraries.

### Data (\`data/\`)
Libraries for data access, databases, ORM, and persistence.

### Infrastructure (\`infrastructure/\`)
Libraries for networking, HTTP, threading, and system-level operations.

### Auth (\`auth/\`)
Authentication and authorization libraries.

### Validation (\`validation/\`)
Validation and schema libraries.

### Testing (\`testing/\`)
Testing frameworks and utilities.

### Observability (\`observability/\`)
Logging, monitoring, and tracing libraries.

### Build (\`build/\`)
Build tools and bundlers.

### Domain (\`domain/\`)
Libraries for domain logic, business rules, and core models.

### Util (\`util/\`)
Utility libraries, helpers, and common functions.

## Library Index

| Library | Category | Importance | Basepoint |
|---------|----------|------------|-----------|
INDEX_EOF

# Add each library to the index
if [ -f "/tmp/libraries_list.txt" ]; then
    cat /tmp/libraries_list.txt | while read library_line; do
        LIBRARY_NAME=$(echo "$library_line" | cut -d'|' -f1)
        LIBRARY_IMPORTANCE=$(echo "$library_line" | cut -d'|' -f4)
        LIBRARY_CATEGORY=$(categorize_library "$LIBRARY_NAME" "")
        LIBRARY_SLUG=$(echo "$LIBRARY_NAME" | tr '[:upper:]' '[:lower:]' | tr ' /:' '---' | tr -cd 'a-z0-9-')
        
        echo "| $LIBRARY_NAME | $LIBRARY_CATEGORY | $LIBRARY_IMPORTANCE | [$LIBRARY_SLUG](./$LIBRARY_CATEGORY/$LIBRARY_SLUG.md) |" >> "$LIBRARIES_DIR/README.md"
    done
fi

cat >> "$LIBRARIES_DIR/README.md" << FOOTER_EOF

## Usage

These library basepoints are used by:
- \`extract-library-basepoints-knowledge\` workflow for knowledge extraction
- Spec/implementation commands for context enrichment
- \`fix-bug\` command for library-specific troubleshooting

## Regeneration

To regenerate library basepoints after adding new libraries:
1. Update \`geist/product/tech-stack.md\`
2. Run \`/update-basepoints-and-redeploy\`

## Supported Project Types

This workflow supports dependency extraction from:

| Ecosystem | Files Detected |
|-----------|----------------|
| **Node.js/TypeScript** | package.json |
| **Python** | requirements.txt, pyproject.toml, Pipfile |
| **Rust** | Cargo.toml |
| **Go** | go.mod |
| **Ruby** | Gemfile |
| **PHP** | composer.json |
| **Java (Maven)** | pom.xml |
| **Java/Kotlin (Gradle)** | build.gradle, build.gradle.kts |
| **Android** | app/build.gradle, app/build.gradle.kts |
| **iOS (CocoaPods)** | Podfile |
| **iOS (SPM)** | Package.swift |
| **Flutter/Dart** | pubspec.yaml |
| **React Native** | package.json (with react-native) |
| **.NET/C#** | *.csproj, packages.config |
| **Elixir** | mix.exs |
| **Scala** | build.sbt |
| **Clojure** | deps.edn, project.clj |
| **Haskell** | package.yaml, *.cabal |
| **C/C++ (CMake)** | CMakeLists.txt |
| **C/C++ (Conan)** | conanfile.txt, conanfile.py |
| **C/C++ (vcpkg)** | vcpkg.json |
| **Julia** | Project.toml |
| **R** | DESCRIPTION, renv.lock |
| **Perl** | cpanfile |
| **Lua** | *.rockspec |
| **Zig** | build.zig.zon |
| **Nim** | *.nimble |

---

*Generated: $(date)*
*By: Geist Library Basepoints Workflow*
FOOTER_EOF

echo "✅ Library index generated"
```

### Step 11: Cleanup and Return Status

```bash
echo ""
echo "═══════════════════════════════════════════════════════"
echo "  LIBRARY BASEPOINTS GENERATION COMPLETE"
echo "═══════════════════════════════════════════════════════"
echo ""
echo "  Project Type: $PROJECT_TYPE"

if [ -f "/tmp/libraries_list.txt" ]; then
    echo "  Libraries Processed: $(cat /tmp/libraries_list.txt | wc -l | tr -d ' ')"
    echo "  Critical: $(grep -c "|critical$" /tmp/libraries_list.txt 2>/dev/null || echo 0)"
    echo "  Important: $(grep -c "|important$" /tmp/libraries_list.txt 2>/dev/null || echo 0)"
    echo "  Supporting: $(grep -c "|supporting$" /tmp/libraries_list.txt 2>/dev/null || echo 0)"
else
    echo "  Libraries Processed: 0"
    echo "  ⚠️ No dependencies detected for this project type"
fi

echo ""
echo "  Output Location: $LIBRARIES_DIR"
echo "  Index: $LIBRARIES_DIR/README.md"
echo ""
echo "  Categories:"
for category in framework ui state data infrastructure auth validation testing observability build domain util; do
    COUNT=$(ls -1 "$LIBRARIES_DIR/$category" 2>/dev/null | wc -l | tr -d ' ')
    if [ "$COUNT" -gt 0 ]; then
        echo "    - $category/: $COUNT basepoints"
    fi
done
echo ""

# Cleanup temp files
rm -f /tmp/libraries_raw.txt /tmp/libraries_list.txt /tmp/libraries_classified.txt
rm -f /tmp/library-usage-*.txt /tmp/library-impl-*.txt
rm -f /tmp/library-security-*.txt /tmp/library-version-*.txt

echo "═══════════════════════════════════════════════════════"
```

---

## Knowledge Sources Combined

This workflow combines knowledge from multiple sources:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    LIBRARY BASEPOINT KNOWLEDGE SOURCES                       │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  PRODUCT DOCS   │     │   BASEPOINTS    │     │    CODEBASE     │
│                 │     │                 │     │                 │
│ tech-stack.md   │     │ headquarter.md  │     │ Source files    │
│ • Libraries     │     │ • Project       │     │ • Imports       │
│ • Versions      │     │   overview      │     │ • Usage         │
│ • Categories    │     │                 │     │   patterns      │
│                 │     │ agent-base-*.md │     │ • Integration   │
│                 │     │ • Module usage  │     │   points        │
│                 │     │ • Patterns      │     │                 │
└────────┬────────┘     └────────┬────────┘     └────────┬────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                                 ▼
                    ┌─────────────────────┐
                    │  OFFICIAL RESEARCH  │
                    │                     │
                    │ • Documentation     │
                    │ • Best practices    │
                    │ • Architecture      │
                    │ • Troubleshooting   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │  LIBRARY BASEPOINT  │
                    │                     │
                    │ • How WE use it     │
                    │ • What WE use       │
                    │ • What WE don't use │
                    │ • OUR patterns      │
                    │ • Official guidance │
                    │ • Troubleshooting   │
                    └─────────────────────┘
```

## Supported Ecosystems

| Category | Languages/Platforms | Package Managers |
|----------|---------------------|------------------|
| **Web Frontend** | JavaScript, TypeScript | npm, yarn, pnpm |
| **Web Backend** | Node.js, Python, Ruby, PHP, Go, Rust | npm, pip, gem, composer, go mod, cargo |
| **Mobile** | Swift, Kotlin, Dart, JavaScript | CocoaPods, SPM, Gradle, pub, npm |
| **Native** | Java, Kotlin, C#, C/C++ | Maven, Gradle, NuGet, CMake, Conan, vcpkg |
| **Data/ML** | Python, R, Julia | pip, CRAN, Pkg |
| **Functional** | Elixir, Scala, Clojure, Haskell | Hex, SBT, Leiningen, Cabal |
| **Systems** | Rust, Go, C/C++, Zig, Nim | Cargo, go mod, CMake, zig, nimble |
| **Scripting** | Perl, Lua | CPAN, LuaRocks |

## Important Constraints

- Must run AFTER module basepoints are generated (Phase 5-7)
- Must combine project-specific knowledge with official documentation
- Must document boundaries (what is/isn't used)
- Must classify importance based on actual usage
- Must categorize by domain (framework, ui, state, data, infrastructure, auth, validation, testing, observability, build, domain, util)
- Must be technology-agnostic in structure
- Research depth must match library importance
- Supports 25+ package manager formats across all major ecosystems
