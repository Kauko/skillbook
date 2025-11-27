---
name: beads-tasks
description: Use when managing project tasks, tracking work items, or when user asks about what to work on next. Provides git-native task management with Beads - dependency tracking, ready work detection.
---

# Beads Task Management for AI Agents

## What is Beads?

Beads is a distributed, git-native issue tracker designed for AI agents and developers. Unlike traditional issue trackers, Beads:

- Stores tasks as JSONL files in `.beads/` directory within your repository
- Integrates seamlessly with git workflow (auto-syncs on commit/push/pull)
- Works offline-first with distributed task tracking
- Provides dependency tracking and automatic ready-work detection
- Outputs JSON for programmatic access by AI agents
- Requires no external services or databases

Repository: https://github.com/steveyegge/beads

## Installation Check and Setup

Before using Beads commands, ALWAYS check if Beads is installed:

```bash
which bd
```

### If Beads is Not Installed

Guide the user to install Beads:

**macOS/Linux:**
```bash
# Clone and install from source
git clone https://github.com/steveyegge/beads.git
cd beads
make install  # or follow installation instructions in README
```

**Alternative (if available via package manager):**
```bash
# Check if available via Homebrew or other package managers
brew install beads  # May not be available yet
```

After installation, verify:
```bash
bd --version
```

### Initialize Beads in Repository

If `.beads/` directory doesn't exist:
```bash
bd init
```

This creates the `.beads/` directory structure for task storage.

## Core Commands

### Adding Tasks

```bash
# Add a simple task
bd add "Implement user authentication"

# Add task with description
bd add "Fix login bug" --description "Users can't login with special characters in password"

# Add task with tags
bd add "Update dependencies" --tags security,maintenance

# Add task with priority
bd add "Critical database migration" --priority high
```

### Listing Tasks

```bash
# List all tasks
bd list

# List tasks with JSON output (for programmatic access)
bd list --json

# List only open tasks
bd list --status open

# List tasks by tag
bd list --tag security

# List tasks by priority
bd list --priority high
```

### Finding Ready Work

The most important command for agents - finds tasks that have no blockers:

```bash
# Show tasks ready to work on
bd ready

# Ready tasks as JSON (recommended for agents)
bd ready --json

# Ready tasks with specific tag
bd ready --tag feature
```

**When to use `bd ready`:**
- User asks "what should I work on next?"
- Starting a new work session
- After completing a task to find the next one
- Checking if blocked tasks have become unblocked

### Completing Tasks

```bash
# Mark task as done by ID
bd done <task-id>

# Mark task done with completion note
bd done <task-id> --note "Fixed by implementing OAuth2 flow"

# Close task without marking as done
bd close <task-id>
```

### Managing Dependencies

Beads supports four types of relationships:

#### 1. Blocks Relationship
Task A blocks Task B (B cannot start until A is done):

```bash
# Task 123 blocks task 456
bd block 123 456

# Check what tasks are blocked
bd list --blocked

# Check what tasks are blocking others
bd list --blocking
```

#### 2. Related Relationship
Tasks are related but not dependent:

```bash
# Link related tasks
bd relate 123 456

# Find related tasks
bd show 123  # Shows relationships
```

#### 3. Parent-Child Relationship
Hierarchical task organization:

```bash
# Make task 123 a child of task 456
bd child 123 456

# List children of a task
bd children 456

# Show task tree
bd tree 456
```

#### 4. Discovered-From Relationship
Track which task led to discovering another:

```bash
# Task 789 was discovered while working on task 123
bd discover 789 123

# Show discovery chain
bd show 789
```

### Viewing Task Details

```bash
# Show full task details
bd show <task-id>

# Show task with JSON output
bd show <task-id> --json

# Show task history
bd history <task-id>
```

### Updating Tasks

```bash
# Update task description
bd update <task-id> --description "New description"

# Update task status
bd update <task-id> --status in-progress

# Add tags to existing task
bd tag <task-id> bug,urgent

# Change priority
bd update <task-id> --priority critical
```

