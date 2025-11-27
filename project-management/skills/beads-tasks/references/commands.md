# Beads Commands Reference

Complete reference for all `bd` commands, options, and syntax.

## Installation Commands

### Quick Install (macOS/Linux)
```bash
curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/scripts/install.sh | bash
```

### npm (Node.js)
```bash
npm install -g @beads/bd
```

### Homebrew
```bash
brew tap steveyegge/beads
brew install bd
```

### Linux Requirements
- glibc 2.32+ (Ubuntu 22.04+, Debian 11+, RHEL 9+)

## Initialization Commands

### bd init
Initialize a Beads repository.

```bash
bd init                          # Standard workflow
bd init --contributor           # Fork-based contribution
bd init --team                  # Branch-based team workflow
bd init --branch beads-metadata # Protected branch support
bd init --quiet                 # Non-interactive (for agents)
bd init --stealth               # Local-only, hidden from repo
```

**Options:**
- `--contributor`: Set up for fork-based contribution workflow
- `--team`: Set up for branch-based team collaboration
- `--branch <name>`: Specify custom branch for metadata (for protected main branches)
- `--quiet`: Non-interactive initialization, good for automation
- `--stealth`: Store `.beads/` in `~/.config/git/ignore` to keep it hidden from repository

**Files Created (Standard Init):**
- **Committed:** `.beads/beads.jsonl`, `.beads/deletions.jsonl`, `.beads/config.yaml`, `.gitattributes`
- **Local (gitignored):** `.beads/beads.db`, `.beads/bd.sock`, daemon lock files

### bd onboard
Interactive guide for agents to get started with Beads.

```bash
bd onboard
```

Provides step-by-step instructions for:
1. Understanding Beads integration
2. Reviewing workflow documentation
3. Updating AGENTS.md with bd workflows
4. Updating CLAUDE.md if present
5. Removing bootstrap instructions

### bd quickstart
Full workflow tutorial for learning Beads.

```bash
bd quickstart
```

## Issue Creation Commands

### bd create
Create a new issue.

```bash
bd create "Issue title"
bd create "Title" -d "Description"
bd create "Title" --priority 1 --type bug
bd create "Title" -p 2 -t feature -l "backend,urgent"
bd create "Title" --assignee alice --json
bd create --from-template epic "Epic Name"
```

**Options:**
- `-d, --description <text>`: Issue description
- `-p, --priority <0-4>`: Priority level (0=lowest, 4=highest)
- `-t, --type <type>`: Issue type (bug, feature, task, epic, etc.)
- `-l, --labels <labels>`: Comma-separated labels
- `--assignee <name>`: Assign to user
- `--json`: Output in JSON format
- `--from-template <name>`: Create from template

**Priority Levels:**
- 0: Lowest priority
- 1: Low priority
- 2: Medium priority (default)
- 3: High priority
- 4: Critical/Urgent

**Common Types:**
- `bug`: Bug fix
- `feature`: New feature
- `task`: General task
- `epic`: Large feature or initiative
- `chore`: Maintenance work
- `doc`: Documentation

## Issue Listing Commands

### bd list
List all issues with optional filters.

```bash
bd list                    # Show all issues
bd list --status open     # Filter by status
bd list --status closed   # Show closed issues
bd list --type bug        # Filter by type
bd list --priority 3      # Filter by priority
bd list --label backend   # Filter by label
bd list --assignee alice  # Filter by assignee
bd list --json            # JSON output
```

**Options:**
- `--status <status>`: Filter by status (open, in_progress, closed, etc.)
- `--type <type>`: Filter by type
- `--priority <level>`: Filter by priority
- `--label <label>`: Filter by label
- `--assignee <name>`: Filter by assignee
- `--json`: Output in JSON format

### bd ready
Show issues ready to work on (no blocking dependencies).

```bash
bd ready                   # Show ready issues
bd ready --json           # JSON output
bd ready --priority 3     # Ready high-priority issues
bd ready --label backend  # Ready backend issues
```

**Key Command for Agents**: This finds actionable work without blockers.

