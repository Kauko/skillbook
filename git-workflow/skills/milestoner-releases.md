# Release Management with Milestoner

## Description

Use when preparing releases, generating changelogs, or managing version tags. Milestoner automates changelog generation from git commits, creates release notes, and manages semantic versioning workflows.

This skill provides workflow guidance. For comprehensive technical details, see:
- **[Configuration Reference](milestoner-releases/references/configuration.md)** - All `.milestoner.yml` options with examples
- **[Commands Reference](milestoner-releases/references/commands.md)** - Complete CLI command documentation

## When to Use

- User mentions "release", "version", "changelog", or "tag"
- Preparing to publish a new version
- Need to generate or update CHANGELOG.md
- Creating release notes for stakeholders
- User asks about version history or what's changed
- Completing a milestone or sprint with deployable changes

## Proactive Suggestions

Suggest using this skill when:
- Multiple features/fixes have accumulated since last release
- User completes a significant feature branch
- User mentions deployment or publishing
- CHANGELOG.md is outdated or missing

## Setup Process

### 1. Check for Milestoner

```bash
which milestoner
```

If not found, guide installation:

```bash
gem install milestoner
```

For project-specific installation, add to `Gemfile`:
```ruby
source "https://rubygems.org"
gem "milestoner"
gem "git-lint"  # Recommended for better commit messages
```

Then: `bundle install`

### 2. Create Milestoner Configuration

Create `.milestoner.yml` in repository root. Here's a workflow-focused configuration:

```yaml
# Milestoner Configuration for Workflow
project:
  name: "myproject"
  owner: "myorg"
  description: "Project description"

build:
  format: "stream"      # Console output by default
  output: "tmp/milestones"

commit:
  # Categorize commits by emoji prefix
  categories:
    added:
      emoji: "✨"
      label: "Features"
    fixed:
      emoji: "🐛"
      label: "Bug Fixes"
    updated:
      emoji: "⬆️"
      label: "Updates"
    security:
      emoji: "🔒"
      label: "Security"
    performance:
      emoji: "⚡"
      label: "Performance"
    refactored:
      emoji: "♻️"
      label: "Refactoring"
    removed:
      emoji: "🗑️"
      label: "Removed"

  format: "markdown"
  uri: "https://github.com/%<project_owner>s/%<project_name>s/commit/%<commit_sha>s"

tag:
  subject: "Release %<project_version>s"

tracker:
  uri: "https://github.com/%<project_owner>s/%<project_name>s/issues/%<issue_id>s"
```

For comprehensive configuration options including syndication feeds, custom templates, avatar configuration, and advanced categorization, see [Configuration Reference](milestoner-releases/references/configuration.md).

### 3. Initialize Changelog

If CHANGELOG.md doesn't exist:

```bash
milestoner --init
```

This creates a basic CHANGELOG.md structure:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
```

## Release Workflow

### 1. Review Changes Since Last Release

```bash
# View commits since last tag
git log $(git describe --tags --abbrev=0)..HEAD --oneline

# Or use milestoner to preview
milestoner build --format stream
```

**Using Git Trailers for Automatic Versioning**:

Milestoner can automatically determine version bumps from Git trailers in commit messages:

```bash
# Commit with milestone trailer
git commit -m "Add OAuth2 authentication

Implements OAuth2 support for Google and GitHub providers.

Milestone: minor"
```

Trailer values:
- `Milestone: patch` - Bug fixes (0.1.0 -> 0.1.1)
- `Milestone: minor` - New features (0.1.0 -> 0.2.0)
- `Milestone: major` - Breaking changes (0.1.0 -> 1.0.0)

The highest milestone across all commits wins. See [Commands Reference](milestoner-releases/references/commands.md) for details.

### 2. Determine Version Bump

Follow semantic versioning (MAJOR.MINOR.PATCH):

- **MAJOR**: Breaking changes, incompatible API changes
- **MINOR**: New features, backward-compatible additions
- **PATCH**: Bug fixes, backward-compatible fixes

### 3. Generate Changelog

```bash
# For patch release (0.1.0 -> 0.1.1)
milestoner --publish patch

