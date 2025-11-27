# Git Lint Analyzers Reference

Complete reference for all 40+ built-in analyzers and customization guidance.

## Analyzer Overview

Git Lint provides analyzers organized into five categories:

1. **Author Analyzers (3)** - Validate commit author metadata
2. **Body Analyzers (10)** - Validate commit message body content
3. **Subject Analyzers (4)** - Validate subject line format
4. **Trailer Analyzers (21)** - Validate Git trailers and metadata
5. **Signature Analyzers (1)** - Validate GPG signatures

## Author Analyzers

### Capitalization

**Purpose:** Ensures author names use proper capitalization.

**Configuration:**
```yaml
commits:
  author:
    capitalization:
      enabled: true
      severity: error
```

**Validates:**
- First letter of each name part capitalized
- Proper noun formatting

**Examples:**
```
✓ John Doe
✓ Mary Jane Smith
✗ john doe
✗ JOHN DOE
```

### Email

**Purpose:** Validates author email format.

**Configuration:**
```yaml
commits:
  author:
    email:
      enabled: true
      severity: error
```

**Validates:**
- Standard email format: user@domain.ext
- Domain has valid TLD
- No spaces or invalid characters

**Examples:**
```
✓ john@example.com
✓ jane.smith@company.co.uk
✗ john@example
✗ @example.com
✗ john @example.com
```

### Name

**Purpose:** Requires minimum two-word author names.

**Configuration:**
```yaml
commits:
  author:
    name:
      enabled: true
      severity: error
      minimum: 2
```

**Validates:**
- Minimum word count (default: 2)
- Space-separated name parts

**Examples:**
```
✓ John Doe
✓ Mary Jane Smith
✗ John
✗ J
```

**Rationale:** Single names lack clarity in collaborative environments.

## Body Analyzers

### Bullet Capitalization

**Purpose:** Enforces capital letters after bullet points.

**Configuration:**
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

**Validates:**
- First word after bullet is capitalized
- Applies to asterisks and hyphens by default

**Examples:**
```
✓ - Fix authentication bug
✓ * Add new feature
✗ - fix authentication bug
✗ * add new feature
```

### Bullet Delimiter

**Purpose:** Requires space after bullet characters.

**Configuration:**
```yaml
commits:
  body:
    bullet_delimiter:
      enabled: true
      severity: error
      delimiter: " "
```

**Validates:**
- Space present after bullet character
- Consistent delimiter usage

**Examples:**
```
✓ - Item one
✓ * Item two
✗ -Item one
✗ *Item two
```

### Bullet Only

**Purpose:** Prevents single bullet items when paragraphs are more appropriate.

**Configuration:**
```yaml
commits:
  body:
    bullet_only:
      enabled: true
      severity: warn
```

**Validates:**
- Multiple bullets present if any bullets used
- Flags single-bullet lists

**Examples:**
```
✓ - First item
   - Second item

✓ This is a paragraph without bullets.

✗ - Only one bullet item
```

**Rationale:** Single bullets add formatting overhead without value.

### Leading Line

**Purpose:** Ensures blank line between subject and body.

**Configuration:**
```yaml
commits:
  body:
    leading_line:
      enabled: true
      severity: error
```

**Validates:**
- Empty line after subject before body starts
- Git message format convention

**Examples:**
```
✓ Subject line

   Body content here

✗ Subject line
   Body without blank line
```

**Rationale:** Git tools expect this format for proper parsing.

### Line Length

**Purpose:** Checks maximum character width per line.

**Configuration:**
```yaml
commits:
  body:
    line_length:
      enabled: false
      severity: error
      maximum: 72
```

**Validates:**
- Line length under maximum (default: 72)
- Applied per line in body

**Default:** Disabled to avoid conflicts with URLs and code snippets.

**When to enable:**
- Projects without URLs in commits
- Teams with strict formatting standards
- When using text-based Git tools exclusively

### Paragraph Capitalization

**Purpose:** Capitalizes paragraph starting words.

**Configuration:**
```yaml
commits:
  body:
    paragraph_capitalization:
      enabled: true
      severity: error
```

**Validates:**
- First word of each paragraph capitalized
- Proper sentence formatting

**Examples:**
```
✓ This is the first paragraph.

   Another paragraph starts here.

✗ this paragraph isn't capitalized.
```

### Phrase

**Purpose:** Excludes non-descriptive or filler words.

**Configuration:**
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

