# cljfmt Configuration Reference

This document provides comprehensive reference for all cljfmt configuration options.

## Configuration Files

cljfmt searches for configuration files in parent directories in this order:
- `.cljfmt.edn`
- `.cljfmt.clj`
- `cljfmt.edn`
- `cljfmt.clj`

## Complete Configuration Options

### Formatting Behavior Options

#### `:indentation?` (boolean)
- **Default:** `true`
- **Description:** Enable or disable indentation correction
- **Example:**
  ```clojure
  {:indentation? true}
  ```

#### `:remove-trailing-whitespace?` (boolean)
- **Default:** `true`
- **Description:** Remove whitespace at the end of lines
- **Example:**
  ```clojure
  {:remove-trailing-whitespace? true}
  ```

#### `:remove-surrounding-whitespace?` (boolean)
- **Default:** `true`
- **Description:** Remove whitespace around inner forms (e.g., `( foo )` becomes `(foo)`)
- **Example:**
  ```clojure
  {:remove-surrounding-whitespace? true}
  ```

#### `:insert-missing-whitespace?` (boolean)
- **Default:** `true`
- **Description:** Add spaces between elements when missing
- **Example:**
  ```clojure
  {:insert-missing-whitespace? true}
  ```

#### `:remove-consecutive-blank-lines?` (boolean)
- **Default:** `true`
- **Description:** Collapse multiple consecutive blank lines
- **Example:**
  ```clojure
  {:remove-consecutive-blank-lines? true}
  ```
- **Related:** Use `:max-consecutive-blank-lines` to control how many blank lines are allowed

#### `:max-consecutive-blank-lines` (integer)
- **Default:** `2`
- **Description:** Maximum number of consecutive blank lines allowed
- **Example:**
  ```clojure
  {:max-consecutive-blank-lines 1}  ; Allow only single blank lines
  ```

#### `:remove-multiple-non-indenting-spaces?` (boolean)
- **Default:** `false`
- **Description:** Reduce multiple spaces between elements to single spaces
- **Example:**
  ```clojure
  {:remove-multiple-non-indenting-spaces? false}
  ```
- **Note:** When enabled, transforms `(foo  bar  baz)` to `(foo bar baz)`

#### `:indent-line-comments?` (boolean)
- **Default:** `false`
- **Description:** Align `;;` comments with the surrounding code indentation
- **Example:**
  ```clojure
  {:indent-line-comments? false}
  ```

#### `:sort-ns-references?` (boolean)
- **Default:** `false`
- **Description:** Sort namespace declarations alphabetically
- **Example:**
  ```clojure
  {:sort-ns-references? false}
  ```

#### `:split-keypairs-over-multiple-lines?` (boolean)
- **Default:** `false`
- **Description:** Break map key-value pairs across multiple lines
- **Example:**
  ```clojure
  {:split-keypairs-over-multiple-lines? false}
  ```

#### `:remove-blank-lines-in-forms?` (boolean)
- **Default:** `false`
- **Description:** Remove blank lines within form bodies
- **Example:**
  ```clojure
  {:remove-blank-lines-in-forms? false}
  ```

#### `:align-form-columns?` (boolean)
- **Default:** `false`
- **Description:** Align columns in forms (experimental feature)
- **Example:**
  ```clojure
  {:align-form-columns? false}
  ```
- **Note:** This is an experimental feature and may change

#### `:align-map-columns?` (boolean)
- **Default:** `false`
- **Description:** Align map key-value columns (experimental feature)
- **Example:**
  ```clojure
  {:align-map-columns? false}
  ```
- **Note:** This is an experimental feature and may change

### Indentation Configuration

#### `:indents` (map)
- **Default:** Built-in rules for Clojure core and common libraries
- **Description:** Completely replaces all default indentation rules
- **Example:**
  ```clojure
  {:indents {when [[:block 1]]
             defn [[:inner 0]]}}
  ```
- **Warning:** This replaces ALL default rules. Use `:extra-indents` to add to defaults instead.

#### `:extra-indents` (map)
- **Default:** `{}`
- **Description:** Add custom indentation rules while preserving defaults
- **Example:**
  ```clojure
  {:extra-indents {myapp.core/with-context [[:inner 1]]
                   myapp.routes/defapi [[:block 0]]
                   #"^def.*" [[:inner 0]]}}
  ```
- **Recommended:** Use this instead of `:indents` to avoid losing default rules

#### `:function-arguments-indentation` (keyword)
- **Default:** `:community`
- **Options:** `:community`, `:cursive`, `:zprint`
- **Description:** Choose function argument indentation style
  - `:community` - 1 space indent (Clojure Style Guide)
  - `:cursive` - 2 space indent (Cursive IDE style)
  - `:zprint` - zprint tool style
- **Example:**
  ```clojure
  {:function-arguments-indentation :community}
  ```
- **Visual comparison:**
  ```clojure
  ;; :community (1 space)
  (some-function arg1
   arg2
   arg3)

  ;; :cursive (2 spaces)
  (some-function arg1
    arg2
    arg3)
  ```

#### `:legacy/merge-indents?` (boolean)
- **Default:** `false`
- **Description:** Enable backward compatibility for `:indents` merging behavior
- **Example:**
  ```clojure
  {:legacy/merge-indents? true}
  ```
- **Note:** In versions before 0.11.x, `:indents` merged with defaults. Use this flag for backward compatibility.

