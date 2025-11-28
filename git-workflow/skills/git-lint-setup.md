---
name: git-lint-setup
description: Use when user wants to set up commit message conventions, configure commit linting, or establish gitmoji standards for AI commits.
requires:
  tools: [git-lint]
  skills: []
---

# Git Commit Conventions

Two-tier system: humans commit freely, AI agents follow strict gitmoji conventions.

## Prerequisites

```bash
command -v git-lint >/dev/null || { echo "Install: gem install git-lint"; exit 1; }
```

## Agent Commit Format

```
🤖 <gitmoji> <message>

Optional body explaining why.
```

**Rules:**
- Start with 🤖
- Use appropriate gitmoji
- One concern per commit
- Present tense, imperative: "Add feature" not "Added"

## Gitmoji Reference

| Emoji | Code | Use |
|-------|------|-----|
| ✨ | `:sparkles:` | New feature |
| 🐛 | `:bug:` | Bug fix |
| 🚑️ | `:ambulance:` | Critical hotfix |
| ♻️ | `:recycle:` | Refactor |
| ✅ | `:white_check_mark:` | Tests |
| 📝 | `:memo:` | Documentation |
| 🔧 | `:wrench:` | Configuration |
| 🔒️ | `:lock:` | Security |
| ⬆️ | `:arrow_up:` | Upgrade deps |
| 🗑️ | `:wastebasket:` | Remove code |
| 🚧 | `:construction:` | Work in progress |

## Configuration

`.git-lint.yml`:
```yaml
analyzers:
  commit_body_leading_line:
    enabled: true
  commit_subject_length:
    enabled: true
    maximum: 72
  commit_subject_prefix:
    enabled: true
    includes:
      - "🤖 ✨"
      - "🤖 🐛"
      - "🤖 ♻️"
      - "🤖 📝"
```

## Pre-commit Hook

`.git/hooks/commit-msg`:
```bash
#!/bin/bash
# Only validate AI commits (start with 🤖)
if head -1 "$1" | grep -q "^🤖"; then
    git-lint --hook commit-msg "$1"
fi
```

Make executable:
```bash
chmod +x .git/hooks/commit-msg
```

## Example Commits

```bash
# Agent commits
🤖 ✨ Add user authentication
🤖 🐛 Fix login timeout on slow connections
🤖 ♻️ Extract validation logic to separate module
🤖 📝 Document API endpoints

# Human commits - any format
Fixed the thing
WIP stuff
```

## Success Criteria

- [ ] `.git-lint.yml` configuration exists
- [ ] Pre-commit hook installed and executable
- [ ] AI commits validated with gitmoji prefix

## Related Skills

- `milestoner-releases` - Uses commit messages for changelogs
