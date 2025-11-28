---
name: beads-tasks
description: Use when user wants to track tasks in git, manage dependencies between tasks, or needs offline-first task management.
requires:
  tools: [bd]
  skills: []
---

# Beads Task Management

Git-native task tracker stored in `.beads/` directory. Works offline, syncs with git.

## Prerequisites

```bash
command -v bd >/dev/null || { echo "Install: npm install -g @beads/bd"; exit 1; }
```

## Quick Reference

| Command | Purpose |
|---------|---------|
| `bd` | List open tasks |
| `bd add "Task"` | Create task |
| `bd show ID` | View details |
| `bd done ID` | Mark complete |
| `bd dep ID1 ID2` | ID1 depends on ID2 |
| `bd ready` | Show unblocked tasks |
| `bd json` | JSON output for scripts |

## Common Workflows

### View Open Tasks

```bash
bd                    # All open tasks
bd ready              # Ready to work on (unblocked)
bd -a                 # Include done tasks
```

### Create Tasks

```bash
bd add "Implement authentication"
bd add "Write tests" --dep bd-a3f8     # With dependency
bd add "Fix bug" --priority high
```

### Update Tasks

```bash
bd done bd-a3f8              # Complete
bd block bd-a3f8             # Mark blocked
bd edit bd-a3f8 --desc "..."  # Edit description
```

## Parallel Task Execution with Subagents

**When multiple tasks are ready (no dependencies), dispatch parallel subagents:**

```bash
# 1. Get ready tasks
bd ready --json
```

```
# 2. If 3+ independent tasks, dispatch subagents:

Subagent 1:
  Work on task bd-a3f8
  Mark done: bd done bd-a3f8

Subagent 2:
  Work on task bd-b2c4
  Mark done: bd done bd-b2c4

Subagent 3:
  Work on task bd-c5d6
  Mark done: bd done bd-c5d6
```

```bash
# 3. After subagents complete, check for newly unblocked tasks
bd ready
```

**Note:** Only parallelize tasks with no shared dependencies. Check with `bd graph` first.

## Git Integration

Beads syncs automatically:
- `git pull` → imports teammate's tasks
- `git push` → shares your tasks
- Tasks stored in `.beads/*.jsonl`

## Task Structure

```json
{
  "id": "bd-a3f8e9",
  "title": "Implement auth",
  "status": "open",
  "priority": "normal",
  "deps": ["bd-b2c4f1"],
  "created": "2025-01-15T..."
}
```

## Success Criteria

- [ ] `.beads/` directory initialized
- [ ] `bd` lists tasks correctly
- [ ] Task dependencies tracked with `bd dep`

## Related Skills

- `adr-management` - Link tasks to architectural decisions
