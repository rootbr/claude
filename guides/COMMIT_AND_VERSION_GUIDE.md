# Conventional Commits and Semantic Versioning Guide

This guide covers the rules and best practices for commit messages and version numbering based on the official specifications.

## Conventional Commits Specification

### Commit Message Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Commit Types

#### Primary Types
- **`feat`**: A new feature (correlates with MINOR in SemVer)
- **`fix`**: A bug fix (correlates with PATCH in SemVer)

#### Additional Types
- **`build`**: Changes affecting build system or external dependencies
- **`chore`**: Other changes that don't modify src or test files
- **`ci`**: Changes to CI configuration files and scripts
- **`docs`**: Documentation only changes
- **`style`**: Changes that don't affect code meaning (formatting, whitespace)
- **`refactor`**: Code changes that neither fix bugs nor add features
- **`perf`**: Code changes that improve performance
- **`test`**: Adding missing tests or correcting existing tests

### Breaking Changes

Two methods to indicate breaking changes:
1. Add `!` after type/scope: `feat(api)!: remove deprecated endpoint`
2. Include `BREAKING CHANGE:` footer with description

### Scope Usage

- Optional but recommended for clarity
- Provides additional contextual information
- Examples: `feat(parser):`, `fix(database):`, `docs(api):`
- Should be a noun describing the section of codebase

### Footer Format

Footers follow the format: `<token>: <value>` or `<token> #<value>`

Common footers:
- `BREAKING CHANGE: <description>`
- `Fixes #123`
- `Refs #456`
- `Reviewed-by: <name>`
- `Co-authored-by: <name>`

### Commit Message Examples

#### Simple Commits
```
feat: add email notifications
fix: resolve memory leak in data processing
docs: update API documentation
style: format code according to standards
test: add unit tests for user validation
```

#### With Scope
```
feat(auth): implement OAuth2 login
fix(database): prevent connection timeout
perf(parser): improve JSON parsing speed
docs(api): update endpoint documentation
ci(build): add automated testing pipeline
```

#### Breaking Changes
```
feat(api)!: remove deprecated /v1/users endpoint

BREAKING CHANGE: The /v1/users endpoint has been removed. Use /v2/users instead.
```

#### Detailed Commit
```
fix: prevent racing of requests

Introduce request ID and reference to latest request. Dismiss
incoming responses other than from latest request.

Remove timeouts which were used to mitigate the racing issue but are
obsolete now.

Reviewed-by: Z
Refs: #123
```

### Commit Message Rules

- **Description must be lowercase** and not end with a period
- **Use imperative mood** ("add feature" not "added feature")
- **Keep first line under 50 characters** when possible
- **Body explains what and why**, not how
- **Reference issues** and pull requests in footers
- **Separate body and footers** with blank lines

## Semantic Versioning 2.0.0

### Version Format

Standard format: `MAJOR.MINOR.PATCH` (e.g., `1.4.2`)

### When to Increment

#### MAJOR Version (X.Y.Z → (X+1).0.0)
Increment when making **incompatible API changes**:
- Breaking existing functionality
- Removing features
- Changing method signatures
- Modifying expected behavior

Examples:
- `1.5.3 → 2.0.0` (removed deprecated API)
- `3.2.1 → 4.0.0` (changed function parameters)

#### MINOR Version (X.Y.Z → X.(Y+1).0)
Increment when adding **backward-compatible functionality**:
- New features
- New methods/functions
- Enhancements that don't break existing code
- Deprecating functionality (without removing)

Examples:
- `1.3.2 → 1.4.0` (added new feature)
- `2.1.5 → 2.2.0` (added new API endpoint)

#### PATCH Version (X.Y.Z → X.Y.(Z+1))
Increment when making **backward-compatible bug fixes**:
- Bug fixes
- Security patches
- Performance improvements without API changes

Examples:
- `1.2.1 → 1.2.2` (fixed memory leak)
- `2.0.0 → 2.0.1` (security patch)

### Pre-release Versions

Format: `X.Y.Z-<pre-release>`

Examples:
- `1.0.0-alpha`
- `1.0.0-beta.1`
- `1.0.0-rc.1`
- `2.1.0-alpha.beta.1`

Rules:
- Identifiers separated by dots
- May contain alphanumeric characters and hyphens
- Pre-release versions have lower precedence than normal versions

### Build Metadata

Format: `X.Y.Z+<build>`

Examples:
- `1.0.0+20130313144700`
- `1.0.0-beta+exp.sha.5114f85`
- `1.2.3+build.123`

Rules:
- Should be ignored when determining version precedence
- Used for build identification only

### Version Precedence

Precedence rules (from lowest to highest):
1. Compare MAJOR, MINOR, PATCH numerically
2. Pre-release versions have lower precedence than normal versions
3. Compare pre-release identifiers from left to right:
   - Numeric identifiers: compared numerically
   - Alphanumeric identifiers: compared lexically (ASCII sort)
   - Numeric identifiers always have lower precedence than non-numeric

Example precedence order:
```
1.0.0-alpha < 1.0.0-alpha.1 < 1.0.0-alpha.beta < 1.0.0-beta
< 1.0.0-beta.2 < 1.0.0-beta.11 < 1.0.0-rc.1 < 1.0.0
< 2.0.0-alpha < 2.0.0
```

### Typical Development Cycle

```
0.1.0 → 0.2.0 → 0.2.1 → 1.0.0-alpha → 1.0.0-beta → 1.0.0
→ 1.0.1 → 1.1.0 → 1.1.1 → 2.0.0
```

## Best Practices

### For Conventional Commits

1. **Be consistent** with type usage across the project
2. **Use descriptive scopes** to provide context
3. **Write clear descriptions** that explain the change
4. **Include body** for complex changes to explain reasoning
5. **Reference issues** in footers when applicable
6. **Use tools** like commitizen or conventional-changelog for automation

### For Semantic Versioning

1. **Define your public API clearly** and document it
2. **Think carefully** about backward compatibility
3. **Use 0.y.z** for initial development (anything may change)
4. **Release 1.0.0** when you have a stable public API
5. **Never modify released versions** - always create new ones
6. **Consider automation tools** for version management
7. **Communicate breaking changes** clearly in release notes

## Integration Example

When using both specifications together:

```bash
# Feature commit that adds new functionality
git commit -m "feat(auth): add two-factor authentication support"
# Version bump: 1.2.3 → 1.3.0 (MINOR)

# Bug fix commit
git commit -m "fix(database): resolve connection pool exhaustion"
# Version bump: 1.3.0 → 1.3.1 (PATCH)

# Breaking change commit
git commit -m "feat(api)!: remove deprecated v1 endpoints

BREAKING CHANGE: All v1 API endpoints have been removed. 
Migrate to v2 endpoints which provide enhanced functionality."
# Version bump: 1.3.1 → 2.0.0 (MAJOR)
```

## Tools and Automation

### Recommended Tools
- **commitizen**: Interactive commit message builder
- **conventional-changelog**: Generate changelogs from commits
- **semantic-release**: Automate version management and publishing
- **commitlint**: Lint commit messages

### Example Configuration

**.commitlintrc.json**:
```json
{
  "extends": ["@commitlint/config-conventional"],
  "rules": {
    "type-enum": [2, "always", [
      "feat", "fix", "docs", "style", "refactor", 
      "perf", "test", "build", "ci", "chore"
    ]]
  }
}
```

This guide ensures consistent commit messages and proper version management following industry standards.