**Validates:**
- Excludes patronizing language
- Removes assumed knowledge phrases
- Flags subjective difficulty claims

**Examples:**
```
✗ Just add this simple change
✗ Obviously this fixes the bug
✗ It's basically a quick fix
✓ Add configuration option for timeouts
✓ Fix race condition in order processing
```

**Rationale:**
- "Just" and "simply" minimize complexity
- "Obviously" and "of course" assume reader knowledge
- "Easy" and "basically" are subjective

**Custom additions:**
```yaml
phrase:
  excludes:
    - "FIXME"
    - "TODO"
    - "hack"
    - "quick fix"
```

### Presence

**Purpose:** Requires minimum body lines for complex changes.

**Configuration:**
```yaml
commits:
  body:
    presence:
      enabled: true
      severity: warn
      minimum: 1
```

**Validates:**
- Minimum line count in body
- Encourages detailed explanations

**Default:** Warning level, minimum 1 line

**Examples:**
```
✓ Add OAuth2 authentication

   Implements Google and GitHub OAuth2 flows with PKCE
   support for enhanced security.

✗ Add OAuth2 authentication
```

**Rationale:** Bodies explain WHY, subjects explain WHAT.

**When to adjust:**
```yaml
# Require detailed explanations
presence:
  minimum: 3
  severity: error

# Allow minimal bodies for trivial changes
presence:
  minimum: 0
  severity: warn
```

### Tracker Shorthand

**Purpose:** Prevents issue tracker shorthand references in body.

**Configuration:**
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

**Validates:**
- No shorthand issue references in body text
- Forces proper trailer format

**Examples:**
```
✗ Fix bug mentioned in #123

✓ Fix authentication timeout bug

   Issue: #123
```

**Rationale:** Trailers provide structured, parseable metadata.

**Custom patterns:**
```yaml
tracker_shorthand:
  excludes:
    - "#\\d+"           # GitHub: #123
    - "GH-\\d+"         # GitHub: GH-123
    - "JIRA-\\d+"       # Jira: JIRA-123
    - "LP#\\d+"         # Launchpad: LP#123
```

### Word Repeat

**Purpose:** Detects duplicate consecutive words.

**Configuration:**
```yaml
commits:
  body:
    word_repeat:
      enabled: true
      severity: error
```

**Validates:**
- No duplicate consecutive words
- Case-insensitive matching

**Examples:**
```
✗ This fixes the the bug
✗ Add add support for OAuth
✓ This fixes the bug
✓ Add support for OAuth
```

**Rationale:** Catches common typing errors.

## Subject Analyzers

### Length

**Purpose:** Enforces maximum subject line length.

**Configuration:**
```yaml
commits:
  subject:
    length:
      enabled: true
      severity: error
      maximum: 72
```

**Validates:**
- Character count under maximum
- Standard is 72 characters

**Examples:**
```
✓ Add OAuth2 authentication support (34 chars)
✗ Add comprehensive OAuth2 authentication support with PKCE security for Google and GitHub (85 chars)
```

**Rationale:**
- Git log output is typically 80 columns
- Subject should be scannable in one line
- Long subjects indicate lack of focus

**Adjusting limits:**
```yaml
# Strict 50-character limit
length:
  maximum: 50

# Relaxed 80-character limit
length:
  maximum: 80
```

### Prefix

**Purpose:** Validates semantic prefixes.

**Configuration:**
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

**Validates:**
- Subject starts with approved prefix
- Consistent verb tense (past tense)

**Default prefixes:**
- `Added` - New features or capabilities
- `Updated` - Changes to existing features
- `Fixed` - Bug fixes
- `Removed` - Deleted features or code
- `Refactored` - Code restructuring

**Examples:**
```
✓ Added OAuth2 authentication
✓ Fixed race condition in orders
✗ Adding OAuth2 authentication (wrong tense)
✗ Implement OAuth2 (not in prefix list)
```

**For gitmoji workflow:**
```yaml
# Disable prefix validation for gitmoji
subject:
  prefix:
    enabled: false
```

**Custom prefixes:**
```yaml
subject:
  prefix:
    includes:
      - Add      # Present tense
      - Update
      - Fix
      - Remove
      - Refactor
      - Bump     # For dependency updates
      - Merge    # For merge commits
```

### Suffix

**Purpose:** Prevents ending punctuation.

**Configuration:**
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

