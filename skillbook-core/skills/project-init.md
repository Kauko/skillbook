---
name: project-init
description: Use when starting a new project or initializing an existing project with skillbook capabilities. Discovers installed skills, lets user select which to enable, runs init skills in dependency order.
---

# Project Initialization with Skillbook

This skill orchestrates the initialization of projects with selected skills from the skillbook ecosystem.

## When to Use This Skill

Use this skill when:
- Starting a new project and want to set up documentation, architecture, and workflow practices
- Initializing an existing project with skillbook capabilities
- Adding new skillbook skills to an already-initialized project
- The user explicitly asks to "initialize project" or "set up skillbook"

## Initialization Process

### Phase 1: Discovery

1. **Scan for installed skillbook plugins:**
   ```bash
   ls -d ~/.config/claude-code/plugins/skillbook-*/
   ```

2. **Extract available init skills:**
   - Look for skills matching pattern `init-*` or `setup-*` in each plugin's `skills/` directory
   - Read skill YAML frontmatter to get name and description
   - Group skills by their plugin for organized presentation

3. **Common skillbook plugins to discover:**
   - `skillbook-requirements-documentation`: Project vision, stakeholder analysis, requirements documentation
   - `skillbook-architecture-modeling`: C4 diagrams, ADRs, arc42 documentation
   - `skillbook-security-compliance`: Security controls, compliance frameworks
   - `skillbook-clojure-quality`: Code quality, testing, linting for Clojure
   - `skillbook-clojure-libraries`: Clojure library selection and integration
   - `skillbook-git-workflow`: Git branching strategies, commit conventions
   - `skillbook-infrastructure-ops`: Docker setup, deployment practices

### Phase 2: Skill Selection

Use `AskUserQuestion` with `multiSelect: true` to present discovered skills:

```javascript
{
  "questions": [{
    "question": "Which skillbook capabilities do you want to enable for this project?",
    "header": "Skills",
    "multiSelect": true,
    "options": [
      {
        "label": "Obsidian Vault + arc42 Docs",
        "description": "Initialize documentation vault with arc42 architecture documentation structure"
      },
      {
        "label": "Requirements Documentation",
        "description": "Project vision, stakeholder analysis, user stories, and requirements tracking"
      },
      {
        "label": "C4 Architecture Diagrams",
        "description": "Context, Container, Component, and Code diagrams for system architecture"
      },
      {
        "label": "Security & Compliance",
        "description": "Security controls, threat modeling, compliance framework templates"
      },
      {
        "label": "Clojure Quality Tools",
        "description": "clj-kondo, eastwood, cljfmt, and testing setup for Clojure projects"
      },
      {
        "label": "Git Workflow",
        "description": "Branching strategy, commit conventions, PR templates"
      },
      // Add other discovered skills
    ]
  }]
}
```

**Important:** Always present `init-obsidian-vault` (or equivalent documentation foundation) as the first option, as it's a prerequisite for documentation-heavy skills.

### Phase 3: Dependency Resolution

Skills must be initialized in the correct order to handle dependencies:

**Dependency Order:**
1. **Foundation** (must run first):
   - `init-obsidian-vault`: Creates documentation vault structure
   - `init-project-structure`: Creates base project directories

2. **Requirements & Documentation** (depends on vault):
   - `init-project-vision`
   - `init-stakeholder-analysis`
   - `init-requirements-docs`

3. **Architecture Modeling** (depends on vault + requirements):
   - `init-c4-diagrams`
   - `init-arc42-docs`
   - `init-adrs`

4. **Security & Compliance** (depends on architecture):
   - `init-security-controls`
   - `init-compliance-framework`

5. **Technology-Specific** (depends on project structure):
   - `init-clojure-quality`
   - `init-clojure-libraries`

6. **Workflow & Process** (can run anytime):
   - `init-git-workflow`
   - `init-pr-templates`

7. **Infrastructure** (depends on project structure):
   - `init-docker-setup`
   - `init-deployment-docs`

**Dependency Algorithm:**
```
selected_skills = user_selected_skills
ordered_skills = []

# Phase 1: Foundation
for skill in selected_skills:
  if skill.name in ["init-obsidian-vault", "init-project-structure"]:
    ordered_skills.append(skill)

# Phase 2-7: Add skills in dependency order
for phase in [requirements, architecture, security, tech_specific, workflow, infrastructure]:
  for skill in selected_skills:
    if skill.category == phase and skill not in ordered_skills:
      ordered_skills.append(skill)

return ordered_skills
```

### Phase 4: Skill Execution

For each skill in dependency order:

1. **Invoke the skill:**
   ```
   Use the Skill tool to invoke: "skillbook-{plugin}:{skill-name}"
   ```

2. **Monitor execution:**
   - Wait for skill completion before proceeding to next
   - Log any errors or warnings
   - Track which skills succeeded/failed

