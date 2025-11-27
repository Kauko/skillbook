# Git Lint Configuration Reference

Complete reference for `.git-lint.yml` configuration options.

## Configuration File Location

Store at repository root: `.git-lint.yml`

For global configuration: `$HOME/.config/git-lint/configuration.yml`

## Configuration Structure

Configuration organizes settings into four main sections:

```yaml
commits:
  author: {}      # Author validation
  body: {}        # Commit body rules
  subject: {}     # Subject line rules
  trailer: {}     # Trailer/metadata rules
  signature: {}   # GPG signature validation
branches: {}      # Branch naming rules
issues: {}        # Issue tracker integration
```

## Common Parameters

### Severity Levels

- `warn` - Display issue without failing build
- `error` - Display issue and cause build failure

### Filter Types

- `includes` - Whitelist values (required match)
- `excludes` - Blacklist values (forbidden patterns)

### Standard Options

- `enabled` (boolean) - Toggle analyzer on/off
- `severity` (string) - Set to "warn" or "error"
- `minimum` (integer) - Minimum threshold
- `maximum` (integer) - Maximum threshold
- `delimiter` (string) - Separator character

## Author Settings

### commits.author.capitalization

Ensures author names use proper capitalization.

```yaml
commits:
  author:
    capitalization:
      enabled: true
      severity: error
```

**Default:** enabled=true, severity=error

### commits.author.email

Validates author email format.

```yaml
commits:
  author:
    email:
      enabled: true
      severity: error
```

**Default:** enabled=true, severity=error

### commits.author.name

Requires minimum two-word author names.

```yaml
commits:
  author:
    name:
      enabled: true
      severity: error
      minimum: 2
```

**Default:** enabled=true, severity=error, minimum=2

**Example:** "John Doe" passes, "John" fails

## Body Settings

### commits.body.bullet_capitalization

Enforces capital letters after bullet points.

```yaml
commits:
  body:
    bullet_capitalization:
      enabled: true
      severity: error
      includes:
        - "*"
        - "-"
```

**Default:** enabled=true, severity=error, includes=[asterisks, hyphens]

**Example:**
```
✓ - Fix login bug
✗ - fix login bug
```

### commits.body.bullet_delimiter

Requires space after bullet characters.

```yaml
commits:
  body:
    bullet_delimiter:
      enabled: true
      severity: error
      delimiter: " "
```

**Default:** enabled=true, severity=error, delimiter=" "

**Example:**
```
✓ - Item one
✗ -Item one
```

### commits.body.bullet_only

Prevents single bullet items when paragraphs are more appropriate.

```yaml
commits:
  body:
    bullet_only:
      enabled: true
      severity: warn
```

**Default:** enabled=true, severity=warn

**Rationale:** Single bullets add unnecessary formatting overhead.

### commits.body.leading_line

Ensures blank line between subject and body.

```yaml
commits:
  body:
    leading_line:
      enabled: true
      severity: error
```

**Default:** enabled=true, severity=error

**Example:**
```
✓ Subject line

   Body starts here

✗ Subject line
   Body without blank line
```

### commits.body.line_length

Checks maximum character width per line.

```yaml
commits:
  body:
    line_length:
      enabled: false
      severity: error
      maximum: 72
```

**Default:** enabled=false, severity=error, maximum=72

**Note:** Disabled by default to avoid URL/code snippet conflicts.

### commits.body.paragraph_capitalization

Capitalizes paragraph starting words.

```yaml
commits:
  body:
    paragraph_capitalization:
      enabled: true
      severity: error
```

**Default:** enabled=true, severity=error

### commits.body.phrase

Excludes non-descriptive or filler words.

```yaml
commits:
  body:
    phrase:
      enabled: true
      severity: warn
      excludes:
        - "absolutely"
        - "actually"
        - "basically"
        - "easy"
        - "just"
        - "obvious"
        - "obviously"
        - "of course"
        - "simply"
```

**Default:** enabled=true, severity=warn, excludes=[common filler words]

**Rationale:** These words rarely add value and can patronize readers.

### commits.body.presence

Requires minimum body lines for complex changes.

```yaml
commits:
  body:
    presence:
      enabled: true
      severity: warn
      minimum: 1
```

**Default:** enabled=true, severity=warn, minimum=1

**Rationale:** Encourages detailed explanations for WHY changes were made.

