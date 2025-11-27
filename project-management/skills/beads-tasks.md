---
name: beads-tasks
description: Use when managing project tasks, tracking work items, or when user asks about what to work on next. Provides git-native task management with Beads - dependency tracking, ready work detection.
---

# Beads Task Management for AI Agents

## What is Beads?

Beads is a distributed, git-native issue tracker designed for AI agents and developers. It provides **"a memory upgrade for your coding agent"** by maintaining project context across sessions through git-backed storage.

**Key Features:**
- Stores tasks as JSONL files in `.beads/` directory within your repository
- Integrates seamlessly with git workflow (auto-syncs on commit/push/pull)
- Works offline-first with distributed task tracking
- Provides dependency tracking and automatic ready-work detection
- Outputs JSON for programmatic access by AI agents
- Requires no external services or databases

**Repository**: https://github.com/steveyegge/beads

**Hash-Based IDs** (v0.20.1+): Uses progressive-length hash IDs (bd-a3f8e9) instead of sequential numbers to eliminate collision issues in multi-agent/multi-branch workflows.

## Installation Check and Setup

Before using Beads commands, ALWAYS check if Beads is installed:

```bash
which bd
```

### If Beads is Not Installed

Guide the user to install Beads:

**macOS/Linux (Quick Install):**
```bash
curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/scripts/install.sh | bash
```

**npm:**
```bash
npm install -g @beads/bd
```

**Homebrew:**
```bash
brew tap steveyegge/beads
brew install bd
```

After installation, verify:
```bash
bd --version
```

### Initialize Beads in Repository

If `.beads/` directory doesn't exist:

```bash
bd init                    # Standard workflow
bd init --quiet            # Non-interactive (for agents)
bd init --stealth          # Local-only, hidden from repo
```

**For first-time users, run interactive onboarding:**
```bash
bd onboard
```

## Core Agent Workflow

### Session Start: Orienting

**Goal**: Understand current state and find actionable work.

```bash
# 1. Sync database with git
bd sync

# 2. Find ready work (no blockers)
bd ready --json | jq -r '.[] | "\(.id): \(.title) [P\(.priority)]"'

# 3. Review specific issue
bd show <issue-id>

# 4. Visualize dependencies
bd dep tree <issue-id>

# 5. Start work
bd update <issue-id> --status in_progress
```

**Key Command**: `bd ready` finds issues with no blocking dependencies.

### During Work: Tracking Discovery

**Goal**: Capture new issues discovered during implementation.

```bash
# File discovered bug
NEW_BUG=$(bd create "Memory leak in cache" -t bug -p 1 --json | jq -r '.id')

# Link discovery back to current work
CURRENT_ISSUE="bd-a3f8e9"
bd dep add $NEW_BUG $CURRENT_ISSUE --type discovered-from

# If bug blocks current work
bd dep add $NEW_BUG $CURRENT_ISSUE --type blocks
bd update $CURRENT_ISSUE --status blocked
```

**Discovery Types:**
- `discovered-from`: Issue found while working on another
- `blocks`: Issue prevents progress on another
- `related`: Connected but not blocking
- `parent-child`: Hierarchical decomposition

### Session End: Landing the Plane

**CRITICAL PROTOCOL**: Ensure all work is captured before session ends.

```bash
# 1. File remaining issues and TODOs
bd create "Add error handling to API" -t task -p 2

# 2. Run quality gates
npm test
if [ $? -ne 0 ]; then
  bd create "Tests failing after implementation" -t bug -p 4 -l "urgent"
fi

# 3. Sync carefully
bd sync
git add .
git commit -m "Complete work on issue"
git push

# 4. Verify clean state
git status
bd info

# 5. Suggest next work
NEXT=$(bd ready --json | jq -r 'sort_by(.priority) | reverse | .[0].id')
bd show $NEXT
echo "Ready for next session. Recommended: $NEXT"
```

**Why This Matters**: Prevents common problem where agents create issues during work but forget to sync them at session end.

## Essential Commands

### Finding Work

```bash
# Show ready issues (no blockers)
bd ready --json

# High-priority ready work
bd ready --json | jq '.[] | select(.priority >= 3)'

# Ready work by label
bd ready --json | jq '.[] | select(.labels[] | contains("backend"))'
```

### Creating Issues

```bash
# Basic issue
bd create "Issue title"

# With full context (recommended)
bd create "Optimize user search query" \
  -t task \
  -p 2 \
  -l "backend,performance" \
  -d "Current query takes >5s for users with many connections. Profile query, add indexes." \
  --json
```

**Priority Levels**: 0 (lowest) to 4 (critical)
**Common Types**: bug, feature, task, epic, chore, doc

### Managing Dependencies

