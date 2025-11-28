---
name: clojure-style
description: Use when WRITING new Clojure code or REVIEWING existing code. Apply these conventions to all Clojure code you generate. This is prerequisite knowledge for quality-check.
requires:
  tools: []
  skills: []
skip_when:
  - Only running automated tools (use quality-check instead)
  - Working on non-Clojure files
  - User explicitly requests a different style (document deviation)
---

# Clojure Style Guide

Apply these rules when writing Clojure. When reviewing, cite specific rules.

## Quick Reference

### Naming

| Type | Convention | Example |
|------|------------|---------|
| Functions/vars | `kebab-case` | `calculate-total` |
| Predicates | end with `?` | `valid?`, `empty?` |
| Unsafe/side-effects | end with `!` | `swap!`, `send!` |
| Conversion | `x->y` | `map->vec` |
| Private | leading `-` or `^:private` | `(defn- helper ...)` |
| Constants | `*earmuffs*` for dynamic, otherwise normal | `default-timeout` |
| Protocols | `PascalCase` | `Lifecycle`, `Component` |

### Structure

```clojure
;; Namespace: require over use, explicit :as aliases
(ns myapp.core
  (:require
   [clojure.string :as str]
   [myapp.db :as db]))

;; Threading: -> for objects, ->> for collections
(-> user :address :city str/upper-case)
(->> items (filter active?) (map :name))

;; Let: prefer descriptive intermediate bindings
(let [valid-items (filter valid? items)
      total (reduce + (map :price valid-items))]
  {:items valid-items :total total})
```

### Key Rules

1. **Pure functions over side effects** - isolate I/O at boundaries
2. **Destructuring** - use it, but keep readable
3. **Small functions** - prefer many small over few large
4. **Explicit over clever** - readable code wins
5. **No nested anonymous functions** - extract and name them
6. **Avoid `def` inside functions** - use `let` for local bindings
7. **Prefer `when` over `(if x y nil)`**
8. **Use `cond` over nested `if`**

### Anti-Patterns

```clojure
;; BAD: overloaded let
(let [a (f1)
      b (f2 a)
      c (f3 b)
      d (f4 c)]
  (f5 d))

;; GOOD: threading
(-> (f1) f2 f3 f4 f5)

;; BAD: nested anonymous
(map #(filter (fn [x] ...) %) colls)

;; GOOD: named function
(defn filter-valid [coll] (filter valid? coll))
(map filter-valid colls)
```

## Reference Documentation

- `references/stuart-sierra-rules.md` - Complete Stuart Sierra conventions
- `references/style-guide.md` - Comprehensive style guide

## Success Criteria

- [ ] Code follows naming conventions (kebab-case, predicates with `?`)
- [ ] Threading macros used appropriately (`->` vs `->>`)
- [ ] No nested anonymous functions

## Related Skills

- `clj-kondo-linting` - Static analysis enforcement
- `cljfmt-formatting` - Automatic formatting