**Options:**
- `--json`: Output in JSON format
- `--priority <level>`: Filter by priority
- `--label <label>`: Filter by label
- `--type <type>`: Filter by type

### bd show
View details of a specific issue.

```bash
bd show <issue-id>        # Show issue details
bd show <issue-id> --json # JSON output
```

**Output includes:**
- Issue metadata (ID, title, status, type, priority)
- Description
- Labels and assignee
- Dependencies (blocks, blocked-by, related, parent, children)
- Creation and update timestamps

## Issue Update Commands

### bd update
Update an existing issue.

```bash
bd update <issue-id> --status in_progress
bd update <issue-id> --title "New Title"
bd update <issue-id> --description "Updated description"
bd update <issue-id> --priority 4
bd update <issue-id> --type bug
bd update <issue-id> --assignee bob
bd update <issue-id> --labels "urgent,backend"
bd update <issue-id> --json
```

**Options:**
- `--status <status>`: Change status
- `--title <text>`: Update title
- `--description <text>`: Update description
- `--priority <level>`: Change priority (0-4)
- `--type <type>`: Change type
- `--assignee <name>`: Change assignee
- `--labels <labels>`: Update labels (comma-separated)
- `--json`: Output in JSON format

**Common Status Values:**
- `open`: Issue is open and not started
- `in_progress`: Currently being worked on
- `blocked`: Cannot proceed due to dependencies
- `review`: Ready for review
- `closed`: Completed or resolved

### bd close
Close an issue.

```bash
bd close <issue-id>
bd close <issue-id> --reason "Implemented successfully"
bd close <issue-id> --json
```

**Options:**
- `--reason <text>`: Provide closure reason or notes
- `--json`: Output in JSON format

## Dependency Management Commands

### bd dep add
Add a dependency relationship between issues.

```bash
bd dep add <from-id> <to-id> --type blocks
bd dep add <from-id> <to-id> --type related
bd dep add <from-id> <to-id> --type parent-child
bd dep add <from-id> <to-id> --type discovered-from
```

**Dependency Types:**

1. **blocks**: Issue A blocks Issue B (B cannot start until A is done)
   ```bash
   bd dep add bd-a3f8 bd-b4c9 --type blocks
   # bd-a3f8 must be completed before bd-b4c9 can start
   ```

2. **related**: Issues are related but not dependent
   ```bash
   bd dep add bd-a3f8 bd-b4c9 --type related
   # Issues are connected but can work independently
   ```

3. **parent-child**: Hierarchical task decomposition
   ```bash
   bd dep add bd-a3f8 bd-b4c9 --type parent-child
   # bd-a3f8 is parent, bd-b4c9 is child
   ```

4. **discovered-from**: Issue found while working on another
   ```bash
   bd dep add bd-b4c9 bd-a3f8 --type discovered-from
   # bd-b4c9 was discovered while working on bd-a3f8
   ```

**Options:**
- `--type <type>`: Dependency type (required)
- `--json`: Output in JSON format

### bd dep rm
Remove a dependency relationship.

```bash
bd dep rm <from-id> <to-id>
bd dep rm <from-id> <to-id> --json
```

### bd dep list
List all dependencies for an issue.

```bash
bd dep list <issue-id>
bd dep list <issue-id> --json
```

Shows:
- Issues this one blocks
- Issues blocking this one
- Related issues
- Parent and children
- Discovery relationships

### bd dep tree
Visualize dependency graph.

```bash
bd dep tree <issue-id>
bd dep tree <issue-id> --json
```

Displays hierarchical view of dependencies and relationships.

## Synchronization Commands

### bd sync
Manually trigger synchronization with git.

```bash
bd sync                    # Immediate flush & import
bd sync --force           # Force sync even if up to date
```

**Automatic Sync Behavior:**
- **On Write**: 5-second debounce before auto-export to `.beads/beads.jsonl`
- **On Read**: Immediate import detection if `.beads/beads.jsonl` changed
- **Git Operations**: Auto-sync on commit, push, pull, merge via git hooks

### Global Flags for Sync Control

```bash
bd --no-auto-flush create "Issue"  # Skip auto-export
bd --no-auto-import list          # Skip auto-import check
```