```bash
# Issue A blocks Issue B (B cannot start until A is done)
bd dep add <issue-a-id> <issue-b-id> --type blocks

# Issues are related but not dependent
bd dep add <issue-a-id> <issue-b-id> --type related

# Hierarchical task organization
bd dep add <child-id> <parent-id> --type parent-child

# Track discovery chain
bd dep add <new-issue-id> <source-issue-id> --type discovered-from

# Remove dependency
bd dep rm <from-id> <to-id>

# List dependencies
bd dep list <issue-id>

# Visualize dependency tree
bd dep tree <issue-id>
```

### Updating Status

```bash
# Update issue status
bd update <issue-id> --status in_progress
bd update <issue-id> --status blocked
bd update <issue-id> --status review

# Complete issue
bd close <issue-id> --reason "Implemented and tested successfully"

# Update other fields
bd update <issue-id> --priority 4
bd update <issue-id> --labels "urgent,backend"
```

**Common Statuses**: open, in_progress, blocked, review, closed

### Querying Issues

```bash
# Show issue details
bd show <issue-id>
bd show <issue-id> --json

# List all issues
bd list --json

# Filter by status
bd list --status open --json

# Filter by type
bd list --type bug --json

# Filter by label
bd list --label backend --json
```

## Common Workflow Patterns

### Task Decomposition

Break large issues into manageable subtasks:

```bash
# Create epic
EPIC=$(bd create "User Authentication System" -t epic -p 3 --json | jq -r '.id')

# Create subtasks with parent relationship
SCHEMA=$(bd create "Design auth database schema" -t task -p 3 --json | jq -r '.id')
bd dep add $SCHEMA $EPIC --type parent-child

API=$(bd create "Implement authentication API" -t task -p 3 --json | jq -r '.id')
bd dep add $API $EPIC --type parent-child

UI=$(bd create "Build login UI components" -t task -p 3 --json | jq -r '.id')
bd dep add $UI $EPIC --type parent-child

# Add sequential dependencies
bd dep add $SCHEMA $API --type blocks    # API needs schema first
bd dep add $API $UI --type blocks        # UI needs API first

# View task tree
bd dep tree $EPIC
```

**Hierarchical IDs** (automatic):
```
bd-a3f8e9          [epic] User Authentication System
bd-a3f8e9.1        [task] Design auth database schema
bd-a3f8e9.2        [task] Implement authentication API
bd-a3f8e9.3        [task] Build login UI components
```

### Handling Blockers

```bash
# Identify blocked work
bd show <blocked-issue-id> --json | jq '.blocked_by[]'

# Review blocker details
bd show <blocker-issue-id>

# Work on blocker first
bd update <blocker-issue-id> --status in_progress

# Complete blocker
bd close <blocker-issue-id> --reason "Resolved"

# Verify blocked issue is now ready
bd ready --json | jq '.[] | select(.id == "<blocked-issue-id>")'
```

### Multi-Agent Coordination

Hash-based IDs enable collision-free concurrent work:

```bash
# Agent A checks out work
AGENT_A_WORK=$(bd ready --json | jq -r '.[0].id')
bd update $AGENT_A_WORK --status in_progress --assignee "agent-a"

# Agent B finds different work (Agent A's work no longer ready)
AGENT_B_WORK=$(bd ready --json | jq -r '.[0].id')
bd update $AGENT_B_WORK --status in_progress --assignee "agent-b"

# Periodic sync to share progress
bd sync
git pull
git push
```

### CI/CD Integration

```bash
# Pre-deployment check: No blocked deployment tasks
if bd list --label deployment --status blocked --json | jq -e 'length > 0' > /dev/null; then
  echo "ERROR: Blocked deployment tasks exist"
  exit 1
fi

# Pre-deployment check: All critical issues resolved
if bd list --priority 4 --status open --json | jq -e 'length > 0' > /dev/null; then
  echo "WARNING: Critical issues still open"
fi
```

## Useful Query Patterns

### Find Highest Priority Ready Work
```bash
bd ready --json | jq 'sort_by(.priority) | reverse | .[0]'
```

### Group Issues by Status
```bash
bd list --json | jq 'group_by(.status) | map({status: .[0].status, count: length})'
```

### Find All Open Bugs by Priority
```bash
bd list --json | jq '.[] | select(.type == "bug" and .status == "open")' | jq -s 'sort_by(.priority) | reverse'
```

### Get Issues Modified Recently
```bash
# Last 24 hours
bd list --json | jq --arg date "$(date -u -d '24 hours ago' +%Y-%m-%dT%H:%M:%SZ)" '.[] | select(.updated > $date)'
```

### Find Issues with No Dependencies
```bash
bd list --json | jq '.[] | select(.blocks == [] and .blocked_by == [] and .related == [])'
```

## Best Practices for AI Agents

### 1. Always Use JSON Output

```bash
# Good: Programmatic access
bd ready --json | jq -r '.[0].id'

# Avoid: Parsing human-readable output
bd ready | grep -oP 'bd-\w+'
```

### 2. Capture Rich Context