3. **Handle failures:**
   - If a foundation skill fails, halt and report error
   - If optional skill fails, log warning and continue
   - Offer to retry failed skills at the end

### Phase 5: Manifest Generation

Create `.skillbook/manifest.edn` to track initialized skills:

```clojure
{:project/name "project-name"
 :project/initialized-at "2025-11-27T10:00:00Z"
 :skillbook/version "1.0.0"

 :active-skills
 [{:plugin "skillbook-requirements-documentation"
   :skill "init-obsidian-vault"
   :initialized-at "2025-11-27T10:01:00Z"
   :status :active}

  {:plugin "skillbook-architecture-modeling"
   :skill "init-arc42-docs"
   :initialized-at "2025-11-27T10:02:00Z"
   :status :active
   :depends-on ["init-obsidian-vault"]}

  {:plugin "skillbook-clojure-quality"
   :skill "init-clojure-quality"
   :initialized-at "2025-11-27T10:03:00Z"
   :status :active}]

 :mcp-servers
 [{:name "obsidian-mcp"
   :description "MCP server for Obsidian vault integration"
   :recommended true
   :installed false}

  {:name "clojure-mcp-light"
   :description "Lightweight Clojure REPL and namespace tools"
   :recommended true
   :installed false}]}
```

**Manifest Structure:**
- `:project/*` - Project metadata
- `:active-skills` - Vector of initialized skills with timestamps and dependencies
- `:mcp-servers` - Recommended MCP servers for the selected skills

### Phase 6: MCP Server Recommendations

Based on selected skills, recommend relevant MCP servers:

**Recommendation Logic:**
- If any documentation skill selected → Suggest `obsidian-mcp`
- If any Clojure skill selected → Suggest `clojure-mcp-light`
- If security/compliance selected → Suggest `security-scanner-mcp` (if available)

**Present recommendations:**
```
Project initialized successfully!

Recommended MCP Servers:
1. obsidian-mcp - Enables Claude to read/write Obsidian vault directly
   Installation: Follow instructions at https://github.com/user/obsidian-mcp

2. clojure-mcp-light - Lightweight Clojure REPL integration
   Installation: Follow instructions at https://github.com/user/clojure-mcp-light

Would you like help installing these MCP servers?
```

## Post-Initialization

After successful initialization:

1. **Create summary report:**
   ```
   Skillbook Initialization Complete

   Initialized Skills:
   - init-obsidian-vault: Documentation vault created at ./docs/
   - init-arc42-docs: arc42 templates installed
   - init-clojure-quality: Quality tools configured

   Next Steps:
   1. Review generated documentation structure
   2. Install recommended MCP servers (optional)
   3. Begin using skills: /skillbook-architecture-modeling:create-c4-context

   Manifest: .skillbook/manifest.edn
   ```

2. **Offer next actions:**
   - Start documenting project vision
   - Create first architecture diagram
   - Set up git workflow

## Re-initialization and Updates

If `.skillbook/manifest.edn` already exists:

1. **Read current manifest**
2. **Show currently active skills**
3. **Ask user:**
   - Add new skills?
   - Remove skills?
   - Re-run specific init skills?
   - Update manifest only?

4. **Handle accordingly:**
   - For additions: Run new init skills only
   - For removals: Update manifest (warn about manual cleanup needed)
   - For re-runs: Confirm overwrite, then re-execute
   - For updates: Modify manifest without running skills

## Error Handling

**Common errors:**

1. **No skillbook plugins found:**
   ```
   Error: No skillbook plugins detected in ~/.config/claude-code/plugins/

   To install skillbook plugins:
   1. Clone skillbook repository
   2. Install desired plugins using Claude Code plugin manager
   3. Re-run project-init
   ```

2. **Dependency skill not selected:**
   ```
   Warning: You selected 'init-arc42-docs' but not 'init-obsidian-vault'
   The arc42 skill requires an Obsidian vault to function.

   Would you like to automatically enable 'init-obsidian-vault'? (Yes/No)
   ```

3. **Skill execution failed:**
   ```
   Error: Failed to initialize 'init-clojure-quality'
   Reason: deps.edn not found in project root

   This skill requires a Clojure project with deps.edn
   Continue with remaining skills? (Yes/No)
   ```

## Implementation Notes

**Critical Requirements:**
- Always respect dependency order - never run dependent skills before their prerequisites
- Use `AskUserQuestion` with `multiSelect: true` for skill selection
- Generate manifest after successful initialization
- Handle partial failures gracefully
- Provide clear next steps and documentation references

**Best Practices:**
- Group skills by plugin in selection UI
- Show skill descriptions to help user decide
- Log all operations to `.skillbook/init.log`
- Create backup before re-initialization
- Validate manifest structure after generation

**Performance:**
- Cache plugin discovery results during session
- Run independent skills in parallel when possible (future enhancement)
- Skip manifest regeneration if no changes made
