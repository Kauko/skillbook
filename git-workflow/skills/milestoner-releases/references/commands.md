# Milestoner CLI Commands Reference

Complete reference for all Milestoner command-line interface commands and options.

## Installation

### Standard Installation

```bash
gem install milestoner
```

### Secure Installation with Verification

```bash
# Add Alchemists gem certificate
gem cert --add <(curl --compressed --location https://alchemists.io/gems.pem)

# Install with high security policy
gem install milestoner --trust-policy HighSecurity
```

### Project Installation

Add to `Gemfile`:

```ruby
source "https://rubygems.org"

gem "milestoner"
```

Then run:

```bash
bundle install
```

## Command Overview

Milestoner provides four main commands:

1. `milestoner build` - Generate release notes
2. `milestoner config` - Manage configuration
3. `milestoner cache` - Manage user information
4. `milestoner --publish` - Publish new milestone

## Build Command

Generate release notes in various formats from Git history.

### Basic Usage

```bash
# Build latest release (default: stream format to console)
milestoner build

# Build with specific format
milestoner build --format markdown
milestoner build --format ascii_doc
milestoner build --format web
milestoner build --format feed
```

### Options

**--format FORMAT**

Output format for release notes.

Values: `ascii_doc`, `feed`, `markdown`, `stream`, `web`

```bash
# Generate Markdown
milestoner build --format markdown

# Generate HTML web page
milestoner build --format web

# Generate Atom feed
milestoner build --format feed

# Output to console (default)
milestoner build --format stream
```

**--max NUMBER**

Maximum number of Git tags to build.

Default: `1` (latest release only)

```bash
# Build latest release only
milestoner build --max 1

# Build last 5 releases
milestoner build --max 5

# Build all releases
milestoner build --max 999999
```

**--version VERSION**

Override automatic version calculation.

```bash
# Force specific version
milestoner build --version 2.0.0

# Use with custom version string
milestoner build --version 1.0.0-beta.1
```

**--help**

Display help documentation.

```bash
milestoner build --help
```

### Output Location

Output files are written to directory specified in configuration:

```yaml
build:
  output: "tmp/milestones"  # Default
```

Generated files:
- `{basename}.{ext}` - Release notes in chosen format
- `index.{ext}` - Index page (if enabled)
- `manifest.json` - Metadata manifest (if enabled)

### Examples

```bash
# Generate Markdown for latest release
milestoner build --format markdown

# Generate last 10 releases as web pages
milestoner build --format web --max 10

# Build specific version
milestoner build --version 1.5.0 --format markdown

# Generate Atom feed of all releases
milestoner build --format feed --max 999999
```

## Publish Command

Publish a new milestone by creating a Git tag and generating release notes.

### Basic Usage

```bash
# Publish with automatic version bump
milestoner --publish patch
milestoner --publish minor
milestoner --publish major

# Publish with specific version
milestoner --publish 2.5.0
```

### Semantic Versioning

**patch** - Bug fixes, backward-compatible (0.1.0 -> 0.1.1)

Use for:
- Bug fixes
- Security patches
- Documentation updates
- Minor improvements

```bash
milestoner --publish patch
```

**minor** - New features, backward-compatible (0.1.0 -> 0.2.0)

Use for:
- New features
- New APIs
- Enhancements
- Deprecations

```bash
milestoner --publish minor
```

**major** - Breaking changes (0.1.0 -> 1.0.0)

Use for:
- Breaking API changes
- Removing deprecated features
- Incompatible changes
- Major redesigns

```bash
milestoner --publish major
```

**Specific Version** - Explicit version number

```bash
milestoner --publish 2.5.0
milestoner --publish 1.0.0-alpha.1
milestoner --publish 3.0.0-beta.2
milestoner --publish 2.1.0-rc.1
```

### Pre-Release Versions

Publish pre-release versions using semantic versioning extensions.

**Alpha Release** - Early testing

```bash
milestoner --publish 2.0.0-alpha.1
milestoner --publish 2.0.0-alpha.2
```

**Beta Release** - Feature complete, testing

```bash
milestoner --publish 2.0.0-beta.1
milestoner --publish 2.0.0-beta.2
```

**Release Candidate** - Final testing before release

```bash
milestoner --publish 2.0.0-rc.1
milestoner --publish 2.0.0-rc.2
```

### GPG Signing

Automatically sign tags when GPG is configured.

**Setup GPG Signing**:

1. Generate GPG key:
```bash
gpg --gen-key
```

2. List keys and get ID:
```bash
gpg --list-secret-keys --keyid-format LONG
```

3. Configure Git:
```bash
git config --global user.signingkey YOUR_KEY_ID
git config --global tag.gpgSign true
git config --global commit.gpgSign true
```

