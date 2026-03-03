---
description: Review changes and create atomic commits following Conventional Commits
---

# Commit Changes

Review all staged and modified files. Split changes into atomic commits following Conventional Commits specification.

## Workflow

1. **Analyze changes**: Run `git status` and `git diff --cached`
2. **Group related changes**: Identify logical units
3. **Create commits**: One commit per logical change
4. **Update docs**: If architecture changes, update CLAUDE.md

## Commit Format

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

## Types

- `feat`: New feature (MINOR version bump)
- `fix`: Bug fix (PATCH version bump)
- `docs`: Documentation only
- `style`: Formatting, whitespace (no code change)
- `refactor`: Code change that neither fixes bug nor adds feature
- `perf`: Performance improvement
- `test`: Adding or updating tests
- `build`: Build system or dependencies
- `ci`: CI configuration
- `chore`: Other changes

## Breaking Changes

- Add `!` after type: `feat!:` or `feat(scope)!:`
- Or add footer: `BREAKING CHANGE: description`
- Triggers MAJOR version bump

## Rules

- Subject: imperative mood, lowercase, no period, max 50 chars
- Body: explain WHAT and WHY, not HOW (wrap at 72 chars)
- Footer: reference issues (`Fixes #123`, `Refs #456`)
- Each commit must be atomic and functional

## Grouping Strategy

### ✅ Group together:
- Changes fixing the same bug
- Files for one complete feature
- Refactoring of the same module

### ❌ Split separately:
- Unrelated features
- Bug fix + new feature
- Different modules/layers

## Examples

```
feat(auth): add OAuth2 login support

Implement OAuth2 authentication flow with Google and GitHub providers.
Includes token refresh mechanism and session management.

Refs #234

---

fix(api): handle null user response

Add null check before accessing user.email to prevent NPE.

Fixes #456

---

docs(readme): update installation instructions

---

refactor(utils): extract date formatting helpers

Move date formatting logic from components to utils/dateHelper.
No functional changes.

---

perf(parser): optimize JSON parsing

Replace recursive parser with iterative approach.
Reduces parse time by 40% for large payloads.

---

feat(api)!: remove deprecated v1 endpoints

BREAKING CHANGE: All /api/v1/* endpoints removed.
Migrate to /api/v2/* which provides enhanced functionality.
```

Now create the commits.
