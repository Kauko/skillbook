---
name: milestoner-releases
description: Use when user wants to create a release, generate changelog, bump version, or tag a new version for deployment.
requires:
  tools: [milestoner]
  skills: [git-lint-setup]
---

# Release Management with Milestoner

## Prerequisites

```bash
command -v milestoner >/dev/null || { echo "Install: gem install milestoner"; exit 1; }
```

## Configuration

`.milestoner.yml`:
```yaml
version: patch  # Default bump: major|minor|patch
format: markdown
prefix: v
changelog:
  path: CHANGELOG.md
  template: |
    ## [%<version>s] - %<date>s
    %<commits>s
commit_categories:
  - prefix: feat
    label: Added
  - prefix: fix
    label: Fixed
  - prefix: docs
    label: Documentation
  - prefix: refactor
    label: Changed
```

## Common Workflows

### Preview Release

```bash
milestoner status              # Show commits since last tag
milestoner changelog --preview  # Preview changelog
```

### Create Release

```bash
# Bump version and generate changelog
milestoner release patch       # 1.0.0 -> 1.0.1
milestoner release minor       # 1.0.0 -> 1.1.0
milestoner release major       # 1.0.0 -> 2.0.0

# Or specific version
milestoner release --version 2.1.0
```

### Changelog Only

```bash
milestoner changelog           # Generate/update CHANGELOG.md
```

## Commit Message Format

Milestoner parses prefixed commits:
```
feat: Add user authentication
fix: Resolve login timeout
docs: Update API documentation
refactor: Simplify database queries
```

## Release Workflow

```bash
# 1. Check what's changed
milestoner status

# 2. Preview changelog
milestoner changelog --preview

# 3. Create release (updates CHANGELOG, creates tag)
milestoner release minor

# 4. Push with tags
git push && git push --tags
```

## CI Integration

```yaml
- name: Create Release
  run: |
    gem install milestoner
    milestoner release ${{ inputs.bump_type }}
    git push && git push --tags
```

## Reference Documentation

- `references/configuration.md` - All config options
- `references/commands.md` - CLI reference

## Success Criteria

- [ ] `.milestoner.yml` configuration exists
- [ ] `milestoner changelog --preview` shows expected changes
- [ ] Git tag created with correct version
- [ ] CHANGELOG.md updated

## Related Skills

- `git-lint-setup` - Better commit messages for changelog