**Validates:**
- Subject doesn't end with punctuation
- Treats subjects as titles, not sentences

**Examples:**
```
✗ Add OAuth2 authentication.
✗ Fix bug!
✗ Does this work?
✓ Add OAuth2 authentication
✓ Fix authentication timeout bug
```

**Rationale:** Subject lines are titles/labels, not complete sentences.

### Word Repeat

**Purpose:** Detects duplicate consecutive words.

**Configuration:**
```yaml
commits:
  subject:
    word_repeat:
      enabled: true
      severity: error
```

**Validates:**
- No duplicate consecutive words
- Case-insensitive matching

**Examples:**
```
✗ Add add OAuth2 support
✗ Fix the the timeout bug
✓ Add OAuth2 support
✓ Fix the timeout bug
```

## Trailer Analyzers

Trailers are key-value pairs at the end of commit messages following Git conventions.

### Standard Trailer Format

```
Commit subject line

Commit body with details about the change.

Trailer-Key: Trailer value
Another-Key: Another value
```

### Collaborator Analyzers (4)

Validate `Co-Authored-By` trailers for pair programming.

**Capitalization:**
```yaml
commits:
  trailer:
    collaborator_capitalization:
      enabled: true
      severity: error
```

**Email:**
```yaml
commits:
  trailer:
    collaborator_email:
      enabled: true
      severity: error
```

**Key:**
```yaml
commits:
  trailer:
    collaborator_key:
      enabled: true
      severity: error
```

**Name:**
```yaml
commits:
  trailer:
    collaborator_name:
      enabled: true
      severity: error
      minimum: 2
```

**Example:**
```
Add payment processing feature

Implements Stripe integration.

Co-Authored-By: Jane Smith <jane@example.com>
```

### Duplicate

**Purpose:** Prevents duplicate trailer entries.

**Configuration:**
```yaml
commits:
  trailer:
    duplicate:
      enabled: true
      severity: error
```

**Validates:**
- No duplicate key-value pairs
- Unique trailer entries

**Examples:**
```
✗ Issue: #123
   Issue: #123

✓ Issue: #123
   Fixes: #456
```

### Format Analyzers (2)

**Format Key:**
```yaml
commits:
  trailer:
    format_key:
      enabled: true
      severity: error
```

Validates trailer keys follow Git conventions:
- Capitalized words
- Hyphen-separated
- No spaces

**Format Value:**
```yaml
commits:
  trailer:
    format_value:
      enabled: true
      severity: error
```

Validates trailer values:
- Non-empty
- Properly formatted

**Examples:**
```
✓ Issue-Tracker: #123
✓ Reviewed-By: @username
✗ issue tracker: #123
✗ ReviewedBy: (no value)
```

### Issue

**Purpose:** Validates issue tracker references.

**Configuration:**
```yaml
commits:
  trailer:
    issue:
      enabled: true
      severity: error
```

**Validates:**
- Proper issue reference format
- Compatible with GitHub, Jira, etc.

**Examples:**
```
✓ Issue: #123
✓ Fixes: #456
✓ Closes: JIRA-789
```

### Milestone Analyzers (3)

Validate version trailers for releases.

**Configuration:**
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

**Validates:**
- Semantic versioning format
- Valid version components

**Examples:**
```
✓ Version: 1.2.3
✓ Release: 2.0.0
✗ Version: 1.2.3.4 (too many parts)
✗ Version: v1.2.3 (prefix not allowed)
```

### Reviewer Analyzers (6)

Platform-specific reviewer validation.

**Available platforms:**
- ClickUp
- GitHub
- Jira
- Linear
- Shortcut
- Tana

**Configuration:**
```yaml
commits:
  trailer:
    reviewer_github:
      enabled: true
      severity: error
```

**Enable only your platform:**
```yaml
# GitHub projects
reviewer_github:
  enabled: true

# Jira projects
reviewer_jira:
  enabled: true

# Disable others
reviewer_clickup:
  enabled: false
reviewer_linear:
  enabled: false
reviewer_shortcut:
  enabled: false
reviewer_tana:
  enabled: false
```

**Examples:**
```
# GitHub
Reviewed-By: @username

# Jira
Reviewed-By: john.doe
```

### Signer Analyzers (4)

Validate `Signed-Off-By` trailers for DCO compliance.

**Configuration:**
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
Add feature implementation

Implements core functionality.