## Git Integration

Beads automatically syncs with git operations:

### Auto-sync Behavior

```bash
# Tasks sync automatically on:
git commit   # Commits .beads/ changes
git push     # Pushes task updates
git pull     # Pulls task updates from remote
git merge    # Merges task changes
```

### Manual Sync

```bash
# Force sync with remote
bd sync

# Check sync status
bd status
```

### Handling Conflicts

If task conflicts occur during merge:

```bash
# List conflicts
bd conflicts

# Resolve conflict by choosing version
bd resolve <task-id> --ours   # Keep local version
bd resolve <task-id> --theirs # Keep remote version
bd resolve <task-id> --merge  # Attempt automatic merge
```

## Best Practices for AI Agent Task Management

### 1. Always Check for Ready Work First

```bash
# Start work session by checking ready tasks
bd ready --json | jq '.[] | {id, title, priority}'
```

### 2. Use Blocking Relationships Wisely

```bash
# Example: Test task blocks deployment task
bd add "Write integration tests" --tag testing
bd add "Deploy to production" --tag deployment
bd block <test-task-id> <deploy-task-id>
```

### 3. Track Task Discovery

When you discover new tasks while working:

```bash
# Currently working on task 123, discovered new issue
CURRENT_TASK=123
NEW_TASK=$(bd add "Fix memory leak in cache" --json | jq -r '.id')
bd discover $NEW_TASK $CURRENT_TASK
```

### 4. Use JSON Output for Programmatic Access

```bash
# Get ready tasks and parse with jq
READY_TASKS=$(bd ready --json)
echo $READY_TASKS | jq -r '.[] | "\(.id): \(.title)"'

# Filter tasks by criteria
bd list --json | jq '.[] | select(.priority == "high" and .status == "open")'
```

### 5. Organize with Tags and Hierarchy

```bash
# Add feature task with subtasks
FEATURE_ID=$(bd add "User authentication system" --tag feature --json | jq -r '.id')
bd add "Design auth database schema" --tag feature,database --json | jq -r '.id' | xargs -I {} bd child {} $FEATURE_ID
bd add "Implement login endpoint" --tag feature,api --json | jq -r '.id' | xargs -I {} bd child {} $FEATURE_ID
bd add "Add auth middleware" --tag feature,api --json | jq -r '.id' | xargs -I {} bd child {} $FEATURE_ID
```

### 6. Update Task Status Regularly

```bash
# When starting work
bd update <task-id> --status in-progress

# When blocked
bd update <task-id> --status blocked

# When complete
bd done <task-id> --note "Implementation details or outcome"
```

### 7. Provide Context in Descriptions

```bash
# Good: Detailed task with context
bd add "Optimize database query in user search" \
  --description "Search queries taking >5s for users with many connections. Profile query, add indexes, consider caching." \
  --tag performance,database \
  --priority high

# Less helpful: Vague task
bd add "Fix search"
```

## Common Workflows

### Starting a Work Session

```bash
# 1. Sync with remote
bd sync

# 2. Check what's ready
bd ready --json | jq -r '.[] | "\(.id): \(.title) [\(.priority)]"'

# 3. Pick highest priority task and mark in-progress
TASK_ID=<selected-task-id>
bd update $TASK_ID --status in-progress
bd show $TASK_ID
```

### Completing a Task

```bash
# 1. Mark task done
bd done $TASK_ID --note "Implemented solution description"

# 2. Check if any blocked tasks are now ready
bd ready --json

# 3. Commit and sync
git add .
git commit -m "Complete task $TASK_ID"
git push
```

### Handling Blockers

```bash
# 1. Identify blocker
bd show <blocked-task-id>

# 2. Check if blocker can be resolved
bd show <blocker-task-id>

# 3. If blocker needs work, focus on it first
bd update <blocker-task-id> --status in-progress

# 4. After completing blocker, verify blocked task is ready
bd done <blocker-task-id>
bd ready | grep <blocked-task-id>
```

