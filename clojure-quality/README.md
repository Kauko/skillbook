# clojure-quality

Linting, formatting, and style conventions for Clojure code with Splint, clj-kondo, cljfmt.

## Skills

### splint-linting.md
**Use when:** Setting up or running Splint for Clojure idiom linting.

Guides you through:
- Installing Splint
- Creating `.splint.edn` configuration
- Setting up `scripts/lint-splint.sh`
- Configuring optional git pre-commit hooks
- Understanding and fixing common Splint issues

### clj-kondo-linting.md
**Use when:** Setting up or running clj-kondo for static analysis.

Guides you through:
- Installing clj-kondo
- Creating `.clj-kondo/config.edn`
- Auto-importing library configurations
- Setting up `scripts/lint-kondo.sh`
- Configuring optional git pre-commit hooks
- Integration with clojure-mcp-light if available
- Understanding and fixing common linting issues

### cljfmt-formatting.md
**Use when:** Setting up or running cljfmt for code formatting.

Guides you through:
- Installing cljfmt as a Clojure CLI tool
- Creating `.cljfmt.edn` aligned with Clojure Style Guide
- Setting up `scripts/fmt-check.sh` and `scripts/fmt-fix.sh`
- Configuring optional git pre-commit hooks (check-only mode)
- Custom indentation rules for common macros
- Editor integration

### clojure-style.md
**Use when:** Writing or reviewing Clojure code.

Enforces coding conventions from:
- Stuart Sierra's Do's and Don'ts
- Clojure Style Guide
- Custom Metosin conventions

Key rules:
- Naming conventions (kebab-case, predicates with `?`, etc.)
- Code organization (one namespace per file, spacing)
- Idiomatic patterns (threading macros, `seq` for empty tests)
- **ALWAYS use fallback with `get`** (Metosin convention)
- Avoid lazy side effects
- Use polymorphism for consistent interfaces

## Installation

Add to your Claude Code configuration:

```bash
# Clone or link this plugin
cd ~/.claude/plugins
ln -s /path/to/skillbook/clojure-quality .
```

Or use the plugin via skillbook repository.

## Usage

Claude Code automatically applies these skills when:
- You ask to set up linting or formatting
- You write or edit Clojure code
- You request code reviews

You can also explicitly invoke skills:
```
Please set up clj-kondo for this project
Can you format this Clojure code?
Review this code for style issues
```

## Tools Setup

Each skill guides you through installing the necessary tools:

- **Splint:** `brew install noahtheduke/tap/splint`
- **clj-kondo:** `brew install borkdude/brew/clj-kondo`
- **cljfmt:** `clojure -Ttools install io.github.weavejester/cljfmt '{:git/tag "0.12.0"}' :as cljfmt`

## Git Hooks

All skills provide optional git pre-commit hooks that are **disabled by default**. This avoids disrupting existing workflows. To enable, uncomment the relevant sections in `.git/hooks/pre-commit`.

## CI Integration

Each skill includes examples for GitHub Actions and GitLab CI to enforce quality checks in your pipeline.

## Author

**Teemu Kaukoranta**
teemu.kaukoranta@metosin.fi

## License

MIT

## Repository

https://github.com/Kauko/skillbook