# For minor release (0.1.0 -> 0.2.0)
milestoner --publish minor

# For major release (0.1.0 -> 1.0.0)
milestoner --publish major

# For specific version
milestoner --publish 2.5.0
```

This will:
- Calculate version from Git trailers or use specified version
- Generate release notes from commits since last tag
- Create annotated Git tag (with GPG signing if configured)
- Update configured outputs

For complete command reference including build options, cache management, configuration commands, and Git trailer usage, see [Commands Reference](milestoner-releases/references/commands.md).

### 4. Create Detailed Release Notes

Create `vault/releases/vX.Y.Z.md` with comprehensive details:

```markdown
# Release X.Y.Z

Released: YYYY-MM-DD

## Summary

[2-3 sentence overview of this release]

## Highlights

### Feature: [Major Feature Name]

[Detailed explanation with examples]

### Improvement: [Significant Improvement]

[What changed and why it matters]

## Breaking Changes

[If any, with migration guide]

## Full Changelog

[Copy from CHANGELOG.md or reference it]

## Dependencies

- Updated X to v1.2.3
- Removed deprecated Y

## Known Issues

[If any]

## Contributors

Thanks to [list contributors]

## Arc42 Updates

[Reference to architecture documentation updates in section 11]
```

### 5. Review and Edit

Milestoner generates raw changelog from commits. Review and enhance:

- Add context and explanations
- Group related changes
- Highlight breaking changes
- Add migration guides
- Fix formatting and grammar
- Add links to issues/PRs

### 6. Commit and Tag

```bash
# If using git-lint, commit as agent
git add CHANGELOG.md vault/releases/vX.Y.Z.md
git commit -m "🤖 🔖 Release version X.Y.Z"

# Create annotated tag
git tag -a vX.Y.Z -m "Release version X.Y.Z"

# Push with tags
git push origin main --tags
```

## Integration with git-lint

Milestoner works best with structured commits from git-lint:

```bash
# Well-formatted commits
🤖 ✨ (auth) Add OAuth2 support
🤖 🐛 (orders) Fix duplicate creation
🤖 📝 Update API documentation
🤖 ♻️ (database) Extract connection pool

# Produce organized changelog
## Features
* (auth) Add OAuth2 support

## Bug Fixes
* (orders) Fix duplicate creation

## Documentation
* Update API documentation

## Refactoring
* (database) Extract connection pool
```

Poor commit messages produce poor changelogs. Structured commits are essential.

## Release Checklist Template

Use this checklist for each release:

```markdown
## Pre-Release Checklist

- [ ] All tests passing
- [ ] No known critical bugs
- [ ] Dependencies up to date
- [ ] Security vulnerabilities addressed
- [ ] Documentation updated
- [ ] Migration guide written (if breaking changes)
- [ ] Performance tested
- [ ] Backward compatibility verified

## Release Process

- [ ] Run `milestoner --preview` to review changes
- [ ] Determine version bump (major/minor/patch)
- [ ] Generate changelog: `milestoner --publish X.Y.Z`
- [ ] Review and edit CHANGELOG.md
- [ ] Create detailed release notes in vault/releases/
- [ ] Update arc42 section 11 (Quality Requirements)
- [ ] Commit changes with 🤖 🔖 prefix
- [ ] Create annotated git tag
- [ ] Push to remote with tags

## Post-Release

- [ ] Verify tag on remote
- [ ] Create GitHub/GitLab release
- [ ] Announce in team channels
- [ ] Update deployment tracking
- [ ] Monitor for issues
- [ ] Update project roadmap
```

## Advanced Usage

### Custom Grouping

Edit `.milestoner.yml` to customize commit grouping:

```yaml
changelog:
  groups:
    - name: "Breaking Changes"
      emoji: "💥"
      prefixes: ["💥", ":boom:", "BREAKING"]
    - name: "API Changes"
      emoji: "🔌"
      prefixes: ["🔌", ":electric_plug:"]
```

### Multi-Project Releases

For monorepos with multiple packages:

```yaml
projects:
  - name: "frontend"
    path: "packages/frontend"
    changelog: "packages/frontend/CHANGELOG.md"
  - name: "backend"
    path: "packages/backend"
    changelog: "packages/backend/CHANGELOG.md"