Signed-Off-By: John Doe <john@example.com>
```

**Use case:** Developer Certificate of Origin (DCO) compliance.

### Tracker

**Purpose:** Validates tracker name format.

**Configuration:**
```yaml
commits:
  trailer:
    tracker:
      enabled: true
      severity: error
```

**Validates:**
- Tracker key is properly formatted
- Value matches expected pattern

## Signature Analyzer

### Signature

**Purpose:** Validates GPG commit signatures.

**Configuration:**
```yaml
commits:
  signature:
    enabled: false
    severity: error
    includes:
      - Good
```

**Valid signature states:**
- `Bad` - Invalid signature
- `Error` - Verification error
- `Good` - Valid signature
- `None` - No signature
- `Revoked` - Key revoked
- `Unknown` - Unknown key
- `Expired` - Signature expired
- `Expired Key` - Key expired

**Common configurations:**

**Require valid signatures:**
```yaml
signature:
  enabled: true
  includes: [Good]
```

**Allow unsigned commits:**
```yaml
signature:
  enabled: true
  includes: [None, Good]
```

**Require any signature attempt:**
```yaml
signature:
  enabled: true
  excludes: [None]
```

**Check signature setup:**
```bash
git log --show-signature -1
```

## Customizing Analyzers

### Disabling Analyzers

**Disable individual analyzer:**
```yaml
commits:
  body:
    line_length:
      enabled: false
```

**Disable multiple analyzers:**
```yaml
commits:
  subject:
    prefix:
      enabled: false
    suffix:
      enabled: false
```

### Adjusting Severity

**Change to warning:**
```yaml
commits:
  body:
    presence:
      severity: warn
```

**Change to error:**
```yaml
commits:
  body:
    presence:
      severity: error
```

### Modifying Thresholds

**Adjust line length:**
```yaml
commits:
  subject:
    length:
      maximum: 50  # Stricter
```

**Adjust name minimum:**
```yaml
commits:
  author:
    name:
      minimum: 3  # Require three-part names
```

### Customizing Filters

**Add custom prefixes:**
```yaml
commits:
  subject:
    prefix:
      includes:
        - Add
        - Update
        - Fix
        - Remove
        - Bump
        - Merge
```

**Add custom phrase exclusions:**
```yaml
commits:
  body:
    phrase:
      excludes:
        - "absolutely"
        - "FIXME"
        - "TODO"
        - "hack"
        - "quick fix"
        - "temporary"
```

### Platform-Specific Configurations

**For gitmoji workflow:**
```yaml
commits:
  subject:
    prefix:
      enabled: false  # Disable semantic prefixes
    length:
      maximum: 72

  body:
    presence:
      severity: warn  # Encourage but don't require
```

**For Conventional Commits:**
```yaml
commits:
  subject:
    prefix:
      includes:
        - "feat"
        - "fix"
        - "docs"
        - "style"
        - "refactor"
        - "test"
        - "chore"
```

**For DCO compliance:**
```yaml
commits:
  trailer:
    signer_capitalization:
      enabled: true
    signer_email:
      enabled: true
    signer_name:
      enabled: true
  signature:
    enabled: true
    includes: [Good]
```

## Testing Analyzer Configuration

**Test against specific commit:**
```bash
git-lint analyze --commit SHA
```

**Test current branch:**
```bash
git-lint analyze --branch
```

**Test with verbose output:**
```bash
git-lint analyze --branch --verbose
```

**Create test commits:**
```bash
# Test subject length
git commit --allow-empty -m "Very long subject line that exceeds the maximum configured length limit and should trigger validation error"

# Test body presence
git commit --allow-empty -m "Short subject"

# Test trailers
git commit --allow-empty -m "Add feature

Implements new capability.

Issue: #123
Co-Authored-By: Jane Smith <jane@example.com>"
```

## Performance Considerations

- **Disable unused analyzers** - Reduces validation time
- **Use warn for experimental rules** - Test before enforcing
- **Enable signature checking selectively** - GPG validation is slow
- **Configure platform-specific analyzers** - Only enable your tracker

## Best Practices

1. **Start with defaults** - Git Lint's defaults are well-tested
2. **Customize incrementally** - Change one rule at a time
3. **Document exceptions** - Explain why rules are disabled
4. **Team consensus** - Discuss changes before enforcing
5. **Test thoroughly** - Validate configuration with test commits
6. **Use warnings first** - Test rules before making them errors
7. **Platform alignment** - Match your team's tools and workflow