4. Publish with automatic signing:
```bash
milestoner --publish minor
# Tag is automatically GPG signed
```

### What Publish Does

1. Calculates new version based on Git trailers or specified version
2. Generates release notes from commits since last tag
3. Creates annotated Git tag with release message
4. Signs tag with GPG (if configured)
5. Updates any configured outputs (CHANGELOG.md, etc.)

### Git Trailers for Automatic Versioning

Add trailers to commit messages to influence version calculation:

```
Add OAuth2 authentication

This implements OAuth2 support for Google and GitHub.

Milestone: minor
```

**Trailer Values**:
- `Milestone: patch` - Bump patch version
- `Milestone: minor` - Bump minor version
- `Milestone: major` - Bump major version

**Version Calculation**:
- Highest milestone value wins
- If no trailers present, defaults to patch bump
- Multiple commits with different milestones: highest takes precedence

**Example Scenario**:

```bash
# Commit 1
git commit -m "Fix login bug

Milestone: patch"

# Commit 2
git commit -m "Add new dashboard

Milestone: minor"

# Commit 3
git commit -m "Redesign API (breaking)

Milestone: major"

# Publish - will create major version bump
milestoner --publish
# Result: major version bump (highest milestone)
```

### Examples

```bash
# Patch release for bug fix
milestoner --publish patch

# Minor release for new feature
milestoner --publish minor

# Major release for breaking changes
milestoner --publish major

# Specific version
milestoner --publish 2.0.0

# Alpha release
milestoner --publish 2.0.0-alpha.1

# Beta release
milestoner --publish 2.0.0-beta.1
```

## Config Command

Manage Milestoner configuration files.

### Basic Usage

```bash
# View current configuration
milestoner config --view

# Create new configuration
milestoner config --create

# Edit configuration
milestoner config --edit

# Delete configuration
milestoner config --delete
```

### Options

**--view**

Display current configuration.

```bash
milestoner config --view
```

Shows merged configuration from:
1. Global config: `~/.config/milestoner/configuration.yml`
2. Local config: `.milestoner.yml` (overrides global)

**--create**

Create default configuration file.

```bash
# Create global config
milestoner config --create

# Create local project config
cd /path/to/project
touch .milestoner.yml
milestoner config --create
```

**--edit**

Open configuration in default editor.

```bash
milestoner config --edit
```

Uses `$EDITOR` environment variable or falls back to system default.

**--delete**

Delete configuration file.

```bash
milestoner config --delete
```

Prompts for confirmation before deletion.

### Configuration Locations

**Global Configuration**:
- Path: `~/.config/milestoner/configuration.yml`
- Applies to all projects
- Use for personal defaults

**Local Configuration**:
- Path: `.milestoner.yml` in repository root
- Overrides global settings
- Commit to version control
- Use for project-specific settings

### Examples

```bash
# View what configuration is active
milestoner config --view

# Create and edit configuration
milestoner config --create
milestoner config --edit

# Delete old configuration
milestoner config --delete
```

## Cache Command

Manage user information cache for linking contributors.

The cache stores mapping between Git commit authors and external identities (like GitHub user IDs) for avatar display and attribution.

### Basic Usage

```bash
# List all cached users
milestoner cache --list

# Add new user
milestoner cache --create "EXTERNAL_ID,HANDLE,NAME"

# Delete user
milestoner cache --delete "NAME"

# Show cache info
milestoner cache --info
```

### Options

**--list**

Display all cached users.

```bash
milestoner cache --list
```

Output format:
```
External ID | Handle      | Name
------------|-------------|------------------
12345       | jsmith      | Jane Smith
67890       | mjones      | Mike Jones
```

**--create "EXTERNAL_ID,HANDLE,NAME"**

Add user to cache.

Format: `"external_id,username,full name"`

```bash
# Add GitHub user
milestoner cache --create "12345,jsmith,Jane Smith"

# Add multiple users
milestoner cache --create "67890,mjones,Mike Jones"
milestoner cache --create "11111,alee,Alice Lee"
```

**Components**:
- `EXTERNAL_ID` - GitHub user ID (numeric)
- `HANDLE` - Username or handle
- `NAME` - Full name

**Finding GitHub User ID**:

```bash
# Using GitHub API
curl https://api.github.com/users/USERNAME

# Or visit: https://api.github.com/users/USERNAME
```

**--delete "NAME"**

Remove user from cache.

```bash
milestoner cache --delete "Jane Smith"
```

**--info**

Display cache statistics and location.

```bash
milestoner cache --info
```

Shows:
- Cache file path
- Number of users
- Last updated timestamp

