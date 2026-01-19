## Codebase analysis and basepoint generation standards

### Basepoint File Structure

- **Naming Convention**: Basepoint files must follow the pattern `agent-base-[module-name].md` where `[module-name]` is the actual name of the module folder
- **Location**: Basepoint files must be placed in the mirrored structure within `geist/basepoints/`, maintaining the same relative path as the module in the project
- **Content Structure**: Each basepoint file must include:
  - Module Overview
  - Patterns (Design, Coding, Architectural)
  - Standards (Naming, Coding Style, Structure)
  - Flows (Data, Control, Dependency)
  - Strategies (Implementation, Architectural)
  - Testing (Test Patterns, Strategies, Organization)
  - Related Modules
  - Notes

### Pattern Extraction Standards

- **Comprehensive Analysis**: Extract patterns from all code files, configuration files, test files, and documentation
- **Layer-by-Layer Processing**: Process files at each abstraction layer individually
- **Cross-Layer Patterns**: Also identify patterns that span across abstraction layers
- **Pattern Categories**: Document patterns in these categories:
  - Design patterns (creational, structural, behavioral)
  - Coding patterns (idioms, conventions, best practices)
  - Architectural patterns (layers, components, interactions)

### Abstraction Layer Detection Standards

- **Structure Analysis**: Analyze folder structure to identify potential abstraction layers
- **Common Pattern Recognition**: Detect common architecture patterns:
  - Mobile: `data/`, `domain/`, `presentation/` pattern
  - Web: `backend/`, `frontend/` pattern
  - Generic layered structures
- **User Confirmation**: Always present detected layers to user for confirmation or refinement
- **Documentation**: Document detected layers with their paths, purposes, and boundaries

### Basepoint Documentation Standards

- **Completeness**: Each basepoint must document all relevant patterns, standards, flows, strategies, and testing approaches found in that module
- **Hierarchy Maintenance**: Parent basepoints must aggregate knowledge from child basepoints and maintain references to child modules
- **Consistency**: Use consistent structure and format across all basepoint files
- **Navigation**: Include links to related modules and parent/child relationships

### Headquarter File Standards

- **Product Integration**: Headquarter file must bridge product-level abstraction (from mission.md, roadmap.md, tech-stack.md) with software project-level abstraction (from codebase analysis)
- **Architecture Documentation**: Document overall architecture, abstraction layers, and module relationships
- **Navigation**: Provide navigation by abstraction layer and by module
- **Completeness**: Include all high-level insights, patterns, and architectural decisions
