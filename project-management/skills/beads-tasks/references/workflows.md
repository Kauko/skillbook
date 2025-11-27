# Beads Agent Workflows and Patterns

Comprehensive guide to using Beads for AI agent task management, including workflow patterns, best practices, and integration strategies.

## Core Philosophy

Beads provides **"a memory upgrade for your coding agent"** by solving agent amnesia through:
- Git-backed persistent issue tracking
- Automatic dependency tracking
- Ready-work detection (issues with no blockers)
- Session-to-session context preservation
- Distributed, offline-first architecture

## Agent Onboarding

### First-Time Setup

When encountering a Beads repository for the first time:

```bash
bd onboard
```

This interactive guide provides:
1. Integration instructions specific to the repository
2. Workflow documentation review
3. Instructions to update `AGENTS.md` with Beads workflows
4. Instructions to update `CLAUDE.md` if present
5. Removal of bootstrap instructions

### Quick Tutorial

```bash
bd quickstart
```

Full workflow tutorial covering:
- Issue creation and management
- Dependency tracking
- Finding ready work
- Status updates
- Synchronization

## Session Lifecycle Patterns

### 1. Session Start: Orienting

**Goal**: Understand current state and find actionable work.

```bash
# Ensure database is synced
bd sync

# Find ready work (no blockers)
bd ready --json | jq -r '.[] | "\(.id): \(.title) [P\(.priority)]"'

# Review specific issue
ISSUE_ID="bd-a3f8e9"
bd show $ISSUE_ID

# Visualize dependencies
bd dep tree $ISSUE_ID
```

**Pattern**:
1. Sync database
2. Query ready work
3. Select highest priority issue
4. Review dependencies and context
5. Update status to `in_progress`

### 2. During Work: Tracking Discovery

**Goal**: Capture new issues discovered during implementation.

```bash
# Current work
CURRENT_ISSUE="bd-a3f8e9"
bd update $CURRENT_ISSUE --status in_progress

# Discovered bug while working
NEW_BUG=$(bd create "Memory leak in cache" -t bug -p 1 --json | jq -r '.id')

# Link discovery back to original work
bd dep add $NEW_BUG $CURRENT_ISSUE --type discovered-from

# If bug blocks current work
bd dep add $NEW_BUG $CURRENT_ISSUE --type blocks
bd update $CURRENT_ISSUE --status blocked
```

**Discovery Pattern**:
- File issues immediately when found
- Use `discovered-from` dependency type
- Add `blocks` relationship if it prevents progress
- Update parent issue status if blocked

### 3. Session End: Landing the Plane

**Goal**: Ensure all work is captured and synchronized before session ends.

**Recommended Protocol**:

#### Step 1: File/Update All Issues
```bash
# Review current work state
bd show $CURRENT_ISSUE

# Create issues for any remaining TODOs
grep -r "TODO" src/ | while read line; do
  bd create "TODO: $line" -t task -p 1 -l "todo"
done

# File any bugs or follow-up work discovered
bd create "Need to add error handling to API" -t task -p 2
```

#### Step 2: Quality Gates
```bash
# Run tests
npm test

# If tests fail, file P0 issue
if [ $? -ne 0 ]; then
  bd create "Tests failing after implementation" -t bug -p 4 -l "urgent,tests"
fi

# Run linter
npm run lint

# File any critical issues
```

#### Step 3: Sync Carefully
```bash
# Sync with remote
bd sync

# Review any conflicts
bd doctor

# Commit changes
git add .
git commit -m "Complete work on $CURRENT_ISSUE"
git push
```

#### Step 4: Verify Clean State
```bash
# Check git status
git status

# Verify all issues synced
bd info

# Check ready work
bd ready --json
```

#### Step 5: Choose Next Work
```bash
# Suggest next work with context
NEXT_ISSUE=$(bd ready --json | jq -r 'sort_by(.priority) | reverse | .[0] | .id')
bd show $NEXT_ISSUE

echo "Ready for next session. Recommended work: $NEXT_ISSUE"
```