### Runtime Options

#### `:file-pattern` (regex)
- **Default:** `#"\.clj[csx]?$"`
- **Description:** Regular expression pattern matching files to format
- **Example:**
  ```clojure
  {:file-pattern #"\.clj[csx]?$"}  ; Matches .clj, .cljc, .cljs, .cljx
  ```

#### `:parallel?` (boolean)
- **Default:** `false`
- **Description:** Process files concurrently for better performance
- **Example:**
  ```clojure
  {:parallel? true}
  ```

#### `:paths` (vector of strings)
- **Default:** `["src" "test"]`
- **Description:** Directories to scan for files to format
- **Example:**
  ```clojure
  {:paths ["src" "test" "dev"]}
  ```

#### `:exclude-paths` (vector of strings)
- **Default:** `[]`
- **Description:** Directories to exclude from formatting
- **Example:**
  ```clojure
  {:exclude-paths ["src/generated" "resources"]}
  ```

#### `:alias-map` (map)
- **Default:** `{}`
- **Description:** Namespace alias mappings for indentation rules
- **Example:**
  ```clojure
  {:alias-map {test clojure.test
               async clojure.core.async
               s schema.core}}
  ```
- **Usage:** When you have indentation rules like `test/deftest`, the alias map resolves `test` to `clojure.test`

## Complete Example Configuration

Here's a comprehensive configuration file showcasing most options:

```clojure
{;; Formatting behavior
 :indentation? true
 :remove-trailing-whitespace? true
 :remove-surrounding-whitespace? true
 :insert-missing-whitespace? true
 :remove-consecutive-blank-lines? true
 :max-consecutive-blank-lines 1
 :remove-multiple-non-indenting-spaces? false
 :indent-line-comments? false
 :sort-ns-references? false
 :split-keypairs-over-multiple-lines? false
 :remove-blank-lines-in-forms? false

 ;; Experimental alignment (use with caution)
 :align-form-columns? false
 :align-map-columns? false

 ;; Indentation style
 :function-arguments-indentation :community

 ;; Custom indentation rules
 :extra-indents {;; Project-specific macros
                 myapp.core/with-context [[:inner 1]]
                 myapp.routes/defapi [[:block 0]]
                 myapp.db/with-transaction [[:inner 1]]

                 ;; Pattern matching
                 #"^def.*" [[:inner 0]]
                 #"^with-.*" [[:inner 1]]
                 #".*-for-all$" [[:block 1]]}

 ;; Namespace aliases
 :alias-map {test clojure.test
             async clojure.core.async
             s schema.core
             m malli.core}

 ;; Runtime configuration
 :paths ["src" "test" "dev"]
 :exclude-paths ["src/generated"]
 :file-pattern #"\.clj[csx]?$"
 :parallel? true}
```

## Recommended Configuration for Teams

A good starting configuration for most Clojure projects:

```clojure
{:indentation? true
 :remove-surrounding-whitespace? true
 :remove-trailing-whitespace? true
 :insert-missing-whitespace? true
 :remove-consecutive-blank-lines? true
 :max-consecutive-blank-lines 1
 :function-arguments-indentation :community

 :extra-indents {;; Common patterns
                 #"^def.*" [[:inner 0]]
                 #"^with-.*" [[:inner 1]]
                 #"^let.*" [[:block 1]]
                 #".*-for-all$" [[:block 1]]}

 :paths ["src" "test" "dev"]
 :parallel? true}
```

## Configuration in Leiningen

For Leiningen projects, add to `project.clj`:

```clojure
:plugins [[dev.weavejester/lein-cljfmt "0.15.6"]]

:cljfmt {:load-config-file? true  ; Load .cljfmt.edn
         ;; Or specify inline configuration
         :indents {example.core/foo [[:inner 0]]}}
```

## Version Compatibility Notes

### Breaking Changes in v0.11.x

Prior to version 0.11.x, the `:indents` key merged with default indents. Starting from 0.11.x:
- `:indents` completely replaces defaults
- Use `:extra-indents` to add to defaults
- Add `:legacy/merge-indents? true` for backward compatibility

### Current Version (v0.15.6)

Latest stable version with all features documented above. To upgrade:

```bash
# Clojure tools
clj -Ttools install io.github.weavejester/cljfmt '{:git/tag "0.15.6"}' :as cljfmt

# Leiningen
:plugins [[dev.weavejester/lein-cljfmt "0.15.6"]]
```

## Troubleshooting Configuration

### Configuration Not Loading

1. Check file name spelling (`.cljfmt.edn` is most common)
2. Verify the file is in project root or parent directory
3. Check for syntax errors in EDN file
4. Use `:load-config-file? true` in Leiningen

### Indentation Not Working as Expected

1. Verify indent rules use correct format: `[[:inner N]]` or `[[:block N]]`
2. Check namespace qualification in indent rules
3. Use `:alias-map` if using namespace aliases
4. Remember `:indents` replaces ALL defaults - use `:extra-indents` instead

### Performance Issues

1. Enable `:parallel? true` for large codebases
2. Limit `:paths` to only necessary directories
3. Use `:exclude-paths` for generated code or large dependencies
4. Check `:file-pattern` isn't too broad

## Additional Resources

- Main repository: https://github.com/weavejester/cljfmt
- Indentation rules documentation: See `indentation.md` in this directory
- Clojure Style Guide: https://guide.clojure.style