### Cache Location

**File**: `~/.config/milestoner/cache.yml`

**Format**:
```yaml
---
users:
  - external_id: "12345"
    handle: "jsmith"
    name: "Jane Smith"
  - external_id: "67890"
    handle: "mjones"
    name: "Mike Jones"
```

### Use Cases

**Avatar Display**

With cached user data, release notes can display avatars:

```markdown
## Contributors

![Jane Smith](https://avatars.githubusercontent.com/u/12345?s=40) Jane Smith (@jsmith)
![Mike Jones](https://avatars.githubusercontent.com/u/67890?s=40) Mike Jones (@mjones)
```

**Attribution**

Link Git commit authors to external profiles:

```markdown
## Changes

* Fix authentication bug - @jsmith
* Add dashboard feature - @mjones
```

**Team Management**

Maintain consistent identity across:
- Multiple email addresses
- Name variations in Git config
- Different repositories

### Examples

```bash
# View current cache
milestoner cache --list

# Add team members
milestoner cache --create "12345,jsmith,Jane Smith"
milestoner cache --create "67890,mjones,Mike Jones"
milestoner cache --create "11111,alee,Alice Lee"

# Remove former team member
milestoner cache --delete "Former Employee"

# Check cache status
milestoner cache --info

# Bulk add from CSV
while IFS=, read -r id handle name; do
  milestoner cache --create "$id,$handle,$name"
done < team.csv
```

## Additional Git Trailers

Beyond the `Milestone` trailer, Milestoner supports additional trailers for enhanced release notes.

### Format Trailer

Override default commit format per commit.

```
Commit message here

Format: markdown
```

Values: `ascii_doc`, `markdown`

### Issue Trailer

Link commits to issue tracker.

```
Fix critical security vulnerability

Issue: 123
```

With tracker configuration:
```yaml
tracker:
  uri: "https://github.com/%<project_owner>s/%<project_name>s/issues/%<issue_id>s"
```

Automatically links to: `https://github.com/myorg/myproject/issues/123`

### Multiple Trailers

Combine multiple trailers:

```
Add OAuth2 authentication

Implements Google and GitHub authentication providers.

Milestone: minor
Issue: 45
Format: markdown
```

## Global Options

Options available for all commands:

**--help, -h**

Display help information.

```bash
milestoner --help
milestoner build --help
milestoner config --help
milestoner cache --help
```

**--version, -v**

Display Milestoner version.

```bash
milestoner --version
```

## Environment Variables

### EDITOR

Default editor for `milestoner config --edit`.

```bash
export EDITOR=vim
milestoner config --edit

export EDITOR=code
milestoner config --edit
```

### GPG_TTY

Required for GPG signing in some terminals.

```bash
export GPG_TTY=$(tty)
```

Add to shell profile (`.bashrc`, `.zshrc`):
```bash
echo 'export GPG_TTY=$(tty)' >> ~/.zshrc
```

## Exit Codes

- `0` - Success
- `1` - General error
- `2` - Configuration error
- `3` - Git error
- `4` - File system error

## Workflow Integration

### With Git Hooks

**pre-commit hook** - Validate commit messages:

```bash
#!/bin/bash
# .git/hooks/pre-commit

commit_msg=$(git log -1 --pretty=%B)

# Ensure milestone trailer present for releases
if [[ "$commit_msg" =~ "Release" ]]; then
  if ! grep -q "Milestone:" <<< "$commit_msg"; then
    echo "Error: Release commits must include Milestone trailer"
    exit 1
  fi
fi
```

**post-commit hook** - Auto-publish on release commits:

```bash
#!/bin/bash
# .git/hooks/post-commit

commit_msg=$(git log -1 --pretty=%B)

if [[ "$commit_msg" =~ "Release" ]]; then
  milestoner --publish
fi
```

### With CI/CD

**GitHub Actions**:

```yaml
name: Release
on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: 3.2

      - name: Install Milestoner
        run: gem install milestoner

      - name: Generate Release Notes
        run: milestoner build --format markdown

      - name: Create GitHub Release
        uses: actions/create-release@v1
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          body_path: tmp/milestones/index.md
```

**GitLab CI**:

```yaml
release:
  stage: release
  image: ruby:3.2
  script:
    - gem install milestoner
    - milestoner build --format markdown
    - milestoner --publish
  only:
    - main
  when: manual
```

### With Make

**Makefile**:

```makefile
.PHONY: release-patch release-minor release-major preview

preview:
	@milestoner build --format stream

release-patch:
	@milestoner --publish patch
	@git push origin main --tags

release-minor:
	@milestoner --publish minor
	@git push origin main --tags

release-major:
	@milestoner --publish major
	@git push origin main --tags

changelog:
	@milestoner build --format markdown --max 10
```

