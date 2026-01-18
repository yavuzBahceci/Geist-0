## General development conventions

- **Consistent Project Structure**: Organize files and directories in a predictable, logical structure that team members can navigate easily
- **Clear Documentation**: Maintain up-to-date README files with setup instructions, architecture overview, and contribution guidelines
- **Version Control Best Practices**: Use clear commit messages, feature branches, and meaningful pull/merge requests with descriptions
- **Environment Configuration**: Use environment variables for configuration; never commit secrets or API keys to version control
- **Dependency Management**: Keep dependencies up-to-date and minimal; document why major dependencies are used
- **Code Review Process**: Establish a consistent code review process with clear expectations for reviewers and authors
- **Testing Requirements**: Define what level of testing is required before merging (unit tests, integration tests, etc.)
- **Feature Flags**: Use feature flags for incomplete features rather than long-lived feature branches
- **Changelog Maintenance**: Keep a changelog or release notes to track significant changes and improvements

## Agent OS Output Conventions

**CRITICAL: All outputs MUST go to `agent-os/` directory**

- **All generated files** must be written under `agent-os/` - never to project root or other directories
- **Specs**: `agent-os/specs/[date]-[name]/`
- **Product docs**: `agent-os/product/`
- **Basepoints**: `agent-os/basepoints/`
- **Config**: `agent-os/config/`
- **Reports/outputs**: `agent-os/output/`
- **Plans**: `agent-os/output/plans/` (NOT `plans/` at root)
- **Caches**: `agent-os/output/cache/` or `agent-os/specs/[spec]/implementation/cache/`

This keeps the project root clean and all AI-generated content contained in one predictable location.