### commits.body.tracker_shorthand

Prevents issue tracker shorthand references in body.

```yaml
commits:
  body:
    tracker_shorthand:
      enabled: true
      severity: error
      excludes:
        - "#\\d+"
        - "GH-\\d+"
```

**Default:** enabled=true, severity=error

**Rationale:** Use formal trailer format for issue references.

**Example:**
```
✗ Body text with #123 reference
✓ Body text

   Issue: #123
```

### commits.body.word_repeat

Detects duplicate consecutive words.

```yaml
commits:
  body:
    word_repeat:
      enabled: true
      severity: error
```

**Default:** enabled=true, severity=error

**Example:**
```
✗ This fixes the the bug
✓ This fixes the bug
```

## Subject Settings

### commits.subject.length

Enforces maximum subject line length.

```yaml
commits:
  subject:
    length:
      enabled: true
      severity: error
      maximum: 72
```

**Default:** enabled=true, severity=error, maximum=72

**Rationale:** 72 characters ensures readability in `git log` output.

### commits.subject.prefix

Validates semantic prefixes.

```yaml
commits:
  subject:
    prefix:
      enabled: true
      severity: error
      includes:
        - Added
        - Updated
        - Fixed
        - Removed
        - Refactored
```

**Default:** enabled=true, severity=error, includes=[Added, Updated, Fixed, Removed, Refactored]

**Note:** For gitmoji workflow, disable this or customize includes.

### commits.subject.suffix

Prevents ending punctuation.

```yaml
commits:
  subject:
    suffix:
      enabled: true
      severity: error
      excludes:
        - "!"
        - "."
        - "?"
```

**Default:** enabled=true, severity=error, excludes=[!, ., ?]

**Rationale:** Subject lines are titles, not sentences.

### commits.subject.word_repeat

Detects duplicate consecutive words.

```yaml
commits:
  subject:
    word_repeat:
      enabled: true
      severity: error
```

**Default:** enabled=true, severity=error

## Trailer Settings

### commits.trailer.collaborator_*

Validates Git trailers for collaborators.

```yaml
commits:
  trailer:
    collaborator_capitalization:
      enabled: true
      severity: error
    collaborator_email:
      enabled: true
      severity: error
    collaborator_key:
      enabled: true
      severity: error
    collaborator_name:
      enabled: true
      severity: error
      minimum: 2
```

**Example:**
```
Co-Authored-By: Jane Smith <jane@example.com>
```

### commits.trailer.duplicate

Prevents duplicate trailer entries.

```yaml
commits:
  trailer:
    duplicate:
      enabled: true
      severity: error
```

### commits.trailer.format_*

Validates trailer formatting (key: value pairs).

```yaml
commits:
  trailer:
    format_key:
      enabled: true
      severity: error
    format_value:
      enabled: true
      severity: error
```

### commits.trailer.issue

Validates issue tracker references.

```yaml
commits:
  trailer:
    issue:
      enabled: true
      severity: error
```

**Example:**
```
Issue: #123
Fixes: #456
```

### commits.trailer.milestone_*

Validates version trailers for releases.

```yaml
commits:
  trailer:
    milestone_major:
      enabled: true
      severity: error
    milestone_minor:
      enabled: true
      severity: error
    milestone_patch:
      enabled: true
      severity: error
```

**Example:**
```
Version: 1.2.3
```

### commits.trailer.reviewer_*

Platform-specific reviewer validation.

```yaml
commits:
  trailer:
    reviewer_clickup:
      enabled: false
      severity: error
    reviewer_github:
      enabled: true
      severity: error
    reviewer_jira:
      enabled: false
      severity: error
    reviewer_linear:
      enabled: false
      severity: error
    reviewer_shortcut:
      enabled: false
      severity: error
    reviewer_tana:
      enabled: false
      severity: error
```

**Example:**
```
Reviewed-By: @username
```

### commits.trailer.signer_*

Validates commit signer trailers.

```yaml
commits:
  trailer:
    signer_capitalization:
      enabled: true
      severity: error
    signer_email:
      enabled: true
      severity: error
    signer_key:
      enabled: true
      severity: error
    signer_name:
      enabled: true
      severity: error
      minimum: 2
```

**Example:**
```
Signed-Off-By: John Doe <john@example.com>
```

### commits.trailer.tracker

Validates tracker name format.