Usage:
```bash
make preview
make release-patch
make release-minor
make release-major
make changelog
```

### With npm/package.json

**package.json scripts**:

```json
{
  "scripts": {
    "release:preview": "milestoner build --format stream",
    "release:patch": "milestoner --publish patch && git push --follow-tags",
    "release:minor": "milestoner --publish minor && git push --follow-tags",
    "release:major": "milestoner --publish major && git push --follow-tags"
  }
}
```

Usage:
```bash
npm run release:preview
npm run release:patch
npm run release:minor
npm run release:major
```

## Troubleshooting

### Command Not Found

```bash
# Check installation
which milestoner

# Install if missing
gem install milestoner

# Check Ruby gems path
gem environment
```

### Permission Denied

```bash
# Fix gem installation permissions
gem install milestoner --user-install

# Or use bundler
bundle install --path vendor/bundle
```

### GPG Signing Fails

```bash
# Check GPG configuration
git config --get user.signingkey
git config --get tag.gpgSign

# Test GPG
echo "test" | gpg --clearsign

# Set GPG_TTY
export GPG_TTY=$(tty)
```

### Configuration Not Found

```bash
# Check config location
milestoner config --info

# Create default config
milestoner config --create

# Verify file exists
ls -la ~/.config/milestoner/configuration.yml
```

### No Tags Found

```bash
# Check for existing tags
git tag -l

# Create initial tag
git tag -a v0.1.0 -m "Initial release"
git push origin v0.1.0

# Then publish next version
milestoner --publish patch
```

### Wrong Version Calculated

```bash
# Check last tag
git describe --tags --abbrev=0

# Check commits since last tag
git log $(git describe --tags --abbrev=0)..HEAD

# Look for Milestone trailers
git log $(git describe --tags --abbrev=0)..HEAD --format="%B"

# Force specific version
milestoner --publish 1.2.3
```

## Advanced Usage

### Custom Templates

Copy default templates to customize:

```bash
# Find gem installation
gem which milestoner

# Copy templates
cp -r $(gem which milestoner | sed 's|lib/milestoner.rb|templates|') \
  ~/.config/milestoner/templates/

# Edit templates
vim ~/.config/milestoner/templates/layouts/page.html.erb
```

### Scripting

Automate release workflows:

```bash
#!/bin/bash
# release.sh - Automated release script

set -e

# Determine version bump
if grep -q "BREAKING" CHANGELOG.md; then
  VERSION_TYPE="major"
elif grep -q "feat:" CHANGELOG.md; then
  VERSION_TYPE="minor"
else
  VERSION_TYPE="patch"
fi

# Preview changes
echo "Preview of changes:"
milestoner build --format stream

# Confirm release
read -p "Release as $VERSION_TYPE? (y/n) " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
  # Publish
  milestoner --publish "$VERSION_TYPE"

  # Push
  git push origin main --tags

  # Announce
  echo "Released version $(git describe --tags)"
fi
```

### Integration with Changelog

Generate and commit changelog:

```bash
# Generate markdown changelog
milestoner build --format markdown --max 10

# Move to root as CHANGELOG.md
mv tmp/milestones/index.md CHANGELOG.md

# Commit
git add CHANGELOG.md
git commit -m "Update CHANGELOG.md"
```

### Manifest Generation

Generate JSON manifest for programmatic access:

```yaml
# .milestoner.yml
build:
  manifest: true
```

```bash
# Build with manifest
milestoner build --max 10

# Read manifest
cat tmp/milestones/manifest.json
```

Manifest structure:
```json
{
  "generator": {
    "label": "Milestoner",
    "version": "16.0.0"
  },
  "project": {
    "label": "MyProject",
    "version": "2.5.0"
  },
  "versions": [
    "2.5.0",
    "2.4.0",
    "2.3.0"
  ]
}
```

## Best Practices

1. **Use Git Trailers**: Add Milestone trailers for automatic versioning
2. **Preview First**: Run `milestoner build --format stream` before publishing
3. **Semantic Versioning**: Follow SemVer strictly
4. **GPG Signing**: Enable for security and authenticity
5. **Cache Team Members**: Populate cache for proper attribution
6. **Test Configuration**: Validate `.milestoner.yml` after changes
7. **Automate**: Integrate into CI/CD pipelines
8. **Document Custom Categories**: Comment configuration for team clarity

## References

- [Milestoner Documentation](https://alchemists.io/projects/milestoner)
- [Semantic Versioning](https://semver.org)
- [Git Trailers](https://git-scm.com/docs/git-interpret-trailers)
- [GPG Signing](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work)
