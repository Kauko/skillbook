# Git Commit Conventions with git-lint

## Description

Use when setting up git commit conventions in a repository. Implements a two-tier commit system: humans commit freely without restrictions, while AI agents follow strict gitmoji-based conventions validated by git-lint.

## When to Use

- User asks to set up commit conventions or standards
- User mentions git-lint, gitmoji, or commit message formatting
- Starting a new project that needs commit guidelines
- User wants to enforce commit quality for AI agents only

## Two-Tier Commit System

### HUMANS: Freedom First

- No rules, no validation, no friction
- Commit in any format, any style
- git-lint hook automatically skips human commits
- Full creative freedom for human developers

### AGENTS: Structured Precision

AI agents MUST follow this format:
```
🤖 <gitmoji> <message>

Optional body with more details.
```

Rules for agents:
- Every commit starts with 🤖 emoji
- Followed by appropriate gitmoji from table below
- Single concern per commit (no mixing features + docs)
- Scope required for code changes: `🤖 ✨ (orders) Add bulk processing`
- Present tense, imperative mood: "Add feature" not "Added feature"
- Clear, descriptive messages explaining WHY, not just WHAT

## Gitmoji Reference for Agents

| Category | Emoji | Code | Use |
|----------|-------|------|-----|
| **Feature** | ✨ | `:sparkles:` | New features or capabilities |
| **Bug Fix** | 🐛 | `:bug:` | Bug fixes (non-critical) |
| **Hotfix** | 🚑️ | `:ambulance:` | Critical production fixes |
| **Refactor** | ♻️ | `:recycle:` | Code refactoring without behavior change |
| **Tests** | ✅ | `:white_check_mark:` | Adding or updating tests |
| **Architecture** | 🏗️ | `:building_construction:` | Architectural changes |
| **Security** | 🔒️ | `:lock:` | Security fixes or improvements |
| **Docs** | 📝 | `:memo:` | General documentation |
| **ADR** | 🧭 | `:compass:` | Architecture Decision Records |
| **Formal Spec** | 🔬 | `:microscope:` | TLA+ or Recife specifications |
| **BDD Spec** | 🥒 | `:cucumber:` | Gherkin/Cucumber scenarios |
| **Types/Schemas** | 🏷️ | `:label:` | Type definitions, Malli schemas |
| **Validation** | 🦺 | `:safety_vest:` | Guardrails contracts, validation |
| **Infrastructure** | 🧱 | `:bricks:` | Infrastructure as Code changes |
| **Config** | 🔧 | `:wrench:` | Configuration changes |
| **Release** | 🔖 | `:bookmark:` | Version tags and releases |
| **Performance** | ⚡️ | `:zap:` | Performance improvements |
| **CI/CD** | 👷 | `:construction_worker:` | CI/CD pipeline changes |
| **Dependencies** | ⬆️ | `:arrow_up:` | Dependency upgrades |
| **Revert** | ⏪️ | `:rewind:` | Reverting changes |

## Setup Process

### 1. Check for git-lint

```bash
which git-lint
```

If not found, guide installation:

```bash
gem install git-lint
```

For project-specific installation, create `Gemfile`:
```ruby
source "https://rubygems.org"
gem "git-lint"
```

Then: `bundle install`

### 2. Create git-lint Configuration

Create `.git-lint.yml` in repository root:

```yaml
commits:
  enabled: true
  subject_length: 72
  subject_prefix: "\\A🤖 "  # Require 🤖 prefix for agent commits
  subject_capitalized: true

branches:
  enabled: true
  name: "\\A[a-z]+[a-z0-9-]*\\Z"

issues:
  enabled: false  # Customize per project needs
```

### 3. Install Custom commit-msg Hook

CRITICAL: The hook MUST only validate commits starting with 🤖:

```bash
#!/bin/bash
# .git/hooks/commit-msg
# Only validate agent commits (🤖 prefix)

commit_message=$(head -1 "$1")

if [[ "$commit_message" == 🤖* ]]; then
  git-lint --commit-msg "$1"
  exit $?
fi

# Human commits always pass
exit 0
```

Make executable:
```bash
chmod +x .git/hooks/commit-msg
```

### 4. Document Conventions

Create `vault/decisions/0000-commit-conventions.md`:

```markdown
# ADR 0000: Commit Conventions

## Status

Accepted

## Context

Need clear commit conventions that:
- Allow human flexibility and creativity
- Enforce structure for AI agent commits
- Enable automated changelog generation
- Support semantic release workflows

## Decision

Implement two-tier commit system:

### Tier 1: Human Commits
- No restrictions or validations
- Any format acceptable
- Maximum flexibility for developers

### Tier 2: Agent Commits
- Must start with 🤖 emoji
- Follow gitmoji conventions
- Validated by git-lint
- Format: `🤖 <gitmoji> <message>`

## Consequences

### Positive
- Zero friction for human developers
- Structured, parseable agent commits
- Better changelog generation
- Clear distinction between human/agent work

### Negative
- Agents must learn gitmoji taxonomy
- Hook maintenance required
- Two different commit styles in history

## References

- [git-lint](https://alchemists.io/projects/git-lint)
- [gitmoji](https://gitmoji.dev)
- [Conventional Commits](https://www.conventionalcommits.org)
```

### 5. Guide Agent Usage

When making commits as an agent:

1. Analyze the change type
2. Select appropriate gitmoji from reference table
3. Write clear, descriptive message
4. Include scope for code changes
5. Commit using the format

Example workflow:
```bash
# Feature addition
git commit -m "🤖 ✨ (auth) Add OAuth2 provider support

Implements Google and GitHub OAuth2 authentication flows
with PKCE support for enhanced security."

# Bug fix
git commit -m "🤖 🐛 (orders) Fix duplicate order creation

Prevent race condition when users double-click submit button."

# Documentation
git commit -m "🤖 📝 Update API documentation for v2 endpoints"

# Refactoring
git commit -m "🤖 ♻️ (database) Extract connection pooling logic"
```

## Using gitmoji-cli (Optional)

For interactive emoji selection:

```bash
npm install -g gitmoji-cli
gitmoji -c
```

This provides an interactive prompt for selecting appropriate gitmoji.

## Validation Testing

Test the hook works correctly:

```bash
# Human commit (should pass)
git commit -m "wip stuff"

# Agent commit without proper format (should fail)
git commit -m "🤖 add feature"

# Agent commit with proper format (should pass)
git commit -m "🤖 ✨ (core) Add feature X"
```

## Integration with Milestoner

Well-formatted commits enable better changelog generation:
- Gitmoji provides visual categorization
- Scopes group related changes
- Clear messages become readable changelog entries

When using Milestoner for releases, it can parse these structured commits to generate organized, scannable changelogs.

## Troubleshooting

### Hook not running
- Check hook is executable: `ls -la .git/hooks/commit-msg`
- Verify shebang is correct: `#!/bin/bash`
- Test hook directly: `.git/hooks/commit-msg .git/COMMIT_EDITMSG`

### git-lint errors
- Check `.git-lint.yml` syntax is valid YAML
- Verify regex patterns are properly escaped
- Run `git-lint --help` for configuration options

### Emoji rendering
- Ensure terminal supports UTF-8
- Check font includes emoji support
- Test with: `echo "🤖 ✨ 🐛"`

## References

- [git-lint Documentation](https://alchemists.io/projects/git-lint)
- [gitmoji Guide](https://gitmoji.dev)
- [GitHub Emoji Cheat Sheet](https://github.com/ikatyang/emoji-cheat-sheet)
- [Conventional Commits](https://www.conventionalcommits.org)
