---
name: beads-tasks
description: Use throughout ALL workflow phases to track tasks - from planning through verification. Beads is the primary task tracker for skillbook projects.
requires:
  tools: [bd]
  skills: []
---

# Beads Task Management

Git-native task tracker stored in `.beads/` directory. Works offline, syncs with git.

**CRITICAL: Use beads in EVERY workflow phase, not just development.**

Beads tasks create a traceable record from inception to completion. Every phase produces tasks.

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

## Phase-Based Task Creation

**Create beads tasks as you enter each workflow phase:**

### 1. Plan Phase
```bash
bd add "Brainstorm: <feature-name>"
bd add "Write design document: <feature-name>"
bd add "Write implementation plan: <feature-name>"
```

### 2. Document Phase
```bash
bd add "Write ADR: <decision-title>" --dep <plan-task-id>
bd add "Update arc42 section: <section-name>"
bd add "Define quality requirements: <feature-name>"
```

### 3. Model Phase
```bash
bd add "Create/update architecture model" --dep <adr-task-id>
bd add "Generate C4 diagrams"
bd add "Define Malli schemas: <domain>"
bd add "Create threat model" --priority high  # if security-relevant
bd add "Create UI mockup: <component>"        # if UI work
```

### 4. Test Phase
```bash
bd add "Write failing tests: <component>" --dep <schema-task-id>
bd add "Review test design"
```

### 5. Develop Phase
```bash
bd add "Implement: <component>" --dep <test-task-id>
bd add "Add to component library" --dep <implement-task-id>  # if UI
bd add "Run quality checks"
bd add "Code review: <component>"
```

### 6. Verify Phase
```bash
bd add "Run Playwright tests" --dep <implement-task-id>
bd add "Get user approval: <feature-name>"
bd add "Final code review before merge"
```

## Common Commands

### View Tasks

```bash
bd                    # All open tasks
bd ready              # Ready to work on (unblocked)
bd -a                 # Include done tasks
```

### Create Tasks

```bash
bd add "Task description"
bd add "Task" --dep bd-a3f8     # With dependency
bd add "Task" --priority high
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

## Linking Requirements

When creating tasks:
- Link to ADRs: `Decision: [[decisions/0005-feature-approach]]`
- Link to specs: `Spec: [[specifications/feature.feature]]`
- Link to components: `Component: [[components/button]]`
- Link to arc42 sections: `Docs: [[arc42/06-runtime#scenario]]`

## Related Skills

- `adr-management` - Link tasks to architectural decisions