## Database Management Commands

### bd doctor
Run health check and diagnostics.

```bash
bd doctor
bd doctor --json
bd doctor --fix          # Attempt automatic fixes
```

Checks for:
- Database integrity
- File consistency
- Synchronization status
- Conflict detection

### bd info
Show database metadata and statistics.

```bash
bd info
bd info --json
```

Displays:
- Total issue count
- Issue status breakdown
- Database version
- Storage location
- Last sync time

### bd migrate
Upgrade database to hash-based IDs (v0.20.1+).

```bash
bd migrate                    # Perform migration
bd migrate --dry-run         # Preview without changes
bd migrate --inspect         # Show migration plan
bd migrate --inspect --json  # JSON migration plan
```

**Important**: Migrates from sequential IDs to hash-based IDs for collision resistance in multi-agent workflows.

**Hash-Based ID Progression:**
- 4 characters: 0-500 issues
- 5 characters: 500-1,500 issues
- 6 characters: 1,500+ issues

## Configuration Commands

### Git Merge Driver Setup

```bash
git config merge.beads.driver "bd merge %A %O %A %B"
git config merge.beads.name "bd JSONL merge driver"
echo ".beads/beads.jsonl merge=beads" >> .gitattributes
```

Enables automatic conflict resolution for `.beads/beads.jsonl` during merges.

## Command Output Formats

### Human-Readable Output (Default)
```
bd list
bd-a3f8e9  [open]    Fix login bug                    P2  bug
bd-b4c92d  [closed]  Implement user profile          P3  feature
```

### JSON Output
```bash
bd list --json
```
```json
[
  {
    "id": "bd-a3f8e9",
    "title": "Fix login bug",
    "status": "open",
    "priority": 2,
    "type": "bug",
    "labels": ["backend", "auth"],
    "created": "2025-01-15T10:30:00Z",
    "updated": "2025-01-15T10:30:00Z"
  }
]
```

**Use JSON output for programmatic access** - all commands support `--json` flag.

## Common Command Patterns

### Starting Work Session
```bash
# Check for ready work
bd ready --json | jq '.[0]'

# Review issue details
bd show <issue-id>

# Update status
bd update <issue-id> --status in_progress
```

### During Work
```bash
# Create discovered issue
bd create "Bug found during work" -t bug -p 0 --json

# Link to parent work
bd dep add <new-id> <parent-id> --type discovered-from

# Update progress
bd update <issue-id> --status review
```

### Completing Work
```bash
# Close issue with notes
bd close <issue-id> --reason "Implemented and tested"

# Check if any blocked issues are now ready
bd ready --json

# Sync with git
bd sync
```

### Query Examples with jq

```bash
# Get highest priority ready issue
bd ready --json | jq 'sort_by(.priority) | reverse | .[0]'

# List all open bugs
bd list --json | jq '.[] | select(.type == "bug" and .status == "open")'

# Count issues by status
bd list --json | jq 'group_by(.status) | map({status: .[0].status, count: length})'

# Find issues blocked by specific issue
bd show <issue-id> --json | jq '.blocks[]'
```

## Hierarchical Child IDs

Child issues use dot notation for natural work breakdown:

```
bd-a3f8e9          [epic] Auth System
bd-a3f8e9.1        [task] Design login UI
bd-a3f8e9.2        [task] Backend validation
bd-a3f8e9.3        [epic] Password Reset
bd-a3f8e9.3.1      [task] Email templates
bd-a3f8e9.3.2      [task] Reset flow tests
```

Auto-assigned sequentially within parent namespace. Supports up to 3 levels of nesting.

Create child issues:
```bash
# Create parent
PARENT=$(bd create "Auth System" -t epic --json | jq -r '.id')

# Create children (automatically get .1, .2, etc.)
bd create "Login UI" -t task --parent $PARENT
bd create "Backend validation" -t task --parent $PARENT
```

## Performance Notes

- Batch operations: ~950ms for 1,000 issues
- Local SQLite cache for fast queries
- Git-backed JSONL for version control
- Five-second debounce on writes prevents excessive disk I/O
- Immediate import detection ensures consistency