```yaml
commits:
  trailer:
    tracker:
      enabled: true
      severity: error
```

## Signature Settings

### commits.signature

Validates GPG commit signatures.

```yaml
commits:
  signature:
    enabled: false
    severity: error
    includes:
      - Good
```

**Default:** enabled=false, severity=error

**Valid includes values:**
- `Bad` - Invalid signature
- `Error` - Verification error occurred
- `Good` - Valid signature
- `None` - No signature present
- `Revoked` - Signature key revoked
- `Unknown` - Unknown signature key
- `Expired` - Signature expired
- `Expired Key` - Signing key expired

**Example use cases:**
```yaml
# Require valid signatures
signature:
  enabled: true
  includes: [Good]

# Allow unsigned commits
signature:
  enabled: true
  includes: [None, Good]

# Require any signature attempt
signature:
  enabled: true
  excludes: [None]
```

## Branch Settings

### branches.name

Validates branch naming conventions.

```yaml
branches:
  enabled: true
  name: "\\A[a-z]+[a-z0-9-]*\\Z"
```

**Default:** enabled=true, name="\\A[a-z]+[a-z0-9-]*\\Z"

**Examples:**
```
✓ feature-new-login
✓ bugfix-123
✓ hotfix-security
✗ Feature-Branch (uppercase)
✗ -invalid-start (starts with hyphen)
```

## Issues Settings

### issues.enabled

Enables issue tracker integration.

```yaml
issues:
  enabled: false
```

**Default:** enabled=false

**Note:** Customize based on project needs and tracker type.

## Regular Expression Support

String values automatically convert to regex patterns. Use YAML escaping for special characters:

```yaml
# Simple string (becomes /example/)
subject:
  prefix:
    includes:
      - "Added"

# Regex with word boundary
body:
  phrase:
    excludes:
      - "\\beasy\\b"

# Complex pattern
subject:
  prefix:
    includes:
      - "\\A🤖 "  # Robot emoji prefix
```

## Complete Example Configuration

```yaml
# Two-tier commit system: humans free, agents validated
commits:
  author:
    capitalization:
      enabled: true
      severity: error
    email:
      enabled: true
      severity: error
    name:
      enabled: true
      severity: error
      minimum: 2

  body:
    bullet_capitalization:
      enabled: true
      severity: error
    bullet_delimiter:
      enabled: true
      severity: error
    leading_line:
      enabled: true
      severity: error
    line_length:
      enabled: false  # Disabled to allow URLs
    paragraph_capitalization:
      enabled: true
      severity: error
    phrase:
      enabled: true
      severity: warn
      excludes:
        - "absolutely"
        - "actually"
        - "basically"
        - "easy"
        - "just"
        - "obvious"
        - "obviously"
        - "of course"
        - "simply"
    presence:
      enabled: true
      severity: warn
      minimum: 1
    tracker_shorthand:
      enabled: true
      severity: error
    word_repeat:
      enabled: true
      severity: error

  subject:
    length:
      enabled: true
      severity: error
      maximum: 72
    prefix:
      enabled: false  # Disabled for gitmoji workflow
    suffix:
      enabled: true
      severity: error
      excludes: ["!", ".", "?"]
    word_repeat:
      enabled: true
      severity: error

  trailer:
    collaborator_capitalization:
      enabled: true
      severity: error
    collaborator_email:
      enabled: true
      severity: error
    collaborator_key:
      enabled: true
      severity: error
    collaborator_name:
      enabled: true
      severity: error
      minimum: 2

  signature:
    enabled: false
    severity: error

branches:
  enabled: true
  name: "\\A[a-z]+[a-z0-9-]*\\Z"

issues:
  enabled: false
```

## Tips for Customization

1. **Start strict, relax as needed** - Begin with default rules, disable problematic ones
2. **Use warn for experimental rules** - Test new rules with warnings before making them errors
3. **Document exceptions** - Add YAML comments explaining why rules are disabled
4. **Team consensus** - Discuss configuration changes with team before enforcing
5. **Test thoroughly** - Create test commits to validate configuration behaves as expected

## Debugging Configuration

**Check YAML syntax:**
```bash
ruby -ryaml -e "YAML.load_file('.git-lint.yml')"
```

**Test specific commit:**
```bash
git-lint analyze --commit SHA
```

**Dry run with verbose output:**
```bash
git-lint analyze --branch --verbose
```