```

### Pre-Release Versions

For alpha, beta, or RC releases:

```bash
milestoner --publish 2.0.0-alpha.1
milestoner --publish 2.0.0-beta.2
milestoner --publish 2.0.0-rc.1
```

### Hotfix Releases

For urgent production fixes:

```bash
# On hotfix branch
git checkout -b hotfix/critical-bug

# Make fix
git commit -m "🤖 🚑️ Fix critical security vulnerability"

# Generate hotfix release
milestoner --publish patch

# Merge back
git checkout main
git merge hotfix/critical-bug
git push --tags
```

## Arc42 Integration

When releasing, update arc42 documentation section 11 (Quality Requirements):

```markdown
## 11.2 Quality Scenarios

### Release X.Y.Z Quality Metrics

- Performance: [metrics and improvements]
- Reliability: [uptime, error rates]
- Security: [vulnerabilities fixed]
- Maintainability: [code quality metrics]
- Usability: [user feedback, UX improvements]

## 11.3 Quality Tree

[Update quality tree with new requirements from this release]
```

Link release notes to arc42:

```markdown
See [Release X.Y.Z](/vault/releases/vX.Y.Z.md) for detailed changes
affecting these quality scenarios.
```

## Troubleshooting

### No tags found

```bash
# Create initial tag
git tag -a v0.1.0 -m "Initial release"
git push --tags
```

### Commits not grouped correctly

- Check `.milestoner.yml` group prefixes match commit format
- Verify emoji codes are correct
- Test regex patterns: `echo "🤖 ✨ Feature" | grep -E "^.*✨"`

### Changelog formatting issues

- Review `.milestoner.yml` format strings
- Check for special characters needing escaping
- Validate YAML syntax: `ruby -ryaml -e "YAML.load_file('.milestoner.yml')"`

### Git tag conflicts

```bash
# List tags
git tag -l

# Delete local tag
git tag -d vX.Y.Z

# Delete remote tag
git push origin :refs/tags/vX.Y.Z
```

## Best Practices

### 1. Regular Releases

Release often, ideally:
- Patch releases: Weekly or as needed for bugs
- Minor releases: Monthly or per feature completion
- Major releases: Quarterly or for breaking changes

### 2. Semantic Versioning Discipline

Be strict about version semantics:
- Breaking change? MAJOR bump required
- New feature? MINOR bump minimum
- Bug fix only? PATCH bump

### 3. Comprehensive Release Notes

Good release notes include:
- High-level summary for stakeholders
- Technical changelog for developers
- Migration guide for breaking changes
- Known issues and workarounds
- Links to relevant documentation

### 4. Changelog Curation

Don't just dump commits:
- Group related changes
- Explain context and rationale
- Remove noise (typo fixes, minor formatting)
- Highlight important changes
- Add examples where helpful

### 5. Coordinate with git-lint

Structured commits from git-lint make Milestoner shine:
- Consistent emoji prefixes
- Clear scopes
- Descriptive messages
- Single concern per commit

## Examples

### Patch Release

```bash
# Review changes
milestoner --preview

# Generate changelog
milestoner --publish patch

# Create release note
cat > vault/releases/v1.2.3.md << 'EOF'
# Release 1.2.3

Released: 2024-01-15

## Summary

Maintenance release with bug fixes and minor improvements.

## Bug Fixes

- Fixed memory leak in connection pool
- Resolved race condition in order processing
- Corrected date formatting in reports

## Improvements

- Updated dependencies to latest stable versions
- Improved error messages for validation failures

## Full Changelog

