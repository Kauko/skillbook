Initialize a new skillbook project with full workflow enforcement.

Dispatch the `skillbook:project-init` agent to:

1. **Pre-flight check** - Verify all required tools and MCPs are installed
2. **Brainstorm** - Use superpowers:brainstorming to clarify what we're building
3. **Initialize** - Create vault structure, CLAUDE.md, tool configs
4. **Report** - Summarize what was created and any limitations

After initialization, use `skillbook:clojure-developer` agent for feature implementation.
