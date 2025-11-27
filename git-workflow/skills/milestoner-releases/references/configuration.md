# Milestoner Configuration Reference

Complete reference for all `.milestoner.yml` configuration options.

## Configuration File Location

**Global**: `~/.config/milestoner/configuration.yml`
**Project**: `.milestoner.yml` in repository root

Project configuration overrides global settings.

## Configuration Structure

### avatar

Controls avatar display in release notes and web output.

```yaml
avatar:
  uri: "https://avatars.githubusercontent.com/u/%<external_id>s?s=40"
```

- `uri` - Avatar URL format with `%<external_id>s` placeholder for GitHub user IDs

### build

Controls output generation and formatting.

```yaml
build:
  basename: "index"           # Output filename (without extension)
  format: "stream"            # Output format (ascii_doc, feed, markdown, stream, web)
  index: true                 # Generate versions index page
  layout: "page"              # Hanami Views layout name
  manifest: false             # Generate manifest.json with metadata
  max: 1                      # Maximum Git tags to build (1 = latest only)
  output: "tmp/milestones"    # Output directory path
  stylesheet: true            # Include CSS stylesheet for web format
```

**Format Options**:
- `ascii_doc` - AsciiDoctor rendering
- `feed` - Atom syndication format
- `markdown` - Markdown with syntax highlighting
- `stream` - Console/terminal output
- `web` - Full HTML with styling

### commit

Controls commit categorization and formatting.

```yaml
commit:
  # Categorize commits by prefix
  categories:
    added:
      emoji: "✨"
      label: "Added"
    updated:
      emoji: "⬆️"
      label: "Updated"
    fixed:
      emoji: "🐛"
      label: "Fixed"
    removed:
      emoji: "♻️"
      label: "Removed"
    refactored:
      emoji: "🔧"
      label: "Refactored"

  # Commit message syntax
  format: "asciidoc"

  # Commit link format
  uri: "https://github.com/%<project_owner>s/%<project_name>s/commit/%<commit_sha>s"
```

**Category Configuration**:
- `emoji` - Emoji symbol for visual categorization
- `label` - Category display name

**Format Options**:
- `asciidoc` - AsciiDoc syntax
- `markdown` - Markdown syntax

### project

Core project metadata required for release notes.

```yaml
project:
  author: "Jane Smith"                    # Project author name
  description: "A toolkit for X"          # Short project description (optional)
  label: "MyProject"                      # Display label (optional)
  name: "myproject"                       # Project name (required)
  owner: "myorg"                          # Repository owner/organization
  uri:
    home: "https://example.com"           # Project homepage
    version: "https://github.com/%<project_owner>s/%<project_name>s/tags/%<project_version>s"
```

**Required Fields**:
- `name` - Used in URLs, file paths, and identifiers

**Optional Fields**:
- `author` - Auto-detected from Git config if not specified
- `description` - Shown in feeds and web output
- `label` - Defaults to `name` if not provided
- `owner` - Defaults to "undefined" if not set

### organization

Organization/company information for branding.

```yaml
organization:
  label: "Acme Corporation"
  uri: "https://acme.com"
```

- `label` - Organization display name
- `uri` - Organization homepage URL

### tag

Controls Git tag creation.

```yaml
tag:
  subject: "Version %<project_version>s"
```

- `subject` - Tag message template with `%<project_version>s` placeholder

### syndication

Feed-specific configuration for Atom format output.

```yaml
syndication:
  categories:
    - label: "Software"
      name: "software"
    - label: "Development"
      name: "development"

  entry:
    label: "Milestone %<project_version>s"
    uri: "https://example.com/releases/%<project_version>s"

  links:
    html:
      type: "text/html"
      uri: "https://example.com/milestones"
    xml:
      type: "application/atom+xml"
      uri: "https://example.com/milestones.xml"
```

**Categories**:
- Feed taxonomy for discovery and filtering

**Entry**:
- `label` - Individual release entry title
- `uri` - Permalink to release

**Links**:
- `html` - Human-readable milestones page
- `xml` - Machine-readable feed URL

### tracker

Issue tracking integration.

```yaml
tracker:
  uri: "https://github.com/%<project_owner>s/%<project_name>s/issues/%<issue_id>s"
```

- `uri` - Issue link format with `%<issue_id>s` placeholder

**Git Trailer Integration**:
```
Commit message here

Issue: 123
```
Automatically links to: `https://github.com/myorg/myproject/issues/123`

## Example: Simplified Configuration

Minimal `.milestoner.yml` for quick setup:

```yaml
project:
  name: "my-app"
  owner: "mycompany"
  description: "My application description"

build:
  format: "stream"
  max: 1

commit:
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
```

## Example: Full Production Configuration

Comprehensive setup with all options:

```yaml
avatar:
  uri: "https://avatars.githubusercontent.com/u/%<external_id>s?s=40"

build:
  basename: "changelog"
  format: "markdown"
  index: true
  manifest: true
  max: 10
  output: "docs/releases"
  stylesheet: true

commit:
  categories:
    added:
      emoji: "✨"
      label: "Added"
    updated:
      emoji: "⬆️"
      label: "Updated"
    fixed:
      emoji: "🐛"
      label: "Fixed"
    security:
      emoji: "🔒"
      label: "Security"
    performance:
      emoji: "⚡"
      label: "Performance"
    refactored:
      emoji: "♻️"
      label: "Refactored"
    removed:
      emoji: "🗑️"
      label: "Removed"
    deprecated:
      emoji: "⚠️"
      label: "Deprecated"
  format: "markdown"
  uri: "https://github.com/%<project_owner>s/%<project_name>s/commit/%<commit_sha>s"

project:
  author: "Development Team"
  description: "Enterprise application platform"
  label: "MyApp"
  name: "myapp"
  owner: "mycompany"
  uri:
    home: "https://myapp.example.com"
    version: "https://github.com/%<project_owner>s/%<project_name>s/releases/tag/%<project_version>s"

organization:
  label: "My Company Inc"
  uri: "https://mycompany.com"

tag:
  subject: "Release %<project_version>s"

syndication:
  categories:
    - label: "Software"
      name: "software"
    - label: "Enterprise"
      name: "enterprise"
  entry:
    label: "Release %<project_version>s"
    uri: "https://myapp.example.com/releases/%<project_version>s"
  links:
    html:
      type: "text/html"
      uri: "https://myapp.example.com/releases"
    xml:
      type: "application/atom+xml"
      uri: "https://myapp.example.com/releases.xml"

tracker:
  uri: "https://github.com/%<project_owner>s/%<project_name>s/issues/%<issue_id>s"
```

## Template Placeholders

Available placeholders for URI templates:

- `%<project_name>s` - Project name from configuration
- `%<project_owner>s` - Repository owner from configuration
- `%<project_version>s` - Current version being generated
- `%<commit_sha>s` - Git commit SHA
- `%<issue_id>s` - Issue ID from Git trailer
- `%<external_id>s` - User's external ID (GitHub user ID)

## Configuration Validation

Validate YAML syntax:

```bash
# Using Ruby
ruby -ryaml -e "YAML.load_file('.milestoner.yml')"

# Using yq (if installed)
yq eval '.milestoner.yml'
```

## Configuration Management Commands

```bash
# View current configuration
milestoner config --view

# Create default configuration
milestoner config --create

# Edit configuration
milestoner config --edit

# Delete configuration
milestoner config --delete
```

## Migration from Legacy Configuration

If using older `.milestoner` format (without `.yml` extension), migrate by:

1. Rename `.milestoner` to `.milestoner.yml`
2. Validate YAML syntax
3. Update any deprecated keys (check release notes)
4. Test with `milestoner build --format stream`

## Configuration Best Practices

1. **Start Simple**: Begin with minimal config, add options as needed
2. **Version Control**: Commit `.milestoner.yml` to repository
3. **Team Consistency**: Use same categories and emoji across team
4. **Document Custom Categories**: Add comments explaining project-specific categories
5. **Test Changes**: Run `milestoner build` after config changes
6. **Global Defaults**: Set personal preferences in global config
7. **Project Overrides**: Override only what differs in project config

## Common Configuration Patterns

### Pattern: Conventional Commits

Map to Conventional Commits specification:

```yaml
commit:
  categories:
    feat:
      emoji: "✨"
      label: "Features"
    fix:
      emoji: "🐛"
      label: "Bug Fixes"
    docs:
      emoji: "📝"
      label: "Documentation"
    style:
      emoji: "💎"
      label: "Style"
    refactor:
      emoji: "♻️"
      label: "Refactoring"
    perf:
      emoji: "⚡"
      label: "Performance"
    test:
      emoji: "✅"
      label: "Tests"
    chore:
      emoji: "🔧"
      label: "Chores"
```

### Pattern: Gitmoji

Use gitmoji.dev emojis:

```yaml
commit:
  categories:
    sparkles:
      emoji: "✨"
      label: "New Features"
    bug:
      emoji: "🐛"
      label: "Bug Fixes"
    fire:
      emoji: "🔥"
      label: "Remove Code/Files"
    lock:
      emoji: "🔒"
      label: "Security"
    zap:
      emoji: "⚡"
      label: "Performance"
    lipstick:
      emoji: "💄"
      label: "UI/Style"
    recycle:
      emoji: "♻️"
      label: "Refactoring"
    white_check_mark:
      emoji: "✅"
      label: "Tests"
    arrow_up:
      emoji: "⬆️"
      label: "Upgrade Dependencies"
    arrow_down:
      emoji: "⬇️"
      label: "Downgrade Dependencies"
```

### Pattern: Monorepo

Different configs per package:

```yaml
# Root .milestoner.yml
project:
  name: "monorepo"
  owner: "myorg"

# packages/frontend/.milestoner.yml
project:
  name: "frontend"
  owner: "myorg"
  description: "Frontend application"

build:
  output: "packages/frontend/docs"

# packages/backend/.milestoner.yml
project:
  name: "backend"
  owner: "myorg"
  description: "Backend API"

build:
  output: "packages/backend/docs"
```

## Troubleshooting

### Configuration Not Loading

1. Check file name is exactly `.milestoner.yml`
2. Verify YAML syntax is valid
3. Ensure file is in repository root
4. Check file permissions (must be readable)

### Categories Not Working

1. Verify emoji matches exactly (including variation selectors)
2. Check category keys match commit prefixes
3. Test regex if using pattern matching
4. Review commit message format

### URIs Not Generated Correctly

1. Verify all required placeholders are present
2. Check owner and name are set correctly
3. Ensure URI template syntax uses `%<placeholder>s` format
4. Test with simple commit first

### Template Not Applied

1. Check template exists in `~/.config/milestoner/templates/`
2. Verify directory structure matches gem structure
3. Ensure file extensions are correct (`.html.erb`, `.adoc.erb`)
4. Review Hanami Views documentation for syntax