See [CHANGELOG.md](../../CHANGELOG.md#123---2024-01-15)
EOF

# Commit and tag
git add CHANGELOG.md vault/releases/v1.2.3.md
git commit -m "🤖 🔖 Release version 1.2.3"
git tag -a v1.2.3 -m "Release version 1.2.3"
git push origin main --tags
```

### Minor Release with Features

```bash
# Generate changelog
milestoner --publish minor

# Create detailed release note
cat > vault/releases/v1.3.0.md << 'EOF'
# Release 1.3.0

Released: 2024-02-01

## Summary

Feature release adding OAuth2 authentication and bulk operations.

## New Features

### OAuth2 Authentication

Users can now authenticate using Google and GitHub accounts:

```typescript
const auth = new OAuth2Provider({
  provider: 'google',
  clientId: process.env.GOOGLE_CLIENT_ID,
});
```

### Bulk Order Processing

Process multiple orders in a single request:

```typescript
await orderService.bulkCreate(orders);
```

## Improvements

- Enhanced error handling with detailed context
- Improved API response times by 30%
- Better TypeScript type inference

## Bug Fixes

- Fixed session timeout handling
- Resolved CSV export formatting issues

## Migration Guide

No breaking changes. OAuth2 is opt-in feature.

## Full Changelog

See [CHANGELOG.md](../../CHANGELOG.md#130---2024-02-01)
EOF

# Commit and tag
git add CHANGELOG.md vault/releases/v1.3.0.md
git commit -m "🤖 🔖 Release version 1.3.0"
git tag -a v1.3.0 -m "Release version 1.3.0"
git push origin main --tags
```

### Major Release with Breaking Changes

```bash
# Generate changelog
milestoner --publish major

# Create comprehensive release note
cat > vault/releases/v2.0.0.md << 'EOF'
# Release 2.0.0

Released: 2024-03-01

## Summary

Major release with API redesign and significant architectural improvements.

## Breaking Changes

### API Restructure

Previous endpoint structure:
```
POST /api/v1/order/create
GET /api/v1/order/list
```

New RESTful structure:
```
POST /api/v2/orders
GET /api/v2/orders
```

### Configuration Changes

Environment variables renamed:
- `DB_HOST` → `DATABASE_HOST`
- `DB_PORT` → `DATABASE_PORT`

### Removed Deprecated Features

- Removed legacy XML API (deprecated in v1.5.0)
- Removed compatibility mode for v0.x clients

## Migration Guide

### Step 1: Update API Endpoints

Replace all API calls to use `/api/v2/` prefix:

```diff
- fetch('/api/v1/order/create', {...})
+ fetch('/api/v2/orders', {...})
```

### Step 2: Update Environment Variables

Rename in your `.env` files:

```diff
- DB_HOST=localhost
- DB_PORT=5432
+ DATABASE_HOST=localhost
+ DATABASE_PORT=5432
```

### Step 3: Remove XML API Usage

Migrate XML API calls to REST API:

```typescript
// Before
const result = await client.postXML(data);

// After
const result = await client.post('/api/v2/orders', data);
```

## New Features

- GraphQL API endpoint
- Real-time subscriptions via WebSocket
- Advanced filtering and pagination
- Audit logging for all operations

## Improvements

- 50% faster query performance
- Reduced memory usage by 40%
- Better error messages with correlation IDs

## Full Changelog

See [CHANGELOG.md](../../CHANGELOG.md#200---2024-03-01)

## Arc42 Updates

Updated sections:
- Section 6: Runtime View (new API flows)
- Section 8: Concepts (authentication redesign)
- Section 11: Quality Requirements (performance metrics)

## Support

For migration assistance, see [Migration Guide](../migration/v1-to-v2.md)
or contact support@example.com.
EOF

# Commit and tag
git add CHANGELOG.md vault/releases/v2.0.0.md
git commit -m "🤖 🔖 Release version 2.0.0"
git tag -a v2.0.0 -m "Release version 2.0.0"
git push origin main --tags
```

## Detailed References

See the `references/` directory for comprehensive documentation:

- **[Configuration Reference](milestoner-releases/references/configuration.md)** - Complete `.milestoner.yml` options with examples
- **[Commands Reference](milestoner-releases/references/commands.md)** - All CLI commands, options, and workflows

## External Resources

- [Milestoner Documentation](https://alchemists.io/projects/milestoner)
- [Semantic Versioning](https://semver.org)
- [Keep a Changelog](https://keepachangelog.com)
- [git-lint](https://alchemists.io/projects/git-lint)
- [Conventional Commits](https://www.conventionalcommits.org)