**Why This Matters**: Prevents common problem where "agents create issues during work but forget to sync them at session end."

## Workflow Patterns

### Pattern: Finding Ready Work

**Purpose**: Locate actionable issues without blockers.

```bash
# Basic ready work query
bd ready --json | jq '.[] | "\(.id): \(.title)"'

# High-priority ready work
bd ready --json | jq '.[] | select(.priority >= 3) | "\(.id): \(.title) [P\(.priority)]"'

# Ready work by label
bd ready --json | jq '.[] | select(.labels[] | contains("backend"))'

# Ready work with full context
bd ready --json | jq -r '.[] | "ID: \(.id)\nTitle: \(.title)\nPriority: \(.priority)\nType: \(.type)\n"'
```

**Best Practice**: Always start work sessions with `bd ready` to ensure you're working on unblocked tasks.

### Pattern: Task Decomposition

**Purpose**: Break large issues into manageable subtasks with clear dependencies.

```bash
# Create epic
EPIC=$(bd create "User Authentication System" -t epic -p 3 --json | jq -r '.id')

# Create subtasks with parent relationship
SCHEMA=$(bd create "Design auth database schema" -t task -p 3 -l "database" --json | jq -r '.id')
bd dep add $SCHEMA $EPIC --type parent-child

API=$(bd create "Implement authentication API" -t task -p 3 -l "backend,api" --json | jq -r '.id')
bd dep add $API $EPIC --type parent-child

UI=$(bd create "Build login UI components" -t task -p 3 -l "frontend" --json | jq -r '.id')
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

### Pattern: Handling Blockers

**Purpose**: Systematically address blocking dependencies.

```bash
# Identify blocked work
BLOCKED_ISSUE="bd-b4c9"
bd show $BLOCKED_ISSUE --json | jq '.blocked_by[]'

# Review blocker details
BLOCKER_ISSUE="bd-a3f8"
bd show $BLOCKER_ISSUE

# Work on blocker first
bd update $BLOCKER_ISSUE --status in_progress

# Complete blocker
bd close $BLOCKER_ISSUE --reason "Implemented and tested"

# Verify blocked issue is now ready
bd ready --json | jq '.[] | select(.id == "bd-b4c9")'
```

**Decision Tree**:
1. Can blocker be resolved quickly? → Work on it now
2. Blocker needs significant work? → Prioritize it, find other ready work
3. Blocker is external? → Mark parent as blocked, document in description

### Pattern: Bug Discovery Workflow

**Purpose**: Capture and triage bugs found during work.

```bash
# Working on feature
FEATURE_ISSUE="bd-a3f8"
bd update $FEATURE_ISSUE --status in_progress

# Discover bug
BUG=$(bd create "Null pointer in user service" \
  -t bug \
  -p 2 \
  -l "backend,bug" \
  -d "Found while testing auth. User service returns null when email is empty." \
  --json | jq -r '.id')

# Link discovery
bd dep add $BUG $FEATURE_ISSUE --type discovered-from

# Triage: Does it block current work?
# Option A: Blocks current work
bd dep add $BUG $FEATURE_ISSUE --type blocks
bd update $FEATURE_ISSUE --status blocked

# Option B: Can work around it
bd dep add $BUG $FEATURE_ISSUE --type related
# Continue with feature work
```

### Pattern: Multi-Agent Coordination

**Purpose**: Multiple agents working on same repository without conflicts.

```bash
# Agent A: Checks out work
AGENT_A_WORK=$(bd ready --json | jq -r '.[0].id')
bd update $AGENT_A_WORK --status in_progress --assignee "agent-a"

# Agent B: Finds different work (Agent A's work no longer shows as ready)
AGENT_B_WORK=$(bd ready --json | jq -r '.[0].id')
bd update $AGENT_B_WORK --status in_progress --assignee "agent-b"

# Periodic sync to share progress
bd sync
git pull
git push