```bash
# Good: Detailed description
bd create "Optimize user search query" \
  -d "Current query takes >5s for users with many connections.
      Profile the query, consider adding indexes on user_connections.user_id.
      Target: <1s for 99th percentile."

# Bad: Vague
bd create "Fix search"
```

### 3. Track Discovery Chain

Always link discovered work back to source:

```bash
bd dep add $NEW_ISSUE $CURRENT_ISSUE --type discovered-from
```

Creates audit trail: "Why did we create this issue?"

### 4. Use Blocking Dependencies Correctly

```bash
# Good: Clear blocking relationship
bd dep add $TEST_ID $DEPLOY_ID --type blocks

# Good: Related but not blocking
bd dep add $DOC_ID $API_ID --type related
```

### 5. Update Status Regularly

```bash
# Start work
bd update $ISSUE_ID --status in_progress

# Hit blocker
bd update $ISSUE_ID --status blocked

# Complete
bd close $ISSUE_ID --reason "Implemented and tested"
```

### 6. Sync Before and After Work

```bash
# Session start
bd sync

# Work on issues...

# Session end
bd sync
git add .
git commit -m "Update issues"
git push
```

### 7. Use Labels for Organization

```bash
# Multi-dimensional tagging
bd create "Implement payment gateway" \
  -l "backend,api,payments,security,high-effort"

# Query by multiple dimensions
bd list --json | jq '.[] | select(.labels[] | contains("backend")) | select(.labels[] | contains("security"))'
```

## Git Integration

Beads automatically syncs with git operations:

```bash
# Auto-sync on:
git commit   # Commits .beads/ changes
git push     # Pushes task updates
git pull     # Pulls task updates from remote
git merge    # Merges task changes

# Manual sync
bd sync

# Check sync status
bd info
```

### Git Merge Driver (Recommended)

Enable automatic conflict resolution:

```bash
git config merge.beads.driver "bd merge %A %O %A %B"
git config merge.beads.name "bd JSONL merge driver"
echo ".beads/beads.jsonl merge=beads" >> .gitattributes
```

## Database Management

```bash
# Health check and diagnostics
bd doctor

# Database metadata and statistics
bd info

# Migrate to hash-based IDs (if on older version)
bd migrate --dry-run    # Preview migration
bd migrate              # Perform migration
```

## Error Handling

### Common Issues

**Issue: `bd: command not found`**
- Solution: Install Beads (see Installation section)

**Issue: `Not a beads repository`**
- Solution: Run `bd init` in repository root

**Issue: Task not found**
- Solution: Verify task ID with `bd list --json | jq '.[] | .id'`

**Issue: Task stuck in blocked state**
- Solution: Check blocking tasks with `bd show <task-id>` and resolve blockers

## Repository Configuration

### AGENTS.md Template

Create or update `AGENTS.md` in repository root:

```markdown
# Agent Instructions

## BEFORE ANYTHING ELSE

Run `bd onboard` at session start and follow instructions.

## Task Management with Beads

This project uses Beads for distributed issue tracking.

### Finding Work
- Use `bd ready` to find issues with no blockers
- Review issue with `bd show <issue-id>`
- Visualize dependencies with `bd dep tree <issue-id>`

### During Work
- Update status: `bd update <id> --status in_progress`
- File discovered issues: `bd create "Issue" -t bug`
- Link discoveries: `bd dep add <new> <parent> --type discovered-from`

### Completing Work
- Close issue: `bd close <id> --reason "Details"`
- Sync: `bd sync`
- Find next work: `bd ready`

### Session End Protocol
1. File all remaining issues/TODOs
2. Run quality gates (tests, linters)
3. Sync: `bd sync && git push`
4. Verify clean state
5. Suggest next work
```

## Summary for AI Agents

**Core Loop**:
1. **Start**: `bd sync` → `bd ready --json` → Select work
2. **Work**: Update status, file discoveries, link dependencies
3. **End**: Close work, sync, suggest next

**Key Commands**:
- `bd ready --json`: Find actionable work
- `bd create --json`: Capture new issues
- `bd dep add`: Link dependencies
- `bd update --status`: Track progress
- `bd close --reason`: Complete work
- `bd sync`: Synchronize with git

**Critical Practices**:
- Use JSON output for programmatic access
- Capture context in descriptions
- Link discovered work with `discovered-from`
- Use blocking dependencies correctly
- Sync at session start and end
- Run "landing the plane" protocol before ending session

**Key Advantage**: Beads provides **dependency-aware task management that lives in git**, enabling distributed, offline-first collaboration with automatic conflict resolution and ready-work detection. Perfect for AI agents needing persistent memory across sessions.

## Additional Resources

For comprehensive details, see:
- **[commands.md](references/commands.md)**: Complete command reference with all options and syntax
- **[workflows.md](references/workflows.md)**: Detailed workflow patterns, query examples, and integration strategies
- **[Official Repository](https://github.com/steveyegge/beads)**: Source code and latest updates