### Breaking Down Large Tasks

```bash
# 1. Create parent task
PARENT_ID=$(bd add "Implement user profile feature" --tag feature --json | jq -r '.id')

# 2. Add subtasks with dependencies
SCHEMA_ID=$(bd add "Create profile database schema" --tag database --json | jq -r '.id')
bd child $SCHEMA_ID $PARENT_ID

API_ID=$(bd add "Create profile API endpoints" --tag api --json | jq -r '.id')
bd child $API_ID $PARENT_ID
bd block $SCHEMA_ID $API_ID  # API depends on schema

UI_ID=$(bd add "Build profile UI components" --tag frontend --json | jq -r '.id')
bd child $UI_ID $PARENT_ID
bd block $API_ID $UI_ID  # UI depends on API

# 3. View task tree
bd tree $PARENT_ID
```

## Querying Task Status

### Find Specific Tasks

```bash
# Tasks I'm currently working on
bd list --status in-progress --json

# High priority tasks that are ready
bd ready --json | jq '.[] | select(.priority == "high")'

# All blocked tasks
bd list --blocked --json

# Tasks discovered from a specific task
bd show <task-id> --json | jq '.discovered'

# Recent tasks
bd list --json | jq 'sort_by(.created) | reverse | .[:5]'
```

### Generate Reports

```bash
# Tasks by status
bd list --json | jq 'group_by(.status) | map({status: .[0].status, count: length})'

# Tasks by priority
bd list --json | jq 'group_by(.priority) | map({priority: .[0].priority, count: length})'

# Tasks by tag
bd list --json | jq '[.[].tags // []] | flatten | group_by(.) | map({tag: .[0], count: length})'
```

## Error Handling

### Common Issues

**Issue: `bd: command not found`**
- Solution: Install Beads (see Installation Check section)

**Issue: `Not a beads repository`**
- Solution: Run `bd init` in repository root

**Issue: `Task not found`**
- Solution: Verify task ID with `bd list --json | jq '.[] | .id'`

**Issue: Merge conflicts in `.beads/`**
- Solution: Use `bd conflicts` and `bd resolve`

**Issue: Task stuck in blocked state**
- Solution: Check blocking tasks with `bd show <task-id>` and resolve blockers

## Integration Tips

### With Git Workflow

```bash
# Create task from commit message
git log -1 --pretty=%B | xargs -I {} bd add "Follow up: {}"

# Reference tasks in commits
git commit -m "Implement feature (beads:#123)"

# Create tasks from grep results
grep -r "TODO" src/ | while read line; do
  bd add "TODO: $line" --tag todo
done
```

### With CI/CD

```bash
# Check for blocking tasks before deployment
if bd list --tag deployment --blocked --json | jq -e 'length > 0' > /dev/null; then
  echo "Cannot deploy: blocked deployment tasks exist"
  exit 1
fi
```

### With Code Review

```bash
# Create task for review feedback
bd add "Address code review feedback for PR #123" \
  --description "$(gh pr view 123 --json reviews --jq '.reviews[].body')" \
  --tag code-review
```

## Summary for AI Agents

When managing tasks:

1. **Check installation**: Always verify `bd` is available
2. **Use `bd ready`**: Find actionable work without blockers
3. **Prefer JSON output**: Use `--json` flag for programmatic access
4. **Track dependencies**: Use `bd block` to prevent working on tasks out of order
5. **Update status**: Keep task states current for accurate ready work detection
6. **Document completion**: Use `--note` when marking tasks done
7. **Discover new work**: Use `bd discover` to track task relationships
8. **Query intelligently**: Combine `--json` output with `jq` for complex queries
9. **Sync regularly**: Beads auto-syncs with git, but manual `bd sync` ensures freshness
10. **Provide context**: Rich descriptions help future agents (and humans) understand tasks

The key advantage of Beads for AI agents: **dependency-aware task management that lives in git**, enabling distributed, offline-first collaboration with automatic conflict resolution and ready-work detection.