# Hash-based IDs prevent collisions
# Auto-merge handles concurrent edits
```

**Key Feature**: Hash-based IDs (v0.20.1+) eliminate collision issues in concurrent workflows.

### Pattern: Sprint Planning

**Purpose**: Organize work for iteration/sprint.

```bash
# Create sprint epic
SPRINT=$(bd create "Sprint 15: Auth & Profile Features" -t epic -p 3 --json | jq -r '.id')

# Add sprint goals as children
GOAL1=$(bd create "Complete user authentication" -t epic -p 3 -l "sprint15,auth" --json | jq -r '.id')
bd dep add $GOAL1 $SPRINT --type parent-child

GOAL2=$(bd create "Implement user profiles" -t epic -p 3 -l "sprint15,profile" --json | jq -r '.id')
bd dep add $GOAL2 $SPRINT --type parent-child

# Break down goals into tasks
# (use task decomposition pattern)

# Track sprint progress
bd dep tree $SPRINT
bd list --label sprint15 --json | jq 'group_by(.status) | map({status: .[0].status, count: length})'
```

### Pattern: Integration with CI/CD

**Purpose**: Automated checks in deployment pipeline.

```bash
# Pre-deployment check: No blocked deployment tasks
if bd list --label deployment --status blocked --json | jq -e 'length > 0' > /dev/null; then
  echo "ERROR: Blocked deployment tasks exist"
  bd list --label deployment --status blocked
  exit 1
fi

# Pre-deployment check: All critical issues resolved
if bd list --priority 4 --status open --json | jq -e 'length > 0' > /dev/null; then
  echo "WARNING: Critical issues still open"
  bd list --priority 4 --status open
fi

# Post-deployment: Create monitoring task
bd create "Monitor deployment of v2.1.0" \
  -t task \
  -p 3 \
  -l "deployment,monitoring" \
  -d "Watch metrics for 24h after deployment"
```

### Pattern: Code Review Integration

**Purpose**: Create issues from code review feedback.

```bash
# Create issue from PR review
PR_NUMBER="123"
REVIEW_FEEDBACK=$(gh pr view $PR_NUMBER --json reviews --jq '.reviews[].body')

REVIEW_ISSUE=$(bd create "Address code review feedback for PR #$PR_NUMBER" \
  -t task \
  -p 2 \
  -l "code-review" \
  -d "$REVIEW_FEEDBACK" \
  --json | jq -r '.id')

# Link to original feature issue
FEATURE_ISSUE="bd-a3f8"
bd dep add $REVIEW_ISSUE $FEATURE_ISSUE --type discovered-from

# Block merge until addressed
bd dep add $REVIEW_ISSUE $FEATURE_ISSUE --type blocks
```

### Pattern: Technical Debt Tracking

**Purpose**: Capture and prioritize technical debt.

```bash
# Scan for TODOs and FIXMEs
grep -rn "TODO\|FIXME" src/ | while IFS=: read -r file line content; do
  bd create "Tech debt: $content" \
    -t task \
    -p 1 \
    -l "tech-debt" \
    -d "File: $file\nLine: $line\n\n$content"
done

# Create quarterly debt cleanup epic
DEBT_CLEANUP=$(bd create "Q1 Technical Debt Cleanup" -t epic -p 2 -l "tech-debt,q1" --json | jq -r '.id')

# Link existing debt issues
bd list --label tech-debt --json | jq -r '.[] | .id' | while read debt_id; do
  bd dep add $debt_id $DEBT_CLEANUP --type parent-child
done

# Track progress
bd dep tree $DEBT_CLEANUP
```

## Best Practices for Agents

### 1. Always Use JSON Output

```bash
# Good: Programmatic access
bd ready --json | jq -r '.[0].id'

# Avoid: Parsing human-readable output
bd ready | grep -oP 'bd-\w+'
```

**Why**: JSON output is stable and designed for programmatic access.

### 2. Capture Context in Descriptions

```bash
# Good: Rich context
bd create "Optimize user search query" \
  -t task \
  -p 2 \
  -d "Current query takes >5s for users with many connections.
      Profile the query, consider adding indexes on user_connections.user_id.
      Target: <1s for 99th percentile.
      Related to performance issue reported in #456."

