---
name: clojure-repl
description: Use when programming Clojure to evaluate code in a running nREPL, discover available REPLs, and leverage REPL-driven development. Guides interactive development workflow using clojure-mcp-light tools.
---

# Clojure REPL Development

Guide for REPL-driven Clojure development using clojure-mcp-light.

## Prerequisites

Install clojure-mcp-light (requires babashka and bbin):

```bash
# Install bbin if not already installed
curl -sL https://raw.githubusercontent.com/babashka/bbin/main/bbin | bb -Sdeps '{:deps {babashka/bbin {:git/url "https://github.com/babashka/bbin" :git/sha "main"}}}' -m babashka.bbin.cli

# Install clojure-mcp-light tools
bbin install https://github.com/bhauman/clojure-mcp-light.git --tag v0.2.0
bbin install https://github.com/bhauman/clojure-mcp-light.git --tag v0.2.0 \
  --as clj-nrepl-eval --main-opts '["-m" "clojure-mcp-light.nrepl-eval"]'
```

## When to Use

Suggest this workflow when:
- Working on Clojure/ClojureScript projects
- User wants to test code interactively
- Debugging or exploring behavior
- Developing incrementally with immediate feedback

## Available Tools

### clj-nrepl-eval

Evaluate Clojure code in a running nREPL server.

**Discover running REPLs:**
```bash
clj-nrepl-eval --discover-ports
```

**Evaluate code:**
```bash
clj-nrepl-eval -p PORT "(+ 1 2 3)"
```

**With timeout for long operations:**
```bash
clj-nrepl-eval -p PORT --timeout 10000 "(slow-operation)"
```

**Check connected sessions:**
```bash
clj-nrepl-eval --connected-ports
```

**Reset session:**
```bash
clj-nrepl-eval -p PORT --reset-session
```

### clj-paren-repair-claude-hook

Automatic delimiter fixing (configured in hooks, runs transparently).

- Fixes mismatched parentheses, brackets, braces
- Applies cljfmt formatting when enabled
- Creates backups before risky edits

## REPL-Driven Development Workflow

### 1. Start a REPL

Ensure an nREPL server is running:

```bash
# Clojure CLI
clj -M:repl  # or however the project starts nREPL

# Leiningen
lein repl :headless

# Babashka
bb nrepl-server 7888
```

### 2. Discover the Port

```bash
clj-nrepl-eval --discover-ports
```

This scans for `.nrepl-port` files and running servers.

### 3. Evaluate Incrementally

When writing Clojure code:

1. **Write a function** - Edit the file
2. **Load it into REPL** - Evaluate the namespace or function
3. **Test interactively** - Try calling it with sample data
4. **Iterate** - Fix issues, re-evaluate

**Example workflow:**

```bash
# Load the namespace
clj-nrepl-eval -p 7888 "(require '[myapp.core :reload])"

# Test a function
clj-nrepl-eval -p 7888 "(myapp.core/process-data {:id 1 :name \"test\"})"

# Check state
clj-nrepl-eval -p 7888 "(keys @myapp.core/state)"
```

### 4. Debugging

**Inspect values:**
```bash
clj-nrepl-eval -p 7888 "(tap> some-value)"
clj-nrepl-eval -p 7888 "(clojure.pprint/pprint large-data)"
```

**Check for errors:**
```bash
clj-nrepl-eval -p 7888 "*e"  # Last exception
```

**Explore namespaces:**
```bash
clj-nrepl-eval -p 7888 "(ns-publics 'myapp.core)"
```

## Best Practices

### Evaluate Early and Often

Don't write large blocks of code before testing. Evaluate:
- Each function as you write it
- Edge cases immediately
- Data transformations step by step

### Use the REPL for Exploration

Before implementing:
```bash
# What does this library function do?
clj-nrepl-eval -p 7888 "(doc clojure.string/split)"

# What's the shape of this data?
clj-nrepl-eval -p 7888 "(keys (first results))"

# Does this regex work?
clj-nrepl-eval -p 7888 "(re-matches #\"...\" test-string)"
```

### Keep State Clean

```bash
# Reload changed namespaces
clj-nrepl-eval -p 7888 "(require '[myapp.core :reload])"

# Or reload all
clj-nrepl-eval -p 7888 "(require '[myapp.core :reload-all])"

# Reset REPL session if things get weird
clj-nrepl-eval -p 7888 --reset-session
```

### Handle Long-Running Operations

```bash
# Set appropriate timeout
clj-nrepl-eval -p 7888 --timeout 30000 "(run-migrations)"
```

## Integration with Claude Code

The hooks in `~/.claude/settings.json` automatically:
- Fix delimiter errors before writing Clojure files
- Apply cljfmt formatting
- Repair malformed code after edits

This means you can write Clojure naturally and trust that parentheses will be balanced.

## Troubleshooting

**No REPL found:**
- Check if nREPL server is running
- Look for `.nrepl-port` file in project root
- Try `clj-nrepl-eval --discover-ports`

**Evaluation timeout:**
- Increase timeout: `--timeout 30000`
- Check if REPL is responsive
- Reset session: `--reset-session`

**Delimiter errors persisting:**
- Check hook is configured in settings.json
- Run manually: `clj-paren-repair-claude-hook --cljfmt`
- Check logs: `clj-paren-repair-claude-hook --log-level debug`