# Bad: Vague
bd create "Fix search"
```

### 3. Update Status Regularly

```bash
# Start work
bd update $ISSUE_ID --status in_progress

# Hit blocker
bd update $ISSUE_ID --status blocked

# Ready for review
bd update $ISSUE_ID --status review

# Complete
bd close $ISSUE_ID --reason "Implemented and tested successfully"
```

### 4. Use Blocking Dependencies Wisely

```bash
# Good: Clear blocking relationship
bd create "Write integration tests" -t task
bd create "Deploy to production" -t task
bd dep add $TEST_ID $DEPLOY_ID --type blocks

# Good: Related but not blocking
bd create "Update API documentation" -t task
bd create "Add API endpoint" -t task
bd dep add $DOC_ID $API_ID --type related
```

### 5. Leverage Labels for Organization

```bash
# Multi-dimensional tagging
bd create "Implement payment gateway" \
  -t feature \
  -l "backend,api,payments,security,high-effort"

# Query by multiple dimensions
bd list --json | jq '.[] | select(.labels[] | contains("backend")) | select(.labels[] | contains("security"))'
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

### 7. Use Discovery Tracking

```bash
# Always link discovered work back to source
bd dep add $NEW_ISSUE $CURRENT_ISSUE --type discovered-from
```

This creates audit trail: "Why did we create this issue?"

## Query Patterns with jq

### Find Highest Priority Ready Work
```bash
bd ready --json | jq 'sort_by(.priority) | reverse | .[0]'
```

### Group Issues by Status
```bash
bd list --json | jq 'group_by(.status) | map({status: .[0].status, count: length})'
```

### Find All Bugs by Priority
```bash
bd list --json | jq '.[] | select(.type == "bug") | {id, title, priority} | @json' | jq -s 'sort_by(.priority) | reverse'
```

### Get Issues Modified in Last 24 Hours
```bash
bd list --json | jq --arg date "$(date -u -d '24 hours ago' +%Y-%m-%dT%H:%M:%SZ)" '.[] | select(.updated > $date)'
```

### Find Issues with No Dependencies
```bash
bd list --json | jq '.[] | select(.blocks == [] and .blocked_by == [] and .related == [])'
```

### Get Full Dependency Chain
```bash
# Recursively fetch dependencies (requires scripting)
get_deps() {
  local issue_id=$1
  bd show $issue_id --json | jq -r '.blocked_by[]?' | while read blocker; do
    echo $blocker
    get_deps $blocker
  done
}
```

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
- If blocked: `bd dep add <blocker> <issue> --type blocks`

### Completing Work
- Close issue: `bd close <id> --reason "Details"`
- Sync: `bd sync`
- Find next work: `bd ready`

### Session End Protocol
1. File all remaining issues/TODOs
2. Run quality gates (tests, linters)
3. Sync carefully: `bd sync && git push`
4. Verify clean state
5. Suggest next work

See [Beads documentation](https://github.com/steveyegge/beads) for details.
```

### Stealth Mode for Private Repos

If repository should not expose `.beads/` to other contributors:

```bash
bd init --stealth
```

Stores `.beads/` path in `~/.config/git/ignore`, enabling local issue tracking without affecting repository.

## Troubleshooting

### Issue Not Showing in Ready List

```bash
# Check status
bd show $ISSUE_ID

# Check for blockers
bd show $ISSUE_ID --json | jq '.blocked_by'

# Verify blocker is actually blocking
bd dep list $ISSUE_ID
```

### Sync Conflicts

```bash
# Check for conflicts
bd doctor

# List specific conflicts
bd conflicts  # If command exists

# Resolve by examining both versions
bd show $CONFLICT_ISSUE --json
git show origin/main:.beads/beads.jsonl | grep $CONFLICT_ISSUE
```

### Performance Issues

```bash
# Check database size
bd info

# Consider migration to hash IDs
bd migrate --inspect

# Compact closed issues (if supported)
bd compact --closed --older-than 90d
```

## Summary for AI Agents

**Core Workflow Loop**:
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
